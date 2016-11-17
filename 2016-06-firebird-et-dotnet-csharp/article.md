<!--/2016/06/firebird-et-dotnet-csharp-->
# Firebird et .NET/C# #

Pour les besoins d’un petit projet j’ai recherché une solution de base de données légère, trouvant que "SQL Express" était surdimensionné pour le besoin. Et c’est là que m’est revenu un nom du fin fond de mes souvenir de développeur Delphi: [Firebird](http://www.firebirdsql.org/).


Firebird (appelé également FB) est une base de données open-source basée initialement sur la base de données Interbase. Vous trouverez plus d’informations sur [http://www.firebirdsql.org/](http://www.firebirdsql.org/).

Cette base de données  est extrêmement légère (~60Mo une fois installée), elle applique une grande partie de la norme SQL 92, supporte un langage de programmation PSQL permettant des triggers et des procédures, elle supporte le mode embarqué, et bien d’autres choses, et comble du bonheur le DataProvider pour .Net est disponible, ainsi qu’un provider Entity Framework. Pour un rapide tour de ce que sait faire Firebird vous pouvez regarder l’introduction en français (elle date un peu) [http://www.firebirdnews.org/docs/fb2min_fr.html](http://www.firebirdnews.org/docs/fb2min_fr.html).

Bon j’ai brossé un portrait plutôt flatteur, mais elle a également quelques défauts :)

- la référence aux bases de données sur fait sur le fichier de la base, ce qui implique de connaître les bases qui se trouvent sur le serveur, il n’y a pas de "référentiel" comme on peut avoir sur MySQL ou SQLServer. Mais ca a également son bon coté, vous pouvez monter votre fichier ".fdb" sur votre serveur, et le référencer, Firebird se débrouille. Ce qui permet également de proposer des bases de données en lecture seule par exemple sur un CD/DVD interactif.
- pour les habitués de SQLServer ou de MySQL la syntaxe SQL et certaines fonctionnalités peuvent être un peu déroutantes (la gestion des autoincrément/identité par exemple). Il faut un peu étudier la documentation pour s’en sortir.
- il n’y a pas d’outils d’administration aussi "pratiques" que "MySQL Workbench" ou "SQL Server Management" (pour SQL Server). C’est mon avis personnel qui n’engage que moi ;) Même si je trouve [FlameRobin](http://www.flamerobin.org/) efficace, il lui manque un coté pratique je trouve.

Passons à la pratique.

# Installation de Firebird

Actuellement il y a deux versions officielles:

- Firebird 2.5 : [http://www.firebirdsql.org/en/firebird-2-5-5/](http://www.firebirdsql.org/en/firebird-2-5-5/)
- Firebird 3.0 : [http://www.firebirdsql.org/en/firebird-3-0-0/](http://www.firebirdsql.org/en/firebird-3-0-0/)

La version 3.0 améliore l’architecture et la sécurité, ce qui peut poser des problèmes pour réutiliser du code "ancien" utilisant le client 2.5. Des modifications dans la configuration permettent de compenser ces problèmes. C’est un problème que l’on rencontre avec .Net/C# mais que l’on corrige facilement.

Mais si vous n’avez jamais utilisé FB vous pouvez commencer par la version 3.0.

Pour l’installation rien de plus simple : lancez l’installeur de la version qui vous intéresse et suivez le guide avec les options par défaut.

Si on vous demande un mot de passe pour le "SYSDBA" qui est le super admin de Firebird, utiliser "masterkey" qui est le mot de passe par défaut.

# Outil d’administration

Comme je l’ai dit je ne trouve pas les outils disponibles très pratiques, mais [FlameRobin](http://www.flamerobin.org/) reste mon préféré.

Là également il y a un installeur.

Vous trouverez la liste des outils disponibles ici: [http://www.firebirdsql.org/en/third-party-tools/](http://www.firebirdsql.org/en/third-party-tools/).

# Gestion d’une base de données

On va faire simple, FB nous fourni une base de données d’exemple "Employee.fdb", nous allons vérifier son accès avec FlameRobin.

Lancer FlameRobin, le serveur local est déjà enregistré par défaut sous le nom "Localhost".

![FlameRobin](http://blog.ygrenier.com/wp-content/uploads/2016/11/FlameRobin.png)

Faire un clic droit sur le serveur > "Register existing database …" pour enregistrer une base de données

![Enregistrer une base de données](http://blog.ygrenier.com/wp-content/uploads/2016/11/RegisterExistingDatabase.png)


Saisir les différentes informations:

- Display name : nom affiché dans l’arbre de FlameRobin
- Database path: nom du fichier de la base de données
- User name/Password: devinez !!
- Charset: Code page utilisé, la base d’exemple n’en n’a pas donc on laisse à « NONE ».
- Role: on ne saisi rien, les rôles sont des sortes de groupe d’utilisateurs.

Enregistrer et connecter la base de données, avec un double-clic sur la base ou avec un clic droit sur la base de données > "Connect".

![Base connectée](http://blog.ygrenier.com/wp-content/uploads/2016/11/DatabaseConnected.png)

Vous pouvez voir les différents éléments que contient la base de données.

Vous pouvez parcourir la base de données, notamment les tables et les vues, et utiliser le menu contextuel pour ouvrir des requêtes pour afficher les données.

![Requête](http://blog.ygrenier.com/wp-content/uploads/2016/11/MakeQuery.png)

# Points à savoir sur Firebird

Il y a quelques points à savoir sur Firebird avant de commencer.

## Les noms des objets

Une des particularités "piégeuse" de Firebird c’est que les noms des objets (tables, vues , procédures, etc.) sont sensibles à la casse. Ce qui veut que "Employee" et "EMPLOYEE" sont deux objets différents pour FB.

Mais les concepteurs de FB ont été malins; par défaut tous les noms sont convertis en majuscules dans une requête, ainsi ma requête:

```sql
select * from employee
```

est transformée en

```sql
select * from EMPLOYEE
```

Ainsi nous n’avons pas de mauvaises surprises si on se trompe dans la casse. C’est la raison pour laquelle si nous regardons le contenu de la base "employee.fdb" tous les objets ont un nom en majuscules, cela évite les problèmes.

Mais il faut savoir que si on utilise les noms selon la norme SQL (le nom placé entre guillemet), la casse n’est pas modifiée.

```sql
select * from "Employee"
```

provoque une erreur "table unknown" car cette fois dans notre requête le nom ne sera pas mis en majuscule.

## Les transactions

Firebird est l’une des premières base données à proposée un mécanisme très puissant de transactions.

Les transactions sont des mécanismes qui permettent de créer des "unité de travail" que l’on peut valider ou annuler d’un bloc. C’est le principe "d’atomicité" en on peut faire un ensemble de requêtes qui sont toutes validées d’un coup ou pas.

L’exemple classique concerne l’a réservation d’une place d’avion. On va exécuter au moins deux requêtes, l’une pour insérer le billet du voyageur dans la base, l’autre réserver la place dans l’avion. Imaginons qu’une erreur a lieu lors de la deuxième requête, la place n’est pas réservée, mais le billet est dans la base. Donc le voyageur a officiellement une place, mais sa place a été perdue dans l’avion. Une transaction nous permet d’enregistrer une liste de requêtes dans une "transaction", et de valider (Commit) ou annuler (Rollback) toutes les requêtes en une seule fois. Ainsi si tout va bien dans notre réservation, on "Commit" et toutes les informations sont correctement enregistrées dans la base. En revanche si on détecte une erreur lors de notre transaction, on fait un "Rollback" ce qui va annuler toutes les requêtes qui ont été faite, la base revient à son état initial.

Les transactions sont "isolées" des autres connections à la base, certains paramètres permettent de gérer cette isolation, mais plupart du temps, une autre connexion ne voit pas les modifications qui sont en cours dans les transactions tant qu’elles ne sont pas commitées.

Pour plus d’informations sur les transactions lire l’article de Frédéric Brouard : [http://sqlpro.developpez.com/cours/sqlaz/techniques/#L1](http://sqlpro.developpez.com/cours/sqlaz/techniques/#L1)

FB utilise énormément les transactions, cela fait partie des "bonnes pratiques" de FB, par conséquent la plupart des outils, dont FlameRobin, utilise les transactions, il faut donc penser à valider les transactions pour "appliquer" les modifications à la base de données.

Dans une fenêtre de requête de FlameRobin on a deux boutons qui nous permettent de commiter ou de rollbacker la transaction en cours. Dans la barre d’état en bas à droite s’affiche l’état de la transaction en cours.

![Gestion de la transaction](http://blog.ygrenier.com/wp-content/uploads/2016/11/ManageTransaction.png)

Donc n’oubliez pas de commiter votre transaction avant de fermer une fenêtre sinon vous ne verrez pas apparaître vos modifications dans d’autre requêtes.

# Utilisation dans un projet .Net/C# #

Pour cela rien de plus simple il suffit d’installer le package Nuget "[FirebirdSql.Data.FirebirdClient](https://www.nuget.org/packages/FirebirdSql.Data.FirebirdClient)" :

```
PM> Install-Package FirebirdSql.Data.FirebirdClient
```

ATTENTION: cette librairie n’est compatible que .Net 40/4.5.

Une fois installée vous avez accès à l’espace de nom "FirebirdSql.Data.Firebirdient" donnant accès aux objets ```FbConnection```, ```FbCommand```, etc. selon la nomenclature classique des providers ADO.NET.

Pour se connecter à notre base de données "employee" en local :

```csharp
using (var con = new FbConnection("data source=localhost;port number=3050;server type=Default;user id=sysdba;password=masterkey;initial catalog=employee.fdb;dialect=3;"))
{
  con.Open();

  cmd.CommandText = "select * from employee";
  cmd.CommandType = System.Data.CommandType.Text;
  using (var rdr = cmd.ExecuteReader())
  {
    for (int i = 0; i < rdr.FieldCount; i++)
      Console.Write(rdr.GetName(i) + " | ");
    Console.WriteLine();
    while (rdr.Read())
    {
      for (int i = 0; i < rdr.FieldCount; i++)
      {
        Console.Write(rdr.GetValue(i));
        Console.Write(" | ");
      }
      Console.WriteLine();
    }
  }
}
```

Malheureusement si vous avons un FB 3.0, une erreur "Incompatible wire encryption levels requested on client and server " va venir briser votre espoir d’afficher la liste des employés de votre base :)

C’est normal, comme je l’ai expliqué en début d’article, la version 3.0 a fortement modifié la sécurité, et il s’avère que le client .Net ne supporte pas encore ces nouvelles fonctionnalités. Nous allons donc dire à notre serveur d’être un peu moins "sécuritaire".


- Ouvrir le gestionnaire de service
- Rechercher le service "Firebird Server – DefaultInstance"
- l’arrêter
- Ouvrir un éditeur de texte en mode administrateur
- Ouvrir le fichier "C:\Program Files\Firebird\Firebird_3_0\firebird.conf" (ou "C:\Program Files (x86)\Firebird\Firebird_3_0\firebird.conf" si vous avez installé la version 32bits).
- Rechercher la ligne ```#WireCrypt = Enabled (for client) / Required (for server)``` normalement c’est la ligne 567.
- Ajouter la ligne

```
WireCrypt = Enabled
```

- Enregistrer les modifications
- Relancer le service

Normalement maintenant votre code fonctionne.

Nous avons simplement indiqué au serveur de crypter la communication uniquement si le client le supporte.

# Conclusion

Voilà maintenant vous savez que vous pouvez utiliser un autre outil de base de données en .Net.

A bientôt,

Yanos
