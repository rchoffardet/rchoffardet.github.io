---
title: "Les constructeurs nommés"
date: 2023-04-11T11:40:21+02:00
draft: true
---

Il existe une bonne pratique pour rajouter un peu de sémantique dans le code qui consiste à exposer des constructeur nommés (des méthodes statiques qui instancient la classe).

Par exemple, j'ai une classe qui permet de whitelister ou blacklister des valeurs :
```csharp
public class AccessList<T>
{
    private T[] elements;
    private bool isWhitelist;

    public AccessList(T[] elements, bool isWhitelist)
    {
        this.elements = elements;
        this.isWhitelist = isWhitelist;
    }

    /* ... */
}
```
Avec des constructeurs nommés, on rajoute un peu de sens, et on évite d'avoir un `bool` qui traîne.

En rajoutant :
```csharp
public static AccessList<T> Allow<T>(params T[] elements)
{
    return new AccessList<T>(elements, true);
}
```
On peut remplacer :
```
var items = new AccessList<int>(new []{1,2,3}, true);
```
Par :
```
var items = AccessList<int>.Allow(1,2,3);
```
#### Petits tips supplémentaires :  
Le ou les types génériques peuvent devenir encombrant, que ce soit via l'instanciation classique ou via le constructeur nommé. On peut s'en débarrasser, en créant une classe statique supplémentaire pour perdre le ou les paramètres génériques (perso, je rajoute cette classe à la suite de l'autre) :
```csharp
static class AccessList
{
    static AccessList<T> Allow<T>(params T[] elements)
    { /* ... */ }
}
```
Elle va permettre d'omettre le paramètre générique par inférence du compilateur :
```csharp
var items = AccessList.Allow(1,2,3);
```
Enfin, on peut utiliser `using static Namespace.Vers.Ma.Classe.AccessList` qui permet d'omettre le nom de la classe :
```csharp
var items = Allow(1,2,3);
```