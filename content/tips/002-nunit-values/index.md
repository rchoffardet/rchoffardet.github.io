---
title: "Paramêtrer ses tests avec l'attribut `[Values()]` avec NUnit"
date: 2023-04-10T11:40:21+02:00
draft: true
---

Si vous utilisez NUnit, vous connaissez sûrement les tests cases, qui permettent de jouer plusieurs fois le même test avec des données différentes.
Mais connaissez-vous l'attribut [`[Values()]`](https://docs.nunit.org/articles/nunit/writing-tests/attributes/values.html) qui se combine avec les autres paramètres ?  
Exemple :
```csharp
[Test]
public void Mon_test(
    [Values("a", "b)] string text,
    [Values(1, 2, 3)] int number
)
{ /* ... */ }
```
Le test va s'exécuter 6 fois : "a1", "a2", "a3", "b1", "b2", "b3" !