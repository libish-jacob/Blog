---
layout: post
modified: '2015-09-22 20:10 +0530'
published: true
comments: true
title: >-
  Consuming self-hosted WCF service by Creating WCF net.tcp and net.pipe proxy
  at runtime and by custom proxy, without adding service reference
description: >-
  In this article we will see how to call a WCF service without adding a service
  reference. In this article we will see two ways of calling WCF service method.
  One is to call the WCF service method on the fly and second is by creating a
  custom proxy without the help of SvcUtil or any other tool. We will be using a
  two way service in this article.
tags:
  - WCF
categories:
  - .Net
date: '2015-09-22 20:10 +0530'
---
### Introduction
In this article we will see how to call a WCF service without adding a service reference. In this article we will see two ways of calling WCF service method. One is to call the WCF service method on the fly and second is by creating a custom proxy without the help of SvcUtil or any other tool. We will be using a two way service in this article.

### Background
This article will be a continuation of [[Article][Beginner] Creating WCF Service and Self Hosting in a console application and Creating Proxy of net.tcp and net.pipe endpoint via Visual Studio.](http://simplebasics.net/.net/creating-wcf-service-and-self-hosting-in-a-console-application-and-creating-proxy-for-net-tcp-and-net-pipe-endpoint-with-visual-studio/) Here we will see how to call a WCF service method without adding WCF service reference in the client project. Here we will be using the same WCF service project from the previous article but with some modification. Here we will add a common library project to which we will add all WCF service interfaces. Now this common project will be referred from client as well. Here you may feel like it’s not the proper way, but this kind of approach is necessary when service and client both are a part of a single application. Such situations arise when you are designing service oriented architecture.

In service oriented architecture, your services will be business functionalities that are built as software components that can be reused for different purposes. We will use WCF to create services which will implement business functionality.

Here in the first section we will see how to consume WCF service without adding service reference or by creating proxy via some tools. We will use ChannelFactory to achieve this. And in the second part we will see how to create a custom WCF proxy and to call the WCF service methods. The second part of this article comes in handy when you don’t have to update service reference each time you add some methods on to the service implementation. We will create WCF proxy directly by using the WCF service interface.

### Problem it addresses
When we use WCF to create service oriented architecture, we cannot always add service reference. We need a mechanism to call the service methods without adding service reference to WCF service. Here in this article we will see how to call a two way WCF service method by creating proxy on the fly using ChannelFactory and later we will see how to call a two way WCF service method by using custom proxy and callback.

### Table of contents
- How to call a two way service method by creating proxy on the fly using ChannelFactory.
- How to call a two way service method by using custom proxy.

### Requirements
If you don’t know to create WCF service, then you need to have a look at this article [[Article][Beginner] Creating WCF Service and Self Hosting in a console application and Creating Proxy of net.tcp and net.pipe endpoint via Visual Studio.](http://simplebasics.net/.net/creating-wcf-service-and-self-hosting-in-a-console-application-and-creating-proxy-for-net-tcp-and-net-pipe-endpoint-with-visual-studio/)

### Using the code
As this article is a continuation of the article [[Article][Beginner] Creating WCF Service and Self Hosting in a console application and Creating Proxy of net.tcp and net.pipe endpoint via Visual Studio](http://simplebasics.net/.net/creating-wcf-service-and-self-hosting-in-a-console-application-and-creating-proxy-for-net-tcp-and-net-pipe-endpoint-with-visual-studio/), you can continue using the service code which is available as an attachment in that article (You can find it at the end of the article). We will be doing some refactoring to the project which is used in that article but bulk of the code remains the same.

### Re-factoring required to the project.
Now in the project, WCF interface and the implementing class remains in the same project. We are going to move the service interface to another project and will make the interface public.

Though WCF, when added service contract attribute, will make the interface public, we will mandate that interface is public because we will be referring to this interface from projects which won’t implement this interface as service but considers this as a simple interface.

Now create a class library project to the solution and name it as “ServiceFramework”. Now move the “IService” interface to this library project and make it public. Now to the “WcfService” project, add reference to the newly added project. Also add a reference to the “ServiceFramework” project from “WcfServiceHost”. Though we are using the service class name to create service host, WCF internally will try to refer to the interface which is been implemented by WCF service class.

### Client application.
Create one Windows/WPF application with three textboxes and two buttons. One buttons is to calculate Sum and other to calculate total. Two textbox will accept your input and one will display the result from WCF service. Now after you have created the application, it will look like the image below.

![WcfClientApplication]({{site.baseurl}}/images/WcfClientApplication.png)

Now to the client project, add reference to “ServiceFramework” project. This is because we need to have a reference to IService interface to call service methods.  With this, basic settings for your client application are ready now we will see how to call the service methods.

### 1. How to call a two way service method by creating proxy at runtime / on the fly using ChannelFactory.
Here we will use ChannelFactory to create a channel with which we can call the service methods. Let’s see how to do this. On to the button click action of the sum, add the code below.

{% highlight csharp %}
Binding binding = new NetTcpBinding();
String remoteAdress = "net.tcp://localhost:7550/IService";
ChannelFactory<IService> channelFactory = new ChannelFactory<IService>(binding, remoteAdress);
IService serviceChannel = channelFactory.CreateChannel();
int result = serviceChannel.sum(Convert.ToInt32(textBox1.Text), Convert.ToInt32(textBox1.Text));
textBox3.Text = result.ToString();
{% endhighlight %}
  
Now run the application and find this working. But before you need the servicehost running. Go to the bin folder of WcfServiceHost project and find the .exe double click on it and run it. Keep that executable running and come back to visual studio and run your client application.

**NOTE :** You need your service running before you run your client.
  
### 2. How to call a two way service method by using custom proxy.
Creating a custom proxy is simple. Just inherit it from ClientBase<TChannel> provided by System.ServiceModel. Here ClientBase Provides the base implementation used to create client object that can call services. And TChannel is the WCF interface. Create a class file into your client project and name it as ServiceProxy. Inherit it from ClientBase and WCF service interface. This class will act as the proxy for our WCF service.

{% highlight csharp %}
class ServiceProxy : ClientBase<IService>, IService
    {
    }
{% endhighlight %}
  
Now implement the class as shown below.

  {% highlight csharp %}
using System.ServiceModel;
using WcfService;
using System.ServiceModel.Channels;

namespace WcfClient
{
    class ServiceProxy : ClientBase<IService>, IService
    {
        public ServiceProxy(string endpointConfigurationName) :
            base(endpointConfigurationName)
        {
        }

        public ServiceProxy(string endpointConfigurationName, string remoteAddress) :
            base(endpointConfigurationName, remoteAddress)
        {
        }

        public ServiceProxy(string endpointConfigurationName, EndpointAddress remoteAddress) :
            base(endpointConfigurationName, remoteAddress)
        {
        }

        public ServiceProxy(Binding binding, EndpointAddress remoteAddress) :
            base(binding, remoteAddress)
        {
        }

        public int sum(int param1, int param2)
        {
            return Channel.sum(param1, param2);
        }

        public int Total(int param1, int param2)
        {
            return Channel.Total(param1, param2);
        }
    }
}
  {% endhighlight %}

Now let’s see how to call our proxy to call the service method. Now on to your button click action for your “total” button, add this code.

  {% highlight csharp %}
EndpointAddress endpointAddress = new EndpointAddress("net.tcp://localhost:7550/IService");
ServiceProxy proxy = new ServiceProxy(new NetTcpBinding(),endpointAddress);
int result = proxy.Total(Convert.ToInt32(textBox1.Text), Convert.ToInt32(textBox2.Text));
textBox3.Text = result.ToString();
 {% endhighlight %}
  
Now run your application and find it working. You need to have your service running before running your client application. Run the serviceHost application separately from visual studio. Once it is running, come back to visual studio and run your client application.