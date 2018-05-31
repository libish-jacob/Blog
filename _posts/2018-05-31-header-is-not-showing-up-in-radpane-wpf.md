---
layout: post
modified: '2014-06-31 13:43 +0530'
published: true
comments: true
title: Header is not showing up in RadPane wpf
description: >-
  RadPane in telerik controls can show a header. We normally use the Header
  property to set the header for RadPane. This works most of the time. But in
  some scenarios
tags:
  - WPF
categories:
  - UI Controls
---
## Introduction
  RadPane in telerik controls can show a header. We normally use the Header property to set the header for RadPane. This works most of the time. But in some scenarios, say for example when we use nested RadPane, this kind of usage doesn't seem to work. The header suddenly disappears. This is a know issue in telerik.

## How did I fix this?

To fix this, we have to use a different property in RadPane. This is the “Title” property, use this  property and this will show the header in all scenarios.

{% highlight csharp %}
<telerik:RadPane Title="Your Header" />
{% endhighlight %}