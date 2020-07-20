---
layout: post
modified: '2015-03-06 17:18 +0530'
published: true
comments: true
title: >-
  Could not resolve this reference. Could not locate the assembly
  "Microsoft.Office.Interop.Excel, Version=14.0.0.0
description: >-
  You may experience this error if you try to compile a visual studio
  solution/project where it is referring to Microsoft.Office.Interop.Excel from
  GAC. This could also happen in a build server where you don’t have or want any
  tools which are licensed because you don’t want to pay for it as you spawn
  servers at runtime. You may feel like you hit the dead end where you are
  forced to install Microsoft office tools. But we have another alternative
  which is completely legal and ethical.
tags:
  - 'C#'
categories:
  - .Net
date: '2015-03-06 17:18 +0530'
---
### Introduction
You may experience this error if you try to compile a visual studio solution/project where it is referring to Microsoft.Office.Interop.Excel from GAC. This could also happen in a build server where you don’t have or want any tools which are licensed because you don’t want to pay for it as you spawn servers at runtime. You may feel like you hit the dead end where you are forced to install Microsoft office tools. But we have another alternative which is completely legal and ethical.

### How did I fix this?
This is the nuget package. We have this Microsoft.Office.Interop.Excel dll available as a nuget package. This is called Excel-DNA.Interop. This Microsoft.Office.Interop.Excel dll which comes in this nuget package has no dependency and is pretty straightforward to use. Use it just like any other nugget package!
