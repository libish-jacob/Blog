---
layout: post
modified: '2015-08-06 15:38 +0530'
published: true
comments: true
title: How to cancel a running task and to make sure it always cancels
description: >-
  Task class in .net library is extremely useful when you want to handle thread.
  Developers don’t have to create or maintain thread explicitly. Methods
  executed by a Task typically execute asynchronously and it uses background
  threads from the thread pool. There is no direct way of cancelling a task.
  .Net framework provides CancellationToken  which we can use to cancel a task
  with limitation. We will first look at how to cancel a task with cancellation
  token and later we will see in which all scenario this doesn't work and we
  will also see the workaround for this.
tags:
  - 'C#'
categories:
  - .Net
date: '2015-08-06 15:38 +0530'
---
## How to cancel a running task and to make sure it always cancels

In this article we will see how we can cancel a running task. Task class in .net library is extremely useful when you want to handle thread. Developers don’t have to create or maintain thread explicitly. Methods executed by a Task typically execute asynchronously and it uses background threads from the thread pool. There is no direct way of cancelling a task. .Net framework provides CancellationToken  which we can use to cancel a task with limitation. We will first look at how to cancel a task with cancellation token and later we will see in which all scenario this doesn't work and we will also see the workaround for this.


### Cancelling a task using CancellationToken
You can register an action to execute when a cancel is invoked on a cancellation token, To cancel a task we have to hook the thread on which the task executes the method and we have to call abort on that thread to stop the thread and there by cancelling the task as shown below.

{% highlight csharp %}
int DoWork(CancellationToken token)
{
  Thread t = Thread.CurrentThread;
  using (token.Register(t.Abort))
  {
    // CPU-bound work here
  }
}
{% endhighlight %}

You can find the complete code below. Here we have a generic method called ExecuteAsync which can execute any given method. Here TReturn is the type which we return from the method which we want to execute.

{% highlight csharp %}
namespace ConsoleApplicationTest
{
    using System;
    using System.Threading;
    using System.Threading.Tasks;

    public class AsyncTaskResult<TResult>
    {
        public TResult Result { get; set; }

        public Exception Exception { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Press any key to proceed!");
            Console.ReadLine();

            var canceller = new CancellationTokenSource();
            Action<AsyncTaskResult<bool>> act = (o) =>
            {
                // you will get the result as a callback here.
                bool result = o.Result;
                Exception exception = o.Exception;
            };
            ExecuteAsync(DoWork, act, canceller, null);
            Thread.Sleep(2000); // cancel the task after 2 seconds.
            canceller.Cancel();

            Console.WriteLine("Press any key Exit!");
            Console.ReadLine();
        }

        private static bool DoWork(object parameter)
        {
            try
            {
                // make it sleep for 60 seconds.
                Thread.Sleep(1000 * 60);
            }
            catch (ThreadAbortException ex)
            {
                /* Do all the cleanup required. 
                 This is very important as this is the only chance to cleanup resources used by the task.
                 */
            }

            return true;
        }

        private static void ExecuteAsync<TReturn>(Func<object, TReturn> action, Action<AsyncTaskResult<TReturn>> callback, CancellationTokenSource canceller, object parameter)
        {
            Exception thrownException = null;
            TReturn returnValue = default(TReturn);

            Func<object, TReturn> work = o =>
            {
                Thread currentThread = Thread.CurrentThread;

                /* register abort as the action for cancellation tocken. This method will be invoked when we call cancell on cancellation tocken */
                using (canceller.Token.Register(
                    () =>
                    {
                        try
                        {
                            currentThread.Abort();
                        }
                        catch (Exception)
                        {
                            // silence it. Cancelable operations and callbacks registered with the token should not throw exceptions.
                        }
                    }))
                {
                    return action(parameter);
                }
            };

            Task<TReturn> startNew = Task.Factory.StartNew(work, null, canceller.Token);

            startNew.ContinueWith(
                t =>
                {
                    try
                    {
                        AggregateException aggregateException = t.Exception;
                        if (aggregateException != null)
                        {
                            if (null != aggregateException.InnerException)
                            {
                                thrownException = aggregateException.InnerException;
                            }
                            else
                            {
                                thrownException = aggregateException;
                            }
                        }
                        else
                        {
                            returnValue = t.Result;
                        }
                    }
                    catch (Exception)
                    {
                        // silence it.
                    }
                    finally
                    {
                        var ret = new AsyncTaskResult<TReturn>
                        {
                            Result = returnValue,
                            Exception = thrownException
                        };

                        callback.Invoke(ret);
                    }
                });
        }
    }
}
{% endhighlight %}

