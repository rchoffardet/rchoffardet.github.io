---
title: "Le pattern matching"
date: 2023-04-17T11:40:21+02:00
draft: true
---

# Le Pattern matching
## Avant C# 7.0 (utilisable classiquement)

Avec le mot-clé `is` il est possible de déterminer si une variable est d'un certain type ou d'un de ces dérivés.
`foo is Bar` est l'équivalent de : `typeof(Bar).IsAssignableFrom(foo.GetType())`


Il existe un mot-clé "complémentaire" au `is`: le mot-clé `as` qui permet de faire du safe casting (retourne `null` à la place du lancement d'une exception)
```csharp
// le cast "normal" qui lance une exception en cas d'échec
var foo = (string) bar;
// le case "safe" qui retourne null en cas d'échec
var foo = bar as string;
```

Cette fonctionnalité est bien pratique pour changer un comportement en fonction du type dérivé :

```csharp
if(foo is string)
{
    var fooString = foo as string;
    DoSomething(fooString);
}
else if(foo is Bar)
{
    var fooBar = foo as Bar;
    DoSomethingElse(fooBar);
}
```
Dans le cas ci-dessus, on peut utilier le cast normal puisqu'on vérifie bien qu'il soit possible avant.

## Avec C# 7.0 (utilisable en webfarm en modifiant le csproj)

Avec le passage à C# 7.0, il est désormais possible de vérifier le type d'une variable et d'en affecter une autre, du bon type :
```csharp
if(foo is string fooString)
{
    DoSomething(fooString);
}
else if(foo is Bar fooBar)
{
    DoSomethingElse(fooBar);
}
```
On gagne un peu en verbosité :)

##  C# 8/9/10 (uniquement sur le cloud, pour le moment)

Les "nouvelles" versions de C# ont apporté plusieurs autres types de pattern, en plus du pattern de type qu'on a vu avant.

### les patterns logiques
- `foo is not A`
- `foo is (A or B)`
- `foo is ((A or B) and not C)`

### les patterns sur des valeurs constantes ou relative à des constantes
- `foo is null`
- `foo is "foo"`
- `foo is 42`
- `foo is > 10 and < 100`

Ces comparaisons sont déjà possibles avec les opérateurs logiques "classiques" : `==`, `>`, `<=`, ...
Mais leur force réside dans le fait qu'ils soient cumulables aux autres et utilisables dans les `switch` :

```csharp
// type: object
switch(obj) 
{
    case (string s):
        return s;
    case (int i and < 100) :
        return "petit";
    default:
        return "autre";
}
```

### les patterns sur le contenu (item, propriété, etc)
- pattern sur une propriété : `foo is { Length: >= 2}` 
- pattern sur un tuple `foo is ("tuple", < 42)`
- pattern sur une collection `foo is [1,2,3]`

Pour les tuples et les collections, il est possible d'ignorer une vérification :
`foo is (0, _, 2)`

Enfin, pour les collections, il est possible d'ignorer une série de valeur :
`foo is [0, .., 4, .., 6]` matchera la collection `[0, 1, 2, 3, 4, 5, 6]`

## Expression de switch

Dans le pavé du dessus, j'ai donné les exemples avec des blocs de `switch`, mais avec les nouvelles versions de C#, il est possible d'utiliser des expressions de `switch`.
on gagne encore, niveau verbosité.

Si je traduis :
```csharp
switch(obj) 
{
    case (string s):
        return s;
    case (int i and < 100) :
        return "petit";
    default:
        return "autre";
}
```
j'obtiens :
```csharp
return obj switch{
    (string s) => s,
    (int i and < 100) => "petit",
    _ => "autre",
};
```