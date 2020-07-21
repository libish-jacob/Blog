---
layout: post
modified: '2016-07-21 10:31 +0530'
published: true
comments: true
title: >-
  An endpoint configuration section for contract could not be loaded because
  more than one endpoint configuration for that contract was found. Please
  indicate the preferred endpoint configuration section by name
description: >-
  This fix explained here is helpful if you are using WCF and when you try to
  access the service, the client throws an exception like "An endpoint
  configuration section for contract ‘YourContract' could not be loaded because
  more than one endpoint configuration for that contract was found. Please
  indicate the preferred endpoint configuration section by name".
tags:
  - WCF
categories:
  - .Net
date: '2016-07-21 10:31 +0530'
---
### Introduction
This fix explained here is helpful if you are using WCF and when you try to access the service, the client throws an exception like "An endpoint configuration section for contract ‘YourContract' could not be loaded because more than one endpoint configuration for that contract was found. Please indicate the preferred endpoint configuration section by name".

### How to fix this exception
WCF service can expose more than one endpoint and client can use any of this endpoint but not more than one at a time. This type of exception occurs when you have more than one endpoint configured at your configuration file (app.config) file while adding service reference or while configuring service at client side manually. You can have only one and only endpoint for the client. If you are getting this exception then check your client side configuration file and make sure the client tag under system.serviceModel has only one endpoint configured.

{% highlight csharp %}
<system.serviceModel>
   <client>
      <endpoint [...]>
     </endpoint>
   </client>
</system.serviceModel>
{% endhighlight %}
