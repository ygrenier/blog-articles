En ASP.NET Core on a la possibilité de modulariser son application Web en créant des librairies permettant de partager le code (les contrôleurs, modèles, etc.).

Nous avons également la possibilité d'incorporer des ressources (fichiers script, images, vues Razor) auxquelles on peut ensuite accéder via le fournisseur de fichier [EmbeddedFileProvider](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.fileproviders.embeddedfileprovider). En enregistrant ce fournissant dans le moteur Razor, on peut ainsi accéder aux vues qui sont embarquées dans la librarie.

Toutefois une difficulté ce pose, VS2017 n'est pas capable de prendre entièrement en charge l'édition des vues Razor, un ensemble d'erreurs s'affichent et certains fonctionnalités IntelliSense ne sont pas disponibles.

<!--more-->

# Pourquoi ?

VS2017 reconnaît par défaut les fichiers Razor (.cshtml, .vbhtml), toutefois ASP.NET MVC étend les fonctionnalités de Razor, et ce sont ces fonctionnalités que VS2017 ne reconnaît pas car il ne "sait pas" que nous cherchons a éditer un fichier Razor pour ASP.NET MVC.

# Résolution

Il faut indiquer à VS2017 qu'il édite un projet Web pour qu'il intègre les fonctionnalités étendues de Razor. Pour cela c'est assez simple:

En premier, ouvrir le fichier ".csproj" de votre librairie (Clic-droit sur le projet de la librairie "Modifier MonProjet.csproj").

Il faut modifier la première balise du projet

```xml
<Project Sdk="Microsoft.NET.Sdk">
```

en

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

dans la balise `PropertyGroup` ajouter une balise `PreserveCompilationContext``

```xml
  <PropertyGroup>
    ...
    <PreserveCompilationContext>true</PreserveCompilationContext>
  </PropertyGroup>
```

Enregistrer a attendre que VS2017 recharge votre solution. Votre librairie change d'icône similaire à une application ASP.NET Core.

Ouvrir les propriétés de de votre librairie pour modifier le "Type de sortie" en "Bibliothèque de classes".

# Références
- [https://stackoverflow.com/questions/43336498/when-embedding-cshtml-files-there-are-errors-parsing-the-files-types-cannot-b](https://stackoverflow.com/questions/43336498/when-embedding-cshtml-files-there-are-errors-parsing-the-files-types-cannot-b)

# Conclusion

A voilà maintenant vous pouvez éditer vos vues Razor. 

Attention toutes les fonctionnalités n'apparaissent pas, comme les TagHelpers ASP.NET MVC qui n'apparaissent pas dans l'IntelliSense. C'est normal car notre projet n'a pas de fichier "_ViewImports.cshtml" contenant `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` qui importe les TagHelpers ASP.NET MVC.

A bientôt,

Yanos
