---
layout: post
modified: '2014-07-23 15:56 +0530'
published: true
comments: true
title: >-
  How to transfer / Upload / Download huge / large file to and from a WCF
  service using streaming.
description: >-
  In this article we will see how to transfer a huge file to and from WCF
  services efficiently. WCF supports streaming of data. We will use streaming to
  transfer huge file efficiently to and from a WCF service. This article is all
  about transferring file using streaming transfer mode in WCF and it does not
  enforce you to use streaming always, because of its limitations.
tags:
  - WCF
categories:
  - .Net
date: '2014-07-23 15:56 +0530'
---
## Introduction
 In this article we will see how to transfer a huge file to and from WCF services efficiently. WCF supports streaming of data. We will use streaming to transfer huge file efficiently to and from a WCF service. This article is all about transferring file using streaming transfer mode in WCF and it does not enforce you to use streaming always, because of its limitations.

## Background
 To transfer huge files or data, streaming is the effective and efficient method. By default WCF uses buffered transfer mode. Here in this article, to get an efficient system which does not eat up resources, we will use Streaming transfer mode of WCF. But streaming transfer mode of WCF has some limitations. We will be covering those limitations in this article. In this article we will see how to implement the operation contract which uses the built in data type which is stream and then we will see how to use data contract and message contract to transfer file and if not possible then we will analyze why it is not possible to transfer file using any of this methods.
 
## Problem it addresses

 When we use the default WCF transfer mode, which is Buffered transfer mode, WCF will load or buffer the entire file content into memory and then will transfer. Here if the file or data is huge, then it is not possible to load the entire data to the memory. In such cases it is impossible to transfer such huge data. To handle such situation, we can use the streaming transfer mode of WCF. In fact, you should consider streaming when you have a reasonably big file say 2MB file to transfer. This is because if you have many clients which will access the same service at the same time, then the memory requirement will be the product of file size and the number of client accessing the service simultaneously. We can’t select streaming only based on this fact. Streaming has some limitations.

## Limitations in Streaming Transfer mode of WCF

Using the streamed transfer mode causes the run time to enforce additional restrictions. Operations that occur across a streamed transport can have a contract with at most one input or output parameter. That parameter corresponds to the entire body of the message and must be a Message, a derived type of Stream, or an IXmlSerializable implementation. Having a return value for an operation is equivalent to having an output parameter.
Some WCF features, such as reliable messaging, transactions, and SOAP message-level security, rely on buffering messages for transmissions. Using these features may reduce or eliminate the performance benefits gained by using streaming. To secure a streamed transport, use transport-level security only or use transport-level security plus authentication-only message security.

SOAP headers are always buffered, even when the transfer mode is set to streamed. The headers for a message must not exceed the size of the MaxBufferSize transport quota. So when you take the decision to go for streaming, you should consider these facts.

## Table of contents


1. Creating operation contract which returns Stream.
2. Creating operation which returns DataContract to stream file.
3. Creating operation which returns MessageContract to stream file.

## Requirements
 There are certain configurations that we have to set before using streaming. The first thing that we need to do is to change the default transfer mode of WCF. We have to configure the default transfer mode of WCF to Streaming. To do this set the transferMode="Streamed".You can set this property only if you are using any of these binding.
 
- BasicHttpBinding 
- NetTcpBinding 
- NetNamedPipeBinding 
- WebHttpBinding

{% highlight css %}
<bindings>
 <netNamedPipeBinding>
   <binding name="NamedPipeBinding" transferMode="Streamed" />
   </netNamedPipeBinding>
</bindings>
{% endhighlight %}

After this we have to decide on the maximum file size which we allow in our application to be transferred. It is good to limit the size of the file which is transferred. Once you have decided on the size, you have to set maxReceivedMessageSize property of the binding to the size which you have decided. This property sets the Max size of the received message which is processed by the binding. This property value is in bites. You can set it up to long.MaxValue which is 9,223,372,036,854,775,807. Here in our example we are setting it to 4294967294 which is around 4 GB.

{% highlight css %}
<bindings>
 <netNamedPipeBinding>
 <binding name="NamedPipeBinding" transferMode="Streamed" maxReceivedMessageSize="4294967294" />
 </netNamedPipeBinding>
</bindings>
{% endhighlight %}

Now we have to set the maxBufferSize of the binding. This property sets the Max number of bytes used to buffer the incoming message in the memory. Here this is the max size of the message which is processed at a time. SOAP headers are always buffered, even when the transfer mode is set to streamed. The headers for a message must not exceed the size of the MaxBufferSize transport quota. Here in our example we are sending 64k chunks of data at a time so we are setting it to 65536 which is 64K. Here this is the default setting of WCF so we don’t have to change it explicitly. If you decide to change this value, then you have to change this value.

