# PARTIE 2A — LES FONDAMENTAUX DU JEU 2D
# CHAPITRE 25 — CAMÉRA, MONDE ET NIVEAUX

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitre 24 — Collisions et hitboxes (AABB, résolution, hitbox vs sprite)
> **Ce que vous saurez faire à la fin :** construire un monde bien plus grand que l'écran, écrire une caméra maison qui suit le joueur en douceur, dessiner un décor en parallaxe, charger un niveau depuis une tilemap ou un fichier JSON, et enchaîner plusieurs salles de donjon.

---

## 25.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi un monde de jeu dépasse presque toujours la taille de l'écran ;
- distinguer sans hésiter **coordonnées monde** et **coordonnées écran** ;
- écrire la formule de conversion monde → écran et l'appliquer à la main ;
- écrire la formule inverse écran → monde et l'utiliser pour interpréter un clic ;
- concevoir une classe `Camera` maison, sans aucune bibliothèque externe ;
- déplacer toute la scène avec `canvas.translate()` encadré par `save()` / `restore()` ;
- centrer la caméra sur le joueur ;
- reconnaître le suivi brutal et expliquer pourquoi il fatigue l'œil ;
- lisser un suivi avec une interpolation linéaire (`lerp`, revu au chapitre 23) ;
- implémenter une **zone morte** pour que la caméra ne bouge pas au moindre pas ;
- ajouter un **look-ahead** qui montre ce qui arrive devant le joueur ;
- borner la caméra aux limites du niveau avec `clamp` ;
- appliquer un facteur de **zoom** et corriger les deux formules de conversion ;
- déclencher un **tremblement de caméra** à l'impact, avec amortissement ;
- pratiquer le **culling** : ne dessiner que ce qui est visible ;
- expliquer l'effet de **parallaxe** et le relier à la perception de profondeur ;
- implémenter un fond en plusieurs plans qui défilent à des vitesses différentes ;
- définir une **tilemap**, une **tuile** et une **taille de tuile** ;
- représenter un niveau avec une `List<List<int>>` (rappel du chapitre 06) ;
- dessiner une tilemap et n'en dessiner que la partie visible ;
- convertir une position monde en indices de tuile, et l'inverse ;
- faire collisionner un joueur avec une tilemap, axe par axe (rappel du chapitre 24) ;
- écrire un niveau lisible sous forme de `List<String>` ;
- charger un niveau depuis du JSON (rappel du chapitre 17) ;
- placer des objets de niveau : point d'apparition, portes, coffres, ennemis ;
- comprendre ce qu'est l'éditeur Tiled et à quoi ressemble un fichier `.tmx` ;
- enchaîner plusieurs niveaux et gérer la transition ;
- assembler un donjon de plusieurs salles avec caméra, parallaxe et tilemap.

---

## 25.1 — Le monde est plus grand que l'écran

Depuis le chapitre 20, tous vos programmes partagent une limite invisible : le héros vit dans un rectangle qui fait exactement la taille de la fenêtre. Quand il atteint le bord droit, vous le faites rebondir, ou vous le bloquez, ou vous l'enroulez de l'autre côté. Ce n'est pas un choix de game design, c'est un aveu : vous n'aviez pas encore de caméra.

Un vrai jeu ne fonctionne pas ainsi. Le Donjon de Dart que nous allons construire mesure 3 200 pixels de large et 1 600 pixels de haut. Votre téléphone en affiche peut-être 800 sur 400. Autrement dit, à tout instant, le joueur voit **un huitième** du niveau.

```text
  LE MONDE (3200 x 1600 pixels)

  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │   salle 1        salle 2          salle 3       salle 4      │
  │  ┌───────┐      ┌───────┐        ┌───────┐     ┌───────┐     │
  │  │  P    │══════│   G   │════════│  C    │═════│  BOSS │     │
  │  └───────┘      └───────┘        └───────┘     └───────┘     │
  │                                                              │
  │        ┌ ─ ─ ─ ─ ─ ─ ┐                                       │
  │        │   L'ÉCRAN   │  <- le joueur ne voit QUE ça          │
  │        │  800 x 400  │                                       │
  │        └ ─ ─ ─ ─ ─ ─ ┘                                       │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

Ce petit rectangle en pointillés, c'est la caméra. Elle se promène dans le monde et décide de ce qui est affiché.

### Pourquoi ne pas simplement faire un monde de la taille de l'écran ?

On pourrait. Certains jeux le font : *Pac-Man*, *Tetris*, *Space Invaders*, les échecs. Ils tiennent entièrement sur un écran. Mais dès que vous voulez :

- une exploration (le joueur découvre le niveau petit à petit) ;
- une progression spatiale (avancer vers la droite, c'est avancer dans l'histoire) ;
- un sentiment d'échelle (une salle du boss immense) ;
- un secret caché hors du champ de vision ;

alors il vous faut un monde plus grand que l'écran. Et donc une caméra.

### Ce que la caméra n'est pas

Voici la confusion la plus fréquente chez le débutant, et elle coûte des heures :

> **La caméra n'est pas un objet dessiné à l'écran.**
> **La caméra n'est pas un objet qui se déplace dans le monde comme le joueur.**

La caméra est un **changement de point de vue**. Concrètement, c'est une paire de nombres, `(camX, camY)`, qu'on soustrait à tout ce qu'on dessine. Rien de plus.

```text
  IMAGE MENTALE JUSTE

  Le monde ne bouge pas. La fenêtre se déplace au-dessus.

  monde ─────────────────────────────────────────────>
        [........#####...O...####...........G......]
                 ┌──────────────┐
                 │   fenêtre    │   camX = 9
                 └──────────────┘


  IMAGE MENTALE ÉQUIVALENTE (celle du code)

  La fenêtre ne bouge pas. Le monde glisse sous elle.

        ┌──────────────┐
        │ #####...O... │   on a décalé tout le monde de -9
        └──────────────┘
```

Les deux images sont vraies. La première est celle du joueur. La seconde est celle du `Canvas`. Votre travail de programmeur, c'est de passer de l'une à l'autre sans vous tromper de signe.

> **À retenir.** Une caméra, c'est une soustraction. Tout le chapitre découle de cette phrase, exactement comme le chapitre 20 découlait de « c'est le temps qui décide ».

---

## 25.2 — Coordonnées monde et coordonnées écran

Il existe désormais **deux repères** dans votre jeu. Confondre les deux est l'erreur numéro un du chapitre. Nommons-les proprement une fois pour toutes.

**Les coordonnées monde.** C'est la position réelle d'une chose dans le niveau. Le coffre est à `(2400, 300)`. Cette valeur ne change **jamais** quand la caméra bouge. Le coffre ne se déplace pas parce que le joueur regarde ailleurs.

**Les coordonnées écran.** C'est la position d'un pixel dans la fenêtre, entre `(0, 0)` en haut à gauche et `(largeur, hauteur)` en bas à droite. C'est le repère du chapitre 21, celui que `Canvas` comprend. Cette valeur change **tout le temps**, parce que la caméra bouge.

Voici le schéma à retenir.

```text
  ┌────────────────────────────────────────────────────────────────────┐
  │  MONDE  (repère absolu, le niveau)                                 │
  │  (0,0)                                                             │
  │   ┌───────────────────────────────────────────────────────────┐    │
  │   │                                                           │    │
  │   │           camX=600                                        │    │
  │   │           camY=200                                        │    │
  │   │              ├───── ÉCRAN 800 x 400 ─────┤                │    │
  │   │              ┌──────────────────────────┐                 │    │
  │   │      ────────│(0,0)écran                │                 │    │
  │   │              │                          │                 │    │
  │   │              │      @ héros             │                 │    │
  │   │              │      monde  (900, 340)   │                 │    │
  │   │              │      écran  (300, 140)   │                 │    │
  │   │              │                          │                 │    │
  │   │              └──────────────────────────┘                 │    │
  │   │                                    (800,400)écran         │    │
  │   │                                                           │    │
  │   └───────────────────────────────────────────────────────────┘    │
  │                                                     (3200,1600)    │
  └────────────────────────────────────────────────────────────────────┘

  Le héros est à (900, 340) dans le monde.
  La caméra est en (600, 200).
  Le héros apparaît donc en (900-600, 340-200) = (300, 140) à l'écran.
```

Prenez trente secondes sur ce schéma. Tout le reste du chapitre n'est que sa mise en équations.

### Un tableau pour ne plus se tromper

| Question | Repère | Exemple |
| --- | --- | --- |
| Où est le coffre dans le niveau ? | monde | `(2400, 300)` |
| Où dois-je peindre le coffre ? | écran | `(?, ?)` selon la caméra |
| Où le joueur a-t-il tapé du doigt ? | écran | `(412, 208)` |
| Quel objet le joueur a-t-il touché ? | monde | il faut convertir |
| Où est la hitbox du gobelin ? | monde | toujours en monde |
| Où placer la barre de vie du HUD ? | écran | `(20, 20)`, jamais en monde |

Cette dernière ligne est importante : le **HUD** (score, vies, minicarte) vit en coordonnées écran. Il ne doit surtout pas défiler avec la caméra. Nous y reviendrons à la section 25.6.

### Une convention de nommage

Adoptez dès maintenant une discipline de nommage. Elle vous évitera des bugs entiers.

```dart
// Toujours dire dans quel repère on est.
double joueurMondeX = 900;   // clair
double joueurEcranX = 300;   // clair

double joueurX = 900;        // ambigu, à éviter dans le code de rendu
```

Dans les classes métier (le joueur, le gobelin, le coffre), les positions sont **toujours** en monde. On ne stocke jamais une position écran dans une entité. La position écran est calculée au dernier moment, dans le rendu, et jetée.

> **À retenir.** Une entité connaît sa position monde. Elle ignore où elle est à l'écran, et elle ignore même s'il existe un écran.

---

## 25.3 — La formule de conversion monde → écran

Passons aux mathématiques. Elles tiennent en une soustraction.

Si la caméra désigne le **coin haut-gauche** de la zone visible, alors :

```text
  ecranX = mondeX - cameraX
  ecranY = mondeY - cameraY
```

C'est tout. Vérifions sur le schéma de la section précédente :

```text
  mondeX = 900, cameraX = 600  ->  ecranX = 900 - 600 = 300   OK
  mondeY = 340, cameraY = 200  ->  ecranY = 340 - 200 = 140   OK
```

Écrivons-la en Dart, et testons-la dans DartPad avant même de toucher à Flutter.

```dart
// Conversion monde -> écran, en Dart pur. Exécutable dans DartPad.

class Point2 {
  const Point2(this.x, this.y);
  final double x;
  final double y;

  @override
  String toString() => '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

/// Convertit une position monde en position écran.
Point2 mondeVersEcran(Point2 monde, Point2 camera) {
  return Point2(monde.x - camera.x, monde.y - camera.y);
}

void main() {
  const Point2 camera = Point2(600, 200);

  const Map<String, Point2> objets = <String, Point2>{
    'heros': Point2(900, 340),
    'gobelin': Point2(1250, 340),
    'coffre': Point2(640, 260),
    'torche': Point2(300, 180),
  };

  print('Camera au coin haut-gauche : $camera');
  print('');
  print('objet      monde            ecran');
  print('---------  ---------------  ---------------');

  objets.forEach((String nom, Point2 monde) {
    final Point2 ecran = mondeVersEcran(monde, camera);
    print('${nom.padRight(9)}  ${monde.toString().padRight(15)}  $ecran');
  });
}
```

**Résultat :**

```text
Camera au coin haut-gauche : (600.0, 200.0)

objet      monde            ecran
---------  ---------------  ---------------
heros      (900.0, 340.0)   (300.0, 140.0)
gobelin    (1250.0, 340.0)  (650.0, 140.0)
coffre     (640.0, 260.0)   (40.0, 60.0)
torche     (300.0, 180.0)   (-300.0, -20.0)
```

Observez la dernière ligne. La torche donne des coordonnées écran **négatives**. Ce n'est pas un bug : cela signifie simplement que la torche est hors du champ, à gauche et au-dessus de la fenêtre. C'est une information précieuse, et nous l'exploiterons au culling (section 25.15).

### Le piège du signe

Le débutant écrit une fois sur deux :

```dart
// FAUX
final double ecranX = monde.x + camera.x;
```

Le symptôme est immédiat et spectaculaire : quand le joueur avance vers la droite, le décor défile **dans le mauvais sens**, et le joueur semble accélérer deux fois plus vite. Si vous voyez cela, vous avez un signe inversé quelque part.

Retenez la phrase mnémotechnique :

> **La caméra avance, le monde recule.** D'où le moins.

### Et si la caméra désignait le centre ?

Certaines conventions placent la caméra au **centre** de l'écran plutôt qu'au coin. La formule devient alors :

```text
  ecranX = mondeX - cameraX + largeurEcran / 2
  ecranY = mondeY - cameraY + hauteurEcran / 2
```

Les deux conventions sont valides. Dans ce chapitre nous choisissons **le coin haut-gauche**, car les formules sont plus simples et le rectangle visible s'écrit directement `Rect.fromLTWH(camX, camY, largeur, hauteur)`. Nous verrons à la section 25.7 comment centrer facilement la caméra sur le joueur malgré cette convention.

> **Remarque.** Flame, à partir du chapitre 31, utilise la convention « centre ». Vous saurez alors traduire d'une convention à l'autre sans y penser.

---

## 25.4 — La formule inverse écran → monde

Cette section est indispensable, et elle est presque toujours oubliée par le débutant. Voici pourquoi.

Le joueur touche l'écran en `(412, 208)`. Ce sont des **coordonnées écran** : Flutter ne connaît que celles-là. Mais le coffre que le joueur essaie d'ouvrir est en `(2400, 300)` **en monde**. Si vous comparez directement `(412, 208)` à la hitbox du coffre, vous comparez des choux et des carottes. Le clic ne marchera jamais, sauf quand la caméra est par hasard en `(0, 0)` — ce qui explique le grand classique : « ça marche au début du niveau, et plus après ».

La formule inverse s'obtient en isolant `mondeX` :

```text
  ecranX = mondeX - cameraX
  donc
  mondeX = ecranX + cameraX
  mondeY = ecranY + cameraY
```

Encore une fois : une addition, rien de plus. Mais il faut y penser.

```dart
// Aller-retour monde <-> écran, en Dart pur.

class Point2 {
  const Point2(this.x, this.y);
  final double x;
  final double y;

  @override
  String toString() => '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

Point2 mondeVersEcran(Point2 monde, Point2 camera) =>
    Point2(monde.x - camera.x, monde.y - camera.y);

Point2 ecranVersMonde(Point2 ecran, Point2 camera) =>
    Point2(ecran.x + camera.x, ecran.y + camera.y);

void main() {
  const Point2 camera = Point2(1800, 120);

  // Le joueur touche l'écran ici.
  const Point2 doigt = Point2(412, 208);
  final Point2 cible = ecranVersMonde(doigt, camera);
  print('Doigt a l ecran : $doigt');
  print('Point du monde  : $cible');

  // Vérification : on refait le chemin dans l'autre sens.
  final Point2 retour = mondeVersEcran(cible, camera);
  print('Retour a l ecran: $retour');
  print('Aller-retour identique : ${retour.x == doigt.x && retour.y == doigt.y}');

  // Le coffre est-il touché ?
  const Point2 coffre = Point2(2200, 300);
  const double demiTaille = 24;
  final bool touche = (cible.x - coffre.x).abs() <= demiTaille &&
      (cible.y - coffre.y).abs() <= demiTaille;
  print('');
  print('Coffre en $coffre, touche : $touche');
}
```

**Résultat :**

```text
Doigt a l ecran : (412.0, 208.0)
Point du monde  : (2212.0, 328.0)
Retour a l ecran: (412.0, 208.0)
Aller-retour identique : true

Coffre en (2200.0, 300.0), touche : true
```

### Le test de l'aller-retour

Retenez cette technique de vérification : convertissez dans un sens, puis dans l'autre, et vérifiez que vous retombez sur la valeur de départ. Si l'aller-retour ne donne pas l'identité, l'une de vos deux formules est fausse. C'est un test à trente secondes qui vous économisera une soirée.

Nous en ferons un vrai test unitaire à l'exercice 2.

### Où cela sert-il concrètement ?

| Situation | Conversion nécessaire |
| --- | --- |
| Dessiner le héros | monde → écran |
| Dessiner une tuile de mur | monde → écran |
| Placer le score en haut à gauche | aucune (déjà en écran) |
| Interpréter un tap | écran → monde |
| Viser avec la souris | écran → monde |
| Faire glisser un objet posé | écran → monde |
| Placer un bouton de saut tactile | aucune (déjà en écran) |

Deux colonnes, deux mondes. Chaque fois que vous écrivez du code qui mélange les deux, arrêtez-vous et demandez-vous dans quel repère vous êtes.

> **À retenir.** Tout ce qui vient du joueur (doigt, souris) arrive en **écran** et doit être converti. Tout ce qui vient du jeu (entités, tuiles) vit en **monde** et doit être converti pour être dessiné.

---

## 25.5 — Une classe `Camera` maison

Assez de fonctions isolées. Une caméra est un objet : elle a un état (sa position), et des comportements (convertir, suivre, se borner). C'est un cas d'école pour la POO du chapitre 08.

Voici la première version, volontairement minimale. Nous l'enrichirons section après section jusqu'au mini-projet.

```dart
import 'dart:ui';

/// Caméra 2D. Sa position (x, y) désigne le COIN HAUT-GAUCHE
/// de la zone visible, exprimé en coordonnées monde.
class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    required this.largeur,
    required this.hauteur,
  });

  /// Coin haut-gauche de la vue, en coordonnées monde.
  double x;
  double y;

  /// Taille de la fenêtre, en pixels écran.
  double largeur;
  double hauteur;

  /// Le rectangle du monde actuellement visible.
  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);

  /// Monde -> écran.
  Offset versEcran(Offset monde) => Offset(monde.dx - x, monde.dy - y);

  /// Écran -> monde.
  Offset versMonde(Offset ecran) => Offset(ecran.dx + x, ecran.dy + y);

  /// Place le coin de la caméra de façon à centrer un point du monde.
  void centrerSur(Offset monde) {
    x = monde.dx - largeur / 2;
    y = monde.dy - hauteur / 2;
  }
}
```

Trois remarques sur ce code.

**`Rect get vue`** est un getter calculé (chapitre 10). Il ne stocke rien : il recompose le rectangle visible à la demande. C'est ce rectangle qui servira au culling.

**`versEcran` et `versMonde`** prennent et rendent des `Offset`, le type de Flutter pour un couple de `double`. Vous l'avez rencontré au chapitre 21 avec `canvas.drawCircle(Offset(x, y), r, paint)`.

**`largeur` et `hauteur` ne sont pas `final`.** La fenêtre peut changer de taille (rotation du téléphone, redimensionnement du navigateur). Nous mettrons ces deux champs à jour à chaque frame depuis un `LayoutBuilder`.

### Vérifier la classe sans Flutter

Un objet aussi simple se teste en console. C'est une bonne habitude : la logique de caméra est du calcul pur, elle n'a aucune raison d'exiger un écran.

```dart
import 'dart:ui';

class Camera {
  Camera({this.x = 0, this.y = 0, required this.largeur, required this.hauteur});

  double x;
  double y;
  double largeur;
  double hauteur;

  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset versEcran(Offset monde) => Offset(monde.dx - x, monde.dy - y);
  Offset versMonde(Offset ecran) => Offset(ecran.dx + x, ecran.dy + y);

  void centrerSur(Offset monde) {
    x = monde.dx - largeur / 2;
    y = monde.dy - hauteur / 2;
  }
}

void main() {
  final Camera camera = Camera(largeur: 800, hauteur: 400);

  print('Camera initiale : vue = ${camera.vue}');

  camera.centrerSur(const Offset(1600, 800));
  print('Apres centrerSur(1600, 800) :');
  print('  coin  = (${camera.x}, ${camera.y})');
  print('  vue   = ${camera.vue}');

  const Offset heros = Offset(1600, 800);
  print('  heros a l ecran = ${camera.versEcran(heros)}');

  const Offset tapEcran = Offset(0, 0);
  print('  coin haut-gauche de l ecran en monde = ${camera.versMonde(tapEcran)}');
}
```

**Résultat :**

```text
Camera initiale : vue = Rect.fromLTRB(0.0, 0.0, 800.0, 400.0)
Apres centrerSur(1600, 800) :
  coin  = (1200.0, 600.0)
  vue   = Rect.fromLTRB(1200.0, 600.0, 2000.0, 1000.0)
  heros a l ecran = Offset(400.0, 200.0)
  coin haut-gauche de l ecran en monde = Offset(1200.0, 600.0)
```

Le héros centré tombe bien en `(400, 200)`, c'est-à-dire au milieu d'une fenêtre de 800 sur 400. La classe est correcte.

> **Remarque.** `dart:ui` fournit `Offset` et `Rect` sans importer tout Flutter. Le code ci-dessus tourne dans DartPad en mode Flutter, et se compile aussi dans un test unitaire (`flutter test`).

---

## 25.6 — Déplacer la caméra avec `canvas.translate()`

Il y a deux manières d'appliquer une caméra au rendu.

**Méthode 1 — soustraire à la main.** Pour chaque objet, on calcule sa position écran avant de dessiner.

```dart
final Offset e = camera.versEcran(Offset(gobelin.x, gobelin.y));
canvas.drawRect(Rect.fromLTWH(e.dx, e.dy, 32, 32), peinture);
```

Cela fonctionne, mais il faut y penser **partout**. Le jour où vous oubliez une soustraction sur un seul objet, cet objet reste collé à l'écran pendant que le reste défile. Le bug est déroutant.

**Méthode 2 — translater le `Canvas`.** On décale une fois pour toutes le repère du canevas, puis on dessine tout en coordonnées **monde**, sans y penser.

```dart
canvas.save();
canvas.translate(-camera.x, -camera.y);

// Ici, on dessine en coordonnées MONDE. Directement.
canvas.drawRect(Rect.fromLTWH(gobelin.x, gobelin.y, 32, 32), peinture);

canvas.restore();
```

C'est la méthode que nous retenons. Elle est plus courte, moins fragile, et elle correspond à ce que fait Flame en interne.

### Pourquoi `-camera.x` et non `+camera.x` ?

Même raison qu'à la section 25.3, vue autrement. `canvas.translate(dx, dy)` décale **tout ce qui sera dessiné ensuite** de `(dx, dy)`. Si la caméra est en `x = 600` et que l'on veut que le point monde 600 se retrouve au bord gauche de l'écran (écran 0), il faut décaler de `-600`.

```text
  canvas.translate(-600, 0)

  monde   600 ────────────> devient écran 0
  monde   900 ────────────> devient écran 300
  monde  1400 ────────────> devient écran 800  (bord droit)
```

### `save()` et `restore()` : la paire sacrée

`canvas.translate()` modifie le repère **de façon persistante**. Si vous ne le remettez pas en place, tout ce que vous dessinez ensuite sera décalé — y compris le HUD.

- `canvas.save()` : mémorise l'état actuel du repère (et du clip).
- `canvas.restore()` : revient exactement à cet état.

C'est une **pile**. Vous pouvez empiler plusieurs `save()`, tant que chacun a son `restore()`.

```text
  DÉROULÉ D'UNE FRAME

  paint(canvas, size)
    │
    ├─ dessiner le ciel                     <- repère écran
    ├─ dessiner le parallaxe                <- repère écran (décalage partiel)
    │
    ├─ canvas.save()                        ┐
    ├─ canvas.translate(-camX, -camY)       │  repère MONDE
    │    ├─ dessiner les tuiles             │
    │    ├─ dessiner les coffres            │
    │    ├─ dessiner les gobelins           │
    │    └─ dessiner le héros               │
    ├─ canvas.restore()                     ┘
    │
    └─ dessiner le HUD (vies, score)        <- repère écran de nouveau
```

Oubliez le `restore()`, et votre score part se promener dans le donjon. C'est l'erreur numéro un du chapitre, et elle figure en tête du tableau des erreurs fréquentes.

### Premier programme complet avec caméra

Voici un `main.dart` complet et copiable. Un monde de 2 400 sur 1 200 pixels, une grille de repérage, quelques objets, et une caméra que vous déplacez en faisant glisser le doigt (ou la souris).

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AppliCamera());

class AppliCamera extends StatelessWidget {
  const AppliCamera({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14101E),
        body: EcranCamera(),
      ),
    );
  }
}

/// Caméra 2D : (x, y) = coin haut-gauche de la vue, en coordonnées monde.
class Camera {
  Camera({this.x = 0, this.y = 0, this.largeur = 0, this.hauteur = 0});

  double x;
  double y;
  double largeur;
  double hauteur;

  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset versEcran(Offset monde) => Offset(monde.dx - x, monde.dy - y);
  Offset versMonde(Offset ecran) => Offset(ecran.dx + x, ecran.dy + y);

  void centrerSur(Offset monde) {
    x = monde.dx - largeur / 2;
    y = monde.dy - hauteur / 2;
  }

  /// Applique la caméra au canevas. À refermer avec [retirer].
  void appliquer(Canvas canvas) {
    canvas.save();
    canvas.translate(-x, -y);
  }

  void retirer(Canvas canvas) => canvas.restore();
}

/// Un objet du monde, réduit à une position, une taille et une couleur.
class ObjetMonde {
  const ObjetMonde(this.nom, this.position, this.taille, this.couleur);

  final String nom;
  final Offset position;
  final double taille;
  final Color couleur;

  Rect get boite => Rect.fromLTWH(position.dx, position.dy, taille, taille);
}

class EcranCamera extends StatefulWidget {
  const EcranCamera({super.key});

  @override
  State<EcranCamera> createState() => _EcranCameraState();
}

class _EcranCameraState extends State<EcranCamera> {
  static const double largeurMonde = 2400;
  static const double hauteurMonde = 1200;

  final Camera _camera = Camera(x: 0, y: 0);

  Offset? _dernierTapMonde;

  final List<ObjetMonde> _objets = const <ObjetMonde>[
    ObjetMonde('depart', Offset(120, 200), 40, Color(0xFF4EC9B0)),
    ObjetMonde('coffre', Offset(900, 520), 48, Color(0xFFD7A24C)),
    ObjetMonde('gobelin', Offset(1500, 300), 36, Color(0xFF7FB84E)),
    ObjetMonde('potion', Offset(1980, 780), 28, Color(0xFFC94F5E)),
    ObjetMonde('boss', Offset(2200, 950), 80, Color(0xFF8E5AD1)),
  ];

  void _glisser(DragUpdateDetails d) {
    setState(() {
      // On fait glisser le MONDE : la caméra va dans le sens opposé.
      _camera.x -= d.delta.dx;
      _camera.y -= d.delta.dy;
      _borner();
    });
  }

  void _taper(TapDownDetails d) {
    setState(() {
      _dernierTapMonde = _camera.versMonde(d.localPosition);
    });
  }

  void _borner() {
    _camera.x = _camera.x.clamp(0.0, largeurMonde - _camera.largeur);
    _camera.y = _camera.y.clamp(0.0, hauteurMonde - _camera.hauteur);
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints contraintes) {
        _camera.largeur = contraintes.maxWidth;
        _camera.hauteur = contraintes.maxHeight;
        return GestureDetector(
          behavior: HitTestBehavior.opaque,
          onPanUpdate: _glisser,
          onTapDown: _taper,
          child: CustomPaint(
            painter: PeintreCamera(
              camera: _camera,
              objets: _objets,
              largeurMonde: largeurMonde,
              hauteurMonde: hauteurMonde,
              dernierTapMonde: _dernierTapMonde,
            ),
            size: Size.infinite,
          ),
        );
      },
    );
  }
}

class PeintreCamera extends CustomPainter {
  PeintreCamera({
    required this.camera,
    required this.objets,
    required this.largeurMonde,
    required this.hauteurMonde,
    required this.dernierTapMonde,
  });

  final Camera camera;
  final List<ObjetMonde> objets;
  final double largeurMonde;
  final double hauteurMonde;
  final Offset? dernierTapMonde;

  void _texte(Canvas canvas, String contenu, Offset position,
      {Color couleur = const Color(0xFFD8D8E8), double taille = 13}) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: TextStyle(color: couleur, fontSize: taille),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  void _dessinerGrilleMonde(Canvas canvas) {
    final Paint trait = Paint()
      ..color = const Color(0xFF2A2340)
      ..strokeWidth = 1;

    for (double x = 0; x <= largeurMonde; x += 100) {
      canvas.drawLine(Offset(x, 0), Offset(x, hauteurMonde), trait);
    }
    for (double y = 0; y <= hauteurMonde; y += 100) {
      canvas.drawLine(Offset(0, y), Offset(largeurMonde, y), trait);
    }

    // Bordure du monde.
    canvas.drawRect(
      Rect.fromLTWH(0, 0, largeurMonde, hauteurMonde),
      Paint()
        ..color = const Color(0xFF6A5A99)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 4,
    );
  }

  @override
  void paint(Canvas canvas, Size size) {
    // --- REPÈRE ÉCRAN : le fond ---
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF14101E),
    );

    // --- REPÈRE MONDE ---
    camera.appliquer(canvas);

    _dessinerGrilleMonde(canvas);

    for (final ObjetMonde objet in objets) {
      canvas.drawRect(objet.boite, Paint()..color = objet.couleur);
      _texte(canvas, objet.nom,
          Offset(objet.position.dx, objet.position.dy - 18),
          couleur: objet.couleur, taille: 12);
    }

    final Offset? tap = dernierTapMonde;
    if (tap != null) {
      canvas.drawCircle(
        tap,
        14,
        Paint()
          ..color = const Color(0xFFFFFFFF)
          ..style = PaintingStyle.stroke
          ..strokeWidth = 2,
      );
    }

    camera.retirer(canvas);
    // --- RETOUR AU REPÈRE ÉCRAN : le HUD ---

    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 300, 106),
      Paint()..color = const Color(0xCC000000),
    );
    _texte(canvas, 'DONJON DE DART — camera libre', const Offset(12, 10));
    _texte(
      canvas,
      'camera : (${camera.x.toStringAsFixed(0)}, '
      '${camera.y.toStringAsFixed(0)})',
      const Offset(12, 32),
    );
    _texte(
      canvas,
      'vue    : ${camera.largeur.toStringAsFixed(0)} x '
      '${camera.hauteur.toStringAsFixed(0)}',
      const Offset(12, 52),
    );
    _texte(
      canvas,
      tap == null
          ? 'tapez pour convertir un point'
          : 'tap monde : (${tap.dx.toStringAsFixed(0)}, '
              '${tap.dy.toStringAsFixed(0)})',
      const Offset(12, 78),
      couleur: const Color(0xFF4EC9B0),
    );
  }

  @override
  bool shouldRepaint(PeintreCamera oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART — camera libre
camera : (0, 0)
vue    : 800 x 400
tapez pour convertir un point

Une grille violette de 100 pixels de pas remplit le monde.
Cinq carrés colorés portent leur nom : depart, coffre, gobelin, potion, boss.
En faisant glisser le doigt vers la gauche, la caméra avance vers la droite :
la grille défile, le coffre puis le gobelin entrent dans le champ.
Le panneau noir en haut à gauche NE BOUGE PAS : il est dessiné après restore().
Chaque tap affiche le point du monde correspondant, et pose un cercle blanc
exactement sous le doigt, quelle que soit la position de la caméra.
```

