---
layout: post
modified: '2014-05-19 13:05 +0530'
published: true
comments: true
title: Intermittent bug. Order of attribute does matter in WPF
description: >-
  In WPF, some properties have built-in dependency on other properties. WPF will
  process the attribute in its order. If the properties are not placed based on
  their priority, then there is no other ways to express such a dependency. In
  such cases order becomes important.
tags:
  - WPF
categories:
  - UI Controls
  - .Net
date: '2014-05-19 13:05 +0530'
---
## Introduction
  When I was developing some XAML page, I encountered one situation where I experienced some intermittent bug from WPF. Some of my buttons will go disabled at times even if the CommandParameter bind to a buttons command has proper value to enable button. Below is the solution for this.

## How did I fix it:

For example, you might write something like this for your buttons command binding to the corresponding view model:

{% highlight csharp %}
<button command="{Binding Path=Command}" commandparameter="{Binding Path=PathTo}">Test</button>
{% endhighlight %}

Here in this example the command expects a non-null parameter as the CommandParameter.
In WPF, some properties have built-in dependency on other properties. WPF will process the attribute in its order. If the properties are not placed based on their priority, then there is no other ways to express such a dependency. In such cases order becomes important.

In most of the situation, this won’t be a problem. Because in the process of creating the control, focus is shifted to the window that contains it, you will fire enough checks that the command will be evaluated multiple times and the button will get enabled. But if you are creating the control on another window or control that does not contain the focus, the order of the above code will result in a disabled button until you shift focus.

To correct this problem, respect the order of attributes in XAML. If you just swap the attributes so that you bind the CommandParameter first, then the Command will have the correct parameter when it gets evaluated.

{% highlight csharp %}
<button commandparameter="{Binding Path= PathTo }" command="{Binding Path=Command}"> Test</button>
{% endhighlight %}

In WPF attributes have priorities and we have to sort attributes based on their priority.
To sort it manually is very difficult as you have to remember the order of priority. Here what I would recommend is to use some tool which sorts attributes in its order based on its priority. XAMLStyler is one such tool. By using such a tool we can get rid of any possibility of such errors. The above mentioned example is just an example. There are many other occasions where we need to take care of the priority of the attributes.