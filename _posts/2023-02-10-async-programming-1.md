---
layout: post
category: blog
title: Async Programming Part 1
permalink: /blog/async-programming-1
description: Async Programming Part 1
image: async1.jpg
---

![Async Programming](../../../img/async-juggling.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@stevenliuyi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yi Liu</a> on <a href="https://unsplash.com/photos/hiFZxXC6pGw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

I have struggled to wrap my head around this topic for a while and the more I read the less I seem to understand. This guide is an attempt to demystify this concept for myself. Hopefully it will be useful to others as well. 
  
## Introduction

My morning starts with boiling water for tea. While the water is boiling, I unload the dishwasher. The water has started to boil by then and it is tea time. Later in the morning, I grind beans and start the coffee machine which takes about 3 minutes to brew. Meanwhile I put bread in the toaster, and go out to water the plants in the yard. By the time I return, I can smell fresh toast and coffee.  

This is asynchronous processing in the kitchen. Likewise, in a program, it means starting a task on a thread, letting the task run in the background (CPU bound tasks might run on a separate thread; IO bound tasks are just waiting around so don't even need to engage a thread) while the thread (you) is released to work on other tasks, and being notified when the task has completed so that you can consume the results at your convenience. By contrast, in a synchronous program, the thread (you) would be sitting around waiting for each task to complete before embarking on the next task. Not an optimal use of resources.

Like in a kitchen, it is possible to have many balls in the air at any one time. Async programming increases responsiveness and throughput by letting the program juggle tasks. Juggling isn't easy in real life nor in a program. The program has to worry about maintaining state and re-entry points and exception handling. It can get quite hairy and error-prone to write this code for each program. The beauty of .NET is that this complexity is abstracted away into a breathtakingly simple interface consisting of just two keywords - `async` and `await`. Evidence that this abstraction is so much superior to alternatives lies in the fact that it has been copied by other languages like [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and [Python](https://realpython.com/async-io-python/). 

## Definition

**Asynchronous programming** is a model where (relatively) long-running tasks can be started in the background, allowing the thread to do other work instead of blocking while users or other requests are waiting. 

Both user interfaces as well as backend services can benefit from this, the former in being responsive to the user, the latter in being able to process other requests while the async operation is in progress. 

.NET provides multiple patterns to support asynchronous programming that have evolved over the years to be more elegant and concise:

- [Asynchronous Programming Model (APM)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm) using `IAsyncResult` and `BeginXXX/EndXXX` from .NET 1 
- [Event based asynchronous (EAP)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/event-based-asynchronous-pattern-eap) pattern from .NET 2
- [Task-based Asynchronous Pattern (TAP)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) from .NET 4, with language support in the form of the `async/await` keywords from .NET 5.

We focus here on `async/await` as the pattern [recommended by Microsoft](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) for new development. 

Long-running tasks can be I/O bound (disk or network operations) or CPU bound (CPU intensive calculations). In C#, the same `async/await` pattern can be polymorphically used for both.


## User Guide


The async model in C# using `async/await` is:

### IO-bound operations 

Asynchronous methods are marked with the `async` modifier and immediately return with a token of type `Task` or `Task<T>` which can be "awaited" to retrieve the result of the operation which is of type `T`. The non-generic `Task` type is returned by async methods that don't return any data (return type `void`).

**Example:**

Consider a web service that returns a random activity. The `/api/activity` endpoint maps to the `GetAsync` asynchronous method. This method is marked with the `async` modifier and has an `Async` suffix by convention. It makes an HTTP call to an external API using the `_httpClient.GetAsync` asynchronous method, which returns an object of type `Task<HttpResponse>` - a token that can be used to retrieve a result of type `HttpResponse`. 

The `Task<HttpResponse>` token is returned immediately while the HTTP network call runs in the background. This token can be "awaited" to simulate a synchronous flow where the subsequent statement (`var content = response.Content;`) isn't executed until the result from the async HTTP call is available.  

```
using System.Net.Http;using System.Threading.Tasks;using System.Web.Http;using Newtonsoft.Json.Linq;namespace async_samples.Controllers{    public class ActivityController : ApiController    {        private static HttpClient _httpClient;        static ActivityController()        {            _httpClient = new HttpClient();        }        [Route("api/activity")]        public async Task<string> GetAsync()        {            var uri = "https://www.boredapi.com/api/activity";            var response = await _httpClient.GetAsync(uri);            var content = response.Content;            if (content != null)            {                var responseJson = await content.ReadAsStringAsync();                if (!string.IsNullOrEmpty(responseJson))                {                    dynamic jObj = JObject.Parse(responseJson);                    return jObj.activity?.ToString();                }            }            return null;        }    }}
```

By itself, this is a real benefit, since the calling thread that would have remained blocked during the HTTP call is now returned to the threadpool and can handle web requests waiting in the queue. The web service can now handle much more traffic. But it gets even better.

### Async tasks in parallel

The caller can choose to do other work instead of "awaiting" if the result of the HTTP call isn't immediately required. The "await" can be deferred until a point where the code can't continue without the HTTP result. This can make a big difference in execution time since we can now have independent tasks running in parallel. 

For example, if we wanted to return 10 activities instead of one, we could write something like this. This would suspend the thread for the duration of each HTTP call, effectively serializing them:

```
[Route("api/activities/serial")]public async Task<string[]> GetListSerialAsync(int n = 10){	var activities = new List<string>();	for (int i = 0; i < n; i++)	{		activities.Add(await GetAsync());	}	return activities.ToArray();}
```

Or, we could write the following code, which stashes the task objects from each call into a list instead of individually awaiting each. It then creates a task using `Task.WhenAll` that completes when all the tasks in the list complete, and awaits this task.  

For n = 10, the difference in execution time is small but evident. For n=100, it is significant (12 seconds vs 1.5 seconds on my VM).

```
[Route("api/activities")]public async Task<string[]> GetListAsync(int n = 10){	var tasks = new List<Task<string>>();	var activities = new List<string>();	for (int i = 0; i < n; i++)	{		tasks.Add(GetAsync());	}	// Wait for completion task	await Task.WhenAll(tasks);	// Harvest results	tasks.ForEach(async t => activities.Add(await t));	return activities.ToArray();}
```


### CPU-bound operations

CPU-bound asynchronous methods are also marked with the `async` modifier and return `Task` or `Task<T>` which can be stashed and queried later or awaited immediately. On the surface, calling a CPU-bound async method is identical to calling an IO-bound async method. Internally though, the CPU-bound task runs on a threadpool thread using `Task.Run` whereas no thread is involved in IO bound operations.

```
static async Task Main(){	Console.WriteLine(await DoCalcAsync());}static async Task<double> DoCalcAsync(){
	// This starts a background thread to do the calculation and
	// returns the main thread to the pool	return await Task.Run(() => SumOfSqrt(1000000000));}static double SumOfSqrt(int n){	return Enumerable.Range(1, n).Sum(i => Math.Sqrt(i));}
```

### Async all the way up

Only methods marked `async` can `await` other async methods. This leads to a cascading chain of `async` methods up the stack, and is the most unnerving aspect of converting synchronous code to asynchrous. Can the top level method be marked `async`? Yes, it can starting with [C# 7.1](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history?WT.mc_id=DOP-MVP-5003880#c-version-71). 

If an async method needs to be called from a non-async method, then `GetAwaiter().GetResult()` can be called on the returned `Task`/`Task<T>`. MVC/Web API controller methods, WinForms event handlers can all be marked as `async`. 

```
static void Main(){	var activity = GetActivityAsync().GetAwaiter().GetResult();	Console.WriteLine(activity);}
```  

## Under the hood

This was a short user guide. You could be done at this point and reap most of the benefits of using async. However, this is just the tip of the iceberg. It helps to know how things work internally for scenarios where something doesn't work as expected or you are trying to do something atypical and encounter the rough edges of the abstraction.

In a nutshell, when an async call is encountered, .NET captures the ambient context (called the synchronization context) and passes the continuation (the part of the method following the async call) to the async call and immediately returns a token - the `Task` object - which can be used to retrieve the results. The C# compiler implements the continuation as a state machine which it creates for each async method. The continuation gets passed all the way to the device driver which is async by design. Even synchronous operations are performed asynchronously by the device driver through a contraption of IRPs (I/O request packets), ISRs (Interrupt service request), DPC (Deferred procedure call) and kernel-mode APC (asynchronous procedure call) which runs on the IO thread pool and notifies the task that it is complete, which then queues the continuation to run on a threadpool thread, which updates the task object status. It is a [Rube Goldberg machine](https://en.wikipedia.org/wiki/Rube_Goldberg_machine) lurking just beneath the surface.

In future parts, we explore more nuances of async usage and delve into the internals of this complex machine.
  

## Takeaways

- Async programming helps us effectively utilize a computer's resources and make programs responsive while long-running I/O or CPU bound tasks are in progress. 
- Instead of blocking the thread while the operation is in progress, async methods queue a completion task to run when the operation is completed and return right away, releasing the calling thread.
- The same async/await pattern can be used for both I/O bound and CPU bound tasks. In the latter, a background thread does the work, while in the former, no thread is required while waiting for the network operation. 
- The part of the method following await (called the continuation) runs on a different threadpool thread from the calling thread.
- .NET creates a state machine for each async method to keep track of the code to be run following the async operation(s).
- The continuation captures and replays the synchronization context except when `ConfigureAwait(false)` is called on the task.


## Code
[https://github.com/cs31415/samples/tree/main/async/part1](https://github.com/cs31415/samples/tree/main/async/part1)


## Further Reading

- [Asynchronous programming - C#](https://docs.microsoft.com/en-us/dotnet/csharp/async)
- [Async/Await - Best Practices in Asynchronous Programming](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)