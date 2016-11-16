<!--/2015/03/small-basic-ligne-de-temps-pour-les-jeux/-->
# Small Basic – Ligne de temps pour les jeux

Dans un jeu plusieurs éléments sont généralement synchronisés suivant une ligne de temps. Par exemple une balle se déplaçant de 200 pixels en 2 secondes.

Nous avons plusieurs approches possibles, cet article va en étudier deux en particulier:

- une classique et simple, mais qui n’est pas toujours d’une grande précision.
- une seconde approche permettant  d’avoir d’une ligne de temps plus précise.

<!--more-->

# L'approche simple

L’approche la plus simple et la plus couramment utilisée est de calculer le temps qu’il faut attendre à chaque fois qu’on se déplace d’un pixel. Dans notre cas, 2 secondes correspondent à 2000 millisecondes et par conséquent on va se déplacer d’un pixel toutes les 10 ms (2000 ms/200 pixels). Dans une boucle on déplace notre balle d’un pixel et on attends 10ms, et on répète cela 200 fois, on aura déplacé notre balle de 200 pixels en 2 secondes.

```basic
' Durée de déplacement
delay = 2000
' Longueur de déplacement
move = 200

' Création de la balle
GraphicsWindow.Show()
ball = Shapes.AddEllipse(20,20)
ballPos = 0
Shapes.Move(ball, 10 + ballPos,10)

' Calcul la référence de temps
start = Clock.ElapsedMilliseconds
' Boucle
For i = 1 To move
  ' On déplace la balle d'un pixel
  ballPos = ballPos + 1
  Shapes.Move(ball, 10 + ballPos,10)
  ' On attends delay/move millisecondes
  Program.Delay(delay/move)
EndFor
' On calcul le temps qu'on a mis
GraphicsWindow.Title = Clock.ElapsedMilliseconds - start
```

Si nous exécutons ce programme notre balle se déplace apparemment de 200 pixels en 2 secondes, toutefois si nous regardons le titre de la fenêtre où nous affichons le temps réellement mis au final, on constate que l’ensemble à pris plus de temps (sur mon ordinateur nous sommes à peu près à 200 ms de plus).

Pourquoi un tel écart ? Tout simplement parce que notre programme ne fait pas que attendre, il déplace une balle, il fait des calculs, et cela prend du temps, et tout cumulé ca se rajoute à nos 2 secondes. Plus nous aurons un ordinateur puissant et plus l’écart se réduira, a l’inverse un ordinateur plus lent va augmenter cet écart. De plus dans un vrai jeu il faut également prendre en compte le fait le joueur va utiliser la souris ou le clavier ce qui va "ralentir" encore plus notre programme.

Notre balle ne va donc pas se trouver vraiment à l’endroit où elle devrait être en fonction de cet écart.

Ce manque de précision n’est pas toujours un problème dans un jeu, toutefois si nous voulons faire un quizz avec un chronomètre, et que nous voulons que tous les joueurs aient les mêmes chances, nous ne pouvons pas nous contenter de cela, sinon les joueurs qui auront un ordinateur moins puissant auront plus de temps pour répondre :)

Cette méthode nous pose également une seconde difficulté, imaginons le cas où nous avons plusieurs éléments qui se déplacent différemment (pas à la même vitesse par exemple), comment gérer les temps pour chacun des éléments ? Cette méthode ne nous le permet pas.

# Une ligne de temps plus précise

Pour résoudre notre problème de précision quel que soit l’ordinateur ou le temps consommé par le programme pour ces calculs, nous avons une autre approche disponible : le principe de la ligne de temps (timeline).

Une ligne de temps ou timeline représente le temps qui se « déroule » pendant notre jeu. A tout moment dans notre jeu elle est capable de nous dire où on se trouve par rapport au début de la timeline (ou de la durée qui nous intéresse), et grâce à cela on pourra déterminer dans quel état nous sommes sensé être dans notre jeu.

Si je reprends mon exemple de ma balle, quand je serais à 500ms dans ma timeline, je sais que ma balle sera à 50 pixels de sa position d’origine, à 1500ms je serais à 150 pixels.

En Small Basic c’est assez simple de simuler une timeline. On utilise ```Clock.ElapsedMilliseconds``` qui nous retourne le nombre de millisecondes depuis le 1 Janvier 1900. Si j’enregistre cette valeur au moment où je démarre ma timeline :

```basic
startTimeline = Clock.ElapsedMilliseconds
```

ensuite si je veux savoir où je me trouve dans ma timeline, il suffit de faire une soustraction :

```basic
timeline = Clock.ElapsedMilliseconds - startTimeline
```

Reprenons notre balle qui se déplace et cette fois déplaçons la en fonction de la timeline pour plus de précision.

