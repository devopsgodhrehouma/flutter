# PARTIE 2C — LE JEU COMPLET « DONJON DE DART »
# CHAPITRE 38 — OBJETS, COLLECTIBLES, SCORE ET HUD

> **Niveau :** intermédiaire
> **Durée estimée :** 9 h
> **Pré-requis :** chapitres 35 à 37 (architecture du jeu, joueur, ennemis et combat), chapitre 31 (caméra, viewport, HUD), chapitre 32 (collisions, hitbox passives, capteurs), chapitre 33 (effets, particules, timers), chapitre 30 (joystick et boutons tactiles)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** poser dans le donjon des objets ramassables typés, faire monter un score avec un multiplicateur de combo, et afficher au-dessus du jeu une interface complète — barre de vie animée, jauge d'énergie, compteur de score qui défile, vies, clés et objectif — qui ne bouge jamais quand la caméra bouge.

---

## 38.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- décrire précisément l'état du projet « Donjon de Dart » à la fin du chapitre 37 ;
- écrire une **classe abstraite** `Collectible` qui factorise tout le comportement commun d'un objet ramassable ;
- expliquer pourquoi un objet ramassable porte une hitbox **passive** et non active ;
- distinguer un **capteur** (trigger) d'un obstacle solide, et le prouver dans le code ;
- implémenter `ramasser(Joueur joueur)` et garantir qu'un objet n'est ramassé **qu'une seule fois** ;
- dessiner une pièce, une potion et une clé **sans un seul fichier image**, uniquement avec `Canvas` ;
- animer un flottement vertical infini avec `MoveEffect` et `EffectController(alternate: true, infinite: true)` ;
- désynchroniser des animations identiques avec `startDelay` pour éviter l'effet « métronome » ;
- enchaîner un pop, une gerbe de particules et une disparition avec `SequenceEffect` et `onComplete` ;
- soigner le joueur sans jamais dépasser `Constantes.pvJoueurMax`, grâce à `clamp` ;
- convertir un soin perdu en points plutôt que de le gaspiller ;
- incrémenter le compteur de clés du joueur et préparer l'ouverture de la porte du chapitre 39 ;
- écrire `ajouterScore(int points)` sur `DonjonGame` en respectant exactement la signature du contrat ;
- implémenter un **multiplicateur de combo** avec une fenêtre de temps qui se referme ;
- expliquer, schéma à l'appui, pourquoi le HUD vit dans le `viewport` et jamais dans le `world` ;
- construire un conteneur `Hud` et l'ajouter à `camera.viewport` ;
- placer des éléments par rapport aux bords avec `HudMarginComponent` et `EdgeInsets` ;
- connaître le piège de `margin.left == 0` dans `HudMarginComponent` ;
- construire une barre de vie avec plusieurs `RectangleComponent` empilés ;
- animer la barre avec une **barre fantôme** en retard, technique standard des jeux de combat ;
- afficher une jauge d'énergie et signaler visuellement qu'une attaque spéciale est prête ;
- afficher un score avec `TextComponent` et `TextPaint` ;
- faire **défiler** le score jusqu'à sa valeur cible avec une interpolation exponentielle ;
- éviter de reconstruire le `TextPainter` à chaque frame ;
- dessiner les vies restantes et les clés ramassées sans image ;
- comparer un HUD en composants Flame et un HUD en overlay Flutter, et choisir en connaissance de cause ;
- choisir entre synchronisation **pull** et **push** entre le HUD et l'état du jeu ;
- faire monter des textes de dégâts flottants dans le monde ;
- afficher un indicateur d'objectif qui change quand la clé est ramassée ;
- placer le HUD sur mobile sans jamais recouvrir le joystick ni le bouton d'attaque ;
- livrer un projet qui compile, se lance et se joue, sans aucun fichier image.

---

## 38.1 — Où on en est et ce qu'on ajoute

### Ce que le chapitre 37 vous a laissé entre les mains

Faisons le point, honnêtement, sur ce que votre projet sait faire à cet instant précis. C'est important : ce chapitre ne repart pas de zéro, il se branche sur du code existant, et il faut savoir exactement sur quoi.

```text
  donjon_de_dart/  ── ÉTAT À LA FIN DU CHAPITRE 37
  │
  └── lib/
      ├── main.dart                     runApp + GameWidget + overlayBuilderMap
      ├── donjon_game.dart              DonjonGame : FlameGame, score, vies, états
      ├── config/
      │   ├── constantes.dart           gravité, vitesses, PV max, zoom...
      │   └── palette.dart              les couleurs du donjon
      ├── core/
      │   ├── game_state.dart           enum GameState
      │   ├── entite.dart               PositionComponent de base
      │   └── sante.dart                mixin Sante (pv, pvMax, subirDegats, soigner)
      ├── composants/
      │   ├── joueur.dart               Joueur : déplacement, saut, attaque, PV
      │   ├── plateforme.dart           le sol et les murs
      │   ├── ennemi.dart               classe abstraite Ennemi
      │   ├── gobelin.dart              patrouille + charge
      │   ├── chauvesouris.dart         vol sinusoïdal + piqué
      │   └── projectile.dart           le projectile du joueur
      └── ecrans/
          └── menu_principal.dart       overlay du menu
```

Concrètement, si vous lancez le jeu maintenant :

| Ce qui marche déjà | Chapitre |
| --- | --- |
| Le menu principal s'affiche, le bouton « Jouer » démarre la partie | 35 |
| La caméra suit le héros dans une salle plus grande que l'écran | 35 |
| Le héros marche, saute, retombe, s'arrête aux murs | 36 |
| Le héros a une machine à états : `immobile`, `marche`, `saut`, `chute`, `attaque`, `touche`, `mort` | 36 |
| Les gobelins patrouillent et chargent quand le héros s'approche | 37 |
| Les chauves-souris volent en sinusoïde et piquent | 37 |
| Le héros perd des PV au contact, clignote pendant son invincibilité | 37 |
| Le héros tue un ennemi, l'ennemi explose en particules | 37 |
| `game.score` augmente quand un ennemi meurt | 37 |

Et voici, tout aussi honnêtement, ce qui **ne marche pas** :

| Ce qui manque | Conséquence pour le joueur |
| --- | --- |
| Aucun objet à ramasser | Le donjon est vide entre deux ennemis. Explorer ne rapporte rien. |
| Aucun moyen de se soigner | Une fois à 20 PV, la partie est perdue d'avance. |
| Aucune clé | La porte du chapitre 39 n'aura aucun moyen de s'ouvrir. |
| `score` existe mais n'est **affiché nulle part** | Le joueur ne sait pas s'il joue bien. |
| `pv` existe mais n'est **affiché nulle part** | Le joueur découvre sa mort sans l'avoir vue venir. |
| `vies` existe mais n'est **affiché nulle part** | Aucune notion de risque. |
| Aucun retour visuel sur les dégâts infligés | Frapper un gobelin donne la même sensation que frapper le vide. |

Le point commun de toute cette colonne de droite est le même : **le jeu sait des choses qu'il ne dit pas**. Un jeu qui ne communique pas son état est un jeu injuste. C'est exactement ce que ce chapitre corrige.

### Ce que ce chapitre ajoute

Deux moitiés, clairement séparées.

**Première moitié — les objets** (sections 38.2 à 38.13). On crée une hiérarchie de collectibles, on les fait ramasser, et on branche le score.

```text
  Collectible  (abstraite)          lib/composants/collectible.dart
      ├── Piece                     lib/composants/piece.dart      +10 points
      ├── Potion                    lib/composants/potion.dart     +25 PV
      └── Cle                       lib/composants/cle.dart        +1 clé
```

**Seconde moitié — le HUD** (sections 38.14 à 38.29). On construit l'interface qui affiche tout ça.

```text
  Hud  (conteneur)                  lib/hud/hud.dart
      ├── BarreDeVie                lib/hud/barre_de_vie.dart
      ├── BarreEnergie              lib/hud/barre_de_vie.dart
      ├── CompteurScore             lib/hud/compteur_score.dart
      ├── CompteurVies              lib/hud/compteur_score.dart
      ├── CompteurCles              lib/hud/compteur_score.dart
      └── IndicateurObjectif        lib/hud/compteur_score.dart
```

Plus un composant qui n'appartient à aucune des deux moitiés, parce qu'il vit dans le monde et pas à l'écran :

```text
  TexteFlottant                     lib/composants/texte_flottant.dart
```

### La règle qui n'a pas changé

Rappel de la règle de la PARTIE 2C, énoncée au chapitre 35 : **aucun fichier image**. Tout ce que vous allez voir à l'écran est dessiné avec `Canvas`, `RectangleComponent`, `CircleComponent` et `TextComponent`. Une pièce sera un disque doré qui tourne. Une potion sera une fiole rouge. Une clé sera un anneau avec des dents.

Ce n'est pas une contrainte artificielle. C'est une contrainte **pédagogique** : tant que vous n'avez pas d'assets, vous êtes obligé de comprendre la géométrie de ce que vous affichez. Le jour où vous brancherez de vrais sprites, il vous suffira de remplacer une méthode `render`. À chaque fois que ce point arrivera, une remarque « Avec de vrais assets » vous indiquera exactement quelle ligne changer.

---

## 38.2 — `lib/composants/collectible.dart` : la classe abstraite

### Pourquoi une classe abstraite plutôt que trois classes indépendantes

Posez-vous la question à l'envers : qu'est-ce qu'une pièce, une potion et une clé ont en commun ?

```text
  ┌────────────────────────┬────────┬────────┬────────┐
  │ Comportement           │ Pièce  │ Potion │  Clé   │
  ├────────────────────────┼────────┼────────┼────────┤
  │ posée dans le monde    │  oui   │  oui   │  oui   │
  │ flotte doucement       │  oui   │  oui   │  oui   │
  │ détecte le joueur      │  oui   │  oui   │  oui   │
  │ ne bloque pas le joueur│  oui   │  oui   │  oui   │
  │ ramassable UNE fois    │  oui   │  oui   │  oui   │
  │ fait un pop + particules  oui   │  oui   │  oui   │
  │ disparaît ensuite      │  oui   │  oui   │  oui   │
  ├────────────────────────┼────────┼────────┼────────┤
  │ EFFET du ramassage     │ score  │  soin  │ +1 clé │
  │ APPARENCE              │ disque │ fiole  │  clé   │
  └────────────────────────┴────────┴────────┴────────┘
```

Sept lignes identiques, deux lignes différentes. C'est la définition même d'une classe abstraite, telle que vous l'avez apprise au **chapitre 11** : la classe mère écrit tout ce qui est commun, et déclare **abstraits** les deux points qui varient.

Si vous écriviez trois classes indépendantes, vous copieriez sept comportements trois fois. Le jour où vous voudriez que tous les collectibles brillent quand le joueur s'approche, vous auriez trois endroits à modifier — et vous en oublieriez un.

### La signature imposée par le contrat

Le fichier `_SPEC-JEU-2C.md` fixe la signature. Elle n'est pas négociable, parce que les chapitres 39 et 40 s'appuient dessus :

```dart
abstract class Collectible extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  int get valeur;
  void ramasser(Joueur joueur);
}
```

Décortiquons chaque morceau, parce que chacun a une raison d'être.

| Morceau | Pourquoi |
| --- | --- |
| `abstract class` | On ne pose jamais un « collectible générique » dans le donjon. On pose une pièce, une potion ou une clé. La classe mère ne doit pas être instanciable. |
| `extends PositionComponent` | L'objet a une position et une taille dans le monde. C'est le minimum pour être vu et pour porter une hitbox (chapitre 28). |
| `with HasGameReference<DonjonGame>` | Le ramassage doit pouvoir appeler `game.ajouterScore(...)`. Le mixin donne la propriété `game`, typée. C'est le remplaçant de `HasGameRef`, déprécié (chapitre 31). |
| `with CollisionCallbacks` | Sans ce mixin, `onCollisionStart` ne sera jamais appelé (chapitre 32). |
| `int get valeur` | Un **getter abstrait**. Chaque sous-classe décide ce que « valeur » signifie pour elle : des points, des PV, un nombre de clés. |
| `void ramasser(Joueur joueur)` | La méthode abstraite qui contient l'effet. Elle reçoit le joueur, parce qu'un soin ou une clé s'appliquent au joueur, pas au jeu. |

> **Attention au sens de `valeur`.** `valeur` n'est pas « le nombre de points ». C'est **la grandeur caractéristique** du collectible, dans son unité à lui. Pour `Piece`, ce sont des points. Pour `Potion`, ce sont des PV. Pour `Cle`, c'est un nombre de clés. Cette imprécision est volontaire dans le contrat : elle permet aux trois classes de partager le même getter. En revanche, elle vous oblige à documenter chaque `valeur` dans sa sous-classe. Nous le ferons.

### Le squelette, avec ses champs internes

Ajoutons maintenant ce que le contrat ne dit pas, mais dont on a besoin.

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../donjon_game.dart';
import 'joueur.dart';

abstract class Collectible extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Collectible({Vector2? position, Vector2? size})
      : super(
          position: position,
          size: size ?? Vector2.all(14),
          anchor: Anchor.center,
        );

  /// Générateur commun à tous les collectibles : évite d'en créer un par objet.
  static final Random hasard = Random();

  /// Verrou anti double ramassage. Voir la section 38.4.
  bool _ramasse = false;
  bool get estRamasse => _ramasse;

  // ---- Ce que chaque sous-classe DOIT fournir -------------------------

  /// Grandeur caractéristique, dans l'unité de la sous-classe.
  int get valeur;

  /// Couleur dominante : sert aux particules et au texte flottant.
  Color get couleur;

  /// Nom lisible, utile pour le débogage et les futurs sous-titres.
  String get libelle;

  /// L'effet du ramassage. Appelée une seule fois, jamais deux.
  void ramasser(Joueur joueur);
}
```

Trois choix méritent une explication.

**`anchor: Anchor.center`.** Un collectible est un petit objet rond ou symétrique. Le positionner par son centre rend tous les calculs plus simples : le texte flottant part de son centre, les particules aussi, et le placement dans le niveau du chapitre 39 tombera pile au milieu d'une tuile. Rappel du chapitre 28 : l'ancre par défaut d'un `PositionComponent` est `Anchor.topLeft`.

**`static final Random hasard`.** Créer un `Random()` par objet est un gaspillage, et surtout une source de bugs subtils : deux `Random()` créés dans la même milliseconde peuvent produire la même séquence. Un seul générateur partagé par toute la hiérarchie règle la question. C'est le même raisonnement que pour les `Paint` statiques du chapitre 21.

**`Color get couleur` et `String get libelle`.** Ils ne sont pas dans le contrat, mais ils ne le contredisent pas : le contrat fixe un **minimum**, pas un maximum. Ces deux getters évitent des `if (this is Piece)` disgracieux dans le code des particules.

---

## 38.3 — Le capteur (trigger) : une hitbox passive (rappel chapitre 32)

### Deux familles de collisions

Au chapitre 32, section 32.9, vous avez rencontré l'énumération `CollisionType` :

```dart
enum CollisionType {
  active,   // teste contre les hitbox active ET passive
  passive,  // ne teste contre rien ; seules les actives la trouvent
  inactive, // aucun test
}
```

Et à la section 32.10, la règle de choix : **le décor est passif, ce qui bouge est actif**. La raison est de performance pure. Le moteur compare les hitbox deux à deux. Si toutes les hitbox sont actives, avec `n` objets il fait de l'ordre de `n²/2` comparaisons. En marquant passives toutes celles qui ne bougent pas, on divise ce nombre par un facteur énorme dans un donjon rempli de murs et de pièces.

```text
  40 collectibles + 1 joueur, TOUS EN ACTIF
     paires testées : 41 × 40 / 2 = 820 par frame
     dont 780 paires « pièce contre pièce » parfaitement inutiles

  40 collectibles PASSIFS + 1 joueur ACTIF
     paires testées : 40
     soit 20 fois moins de travail, pour le même résultat
```

### Mais le vrai motif n'est pas la performance

Il y a une raison plus profonde, et c'est celle qu'il faut retenir. Un collectible n'est pas un obstacle : **il ne doit rien bloquer**.

```text
  UN MUR                            UN CAPTEUR (trigger)

   ┌──────┐                          ┌╌╌╌╌╌╌┐
   │██████│  <── le joueur           ╎      ╎  <── le joueur
   │██████│      est REPOUSSÉ        ╎      ╎      TRAVERSE
   └──────┘                          └╌╌╌╌╌╌┘      et déclenche

   on lit la collision               on lit la collision
   ET on corrige la position         et on ne touche à RIEN
```

La différence entre un mur et un capteur n'est pas dans la hitbox : elle est dans **ce que vous faites** de `onCollisionStart`. Un mur corrige la position du joueur (chapitre 32, section 32.19). Un capteur se contente de réagir. Marquer la hitbox `passive` est ce qui rend cette intention explicite et lisible par la personne qui relira votre code dans six mois.

### La hitbox du collectible

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  // Capteur : passif. Il ne cherche personne, c'est le joueur (actif)
  // qui le trouve. Et il ne bloque jamais rien.
  add(RectangleHitbox(collisionType: CollisionType.passive));
}
```

`RectangleHitbox()` sans argument remplit automatiquement le parent (fait confirmé au chapitre 32, section 32.5) : la hitbox aura donc exactement la taille du collectible.

Une variante utile, si vous trouvez le ramassage trop sévère :

```dart
// Une hitbox 30 % plus grande que l'objet : le ramassage devient
// « généreux », le joueur n'a pas besoin de viser au pixel près.
add(
  RectangleHitbox.relative(
    Vector2.all(1.3),
    parentSize: size,
    anchor: Anchor.center,
    collisionType: CollisionType.passive,
  ),
);
```

> **Règle de game design.** Une hitbox de ramassage doit toujours être **plus généreuse** que le visuel, et une hitbox de dégâts toujours **plus petite**. Le joueur doit avoir l'impression de ramasser facilement et d'esquiver de justesse. Cette asymétrie est invisible et pourtant elle change tout au ressenti. Vous l'aviez déjà appliquée aux ennemis au chapitre 37.

### Vérifier que le joueur est bien actif

Un capteur passif ne sert à rien si personne n'est actif en face. Vérifiez dans `joueur.dart` (chapitre 36) que la hitbox du joueur n'a pas été marquée passive :

```dart
// lib/composants/joueur.dart — extrait du chapitre 36
@override
Future<void> onLoad() async {
  await super.onLoad();
  add(RectangleHitbox());   // active par défaut : c'est bien ce qu'on veut
}
```

**Résultat attendu si vous vous trompez :**

```text
Le joueur traverse les pièces sans rien déclencher.
Aucune erreur, aucun message : juste un jeu qui ne réagit pas.
Cause : deux hitbox passives ne se voient jamais.
```

C'est le bug numéro un de cette section. Il ne produit aucune exception, ce qui le rend particulièrement pénible à diagnostiquer. Le réflexe : activer `debugMode = true` sur le jeu (chapitre 32, section 32.8) et regarder si les deux hitbox sont bien dessinées.

---

## 38.4 — `ramasser(Joueur joueur)`

### Qui appelle qui

`ramasser` n'est jamais appelée directement par vous. Elle est appelée par `onCollisionStart`, dans la classe mère, une fois et une seule.

```text
  frame N
    │
    ├─ le moteur détecte : hitbox(Joueur) ∩ hitbox(Piece)
    │
    ├─ Flame appelle Piece.onCollisionStart(points, joueur)
    │       (hérité de Collectible — vous ne le redéfinissez pas)
    │
    ├─ Collectible.onCollisionStart :
    │       1. l'objet est-il déjà ramassé ?  oui -> on sort
    │       2. l'autre est-il un Joueur ?     non -> on sort
    │       3. on VERROUILLE (_ramasse = true)
    │       4. on appelle ramasser(joueur)   <── VOTRE code de sous-classe
    │       5. on joue l'effet visuel et on programme le retrait
    │
    └─ frame N+1 … N+15 : le pop se joue, puis l'objet est retiré
```

### Le code de la classe mère

```dart
@override
void onCollisionStart(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  // @mustCallSuper : sans cette ligne, activeCollisions n'est plus tenu
  // à jour (chapitre 32, section 32.12).
  super.onCollisionStart(intersectionPoints, other);

  // Verrou : un objet ne se ramasse qu'une fois.
  if (_ramasse) {
    return;
  }

  // Un gobelin qui passe sur une pièce ne la ramasse pas.
  if (other is! Joueur) {
    return;
  }

  _ramasse = true;      // on verrouille AVANT d'agir
  ramasser(other);      // l'effet propre à la sous-classe
  _jouerRamassage();    // le pop, les particules, la disparition
}
```

Quatre points, tous importants.

**`super.onCollisionStart(...)` en premier.** Le mixin `CollisionCallbacks` annote ses trois callbacks `@mustCallSuper`. Oublier l'appel casse `activeCollisions` et `isColliding` pour ce composant. Le linter vous le signalera, mais seulement si vous l'avez activé.

**Le verrou avant l'action.** L'ordre `_ramasse = true;` puis `ramasser(other);` n'est pas cosmétique. Si `ramasser` déclenchait, directement ou indirectement, une seconde collision, le verrou serait déjà posé. Poser le verrou après l'action laisserait une fenêtre.

**`other is! Joueur`.** C'est le patron de reconnaissance de type du chapitre 32, section 32.16, qui repose lui-même sur la promotion de type du **chapitre 10**. Après ce test, `other` est promu en `Joueur` dans la suite du bloc : c'est pour cela que `ramasser(other)` compile sans transtypage.

**`ramasser` avant l'effet visuel.** L'effet visuel a besoin de savoir ce qui s'est passé — par exemple, la potion n'affiche pas le même texte selon que le soin a servi ou non. En appelant l'effet en dernier, la sous-classe a déjà pu mettre à jour ce qu'elle voulait.

### Pourquoi le verrou est indispensable

Ce point mérite qu'on s'y arrête, parce qu'il est contre-intuitif.

`onCollisionStart` est censée être appelée **une seule fois** par paire de hitbox, au début du chevauchement. En théorie, le verrou est donc inutile. En pratique, il l'est pour trois raisons :

1. **Deux hitbox sur le joueur.** Si votre joueur porte une hitbox de corps et une hitbox d'attaque, le collectible reçoit **deux** `onCollisionStart`, une par hitbox. Sans verrou, la pièce vaut 20 points au lieu de 10.
2. **Le retrait n'est pas immédiat.** `removeFromParent()` programme un retrait, il ne l'exécute pas sur-le-champ (chapitre 28). L'objet reste dans l'arbre jusqu'à la fin de la frame. Notre pop dure en plus 0,26 seconde, soit une quinzaine de frames pendant lesquelles l'objet existe encore.
3. **La sortie et le retour.** Un joueur qui traverse la hitbox, en sort et y revient déclenche un nouveau `onCollisionStart`. Sans verrou, il ramasse la même pièce deux fois.

**Résultat sans verrou, avec un joueur à deux hitbox :**

```text
Score après avoir ramassé 5 pièces à 10 points : 100
Score attendu : 50
```

Ce bug est classique, silencieux, et il ruine l'équilibrage d'un jeu.

### Neutraliser la hitbox tout de suite

Le verrou booléen suffit, mais il fait quand même travailler le moteur pour rien pendant la durée du pop. Autant couper à la source :

```dart
void _desactiverLeCapteur() {
  for (final hitbox in children.whereType<ShapeHitbox>()) {
    hitbox.collisionType = CollisionType.inactive;
  }
}
```

`ShapeHitbox` est la classe mère de `RectangleHitbox`, `CircleHitbox` et `PolygonHitbox`. La parcourir par `whereType` évite d'avoir à mémoriser la hitbox dans un champ. `CollisionType.inactive` retire complètement la hitbox des tests (chapitre 32, section 32.9).

> **Ceinture et bretelles.** Nous gardons **les deux** : le booléen `_ramasse` et la désactivation de la hitbox. Le booléen protège contre les appels de la frame en cours, déjà en file d'attente ; la désactivation protège contre les frames suivantes. Dans un jeu, la robustesse d'un compteur de score vaut largement trois lignes de code.

---

## 38.5 — `lib/composants/piece.dart`

### La sous-classe la plus simple du chapitre

Une pièce fait exactement une chose : elle donne des points. C'est le collectible modèle, celui sur lequel on va tout expliquer.

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

class Piece extends Collectible {
  Piece({Vector2? position})
      : super(position: position, size: Vector2.all(14));

  /// Points rapportés par une pièce, AVANT multiplicateur de combo.
  static const int points = 10;

  /// Énergie rendue au joueur : ramasser, c'est aussi recharger.
  static const double energie = 6;

  @override
  int get valeur => points;

  @override
  Color get couleur => const Color(0xFFE8B04B);

  @override
  String get libelle => 'Pièce';

  @override
  void ramasser(Joueur joueur) {
    // On calcule le gain AVANT d'appeler ajouterScore : cette méthode
    // fait monter le combo, donc le multiplicateur, donc le gain suivant.
    final int gain = valeur * game.multiplicateur;

    game.ajouterScore(valeur);
    game.piecesRamassees++;
    joueur.gagnerEnergie(energie);
    game.afficherTexteFlottant(position, '+$gain', couleur);
  }
}
```

Cinq lignes de logique, et déjà trois décisions de conception.

**`Vector2.all(14)`.** La tuile du jeu fait 32 pixels (`Constantes.tailleTuile`). Une pièce de 14 pixels occupe donc un peu moins de la moitié d'une tuile : elle se lit bien, sans écraser le décor. Avec le zoom de caméra de 2 fixé au chapitre 35, elle occupera 28 pixels à l'écran.

**`static const int points = 10`.** La valeur est nommée et publique. Le chapitre 39 en aura besoin pour calculer le score maximum théorique d'un niveau, et le chapitre 40 pour la sauvegarde du meilleur score. Une constante nommée se refactorise ; un `10` écrit en dur se cherche à la main dans tout le projet.

**Le gain calculé avant `ajouterScore`.** C'est le point subtil. Le contrat impose `void ajouterScore(int points)` : la méthode ne **renvoie rien**. Or elle incrémente le combo, ce qui change le multiplicateur. Si vous lisiez `game.multiplicateur` après l'appel, vous afficheriez un `+20` pour un gain réel de `+10`. On lit donc le multiplicateur avant, on calcule, puis on appelle.

**Résultat, trois pièces ramassées coup sur coup (combo initial à 0) :**

```text
Pièce 1 : multiplicateur ×1  ->  texte « +10 »  ->  score 10
Pièce 2 : multiplicateur ×1  ->  texte « +10 »  ->  score 20
Pièce 3 : multiplicateur ×1  ->  texte « +10 »  ->  score 30
Pièce 4 : multiplicateur ×2  ->  texte « +20 »  ->  score 50
```

Le multiplicateur passe à ×2 à la quatrième pièce, parce qu'il monte d'un cran tous les trois ramassages. Nous reviendrons sur ce calcul en 38.13.

### Dessiner une pièce sans image

Une pièce dessinée comme un simple disque jaune est terne. Le truc classique, utilisé depuis les années 1980, est de simuler la rotation en faisant **osciller la largeur** de l'ellipse. Le disque devient une tranche, puis un disque, puis une tranche : le cerveau interprète une rotation en trois dimensions.

```text
   t = 0.0      t = 0.4      t = 0.8      t = 1.2      t = 1.6

    ╭───╮        ╭─╮          │            ╭─╮         ╭───╮
   │  ●  │      │ ● │         ▮           │ ● │       │  ●  │
    ╰───╯        ╰─╯          │            ╰─╯         ╰───╯

   largeur      largeur      largeur      largeur     largeur
    100 %        60 %         15 %         60 %        100 %

   largeur = |cos(t × vitesse)|, bornée à 15 % minimum
```

Voici le code de rendu, à ajouter dans `Piece` :

```dart
  static final Paint _corps = Paint()..color = const Color(0xFFE8B04B);
  static final Paint _reflet = Paint()..color = const Color(0xFFFFE9A8);
  static final Paint _contour = Paint()
    ..color = const Color(0xFF8A5A16)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 1.5;

  /// Vitesse de la fausse rotation, en radians par seconde.
  static const double vitesseRotation = 3.2;

  double _phase = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // Décalage initial aléatoire : deux pièces côte à côte ne tournent
    // pas en même temps. Sans cela, l'ensemble ressemble à un métronome.
    _phase = Collectible.hasard.nextDouble() * pi;
  }

  @override
  void update(double dt) {
    super.update(dt);
    _phase += dt * vitesseRotation;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final Offset centre = Offset(size.x / 2, size.y / 2);

    // |cos| oscille entre 0 et 1 ; on l'empêche de descendre sous 0,15
    // pour que la pièce ne disparaisse jamais complètement.
    final double facteur = cos(_phase).abs().clamp(0.15, 1.0);
    final double demiLargeur = (size.x / 2) * facteur;

    final Rect ovale = Rect.fromCenter(
      center: centre,
      width: demiLargeur * 2,
      height: size.y,
    );

    canvas.drawOval(ovale, _corps);
    canvas.drawOval(ovale, _contour);

    // Un petit reflet, décalé vers le haut à gauche : c'est lui qui
    // fait passer le disque plat pour un objet en volume.
    if (facteur > 0.35) {
      canvas.drawOval(
        Rect.fromCenter(
          center: centre.translate(-demiLargeur * 0.25, -size.y * 0.18),
          width: demiLargeur * 0.7,
          height: size.y * 0.34,
        ),
        _reflet,
      );
    }
  }
```

Trois remarques de fond.

**Les `Paint` sont `static final`.** Créer un `Paint` par frame et par pièce, avec quarante pièces à l'écran et soixante frames par seconde, c'est 2 400 objets alloués par seconde pour rien. Vous aviez déjà rencontré cette règle au chapitre 21, section 21.9.

**Le canvas est déjà transformé.** À l'intérieur de `render`, l'origine `(0, 0)` est le coin haut-gauche du composant, quelle que soit sa position dans le monde et quelle que soit l'ancre. C'est pour cela qu'on écrit `size.x / 2` et non `position.x`. Rappel du chapitre 28.

**`super.render(canvas)` d'abord.** Il dessine les enfants… non : dans Flame, `super.render` de `PositionComponent` ne dessine rien par lui-même, les enfants sont dessinés par `renderTree` après. L'appeler reste la bonne habitude, et c'est indispensable si un jour vous insérez une classe intermédiaire qui dessine quelque chose.

> **Avec de vrais assets.** Remplacez toute la méthode `render` par un `SpriteAnimationComponent` enfant, chargé depuis une planche de huit images (`coins.png`), exactement comme l'exemple officiel du chapitre 29. La logique de `Piece` — `valeur`, `ramasser` — ne change pas d'une ligne. C'est tout l'intérêt d'avoir séparé le comportement du rendu.

---

## 38.6 — L'animation de flottement (rappel chapitre 33)

### Pourquoi faire flotter un objet

Un objet immobile posé sur un sol se confond avec le décor. Un objet qui bouge, même de trois pixels, attire l'œil. C'est un principe d'ergonomie visuelle : **le mouvement capte l'attention avant la couleur et avant la forme**.

Le flottement fait aussi passer un message implicite au joueur : « cet objet n'appartient pas au décor, il est interactif ». Aucun mur ne flotte. Aucune plateforme ne flotte. Ce qui flotte se ramasse.

### L'effet, en une expression

Au chapitre 33, section 33.3, vous avez vu `MoveEffect.by`. Section 33.11, vous avez vu `infinite`. Section 33.12, `alternate`. On assemble les trois :

```dart
MoveEffect.by(
  Vector2(0, -amplitudeFlottement),          // 4 pixels vers le HAUT
  EffectController(
    duration: dureeFlottement,               // 1,1 s pour monter
    alternate: true,                         // puis 1,1 s pour redescendre
    infinite: true,                          // et on recommence, sans fin
    curve: Curves.easeInOut,                 // ralenti aux deux extrémités
  ),
);
```

```text
  COURSE VERTICALE DE L'OBJET, AVEC alternate + infinite

  y-4  ┈┈┈╭──────╮┈┈┈┈┈┈┈┈┈┈╭──────╮┈┈┈┈┈┈┈┈┈┈╭──────╮
         ╱        ╲        ╱        ╲        ╱        ╲
  y    ─╯          ╰──────╯          ╰──────╯          ╰──
       0         1,1 s          2,2 s          3,3 s

  easeInOut : la vitesse est nulle en haut et en bas,
  maximale au milieu. C'est le mouvement d'un objet qui flotte,
  pas celui d'un ascenseur.
```

Le signe négatif sur `y` est la conséquence directe de l'axe `y` descendant du canvas, vu au chapitre 21 : pour monter, on soustrait.

### Le piège du métronome

Posez quinze pièces alignées avec exactement le même effet. Résultat : **elles montent et descendent toutes en même temps**. L'ensemble ne ressemble plus à quinze objets qui flottent, mais à une seule barre qui oscille. L'illusion est détruite.

```text
  SANS DÉCALAGE — 15 pièces, même effet, même instant de départ

    ● ● ● ● ● ● ● ● ● ● ● ● ● ● ●      <- toutes en haut
    ─────────────────────────────
    ● ● ● ● ● ● ● ● ● ● ● ● ● ● ●      <- toutes en bas

  AVEC startDelay ALÉATOIRE

    ●   ●     ●   ●       ●   ●
      ●   ●     ●   ●   ●     ●  ●     <- une vague désordonnée
    ─────────────────────────────
```

La solution tient en un paramètre, vu au chapitre 33, section 33.15 : `startDelay`. On donne à chaque objet un retard au démarrage tiré au hasard entre 0 et la durée d'un aller.

```dart
EffectController(
  duration: dureeFlottement,
  alternate: true,
  infinite: true,
  curve: Curves.easeInOut,
  startDelay: hasard.nextDouble() * dureeFlottement,
)
```

> **Règle générale.** Chaque fois que vous répétez une animation identique sur plusieurs objets, désynchronisez-la. Un `startDelay` aléatoire, ou une phase initiale aléatoire comme dans le `render` de la pièce, coûte une ligne et transforme le rendu. C'est valable pour les torches, les herbes qui bougent, les nuages, les vagues.

### Le code complet de `onLoad`

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  // 1. Le capteur (section 38.3)
  add(RectangleHitbox(collisionType: CollisionType.passive));

  // 2. Le flottement, désynchronisé
  add(
    MoveEffect.by(
      Vector2(0, -amplitudeFlottement),
      EffectController(
        duration: dureeFlottement,
        alternate: true,
        infinite: true,
        curve: Curves.easeInOut,
        startDelay: hasard.nextDouble() * dureeFlottement,
      ),
    ),
  );
}
```

Les deux paramètres du flottement sont des getters redéfinissables, ce qui permet à chaque sous-classe d'avoir son propre rythme :

```dart
  /// Amplitude du flottement, en pixels. Redéfinissable.
  double get amplitudeFlottement => 4.0;

  /// Durée d'un aller (montée), en secondes. Redéfinissable.
  double get dureeFlottement => 1.1;
```

Une clé, objet rare et important, flottera plus haut et plus lentement qu'une pièce :

```dart
// dans Cle
@override
double get amplitudeFlottement => 6.0;

@override
double get dureeFlottement => 1.5;
```

C'est un détail. Les détails de ce genre, mis bout à bout, sont exactement ce qui sépare un prototype d'un jeu.

---

## 38.7 — L'effet de ramassage : pop, particules, disparition

### Trois signaux pour un seul événement

Quand le joueur ramasse un objet, le jeu doit le confirmer immédiatement. Un objet qui disparaît sans transition laisse un doute : « est-ce que je l'ai eu ? » Un objet qui explose visuellement ne laisse aucun doute.

On empile trois signaux, chacun jouant sur un canal différent :

| Signal | Canal perceptif | Durée | Outil Flame |
| --- | --- | --- | --- |
| Le **pop** : l'objet grossit puis se réduit à rien | forme | 0,26 s | `SequenceEffect` de deux `ScaleEffect` |
| Les **particules** : une gerbe de la couleur de l'objet | mouvement | 0,5 s | `ParticleSystemComponent` |
| Le **texte flottant** : `+10` qui monte | symbole | 0,9 s | `TexteFlottant` (section 38.27) |

Trois canaux, trois durées différentes. C'est important : si les trois se terminaient en même temps, l'effet paraîtrait mécanique. En les échelonnant, on obtient une petite cascade.

```text
  CHRONOLOGIE DU RAMASSAGE, EN SECONDES

  0,00 ─┬─ collision détectée, verrou posé, ramasser() appelée
        │
  0,00 ─┼─ pop : l'objet grossit ×1,6 ────────╮
        │  particules : 10 disques partent    │
        │  texte « +10 » commence à monter    │
        │                                     │
  0,12 ─┼─ pop : l'objet se réduit à 0,1 ◄────╯
        │
  0,26 ─┼─ removeFromParent() — l'objet quitte l'arbre
        │
  0,50 ─┼─ les particules ont fini, leur composant s'auto-retire
        │
  0,90 ─┴─ le texte a fini de s'effacer, il s'auto-retire
```

### Le pop

```dart
add(
  SequenceEffect(
    [
      // 1. grossir vite, avec un easeOut : départ sec, arrivée douce
      ScaleEffect.to(
        Vector2.all(1.6),
        EffectController(duration: 0.12, curve: Curves.easeOut),
      ),
      // 2. se réduire à presque rien, avec un easeIn : départ doux,
      //    arrivée sèche. L'objet « aspiré ».
      ScaleEffect.to(
        Vector2.all(0.1),
        EffectController(duration: 0.14, curve: Curves.easeIn),
      ),
    ],
    onComplete: removeFromParent,
  ),
);
```

Deux détails techniques.

**`Vector2.all(0.1)` et non `Vector2.zero()`.** Une échelle strictement nulle rend la matrice de transformation non inversible, ce qui peut produire des `NaN` dans certains calculs géométriques. Une échelle de 0,1 est visuellement indiscernable de zéro sur un objet de 14 pixels — il reste 1,4 pixel — et elle est mathématiquement saine. Prenez cette habitude partout.

**`onComplete: removeFromParent`.** On passe la **méthode** `removeFromParent`, sans parenthèses : c'est un tear-off, la notation vue au chapitre 7. Elle est équivalente à `onComplete: () => removeFromParent()` mais plus courte. `SequenceEffect` accepte `onComplete` par héritage de la classe `Effect` (chapitre 33, section 33.18).

**Le choix des courbes.** `easeOut` puis `easeIn` : c'est le contraire de l'intuition. On veut que le grossissement soit **immédiat** — donc rapide au départ, `easeOut` — et que la disparition **s'accélère** — donc lente au départ, `easeIn`. Le résultat est un « pop » sec suivi d'une aspiration. Inversez les deux courbes et l'effet devient mou : essayez, c'est instructif.

### Les particules

```dart
parent?.add(
  ParticleSystemComponent(
    position: position.clone(),
    particle: Particle.generate(
      count: 10,
      lifespan: 0.5,
      generator: (i) => AcceleratedParticle(
        speed: Vector2(
          hasard.nextDouble() * 120 - 60,     // entre -60 et +60
          hasard.nextDouble() * -110 - 30,    // entre -140 et -30 (vers le haut)
        ),
        acceleration: Vector2(0, 320),        // la gravité les fait retomber
        child: CircleParticle(
          radius: 1.5 + hasard.nextDouble() * 1.5,
          paint: Paint()..color = couleur,
        ),
      ),
    ),
  ),
);
```

Le point crucial est le premier mot : **`parent?.add`**, et non `add`.

```text
  SI ON ÉCRIT  add(ParticleSystemComponent(...))

     Collectible
        └── ParticleSystemComponent      <- enfant de l'objet

     L'objet est retiré à t = 0,26 s.
     Ses enfants partent avec lui.
     Les particules disparaissent AVANT la fin de leur vie (0,5 s).
     Pire : elles subissent aussi le ScaleEffect du pop, donc
     elles rétrécissent en même temps que l'objet.

  SI ON ÉCRIT  parent?.add(ParticleSystemComponent(...))

     World
        ├── Collectible                  <- retiré à 0,26 s
        └── ParticleSystemComponent      <- survit jusqu'à 0,5 s
```

C'est la règle générale des effets de mort, déjà rencontrée au chapitre 37 pour l'explosion des ennemis : **un effet qui doit survivre à un objet ne doit pas être son enfant**.

`position.clone()` est également obligatoire. `Vector2` est **mutable** (piège rappelé dans la fiche de référence Flame). Écrire `position: position` partagerait la même instance : le `ScaleEffect` du pop, ou un futur effet, déplacerait les particules avec l'objet. `clone()` fige les coordonnées à l'instant du ramassage.

`AcceleratedParticle` et `Particle.generate` viennent tout droit du chapitre 33, sections 33.24 et 33.26. La gerbe part vers le haut et retombe : c'est le mouvement du projectile du chapitre 23, appliqué à dix petits disques.

### La méthode complète

```dart
void _jouerRamassage() {
  // 1. Le capteur ne doit plus répondre.
  for (final hitbox in children.whereType<ShapeHitbox>()) {
    hitbox.collisionType = CollisionType.inactive;
  }

  // 2. On coupe le flottement : sinon il continue pendant le pop
  //    et l'objet part en biais.
  for (final effet in children.whereType<MoveEffect>()) {
    effet.removeFromParent();
  }

  // 3. Les particules, ATTACHÉES AU PARENT pour survivre à l'objet.
  parent?.add(
    ParticleSystemComponent(
      position: position.clone(),
      particle: Particle.generate(
        count: 10,
        lifespan: 0.5,
        generator: (i) => AcceleratedParticle(
          speed: Vector2(
            hasard.nextDouble() * 120 - 60,
            hasard.nextDouble() * -110 - 30,
          ),
          acceleration: Vector2(0, 320),
          child: CircleParticle(
            radius: 1.5 + hasard.nextDouble() * 1.5,
            paint: Paint()..color = couleur,
          ),
        ),
      ),
    ),
  );

  // 4. Le pop, puis le retrait.
  add(
    SequenceEffect(
      [
        ScaleEffect.to(
          Vector2.all(1.6),
          EffectController(duration: 0.12, curve: Curves.easeOut),
        ),
        ScaleEffect.to(
          Vector2.all(0.1),
          EffectController(duration: 0.14, curve: Curves.easeIn),
        ),
      ],
      onComplete: removeFromParent,
    ),
  );
}
```

L'étape 2 est facile à oublier et son symptôme est déroutant : la pièce, pendant qu'elle « pope », continue de monter ou de descendre, et le pop paraît décentré. Un `MoveEffect` infini ne s'arrête jamais tout seul ; il faut le retirer.

**Résultat à l'écran :**

```text
Le joueur touche une pièce.
La pièce grossit d'un coup, dix éclats dorés jaillissent vers le haut
et retombent, un « +10 » doré monte et s'efface.
La pièce a disparu. Le compteur de score du HUD grimpe jusqu'à sa
nouvelle valeur en un quart de seconde.
```

---

## 38.8 — `lib/composants/potion.dart` : soigner le joueur

### Un collectible dont l'effet peut être inutile

La pièce est simple : elle donne toujours ses points. La potion pose un problème que la pièce n'a pas. **Que se passe-t-il si le joueur est déjà à 100 PV ?**

Trois réponses possibles, et il faut choisir consciemment :

| Politique | Comportement | Effet ressenti |
| --- | --- | --- |
| **Gaspillage** | La potion est ramassée, le soin est perdu | Frustrant. Le joueur a l'impression de s'être fait voler. |
| **Refus** | La potion n'est pas ramassable tant que les PV sont pleins | Correct, mais demande une hitbox conditionnelle et déroute : « pourquoi je ne peux pas la prendre ? » |
| **Conversion** | La potion est ramassée, le soin inutile devient des points | Toujours gratifiant. Aucun ramassage n'est jamais « perdu ». |

Nous choisissons la **conversion**. C'est la politique la plus généreuse et la plus lisible, et c'est celle qu'emploient la plupart des jeux d'action modernes.

### Le code

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

class Potion extends Collectible {
  Potion({Vector2? position})
      : super(position: position, size: Vector2(14, 18));

  /// Points de vie rendus. C'est l'unité de `valeur` pour une potion.
  static const int soin = 25;

  /// Points donnés quand le soin ne sert à rien (PV déjà au maximum).
  static const int pointsSiInutile = 5;

  @override
  int get valeur => soin;

  @override
  Color get couleur => const Color(0xFFE04F5F);

  @override
  String get libelle => 'Potion';

  @override
  void ramasser(Joueur joueur) {
    final double avant = joueur.pv;
    joueur.soigner(valeur.toDouble());
    final double rendu = joueur.pv - avant;

    if (rendu > 0) {
      game.afficherTexteFlottant(
        position,
        '+${rendu.round()} PV',
        couleur,
      );
    } else {
      // PV déjà au maximum : on convertit plutôt que de gaspiller.
      final int gain = pointsSiInutile * game.multiplicateur;
      game.ajouterScore(pointsSiInutile);
      game.afficherTexteFlottant(position, '+$gain', const Color(0xFFE8B04B));
    }
  }
}
```

Le patron « mesurer avant, mesurer après, comparer » est très général :

```text
  avant  = 88 PV
  soigner(25)
  après  = 100 PV      (bloqué au maximum)
  rendu  = 12 PV       <- ce qui a RÉELLEMENT été rendu

  On affiche « +12 PV », pas « +25 PV ».
  Le joueur voit la vérité, pas la promesse.
```

Afficher `+25` alors que le joueur n'a gagné que 12 PV serait un mensonge que la barre de vie démentirait immédiatement. Le joueur ne saurait pas lequel des deux croire, et c'est exactement le genre de petite incohérence qui érode la confiance dans un jeu.

### Dessiner une fiole

```dart
  static final Paint _verre = Paint()..color = const Color(0x66FFFFFF);
  static final Paint _liquide = Paint()..color = const Color(0xFFE04F5F);
  static final Paint _bouchon = Paint()..color = const Color(0xFF6D4C33);
  static final Paint _brillance = Paint()..color = const Color(0x99FFFFFF);

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final double l = size.x;
    final double h = size.y;

    // Le corps de la fiole : un rectangle aux coins arrondis.
    final RRect corps = RRect.fromRectAndRadius(
      Rect.fromLTWH(0, h * 0.30, l, h * 0.70),
      Radius.circular(l * 0.35),
    );
    canvas.drawRRect(corps, _verre);

    // Le liquide : les deux tiers inférieurs du corps.
    canvas.save();
    canvas.clipRRect(corps);
    canvas.drawRect(Rect.fromLTWH(0, h * 0.50, l, h * 0.50), _liquide);
    canvas.restore();

    // Le goulot, puis le bouchon.
    canvas.drawRect(
      Rect.fromLTWH(l * 0.32, h * 0.12, l * 0.36, h * 0.22),
      _verre,
    );
    canvas.drawRect(
      Rect.fromLTWH(l * 0.26, 0, l * 0.48, h * 0.14),
      _bouchon,
    );

    // Une brillance verticale sur la gauche : le verre.
    canvas.drawRect(
      Rect.fromLTWH(l * 0.18, h * 0.42, l * 0.10, h * 0.40),
      _brillance,
    );
  }
```

`canvas.save()` / `canvas.clipRRect(...)` / `canvas.restore()` est le trio de découpe vu au chapitre 21, section 21.18 : on limite le dessin du liquide à l'intérieur de la forme du corps, ce qui donne un liquide aux bords arrondis sans avoir à recalculer la géométrie. **Ne jamais oublier `restore()`** : un `save()` sans `restore()` fait fuir l'état du canvas sur tous les composants dessinés ensuite.

---

## 38.9 — Ne pas dépasser les PV max

### La méthode `soigner` du joueur

Le contrat du chapitre 37 donne au mixin `Sante` une méthode `soigner(double points)`, mais **`Joueur` n'utilise pas ce mixin** : le contrat lui donne un champ `double pv` en propre. Il faut donc ajouter la méthode à `Joueur`. C'est un ajout, pas une redéfinition : le fichier `joueur.dart` du chapitre 36 se complète.

```dart
// lib/composants/joueur.dart — AJOUT DU CHAPITRE 38

/// Rend des points de vie, sans jamais dépasser le maximum.
/// Un joueur mort ne se soigne pas.
void soigner(double points) {
  if (points <= 0 || pv <= 0) {
    return;
  }
  pv = (pv + points).clamp(0.0, Constantes.pvJoueurMax);
}
```

Trois gardes en trois lignes.

**`points <= 0`.** Une méthode `soigner` qui reçoit un nombre négatif ferait des dégâts. Ce serait un chemin détourné pour blesser le joueur, hors de toute logique d'invincibilité. On refuse.

**`pv <= 0`.** Un joueur mort ne se soigne pas. Sans cette garde, une potion ramassée dans la même frame que la mort ressusciterait le héros — bug spectaculaire et parfaitement reproductible.

**`clamp(0.0, Constantes.pvJoueurMax)`.** C'est le cœur de la section.

### Pourquoi `clamp` et pas un `if`

Les deux écritures suivantes donnent le même résultat :

```dart
// Version 1 : le if
pv = pv + points;
if (pv > Constantes.pvJoueurMax) {
  pv = Constantes.pvJoueurMax;
}

// Version 2 : le clamp
pv = (pv + points).clamp(0.0, Constantes.pvJoueurMax);
```

La version `clamp` est préférable pour trois raisons :

1. **Elle borne des deux côtés.** Le `if` ne protège que le haut. `clamp` protège aussi contre un `pv` négatif, ce qui simplifie `subirDegats`.
2. **Elle est une expression.** Elle s'utilise dans une affectation, un retour de fonction, un `..` en cascade.
3. **Elle exprime l'intention.** « Cette valeur vit entre 0 et 100 » se lit d'un coup d'œil, alors qu'un `if` isolé peut être déplacé ou oublié lors d'un refactoring.

> **Piège de type.** Sur un `double`, `clamp` renvoie un `num` en Dart. Ici, comme les deux bornes sont des `double` (`0.0` et `Constantes.pvJoueurMax` qui vaut `100.0`), l'inférence donne bien un `double` et l'affectation compile. Si vous écriviez `clamp(0, 100)` avec des littéraux entiers, vous obtiendriez une erreur de type sur `pv = ...`. Écrivez toujours vos bornes avec le même type que la valeur : `0.0` et non `0`.

### Le même raisonnement pour l'énergie

L'énergie du joueur, qui alimentera la jauge d'attaque spéciale de la section 38.20, se borne exactement de la même manière :

```dart
// lib/composants/joueur.dart — AJOUT DU CHAPITRE 38

/// Énergie courante, de 0 à [energieMax].
double energie = 0;

/// Plafond de la jauge d'énergie.
static const double energieMax = 100;

/// L'attaque spéciale est prête quand la jauge est pleine.
bool get attaqueSpecialePrete => energie >= energieMax;

/// Recharge la jauge. Bornée, comme les PV.
void gagnerEnergie(double points) {
  if (points <= 0) {
    return;
  }
  energie = (energie + points).clamp(0.0, energieMax);
}

/// Vide la jauge après une attaque spéciale.
void consommerEnergie() {
  energie = 0;
}
```

**Résultat, en soignant un joueur à 88 PV :**

```text
pv avant   : 88.0
soigner(25)
pv après   : 100.0        (et non 113.0)
rendu réel : 12.0         -> le texte flottant affiche « +12 PV »
```

**Résultat, en soignant un joueur déjà plein :**

```text
pv avant   : 100.0
soigner(25)
pv après   : 100.0
rendu réel : 0.0          -> conversion : « +5 » de score
```

### La table des grandeurs bornées du jeu

Prenez l'habitude de tenir cette table à jour. Toute valeur qui a un plafond ou un plancher doit être bornée **à l'endroit où elle est modifiée**, jamais dans le code d'affichage.

| Grandeur | Où elle vit | Plancher | Plafond | Bornée par |
| --- | --- | --- | --- | --- |
| `joueur.pv` | `Joueur` | 0 | `Constantes.pvJoueurMax` | `soigner`, `subirDegats` |
| `joueur.energie` | `Joueur` | 0 | `Joueur.energieMax` | `gagnerEnergie` |
| `joueur.cles` | `Joueur` | 0 | aucun | rien à borner |
| `game.vies` | `DonjonGame` | 0 | `Constantes.viesDepart` | `perdreUneVie` |
| `game.score` | `DonjonGame` | 0 | aucun | `ajouterScore` refuse les points négatifs |
| `game.combo` | `DonjonGame` | 0 | aucun | le multiplicateur est plafonné, pas le combo |

> **Le principe.** Borner à la source, pas à l'affichage. Si vous laissez `pv` monter à 113 en comptant sur la barre de vie pour ne pas dépasser 100 %, alors le jour où vous ajouterez un second affichage — un texte, une sauvegarde, une condition de victoire — le bug ressortira. Une valeur invalide ne doit jamais exister, même une frame.

---

## 38.10 — `lib/composants/cle.dart` : ouvrir la porte du chapitre 39

### Un objet qui ne sert à rien… aujourd'hui

La clé est le premier objet du jeu dont l'effet n'est **pas immédiat**. Une pièce donne des points tout de suite. Une potion soigne tout de suite. Une clé ne fait rien : elle rend possible quelque chose qui n'existe pas encore, la porte du chapitre 39.

C'est un cas de figure fréquent en développement de jeu, et il faut savoir le gérer proprement. La tentation est d'attendre le chapitre 39 pour écrire la clé. C'est une mauvaise idée pour deux raisons :

1. La clé et la porte sont deux moitiés d'un même mécanisme. Écrire la clé maintenant oblige à décider **maintenant** comment on compte les clés. Ce choix, fait tôt, évite un refactoring douloureux plus tard.
2. Une clé ramassable, même sans porte, est déjà testable : le compteur du HUD doit passer de 0 à 1.

### Le code

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

class Cle extends Collectible {
  Cle({Vector2? position})
      : super(position: position, size: Vector2(18, 9));

  /// Points de score offerts en prime : trouver la clé est un exploit.
  static const int primeScore = 50;

  /// Pour une clé, `valeur` est un NOMBRE DE CLÉS, pas des points.
  @override
  int get valeur => 1;

  @override
  Color get couleur => const Color(0xFFF2E27A);

  @override
  String get libelle => 'Clé';

  // Un objet rare flotte plus haut et plus lentement.
  @override
  double get amplitudeFlottement => 6.0;

  @override
  double get dureeFlottement => 1.5;

  @override
  void ramasser(Joueur joueur) {
    joueur.cles += valeur;

    final int gain = primeScore * game.multiplicateur;
    game.ajouterScore(primeScore);
    game.afficherTexteFlottant(position, 'Clé ! +$gain', couleur);
  }
}
```

### Dessiner une clé

Une clé, géométriquement, c'est un anneau, une tige et deux dents.

```text
       anneau        tige            dents
        ╭──╮   ═══════════════════   ▄  ▄
       │ ○ │                         █  █
        ╰──╯

   0   0.2                    0.75          1.0     (fraction de la largeur)
```

```dart
  static final Paint _metal = Paint()..color = const Color(0xFFF2E27A);
  static final Paint _ombre = Paint()..color = const Color(0xFF9A8A32);

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final double l = size.x;
    final double h = size.y;
    final double cy = h / 2;

    // L'anneau : un cercle plein, puis un trou de la couleur du fond.
    canvas.drawCircle(Offset(h / 2, cy), h / 2, _metal);
    canvas.drawCircle(Offset(h / 2, cy), h / 4, _ombre);

    // La tige.
    canvas.drawRect(
      Rect.fromLTWH(h / 2, cy - h * 0.14, l - h / 2, h * 0.28),
      _metal,
    );

    // Deux dents, vers le bas, à l'extrémité droite.
    canvas.drawRect(
      Rect.fromLTWH(l - h * 0.85, cy, h * 0.22, h * 0.45),
      _metal,
    );
    canvas.drawRect(
      Rect.fromLTWH(l - h * 0.35, cy, h * 0.22, h * 0.45),
      _metal,
    );
  }
```

> **Le trou de l'anneau.** On le dessine en `_ombre`, une teinte plus sombre, et non en transparent. Dessiner un vrai trou demanderait un `Path` avec `PathFillType.evenOdd` ou un `BlendMode.clear`, ce qui coûte une couche de composition. Sur un objet de 18 pixels, personne ne verra la différence. C'est le genre d'arbitrage constant en rendu 2D : le résultat perçu prime sur la pureté de la méthode.

### Ce que le chapitre 39 en fera

