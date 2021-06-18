# Lab 5 - Durable Functions

[Back to Main Menu](../../../README.md)

## Lab Summary

In this lab we are going to rethink the implementation of the Taxi Booking Request API from the earlier Lab. We are going to take advantage of Durable Function Orchestration to request a booking, and wait for a driver to accept.

### What you'll build

- The user will make a request to book a taxi.
- The API will:
  - Look up some user details
  - Log the request into a Queue and Table
  - Wait for a Driver to accept the job
  - Complete

## Step 1: Scaffolding

- Create a new Functions Project using select `Durable Functions Orchestration` as the template.
- Rename `Function1` function (created by default) with the name `BookingOrchstrator`.
- Add a new HTTP triggered function called `BookingStarter`. Go back to the `BookingOrchestrator.cs` file and notice that there is a starter function there called `BookingOrchestrator_HttpStart` with the method name `HttpStart`. Let's copy the code from the starter function in the Orchestrator class into  the `BookingStarter` function.
  - Tip: As we're using v3 of Azure Functions for this function, we'll need to modify the return statements slightly. Delete the existing return statement and replace it with:

    ```CSharp
    var asyncTracking = starter.CreateHttpManagementPayload(instanceId);
    return new OkObjectResult(asyncTracking);
    ```

- Remove the boilerplate Starter and Activity functions from the Orchestrator file.
- Create a new empty class file to store the following POCOs, brought over from the previous BookingRequest project:

```CSharp
public class BookingRequest
{
    public long UserId { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
    public string TrackingId { get; set; }
}

public class BookingRequestEntity
{
    public long UserId { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
}
```

## Step 2: Add Activity Functions

- Add 2 new HTTP Triggered functions to our project:
  - `GetUserName`
  - `LogRequest`

We need to change these functions from standard HTTP triggers to Activity triggers. Replace the `HTTPTrigger` attribute and parameter with

```CSharp
[ActivityTrigger] BookingRequest req,
```

This will allow them to be called by the orchestrator, and will accept a custom `BookingRequest` object, which represents our taxi booking request.

For the `GetUserName` function, add a delay or sleep to simulate some work and return a hard-coded string. This will simulate a data-store lookup to retrieve some user details.

```CSharp
await Task.Delay(1000);
return "John.Smith";
```

>**Note:** You will need to change the method return type to `Task<string>`

For the `LogRequest` function, we are going to replicate the functionality from the `BookingRequest` lab. It might be useful to review the code in that lab as a reminder. There will need to be some changes. This function should:

- Accept the `BookingRequest` object
- Output the request to a table
- Output the TrackingId to a queue

The `LogRequest` function code should look like this:

```CSharp
[FunctionName("LogRequest")]
public static async Task Run(
    [ActivityTrigger]BookingRequest req,
    [Table("bookingrequests", Connection = "StorageConnectionString")] IAsyncCollector<BookingRequestEntity> tableOutput,
    [Queue("bookingrequests", Connection = "StorageConnectionString")] IAsyncCollector<string> queueOutput,
    ILogger log)
{
    req.TrackingId = Guid.NewGuid().ToString();

    // output to Table
    var tableMsg = new BookingRequestEntity
    {
        UserId = req.UserId,
        FromAddress = req.FromAddress,
        ToAddress = req.ToAddress,
        PartitionKey = "RequestsTemp",
        RowKey = req.TrackingId
    };

    //Wait for the output to be sent to the queue and the table
    // Notice this time, we are just adding the `TrackingId` value to the queue.
    await Task.WhenAll(tableOutput.AddAsync(tableMsg),
        queueOutput.AddAsync(req.TrackingId));
}
```

>**Note:** You will need to add the `Microsoft.Azure.WebJobs.Extensions.Storage` NuGet package to the project.

In your `local.settings.json` file, set the value for `StorageConnectionString` to `UseDevelopmentStorage=true` to use the Azure storage emulator to run this locally.

## Step 3: Wire up the Orchestrator

First, add a new POCO class:

