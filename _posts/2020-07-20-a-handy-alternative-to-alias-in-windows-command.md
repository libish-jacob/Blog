---
layout: post
modified: '2018-08-05 15:27 +0530'
published: true
comments: true
title: A handy alternative to Alias in windows command
tags:
  - Dos Command
categories:
  - Operating System
description: >-
  A recent encounter in windows server where a need for a folder alias was
  unavoidable, we made a discovery that unlike linux, folder alias is hard to
  achieve in windows command for windows version less than windows 10. Though it
  is achievable, it is hard to achieve when the target directory path is
  dynamic.
date: '2018-08-05 15:27 +0530'
---
A recent encounter in windows server where a need for a folder alias was unavoidable, we made a discovery that unlike linux, folder alias is hard to achieve in windows command for windows version less than windows 10. Though it is achievable, it is hard to achieve when the target directory path is dynamic. This lead to a search for the alternative to deploy in this scenario. The scenario was when we have to use the editbin.exe and its LARGEADDRESSAWARE where we discovered that LARGEADDRESSAWARE take only a particular character length for the input file and if there are more then it will trim it. This was leading to an error.

The alternative was to CD into the folder and execute the command and get out of the folder. And we have something called as setlocal which help us achieve this by scoping the step in and out of a folder. Setlocal will help us scope the working directory and reverts the working directory when the scope is over. The below snippet shows how we can achieve this. I was using this in VisualStudio postbuild event.

{% highlight csharp %}
setlocal
cd
cd $(TargetDir)
cd
IF EXIST C:\masm32\bin\editbin.exe (
  C:\masm32\bin\editbin.exe /LARGEADDRESSAWARE $(TargetFileName)
) 
endlocal
{% endhighlight %}
