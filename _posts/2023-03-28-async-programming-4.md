---
layout: post
category: blog
title: Async Programming Part 3
permalink: /blog/async-programming-3
description: Async Programming Part 3
image: async2.jpg
---

![Async Programming](../img/async3.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@bradyn?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Bradyn Trollip</a> on <a href="https://unsplash.com/photos/person-holding-white-and-blue-plastic-blocks-pxVOztBa6mY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a></span>
  
In the last part, we looked at the continuation pattern and the awaiter pattern. In this part, we'll look at async method builder and the state machine.

### Async Method Builder

Orchestrates the task, state machine, attaches continuations etc. Builds async method body by constructing and firing up the state machine. Houses a reference to the task instance which it returns as a token to the caller.

### State Machine

The state machine is where continuations (of which there may be more than one, since there could be more than one async operation in a method) and the awaiter pattern are all assembled together.

A state machine, also known as a finite-state machine, is a mathematical model of computation that represents a system as a set of discrete states and transitions between those states based on inputs. We use them every day - traffic lights, elevators, air conditioning systems, microwaves, washing machines, dishwashers, etc. 

In .NET, state machines are used to model asynchronous methods. They bear an uncanny resemblance to iterators, and have a `MoveNext` method for advancing to the next state. They capture the program state at each await using instance variables and are responsible for attaching continuations to asynchronous operations in the form of their own `MoveNext` methods. When the async operation completes, the continuation runs, the state machine resumes from the point where it paused, and runs until the next await is reached, at which point this process repeats. 

Why do we need state machine? 
Says [Sergey Tepliakov](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c/):
> A regular method has just one entry point and one exit point (it could have more than one return statement but at the runtime there is just one exit point for a given call). But async methods (*) and iterators (methods with yield return) are different. In the case of an async method, a method caller can get the result (i.e. Task or Task<T>) almost immediately and then “await” the actual result of the method via the resulting task.

Thus, a single instance of an async method can be entered and exited multiple times - this is the definition of a couroutine, a construct that allows for execution to be suspended and resumed.


Consider the following async method that reads a file's contents:
```
static async Task<string> GetFileContentsAsync() 
{
    string filePath = @"hello.txt"; 

    try
    {
        return await File.ReadAllTextAsync(filePath);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An error occurred: {ex.Message}");
    }

    return null;
}
```
It is invoked like this:
```
static async Task Main(string[] args)
{
    var contents = await GetFileContentsAsync();
    WriteLine($"file contents: {contents}");
}
```

The compiler generates a state machine class for each method marked with `async`, and uses an `AsyncTaskMethodBuilder` to instantiate the state machine. 

The `GetFileContentsAsync` method is transformed by the compiler into:
```
[AsyncStateMachine(typeof(<GetFileContentsAsync>d__4))]
[DebuggerStepThrough]
private static Task<string> GetFileContentsAsync()
{
	<GetFileContentsAsync>d__4 stateMachine = new <GetFileContentsAsync>d__4();
	stateMachine.<>t__builder = AsyncTaskMethodBuilder<string>.Create();
	stateMachine.<>1__state = -1;
	stateMachine.<>t__builder.Start(ref stateMachine);
	return stateMachine.<>t__builder.Task;
}
```

The state machine class `<GetFileContentsAsync>d__4` has an internal state tracker, program state instance variables and an awaiter reference. `MoveNext` is where all the action is. This is the method that the builder's `Start` method invokes to kickstart the state machine, and is also the continuation that is attached to async operations every time an await is reached. The state tracker (`<>1__state`) transitions from -1 (initial state) to 0 (1st async operation in progress) to 1 (2nd async operation in progress), 2, 3...n (nth async operation in progress), to -2 (end state).    

```
private sealed class <GetFileContentsAsync>d__3 : IAsyncStateMachine
{
	public int <>1__state; // state tracker
	public AsyncTaskMethodBuilder<string> <>t__builder; // task method builder reference
	private string <filePath>5__1;
	private string <>s__2;
	private Exception <ex>5__3;
	private TaskAwaiter<string> <>u__1; // awaiter reference

	private void MoveNext()
	{
		int num = <>1__state;
		string result;
		try
		{
			if (num != 0)
			{
				<filePath>5__1 = "hello.txt";
			}
			try
			{
				TaskAwaiter<string> awaiter;
				if (num != 0)
				{
					awaiter = File.ReadAllTextAsync(<filePath>5__1).GetAwaiter();
					// check here if operation has already completed synchronously
					if (!awaiter.IsCompleted)
					{
						// if not completed						
						num = (<>1__state = 0);
						<>u__1 = awaiter;
						<GetFileContentsAsync>d__3 stateMachine = this;

						// continuation attached at this point
						<>t__builder.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine);
						return;
					}
				}
				else
				{
					// if here, it means the async operation completed
					awaiter = <>u__1;
					<>u__1 = default(TaskAwaiter<string>);
					num = (<>1__state = -1);
				}

				// get async operation result 
				<>s__2 = awaiter.GetResult();
				result = <>s__2;
			}
			catch (Exception ex)
			{
				<ex>5__3 = ex;
				Console.WriteLine("An error occurred: " + <ex>5__3.Message);
				goto IL_00c9;
			}
			goto end_IL_0007;
			IL_00c9:
			result = null;
			end_IL_0007:;
		}
		catch (Exception ex)
		{
			<>1__state = -2;
			<filePath>5__1 = null;
			<>t__builder.SetException(ex);
			return;
		}
		<>1__state = -2;
		<filePath>5__1 = null;

		// set result into task instance 
		<>t__builder.SetResult(result);
	}

	void IAsyncStateMachine.MoveNext()
	{
		//ILSpy generated this explicit interface implementation from .override directive in MoveNext
		this.MoveNext();
	}

	[DebuggerHidden]
	private void SetStateMachine(IAsyncStateMachine stateMachine)
	{
	}

	void IAsyncStateMachine.SetStateMachine(IAsyncStateMachine stateMachine)
	{
		//ILSpy generated this explicit interface implementation from .override directive in SetStateMachine
		this.SetStateMachine(stateMachine);
	}
}
```

`File.ReadAllTextAsync` eventually calls the Windows system API [`ReadFile`](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-readfile
) which internally calls the I/O manager in the Windows kernel. At this point, the system transitions from user mode to kernel mode, a privileged  mode with low-level hardware access. The I/O manager routes the request to the file system driver. The sequence of calls looks like this:
```
StreamReader.ReadAsync:
FileStream.ReadAsync:
	AsyncWindowsFileStreamStrategy.ReadAsync:
	OSFileStreamStrategy.ReadAsync:
		RandomAccess
			.ReadAtOffsetAsync
			.QueueAsyncReadFile:
			SafeFileHandle.OverlappedValueTaskSource vts = handle.GetOverlappedValueTaskSource();
			int errorCode = Interop.Errors.ERROR_SUCCESS;
			try
			{
				NativeOverlapped* nativeOverlapped = vts.PrepareForOperation(buffer, fileOffset, strategy);
				Debug.Assert(vts._memoryHandle.Pointer != null);

				// Queue an async ReadFile operation.
				if (Interop.Kernel32.ReadFile(handle, (byte*)vts._memoryHandle.Pointer, buffer.Length, IntPtr.Zero, nativeOverlapped) == 0)
				{...}
``` 

`ReadFile` uses the Windows Overlapped I/O mechanism which is asynchronous. An `OverlappedValueTaskSource` instance `vts` is obtained and used to setup the OVERLAPPED structure. The `ReadFile` returns immediately after which 
`vts.RegisterForCancellation` is called
`vts.FinishedScheduling` is called which in turn invokes `ManualResetValueTaskSourceCore<TResult>.SignalCompletion` to queue up the continuation.

```
private void SignalCompletion()
{
    if (_completed)
    {
        ThrowHelper.ThrowInvalidOperationException();
    }
    _completed = true;

    if (_continuation is null && Interlocked.CompareExchange(ref _continuation, ManualResetValueTaskSourceCoreShared.s_sentinel, null) is null)
    {
        return;
    }

    if (_executionContext is null)
    {
        if (_capturedContext is null)
        {
            if (RunContinuationsAsynchronously)
            {
                ThreadPool.UnsafeQueueUserWorkItem(_continuation, _continuationState, preferLocal: true);
            }
            else
            {
                _continuation(_continuationState);
            }
        }
        else
        {
            InvokeSchedulerContinuation();
        }
    }
    else
    {
        InvokeContinuationWithContext();
    }
}
```

Completion Notification: The completion of an overlapped I/O operation can be signaled in several ways, including setting a Win32 event handle, using I/O completion ports, or specifying a completion routine (e.g., ReadFileEx, WriteFileEx).


> Some time after the write request started, the device finishes writing. It notifies the CPU via an interrupt.

> Anyway, the ISR is properly written, so all it does is tell the device “thank you for the interrupt” and queue a Deferred Procedure Call (DPC).

> When the CPU is done being bothered by interrupts, it will get around to its DPCs. DPCs also execute at a level so low that to speak of “threads” is not quite right; like ISRs, DPCs execute directly on the CPU, “beneath” the threading system.

> The DPC takes the IRP representing the write request and marks it as “complete”. However, that “completion” status only exists at the OS level; the process has its own memory space that must be notified. So the OS queues a special-kernel-mode Asynchronous Procedure Call (APC) to the thread owning the `HANDLE`.

> Since the library/BCL is using the standard P/Invoke overlapped I/O system, it has already [registered the handle](https://msdn.microsoft.com/en-us/library/system.threading.threadpool.bindhandle.aspx) with the I/O Completion Port (IOCP), which is part of the thread pool. So an I/O thread pool thread is borrowed briefly to execute the APC, which notifies the task that it’s complete.

> The task has captured the UI context, so it does not resume the `async` method directly on the thread pool thread. Instead, it queues the continuation of that method onto the UI context, and the UI thread will resume executing that method when it gets around to it.

> So, we see that there was no thread while the request was in flight. When the request completed, various threads were “borrowed” or had work briefly queued to them. This work is usually on the order of a millisecond or so (e.g., the APC running on the thread pool) down to a microsecond or so (e.g., the ISR). But there is no thread that was blocked, just waiting for that request to complete.

> The ThreadPool keeps a number of threads registered in its IOCP; these are different than the worker threads that most people associate with the ThreadPool. The ThreadPool manages both worker thread and I/O threads.

> I/O is "asynchronous" from a much broader scope (it would be "asynchronous" from the perspective of _any_ thread, or process, or the OS, or the CPU).

> The device driver’s Interrupt Service Routine (ISR) responds to the interrupt.

- [Overlapped Operations - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/devio/overlapped-operations)
> Functions called for overlapped operation can return immediately, even though the operation has not been completed. This enables a time-consuming I/O operation to be executed in the background while the calling thread is free to perform other tasks.

- [Platform Invoke (P/Invoke) | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke#:~:text=P%2FInvoke%20is%20a%20technology,InteropServices%20.&text=InteropServices%20namespace%20that%20holds%20all%20the%20items%20needed.)
> P/Invoke is **a technology that allows you to access structs, callbacks, and functions in unmanaged libraries from your managed code**. Most of the P/Invoke API is contained in two namespaces: System and System. Runtime. InteropServices . ... InteropServices namespace that holds all the items needed.

- [MSDN Magazine: Parallel Computing - It's All About the SynchronizationContext | Microsoft Docs](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)

> One aspect of SynchronizationContext is that it provides a way to queue a unit of work to a context.

> every thread has a “current” context. A thread’s context isn’t necessarily unique; its context instance may be shared with other threads.

> A third aspect of SynchronizationContext is that it keeps a count of outstanding asynchronous operations. This enables the use of ASP.NET asynchronous pages and any other host needing this kind of count. In most cases, the count is incremented when the current SynchronizationContext is captured, and the count is decremented when the captured SynchronizationContext is used to queue a completion notification to the context.

> The ASP.NET SynchronizationContext is installed on thread pool threads as they execute page code. When a delegate is queued to a captured AspNetSynchronizationContext, it restores the identity and culture of the original page and then executes the delegate directly.

> By default, **the current SynchronizationContext is captured at an await point**, and this SynchronizationContext is used to resume after the await (more precisely, it captures the current SynchronizationContext _unless it is null_, in which case it captures the current TaskScheduler):

> **ConfigureAwait provides a means to avoid the default SynchronizationContext capturing behavior; passing false for the flowContext parameter prevents the SynchronizationContext from being used to resume execution after the await.**

- [Async and Await](https://blog.stephencleary.com/2012/02/async-and-await.html)
> when you await a built-in awaitable, then the awaitable will capture the current “context” and later apply it to the remainder of the async method. What exactly is that “context”?

> Simple answer:
1.  If you’re on a UI thread, then it’s a UI context.
2.  If you’re responding to an ASP.NET request, then it’s an ASP.NET request context.
3.  Otherwise, it’s usually a thread pool context.

> Complex answer:
1.  If SynchronizationContext.Current is not null, then it’s the current SynchronizationContext. (UI and ASP.NET request contexts are SynchronizationContext contexts).
2.  Otherwise, it’s the current TaskScheduler (TaskScheduler.Default is the thread pool context).

- `Task<T>.Result` can be used to access the result of an `awaited` method.
	- This might work in a console application, but not in a service or Windows app. The reason is that there is no synchronization context in a console app
	
### Best practices

### Field notes on converting sync code to async
- async methods and recursion
- async methods and out/ref parameters
- async LINQ handlers
- async lambdas
- what the heck is ConfigureAwait(false)?
	- [ConfigureAwait FAQ - .NET Blog](https://devblogs.microsoft.com/dotnet/configureawait-faq/)
	- `SynchronizationContext` is a general abstraction for a “scheduler”.
- HttpContext.Current in async controllers
- async getters/setters
- async code in aspx event handlers
- Unit test setup for protected async methods

### Takeaways


### Further Reading

### NOTES:
- [Wikipedia - Finite-State Machine](https://en.wikipedia.org/wiki/Finite-state_machine)
- [Exploring the async/await State Machine – The Awaitable Pattern](https://vkontech.com/exploring-the-async-await-state-machine-the-awaitable-pattern/)
- [Asynchronous programming - C# | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/async)
- [Async in depth | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth)
- [Async Programming - Introduction to Async/Await on ASP.NET | Microsoft Docs](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/october/async-programming-introduction-to-async-await-on-asp-net)
- [Async/Await - Best Practices in Asynchronous Programming | Microsoft Docs](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- Notes from fixing async usage in .NET code:
- Tendency to use `.Result` instead of awaiting async method calls
- async method cannot use out parameters. A workaround for this is to use named tuples (C# 7 onwards)
- [c# - How to write an async method with out parameter? - Stack Overflow](https://stackoverflow.com/questions/18716928/how-to-write-an-async-method-with-out-parameter)
- [Tuples in C# 7 - Thomas Levesque's .NET Blog](https://thomaslevesque.com/2016/07/25/tuples-in-c-7/)
- aspx event handlers cannot be marked as async. The recommended pattern is to set Page.Async to true and use `RegisterAsyncTask`.
- [c# - Is it safe to use async/await in ASP.NET event handlers? - Stack Overflow](https://stackoverflow.com/questions/27282617/is-it-safe-to-use-async-await-in-asp-net-event-handlers)
- [Using Asynchronous Methods in ASP.NET 4.5 | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45)
- When passing async lambdas around, make sure they return `Task<T>`
- [c# - Using Func delegate with Async method - Stack Overflow](https://stackoverflow.com/questions/37280405/using-func-delegate-with-async-method)
- [Synchronous and Asynchronous Delegate Types](https://blog.stephencleary.com/2014/02/synchronous-and-asynchronous-delegate.html)
- LINQ and async lambdas:
- [c# - Linq and Async Lambdas - Stack Overflow](https://stackoverflow.com/questions/36445257/linq-and-async-lambdas)
- `HttpContext.Current` is null in async controller actions
- [Don't use HttpContext.Current, especially when using async](https://swimburger.net/blog/dotnet/don-t-use-httpcontext-current-especially-when-using-async)
- [All about - .NET Blog](https://devblogs.microsoft.com/dotnet/all-about-httpruntime-targetframework/)
- Executing async code in catch blocks. Seems like this wasn't possible prior to C# 6, but is now supported.
- Protected async method setup must return `Task.FromResult()`, and `Setup` should be templated with `Task<returnType>`
- Async getters/setters: [Async OOP 3: Properties](https://blog.stephencleary.com/2013/01/async-oop-3-properties.html)