```CSharp
public class AcceptJob
{
    public int DriverId { get; set; }
    public string DriverName { get; set; }
}
```

Then replace the boilerplate code in the `BookingOrchestrator` class to achieve the following:

- Retrieve the custom `BookingRequest` object (context.GetInput<>)
- Call the `GetUserName` activity function, passing the request object in
- *Then* call the `LogRequest` activity, passing the request object in
- *Then* wait for a driver to accept the job (see the docs here: [Durable Functions External Events](https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-external-events))
- The Function should only return a `Task`
- Include an instance of `ILogger` so that we can log status out to the console for testing.

**Note:** the `WaitForExternalEvent<>` method can accept a custom object. This will be fed back into the orchestration when sent from an external client. We will use the `AcceptJob` class we added above for this.

The `BookingOrchestrator` function should now look like this:

```CSharp
[FunctionName("BookingOrchestrator")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context,
    ILogger log)
{
    // Get the input request
    var req = context.GetInput<BookingRequest>();

    // Get user details
    var userName = await context.CallActivityAsync<string>("GetUserName", req);
    log.LogInformation($"Got username {userName}");

    // Log the request to storage
    await context.CallActivityAsync("LogRequest", req);

    var driver = await context.WaitForExternalEvent<AcceptJob>("AcceptJob");
    log.LogInformation($"Driver accepted: {driver.DriverName} journey from {req.FromAddress} to {req.ToAddress} for user ID {req.UserId} with username {userName}");
}
```

## Step 4: Update your starter function

Your starter function needs to accept the custom `BookingRequest` object as a POST. It then needs to pass that to the orchestrator when it starts it.

The `BookingStarter` function should now look like this:

```CSharp
[FunctionName("BookingStarter")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] BookingRequest req,
    [DurableClient] IDurableOrchestrationClient starter,
    ILogger log)
{
    // Function input comes from the request content.
    string instanceId = await starter.StartNewAsync("BookingOrchestrator", req);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    var asyncTracking = starter.CreateHttpManagementPayload(instanceId);
    return new OkObjectResult(asyncTracking);
}
```

## Step 5: Test

Hit `F5` to run it locally.

Use Postman to construct a POST call to your starter function, and step through the code in Visual Studio.

Review the tracking information returned and check the status.

## Step 6: Send External Event

At this point your Orchestrator should be sleeping, waiting for the external event to wake it. You can send this in 2 ways: manually, via Postman - or via another HTTP function.

### Via Postman

Use the `sendEventPostUri` as a new `POST` request. Note that this URI contains the instance id already.

- Replace the {EventName} attribute in the URI with the one you specified in your orchestrator (`AcceptJob`).
- In the request body, send a json version of our `AcceptJob` class:

```JSON
{"DriverId": 271194, "DriverName": "Robert"}
```

>**Note:** All being well, you should see a log entry in the console stating the start and end address, the name of the driver and the user ID and username.

### Via another Function

You can also create another HTTP Triggered function that the driver app would call to accept the job

- Create a new HTTP triggered function
- Extend our `AcceptJob` to include a string instancedId
- Accept the `AcceptJob` class as a parameter in your function
- Add the `[DurableClient] IDurableOrchestrationClient client` binding
- Use the orchestration client to `RaiseEventAsync`. Don't forget to pass the `AcceptJob` object to the event.

## Step 7: Return all instances

Use the example at [Querying All Instances](https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-instance-management#querying-all-instances) to write a function that returns all running instances of the orchestration. Call your function `GetAllInstances`.

## Extending the scenario

Other features we could add here, time permitting.

- Custom statuses throughout the pipeline, so a client application could be kept informed of progress of finding a driver
- Add a timeout for the external event, to cancel the orchestration if nobody accepts the job in a reasonable timeframe
- Add some error handling in case user details can't be found, or the request cannot be logged

## Summary

In this lab you've created a new Durable Function Orchestration.
You are now familiar with the concepts of Orchestrators, Activities and Clients. You've built a serverless execution pipeline, including human interaction.
