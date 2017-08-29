---
title: Entity Framework : Erreur "The conversion of a datetime2 data type to a datetime data type resulted in an out-of-range value"
published: 2015-11-22
categories: C#/.NET, Mode boulet
tags: Entity Framework
url: /2015/11/entity-framework-erreur-the-conversion-of-a-datetime2-data-type-to-a-datetime-data-type-resulted-in-an-out-of-range-value
---

# Entity Framework : Erreur "The conversion of a datetime2 data type to a datetime data type resulted in an out-of-range value"

Bon on est dimanche, et certainement pour cela que mon cerveau est en mode boulet (d’ailleurs j’inaugure une nouvelle catégorie tellement je suis dépité).

Sur un projet perso lors d’un "Update-Database" de mon contexte EF, je me prends l’erreur "The conversion of a datetime2 data type to a datetime data type resulted in an out-of-range value".

Je perds 15 minutes a vérifier ma base de données avant d’en retrouver la raison. Et ce n’est pas comme si cette erreur ne m’était jamais arrivée&nbsp;!

<!--more-->

Contrairement à ce que pourrait nous faire croire dans le message d’erreur, pas la peine de chercher un champ de type ```datetime2```.

Non l’erreur est bien plus simple: lors de l’insertion d’une nouvelle entité qui possède une propriété ```DateTime``` (non nullable), il faut tout simplement lui affecter une valeur, car sinon elle sera définie avec la valeur par défaut ```DateTime.MinValue``` qui est une valeur non supportée par MSSQL.

15 minutes pour retrouver la raison de cette erreur alors que je me suis déjà fait piégé&nbsp;!  C’est un vrai talent de boulet&nbsp;!

Allez je vais manger pour me consoler,

A bientôt,

Yanos

