---
layout: post
category: blog
title: Async Programming Part 2
permalink: /blog/async-programming-2
description: Async Programming Part 2
image: async2.jpg
---

![Async Programming](../img/async2.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@edenconstantin0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Eden Constantino</a> on <a href="https://unsplash.com/photos/person-holding-purple-and-pink-box-iJg1YzsEfqo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a></span>
  
In [Part 1](/blog/async-programming-1), we saw what .NET async programming is and when and how to use it. It all seems to work like magic but there's a lot going on underneath, just like in an automobile. Let's open the hood and look inside.

As our eyes squint and adjust to the gloom, we notice parts inside labeled *Task, continuations, awaiter, state machine*, etc. Let's examine them in bite sizes one at a time and then see how they all fit together to form a magnificent contraption. 

### Task Type

The `Task` and `Task<T>` types are at the heart of async programming. `Task` is used for operations that don't return a value. `Task<T>` is used for operations that return a value of type `T`. But what exactly does a task object represent conceptually? 

It is a promise or future value for an async operation - a token, or rather a bag for a value that will be available at a later time. When the operation completes the result magically appears within the bag, and the bag turns green. If there was a snafu, then it turns red and tells you what went wrong in a sad voice.  

The `Task`/`Task<T>` objects are not quite as dramatic but have properties for the result, exception if any, and statuses to indicate completion, failure and cancelation.  `Task<T>` inherits from `Task`.

Typical `Task<T>` constructors look like this:

```clojure
Task<TResult>(Func<Object,TResult> doSomething, Object inputToDoSomething);
Task<TResult>(Func<Object,TResult> doSomething, Object inputToDoSomething, CancellationToken cancellationToken);
```
where `doSomething` is a delegate (a function variable) that contains the code to run in the task, `inputToDoSomething` is the input to the delegate, and `cancellationToken` is used to signal the delegate to abort.

Tasks can be started, waited on, canceled or yielded (another fascinating tangent). Starting a task schedules it for execution using the TaskScheduler, while queues it up on the thread pool.

### Successful Task

Here we create a task, start it and wait for it to be done:
```clojure
// task with parameter that completes successfully
string TalkTrash(object? name)
{
    return $"How's it going, {name}?";
}

var task = new Task<string>(TalkTrash, "Alice");

task.Start();
task.Wait();

var result = task.Result;
Console.WriteLine($"task returned: {result}");

PrintTask(task);
```

This code prints the following when run:
```text
task returned: How's it going, Alice?
task =
{
  "Result": "How's it going, Alice?",
  "Id": 1,
  "Exception": null,
  "Status": 5,
  "IsCanceled": false,
  "IsCompleted": true,
  "IsCompletedSuccessfully": true,
  "CreationOptions": 0,
  "AsyncState": "Alice",
  "IsFaulted": false
}
```

The result of the task, represented by the return value of the `TalkTrash` function, is contained in `task.Result`. The `Status` value of `5` indicates `RanToCompletion`, `IsCompleted` indicates the task is done, `IsCompletedSuccessfully` indicates it was successful, and `AsyncState` contains the original input to the function.

### Failed Task

What would a task instance gone bad look like?
```clojure
// faulted task
void ThrowUp()
{
    throw new Exception("I'm sick");
}

var task2 = new Task(ThrowUp);

try
{
    task2.Start();
    task2.Wait();
}
catch (Exception e)
{
    Console.WriteLine("e = " + e.Message);
}

PrintTask(task2);
```

Like this:
```text
e = One or more errors occurred. (I'm sick)
task =
{
  "Id": 2,
  "Exception": {
    "ClassName": "System.AggregateException",
    "Message": "One or more errors occurred.",
    "Data": null,
    "InnerException": {
      "ClassName": "System.Exception",
      "Message": "I'm sick",
      ...
    },
		...
  },
  "Status": 7,
  "IsCanceled": false,
  "IsCompleted": true,
  "IsCompletedSuccessfully": false,
  "CreationOptions": 0,
  "AsyncState": null,
  "IsFaulted": true
}
```

The `Status` value is `7` (`Faulted`), `IsCompleted` indicates the task finished, `IsCompletedSuccessfully` and `IsFaulted` tell us it didn't complete successfully due to an unhandled exception, and `Exception` contains the exception details.  

### Canceled Task

If a task is taking too long, we can cancel it.
```clojure
// canceled task
var tokenSource = new CancellationTokenSource();
var token = tokenSource.Token;
void LongTask()
{
    for (int i = 0; i < 100; i++)
    {
        Console.WriteLine(i);
        Thread.Sleep(100);
        // Abort if cancellation request received 
        if (token.IsCancellationRequested)
            token.ThrowIfCancellationRequested();
    }
}

var task3 = new Task(LongTask, token);
try
{
    task3.Start();

    Thread.Sleep(1000);

    // Cancel the task
    tokenSource.Cancel();

    task3.Wait();
}
catch (Exception e)
{
    Console.WriteLine("e = " + e.Message);
}

PrintTask(task3);
```

This outputs:
```text
0
1
2
3
4
5
6
7
8
9
task =
{
  "Id": 3,
  "Exception": null,
  "Status": 3,
  "IsCanceled": false,
  "IsCompleted": false,
  "IsCompletedSuccessfully": false,
  "CreationOptions": 0,
  "AsyncState": null,
  "IsFaulted": false
}
e = One or more errors occurred. (A task was canceled.)
task =
{
  "Id": 3,
  "Exception": null,
  "Status": 6,
  "IsCanceled": true,
  "IsCompleted": true,
  "IsCompletedSuccessfully": false,
  "CreationOptions": 0,
  "AsyncState": null,
  "IsFaulted": false
}
```
We are printing the task before canceling and after it returns. Notice that while the task is still running, `IsCompleted` is false.

When the task completes, `Status` is `6`(`Canceled`), `IsComplete` is `true` meaning the task is done, but `IsCompletedSuccessfully` is false meaning it wasn't all good, and `IsCanceled` tells us that it was cancelled.

### Tasks as monads

This is all wonderful, but there's a lot more to `Task`. The `Task` (and `Task<T>`) types are monads. Monads are a concept from the dark corridors of functional programming. They encapsulate structured data, and allow composing operations that accept and return the same structure (by returning copies of it rather than mutating it) through a `bind` function. They are complex enough to deserve their own blog post(s). Maybe in this very space in some distant future. (For those not faint of heart, Eric Lippert has a [13 part series](https://ericlippert.com/2013/02/21/monads-part-one/)!) 

`Task` has a `ContinueWith` method which acts like `bind`, allowing chaining of async operations. It takes a `Task` input (known as the antecedent task) and returns a `Task` (the continuation).

```clojure
// continuation task
string AskName()
{
    return "Bob";
}

var antecedent = new Task<string>(AskName);
var continuation = antecedent.ContinueWith(antecedent =>
{
		// Note that the continuation runs following the completion of
		// the antecedent and thus has access to its result
    Console.WriteLine($"Hi {antecedent.Result}!");
});

antecedent.Start();
antecedent.Wait();

PrintTask(antecedent, "antecedent");
PrintTask(continuation, "continuation");
```

This outputs:
```text
Hi Bob!
antecedent task =
{
  "Result": "Bob",
  "Id": 4,
  "Exception": null,
  "Status": 5,
  "IsCanceled": false,
  "IsCompleted": true,
  "IsCompletedSuccessfully": true,
  "CreationOptions": 0,
  "AsyncState": null,
  "IsFaulted": false
}
continuation task =
{
  "Id": 5,
  "Exception": null,
  "Status": 5,
  "IsCanceled": false,
  "IsCompleted": true,
  "IsCompletedSuccessfully": true,
  "CreationOptions": 0,
  "AsyncState": null,
  "IsFaulted": false
}
```

This behavior is important for the operation of the async machine as it hops from one async operation to another. It enables multiple async operations to be chained together to give the illusion of synchronous flow. This provides a convenient segue into continuations which we'll cover in Part 3.

### Takeaways

- The `Task` type represents the result and state of an asynchronous operation, i.e. a future or promise.
- Task provides monad like behavior through its `ContinueWith` method, which allows tasks to be composed and chained together and provides the glue that holds the async machine together. 

### Further Reading
- [Task Class](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task?view=net-8.0)
- [Task return type](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)
- [Jeremy Clark Explains Task, Await and Asynchronous Methods in C#](https://visualstudiomagazine.com/Articles/2023/05/15/asynchronous-programming.aspx)
- [Useful Abstractions Enabled with ContinueWith](https://devblogs.microsoft.com/pfxteam/useful-abstractions-enabled-with-continuewith/)
- [Tasks, Monads and LINQ](https://devblogs.microsoft.com/pfxteam/tasks-monads-and-linq/)
- [Monads, part one](https://ericlippert.com/2013/02/21/monads-part-one/)
- [The Marvels of Monads](https://learn.microsoft.com/en-us/archive/blogs/wesdyer/the-marvels-of-monads)
- [Maybe monad through async/await in C# (no Tasks!)](https://habr.com/en/articles/458692/)