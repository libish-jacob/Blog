---
layout: post
modified: '2014-04-13 11:49 +0530'
published: true
comments: true
title: IP Control for WPF
categories:
  - UI Controls
  - .Net
tags:
  - 'C#'
description: >-
  This control is a simple to use IP-Control made out of WPF text box control.
  You can use this IP-Control in any Microsoft WPF/Silverlight application. This
  control is inherited from TextBox control.
date: '2014-04-13 11:49 +0530'
---
## Introduction
  Here I am introducing a WPF IP-Control. This control is a simple to use IP-Control made out of WPF text box control. You can use this IP-Control in any Microsoft WPF/Silverlight application. This control is inherited from TextBox control. This can be used like any text box control. But the display will be rendered to look like an IP-Control. Below is the image of the proposed WPF IP-Address Control.

![IP Control]({{site.baseurl}}/images/IP-Control.JPG)

How to use this IP-Address Control? 
     Attached here in this article is the IP-Address Control class. You can download it from [Github page](https://github.com/libish-jacob/IP-Control). Contribution to this control is appreciated. It can be used like any other TextBox control. You can bind or enter any valid IP-Address value to this control but the output will be rendered in an IP-Address v4 format. Now we will see how we can use this control in a xaml.

{% highlight csharp %}
<wizardDemo:IPControl Height="25" Width="100" Text="{Binding IPText, Mode=TwoWay}" />
{% endhighlight %}