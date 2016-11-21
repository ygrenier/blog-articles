<!--2015-11-entity-framework-erreur-impossible-de-determiner-un-tri-valide-pour-les-operations-dependantes-->

# Entity Framework: Erreur "Impossible de déterminer un tri valide pour les opérations dépendantes."

Bon ce n’est franchement pas ma journée. Après [cette première blague](http://blog.ygrenier.com/2015/11/entity-framework-erreur-the-conversion-of-a-datetime2-data-type-to-a-datetime-data-type-resulted-in-an-out-of-range-value/) en voici une seconde dans la foulée.

Toujours lors d’une mise à jour d’une base EF, cette fois je récupère l’erreur "Impossible de déterminer un tri valide pour les opérations dépendantes. Des dépendances peuvent exister en raison de contraintes de clé étrangère, d’exigences en matière de modèle ou de valeurs générées par le magasin.".

Là également ce n’est pas la première fois que je tombe dessus, mais apparemment mon cerveau a décidé de mettre en veille toutes mes capacités mémorielles :(

<!--more-->

Apriori il y a plusieurs raisons à ce genre d’erreur, mais la mienne est assez simple: il s’agit d’un problème de référence circulaire, une entité A à une référence vers une entité B. Le problème c’est que j’ai essayé de créer deux nouvelles entités en une seule fois, et on se prend cette erreur dés que l’on sauvegarde les modifications.

Pour résoudre ce problème rien de plus simple, oubliez de vouloir tout faire en une seule passe :

- Créer l’entité A
- Enregistrer les modifications
- Créer l’entité B
- Définir toutes les références entre A et B
- Enregistrer les modifications-  

A bientôt,

Yanos, le boulet

