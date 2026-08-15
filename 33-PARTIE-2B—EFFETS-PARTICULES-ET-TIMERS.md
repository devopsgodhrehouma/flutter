# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 33 — EFFETS, PARTICULES ET TIMERS

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitres 27 à 32 (installation de Flame, composants et cycle de vie, sprites, entrées, caméra, collisions), chapitre 26 (architecture et états), chapitre 23 (vecteurs et vitesse), chapitre 15 (asynchrone)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** transformer une scène du « Donjon de Dart » qui *fonctionne* en une scène qui *fait plaisir à jouer*, à l'aide d'effets, de particules et de minuteries, sans un seul fichier image.

---

## 33.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est le **game feel** et pourquoi un jeu « sec » lasse en deux minutes ;
- décrire le fonctionnement interne d'un **effet** Flame : un composant qui applique une progression ;
- utiliser `MoveEffect.by()` et `MoveEffect.to()` et choisir entre les deux ;
- animer une échelle avec `ScaleEffect` et comprendre la différence avec `SizeEffect` ;
- faire tourner un composant avec `RotateEffect` en radians, et déplacer un pivot avec `AnchorToEffect` ;
- faire apparaître et disparaître un composant avec `OpacityEffect` ;
- teinter un composant avec `ColorEffect` et connaître sa limitation ;
- paramétrer un `EffectController` : `duration`, `reverseDuration`, `alternate`, `repeatCount`, `infinite`, `startDelay` ;
- lire un **chronogramme d'effet** et prévoir ce que l'on va voir à l'écran ;
- choisir une **courbe** d'interpolation adaptée à l'intention (`easeOut`, `elasticOut`, `bounceOut`…) ;
- enchaîner des effets avec `SequenceEffect` et les superposer en parallèle ;
- utiliser `onComplete` et `removeOnFinish` sans fuite de composants ;
- coder le **flash blanc** d'un ennemi touché et le **pop** d'un objet ramassé ;
- coder un **tremblement d'écran** en appliquant un effet au `Viewfinder` ;
- distinguer `Particle` et `ParticleSystemComponent` ;
- utiliser `CircleParticle`, `SpriteParticle`, `ComputedParticle`, `AcceleratedParticle` et `MovingParticle` ;
- générer une gerbe avec `Particle.generate` : explosion de pièces, poussière, étincelles ;
- estimer le **coût** d'un système de particules et le brider ;
- utiliser la classe `Timer` de Flame, son couple `update(dt)` / `onTick`, et `TimerComponent` ;
- faire apparaître un ennemi toutes les trois secondes ;
- implémenter un **cooldown** d'attaque et une **invincibilité temporaire** avec clignotement ;
- assembler le tout dans une scène jouable et « juteuse » du « Donjon de Dart ».

---

## 33.1 — Le game feel : pourquoi un jeu « sec » n'est pas amusant

Depuis le chapitre 27, vous savez faire tourner un jeu. Le héros bouge, le gobelin meurt, la pièce disparaît, le score monte. Tout est **correct**. Et pourtant, si vous faites tester votre jeu à quelqu'un, il repose le téléphone au bout de quatre-vingt-dix secondes.

Ce n'est pas un problème de contenu. C'est un problème de **retour**. Regardons ce qui se passe quand le héros frappe un gobelin, dans la version « sèche » du chapitre 32 puis dans la version de ce chapitre.

```text
  VERSION SÈCHE                          VERSION « JUTEUSE »

  frame 0  le héros touche le gobelin    t=0.00  le héros touche le gobelin
  frame 1  gobelin.vies : 2 -> 1         t=0.00  le gobelin devient BLANC
  frame 2  rien                          t=0.00  il est repoussé de 12 px
  frame 3  rien                          t=0.00  l'écran tremble 0,15 s
  ...                                    t=0.00  6 étincelles jaillissent
  frame 60 rien                          t=0.05  un « -1 » monte au-dessus
                                         t=0.10  il retrouve sa couleur
                                         t=0.40  les étincelles ont disparu

  Retour visuel : 0 image.               Retour visuel : environ 48 images.
  Information transmise : aucune.        « ça a touché, ça a fait 1 dégât,
                                           c'était fort ».
```

Rien n'a changé dans la **logique** : le gobelin perd toujours un point de vie. Ce qui a changé, c'est que la logique est maintenant **visible**. C'est cela, le game feel.

Le joueur qui n'obtient aucune réponse ne peut rien apprendre : il appuie à nouveau, au hasard. Un jeu qui ne répond pas est un jeu qu'on ne peut pas apprendre à jouer.

| Qualité d'un bon retour | Question à laquelle elle répond | Outil du chapitre |
| --- | --- | --- |
| **Immédiateté** | « Est-ce que mon action est partie ? » | effet lancé dans la même frame que l'action |
| **Lisibilité** | « Qu'est-ce qui s'est passé exactement ? » | couleur, direction du recul, nombre de particules |
| **Proportionnalité** | « C'était fort ou faible ? » | amplitude et durée de l'effet |

La proportionnalité est la plus souvent oubliée : si un coup normal et un coup critique produisent le même flash, le joueur n'apprend rien.

> **À retenir.** Le game feel n'est pas de la décoration. C'est un **canal d'information** entre le jeu et le joueur. Chaque effet doit répondre à une question que le joueur se pose.

Dernier point : le game feel a un **budget de temps**, car un effet qui dure une seconde bloque la lecture pendant une seconde.

```text
  BUDGET DE TEMPS DES EFFETS (valeurs de départ éprouvées)

  0.05 – 0.10 s   flash de dégât, clignotement d'un cadre
  0.10 – 0.20 s   tremblement d'écran, recul d'un ennemi
  0.15 – 0.30 s   pop d'un objet ramassé, rebond d'un bouton
  0.30 – 0.60 s   gerbe de particules, texte de dégât qui monte
  0.60 – 1.20 s   transition d'écran, ouverture de porte
  > 1.5 s         RÉSERVÉ aux cinématiques : jamais pendant l'action
```

En cas d'hésitation, partez de ce tableau puis divisez par deux : les débutants font toujours des effets trop longs.

---

## 33.2 — Les effets Flame : le principe

Un effet, dans Flame, n'est pas une méthode magique. C'est un **composant** comme un autre, que l'on ajoute à sa cible et qui se supprime tout seul quand il a fini.

```dart
fleur.add(MoveEffect.by(Vector2(30, 30), EffectController(duration: 1.0)));
```

```text
  ANATOMIE D'UN EFFET

  fleur.add(  MoveEffect.by(  Vector2(30, 30),  EffectController(duration: 1.0)  ));
     │              │                │                        │
     │              │                │                        └─ QUAND : le contrôleur
     │              │                │                           de temps, qui produit
     │              │                │                           une progression 0 -> 1.
     │              │                └─ COMBIEN : valeur cible ou déplacement.
     │              └─ QUOI : la propriété animée (position, échelle, angle…).
     └─ QUI : la cible. L'effet devient un ENFANT de ce composant.
```

À chaque frame, Flame appelle `update(dt)` sur l'effet ; celui-ci transmet `dt` à son `EffectController`, qui avance sa **progression** entre `0.0` et `1.0` ; l'effet appelle alors `apply(progress)`, qui modifie la propriété du parent. Quand le contrôleur signale `completed`, l'effet appelle `onComplete` puis se retire de l'arbre.

```text
  add() -> apply(0.02) -> apply(0.05) -> ... -> apply(1.00)
        -> onComplete()         (si vous en avez fourni un)
        -> removeFromParent()   (removeOnFinish vaut true par défaut)
```

Trois conséquences expliquent la moitié des bugs d'effets. **Un effet occupe une place dans l'arbre** : en ajouter un à chaque frame en crée soixante par seconde, tous en train de tirer sur la même propriété (section 33.18). **Un effet `.by` ne connaît pas la valeur finale absolue** : il applique à chaque frame la différence entre la progression actuelle et la précédente, ce qui permet d'empiler deux déplacements sans qu'ils se battent. **Un effet se supprime seul** : `removeOnFinish` vaut `true` par défaut, il n'y a rien à nettoyer dans le cas courant.

| Effet | Propriété animée | Section |
| --- | --- | --- |
| `MoveEffect.by` / `.to`, `MoveAlongPathEffect` | `position` | 33.3 |
| `ScaleEffect.by` / `.to` | `scale` | 33.4 |
| `RotateEffect.by` / `.to`, `RotateAroundEffect` | `angle` | 33.5 |
| `OpacityEffect.to` / `.by` / `.fadeIn` / `.fadeOut` | opacité de la `Paint` | 33.6 |
| `ColorEffect` | teinte de la `Paint` | 33.7 |
| `SizeEffect.by` / `.to` | `size` | 33.8 |
| `AnchorToEffect` / `AnchorByEffect` | `anchor` | 33.9 |
| `SequenceEffect` | enchaînement d'effets | 33.16 |
| `RemoveEffect` | retrait différé du composant | 33.18 |

Tous s'importent depuis un seul fichier : `import 'package:flame/effects.dart';`.

---

## 33.3 — `MoveEffect.by()` et `MoveEffect.to()`

`MoveEffect` est une classe **abstraite**. On ne l'instancie jamais directement : on passe par l'une de ses deux fabriques.

```dart
// Déplacement RELATIF : « avance de 30 px vers la droite et 30 px vers le bas ».
MoveEffect.by(Vector2(30, 30), EffectController(duration: 1.0));

// Déplacement ABSOLU : « va au point (30, 30) du repère du parent ».
MoveEffect.to(Vector2(30, 30), EffectController(duration: 1.0));
```

La différence tient en une phrase : **`by` ajoute, `to` remplace.**

```text
  Le héros est en (100, 200).
  MoveEffect.by(Vector2(30, 0), ...)   ->  il finit en (130, 200)
  MoveEffect.to(Vector2(30, 0), ...)   ->  il finit en  (30, 200)

  On relance le même effet depuis (130, 200) :
  MoveEffect.by(Vector2(30, 0), ...)   ->  (160, 200)   <- cumulatif
  MoveEffect.to(Vector2(30, 0), ...)   ->   (30, 200)   <- toujours le même point
```

**Utilisez `by`** pour un recul, un saut, un tremblement : tout ce qui s'exprime « par rapport à là où je suis », soit le cas le plus fréquent en game feel. **Utilisez `to`** pour rejoindre une destination connue : une case de grille, un emplacement de HUD, un point de patrouille.

Voici le **banc d'essai** du chapitre : gardez-le sous la main, les sections suivantes se contenteront de remplacer la ligne de l'effet.

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() =>
    runApp(MaterialApp(home: Scaffold(body: GameWidget(game: BancDEssai()))));

/// Couleurs du « Donjon de Dart », utilisées dans tout le chapitre.
class Palette {
  static const Color dalle = Color(0xFF2B2B3A);
  static const Color heros = Color(0xFF4CAF50);
  static const Color gobelin = Color(0xFFB0413E);
  static const Color piece = Color(0xFFE8B04B);
  static const Color potion = Color(0xFF7E57C2);
  static const Color blanc = Color(0xFFFFFFFF);
}

class BancDEssai extends FlameGame {
  late final RectangleComponent cible;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent(
        position: Vector2(20, 20),
        size: Vector2(360, 240),
        paint: Paint()..color = Palette.dalle));

    cible = RectangleComponent(
        position: Vector2(80, 140),
        size: Vector2(48, 48),
        anchor: Anchor.center, // indispensable pour les effets d'échelle
        paint: Paint()..color = Palette.heros);
    await world.add(cible);

    // L'EFFET À TESTER : on le remplacera dans les sections suivantes.
    cible.add(MoveEffect.by(
      Vector2(200, 0),
      EffectController(duration: 1.5, alternate: true, infinite: true),
    ));
  }
}
```

**Résultat :** un carré vert de 48 px fait l'aller-retour entre `x = 80` et `x = 280`, en 1,5 s à l'aller et 1,5 s au retour, indéfiniment.

Notez `anchor: Anchor.center` : avec l'ancre par défaut, toutes les rotations et mises à l'échelle se feraient depuis le coin supérieur gauche. **Pour un composant destiné à recevoir des effets, mettez l'ancre au centre.**

Le code source précise que les fabriques `MoveEffect.by` et `MoveEffect.to` « pourraient être dépréciées à l'avenir » ; les classes concrètes `MoveByEffect` et `MoveToEffect` prennent les mêmes arguments et sont strictement équivalentes.

Il existe un troisième déplacement, plus rare, qui suit un tracé Flutter :

```dart
cible.add(MoveAlongPathEffect(
  Path()..quadraticBezierTo(100, 0, 50, -50),
  EffectController(duration: 1.5),
  // absolute: false -> chemin relatif à la position actuelle
  // oriented: true  -> le composant pivote pour suivre la tangente
));
```

---

## 33.4 — `ScaleEffect`

`ScaleEffect` anime la propriété `scale`, un facteur multiplicatif appliqué **autour de l'ancre**.

```dart
ScaleEffect.by(Vector2.all(1.5), EffectController(duration: 0.3)); // relatif
ScaleEffect.to(Vector2.all(0.5), EffectController(duration: 0.5)); // absolu
```

Attention : le vecteur de `ScaleEffect.by` est un **multiplicateur**, pas un ajout.

```text
  scale de départ : 1.0
  ScaleEffect.by(Vector2.all(1.5), ...)  -> 1.5   (1.0 x 1.5)
  ScaleEffect.by(Vector2.all(1.5), ...)  -> 2.25  (1.5 x 1.5)
  ScaleEffect.to(Vector2.all(1.5), ...)  -> 1.5   (quelle que soit la valeur avant)
```

Le vecteur ayant deux composantes, on peut déformer de façon non uniforme : c'est la base du **squash and stretch**, la technique d'animation la plus rentable qui existe.

```dart
// SQUASH : le personnage s'écrase à l'atterrissage.
cible.add(ScaleEffect.to(
    Vector2(1.3, 0.7), EffectController(duration: 0.08, alternate: true)));

// STRETCH : il s'étire au décollage.
cible.add(ScaleEffect.to(
    Vector2(0.75, 1.35), EffectController(duration: 0.10, alternate: true)));
```

Remplacez l'effet du banc d'essai par celui-ci pour voir un « battement de cœur » : le carré grossit de 25 %, revient à sa taille et recommence, une pulsation complète durant 0,7 s.

```dart
cible.add(ScaleEffect.by(
  Vector2.all(1.25),
  EffectController(
      duration: 0.35, alternate: true, infinite: true, curve: Curves.easeInOut),
));
```

> **Remarque.** `scale` étant appliqué autour de l'**ancre**, un composant d'ancre `Anchor.topLeft` grossit vers le bas et vers la droite : il a l'air de glisser. Avec `Anchor.center`, il grossit dans toutes les directions à la fois.

---

## 33.5 — `RotateEffect`

`RotateEffect` anime la propriété `angle`, **exprimée en radians**, autour de l'ancre du composant.

```dart
RotateEffect.by(tau / 4, EffectController(duration: 2)); // un quart de tour de plus
RotateEffect.to(tau / 4, EffectController(duration: 2)); // se placer à un quart de tour
```

La constante `tau` vaut `2 * pi`, soit un tour complet. Elle est fournie par `import 'package:flame/geometry.dart';` ; `pi` vient de `dart:math`.

| Angle en degrés | 15° | 90° | 180° | 360° |
| --- | --- | --- | --- | --- |
| En radians | `tau / 24` | `tau / 4` ou `pi / 2` | `tau / 2` ou `pi` | `tau` |
| Valeur | 0,2618 | 1,5708 | 3,1416 | 6,2832 |

Un cercle qui tourne ne se voit pas ; pour une pièce, on simule la rotation en écrasant sa largeur.

```dart
piece.add(ScaleEffect.to(
  Vector2(0.1, 1.0),
  EffectController(duration: 0.6, alternate: true, infinite: true),
));
```

Le tremblement de colère d'un boss est une petite oscillation autour de zéro :

```dart
boss.add(RotateEffect.by(
  tau / 40, // environ 9 degrés
  EffectController(duration: 0.06, alternate: true, repeatCount: 6),
));
```

Enfin, `RotateAroundEffect` fait tourner un composant autour d'un **autre point** que son ancre : c'est une orbite.

```dart
satellite.add(RotateAroundEffect(
  tau,
  EffectController(duration: 3.0, infinite: true),
  center: Vector2(200, 140), // centre de l'orbite, dans le repère du parent
));
```

> **Attention à l'ordre des arguments.** `RotateAroundEffect` prend l'angle et le contrôleur en **positionnel**, puis `center` en **nommé obligatoire**. En Dart, tous les arguments positionnels passent avant les nommés : `RotateAroundEffect(tau, center: ..., controller)` ne compile pas.

---

## 33.6 — `OpacityEffect`

`OpacityEffect` anime l'opacité de la peinture du composant. Il exige que la cible fournisse une opacité : c'est le cas de tous les composants portant le mixin `HasPaint` (`RectangleComponent`, `CircleComponent`, `PolygonComponent`, `SpriteComponent`, `SpriteAnimationComponent`) et, depuis Flame 1.38.0, des composants texte.

```dart
OpacityEffect.to(0.2, EffectController(duration: 0.75));  // aller à 20 %
OpacityEffect.by(-0.9, EffectController(duration: 0.75)); // retirer 90 points
OpacityEffect.fadeIn(EffectController(duration: 0.75));   // raccourci de to(1.0)
OpacityEffect.fadeOut(EffectController(duration: 0.75));  // raccourci de to(0.0)
```

Le cas d'usage numéro un est le **clignotement d'invincibilité** (section 33.36).

```dart
heros.add(OpacityEffect.to(
  0.25,
  EffectController(duration: 0.08, alternate: true, repeatCount: 8),
));
```

```text
  opacité
   1.00 |\    /\    /\    /\    /\    /\    /\    /\
        | \  /  \  /  \  /  \  /  \  /  \  /  \  /
   0.25 |  \/    \/    \/    \/    \/    \/    \/
        +----------------------------------------------> temps
        0                                          1.28 s
