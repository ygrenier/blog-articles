---
title: ASP.NET MVC : binding de modèle à "null"
published: 2015-09-25
categories: ASP.NET
tags: ASP.NET, ASP.NET MVC, C#
url: /2015/09/asp-net-mvc-binding-de-modele-a-null
---

# ASP.NET MVC : binding de modèle à "null"

Petit rappel des règles de binding de modèle en ASP.NET MVC pour éviter de perdre des heures, comme je viens de le faire ;)

Soit notre modèle :

```csharp
public class Hardware
{
  public int ID{ get; set; }
  public String Manufacturer { get; set; }
  public String Model { get; set ;}
}
```

Notre action dans votre contrôleur:

```csharp
[HttpPost]
public ActionResult Update(Hardware model)
{
  if(ModelState.IsValid){
    // ...
  }
  return View(model);
}
```

Lorsque que vous postez votre formulaire, votre modèle est ```null```.

Cela est du au fait que le nom de votre argument représentant votre modèle est le même que celui d’une des propriétés de votre modèle.

Dans notre cas c’est ```model``` vs ```Hardware.Model```. En effet la priorité du binder est de mapper les éléments d’un formulaire vers les arguments de même nom. Dans notre cas ```Hardware.Model``` est une chaîne, et l’argument ```model``` est un ```Hardware``` qui n’est pas compatible avec une chaîne donc notre ```model``` est ```null```.

Nous n’avons qu’a modifier le nom de notre argument ```hardwareModel``` par exemple, et notre modèle est de nouveau bindé correctement.

A bientôt,

Yanos