Remarquez trois choses dans ce programme.

1. **Le fond et le HUD sont dessinés hors du bloc `appliquer` / `retirer`.** Ils sont donc en coordonnées écran et ne défilent pas.
2. **La grille, les objets et le marqueur de tap sont dessinés en coordonnées monde**, sans aucune soustraction manuelle. C'est tout le bénéfice de `translate`.
3. **Le cercle blanc reste sous le doigt même après avoir déplacé la caméra.** C'est la preuve visuelle que `versMonde` est correcte. Si vous supprimez la conversion et stockez le tap en écran, le cercle se décalera dès que vous ferez glisser la vue.

> **À retenir.** Un bloc `save()` / `translate()` / … / `restore()` par caméra. Ce qui est dedans est en monde, ce qui est dehors est en écran.

---

## 25.7 — Centrer la caméra sur le joueur

Une caméra qu'on déplace au doigt, c'est bien pour un jeu de stratégie. Pour un jeu de plateforme ou d'action, la caméra doit suivre le héros toute seule.

La règle la plus simple : **le héros est toujours au centre de l'écran.**

```text
  ┌──────────────────────────────────┐
  │                                  │
  │                                  │
  │               @                  │   le héros ne quitte jamais
  │            (centre)              │   le point central
  │                                  │
  │                                  │
  └──────────────────────────────────┘
```

Avec notre convention « coin haut-gauche », centrer signifie reculer d'une demi-fenêtre :

```dart
camera.x = joueur.x - camera.largeur / 2;
camera.y = joueur.y - camera.hauteur / 2;
```

C'est exactement la méthode `centrerSur` écrite à la section 25.5. Vérifions le calcul à la main :

```text
  joueur en monde       : (1600, 800)
  fenêtre               : 800 x 400
  camera.x = 1600 - 400 = 1200
  camera.y =  800 - 200 =  600

  position écran du joueur = 1600 - 1200 = 400  -> milieu horizontal   OK
                             800 -  600 = 200  -> milieu vertical     OK
```

### Centrer sur le centre du héros, pas sur son coin

Attention à un détail qui gêne visuellement. Si le héros est un rectangle de 32 sur 48 dont `(x, y)` désigne le coin haut-gauche, alors centrer sur `(x, y)` place son **coin** au milieu de l'écran, pas son corps. Le personnage paraît décalé en haut à gauche.

Il faut centrer sur le centre de sa hitbox :

```dart
final Offset centreHeros = Offset(
  heros.x + heros.largeur / 2,
  heros.y + heros.hauteur / 2,
);
camera.centrerSur(centreHeros);
```

Le décalage n'est que de 16 pixels horizontalement, mais l'œil le voit. Prenez l'habitude d'ajouter un getter `centre` à vos entités.

```dart
class Heros {
  Heros({required this.x, required this.y});

  double x;
  double y;
  final double largeur = 32;
  final double hauteur = 48;

  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset get centre => Offset(x + largeur / 2, y + hauteur / 2);
}
```

### Le point d'ancrage vertical

En 2D à défilement latéral, on ne centre pas toujours verticalement sur le centre du héros. Beaucoup de jeux visent un point situé **un peu au-dessus** du personnage, pour montrer davantage le ciel que le sol.

```dart
// On vise 40 pixels au-dessus du centre du héros.
camera.centrerSur(heros.centre - const Offset(0, 40));
```

C'est un réglage de confort, pas une règle. Testez avec votre niveau : si le joueur ne voit pas où il va tomber, descendez le point de visée ; s'il ne voit pas les plateformes au-dessus, remontez-le.

> **Remarque.** Ce point visé s'appelle la **cible** de la caméra. Nous allons lui donner un nom dans le code dès la section 25.9, car toutes les améliorations qui suivent (lissage, zone morte, look-ahead) consistent à modifier la cible ou la façon de la rejoindre.

---

## 25.8 — Le suivi brutal et pourquoi il donne la nausée

Le code de la section précédente s'appelle un **suivi brutal** (ou *hard follow*, *snap follow*). À chaque frame, la caméra se téléporte exactement sur la cible.

```dart
void mettreAJour(double dt) {
  camera.centrerSur(heros.centre); // téléportation immédiate
}
```

Techniquement, c'est correct. Visuellement, c'est souvent désagréable. Voici pourquoi, en trois raisons distinctes.

### Raison 1 — le décor devient le seul objet en mouvement

Le héros est cloué au centre de l'écran. Il ne bouge **jamais** d'un pixel à l'écran. C'est donc le monde entier qui glisse. Le cerveau du joueur perçoit alors un décor instable plutôt qu'un personnage qui se déplace. Sur un jeu de plateforme rapide, c'est fatigant au bout de quelques minutes.

### Raison 2 — la moindre secousse du héros secoue tout l'écran

Un héros qui saute monte et descend. En suivi brutal, c'est l'écran entier qui monte et descend à chaque saut, à pleine amplitude.

```text
  SUIVI BRUTAL PENDANT UN SAUT

  hauteur du héros (monde)     ─╮      ╭─
                                ╰──────╯

  ce que voit le joueur :  tout le décor fait le mouvement inverse,
                           de la même amplitude, à chaque saut.
```

Sur une série de petits sauts, l'écran devient un trampoline. C'est le principal reproche fait aux caméras naïves.

### Raison 3 — les changements de direction sont instantanés

Le héros marche à droite, il fait demi-tour. La caméra fait demi-tour dans la même frame, sans transition. Il n'y a aucune inertie, donc aucune sensation de poids. Le résultat semble « mécanique », terme que vous entendrez souvent dans les retours de test.

### Le mesurer plutôt que le sentir

Comparons dans un tableau ce que fait la caméra sur quelques frames, quand le héros oscille légèrement autour de x = 1000 (une marche avec un petit rebond).

| Frame | Héros x (monde) | Caméra x (suivi brutal) | Déplacement caméra |
| --- | --- | --- | --- |
| 1 | 1000 | 600 | — |
| 2 | 1006 | 606 | +6 |
| 3 | 1003 | 603 | -3 |
| 4 | 1011 | 611 | +8 |
| 5 | 1007 | 607 | -4 |
| 6 | 1016 | 616 | +9 |

La colonne de droite change de signe trois fois en cinq frames. Chacun de ces changements est un micro-recul du décor entier. Multiplié par 60 frames par seconde, c'est ce qu'on appelle du **jitter** de caméra.

### Quand le suivi brutal est-il un bon choix ?

Il ne faut pas le condamner sans nuance. Il est parfait pour :

