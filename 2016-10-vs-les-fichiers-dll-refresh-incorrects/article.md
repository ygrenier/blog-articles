<!--2016-10-vs-les-fichiers-dll-refresh-incorrects-->
# VS: Les fichiers "*.dll.refresh" incorrects

Dans Visual Studio lorsqu’on utilise des projets particuliers comme les "Site Web" qui n’ont pas de fichier de projet (comme un .csproj), pour gérer les DLL venant des packages Nuget, un fichier "*.dll.refresh" et créé pour chaque DLL.

Malheureusement on peut rencontrer quelques problèmes avec ces fichiers.

<!--more-->

# Qu’est-ce qu’un fichier "*.dll.refresh" ?

Les fichiers "*.refresh" contiennent le nom du fichier ".dll" qui sert de référence, et quand VS à besoin d’actualiser la DLL, il recopie la DLL de référence.

Ainsi lorsqu’on demande à actualiser les packages Nuget (restauration ou upgrade d’une version) les DLL sont mises à jour dans le dossier "packages", puis VS va recopier le fichier qui trouve dans le ".dll.refresh".

# Quel est le problème ?

Vous pouvez rencontrer un problème où le fichier ".dll.refresh" pointe sur un fichier qui n’existe pas et VS remonte une erreur/avertissement.

Tant que vous travaillez sur le même poste tout va bien, mais si vous travaillez sur plusieurs postes (c’est mon cas), ou que vous travaillez en équipe avec un partage de fichier via un CVS quelconque (ce qui est toujours mon cas), là vous pouvez avoir des soucis, car généralement le fichier ".dll.refresh" contient le fichier complet **absolu**. Donc si votre projet ne se trouve pas au même endroit que celui où à été créé le fichier ".dll.refresh" VS ne trouvera pas la DLL au bon endroit.


# Comment on le résout ?

Le plus simplement du monde :) En modifiant le fichier ".dll.refresh" pour le définir en relatif.

Ces fichiers sont de simple fichier texte, il vous suffit d’ouvrir le fichier et de corriger le contenu, en rendant en relatif par rapport à votre projet.

Par exemple j’ai un site web "WebSite" dans le dossier de solution "d:\travail\sites\MonWebSite\".

Ce projet utilise le package Nuget "Newtonsoft.Json".

Dans le dossier "d:\travail\sites\MonWebSite\WebSite\bin" se trouve la dll "Newtonsoft.Json.dll" avec un fichier "Newtonsoft.dll.refresh".

Ce fichier va donc contenir:

```
d:\travail\sites\MonWebSite\packages\Newtonsoft.Json.8.0.3\lib\net45\Newtonsoft.Json.dll
```

pour le rendre relatif il suffit de le corriger en

```
..\packages\Newtonsoft.Json.8.0.3\lib\net45\Newtonsoft.Json.dll
```

**ATTENTION** rappelez vous que c’est relatif au **projet**, et pas au dossier où se trouve la DLL (en l’occurrence le dossier "bin"). Bien sur faites attention à ne pas avoir de "/" ou "\" en début du nom.

Voilà maintenant quelque soit le poste de travail, le fichier refresh pointera toujours sur la bonne DLL.

A bientôt,

Yanos

