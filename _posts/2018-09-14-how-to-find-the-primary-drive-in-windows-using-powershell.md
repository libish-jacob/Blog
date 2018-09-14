---
layout: post
modified: '2018-09-14 09:49 +0530'
published: true
comments: true
title: How to find the primary drive in windows using powershell
tags:
  - Powershell
categories:
  - Operating System
date: '2018-09-14 09:49 +0530'
description: >-
  At times we may have to find some space in the machine where we can store some
  data. We cannot guess the drive as it may be different on different machine.
  With powershell we can find the drive to where the OS is mounted and can try
  to store the data there. The below line of code will help you achieve this
  using powershell.
---
## Introduction

At times we may have to find some space in the machine where we can store some data. We cannot guess the drive as it may be different on different machine. With powershell we can find the drive to where the OS is mounted and can try to store the data there. The below line of code will help you achieve this using powershell.

{% highlight csharp %}
$disk = Get-WmiObject -Class Win32_logicaldisk -Filter "VolumeName = 'OSDisk'"
echo ($disk.Properties | Where-Object{ $_.Name -eq  "Caption"}).Value.Trim();
{% endhighlight %}