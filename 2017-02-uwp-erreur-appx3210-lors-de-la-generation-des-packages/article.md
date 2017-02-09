Lorsque l'on veut publier une application Universelle, on doit générer les packages. 

Une fois générés, ces packages passent par une "pré-validation" qui, si vous n'avez pas touché à votre manifeste, va détecter que vous n'avez pas modifié les images par défaut.

Donc vous faites de jolies icônes, mais comme il y en a une liste impressionnante avec les différentes tailles, certains d'entre vous (devrais-je dire nous ;) ) se contentent du minimum syndical, c'est-à-dire qu'on ne fait pas toutes les images.

Et là lors de la génération des packages, vous vous prenez un erreur "APPX3210" vous indiquant que le manifeste fait référence à une image qu'il ne trouve pas, alors que cette fameuse image est bien présente dans votre projet.

<!--more-->

# Explications

Les images sont définies en différentes tailles pour prendre en compte les différentes densités d'écran disponibles. Ces tailles se définissent par un pourcentage généralement: 100, 125, 150, 200, 400. La taille 100% et obligatoire. 

Chaque image porte un nom, par exemple l'image du logo pour le store est par défaut "StoreLogo.png". Si vous ne définissez que l'image pour sa taille par défaut, alors vous n'aurez que ce fichier. En revanche si vous voulez définir les images avec les tailles 100% et 200%, vous devrez avoir deux fichiers "StoreLogo-100.png" et "StoreLogo-200.png" et "StoreLogo.png" devra être supprimé. Bon l'éditeur de manifeste de VS se charge de tout cela pour nous.

Alors quel est le problème ?

Hé bien c'est simple, il s'avère que en plus de la taille 100%, le manifeste à besoin des tailles 200% dans la plupart des cas. 

Donc si vous rencontrez cette erreur, c'est que vous avez certainement l'image 200% qui est manquante.

A bientôt,

Yanos