```

Le second cas d'usage est la **disparition en fondu** d'un objet ramassé ou d'un ennemi vaincu.

```dart
gobelin.add(OpacityEffect.fadeOut(
  EffectController(duration: 0.3),
  onComplete: gobelin.removeFromParent,
));
```

> **Attention.** L'opacité appartient au composant, pas à l'effet. Si un fondu est interrompu, l'opacité reste **là où elle en était**. Après une disparition avortée, remettez `composant.opacity = 1.0;` avant de réutiliser le composant : c'est la source classique du « héros à moitié transparent pour toujours ».

---

## 33.7 — `ColorEffect`

`ColorEffect` applique un **filtre de couleur** sur la peinture du composant. C'est l'outil du flash de dégât.

```dart
ColorEffect(
  Color color,
  EffectController controller, {
  double opacityFrom = 0,
  double opacityTo = 1,
  String? paintId,
  void Function()? onComplete,
  ComponentKey? key,
});
```

`opacityFrom` et `opacityTo` ne désignent pas l'opacité du composant, mais l'**intensité du mélange** entre la couleur d'origine et celle du filtre : `0.0` laisse la couleur d'origine, `1.0` donne la couleur du filtre pure.

```dart
gobelin.add(ColorEffect(
  Palette.blanc,
  EffectController(duration: 0.06, alternate: true),
  opacityFrom: 0.0,
  opacityTo: 1.0,
));
```

```text
  intensité du blanc
   1.0 |     /\
   0.0 |__ /   \ ______
       +------------------> temps
       0   0.06  0.12 s

  Le gobelin devient blanc en 60 ms, puis retrouve sa couleur en 60 ms.
```

Un flash partiel (`opacityTo: 0.55`) convient à un dégât mineur. C'est ici que se joue la **proportionnalité** de la section 33.1 : un coup normal monte à 0,55, un coup critique à 1,0 et dure deux fois plus longtemps. Le joueur comprend sans qu'on lui écrive un chiffre.

**La limitation à connaître par cœur.** La documentation est explicite : *« This effect can't be mixed with other ColorEffects, when more than one is added to the component, only the last one will have effect. »*

```dart
// NE FAITES PAS CELA : seul le dernier sera visible.
gobelin.add(ColorEffect(Palette.blanc, EffectController(duration: 0.1)));
gobelin.add(ColorEffect(Palette.piece, EffectController(duration: 0.4)));
```

Si un ennemi doit être empoisonné (vert permanent) **et** flasher au coup, ne combinez pas deux `ColorEffect` : gardez-en un seul dont vous changez la couleur cible, ou superposez un composant enfant transparent que vous faites clignoter avec `OpacityEffect`.

---

## 33.8 — `SizeEffect`

`SizeEffect` anime la propriété `size` du composant, en pixels.

```dart
SizeEffect.by(Vector2(-15, 30), EffectController(duration: 1));
SizeEffect.to(Vector2(90, 80), EffectController(duration: 1));
```

Contrairement à `ScaleEffect.by`, `SizeEffect.by` est un **ajout** en pixels, pas une multiplication. Le tableau ci-dessous résume la différence entre les deux effets, qui est la question la plus posée par les débutants.

| Critère | `ScaleEffect` | `SizeEffect` |
| --- | --- | --- |
| Propriété animée | `scale` (sans unité) | `size` (en pixels) |
| `.by` signifie | multiplier | ajouter |
| Agit sur les enfants | **oui**, mis à l'échelle aussi | **non**, ils gardent leur taille |
| Épaisseur d'un contour | déformée | conservée |
| Exige de la cible | rien de spécial | l'interface `SizeProvider` |
| Usage typique | pop, squash and stretch | barre de vie, jauge, panneau |

**Règle pratique :** pour un effet de *game feel* (un pop, un rebond), utilisez `ScaleEffect`. Pour une **jauge** dont la longueur porte une information, utilisez `SizeEffect`.

Voici l'exemple canonique : la barre de vie qui se vide. Le `TimerComponent` employé ici est expliqué en section 33.33 ; lisez-le pour l'instant comme « appelle `onTick` toutes les `period` secondes ».

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() =>
    runApp(MaterialApp(home: Scaffold(body: GameWidget(game: DemoJauge()))));

class DemoJauge extends FlameGame {
  static const double largeurMax = 200;

  late final RectangleComponent remplissage;
  int viesRestantes = 5;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent( // cadre sombre
        position: Vector2(40, 60),
        size: Vector2(largeurMax, 18),
        paint: Paint()..color = const Color(0xFF2B2B3A)));
    remplissage = RectangleComponent(
        position: Vector2(40, 60),
        size: Vector2(largeurMax, 18),
        paint: Paint()..color = const Color(0xFFB0413E));
    await world.add(remplissage);

    await add(TimerComponent(period: 1.0, repeat: true, onTick: retirerUneVie));
  }

  void retirerUneVie() {
    if (viesRestantes == 0) return;
    viesRestantes--;
    remplissage.add(SizeEffect.to(
      Vector2(largeurMax * viesRestantes / 5, 18),
      EffectController(duration: 0.25, curve: Curves.easeOut),
    ));
  }
}
```

**Résultat :**

```text
  t = 0 s [########################] 5/5    t = 1 s [###################  ] 4/5
  t = 3 s [###########             ] 2/5    t = 5 s [                     ] 0/5
```

La barre ne saute pas d'un cran : elle **glisse** en 250 ms. Le joueur voit le mouvement, donc il remarque qu'il a perdu une vie.

> **Remarque.** L'ancre du remplissage reste `Anchor.topLeft` : la barre se vide **par la droite**, bord gauche fixe. Avec `Anchor.center` elle rétrécirait des deux côtés à la fois.

---

## 33.9 — `AnchorEffect`

Le dernier effet géométrique anime l'`anchor`, c'est-à-dire le point de référence du composant.

```dart
AnchorToEffect(Anchor destination, EffectController controller);
AnchorByEffect(Vector2 offset, EffectController controller);
```

`AnchorToEffect` prend un `Anchor` (`Anchor.center`, `Anchor.bottomLeft`…) ; `AnchorByEffect` prend un décalage en coordonnées relatives, où `(0, 0)` est le coin supérieur gauche et `(1, 1)` le coin inférieur droit.

```text
  (0,0) topLeft ────── (0.5,0) topCenter ────── (1,0) topRight
     │                                              │
  (0,0.5) centerLeft   (0.5,0.5) center     (1,0.5) centerRight
     │                                              │
  (0,1) bottomLeft ── (0.5,1) bottomCenter ── (1,1) bottomRight
```

À quoi cela sert-il ? À changer le **pivot** d'une animation en cours de route. L'exemple le plus parlant est la porte du donjon, qui doit pivoter autour de son gond et non autour de son centre.

```dart
porte.add(SequenceEffect([
  AnchorToEffect(Anchor.topCenter, EffectController(duration: 0.2)),
  RotateEffect.by(
      tau / 5, EffectController(duration: 0.6, curve: Curves.easeOut)),
]));
```

Dans la pratique, `AnchorEffect` est l'effet le moins utilisé de la liste. Retenez surtout qu'il permet de **déplacer le pivot d'une rotation à venir**.

> **Piège.** Modifier l'ancre déplace visuellement le composant sans changer sa `position`. La logique de votre jeu qui lit `composant.position` lira toujours le même chiffre alors que le composant a bougé à l'écran : ne mélangez pas `AnchorEffect` et logique de collision fine.

---

## 33.10 — `EffectController` : le contrôleur de temps

Vous avez maintenant huit types d'effets. Tous partagent le **même** deuxième argument : un `EffectController`, qui décide **quand** et **à quelle vitesse** la progression passe de 0 à 1.

Séparer le « quoi » du « quand » est un excellent choix de conception : n'importe quel contrôleur se branche sur n'importe quel effet. Savoir régler un contrôleur, c'est savoir régler les huit effets.

```dart
EffectController({
  required double duration,
  Curve curve = Curves.linear,
  double? reverseDuration,
  Curve? reverseCurve,
  bool alternate = false,
  double atMaxDuration = 0.0,
  double atMinDuration = 0.0,
  int? repeatCount,
  bool infinite = false,
  double startDelay = 0.0,
  VoidCallback? onMax,
  VoidCallback? onMin,
});
```

Onze paramètres, un seul obligatoire : `EffectController(duration: 1.0)` produit une montée linéaire de 0 à 1 en une seconde, puis l'effet se termine. Le contrôleur expose `started`, `completed` et `progress` en lecture.

| Terme | Définition |
| --- | --- |
| **progression** | valeur entre 0 et 1 produite par le contrôleur à chaque frame |
| **phase aller** | la partie qui va de 0 à 1, de durée `duration` |
| **phase retour** | la partie qui revient de 1 à 0, de durée `reverseDuration` |
| **cycle** | un aller, plus éventuellement un retour et les pauses |
| **répétition** | la relance d'un cycle complet |

> **À retenir.** `EffectController(...)` est une **fabrique** : selon les paramètres donnés, elle assemble une chaîne de contrôleurs internes (`LinearEffectController`, `ReverseCurvedEffectController`, `PauseEffectController`, `RepeatedEffectController`, `DelayedEffectController`…). Vous ne les manipulez jamais directement, mais leur nom apparaît parfois dans les messages d'erreur.

---

## 33.11 — `duration`, `reverseDuration`, `repeatCount`, `infinite`

Passons les paramètres en revue. Tous les exemples s'appliquent à un `MoveEffect.by(Vector2(100, 0), ...)` sur un carré placé en `x = 80`.

**`duration` — la durée de l'aller.** Le carré part de 80, arrive à 180, et **y reste** : un effet ne rembobine pas tout seul.

**`reverseDuration` — la durée du retour.** La durée totale devient `duration + reverseDuration`.

```dart
EffectController(duration: 1.0, reverseDuration: 0.4)
```

```text
  x  180 |                    ,--.
         |                ,--'    \
      80 |,-------,--'              '---
         +------------------------------> temps
         0                     1.0    1.4 s
```

C'est ce paramètre qui donne un **retour asymétrique**, très utile : un pop d'objet doit grossir vite (0,08 s) et redescendre lentement (0,22 s). L'inverse donnerait une impression de mollesse.

```dart
EffectController(
  duration: 0.08,
  reverseDuration: 0.22,
  curve: Curves.easeOut,
  reverseCurve: Curves.easeIn,
)
```

**`repeatCount` — le nombre de cycles.**

```dart
EffectController(duration: 0.3, reverseDuration: 0.3, repeatCount: 3)
```

```text
  x  180 |    ,-.        ,-.        ,-.
         |   /   \      /   \      /   \
      80 |,-'     '----'     '----'     '---
         +------------------------------------> temps
         0    0.6       1.2       1.8 s
```

Sans `reverseDuration`, chaque nouveau cycle repart brutalement de la position initiale : **`repeatCount` sans retour donne un effet qui « claque »**. Parfois c'est voulu ; le plus souvent, non.

**`infinite` — la répétition sans fin.** L'effet ne se termine jamais, donc `onComplete` n'est jamais appelé et `removeOnFinish` n'agit jamais. Si vous retirez le composant cible, l'effet part avec lui : pas de fuite.

```text
  RÈGLE DE DÉCISION
  L'effet répond à une ACTION du joueur ?
      -> durée courte, pas d'infinite. Il doit finir.
  L'effet fait vivre le DÉCOR ?
      -> infinite: true, alternate: true. Il ne doit jamais finir.
```

**`atMaxDuration` et `atMinDuration` — les pauses aux extrémités.**

```dart
EffectController(
  duration: 0.3,
  atMaxDuration: 0.5,  // reste 0,5 s à 100 %
  reverseDuration: 0.3,
  atMinDuration: 1.0,  // reste 1 s à 0 %
  infinite: true,
)
```

```text
  progression
    1.0 |      ,--------.                      ,--------.
        |     /          \                    /
    0.0 |,--'              '----------------'
        +---------------------------------------------------> temps
         0.3    0.5     0.3        1.0        0.3    0.5

  Un cycle complet dure 0.3 + 0.5 + 0.3 + 1.0 = 2.1 s.
```

C'est ainsi qu'on écrit un clignotement d'alerte qui laisse le temps de lire : allumage rapide, maintien, extinction rapide, longue pause.

---

## 33.12 — `alternate`

`alternate: true` est un raccourci dont la définition officielle tient en une phrase : *« setting this to true is equivalent to specifying the reverseDuration equal to the duration »*. Les deux lignes suivantes sont donc strictement identiques :

```dart
EffectController(duration: 0.4, alternate: true);
EffectController(duration: 0.4, reverseDuration: 0.4);
```

Pourquoi un raccourci ? Parce que c'est de loin le cas le plus fréquent : un flash, une pulsation, un clignotement, un tremblement vont et reviennent symétriquement.

**Le piège du calcul de durée :** `alternate: true` **double** la durée totale.

```dart
// « Je veux un flash de 100 ms. »
EffectController(duration: 0.1, alternate: true);  // NON : cela dure 200 ms
EffectController(duration: 0.05, alternate: true); // OUI : 0,05 + 0,05 = 0,1 s
```

Retenez la formule générale, elle sert à régler tous vos effets :

```text
  DURÉE TOTALE D'UN EFFET

  cycle = duration
        + atMaxDuration
        + reverseDuration   (= duration si alternate: true, 0 sinon)
        + atMinDuration

  total = startDelay + cycle x (repeatCount ?? 1)   si infinite == false
        = infini                                     si infinite == true

  ATTENTION : startDelay n'est appliqué qu'UNE FOIS, avant le premier cycle.
```

Vérifions sur un exemple, en calculant à la main avant de lancer le programme :

```dart
EffectController(startDelay: 0.5, duration: 0.2, alternate: true, repeatCount: 3)
```

```text
  cycle = 0.2 (aller) + 0.2 (retour) = 0.4 s
  total = 0.5 + 3 x 0.4 = 1.7 s

  progression
    1.0 |              /\      /\      /\
    0.0 |-------------/  \____/  \____/  \___
        +-----------------------------------------> temps
        0          0.5                    1.7 s
```

`alternate` s'applique aussi à `SequenceEffect`, avec un sens légèrement différent : la séquence entière est rejouée à l'envers (section 33.16).

---

## 33.13 — Les courbes (`Curves.easeIn`, `easeOut`, `elasticOut`, `bounceOut`)

La progression brute d'un contrôleur est linéaire : elle avance de la même quantité à chaque milliseconde. Or **rien dans la nature ne bouge de façon linéaire**. Une balle accélère en tombant. Un bras ralentit avant de s'arrêter. Une porte qui claque rebondit.

Une **courbe** transforme la progression linéaire en une progression déformée : avec `Curves.easeOut`, une progression brute de 0,5 devient 0,79. Le paramètre s'appelle `curve` et attend une valeur de la classe `Curves` de Flutter, disponible dès que l'on importe `package:flutter/material.dart`.

```dart
EffectController(duration: 0.4, curve: Curves.easeOut);
```

Voici les cinq courbes que vous utiliserez dans 95 % des cas, superposées sur un même graphique.

```text
  sortie
   1.3 |                       elasticOut : ,-.
       |                                   / | '-.
   1.0 |...........................,-''----'--+----'---
       |            easeOut  ,-''      ,'     |
       |                  ,-'        ,'    bounceOut : rebondit
       |   linear      ,-'         ,'       sur la valeur 1.0
   0.5 |- - - - - -  ,'          ,'
       |          ,-'          ,'    easeIn : démarre à plat,
       |       ,-'         _,-'      puis s'envole
   0.0 |____,-'______,,--''
       +--------------------------------------------> entrée
       0                                          1.0
```

