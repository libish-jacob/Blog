---
layout: post
modified: '2015-07-07 10:28 +0530'
published: true
comments: true
title: >-
  Cannot be processed at the receiver, due to a ContractFilter mismatch at the
  EndpointDispatcher
description: >-
  A "ContractFilter mismatch at the EndpointDispatcher" can happen because the
  client could not process the message because no contract claimed it. This can
  happen because of the following reasons:
tags:
  - WCF
  - 'C#'
categories:
  - .Net
date: '2015-07-07 10:28 +0530'
---
### ContractFilter mismatch at the EndpointDispatcher
A "ContractFilter mismatch at the EndpointDispatcher" can happen because the client could not process the message because no contract claimed it. This can happen because of the following reasons:

- Your client and the service have different contracts in between.
- Your client and service is using a different binding in between.
- Your client's and service's message security settings are not consistent in between.

Most of ContractFilter mismatch at the EndpointDispatcher bugs are because you use custom binding and your binding has extra tag which is missing either at client or service side. Have a close look at your binding, verify that it is same at both client and server side. The tags has to be same at client and server side, attribute value such as timeouts can vary at client and server. This is common and will not create issue. But if you have a tag say <ReliableSession /> at client side and is missing at server side, 'http://schemas.xmlsoap.org/ws/2005/02/rm/CreateSequence' cannot be processed at the receiver, due to a ContractFilter mismatch at the EndpointDispatcher can happen.
