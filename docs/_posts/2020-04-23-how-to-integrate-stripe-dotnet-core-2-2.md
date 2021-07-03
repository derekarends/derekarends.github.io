---
layout: post
title: "How to Integrate with Stripe ASP.NET Core 2.2"
date: 2020-04-23 17:09:39 -0500
categories: .net "Stripe Integration"
author: Derek Arends
---
The Stripe API is fabulous at documentation and giving examples of how to do all sorts of things to allow your application to take credit cards.

This example will describe how to create a simple charge using ASP.NET Core 2.2

First we will need to install the Stripe.Net nuget package.

Then we will need to create the StripeChargeCreateOptions model and fill in some of the properties.

{% highlight ruby %}
var chargeOptions = new StripeChargeCreateOptions()
{
  Amount = 200,
  Currency = "usd",
  Description = $"Charge to {customer_email} for {product_name}",
  Capture = true,
  SourceTokenOrExistingSourceId = "tokenId_from_stripe_libraries"
};
{% endhighlight %}

The "tokenId_stripe_libraries" is going to be the id returned from Stripe when using the Stripe libraries and validating your request with your public Stripe API key.

We then just need to create the StripeChargeService and have it create a charge.

{% highlight ruby %}
var chargeService = new StripeChargeService();
try
{
  await chargeService.CreateAsync(chargeOptions);
}
catch (StripeException ex)
{
  Console.WriteLine(ex.StripeError.Message);
}
{% endhighlight %}

The Stripe API has many more features you can incorporate into your application, I just wanted to give you a brief view of what it would look like to use .NET Core to charge Stripe.