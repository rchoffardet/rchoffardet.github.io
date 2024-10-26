---
title: "The danger of async void"
date: 2024-10-26T16:09:09
draft: false
summary: 
tags: 
 - C#
 - dotnet
 - lowering
 - async
 - void
 - await
---

# The danger of `async void`

_You know those innocent little errors that end up bringing production to its knees?  
Well, let me tell you the story of how I managed to do just that... thanks to `async void`.  
Yes, it was a moment of glory - or not really._

First of all, you should be aware that the use of `async void` is notorious and strongly discouraged, especially in ASP.NET. [David Fowler](https://www.linkedin.com/in/davidfowl/) (Architect on the .NET Team) discusses this in his [Async Guidance](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#async-void).

## What is `async void` and why did I use it despite the warnings?

When writing asynchronous methods, the method is markes as `async` and its return type is usually either `Task` or `Task<T>`. But it's also possible to return `void` instead of a task. This happens when trying to use synchronous code you don't own (or can't change) while making asynchronous operation.

In my case, I was registering an asynchronous callback as a synchronous method to handle configuration changes:

```C#
public void Register(IOptionsMonitor<MyServiceOptions> options)
{
    // The callback is supposed to be synchronous
    options.Onchange(async () => { /* ... */ });
}
```
The .NET API doesn't allow an asynchronous delegate to be passed here. It's [a known issue](https://github.com/dotnet/runtime/issues/69099) (filed by David Fowler) that has been reporterd a few years ago, in the .NET runtime's GitHub repository.

David speaks of two potential workarounds: either use `async void` or block the thread. In my work context (I work in a high QPS and low-latency environment), blocking the thread is not an option. So, I opted for `async void`, thinking it would be fine...

Spoiler alert: it wasn't.

## What happened?

So, I had a asynchronous callback registered to this synchronous `OnChange` method and because of a bug a change of configuration threw an exception. This immediately caused all apps running the code to crash, and the bug prevented them to restart properly. Fortunately, the configuration change was reverted and the apps were able to restart 😰

The bug was already quite severe but the `async void` callback caused the discontinuity of service.

## Why throwing an exception in a `async void` method makes the app crash?

That's complicated because it's not always true. For example it does not for console or GUI apps, but it does for ASP.NET apps by default. The reason is because there is no `SynchronizationContext` in ASP.NET.

### What is the `SynchronizationContext`?

The `SynchronizationContext` ensures that an async method that started on the UI thread, will continue its execution on the same one once the async call has been awaited. This has the advantage of avoiding multi-threading issues, but at the cost of lower performance (and sometimes other issues). In ASP.NET, there is no UI thread and there is no need for to end executing on the same thread. On the other hand, we want to maximize the performance of our apps, hence, there is no need for the `SynchronizationContext`.

### What SynchronizationContext has to do with `async void`?

The obvious difference between `async void` and `async Task` is their return type. In the former method, there is none. This is a major difference for exception handling.

In a `async Task` case, when an exception occurs in the async call, it can't really be handled immediatly because it's not happening in the same context of execution, and probably not on the same thread. Instead, the exception is stored in the `Task` instance and will rethrown once the async call is awaited and the execution of the caller is resumed.

In a `async void` case, there is no `Task`. Instead, the exception is stored in the `SynchronizationContext` as a fallback. But in ASP.NET (with no `SynchronizationContext`), the application can't do anything with the exceptions, and crashes.

It's quite surprising at the first. At least, it surpising to me that the application decides to crash by itself. But diving in the .NET internals proved it. Let's do it again, together.

## Diving into .NET internals

Before that, you must know that async/await is one the high-level feature that has no meaning in the .NET CLR, but does in C#. To be able to execute async code on the CLR, the compiler has to translate it into "low-level" code. This step is called "lowering". It is completely transparent to the developer, but if we want to understand what's going on, we have to be able to see it. To do so, I use a quite handy online tool called [sharplab.io](https://sharplab.io/). 

### Lowering a "normal" async method

```C#
using System.Threading.Tasks;

public class MyClass {
    public async Task NormalAsync() {
        await Task.Yield();
    }
}
```
See the lowered version [here](https://sharplab.io/#v2:CYLg1APgAgDABFAjAVgNwFgBQWoGYEBMcAsgJ4DCANgIYDOtcA3lnKwvlABwIBscAcgHsATgFtqlAIK1SAOwDGACgCUTFmw1QAnLwB0ATQCWAU0rAVGTBoC+Wa0A).

I won't describe how this all works, it might be the subject of another article one day. Let's focus on our issue.

The main points of focus, are:
 1. The async method body is converted in and replaced by a state machine
```C#
public Task NormalAsync()
{
    <NormalAsync>d__0 stateMachine = new <NormalAsync>d__0();
    stateMachine.<>t__builder = AsyncTaskMethodBuilder.Create();
    stateMachine.<>4__this = this;
    stateMachine.<>1__state = -1;
    stateMachine.<>t__builder.Start(ref stateMachine);
    return stateMachine.<>t__builder.Task;
}
```
 2. An `AsyncTaskMethodBuilder` instance is created and assigned to the state machine
```C#
stateMachine.<>t__builder = AsyncTaskMethodBuilder.Create();
```
 3. The same instance is responsible of dealing with the exception when one occurs
```C#
catch (Exception exception)
{
    <>1__state = -2;
    <>t__builder.SetException(exception);
    return;
}
```

So let's what this does by looking at the source of the .NET runtime!

[AsyncTaskMethodBuilder.cs,119](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncTaskMethodBuilder.cs,119):
```c#
public void SetException(Exception exception) => AsyncTaskMethodBuilder<VoidTaskResult>.SetException(exception, ref m_task);
```
Nothing interesting, let's follow the call.

[AsyncTaskMethodBuilderT.cs,509](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncTaskMethodBuilderT.cs,509):
```C#
internal static void SetException(Exception exception, ref Task<TResult>? taskField)
{
    /* ... */

    bool successfullySet = exception is OperationCanceledException oce ?
        task.TrySetCanceled(oce.CancellationToken, oce) :
        task.TrySetException(exception);

    /* ... */
}
```
You can follow these calls, but to keep things short, I'd say it's enough to prove that the exception is stored in the task when it arises during the task execution.

### Lowering an async void method

Let's lowering this code:

```C#
using System.Threading.Tasks;

public class MyClass {
    public async void AsyncVoid() {
        await Task.Yield();
    }
}
```
See the lowered version [here](https://sharplab.io/#v2:CYLg1APgAgDABFAjAVgNwFgBQWoGYEBMcAsgJ4DCANgIYDOtcA3lnKwvlABwIAscAgrVIA7AMYA1APYBLYAAoAlExZtVUAJwIAbADoAmtICmleQoyZVAXyyWgA==).

The main difference between the two codes is this line:
```C#
stateMachine.<>t__builder = AsyncVoidMethodBuilder.Create();
```
Check [here](https://www.diffchecker.com/OS465t7y/) for the diff.

So the type of the class changed. Let's see if the behavior changed as well for handling the exception.

[AsyncVoidMethodBuilder.cs,111](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncVoidMethodBuilder.cs,111):

```C#
public void SetException(Exception exception)
{
    /* ... */

    SynchronizationContext? context = _synchronizationContext;
    if (context != null)
    {
        // If we captured a synchronization context, Post the throwing of the exception to it
        // and decrement its outstanding operation count.
        try
        {
            Task.ThrowAsync(exception, targetContext: context);
        }
        finally
        {
            NotifySynchronizationContextOfCompletion(context);
        }
    }
    else
    {
        // Otherwise, queue the exception to be thrown on the ThreadPool.  This will
        // result in a crash unless legacy exception behavior is enabled by a config
        // file or a CLR host.
        Task.ThrowAsync(exception, targetContext: null);
    }

    // The exception was propagated already; we don't need or want to fault the builder, just mark it as completed.
    _builder.SetResult();
}
```

Of course it does!  
First, it checks if there is a `SynchronizationContext` with a null check.  
Then, if there isn't and there is no special config to handle this case, the app crashes (I'll let follow the call to see how)🤯.

## How to prevent the app from crashing?

We saw that I had to alternatives to chose from, either:
- Block the thread
- Use `async void`

The first option is not an option, and the second option isn't an option either.
Let's introduce a third option: use `Task.Run()` to execute the task:

```C#
public void Register(IOptionsMonitor<MyServiceOptions> options)
{
    // This time the callback is synchronous
    options.Onchange(() => {
        Task.Run(async () => { /* but this part is asynchronous */ });
    });
}
```
It's not perfect, though: as we aren't awaiting the async call, we won't be able to observe the exceptions and thus it won't be possible to take action if one occurs.

This can be worked around with a continuation task:

```C#
public void Register(IOptionsMonitor<MyServiceOptions> options)
{
    // This time the callback is synchronous
    options.Onchange(() => {
        Task.Run(async () => { /* but this part is asynchronous */ })
            .ContinueWith(
                task => { /* do something, like logging */}, 
                TaskContinuationOptions.OnlyOnFaulted
            );
    });
}
```

## Conclusion

Moral of the story: `async void` is quite dangerous, there are ways to avoid it and it's largely preferable to do so than to take the risk of crashing the app.

I learned a great deal during the investigation and I cantt recommend you enough to dive into .NET internals and really understand how things work. There are a few features that are high-level and lowering them is always a great way to learn.

I can't recommend you to break your apps (🤪) but when you do so (and you will probably do and we all do), it's a great opportunity to deepen your knowledge, so take it!

I hope this article made you learn a few things, and will keep you from crashing the prod!
