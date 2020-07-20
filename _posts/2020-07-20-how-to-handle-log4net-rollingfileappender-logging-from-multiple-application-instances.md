---
layout: post
modified: '2017-08-10 15:51 +0530'
published: false
comments: true
title: >-
  How to handle log4net RollingFileAppender logging from multiple application
  instances
description: >-
  Logging is one of the very important factor in any application. You require it
  to analyse what when and how things went wrong. It serves as the log of the
  application. But when you have multiple instance of the same application
  running in parallel, then you don’t have the purpose served properly by the
  logging mechanism as it will be writing all the log entry into the same file
  and the log will be cluttered. Here in this post we will discuss how to handle
  this typical situation when you have multiple instances running at the same
  time and log4net writes all the logs into the same file.
tags:
  - 'C#'
  - Log4Net
categories:
  - .Net
date: '2017-08-10 15:51 +0530'
---
## How to handle log4net RollingFileAppender logging from multiple application instances

Logging is one of the very important factor in any application. You require it to analyse what when and how things went wrong. It serves as the log of the application. But when you have multiple instance of the same application running in parallel, then you don’t have the purpose served properly by the logging mechanism as it will be writing all the log entry into the same file and the log will be cluttered. Here in this post we will discuss how to handle this typical situation when you have multiple instances running at the same time and log4net writes all the logs into the same file.

### Possible solutions
- Let log4net write each log entry with the process id.
- We can have log4net write the log in to different file appended by a process id or anything similar.
- Log it into a central location with process Id or something similar.
- Log everything into same file but append an identifier to each log entry.

Here we are going to look at a solution which can be called as a combination of 1&2. We will have log4net write to a different file if another instance is running. If not then it will use the same log file. From management perspective it would be nice to have only one file because then we can have a log rotate and keep the file under manageable size. when we have another instance running then it will be logged into a file with the time appended to the file name. This way we can handle logging by second instance. Let us now see how this is achieved using log4net.

### Drawback of this solution:
When you have an application which has high probability of running multiple instances, then you are going to have many log files which will not be rotated or handled by the logging mechanism. So choose this solution if the situation where you have multiple instances running in parallel are rare.

### To achieve this we will be using two concepts.
- Pattern Converter in log4net
- Inter process synchronization technique.

### Pattern Converter in log4net
Log4net has this pattern converter which you can use in the configuration. Here we will make use of it to handle the file name. Let us see how to achieve this. I will not be discussing the complete configuration of log4net and the below shown code is just a snippet from the configuration. This is as shown below.

{% highlight csharp %}
<appender name="FileAppender" type="log4net.Appender.RollingFileAppender">
    <file type="log4net.Util.PatternString">
      <converter>
        <name value="fileName" />
        <type value="SampleApp.FilePatternConverter,SampleApp" />
      </converter>
      <conversionPattern value="${APPDATA}\LogFolder\%fileName" />
    </file>
</appender>
{% endhighlight %}

Here we have defined a converter for the file name. Conversion pattern is the format for the file name. Here "${APPDATA} specifies the system defined application folder and it has nothing to do with the pattern. %fileName is an identifier which redirects to the converter with name as filename. We have defined a converter with name as filename and value as SampleApp.FilePatternConverter,SampleApp. Here SampleApp.FilePatternConverter specifies the namespace and the class name of our converter class and SampleApp is the assembly name where we can find the converter. Now let us see how the converter class looks like. It is as shown below.

{% highlight csharp %}
namespace SampleApp
{
    using System.IO;

    public class FilePatternConverter: log4net.Util.PatternConverter
    {
        protected override void Convert(TextWriter writer, object state)
        {
            String filename = “logfile.log”;
            var logFileName = LogFileNameConverter.GetLogFileName(filename);
            writer.Write(logFileName);
        }
    }
}
{% endhighlight %}

The text writer will then replace the pattern in the configuration. To be precise it will replace %fileName from the conversionPattern with the log file name. Here the core of the logic is in another file and lets have a look at it. It is as shown below.

