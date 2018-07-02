---
layout: post
modified: '2014-08-26 10:55 +0530'
published: true
comments: true
title: Error Two-way binding requires Path or XPath
description: >-
  Worried over this exception? This happens when you haven’t set the Path= on
  binding. By default WPF binding will take the Path= part by default. But on
  controls which have a two way binding by default, this seems not to be
  working.
tags:
  - WPF
categories:
  - UI Controls
date: '2014-08-26 10:55 +0530'
---
## Introduction

  Worried over this exception? This happens when you haven’t set the Path= on binding. By default WPF binding will take the Path= part by default. But on controls which have a two way binding by default, this seems not to be working. You may have to set it explicitly. But in certain scenarios, you may be binding the value in a DataTemplate, for example,

{% highlight csharp %}
 <DataTemplate x:Key="BoolValueTemplate">
    <DockPanel>
       <RadioButton IsChecked="{Binding }" GroupName="IsSelectedGroup">
       </RadioButton>
       <RadioButton IsChecked="{Binding Converter={StaticResource InvertedValueConverter}}" GroupName="IsSelectedGroup">
       </RadioButton>
    </DockPanel>
 </DataTemplate>
{% endhighlight %}

Here you prefer the control to take the default value which is set by the ContentControl or TemplateParent. If you try to bind it like this, you will get an exception which says like, Two-way binding requires Path or XPath. To fix this, you have to specify the path like as shown below.

{% highlight csharp %}
Path=.
{% endhighlight %}

{% highlight csharp %}
<DataTemplate x:Key="BoolValueTemplate">
    <DockPanel>
       <RadioButton IsChecked="{Binding Path=.}" GroupName="IsSelectedGroup">
       </RadioButton>
       <RadioButton IsChecked="{Binding Path=. , Converter={StaticResource InvertedValueConverter}}" GroupName="IsSelectedGroup">
       </RadioButton>
     </DockPanel>
</DataTemplate>
{% endhighlight %}

Now here this cannot be called as a bug, but it’s a feature which blocks developers from adding a binding which they assume to be a one way binding. Those controls which take two-way binding by default will expect developers to explicitly specify Path.
