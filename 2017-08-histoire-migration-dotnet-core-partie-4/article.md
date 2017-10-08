---
title: Histoire d’une migration en .Net Core - Partie 4
published: 2017-08-30
categories: .NET Core
tags: .NET Core, C#
url: /2017/08/histoire-dune-migration-en-dotnet-core-part-4
---

# Histoire d’une migration en .Net Core - Partie 4

Avec la disponibilité de VS2017, la prise en charge des projets .NET Core et .NET Standard a été grandement améliorée.

Courant Août sont sortis ".NET Core 2.0" et ".NET Standard 2.0" apportant pas mal d'améliorations. Vous trouverez les annonces officielles sur ces pages:
- [https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-core-2-0/](https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-core-2-0/)
- [https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-standard-2-0/](https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-standard-2-0/)


J'ai donc décidé de convertir le projet [SwissEphNet](https://github.com/ygrenier/SwissEphNet/)  en projet VS2017 pour avoir une meilleure prise en charge et supprimer les "bidouilles" mises en place. Je vous renvoi au [premier article de la série](http://blog.ygrenier.com/2016/11/histoire-dune-migration-en-dotnet-core-part-1/) pour plus d'informations.

<!--more-->

# Pourquoi convertir en projet VS2017 ?

Il y a plusieurs raisons à celà, les principales sont:
- Meilleures prise en charge des projets .NET Core et .NET Standard
- Les projets .NET 4.0 peuvent désormais cibler un projet multicibles (.NET Standard + .NET 4.0 par exemple)
- Les fichiers de projets deviennent des ".csproj", on supprime tout ce qui est "project.json" et ".xproj"
- Les projets ".csproj" n'ont plus besoin de référence tous les fichiers qu'ils veulent compiler (il suffit d'ajouter votre fichier dans votre dossier pour qu'il soit pris en charge par le projet - il apparait automatiquement dans VS)
- L'utilisation de .NET Core 2.0 améliore la compilation des projets

# Préambule

Avec l'arrivée de .NET Standard 2.0 les librairies PCL sont devenues non recommandées (bientôt dépréciées). On peut toujours en créer et les utiliser, mais la méthode officielle est désormais de créer des librairies .NET Standard.

VS 2017 et les dernières updates de VS 2015 (14.3 à ce jour) supportent complètement .NET Standard.

Pour ces raisons j'ai décidé de supprimer toutes les versions PCL et spécifiques du package Nuget, je ne vais garder que les versions:
- **.NET 4.0** car elle n'est pas supportée par .NET Standard
- **.NET Standard 1.0** pour tous les autres types de projet

Le choix de ".NET Standard 1.0" est du au fait que la librairie n'utilise rien de particulier. j'avais rencontré un problème avec les encodages qui m'avais forcé à utiliser la version ".NET Standard 1.3", mais ce problème n'a plus lieu.

Mon installation est VS2017 Community avec .NET Core 2.0 installé.

# Conversion des projets

Pour démarrer la conversion, j'ai tout simplement ouvert la solution avec VS2017.

Ce dernier détecte les anciens formats des projets et nous demande de faire une "Mise à niveau définitive". J'ai validé.

On laisse VS2017 travailler. Un fois terminé on constate que certains fichiers ont disparus. Mais si on essaye de regénérer la solution on se retrouve avec une centaine d'erreur notamment lors de la restauration des packages.

Regardons un peu ce que contient notre nouveau ".csproj", pour celà rien de plus simple depuis VS2017, on clic-droit sur notre projet SwissEphNet on trouve une option "Modifier SwissEphNet.csproj" qui nous ouvre notre fichier directement dans VS2017 sans avoir à le "Décharger" comme auparavant.

