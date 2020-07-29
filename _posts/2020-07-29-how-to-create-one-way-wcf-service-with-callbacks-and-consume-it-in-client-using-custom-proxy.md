---
layout: post
modified: '2015-09-20 11:55 +0530'
published: true
comments: true
title: >-
  How to Create One way WCF service with Callbacks and consume it in client
  using custom proxy.
description: >-
  There are cases when an operation has no return value, and the client does not
  care about the success or failure of the invocation. To support this sort of
  fire and forget invocation, WCF offers one-way operations. In this article we
  will see how to create a one way service which responds to client via
  callback.
tags:
  - WCF
categories:
  - .Net
date: '2015-09-20 11:55 +0530'
---
### Introduction
There are cases when an operation has no return value, and the client does not care about the success or failure of the invocation. To support this sort of fire and forget invocation, WCF offers one-way operations. In this article we will see how to create a one way service which responds to client via callback.

### Background
In this article we will see how to create and call a one way call service which uses callbacks to reply back to client. In this article we will create a simple service with two methods, Sum and Total. Here the logic implemented in these methods will be simple, we will just take the parameter add it and will return. In this article we will see how to create a one way service, How to create callback contracts, and how to call the service from a custom proxy that we will be creating. Also this example won’t add reference to service but we will create a custom proxy which will call the service.

### Problem it addresses
These kinds of services are required when we don’t want client to wait for a reply from service or client is not interested in the reply from service method. When we have a service which has methods which is suppose to take more time to process the request and we don’t want the client to wait for the reply from service, we want a mechanism to fire request to service with a callback and continue. When service is done with the processing of the method which we requested, it will respond back to client using the callback.

### Table of contents
- How to create a one way service operation.
- How to create callback contract.
- How to create custom duplex proxy.
- How to call a one way service.