- les jeux à défilement automatique (shoot'em up vertical) ;
- les jeux au tour par tour, où le héros ne bouge que par sauts d'une case ;
- une caméra pilotée directement par le joueur (jeu de stratégie) ;
- le débogage : quand on cherche un bug, on veut voir exactement où est l'entité.

Il est mauvais pour un jeu de plateforme ou d'action à mouvement continu, c'est-à-dire précisément le Donjon de Dart.

> **À retenir.** Le suivi brutal n'est pas faux, il est **dur**. Les trois sections suivantes ajoutent successivement de la douceur (25.9), de la tolérance (25.10) et de l'anticipation (25.11).

---

## 25.9 — Le suivi lissé (`lerp`, rappel du chapitre 23)

La solution au suivi brutal tient en une ligne. Au lieu de sauter sur la cible, la caméra **s'en rapproche d'un pourcentage à chaque frame**.

C'est l'interpolation linéaire, le `lerp` du chapitre 23 :

```text
  lerp(a, b, t) = a + (b - a) * t
```

Avec `t = 0`, on reste en `a`. Avec `t = 1`, on saute en `b`. Avec `t = 0,1`, on parcourt un dixième du chemin.

```dart
double lerp(double a, double b, double t) => a + (b - a) * t;
```

Appliqué à la caméra :

```dart
camera.x = lerp(camera.x, cibleX, 0.1);
camera.y = lerp(camera.y, cibleY, 0.1);
```

À chaque frame, la caméra comble 10 % de l'écart restant. Elle n'atteint jamais mathématiquement la cible, mais elle s'en approche exponentiellement vite.

```text
  RATTRAPAGE AVEC t = 0,2 (écart initial 100 px)

  frame 0   ████████████████████  100.0
  frame 1   ████████████████       80.0
  frame 2   ████████████           64.0
  frame 3   ██████████             51.2
  frame 4   ████████               41.0
  frame 5   ██████                 32.8
  frame 6   █████                  26.2
  frame 7   ████                   21.0
  ...
  frame 20  ░                       1.2
```

### Le piège : `lerp` avec un `t` constant dépend des FPS

Cette écriture a un défaut sérieux, et c'est exactement le sujet du chapitre 20. Avec `t = 0,1` par frame :

- à 60 FPS, la caméra fait 60 pas de 10 % par seconde ;
- à 30 FPS, elle n'en fait que 30.

La caméra suit donc **deux fois plus lentement** sur une machine lente. Ce n'est pas acceptable : la sensation de jeu change avec le matériel.

La correction consiste à exprimer le lissage en fonction de `dt`. La formule correcte, mathématiquement, est :

```text
  t = 1 - exp(-vitesse * dt)
```

Elle garantit que le rattrapage par seconde est le même quel que soit le nombre de frames.

```dart
import 'dart:math' as math;

/// Facteur de lerp indépendant du framerate.
/// [vitesse] est en "unités par seconde" : plus c'est grand, plus c'est rapide.
double facteurLissage(double vitesse, double dt) {
  return 1 - math.exp(-vitesse * dt);
}
```

Vérifions numériquement.

```dart
import 'dart:math' as math;

double facteurLissage(double vitesse, double dt) => 1 - math.exp(-vitesse * dt);

double lerp(double a, double b, double t) => a + (b - a) * t;

/// Simule 1 seconde de rattrapage à un framerate donné.
double simuler(double fps, double vitesse) {
  final double dt = 1 / fps;
  double camera = 0;
  const double cible = 100;
  for (int i = 0; i < fps.round(); i++) {
    camera = lerp(camera, cible, facteurLissage(vitesse, dt));
  }
  return camera;
}

/// La même chose avec un t constant : dépendant du framerate.
double simulerNaif(double fps, double t) {
  double camera = 0;
  const double cible = 100;
  for (int i = 0; i < fps.round(); i++) {
    camera = lerp(camera, cible, t);
  }
  return camera;
}

void main() {
  print('Apres 1 seconde, ecart initial = 100 px, cible = 100');
  print('');
  print('VERSION CORRECTE  (t = 1 - exp(-6 * dt))');
  for (final double fps in <double>[30, 60, 120]) {
    print('  ${fps.toStringAsFixed(0).padLeft(3)} FPS -> '
        'camera = ${simuler(fps, 6).toStringAsFixed(2)}');
  }
  print('');
  print('VERSION NAIVE     (t = 0.1 par frame)');
  for (final double fps in <double>[30, 60, 120]) {
    print('  ${fps.toStringAsFixed(0).padLeft(3)} FPS -> '
        'camera = ${simulerNaif(fps, 0.1).toStringAsFixed(2)}');
  }
}
```

**Résultat :**

```text
Apres 1 seconde, ecart initial = 100 px, cible = 100

VERSION CORRECTE  (t = 1 - exp(-6 * dt))
   30 FPS -> camera = 99.75
   60 FPS -> camera = 99.75
  120 FPS -> camera = 99.75

VERSION NAIVE     (t = 0.1 par frame)
   30 FPS -> camera = 95.76
   60 FPS -> camera = 99.82
  120 FPS -> camera = 100.00
```

La version correcte donne **exactement le même résultat** aux trois framerates. La version naïve, elle, varie de 95,8 à 100. Sur une seconde ce n'est pas dramatique, mais sur un mouvement continu la différence de « feeling » est nette.

### Choisir la vitesse de lissage

| Vitesse | Effet | Usage |
| --- | --- | --- |
| 2 | très mou, la caméra traîne | scènes contemplatives, cinématiques |
| 5 | doux, légère traîne perceptible | exploration, plateforme calme |
| 8 | réactif, à peine adouci | action, plateforme rapide |
| 15 | presque brutal | combats, boss |
| 1000 | équivalent au suivi brutal | débogage |

Il n'y a pas de bonne valeur absolue. Essayez avec votre niveau, et notez la valeur choisie dans un `static const` bien nommé plutôt que de laisser un nombre magique au milieu du code.

### La caméra lissée dans la classe

Enrichissons `Camera` :

```dart
import 'dart:math' as math;
import 'dart:ui';

class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    this.largeur = 0,
    this.hauteur = 0,
    this.vitesseSuivi = 6,
  });

  double x;
  double y;
  double largeur;
  double hauteur;

  /// Vitesse de rattrapage, en unités par seconde.
  double vitesseSuivi;

  /// Point du monde que la caméra doit centrer.
  Offset cible = Offset.zero;

  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset versEcran(Offset m) => Offset(m.dx - x, m.dy - y);
  Offset versMonde(Offset e) => Offset(e.dx + x, e.dy + y);

  /// Coin haut-gauche idéal pour centrer [cible].
  double get _cibleCoinX => cible.dx - largeur / 2;
  double get _cibleCoinY => cible.dy - hauteur / 2;

  /// Se place immédiatement sur la cible (utile au démarrage d'un niveau).
  void placerImmediatement() {
    x = _cibleCoinX;
    y = _cibleCoinY;
  }

  void mettreAJour(double dt) {
    final double t = 1 - math.exp(-vitesseSuivi * dt);
    x += (_cibleCoinX - x) * t;
    y += (_cibleCoinY - y) * t;
  }

  void appliquer(Canvas canvas) {
    canvas.save();
    canvas.translate(-x, -y);
  }

  void retirer(Canvas canvas) => canvas.restore();
}
```

Deux nouveautés utiles.

**`cible`** est un `Offset` en coordonnées monde. Le jeu écrit `camera.cible = heros.centre;` et n'a plus à connaître la mécanique du lissage.

**`placerImmediatement()`** évite un défaut classique : au tout premier chargement, la caméra part de `(0, 0)` et « vole » jusqu'au héros pendant une seconde. Ce n'est joli que si c'est voulu. Appelez `placerImmediatement()` après avoir défini la cible initiale.

> **À retenir.** `x += (cible - x) * t` avec `t = 1 - exp(-vitesse * dt)`. Trois lignes, et votre caméra ne donne plus la nausée.

---

## 25.10 — La zone morte (dead zone)

Le lissage adoucit les mouvements, mais il ne les supprime pas. Si le héros fait un pas de 3 pixels, la caméra bouge encore un peu. Sur un jeu où le personnage oscille (respiration, petits sauts, recul d'attaque), le décor n'est jamais parfaitement immobile.

La **zone morte** répond à ce problème par une idée simple :

> Tant que le héros reste à l'intérieur d'un rectangle central, la caméra ne bouge pas du tout.

```text
  LA ZONE MORTE

  ┌──────────────── ÉCRAN ─────────────────┐
  │                                        │
  │        ┌───── zone morte ─────┐        │
  │        │                      │        │
  │        │        @ héros       │        │   la caméra NE BOUGE PAS
  │        │                      │        │
  │        └──────────────────────┘        │
  │                                        │
  └────────────────────────────────────────┘


  ┌──────────────── ÉCRAN ─────────────────┐
  │                                        │
  │        ┌───── zone morte ─────┐        │
  │        │                      │ @      │   le héros est SORTI
  │        │                      │→→→     │   la caméra pousse
  │        └──────────────────────┘        │   juste ce qu'il faut
  │                                        │
  └────────────────────────────────────────┘
```

### Le principe de calcul

On travaille en coordonnées écran. On calcule la position écran du héros, puis on compare aux bords de la zone morte.

- Si `ecranX > bordDroit`, la caméra doit avancer de `ecranX - bordDroit`.
- Si `ecranX < bordGauche`, la caméra doit reculer de `bordGauche - ecranX`.
- Sinon, elle ne bouge pas.

Même chose sur l'axe vertical.

```dart
/// Ajuste la caméra pour que [cibleMonde] reste dans la zone morte.
/// [marge] est la distance entre le bord de l'écran et le bord de la zone.
void appliquerZoneMorte(Camera camera, Offset cibleMonde, Size marge) {
  final Offset e = camera.versEcran(cibleMonde);

  final double gauche = marge.width;
  final double droite = camera.largeur - marge.width;
  final double haut = marge.height;
  final double bas = camera.hauteur - marge.height;

  if (e.dx > droite) {
    camera.x += e.dx - droite;
  } else if (e.dx < gauche) {
    camera.x -= gauche - e.dx;
  }

  if (e.dy > bas) {
    camera.y += e.dy - bas;
  } else if (e.dy < haut) {
    camera.y -= haut - e.dy;
  }
}
```

### Simuler la zone morte en console

Vérifions le comportement sur une marche vers la droite.

```dart
import 'dart:ui';

class Camera {
  Camera({this.x = 0, this.y = 0, required this.largeur, required this.hauteur});
  double x;
  double y;
  double largeur;
  double hauteur;
  Offset versEcran(Offset m) => Offset(m.dx - x, m.dy - y);
}

void appliquerZoneMorte(Camera camera, Offset cible, Size marge) {
  final Offset e = camera.versEcran(cible);
  final double gauche = marge.width;
  final double droite = camera.largeur - marge.width;
  final double haut = marge.height;
  final double bas = camera.hauteur - marge.height;

  if (e.dx > droite) {
    camera.x += e.dx - droite;
  } else if (e.dx < gauche) {
    camera.x -= gauche - e.dx;
  }
  if (e.dy > bas) {
    camera.y += e.dy - bas;
  } else if (e.dy < haut) {
    camera.y -= haut - e.dy;
  }
}

void main() {
  final Camera camera = Camera(x: 0, y: 0, largeur: 400, hauteur: 200);
  const Size marge = Size(150, 60); // zone morte de 100 x 80 au centre

  print('heros.x   ecran.x   camera.x   commentaire');
  print('-------   -------   --------   -----------');

  for (double herosX = 100; herosX <= 400; herosX += 30) {
    final Offset heros = Offset(herosX, 100);
    final double avant = camera.x;
    appliquerZoneMorte(camera, heros, marge);
    final Offset e = camera.versEcran(heros);
    final String note = camera.x == avant ? 'immobile' : 'pousse';
    print('${herosX.toStringAsFixed(0).padLeft(7)}   '
        '${e.dx.toStringAsFixed(0).padLeft(7)}   '
        '${camera.x.toStringAsFixed(0).padLeft(8)}   $note');
  }
}
```

**Résultat :**

```text
heros.x   ecran.x   camera.x   commentaire
-------   -------   --------   -----------
    100       150          0   immobile
    130       180          0   immobile
    160       210          0   immobile
    190       240          0   immobile
    220       250        -30   pousse
    250       250        -60   pousse
    280       250        -90   pousse
    310       250       -120   pousse
    340       250       -150   pousse
    370       250       -180   pousse
    400       250       -210   pousse
```

Lisez la colonne du milieu. Tant que le héros est entre 150 et 250 pixels à l'écran, sa position écran suit son mouvement et la caméra reste rigoureusement à 0. Dès qu'il franchit 250, il **reste collé à 250** : c'est la caméra qui prend le relais.

> **Remarque.** La caméra part ici en négatif car nous n'avons pas encore borné le monde. La section 25.12 corrige cela.

### Combiner zone morte et lissage

Les deux techniques ne s'opposent pas, elles se complètent. La recette habituelle :

1. calculer la **cible corrigée** par la zone morte (position que la caméra devrait avoir) ;
2. rejoindre cette cible en lissé, pas d'un coup.

```dart
void mettreAJourCamera(Camera camera, Offset heros, double dt) {
  // 1. Cible brute : centrer le héros.
  double cibleX = heros.dx - camera.largeur / 2;
  double cibleY = heros.dy - camera.hauteur / 2;

  // 2. Correction par la zone morte : on ne bouge que le nécessaire.
  const double margeX = 140;
  const double margeY = 70;
  final Offset e = camera.versEcran(heros);
  cibleX = camera.x;
  cibleY = camera.y;
  if (e.dx > camera.largeur - margeX) cibleX += e.dx - (camera.largeur - margeX);
  if (e.dx < margeX) cibleX -= margeX - e.dx;
  if (e.dy > camera.hauteur - margeY) cibleY += e.dy - (camera.hauteur - margeY);
  if (e.dy < margeY) cibleY -= margeY - e.dy;

  // 3. Rejoindre la cible en douceur.
  final double t = 1 - math.exp(-8 * dt);
  camera.x += (cibleX - camera.x) * t;
  camera.y += (cibleY - camera.y) * t;
}
```

### Choisir la taille de la zone morte

| Zone morte | Effet |
| --- | --- |
| nulle (0 x 0) | caméra toujours centrée, retour au suivi classique |
| petite (60 x 40) | supprime le jitter des micro-mouvements |
| moyenne (200 x 120) | le héros se déplace visiblement dans l'écran |
| énorme (écran entier) | caméra figée, on retrouve un écran fixe par salle |

Une zone morte verticale **très haute** est courante en plateforme : elle empêche l'écran de suivre les sauts, tout en suivant les chutes longues. C'est exactement ce que fait le mini-projet.

> **À retenir.** La zone morte transforme « la caméra suit le héros » en « la caméra ne suit le héros que quand il essaie de sortir ». C'est plus confortable et plus lisible.

---

## 25.11 — Le look-ahead : regarder devant le joueur

Un joueur a besoin de voir **où il va**, pas où il était. Si la caméra centre exactement le héros, le champ de vision est symétrique : autant d'espace derrière que devant. C'est un gâchis : la moitié gauche de l'écran montre un couloir déjà parcouru.

Le **look-ahead** (regard en avant) décale la cible dans le sens du mouvement.

```text
  SANS LOOK-AHEAD (héros centré)

  ┌────────────────────────────────────┐
  │  déjà vu          │   à découvrir  │
  │███████████████████│                │
  │                   @──>             │
  └────────────────────────────────────┘
     400 px inutiles    400 px utiles


  AVEC LOOK-AHEAD (+120 px vers la droite)

  ┌────────────────────────────────────┐
  │ déjà vu   │        à découvrir     │
  │███████████│                        │
  │           @──>                     │
  └────────────────────────────────────┘
     280 px      520 px utiles
```

### Implémentation naïve, et son défaut

Le premier réflexe est de décaler selon la direction du regard :

```dart
// Version naïve : à ne pas garder telle quelle.
final double avance = heros.regardeAGauche ? -120 : 120;
camera.cible = heros.centre + Offset(avance, 0);
```

Le défaut apparaît dès que le héros fait demi-tour : la cible saute instantanément de +120 à -120, soit un bond de 240 pixels. Même lissée, la caméra fait un balayage désagréable à chaque changement de direction.

### Implémentation correcte : lisser l'avance elle-même

On stocke l'avance dans une variable, et on la fait tendre vers sa valeur souhaitée avec le même `lerp`. Le retournement devient progressif.

```dart
class Camera {
  // ... champs précédents ...

  /// Décalage horizontal courant du regard, en pixels monde.
  double avance = 0;

  /// Amplitude maximale du look-ahead.
  double avanceMax = 120;

  /// Vitesse à laquelle l'avance se retourne.
  double vitesseAvance = 3;

  /// [directionX] vaut -1, 0 ou 1.
  void mettreAJourAvance(double directionX, double dt) {
    final double souhaitee = directionX * avanceMax;
    final double t = 1 - math.exp(-vitesseAvance * dt);
    avance += (souhaitee - avance) * t;
  }
}
```

Et dans la boucle de jeu :

```dart
camera.mettreAJourAvance(heros.directionX, dt);
camera.cible = heros.centre + Offset(camera.avance, 0);
camera.mettreAJour(dt);
```

Remarquez que `vitesseAvance` (3) est plus lente que `vitesseSuivi` (6 ou 8). C'est volontaire : on veut que la caméra rattrape vite le héros, mais qu'elle change de regard lentement. Si les deux vitesses sont égales, le look-ahead devient nerveux.

### Look-ahead basé sur la vitesse plutôt que sur la direction

Une variante plus fine utilise la vélocité du héros (chapitre 23) au lieu de sa direction. La caméra regarde alors d'autant plus loin que le héros va vite.

```dart
/// [vitesseX] en pixels/seconde, [vitesseMax] la vitesse de course.
void mettreAJourAvanceParVitesse(double vitesseX, double vitesseMax, double dt) {
  final double ratio = (vitesseX / vitesseMax).clamp(-1.0, 1.0);
  final double souhaitee = ratio * avanceMax;
  final double t = 1 - math.exp(-vitesseAvance * dt);
  avance += (souhaitee - avance) * t;
}
```

Le héros qui marche doucement décale peu la caméra ; le héros qui sprinte la décale au maximum. C'est ce que font la plupart des jeux de plateforme modernes.

### Le look-ahead vertical

En plateforme, on évite généralement le look-ahead vertical automatique : il rend les sauts illisibles. En revanche, on l'active souvent **à la demande** : le joueur maintient la flèche du bas, et la caméra descend pour lui montrer ce qu'il y a en dessous.

```dart
if (toucheBasEnfoncee) {
  avanceY = 100;   // on regarde vers le bas
} else if (toucheHautEnfoncee) {
  avanceY = -100;  // on regarde vers le haut
} else {
  avanceY = 0;
}
```

> **À retenir.** Le look-ahead donne au joueur de l'information utile. Mais il doit toujours être lissé indépendamment du suivi, sinon chaque demi-tour devient un coup de balai.

---

## 25.12 — Borner la caméra aux limites du niveau (clamp)

Voici un défaut visible en dix secondes de test : le héros s'approche du bord gauche du niveau, la caméra continue de le centrer, et l'écran affiche du vide au-delà de la limite du monde.

```text
  SANS BORNAGE

  monde     |################################
  caméra  ┌──────────┐
          │▒▒▒▒▒|####│     ▒ = hors du monde, vide, moche
          └──────────┘
```

La correction s'appelle le **clamp**, et vous connaissez déjà la méthode : `num.clamp(min, max)`, vue au chapitre 03 et réutilisée au chapitre 20 pour plafonner le `dt`.

```dart
void borner(double largeurMonde, double hauteurMonde) {
  x = x.clamp(0.0, largeurMonde - largeur);
  y = y.clamp(0.0, hauteurMonde - hauteur);
}
```

Le coin haut-gauche de la caméra ne peut pas descendre sous 0, ni dépasser `largeurMonde - largeur` (sinon le bord droit de la vue sortirait du monde).

### Le cas piège : un monde plus petit que l'écran

Si `largeurMonde < largeur` (par exemple, une petite salle de 600 pixels affichée sur un écran de 800), alors `largeurMonde - largeur` vaut -200, et l'appel devient `x.clamp(0.0, -200.0)`. Dart lève une exception :

```text
Unhandled exception:
Invalid argument(s): Invalid clamp range
```

C'est un plantage bien réel, qui apparaît le jour où quelqu'un joue en plein écran sur un grand moniteur. La solution est de centrer le monde dans ce cas :

```dart
void borner(double largeurMonde, double hauteurMonde) {
  if (largeurMonde <= largeur) {
    // Le monde tient dans l'écran : on le centre horizontalement.
    x = (largeurMonde - largeur) / 2;
  } else {
    x = x.clamp(0.0, largeurMonde - largeur);
  }

  if (hauteurMonde <= hauteur) {
    y = (hauteurMonde - hauteur) / 2;
  } else {
    y = y.clamp(0.0, hauteurMonde - hauteur);
  }
}
```

Testons les trois cas.

```dart
import 'dart:ui';

class Camera {
  Camera({this.x = 0, this.y = 0, required this.largeur, required this.hauteur});
  double x;
  double y;
  double largeur;
  double hauteur;

  void borner(double largeurMonde, double hauteurMonde) {
    if (largeurMonde <= largeur) {
      x = (largeurMonde - largeur) / 2;
    } else {
      x = x.clamp(0.0, largeurMonde - largeur);
    }
    if (hauteurMonde <= hauteur) {
      y = (hauteurMonde - hauteur) / 2;
    } else {
      y = y.clamp(0.0, hauteurMonde - hauteur);
    }
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

void main() {
  const double lMonde = 2400;
  const double hMonde = 1200;

  // Cas 1 : la caméra veut aller trop à gauche.
  final Camera c1 = Camera(x: -300, y: 200, largeur: 800, hauteur: 400);
  c1.borner(lMonde, hMonde);
  print('trop a gauche  -> $c1   (attendu 0.0, 200.0)');

  // Cas 2 : la caméra veut aller trop à droite.
  final Camera c2 = Camera(x: 5000, y: 1100, largeur: 800, hauteur: 400);
  c2.borner(lMonde, hMonde);
  print('trop a droite  -> $c2   (attendu 1600.0, 800.0)');

  // Cas 3 : monde plus petit que l'écran.
  final Camera c3 = Camera(x: 120, y: 0, largeur: 800, hauteur: 400);
  c3.borner(600, 300);
  print('monde minuscule-> $c3   (attendu -100.0, -50.0)');
}
```

**Résultat :**

```text
trop a gauche  -> (0.0, 200.0)   (attendu 0.0, 200.0)
trop a droite  -> (1600.0, 800.0)   (attendu 1600.0, 800.0)
monde minuscule-> (-100.0, -50.0)   (attendu -100.0, -50.0)
```

Dans le troisième cas, la caméra se place en `-100` : le monde de 600 pixels apparaît alors centré dans une fenêtre de 800, avec 100 pixels de marge de chaque côté. C'est exactement le comportement souhaité.

### Où appeler `borner()` ?

**Toujours en dernier**, après le lissage, la zone morte, le look-ahead et le zoom. C'est la contrainte finale, celle qui a le dernier mot.

```dart
void mettreAJourCamera(double dt) {
  camera.mettreAJourAvance(heros.directionX, dt);   // 1. look-ahead
  camera.cible = heros.centre + Offset(camera.avance, 0);
  appliquerZoneMorte(camera, camera.cible, marge);   // 2. zone morte
  camera.mettreAJour(dt);                            // 3. lissage
  camera.borner(niveau.largeur, niveau.hauteur);     // 4. bornage — EN DERNIER
}
```

Si vous bornez avant de lisser, le lissage fera ressortir la caméra du monde. Le symptôme : une bande vide qui apparaît puis disparaît au bord du niveau, comme un clignotement.

### Bornes personnalisées par salle

Rien n'oblige les bornes à être celles du monde entier. Beaucoup de jeux définissent une **zone de caméra** par salle : la caméra reste enfermée dans la salle courante, même si le monde continue au-delà.

```dart
class ZoneCamera {
  const ZoneCamera(this.rect);
  final Rect rect;
}

void bornerDans(Camera camera, Rect zone) {
  final double maxX = zone.right - camera.largeur;
  final double maxY = zone.bottom - camera.hauteur;
  camera.x = maxX <= zone.left ? zone.left : camera.x.clamp(zone.left, maxX);
  camera.y = maxY <= zone.top ? zone.top : camera.y.clamp(zone.top, maxY);
}
```

C'est ainsi que fonctionne le cadrage par salle du mini-projet de la section 25.30.

> **À retenir.** `clamp` est le dernier maillon de la chaîne de la caméra. Et vérifiez toujours que `max >= min`, sinon Dart lève une exception.

---

## 25.13 — Le zoom

Le zoom est un facteur d'échelle appliqué au rendu. Un zoom de 2 signifie « un pixel du monde occupe deux pixels d'écran ». Un zoom de 0,5 signifie « on voit deux fois plus de monde, en plus petit ».

```text
  ZOOM = 1                ZOOM = 2                 ZOOM = 0.5
  ┌────────────┐          ┌────────────┐           ┌────────────┐
  │ ▪ ▪ ▪ ▪ ▪  │          │  ▪▪  ▪▪    │           │▪▪▪▪▪▪▪▪▪▪▪ │
  │ ▪ ▪ ▪ ▪ ▪  │          │  ▪▪  ▪▪    │           │▪▪▪▪▪▪▪▪▪▪▪ │
  │            │          │            │           │▪▪▪▪▪▪▪▪▪▪▪ │
  └────────────┘          └────────────┘           └────────────┘
  vue monde 800           vue monde 400            vue monde 1600
```

### Les formules à corriger

Le zoom modifie les deux conversions. Monde vers écran :

```text
  ecranX = (mondeX - cameraX) * zoom
  ecranY = (mondeY - cameraY) * zoom
```

Et l'inverse, obtenue en isolant `mondeX` :

```text
  mondeX = ecranX / zoom + cameraX
  mondeY = ecranY / zoom + cameraY
```

C'est ici que le débutant se fait piéger : il applique le zoom au rendu (`canvas.scale`) mais oublie de le mettre dans `versMonde`. Résultat : les clics tombent au bon endroit à zoom 1, et de plus en plus à côté quand on zoome. Symptôme caractéristique : « plus je zoome, plus mon curseur est décalé ».

### La taille de la vue change aussi

À zoom 2, un écran de 800 pixels n'affiche plus que 400 pixels de monde. Il faut donc corriger le rectangle visible :

```dart
Rect get vue => Rect.fromLTWH(x, y, largeur / zoom, hauteur / zoom);
```

Et le centrage :

```dart
double get _cibleCoinX => cible.dx - (largeur / zoom) / 2;
double get _cibleCoinY => cible.dy - (hauteur / zoom) / 2;
```

Ainsi que le bornage :

```dart
void borner(double lMonde, double hMonde) {
  final double vueL = largeur / zoom;
  final double vueH = hauteur / zoom;
  x = lMonde <= vueL ? (lMonde - vueL) / 2 : x.clamp(0.0, lMonde - vueL);
  y = hMonde <= vueH ? (hMonde - vueH) / 2 : y.clamp(0.0, hMonde - vueH);
}
```

Retenez la règle générale : **partout où apparaissait `largeur`, il faut désormais `largeur / zoom`.**

### L'application au canevas

L'ordre des transformations compte, et c'est contre-intuitif. Il faut d'abord `scale`, puis `translate` :

```dart
void appliquer(Canvas canvas) {
  canvas.save();
  canvas.scale(zoom);        // 1
  canvas.translate(-x, -y);  // 2
}
```

Pourquoi cet ordre ? Parce que les transformations d'un `Canvas` s'appliquent **du dernier vers le premier** sur les points dessinés. Un point `p` subit d'abord `translate` (donc `p - camera`), puis `scale` (donc `(p - camera) * zoom`). C'est exactement la formule voulue.

Si vous inversez :

```dart
canvas.translate(-x, -y);
canvas.scale(zoom);
// donne (p * zoom) - camera : FAUX
```

Le décor est alors zoomé mais la caméra ne l'est pas : le monde glisse dans le mauvais sens quand on zoome. Testez les deux ordres une fois, pour bien voir la différence — vous ne l'oublierez plus.

### La classe `Camera` complète avec zoom

```dart
import 'dart:math' as math;
import 'dart:ui';

class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    this.largeur = 0,
    this.hauteur = 0,
    this.zoom = 1,
    this.vitesseSuivi = 6,
  });

  double x;
  double y;
  double largeur;
  double hauteur;
  double zoom;
  double vitesseSuivi;

  Offset cible = Offset.zero;

  double get vueLargeur => largeur / zoom;
  double get vueHauteur => hauteur / zoom;
  Rect get vue => Rect.fromLTWH(x, y, vueLargeur, vueHauteur);

  Offset versEcran(Offset m) =>
      Offset((m.dx - x) * zoom, (m.dy - y) * zoom);

  Offset versMonde(Offset e) =>
      Offset(e.dx / zoom + x, e.dy / zoom + y);

  void centrerSur(Offset monde) {
    x = monde.dx - vueLargeur / 2;
    y = monde.dy - vueHauteur / 2;
  }

  void mettreAJour(double dt) {
    final double t = 1 - math.exp(-vitesseSuivi * dt);
    x += (cible.dx - vueLargeur / 2 - x) * t;
    y += (cible.dy - vueHauteur / 2 - y) * t;
  }

  void borner(double lMonde, double hMonde) {
    x = lMonde <= vueLargeur
        ? (lMonde - vueLargeur) / 2
        : x.clamp(0.0, lMonde - vueLargeur);
    y = hMonde <= vueHauteur
        ? (hMonde - vueHauteur) / 2
        : y.clamp(0.0, hMonde - vueHauteur);
  }

  void appliquer(Canvas canvas) {
    canvas.save();
    canvas.scale(zoom);
    canvas.translate(-x, -y);
  }

  void retirer(Canvas canvas) => canvas.restore();
}
```

### Zoomer sans perdre le point visé

Quand on zoome au centre de l'écran, tout va bien : la cible reste centrée. Mais si vous voulez zoomer **sur le curseur** (comme dans un éditeur de carte), il faut compenser.

La recette : mémoriser le point monde sous le curseur, changer le zoom, puis déplacer la caméra pour que ce point revienne sous le curseur.

```dart
void zoomerSur(Offset pointEcran, double nouveauZoom) {
  final Offset avant = versMonde(pointEcran);
  zoom = nouveauZoom;
  final Offset apres = versMonde(pointEcran);
  x += avant.dx - apres.dx;
  y += avant.dy - apres.dy;
}
```

Trois lignes, et le zoom « colle » au curseur. C'est la même astuce dans tous les logiciels de dessin.

### Le zoom et le pixel art

Attention : zoomer une image pixel art avec un facteur non entier produit des colonnes de pixels irrégulières. Si votre jeu utilise des sprites 16 sur 16, préférez des zooms entiers (2, 3, 4) et appliquez `FilterQuality.none`, comme au chapitre 22.

Pour un zoom animé (transition de 2 à 3), le flou est acceptable pendant le mouvement, mais revenez à un entier une fois l'animation terminée.

> **À retenir.** Le zoom multiplie dans un sens, divise dans l'autre, et se glisse dans `versEcran`, `versMonde`, `vue`, `centrerSur` et `borner`. Oubliez-en un seul et vous aurez un décalage difficile à diagnostiquer.

---

## 25.14 — Le tremblement de caméra (screen shake)

Le tremblement d'écran est l'un des effets les plus rentables du jeu vidéo : quelques lignes de code, et chaque coup porté devient satisfaisant. Il ne change rien à la logique du jeu, il change la sensation.

### Le principe

On ajoute un décalage aléatoire à la position de la caméra, uniquement au moment du rendu, et on fait décroître ce décalage jusqu'à zéro.

```text
  INTENSITÉ DU TREMBLEMENT AU FIL DU TEMPS

  intensité
     8 │█
       │██
     6 │███
       │████
     4 │█████
       │███████
     2 │██████████
       │████████████████
     0 └──────────────────────────> temps
       ^
    l'impact
```

Trois erreurs classiques à éviter dès maintenant :

1. **Modifier `camera.x` directement.** Le tremblement se cumulerait avec le suivi et la caméra dériverait. Il faut un décalage **séparé**.
2. **Ne pas amortir.** Un tremblement constant est insupportable au bout d'une seconde.
3. **Tirer un nouveau nombre aléatoire à chaque frame sans lisser.** Cela produit un scintillement dur plutôt qu'une secousse.

### Implémentation

```dart
import 'dart:math' as math;

class Secousse {
  Secousse({int graine = 7}) : _alea = math.Random(graine);

  final math.Random _alea;

  /// Intensité courante, en pixels.
  double _intensite = 0;

  /// Vitesse d'amortissement, en pixels par seconde.
  double amortissement = 24;

  /// Décalage courant, à ajouter à la position de la caméra au rendu.
  double decalageX = 0;
  double decalageY = 0;

  bool get active => _intensite > 0.01;

  /// Déclenche (ou renforce) une secousse.
  void declencher(double intensite) {
    // On garde la plus forte : deux coups simultanés ne s'additionnent pas.
    if (intensite > _intensite) _intensite = intensite;
  }

  void mettreAJour(double dt) {
    if (!active) {
      _intensite = 0;
      decalageX = 0;
      decalageY = 0;
      return;
    }
    // Décalage aléatoire dans un carré de côté 2 * intensité.
    decalageX = (_alea.nextDouble() * 2 - 1) * _intensite;
    decalageY = (_alea.nextDouble() * 2 - 1) * _intensite;

    // Amortissement linéaire, indépendant du framerate.
    _intensite -= amortissement * dt;
    if (_intensite < 0) _intensite = 0;
  }
}
```

Et l'intégration dans la caméra : le décalage s'ajoute **au moment d'appliquer** la transformation, jamais à `x` et `y`.

```dart
void appliquer(Canvas canvas) {
  canvas.save();
  canvas.scale(zoom);
  canvas.translate(-(x + secousse.decalageX), -(y + secousse.decalageY));
}
```

Attention : si vous décalez la vue, il faut décaler aussi `versEcran` et `versMonde`, sinon un clic pendant une secousse tombera légèrement à côté. En pratique, on accepte souvent l'imprécision (les secousses durent 0,3 seconde) ; mais pour un jeu de visée, ajoutez le décalage aux deux conversions.

### Simulation en console

```dart
import 'dart:math' as math;

class Secousse {
  Secousse({int graine = 7}) : _alea = math.Random(graine);
  final math.Random _alea;
  double _intensite = 0;
  double amortissement = 24;
  double decalageX = 0;
  double decalageY = 0;

  bool get active => _intensite > 0.01;
  double get intensite => _intensite;

  void declencher(double i) {
    if (i > _intensite) _intensite = i;
  }

  void mettreAJour(double dt) {
    if (!active) {
      _intensite = 0;
      decalageX = 0;
      decalageY = 0;
      return;
    }
    decalageX = (_alea.nextDouble() * 2 - 1) * _intensite;
    decalageY = (_alea.nextDouble() * 2 - 1) * _intensite;
    _intensite -= amortissement * dt;
    if (_intensite < 0) _intensite = 0;
  }
}

void main() {
  final Secousse s = Secousse();
  s.declencher(8);

  const double dt = 1 / 30; // on simule à 30 FPS pour tenir en 10 lignes
  print('frame  intensite  |decalage| <= intensite ?');
  print('-----  ---------  -------------------------');
  for (int f = 0; f < 12; f++) {
    s.mettreAJour(dt);
    final bool borne = s.decalageX.abs() <= s.intensite + 0.8 &&
        s.decalageY.abs() <= s.intensite + 0.8;
    print('${f.toString().padLeft(5)}  '
        '${s.intensite.toStringAsFixed(2).padLeft(9)}  '
        '${borne ? "oui" : "NON"}');
  }
  print('');
  print('secousse encore active : ${s.active}');
}
```

**Résultat :**

```text
frame  intensite  |decalage| <= intensite ?
-----  ---------  -------------------------
    0       7.20  oui
    1       6.40  oui
    2       5.60  oui
    3       4.80  oui
    4       4.00  oui
    5       3.20  oui
    6       2.40  oui
    7       1.60  oui
    8       0.80  oui
    9       0.00  oui
   10       0.00  oui
   11       0.00  oui

secousse encore active : false
```

L'intensité descend de 8 à 0 en dix frames, soit un tiers de seconde, en marches régulières de 0,8 (soit `24 * 1/30`). Le décalage reste toujours dans les bornes de l'intensité de la frame précédente, et s'annule à la fin.

> **Remarque.** Nous n'affichons pas les valeurs aléatoires elles-mêmes : elles dépendent de la graine **et** de l'implémentation de `math.Random`. Ce qui doit être testé, c'est l'invariant : le décalage est borné et l'intensité décroît. Fixer la graine rend malgré tout la secousse reproductible d'une exécution à l'autre, ce qui est précieux en débogage.

### Quelle intensité pour quel événement ?

| Événement | Intensité | Amortissement |
| --- | --- | --- |
| Le héros atterrit d'un petit saut | 1,5 | 30 |
| Le héros prend un coup | 5 | 24 |
| Le héros meurt | 12 | 12 |
| Le coffre s'ouvre | 2 | 40 |
| Le boss frappe le sol | 16 | 18 |
| Explosion | 10 | 25 |

Une règle empirique : au-delà de 20 pixels d'intensité, l'écran devient illisible. Réservez ces valeurs aux fins de niveau.

> **À retenir.** Le tremblement est un décalage **temporaire et séparé**. Il ne touche jamais la position réelle de la caméra, et il s'amortit toujours.

---

## 25.15 — Le culling : ne dessiner que le visible

Votre monde contient 4 000 tuiles, 60 gobelins et 200 torches. L'écran en montre 5 %. Dessiner les 95 % restants est du travail pur perdu : le `Canvas` doit quand même calculer chaque `drawRect`, chaque `drawImageRect`, même si le résultat tombe hors de la zone visible.

Le **culling** (littéralement « élagage ») consiste à tester, avant de dessiner, si l'objet intersecte la vue.

```dart
/// Renvoie true si [boite] (en monde) est visible par [camera].
bool estVisible(Camera camera, Rect boite) => camera.vue.overlaps(boite);
```

`Rect.overlaps` est fourni par Flutter et fait exactement le test AABB du chapitre 24. Utilisez-le, il est correct et rapide.

Dans le rendu :

```dart
int dessinees = 0;
for (final Gobelin g in gobelins) {
  if (!camera.vue.overlaps(g.boite)) continue; // élagué
  canvas.drawRect(g.boite, peintureGobelin);
  dessinees++;
}
```

### La marge de sécurité

Un objet dont la hitbox est juste hors de la vue peut avoir un **sprite plus large** que sa hitbox (une aura, une ombre, une arme tendue). Si vous cullez sur la hitbox stricte, ces objets disparaissent brutalement au bord de l'écran.

La parade tient en une ligne : élargir la vue de quelques dizaines de pixels.

```dart
Rect get vueElargie => vue.inflate(64);
```

`inflate(64)` agrandit le rectangle de 64 pixels dans les quatre directions. Choisissez une marge au moins égale au plus grand sprite du jeu.

### Ce que le culling fait gagner

| Situation | Objets dans le monde | Objets dessinés sans culling | Avec culling |
| --- | --- | --- | --- |
| Salle 1 du donjon | 640 tuiles | 640 | 63 |
| Monde complet 4 salles | 4 000 tuiles | 4 000 | 63 |
| 60 gobelins répartis | 60 | 60 | 4 à 8 |

Le point crucial : **le nombre d'objets dessinés ne dépend plus de la taille du monde**, seulement de la taille de l'écran. Vous pouvez multiplier votre niveau par dix sans perdre une image par seconde.

### Le culling n'est pas gratuit

Tester `overlaps` sur 100 000 objets coûte lui aussi du temps. Pour de très grands mondes, on utilise une structure spatiale (grille, quadtree) afin de ne même pas parcourir la liste complète. Pour une tilemap, le problème disparaît naturellement : on calcule directement les indices de tuiles visibles, comme nous le verrons à la section 25.22.

> **À retenir.** Culler, c'est répondre à « faut-il dessiner ? » avant de dessiner. Toujours avec une marge, jamais sur la hitbox nue.

---

## 25.16 — Le parallaxe : plusieurs plans à des vitesses différentes

Regardez par la fenêtre d'un train. Les poteaux au bord de la voie filent. Les arbres du champ défilent plus lentement. Les montagnes au fond semblent presque immobiles. Votre cerveau en déduit les distances sans y penser.

Cette différence de vitesse apparente s'appelle la **parallaxe**. En jeu 2D, on la simule en faisant défiler chaque plan de décor à une fraction de la vitesse de la caméra.

```text
  UN MONDE À QUATRE PLANS

  plan            facteur   quand la caméra avance de 100 px, le plan
                            se décale à l'écran de :

  ciel / lune       0.0     0 px      (immobile, collé à l'écran)
  montagnes         0.2     20 px     (très lent)
  arbres lointains  0.5     50 px     (moitié moins vite)
  JEU (héros, sol)  1.0     100 px    (référence)
  herbes devant     1.4     140 px    (plus vite que le jeu)


  ┌─────────────────────────────────────────────┐
  │  O                                    ciel  │  0.0
  │      ╱╲    ╱╲╲    ╱╲          montagnes     │  0.2
  │    Y  Y  Y   Y  Y  Y   Y       arbres       │  0.5
  │ ███████████████████████████       sol       │  1.0
  │ ww    www      ww    www       herbes       │  1.4
  └─────────────────────────────────────────────┘
```

### La formule

Pour un plan de facteur `f`, le décalage à l'écran vaut :

```text
  decalagePlan = -cameraX * f
```

Avec `f = 1`, on retrouve exactement `-cameraX` : c'est le plan du jeu. Avec `f = 0`, le plan ne bouge jamais : c'est le ciel. Avec `f = 1,4`, le plan va plus vite que le jeu : il paraît **devant** le héros, c'est un premier plan (*foreground*).

En code, on n'utilise pas la caméra du jeu : on translate le canevas d'une fraction.

```dart
void dessinerPlan(Canvas canvas, Camera camera, double facteur, void Function() contenu) {
  canvas.save();
  canvas.translate(-camera.x * facteur, -camera.y * facteur);
  contenu();
  canvas.restore();
}
```

### Le parallaxe vertical

En plateforme, on applique souvent un facteur vertical **plus faible** que l'horizontal, voire nul. Raison : quand le héros saute, on ne veut pas que les montagnes descendent d'un coup. Un facteur vertical de 0,1 suffit à suggérer la profondeur.

```dart
canvas.translate(-camera.x * facteurX, -camera.y * facteurY);
```

### Le tableau des facteurs, pour s'y retrouver

| Facteur | Position perçue | Exemple |
| --- | --- | --- |
| 0,0 | infiniment loin | ciel, lune, dégradé |
| 0,1 – 0,3 | très loin | montagnes, silhouette de château |
| 0,4 – 0,7 | arrière-plan proche | murs de la salle, colonnes |
| 1,0 | plan de jeu | sol, héros, ennemis, coffres |
| 1,2 – 2,0 | premier plan | herbes, grilles, torches devant |

Un premier plan à facteur supérieur à 1 est très efficace, mais il masque le jeu. Utilisez-le partiellement transparent, ou seulement en bas de l'écran.

> **À retenir.** Parallaxe = un facteur multiplicatif sur la translation. Aucune notion de 3D, aucune trigonométrie.

---

## 25.17 — Implémenter un fond parallaxe

Un plan de fond pose un problème pratique : le monde fait 3 200 pixels, mais votre image de montagnes en fait 400. Il faut donc **répéter** le motif, et le répéter à l'infini si le facteur est très faible.

La technique s'appelle le **pavage par modulo**.

```text
  RÉPÉTITION PAR MODULO (motif de largeur 200)

  décalage = -cameraX * 0.3
  origine  = décalage % 200      <- toujours dans [-200, 200[

  écran  ┌───────────────────────────────────────┐
         │  ▲▲▲  ▲▲▲  ▲▲▲  ▲▲▲  ▲▲▲  ▲▲▲  ▲▲▲   │
         └───────────────────────────────────────┘
          ^     ^     ^     ^     ^     ^
       origine  +200  +400  +600  +800  ...
```

On calcule une origine ramenée dans l'intervalle du motif, puis on dessine autant de copies que nécessaire pour couvrir l'écran.

```dart
/// Dessine un motif répété horizontalement pour couvrir tout l'écran.
void paverHorizontal({
  required Canvas canvas,
  required double largeurEcran,
  required double largeurMotif,
  required double decalage,
  required void Function(double x) dessinerMotif,
}) {
  // Ramène le décalage dans [-largeurMotif, 0].
  double origine = decalage % largeurMotif;
  if (origine > 0) origine -= largeurMotif;

  for (double x = origine; x < largeurEcran; x += largeurMotif) {
    dessinerMotif(x);
  }
}
```

Le `if (origine > 0)` est indispensable : en Dart, l'opérateur `%` sur des `double` négatifs renvoie un résultat du signe du dividende. Sans cette correction, une colonne de motif manque au bord gauche quand la caméra part vers la gauche.

### Programme complet : trois plans de parallaxe

Voici un `main.dart` complet. Le héros se déplace automatiquement de gauche à droite ; les montagnes, les colonnes du donjon et les herbes du premier plan défilent à trois vitesses différentes.

```dart
import 'dart:math' as math;
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const AppliParallaxe());

class AppliParallaxe extends StatelessWidget {
  const AppliParallaxe({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0C0A16),
        body: EcranParallaxe(),
      ),
    );
  }
}

class Camera {
  Camera({this.x = 0, this.y = 0, this.largeur = 0, this.hauteur = 0});

  double x;
  double y;
  double largeur;
  double hauteur;

  void centrerSur(Offset monde) {
    x = monde.dx - largeur / 2;
    y = monde.dy - hauteur / 2;
  }

  void borner(double lMonde, double hMonde) {
    x = lMonde <= largeur ? (lMonde - largeur) / 2 : x.clamp(0.0, lMonde - largeur);
    y = hMonde <= hauteur ? (hMonde - hauteur) / 2 : y.clamp(0.0, hMonde - hauteur);
  }
}

class EcranParallaxe extends StatefulWidget {
  const EcranParallaxe({super.key});

  @override
  State<EcranParallaxe> createState() => _EcranParallaxeState();
}

class _EcranParallaxeState extends State<EcranParallaxe>
    with SingleTickerProviderStateMixin {
  static const double largeurMonde = 3200;
  static const double hauteurMonde = 600;

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  final Camera _camera = Camera();
  double _herosX = 200;
  double _direction = 1;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick)..start();
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;
    if (_camera.largeur == 0) return;

    _herosX += 160 * _direction * dt;
    if (_herosX > largeurMonde - 100) _direction = -1;
    if (_herosX < 100) _direction = 1;

    _camera.centrerSur(Offset(_herosX, hauteurMonde / 2));
    _camera.borner(largeurMonde, hauteurMonde);

    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        _camera.largeur = c.maxWidth;
        _camera.hauteur = c.maxHeight;
        return CustomPaint(
          painter: PeintreParallaxe(
            camera: _camera,
            herosX: _herosX,
            versLaDroite: _direction > 0,
          ),
          size: Size.infinite,
        );
      },
    );
  }
}

class PeintreParallaxe extends CustomPainter {
  PeintreParallaxe({
    required this.camera,
    required this.herosX,
    required this.versLaDroite,
  });

  final Camera camera;
  final double herosX;
  final bool versLaDroite;

  /// Pave un motif horizontalement pour couvrir tout l'écran.
  void _paver({
    required Canvas canvas,
    required double largeurEcran,
    required double largeurMotif,
    required double decalage,
    required void Function(double x) motif,
  }) {
    double origine = decalage % largeurMotif;
    if (origine > 0) origine -= largeurMotif;
    for (double x = origine; x < largeurEcran; x += largeurMotif) {
      motif(x);
    }
  }

  void _texte(Canvas canvas, String s, Offset p,
      {Color c = const Color(0xFFD8D8E8)}) {
    final TextPainter tp = TextPainter(
      text: TextSpan(text: s, style: TextStyle(color: c, fontSize: 13)),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, p);
  }

  @override
  void paint(Canvas canvas, Size size) {
    final double sol = size.height * 0.78;

    // --- PLAN 0 : le ciel. Facteur 0, totalement immobile. ---
    canvas.drawRect(
      Offset.zero & size,
      Paint()
        ..shader = const LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: <Color>[Color(0xFF1A1230), Color(0xFF3A2450)],
        ).createShader(Offset.zero & size),
    );
    canvas.drawCircle(
      Offset(size.width - 90, 70),
      26,
      Paint()..color = const Color(0xFFE8E0C0),
    );

    // --- PLAN 1 : montagnes, facteur 0.15 ---
    final Paint montagne = Paint()..color = const Color(0xFF241A3C);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 320,
      decalage: -camera.x * 0.15,
      motif: (double x) {
        final Path p = Path()
          ..moveTo(x, sol)
          ..lineTo(x + 160, sol - 190)
          ..lineTo(x + 320, sol)
          ..close();
        canvas.drawPath(p, montagne);
      },
    );

    // --- PLAN 2 : colonnes du donjon, facteur 0.45 ---
    final Paint colonne = Paint()..color = const Color(0xFF35284F);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 180,
      decalage: -camera.x * 0.45,
      motif: (double x) {
        canvas.drawRect(Rect.fromLTWH(x + 40, sol - 150, 34, 150), colonne);
        canvas.drawRect(Rect.fromLTWH(x + 30, sol - 162, 54, 14), colonne);
      },
    );

    // --- PLAN 3 : le jeu, facteur 1.0 ---
    canvas.save();
    canvas.translate(-camera.x, 0);

    canvas.drawRect(
      Rect.fromLTWH(0, sol, _EcranParallaxeState.largeurMonde,
          size.height - sol),
      Paint()..color = const Color(0xFF1D1728),
    );
    final Paint dalle = Paint()..color = const Color(0xFF2B2338);
    for (double x = 0; x < _EcranParallaxeState.largeurMonde; x += 64) {
      canvas.drawRect(Rect.fromLTWH(x + 2, sol + 4, 60, 10), dalle);
    }

    // Le héros.
    canvas.drawRect(
      Rect.fromLTWH(herosX - 12, sol - 42, 24, 42),
      Paint()..color = const Color(0xFF4EA3D8),
    );
    canvas.drawRect(
      Rect.fromLTWH(herosX - 8, sol - 54, 16, 14),
      Paint()..color = const Color(0xFFE0B48C),
    );

    canvas.restore();

    // --- PLAN 4 : premier plan, facteur 1.6 ---
    final Paint herbe = Paint()..color = const Color(0xFF12202A);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 90,
      decalage: -camera.x * 1.6,
      motif: (double x) {
        for (int i = 0; i < 5; i++) {
          final double bx = x + i * 18.0;
          final Path p = Path()
            ..moveTo(bx, size.height)
            ..lineTo(bx + 5, size.height - 34 - (i % 3) * 8)
            ..lineTo(bx + 11, size.height)
            ..close();
          canvas.drawPath(p, herbe);
        }
      },
    );

    // --- HUD, repère écran ---
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 260, 100),
      Paint()..color = const Color(0xAA000000),
    );
    _texte(canvas, 'DONJON DE DART — parallaxe', const Offset(12, 10));
    _texte(canvas, 'camera.x : ${camera.x.toStringAsFixed(0)}',
        const Offset(12, 32));
    _texte(canvas, 'heros.x  : ${herosX.toStringAsFixed(0)}',
        const Offset(12, 52));
    _texte(canvas, 'sens     : ${versLaDroite ? "droite" : "gauche"}',
        const Offset(12, 72), c: const Color(0xFF4EC9B0));
  }

  @override
  bool shouldRepaint(PeintreParallaxe oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART — parallaxe
camera.x : 0
heros.x  : 200
sens     : droite

Le héros bleu traverse un couloir de donjon à 160 pixels par seconde.
La lune reste rigoureusement immobile en haut à droite.
Les montagnes violettes glissent très lentement vers la gauche.
Les colonnes du donjon vont trois fois plus vite que les montagnes.
Le sol dallé et le héros défilent à la vitesse de référence.
Les herbes sombres du bas filent plus vite que tout le reste et donnent
l'impression de passer devant le personnage.
Arrivé au bout, le héros repart en sens inverse et tous les plans
s'inversent proportionnellement.
```

**Remarque sur `shader`.** Le dégradé du ciel est recréé à chaque frame. Pour un jeu réel, mettez-le en cache dans un champ, comme au chapitre 21 pour les `Paint`.

> **À retenir.** Un plan de parallaxe = un `save()`, un `translate(-camX * facteur, ...)`, un pavage par modulo, un `restore()`. Répétez pour chaque plan, du plus lointain au plus proche.

---

## 25.18 — Qu'est-ce qu'une tilemap ?

Jusqu'ici, nos niveaux étaient une poignée d'objets placés à la main. Cela ne monte pas en charge : un donjon complet compte des milliers de blocs de mur.

Une **tilemap** (carte de tuiles) résout le problème en posant deux contraintes salutaires :

1. le monde est découpé en une **grille régulière** de cases carrées ;
2. chaque case contient un **numéro** qui désigne un type de tuile.

```text
  LE NIVEAU N'EST PLUS UNE LISTE D'OBJETS,
  C'EST UN TABLEAU DE NOMBRES.

  colonnes ->   0  1  2  3  4  5  6  7  8  9
              ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  ligne 0     │ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│
              ├──┼──┼──┼──┼──┼──┼──┼──┼──┼──┤
  ligne 1     │ 1│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 1│
              ├──┼──┼──┼──┼──┼──┼──┼──┼──┼──┤
  ligne 2     │ 1│ 0│ 0│ 2│ 2│ 0│ 0│ 0│ 0│ 1│
              ├──┼──┼──┼──┼──┼──┼──┼──┼──┼──┤
  ligne 3     │ 1│ 0│ 0│ 0│ 0│ 0│ 0│ 3│ 0│ 1│
              ├──┼──┼──┼──┼──┼──┼──┼──┼──┼──┤
  ligne 4     │ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│ 1│
              └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘

  0 = vide      1 = mur de pierre
  2 = plateforme de bois     3 = pointes
```

### Pourquoi c'est un si bon choix

| Avantage | Explication |
| --- | --- |
| Mémoire | un `int` par case, pas un objet complet |
| Édition | un niveau se dessine dans un tableau, ou dans Tiled |
| Collision | on trouve la tuile sous le héros par une division, sans parcourir de liste |
| Rendu | on sait exactement quelles tuiles sont visibles |
| Sérialisation | un tableau d'entiers se met en JSON trivialement |
| Réutilisation | un même sprite sert 500 fois |

Le point le plus important est le troisième. Chercher « sur quoi le héros marche » dans une liste de 4 000 murs demanderait 4 000 tests par frame. Dans une tilemap, c'est **une division**. Nous y venons à la section 25.23.

### Le vocabulaire

| Terme | Sens |
| --- | --- |
| Tuile (*tile*) | une case de la grille |
| Taille de tuile | le côté d'une case en pixels monde, souvent 16, 24, 32 ou 48 |
| Colonne / ligne | les indices entiers d'une case |
| Calque (*layer*) | une grille complète ; on en superpose plusieurs |
| Atlas / tileset | la planche d'images où sont rangés les dessins de tuiles |
| Index de tuile | le numéro stocké dans la grille |

Les calques méritent un mot. Un niveau réel en utilise souvent trois :

```text
  calque 2  DÉCOR AVANT   (torches, grilles)     dessiné après le héros
  calque 1  COLLISION     (murs, sol)            dessiné et solide
  calque 0  DÉCOR ARRIÈRE (mur du fond)          dessiné en premier
```

Nous en resterons à un seul calque de collision dans ce chapitre, plus un calque d'objets (section 25.27).

> **À retenir.** Une tilemap échange la liberté de placement contre une immense simplicité de calcul. C'est un échange presque toujours gagnant en 2D.

---

## 25.19 — Le tile et la taille de tuile

La **taille de tuile** est la constante la plus structurante de votre jeu. Elle relie le monde des indices (colonnes, lignes) au monde des pixels.

```text
  taille = 32

  colonne 0 : x monde de   0 à  31
  colonne 1 : x monde de  32 à  63
  colonne 2 : x monde de  64 à  95
  colonne c : x monde de c*32 à c*32+31
```

### Comment la choisir

| Taille | Avantages | Inconvénients |
| --- | --- | --- |
| 8 | très fin, détails précis | beaucoup de tuiles, édition longue |
| 16 | standard rétro (NES, Game Boy) | héros de 1 à 2 tuiles seulement |
| 24 | bon compromis | moins de tilesets tout faits |
| 32 | lisible sur mobile, standard courant | niveaux moins détaillés |
| 48 / 64 | gros pixels, style moderne | mondes énormes en mémoire écran |

Pour le Donjon de Dart, nous prenons **32 pixels**. Le héros mesurera 24 sur 40, soit un peu moins d'une tuile de large et un peu plus d'une tuile de haut : c'est la proportion classique du jeu de plateforme.

### Une règle de conception importante

> Le héros ne doit **jamais** être exactement de la largeur d'une tuile.

S'il fait pile 32 de large, il se coincera dans le moindre couloir de 32 pixels : les erreurs d'arrondi en virgule flottante le feront alterner entre « ça passe » et « ça bloque ». Prenez 24 ou 28. Le passage devient confortable et les collisions cessent d'accrocher.

### Constante, pas nombre magique

```dart
/// Taille d'une tuile du Donjon de Dart, en pixels monde.
const double kTailleTuile = 32;
```

Écrivez cette constante une seule fois, et n'écrivez plus jamais `32` ailleurs. Le jour où vous passerez à 48, vous changerez une ligne. Le jour où vous aurez semé des `32` partout, vous en oublierez un et passerez la soirée à chercher pourquoi les pointes sont décalées d'un demi-bloc.

---

## 25.20 — Représenter un niveau avec une `List<List<int>>` (rappel du chapitre 06)

Nous y voilà : la structure de données du niveau. C'est une liste de lignes, chaque ligne étant une liste d'entiers. Vous avez manipulé exactement cela au chapitre 06 avec les grilles 2D.

```dart
final List<List<int>> salle = <List<int>>[
  <int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
  <int>[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
  <int>[1, 0, 0, 2, 2, 0, 0, 0, 0, 1],
  <int>[1, 0, 0, 0, 0, 0, 0, 3, 0, 1],
  <int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
];
```

### L'ordre des indices : `[ligne][colonne]`

C'est le piège classique, et il vaut la peine d'être martelé :

```text
  grille[ligne][colonne]      et non      grille[colonne][ligne]

  grille[y][x]                et non      grille[x][y]
```

Pourquoi cet ordre ? Parce que la liste extérieure contient les lignes, comme dans un texte. `grille[2]` est la troisième ligne, et `grille[2][3]` est sa quatrième case.

Inverser les deux produit un niveau **transposé** : les murs horizontaux deviennent verticaux. Le symptôme est facile à reconnaître une fois qu'on l'a vu.

### La classe `Tilemap`

Encapsulons la grille pour ne plus jamais manipuler la liste nue.

```dart
/// Une carte de tuiles : une grille d'entiers plus la taille d'une case.
class Tilemap {
  Tilemap({required this.tuiles, this.taille = 32});

  /// tuiles[ligne][colonne]
  final List<List<int>> tuiles;

  /// Côté d'une tuile, en pixels monde.
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;

  double get largeurMonde => colonnes * taille;
  double get hauteurMonde => lignes * taille;

  /// Renvoie l'index de tuile en (colonne, ligne).
  /// Hors de la carte, renvoie 1 : le monde est entouré de murs.
  int tuileA(int colonne, int ligne) {
    if (ligne < 0 || ligne >= lignes) return 1;
    if (colonne < 0 || colonne >= colonnes) return 1;
    return tuiles[ligne][colonne];
  }

  /// Une tuile est solide si son index n'est pas 0.
  bool solideA(int colonne, int ligne) => tuileA(colonne, ligne) != 0;

  /// Rectangle monde occupé par la tuile (colonne, ligne).
  Rect rectDe(int colonne, int ligne) =>
      Rect.fromLTWH(colonne * taille, ligne * taille, taille, taille);
}
```

Trois décisions de conception à souligner.

**Hors carte = mur.** Renvoyer 1 hors de la grille évite tous les `RangeError` et donne gratuitement des bords infranchissables. C'est un choix courant et très pratique. La variante « hors carte = vide » (renvoyer 0) convient aux mondes où le héros peut tomber dans le vide.

**`solideA` sépare la donnée de la règle.** Aujourd'hui « solide » signifie « différent de 0 ». Demain, quand la tuile 3 sera des pointes traversables, vous changerez cette seule méthode.

**`rectDe`** fournit directement la hitbox d'une tuile. Elle servira au rendu et aux collisions.

### Vérification en console

```dart
import 'dart:ui';

class Tilemap {
  Tilemap({required this.tuiles, this.taille = 32});
  final List<List<int>> tuiles;
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;
  double get largeurMonde => colonnes * taille;
  double get hauteurMonde => lignes * taille;

  int tuileA(int c, int l) {
    if (l < 0 || l >= lignes) return 1;
    if (c < 0 || c >= colonnes) return 1;
    return tuiles[l][c];
  }

  bool solideA(int c, int l) => tuileA(c, l) != 0;
  Rect rectDe(int c, int l) =>
      Rect.fromLTWH(c * taille, l * taille, taille, taille);
}

void main() {
  final Tilemap carte = Tilemap(tuiles: <List<int>>[
    <int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    <int>[1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    <int>[1, 0, 0, 2, 2, 0, 0, 0, 0, 1],
    <int>[1, 0, 0, 0, 0, 0, 0, 3, 0, 1],
    <int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
  ]);

  print('grille  : ${carte.colonnes} colonnes x ${carte.lignes} lignes');
  print('monde   : ${carte.largeurMonde} x ${carte.hauteurMonde} pixels');
  print('tuile (3,2) = ${carte.tuileA(3, 2)}  solide : ${carte.solideA(3, 2)}');
  print('tuile (4,1) = ${carte.tuileA(4, 1)}  solide : ${carte.solideA(4, 1)}');
  print('tuile hors carte (-1,0) = ${carte.tuileA(-1, 0)}');
  print('rect de (3,2) = ${carte.rectDe(3, 2)}');
}
```

**Résultat :**

```text
grille  : 10 colonnes x 5 lignes
monde   : 320.0 x 160.0 pixels
tuile (3,2) = 2  solide : true
tuile (4,1) = 0  solide : false
tuile hors carte (-1,0) = 1
rect de (3,2) = Rect.fromLTRB(96.0, 64.0, 128.0, 96.0)
```

> **À retenir.** `tuiles[ligne][colonne]`, jamais l'inverse. Et encapsulez toujours l'accès dans une méthode qui gère les bords.

---

## 25.21 — Dessiner une tilemap

La version naïve est celle à laquelle tout le monde pense en premier : deux boucles imbriquées sur toute la grille.

```dart
void dessinerTout(Canvas canvas, Tilemap carte) {
  for (int l = 0; l < carte.lignes; l++) {
    for (int c = 0; c < carte.colonnes; c++) {
      final int index = carte.tuileA(c, l);
      if (index == 0) continue;                 // le vide ne se dessine pas
      canvas.drawRect(carte.rectDe(c, l), _peintureDe(index));
    }
  }
}
```

Elle fonctionne, et elle est parfaitement acceptable pour une petite salle. Trois remarques quand même.

**Le `continue` sur le vide** est l'optimisation la plus rentable de toutes : dans un niveau typique, 70 % des cases sont vides. Ne pas les dessiner divise le travail par trois pour zéro effort.

**Un `Paint` par type de tuile, créé une fois.** Ne créez pas un `Paint` dans la boucle : vous en fabriqueriez des milliers par seconde.

```dart
class PaletteTuiles {
  static final Map<int, Paint> _peintures = <int, Paint>{
    1: Paint()..color = const Color(0xFF4A3F63), // mur de pierre
    2: Paint()..color = const Color(0xFF7A5230), // plateforme de bois
    3: Paint()..color = const Color(0xFFB84A4A), // pointes
    4: Paint()..color = const Color(0xFF2E6B4F), // mousse
  };

  static Paint de(int index) =>
      _peintures[index] ?? (Paint()..color = const Color(0xFFFF00FF));
}
```

Le magenta pour les index inconnus est une convention très utile : une tuile magenta dans le jeu signale immédiatement « index non prévu dans la palette ».

### Le liseré entre les tuiles

Si vous dessinez chaque tuile avec un contour, la grille devient visible et le niveau ressemble à un tableur. Pour un rendu propre, deux options :

- ne dessiner aucun contour, et laisser les tuiles adjacentes se fondre ;
- dessiner un **léger** liseré plus clair sur le haut de chaque tuile de sol seulement, ce qui suggère un relief.

```dart
// Un liseré uniquement quand la tuile au-dessus est vide : c'est un bord.
if (!carte.solideA(c, l - 1)) {
  canvas.drawRect(
    Rect.fromLTWH(c * carte.taille, l * carte.taille, carte.taille, 4),
    Paint()..color = const Color(0xFF6A5A8C),
  );
}
```

Ce test « la tuile du dessus est-elle vide ? » est le début de l'**auto-tiling** : choisir automatiquement le bon sprite selon les voisins. Nous n'irons pas plus loin ici, mais retenez le principe.

### Avec un vrai tileset

Quand vous aurez des images (chapitre 22), le `drawRect` devient un `drawImageRect` : le rectangle source est calculé à partir de l'index, exactement comme une frame de sprite sheet.

```dart
// index 1 -> case 0 du tileset, index 2 -> case 1, etc.
final int caseTileset = index - 1;
final int col = caseTileset % colonnesTileset;
final int lig = caseTileset ~/ colonnesTileset;
final Rect source = Rect.fromLTWH(
  col * tailleSource, lig * tailleSource, tailleSource, tailleSource);
canvas.drawImageRect(tileset, source, carte.rectDe(c, l), peinture);
```

C'est la formule ligne / colonne de la section 22.17. Rien de nouveau : une tilemap dessine des frames, comme une animation dessine des frames.

> **À retenir.** Sautez le vide, mutualisez les `Paint`, et signalez les index inconnus par une couleur criarde.

---

## 25.22 — Ne dessiner que les tuiles visibles

Le culling de la section 25.15 testait chaque objet. Sur une tilemap, on peut faire beaucoup mieux : **calculer directement la plage d'indices visibles**, et ne boucler que sur elle.

```text
  LA VUE DÉCOUPE UN RECTANGLE D'INDICES

  vue monde : x de 640 à 1440, y de 320 à 720
  taille    : 32

  colonne min = floor( 640 / 32) = 20
  colonne max = ceil (1440 / 32) = 45
  ligne   min = floor( 320 / 32) = 10
  ligne   max = ceil ( 720 / 32) = 23

  -> on boucle sur 26 colonnes x 14 lignes = 364 tuiles
     au lieu de 100 x 50 = 5 000
```

En code :

```dart
void dessinerVisible(Canvas canvas, Tilemap carte, Rect vue) {
  final int colMin = (vue.left / carte.taille).floor().clamp(0, carte.colonnes - 1);
  final int colMax = (vue.right / carte.taille).ceil().clamp(0, carte.colonnes - 1);
  final int ligMin = (vue.top / carte.taille).floor().clamp(0, carte.lignes - 1);
  final int ligMax = (vue.bottom / carte.taille).ceil().clamp(0, carte.lignes - 1);

  for (int l = ligMin; l <= ligMax; l++) {
    for (int c = colMin; c <= colMax; c++) {
      final int index = carte.tuiles[l][c];
      if (index == 0) continue;
      canvas.drawRect(carte.rectDe(c, l), PaletteTuiles.de(index));
    }
  }
}
```

### Les trois pièges de ce calcul

**Piège 1 — `floor` d'un côté, `ceil` de l'autre.** Le bord gauche doit être arrondi vers le bas (la tuile partiellement visible à gauche doit être dessinée) ; le bord droit vers le haut. Si vous mettez `floor` des deux côtés, une colonne manque au bord droit et vous verrez une bande vide qui clignote pendant le défilement.

**Piège 2 — oublier le `clamp`.** Quand la caméra montre le bord du monde, `colMin` peut valoir -1 et `colMax` dépasser le nombre de colonnes. Sans `clamp`, c'est un `RangeError` immédiat.

**Piège 3 — `clamp` sur une carte vide.** Si `carte.colonnes` vaut 0, `clamp(0, -1)` lève une exception. Ajoutez une garde si vos cartes peuvent être vides.

### Vérifier le gain

```dart
void main() {
  const double taille = 32;
  const int colonnes = 100;
  const int lignes = 50;

  // Vue de 800 x 400 quelque part au milieu.
  const double vueX = 640, vueY = 320, vueL = 800, vueH = 400;

  final int colMin = (vueX / taille).floor().clamp(0, colonnes - 1);
  final int colMax = ((vueX + vueL) / taille).ceil().clamp(0, colonnes - 1);
  final int ligMin = (vueY / taille).floor().clamp(0, lignes - 1);
  final int ligMax = ((vueY + vueH) / taille).ceil().clamp(0, lignes - 1);

  final int total = colonnes * lignes;
  final int visibles = (colMax - colMin + 1) * (ligMax - ligMin + 1);

  print('carte      : $colonnes x $lignes = $total tuiles');
  print('colonnes   : $colMin a $colMax');
  print('lignes     : $ligMin a $ligMax');
  print('a dessiner : $visibles tuiles');
  print('economie   : '
      '${(100 * (1 - visibles / total)).toStringAsFixed(1)} %');
}
```

**Résultat :**

```text
carte      : 100 x 50 = 5000 tuiles
colonnes   : 20 a 45
lignes     : 10 a 23
a dessiner : 364 tuiles
economie   : 92.7 %
```

Plus de 92 % du travail supprimé, sur une carte pourtant modeste. Sur une carte de 500 sur 200, l'économie dépasse 99 % — et surtout, le nombre de tuiles dessinées reste constant.

> **À retenir.** Sur une tilemap, on ne teste pas la visibilité : on la **calcule**. `floor` à gauche et en haut, `ceil` à droite et en bas, `clamp` partout.

---

## 25.23 — Convertir une position monde en indices de tuile

Deux conversions, symétriques, et deux lignes de code. Elles sont au cœur de toutes les collisions de la section suivante.

**Monde vers indices :**

```dart
int colonneDe(double mondeX) => (mondeX / taille).floor();
int ligneDe(double mondeY) => (mondeY / taille).floor();
```

**Indices vers monde** (coin haut-gauche de la tuile) :

```dart
double xDeColonne(int colonne) => colonne * taille;
double yDeLigne(int ligne) => ligne * taille;
```

### Pourquoi `floor()` et non `round()` ni `toInt()`

C'est le point délicat, et il concerne les coordonnées négatives.

```text
  taille = 32

  monde x = 100    ->  100 / 32 = 3.125
                       floor  -> 3    CORRECT (la tuile 3 va de 96 à 127)
                       round  -> 3    correct par hasard
                       toInt  -> 3    correct par hasard

  monde x = -10    ->  -10 / 32 = -0.3125
                       floor  -> -1   CORRECT (la tuile -1 va de -32 à -1)
                       round  ->  0   FAUX
                       toInt  ->  0   FAUX (toInt tronque vers zéro)
```

`toInt()` tronque **vers zéro**, ce qui est faux pour tout ce qui est à gauche ou au-dessus de l'origine. Le bug ne se voit pas tant que le héros reste dans le monde, puis se manifeste le jour où il sort d'un pixel par la gauche : il traverse le mur.

Utilisez **toujours** `floor()`.

### Le piège du bord exact

Que vaut la colonne du point `x = 96` avec des tuiles de 32 ? `96 / 32 = 3.0`, donc colonne 3. C'est correct : la tuile 3 commence exactement à 96.

Mais que vaut la colonne du **bord droit** d'une hitbox qui s'arrête à `x = 96` ? Le rectangle `[64, 96[` occupe les colonnes 2 et 3 selon ce calcul, alors qu'il ne touche pas la colonne 3 : il s'arrête pile à sa frontière.

La convention habituelle est de retirer un epsilon au bord droit et au bord bas :

```dart
final int colDroite = ((boite.right - 0.001) / taille).floor();
final int ligBas = ((boite.bottom - 0.001) / taille).floor();
```

Sans cela, un héros posé exactement sur une frontière détectera une collision avec la tuile suivante et restera bloqué contre du vide. C'est un bug pénible, difficile à reproduire, et cette petite soustraction l'élimine.

### La plage de tuiles occupée par un rectangle

```dart
/// Renvoie les indices (colMin, ligMin, colMax, ligMax) couverts par [boite].
class PlageTuiles {
  const PlageTuiles(this.colMin, this.ligMin, this.colMax, this.ligMax);
  final int colMin;
  final int ligMin;
  final int colMax;
  final int ligMax;

  @override
  String toString() =>
      'colonnes $colMin..$colMax, lignes $ligMin..$ligMax';
}

PlageTuiles plageDe(Rect boite, double taille) {
  const double eps = 0.001;
  return PlageTuiles(
    (boite.left / taille).floor(),
    (boite.top / taille).floor(),
    ((boite.right - eps) / taille).floor(),
    ((boite.bottom - eps) / taille).floor(),
  );
}

void main() {
  const double t = 32;

  print(plageDe(const Rect.fromLTWH(70, 40, 24, 40), t));
  print(plageDe(const Rect.fromLTWH(64, 0, 32, 32), t));
  print(plageDe(const Rect.fromLTWH(-10, -5, 24, 24), t));
}
```

**Résultat :**

```text
colonnes 2..3, lignes 1..2
colonnes 2..2, lignes 0..0
colonnes -1..0, lignes -1..0
```

Vérifions la deuxième ligne : le rectangle `[64, 96[` sur `[0, 32[` occupe exactement une tuile, la `(2, 0)`. Sans l'epsilon, on aurait obtenu `colonnes 2..3` et `lignes 0..1`, soit quatre tuiles testées au lieu d'une, dont trois à tort.

> **À retenir.** `floor()` pour convertir, epsilon sur les bords droit et bas. Ces deux détails règlent 90 % des bugs de collision sur tilemap.

---

## 25.24 — Collision joueur / tilemap (rappel du chapitre 24)

Nous savons détecter une collision AABB (chapitre 24) et trouver les tuiles occupées (section 25.23). Il reste à **résoudre** : repousser le héros hors du mur.

### La règle d'or : un axe à la fois

Ne déplacez jamais le héros en diagonale d'un seul coup. Faites :

1. déplacer sur X, chercher les tuiles solides, repousser horizontalement ;
2. déplacer sur Y, chercher les tuiles solides, repousser verticalement.

```text
  DÉPLACEMENT EN DIAGONALE, RÉSOLU EN UN COUP : AMBIGU

      ┌───┐              on chevauche le coin du mur.
      │ @ │              Faut-il repousser en X ou en Y ?
      └───┼───┐          Impossible de trancher : le héros
          │███│          "accroche" les coins.
          └───┘

  RÉSOLU AXE PAR AXE : SANS AMBIGUÏTÉ

  étape 1 (X)     @ →│███│   on repousse en X, sans toucher à Y
  étape 2 (Y)     @ ↓        on repousse en Y, sans toucher à X
```

Cette séparation supprime le cas ambigu, permet de savoir précisément si l'on a heurté un mur (X) ou le sol (Y), et donne gratuitement le drapeau `auSol` nécessaire au saut.

### L'implémentation

```dart
class Heros {
  Heros({required this.x, required this.y});

  double x;              // coin haut-gauche de la hitbox, en monde
  double y;
  double vx = 0;         // vitesse, chapitre 23
  double vy = 0;

  static const double largeur = 24;
  static const double hauteur = 40;

  bool auSol = false;

  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset get centre => Offset(x + largeur / 2, y + hauteur / 2);

  void deplacer(Tilemap carte, double dt) {
    // --- AXE X ---
    x += vx * dt;
    _resoudreX(carte);

    // --- AXE Y ---
    vy += 900 * dt;         // gravité, chapitre 23
    y += vy * dt;
    auSol = false;
    _resoudreY(carte);
  }

  void _resoudreX(Tilemap carte) {
    if (vx == 0) return;
    const double eps = 0.001;
    final Rect b = boite;
    final int ligHaut = (b.top / carte.taille).floor();
    final int ligBas = ((b.bottom - eps) / carte.taille).floor();

    if (vx > 0) {
      final int col = ((b.right - eps) / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = col * carte.taille - largeur; // collé au bord gauche du mur
          vx = 0;
          return;
        }
      }
    } else {
      final int col = (b.left / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = (col + 1) * carte.taille;     // collé au bord droit du mur
          vx = 0;
          return;
        }
      }
    }
  }

  void _resoudreY(Tilemap carte) {
    if (vy == 0) return;
    const double eps = 0.001;
    final Rect b = boite;
    final int colGauche = (b.left / carte.taille).floor();
    final int colDroite = ((b.right - eps) / carte.taille).floor();

    if (vy > 0) {
      final int lig = ((b.bottom - eps) / carte.taille).floor();
      for (int c = colGauche; c <= colDroite; c++) {
        if (carte.solideA(c, lig)) {
          y = lig * carte.taille - hauteur;
          vy = 0;
          auSol = true;
          return;
        }
      }
    } else {
      final int lig = (b.top / carte.taille).floor();
      for (int c = colGauche; c <= colDroite; c++) {
        if (carte.solideA(c, lig)) {
          y = (lig + 1) * carte.taille;
          vy = 0;                           // on se cogne au plafond
          return;
        }
      }
    }
  }
}
```

### Comprendre les deux formules de repositionnement

Ce sont les deux lignes que l'on écrit de travers une fois sur deux.

```text
  COLLISION À DROITE (vx > 0)

  on veut : bord droit du héros = bord gauche du mur
            x + largeur = col * taille
  donc      x = col * taille - largeur


  COLLISION À GAUCHE (vx < 0)

  on veut : bord gauche du héros = bord droit du mur
            x = (col + 1) * taille
```

Si vous inversez, le héros est téléporté à l'intérieur du mur et le bug est spectaculaire.

### Pourquoi `auSol = false` avant `_resoudreY`

On remet le drapeau à faux **à chaque frame**, puis on ne le repasse à vrai que si une collision par le bas a effectivement eu lieu. Sans cette remise à zéro, le héros resterait éternellement « au sol » après son premier atterrissage, et pourrait sauter en plein vide indéfiniment.

### Les limites de cette méthode

Elle suppose que le héros se déplace de **moins d'une tuile par frame**. À 60 FPS avec des tuiles de 32 pixels, cela autorise jusqu'à 1 920 pixels par seconde : largement suffisant. Mais un projectile rapide, ou un gros pic de lag, traversera le mur : c'est le **tunneling** du chapitre 24. La parade habituelle est de plafonner le déplacement par frame :

```dart
final double pasMax = carte.taille * 0.9;
final double dx = (vx * dt).clamp(-pasMax, pasMax);
x += dx;
```

> **À retenir.** X d'abord, puis Y. Repositionner, annuler la vitesse de l'axe, lever `auSol` uniquement sur une collision par le bas.

---

## 25.25 — Représenter un niveau avec une carte en texte (`List<String>`)

Écrire un niveau en `List<List<int>>` est correct, mais illisible. Comparez :

```dart
// Illisible.
<int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
<int>[1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
<int>[1, 0, 0, 2, 2, 2, 0, 0, 0, 0, 0, 1],
```

```dart
// Lisible d'un coup d'œil.
'############',
'#..........#',
'#..===.....#',
```

La deuxième forme est un **dessin**. On y voit la salle. On peut la modifier dans n'importe quel éditeur de texte, l'envoyer par message, la relire six mois plus tard.

### La légende

On associe un caractère à chaque index de tuile. C'est une `Map<String, int>`, structure du chapitre 06.

```dart
const Map<String, int> kLegende = <String, int>{
  '.': 0, // vide
  '#': 1, // mur de pierre
  '=': 2, // plateforme de bois
  '^': 3, // pointes
  '~': 4, // eau
};
```

### La conversion

```dart
/// Convertit une carte texte en grille d'entiers.
/// Lève une exception claire si un caractère est inconnu
/// ou si les lignes n'ont pas la même longueur.
List<List<int>> grilleDepuisTexte(
  List<String> plan,
  Map<String, int> legende,
) {
  if (plan.isEmpty) {
    throw ArgumentError('Le plan du niveau est vide.');
  }

  final int largeur = plan.first.length;
  final List<List<int>> grille = <List<int>>[];

  for (int l = 0; l < plan.length; l++) {
    final String ligne = plan[l];
    if (ligne.length != largeur) {
      throw FormatException(
        'Ligne $l de longueur ${ligne.length}, attendu $largeur.',
      );
    }

    final List<int> cases = <int>[];
    for (int c = 0; c < ligne.length; c++) {
      final String caractere = ligne[c];
      final int? index = legende[caractere];
      if (index == null) {
        throw FormatException(
          'Caractere inconnu "$caractere" en ligne $l, colonne $c.',
        );
      }
      cases.add(index);
    }
    grille.add(cases);
  }

  return grille;
}
```

Les deux `throw` ne sont pas décoratifs. Ce sont les deux erreurs que vous ferez :

- **lignes de longueurs différentes** : un espace en trop en fin de ligne, et votre niveau devient bancal ;
- **caractère inconnu** : une faute de frappe, un `O` majuscule pris pour un zéro.

Sans exception, ces erreurs produisent des bugs silencieux. Avec exception, le message vous donne la ligne et la colonne. C'est exactement l'esprit du chapitre 13.

### Programme de démonstration

```dart
const Map<String, int> kLegende = <String, int>{
  '.': 0,
  '#': 1,
  '=': 2,
  '^': 3,
};

List<List<int>> grilleDepuisTexte(List<String> plan, Map<String, int> legende) {
  if (plan.isEmpty) throw ArgumentError('Le plan du niveau est vide.');
  final int largeur = plan.first.length;
  final List<List<int>> grille = <List<int>>[];
  for (int l = 0; l < plan.length; l++) {
    final String ligne = plan[l];
    if (ligne.length != largeur) {
      throw FormatException(
          'Ligne $l de longueur ${ligne.length}, attendu $largeur.');
    }
    final List<int> cases = <int>[];
    for (int c = 0; c < ligne.length; c++) {
      final int? index = legende[ligne[c]];
      if (index == null) {
        throw FormatException(
            'Caractere inconnu "${ligne[c]}" en ligne $l, colonne $c.');
      }
      cases.add(index);
    }
    grille.add(cases);
  }
  return grille;
}

void main() {
  const List<String> salle = <String>[
    '################',
    '#..............#',
    '#..===.........#',
    '#.........===..#',
    '#..............#',
    '#....^^^.......#',
    '################',
  ];

  final List<List<int>> grille = grilleDepuisTexte(salle, kLegende);
  print('Grille : ${grille[0].length} x ${grille.length}');
  print('Ligne 2 : ${grille[2]}');
  print('Ligne 5 : ${grille[5]}');

  // Erreur volontaire : ligne trop courte.
  try {
    grilleDepuisTexte(<String>['####', '#..#', '###'], kLegende);
  } on FormatException catch (e) {
    print('Erreur attrapee : ${e.message}');
  }

  // Erreur volontaire : caractère inconnu.
  try {
    grilleDepuisTexte(<String>['####', '#.@#', '####'], kLegende);
  } on FormatException catch (e) {
    print('Erreur attrapee : ${e.message}');
  }
}
```

**Résultat :**

```text
Grille : 16 x 7
Ligne 2 : [1, 0, 0, 2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1]
Ligne 5 : [1, 0, 0, 0, 0, 3, 3, 3, 0, 0, 0, 0, 0, 0, 0, 1]
Erreur attrapee : Ligne 2 de longueur 3, attendu 4.
Erreur attrapee : Caractere inconnu "@" en ligne 1, colonne 2.
```

> **À retenir.** Écrivez vos niveaux en texte, convertissez-les en entiers au chargement, et validez agressivement. Un niveau mal formé doit planter au démarrage, pas produire un mur invisible au milieu de la partie.

---

## 25.26 — Charger un niveau depuis un fichier JSON (rappel du chapitre 17)

Une carte en dur dans le code, c'est bien pour prototyper. Pour un vrai jeu, on veut :

- ajouter un niveau sans recompiler ;
- laisser un game designer éditer les niveaux ;
- télécharger des niveaux supplémentaires.

Le JSON du chapitre 17 est la réponse naturelle.

### Le format

```json
{
  "nom": "Salle des gardes",
  "tailleTuile": 32,
  "legende": { ".": 0, "#": 1, "=": 2, "^": 3 },
  "plan": [
    "################",
    "#..............#",
    "#..===.........#",
    "#.........===..#",
    "#....^^^.......#",
    "################"
  ],
  "objets": [
    { "type": "depart",  "col": 2,  "lig": 4 },
    { "type": "coffre",  "col": 12, "lig": 4 },
    { "type": "gobelin", "col": 8,  "lig": 4 },
    { "type": "porte",   "col": 14, "lig": 4, "vers": "salle-2" }
  ]
}
```

Ce format coche toutes les cases : lisible, éditable à la main, extensible (ajouter un champ ne casse rien), et directement convertible.

### Le modèle Dart

On applique la méthode du chapitre 17 : une classe par entité, un constructeur nommé `fromJson`, une méthode `toJson`.

```dart
import 'dart:convert';

class ObjetNiveau {
  const ObjetNiveau({
    required this.type,
    required this.col,
    required this.lig,
    this.vers,
  });

  final String type;
  final int col;
  final int lig;

  /// Pour les portes : identifiant du niveau de destination.
  final String? vers;

  factory ObjetNiveau.fromJson(Map<String, dynamic> json) {
    return ObjetNiveau(
      type: json['type'] as String,
      col: json['col'] as int,
      lig: json['lig'] as int,
      vers: json['vers'] as String?,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'type': type,
        'col': col,
        'lig': lig,
        if (vers != null) 'vers': vers,
      };

  @override
  String toString() => '$type($col,$lig)${vers == null ? '' : ' -> $vers'}';
}

class Niveau {
  Niveau({
    required this.nom,
    required this.tailleTuile,
    required this.grille,
    required this.objets,
  });

  final String nom;
  final double tailleTuile;
  final List<List<int>> grille;
  final List<ObjetNiveau> objets;

  int get colonnes => grille.isEmpty ? 0 : grille[0].length;
  int get lignes => grille.length;
  double get largeurMonde => colonnes * tailleTuile;
  double get hauteurMonde => lignes * tailleTuile;

  factory Niveau.fromJson(Map<String, dynamic> json) {
    // 1. La légende : Map<String, dynamic> -> Map<String, int>
    final Map<String, dynamic> legendeBrute =
        json['legende'] as Map<String, dynamic>;
    final Map<String, int> legende = legendeBrute.map(
      (String k, dynamic v) => MapEntry<String, int>(k, v as int),
    );

    // 2. Le plan texte -> grille d'entiers.
    final List<String> plan =
        (json['plan'] as List<dynamic>).cast<String>();
    final List<List<int>> grille = <List<int>>[];
    for (int l = 0; l < plan.length; l++) {
      final List<int> cases = <int>[];
      for (int c = 0; c < plan[l].length; c++) {
        final int? index = legende[plan[l][c]];
        if (index == null) {
          throw FormatException(
              'Caractere inconnu "${plan[l][c]}" ligne $l colonne $c.');
        }
        cases.add(index);
      }
      grille.add(cases);
    }

    // 3. Les objets.
    final List<ObjetNiveau> objets = (json['objets'] as List<dynamic>? ?? <dynamic>[])
        .map((dynamic o) => ObjetNiveau.fromJson(o as Map<String, dynamic>))
        .toList();

    return Niveau(
      nom: json['nom'] as String,
      tailleTuile: (json['tailleTuile'] as num).toDouble(),
      grille: grille,
      objets: objets,
    );
  }

  /// Position monde du centre d'une case.
  Offset centreDe(int col, int lig) => Offset(
        (col + 0.5) * tailleTuile,
        (lig + 0.5) * tailleTuile,
      );
}
```

Notez deux détails que le chapitre 17 avait signalés.

**`(json['tailleTuile'] as num).toDouble()`.** En JSON, `32` est décodé en `int`, `32.0` en `double`. Un cast direct `as double` planterait sur `32`. Passer par `num` couvre les deux cas. C'est une source de plantage très fréquente.

**`json['objets'] as List<dynamic>? ?? <dynamic>[]`.** Si le champ `objets` est absent, on prend une liste vide plutôt que de planter. Un niveau sans objet reste un niveau valide.

### Charger depuis une chaîne, puis depuis un fichier

Testons d'abord sans aucun fichier, pour vérifier le modèle.

```dart
import 'dart:convert';
import 'dart:ui';

// ... classes ObjetNiveau et Niveau ci-dessus ...

const String kJsonSalle1 = '''
{
  "nom": "Salle des gardes",
  "tailleTuile": 32,
  "legende": { ".": 0, "#": 1, "=": 2, "^": 3 },
  "plan": [
    "################",
    "#..............#",
    "#..===.........#",
    "#.........===..#",
    "#....^^^.......#",
    "################"
  ],
  "objets": [
    { "type": "depart",  "col": 2,  "lig": 4 },
    { "type": "coffre",  "col": 12, "lig": 4 },
    { "type": "gobelin", "col": 8,  "lig": 4 },
    { "type": "porte",   "col": 14, "lig": 4, "vers": "salle-2" }
  ]
}
''';

void main() {
  final Map<String, dynamic> brut =
      jsonDecode(kJsonSalle1) as Map<String, dynamic>;
  final Niveau niveau = Niveau.fromJson(brut);

  print('Niveau  : ${niveau.nom}');
  print('Grille  : ${niveau.colonnes} x ${niveau.lignes}');
  print('Monde   : ${niveau.largeurMonde} x ${niveau.hauteurMonde} px');
  print('Objets  :');
  for (final ObjetNiveau o in niveau.objets) {
    print('  - $o  centre monde ${niveau.centreDe(o.col, o.lig)}');
  }
}
```

**Résultat :**

```text
Niveau  : Salle des gardes
Grille  : 16 x 6
Monde   : 512.0 x 192.0 px
Objets  :
  - depart(2,4)  centre monde Offset(80.0, 144.0)
  - coffre(12,4)  centre monde Offset(400.0, 144.0)
  - gobelin(8,4)  centre monde Offset(272.0, 144.0)
  - porte(14,4) -> salle-2  centre monde Offset(464.0, 144.0)
```

### Depuis un vrai fichier d'assets

Dans une application Flutter, les fichiers de niveau vivent dans `assets/`. Déclarez-les dans `pubspec.yaml` (chapitre 16) :

```yaml
flutter:
  assets:
    - assets/niveaux/salle-1.json
    - assets/niveaux/salle-2.json
```

Puis chargez-les de façon asynchrone (chapitre 15) :

```dart
import 'package:flutter/services.dart' show rootBundle;

Future<Niveau> chargerNiveau(String identifiant) async {
  final String contenu =
      await rootBundle.loadString('assets/niveaux/$identifiant.json');
  final Map<String, dynamic> json =
      jsonDecode(contenu) as Map<String, dynamic>;
  return Niveau.fromJson(json);
}
```

Et, comme au chapitre 22, **chargez avant de démarrer la boucle**. Un `await` au milieu d'une frame produirait un affichage vide pendant un instant.

```dart
Future<void> _demarrer() async {
  final Niveau niveau = await chargerNiveau('salle-1');
  if (!mounted) return;
  setState(() => _niveau = niveau);
  _ticker.start();
}
```

> **À retenir.** JSON pour les niveaux, `fromJson` pour le modèle, `num.toDouble()` pour les nombres, chargement asynchrone avant le premier `tick`.

---

## 25.27 — Les objets du niveau : point d'apparition, portes, coffres, ennemis

Une grille de tuiles décrit la géométrie. Elle ne décrit pas le **contenu** : où le héros apparaît, où sont les gobelins, quelle porte mène où. C'est le rôle du calque d'objets.

### Pourquoi ne pas mettre les objets dans la grille ?

On pourrait donner l'index 9 au coffre et le poser dans la tilemap. C'est une mauvaise idée, pour trois raisons.

1. **Un objet a des propriétés.** Une porte a une destination, un coffre a un contenu, un gobelin a des points de vie. Un `int` ne peut pas les porter.
2. **Un objet bouge.** Le gobelin patrouille, le coffre s'ouvre. La grille est statique par nature.
3. **Un objet disparaît.** Une potion ramassée doit être retirée. Modifier la tilemap pour cela mélange décor et logique.

La règle : **la grille pour ce qui est fixe et solide, la liste d'objets pour ce qui vit.**

### Les types du Donjon de Dart

| Type | Rôle | Champs supplémentaires |
| --- | --- | --- |
| `depart` | position d'apparition du héros | — |
| `porte` | transition vers un autre niveau | `vers` |
| `coffre` | contient un objet | `contenu` |
| `gobelin` | ennemi de base | `pv`, `portee` |
| `potion` | soigne le héros | `soin` |
| `cle` | ouvre une porte verrouillée | `couleur` |
| `boss` | ennemi final | `pv` |

### Le point d'apparition

C'est l'objet le plus important, et celui qu'on oublie. Sans lui, le héros démarre en `(0, 0)`, c'est-à-dire dans le mur du coin haut-gauche.

```dart
/// Cherche le point d'apparition. Lève une exception si le niveau n'en a pas.
Offset trouverDepart(Niveau niveau) {
  for (final ObjetNiveau o in niveau.objets) {
    if (o.type == 'depart') {
      // On aligne le héros sur le BAS de la case, pas sur son centre :
      // sinon il commence encastré dans le sol.
      return Offset(
        o.col * niveau.tailleTuile,
        (o.lig + 1) * niveau.tailleTuile - Heros.hauteur,
      );
    }
  }
  throw StateError(
      'Le niveau "${niveau.nom}" n a pas d objet de type "depart".');
}
```

Le calcul vertical mérite un mot : `(lig + 1) * taille` est le **bas** de la case ; on remonte de la hauteur du héros pour poser ses pieds dessus. Si vous utilisez le centre de la case, le héros démarre à moitié dans le sol et la résolution de collision le fait sursauter à la première frame.

### Transformer les objets en entités

Le chargement produit des `ObjetNiveau` (données brutes). Le jeu a besoin d'entités vivantes. On les fabrique une fois, au démarrage du niveau — c'est une petite **fabrique**, au sens du chapitre 09.

```dart
class Gobelin {
  Gobelin({required this.x, required this.y, this.pv = 3});
  double x;
  double y;
  int pv;
  double vx = 40;
  Rect get boite => Rect.fromLTWH(x, y, 26, 30);
}

class Coffre {
  Coffre({required this.x, required this.y, required this.contenu});
  final double x;
  final double y;
  final String contenu;
  bool ouvert = false;
  Rect get boite => Rect.fromLTWH(x, y, 30, 26);
}

class Porte {
  Porte({required this.x, required this.y, required this.vers});
  final double x;
  final double y;
  final String vers;
  Rect get boite => Rect.fromLTWH(x, y, 32, 64);
}

class ContenuNiveau {
  ContenuNiveau({
    required this.depart,
    required this.gobelins,
    required this.coffres,
    required this.portes,
  });

  final Offset depart;
  final List<Gobelin> gobelins;
  final List<Coffre> coffres;
  final List<Porte> portes;
}

ContenuNiveau construireContenu(Niveau niveau) {
  Offset? depart;
  final List<Gobelin> gobelins = <Gobelin>[];
  final List<Coffre> coffres = <Coffre>[];
  final List<Porte> portes = <Porte>[];
  final double t = niveau.tailleTuile;

  for (final ObjetNiveau o in niveau.objets) {
    final double x = o.col * t;
    final double basCase = (o.lig + 1) * t;

    switch (o.type) {
      case 'depart':
        depart = Offset(x, basCase - Heros.hauteur);
      case 'gobelin':
        gobelins.add(Gobelin(x: x, y: basCase - 30));
      case 'coffre':
        coffres.add(Coffre(x: x, y: basCase - 26, contenu: 'potion'));
      case 'porte':
        portes.add(Porte(x: x, y: basCase - 64, vers: o.vers ?? ''));
      default:
        // Un type inconnu ne doit pas faire planter le jeu,
        // mais il doit se voir en développement.
        assert(false, 'Type d objet inconnu : ${o.type}');
    }
  }

  if (depart == null) {
    throw StateError('Niveau "${niveau.nom}" sans point de depart.');
  }

  return ContenuNiveau(
    depart: depart,
    gobelins: gobelins,
    coffres: coffres,
    portes: portes,
  );
}
```

Le `switch` sur `String` avec des cas sans `break` est la syntaxe moderne de Dart 3 (chapitre 04). Le `assert` du `default` est une bonne pratique : il crie en développement et se tait en production.

> **À retenir.** La grille décrit le décor, la liste d'objets décrit le contenu. Convertissez les objets en entités une seule fois, au chargement.

---

## 25.28 — L'éditeur Tiled : présentation et format `.tmx`

Écrire des niveaux à la main en `List<String>` fonctionne jusqu'à une trentaine de colonnes. Au-delà, on veut un éditeur graphique. Le standard libre du monde 2D s'appelle **Tiled** (`mapeditor.org`), gratuit, disponible sur Windows, macOS et Linux.

### Ce que Tiled apporte

| Fonction | Bénéfice |
| --- | --- |
| Pinceau, remplissage, sélection | dessiner un niveau au lieu de le taper |
| Calques multiples | fond, collision, décor avant, objets |
| Tilesets images | voir les vraies tuiles pendant l'édition |
| Calque d'objets | poser des rectangles nommés (départ, portes, zones) |
| Propriétés personnalisées | ajouter `pv`, `vers`, `contenu` à n'importe quel objet |
| Export | `.tmx` (XML) ou `.json` |

### À quoi ressemble un `.tmx`

Un fichier `.tmx` est du XML. En voici une version très réduite, pour que vous reconnaissiez la structure quand vous en ouvrirez un :

```text
<?xml version="1.0" encoding="UTF-8"?>
<map version="1.10" orientation="orthogonal" renderorder="right-down"
     width="16" height="6" tilewidth="32" tileheight="32">

  <tileset firstgid="1" source="donjon.tsx"/>

  <layer id="1" name="collision" width="16" height="6">
    <data encoding="csv">
      1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
      1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,
      1,0,0,3,3,3,0,0,0,0,0,0,0,0,0,1,
      1,0,0,0,0,0,0,0,0,0,3,3,3,0,0,1,
      1,0,0,0,0,4,4,4,0,0,0,0,0,0,0,1,
      1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
    </data>
  </layer>

  <objectgroup id="2" name="objets">
    <object id="1" name="depart" type="depart" x="64" y="160"/>
    <object id="2" name="porte"  type="porte"  x="448" y="160">
      <properties>
        <property name="vers" value="salle-2"/>
      </properties>
    </object>
  </objectgroup>
</map>
```

Trois éléments à repérer.

**`tilewidth` et `tileheight`** : c'est votre `tailleTuile`.

**`<data encoding="csv">`** : la grille, ligne par ligne, séparée par des virgules. C'est exactement votre `List<List<int>>`, à un détail près.

**`firstgid="1"`** : Tiled réserve **le 0 pour le vide** et numérote les tuiles à partir de 1. Un index Tiled de 3 correspond donc à la troisième tuile du tileset, pas à la quatrième. C'est le décalage qui surprend tout le monde la première fois.

### Le piège du `y` des objets

Dans un calque d'objets Tiled, la coordonnée `y` d'un objet ponctuel désigne son **bas**, pas son haut. Un objet à `y="160"` avec un sprite de 40 pixels de haut occupe donc `[120, 160]`. Si vous l'ignorez, tous vos objets sont décalés d'une hauteur vers le bas.

### Faut-il lire le `.tmx` soi-même ?

Non, et c'est le message de cette section. Écrire un parseur XML complet est un travail ingrat. Deux solutions raisonnables :

1. **Exporter en JSON depuis Tiled** (Fichier → Exporter en → JSON) et réutiliser le code de la section 25.26, à peine adapté.
2. **Utiliser `flame_tiled`**, le paquet officiel qui lit les `.tmx` et construit les calques automatiquement. C'est le sujet du **chapitre 34**.

Ce que vous avez écrit dans ce chapitre n'est pas perdu pour autant : `flame_tiled` vous donnera les mêmes grilles d'entiers, et vous saurez exactement quoi en faire.

> **À retenir.** Tiled est l'outil standard, `.tmx` est du XML, l'index 0 est le vide, et l'on ne parse pas le XML soi-même : on exporte en JSON ou on attend le chapitre 34.

---

## 25.29 — Passer d'un niveau au suivant

Le héros atteint la porte de droite. Que se passe-t-il, exactement ?

### La séquence complète

```text
  1. DÉTECTION      le héros chevauche la hitbox d'une porte
  2. VERROU         on lève un drapeau "transition en cours"
                    (sinon la détection se répète 60 fois par seconde)
  3. SORTIE         fondu au noir, le jeu continue de tourner
  4. CHARGEMENT     on charge le niveau destination (await)
  5. PLACEMENT      on pose le héros au point d'arrivée
  6. CAMÉRA         on la place IMMÉDIATEMENT, sans lissage
  7. ENTRÉE         fondu depuis le noir
  8. DÉVERROU       on abaisse le drapeau
```

Les étapes 2 et 6 sont celles qu'on oublie, et elles produisent chacune un bug caractéristique.

**Sans verrou :** le héros entre dans la porte, on charge le niveau, mais la frame suivante il chevauche encore la porte (ou la porte d'arrivée), et on recharge. Le jeu se bloque dans une boucle de chargement.

**Sans placement immédiat de la caméra :** au chargement de la nouvelle salle, la caméra est encore à l'ancienne position et « vole » à travers tout le niveau pendant une seconde. C'est très visible et très laid.

### Le gestionnaire de niveaux

```dart
class GestionnaireNiveaux {
  GestionnaireNiveaux({required this.chargeur});

  /// Fonction qui sait charger un niveau par identifiant.
  final Future<Niveau> Function(String id) chargeur;

  Niveau? courant;
  bool enTransition = false;

  /// Progression du fondu : 0 = visible, 1 = noir complet.
  double fondu = 0;

  String? _destination;

  /// Demande une transition vers [id]. Ignorée si une transition est en cours.
  void demanderTransition(String id) {
    if (enTransition) return;
    enTransition = true;
    _destination = id;
  }

  Future<void> mettreAJour(double dt, void Function(Niveau) onCharge) async {
    if (!enTransition) {
      // Fondu de retour vers la visibilité.
      fondu = (fondu - 3 * dt).clamp(0.0, 1.0);
      return;
    }

    // Phase de sortie : on assombrit.
    if (fondu < 1) {
      fondu = (fondu + 3 * dt).clamp(0.0, 1.0);
      return;
    }

    // Écran noir atteint : on charge.
    final String? id = _destination;
    if (id == null) {
      enTransition = false;
      return;
    }
    _destination = null;

    final Niveau nouveau = await chargeur(id);
    courant = nouveau;
    onCharge(nouveau);
    enTransition = false;
  }
}
```

Et le rendu du fondu, en coordonnées **écran**, donc après `camera.retirer(canvas)` :

```dart
if (gestionnaire.fondu > 0) {
  canvas.drawRect(
    Offset.zero & size,
    Paint()..color = Color.fromRGBO(0, 0, 0, gestionnaire.fondu.clamp(0, 1)),
  );
}
```

### Le point d'arrivée

Une porte doit savoir où déposer le héros dans le niveau suivant. Deux conventions courantes :

**Convention A — point d'arrivée nommé.** La porte porte `vers: "salle-2"` et `arrivee: "porte-ouest"`. Le niveau 2 contient un objet `arrivee` nommé `porte-ouest`. C'est explicite et robuste.

**Convention B — le point de départ par défaut.** Le héros arrive toujours sur l'objet `depart` du niveau. C'est simple, mais impossible de revenir en arrière proprement.

Pour le Donjon de Dart, nous adoptons la convention A dans le format JSON :

```json
{ "type": "porte", "col": 14, "lig": 4, "vers": "salle-2", "arrivee": "ouest" },
{ "type": "arrivee", "col": 1, "lig": 4, "nom": "est" }
```

### Réinitialiser ce qu'il faut, et seulement cela

| Élément | Au changement de niveau |
| --- | --- |
| Position du héros | remise au point d'arrivée |
| Vitesse du héros | remise à zéro |
| Points de vie | **conservés** |
| Inventaire, clés | **conservés** |
| Score | **conservé** |
| Gobelins, coffres | reconstruits depuis le nouveau niveau |
| Caméra | placée immédiatement, sans lissage |
| Secousse de caméra | remise à zéro |

L'erreur classique est de tout remettre à zéro en rechargeant, y compris les points de vie : le joueur se soigne alors en franchissant une porte. L'erreur inverse est de conserver la liste des gobelins d'une salle à l'autre.

> **À retenir.** Verrou, fondu, chargement, placement immédiat de la caméra. Et distinguez ce qui appartient au **joueur** (conservé) de ce qui appartient au **niveau** (reconstruit).

---

## 25.30 — Mini-projet : un donjon de plusieurs salles avec caméra qui suit le joueur

Nous assemblons tout le chapitre en un seul programme. Voici le cahier des charges.

| Exigence | Section d'origine |
| --- | --- |
| Trois salles de 60 sur 16 tuiles, soit 1 920 x 512 pixels | 25.20 |
| Niveaux écrits en texte lisible | 25.25 |
| Caméra qui suit le héros en lissé | 25.9 |
| Zone morte pour ignorer les micro-mouvements | 25.10 |
| Look-ahead horizontal | 25.11 |
| Caméra bornée à la salle | 25.12 |
| Tremblement à chaque dégât | 25.14 |
| Culling des tuiles et des entités | 25.15, 25.22 |
| Deux plans de parallaxe | 25.17 |
| Collision héros / tilemap axe par axe | 25.24 |
| Objets : départ, arrivées, gobelins, coffres, portes, boss | 25.27 |
| Portes avec fondu et verrou de transition | 25.29 |

Le programme se pilote au clavier (flèches et espace) sur ordinateur, et par trois boutons tactiles sur téléphone.

### Le code complet

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter/services.dart';

void main() => runApp(const AppliDonjon());

// =====================================================================
// CONSTANTES
// =====================================================================

/// Côté d'une tuile, en pixels monde. Une seule source de vérité.
const double kTuile = 32;

/// Correspondance caractère du plan -> index de tuile.
const Map<String, int> kLegende = <String, int>{
  '.': 0, // vide
  '#': 1, // mur de pierre
  '=': 2, // plateforme de bois
  '^': 3, // pointes
};

// =====================================================================
// LES PLANS DES SALLES (60 colonnes x 16 lignes)
// =====================================================================

const Map<String, List<String>> kPlans = <String, List<String>>{
  'salle-1': <String>[
    '############################################################',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#.......................................=======............#',
    '#.................=======..................................#',
    '#..........................................................#',
    '#.............................======.......................#',
    '#.......======.............................................#',
    '#................................................##........#',
    '#........................^^^....................###........#',
    '############################################################',
    '############################################################',
  ],
  'salle-2': <String>[
    '############################################################',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#...............................======.....................#',
    '#..........................................................#',
    '#.....................======...............................#',
    '#.............................................======.......#',
    '#...............###........................................#',
    '#............######........................................#',
    '#.........#########.....................^^^^^..............#',
    '############################################################',
    '############################################################',
  ],
  'salle-3': <String>[
    '############################################################',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#..........................................................#',
    '#...........................======.........................#',
    '#...................##..................##.................#',
    '#...................##..................##.................#',
    '#...................##..................##.................#',
    '#...................##..................##....^^^..........#',
    '############################################################',
    '############################################################',
  ],
};

// =====================================================================
// LES OBJETS DE CHAQUE SALLE
// =====================================================================

class DefObjet {
  const DefObjet(this.type, this.col, this.lig, {this.vers, this.nom});

  final String type;
  final int col;
  final int lig;

  /// Pour une porte : "identifiantSalle:nomArrivee".
  final String? vers;

  /// Pour une arrivée : son nom.
  final String? nom;
}

const Map<String, List<DefObjet>> kObjets = <String, List<DefObjet>>{
  'salle-1': <DefObjet>[
    DefObjet('depart', 3, 13),
    DefObjet('arrivee', 5, 13, nom: 'ouest'),
    DefObjet('arrivee', 52, 13, nom: 'est'),
    DefObjet('gobelin', 20, 13),
    DefObjet('gobelin', 36, 13),
    DefObjet('coffre', 44, 13),
    DefObjet('porte', 57, 13, vers: 'salle-2:ouest'),
  ],
  'salle-2': <DefObjet>[
    DefObjet('depart', 5, 13),
    DefObjet('arrivee', 5, 13, nom: 'ouest'),
    DefObjet('arrivee', 52, 13, nom: 'est'),
    DefObjet('porte', 1, 13, vers: 'salle-1:est'),
    DefObjet('gobelin', 24, 13),
    DefObjet('gobelin', 34, 13),
    DefObjet('gobelin', 54, 13),
    DefObjet('coffre', 30, 13),
    DefObjet('porte', 57, 13, vers: 'salle-3:ouest'),
  ],
  'salle-3': <DefObjet>[
    DefObjet('depart', 5, 13),
    DefObjet('arrivee', 5, 13, nom: 'ouest'),
    DefObjet('porte', 1, 13, vers: 'salle-2:est'),
    DefObjet('coffre', 12, 13),
    DefObjet('boss', 50, 13),
  ],
};

// =====================================================================
// TILEMAP
// =====================================================================

class Tilemap {
  Tilemap({required this.tuiles, this.taille = kTuile});

  /// tuiles[ligne][colonne]
  final List<List<int>> tuiles;
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;
  double get largeurMonde => colonnes * taille;
  double get hauteurMonde => lignes * taille;

  /// Hors carte : on renvoie 1, donc du mur. Le monde est clos.
  int tuileA(int c, int l) {
    if (l < 0 || l >= lignes) return 1;
    if (c < 0 || c >= colonnes) return 1;
    return tuiles[l][c];
  }

  bool solideA(int c, int l) => tuileA(c, l) != 0;

  Rect rectDe(int c, int l) =>
      Rect.fromLTWH(c * taille, l * taille, taille, taille);

  static Tilemap depuisTexte(List<String> plan) {
    final List<List<int>> grille = <List<int>>[];
    for (int l = 0; l < plan.length; l++) {
      final List<int> cases = <int>[];
      for (int c = 0; c < plan[l].length; c++) {
        final int? index = kLegende[plan[l][c]];
        if (index == null) {
          throw FormatException(
              'Caractere inconnu "${plan[l][c]}" ligne $l colonne $c.');
        }
        cases.add(index);
      }
      grille.add(cases);
    }
    return Tilemap(tuiles: grille);
  }
}

// =====================================================================
// CAMÉRA
// =====================================================================

class Secousse {
  Secousse({int graine = 1907}) : _alea = math.Random(graine);

  final math.Random _alea;
  double _intensite = 0;
  double amortissement = 26;
  double decalageX = 0;
  double decalageY = 0;

  bool get active => _intensite > 0.01;

  void declencher(double intensite) {
    if (intensite > _intensite) _intensite = intensite;
  }

  void reinitialiser() {
    _intensite = 0;
    decalageX = 0;
    decalageY = 0;
  }

  void mettreAJour(double dt) {
    if (!active) {
      reinitialiser();
      return;
    }
    decalageX = (_alea.nextDouble() * 2 - 1) * _intensite;
    decalageY = (_alea.nextDouble() * 2 - 1) * _intensite;
    _intensite = math.max(0, _intensite - amortissement * dt);
  }
}

class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    this.largeur = 0,
    this.hauteur = 0,
    this.zoom = 1,
    this.vitesseSuivi = 7,
  });

  double x;
  double y;
  double largeur;
  double hauteur;
  double zoom;
  double vitesseSuivi;

  /// Point du monde à suivre.
  Offset cible = Offset.zero;

  /// Look-ahead horizontal courant.
  double avance = 0;
  double avanceMax = 110;
  double vitesseAvance = 3;

  /// Demi-dimensions de la zone morte, en pixels écran.
  double zoneMorteX = 90;
  double zoneMorteY = 70;

  final Secousse secousse = Secousse();

  double get vueLargeur => largeur / zoom;
  double get vueHauteur => hauteur / zoom;
  Rect get vue => Rect.fromLTWH(x, y, vueLargeur, vueHauteur);
  Rect get vueElargie => vue.inflate(96);

  Offset versEcran(Offset m) => Offset((m.dx - x) * zoom, (m.dy - y) * zoom);
  Offset versMonde(Offset e) => Offset(e.dx / zoom + x, e.dy / zoom + y);

  void placerImmediatement() {
    x = cible.dx - vueLargeur / 2;
    y = cible.dy - vueHauteur / 2;
  }

  void mettreAJourAvance(double directionX, double dt) {
    final double souhaitee = directionX * avanceMax;
    final double t = 1 - math.exp(-vitesseAvance * dt);
    avance += (souhaitee - avance) * t;
  }

  void mettreAJour(double dt) {
    // 1. Cible corrigée par la zone morte (on part de la position actuelle).
    double cibleX = x;
    double cibleY = y;

    final Offset e = versEcran(cible);
    final double gauche = largeur / 2 - zoneMorteX;
    final double droite = largeur / 2 + zoneMorteX;
    final double haut = hauteur / 2 - zoneMorteY;
    final double bas = hauteur / 2 + zoneMorteY;

    if (e.dx > droite) cibleX += (e.dx - droite) / zoom;
    if (e.dx < gauche) cibleX -= (gauche - e.dx) / zoom;
    if (e.dy > bas) cibleY += (e.dy - bas) / zoom;
    if (e.dy < haut) cibleY -= (haut - e.dy) / zoom;

    // 2. Rattrapage lissé, indépendant du framerate.
    final double t = 1 - math.exp(-vitesseSuivi * dt);
    x += (cibleX - x) * t;
    y += (cibleY - y) * t;

    // 3. Secousse.
    secousse.mettreAJour(dt);
  }

  /// Toujours appelé EN DERNIER.
  void borner(double lMonde, double hMonde) {
    x = lMonde <= vueLargeur
        ? (lMonde - vueLargeur) / 2
        : x.clamp(0.0, lMonde - vueLargeur);
    y = hMonde <= vueHauteur
        ? (hMonde - vueHauteur) / 2
        : y.clamp(0.0, hMonde - vueHauteur);
  }

  void appliquer(Canvas canvas) {
    canvas.save();
    canvas.scale(zoom);
    canvas.translate(-(x + secousse.decalageX), -(y + secousse.decalageY));
  }

  void retirer(Canvas canvas) => canvas.restore();
}

