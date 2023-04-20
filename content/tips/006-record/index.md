---
title: "le nouveau type `record` de C# 9"
date: 2023-04-14T11:40:21+02:00
draft: true
---

Première nouveauté dans les nouvelles fonctionnalités C# : les classes `record`.
Principalement adaptées pour les `ValueObject` : les objets dont l'identité est définie par la somme de ses champs, à l'inverse d'une `Entity` dont l'identité est définie par la valeur d'un champ (généralement `id`)
Ces classes se définissent sous cette forme :
```csharp
public record Person(
    string firstname,
    string lastname, 
    int age
)
{ }
```
Elle permet de générer :
- Une propriété par arguments
- Un constructeur avec les arguments du `record`
- Une méthode `Equals` et `GetHashCode`
- Un deconstructeur (pour le transformer en tuple : `(var firstname, var lastname, var age) = person`)

Le `record`est immutable par défaut, et lancera une erreur lorsqu'on voudra le modifier. Mais il est possible de le dupliquer et de modifier sa valeur lors de la copie, avec le mot-clé `with` :
```csharp
var john = new User("John", "Doe", 42); 
var paul = john with { firstname = "Paul" };
```