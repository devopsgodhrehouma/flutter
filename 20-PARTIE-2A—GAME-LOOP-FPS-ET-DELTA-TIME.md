# PARTIE 2A — LES FONDAMENTAUX DU JEU 2D
# CHAPITRE 20 — LA BOUCLE DE JEU, LES FPS ET LE DELTA TIME

> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 19 — Flutter en accéléré pour le jeu (`runApp`, `StatefulWidget`, `setState`, `Ticker`, `CustomPainter`)
> **Ce que vous saurez faire à la fin :** écrire une boucle de jeu complète en Dart et en Flutter pur, mesurer les FPS, et faire bouger un objet à la même vitesse réelle sur toutes les machines grâce au delta time.

---

## 20.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qui distingue techniquement un jeu d'une application classique ;
- décrire le schéma **entrées → mise à jour → rendu** et le redessiner de mémoire ;
- écrire une première boucle de jeu en Dart console ;
- expliquer pourquoi un `while (true)` gèle une application Flutter ;
- définir une **image** (frame) et un **taux d'images par seconde** (FPS) ;
- convertir un nombre de FPS en durée de frame en millisecondes, et l'inverse ;
- démontrer le bug d'un déplacement écrit `x += 5` par frame ;
- définir le **delta time** et écrire `x += vitesse * dt` ;
- raisonner en **pixels par seconde** plutôt qu'en pixels par frame ;
- calculer un `dt` à partir d'un `Stopwatch` ;
- utiliser un `Ticker` et `SchedulerBinding` pour piloter une boucle Flutter ;
- afficher un compteur de FPS et le lisser avec une moyenne glissante ;
- reconnaître un `dt` aberrant après un pic de lag et le plafonner avec `clamp` ;
- implémenter un **pas de temps fixe** avec un accumulateur ;
- comparer pas fixe et pas variable et choisir en connaissance de cause ;
- découpler la mise à jour du rendu et interpoler entre deux états ;
- mettre le jeu en pause, au ralenti ou en accéléré avec un facteur de temps ;
- nommer précisément ce que le moteur Flame fera à votre place à partir du chapitre 27 ;
- assembler un petit moteur de boucle réutilisable pour tous les chapitres suivants.

---

## 20.1 — Ce qui distingue un jeu d'une application classique

Vous avez écrit dix-huit chapitres de Dart, puis un chapitre de Flutter en accéléré. Vous savez donc déjà construire une application : des widgets, un `setState`, un bouton, une liste. Le chapitre 20 est le moment où l'on change de monde.

Regardons d'abord une application classique. Prenons une simple liste de potions avec un bouton « acheter ».

```text
  APPLICATION CLASSIQUE

  L'utilisateur ne touche à rien
      -> l'écran ne change pas
      -> le processeur ne fait rien
      -> la batterie ne descend pas

  L'utilisateur appuie sur « acheter »
      -> l'application se réveille
      -> elle modifie son état
      -> elle redessine
      -> elle se rendort
```

C'est un fonctionnement **événementiel** : le programme dort, un événement le réveille, il travaille une fraction de seconde, il se rendort. C'est exactement ce que fait `setState` : il signale « quelque chose a changé, redessine ».

Maintenant, un jeu.

```text
  JEU

  L'utilisateur ne touche à rien
      -> le gobelin avance quand même
      -> la torche vacille quand même
      -> le chronomètre descend quand même
      -> l'écran est redessiné 60 fois par seconde
```

Un jeu ne dort jamais. Même quand le joueur pose sa manette, le monde continue d'exister. C'est la première grande différence, et elle change absolument tout dans la façon d'écrire le code.

Récapitulons dans un tableau.

| Critère | Application classique | Jeu |
| --- | --- | --- |
| Déclencheur du travail | un événement utilisateur | le temps qui passe |
| Fréquence de redessin | quand l'état change | 60 fois par seconde (ou plus) |
| État au repos | rien ne bouge | tout continue de bouger |
| Notion centrale | l'événement | l'image (frame) |
| Question posée au code | « que faire quand on clique ? » | « où en est le monde maintenant ? » |
| Consommation | proche de zéro au repos | continue |
| Structure | callbacks + widgets | une boucle infinie |

Retenez la ligne la plus importante :

> Dans une application, **c'est l'utilisateur qui décide** quand le code s'exécute.
> Dans un jeu, **c'est le temps qui décide**.

Tout le reste du chapitre découle de cette phrase.

---

## 20.2 — Une application réagit, un jeu tourne en permanence

Pour bien ancrer l'idée, comparons deux morceaux de code qui traitent le même sujet : faire avancer un gobelin.

### Version « application »

```dart
// Pseudo-code Flutter, style application classique.
ElevatedButton(
  onPressed: () {
    setState(() {
      gobelinX += 10;
    });
  },
  child: const Text('Avancer'),
)
```

Le gobelin avance de 10 pixels **quand on clique**. Si personne ne clique, il ne bouge pas. C'est un comportement d'application.

### Version « jeu »

```dart
// Pseudo-code, style jeu.
void miseAJour(double dt) {
  gobelinX += 60 * dt; // 60 pixels par seconde, en continu
}
```

Le gobelin avance **tout le temps**, sans que personne ne clique. Il n'y a plus d'événement déclencheur, il y a un flux de temps.

La différence est profonde. Dans le premier cas, l'unité de raisonnement est « le clic ». Dans le second, l'unité de raisonnement est « la seconde ».

Voici un schéma qui compare les deux dans le temps.

```text
  APPLICATION (événementielle)

  temps ──────────────────────────────────────────────────────>
          │              │                    │
        clic           clic                  clic
          ▼              ▼                    ▼
        [travail]     [travail]            [travail]
          ░░░░░          ░░░░░                ░░░░░
        (rien)  (rien, rien, rien...)   (rien, rien...)


  JEU (boucle continue)

  temps ──────────────────────────────────────────────────────>
        ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼
       [f][f][f][f][f][f][f][f][f][f][f][f][f][f][f][f][f][f]
        une frame toutes les ~16 millisecondes, sans interruption
```

Sur la ligne du jeu, aucun trou. Le programme travaille en permanence. Ce travail permanent porte un nom : **la boucle de jeu**, en anglais *game loop*.

> **Remarque.** Une application Flutter ordinaire redessine aussi, parfois beaucoup. La différence n'est pas « redessiner » mais **« redessiner sans qu'on le lui demande »**. Un jeu s'auto-réveille.

---

## 20.3 — La boucle de jeu : le schéma fondamental

Voici le schéma le plus important de toute la PARTIE 2. Prenez le temps de le lire, ligne par ligne. Vous le reverrez dans chaque chapitre suivant, et Flame ne fera rien d'autre que l'exécuter pour vous.

```text
  ┌───────────────────────────────────────────────────────────────┐
  │                      LA BOUCLE DE JEU                         │
  └───────────────────────────────────────────────────────────────┘

        ┌──────────────────────────────────────────────┐
        │                                              │
        │   ┌──────────────┐                           │
        └──>│  1. ENTRÉES  │  Que fait le joueur ?     │
            │   (input)    │  touches, doigt, souris   │
            └──────┬───────┘                           │
                   │                                   │
                   ▼                                   │
            ┌──────────────┐                           │
            │ 2. MISE À    │  Le monde avance de dt.   │
            │    JOUR      │  positions, vies, IA,     │
            │  (update)    │  collisions, score        │
            └──────┬───────┘                           │
                   │                                   │
                   ▼                                   │
            ┌──────────────┐                           │
            │  3. RENDU    │  On DESSINE l'état        │
            │   (render)   │  actuel du monde          │
            └──────┬───────┘                           │
                   │                                   │
                   └───────────────────────────────────┘
                        et on recommence, ~60x / seconde
```

Trois étapes, toujours dans le même ordre, répétées indéfiniment.

**1. Entrées.** On lit ce que le joueur demande. On ne fait rien d'autre : on note « la flèche droite est enfoncée », « le doigt est à (120, 340) ». On ne déplace personne à cette étape.

**2. Mise à jour.** C'est le cœur du jeu. On fait avancer le monde d'une petite tranche de temps. Le joueur se déplace, les gobelins patrouillent, la torche perd de l'huile, les projectiles avancent, les collisions sont détectées, le score monte. **Aucun pixel n'est dessiné ici.**

**3. Rendu.** On regarde l'état du monde et on le dessine. **Aucune règle de jeu ici.** On ne décide pas si le joueur perd une vie pendant le rendu ; on affiche seulement le nombre de vies qu'il a.

Cette séparation stricte entre `update` et `render` est une règle d'or. Un débutant mélange les deux et se retrouve avec un jeu dont la difficulté change quand on branche un écran plus rapide. Nous verrons pourquoi en détail à partir de la section 20.10.

> **À retenir.** `update` fait avancer le monde. `render` le photographie. Jamais l'inverse, jamais les deux en même temps.

Un mot de vocabulaire pour la suite : une exécution complète des trois étapes s'appelle **un tour de boucle**, et produit **une image**. Nous y revenons en 20.6.

---

## 20.4 — Une première boucle naïve en Dart console

Passons au code. Nous restons volontairement en Dart console, sans Flutter, pour que rien ne cache le mécanisme.

Notre fil rouge est le **Donjon de Dart**, le même donjon que le mini-projet du chapitre 18. Un héros est à la position `x = 0` et avance vers la droite.

Première tentative, la plus naïve possible : une boucle `for` qui simule 10 tours.

```dart
void main() {
  double heroX = 0;
  int frame = 0;

  for (frame = 1; frame <= 10; frame++) {
    // 1. ENTRÉES (rien pour l'instant : le héros avance tout seul)

    // 2. MISE À JOUR
    heroX += 5;

    // 3. RENDU
    print('Frame $frame : héros en x = $heroX');
  }

  print('Fin de la boucle.');
}
```

**Résultat :**

```text
Frame 1 : héros en x = 5.0
Frame 2 : héros en x = 10.0
Frame 3 : héros en x = 15.0
Frame 4 : héros en x = 20.0
Frame 5 : héros en x = 25.0
Frame 6 : héros en x = 30.0
Frame 7 : héros en x = 35.0
Frame 8 : héros en x = 40.0
Frame 9 : héros en x = 45.0
Frame 10 : héros en x = 50.0
Fin de la boucle.
```

C'est déjà une boucle de jeu. Elle a les trois étapes. Elle produit dix images. Elle s'arrête toute seule, ce qui est pratique pour apprendre mais faux pour un vrai jeu : un jeu tourne jusqu'à ce que le joueur quitte.

Rendons le rendu un peu plus visuel. Nous allons dessiner le donjon en caractères.

```dart
void main() {
  const int largeurDonjon = 40;
  double heroX = 0;

  for (int frame = 1; frame <= 8; frame++) {
    // 2. MISE À JOUR
    heroX += 4;

    // 3. RENDU
    final int colonne = heroX.round();
    final StringBuffer ligne = StringBuffer();
    for (int i = 0; i < largeurDonjon; i++) {
      ligne.write(i == colonne ? '@' : '.');
    }
    print('${frame.toString().padLeft(2)} |$ligne|');
  }
}
```

**Résultat :**

```text
 1 |....@...................................|
 2 |........@...............................|
 3 |............@...........................|
 4 |................@.......................|
 5 |....................@...................|
 6 |........................@...............|
 7 |............................@...........|
 8 |................................@.......|
```

Vous voyez le héros `@` traverser le donjon, une image à la fois. C'est très exactement ce que fait un jeu, en beaucoup plus rapide et avec des pixels au lieu de points.

Il manque cependant deux choses essentielles :

1. la boucle ne tourne pas indéfiniment ;
2. la boucle ne tient aucun compte du **temps réel** : les huit images sont produites en une fraction de milliseconde.

Ajoutons le temps. En Dart console, on peut faire une pause avec `Future.delayed`, ce qui demande une fonction `async` (revoyez le chapitre 15 si nécessaire).

```dart
import 'dart:async';

Future<void> main() async {
  const int largeurDonjon = 30;
  double heroX = 0;

  for (int frame = 1; frame <= 6; frame++) {
    heroX += 4;

    final int colonne = heroX.round().clamp(0, largeurDonjon - 1);
    final StringBuffer ligne = StringBuffer();
    for (int i = 0; i < largeurDonjon; i++) {
      ligne.write(i == colonne ? '@' : '.');
    }
    print('|$ligne|');

    // On attend 200 millisecondes avant la frame suivante.
    await Future<void>.delayed(const Duration(milliseconds: 200));
  }

  print('Le héros a atteint la porte.');
}
```

**Résultat :**

```text
|....@.........................|
|........@.....................|
|............@.................|
|................@.............|
|....................@.........|
|........................@.....|
Le héros a atteint la porte.
```

Les six lignes s'affichent maintenant l'une après l'autre, à 200 millisecondes d'intervalle. Le programme dure environ 1,2 seconde. Nous venons d'introduire la notion de **cadence** : la boucle ne va plus « aussi vite que possible », elle respecte un rythme.

> **Remarque.** `Future.delayed` n'est pas la bonne méthode pour un vrai jeu, nous le verrons en 20.16. Mais elle est parfaite pour comprendre.

---

## 20.5 — Pourquoi `while (true)` bloque tout

La boucle de jeu est « infinie ». Un débutant en déduit naturellement qu'il faut écrire :

```dart
void main() {
  double heroX = 0;

  while (true) {
    heroX += 1;
    print('x = $heroX');
  }
}
```

En Dart console, ce programme fonctionne : il affiche des lignes jusqu'à ce que vous l'arrêtiez avec `Ctrl+C`. Il consomme 100 % d'un cœur du processeur, mais il tourne.

Dans Flutter, le même code **gèle l'application**. Écran figé, aucun bouton ne répond, et au bout de quelques secondes le système d'exploitation propose de fermer l'application « qui ne répond plus ».

Pour comprendre, il faut se souvenir d'une chose vue au chapitre 15 : Dart est **mono-thread** pour votre code. Il n'y a qu'un seul fil d'exécution, appelé l'**isolate principal**, et une file d'attente d'événements (l'*event loop*).

```text
  L'ISOLATE PRINCIPAL DE FLUTTER

  ┌──────────────────────────────────────────────────────────────┐
  │  Une seule file d'attente, traitée par un seul fil           │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  [tap du doigt] [timer] [image réseau] [redessin] [clavier]  │
  │        ▲                                                     │
  │        └── traité un par un, dans l'ordre                    │
  └──────────────────────────────────────────────────────────────┘

  Le redessin de l'écran EST un de ces événements.
```

Autrement dit : c'est le **même** fil qui exécute votre code et qui dessine l'interface.

Maintenant, regardez ce que fait `while (true)` :

```text
  AVEC while (true)

  ┌──────────────────────────────────────────────────────────────┐
  │  [votre while (true)] ############################ (jamais   │
  │                                                     fini)    │
  ├──────────────────────────────────────────────────────────────┤
  │  En attente, pour toujours :                                 │
  │    [tap du doigt]  [redessin]  [animation]  [clavier]        │
  └──────────────────────────────────────────────────────────────┘

  Rien ne sera JAMAIS traité : le fil ne rend jamais la main.
```

Votre boucle prend le fil et ne le rend jamais. Le redessin, qui est un événement comme un autre, n'a jamais son tour. L'écran reste donc figé sur la dernière image affichée.

Le point à retenir, et il est contre-intuitif :

> Une boucle de jeu ne s'écrit **jamais** avec `while (true)` dans une application graphique.
> Elle s'écrit comme une **fonction rappelée à chaque image** par le système.

C'est un renversement mental important. Vous n'écrivez pas :

```text
  « répète pour toujours : mets à jour, dessine »
```

Vous écrivez :

```text
  « voici ce qu'il faut faire à chaque image ; appelle-moi quand c'est l'heure »
```

La boucle existe toujours, mais c'est **le moteur graphique qui la tient**, pas vous. Votre code n'est qu'un morceau branché dedans. En Flutter, le mécanisme qui vous rappelle s'appelle le `Ticker` (section 20.16).

Et le `Future.delayed` de la section précédente ? Il ne bloque pas, lui, car `await` **rend la main** à la file d'événements pendant l'attente. C'est pour cela qu'il fonctionne. Mais sa précision est mauvaise et il n'est pas synchronisé avec l'écran ; ce n'est donc pas la bonne solution non plus.

| Approche | Bloque l'interface ? | Synchronisé avec l'écran ? | Verdict |
| --- | --- | --- | --- |
| `while (true)` | oui | non | à proscrire |
| `Future.delayed` en boucle | non | non | pédagogique seulement |
| `Timer.periodic` | non | non | acceptable, mais dérive |
| `Ticker` | non | **oui** | **la bonne solution** |

---

## 20.6 — La notion d'image (frame)

Une **image**, ou **frame**, est le résultat d'un tour complet de boucle : entrées lues, monde mis à jour, écran dessiné.

L'écran de votre téléphone n'affiche pas un mouvement continu. Il affiche une succession d'images fixes, très rapprochées. Votre œil fait le reste : au-delà d'une certaine cadence, il perçoit du mouvement là où il n'y a qu'une suite de photos.

```text
  CE QUE L'ÉCRAN AFFICHE RÉELLEMENT

  frame 1        frame 2        frame 3        frame 4
  ┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐
  │@       │     │  @     │     │    @   │     │      @ │
  └────────┘     └────────┘     └────────┘     └────────┘
     t=0ms          t=16ms         t=33ms        t=50ms

  CE QUE L'OEIL PERÇOIT

  ┌────────────────────────────────────────────────────┐
  │  @ ──────────────────────────────────────────────> │
  └────────────────────────────────────────────────────┘
                un mouvement fluide
```

Chaque image est une **photographie de l'état du monde à un instant donné**. C'est pour cela que le rendu ne doit rien décider : une photographie ne change pas ce qu'elle photographie.

Trois conséquences pratiques découlent de cette définition.

**Première conséquence : une frame a un budget de temps.** Si vous voulez 60 images par seconde, chaque image doit être produite en moins de 1/60 de seconde. Tout ce que vous mettez dans `update` et dans `render` doit tenir dans ce budget.

**Deuxième conséquence : entre deux images, rien n'existe.** Si un projectile va très vite et que vous le déplacez de 200 pixels d'un coup, il n'existe à aucun moment aux positions intermédiaires. C'est la cause du bug de *tunneling* que nous verrons au chapitre 24 sur les collisions.

**Troisième conséquence : le mouvement est une illusion d'échantillonnage.** Ce que vous programmez n'est pas « le héros glisse », mais « le héros est ici, puis là, puis là ». Cette liste de positions doit être calculée en fonction du temps écoulé, pas en fonction du nombre de tours de boucle. C'est tout le sujet du delta time.

Vérifions cela avec un petit programme qui compte les images produites en une seconde environ.

```dart
import 'dart:async';

Future<void> main() async {
  int images = 0;
  final Stopwatch chrono = Stopwatch()..start();

  while (chrono.elapsedMilliseconds < 1000) {
    images++;
    // Une frame « vide » : on ne fait presque rien.
    await Future<void>.delayed(const Duration(milliseconds: 16));
  }

  chrono.stop();
  print('Images produites : $images');
  print('Durée réelle     : ${chrono.elapsedMilliseconds} ms');
}
```

**Résultat (ordre de grandeur, variable selon la machine) :**

```text
Images produites : 58
Durée réelle     : 1004 ms
```

Nous avons demandé 16 millisecondes d'attente, ce qui donnerait théoriquement 62 images. Nous en obtenons 58. La différence vient du temps que prend le reste du travail, et de l'imprécision du minuteur. Retenez ce chiffre : **la cadence réelle n'est jamais exactement la cadence demandée**.

---

## 20.7 — Les FPS

**FPS** signifie *frames per second*, en français **images par seconde**. C'est le nombre d'images produites et affichées en une seconde.

```text
  FPS = nombre d'images / nombre de secondes
```

Si votre jeu produit 120 images en 2 secondes, il tourne à 60 FPS.

C'est l'indicateur numéro un de la santé d'un jeu. Un jeu à 60 FPS est fluide. Un jeu à 20 FPS est saccadé et désagréable. Un jeu dont les FPS varient sans arrêt entre 60 et 25 est pire qu'un jeu stable à 30 : l'irrégularité se voit davantage que la lenteur.

Il faut distinguer trois nombres que les débutants confondent.

| Nom | Signification | Exemple |
| --- | --- | --- |
| Taux de rafraîchissement de l'écran | ce que le matériel peut afficher | 60 Hz, 90 Hz, 120 Hz |
| FPS du jeu | ce que votre code arrive à produire | 47 FPS |
| FPS affichés | ce que le joueur voit réellement | min(les deux), en simplifiant |

Un écran à 60 Hz ne montrera jamais plus de 60 images par seconde, même si votre jeu en calcule 300. Inversement, un écran à 120 Hz n'aidera pas si votre code ne produit que 30 images par seconde.

Écrivons un compteur de FPS en console. Nous faisons tourner une boucle pendant trois secondes et nous comptons.

```dart
import 'dart:async';

Future<void> main() async {
  final Stopwatch total = Stopwatch()..start();
  int imagesTotal = 0;

  int imagesSeconde = 0;
  int prochaineSeconde = 1000;

  while (total.elapsedMilliseconds < 3000) {
    imagesTotal++;
    imagesSeconde++;

    if (total.elapsedMilliseconds >= prochaineSeconde) {
      print('FPS de la seconde ${prochaineSeconde ~/ 1000} : $imagesSeconde');
      imagesSeconde = 0;
      prochaineSeconde += 1000;
    }

    await Future<void>.delayed(const Duration(milliseconds: 16));
  }

  total.stop();
  final double fpsMoyen = imagesTotal / (total.elapsedMilliseconds / 1000);
  print('---');
  print('Images totales : $imagesTotal');
  print('FPS moyen      : ${fpsMoyen.toStringAsFixed(1)}');
}
```

