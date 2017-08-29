---
title: Installation d'extensions IIS sur Windows 10
published: 2016-06-22
categories: Développement
tags: IIS, Windows 10, WPI
url: /2016/06/installation-extensions-iis-windows-10
---

# Installation d'extensions IIS sur Windows 10

Dans un [précédent article](http://blog.ygrenier.com/2015/11/installer-php-manager-sur-windows-10/) j’expliquais que l’on rencontrait quelques difficultés pour installer "PHP Manager" sur IIS de Windows 10, en particulier nous rencontrons un problème de version de IIS (10 sous Windows 10) que l’installeur ne connaît pas.

Il s’avère que ce n’est pas uniquement l’extension "PHP Manager" qui se trouve dans cette situation, d’autres extensions comme le module "Réécriture d’URL 2.0" sont sujet à ce problème car elles n’ont pas été corrigées pour supporter cette version de Windows.

<!--more-->

# Contourner le problème de version

Pour résoudre ce problème, il suffit de modifier le numéro de version de IIS dans la base de registre, lancer l’installation de l’extensions, et restaurer le numéro de version.

Pour faire croire que l’on est sur une version 8:

- Ouvrir le programme "regedit.exe" (Editeur de registre)
- Ouvrir la clé ```HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC\Parameters```
- La valeur "MajorVersion" doit être "0x0000000a" soit la valeur 10 en décimal.

![Regedit](http://blog.ygrenier.com/wp-content/uploads/2016/11/Regedit.png)

- Modifier la valeur en mettant "8"
- Ouvrir WPI et installer l'extension
- Restaurer la valeur "MajorVersion" avec la valeur "a" en hexadécimal ou "10" en décimal.

Et voilààààà :)

A bientôt,

Yanos
