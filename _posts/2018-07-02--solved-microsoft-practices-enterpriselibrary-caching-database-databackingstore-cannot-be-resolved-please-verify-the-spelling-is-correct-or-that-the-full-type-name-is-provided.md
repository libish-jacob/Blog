---
layout: post
modified: '2014-10-23 11:54 +0530'
published: true
comments: true
title: >-
  [Solved]
  Microsoft.Practices.EnterpriseLibrary.Caching.Database.DataBackingStore cannot
  be resolved. Please verify the spelling is correct or that the full type name
  is provided
tags:
  - Enterprise Library
categories:
  - .Net
description: >-
  I was trying to use SQlite Backing Store for my Enterprise Library Caching
  block. When i try to run my application I started getting an exception an it
  was like this
date: '2014-10-23 11:54 +0530'
---
## Introduction
   I was trying to use SQlite Backing Store for my Enterprise Library Caching block. When i try to run my application I started getting an exception an it was like this

System.ArgumentException was unhandled

Message=The type _'Microsoft.Practices.EnterpriseLibrary.Caching.Database.DataBackingStore, Microsoft.Practices.EnterpriseLibrary.Caching.Database, Version=5.0.414.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' cannot be resolved. Please verify the spelling is correct or that the full type name is provided._

## How did i fix this.
  I was using database backing store and backing store required one reference to Microsoft.Practices.EnterpriseLibrary.Caching.Database. After adding a reference to the same, It got fixed.
