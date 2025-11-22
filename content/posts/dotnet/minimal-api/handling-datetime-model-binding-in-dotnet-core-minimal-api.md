+++
title = 'Handling Datetime Model Binding in Dotnet Core Minimal Api'
date = 2025-11-22T12:06:16Z
draft = true
+++

# Handling DateTime Model Binding in .NET 8 Minimal APIs

Working with dates and times in APIs is deceptively complex. In .NET 8 Minimal APIs, model binding automatically attempts to deserialize incoming JSON into your endpoint parameters. When the input format doesn’t match what the runtime expects, binding fails before your validation logic ever runs. This post explores how DateTime and DateOnly are bound in Minimal APIs, the pitfalls you’ll encounter, and best practices for handling them gracefully.

## Why Date Model Binding Is Tricky

- **Early binding failure**  
  Minimal API uses `System.Text.Json` to bind request bodies. If a date string isn’t in a supported format (ISO 8601 by default), deserialization throws a `BadHttpRequestException` before your endpoint executes.

- **Multiple date types**  
  `DateTime`, `DateTimeOffset`, and `DateOnly` each have distinct semantics. Using the wrong type can introduce timezone bugs or unnecessary precision.

- **Culture and format variance**  
  Clients often send dates in formats like `25-11-22T11:38:48.467Z`. These fail because `System.Text.Json` expects ISO 8601 (`yyyy-MM-ddTHH:mm:ssZ`).

- **Validation skipped**  
  If binding fails, FluentValidation never runs. You lose control over error messages and consistency.

## Approaches to Handling Dates

### 1. Native Binding (ISO 8601)

- **Pros:** Simple, fast, no extra code.
- **Cons:** Fails for non-ISO inputs, poor error messages.
- **Use when:** You fully control client formats.

### 2. Custom JSON Converters

- **Pros:** Enables clean `DateOnly` usage with consistent format.
- **Cons:** Still fails early if input doesn’t match expected format.
- **Use when:** You want strict `yyyy-MM-dd` semantics.

### 3. String DTO + FluentValidation (**Recommended**)

- **Pros:** Binding always succeeds (strings), validation controls parsing and error messages.
- **Cons:** Requires mapping to domain model after validation.
- **Use when:** You need robust, user-friendly error handling.

### 4. Endpoint Filters

- **Pros:** Centralizes validation, keeps endpoints clean, adheres to SRP/OCP.
- **Cons:** Requires validator registration and filter attachment.
- **Use when:** You want reusable validation across endpoints.

## Implementation Walkthrough

### Transport vs Domain Models

```csharp
// Transport DTO — binds reliably as strings
public record AppointmentRequestDto(string StartDate, string EndDate, string DateOfBirth);

// Domain model — strongly typed after validation
public record Appointment(DateTime StartDate, DateTime EndDate, DateOnly DateOfBirth);
```

#### FluentValidation Rules

```csharp

public class AppointmentRequestDtoValidator : AbstractValidator<AppointmentRequestDto>
{
    public AppointmentRequestDtoValidator()
    {
        RuleFor(x => x.StartDate)
            .NotEmpty().WithMessage("Start date is required.")
            .Must(BeValidDateTime).WithMessage("Start date must be a valid ISO date (yyyy-MM-ddTHH:mm:ssZ).");

        RuleFor(x => x.EndDate)
            .NotEmpty().WithMessage("End date is required.")
            .Must(BeValidDateTime).WithMessage("End date must be a valid ISO date (yyyy-MM-ddTHH:mm:ssZ).")
            .Must((dto, end) => EndAfterStart(dto.StartDate, end))
            .WithMessage("End date must be after start date.");

        RuleFor(x => x.DateOfBirth)
            .NotEmpty().WithMessage("Date of birth is required.")
            .Must(BeValidDateOnly).WithMessage("Date of birth must be a valid date (yyyy-MM-dd).")
            .Must(BeAdult).WithMessage("You must be at least 18 years old.");
    }

    private static bool BeValidDateTime(string value) => DateTime.TryParse(value, out _);
    private static bool BeValidDateOnly(string value) => DateOnly.TryParse(value, out _);

    private static bool EndAfterStart(string start, string end)
    {
        if (DateTime.TryParse(start, out var s) && DateTime.TryParse(end, out var e))
            return e > s;
        return false;
    }

    private static bool BeAdult(string dob)
    {
        if (!DateOnly.TryParse(dob, out var date)) return false;
        var today = DateOnly.FromDateTime(DateTime.UtcNow);
        var age = today.Year - date.Year;
        if (date > today.AddYears(-age)) age--;
        return age >= 18;
    }
}

```

