---
layout: post
modified: '2015-06-07 16:28 +0530'
published: false
comments: true
title: >-
  Mixed mode assembly is built against version 'v2.0.*' of the runtime and
  cannot be loaded in the 4.0 runtime without additional configuration
  information
description: >-
  In this post we will see how to fix the exception due to mixed mode assembly
  while working with version 4.0 of runtime.
tags:
  - 'C#'
  - Visual Studio
categories:
  - .Net
date: '2015-06-07 16:28 +0530'
---
##  Mixed mode assembly is built against version 'v2.0.*' of the runtime and cannot be loaded in the 4.0 runtime without additional configuration information.

In this post we will see how to fix the exception due to mixed mode assembly while working with version 4.0 of runtime.

### Background
I was working with a project which uses an older mixed mode assembly and I got an exception like mixed mode assembly is built against another version of the runtime and cannot be loaded in the 4.0 runtime without additional configuration information. When I was able to fix this, I felt like this should be shared with all so that people can save their time.

### Using the code
Mixed mode assembly is built against another version of the runtime and cannot be loaded in the 4.0 runtime without additional configuration information.

If you are using .net version 4.0 there can be a chance of getting an error like this.

Additional information: Mixed mode assembly is built against version 'v2.0.50727' of the runtime and cannot be loaded in the 4.0 runtime without additional configuration information.

This error happens because of the support for side-by-side runtimes, .NET 4.0 has changed the way that it binds to older mixed-mode assemblies. These assemblies are, for example, those that are compiled from C++\CLI. Currently available DirectX assemblies are mixed mode.

### How did I fixed it
To fix this, You have to add the below settings to your App.config file under configuration tag.
{% highlight csharp %}
<startup useLegacyV2RuntimeActivationPolicy="true">
    <supportedRuntime version="v4.0"/>
    <requiredRuntime version="v4.0.20506"/>
</startup>
{% endhighlight %}


