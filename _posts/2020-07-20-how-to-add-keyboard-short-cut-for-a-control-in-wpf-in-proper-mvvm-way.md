---
layout: post
modified: '2014-04-13 15:47 +0530'
published: true
comments: true
title: How to add keyboard short cut for a control in WPF in proper MVVM way
description: >-
  When you create an enterprise application, Keyboard shortcuts are a must.
  Enterprise applications will need keyboard shortcuts to execute certain
  actions in controls which are possible only by using mouse. Keyboard shortcut
  is different from keyboard access key where it will help you to get the focus
  on to control. Here we will use InputBindings to achieve this.
tags:
  - WPF
categories:
  - UI Controls
date: '2014-04-13 15:47 +0530'
---
## How to add keyboard short cut for a control in WPF in proper MVVM way

 When you create an enterprise application, Keyboard shortcuts are a must. Enterprise applications will need keyboard shortcuts to execute certain actions in controls which are possible only by using mouse. Keyboard shortcut is different from keyboard access key where it will help you to get the focus on to control. Here we will use InputBindings to achieve this. InputBindings represents a binding between an input gesture and a command.
 
### How did I fix it:

Here in this example, we will try to add a custom command handler to handle the input gesture ctrl+Del which we will bind to DeleteCommandHandler command. Now the DeleteCommandHandler command will get executed when user uses ctrl+Del.

{% highlight csharp %}
<UserControl.InputBindings>
    <KeyBinding Command="{Binding CancelButtonCommand}" Key="Escape" Modifiers="Control" />
</UserControl.InputBindings>
{% endhighlight %}
