---
layout: post
modified: '2014-07-23 15:59 +0530'
published: true
comments: true
title: >-
  There was an error while trying to serialize parameter fileStream. The
  InnerException message was Type System.IO.FileStream with data contract name
  FileStream: System.IO is not expected.
tags:
  - WCF
categories:
  - .Net
description: >-
  This fix is helpful if you are using WCF streaming and you got this exception
  while transferring huge file using data contract. If you are using streaming
  and the stream is transferred to or from WCF service, then you will get an
  exception like
---
## Introduction

This fix is helpful if you are using WCF streaming and you got this exception while transferring huge file using data contract. If you are using streaming and the stream is transferred to or from WCF service, then you will get an exception like

“There was an error while trying to serialize parameter http://tempuri.org/:fileStream. The InnerException message was 'Type 'System.IO.FileStream' with data contract name 'FileStream: http://schemas.datacontract.org/2004/07/System.IO' is not expected. Consider using a DataContractResolver or add any types not known statically to the list of known types - for example, by using the KnownTypeAttribute attribute or by adding them to the list of known types passed to DataContractSerializer.'.  Please see InnerException for more details.”

## How to fix this exception
Streaming in WCF has some limitations. And the limitation which we face here is that the parameter that we use in contract should be of type IXmlSerializable. Here as we are using data contract, it is not IXmlSerializable. i.e. WCF expects XmlSerializer to serialize and de-serialize messages. Data contract uses DataContractSerializer by default to serialize and de-serialize messages. Unfortunately there is no fix for this bug if you use data contract. You can fix this by using message contract.