Pour que la suite soit claire, voici par anticipation le code de la porte. **Ne l'écrivez pas maintenant**, il est ici pour montrer où va la clé :

```dart
// lib/composants/porte.dart — CHAPITRE 39, pour information seulement
@override
void onCollisionStart(Set<Vector2> points, PositionComponent other) {
  super.onCollisionStart(points, other);
  if (other is! Joueur) return;

  if (verrouillee && other.cles < 1) {
    game.afficherTexteFlottant(position, 'Verrouillée', const Color(0xFFBBBBBB));
    return;
  }
  if (verrouillee) {
    other.cles -= 1;      // la clé est CONSOMMÉE
    verrouillee = false;
  }
  game.terminerNiveau();
}
```

Ce qui compte ici est la ligne `other.cles -= 1;`. Elle impose une décision : **une clé est consommée par la porte qu'elle ouvre**. Ce n'est pas un objet permanent qu'on garde pour toujours. C'est cette décision, prise au chapitre 38, qui rend possible un niveau à deux portes et deux clés au chapitre 39.

---

## 38.11 — Le compteur de clés du joueur

### Où stocker le compteur

Trois emplacements possibles pour `cles`, et il faut choisir :

| Emplacement | Avantage | Inconvénient |
| --- | --- | --- |
| `Joueur.cles` | L'inventaire appartient au personnage. Naturel. | Le joueur est recréé à chaque niveau : le compteur repart à zéro. |
| `DonjonGame.cles` | Survit au rechargement du niveau. | Mélange l'état de la partie et l'inventaire du héros. |
| Un service `Inventaire` | Extensible : armes, armures, parchemins. | Une abstraction de plus pour un seul entier. |

Le contrat tranche : `int cles = 0;` dans `Joueur`. C'est le bon choix, et il faut comprendre pourquoi.

**Une clé est locale à un niveau.** Vous ramassez la clé de la salle 2, vous ouvrez la porte de la salle 2. Vous n'emportez pas la clé dans la salle 3. Le fait que le compteur reparte à zéro au rechargement du niveau n'est donc pas un bug : c'est exactement la règle du jeu.

```text
  NIVEAU 1                       NIVEAU 2
  ┌────────────────────┐         ┌────────────────────┐
  │  J    k        D   │         │  J   k  k      D   │
  └────────────────────┘         └────────────────────┘
   cles : 0 -> 1 -> 0             cles : 0 -> 1 -> 2 -> 1

  Le joueur est reconstruit entre les deux niveaux.
  Le compteur repart à 0. C'est voulu.
```

Si un jour vous voulez une clé « maîtresse » qui traverse les niveaux, elle ira dans `DonjonGame` ou dans le `SauvegardeService` du chapitre 40 — pas dans `Joueur`.

### Le champ et ses accesseurs

Le champ existe déjà depuis le chapitre 36, mais on ne l'utilisait pas. Complétons-le :

```dart
// lib/composants/joueur.dart

/// Clés en poche. Consommées par les portes verrouillées (chapitre 39).
int cles = 0;

/// Vrai si le joueur peut ouvrir une porte verrouillée.
bool get possedeUneCle => cles > 0;

/// Consomme une clé. Renvoie faux s'il n'y en avait pas.
bool consommerUneCle() {
  if (cles <= 0) {
    return false;
  }
  cles--;
  return true;
}
```

`consommerUneCle` illustre un patron utile : **une méthode qui modifie l'état et renvoie si elle a réussi**. L'appelant écrit alors :

```dart
if (joueur.consommerUneCle()) {
  ouvrirLaPorte();
} else {
  afficherMessage('Il vous faut une clé.');
}
```

C'est plus sûr que `if (joueur.cles > 0) { joueur.cles--; ouvrir(); }`, où le test et la décrémentation peuvent se désynchroniser lors d'un refactoring.

### Le HUD lira ce compteur

La section 38.24 affichera ce nombre. Le HUD a donc besoin d'atteindre le joueur. Il faut une référence, exposée sur le jeu :

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 38

/// Le héros de la partie en cours.
/// Nulle avant `demarrerPartie()` et pendant les transitions de niveau :
/// tout code du HUD doit tester ce null.
Joueur? joueur;
```

Le type est **nullable**, et ce n'est pas une facilité : c'est la vérité. Entre deux niveaux, pendant une demi-seconde, il n'y a pas de joueur dans le monde. Un `late Joueur joueur` non initialisé lèverait une `LateInitializationError` au premier `update` du HUD. Le null safety du **chapitre 12** vous force ici à écrire la vérité, et c'est un service qu'il vous rend.

Chaque composant du HUD commencera donc son `update` par :

```dart
final Joueur? j = game.joueur;
if (j == null) {
  return;         // pas de joueur : rien à afficher, on ne plante pas
}
// ici, j est promu en Joueur non nullable
```

C'est la promotion de type du chapitre 12, section 12.9. On stocke la valeur dans une variable locale **avant** de tester : un champ d'instance nullable n'est pas promu par Dart, parce qu'il pourrait changer entre le test et l'usage.

---

## 38.12 — `ajouterScore()` sur `DonjonGame`

### La signature du contrat, et ce qu'elle interdit

```dart
void ajouterScore(int points);
```

Trois contraintes découlent de cette ligne :

- **Elle ne renvoie rien.** L'appelant ne peut pas connaître le gain réel. C'est pourquoi `Piece` calcule `valeur * game.multiplicateur` avant l'appel (section 38.5).
- **Elle prend le score de base**, pas le score final. C'est `ajouterScore` qui applique le multiplicateur, jamais l'appelant. Un seul endroit fait le calcul.
- **Elle est publique.** Les ennemis (chapitre 37), les collectibles (ici) et le boss (chapitre 39) l'appellent tous.

### L'implémentation

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 38

int score = 0;

/// Meilleur score de la session ; persisté au chapitre 40.
int meilleurScore = 0;

/// Ajoute des points, après application du multiplicateur de combo,
/// et relance la fenêtre de combo.
void ajouterScore(int points) {
  if (points <= 0) {
    return;
  }

  score += points * multiplicateur;

  combo++;
  _tempsRestantCombo = dureeCombo;

  if (score > meilleurScore) {
    meilleurScore = score;
  }
}
```

**`if (points <= 0) return;`.** Une méthode nommée « ajouter » ne doit pas pouvoir retirer. Si un jour vous voulez une pénalité, écrivez `retirerScore`. Cette garde évite aussi qu'un bug de calcul en amont — une valeur non initialisée, une soustraction ratée — ne fasse silencieusement baisser le score.

**L'ordre `score +=` puis `combo++`.** Le multiplicateur appliqué est celui **d'avant** l'incrément. Le premier objet d'une série rapporte donc ×1, et c'est ce qu'il faut : le combo est une récompense pour la série, pas pour le premier élément.

**La mise à jour de `meilleurScore` ici.** On pourrait la faire à la fin de la partie. La faire à chaque point garantit que le meilleur score est correct même si le joueur ferme brutalement l'application. Le chapitre 40 branchera la persistance sur ce champ.

### Qui appelle `ajouterScore`, et avec quelles valeurs

Récapitulons toutes les sources de points du jeu. Cette table est le document d'équilibrage : c'est en la lisant qu'on décide si le jeu est trop généreux ou trop avare.

| Source | Points de base | Chapitre | Fait monter le combo |
| --- | --- | --- | --- |
| Pièce ramassée | 10 | 38 | oui |
| Potion inutile (PV pleins) | 5 | 38 | oui |
| Clé ramassée | 50 | 38 | oui |
| Gobelin tué | 25 | 37 | oui |
| Chauve-souris tuée | 15 | 37 | oui |
| Niveau terminé | 100 | 39 | non (appel direct sans combo) |
| Boss vaincu | 500 | 39 | non |

Une remarque d'équilibrage : la clé rapporte 50 points, soit cinq pièces. C'est volontairement beaucoup. Trouver la clé est l'objectif principal du niveau ; le score doit le refléter, sinon le joueur optimisera le ramassage des pièces et ignorera la progression.

---

## 38.13 — Le multiplicateur de combo

### Le problème que le combo résout

Sans combo, ramasser cinquante pièces éparpillées est aussi rentable que ramasser cinquante pièces alignées. Le score ne récompense que la **quantité**, jamais l'**adresse**.

Le combo change cela. Il récompense l'enchaînement rapide : ramasser trois pièces en trois secondes vaut plus que trois pièces en trente secondes. Le joueur qui maîtrise ses sauts est payé pour sa maîtrise.

```text
  LE COMBO EST UNE FENÊTRE GLISSANTE DE 3 SECONDES

  temps ──────────────────────────────────────────────────────────>

  ramassage    ●     ●   ●      ●                        ●      ●
  combo        1     2   3      4         (expire)       1      2
  multipl.    ×1    ×1  ×1     ×2                       ×1     ×1
                                    └─── 3 s sans rien ───┘

  Chaque ramassage RECHARGE la fenêtre à 3 s.
  Un silence de 3 s la ferme et remet le combo à zéro.
```

### Les champs et le calcul

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 38

/// Nombre de ramassages enchaînés dans la fenêtre de combo.
int combo = 0;

/// Durée de la fenêtre, en secondes. Rechargée à chaque ramassage.
static const double dureeCombo = 3.0;

/// Plafond du multiplicateur : sans lui, un joueur patient
/// pourrait atteindre ×50 et faire exploser tout l'équilibrage.
static const int multiplicateurMax = 4;

/// Temps restant avant la fermeture de la fenêtre.
double _tempsRestantCombo = 0;

/// Multiplicateur courant : un cran de plus tous les trois ramassages,
/// plafonné à [multiplicateurMax].
int get multiplicateur => min(1 + combo ~/ 3, multiplicateurMax);

/// Ferme la fenêtre et remet le compteur à zéro.
void reinitialiserCombo() {
  combo = 0;
  _tempsRestantCombo = 0;
}
```

`~/` est la division entière du **chapitre 3**. Déroulons la table de vérité, elle est plus parlante qu'un long discours :

| `combo` | `combo ~/ 3` | `1 + ...` | `multiplicateur` |
| --- | --- | --- | --- |
| 0, 1, 2 | 0 | 1 | ×1 |
| 3, 4, 5 | 1 | 2 | ×2 |
| 6, 7, 8 | 2 | 3 | ×3 |
| 9, 10, 11 | 3 | 4 | ×4 |
| 12 et plus | 4 et plus | 5 et plus | **×4** (plafonné) |

`min` vient de `dart:math`. Sans le plafond, un joueur ramassant deux cents objets d'affilée atteindrait ×67 : le score n'aurait plus aucun sens et le compteur du HUD déborderait de l'écran.

### Faire s'écouler le temps

Le décompte se fait dans `update`. C'est la boucle de jeu du **chapitre 20**, appliquée à une variable de gameplay plutôt qu'à une position.

```dart
// lib/donjon_game.dart

@override
void update(double dt) {
  super.update(dt);     // INDISPENSABLE : propage le tick à tout l'arbre

  if (_tempsRestantCombo > 0) {
    _tempsRestantCombo -= dt;
    if (_tempsRestantCombo <= 0) {
      reinitialiserCombo();
    }
  }
}
```

Trois pièges à connaître.

**Oublier `super.update(dt)`.** Le monde entier se fige : plus de joueur, plus d'ennemis, plus d'effets. C'est le piège numéro un de Flame, rappelé dans la fiche de référence.

**Décompter en frames plutôt qu'en secondes.** `_tempsRestantCombo--` à chaque frame donnerait une fenêtre de 3 frames, soit 0,05 seconde à 60 images par seconde. Et sur un écran à 120 Hz, elle durerait deux fois moins longtemps. `dt` est en **secondes** : c'est tout l'objet du chapitre 20.

**Ne pas remettre le compteur à zéro en même temps que le temps.** Si vous ne réinitialisez que `_tempsRestantCombo`, `combo` continue de grimper d'une session de jeu à l'autre et le multiplicateur reste bloqué à ×4. Les deux champs forment un état : ils se réinitialisent ensemble. C'est précisément pour cela qu'on passe par la méthode `reinitialiserCombo()` plutôt que d'écrire les affectations à la main.

### Casser le combo quand le joueur est touché

Un combo qui survit aux dégâts est un combo trop facile. Ajoutons la rupture dans `perdreUneVie` et au moment des dégâts. Ce sont deux lignes dans du code du chapitre 37 :

```dart
// lib/donjon_game.dart — MODIFICATION du chapitre 37
void perdreUneVie() {
  vies--;
  reinitialiserCombo();          // <── AJOUT DU CHAPITRE 38
  if (vies <= 0) {
    changerEtat(GameState.gameOver);
  } else {
    // ... réapparition du joueur (chapitre 37)
  }
}
```

```dart
// lib/composants/joueur.dart — MODIFICATION du chapitre 36
void subirDegats(double degats) {
  if (invincible || pv <= 0) {
    return;
  }
  pv = (pv - degats).clamp(0.0, Constantes.pvJoueurMax);
  game.reinitialiserCombo();     // <── AJOUT DU CHAPITRE 38
  // ... invincibilité temporaire, clignotement (chapitre 37)
}
```

> **Décision de game design assumée.** Casser le combo au moindre dégât rend le jeu plus tendu et récompense l'évitement. Si vos testeurs trouvent cela trop punitif, une variante douce consiste à diviser le combo par deux plutôt que de le remettre à zéro : `combo = combo ~/ 2;`. Le principe reste : **prendre un coup doit coûter quelque chose d'autre que des PV.**

### La fenêtre visible

Le joueur ne peut pas jouer avec un mécanisme qu'il ne voit pas. La section 38.21 affichera le multiplicateur à côté du score, et la section 38.26 y ajoutera une petite barre qui se vide — la fenêtre de combo qui se referme. Un mécanisme invisible n'existe pas.

```dart
/// Fraction de la fenêtre de combo restante, de 0 à 1.
/// Utilisée par le HUD pour dessiner la jauge qui se vide.
double get fractionCombo =>
    dureeCombo <= 0 ? 0 : (_tempsRestantCombo / dureeCombo).clamp(0.0, 1.0);
```

**Résultat, en jouant :**

```text
Le joueur enchaîne cinq pièces en deux secondes.
Affichage : « Score 00080 »  et en dessous  « × 2 combo »
Une fine barre orange sous le score se vide en 3 secondes.
Le joueur s'arrête. La barre atteint zéro. Le « × 2 combo » disparaît.
```

---

## 38.14 — Le HUD : pourquoi il vit dans le viewport et pas dans le monde

### Le rappel du chapitre 31

Au chapitre 31, section 31.19, vous avez appris la règle. Elle tient en une phrase :

> **Un composant ajouté au `viewport` reste fixe à l'écran, devant le monde. Un composant ajouté au `world` vit dans le donjon et subit la caméra.**

Et à la section 31.20, vous avez vu ce qui casse quand on se trompe. Reprenons ce schéma, parce que c'est le moment de l'appliquer pour de bon.

```text
  L'ARBRE DE COMPOSANTS DU « DONJON DE DART » APRÈS CE CHAPITRE

  DonjonGame  (FlameGame)
   │
   ├── World                      <- LE DONJON
   │    ├── Plateforme  ×n            ces objets ont une position
   │    ├── Joueur                    EN COORDONNÉES DE MONDE
   │    ├── Gobelin ×n                et la caméra les transforme
   │    ├── Chauvesouris ×n
   │    ├── Piece ×n
   │    ├── Potion ×n
   │    ├── Cle
   │    ├── ParticleSystemComponent   (éphémères)
   │    └── TexteFlottant             (éphémères, monde !)
   │
   └── CameraComponent
        ├── backdrop                  fond fixe, DERRIÈRE le monde
        ├── Viewfinder                position / zoom = 2 / anchor
        └── Viewport                  <- L'ÉCRAN
             ├── Hud                      ces objets ont une position
             │    ├── BarreDeVie          EN COORDONNÉES D'ÉCRAN
             │    ├── BarreEnergie        et la caméra ne les touche pas
             │    ├── CompteurScore
             │    ├── CompteurVies
             │    ├── CompteurCles
             │    └── IndicateurObjectif
             ├── JoystickComponent        (chapitre 30, mobile)
             └── HudButtonComponent       (chapitre 30, mobile)
```

### Ce qui casse si on se trompe

Faites l'expérience une fois, c'est la meilleure façon de ne jamais l'oublier. Remplacez :

```dart
await camera.viewport.add(Hud());
```

par :

```dart
await world.add(Hud());     // ERREUR VOLONTAIRE
```

**Résultat :**

```text
1. La barre de vie apparaît au coin haut-gauche du DONJON,
   pas de l'écran. Elle est donc quelque part dans le décor.
2. Dès que le héros avance de trois pas, elle sort du cadre.
   Le joueur ne voit plus ses PV.
3. Le zoom de caméra vaut 2 : la barre est dessinée deux fois
   trop grande, et la police du score est floue.
4. La barre de vie passe DERRIÈRE les murs, parce qu'elle est
   maintenant un objet du monde comme un autre.
5. Un gobelin peut entrer en collision avec le compteur de score.
```

Les cinq symptômes viennent de la même cause. Ils sont si caractéristiques qu'ils vous serviront de diagnostic : **du HUD qui glisse ou qui grossit, c'est du HUD dans le monde**.

### Trois emplacements, trois usages

Le `CameraComponent` offre en réalité trois points d'accroche. Il faut savoir choisir entre eux.

| Où l'on ajoute | Suit la caméra ? | Subit le zoom ? | Pour quoi |
| --- | --- | --- | --- |
| `world.add(...)` | oui | oui | tout ce qui est **dans** le donjon : entités, décor, textes de dégâts |
| `camera.viewport.add(...)` | non | non | le **HUD** : barres, compteurs, joystick, boutons |
| `camera.backdrop.add(...)` | non | non | le **fond fixe**, dessiné derrière le monde : dégradé, ciel |
| `camera.viewfinder.add(...)` | oui | oui | rare : un cadre qui suit le regard, une vignette |

Une quatrième possibilité existe : `overlays.add('hud')`, qui affiche un **widget Flutter** au-dessus du canvas. C'est un choix radicalement différent, comparé en détail à la section 38.25.

### Le cas particulier du texte flottant

Attention à ne pas surgénéraliser. Le « +10 » qui monte quand on ramasse une pièce **n'est pas du HUD**. Il doit rester collé à l'endroit du donjon où la pièce se trouvait ; si le joueur recule, le texte doit reculer avec le décor.

```text
  CE QUI VA DANS LE VIEWPORT        CE QUI VA DANS LE MONDE

  « Score 01240 »                    « +10 » au-dessus de la pièce
  la barre de vie                    « -15 » au-dessus du gobelin
  le nombre de vies                  la bulle « ! » d'un ennemi alerté
  le nombre de clés                  le nom d'un objet au sol
  « Objectif : la clé »              une flèche indiquant une sortie proche

  Règle : si l'information concerne UN ENDROIT du donjon,
  elle vit dans le monde. Si elle concerne LA PARTIE,
  elle vit dans le viewport.
```

---

## 38.15 — `lib/hud/hud.dart` : le conteneur

### Pourquoi un conteneur plutôt que six ajouts

On pourrait écrire :

```dart
// Possible, mais à éviter
await camera.viewport.addAll([
  BarreDeVie(...),
  BarreEnergie(...),
  CompteurScore(...),
  CompteurVies(...),
  CompteurCles(...),
  IndicateurObjectif(...),
]);
```

Cela marche. Mais on perd trois choses.

**Le retrait en un geste.** Au chapitre 40, l'écran de Game Over devra cacher le HUD. Avec un conteneur : `hud.removeFromParent()`. Sans conteneur : six retraits, et il faudra les retrouver dans le viewport.

**La visibilité groupée.** Pendant une cinématique de fin de niveau (chapitre 39), on veut faire disparaître tout le HUD en fondu. Un conteneur permet un seul effet.

**La lisibilité.** Le fichier `hud.dart` devient la carte de l'interface : on y lit d'un coup d'œil quels éléments existent et où ils sont placés. C'est le principe de séparation des responsabilités du **chapitre 26**.

### `Component` ou `PositionComponent` ?

Question importante, et la réponse est contre-intuitive : le conteneur est un **`Component` simple**, sans position ni taille.

```dart
class Hud extends Component with HasGameReference<DonjonGame> { ... }
```

Pourquoi ? Parce que ses enfants sont des `HudMarginComponent`, et que ceux-ci cherchent, en remontant l'arbre, **le premier ancêtre qui fournit une taille**. Le code source de Flame est explicite :

```dart
// flame/lib/src/components/input/hud_margin_component.dart
_sizeProvider =
    ancestors().firstWhereOrNull((c) => c is ReadOnlySizeProvider)
        as ReadOnlySizeProvider?;
```

Si `Hud` était un `PositionComponent`, il serait ce fournisseur de taille — et comme sa taille vaut `Vector2.zero()` par défaut, **tous les enfants se colleraient au coin haut-gauche**. En le laissant `Component`, la recherche remonte jusqu'au `Viewport`, qui implémente `SizeProvider` et dont la taille est celle de l'écran. C'est exactement ce qu'on veut.

```text
  RECHERCHE DU FOURNISSEUR DE TAILLE, DEPUIS BarreDeVie

  BarreDeVie
     ↑ ancestors()
  Hud                 <- Component : pas de taille, on continue
     ↑
  Viewport            <- implements SizeProvider  ✔  TROUVÉ
     ↑                   size = taille de l'écran
  CameraComponent
     ↑
  DonjonGame
```

### Le code

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../donjon_game.dart';
import 'barre_de_vie.dart';
import 'compteur_score.dart';

class Hud extends Component with HasGameReference<DonjonGame> {
  Hud({super.priority = 10});

  /// Marge de sécurité par rapport aux bords de l'écran.
  static const double marge = 16;

  late final BarreDeVie barreDeVie;
  late final BarreEnergie barreEnergie;
  late final CompteurVies compteurVies;
  late final CompteurScore compteurScore;
  late final CompteurCles compteurCles;
  late final IndicateurObjectif indicateurObjectif;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // ---- Colonne de gauche : l'état du héros -------------------------
    barreDeVie = BarreDeVie(
      margin: const EdgeInsets.only(left: marge, top: marge),
    );
    barreEnergie = BarreEnergie(
      margin: const EdgeInsets.only(left: marge, top: marge + 20),
    );
    compteurVies = CompteurVies(
      margin: const EdgeInsets.only(left: marge, top: marge + 36),
    );

    // ---- Colonne de droite : l'état de la partie ----------------------
    compteurScore = CompteurScore(
      margin: const EdgeInsets.only(top: marge, right: marge),
    );
    compteurCles = CompteurCles(
      margin: const EdgeInsets.only(top: marge + 44, right: marge),
    );
    indicateurObjectif = IndicateurObjectif(
      margin: const EdgeInsets.only(top: marge + 70, right: marge),
    );

    await addAll([
      barreDeVie,
      barreEnergie,
      compteurVies,
      compteurScore,
      compteurCles,
      indicateurObjectif,
    ]);
  }

  /// Fait apparaître ou disparaître tout le HUD d'un coup.
  /// Utilisé par les écrans de pause et de fin (chapitre 40).
  void definirVisibilite(bool visible) {
    for (final enfant in children.whereType<PositionComponent>()) {
      enfant.scale = visible ? Vector2.all(1) : Vector2.all(0.001);
    }
  }
}
```

Notez `super.priority = 10`. Le HUD doit être dessiné **devant** le joystick ? Non, l'inverse : les boutons tactiles doivent rester cliquables et visibles. On donne au HUD une priorité modeste et aux boutons une priorité supérieure. La section 38.29 revient sur ce point.

### L'installer

Une seule ligne, dans `DonjonGame` :

```dart
// lib/donjon_game.dart

late final Hud hud;

Future<void> demarrerPartie() async {
  // ... (chapitre 35 : remise à zéro du score, des vies, chargement)

  hud = Hud();
  await camera.viewport.add(hud);      // <── LE HUD, DANS LE VIEWPORT

  changerEtat(GameState.enJeu);
}
```

> **`await` ou pas ?** `Component.add()` renvoie un `FutureOr<void>` : le composant n'est pas monté immédiatement. Ici, on `await` parce que la suite de `demarrerPartie` change l'état du jeu et pourrait vouloir manipuler le HUD. Dans le doute, `await` : le coût est nul et le bug évité est réel.

---

## 38.16 — `HudMarginComponent` et le placement relatif aux bords

### Le problème que ce composant résout

Un HUD placé en coordonnées absolues casse dès que la fenêtre change de taille.

```text
  POSITION ABSOLUE : position = Vector2(700, 16)

  Écran 800 × 450                      Écran 400 × 800 (mobile portrait)
  ┌──────────────────────┐             ┌──────────┐
  │              [score] │             │          │  le score est
  │                      │             │          │  hors écran :
  │                      │             │          │  x = 700 > 400
  └──────────────────────┘             │          │
                                       └──────────┘
```

Il faudrait recalculer `position` à chaque `onGameResize`. `HudMarginComponent` le fait pour vous.

### La signature réelle

```dart
HudMarginComponent({
  EdgeInsets? margin,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
  ComponentKey? key,
});
```

Le constructeur porte une assertion : `margin != null || position != null`. Vous devez fournir **au moins l'un des deux**. Si vous donnez une `position`, le composant calcule lui-même la marge correspondante à cet instant et la conservera ensuite.

### Comment la position est calculée

Voici le cœur de la mécanique, tirée du code source de Flame 1.38.0 :

```dart
void _updateMargins() {
  final x = margin.left != 0
      ? margin.left + scaledSize.x / 2
      : _sizeProvider!.size.x - margin.right - scaledSize.x / 2;
  final y = margin.top != 0
      ? margin.top + scaledSize.y / 2
      : _sizeProvider!.size.y - margin.bottom - scaledSize.y / 2;
  position.setValues(x, y);
  position = Anchor.center.toOtherAnchorPosition(position, anchor, scaledSize);
}
```

Traduction en français :

```text
  POUR L'AXE X
    si margin.left n'est PAS zéro  ->  on s'accroche à GAUCHE
    sinon                          ->  on s'accroche à DROITE (margin.right)

  POUR L'AXE Y
    si margin.top n'est PAS zéro   ->  on s'accroche en HAUT
    sinon                          ->  on s'accroche en BAS (margin.bottom)
```

D'où les quatre recettes de coin :

```dart
// Coin haut-gauche
HudMarginComponent(
  margin: const EdgeInsets.only(left: 16, top: 16),
  anchor: Anchor.topLeft,
  size: Vector2(160, 14),
);

// Coin haut-droit
HudMarginComponent(
  margin: const EdgeInsets.only(right: 16, top: 16),
  anchor: Anchor.topRight,
  size: Vector2(160, 24),
);

// Coin bas-gauche
HudMarginComponent(
  margin: const EdgeInsets.only(left: 16, bottom: 16),
  anchor: Anchor.bottomLeft,
  size: Vector2(120, 120),
);

// Coin bas-droit
HudMarginComponent(
  margin: const EdgeInsets.only(right: 16, bottom: 16),
  anchor: Anchor.bottomRight,
  size: Vector2(64, 64),
);
```

```text
  LES QUATRE ANCRAGES, AVEC UNE MARGE DE 16 PIXELS

  ┌────────────────────────────────────────────────────┐
  │  ┌──────────┐                        ┌──────────┐  │
  │  │ topLeft  │                        │ topRight │  │
  │  └──────────┘                        └──────────┘  │
  │                                                    │
  │                                                    │
  │  ┌──────────┐                        ┌──────────┐  │
  │  │bottomLeft│                        │bottomRght│  │
  │  └──────────┘                        └──────────┘  │
  └────────────────────────────────────────────────────┘
      ↑ 16 px                                  16 px ↑

  Quelle que soit la taille de la fenêtre, les marges tiennent.
```

### Le piège de `margin.left == 0`

Relisez la ligne du code source : `margin.left != 0 ? ... : ...`. Le test porte sur **la valeur**, pas sur la présence.

```dart
// PIÈGE : on croit demander « collé au bord gauche, sans marge »
HudMarginComponent(
  margin: const EdgeInsets.only(left: 0, top: 16),
  anchor: Anchor.topLeft,
  size: Vector2(160, 14),
);
```

**Résultat :**

```text
Le composant se colle au bord DROIT.
Explication : margin.left vaut 0, donc la branche « gauche » est
ignorée ; Flame utilise margin.right, qui vaut 0 lui aussi,
et place le composant à x = largeurEcran - taille/2.
```