| Courbe | Allure | Sensation | Usage |
| --- | --- | --- | --- |
| `Curves.linear` | droite | mécanique, robotique | rotation continue, défilement de fond |
| `Curves.easeIn` | lent puis rapide | quelque chose s'échappe, tombe | disparition, chute, fondu au noir |
| `Curves.easeOut` | rapide puis lent | quelque chose arrive et se pose | apparition, recul d'ennemi, ouverture de menu |
| `Curves.easeInOut` | lent aux deux bouts | va-et-vient naturel | plateforme mobile, lévitation |
| `Curves.elasticOut` | dépasse puis oscille | ressort, énergie, joie | pop d'objet ramassé, apparition de bouton |
| `Curves.bounceOut` | rebondit à l'arrivée | masse lourde qui tombe | atterrissage, coffre qui chute |

Testez-les immédiatement : reprenez le banc d'essai de la section 33.3 et bouclez sur cette liste, un carré par courbe.

```dart
const courbes = <(String, Curve)>[
  ('linear', Curves.linear),
  ('easeIn', Curves.easeIn),
  ('easeOut', Curves.easeOut),
  ('elasticOut', Curves.elasticOut),
  ('bounceOut', Curves.bounceOut),
];

for (var i = 0; i < courbes.length; i++) {
  final (nom, courbe) = courbes[i];
  final carre = RectangleComponent(
      position: Vector2(100, 40.0 + i * 44),
      size: Vector2(24, 24),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFF4CAF50));
  await world.add(carre);
  await world.add(TextComponent(
      text: nom,
      position: Vector2(8, 30.0 + i * 44),
      textRenderer: TextPaint(
          style: const TextStyle(fontSize: 13, color: Color(0xFFAAAAAA)))));
  carre.add(MoveEffect.by(
    Vector2(200, 0),
    EffectController(
        duration: 1.2,
        curve: courbe,
        alternate: true,
        infinite: true,
        atMinDuration: 0.4),
  ));
}
```

> **Attention.** `elasticOut` et `bounceOut` **sortent de l'intervalle [0, 1]**. Avec `MoveEffect.by(Vector2(200, 0), ...)` et `elasticOut`, le carré ira jusqu'à environ x = 320 avant de revenir à 300. Ce dépassement donne l'impression de vie ; mais sur une opacité, tout ce qui dépasse 1 est écrêté et l'effet est perdu : **n'utilisez jamais `elasticOut` sur `OpacityEffect`**.

Enfin, `reverseCurve` permet de choisir une courbe différente pour la phase retour :

```dart
EffectController(
  duration: 0.08,
  reverseDuration: 0.25,
  curve: Curves.easeOut,       // montée nette
  reverseCurve: Curves.easeIn, // descente qui s'accélère
)
```

---

## 33.14 — Choisir la bonne courbe (tableau)

Voici le tableau de décision. Accrochez-le au mur : il vous évitera des heures de tâtonnement.

| Ce que vous voulez exprimer | Courbe | Durée conseillée | Exemple dans le « Donjon de Dart » |
| --- | --- | --- | --- |
| Une machine, un rythme régulier | `Curves.linear` | selon le besoin | rotation d'un piège à lames, défilement du fond |
| Quelque chose qui arrive et se pose | `Curves.easeOut` | 0,15 – 0,35 s | le recul du gobelin frappé, l'ouverture du menu de pause |
| Quelque chose qui part et s'échappe | `Curves.easeIn` | 0,20 – 0,40 s | la potion aspirée par le héros, le fondu au noir |
| Un aller-retour naturel | `Curves.easeInOut` | 0,30 – 0,80 s | la plateforme mobile, la lévitation d'un cristal |
| De l'énergie, de la joie | `Curves.elasticOut` | 0,30 – 0,50 s | le pop de la pièce ramassée, l'apparition du score |
| Un impact lourd | `Curves.bounceOut` | 0,40 – 0,70 s | le coffre qui tombe, l'atterrissage du boss |
| Une anticipation avant le coup | `Curves.easeInBack` | 0,20 – 0,30 s | l'arme qui recule avant de frapper |
| Un dépassement léger, propre | `Curves.easeOutBack` | 0,20 – 0,35 s | le bouton du menu qui se met en place |
| Une secousse, une vibration | `Curves.linear` + `alternate` | 0,04 – 0,08 s par cycle | le tremblement d'écran |

Trois règles de bon sens complètent le tableau.

**Règle 1 — En cas de doute, `easeOut`.** Elle convient à quatre-vingts pour cent des situations et n'est jamais choquante.

**Règle 2 — Une courbe fantaisiste par écran, pas plus.** Si la pièce fait `elasticOut`, le coffre `bounceOut` et le menu `elasticOut`, l'ensemble devient une gelée. Réservez les courbes spectaculaires aux moments importants.

**Règle 3 — Ne mettez jamais de courbe sur un effet inférieur à 60 ms.** À cette durée, le joueur voit trois ou quatre images : la forme de la courbe n'est plus perceptible, seule l'amplitude compte.

---

## 33.15 — `startDelay`

`startDelay` insère une attente avant le démarrage de l'effet. C'est, selon la documentation officielle, *« la méthode la plus simple pour enchaîner des effets »*.

```text
  progression   1.0 |                        ,---
                0.0 |--------------------,--'
                    +---------------------------> temps
                    0        0.5        0.8 s
                    |<-- rien ne bouge ->|
```

**Usage 1 — décaler des effets identiques pour créer une vague.** L'astuce la plus rentable du chapitre : trois torches qui vacillent en phase ont l'air fausses ; décalées de 0,2 s, elles ont l'air vivantes.

```dart
for (var i = 0; i < 5; i++) {
  final torche = CircleComponent(
      radius: 8,
      position: Vector2(60.0 + i * 60, 60),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFFFFB300));
  await world.add(torche);
  torche.add(ScaleEffect.by(
    Vector2.all(1.35),
    EffectController(
        duration: 0.6,
        alternate: true,
        infinite: true,
        startDelay: i * 0.18, // décalage progressif
        curve: Curves.easeInOut),
  ));
}
```

```text
  torche 0   O---.---O---.---O
  torche 1     O---.---O---.---O
  torche 2       O---.---O---.---O
  torche 3         O---.---O---.---O
             |----|
             0.18 s de décalage
```

**Usage 2 — chaîner deux effets sans `SequenceEffect`.** Le gobelin flashe, puis 0,1 s plus tard il est repoussé.

```dart
gobelin.add(ColorEffect(
    Palette.blanc, EffectController(duration: 0.05, alternate: true)));
gobelin.add(MoveEffect.by(Vector2(20, 0),
    EffectController(duration: 0.18, startDelay: 0.10, curve: Curves.easeOut)));
```

```text
  QUAND UTILISER QUOI
  startDelay      -> effets INDÉPENDANTS qui doivent se croiser
                     (décalage de phase, superposition)
  SequenceEffect  -> effets DÉPENDANTS qui s'enchaînent
                     (l'un commence exactement quand l'autre finit)
```

L'inconvénient du `startDelay` est que les durées sont écrites en dur : modifier la durée du premier effet oblige à recalculer le délai du second.

> **Piège.** `startDelay` s'applique **une seule fois**, avant le premier cycle. Avec `repeatCount: 3` et `startDelay: 0.5`, vous n'aurez pas trois pauses de 0,5 s mais une seule au début. Pour une pause entre chaque cycle, utilisez `atMinDuration`.

---

## 33.16 — `SequenceEffect` : enchaîner

`SequenceEffect` exécute une liste d'effets **l'un après l'autre**. Sa particularité : il ne prend pas d'`EffectController`, car sa durée est la somme des durées de ses enfants.

```dart
SequenceEffect(
  List<Effect> effects, {
  bool alternate = false,
  bool infinite = false,
  int repeatCount = 1,
  void Function()? onComplete,
  ComponentKey? key,
});
```

Une séquence classique du « Donjon de Dart » : le coffre qui s'ouvre en trois temps.

```dart
coffre.add(SequenceEffect([
  // 1. Anticipation : le coffre s'écrase légèrement.
  ScaleEffect.to(Vector2(1.15, 0.85),
      EffectController(duration: 0.10, curve: Curves.easeOut)),
  // 2. Détente : il se redresse en dépassant.
  ScaleEffect.to(Vector2(0.92, 1.18),
      EffectController(duration: 0.12, curve: Curves.easeOut)),
  // 3. Repos : retour à la normale, avec du ressort.
  ScaleEffect.to(Vector2.all(1.0),
      EffectController(duration: 0.35, curve: Curves.elasticOut)),
]));
```

```text
  échelle X
   1.2 |    ,-.
   1.0 |,--'   \        ,-.,-.,----
   0.9 |        '------'
       +-------------------------------> temps
       0   0.1  0.22          0.57 s

  Anticipation, détente, retour : la structure de base de toute
  animation vivante, quel que soit le moteur.
```

### `alternate`, `repeatCount` et `infinite` sur une séquence

```dart
SequenceEffect(
  [
    MoveEffect.by(Vector2(60, 0), EffectController(duration: 0.5)),
    MoveEffect.by(Vector2(0, 60), EffectController(duration: 0.5)),
  ],
  alternate: true,
  infinite: true,
)
```

```text
  Sans alternate : droite, bas, droite, bas...  (le composant dérive)
  Avec alternate : droite, bas, HAUT, GAUCHE... (il revient au départ)
```

`alternate: true` rejoue la liste **dans l'ordre inverse et à l'envers**. C'est exactement ce qu'il faut pour une patrouille. Notez que `repeatCount` vaut ici `1` par défaut, alors que celui d'`EffectController` vaut `null`.

### Séquences imbriquées et réutilisation

Une `SequenceEffect` est un `Effect` : elle peut en contenir d'autres. Mais **un effet ne peut être ajouté qu'à un seul parent**. Pour appliquer le même enchaînement à plusieurs composants, écrivez une **fonction** qui construit une instance neuve à chaque appel.

```dart
SequenceEffect clignotement() => SequenceEffect(
      [
        OpacityEffect.to(0.2, EffectController(duration: 0.08)),
        OpacityEffect.to(1.0, EffectController(duration: 0.08)),
      ],
      repeatCount: 3,
    );

for (final g in gobelins) {
  g.add(clignotement()); // une instance neuve par gobelin
}
```

---

## 33.17 — Les effets en parallèle

Il n'existe pas de `ParallelEffect` dans l'API publique de Flame 1.38.0, et il n'en faut pas : **ajouter deux effets au même composant les fait déjà jouer en parallèle**.

```dart
piece.add(MoveEffect.by(
    Vector2(0, -40), EffectController(duration: 0.5, curve: Curves.easeOut)));
piece.add(ScaleEffect.by(
    Vector2.all(1.4), EffectController(duration: 0.5, alternate: true)));
piece.add(OpacityEffect.fadeOut(
    EffectController(duration: 0.5, startDelay: 0.2)));
```

```text
  t = 0.0 s   la pièce monte, grossit et reste opaque
  t = 0.2 s   elle commence à s'effacer tout en continuant de monter
  t = 0.25 s  elle atteint sa taille maximale et commence à rétrécir
  t = 0.5 s   elle est en haut, à sa taille normale, invisible
```

Le résultat est bien plus riche que la somme de ses parties, pour trois lignes de code : **c'est le geste le plus rentable du chapitre.**

Le mécanisme fonctionne parce que les effets touchent des propriétés **différentes**. Deux effets `.by` sur la même propriété s'additionnent proprement, puisqu'ils appliquent des incréments relatifs : c'est ce qui permet de superposer une patrouille et un tremblement. Deux effets `.to` se **battent** : chacun calcule son déplacement depuis la valeur lue à son démarrage, et le résultat est imprévisible.

| Combinaison | Résultat |
| --- | --- |
| `MoveEffect.by` + `ScaleEffect.by` | parfait, propriétés différentes |
| `MoveEffect.by` + `MoveEffect.by` | s'additionnent proprement |
| `MoveEffect.to` + `MoveEffect.by` | fonctionne, mais difficile à prévoir |
| `MoveEffect.to` + `MoveEffect.to` | conflit, résultat imprévisible |
| `OpacityEffect.to` + `OpacityEffect.to` | conflit, même problème |
| `ColorEffect` + `ColorEffect` | seul le dernier est visible (limitation documentée) |

Depuis Flame 1.34.0, `CombinedEffect` regroupe explicitement plusieurs effets simultanés dans un seul composant : utile pour retirer, mettre en pause ou réinitialiser le groupe d'un seul geste. Pour un usage courant, ajouter les effets un par un suffit.

---

## 33.18 — `onComplete` et le retrait automatique (`removeOnFinish`)

### `removeOnFinish`

Tout effet possède un champ `removeOnFinish`, dont la valeur par défaut est **`true`**. Dans le cas normal, vous n'avez donc rien à faire : l'effet disparaît tout seul.

Le passer à `false` permet de **rejouer** l'effet sans le reconstruire, grâce à `reset()` :

```dart
class Gobelin extends RectangleComponent {
  late final ColorEffect _flash;

  @override
  Future<void> onLoad() async {
    _flash = ColorEffect(
      const Color(0xFFFFFFFF),
      EffectController(duration: 0.05, alternate: true),
    )..removeOnFinish = false;
    add(_flash);
  }

  void subirDegats() => _flash.reset(); // relance depuis le début
}
```

C'est une micro-optimisation : utile quand un même effet est déclenché des centaines de fois par minute, inutile pour un gobelin frappé toutes les trois secondes.

### `onComplete`

Le callback est *« invoked when the effect has just completed its execution but before it is removed »* : le composant cible est donc **encore dans l'arbre**.

```text
  ... apply(1.0) -> onComplete() -> removeFromParent()
```

```dart
gobelin.add(OpacityEffect.fadeOut(
  EffectController(duration: 0.35),
  onComplete: () {
    gobelin.removeFromParent();
    game.score += 10;
  },
));
```

> **Piège majeur.** `onComplete` n'est **jamais** appelé si le contrôleur est `infinite: true`. Ne mettez jamais la logique de fin d'une animation dans le `onComplete` d'un effet infini.

Pour le cas très courant « fais ceci puis disparais », Flame fournit `RemoveEffect(delay: 3.0)`, qui retire le composant après le délai indiqué :

```dart
texteDegats.add(SequenceEffect([
  MoveEffect.by(Vector2(0, -30),
      EffectController(duration: 0.6, curve: Curves.easeOut)),
  OpacityEffect.fadeOut(EffectController(duration: 0.2)),
  RemoveEffect(),
]));
```

### L'erreur du siècle : ajouter un effet à chaque frame

```dart
// NE FAITES JAMAIS CELA.
@override
void update(double dt) {
  super.update(dt);
  if (isColliding) {
    add(ColorEffect(Palette.blanc,
        EffectController(duration: 0.1, alternate: true)));
  }
}
```

Tant que la collision dure, `update` est appelé soixante fois par seconde : **soixante effets créés par seconde**. Au bout de trois secondes, cent quatre-vingts `ColorEffect` cohabitent (`[A]`, puis `[A][B]`, puis `[A][B][C]`…), le framerate s'effondre et l'effet visuel devient un blanc constant au lieu d'un flash.

Trois corrections, selon le contexte :

```dart
// 1. Déclencher sur l'ÉVÉNEMENT (appelé une seule fois par contact).
@override
void onCollisionStart(Set<Vector2> points, PositionComponent other) {
  super.onCollisionStart(points, other);
  add(ColorEffect(Palette.blanc,
      EffectController(duration: 0.05, alternate: true)));
}

// 2. Vérifier qu'aucun effet du même type n'est en cours.
void flasher() {
  if (children.whereType<ColorEffect>().isNotEmpty) return;
  add(ColorEffect(Palette.blanc,
      EffectController(duration: 0.05, alternate: true)));
}

// 3. Garder une référence unique et la réinitialiser (removeOnFinish = false).
```

> **À retenir.** Un effet se déclenche sur une **transition** (« il vient d'être touché »), jamais sur un **état** (« il est en contact »). Si vous écrivez `add(...Effect(...))` dans `update`, arrêtez-vous et cherchez l'événement correspondant.

---

## 33.19 — Le flash blanc quand un ennemi est touché

Assemblons tout ce qui précède dans le premier retour de jeu complet : le gobelin frappé.

```text
  QUAND le gobelin est touché :
    1. il devient blanc pendant 100 ms                     -> ColorEffect
    2. il est repoussé de 14 px dans la direction du coup  -> MoveEffect.by
    3. il s'écrase légèrement puis se redresse             -> ScaleEffect
    4. si ses vies tombent à zéro, il gonfle et implose    -> SequenceEffect + onComplete
```

Programme complet et exécutable, sans aucune image. Touchez l'écran pour frapper.

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: JeuFlash()))));

class JeuFlash extends FlameGame with TapCallbacks {
  late final Gobelin gobelin;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent(
      position: Vector2(20, 40),
      size: Vector2(360, 200),
      paint: Paint()..color = const Color(0xFF2B2B3A),
    ));

    gobelin = Gobelin(position: Vector2(200, 140));
    await world.add(gobelin);
  }

  @override
  void onTapDown(TapDownEvent event) {
    if (gobelin.isMounted && gobelin.vies > 0) {
      // Le coup vient de la gauche : le gobelin part vers la droite.
      gobelin.subirDegats(Vector2(1, 0));
    }
  }
}

