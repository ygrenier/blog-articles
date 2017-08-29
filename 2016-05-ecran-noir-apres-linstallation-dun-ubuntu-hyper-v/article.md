---
title: Ecran noir après l’installation d’un Ubuntu sur Hyper-V
published: 2016-05-10
categories: Trucs & Astuces
tags: Hyper-V, Linux
url: /2016/05/ecran-noir-apres-linstallation-dun-ubuntu-hyper-v
---

# Ecran noir après l'installation d'un Ubuntu sur Hyper-V

Lors d’une installation d’un Ubuntu sur un Hyper-V, une fois la VM rebootée, un écran noir apparaît ce qui ne vous permet pas de contrôler votre VM.

Ce problème apparaît en particulier quand on fait une installation depuis une image minimale. Si vous utilisez l’image "Ubuntu Server" vous ne devriez pas rencontrer le problème.

<!--more-->

Comme il faut faire des manipulations sur la machine il faut espérer que vous avez installé dés le départ le serveur SSH. Ce qui vous permet de vous connecter à votre VM a distance.

Si ce n’est pas le cas, essayez de redémarrer avec votre image de boot et utilisez le mode récupération pour faire les modifications. Toutefois je n’ai pas testé cette méthode car j’installe toujours le serveur SSH.

Le problème provient de la configuration de GRUD (outil gérant le boot), les options sont définies pour un affichage "Desktop" donc graphique. Donc on va modifier cette configuration.

```
sudo vim /etc/default/grub
```

Il faut supprimer "quiet splash" dans l’une des lignes suivantes:

```
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""
```

Ensuite il faut décommenter la ligne

```
GRUB_TERMINAL=console
```

On enregistre les modifications et on quitte l’éditeur.

Ensuite on enregistre les modifications dans le boot avec la commande:

```
sudo update-grub
```

ensuite on reboot l’image.

```
sudo reboot
```

Et voilaaa !

Voici les références sur lesquelles je me suis basé

- [http://wellsie.net/p/395/](http://wellsie.net/p/395/)

A bientôt,

Yanos

