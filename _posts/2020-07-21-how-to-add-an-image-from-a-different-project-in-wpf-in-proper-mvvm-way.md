---
layout: post
modified: '2014-05-09 11:32 +0530'
published: true
comments: true
title: How to add an image from a different project in WPF in proper MVVM way
description: >-
  At times we will face with a situation where we need to show an image from
  another project in the same solution. Say for example: If you are building an
  enterprise application then we will modularize the application. And at times
  we want to show some image from this module in the main shell view. This at
  times can run you crazy. Here we will see how to show an image from a
  different module in the shell view.
tags:
  - WPF
categories:
  - .Net
date: '2014-05-09 11:32 +0530'
---
### Issue
At times we will face with a situation where we need to show an image from another project in the same solution. Say for example: If you are building an enterprise application then we will modularize the application. And at times we want to show some image from this module in the main shell view. This at times can run you crazy. Here we will see how to show an image from a different module in the shell view.

### How did I fix this?
You cannot show an image by simply binding to its path. What Image accepts as source is a BitmapImage. So unless you are not binding to a BitmapImage, you cannot bind it directly. So create an Image control and assign a BitmapImage control as source as shown below.
{% highlight csharp %}
<Image>
    <Image.Source>
        <BitmapImage UriSource="{Binding Path=ImagePath}" />
    </Image.Source>
 </Image>
{% endhighlight %}

Now in your module, add the image which you want to show in a sub folder of that module. Normally we add it in a folder called “Resources” in the root of the module project. Now right click on that image and select properties. Now change the “Build Action” to “Resource”. Now this image will be available as a resource, embedded into the dll.
![Adding to resource]({{site.baseurl}}/images/Adding-to-resource.JPG)

Now from this project, pass the path to this resource file as shown below.
{% highlight csharp %}
"pack://application:,,,/TestModuleProject;Component/Resources/Image.png"
{% endhighlight %}

Here “TestModuleProject” is the name of the project where you have the image. Now take this path in your shellViewModel and assign it to “ImagePath”, where ImagePath is bind to the BitmapImage. Now this will display your image.