class Gobelin extends RectangleComponent {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          size: Vector2(40, 52),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFB0413E));

  int vies = 3;

  void subirDegats(Vector2 direction) {
    vies--;

    // 1. Flash blanc. 2. Recul. 3. Écrasement. Les trois en parallèle.
    add(ColorEffect(const Color(0xFFFFFFFF),
        EffectController(duration: 0.05, alternate: true),
        opacityTo: 1.0));
    add(MoveEffect.by(direction.normalized() * 14,
        EffectController(duration: 0.06, alternate: true, curve: Curves.easeOut)));
    add(ScaleEffect.to(Vector2(1.25, 0.8),
        EffectController(duration: 0.07, alternate: true, curve: Curves.easeOut)));

    if (vies <= 0) mourir();
  }

  void mourir() {
    add(SequenceEffect(
      [
        ScaleEffect.to(Vector2.all(1.4),
            EffectController(duration: 0.08, curve: Curves.easeOut)),
        ScaleEffect.to(Vector2.zero(),
            EffectController(duration: 0.22, curve: Curves.easeIn)),
      ],
      onComplete: removeFromParent,
    ));
  }
}
```

**Résultat :**

```text
  Tap 1 et 2 : le gobelin blanchit, sursaute vers la droite, s'écrase.
  Tap 3      : idem, puis il gonfle brusquement et implose ; il est
               retiré de l'arbre. Tap 4+ : plus rien.
```

Trois points de conception méritent d'être soulignés. **Le recul utilise `alternate: true`** : le gobelin part de 14 px et **revient** à sa place ; sans le retour, il dériverait à chaque coup et finirait hors de l'écran. **`direction.normalized()` garantit une amplitude constante**, quelle que soit la longueur du vecteur passé (chapitre 23). **`onComplete: removeFromParent` passe la référence de la méthode**, sans parenthèses : c'est la syntaxe de tear-off du chapitre 7.

> **Ordre des effets.** Le flash, le recul et l'écrasement sont ajoutés dans la même frame et jouent **en parallèle** (section 33.17). Ils touchent trois propriétés différentes — couleur, position, échelle — donc aucun conflit n'est possible.

---

## 33.20 — Le pop d'un objet ramassé

Le retour de jeu symétrique du précédent : quelque chose de **positif** vient d'arriver. La grammaire d'un ramassage réussi est bien établie, et ses quatre parties sont **simultanées ou décalées de peu**, jamais séquentielles ; l'ensemble tient en 500 ms.

```text
  1. L'objet grossit brusquement -> « je t'ai vu »
  2. Il monte                    -> « je m'en vais »
  3. Il s'efface                 -> « je ne suis plus là »
  4. Un chiffre monte à sa place -> « voilà ce que tu as gagné »
```

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: JeuPop()))));

class JeuPop extends FlameGame with TapCallbacks {
  int score = 0;
  late final TextComponent affichageScore;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    affichageScore = TextComponent(
      text: 'Score : 0',
      position: Vector2(16, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 18, color: Color(0xFFE8B04B)),
      ),
    );
    await world.add(affichageScore);

    for (var i = 0; i < 5; i++) {
      await world.add(Piece(position: Vector2(70.0 + i * 62, 150)));
    }
  }

  @override
  void onTapDown(TapDownEvent event) {
    final p = event.localPosition;
    for (final piece in world.children.whereType<Piece>()) {
      if (!piece.ramassee && (piece.position - p).length < 40) {
        piece.ramasser();
        break;
      }
    }
  }

  /// Affiche un « +N » qui monte et s'efface, puis met le score à jour.
  Future<void> afficherGain(Vector2 position, int valeur) async {
    final texte = TextComponent(
      text: '+$valeur',
      position: position.clone(),
      anchor: Anchor.center,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 20, color: Color(0xFFFFF176)),
      ),
    );
    await world.add(texte);
    texte.add(MoveEffect.by(Vector2(0, -46),
        EffectController(duration: 0.65, curve: Curves.easeOut)));
    texte.add(OpacityEffect.fadeOut(
      EffectController(duration: 0.30, startDelay: 0.35),
      onComplete: texte.removeFromParent,
    ));

    score += valeur;
    affichageScore.text = 'Score : $score';
    affichageScore.add(ScaleEffect.by(Vector2.all(1.25),
        EffectController(duration: 0.10, alternate: true, curve: Curves.easeOut)));
  }
}

class Piece extends CircleComponent with HasGameReference<JeuPop> {
  Piece({required Vector2 position})
      : super(
          radius: 12,
          position: position,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B));

  bool ramassee = false;

  @override
  Future<void> onLoad() async {
    // Lévitation permanente : la pièce attire l'oeil avant d'être ramassée.
    add(MoveEffect.by(
      Vector2(0, -6),
      EffectController(
          duration: 0.8, alternate: true, infinite: true, curve: Curves.easeInOut),
    ));
  }

  void ramasser() {
    if (ramassee) return;
    ramassee = true;

    game.afficherGain(position.clone(), 10);

    add(ScaleEffect.to(Vector2.all(1.9),
        EffectController(duration: 0.14, curve: Curves.easeOut)));
    add(MoveEffect.by(Vector2(0, -26),
        EffectController(duration: 0.30, curve: Curves.easeOut)));
    add(OpacityEffect.fadeOut(
      EffectController(duration: 0.22, startDelay: 0.08),
      onComplete: removeFromParent,
    ));
  }
}
```

**Résultat :**

```text
  Score : 0
     ()   ()   ()   ()   ()      5 pièces qui lévitent doucement
  [tap sur la 3e pièce]
     ()   ()   (O)  ()   ()      t = 0.00 s  elle gonfle
     ()   ()   +10  ()   ()      t = 0.10 s  le « +10 » apparaît et monte
     ()   ()        ()   ()      t = 0.35 s  la pièce a disparu
  Score : 10                     le score a fait un pop
```

Deux points de méthode importants. **Le drapeau `ramassee`** : sans lui, deux taps rapprochés lanceraient deux fois `ramasser()`, donc deux `+10`, car le composant existe encore pendant les 300 ms de son animation de sortie. **Toute animation de disparition doit être protégée par un drapeau.** **`position.clone()`** : `Vector2` est **mutable** ; sans le clone, le `TextComponent` partagerait l'objet de la pièce et le « +10 » la suivrait dans son mouvement de sortie.

---

## 33.21 — Le tremblement d'écran avec un effet sur le viewfinder

Au chapitre 31, vous avez appris que la caméra n'est pas un objet monolithique : le `CameraComponent` contient un `Viewport` (la fenêtre à l'écran) et un `Viewfinder` (le point du monde que l'on regarde).

Le `Viewfinder` implémente `PositionProvider`, `AngleProvider`, `ScaleProvider` et `AnchorProvider`. C'est précisément ce qu'il faut pour **lui appliquer des effets**.

```dart
camera.viewfinder.add(
  MoveEffect.by(Vector2(8, 0), EffectController(duration: 0.05, alternate: true)),
);
```

Un seul aller-retour horizontal donne une secousse plate. Un bon tremblement obéit à trois règles.

```text
  1. Il est COURT.       0,15 à 0,30 s ; au-delà, le joueur a mal au coeur.
  2. Il est IRRÉGULIER.  Amplitudes et directions variables, décroissantes.
  3. Il REVIENT À ZÉRO.  La somme des déplacements doit être nulle, sinon
                         la caméra dérive à chaque secousse.
```

La troisième règle est la plus oubliée. L'implémentation ci-dessous la respecte par construction : on note les décalages, et le dernier segment est calculé pour annuler la somme.

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() =>
    runApp(MaterialApp(home: Scaffold(body: GameWidget(game: JeuSecousse()))));

class JeuSecousse extends FlameGame with TapCallbacks {
  final Random _rnd = Random();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Un damier : sans repères visuels, on ne voit pas la secousse.
    for (var i = 0; i < 8; i++) {
      for (var j = 0; j < 5; j++) {
        await world.add(RectangleComponent(
          position: Vector2(20.0 + i * 46, 30.0 + j * 46),
          size: Vector2(42, 42),
          paint: Paint()
            ..color =
                (i + j).isEven ? const Color(0xFF2B2B3A) : const Color(0xFF23232F),
        ));
      }
    }
  }

  @override
  void onTapDown(TapDownEvent event) => secouer(intensite: 10, duree: 0.25);

  /// [intensite] : amplitude maximale en pixels. [duree] : durée totale.
  void secouer({double intensite = 8, double duree = 0.2, int segments = 6}) {
    // Une secousse à la fois : sinon les amplitudes s'additionnent.
    if (camera.viewfinder.children.whereType<SequenceEffect>().isNotEmpty) return;

    final pas = duree / segments;
    final decalages = <Vector2>[];
    final cumul = Vector2.zero();

    for (var i = 0; i < segments - 1; i++) {
      final force = intensite * (1 - i / segments); // amplitude décroissante
      final d = Vector2(
        (_rnd.nextDouble() * 2 - 1) * force,
        (_rnd.nextDouble() * 2 - 1) * force,
      );
      decalages.add(d);
      cumul.add(d);
    }
    decalages.add(-cumul); // dernier segment : annule exactement le cumul

    camera.viewfinder.add(
      SequenceEffect(
        decalages
            .map((d) => MoveEffect.by(d, EffectController(duration: pas)))
            .toList(),
      ),
    );
  }
}
```

**Résultat :**

```text
  décalage horizontal du viewfinder

   +10 |  ,-.
       | /   \    ,-.
     0 |'     \  /   \  ,-.    ,--------
       |       \/     \/   \  /
   -10 |                    '-'
       +----------------------------------> temps
       0                             0.25 s

  L'amplitude décroît, et la courbe revient exactement à zéro.
```

Un tremblement d'amplitude constante a l'air d'une vibration de téléphone ; décroissant, il a l'air d'un impact. Pour les gros chocs, ajoutez une composante de rotation :

```dart
camera.viewfinder.add(
  RotateEffect.by(
    0.02, // environ 1,1 degré
    EffectController(duration: 0.05, alternate: true, repeatCount: 2),
  ),
);
```

> **Attention.** Si la caméra suit le héros avec `camera.follow(heros)` (chapitre 31), le suivi et l'effet modifient tous deux la position du viewfinder. La secousse reste parfaitement visible, mais l'écran peut ne pas revenir exactement à sa position d'origine ; `follow` la ramène dans les frames suivantes.

> **Accessibilité.** Certains joueurs souffrent de cinétose. Prévoyez dès maintenant un champ `double facteurSecousse = 1.0;` dans votre classe de jeu, multipliez `intensite` par ce facteur, et proposez une option « réduire les secousses » qui le met à zéro.

---

## 33.22 — Les particules : `Particle` et `ParticleSystemComponent`

Un effet anime **un** composant. Une particule sert à en animer **cent** sans créer cent composants. Flame sépare nettement deux notions, et la confusion entre les deux est la source de la plupart des erreurs.

```text
  Particle                        ParticleSystemComponent
  ─────────────────────────       ─────────────────────────────
  Objet léger.                    Composant de l'arbre Flame.
  N'est PAS un Component.         EST un Component.
  Ne connaît pas l'arbre.         A une position, une priority.
  Sait se dessiner et vieillir.   Héberge UNE Particle et la fait vivre.
  A une durée de vie (lifespan).  Se retire seul quand la particule est morte.
```

La `Particle` décrit **quoi dessiner et comment cela évolue** ; le `ParticleSystemComponent` est le **porteur** qui l'insère dans le jeu.

```dart
import 'package:flame/particles.dart';

world.add(ParticleSystemComponent(particle: CircleParticle()));
```

```dart
ParticleSystemComponent({
  Particle? particle,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  Anchor? anchor,
  int? priority,
  ComponentKey? key,
});
```

Toutes les particules partagent trois membres : `lifespan` (durée de vie en secondes), `progress` (avancement de 0.0 à 1.0) et `setLifespan(double)`. Le porteur surveille `shouldRemove` : quand la particule a épuisé sa durée de vie, le composant se retire tout seul. **Vous n'avez rien à nettoyer.**

**Point capital sur le repère.** Une particule se dessine **autour de l'origine (0, 0)** du canvas qu'on lui donne. Comme le porteur a déjà translaté le canvas à sa propre position, la particule apparaît à cette position. Pour placer une gerbe au point de contact d'un coup, on fixe donc la position du **composant**, pas celle de la particule.

---

## 33.23 — `CircleParticle`, `SpriteParticle`, `ComputedParticle`

Trois façons de dessiner une particule, de la plus simple à la plus libre.

### `CircleParticle` — un disque

```dart
CircleParticle({required Paint paint, double radius = 10.0, double? lifespan});
```

```dart
world.add(ParticleSystemComponent(
  position: Vector2(200, 140),
  particle: CircleParticle(
      radius: 6, lifespan: 0.5, paint: Paint()..color = const Color(0xFFE8B04B)),
));
```

C'est la particule à privilégier quand on n'a pas d'image : elle couvre les étincelles, la poussière, le sang, les bulles et les éclats.

### `SpriteParticle` — une image

Elle exige un `Sprite`, donc normalement un fichier. Mais on peut **fabriquer une image à l'exécution** avec le `PictureRecorder` de Flutter : le cours reste ainsi utilisable sans aucun asset.

```dart
import 'dart:ui' as ui;

/// Fabrique une petite image en losange, sans fichier.
Future<ui.Image> creerEclat(Color couleur, int taille) async {
  final recorder = ui.PictureRecorder();
  final c = taille / 2;
  Canvas(recorder).drawPath(
    Path()
      ..moveTo(c, 0)
      ..lineTo(taille.toDouble(), c)
      ..lineTo(c, taille.toDouble())
      ..lineTo(0, c)
      ..close(),
    Paint()..color = couleur,
  );
  return recorder.endRecording().toImage(taille, taille);
}

// Puis, dans onLoad :
final sprite = Sprite(await creerEclat(const Color(0xFFFFD54F), 12));
world.add(ParticleSystemComponent(
  position: Vector2(200, 140),
  particle: SpriteParticle(sprite: sprite, size: Vector2.all(12), lifespan: 0.5),
));
```

### `ComputedParticle` — vous dessinez tout

Vous fournissez une fonction de rendu qui reçoit le canvas et la particule. Comme elle lit `particle.progress`, tout peut varier dans le temps.

```dart
// typedef ParticleRenderDelegate = void Function(Canvas c, Particle particle);

