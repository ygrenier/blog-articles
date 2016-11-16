<!--/2016/04/asp-net-mvc-rendersection-optionnel-->
# ASP.NET MVC: faire un RenderSection() optionnel

Dans le cadre d’un développement j’ai eu besoin de rendre optionnel un appel à un ```RenderSection()``` dans mon layout (ma section devant être générée selon certaines conditions).

```
@if(condition)
{
    @RenderSection("section", required:false)
}
```

Le problème c’est que si on a une section définie dans une vue, mais qu’elle n’est pas rendue dans le layout (par exemple notre condition est fausse), une exception est levée: "Les sections suivantes ont déjà été définies mais n’ont pas été rendues pour la page de disposition « ~/Views/Shared/_Layout.cshtml » : « section »".

En effet comme Razor nous impose de générer toutes les sections définies dans les vues, si notre condition est fausse l’appel à ```RenderSection()``` n’a pas lieu.

Pour résoudre ce problème il nous faut donc appeler « RenderSection() » sans émettre le code générer par la section.

[Deepak Aggarwal](https://plus.google.com/103908432244378021535) à déjà répondu à cette question dans ce post (en anglais) [http://discusscode.blogspot.fr/2012/10/conditionally-render-mvc-layout-section.html](http://discusscode.blogspot.fr/2012/10/conditionally-render-mvc-layout-section.html) toutefois je suis tombé sur un problème supplémentaire.

En premier lieu, que fait Deepak ?

```
if(condition)
{
    @RenderSection("section", required: false)
}
else
{
    @RenderSection("section", required: false).ToString().Remove(0)
}
```

Il provoque le rendu de la section qui retourne un objet "HelperResult", il le transforme en chaîne de caractère dont il appelle la méthode ```Remove(0)``` qui va retourner une chaîne vide. Jusque là tout va bien.

Sauf que si vous n’avez pas de section "section" dans votre vue, ```RenderSection()``` retourne ```System.Null``` ce qui provoque une exception de référence null. Je rajoute donc la création d’un "HelperResult" qui ne retourne rien dans ce cas là.

```
if(condition)
{
    @RenderSection("section", required: false)
}
else
{
    @(((RenderSection("ribbon", required: false) ?? new HelperResult(w=> { })).ToString()+" ").Remove(0))
}
```

Je rajoute un espace au résultat de ```ToString()``` pour être sur de ne pas avoir une chaîne vide qui provoquerait une erreur avec ```Remove(0)```.

Attention ce code n’est pas un exemple de performance, il faut donc l’utiliser avec parcimonie.

Si vous devez avoir beaucoup de section optionnelle privilégier la technique inverse qui consiste à inclure vos balises ```@section``` dans une condition dans chaque vue (ne pas oublier de placer votre section dans des balises ```<text></text>```). Cette technique est plus lourde, mais plus optimisée.

A bientôt

Yanos

