---
layout: post
modified: '2014-07-21 10:39 +0530'
published: true
comments: true
title: >-
  Solution files bind to TFS but files edited from visual studio are not showing
  in pending changes
description: >-
  This error can happen when you use Microsoft Team Foundation Server(TFS) and
  if you don’t have proper binding files. This can happen if the solution is new
  in TFS and there are no binding files associated with it. You can easily fix
  this. To fix it, you have to go to
categories:
  - Source control
date: '2014-07-21 10:39 +0530'
---
### Introduction
This error can happen when you use Microsoft Team Foundation Server(TFS) and if you don’t have proper binding files. This can happen if the solution is new in TFS and there are no binding files associated with it. You can easily fix this. To fix it, you have to go to

{% highlight csharp %}
File -> Sourcecontrol-> change source control.
{% endhighlight %}

Click on the project which don’t have binding and say bind. This will automatically bind your solution with the source control. Do the same for the entire project which don’t have proper binding.

