---
title: "Erro handling in .NET Part2: The Try and the Result patterns"
date: 2023-04-20T11:40:21+02:00
draft: false
summary: | 
    Le tableau est la structure de données la plus connues et la plus utilisée, actuellement.  
    Mais savez-vous différencier les listes et les tableaux et quand les utiliser ?
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

The `out` parameter is quite neat and was necessary back then: The "new" (introduced in C# 7.0) tuple notation (ex: `(string, int)`) leverages a new structure the `ValueTuple`, which is a value type instead of the "old" `Tuple` that was a reference type. There is a lot to say about both value and reference type, this a not the subject of this blog post (may be a future one, though). The big difference between `ValueTuple` and the regular `Tuple` is that the first one is stack allocated whereas the latter is heap allocated. This means that every time you create a new `Tuple`, it has to be regularly checked up by the garbage collector (GC), and destroyed when it becomes unused. This puts pressure on the GC, and overall it's bad performance-wise when it happens too often. Of course, it's even worse on a hot path.

## The Result pattern



// TODO make an example with the three methods + valuetuple / tuple
// and benchmark them