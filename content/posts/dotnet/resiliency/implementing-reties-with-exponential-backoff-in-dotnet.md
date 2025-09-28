+++
title = 'Implementing Reties With Exponential Backoff in .NET'
date = 2025-09-28T12:15:51+01:00
draft = true
+++

In this blog, we’ll move from the theory of exponential backoff to production-ready patterns you can safely apply in .NET 8. Along the way, we’ll cover common pitfalls, practical best practices, and provide concise code samples you can can plug into your projects. Specifically, we’ll explore:

- Why exponential backoff combined with jitter is the recommended strategy for handling transient remote failures.
- How to integrate IHttpClientFactory with Polly to build resilient retry policies, add circuit breakers, and define sensible defaults in .NET 8.
- Clear, working code examples using typed clients and DI registration.
- When retries make sense, when they don’t, and how to avoid the hidden disadvantages of overusing them.

By the end, you’ll have both the rationale and the tools to implement resilient HTTP calls in your applications with confidence.

## What is exponential backoff (and jitter) - rationale & core idea?

- **Exponential backoff**: when an operation fails transiently, retry it after increasing delays, typically `base * 2^attempt`. This gives the remote system time to recover and prevents immediate retry storms.

- **Jitter**: add randomness to retry delays so many clients don't all retry at the same instant (which would cause a “thundering herd” and worsen recovery). AWS and other cloud teams recommend combining exponential backoff with jitter as the most robust pattern for distributed clients.

### Why use it?

- Controls client retry traffic against overloaded services.
- Increases probability of success for transient faults (network blips, rate limits, temporary 5xx).
- Works well combined with other resilience primitives (circuit breaker, timeout, bulkhead).

**Key trade-offs**: retries add latency and duplicate attempts (possible side-effects). So we apply retries only for transient errors or idempotent operations and always combine with timeouts, limits, and observability.

## High-level pattern with .NET 8

- Use `IHttpClientFactory` (centralized HttpClient config, handler lifetime, DNS fixes, DI).
- Use `Polly` to express retry policies and other resilience strategies, attach policies via `.AddPolicyHandler(...)` on `HttpClient` registrations.
- Add jitter (randomized delay) or use Polly.Extensions/Polly.Contrib helper backoffs (decorrelated jitter).

## NuGet packages

```bash
dotnet add package Polly
dotnet add package Microsoft.Extensions.Http.Polly
# optional (recommended) for better jitter strategies:
dotnet add package Polly.Contrib.WaitAndRetry

```

Polly v8 introduced new pipelines; the classic `AddPolicyHandler` approach remains common.

## Example: Minimal .NET 8 app — typed client + retry + circuit breaker + jitter

Below example shows:

- Typed client MyApiClient.
- Registration with IHttpClientFactory.
- Retry policy: exponential backoff + jitter, respects 429/Retry-After when available.
- Circuit breaker for escalation control.

File: `Program.cs` (minimal host)