// =====================================================================
// ENTITÉS
// =====================================================================

class Heros {
  Heros({required this.x, required this.y});

  static const double largeur = 24;
  static const double hauteur = 40;
  static const double vitesse = 190;
  static const double gravite = 1100;
  static const double impulsionSaut = 460;

  double x;
  double y;
  double vx = 0;
  double vy = 0;
  bool auSol = false;
  bool regardeADroite = true;

  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset get centre => Offset(x + largeur / 2, y + hauteur / 2);

  void deplacer(Tilemap carte, double dt) {
    final double pasMax = carte.taille * 0.9;

    // --- AXE X ---
    x += (vx * dt).clamp(-pasMax, pasMax);
    _resoudreX(carte);

    // --- AXE Y ---
    vy += gravite * dt;
    if (vy > 1400) vy = 1400;
    y += (vy * dt).clamp(-pasMax, pasMax);
    auSol = false;
    _resoudreY(carte);
  }

  void _resoudreX(Tilemap carte) {
    if (vx == 0) return;
    const double eps = 0.001;
    final Rect b = boite;
    final int ligHaut = (b.top / carte.taille).floor();
    final int ligBas = ((b.bottom - eps) / carte.taille).floor();

    if (vx > 0) {
      final int col = ((b.right - eps) / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = col * carte.taille - largeur;
          return;
        }
      }
    } else {
      final int col = (b.left / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = (col + 1) * carte.taille;
          return;
        }
      }
    }
  }

  void _resoudreY(Tilemap carte) {
    const double eps = 0.001;
    final Rect b = boite;
    final int colG = (b.left / carte.taille).floor();
    final int colD = ((b.right - eps) / carte.taille).floor();

    if (vy > 0) {
      final int lig = ((b.bottom - eps) / carte.taille).floor();
      for (int c = colG; c <= colD; c++) {
        if (carte.solideA(c, lig)) {
          y = lig * carte.taille - hauteur;
          vy = 0;
          auSol = true;
          return;
        }
      }
    } else if (vy < 0) {
      final int lig = (b.top / carte.taille).floor();
      for (int c = colG; c <= colD; c++) {
        if (carte.solideA(c, lig)) {
          y = (lig + 1) * carte.taille;
          vy = 0;
          return;
        }
      }
    }
  }
}

