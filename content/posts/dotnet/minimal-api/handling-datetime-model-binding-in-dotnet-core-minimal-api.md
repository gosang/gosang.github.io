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
