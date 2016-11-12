# Débutons avec .Net Core

Suite à mes articles précédents sur la [migration d'une librairie en .Net Core ](http://blog.ygrenier.com/2016/11/histoire-dune-migration-en-dotnet-core-part-1/), j'ai reçu des remarques de lecteurs concernant le coté un peu obscure de .Net Core dans mes explications.

J'en convient, j'ai fourni des liens sur des explications généralistes, mais finalement pas vraiment d'explications "techniques" sur ce nouveau framework.

Qu'à celà ne tienne, allons à la rencontre de notre nouvel ami.

<!--more-->

# Quelques informations utiles

- Site officiel .Net: [https://www.microsoft.com/net](https://www.microsoft.com/net) / [http://www.dot.net](http://www.dot.net) 
- Documentation officielle sur .Net: [https://docs.microsoft.com/fr-fr/dotnet/](https://docs.microsoft.com/fr-fr/dotnet/)

Actuellement la documentation étant en totale refonte, elle n'existe qu'en anglais.

Je rappelle les articles écrit par [Olivier Dahan](http://www.e-naxos.com) 
- [.NET Core ASP.NET Core et Xamarin](http://www.e-naxos.com/Blog/post/NET-Core-ASPNET-Core-et-Xamarin.aspx ".NET Core ASP.NET Core et Xamarin") 
- [.NET Standard arrive !](http://www.e-naxos.com/Blog/post/NET-Standard-arrive-!.aspx) 

Je résume malgré tout les caractéristiques de base de .Net Core:
- **Modulaire**: le framework est composé paquets qu'on utilise au besoin
- **Flexible**: il peut être inclus dans votre application ou installé côte-à-côte avec d'autre versions
- **Cross-Platform**/**Portable**: il s'exécute sur Windows, Mac OS et Linux. Il peut être portés sur d'autres OS
- **Utilitaires en ligne de commande**: tous les scénarios de production peuvent être appliqués avec des lignes de commande
- **Compatible**: .Net Core est compatible avec le framework .Net, Xamarin et Mono grâce aux [librairies .Net Standard](https://docs.microsoft.com/fr-fr/dotnet/articles/standard/library).
- **Open Source**: La plateforme .Net Core est open source, sous license MIT et Apache 2. La documentation est sous licence CC-BY. .net Core est un projet de la [Fondation .NET](http://www.dotnetfoundation.org/).


# Installons nous confortablement

Tout d'abord il nous installer .Net Core, et un éditeur de texte.

Vous trouverez tout pour l'installation à cette adresse: https://www.microsoft.com/net/core#windows

Si vous avez Visual Studio 2015, assurez-vous d'avoir l'[Update 3](https://www.visualstudio.com/fr/downloads/) et installer les [.NET Core 1.0.1 - VS 2015 Tooling Preview 2](https://go.microsoft.com/fwlink/?LinkID=827546). 

Si vous voulez utiliser les outils en ligne de commande avec un éditeur autre (VS Code, Sublime Text, notepad) vous pouvez vous contenter d'installer [.NET Core SDK for Windows.](https://go.microsoft.com/fwlink/?LinkID=827524). Le SDK est inclus dans ".NET Core 1.0.1 - VS 2015 Tooling".

Pour ce tutoriel on va utiliser la CLI (command-Line Interface) c'est-à-dire les lignes de commandes.

Pour vérifier que le .Net Core est installé il suffit d'ouvrir un "Invite de commande" ou une console "Powershell" et de saisir ```dotnet```, vous devez obtenir quelque chose comme:

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

Tout ce dont nous avons besoin pour développer une application .Net Core utilise la commande ```dotnet```. Cette commande regroupe un ensemble d'outils qui sont une combinaison de commandes et de paramètres. C'est le même principe qu'un ```git```, ```npm``` ou ```composer``` pour ceux qui connaissent.

```dotnet``` supporte de base plusieurs commandes:
- new : pour créer un nouveau projet
- restore : pour restaurer tous les paquets nécessaires à la compilation du projet 
- run : pour exécuter une application
- build : pour compiler le projet
- test : pour exécuter les tests unitaires
- publish : pour générer un dossier contenant tout ce qui est nécessaire pour faire fonctionner l'application (ou la librairie)
- pack : pour générer un paquet Nuget

A savoir que ```dotnet``` est conçu pour pouvoir être étendu via des paquet Nuget, donc cet outil est totalement extensible.

Celà peut être perturbant, si vous êtes habitué à utiliser des outils comme Visual Studio, de manipuler une ligne de commande de ce genre. En réalité ce que fait VS c'est d'exécuter des lignes de commande à notre place. Toutefois actuellement on gère tout un ensemble de commande pour construire une application .Net ```msbuild```, ```csc```, etc. 

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
- le pilote détermine la version des outils que vous voulez utiliser (.Net Core pouvant s'exécuter côte-à-côte avec d'autres versions). Cette version peut être spécifié dans un fichier 'global.json'. Par défaut il s'agit de la version la plus récente.
- le pilote exécute la commande dans l'environnement déterminé.

## Le "verb"

Le verbe est simplement une commande qui exécute une action. ```dotnet build``` compile votre code. ```dotnet publish``` publie votre code. Le verbe est définie dans une application console qui est nommée par convention ```dotnet-{verb}```. Toute la logique est implémentée dans l'application console qui représente le verbe.

## Les arguments

Les arguments que vous passez dans la ligne de commande sont les arguments pour le verbe/commande que vous invoquez. Par exemple ```dotnet publish --output publishedapp```, ```--ouput ...``` est l'argument passé à la commande ```publish```.

# Hello World!

Allez on ne déroge pas à la tradition des développeur, faisons une petite application "Hello World" :)

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

La commande va analyzer le fichier ```project.json```, télécharger les dépendances (paquets Nuget) et écrire le fichier ```project.lock.json```. Ce fichier est nécessaire pouor la compilation et l'exécution.

Le fichier ```project.lock.json``` contient l'ensemble des dépendances (même indirectes) et des informations qui décrivent l'application. Ce fichier est lu par d'autres commandes comme ```dotnet build``` et ```dotnet run```, leur permettant de traiter le code source avec les dépendances correctes.

## dotnet run

Maintenant que notre projet est prêt on peut l'exécuter:

```
> dotnet run
```

Comme nous n'avons pas compilée l'application avant de l'exécuter, ```dotnet run``` l'a détecté et à lancer ```dotnet build``` pour compiler l'application. Nous aurions pu le faire nous-même.

## Analyse du dossier

On constate que nous avons deux dossiers "bin" et "obj" qui sont apparus, comme tout projet .Net qui se respecte.

Dans le dossier "bin" on va trouver un dossier "Debug", ce qui reste classique, et surtout ca indique que par défaut nous compilons en "Debug".

En revanche dans le dossier "Debug" on ne trouve pas d'exécutable, mais un dossier "netcoreapp1.0".

En effet .Net Core permet de cibler plusieurs framework. C'est-à-dire que nous pouvons générer a partir du même projet, plusieurs framework cible (par exemple une version .Net 4.0, .Net 4.6, .Net Core, PCL, ...). Dans le fichier "project.json" on trouve une liste de "frameworks", et pour chaque framework que trouvera .Net Core il va générer un dossier contenant le résultat de la compilation.

Dans le dossier "netcoreapp1.0" en revanche on trouve une chose a laquelle nous ne sommes pas habitués: il n'y a pas de ".exe" mais un ".dll" portant le nom du dossier dans lequel se trouve le projet. Dans mon cas j'ai fait mon application dans un dossier "netcore" j'ai donc un fichier "netcore.dll".

Comment ce fait-il qu'en ayant créée une application, nous ayons une DLL et pas un exécutable ?

.Net Core est portable, or les ".exe" sont des formats exécutable ne fonctionnant que sous Windows, donc ils ne sont pas portables. Par conséquent par défaut les applications sont compilées dans des DLL et sont exécutées via ```dotnet```. Ainsi une DLL compilée sous Windows peut être exécutée sous Linux (selon le même principe que Java pour ceux qui connaissent).

# Types de déploiements

Comme nous venons de la voir les applications compilées sont "portables". En .Net Core on parle de "Framework Dependent deployment" ou FDD. 

Mais .Net Core est capable également de générer des applications "auto-contenu" (SDC pour "Self-Contained deployment").

Je ne vais pas m'étendre sur ce principe, je vous renvoi vers la [documentation](https://docs.microsoft.com/fr-fr/dotnet/articles/core/deploying/index).

Je vais juste faire un résumé.

## FDD : Framework Dependent deployment

C'est le comportement pas défaut, dans ce mode tout est compilé selon le framework ciblé, et en aucun cas d'un OS.

Pour exécuter ce type d'application il est nécessaire d'avoir .Net Core dans la version cibléé à la compilation d'installée sur le système et de faire appel à ```dotnet```.

## SCD : Self-Contained deployment"

Dans ce mode, l'application est compilée selon un "runtime" (cad un OS cible) et inclus le code de l'application mais également toutes les librairies et le .Net Core utilisés.

L'avantage principal c'est que l'application s'exécute sur l'OS sans dépendances préinstallées. La contre-partie est la taille du fichier de l'application qui peut être conséquent car on inclus l'ensemble du code.

## Tester l'application en SCD

On va compiler notre application en "Natif", en modifiant notre "project.json".

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

"win7-x64" sont "osx.10.11-x64" des identifients appelés RID (Runtime IDentifier) la liste des RIDs disponibles ce trouvent sur [cette page](https://docs.microsoft.com/fr-fr/dotnet/articles/core/rid-catalog).

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

On constate que tous les runtimes sont installés.

Compilons notre application application:
```
> dotnet build
```

on peut voir dans notre dossier "bin/Debug/netcoreapp1.0/" un nouveau dossier "win7-x64" dans lequel se trouvent "{project}.dll" qui est notre application et "{project}.exe" qui est l'exécutable qui démarre le framework et ensuite notre application.

On peut appeler cette directement notre application sans ```dotnet```:
```
> .\bin\Debug\netcoreapp1.0\win7-x64\netcore.exe
```
Sachez ```dotnet run``` fonctionne toujours.

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

tout ce passe bien, on constante donc qu'à partir du moment où on ajoute un fichier ".cs" dans le dossier, il est compilé et intégré à l'application. Pas besoin d'enregistrer les fichiers dans un fichier projet comme pour les ".csproj".

Améliorons notre calcul de Fibonacci pour mettre en cache les valeurs:

