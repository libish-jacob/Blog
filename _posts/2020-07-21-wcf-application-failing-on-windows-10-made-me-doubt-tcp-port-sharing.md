---
layout: post
modified: '2016-12-14 19:10 +0530'
published: true
comments: true
title: WCF Application failing on windows 10 made me doubt tcp port sharing
description: >-
  I started on this issue when I had my WCF service times out when I am trying a
  normal socket connection. This was running fine in windows 8. We had a
  keep-alive WCF call which tells the server that the client is still active.
  For this we need the port sharing to be enabled on the machine.
categories:
  - Operating System
date: '2016-12-14 19:10 +0530'
---
### Introduction
I started on this issue when I had my WCF service times out when I am trying a normal socket connection. This was running fine in windows 8. We had a keep-alive WCF call which tells the server that the client is still active. For this we need the port sharing to be enabled on the machine. I was on an impression that the port sharing was not working and because of which it fails. From the log I came to a conclusion that the keep alive was failing so this made me doubt the TCP port sharing. But I could also see that when I install visual studio, then the application works. This made me doubt the .net framework. Finally I detected the slow network. This was the real culprit.

Windows 10 comes with a real time protection which scans each packets received across TCP. This slows down the network. The fix for this is to disable the real time protection. On the windows settings you can find windows defender. You can switch off real-time protection.

![windows defender]({{site.baseurl}}/images/windows-defender.PNG)

![switch off defender]({{site.baseurl}}/images/Switch off defender.PNG)

But nothing is intended for bad purpose. So let’s not disable it. We have an option of setting the exclusion list. We have to use that to ignore known applications from this real time scanning. This was we can bring the application back to normal.

![update security]({{site.baseurl}}/images/update-security.PNG)

![windows defender]({{site.baseurl}}/images/Capture4.PNG)

