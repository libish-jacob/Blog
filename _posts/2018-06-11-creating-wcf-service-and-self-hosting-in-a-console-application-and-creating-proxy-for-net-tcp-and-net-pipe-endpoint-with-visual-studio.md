---
layout: post
modified: '2014-07-14 15:37 +0530'
published: true
comments: true
title: >-
  Creating WCF Service and Self Hosting in a console application and Creating
  Proxy for net.tcp and net.pipe endpoint with Visual Studio
tags:
  - WCF
categories:
  - .Net
date: '2014-07-14 15:37 +0530'
description: >-
  In this article, we will see how to create a simple WCF service. Then we will
  see how to host this WCF service in a console application with so called
  self-hosting. After that we will see how to expose service metadata via MEX
  endpoint. After that we will see how to create a proxy for the WCF service
  which has net.tcp and net.pipe endpoints using visual studio 2010.
---
## Introduction
 In this article, we will see how to create a simple WCF service. Then we will see how to host this WCF service in a console application with so called self-hosting. After that we will see how to expose service metadata via MEX endpoint. After that we will see how to create a proxy for the WCF service which has net.tcp and net.pipe endpoints using visual studio 2010.
 
## Background
 This article is all about creating, hosting and adding service reference to WCF service in client application. We will start with creating a WCF service as a class library, and then we will host it in a console application. We will expose net.pipe and net.tcp endpoints for our service. In WCF, when we add service reference via visual studio, visual studio will automatically create proxy for WCF service at client side. With basic settings, we cannot add service reference to WCF service which exposes net.tcp and net.pipe endpoints via visual studio. We will make necessary changes on WCF service so that visual studio can detect service and can add service reference to our client project.

In this article, instead of creating a client project, we will be using WCF provided test client. But it will be explained here how to add service reference to WCF service which is hosted in a console application. In this article, we will also see some WCF provided tools which will come in handy while working with WCF. One such tool is the WCF provided test client.

## Table of contents
1. How to create a WCF service.
2. How to host a WCF service in a console application.
3. How to use WCF Service Configuration Editor Tool.
4. How to expose service metadata by exposing MEX end point.
5. How to add service reference to a WCF service hosted in a console application which exposes either net.tcp or net.pipe or both.

To begin with, let’s create a WCF service which will have only two methods, Sum and Total. We are not interested in the logic inside that method. We will keep this method as simple as possible.

### 1. How to create a WCF service.
Create a class library project and name it as WcfService. Here the name is of no relevance. You can keep any name as you like. Now to the project, add a public interface file and name it as IService. Add two method signatures to the interface as shown below.

{% highlight csharp %}
int sum(int param1, int param2); 
int Total(int param1, int param2);
{% endhighlight %}

Now create a class file and name it as Service. Now inherit this class from interface IService and implement the methods. Keep the logic simple and return sum of param1 and param2 from both these methods just to make it simple.

Now we will see how to convert our simple interface to WCF service. To convert a simple interface to WCF service, we have to decorate it with WCF attributes. Here we will see how to add basic attributes to make our interface to behave as a WCF service.

A WCF endpoint will have three parts, Address, Contract and Binding. And it is our interface, once decorated with WCF attributes, which will be the contract for our WCF service. It is not necessary to have an interface which will behave as the contract. You can have a class without inheriting from an interface to behave as contract by decorating it with WCF attributes. Here in our example just like any standard WCF service, will be using Interface to behave as WCF contract.

To make an interface a WCF contract, decorate it with “ServiceContract” attribute. And to expose methods inside the interface as operation of that service, we have to decorate the method with “OperationContract“ attribute. You can find these attributes in System.ServiceModel. For this you have to add reference to System.ServiceModel into your project. With this much change your simple WCF service is ready.

{% highlight csharp %}
using System.ServiceModel;

namespace WcfService {
    [ServiceContract()]
    public interface IService {
        [OperationContract()]
        int sum(int param1, int param2);
        
        [OperationContract()]
        int Total(int param1, int param2);
    }
}
{% endhighlight %}

Now we will see how to host this service in a console application.

### 2. How to host a WCF service in a console application
The WCF service class cannot exist in void. Every WCF service must be hosted in a windows process called the host process. Let’s create this host process as a console application. Let’s create a console application and name it as WcfServiceHost. Now when you create the project, you will get one default class file called Program.cs. Let’s write code inside the Main method of this file to write code to host our service. But before, add reference to System.ServiceModel and also a reference to the WcfService project which we created earlier.

Now create a serviceHost instance inside the Main method and call Open() method on that ServiceHost instance as shown below.

{% highlight csharp %}
using System.ServiceModel;
namespace WcfServiceHost {
    class Program {
        static void Main(string[] args) {
            ServiceHost sh = new ServiceHost(typeof(WcfService.Service));
            sh.Open();
            Console.ReadKey();/*This will help the console application to remain up. If not, the console application will go down and the service host will die.*/

            sh.Close();
        }
    }
}
{% endhighlight %}


