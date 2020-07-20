---
layout: post
modified: '2014-04-13 17:21 +0530'
published: true
comments: true
title: >-
  How to sort items in RadGridView based on column when it is loaded using
  SortDescriptors
description: >-
  Use SortDescriptors to Sort RadGridView in ascending order based on a column
  value when it is loaded. Sort descriptors can be used as shown below. Here the
  list bind to RadGridView is list of Signals. Structure of Signal is as shown
  below.
tags:
  - WPF
categories:
  - .Net
date: '2014-04-13 17:21 +0530'
---
Issue
How to Sort RadGridView in ascending order based on a column value when it is loaded.

### How did I fix it:
Use SortDescriptors to Sort RadGridView in ascending order based on a column value when it is loaded. Sort descriptors can be used as shown below. Here the list bind to RadGridView is list of Signals. Structure of Signal is as shown below.

{% highlight csharp %}
struct Signal
 {
   public int Id { get; set; }
   public string Name { get; set; }
 }
{% endhighlight %}

Here we are trying to sort the list displayed based in the Id in ascending order.

{% highlight csharp %}
<telerik:RadGridView ItemsSource="{Binding Events}">
<telerik:RadGridView.SortDescriptors>
    <telerik:SortDescriptor Member="Timestamp" SortDirection="Descending" />
    </telerik:RadGridView.SortDescriptors>
<telerik:RadGridView.Columns>
   <telerik:GridViewDataColumn DataMemberBinding="{Binding Id}" Header="ID" />
 <telerik:GridViewDataColumn DataMemberBinding="{Binding Description}" Header="Description" />
</telerik:RadGridView.Columns>
</telerik:RadGridView>
{% endhighlight %}