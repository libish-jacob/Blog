---
layout: post
modified: '2015-10-21 20:34 +0530'
published: true
comments: true
title: >-
  How to run multiple WCF services under one website in IIS 8 where each service
  is running on separate app-domain
description: >-
  Starting with IIS 7, we can have IIS hosting application which can run on
  separate App-domain. Running application on separate App-domain is required
  when we need application to work in isolation i.e. to prevent applications in
  one application pool from affecting applications in another application pool
  on the server. An application can have several virtual directories, and each
  one will be served by the same App-domain as the application to which they
  belong.
tags:
  - WCF
categories:
  - Web Server
  - .Net
date: '2015-10-21 20:34 +0530'
---
### Introduction
Starting with IIS 7, we can have IIS hosting application which can run on separate App-domain. Running application on separate App-domain is required when we need application to work in isolation i.e. to prevent applications in one application pool from affecting applications in another application pool on the server. An application can have several virtual directories, and each one will be served by the same App-domain as the application to which they belong. In most of the cases it should be fine to run an application in the default app-domain created by IIS. But on special scenarios say for example, if you want your application to be served by a different version of the run-time, then you can create an app-domain by specifying the run-time version and can use that to serve the application. Or if you want your applications to work in complete isolation then also you can do the same. Here in this post we will see how to host multiple WCF service under one website where each service is running in isolation with each on separate app-domain.

### Hosting WCF service using separate app-domain.
  An application is an object important to the server at run-time. Every website object in IIS must have an application and every application must have at least one virtual directory where each virtual directory points to a physical directory path. When you create a website object in IIS, you are creating a default application. An application can have several virtual directories. To know how you can host a WCF service in default application in IIS, you can refer to the article [[Article] How to host a WCF service in IIS 8.](http://simplebasics.net/web%20server/how-to-host-a-wcf-service-in-iis-8/) Let us now examine how we have created a website object with default application.

![adding website object]({{site.baseurl}}/images/Adding-website-object.png)

Here in the IIS, in connections section, on Sites, when we right click and say “Add website” it opens up a website object(default application) creation wizard like as shown in the image above. Here “Site name” refers to the name of the site(application) which you want. “Physical directory” should point to the root directory of the website. Just for the demo purpose we will point the physical directory to an empty folder. It could also be a root folder of one WCF service where you have the .svc file. And in the section where it is asking for the “Application pool” is where we have to select the application pool or so called app-domain. Here at the moment we only have a default app domain so we will leave it as is. And if you say ok, then the website object is ready. Now what we have done here is we have created a website object, we have created a default application, we have created a virtual directory which will point to the physical directory as specified in “Physical directory” and the default application will now run in an app domain specified by us in the “Application pool” section. Now we will see how we can create another app-domain.

### Creating Application Pool(App-domain)
In IIS, on the connection section, if you expand the server node, you can see “Application Pool” node. On the “Application Pools” node if you right-click and say “Add Application Pool” it will open up a window as shown below. Give a name for your new app-domain, this could be any unique name. and select the version of run-time which you like the application to run with and say Ok. Now your new app-domain is ready to use. 

![creating app domain]({{site.baseurl}}/images/Creating-AppDomain.png)

### Adding new application object
Now on to the website object which we just created, right click and say Add Application, it opens up a wizard to create an application object under the website object. Now in this wizard, Alias is the name which we are going to give to the application and this name is also going to be the part of the url of the application so give it a proper name. The Physical Path is where your root folder for your wcf service where you have the .svc file. Now the Application pool section is where you can select your App-domain which you created just before.

![creating application]({{site.baseurl}}/images/creating-application.png)

![Adding application]({{site.baseurl}}/images/Adding-Application.png)

Now when you browse the .svc file from the application folder, IIS will then use the App-domain which you have assigned for the application to serve the request. In this way you can create more app domain and more WCF services which will then work in isolation.