Et voilà ce que l'on a

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <Description>A Swiss Ephemeris .Net portage</Description>
    <Copyright>Copyright 2014-2017</Copyright>
    <AssemblyTitle>SwissEph for .Net</AssemblyTitle>
    <VersionPrefix>2.6.0.18</VersionPrefix>
    <Authors>Yan Grenier</Authors>
    <TargetFrameworks>net40;portable40-net40+sl5+win8+wp8+wpa81;netcore50;portable45-net45+win8+wpa81;portable45-net45+win8+wp8+wpa81;netstandard1.0</TargetFrameworks>
    <PlatformTarget>anycpu</PlatformTarget>
    <DebugType>portable</DebugType>
    <AssemblyName>SwissEphNet</AssemblyName>
    <PackageId>SwissEphNet</PackageId>
    <PackageTags>Swiss Ephemeris</PackageTags>
    <PackageReleaseNotes>2.6.0.18:
 - Update the code to the Swiss Ephemeris 2.06.00 version.
2.5.1.16:
 - Add .NETCore 5.0
 - Add PCL Profile 111
 - Add PCL Profile 259
 - Change PCL profile 136 to profile 328
2.5.1.14:
 - Update package to .net Core
2.5.1.13 :
 - Update the code to the Swiss Ephemeris 2.05.01 version.
2.4.0.12 :
 - Bug fixes.
2.4.0.11 :
 - Bug fix on the sideral calculation.
2.4.0.10 :
 - Update the code to the Swiss Ephemeris 2.04.00 version.
 - Correct the code remove all static fields to permit multiple calculations with multiple SwissEph instances.
2.2.1.9 :
 - Update the code to the Swiss Ephemeris 2.02.01 version.
</PackageReleaseNotes>
    <PackageProjectUrl>https://github.com/ygrenier/SwissEphNet</PackageProjectUrl>
    <PackageLicenseUrl>https://raw.githubusercontent.com/ygrenier/SwissEphNet/master/LICENSE</PackageLicenseUrl>
    <RepositoryType>git</RepositoryType>
    <RepositoryUrl>https://github.com/ygrenier/SwissEphNet</RepositoryUrl>
    <PackageTargetFallback Condition=" '$(TargetFramework)' == 'netstandard1.0' ">$(PackageTargetFallback);dnxcore50</PackageTargetFallback>
    <GenerateAssemblyTitleAttribute>false</GenerateAssemblyTitleAttribute>
    <GenerateAssemblyDescriptionAttribute>false</GenerateAssemblyDescriptionAttribute>
    <GenerateAssemblyConfigurationAttribute>false</GenerateAssemblyConfigurationAttribute>
    <GenerateAssemblyCompanyAttribute>false</GenerateAssemblyCompanyAttribute>
    <GenerateAssemblyProductAttribute>false</GenerateAssemblyProductAttribute>
    <GenerateAssemblyCopyrightAttribute>false</GenerateAssemblyCopyrightAttribute>
    <GenerateNeutralResourcesLanguageAttribute>false</GenerateNeutralResourcesLanguageAttribute>
    <GenerateAssemblyVersionAttribute>false</GenerateAssemblyVersionAttribute>
    <GenerateAssemblyFileVersionAttribute>false</GenerateAssemblyFileVersionAttribute>
  </PropertyGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'net40' ">
    <Reference Include="System" />
    <Reference Include="Microsoft.CSharp" />
  </ItemGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'portable40-net40+sl5+win8+wp8+wpa81' ">
    <Reference Include="mscorlib" />
    <Reference Include="System" />
    <Reference Include="System.Core" />
  </ItemGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'netcore50' ">
    <PackageReference Include="Microsoft.NETCore.UniversalWindowsPlatform" Version="5.0.0" />
  </ItemGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'portable45-net45+win8+wpa81' ">
    <Reference Include="mscorlib" />
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Runtime" />
    <Reference Include="System.Runtime.Extensions" />
    <Reference Include="System.Collections" />
    <Reference Include="System.IO" />
    <Reference Include="System.Linq" />
    <Reference Include="System.Globalization" />
    <Reference Include="System.Text.Encoding" />
    <Reference Include="System.Text.RegularExpressions" />
    <Reference Include="System.Reflection" />
    <Reference Include="System.Resources.ResourceManager" />
    <Reference Include="System.Diagnostics.Debug" />
  </ItemGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'portable45-net45+win8+wp8+wpa81' ">
    <Reference Include="mscorlib" />
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Runtime" />
    <Reference Include="System.Runtime.Extensions" />
    <Reference Include="System.Collections" />
    <Reference Include="System.IO" />
    <Reference Include="System.Linq" />
    <Reference Include="System.Globalization" />
    <Reference Include="System.Text.Encoding" />
    <Reference Include="System.Text.RegularExpressions" />
    <Reference Include="System.Reflection" />
    <Reference Include="System.Resources.ResourceManager" />
    <Reference Include="System.Diagnostics.Debug" />
  </ItemGroup>

  <PropertyGroup Condition=" '$(TargetFramework)' == 'portable40-net40+sl5+win8+wp8+wpa81' ">
    <DefineConstants>$(DefineConstants);PCL</DefineConstants>
  </PropertyGroup>

  <PropertyGroup Condition=" '$(TargetFramework)' == 'netcore50' ">
    <DefineConstants>$(DefineConstants);NOTYPECODE</DefineConstants>
  </PropertyGroup>

  <PropertyGroup Condition=" '$(TargetFramework)' == 'netstandard1.0' ">
    <DefineConstants>$(DefineConstants);NET_STANDARD</DefineConstants>
  </PropertyGroup>

  <ItemGroup Condition=" '$(TargetFramework)' == 'netstandard1.0' ">
    <PackageReference Include="Microsoft.NETCore.Platforms" Version="1.1.0" />
    <PackageReference Include="System.Collections" Version="4.0.11" />
    <PackageReference Include="System.ObjectModel" Version="4.0.12" />
    <PackageReference Include="System.Reflection" Version="4.1.0" />
    <PackageReference Include="System.Reflection.Extensions" Version="4.0.1" />
    <PackageReference Include="System.Reflection.Primitives" Version="4.0.1" />
    <PackageReference Include="System.Linq" Version="4.1.0" />
    <PackageReference Include="System.Runtime" Version="4.1.0" />
    <PackageReference Include="System.Runtime.Extensions" Version="4.1.0" />
    <PackageReference Include="System.Runtime.InteropServices" Version="4.1.0" />
    <PackageReference Include="System.Text.RegularExpressions" Version="4.1.0" />
    <PackageReference Include="System.Resources.ResourceManager" Version="4.0.1" />
    <PackageReference Include="System.Diagnostics.Debug" Version="4.0.11" />
  </ItemGroup>

