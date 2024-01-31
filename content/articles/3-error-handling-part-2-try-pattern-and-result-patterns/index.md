---
title: "Error handling in .NET Part2: The Try and the Result patterns"
date: 2024-02-01T00:10:00
draft: false
summary: 
tags: 
 - C#
 - dotnet
 - error handling
 - try pattern
 - result pattern
 - patterns
 - performance
---

# Alternatives to the Exception pattern

In [my last article](../2-error-handling-part-1-exception-pattern), I wrote about the exception pattern. It has great benefits but also great performance implications. Because of these, and following Microsoft recommendations, I've also written that we should only use them for exceptional errors. 

So what do we do when we have to handle an error? Let's get to it.

## The "Check and Do" pattern

_Disclaimer: I'll include this pattern just for completeness. I don't encourage the use of this pattern because of its poor performance and verbosity._

The principle behind this pattern is to check the requirements before doing the action, hence the name.
It has the merit of being easily understandable at the expense of being costly. 

If I want to parse an integer from a string, I must :
  1. check if the string only includes digits
  2. parse the string

When implementing this, we will have to loop twice over the string and most likely have to double some operations. Overall, doubling the cost of an algorithm just to avoid having to throw an exception is a bad choice. But that's still one way of doing it.

Another example is when we want to add a value to a dictionary without overwriting the potential existing value. Using the bracket syntax is not a suitable solution as it will overwrite the value. 

To do so, we have to :
 1. check if the value exists or not
 2. add if needed
Once again, we have a suboptimal algorithm.

You might know the `TryAdd` method (doc [here](https://learn.microsoft.com/fr-fr/dotnet/api/system.collections.generic.dictionary-2.tryadd?view=net-8.0)) which has been available in the framework since .NET Core 2.0. It is one example of the following pattern, the Try pattern.

## The Try pattern

I like to see this pattern as an improved version of the previous one. It does the same things: first checking then doing, but it does that at the same time, in one method call. Thus, it's more efficient.
The `TryAdd` method of a dictionary (doc [here](https://learn.microsoft.com/fr-fr/dotnet/api/system.collections.generic.dictionary-2.tryadd?view=net-8.0)) is an example of a write action: there is also a read part that we'll see a bit later.

This method will do the action and will return a boolean to signal whether or not the action was properly done. If we get `true` then we know that the key/value was added. If we get `false` the key already exists and thus the value wasn't changed. Having this information, we can handle our business case. 

What about reading value, like getting the value of a key that may not exist? 
A specific method exists for that, the `TryGetValue` (doc [here](https://learn.microsoft.com/fr-fr/dotnet/api/system.collections.generic.dictionary-2.trygetvalue?view=net-8.0)). This method has existed since .NET Framework 2.0 (back in 2005!) and uses a very old feature (from the initial release of .NET) the `out` parameters. The `out` parameter allows the developer to assign a value to a variable by passing it to a method. It's been improved in C# 7.0 to allow both declaration and assignment at the same time, like so:

```csharp
var dictionary = new Dictionary<int, string>();
if(dictionary.TryGetValue(42, out string value)) // value is both declared and assigned
{
    Console.WriteLine($"The key 42 exists with value: {value}");
}
```
Thanks to the returned boolean, we can safely call `Console.WriteLine` without taking the risk to throw a `KeyNotFoundException`.

The `out` parameter is quite neat and was necessary back then: The "new" (introduced in C# 7.0) tuple notation (ex: `(bool, int)`) leverages a new structure the `ValueTuple`, which is a value type instead of the "old" `Tuple` that was a reference type. There is a lot to say about both value and reference type, this a not the subject of this blog post (may be a future one, though). The big difference between `ValueTuple` and the regular `Tuple` is that the first one is stack allocated whereas the latter is heap allocated. This means that every time you create a new `Tuple`, it has to be regularly checked up by the garbage collector (GC), and destroyed when it becomes unused. This puts pressure on the GC, and overall it's bad performance-wise when it happens too often. Of course, that's especially true for hot paths.

## The Result pattern

In its most basic form, this pattern is more or less the same as returning the tuple `(bool, ReturnValue)`. If the first item is `true`, then `ReturnValue` is provided if not, it's null. Of course, the return value's type changes depending on the method, so it has to be a generic type:

```csharp
public struct Result<T>
{
    public bool IsSuccessFul { get; private set; }
    public T? Value { get; private set; }

    public Result(T Value)
    {
        IsSuccessful = true;
        Value = Value;
    }

    public Result()
    {
        IsSuccessful = false;
    }
}
```
There is no official implementation of this pattern, so you'll have to make your own or use some open-sourced ones. Although most of them are based on classes, I think that it's suboptimal: it produces allocations every time the method is called, which puts additional pressure on the GC for no reason. A small and short-lived bunch of data like this is a typical example where a `struct` is more appropriate. 

The first two patterns ("Check and do" and "Try") are a bit verbose and require conditional blocks. The Result pattern works around this drawback by leveraging a _fluent_ API. This kind of API lets you chain method calls to apply them successively to an initial value, such as Linq does with `IEnumerable<T>`. However, in the case of this pattern, the next link will only be executed if the previous one was successful. Instead of the `Select` as name for the method, let's name it `Then`.
```csharp
public static Result<U> Then<T, U>(this Result<T> result, Func<T, U> fn)
{
    return result.IsSuccessFul 
        ? new Result<U>(fn(result.Value))
        : new Result<U>();
}
```
As you can see, `fn` will only be executed if the result is successful. And that's the basics of the Result pattern. From there, there is a whole tree of possibilities for augmentations. For instance, you could:
- embed an error value, to contextualize the erroneous case
- add additional extension methods:
  - `Or` method that does the opposite of `Then` and allows you to handle a fallback value
  - `Switch`/`Match` to return one value in case of error and another in case of success
  - `Try` to wrap methods that throw exceptions and return a result instead 
  - ...

# Comparison between the patterns

Let's build the same example with the different patterns we saw: We have two methods `One` and `Two` that have to handle an erroneous case. `Two` takes the result of the first one as a parameter. If one of them fails we have to return a fallback value.  

## Exception pattern

```csharp
try
{
    var value = One();
    return Two(value);
}
catch(Exception)
{
    return fallbackValue;
}
```

## Check and Do pattern

```csharp
if(!CanDoOne())
{
  return fallbackValue;
}

var value = One();

if(!CanDoTwo(value))
{
    return fallbackValue;
}

return Two(value);
```

## Try pattern

```csharp
if(!TryOne(out var value) && !TryTwo(value, out var result))
{
    return fallbackValue;
}
return result;
```

## Result pattern

```csharp
return One().Then(x => Two(x)).Or(fallbackValue);
```

From what we saw, what's your favorite pattern?