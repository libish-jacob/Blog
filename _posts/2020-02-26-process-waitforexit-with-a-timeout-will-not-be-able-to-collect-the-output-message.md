---
layout: post
modified: '2020-02-26 13:45 +0530'
published: true
comments: true
title: >-
  Process WaitForExit with a timeout will not be able to collect the output
  message.
tags:
  - 'C#'
categories:
  - .Net
description: >-
  If you are using process.WaitForExit with a timeout, then it is the culprit.
  process.WaitForExit with a timeout is known to create issue when we have some
  parallelism in place and we are trying to execute say the same process
  multiple times in parallel or many different processes in parallel.
date: '2020-02-26 13:45 +0530'
---
In this post we will evaluate a scenario where we try to execute a process and are trying to collect the output from the process but the output from a process is not collected by the calling process. If you are in this page, then you must have experienced the similar issue where the process is tested to collect the output from the process which we executed but it fails intermittently while collecting the output from the process. You might have done everything right by collecting the output from the process in an asynchronous manner but still it fails to collect the output in between. If you are using process.WaitForExit with a timeout, then it is the culprit. process.WaitForExit with a timeout is known to create issue when we have some parallelism in place and we are trying to execute say the same process multiple times in parallel or many different processes in parallel. The way to get out of this is to wait for the process without a timeout. This may have practical difficulties because if we don’t have a timeout, then there can be situation where we may wait forever for the process to exit. The way to get out of this is to run the process and wait for it without a timeout but timeout via the thread which is executing it. The below example will show you how to implement this.

In this example we will make use of the task library in .Net. Tasks are as you know a wrapper for the thread pool class and all the tasks are executed by a thread in the thread-pool. Even when we are asking the Task library to start the task, it must wait till the thread is available in the thereadpool. If all available threads are busy, then the task must wait until one is available in the pool (There are certain exceptions tough. We will not discuss those here.). So, we must be careful with the timeout. If the task does not start immediately then we will have issue with the timeout. Task may timeout after the specified timeout. To counter this, we are making use of the reset events. The reset events will tell us when did the thread started executing and from that point onwards, we will start waiting for the task to timeout. 

The example below is trying to run the same process multiple times and is trying to collect and print the output from the process. The code is written for example purpose. 

{% highlight csharp %}


    private void Button_Click(object senderItem, RoutedEventArgs eventItem)
    {
      var arguments = "Test";
      
      var Program = @".\ConsoleApp1.exe";
      bool openCMD_Window = false;

      //lets run the same process 50 times in parallel.
      for (int i = 0; i < 50; i++)
      {
        Task.Factory.StartNew(() =>
        {
          // lets make use of this event to start waiting.
          AutoResetEvent ae = new AutoResetEvent(false);          
          string Output = string.Empty;
          /*The process has an internal sleep of 50 seconds.*/
          var task = Task.Factory.StartNew(() => Process(Program, arguments, openCMD_Window, ae, out Output));

          //lets wait until the thread starts.
          ae.WaitOne();
          if (task.Wait(TimeSpan.FromSeconds(60)))
          {
            Console.WriteLine(Output);
          }
          else
          {
            Console.WriteLine("Process timed out....");
          }
        });
      }
    }

    public void Process(string Program, string Arguments, bool OpenCommandWindow, AutoResetEvent ae, out string OutPut)
    {
      /*This is important since we are running on a task and the thread may not start immediately. We have to wait right after this thread has started.*/
      ae.Set();
      
      using (Process process = new Process())
      {
        if (Arguments != null)
        {
          process.StartInfo.FileName = "cmd";
          // For reading the files with special characters. 
          process.StartInfo.Arguments = "/C \"" + Program + " " + Arguments + "\"";
        }
        else
        {
          process.StartInfo.FileName = Program;
        }

        process.StartInfo.UseShellExecute = false;
        process.StartInfo.RedirectStandardError = !OpenCommandWindow;
        process.StartInfo.CreateNoWindow = !OpenCommandWindow;
        process.StartInfo.RedirectStandardOutput = !OpenCommandWindow;

        StringBuilder output = new StringBuilder();

        Action<object, DataReceivedEventArgs> dataCallback = (sender, e) =>
        {
          try
          {
            output.AppendLine(e.Data);
          }
          catch (ObjectDisposedException)
          {
              // It can happen and is ok. It can happen if there is still data and we timed out and the even fired before we unsubscribe.
            }
        };
        DataReceivedEventHandler handler = new DataReceivedEventHandler(dataCallback);
        process.OutputDataReceived += handler;

        process.Start();
        var processId = process.Id;

        if (!OpenCommandWindow)
        {
          process.BeginOutputReadLine();
        }

        process.WaitForExit();

        // unsubscribe it to avoid object disposed exception.
        process.OutputDataReceived -= handler;
        OutPut = output.ToString();
        if (string.IsNullOrWhiteSpace(OutPut))
        {
          /*This piece of code is here for demo purpose. If the output is empty, then pop up a notification.*/
          MessageBox.Show("Empty output");
        }
      }
    }
{% endhighlight %}

The project code is available at the github page [ProcessOutputCollection](https://github.com/libish-jacob/ProcessOutputCollection "ProcessOutputCollection")
