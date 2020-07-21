---
layout: post
modified: '2016-07-21 10:34 +0530'
published: true
comments: true
title: >-
  How to use Resource Dictionary in WPF and How to call Resource Dictionary from
  xaml
description: >-
  WPF applications support rich media and graphics, this require styles and
  styles can be reusable across different window or element in WPF. Here we need
  a mechanism by which we can share this style across different window or say
  application. By adding the style in application resource file, we can share it
  across windows in application but not across application.
tags:
  - WPF
categories:
  - .Net
date: '2016-07-21 10:34 +0530'
---
### Introduction
WPF applications support rich media and graphics, this require styles and styles can be reusable across different window or element in WPF. Here we need a mechanism by which we can share this style across different window or say application. By adding the style in application resource file, we can share it across windows in application but not across application. Here reusable styles need to be utilized and in a managed way. Here WPF provide an easy way to manage these styles and other reusable resources which can be used across WPF windows and across application. This is called Resources Dictionary. We can define the styles in WPF XAML files and can manage all our useful styles for a particular application in a resource dictionary file. This resources dictionary can then be referred from other WPF projects and can use the styles in this resources dictionary in that project.

### Using the code
Adding a resource dictionary is pretty simple. We have to select the project or folder in Solution Explorer and then right click and select “Add”. We will get a menu item called “Resource Dictionary”. Selecting that menu item will pop up the Add New Item wizard with the Resource Dictionary Item template selected. Rename the item as you wish. In a Resource Dictionary, we can keep our custom styles, Templates, custom definitions for Brush, Color, Background and a lot of other stuff. Most important thing is that we have to assign a key to each of them since it is a Dictionary and it stores the values besed on the key. Or perhaps, we can give names to the styles.

### How to refer to Resource Dictionary and use it in WPF xaml.
In this section, we are going to see how we can import a resource file to a XAML file for a WPF Window, user control or a page.  We can refer to the resource dictionary that we created for our window resources section. Since we can have a resource dictionary for each control, we are going to merge the other resource files to the existing resource dictionary.

{% highlight csharp %}
<Window […] >
    <Window.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary 
                  Source="ResourceDictionaryReferencePath/ResourceDictionaryName.xaml">
                </ResourceDictionary>
                <ResourceDictionary 
                  Source=" ResourceDictionaryReferencePath/ResourceDictionaryName.xaml ">
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Window.Resources>
    </Window>
{% endhighlight %}

### How to refer to resource dictionary from another project.
If your resource dictionary is in a different project, then make sure that the project can be referred from the project that you need the resource. If not then WPF will throw an exception at runtime. Either add reference to that project or drop that project dll to your applications folder so that your application can refer to it. Here the source path depends on where you have added your dictionary. If you have added resource file to the root of your application, then you can refer to this file as
{% highlight csharp %}
Source="ResourceDictionaryName.xaml"
{% endhighlight %}

If you have added this file to a directory in your project then you can refer to it as
{% highlight csharp %}
Source= "ResourcesDirectoryReferencePath/MyResourceDictionary.xaml"
{% endhighlight %}

If you your resources dictionary is in another project then you can refer to it as
{% highlight csharp %}
Source= "pack://application:,,,/ApplicationNameWhereResourceDictionaryResides; component/ResourceDictionaryName.xaml"
{% endhighlight %}
