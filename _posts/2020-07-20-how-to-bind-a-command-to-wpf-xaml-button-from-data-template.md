---
layout: post
modified: '2015-06-07 17:31 +0530'
published: true
comments: true
title: How to bind a command to WPF XAML button from Data template.
description: >-
  This post will help you in binding a command to a button, where the button is
  defined in a template say a data template. With normal command binding it
  won’t work. We need some sort of workaround to achieve this.
tags:
  - WPF
categories:
  - UI Controls
date: '2015-06-07 17:31 +0530'
---
### Introduction
This post will help you in binding a command to a button, where the button is defined in a template say a data template. With normal command binding it won’t work. We need some sort of workaround to achieve this.

### How to fix this.
Here the requirement is to bind a command to an element which is defined in a template. When we do a normal command binding, it is not aware of the data context and will not resolve. We need to provide the binding with the data context. This can be achieved in many ways say you can do

{% highlight csharp %}
<Button Content="{Binding}" Command="{Binding RelativeSource={RelativeSource Window}, Path=DataContext.ClickHandlerCommand }" />
<Button Content="{Binding}" Command="{Binding Path= ClickHandlerCommand, Source={StaticResource MainWindow}}" />
<Button Content="{Binding}" Command="{Binding Path= ClickHandlerCommand, RelativeSource={RelativeSource AncestorType={x:Type Window}}}" />
{% endhighlight %}

But these won’t work with all scenarios. But one solution will work with all scenarios. Define your data context as a static resource and refer it as the source of the binding. As shown below.

{% highlight csharp %}
<Window […]
        xmlns:this="clr-namespace:Your.Namespace.For.The.ViewModel"
        […]>

    <Window.Resources>
        <this:MainWindowViewModel x:Key="Model" />

        <DataTemplate x:Key="ItemsDataTemplate">
          <Button Content="{Binding }" Command="{Binding Command, Source=                            {StaticResource Model}}" />
        </DataTemplate>

    </Window.Resources>

    <Window.DataContext>
        <this:MainWindowViewModel></this:MainWindowViewModel>
    </Window.DataContext>

        <Grid>
        <ItemsControl ItemTemplate="{DynamicResource ItemsDataTemplate"  ItemsSource="{Binding Collections, Mode=OneTime}" >
            
        </ItemsControl>
    </Grid>
</Window>
{% endhighlight %}
