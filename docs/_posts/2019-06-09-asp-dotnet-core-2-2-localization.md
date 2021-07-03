---
layout: post
title:  "How to Add Localization to ASP.NET Core 2.2"
date:   2019-06-09 17:09:39 -0500
categories: .net
author: Derek Arends
---
 
With the need for localization becoming more and more popular, I thought I would create a little how-to article for localizing ASP.NET Core 2.2 applications where the localized resx files are in a separate project.

I am going to make the assumptions you already know how to create/build/run a .NET Core application and the full code sample is available on GitHub - [https://github.com/derekarends/dotnetcore-localization]

For project setup we will want a new ASP.NET Core Web Application called DotNetLocalization for the solution name and API for the project name.  Once the solution is created we will then add a new .NET Core Class Library targeting the .netstandard2.0 framework project called Core.  Your project should look similar to:

<img src="{{site.url}}/assets/DotNetLocalizationProjectStructure.png" alt="Solution Structure" width="388" height="88"/>

## Core Project Setup

We will want to add a few files to the Core project.  We can start by creating a directory called Resources.  Once the folder is created we can created a class file named SharedResources.cs file in it.  This SharedResources class is just a dummy classed used by .NET for referencing purposes for simply locating the resource files.

We will also want to create two resx files called SharedResources.en-US.resx and SharedResources.fr-FR.resx in the Resources directory and we can add the follow data just above the </root> tag in the resx files.

### SharedResources.en-US.resx

{% highlight ruby %}
<data name="value1" xml:space="preserve">
    <value>Value 1-en</value>
</data>
<data name="value2" xml:space="preserve">
    <value>Value 2-en</value>
</data>
{% endhighlight %}

### SharedResources.fr-FR.resx

{% highlight ruby %}
<data name="value1" xml:space="preserve">
    <value>Value 1-fr</value>
</data>
<data name="value2" xml:space="preserve">
    <value>Value 2-fr</value>
</data>
{% endhighlight %}

Once those changes have been made to the Core project it should look like the image below and we can focus on hooking it all together.

<img src="{{site.url}}/assets/DotNetLocalizationCoreStructure.png" alt="Core Project Structure" width="368" height="148"/>

## Hooking Up the API

The first thing we will do is add a reference to the Core project from the API web application project.

Next we will add localization to the service collection by updating Startup.cs > public void ConfigureServices(IServiceCollection services) method and adding the code above the services.AddMvc() call.

{% highlight ruby %}
services.AddLocalization();
services.Configure<RequestLocalizationOptions>(o =>
{
    var supportedCultures = new[]
    {
        new CultureInfo("en-US"),
        new CultureInfo("fr-FR")
    };

    // State what the default culture for your application is. This will be used if no specific culture
    // can be determined for a given request.
    o.DefaultRequestCulture = new RequestCulture("en-US", "en-US");
    o.SupportedCultures = supportedCultures;
    o.SupportedUICultures = supportedCultures;

    // - QueryStringRequestCultureProvider, sets culture via "culture" and "ui-culture" query string values, useful for testing
    // - CookieRequestCultureProvider, sets culture via "ASPNET_CULTURE" cookie
    // - AcceptLanguageHeaderRequestCultureProvider, sets culture via the "Accept-Language" request header
    o.RequestCultureProviders.Insert(0, new QueryStringRequestCultureProvider());
});
{% endhighlight %}

A little explanation to what is going on here. We are first telling the Service collection to AddLocalization to the dependency tree. We are then configuring the options available to the localization.

* We build an array of supported cultures
* o.DefaultRequestCulture - Defines what the default and fall back cultures are
* o.SupportedCultures - Define what cultures are supported by the application
* o.SupportedUICultures - Determines what cultures should be used when looking for resx
* o.RequestCultureProviders - Is used mainly for testing but can define which order you expect to check for cultures.  Here we are using the QueryStringRequestCultureProvider to easily test the application by changing the culture in the URL

We may also want to add data annotation localization to our project.  This will allow any property that has an annotation that may be displayed to get localized as well.  An example could look like the following:

{% highlight ruby %}
[Required(ErrorMessage="Name is required")]
public string Name {get;set;}
{% endhighlight %}

Now if Name is not provided it will try to use localization for the correct text of "Name is required".  To do this we just need to add the following to the services.AddMvc() in Startup.cs > public void ConfigureServices(IServiceCollection services) method.

{% highlight ruby %}
services.AddMvc()
  .AddDataAnnotationsLocalization(options =>
  {
      options.DataAnnotationLocalizerProvider = (type, factory) =>
          factory.Create(typeof(SharedResources));
  })
  .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
{% endhighlight %}

Last thing we have to do in the Startup.cs file is tell the app to use localization on the request.  This is accomplished by adding the following above the app.UseMvc() call in Startup.cs > public void Configure(IApplicationBuilder app, IHostingEnvironment env) method.

{% highlight ruby %}
var locOptions = app.ApplicationServices.GetService<IOptions>RequestLocalizationOptions>>();
      app.UseRequestLocalization(locOptions.Value);
{% endhighlight %}

## Using the Localizer

Almost done! Now it is time to start using the localizer.  .NET Core 2.2 has a pretty cool localizing concept where it uses a dictionary like syntax so you do not need to have the resx files immediately available just to start using the localizer.  To use the localizer we will want to inject it into our ValuesController.cs constructor and assign it to a private variable.  Now we are able to reference localized values based on the name in the resx.

{% highlight ruby %}
private readonly IStringLocalizer<SharedResources> _localizer;

public ValuesController(IStringLocalizer&lt;SharedResources> localizer)
{
    _localizer = localizer;
}

// GET api/values
[HttpGet]
public ActionResult<IEnumerable<string>> Get()
{
    return new string[] {_localizer["value1"], _localizer["value2"]};
}
{% endhighlight %}

These names can contain spaces so we could have done _localizer["value 1"] and it will work if the name in the resx file was <data name="value 1">.

Finally, to test the application you can ignore the culture and it will default to en-US as that is what we defined or you can pass the cultures in via the URL by going to:

* https://localhost:5001/api/values?culture=en-US
* https://localhost:5001/api/values?culture=fr-FR

[https://github.com/derekarends/dotnetcore-localization]: https://github.com/derekarends/dotnetcore-localization
[https://localhost:5001/api/values?culture=en-US]: https://localhost:5001/api/values?culture=en-US
[https://localhost:5001/api/values?culture=fr-FR]: https://localhost:5001/api/values?culture=fr-FR
