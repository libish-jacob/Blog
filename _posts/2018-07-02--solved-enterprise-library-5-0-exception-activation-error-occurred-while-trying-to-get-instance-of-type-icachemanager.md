---
layout: post
modified: '2014-10-23 11:29 +0530'
published: true
comments: true
title: >-
  [Solved] Enterprise Library 5.0 exception Activation error occurred while
  trying to get instance of type ICacheManager
description: >-
  I was working on a project which uses Enterprise Library to store cache data
  to database backing store and I encountered an exception as Activation error
  occurred while trying to get instance of type ICacheManager.
categories:
  - .Net
date: '2014-10-23 11:29 +0530'
tags:
  - Enterprise Library
---
## Introduction
I was working on a project which uses Enterprise Library to store cache data to database backing store and I encountered an exception as Activation error occurred while trying to get instance of type ICacheManager.

I believed I have the entire configuration done properly. But there was one important configuration missing. In this post we will see how to fix the exception which occurs while using Enterprise library to cache or log data to backing store say database or any backing store. This happens when we try to use database backing store and misses the steps which is required to setup database backing store. Now to have the database to hold the cache data, we need some tables and stored procedures.

## How did I fix this exception
  Enterprise library will provide you with one command batch file which you need to run against your database. This batch file will run Sql commands against your database and will create the stored procedures and required tables. You can find this batch file at, “EntLib50Src\Blocks\Caching\Src\Database\Scripts”. This is just a relative path. Actual path may vary based on your installation. You can find two files here; one is CreateCachingDb.cmd which has command which will run one sql script which will create tables and stored procedures to your database. Second is CreateCachingDatabase.sql which will have sql query to create necessary stored procedures and tables. If you can’t find the scripts folder, then you may have to install the “Enterprise Library 5.0 - Source Code.msi” which comes along with Enterprise library 5.0 install.