world.add(ParticleSystemComponent(
  position: Vector2(200, 140),
  particle: ComputedParticle(
    lifespan: 0.6,
    renderer: (canvas, particle) {
      final p = particle.progress; // 0.0 -> 1.0
      canvas.drawCircle(
        Offset.zero,
        14 * (1 - p), // le rayon décroît
        Paint()
          ..color = Color.lerp(
            const Color(0xFFFFF176), // jaune au début
            const Color(0xFFB0413E), // rouge à la fin
            p,
          )!,
      );
    },
  ),
));
```

**Résultat :**

```text
  t = 0.0 s  (#)  jaune, 14 px  |  t = 0.3 s  (o)  orange, 7 px
  t = 0.6 s   .   rouge, 0 px, puis disparition
```

| Particule | Coût | Souplesse | Quand la choisir |
| --- | --- | --- | --- |
| `CircleParticle` | très faible | nulle | poussière, étincelles, sang |
| `SpriteParticle` | faible | image fixe | feuilles, flocons, débris dessinés |
| `ComputedParticle` | selon votre code | totale | dégradés, formes qui changent |
| `ComponentParticle` | élevé | totale | à éviter en grande quantité |

---

## 33.24 — `AcceleratedParticle`

Une particule qui ne bouge pas n'est pas convaincante. `AcceleratedParticle` est un **enveloppeur** : il prend une particule enfant et lui applique une physique de position, vitesse et accélération.

```dart
AcceleratedParticle({
  required Particle child,
  Vector2? acceleration,
  Vector2? speed,
  Vector2? position,
  double? lifespan,
});
```

- `position` : décalage de départ par rapport à l'origine du canvas ;
- `speed` : vitesse initiale, en **pixels logiques par seconde** ;
- `acceleration` : accélération constante ; une valeur positive en `y` simule la gravité, l'axe `y` descendant (chapitre 21).

```dart
world.add(ParticleSystemComponent(
  position: Vector2(200, 120),
  particle: AcceleratedParticle(
    lifespan: 1.0,
    speed: Vector2(60, -180),      // part vers la droite et vers le haut
    acceleration: Vector2(0, 420), // gravité
    child:
        CircleParticle(radius: 4, paint: Paint()..color = const Color(0xFFE8B04B)),
  ),
));
```

Le résultat est une parabole : montée puis chute accélérée, c'est-à-dire le mouvement du projectile du chapitre 23 appliqué à une particule.

Remarquez la **composition** : `AcceleratedParticle` ne sait pas dessiner, il sait bouger ; `CircleParticle` sait dessiner mais pas bouger. On les emboîte, comme les widgets Flutter.

```text
  AcceleratedParticle          <- fait bouger
    └── CircleParticle         <- fait dessiner
```

Une écriture raccourcie existe, via les méthodes de composition de `Particle` :

```dart
CircleParticle(radius: 4, paint: peinture)
    .accelerated(acceleration: Vector2(0, 420), speed: Vector2(60, -180));
```

Ces méthodes sont : `translated`, `moving`, `accelerated`, `scaled`, `scaling`, `rotated`, `rotating`.

---

## 33.25 — `MovingParticle`

`MovingParticle` déplace son enfant d'un point à un autre **pendant toute sa durée de vie**, avec une courbe. Si `from` est omis, le trajet part de l'origine du canvas.

```dart
MovingParticle({
  required Particle child,
  required Vector2 to,
  Vector2? from,
  double? lifespan,
  Curve curve = Curves.linear,
});
```

```dart
world.add(ParticleSystemComponent(
  position: Vector2(200, 200),
  particle: MovingParticle(
    lifespan: 0.6,
    from: Vector2.zero(),
    to: Vector2(0, -70),
    curve: Curves.easeOut,
    child:
        CircleParticle(radius: 5, paint: Paint()..color = const Color(0xFF7E57C2)),
  ),
));
```

| Critère | `AcceleratedParticle` | `MovingParticle` |
| --- | --- | --- |
| Point d'arrivée | inconnu à l'avance | **imposé** (`to`) |
| Trajectoire | parabole physique | ligne droite |
| Contrôle du rythme | par l'accélération | par une `curve` |
| Usage typique | explosion, éclats, gravats | objet aspiré, gain qui rejoint le HUD |

`MovingParticle` est donc l'outil du **trajet dirigé** : une pièce qui vole du sol jusqu'au compteur du HUD, une potion aspirée par le héros, une flèche de dégât.

---

## 33.26 — Composer des particules (`Particle.generate`)

Une particule seule ne fait pas une explosion. `Particle.generate` en fabrique un lot et les regroupe dans une unique `ComposedParticle`.

```dart
static Particle generate({
  required ParticleGenerator generator, // Particle Function(int)
  int count = 10,
  double? lifespan,
  bool applyLifespanToChildren = true,
});
```

`generator` reçoit l'index (0, 1, 2, …) et renvoie une particule. C'est là que l'on introduit le **hasard**, indispensable : dix particules identiques n'ont pas l'air d'une explosion, elles ont l'air d'un bug.

```dart
import 'dart:math';

final Random rnd = Random();

Particle gerbe(Color couleur) => Particle.generate(
      count: 14,
      lifespan: 0.7,
      generator: (i) => AcceleratedParticle(
        speed: Vector2(
          rnd.nextDouble() * 240 - 120, // entre -120 et +120
          rnd.nextDouble() * -200 - 40, // entre -240 et -40 (vers le haut)
        ),
        acceleration: Vector2(0, 500),
        child: CircleParticle(
          radius: 2 + rnd.nextDouble() * 2.5,
          paint: Paint()..color = couleur,
        ),
      ),
    );
```

```text
  STRUCTURE PRODUITE PAR Particle.generate

  ComposedParticle              <- une seule particule pour Flame
    ├── AcceleratedParticle → CircleParticle   (index 0)
    ├── ...
    └── AcceleratedParticle → CircleParticle   (index 13)

  Un seul ParticleSystemComponent héberge les 14.
```

C'est le point clé : **un seul composant** dans l'arbre, quel que soit le nombre de particules. C'est ce qui rend le système bon marché.


`applyLifespanToChildren`, à `true` par défaut, impose la durée de vie du lot à tous les enfants. Mettez-le à `false` pour donner à chacun sa propre durée : des durées variées cassent la synchronisation et rendent l'ensemble beaucoup plus naturel.

> **Réutiliser l'index `i`.** Le générateur reçoit l'index, ce qui permet des motifs réguliers : `final angle = i / count * tau;` puis `speed: Vector2(cos(angle), sin(angle)) * 150` produit une **couronne** parfaite plutôt qu'un nuage aléatoire. Idéal pour l'onde de choc d'un boss.

---

## 33.27 — Une explosion de pièces

Premier système complet du fil rouge : le coffre du « Donjon de Dart » qui crache ses pièces. Dix-huit disques dorés partent vers le haut en éventail, retombent sous l'effet de la gravité, avec des tailles et des durées de vie variées ; le coffre lui-même sursaute.

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/particles.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: JeuCoffre()))));

class JeuCoffre extends FlameGame with TapCallbacks {
  final Random rnd = Random();
  late final RectangleComponent coffre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent(
      position: Vector2(0, 220),
      size: Vector2(400, 60),
      paint: Paint()..color = const Color(0xFF2B2B3A),
    ));

    coffre = RectangleComponent(
      position: Vector2(200, 210),
      size: Vector2(54, 40),
      anchor: Anchor.bottomCenter,
      paint: Paint()..color = const Color(0xFF8D6E63),
    );
    await world.add(coffre);
  }

  @override
  void onTapDown(TapDownEvent event) => ouvrirCoffre();

  void ouvrirCoffre() {
    coffre.add(
      SequenceEffect([
        ScaleEffect.to(Vector2(1.2, 0.75),
            EffectController(duration: 0.07, curve: Curves.easeOut)),
        ScaleEffect.to(Vector2.all(1.0),
            EffectController(duration: 0.32, curve: Curves.elasticOut)),
      ]),
    );

    world.add(ParticleSystemComponent(
      position: Vector2(200, 178), // le haut du coffre
      particle: Particle.generate(
        count: 18,
        applyLifespanToChildren: false,
        generator: (i) => AcceleratedParticle(
          lifespan: 0.7 + rnd.nextDouble() * 0.5,
          speed: Vector2(rnd.nextDouble() * 260 - 130,
              -140 - rnd.nextDouble() * 180), // toujours vers le haut
          acceleration: Vector2(0, 620),
          child: CircleParticle(
            radius: 2.5 + rnd.nextDouble() * 2.0,
            paint: Paint()
              ..color =
                  i.isEven ? const Color(0xFFE8B04B) : const Color(0xFFFFD54F),
          ),
        ),
      ),
    ));
  }
}
```

**Résultat :**

```text
  t = 0.00 s    ·  ·   ·  ·      les pièces jaillissent
              ·   ·  ·   ·  ·
                 [COFFRE]
  t = 0.60 s ·             ·     elles retombent, certaines sont mortes
                 [COFFRE]
  t = 1.20 s     [COFFRE]        le ParticleSystemComponent s'est retiré
```

Trois réglages à retenir, transposables à toutes vos gerbes. **La vitesse verticale est toujours négative** : `-140 - rnd.nextDouble() * 180` donne une plage de `-320` à `-140`, donc tout part vers le haut ; un `speed.y` positif ferait plonger les pièces dans le sol. **L'accélération dépasse la vitesse initiale** : avec 620 de gravité et 230 de vitesse verticale moyenne, le sommet est atteint en 0,37 s, et les pièces retombent bien dans leur durée de vie. **Deux teintes alternées** : `i.isEven` produit deux couleurs sans coût, et un lot monochrome a toujours l'air plat.

---

## 33.28 — Un nuage de poussière au saut

L'explosion projette **vers le haut**. La poussière, elle, s'étale **au sol** et **s'efface**. Deux différences de réglage suffisent.

```dart
Particle poussiere() => Particle.generate(
      count: 10,
      lifespan: 0.45,
      generator: (i) {
        final vers = i.isEven ? 1 : -1; // moitié à gauche, moitié à droite
        return AcceleratedParticle(
          speed: Vector2(
            vers * (30 + rnd.nextDouble() * 70), // surtout horizontal
            -10 - rnd.nextDouble() * 25,         // à peine vers le haut
          ),
          acceleration: Vector2(0, 90),          // gravité faible
          child: ComputedParticle(
            renderer: (canvas, particle) {
              final p = particle.progress;
              canvas.drawCircle(
                Offset.zero,
                3 + p * 6, // le grain GROSSIT
                Paint()
                  ..color = const Color(0xFF9E9E9E)
                      .withValues(alpha: 0.45 * (1 - p)), // et s'efface
              );
            },
          ),
        );
      },
    );
```

```text
  RÉGLAGES : EXPLOSION contre POUSSIÈRE

                       explosion de pièces      nuage de poussière
  vitesse X            ± 130                    ± 100
  vitesse Y            -140 à -320 (fort)       -10 à -35 (faible)
  gravité              620 (forte)              90 (faible)
  durée de vie         0.7 à 1.2 s              0.45 s
  taille dans le temps constante                croissante
  opacité              constante                décroissante
  couleur              vive                     terne, désaturée
```

Le grain qui **grossit en s'effaçant** est la signature de la fumée et de la poussière : de la matière qui se disperse. Le grain qui **rétrécit** est celle de l'étincelle : de l'énergie qui s'éteint. La poussière doit apparaître **sous les pieds**, pas au centre du personnage, et **derrière** lui.

```dart
game.world.add(ParticleSystemComponent(
  position: Vector2(position.x, position.y + size.y / 2), // les pieds
  particle: poussiere(),
  priority: -1, // dessinée DERRIÈRE le personnage
));
```

> **Remarque sur `priority`.** La poussière derrière le personnage donne de la profondeur ; les étincelles d'un coup, elles, doivent passer **devant** (`priority: 10`). C'est le z-index vu au chapitre 28.

---

## 33.29 — Un effet de sang ou d'étincelles au contact

Troisième variante : la gerbe **directionnelle**. Contrairement à l'explosion qui part dans toutes les directions, elle jaillit dans un **cône** orienté à l'opposé du coup.

```text
  Le héros frappe vers la droite.
     héros  ->|  gobelin
                 \ · ·
                  \  · ·      les étincelles partent vers la droite,
                  /   · ·     dans un cône d'environ 70 degrés
                 / · ·
```

Le calcul du cône utilise la trigonométrie du chapitre 23 : on tire un angle au hasard autour de la direction du coup.

```dart
import 'dart:math';

/// Gerbe orientée. [direction] doit être normalisée.
Particle etincelles(Vector2 direction, Color couleur, {int nombre = 9}) {
  final angleBase = atan2(direction.y, direction.x);
  const ouverture = pi / 2.6; // environ 70 degrés au total

  return Particle.generate(
    count: nombre,
    lifespan: 0.35,
    generator: (i) {
      final angle = angleBase + (rnd.nextDouble() - 0.5) * ouverture;
      return AcceleratedParticle(
        speed: Vector2(cos(angle), sin(angle)) * (120 + rnd.nextDouble() * 160),
        acceleration: Vector2(0, 260),
        child: ComputedParticle(
          renderer: (canvas, particle) {
            final p = particle.progress;
            canvas.drawCircle(
              Offset.zero,
              2.6 * (1 - p), // l'étincelle RÉTRÉCIT
              Paint()..color = couleur.withValues(alpha: 1 - p * p),
            );
          },
        ),
      );
    },
  );
}
```

Le même code, réglé autrement, donne du sang : moins de particules, plus lentes, plus lourdes, et une couleur sombre.

```dart
Particle sang(Vector2 direction) => Particle.generate(
      count: 7,
      lifespan: 0.55,
      generator: (i) {
        final angle = atan2(direction.y, direction.x) +
            (rnd.nextDouble() - 0.5) * (pi / 2);
        return AcceleratedParticle(
          speed: Vector2(cos(angle), sin(angle)) * (60 + rnd.nextDouble() * 90),
          acceleration: Vector2(0, 520), // retombe vite
          child: CircleParticle(
            radius: 1.8 + rnd.nextDouble() * 2.2,
            paint: Paint()..color = const Color(0xFF8E1F1B),
          ),
        );
      },
    );
```

Le branchement se fait dans le callback de collision du chapitre 32, en utilisant le **point d'intersection** fourni par Flame :

```dart
@override
void onCollisionStart(Set<Vector2> intersectionPoints, PositionComponent other) {
  super.onCollisionStart(intersectionPoints, other);
  if (other is Gobelin) {
    final contact = intersectionPoints.first; // coordonnées du monde
    final direction = (other.position - position).normalized();
    game.world.add(ParticleSystemComponent(
      position: contact,
      priority: 20,
      particle: etincelles(direction, const Color(0xFFFFE082)),
    ));
  }
}
```

Utiliser `intersectionPoints.first` plutôt que la position de l'ennemi change tout : les étincelles jaillissent **du point d'impact réel**, pas du centre du corps.

> **Réglage culturel.** Le sang est une décision de conception, pas un effet technique. Prévoyez systématiquement une option `bool sangActif` : le même code produit alors des étoiles blanches, et votre jeu reste accessible à tous les publics.

---

## 33.30 — Le coût des particules et comment le limiter

Les particules sont bon marché, mais pas gratuites. Le poste principal est le **nombre d'appels de dessin** : mille particules à l'écran, c'est mille appels par frame, soixante mille par seconde. S'y ajoute une allocation d'objet par particule créée, donc une pression sur le ramasse-miettes.

| Particules simultanées | Effet sur un mobile milieu de gamme |
| --- | --- |
| < 200 | aucun effet mesurable |
| 200 – 600 | encore fluide, marge confortable |
| 600 – 1500 | chute possible sous 60 images par seconde |
| > 1500 | saccades visibles, à éviter |

Cinq techniques de bridage, de la plus simple à la plus fine.

**1. Réduire `count` avant tout.** Six particules bien réglées valent mieux que trente mal réglées : c'est le meilleur rapport qualité/coût.

**2. Raccourcir la durée de vie.** Le nombre de particules présentes à l'écran est le produit du taux d'émission par la durée de vie.

```text
  particules à l'écran ≈ (gerbes par seconde) x (count) x (lifespan)
  4 gerbes/s x 9 étincelles x 0.35 s ≈ 13 particules.  Négligeable.
  60 gerbes/s x 20 x 1.0 s          = 1200 particules. Dangereux.
```

**3. Plafonner le nombre de systèmes vivants.**

```dart
void emettre(Particle p, Vector2 position, {int priority = 10}) {
  final vivants = world.children.whereType<ParticleSystemComponent>().length;
  if (vivants >= 24) return; // on saute cette gerbe
  world.add(ParticleSystemComponent(
      particle: p, position: position, priority: priority));
}
```

Sauter une gerbe est invisible quand vingt-quatre autres sont déjà à l'écran ; une chute de framerate, elle, se voit immédiatement.

**4. Ne pas émettre hors champ.** Un système hors de la vue coûte autant qu'un système visible ; `camera.visibleWorldRect` (chapitre 31) permet de le savoir.

```dart
if (!camera.visibleWorldRect.inflate(60).contains(position.toOffset())) return;
```

**5. Préférer `CircleParticle` à `ComponentParticle`,** qui embarque un composant complet avec son cycle de vie.

Enfin, le réflexe de mesure : affichez `world.children.whereType<ParticleSystemComponent>().length` dans un `TextComponent` pendant vos essais.

> **À retenir.** Si ce nombre **monte sans jamais redescendre**, vous avez oublié un `lifespan` : une particule sans durée de vie ne meurt pas, et son porteur ne se retire jamais.

---

## 33.31 — `Timer` : le compte à rebours

Les effets et les particules règlent l'apparence. Reste le **rythme** : quand faire apparaître un ennemi, combien de temps dure une invincibilité, quand une attaque redevient disponible. Flame fournit pour cela une classe `Timer`, indépendante de l'arbre de composants.

```dart
import 'package:flame/timer.dart';

Timer(
  double limit, {
  VoidCallback? onTick,
  bool repeat = false,
  bool autoStart = true,
  int? tickCount,
});
```

| Membre | Rôle |
| --- | --- |
| `limit` | durée visée, en secondes (modifiable) |
| `current` | temps écoulé depuis le début |
| `progress` | `current / limit`, entre 0 et 1 |
| `finished` | vrai quand `current` a atteint `limit` |
| `repeat` | relance automatique après chaque fin |
| `onTick` | appelé à chaque fin de période |
| `isRunning()` | **méthode** : vrai si le minuteur tourne |
| `start()` / `stop()` / `pause()` / `resume()` / `reset()` | pilotage manuel |

Attention : `isRunning()` est une **méthode** (avec parenthèses), alors que `finished` et `progress` sont des **propriétés**. C'est une source classique d'erreurs de compilation.

---

## 33.32 — `Timer.update(dt)` et `onTick`

Un `Timer` de Flame n'est **pas** un `Timer` de `dart:async`. Il ne s'exécute pas tout seul : **vous devez l'alimenter** avec le `dt` de la boucle de jeu.

```dart
class Porte extends RectangleComponent {
  final Timer _fermeture = Timer(3.0);

  @override
  void update(double dt) {
    super.update(dt);
    _fermeture.update(dt); // SANS cette ligne, rien n'avance
    if (_fermeture.finished) { /* la porte se referme */ }
  }
}
```

C'est un avantage, pas une contrainte : un `Timer` de Flame se met automatiquement en pause avec le jeu, puisqu'un jeu en pause n'appelle plus `update`.

```text
  Timer de dart:async        |  Timer de Flame
  s'exécute tout seul        |  il faut l'alimenter avec dt
  ignore la pause du jeu     |  se met en pause avec le jeu
  temps réel                 |  temps de jeu
  à éviter dans un jeu       |  à utiliser dans un jeu
```

Deux façons de réagir à la fin d'un minuteur.

**Par interrogation (`finished`).** Le `stop()` est indispensable : `finished` reste `true` tant que le minuteur n'est pas réinitialisé, donc l'action serait déclenchée soixante fois par seconde.

```dart
if (_fermeture.finished) {
  fermer();
  _fermeture.stop();
}
```

**Par rappel (`onTick`).** Presque toujours préférable : il ne peut pas se déclencher deux fois, et il exprime clairement l'intention.

```dart
final Timer _fermeture = Timer(3.0, onTick: fermer);
final intervalle = Timer(1, onTick: () => secondes += 1, repeat: true);
```

```text
  Timer(1, repeat: true, onTick: f)
  current  1 |   /|   /|   /|   /|
           0 | / | / | / | / |
             +------------------------> temps
               f    f    f    f      <- onTick à chaque retour à zéro
```

Le paramètre `tickCount` limite le nombre de déclenchements : `Timer(1, repeat: true, tickCount: 5)` s'arrête après cinq ticks.

---

## 33.33 — `TimerComponent`

Écrire `_timer.update(dt)` dans chaque `update` est répétitif, et l'oublier est l'erreur la plus fréquente du chapitre. `TimerComponent` supprime le problème : c'est un composant qui contient un `Timer` et l'alimente pour vous.

```dart
TimerComponent({
  required double period,      // en secondes
  bool repeat = false,
  bool autoStart = true,
  bool removeOnFinish = false,
  VoidCallback? onTick,
  bool tickWhenLoaded = false,
  int? tickCount,
  ComponentKey? key,
});
```

| Paramètre | Utilité |
| --- | --- |
| `period` | durée d'une période, en secondes |
| `repeat` | relance automatiquement après chaque tick |
| `autoStart` | démarre dès le montage ; `false` pour un déclenchement différé |
| `removeOnFinish` | retire le composant après le dernier tick : indispensable pour un minuteur jetable |
| `tickWhenLoaded` | déclenche `onTick` immédiatement au montage |
| `tickCount` | nombre total de ticks avec `repeat: true` |

Comparons les deux approches sur le même besoin : « faire apparaître un ennemi dans 5 secondes, une seule fois ».

```dart
// Version Timer manuel : du code réparti dans deux méthodes.
class JeuA extends FlameGame {
  final Timer _apparition = Timer(5);
  @override
  void update(double dt) {
    super.update(dt);
    _apparition.update(dt);
    if (_apparition.finished) {
      _apparition.stop();
      faireApparaitreEnnemi();
    }
  }
  void faireApparaitreEnnemi() {}
}

// Version TimerComponent : une instruction, rien dans update.
class JeuB extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    add(TimerComponent(
        period: 5, removeOnFinish: true, onTick: faireApparaitreEnnemi));
  }
  void faireApparaitreEnnemi() {}
}
```

> **Règle.** Utilisez `TimerComponent` par défaut. Ne descendez au `Timer` manuel que lorsque vous devez **lire** `progress` à chaque frame, par exemple pour dessiner une jauge de rechargement. Dans ce cas, `TimerComponent` expose lui aussi son minuteur : `monTimerComponent.timer.progress`.

---

## 33.34 — Le timer répétitif : faire apparaître un ennemi toutes les 3 secondes

Le cas d'usage numéro un de `TimerComponent`. Programme complet, sans image.

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() =>
    runApp(MaterialApp(home: Scaffold(body: GameWidget(game: JeuVagues()))));

class JeuVagues extends FlameGame {
  final Random rnd = Random();
  int apparus = 0;
  late final TextComponent info;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    info = TextComponent(
      text: 'Gobelins apparus : 0',
      position: Vector2(16, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDDD)),
      ),
    );
    await world.add(info);

    // Le générateur de vagues : un composant sans apparence.
    await add(TimerComponent(
      period: 3.0,
      repeat: true,
      tickWhenLoaded: true, // le premier gobelin arrive tout de suite
      onTick: faireApparaitre,
    ));
  }

  void faireApparaitre() {
    apparus++;
    info.text = 'Gobelins apparus : $apparus';

    final gobelin = RectangleComponent(
      position: Vector2(40 + rnd.nextDouble() * 320, 60 + rnd.nextDouble() * 180),
      size: Vector2(34, 44),
      anchor: Anchor.center,
      scale: Vector2.zero(), // il commence invisible
      paint: Paint()..color = const Color(0xFFB0413E),
    );
    world.add(gobelin);

    gobelin.add(ScaleEffect.to(Vector2.all(1.0),
        EffectController(duration: 0.35, curve: Curves.elasticOut)));
    gobelin.add(OpacityEffect.fadeOut(
      EffectController(duration: 0.4, startDelay: 4.0),
      onComplete: gobelin.removeFromParent,
    ));
  }
}
```

**Résultat :**

```text
  t = 0 s   Gobelins apparus : 1   [G]
  t = 3 s   Gobelins apparus : 2   [G]      [G]
  t = 4.4 s le premier s'efface             [G]
  t = 6 s   Gobelins apparus : 3   [G]  [G]
