---
layout: post
modified: '2014-07-23 16:36 +0530'
published: true
comments: true
title: >-
  [Solved] There was an error reading from the pipe: Unrecognized error 109
  (0x6d)
description: >-
  If you are using netNamedPipeBinding for your communication then you may get
  an exception like “There was an error reading from the pipe: Unrecognized
  error 109 (0x6d).”
tags:
  - WCF
categories:
  - .Net
date: '2014-07-23 16:36 +0530'
---
## Introduction
If you are using netNamedPipeBinding for your communication then you may get an exception like “There was an error reading from the pipe: Unrecognized error 109 (0x6d).”

## How to fix this exception
1. If you are using netNamedPipeBinding and your service went down while the operation is in progress, you may get this exception. So make sure that your service is up and running.

2. If you are using Streaming in WCF, and your binding is netNamedPipeBinding, you may get this exception. To fix this exception, make sure that you are not using DataContract to transfer your data. Also make sure that you are not violating any requirement implied by WCF streaming. Streaming has some limitation. If you are using streaming then compare your implementation with the given article. [Article](https://libish-jacob.github.io/Blog/.net/how-to-transfer-upload-download-huge-large-file-to-and-from-a-wcf-service-using-streaming/) How to transfer / Upload / Download huge / large file to and from a WCF service using streaming.
