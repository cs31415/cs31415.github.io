---
layout: post
category: blog
title: My Stupid Guide to Async Programming Part 1
permalink: /blog/async-programming-1
description: Async Programming Part 1
image: async1.jpg
---

![Async Programming](../../../img/async-juggling.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@stevenliuyi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Yi Liu</a> on <a href="https://unsplash.com/photos/hiFZxXC6pGw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

I have always had a mental fog around this topic and the more I read the less I seem to understand. Writing this guide is an attempt to demystify this important concept to my stupid self. Hopefully it will be useful to others engaged in the same struggle. 
  
## Introduction

My morning starts with boiling water for tea. While the water is boiling, I unload the dishwasher. The water has started to boil by then and it is tea time. Later in the morning, I grind beans and start the coffee machine which takes about 3 minutes to brew. Meanwhile I put bread in the toaster, and go out to water the plants in the yard. By the time I return, I can smell fresh toast and coffee.  

This is asynchronous processing in the kitchen. Likewise, in a program, it means starting a task on a thread, letting the task run in the background (CPU bound tasks might run on a separate thread; IO bound tasks are just waiting around so don't even need to engage a thread) while the thread (you) is released to work on other tasks, and being notified when the task has completed so that you can consume the results at your convenience. By contrast, in a synchronous program, the thread (you) would be sitting around waiting for each task to complete before embarking on the next task. Not an optimal use of resources.

Like in a kitchen, it is possible to have many balls in the air at any one time. Async programming increases responsiveness and throughput by letting the program juggle tasks. Juggling isn't easy in real life nor in a program. The program has to worry about maintaining state and re-entry points and exception handling. It can get quite hairy and error-prone to write this code for each program. The beauty of .NET is that this complexity is abstracted away into a breathtakingly simple interface consisting of just two keywords - `async` and `await`. Evidence that this abstraction is so much superior to alternatives lies in the fact that it has been copied by other languages like [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and [Python](https://realpython.com/async-io-python/). 

## Definition

**Asynchronous programming** is a model where (relatively) long-running tasks can be started in the background, allowing the thread to do other work (on the same thread if it doesn't require the preceding task result, or by releasing it to the threadpool if it cannot proceed without the result) instead of blocking while users or other requests are waiting. 

Both user interfaces as well as backend services can benefit from this, the former in being responsive to the user, the latter in being able to process other requests while the async operation is in progress. 

.NET provides multiple patterns to support asynchronous programming that have evolved over the years to be more elegant and concise:

- [Asynchronous Programming Model (APM)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm) using `IAsyncResult` and `BeginXXX/EndXXX` from .NET 1 
- [Event based asynchronous (EAP)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/event-based-asynchronous-pattern-eap) pattern from .NET 2
- [Task-based Asynchronous Pattern (TAP)](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) from .NET 4, with language support in the form of the `async/await` keywords from .NET 5.

We focus here on `async/await` as the pattern [recommended by Microsoft](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) for new development. 

Long-running tasks fall into two buckets - I/O bound (disk or network operations) and CPU bound (CPU intensive calculations). In C#, the same `async/await` pattern can be polymorphically used for both.


## User Guide


The async model in C# using `async/await` is:

### IO-bound operations 

Asynchronous methods are marked with the `async` modifier and immediately return with a token of type `Task` or `Task<T>` which can be used to retrieve the result of the operation which is of type `T`. Async methods that don't return any data return the non-generic `Task` type.

**Example:**

Consider a web service that returns a random activity. The `GetAsync` method is invoked for the `/api/activity` endpoint. This method is marked with the `async` modifier, since it is an asynchronous method. It makes an HTTP call to an external API using the `_httpClient.GetAsync` async method. `HttpClient`'s `GetAsync` method is an asynchronous method that returns a `Task<HttpResponse>` type - a token that can be used to retrieve a result of type `HttpResponse`. 

Asynchronous methods are suffixed with `Async` to distinguish them from synchronous methods. The `Task<HttpResponse>` type is a token that is returned immediately while the HTTP call runs in the background. This token can be `awaited` to simulate a synchronous flow. This will ensure that the subsequent statement isn't executed until the result from the async HTTP call is available.  

```
using System.Net.Http;using System.Threading.Tasks;using System.Web.Http;using Newtonsoft.Json.Linq;namespace async_samples.Controllers{    public class ActivityController : ApiController    {        private static HttpClient _httpClient;        static ActivityController()        {            _httpClient = new HttpClient();        }        [Route("api/activity")]        public async Task<string> GetAsync()        {            var uri = "https://www.boredapi.com/api/activity";            var response = await _httpClient.GetAsync(uri);            var content = response.Content;            if (content != null)            {                var responseJson = await content.ReadAsStringAsync();                if (!string.IsNullOrEmpty(responseJson))                {                    dynamic jObj = JObject.Parse(responseJson);                    return jObj.activity?.ToString();                }            }            return null;        }    }}
```

By itself, this is a real benefit, since the calling thread that would have remained blocked during the HTTP call is now released to the threadpool and can service other web requests. The web service's throughput can be much higher. But it gets even better.

### Async tasks in parallel

The caller can choose to do other things if the result of the HTTP call isn't immediately required and only await the token when it's result is required. This can make a big difference in execution time since we can now have independent tasks running in parallel. 

As an example, if we wanted to return 10 activities instead of one, we could write something like this. This would suspend the thread for the duration of each HTTP call, effectively serializing them:

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

## To sum up

This was a short user guide. You could be done at this point and still reap all the benefits of using async. However, this is just the tip of the iceberg. It helps to know how things work internally for those cases where something doesn't work as expected or you are trying to do something atypical and encounter the rough edges of the abstraction.

In a nutshell, .NET creates a state machine for each async method to keep track of the current execution point, and passes a reference to the part of the method following the `await` (called the continuation and captured by the state machine) to the underlying operating system API to be called (by the device driver) once the async operation is complete. This continuation runs on a different (threadpool) thread from the calling thread. The ambient context of the thread (called the synchronization context) is captured by the continuation and replayed when it runs. There are times when we may not want the context to be captured (especially in libraries that can be called by clients with different types of contexts) and this can be done through calling `ConfigureAwait(false)` on the task object. This may not make much sense at this time, but conveys a hint of the complexities lurking underneath.

Part 2 (TBD) descends into the labyrinth. 

## Takeaways

- Async programming helps us effectively utilize a computer's resources and make programs responsive while long-running I/O or CPU bound tasks are in progress. 
- Instead of blocking the thread while the operation is in progress, async methods queue a completion task to run when the operation is completed and return right away, releasing the calling thread.
- The same async/await pattern can be used for both I/O bound and CPU bound tasks. In the latter, a background thread does the work, while in the former, no thread is required while waiting for the network operation. 
- The part of the method following await (called the continuation) runs on a different threadpool thread from the calling thread.
- .NET creates a state machine for each async method to keep track of the code to be run following the async operation(s).
- The continuation captures and replays the synchronization context except when `ConfigureAwait(false)` is called on the task.



## Further Reading

- [Asynchronous programming - C#](https://docs.microsoft.com/en-us/dotnet/csharp/async)
- [Async/Await - Best Practices in Asynchronous Programming](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)