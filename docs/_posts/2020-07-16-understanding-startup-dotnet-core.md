---
layout: post
title: "Understanding Startup.CS in .Net Core"
date: 2020-07-16 17:09:39 -0500
categories: .net startup.cs
author: Derek Arends
---
One file that has always gotten to me in a .Net Core project is the Startup.CS file.  It seems like a simple class but when it came to knowing which method to update I would get confused...  Is it the ConigureServices method or Configure method.

I have decided to help demystify the startup file and explain what it does, how the methods are used, and which method to update when.

## In The Beginning

A long long time ago there was something called OWIN which introduced me to the Startup.CS file.  It was simple, it had one method called Configuration and it took in an IAppBuilder that allowed you to configure your application by injecting middleware into the OWIN pipeline.

With the introduction to .Net Core dependency injection configuration was brought into the startup file, introduction a new method(ConfigureServices) to help distinguish between application configuration and dependency configuration.

## The Configure Method

The configure method is still used to register middleware for configuring the application.  This is where you can register any code that needs to be ran as part of the application pipeline(request/response actions).  Examples of things you may want the pipeline to do:

* Serve Static Files (app.UseStaticFiles())
* Enabling CORS (app.UseCors())
* Setting up routing rules (app.UseRouting())
* Injecting authentication (app.UseAuthentication())
* Inject authorization (app.UseAuthorization())

The list above is all middleware that will now run as part of the request/response pipeline for the application.  This helps abstract how the application will handle each request/response.

## The ConfigureServices Method

The ConfigureServices method is used for configuring dependency injection for the application.  This is where you tell the app how to create and configure instances of dependencies used in other classes. When registering the dependency there are three types to consider:

* Transient - The dependency is created each time it is needed throughout the request
* Scoped - The dependency is created once per client request
* Singleton - The dependency is created ones per the lifetime of the application

Each type will be used to determine the lifetime of dependency being created for the application.  Example services you may register:

* Adding MVC - services.AddMVC()
* Add and configure Entity Framework - services.AddDbContext<ApplicationDbContext>(options =>  options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

## Wrapping Up

The overall file is still pretty simple and it provides a much easier way to configure dependencies for .Net projects now.  Let me know what you think or if you would like to have a deeper dive into how the middleware/dependency injection are used behind the scenes.
