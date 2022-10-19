# WebApiDus

### Existing Challenges

Web APIs will often return one or more responses to the user depending on the state of the request:

- One type to represent a request that has been rejected because the user is not authenticated
- Another type to represent a request contained invalid data
- Another type to represent the request was valid and the response contains some data

In a controller-based application, this is represented by some code that looks like this:

```cs
[HttpGet(Name = "GetWeatherForecast")]
public IActionResult Get(int id)
{
    return id % 2 == 0
        ? Ok(Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        }).ToArray())
        : NotFound();
}
```

The `IActionResult` interface is implemented by both the `Ok` and `NotFound` types. However, we lose the fidelity of expressing these states in the method's return type.

Minimal APIs work around this problem by providing a `Results<TypeA, ..., TypeN>` generic union type that users can leverage to represent the multiple return types.

```cs
app.MapGet("/minimal-dus/{id}", Results<Ok<string>, ProblemHttpResult> (int id) => {
    return id % 2 == 0
        ? TypedResults.Ok("Valid ID")
        : TypedResults.Problem("Invalid Id");
});
```

Support for DUs at the langauge level would provide a unified strategy for representing multiple return types across both controller-based and minimal APIs.

## Future Possibilities

In addition to helping rectify the existing deficiency with representing multiple response types, discriminated unions can also make it easy to represent multiple types that can be processed from the _request_.

Let's assume that a user has an API that expects to receive a `Todo` POCO as an input.

```cs
app.MapPost("/todos", (Todo todo) => ...);
```

We want to introduce a validation layer to this API, where the user can leverage union types and exhaustive pattern matching to enumerate through inputs that can either represent a validated `T` or some error type.

```csharp
union Result<T> = T | ValidationError;
app.MapPost("/todos, (Result<Todo> todo) =>
{
   return todo switch 
   {
      Todo todo => Results.Ok(todo)
      ValidationError e => Results.Problem(e);
   };
})
```