Aucune exception, aucun avertissement. Simplement un élément à l'opposé de ce que vous vouliez. La parade est simple : **n'utilisez jamais 0 comme marge**. Si vous voulez coller au bord, écrivez `left: 0.01`. Ou, plus honnêtement, laissez une vraie marge : un HUD collé au bord de l'écran est de toute façon illisible sur un téléphone à encoche (section 38.29).

### Le redimensionnement

Le composant écoute deux choses :

- sa propre `size` — si vous changez la taille d'une barre en cours de jeu, la marge est recalculée ;
- `onGameResize` — si la fenêtre change, les marges sont recalculées.

Vous n'avez donc rien à écrire. Testez : lancez le jeu sur bureau et redimensionnez la fenêtre à la souris. Le score doit rester collé au coin haut-droit pendant tout le redimensionnement.

> **Erreur de montage.** Si vous ajoutez un `HudMarginComponent` à un parent sans taille et sans ancêtre dimensionné, Flame lève une assertion explicite : « The parent of a HudMarginComponent needs to provide a size, for example by being a PositionComponent. » C'est ce qui arriverait si vous ajoutiez le HUD directement au `FlameGame` sans passer par le viewport.

---

## 38.17 — `lib/hud/barre_de_vie.dart`

### Ce qu'une barre de vie doit dire

Une barre de vie n'affiche pas un nombre : elle affiche un **rapport**. Le joueur ne lit pas « 43 PV », il voit « moins de la moitié ». C'est plus rapide à traiter, et c'est ce dont il a besoin en pleine action.

Une bonne barre de vie répond à quatre questions, dans cet ordre de priorité :

1. **Quelle proportion me reste-t-il ?** — la longueur du remplissage.
2. **Est-ce grave ?** — la couleur : vert, orange, rouge.
3. **Viens-je de perdre de la vie, et combien ?** — la barre fantôme (section 38.19).
4. **Quel est le total ?** — le texte, optionnel, en petit.

### La classe

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../composants/joueur.dart';
import '../config/constantes.dart';
import '../donjon_game.dart';

class BarreDeVie extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  BarreDeVie({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(largeur, hauteur),
          anchor: Anchor.topLeft,
        );

  static const double largeur = 168;
  static const double hauteur = 14;
  static const double bordure = 2;

  /// Vitesse de rattrapage de la barre fantôme, en fraction par seconde.
  static const double vitesseFantome = 0.55;

  late final RectangleComponent _cadre;
  late final RectangleComponent _creux;
  late final RectangleComponent _fantome;
  late final RectangleComponent _remplissage;

  double _ratio = 1;
  double _ratioFantome = 1;

  /// Largeur utile, à l'intérieur de la bordure.
  static double get _utile => largeur - bordure * 2;
  // ...
}
```

`HudMarginComponent` **plus** `HasGameReference<DonjonGame>` : le premier gère le placement, le second donne l'accès à `game.joueur`. Les deux mixins sont indépendants et se combinent sans problème — c'est la composition de mixins du **chapitre 11**.

---

## 38.18 — Dessiner une barre avec deux `RectangleComponent`

### Le principe : deux rectangles superposés

Le plus simple des affichages de jauge tient en deux rectangles :

```text
  ┌────────────────────────────────────────────┐
  │████████████████████████░░░░░░░░░░░░░░░░░░░░│
  └────────────────────────────────────────────┘
   └──── remplissage ─────┘
   └────────────── creux (fond) ───────────────┘

  Le CREUX a toujours la largeur totale.
  Le REMPLISSAGE a une largeur de  utile × ratio.
  Les deux partagent le même coin haut-gauche.
```

Dans notre barre, on empile quatre rectangles plutôt que deux, mais l'idée est identique. Chaque couche a un rôle :

```text
  ORDRE D'AJOUT = ORDRE DE DESSIN, DU FOND VERS L'AVANT

  1. _cadre        largeur totale       noir      le contour
  2. _creux        largeur utile        bordeaux  la partie vide
  3. _fantome      utile × ratioFantome blanc     la vie qu'on vient de perdre
  4. _remplissage  utile × ratio        vert/orange/rouge   la vie restante
```

### Le code de construction

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  _cadre = RectangleComponent(
    size: Vector2(largeur, hauteur),
    paint: Paint()..color = const Color(0xFF15151E),
  );

  _creux = RectangleComponent(
    position: Vector2.all(bordure),
    size: Vector2(_utile, hauteur - bordure * 2),
    paint: Paint()..color = const Color(0xFF3B2530),
  );

  _fantome = RectangleComponent(
    position: Vector2.all(bordure),
    size: Vector2(_utile, hauteur - bordure * 2),
    paint: Paint()..color = const Color(0xFFF6F1E7),
  );

  _remplissage = RectangleComponent(
    position: Vector2.all(bordure),
    size: Vector2(_utile, hauteur - bordure * 2),
    paint: Paint()..color = const Color(0xFF5CBF54),
  );

  await addAll([_cadre, _creux, _fantome, _remplissage]);
}
```

Trois précisions techniques.

**Aucun `anchor` n'est donné aux enfants.** Leur ancre est donc `Anchor.topLeft`, la valeur par défaut. C'est indispensable : quand on réduira leur largeur, ils doivent se raccourcir **vers la droite** en gardant leur bord gauche fixe. Avec `Anchor.center`, la barre rétrécirait des deux côtés à la fois, ce qui n'a aucun sens pour une jauge.

**Les positions sont locales.** `Vector2.all(bordure)` place l'enfant à 2 pixels du coin haut-gauche **de la barre**, pas de l'écran. Le placement absolu, lui, est géré une fois pour toutes par `HudMarginComponent`.

**L'ordre dans `addAll` fixe l'ordre de dessin.** Aucun `priority` n'est nécessaire tant que les composants sont ajoutés dans le bon ordre et ont la même priorité (chapitre 28).

### Modifier la largeur d'un `RectangleComponent`

Point technique important. `RectangleComponent` hérite de `PolygonComponent` : il est défini par quatre sommets, pas par un `Rect`. Heureusement, son constructeur installe un écouteur sur `size` :

```dart
// flame/lib/src/geometry/rectangle_component.dart
size.addListener(
  () => refreshVertices(
    newVertices: sizeToVertices(size, anchor),
    shrinkToBoundsOverride: false,
  ),
);
```

Autrement dit : **changer `size` suffit**, les sommets sont recalculés automatiquement. On peut donc écrire :

```dart
void _appliquerLargeur(RectangleComponent barre, double valeur) {
  final double l = valeur.clamp(0.01, _utile);

  // Ne rien faire si la largeur n'a pas bougé : chaque écriture
  // déclenche un recalcul des quatre sommets.
  if ((barre.size.x - l).abs() < 0.05) {
    return;
  }
  barre.size.setValues(l, barre.size.y);
}
```

Deux détails.

**`clamp(0.01, _utile)` et non `clamp(0, _utile)`.** Un rectangle de largeur nulle a ses quatre sommets confondus : certaines opérations géométriques y calculent des normales et produisent des `NaN`. Un centième de pixel est invisible et mathématiquement sain. C'est la même prudence que pour `Vector2.all(0.1)` du pop (section 38.7).

**Le garde-fou de 0,05 pixel.** Sans lui, on réécrit `size` soixante fois par seconde pour un déplacement sous-pixel, et on recalcule à chaque fois quatre sommets pour rien. Avec quatre barres à l'écran, cela reste négligeable ; avec quarante barres de vie d'ennemis, cela ne l'est plus.

> **Variante plus économique : `scale`.** Au lieu de changer `size`, on peut poser `barre.scale.x = ratio` en gardant la largeur pleine. C'est moins de calculs — une simple matrice — mais cela met aussi à l'échelle les enfants éventuels et déforme les bordures. Pour une jauge simple, `scale` est parfaitement acceptable ; pour une barre décorée, `size` est plus sûr. Nous gardons `size`, plus explicite.

### La couleur selon le ratio

```dart
static Color _couleurSelonRatio(double ratio) {
  if (ratio > 0.5) {
    return const Color(0xFF5CBF54);   // vert : tout va bien
  }
  if (ratio > 0.25) {
    return const Color(0xFFE0A128);   // orange : attention
  }
  return const Color(0xFFE23F4C);     // rouge : danger
}
```

Trois paliers, pas dix. Un dégradé continu est joli mais **illisible** : le joueur ne perçoit pas la différence entre 62 % et 58 %. Trois couleurs franches donnent trois messages nets : « ça va », « attention », « danger ».

> **Accessibilité.** Environ 8 % des hommes distinguent mal le rouge et le vert. Ne faites jamais reposer une information vitale sur la seule couleur. Ici, la **longueur** de la barre porte déjà l'information : la couleur ne fait que la renforcer. C'est la bonne façon de procéder. Au chapitre 42, nous ajouterons une option « barre de vie chiffrée » pour les mêmes raisons.

---

## 38.19 — Animer la barre quand la vie descend

### Le problème d'une barre qui saute

Un gobelin inflige 15 dégâts. Sans animation :

```text
  frame N     ████████████████████░░░░  100 PV
  frame N+1   █████████████████░░░░░░░   85 PV
```

Un seul frame, soit 16 millisecondes. **Le joueur ne voit rien.** Il constate seulement, plus tard, que sa barre est plus courte. L'information « je viens de prendre 15 dégâts » est perdue.

### La barre fantôme

La solution standard, utilisée dans presque tous les jeux de combat depuis vingt-cinq ans, consiste à afficher **deux** barres :

```text
  AVANT LE COUP
  ┌────────────────────────────────────────────┐
  │████████████████████████████████████████████│  vert, 100 %
  └────────────────────────────────────────────┘

  IMMÉDIATEMENT APRÈS (frame N+1)
  ┌────────────────────────────────────────────┐
  │██████████████████████████████████▒▒▒▒▒▒▒▒▒▒│
  └────────────────────────────────────────────┘
   └────────── vert, 85 % ───────────┘└ blanc ┘
                                       le fantôme, encore à 100 %

  0,3 SECONDE PLUS TARD
  ┌────────────────────────────────────────────┐
  │██████████████████████████████████▒▒▒░░░░░░░│
  └────────────────────────────────────────────┘
                                     le fantôme rattrape

  0,6 SECONDE PLUS TARD
  ┌────────────────────────────────────────────┐
  │██████████████████████████████████░░░░░░░░░░│
  └────────────────────────────────────────────┘
                                     rattrapé
```

La barre verte **saute** immédiatement à la nouvelle valeur : l'information est exacte tout de suite. La barre blanche **glisse** derrière elle : elle raconte visuellement combien on vient de perdre.

C'est une idée remarquable, parce qu'elle résout la contradiction entre exactitude et lisibilité. Une barre qui glisserait toute seule serait jolie mais mensongère pendant une demi-seconde — dangereux quand le joueur décide de fuir ou de continuer.

### Le code

```dart
@override
void update(double dt) {
  super.update(dt);

  final Joueur? joueur = game.joueur;
  if (joueur == null) {
    return;
  }

  // 1. La valeur exacte, tout de suite.
  _ratio = (joueur.pv / Constantes.pvJoueurMax).clamp(0.0, 1.0);

  // 2. Le fantôme rattrape, à vitesse constante, seulement vers le bas.
  if (_ratioFantome > _ratio) {
    _ratioFantome = max(_ratio, _ratioFantome - vitesseFantome * dt);
  } else {
    // Un SOIN doit être instantané : on ne fait pas attendre
    // une bonne nouvelle.
    _ratioFantome = _ratio;
  }

  // 3. On applique aux rectangles.
  _appliquerLargeur(_remplissage, _utile * _ratio);
  _appliquerLargeur(_fantome, _utile * _ratioFantome);
  _remplissage.paint.color = _couleurSelonRatio(_ratio);
}
```

L'asymétrie entre la descente et la montée est délibérée. En descente, le fantôme traîne : on veut montrer la perte. En montée, il colle : une potion doit produire un effet immédiat et gratifiant.

**`vitesseFantome = 0.55`** signifie « 55 % de la barre par seconde ». Une perte de 15 PV sur 100, soit 15 % de barre, met donc environ 0,27 seconde à se résorber. Une perte massive de 60 PV met une seconde entière : plus le coup est violent, plus l'animation dure. C'est exactement le message qu'on veut faire passer, et on l'obtient gratuitement avec une vitesse constante.

> **Vitesse constante ou proportionnelle ?** Si vous écriviez `_ratioFantome += (_ratio - _ratioFantome) * 5 * dt`, le rattrapage serait exponentiel : très rapide au début, interminable à la fin, et de durée identique quelle que soit la perte. Pour une barre de vie, la vitesse constante est meilleure : elle rend la durée de l'animation proportionnelle aux dégâts. Nous verrons en revanche que le compteur de score, lui, gagne à être exponentiel (section 38.22).

### Le tremblement à l'impact

Un raffinement à trois lignes, qui utilise les effets du chapitre 33. Quand la vie descend d'un coup, on secoue la barre :

```dart
/// À appeler depuis Joueur.subirDegats via le HUD.
void secouer() {
  if (children.whereType<MoveEffect>().isNotEmpty) {
    return;    // une secousse est déjà en cours : on n'empile pas
  }
  add(
    MoveEffect.by(
      Vector2(5, 0),
      EffectController(
        duration: 0.05,
        alternate: true,
        repeatCount: 3,
      ),
    ),
  );
}
```

`duration: 0.05` avec `alternate: true` et `repeatCount: 3` donne six allers-retours de 50 ms, soit 0,3 seconde de vibration. Le test `isNotEmpty` empêche l'empilement : deux secousses simultanées feraient dériver la barre hors de sa position, puisque `MoveEffect.by` est **relatif**.

> **Attention avec `HudMarginComponent`.** Ce composant repositionne l'élément à chaque redimensionnement de l'écran. Une secousse en cours pendant un redimensionnement sera écrasée. C'est sans conséquence en pratique — les deux événements ne se produisent jamais ensemble — mais c'est bon à savoir si vous observez un jour une barre qui « saute ».

---

## 38.20 — La barre d'énergie et la jauge d'attaque spéciale

### Une seconde jauge, une autre lecture

L'énergie n'est pas la vie. Elle se lit à l'envers :

| | Barre de vie | Barre d'énergie |
| --- | --- | --- |
| État idéal | pleine | pleine |
| Ce que le joueur surveille | **ne pas** tomber à zéro | **atteindre** le maximum |
| Sens de l'évolution | descend au combat | monte au combat |
| Ce qui se passe au maximum | rien | l'attaque spéciale se débloque |
| Couleur | vert / orange / rouge | bleu, puis doré quand pleine |

Comme l'énergie se remplit en frappant et en ramassant, elle récompense l'agressivité, là où la vie punit l'imprudence. Les deux jauges tirent le joueur dans des directions opposées : c'est précisément ce qui crée une tension intéressante.

### Le remplissage

L'énergie monte à trois occasions, toutes déjà écrites ou en cours d'écriture :

```dart
// Pièce ramassée (section 38.5)
joueur.gagnerEnergie(Piece.energie);        // +6

// Coup porté à un ennemi (chapitre 37, à compléter)
joueur.gagnerEnergie(4);

// Ennemi tué (chapitre 37, à compléter)
joueur.gagnerEnergie(10);
```

Il faut donc environ dix pièces, ou une poignée de combats, pour remplir la jauge. Ce rythme se règle en une constante — c'est tout l'intérêt de les avoir nommées.

### La classe `BarreEnergie`

Elle vit dans le même fichier que `BarreDeVie`, parce que c'est la même idée sous une autre couleur.

```dart
class BarreEnergie extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  BarreEnergie({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(largeur, hauteur),
          anchor: Anchor.topLeft,
        );

  static const double largeur = 132;
  static const double hauteur = 8;
  static const double bordure = 1.5;

  static const Color couleurCharge = Color(0xFF4A90D9);
  static const Color couleurPrete = Color(0xFFF2C14E);

  late final RectangleComponent _cadre;
  late final RectangleComponent _creux;
  late final RectangleComponent _remplissage;
  late final TextComponent _mention;

  bool _etaitPrete = false;

  static double get _utile => largeur - bordure * 2;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _cadre = RectangleComponent(
      size: Vector2(largeur, hauteur),
      paint: Paint()..color = const Color(0xFF15151E),
    );
    _creux = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFF20303F),
    );
    _remplissage = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(0.01, hauteur - bordure * 2),
      paint: Paint()..color = couleurCharge,
    );
    _mention = TextComponent(
      text: '',
      position: Vector2(largeur + 6, -2),
      textRenderer: TextPaint(
        style: const TextStyle(
          fontSize: 10,
          color: couleurPrete,
          fontWeight: FontWeight.bold,
        ),
      ),
    );

    await addAll([_cadre, _creux, _remplissage, _mention]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final Joueur? joueur = game.joueur;
    if (joueur == null) {
      return;
    }

    final double ratio =
        (joueur.energie / Joueur.energieMax).clamp(0.0, 1.0);
    final double l = (_utile * ratio).clamp(0.01, _utile);
    if ((_remplissage.size.x - l).abs() > 0.05) {
      _remplissage.size.setValues(l, _remplissage.size.y);
    }

    final bool prete = joueur.attaqueSpecialePrete;
    if (prete != _etaitPrete) {
      _etaitPrete = prete;
      _basculerEtatPret(prete);
    }
  }

  void _basculerEtatPret(bool prete) {
    _remplissage.paint.color = prete ? couleurPrete : couleurCharge;
    _mention.text = prete ? 'SPÉCIALE PRÊTE' : '';

    // On retire toujours l'ancienne pulsation avant d'en poser une neuve.
    for (final effet in _cadre.children.whereType<ScaleEffect>()) {
      effet.removeFromParent();
    }
    _cadre.scale = Vector2.all(1);

    if (prete) {
      _cadre.add(
        ScaleEffect.by(
          Vector2.all(1.06),
          EffectController(
            duration: 0.45,
            alternate: true,
            infinite: true,
            curve: Curves.easeInOut,
          ),
        ),
      );
    }
  }
}
```

### La bascule d'état, et pourquoi elle est protégée

Le champ `_etaitPrete` est le point crucial. Sans lui :

```dart
// CODE FAUX
if (joueur.attaqueSpecialePrete) {
  _cadre.add(ScaleEffect.by(...));   // AJOUTÉ 60 FOIS PAR SECONDE
}
```

**Résultat :**

```text
Au bout de deux secondes, 120 ScaleEffect infinis sont empilés
sur le même composant. Le cadre grossit sans limite, le jeu
ralentit, puis la barre disparaît de l'écran.
```

C'est le bug classique de la confusion entre **état** et **transition**. `attaqueSpecialePrete` est un **état**, vrai à chaque frame. Ce qu'on veut détecter, c'est la **transition** faux → vrai, qui ne se produit qu'une fois. Le patron « mémoriser la valeur précédente et comparer » est la façon la plus simple de transformer un état en transition.

```text
  frame :        1      2      3      4      5      6
  état   :     faux   faux   VRAI   VRAI   VRAI   faux
  précéd.:     faux   faux   faux   VRAI   VRAI   VRAI
  transition :   -      -    ▲OUI     -      -    ▼OUI

  On n'agit QUE sur les deux flèches.
```

Vous retrouverez ce patron partout : détecter qu'un joueur vient de toucher le sol, qu'un ennemi vient de mourir, qu'un niveau vient d'être terminé.

### Consommer l'énergie

L'attaque spéciale elle-même appartient au chapitre 39 (le boss lui donnera son utilité), mais le branchement est ici :

```dart
// lib/composants/joueur.dart
void attaquerSpecial() {
  if (!attaqueSpecialePrete) {
    return;
  }
  consommerEnergie();
  // ... l'onde de choc, chapitre 39
}
```

La jauge redescend à zéro, `_etaitPrete` détecte la transition inverse, la pulsation s'arrête et la couleur redevient bleue. Tout cela sans une ligne de plus : c'est le bénéfice d'avoir géré la transition proprement.

---

## 38.21 — `lib/hud/compteur_score.dart` avec `TextComponent`

### `TextComponent` et `TextPaint`

Flame sépare le **texte** et la **façon de le peindre** :

```dart
TextComponent({
  String? text,
  T? textRenderer,        // T extends TextRenderer
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
  ComponentKey? key,
});
```

`TextPaint` est l'implémentation intégrée de `TextRenderer`, bâtie sur le `TextPainter` de Flutter. Elle prend un `TextStyle` de Flutter, donc tout ce que vous savez du chapitre 19 s'applique : `fontSize`, `color`, `fontWeight`, `letterSpacing`, `shadows`.

Point capital, souvent ignoré : **la `size` du `TextComponent` est recalculée automatiquement à chaque changement de `text` ou de `textRenderer`**. Vous ne fixez jamais la taille d'un texte à la main.

### Un style lisible sur n'importe quel fond

Un HUD se superpose au jeu. Sous un texte blanc peuvent passer un mur clair, une torche, un ciel. Sans précaution, le texte devient illisible pendant une seconde — précisément la seconde où le joueur en avait besoin.

Trois parades, par ordre d'efficacité :

```dart
// 1. Une ombre portée : la moins coûteuse, la plus discrète.
static final TextPaint styleScore = TextPaint(
  style: const TextStyle(
    fontSize: 20,
    color: Color(0xFFF6F1E7),
    fontWeight: FontWeight.bold,
    shadows: [
      Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 3),
    ],
  ),
);

// 2. Un bandeau semi-transparent derrière le texte :
//    un RectangleComponent noir à 40 % d'opacité, ajouté avant le texte.

// 3. Un contour complet (deux TextComponent superposés, l'un décalé) :
//    le plus lisible, le plus coûteux. Réservé aux gros titres.
```

Nous prenons la première, suffisante et gratuite.

### La classe

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../config/constantes.dart';
import '../donjon_game.dart';

class CompteurScore extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurScore({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(180, 44),
          anchor: Anchor.topRight,
        );

  static final TextPaint _styleScore = TextPaint(
    style: const TextStyle(
      fontSize: 20,
      color: Color(0xFFF6F1E7),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 3),
      ],
    ),
  );

  static final TextPaint _styleCombo = TextPaint(
    style: const TextStyle(
      fontSize: 12,
      color: Color(0xFFF2C14E),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _texteScore;
  late final TextComponent _texteCombo;
  late final RectangleComponent _jaugeCombo;

  /// Score affiché, en virgule flottante pour permettre le défilement.
  double _valeurAffichee = 0;

  /// Dernier entier réellement écrit dans le TextComponent.
  int _dernierEcrit = -1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _texteScore = TextComponent(
      text: 'Score 00000',
      textRenderer: _styleScore,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 0),
    );

    _texteCombo = TextComponent(
      text: '',
      textRenderer: _styleCombo,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 24),
    );

    _jaugeCombo = RectangleComponent(
      position: Vector2(size.x, 40),
      size: Vector2(0.01, 3),
      anchor: Anchor.topRight,
      paint: Paint()..color = const Color(0xFFF2C14E),
    );

    await addAll([_texteScore, _texteCombo, _jaugeCombo]);
  }
  // update() : section 38.22
}
```

Deux points de placement méritent une explication.

**`anchor: Anchor.topRight` sur les enfants.** Le score grandit : « 00000 » puis « 01240 » puis « 118400 ». Ancré à gauche, il déborderait vers la droite et sortirait de l'écran. Ancré à droite, il pousse vers la gauche, à l'intérieur du cadre. Toute valeur numérique alignée sur un bord droit doit être ancrée `topRight`.

**`position: Vector2(size.x, 0)`.** Les coordonnées d'un enfant sont relatives au coin **haut-gauche** du parent, quelle que soit l'ancre du parent. Placer l'enfant en `x = size.x` le met donc sur le bord droit du parent, et son ancre `topRight` fait que son propre bord droit y est aligné.

```text
  LE CADRE DU COMPTEUR (180 × 44), ANCRÉ topRight À 16 PX DU BORD

  bord droit de l'écran ─────────────────────────────────┐
                                                         │
      ┌────────────────────────────────────┐  16 px      │
      │                        Score 01240 │◄────────────┤
      │                          × 2 combo │             │
      │                        ▮▮▮▮▮▮▮▮▮▮▮ │             │
      └────────────────────────────────────┘             │
      ↑                                   ↑              │
      x = 0 local                x = size.x local        │
```

### Afficher le combo

Le texte du combo n'existe que lorsqu'il y a quelque chose à dire :

```dart
final int m = game.multiplicateur;
final String voulu = m > 1 ? '× $m combo' : '';
if (_texteCombo.text != voulu) {
  _texteCombo.text = voulu;
}
```

Afficher « × 1 combo » en permanence serait du bruit : le joueur apprendrait à ignorer cette ligne, et il ignorerait donc aussi le « × 3 » quand il apparaîtrait. **Une information permanente cesse d'être une information.**

---

## 38.22 — Le score qui défile jusqu'à sa valeur cible

### Pourquoi ne pas afficher directement `game.score`

Un score qui saute de 1240 à 1290 en une frame ne se voit pas. Un score qui **défile** de 1240 à 1290 en un quart de seconde attire l'œil et donne une sensation de récompense. C'est un procédé vieux comme les machines à sous, et il fonctionne toujours.

Le principe : le HUD conserve **sa propre** valeur, distincte de celle du jeu, et la fait converger vers la valeur réelle.

```text
  game.score    ────────┐ 1290
                        │
                        │  (saut instantané, invisible)
                1240 ───┘

  _valeurAffichee
                1240 ───╮
                         ╰──╮
                            ╰───╮
                                ╰────╮
                                     ╰──────── 1290
                        │◄── 0,25 s ──►│
```

### L'interpolation exponentielle

```dart
@override
void update(double dt) {
  super.update(dt);

  final double cible = game.score.toDouble();

  if ((cible - _valeurAffichee).abs() < 0.5) {
    // Assez proche : on colle exactement, sinon on n'arrive jamais.
    _valeurAffichee = cible;
  } else {
    // On parcourt une FRACTION de l'écart restant à chaque frame.
    _valeurAffichee += (cible - _valeurAffichee) * min(1.0, 9 * dt);
  }

  final int arrondi = _valeurAffichee.round();
  if (arrondi != _dernierEcrit) {
    _dernierEcrit = arrondi;
    _texteScore.text = 'Score ${arrondi.toString().padLeft(5, '0')}';
  }

  // ... combo et jauge
}
```

Décortiquons les trois subtilités.

**`* min(1.0, 9 * dt)`.** Le facteur `9 * dt` vaut 0,15 à 60 images par seconde : on comble 15 % de l'écart par frame. Le `min(1.0, ...)` est une sécurité : si une frame dure exceptionnellement longtemps — chargement, changement d'onglet du navigateur — `9 * dt` pourrait dépasser 1 et le score **dépasserait** sa cible, produisant une oscillation. Bornez toujours un facteur d'interpolation à 1.

**Le seuil de 0,5.** Une interpolation exponentielle n'atteint jamais mathématiquement sa cible : elle s'en approche indéfiniment. Sans le seuil, `_valeurAffichee` resterait éternellement à 1289,9997 et le composant travaillerait pour rien à chaque frame. Le seuil « accroche » la valeur.

**Le garde `arrondi != _dernierEcrit`.** C'est le point de performance de la section. Affecter `text` sur un `TextComponent` déclenche la reconstruction complète d'un `TextPainter` de Flutter — mesure, mise en page, recalcul de la `size` du composant. À 60 frames par seconde, faire cela pour rien est un gaspillage mesurable. Comme le score affiché est un **entier**, on ne réécrit le texte que lorsque cet entier change.

```text
  frame  _valeurAffichee  arrondi   SANS garde   AVEC garde
    1        1247,05        1247      écrit       écrit
    2        1247,12        1247      écrit         -
    3        1247,19        1247      écrit         -
    4        1247,25        1247      écrit         -
    5        1247,31        1247      écrit         -
    6        1247,52        1248      écrit       écrit

  Le TextPainter est reconstruit 6 fois … contre 2 fois.
  Sur une montée de 300 points, l'écart se compte en centaines
  de reconstructions inutiles.
