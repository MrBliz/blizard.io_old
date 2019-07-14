Title: Authenticating with Auth0 using Function Monkey
Published: 2019-07-05
Tags: 
    - Auth0
    - Function Monkey
    - Azure Functions
    - Net
    - Azure

---

[Function Monkey](https://functionmonkey.azurefromthetrenches.com) is a great framework for Azure Functions by [James Randall](https://twitter.com/AzureTrenches) (AKA [Azure from the Trenches](https://www.azurefromthetrenches.com)) that uses the Mediation pattern, and includes dependency injection out of the box, as well as other niceties such as support for generating openAPI docs, Fluent Validation, and a fluent configuration API.

In this post i'll show you how to integrate Function Monkey with Auth0 so that functions can be locked down to only logged in users. The scope of this post is restricted to back end logic. 

### What we'll be using in this post

Function Monkey

AuthO

Postman

Create an Azure Functions App using .Net Core 2.2x. You should have a minimal project structure that looks something like this (I'm using Rider on Mac).

Next install two Function Monkey Nuget packages




First of all if you haven't got one already, got to auth0.com and create a free account. 

After that, create an application and give it any name you like, and under application type, select Regular Web Application, and click on Create.



The domain is *not the domain of your app**. It's the domain of Auth0's openId configuration file. you can find this in Auth0's settings page.

If you get the domain wrong you will get an error something like this.

` Microsoft.IdentityModel.Protocols: IDX20803: Unable to obtain configuration from: '[PII is hidden]'`

The `audience` variable is whatever identifier you gave to your API in Auth0. if you get this wrong then the token bearer will throw  a `SecurityTokenException`.

You may need to clean your solution if you change these values in `local.settings.json` to ensure the function app picks up the changes.


### Manual Testing

We'll use Postman to test our endpoint. If you copy the url from the console that 
In Postman test that the API works with the the Authentication, by navigating to the Authorization tab, and selecting 'bearer token from the type dropdown.

Go to Auth0's management dashboard, and click on 'API's' in the side bar, select the API you created earlier, and then click on the 'test' tab. Copy the token from the response section , and then paste that into postman. 

If all goes well you will get a 200 response.

In the next post, i will investigate how to restrict access to a resource using function monkey and Auth0 using claims, ie: authenticating that a user is allowed to access a resource.

Free free to contact me on [Twitter](https://twitter.com/LiamBlizard) if you have any questions about this tutorial. Happy coding!


* and yes i did lose an hour or so to me being stupid and not realising this.