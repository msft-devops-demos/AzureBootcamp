# Lab 4 Part 2 - Azure Functions Proxies

[Back to REST and Microservices](Lab.md)

## Lab Summary

In this lab you're going to bring various RESTful function APIs together under a single API surface.

## Step 1: Create a new Function app in the Portal

We'll put our proxies in their own dedicated Function App.

- In the Azure Portal, create a new Function App called "TaxiApp" 

> ***Remember**: the name needs to be unique, so you can add your initials or some random characters to the name.*

## Step 2: Create Proxies to bring your APIs together

In your new function app, create the following proxies, ensuring you pass the appropriate parameters through with the request:

- **UserDetails**. Map the proxy to your published UserDetails services.
- **JourneyHistory**. Map this proxy to your published JourneyHistory service.

> **Note**: In your proxy route template, you can choose to change or remove the */api* section of the URL. In fact, you can change all of the route if you want to!

Test all your endpoints, making sure authentication works and your parameters are passed back and forth correctly.

## Step 3: Override a Response

The Contoso Cabs UX team are building the pages for the user profile section of the app. They have asked for an API to code against. Since we're not ready with the real API yet you'll provide a static response from the endpoint.

- Implement a response override on the /users/ endpoint to provide the following JSON for each request:

  ```JSON
  {
      "userId": 7,
      "userName": "Adam.Smith",
      "currentBalance": 20.56,
      "savedAddresses": [
      {
          "Line1": "123 Camden Road",
          "City": "London",
          "PostCode": "NW1234"
      },
      {
          "Line1": "443 City Street",
          "City": "Manchester",
          "PostCode": "MCC123"
      },
      {
          "Line1": "221B Baker St",
          "City": "London",
          "PostCode": "W1234"
      }]
  }
  ```

## Stretch Goals

If you've time, here are some extra objectives.

- Experiment with embedding request parameters in the response to provide a more dynamic-feeling mock API.
- Use App Settings to store the base URLs of your service functions and reference that in your backend URL.
- Explore the proxy JSON in the JSON editor in the portal.

### Hint

See the [Proxies Documentation](https://docs.microsoft.com/en-us/azure/azure-functions/functions-proxies) for help.

## Summary

You've now got 2 RESTful microservice skeletons being surfaced through a single API endpoint. You've also unblocked another team waiting for a version of your API to continue their work.
