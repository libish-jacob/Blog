---
layout: post
modified: '2020-02-26 13:45 +0530'
published: true
comments: true
title: >-
  Process.WaitForExit with a timeout will not be able to collect the output
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
## process.WaitForExit with a timeout will not be able to collect the output message from process when executed using Process class in .net
In this post we will evaluate a scenario where we try to execute a process and are trying to collect the output from the process but the output from a process is not collected by the calling process. If you are in this page, then you must have experienced the similar issue where the process is tested to collect the output from the process which we executed but it fails intermittently while collecting the output from the process. You might have done everything right by collecting the output from the process in an asynchronous manner but still it fails to collect the output in between. If you are using process.WaitForExit with a timeout, then it is the culprit. process.WaitForExit with a timeout is known to create issue when we have some parallelism in place and we are trying to execute say the same process multiple times in parallel or many different processes in parallel. The way to get out of this is to wait for the process without a timeout. This may have practical difficulties because if we don’t have a timeout, then there can be situation where we may wait forever for the process to exit. The way to get out of this is to run the process and wait for it without a timeout but timeout via the thread which is executing it. The below example will show you how to implement this.

In this example we will make use of the task library in .Net. Tasks are as you know a wrapper for the thread pool class and all the tasks are executed by a thread in the thread-pool. Even when we are asking the Task library to start the task, it must wait till the thread is available in the thereadpool. If all available threads are busy, then the task must wait until one is available in the pool (There are certain exceptions tough. We will not discuss those here.). So, we must be careful with the timeout. If the task does not start immediately then we will have issue with the timeout. Task may timeout after the specified timeout. To counter this, we are making use of the reset events. The reset events will tell us when did the thread started executing and from that point onwards, we will start waiting for the task to timeout. 

The example below is trying to run the same process multiple times and is trying to collect and print the output from the process. The code is written for example purpose. 