### Requirements
This article will be a more advanced continuation of [[Article] Creating WCF Service and Self Hosting in a console application and Creating Proxy for net.tcp and net.pipe endpoint with Visual Studio](http://simplebasics.net/.net/creating-wcf-service-and-self-hosting-in-a-console-application-and-creating-proxy-for-net-tcp-and-net-pipe-endpoint-with-visual-studio/). In this article we will be using self-hosting with a console application. If you are not sure of how to do it then you should have a look at the parent article.

### 1. How to create a one way service.
Configuring a one-way service contract is simple, just set the IsOneWay property of OperationContract to true. Add [OperationContract (IsOneWay=true)] to the method which you want to make one way. Once you added this, then WCF will take care of the rest. WCF will update the metadata with this info and client can call this method as a one way operation.



Here as this article is a continuation of  [[Article] Consuming self-hosted WCF service by Creating WCF net.tcp and net.pipe proxy at runtime and by custom proxy, without adding service reference](http://simplebasics.net/.net/consuming-self-hosted-wcf-service-by-creating-wcf-net-tcp-and-net-pipe-proxy-at-runtime-and-by-custom-proxy-without-adding-service-reference/)., we will add new interface to our existing application. Create a new service interface and name it as ISecondService.

Add your service methods signatures to this interface and decorate it with WCF service attributes. And to make it a one way service, set IsOneWay property of OperationContract to true for those methods which you need as one way. Now your service interface will look like as given below.

{% highlight csharp %}
[ServiceContract()]

public interface ISecondService
{
[OperationContract(IsOneWay=true)]
void sum(Guid commandId, int param1, int param2);

[OperationContract(IsOneWay = true)]
void Total(Guid commandId, int param1, int param2);
} 
{% endhighlight %}


Here we are using one extra parameter called “commandId”. This parameter is to map the callback logic that we will be adding later. This is not necessary but our example is oriented around this parameter so better keep it. Now create a service class and name it as SecondService and implement our new service interface as shown below.

{% highlight csharp %}
[ServiceBehavior()]
    public class SecondService : ISecondService
    {
        ISecondServiceCallback callbackObject;
        public SecondService()
        {
            callbackObject =
                OperationContext.Current.GetCallbackChannel<ISecondServiceCallback>();
        }

        public void sum(Guid commandId, int param1, int param2)
        {
            int sum = (param1 + param2);
            callbackObject.OnSumComplete(commandId, sum);
        }

        public void Total(Guid commandId, int param1, int param2)
        {
            int total = (param1 + param2);
            callbackObject.OnTotalComplete(commandId, total);
        }
}
{% endhighlight %}
  
Here when we say OperationContext.Current.GetCallbackChannel<ISecondServiceCallback>(), it will give the client side implementation of the callback class’s object which is registered with the proxy with which we will call our service. With this we can invoke the callback methods. We will see later in this article how to create and register the callback object with proxy. With this our one way service is complete. Now WCF will handle the rest by publishing it in metadata.
  
### 2. How to create callback contract.
If we are not interested in the reply from service, then by simply creating a one way service as explained above will suffice. But if you have designed your service as one way because your service method is a special case where it will reply only after a certain amount of time or say if the service is designed to respond to client on some event, then service has to have a mechanism to respond to client. With basic one way service, service cannot reply back to client. When a client call one way service, then the call will be added to service side buffer and client will be released even without waiting for the service method to fire. WCF will then process the method calls one by one from the buffer. But how will we reply back to client? WCF has provided callback contract with which we can register a callback which can be invoked with the result from the service. This way we can respond back to client. Let’s see how to achieve this. For this first we have to create a callback contract. Callback contract is nothing but an interface which will define the callback method signature. Let’s create a callback contract for our service methods. Create an interface and name it as ISecondServiceCallback and add two callback methods and decorate it with WCF attributes. Here we are using an extra parameter to the callback methods called “commanded” this is because we will be using this parameter in our callback logic. This parameter will map the callback in a dictionary which we will see later in this article. Remember, the callback interface is implemented at client side and at server side, you just need that interface.

  {% highlight csharp %}
public interface ISecondServiceCallback
    {
        [OperationContract(IsOneWay=true)]
        void OnSumComplete(Guid commandId, int sum);

        [OperationContract(IsOneWay = true)]
        void OnTotalComplete(Guid commandId, int Total);
    }
{% endhighlight %}
  
Now we have to register our callback contract with our service. For this, we have to set the CallbackContract attribute of the ServiceContract Attribute. Let’s see how.

{% highlight csharp %}
[ServiceContract(CallbackContract = typeof(ISecondServiceCallback))]
    public interface ISecondService
    {
        [OperationContract(IsOneWay=true)]
        void sum(Guid commandId, int param1, int param2);

        [OperationContract(IsOneWay = true)]
        void Total(Guid commandId, int param1, int param2);
}
{% endhighlight %}
  
Now implement the service and host it in console application as given in the article  [Article] Consuming self-hosted WCF service by Creating WCF net.tcp and net.pipe proxy at runtime and by custom proxy, without adding service reference. Also configure net.tcp and net.pipe endpoints as given in the same article. Now host the service in our host application as shown below.

  {% highlight csharp %}
ServiceHost sh2 = new ServiceHost(typeof(WcfService.SecondService));
sh2.Open();
Console.ReadKey();
sh2.Close();
  {% endhighlight %}

After this, your service is ready to be consumed.

NOTE: In our example, we need our service running before calling this service from client side.

### 3. How to create custom duplex proxy.
Here in our example we don’t add service reference to our service, instead we will create custom proxy and will call service using that. If you are not using callback and you just create a one-way service, then you don’t need a duplex proxy, just a simple proxy as given in article  [[Article] Consuming self-hosted WCF service by Creating WCF net.tcp and net.pipe proxy at runtime and by custom proxy, without adding service reference.](http://simplebasics.net/.net/consuming-self-hosted-wcf-service-by-creating-wcf-net-tcp-and-net-pipe-proxy-at-runtime-and-by-custom-proxy-without-adding-service-reference/)  will do. Let’s see how to create a duplex proxy which is required to call a one way service with callback. To create a Duplexproxy, inherit it from generic DuplexClientBase class with type equal to the type as your service interface. Also implement your service interface in this proxy and implement method to call methods from channel provided by DuplexClientBase class as shown below.

{% highlight csharp %}
internal class SecondServiceProxy : DuplexClientBase<ISecondService>, ISecondService 
    {
        public SecondServiceProxy(InstanceContext callbackInstance) : 
            base(callbackInstance)
    {
    }    

    public SecondServiceProxy(InstanceContext callbackInstance, string endpointConfigurationName) : 
            base(callbackInstance, endpointConfigurationName)
    {
    }    

    public SecondServiceProxy(InstanceContext callbackInstance, string endpointConfigurationName, string remoteAddress) : 
            base(callbackInstance, endpointConfigurationName, remoteAddress)
    {
    }    

    public SecondServiceProxy(InstanceContext callbackInstance, string endpointConfigurationName, EndpointAddress remoteAddress) : 
            base(callbackInstance, endpointConfigurationName, remoteAddress)
    {
    }

    public SecondServiceProxy(InstanceContext callbackInstance, Binding binding, EndpointAddress remoteAddress) : 
            base(callbackInstance, binding, remoteAddress)
    {
    }

   public void sum(Guid commandId, int param1, int param2)
        {
            Channel.sum(commandId, param1, param2);
        }

        public void Total(Guid commandId, int param1, int param2)
        {
            Channel.Total(commandId, param1, param2);
        }
    }
  {% endhighlight %}
  
  With this, our proxy is ready to use. Now we will see how to call service using this proxy.

### 4. How to call a one way service.
To call the service, we have to create an instance of the proxy that we created. Here the proxy constructor needs one callback instance. For this we have to implement our callback interface that we created for our service. Remember the only implementation of the callback interface is at client side.  Let’s see the callback interface implementation.

  {% highlight csharp %}
  class SecondServiceCallback : ISecondServiceCallback
    {
        IDictionary<Guid, Action<int>> callbackDictionary;
        public SecondServiceCallback()
        {
            callbackDictionary = new Dictionary<Guid, Action<int>>();
        }

        public void RegisterCallback(Guid commandId, Action<int> action)
        {
            callbackDictionary.Add(commandId, action);
        }

        public void OnSumComplete(Guid commandId, int sum)
        {
            Action<int> action;
            callbackDictionary.TryGetValue(commandId, out action);
            if (null != action)
            {
                action.Invoke(sum);
            }
        }

        public void OnTotalComplete(Guid commandId, int total)
        {
            Action<int> action;
            callbackDictionary.TryGetValue(commandId, out action);
            if (null != action)
            {
                action.Invoke(total);
            }
          }
     }
{% endhighlight %}
  
Now create the proxy instance and call the service.

{% highlight csharp %}
EndpointAddress ena = new EndpointAddress("net.tcp://localhost:7551/ISecondService");
Guid commandId = Guid.NewGuid();
SecondServiceCallback ob = new SecondServiceCallback();
  
ob.RegisterCallback(commandId, (total) =>
{
    textBox3.Text = total.ToString();
});

InstanceContext con = new InstanceContext(ob);
SecondServiceProxy proxy = new SecondServiceProxy(con, new NetTcpBinding(), ena);
proxy.Total(commandId, Convert.ToInt32(textBox1.Text), Convert.ToInt32(textBox2.Text));
 {% endhighlight %} 

