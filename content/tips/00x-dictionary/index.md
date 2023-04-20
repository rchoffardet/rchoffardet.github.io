---
title: "Qu'est-ce qu'un dictionnaire dans .NET ?"
date: 2023-04-19T11:40:21+02:00
draft: true
---

> Qu'est-ce  qu'un dictionnaire ?

On pourrait représenter un dictionnaire comme étant une collection de couples clé/valeurs, ou autrement dit un tableau de deux colonnes au la première serait la clé (unique) et la deuxième la valeur. 
On y insère une valeur en ajoutant une nouvelle ligne (et donc également une clé). On supprime la valeur en supprimant la ligne. Associer une nouvelle valeur à une clé détruit l'ancienne valeur puisqu'on réécrit sur la ligne.

> Comment est-ce que la clé et la valeur sont associées ?

Il exisent plusieurs façon différentes de faire l'association. On y retrouve les arbre (`Trie`) comme l'arbre binaire ou le B-Tree et des tables de hachage.
En l'occurence en C#, ce sont des tables de hachage qui sont utilisés pour les dictionaires.

Ces tables ressemblent au tableau que l'on décrit ci-dessus, mais au lieu d'y placer clé en première colonne et valeur en deuxième colonne, c'est le hash qui est en première colonne et une collection de blob de données (appelé `bucket`) comprenant la clé et la valeur.

> Pourquoi une collection et pas juste le `bucket` ?

Comme on utilise des hashes de la clé et pas la clé directement, il est possible que nous tombions sur deux clés qui produisent le même hash. Il faut alors comparer les clés entre-elles pour s'assurer qu'on récupère le bon bucket et donc la bonne valeur.
On stocke donc les différents buckets dont la clé produit un hash identique dans une même collection (une liste en l'occurence) pour pouvoir itérer dessus plus tard.

# à refaire
> Comment est-ce qu'on pourrait l'implémenter ?

Une implémentation littérale de la représentation ci-dessus serait une liste de clé/valeur (ex: `List<KeyValuePair<Key, Value>>`). Cela dit, pour retrouver le couple clé/Valeur, il faut parcourir la liste depuis le début, ce qui présage d'une limitation forte en terme de performance sur toutes les opérations : lecture, suppression, modification et création (il faut s'assurer que la clé ne soit pas déjà présente).

Il existe plusieurs techniques pour faire correspondre une clé et une valeur : 
 - les arbres (`Trie`) notamment le B-Tree utilisé par des SGBD et systèmes de fichiers 
 - les tables de hachage (`HashMap`) qui nous intéresse ici

Une table de hachage fait correspondre le hash de la clé à la valeur plutôt que la clé elle-même. Un hash est le résultat d'une fonction de hachage et en limitant leur taille à 64 bits par exemple, on peut s'en servir en tant qu'index d'un array.