#### DateOnly JSON Converter

```csharp

public sealed class DateOnlyJsonConverter : JsonConverter<DateOnly>
{
    private const string Format = "yyyy-MM-dd";

    public override DateOnly Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        => DateOnly.ParseExact(reader.GetString()!, Format);

    public override void Write(Utf8JsonWriter writer, DateOnly value, JsonSerializerOptions options)
        => writer.WriteStringValue(value.ToString(Format));
}

// Program.cs
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.Converters.Add(new DateOnlyJsonConverter());
});

```

#### ValidationFilter for Clean Endpoints

```csharp
public class ValidationFilter<T> : IEndpointFilter where T : class
{
    private readonly IValidator<T> _validator;
    public ValidationFilter(IValidator<T> validator) => _validator = validator;

    public async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext ctx, EndpointFilterDelegate next)
    {
        var request = ctx.Arguments.OfType<T>().FirstOrDefault();
        if (request is null) return Results.BadRequest(new { error = "Invalid request payload." });

        var result = await _validator.ValidateAsync(request);
        if (!result.IsValid)
        {
            var errors = result.Errors.Select(e => new { e.PropertyName, e.ErrorMessage });
            return Results.BadRequest(new { errors });
        }

        return await next(ctx);
    }
}

```

#### Endpoint

```csharp
app.MapPost("/appointments", (AppointmentRequestDto dto) =>
{
    var appointment = new Appointment(
        DateTime.Parse(dto.StartDate),
        DateTime.Parse(dto.EndDate),
        DateOnly.Parse(dto.DateOfBirth));

    return Results.Ok(new { message = "Appointment created successfully.", appointment });
})
.AddEndpointFilter<ValidationFilter<AppointmentRequestDto>>();

```

#### Example Error Response

Input:

```json
{
  "startDate": "25-11-22T11:38:48.467Z",
  "endDate": "25-11-22T11:38:48.467Z",
  "dateOfBirth": "25-11-22"
}
```

Response:

```json
{
  "errors": [
    {
      "propertyName": "StartDate",
      "errorMessage": "Start date must be a valid ISO date (yyyy-MM-ddTHH:mm:ssZ)."
    },
    {
      "propertyName": "EndDate",
      "errorMessage": "End date must be a valid ISO date (yyyy-MM-ddTHH:mm:ssZ)."
    },
    {
      "propertyName": "DateOfBirth",
      "errorMessage": "Date of birth must be a valid date (yyyy-MM-dd)."
    }
  ]
}
```

## Best Practices

- Accept strings, validate explicitly

  - Prevent binding failures by using string DTOs and parsing in validators.

- Use the right type DateOnly for birthdays and schedules,

  - DateTimeOffset for absolute instants, DateTime only when timezone is irrelevant.

- Centralize validation
- Use endpoint filters to keep endpoints clean and consistent.

- Document formats

  - Clearly specify expected formats in OpenAPI/Swagger.

- Test edge cases
  - Validate leap years, DST transitions, and age boundaries.

## Conclusion

Date handling in Minimal APIs is about designing contracts that survive real-world variability. By accepting strings, validating with FluentValidation, and converting to strong types only after validation, you gain control over error messages and consistency. With DateOnly for date-only values, custom converters, and reusable filters, your API becomes robust, predictable, and production-ready.

## References

- [Microsoft Docs: Minimal APIs overview](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-10.0)
- [System.Text.Json Date and Time support](https://learn.microsoft.com/en-us/dotnet/standard/datetime/system-text-json-support)
- [FluentValidation documentation](https://docs.fluentvalidation.net/en/latest/)
- [DateOnly struct in .NET](https://learn.microsoft.com/en-us/dotnet/api/system.dateonly?view=net-10.0)
