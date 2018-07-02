---
layout: post
modified: '2014-08-21 10:46 +0530'
published: true
comments: true
title: >-
  TextWrapping on TextBlock results in Poor Performance When Dynamically
  Resizing a WPF TextBlock
description: >-
  TextWrapping is required when your text to display has to align itself inside
  a control. TextWrapping should work seamlessly when your control is of fixed
  height and width. But this won’t be the scenario in all cases. When we have a
  dynamic parent control whose width and height cannot be predicted, we would
  like the TextBlock which is placed inside it to adjust itself to fit into the
  container.
tags:
  - WPF
categories:
  - UI Controls
date: '2014-08-21 10:46 +0530'
---
## Introduction
TextWrapping is required when your text to display has to align itself inside a control. TextWrapping should work seamlessly when your control is of fixed height and width. But this won’t be the scenario in all cases. When we have a dynamic parent control whose width and height cannot be predicted, we would like the TextBlock which is placed inside it to adjust itself to fit into the container. In this case we would prefer to have TextBlock decide on the height and width based on the parent control. So we won’t set any width or height.
  
  Here, the TextWrapping algorithm will consider TextBlock to have infinite width and height. Also wrapping text is a very expensive operation. The algorithm must measure the size of every single line to figure out how to draw. When we don’t specify any width or height or say if we don’t specify a MaxWidth and MaxHeight, then we are adding load to the algorithm. So the algorithm will take time to calculate.

The answer which comes first is to set a MaxWidth and MaxHeight to TextBlock. But this won’t meet the requirement in most of the scenario. So there is an effective and ingenious solution.

Answer: Just wrap it around a Grid.

Grid layout panel will first get its height and width based on its parent container and then it will put the child control inside it. In this case, the TextBlock will get a MaxWidth and MaxHeight from its parent which is Grid. So now the algorithm can see that there is a MaxWidth and MaxHeight specified, so it will perform well.

{% highlight csharp %}
<DataTemplate x:Key="StringDataTemplate">
  <Grid>
     <Grid.ColumnDefinitions>
        <ColumnDefinition Width="55" />
        <ColumnDefinition Width="*" />
      </Grid.ColumnDefinitions>

      <Image Grid.Column="0"
             Margin="5"
             Source="{Binding Category,
             Converter= {StaticResource StatusImageConverter}}" />
             <TextBlock Grid.Column="1"
                    Padding="5"
                    Text="{Binding Content}"
                    TextWrapping="Wrap" />
  </Grid>
</DataTemplate>
{% endhighlight %}

I was implementing a dialog service which is supposed to display text messages as well as WPF view. Here the dialog which pops up will have to adjust its width and height based on the content. So when I was implementing the TextBox TextWrapping, I got this problem. I could not compromise on setting the width and height to the TextBlock and I was burning my time to figure out a solution to this. But I could get it working by wrapping it around a grid.