```

### `padLeft` et la stabilité visuelle

```dart
arrondi.toString().padLeft(5, '0')    // 42 -> « 00042 »
```

Sans remplissage, le texte passe de « Score 999 » à « Score 1000 » : sa largeur change, et comme il est ancré à droite, tout le bloc se décale. Avec `padLeft(5, '0')`, la largeur est constante jusqu'à 99 999 points. C'est une astuce de typographie de tableau de bord, empruntée aux bornes d'arcade.

`padLeft` est une méthode de `String` vue au **chapitre 2**.

### Le score qui saute quand il baisse

Un cas à ne pas oublier : au démarrage d'une nouvelle partie, `game.score` retombe à 0. L'interpolation ferait défiler le compteur de 12 400 à 0 en une seconde, pendant que le menu s'efface. Ce n'est pas souhaitable.

```dart
/// Force l'affichage sur la valeur réelle, sans défilement.
/// Appelée par DonjonGame au démarrage d'une partie.
void synchroniserImmediatement() {
  _valeurAffichee = game.score.toDouble();
  _dernierEcrit = -1;      // force la réécriture du texte
}
```

**Résultat :**

```text
Nouvelle partie : le compteur affiche « Score 00000 » immédiatement.
Ramassage d'une pièce : il défile de 00000 à 00010 en 0,2 s.
```

---

## 38.23 — Afficher les vies restantes

### Des symboles plutôt qu'un nombre

`Vies : 3` est correct mais lent à lire. Trois cœurs se comptent d'un coup d'œil, sans lire. C'est le principe de la **subitisation** : l'œil humain reconnaît instantanément des quantités jusqu'à quatre, sans compter.

```text
  ♥ ♥ ♥        se lit en 0,1 s, sans effort
  Vies : 3     demande de lire, décoder « 3 », comprendre
```

Comme `Constantes.viesDepart` vaut 3, on reste dans la zone de subitisation : les symboles sont le bon choix. Au-delà de cinq vies, il faudrait revenir à un nombre, ou à « ♥ × 8 ».

### Dessiner un cœur sans image

Un cœur, géométriquement, ce sont deux disques et un triangle.

```text
      ╭───╮ ╭───╮
     │  ●  │  ●  │      deux cercles de rayon r,
     │     │     │      centres écartés de r
      ╲    │    ╱
       ╲   │   ╱        un triangle qui rejoint la pointe
        ╲  │  ╱
         ╲ │ ╱
          ╲│╱
           ▼
```

```dart
class CompteurVies extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurVies({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(
            Constantes.viesDepart * (tailleCoeur + ecart),
            tailleCoeur,
          ),
          anchor: Anchor.topLeft,
        );

  static const double tailleCoeur = 12;
  static const double ecart = 5;

  static final Paint _plein = Paint()..color = const Color(0xFFE23F4C);
  static final Paint _vide = Paint()..color = const Color(0x552B1B22);

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    for (int i = 0; i < Constantes.viesDepart; i++) {
      final double x = i * (tailleCoeur + ecart);
      final bool acquise = i < game.vies;
      _dessinerCoeur(canvas, x, 0, tailleCoeur, acquise ? _plein : _vide);
    }
  }

  void _dessinerCoeur(
    Canvas canvas,
    double x,
    double y,
    double t,
    Paint peinture,
  ) {
    final double r = t / 4;

    // Les deux lobes.
    canvas.drawCircle(Offset(x + r, y + r + 1), r, peinture);
    canvas.drawCircle(Offset(x + t - r, y + r + 1), r, peinture);

    // La pointe.
    final Path pointe = Path()
      ..moveTo(x, y + r + 1)
      ..lineTo(x + t / 2, y + t)
      ..lineTo(x + t, y + r + 1)
      ..close();
    canvas.drawPath(pointe, peinture);
  }
}
```

### Toujours dessiner tous les emplacements

Le détail qui compte : on dessine **`Constantes.viesDepart` cœurs**, pas `game.vies` cœurs. Les vies perdues sont dessinées en gris translucide.

```text
  3 VIES              2 VIES              1 VIE
  ♥ ♥ ♥               ♥ ♥ ♡               ♥ ♡ ♡

  Le joueur voit toujours COMBIEN IL EN AVAIT.
  La largeur du bloc ne change jamais.
  Rien ne bouge à l'écran quand une vie est perdue.
```

Si on ne dessinait que les vies restantes, le bloc rétrécirait, les éléments voisins bougeraient, et le joueur perdrait le repère du total. C'est un principe général d'interface : **un conteneur d'information ne doit pas changer de taille en cours de jeu.**

### Pourquoi `render` et pas des composants enfants

On aurait pu créer trois `CircleComponent` et les retirer un à un. Le rendu direct est préférable ici pour trois raisons :

1. **Pas de synchronisation.** Le nombre affiché est lu dans `render`, donc toujours à jour. Aucun risque d'oublier de retirer un cœur.
2. **Moins d'objets dans l'arbre.** Trois cœurs, c'est trois composants de moins à parcourir 60 fois par seconde.
3. **Une forme composite.** Un cœur est fait de trois primitives. En faire un composant demanderait une classe supplémentaire pour un gain nul.

La règle générale : **un élément statique et purement décoratif se dessine ; un élément qui doit être animé, cliqué ou déplacé devient un composant.** Si vous vouliez qu'un cœur clignote à sa perte, il faudrait passer aux composants.

---

## 38.24 — Afficher les clés ramassées

### L'icône plus le nombre

Pour les clés, la subitisation ne s'applique pas : leur nombre est variable et peut monter. On affiche donc **une icône et un nombre**, format universel des inventaires.

```text
  ╭──╮══╗
 │ ○ │  ║  × 1
  ╰──╯══╝
