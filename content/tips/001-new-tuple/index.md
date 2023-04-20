---
title: "les nouveaux Tuple avec C# 7.3"
date: 2023-04-09T11:40:21+02:00
draft: true
---

Avec C# 7.3, on peut utiliser les "nouveaux" Tuples qui sont beaucoup plus sympas que les "anciens" :
```csharp
public static (string, int) GetFullnameAndAge()
{
    return ("John DOE", 23);
}

// Affectation par deconstruction
var (fullname, age) = GetFullnameAndAge();
Console.WriteLine($"My name is {fullname} and I'm {age} years old.");
``` 