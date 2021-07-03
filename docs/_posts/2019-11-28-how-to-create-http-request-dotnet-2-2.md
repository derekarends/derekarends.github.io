---
layout: post
title:  "How to Create an HTTP Request in ASP.NET Core 2.2"
date:   2019-11-28 17:09:39 -0500
categories: .net http request
author: Derek Arends
---

Sometimes your application may need to talk to an external application.  One way to accomplish this is through an HTTP request.  I will be walking you through how to create an HTTP POST and GET request in ASP.NET Core 2.2.

Sample code is available on github - [https://github.com/derekarends/dotnetcore-httprequest]

## HTTP GET Request

The first thing we will want to do is create a single instance of the HttpClient.  This is to prevent socket exhaustion.  To learn more about it - [https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-2.2#remarks]

{% highlight ruby %}
private static readonly HttpClient HttpClient = new HttpClient();
{% endhighlight %}

The first request we will create is probably one of the simpler ones and that is an HTTP GET request.  

For demo purposes I created another API controller called ExternalEndpointController that our ValuesController will make it's HTTP requests to but the sample code will give you an idea of what a GET request to an external API may look like.

{% highlight ruby %}
// GET api/values/getFromExternal
[HttpGet, Route("getFromExternal")]
public async Task&lt;IActionResult> GetFromExternal()
{
  using(var response = await HttpClient.GetAsync("https://localhost:5001/api/externalEndpoint"))
  {
    if (!response.IsSuccessStatusCode)
      return StatusCode((int) response.StatusCode);

    var responseContent = await response.Content.ReadAsStringAsync();
    var deserializedResponse = JsonConvert.DeserializeObject&lt;List&lt;string>>(responseContent);

    return Ok(deserializedResponse);
  }
}
{% endhighlight %}

Overall pretty straight forward, there are some cool things you can do with this new HttpClient.  Instead of response.Content.ReadAsStringAsync we could have had it read as a stream or bytes to help efficiency if we knew we were going to get back large amounts of data.

Also, using JsonConvert made it easy to take the result from the external endpoint and covert it into something useful for the consuming application.

## HTTP POST Request

POST requests are a little more complex as there are things you may have to do to make them work that you didn't have to do for the GET.  In the sample code below we again are calling the external endpoint controller but this time we are going to be posting some JSON data to it.

{% highlight ruby %}
// GET api/values/postToExternal
[HttpGet, Route("postToExternal")]
public async Task&lt;HttpStatusCode> PostToExternal()
{
  var postData = new
  {
    firstName = "Derek",
    lastName = "Arends"
  };

  var serializedRequest = JsonConvert.SerializeObject(postData);

  var requestBody = new  StringContent(serializedRequest);
  requestBody.Headers.ContentType = new MediaTypeHeaderValue("application/json");

  using(var response = await HttpClient.PostAsync("https://localhost:5001/api/externalEndpoint", requestBody))
  {
    return response.StatusCode;
  }
}
{% endhighlight %}

Details to what the POST is doing:

* We have to serialized the data to a JSON string format before assigning it as a StringContent.
* StringContent is used to tell the request what type of format the body is in.  Other options may be StreamContent or MutlipartContent depending on what you are posting to the endpoint.
* We set the request header to "application/json" to make sure the communication knows this is JSON data.
* Once we POST and get a response back we are able to parse the response like we did with the GET but for this example I simply return the status code to see if the post was successful.

## Other HTTP Types

The ASP.NET Core HttpClient can do other HTTP Request types such as PUT, PATCH, DELETE.  They are all very similar setup to how the POST is done.

[https://github.com/derekarends/dotnetcore-httprequest]: https://github.com/derekarends/dotnetcore-httprequest
[https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-2.2#remarks]: https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-2.2#remarks