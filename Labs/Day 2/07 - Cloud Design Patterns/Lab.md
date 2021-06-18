# Lab 6 - Design Patterns - Circuit Breaker

[Back to Main Menu](../../../README.md)

## Lab Summary

In this lab we'll experiment with the `Polly` library and implement calls with Retry & Circuit Breaker logic.

To do this we'll need:

- A Function App in Azure acting as the downstream system
- A local Function app acting as the 'client' making the calls

## Step 1: Create (or locate) a Function App in Azure

We need a simple HTTP Function app to return an OK response to the caller. You can either create + publish one, or use one you've made previously (perhaps the function you created in the Portal in Lab 1).

Make a note of the URL to this Function.

## Step 2: Create a new Local Function and Add `Polly`

- Create a new Function app project and add a HTTP triggered function to it.
- Add the `Polly` NuGet package.

>**Note:** Make sure you add version 7.2.0+ of Polly.

Above the `Run` method declare a new static variable for the `HttpClient`:

```CSharp
private static HttpClient client = new HttpClient();
```

Add a new method which will call the downstream service:

```CSharp
private static async Task<string> CallService()
{
    // call something here
    var response = await client.GetAsync("your-live-func-url-here");
    if(!response.IsSuccessStatusCode)
    {
        // this will get caught by the Polly Policy
        throw new HttpRequestException();
    }

    return await response.Content.ReadAsStringAsync();
}
```

Next, create and execute the policy in your `Run` method:

```CSharp
 var resp = await Policy
    .Handle<HttpRequestException>()
    .RetryAsync(10, (exception, retryCount) =>
    {
        log.LogWarning($"Retry number {retryCount}...");
    })
    .ExecuteAsync(async () => await CallService());

return new OkObjectResult(resp);
```

> This Policy will retry the call each time it catches the `HttpRequestException`, for 10 times before failing.

Set some breakpoints and execute the Function. It should succeed first time.

- Try stopping your live Function in Azure or changing the response to something other than an OkObjectResult (for example, you could throw an exception), and try this code again - you should see the retries being logged to the console.

### Further Retries

If needed, refer to the docs here: <https://github.com/App-vNext/Polly>

- What about `RetryForever` ?
- What about backing off the time between retries (`WaitAndRetry`)?

## Step 3: Circuit Breaker

For the purposes of this lab, we'll implement a Circuit Breaker per Function (no shared state). This would help isolate problematic Function instances.

Due to the ephemeral nature of Functions, we'll need to instantiate a static version of the Policy, as Polly will keep a count of errors across a number of executions. Above your run method add:

```CSharp
private static AsyncPolicy breaker = Policy
  .Handle<HttpRequestException>()
  .CircuitBreakerAsync(3,
      TimeSpan.FromSeconds(30),
      (exception, timeSpan) => onBreak(),
      () => onReset());
```

Add the two methods required to catch the break and reset activities:

```CSharp
private static void onBreak()
{
    // do something
}

private static void onReset()
{
    // do something
}
```

Now replace the code in your `Run` method with:

```CSharp
var resp = await breaker.ExecuteAsync(async () => await CallService());
return new OkObjectResult(resp);
```

Add a number of breakpoints and run the above code, with the live function stopped. You should notice the following:

- `CallService` will execute 3 times (**once per request** - we've removed any automatic retries here)
  - [Circuit: `closed`]
- Following the 3rd failure, `onBreak` will be called and further calls will not hit `CallService`
  - [Circuit: `open`]
- After 30 seconds `CallService` will be hit again
  - [Circuit: `half open`]
- Following 1 more failure, the circuit will break again.
  - [Circuit: `closed`]

### Further Circuit Breaking

Time permitting, refer to the docs and see if you could combine a Circuit Breaker with a Retry policy (look for `PolicyWrap`).

Try to log the circuit state to the console.

## Summary

Here you've experimented with Retry and Circuit Breaker concepts, and used a popular library to facilitate these - [Polly](http://www.thepollyproject.org/).
