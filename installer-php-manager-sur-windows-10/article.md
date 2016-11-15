<!-- /​2015/​11/​installer-​php-​manager-​sur-​windows-​10 -->
# Installer PHP Manager sur Windows 10

Depuis WPI (Microsoft Web Platform Installer) on peut installer PHP pour IIS (Internet Information Service ou en français Gestionnaire des services Internet) sous Windows. Pour faciliter la gestion de PHP sous IIS généralement on installe un module qui se nomme "PHP Manager".

Malheureusement depuis Windows 10 l’installation de ce module pose un problème. Heureusement il y a une solution :)


Deux problèmes sont courants :

- Un problème de Framework .Net 2.0 qui n’est pas installé sur la machine
- Sous Windows 10, un numéro de version n’est pas encore supporté par l’installeur

<!--more-->

# Vérifier que le Framework .Net 2.0 est installé

La première chose est de s’assurer que le Framework .Net en version 2.0 est installé sur la machine. Ce n’est pas le cas par défaut sur les Windows 7+.

Pour ce faire il faut ouvrir la section « Programmes et Fonctionnalités » du panneau de configuration. Puis cliquer sur « Activer ou désactiver des fonctionnalités Windows »

![Programmes et fonctionnalités](http://blog.ygrenier.com/wp-content/uploads/2016/11/ProgrammesEtFonctionnalites.png)

Assurez vous que la fonctionnalité ".NET Framework 3.5 (inclut .NET 2.0 et 3.0)" soit cochée.

![Fonctionnalité .NET 3.5](http://blog.ygrenier.com/wp-content/uploads/2016/11/FonctionnaliteNet35.png)

# Contourner le problème de version sous Windows 10

L’installeur de PHP Manager ne connaît pas encore Windows 10, et il fait une vérification de la version des services Web disponibles sur la machine. Comme il ne connaît pas Windows 10, l’installation échoue.

Pour contourner cela, on va simplement lui faire croire qu’il est sur une version 8, juste le temps de l’installation.

- Ouvrir le programme "regedit.exe" (Editeur de registre)
- Ouvrir la clé ```HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC\Parameters```
- La valeur "MajorVersion" doit être "0x0000000a" soit la valeur 10 en décimal.

![Regedit](http://blog.ygrenier.com/wp-content/uploads/2016/11/Regedit.png)

- Modifier la valeur en mettant "8"
- Ouvrir WPI et installer "PHP Manager", ce qui devrait désormais se faire sans problème.
- Restaurer la valeur "MajorVersion" avec la valeur "a" en hexadécimal ou "10" en décimal.

**Références :**

- [https://phpmanager.codeplex.com/workitem/2653](https://phpmanager.codeplex.com/workitem/2653)
- [http://forums.iis.net/t/1228101.aspx?PHP+Manager+Installation+for+IIS](http://forums.iis.net/t/1228101.aspx?PHP+Manager+Installation+for+IIS)
- [http://answers.microsoft.com/en-us/windows/forum/windows_10-other_settings/php-manager-for-iis-on-windows-10/33ef32f0-6a86-4803-abc1-6de81110f9a8?auth=1](http://answers.microsoft.com/en-us/windows/forum/windows_10-other_settings/php-manager-for-iis-on-windows-10/33ef32f0-6a86-4803-abc1-6de81110f9a8?auth=1)

A bientôt,

Yanos
