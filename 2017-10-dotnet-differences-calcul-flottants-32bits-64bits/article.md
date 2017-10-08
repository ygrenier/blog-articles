---
title: [.NET] Différences de calcul des flottants entre le 32bits et le 64bits
published: 2017-10-08
categories: C#/.NET
tags: .NET
url: /2017/10/dotnet-differences-calcul-flottants-32bits-64bits
---

# [.NET] Différences de calcul des flottants entre le 32bits et le 64bits

Dans le cadre du projet [SwissEph.Net](https://github.com/ygrenier/SwissEphNet) j'ai eu des retours d'utilisateurs indiquant une différence de résultat dans les calculs entre la librairie .NET et les DLLs C originales.

Comme je n'ai pas eu de retour de la part des utilisateurs concernant le code utilisé et les différences trouvées, j'ai pris un peu de temps pour créer un projet afin d'obtenir des tests de comparaison de valeurs entre les différentes versions: [https://github.com/ygrenier/SwissEphNet.TestsAndCompare](https://github.com/ygrenier/SwissEphNet.TestsAndCompare).

Une fois les tests effectués, il s'avère qu'il y a bien une différence de calcul sur les flottants en .NET, mais pas entre du code C et .NET, mais entre la version 32bits et 64bits du code .NET.

<!--more-->

# Tests effectués

L'application définie différents tests et enregistre le résultat dans plusieurs valeurs.

Je fais un chargement automatique de la DLL 32/64 bits en fonction de la plateforme (suivant [cet article](http://blog.ygrenier.com/2016/03/pinvoke-utiliser-dll-32bits-64bits-fonction-de-plateforme-mode-any-cpu/)).

Pour chaque plateforme les tests sont exécutés et exportés en fichier CSV.

Ensuite un fichier excel à été créé pour fusionner les résultats et comparer les valeurs.

# Analyse

Les tests ont donc été effectués sur ma machine en Windows 10 64 bits.

Vous trouverez tous les fichiers dans le dossier "tests" du dépot.

Le résultat des comparaisons entre les différentes plateformes sont les suivants

| Compare-DLL-32-64 | Compare-.NET-32-64 | Compare-DLL-.NET-32 | Compare-DLL-.NET-64 |
|-------------------|--------------------|---------------------|---------------------|
| true              | false              | false               | true                |

On constate qu'il n'y a pas de différences entre les DLL C en 32/64 bits, en revanche on peut voir qu'il y a une différence dans les résultats avec .NET en 32 bits (la version 64 bits ayant les mêmes résultats que la DLL C).

# Le calcul des flottants

Tout d'abord je rappelle que les flottants sont une approximation, il y a toujours des petites erreurs de calcul, d'où le fait qu'on les nomme des "Pseudo-réels" car il ne sont pas tout à fait des réels.

Leur format est codifié selon le [standard IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) et le .NET utilise pour le type `float` (`System.Single`) un flottant sur 32 bits, et pour le type `double` (`System.Double`) un flottant sur 64 bits (le `decimal` est codé d'une manière différente).

Les calculs sur les flottants est assez couteux en CPU, c'est pour celà qu'on a créé des coprocesseurs spécifiques nommés FPU (Floating Point Unit). Depuis longtemps les FPU sont intégrés dans les CPU.

Depuis pas mal d'années également les CPU intègrent de nouvelles instructions comme les [instructions SSE](https://fr.wikipedia.org/wiki/Streaming_SIMD_Extensions) qui optimisent le calcul, notamment concernant les flottants.

C'est là que nos problèmes commencent :)

Les FPU et les instructions SSE ne calculent pas les flottants de la même manière. Les FPU font des calculs sur 80 bits, tandis que les SSE font des calculs sur 32 bits (ou 64bits à partir des SSE2).

Ce qui veut dire que si on utilise les FPU ont aura une précision de calcul plus importante, mais comme dans notre cas on utilise des `double` (donc 64 bits) les calculs sont "tronqués" pour passer de 80 à 64 bits.

Ce qui n'est pas forcément le cas avec les instructions SSE puisqu'on est dans les mêmes tailles.

Ce qui fait qu'en fonction du type d'instructions utilisées (FPU ou SSE) vous n'obtiendrez pas toujours les mêmes valeurs. La différence est généralement assez faibles, mais dans un calcul complexe ces différences peuvent se multipler et donner des calculs avec des écarts considérables.

C'est l'une des raisons pour laquelle lorsqu'on fait des calculs scientifiques, ont utilise des librairies mathématiques spécifiques qui réduisent considérablement ce risque en utilisant des réels à virgules fixes, beaucoup moins sujet à ce type d'erreur de calcul (mais souvent plus gourmand en mémoire et en temps de calcul).

# Pourquoi le problème se rencontre pas dans la librairie C ?

Tout simplement parce que le C est compilé en natif, donc c'est à la compilation que l'on décide du calcul des flottants. Dans le cas de la Swiss Ephemeris je pense que la DLL a été compilée de la même manière en 32 et en 64 bits. D'où aucune différence de calcul.

# Et pourquoi il apparaît dans .NET ?

En revanche je pense que vous commencez à comprendre pourquoi il y a une différence de calcul dans la version .NET de la librairie: **la version 32 bits n'utilise pas les mêmes instructions de calcul que la version 64 bits**.

Pour être plus précis, il s'agit du Runtime qui va compiler à la volée le code intermédiaire CIL lors de l'exécution de l'application, et qui va décider d'utilise les instructions FPU si on est en 32bits et les instructions SSE si on est en 64 bits (et que les instructions sont disponibles).

Comme il s'agit du Runtime on ne peut pas influencer ce comportement (en tout cas pas à ma connaissance), et de plus une autre implémentation (comme Mono) peut très bien avoir un autre comportement. Apparement la norme .NET n'impose rien à ce sujet.

# Les solutions 

Pour résoudre ce problème, il suffit de ne pas utiliser les `float`/`double` et d'utiliser les `decimal` ou une librairie qui utilise sont propre moteur de calcul sans être influencé par le FPU ou les SSE.

Toutefois les `decimal` sont des réels codés sur 128 bits optimisés pour les calculs financiers et comptables (en réduisant les problèmes d'arrondis rencontrés sur les pseudo-réels). Ils ont une meilleure précision en revanche ils ont des limites plus faibles que les pseudo réels et ne sont pas conseillés pour calculer l'infiniment grand ou petit. De plus ces calculs sont généralement plus longs.

# Conclusion

Ne soyez pas surpris si vous trouvez des différences de calcul en .NET entre les plateformes. Et si ce comportement vous pose un problème il ne vous reste plus qu'à:
- choisir une plateforme 32 ou 64 bit et ne pas permettre d'autre choix (ce qui est un problème pour une librairie)
- utiliser des `decimal` si vous le pouvez
- utiliser une librairie tierce qui va vous éviter de rencontrer ce problème 

A bientôt,

Yanos
