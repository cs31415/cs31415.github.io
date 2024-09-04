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

Now, armed with knowledge of internals (see parts [1](/blog/async-programming-1), [2](/blog/async-programming-2), [3](/blog/async-programming-3), [4](/blog/async-programming-4), [5](/blog/async-programming-5)), we focus on some usage topics and practical matters in this and the next few posts. 

1. #### Don't use `.Result`

##### `.Result`
Calling `.Result` on an async method is a blocking call. Consider the example below.

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

Looking into the framework code for `Task.Result`, we see that `Thread.SpinWait` is eventually called, which is an implementation of an adaptive [spin wait](https://en.wikipedia.org/wiki/Busy_waiting) algorithm, which tries to balance responsiveness and CPU usage. This implementation can call `Thread.Sleep(1)` which can block the thread or `Thread.Sleep(0)` and `Thread.Yield()` which can consume CPU cycles.   

Also, since this is a blocking call, and the async method's continuation can potentially run on the same thread, it can cause deadlocks. Since threads are tied up waiting, it can result in thread starvation issues.

Also, in `GetExceptions` below, we see that any exceptions that arise are wrapped in an `AggregateException`:
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

##### `.GetAwaiter().GetResult()`

Some people suggest using `Task.GetAwaiter().GetResult()` instead of `.Result`. Is this any better? Only slightly.

Consider the same example above but using `.GetAwaiter().GetResult()`:

```csharp
static async Task Main(string[] args)
{
  var activity = GetActivityAsync().GetAwaiter().GetResult();
  Console.WriteLine($"{activity}");
}
```

Looking at the framework code for `TaskAwaiter<T>.GetResult()`, notice the `throw task.Exception` in `ThrowForNonSuccess`. Also, notice that the same `task.InternalWait` spin wait is called as in `Task<T>.Result`:
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
- `.Result` wraps exceptions in an `AggregateException`. `Task.GetAwaiter().GetResult()` surfaces them directly.  Exception handling is more intuitive in the latter case.
- Both are blocking calls and can lead to deadlocks when synchronization context is involved and thread starvation issues.
- Everything else looks to be the same including the spin wait.

Bottom line: avoid blocking calls unless you have to – for example in a legacy application where async all the way up would end up modifying most of your codebase. Use async/await throughout your code. This is the way.


### Further Reading
- [How to Run an Async Method Synchronously in .NET
](https://code-maze.com/run-async-method-synchronously-dotnet/)
- [Avoid GetAwaiter().GetResult() at all cost](https://guiferreira.me/archive/2020/08/avoid-getawaiter-getresult-at-all-cost/)
- [Why is .GetAwaiter().GetResult() bad in C#?](https://www.nikouusitalo.com/blog/why-is-getawaiter-getresult-bad-in-c/)
  