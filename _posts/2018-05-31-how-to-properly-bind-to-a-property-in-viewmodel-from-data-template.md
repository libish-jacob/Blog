---
layout: post
modified: '2014-05-20 13:11 +0530'
published: true
comments: true
title: How to properly bind to a property in ViewModel from data template
description: >-
  Often we face a situation where we have defined a DataTemplate and we need to
  bind to a property, most often to a Command in ViewModel. To achieve this we
  will define the binding and will set a RelativeSource for that binding which
  in some cases can ends up in an error like this.
tags:
  - WPF
categories:
  - UI Controls
  - .Net
date: '2014-05-20 13:11 +0530'
---
## Introduction
 Often we face a situation where we have defined a DataTemplate and we need to bind to a property, most often to a Command in ViewModel. To achieve this we will define the binding and will set a RelativeSource for that binding which in some cases can ends up in an error like this. BindingExpression path error: … property not found on 'object' … BindingExpression:Path=… DataItem=…

{% highlight csharp %}
<Window.Resources>
        <DataTemplate x:Key="TestTemplate">
            <DockPanel>
                <Label Content="{Binding}" />
                <Button Command="{Binding CommandHandler, RelativeSource={RelativeSource AncestorType=Grid}}" />
            </DockPanel>
        </DataTemplate>
</Window.Resources>
<Grid>
    <ItemsControl ItemTemplate="{StaticResource TestTemplate}" ItemsSource="{Binding Values}" />
</Grid>
{% endhighlight %}

In most of the situation, the binding which is shown above works, but in certain situation like this example, this will fail to bind and if you examine the output window in visual studio, you can see a similar error as below.

_BindingExpression path error: […] property not found on 'object' […] BindingExpression:Path=[…] DataItem=…_

Here in this example, the exact error which you will get is:-

 _System.Windows.Data Error: BindingExpression path error: 'CommandHandler' property not found on 'object' ''Grid' (Name='')'. BindingExpression:Path=CommandHandler; DataItem='Grid' (Name=''); target element is 'Button' (Name=''); target property is 'Command' (type 'ICommand')_

## How did I fix this?

This happens because, when we define a RelativeSource, WPF tries to resolve the property which we are binding, from the object it finds up in the hierarchy whose type is equal to the type which we have defined for AncestorType. Here the relative source is Grid and Grid object does not have a property called CommandHandler, this is exactly what the error says.

Now if we examine, the Grid object has a property called DataContext, which is set or inherited from its parent to as the ViewModel, and the ViewModel has the property which we need. So a simple fix is to say DataContext.CommandHandler which binds to the CommandHandler property in ViewModel. The final binding is as shown below.

{% highlight csharp %}
<Window.Resources>
    <DataTemplate x:Key="TestTemplate">
        <DockPanel>
            <Label Content="{Binding}"/>
            <Button Command="{Binding DataContext.CommandHandler,RelativeSource={RelativeSource AncestorType=Grid}}" />
        </DockPanel>           
    </DataTemplate>
</Window.Resources>

<Grid>
    <ItemsControl ItemsSource="{Binding Values}" ItemTemplate="{StaticResource TestTemplate}" />
</Grid>
{% endhighlight %}