class Gobelin {
  Gobelin({required this.x, required this.y, this.boss = false});

  double x;
  double y;
  final bool boss;
  late double vx = boss ? 70 : 45;

  double get largeur => boss ? 48 : 26;
  double get hauteur => boss ? 56 : 28;
  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);

  void mettreAJour(Tilemap carte, double dt) {
    x += vx * dt;
    final Rect b = boite;
    final int lig = ((b.bottom - 1) / carte.taille).floor();
    final int colAvant = vx > 0
        ? ((b.right - 0.001) / carte.taille).floor()
        : (b.left / carte.taille).floor();

    final bool murDevant = carte.solideA(colAvant, lig);
    final bool videDevant = !carte.solideA(colAvant, lig + 1);
    if (murDevant || videDevant) {
      vx = -vx;
      x += vx * dt * 2;
    }
  }
}

class Coffre {
  Coffre({required this.x, required this.y});
  final double x;
  final double y;
  bool ouvert = false;
  Rect get boite => Rect.fromLTWH(x, y, 30, 26);
}

class Porte {
  Porte({required this.x, required this.y, required this.vers});
  final double x;
  final double y;
  final String vers;
  Rect get boite => Rect.fromLTWH(x, y, 32, 64);
}

// =====================================================================
// BOUTONS TACTILES (coordonnées écran)
// =====================================================================

