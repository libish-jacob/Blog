---
layout: post
modified: '2016-10-21 10:44 +0530'
published: true
comments: true
title: How to know if the window state has changed to maximized in WPF
description: >-
  In this post we will see how we can identify if the application has been
  maximized or it is restored to normal state. We will use the Resize event
  equivalent in WPF which is SizeChanged event. We will subscribe to this event
  and will check the window state to identify if the window has been maximized
  or it is restored to normal.
tags:
  - WPF
categories:
  - .Net
date: '2016-10-21 10:44 +0530'
---
### Introduction
In this post we will see how we can identify if the application has been maximized or it is restored to normal state. We will use the Resize event equivalent in WPF which is SizeChanged event. We will subscribe to this event and will check the window state to identify if the window has been maximized or it is restored to normal.

{% highlight csharp %}
Application.Current.MainWindow.SizeChanged += WindowSizeChanged;
private void WindowSizeChanged(object sender, SizeChangedEventArgs e)
{
    this.HandleWindowState();
}
 
private void HandleWindowState()
{
    WindowState windowState = Application.Current.MainWindow.WindowState;
    if (windowState == WindowState.Maximized)
     {                    
     }
    else if (windowState == WindowState.Normal)
     {                    
     }            
}
{% endhighlight %}

And let’s not forget to clean it by unsubscribing to the SizeChanged event by
{% highlight csharp %}
Application.Current.MainWindow.SizeChanged -= WindowSizeChanged;
{% endhighlight %}