**Résultat (ordre de grandeur) :**

```text
FPS de la seconde 1 : 58
FPS de la seconde 2 : 59
FPS de la seconde 3 : 58
---
Images totales : 176
FPS moyen      : 58.6
```

Notez la façon de calculer le FPS moyen : **images divisées par secondes**. C'est la seule formule à connaître, et nous la réutiliserons dans le compteur graphique de la section 20.18.

> **Remarque.** Les FPS ne sont pas un objectif esthétique, ce sont un **budget**. Nous allons maintenant voir combien de millisecondes ce budget représente.

---

## 20.8 — 30, 60, 120 FPS : ce que cela change

Les trois cadences que vous rencontrerez sont 30, 60 et 120 images par seconde.

```text
  UNE SECONDE, VUE À TROIS CADENCES

  30 FPS   |█   |█   |█   |█   |█   |█   |█   |█   | ...
           <--33ms-->

  60 FPS   |█ |█ |█ |█ |█ |█ |█ |█ |█ |█ |█ |█ |█ | ...
           <-16ms->

  120 FPS  |█|█|█|█|█|█|█|█|█|█|█|█|█|█|█|█|█|█|█|█| ...
           <8ms>
```

Ce que change concrètement chaque palier :

**30 FPS.** Suffisant pour un jeu lent : un jeu de cartes, un puzzle, un jeu de gestion, un jeu au tour par tour. Le mouvement reste lisible. En revanche, un mouvement rapide devient visiblement saccadé, et la latence entre l'appui sur une touche et la réaction à l'écran atteint 33 millisecondes au minimum, ce qui se sent dans un jeu d'action.

**60 FPS.** La référence. C'est la cadence de la quasi-totalité des écrans de téléphone et d'ordinateur portable. Le mouvement est fluide, la latence est faible. C'est l'objectif par défaut de tout ce que nous ferons dans cette formation.

**120 FPS.** Réservé aux écrans dits « à haut taux de rafraîchissement », de plus en plus courants sur les téléphones récents. Le gain est réel mais subtil, surtout perceptible sur les mouvements de caméra rapides. Le coût est double : il faut produire deux fois plus d'images, donc chaque image doit être calculée deux fois plus vite.

Le tableau des conséquences :

| Cadence | Budget par image | Ressenti | Usage typique |
| --- | --- | --- | --- |
| 24 FPS | 41,7 ms | cinéma, saccadé en interactif | jamais pour un jeu |
| 30 FPS | 33,3 ms | acceptable, un peu mou | jeux lents, mobiles anciens |
| 60 FPS | 16,7 ms | fluide, standard | **notre objectif** |
| 90 FPS | 11,1 ms | très fluide | mobiles récents, VR |
| 120 FPS | 8,3 ms | très fluide | écrans haut de gamme |
| 144 FPS | 6,9 ms | très fluide | moniteurs de jeu PC |

Un point capital, et c'est la raison d'être de la moitié de ce chapitre :

> **Vous ne choisissez pas les FPS.** C'est le matériel du joueur qui les impose.
> Votre code doit produire **le même jeu** à 30, à 60 et à 120 FPS.

Un jeu dont le héros va deux fois plus vite sur un téléphone haut de gamme est un jeu cassé. Nous allons voir exactement pourquoi, et comment l'éviter.

---

## 20.9 — Le temps d'une frame en millisecondes

Passons de la cadence à la durée. Les deux sont les deux faces d'une même pièce.

```text
  durée d'une frame (secondes)      = 1 / FPS
  durée d'une frame (millisecondes) = 1000 / FPS

  FPS = 1000 / durée d'une frame (ms)
```

Vérifions :

```text
  60 FPS   ->  1000 / 60  = 16,67 ms
  30 FPS   ->  1000 / 30  = 33,33 ms
  120 FPS  ->  1000 / 120 =  8,33 ms

  16,67 ms ->  1000 / 16,67 = 60 FPS
  50 ms    ->  1000 / 50    = 20 FPS
```

Écrivons ces conversions en Dart, avec un petit utilitaire que vous garderez.

```dart
double msParFrame(double fps) => 1000 / fps;
double fpsDepuisMs(double ms) => 1000 / ms;

void main() {
  const List<double> cadences = [24, 30, 60, 90, 120, 144];

  print('FPS  ->  ms par image');
  for (final double f in cadences) {
    print('${f.toStringAsFixed(0).padLeft(3)}  ->  '
        '${msParFrame(f).toStringAsFixed(2)} ms');
  }

  print('');
  print('ms par image  ->  FPS');
  for (final double ms in [8.3, 16.7, 33.3, 50, 100]) {
    print('${ms.toString().padLeft(5)} ms  ->  '
        '${fpsDepuisMs(ms).toStringAsFixed(1)} FPS');
  }
}
```

**Résultat :**

```text
FPS  ->  ms par image
 24  ->  41.67 ms
 30  ->  33.33 ms
 60  ->  16.67 ms
 90  ->  11.11 ms
120  ->  8.33 ms
144  ->  6.94 ms

ms par image  ->  FPS
  8.3 ms  ->  120.5 FPS
 16.7 ms  ->  59.9 FPS
 33.3 ms  ->  30.0 FPS
   50 ms  ->  20.0 FPS
  100 ms  ->  10.0 FPS
```

Ce tableau est votre **budget**. À 60 FPS, vous disposez de 16,67 millisecondes pour tout faire : lire les entrées, mettre à jour cent entités, détecter les collisions, dessiner le décor, les sprites et l'interface.

Et vous n'avez même pas ces 16,67 millisecondes pour vous seul. Flutter doit encore composer les couches, envoyer les commandes au GPU, gérer le système. Une règle de terrain :

> Visez **moins de 10 millisecondes** de travail dans votre code par image.
> Au-delà, vous n'avez plus de marge pour les pics.

Un dernier point qui surprend souvent. Passer de 60 à 120 FPS ne « gagne » que 8,3 millisecondes, alors que passer de 15 à 30 FPS en gagne 33. Les gains de fluidité sont donc très inégaux :

```text
  ms gagnées en montant d'un palier

  15 -> 30 FPS   : 66,7 - 33,3 = 33,4 ms gagnées   ÉNORME
  30 -> 60 FPS   : 33,3 - 16,7 = 16,6 ms gagnées   très net
  60 -> 120 FPS  : 16,7 -  8,3 =  8,4 ms gagnées   subtil
 120 -> 240 FPS  :  8,3 -  4,2 =  4,1 ms gagnées   imperceptible
```

C'est pour cela qu'on se bat pour sortir des 30 FPS, et beaucoup moins pour dépasser 60.

---

## 20.10 — Le problème : toutes les machines ne vont pas à la même vitesse

Nous arrivons au problème central du chapitre.

Vous développez le Donjon de Dart sur votre ordinateur. Le jeu tourne à 60 FPS. Vous réglez la vitesse du héros pour qu'elle vous paraisse bonne. Vous êtes content.

Puis :

- votre camarade lance le jeu sur un téléphone à 120 Hz : **le héros va deux fois trop vite** ;
- votre professeur le lance sur une vieille tablette qui tient 30 FPS : **le héros va deux fois trop lentement** ;
- vous ouvrez vingt onglets et le jeu tombe à 45 FPS : **le héros ralentit en cours de partie**.

```text
  LE MÊME CODE, TROIS MACHINES

  Machine A (30 FPS)   Machine B (60 FPS)   Machine C (120 FPS)

  30 frames/s          60 frames/s          120 frames/s
  x += 5 par frame     x += 5 par frame     x += 5 par frame
  = 150 px/s           = 300 px/s           = 600 px/s
       │                    │                     │
       ▼                    ▼                     ▼
  ┌─────────┐          ┌─────────┐          ┌─────────┐
  │@..      │          │....@    │          │........@│
  └─────────┘          └─────────┘          └─────────┘
   lent                 normal               ultra-rapide
```

Le code est identique. Le comportement du jeu ne l'est pas. C'est le bug le plus classique du développement de jeux, et il a une cause unique :

> Le code raisonne en **pixels par image**, alors que le joueur perçoit des **pixels par seconde**.

Le nombre d'images par seconde varie d'une machine à l'autre, et même d'une seconde à l'autre sur la même machine. Toute grandeur exprimée « par image » est donc une grandeur instable.

Ce que cela casse, concrètement, dans un vrai jeu :

| Élément écrit « par frame » | Ce qui casse |
| --- | --- |
| déplacement du héros | vitesse différente selon la machine |
| gravité | hauteur de saut différente |
| cadence de tir | plus de balles sur une machine rapide |
| perte d'énergie | on meurt plus vite sur une machine rapide |
| chronomètre | le compte à rebours n'est pas le même |
| animation de sprite | le personnage marche trop vite |
| descente d'un compteur de score | le score n'est plus comparable |

Autrement dit, tout le jeu. La solution s'appelle le **delta time**, et nous y venons après avoir bien vu le bug en action.

---

## 20.11 — Démonstration du bug : un mouvement de `x += 5` par frame

Ne croyez pas sur parole : mesurons.

Le programme suivant simule la même scène (« le héros avance pendant 2 secondes ») sur trois machines fictives de cadences différentes. La logique de déplacement est la mauvaise : `x += 5` à chaque image.

```dart
/// Simule une machine qui tourne à [fps] images par seconde
/// pendant [secondes] secondes, avec un déplacement PAR FRAME.
double simulerParFrame(int fps, double secondes) {
  final int nombreImages = (fps * secondes).round();
  double x = 0;

  for (int i = 0; i < nombreImages; i++) {
    x += 5; // MAUVAIS : 5 pixels par image
  }

  return x;
}

void main() {
  const double duree = 2.0;

  print('Déplacement « x += 5 » par image, pendant $duree s');
  print('');
  print('Machine        Images   Position finale');
  print('------------------------------------------');

  for (final int fps in [30, 60, 120]) {
    final int images = (fps * duree).round();
    final double x = simulerParFrame(fps, duree);
    print('${fps.toString().padLeft(3)} FPS'.padRight(15) +
        images.toString().padLeft(6) +
        x.toStringAsFixed(0).padLeft(18));
  }
}
```

**Résultat :**

```text
Déplacement « x += 5 » par image, pendant 2.0 s

Machine        Images   Position finale
------------------------------------------
 30 FPS            60               300
 60 FPS           120               600
120 FPS           240              1200
```

Le verdict est sans appel. Au bout des **mêmes** deux secondes, le héros est à 300, 600 ou 1200 pixels selon la machine. Sur un écran de 800 pixels de large, il est encore au premier tiers sur la machine lente et déjà sorti de l'écran sur la machine rapide.

Illustrons visuellement.

```text
  APRÈS 2 SECONDES (écran de 800 px)

   30 FPS  |@@@@@@@@@@@@..............................|  x = 300
   60 FPS  |@@@@@@@@@@@@@@@@@@@@@@@@..................|  x = 600
  120 FPS  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|  x = 1200 (hors écran)
```

Le saut souffre du même mal. Si l'impulsion et la gravité sont exprimées « par image », la trajectoire compte toujours le même nombre d'images, mais ces images ne durent pas la même chose : un saut de 48 images dure 1,6 seconde à 30 FPS et seulement 0,4 seconde à 120 FPS. Même hauteur, quatre fois plus rapide, jeu injouable.

C'est le même bug sous un autre angle : **quand on compte en images, on ne contrôle pas le temps réel**.

> **À retenir.** Tout ce qui s'écrit `+= constante` dans `update` est un bug en puissance.
> Nous allons remplacer chaque `+= constante` par `+= vitesse * dt`.

---

## 20.12 — Le delta time : définition

Le **delta time** est le temps écoulé depuis l'image précédente.

Le mot « delta » vient de la lettre grecque Δ, qui désigne en mathématiques et en physique une **différence**. `dt` se lit donc « différence de temps ».

```text
  dt = instant de cette image − instant de l'image précédente
```

En code, on le note presque toujours `dt`, parfois `deltaTime` ou `elapsed`.

Voici une timeline qui montre ce que représente `dt`.

```text
  TIMELINE DES FRAMES ET VALEURS DE dt

  temps (ms)  0      16      33      49      66      98     114
              │      │       │       │       │       │       │
  frames      F1     F2      F3      F4      F5      F6      F7
              ●──────●───────●───────●───────●───────●───────●
                 16     17      16      17      32      16
                 ms     ms      ms      ms      ms      ms
                  ▲                              ▲
                  │                              │
             frame normale               frame en retard
             dt = 0,016 s                (un pic de lag)
                                          dt = 0,032 s
```

Trois remarques essentielles sur ce schéma.

**Premièrement, `dt` n'est pas constant.** Il vaut 16 ou 17 millisecondes la plupart du temps, mais il grimpe à 32 quand une image est manquée. C'est normal et c'est même le but : `dt` **mesure** la réalité au lieu de la supposer.

**Deuxièmement, `dt` est une mesure du passé.** Quand la frame F5 commence, on connaît le temps qu'a pris l'intervalle F4→F5. On ne connaît pas encore la durée de la frame en cours. On utilise donc toujours le `dt` de l'intervalle précédent, en supposant que la prochaine sera similaire.

**Troisièmement, l'unité compte énormément.** En développement de jeux, la convention universelle est :

> **`dt` est exprimé en SECONDES, sous forme de `double`.**

À 60 FPS, `dt` vaut donc environ `0.0167`, et non `16.7`. C'est la source d'erreur numéro un des débutants : un jeu qui va mille fois trop vite parce que `dt` était en millisecondes.

| Cadence | `dt` en secondes | `dt` en millisecondes |
| --- | --- | --- |
| 30 FPS | 0.0333 | 33.3 |
| 60 FPS | 0.0167 | 16.7 |
| 120 FPS | 0.0083 | 8.3 |

Flame, comme la plupart des moteurs, vous fournit `dt` en secondes dans `update(double dt)`. Nous ferons pareil dès maintenant, pour prendre les bons réflexes.

---

## 20.13 — `x += vitesse * dt`

Voici la formule centrale de tout le développement de jeux.

```text
  nouvelle position = ancienne position + vitesse × dt
```

En Dart :

```dart
x += vitesse * dt;
```

C'est tout. Cette ligne remplace `x += 5` et règle le problème de la section 20.11. Voyons pourquoi, avec des chiffres.

Posons `vitesse = 300` (nous verrons l'unité en 20.14) et regardons trois machines pendant une seconde.

```text
  Machine à 30 FPS
    dt = 1/30 = 0,0333 s
    par image : 300 × 0,0333 = 10 px
    30 images  : 30 × 10 = 300 px en une seconde

  Machine à 60 FPS
    dt = 1/60 = 0,0167 s
    par image : 300 × 0,0167 = 5 px
    60 images  : 60 × 5 = 300 px en une seconde

  Machine à 120 FPS
    dt = 1/120 = 0,0083 s
    par image : 300 × 0,0083 = 2,5 px
    120 images : 120 × 2,5 = 300 px en une seconde
```

Les trois machines aboutissent au **même endroit**. La machine rapide fait de plus petits pas, mais elle en fait plus. La machine lente fait de grands pas, mais moins nombreux. Le produit est constant.

C'est le principe physique le plus élémentaire : **distance = vitesse × temps**. Un jeu ne fait rien d'autre qu'appliquer cette formule, soixante fois par seconde.

Reprenons la simulation de la section 20.11, corrigée.

```dart
/// Simule une machine à [fps] images par seconde pendant [secondes],
/// avec un déplacement basé sur le DELTA TIME.
double simulerAvecDt(int fps, double secondes, double vitesse) {
  final int nombreImages = (fps * secondes).round();
  final double dt = 1 / fps;
  double x = 0;

  for (int i = 0; i < nombreImages; i++) {
    x += vitesse * dt; // BON : indépendant de la cadence
  }

  return x;
}

void main() {
  const double duree = 2.0;
  const double vitesse = 300; // pixels par seconde

  print('Déplacement « x += vitesse * dt », vitesse = $vitesse, '
      'pendant $duree s');
  print('');
  print('Machine        Images   Position finale');
  print('------------------------------------------');

  for (final int fps in [30, 60, 120, 144]) {
    final int images = (fps * duree).round();
    final double x = simulerAvecDt(fps, duree, vitesse);
    print('${fps.toString().padLeft(3)} FPS'.padRight(15) +
        images.toString().padLeft(6) +
        x.toStringAsFixed(1).padLeft(18));
  }
}
```

**Résultat :**

```text
Déplacement « x += vitesse * dt », vitesse = 300.0, pendant 2.0 s

Machine        Images   Position finale
------------------------------------------
 30 FPS            60             600.0
 60 FPS           120             600.0
120 FPS           240             600.0
144 FPS           288             600.0
```

Quatre machines, quatre cadences, **une seule position finale**. Le bug a disparu.

Comparons les deux approches côte à côte, une bonne fois pour toutes.

| | Sans `dt` | Avec `dt` |
| --- | --- | --- |
| Code | `x += 5;` | `x += 300 * dt;` |
| Unité de la constante | pixels par image | pixels par seconde |
| Résultat à 30 FPS (2 s) | 300 px | 600 px |
| Résultat à 60 FPS (2 s) | 600 px | 600 px |
| Résultat à 120 FPS (2 s) | 1200 px | 600 px |
| Le jeu est-il jouable partout ? | non | oui |

> **Règle absolue.** Dans `update`, **toute** grandeur qui évolue dans le temps est multipliée par `dt`.
> Position, vitesse, énergie, rechargement, chronomètre, opacité d'un fondu, tout.

---

## 20.14 — Unités : pixels par seconde

Changer de formule, c'est aussi changer d'unité de pensée. C'est le point que les débutants négligent, et qui rend pourtant les réglages du jeu compréhensibles.

```text
  AVANT : x += 5
          « 5 » = 5 pixels PAR IMAGE
          combien de pixels par seconde ? ça dépend de la machine.
          -> nombre sans signification physique

  APRÈS : x += 300 * dt
          « 300 » = 300 pixels PAR SECONDE
          -> nombre qui décrit le jeu, pas la machine
```

Avec l'ancienne écriture, si un collègue vous demande « à quelle vitesse va le gobelin ? », vous ne pouvez pas répondre. Avec la nouvelle, vous répondez « 80 pixels par seconde », et cette phrase a un sens sur toutes les machines du monde.

Voici un barème utile pour le Donjon de Dart, en supposant un héros d'environ 32 pixels de haut sur un écran de 800 × 450.

| Élément | Vitesse (px/s) | Repère intuitif |
| --- | --- | --- |
| Nuage de fond (parallaxe) | 10 | à peine perceptible |
| Torche qui vacille | 20 | très lent |
| Gobelin en patrouille | 60 | marche tranquille |
| Héros qui marche | 120 | traverse l'écran en ~6,5 s |
| Héros qui court | 220 | traverse l'écran en ~3,6 s |
| Flèche | 400 | rapide |
| Boule de feu du boss | 550 | très rapide |
| Projectile ultra-rapide | 900 | attention au tunneling (ch. 24) |

Une méthode simple pour choisir une vitesse : décidez **en combien de secondes** l'objet doit traverser l'écran.

```text
  vitesse (px/s) = largeur de l'écran (px) / durée voulue (s)

  Traverser 800 px en 4 s   ->  800 / 4  = 200 px/s
  Traverser 800 px en 1,5 s ->  800/1,5  = 533 px/s
```

Les autres unités qui vont avec :

| Grandeur | Unité | Écriture dans `update` |
| --- | --- | --- |
| Position | pixels (px) | `x += vitesse * dt` |
| Vitesse | pixels par seconde (px/s) | `vitesse += accel * dt` |
| Accélération | pixels par seconde carrée (px/s²) | constante |
| Gravité | px/s² | typiquement 800 à 2000 |
| Angle | radians (rad) | `angle += vitesseAngulaire * dt` |
| Vitesse angulaire | radians par seconde (rad/s) | `2 * pi` = un tour par seconde |
| Régénération d'énergie | points par seconde | `energie += 5 * dt` |
| Chronomètre | secondes | `tempsRestant -= dt` |

Un exemple complet en console, qui met tout cela ensemble.

```dart
void main() {
  const double dt = 1 / 60; // on simule 60 FPS
  const double vitesseHero = 120;   // px/s
  const double regenEnergie = 5;    // points/s
  const double vitesseRotation = 3.14159; // rad/s (un demi-tour par seconde)

  double x = 0;
  double energie = 40;
  double angle = 0;
  double tempsRestant = 3; // secondes

  int frame = 0;
  while (tempsRestant > 0) {
    frame++;

    x += vitesseHero * dt;
    energie += regenEnergie * dt;
    angle += vitesseRotation * dt;
    tempsRestant -= dt;

    if (frame % 60 == 0) {
      print('t = ${(frame * dt).toStringAsFixed(1)} s | '
          'x = ${x.toStringAsFixed(1)} px | '
          'énergie = ${energie.toStringAsFixed(1)} | '
          'angle = ${angle.toStringAsFixed(2)} rad');
    }
  }

  print('---');
  print('Temps écoulé, le héros a parcouru ${x.toStringAsFixed(1)} px.');
}
```

**Résultat :**

```text
t = 1.0 s | x = 120.0 px | énergie = 45.0 | angle = 3.14 rad
t = 2.0 s | x = 240.0 px | énergie = 50.0 | angle = 6.28 rad
t = 3.0 s | x = 360.0 px | énergie = 55.0 | angle = 9.42 rad
---
Temps écoulé, le héros a parcouru 360.0 px.
```

Chaque nombre est vérifiable de tête : 120 px/s pendant 3 s font 360 px, 5 points/s pendant 3 s font 15 points ajoutés à 40, et une vitesse de π rad/s pendant 3 s fait 3π ≈ 9,42 rad. C'est là tout l'intérêt des unités physiques : **on peut vérifier le jeu au crayon**.

---

## 20.15 — Calculer dt à partir d'un `Stopwatch`

Jusqu'ici nous avons *supposé* `dt`. Il est temps de le **mesurer**.

Dart fournit la classe `Stopwatch` (chronomètre). Elle démarre, elle accumule, on lit son temps écoulé. Ses propriétés utiles :

| Membre | Rôle |
| --- | --- |
| `start()` | démarre ou reprend |
| `stop()` | met en pause |
| `reset()` | remet à zéro |
| `elapsedMilliseconds` | temps écoulé en ms (`int`) |
| `elapsedMicroseconds` | temps écoulé en microsecondes (`int`) |
| `elapsed` | temps écoulé sous forme de `Duration` |

Le principe de la mesure de `dt` :

```text
  MESURER dt AVEC UN STOPWATCH

  ┌──────────────────────────────────────────────────────────┐
  │  On garde en mémoire l'instant de la frame précédente.   │
  │  À chaque frame :                                        │
  │     maintenant  = chrono.elapsedMicroseconds             │
  │     dt          = (maintenant − precedent) / 1 000 000   │
  │     precedent   = maintenant                             │
  └──────────────────────────────────────────────────────────┘
```

Pourquoi les **microsecondes** et pas les millisecondes ? Parce que `elapsedMilliseconds` est un entier. À 120 FPS, une frame dure 8,33 ms ; l'entier vaudra tantôt 8, tantôt 9, ce qui introduit jusqu'à 8 % d'erreur. En microsecondes, la précision est mille fois meilleure.

```text
  PRÉCISION

  120 FPS, frame réelle = 8333 µs

  elapsedMilliseconds  ->  8 ms   (erreur jusqu'à 4 %)
  elapsedMicroseconds  ->  8333 µs (erreur négligeable)
```

Voici une petite classe `Horloge`, mise à l'épreuve dans une boucle console qui fait avancer le héros pendant une seconde.

```dart
import 'dart:async';

class Horloge {
  final Stopwatch _chrono = Stopwatch();
  int _precedentMicro = 0;

  void demarrer() {
    _chrono.start();
    _precedentMicro = _chrono.elapsedMicroseconds;
  }

  double tick() {
    final int maintenant = _chrono.elapsedMicroseconds;
    final int deltaMicro = maintenant - _precedentMicro;
    _precedentMicro = maintenant;
    return deltaMicro / 1000000.0;
  }

  double get tempsTotal => _chrono.elapsedMicroseconds / 1000000.0;
}

Future<void> main() async {
  final Horloge horloge = Horloge()..demarrer();

  double heroX = 0;
  const double vitesse = 200; // px/s
  int frames = 0;

  while (horloge.tempsTotal < 1.0) {
    final double dt = horloge.tick();

    // MISE À JOUR
    heroX += vitesse * dt;
    frames++;

    await Future<void>.delayed(const Duration(milliseconds: 16));
  }

  print('Frames    : $frames');
  print('Temps     : ${horloge.tempsTotal.toStringAsFixed(3)} s');
  print('Position  : ${heroX.toStringAsFixed(1)} px');
  print('Attendu   : ~${(vitesse * horloge.tempsTotal).toStringAsFixed(1)} px');
}
```

**Résultat (ordre de grandeur) :**

```text
Frames    : 59
Temps     : 1.004 s
Position  : 200.7 px
Attendu   : ~200.9 px
```

La position obtenue correspond bien à `vitesse × temps`, à quelques dixièmes près. Ces dixièmes viennent du fait que la dernière frame n'est pas comptée entièrement ; c'est normal et sans conséquence.

Relancez ce programme en changeant `16` en `33` (donc environ 30 FPS) : le nombre de frames tombera à une trentaine, mais la position finale restera autour de 200 pixels. **C'est exactement le comportement que l'on veut.**

---

## 20.16 — `Ticker` et `SchedulerBinding` dans Flutter

Nous savons mesurer `dt`. Il reste à trouver qui va nous rappeler à chaque image dans Flutter. Deux outils existent : le `Ticker` et le `SchedulerBinding`.

### Le `Ticker`

Un `Ticker` (littéralement « chose qui fait tic ») est un objet Flutter qui appelle une fonction **à chaque rafraîchissement de l'écran**. C'est la brique sur laquelle repose tout le système d'animation de Flutter.

```text
  QUI APPELLE QUI

  ┌──────────────┐   « nouvelle image ! »   ┌──────────────┐
  │  Le matériel │ ───────────────────────> │  Le moteur   │
  │  (écran)     │      60 fois/seconde     │  Flutter     │
  └──────────────┘                          └──────┬───────┘
                                                   │
                                     appelle votre │ callback
                                                   ▼
                                          ┌──────────────────┐
                                          │  votre fonction  │
                                          │  _onTick(Duration│
                                          │          elapsed)│
                                          └──────────────────┘
```

Points clés du `Ticker` :

- il est **synchronisé avec l'écran** (60 Hz, 90 Hz, 120 Hz selon l'appareil) ;
- il vous donne un `Duration elapsed` : **le temps total écoulé depuis le démarrage**, pas le `dt` ;
- il se met automatiquement en pause quand l'application passe en arrière-plan ;
- il **doit** être arrêté et libéré dans `dispose()`, sinon vous avez une fuite mémoire.