```csharp
using System.Net;
using System.Net.Http.Headers;
using Microsoft.Extensions.DependencyInjection;
using Polly;
using Polly.Extensions.Http;
using Polly.Contrib.WaitAndRetry; // optional, for decorrelated jitter
using System.Text.Json;

// minimal host building
var builder = WebApplication.CreateBuilder(args);

// Register typed HttpClient with policies
builder.Services.AddHttpClient<MyApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com/");
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
})
// only retry GET/HEAD by policy (protect non-idempotent methods)
.AddPolicyHandler((request) =>
{
    if (request.Method == HttpMethod.Get || request.Method == HttpMethod.Head)
        return GetRetryPolicy();
    // No-op for other methods (POST/PUT/etc.). You may choose to return a custom policy.
    return Policy.NoOpAsync<HttpResponseMessage>();
})
.AddPolicyHandler(GetCircuitBreakerPolicy());

var app = builder.Build();

app.MapGet("/", async (MyApiClient api, CancellationToken ct) =>
{
    var data = await api.GetResourceAsync(ct);
    return Results.Text(data);
});

app.Run();


// -- typed client implementation
public sealed class MyApiClient
{
    private readonly HttpClient _http;

    public MyApiClient(HttpClient http) => _http = http;

    public async Task<string> GetResourceAsync(CancellationToken ct = default)
    {
        using var resp = await _http.GetAsync("resource", ct);
        resp.EnsureSuccessStatusCode();
        return await resp.Content.ReadAsStringAsync(ct);
    }
}


// -- policies

static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    // Option A: simple exponential backoff + small jitter
    var jitterer = new Random(); // or Random.Shared

    return HttpPolicyExtensions
        .HandleTransientHttpError() // 5xx, 408 or HttpRequestException
        .OrResult(msg => msg.StatusCode == (HttpStatusCode)429) // TooManyRequests (rate limiting)
        .WaitAndRetryAsync(
            retryCount: 5,
            sleepDurationProvider: (retryAttempt, response, context) =>
            {
                // Respect server Retry-After header if provided
                if (response?.Result?.Headers?.RetryAfter is var retryAfter && retryAfter != null)
                {
                    // Prefer server-specified wait
                    if (retryAfter.Delta.HasValue)
                        return retryAfter.Delta.Value;
                    if (DateTimeOffset.TryParse(retryAfter.Date?.ToString(), out var date))
                        return date - DateTimeOffset.UtcNow;
                }

                // Exponential backoff (2^attempt seconds) + jitter up to 1 second
                var exponential = TimeSpan.FromSeconds(Math.Pow(2, retryAttempt));
                var jitter = TimeSpan.FromMilliseconds(jitterer.NextDouble() * 1000);
                return exponential + jitter;
            },
            onRetry: (outcome, timespan, retryAttempt, context) =>
            {
                // Hook for diagnostics: integrate with ILogger in real apps
                Console.WriteLine($"Retry {retryAttempt} after {timespan} because " +
                                  $"{(outcome.Exception != null ? outcome.Exception.Message : outcome.Result.StatusCode.ToString())}");
            }
        );
}

static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 3,
            durationOfBreak: TimeSpan.FromSeconds(30),
            onBreak: (outcome, breakDelay) =>
            {
                Console.WriteLine($"Circuit broken for {breakDelay}. Reason: {outcome.Exception?.Message ?? outcome.Result.StatusCode.ToString()}");
            },
            onReset: () => Console.WriteLine("Circuit reset"),
            onHalfOpen: () => Console.WriteLine("Circuit half-open (testing)")
        );
}

```

- `HttpPolicyExtensions.HandleTransientHttpError()` matches `HttpRequestException`, HTTP 5xx and 408. We add 429 explicitly to catch rate-limiting responses.

- We only attach the retry policy to GET/HEAD requests (idempotent) — you can choose a different strategy for non-idempotent calls (e.g., idempotency keys, no retry, or special server support).

- We check `Retry-After` header and prefer the server-specified delay if present (helps cooperate with server throttling responses). The `WaitAndRetryAsync` overloads allow supplying the response in the sleep provider so you can implement this behavior.

## Better jitter: decorrelated jitter (Polly.Contrib.WaitAndRetry)

Polly community recommends decorrelated jitter formulas to give better distribution and avoid pathological retry schedules. The `Polly.Contrib.WaitAndRetry` package provides helpers such as `Backoff.DecorrelatedJitterBackoffV2(...)` you can pass into `WaitAndRetryAsync(...)`. Using these yields significantly improved behavior versus simple `2^n + random` jitter.

Example snippet (replace `sleepDurationProvider` with `Backoff.DecorrelatedJitterBackoffV2(...) ` sequence).

## Advantages & disadvantages

### Advantages

- Improved availability for transient faults.
- Graceful handling of rate limiting and temporary outages.
- When combined with circuit breakers, reduces load on failing services.

### Disadvantages

- Additional latency for the client (retries increase end-to-end time).
- Retries can cause duplicate side-effects if requests are not idempotent.
- Poorly designed retries (no jitter, no limits) can create retry storms and cascading failures.

