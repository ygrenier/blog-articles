# Histoire d'une migration en .Net Core - Partie 3

Bien nous avons migré la librairie et les tests unitaires, il nous reste les deux applications console, SweMini et SweTest.

<!--more-->

# L'application SweMini

On va appliquer la même recette que pour les tests unitaires:

- renommage du projet
- création d'un nouveau projet .Net Core dans un dossier temporaire
- suppression du nouveau projet de la solution
- copie des fichiers "project.json" et .xproj" dans le dossier
- ajout du nouveau projet existant dans la solution
- suppression du projet temporaire
- suppression de l'ancien projet de la solution
- suppression de l'ancien fichier '.csproj'

La seule différence marquante c'est que l'on va créé une application console plutôt qu'une librairie.

La compilation va échouer car "AppDomain" n'existe plus en .Net Core, pour obtenir le chemin de base de l'application on a deux options:
- on utilise ```System.Path.Directory.GetCurrentDirectory()``` pour obtenir le dossier courant, mais comme on peut appeler l'application console depuis un autre dossier, le dossier courant changera
- on récupère le dossier du nom du fichier de l'assembly de l'application: ```Path.GetDirectoryName(typeof(Program).GetAssembly().Location)```

Une fois corrigé, ca compile.

Exécutons notre application... et là c'est le drame !!!! 

Une erreur à lieu

> 'Windows-1252' is not a supported encoding name. For information on defining a custom encoding, see the documentation for the Encoding.RegisterProvider method.
Parameter name: name

En fait de base le code page 'Windows-1252' n'est pas pris en charge par .Net Core. Il n'y a rien de dramatique, comme le .Net Core est modulaire, on a un paquet qui prend en charge les code pages supplémentaire. 

Pourquoi cette erreur n'a pas eu lieu lors de nos tests, j'avoue ne pas le savoir pour le moment (je n'ai pas beaucoup cherché, il faut bien l'avouer).

# Prise en charge de 'Windows-1252'

On utilise ce code page car c'est le format des fichiers d'éphémérides. Il est défini dans la propriété statique ```SwissEph.DefaultEncoding```.

Donc comme je l'ai dit il y a des paquets qui prennent en charge les codes pages supplémentaires. Seulement ces paquets ne sont supportés qu'a partir du .Net Standard 1.3, c'est la raison pour laquelle j'ai choisi cette version pour la librairie.

Il faut ajouter une référence au paquet "System.Text.Encoding.CodePages" (version 4.0.1 actuellement) aux dépendances de la version .Net Standard de la librarie car il n'y a qu'elle qui est concernée. Le project.json devient :

```js
  ...
  "frameworks": {
    "netstandard1.3": {
      "dependencies": {
        "NETStandard.Library": "1.6.0",
        "System.Text.Encoding.CodePages": "4.0.1"
      },
      "imports": "dnxcore50",
      "buildOptions": {
        "define": [ "NET_STANDARD" ]
      }
    },
  }
  ...
``` 

Seulement ce n'est pas suffisant, il faut initialiser le moteur de code page. Nous allons donc devoir changer la création de la propriété statique ```SwissEph.DefaultEncoding```.

Nous allons créer un constructeur statique qui va exécuter le code d'initialisation de cette propriété. Comme seule la version .Net Standard est concernée nous allons a nouveau réutiliser la variable conditionnelle "NET_STANDARD".

Il n'y a rien de compliqué :

```csharp
        /// <summary>
        /// Static constructor
        /// </summary>
        static SwissEph()
        {
#if NET_STANDARD
            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
#endif
            DefaultEncoding = Encoding.GetEncoding("Windows-1252");
        }

		...

        /// <summary>
        /// Default encoding
        /// </summary>
        public static Encoding DefaultEncoding = null;

```

On compile et on exécute à nouveau l'application.... et là c'est GAGNE !!! :)


# L'application SweTest

Cette application est un peu plus complexe que SweMini car c'est l'application officielle d'interrogation des Swiss Ephemeris.

Là encore rien de neuf, on reproduit les mêmes étapes que SweMini jusqu'à ajouter la référence à la SwissEphNet.

A la compilation nous n'avons qu'une seule erreur à la ligne 3394 : 

> 'Environment' does not contain a definition for 'CurrentDirectory'


Il suffit de modifier le ligne

```csharp
                sp[0] = Environment.CurrentDirectory;
```

par

```csharp
                sp[0] = System.IO.Directory.GetCurrentDirectory();
```

Et ça compile et ca s'exécute !!!

# Pour finir

Les quelques points qu'il a fallut corriger:

- nettoyer les fichiers .nuspec qui ne servent plus
- modifier le fichier "appveyor.yml" qui est le script de déploiement continu de la librairie afin de n'utiliser que les commandes "dotnet".

# Conclusion

Globalement la migration à été plus simple que je ne m'y attendais. Une fois que l'on contourne les quelques désagréments dû aux outils encore en preview, et que l'on comprend la philosophie de base de .Net Core, les choses se passent relativement bien.

Le framework est suffisamment mature pour être utilisé, et le fait que l'on puisse cibler plusieurs frameworks avec le même projet, permet de compenser un manque potentiel de fonctionnalités, vous avez un recours. 

Par conséquent rien ne vous retient pour convertir vos librairies et les faire entrer dans une nouvelle ère très prometteuse.

A bientôt,

Yanos