```

```dart
class CompteurCles extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurCles({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(64, 18),
          anchor: Anchor.topRight,
        );

  static final Paint _metal = Paint()..color = const Color(0xFFF2E27A);
  static final Paint _creux = Paint()..color = const Color(0xFF7A6C28);

  static final TextPaint _style = TextPaint(
    style: const TextStyle(
      fontSize: 14,
      color: Color(0xFFF6F1E7),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _texte;
  int _dernierNombre = -1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    _texte = TextComponent(
      text: '× 0',
      textRenderer: _style,
      anchor: Anchor.centerRight,
      position: Vector2(size.x, size.y / 2),
    );
    await add(_texte);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final int nombre = game.joueur?.cles ?? 0;
    if (nombre == _dernierNombre) {
      return;
    }
    _dernierNombre = nombre;
    _texte.text = '× $nombre';

    if (nombre > 0) {
      _texte.add(
        ScaleEffect.by(
          Vector2.all(1.4),
          EffectController(duration: 0.12, alternate: true),
        ),
      );
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    // L'icône, à gauche du texte.
    const double h = 12;
    final double cy = size.y / 2;
    canvas.drawCircle(Offset(h / 2, cy), h / 2, _metal);
    canvas.drawCircle(Offset(h / 2, cy), h / 4, _creux);
    canvas.drawRect(
      Rect.fromLTWH(h / 2, cy - 1.6, 16, 3.2),
      _metal,
    );
    canvas.drawRect(Rect.fromLTWH(h / 2 + 10, cy, 2.4, 5), _metal);
    canvas.drawRect(Rect.fromLTWH(h / 2 + 14, cy, 2.4, 5), _metal);
  }
}
```

### Le `?? 0` et l'accès sûr

```dart
final int nombre = game.joueur?.cles ?? 0;
```

Une ligne, deux opérateurs du **chapitre 12** :

- `?.` — accès conditionnel : si `joueur` est nul, l'expression vaut `null` sans lever d'exception ;
- `??` — valeur par défaut : si le résultat est nul, on prend `0`.

Sans eux, il faudrait quatre lignes de `if`. Et surtout, sans eux, le HUD planterait à chaque transition de niveau, quand `joueur` est momentanément nul.

### Le petit sursaut

`ScaleEffect.by(Vector2.all(1.4), EffectController(duration: 0.12, alternate: true))` fait grossir le texte de 40 % puis revenir, en 0,24 seconde au total. C'est un **retour d'information** : le joueur regarde le donjon, pas le HUD ; un mouvement dans son champ périphérique attire son regard au bon endroit, au bon moment.

Le sursaut est déclenché **dans le bloc de changement**, après le `return` anticipé. Il ne peut donc se produire qu'à la transition, jamais en continu. C'est le même patron qu'à la section 38.20.

> **Attention à `ScaleEffect.by` sur un texte.** L'effet est relatif : ×1,4 puis retour à ×1/1,4. Si deux effets se chevauchent, l'échelle finale peut dériver légèrement. Avec `alternate: true` et une durée courte, le risque est nul en pratique. Pour un cas critique, préférez `ScaleEffect.to`, qui est absolu.

---

## 38.25 — Le HUD en overlay Flutter plutôt qu'en composant Flame : comparaison

### Les deux mondes

Vous avez maintenant un HUD entièrement construit en composants Flame. Il existe une alternative complète : construire le même HUD en **widgets Flutter**, affichés au-dessus du canvas par le système d'overlays du chapitre 35.

```dart
// La même barre de vie, en widget Flutter
GameWidget<DonjonGame>(
  game: jeu,
  initialActiveOverlays: const [Overlays.hud],
  overlayBuilderMap: {
    Overlays.hud: (BuildContext context, DonjonGame game) {
      return Padding(
        padding: const EdgeInsets.all(16),
        child: Align(
          alignment: Alignment.topLeft,
          child: SizedBox(
            width: 168,
            height: 14,
            child: LinearProgressIndicator(
              value: (game.joueur?.pv ?? 0) / Constantes.pvJoueurMax,
              backgroundColor: const Color(0xFF3B2530),
              color: const Color(0xFF5CBF54),
            ),
          ),
        ),
      );
    },
  },
);
```

Cinq lignes contre cent. Alors pourquoi ne l'avons-nous pas fait ainsi ?

### Le tableau de comparaison

| Critère | HUD en composants Flame | HUD en overlay Flutter |
| --- | --- | --- |
| **Où il vit** | `camera.viewport`, dans l'arbre de composants | par-dessus le canvas, dans l'arbre de widgets |
| **Rythme de mise à jour** | à chaque frame du jeu, dans `update(dt)` | à chaque `setState` / rebuild du widget |
| **Coût d'un rafraîchissement** | quasi nul : on modifie un champ | reconstruction d'un sous-arbre de widgets |
| **Animation à 60 fps** | naturelle, via `dt` et les effets Flame | possible, mais demande un `AnimationController` |
| **Widgets prêts à l'emploi** | aucun : tout se dessine | tout Material et Cupertino |
| **Texte multiligne, ellipse, i18n** | `TextBoxComponent`, limité | complet, avec `Text`, `RichText`, `Intl` |
| **Boutons, champs, listes déroulantes** | à écrire soi-même | natifs, accessibles, testés |
| **Accessibilité (lecteur d'écran)** | inexistante | intégrée à Flutter |
| **Réaction au clavier système, au focus** | manuelle | native |
| **Se met en pause avec `pauseEngine()`** | oui : l'arbre de composants est gelé | **non** : les widgets continuent de vivre |
| **Cohérence visuelle avec le jeu** | totale : même canvas, mêmes pixels | risque de décalage d'un pixel, de police différente |
| **Filtres, shaders, mélange avec le monde** | possibles | non |
| **Test unitaire** | `flame_test` | `flutter_test` et `WidgetTester` |
| **Coût quand rien ne change** | on parcourt quand même l'arbre | zéro si aucun rebuild |

### La règle de décision

```text
  ┌─────────────────────────────────────────────────────────────┐
  │  L'élément change-t-il PLUSIEURS FOIS PAR SECONDE ?          │
  └─────────────────────────────────────────────────────────────┘
          │ OUI                                │ NON
          ▼                                    ▼
   COMPOSANT FLAME                     ┌───────────────────────────┐
   barre de vie, score,                │ Contient-il des boutons,  │
   jauge d'énergie, combo,             │ du texte long, des menus ?│
   minuteur, minimap                   └───────────────────────────┘
                                         │ OUI          │ NON
                                         ▼              ▼
                                  OVERLAY FLUTTER   au choix
                                  menu, pause,     (prenez Flame
                                  game over,        pour rester
                                  inventaire,       homogène)
                                  options
```

Notre projet applique exactement cette règle :

| Élément | Technologie | Chapitre |
| --- | --- | --- |
| Menu principal | overlay Flutter | 35 |
| Barre de vie, énergie, score, vies, clés, objectif | composants Flame | 38 |
| Textes flottants de dégâts | composants Flame, dans le monde | 38 |
| Écran de pause | overlay Flutter | 40 |
| Écran de Game Over, de victoire | overlay Flutter | 40 |

### Le piège de la pause

Le point le plus vicieux du tableau mérite un développement, parce qu'il produit un bug que personne ne voit venir.

```dart
void mettreEnPause() {
  pauseEngine();               // gèle l'arbre de COMPOSANTS
  overlays.add(Overlays.pause);
}
```

`pauseEngine()` arrête la boucle de jeu. Aucun `update(dt)` n'est plus appelé, donc :

- le HUD en composants Flame **se fige** — c'est ce qu'on veut ;
- un HUD en widgets Flutter **continue de s'animer**, parce que les widgets ont leur propre cycle de rendu.

**Résultat avec un HUD en overlay :**

```text
Le joueur met le jeu en pause pendant que le score défile.
Le monde est figé, les ennemis sont immobiles…
mais le compteur de score continue de grimper tranquillement.
Le joueur croit à un bug. Il a raison : c'en est un.
```

Ce n'est pas insurmontable — il suffit de figer les animations Flutter à la pause — mais c'est du travail supplémentaire, et c'est un travail qu'on oublie de faire.

### Peut-on mélanger les deux ?

Oui, et c'est même la norme dans les jeux Flutter sérieux. La question n'est pas « lequel des deux ? » mais « lequel pour quoi ? ». Un seul conseil : **ne dupliquez jamais la même information dans les deux systèmes**. Une barre de vie en Flame et une en Flutter finiront par se contredire, et vous passerez une soirée à chercher laquelle a raison.

---

## 38.26 — Synchroniser le HUD et l'état du jeu

### Deux stratégies

Il existe exactement deux façons de tenir un affichage à jour.

```text
  PULL (tirer) — le HUD interroge le jeu

     Hud.update(dt)  ──── lit ────►  game.score
                                     game.vies
                                     game.joueur.pv

     « À chaque frame, je regarde où en sont les choses. »


  PUSH (pousser) — le jeu prévient le HUD

     game.ajouterScore(10)  ──── notifie ────►  Hud.surScoreChange(1250)

     « Quand quelque chose change, je préviens ceux que ça intéresse. »
```

| | Pull | Push |
| --- | --- | --- |
| Couplage | le HUD connaît le jeu | le jeu connaît ses observateurs |
| Code à écrire | une lecture dans `update` | un mécanisme de notification |
| Risque d'oubli | nul : on relit tout, tout le temps | réel : on oublie de notifier quelque part |
| Coût quand rien ne change | on lit quand même | zéro |
| Ordre de mise à jour | toujours cohérent | dépend de l'ordre des notifications |
| Adapté à | valeurs continues (PV, énergie, score) | événements rares (niveau terminé, objet rare) |

### Nous utilisons le pull, et voici pourquoi

Le HUD lit `game.score`, `game.vies`, `game.joueur?.pv` à chaque frame. C'est la stratégie **pull**, et pour un HUD de jeu d'action, elle est presque toujours la bonne :

1. **Le jeu tourne déjà à 60 frames par seconde.** L'`update` du HUD est appelé de toute façon. Lire cinq champs y coûte quelques nanosecondes.
2. **Aucune notification à oublier.** Si demain vous ajoutez une source de points — un coffre, un bonus de temps — le HUD la reflétera sans que vous touchiez à son code. Avec le push, il faudrait penser à notifier.
3. **Toujours cohérent.** Le HUD lit l'état à un instant donné, après que tout le monde a fini son `update`. Avec le push, un observateur peut être notifié au milieu d'une mise à jour et lire un état incohérent.
4. **Le HUD se fige avec `pauseEngine()`.** Puisqu'il n'est mis à jour que par `update`, la pause le fige. C'est gratuit.

L'inconvénient — travailler même quand rien ne change — est réel mais négligeable ici : nous avons **six** composants de HUD. Sur une liste d'inventaire de deux cents lignes, le raisonnement s'inverserait.

### Neutraliser le coût du pull

Le pull ne coûte cher que si la **lecture** entraîne un **travail**. Nous avons déjà appliqué la parade partout : on lit à chaque frame, mais on n'agit que si la valeur a changé.

```dart
// Patron appliqué dans TOUS les composants du HUD

final int nouveau = game.quelqueChose;
if (nouveau == _dernierConnu) {
  return;                       // rien à faire, on sort tout de suite
}
_dernierConnu = nouveau;
// ... ici seulement, le travail coûteux
```

| Composant | Ce qu'il lit chaque frame | Ce qu'il ne fait que si ça change |
| --- | --- | --- |
| `BarreDeVie` | `joueur.pv` | écrire `size` des rectangles (seuil 0,05 px) |
| `BarreEnergie` | `joueur.energie` | écrire `size`, changer la couleur, poser la pulsation |
| `CompteurScore` | `game.score` | réécrire le texte (seulement si l'entier change) |
| `CompteurVies` | `game.vies` | rien : le rendu direct est déjà minimal |
| `CompteurCles` | `joueur.cles` | réécrire le texte, jouer le sursaut |
| `IndicateurObjectif` | `joueur.cles`, `game.piecesRamassees` | réécrire le texte |

### Quand le push devient nécessaire

Trois cas où le pull ne suffit pas, et où il faut prévenir explicitement :

**1. Un événement ponctuel.** « Le joueur vient de perdre une vie » n'est pas un état, c'est un instant. On peut le déduire en mémorisant la valeur précédente — c'est le patron de la section 38.20 — mais un appel direct est plus clair :

```dart
// lib/donjon_game.dart
void perdreUneVie() {
  vies--;
  reinitialiserCombo();
  hud.barreDeVie.secouer();          // PUSH : un événement, pas un état
  // ...
}
```

**2. Une remise à zéro.** Au démarrage d'une partie, le HUD doit se recaler sans animation :

```dart
Future<void> demarrerPartie() async {
  score = 0;
  vies = Constantes.viesDepart;
  reinitialiserCombo();
  // ...
  hud.compteurScore.synchroniserImmediatement();   // PUSH
}
```

**3. Un changement structurel.** Au chargement d'un niveau (chapitre 39), le nombre total de pièces change. Le HUD doit le savoir pour afficher « 0 / 14 » :

```dart
Future<void> chargerNiveau(int index) async {
  // ...
  piecesDuNiveau = monde.children.whereType<Piece>().length;
  piecesRamassees = 0;
}
```

Ici, ce n'est même pas un push vers le HUD : on met simplement à jour un champ que le HUD lit déjà en pull. C'est souvent la solution la plus simple.

> **La bonne architecture est mixte.** Pull pour tout ce qui est continu, push pour les événements et les remises à zéro. Ce n'est pas un compromis mou : c'est le fait que « une valeur » et « un événement » sont deux choses différentes, qui appellent deux mécanismes différents.

### La jauge de combo, cas d'école du pull

Terminons par le morceau qui restait en suspens à la section 38.13 : la petite barre qui montre la fenêtre de combo se refermer.

```dart
// dans CompteurScore.update
final double fraction = game.fractionCombo;
final double l = (size.x * 0.5 * fraction).clamp(0.01, size.x);
if ((_jaugeCombo.size.x - l).abs() > 0.3) {
  _jaugeCombo.size.setValues(l, _jaugeCombo.size.y);
}
_jaugeCombo.paint.color = game.multiplicateur > 1
    ? const Color(0xFFF2C14E)
    : const Color(0x00000000);
```

Cette valeur change **à chaque frame** par nature : elle représente un temps qui s'écoule. Aucun push n'aurait de sens. C'est le cas parfait du pull, et le seuil est ici plus large — 0,3 pixel — parce que la précision n'a aucune importance sur une jauge de 3 pixels de haut.

---

## 38.27 — Les textes flottants de dégâts

### Ce qu'ils apportent

Un chiffre qui monte au-dessus d'un ennemi frappé donne trois informations d'un coup :

1. **Le coup a porté.** Sans lui, le joueur doute.
2. **Combien il a fait.** Utile dès que plusieurs armes existent.
3. **Où.** Le texte apparaît sur la cible, ce qu'un compteur de HUD ne peut pas faire.

C'est le retour d'information le plus rentable de tout le chapitre : quinze lignes de code pour un gain de lisibilité considérable.

### La classe

Elle vit dans `lib/composants/`, pas dans `lib/hud/`. C'est important : **ce texte est un objet du monde**, pas un élément d'interface (section 38.14).

```dart
// lib/composants/texte_flottant.dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

class TexteFlottant extends TextComponent {
  TexteFlottant({
    required String texte,
    required Vector2 position,
    Color couleur = const Color(0xFFF6F1E7),
    double taille = 11,
  }) : super(
          text: texte,
          position: position,
          anchor: Anchor.center,
          priority: 100,
          textRenderer: TextPaint(
            style: TextStyle(
              fontSize: taille,
              color: couleur,
              fontWeight: FontWeight.bold,
              shadows: const [
                Shadow(
                  color: Color(0xEE000000),
                  offset: Offset(0.8, 0.8),
                  blurRadius: 1.5,
                ),
              ],
            ),
          ),
        );

  static const double duree = 0.9;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // 1. Il monte, en ralentissant.
    add(
      MoveEffect.by(
        Vector2(0, -26),
        EffectController(duration: duree, curve: Curves.easeOut),
      ),
    );

    // 2. Il s'efface, après un court délai pendant lequel il est lisible.
    add(
      OpacityEffect.to(
        0,
        EffectController(duration: 0.5, startDelay: 0.4),
      ),
    );

    // 3. Il se retire tout seul, quoi qu'il arrive.
    add(RemoveEffect(delay: duree + 0.05));
  }
}
```

Quatre points techniques.

**`priority: 100`.** Le texte doit passer devant les murs, les ennemis et le joueur. Une priorité élevée le place au premier plan (chapitre 28). Sans elle, un « -15 » peut se retrouver caché derrière le gobelin qu'il concerne.

**`OpacityEffect` sur un `TextComponent`.** C'est une nouveauté de Flame **1.38.0** — les versions antérieures ne le permettaient pas. C'est l'une des raisons pour lesquelles ce cours fixe une version de référence.

**`startDelay: 0.4`.** Le texte reste pleinement opaque pendant 0,4 seconde, puis s'efface en 0,5. Un texte qui commence à disparaître immédiatement n'est jamais vraiment lisible.

**`RemoveEffect(delay: ...)`.** Ceinture et bretelles : même si un effet est interrompu, le composant s'auto-retire. Sans cela, un texte totalement transparent resterait dans l'arbre pour toujours — une fuite invisible qui finit par coûter cher.

### La fabrique sur `DonjonGame`

```dart
// lib/donjon_game.dart
void afficherTexteFlottant(
  Vector2 positionMonde,
  String texte,
  Color couleur, {
  double taille = 11,
}) {
  world.add(
    TexteFlottant(
      texte: texte,
      // clone() : sinon le texte suivrait l'objet qui l'a créé.
      position: positionMonde.clone()..y -= 8,
      couleur: couleur,
      taille: taille,
    ),
  );
}
```

`positionMonde.clone()` est **obligatoire**. `Vector2` est mutable : passer `position` directement ferait partager la même instance entre l'objet et son texte. Le texte suivrait alors le gobelin en fuite, ou pire, le `MoveEffect` du texte déplacerait le gobelin.

Le `..y -= 8` est la cascade du **chapitre 9** : elle décale le texte au-dessus de la cible et renvoie le vecteur.

### Les usages dans le jeu

```dart
// Dégâts infligés à un ennemi (chapitre 37, à compléter)
game.afficherTexteFlottant(position, '-${degats.round()}',
    const Color(0xFFFFFFFF));

// Coup critique
game.afficherTexteFlottant(position, 'CRITIQUE ${degats.round()}',
    const Color(0xFFFF6B3D), taille: 14);

// Dégâts subis par le joueur
game.afficherTexteFlottant(position, '-${degats.round()}',
    const Color(0xFFE23F4C));

// Soin
game.afficherTexteFlottant(position, '+12 PV', const Color(0xFF5CBF54));

// Ramassage (section 38.5)
game.afficherTexteFlottant(position, '+10', const Color(0xFFE8B04B));
```

Un code couleur cohérent tient en trois règles, et il faut s'y tenir dans tout le jeu :

| Couleur | Signification | Exemple |
| --- | --- | --- |
| Rouge | le joueur perd quelque chose | `-15` sur le héros |
| Vert | le joueur gagne de la vie | `+12 PV` |
| Doré | le joueur gagne des points | `+10` |
| Blanc | dégâts infligés à un ennemi | `-25` sur le gobelin |
| Orange vif | événement exceptionnel | `CRITIQUE`, `×4 COMBO` |

### Ne pas en abuser

Un texte flottant par coup, c'est parfait. Un texte flottant par frame de dégâts continus — le poison, le feu — c'est une bouillie illisible.

```dart
/// Empêche deux textes flottants de se superposer sur la même cible.
double _prochainTexteAutorise = 0;

void afficherDegatsAvecDelai(double degats) {
  final double maintenant = game.currentTime();
  if (maintenant < _prochainTexteAutorise) {
    return;
  }
  _prochainTexteAutorise = maintenant + 0.25;
  game.afficherTexteFlottant(position, '-${degats.round()}',
      const Color(0xFFFFFFFF));
}
```

Ce patron s'appelle un **throttle** : « au plus une fois toutes les 0,25 seconde ». Vous l'aviez déjà croisé pour la cadence de tir au chapitre 33.

---

## 38.28 — L'indicateur d'objectif

### Le problème du joueur perdu

Un joueur lancé dans une salle inconnue se pose une question : « qu'est-ce que je dois faire ? ». Si le jeu ne répond pas, il erre, s'ennuie, et arrête.

L'indicateur d'objectif répond en une ligne de texte, en permanence, sans jamais interrompre le jeu.

```text
  ÉTAT DU NIVEAU                     TEXTE AFFICHÉ

  Le joueur n'a pas la clé      ->   « Objectif : trouver la clé »
  Le joueur a la clé            ->   « Objectif : atteindre la porte »
  Toutes les pièces ramassées   ->   « Salle vidée ! »        (seconde ligne)
  Sinon                         ->   « Pièces 7 / 14 »        (seconde ligne)
```

### La classe

```dart
class IndicateurObjectif extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  IndicateurObjectif({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(220, 32),
          anchor: Anchor.topRight,
        );

  static final TextPaint _stylePrincipal = TextPaint(
    style: const TextStyle(
      fontSize: 12,
      color: Color(0xFFD9D2C4),
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  static final TextPaint _styleSecondaire = TextPaint(
    style: const TextStyle(
      fontSize: 11,
      color: Color(0xFF9C948A),
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _principal;
  late final TextComponent _secondaire;

  String _dernierPrincipal = '';
  String _dernierSecondaire = '';

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _principal = TextComponent(
      text: '',
      textRenderer: _stylePrincipal,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 0),
    );
    _secondaire = TextComponent(
      text: '',
      textRenderer: _styleSecondaire,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 16),
    );

    await addAll([_principal, _secondaire]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final int cles = game.joueur?.cles ?? 0;

    final String principal = cles > 0
        ? 'Objectif : atteindre la porte'
        : 'Objectif : trouver la clé';

    final String secondaire = game.piecesDuNiveau == 0
        ? ''
        : (game.piecesRamassees >= game.piecesDuNiveau
            ? 'Salle vidée !'
            : 'Pièces ${game.piecesRamassees} / ${game.piecesDuNiveau}');

    if (principal != _dernierPrincipal) {
      _dernierPrincipal = principal;
      _principal.text = principal;
      _principal.add(
        ScaleEffect.by(
          Vector2.all(1.15),
          EffectController(duration: 0.15, alternate: true),
        ),
      );
    }

    if (secondaire != _dernierSecondaire) {
      _dernierSecondaire = secondaire;
      _secondaire.text = secondaire;
    }
  }
}
```

Le patron est désormais familier : on **construit** la chaîne voulue à chaque frame — c'est bon marché — mais on ne l'**affecte** que si elle a changé.

### Les champs du jeu qu'il utilise

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 38

/// Nombre total de pièces posées dans le niveau courant.
/// Rempli au chargement du niveau (chapitre 39).
int piecesDuNiveau = 0;

/// Nombre de pièces déjà ramassées dans le niveau courant.
int piecesRamassees = 0;
```

En attendant le chapitre 39, on remplit `piecesDuNiveau` en comptant les pièces présentes dans le monde après avoir peuplé la salle :

```dart
piecesDuNiveau = world.children.whereType<Piece>().length;
piecesRamassees = 0;
```

`whereType<T>()` est la méthode de filtrage par type du **chapitre 14**, appliquée ici à l'ensemble des enfants d'un composant.

> **Attention au moment de l'appel.** `Component.add()` est asynchrone : les pièces ne sont pas dans `world.children` immédiatement après l'appel. Comptez-les **après** un `await world.addAll(...)`, ou comptez la liste que vous avez construite plutôt que l'arbre. La seconde solution est plus sûre :
>
> ```dart
> final pieces = <Piece>[ /* ... */ ];
> piecesDuNiveau = pieces.length;    // compté AVANT l'ajout
> await world.addAll(pieces);
> ```

### Ne pas transformer l'indicateur en tutoriel permanent

Deux lignes de texte, c'est le maximum. Au-delà, le joueur cesse de lire. Si vous avez besoin d'expliquer davantage, faites-le au moment opportun, par un texte flottant dans le monde, à l'endroit concerné :

```dart
// Quand le joueur touche une porte verrouillée sans clé (chapitre 39)
game.afficherTexteFlottant(
  porte.position,
  'Il vous faut une clé',
  const Color(0xFFBBBBBB),
);
```

Le message apparaît là où le joueur regarde, au moment où il en a besoin, et disparaît. C'est infiniment plus efficace qu'une ligne de plus dans un coin de l'écran.

---

## 38.29 — Le HUD sur mobile : ne pas cacher le joystick (rappel chapitre 30)

### Le problème

Au chapitre 30, section 30.26, vous avez placé un `JoystickComponent` en bas à gauche et un `HudButtonComponent` en bas à droite, tous deux dans `camera.viewport`. Ce sont donc des voisins directs de notre HUD, dans le même conteneur.

```text
  ÉCRAN MOBILE EN PAYSAGE — 800 × 400

  ┌──────────────────────────────────────────────────────────┐
  │ ▬▬▬▬▬▬▬▬▬▬▬▬                             Score 01240     │  <- zone HUD
  │ ▬▬▬▬▬▬▬▬                                   × 2 combo     │
  │ ♥ ♥ ♡                                     ╭─╮ × 1        │
  │                                    Objectif : la porte   │
  │                                                          │
  │                      LE JEU                              │
  │                                                          │
  │                                                          │
  │      ╭───╮                                    ╭───╮      │
  │     │  ●  │                                  │ ATT │     │  <- zone tactile
  │      ╰───╯                                    ╰───╯      │
  └──────────────────────────────────────────────────────────┘
    joystick                                    bouton
    bas-gauche                                bas-droite
```

La règle est simple : **le HUD occupe le haut, les contrôles occupent le bas**. Aucun recouvrement possible.

### Les trois erreurs classiques

**1. Placer une information en bas.** Une barre de vie en bas à gauche est un choix esthétique courant sur bureau. Sur mobile, elle passe sous le pouce gauche. Le joueur ne la voit **jamais**, parce que sa main la cache en permanence.

```text
  MAUVAIS                            BON

  ┌────────────────────┐             ┌────────────────────┐
  │                    │             │ ▬▬▬▬▬▬▬  la vie    │
  │                    │             │                    │
  │ ▬▬▬▬▬▬▬  la vie    │             │                    │
  │  ╭─╮               │             │  ╭─╮               │
  │ │ ● │  la main     │             │ │ ● │              │
  │  ╰─╯   du joueur   │             │  ╰─╯               │
  └────────────────────┘             └────────────────────┘
      ▲ cachée par le pouce
```

**2. Oublier les zones inaccessibles.** Encoches, coins arrondis, barre de gestes système : les bords d'un écran de téléphone ne sont pas tous utilisables.

```dart
// Marge de sécurité, plus généreuse sur mobile
static double margeSelonPlateforme() {
  // dart:io — attention, ne compile pas pour le Web tel quel.
  // Sur Web, utilisez kIsWeb de package:flutter/foundation.dart.
  return 16;   // bureau
  // return 28;  // mobile : encoches, coins arrondis, barre de gestes
}
```

Une valeur de 28 à 32 pixels sur mobile met le HUD à l'abri de tous les cas de figure courants.

**3. Se tromper de priorité entre le HUD et les contrôles.** Les deux vivent dans le même viewport. Si un élément de HUD passe devant un bouton, il peut **intercepter les taps**.

```dart
// lib/donjon_game.dart
await camera.viewport.addAll([
  Hud(priority: 10),                     // l'information : derrière
  joystick..priority = 20,               // les contrôles : devant
  boutonAttaque..priority = 20,
]);
```

Rappel du chapitre 28 : `priority` est un **z-index**, pas un ordre d'ajout. Une valeur plus grande est dessinée devant — et reçoit les événements en premier.

### Vérifier avec `debugMode`

Le moyen le plus rapide de contrôler l'absence de recouvrement :

```dart
// dans DonjonGame.onLoad, temporairement
debugMode = true;
```

Chaque composant affiche alors son rectangle englobant et sa position. Si un rectangle de HUD chevauche celui du joystick, vous le verrez immédiatement, sans avoir à tester au doigt.

### La liste de contrôle mobile

Avant de publier, vérifiez ces sept points. Ils vous éviteront la moitié des retours de testeurs :

| Point à vérifier | Comment |
| --- | --- |
| Le HUD est entièrement dans le tiers supérieur | capture d'écran, règle |
| Aucun élément de HUD sous 60 % de la hauteur | idem |
| Marge des bords ≥ 24 px | `debugMode` |
| Le joystick reste cliquable sur toute sa surface | test au doigt, aux quatre coins de sa zone |
| Le texte le plus petit fait ≥ 11 px | relire les `fontSize` |
| Tout est lisible sur fond clair **et** sur fond sombre | jouer dans deux salles différentes |
| En portrait, rien ne se chevauche | tourner l'appareil |

> **Le test du pouce.** Tenez votre téléphone à deux mains, en position de jeu, et regardez ce que vos pouces cachent. Environ 25 % de la surface d'un écran de téléphone en paysage est masquée par les mains. Le HUD doit vivre dans les 75 % restants. Aucune théorie ne remplace ce test de dix secondes.

---

## 38.30 — Ce que le projet fait à la fin de ce chapitre

### La salle d'essai

Le chapitre 39 apportera le vrai chargement de niveaux. En attendant, on peuple la salle du chapitre 35 avec les nouveaux objets, pour que tout soit jouable dès maintenant.

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 38

/// Pose les collectibles de la salle d'essai.
/// Remplacée au chapitre 39 par le chargement d'un vrai niveau.
Future<void> peuplerSalleDEssai() async {
  const double t = Constantes.tailleTuile;

  final List<Collectible> objets = <Collectible>[
    // Une rangée de pièces au sol, à ramasser en courant.
    for (int i = 0; i < 6; i++) Piece(position: Vector2(4 * t + i * t, 7 * t)),

    // Trois pièces sur la plateforme haute : il faut sauter.
    for (int i = 0; i < 3; i++)
      Piece(position: Vector2(11 * t + i * t, 4 * t)),

    // Deux potions, loin l'une de l'autre.
    Potion(position: Vector2(2 * t, 7 * t)),
    Potion(position: Vector2(17 * t, 4 * t)),

    // La clé, au bout du parcours.
    Cle(position: Vector2(19 * t, 7 * t)),
  ];

  // On compte AVANT l'ajout : add() est asynchrone (chapitre 28).
  piecesDuNiveau = objets.whereType<Piece>().length;
  piecesRamassees = 0;

  await world.addAll(objets);
}
```

Et l'appel, dans `demarrerPartie` :

```dart
Future<void> demarrerPartie() async {
  score = 0;
  vies = Constantes.viesDepart;
  reinitialiserCombo();

  await chargerSalle();              // chapitre 35 : décor, joueur, ennemis
  await peuplerSalleDEssai();        // chapitre 38 : les collectibles

  hud = Hud();
  await camera.viewport.add(hud);    // chapitre 38 : le HUD
  hud.compteurScore.synchroniserImmediatement();

  changerEtat(GameState.enJeu);
}
```

### La liste de contrôle

Lancez `flutter run`. Voici ce qui doit se produire, point par point. Si l'un d'eux échoue, la section indiquée vous dira où chercher.

| # | Ce qui doit se produire | Section |
| --- | --- | --- |
| 1 | Le menu s'affiche, « Jouer » démarre la partie | 35 |
| 2 | Le HUD apparaît : barre de vie pleine, énergie vide, 3 cœurs, score 00000 | 38.15 |
| 3 | Les pièces flottent, chacune à son rythme, et tournent sur elles-mêmes | 38.5, 38.6 |
| 4 | Le héros traverse une pièce : pop, éclats dorés, « +10 » qui monte | 38.7 |
| 5 | Le score défile de 00000 à 00010 en un quart de seconde | 38.22 |
| 6 | Quatre pièces en trois secondes : « × 2 combo » apparaît, la 4e vaut 20 | 38.13 |
| 7 | Trois secondes sans rien ramasser : le combo disparaît | 38.13 |
| 8 | Un gobelin touche le héros : la barre saute, une barre blanche traîne | 38.19 |
| 9 | Sous 50 % la barre passe à l'orange, sous 25 % au rouge | 38.18 |
| 10 | Une potion soigne, le texte affiche le soin **réel** | 38.8, 38.9 |
| 11 | Une potion à PV pleins donne « +5 » de score au lieu de rien | 38.8 |
| 12 | Ramasser des pièces remplit la jauge d'énergie | 38.20 |
| 13 | Jauge pleine : elle devient dorée, pulse, « SPÉCIALE PRÊTE » s'affiche | 38.20 |
| 14 | La clé fait passer le compteur à « × 1 », avec un sursaut | 38.24 |
| 15 | L'objectif passe de « trouver la clé » à « atteindre la porte » | 38.28 |
| 16 | La caméra suit le héros ; **le HUD ne bouge pas d'un pixel** | 38.14 |
| 17 | Redimensionner la fenêtre : le HUD se recale sur les bords | 38.16 |
| 18 | Perdre une vie : un cœur passe en gris, les autres restent en place | 38.23 |
| 19 | Aucune image n'est chargée : le dossier `assets/images/` est vide | — |

### L'arborescence du projet

```text
  donjon_de_dart/  ── ÉTAT À LA FIN DU CHAPITRE 38
  │
  └── lib/
      ├── main.dart
      ├── donjon_game.dart              ← MODIFIÉ : score, combo, joueur, HUD
      ├── config/
      │   ├── constantes.dart
      │   └── palette.dart
      ├── core/
      │   ├── game_state.dart
      │   ├── entite.dart
      │   └── sante.dart
      ├── composants/
      │   ├── joueur.dart               ← MODIFIÉ : soigner, énergie, clés
      │   ├── plateforme.dart
      │   ├── ennemi.dart
      │   ├── gobelin.dart
      │   ├── chauvesouris.dart
      │   ├── projectile.dart
      │   ├── collectible.dart          ← NOUVEAU
      │   ├── piece.dart                ← NOUVEAU
      │   ├── potion.dart               ← NOUVEAU
      │   ├── cle.dart                  ← NOUVEAU
      │   └── texte_flottant.dart       ← NOUVEAU
      ├── hud/                          ← NOUVEAU DOSSIER
      │   ├── hud.dart                  ← NOUVEAU
      │   ├── barre_de_vie.dart         ← NOUVEAU
      │   └── compteur_score.dart       ← NOUVEAU
      └── ecrans/
          └── menu_principal.dart
```

### Ce qui manque encore

Pour être honnête jusqu'au bout, voici ce que le joueur ne peut toujours pas faire :

| Manque | Chapitre qui le comble |
| --- | --- |
| Passer d'une salle à la suivante | 39 |
| Utiliser la clé sur une porte | 39 |
| Affronter un boss | 39 |
| Utiliser l'attaque spéciale que la jauge annonce | 39 |
| Entendre quoi que ce soit | 40 |
| Mettre le jeu en pause | 40 |
| Voir un écran de Game Over | 40 |
| Conserver son meilleur score entre deux lancements | 40 |

Le jeu est jouable ; il n'est pas encore fini. C'est exactement l'état attendu à la fin du chapitre 38.

---

## 38.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le joueur traverse les pièces sans rien déclencher | Les deux hitbox sont `passive` : deux passives ne se voient jamais | La hitbox du joueur doit rester `active` ; seul le collectible est `passive` |
| Le joueur est bloqué par une pièce comme par un mur | On a écrit du code de résolution de collision dans `onCollisionStart` du collectible | Un capteur ne corrige jamais de position : il se contente de réagir |
| Une pièce rapporte 20 points au lieu de 10 | Le joueur porte deux hitbox (corps + attaque) : `onCollisionStart` est appelée deux fois | Le verrou `_ramasse` **et** le passage de la hitbox en `CollisionType.inactive` |
| Le texte flottant affiche « +10 » mais le score monte de 20 | On lit `game.multiplicateur` **après** `ajouterScore`, qui a déjà incrémenté le combo | Calculer `valeur * game.multiplicateur` **avant** l'appel |
| Les particules disparaissent instantanément, ou rétrécissent | Le `ParticleSystemComponent` a été ajouté avec `add()` : il est enfant de l'objet retiré et subit son `ScaleEffect` | `parent?.add(ParticleSystemComponent(...))` |
| Les particules suivent l'objet au lieu de rester sur place | `position: position` partage la même instance de `Vector2`, qui est **mutable** | `position: position.clone()` |
| L'objet part en biais pendant son pop | Le `MoveEffect` de flottement, `infinite`, n'a pas été retiré | Retirer les `MoveEffect` avant d'ajouter le `SequenceEffect` |
| Toutes les pièces montent et descendent exactement ensemble | Effets identiques démarrés au même instant | `startDelay: hasard.nextDouble() * dureeFlottement` |
| `pv` monte à 113 après une potion | Aucun plafond dans `soigner` | `pv = (pv + points).clamp(0.0, Constantes.pvJoueurMax)` |
| `A value of type 'num' can't be assigned to a variable of type 'double'` | `clamp(0, 100)` avec des bornes entières sur un `double` | Écrire les bornes dans le même type : `clamp(0.0, 100.0)` |
| Un joueur mort ressuscite en ramassant une potion | `soigner` ne vérifie pas `pv <= 0` | Ajouter la garde `if (points <= 0 || pv <= 0) return;` |
| Le multiplicateur atteint ×27 et le score explose | Aucun plafond sur le multiplicateur | `min(1 + combo ~/ 3, multiplicateurMax)` |
| Le combo ne se ferme jamais | On décrémente `_tempsRestantCombo` mais on oublie de remettre `combo` à zéro | Passer par `reinitialiserCombo()`, qui remet **les deux** champs |
| La fenêtre de combo dure 0,05 s sur un écran 60 Hz | On décompte en **frames** au lieu de secondes | `_tempsRestantCombo -= dt;` — `dt` est en secondes (chapitre 20) |
| Le monde entier est figé, seul le HUD bouge | `super.update(dt)` a été oublié dans `DonjonGame.update` | Toujours appeler `super.update(dt)` en premier |
| La barre de vie glisse hors de l'écran quand le héros avance | Le HUD a été ajouté au `world` | `camera.viewport.add(hud)` |
| Le HUD est deux fois trop grand et flou | Il est dans le monde et subit le zoom de caméra (×2) | Idem : le mettre dans le `viewport` |
| Tous les éléments du HUD sont empilés au coin haut-gauche | Le conteneur `Hud` est un `PositionComponent` de taille nulle : c'est lui que les `HudMarginComponent` prennent pour référence | Faire hériter `Hud` de `Component`, sans taille, pour que la recherche remonte jusqu'au `Viewport` |
| Assertion « The parent of a HudMarginComponent needs to provide a size » | Aucun ancêtre n'implémente `ReadOnlySizeProvider` | Ajouter le HUD au `camera.viewport`, pas directement au `FlameGame` |
| Un élément demandé « collé à gauche » apparaît à droite | `margin: EdgeInsets.only(left: 0, ...)` : le test du code source est `margin.left != 0` | Ne jamais utiliser `0` comme marge ; utiliser `0.01`, ou une vraie marge |
| Assertion « Either margin or position must be defined » | `HudMarginComponent` construit sans `margin` ni `position` | Fournir l'un des deux |
| `A super initializer cannot be used with super-parameters` | On mélange `{super.margin}` et un appel explicite `: super(size: ...)` | Choisir : soit les super-paramètres seuls, soit un paramètre normal transmis dans le `super(...)` |
| La barre de vie rétrécit des deux côtés | Les rectangles enfants ont `Anchor.center` | Laisser l'ancre par défaut `Anchor.topLeft` sur les enfants d'une jauge |
| Des `NaN` ou un rendu erratique quand la vie tombe à zéro | Un `RectangleComponent` de largeur exactement 0 a ses quatre sommets confondus | `clamp(0.01, largeurUtile)` |
| L'objet disparu laisse un artefact ou fait planter un calcul | `ScaleEffect.to(Vector2.zero(), ...)` rend la matrice non inversible | `ScaleEffect.to(Vector2.all(0.1), ...)` |
| Le cadre de la jauge d'énergie grossit sans fin et le jeu ralentit | Un `ScaleEffect` infini est ajouté à chaque frame parce qu'on teste un **état** au lieu d'une **transition** | Mémoriser la valeur précédente (`_etaitPrete`) et n'agir qu'au changement |
| Le compteur de score consomme 15 % du temps de frame | `text` est réaffecté à chaque frame, ce qui reconstruit un `TextPainter` complet | Ne réécrire que si l'entier affiché a changé |
| Le score n'atteint jamais tout à fait sa cible | Une interpolation exponentielle converge sans jamais arriver | Ajouter un seuil : `if ((cible - v).abs() < 0.5) v = cible;` |
| Le score dépasse sa cible et oscille | Une frame très longue rend `9 * dt` supérieur à 1 | `min(1.0, 9 * dt)` |
| Le bloc de score se décale quand on passe de 999 à 1000 | La largeur du texte change | `toString().padLeft(5, '0')` et `anchor: Anchor.topRight` |
| Le HUD plante avec `LateInitializationError` entre deux niveaux | `late Joueur joueur` alors qu'il n'y a pas de joueur pendant la transition | Déclarer `Joueur? joueur` et écrire `final j = game.joueur; if (j == null) return;` |
| `The property 'pv' can't be unconditionally accessed because the receiver can be null` | On teste `game.joueur != null` puis on utilise `game.joueur.pv` : un champ d'instance n'est pas promu | Copier dans une variable locale avant de tester (chapitre 12) |
| Le « +10 » d'une pièce reste figé à l'écran quand la caméra bouge | Le `TexteFlottant` a été ajouté au `viewport` | Il appartient au **monde** : `world.add(...)` |
| Le « -15 » d'un gobelin est caché derrière lui | Priorité par défaut | `priority: 100` sur le `TexteFlottant` |
| Des textes flottants s'accumulent, illisibles, sur un ennemi empoisonné | Un texte est créé à chaque frame de dégâts continus | Un throttle : au plus un texte toutes les 0,25 s |
| Le HUD s'affiche mais le joystick ne répond plus au doigt | Un élément de HUD est dessiné devant le bouton et intercepte les taps | Priorité 10 pour le HUD, 20 pour les contrôles |
| La barre de vie est invisible sur mobile | Elle est en bas à gauche, sous le pouce du joueur | L'information vit en haut, les contrôles en bas |
| `piecesDuNiveau` vaut 0 alors qu'il y a des pièces | On compte `world.children` juste après `add()`, qui est asynchrone | Compter la liste locale **avant** l'ajout, ou après un `await` |
| Le compteur de score défile de 12 400 à 0 pendant le menu | Aucune remise à zéro immédiate au démarrage d'une partie | Appeler `synchroniserImmediatement()` dans `demarrerPartie` |

---

## 38.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `Collectible` | Classe **abstraite** : elle écrit les sept comportements communs et déclare abstraits les deux qui varient, `valeur` et `ramasser`. |
| `int get valeur` | Grandeur caractéristique **dans l'unité de la sous-classe** : points pour `Piece`, PV pour `Potion`, nombre de clés pour `Cle`. |
| Capteur (trigger) | Une hitbox `CollisionType.passive` qui ne bloque rien. Le passif n'est vu que par l'actif : le joueur doit rester actif. |
| Verrou de ramassage | Un `bool _ramasse` **plus** le passage de la hitbox en `inactive`. Deux protections, parce que `removeFromParent` n'est pas immédiat. |
| Flottement | `MoveEffect.by(Vector2(0, -4), EffectController(alternate: true, infinite: true))`, avec un `startDelay` aléatoire pour désynchroniser. |
| Pop | `SequenceEffect` de deux `ScaleEffect` — `easeOut` pour grossir, `easeIn` pour disparaître — avec `onComplete: removeFromParent`. |
| Effets qui survivent | Un effet plus long que l'objet qui le déclenche doit être ajouté au **parent**, jamais à l'objet. |
| `Vector2` mutable | Toujours `clone()` avant de partager une position entre deux composants. |
| `clamp` | Borne des deux côtés, s'utilise en expression, et exprime l'invariant. Bornez **à la source**, jamais à l'affichage. |
| Soin converti | Un soin inutile devient des points : aucun ramassage n'est jamais « perdu ». |
| `ajouterScore(int)` | Renvoie `void` : l'appelant calcule `valeur * game.multiplicateur` **avant** l'appel s'il veut afficher le gain. |
| Combo | Fenêtre glissante rechargée à chaque ramassage, refermée par `update(dt)`. Multiplicateur `min(1 + combo ~/ 3, 4)`. |
| HUD et viewport | `camera.viewport.add(hud)` : fixe à l'écran, hors du zoom. `world` = le donjon, `viewport` = l'écran. |
| `Hud extends Component` | Sans taille, pour que les `HudMarginComponent` remontent jusqu'au `Viewport` en cherchant un `ReadOnlySizeProvider`. |
| `HudMarginComponent` | Place par rapport aux bords, se recale au redimensionnement. Piège : `margin.left == 0` bascule l'ancrage à droite. |
| Jauge | Deux `RectangleComponent` superposés, ancre `topLeft`, on modifie la `size.x` du remplissage. |
| Barre fantôme | La barre exacte **saute**, une barre claire **glisse** derrière à vitesse constante : exactitude et lisibilité en même temps. |
| État contre transition | Un état est vrai à chaque frame ; une transition ne se produit qu'une fois. Mémoriser la valeur précédente pour n'agir qu'au changement. |
| `TextComponent` | Sa `size` est recalculée à chaque écriture de `text` : ne la fixez jamais, et n'écrivez `text` que si la valeur affichée a changé. |
| Score qui défile | Interpolation exponentielle bornée par `min(1.0, k * dt)`, avec un seuil d'accrochage et `padLeft` pour une largeur stable. |
| Vies | Dessiner **tous** les emplacements, les perdus en gris : le bloc ne change jamais de taille. |
| Pull ou push | Pull pour les valeurs continues, push pour les événements et les remises à zéro. |
| Overlay Flutter | À réserver aux menus, boutons et textes longs. Attention : les widgets **continuent de vivre** pendant `pauseEngine()`. |
| Textes flottants | Objets du **monde**, `priority: 100`, `OpacityEffect` (Flame 1.38.0), `RemoveEffect` de sécurité, throttle sur les dégâts continus. |
| HUD sur mobile | L'information en haut, les contrôles en bas, marge ≥ 24 px, priorité supérieure aux boutons tactiles. |
| Aucune image | Pièce, potion, clé, cœur et clé du HUD sont dessinés au `Canvas`. Le passage aux sprites ne touchera que `render`. |

---

## 38.33 — Code complet du chapitre

Cette section rassemble **tout** le code du chapitre. Les cinq premiers fichiers et les trois du dossier `hud/` sont nouveaux : copiez-les tels quels. Les deux derniers blocs sont des **ajouts** à des fichiers écrits aux chapitres 35 à 37 : insérez-les dans les fichiers existants, ne les remplacez pas.

### `lib/composants/collectible.dart`

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/particles.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../donjon_game.dart';
import 'joueur.dart';

/// Tout ce qui se ramasse dans le donjon.
///
/// La classe écrit le comportement commun : capteur passif, flottement,
/// verrou anti double ramassage, pop, particules et disparition.
/// Chaque sous-classe fournit `valeur`, `couleur`, `libelle` et `ramasser`.
abstract class Collectible extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Collectible({Vector2? position, Vector2? size})
      : super(
          position: position,
          size: size ?? Vector2.all(14),
          anchor: Anchor.center,
        );

  /// Générateur partagé par toute la hiérarchie.
  static final Random hasard = Random();

  bool _ramasse = false;
  bool get estRamasse => _ramasse;

  // ---- À fournir par les sous-classes --------------------------------

  /// Grandeur caractéristique, dans l'unité de la sous-classe.
  int get valeur;

  /// Couleur dominante : particules et texte flottant.
  Color get couleur;

  /// Nom lisible, pour le débogage.
  String get libelle;

  /// L'effet du ramassage. Appelée une seule fois.
  void ramasser(Joueur joueur);

  // ---- Réglages redéfinissables --------------------------------------

  /// Amplitude du flottement, en pixels.
  double get amplitudeFlottement => 4.0;

  /// Durée d'une montée, en secondes.
  double get dureeFlottement => 1.1;

  // ---- Cycle de vie ---------------------------------------------------

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Capteur : passif, il ne bloque rien et n'est vu que par l'actif.
    add(RectangleHitbox(collisionType: CollisionType.passive));

    // Flottement infini, désynchronisé objet par objet.
    add(
      MoveEffect.by(
        Vector2(0, -amplitudeFlottement),
        EffectController(
          duration: dureeFlottement,
          alternate: true,
          infinite: true,
          curve: Curves.easeInOut,
          startDelay: hasard.nextDouble() * dureeFlottement,
        ),
      ),
    );
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    if (_ramasse) {
      return;
    }
    if (other is! Joueur) {
      return;
    }

    _ramasse = true;
    ramasser(other);
    _jouerRamassage();
  }

  // ---- Effet de ramassage ---------------------------------------------

  void _jouerRamassage() {
    // 1. Le capteur ne doit plus répondre.
    for (final hitbox in children.whereType<ShapeHitbox>()) {
      hitbox.collisionType = CollisionType.inactive;
    }

    // 2. On coupe le flottement : sinon l'objet part en biais.
    for (final effet in children.whereType<MoveEffect>()) {
      effet.removeFromParent();
    }

    // 3. Les particules sont attachées au PARENT pour survivre à l'objet.
    parent?.add(
      ParticleSystemComponent(
        position: position.clone(),
        particle: Particle.generate(
          count: 10,
          lifespan: 0.5,
          generator: (int i) => AcceleratedParticle(
            speed: Vector2(
              hasard.nextDouble() * 120 - 60,
              hasard.nextDouble() * -110 - 30,
            ),
            acceleration: Vector2(0, 320),
            child: CircleParticle(
              radius: 1.5 + hasard.nextDouble() * 1.5,
              paint: Paint()..color = couleur,
            ),
          ),
        ),
      ),
    );

    // 4. Le pop, puis le retrait.
    add(
      SequenceEffect(
        [
          ScaleEffect.to(
            Vector2.all(1.6),
            EffectController(duration: 0.12, curve: Curves.easeOut),
          ),
          ScaleEffect.to(
            Vector2.all(0.1),
            EffectController(duration: 0.14, curve: Curves.easeIn),
          ),
        ],
        onComplete: removeFromParent,
      ),
    );
  }
}
```

### `lib/composants/piece.dart`

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

/// Une pièce d'or. `valeur` est exprimée en POINTS.
class Piece extends Collectible {
  Piece({Vector2? position})
      : super(position: position, size: Vector2.all(14));

  /// Points rapportés, avant multiplicateur de combo.
  static const int points = 10;

  /// Énergie rendue au joueur.
  static const double energie = 6;

  /// Vitesse de la fausse rotation, en radians par seconde.
  static const double vitesseRotation = 3.2;

  static final Paint _corps = Paint()..color = const Color(0xFFE8B04B);
  static final Paint _reflet = Paint()..color = const Color(0xFFFFE9A8);
  static final Paint _contour = Paint()
    ..color = const Color(0xFF8A5A16)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 1.5;

  double _phase = 0;

  @override
  int get valeur => points;

  @override
  Color get couleur => const Color(0xFFE8B04B);

  @override
  String get libelle => 'Pièce';

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    _phase = Collectible.hasard.nextDouble() * pi;
  }

  @override
  void update(double dt) {
    super.update(dt);
    _phase += dt * vitesseRotation;
  }

  @override
  void ramasser(Joueur joueur) {
    // Le gain est calculé AVANT ajouterScore, qui fait monter le combo.
    final int gain = valeur * game.multiplicateur;

    game.ajouterScore(valeur);
    game.piecesRamassees++;
    joueur.gagnerEnergie(energie);
    game.afficherTexteFlottant(position, '+$gain', couleur);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final Offset centre = Offset(size.x / 2, size.y / 2);
    final double facteur = cos(_phase).abs().clamp(0.15, 1.0);
    final double demiLargeur = (size.x / 2) * facteur;

    final Rect ovale = Rect.fromCenter(
      center: centre,
      width: demiLargeur * 2,
      height: size.y,
    );

    canvas.drawOval(ovale, _corps);
    canvas.drawOval(ovale, _contour);

    if (facteur > 0.35) {
      canvas.drawOval(
        Rect.fromCenter(
          center: centre.translate(-demiLargeur * 0.25, -size.y * 0.18),
          width: demiLargeur * 0.7,
          height: size.y * 0.34,
        ),
        _reflet,
      );
    }
  }
}
```

### `lib/composants/potion.dart`

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

/// Une potion de soin. `valeur` est exprimée en POINTS DE VIE.
class Potion extends Collectible {
  Potion({Vector2? position})
      : super(position: position, size: Vector2(14, 18));

  /// Points de vie rendus.
  static const int soin = 25;

  /// Points donnés quand le soin ne sert à rien.
  static const int pointsSiInutile = 5;

  static final Paint _verre = Paint()..color = const Color(0x66FFFFFF);
  static final Paint _liquide = Paint()..color = const Color(0xFFE04F5F);
  static final Paint _bouchon = Paint()..color = const Color(0xFF6D4C33);
  static final Paint _brillance = Paint()..color = const Color(0x99FFFFFF);

  @override
  int get valeur => soin;

  @override
  Color get couleur => const Color(0xFFE04F5F);

  @override
  String get libelle => 'Potion';

  @override
  void ramasser(Joueur joueur) {
    final double avant = joueur.pv;
    joueur.soigner(valeur.toDouble());
    final double rendu = joueur.pv - avant;

    if (rendu > 0) {
      game.afficherTexteFlottant(position, '+${rendu.round()} PV', couleur);
    } else {
      // PV déjà au maximum : on convertit plutôt que de gaspiller.
      final int gain = pointsSiInutile * game.multiplicateur;
      game.ajouterScore(pointsSiInutile);
      game.afficherTexteFlottant(position, '+$gain', const Color(0xFFE8B04B));
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final double l = size.x;
    final double h = size.y;

    final RRect corps = RRect.fromRectAndRadius(
      Rect.fromLTWH(0, h * 0.30, l, h * 0.70),
      Radius.circular(l * 0.35),
    );
    canvas.drawRRect(corps, _verre);

    canvas.save();
    canvas.clipRRect(corps);
    canvas.drawRect(Rect.fromLTWH(0, h * 0.50, l, h * 0.50), _liquide);
    canvas.restore();

    canvas.drawRect(
      Rect.fromLTWH(l * 0.32, h * 0.12, l * 0.36, h * 0.22),
      _verre,
    );
    canvas.drawRect(Rect.fromLTWH(l * 0.26, 0, l * 0.48, h * 0.14), _bouchon);
    canvas.drawRect(
      Rect.fromLTWH(l * 0.18, h * 0.42, l * 0.10, h * 0.40),
      _brillance,
    );
  }
}
```

### `lib/composants/cle.dart`

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

/// La clé d'une porte verrouillée. `valeur` est un NOMBRE DE CLÉS.
class Cle extends Collectible {
  Cle({Vector2? position})
      : super(position: position, size: Vector2(18, 9));

  /// Prime de score : trouver la clé est l'objectif du niveau.
  static const int primeScore = 50;

  static final Paint _metal = Paint()..color = const Color(0xFFF2E27A);
  static final Paint _ombre = Paint()..color = const Color(0xFF9A8A32);

  @override
  int get valeur => 1;

  @override
  Color get couleur => const Color(0xFFF2E27A);

  @override
  String get libelle => 'Clé';

  // Un objet rare flotte plus haut et plus lentement.
  @override
  double get amplitudeFlottement => 6.0;

  @override
  double get dureeFlottement => 1.5;

  @override
  void ramasser(Joueur joueur) {
    joueur.cles += valeur;

    final int gain = primeScore * game.multiplicateur;
    game.ajouterScore(primeScore);
    game.afficherTexteFlottant(position, 'Clé ! +$gain', couleur);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final double l = size.x;
    final double h = size.y;
    final double cy = h / 2;

    canvas.drawCircle(Offset(h / 2, cy), h / 2, _metal);
    canvas.drawCircle(Offset(h / 2, cy), h / 4, _ombre);

    canvas.drawRect(
      Rect.fromLTWH(h / 2, cy - h * 0.14, l - h / 2, h * 0.28),
      _metal,
    );
    canvas.drawRect(
      Rect.fromLTWH(l - h * 0.85, cy, h * 0.22, h * 0.45),
      _metal,
    );
    canvas.drawRect(
      Rect.fromLTWH(l - h * 0.35, cy, h * 0.22, h * 0.45),
      _metal,
    );
  }
}
```

### `lib/composants/texte_flottant.dart`

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

/// Un texte qui monte au-dessus d'un point du MONDE puis s'efface.
/// Ce n'est pas du HUD : il vit dans le donjon et suit la caméra.
class TexteFlottant extends TextComponent {
  TexteFlottant({
    required String texte,
    required Vector2 position,
    Color couleur = const Color(0xFFF6F1E7),
    double taille = 11,
  }) : super(
          text: texte,
          position: position,
          anchor: Anchor.center,
          priority: 100,
          textRenderer: TextPaint(
            style: TextStyle(
              fontSize: taille,
              color: couleur,
              fontWeight: FontWeight.bold,
              shadows: const [
                Shadow(
                  color: Color(0xEE000000),
                  offset: Offset(0.8, 0.8),
                  blurRadius: 1.5,
                ),
              ],
            ),
          ),
        );

  static const double duree = 0.9;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    add(
      MoveEffect.by(
        Vector2(0, -26),
        EffectController(duration: duree, curve: Curves.easeOut),
      ),
    );

    // OpacityEffect sur un composant texte : disponible depuis Flame 1.38.0.
    add(
      OpacityEffect.to(
        0,
        EffectController(duration: 0.5, startDelay: 0.4),
      ),
    );

    // Filet de sécurité : le composant se retire quoi qu'il arrive.
    add(RemoveEffect(delay: duree + 0.05));
  }
}
```

### `lib/hud/hud.dart`

```dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../donjon_game.dart';
import 'barre_de_vie.dart';
import 'compteur_score.dart';

/// Conteneur de tous les éléments d'interface.
///
/// IMPORTANT : c'est un `Component` sans taille. Les `HudMarginComponent`
/// enfants remontent alors jusqu'au `Viewport` pour connaître la taille
/// de l'écran. En faire un `PositionComponent` collerait tout le HUD
/// au coin haut-gauche.
class Hud extends Component with HasGameReference<DonjonGame> {
  Hud({super.priority = 10});

  /// Marge de sécurité par rapport aux bords.
  static const double marge = 16;

  late final BarreDeVie barreDeVie;
  late final BarreEnergie barreEnergie;
  late final CompteurVies compteurVies;
  late final CompteurScore compteurScore;
  late final CompteurCles compteurCles;
  late final IndicateurObjectif indicateurObjectif;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Colonne de gauche : l'état du héros.
    barreDeVie = BarreDeVie(
      margin: const EdgeInsets.only(left: marge, top: marge),
    );
    barreEnergie = BarreEnergie(
      margin: const EdgeInsets.only(left: marge, top: marge + 20),
    );
    compteurVies = CompteurVies(
      margin: const EdgeInsets.only(left: marge, top: marge + 36),
    );

    // Colonne de droite : l'état de la partie.
    compteurScore = CompteurScore(
      margin: const EdgeInsets.only(top: marge, right: marge),
    );
    compteurCles = CompteurCles(
      margin: const EdgeInsets.only(top: marge + 44, right: marge),
    );
    indicateurObjectif = IndicateurObjectif(
      margin: const EdgeInsets.only(top: marge + 70, right: marge),
    );

    await addAll([
      barreDeVie,
      barreEnergie,
      compteurVies,
      compteurScore,
      compteurCles,
      indicateurObjectif,
    ]);
  }

  /// Montre ou cache tout le HUD d'un coup (pause, fin de niveau).
  void definirVisibilite(bool visible) {
    for (final enfant in children.whereType<PositionComponent>()) {
      enfant.scale = visible ? Vector2.all(1) : Vector2.all(0.001);
    }
  }
}
```

### `lib/hud/barre_de_vie.dart`

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../composants/joueur.dart';
import '../config/constantes.dart';
import '../donjon_game.dart';

/// Barre de vie : une barre exacte qui saute, une barre fantôme qui traîne.
class BarreDeVie extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  BarreDeVie({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(largeur, hauteur),
          anchor: Anchor.topLeft,
        );

  static const double largeur = 168;
  static const double hauteur = 14;
  static const double bordure = 2;

  /// Vitesse de rattrapage du fantôme, en fraction de barre par seconde.
  static const double vitesseFantome = 0.55;

  static double get _utile => largeur - bordure * 2;

  late final RectangleComponent _cadre;
  late final RectangleComponent _creux;
  late final RectangleComponent _fantome;
  late final RectangleComponent _remplissage;

  double _ratio = 1;
  double _ratioFantome = 1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _cadre = RectangleComponent(
      size: Vector2(largeur, hauteur),
      paint: Paint()..color = const Color(0xFF15151E),
    );
    _creux = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFF3B2530),
    );
    _fantome = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFFF6F1E7),
    );
    _remplissage = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFF5CBF54),
    );

    await addAll([_cadre, _creux, _fantome, _remplissage]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final Joueur? joueur = game.joueur;
    if (joueur == null) {
      return;
    }

    // 1. La valeur exacte, tout de suite.
    _ratio = (joueur.pv / Constantes.pvJoueurMax).clamp(0.0, 1.0);

    // 2. Le fantôme ne traîne qu'à la DESCENTE.
    if (_ratioFantome > _ratio) {
      _ratioFantome = max(_ratio, _ratioFantome - vitesseFantome * dt);
    } else {
      _ratioFantome = _ratio;
    }

    // 3. Application.
    _appliquerLargeur(_remplissage, _utile * _ratio);
    _appliquerLargeur(_fantome, _utile * _ratioFantome);
    _remplissage.paint.color = _couleurSelonRatio(_ratio);
  }

  void _appliquerLargeur(RectangleComponent barre, double valeur) {
    final double l = valeur.clamp(0.01, _utile);
    if ((barre.size.x - l).abs() < 0.05) {
      return;
    }
    barre.size.setValues(l, barre.size.y);
  }

  static Color _couleurSelonRatio(double ratio) {
    if (ratio > 0.5) {
      return const Color(0xFF5CBF54);
    }
    if (ratio > 0.25) {
      return const Color(0xFFE0A128);
    }
    return const Color(0xFFE23F4C);
  }

  /// Petite secousse à l'impact. Appelée par `DonjonGame`.
  void secouer() {
    if (children.whereType<MoveEffect>().isNotEmpty) {
      return;
    }
    add(
      MoveEffect.by(
        Vector2(5, 0),
        EffectController(duration: 0.05, alternate: true, repeatCount: 3),
      ),
    );
  }
}

/// Jauge d'énergie : elle se remplit et débloque l'attaque spéciale.
class BarreEnergie extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  BarreEnergie({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(largeur, hauteur),
          anchor: Anchor.topLeft,
        );

  static const double largeur = 132;
  static const double hauteur = 8;
  static const double bordure = 1.5;

  static const Color couleurCharge = Color(0xFF4A90D9);
  static const Color couleurPrete = Color(0xFFF2C14E);

  static double get _utile => largeur - bordure * 2;

  late final RectangleComponent _cadre;
  late final RectangleComponent _creux;
  late final RectangleComponent _remplissage;
  late final TextComponent _mention;

  bool _etaitPrete = false;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _cadre = RectangleComponent(
      size: Vector2(largeur, hauteur),
      paint: Paint()..color = const Color(0xFF15151E),
    );
    _creux = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFF20303F),
    );
    _remplissage = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(0.01, hauteur - bordure * 2),
      paint: Paint()..color = couleurCharge,
    );
    _mention = TextComponent(
      text: '',
      position: Vector2(largeur + 6, -2),
      textRenderer: TextPaint(
        style: const TextStyle(
          fontSize: 10,
          color: couleurPrete,
          fontWeight: FontWeight.bold,
        ),
      ),
    );

    await addAll([_cadre, _creux, _remplissage, _mention]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final Joueur? joueur = game.joueur;
    if (joueur == null) {
      return;
    }

    final double ratio = (joueur.energie / Joueur.energieMax).clamp(0.0, 1.0);
    final double l = (_utile * ratio).clamp(0.01, _utile);
    if ((_remplissage.size.x - l).abs() > 0.05) {
      _remplissage.size.setValues(l, _remplissage.size.y);
    }

    // On agit sur la TRANSITION, pas sur l'état.
    final bool prete = joueur.attaqueSpecialePrete;
    if (prete != _etaitPrete) {
      _etaitPrete = prete;
      _basculerEtatPret(prete);
    }
  }

  void _basculerEtatPret(bool prete) {
    _remplissage.paint.color = prete ? couleurPrete : couleurCharge;
    _mention.text = prete ? 'SPÉCIALE PRÊTE' : '';

    for (final effet in _cadre.children.whereType<ScaleEffect>()) {
      effet.removeFromParent();
    }
    _cadre.scale = Vector2.all(1);

    if (prete) {
      _cadre.add(
        ScaleEffect.by(
          Vector2.all(1.06),
          EffectController(
            duration: 0.45,
            alternate: true,
            infinite: true,
            curve: Curves.easeInOut,
          ),
        ),
      );
    }
  }
}
```

