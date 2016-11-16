<!--/2016/03/pinvoke-utiliser-dll-32bits-64bits-fonction-de-plateforme-mode-any-cpu-->
# P/Invoke : utiliser une DLL native 32bits ou 64bits en fonction de la plateforme en mode "ANY CPU"

Lorsqu’on utilise P/Invoke avec une DLL native (via l’attribut DllImport), nous devons forcer la compilation de notre application dans la plateforme de la DLL (32 ou 64 bits). Toutefois on peut avoir la DLL dans les deux plateformes (par exemple Lua 5.3) et vouloir compiler notre application en "Any CPU" et que la DLL soit chargée dans sa bonne version. Ce qui n’est pas possible directement avec DllImport().

<!--more-->

# Fonctionnement de DllImportAttribute

L’attribut ```[DllImport()]``` ne créé pas de liaisons statiques (la DLL est chargée et liée au démarrage de l’exécutable), mais au contraire indique une liaison dynamique.

Cela implique qu’au moment de l’appel de la méthode le système va déterminer si la DLL est en mémoire, si ce n’est pas le cas il va la chercher en fonction du nom indiqué dans ```DllImport()``` et la monter en mémoire. Une fois en mémoire la DLL sert de référence pour chaque ```DllImport()``` de même nom.

Notre problème réside dans le fait que nous devons définir une constante comme nom dans ```DllImport()``` ce qui veut dire qu’on ne peut pas "calculer" le nom en fonction du contexte.

Pour charger la DLL, le Framework .NET utilise l’API Windows on ne peut plus classique "[LoadLibrary()](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684175(v=vs.85).aspx)" a qui on transmet soit un nom de fichier qui va être recherché dans le dossier de l’exécutable appelant ou dans le système (principalement "%windir%\System32"), soit en transmettant le chemin complet de la DLL.

# Détails importants

Il y a deux choses qui vont nous intéresser:

- Une fois chargée la DLL reste mémoire
- Une fois chargée la DLL sert de référence pour toutes les DLL de même nom

Ce qu’il faut savoir c’est qu’une DLL est identifiée par son nom de fichier uniquement, pas par son chemin complet.

# Résolution de notre problème

Donc si nous provoquons un appel à "LoadLibrary()" avec la DLL en fonction de la plateforme avant que le premier ```DllImport()``` ne soit exécuté (l’idéal étant le constructeur), et que notre DLL possède le même nom quelque soit la plateforme, alors nous avons résolu notre problème.

L’idée générale est la suivante:

- Dans le projet créer les dossiers "x86" et "x64" dans lesquels on place une copie de notre DLL en fonction de la plateforme. Elle doit porter le même nom de fichier dans les deux dossiers. On marque chaque DLL comme "Contenu à copier si nouveau".
- Dans le constructeur statique de notre classe qui défini les méthodes externes (où se trouve nos ```DllImport()```) on détermine la plateforme, puis on calcul le nom complet de la DLL à utiliser.
- On fait un appel à "LoadLibrary" avec le nom complet

Désormais la DLL est chargée en mémoire dans la bonne version. Tous les ```DllImport()``` sur cette DLL utiliseront la version en mémoire.

# Mise en œuvre

Nous allons prendre le cas de Lua 5.3. Vous trouverez une implémentation complète sur GitHub (le projet n’est pas finalisé, mais le code est fonctionnel sur cette partie).

Lua possède une fonction ```lua_version``` qui retourne un pointeur sur la valeur de la version. Comme c’est un pointeur, si nous n’utilisons pas la bonne plateforme nous provoquerons une exception ```BadImageFormatException```. Donc nos tests sont facilement vérifiables :)

Les binaires se trouvent sur SourceForge : [http://luabinaries.sourceforge.net/download.html](http://luabinaries.sourceforge.net/download.html).

## Créer le projet

Nous allons créer une application Console.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/create-project.png)

Si vous créez un projet .Net 4.5 il faut désactiver l’option "Préférer 32 bits" pour que notre exemple fonctionne correctement.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/disable-32bits.png)

## Configuration des plateformes

Pour faire nos tests on va créer des configurations pour forcer la compilation de notre application en 32 bits (x86) ou en 64 bits (x64).

Nous allons commencer par la version 32 bits.

Ouvrir le gestionnaire de configuration:

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/open-configuration-manager.png)

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/configuration-manager.png)

Pour créer une nouvelle plateforme, cliquer sur cliquer sur le sélecteur "Plateforme de la solution active" et sélectionner "< Nouveau… >":

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/create-configuration.png)

Saisir **x86** dans la nouvelle plateforme

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/new-configuration.png)

et valider la création.

Vérifier que la nouvelle plateforme créée est sélectionnée comme plateforme active, et s’assurer que la "Plateforme" du projet est également en "x86".

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/check-plateform-configuration.png)

Répéter la procédure pour le 64 bits, en créant une configuration "**x64**".

Fermer le gestionnaire.

## Intégration des DLLs

Dans le projet créer un dossier "x86", récupérer la DLL "lua53.dll" en 32 bits depuis l’archive [http://sourceforge.net/projects/luabinaries/files/5.3.2/Tools%20Executables/lua-5.3.2_Win32_bin.zip/download](http://sourceforge.net/projects/luabinaries/files/5.3.2/Tools%20Executables/lua-5.3.2_Win32_bin.zip/download) et l’ajouter dans le dossier du projet.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/move-dll-in-x86.png)

Dans le propriétés de la DLL s’assurer qu’elle est définie comme "Contenu" et "Copier si plus récent".

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/mark-dll-as-content.png)