Pour créer un `Ticker`, il faut un `TickerProvider`. En pratique, on ajoute un mixin à l'état du widget (revoyez les mixins au chapitre 11) :

| Mixin | Quand l'utiliser |
| --- | --- |
| `SingleTickerProviderStateMixin` | un seul `Ticker` dans ce `State` |
| `TickerProviderStateMixin` | plusieurs `Ticker` dans ce `State` |

Le squelette minimal :

```dart
class _MonJeuState extends State<MonJeu>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick);
    _ticker.start();
  }

  void _onTick(Duration elapsed) {
    // elapsed = temps TOTAL depuis le start, pas le dt !
  }

  @override
  void dispose() {
    _ticker.dispose(); // OBLIGATOIRE
    super.dispose();
  }
}
```

> **Attention.** `elapsed` est cumulatif. Si vous écrivez `x += elapsed.inMilliseconds`, votre héros partira en accélération exponentielle. Il faut faire la **différence** avec l'`elapsed` précédent.

### Le `SchedulerBinding`

`SchedulerBinding.instance` est l'ordonnanceur de Flutter. Il propose deux méthodes utiles :

| Méthode | Rôle |
| --- | --- |
| `addPostFrameCallback(cb)` | appelle `cb` **une fois**, après la prochaine image |
| `scheduleFrameCallback(cb)` | demande une image et rappelle `cb` |

On peut construire une boucle avec `scheduleFrameCallback` en se re-planifiant soi-même :

```dart
void _boucle(Duration elapsed) {
  // ... travail ...
  SchedulerBinding.instance.scheduleFrameCallback(_boucle);
}
```

Cela fonctionne, mais le `Ticker` fait la même chose en gérant en plus la pause automatique et le cycle de vie. Le tableau de choix :

| Besoin | Outil |
| --- | --- |
| Boucle de jeu continue | `Ticker` |
| Faire une chose après le premier rendu (mesurer une taille, par exemple) | `addPostFrameCallback` |
| Boucle bas niveau, sans `State` | `scheduleFrameCallback` |
| Cadence fixe indépendante de l'écran (logique réseau, autosave) | `Timer.periodic` |

Pour toute la PARTIE 2A, nous utilisons le `Ticker`.

### Convertir `elapsed` en `dt`

C'est la seule subtilité. Voici le motif à mémoriser :

```dart
Duration _precedent = Duration.zero;

void _onTick(Duration elapsed) {
  final double dt =
      (elapsed - _precedent).inMicroseconds / 1000000.0;
  _precedent = elapsed;

  // ... utiliser dt ...
}
```

Notez encore `inMicroseconds` et la division par un million pour obtenir des **secondes**.

```text
  elapsed reçu   :  0 ms   16 ms   33 ms   49 ms   66 ms
  précédent      :  —       0       16      33      49
  différence     :  0       16      17      16      17
  dt (secondes)  :  0.000   0.016   0.017   0.016   0.017
```

La toute première frame donne un `dt` de zéro (ou très petit). C'est sans danger : rien ne bouge pendant une image.

---

## 20.17 — Implémenter une vraie boucle de jeu avec `Ticker`

Assemblons tout. Voici un `main.dart` complet et copiable : le héros du Donjon de Dart traverse l'écran, rebondit sur les bords, et tout est piloté par le delta time.

Créez un projet Flutter, remplacez `lib/main.dart` par ce fichier et lancez.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Donjon de Dart — Boucle de jeu',
      debugShowCheckedModeBanner: false,
      home: EcranJeu(),
    );
  }
}

class EcranJeu extends StatefulWidget {
  const EcranJeu({super.key});

  @override
  State<EcranJeu> createState() => _EcranJeuState();
}