</Project>

``` 

On constate que tout a été converti convenablement, et on nous avons notamment `<TargetFrameworks>net40;portable40-net40+sl5+win8+wp8+wpa81;netcore50;portable45-net45+win8+wpa81;portable45-net45+win8+wp8+wpa81;netstandard1.0</TargetFrameworks>` qui indique toutes les versions à compiler.

Comme je l'ai indiqué en préabule je ne garde que deux version, on va donc corriger la balise en `<TargetFrameworks>net40;netstandard1.0</TargetFrameworks>`. En regénérant la solution on a déjà beaucoup moins d'erreur, mais on en a toujours. Et comme il s'agit principalement de problème de référence de package on va supprimer toutes les `<ItemGroup Condition=" '$(TargetFramework)' == XXX "></ItemGroup>` c'est à dire dans notre cas tous les `<ItemGroup></ItemGroup>`. Nous les reconstruirons si nécessaire.

On obtient quelque chose comme:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <Description>A Swiss Ephemeris .Net portage</Description>
    <Copyright>Copyright 2014-2017</Copyright>
    <AssemblyTitle>SwissEph for .Net</AssemblyTitle>
    <VersionPrefix>2.6.0.18</VersionPrefix>
    <Authors>Yan Grenier</Authors>
    <TargetFrameworks>net40;netstandard1.0</TargetFrameworks>
    <PlatformTarget>anycpu</PlatformTarget>
    <DebugType>portable</DebugType>
    <AssemblyName>SwissEphNet</AssemblyName>
    <PackageId>SwissEphNet</PackageId>
    <PackageTags>Swiss Ephemeris</PackageTags>
    <PackageReleaseNotes>2.6.0.18:
...
</PackageReleaseNotes>
    <PackageProjectUrl>https://github.com/ygrenier/SwissEphNet</PackageProjectUrl>
    <PackageLicenseUrl>https://raw.githubusercontent.com/ygrenier/SwissEphNet/master/LICENSE</PackageLicenseUrl>
    <RepositoryType>git</RepositoryType>
    <RepositoryUrl>https://github.com/ygrenier/SwissEphNet</RepositoryUrl>
    <PackageTargetFallback Condition=" '$(TargetFramework)' == 'netstandard1.0' ">$(PackageTargetFallback);dnxcore50</PackageTargetFallback>
    <GenerateAssemblyTitleAttribute>false</GenerateAssemblyTitleAttribute>
    <GenerateAssemblyDescriptionAttribute>false</GenerateAssemblyDescriptionAttribute>
    <GenerateAssemblyConfigurationAttribute>false</GenerateAssemblyConfigurationAttribute>
    <GenerateAssemblyCompanyAttribute>false</GenerateAssemblyCompanyAttribute>
    <GenerateAssemblyProductAttribute>false</GenerateAssemblyProductAttribute>
    <GenerateAssemblyCopyrightAttribute>false</GenerateAssemblyCopyrightAttribute>
    <GenerateNeutralResourcesLanguageAttribute>false</GenerateNeutralResourcesLanguageAttribute>
    <GenerateAssemblyVersionAttribute>false</GenerateAssemblyVersionAttribute>
    <GenerateAssemblyFileVersionAttribute>false</GenerateAssemblyFileVersionAttribute>
  </PropertyGroup>

</Project>
```