Faire la même chose avec la DLL 64 bits ([http://sourceforge.net/projects/luabinaries/files/5.3.2/Tools%20Executables/lua-5.3.2_Win64_bin.zip/download](http://sourceforge.net/projects/luabinaries/files/5.3.2/Tools%20Executables/lua-5.3.2_Win64_bin.zip/download)) que l’on va ajouter dans un dossier "x64".

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/add-dll-x64.png)

## Création du wrapper de la DLL

Ajoutons un fichier de classe "Lua.cs" la classe "Lua" va être statique dans laquelle nous allons mettre en place le DllImport pour accéder la fonction "lua_version".

```csharp
/// <summary>
/// Lua DLL Wrapper
/// </summary>
public static class Lua
{

  /// <summary>
  /// DLL Name
  /// </summary>
  const String LuaDllName = "Lua53.dll";

  /// <summary>
  /// Get Lua version
  /// </summary>
  /// <param name="L">Lua state. Can be null.</param>
  /// <returns>Number represents version</returns>
  [DllImport(LuaDllName, CallingConvention = CallingConvention.Cdecl, CharSet = CharSet.Ansi, EntryPoint = "lua_version")]
  private extern static IntPtr _lua_version(IntPtr L);
  public static double lua_version(IntPtr L)
  {
    var ptr = _lua_version(L);
    return (double)Marshal.PtrToStructure(ptr, typeof(double));
  }
}
```

- Nous définissons une constante "LuaDllName" qui contient le nom de la DLL pour des raisons pratiques
- On défini la méthode "_lua_version" avec un attribut ```DllImport``` et on la déclare privée. "extern" permet de ne pas avoir à déclarer un corps de méthode
- On créé une méthode "lua_version" qui va récupérer le pointeur (IntPtr), puis on utilise les méthodes de marshaling pour convertir les données dans la mémoire pointée en donnée managée (```double``` dans notre cas)

## Ajout du code de préchargement

Il nous faut d’abord définir l’accès à l’API ```LoadLibrary()``` pour charger la DLL.

```csharp
[DllImport("Kernel32.dll", CallingConvention = CallingConvention.StdCall, CharSet = CharSet.Ansi, SetLastError = false)]
private static extern IntPtr LoadLibrary([MarshalAs(UnmanagedType.LPStr)]string lpFileName);
```

Ensuite on défini le constructeur statique qui va charger la DLL en fonction

```csharp
  /// <summary>
  /// Preload the Lua DLL
  /// </summary>
  static Lua()
  {
    // Check the size of the pointer
    string folder = IntPtr.Size == 8 ? "x64" : "x86";
    // Build the full library file name
    string libraryFile = Path.Combine(Path.GetDirectoryName(typeof(Lua).Assembly.Location), folder, LuaDllName);
    // Load the library
    var res = LoadLibrary(libraryFile);
    if (res == IntPtr.Zero)
      throw new InvalidOperationException("Failed to load the library.");
    System.Diagnostics.Debug.WriteLine(libraryFile);
  }
```

Pour déterminer la plateforme il existe plusieurs méthodes. Dans notre cas j’utilise la technique de la taille du pointeur: sur une plateforme 32 bits un pointeur fait 4 octets (4 * 8 = 32 bits), alors que sur une plateforme 64 bits un pointeur fait 8 octets (8 * 8 = 64 bits).

En fonction de cette taille on détermine le dossier dans lequel on va trouver la DLL, et on la charge en mémoire avec l’API ```LoadLibrary```.

Une fois chargée, nous affichons le nom du fichier dans la console de déboguage pour nous permettre de vérifier le nom du fichier.

## Création du code de test

Dans notre "Program.cs" nous allons ajouter le code qui va appeler "lua_version" pour afficher le numéro de version de la DLL via l’API Lua.

```csharp
class Program
{
  static void Main(string[] args)
  {
    try
    {
      Console.WriteLine("Version {0}", Lua.lua_version(IntPtr.Zero));
    }
    catch (Exception ex)
    {
      Console.WriteLine("Err ({0}): {1}", ex.GetType().Name, ex.GetBaseException().Message);
    }
    Console.Read();
  }
}
```

Sélectionnons la configuration "x86":

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/select-x86.png)

Et exécutons notre programme.

![](http://blog.ygrenier.com/wp-content/uploads/2016/11/display-version.png)

Si nous regardons la console de débogage nous verrons que la librairie chargée est "x86/lua53.dll".

Appuyer sur une touche pour quitter le programme.

Sélectionner "x64" dans les configurations pour tester la version 64 bits et vérifier que le programme fonctionne et que c’est bien la librairie "x64/lua53.dll" qui est chargée.

# Conclusion

En compilant en "Any CPU" le Framework .Net va déterminer la plateforme en cours d’exécution, la taille du pointeur sera modifiée en conséquence, et la bonne version de la librairie sera chargée.

Vous trouverez finalement sur Github [https://github.com/ygrenier/tests/tree/master/PInvokeMultiPlatform](https://github.com/ygrenier/tests/tree/master/PInvokeMultiPlatform) le projet résultat de cet article. Vous y trouverez également un article similaire mais avec une approche plus progressive.

A bientôt

Yanos
