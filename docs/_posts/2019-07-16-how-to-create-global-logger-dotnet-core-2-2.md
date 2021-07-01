---
layout: post
title:  "How to Create a Global Logger for ASP.NET Core 2.2"
date:   2019-07-16 17:09:39 -0500
categories: .net logging
author: Derek Arends
---

Catching all errors is important, not just for helping developers troubleshoot their application but also to help provider a better user experience when handled correctly.  I want to help make software a better experience for everyone so here is how I implemented a global logger for ASP.NET Core 2.2

Full Code Example available on Github - [https://github.com/derekarends/dotnetcore-globallogger]</a>

One of my favorite libraries for logging is called Serilog - [https://serilog.net/] - It provides a TON of different types of what they call "sinks" or ways of logging.  This will be the one used in this tutorial but the basic concepts are the same.

## First you will want to install a few Nuget Packages

* Serilog.AspNetCore - Main package for Serilogging in ASP.NET Core
* Serilog.Settings.Configuration - Allows us to configure Serilog via appsettings.json files
* Serilog.Sinks.Async - Allows Serilog to write to file asynchronously
* Serilog.Sinks.RollingFile - Will have Serilog write to a rolling file
* Serilog.Sinks.Console - Will have Serilog write to console
* For a list of all sinks visit - [https://github.com/serilog/serilog/wiki/Provided-Sinks]

To Configure Serilog we will want to add the following to our appsetting.{environment}.json files.

{% highlight ruby %}
"Serilog": {
    "Using": [
      "Serilog.Sinks.Async",
      "Serilog.Sinks.RollingFile",
      "Serilog.Sinks.Console"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "RollingFile",
              "Args": {
                "pathFormat": "logs/log-{Date}.log"
              }
            }
          ]
        }
      },
      {
        "Name": "Console",
        "Args": {
          "theme": "Serilog.Sinks.SystemConsole.Themes.AnsiConsoleTheme::Code, Serilog.Sinks.Console",
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj} &lt;s:{SourceContext}>{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": [
      "FromLogContext",
      "WithMachineName",
      "WithThreadId"
    ],
    "Properties": {
      "Application": "DotNetCoreGlobalLogger"
    }
  }
{% endhighlight %}

## A little explaining

* "Using" - This describes which sinks we want enabled for our logging.
* "MinimumLevel" - This sets the log level for Serilog.  We are telling it to override the Microsoft default and use the settings we described for Serilog.
* "WriteTo" - Takes an array of sinks that we will be and their configurations for where we will be writing our logs.
* "Name" -  Defines which Sinks we will be using. Here we will be using "Async" and "Console"
* "Args" - Allows us to pass in additional arguments for the sinks to use
* "Enrich" - Gives us the ability to specify some additional details when logging
* "Properties" - Allows use to give basic properties of the logger.
* For a complete list of configuration properties visit - [https://github.com/serilog/serilog/wiki/Configuration-Basics]

## Configuring Program.cs to use global logging

We will want to add a new property to at the top of Program.cs to get the Configuration based on environment.

{% highlight ruby %}
private static IConfiguration Configuration
{
  get
  {
    var env = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
    var builder = new ConfigurationBuilder();
    builder.AddJsonFile($"appsettings.{env}.json", false, true);

    builder.AddEnvironmentVariables();
    return builder.Build();
  }
}
{% endhighlight %}

Next we will want to override the main method with the follow cold to to setup Serilog and tell the application to use it for the logging.

{% highlight ruby %}
public static void Main(string[] args)
{
  Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(Configuration)
    .CreateLogger();

  try
  {
    Log.Information("Starting web host");
    CreateWebHostBuilder(args)
      .UseSerilog() // Important!
      .Build()
      .Run();
  }
  catch (Exception ex)
  {
    Log.Fatal(ex, "Host terminated unexpectedly");
  }
  finally
  {
    Log.CloseAndFlush();
  }
}
{% endhighlight %}

## A little middleware never hurts

Now we will want to add a little middleware so if something unexpected fails in any of our requests it will be logged and a response of our choosing can be returned.

For simple organization we will create a folder called Middleware and then create a class called ErrorResult.cs.  This class will allow us to log and return a typed object from our error.  We also override the ToString method so it returns a serialized version of this object.

{% highlight ruby %}
public class ErrorResult
{
  public int StatusCode { get; set; }
  public string Message { get; set; }
  public override string ToString()
  {
    return JsonConvert.SerializeObject(this);
  }
}
{% endhighlight %}

Next we will create a static class called ExceptionMiddlewareExtensions.cs in the Middleware folder with the following method.

{% highlight ruby %}
public static class ExceptionMiddlewareExtensions
{
  public static void ConfigureExceptionHandler(this IApplicationBuilder app, ILoggerFactory logger)
  {
    app.UseExceptionHandler(appError =>
    {
      appError.Run(async context =>
      {
        context.Response.StatusCode = (int) HttpStatusCode.InternalServerError;
        context.Response.ContentType = "application/json";

        var contextFeature = context.Features.Get&lt;IExceptionHandlerFeature>();
        if (contextFeature != null)
        {
          logger.CreateLogger("GlobalException")
            .LogError($"Something went wrong: {contextFeature.Error}");

          await context.Response.WriteAsync(new ErrorResult
          {
            StatusCode = context.Response.StatusCode,
            Message = "Internal Server Error."
          }.ToString());
        }
      });
    });
  }
}
{% endhighlight %}

This method tells the application to use the exception handler middleware and we created a simple lamda expression to set the context response to whatever we would like when an exception is caught.  I have seen this return 200's with a message or my preference to return 500 with message and let the consumer of the API know something went unexpectedly and they should handle it accordingly.

## Last Step

Finally, after all that setup the last thing we will want to do is have the ConfigureExceptionHandler method we just created to be called in the Startup.cs > Configure method.

{% highlight ruby %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
  app.ConfigureExceptionHandler(_loggerFactory);
  app.UseMvc();
}
{% endhighlight %}

## Testing

Now if you would like to test this out simply have one of your API endpoints throw an exception and see the exception getting logged to the console and written to the directory described in the appsettings.{environment}.json > "pathFormat": "logs/log-{Date}.log" field.

Thanks for reading and let me know how you have used global logging to help track down unexpected exceptions!

[https://github.com/derekarends/dotnetcore-globallogger]: https://github.com/derekarends/dotnetcore-globallogger
[https://serilog.net/]: https://serilog.net/
[https://github.com/serilog/serilog/wiki/Provided-Sinks]: https://github.com/serilog/serilog/wiki/Provided-Sinks
[https://github.com/serilog/serilog/wiki/Configuration-Basics]: https://github.com/serilog/serilog/wiki/Configuration-Basics
