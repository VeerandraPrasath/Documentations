# ASP .NET CORE

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








