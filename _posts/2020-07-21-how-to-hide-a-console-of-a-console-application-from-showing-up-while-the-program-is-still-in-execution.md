---
layout: post
modified: '2015-06-07 10:24 +0530'
published: true
comments: true
title: >-
  How to hide a console of a console application from showing up while the
  program is still in execution
description: >-
  While I was working with WCF service application, I came across a situation
  where I am hosting my WCF service in a console application and I don't want
  the console application to show up when my service is up. I fixed it this by
  Pinvoke a call to FindWindow() to get a handle to your window and then call
  ShowWindow() to hide the window. I have created a sample code to show how I
  did this. It is as given below.
tags:
  - WCF
  - 'C#'
categories:
  - .Net
date: '2015-06-07 10:24 +0530'
---
### Introduction
While I was working with WCF service application, I came across a situation where I am hosting my WCF service in a console application and I don't want the console application to show up when my service is up. I fixed it this by Pinvoke a call to FindWindow() to get a handle to your window and then call ShowWindow() to hide the window. I have created a sample code to show how I did this. It is as given below.

{% highlight csharp %}
using System;
using System.Runtime.InteropServices;
using System.Threading;
namespace TestShowHideConsole
{
    internal class Program
    {
        [DllImport("user32.dll")]
        public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);

        [DllImport("user32.dll")]
        private static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);

        private static void Main(string[] args)
        {
            string consoleWindowTitle = Console.Title;
            IntPtr hWnd = FindWindow(null, consoleWindowTitle);
            if (hWnd != IntPtr.Zero)
            {
                ShowWindow(hWnd, 0); /* 0 = SW_HIDE */
            }
            Thread.Sleep(1000*10);/* will show your console after 10 second */
            if (hWnd != IntPtr.Zero)
            {
                ShowWindow(hWnd, 1); /*1 = SW_SHOWNORMA */
            }
            ConsoleKeyInfo consoleKeyInfo = Console.ReadKey();/* this will make the window wait till some key is pressed.*/
        }
    }
}
{% endhighlight %}

Here in this example, this will open up a console and it will be hidden for 10 seconds and will come up after that.

### How to hide a console application If it is running from another application.
If you are running the console application from another application and you want to hide the console window, then you can do like this. This will start your console application but the window will be hidden. You can see your application still running from the task manager. If you cant find your application running in the task manager, then some error is there in your application and you may need to debug it.

{% highlight csharp %}
Process p = new Process
        {
          StartInfo =  {
                       FileName = YourConsoleaApplicationFileHere,
                       Arguments = "Argument to console application if                     any.",
                       WindowStyle = ProcessWindowStyle.Hidden
/*Or you can use CreateNoWindow = true instead of WindowStyle*/
                       }
         };
p.Start();
{% endhighlight %}
