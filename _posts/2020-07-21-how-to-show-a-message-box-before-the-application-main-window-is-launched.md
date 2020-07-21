---
layout: post
modified: '2016-10-06 10:41 +0530'
published: true
comments: true
title: How to show a message box before the application main window is launched
description: >-
  There are scenarios where we have to show a standalone message box even before
  the application main window is launched. In such scenario, we need to do
  something different. Say if we have a composite application with a
  bootstrapper, and we want to show some error while running a bootstrapper, it
  may look difficult because we don’t have a parent window. In such scenario, we
  can show the message box as shown below.
tags:
  - 'C#'
categories:
  - .Net
date: '2016-10-06 10:41 +0530'
---
### Introduction
There are scenarios where we have to show a standalone message box even before the application main window is launched. In such scenario, we need to do something different. Say if we have a composite application with a bootstrapper, and we want to show some error while running a bootstrapper, it may look difficult because we don’t have a parent window. In such scenario, we can show the message box as shown below.

{% highlight csharp %}
private static void ShowError(string message)
{            
/* We cannot show a message out in void. So we are creating a dummy window and setting visibility as hidden. we will then use this as the parent of the message box. */

var w = new Window() { Height = 0, Width = 0, Visibility = Visibility.Hidden, WindowStyle = WindowStyle.None };

w.Show();
System.Windows.MessageBox.Show(w, message);
w.Close();
}
{% endhighlight %}

Here we will create and show a window where the visibility of the window is set to hidden, now we will use this window as the parent of the message box and will show the message box.
