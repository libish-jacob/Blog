---
layout: post
modified: '2014-04-18 12:34 +0530'
published: true
comments: true
title: >-
  Visual Studio crashes while developing WPF XAML. Out of memory, taking too
  much memory to process
tags:
  - WPF
  - Visual Studio
description: >-
  You may have experienced an annoying behavior of visual studio while
  developing WPF XAML. Say for example Visual Studio consumes large amounts of
  memory and shows Not Responding message on a regular basis while designing
  XAML This is because of the XAML designer (Microsoft Visual Studio XAML UI
  Designer) which visual studio loads while editing XAML. There will be one for
  each XAML that you open for editing.
date: '2014-04-18 12:34 +0530'
categories:
  - IDE
---
## Introduction
You may have experienced an annoying behavior of visual studio while developing WPF XAML.
Say for example Visual Studio consumes large amounts of memory and shows Not Responding message on a regular basis while designing XAML
This is because of the XAML designer (Microsoft Visual Studio XAML UI Designer) which visual studio loads while editing XAML. There will be one for each XAML that you open for editing.

## How did I fix it:

While developing XAML, we rarely looks at the designer and we tend to rely on our expertise in writing XAML. If you are also the one who don’t rely on XAML designer, then you can ask visual studio to not to load designer and thereby saving a lot of memory thereby avoid visual studio crash because of out of memory. Below described are the steps which you have to follow to avoid loading XAML designer.

1. In visual studio, go to Tools –> Options menu,
2. Open the Text Editor node, then the XAML node,
3. Then select the Miscellaneous node; make sure that under the Default View heading there is a checkbox beside Always open documents in full XAML view.
4. Go to solution explorer -> Right click on any .xaml and say open with. Now select Source Code (Text) Editor and set it as default.

Restart visual studio and enjoy a new development experience.
Here if you look at the task manager, you still can see one xaml designer running (Microsoft Visual Studio XAML UI Designer). This is required by visual studio for compilation purpose so just ignore it.