### `lib/hud/compteur_score.dart`

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/text.dart';
import 'package:flutter/widgets.dart' hide Image;

import '../config/constantes.dart';
import '../donjon_game.dart';

/// Le score, avec défilement et affichage du combo.
class CompteurScore extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurScore({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(180, 44),
          anchor: Anchor.topRight,
        );

  static final TextPaint _styleScore = TextPaint(
    style: const TextStyle(
      fontSize: 20,
      color: Color(0xFFF6F1E7),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 3),
      ],
    ),
  );

  static final TextPaint _styleCombo = TextPaint(
    style: const TextStyle(
      fontSize: 12,
      color: Color(0xFFF2C14E),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _texteScore;
  late final TextComponent _texteCombo;
  late final RectangleComponent _jaugeCombo;

  double _valeurAffichee = 0;
  int _dernierEcrit = -1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _texteScore = TextComponent(
      text: 'Score 00000',
      textRenderer: _styleScore,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 0),
    );
    _texteCombo = TextComponent(
      text: '',
      textRenderer: _styleCombo,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 24),
    );
    _jaugeCombo = RectangleComponent(
      position: Vector2(size.x, 40),
      size: Vector2(0.01, 3),
      anchor: Anchor.topRight,
      paint: Paint()..color = const Color(0xFFF2C14E),
    );

    await addAll([_texteScore, _texteCombo, _jaugeCombo]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    // 1. Le défilement.
    final double cible = game.score.toDouble();
    if ((cible - _valeurAffichee).abs() < 0.5) {
      _valeurAffichee = cible;
    } else {
      _valeurAffichee += (cible - _valeurAffichee) * min(1.0, 9 * dt);
    }

    // 2. On n'écrit le texte que si l'entier affiché change.
    final int arrondi = _valeurAffichee.round();
    if (arrondi != _dernierEcrit) {
      _dernierEcrit = arrondi;
      _texteScore.text = 'Score ${arrondi.toString().padLeft(5, '0')}';
    }

    // 3. Le combo.
    final int m = game.multiplicateur;
    final String voulu = m > 1 ? '× $m combo' : '';
    if (_texteCombo.text != voulu) {
      _texteCombo.text = voulu;
    }

    // 4. La fenêtre de combo qui se referme.
    final double l = (size.x * 0.5 * game.fractionCombo).clamp(0.01, size.x);
    if ((_jaugeCombo.size.x - l).abs() > 0.3) {
      _jaugeCombo.size.setValues(l, _jaugeCombo.size.y);
    }
    _jaugeCombo.paint.color =
        m > 1 ? const Color(0xFFF2C14E) : const Color(0x00000000);
  }

  /// Recale l'affichage sans défilement (début de partie).
  void synchroniserImmediatement() {
    _valeurAffichee = game.score.toDouble();
    _dernierEcrit = -1;
  }
}

/// Les vies restantes, sous forme de cœurs. Tous les emplacements sont
/// dessinés : les vies perdues apparaissent en gris.
class CompteurVies extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurVies({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(
            Constantes.viesDepart * (tailleCoeur + ecart),
            tailleCoeur,
          ),
          anchor: Anchor.topLeft,
        );

  static const double tailleCoeur = 12;
  static const double ecart = 5;

  static final Paint _plein = Paint()..color = const Color(0xFFE23F4C);
  static final Paint _vide = Paint()..color = const Color(0x552B1B22);

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    for (int i = 0; i < Constantes.viesDepart; i++) {
      final double x = i * (tailleCoeur + ecart);
      final bool acquise = i < game.vies;
      _dessinerCoeur(canvas, x, 0, tailleCoeur, acquise ? _plein : _vide);
    }
  }

  void _dessinerCoeur(
    Canvas canvas,
    double x,
    double y,
    double t,
    Paint peinture,
  ) {
    final double r = t / 4;

    canvas.drawCircle(Offset(x + r, y + r + 1), r, peinture);
    canvas.drawCircle(Offset(x + t - r, y + r + 1), r, peinture);

    final Path pointe = Path()
      ..moveTo(x, y + r + 1)
      ..lineTo(x + t / 2, y + t)
      ..lineTo(x + t, y + r + 1)
      ..close();
    canvas.drawPath(pointe, peinture);
  }
}

