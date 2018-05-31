---
layout: post
modified: '2014-04-13 10:14 +0530'
published: true
comments: true
title: How to hook into key press in WPF
tags:
  - WPF
categories:
  - .Net
date: '2014-04-13 10:14 +0530'
description: >-
  Here in this example, we will try to add a custom command handler to handle
  the input gesture ctrl+Del which we will bind to DeleteCommandHandler command.
---
## Introduction:
	This post is regarding implementing keyboard shortcuts for your control. When you create an enterprise application, Keyboard shortcuts are a must. Enterprise applications will need keyboard shortcuts to execute certain actions in controls which are possible only by using the mouse. Keyboard shortcut is different from keyboard access key where it will help you to get the focus on to control. Here we will use InputBindings to achieve this. InputBindings represents a binding between an input gesture and a command.

## How did I do it:
	Here in this example, we will try to add a custom command handler to handle the input gesture ctrl+Del which we will bind to DeleteCommandHandler command. Now the DeleteCommandHandler command will get executed when user uses ctrl+Del. You will define DeleteCommandHandler in your view model.

{% highlight csharp %}
<UserControl.InputBindings>
 <KeyBinding Command="{Binding CancelButtonCommand}" Key="Escape" Modifiers="Control" />
</UserControl.InputBindings>
{% endhighlight %}