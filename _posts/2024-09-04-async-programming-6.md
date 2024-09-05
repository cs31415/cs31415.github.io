---
layout: post
category: blog
title: Async Programming Part 6
permalink: /blog/async-programming-6
description: Async Programming Part 6
image: async6.jpg
---

![Async Programming](../img/async6.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@timwilson7?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Tim Wilson</a> on <a href="https://unsplash.com/photos/brown-bison-on-gray-asphalt-road-during-daytime-rIC-q1ds6dM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a></span>
  

### Notes on async usage

Now, armed with knowledge of internals (see parts [1](/blog/async-programming-1), [2](/blog/async-programming-2), [3](/blog/async-programming-3), [4](/blog/async-programming-4), [5](/blog/async-programming-5)), we focus on some usage topics in this and the next few posts. 

#### Don't use `.Result`

Calling `.Result` on an async method is a synchronous, blocking call. Calling `.Result` on an async method is a synchronous, blocking call. Blocking negates the benefit of async code by tying up the very thread that the async state machine so mightily labors to release. Consider the example below.

```csharp
static async Task Main(string[] args)
{
  var activity = GetActivityAsync().Result;
  Console.WriteLine($"{activity}");
}

static async Task<string> GetActivityAsync()
{
  var uri = "https://www.boredapi.com/api/activity";
  var response = await _httpClient.GetAsync(uri);
  var content = response.Content;
  
  var responseJson = await content.ReadAsStringAsync();
  
  if (!string.IsNullOrEmpty(responseJson))
  {
      dynamic jObj = JObject.Parse(responseJson);
      var activity = jObj.activity?.ToString();
      return activity;
  }

  return null;
}
```

Looking into the framework code for `Task.Result` below, we see that `Thread.SpinWait` is eventually called, which is an implementation of an adaptive [spin wait](https://en.wikipedia.org/wiki/Busy_waiting) algorithm, which tries to balance responsiveness and CPU usage. This implementation can call `Thread.Sleep(1)` which can block the thread or `Thread.Sleep(0)` and `Thread.Yield()` which can consume CPU cycles.   

The async method's continuation is scheduled to run on the same synchronization context as the method's caller (see [part 3](blog/async-programming-3) for more on synchronization context). A synchronization context only allows one unit of work to execute at a time. So, if the caller is already executing (because it is doing a blocking wait), the continuation can't run on the same context. And the caller is blocked waiting for the continuation to complete. So we have a deadlock. This can happen in frameworks that use a synchronization context, such as Windows Forms or ASP.NET (non-core). 

Even if we disable the synchronization context using `.ConfigureAwait(false)`, it can result in thread starvation issues since threads are tied up waiting on IO.

A note about exception handling when using `.Result`. Looking at `GetExceptions` below, we see that any exceptions that might arise from the async task are wrapped in an `AggregateException`:
```csharp
public TResult Result
{
  get
  {
    if (!base.IsWaitNotificationEnabledOrNotRanToCompletion)
    {
      return m_result;
    }
    return GetResultCore(waitCompletionNotification: true);
  }
}

internal TResult GetResultCore(bool waitCompletionNotification)
{
  if (!base.IsCompleted)
  {
    InternalWait(-1, default(CancellationToken));
  }
  if (!base.IsCompletedSuccessfully)
  {
    ThrowIfExceptional(includeTaskCanceledExceptions: true);
  }
  return m_result;
}

internal void ThrowIfExceptional(bool includeTaskCanceledExceptions)
{
	Exception exceptions = GetExceptions(includeTaskCanceledExceptions);
	if (exceptions != null)
	{
		UpdateExceptionObservedStatus();
		throw exceptions;
	}
}

private AggregateException GetExceptions(bool includeTaskCanceledExceptions)
{
	Exception ex = null;
	if (includeTaskCanceledExceptions && IsCanceled)
	{
		ex = new TaskCanceledException(this);
		ex.SetCurrentStackTrace();
	}
	if (ExceptionRecorded)
	{
		return m_contingentProperties.m_exceptionsHolder.CreateExceptionObject(calledFromFinalizer: false, ex);
	}
	if (ex != null)
	{
		return new AggregateException(ex);
	}
	return null;
}

internal bool InternalWait(int millisecondsTimeout, CancellationToken cancellationToken)
{
  return InternalWaitCore(millisecondsTimeout, cancellationToken);
}

private bool InternalWaitCore(int millisecondsTimeout, CancellationToken cancellationToken)
{
  if (IsCompleted)
  {
    return true;
  }
  bool result = 
  (millisecondsTimeout == -1 && 
  !cancellationToken.CanBeCanceled && 
  WrappedTryRunInline() && 
  IsCompleted) || 
  SpinThenBlockingWait(millisecondsTimeout, cancellationToken);

  return result;
}

private bool SpinThenBlockingWait(int millisecondsTimeout, CancellationToken cancellationToken)
{
  bool flag = millisecondsTimeout == -1;
  uint num = ((!flag) ? ((uint)Environment.TickCount) : 0u);
  bool flag2 = SpinWait(millisecondsTimeout);
  ....
}

private bool SpinWait(int millisecondsTimeout)
{
  if (IsCompleted)
  {
    return true;
  }
  if (millisecondsTimeout == 0)
  {
    return false;
  }
  int spinCountforSpinBeforeWait = System.Threading.SpinWait.SpinCountforSpinBeforeWait;
  SpinWait spinWait = default(SpinWait);
  while (spinWait.Count < spinCountforSpinBeforeWait)
  {
    spinWait.SpinOnce(-1);
    if (IsCompleted)
    {
      return true;
    }
  }
  return false;
}
```

#### Use `.GetAwaiter().GetResult()` if you have to be synchronous

Some people suggest using `Task.GetAwaiter().GetResult()` instead of `.Result` if you are compelled to go synchronous (which might be the case with legacy codebases that aren't async by design). Is this any better? Only slightly.

Consider the same example above but using `.GetAwaiter().GetResult()`:

```csharp
static async Task Main(string[] args)
{
  var activity = GetActivityAsync().GetAwaiter().GetResult();
  Console.WriteLine($"{activity}");
}
```

Looking at the framework code below for `TaskAwaiter<T>.GetResult()`, notice the `throw task.Exception` in `ThrowForNonSuccess`. Also, notice that the same `task.InternalWait` spin wait is called as in `Task<T>.Result`. So, `Task.GetAwaiter().GetResult()` is also a synchronous call just like `Task.Result`.

```csharp
public TResult GetResult()
{
	TaskAwaiter.ValidateEnd(m_task);
	return m_task.ResultOnSuccess;
}
internal static void ValidateEnd(Task task, ConfigureAwaitOptions options = ConfigureAwaitOptions.None)
{
	if (task.IsWaitNotificationEnabledOrNotRanToCompletion)
	{
		HandleNonSuccessAndDebuggerNotification(task, options);
	}
}
private static void HandleNonSuccessAndDebuggerNotification(Task task, ConfigureAwaitOptions options)
{
	if (!task.IsCompleted)
	{
		bool flag = task.InternalWait(-1, default(CancellationToken));
	}
	task.NotifyDebuggerOfWaitCompletionIfNecessary();
	if (!task.IsCompletedSuccessfully)
	{
		if ((options & ConfigureAwaitOptions.SuppressThrowing) == 0)
		{
			ThrowForNonSuccess(task);
		}
		task.MarkExceptionsAsHandled();
	}
}
private static void ThrowForNonSuccess(Task task)
{
	switch (task.Status)
	{
	case TaskStatus.Canceled:
		task.GetCancellationExceptionDispatchInfo()?.Throw();
		throw new TaskCanceledException(task);
	case TaskStatus.Faulted:
	{
		List<ExceptionDispatchInfo> exceptionDispatchInfos = task.GetExceptionDispatchInfos();
		if (exceptionDispatchInfos.Count > 0)
		{
			exceptionDispatchInfos[0].Throw();
			break;
		}
		throw task.Exception;
	}
	}
}
```

So
- `.Result` wraps exceptions in an `AggregateException`. `Task.GetAwaiter().GetResult()` surfaces them unvarnished.  Exception handling is more intuitive in the latter case since we can catch specific exceptions instead of catching the `AggregateException` and inspecting inner exceptions.
- Both are blocking calls and can lead to deadlocks (when synchronization context is involved) and thread starvation issues.

Bottom line: avoid blocking calls unless you have to – for example in a legacy application where async all the way would end up in cascading changes throughout your codebase. Or if you have to call an async method in a constructor (don't do this – constructors should simply initialize objects; it is not a good idea to do IO in a constructor). Use async/await throughout your code. [This is the way](https://www.youtube.com/watch?v=uelA7KRLINA).

#### What about `Task.Run`?

Consider the following code:
```csharp
  var task = Task.Run(() =>
  {
      var result = GetNumberAsync(5).GetAwaiter().GetResult();
      return result;
  });

  var number = task.GetAwaiter().GetResult();
  Console.WriteLine($"{number}");
```

What exactly does `Task.Run` do? Dissecting the implementation in [ILSpy](https://github.com/icsharpcode/ILSpy/releases), we see that it creates a task instance and schedules it to run using the default `ThreadPoolTaskScheduler`, i.e. on a threadpool thread:
```csharp
public static Task<TResult> Run<TResult>(Func<TResult> function)
{
	return Task<TResult>.StartNew(null, function, default(CancellationToken), TaskCreationOptions.DenyChildAttach, InternalTaskOptions.None, TaskScheduler.Default);
}
internal static Task<TResult> StartNew(Task parent, Func<TResult> function, CancellationToken cancellationToken, TaskCreationOptions creationOptions, InternalTaskOptions internalOptions, TaskScheduler scheduler)
{
	if (function == null)
	{
		ThrowHelper.ThrowArgumentNullException(ExceptionArgument.function);
	}
	if (scheduler == null)
	{
		ThrowHelper.ThrowArgumentNullException(ExceptionArgument.scheduler);
	}
	Task<TResult> task = new Task<TResult>(function, parent, cancellationToken, creationOptions, internalOptions | InternalTaskOptions.QueuedByRuntime, scheduler);
	task.ScheduleAndStart(needsProtection: false);
	return task;
}
```

So, was `Task.Run` really required in our code above? No. We didn't need to run the `GetNumberAsync` method on a separate thread. Ideally, we would have just `await`ed it and taken a tea break. If we wanted to forego the tea break and obsessively check on it in synchronous fashion, we could have just done it on the calling thread using `.GetAwaiter().GetResult()`. With `Task.Run`, we have 2 threads doing a blocking wait. Also worth keeping in mind is that the threadpool thread doesn't share synchronization context with the calling thread. 

So when should `Task.Run` be used? One legitimate use case is for long-running CPU intensive tasks which can now be "awaited" just like async IO tasks. Another would be to run multiple CPU bound tasks in parallel on multiple CPUs. 

### Takeaways
- `Task.Result` and `Task.GetAwaiter().GetResult()` are both synchronous calls and should be avoided in async code. If unavoidable in legacy codebases, prefer `Task.GetAwaiter().GetResult()` since it propagates exceptions instead of wrapping them in `AggregateException`, and makes exception handling easier, since we can catch specific exceptions instead of inspecting inner exceptions in the `AggregateException`.
- `Task.Run` creates a new task and schedules it on a threadpool thread. Avoid using this for async operations since it schedules an additional threadpool thread and doesn't run under the same synchronization context as the calling thread. Legitimate use cases are for CPU bound tasks or to parallelize CPU intensive work that can be farmed out to multiple processors.

### Further Reading
- [How to Run an Async Method Synchronously in .NET
](https://code-maze.com/run-async-method-synchronously-dotnet/)
- [Avoid GetAwaiter().GetResult() at all cost](https://guiferreira.me/archive/2020/08/avoid-getawaiter-getresult-at-all-cost/)
- [Why is .GetAwaiter().GetResult() bad in C#?](https://www.nikouusitalo.com/blog/why-is-getawaiter-getresult-bad-in-c/)
- [ExecutionContext vs SynchronizationContext](https://devblogs.microsoft.com/pfxteam/executioncontext-vs-synchronizationcontext/)
- [ASP.NET Core SynchronizationContext
](https://blog.stephencleary.com/2017/03/aspnetcore-synchronization-context.html)
- [Parallel Computing - It's All About the SynchronizationContext](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)
  