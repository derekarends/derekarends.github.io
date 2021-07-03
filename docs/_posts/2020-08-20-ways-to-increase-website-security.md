---
layout: post
title: "Simple Ways to Greatly Increase Website Security"
date: 2019-08-09 17:09:39 -0500
categories: security
author: Derek Arends
---

Adding just a few extra headers to your website's response can help improve the overall security of your website.  By doing this it will help prevent clickjacking, MIME based, and cross-site scripting attacks.

## Clickjacking

Clickjacking occurs when an attacker is able to load a transparent page over  top your website.  This allows an unsuspecting user to think they are clicking items and actions on your website but are really interacting with the invisible page.  This would allow malicious actions to occur that the user never intended.  To help prevent this we can add the following header.

{% highlight ruby %}
# if you need to allow [i]frames, you can use SAMEORIGIN 
# or even set a uri with ALLOW-FROM uri
add_header X-Frame-Options SAMEORIGIN;
{% endhighlight %}

For additional X-Frame-Options visit - [https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options]

## MIME based attacks

MIME based attacks occur when the browser attempts to "sniff" a response or override the response content type based on what it thinks the response should be.  This can open a door for hackers to put malicious content types into your system and cause unintended side affects when your application goes to render what you thought was HTML but the browser sniffs the response and guesses it to be javascript to executed.  To help prevent this on most browsers add the following heading to your response:

{% highlight ruby %}
add_header X-Content-Type-Options nosniff;
{% endhighlight %}

This is may not be supported by all browsers but it is one more line of defense to help your application become more secure.

## Cross-Site Scripting

This vulnerability allows scripts to be ran on your website that you did not write yourself.  This usually occurs when you allow generic user input and do not sanitize what the input is.  There has been a lot of improvements in this area if you were to use a javascript framework like Angular or React.  However, there may still be ways around it and one more level of security shouldn't hurt.  

Most browsers will have this next header turned on by default, however, we will make sure it is turned on when accessing our application in a scenario where the browser had it turned off.

{% highlight ruby %}
 add_header X-XSS-Protection "1; mode=block";
{% endhighlight %}

There are other options for protection and different levels of protection. If you wanted to adjust according to what you would like your application to do.

* **0** - Filter is Disabled
* **1** - Filter enabled. If a cross-site scripting attack is detected, in order to stop the attack, the browser will sanitize the page.
* **1; mode=block** - Filter enabled. Rather than sanitize the page, when a XSS attack is detected, the browser will prevent rendering of the page.
* **1; report=YOUR_REPORT_URI** - Filter enabled. The browser will sanitize the page and report the violation. This is a Chromium function utilizing CSP violation reports to send details to a URI of your choice.

## Content Security Policy(CSP)

CSP is an additional layer of security to help detect and mitigate cross-site scripting and data injection attacks.  CSP is designed to be backwards compatible but if a browser does not respect this header it will simply ignore it and render as normal.  

The thing I like about CSP is it seems to allow you as the owner of the website to be very specific with which scripts, urls, and files your website is expected to work on or with.  

One thing I would recommend is to first use 'Content-Security-Policy-Report-Only' as this will allow your website to continue to run without blocking scripts and you can see which policies are failing.  Once you are confident then flip to 'Content-Security-Policy'

The following would be an example of how to ratchet down your website so it only allows scripts from your website, google apis, google analytics, and stripe.

{% highlight ruby %}
add_header Content-Security-Policy "default-src 'self'; script-src 'self' *.googleapis.com *.google-analytics.com; img-src 'self'; font-src 'self' fonts.gstatic.com data:; style-src 'self' *.googleapis.com; frame-src 'self' js.stripe.com; object-src 'self'; report-uri SOME_REPORTING_URI;

{% endhighlight %}

## CSP Options

Here is a list of possible options to provide to Content-Security-Policy and the impacts they would have.

* **default-src** - Define loading policy for all resources type in case of a resource type dedicated directive is not defined (fallback)
* **script-src** - Define which scripts the protected resource can execute,
* **object-src** - Define from where the protected resource can load plugins
* **style-src** - Define which styles (CSS) the user applies to the protected resource
* **img-src** - Define from where the protected resource can load images
* **media-src** - Define from where the protected resource can load video and audio
* **frame-src** - Define from where the protected resource can embed frames
* **font-src** - Define from where the protected resource can load fonts
* **connect-src** - Define which URIs the protected resource can load using script interfaces
* **form-action** - Define which URIs can be used as the action of HTML form elements
* **sandbox** - Specifies an HTML sandbox policy that the user agent applies to the protected resource
* **script-nonce** - Define script execution by requiring the presence of the specified nonce on script elements
* **plugin-types** - Define the set of plugins that can be invoked by the protected resource by limiting the types of resources that can be embedded
* **reflected-xss** - Instructs a user agent to activate or deactivate any heuristics used to filter or block reflected cross-site scripting attacks, equivalent to the effects of the non-standard X-XSS-Protection header
* **report-uri** - Specifies a URI to which the user agent sends reports about policy violation

## Parting Words

Hopefully this will help increase the security of your website.  I would like to hear from you on what things you are doing to secure your website and if there are better/more ways of improving website security.

[https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options]: https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options
