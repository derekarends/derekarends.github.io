---
layout: post
title: "How to Create Email Templates with ASP.NET Core 2.2"
date: 2020-03-13 17:09:39 -0500
categories: .net "Emailt Templates"
author: Derek Arends
---

<img src="{{site.url}}/assets/EmailTemplate.png" alt="Hello World Email Template" />

Having your application send nice looking emails is becoming a must when notifying your customers of some action or item related to your business.

This tutorial will walk you through how to add the ability for your ASP.NET Core application to send customizable email templates using Razor pages, HTML, and CSS.

The code for this sample is available at - [https://github.com/derekarends/dotnetcore-emailtemplateengine]

First we will want to create a couple projects, one will be the main project that sends emails called Api the other being a project for templates we will simple call Templates.

## Project Setup

To start we will create an ASP.NET Core 2.2 Web Application with a Project Type of  Web API and call it Api.

Next we will want to create another project called Templates by going to a new ASP.NET Core 2.2 Web Application and selecting Razor Class Library in the Project Type.  This project type will allow us to create Razor Pages for our email rendering.

Once the Razor Project is created we will want to add a few directories to it to help organize the code.

* Create directory called Views
* Inside Views create a directory called Shared
* Inside Views create a directory called Emails
* Inside Emails create a directory called HelloWorld
* Create a directory called ViewModels

Lastly for the project setup we will want to add a reference to the Templates project in our Api project.

## View Model Creation

The full code for these files is available on github so feel free to go over there and copy/paste as needed. To keep this short I will only show the import contents of the files.

To get started we will want to create a couple files in the ViewModels directory.

### EmailButtonViewModel.cs

This is a reusable button to display in the email.

{% highlight ruby %}
namespace Templates.ViewModels
{
  public class EmailButtonViewModel
  {
    public string Text { get; set; }
    public string Url { get; set; }

    public EmailButtonViewModel(string text, string url)
    {
      Text = text;
      Url = url;
    }
  }
}
{% endhighlight %}

### HelloWorldViewModel.cs

This is going to contain any dynamic variables you would like to use in the HelloWorld email template.

{% highlight ruby %}
namespace Templates.ViewModels
{
  public class HelloWorldViewModel
  {
    public string ButtonLink { get; set; }

    public HelloWorldViewModel(string buttonLink)
    {
      ButtonLink = buttonLink;
    }
  }
}
{% endhighlight %}

## Email Template Creation

In the Shared directory create the following Razor Pages.

### EmailButton.cshtml

This file shows how we import the view model we would like to use and the properties available on that model.

{% highlight ruby %}
@using Templates.ViewModels
@model EmailButtonViewModel
<table width="100%" border="0" cellspacing="0" cellpadding="0">
  <tr>
    <td bgcolor="#ffffff" align="center" style="padding: 30px;">
      <table border="0" cellspacing="0" cellpadding="0">
        <tr>
          <td align="center" style="border-radius: 30px;" bgcolor="#0088f3">
            <a href="@Model.Url" target="_blank" style="font-size: 20px; font-family: Helvetica, Arial, sans-serif; color: #ffffff; text-decoration: none; padding: 15px 25px; display: inline-block;">
              @Model.Text
            </a>
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
{% endhighlight %}

## Email Layout

We will want to create two different types of layouts one for HTML and one for text.  The content of the files is available on github.

* _EmailLayoutHtml.cshtml
* _EmailLayoutText.cshtml

## Creating the Specific Email Template

Finally we can create the specific email we would like to send in the Views/HelloWorld directory.

### HelloWorldHtml.cshtml

{% highlight ruby %}
@using Templates.ViewModels
@model HelloWorldViewModel

@{
  Layout = "_EmailLayoutHtml";
  ViewContext.ViewData["EmailTitle"] = "Hello World!";
}

<p>
  It looks like you may have just sent your first custom email!
</p>

@await Html.PartialAsync("_EmailButton", new EmailButtonViewModel("Let's Go!", Model.ButtonLink))

<p>
  Derek Arends
</p>
{% endhighlight %}

### HelloWorldText.cshtml

{% highlight ruby %}
@using Templates.ViewModels
@model HelloWorldViewModel

@{
  Layout = "_EmailLayoutText";
  ViewContext.ViewData["EmailTitle"] = "Hello World!";
}

It looks like you may have just sent your first custom email! Let's Go to @Model.ButtonLink.
- Derek Arends
{% endhighlight %}

### RazorViewToStringRenderer.cs

This class is what does a lot of the magic for us.  It will use the Razor engine to find the view we specify and build a rendered version of it to be returned as a string.  This is what glues all these files together to build the email to be sent.

{% highlight ruby %}
namespace Templates
{
  // Code from: https://github.com/aspnet/Entropy/blob/dev/samples/Mvc.RenderViewToString/RazorViewToStringRenderer.cs
  public interface IRazorViewToStringRenderer
  {
    Task<string> RenderViewToStringAsync<TModel>(string viewName, TModel model);
  }

  public class RazorViewToStringRenderer : IRazorViewToStringRenderer
  {
    private readonly IRazorViewEngine _viewEngine;
    private readonly ITempDataProvider _tempDataProvider;
    private readonly IServiceProvider _serviceProvider;

    public RazorViewToStringRenderer(
      IRazorViewEngine viewEngine,
      ITempDataProvider tempDataProvider,
      IServiceProvider serviceProvider)
    {
      _viewEngine = viewEngine;
      _tempDataProvider = tempDataProvider;
      _serviceProvider = serviceProvider;
    }

    public async Task<string> RenderViewToStringAsync<TModel>(string viewName, TModel model)
    {
      var actionContext = GetActionContext();
      var view = FindView(actionContext, viewName);

      using (var output = new StringWriter())
      {
        var viewContext = new ViewContext(
          actionContext,
          view,
          new ViewDataDictionary<TModel>(new EmptyModelMetadataProvider(), new ModelStateDictionary())
          {
            Model = model
          },
          new TempDataDictionary(actionContext.HttpContext, _tempDataProvider),
          output,
          new HtmlHelperOptions());

        await view.RenderAsync(viewContext);

        return output.ToString();
      }
    }

    private IView FindView(ActionContext actionContext, string viewName)
    {
      var getViewResult = _viewEngine.GetView(executingFilePath: null, viewPath: viewName, isMainPage: true);
      if (getViewResult.Success)
      {
        return getViewResult.View;
      }

      var findViewResult = _viewEngine.FindView(actionContext, viewName, isMainPage: true);
      if (findViewResult.Success)
      {
        return findViewResult.View;
      }

      var searchedLocations = getViewResult.SearchedLocations.Concat(findViewResult.SearchedLocations);
      var errorMessage = string.Join(
        Environment.NewLine,
        new[] {$"Unable to find view '{viewName}'. The following locations were searched:"}.Concat(searchedLocations));

      throw new InvalidOperationException(errorMessage);
    }

    private ActionContext GetActionContext()
    {
      var httpContext = new DefaultHttpContext {RequestServices = _serviceProvider};
      return new ActionContext(httpContext, new RouteData(), new ActionDescriptor());
    }
  }
}
{% endhighlight %}

## Sending the Email

The final step to all of this is to send the email off.  To do this we will create a simple endpoint that will call the RazorViewToString class to generate the email content.

In the Api project create a controller called EmailsController.cs and add a simple GET method to send an email. (This is only for testing purposes)

### EmailsController.cs

In the send method you will need to replace some of the variables for this to work correctly.

{% highlight ruby %}
// GET api/emails/send
[HttpGet, Route("send")]
public async Task<IActionResult> Send()
{
  try
  {
    var from = new MailAddress("from@mail.com", "Derek Arends");
    var to = new MailAddress("to@mail.com");

    var model = new HelloWorldViewModel("https://www.google.com");

    const string view = "/Views/Emails/HelloWorld/HelloWorld";
    var htmlBody = await _renderer.RenderViewToStringAsync($"{view}Html.cshtml", model);
    var textBody = await _renderer.RenderViewToStringAsync($"{view}Text.cshtml", model);

    var message = new MailMessage(from, to)
    {
      Subject = "Hello World!",
      Body = textBody
    };

    message.AlternateViews.Add(
      AlternateView.CreateAlternateViewFromString(htmlBody, Encoding.UTF8, MediaTypeNames.Text.Html));

    using (var smtp = new SmtpClient("smtp.mailserver.com", 587))
    {
      smtp.DeliveryMethod = SmtpDeliveryMethod.Network;
      smtp.UseDefaultCredentials = false;
      smtp.EnableSsl = true;
      smtp.Credentials = new NetworkCredential("smtp_user","smtp_password");
      await smtp.SendMailAsync(message);
    }
  }
  catch (Exception e)
  {
    return StatusCode(500, $"Failed to send email: {e.Message}");
  }

  return Ok("Email Sent!");
}
{% endhighlight %}

In the Startup.cs we will need to add the following line to the ConfigureServices method.

{% highlight ruby %}
services.AddTransient<IRazorViewToStringRenderer, RazorViewToStringRenderer>();
{% endhighlight %}

Last step is to build the project, go to [https://localhost:5001/api/emails/send] and see your new custom email being delivered.

## Final Thoughts

This allows for a very flexible email templating engine. I could see how we could turn this into its own solution to allow many services and projects to use this structure.  What are some other ways you guys are building custom email templates for your applications?

[https://github.com/derekarends/dotnetcore-emailtemplateengine]: https://github.com/derekarends/dotnetcore-emailtemplateengine
[https://localhost:5001/api/emails/send]: https://localhost:5001/api/emails/send
