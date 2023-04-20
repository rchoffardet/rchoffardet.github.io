---
title: "Dictionnaire et table de recherche"
date: 2023-04-14T11:40:21+02:00
draft: true
---

Il n'est pas rare de devoir faire des associations entre deux collections.
Par exemple, entre des messages et leurs auteurs : 
```csharp
var messages = GetMessages(/*...*/);
var authors = GetAuthors(/*...*/);

var messagesWithAuthor = messages.Select(
    message => (
        message, 
        authors.First(author => message.authorId == author.id)
   )
); // IEnumerable<(message, author)>
```
Le code ci-dessus est sub-optimal, car à chaque itération sur `messages` on parcourt `authors` à la recherche du bon élément.

Pour éviter de parcourir la collection à chaque fois, on peut la transformer en dictionnaire :
```csharp
var messages = GetMessages(/*...*/);
var authors = GetAuthors(/*...*/);
var authorsById = authors.ToDictionary(author => author.id, author => author);

var messagesWithAuthor = messages.Select(
    message => (
        message,
        authorsById[message.authorId]
   )
); // IEnumerable<(message, author)>
```

Autre situation, admettons, qu'on soit obligé de le faire dans l'autre sens :
```csharp
var messages = GetMessages(/*...*/);
var authors = GetAuthors(/*...*/);

var authorsWithMessages = authors.Select(
    author => (
        author,
        messages.Where(message => message.authorId == author.id).ToArray()
   )
);
```
Pareil qu'avant, on parcours les `messages` à chaque itération de `authors`, mais l'usage d'un dictionnaire est pas facile à cause du caractère multiple de `messages`, on peut utiliser des tables de lookup :
```csharp
var messages = GetMessages(/*...*/);
var messagesByAuthorId = messages.ToLookup(x => x.authorId);
// ILookup<Int, Message>

var authors = GetAuthors(/*...*/);

var authorsWithMessages = authors.Select(
    author => (
        author,
        messagesByAuthorId[author.id].ToArray()
   )
);
```
`ILookup<int, Message>` est une sorte  d'équivalent de `IDictionnary<int, IEnumerable<Message>>`


Enfin, si on doit discriminer par plusieurs valeurs (par exemple par `Author.Id` et `Author.Federation.Id`, on peut utiliser un `ValueTuple` en clé. 
Ça fonctionne avec un dictionnaire ou une table de correspondance (Lookup Table).

```csharp
var messages = GetMessages(/*...*/);
var authors = GetAuthors(/*...*/);
var authorsByIdAndFederationId = authors.ToDictionary(author => (author.id, author.federation.id), author => author);

var messagesWithAuthor = messages.Select(
    message => (
        message,
        authorsByIdAndFederationId[(message.authorId, message.federationId)]
   )
);
```

:warning: Attention cependant, seuls les types "valeurs"  (types primitifs, `struct`, ...)  fonctionneront en tant que clé, par défaut.
Les types référence (comme les classes) devront surchargé la méthode `Equals` et `GetHashCode` pour fonctionner.