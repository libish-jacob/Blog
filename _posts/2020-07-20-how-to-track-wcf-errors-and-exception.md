---
layout: post
modified: '2015-06-07 16:21 +0530'
published: true
comments: true
title: How to track WCF errors and exception
description: >-
  By default Windows Communication Foundation (WCF) does not log messages. To
  enable message logging, you must add a trace listener to the
  System.ServiceModel.MessageLogging trace source and set attributes for the
  <messagelogging> element in the configuration file. Lets see how to enable WCF
  logging.  You can use WCF provided Configuration Editor Tool
  (SvcConfigEditor.exe) to configure these settings.
tags:
  - WCF
  - 'C#'
categories:
  - .Net
date: '2015-06-07 16:21 +0530'
---
This post is for those who are working with WCF and at times sulking because of errors which does not show up and who spends lots of time debugging what is going wrong with their service and client.

I was working with a WCF service and I came across a situation where my client application is trying to connect to service via proxy and client is waiting for the call to return and it never returns finally it times out. I was not having any clue what was happening. I had application level tracing which was not able to track anything. Finally I decided to  enable WCF built in tracing!.

### How to enable WCF Trace listener
By default Windows Communication Foundation (WCF) does not log messages. To enable message logging, you must add a trace listener to the System.ServiceModel.MessageLogging trace source and set attributes for the <messagelogging> element in the configuration file. Lets see how to enable WCF logging.  You can use WCF provided Configuration Editor Tool (SvcConfigEditor.exe) to configure these settings.

### Basic Setting
{% highlight csharp %}
<system.diagnostics>
  <sources>
      <source name="System.ServiceModel.MessageLogging">
        <listeners>
                 <add name="messages"
                 type="System.Diagnostics.XmlWriterTraceListener"
                 initializeData="c:\logs\messages.svclog" />
          </listeners>
      </source>
    </sources>
</system.diagnostics>

<system.serviceModel>
  <diagnostics>
    <messageLogging 
         logEntireMessage="true" 
         logMalformedMessages="false"
         logMessagesAtServiceLevel="true" 
         logMessagesAtTransportLevel="false"
         maxMessagesToLog="3000"
         maxSizeOfMessageToLog="2000"/>
  </diagnostics>
</system.serviceModel>
{% endhighlight %}

- maxSizeOfMessageToLog is the message size in memory . This size can differ from the actual length of the message string that is being logged, and in many occasions is bigger than the actual size. As a result, messages may not be logged. So consider this situation and set the value of maxSizeOfMessageToLog appropriately.
- logMessagesAtServiceLevel This boolean value is for logging messages at service level. Messages logged at this layer are about to enter (on receiving) or leave (on sending) user code. By default all messages at the service level are logged. Infrastructure messages (transactions, peer channel, and security) are also logged at this level, except for Reliable Messaging messages. On streamed messages, only the headers are logged. In addition, secure messages are logged decrypted at this level.
- logMessagesAtTransportLevel This boolean value is for logging messages at transport level. Messages logged at this layer are ready to be encoded or decoded for or after transportation on the wire. By default all messages at the transport layer are logged. All infrastructure messages are logged at this layer, including reliable messaging messages. On streamed messages, only the headers are logged. In addition, secure messages are logged as encrypted at this level, except if a secure transport such as HTTPS is used.

### Where to apply the log
You can apply the configuration on both service side and client side configuration file. You can analyse the log file which will be created after you run the service. Here if you are using the trace listener as-is then it will create a log file at "c:\logs\messages.svclog". you can change the file location as you like.

### Recommended Setting for debugging

When you are using logging in  debugging environment , and you are using WCF trace sources, set the switchValue to Information or Verbose, along with ActivityTracing. To enhance debugging, you should also add an additional trace source (System.ServiceModel.MessageLogging) to the configuration to enable message logging. Notice that the switchValue attribute has no impact on this trace source.

You can set the switchValue attribute to Information in all the previously mentioned cases, When you set the switchValue attribute to Information, it will log all messages. A recommended configuration is as shown below.

{% highlight csharp %}
<configuration>
 <system.diagnostics>
  <sources>
    <source name="System.ServiceModel" switchValue="Information,
 ActivityTracing"
            propagateActivity="true" >
      <listeners>
        <add name="xml"/>
      </listeners>
    </source>
    <source name="System.ServiceModel.MessageLogging">
      <listeners>
        <add name="xml"/>
      </listeners>
    </source>
    <source name="myUserTraceSource" switchValue="Information, ActivityTracing">
      <listeners>
        <add name="xml"/>
      </listeners>
    </source>
  </sources>
  <sharedListeners>
    <add name="xml"
         type="System.Diagnostics.XmlWriterTraceListener"
               initializeData="C:\logs\Traces.svclog" />
  </sharedListeners>
 </system.diagnostics>

 <system.serviceModel>
  <diagnostics wmiProviderEnabled="true">
      <messageLogging 
           logEntireMessage="true" 
           logMalformedMessages="true"
           logMessagesAtServiceLevel="true" 
           logMessagesAtTransportLevel="true"
           maxMessagesToLog="3000" 
       />
  </diagnostics>
 </system.serviceModel>
</configuration>
{% endhighlight %}

### Recommended setting for Production environment

When you are using logging in production environment, and you are using WCF trace sources, set the switchValue to Warning. If you are using the WCF System.ServiceModel trace source, set the switchValue attribute to Warning and the propagateActivity attribute to true. If you are using a user-defined trace source, set the switchValue attribute to Warning, ActivityTracing. If you do not fell like there will be any performance hit, you can set the switchValue attribute to Information in all the previously mentioned cases, When you set the switchValue attribute to Information, it will log all messages which will be a fairly large amount of data and this is why it should be avoided if you feel it will cause you performance hit. A recommended configuration is as shown below.

{% highlight csharp %}
<configuration>
 <system.diagnostics>
  <sources>
    <source name="System.ServiceModel" switchValue="Warning"
            propagateActivity="true" >
      <listeners>
        <add name="xml"/>
      </listeners>
    </source>
    <source name="myUserTraceSource"
            switchValue="Warning, ActivityTracing">
      <listeners>
        <add name="xml"/>
      </listeners>
    </source>
  </sources>
  <sharedListeners>
    <add name="xml"
         type="System.Diagnostics.XmlWriterTraceListener"
               initializeData="C:\logs\Traces.svclog" />
  </sharedListeners>
 </system.diagnostics>

<system.serviceModel>
  <diagnostics wmiProviderEnabled="true">
  </diagnostics>
 </system.serviceModel>
</configuration>
{% endhighlight %}

After enabling WCF level logging you will come to know what is wrong in your system. When you have the type of exception that you have, its much simpler to fix it. :)
