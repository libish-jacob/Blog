---
layout: post
modified: '2015-10-12 19:43 +0530'
published: true
comments: true
title: How to host a WCF service in console application using wsHttp binding
description: >-
  In this article we will see how to host a wcf service using the wsHttpBinding.
  When you use http, you would normally host it in IIS, but if you would like to
  host it in a console application, you could follow this article. Here we will
  try to host a service with a wsHttpBinding and will expose a MEX endpoint in
  order to expose the metadata so that any tool which can generate a proxy can
  use it.
tags:
  - WCF
categories:
  - .Net
date: '2015-10-12 19:43 +0530'
---
### Introduction
In this article we will see how to host a wcf service using the wsHttpBinding. When you use http, you would normally host it in IIS, but if you would like to host it in a console application, you could follow this article. Here we will try to host a service with a wsHttpBinding and will expose a MEX endpoint in order to expose the metadata so that any tool which can generate a proxy can use it. Here in this example we are exposing the MEX endpoint particularly to be used in the WCF test client. This article will not cover how you can generate a wcf service. If you like to know how a basic service is created, you need to read [Creating WCF Service and Self Hosting in a console application.](http://simplebasics.net/.net/creating-wcf-service-and-self-hosting-in-a-console-application-and-creating-proxy-for-net-tcp-and-net-pipe-endpoint-with-visual-studio/)

### How to edit wcf configuration.
All your wcf configuration will be stored in the app.conf file of your host project. Here in our case as we are going to host it in a console application, it will be the app.conf of the console application. You could use the wcf configuration edit tool to edit this configuration. To open the wcf configuration tool, do as shown in the image.

![edit wcf]({{site.baseurl}}/images/edit-wcf-conf.png)

### Adding an endpoint with wsHttpBinding
Here we need to add an wshttp endpoint. Create your endpoint as shown in the image below.

![Wcf service endpoint]({{site.baseurl}}/images/Wcf-service-endpoint.PNG)

Here in the address field, give the name of the interface where you have defined a service contract. Here the address name could be anything but we will keep it as the name of the interface, this is the relative address which we are defining; we will later add a base address for this. Add wsHttpBinding on to the binding column. Here we defined wsHttp as the binding for this endpoint. To the contract field, browse to the dll where we have the interface on which we have defined the service contract. You could point to the dll where we have this interface. 

### Adding a base address for the endpoint.
Now we need to add a base address to the service, So that the complete address for the service will look like “Base Adress + Name of the service” Add it as shown in the image. Here the port number can be any port number except the reserved and already used.

![Adding wcf base address]({{site.baseurl}}/images/Adding-wcf-base-address.PNG)

With this, we are done creating a basic endpoint.

### Adding MEX Endpoint.
Now create a wsHttp MEX endpoint as shown in the image. Here IMetadataExchange is the default contract implementation from Microsoft. We will just add this text as the contract implementation of MEX.
![mex]({{site.baseurl}}/images/Mex.PNG)

Now this is a MEX end point exposed via http, so we need to enable some more setting for this MEX end point to work. Go to advanced->Service Behavior->MEX and set HttpGetEnabled to true as shown in the image below. 

![Enabling Mex over http]({{site.baseurl}}/images/Enabling-Mex-over-http.PNG)

### Adding Mex behaviour to the service.
Now we need to enable MEX behaviour to the service. Do as shown in the image. Select MEX from the dropdown for BehaviorConfiguration. Remember, MEX is the name given to the Mex endpoint.

![Adding Mex Behavior]({{site.baseurl}}/images/Adding-Mex-Behavior.PNG)

Now we are ready, now remember to run the service host as Admin, because when we have http endpoint, it requires this or else it will throw an access violation exception. Now open the WCF test client and add the service endpoint to test it.

![Wcf test client]({{site.baseurl}}/images/Wcf-test-client.PNG)