{% highlight css %}
<bindings>
 <netNamedPipeBinding>
   <binding name="NamedPipeBinding" transferMode="Streamed" maxReceivedMessageSize="4294967294" maxBufferSize="65536" />
 </netNamedPipeBinding>
</bindings>
{% endhighlight %}

As in our example we will be using upload and download features, we have to set these properties both at service and client side.

## Using the code
We will be using MessageContract in our example. Lets create our message contract before moving further. Our message contract class, which we will be using in our code will look like as shown below

{% highlight css %}
[MessageContract]
public class FileUploadMessage {
   [MessageHeader(MustUnderstand = true)]
   public string Filename;
   
   [MessageBodyMember(Order = 1)]
   public Stream FileByteStream;
}
{% endhighlight %}

### 1. Creating operation contract which returns Stream Using Stream in contract to upload file.

Your service side code will be like as shown below.

{% highlight css %}
public Stream DownloadFile(string fileName) {
/* This is your contract*/
/* _basePath is the directory path where you will be storing the file */

string filePath = Path.Combine(_basePath, fileName);
FileStream stream = new FileStream(filePath, FileMode.Open);
return stream;
}
{% endhighlight %}

And at the client side your code will be as shown below. Remember this is a pseudo code. Please optimize it while using it in your code.

{% highlight css %}
string baseDir = @"D:\Test";
string downloadedFileName = "downloadedFestFile.txt";
/* here we are creating proxy at runtime. If you are using proxy after adding service reference or by creating your own proxy, make sure the properties explained aove are set properly.*/

NetNamedPipeBinding binding = new NetNamedPipeBinding();
binding.TransferMode = TransferMode.Streamed;
binding.SendTimeout = TimeSpan.MaxValue;
binding.MaxReceivedMessageSize = long.MaxValue;

EndpointAddress adress = new EndpointAddress("net.pipe://localhost/IFileTransfer");
ChannelFactory<IFileTransfer> channelFactory = new ChannelFactory<IFileTransfer>(binding, adress);
IFileTransfer fileTransferProxy = channelFactory.CreateChannel();
string serverFileName = "testFile.txt";
string path = System.IO.Path.Combine(baseDir, serverFileName);
string downloadedFilePath = System.IO.Path.Combine(baseDir, downloadedFileName);
using (Stream fs = fileTransferProxy.DownloadFile(serverFileName)) {                
using (FileStream outStream = new FileStream(downloadedFilePath, FileMode.OpenOrCreate))  {
  const int bufferSize = 65536; // 64K
  Byte[] buffer = new Byte[bufferSize];
  int bytesRead = fs.Read(buffer, 0, bufferSize);
  while (bytesRead > 0)  {
  outStream.Write(buffer, 0, bytesRead);
  bytesRead = fs.Read(buffer, 0, bufferSize);
  }
  }
 }
{% endhighlight %}
  
### 2. Creating operation which returns DataContract to stream file using DataContract in contract to stream file.

If you try to use data contract in your contract where you use Stream as one of the data members to transfer file, then it will throw exception. This is because of the limitation with streaming in WCF. And the limitation which we face here is that the parameter that we use in contract should be of type IXmlSerializable. Here as we are using data contract, it is not IXmlSerializable. i.e. WCF expects XmlSerializer to serialize and de-serialize messages. Data contract uses DataContractSerializer by default to serialize and de-serialize messages. So we can’t use data contract to pass stream.

### 3. Creating operation which returns MessageContract to stream file using  MessageContract to stream file.
When you are using MessageContract to stream file, your service side contract code is as shown below  

{% highlight css %}
public void TransferFileWithMessageContract(FileUploadMessage fileStream) {
            SaveFile(fileStream.Filename, fileStream.FileByteStream);
        }
        
private void SaveFile(string filename, Stream fileStream) {
            string serverFileName = Path.Combine(_basePath, filename);
            try {
                using (FileStream outfile = new FileStream(serverFileName, FileMode.Create)) {
                    const int bufferSize = 65536; // 64K
                    Byte[] buffer = new Byte[bufferSize];
                    int bytesRead = fileStream.Read(buffer, 0, bufferSize);
                    while (bytesRead > 0) {
                        outfile.Write(buffer, 0, bytesRead);
                        bytesRead = fileStream.Read(buffer, 0, bufferSize);
                    }
                }
            }
            catch (Exception) {
                throw;
            }
        }
{% endhighlight %}

