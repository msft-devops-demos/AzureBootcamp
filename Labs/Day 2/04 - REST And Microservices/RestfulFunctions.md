# Lab 4 Part 1 - REST Functions

[Back to REST and Microservices](Lab.md)

## Lab Summary

In this lab you're going to build the skeleton for some user profile management functions. The Contoso Cabs app will allow users to view their own profile and recent journey history.

You're going to build 2 Function Apps and implement RESTful URL's

## Step 1: Create a 'UserDetails' Function App

Our first function app will contain functions to return and update the user profile information.

- Create a new Functions app with 2 HTTP Triggered functions, and set the **routes** as described:
  - **GetUserDetails**. GET. URL = *users/{userId}*. Return a mock response containing the user id supplied.
  - **UpdateUserDetails**. POST. URL = *users*. Accept a UserDetails object and return a mock response.

Test locally. You should be able to call both in Postman, in the form: <http://localhost:7071/api/users/>

## Step 2: Create a 'JourneyHistory' Function App

A separate team is working on the Journey History functionality, in their own function app.

- Create a new Functions app call 'JourneyHistory', with 1 HTTP function:
  - **GetJourneyHistory**. GET. URL = *users/{userId}/journeyhistory*. Return a mock response containing the user id.

Test this locally. You should be able to call the function using the url <http://localhost:7071/api/users/4/journeyhistory>

## Step 3: Publish both functions to Azure

Publish the 2 function apps to Azure and test using the function URLs.

## Summary

In this lab you've created the skeleton of 2 RESTful microservice APIs. They are not fully implemented. Next we'll explore how we can unblock teams that need to use these APIs.
