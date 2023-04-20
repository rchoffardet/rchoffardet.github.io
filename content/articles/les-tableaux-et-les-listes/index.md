---
title: "Les tableaux et listes en C#"
date: 2023-04-20T11:40:21+02:00
draft: false
summary: | 
    Le tableau est la structure de données la plus connues et la plus utilisée, actuellement.  
    Mais savez-vous différencier les listes et les tableaux et quand les utiliser ?
tags: 
 - C#
 - dotnet
 - Structure de données
 - System design
 - Collections
---

## La théorie

Dans un cursus d'ingénieur en informatique on apprend assez vite la différence théorique entre un tableau et une liste :
 - un tableau est une référence vers une plage en mémoire continue dont la taille équivaut à la taille d'un élément multiplié par la quantité d'élément dans ce tableau. Autrement dit : `sizeof(T) * Array.Length`. Si `T` est de type référence, il s'agira de la taille du pointeur, c'est-à-dire 4 ou 8 octets en fonction de l'architecture 32 ou 64 bits.
 - Une liste est une chaîne de blocs de mémoire discontinus reliés entre eux par un ou deux pointeurs (liste chaînée ou doublement chaînée).

## En pratique dans le framework dotnet

Pourtant, ce n'est pas tout à fait exact. Structurellement parlant, il n'y a aucune différence entre un tableau (`Array<T>`) et une liste (`List<T>`) et c'est d'ailleurs vérifiable dans [le code source du framework](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Collections/Generic/List.cs,25) :
```csharp
public class List<T> : IList<T>, IList, IReadOnlyList<T>
{
    // ...
    internal T[] _items; 
    // ...
}
```
La structure qui implémente la partie "liste" de la théorie est la suivante : [`LinkedList`](https://learn.microsoft.com/fr-fr/dotnet/api/system.collections.generic.linkedlist-1?view=net-7.0).

Cette erreur est assez répandue dans la communauté C# comme le témoigne les nombreux articles trompeurs à ce sujet :
- https://csharp-station.com/c-arrays-vs-lists/
- https://www.shekhali.com/c-array-vs-list/
- ...

## Comparaison entre un tableau, une liste et une liste chaînée
### Différence entre un tableau et une liste

La différence la plus importante entre tableau et liste est que la deuxième inclut un mécanisme de redimensionnement dynamique. Par défaut, La liste commence avec [une capacité de 4](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Collections/Generic/List.cs,23) éléments, puis une fois le seuil atteint, [la capacité est multipliée par 2](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Collections/Generic/List.cs,455).  
D'ailleurs, si on connait le nombre maximal d'éléments qu'aura la liste, il est possible d'instancier la liste en spécifiant cette capacité, cela permettra d'éviter les redimensionnements inutiles.

En revanche, il n'y a pas de réduction automatique de la capacité lors de la suppression d'un ou plusieurs éléments et à cause de son redimensionnement en puissance de 2, la liste gaspille de l'espace et occupe donc un peu plus de mémoire.

### Différence entre une listée et une liste chaînée

La différence principale entre une liste et une liste chaînée est la (dis)continuité de leur représentation en RAM : chaque maillon est relié à son voisin (ou ses voisins dans le cas d'une liste double chaînée) par un pointeur/référence.

Ainsi, la liste chaînée n'a pas de contrainte de taille, puisqu'il suffit de rajouter un maillon. Elle permet également d'insérer aisément un maillon en ne modifiant que la référence de son futur voisin. En revanche, elle ne permet pas un accès aléatoire aux éléments, il faut boucler depuis le début de la liste, itération qui est relativement lente, puisqu'il n'est pas possible de la mettre en cache CPU en une fois.

Comparativement, la liste simple profite des mêmes avantages qu'un tableau en terme d'accès aléatoire et d'itération, mais nécessite un redimensionnement lors d'un ajout en dehors de sa capacité et d'une copie des éléments "à droite" lors d'une insertion.

## Conclusion

Aujourd'hui, il n'y a quasiment plus aucune raison d'utiliser des tableaux : la liste simple possède les mêmes avantages que le tableau avec des désavantages négligeables. Les tableaux sont là pour des raisons historiques et de rétrocompatibilité, mais leur représentation trop matérielle les rendent indésirables, comme expliqué dans [ce ticket](https://learn.microsoft.com/fr-fr/archive/blogs/ericlippert/arrays-considered-somewhat-harmful) par Eric Lippert, ex-membre de l'équipe en charge de C#.

Pour les listes chaînées, en théorie, elles auraient leur usage notamment pour les listes de `struct` lourd (ou équivalent) avec beaucoup de d'insertion/suppression et peu d'itérations. En pratique, le fait qu'il faille itérer sur la liste chaînée pour trouver l'endroit où insérer/supprimer un élément empêche de véritablement profiter de ses avantages.

Personnellement et par défaut, j'utiliserais désormais des listes dans mon code. Si je sens qu'il peut être préférable d'utiliser une liste chaîne, je pourrais l'utiliser et je confirmerais l'usage par un benchmark.