And on the client side...

{% highlight css %}
string file = textBox1.Text;
//NetTcpBinding binding = new NetTcpBinding();
NetNamedPipeBinding binding = new NetNamedPipeBinding();
binding.TransferMode = TransferMode.Streamed;
binding.SendTimeout = TimeSpan.MaxValue;
EndpointAddress adress = new EndpointAddress("net.pipe://localhost/IFileTransfer");
ChannelFactory<IFileTransfer> channelFactory = new ChannelFactory<IFileTransfer>(binding, adress);
IFileTransfer fileTransferProxy = channelFactory.CreateChannel();
using (FileStream fs = new FileStream(file, FileMode.Open, FileAccess.Read, FileShare.Read)) {
 FileUploadMessage msg = new FileUploadMessage();
 msg.Filename = "UploadFile.txt";
 msg.FileByteStream = fs;
 fileTransferProxy.TransferFileWithMessageContract(msg);
 /* OR if you are adding service reference using visual studio then you cab cal   the service as below. */
 //ServiceReference1.FileTransferClient proxy = new      ServiceReference1.FileTransferClient();
 //proxy.TransferFileWithMessageContract(msg.Filename, fs);
}
{% endhighlight %}
  
### 4. Downoad file using MesageContract.
For download contract, your service side code looks like as shown below.

{% highlight css %}
public FileUploadMessage DownloadFileWithMessageContract(FileUploadMessage fileMessage) {
            Stream fileStream = GetFileStream(fileMessage.Filename);
            FileUploadMessage upload = new FileUploadMessage();
            upload.FileByteStream = fileStream;
            upload.FileByteStream = fileStream;
            return upload;
}

private Stream GetFileStream(string fileName) {
            string filePath = Path.Combine(_basePath, fileName);
            FileStream stream = new FileStream(filePath, FileMode.Open);
            return stream;
        }
{% endhighlight %}

And at the client side code should look like as shown below.

{% highlight css %}
//download with message contract
string baseDir = @"D:\Test";
string downloadedFileName = "downloadedFestFile.txt";

NetNamedPipeBinding binding = new NetNamedPipeBinding();
binding.TransferMode = TransferMode.Streamed;
binding.SendTimeout = TimeSpan.MaxValue;
binding.MaxReceivedMessageSize = long.MaxValue;

EndpointAddress adress = new EndpointAddress("net.pipe://localhost/IFileTransfer");
ChannelFactory<IFileTransfer> channelFactory = new ChannelFactory<IFileTransfer>(binding, adress);
IFileTransfer fileTransferProxy = channelFactory.CreateChannel();
string serverFileName = "testFile.txt";
string downloadedFilePath = System.IO.Path.Combine(baseDir, downloadedFileName);

FileUploadMessage message = new FileUploadMessage();
message.Filename = serverFileName;
message.FileByteStream = FileStream.Null;

FileUploadMessage ob = fileTransferProxy.DownloadFileWithMessageContract(message);
DownloadFile(downloadedFilePath, ob.FileByteStream);

  private static void DownloadFile(string downloadedFilePath, Stream fs)
    {
     using (FileStream outStream = new FileStream(downloadedFilePath, FileMode.OpenOrCreate))
       {
          const int bufferSize = 65536; // 64K
          Byte[] buffer = new Byte[bufferSize];
          int bytesRead = fs.Read(buffer, 0, bufferSize);
          while (bytesRead > 0)
           {
             outStream.Write(buffer, 0, bytesRead);
             bytesRead = fs.Read(buffer, 0, bufferSize);
           }
       }
   }
{% endhighlight %}

### 5. Download file using simple Stream.
Your service side code when you are using simple stream to transfer file is as shown below.

{% highlight css %}
public Stream DownloadFile(string fileName) {
    return GetFileStream(fileName);
 }

private Stream GetFileStream(string fileName) {
   string filePath = Path.Combine(_basePath, fileName);
   FileStream stream = new FileStream(filePath, FileMode.Open);
   return stream;
}
{% endhighlight %}

## Point of interest
Possible exception when you try to stream data.
1. The maximum message size quota for incoming messages (65536) has been exceeded. To increase the quota, use the MaxReceivedMessageSize property on the appropriate binding element.
2. This can happen if the max size of the message or file exceed the size which we set for the property MaxReceivedMessageSize in the binding. Set it to more than the size of the file which you are streaming.
