---
layout: post
title:  "How to Setup ASP.NET Core 2.2 with a Multiple Build Dockerfile"
date:   2019-09-11 17:09:39 -0500
categories: docker
author: Derek Arends
---

Now that Docker is becoming a must have when creating small services and needing them to be resilient and easy to deploy, I wanted to create a simple how-to for using ASP.NET Core 2.2 and a Dockerfile with multiple build steps.

This tutorial assumes you know how to create an ASP.NET Core 2.2 project but for context on how the Dockerfile is written, here is my project structure.

{% highlight ruby %}
Project
- src
-- Api
--- Api.csproj
-- Core
--- Core.csproj
- Dockerfile
{% endhighlight %}

## Dockerfile

{% highlight ruby %}
FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src

COPY . .
RUN dotnet restore
RUN dotnet build -c Release --no-restore

FROM build AS publish
RUN dotnet publish /src/Api/Api.csproj -c Release -o /app --no-restore

FROM microsoft/dotnet:2.2-aspnetcore-runtime
WORKDIR /src
COPY --from=publish /app .
ENTRYPOINT [ "dotnet", "Api.dll"]
{% endhighlight %}

## Walkthrough

The Dockerfile is short and sweet but there is a lot of things going on and a few gotchas that you may want to clean up before using it as a production Dockerfile.

## Building

{% highlight ruby %}
FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src
{% endhighlight %}

This chunk from the file tells Docker to use image microsoft/dotnet:2.2-sdk  as the build image.  We use the "AS build" to reference this point later on in the Docker build process.

WORKDIR is telling the Docker build where to work out because of my project structure I need to start in src to be able to access all my projects.

{% highlight ruby %}

COPY . .
{% endhighlight %}

This tells Docker to copy ALL files from the working directory to a directory in the Docker build pipeline called src

{% highlight ruby %}
RUN dotnet restore
RUN dotnet build -c Release --no-restore
{% endhighlight %}

These RUN commands are telling Docker to execute  a dotnet restore to pull in all dependencies and restore any packages needed for the application.  It is then building the application in Release mode.

## Publishing

{% highlight ruby %}
FROM build AS publish
RUN dotnet publish /src/Api/Api.csproj -c Release -o /app --no-restore
{% endhighlight %}

Here we are telling Docker to "start" from the build pipeline and allow it to be referenced later as the publish step.  We are then telling it to run the dotnet publish command putting the compiled binary(Api.dll) in the /app directory publish pipelines "app" directory.

## Runtime Image

{% highlight ruby %}
FROM microsoft/dotnet:2.2-aspnetcore-runtime
WORKDIR /src
COPY --from=publish /app .
ENTRYPOINT [ "dotnet", "Api.dll"]
{% endhighlight %}

This snippet is doing pretty much the same thing as the build step with a few minor changes.  One being the use of the runtime image instead of the SDK.  This allows us to reduce the overall image size by not including all the things needed to compile and build the DLL.

The other difference is ENTRYPOINT.  This tells Docker how to start a container running your image.

## Improvements

I think this file could be improve by limiting the files being copied in the build step.  Right now we are copying all files which means if your working directory has appsettings.production.json or appsettings.staging.json those will both be copied over as well as anything else environmental specific.

One way to change this would be to pass in Docker parameters and use those to specify what should be copied or better yet might be to avoid having those files copied in the image and allow them to be put in a Docker volume that your application is running on.

## Thoughts

I liking having a multi-step process as it allows for smaller images and prevents having bloat where we don't need it.  It does take a bit of fidgeting with it depending on project structure and how to get all the projects/packages to be imported correctly but once done it makes building and testing the application much easier!
