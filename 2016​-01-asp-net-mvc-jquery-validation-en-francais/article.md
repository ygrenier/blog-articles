---
title: ASP.NET MVC: jQuery Validation en français
published: 2016-01-28
categories: ASP.NET
tags: ASP.NET MVC, jQuery
url: /2016/01/asp-net-mvc-jquery-validation-en-francais
---

# ASP.NET MVC: jQuery Validation en français

ASP.NET MVC nous fourni des mécanismes de validation des informations saisies dans les formulaires. Nous avons deux types de validation:

- Validation à la saisie en JavaScript via jQuery.Validation
- Validation dans le contrôleur  lorsqu’il bind les valeurs dans les modèles. Utile lorsque la validation JS n’est pas active ou que le Javascript a été désactivé sur le navigateur du client.

Nous, petits français que nous sommes, utilisons un format de nombre et de date spécifique. Pour la partie MVC il nous suffit de force la culture du thread dans la langue qui nous intéresse pour que la conversion du texte en nombre ou date se passe sans problème.

Cela se corse lorsqu’il faut toucher à la validation jQuery.

# La méthode "officielle"

Toute la documentation que j’ai trouvée indique qu’il faut utiliser la librairie JS Globalize : [https://github.com/jquery/globalize](https://github.com/jquery/globalize), dont le package Nuget existe : [https://www.nuget.org/packages/jquery-globalize/](https://www.nuget.org/packages/jquery-globalize/).

Autant être honnête, je n’ai pas compris grand chose à cette librarie, notamment sur son fonctionnement (la fatigue peut-être). De plus la plupart des articles trouvés parlent de la librairie dans sa version antérieure à 1.x, hors il s’avère qu’il y a eu de fortes modifications dans le code, ce qui fait que les exemples trouvés ne fonctionnent plus avec la nouvelle version.

La chance à voulu que je tombe sur une librairie de John Reilly : [https://johnnyreilly.github.io/jQuery.Validation.Unobtrusive.Native/AdvancedDemo/Globalize.html](https://johnnyreilly.github.io/jQuery.Validation.Unobtrusive.Native/AdvancedDemo/Globalize.html) qui fait le lien entre Globalize et jQuery.Validation. Et là encore avec un package Nuget : [https://www.nuget.org/packages/jQuery.Validation.Globalize/](https://www.nuget.org/packages/jQuery.Validation.Globalize/).

Que demande le peuple me direz-vous !!!! Bin que ca fonctionne :)

En effet je suis tombé sur l’erreur suivante : [https://github.com/jquery/globalize/issues/507](https://github.com/jquery/globalize/issues/507) .

Donc la solution serait d’installer les différentes lib via NPM (si vous avez VS 2015, sinon en dehors de VS) car apparemment les packages Nuget ne contiennent pas tous les fichiers.

Pour être franc je n’ai pas eu le courage car il est difficile de mixer les systèmes de packages. Et que je n’ai pas le temps de batailler autant pour quelque chose de finalement plutôt simple.

# La méthode bourrin à la Yanos

Donc comme j’avais autre chose à faire, j’ai rapidement pondu un script que vous trouverez sur gist : [https://gist.github.com/ygrenier/568fda6b3ccc82321872](https://gist.github.com/ygrenier/568fda6b3ccc82321872)

Se code modifie les méthodes d’analyse des nombres et des dates. En revanche elle utilise les fonctions ```parse``` du javascript, vous devez donc avoir un navigateur en français. Je ferais évoluer le code plus tard. Si toutefois vous voulez améliorer les méthodes d’analyse, il vous suffit de modifier les fonctions ```parseFrenchNum``` et ```parseFrenchDate```.

Ce script doit être inclus APRES le script "jquery.validate.js". Dans ASP.NET vous pouvez modifier votre bundle "jqueryval" de la manière suivante pour que le script soit utilisé automatiquement avec jQuery.Validation :

```csharp
bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
  "~/Scripts/jquery.validate.*",
  "~/Scripts/jquery.validate-french.js"
));
```

Voilà,

A bientôt,

Yanos
