---
title: La fonction BrowserLink ne fonctionne pas avec les applications ASP.NET Core
published: 2016-10-29
categories: .NET Core, C#/.NET, Développement
tags: .NET, .NET Core, C#, Visual Studio
url: /2016/10/fonction-browserlink-ne-fonctionne-applications-asp-net-core
---

# La fonction BrowserLink ne fonctionne pas avec les applications ASP.NET Core

Si vous êtes un fidèle de Visual Studio pour faire du développement d’application ASP.NET, vous utilisez certainement l’une des fonctionnalités très intéressante "BrowserLink" qui couplée aux "Web Essentials" rend le développement Web extrêmement pratique.

Si maintenant vous vous lancez dans le développement "ASP.NET Core" le futur du développement Web de Microsoft, vous pouvez constater avec regret que cette fonctionnalité n’a pas l’air d’être disponible.

Pas de panique Yanos est là :)

<!--more-->


# Activation de la fonctionnalité dans votre application ASP.NET Core

La première chose à vérifier c’est que votre application inclus bien le paquet et le code nécessaire. C’est normalement le cas si vous utilisez le Template "Application Web".

## Dépendances

Dans votre fichier "project.json", vérifiez que vous avez le paquet "Microsoft.VisualStudio.Web.BrowserLink.Loader" à la version "14.0.0" (au moment où cet article a été écrit).

Si ce n’est pas le cas l’ajouter dans les dépendances:

```js
{
  ...
  "dependencies": {
    ...
    "Microsoft.VisualStudio.Web.BrowserLink.Loader": "14.0.0"
  },
  ...
}
```

## Configuration du plugin

Le BrowserLink n’est utilisé que lors du développement, donc nous devons l’activer que si on est dans l’environnement "development". Dans le "Startup.cs", repérer la méthode ```Configure```.

Insérer ou modifier le code suivant:

```csharp
if (env.IsDevelopment())
{
  app.UseDeveloperExceptionPage();
  app.UseBrowserLink();
}
else
{
  app.UseExceptionHandler("/Home/Error");
}
```

# Ca ne fonctionne toujours pas !!

Bon votre application est parfaitement configurée et ca ne fonctionne toujours pas:(

Il vous reste a réparer votre installation. En effet comme les outils de développement pour ASP.NET Core sont encore en preview, il y a quelques accrocs.

Dans la liste des programmes installés, rechercher "Microsoft ASP.NET 5 RC1 Update 1" et "Microsoft ASP.NET Web Frameworks and Tools 2015" puis réparez les (Cliquer sur le bouton "Modifier" puis choisir l’option "Repair").

Si ca ne suffit pas, alors désinstaller tout ce qui concerne ".Net Core" puis réinstaller les derniers outils depuis : [https://www.microsoft.com/net/core#windows](https://www.microsoft.com/net/core#windows).

# Pour finir

Sur tous les postes sur lesquels j’ai rencontré ce problème, la réparation a suffit. Toutefois j’ai fait un peu le ménage car j’avais installé les versions antérieures qui n’avaient pas été désinstallées.

A bientôt,

Yanos