```basic
' Durée de déplacement
delay = 2000
' Longueur de déplacement
move = 200

' Création de la balle
GraphicsWindow.Show()
ball = Shapes.AddEllipse(20,20)
ballPos = 0
Shapes.Move(ball, 10 + ballPos,10)

' Démarre la timeline
startTimeline = Clock.ElapsedMilliseconds
' Boucle
While ballPos < move
  ' On calcul où on se trouve dans la timeline
  timeline = Clock.ElapsedMilliseconds - startTimeline
  ' On calcul la position de la balle en fonction de la timeline
  ballPos = (timeline * move) / delay
  ' Déplacement de la balle
  Shapes.Move(ball, 10 + ballPos,10)
  ' Simule un calcul plus ou moins long qui perturbe notre programme
  If ballPos < move/2 Then
    Program.Delay(Math.GetRandomNumber(150))
  EndIf
EndWhile
' On calcul le temps qu'on a mis
GraphicsWindow.Title = Clock.ElapsedMilliseconds - startTimeline
```

Si nous lançons ce programme nous constatons que nous n’avons plus vraiment d’écart (sauf de temps en temps du aux aléas de l’informatique).

Pourtant dans la première partie de la distance à parcourir je fais des pauses aléatoires, on le constate car la balle "saccade", mais elle revient toujours à la place qui est la sienne par rapport à la timeline.

Pour calculer la position de la balle il suffit de faire une règle de trois :

```
2000 ms (delay) => 200 pixels (move)
timeline        => ballPos
```

donc

```
ballPos = (move * timeline) / ballPos
```

# Le temps écoulé

Très bien nous avons une balle qui est synchronisée a notre timeline. Notre exemple est plutôt simple, mais dans la réalité d’un jeu, devoir recalculer une situation depuis le début de la timeline n’est pas toujours la meilleure option.

Il est parfois préférable de calculer le temps qui s’est écoulé entre deux consultations de la timeline. Avec notre balle, plutôt que de calculer la position en fonction de la timeline à chaque itération de la boucle, nous allons calculer le temps qu’il c’est écoulé depuis la dernière boucle, ensuite nous calculerons le nombre de pixels que notre balle est sensée avoir parcouru dans ce laps de temps.

```basic
' Durée de déplacement
delay = 2000
' Longueur de déplacement
move = 200

' Création de la balle
GraphicsWindow.Show()
ball = Shapes.AddEllipse(20,20)
ballPos = 0
Shapes.Move(ball, 10 + ballPos,10)

' Démarre la timeline
startTimeline = Clock.ElapsedMilliseconds
lastTimeline = 0
' Boucle
While ballPos < move
  ' On calcul où on se trouve dans la timeline
  timeline = Clock.ElapsedMilliseconds - startTimeline
  ' On calcul le temps écoulé depuis la dernière boucle
  elapsed = timeline - lastTimeline
  ' On calcul le déplacement qu'a fait la balle pendant ce laps de temp
  ballMovement = (elapsed * move) / delay
  ' On ajoute le déplacement à la position de la balle
  ballPos = ballPos + ballMovement
  ' Déplacement de la balle
  Shapes.Move(ball, 10 + ballPos,10)
  ' Simule un calcul plus ou moins long qui perturbe notre programme
  If ballPos < move/2 Then
    Program.Delay(Math.GetRandomNumber(150))
  EndIf
  ' On enregistre la timeline pour la prochaine boucle
  lastTimeline = timeline
EndWhile
' On calcul le temps qu'on a mis
GraphicsWindow.Title = Clock.ElapsedMilliseconds - startTimeline
```

Cette méthode est  absolument nécessaire dans certains cas. Imaginons que notre balle se déplace en fonction des touches clavier qu’appuie le joueur, il nous est impossible de calculer la position la balle depuis le début de la timeline. En revanche faire ce calcul en fonction du temps écoulé est faisable.

```basic
' Durée de déplacement
delay = 2000
' Longueur de déplacement
move = 200

' Création de la balle
GraphicsWindow.Show()
ball = Shapes.AddEllipse(20,20)
ballPos = 0
ballPosX = 0
ballPosY = 0
offX = 1
offY = 0
Shapes.Move(ball, 10 + ballPosX,10 + ballPosY)

' Branche l'appui de touche
GraphicsWindow.KeyDown = OnKeyDown

' Démarre la timeline
startTimeline = Clock.ElapsedMilliseconds
lastTimeline = 0
' Boucle
While ballPos < move
  ' On calcul où on se trouve dans la timeline
  timeline = Clock.ElapsedMilliseconds - startTimeline
  ' On calcul le temps écoulé depuis la dernière boucle
  elapsed = timeline - lastTimeline
  ' On calcul le déplacement qu'a fait la balle pendant ce laps de temp
  ballMovement = (elapsed * move) / delay
  ' On ajoute le déplacement à la position de la balle
  ballPos = ballPos + ballMovement
  ballPosX = ballPosX + (ballMovement * offX)
  ballPosY = ballPosY + (ballMovement * offY)
  ' Déplacement de la balle
  Shapes.Move(ball, 10 + ballPosX,10 + ballPosY)
  ' Simule un calcul plus ou moins long qui perturbe notre programme
  If ballPos < move/2 Then
    Program.Delay(Math.GetRandomNumber(150))
  EndIf
  ' On enregistre la timeline pour la prochaine boucle
  lastTimeline = timeline
EndWhile
' On calcul le temps qu'on a mis
GraphicsWindow.Title = Clock.ElapsedMilliseconds - startTimeline

Sub OnKeyDown
  If GraphicsWindow.LastKey = "Up" Then
    offX = 0
    offY = -1
  ElseIf GraphicsWindow.LastKey = "Down" Then
    offX = 0
    offY = 1
  ElseIf GraphicsWindow.LastKey = "Left" Then
    offX = -1
    offY = 0
  ElseIf GraphicsWindow.LastKey = "Right" Then
    offX = 1
    offY = 0
  EndIf
EndSub
```

