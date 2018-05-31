---
layout: post
modified: '2013-06-31 13:18 +0530'
published: true
comments: true
title: How to install and configure Varnish on Linux
description: >-
  Varnish is an HTTP accelerator or so called reverse proxy. Varnish is a type
  of proxy server that retrieves resources on behalf of a client from one or
  more servers. These resources are then returned to the client as though they
  originated from the server or servers themselves. Varnish then caches those
  pages whose expiry header is set to some future date. This means those pages
  which won’t change until that date.
categories:
  - Web Server
date: '2013-06-31 13:18 +0530'
tags:
  - Apache
---
## Introduction
  Varnish is an HTTP accelerator or so called reverse proxy. Varnish is a type of proxy server that retrieves resources on behalf of a client from one or more servers. These resources are then returned to the client as though they originated from the server or servers themselves. Varnish then caches those pages whose expiry header is set to some future date. This means those pages which won’t change until that date. Mostly static pages in a site will have expiry header so the next time when a client requests a page, varnish will look at the header to determine that the page has not expired. If not expired, then it will then serve that page without contacting the web server. This way, load on the server will be less. All static pages will be served by varnish without the knowledge of webserver.

After installing and configuring varnish, varnish will listen to the port where the client send request. Real web server will now sit as a backend for varnish, if varnish is not able to find the page in its cache, then it will request the backend webserver to get the page and varnish will serve that page to requesting client and at the same time it will cache it and next time it will serve the cached page.

## How to install varnish:
First of all stop the webserver if it is running. Here the server which I am running is Apache so the command to stop apache is as below.

{% highlight csharp %}
Service httpd stop
{% endhighlight %}

Now install varnish using yum.

{% highlight csharp %}
Yum install varnish
{% endhighlight %}

Now here in this example we are going to run varnish on default webserver port i.e. 80 and apache on port 8080. This is because a user doesn’t care who serves the page. Here pages will be served by varnish and varnish will intern request apache if the page is not cached. You are free to change the port numbers to any available port if you really want to change it.

{% highlight csharp %}
Configure webserver to listen on port 8080.
{% endhighlight %}

By default a webserver is configured to listen to port 80. But as we are going to run varnish on this port, we will have to change this default setting of webserver so that it start to listen to a different port number. As we are going to configure our webserver to listen to port 8080, below is how you could do that.

Find the apache config file httpd.conf. It can be found at /etc/httpd/conf/
Now we have to edit this file and change the default port number from 80 to 8080.

After changing the default port number, start apache and verify that it is running on port 8080. You can test this simply by requesting the web page by appending :8080 to the address.

Eg: http://www.testsite:8080/

{% highlight csharp %}
service httpd start
{% endhighlight %}

Now our web server will be running on port 8080. So we have to tell varnish that backend i.e. Apache is listening on port 8080. For that we have to enable the below lines in the varnish configuration file. This file can be found at /etc/varnish/default.vcl.

Now open this file and uncomment the below line. If this line is not available then append this piece of code on to the file.

{% highlight csharp %}
 backend default {
   .host = "127.0.0.1";
   .port = "8080";
 }
{% endhighlight %}

Now varnish is configured to call your backend webserver which listens to port 8080. Now we have to start varnish so that it can serve web pages. Now here we have to start varnish. Now here there is one more dilemma. Where will varnish store its cached pages? It is possible to keep the cache pages in memory or on disk. If you are running a memory critical webserver, then you can store it in disk but this will slow down the time which varnish takes to serve the pages. If you are not really in such a need, then go for memory caching. You have to use the below command if you want to start varnish in a mode where it stores the cache page in memory.

{% highlight csharp %}
varnishd -f /etc/varnish/default.vcl -s malloc,50M -T 127.0.0.1:2000 -a 0.0.0.0:80 -p thread_pool_min=500 -p thread_pool_max=4000 -p thread_pool_add_delay=2
{% endhighlight %}

Now here the cached pages will be stored in the memory. Here varnish will run with 50MB space for cached pages. You can change this to whichever size you need. Choosing a size greater than 1G will not be advisable. You should keep this size minimum by calculating the total size of the webpages. 512M is sufficient enough for a decently big website. Optionally you can store all cached pages to a file by starting varnish with the command as shown below.

{% highlight csharp %}
varnishd -f /etc/varnish/default.vcl -s file,/var/www/cacheground/varnish,1G -T 127.0.0.1:2000 -a 0.0.0.0:80 -p thread_pool_min=500 -p thread_pool_max=4000 -p thread_pool_add_delay=2
{% endhighlight %}

You can use the below commands to find the status of varnish.

- varnishstat
- varnishhist
- varnishlog
- varnishtop
- varnishsizes

{% highlight csharp %}
varnishadm –T localhost:2000 ban.url “^/$”
{% endhighlight %}

Varnish Launch Command Explanation:

Varnishd –> Command

-f –> file location of VCL file

-s –> Backend storage specification. By default the storage is “file” we change it to “malloc” so that the information will be stored in memory

-T –> Telnet listen address and port. Host is set to localhost and port is some random port e.g. 2000

-a –> HTTP listen address and port. Host is this server’s IP and listening port is 80

-p –> Set parameter for service launch.

There are few essential performance based parameters to be set while launch they are as follows

§  thread_pool_min is the minimum number of threads for each thread pool

§  thread_pool_max is the maximum total number of threads

§  thread_pool_add_delay – Reducing the add_delay lets you create threads faster which is essential - specially at startup - to avoid filling up the queue and dropping requests

The cli_timeout acts a check to see if the worker thread is still alive. You should keep this value in between 2-10 seconds. The intention behind the relatively short timeout is to be able to kill and restart the worker process in a few seconds.

## Configure boot startup for Varnish Server

For the Varnish server to start automatically when the server reboots, we need to update the “rc.local” file in the server instance. The process for the same is as follows,

{% highlight csharp %}
vim /etc/rc.local
{% endhighlight %}

Update these lines below in the file

/usr/local/sbin/varnishd  -f /usr/local/etc/varnish/default.vcl -s malloc,1G -T 127.0.0.1:2000 -a 10.64.43.79:80 -p thread_pool_min=500 -p thread_pool_max=4000 -p thread_pool_add_delay=2

Now save the changes made and you are done configuring varnish. Varnish will now add some header to the pages which it serves; this can be detected by checking the mime type. There are websites which will help you to detect this if you don’t want to spend time detecting the mime type. You can search for such web sites.
