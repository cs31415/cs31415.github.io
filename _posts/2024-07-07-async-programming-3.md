---
layout: post
category: blog
title: Async Programming Part 3
permalink: /blog/async-programming-3
description: Async Programming Part 3
image: async3.jpg
---

![Async Programming](../img/async3.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@bradyn?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Bradyn Trollip</a> on <a href="https://unsplash.com/photos/person-holding-white-and-blue-plastic-blocks-pxVOztBa6mY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a></span>
  
Building on from parts [1](/blog/async-programming-1) & [2](/blog/async-programming-2), we look at continuations.

### Continuations

A continuation is a function that encapsulates the remainder of the computation at any given point. Its inputs are the interim results upto that point, and its output is the final result returned by the computation. Computation here can refer to a program or to an operation within a larger program. 

With respect to asynchronous programming, a continuation represents the code that should run after the asynchronous operation completes. The continuation also captures the program's local context, so that the program can resume on the same context.

An async function doesn't block, but immediately returns with a `Task` or `Task<T>` instance, a token for retrieving the result. When the async operation completes, the downstream code runs in the continuation, which has access to the result of the operation.  

Says [MSDN](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks):
> In asynchronous programming, it's common for one asynchronous operation to invoke a second operation on completion. Continuations allow descendant operations to consume the results of the first operation. 

As we saw in [part 2](/blog/async-programming-2), the results of the first operation, called the *antecedent*, are passed  to the continuation. Thus, continuations can be used to chain tasks together, in a monadic fashion (From [part 2](/blog/async-programming-2), monads are types that encapsulate structured data and allow composing operations that accept and return the same structure via the `bind` method), with the `ContinueWith` method acting as a `bind` operator. This enables async functions to register a completion routine to be run when the async operation completes. The completion routine might, in turn, call other async functions with their own continuations, and so on. 

Let's take another look at that example:
```csharp
// continuation task
string AskName()
{
  Console.WriteLine("What is your name?");
  return Console.ReadLine() ?? "None";
}

Console.WriteLine($"Thread id = {Thread.CurrentThread.ManagedThreadId}");

var antecedentTask = new Task<string>(AskName);
var continuation = antecedentTask.ContinueWith(antecedent =>
{
  Console.WriteLine(
    $"Continuation task running. Thread id = {Thread.CurrentThread.ManagedThreadId}");
  Console.WriteLine($"Hi {antecedent.Result}!");
});

Console.WriteLine("starting antecedent task");
antecedentTask.Start();
Console.WriteLine("waiting for antecedent task to finish");
antecedentTask.Wait();

Console.WriteLine(
  $"waiting for continuation task to finish. Thread id = {Thread.CurrentThread.ManagedThreadId}");
continuation.Wait();
```

The output of this code is (notice the continuation running on a different thread):
```text
Thread id = 1
starting antecedent task
waiting for antecedent task to finish
What is your name?
Alice
waiting for continuation task to finish. Thread id = 1
Continuation task running. Thread id = 6
Hi Alice!
```

How is a continuation different from a closure, a callback function that captures references to surrounding variables? 

Continuations can make use of closures, but are a more powerful control flow construct that can be used to compose tasks. As seen above, they can connect async tasks and their post-completion routines. In part [1](/blog/async-programming-1), we saw how async operations can be started in parallel and tied to a single completion routine that can run when either all tasks complete (`Task.WhenAll`) or the first task completes (`Task.WhenAny`). In fact, continuations are used under the hood not just for async/await but also for iterators, which is a future blog post.

### Continuations in .NET

A .NET continuation is not just a callback, a piece of code to be run following an operation, but code that is to be run in a specific program context. The word context here represents the local environment (thread, HTTP request etc.) that the code runs in and has access to.  The continuation captures its local context (thread id, `HttpContext.Current` etc.), represented by the `SynchronizationContext` base type, and schedules itself to run on that same context when the async operation ends. This might be a specific thread in case of UI frameworks like Windows Forms, or a threadpool thread in case of ASP.NET. 

As [Stephen Cleary](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext) explains:
> One aspect of SynchronizationContext is that it provides a way to queue a unit of work to a context. Note that this unit of work is queued to a context rather than a specific thread. This distinction is important, because many implementations of SynchronizationContext aren’t based on a single, specific thread.

Windows Forms and ASP.NET provide their own implementations by extending `SynchronizationContext`. In Windows Forms, the context is the UI thread where all UI code runs in serial. In ASP.NET (non-core), the context is the HTTP request, and code targeting a specific request runs serially, on one or more threadpool threads. 

Thus, a continuation is a callback that resumes program flow following the async method call, runs in the same context and has access to the same ambient state as the original code. It may run on the same thread as the original code or on a threadpool thread depending on the specific implementation of `SynchronizationContext` used in the application.

For example, consider the web service from [part 1](/blog/async-programming-1) that returns a random activity:
```csharp
[Route("api/activity")]
public async Task<string> GetAsync()
{
  var uri = "https://www.boredapi.com/api/activity";
  var response = await _httpClient.GetAsync(uri);
  var content = response.Content;
  if (content != null)
  {
    var responseJson = await content.ReadAsStringAsync();
    if (!string.IsNullOrEmpty(responseJson))
    {
      dynamic jObj = JObject.Parse(responseJson);
      return jObj.activity?.ToString();
    }
  }

  return null;
}
```

The continuation following the `_httpClient.GetAsync` call can be written as:

```csharp
async Task<string> Continuation(Task<HttpResponse> antecedent, SynchronizationContext context)
{ 
  // Capture and restore the original context, since this could be a reusable 
  // threadpool thread
  var oldContext = SynchronizationContext.Current;
  try 
  {
    // set context to the original code's context
    SynchronizationContext.SetSynchronizationContext(context);	

    var response = antecedent.Result;
    var content = response.Content;
    if (content != null)
    {
      var responseJson = await content.ReadAsStringAsync();
      if (!string.IsNullOrEmpty(responseJson))
      {
        dynamic jObj = JObject.Parse(responseJson);
        return jObj.activity?.ToString();
      }
    }

    return null;	
  }
  finally 
  {
    SynchronizationContext.SetSynchronizationContext(oldContext);
  }
}
```

and can be called thus:
```csharp
public async Task<string> GetAsync()
{
  var uri = "https://www.boredapi.com/api/activity";
  var responseTask = _httpClient.GetAsync(uri);
  responseTask.ContinueWith(antecedent => Continuation(antecedent, SynchronizationContext.Current));	
}
```

This is an oversimplification, though conceptually accurate. So, continuations are context-aware callbacks. They dovetail with the awaitable/awaiter pattern to enable linear flow control in async code. We will examine this pattern in the next post.

### Takeaways
- A continuation is a context-aware callback that encapsulates the remainder of the computation at any given point. Its inputs are the interim results upto that point, and its output is the final result returned by the computation.
- Continuations can be used to chain tasks together. This enables async functions to recursively call other async functions.
- Continuations run in the original program context, and might run on a separate thread from the original code.


### Further Reading
- [Chaining tasks using continuation tasks](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks)
- [A Tour of Task, Part 7: Continuations](https://blog.stephencleary.com/2015/01/a-tour-of-task-part-7-continuations.html)
- [Stephen Cleary: Parallel Computing: Its all about the SynchronizationContext](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext) 
