J'ai récemment ressorti de la naphtaline un vieux projet (librairie en .Net 4.0), avec une vieille version de [xUnit](https://xunit.github.io/).

Comme je devais améliorer la lib, en profite pour la passer en 4.5, et je met à jour tous les packages Nuget. 

Je lance les tests unitaires pour vérifier que rien n'a changé et là ... aucun tests n'apparaît dans l'Explorateur des tests de Visual Studio 2015 !!!!

Bon j'ai cherché pendant un bon moment, sans trouver de réponse satisfaisante, et j'ai trouvé la solution en recréant mon projet de test de 0, et là tout fonctionne. Alors j'ai fait quelques tests et j'ai trouvé:

La solution miracle: Modifier le projet de tests pour qu'il utilise le **Framework .Net 4.6** :)

A bientôt 

Yanos