class _EcranJeuState extends State<EcranJeu>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;

  // Instant de la frame précédente, tel que fourni par le Ticker.
  Duration _precedent = Duration.zero;

  // ÉTAT DU MONDE
  double _heroX = 40;
  double _heroY = 200;
  double _vitesseX = 180; // pixels par seconde
  double _tempsTotal = 0;

  static const double _tailleHero = 32;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick);
    _ticker.start();
  }

  void _onTick(Duration elapsed) {
    // 1. Calcul du delta time, EN SECONDES.
    final double dt =
        (elapsed - _precedent).inMicroseconds / 1000000.0;
    _precedent = elapsed;

    // 2. Mise à jour du monde, puis demande de redessin.
    setState(() {
      _update(dt);
    });
  }

  void _update(double dt) {
    _tempsTotal += dt;

    // Déplacement indépendant de la cadence.
    _heroX += _vitesseX * dt;

    // Rebond sur les bords.
    final double largeur = MediaQuery.of(context).size.width;
    if (_heroX < 0) {
      _heroX = 0;
      _vitesseX = -_vitesseX;
    } else if (_heroX + _tailleHero > largeur) {
      _heroX = largeur - _tailleHero;
      _vitesseX = -_vitesseX;
    }
  }

  @override
  void dispose() {
    _ticker.dispose(); // indispensable
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF14161C),
      body: Stack(
        children: [
          // Le héros.
          Positioned(
            left: _heroX,
            top: _heroY,
            child: Container(
              width: _tailleHero,
              height: _tailleHero,
              decoration: BoxDecoration(
                color: const Color(0xFFE8B04B),
                borderRadius: BorderRadius.circular(6),
              ),
            ),
          ),
          // Informations.
          Positioned(
            left: 16,
            top: 40,
            child: DefaultTextStyle(
              style: const TextStyle(
                color: Colors.white70,
                fontSize: 14,
                fontFamily: 'monospace',
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('temps : ${_tempsTotal.toStringAsFixed(2)} s'),
                  Text('x     : ${_heroX.toStringAsFixed(1)} px'),
                  Text('vx    : ${_vitesseX.toStringAsFixed(0)} px/s'),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :** un carré doré traverse l'écran de gauche à droite, rebondit sur le bord droit, revient, et ainsi de suite. Les trois lignes de texte affichent le temps écoulé, la position et la vitesse.

Passons en revue les points importants de ce code.

**`SingleTickerProviderStateMixin`.** Il fournit le `createTicker`. Sans lui, `createTicker` n'existe pas.

**`_precedent`.** C'est la mémoire nécessaire pour transformer `elapsed` (cumulatif) en `dt` (différentiel).

**`setState` autour de `_update`.** Il signale à Flutter que l'état a changé et qu'il faut reconstruire. Sans lui, le monde avancerait mais l'écran ne changerait jamais.

**Séparation `_update` / `build`.** `_update` ne dessine rien. `build` ne décide rien. C'est la règle de la section 20.3, appliquée.

**`dispose`.** Sans `_ticker.dispose()`, le ticker continue de tourner après la destruction du widget. Vous obtenez l'erreur « setState() called after dispose() » et une fuite mémoire.

> **Remarque sur `setState` à chaque frame.** C'est acceptable pour ce chapitre. Dès le chapitre 21, nous passerons à `CustomPainter` avec un `Listenable`, qui redessine sans reconstruire l'arbre de widgets, ce qui est bien plus efficace.

Pour vérifier que le delta time fait son travail, essayez ceci : lancez le jeu sur un émulateur limité à 30 FPS, puis sur un vrai téléphone à 120 Hz. Le carré met le même temps à traverser l'écran. Remplacez ensuite `_heroX += _vitesseX * dt;` par `_heroX += 3;` et refaites l'essai : la différence saute aux yeux.

---

## 20.18 — Afficher les FPS à l'écran

On ne règle pas ce qu'on ne mesure pas. Un compteur de FPS est le premier outil de tout développeur de jeu.

Le calcul le plus simple, image par image :

```dart
final double fps = 1 / dt;
```

À `dt = 0,0167`, cela donne 59,9 FPS. Attention toutefois : si `dt` vaut zéro (première frame), cette division donne l'infini. Il faut donc se protéger :

```dart
final double fps = dt > 0 ? 1 / dt : 0;
```

Voici un `main.dart` complet qui affiche à la fois le compteur instantané et quelques statistiques.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: EcranFps(),
    );
  }
}

class EcranFps extends StatefulWidget {
  const EcranFps({super.key});

  @override
  State<EcranFps> createState() => _EcranFpsState();
}

class _EcranFpsState extends State<EcranFps>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _dt = 0;
  double _fpsInstantane = 0;
  int _framesTotal = 0;
  double _tempsTotal = 0;
  double _dtMax = 0;

  double _heroX = 20;
  double _vx = 200;
  static const double _taille = 28;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick)..start();
  }

  void _onTick(Duration elapsed) {
    final double dt =
        (elapsed - _precedent).inMicroseconds / 1000000.0;
    _precedent = elapsed;

    setState(() {
      _dt = dt;
      _fpsInstantane = dt > 0 ? 1 / dt : 0;
      _framesTotal++;
      _tempsTotal += dt;
      if (dt > _dtMax) _dtMax = dt;

      _heroX += _vx * dt;
      final double largeur = MediaQuery.of(context).size.width;
      if (_heroX < 0 || _heroX + _taille > largeur) {
        _vx = -_vx;
        _heroX = _heroX.clamp(0.0, largeur - _taille);
      }
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final double fpsMoyen =
        _tempsTotal > 0 ? _framesTotal / _tempsTotal : 0;

    return Scaffold(
      backgroundColor: const Color(0xFF14161C),
      body: Stack(
        children: [
          Positioned(
            left: _heroX,
            top: 260,
            child: Container(
              width: _taille,
              height: _taille,
              color: const Color(0xFFE8B04B),
            ),
          ),
          Positioned(
            left: 16,
            top: 48,
            child: DefaultTextStyle(
              style: const TextStyle(
                color: Colors.greenAccent,
                fontSize: 15,
                fontFamily: 'monospace',
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('FPS   : ${_fpsInstantane.toStringAsFixed(1)}'),
                  Text('dt    : ${(_dt * 1000).toStringAsFixed(2)} ms'),
                  Text('moy.  : ${fpsMoyen.toStringAsFixed(1)} FPS'),
                  Text('pire  : ${(_dtMax * 1000).toStringAsFixed(1)} ms'),
                  Text('frames: $_framesTotal'),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat (à l'écran) :**

```text
FPS   : 59.4
dt    : 16.83 ms
moy.  : 59.8 FPS
pire  : 41.2 ms
frames: 1043
```

Vous verrez immédiatement un défaut : le nombre affiché à la ligne `FPS` **danse en permanence**. Il passe de 58,1 à 61,7 puis à 59,3, dix fois par seconde. Il est illisible.

```text
  FPS INSTANTANÉS, VALEURS SUCCESSIVES

  61.2  58.4  60.1  57.9  62.3  59.0  55.4  61.8  60.2  58.7
   ↑     ↓     ↑     ↓     ↑     ↓     ↓     ↑     ↑     ↓

  -> impossible à lire, et pourtant tout va bien
```

Ces variations sont normales : elles reflètent les micro-irrégularités de l'ordonnanceur. Ce ne sont pas des problèmes de performance. Pour obtenir un nombre lisible, il faut **lisser**.

Notez au passage la ligne « pire », qui affiche le pire `dt` rencontré. Elle est souvent plus utile que la moyenne : un jeu à 60 FPS de moyenne mais avec des pointes à 120 ms donne une sensation de saccade que la moyenne ne montre pas.

---

## 20.19 — Lisser les FPS (moyenne glissante)

Lisser signifie remplacer la dernière valeur par une **moyenne des dernières valeurs**. Deux techniques sont courantes.

### Technique 1 : la moyenne sur une fenêtre glissante

On garde les N derniers `dt` dans une liste, et on fait la moyenne.

```text
  FENÊTRE GLISSANTE DE 5 VALEURS

  frames   ... F6    F7    F8    F9    F10   F11 -->
  dt (ms)      16.2  17.0  15.8  16.4  33.1  16.1

  fenêtre au moment de F10 :
      [17.0, 15.8, 16.4, 33.1]  + la nouvelle 16.1
      on jette la plus ancienne si on dépasse 5
      moyenne = (17.0+15.8+16.4+33.1+16.1)/5 = 19.68 ms
      FPS lissés = 1000 / 19.68 = 50.8
```

En Dart, une file (`Queue`) ou une simple `List` suffit.

```dart
import 'dart:collection';

/// Calcule une moyenne glissante des delta times.
class CompteurFps {
  CompteurFps({this.taille = 60});

  final int taille;
  final Queue<double> _dts = Queue<double>();
  double _somme = 0;

  void ajouter(double dt) {
    if (dt <= 0) return;
    _dts.addLast(dt);
    _somme += dt;
    while (_dts.length > taille) {
      _somme -= _dts.removeFirst();
    }
  }

  /// Durée moyenne d'une image, en secondes.
  double get dtMoyen => _dts.isEmpty ? 0 : _somme / _dts.length;

  /// FPS lissés.
  double get fps => dtMoyen > 0 ? 1 / dtMoyen : 0;

  void vider() {
    _dts.clear();
    _somme = 0;
  }
}

void main() {
  final CompteurFps compteur = CompteurFps(taille: 5);

  // On simule des frames irrégulières (en secondes).
  const List<double> mesures = [
    0.0162, 0.0170, 0.0158, 0.0164, 0.0331, 0.0161, 0.0166,
  ];

  for (final double dt in mesures) {
    compteur.ajouter(dt);
    print('dt = ${(dt * 1000).toStringAsFixed(1)} ms  ->  '
        'FPS bruts = ${(1 / dt).toStringAsFixed(1)}  |  '
        'FPS lissés = ${compteur.fps.toStringAsFixed(1)}');
  }
}
```

**Résultat :**

```text
dt = 16.2 ms  ->  FPS bruts = 61.7  |  FPS lissés = 61.7
dt = 17.0 ms  ->  FPS bruts = 58.8  |  FPS lissés = 60.2
dt = 15.8 ms  ->  FPS bruts = 63.3  |  FPS lissés = 61.2
dt = 16.4 ms  ->  FPS bruts = 61.0  |  FPS lissés = 61.2
dt = 33.1 ms  ->  FPS bruts = 30.2  |  FPS lissés = 50.8
dt = 16.1 ms  ->  FPS bruts = 62.1  |  FPS lissés = 50.9
dt = 16.6 ms  ->  FPS bruts = 60.2  |  FPS lissés = 51.2
```

Observez la ligne du pic : les FPS bruts s'effondrent brutalement à 30,2, alors que les FPS lissés descendent doucement à 50,8. C'est exactement l'effet recherché : on voit le problème sans que l'affichage devienne illisible.

> **Attention à un piège classique.** Il faut faire la moyenne des **`dt`**, puis convertir en FPS. Faire la moyenne des **FPS** donne un résultat mathématiquement faux (c'est une moyenne harmonique qu'il faudrait). Avec deux frames de 10 ms et 30 ms : la bonne réponse est `1/0,020 = 50` FPS ; la moyenne des FPS donnerait `(100+33,3)/2 = 66,7`, ce qui est trop optimiste.

### Technique 2 : le lissage exponentiel

Plus léger : on ne garde aucune liste, seulement la valeur courante.

```dart
dtLisse = dtLisse * (1 - a) + dt * a;
```

Le coefficient `a` (entre 0 et 1) règle la réactivité :

| `a` | Comportement |
| --- | --- |
| 0.01 | très lisse, très lent à réagir |
| 0.1 | bon compromis pour un compteur de FPS |
| 0.5 | réactif, encore un peu nerveux |
| 1.0 | aucun lissage |

```dart
void main() {
  const double a = 0.1;
  double dtLisse = 0.0166;

  const List<double> mesures = [
    0.0162, 0.0170, 0.0158, 0.0331, 0.0161, 0.0166, 0.0163,
  ];

  for (final double dt in mesures) {
    dtLisse = dtLisse * (1 - a) + dt * a;
    print('dt = ${(dt * 1000).toStringAsFixed(1)} ms  ->  '
        'FPS lissés = ${(1 / dtLisse).toStringAsFixed(1)}');
  }
}
```

**Résultat :**

```text
dt = 16.2 ms  ->  FPS lissés = 60.4
dt = 17.0 ms  ->  FPS lissés = 60.2
dt = 15.8 ms  ->  FPS lissés = 60.4
dt = 33.1 ms  ->  FPS lissés = 55.5
dt = 16.1 ms  ->  FPS lissés = 55.9
dt = 16.6 ms  ->  FPS lissés = 56.2
dt = 16.3 ms  ->  FPS lissés = 56.6
```

Le pic est absorbé, puis le compteur remonte progressivement. Choisissez la technique 1 si vous voulez aussi les valeurs minimum et maximum ; la technique 2 si vous voulez le code le plus court.

> **Règle de présentation.** Affichez les FPS lissés **et** le pire `dt` de la dernière seconde. Le premier rassure, le second dit la vérité.

---

## 20.20 — Le pic de lag et le dt aberrant

Un jour, votre héros va traverser un mur. Vous relancerez, et ce sera impossible à reproduire. Voici l'explication.

Un **pic de lag** est une image qui prend anormalement longtemps. Causes fréquentes :

| Cause | Ordre de grandeur |
| --- | --- |
| Chargement d'une image depuis le disque | 20 à 200 ms |
| Passage du ramasse-miettes (garbage collector) | 5 à 50 ms |
| Une autre application qui démarre | 50 à 500 ms |
| Compilation JIT au premier passage d'un code | 10 à 100 ms |
| Application mise en arrière-plan | **plusieurs secondes** |
| Point d'arrêt dans le débogueur | **plusieurs minutes** |

Le dernier cas est le plus violent, et le plus fréquent en développement. Regardez ce qui se passe.

```text
  L'UTILISATEUR REÇOIT UN APPEL, PUIS REVIENT 30 SECONDES PLUS TARD

  frames    F1    F2    F3                              F4
            ●─────●─────●───────────────────────────────●
            16ms  16ms      30 000 ms (30 secondes !)

  dt de F4 = 30.0 secondes

  Le code fait :  x += 200 * 30.0  =  x += 6000 pixels
                  y += gravite * 30.0 -> le héros est à 400 000 px sous le sol
                  collisions -> le héros a traversé tout le niveau
                  vie -= degats * 30 -> mort instantanée
```

Un `dt` de 30 secondes appliqué d'un coup détruit la cohérence du monde. Les symptômes typiques :

- le héros se téléporte hors du niveau ;
- il traverse les murs (le déplacement d'une frame dépasse l'épaisseur du mur) ;
- toutes les barres de vie se vident ;
- les valeurs deviennent `NaN` ou `Infinity` et plus rien ne s'affiche ;
- au retour d'une pause, le jeu « rattrape » brutalement 30 secondes de simulation.

Le `Ticker` de Flutter se met en pause en arrière-plan, ce qui limite le problème, mais pas dans tous les cas ; et le débogueur, lui, ne prévient personne.

Un exemple minimal qui montre les dégâts :

```dart
void main() {
  double x = 100;
  double y = 300;
  double vy = 0;
  const double vitesse = 200;   // px/s
  const double gravite = 1200;  // px/s²

  // Suite de dt réalistes... avec une aberration au milieu.
  const List<double> dts = [0.016, 0.017, 0.016, 12.5, 0.016, 0.017];

  for (final double dt in dts) {
    x += vitesse * dt;
    vy += gravite * dt;
    y += vy * dt;
    print('dt = ${dt.toStringAsFixed(3)} s  ->  '
        'x = ${x.toStringAsFixed(1)}  y = ${y.toStringAsFixed(1)}');
  }
}
```

**Résultat :**

```text
dt = 0.016 s  ->  x = 103.2  y = 300.3
dt = 0.017 s  ->  x = 106.6  y = 301.0
dt = 0.016 s  ->  x = 109.8  y = 301.8
dt = 12.500 s  ->  x = 2609.8  y = 189739.3
dt = 0.016 s  ->  x = 2613.0  y = 190982.6
dt = 0.017 s  ->  x = 2616.4  y = 192304.1
```

Une seule frame aberrante, et le héros passe de `y = 302` à `y = 189 739`. Il ne reviendra jamais. Aucun mur, aucune plateforme, aucun test de collision ne l'a arrêté : il est passé « au travers » de tout le niveau en un seul pas.

La parade tient en une ligne. C'est la section suivante.

---

## 20.21 — Plafonner le dt (`dt.clamp`)

La solution universelle : **on refuse tout `dt` supérieur à une limite**.

```dart
dt = dt.clamp(0.0, 0.05);
```

`clamp(min, max)` est une méthode de `num` en Dart : elle renvoie la valeur si elle est dans l'intervalle, sinon la borne la plus proche.

```text
  EFFET DE dt.clamp(0.0, 0.05)

  dt mesuré      dt utilisé     commentaire
  ─────────────────────────────────────────────────────
  0.0000    ->   0.0000         première frame
  0.0166    ->   0.0166         60 FPS, inchangé
  0.0333    ->   0.0333         30 FPS, inchangé
  0.0500    ->   0.0500         20 FPS, à la limite
  0.1200    ->   0.0500         PLAFONNÉ
  12.500    ->   0.0500         PLAFONNÉ (retour d'arrière-plan)
```

Quelle valeur de plafond choisir ?

| Plafond | Équivaut à | Usage |
| --- | --- | --- |
| 0.0333 s | 30 FPS minimum simulés | jeux d'action rapides, très sûr |
| 0.05 s | 20 FPS minimum simulés | **valeur par défaut recommandée** |
| 0.1 s | 10 FPS minimum simulés | jeux lents, tolérant |
| 0.25 s | 4 FPS | trop permissif |

Que se passe-t-il quand on plafonne ? Le monde avance **au ralenti** pendant les pics. Une image qui a réellement pris 200 ms ne fait avancer le monde que de 50 ms. Le jeu prend donc du retard sur l'horloge murale.

C'est un compromis assumé :

> Mieux vaut un jeu **qui ralentit** qu'un jeu **qui explose**.

Aucun joueur ne remarque que le jeu a « perdu » 150 millisecondes. Tout le monde remarque un héros téléporté hors du niveau.

Reprenons l'exemple catastrophique de la section précédente, avec le plafond.

```dart
void main() {
  double x = 100;
  double y = 300;
  double vy = 0;
  const double vitesse = 200;
  const double gravite = 1200;
  const double dtMax = 0.05;

  const List<double> dtsMesures = [0.016, 0.017, 0.016, 12.5, 0.016, 0.017];

  for (final double brut in dtsMesures) {
    final double dt = brut.clamp(0.0, dtMax);

    x += vitesse * dt;
    vy += gravite * dt;
    y += vy * dt;

    final String note = brut > dtMax ? '  <-- PLAFONNÉ' : '';
    print('brut = ${brut.toStringAsFixed(3)}  '
        'utilisé = ${dt.toStringAsFixed(3)}  ->  '
        'x = ${x.toStringAsFixed(1)}  y = ${y.toStringAsFixed(1)}$note');
  }
}
```

**Résultat :**

```text
brut = 0.016  utilisé = 0.016  ->  x = 103.2  y = 300.3
brut = 0.017  utilisé = 0.017  ->  x = 106.6  y = 301.0
brut = 0.016  utilisé = 0.016  ->  x = 109.8  y = 301.8
brut = 12.500  utilisé = 0.050  ->  x = 119.8  y = 305.5  <-- PLAFONNÉ
brut = 0.016  utilisé = 0.016  ->  x = 123.0  y = 306.9
brut = 0.017  utilisé = 0.017  ->  x = 126.4  y = 308.5
```

Le héros est resté dans le niveau. Le pic a été absorbé sans dégât.

Voici la fonction de calcul du `dt` que vous devez utiliser désormais, systématiquement :

```dart
/// Convertit l'elapsed cumulatif d'un Ticker en delta time sûr.
double calculerDt(Duration elapsed, Duration precedent,
    {double dtMax = 0.05}) {
  final double brut =
      (elapsed - precedent).inMicroseconds / 1000000.0;
  return brut.clamp(0.0, dtMax);
}
```

> **Attention à un détail Dart.** Sur un `double`, `clamp` renvoie un `num` dans certaines versions d'analyse. Si l'analyseur se plaint, écrivez `brut.clamp(0.0, dtMax).toDouble()`. Veillez aussi à passer des bornes `double` (`0.0` et non `0`), sinon le type de retour se dégrade en `num`.

---

## 20.22 — Le pas de temps fixe (fixed timestep)

Le plafonnement règle les catastrophes. Il ne règle pas un problème plus subtil : **la simulation n'est pas reproductible**.

Avec un `dt` variable, deux exécutions du même jeu, avec les mêmes touches appuyées aux mêmes instants, ne donnent pas exactement le même résultat, parce que les `dt` ne sont jamais identiques.

Un exemple frappant : la hauteur d'un saut.

```dart
/// Simule un saut avec un dt donné et renvoie la hauteur maximale.
double hauteurSaut(double dt) {
  double y = 0;
  double vy = 500;              // impulsion, px/s
  const double gravite = 1200;  // px/s²
  double hauteurMax = 0;

  while (y >= 0) {
    vy -= gravite * dt;
    y += vy * dt;
    if (y > hauteurMax) hauteurMax = y;
  }

  return hauteurMax;
}

void main() {
  print('Hauteur de saut selon le pas de temps');
  print('');
  for (final double dt in [1 / 30, 1 / 60, 1 / 120, 1 / 240]) {
    print('dt = ${(dt * 1000).toStringAsFixed(2)} ms  ->  '
        'hauteur = ${hauteurSaut(dt).toStringAsFixed(2)} px');
  }
}
```

**Résultat :**

```text
Hauteur de saut selon le pas de temps

dt = 33.33 ms  ->  hauteur = 100.00 px
dt = 16.67 ms  ->  hauteur = 102.78 px
dt = 8.33 ms  ->  hauteur = 103.99 px
dt = 4.17 ms  ->  hauteur = 104.60 px
```

Les hauteurs ne sont pas identiques : 100,00 puis 102,78 puis 103,99. L'écart est petit (moins de 5 %), mais il est réel, et il vient de la méthode d'intégration numérique : additionner `vitesse × dt` est une **approximation** de l'intégrale, et l'approximation dépend du pas.

Dans un jeu de plateforme précis, 4 pixels de saut décident si le héros atteint une plateforme ou tombe dans le vide. Le même niveau devient donc franchissable sur un téléphone à 120 Hz et infranchissable sur une tablette à 30 Hz.

### La solution : le pas fixe

L'idée est de **découpler le temps de la simulation du temps du rendu** :

> Le rendu se produit quand l'écran le demande, à cadence variable.
> La simulation avance **toujours par pas de durée identique**, par exemple 1/60 de seconde.

```text
  PAS VARIABLE (ce que nous faisions)

  frames  ●───────●─────●──────────●────●
  dt      16ms    12ms  20ms       31ms  9ms
  update  update(0.016) update(0.012) update(0.020) ...
          -> chaque update est différent


  PAS FIXE (ce que nous allons faire)

  frames  ●───────●─────●──────────●────●
  dt      16ms    12ms  20ms       31ms  9ms
  update  U       U     U U        U U   (rien)
          toutes les update reçoivent EXACTEMENT 1/60 s
```

Sur une frame longue, on exécute plusieurs `update` d'affilée. Sur une frame courte, on n'en exécute aucun. Le nombre d'`update` varie, mais **leur durée ne varie jamais**.

Les bénéfices sont immédiats :

| Bénéfice | Explication |
| --- | --- |
| Reproductibilité | même entrée, même résultat, sur toutes les machines |
| Physique stable | plus de saut qui change de hauteur |
| Collisions fiables | le pas maximal est connu à l'avance |
| Réseau et rejeu possibles | on peut numéroter les pas |
| Débogage plus simple | on peut avancer « pas à pas » |

Le mécanisme qui rend cela possible s'appelle l'**accumulateur**.

---

## 20.23 — L'accumulateur

L'accumulateur est une variable qui stocke le temps « pas encore simulé ».

```text
  ┌─────────────────────────────────────────────────────────────┐
  │                   L'ACCUMULATEUR                            │
  └─────────────────────────────────────────────────────────────┘

  À chaque frame :

     1. accumulateur += dt réel

     2. tant que accumulateur >= pasFixe :
             update(pasFixe)
             accumulateur -= pasFixe

     3. render()   (le reste dort dans l'accumulateur)
```

Déroulons un exemple concret, avec `pasFixe = 16 ms`.

```text
  DÉROULÉ SUR 6 FRAMES  (pas fixe = 16 ms)

  frame  dt réel   accu avant   updates   accu après
  ────────────────────────────────────────────────────
   F1     17 ms      0 -> 17        1        1 ms
   F2     16 ms      1 -> 17        1        1 ms
   F3     15 ms      1 -> 16        1        0 ms
   F4     34 ms      0 -> 34        2        2 ms   <- frame lente
   F5      8 ms      2 -> 10        0       10 ms   <- frame rapide
   F6      9 ms     10 -> 19        1        3 ms

  Total : 99 ms écoulés, 6 updates exécutés  (6 x 16 = 96 ms simulés)
          il reste 3 ms dans l'accumulateur, ils seront simulés plus tard.
```

Rien n'est perdu. Le temps est simplement **mis en réserve** jusqu'à ce qu'il y en ait assez pour un pas complet.

Vue en schéma :

```text
  L'ACCUMULATEUR COMME UN RÉSERVOIR

              dt réel entre
                  │
                  ▼
        ┌───────────────────┐
        │   ACCUMULATEUR    │
        │   ▓▓▓▓▓▓▓░░░░░░   │   niveau = temps en attente
        └─────────┬─────────┘
                  │
                  │  tant qu'il y a >= 16 ms,
                  │  on prélève 16 ms
                  ▼
        ┌───────────────────┐
        │  update(0.0166)   │  <- toujours la même valeur
        └───────────────────┘
```

Voici l'implémentation, en console, pour bien voir les nombres.

```dart
void main() {
  const double pasFixe = 1 / 60; // 16,666 ms
  double accumulateur = 0;
  int updatesTotal = 0;

  double x = 0;
  const double vitesse = 300; // px/s

  // Une série de dt réels irréguliers (en secondes).
  const List<double> dts = [
    0.017, 0.016, 0.015, 0.034, 0.008, 0.009, 0.050, 0.016,
  ];

  print('pas  dt réel   updates   accu restant   x');
  print('------------------------------------------------');

  int frame = 0;
  for (final double dt in dts) {
    frame++;
    accumulateur += dt;

    int updatesCetteFrame = 0;
    while (accumulateur >= pasFixe) {
      // MISE À JOUR à pas constant
      x += vitesse * pasFixe;
      accumulateur -= pasFixe;
      updatesCetteFrame++;
      updatesTotal++;
    }

    print('${frame.toString().padLeft(3)}  '
        '${(dt * 1000).toStringAsFixed(0).padLeft(6)} ms  '
        '${updatesCetteFrame.toString().padLeft(7)}  '
        '${(accumulateur * 1000).toStringAsFixed(1).padLeft(11)} ms  '
        '${x.toStringAsFixed(1).padLeft(7)}');
  }

  final double tempsReel = dts.reduce((a, b) => a + b);
  print('------------------------------------------------');
  print('Temps réel écoulé : ${(tempsReel * 1000).toStringAsFixed(1)} ms');
  print('Temps simulé      : '
      '${(updatesTotal * pasFixe * 1000).toStringAsFixed(1)} ms');
  print('En attente        : '
      '${(accumulateur * 1000).toStringAsFixed(1)} ms');
}
```

**Résultat :**

```text
pas  dt réel   updates   accu restant   x
------------------------------------------------
  1      17 ms        1          0.3 ms      5.0
  2      16 ms        1          0.0 ms     10.0
  3      15 ms        0         15.0 ms     10.0
  4      34 ms        2         15.7 ms     20.0
  5       8 ms        1          7.0 ms     25.0
  6       9 ms        0         16.0 ms     25.0
  7      50 ms        3         16.0 ms     40.0
  8      16 ms        1         15.4 ms     45.0
------------------------------------------------
Temps réel écoulé : 165.0 ms
Temps simulé      : 150.0 ms
En attente        : 15.4 ms
```

Vérifions la cohérence : 150 ms simulés + 15,4 ms en attente = 165,4 ms, soit le temps réel à l'arrondi près. **Aucun temps n'est perdu.**

Observez aussi la ligne 6 : `0 update`. Cette frame a été dessinée sans que le monde n'avance. Le joueur voit donc deux images identiques. C'est le défaut du pas fixe, appelé *stuttering* (micro-saccade), et nous verrons son remède en 20.26 avec l'interpolation.

### La spirale de la mort

Il reste un danger à connaître. Si chaque `update` prend plus de temps que `pasFixe`, l'accumulateur grossit sans jamais se vider :

```text
  SPIRALE DE LA MORT

  frame 1 : accu = 20 ms -> 1 update (qui prend 25 ms de calcul)
  frame 2 : accu = 20 + 25 = 45 ms -> 2 updates (50 ms de calcul)
  frame 3 : accu = 20 + 50 = 70 ms -> 4 updates (100 ms de calcul)
  frame 4 : accu = ... -> 8 updates ...
  -> le jeu se fige définitivement
```

La parade est simple : **limiter le nombre d'`update` par frame**.

```dart
const int maxUpdatesParFrame = 5;

int compte = 0;
while (accumulateur >= pasFixe && compte < maxUpdatesParFrame) {
  update(pasFixe);
  accumulateur -= pasFixe;
  compte++;
}
// Sécurité : si on est sorti par la limite, on jette le surplus.
if (accumulateur > pasFixe * maxUpdatesParFrame) {
  accumulateur = 0;
}
```

Le jeu ralentira (le temps simulé prendra du retard sur le temps réel), mais il ne se figera pas. Là encore : **mieux vaut ralentir qu'exploser**.

---

## 20.24 — Pas fixe vs pas variable : avantages et inconvénients

Les deux approches coexistent dans l'industrie. Voici de quoi choisir en connaissance de cause.

| Critère | Pas variable (`update(dt)`) | Pas fixe (accumulateur) |
| --- | --- | --- |
| Simplicité du code | très simple | plus complexe |
| Reproductibilité | non | **oui** |
| Physique stable | approximative | **exacte à chaque exécution** |
| Hauteur de saut identique partout | non | **oui** |
| Collisions fiables à haute vitesse | risque de tunneling | risque maîtrisé |
| Rejeu (replay) et réseau | quasi impossible | **possible** |
| Fluidité visuelle sans interpolation | **parfaite** | micro-saccades |
| Coût processeur | 1 update par frame | 0 à N updates par frame |
| Risque de spirale de la mort | non | oui, à protéger |
| Débogage pas à pas | difficile | **facile** |
| Utilisé par | Flame par défaut, jeux casuals | Unity `FixedUpdate`, Box2D, jeux de plateforme précis |

En pratique, la recommandation professionnelle est **l'approche hybride** :

```text
  RÉPARTITION RECOMMANDÉE

  PAS FIXE  ────────────────────────────────
    physique (gravité, vitesses, forces)
    collisions
    logique de jeu déterministe (IA, dégâts)

  PAS VARIABLE  ────────────────────────────
    animations de sprite
    effets visuels, particules
    interface, fondus, transitions
    caméra (souvent lissée sur dt réel)
```

Ce que nous ferons dans cette formation :

| Chapitre | Approche |
| --- | --- |
| 20 à 22 | pas variable, avec `dt` plafonné |
| 23 (physique) et 24 (collisions) | pas fixe présenté et utilisé |
| 27 et suivants (Flame) | `update(dt)` de Flame, donc pas variable |
| 39 (boss, précision) | pas fixe pour la physique du boss |

> **À retenir.** Le pas variable suffit pour 90 % des jeux 2D. Le pas fixe devient nécessaire dès que la précision de la physique décide de la jouabilité, ou dès qu'il faut rejouer une partie à l'identique.

---

## 20.25 — Découpler mise à jour et rendu

Le pas fixe nous a déjà obligés à séparer `update` et `render`. Formalisons cette idée, car elle structure toute l'architecture d'un jeu (chapitre 26).

```text
  DEUX HORLOGES INDÉPENDANTES

  HORLOGE DE SIMULATION           HORLOGE DE RENDU
  (le monde avance)               (l'écran s'actualise)

  cadence : fixe, 60 Hz           cadence : celle de l'écran
  pilotée par : l'accumulateur    pilotée par : le Ticker
  produit : un état du monde      produit : des pixels
  déterministe : oui              déterministe : non

           │                                │
           └──────────► le rendu LIT ◄──────┘
                        l'état de la simulation
                        sans jamais le modifier
```

Les règles à respecter, sans exception :

**Règle 1 — `render` ne modifie rien.** Aucune affectation sur l'état du monde dans le code de dessin. Si vous écrivez `x += ...` dans un `CustomPainter`, la vitesse du héros dépendra du nombre de redessins, et vous retrouverez le bug de la section 20.11 par la porte de derrière.

**Règle 2 — `update` ne dessine rien.** Pas de `print`, pas de création de `Paint`, pas de `TextPainter` dans la mise à jour.

**Règle 3 — Aucun calcul lourd dans `render`.** Pas de tri de liste, pas de recherche de chemin, pas de lecture de fichier. Le rendu doit se contenter de parcourir et d'afficher.

Voici le tableau de répartition à garder sous les yeux.

| Tâche | `update` | `render` |
| --- | --- | --- |
| Déplacer le héros | oui | non |
| Appliquer la gravité | oui | non |
| Détecter une collision | oui | non |
| Retirer une vie | oui | non |
| Faire avancer l'IA d'un gobelin | oui | non |
| Choisir l'image d'animation courante | oui | non |
| Dessiner le décor | non | oui |
| Dessiner les sprites | non | oui |
| Dessiner la barre de vie | non | oui |
| Écrire le score à l'écran | non | oui |

Un mauvais exemple, qu'il faut savoir reconnaître :

```dart
// MAUVAIS : la logique est dans le rendu.
@override
void paint(Canvas canvas, Size size) {
  heroX += 3; // NON : le monde avance pendant qu'on le dessine
  if (heroX > size.width) {
    vies--;   // NON : une règle de jeu dans le rendu
  }
  canvas.drawRect(Rect.fromLTWH(heroX, 100, 32, 32), _paint);
}
```

La version correcte :

```dart
// BON : le rendu ne fait que lire.
@override
void paint(Canvas canvas, Size size) {
  canvas.drawRect(Rect.fromLTWH(monde.heroX, 100, 32, 32), _paint);
}
```

et, séparément :

```dart
void update(double dt) {
  monde.heroX += monde.vitesseX * dt;
  if (monde.heroX > monde.largeur) {
    monde.vies--;
    monde.heroX = 0;
  }
}
```

> **Test mental infaillible.** Si vous appeliez `render` deux fois de suite sans appeler `update` entre les deux, le jeu devrait être **strictement inchangé**. Si ce n'est pas le cas, votre rendu contient de la logique.

---

## 20.26 — L'interpolation entre deux états

Reprenons le défaut du pas fixe vu en 20.23 : certaines frames ne déclenchent aucun `update`, donc l'écran affiche deux fois la même image, puis fait un « double pas ». Cela produit un léger tremblement.

```text
  SANS INTERPOLATION (pas fixe 60 Hz, écran 100 Hz)

  frame   1     2     3     4     5     6     7     8
  update  U     U     -     U     U     -     U     U
  x       0    16    16    32    48    48    64    80
                     ▲                 ▲
                     └── répétition ───┘  -> saccade visible
```

Le remède est l'**interpolation**. Puisqu'il reste du temps dans l'accumulateur, on sait où en est le monde « entre deux pas ». On calcule un facteur `alpha` :

```text
  alpha = accumulateur / pasFixe        (valeur entre 0 et 1)

  position affichée = positionPrecedente * (1 - alpha)
                    + positionActuelle   * alpha
```

Visuellement :

```text
  INTERPOLATION ENTRE DEUX ÉTATS

  état précédent            état actuel
  (pas N-1)                 (pas N)
     x = 100                   x = 116
       ●───────────────────────────●
              ▲
              │  alpha = 0,4
              │
       position affichée = 100*0,6 + 116*0,4 = 106,4
```

Le monde n'est simulé qu'aux positions 100 et 116, mais on **affiche** 106,4. L'œil voit un mouvement parfaitement lisse.

Implémentation type :

```dart
class EtatHero {
  double x;
  double y;
  EtatHero(this.x, this.y);
}

class Monde {
  EtatHero precedent = EtatHero(0, 200);
  EtatHero actuel = EtatHero(0, 200);
  double vitesse = 300;

  void update(double pasFixe) {
    // On archive l'état avant de le faire avancer.
    precedent = EtatHero(actuel.x, actuel.y);
    actuel.x += vitesse * pasFixe;
  }

  /// Position à afficher, interpolée avec [alpha] entre 0 et 1.
  double xAffiche(double alpha) =>
      precedent.x * (1 - alpha) + actuel.x * alpha;
}

void main() {
  final Monde monde = Monde();
  const double pasFixe = 1 / 60;
  double accumulateur = 0;

  // dt réels d'un écran à 100 Hz (10 ms par frame).
  for (int frame = 1; frame <= 6; frame++) {
    const double dt = 0.010;
    accumulateur += dt;

    int updates = 0;
    while (accumulateur >= pasFixe) {
      monde.update(pasFixe);
      accumulateur -= pasFixe;
      updates++;
    }

    final double alpha = accumulateur / pasFixe;
    print('frame $frame | updates=$updates | '
        'alpha=${alpha.toStringAsFixed(2)} | '
        'x simulé=${monde.actuel.x.toStringAsFixed(1)} | '
        'x affiché=${monde.xAffiche(alpha).toStringAsFixed(1)}');
  }
}
```

**Résultat :**

```text
frame 1 | updates=0 | alpha=0.60 | x simulé=0.0 | x affiché=0.0
frame 2 | updates=1 | alpha=0.20 | x simulé=5.0 | x affiché=1.0
frame 3 | updates=1 | alpha=0.80 | x simulé=10.0 | x affiché=9.0
frame 4 | updates=0 | alpha=0.40 | x simulé=10.0 | x affiché=7.0
frame 5 | updates=1 | alpha=1.00 | x simulé=15.0 | x affiché=15.0
frame 6 | updates=1 | alpha=0.60 | x simulé=20.0 | x affiché=18.0
```

Les positions affichées progressent de façon nettement plus régulière que les positions simulées, qui restent bloquées deux frames de suite.

> **Nuance honnête.** L'interpolation ajoute une latence d'un pas (on affiche une position légèrement en retard). Certains jeux compétitifs préfèrent extrapoler, c'est-à-dire prédire un peu en avance. Pour un jeu 2D d'apprentissage, l'interpolation simple est le bon choix, et Flame ne vous demandera même pas d'y penser.

---

## 20.27 — La pause : mettre dt à zéro

Comment met-on un jeu en pause ? La réponse est élégante :

> Un jeu en pause est un jeu dont le `dt` vaut zéro.

Puisque tout est écrit `x += vitesse * dt`, poser `dt = 0` fige **tout** d'un seul coup : les positions, la gravité, les rechargements, l'IA, le chronomètre. Sans écrire une seule condition dans le code des entités.

```dart
void _onTick(Duration elapsed) {
  double dt = (elapsed - _precedent).inMicroseconds / 1000000.0;
  _precedent = elapsed;
  dt = dt.clamp(0.0, 0.05);

  if (_enPause) {
    dt = 0; // le monde s'arrête
  }

  setState(() => _update(dt));
}
```

Trois façons de faire, avec leurs conséquences :

| Méthode | Le rendu continue ? | Les animations d'interface ? | Consommation |
| --- | --- | --- | --- |
| `dt = 0` | oui | oui (elles n'utilisent pas ce `dt`) | normale |
| `return` avant `update` | oui | oui | légèrement réduite |
| `_ticker.stop()` | non | non | **minimale** |

Le plus souvent, on veut `dt = 0` : l'écran continue d'être dessiné, ce qui permet d'afficher un menu de pause par-dessus le jeu figé, avec des boutons animés.

`_ticker.stop()` est réservé aux cas où le jeu est réellement caché : application en arrière-plan, écran de chargement, autre page ouverte. Il économise la batterie.

> **Attention au piège du redémarrage.** Si vous utilisez `_ticker.stop()` puis `_ticker.start()`, le `elapsed` du `Ticker` **reprend là où il s'était arrêté** : il ne compte pas le temps de la pause. C'est un bon comportement. En revanche, si vous mesurez le temps avec votre propre `Stopwatch`, pensez à l'arrêter aussi, sinon la première frame après la pause aura un `dt` énorme. Le plafonnement de la section 20.21 vous sauve dans tous les cas.

Vérifions le mécanisme en console, sans interface, pour bien voir les nombres. Nous simulons une partie de six frames dont deux en pause.

```dart
void main() {
  double heroX = 0;
  double tempsDeJeu = 0;
  double tempsReel = 0;
  const double vitesse = 200; // px/s

  // (dt réel, en pause ?)
  const List<List<Object>> frames = [
    [0.016, false],
    [0.016, false],
    [0.016, true],
    [0.016, true],
    [0.016, false],
    [0.016, false],
  ];

  for (int i = 0; i < frames.length; i++) {
    final double dtReel = frames[i][0] as double;
    final bool pause = frames[i][1] as bool;

    tempsReel += dtReel;
    final double dt = pause ? 0.0 : dtReel;

    tempsDeJeu += dt;
    heroX += vitesse * dt;

    print('frame ${i + 1} | ${pause ? 'PAUSE ' : 'jeu   '} | '
        'réel = ${tempsReel.toStringAsFixed(3)} s | '
        'jeu = ${tempsDeJeu.toStringAsFixed(3)} s | '
        'x = ${heroX.toStringAsFixed(2)}');
  }
}
```

**Résultat :**

```text
frame 1 | jeu    | réel = 0.016 s | jeu = 0.016 s | x = 3.20
frame 2 | jeu    | réel = 0.032 s | jeu = 0.032 s | x = 6.40
frame 3 | PAUSE  | réel = 0.048 s | jeu = 0.032 s | x = 6.40
frame 4 | PAUSE  | réel = 0.064 s | jeu = 0.032 s | x = 6.40
frame 5 | jeu    | réel = 0.080 s | jeu = 0.048 s | x = 9.60
frame 6 | jeu    | réel = 0.096 s | jeu = 0.048 s | x = 12.80
```

Pendant les frames 3 et 4, l'horloge réelle continue d'avancer, mais le temps de jeu et la position du héros sont **strictement figés**. Aucune ligne de code n'a été ajoutée dans la logique du héros : seul `dt` a changé.

La version graphique complète de ce mécanisme, avec un voile « PAUSE » et des boutons, se trouve dans le mini-projet de la section 20.30.

---

## 20.28 — Le ralenti et l'accéléré (facteur de temps)

Si `dt = 0` met en pause, que fait `dt × 0,5` ? Un ralenti. Et `dt × 2` ? Un accéléré.

Introduisons une variable, appelée traditionnellement `timeScale` (échelle de temps) :

```dart
double timeScale = 1.0;

// dans la boucle :
final double dtJeu = dtReel * timeScale;
update(dtJeu);
```

Le tableau des effets :

| `timeScale` | Effet | Cas d'usage |
| --- | --- | --- |
| 0.0 | pause complète | menu de pause |
| 0.15 | ralenti extrême | coup fatal sur le boss |
| 0.5 | ralenti | potion de lenteur, esquive |
| 1.0 | vitesse normale | jeu standard |
| 2.0 | double vitesse | mode accéléré d'un jeu de gestion |
| 4.0 | avance rapide | tests, rejeu de cinématique |
| -1.0 | marche arrière | rembobinage (très délicat) |

C'est un outil de game design puissant. L'effet de ralenti au moment d'un coup critique, très courant dans les jeux d'action, ne coûte qu'une ligne de code.

Il faut cependant distinguer **deux horloges** :

```text
  DEUX TEMPS DANS UN JEU

  ┌──────────────────────────────────────────────────────────┐
  │  dtJeu = dtReel * timeScale                              │
  │     -> le héros, les ennemis, la physique, les projectiles│
  │     -> ce qui doit ralentir                              │
  ├──────────────────────────────────────────────────────────┤
  │  dtReel                                                  │
  │     -> le menu, le compteur de FPS, les transitions       │
  │        d'interface, le chronomètre réel                   │
  │     -> ce qui NE doit PAS ralentir                       │
  └──────────────────────────────────────────────────────────┘
```

Si vous oubliez cette distinction, votre menu de pause s'animera au ralenti pendant l'effet de lenteur, ce qui paraît cassé.

Voici un `main.dart` complet avec trois vitesses, un effet de ralenti temporaire, et les deux horloges bien séparées.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: EcranTimeScale(),
    );
  }
}

class EcranTimeScale extends StatefulWidget {
  const EcranTimeScale({super.key});

  @override
  State<EcranTimeScale> createState() => _EcranTimeScaleState();
}

class _EcranTimeScaleState extends State<EcranTimeScale>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _timeScale = 1.0;

  // Ralenti temporaire déclenché par un bouton.
  double _ralentiRestant = 0; // en SECONDES RÉELLES

  double _heroX = 20;
  double _vx = 220;
  double _tempsJeu = 0;
  double _tempsReel = 0;
  static const double _taille = 28;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_onTick)..start();
  }

  void _onTick(Duration elapsed) {
    double dtReel = (elapsed - _precedent).inMicroseconds / 1000000.0;
    _precedent = elapsed;
    dtReel = dtReel.clamp(0.0, 0.05);

    // Le ralenti temporaire s'écoule en temps RÉEL.
    double echelle = _timeScale;
    if (_ralentiRestant > 0) {
      _ralentiRestant -= dtReel;
      echelle = 0.2;
    }

    final double dtJeu = dtReel * echelle;

    setState(() {
      _tempsReel += dtReel; // horloge réelle
      _tempsJeu += dtJeu;   // horloge du jeu

      _heroX += _vx * dtJeu;
      final double largeur = MediaQuery.of(context).size.width;
      if (_heroX < 0 || _heroX + _taille > largeur) {
        _vx = -_vx;
        _heroX = _heroX.clamp(0.0, largeur - _taille);
      }
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  Widget _bouton(String texte, VoidCallback action) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: ElevatedButton(onPressed: action, child: Text(texte)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF14161C),
      body: SafeArea(
        child: Stack(
          children: [
            Positioned(
              left: _heroX,
              top: 220,
              child: Container(
                width: _taille,
                height: _taille,
                color: const Color(0xFFE8B04B),
              ),
            ),
            Positioned(
              left: 16,
              top: 16,
              child: DefaultTextStyle(
                style: const TextStyle(
                  color: Colors.white70,
                  fontFamily: 'monospace',
                  fontSize: 14,
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('timeScale  : '
                        '${_timeScale.toStringAsFixed(2)}'),
                    Text('temps réel : '
                        '${_tempsReel.toStringAsFixed(2)} s'),
                    Text('temps jeu  : '
                        '${_tempsJeu.toStringAsFixed(2)} s'),
                    if (_ralentiRestant > 0)
                      Text('RALENTI : '
                          '${_ralentiRestant.toStringAsFixed(2)} s'),
                  ],
                ),
              ),
            ),
            Positioned(
              left: 12,
              bottom: 20,
              child: Row(
                children: [
                  _bouton('0x', () => setState(() => _timeScale = 0)),
                  _bouton('0.5x', () => setState(() => _timeScale = 0.5)),
                  _bouton('1x', () => setState(() => _timeScale = 1)),
                  _bouton('2x', () => setState(() => _timeScale = 2)),
                  _bouton('Coup critique !',
                      () => setState(() => _ralentiRestant = 1.5)),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** cinq boutons pilotent la vitesse du monde. Le compteur « temps réel » avance toujours à la même allure, tandis que « temps jeu » ralentit, s'arrête ou double selon le réglage. Le bouton « Coup critique ! » déclenche 1,5 seconde réelle de ralenti à 20 %, puis le jeu reprend sa vitesse.

> **Remarque de conception.** Notez que `_ralentiRestant` est décrémenté avec `dtReel`, pas avec `dtJeu`. Si vous utilisiez `dtJeu`, le ralenti se prolongerait lui-même : plus le jeu est lent, plus le compteur descend lentement, et l'effet durerait cinq fois plus longtemps. C'est une erreur très fréquente.

---

## 20.29 — Ce que Flame fait à votre place

Vous venez d'écrire à la main tout ce qui constitue le cœur d'un moteur de jeu. À partir du chapitre 27, le moteur Flame vous fournira tout cela. Il est important de savoir précisément ce qu'il prend en charge, pour ne pas le considérer comme une boîte noire.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  CE QUE VOUS AVEZ ÉCRIT      │  CE QUE FLAME FOURNIT           │
  ├──────────────────────────────┼─────────────────────────────────┤
  │  createTicker(_onTick)       │  inclus dans FlameGame          │
  │  _precedent / soustraction   │  inclus                         │
  │  conversion en secondes      │  inclus, dt est DÉJÀ en secondes│
  │  dt.clamp(0, 0.05)           │  inclus (dt plafonné)           │
  │  _update(dt)                 │  void update(double dt)         │
  │  build / CustomPainter       │  void render(Canvas canvas)     │
  │  setState à chaque frame     │  inutile, rendu direct          │
  │  ticker.dispose()            │  inclus dans onRemove           │
  │  pause : dt = 0              │  game.paused = true             │
  │  timeScale                   │  à écrire vous-même             │
  │  compteur de FPS             │  FpsTextComponent               │
  │  pas fixe / accumulateur     │  à écrire vous-même             │
  └────────────────────────────────────────────────────────────────┘
```

Concrètement, un jeu Flame minimal ressemble à ceci (nous le détaillerons au chapitre 27, ne le tapez pas encore) :

```dart
// APERÇU du chapitre 27 — ne fonctionne qu'avec le paquet flame installé.
class DonjonGame extends FlameGame {
  double heroX = 0;
  final double vitesse = 200;

  @override
  void update(double dt) {
    super.update(dt);
    heroX += vitesse * dt; // exactement ce que vous savez déjà faire
  }
}
```

Comparez avec le fichier de 120 lignes de la section 20.17. Flame supprime toute la plomberie et vous laisse **les deux méthodes qui comptent** : `update(dt)` et `render(canvas)`.

Ce que Flame ne fait **pas** à votre place, et que ce chapitre vous a appris :

| Responsabilité | Qui s'en occupe |
| --- | --- |
| Multiplier par `dt` dans vos calculs | **vous** |
| Choisir des vitesses en px/s | **vous** |
| Séparer la logique du rendu | **vous** |
| Ne pas mettre de calcul lourd dans `render` | **vous** |
| Décider d'un pas fixe si la physique l'exige | **vous** |
| Gérer un `timeScale` de ralenti | **vous** |

> **Conclusion importante.** Flame ne vous dispense pas de comprendre la boucle de jeu. Il vous dispense de la réécrire. Un développeur qui n'a jamais écrit sa propre boucle ne saura pas diagnostiquer un jeu qui va trop vite sur un téléphone récent. Vous, si.

---

## 20.30 — Mini-projet du chapitre : un moteur de boucle réutilisable

Assemblons tout ce chapitre dans un petit moteur que nous réutiliserons aux chapitres 21 à 26.

Cahier des charges :

1. une classe `MoteurDeBoucle` qui encapsule `Ticker`, `dt`, plafonnement, `timeScale`, pause et compteur de FPS ;
2. une classe abstraite `Jeu` avec deux méthodes à redéfinir : `update(double dt)` et `dessiner(Canvas, Size)` ;
3. un widget `VueDeJeu` qui branche les deux et redessine avec un `CustomPainter` ;
4. un exemple concret : trois entités du Donjon de Dart qui se déplacent à des vitesses différentes.

Voici le `main.dart` complet.

```dart
import 'dart:collection';
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

// ===========================================================
//  1. LE COMPTEUR DE FPS
// ===========================================================

class CompteurFps {
  CompteurFps({this.taille = 60});

  final int taille;
  final Queue<double> _dts = Queue<double>();
  double _somme = 0;
  double _pire = 0;

  void ajouter(double dt) {
    if (dt <= 0) return;
    _dts.addLast(dt);
    _somme += dt;
    if (dt > _pire) _pire = dt;
    while (_dts.length > taille) {
      _somme -= _dts.removeFirst();
    }
  }

  double get dtMoyen => _dts.isEmpty ? 0 : _somme / _dts.length;
  double get fps => dtMoyen > 0 ? 1 / dtMoyen : 0;
  double get pireDtMs => _pire * 1000;

  void reinitialiserPire() => _pire = 0;
}

// ===========================================================
//  2. LA CLASSE DE BASE D'UN JEU
// ===========================================================

/// Tout jeu de cette formation redéfinit ces deux méthodes.
abstract class Jeu {
  /// Taille de la zone de dessin, mise à jour par la vue.
  Size taille = Size.zero;

  /// Fait avancer le monde de [dt] secondes.
  void update(double dt);

  /// Dessine l'état courant. NE MODIFIE RIEN.
  void dessiner(Canvas canvas, Size taille);
}

// ===========================================================
//  3. LE MOTEUR DE BOUCLE
// ===========================================================

class MoteurDeBoucle extends ChangeNotifier {
  MoteurDeBoucle({
    required this.jeu,
    required TickerProvider vsync,
    this.dtMax = 0.05,
  }) {
    _ticker = vsync.createTicker(_onTick);
  }

  final Jeu jeu;
  final double dtMax;

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  final CompteurFps compteur = CompteurFps();

  bool enPause = false;
  double timeScale = 1.0;

  double tempsReel = 0;
  double tempsJeu = 0;
  int frames = 0;

  void demarrer() {
    if (!_ticker.isActive) _ticker.start();
  }

  void arreter() {
    if (_ticker.isActive) _ticker.stop();
  }

  void _onTick(Duration elapsed) {
    // 1. dt brut, en secondes.
    double dtReel =
        (elapsed - _precedent).inMicroseconds / 1000000.0;
    _precedent = elapsed;

    // 2. plafonnement : protection contre les pics.
    dtReel = dtReel.clamp(0.0, dtMax);

    // 3. statistiques (toujours en temps réel).
    compteur.ajouter(dtReel);
    tempsReel += dtReel;
    frames++;

    // 4. temps du jeu = temps réel x échelle, zéro si pause.
    final double dtJeu = enPause ? 0.0 : dtReel * timeScale;
    tempsJeu += dtJeu;

    // 5. mise à jour du monde.
    jeu.update(dtJeu);

    // 6. demande de redessin (sans reconstruire les widgets).
    notifyListeners();
  }

  @override
  void dispose() {
    _ticker.dispose(); // indispensable
    super.dispose();
  }
}

// ===========================================================
//  4. LE PEINTRE
// ===========================================================

class PeintreDeJeu extends CustomPainter {
  PeintreDeJeu({required this.jeu, required Listenable repaint})
      : super(repaint: repaint);

  final Jeu jeu;

  @override
  void paint(Canvas canvas, Size size) {
    jeu.taille = size;
    jeu.dessiner(canvas, size); // lecture seule
  }

  @override
  bool shouldRepaint(covariant PeintreDeJeu ancien) => false;
}

// ===========================================================
//  5. LA VUE
// ===========================================================

class VueDeJeu extends StatefulWidget {
  const VueDeJeu({super.key, required this.jeu});

  final Jeu jeu;

  @override
  State<VueDeJeu> createState() => _VueDeJeuState();
}

class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  late final MoteurDeBoucle moteur;

  @override
  void initState() {
    super.initState();
    moteur = MoteurDeBoucle(jeu: widget.jeu, vsync: this);
    // La barre de contrôle affiche les FPS : elle se reconstruit
    // à chaque frame, mais le dessin du jeu, lui, passe par repaint.
    moteur.addListener(_rafraichirBarre);
    moteur.demarrer();
  }

  void _rafraichirBarre() {
    if (mounted) setState(() {});
  }

  @override
  void dispose() {
    moteur.removeListener(_rafraichirBarre);
    moteur.dispose();
    super.dispose();
  }

  Widget _bouton(String texte, VoidCallback action) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: ElevatedButton(onPressed: action, child: Text(texte)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Expanded(
          child: CustomPaint(
            painter: PeintreDeJeu(jeu: widget.jeu, repaint: moteur),
            size: Size.infinite,
          ),
        ),
        Container(
          color: const Color(0xFF1B1F27),
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
          child: Row(
            children: [
              _bouton(
                moteur.enPause ? 'Reprendre' : 'Pause',
                () => moteur.enPause = !moteur.enPause,
              ),
              _bouton('Ralenti', () => moteur.timeScale = 0.3),
              _bouton('Normal', () => moteur.timeScale = 1.0),
              const Spacer(),
              Text(
                '${moteur.compteur.fps.toStringAsFixed(0)} FPS',
                style: const TextStyle(
                  color: Colors.greenAccent,
                  fontFamily: 'monospace',
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// ===========================================================
//  6. LE JEU : DONJON DE DART
// ===========================================================

class Entite {
  Entite({
    required this.nom,
    required this.x,
    required this.y,
    required this.vitesse,
    required this.couleur,
    required this.taille,
  });

  final String nom;
  double x;
  double y;
  double vitesse; // pixels par seconde
  final Color couleur;
  final double taille;
}

class DonjonJeu extends Jeu {
  DonjonJeu() {
    entites = [
      Entite(
        nom: 'héros',
        x: 20,
        y: 120,
        vitesse: 160,
        couleur: const Color(0xFFE8B04B),
        taille: 30,
      ),
      Entite(
        nom: 'gobelin',
        x: 20,
        y: 200,
        vitesse: 70,
        couleur: const Color(0xFF6FBF73),
        taille: 24,
      ),
      Entite(
        nom: 'flèche',
        x: 20,
        y: 280,
        vitesse: 380,
        couleur: const Color(0xFFCF5C5C),
        taille: 14,
      ),
    ];
  }

  late final List<Entite> entites;
  double temps = 0;

  @override
  void update(double dt) {
    temps += dt;

    for (final Entite e in entites) {
      e.x += e.vitesse * dt;

      // Rebond sur les bords de la zone.
      if (e.x < 0) {
        e.x = 0;
        e.vitesse = -e.vitesse;
      } else if (e.x + e.taille > taille.width && taille.width > 0) {
        e.x = taille.width - e.taille;
        e.vitesse = -e.vitesse;
      }
    }
  }

  @override
  void dessiner(Canvas canvas, Size size) {
    // Fond.
    final Paint fond = Paint()..color = const Color(0xFF14161C);
    canvas.drawRect(Offset.zero & size, fond);

    // Lignes de couloir.
    final Paint ligne = Paint()
      ..color = const Color(0xFF262A33)
      ..strokeWidth = 1;
    for (final Entite e in entites) {
      canvas.drawLine(
        Offset(0, e.y + e.taille + 6),
        Offset(size.width, e.y + e.taille + 6),
        ligne,
      );
    }

    // Entités.
    for (final Entite e in entites) {
      final Paint p = Paint()..color = e.couleur;
      canvas.drawRRect(
        RRect.fromRectAndRadius(
          Rect.fromLTWH(e.x, e.y, e.taille, e.taille),
          const Radius.circular(4),
        ),
        p,
      );
    }
  }
}

// ===========================================================
//  7. L'APPLICATION
// ===========================================================

void main() => runApp(DonjonApp());

class DonjonApp extends StatelessWidget {
  DonjonApp({super.key});

  final DonjonJeu jeu = DonjonJeu();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: SafeArea(child: VueDeJeu(jeu: jeu)),
      ),
    );
  }
}
```

**Résultat :** trois rectangles colorés traversent l'écran à trois vitesses différentes (160, 70 et 380 pixels par seconde), rebondissent sur les bords, et une barre de contrôle permet de mettre en pause, de passer au ralenti et de revenir en vitesse normale. Le compteur de FPS s'affiche à droite.

Ce qu'il faut retenir de l'architecture :

```text
  ARCHITECTURE DU MINI-PROJET

  ┌────────────────┐
  │  VueDeJeu      │  widget Flutter, possède le TickerProvider
  └───────┬────────┘
          │ crée
          ▼
  ┌────────────────┐  Ticker -> dt -> clamp -> timeScale -> pause
  │ MoteurDeBoucle │  puis jeu.update(dt) puis notifyListeners()
  └───────┬────────┘
          │ notifie
          ▼
  ┌────────────────┐
  │ PeintreDeJeu   │  CustomPainter, repaint: moteur
  └───────┬────────┘
          │ appelle
          ▼
  ┌────────────────┐
  │  Jeu (abstrait)│  update(dt)  +  dessiner(canvas, size)
  └───────┬────────┘
          │ implémenté par
          ▼
  ┌────────────────┐
  │  DonjonJeu     │  les entités du Donjon de Dart
  └────────────────┘
```

Deux détails techniques valent d'être soulignés.

**`ChangeNotifier` et `repaint:`.** Le `MoteurDeBoucle` étend `ChangeNotifier`. En le passant comme `repaint:` au `CustomPainter`, on demande à Flutter de **redessiner sans reconstruire l'arbre de widgets**. C'est bien plus efficace qu'un `setState` par frame. Nous approfondirons au chapitre 21.

**`shouldRepaint` renvoie `false`.** Ce n'est pas une erreur : le redessin est déclenché par le `Listenable`, pas par la comparaison des peintres.

> **Gardez ce fichier.** Nous le ferons évoluer dans tous les chapitres de la PARTIE 2A : le chapitre 21 y ajoutera un vrai dessin au `Canvas`, le 23 la physique, le 24 les collisions, le 26 les états de jeu.

---

## 20.31 — Erreurs fréquentes

Ce tableau rassemble les erreurs que l'on retrouve dans presque tous les premiers jeux. Relisez-le avant chaque chapitre de la PARTIE 2 : la moitié des bugs de jeu vient de cette liste.

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le héros va deux fois plus vite sur un téléphone à 120 Hz | Déplacement écrit `x += 5` par frame : la vitesse dépend du nombre d'images, pas du temps | Écrire `x += vitesse * dt` et exprimer `vitesse` en pixels par seconde |
| L'application se fige et affiche « Not responding » | Boucle `while (true)` dans `main()` ou dans un `build()` : le thread d'interface ne peut plus dessiner | Piloter la boucle avec un `Ticker` (`createTicker`) ou `SchedulerBinding`, jamais avec `while (true)` |
| Après un retour d'arrière-plan, le joueur a traversé tout le niveau | Le premier `dt` après la reprise vaut plusieurs secondes, donc `vitesse * dt` est énorme | Plafonner : `dt = dt.clamp(0.0, 0.05)` avant tout calcul |
| Le personnage traverse les murs quand le jeu rame | Un `dt` important déplace l'entité au-delà du mur en une seule étape (tunneling) | Plafonner le `dt`, ou passer à un pas de temps fixe avec accumulateur (section 20.23) |
| `dt` vaut zéro à la première frame et provoque une division par zéro | `_precedent` est initialisé à `Duration.zero` et la première soustraction donne 0 | Ignorer la première frame, ou tester `if (dt <= 0) return;` avant de calculer des FPS |
| Le compteur de FPS clignote entre 47 et 63 et devient illisible | Les FPS sont recalculés sur une seule frame : `1 / dt` est très instable | Lisser sur une fenêtre de 30 à 60 frames (moyenne glissante, section 20.19) |
| Les FPS restent bloqués à 60 alors que l'écran est à 120 Hz | Le mode debug bride le rendu et ajoute des vérifications coûteuses | Mesurer les performances en `flutter run --profile`, jamais en debug |
| Le jeu consomme la batterie même quand l'écran ne bouge pas | Le `Ticker` continue de tourner alors que rien n'a besoin d'être animé | Appeler `ticker.stop()` sur les menus et les écrans statiques, `start()` en revenant au jeu |
| `Ticker` toujours actif après avoir quitté l'écran, et exception dans la console | Le `Ticker` n'a pas été libéré dans `dispose()` | Appeler `_ticker.dispose()` dans `dispose()` du `State`, avant `super.dispose()` |
| `Unhandled exception: Ticker created but not disposed` au hot reload | Un `Ticker` a été créé sans `SingleTickerProviderStateMixin` correctement libéré | Utiliser `with SingleTickerProviderStateMixin` (un seul ticker) ou `TickerProviderStateMixin` (plusieurs), et libérer dans `dispose()` |
| Le monde continue d'avancer pendant l'écran de pause | La pause a été gérée sur l'affichage, pas sur le temps : `update` reçoit toujours le `dt` réel | Passer `dt = 0` à `update` en pause, ou ne pas appeler `update` du tout |
| En pause, la première frame après reprise fait un bond énorme | Le temps de la pause s'est accumulé dans `_precedent` | Continuer à mettre `_precedent` à jour pendant la pause, et n'annuler que le `dt` transmis au jeu |
| Le ralenti ralentit aussi l'interface et le compteur de FPS | Le `timeScale` a été appliqué au `dt` utilisé pour les statistiques | Séparer `dtReel` (statistiques, animations d'interface) et `dtJeu = dtReel * timeScale` (monde) |
| La physique explose quand le jeu passe sous 20 FPS | Avec un pas variable, un grand `dt` produit une intégration très imprécise | Utiliser un pas de temps fixe pour la physique (section 20.22) et laisser le rendu en pas variable |
| Le jeu ralentit de plus en plus jusqu'à se bloquer | Spirale de la mort : l'accumulateur exige plus de pas que la machine ne peut en calculer | Limiter le nombre de pas par frame (`while (accu >= pas && n < 5)`) et vider l'excédent |
| Le mouvement est saccadé alors que les FPS sont bons | Rendu à 60 Hz d'un monde simulé à 10 Hz, sans interpolation | Interpoler entre l'état précédent et l'état courant avec `alpha = accumulateur / pasFixe` (section 20.26) |
| Le rendu modifie l'état du jeu | Du code de logique (déplacement, collision, score) a été écrit dans `paint()` ou `render()` | `update` écrit, `render` lit : aucune modification d'état dans la méthode de dessin |
| `setState` appelé 60 fois par seconde et interface qui rame | Chaque frame reconstruit tout l'arbre de widgets | Passer un `Listenable` en `repaint:` au `CustomPainter` : Flutter redessine sans reconstruire |
| Le `dt` semble correct mais tout est mille fois trop lent | `elapsed` a été divisé par `1000` (millisecondes) au lieu de `1000000` (microsecondes) | `dt = (elapsed - precedent).inMicroseconds / 1000000.0` |
| Les FPS mesurés à 61 ou 59 au lieu de 60 pile | `elapsedMilliseconds` est un entier : 16 ou 17 ms selon les frames | Mesurer en microsecondes, jamais en millisecondes entières |

---

## 20.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Application vs jeu | Une application **réagit** à des événements, un jeu **tourne** en permanence, même sans action du joueur |
| Boucle de jeu | Un cycle sans fin : **entrées → mise à jour → rendu**, répété plusieurs dizaines de fois par seconde |
| Frame (image) | Un tour complet de la boucle, qui se termine par une image affichée à l'écran |
| FPS | Nombre d'images affichées par seconde. `FPS = 1 / dt` |
| Budget de frame | `durée en ms = 1000 / FPS`. 60 FPS = 16,67 ms ; 30 FPS = 33,33 ms ; 120 FPS = 8,33 ms |
| Problème du framerate variable | Toutes les machines ne rendent pas le même nombre de frames par seconde |
| Bug `x += 5` | Déplacer d'une valeur fixe **par frame** rend la vitesse dépendante de la machine |
| Delta time (`dt`) | Temps écoulé, en secondes, depuis la frame précédente |
| Formule fondamentale | `position += vitesse * dt` — à appliquer à **tout** ce qui évolue dans le temps |
| Unités | Les vitesses s'expriment en **pixels par seconde**, les accélérations en pixels par seconde carrée |
| `Stopwatch` | Mesure le temps écoulé ; utiliser `elapsedMicroseconds` pour la précision |
| `Ticker` | Objet Flutter qui rappelle une fonction à chaque frame, synchronisé sur l'écran |
| `SingleTickerProviderStateMixin` | Fournit le `vsync` nécessaire à `createTicker` ; impose de libérer le ticker dans `dispose()` |
| `SchedulerBinding` | Alternative bas niveau au `Ticker` : `addPostFrameCallback`, exécuté une seule fois par frame |
| Affichage des FPS | `fps = 1 / dt` est trop instable pour être lu directement |
| Moyenne glissante | Lisser sur 30 à 60 frames donne un compteur stable et honnête |
| `dt` aberrant | Après un pic de lag ou un retour d'arrière-plan, `dt` peut valoir plusieurs secondes |
| Plafonnement | `dt = dt.clamp(0.0, 0.05)` : le jeu ralentit un instant au lieu de se téléporter |
| Pas de temps variable | `update` reçoit le `dt` réel. Simple, fluide, mais physique non reproductible |
| Pas de temps fixe | `update` reçoit toujours la même valeur (`1/60`). Reproductible, stable, indispensable en physique |
| Accumulateur | Stocke le temps non encore simulé et le consomme par pas entiers |
| Spirale de la mort | Si chaque frame demande plus de pas que la machine ne peut en faire, tout s'effondre : limiter le nombre de pas |
| Découplage | La mise à jour et le rendu n'ont pas à tourner à la même cadence |
| Interpolation | `alpha = accumulateur / pasFixe`, puis `position affichée = ancienne + (nouvelle - ancienne) * alpha` |
| Pause | Ne pas arrêter le ticker : transmettre `dt = 0` au monde tout en continuant à mesurer le temps réel |
| `timeScale` | `dtJeu = dtReel * timeScale` : 0 = pause, 0,3 = ralenti, 2 = accéléré |
| Rôle de Flame | Flame fournit la boucle, le `dt` déjà en secondes, le plafonnement, `update` et `render` |
| Ce qui reste à votre charge | Multiplier par `dt`, choisir les unités, séparer logique et rendu, décider du pas fixe |
| Règle d'or | `update` **écrit** l'état, `render` **lit** l'état. Jamais l'inverse |

---

## 20.33 — Exercices

Les exercices 1 à 5 et 8 tournent dans DartPad en mode Dart (console). Les exercices 6, 7, 9 et 10 sont des `main.dart` Flutter complets : créez un projet avec `flutter create`, remplacez `lib/main.dart` par le code, puis `flutter run`.

### Exercice 1 — Le budget d'une frame (facile)

Écrivez un programme console qui affiche, pour les cadences 24, 30, 60, 90, 120 et 144 FPS, la durée d'une frame en millisecondes avec trois décimales.

Puis faites l'opération inverse : pour les durées 33,3 ms, 16,7 ms et 8,3 ms, affichez le nombre de FPS correspondant avec deux décimales.

Attendu : un tableau texte lisible, aligné.

### Exercice 2 — Réparer un déplacement (facile)

Voici un déplacement buggé, tiré d'un vieux prototype du Donjon de Dart :

```dart
// BUGGÉ : la vitesse dépend du nombre de frames.
heroX += 5;
```

Le jeu avait été réglé sur un écran 30 Hz, et le héros devait parcourir 1500 pixels en 10 secondes.

1. Calculez la vitesse correspondante en pixels par seconde.
2. Écrivez un programme qui simule 10 secondes de jeu à 30 FPS puis à 60 FPS, une fois avec la version buggée, une fois avec la version corrigée.
3. Affichez les quatre distances finales et concluez.

### Exercice 3 — Mesurer un vrai `dt` (facile)

Écrivez une classe `Horloge` qui expose une méthode `double tick()` renvoyant le temps écoulé depuis l'appel précédent, en secondes, mesuré avec `elapsedMicroseconds`.

Utilisez-la dans une boucle de cinq itérations, chacune séparée par une attente de 40 millisecondes, et affichez à chaque tour le `dt` en millisecondes ainsi que les FPS instantanés.

### Exercice 4 — Compteur de FPS lissé (moyen)

Écrivez une classe `FpsLisses` avec :

- un constructeur `FpsLisses({int fenetre = 5})` ;
- une méthode `void ajouter(double dt)` qui ignore les `dt` négatifs ou nuls ;
- un accesseur `double get dtMoyen` ;
- un accesseur `double get fps`.

La fenêtre doit être glissante : au-delà de `fenetre` valeurs, la plus ancienne est retirée. Utilisez une `Queue` (chapitre 6) et maintenez une somme courante plutôt que de tout re-parcourir.

Testez-la sur la série suivante, qui contient un pic de lag, et affichez frame par frame les FPS instantanés et les FPS lissés :

```dart
const List<double> dts = [
  0.016, 0.017, 0.016, 0.016, 0.017,
  0.200,
  0.016, 0.017, 0.016, 0.016, 0.017, 0.016,
];
```

### Exercice 5 — Protéger le jeu d'un `dt` aberrant (moyen)

Le joueur passe le jeu en arrière-plan pendant 1,4 seconde, puis revient. Une flèche se déplace à 400 pixels par seconde.

1. Écrivez une fonction `double dtSecurise(double dt, {double maximum = 0.05})`.
2. Simulez la série `[0.016, 0.017, 1.400, 0.016, 0.017]` deux fois : sans protection, puis avec.
3. Affichez les deux positions finales et l'écart.
4. Expliquez en une phrase ce que le joueur aurait vu à l'écran sans protection.

### Exercice 6 — Un carré qui rebondit (moyen)

Écrivez un `main.dart` Flutter complet qui affiche un carré orange de 40 pixels se déplaçant horizontalement à 220 pixels par seconde, rebondissant sur les bords de l'écran.

Contraintes :

- boucle pilotée par un `Ticker` créé avec `createTicker` ;
- `dt` calculé en microsecondes puis plafonné à 0,05 s ;
- `Ticker` libéré dans `dispose()` ;
- dessin avec un `CustomPainter` ;
- affichage du nombre de rebonds en haut de l'écran.

### Exercice 7 — Ralenti, normal, accéléré (moyen)

Reprenez l'exercice 6 et ajoutez trois boutons : « Ralenti » (`timeScale = 0.25`), « Normal » (`timeScale = 1.0`) et « Accéléré » (`timeScale = 3.0`), plus un bouton « Pause ».

Affichez en permanence deux chronomètres : le **temps réel** écoulé depuis le lancement et le **temps de jeu** écoulé. Vérifiez que le second prend du retard en ralenti, de l'avance en accéléré, et se fige en pause.

### Exercice 8 — L'accumulateur à pas fixe (difficile)

Simulez une boucle à pas fixe de 1/50 de seconde (20 ms) alimentée par la série de `dt` réels suivante :

```dart
const List<double> dts = [
  0.021, 0.019, 0.045, 0.007, 0.009, 0.062,
  0.020, 0.011, 0.033, 0.018, 0.005, 0.041,
];
```

Un objet avance à 100 pixels par seconde.

Affichez frame par frame : le `dt` réel en millisecondes, le nombre de pas fixes exécutés et le contenu restant de l'accumulateur en millisecondes.

Puis vérifiez la propriété fondamentale de l'accumulateur :

```text
  temps simulé + accumulateur restant = temps réel écoulé
```

### Exercice 9 — Interpolation entre deux états (difficile)

Écrivez un `main.dart` Flutter qui simule le monde à **8 pas fixes par seconde** seulement (`pasFixe = 1 / 8`), tout en rendant à la cadence de l'écran.

Affichez deux carrés :

- le carré du haut est dessiné à la position brute issue de la simulation (il avancera par à-coups) ;
- le carré du bas est dessiné à la position **interpolée** entre l'état précédent et l'état courant, avec `alpha = accumulateur / pasFixe`.

Ajoutez un bouton qui active et désactive l'interpolation du carré du bas pour comparer les deux rendus.

### Exercice 10 — Le moteur de boucle complet (difficile)

Reprenez le `MoteurDeBoucle` de la section 20.30 et enrichissez-le :

1. un accesseur `double get fps` (FPS lissés sur 60 frames) ;
2. un accesseur `double get pireFrameMs` (la frame la plus lente de la fenêtre) ;
3. deux compteurs séparés `tempsReel` et `tempsJeu`, affichés à l'écran ;
4. une méthode `void bulletTime(double duree)` qui met `timeScale` à 0,2 puis le fait revenir progressivement à 1,0 en `duree` secondes ;
5. un bouton « Coup critique » qui déclenche `bulletTime(1.5)`.

Le jeu affiché reste celui du Donjon de Dart : le héros, le gobelin et la flèche, à trois vitesses différentes.

---

## 20.34 — Corrections des exercices

### Correction 1

```dart
void main() {
  const List<int> cadences = [24, 30, 60, 90, 120, 144];

  print('FPS    durée d\'une frame');
  print('---------------------------');
  for (final int fps in cadences) {
    final double ms = 1000 / fps;
    print('${fps.toString().padLeft(3)}    '
        '${ms.toStringAsFixed(3).padLeft(7)} ms');
  }

  print('');

  const List<double> durees = [33.3, 16.7, 8.3];
  print('durée      FPS');
  print('---------------------------');
  for (final double ms in durees) {
    final double fps = 1000 / ms;
    print('${ms.toStringAsFixed(1).padLeft(5)} ms  '
        '${fps.toStringAsFixed(2).padLeft(6)}');
  }
}
```

**Résultat :**

```text
FPS    durée d'une frame
---------------------------
 24     41.667 ms
 30     33.333 ms
 60     16.667 ms
 90     11.111 ms
120      8.333 ms
144      6.944 ms

durée      FPS
---------------------------
 33.3 ms   30.03
 16.7 ms   59.88
  8.3 ms  120.48
```

**Explication :** les deux calculs sont l'inverse l'un de l'autre. `durée = 1000 / FPS` et `FPS = 1000 / durée`, quand la durée est exprimée en millisecondes. Ce tableau est le **budget** dont vous disposez : à 60 FPS, tout ce que fait votre jeu pendant une frame — entrées, mise à jour, dessin — doit tenir dans 16,667 millisecondes. Retenez surtout la troisième et la cinquième ligne : 16,7 ms et 8,3 ms sont les deux budgets que vous rencontrerez le plus souvent, sur un écran 60 Hz et sur un écran 120 Hz. La méthode `padLeft` sert uniquement à aligner les colonnes ; elle vient de la classe `String` vue au chapitre 2.

---

### Correction 2

```dart
void main() {
  const double duree = 10.0;      // secondes
  const double distance = 1500.0; // pixels
  const double vitesse = distance / duree; // 150 px/s

  print('Vitesse voulue : ${vitesse.toStringAsFixed(0)} px/s');
  print('Distance visée : ${distance.toStringAsFixed(1)} px '
      'en ${duree.toStringAsFixed(0)} s');
  print('');

  for (final int fps in [30, 60]) {
    final int frames = (fps * duree).round();
    final double dt = 1 / fps;

    double bugge = 0;
    double corrige = 0;

    for (int i = 0; i < frames; i++) {
      bugge += 5; // valeur FIXE par frame  -> dépend de la machine
      corrige += vitesse * dt; // proportionnel au TEMPS -> indépendant
    }

    print('$fps FPS  ($frames frames)');
    print('   buggé   : ${bugge.toStringAsFixed(1)} px');
    print('   corrigé : ${corrige.toStringAsFixed(1)} px');
  }
}
```

**Résultat :**

```text
Vitesse voulue : 150 px/s
Distance visée : 1500.0 px en 10 s

30 FPS  (300 frames)
   buggé   : 1500.0 px
   corrigé : 1500.0 px
60 FPS  (600 frames)
   buggé   : 3000.0 px
   corrigé : 1500.0 px
```

**Explication :** la vitesse se déduit d'une simple division : 1500 pixels en 10 secondes font 150 pixels par seconde. Observez la première ligne de résultats : à 30 FPS, la version buggée donne exactement le bon résultat. C'est normal, et c'est le piège. Le jeu avait été **réglé** sur cette cadence : `5 pixels × 30 frames = 150 pixels par seconde`. Tant que le développeur teste sur son écran 30 Hz, tout va bien.

Le jour où le jeu tourne sur un écran 60 Hz, le nombre de frames double, la valeur ajoutée par frame ne change pas, et le héros parcourt le double de la distance. La version corrigée, elle, donne 1500 pixels dans les deux cas : le nombre de frames a doublé, mais `dt` a été divisé par deux, et le produit `vitesse × dt` compense exactement. C'est toute la démonstration du chapitre en six lignes.

---

### Correction 3

```dart
import 'dart:async';

class Horloge {
  final Stopwatch _chrono = Stopwatch();
  int _precedentMicro = 0;

  void demarrer() {
    _chrono.start();
    _precedentMicro = _chrono.elapsedMicroseconds;
  }

  /// Temps écoulé, en secondes, depuis l'appel précédent.
  double tick() {
    final int maintenant = _chrono.elapsedMicroseconds;
    final int delta = maintenant - _precedentMicro;
    _precedentMicro = maintenant;
    return delta / 1000000.0;
  }

  double get tempsTotal => _chrono.elapsedMicroseconds / 1000000.0;
}

Future<void> main() async {
  final Horloge horloge = Horloge()..demarrer();

  print('tour    dt (ms)     FPS');
  print('--------------------------');

  for (int i = 1; i <= 5; i++) {
    await Future<void>.delayed(const Duration(milliseconds: 40));

    final double dt = horloge.tick();
    final double fps = dt > 0 ? 1 / dt : 0;

    print('${i.toString().padLeft(3)}  '
        '${(dt * 1000).toStringAsFixed(2).padLeft(9)}  '
        '${fps.toStringAsFixed(2).padLeft(8)}');
  }

  print('');
  print('Temps total : ${horloge.tempsTotal.toStringAsFixed(3)} s');
}
```

**Résultat (valeurs variables d'une exécution à l'autre) :**

```text
tour    dt (ms)     FPS
--------------------------
  1      41.37     24.17
  2      40.92     24.44
  3      41.05     24.36
  4      40.88     24.46
  5      41.11     24.32

Temps total : 0.205 s
```

**Explication :** trois points méritent votre attention.

Premièrement, la mesure se fait en **microsecondes**. Si vous aviez utilisé `elapsedMilliseconds`, vous auriez lu tantôt 40, tantôt 41, avec une précision insuffisante pour un jeu à 120 FPS.

Deuxièmement, le `dt` mesuré n'est jamais exactement 40 millisecondes. Il vaut 41,37 ou 40,92 : un `Future.delayed` de 40 ms garantit **au moins** 40 ms, jamais exactement 40 ms. C'est précisément la raison d'être du delta time : on ne suppose pas le temps écoulé, on le mesure.

Troisièmement, les FPS affichés tournent autour de 24, ce qui est cohérent : `1 / 0,041 ≈ 24`. Vos chiffres seront différents des miens ; seul l'ordre de grandeur compte.

---

### Correction 4

```dart
import 'dart:collection';

class FpsLisses {
  FpsLisses({this.fenetre = 5});

  final int fenetre;
  final Queue<double> _dts = Queue<double>();
  double _somme = 0;

  void ajouter(double dt) {
    if (dt <= 0) return; // on ignore les valeurs absurdes

    _dts.addLast(dt);
    _somme += dt;

    // Fenêtre glissante : on retire les plus anciennes.
    while (_dts.length > fenetre) {
      _somme -= _dts.removeFirst();
    }
  }

  double get dtMoyen => _dts.isEmpty ? 0 : _somme / _dts.length;
  double get fps => dtMoyen > 0 ? 1 / dtMoyen : 0;
}

void main() {
  const List<double> dts = [
    0.016, 0.017, 0.016, 0.016, 0.017,
    0.200,
    0.016, 0.017, 0.016, 0.016, 0.017, 0.016,
  ];

  final FpsLisses compteur = FpsLisses(fenetre: 5);

  print('frame   dt(ms)   FPS instantané   FPS lissé');
  print('---------------------------------------------');

  for (int i = 0; i < dts.length; i++) {
    final double dt = dts[i];
    compteur.ajouter(dt);

    print('${(i + 1).toString().padLeft(3)} '
        '${(dt * 1000).toStringAsFixed(1).padLeft(7)} '
        '${(1 / dt).toStringAsFixed(1).padLeft(12)} '
        '${compteur.fps.toStringAsFixed(1).padLeft(12)}');
  }
}
```

**Résultat :**

```text
frame   dt(ms)   FPS instantané   FPS lissé
---------------------------------------------
  1    16.0         62.5         62.5
  2    17.0         58.8         60.6
  3    16.0         62.5         61.2
  4    16.0         62.5         61.5
  5    17.0         58.8         61.0
  6   200.0          5.0         18.8
  7    16.0         62.5         18.9
  8    17.0         58.8         18.8
  9    16.0         62.5         18.8
 10    16.0         62.5         18.9
 11    17.0         58.8         61.0
 12    16.0         62.5         61.0
```

**Explication :** la colonne « FPS instantané » saute de 62,5 à 5,0 puis remonte immédiatement à 62,5. Sur un écran, ce chiffre serait illisible.

La colonne lissée raconte une histoire plus utile. Elle chute à 18,8 à la frame 6 et **reste basse pendant cinq frames**, le temps que le pic sorte de la fenêtre. À la frame 11, la valeur `0,200` est enfin évacuée et le compteur remonte à 61. Un compteur lissé vous dit donc : « une grosse frame vient de se produire, et voici son impact moyen ». C'est exactement l'information dont vous avez besoin.

Notez l'optimisation : on ne recalcule jamais la somme complète. On ajoute la nouvelle valeur, on retranche l'ancienne. Le coût est constant quelle que soit la taille de la fenêtre. Avec une fenêtre de 60 frames, comme dans le mini-projet, le lissage sera encore plus doux mais réagira plus lentement.

---

### Correction 5

```dart
/// Empêche un dt aberrant de traverser le moteur de jeu.
double dtSecurise(double dt, {double maximum = 0.05}) {
  return dt.clamp(0.0, maximum);
}

void main() {
  const List<double> dts = [0.016, 0.017, 1.400, 0.016, 0.017];
  const double vitesse = 400; // pixels par seconde

  double sansProtection = 0;
  double avecProtection = 0;

  print('frame   dt réel(ms)   dt utilisé(ms)   x sans   x avec');
  print('--------------------------------------------------------');

  for (int i = 0; i < dts.length; i++) {
    final double dtReel = dts[i];
    final double dtUtile = dtSecurise(dtReel);

    sansProtection += vitesse * dtReel;
    avecProtection += vitesse * dtUtile;

    print('${(i + 1).toString().padLeft(3)} '
        '${(dtReel * 1000).toStringAsFixed(1).padLeft(12)} '
        '${(dtUtile * 1000).toStringAsFixed(1).padLeft(16)} '
        '${sansProtection.toStringAsFixed(1).padLeft(8)} '
        '${avecProtection.toStringAsFixed(1).padLeft(8)}');
  }

  print('');
  print('Position finale sans protection : '
      '${sansProtection.toStringAsFixed(1)} px');
  print('Position finale avec protection : '
      '${avecProtection.toStringAsFixed(1)} px');
  print('Écart                           : '
      '${(sansProtection - avecProtection).toStringAsFixed(1)} px');
}
```

**Résultat :**

```text
frame   dt réel(ms)   dt utilisé(ms)   x sans   x avec
--------------------------------------------------------
  1         16.0             16.0      6.4      6.4
  2         17.0             17.0     13.2     13.2
  3       1400.0             50.0    573.2     33.2
  4         16.0             16.0    579.6     39.6
  5         17.0             17.0    586.4     46.4

Position finale sans protection : 586.4 px
Position finale avec protection : 46.4 px
Écart                           : 540.0 px
```

**Explication :** à la frame 3, le `dt` réel vaut 1,4 seconde. Sans protection, la flèche avance de `400 × 1,4 = 560` pixels **en une seule image**. Le joueur ne voit pas la flèche voler : il la voit disparaître d'un côté de l'écran et réapparaître de l'autre. Si un mur ou un ennemi se trouvait sur ce segment de 560 pixels, la flèche l'aurait purement et simplement traversé sans qu'aucune collision ne soit détectée. C'est le phénomène de **tunneling** que nous étudierons au chapitre 24.

Avec la protection, le `dt` de la frame 3 est ramené à 50 millisecondes. La flèche avance de 20 pixels. Le jeu a « perdu » 1,35 seconde de temps simulé : le monde a ralenti un instant. C'est un compromis assumé. Entre un jeu qui ralentit une demi-seconde et un jeu où le personnage se téléporte à travers les murs, tous les moteurs choisissent le ralentissement.

Le choix de `0.05` correspond à 20 FPS : en dessous de cette cadence, on considère que la machine est en difficulté et on refuse de simuler davantage. Certains moteurs utilisent `1/30` (0,033) pour être plus prudents encore.

---

### Correction 6

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const RebondApp());

class RebondApp extends StatelessWidget {
  const RebondApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14161C),
        body: SafeArea(child: EcranRebond()),
      ),
    );
  }
}

class EcranRebond extends StatefulWidget {
  const EcranRebond({super.key});

  @override
  State<EcranRebond> createState() => _EcranRebondState();
}

class _EcranRebondState extends State<EcranRebond>
    with SingleTickerProviderStateMixin {
  static const double cote = 40;

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _x = 0;
  double _vitesse = 220; // pixels par seconde
  int _rebonds = 0;
  double _largeur = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surFrame)..start();
  }

  void _surFrame(Duration ecoule) {
    // 1. dt en secondes, mesuré en microsecondes.
    double dt = (ecoule - _precedent).inMicroseconds / 1000000.0;
    _precedent = ecoule;

    // 2. protection contre les pics de lag.
    dt = dt.clamp(0.0, 0.05);

    // 3. mise à jour du monde.
    _x += _vitesse * dt;

    if (_x < 0) {
      _x = 0;
      _vitesse = -_vitesse;
      _rebonds++;
    } else if (_largeur > 0 && _x + cote > _largeur) {
      _x = _largeur - cote;
      _vitesse = -_vitesse;
      _rebonds++;
    }

    // 4. demande de rendu.
    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose(); // OBLIGATOIRE
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints contraintes) {
        _largeur = contraintes.maxWidth;
        final double y = contraintes.maxHeight / 2 - cote / 2;

        return Stack(
          children: [
            Positioned.fill(
              child: CustomPaint(
                painter: PeintreCarre(x: _x, y: y, cote: cote),
              ),
            ),
            Positioned(
              top: 12,
              left: 12,
              child: Text(
                'Rebonds : $_rebonds\n'
                'x       : ${_x.toStringAsFixed(0)} px\n'
                'vitesse : ${_vitesse.toStringAsFixed(0)} px/s',
                style: const TextStyle(
                  color: Colors.white70,
                  fontFamily: 'monospace',
                  fontSize: 14,
                ),
              ),
            ),
          ],
        );
      },
    );
  }
}

class PeintreCarre extends CustomPainter {
  const PeintreCarre({
    required this.x,
    required this.y,
    required this.cote,
  });

  final double x;
  final double y;
  final double cote;

  @override
  void paint(Canvas canvas, Size size) {
    // Fond.
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF14161C),
    );

    // Ligne de sol.
    canvas.drawLine(
      Offset(0, y + cote + 10),
      Offset(size.width, y + cote + 10),
      Paint()
        ..color = const Color(0xFF2A2F3A)
        ..strokeWidth = 2,
    );

    // Le carré.
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(x, y, cote, cote),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFFE8B04B),
    );
  }

  @override
  bool shouldRepaint(covariant PeintreCarre ancien) => ancien.x != x;
}
```

**Résultat :** un carré orange traverse l'écran en un peu plus de deux secondes sur un téléphone de 500 pixels de large, rebondit sur le bord droit, repart vers la gauche, et le compteur de rebonds s'incrémente à chaque contact. La vitesse affichée alterne entre `220` et `-220`.

**Explication :** ce programme contient les quatre étapes de toute boucle de jeu Flutter, dans cet ordre exact.

D'abord la **mesure** : `(ecoule - _precedent).inMicroseconds / 1000000.0`. Le `Ticker` fournit un temps cumulé depuis le démarrage, pas un delta ; c'est à nous de faire la soustraction et de mémoriser la valeur précédente.

Ensuite la **protection** : `dt.clamp(0.0, 0.05)`. Sans cette ligne, un retour d'arrière-plan enverrait le carré à plusieurs milliers de pixels du bord.

Ensuite la **mise à jour** : `_x += _vitesse * dt`. La vitesse est en pixels par seconde, donc le carré parcourt bien 220 pixels en une seconde, quel que soit l'écran.

Enfin le **rendu**, via `setState`. Le `CustomPainter` ne fait que lire `_x` : il ne modifie rien. C'est la règle d'or.

Deux détails de robustesse. Le test `_largeur > 0` évite le rebond parasite lors de la toute première frame, avant que `LayoutBuilder` n'ait fourni une largeur. Et le repositionnement `_x = _largeur - cote` avant l'inversion évite que le carré ne reste coincé hors de l'écran : sans lui, si `_x` dépassait franchement le bord, l'inversion suivante pourrait le renvoyer dans le mauvais sens à la frame d'après.

---

### Correction 7

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const TempsApp());

class TempsApp extends StatelessWidget {
  const TempsApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14161C),
        body: SafeArea(child: EcranTemps()),
      ),
    );
  }
}

class EcranTemps extends StatefulWidget {
  const EcranTemps({super.key});

  @override
  State<EcranTemps> createState() => _EcranTempsState();
}

class _EcranTempsState extends State<EcranTemps>
    with SingleTickerProviderStateMixin {
  static const double cote = 40;

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _x = 0;
  double _vitesse = 220;
  double _largeur = 0;

  // Contrôle du temps.
  bool _enPause = false;
  double _timeScale = 1.0;

  double _tempsReel = 0;
  double _tempsJeu = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surFrame)..start();
  }

  void _surFrame(Duration ecoule) {
    double dtReel = (ecoule - _precedent).inMicroseconds / 1000000.0;
    _precedent = ecoule;
    dtReel = dtReel.clamp(0.0, 0.05);

    // Le temps réel avance TOUJOURS, même en pause.
    _tempsReel += dtReel;

    // Le temps du jeu dépend de l'échelle et de la pause.
    final double dtJeu = _enPause ? 0.0 : dtReel * _timeScale;
    _tempsJeu += dtJeu;

    _x += _vitesse * dtJeu;

    if (_x < 0) {
      _x = 0;
      _vitesse = -_vitesse;
    } else if (_largeur > 0 && _x + cote > _largeur) {
      _x = _largeur - cote;
      _vitesse = -_vitesse;
    }

    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  Widget _bouton(String texte, bool actif, VoidCallback action) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: ElevatedButton(
        onPressed: action,
        style: ElevatedButton.styleFrom(
          backgroundColor:
              actif ? const Color(0xFFE8B04B) : const Color(0xFF2A2F3A),
          foregroundColor: actif ? Colors.black : Colors.white70,
        ),
        child: Text(texte),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Expanded(
          child: LayoutBuilder(
            builder: (BuildContext context, BoxConstraints contraintes) {
              _largeur = contraintes.maxWidth;
              final double y = contraintes.maxHeight / 2 - cote / 2;

              return Stack(
                children: [
                  Positioned.fill(
                    child: CustomPaint(
                      painter: PeintreCarre(x: _x, y: y, cote: cote),
                    ),
                  ),
                  Positioned(
                    top: 12,
                    left: 12,
                    child: Text(
                      'temps réel : ${_tempsReel.toStringAsFixed(2)} s\n'
                      'temps jeu  : ${_tempsJeu.toStringAsFixed(2)} s\n'
                      'écart      : '
                      '${(_tempsReel - _tempsJeu).toStringAsFixed(2)} s\n'
                      'timeScale  : ${_timeScale.toStringAsFixed(2)}'
                      '${_enPause ? "   (PAUSE)" : ""}',
                      style: const TextStyle(
                        color: Colors.white70,
                        fontFamily: 'monospace',
                        fontSize: 14,
                      ),
                    ),
                  ),
                ],
              );
            },
          ),
        ),
        Container(
          color: const Color(0xFF1B1F27),
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
          child: Row(
            children: [
              _bouton('Ralenti', !_enPause && _timeScale == 0.25,
                  () => setState(() => _timeScale = 0.25)),
              _bouton('Normal', !_enPause && _timeScale == 1.0,
                  () => setState(() => _timeScale = 1.0)),
              _bouton('Accéléré', !_enPause && _timeScale == 3.0,
                  () => setState(() => _timeScale = 3.0)),
              _bouton(_enPause ? 'Reprendre' : 'Pause', _enPause,
                  () => setState(() => _enPause = !_enPause)),
            ],
          ),
        ),
      ],
    );
  }
}

class PeintreCarre extends CustomPainter {
  const PeintreCarre({
    required this.x,
    required this.y,
    required this.cote,
  });

  final double x;
  final double y;
  final double cote;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF14161C),
    );
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(x, y, cote, cote),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFFE8B04B),
    );
  }

  @override
  bool shouldRepaint(covariant PeintreCarre ancien) => ancien.x != x;
}
```

**Résultat :** le carré se déplace normalement. En appuyant sur « Ralenti », il rampe et le temps de jeu prend visiblement du retard sur le temps réel. En « Accéléré », il fuse et le temps de jeu dépasse le temps réel. En « Pause », le carré se fige et le temps de jeu cesse d'augmenter, tandis que le temps réel continue de défiler.

**Explication :** la clé de cet exercice tient en trois lignes :

```dart
_tempsReel += dtReel;
final double dtJeu = _enPause ? 0.0 : dtReel * _timeScale;
_tempsJeu += dtJeu;
```

Il existe désormais **deux temps** dans votre programme. Le temps réel est celui de l'horloge murale : il sert aux statistiques, aux FPS et aux animations d'interface, qui ne doivent jamais ralentir. Le temps du jeu est celui du monde simulé : c'est lui, et lui seul, que l'on transmet aux entités.

Remarquez surtout que le `Ticker` **n'est jamais arrêté**, même en pause. C'est volontaire. Si on l'arrêtait, la valeur `ecoule` reprendrait à sa dernière valeur au redémarrage et le premier `dt` après reprise serait faux. En laissant tourner le ticker et en mettant simplement `dtJeu` à zéro, l'interface reste vivante, le compteur de FPS continue de fonctionner, et la reprise est parfaitement propre.

L'écart affiché est instructif : en ralenti à 0,25, il grandit de 0,75 seconde par seconde écoulée. C'est la vérification arithmétique de `dtJeu = dtReel × timeScale`.

---

### Correction 8

```dart
void main() {
  const double pasFixe = 1 / 50; // 20 ms
  const double vitesse = 100; // pixels par seconde

  const List<double> dts = [
    0.021, 0.019, 0.045, 0.007, 0.009, 0.062,
    0.020, 0.011, 0.033, 0.018, 0.005, 0.041,
  ];

  double accumulateur = 0;
  double tempsReel = 0;
  int pasTotal = 0;
  double x = 0;

  print('frame  dt(ms)  pas  accu(ms)');
  print('------------------------------');

  for (int i = 0; i < dts.length; i++) {
    final double dt = dts[i];

    tempsReel += dt;
    accumulateur += dt;

    int pas = 0;
    while (accumulateur >= pasFixe) {
      x += vitesse * pasFixe; // TOUJOURS la même valeur
      accumulateur -= pasFixe;
      pas++;
      pasTotal++;
    }

    print('${(i + 1).toString().padLeft(3)} '
        '${(dt * 1000).toStringAsFixed(0).padLeft(7)} '
        '${pas.toString().padLeft(4)} '
        '${(accumulateur * 1000).toStringAsFixed(1).padLeft(9)}');
  }

  final double tempsSimule = pasTotal * pasFixe;

  print('');
  print('Pas fixes exécutés  : $pasTotal');
  print('Temps réel          : ${tempsReel.toStringAsFixed(3)} s');
  print('Temps simulé        : ${tempsSimule.toStringAsFixed(3)} s');
  print('Accumulateur restant: ${accumulateur.toStringAsFixed(3)} s');
  print('Somme               : '
      '${(tempsSimule + accumulateur).toStringAsFixed(3)} s');
  print('');
  print('Position x          : ${x.toStringAsFixed(1)} px');
  print('Position attendue   : '
      '${(vitesse * tempsReel).toStringAsFixed(1)} px');
}
```

**Résultat :**

```text
frame  dt(ms)  pas  accu(ms)
------------------------------
  1      21    1       1.0
  2      19    1       0.0
  3      45    2       5.0
  4       7    0      12.0
  5       9    1       1.0
  6      62    3       3.0
  7      20    1       3.0
  8      11    0      14.0
  9      33    2       7.0
 10      18    1       5.0
 11       5    0      10.0
 12      41    2      11.0

Pas fixes exécutés  : 14
Temps réel          : 0.291 s
Temps simulé        : 0.280 s
Accumulateur restant: 0.011 s
Somme               : 0.291 s

Position x          : 28.0 px
Position attendue   : 29.1 px
```

**Explication :** trois observations, dans l'ordre d'importance.

**Le nombre de pas varie, la valeur du pas jamais.** Les frames 4, 8 et 11 n'exécutent aucun pas : elles étaient trop courtes, leur temps est simplement mis en réserve. La frame 6, longue de 62 ms, en exécute trois d'un coup. Mais chaque appel reçoit toujours exactement `1/50` de seconde. Une simulation physique alimentée par cette boucle produira toujours le même résultat, sur toutes les machines. C'est ce que le pas variable ne peut pas garantir.

**Rien n'est perdu.** La dernière section du résultat le démontre : `0,280 + 0,011 = 0,291`, exactement le temps réel écoulé. L'accumulateur n'est pas une approximation, c'est une comptabilité exacte. Les 11 millisecondes restantes seront consommées à la frame suivante.

**L'écart de position est normal.** `x` vaut 28,0 pixels alors que `vitesse × tempsReel` donnerait 29,1. La différence de 1,1 pixel correspond très exactement aux 11 millisecondes encore dans l'accumulateur : `100 × 0,011 = 1,1`. La simulation n'est pas en retard sur la réalité, elle est en avance de zéro et en attente de 11 millisecondes. C'est justement cet écart, toujours compris entre 0 et un pas, que l'interpolation de l'exercice 9 va combler visuellement.

---

### Correction 9

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const InterpolationApp());

class InterpolationApp extends StatelessWidget {
  const InterpolationApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14161C),
        body: SafeArea(child: EcranInterpolation()),
      ),
    );
  }
}

class EcranInterpolation extends StatefulWidget {
  const EcranInterpolation({super.key});

  @override
  State<EcranInterpolation> createState() => _EcranInterpolationState();
}

class _EcranInterpolationState extends State<EcranInterpolation>
    with SingleTickerProviderStateMixin {
  // Simulation volontairement très lente pour rendre l'effet visible.
  static const double pasFixe = 1 / 8; // 8 pas par seconde
  static const double cote = 40;
  static const double vitesse = 200; // px/s

  late final Ticker _ticker;
  Duration _precedentTick = Duration.zero;

  // Deux états successifs du monde.
  double _xPrecedent = 0;
  double _xCourant = 0;
  double _sens = 1;

  double _accumulateur = 0;
  double _largeur = 0;
  int _pasTotal = 0;

  bool _interpolationActive = true;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surFrame)..start();
  }

  /// Un pas de simulation, toujours de la même durée.
  void _simulerUnPas() {
    _xPrecedent = _xCourant;
    _xCourant += vitesse * _sens * pasFixe;

    if (_xCourant < 0) {
      _xCourant = 0;
      _sens = -_sens;
      _xPrecedent = _xCourant; // pas d'interpolation à travers un rebond
    } else if (_largeur > 0 && _xCourant + cote > _largeur) {
      _xCourant = _largeur - cote;
      _sens = -_sens;
      _xPrecedent = _xCourant;
    }

    _pasTotal++;
  }

  void _surFrame(Duration ecoule) {
    double dt = (ecoule - _precedentTick).inMicroseconds / 1000000.0;
    _precedentTick = ecoule;
    dt = dt.clamp(0.0, 0.25);

    _accumulateur += dt;

    int securite = 0;
    while (_accumulateur >= pasFixe && securite < 5) {
      _simulerUnPas();
      _accumulateur -= pasFixe;
      securite++;
    }

    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // alpha : où en est-on entre l'état précédent et l'état courant ?
    final double alpha = (_accumulateur / pasFixe).clamp(0.0, 1.0);
    final double xInterpole =
        _xPrecedent + (_xCourant - _xPrecedent) * alpha;

    return Column(
      children: [
        Expanded(
          child: LayoutBuilder(
            builder: (BuildContext context, BoxConstraints contraintes) {
              _largeur = contraintes.maxWidth;

              return Stack(
                children: [
                  Positioned.fill(
                    child: CustomPaint(
                      painter: PeintreDeux(
                        xHaut: _xCourant,
                        xBas: _interpolationActive ? xInterpole : _xCourant,
                        cote: cote,
                      ),
                    ),
                  ),
                  Positioned(
                    top: 8,
                    left: 12,
                    child: Text(
                      'HAUT : position brute (8 mises à jour/s)\n'
                      'BAS  : ${_interpolationActive ? "interpolée" : "brute"}\n'
                      'alpha = ${alpha.toStringAsFixed(2)}   '
                      'pas = $_pasTotal',
                      style: const TextStyle(
                        color: Colors.white70,
                        fontFamily: 'monospace',
                        fontSize: 13,
                      ),
                    ),
                  ),
                ],
              );
            },
          ),
        ),
        Container(
          color: const Color(0xFF1B1F27),
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
          child: Row(
            children: [
              ElevatedButton(
                onPressed: () => setState(
                  () => _interpolationActive = !_interpolationActive,
                ),
                child: Text(
                  _interpolationActive
                      ? 'Désactiver l\'interpolation'
                      : 'Activer l\'interpolation',
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

class PeintreDeux extends CustomPainter {
  const PeintreDeux({
    required this.xHaut,
    required this.xBas,
    required this.cote,
  });

  final double xHaut;
  final double xBas;
  final double cote;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF14161C),
    );

    final double yHaut = size.height * 0.35;
    final double yBas = size.height * 0.65;

    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(xHaut, yHaut, cote, cote),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFFCF5C5C),
    );

    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(xBas, yBas, cote, cote),
        const Radius.circular(6),
      ),
      Paint()..color = const Color(0xFF6FBF73),
    );
  }

  @override
  bool shouldRepaint(covariant PeintreDeux ancien) =>
      ancien.xHaut != xHaut || ancien.xBas != xBas;
}
```

**Résultat :** le carré rouge du haut avance par sauts de 25 pixels, huit fois par seconde : le mouvement est visiblement saccadé, comme un vieux diaporama. Le carré vert du bas parcourt exactement la même trajectoire, aux mêmes instants, mais de façon parfaitement fluide. En appuyant sur le bouton, le carré vert redevient saccadé et se superpose au rouge.

**Explication :** les deux carrés sont simulés par la **même** boucle, aux **mêmes** huit pas par seconde. La seule différence est la position à laquelle on les dessine.

```text
  UNE MISE À JOUR TOUTES LES 125 ms, UN RENDU TOUTES LES 16,7 ms

  simulation :  [====== pas n ======][====== pas n+1 ======]
                xPrecedent          xCourant

  rendus     :   R  R  R  R  R  R  R  R
                 |  |  |  |  |  |  |  |
  alpha      :  .1 .2 .4 .5 .6 .8 .9 1.0

  position affichée = xPrecedent + (xCourant - xPrecedent) * alpha
```

La variable `alpha` mesure la progression à l'intérieur du pas courant : elle vaut 0 juste après une mise à jour et approche 1 juste avant la suivante. Elle se calcule très simplement : `accumulateur / pasFixe`, c'est-à-dire « quelle fraction d'un pas dort actuellement dans l'accumulateur ».

Trois précautions figurent dans le code. Le `clamp(0.0, 1.0)` sur `alpha` évite une extrapolation si l'accumulateur dépassait un pas complet. Le compteur `securite < 5` protège de la spirale de la mort vue en section 20.23. Enfin, lors d'un rebond, on écrit `_xPrecedent = _xCourant` : sans cela, l'interpolation ferait glisser le carré entre deux positions séparées par un demi-tour, ce qui produirait un éclair visuel désagréable.

Retenez la conséquence pratique : **la fluidité perçue n'exige pas de simuler plus souvent**. Simuler huit fois par seconde et interpoler donne un rendu aussi lisse qu'une simulation à 60 Hz, pour huit fois moins de calcul. C'est exactement la technique utilisée par les jeux en réseau, où le serveur n'envoie que 10 à 20 états par seconde.

---

### Correction 10

```dart
import 'dart:collection';
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

// ===========================================================
//  1. STATISTIQUES DE FRAME
// ===========================================================

class StatsFrames {
  StatsFrames({this.fenetre = 60});

  final int fenetre;
  final Queue<double> _dts = Queue<double>();
  double _somme = 0;

  void ajouter(double dt) {
    if (dt <= 0) return;
    _dts.addLast(dt);
    _somme += dt;
    while (_dts.length > fenetre) {
      _somme -= _dts.removeFirst();
    }
  }

  double get dtMoyen => _dts.isEmpty ? 0 : _somme / _dts.length;
  double get fps => dtMoyen > 0 ? 1 / dtMoyen : 0;

  /// La frame la plus lente de la fenêtre, en millisecondes.
  double get pireFrameMs {
    if (_dts.isEmpty) return 0;
    double pire = 0;
    for (final double d in _dts) {
      if (d > pire) pire = d;
    }
    return pire * 1000;
  }
}

// ===========================================================
//  2. LA CLASSE DE BASE D'UN JEU
// ===========================================================

abstract class Jeu {
  Size taille = Size.zero;

  void update(double dt);

  void dessiner(Canvas canvas, Size taille);
}

// ===========================================================
//  3. LE MOTEUR DE BOUCLE
// ===========================================================

class MoteurDeBoucle extends ChangeNotifier {
  MoteurDeBoucle({
    required this.jeu,
    required TickerProvider vsync,
    this.dtMax = 0.05,
  }) {
    _ticker = vsync.createTicker(_surFrame);
  }

  final Jeu jeu;
  final double dtMax;

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  final StatsFrames stats = StatsFrames(fenetre: 60);

  bool enPause = false;
  double timeScale = 1.0;

  double tempsReel = 0;
  double tempsJeu = 0;

  // Ralenti temporaire.
  double _bulletRestant = 0;
  double _bulletDuree = 0;
  static const double _bulletEchelle = 0.2;

  double get fps => stats.fps;
  double get pireFrameMs => stats.pireFrameMs;

  void demarrer() {
    if (!_ticker.isActive) _ticker.start();
  }

  void arreter() {
    if (_ticker.isActive) _ticker.stop();
  }

  /// Déclenche un ralenti qui revient progressivement à la normale.
  void bulletTime(double duree) {
    _bulletDuree = duree;
    _bulletRestant = duree;
    timeScale = _bulletEchelle;
  }

  void _surFrame(Duration ecoule) {
    // 1. dt réel, plafonné.
    double dtReel = (ecoule - _precedent).inMicroseconds / 1000000.0;
    _precedent = ecoule;
    dtReel = dtReel.clamp(0.0, dtMax);

    // 2. statistiques : toujours en temps réel.
    stats.ajouter(dtReel);
    tempsReel += dtReel;

    // 3. remontée progressive du ralenti (en temps réel, pas en temps jeu).
    if (_bulletRestant > 0) {
      _bulletRestant -= dtReel;
      if (_bulletRestant <= 0) {
        _bulletRestant = 0;
        timeScale = 1.0;
      } else {
        final double progression =
            1 - (_bulletRestant / _bulletDuree).clamp(0.0, 1.0);
        timeScale =
            _bulletEchelle + (1.0 - _bulletEchelle) * progression;
      }
    }

    // 4. temps du jeu.
    final double dtJeu = enPause ? 0.0 : dtReel * timeScale;
    tempsJeu += dtJeu;

    // 5. monde puis rendu.
    jeu.update(dtJeu);
    notifyListeners();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }
}

// ===========================================================
//  4. PEINTRE ET VUE
// ===========================================================

class PeintreDeJeu extends CustomPainter {
  PeintreDeJeu({required this.jeu, required Listenable repaint})
      : super(repaint: repaint);

  final Jeu jeu;

  @override
  void paint(Canvas canvas, Size size) {
    jeu.taille = size;
    jeu.dessiner(canvas, size);
  }

  @override
  bool shouldRepaint(covariant PeintreDeJeu ancien) => false;
}

class VueDeJeu extends StatefulWidget {
  const VueDeJeu({super.key, required this.jeu});

  final Jeu jeu;

  @override
  State<VueDeJeu> createState() => _VueDeJeuState();
}

class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  late final MoteurDeBoucle moteur;

  @override
  void initState() {
    super.initState();
    moteur = MoteurDeBoucle(jeu: widget.jeu, vsync: this);
    moteur.addListener(_rafraichirBarre);
    moteur.demarrer();
  }

  void _rafraichirBarre() {
    if (mounted) setState(() {});
  }

  @override
  void dispose() {
    moteur.removeListener(_rafraichirBarre);
    moteur.dispose();
    super.dispose();
  }

  Widget _bouton(String texte, VoidCallback action) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: ElevatedButton(onPressed: action, child: Text(texte)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Expanded(
          child: Stack(
            children: [
              Positioned.fill(
                child: CustomPaint(
                  painter: PeintreDeJeu(jeu: widget.jeu, repaint: moteur),
                ),
              ),
              Positioned(
                top: 10,
                left: 12,
                child: Text(
                  'FPS         : ${moteur.fps.toStringAsFixed(0)}\n'
                  'pire frame  : '
                  '${moteur.pireFrameMs.toStringAsFixed(1)} ms\n'
                  'temps réel  : ${moteur.tempsReel.toStringAsFixed(2)} s\n'
                  'temps jeu   : ${moteur.tempsJeu.toStringAsFixed(2)} s\n'
                  'timeScale   : ${moteur.timeScale.toStringAsFixed(2)}',
                  style: const TextStyle(
                    color: Colors.greenAccent,
                    fontFamily: 'monospace',
                    fontSize: 13,
                  ),
                ),
              ),
            ],
          ),
        ),
        Container(
          color: const Color(0xFF1B1F27),
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
          child: Row(
            children: [
              _bouton(
                moteur.enPause ? 'Reprendre' : 'Pause',
                () => setState(() => moteur.enPause = !moteur.enPause),
              ),
              _bouton('Coup critique', () => moteur.bulletTime(1.5)),
              _bouton('Normal', () => setState(() => moteur.timeScale = 1.0)),
            ],
          ),
        ),
      ],
    );
  }
}

// ===========================================================
//  5. LE JEU : DONJON DE DART
// ===========================================================

class Entite {
  Entite({
    required this.nom,
    required this.x,
    required this.y,
    required this.vitesse,
    required this.couleur,
    required this.taille,
  });

  final String nom;
  double x;
  double y;
  double vitesse; // pixels par seconde
  final Color couleur;
  final double taille;
}

class DonjonJeu extends Jeu {
  DonjonJeu() {
    entites = [
      Entite(
        nom: 'héros',
        x: 20,
        y: 160,
        vitesse: 160,
        couleur: const Color(0xFFE8B04B),
        taille: 30,
      ),
      Entite(
        nom: 'gobelin',
        x: 20,
        y: 230,
        vitesse: 70,
        couleur: const Color(0xFF6FBF73),
        taille: 24,
      ),
      Entite(
        nom: 'flèche',
        x: 20,
        y: 300,
        vitesse: 380,
        couleur: const Color(0xFFCF5C5C),
        taille: 14,
      ),
    ];
  }

  late final List<Entite> entites;

  @override
  void update(double dt) {
    for (final Entite e in entites) {
      e.x += e.vitesse * dt;

      if (e.x < 0) {
        e.x = 0;
        e.vitesse = -e.vitesse;
      } else if (taille.width > 0 && e.x + e.taille > taille.width) {
        e.x = taille.width - e.taille;
        e.vitesse = -e.vitesse;
      }
    }
  }

  @override
  void dessiner(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF14161C),
    );

    final Paint ligne = Paint()
      ..color = const Color(0xFF262A33)
      ..strokeWidth = 1;

    for (final Entite e in entites) {
      canvas.drawLine(
        Offset(0, e.y + e.taille + 6),
        Offset(size.width, e.y + e.taille + 6),
        ligne,
      );
      canvas.drawRRect(
        RRect.fromRectAndRadius(
          Rect.fromLTWH(e.x, e.y, e.taille, e.taille),
          const Radius.circular(4),
        ),
        Paint()..color = e.couleur,
      );
    }
  }
}

// ===========================================================
//  6. L'APPLICATION
// ===========================================================

void main() => runApp(DonjonApp());

class DonjonApp extends StatelessWidget {
  DonjonApp({super.key});

  final DonjonJeu jeu = DonjonJeu();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF14161C),
        body: SafeArea(child: VueDeJeu(jeu: jeu)),
      ),
    );
  }
}
```

**Résultat :** les trois entités du Donjon de Dart traversent l'écran à 160, 70 et 380 pixels par seconde. Le panneau d'information affiche les FPS lissés sur 60 frames, la durée de la pire frame de la fenêtre, les deux chronomètres et le facteur de temps courant. En appuyant sur « Coup critique », tout ralentit brutalement à 20 % de la vitesse normale, puis reprend progressivement sa vitesse en une seconde et demie ; la valeur de `timeScale` remonte visiblement de 0,20 vers 1,00 sous vos yeux.

**Explication :** trois points sont à retenir de cette version enrichie.

**Le ralenti se résorbe en temps réel, pas en temps de jeu.** Regardez la ligne `_bulletRestant -= dtReel;`. Si l'on avait utilisé `dtJeu`, le ralenti se serait auto-entretenu : plus le jeu ralentit, plus le temps de jeu avance lentement, donc plus le ralenti dure longtemps. Le joueur serait resté bloqué dans un ralenti quasi éternel. C'est une erreur classique, et elle n'est pas évidente à diagnostiquer.

**La progression est une interpolation linéaire.** `timeScale = 0,2 + (1 − 0,2) × progression`, avec `progression` qui va de 0 à 1. C'est exactement la même formule que l'interpolation de l'exercice 9. Vous retrouverez cette formule partout : fondus, barres de vie, déplacements de caméra, courbes d'animation.

**`pireFrameMs` est plus utile que les FPS moyens.** Un jeu qui affiche 59 FPS de moyenne mais dont la pire frame dure 90 ms est perçu comme saccadé par le joueur, alors que la moyenne semble excellente. Ce sont les frames isolées et longues qui se voient, pas la moyenne. C'est pour cela que les outils de profilage professionnels affichent des centiles plutôt que des moyennes. Gardez cet indicateur sous les yeux pendant tous les chapitres à venir.

Ce fichier est la version définitive du moteur pour la PARTIE 2A. Sauvegardez-le : le chapitre 21 remplacera les rectangles par de vrais dessins au `Canvas`, sans toucher une ligne du `MoteurDeBoucle`.

---

## Et maintenant ?

Vous savez désormais faire **battre le cœur** d'un jeu. Votre boucle tourne, elle mesure le temps qui passe, elle résiste aux pics de lag, elle sait ralentir, accélérer et se mettre en pause. Surtout, vous avez acquis le réflexe qui sépare un programme d'application d'un programme de jeu : **tout ce qui évolue dans le temps se multiplie par `dt`**.

Mais si vous relisez le mini-projet, vous verrez que nos entités sont encore de simples rectangles arrondis, posés sur des lignes horizontales. Nous avons un moteur de temps, il nous manque un moteur d'espace.

Le chapitre 21 comble ce manque. Vous y découvrirez le repère du `Canvas` de Flutter, où l'axe Y descend au lieu de monter, ce qui surprend tous les débutants. Vous apprendrez à dessiner des rectangles, des cercles, des lignes, des polygones et du texte, à les colorer et à les contourner avec la classe `Paint`, puis à les déplacer, les faire tourner et les mettre à l'échelle avec `translate`, `rotate` et `scale`. Vous verrez enfin `save` et `restore`, les deux méthodes qui empêchent une transformation de contaminer tout le reste du dessin.

À la fin du chapitre 21, le Donjon de Dart aura une vraie apparence : un sol, des murs, un héros orienté dans son sens de déplacement et une flèche qui pivote pendant son vol. Le tout continuera d'être animé par la boucle que vous venez d'écrire.

Rendez-vous au chapitre suivant : [21-PARTIE-2A—COORDONNÉES-CANVAS-ET-DESSIN-2D.md](./21-PARTIE-2A—COORDONNÉES-CANVAS-ET-DESSIN-2D.md)
