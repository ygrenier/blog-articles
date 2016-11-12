# Histoire d'une migration en .Net Core

A moins que vous ne viviez dans une grotte, vous devez savoir que Microsoft est en train de développer un nouveau framework, totalement réécrit, modulaire, open source, bref que du bon en perspective. Le nom de ce nouveau venu : .Net Core.

Dans sa lancée Microsoft propose une nouvelle approche aux librairies portables. En effet les PCL (Portable Class Libraries) sont diablement efficaces, mais la multiplication des framework désormais disponibles apportent quelques complications dans leur relations et dépendances. Cette nouvelle approche se nomme: .Net Standard.

Peut-être que j'aurais le temps d'en parler plus longuement un jour, en attendant [Olivier Dahan](http://www.e-naxos.com) à déjà écrit des articles intéressants à lire [.NET Core ASP.NET Core et Xamarin](http://www.e-naxos.com/Blog/post/NET-Core-ASPNET-Core-et-Xamarin.aspx ".NET Core ASP.NET Core et Xamarin") et [.NET Standard arrive !](http://www.e-naxos.com/Blog/post/NET-Standard-arrive-!.aspx). 

Au cas où certains ne le saurait pas (ce dont je doute ;)) j'ai effectué le portage
en .Net de la célèbre librairie [Astrodient Swiss Ephemeris](http://www.astro.com/swisseph/) qui est écrite en C.

Dés le départ j'ai effectué ce portage comme une librairie PCL (Profile 136) afin d'être utilisable dans la plupart des applications .Net que nous pouvons développer. 

Avec .Net Core qui pointe le bout de son nez, j'ai commencé l'étudier, et quoi de plus efficace que de mettre en pratique ce que l'on étudie ;) Donc en route pour migrer [SwissEphNet](https://github.com/ygrenier/SwissEphNet). 

# Avant de commencer

Au moment où j'écris ces lignes j'ai déjà écris et publié la version .Net Core de la librairie (depuis une quinzaine de jour), mais je n'ai pas noté les différentes étapes.

Donc pour cet article je vais refaire le processus de migration, mais connaissant déjà les écueils je vais éviter de retomber dans les mêmes pièges ;) Dans la réalité j'ai tatonné un peu plus, mais ca n'a pas été non plus été un chemin croix :)

## Le projet

Le projet SwissEphNet est composé de :

- une librarie portable: SwissEphNet/SwissEphNet.csproj
- des tests unitaires: Tests/SwissEphNet.Tests/SwissEphNet.Tests.csproj
- une application console de calcul simple: Programs/SweMini/SweMini.csproj
- une application console d'interrogation des éphémérides: Programs/SweTest/SweTest.csproj
- une application WinForms d'interrogation des éphémérides: Programs/SweWin/SweWin.csproj

Les trois applications sont des conversions des programmes officiels de la Swiss Ephemeris.

## Le .Net Core

Il faut savoir que .Net Core permet de compiler les projets en différents frameworks, et à la capacité de de générer directement des paquets Nuget. Dans mon cas j'ai choisi de compiler la librarie en ".Net 4.0" (plus pour le fun qu'autre chose), ".Net Standard" (j'y reviendrais en ce qui concerne la version) et en "PCL Profile 136" (afin de maintenir la compatibilité avec l'existant).

Comme le .Net Core n'est destiné qu'aux applications console et aux applications ASP.Net Core, je ne migrerais que la librairie et les applications consoles. L'application WinForms restera telle qu'elle sauf qu'elle référencera la version PCL ou .Net 4.0.

Les projets .Net Core sont gérés grâce à un fichier Json "project.json". Tous les fichiers ".cs" qui se trouve dans le même dossier (et ces sous-dossier) font partis du projet et sont intégré dans la compilation.

Dans le cas de VS2015, un projet ".xproj" est utilisé en plus du "project.json", il est l'équivalent du ".csproj" sauf qu'il ne contient que des paramètres de projet, il ne contient aucune référence aux fichiers à compiler, même principe tous les fichiers ".cs" dans le dossier seront compilés, VS2015 analyse le dossier du projet pour nous montrer les fichiers. En clair a partir du moment ou vous ajoutez un fichier dans le dossier VS2015 le détecte automatiquement et l'affiche et il sera compilé, il n'est pas nécessaire de l'ajouter au projet comme avec unn ".csproj".


## Les difficultés

