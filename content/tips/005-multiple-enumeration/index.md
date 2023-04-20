---
title: "L'énumération multiple de `IEnumerable`"
date: 2023-04-13T11:40:21+02:00
draft: true
---


Si je vous montre le code suivant, combien de fois diriez vous que "traitement lourd" s'affiche ?
```csharp
IEnumerable<int> Items()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

var items = Items()
    .Where(x => x > 1)
    .Select(x => { 
        Console.WriteLine("traitement lourd");
        return x;
    });

if(items.Count() > 0)
{
    Console.WriteLine(string.Join(",", items));
}
```

Si vous n'êtes pas familier avec la problématique, vous répondrez en toute logique : "2 fois"
Mais c'est faux : il s'affichera __4 fois__.

Pourquoi ? Eh bien parce que les méthodes de Linq comme `Where` et `Select`, s'éxécutent à chaque itération de `IEnumerable`.
Et qu'ici, on itère `ìtems` deux fois entièrement (une fois avec le `Count` et une autre avec le `Join`).

Comment régler le problème ? 
Eh bien, premièrement, il n'est pas nécessaire de compter entièrement `items` pour savoir s'il est vide ou pas.
Le code précédent est remplaçable par :
```csharp
// ...
if(items.Any())
// ...
```
Ce qui ne règle pas entièrement le problème, on énumérera quand même une deuxième fois, mais on s'arrêtera au premier élément.  
"traitement lourd" s'affichera __3 fois__.

La deuxième solution (cumulable à la première) est de stocker le résultat de l'énumération en mémoire (via `ToArray` par exemple).

```csharp
IEnumerable<int> Items()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

var items = Items()
    .Where(x => x > 1)
    .Select(x => { 
        Console.WriteLine("traitement lourd");
        return x;
    }).ToArray();

if(items.Any())
{
    Console.WriteLine(string.Join(",", items));
}
```
Le code ci-dessus affichera "traitement lourd" uniquement __2 fois__.