```

Deux réglages que ce squelette rend faciles. **Une difficulté croissante** : le `Timer` interne est accessible, donc sa période est modifiable en cours de partie avec `generateur.timer.limit = max(0.6, generateur.timer.limit * 0.9);`. **Un nombre limité de vagues** : `tickCount: 10` associé à `repeat: true` arrête le générateur après dix gobelins, sans code supplémentaire.

---

## 33.35 — Le cooldown d'une attaque

Un **cooldown** empêche le joueur de spammer une action. C'est le cas où l'on veut lire `progress` en continu, donc celui où le `Timer` manuel garde tout son sens.

```dart
class Heros extends RectangleComponent with HasGameReference<JeuDonjon> {
  Heros() : super(size: Vector2(36, 48), anchor: Anchor.center);

  final Timer _recharge = Timer(0.8, autoStart: false);

  bool get peutAttaquer => !_recharge.isRunning();

  /// De 0.0 (juste attaqué) à 1.0 (prêt).
  double get chargeRestante => _recharge.isRunning() ? _recharge.progress : 1.0;

  @override
  void update(double dt) {
    super.update(dt);
    _recharge.update(dt);
  }

  void attaquer() {
    if (!peutAttaquer) {
      // Retour d'ÉCHEC : un petit sursaut sec, sans dégâts.
      add(ScaleEffect.to(Vector2(0.92, 1.06),
          EffectController(duration: 0.05, alternate: true)));
      return;
    }
    _recharge.start();
    add(SequenceEffect([
      MoveEffect.by(Vector2(-8, 0),
          EffectController(duration: 0.06, curve: Curves.easeOut)),
      MoveEffect.by(Vector2(22, 0),
          EffectController(duration: 0.09, curve: Curves.easeIn)),
      MoveEffect.by(Vector2(-14, 0),
          EffectController(duration: 0.14, curve: Curves.easeOut)),
    ]));
  }
}
```

```text
  attaquer()   |####################|          |####...
               0                   0.8 s       1.0 s
               |<-- recharge en cours -->|
  Un appui pendant cette zone déclenche le retour d'ÉCHEC, pas
  l'attaque : le joueur comprend qu'il doit attendre.
```

Le retour d'échec fait toute la différence : sans lui, appuyer pendant le cooldown ne produit **rien** et le joueur croit que le bouton est cassé. La jauge de rechargement du HUD, elle, lit `chargeRestante` à chaque frame :

```dart
class JaugeRecharge extends RectangleComponent with HasGameReference<JeuDonjon> {
  JaugeRecharge() : super(size: Vector2(80, 6), position: Vector2(16, 40));

  @override
  void update(double dt) {
    super.update(dt);
    size.x = 80 * game.heros.chargeRestante;
    paint.color = game.heros.peutAttaquer
        ? const Color(0xFF4CAF50)
        : const Color(0xFF757575);
  }
}
```

> **Remarque.** La taille est écrite **directement** à chaque frame, sans `SizeEffect` : la jauge doit refléter l'état exact du minuteur, pas une animation indépendante qui créerait un décalage.

---

## 33.36 — L'invincibilité temporaire avec clignotement

Au chapitre 32, votre héros perdait une vie à chaque frame de contact avec un gobelin : trois vies évaporées en cinquante millisecondes. La correction se compose de deux briques.

```text
  1. UN MINUTEUR     : pendant N secondes, les dégâts sont ignorés. (logique)
  2. UN CLIGNOTEMENT : le joueur doit VOIR qu'il est invincible.    (visuel)
```

Ne gardez jamais l'une sans l'autre : une invincibilité invisible passe pour un bug de collision.

```dart
class Heros extends RectangleComponent with HasGameReference<JeuDonjon> {
  Heros() : super(size: Vector2(36, 48), anchor: Anchor.center);

  static const double dureeInvincibilite = 1.2;

  int vies = 3;
  final Timer _invincibilite = Timer(dureeInvincibilite, autoStart: false);

  bool get estInvincible => _invincibilite.isRunning();

  @override
  void update(double dt) {
    super.update(dt);
    _invincibilite.update(dt);
  }

  void subirDegats(int degats) {
    if (estInvincible) return; // le coeur du mécanisme

    vies -= degats;
    _invincibilite.start();

    // Flash rouge immédiat : « ça m'a touché ».
    add(ColorEffect(const Color(0xFFFF5252),
        EffectController(duration: 0.06, alternate: true)));

    // Clignotement : 0.075 x 2 (alternate) x 8 = 1.2 s exactement.
    add(OpacityEffect.to(
      0.25,
      EffectController(duration: 0.075, alternate: true, repeatCount: 8),
      onComplete: () => opacity = 1.0, // filet de sécurité
    ));

    game.secouer(intensite: 7, duree: 0.18);
    if (vies <= 0) game.gameOver();
  }
}
```

Le calcul du nombre de répétitions mérite d'être posé explicitement, car c'est là que tout le monde se trompe :

```text
  durée totale = duration x 2 x repeatCount   (le x 2 vient de alternate)
  Pour 1.2 s avec des clignotements de 75 ms :
     repeatCount = 1.2 / (0.075 x 2) = 8
```

Si le calcul ne tombe pas juste, l'opacité du héros restera à une valeur intermédiaire. Le `onComplete: () => opacity = 1.0` évite le fameux « héros à moitié transparent pour toujours ».

Variante plus robuste, qui ne demande aucun calcul : on pose un clignotement **infini**, et c'est un `TimerComponent` qui le retire.

```dart
final clignotement = OpacityEffect.to(0.25,
    EffectController(duration: 0.075, alternate: true, infinite: true));
add(clignotement);

add(TimerComponent(
  period: dureeInvincibilite,
  removeOnFinish: true,
  onTick: () {
    clignotement.removeFromParent();
    opacity = 1.0; // OBLIGATOIRE : l'effet infini n'a pas de fin propre
  },
));
```

Cette version reste juste si vous changez `dureeInvincibilite`. En contrepartie, la remise à `opacity = 1.0` devient obligatoire : un effet infini interrompu laisse la propriété exactement là où elle en était.

---

## 33.37 — Mini-projet : polish complet d'une scène du « Donjon de Dart »

Assemblons tout. La scène contient un héros déplacé au tap, des gobelins qui apparaissent régulièrement, des pièces qui jaillissent quand un gobelin meurt, et l'ensemble des retours de jeu du chapitre.

```text
  CAHIER DES CHARGES

  [Entrées]     un tap déplace le héros vers le point touché   (MoveEffect.to)
  [Rythme]      un gobelin apparaît toutes les 2,5 s           (TimerComponent)
  [Apparition]  le gobelin surgit avec du ressort              (ScaleEffect + elasticOut)
  [Coup porté]  flash blanc + recul + étincelles + secousse    (33.19, 33.29, 33.21)
  [Mort]        implosion + gerbe de pièces + « +10 »          (33.16, 33.27, 33.20)
  [Cooldown]    0,45 s entre deux coups                        (Timer)
  [Sécurité]    au plus 24 systèmes de particules              (33.30)
```

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/particles.dart';
import 'package:flame/timer.dart';
import 'package:flutter/material.dart';

void main() =>
    runApp(MaterialApp(home: Scaffold(body: GameWidget(game: DonjonPolish()))));

class Palette {
  static const Color heros = Color(0xFF4CAF50);
  static const Color gobelin = Color(0xFFB0413E);
  static const Color piece = Color(0xFFE8B04B);
  static const Color blanc = Color(0xFFFFFFFF);
}

class DonjonPolish extends FlameGame with TapCallbacks {
  static const int maxSystemes = 24;

  final Random rnd = Random();
  late final Heros heros;
  late final TextComponent hud;
  int score = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Damier : sans repères, la secousse ne se voit pas.
    for (var i = 0; i < 9; i++) {
      for (var j = 0; j < 6; j++) {
        await world.add(RectangleComponent(
          position: Vector2(i * 46, 40.0 + j * 46),
          size: Vector2(46, 46),
          priority: -10,
          paint: Paint()
            ..color =
                (i + j).isEven ? const Color(0xFF2B2B3A) : const Color(0xFF23232F),
        ));
      }
    }

    heros = Heros(position: Vector2(200, 180));
    await world.add(heros);

    hud = TextComponent(
      text: 'Score : 0',
      position: Vector2(14, 10),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 17, color: Color(0xFFDDDDDD)),
      ),
    );
    await camera.viewport.add(hud);

    await add(
      TimerComponent(
        period: 2.5,
        repeat: true,
        tickWhenLoaded: true,
        onTick: apparitionGobelin,
      ),
    );
  }

  @override
  void onTapDown(TapDownEvent event) => heros.allerVers(event.localPosition);

  // ------------------------------------------------- services de game feel

  /// Émet un système de particules en respectant le plafond.
  void emettre(Particle p, Vector2 position, {int priority = 20}) {
    if (world.children.whereType<ParticleSystemComponent>().length >= maxSystemes) {
      return;
    }
    world.add(ParticleSystemComponent(
      particle: p,
      position: position.clone(),
      priority: priority,
    ));
  }

  /// Secousse de caméra qui revient toujours exactement à zéro.
  void secouer({double intensite = 8, double duree = 0.2, int segments = 6}) {
    if (camera.viewfinder.children.whereType<SequenceEffect>().isNotEmpty) return;

    final pas = duree / segments;
    final decalages = <Vector2>[];
    final cumul = Vector2.zero();
    for (var i = 0; i < segments - 1; i++) {
      final force = intensite * (1 - i / segments);
      final d = Vector2((rnd.nextDouble() * 2 - 1) * force,
          (rnd.nextDouble() * 2 - 1) * force);
      decalages.add(d);
      cumul.add(d);
    }
    decalages.add(-cumul);

    camera.viewfinder.add(SequenceEffect(
      decalages.map((d) => MoveEffect.by(d, EffectController(duration: pas))).toList(),
    ));
  }

  /// Gerbe orientée d'étincelles, dans un cône d'environ 70 degrés.
  Particle etincelles(Vector2 direction, Color couleur, {int nombre = 9}) {
    final base = atan2(direction.y, direction.x);
    return Particle.generate(
      count: nombre,
      lifespan: 0.35,
      generator: (i) {
        final a = base + (rnd.nextDouble() - 0.5) * (pi / 2.6);
        return AcceleratedParticle(
          speed: Vector2(cos(a), sin(a)) * (120 + rnd.nextDouble() * 160),
          acceleration: Vector2(0, 260),
          child: ComputedParticle(
            renderer: (canvas, particle) {
              final p = particle.progress;
              canvas.drawCircle(Offset.zero, 2.6 * (1 - p),
                  Paint()..color = couleur.withValues(alpha: 1 - p * p));
            },
          ),
        );
      },
    );
  }

  /// Gerbe de pièces qui retombent.
  Particle gerbeDePieces() => Particle.generate(
        count: 12,
        applyLifespanToChildren: false,
        generator: (i) => AcceleratedParticle(
          lifespan: 0.6 + rnd.nextDouble() * 0.4,
          speed: Vector2(
              rnd.nextDouble() * 220 - 110, -120 - rnd.nextDouble() * 150),
          acceleration: Vector2(0, 600),
          child: CircleParticle(
            radius: 2.0 + rnd.nextDouble() * 2.0,
            paint: Paint()
              ..color = i.isEven ? Palette.piece : const Color(0xFFFFD54F),
          ),
        ),
      );

  /// Texte flottant « +N » et pop du score.
  Future<void> gagnerPoints(int valeur, Vector2 position) async {
    score += valeur;
    hud.text = 'Score : $score';
    hud.add(ScaleEffect.by(Vector2.all(1.22),
        EffectController(duration: 0.09, alternate: true, curve: Curves.easeOut)));

    final texte = TextComponent(
      text: '+$valeur',
      position: position.clone(),
      anchor: Anchor.center,
      priority: 60,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 18, color: Color(0xFFFFF176)),
      ),
    );
    await world.add(texte);
    texte.add(MoveEffect.by(
        Vector2(0, -44), EffectController(duration: 0.7, curve: Curves.easeOut)));
    texte.add(OpacityEffect.fadeOut(
      EffectController(duration: 0.3, startDelay: 0.4),
      onComplete: texte.removeFromParent,
    ));
  }

  void apparitionGobelin() {
    world.add(Gobelin(
      position: Vector2(50 + rnd.nextDouble() * 300, 80 + rnd.nextDouble() * 170),
    ));
  }
}

class Heros extends RectangleComponent with HasGameReference<DonjonPolish> {
  Heros({required Vector2 position})
      : super(
          position: position,
          size: Vector2(34, 44),
          anchor: Anchor.center,
          priority: 10,
          paint: Paint()..color = Palette.heros);

  final Timer _recharge = Timer(0.45, autoStart: false);
  bool get peutFrapper => !_recharge.isRunning();

  @override
  Future<void> onLoad() async {
    // Respiration : le héros n'est jamais parfaitement immobile.
    add(ScaleEffect.by(
      Vector2(1.0, 1.05),
      EffectController(
        duration: 0.9,
        alternate: true,
        infinite: true,
        curve: Curves.easeInOut,
      ),
    ));
  }

  @override
  void update(double dt) {
    super.update(dt);
    _recharge.update(dt);
    for (final g in game.world.children.whereType<Gobelin>()) {
      if (g.vivant && (g.position - position).length < 34) {
        frapper(g);
        break;
      }
    }
  }

  void allerVers(Vector2 cible) {
    // Un seul MoveEffect.to à la fois : on retire le précédent.
    children.whereType<MoveToEffect>().forEach((e) => e.removeFromParent());
    add(MoveEffect.to(
        cible.clone(), EffectController(duration: 0.45, curve: Curves.easeOut)));
  }

  void frapper(Gobelin g) {
    if (!peutFrapper) return;
    _recharge.start();

    final direction = (g.position - position).normalized();
    add(SequenceEffect([
      MoveEffect.by(direction * -6, EffectController(duration: 0.05)),
      MoveEffect.by(direction * 16,
          EffectController(duration: 0.07, curve: Curves.easeIn)),
      MoveEffect.by(direction * -10,
          EffectController(duration: 0.12, curve: Curves.easeOut)),
    ]));

    g.subirDegats(direction);
  }
}

class Gobelin extends RectangleComponent with HasGameReference<DonjonPolish> {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          size: Vector2(32, 42),
          anchor: Anchor.center,
          scale: Vector2.zero(),
          paint: Paint()..color = Palette.gobelin);

  int vies = 2;
  bool vivant = true;

  @override
  Future<void> onLoad() async {
    add(ScaleEffect.to(Vector2.all(1.0),
        EffectController(duration: 0.4, curve: Curves.elasticOut)));
  }

  void subirDegats(Vector2 direction) {
    if (!vivant) return;
    vies--;

    add(ColorEffect(Palette.blanc,
        EffectController(duration: 0.05, alternate: true),
        opacityTo: 1.0));
    add(MoveEffect.by(direction * 16,
        EffectController(duration: 0.07, alternate: true, curve: Curves.easeOut)));
    add(ScaleEffect.to(Vector2(1.25, 0.78),
        EffectController(duration: 0.06, alternate: true, curve: Curves.easeOut)));

    game.emettre(
      game.etincelles(direction, const Color(0xFFFFE082)),
      position + direction * 14,
    );
    game.secouer(
      intensite: vies <= 0 ? 11 : 5,
      duree: vies <= 0 ? 0.26 : 0.14,
    );

    if (vies <= 0) mourir();
  }

  void mourir() {
    vivant = false;
    game.emettre(game.gerbeDePieces(), position, priority: 30);
    game.gagnerPoints(10, position);

    add(SequenceEffect(
      [
        ScaleEffect.to(Vector2.all(1.45),
            EffectController(duration: 0.07, curve: Curves.easeOut)),
        ScaleEffect.to(Vector2.zero(),
            EffectController(duration: 0.20, curve: Curves.easeIn)),
      ],
      onComplete: removeFromParent,
    ));
  }
}
```

