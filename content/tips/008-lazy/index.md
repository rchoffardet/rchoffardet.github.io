---
title: "Evaluation paresseuse"
date: 2023-04-16T11:40:21+02:00
draft: true
---

Nous faisons régulièrement des traitement lourds, et il arrive parfois qu'on final nous en n'ayons pas besoin, ou bien nous soyons amené à le faire plusieurs fois lors de l'exécution.  
Dans ces deux cas de figure, ce n'est pas optimal et il existe des parades.

## Le backing field
Quand je suis arrivé chez EI, c'est la technique que nous utilisions, et ça se présente sous cette forme :
```csharp
class Foo
{
    private string backingfield = null;
    public string Output { get { return backingfield == null ? backingfield = GetResult() : backingfield; }}

    private string GetResult()
    {
        Console.WriteLine("loaded.");
        return "blop";
    }
}
```
Si on l'appelle plusieurs fois la propriété : 
```csharp
var foo = new Foo();
Console.WriteLine(foo.Output);
Console.WriteLine(foo.Output);
```
La méthode, elle n'est appelée qu'une seule fois :
```text
loaded.
blop
blop
```
## `Lazy<T>`
Depuis .NET Frameowork 4.X il est possible d'utiliser un type spécialement prévu pour ce cas : `Lazy<T>` et ça se présente de la forme suivante :

```csharp
class Foo
{
    public readonly Lazy<string> output;

    public Foo()
    {
        output = new Lazy<string>(() => GetResult());
    }

    private string GetResult()
    {
        Console.WriteLine("loaded.");
        return "blop";
    }
}
```
A l'usage ça donne ça :
```csharp
var foo = new Foo();
Console.WriteLine(foo.output.Value);
Console.WriteLine(foo.output.Value);
```
Le `Value` est nécessaire puisqu'on expose le `Lazy<T>` directement. (On pourrait le masquer via une propriété intermédiaire à nouveau)

Et la sortie :
```text
loaded.
blop
blop
```

### Peut être utiliser de façon trompeuse  :warning: 
La raison de mon poste est qu'il peut être utilisé de manière trompeur :
```csharp
private Lazy<PublicationTask> LastPublicationLazy
{
     get { return new Lazy<PublicationTask>(GetUserLastPublication); }
}
```
Dans ce cas, un nouveau lazy est créé à chaque fois, et le résultat n'est donc pas stocké :(
En l'occurence, ce problème provoque l'exécution successive de 4 fois la même requête, au lieu d'une seule fois :/