---
layout: post
modified: '2014-12-09 11:59 +0530'
published: true
comments: true
title: '[Solved] How to subscribe to a mouse click event on a row in radgridview'
tags:
  - WPF
categories:
  - UI Controls
description: >-
  Subscribing to a row click event is not straight forward in WPF Telerik
  RadGridView. In this post we will see how we can achieve this.
date: '2014-12-09 11:59 +0530'
---
## Introduction
 Subscribing to a row click event is not straight forward in WPF Telerik RadGridView. In this post we will see how we can achieve this.
  Subscribing to a mouse click event would have been straight forward if all the rows were available at the time of gridview loaded event. But as in most scenarios, not all rows are available at this time, so when you subscribe to a normal Mouse Click event, only a click on the Header row will trigger the events. As the GridView can be updated with values at a later stage, it is not easy to subscribe to those rows which are loaded at a later stage.

### How to achieve this?

 We can achieve this by subscribing to mouse events at some later point. The most appropriate one is when row is loaded, this is because whenever a new value is added to the gridview, this event will be fired, this way we can make sure that all rows will fire the mouse click event.

{% highlight csharp %}
public ViewConstructor() {
/* Please note this is the constructor of your view*/
InitializeComponent();
RadGridView1.RowLoaded += this.RadGridViewRowLoaded;
}

private void RadGridViewRowLoaded(object sender, RowLoadedEventArgs e) {
   var row = e.Row as GridViewRow;

   if (row != null) {
        RadGridView1.AddHandler(GridViewRow.MouseLeftButtonDownEvent, new MouseButtonEventHandler(this.GridViewRowMouseLeftButtonDown), true);
      }
  }

private void GridViewRowMouseLeftButtonDown(object sender, MouseButtonEventArgs e) {
    // Now you will start getting a call here when you click on a row.
}
{% endhighlight %}
