# Histoire d'une migration en .Net Core - Partie 2

Dans la première partie nous avons effectué la migration de la librairie en elle-même.

Maintenant nous allons travailler sur les tests unitaires qui vont nous permettre de valider notre librairie (et découvrir quelques blagues).

<!--more-->

# Les tests en .Net Core

.Net Core inclus dans sa logique les tests unitaires. C'est à dire que l'on peut indiquer qu'un projet possède des tests unitaires et que l'on pourra les exécuter avec la commande ```dotnet test```.

Initialement j'ai utiliser les tests de Microsoft. Mais depuis j'ai privilégié la librairie [xUnit](http://xunit.github.io/) car j'aime bien son approche.

La chance veuille que .Net Core supporte les tests "xunit", donc la migration des tests va nécessité de créer un nouveau projet mais également de convertir tous les tests car ils ne fonctionnent pas de la même manière que les tests MS.

# Migration des tests

## Création du projet

Pour commencer nous allons faire comme avec la librairie SwissEphNet:
- renommage du projet de la librairie des tests ("SwissEphNet.Tests" => "SwissEphNet.Tests-Old")
- création d'un nouveau projet .Net Core dans un dossier temporaire
- suppression de ce nouveau projet de la solution
- copie des fichiers "project.json" et .xproj" dans le dossier de la librarie
- ajout du nouveau projet dans la solution
- suppression du projet temporaire
je vous renvoi vers la première partie pour la procédure.

En plus nous supprimerons l'ancien projet de la solution et du dossier (SwissEphNet.Tests-Old.csproj).

## Configuration du projet

Ouvrir le fichier "project.json" des tests et remplacer le contenu par 

```js
{
  "version": "1.0.0-*",
  "testRunner": "xunit",
  "dependencies": {
    "xunit": "2.2.0-beta4-build3444", 
    "dotnet-test-xunit": "2.2.0-preview2-build1029" 
  },
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.0"
        }
      }
    }
  }
}
```

comme indiqué sur [sur la page xUnit pour .Net Core](https://xunit.github.io/docs/getting-started-dotnet-core.html) sauf que j'ai référencée la dernière version de la librairie xUnit (la build 3444).

Dans les dépendances on va ajouter la librairie "SwissEphNet" à la version "2.5.*".

```js
  "dependencies": {
    "xunit": "2.2.0-beta4-build3444", 
    "dotnet-test-xunit": "2.2.0-preview2-build1029",
    "SwissEphNet": "2.5.*" 
  },
```

Le fait d'indiquer "2.5.*" indique la "dernière" version disponible de la librairie (selon les règles du [semver](http://semver.org/lang/fr/) que .Net Core suit). Comme notre solution contient la plus haute version de la librairie, c'est le projet local qui est pris en compte (et pas une version sur nuget.org).

## Conversion des tests MS en xUnit

Je ne vais pas faire un tutoriel sur la migration des tests, car ce n'est pas le propos de cette série d'article, je vous conseille de regarder le [projet sur Github](https://github.com/ygrenier/SwissEphNet/tree/master/Tests/SwissEphNet.Tests) pour voir le résultat final.

Mais pour résumer:
- remplacement de ```using Microsoft.VisualStudio.TestTools.UnitTesting;``` par ```using Xunit;```
- suppression des ```[TestClass]``` qui n'ont pas d'équivalent en xUnit
- remplacement de ```[TestMethod]``` par ```[Fact]```
- remplacement des méthodes assert, par exemple: ```Assert.AreEqual()``` par ```Assert.Equal()``` en xUnit les "Assert.Are\*()" et "Assert.Is\*()" n'existent pas.
- le code des méthodes marquées ```[TestInitialize]``` sont déplacées dans le constructeur du test
- le code des méthodes marquées ```[TestCleanup]``` sont déplacée dans une méthode "Dispose()" et on fait implémenter ```IDisposable``` la classe du test. 
- les tests ```Assert.Inconclusive();``` n'ont pas vraiment d'équivalent en xUnit, on le remplace par un paramètre "Skip" dans l'attribut "Fact": ```[Fact(Skip = "")]```
- les tests d'exception ne se font plus avec un méthode marquée par un attribut ```[ExpectedException]``` mais par une méthode d'assertion ```Assert.Throws<>()```
- les tests d'égalité de nombre réels avec un précision: xUnit utilise qu'un nombre de décimales pour la précision alors que MS Tests utilise un valeur delta ce qui pose quelques problèmes.

Une fois que tout le code à été converti, on peut enfin compiler. 

Ensuite on peut exécuter les tests depuis VS 

> Menu "Test>Exécuter>Tous les tests"

ou depuis une ligne de commande 

```shell
dotnet test 
```

Vous pouvez rencontrer des tests qui échouent car certaines précisions n'ont pa été correctement adaptés. Mais normalement seul le test "SwissEphNet.Tests.Issue18Test.LoadAsteroidData" doit échouer.

Ce test existe pour vérifier le bon fonctionnement du chargement des fichiers astéroïde, pour celà il nous faut lire un fichier "réel". Pour des raisons pratiques j'avais inclus ce fichier dans les ressources de la librairie de tests. Donc il faut faire de même pour ce projet. Pour celà on retourne dans le fichier "project.json", et on ajoute une section de configuration de la compilation:

```js
{
  "version": "1.0.0-*",
  "testRunner": "xunit",
  "buildOptions": {
    "embed": [ "files/*" ]
  },
  "dependencies": {
    "xunit": "2.2.0-beta4-build3444",
    "dotnet-test-xunit": "2.2.0-preview2-build1029",
    "SwissEphNet": "2.5.*"
  },
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.0"
        }
      }
    }
  }
}
```

cette configuration indique qu'il faut incorporer tous les fichiers se trouvant dans le dossier "files" du projet comme ressources.

# Conclusion

La vraie difficulté pour les tests est le changement de framework de tests.

Une autre différence c'est qu'on ajouté une référence à un autre projet via le project.json, mais sachez que vous pouvez faire la même chose via VS et l'option "Ajouter une référence" comme auparavant, VS modifiera le project.json a votre place.

A la prochaine,

Yanos