Now the coding part of the WCF service is over. Now we have to configure endpoints for our WCF service. Though it is possible to create endpoints via code, we will do all these configurations via application config file. Here we will use WCF provided “WCF Service Configuration Editor” tool to add service endpoints to config file. For this we need a configuration file, Add new item as application configuration file to WcfServiceHost project and name it as App.config.

### 3. How to use WCF Service Configuration Editor Tool.
Having some limitation in Visual Studio 2010, we have to select WCF Service Configuration Editor from tools menu. Second time onwards you can right click on the configuration file and can select this tool to edit the config file.

Right click on the App.config file and say “Edit WCF Configuration”. This will open the configuration in WCF Service Configuration Editor.

Now we have to add a new service. For this, in the configuration section tree view, you can see “Services” node. Right click on that and say “New Service”.
![AddingService.png]({{site.baseurl}}/images/AddingService.png)

Now in the name section as shown in the image above, browse to the WcfService.dll which you can find in the bin/debug or bin/release folder of your service project which we created and select the Service class. Now this will create a service. Now to the service we have to add end points as shown in the image below.

![AddingEndpoint.png]({{site.baseurl}}/images/AddingEndpoint.png)
And to the host section, add base addresses for net.tcp and net.pipe as shown in the image.

![AddingBaseAdress.png]({{site.baseurl}}/images/AddingBaseAdress.png)

Now save this file. Here our basic service is ready to use. But as we want to add reference to our service from client project via visual studio, this is not enough. The service needs to expose a metadata on each endpoint. By default WCF exposes metadata on http endpoint but for other protocol it is not enabled by default. As we are using net.tcp and net.pipe, we have to configure this metadata endpoint to expose this metadata so that visual studio can add WCF service via service reference.

### 4. How to expose service metadata by exposing MEX end point.
There is a standard way of publishing metadata over a special endpoint called metadata exchange endpoint referred to as MEX endpoint. To add metadata endpoint, we need to create metadata end points. You only have to expose any one MEX endpoint. If you have HTTP end point defined in your service, then you don’t have to do anything at all. WCF by default will expose service metadata via MEX endpoint for HTTP. But if you don’t have HTTP endpoint defined at your service side, then you have to create one MEX endpoint. You don’t have to create MEX endpoint for all the endpoints any one will do. But to understand how it is made, we will create MEX endpoint for both net.tcp and net.pipe endpoints. Here we will create two MEX endpoints, one for net.tcp and other for net.pipe as we have two endpoints configured for our service. From the client side, we will be using any one of these endpoints say net.pipe or net.tcp but for flexibility we are configuring two endpoints net.tcp and net.pipe. If you want you can avoid any one of these end point. But in this example we will configure both these endpoints.

To add MEX endpoint, follow the steps that we did for creating net.pipe or net.tcp endpoint but select mexTcpBinding and mexNamedPipeBinding like shown in figure.
![AddingTcpMexEndpoint.png]({{site.baseurl}}/images/AddingTcpMexEndpoint.png)

The MEX endpoint supports an industry standard for exchanging metadata represented in WCF by the IMetadataExchange interface. WCF can have the service host automatically provide the implementation of IMetadataExchange and expose the metadata exchange endpoint. All you need to do is designate the address and the binding to use and add the service metadata behaviour. All you have to do to have the host implement the MEX endpoint for your service is include the serviceMetadata tag in the behaviour. If you do not reference the behaviour, the host will expect your service to implement IMetadataExchange. Let’s see how add service behaviour.
![AddingMexEndpoint.png]({{site.baseurl}}/images/AddingMexEndpoint.png)

Now we need to add this service behaviour in our service. Let’s see how to do this. Image below shows how to do this.
![AddingServiceMetadataBehaviourInServiceEndpoint.png]({{site.baseurl}}/images/AddingServiceMetadataBehaviourInServiceEndpoint.png)

### 5. How to add service reference to a WCF service hosted in a console application which exposes either net.tcp or net.pipe or both.

Now we will see how to add service reference to the service that we created via visual studio 2010. Here we will not create a project just to show adding a service reference. Instead we will use WCF provided test client tool to show this. You can find this tool at C:\Program Files\Microsoft Visual Studio 10.0\Common7\IDE\ WcfTestClient.exe. Run this application. Now on to the left sidebar of this test client, you can see a tree view which says “My Service Projects”. Now on to “My Service Projects” right click and say “Add Service” now on to the text-box which comes up, add the MEX endpoint address. MEX endpoint for net.pipe in our example will be net.pipe://localhost/7550/MEX. Before clicking “ok” make sure that the WcfServiceHost console application which hosts our WCF service is running. If it is not running then Adding reference will fail. After clicking “ok” you can see the service and the methods it exposes. Now double click on the method and you will be provided with a window where you can invoke the method. Try this.

Now adding service reference it a client project in visual studio is simple, just add service reference and give the MEX endpoint address instead of the service endpoint and visual studio should be able to add reference to WCF service and will create ready made proxy for you.