Autant le runtime .Net Core (ce qui permet d'exécuter une application .Net Core) est release 1.0.1, autant le SDK (ce qui nous permet de développer les applications) est en preview. Et qui dit preview dit encore quelques couacs.

Je n'ai pas rencontré de problème avec le SDK, en revanche son intégration dans Visual Studio 2015 (avec le "VS 2015 Tooling" qui est forcément preview également) apporte quelques désagréments.

Je passe sur les tracasseries de confort (mauvais rafraichissement des icônes de l'état Git par exemple) pour me focaliser sur un point plus problématique: les références d'une librairie .Net Core depuis un projet .Net Full (framework original).

Dans le cas du programme SweWin qui est en WinForms, il faudra cibler la version 4.0 ou PCL de la librarie. Mais dans les fait VS2015 sera capable d'intégrer le projet .Net Core comme référence, mais n'arrivera pas à trouver la librairie et donc échouera à la compilation.

Comme .Net Core compile en plusieurs framework, il génère les DLL dans un dossier spécifique à chaque framework, et dans le cas de SweWin VS2015 n'arrive pas à trouver les DLL dans le bon dossier. Il y a une méthode de contournement un peu bourrine mais qui fonctionne.

La méthode est très simple: actuellement VS2015 gère les projets .Net Core via un fichier ".xproj" (je dis actuellement car a terme ce fichier devrait disparaître), il nous suffit donc de rajouter un fichier ".xproj" et "projct.json" dans le dossier contenant la librairie. On renomme l'un des projets car VS2015 n'accepte pas deux projets de même nom dans une solution (l'extension n'est pas prise en compte) et on intègre le second projet à la solution. Ainsi en fonction du type de projet qui va référencer la librarie on va sélectionner le bon projet. Ce qu'il y a de bien c'est que pour les projet ".Net Core" tous les fichiers qui se trouvent dans le dossier sont compilés dans la librairie, donc s'il y a quelque chose à ajouter ou à modifier, on le fait dans le projet .Net Full (".csproj"), se sera répercuté automatiquement dans le projet ".Net Core" (".xproj").

# La migration

## SwissEphNet

Attaquons par la pièce maitresse.

### 1. Mise en place du "duo" de projets

Donc on va commencer par mettre en place dans le même dossier les deux types de projets. 

#### Renommer le projet existant

Le première chose est de renommer le projet "SwissEphNet" en "SwissEphNet-Portable" ce qui va permettre de modifier également le nom du fichier ".csproj".

> Clic-Droit sur le projet **SwissEphNet**, option "Renommer", modification en **SwissEphNet-Portable**

On peut vérifier que le nom du fichier à bien été modifier.

#### Créer un projet .Net Core

On va demander à VS2015 de créer un projet .Net Core afin d'obtenir les deux fichiers qui nous intéressent qu'on recopiera dans le dossier de la librairie.

Première étape on créé un nouveau dossier "temp" dans le dossier de la solution.

En second on créé un nouveau projet "SwissEphNet" de type "Librairie .Net Core" dans le fichier temporaire.

> Clic-Droit sur la solution "Ajouter > Nouveau projet ..."


![Nouveau projet](http://blog.ygrenier.com/wp-content/uploads/2016/11/new-netcore-library.png)

Une fois créé, on supprime le projet de la solution.

> Clic-Droit sur le projet "Supprimer"

On enregistre la solution : Ctrl+Maj+S.

On se rend dans le dossier du projet temporaire, on copie les fichiers "SwissEphNet.xproj" et "project.json" pour les coller dans le dossier "SwissEphNet" original où se trouver le fichier "SwissEphNet-Portable.csproj".

Puis on supprime le dossier "temp".

Maintenant on ajoute un projet existant pour intégrer le projet .Net Core de SwissEphNet.

> Clic-Droit sur la solution "Ajouter ...>Projet existant..."
> Sélectionner le projet "SwissEphNet/SwissEphNet.xproj"

Et là miracle le projet s'ouvre avec tous les fichiers de la librairie.

![Les deux projets ensemble](http://blog.ygrenier.com/wp-content/uploads/2016/11/two-projects.png)

Ne cherchez pas à recompiler la solution, il y a quelques erreurs qu'il va falloir résoudre.

### 2. Modification de 'project.json'

Ouvrez le fichier 'project.json', vous avez quelque chose comme:

```js
{
  "version": "1.0.0-*",

  "dependencies": {
    "NETStandard.Library": "1.6.0"
  },

  "frameworks": {
    "netstandard1.6": {
      "imports": "dnxcore50"
    }
  }
}

```

En gros ce fichier indique que notre projet est en version "1.0.0", qu'il référence la librairie "NETSrandard.Library" à sa version "1.6.0", et qu'il est compilé en "Net Standard 1.6".

Comme je l'ai indiqué en début d'article le ".Net Standard" est la nouvelle approche des librairies portables, vous trouverez plus d'informations sur la documentation officielle si cela vous intéresse: [.NET Standard Library](https://docs.microsoft.com/en-us/dotnet/articles/standard/library).

La version 1.6.0 est la dernière release (la 2.0.0 est encore en preview à l'heure où j'écris ces lignes). Comme ce qui m'intéresse c'est de pouvoir cibler un maximum de plateforme, mon idée était de faire une librairie en .Net Standard 1.0. Malheureusement il s'avère que pour gérer le code page "windows-1252" nécessaire pour les fichiers Swiss Ephemeris, je ne peux pas descendre en dessous de la version 1.3 du .Net Standard. Seulement cette version ne me permet pas de cibler les Windows Phone 8.1.

Heureusement le .Net Core est bien conçu, il nous permet de compiler plusieurs cibles depuis le même projet. Et c'est ce que j'ai fait, j'ai ajouté la version Portable et la version .Net 4.0.

#### Configuration de la version .Net Standard 1.3

Dans la sections "frameworks", on modifie "netstandard1.6" en "netstandard1.3".

La section "dependencies" contient les références "externes", en l'occurence la librairie .Net Standard. On ne touche pas à sa version (la librairie n'existe pas en 1.3.0), c'est une sorte de grosse librairie de référence donc ce n'est pas un problème. 

En revanche là où se situe la section "dependencies" indique des références pour toutes les versions compilées, donc quand on va ajouter la compilation en PCL, cette derniere va référencer la librairie, ce qui va nous poser un problème car cette référence ne concerne que les version .Net Core.

Pour résoudre ce problème on va déplacer la section "dependencies" dans la configuration "netstandard1.3".

On obtient quelque chose comme:

```js
{
  "version": "1.0.0-*",

  "frameworks": {
    "netstandard1.3": {
      "dependencies": {
        "NETStandard.Library": "1.6.0"
      },
      "imports": "dnxcore50"
    }
  }

}
```

#### Configuration de la version .Net 4.0

Pour ajouter une cible de compilation, il suffit de rajouter une section dans "frameworks", au même niveau que "netstandard1.3". Pour le .Net 4.0 cette section doit s'appeler "net40". Nous n'avons pas de paramètres particulier dans cette section, on va donc la laisser vide pour le moment.

```js
{
  "version": "1.0.0-*",

  "frameworks": {
    "netstandard1.3": {
      "dependencies": {
        "NETStandard.Library": "1.6.0"
      },
      "imports": "dnxcore50"
    },
    "net40": {}
  }
}
```

#### Configuration de la version PCL

Pour la version PCL on va faire la même chose que précédemment, sauf que le nom de la section est beaucoup plus compliquée: ".NETPortable,Version=v4.0,Profile=Profile136".

Il s'avère que la gestion des PCL est un peu plus compliquée, par défaut ce type de projet ne référence AUCUNE dll même "System". Il va donc falloir indiquer les "assemblies" dont nous allons avoir besoin dans une section "frameworkAssemblies".

Les DLL système dont à besoin SwissEphNet sont:
- mscorlib
- System
- System.Core

Comme je le sais ? En fait je me suis contenté de compiler la librairie et de lire les erreurs. En regardant de plus près dans les erreurs VS2015 indique qu'il nous manque peut-être une DLL. J'ai juste ajouté les noms des DLL qu'il m'indiquait. Compliqué n'est-ce pas :) !

Voila à quoi ressemble notre fichier:

```js
{
  "version": "1.0.0-*",

  "frameworks": {
    "netstandard1.3": {
      "dependencies": {
        "NETStandard.Library": "1.6.0"
      },
      "imports": "dnxcore50"
    },
    "net40": {},
    ".NETPortable,Version=v4.0,Profile=Profile136": {
      "frameworkAssemblies": {
        "mscorlib": "",
        "System": "",
        "System.Core": ""
      }
    }
  }
}
``` 

### 3. Correction des erreurs de compilations

En c'est instant nous avons toujours des erreurs mais qui ne concerne que deux choses:
- Le type 'Type' ne contient pas de propriété 'Assembly'
- Le type 'Type' ne contient pas de méthode 'GetTypeCode'

Ce n'est pas à cause d'une DLL manquante, mais dû au fait que la gestion de la réflection à changé dans les nouveaux framework .Net dont le .Net Core fait partie. 

En .Net 4x et PCL on obtient l'assembly ainsi: ```monType.Assembly```
En WinRT/UWP/.Net Core la syntaxe est désormais: ```monType.GetTypeInfo().Assembly```  

Nous devons donc modifier le code en fonction de la plateforme, pour celà la méthode la plus simple est de définir un DEFINE pour chaque plateforme, et mettre en place une compilation conditionnelle avec un ```#IF xxx```.

Plutôt que de répéter cette condition, nous allons mettre au point une méthode d'extension qui retourne l'assembly d'un type et remplacer l'accès à l'assembly par cette méthode.

#### Mise en place d'un DEFINE par plateforme

Pour définir un DEFINE on ajoute une section "buildOptions" dans les frameworks qui nous intéressent. Dans notre cas on va définir "NET_STANDARD", car ce dont nous avons besoin c'est de savoir si on est en .Net Core ou pas. Notre 'project.json' devient donc:

```js
{
  "version": "1.0.0-*",

  "frameworks": {
    "netstandard1.3": {
      "dependencies": {
        "NETStandard.Library": "1.6.0"
      },
      "imports": "dnxcore50",
      "buildOptions": {
        "define": [ "NET_STANDARD" ]
      }
    },
    "net40": {},
    ".NETPortable,Version=v4.0,Profile=Profile136": {
      "frameworkAssemblies": {
        "mscorlib": "",
        "System": "",
        "System.Core": ""
      }
    }
  }
}
``` 

#### Mise en place d'une méthode d'extension

Dans le dossier "Extensions" du projet "SwissEpheNet-Portable" (le fichier sera automatiquement pris en compte par le projet .Net Core) on ajoute un fichier "TypeExtensions.cs" :

```csharp
using System;
using System.Reflection;

namespace SwissEphNet
{
    /// <summary>
    /// Extensions for <see cref="System.Type"/>
    /// </summary>
    public static class TypeExtensions
    {
        /// <summary>
        /// Returns the <see cref="System.Reflection.Assembly"/> of a type
        /// </summary>
        public static Assembly GetAssembly(this Type type)
        {
#if NET_STANDARD
            return type?.GetTypeInfo()?.Assembly;
#else
            return type?.Assembly;
#endif
        }

    }
}

```

#### Remplacement de Assembly par GetAssembly()

On remplace les accès ```.Assembly``` par la méthode ```.GetAssembly()``` à deux endroits:
- Dans le fichier "Sweph.cs" ligne 317
- Dans le fichier "SwissEph.swephexp.h.cs" ligne 633

#### Correction de GetTypeCode()

```GetTypeCode()``` est une méthode qui a disparue dans .NetCore, en revanche l'énuméré qu'elle retourne existe toujours. Nous allons donc créer une méthode d'extension qui reproduit le comportement de cette méthode, et nous allons faire les remplacements comme précédemment.

Dans "TypeExtensions" précédemment créé on ajoute la méthode suivante :

```csharp
        /// <summary>
        /// Returns the <see cref="System.TypeCode"/> of a type
        /// </summary>
        public static TypeCode GetTypeCode(this Type type)
        {
#if NET_STANDARD
            if (type == null) return TypeCode.Empty;
            else if (type == typeof(bool)) return TypeCode.Boolean;
            else if (type == typeof(byte)) return TypeCode.Byte;
            else if (type == typeof(char)) return TypeCode.Char;
            else if (type == typeof(ushort)) return TypeCode.UInt16;
            else if (type == typeof(uint)) return TypeCode.UInt32;
            else if (type == typeof(ulong)) return TypeCode.UInt64;
            else if (type == typeof(sbyte)) return TypeCode.SByte;
            else if (type == typeof(short)) return TypeCode.Int16;
            else if (type == typeof(int)) return TypeCode.Int32;
            else if (type == typeof(long)) return TypeCode.Int64;
            else if (type == typeof(string)) return TypeCode.String;
            else if (type == typeof(float)) return TypeCode.Single;
            else if (type == typeof(double)) return TypeCode.Double;
            else if (type == typeof(DateTime)) return TypeCode.DateTime;
            else if (type == (typeof(Decimal))) return TypeCode.Decimal;
            else return TypeCode.Object;
#else
            return Type.GetTypeCode(type);
#endif
        }
```

Le remplacement est a peine plus compliqué, on ne les trouvent que dans le fichier "C.printf.cs": ```Type.GetTypeCode(Value.GetType())``` par ```Value.GetType().GetTypeCode()```.

### Vérification du résultat

Maintenant notre librairie se compile sans erreur.

Si l'on se rend dans le dossier "bin/debug" de la librairie on trouve:
- un dossier "net40": contenant la librairie compilée en .Net 4.0 par le projet .Net Core
- un dossier "netstandard1.3": contenant la librairie compile en .Net Standard 1.3
- un dossier "portable40-net40+sl5+win8+wp8": contenant la librairie en version PCL Profile 136
- la librairie compilée par le projet "SwissEphNet-portable" qui est normalement identique à celle se trouvant dans "net40".


### Dernière touche

Comme le .Net Core a la capacité de générer des paquets Nuget, dans "project.json" on peut définir toutes les informations que l'on mettait auparavant dans les fichiers ".nuspec".

Donc ajoute toutes les informations nécessaires en début de fichier (en écrasant "version").

```js
{
  "name": "SwissEphNet",
  "version": "2.5.1.14",
  "title": "SwissEph for .Net",
  "authors": [ "Yan Grenier" ],
  "description": "A Swiss Ephemeris .Net portage",
  "copyright": "Copyright 2014-2016",
  "packOptions": {
    "summary": "A Swiss Ephemeris .Net portage",
    "owners": [ "Yan Grenier" ],
    "licenseUrl": "https://raw.githubusercontent.com/ygrenier/SwissEphNet/master/LICENSE",
    "projectUrl": "https://github.com/ygrenier/SwissEphNet",
    "requireLicenseAcceptance": false,
    "tags": [ "Swiss Ephemeris" ],
    "repository": {
      "type": "git",
      "url": "https://github.com/ygrenier/SwissEphNet"
    },
    "releaseNotes": "2.5.1.14:\n - Update package to .net Core\n2.5.1.13 :\n - Update the code to the Swiss Ephemeris 2.05.01 version.\n2.4.0.12 :\n - Bug fixes.\n2.4.0.11 :\n - Bug fix on the sideral calculation.\n2.4.0.10 :\n - Update the code to the Swiss Ephemeris 2.04.00 version.\n - Correct the code remove all static fields to permit multiple calculations with multiple SwissEph instances.\n2.2.1.9 :\n - Update the code to the Swiss Ephemeris 2.02.01 version.\n2.1.0.6 :\n - Update the code to the Swiss Ephemeris 2.01.00 version.\n2.0.0.5 :\n - Fix an error : some swed data (like topocentric position) was reseted on each swe_close() call.\n"
  },
  
// Suite du fichier

}
```

En ouvrant une ligne de commande et en se plaçant dans le dossier du projet .Net Core, saisir la commande :
```
dotnet pack
```

On va générer deux fichiers dans le dossier "bin/debug":
- SwissEphNet.2.5.1.14.nupkg
- SwissEphNet.2.5.1.14.symbols.nupkg

qui sont les paquets Nuget (l'un contenant les librairies, l'autres contenant les sources et les symboles pour le déboggage), on peut ouvrir "SwissEphNet.2.5.1.14.nupkg" pour constater qu'il contient les trois versions de la librairie.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/nuget-details.png).

# Conclusion

Voilà la première étape est faite, nous avons une librairie qui se compile correctement.

Vous devez normalement avoir un avertissement lorsque vous compilez le projet "SwissEphNet-portable" : 

> Les packages contenant des cibles et des fichiers de propriétés MSBuild ne peuvent pas être installés complètement dans des projets ciblant plusieurs infrastructures. Les cibles et les fichiers de propriétés MSBuild ont été ignorés.

Pas d'inquiétude, il s'agit simplement du fait que VS2015 détecte un 'project.json' et comme nous avons définis plusieurs cibles de compilations, VS nous indique qu'il ne sait pas gérer des cibles multiples avec un projet ".csproj".

Les prochaines étapes seront de convertir les tests ainsi que les applications console. Nous verrons que notre librairie compile, mais qu'en réalité il y a quelques problèmes à résoudre à l'exécution.

A la prochaine.

Yanos