Rect boutonGauche(Size s) => Rect.fromLTWH(24, s.height - 104, 76, 76);
Rect boutonDroite(Size s) => Rect.fromLTWH(112, s.height - 104, 76, 76);
Rect boutonSaut(Size s) => Rect.fromLTWH(s.width - 108, s.height - 112, 84, 84);

// =====================================================================
// L'APPLICATION
// =====================================================================

class AppliDonjon extends StatelessWidget {
  const AppliDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0B0912),
        body: EcranDonjon(),
      ),
    );
  }
}

class EcranDonjon extends StatefulWidget {
  const EcranDonjon({super.key});

  @override
  State<EcranDonjon> createState() => _EcranDonjonState();
}

class _EcranDonjonState extends State<EcranDonjon>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  final FocusNode _focus = FocusNode();
  Duration _precedent = Duration.zero;

  final Camera _camera = Camera();
  late Tilemap _carte;
  late Heros _heros;

  String _salle = 'salle-1';
  List<Gobelin> _gobelins = <Gobelin>[];
  List<Coffre> _coffres = <Coffre>[];
  List<Porte> _portes = <Porte>[];
  Map<String, Offset> _arrivees = <String, Offset>{};

  // Entrées.
  bool _clavGauche = false;
  bool _clavDroite = false;
  bool _clavSaut = false;
  final Map<int, String> _pointeurs = <int, String>{};
  Size _taille = Size.zero;

  bool get _allerGauche =>
      _clavGauche || _pointeurs.containsValue('gauche');
  bool get _allerDroite =>
      _clavDroite || _pointeurs.containsValue('droite');
  bool get _sauter => _clavSaut || _pointeurs.containsValue('saut');

  // Caméra et transition.
  bool _cameraPlacee = false;
  bool _enTransition = false;
  String? _destination;
  double _fondu = 1; // on commence en noir et on éclaircit

  // État du joueur (conservé d'une salle à l'autre).
  int _pv = 5;
  int _pieces = 0;
  double _invincible = 0;

  double _fps = 0;

  @override
  void initState() {
    super.initState();
    _chargerSalle('salle-1');
    _ticker = createTicker(_tick)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    _focus.dispose();
    super.dispose();
  }

  // -------------------------------------------------------------------
  // CHARGEMENT D'UNE SALLE
  // -------------------------------------------------------------------
  void _chargerSalle(String id, {String? arrivee}) {
    _carte = Tilemap.depuisTexte(kPlans[id]!);
    _salle = id;

    _gobelins = <Gobelin>[];
    _coffres = <Coffre>[];
    _portes = <Porte>[];
    _arrivees = <String, Offset>{};

    Offset? depart;

    for (final DefObjet o in kObjets[id]!) {
      final double x = o.col * kTuile;
      final double bas = (o.lig + 1) * kTuile;

      switch (o.type) {
        case 'depart':
          depart = Offset(x, bas - Heros.hauteur);
        case 'arrivee':
          _arrivees[o.nom!] = Offset(x, bas - Heros.hauteur);
        case 'gobelin':
          _gobelins.add(Gobelin(x: x, y: bas - 28));
        case 'boss':
          _gobelins.add(Gobelin(x: x, y: bas - 56, boss: true));
        case 'coffre':
          _coffres.add(Coffre(x: x, y: bas - 26));
        case 'porte':
          _portes.add(Porte(x: x, y: bas - 64, vers: o.vers!));
        default:
          assert(false, 'Type d objet inconnu : ${o.type}');
      }
    }

    if (depart == null) {
      throw StateError('La salle "$id" n a pas de point de depart.');
    }

    final Offset position =
        arrivee == null ? depart : (_arrivees[arrivee] ?? depart);
    _heros = Heros(x: position.dx, y: position.dy);

    // La caméra se place IMMÉDIATEMENT : pas de vol plané.
    _camera.cible = _heros.centre;
    _camera.avance = 0;
    _camera.secousse.reinitialiser();
    _cameraPlacee = false;
    if (_camera.largeur > 0) {
      _camera.placerImmediatement();
      _camera.borner(_carte.largeurMonde, _carte.hauteurMonde);
      _cameraPlacee = true;
    }
  }

  // -------------------------------------------------------------------
  // BOUCLE
  // -------------------------------------------------------------------
  void _tick(Duration horodatage) {
    double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0) return;
    if (dt > 0.05) dt = 0.05; // plafond anti-lag, chapitre 20
    if (_camera.largeur == 0) return;

    _fps = 1 / dt;

    if (_enTransition) {
      _fondu = (_fondu + 3 * dt).clamp(0.0, 1.0);
      if (_fondu >= 1) {
        final List<String> morceaux = _destination!.split(':');
        _chargerSalle(morceaux[0],
            arrivee: morceaux.length > 1 ? morceaux[1] : null);
        _destination = null;
        _enTransition = false;
      }
    } else {
      _fondu = (_fondu - 3 * dt).clamp(0.0, 1.0);
      _majJeu(dt);
    }

    setState(() {});
  }

  void _majJeu(double dt) {
    // --- ENTRÉES ---
    final double direction =
        (_allerDroite ? 1.0 : 0.0) - (_allerGauche ? 1.0 : 0.0);
    _heros.vx = direction * Heros.vitesse;
    if (direction != 0) _heros.regardeADroite = direction > 0;
    if (_sauter && _heros.auSol) {
      _heros.vy = -Heros.impulsionSaut;
      _heros.auSol = false;
    }

    // --- MONDE ---
    _heros.deplacer(_carte, dt);
    for (final Gobelin g in _gobelins) {
      g.mettreAJour(_carte, dt);
    }

    if (_invincible > 0) _invincible -= dt;

    // --- POINTES sous les pieds ---
    if (_heros.auSol) {
      final int col = (_heros.centre.dx / kTuile).floor();
      final int lig = (_heros.boite.bottom / kTuile).floor();
      if (_carte.tuileA(col, lig) == 3) {
        _blesser(1, 7);
      }
    }

    // --- GOBELINS ---
    for (final Gobelin g in _gobelins) {
      if (g.boite.overlaps(_heros.boite)) {
        _blesser(g.boss ? 2 : 1, g.boss ? 12 : 6);
        _heros.vx = 0;
        _heros.x += _heros.centre.dx < g.boite.center.dx ? -14 : 14;
      }
    }

    // --- COFFRES ---
    for (final Coffre c in _coffres) {
      if (!c.ouvert && c.boite.overlaps(_heros.boite)) {
        c.ouvert = true;
        _pieces += 10;
        _camera.secousse.declencher(2.5);
      }
    }

    // --- PORTES ---
    for (final Porte p in _portes) {
      if (p.boite.overlaps(_heros.boite)) {
        _demanderTransition(p.vers);
        break;
      }
    }

    // --- CAMÉRA (l'ordre compte) ---
    _camera.mettreAJourAvance(direction, dt);
    _camera.cible = _heros.centre + Offset(_camera.avance, -20);
    if (!_cameraPlacee) {
      // Première frame connaissant la taille de la fenêtre.
      _camera.placerImmediatement();
      _cameraPlacee = true;
    }
    _camera.mettreAJour(dt);
    _camera.borner(_carte.largeurMonde, _carte.hauteurMonde);
  }

  void _blesser(int degats, double secousse) {
    if (_invincible > 0) return;
    _pv = math.max(0, _pv - degats);
    _invincible = 1.2;
    _camera.secousse.declencher(secousse);
    if (_pv == 0) {
      _pv = 5;
      _pieces = 0;
      _demanderTransition('salle-1:ouest');
    }
  }

  void _demanderTransition(String destination) {
    if (_enTransition) return; // le verrou de la section 25.29
    _enTransition = true;
    _destination = destination;
  }

  // -------------------------------------------------------------------
  // ENTRÉES
  // -------------------------------------------------------------------
  void _majPointeur(int id, Offset p) {
    if (boutonGauche(_taille).contains(p)) {
      _pointeurs[id] = 'gauche';
    } else if (boutonDroite(_taille).contains(p)) {
      _pointeurs[id] = 'droite';
    } else if (boutonSaut(_taille).contains(p)) {
      _pointeurs[id] = 'saut';
    } else {
      _pointeurs.remove(id);
    }
  }

  void _clavier(KeyEvent evenement) {
    if (evenement is KeyRepeatEvent) return;
    final bool appui = evenement is KeyDownEvent;
    final LogicalKeyboardKey touche = evenement.logicalKey;

    if (touche == LogicalKeyboardKey.arrowLeft ||
        touche == LogicalKeyboardKey.keyQ ||
        touche == LogicalKeyboardKey.keyA) {
      _clavGauche = appui;
    } else if (touche == LogicalKeyboardKey.arrowRight ||
        touche == LogicalKeyboardKey.keyD) {
      _clavDroite = appui;
    } else if (touche == LogicalKeyboardKey.space ||
        touche == LogicalKeyboardKey.arrowUp ||
        touche == LogicalKeyboardKey.keyW ||
        touche == LogicalKeyboardKey.keyZ) {
      _clavSaut = appui;
    }
  }

  @override
  Widget build(BuildContext context) {
    return KeyboardListener(
      focusNode: _focus,
      autofocus: true,
      onKeyEvent: _clavier,
      child: Listener(
        behavior: HitTestBehavior.opaque,
        onPointerDown: (PointerDownEvent e) =>
            _majPointeur(e.pointer, e.localPosition),
        onPointerMove: (PointerMoveEvent e) =>
            _majPointeur(e.pointer, e.localPosition),
        onPointerUp: (PointerUpEvent e) => _pointeurs.remove(e.pointer),
        onPointerCancel: (PointerCancelEvent e) =>
            _pointeurs.remove(e.pointer),
        child: LayoutBuilder(
          builder: (BuildContext context, BoxConstraints contraintes) {
            _taille = Size(contraintes.maxWidth, contraintes.maxHeight);
            _camera.largeur = _taille.width;
            _camera.hauteur = _taille.height;
            return CustomPaint(
              painter: PeintreDonjon(
                camera: _camera,
                carte: _carte,
                heros: _heros,
                gobelins: _gobelins,
                coffres: _coffres,
                portes: _portes,
                salle: _salle,
                pv: _pv,
                pieces: _pieces,
                invincible: _invincible,
                fondu: _fondu,
                fps: _fps,
                gauche: _allerGauche,
                droite: _allerDroite,
                saut: _sauter,
              ),
              size: Size.infinite,
            );
          },
        ),
      ),
    );
  }
}

// =====================================================================
// RENDU
// =====================================================================

class PeintreDonjon extends CustomPainter {
  PeintreDonjon({
    required this.camera,
    required this.carte,
    required this.heros,
    required this.gobelins,
    required this.coffres,
    required this.portes,
    required this.salle,
    required this.pv,
    required this.pieces,
    required this.invincible,
    required this.fondu,
    required this.fps,
    required this.gauche,
    required this.droite,
    required this.saut,
  });

  final Camera camera;
  final Tilemap carte;
  final Heros heros;
  final List<Gobelin> gobelins;
  final List<Coffre> coffres;
  final List<Porte> portes;
  final String salle;
  final int pv;
  final int pieces;
  final double invincible;
  final double fondu;
  final double fps;
  final bool gauche;
  final bool droite;
  final bool saut;

  static final Map<int, Paint> _peintures = <int, Paint>{
    1: Paint()..color = const Color(0xFF443A5E),
    2: Paint()..color = const Color(0xFF7A5230),
    3: Paint()..color = const Color(0xFF9E3B4A),
  };

  static final Paint _bordSuperieur = Paint()..color = const Color(0xFF6A5A8C);

  Paint _peintureTuile(int index) =>
      _peintures[index] ?? (Paint()..color = const Color(0xFFFF00FF));

  void _texte(Canvas canvas, String s, Offset p,
      {Color c = const Color(0xFFD8D8E8), double taille = 13}) {
    final TextPainter tp = TextPainter(
      text: TextSpan(text: s, style: TextStyle(color: c, fontSize: taille)),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, p);
  }

  void _paver({
    required Canvas canvas,
    required double largeurEcran,
    required double largeurMotif,
    required double decalage,
    required void Function(double x) motif,
  }) {
    double origine = decalage % largeurMotif;
    if (origine > 0) origine -= largeurMotif;
    for (double x = origine; x < largeurEcran; x += largeurMotif) {
      motif(x);
    }
  }

  /// Dessine les tuiles visibles et renvoie leur nombre.
  int _dessinerTuiles(Canvas canvas) {
    final Rect vue = camera.vueElargie;
    final int colMin =
        (vue.left / carte.taille).floor().clamp(0, carte.colonnes - 1);
    final int colMax =
        (vue.right / carte.taille).ceil().clamp(0, carte.colonnes - 1);
    final int ligMin =
        (vue.top / carte.taille).floor().clamp(0, carte.lignes - 1);
    final int ligMax =
        (vue.bottom / carte.taille).ceil().clamp(0, carte.lignes - 1);

    int compte = 0;
    for (int l = ligMin; l <= ligMax; l++) {
      for (int c = colMin; c <= colMax; c++) {
        final int index = carte.tuiles[l][c];
        if (index == 0) continue;

        final Rect r = carte.rectDe(c, l);

        if (index == 3) {
          // Les pointes : trois triangles dans la case.
          canvas.drawRect(
            Rect.fromLTWH(r.left, r.bottom - 6, r.width, 6),
            _peintureTuile(1),
          );
          for (int i = 0; i < 3; i++) {
            final double bx = r.left + i * (carte.taille / 3);
            final Path p = Path()
              ..moveTo(bx, r.bottom - 6)
              ..lineTo(bx + carte.taille / 6, r.top + 4)
              ..lineTo(bx + carte.taille / 3, r.bottom - 6)
              ..close();
            canvas.drawPath(p, _peintureTuile(3));
          }
        } else {
          canvas.drawRect(r, _peintureTuile(index));
          if (!carte.solideA(c, l - 1)) {
            canvas.drawRect(
              Rect.fromLTWH(r.left, r.top, r.width, 4),
              _bordSuperieur,
            );
          }
        }
        compte++;
      }
    }
    return compte;
  }

  void _dessinerHeros(Canvas canvas) {
    // Clignotement pendant l'invincibilité.
    final bool visible =
        invincible <= 0 || ((invincible * 12).floor() % 2 == 0);
    if (!visible) return;

    final Rect b = heros.boite;
    canvas.drawRect(
      Rect.fromLTWH(b.left, b.top + 12, b.width, b.height - 12),
      Paint()..color = const Color(0xFF4EA3D8),
    );
    canvas.drawRect(
      Rect.fromLTWH(b.left + 3, b.top, b.width - 6, 13),
      Paint()..color = const Color(0xFFE0B48C),
    );
    // Un oeil, du côté du regard.
    canvas.drawRect(
      Rect.fromLTWH(
          heros.regardeADroite ? b.right - 9 : b.left + 6, b.top + 4, 3, 3),
      Paint()..color = const Color(0xFF20202C),
    );
  }

  @override
  void paint(Canvas canvas, Size size) {
    // ============ REPÈRE ÉCRAN : ciel et parallaxe ============
    canvas.drawRect(
      Offset.zero & size,
      Paint()
        ..shader = const LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: <Color>[Color(0xFF130E22), Color(0xFF2A1E3C)],
        ).createShader(Offset.zero & size),
    );

    final Paint lointain = Paint()..color = const Color(0xFF1C1533);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 260,
      decalage: -camera.x * 0.18,
      motif: (double x) {
        final Path p = Path()
          ..moveTo(x, size.height)
          ..lineTo(x + 130, size.height * 0.30)
          ..lineTo(x + 260, size.height)
          ..close();
        canvas.drawPath(p, lointain);
      },
    );

    final Paint colonne = Paint()..color = const Color(0xFF2A2044);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 190,
      decalage: -camera.x * 0.45,
      motif: (double x) {
        canvas.drawRect(
            Rect.fromLTWH(x + 50, size.height * 0.34, 38, size.height), colonne);
        canvas.drawRect(
            Rect.fromLTWH(x + 40, size.height * 0.31, 58, 16), colonne);
      },
    );

    // ============ REPÈRE MONDE ============
    camera.appliquer(canvas);

    final int tuiles = _dessinerTuiles(canvas);
    final Rect vue = camera.vueElargie;

    for (final Porte p in portes) {
      if (!vue.overlaps(p.boite)) continue;
      canvas.drawRect(p.boite, Paint()..color = const Color(0xFF2E2438));
      canvas.drawRect(
        p.boite.deflate(5),
        Paint()..color = const Color(0xFF6FD3B8),
      );
    }

    for (final Coffre c in coffres) {
      if (!vue.overlaps(c.boite)) continue;
      canvas.drawRect(
        c.boite,
        Paint()
          ..color = c.ouvert ? const Color(0xFF52483A) : const Color(0xFFD7A24C),
      );
      canvas.drawRect(
        Rect.fromLTWH(c.boite.left, c.boite.top, c.boite.width, 7),
        Paint()..color = const Color(0xFF8A6C2E),
      );
    }

    for (final Gobelin g in gobelins) {
      if (!vue.overlaps(g.boite)) continue;
      canvas.drawRect(
        g.boite,
        Paint()
          ..color = g.boss ? const Color(0xFF9E4ED1) : const Color(0xFF7FB84E),
      );
      final double ox = g.vx > 0 ? g.boite.right - 8 : g.boite.left + 5;
      canvas.drawRect(
        Rect.fromLTWH(ox, g.boite.top + 6, 3, 3),
        Paint()..color = const Color(0xFF1A1A22),
      );
    }

    _dessinerHeros(canvas);

    camera.retirer(canvas);
    // ============ RETOUR AU REPÈRE ÉCRAN : HUD ============

    // Boutons tactiles.
    void bouton(Rect r, String etiquette, bool enfonce) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(r, const Radius.circular(12)),
        Paint()
          ..color = enfonce ? const Color(0x88FFFFFF) : const Color(0x33FFFFFF),
      );
      _texte(canvas, etiquette,
          Offset(r.center.dx - 12, r.center.dy - 8), taille: 15);
    }

    bouton(boutonGauche(size), '<', gauche);
    bouton(boutonDroite(size), '>', droite);
    bouton(boutonSaut(size), 'SAUT', saut);

    // Panneau d'information.
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 300, 128),
      Paint()..color = const Color(0xCC000000),
    );
    _texte(canvas, 'DONJON DE DART — $salle', const Offset(12, 8));
    _texte(canvas, 'PV      : ${'[#]' * pv}${'[ ]' * (5 - pv)}',
        const Offset(12, 30), c: const Color(0xFFE05A6A));
    _texte(canvas, 'pieces  : $pieces', const Offset(12, 50),
        c: const Color(0xFFD7A24C));
    _texte(
      canvas,
      'camera  : (${camera.x.toStringAsFixed(0)}, '
      '${camera.y.toStringAsFixed(0)})',
      const Offset(12, 70),
    );
    _texte(
      canvas,
      'tuiles  : $tuiles / ${carte.colonnes * carte.lignes}',
      const Offset(12, 90),
      c: const Color(0xFF4EC9B0),
    );
    _texte(canvas, 'FPS     : ${fps.toStringAsFixed(0)}',
        const Offset(12, 110));

    // Fondu de transition, tout à la fin.
    if (fondu > 0) {
      canvas.drawRect(
        Offset.zero & size,
        Paint()..color = Color.fromRGBO(0, 0, 0, fondu),
      );
    }
  }

  @override
  bool shouldRepaint(PeintreDonjon oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART — salle-1
PV      : [#][#][#][#][#]
pieces  : 0
camera  : (0, 112)
tuiles  : 85 / 960
FPS     : 60

L'écran s'éclaircit depuis le noir sur la première salle du donjon.
Le héros bleu se tient sur le sol de pierre, à gauche.
Derrière lui, des silhouettes de montagnes glissent lentement, et des
colonnes de donjon défilent deux fois et demie plus vite.
Les flèches (ou les boutons tactiles) le font marcher ; il saute d'une
plateforme de bois à l'autre.
La caméra ne bouge pas tant qu'il reste dans le rectangle central ;
dès qu'il en sort, elle glisse en douceur et regarde un peu devant lui.
Elle s'arrête net aux bords de la salle : aucun vide n'apparaît jamais.
Un gobelin vert patrouille sur le sol et fait demi-tour tout seul devant
un mur. Le toucher coûte un coeur, l'écran tremble, le héros clignote.
Les pointes rouges font la même chose.
Le coffre doré s'ouvre au contact et ajoute 10 pièces.
La porte turquoise, tout à droite, déclenche un fondu au noir puis la
salle 2, où le héros apparaît côté ouest, caméra déjà en place.
Le compteur « tuiles » reste autour de 100 sur 960, quelle que soit la
salle : c'est le culling qui travaille.
```

### Ce qu'il faut regarder dans ce programme

**La chaîne de la caméra est explicite.** Dans `_majJeu`, les quatre étapes se suivent dans l'ordre imposé par la section 25.12 : look-ahead, cible, zone morte plus lissage, bornage. Changez l'ordre et vous verrez immédiatement apparaître les défauts décrits dans le chapitre.

**Le `-20` de la cible.** `_heros.centre + Offset(_camera.avance, -20)` vise vingt pixels au-dessus du héros : on voit un peu plus de plafond que de sol. C'est le réglage de la section 25.7.

**Un seul `save()` / `restore()`.** Tout ce qui est entre `camera.appliquer` et `camera.retirer` est en monde. Le parallaxe est avant, le HUD après. Aucune soustraction manuelle nulle part.

**Le compteur de tuiles est un outil de diagnostic.** S'il affiche 960 sur 960, votre culling ne fonctionne pas. C'est le genre d'indicateur qu'il faut garder pendant tout le développement.

**Le verrou de transition.** `_demanderTransition` sort immédiatement si `_enTransition` est vrai. Retirez cette ligne et le jeu se bloquera sur la première porte.

**Les points de vie survivent au changement de salle**, mais les gobelins sont reconstruits. C'est la distinction de la section 25.29.

### Comment le prolonger

- ajouter un calque de décor devant le héros (facteur 1,3) ;
- ajouter un zoom qui se resserre dans la salle du boss ;
- charger les salles depuis de vrais fichiers JSON (section 25.26) ;
- remplacer les rectangles par des sprites (chapitre 22) ;
- ajouter une minicarte dans le HUD, qui dessine la grille en tout petit.

---

## 25.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le HUD (score, vies) défile avec le décor | `canvas.restore()` oublié après le `translate` de la caméra | Encadrer strictement : `save()` → `translate()` → monde → `restore()` → HUD |
| Le décor part dans le mauvais sens quand le héros avance | Signe inversé : `monde + camera` au lieu de `monde - camera` | `ecran = monde - camera`, et `canvas.translate(-camX, -camY)` |
| Les clics tombent à côté dès que la caméra bouge | Position du doigt utilisée telle quelle, sans `versMonde` | Convertir : `monde = ecran + camera` (ou `/ zoom + camera` avec zoom) |
| Les clics sont justes à zoom 1 et faux ensuite | Le zoom est appliqué au rendu mais pas à `versMonde` | `monde = ecran / zoom + camera` ; corriger aussi `vue`, `borner`, `centrerSur` |
| Une bande vide apparaît au bord du niveau | Caméra non bornée, ou bornée avant le lissage | Appeler `borner()` **en dernier**, après lissage et zoom |
| `Invalid clamp range` au lancement | `largeurMonde - largeur` négatif : le monde est plus petit que l'écran | Tester `if (lMonde <= vueLargeur)` et centrer le monde dans ce cas |
| La caméra suit deux fois plus lentement sur une machine lente | `lerp` avec un `t` constant par frame | `t = 1 - exp(-vitesse * dt)` |
| Au chargement d'une salle, la caméra traverse tout le niveau | Lissage appliqué depuis l'ancienne position | Appeler `placerImmediatement()` puis `borner()` après le chargement |
| La caméra tremble en permanence | Secousse jamais amortie, ou ajoutée à `camera.x` au lieu d'un décalage séparé | Décalage séparé, amorti par `intensite -= amortissement * dt` |
| Le jeu rame alors que l'écran est presque vide | Tilemap entièrement redessinée à chaque frame | Calculer `colMin`/`colMax`/`ligMin`/`ligMax` depuis la vue et ne boucler que dessus |
| Une colonne de tuiles clignote au bord droit | `floor()` utilisé pour les deux bords de la plage visible | `floor()` à gauche et en haut, `ceil()` à droite et en bas |
| `RangeError (index): Invalid value` au bord du monde | Indices de tuiles non bornés quand la vue dépasse la carte | `.clamp(0, colonnes - 1)` sur les quatre indices |
| Le niveau est transposé, les murs horizontaux sont verticaux | `grille[colonne][ligne]` au lieu de `grille[ligne][colonne]` | Toujours `tuiles[ligne][colonne]`, c'est-à-dire `[y][x]` |
| Le héros traverse le mur en allant à gauche du monde | `toInt()` au lieu de `floor()` : troncature vers zéro sur les négatifs | Toujours `(valeur / taille).floor()` |
| Le héros reste bloqué contre du vide | Bord droit ou bas d'une hitbox exactement sur une frontière de tuile | Retirer un epsilon : `((b.right - 0.001) / taille).floor()` |
| Le héros accroche les coins des plateformes | Déplacement diagonal résolu en un seul test | Résoudre **X d'abord, puis Y**, séparément |
| Le héros peut sauter indéfiniment en l'air | `auSol` jamais remis à `false` | `auSol = false` avant la résolution verticale de chaque frame |
| Le jeu se bloque en boucle de chargement sur une porte | Pas de verrou : la transition se redéclenche à chaque frame | `if (_enTransition) return;` en tête de `demanderTransition` |
| Le héros se soigne en changeant de salle | Les points de vie sont réinitialisés avec le niveau | Distinguer l'état du **joueur** (conservé) de celui du **niveau** (reconstruit) |
| Une colonne du fond parallaxe manque à gauche | `%` renvoie un résultat positif quand le décalage est positif | `if (origine > 0) origine -= largeurMotif;` |
| Le héros démarre encastré dans le sol | Point d'apparition placé au centre de la case | `y = (lig + 1) * taille - hauteurHeros` |
| Une tuile apparaît en magenta dans le jeu | Index présent dans la carte mais absent de la palette | C'est le signal voulu : compléter la palette (ou corriger la carte) |
| Les objets disparaissent brutalement au bord de l'écran | Culling sur la hitbox stricte, alors que le sprite est plus large | Culler sur `vue.inflate(marge)` |
| Un projectile rapide traverse les murs | Déplacement supérieur à une tuile en une frame (tunneling) | Plafonner le pas : `(v * dt).clamp(-taille * 0.9, taille * 0.9)` |

---

## 25.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Monde vs écran | Deux repères. Les entités vivent en monde, le `Canvas` dessine en écran |
| Caméra | Une simple paire `(x, y)` : le coin haut-gauche de la zone visible, en monde |
| Monde → écran | `ecran = (monde - camera) * zoom` |
| Écran → monde | `monde = ecran / zoom + camera` |
| Test de validité | L'aller-retour des deux conversions doit rendre la valeur de départ |
| `canvas.translate` | `save()` / `scale(zoom)` / `translate(-camX, -camY)` / … / `restore()` |
| HUD | Toujours dessiné **après** `restore()`, en coordonnées écran |
| Centrer | `camera.x = cible.x - vueLargeur / 2` |
| Suivi brutal | Correct mais dur : jitter, trampoline pendant les sauts |
| Suivi lissé | `x += (cible - x) * t` avec `t = 1 - exp(-vitesse * dt)` |
| Zone morte | La caméra ne bouge que si la cible sort d'un rectangle central |
| Look-ahead | Décalage de la cible dans le sens du mouvement, lissé séparément et plus lentement |
| Bornage | `clamp` en tout dernier ; cas particulier si le monde est plus petit que l'écran |
| Zoom | Multiplie en rendu, divise en conversion inverse ; à répercuter partout |
| Screen shake | Décalage aléatoire **séparé**, amorti jusqu'à zéro |
| Culling | Tester `vue.inflate(marge).overlaps(boite)` avant de dessiner |
| Parallaxe | `translate(-camX * facteur)` par plan ; facteur < 1 au fond, > 1 devant |
| Pavage | `origine = decalage % largeurMotif`, corrigé si positif |
| Tilemap | Grille régulière d'entiers ; `tuiles[ligne][colonne]` |
| Taille de tuile | Une constante unique ; 32 pixels pour le Donjon de Dart |
| Monde → tuile | `colonne = (mondeX / taille).floor()` — jamais `toInt()` |
| Tuile → monde | `x = colonne * taille` |
| Plage visible | `floor` à gauche/haut, `ceil` à droite/bas, `clamp` partout |
| Collision tilemap | Axe X puis axe Y, repositionnement, vitesse annulée, `auSol` sur collision basse |
| Niveau en texte | `List<String>` + `Map<String, int>` : lisible, éditable, validable |
| Niveau en JSON | `fromJson` du chapitre 17 ; `(v as num).toDouble()` pour les nombres |
| Objets de niveau | La grille pour le décor fixe, la liste d'objets pour ce qui vit |
| Point d'apparition | `y = (lig + 1) * taille - hauteurHeros`, pieds sur la case |
| Tiled | Éditeur standard, `.tmx` en XML, index 0 = vide ; on exporte en JSON ou on attend le chapitre 34 |
| Transition | Verrou, fondu, chargement, caméra placée immédiatement |

---

## 25.33 — Exercices

### Exercice 1 — Les deux conversions (facile)

Écrivez deux fonctions `mondeVersEcran` et `ecranVersMonde` prenant un `Offset` et une caméra `(camX, camY)`. Écrivez ensuite une fonction `verifierAllerRetour` qui prend un point monde, applique les deux conversions à la suite et affiche si l'on retombe sur la valeur de départ. Testez avec trois caméras différentes, dont une aux coordonnées négatives.

### Exercice 2 — Une caméra qui centre (facile)

Écrivez une classe `Camera` avec `x`, `y`, `largeur`, `hauteur`, un getter `vue`, et une méthode `centrerSur(Offset monde)`. Vérifiez en console qu'après `centrerSur`, la position écran de la cible est bien le centre exact de la fenêtre. Affichez également les quatre bords de la vue.

### Exercice 3 — Borner la caméra (facile)

Ajoutez à votre `Camera` une méthode `borner(double largeurMonde, double hauteurMonde)` qui empêche la vue de sortir du monde, **et** qui centre le monde quand celui-ci est plus petit que l'écran. Testez quatre cas : trop à gauche, trop à droite, trop en haut, monde plus petit que l'écran. Aucun appel ne doit lever d'exception.

### Exercice 4 — Le lissage indépendant du framerate (moyen)

Écrivez une fonction `simuler(double fps, double vitesse, double duree)` qui part d'une caméra à 0, d'une cible fixe à 1000, et applique le lissage exponentiel pendant `duree` secondes. Affichez un tableau comparant 20, 60 et 144 FPS, pour la version correcte et pour la version naïve à `t` constant. Concluez par le nom de la ligne qui prouve l'indépendance au framerate.

### Exercice 5 — La zone morte (moyen)

Écrivez une fonction `majZoneMorte(Camera camera, Offset cible, double margeX, double margeY)` qui n'ajuste la caméra que si la cible sort du rectangle central. Simulez un héros qui va de `x = 0` à `x = 600` par pas de 25, avec une fenêtre de 400 sur 200 et une marge de 120 sur 60. Affichez pour chaque pas la position écran du héros et indiquez si la caméra a bougé. Le héros doit rester bloqué à la même abscisse écran une fois la zone franchie.

### Exercice 6 — Le tremblement amorti (moyen)

Écrivez une classe `Secousse` avec `declencher(double intensite)`, `mettreAJour(double dt)` et des champs `decalageX` / `decalageY`. L'intensité doit décroître linéairement et le décalage s'annuler à la fin. Simulez : une secousse de 10, puis, à la cinquième frame, une seconde secousse de 4 (qui ne doit **pas** relancer l'effet), puis à la huitième frame une secousse de 12 (qui doit, elle, prendre le relais). Affichez l'intensité à chaque frame.

### Exercice 7 — Charger et valider une carte texte (moyen)

Écrivez `Tilemap.depuisTexte(List<String> plan, Map<String, int> legende)` qui lève une `FormatException` explicite si les lignes n'ont pas toutes la même longueur, ou si un caractère est absent de la légende. Chargez une salle valide, affichez ses dimensions en tuiles et en pixels, puis provoquez et attrapez les deux erreurs.

### Exercice 8 — Les tuiles sous un rectangle (moyen)

Écrivez une fonction `tuilesSous(Rect boite, double taille)` qui renvoie la liste des couples `(colonne, ligne)` couverts par un rectangle, en gérant correctement l'epsilon des bords droit et bas. Écrivez ensuite `estBloque(Tilemap carte, Rect boite)` qui renvoie `true` si au moins une de ces tuiles est solide. Testez avec un rectangle exactement aligné sur une frontière de tuile, et avec un rectangle à cheval sur quatre tuiles.

### Exercice 9 — Un fond en parallaxe (difficile)

Écrivez un `main.dart` complet affichant un monde de 3 000 pixels de large avec trois plans : un ciel immobile (facteur 0), des montagnes (facteur 0,25) et des colonnes de donjon (facteur 0,6), plus un sol au facteur 1. La caméra se déplace au glisser du doigt et reste bornée au monde. Affichez en HUD la position de la caméra et le facteur de chaque plan.

### Exercice 10 — Une salle jouable complète (difficile)

Écrivez un `main.dart` complet réunissant : une salle en `List<String>` d'au moins 40 colonnes, une `Tilemap`, un héros contrôlable avec gravité et saut, la collision axe par axe, une caméra à suivi lissé, zone morte et bornage, et le culling des tuiles. Affichez en HUD le nombre de tuiles dessinées sur le nombre total, ainsi que l'état `auSol` du héros.

---

## 25.34 — Corrections des exercices

### Correction 1

```dart
import 'dart:ui';

Offset mondeVersEcran(Offset monde, Offset camera) =>
    Offset(monde.dx - camera.dx, monde.dy - camera.dy);

Offset ecranVersMonde(Offset ecran, Offset camera) =>
    Offset(ecran.dx + camera.dx, ecran.dy + camera.dy);

String f(Offset o) =>
    '(${o.dx.toStringAsFixed(1)}, ${o.dy.toStringAsFixed(1)})';

void verifierAllerRetour(Offset monde, Offset camera) {
  final Offset ecran = mondeVersEcran(monde, camera);
  final Offset retour = ecranVersMonde(ecran, camera);

  // On ne compare JAMAIS deux doubles avec ==, on compare un écart.
  final bool identique = (retour.dx - monde.dx).abs() < 0.0001 &&
      (retour.dy - monde.dy).abs() < 0.0001;

  print('camera ${f(camera).padRight(18)}'
      'monde ${f(monde).padRight(18)}'
      '-> ecran ${f(ecran).padRight(18)}'
      '-> monde ${f(retour).padRight(18)}'
      '${identique ? "OK" : "ECHEC"}');
}

void main() {
  const Offset coffre = Offset(2400, 300);

  verifierAllerRetour(coffre, const Offset(0, 0));
  verifierAllerRetour(coffre, const Offset(1800, 120));
  verifierAllerRetour(coffre, const Offset(-250, -80));
}
```

**Résultat :**

```text
camera (0.0, 0.0)        monde (2400.0, 300.0)   -> ecran (2400.0, 300.0)   -> monde (2400.0, 300.0)   OK
camera (1800.0, 120.0)   monde (2400.0, 300.0)   -> ecran (600.0, 180.0)    -> monde (2400.0, 300.0)   OK
camera (-250.0, -80.0)   monde (2400.0, 300.0)   -> ecran (2650.0, 380.0)   -> monde (2400.0, 300.0)   OK
```

**Explication :** les deux fonctions sont symétriques : une soustraction, une addition. Le test de l'aller-retour est la vérification la plus rentable du chapitre : il détecte immédiatement une inversion de signe, sans avoir besoin de lancer le jeu.

La caméra négative du troisième cas n'est pas absurde : elle se produit quand un monde est plus petit que l'écran (section 25.12), et c'est justement le cas que le débutant oublie de tester. La position écran vaut alors 2 650, très au-delà de la fenêtre, ce qui est correct : l'objet n'est simplement pas visible.

Notez enfin la comparaison par écart plutôt que par `==`. Sur des `double`, une égalité stricte peut échouer pour un dix-millième de pixel. C'est la règle du chapitre 02.

---

### Correction 2

```dart
import 'dart:ui';

class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    required this.largeur,
    required this.hauteur,
  });

  double x;
  double y;
  double largeur;
  double hauteur;

  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);

  Offset versEcran(Offset m) => Offset(m.dx - x, m.dy - y);
  Offset versMonde(Offset e) => Offset(e.dx + x, e.dy + y);

  void centrerSur(Offset monde) {
    x = monde.dx - largeur / 2;
    y = monde.dy - hauteur / 2;
  }
}