### Problem using this way of cancelling a task.
Calling an abort on a thread will not guarantee an abort. If there is any blocking call, say for example, if the method is waiting for a call-back from a socket communication, then calling an abort on that thread won’t stop the thread from execution. So even if you try to call a cancel on a cancellation token, it will have no effect on the thread.
Aborting a thread is not a recommended way of stopping a thread. It will leave the app domain in an unsafe state.
### How to cancel a blocking task?
We will have to rely on abort in this situation as-well but in a different way. Here, to achieve this, we will have to deploy another task! This sounds like a weird way of solving this? We don’t want a failing cancel call!. Here we will deploy another task and the abort will be called on that. The complete code on how to cancel a blocking task can be found below.


{% highlight csharp %}
namespace ConsoleApplicationTest
{
    using System;
    using System.Runtime.CompilerServices;
    using System.Threading;
    using System.Threading.Tasks;

    /// <summary>
    /// Wrapper class which is used to wrap the result and exception from the task.
    /// </summary>
    /// <typeparam name="TResult"></typeparam>
    public class AsyncTaskResult<TResult>
    {
        public TResult Result { get; set; }

        public Exception Exception { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Press any key to proceed!");
            Console.ReadLine();

            var canceller = new CancellationTokenSource();
            Action<AsyncTaskResult<bool>> act = (o) =>
            {
                // you will get the result as a callback here.
                bool result = o.Result;
                Exception exception = o.Exception;
            };
            ExecuteAsync(DoWork, act, canceller, null);
            Thread.Sleep(2000); // cancel the task after 2 seconds.
            canceller.Cancel();

            Console.WriteLine("Press any key Exit!");
            Console.ReadLine();
        }

        private static bool DoWork(object parameter)
        {
            try
            {
                // make it sleep for 60 seconds.
                Thread.Sleep(1000 * 60);
            }
            catch (ThreadAbortException ex)
            {
                /* Do all the cleanup required. 
                 This is very important as this is the only chance to cleanup resources used by the task.
                 */
            }

            return true;
        }

        private static void ExecuteAsync<TReturn>(
            Func<object, TReturn> action,
            Action<AsyncTaskResult<TReturn>> callback,
            CancellationTokenSource canceller,
            object parameter)
        {
            Exception thrownException = null;
            TReturn returnValue = default(TReturn);

            Func<object, TReturn> work = o =>
            {
                bool cancelled = false;

                using (canceller.Token.Register(
                    () =>
                    {
                        // just mark cancelled as true when cancel is invoked on this task.
                        cancelled = true;
                    }))
                {
                    Func<object, TReturn> subWork = op =>
                    {
                        Thread currentThread = Thread.CurrentThread;
                        using (canceller.Token.Register(
                            () =>
                            {
                                try
                                {
                                    // mark abort as the action to execute when cancel is invoked for this task.
                                    currentThread.Abort();
                                }
                                catch (Exception)
                                {
                                    /* silence it. Cancelable operations and callbacks registered with the token should not throw exceptions.*/
                                }
                            }))
                        {
                            return action(parameter);
                        }
                    };

                    Task<TReturn> task = Task.Factory.StartNew(subWork, null, canceller.Token);
                    while (!task.IsCompleted && !cancelled)
                    {
                        /* we will wait for half a second to poll again. We could also reduce this timeout.*/
                        Thread.Sleep(500);
                    }

                    if (cancelled)
                    {
                        /* As we are releasing the thread to kill itself, we have to raise a thread abort exception explicitly.*/
                        throw new TaskCanceledException();
                    }

                    if (task.Exception != null)
                    {
                        /* throw the exception if any while executing the sub task.
                             * Task exception will be aggregate exception, we are here interested in the real exception, so pass the inner exception. 
                             * ExceptionDispatchInfo is a feature in .net 4.5. This lets you capture an exception and re-throw it without changing the stack-trace:
                            */
                            ExceptionDispatchInfo.Capture(task.Exception.InnerException).Throw();
                    }

                    // return the result if the task executed successfully.
                    return task.Result;
                }
            };

            Task<TReturn> startNew = Task.Factory.StartNew(work, null, canceller.Token);

            startNew.ContinueWith(
                t =>
                {
                    try
                    {
                        AggregateException aggregateException = t.Exception;
                        if (aggregateException != null)
                        {
                            // grab the exception thrown by the task if any.
                            if (null != aggregateException.InnerException)
                            {
//ExceptionDispatchInfo will help you capture the exception stack.
thrownException = ExceptionDispatchInfo.Capture(aggregateException.InnerException).SourceException;
                            }
                            else
                            {
                                thrownException = aggregateException;
                            }
                        }
                        else
                        {
                            returnValue = t.Result;
                        }
                    }
                    catch (Exception)
                    {
                        // silence it.
                    }
                    finally
                    {
                        /*wrap the exception and result into a single object to pass it back to caller.*/
                        var ret = new AsyncTaskResult<TReturn> { Result = returnValue, Exception = thrownException };

                        callback.Invoke(ret);
                    }
                });
        }
    }
}
{% endhighlight %}
