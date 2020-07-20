---
layout: post
modified: '2015-06-07 16:40 +0530'
published: false
comments: true
title: The type or namespace name 'Data' does not exist in the namespace
description: >-
  In this post we will see how to fix the exceptions like this which hapens even
  after adding proper dll(s). Here in this post we will see how to fix
  exception, "The type or namespace name Data does not exist in the namespace
  Microsoft.Practices.EnterpriseLibrary (are you missing an assembly
  reference?)"
tags:
  - 'C#'
  - Entity Framework
categories:
  - .Net
date: '2015-06-07 16:40 +0530'
---
### Introduction
In this post we will see how to fix the exceptions like this which hapens even after adding proper dll(s). Here in this post we will see how to fix exception, "The type or namespace name Data does not exist in the namespace Microsoft.Practices.EnterpriseLibrary (are you missing an assembly reference?)"

### Background
I was working on a project when I encountered one exception which was The type or namespace name 'Data' does not exist in the namespace 'Microsoft.Practices.EnterpriseLibrary' (are you missing an assembly reference?)

I added the proper dll reference but still I was getting this compilation error. I was wondering what is going wrong. I tried to browse the class inside this dll using object browser. I was able to find classes inside this dll. I tried to create an object of the class which was inside the dll; I was able to create the object while I was writing the code. When i build it, it again failed.

- Added Proper dll reference but compilation fails.
- Exception the type or namespace name 'Data' does not exist in the namespace even after adding proper dll references.

### How did I fix this exception
Finally when I check the project configuration, I was running .Net Framework 4.0 Client Profile. I made it to .Net Framework 4.0 and everything worked perfectly.

The .NET Framework 4 Client Profile is a subset of the .NET Framework 4 which is optimized for client applications. It provides functionality for most client applications, including Windows Presentation Foundation (WPF), Windows Forms, Windows Communication Foundation (WCF), and ClickOnce features. This enables faster deployment and a smaller install package for applications that target the .NET Framework 4 Client Profile. If you are targeting the .NET Framework 4 Client Profile, you cannot reference an assembly that is not in the .NET Framework 4 Client Profile. Instead you must target the .NET Framework 4. This is what happening in my case. :)

To change the Target Framework, Go to your project in the solution and right click on the project and select properties. In the window that comes up, select the Application tab. There you can see the Target Framework section. Select the .Net Framework 4 instead of .Net Framework Client Profile. :)