void main() {
  final Camera camera = Camera(largeur: 800, hauteur: 450);
  const Offset heros = Offset(1600, 900);

  camera.centrerSur(heros);

  print('coin de la camera : (${camera.x}, ${camera.y})');
  print('vue               : ${camera.vue}');
  print('');
  print('bord gauche : ${camera.vue.left}');
  print('bord droit  : ${camera.vue.right}');
  print('bord haut   : ${camera.vue.top}');
  print('bord bas    : ${camera.vue.bottom}');
  print('');

  final Offset ecran = camera.versEcran(heros);
  print('heros a l ecran   : $ecran');
  print('centre attendu    : '
      '(${camera.largeur / 2}, ${camera.hauteur / 2})');
  print('centrage correct  : '
      '${ecran.dx == camera.largeur / 2 && ecran.dy == camera.hauteur / 2}');
}
```

**Résultat :**

```text
coin de la camera : (1200.0, 675.0)
vue               : Rect.fromLTRB(1200.0, 675.0, 2000.0, 1125.0)

bord gauche : 1200.0
bord droit  : 2000.0
bord haut   : 675.0
bord bas    : 1125.0

heros a l ecran   : Offset(400.0, 225.0)
centre attendu    : (400.0, 225.0)
centrage correct  : true
```

**Explication :** `centrerSur` recule le coin de la caméra d'une demi-fenêtre. C'est la seule ligne à retenir de cette correction :

```text
  coin = cible - taille / 2
```

Le getter `vue` est un `Rect` calculé à la demande. Il ne coûte rien à stocker et sert partout : culling, bornage, tuiles visibles. Prenez l'habitude de l'exposer dès la première version de votre caméra.

Le test final vérifie que la cible tombe exactement au centre. Ici l'égalité stricte est acceptable, car les valeurs sont issues d'une division par 2, exacte en binaire. Dans le doute, comparez toujours avec une tolérance.

---

### Correction 3

```dart
class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    required this.largeur,
    required this.hauteur,
  });

  double x;
  double y;
  double largeur;
  double hauteur;

  void borner(double largeurMonde, double hauteurMonde) {
    // Cas 1 : le monde est plus étroit que l'écran -> on le centre.
    if (largeurMonde <= largeur) {
      x = (largeurMonde - largeur) / 2;
    } else {
      x = x.clamp(0.0, largeurMonde - largeur);
    }

    // Cas 2 : idem verticalement.
    if (hauteurMonde <= hauteur) {
      y = (hauteurMonde - hauteur) / 2;
    } else {
      y = y.clamp(0.0, hauteurMonde - hauteur);
    }
  }

  @override
  String toString() =>
      '(${x.toStringAsFixed(1)}, ${y.toStringAsFixed(1)})';
}

void tester(String titre, Camera c, double lMonde, double hMonde) {
  final String avant = c.toString();
  c.borner(lMonde, hMonde);
  print('${titre.padRight(24)} $avant -> $c');
}

void main() {
  const double lMonde = 2400;
  const double hMonde = 1200;

  tester('trop a gauche',
      Camera(x: -300, y: 200, largeur: 800, hauteur: 400), lMonde, hMonde);
  tester('trop a droite',
      Camera(x: 5000, y: 100, largeur: 800, hauteur: 400), lMonde, hMonde);
  tester('trop en haut',
      Camera(x: 500, y: -90, largeur: 800, hauteur: 400), lMonde, hMonde);
  tester('trop en bas',
      Camera(x: 500, y: 3000, largeur: 800, hauteur: 400), lMonde, hMonde);
  tester('monde plus petit',
      Camera(x: 120, y: 40, largeur: 800, hauteur: 400), 600, 300);
  tester('deja valide',
      Camera(x: 400, y: 300, largeur: 800, hauteur: 400), lMonde, hMonde);
}
```

**Résultat :**

```text
trop a gauche            (-300.0, 200.0) -> (0.0, 200.0)
trop a droite            (5000.0, 100.0) -> (1600.0, 100.0)
trop en haut             (500.0, -90.0) -> (500.0, 0.0)
trop en bas              (500.0, 3000.0) -> (500.0, 800.0)
monde plus petit         (120.0, 40.0) -> (-100.0, -50.0)
deja valide              (400.0, 300.0) -> (400.0, 300.0)
```

**Explication :** les quatre premiers cas sont le `clamp` ordinaire : `0` d'un côté, `tailleMonde - tailleVue` de l'autre. La caméra ne peut jamais montrer un pixel au-delà des bords du niveau.

Le cinquième cas est celui qui plante si on l'oublie. `clamp(0.0, 600 - 800)` vaut `clamp(0.0, -200.0)`, et Dart refuse un intervalle dont la borne haute est inférieure à la borne basse :

```text
Invalid argument(s): Invalid clamp range
```

En centrant le monde à `-100`, on obtient 100 pixels de marge de chaque côté. Le rendu est propre et le programme ne plante pas, y compris sur un très grand écran.

Le dernier cas confirme que `borner` ne modifie rien quand tout va bien : une contrainte doit être neutre lorsqu'elle est déjà satisfaite.

---

### Correction 4

```dart
import 'dart:math' as math;

/// Lissage exponentiel : indépendant du framerate.
double simuler(double fps, double vitesse, double duree) {
  final double dt = 1 / fps;
  final int frames = (fps * duree).round();
  double camera = 0;
  const double cible = 1000;

  for (int i = 0; i < frames; i++) {
    final double t = 1 - math.exp(-vitesse * dt);
    camera += (cible - camera) * t;
  }
  return camera;
}

/// Lissage naïf : t constant par frame, donc dépendant du framerate.
double simulerNaif(double fps, double t, double duree) {
  final int frames = (fps * duree).round();
  double camera = 0;
  const double cible = 1000;

  for (int i = 0; i < frames; i++) {
    camera += (cible - camera) * t;
  }
  return camera;
}

void main() {
  const List<double> tousLesFps = <double>[20, 60, 144];
  const double duree = 1;

  print('Cible = 1000, duree = 1 s');
  print('');
  print('  FPS   correct (v=5)   naif (t=0.1)');
  print('  ---   -------------   ------------');

  for (final double fps in tousLesFps) {
    final double correct = simuler(fps, 5, duree);
    final double naif = simulerNaif(fps, 0.1, duree);
    print('  ${fps.toStringAsFixed(0).padLeft(3)}   '
        '${correct.toStringAsFixed(2).padLeft(13)}   '
        '${naif.toStringAsFixed(2).padLeft(12)}');
  }

  print('');
  final double a = simuler(20, 5, duree);
  final double b = simuler(144, 5, duree);
  print('ecart correct entre 20 et 144 FPS : '
      '${(a - b).abs().toStringAsFixed(6)} px');

  final double c = simulerNaif(20, 0.1, duree);
  final double d = simulerNaif(144, 0.1, duree);
  print('ecart naif    entre 20 et 144 FPS : '
      '${(c - d).abs().toStringAsFixed(6)} px');
}
```

**Résultat :**

```text
Cible = 1000, duree = 1 s

  FPS   correct (v=5)   naif (t=0.1)
  ---   -------------   ------------
   20          993.26         878.42
   60          993.26         998.20
  144          993.26        1000.00

ecart correct entre 20 et 144 FPS : 0.000000 px
ecart naif    entre 20 et 144 FPS : 121.577 px
```

**Explication :** la ligne qui prouve l'indépendance au framerate est la dernière : **l'écart correct est nul**, l'écart naïf dépasse 121 pixels.

La raison est mathématique. À chaque frame, la version correcte multiplie l'écart restant par `exp(-vitesse * dt)`. Après `n` frames :

```text
  ecart_final = ecart_initial * exp(-vitesse * dt) ^ n
              = ecart_initial * exp(-vitesse * dt * n)
              = ecart_initial * exp(-vitesse * duree)
```

Le nombre de frames disparaît de l'expression : seule la durée compte. C'est exactement ce que l'on veut.

La version naïve, elle, multiplie par `0,9` à chaque frame. Après 20 frames, il reste `0,9^20 = 12,2 %` de l'écart ; après 144 frames, `0,9^144 ≈ 0 %`. Un joueur sur machine lente aurait donc une caméra molle, et un joueur sur écran 144 Hz une caméra collée au héros. Deux jeux différents.

C'est le même raisonnement qu'au chapitre 20 pour `x += vitesse * dt` : dès qu'une quantité évolue dans le temps, elle doit dépendre de `dt`, jamais du numéro de la frame.

---

### Correction 5

```dart
import 'dart:ui';

class Camera {
  Camera({
    this.x = 0,
    this.y = 0,
    required this.largeur,
    required this.hauteur,
  });

  double x;
  double y;
  double largeur;
  double hauteur;

  Offset versEcran(Offset m) => Offset(m.dx - x, m.dy - y);
}

/// N'ajuste la caméra que si [cible] sort du rectangle central.
/// Renvoie true si la caméra a bougé.
bool majZoneMorte(Camera camera, Offset cible, double margeX, double margeY) {
  final Offset e = camera.versEcran(cible);

  final double gauche = margeX;
  final double droite = camera.largeur - margeX;
  final double haut = margeY;
  final double bas = camera.hauteur - margeY;

  final double avantX = camera.x;
  final double avantY = camera.y;

  if (e.dx > droite) {
    camera.x += e.dx - droite;
  } else if (e.dx < gauche) {
    camera.x -= gauche - e.dx;
  }

  if (e.dy > bas) {
    camera.y += e.dy - bas;
  } else if (e.dy < haut) {
    camera.y -= haut - e.dy;
  }

  return camera.x != avantX || camera.y != avantY;
}

void main() {
  final Camera camera = Camera(x: 0, y: 0, largeur: 400, hauteur: 200);
  const double margeX = 120;
  const double margeY = 60;

  print('fenetre 400 x 200, zone morte de '
      '${400 - 2 * margeX} x ${200 - 2 * margeY} au centre');
  print('');
  print('heros.x   ecran.x   camera.x   camera bouge ?');
  print('-------   -------   --------   --------------');

  for (double hx = 0; hx <= 600; hx += 50) {
    final Offset heros = Offset(hx, 100);
    final bool bouge = majZoneMorte(camera, heros, margeX, margeY);
    final Offset e = camera.versEcran(heros);
    print('${hx.toStringAsFixed(0).padLeft(7)}   '
        '${e.dx.toStringAsFixed(0).padLeft(7)}   '
        '${camera.x.toStringAsFixed(0).padLeft(8)}   '
        '${bouge ? "oui" : "non"}');
  }
}
```

**Résultat :**

```text
fenetre 400 x 200, zone morte de 160.0 x 80.0 au centre

heros.x   ecran.x   camera.x   camera bouge ?
-------   -------   --------   --------------
      0       120       -120   oui
     50       170       -120   non
    100       220       -120   non
    150       270       -120   non
    200       280        -80   oui
    250       280        -30   oui
    300       280         20   oui
    350       280         70   oui
    400       280        120   oui
    450       280        170   oui
    500       280        220   oui
    550       280        270   oui
    600       280        320   oui
```

**Explication :** trois choses à lire dans ce tableau.

**La première ligne recadre.** Au départ, le héros est en `x = 0` avec une caméra en `0` : il apparaît donc à l'abscisse écran 0, à gauche de la zone morte. La caméra recule de 120 pour le ramener sur le bord gauche de la zone. C'est le comportement attendu, et c'est pourquoi on appelle `placerImmediatement()` au chargement d'un niveau : pour éviter ce recadrage visible.

**Le milieu du tableau est immobile.** De `x = 50` à `x = 150`, la caméra ne bouge pas d'un pixel. Le héros traverse l'écran de 170 à 270 : c'est **lui** qui se déplace dans le cadre, pas le décor. C'est tout l'intérêt de la zone morte, et c'est ce qui supprime le jitter.

**La fin du tableau est verrouillée.** Dès que le héros atteint 280 à l'écran, il y reste. La caméra prend exactement le relais, pixel pour pixel. Le décor défile, le héros ne bouge plus dans le cadre.

Notez enfin que la ligne verticale n'a jamais bougé : le héros est resté à l'ordonnée écran 100, dans la bande `[60, 140]`. Une zone morte verticale haute est précisément ce qui empêche l'écran de sauter avec le héros.

---

### Correction 6

```dart
import 'dart:math' as math;

class Secousse {
  Secousse({int graine = 2025}) : _alea = math.Random(graine);

  final math.Random _alea;

  double _intensite = 0;
  double amortissement = 20;

  double decalageX = 0;
  double decalageY = 0;

  double get intensite => _intensite;
  bool get active => _intensite > 0.01;

  /// Une secousse plus faible que celle en cours est ignorée :
  /// deux coups rapprochés ne doivent pas s'additionner à l'infini.
  void declencher(double intensite) {
    if (intensite > _intensite) _intensite = intensite;
  }

  void mettreAJour(double dt) {
    if (!active) {
      _intensite = 0;
      decalageX = 0;
      decalageY = 0;
      return;
    }
    decalageX = (_alea.nextDouble() * 2 - 1) * _intensite;
    decalageY = (_alea.nextDouble() * 2 - 1) * _intensite;
    _intensite = math.max(0, _intensite - amortissement * dt);
  }
}

void main() {
  final Secousse s = Secousse();
  const double dt = 0.05; // 20 FPS, pour un tableau court

  s.declencher(10);

  print('frame  intensite  evenement');
  print('-----  ---------  ---------------------------');

  for (int f = 1; f <= 12; f++) {
    String note = '';

    if (f == 5) {
      s.declencher(4); // plus faible que l'intensité courante : ignorée
      note = 'declencher(4) ignore';
    }
    if (f == 8) {
      s.declencher(12); // plus forte : elle prend le relais
      note = 'declencher(12) prend le relais';
    }

    s.mettreAJour(dt);

    print('${f.toString().padLeft(5)}  '
        '${s.intensite.toStringAsFixed(2).padLeft(9)}  $note');
  }

  print('');
  print('decalage final : '
      '(${s.decalageX.toStringAsFixed(2)}, '
      '${s.decalageY.toStringAsFixed(2)})');
  print('encore active  : ${s.active}');
}
```

**Résultat :**

```text
frame  intensite  evenement
-----  ---------  ---------------------------
    1       9.00  
    2       8.00  
    3       7.00  
    4       6.00  
    5       5.00  declencher(4) ignore
    6       4.00  
    7       3.00  
    8      11.00  declencher(12) prend le relais
    9      10.00  
   10       9.00  
   11       8.00  
   12       7.00  

decalage final : (...)
encore active  : true
```

**Explication :** l'amortissement vaut 20 pixels par seconde et `dt` vaut 0,05 seconde : chaque frame retire donc exactement 1,0 d'intensité. Le tableau se lit sans calculatrice.

**Frame 5 — la secousse faible est ignorée.** L'intensité courante est 6 ; `declencher(4)` ne fait rien, car `4 > 6` est faux. C'est le comportement voulu : si un gobelin vous effleure pendant l'explosion du boss, l'écran ne doit pas se calmer d'un coup.

**Frame 8 — la secousse forte prend le relais.** L'intensité courante est 3 ; `declencher(12)` la remplace, et l'amortissement repart de 12.

**Le choix du maximum plutôt que de l'addition** est important. Avec `_intensite += intensite`, dix petits coups successifs produiraient une secousse ingérable de plusieurs dizaines de pixels. Avec le maximum, l'effet reste toujours borné par le plus gros événement en cours.

Le décalage final n'est pas reproduit : il dépend du générateur aléatoire. Ce qui est testable, et testé ici, c'est la courbe d'intensité.

---

### Correction 7

```dart
import 'dart:ui';

class Tilemap {
  Tilemap({required this.tuiles, this.taille = 32});

  final List<List<int>> tuiles;
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;
  double get largeurMonde => colonnes * taille;
  double get hauteurMonde => lignes * taille;

  int tuileA(int c, int l) {
    if (l < 0 || l >= lignes) return 1;
    if (c < 0 || c >= colonnes) return 1;
    return tuiles[l][c];
  }

  bool solideA(int c, int l) => tuileA(c, l) != 0;

  Rect rectDe(int c, int l) =>
      Rect.fromLTWH(c * taille, l * taille, taille, taille);

  /// Construit une carte depuis un plan texte et une légende.
  /// Lève une [FormatException] explicite en cas de plan invalide.
  static Tilemap depuisTexte(
    List<String> plan,
    Map<String, int> legende, {
    double taille = 32,
  }) {
    if (plan.isEmpty) {
      throw const FormatException('Le plan du niveau est vide.');
    }

    final int largeur = plan.first.length;
    final List<List<int>> grille = <List<int>>[];

    for (int l = 0; l < plan.length; l++) {
      final String ligne = plan[l];

      if (ligne.length != largeur) {
        throw FormatException(
          'Ligne $l de longueur ${ligne.length}, attendu $largeur.',
        );
      }

      final List<int> cases = <int>[];
      for (int c = 0; c < ligne.length; c++) {
        final int? index = legende[ligne[c]];
        if (index == null) {
          throw FormatException(
            'Caractere inconnu "${ligne[c]}" en ligne $l, colonne $c.',
          );
        }
        cases.add(index);
      }
      grille.add(cases);
    }

    return Tilemap(tuiles: grille, taille: taille);
  }

  /// Compte les tuiles non vides.
  int get nombreDePleines {
    int total = 0;
    for (final List<int> ligne in tuiles) {
      for (final int index in ligne) {
        if (index != 0) total++;
      }
    }
    return total;
  }
}

const Map<String, int> kLegende = <String, int>{
  '.': 0,
  '#': 1,
  '=': 2,
  '^': 3,
};

void main() {
  const List<String> salle = <String>[
    '################',
    '#..............#',
    '#..===.........#',
    '#.........===..#',
    '#..............#',
    '#....^^^.......#',
    '################',
  ];

  final Tilemap carte = Tilemap.depuisTexte(salle, kLegende);

  print('grille  : ${carte.colonnes} x ${carte.lignes} tuiles');
  print('monde   : ${carte.largeurMonde} x ${carte.hauteurMonde} pixels');
  print('pleines : ${carte.nombreDePleines} / '
      '${carte.colonnes * carte.lignes}');
  print('tuile (3,2)   = ${carte.tuileA(3, 2)}');
  print('tuile (5,5)   = ${carte.tuileA(5, 5)}');
  print('hors carte    = ${carte.tuileA(-4, 99)}');
  print('rect de (3,2) = ${carte.rectDe(3, 2)}');
  print('');

  // Erreur 1 : lignes de longueurs différentes.
  try {
    Tilemap.depuisTexte(<String>['####', '#..#', '###'], kLegende);
  } on FormatException catch (e) {
    print('erreur 1 : ${e.message}');
  }

  // Erreur 2 : caractère absent de la légende.
  try {
    Tilemap.depuisTexte(<String>['####', '#.O#', '####'], kLegende);
  } on FormatException catch (e) {
    print('erreur 2 : ${e.message}');
  }

  // Erreur 3 : plan vide.
  try {
    Tilemap.depuisTexte(<String>[], kLegende);
  } on FormatException catch (e) {
    print('erreur 3 : ${e.message}');
  }
}
```

**Résultat :**

```text
grille  : 16 x 7 tuiles
monde   : 512.0 x 224.0 pixels
pleines : 51 / 112
tuile (3,2)   = 2
tuile (5,5)   = 3
hors carte    = 1
rect de (3,2) = Rect.fromLTRB(96.0, 64.0, 128.0, 96.0)

erreur 1 : Ligne 2 de longueur 3, attendu 4.
erreur 2 : Caractere inconnu "O" en ligne 1, colonne 2.
erreur 3 : Le plan du niveau est vide.
```

**Explication :** la fonction fait deux choses distinctes, et c'est volontaire : elle **convertit** et elle **valide**.

La validation n'est pas un luxe. Les trois erreurs testées sont exactement celles que vous ferez :

- une ligne de longueur différente, parce qu'un espace s'est glissé en fin de ligne ou qu'un caractère a été supprimé par erreur ;
- un `O` majuscule tapé à la place d'un zéro, ou un caractère décoratif oublié dans la légende ;
- un plan vide, parce qu'un fichier JSON n'a pas été trouvé.

Sans exception, ces trois cas donnent respectivement : un niveau bancal aux collisions décalées, un `Null check operator used on a null value` incompréhensible, et une division par zéro dans le calcul de la caméra. Avec exception, vous avez le numéro de ligne et de colonne. C'est la leçon du chapitre 13 appliquée aux données de jeu.

Notez `tuileA(-4, 99)` qui renvoie 1 sans planter : le monde est entouré de murs virtuels. Cette seule décision supprime tous les `RangeError` de la boucle de collision.

---

### Correction 8

```dart
import 'dart:ui';

class Tilemap {
  Tilemap({required this.tuiles, this.taille = 32});

  final List<List<int>> tuiles;
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;

  int tuileA(int c, int l) {
    if (l < 0 || l >= lignes) return 1;
    if (c < 0 || c >= colonnes) return 1;
    return tuiles[l][c];
  }

  bool solideA(int c, int l) => tuileA(c, l) != 0;
}

class Case {
  const Case(this.colonne, this.ligne);
  final int colonne;
  final int ligne;

  @override
  String toString() => '($colonne,$ligne)';
}

/// Toutes les cases recouvertes par [boite].
/// L'epsilon évite de compter la tuile suivante quand un bord
/// tombe exactement sur une frontière.
List<Case> tuilesSous(Rect boite, double taille) {
  const double eps = 0.001;

  final int colMin = (boite.left / taille).floor();
  final int ligMin = (boite.top / taille).floor();
  final int colMax = ((boite.right - eps) / taille).floor();
  final int ligMax = ((boite.bottom - eps) / taille).floor();

  final List<Case> cases = <Case>[];
  for (int l = ligMin; l <= ligMax; l++) {
    for (int c = colMin; c <= colMax; c++) {
      cases.add(Case(c, l));
    }
  }
  return cases;
}

bool estBloque(Tilemap carte, Rect boite) {
  for (final Case k in tuilesSous(boite, carte.taille)) {
    if (carte.solideA(k.colonne, k.ligne)) return true;
  }
  return false;
}