On recompile et là miracle plus aucune erreur concernant `SwissEphNet`! Donc on va résoudre le reste avant de continuer.

Dans la [première partie](http://blog.ygrenier.com/2016/11/histoire-dune-migration-en-dotnet-core-part-1/) de la migration il a fallu mettre en place un "Duo de projets" pour prendre en charge la librairie en .NET Core et en PCL (pour être compatible .NET 4.0). Comme on en a plus besoin, on va supprimer le projet "SwissEphNet-NET40 (Portable)" de la solution mais également les fichiers suivant qui se trouvent dans le dossier "SwissEphNet":
- SwissEphNet-Net40.csproj
- SwissEphNet-Net40.nuget.targets

Comme on a supprimé ce projet, l'application `SweWin` à perdu son lien sur la librairie, on rajoute `SwissEphNet` dans ces références.

On regénére la solution: plus que quelques erreurs, les principales concernant les types `GuidAttribute` et `TypeCode` inconnus dans la version 'netstandard1.0'.

C'est normal ces types ne sont pas connus pas .NET Standard 1.0, pas de soucis on va les exclure via des contantes DEFINE comme on faisant auparavant avec les constantes PCL, NET_STANDARD et NOTYPECODE.

.NET Core génère automatiquement des constantes en fonction du framework cible, dans notre cas `NET40` et `NETSTANDARD1_0`. On va utiliser ces constantes pour l'exemple.

Dans le fichier `Properties/Assembly.cs` ligne 19 on remplace

```csharp
#if !PCL
```

par

```csharp
#if NET40
```

Dans le fichier `Extensions/TypeExtensions`, ligne 16 et 44 on remplace

```csharp
#if NET_STANDARD || NOTYPECODE
```

par

```csharp
#if NETSTANDARD1_0
```

et ligne 53


```csharp
#if NOTYPECODE
```

par

```csharp
#if NETSTANDARD1_0
```

On recompile et là miracle tout fonctionne. On peut essayer tous les programmes de test, ils fonctionnent. De même que les tests unitaires.

# Conclusion

Comme vous pouvez le constater la conversion a été assez rapide. Les projets sont devenus plus simple. 

Bien entendu au final je n'ai pas converti le projet de cette manière, vous pouvez regarder les sources sur [GitHub](https://github.com/ygrenier/SwissEphNet/) pour voir ce que j'ai fait exactement.

.NET Core devient vraiment efficace depuis sa version 2.0 :) N'hésitez pas à regarder de plus prêt ce nouveau framework (et notamment ASP.NET Core 2.0). 

A bientôt,

Yanos
