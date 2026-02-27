---
name: fluentresults-best-practices
description: Best practices for FluentResults in .NET. Use this whenever the user asks about FluentResults, Result pattern, error handling without exceptions, or wants guidance on creating/processing Result and Result<T> in C#/.NET, even if they do not mention the library explicitly.
---

# FluentResults Best Practices

## Goal
Guide users to model expected failures with `Result`/`Result<T>`, design meaningful errors, and compose operations without exception-driven control flow.

## Output Expectations
- Provide succinct guidance and targeted C# examples.
- Prefer actionable steps and code patterns over theory.
- Call out boundary concerns (API serialization, logging, DTO mapping) when relevant.

## Decision Rules
- Use `Result`/`Result<T>` only when an operation can legitimately fail.
- Prefer results for business validation and expected failure modes.
- Reserve exceptions for truly exceptional or unexpected conditions.

## Core Usage
- Success: `Result.Ok()` or `Result.Ok(value)`
- Failure: `Result.Fail("message")`, `Result.Fail(new Error("message"))`, or `Result.Fail<T>("message")`
- Check: `result.IsSuccess`, `result.IsFailed`
- Value access: `result.Value` (throws on failure), `result.ValueOrDefault` (safe)

## Error and Success Modeling
- Create custom `Error`/`Success` subclasses for domain meaning.
- Use `WithMetadata(...)` for structured details (error codes, context, timestamps).
- Use `CausedBy(...)` to capture root causes (exception or nested errors).

## Construction Patterns
- Use guards: `Result.FailIf(...)`, `Result.OkIf(...)`, `Result.FailIfNotEmpty(...)`.
- Wrap exception-prone blocks with `Result.Try(...)`.
- Configure `Result.Setup(...)` to standardize error factories and exception mapping.

## Composition and Transformation
- Merge: `Result.Merge(...)` / `results.Merge()`.
- Transform values: `Map(...)`.
- Convert: `ToResult()` / `ToResult<T>()`.
- Chain: `Bind(...)` to stop on first failure and merge reasons.

## Boundary and Serialization
- Do not serialize FluentResults across boundaries (API, queue, job payload).
- Map to DTOs and keep contracts library-agnostic.

## Logging and Observability
- Register `IResultLogger` with `Result.Setup(...)`.
- Use `LogIfSuccess()` / `LogIfFailed()` or `Log(...)` with context and log level.

## Testing Guidance
- Assert `IsSuccess`/`IsFailed` explicitly.
- Verify expected error types or metadata via `HasError<T>` / `HasSuccess<T>`.

## Anti-Patterns
- Throwing for expected validation errors.
- Accessing `Value` without checking success.
- Returning `Result` for methods that cannot fail.
- Exposing `Result`/`Result<T>` in public API models.

## Examples
```csharp
public Result<MyEntity> Create(MyInput input)
{
    if (input is null)
        return Result.Fail<MyEntity>("Input is required");

    if (!IsValid(input))
        return Result.Fail<MyEntity>(new ValidationError(input.Errors));

    var entity = BuildEntity(input);
    return Result.Ok(entity);
}
```

```csharp
var result = Result.Try(() => LoadFromDisk(path));
if (result.IsFailed)
    return result.ToResult<MyDto>();

return Result.Ok(MapToDto(result.Value));
```

## References
- https://github.com/altmann/FluentResults/blob/master/README.md
