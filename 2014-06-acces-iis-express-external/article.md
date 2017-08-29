---
title: Accès à IIS Express depuis l'extérieur
published: 2014-06-20
categories: ASP.NET
tags: ASP.NET, IIS Express, Visual Studio
url: /2014/06/acces-iis-express-external
---

# Accès à IIS Express depuis l'extérieur

Lors d’un développement ASP.NET sous Visual Studio on utilise IIS Express qui permet de faire du débogage de nos sites Web.

Le problème réside dans le fait que IIS Express ne permet qu’un accès via le nom "localhost". Résultat impossible de tester notre site depuis un smartphone ou une tablette par exemple.

En fouillant le net on trouve des réponses à ce problème, toutefois je ne sais pas s’il y a eu des modifications entre temps, mais toutes les solutions que j’ai essayées ne fonctionnent pas complètement. Résultat je suis parvenu à faire un mix, et sur une machine française s’il vous plaît, car ca aussi c’est une petite blague (mais très petite ;) )

Donc première chose : vous munir du port réservé par Visual Studio dans votre projet. Pour cela lancez votre projet et regardez l’URL, où de vous rendre dans les propriétés de votre projet onglet "Web" où vous trouverez l’URL du projet avec son port.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/vs-project-web-properties.png)

Une fois le port déterminé, fermez Visual Studio et vérifiez que IIS Express ne s’exécute plus.

Dans le pare-feu de Windows (si ce dernier est actif) ajouter une règle entrante autorisant le port du site web.

Dans votre dossier "Mes documents" (ou "Documents" dans votre dossier utilisateur sous Win8) vous trouvez un dossier "IISExpress\config". Dans se dossier se trouve un fichier "applicationhost.config".

**MISE A JOUR**: depuis VS 2015 les projets utilisent leur propre fichier de configuration plutôt que d’utiliser la configuration globale. Dans ce cas vous trouverez votre fichier dans le dossier "$(solutiondir)\.vs\config\applicationhost.config".

Editez ce fichier et rechercher une ligne contenant une balise XML avec votre port du genre :

```xml
<binding protocol="http" bindingInformation="*:55713:localhost" />
```

Dupliquez ce noeud en l’ajoutant à la suite du premier et en modifiant "localhost" par l’adresse IP sur laquelle vous voulez-vous connecter. Vous obtenez quelque chose comme :

```xml
<binding protocol="http" bindingInformation="*:55713:localhost" />
<binding protocol="http" bindingInformation="*:55713:192.168.1.1" />
```

Enregistrez le fichier.

Lancez un invité de commande en mode administrateur, puis lancez la commande suivante :

```
netsh http add urlacl url=http://192.168.1.1:55713/ "user=Tout le monde"
```

Attention à bien saisir l’adresse IP avec le port. De même pour la partie "user", il s’agit du groupe autorisé, la plupart des exemples trouvés étant en anglais le groupe se nomme "Everyone", mais comme j’ai un poste en français, il faut penser à traduire le nom du groupe (c’est la fameuse petite blague dont je parlais en introduction :) ).

Voilà a partir de maintenant IIS Express est autorisé à écouter votre adresse IP et votre port quelque soit la source.

Relancez Visual Studio, et exécutez votre projet. Dans la systray (zone d’icône a coté de l’horloge) se trouve une icône "IIS Express",  faire un clic-droit et sélectionner le menu "Afficher toutes les applications". Vous devez voir apparaître votre site deux fois ; une avec le localhost et l’autre avec votre adresse IP.

![Afficher toutes les applications de IIS Express](http://blog.ygrenier.com/wp-content/uploads/2016/11/iis-express-toutes-applications.png)

Maintenant vous pouvez tester l’accès via votre adresse IP, même depuis votre poste de développement.

Certaines solutions trouvées sur internet indique d’utiliser l’étoile "\*" au lieu de l’adresse IP dans la commande "netsh". Cela permet de dire à IIS Express d’écouter TOUTES les adresse IPs que vous avez définies sur votre poste. Donc c’est une option de fainéant ;), mais je ne l’utilise pas car pour que cela fonctionne IIS Express doit <ins>obligatoirement</ins> être exécuté en **mode administrateur**. Et comme je suis très fainéant, je préfère lancer une commande "netsh" pour chaque IP une seule fois pour toute, que de lancer mon Visual Studio en mode admin à chaque fois que je dois bosser : non seulement il faut que j’accepte l’UAC mais en plus je ne peux plus utiliser mes raccourcis sur mes projets.

Donc a vous de choisir entre les deux options. En revanche n’utilisez pas l’étoile dans le fichier de configuration, personnellement je ne suis pas parvenu à le faire fonctionner.

A bientôt,

Yanos

