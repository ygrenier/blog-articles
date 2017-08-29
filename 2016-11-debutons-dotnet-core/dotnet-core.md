---
title: VS: Débutons avec .NET Core
published: 2016-11-13
categories: .NET Core, Développement
tags: .NET Core, C#
url: /2016/11/debutons-avec-dotnet-core
---

# Débutons avec .NET Core

Suite à mes articles précédents sur la [migration d'une librairie en .NET Core ](http://blog.ygrenier.com/2016/11/histoire-dune-migration-en-dotnet-core-part-1/), j'ai reçu des remarques de lecteurs concernant le coté un peu obscure de .NET Core dans mes explications.

J'en convient, j'ai fourni des liens sur des explications généralistes, mais finalement pas vraiment d'explications "techniques" sur ce nouveau framework.

Qu'à celà ne tienne, allons à la rencontre de notre nouvel ami.

<!--more-->

# Quelques informations utiles

- Site officiel .NET: [https://www.microsoft.com/net](https://www.microsoft.com/net) / [http://www.dot.net](http://www.dot.net) 
- Documentation officielle sur .NET: [https://docs.microsoft.com/fr-fr/dotnet/](https://docs.microsoft.com/fr-fr/dotnet/)
- Documentation officielle sur ASP.NET Core: [https://docs.microsoft.com/en-us/aspnet/core/](https://docs.microsoft.com/en-us/aspnet/core/) ce n'est pas le propos de cet article mais il y a également des informations sur .NET Core.

Actuellement la documentation étant en totale refonte, elle n'existe qu'en anglais.

Je rappelle les articles écrit par [Olivier Dahan](http://www.e-naxos.com) 
- [.NET Core ASP.NET Core et Xamarin](http://www.e-naxos.com/Blog/post/NET-Core-ASPNET-Core-et-Xamarin.aspx ".NET Core ASP.NET Core et Xamarin") 
- [.NET Standard arrive !](http://www.e-naxos.com/Blog/post/NET-Standard-arrive-!.aspx) 

Je résume malgré tout les caractéristiques de base de .NET Core:
- **Modulaire**: le framework est composé paquets qu'on utilise au besoin
- **Flexible**: il peut être inclus dans votre application ou installé côte-à-côte avec d'autre versions
- **Cross-Platform**/**Portable**: il s'exécute sur Windows, Mac OS et Linux. Il peut être portés sur d'autres OS
- **Utilitaires en ligne de commande**: tous les scénarios de production peuvent être appliqués avec des lignes de commande
- **Compatible**: .NET Core est compatible avec le framework .NET, Xamarin et Mono grâce aux [librairies .NET Standard](https://docs.microsoft.com/fr-fr/dotnet/articles/standard/library).
- **Open Source**: La plateforme .NET Core est open source, sous license MIT et Apache 2. La documentation est sous licence CC-BY. .net Core est un projet de la [Fondation .NET](http://www.dotnetfoundation.org/).


# Installons nous confortablement

Tout d'abord il nous installer .NET Core, et un éditeur de texte.

Vous trouverez tout pour l'installation à cette adresse: https://www.microsoft.com/net/core#windows

Si vous avez Visual Studio 2015, assurez-vous d'avoir l'[Update 3](https://www.visualstudio.com/fr/downloads/) et installer les [.NET Core 1.0.1 - VS 2015 Tooling Preview 2](https://go.microsoft.com/fwlink/?LinkID=827546). 

Si vous voulez utiliser les outils en ligne de commande avec un éditeur autre (VS Code, Sublime Text, notepad) vous pouvez vous contenter d'installer [.NET Core SDK for Windows.](https://go.microsoft.com/fwlink/?LinkID=827524). Le SDK est inclus dans ".NET Core 1.0.1 - VS 2015 Tooling".

Pour ce tutoriel on va utiliser la CLI (command-Line Interface) c'est-à-dire les lignes de commandes.

Pour vérifier que le .NET Core est installé il suffit d'ouvrir un "Invite de commande" ou une console "Powershell" et de saisir ```dotnet```, vous devez obtenir quelque chose comme:

```
> dotnet

Microsoft .NET Core Shared Framework Host

  Version  : 1.0.1
  Build    : cee57bf6c981237d80aa1631cfe83cb9ba329f12

Usage: dotnet [common-options] [[options] path-to-application]

Common Options:
  --help                           Display .NET Core Shared Framework Host help.
  --version                        Display .NET Core Shared Framework Host version.

Options:
  --fx-version <version>           Version of the installed Shared Framework to use to run the application.
  --additionalprobingpath <path>   Path containing probing policy and assemblies to probe for.

Path to Application:
  The path to a .NET Core managed application, dll or exe file to execute.

If you are debugging the Shared Framework Host, set 'COREHOST_TRACE' to '1' in your environment.

To get started on developing applications for .NET Core, install .NET SDK from:
  http://go.microsoft.com/fwlink/?LinkID=798306&clcid=0x409

``` 

Si ce n'est pas le cas essayez de désintaller et de réinstaller le framework.

# Principes de base

Tout ce dont nous avons besoin pour développer une application .NET Core utilise la commande ```dotnet```. Cette commande regroupe un ensemble d'outils qui sont une combinaison de commandes et de paramètres. C'est le même principe qu'un ```git```, ```npm``` ou ```composer``` pour ceux qui connaissent.

```dotnet``` supporte de base plusieurs commandes:
- new : pour créer un nouveau projet
- restore : pour restaurer tous les paquets nécessaires à la compilation du projet 
- run : pour exécuter une application
- build : pour compiler le projet
- test : pour exécuter les tests unitaires
- publish : pour générer un dossier contenant tout ce qui est nécessaire pour faire fonctionner l'application (ou la librairie)
- pack : pour générer un paquet Nuget

A savoir que ```dotnet``` est conçu pour pouvoir être étendu via des paquet Nuget, donc cet outil est totalement extensible.

Celà peut être perturbant, si vous êtes habitué à utiliser des outils comme Visual Studio, de manipuler une ligne de commande de ce genre. En réalité ce que fait VS c'est d'exécuter des lignes de commande à notre place. Toutefois actuellement on gère tout un ensemble de commande pour construire une application .NET ```msbuild```, ```csc```, etc. 

Ce qui change à partir de maintenant c'est que tout est regroupé sous une seule commande: ```dotnet```.

# Utilisation de la CLI

Etudions un peu plus précisement la CLI. Les commandes qui suivent créé une nouvelle application, restaure les paquets, compile l'application et l'exécute:
```
> dotnet new
> dotnet restore
> dotnet build --output /stuff
> dotnet /stuff/new.dll
```

Si l'on étudie de plus près ces commandes on peut déterminer 3 éléments :
- le pilote 
- la commande (ou verbe)
- les arguments de commande

## Le pilote

Le pilote c'est la partie ```dotnet``` de la commande. Le pilote à deux responsabilités:
- lancer les applications portables
- exécuter les verbes

Tout va dépendre de ce qui défini dans la ligne de commande. Dans le premier cas vous devez spécifier la DLL d'une application portable que ```dotnet``` doit exécuter, par exemple : ```dotnet /path/to/app.dll```.

Dans le second cas, le pilote essaye d'invoquer la commande spécifiée. Ce qui démarre un processus d'exécution:
- le pilote détermine la version des outils que vous voulez utiliser (.NET Core pouvant s'exécuter côte-à-côte avec d'autres versions). Cette version peut être spécifié dans un fichier 'global.json'. Par défaut il s'agit de la version la plus récente.
- le pilote exécute la commande dans l'environnement déterminé.

## Le "verb"

Le verbe est simplement une commande qui exécute une action. ```dotnet build``` compile votre code. ```dotnet publish``` publie votre code. Le verbe est définie dans une application console qui est nommée par convention ```dotnet-{verb}```. Toute la logique est implémentée dans l'application console qui représente le verbe.

## Les arguments

Les arguments que vous passez dans la ligne de commande sont les arguments pour le verbe/commande que vous invoquez. Par exemple ```dotnet publish --output publishedapp```, ```--ouput ...``` est l'argument passé à la commande ```publish```.

# Hello World!

Allez on ne déroge pas à la tradition des développeurs, faisons une petite application "Hello World" :)

## dotnet new

Première chose créez un dossier vide, et ouvrez un invite de commande se situant dans ce dossier.

Ensuite créons une nouvelle application de base:

```
> dotnet new
```

Vous obtenez deux nouveau fichiers:

### project.json

Ce fichier contient toutes les informations nécessaire permettant la génération de l'application (dépendances, options, ...).

```json
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {},
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.1"
        }
      },
      "imports": "dnxcore50"
    }
  }
}
```

### Program.cs

Ce fichier contient le code de base de l'application avec son point d'entrée.

```csharp
using System;

namespace ConsoleApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

## dotnet restore

La seconde chose à faire c'est de mettre à jour les dépendances de l'application.

```
> dotnet restore
```

La commande va analyzer le fichier ```project.json```, télécharger les dépendances (paquets Nuget) et écrire le fichier ```project.lock.json```. Ce fichier est nécessaire pour la compilation et l'exécution.

Le fichier ```project.lock.json``` contient l'ensemble des dépendances (même indirectes) et des informations qui décrivent l'application. Ce fichier est lu par d'autres commandes comme ```dotnet build``` et ```dotnet run```, leur permettant de traiter le code source avec les dépendances correctes.

## dotnet run

Maintenant que notre projet est prêt on peut l'exécuter:

```
> dotnet run
```

Comme nous n'avons pas compilée l'application avant de l'exécuter, ```dotnet run``` l'a détecté et à lancé ```dotnet build``` pour compiler l'application. Nous aurions pu le faire nous-même.

## Analyse du dossier

On constate que nous avons deux dossiers "bin" et "obj" qui sont apparus, comme n'importe quel projet .NET.

Dans le dossier "bin" on va trouver un dossier "Debug", ce qui reste classique, et surtout cela indique que par défaut nous compilons en "Debug".

En revanche dans le dossier "Debug" on ne trouve pas d'exécutable, mais un dossier "netcoreapp1.0".

En effet .NET Core permet de cibler plusieurs frameworks. C'est-à-dire que nous pouvons générer a partir du même projet, plusieurs frameworks cibles (par exemple une version .NET 4.0, .NET 4.6, .NET Core, PCL, ...). Dans le fichier "project.json" on trouve une liste de "frameworks", et pour chaque framework que trouvera .NET Core il va générer un dossier contenant le résultat de la compilation.

Dans le dossier "netcoreapp1.0" en revanche on trouve une chose a laquelle nous ne sommes pas habitués: il n'y a pas de ".exe" mais un ".dll" portant le nom du dossier dans lequel se trouve le projet. Dans mon cas j'ai fait mon application dans un dossier "netcore" j'ai donc un fichier "netcore.dll".

Comment ce fait-il qu'en ayant créée une application, nous ayons une DLL et pas un exécutable ?

.NET Core est portable, or les ".exe" sont des formats exécutables ne fonctionnant que sous Windows, donc ils ne sont pas portables. Par conséquent par défaut les applications sont compilées dans des DLL et sont exécutées via ```dotnet```. Ainsi une DLL compilée sous Windows peut être exécutée sous Linux (selon le même principe que Java pour ceux qui connaissent).

# Types de déploiements

Comme nous venons de la voir les applications compilées sont "portables". En .NET Core on parle de "Framework Dependent deployment" ou FDD. 

Mais .NET Core est capable également de générer des applications "auto-contenues" (ou autonome) (SDC pour "Self-Contained deployment").

Je ne vais pas m'étendre sur ce principe, je vous renvoi vers la [documentation](https://docs.microsoft.com/fr-fr/dotnet/articles/core/deploying/index).

Je vais juste faire un résumé.

## FDD : Framework Dependent deployment

C'est le comportement pas défaut, dans ce mode tout est compilé selon le framework ciblé, et en aucun cas d'un OS.

Pour exécuter ce type d'application il est nécessaire d'avoir .NET Core dans la version cibléé à la compilation d'installée sur le système et de faire appel à ```dotnet```.

## SCD : Self-Contained deployment"

Dans ce mode, l'application est compilée selon un "runtime" (cad un OS cible) et inclus le code de l'application mais également toutes les librairies et le .NET Core utilisés pour ce runtime. 

L'avantage principal c'est que l'application s'exécute sur l'OS sans dépendances préinstallées. La contre-partie est la taille du fichier de l'application qui peut être conséquent, car on inclus l'ensemble du code.

## Tester l'application en SCD

On va compiler notre application en autonome, en modifiant notre "project.json".

Première chose on va copier notre "project.json" pour pouvoir revenir en arrière si besoin.

```
> cp project.json project-portable.json
```

Premier point, supprimons la ligne ```"type" = "platform"``` dans la dépendance "Microsoft.NETCore.App". Ce qui va indiquer qu'on va compiler en SCD.

Mais celà ne suffit pas, il faut également indiquer le "runtime" dans lequel on va compiler notre application. Pour celà on ajoute une section "runtimes" dans "project.json".

Voilà notre nouveau fichier:
```json
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {},
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "version": "1.0.1"
        }
      },
      "imports": "dnxcore50"
    }
  },
  "runtimes": {
    "win7-x64":{},
    "osx.10.11-x64":{}
  }
}
```

Dans "runtimes" j'ai défini deux runtimes "win7-x64" et "osx.10.11-x64" pour indiquer que l'on peut compiler pour Windows 7 en 64 bits et en Mac OS X 64 Bits.

"win7-x64" sont "osx.10.11-x64" des identifiants appelés RID (Runtime IDentifier) la liste des RIDs disponibles ce trouvent sur [cette page](https://docs.microsoft.com/fr-fr/dotnet/articles/core/rid-catalog).

Comme nous avons modifier notre "project.json", il faut restaure les dépendances:

```
> dotnet restore
log  : Restoring packages for C:\Temp\netcore\project.json...
log  : Installing runtime.unix.System.Private.Uri 4.0.1.
log  : Installing runtime.osx.10.10-x64.runtime.native.System.IO.Compression 1.0.1.
log  : Installing runtime.osx.10.10-x64.Microsoft.NETCore.DotNetHost 1.0.1.
log  : Installing runtime.osx.10.10-x64.Microsoft.NETCore.Jit 1.0.4.
log  : Installing runtime.osx.10.10-x64.runtime.native.System.Net.Security 1.0.1.
log  : Installing runtime.unix.System.Net.Primitives 4.0.11.
log  : Installing runtime.unix.System.Net.Sockets 4.1.0.
log  : Installing runtime.unix.System.Console 4.0.0.
log  : Installing runtime.osx.10.10-x64.Microsoft.NETCore.DotNetHostResolver 1.0.1.
log  : Installing runtime.osx.10.10-x64.runtime.native.System.Net.Http 1.0.1.
log  : Installing runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography 1.0.1.
log  : Installing runtime.unix.Microsoft.Win32.Primitives 4.0.1.
log  : Installing runtime.unix.System.IO.FileSystem 4.0.1.
log  : Installing runtime.osx.10.10-x64.runtime.native.System 1.0.1.
log  : Installing runtime.unix.System.Runtime.Extensions 4.1.0.
log  : Installing runtime.unix.System.Diagnostics.Debug 4.0.11.
log  : Installing runtime.osx.10.10-x64.Microsoft.NETCore.DotNetHostPolicy 1.0.1.
log  : Installing runtime.osx.10.10-x64.Microsoft.NETCore.Runtime.CoreCLR 1.0.4.
log  : Writing lock file to disk. Path: C:\Temp\netcore\project.lock.json
log  : C:\Temp\netcore\project.json
log  : Restore completed in 55391ms.
```

On constate que tous les runtimes sont téléchargés et installés.

Compilons notre application application:
```
> dotnet build
```

on peut voir dans notre dossier "bin/Debug/netcoreapp1.0/" un nouveau dossier "win7-x64" dans lequel se trouvent "{project}.dll" qui est notre application et "{project}.exe" qui est l'exécutable qui démarre le framework et ensuite notre application.

On peut appeler cette directement notre application sans ```dotnet```:
```
> .\bin\Debug\netcoreapp1.0\win7-x64\netcore.exe
```

mais ```dotnet run``` fonctionne toujours.

Si on demande à publier notre application:

```
> dotnet publish
```

Un dossier "bin/Debug/netcoreapp1.0/win7-x64/publish" apparaît avec l'application et toutes les DLL nécessaires poour exécuter notre programme. Si on regarde le contenu on a un dossier d'environ 45 Mo. C'est un peu beaucoup pour un simple Hello World :)

Si vous publiez l'application en version portable, vous aurez un dossier avec uniquement la DLL de l'application (5 Ko).

# Allons plus loin

Modifions notre programme pour lui ajouter un calcul de la suite de Fibonacci.

```csharp
using static System.Console;

namespace ConsoleApplication
{
    public class Program
    {

        public static int FibonacciNumber(int n)
        {
            int a = 0;
            int b = 1;
            int tmp;

            for (int i = 0; i < n; i++)
            {
                tmp = a;
                a = b;
                b += tmp;
            }

            return a;   
        }

        public static void Main(string[] args)
        {
            WriteLine("Hello World!");
            WriteLine("Fibonacci Numbers 1-15:");

            for (int i = 0; i < 15; i++)
            {
                WriteLine($"{i+1}: {FibonacciNumber(i)}");
            }

        }
    }
}
```

On compile et on exécute notre application.

```
> dotnet build
> .\bin\Debug\netcoreapp1.0\win7-x64\netcore.exe
Hello World!
Fibonacci Numbers 1-15:
1: 0
2: 1
3: 1
4: 2
5: 3
6: 5
7: 8
8: 13
9: 21
10: 34
11: 55
12: 89
13: 144
14: 233
15: 377
```

Bon ca fonctionne, mais tout mettre dans un seul fichier ce n'est pas une très bonne idée, généralement un projet possède une multitude de fichiers. Nous allons donc déplacer notre calcul de Fibonacci dans un fichier a part.

Créez un nouveau fichier "Fibonacci.cs":

```csharp
using System;

namespace Fibonacci{
    public class FibonacciEngine {
        public static int FibonacciNumber(int n)
        {
            int a = 0;
            int b = 1;
            int tmp;

            for (int i = 0; i < n; i++)
            {
                tmp = a;
                a = b;
                b += tmp;
            }

            return a;   
        }
    }
}
```

modifions notre "Program.cs" pour utiliser notre nouvelle classe:

```csharp
using Fibonacci;
using static System.Console;

namespace ConsoleApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            WriteLine("Hello World!");
            WriteLine("Fibonacci Numbers 1-15:");

            for (int i = 0; i < 15; i++)
            {
                WriteLine($"{i+1}: {FibonacciEngine.FibonacciNumber(i)}");
            }

        }
    }
}
```

On compile et on exécute:

```
> dotnet build
> .\bin\Debug\netcoreapp1.0\win7-x64\netcore.exe
Hello World!
Fibonacci Numbers 1-15:
1: 0
2: 1
3: 1
4: 2
5: 3
6: 5
7: 8
8: 13
9: 21
10: 34
11: 55
12: 89
13: 144
14: 233
15: 377
```

tout ce passe bien, on constante donc qu'à partir du moment où on ajoute un fichier ".cs" dans le dossier, il est compilé et intégré à l'application. Pas besoin d'enregistrer les fichiers dans un fichier projet comme pour les ".csproj". Les fichiers ".cs" peuvent se trouver également dans une sous-dossier.

# Les dépendances

Nos projets font généralement référence à des librairies tiers, c'est ce qu'on appelle les "dépendances". Dans le monde .NET les dépendances sont gérer avec l'outil Nuget avec son dépôt principal : http://www.nuget.org

Dans les projets .NET Core, les dépendances sont définies dans le section "dependencies" du "project.json".

```json
"dependencies": { 
  "Newtonsoft.Json": "9.0.1",
  "SwissEphNet": "2.5.*"
}
```

Chaque élément est constitué du nom du paquet, avec comme valeur la version utilisée. Sans entrer dans les détails de la gestion des versions Nuget ([consultez cette page dans la documentation Nuget](https://docs.nuget.org/ndocs/consume-packages/dependency-resolution#dependency-resolution-in-nuget-3-x)), dans l'exemple fourni on requiert la plus petite version de "Newtonsoft.Json" a partir de la version "9.0.1", et la version la plus récente de "SwissEphNet" a partir de la version "2.5".

La section "dependencies" concerne l'ensemble du projet, si vous ciblez plusieurs frameworks, chaque dépendance sera référencée par chaque cible. Ce qui peut poser des problèmes, en effet certaines librairies ne supportent pas certains framework à partir d'une version, où certaines librairies sont utilisées pour compenser un manque (par exemple la gestion des code pages "Windows-1252" dans les applications .NET Core).

Pour gérer cette situation, on a la possibilité de définir une section "dependencies" pour chaque framework cible, et dans ce cas les dépendances ne s'appliquent qu'au framework concerné. Par exemple:

```json
"frameworks"{
 "net40":{
  "dependencies":{
   "Library": "4.0"
  }
 },
 "netstandard1.6":{
  "dependencies":{
   "Library": "7.0",
   "System.Text.Encoding.CodePages": "4.0.1"
  }
 }
}
```


# Gérer plusieurs projets

Nous nous amusons bien avec ```dotnet```, mais dans notre réalité quotidienne, un projet seul est rare, généralement nous avons une application ainsi que diverses librairies, et des tests. Nous avons donc un ensemble de projets qui se lient mutuellement. Sous VS nous utilisons une "Solution" qui contient l'ensemble des projets qui forment un tout.

En .NET Core nous avons également une notion de "Solution": un dossier qui englobe tous les dossiers de chaque projet.

La structure recommandée par .NET est la suivante:

```
/Solution
|_/src
|_/test
|_global.json
```

Dans le dossier ```src``` se trouve tous les dossiers de chaque projet.

Dans le dossier ```test``` se trouve tous les dossiers de chaque projet de tests.

Le fichier ```global.json``` (qui n'est pas obligatoire mais fortement recommandé) permet de configurer de "dossier global", surtout si vous désirez modifier l'infrastructure. Vous trouverez plus d'informations sur ce fichier [ici](https://docs.microsoft.com/fr-fr/dotnet/articles/core/tools/global-json).

De cette manière chaque projet va pouvoir référencer un autre projet de la même "Solution".

## Création des projets

Faisons un petit test. Dans un nouveau dossier vide:

```
> md src
> md test
> cd src
> md MyApp
> cd MyApp
> dotnet new
> cd ..
> md MyLibrary
> cd MyLibrary
> dotnet new -t lib
> cd ..\..
```

Nous avons créé une application, et une librairie (remarquez la commande ```dotnet new -t lib``` avec l'argument "-t lib" pour définir une librairie comme modèle de projet).

Dans la racine de notre solution on var créer un fichier "global.json":

```json
{
    "projects": ["src", "test"]
}
```

de cette manière nous indiquons à ```dotnet``` où trouver les projets. Depuis cette racine restaurons les dépendances.

```
> dotnet restore
log  : Restoring packages for C:\Temp\netcore\src\MyApp\project.json...
log  : Restoring packages for C:\Temp\netcore\src\MyLibrary\project.json...
log  : Writing lock file to disk. Path: C:\Temp\netcore\src\MyLibrary\project.lock.json
log  : C:\Temp\netcore\src\MyLibrary\project.json
log  : Restore completed in 1506ms.
log  : Writing lock file to disk. Path: C:\Temp\netcore\src\MyApp\project.lock.json
log  : C:\Temp\netcore\src\MyApp\project.json
log  : Restore completed in 2315ms.
```

On constate que nos deux projets ont été détectés et restaurés. 

## Modification de la librairie

Modifions notre librairie, dans le fichiers "src/MyLibrary/Library.cs"
```csharp
using System;

namespace MyLibrary
{
    public class Operations
    {
        public double Add(double a, double b)
        {    
            return a+b;
        }
        public double Mul(double a, double b)
        {    
            return a*b;
        }
    }
}
```

Je ne vous ferais pas l'affront de vous expliquer ce code :)

## Modification de l'application

Maintenant nous allons référencer notre librairie dans notre application, ouvrons "src/MyApp/project.json" et ajoutons une dépendance à la librairie:

```json
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {
    "MyLibrary": "1.0.0"
  },
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.1"
        }
      },
      "imports": "dnxcore50"
    }
  }
}
```
la version de la librairie est définie dans son "project.json".

Faisons un ```dotnet restore``` pour s'assurer que la référence est correcte.

A partir du moment où on défini les dépendances Nuget au même endroit que les références aux projets librairies, une petite explication sur le processus de résolution s'impose.

Lorsque ```dotnet``` résoud les dépendances, il suit le processus suivant:
- il recherche un projet correspondant dans les projets de la "Solution"
- il recherche un paquet dans un cache local
- il recherche un paquet dans les dépôts Nuget

donc les projets sont prioritaires.

Modifions "src/MyApp/Program.cs" pour utiliser notre librairie:
 
```csharp
using System;
using MyLibrary;

namespace ConsoleApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var op = new Operations();
            double a = 12.34;
            double b = 98.76;
            Console.WriteLine($"Add = {op.Add(a,b)}");
            Console.WriteLine($"Mul = {op.Mul(a,b)}");
        }
    }
}
```

Compilons et exécutons

```
> cd src\MyApp
> dotnet build
> dotnet run
Add = 111,1
Mul = 1218,6984
```

## Tests unitaires

Le dernier type de projet que l'on exploite souvent se sont les tests unitaires. .NET Core intègre la notion de test unitaires avec le support de différentes librairies. Les tests Microsoft existe toujours, mais personnellement je préfère xUnit. Ca tombe bien cette librairie est supportée :) Créons un projet pour les tests de notre librairie.

```
> cd ..\..
> cd test
> md MyLibrary.Tests
> cd MyLibrary.Tests
> dotnet new -t xunittest
```

Comme vous voyez il existe un type de projet "xunittest" qui prépare ce qu'il faut pour nous. Modifions le "project.json" pour référencer notre librairie.

```json
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable"
  },
  "dependencies": {
    "System.Runtime.Serialization.Primitives": "4.1.1",
    "xunit": "2.1.0",
    "dotnet-test-xunit": "1.0.0-rc2-build10025",
    "MyLibrary": "1.0.0"
  },
  "testRunner": "xunit",
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.1"
        }
      },
      "imports": [
        "dotnet5.4",
        "portable-net451+win8"
      ]
    }
  }
}
```

au passage j'ai modifié la version de "dotnet-test-xunit". On restaure

```
> dotnet restore
```

Ecrivons nos tests dans "src/test/MyLibrary.Tests/Tests.cs"

```csharp
using System;
using Xunit;
using MyLibrary;

namespace Tests
{
    public class Tests
    {
        [Fact]
        public void TestAdd() 
        {
            var op = new Operations(); 
            Assert.Equal(12.34, op.Add(10.10, 2.24));
        }
        [Fact]
        public void TestMul() 
        {
            var op = new Operations(); 
            Assert.Equal(22.624, op.Mul(10.10, 2.24), 3);
        }
    }
}
```

On compile et on exécute nos tests

```
> dotnet build
> dotnet test
xUnit.net .NET CLI test runner (64-bit .NET Core win10-x64)
  Discovering: MyLibrary.Tests
  Discovered:  MyLibrary.Tests
  Starting:    MyLibrary.Tests
  Finished:    MyLibrary.Tests
=== TEST EXECUTION SUMMARY ===
   MyLibrary.Tests  Total: 2, Errors: 0, Failed: 0, Skipped: 0, Time: 0,228s
SUMMARY: Total: 1 targets, Passed: 1, Failed: 0.
```

# Conclusion

Voilà nous avons fait un peu le tour de .NET Core, ce n'est qu'un aperçu car le sujet est vaste.

J'espère que ca vous aide à y voir plus clair. 

A bientôt,

Yanos



