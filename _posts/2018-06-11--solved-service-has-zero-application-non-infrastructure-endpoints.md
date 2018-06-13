---
layout: post
modified: '2014-07-23 16:41 +0530'
published: true
comments: true
title: >-
  [Solved] Service has zero application (non-infrastructure) endpoints. This
  might be because no configuration file was found for your application, or
  because no service element matching the service name could be found in the
  configuration file, or because no endpoints were defined in the service
  element.
description: >-
  Service has zero application (non-infrastructure) endpoints. This might be
  because no configuration file was found for your application, or because no
  service element matching the service name could be found in the configuration
  file, or because no endpoints were defined in the service element. This can
  happen if there is no proper endpoint configured for the service.
tags:
  - WCF
categories:
  - .Net
date: '2014-07-23 16:41 +0530'
---
## Introduction

Service has zero application (non-infrastructure) endpoints. This might be because no configuration file was found for your application, or because no service element matching the service name could be found in the configuration file, or because no endpoints were defined in the service element. This can happen if there is no proper endpoint configured for the service.

## How to fix this

Create at least one endpoint for the service. If you are configuring endpoint using configuration file, then make sure you have your application configuration file in the output directory. Give proper name for the service.

Possible problem for Service has zero application (non-infrastructure) endpoints exception.

Here if you are getting this exception after you have given proper endpoint, then the main culprit can be the name of the service. The service name has to be the class name of the service. Give the fully qualified name of the class with namespace. If your class name is “Service” and is in namespace “WcfService” as shown below

{% highlight csharp %}
namespace WcfService {
[System.ServiceModel.ServiceBehavior()]
public class Service : IService
{}
}
{% endhighlight %}

Then your service name has to be something as shown below.

{% highlight csharp %}
<service name="WcfService.Service">
{% endhighlight %}
