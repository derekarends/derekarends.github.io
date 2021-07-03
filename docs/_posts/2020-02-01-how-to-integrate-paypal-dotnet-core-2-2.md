---
layout: post
title: "How to Integrate with PayPal ASP.NET Core 2.2"
date: 2020-02-01 17:09:39 -0500
categories: .net
author: Derek Arends
---

Paypal is a great way to allow payments from a broad audience of customers.

It is used around the world and trusted as a safe way to send and receive payments.

Here is a little snippet of what it looks like to integrate Paypal payments into a .Net Core application.

{% highlight ruby %}
public async Task<string> InitPaypal()
{
  PayPalEnvironment environment = null;
  if (_settings.Environment.Equals("Production", StringComparison.OrdinalIgnoreCase))
  {
     environment = new LiveEnvironment(_settings.PaypalClientId, _settings.PaypalSecret);
  }
  else
  {
     environment = new SandboxEnvironment(_settings.PaypalClientId, _settings.PaypalSecret);
  }

  if (environment == null)
    return string.Empty;

  var items = new PaypalPayment.ItemList();
  items.Items = new List<PaypalPayment.Item>
  {
     new PaypalPayment.Item
     {
        Name = "Your product name",
        Currency = "USD",
        Price = "price_in_dollars",
        Quantity = "1",
        Description = "This is my product"
     }
  };

  var paypalPayment = new PaypalPayment.Payment
  {
     Intent = "sale",
     Transactions = new List<PaypalPayment.Transaction>
     {
        new PaypalPayment.Transaction
        {
           Amount = new PaypalPayment.Amount
           {
              Total = "price_in_dollars",
              Currency = "USD"
           },
           Description = "Purchase of my product",
           ItemList = items
        }
     },
     RedirectUrls = new PaypalPayment.RedirectUrls
     {
        CancelUrl = _settings.PaypalCancelUrl,
        ReturnUrl = _settings.PaypalReturnUrl
     },
     Payer = new PaypalPayment.Payer
     {
        PaymentMethod = "paypal"
     }
  };

  var request = new PaypalPayment.PaymentCreateRequest();
  request.RequestBody(paypalPayment);

  try
  {
     var client = new PayPalHttpClient(environment);
     HttpResponse response = await client.Execute(request);
     var statusCode = response.StatusCode;
     var result = response.Result<PaypalPayment.Payment>();
     return result.Id;
  }
  catch (HttpException httpException)
  {
     var debugId = httpException.Headers.GetValues("PayPal-Debug-Id").FirstOrDefault();
     return string.Empty;
  }
}
{% endhighlight %}

{% highlight ruby %}
public async Task<bool> PaypalPayment(string payerId, string paymentId)
{
   PayPalEnvironment environment = null;
   if (_settings.Environment.Equals("Production", StringComparison.OrdinalIgnoreCase))
   {
      environment = new LiveEnvironment(_settings.PaypalClientId, _settings.PaypalSecret);
   }
   else
   {
      environment = new SandboxEnvironment(_settings.PaypalClientId, _settings.PaypalSecret);
   }

   if (environment == null)
     return string.Empty;

   var executionBody = new PaypalPayment.PaymentExecution()
   {
      PayerId = payerId
   };

   var request = new PaypalPayment.PaymentExecuteRequest(paymentId);
   request.RequestBody(executionBody);
   try
   {
      HttpResponse response = await client.Execute(request);
      if (response.StatusCode == System.Net.HttpStatusCode.OK)
        return true;
      else
        return false;
   }
   catch (HttpException httpException)
   {
      var debugId = httpException.Headers.GetValues("PayPal-Debug-Id").FirstOrDefault();
      return false;
   }
}
{% endhighlight %}

There isn't a lot that goes into being able to integrate with Paypal, I think the trickiest part was making sure to hit the test system when not in a production mode.