**Résultat :**

```text
  [tap sur un gobelin]
  -> le héros glisse vers lui en 0,45 s (easeOut)
  -> à moins de 34 px : coup porté
     . le héros pique vers l'avant puis recule
     . le gobelin blanchit, recule, s'écrase
     . 9 étincelles jaillissent du point de contact
     . l'écran tremble de 5 px pendant 0,14 s
  -> second coup 0,45 s plus tard : le gobelin meurt
     . il gonfle puis implose
     . 12 pièces jaillissent et retombent
     . « +10 » monte et s'efface, le score fait un pop
     . l'écran tremble de 11 px pendant 0,26 s
```

Commentez toutes les lignes `add(...Effect(...))` et `game.emettre(...)` : la logique reste rigoureusement identique, l'expérience n'a plus rien à voir. Trois points de conception à retenir. **Les services de retour de jeu sont dans la classe du jeu** : `emettre`, `secouer` et `gagnerPoints` sont des méthodes de `DonjonPolish`, pas du gobelin, qui **demande** un retour sans décider comment il est produit (chapitre 26). **Chaque déclencheur est une transition** : `subirDegats` n'est appelé que depuis `frapper`, lui-même protégé par un cooldown, et aucun effet n'est ajouté dans un `update` sans garde. **Chaque effet a une fin** : le seul effet infini du programme est la respiration du héros, qui appartient au décor.

---

## 33.38 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le framerate s'effondre et l'ennemi reste blanc | un effet est ajouté **à chaque frame** dans `update` tant que la condition est vraie | déclencher sur une transition (`onCollisionStart`, `subirDegats`), ou tester `children.whereType<ColorEffect>().isEmpty` avant d'ajouter |
| Le héros reste à moitié transparent pour toujours | un `OpacityEffect` infini ou interrompu a laissé l'opacité à une valeur intermédiaire | remettre `opacity = 1.0` dans `onComplete` ou après avoir retiré l'effet |
| Le `Timer` ne se déclenche jamais | `Timer.update(dt)` n'a pas été appelé : un `Timer` de Flame ne tourne pas tout seul | appeler `_timer.update(dt)` dans `update`, ou utiliser un `TimerComponent` |
| Le code de fin d'animation ne s'exécute pas | `onComplete` sur un contrôleur `infinite: true` : l'effet ne se termine jamais | retirer `infinite`, ou piloter la fin avec un `TimerComponent` séparé |
| L'action se déclenche 60 fois par seconde | on teste `timer.finished` sans appeler `stop()` ni `reset()` | préférer `onTick`, ou appeler `stop()` juste après avoir traité la fin |
| Le flash dure deux fois trop longtemps | `alternate: true` **double** la durée totale (aller + retour) | diviser `duration` par deux : `duration: 0.05, alternate: true` fait 0,1 s |
| Le jeu saccade après quelques secondes de combat | aucune limite au nombre de systèmes de particules, ou `lifespan` oublié | plafonner le nombre de `ParticleSystemComponent` et toujours fixer un `lifespan` |
| Les particules apparaissent en haut à gauche de l'écran | une particule se dessine autour de l'origine du canvas : c'est la position du **composant** porteur qui compte | renseigner `position:` sur le `ParticleSystemComponent`, pas sur la particule |
| L'effet a l'air mou et faux | courbe inadaptée : `linear` ou `easeIn` sur un impact | utiliser `easeOut` pour tout ce qui arrive et se pose (tableau 33.14) |
| L'opacité n'atteint jamais sa valeur cible | `Curves.elasticOut` ou `bounceOut` sortent de l'intervalle [0, 1] ; le dépassement est écrêté | n'utiliser ces courbes que sur `MoveEffect`, `ScaleEffect` ou `RotateEffect` |
| L'écran dérive un peu plus à chaque secousse | la somme des déplacements appliqués au viewfinder n'est pas nulle | calculer le dernier segment comme l'opposé du cumul, ou utiliser `alternate: true` |
| Le composant grandit vers le bas à droite au lieu de gonfler | `ScaleEffect` s'applique autour de l'**ancre**, ici `Anchor.topLeft` par défaut | mettre `anchor: Anchor.center` sur tout composant qui reçoit des effets |
| Deux effets se battent et la position devient imprévisible | deux `MoveEffect.to` (ou deux `OpacityEffect.to`) sur la même propriété | un seul effet `.to` par propriété ; les effets `.by` s'additionnent sans conflit |
| Le second `ColorEffect` est le seul visible | limitation documentée : un seul `ColorEffect` actif par composant | n'en garder qu'un, ou superposer un composant enfant animé en opacité |
| `Un objet est ramassé deux fois` | l'animation de sortie dure 300 ms pendant lesquelles le composant est encore dans l'arbre | poser un drapeau `bool ramassee` testé en début de méthode |
| `The argument type ... can't be assigned` sur `RotateAroundEffect` | `center` est un argument **nommé** : il ne peut pas précéder le contrôleur positionnel | `RotateAroundEffect(angle, controller, center: v)` |
| Le même effet appliqué à plusieurs composants ne marche que sur un seul | un `Effect` est un composant : il ne peut avoir qu'un seul parent | écrire une **fonction** qui construit une instance neuve à chaque appel |
| Le minuteur continue pendant le menu de pause | on a utilisé le `Timer` de `dart:async` au lieu de celui de Flame | `import 'package:flame/timer.dart';` et alimenter le minuteur avec `dt` |

---

## 33.39 — Résumé du chapitre

| Effet | Ce qu'il anime | Contrôleur typique |
| --- | --- | --- |
| `MoveEffect.by` | `position`, en relatif | `EffectController(duration: 0.07, alternate: true, curve: Curves.easeOut)` — recul |
| `MoveEffect.to` | `position`, vers un point absolu | `EffectController(duration: 0.45, curve: Curves.easeOut)` — déplacement dirigé |
| `MoveAlongPathEffect` | `position` le long d'un `Path` | `EffectController(duration: 1.5)` |
| `ScaleEffect.by` | `scale`, en multipliant | `EffectController(duration: 0.10, alternate: true)` — pop |
| `ScaleEffect.to` | `scale`, valeur absolue | `EffectController(duration: 0.35, curve: Curves.elasticOut)` — apparition |
| `RotateEffect.by` | `angle`, en radians | `EffectController(duration: 2.0, infinite: true)` — rotation continue |
| `RotateAroundEffect` | `angle` autour d'un centre | `EffectController(duration: 3.0, infinite: true)` — orbite |
| `OpacityEffect.to` | opacité de la `Paint` | `EffectController(duration: 0.075, alternate: true, repeatCount: 8)` — clignotement |
| `OpacityEffect.fadeOut` | opacité vers 0 | `EffectController(duration: 0.3, startDelay: 0.4)` — disparition différée |
| `ColorEffect` | teinte de la `Paint` | `EffectController(duration: 0.05, alternate: true)` — flash de dégât |
| `SizeEffect.to` | `size`, en pixels | `EffectController(duration: 0.25, curve: Curves.easeOut)` — jauge |
| `AnchorToEffect` | `anchor` | `EffectController(duration: 0.2)` — déplacement de pivot |
| `SequenceEffect` | enchaînement d'effets | pas de contrôleur : la durée est la somme des enfants |
| `RemoveEffect` | retire le composant | `RemoveEffect(delay: 3.0)` |

| Autre notion | À retenir |
| --- | --- |
| `EffectController` | `duration` + `curve` + `alternate` + `repeatCount` / `infinite` + `startDelay` |
| Durée totale | `startDelay + (duration + reverseDuration + pauses) x repeatCount` |
| `alternate: true` | équivaut à `reverseDuration = duration` : **double** la durée totale |
| Courbes | `easeOut` par défaut ; `elasticOut` pour la joie ; `bounceOut` pour l'impact |
| Effets en parallèle | plusieurs `add()` sur la même cible ; `.by` s'additionne, `.to` se bat |
| `removeOnFinish` | vaut `true` par défaut : l'effet se retire seul |
| `onComplete` | appelé avant le retrait ; **jamais** avec `infinite: true` |
| `Particle` | objet léger, pas un composant ; possède `lifespan` et `progress` |
| `ParticleSystemComponent` | le porteur ; sa `position` détermine où la gerbe apparaît |
| `Particle.generate` | crée `count` particules dans **un seul** composant |
| `AcceleratedParticle` | physique : `position`, `speed`, `acceleration` |
| `MovingParticle` | trajet imposé de `from` à `to`, avec une `curve` |
| `ComputedParticle` | vous dessinez, en lisant `particle.progress` |
| Coût des particules | un appel de dessin par particule ; plafonner le nombre de systèmes |
| `Timer` | doit être alimenté par `update(dt)` ; `isRunning()` est une **méthode** |
| `TimerComponent` | s'alimente seul ; `period`, `repeat`, `removeOnFinish`, `tickWhenLoaded` |
| Game feel | immédiateté, lisibilité, proportionnalité ; budget 0,05 à 0,6 s |

---

## 33.40 — Exercices

### Exercice 1 — La potion qui respire (facile)

Affichez un `CircleComponent` violet de rayon 14 au centre de l'écran. Faites-le pulser indéfiniment entre 100 % et 125 % de sa taille, avec une pulsation complète de 0,9 seconde et une courbe `easeInOut`.

### Exercice 2 — Apparition en fondu (facile)

Ajoutez cinq `RectangleComponent` alignés. Chacun doit apparaître en fondu (`OpacityEffect.fadeIn`) en 0,4 seconde, mais **décalé** de 0,15 seconde par rapport au précédent, de gauche à droite.

### Exercice 3 — Le coffre qui tombe (facile)

Faites tomber un rectangle marron depuis le haut de l'écran jusqu'au sol en 0,8 seconde, avec `Curves.bounceOut`. Ajoutez un écrasement (`ScaleEffect.to(Vector2(1.25, 0.75), ...)`) qui démarre exactement au moment où le coffre touche le sol.

### Exercice 4 — Le flash proportionnel (moyen)

Écrivez une classe `Gobelin` avec une méthode `subirDegats(int degats)`. Le flash blanc doit avoir une intensité (`opacityTo`) et une durée proportionnelles aux dégâts : 1 dégât donne 0,4 d'intensité et 0,04 s ; 5 dégâts donnent 1,0 et 0,10 s. Un tap inflige un nombre de dégâts aléatoire entre 1 et 5.

### Exercice 5 — La clé du donjon (moyen)

Quand on tape sur une clé (un petit rectangle doré), elle doit : monter de 30 px en 0,3 s, faire un tour complet sur elle-même en 0,4 s, puis s'effacer en 0,2 s et se retirer. Utilisez une seule `SequenceEffect`.

### Exercice 6 — La couronne d'onde de choc (moyen)

Écrivez une fonction `Particle couronne(Color c)` qui génère 24 particules réparties **régulièrement** sur un cercle (utilisez l'index `i` du générateur, pas le hasard). Toutes partent à la même vitesse, sans gravité, et rétrécissent en s'effaçant.

### Exercice 7 — La poussière d'atterrissage (moyen)

Écrivez une fonction `Particle poussiere()` produisant 12 grains gris qui partent surtout horizontalement, subissent une gravité faible, **grossissent** de 3 à 9 px et voient leur opacité passer de 0,5 à 0. Durée de vie : 0,5 s. Émettez la poussière sous les pieds d'un carré quand on tape.

### Exercice 8 — Les vagues qui accélèrent (difficile)

Un `TimerComponent` fait apparaître un gobelin toutes les 3 secondes. À chaque apparition, la période doit être multipliée par 0,88, sans jamais descendre sous 0,7 seconde. Affichez la période courante dans un `TextComponent`.

### Exercice 9 — Le cooldown avec jauge (difficile)

Un héros peut frapper toutes les 1,2 seconde. Une jauge horizontale de 100 px sous le héros se remplit progressivement pendant la recharge et devient verte quand l'attaque est prête. Un appui pendant la recharge produit un petit sursaut de refus.

### Exercice 10 — L'invincibilité propre (difficile)

Écrivez un héros à 3 vies qui devient invincible 1,5 seconde après chaque coup reçu. Pendant l'invincibilité, il clignote ; à la fin, son opacité doit valoir **exactement 1.0**, quelle que soit la façon dont l'effet s'est terminé. Un tap inflige un dégât. Affichez les vies restantes.

---

## 33.41 — Corrections des exercices

Les dix programmes ci-dessous sont complets et exécutables tels quels, sans aucun fichier image.

### Correction 1

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex1()))));

