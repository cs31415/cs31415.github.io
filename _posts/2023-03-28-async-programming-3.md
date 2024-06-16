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
  
Building on from parts [1](/blog/async-programming-1) & [2](/blog/async-programming-2), in part 3, we look at continuations and the awaiter pattern.

### Continuations

A continuation conceptually represents the remainder of the program at any point. More [formally](https://opendsa.cs.vt.edu/ODSA/Books/PL/html/FP9.html):
> A continuation is a callback function k that represents the current state of the program’s execution. More precisely, the continuation k is a function of one argument, namely the value that has been computed so far, that returns the final value of the computation after the rest of the program has run to completion. 
 
For example, in this function:
```clojure
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

The continuation following `_httpClient.GetAsync(uri)` is the remainder of the function:
```
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
```

Written as a function, the continuation is :
```
async Task<string> Continuation(string response) 
{
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
And the original function rewritten using the continuation is:
```
public async Task<string> GetAsync()
{
	var uri = "https://www.boredapi.com/api/activity";
	var response = await _httpClient.GetAsync(uri);
	return Continuation(response);
}
```

From [MSDN](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks):
> In asynchronous programming, it's common for one asynchronous operation to invoke a second operation on completion. Continuations allow descendant operations to consume the results of the first operation. 

Continuations are the glue holding the async mechanism together. A continuation is a task that is invoked when an async operation ends. The results of the first operation, called the *antecedent*, are passed through to the continuation. Thus, continuations can be used to chain tasks together. 

You might have seen this [*continuation passing style*](https://2ality.com/2012/06/continuation-passing-style.html) in JavaScript code.

```javascript
function getTicketAvailability(cart, cbReserveTickets, cbErrorHandler) {
	try {
		// make http call to theater point of sale service
		let isAvailable = theaterPos.getAvailability(cart);
		if (isAvailable) {
			cbReserveTickets(cart, availability, cbProcessPayment, cbErrorHandler);
		}
		else {
			cbErrorHandler(new error('no availability'));
		}
	} catch (error) {
		cbErrorHandler(error);
	}
}

function reserveTickets(cart, availability, cbProcessPayment, cbErrorHandler) {
	...
}

function errorHandler(error) {
	log(error);
	throw error;
}

show = { movie: 'Arrival', theater: 'Regal LA', showtime: 'May 1 7:00pm' };
cart = { show: show, tickets: [{ type: 'Adult', qty: 2, seats: 'G1,G2' }], payment: { token: 'cc-token' } }

getTicketAvailability(show, reserveTickets, errorHandler);
```
In the above code, function `getTicketAvailability` connects to a theater point of sale to get ticket availability. In addition to the arguments it needs to perform this task, it takes in two callback functions to be invoked on success and failure respectively - `cbReserveTickets` and `cbErrorHandler`. It passes its result to the success callback function, which in turn passes its result to the next callback function and so on, like a relay race. Notice that control flow is not linear. `getTicketAvailability` returns immediately while the availability call is in progress, asynchronously. Likewise, `cbReserveTickets` returns immediately while ticket reservation is in progress. This type of control flow is not only hard to follow, but also to program with. Continuations and the awaiter pattern solve this problem in .NET by simulating a linear flow for asynchronous code, as we will see shortly. 

Here's a C# example:
```clojure
// continuation task
string AskName()
{
    return "Alice";
}

var antecedentTask = new Task<string>(AskName);
var continuation = antecedentTask.ContinueWith(antecedent =>
{
    Console.WriteLine($"Hi {antecedent.Result}!");
});

Console.WriteLine("antecedent task");
antecedentTask.Start();
antecedentTask.Wait();

PrintTask(antecedentTask);
PrintTask(continuation);

return;
```

Is a continuation similar to a callback function that captures references to surrounding variables, i.e. a closure? 

A continuation can make use of closures, but is actually a more powerful control flow mechanism that can be used to compose tasks. .NET Iterators and async/await for example, both use continuations under the hood.

### Continuations in .NET

A .NET continuation captures not just surrounding variable references, but also the state of its local environment (thread, etc.) - represented by `SynchronizationContext`, and schedules itself to run on that same context when the async operation ends. `SynchronizationContext` tells the code where to resume executing. This might be a specific thread, but not always. As [Stephen Cleary](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext) explains:
> One aspect of SynchronizationContext is that it provides a way to queue a unit of work to a context. Note that this unit of work is queued to a context rather than a specific thread. This distinction is important, because many implementations of SynchronizationContext aren’t based on a single, specific thread.

Windows Forms and ASP.NET override `SynchronizationContext` and provide their own implementations. In Windows Forms, the context is the UI thread on which all UI code runs in a serial fashion. In ASP.NET (non core), the context is the http request, and code targeting a specific request runs serially. Thus, context is an abstraction that is upto each implementation to define. 

Is a continuation just a callback function? It is more than that. A continuation is a special kind of callback that runs the part of the method following the async method call in the original code's context and has access to the same local state as the original code. It may run on the same thread as the original code or on a threadpool thread depending on the specific implementation of `SynchronizationContext` used in the application.

So, in .NET, the continuation function we wrote above becomes:
```clojure
async Task<string> Continuation(string response, SynchronizationContext context) 
	// Capture and restore the original context, since this could be a reusable 
	// threadpool thread
	var oldContext = SynchronizationContext.Current;
	try 
	{
		SynchronizationContext.SetSynchronizationContext(context);	

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
```

And the calling code:
```clojure
public async Task<string> GetAsync()
{
		var uri = "https://www.boredapi.com/api/activity";
		var response = await _httpClient.GetAsync(uri);
		return Continuation(response, SynchronizationContext.Current);
}
```

To recap, continuations are completion tasks that are passed the results of async operations and resume on the same ambient context as the original operation. They complete the loop between the initiation of the async operation and its fulfillment. 

### The Awaiter Pattern

The awaiter pattern is used to make async code look and flow like synchronous code.

An async method runs synchronously until an async operation is encountered. The code then checks if the async operation is already complete (which might happen if there is an input validation error for example). If yes, it just gets the result of the operation and the program continues on synchronously. If the async operation is still in progress, then it captures and sets a continuation to be invoked post-completion and returns an awaiter instance. When the async operation is complete, the continuation runs (possibly on a different thread) and sets the result of the operation in the task instance, which is available to the caller through the awaiter. (see [Vasil Kosturski](https://vkontech.com/exploring-the-async-await-state-machine-the-awaitable-pattern/) for a great explanation).

Let's deconstruct what happens when the `await` keyword is encountered.

The compiler replaces calls to `await` with `Task.GetAwaiter()`, which returns an "awaiter" instance of type `TaskAwaiter`. 

What is an awaiter then? Conceptually, it is an object that blocks execution flow (note that it does not block the thread) until the async operation has completed and helps obtain the result of the operation. It is a bridge between the caller of the async method and the continuation that runs upon completion of the async method and helps maintain the illusion of synchronous control flow. JavaScript in its pre-await days was a mess of callbacks with flow of control jumping all around the place. Awaiter helps avoid all that and write async code just like we would write synchronous code, even though under the hood, the flow of control is jumping around between threads.    

The following code:
```
string text = await File.ReadAllTextAsync(filePath);
```
ultimately becomes (after transformations into a state machine as explained in the section below):
```
TaskAwaiter<string> awaiter = File.ReadAllTextAsync(filePath).GetAwaiter();
```
followed by an eventual call to get the result of the async operation when the system signals that it is done (this happens in the continuation that was passed to the async operation):
```
string text = awaiter.GetResult();
```

As Stephen Toub [explains](https://devblogs.microsoft.com/pfxteam/await-anything/), awaiter is a pattern that the language supports to await any instance that exposes a `GetAwaiter` method. In the case of `Task`, `GetAwaiter` returns a `TaskAwaiter` instance. But you could write your own class `Bar` that exposes an awaiter pattern by just implementing a `GetAwaiter` method that returns a `BarAwaiter` instance that implements the `INotifyCompletion` interface and exposes three members:
```
bool IsCompleted { get; }
void OnCompleted(Action continuation);
TResult GetResult(); // TResult can also be void
```

### Takeaways


### Further Reading

### NOTES:
- [Chaining tasks using continuation tasks](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks)
- [Parallel Computing - It's All About the SynchronizationContext](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)
- [A Tour of Task, Part 7: Continuations](https://blog.stephencleary.com/2015/01/a-tour-of-task-part-7-continuations.html)