void main() {
  const double t = 32;

  // --- Partie 1 : la couverture ---
  print('PARTIE 1 — cases recouvertes (taille = 32)');
  print('');

  const Rect exact = Rect.fromLTWH(64, 0, 32, 32);
  const Rect cheval = Rect.fromLTWH(50, 50, 40, 40);
  const Rect negatif = Rect.fromLTWH(-10, -5, 24, 24);

  print('exact   $exact');
  print('  -> ${tuilesSous(exact, t)}  (${tuilesSous(exact, t).length} case)');
  print('cheval  $cheval');
  print('  -> ${tuilesSous(cheval, t)}  '
      '(${tuilesSous(cheval, t).length} cases)');
  print('negatif $negatif');
  print('  -> ${tuilesSous(negatif, t)}  '
      '(${tuilesSous(negatif, t).length} cases)');

  // --- Partie 2 : le blocage ---
  final Tilemap carte = Tilemap(tuiles: <List<int>>[
    <int>[1, 1, 1, 1, 1],
    <int>[1, 0, 0, 0, 1],
    <int>[1, 0, 2, 0, 1],
    <int>[1, 0, 0, 0, 1],
    <int>[1, 1, 1, 1, 1],
  ]);

  print('');
  print('PARTIE 2 — blocage');
  print('');
  print('centre libre   : '
      '${estBloque(carte, const Rect.fromLTWH(40, 40, 24, 24))}');
  print('touche le bois : '
      '${estBloque(carte, const Rect.fromLTWH(56, 56, 24, 24))}');
  print('dans le mur    : '
      '${estBloque(carte, const Rect.fromLTWH(0, 0, 32, 32))}');
  print('hors carte     : '
      '${estBloque(carte, const Rect.fromLTWH(-40, 40, 24, 24))}');
}
```

**Résultat :**

```text
PARTIE 1 — cases recouvertes (taille = 32)

exact   Rect.fromLTRB(64.0, 0.0, 96.0, 32.0)
  -> [(2,0)]  (1 case)
cheval  Rect.fromLTRB(50.0, 50.0, 90.0, 90.0)
  -> [(1,1), (2,1), (1,2), (2,2)]  (4 cases)
negatif Rect.fromLTRB(-10.0, -5.0, 14.0, 19.0)
  -> [(-1,-1), (0,-1), (-1,0), (0,0)]  (4 cases)

PARTIE 2 — blocage

centre libre   : false
touche le bois : true
dans le mur    : true
hors carte     : true
```

**Explication :** le premier cas est le plus instructif. Le rectangle `[64, 96[ x [0, 32[` occupe **exactement** une tuile. Sans epsilon, `96 / 32 = 3.0` et `floor` donnerait la colonne 3 : on testerait quatre cases au lieu d'une, dont trois à tort. Le héros collé contre un mur détecterait une collision avec le vide derrière et resterait bloqué. Un millième de pixel règle le problème.

Le cas `negatif` montre pourquoi `floor()` est obligatoire. `-10 / 32 = -0,3125`. `floor` donne `-1`, qui est la bonne case (elle couvre `[-32, 0[`). `toInt()` aurait donné `0`, une case qui ne contient pas ce point. Le bug apparaîtrait seulement quand le héros sort du monde par la gauche, c'est-à-dire au pire moment.

En partie 2, `hors carte : true` confirme le choix fait dans `tuileA` : à l'extérieur, tout est mur. Le héros ne peut donc pas quitter le niveau, sans aucun test supplémentaire dans la boucle de jeu.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() => runApp(const AppliFond());

class AppliFond extends StatelessWidget {
  const AppliFond({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0C0A16),
        body: EcranFond(),
      ),
    );
  }
}

class Camera {
  Camera({this.x = 0, this.y = 0, this.largeur = 0, this.hauteur = 0});

  double x;
  double y;
  double largeur;
  double hauteur;

  void borner(double lMonde, double hMonde) {
    x = lMonde <= largeur
        ? (lMonde - largeur) / 2
        : x.clamp(0.0, lMonde - largeur);
    y = hMonde <= hauteur
        ? (hMonde - hauteur) / 2
        : y.clamp(0.0, hMonde - hauteur);
  }
}

/// Un plan de décor : un nom, un facteur, une largeur de motif.
class Plan {
  const Plan(this.nom, this.facteur, this.largeurMotif);
  final String nom;
  final double facteur;
  final double largeurMotif;
}

const List<Plan> kPlans = <Plan>[
  Plan('ciel', 0.00, 0),
  Plan('montagnes', 0.25, 300),
  Plan('colonnes', 0.60, 200),
  Plan('sol', 1.00, 0),
];

class EcranFond extends StatefulWidget {
  const EcranFond({super.key});

  @override
  State<EcranFond> createState() => _EcranFondState();
}

class _EcranFondState extends State<EcranFond> {
  static const double largeurMonde = 3000;
  static const double hauteurMonde = 700;

  final Camera _camera = Camera();

  void _glisser(DragUpdateDetails d) {
    setState(() {
      // On tire le monde : la caméra part dans l'autre sens.
      _camera.x -= d.delta.dx;
      _camera.y -= d.delta.dy;
      _camera.borner(largeurMonde, hauteurMonde);
    });
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints c) {
        _camera.largeur = c.maxWidth;
        _camera.hauteur = c.maxHeight;
        _camera.borner(largeurMonde, hauteurMonde);
        return GestureDetector(
          behavior: HitTestBehavior.opaque,
          onPanUpdate: _glisser,
          child: CustomPaint(
            painter: PeintreFond(
              camera: _camera,
              largeurMonde: largeurMonde,
              hauteurMonde: hauteurMonde,
            ),
            size: Size.infinite,
          ),
        );
      },
    );
  }
}

class PeintreFond extends CustomPainter {
  PeintreFond({
    required this.camera,
    required this.largeurMonde,
    required this.hauteurMonde,
  });

  final Camera camera;
  final double largeurMonde;
  final double hauteurMonde;

  void _texte(Canvas canvas, String s, Offset p,
      {Color c = const Color(0xFFD8D8E8), double taille = 13}) {
    final TextPainter tp = TextPainter(
      text: TextSpan(text: s, style: TextStyle(color: c, fontSize: taille)),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, p);
  }

  /// Pavage par modulo : le motif se répète sur toute la largeur.
  void _paver({
    required Canvas canvas,
    required double largeurEcran,
    required double largeurMotif,
    required double decalage,
    required void Function(double x) motif,
  }) {
    double origine = decalage % largeurMotif;
    if (origine > 0) origine -= largeurMotif; // indispensable
    for (double x = origine; x < largeurEcran; x += largeurMotif) {
      motif(x);
    }
  }

  @override
  void paint(Canvas canvas, Size size) {
    final double sol = size.height * 0.76;

    // --- PLAN 0 : ciel, facteur 0. Aucune translation. ---
    canvas.drawRect(
      Offset.zero & size,
      Paint()
        ..shader = const LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: <Color>[Color(0xFF16102A), Color(0xFF3B2A54)],
        ).createShader(Offset.zero & size),
    );
    canvas.drawCircle(Offset(size.width - 80, 64), 24,
        Paint()..color = const Color(0xFFEDE6C4));

    // --- PLAN 1 : montagnes, facteur 0.25 ---
    final Paint montagne = Paint()..color = const Color(0xFF221838);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 300,
      decalage: -camera.x * 0.25,
      motif: (double x) {
        final Path p = Path()
          ..moveTo(x, sol)
          ..lineTo(x + 150, sol - 210)
          ..lineTo(x + 300, sol)
          ..close();
        canvas.drawPath(p, montagne);
      },
    );

    // --- PLAN 2 : colonnes du donjon, facteur 0.60 ---
    final Paint colonne = Paint()..color = const Color(0xFF33264F);
    _paver(
      canvas: canvas,
      largeurEcran: size.width,
      largeurMotif: 200,
      decalage: -camera.x * 0.60,
      motif: (double x) {
        canvas.drawRect(Rect.fromLTWH(x + 60, sol - 160, 36, 160), colonne);
        canvas.drawRect(Rect.fromLTWH(x + 50, sol - 174, 56, 16), colonne);
      },
    );

    // --- PLAN 3 : le sol, facteur 1.0 ---
    canvas.save();
    canvas.translate(-camera.x, -camera.y);
    canvas.drawRect(
      Rect.fromLTWH(0, sol + camera.y, largeurMonde, size.height),
      Paint()..color = const Color(0xFF1B1526),
    );
    final Paint dalle = Paint()..color = const Color(0xFF2C2340);
    for (double x = 0; x < largeurMonde; x += 70) {
      canvas.drawRect(
        Rect.fromLTWH(x + 3, sol + camera.y + 5, 64, 12),
        dalle,
      );
    }
    // Des repères tous les 500 pixels monde.
    for (double x = 0; x <= largeurMonde; x += 500) {
      canvas.drawRect(
        Rect.fromLTWH(x, sol + camera.y - 40, 3, 40),
        Paint()..color = const Color(0xFF6FD3B8),
      );
    }
    canvas.restore();

    // --- HUD, repère écran ---
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 290, 150),
      Paint()..color = const Color(0xCC000000),
    );
    _texte(canvas, 'DONJON DE DART — parallaxe', const Offset(12, 8));
    _texte(
      canvas,
      'camera : (${camera.x.toStringAsFixed(0)}, '
      '${camera.y.toStringAsFixed(0)})',
      const Offset(12, 30),
    );
    _texte(canvas, 'monde  : ${largeurMonde.toStringAsFixed(0)} x '
        '${hauteurMonde.toStringAsFixed(0)}', const Offset(12, 50));

    double ligne = 74;
    for (final Plan p in kPlans) {
      _texte(
        canvas,
        '${p.nom.padRight(10)} facteur ${p.facteur.toStringAsFixed(2)}',
        Offset(12, ligne),
        c: const Color(0xFF4EC9B0),
        taille: 12,
      );
      ligne += 17;
    }

    _texte(canvas, 'faites glisser pour deplacer la camera',
        Offset(12, size.height - 26), taille: 12);
  }

  @override
  bool shouldRepaint(PeintreFond oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART — parallaxe
camera : (0, 0)
monde  : 3000 x 700
ciel       facteur 0.00
montagnes  facteur 0.25
colonnes   facteur 0.60
sol        facteur 1.00

faites glisser pour deplacer la camera

Le décor comporte quatre profondeurs bien distinctes.
La lune ne bouge jamais, quel que soit le glissement.
Les montagnes se déplacent quatre fois moins vite que le sol.
Les colonnes se déplacent un peu plus vite que les montagnes,
mais nettement moins que le sol dallé.
Les repères turquoise du sol, espacés de 500 pixels monde, permettent de
vérifier que le plan de jeu défile exactement à la vitesse du glissement.
Arrivé au bord du monde, tout s'arrête net : la caméra est bornée et
aucun vide n'apparaît sur les côtés.
```

**Explication :** ce programme isole la seule idée de la parallaxe : **un facteur multiplicatif sur la translation**.

Les trois plans de fond ne passent jamais par `camera.appliquer` : ils sont dessinés en coordonnées écran, avec un décalage calculé à la main (`-camera.x * facteur`). Le plan de jeu, lui, utilise un vrai `translate(-camera.x, -camera.y)`.

Le `if (origine > 0) origine -= largeurMotif;` est la ligne à ne pas oublier. Le `%` de Dart renvoie un résultat du signe du dividende ; quand la caméra revient vers la gauche, `decalage` devient positif et l'origine tomberait à droite du bord gauche de l'écran. Il manquerait alors une colonne de motif, visible comme un trou qui apparaît et disparaît. Testez en retirant la ligne : le défaut saute aux yeux.

Le `sol + camera.y` du plan de jeu mérite un mot : la ligne de sol est exprimée en fraction de la hauteur de **l'écran**, alors que le canevas est déjà translaté en monde. On compense donc la translation verticale. Dans un vrai jeu, le sol serait une donnée du niveau en coordonnées monde, et cette compensation disparaîtrait.

Enfin, `borner` est appelé aussi dans `build`, pas seulement dans le glissement : si l'utilisateur redimensionne la fenêtre, la caméra pourrait se retrouver hors du monde sans qu'aucun glissement n'ait eu lieu.

---

### Correction 10

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter/services.dart';

void main() => runApp(const AppliSalle());

const double kTuile = 32;

const Map<String, int> kLegende = <String, int>{
  '.': 0,
  '#': 1,
  '=': 2,
  '^': 3,
};

const List<String> kPlanSalle = <String>[
  '############################################',
  '#..........................................#',
  '#..........................................#',
  '#..........................................#',
  '#..........................................#',
  '#..........................................#',
  '#.............======.......................#',
  '#..............................=======.....#',
  '#..........................................#',
  '#....======................................#',
  '#........................###...............#',
  '#......................#####...........^^^.#',
  '############################################',
  '############################################',
];

// =====================================================================
// TILEMAP
// =====================================================================

class Tilemap {
  Tilemap({required this.tuiles, this.taille = kTuile});

  final List<List<int>> tuiles;
  final double taille;

  int get lignes => tuiles.length;
  int get colonnes => tuiles.isEmpty ? 0 : tuiles[0].length;
  double get largeurMonde => colonnes * taille;
  double get hauteurMonde => lignes * taille;
  int get total => colonnes * lignes;

  int tuileA(int c, int l) {
    if (l < 0 || l >= lignes) return 1;
    if (c < 0 || c >= colonnes) return 1;
    return tuiles[l][c];
  }

  bool solideA(int c, int l) => tuileA(c, l) != 0;

  Rect rectDe(int c, int l) =>
      Rect.fromLTWH(c * taille, l * taille, taille, taille);

  static Tilemap depuisTexte(List<String> plan) {
    final int largeur = plan.first.length;
    final List<List<int>> grille = <List<int>>[];
    for (int l = 0; l < plan.length; l++) {
      if (plan[l].length != largeur) {
        throw FormatException('Ligne $l de longueur ${plan[l].length}, '
            'attendu $largeur.');
      }
      final List<int> cases = <int>[];
      for (int c = 0; c < plan[l].length; c++) {
        final int? index = kLegende[plan[l][c]];
        if (index == null) {
          throw FormatException(
              'Caractere inconnu "${plan[l][c]}" ligne $l colonne $c.');
        }
        cases.add(index);
      }
      grille.add(cases);
    }
    return Tilemap(tuiles: grille);
  }
}

// =====================================================================
// CAMÉRA
// =====================================================================

class Camera {
  Camera({this.x = 0, this.y = 0, this.largeur = 0, this.hauteur = 0});

  double x;
  double y;
  double largeur;
  double hauteur;

  double vitesseSuivi = 8;
  double zoneMorteX = 80;
  double zoneMorteY = 60;

  Offset cible = Offset.zero;

  Rect get vue => Rect.fromLTWH(x, y, largeur, hauteur);
  Rect get vueElargie => vue.inflate(64);

  Offset versEcran(Offset m) => Offset(m.dx - x, m.dy - y);
  Offset versMonde(Offset e) => Offset(e.dx + x, e.dy + y);

  void placerImmediatement() {
    x = cible.dx - largeur / 2;
    y = cible.dy - hauteur / 2;
  }

  void mettreAJour(double dt) {
    // 1. Zone morte : on part de la position actuelle.
    double cibleX = x;
    double cibleY = y;

    final Offset e = versEcran(cible);
    final double gauche = largeur / 2 - zoneMorteX;
    final double droite = largeur / 2 + zoneMorteX;
    final double haut = hauteur / 2 - zoneMorteY;
    final double bas = hauteur / 2 + zoneMorteY;

    if (e.dx > droite) cibleX += e.dx - droite;
    if (e.dx < gauche) cibleX -= gauche - e.dx;
    if (e.dy > bas) cibleY += e.dy - bas;
    if (e.dy < haut) cibleY -= haut - e.dy;

    // 2. Lissage indépendant du framerate.
    final double t = 1 - math.exp(-vitesseSuivi * dt);
    x += (cibleX - x) * t;
    y += (cibleY - y) * t;
  }

  /// En dernier, toujours.
  void borner(double lMonde, double hMonde) {
    x = lMonde <= largeur
        ? (lMonde - largeur) / 2
        : x.clamp(0.0, lMonde - largeur);
    y = hMonde <= hauteur
        ? (hMonde - hauteur) / 2
        : y.clamp(0.0, hMonde - hauteur);
  }

  void appliquer(Canvas canvas) {
    canvas.save();
    canvas.translate(-x, -y);
  }

  void retirer(Canvas canvas) => canvas.restore();
}

// =====================================================================
// HÉROS
// =====================================================================

class Heros {
  Heros({required this.x, required this.y});

  static const double largeur = 24;
  static const double hauteur = 40;
  static const double vitesse = 180;
  static const double gravite = 1100;
  static const double impulsion = 450;

  double x;
  double y;
  double vx = 0;
  double vy = 0;
  bool auSol = false;
  bool regardeADroite = true;

  Rect get boite => Rect.fromLTWH(x, y, largeur, hauteur);
  Offset get centre => Offset(x + largeur / 2, y + hauteur / 2);

  void deplacer(Tilemap carte, double dt) {
    final double pasMax = carte.taille * 0.9;

    x += (vx * dt).clamp(-pasMax, pasMax);
    _resoudreX(carte);

    vy = math.min(vy + gravite * dt, 1400);
    y += (vy * dt).clamp(-pasMax, pasMax);
    auSol = false;
    _resoudreY(carte);
  }

  void _resoudreX(Tilemap carte) {
    if (vx == 0) return;
    const double eps = 0.001;
    final Rect b = boite;
    final int ligHaut = (b.top / carte.taille).floor();
    final int ligBas = ((b.bottom - eps) / carte.taille).floor();

    if (vx > 0) {
      final int col = ((b.right - eps) / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = col * carte.taille - largeur;
          return;
        }
      }
    } else {
      final int col = (b.left / carte.taille).floor();
      for (int l = ligHaut; l <= ligBas; l++) {
        if (carte.solideA(col, l)) {
          x = (col + 1) * carte.taille;
          return;
        }
      }
    }
  }

  void _resoudreY(Tilemap carte) {
    const double eps = 0.001;
    final Rect b = boite;
    final int colG = (b.left / carte.taille).floor();
    final int colD = ((b.right - eps) / carte.taille).floor();

    if (vy > 0) {
      final int lig = ((b.bottom - eps) / carte.taille).floor();
      for (int c = colG; c <= colD; c++) {
        if (carte.solideA(c, lig)) {
          y = lig * carte.taille - hauteur;
          vy = 0;
          auSol = true;
          return;
        }
      }
    } else if (vy < 0) {
      final int lig = (b.top / carte.taille).floor();
      for (int c = colG; c <= colD; c++) {
        if (carte.solideA(c, lig)) {
          y = (lig + 1) * carte.taille;
          vy = 0;
          return;
        }
      }
    }
  }
}

// =====================================================================
// APPLICATION
// =====================================================================

class AppliSalle extends StatelessWidget {
  const AppliSalle({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0B0912),
        body: EcranSalle(),
      ),
    );
  }
}

class EcranSalle extends StatefulWidget {
  const EcranSalle({super.key});

  @override
  State<EcranSalle> createState() => _EcranSalleState();
}

class _EcranSalleState extends State<EcranSalle>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  final FocusNode _focus = FocusNode();
  Duration _precedent = Duration.zero;

  final Tilemap _carte = Tilemap.depuisTexte(kPlanSalle);
  final Camera _camera = Camera();
  late final Heros _heros =
      Heros(x: 3 * kTuile, y: 12 * kTuile - Heros.hauteur);

  bool _gauche = false;
  bool _droite = false;
  bool _saut = false;
  bool _placee = false;
  double _fps = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    _focus.dispose();
    super.dispose();
  }

  void _tick(Duration horodatage) {
    double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0) return;
    if (dt > 0.05) dt = 0.05;
    if (_camera.largeur == 0) return;

    _fps = 1 / dt;

    // Entrées.
    final double direction = (_droite ? 1.0 : 0.0) - (_gauche ? 1.0 : 0.0);
    _heros.vx = direction * Heros.vitesse;
    if (direction != 0) _heros.regardeADroite = direction > 0;
    if (_saut && _heros.auSol) {
      _heros.vy = -Heros.impulsion;
      _heros.auSol = false;
    }

    // Monde.
    _heros.deplacer(_carte, dt);

    // Caméra : cible, puis zone morte et lissage, puis bornage.
    _camera.cible = _heros.centre - const Offset(0, 20);
    if (!_placee) {
      _camera.placerImmediatement();
      _placee = true;
    }
    _camera.mettreAJour(dt);
    _camera.borner(_carte.largeurMonde, _carte.hauteurMonde);

    setState(() {});
  }

  void _clavier(KeyEvent e) {
    if (e is KeyRepeatEvent) return;
    final bool appui = e is KeyDownEvent;
    final LogicalKeyboardKey k = e.logicalKey;

    if (k == LogicalKeyboardKey.arrowLeft || k == LogicalKeyboardKey.keyQ) {
      _gauche = appui;
    } else if (k == LogicalKeyboardKey.arrowRight ||
        k == LogicalKeyboardKey.keyD) {
      _droite = appui;
    } else if (k == LogicalKeyboardKey.space ||
        k == LogicalKeyboardKey.arrowUp) {
      _saut = appui;
    }
  }

  void _tapDown(TapDownDetails d) {
    final double tiers = _camera.largeur / 3;
    if (d.localPosition.dx < tiers) {
      _gauche = true;
    } else if (d.localPosition.dx > 2 * tiers) {
      _droite = true;
    } else {
      _saut = true;
    }
  }

  void _tapUp() {
    _gauche = false;
    _droite = false;
    _saut = false;
  }

  @override
  Widget build(BuildContext context) {
    return KeyboardListener(
      focusNode: _focus,
      autofocus: true,
      onKeyEvent: _clavier,
      child: GestureDetector(
        behavior: HitTestBehavior.opaque,
        onTapDown: _tapDown,
        onTapUp: (TapUpDetails d) => _tapUp(),
        onTapCancel: _tapUp,
        child: LayoutBuilder(
          builder: (BuildContext context, BoxConstraints c) {
            _camera.largeur = c.maxWidth;
            _camera.hauteur = c.maxHeight;
            return CustomPaint(
              painter: PeintreSalle(
                camera: _camera,
                carte: _carte,
                heros: _heros,
                fps: _fps,
              ),
              size: Size.infinite,
            );
          },
        ),
      ),
    );
  }
}

class PeintreSalle extends CustomPainter {
  PeintreSalle({
    required this.camera,
    required this.carte,
    required this.heros,
    required this.fps,
  });

  final Camera camera;
  final Tilemap carte;
  final Heros heros;
  final double fps;

  static final Map<int, Paint> _peintures = <int, Paint>{
    1: Paint()..color = const Color(0xFF443A5E),
    2: Paint()..color = const Color(0xFF7A5230),
    3: Paint()..color = const Color(0xFF9E3B4A),
  };
  static final Paint _bord = Paint()..color = const Color(0xFF6A5A8C);

  void _texte(Canvas canvas, String s, Offset p,
      {Color c = const Color(0xFFD8D8E8)}) {
    final TextPainter tp = TextPainter(
      text: TextSpan(text: s, style: TextStyle(color: c, fontSize: 13)),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, p);
  }

  int _dessinerTuiles(Canvas canvas) {
    final Rect vue = camera.vueElargie;
    final int colMin =
        (vue.left / carte.taille).floor().clamp(0, carte.colonnes - 1);
    final int colMax =
        (vue.right / carte.taille).ceil().clamp(0, carte.colonnes - 1);
    final int ligMin =
        (vue.top / carte.taille).floor().clamp(0, carte.lignes - 1);
    final int ligMax =
        (vue.bottom / carte.taille).ceil().clamp(0, carte.lignes - 1);

    int compte = 0;
    for (int l = ligMin; l <= ligMax; l++) {
      for (int c = colMin; c <= colMax; c++) {
        final int index = carte.tuiles[l][c];
        if (index == 0) continue; // le vide ne se dessine pas
        final Rect r = carte.rectDe(c, l);
        canvas.drawRect(
          r,
          _peintures[index] ?? (Paint()..color = const Color(0xFFFF00FF)),
        );
        if (!carte.solideA(c, l - 1)) {
          canvas.drawRect(Rect.fromLTWH(r.left, r.top, r.width, 4), _bord);
        }
        compte++;
      }
    }
    return compte;
  }

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = const Color(0xFF120F1E),
    );

    // ----- repère monde -----
    camera.appliquer(canvas);

    final int tuiles = _dessinerTuiles(canvas);

    final Rect b = heros.boite;
    canvas.drawRect(
      Rect.fromLTWH(b.left, b.top + 12, b.width, b.height - 12),
      Paint()..color = const Color(0xFF4EA3D8),
    );
    canvas.drawRect(
      Rect.fromLTWH(b.left + 3, b.top, b.width - 6, 13),
      Paint()..color = const Color(0xFFE0B48C),
    );
    canvas.drawRect(
      Rect.fromLTWH(
          heros.regardeADroite ? b.right - 9 : b.left + 6, b.top + 4, 3, 3),
      Paint()..color = const Color(0xFF20202C),
    );

    camera.retirer(canvas);
    // ----- repère écran -----

    // La zone morte, dessinée pour comprendre ce qui se passe.
    canvas.drawRect(
      Rect.fromLTWH(
        size.width / 2 - camera.zoneMorteX,
        size.height / 2 - camera.zoneMorteY,
        camera.zoneMorteX * 2,
        camera.zoneMorteY * 2,
      ),
      Paint()
        ..color = const Color(0x334EC9B0)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 2,
    );

    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 290, 132),
      Paint()..color = const Color(0xCC000000),
    );
    _texte(canvas, 'DONJON DE DART — une salle', const Offset(12, 8));
    _texte(
      canvas,
      'tuiles dessinees : $tuiles / ${carte.total}',
      const Offset(12, 30),
      c: const Color(0xFF4EC9B0),
    );
    _texte(canvas, 'auSol            : ${heros.auSol}', const Offset(12, 50));
    _texte(
      canvas,
      'heros            : (${heros.x.toStringAsFixed(0)}, '
      '${heros.y.toStringAsFixed(0)})',
      const Offset(12, 70),
    );
    _texte(
      canvas,
      'camera           : (${camera.x.toStringAsFixed(0)}, '
      '${camera.y.toStringAsFixed(0)})',
      const Offset(12, 90),
    );
    _texte(canvas, 'FPS              : ${fps.toStringAsFixed(0)}',
        const Offset(12, 110));
    _texte(
      canvas,
      'fleches / espace, ou touchez gauche - centre - droite',
      Offset(12, size.height - 26),
    );
  }

  @override
  bool shouldRepaint(PeintreSalle oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART — une salle
tuiles dessinees : 115 / 616
auSol            : true
heros            : (96, 344)
camera           : (0, 48)
FPS              : 60

fleches / espace, ou touchez gauche - centre - droite

Une salle de 44 sur 14 tuiles, soit 1408 sur 448 pixels, s'affiche.
Le héros bleu se tient sur le sol de pierre à gauche.
Les flèches le font marcher, l'espace le fait sauter sur les plateformes
de bois et sur le petit escalier de pierre du milieu.
Le rectangle turquoise au centre matérialise la zone morte : tant que le
héros y reste, la caméra ne bouge pas et c'est lui qui se déplace dans
le cadre. Dès qu'il en sort, la caméra glisse en douceur.
Aux deux extrémités de la salle, la caméra s'arrête : le bornage empêche
tout vide d'apparaître, et le héros continue d'avancer vers le bord.
Le compteur de tuiles reste très inférieur au total : le culling ne
dessine que ce qui entre dans la vue élargie.
Le héros ne traverse jamais un mur, ne s'accroche pas aux coins, et
« auSol » repasse à false dès qu'il quitte une plateforme.
```

**Explication :** cette correction est une version réduite du mini-projet, sans parallaxe, sans objets et sans transitions, pour que les quatre mécanismes essentiels restent lisibles.

**La chaîne de la caméra.** Dans `_tick`, l'ordre est strict : cible → zone morte et lissage → bornage. Le bornage vient toujours en dernier. Le `-Offset(0, 20)` vise légèrement au-dessus du héros, comme expliqué à la section 25.7.

**Le drapeau `_placee`.** À la toute première frame, la caméra est en `(0, 0)` alors que le héros est déjà quelque part. Sans `placerImmediatement()`, on verrait la caméra « voler » vers lui pendant une demi-seconde. C'est le même problème qu'au chargement d'une salle (section 25.29).

**La résolution axe par axe.** `_resoudreX` puis `_resoudreY`, chacun avec son epsilon et sa remise à zéro de vitesse. Le `auSol = false` placé entre les deux garantit qu'un héros qui sort d'une plateforme ne peut plus sauter. Essayez de le déplacer ailleurs : vous obtiendrez un double saut involontaire.

**Le compteur de culling est un outil.** S'il affiche 616 sur 616, quelque chose ne va pas dans le calcul de la plage visible. Gardez ce genre d'indicateur pendant tout le développement, et retirez-le à la livraison.

**La zone morte dessinée à l'écran.** Ce rectangle turquoise n'a aucune existence dans le monde : il est peint après `retirer(canvas)`, en coordonnées écran. C'est exactement pour cela qu'il reste immobile alors que tout défile derrière lui, et c'est une excellente démonstration de la séparation des deux repères.

---

## Et maintenant ?

Vous disposez maintenant d'un monde. Un vrai : plus grand que l'écran, découpé en salles, décrit par des données plutôt que par du code, parcouru par une caméra qui suit le héros sans le secouer, et dessiné sans gaspiller une seule tuile.

Mais regardez la classe `_EcranDonjonState` du mini-projet. Elle contient la boucle, les entrées, la physique, les collisions, la caméra, le chargement des salles, les transitions, les points de vie et le score. Elle fait tout. À ce stade, elle tient encore ; ajoutez un menu principal, une pause, un écran de Game Over, un boss avec trois phases, et elle deviendra impossible à faire évoluer.

Le chapitre 26 s'attaque précisément à ce problème. Vous y apprendrez à décrire un jeu comme un ensemble d'**entités** interchangeables plutôt que comme une liste de champs, à séparer strictement la logique du rendu, et à piloter le jeu par une **machine à états** : menu, en jeu, pause, game over, victoire. Vous découvrirez aussi le principe de l'architecture ECS, celle que Flame adopte à sa manière avec ses `Component`, ce qui rendra le chapitre 28 immédiatement familier.

Autrement dit : vous avez construit le monde, il est temps de construire le programme qui saura le faire vivre longtemps.

Chapitre suivant : [26-PARTIE-2A—ARCHITECTURE-ET-ÉTATS-DUN-JEU.md](./26-PARTIE-2A—ARCHITECTURE-ET-ÉTATS-DUN-JEU.md)
