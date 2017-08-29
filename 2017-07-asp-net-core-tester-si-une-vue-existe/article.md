---
title: [ASP.NET Core] Tester si une vue existe
published: 2017-07-20
categories: .NET Core, ASP.NET
tags: ASP.NET Core, ASP.NET MVC, C#
url: /2017/07/asp-net-core-tester-si-une-vue-existe
---

# [ASP.NET Core] Tester si une vue existe

Depuis la version d'ASP.NET Core (MVC 6) les anciens codes pour tester l'existence d'une vue ne fonctionnent plus du tout.

En recherchant un peu sur Internet on trouve essentiellement des tests d'existence des fichiers dans le dossiers "~/Views" de l'application. Toutefois le système de vue utilise les <a href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/file-providers">File Providers</a> pour obtenir les fichiers de vue. C'est notamment mon cas dans un projet où des vues sont "incorporées" dans les DLLs il faut donc ajouter des "EmbeddedFileProvider" (voir mon <a href="http://blog.ygrenier.com/2017/06/asp-net-core-editer-les-vues-razor-embarquees-dans-une-librairie/">post précédent</a>).

Par conséquent le test de fichier ne fonctionne pas dans ce cas.

Ma technique est tout simplement de demander au moteur de vue si la vue existe.

<!--more-->

## Le code

Le code n'est pas bien compliqué je me suis basé sur l'objet  <a href="https://github.com/aspnet/Mvc/blob/master/src/Microsoft.AspNetCore.Mvc.ViewFeatures/Internal/ViewResultExecutor.cs">ViewResultExecutor</a> et en particulier sa méthode `FindView()` ;) 

Je l'ai défini comme une extension de contrôleur et ajouté dans Gist:

<script src="https://gist.github.com/ygrenier/4e46b5de3e4a77ea62fe5f0d6488fa2a.js"></script>

## Conclusion

Voilà un code qui fonctionne exactement comme le moteur d'exécution Razor.

A bientôt,

Yanos

