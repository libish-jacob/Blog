---
layout: post
modified: '2014-04-13 10:27 +0530'
published: true
comments: true
title: How to disable a keyboard shortcut handler in wpf
tags:
  - WPF
categories:
  - UI Controls
  - .Net
date: '2014-04-13 10:27 +0530'
description: >-
  WPF Controls like DataGrid have default shortcut keys. For example, if you try
  to select any row and press Delete button then the grid row will be deleted.
  This is the default behavior. We can disable it in WPF the MVVM way by using
  InputBindings.
---
## Introduction:
WPF Controls like DataGrid have default shortcut keys. For example, if you try to select any row and press Delete button then the grid row will be deleted. This is the default behavior. We can disable it in WPF the MVVM way by using InputBindings. InputBindings represents a binding between an input gesture and a command.

## How did I do it:

Here we will see how to disable default keyboard shortcut in DataGrid.

{% highlight csharp %}
<datagrid itemssource="{Binding Values}">
  <datagrid.inputbindings>
    <keybinding command="{Binding NotACommand}" key="Delete" modifiers="Control"></keybinding>
  </datagrid.inputbindings>
</datagrid>
{% endhighlight %}

And in your viewmodel, the command NotACommand will look like this.

{% highlight csharp %}
internal class MainWindowViewModel {
  public ICommand NotACommand  {
    get {
      return ApplicationCommands.NotACommand;
    }
  }
}
{% endhighlight %}