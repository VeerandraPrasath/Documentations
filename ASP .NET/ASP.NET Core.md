﻿# ASP .NET CORE

## Frontend

A front‑end application is responsible for delivering content to the browser. It has to be content the browser can understand such as HTML, CSS, and JavaScript. We learned that one of the functionalities a web application can provide is accessing data in a data store, such as a database. 
## Backend
Application concerned with data and business rules. This type of application is called a back‑end application, and a type of back‑end application that provides data and applies business rules is called a Web API or a short API.

## What to build with ASP .NET
![Alt text](Asserts/ASPUsage.JPG?raw=true)
 # ASP.NET Core Web API Fundamentals
 ## Entity Framework Core
 Establish a connection between the application and the database. It is an Object Relational Mapper (ORM) that allows us to work with a database using .NET objects.

 ## ASP .NET core big picture

 * Build web apps and services,IOT apps and mobile backends
 * Develop on Windows,Linux and macOs
 * Deploy to the cloud or on-premises
 * Runs on .Net
## ASP .NET Core History
![Alt text](Asserts/history.JPG?raw=true)
### Approaches
* MVC
* Minimal APIs
### Shared Framework
 A shared framework is a set of assemblies (.dll files) installed on a machine, which includes a runtime component and a targeting pack. In the context of .NET Core, the ASP.NET Core shared framework contains fully developed and supported assemblies by Microsoft, allowing multiple applications to utilize these libraries from a central location.

## Run using CLI
Go to the directory where the project is located and run the following command:
```bash
dotnet run --launch-profile https
```
## FrameWork Services
* Identity
* MVC
* Entity Framework core
## Program.cs
```bash 
  var builder = WebApplication.CreateBuilder(args);

            // Add services to the container.

            builder.Services.AddControllers();
            // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            var app = builder.Build();

            // Configure the HTTP request pipeline.
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseHttpsRedirection();

            app.UseAuthorization();


            app.MapControllers();

            app.Run();
## Services
 Once we have a builder, the services collection on that builder is what we can add services to while configuring them. This is the built‑in dependency injection container. So by adding services to it, we can inject them wherever we need them later on in our code.
 ```bash
 builder.Services.AddControllers();
 ```
## Application Services
There's also application services, which are application‑specific and which we can add ourselves.
* AddControllers() 
* AddEndpointsApiExplorer()
* AddSwaggerGen()
### AddControllers
* Calls the register services
* It is needed when we need to build a Web API like support for controllers, model binding, data annotations, and formatters.
### AddEndpointsApiExplorer && AddSwaggerGen
AddEndpointsApiExplorer and AddSwaggerGen both have to do with the Swagger documentation . These statements register the required services on the container that are needed by Swashbuckle's Swagger implementation.
### Builder.Build()
 * This results in an object of type WebApplication, which implements IApplicationBuilder
 * Instances of classes that implement this interface provide the mechanisms to configure an application's request pipeline. By doing so, we specify how an ASP.NET core application will respond to individual HTTP requests. The components that potentially handle these requests are called middleware.


 ### app.UseSwagger() and app.UseSwaggerUI()
 app.UseSwagger and app.UseSwaggerUI. These two pieces of middleware ensure that an OpenAPI specification is generated and a documentation UI that uses that generated OpenAPI specification is shown

 ### Request pipeline
 Whenever an HTTP request comes in, something must handle that request, so it eventually results in an HTTP response. Those pieces of code that can handle requests and result in responses make up the request pipeline
 <br><br>    Configure that request pipeline by adding middleware
 ### Middleware
  Middleware which are software components that are assembled into an application pipeline to handle requests and responses
  <br><br>
 Once a request reaches our application through Kestrel, it is processed by a number of steps that make up the request pipeline. These steps are called middleware. 
 ### ASP.NET core Request pipeline & Middleware
  The ASP.NET Core request pipeline consists of a sequence of request delegates called one after the other from one piece of middleware to the next. Each of those has the opportunity to perform operations before and after the next delegate, and that eventually results in the response we need. Important to know is that each component chooses whether to pass the request on to the next component in the pipeline or not. So the order in which we add middleware to the request pipeline matters. 
  ![Alt text](Asserts/requestPipeline.JPG?raw=true)
  #### Example 
  Example would be authentication middleware. If the middleware component finds that the request is not authorized, it will not pass it on to the next piece of middleware, but it will immediately return an unauthorized response. It's thus important that the authentication middleware in case of our example is added before other components that shouldn't be triggered in case of an unauthorized request.
 ## app.Environment.IsDevelopment
 check whether the current environment is a development environment. By surrounding middleware with it, we signify that the Swagger‑related middleware will only be added when we're running in the development environment.

 ## MVC
 ![Alt text](Asserts/mvc.JPG?raw=true)
 ## Building Controller
 ### ControllerBase
 ControllerBase contains basic functionality controllers need, like access to the model state, the current user, and common methods for returning responses. You could also derive from controller, but that Base class contains additional helper methods to use when returning views, which isn't needed when building an API

 ### ApiController attribute
 This isn't strictly necessary. But doing so, configures this controller with features and behavior aimed at improving the development experience when building APIs. Think about requiring a certain type of routing, automatically returning a 400 Bad Request on bad input, and returning problem details on errors
 ## Routing
 * Routing matches a request URI to an action on a controller.
 *  the preferred way to set this up is through endpoint routing.
 *  To set up endpoint routing, two pieces of middleware must be injected in the request pipeline  by calling into app.UseRouting and app.UseEndpoints
 ### App.UseRouting
 App.UseRouting marks the position in the middleware pipeline where an endpoint is selected.
 ### App.UseEndpoints
 app.UseEndpoints marks the position in the middleware pipeline where the selected endpoint is executed
 ### App.UseRouting  & App.UseEndpoints
 This allows us to inject middleware that runs in between selecting the endpoint and executing the selected endpoint
 ## Mapping Endpoints
 Need to map the endpoints, though. There's two ways of setting these up, convention based or attribute based. For APIs, attribute‑based routing should be used, so we will focus on that. MapControllers add endpoints for our controller actions, but without specifying routes. In other words, by calling into this, no conventions will be applied. This is the preferred approach for APIs.
  ![Alt text](Asserts/mappingControllers.JPG?raw=true)
 ### MapControllers
  MapControllers is a method that adds the necessary endpoints to the request pipeline for attribute routing. This means that the routing information is defined on the controller and action methods themselves, rather than in a centralized location.
  ### MapControllerShortcut
  Instead of adding the Routing middleware and Endpoint middleware to the request pipeline manually and then calling MapControllers when setting up the Endpoint middleware, you can omit adding the middleware to the request pipeline manually, and simply call MapControllers on the WebApplication object. That's because this WebApplication object implements IEndpoint routing
 ![Alt text](Asserts/mappingControllerShortcut.JPG?raw=true) 
 ## Attribute Based Routing
  ![Alt text](Asserts/attributebasedrouting.JPG?raw=true)
## Problem Details
```csharp
   builder.Services.AddProblemDetails(options =>
            {

                options.CustomizeProblemDetails = ctx =>
                {
                    ctx.ProblemDetails.Extensions.Add("additionalInfo", "Additional info example");
                    ctx.ProblemDetails.Extensions.Add("server", Environment.MachineName);
                };
            });
