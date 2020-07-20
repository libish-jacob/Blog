---
layout: post
modified: '2015-01-20 17:05 +0530'
published: true
comments: true
title: Adding Hot Keys ( HotKey or Shortcut Key ) in WPF window
description: >-
  In this post we will see how to add hot keys to WPF window. Hot keys are
  keyboard shortcuts with which we can easily navigate through the window
  without the help of a mouse. Hot keys are those which when pressing “Alt”+
  hotkey character will take you to the control associated with that hot key.
tags:
  - WPF
  - 'C#'
categories:
  - UI Controls
  - .Net
date: '2015-01-20 17:05 +0530'
---
In this post we will see how to add hot keys to WPF window. Hot keys are keyboard shortcuts with which we can easily navigate through the window without the help of a mouse. Hot keys are those which when pressing “Alt”+ hotkey character will take you to the control associated with that hot key.

### Background
When we have a window, which have many controls we need hot keys which will help users to navigate through the UI controls. When users are used with keyboard, they tend to use keyboard itself and avoids mouse most of the time. Keeping such users in mind, we have to provide hot keys in our application window.

You can find the hot keys in an application by pressing the “Alt” button on your keyboard. When you press on the “Alt” button, all those hot keys will be shown as an underlined character. Now if you press “Alt” + “HotKey” you can navigate to that control. For example, if you take the Remote Desktop Connection window in your windows (type mstsc in your run window) and when you press “Alt” button, you can see an underline on each character which is associated with hotkeys. And now if you press “Alt” + that character you can navigate to the control associated with that hot key. Here in this post we will see how to associate a hot key to WPF window.

### Using The Code
Here we will create two textbox and two label associated with that text box. When we press “Alt” button, these labels will highlight the hot keys. When we press Alt + Hot key character, it will take the focus to those text box which is associated with that hot key. Let’s create two textbox and two label which is associated with the text box.

{% highlight csharp %}
<Grid>
   <ItemsControl>
    <Label Content="FirstBox" Width="100" Height="28" Name="label1"/>
    <TextBox Name="textBox1" Text="Title" Height="50" ></TextBox>
   </ItemsControl>
   <ItemsControl>
    <Label Content="SecondBox" Width="100" Height="28" Name="label2"/>
    <TextBox Name="textBox2" Text="Title2" Height="50" ></TextBox>
   </ItemsControl>
</Grid>
{% endhighlight %}

Now let’s see how to associate hot keys to the labels. This is pretty much simple; just add an underscore before the character of the label content which you need as a hot key. Here we will make “F” as the hot key for our label1 and “B” as the hot hey for label2.

{% highlight csharp %}
<Grid>
   <ItemsControl>
    <Label Content="_FirstBox" Width="100" Height="28" Name="label1"/>
    <TextBox Name="textBox1" Text="Title" Height="50" ></TextBox>
   </ItemsControl>
   <ItemsControl>
    <Label Content="Second_Box" Width="100" Height="28" Name="label2"/>
    <TextBox Name="textBox2" Text="Title2" Height="50" ></TextBox>
   </ItemsControl>
</Grid>
{% endhighlight %}


Remember our hot key is intended for textbox so we need some mechanism by which we can pass the control to the text box from labels. We can use the Target property of the Label control to assign the Label to the TextBox and this way we can set the access key for the TextBox. Now bind the label to the textbox using the target attribute.

{% highlight csharp %}
<Grid>
   <ItemsControl>
    <Label Content="_FirstBox" Width="100" Height="28" Name="label1" Target="{Binding ElementName=textBox1}"/>
    <TextBox Name="textBox1" Text="Title" Height="50" ></TextBox>
   </ItemsControl>
   <ItemsControl>
    <Label Content="Second_Box" Width="100" Height="28" Name="label2" Target="{Binding ElementName=textBox2}"/>
    <TextBox Name="textBox2" Text="Title2" Height="50" ></TextBox>
   </ItemsControl>
</Grid>
{% endhighlight %}

Now run the code and see your hot keys working!