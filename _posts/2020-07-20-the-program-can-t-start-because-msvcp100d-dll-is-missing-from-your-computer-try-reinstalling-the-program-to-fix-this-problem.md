---
layout: post
modified: '2015-02-25 16:03 +0530'
published: true
comments: true
title: >-
  The program can't start because MSVCP100D.dll is missing from your computer.
  Try reinstalling the program to fix this problem
description: >-
  We may get this error when we try to execute an application which has CPP
  component. Here MSVCP100 is a file which should be installed along with
  Microsoft Visual C++ 2010 Re-distributable Package. Here 100 means 2010 and D
  means debug version. This error occurs only when you are trying to run your
  application which is compiled using the “debug” configuration instead of
  “Release”.
tags:
  - 'C#'
categories:
  - Operating System
  - .Net
  - IDE
date: '2015-02-25 16:03 +0530'
---
## The program can't start because MSVCP100D.dll is missing from your computer. Try reinstalling the program to fix this problem

We may get this error when we try to execute an application which has CPP component. Here MSVCP100 is a file which should be installed along with Microsoft Visual C++ 2010 Re-distributable Package. Here 100 means 2010 and D means debug version. This error occurs only when you are trying to run your application which is compiled using the “debug” configuration instead of “Release”.

Here when you try to run the application which was the debug output, then it would expect the appropriate file which has the Debug version, when you don’t have visual studio installed on your machine, then you will experience this kind of issues.

You can fix it in three different ways. You only have to follow any of the below. The first two options are straight forward.

- Install visual studio.
- Build your application using release configuration.
- Copy the files to system32 folder.

### 3. Copy the files to system32 folder
Here you can find the missing dll from internet but **never ever do that!** Those files could be contaminated either intentionally or unintentionally and it may help expose your machine to virus attack.

What you have to do is, find a machine which has visual studio 2010 installed, If you are running the application in a 32 bit machine, then go to C:\Windows\System32 and copy the missing file and place it in C:\Windows\System32 of the machine where you want to run your application.

If you are running your application in a 64 bit machine, then the folder where you have to copy the file is bit different. You have to go to C:\Windows\SysWOW64 and copy the missing files and paste it to C:\Windows\SysWOW64 of the machine where you have to run the application. This will help you fix the error.

In in case you have copied from system32 folder and you have placed in SysWOW64 folder, then you may end up getting an error like "The application was unable to start correctly (0xc000007b)".
