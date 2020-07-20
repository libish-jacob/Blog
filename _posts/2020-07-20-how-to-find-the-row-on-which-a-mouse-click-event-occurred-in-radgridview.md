---
layout: post
modified: '2014-12-10 17:28 +0530'
published: true
comments: true
title: How to find the row on which a mouse click event occurred in radgridview
description: >-
  If you are able to subscribe to a mouse click event on a radgridView, then
  your immediate requirement could be to find the row in which the mouse click
  event occurred. In this post we will see how to achieve this.
tags:
  - WPF
categories:
  - UI Controls
date: '2014-12-10 17:28 +0530'
---
### Introduction
If you are able to subscribe to a mouse click event on a radgridView, then your immediate requirement could be to find the row in which the mouse click event occurred. In this post we will see how to achieve this.

### How to achieve this?
You can achieve this by querying the source of this event. In our case we will do something like this.

{% highlight csharp %}
private void GridViewRowMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
  // Either this way

  var senderElement = e.OriginalSource as FrameworkElement;
  var parentRow = senderElement.ParentOfType<GridViewRow>();

  // or this way

  var element = e.Source as FrameworkElement;
  var row = senderElement.ParentOfType<GridViewRow>();           
}
{% endhighlight %}
  
This approach can be applied to all similar controls say RadGridView, RadTreeView etc. Now when you have found the row which the user has clicked, your immediate requirement could be to find the object associated with the row. Say for example if you have found the row and if you want to assign this to the SelectedItem property, then you cannot assign this, this is because to set a selected item, you need to find the exact object which is bound to the row. If you are trying to find the object in the row then you could achieve this like as shown below.

{% highlight csharp %}
var tag = row as RadRowItem;
object item = tag.Item;
{% endhighlight %}