```
No external nuget packages are needed.
![Alt text](Asserts/describingErrorDetails.JPG?raw=true)
```url
datatracker.ietf.org/doc/html/rfc7807
```
The reasoning behind that is that sometimes HTTP status codes are not sufficient to convey enough information about an error to be helpful. In those cases, a response body that follows a specific standard, like the Problem Detail standard, is very useful.
![Alt text](Asserts/problemDetails.JPG?raw=true)
## Content Negotiation
This is the process of selecting the best representation for a given response when there are multiple representations available.
## Formatters
Input formatters are used by Model Binding. Output formatters are used to format responses<br><br>
Formatters are responsible for serializing and deserializing the data. They are used to convert the data from the server to a format that can be sent over the wire, and vice versa. ASP.NET Core supports multiple formatters out of the box, including JSON, XML, and plain text.
if that Accept header has a value of application/json, the consumer states that if your API supports the requested format, it should return that format. If it has a value of application XML, it should return an XML representation, and so on. If no accept error is available or if it doesn't support the requested format, it can always default to its default format.
![Alt text](Asserts/formatters.JPG?raw=true)
## ObjectResult
ObjectResult is a type of ActionResult that can be used to return any type of object. It is the base class for all action results that return an object, including JsonResult and FileResult. ObjectResult is used when you want to return an object that is not a file or a view.

Support for content negotiation from our actions is implemented by ObjectResult. And it's also built into the status code specific action results returned from the helper methods, like the OK and NotFound helper methods we've been using. These action result helper methods are based on ObjectResult and thus support content negotiation. Were we to only return a model class from our actions, it's automatically wrapped in an ObjectResult so that, too, supports content negotiation. We do not have to do anything extra for that. All we need to do to ensure that these ObjectResult deriving results support the format we want is add or configure the correct formatter

![Alt text](Asserts/objectResult.JPG?raw=true)

## Support Formatter

We can also send Not supported response when the requested format is not supported. This is done by adding a NotAcceptable formatter to the list of formatters. This formatter will return a 406 Not Acceptable response if the requested format is not supported.

```csharp
builder.Services.AddControllers(options =>
            {
                options.ReturnHttpNotAcceptable = true; //If the client does not accept the response type, return 406 Not Acceptable.If will not accept the request of the client because the response type will not support the client.
            }).AddXmlDataContractSerializerFormatters(); //Here we are specifying that we can send the reponse in xml format,so it accept the request if the requested format is xml.

