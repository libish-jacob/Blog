---
layout: post
modified: '2014-07-23 16:39 +0530'
published: true
comments: true
title: '[Solved] The .Net Framing mode being used is not supported'
description: >-
  Here we will see how to fix the exception, the .Net Framing mode being used is
  not supported by 'net.tcp://....' See the server logs for more details.
tags:
  - WCF
categories:
  - .Net
date: '2014-07-23 16:39 +0530'
---
## Introduction
Here we will see how to fix the exception, the .Net Framing mode being used is not supported by 'net.tcp://....' See the server logs for more details.

## How to fix this exception
The .Net Framing mode being used is not supported by 'net.tcp://....' See the server logs for more details. This exception occurs when we change the TransferMode of tcp communication either at service side or client side to Streamed and the other side still uses the default mode which is Buffered.

The TCP three way handshake protocols use a Framing mode to transfer packets by default. When we change this behaviour, we have to make sure that the communication happens properly between client and server. If you use Streamed mode then the system expects the communication happens to and fro from service as streamed. If you are not indented to use so and you want only client to stream message and server to behave default then you can use TransferMode.StreamedRequest or TransferMode.StreamedResponse.