class Ex1 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    final potion = CircleComponent(
        radius: 14,
        position: size / 2,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF7E57C2));
    await world.add(potion);
    potion.add(ScaleEffect.by(
      Vector2.all(1.25),
      EffectController(
        duration: 0.45, // 0,45 x 2 = 0,9 s par pulsation complète
        alternate: true,
        infinite: true,
        curve: Curves.easeInOut,
      ),
    ));
  }
}
```

**Explication :** la pulsation complète comprend l'aller **et** le retour ; avec `alternate: true`, on écrit donc la moitié de la durée voulue, soit `0.45`. `ScaleEffect.by(Vector2.all(1.25), ...)` multiplie l'échelle par 1,25, ce qui donne bien « 125 % de la taille », et l'ancre `Anchor.center` fait grossir la potion dans toutes les directions.

---

### Correction 2

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex2()))));

class Ex2 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    for (var i = 0; i < 5; i++) {
      final bloc = RectangleComponent(
          position: Vector2(40.0 + i * 70, 140),
          size: Vector2(52, 52),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF4CAF50));
      bloc.opacity = 0; // invisible au départ
      await world.add(bloc);
      bloc.add(OpacityEffect.fadeIn(
          EffectController(duration: 0.4, startDelay: i * 0.15)));
    }
  }
}
```

**Explication :** `startDelay: i * 0.15` produit la vague : le bloc 0 démarre à 0 s, le bloc 4 à 0,6 s. Le point crucial est `bloc.opacity = 0;` **avant** l'ajout de l'effet : sans cette ligne les blocs seraient déjà opaques et `fadeIn`, qui est un raccourci pour `OpacityEffect.to(1.0, ...)`, n'aurait rien à faire.

---

### Correction 3

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex3()))));

class Ex3 extends FlameGame {
  // Curves.bounceOut atteint 1.0 pour la première fois à t = 1 / 2.75.
  static const double instantContact = 0.8 * (1 / 2.75); // environ 0,291 s

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent(
        position: Vector2(0, 240),
        size: Vector2(400, 60),
        paint: Paint()..color = const Color(0xFF2B2B3A)));

    final coffre = RectangleComponent(
      position: Vector2(200, -30),
      size: Vector2(56, 44),
      anchor: Anchor.bottomCenter,
      paint: Paint()..color = const Color(0xFF8D6E63),
    );
    await world.add(coffre);

    coffre.add(MoveEffect.to(Vector2(200, 240),
        EffectController(duration: 0.8, curve: Curves.bounceOut)));
    coffre.add(ScaleEffect.to(
      Vector2(1.25, 0.75),
      EffectController(
          duration: 0.06, alternate: true, startDelay: instantContact),
    ));
  }
}
```

**Explication :** `Curves.bounceOut` atteint sa valeur maximale une première fois à `t = 1 / 2.75` de la durée, soit environ 36 % : c'est l'instant du premier contact avec le sol. En multipliant par la durée totale (0,8 s) on obtient le `startDelay` de l'écrasement. L'ancre `Anchor.bottomCenter` fait que la position désigne le **bas** du coffre, qui s'arrête donc exactement sur le sol.

---

### Correction 4

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex4()))));

class Ex4 extends FlameGame with TapCallbacks {
  final Random rnd = Random();
  late final Gobelin gobelin;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    gobelin = Gobelin(position: Vector2(200, 160));
    await world.add(gobelin);
  }

  @override
  void onTapDown(TapDownEvent event) => gobelin.subirDegats(1 + rnd.nextInt(5));
}

class Gobelin extends RectangleComponent {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          size: Vector2(44, 56),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFB0413E));

  void subirDegats(int degats) {
    final t = ((degats - 1) / 4).clamp(0.0, 1.0); // 1 -> 0.0 ; 5 -> 1.0
    final intensite = 0.4 + t * 0.6; // 0,4 à 1,0
    final duree = 0.04 + t * 0.06; // 0,04 à 0,10 s

    add(ColorEffect(
      const Color(0xFFFFFFFF),
      EffectController(duration: duree, alternate: true),
      opacityTo: intensite,
    ));
    add(ScaleEffect.to(
      Vector2(1.0 + t * 0.3, 1.0 - t * 0.2),
      EffectController(duration: duree, alternate: true, curve: Curves.easeOut),
    ));
  }
}
```

**Explication :** on ramène les dégâts sur un intervalle normalisé `t` entre 0 et 1, puis on interpole linéairement l'intensité et la durée. C'est le principe de **proportionnalité** de la section 33.1 : le joueur distingue un coup faible d'un coup fort sans lire de chiffre. Un seul `ColorEffect` est actif à la fois, conformément à la limitation de la section 33.7.

---

### Correction 5

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/geometry.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex5()))));

class Ex5 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Cle(position: Vector2(200, 170)));
  }
}

class Cle extends RectangleComponent with TapCallbacks {
  Cle({required Vector2 position})
      : super(
          position: position,
          size: Vector2(30, 12),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B));

  bool prise = false;

  @override
  void onTapDown(TapDownEvent event) {
    if (prise) return;
    prise = true;
    add(SequenceEffect(
      [
        MoveEffect.by(Vector2(0, -30),
            EffectController(duration: 0.3, curve: Curves.easeOut)),
        RotateEffect.by(tau, EffectController(duration: 0.4)),
        OpacityEffect.fadeOut(EffectController(duration: 0.2)),
      ],
      onComplete: removeFromParent,
    ));
  }
}
```

**Explication :** la séquence enchaîne les trois effets dans l'ordre, pour une durée totale de 0,9 s. `tau`, fourni par `package:flame/geometry.dart`, vaut un tour complet en radians. Le drapeau `prise` empêche un second tap de lancer une deuxième séquence pendant que la première joue : la clé est encore dans l'arbre pendant 0,9 s.

---

### Correction 6

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/geometry.dart';
import 'package:flame/particles.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex6()))));

class Ex6 extends FlameGame with TapCallbacks {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
  }

  @override
  void onTapDown(TapDownEvent event) {
    world.add(ParticleSystemComponent(
      position: event.localPosition,
      particle: couronne(const Color(0xFF64B5F6)),
    ));
  }

  Particle couronne(Color c, {int nombre = 24}) {
    return Particle.generate(
      count: nombre,
      lifespan: 0.6,
      generator: (i) {
        final angle = i / nombre * tau; // répartition RÉGULIÈRE
        return AcceleratedParticle(
          speed: Vector2(cos(angle), sin(angle)) * 180,
          acceleration: Vector2.zero(), // aucune gravité : l'onde reste plane
          child: ComputedParticle(
            renderer: (canvas, particle) {
              final p = particle.progress;
              canvas.drawCircle(Offset.zero, 4 * (1 - p),
                  Paint()..color = c.withValues(alpha: 1 - p));
            },
          ),
        );
      },
    );
  }
}
```

**Explication :** la clé est `i / nombre * tau`, qui répartit les 24 particules sur un cercle complet. Comme toutes ont la même vitesse et aucune accélération, elles restent à égale distance du centre : on voit un anneau qui grandit, pas un nuage. Le rayon `4 * (1 - p)` et l'alpha `1 - p` éteignent chaque particule en la faisant rétrécir, signature visuelle de l'énergie.

---

### Correction 7

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/particles.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex7()))));

class Ex7 extends FlameGame with TapCallbacks {
  final Random rnd = Random();
  late final RectangleComponent perso;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(RectangleComponent(
        position: Vector2(0, 220),
        size: Vector2(400, 60),
        paint: Paint()..color = const Color(0xFF2B2B3A)));
    perso = RectangleComponent(
      position: Vector2(200, 220),
      size: Vector2(34, 44),
      anchor: Anchor.bottomCenter, // la position désigne les PIEDS
      paint: Paint()..color = const Color(0xFF4CAF50),
    );
    await world.add(perso);
  }

  @override
  void onTapDown(TapDownEvent event) {
    world.add(ParticleSystemComponent(
      position: perso.position.clone(),
      priority: -1, // derrière le personnage
      particle: poussiere(),
    ));
  }

  Particle poussiere() {
    return Particle.generate(
      count: 12,
      lifespan: 0.5,
      generator: (i) {
        final vers = i.isEven ? 1 : -1;
        return AcceleratedParticle(
          speed: Vector2(
              vers * (35 + rnd.nextDouble() * 75), -8 - rnd.nextDouble() * 22),
          acceleration: Vector2(0, 90), // gravité FAIBLE : la poussière flotte
          child: ComputedParticle(
            renderer: (canvas, particle) {
              final p = particle.progress;
              canvas.drawCircle(
                Offset.zero,
                3 + p * 6, // le grain GROSSIT
                Paint()
                  ..color = const Color(0xFF9E9E9E)
                      .withValues(alpha: 0.5 * (1 - p)),
              );
            },
          ),
        );
      },
    );
  }
}
```

**Explication :** l'ancre `Anchor.bottomCenter` fait que `perso.position` désigne déjà ses pieds : c'est le point d'émission voulu, sans calcul. Le couple « grossit et s'efface » distingue la poussière de l'étincelle, qui rétrécit. La gravité faible et la vitesse verticale presque nulle donnent l'étalement horizontal caractéristique.

---

### Correction 8

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex8()))));

class Ex8 extends FlameGame {
  final Random rnd = Random();
  late final TimerComponent generateur;
  late final TextComponent info;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    info = TextComponent(
      text: 'Periode : 3.00 s',
      position: Vector2(16, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDDD)),
      ),
    );
    await world.add(info);

    generateur = TimerComponent(
      period: 3.0,
      repeat: true,
      tickWhenLoaded: true,
      onTick: apparition,
    );
    await add(generateur);
  }

  void apparition() {
    final g = RectangleComponent(
      position: Vector2(40 + rnd.nextDouble() * 320, 70 + rnd.nextDouble() * 180),
      size: Vector2(30, 40),
      anchor: Anchor.center,
      scale: Vector2.zero(),
      paint: Paint()..color = const Color(0xFFB0413E),
    );
    world.add(g);
    g.add(ScaleEffect.to(Vector2.all(1.0),
        EffectController(duration: 0.35, curve: Curves.elasticOut)));
    g.add(RemoveEffect(delay: 3.0));

    // Accélération, avec un plancher de sécurité.
    generateur.timer.limit = max(0.7, generateur.timer.limit * 0.88);
    info.text = 'Periode : ${generateur.timer.limit.toStringAsFixed(2)} s';
  }
}
```

**Explication :** `TimerComponent` expose son minuteur interne via la propriété `timer`, dont `limit` est modifiable en cours de partie. `max(0.7, ...)` garantit le plancher demandé : sans lui la période tendrait vers zéro. `tickWhenLoaded: true` déclenche la première apparition immédiatement, et `RemoveEffect(delay: 3.0)` nettoie chaque gobelin sans qu'on ait à tenir une liste.

---

### Correction 9

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/timer.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex9()))));

class Ex9 extends FlameGame with TapCallbacks {
  late final Heros heros;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    heros = Heros(position: Vector2(200, 150));
    await world.add(heros);
    await world.add(Jauge(heros: heros, position: Vector2(150, 190)));
  }

  @override
  void onTapDown(TapDownEvent event) => heros.frapper();
}

class Heros extends RectangleComponent {
  Heros({required Vector2 position})
      : super(
          position: position,
          size: Vector2(36, 48),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF4CAF50));

  final Timer recharge = Timer(1.2, autoStart: false);

  bool get pret => !recharge.isRunning();
  double get charge => recharge.isRunning() ? recharge.progress : 1.0;

  @override
  void update(double dt) {
    super.update(dt);
    recharge.update(dt); // indispensable : le Timer ne tourne pas seul
  }

  void frapper() {
    if (!pret) {
      // Retour d'ÉCHEC : sursaut sec, sans attaque.
      add(ScaleEffect.to(Vector2(0.9, 1.08),
          EffectController(duration: 0.05, alternate: true)));
      return;
    }
    recharge.start();
    add(SequenceEffect([
      MoveEffect.by(Vector2(-8, 0), EffectController(duration: 0.06)),
      MoveEffect.by(
          Vector2(24, 0), EffectController(duration: 0.08, curve: Curves.easeIn)),
      MoveEffect.by(Vector2(-16, 0),
          EffectController(duration: 0.14, curve: Curves.easeOut)),
    ]));
  }
}

class Jauge extends RectangleComponent {
  Jauge({required this.heros, required Vector2 position})
      : super(position: position, size: Vector2(0, 8));

  static const double largeur = 100;
  final Heros heros;

  @override
  void update(double dt) {
    super.update(dt);
    size.x = largeur * heros.charge;
    paint.color = heros.pret ? const Color(0xFF4CAF50) : const Color(0xFF757575);
  }
}
```

**Explication :** le `Timer` manuel est ici justifié, car la jauge lit `progress` à **chaque frame**. L'oubli de `recharge.update(dt)` est l'erreur la plus fréquente du chapitre. L'ancre par défaut `Anchor.topLeft` de la jauge la fait se remplir par la droite, bord gauche fixe. Le sursaut de refus informe le joueur que l'action a été reçue mais refusée, au lieu de ne rien afficher du tout.

---

### Correction 10

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/timer.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: Scaffold(body: GameWidget(game: Ex10()))));

class Ex10 extends FlameGame with TapCallbacks {
  late final Heros heros;
  late final TextComponent info;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    heros = Heros(position: Vector2(200, 160));
    await world.add(heros);

    info = TextComponent(
      text: 'Vies : 3',
      position: Vector2(16, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFDDDDDD)),
      ),
    );
    await world.add(info);
  }

  @override
  void onTapDown(TapDownEvent event) {
    heros.subirDegats(1);
    info.text = 'Vies : ${heros.vies}';
  }
}

class Heros extends RectangleComponent {
  Heros({required Vector2 position})
      : super(
          position: position,
          size: Vector2(38, 50),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF4CAF50));

  static const double duree = 1.5;

  int vies = 3;
  final Timer _invincibilite = Timer(duree, autoStart: false);
  OpacityEffect? _clignotement;

  bool get estInvincible => _invincibilite.isRunning();

  @override
  void update(double dt) {
    super.update(dt);
    _invincibilite.update(dt);
  }

  void subirDegats(int degats) {
    if (estInvincible || vies <= 0) return;

    vies -= degats;
    _invincibilite.start();

    add(ColorEffect(const Color(0xFFFF5252),
        EffectController(duration: 0.06, alternate: true)));

    // Clignotement INFINI : sa durée ne dépend d'aucun calcul de repeatCount.
    final effet = OpacityEffect.to(0.25,
        EffectController(duration: 0.075, alternate: true, infinite: true));
    _clignotement = effet;
    add(effet);

    // C'est le minuteur qui décide de la fin.
    add(TimerComponent(
      period: duree,
      removeOnFinish: true,
      onTick: () {
        _clignotement?.removeFromParent();
        _clignotement = null;
        opacity = 1.0; // OBLIGATOIRE après un effet infini interrompu
      },
    ));
  }
}
```

**Explication :** la difficulté est la garantie « opacité exactement 1.0 à la fin ». Un `repeatCount` calculé à la main casse dès que l'on change `duree` ; un effet **infini** retiré par un `TimerComponent` reste juste quelle que soit la durée. En contrepartie, un effet infini n'appelle jamais `onComplete` : la remise à `opacity = 1.0` doit être écrite explicitement dans le `onTick`. Le test `if (estInvincible || vies <= 0) return;` bloque à la fois les dégâts pendant l'invincibilité et ceux infligés à un héros déjà mort.

---

## Et maintenant ?

Votre scène du « Donjon de Dart » ne se contente plus de fonctionner : elle **répond**. Chaque coup porté, chaque pièce ramassée, chaque apparition d'ennemi produit un retour immédiat, lisible et proportionné. Vous savez animer une propriété avec un effet, régler une durée avec un `EffectController`, choisir une courbe qui exprime une intention, projeter des centaines de particules pour le prix d'un seul composant, et rythmer le jeu avec des minuteries qui se mettent en pause en même temps que lui.

Il manque encore une dimension entière, et c'est celle qui porte la moitié de la sensation de jeu : le **son**. Un flash blanc sans impact sonore reste à moitié muet. Il manque aussi le moyen de construire de vrais niveaux sans écrire cinq cents `RectangleComponent` à la main, et une physique capable de gérer des empilements et des rebonds réalistes.

Le chapitre suivant traite les trois. Vous y verrez `flame_audio` et la différence entre un effet court et une musique de fond, `FlameAudio.bgm` et son cycle de vie, la mise en cache des sons pour éviter la saccade au premier tir, puis `flame_tiled` pour charger une carte dessinée dans l'éditeur Tiled et en extraire les objets, et enfin un aperçu de `flame_forge2d` pour savoir quand une vraie physique s'impose et quand elle est un piège.

Rendez-vous au chapitre 34 : [34-PARTIE-2B—AUDIO-TILED-ET-PHYSIQUE-AVANCÉE.md](./34-PARTIE-2B—AUDIO-TILED-ET-PHYSIQUE-AVANCÉE.md)