```
![Alt text](Asserts/xmlFormat.JPG?raw=true)


In the below figure we can see that the xmlFormatter is included.So that it will return the response as xml.
![Alt text](Asserts/supportFormatters.JPG?raw=true)

## Download File
![Alt text](Asserts/downloadFile.JPG?raw=true)

# Manipulating Resources and Validating Input
## Passing Data to the API
 * FromBody is inferred for complex type parameters. A good example would be adding a point of interest, which we'll immediately tackle. Types with special meanings like CancellationToken, IFormCollection and so on are ignored.
 * FromForm then is inferred for action parameters of type IFormFile and IFormFileCollection. It is not inferred for any simple or user‑defined types. 
 * FromRoute is inferred for any action parameter name matching a parameter in the route template. When more than one route matches an action parameter, any route value is considered FromRoute. And lastly,
 * FromQuery is inferred for any other action parameters

 ![Alt text](Asserts/fromBodyAndForm.JPG?raw=true)
 ![Alt text](Asserts/fromRouteAndQuery.JPG?raw=true)
 ## Create Resource
 We can also add the "[FromBody]" attribute to the action parameter. This tells ASP.NET Core to look for the data in the request body, regardless of the content type. This is useful when we want to send complex data types, such as JSON or XML, in the request body.Here in the below figure ,It is not needed for the PointOfInterestForCreationDto object in the parameter.It gets serialize automatically .
 ![Alt text](Asserts/createResource1.JPG?raw=true)
 <br>The "CreateAtRoute" is used to define the location in the response body.So that we comes to know the endpoints to access the PointOfInterest of the newly created one .<br>
  ![Alt text](Asserts/createResource2.JPG?raw=true)
## Validating Inputs
 ### DataAnnotation
 DataAnnotation are used to validate the input body. They are defined in System.ComponentModel.DataAnnotations.
 ![Alt text](Asserts/dataAnnotation.JPG?raw=true)
 ### ModelState
 ModelState is a dictionary containing both the state of the model, that's our PointOfInterestForCreation detail and model‑binding validation. It represents a collection of name‑value pairs that were submitted to our API, one for each property. It also contains a collection of error messages for each value submitted. Whenever a request comes in, the rules we just apply to our model are checked automatically. If one of them doesn't check out, the ModelStates.IsValid property will be false. This property will also be false if an invalid value for a property type is passed in,
 ![Alt text](Asserts/modelState.JPG?raw=true)
 ### *** Note ***
 The nice thing is that this, too, is not necessary, again, thanks to the API controller attribute. Annotations are automatically checked during model binding and affect the ModelSate dictionary, and the API controller attribute ensures that in case of an invalid ModelState, a 400 Bad Request is returned with the validation annotations returned in the response body.

### Patch
 The HTTP PATCH method is used to apply partial modifications to a resource. It is defined in RFC 5789. The PATCH method applies partial modifications to a resource. Unlike PUT, which replaces the entire resource, PATCH only updates the specified fields.
 * Install the neccessary nuget packages for the patch operation.They are 
 ```bash
 Microsoft.AspNetCore.JsonPatch
 Microsoft.AspNetCore.Mvc.NewtonsoftJson
 ```
 * Add the following code in the Program.cs file
 ```bash
 builder.Services.AddControllers()
    .AddNewtonsoftJson();
  ```
 ![Alt text](Asserts/patch1.JPG?raw=true)
* Add the following code in the controller
![Alt text](Asserts/patch2.JPG?raw=true)
    ![Alt text](Asserts/patch3.JPG?raw=true)
```bash
    [HttpPatch("{id}")]
    public IActionResult PartiallyUpdatePointOfInterest(
        int id,
        [FromBody] JsonPatchDocument<PointOfInterestForUpdateDto> patchDoc)
    {
     
        if (!TryValidateModel(pointOfInterestEntity))
        {
            return ValidationProblem(ModelState);
        }
    }
   ```
   
   In the above code the "TryValidateModel" method is used to validate the model. If the model is not valid, it returns a 400 Bad Request with the validation errors.
    ![Alt text](Asserts/patch4.JPG?raw=true)
   For example we send patch request to remove the Id .So those things are updated .To avoid those operations ,we are using this method to validate the model after applying the patch request.

  


### Create File
   ![Alt text](Asserts/createFile1.JPG?raw=true)
# Working With Services and Dependency Injection
Compiler Directives