{% highlight csharp %}
public class LogFileNameConverter
    {
        private const string FileName = "application.log";
        private static readonly Mutex ApplicationSyncLock;
        private static readonly bool LockAcquired;
        private static string dateToAppend;
        
        static LogFileNameConverter()
        {
            try
            {
// register an application exit event and release the mutex lock.
                Application.Current.Exit += CurrentExit;
                bool created;
                ApplicationSyncLock = new Mutex(false, "ApplicationMutexName", out created);
                LockAcquired = ApplicationSyncLock.WaitOne(2000);
            }
            catch (AbandonedMutexException)
            {
// here will will release the abandoned mutex.
                ApplicationSyncLock.ReleaseMutex();

                //lets acquire the lock.
                LockAcquired = ApplicationSyncLock.WaitOne(2000);
            }
            catch (Exception)
            {
                LockAcquired = false;
            }
        }
 
        public static string GetLogFileName(string logFileName)
        {
            if (string.IsNullOrEmpty(logFileName))
            {
                logFileName = FileName;
            }
 
            if (LockAcquired)
            {   
                return logFileName;
            }
            else
            {
                var extension = Path.GetExtension(logFileName);
                var fileName = Path.GetFileNameWithoutExtension(logFileName);
                if (string.IsNullOrEmpty(dateToAppend))
                {
                    Regex rgx = new Regex("[^a-zA-Z0-9]");
                    dateToAppend = rgx.Replace(DateTime.Now.ToString(CultureInfo.InvariantCulture), "-");
                }
 
                fileName = (fileName + "-instance-at-" + dateToAppend + extension).Replace(' ', '-');
                return fileName;
            }
        }
 
        private static void CurrentExit(object sender, ExitEventArgs e)
        {
            ApplicationSyncLock.ReleaseMutex();
        }
 }
{% endhighlight %}

Here we use named Mutex as an inter-process synchronization or signalling. We acquire a mutex lock and keep it locked till the application exits. The new instance will not be able to acquire a lock and will move ahead and create a new logfile name with datetime append to the filename. Here we chose Mutex because it has a mechanism where we can handle the abandoned Mutex which results from an application which had the lock got terminated from task manager. When a new instance of application is launched we can handle this abandoned mutex.
This is all you need and when you run multiple instance, it will start to write to a different file. If you are running it in serial then you have all the log in one file and log rotate will make sure that you have only one file with the specified size.
Now if you like to pass the log file name from the configuration itself, then also it is possible. Let’s see how this is achieved. The converter should have property attribute defined and this will be available in the converter. Let’s see how this looks like. This is as shown below.

{% highlight csharp %}
<appender name="FileAppender" type="log4net.Appender.RollingFileAppender">
      <file type="log4net.Util.PatternString">
        <converter>
          <name value="fileName">
          <type value="SampleApp.FilePatternConverter,SampleApp">
          <property>
            <key value="FileName">
            <value value="app.log">
            </value></key>
</property></type></name></converter>
        <conversionpattern value="${APPDATA}\LogFolder\%fileName">
      </conversionpattern></file>
</appender>
{% endhighlight %}

Here we have defied a property with key as “FileName” and value as “app.log”. This will be available in the converter class as shown below.

{% highlight csharp %}
protected override void Convert(TextWriter writer, object state)
{
            string fileName = this.Properties["FileName"] as string;
            var logFileName = LogFileNameConverter.GetLogFileName(fileName);
            writer.Write(logFileName);
}
{% endhighlight %}

### 4.Log everything into same file but append an identifier to each log entry
Now that we have seen this solution, let us see how "Log everything into same file but append an identifier to each log entry" can be achieved. For simplicity I am using the exact same converter.

We can achieve this by modifying the layout of log by using the below mentioned configuration.

{% highlight csharp %}
<layout type="log4net.Layout.PatternLayout">
<converter>
       <name value="fileName">
       <type value="SampleApp.FilePatternConverter,SampleApp">
       <property>
         <key value="FileName">
         <value value="app.log">
       </value></key></property>
    </type></name></converter>
    <param name="ConversionPattern" value="%fileName %d [%t] %-5p [%C %F %L] - %m%n" />
</layout>
{% endhighlight %}

