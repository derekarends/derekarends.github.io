---
layout: post
title:  "What is .Net Core"
date:   2019-05-07 17:09:39 -0500
categories: .net
author: Derek Arends
---

.NET Core is the latest .NET implementation with a twist.

Microsoft has decided to create it as a open source project that allows running .NET on multiple operating systems.  With this new approach Microsoft has opened up a world for .NET developers that was once off limits.

Using .NET Core allows you to build performant, cross platform console and ASP.NET web applications.  This lets developers and businesses start to use other operating systems on their servers, which in todays world is usually hosted in "the cloud" but we all know Linux servers are cheaper than Windows servers so being able to build for Linux already has one advantage.

## .NET Core vs .NET Standard

In this new era of open source .NET, Microsoft created .NET Core and .NET Standard.  Why was this done and is one better than the other?

I wouldn't say one is better than the other they just have different reasons for being.  .NET Core is kind of a super set of APIs on top of the .NET Standard.  All .NET implementations must implement the .NET Standard(Including .NET Core) because it is considered a base class library or BCL.  

So if you want to create libraries to share across all your .NET apps you would target the .NET Standard because it will work on any .NET implement or OS but it will have a subset of APIs available because it has to run on any implementation.

If you know you are going to stay focused on a particular OS or implementation creating a .NET Core library may allow you more  APIs but will limit your options for being included in other types of .NET projects.

Overall I have started to enjoy this new world of .NET Core.  I think there is some huge potential to for .NET developers to create some applications that are no longer restricted to the Windows environment.  