Notre balle parcours toujours 200 pixels en 2 secondes, mais si on appui sur les touches fléchées notre balle change de direction.

# Une timeline pour chacun

Dans un jeu tout baser sur une seule timeline n’est pas forcément la chose la plus pratique.

La première raison est que nous avons souvent besoin de recommencer une timeline, par exemple si en appuyant sur une touche mon personnage se déplace de 100 pixels, je vais démarrer une timeline uniquement pour calculer le déplacement de ce personnage pour savoir où il se trouve par rapport à cette timeline.

Chaque élément du jeu aura ses propres timelines en fonction de ces besoins. Par exemple on peut avoir une timeline pour calculer son déplacement en cours, et une timeline pour calculer une animation. Le jeu lui-même à souvent une timeline, pour connaître la durée de la partie par exemple.

```basic

' Construction du tableau de jeu
GraphicsWindow.Show()
GraphicsWindow.KeyDown = OnKeyDown
gameTitle = Shapes.AddText("")

' Premier élément : une balle
elm[0]["shape"] = Shapes.AddEllipse(20,20)
elm[0]["x"] = 40
elm[0]["y"] = 40
elm[0]["startTimeline"] = -1
elm[0]["timeline"] = -1
Shapes.Move(elm[0]["shape"], elm[0]["x"], elm[0]["y"])

' Second élément : une boite
GraphicsWindow.BrushColor = "Yellow"
elm[1]["shape"] = Shapes.AddRectangle(20,20)
elm[1]["x"] = 40
elm[1]["y"] = 140
elm[1]["startTimeline"] = Clock.ElapsedMilliseconds
elm[1]["timeline"] = -1
Shapes.Move(elm[1]["shape"], elm[1]["x"], elm[1]["y"])

' Initialisation du jeu
gameStartTime = Clock.ElapsedMilliseconds
gameTime = 0

' Boucle de jeu
While "True"
  ' Calcul la timeline du jeu
  gameTime = Clock.ElapsedMilliseconds -  gameStartTime

  '  Affiche le titre
  title = "Temps de jeu : " + Math.Floor(gameTime / 1000) + " secondes"
  title = title + " | Balle : " + elm[0]["timeline"]
  title = title + " | Boîte : " + elm[1]["timeline"]
  Shapes.SetText(gameTitle, title)

  ' Calcul la timeline de chaque élement
  For i = 0 To 1
    stl = elm[i]["startTimeline"]
    If stl > 0 Then
      elm[i]["timeline"] = Clock.ElapsedMilliseconds - stl
    Endif
  EndFor

  ' Calcul la pulsation de la balle :
  If elm[0]["timeline"] > 0 Then
    zoom =1+(0.5*Math.Sin(elm[0]["timeline"]/100))
    Shapes.Zoom(elm[0]["shape"], zoom, zoom)
  EndIf

  ' Calcul la rotation de la boîte : Un tour complet en deux secondes
  Shapes.Rotate(elm[1]["shape"], Math.Remainder((elm[1]["timeline"]/(1000 * 2)) * 360, 360) )

  ' On repose l'ordinateur
  Program.Delay(50)
EndWhile

' Gestion des touches
Sub OnKeyDown
  ' Si on appui sur la touche Echappe on arrête le jeu
  If GraphicsWindow.LastKey = "Escape" Then
    Program.End()
  EndIf
  ' Si on appuie sur la barre espace on active/désactive la pulsation de la balle
  If GraphicsWindow.LastKey = "Space" Then
    If elm[0]["startTimeline"] > 0 Then
      elm[0]["startTimeline"] = -1
      elm[0]["timeline"] = -1
    Else
      elm[0]["startTimeline"] = Clock.ElapsedMilliseconds
      elm[0]["timeline"] = 0
    EndIf
  EndIf
EndSub

```

Dans l’exemple précédent nous avons trois timelines, une timeline pour le jeu qui affiche le temps depuis le début du jeu. Une seconde timeline pour l’objet "Balle" qui démarre et s’arrête quand on appui sur la barre espace. La dernière timeline est pour la "Boite" qui permet le calcul la rotation.

# Conclusion

Les timelines vous serviront à gérer les déplacements, animations, effets visuels, etc.

Donc en fonction de votre besoin, n’hésitez pas à utiliser l’une ou plusieurs de ces techniques.

Vous pouvez retrouver l’ensemble des exemples sur le dépôts GitHub [https://github.com/Small-Basic-French/Exemples/tree/master/Timeline](https://github.com/Small-Basic-French/Exemples/tree/master/Timeline) .

A bientôt,

Yanos