## When to use retries (and when NOT to)

Use retries when:

- Errors are transient (temporary network failures, timeouts, rate-limits, recoverable 5xx).

- The operation is idempotent, or you have explicit idempotency support on the server (idempotency key).

- You control maximum retries and total retry time; you have time budget for eventual retry latency.

Don’t retry when:

- The operation has side effects you cannot safely run twice (e.g., payment capture without idempotency).

- The error indicates a permanent failure (4xx that signals client error other than 429).

- The system is overloaded and retrying will just amplify load (use circuit breaker, back-pressure, server-side throttling).

## Issues, considerations & best practices

- **Idempotency first**: Prefer server-side idempotency (idempotency keys) when you must retry non-GET operations (e.g., safe POST retries). If not available, avoid automatic retries for non-idempotent calls.

**Use jitter**: Always add jitter (randomness). Even small jitter reduces coordination between clients massively. AWS and the community recommend jittered backoffs.

**Respect server hints**: If the server sends Retry-After, prefer it over your computed delay (cooperative throttling). Polly supports sleep providers that inspect HttpResponseMessage to implement this.

**Combine with circuit breaker**: Retries fight transient faults; circuit breakers limit retries when faults are persistent and give the callee time to recover.

**Limit attempts & total wait**: Cap retry count and optionally cap total wait time; don’t retry indefinitely.

**Timeout and cancellation**: Use an overall timeout or cancellation token per logical operation to bound the total time (per attempt timeouts + retry delays). For example, combine a TimeoutPolicy with retry.

**Observability**: Emit metrics for retry count, failure types, and circuit state. Log reasons in onRetry and onBreak. Include correlation IDs in requests.

**Test your policies**: Unit/integration test policy behavior (simulate transient responses). Use MockHttpMessageHandler (e.g., RichardSzalay.MockHttp) or a local test server to assert retry/circuit behavior.

**Avoid application-layer throttling bypass**: If the server implements rate limiting, respect that and expose back-pressure to the caller. If you must retry, expose the Retry count and expected waiting time through the API so callers can adapt.

**Use policy registries for reuse**: Register policies in a PolicyRegistry to share across clients and to allow runtime inspection/change.

**When to consider more advanced alternatives**

- If you need very high throughput and gentle back-pressure: use bulkhead isolation + rate limiting instead of aggressive retries.
- If you want advanced resiliency pipelines, explore Polly v8's ResiliencePipeline + Microsoft Http extensions for resilience (newer APIs/libraries are emerging)—evaluate migration costs.

## Testing & diagnostics tips

Inject an ILogger into retry onRetry to log attempt numbers, elapsed time, and reason.

For integration tests, create a test server that returns a configured sequence of responses (e.g., 500, 500, 200) and assert that the client returns success and retried the expected number of times.

For unit tests, mock HttpMessageHandler to simulate HttpResponseMessages and exceptions.

## Quick checklist before you ship

- Are retries only applied to safe/idempotent operations?
- Do retries include jitter?
- Are retry counts and total timeout bounded?
- Do you respect Retry-After headers?
- Are circuit breaker and timeout policies in place?
- Do you have visibility (metrics/logs) for retry behavior?
- Are client requests shortened by per-attempt timeouts and overall operation deadline?

## References

- [Use the IHttpClientFactory interface and guidance (setup/configuration)](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)

- [Microsoft - Implement HTTP call retries with exponential backoff with Polly” (microservices/resilience guidance and examples)](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)

- [Polly Retry strategy docs (how retry strategies work and options)](https://www.pollydocs.org/strategies/retry.html)

- [AWS Architecture Blog - Exponential Backoff and Jitter (why to use jitter and common patterns)](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)

- [Microsoft .NET docs on the Http.Resilience package and built-in resilience pipelines in .NET 8 (how to use AddStandardResilienceHandler / HttpRetryStrategyOptions)](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience?tabs=dotnet-cli)