/// Les clés en poche : une icône dessinée et un nombre.
class CompteurCles extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurCles({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(64, 18),
          anchor: Anchor.topRight,
        );

  static final Paint _metal = Paint()..color = const Color(0xFFF2E27A);
  static final Paint _creux = Paint()..color = const Color(0xFF7A6C28);

  static final TextPaint _style = TextPaint(
    style: const TextStyle(
      fontSize: 14,
      color: Color(0xFFF6F1E7),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _texte;
  int _dernierNombre = -1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    _texte = TextComponent(
      text: '× 0',
      textRenderer: _style,
      anchor: Anchor.centerRight,
      position: Vector2(size.x, size.y / 2),
    );
    await add(_texte);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final int nombre = game.joueur?.cles ?? 0;
    if (nombre == _dernierNombre) {
      return;
    }
    _dernierNombre = nombre;
    _texte.text = '× $nombre';

    if (nombre > 0) {
      _texte.add(
        ScaleEffect.by(
          Vector2.all(1.4),
          EffectController(duration: 0.12, alternate: true),
        ),
      );
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    const double h = 12;
    final double cy = size.y / 2;
    canvas.drawCircle(Offset(h / 2, cy), h / 2, _metal);
    canvas.drawCircle(Offset(h / 2, cy), h / 4, _creux);
    canvas.drawRect(Rect.fromLTWH(h / 2, cy - 1.6, 16, 3.2), _metal);
    canvas.drawRect(Rect.fromLTWH(h / 2 + 10, cy, 2.4, 5), _metal);
    canvas.drawRect(Rect.fromLTWH(h / 2 + 14, cy, 2.4, 5), _metal);
  }
}

/// Deux lignes qui disent au joueur ce qu'il doit faire.
class IndicateurObjectif extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  IndicateurObjectif({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(220, 32),
          anchor: Anchor.topRight,
        );

  static final TextPaint _stylePrincipal = TextPaint(
    style: const TextStyle(
      fontSize: 12,
      color: Color(0xFFD9D2C4),
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  static final TextPaint _styleSecondaire = TextPaint(
    style: const TextStyle(
      fontSize: 11,
      color: Color(0xFF9C948A),
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _principal;
  late final TextComponent _secondaire;

  String _dernierPrincipal = '';
  String _dernierSecondaire = '';

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _principal = TextComponent(
      text: '',
      textRenderer: _stylePrincipal,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 0),
    );
    _secondaire = TextComponent(
      text: '',
      textRenderer: _styleSecondaire,
      anchor: Anchor.topRight,
      position: Vector2(size.x, 16),
    );

    await addAll([_principal, _secondaire]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final int cles = game.joueur?.cles ?? 0;

    final String principal = cles > 0
        ? 'Objectif : atteindre la porte'
        : 'Objectif : trouver la clé';

    final String secondaire = game.piecesDuNiveau == 0
        ? ''
        : (game.piecesRamassees >= game.piecesDuNiveau
            ? 'Salle vidée !'
            : 'Pièces ${game.piecesRamassees} / ${game.piecesDuNiveau}');

    if (principal != _dernierPrincipal) {
      _dernierPrincipal = principal;
      _principal.text = principal;
      _principal.add(
        ScaleEffect.by(
          Vector2.all(1.15),
          EffectController(duration: 0.15, alternate: true),
        ),
      );
    }

    if (secondaire != _dernierSecondaire) {
      _dernierSecondaire = secondaire;
      _secondaire.text = secondaire;
    }
  }
}
```

### `lib/donjon_game.dart` — ajouts du chapitre 38

```dart
// ---- IMPORTS À AJOUTER EN HAUT DU FICHIER ---------------------------
// import 'dart:math';
// import 'package:flutter/widgets.dart' hide Image;
// import 'composants/cle.dart';
// import 'composants/collectible.dart';
// import 'composants/joueur.dart';
// import 'composants/piece.dart';
// import 'composants/potion.dart';
// import 'composants/texte_flottant.dart';
// import 'hud/hud.dart';

// ---- À AJOUTER DANS LA CLASSE DonjonGame ----------------------------

  /// Le héros de la partie. Nul entre deux niveaux : le HUD doit le tester.
  Joueur? joueur;

  /// Le HUD, monté dans le viewport de la caméra.
  late final Hud hud;

  int score = 0;
  int meilleurScore = 0;

  /// Ramassages enchaînés dans la fenêtre de combo.
  int combo = 0;

  /// Durée de la fenêtre de combo, en secondes.
  static const double dureeCombo = 3.0;

  /// Plafond du multiplicateur.
  static const int multiplicateurMax = 4;

  double _tempsRestantCombo = 0;

  /// Pièces du niveau courant : total et déjà ramassées.
  int piecesDuNiveau = 0;
  int piecesRamassees = 0;

  /// Un cran de plus tous les trois ramassages, plafonné.
  int get multiplicateur => min(1 + combo ~/ 3, multiplicateurMax);

  /// Fraction restante de la fenêtre de combo, de 0 à 1.
  double get fractionCombo => dureeCombo <= 0
      ? 0
      : (_tempsRestantCombo / dureeCombo).clamp(0.0, 1.0);

  /// Ajoute des points après multiplicateur et relance la fenêtre.
  void ajouterScore(int points) {
    if (points <= 0) {
      return;
    }
    score += points * multiplicateur;
    combo++;
    _tempsRestantCombo = dureeCombo;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  /// Ferme la fenêtre et remet le compteur à zéro.
  void reinitialiserCombo() {
    combo = 0;
    _tempsRestantCombo = 0;
  }

  /// Fait monter un texte au-dessus d'un point du MONDE.
  void afficherTexteFlottant(
    Vector2 positionMonde,
    String texte,
    Color couleur, {
    double taille = 11,
  }) {
    world.add(
      TexteFlottant(
        texte: texte,
        position: positionMonde.clone()..y -= 8,
        couleur: couleur,
        taille: taille,
      ),
    );
  }

  /// Pose les collectibles de la salle d'essai.
  /// Remplacée au chapitre 39 par le chargement d'un vrai niveau.
  Future<void> peuplerSalleDEssai() async {
    const double t = Constantes.tailleTuile;

    final List<Collectible> objets = <Collectible>[
      for (int i = 0; i < 6; i++) Piece(position: Vector2(4 * t + i * t, 7 * t)),
      for (int i = 0; i < 3; i++)
        Piece(position: Vector2(11 * t + i * t, 4 * t)),
      Potion(position: Vector2(2 * t, 7 * t)),
      Potion(position: Vector2(17 * t, 4 * t)),
      Cle(position: Vector2(19 * t, 7 * t)),
    ];

    // On compte AVANT l'ajout : add() est asynchrone.
    piecesDuNiveau = objets.whereType<Piece>().length;
    piecesRamassees = 0;

    await world.addAll(objets);
  }

  @override
  void update(double dt) {
    super.update(dt);      // INDISPENSABLE

    if (_tempsRestantCombo > 0) {
      _tempsRestantCombo -= dt;
      if (_tempsRestantCombo <= 0) {
        reinitialiserCombo();
      }
    }
  }

// ---- À MODIFIER : demarrerPartie (chapitre 35) ----------------------

  Future<void> demarrerPartie() async {
    score = 0;
    vies = Constantes.viesDepart;
    reinitialiserCombo();

    await chargerSalle();            // chapitre 35
    await peuplerSalleDEssai();      // chapitre 38

    hud = Hud();
    await camera.viewport.add(hud);  // LE HUD VA DANS LE VIEWPORT
    hud.compteurScore.synchroniserImmediatement();

    changerEtat(GameState.enJeu);
  }

// ---- À MODIFIER : perdreUneVie (chapitre 37) ------------------------

  void perdreUneVie() {
    vies--;
    reinitialiserCombo();            // AJOUT DU CHAPITRE 38
    hud.barreDeVie.secouer();        // AJOUT DU CHAPITRE 38
    if (vies <= 0) {
      changerEtat(GameState.gameOver);
    } else {
      // ... réapparition du joueur (chapitre 37)
    }
  }
```

### `lib/composants/joueur.dart` — ajouts du chapitre 38

```dart
// ---- À AJOUTER DANS LA CLASSE Joueur --------------------------------

  /// Clés en poche. Consommées par les portes verrouillées (chapitre 39).
  int cles = 0;

  /// Énergie courante, de 0 à [energieMax].
  double energie = 0;

  /// Plafond de la jauge d'énergie.
  static const double energieMax = 100;

  /// L'attaque spéciale est prête quand la jauge est pleine.
  bool get attaqueSpecialePrete => energie >= energieMax;

  /// Vrai si le joueur peut ouvrir une porte verrouillée.
  bool get possedeUneCle => cles > 0;

  /// Consomme une clé. Renvoie faux s'il n'y en avait pas.
  bool consommerUneCle() {
    if (cles <= 0) {
      return false;
    }
    cles--;
    return true;
  }

  /// Rend des points de vie, sans jamais dépasser le maximum.
  /// Un joueur mort ne se soigne pas.
  void soigner(double points) {
    if (points <= 0 || pv <= 0) {
      return;
    }
    pv = (pv + points).clamp(0.0, Constantes.pvJoueurMax);
  }

  /// Recharge la jauge d'énergie. Bornée, comme les PV.
  void gagnerEnergie(double points) {
    if (points <= 0) {
      return;
    }
    energie = (energie + points).clamp(0.0, energieMax);
  }

  /// Vide la jauge après une attaque spéciale.
  void consommerEnergie() {
    energie = 0;
  }

// ---- À MODIFIER : subirDegats (chapitre 37) -------------------------

  void subirDegats(double degats) {
    if (invincible || pv <= 0) {
      return;
    }
    pv = (pv - degats).clamp(0.0, Constantes.pvJoueurMax);
    game.reinitialiserCombo();       // AJOUT DU CHAPITRE 38
    game.afficherTexteFlottant(      // AJOUT DU CHAPITRE 38
      position,
      '-${degats.round()}',
      const Color(0xFFE23F4C),
    );
    // ... invincibilité temporaire et clignotement (chapitre 37)
  }

// ---- À MODIFIER : onMount ou onLoad ---------------------------------
// Le jeu doit connaître son héros pour que le HUD puisse le lire.

  @override
  void onMount() {
    super.onMount();
    game.joueur = this;
  }

  @override
  void onRemove() {
    if (game.joueur == this) {
      game.joueur = null;
    }
    super.onRemove();
  }
```

---

## 38.34 — Exercices

Les exercices vont du plus simple au plus complet. Les six premiers se font dans le projet « Donjon de Dart » tel qu'il est à la fin de ce chapitre.

### Exercice 1 — Le rubis (facile)

Créez `lib/composants/rubis.dart`, une sous-classe de `Collectible` qui rapporte **50 points**, est dessinée comme un losange rouge de 12 × 16 pixels, et flotte avec une amplitude de 5 pixels.

Posez-en un dans `peuplerSalleDEssai` et vérifiez qu'il rapporte bien 50 points, multiplicateur compris.

### Exercice 2 — La potion d'énergie (facile)

Créez `PotionEnergie`, une sous-classe de `Collectible` qui remplit **la moitié** de la jauge d'énergie du joueur (`Joueur.energieMax / 2`) au lieu de soigner.

Contraintes : le texte flottant doit indiquer l'énergie **réellement** gagnée, comme la potion de soin le fait pour les PV ; si la jauge est déjà pleine, convertissez en 5 points de score.

### Exercice 3 — Le compteur de pièces dans le HUD (facile)

Ajoutez à `lib/hud/compteur_score.dart` une classe `CompteurPieces` qui affiche, sous le score, le nombre de pièces ramassées depuis le début de la partie — pas depuis le début du niveau.

Contraintes : ajoutez le champ nécessaire à `DonjonGame` ; le texte ne doit être réécrit que lorsque le nombre change ; placez le compteur en haut à droite, sous l'indicateur d'objectif.

### Exercice 4 — La barre de vie chiffrée (moyen)

Ajoutez à `BarreDeVie` un `TextComponent` enfant qui affiche « 73 / 100 », centré sur la barre.

Contraintes : le texte doit se mettre à jour uniquement quand l'entier affiché change ; il doit rester lisible sur les trois couleurs de barre ; ajoutez un booléen statique `BarreDeVie.afficherLesChiffres` qui permet de le désactiver.

### Exercice 5 — Le combo qui clignote avant d'expirer (moyen)

Faites clignoter le texte « × 3 combo » pendant la dernière seconde de la fenêtre, pour avertir le joueur qu'il va la perdre.

Contraintes : n'ajoutez pas d'effet à chaque frame — souvenez-vous de la distinction entre état et transition ; le clignotement doit s'arrêter net si le joueur ramasse quelque chose et recharge la fenêtre.

### Exercice 6 — Le coffre (moyen)

Créez `Coffre`, un `Collectible` qui, une fois ramassé, fait apparaître **cinq pièces** projetées en éventail autour de lui.

Contraintes : les pièces doivent être ajoutées au **parent**, pas au coffre ; elles doivent partir avec un `MoveEffect` court avant de redevenir immobiles ; le coffre lui-même ne rapporte aucun point directement.

### Exercice 7 — L'aimant à pièces (difficile)

Donnez au joueur un rayon d'attraction de 48 pixels : toute pièce à l'intérieur de ce rayon vole vers lui.

Contraintes : n'utilisez pas de hitbox supplémentaire, mais un calcul de distance dans `update` ; l'attraction doit accélérer à mesure que la pièce se rapproche ; le ramassage doit continuer de passer par la collision normale.

### Exercice 8 — Le HUD compact en portrait (difficile)

Sur un écran plus haut que large, l'espace horizontal manque. Faites en sorte que, lorsque `size.x < size.y`, le HUD se réorganise : barre de vie et score sur la première ligne, le reste sur la seconde, et l'indicateur d'objectif masqué.

Contraintes : détectez le changement d'orientation dans `onGameResize` ; ne reconstruisez pas les composants, contentez-vous de changer leurs `margin`.

### Exercice 9 — Le score final détaillé (difficile)

Ajoutez à `DonjonGame` un suivi séparé des sources de points : pièces, ennemis, clés, bonus. À la fin de la partie, un objet `BilanPartie` doit pouvoir donner le détail.

Contraintes : `ajouterScore` garde sa signature `void ajouterScore(int points)` ; ajoutez une **seconde** méthode `void ajouterScoreDetaille(int points, SourceScore source)` que `ajouterScore` appelle avec une source par défaut ; utilisez une `enum` et une `Map`.

### Exercice 10 — Le HUD entièrement testable (difficile)

Écrivez, sans lancer le jeu, une fonction `String rendreHudEnTexte(DonjonGame jeu)` qui produit une représentation textuelle de l'état du HUD, utilisable dans un test.

Exemple de sortie attendue :

```text
PV      [██████████████░░░░░░]  73/100
ÉNERGIE [████████░░░░░░░░░░░░]  42/100
VIES    ♥ ♥ ♡
SCORE   01240   (× 2 combo)
CLÉS    × 1
OBJECTIF Objectif : atteindre la porte / Pièces 7 / 14
```

Contraintes : la fonction ne doit dépendre d'aucun composant Flame, seulement des champs de `DonjonGame` et de `Joueur` ; elle doit gérer le cas `joueur == null`.

---

## 38.35 — Corrections des exercices

### Correction 1

```dart
// lib/composants/rubis.dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

class Rubis extends Collectible {
  Rubis({Vector2? position})
      : super(position: position, size: Vector2(12, 16));

  static const int points = 50;

  static final Paint _corps = Paint()..color = const Color(0xFFD1305A);
  static final Paint _facette = Paint()..color = const Color(0xFFFF7FA0);

  @override
  int get valeur => points;

  @override
  Color get couleur => const Color(0xFFD1305A);

  @override
  String get libelle => 'Rubis';

  @override
  double get amplitudeFlottement => 5.0;

  @override
  void ramasser(Joueur joueur) {
    final int gain = valeur * game.multiplicateur;
    game.ajouterScore(valeur);
    game.afficherTexteFlottant(position, '+$gain', couleur);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    final Path losange = Path()
      ..moveTo(size.x / 2, 0)
      ..lineTo(size.x, size.y / 2)
      ..lineTo(size.x / 2, size.y)
      ..lineTo(0, size.y / 2)
      ..close();
    canvas.drawPath(losange, _corps);

    // Une facette claire sur la moitié gauche.
    final Path facette = Path()
      ..moveTo(size.x / 2, 0)
      ..lineTo(size.x / 2, size.y)
      ..lineTo(0, size.y / 2)
      ..close();
    canvas.drawPath(facette, _facette);
  }
}
```

**Explication :** l'exercice ne demande que trois choses : redéfinir `valeur`, `couleur` et `ramasser`, plus un `render`. Tout le reste — capteur passif, flottement désynchronisé, verrou, pop, particules — est hérité sans une ligne de code. C'est exactement le bénéfice de la classe abstraite de la section 38.2 : ajouter un collectible coûte quarante lignes, dont trente de dessin. `amplitudeFlottement` est redéfini par un simple getter, ce qui montre l'intérêt d'avoir exposé les réglages sous cette forme plutôt qu'en champs de constructeur.

---

### Correction 2

```dart
// lib/composants/potion_energie.dart
import 'package:flame/components.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';

class PotionEnergie extends Collectible {
  PotionEnergie({Vector2? position})
      : super(position: position, size: Vector2(14, 18));

  static const int pointsSiInutile = 5;

  static final Paint _liquide = Paint()..color = const Color(0xFF4A90D9);
  static final Paint _verre = Paint()..color = const Color(0x66FFFFFF);

  /// `valeur` est ici une QUANTITÉ D'ÉNERGIE.
  @override
  int get valeur => (Joueur.energieMax / 2).round();

  @override
  Color get couleur => const Color(0xFF4A90D9);

  @override
  String get libelle => 'Potion d\'énergie';

  @override
  void ramasser(Joueur joueur) {
    final double avant = joueur.energie;
    joueur.gagnerEnergie(valeur.toDouble());
    final double rendu = joueur.energie - avant;

    if (rendu > 0) {
      game.afficherTexteFlottant(position, '+${rendu.round()} EN', couleur);
    } else {
      final int gain = pointsSiInutile * game.multiplicateur;
      game.ajouterScore(pointsSiInutile);
      game.afficherTexteFlottant(position, '+$gain', const Color(0xFFE8B04B));
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    final RRect corps = RRect.fromRectAndRadius(
      Rect.fromLTWH(0, size.y * 0.30, size.x, size.y * 0.70),
      Radius.circular(size.x * 0.35),
    );
    canvas.drawRRect(corps, _verre);
    canvas.save();
    canvas.clipRRect(corps);
    canvas.drawRect(
      Rect.fromLTWH(0, size.y * 0.45, size.x, size.y * 0.55),
      _liquide,
    );
    canvas.restore();
  }
}
```

**Explication :** on réutilise à l'identique le patron « mesurer avant, mesurer après, comparer » de la section 38.8. C'est le seul moyen de connaître le gain réel, puisque `gagnerEnergie` borne en interne et ne renvoie rien. Notez que `valeur` est ici une quantité d'énergie : l'unité change encore une fois, ce qui confirme que ce getter n'a de sens que dans le contexte de sa sous-classe. La conversion en points quand la jauge est pleine applique la même politique de générosité que la potion de soin, ce qui donne au jeu un comportement cohérent : **aucun ramassage n'est jamais perdu**.

---

### Correction 3

```dart
// lib/donjon_game.dart — ajout
  /// Pièces ramassées depuis le début de la PARTIE (tous niveaux confondus).
  int piecesTotales = 0;
```

```dart
// lib/composants/piece.dart — dans ramasser
    game.piecesRamassees++;
    game.piecesTotales++;      // AJOUT
```

```dart
// lib/hud/compteur_score.dart — nouvelle classe
class CompteurPieces extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurPieces({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(90, 16),
          anchor: Anchor.topRight,
        );

  static final Paint _or = Paint()..color = const Color(0xFFE8B04B);

  static final TextPaint _style = TextPaint(
    style: const TextStyle(
      fontSize: 13,
      color: Color(0xFFF6F1E7),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xCC000000), offset: Offset(1, 1), blurRadius: 2),
      ],
    ),
  );

  late final TextComponent _texte;
  int _dernier = -1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    _texte = TextComponent(
      text: '× 0',
      textRenderer: _style,
      anchor: Anchor.centerRight,
      position: Vector2(size.x, size.y / 2),
    );
    await add(_texte);
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (game.piecesTotales == _dernier) {
      return;
    }
    _dernier = game.piecesTotales;
    _texte.text = '× ${game.piecesTotales}';
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawOval(
      Rect.fromLTWH(0, size.y / 2 - 5, 8, 10),
      _or,
    );
  }
}
```

```dart
// lib/hud/hud.dart — dans onLoad
    compteurPieces = CompteurPieces(
      margin: const EdgeInsets.only(top: marge + 104, right: marge),
    );
    // ... puis l'ajouter à addAll
```

**Explication :** deux compteurs cohabitent désormais et il ne faut pas les confondre. `piecesRamassees` est **local au niveau** : il est remis à zéro par `peuplerSalleDEssai` et sert à l'indicateur d'objectif « 7 / 14 ». `piecesTotales` est **global à la partie** et n'est jamais remis à zéro en cours de route. Chacun répond à une question différente, donc chacun mérite son champ. Le garde `if (... == _dernier) return;` applique le patron de la section 38.26 : on lit à chaque frame, on n'écrit qu'au changement.

---

### Correction 4

```dart
// lib/hud/barre_de_vie.dart — ajouts à BarreDeVie

  /// Permet de couper l'affichage chiffré (option d'accessibilité).
  static bool afficherLesChiffres = true;

  static final TextPaint _styleChiffres = TextPaint(
    style: const TextStyle(
      fontSize: 9,
      color: Color(0xFFFFFFFF),
      fontWeight: FontWeight.bold,
      shadows: [
        Shadow(color: Color(0xFF000000), offset: Offset(0, 0), blurRadius: 3),
        Shadow(color: Color(0xFF000000), offset: Offset(0, 0), blurRadius: 3),
      ],
    ),
  );

  late final TextComponent _chiffres;
  int _dernierPv = -1;

  // Dans onLoad, APRÈS les quatre rectangles :
  //   _chiffres = TextComponent(
  //     text: '',
  //     textRenderer: _styleChiffres,
  //     anchor: Anchor.center,
  //     position: Vector2(largeur / 2, hauteur / 2),
  //   );
  //   await add(_chiffres);

  void _mettreAJourChiffres(Joueur joueur) {
    if (!afficherLesChiffres) {
      if (_chiffres.text.isNotEmpty) {
        _chiffres.text = '';
      }
      return;
    }
    final int pv = joueur.pv.ceil();
    if (pv == _dernierPv) {
      return;
    }
    _dernierPv = pv;
    _chiffres.text = '$pv / ${Constantes.pvJoueurMax.round()}';
  }
```

Et dans `update`, juste après le calcul de `_ratio` :

```dart
    _mettreAJourChiffres(joueur);
```

**Explication :** trois points. Le texte est ajouté **en dernier** dans `onLoad`, donc dessiné par-dessus les quatre rectangles ; sans cela il serait recouvert par le remplissage. La lisibilité sur les trois couleurs est obtenue par **deux ombres noires floues superposées** — une seule ne suffit pas à faire un vrai contour, deux donnent un halo sombre convaincant pour un coût négligeable. Enfin, `pv.ceil()` plutôt que `pv.round()` : un joueur à 0,4 PV doit lire « 1 / 100 » et non « 0 / 100 », car il n'est pas mort. Un affichage qui annonce la mort avant qu'elle survienne est perçu comme un bug.

---

### Correction 5

```dart
// lib/hud/compteur_score.dart — dans CompteurScore

  /// Seuil d'alerte : dernière seconde de la fenêtre de combo.
  static const double seuilAlerte = 1.0 / DonjonGame.dureeCombo;

  bool _enAlerte = false;

  void _gererAlerteCombo() {
    final bool alerte =
        game.multiplicateur > 1 && game.fractionCombo < seuilAlerte;

    if (alerte == _enAlerte) {
      return;                    // ni début ni fin d'alerte : on ne fait rien
    }
    _enAlerte = alerte;

    for (final effet in _texteCombo.children.whereType<OpacityEffect>()) {
      effet.removeFromParent();
    }
    _texteCombo.setOpacity(1);

    if (alerte) {
      _texteCombo.add(
        OpacityEffect.to(
          0.15,
          EffectController(duration: 0.12, alternate: true, infinite: true),
        ),
      );
    }
  }
```

Appelé à la fin de `update`, après la mise à jour du texte de combo.

**Explication :** c'est le patron « état contre transition » de la section 38.20, appliqué une troisième fois. L'expression `alerte` est un **état** vrai à chaque frame de la dernière seconde ; sans la comparaison avec `_enAlerte`, on ajouterait soixante `OpacityEffect` infinis par seconde et le texte deviendrait invisible en clignotant à une fréquence absurde. Le nettoyage explicite des anciens effets **et** la remise de l'opacité à 1 sont indispensables : un effet retiré ne restaure pas la valeur qu'il modifiait. `OpacityEffect` sur un composant texte n'est possible que depuis Flame 1.38.0.

---

### Correction 6

```dart
// lib/composants/coffre.dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flutter/widgets.dart' hide Image;

import 'collectible.dart';
import 'joueur.dart';
import 'piece.dart';

class Coffre extends Collectible {
  Coffre({Vector2? position})
      : super(position: position, size: Vector2(22, 18));

  static const int nombreDePieces = 5;

  static final Paint _bois = Paint()..color = const Color(0xFF7A4E24);
  static final Paint _ferrure = Paint()..color = const Color(0xFFC9A227);

  @override
  int get valeur => nombreDePieces;

  @override
  Color get couleur => const Color(0xFFC9A227);

  @override
  String get libelle => 'Coffre';

  @override
  double get amplitudeFlottement => 2.0;

  @override
  void ramasser(Joueur joueur) {
    final Component? hote = parent;
    if (hote == null) {
      return;
    }

    for (int i = 0; i < nombreDePieces; i++) {
      // Éventail régulier : de -60° à +60° autour de la verticale.
      final double angle = -pi / 2 + (i / (nombreDePieces - 1) - 0.5) * (2 * pi / 3);
      final Vector2 depart = position.clone();
      final Vector2 arrivee = depart + Vector2(cos(angle), sin(angle)) * 34;

      final Piece piece = Piece(position: depart);
      hote.add(piece);
      piece.add(
        MoveEffect.to(
          arrivee,
          EffectController(duration: 0.35, curve: Curves.easeOutCubic),
        ),
      );
    }

    game.afficherTexteFlottant(position, 'Coffre !', couleur);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(Rect.fromLTWH(0, size.y * 0.3, size.x, size.y * 0.7), _bois);
    canvas.drawRect(Rect.fromLTWH(0, 0, size.x, size.y * 0.34), _bois);
    canvas.drawRect(
      Rect.fromLTWH(size.x * 0.42, size.y * 0.2, size.x * 0.16, size.y * 0.35),
      _ferrure,
    );
  }
}
```

**Explication :** le point critique est `hote.add(piece)` et non `add(piece)`. Le coffre est retiré 0,26 seconde après le ramassage ; des pièces ajoutées comme ses enfants partiraient avec lui et subiraient son `ScaleEffect`. C'est exactement la règle des particules de la section 38.7, appliquée à des composants ordinaires. On capture `parent` dans une variable locale avant la boucle, car `parent` est nullable et ne serait pas promu à l'intérieur d'une fermeture. `MoveEffect.to` est absolu : la pièce arrive à un point calculé, ce qui garantit un éventail régulier même si plusieurs coffres explosent en même temps. Enfin, le coffre ne fait aucun appel à `ajouterScore` : ce sont les cinq pièces qui rapporteront, une à une, en faisant monter le combo cinq fois.

---

### Correction 7

```dart
// lib/composants/piece.dart — ajouts

  /// Rayon d'attraction, en pixels.
  static const double rayonAimant = 48;

  /// Vitesse d'attraction de base, en pixels par seconde.
  static const double vitesseAimant = 90;

  @override
  void update(double dt) {
    super.update(dt);
    _phase += dt * vitesseRotation;

    if (estRamasse) {
      return;
    }

    final Joueur? joueur = game.joueur;
    if (joueur == null) {
      return;
    }

    final Vector2 vers = joueur.absoluteCenter - position;
    final double distance = vers.length;

    if (distance > rayonAimant || distance < 0.01) {
      return;
    }

    // Plus la pièce est proche, plus elle accélère : de ×1 à ×4.
    final double proximite = 1 - distance / rayonAimant;
    final double vitesse = vitesseAimant * (1 + proximite * 3);

    position += vers.normalized() * vitesse * dt;
  }
```

**Explication :** aucune hitbox supplémentaire n'est créée. On compare une **distance** à un rayon, ce qui est le test de collision cercle-cercle du chapitre 24 sous sa forme la plus simple. Trois gardes sont nécessaires : `estRamasse` empêche une pièce en train de « poper » de continuer à voler, `joueur == null` protège les transitions, et `distance < 0.01` évite une division par zéro dans `normalized()`. Le facteur `1 + proximite * 3` produit une accélération à l'approche : la pièce hésite au bord du rayon puis se précipite, ce qui est bien plus satisfaisant qu'une vitesse constante. Le ramassage lui-même n'est pas touché : il se produit toujours par collision, quand la pièce finit par toucher le joueur.

---

### Correction 8

```dart
// lib/hud/hud.dart — ajouts

  bool? _etaitEnPortrait;

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);
    if (!isLoaded) {
      return;
    }
    _appliquerDisposition(size.y > size.x);
  }

  void _appliquerDisposition(bool portrait) {
    if (portrait == _etaitEnPortrait) {
      return;
    }
    _etaitEnPortrait = portrait;

    if (portrait) {
      barreDeVie.margin = const EdgeInsets.only(left: marge, top: marge);
      barreEnergie.margin =
          const EdgeInsets.only(left: marge, top: marge + 20);
      compteurVies.margin =
          const EdgeInsets.only(left: marge, top: marge + 36);
      compteurScore.margin =
          const EdgeInsets.only(top: marge + 56, right: marge);
      compteurCles.margin =
          const EdgeInsets.only(top: marge + 84, right: marge);
      indicateurObjectif.scale = Vector2.all(0.001);   // masqué
    } else {
      barreDeVie.margin = const EdgeInsets.only(left: marge, top: marge);
      barreEnergie.margin =
          const EdgeInsets.only(left: marge, top: marge + 20);
      compteurVies.margin =
          const EdgeInsets.only(left: marge, top: marge + 36);
      compteurScore.margin = const EdgeInsets.only(top: marge, right: marge);
      compteurCles.margin =
          const EdgeInsets.only(top: marge + 44, right: marge);
      indicateurObjectif.scale = Vector2.all(1);
    }
  }
```

**Explication :** `margin` est un champ **public et modifiable** de `HudMarginComponent` : on peut donc réorganiser le HUD sans rien reconstruire. Le champ `_etaitEnPortrait` est un `bool?` et non un `bool` : sa valeur initiale `null` garantit que la première disposition sera appliquée, même si le jeu démarre en portrait. C'est encore le patron état/transition, avec une nuance : le troisième état `null` signifie « on ne sait pas encore ». La garde `if (!isLoaded) return;` est nécessaire parce que `onGameResize` est appelée **avant** `onLoad` dans le cycle de vie du chapitre 28 : les champs `late final` ne seraient pas encore initialisés.

---

### Correction 9

```dart
// lib/donjon_game.dart — ajouts

enum SourceScore { piece, ennemi, cle, bonus, niveau }

class BilanPartie {
  const BilanPartie(this.parSource, this.total);

  final Map<SourceScore, int> parSource;
  final int total;

  int pourcentage(SourceScore source) =>
      total == 0 ? 0 : ((parSource[source] ?? 0) * 100 / total).round();

  @override
  String toString() {
    final StringBuffer tampon = StringBuffer('Total : $total\n');
    for (final SourceScore s in SourceScore.values) {
      tampon.writeln(
        '  ${s.name.padRight(8)} ${(parSource[s] ?? 0).toString().padLeft(6)}'
        '  (${pourcentage(s)} %)',
      );
    }
    return tampon.toString();
  }
}
```

```dart
// dans DonjonGame

  final Map<SourceScore, int> _scoreParSource = <SourceScore, int>{};

  /// Signature imposée par le contrat : elle ne change pas.
  void ajouterScore(int points) =>
      ajouterScoreDetaille(points, SourceScore.bonus);

  void ajouterScoreDetaille(int points, SourceScore source) {
    if (points <= 0) {
      return;
    }
    final int gagnes = points * multiplicateur;
    score += gagnes;
    _scoreParSource[source] = (_scoreParSource[source] ?? 0) + gagnes;

    combo++;
    _tempsRestantCombo = dureeCombo;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  BilanPartie get bilan =>
      BilanPartie(Map<SourceScore, int>.unmodifiable(_scoreParSource), score);
```

Et, dans les appelants qui connaissent leur source :

```dart
// Piece.ramasser
game.ajouterScoreDetaille(valeur, SourceScore.piece);

// Cle.ramasser
game.ajouterScoreDetaille(primeScore, SourceScore.cle);
```

**Explication :** le contrat impose `void ajouterScore(int points)`, et on le respecte à la lettre — la méthode existe toujours, avec la même signature. Elle devient simplement une **délégation** vers la version détaillée, avec une source par défaut. Tout le code des chapitres 35 à 37 qui appelle `ajouterScore` continue de fonctionner sans modification : c'est le principe de compatibilité ascendante. `Map.unmodifiable` protège le bilan contre une modification accidentelle par l'appelant, et `(map[cle] ?? 0) + valeur` est l'idiome d'incrémentation d'une entrée éventuellement absente, vu au chapitre 6.

---

### Correction 10

```dart
// lib/hud/rendu_texte.dart
import '../composants/joueur.dart';
import '../config/constantes.dart';
import '../donjon_game.dart';

/// Dessine une jauge en caractères : [████████░░░░].
String _jauge(double ratio, {int largeur = 20}) {
  final double borne = ratio.clamp(0.0, 1.0);
  final int pleins = (borne * largeur).round();
  return '[${'█' * pleins}${'░' * (largeur - pleins)}]';
}

/// Représentation textuelle du HUD, indépendante de Flame.
/// Utilisable dans un test unitaire sans lancer le jeu.
String rendreHudEnTexte(DonjonGame jeu) {
  final StringBuffer t = StringBuffer();
  final Joueur? j = jeu.joueur;

  if (j == null) {
    t.writeln('PV      (aucun joueur)');
    t.writeln('ÉNERGIE (aucun joueur)');
  } else {
    final int pv = j.pv.ceil();
    final int pvMax = Constantes.pvJoueurMax.round();
    t.writeln(
      'PV      ${_jauge(j.pv / Constantes.pvJoueurMax)}  $pv/$pvMax',
    );
    final int en = j.energie.round();
    t.writeln(
      'ÉNERGIE ${_jauge(j.energie / Joueur.energieMax)}  '
      '$en/${Joueur.energieMax.round()}',
    );
  }

  final String coeurs = List<String>.generate(
    Constantes.viesDepart,
    (int i) => i < jeu.vies ? '♥' : '♡',
  ).join(' ');
  t.writeln('VIES    $coeurs');

  final String combo =
      jeu.multiplicateur > 1 ? '   (× ${jeu.multiplicateur} combo)' : '';
  t.writeln('SCORE   ${jeu.score.toString().padLeft(5, '0')}$combo');

  t.writeln('CLÉS    × ${j?.cles ?? 0}');

  final String principal = (j?.cles ?? 0) > 0
      ? 'Objectif : atteindre la porte'
      : 'Objectif : trouver la clé';
  final String secondaire = jeu.piecesDuNiveau == 0
      ? ''
      : ' / Pièces ${jeu.piecesRamassees} / ${jeu.piecesDuNiveau}';
  t.write('OBJECTIF $principal$secondaire');

  return t.toString();
}
```

**Explication :** cette fonction vaut bien plus qu'un exercice de style. En isolant la **logique d'affichage** de son **rendu graphique**, elle rend le HUD testable sans moteur, sans fenêtre et sans image : un test peut fixer `jeu.score = 1240`, appeler `rendreHudEnTexte` et comparer la chaîne obtenue à une chaîne attendue. C'est exactement la séparation logique/rendu défendue au chapitre 26, et c'est ce qui permettra les tests du chapitre 42. Le traitement de `joueur == null` n'est pas une précaution théorique : c'est l'état réel du jeu entre deux niveaux, et un test qui l'oublie passera à côté du plantage le plus probable. Notez enfin l'usage systématique de `?.` et `??`, qui condense en une ligne ce qui demanderait sinon plusieurs branches.

---

## Et maintenant ?

Votre donjon est enfin **bavard**. Il ne se contente plus de savoir : il montre. Les pièces flottent et tournent, les potions soignent sans jamais gaspiller, la clé attend au bout du parcours, le score défile, la barre de vie raconte les coups reçus, la jauge d'énergie promet une attaque spéciale et l'objectif rappelle en permanence ce qu'il faut faire — le tout sans un seul fichier image.

Il reste pourtant une frustration, et elle est délibérée. Cette clé que vous venez de ramasser n'ouvre rien. Cette jauge dorée qui pulse ne débloque encore aucune attaque. Cette salle unique, aussi bien remplie soit-elle, n'a ni entrée ni sortie.

Le chapitre 39 lève ces trois verrous. Vous y écrirez le format de carte en `List<String>` annoncé par le contrat, le chargeur qui le transforme en composants, la classe `Porte` qui consommera enfin votre clé, les transitions d'un niveau au suivant, la courbe de difficulté qui rend la salle 3 plus rude que la salle 1, et surtout le `Boss` : un ennemi à plusieurs phases, avec sa propre barre de vie — que vous saurez construire, puisque vous venez d'en écrire une.

Rendez-vous au chapitre suivant :

[39-PARTIE-2C—NIVEAUX-BOSS-ET-PROGRESSION.md](./39-PARTIE-2C—NIVEAUX-BOSS-ET-PROGRESSION.md)
