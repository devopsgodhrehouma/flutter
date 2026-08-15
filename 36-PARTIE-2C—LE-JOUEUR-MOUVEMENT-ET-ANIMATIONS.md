# PARTIE 2C — CONSTRUCTION DU JEU « DONJON DE DART »
# CHAPITRE 36 — LE JOUEUR : MOUVEMENT, SAUT ET ANIMATIONS

> **Niveau :** intermédiaire / avancé
> **Durée estimée :** 10 h
> **Pré-requis :** chapitre 35 (architecture du jeu, `DonjonGame`, `GameState`, overlays, menu principal), chapitre 23 (vélocité, gravité, saut), chapitre 24 (AABB et résolution de collision), chapitre 28 (composants, ancres, cycle de vie), chapitre 30 (clavier), chapitre 31 (caméra et monde), chapitre 32 (`CollisionCallbacks`), chapitre 33 (effets et timers)
> **Ce que vous saurez faire à la fin :** écrire un personnage de plateforme complet, agréable à jouer, avec gravité, saut variable, coyote time, jump buffering, machine à états d'animation et invincibilité temporaire, sans utiliser une seule image.
> **Version de Flame utilisée dans ce chapitre :** `flame: ^1.38.0`

---

## 36.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- écrire `Entite`, la classe de base commune au joueur, aux ennemis et au boss ;
- écrire un composant `Plateforme` solide, avec une variante traversable par le bas ;
- construire une salle de test entièrement en code, sans fichier de niveau et sans image ;
- écrire la classe `Joueur` et l'énumération `EtatJoueur` conformes au contrat de `_SPEC-JEU-2C.md` ;
- dessiner un personnage lisible avec de simples `RectangleComponent` ;
- choisir l'**ancre** d'un composant en connaissance de cause, et savoir ce que cela change dans les calculs ;
- intégrer une vélocité en position avec `dt`, appliquer la gravité et **limiter la vitesse de chute** ;
- lire le clavier avec `KeyboardHandler`, et distinguer une touche **maintenue** d'une touche **pressée à l'instant** ;
- donner du **game feel** au déplacement grâce à l'accélération et à la décélération ;
- retourner visuellement le personnage sans déplacer sa hitbox ;
- poser une `RectangleHitbox` et comprendre à quoi elle sert — et à quoi elle ne sert pas ;
- détecter de façon fiable si le personnage touche le sol ;
- **résoudre les collisions axe par axe**, la technique standard des jeux de plateforme ;
- implémenter un saut, puis un **saut variable** selon la durée d'appui ;
- implémenter le **coyote time** et le **jump buffering** ;
- ajouter un **double saut** activable par une simple constante ;
- gérer les **plateformes traversables par le bas** et la descente volontaire ;
- écrire une **machine à états d'animation** propre, avec une priorité d'états explicite ;
- représenter chaque état par une couleur et une forme, puis **substituer** de vraies animations ;
- écrire `attaquer()`, `subirDegats()` avec invincibilité clignotante, `mourir()` et la remise à zéro ;
- faire suivre le joueur par la caméra sans à-coups ;
- régler les constantes de game feel à partir d'un tableau de valeurs de départ.

---

## 36.1 — Où on en est et ce qu'on ajoute

### 36.1.1 — L'état du projet à la fin du chapitre 35

Au chapitre 35, vous avez posé les fondations : `lib/main.dart`, `lib/donjon_game.dart`, `lib/config/constantes.dart`, `lib/config/palette.dart`, `lib/core/game_state.dart` et `lib/ecrans/menu_principal.dart`.

| Ce qui existe | Rôle |
| --- | --- |
| `DonjonGame` | la classe `FlameGame` du jeu, avec `HasCollisionDetection` et `HasKeyboardHandlerComponents` |
| `GameState` | `chargement`, `menu`, `enJeu`, `pause`, `gameOver`, `victoire` |
| `changerEtat()` | met en pause le moteur, ajoute et retire les overlays |
| `demarrerPartie()` | remet score et vies à zéro, passe en `GameState.enJeu` |
| `Overlays` | les noms de chaînes des overlays |
| `Constantes`, `Palette` | les réglages numériques et les couleurs partagés |
| `MenuPrincipal` | le widget Flutter du menu, affiché en overlay |
| `monde`, `camera` | le `World` et le `CameraComponent` de `FlameGame`, déjà configurés |

Rappel du point de vocabulaire posé au chapitre 35, qui revient en permanence ici :

```dart
// lib/donjon_game.dart — extrait du chapitre 35
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  /// Alias lisible du World fourni par FlameGame.
  /// On écrit `game.monde.add(...)` plutôt que `game.world.add(...)`.
  World get monde => world;

  GameState etat = GameState.chargement;

  int score = 0;
  int vies = Constantes.viesDepart;
  int niveauCourant = 0;
  int meilleurScore = 0;
}
```

`monde` n'est pas un deuxième monde : c'est le `World` que `FlameGame` crée pour vous (chapitre 31), simplement exposé sous un nom français. De même, `camera` est le `CameraComponent` fourni par `FlameGame` ; on ne le remplace pas, on le configure.

> **À retenir.** Le chapitre 35 a livré un jeu qui affiche un menu et qui sait passer en `GameState.enJeu`. Mais quand on appuie sur « Jouer », l'écran est vide : il n'y a rien dans le monde. Ce chapitre remplit ce vide. Et il le remplit **sans un seul fichier image** : tout sera dessiné avec des `RectangleComponent` colorés, comme le prescrit la règle 1 de `_SPEC-JEU-2C.md`. Ce n'est pas une contrainte subie mais une méthode de travail — le prototypage en *blockout* : un saut qui n'est pas agréable avec des rectangles ne le sera pas davantage avec un joli sprite.

### 36.1.2 — Ce que le chapitre 36 ajoute

Trois fichiers nouveaux, trois fichiers modifiés :

| Fichier | Statut | Contenu |
| --- | --- | --- |
| `lib/core/entite.dart` | **nouveau** | `Entite`, la classe de base de tout ce qui bouge et tombe |
| `lib/composants/plateforme.dart` | **nouveau** | `Plateforme`, le décor solide |
| `lib/composants/joueur.dart` | **nouveau** | `EtatJoueur`, `Joueur`, `ZoneAttaque` |
| `lib/config/constantes.dart` | modifié | les constantes de game feel du joueur |
| `lib/config/palette.dart` | modifié | les couleurs des états du joueur et des plateformes |
| `lib/donjon_game.dart` | modifié | construction de la salle de test, création du joueur, suivi caméra |

Et voici l'arbre de composants visé à la fin du chapitre. Comparez-le avec celui du chapitre 28 : c'est le même principe, appliqué à un vrai jeu.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │        ARBRE DE COMPOSANTS À LA FIN DU CHAPITRE 36             │
  └────────────────────────────────────────────────────────────────┘

  DonjonGame                          (FlameGame)
  │
  ├── World  (game.monde)
  │   │
  │   ├── Plateforme   x N            (sol, murs, plateformes)
  │   │   ├── RectangleComponent      (le corps)
  │   │   ├── RectangleComponent      (le liseré du dessus)
  │   │   └── RectangleHitbox         (passive)
  │   │
  │   └── Joueur                      (Entite + KeyboardHandler + CollisionCallbacks)
  │       ├── PositionComponent       (« visuel », se retourne)
  │       │   ├── RectangleComponent  (le corps, couleur = état)
  │       │   ├── RectangleComponent  (l'œil, indique la direction)
  │       │   └── RectangleComponent  (l'écharpe, s'agite en marche)
  │       ├── RectangleHitbox         (la hitbox de contact)
  │       └── ZoneAttaque             (temporaire, seulement pendant un coup)
  │
  └── CameraComponent  (game.camera)
      ├── Viewfinder                  (zoom = Constantes.zoomCamera, suit le joueur)
      └── Viewport
```

---

## 36.2 — `lib/core/entite.dart` : la classe de base des entités

### 36.2.1 — Pourquoi une classe de base

Dans quelques chapitres, vous aurez un joueur, des gobelins, des chauves-souris, un boss. Tous ces objets ont en commun un ensemble de besoins :

```text
  Ce que partagent le joueur, le gobelin et le boss :

  - une position et une taille                  -> déjà dans PositionComponent
  - une VÉLOCITÉ (pixels par seconde)           -> à ajouter
  - une gravité qui les fait tomber             -> à ajouter
  - la connaissance du sol (auSol)              -> à ajouter
  - une direction de regard (gauche / droite)   -> à ajouter
  - une résolution de collision avec le décor   -> à ajouter
  - un accès au jeu (score, vies, état)         -> HasGameReference<DonjonGame>
```

Écrire tout cela deux fois (une fois dans `Joueur`, une fois dans `Ennemi`) serait une faute. Vous avez appris au chapitre 10 que l'héritage sert précisément à factoriser un comportement commun, et au chapitre 11 qu'une classe **abstraite** sert de socle sans être instanciable elle-même.

`Entite` est donc une classe abstraite qui hérite de `PositionComponent` et qui porte le mixin `HasGameReference<DonjonGame>`.

### 36.2.2 — La signature et le contrat

Souvenez-vous de la signature imposée par `_SPEC-JEU-2C.md` :

```dart
class Joueur extends PositionComponent
    with HasGameReference<DonjonGame>, KeyboardHandler, CollisionCallbacks {
```

En introduisant `Entite`, nous écrirons :

```dart
abstract class Entite extends PositionComponent
    with HasGameReference<DonjonGame> { ... }

class Joueur extends Entite with KeyboardHandler, CollisionCallbacks { ... }
```

C'est **strictement la même chose** du point de vue des types : `Joueur` est bien un `PositionComponent`, il porte bien `HasGameReference<DonjonGame>`, `KeyboardHandler` et `CollisionCallbacks`. Le contrat est respecté ; on a simplement rangé la partie commune dans une classe intermédiaire.

> **Remarque.** Vérifiez toujours ce raisonnement avant de « factoriser » quelque chose qui est imposé par une spécification. Ici, `Entite` s'insère **entre** `PositionComponent` et `Joueur` : rien de ce que promet la spécification n'est perdu.

### 36.2.3 — Les champs d'une entité

```dart
abstract class Entite extends PositionComponent
    with HasGameReference<DonjonGame> {
  Entite({super.position, super.size, super.priority})
      : super(anchor: Anchor.topLeft);

  /// Vitesse en pixels par seconde. x > 0 = vers la droite, y > 0 = vers le bas.
  Vector2 velocite = Vector2.zero();

  /// Vrai si l'entité repose sur une plateforme à cette frame.
  bool auSol = false;

  /// 1 = regarde à droite, -1 = regarde à gauche.
  int direction = 1;

  /// Vrai pendant une descente volontaire à travers une plateforme.
  bool traverseLesPlateformes = false;

  // Accesseurs géométriques : gauche, droite, haut, bas, centre, boite.
  Rect get boite => Rect.fromLTWH(position.x, position.y, size.x, size.y);
}
```

Trois décisions sont prises ici.

**L'ancre est forcée à `Anchor.topLeft`.** La section 36.8 explique pourquoi ; retenez que `position` **est** alors le coin haut-gauche, ce qui rend `boite` — et donc toute la collision — trivial à écrire.

**`velocite` est un `Vector2` et non deux `double`.** C'est la convention du chapitre 23 et de tout Flame. Attention : `Vector2` est **mutable**. Ne partagez jamais la même instance entre deux entités ; pour copier, utilisez `.clone()`.

**`direction` est un `int` valant `1` ou `-1`, pas un `bool`.** Parce qu'on le multiplie : `direction * portee` donne directement le bon côté sans `if`.

### 36.2.4 — La gravité mutualisée

```dart
  /// Ajoute la gravité à la vélocité verticale, puis plafonne la chute.
  void appliquerGravite(double dt, {double multiplicateur = 1.0}) {
    velocite.y += Constantes.gravite * multiplicateur * dt;
    if (velocite.y > Constantes.vitesseMaxChute) {
      velocite.y = Constantes.vitesseMaxChute;
    }
  }
```

Le paramètre `multiplicateur` sert dès la section 36.20 : on fera tomber le joueur **plus vite** quand il redescend que quand il monte. C'est un réglage de game feel classique, et il est déjà prévu ici.

Il manque encore à `Entite` la résolution de collision, écrite en 36.18. Le fichier complet figure en 36.CC.

> **Remarque sur les imports.** `core/entite.dart` importe `composants/plateforme.dart`. C'est une dépendance de `core` vers `composants`, ce qui peut surprendre. On l'assume : le décor solide est une notion aussi fondamentale que la gravité dans un jeu de plateforme. L'alternative — passer la liste des plateformes en paramètre à chaque appel — alourdirait tout le code du chapitre pour un gain de pureté théorique.

---

## 36.3 — `lib/composants/plateforme.dart` : le décor solide

### 36.3.1 — Ce qu'est une plateforme dans ce jeu

Une plateforme est un rectangle immobile. Elle ne bouge pas, ne tombe pas, n'a pas de vélocité : elle **n'hérite donc pas d'`Entite`**. C'est un simple `PositionComponent`.

Elle a exactement deux responsabilités :

1. **être visible** — un rectangle coloré, plus un liseré clair sur le dessus pour que l'œil comprenne où on peut poser les pieds ;
2. **être solide** — exposer sa `boite` pour que les entités puissent l'éviter.

Elle porte aussi une `RectangleHitbox` en `CollisionType.passive`. Attention : cette hitbox ne sert **pas** à empêcher le joueur de traverser (nous le ferons à la main en 36.18). Elle servira aux chapitres suivants, par exemple pour qu'un projectile détecte un mur.

### 36.3.2 — Solide ou traversable

Le format de niveau de `_SPEC-JEU-2C.md` distingue deux caractères : `'#'` (mur) donne `Plateforme(traversable: false)`, `'='` donne `Plateforme(traversable: true)`.

Une plateforme traversable — on dit aussi *one-way platform* — est un classique du genre : on monte au travers par le bas, on se pose dessus en descendant, et l'on peut redescendre volontairement.

```text
    SOLIDE '#'                       TRAVERSABLE '='

       ██                                ██
       ██  <- bloque                     ██  <- bloque par le haut
    ███████████                       ═══════════
       ↑                                  ↑
    bloque aussi                       ne bloque PAS
    par en dessous                     par en dessous
```

Le champ est donc un simple `bool traversable`, lu par la résolution de collision.

### 36.3.3 — Le code

Le fichier complet figure en 36.CC. Voici sa charpente.

```dart
class Plateforme extends PositionComponent {
  Plateforme({
    required Vector2 position,
    required Vector2 size,
    this.traversable = false,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  /// Construit une plateforme à partir de coordonnées en TUILES (voir 36.CC).
  factory Plateforme.tuiles(int colonne, int ligne, int nombreDeColonnes,
      {int nombreDeLignes = 1, bool traversable = false});

  final bool traversable;

  Rect get boite => Rect.fromLTWH(position.x, position.y, size.x, size.y);

  @override
  Future<void> onLoad() async {
    // Le corps, puis le liseré du dessus, puis la hitbox.
    await add(RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.plateforme,
    ));
    await add(RectangleComponent(
      size: Vector2(size.x, 5),
      paint: Paint()..color = Palette.plateformeHaut,
    ));
    await add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}
```

Trois points à noter.

**`size.clone()`.** Le `RectangleComponent` enfant reçoit une **copie** de la taille du parent. Avec `size: size`, les deux composants partageraient le même objet `Vector2` : redimensionner l'un redimensionnerait l'autre. C'est le piège du `Vector2` mutable signalé dans `_REF-FLAME.md`.

**Le constructeur nommé `Plateforme.tuiles`.** Il traduit des coordonnées en tuiles vers des pixels via `Constantes.tailleTuile`. Au chapitre 39, le chargeur de niveaux l'appellera pour chaque caractère `#` et `=` de la carte : le travail est déjà fait.

**`CollisionType.passive`.** Rappel du chapitre 32 : une hitbox `passive` n'est testée que contre des hitbox `active`. Deux plateformes ne se testent donc jamais entre elles. Avec des centaines de blocs par niveau, l'économie n'est pas anecdotique.

La palette reçoit au passage quatre couleurs : `plateforme`, `plateformeHaut`, `plateformeLegere`, `plateformeLegereHaut` (voir 36.CC).

---

## 36.4 — Construire une salle de test avec des plateformes

### 36.4.1 — Pourquoi une salle de test

Les vrais niveaux arrivent au chapitre 39, avec un chargeur de cartes. Mais on ne peut pas régler un saut sans avoir quelque chose à sauter. Il nous faut donc un terrain d'essai qui contienne **exactement les situations à tester** : un sol long (course et décélération), un mur à gauche et un à droite (blocage horizontal), un plafond bas (blocage vers le haut), une marche basse (petit saut), une plateforme haute (hauteur maximale), un rebord (coyote time), une plateforme `'='` (traversée par le bas) et un trou (mort par chute).

Si votre salle de test ne contient pas ces neuf éléments, vous découvrirez les bogues au chapitre 39, quand il sera beaucoup plus coûteux de les corriger.

### 36.4.2 — Le plan de la salle

Nous raisonnons en tuiles de `Constantes.tailleTuile` = 32 pixels. La salle fait 40 colonnes sur 18 lignes.

```text
   colonne  0         10        20        30        39
            |         |         |         |         |
  ligne  0  ########################################
         1  #......................................#
         2  #......................................#
         3  #...........####.......................#
         4  #......................................#
         5  #......................=====...........#
         6  #..................................###.#
         7  #....####..............................#
         8  #......................................#
         9  #.........................=====........#
        10  #..###.................................#
        11  #......................................#
        12  #......................................#
        13  #####........########..........#########
        14  #...#........#      #..........#.......#
        15  #...#        #      #          #.......#
        16  #...#        #      #          #.......#
        17  ####          TROU              ########

  Légende : '#' plateforme solide   '=' plateforme traversable
            '.' vide                 le TROU tue le joueur
```

### 36.4.3 — Le code de construction

On l'écrit dans `DonjonGame`. Une liste de listes d'entiers décrit chaque bloc : colonne, ligne, largeur, hauteur, traversable.

```dart
  Future<void> construireSalleDeTest() async {
    // (colonne, ligne, largeur, hauteur, traversable)
    const blocs = <List<int>>[
      [0, 0, 40, 1, 0], [0, 0, 1, 18, 0], [39, 0, 1, 18, 0],   // cadre
      [0, 13, 13, 1, 0], [30, 13, 10, 1, 0],                   // sol coupé
      [12, 14, 1, 4, 0], [19, 13, 1, 5, 0], [13, 13, 6, 1, 0], // trou
      [5, 7, 4, 1, 0], [2, 10, 3, 1, 0], [12, 3, 4, 1, 0],     // plateformes
      [35, 6, 3, 1, 0], [1, 13, 4, 5, 0],
      [23, 5, 5, 1, 1], [26, 9, 5, 1, 1],                      // traversables
    ];

    for (final b in blocs) {
      await monde.add(
        Plateforme.tuiles(b[0], b[1], b[2],
            nombreDeLignes: b[3], traversable: b[4] == 1),
      );
    }
  }
```

> **Remarque.** `List<int>` plutôt qu'une petite classe : c'est volontairement rustique, parce que ce code est **jetable**. Il disparaîtra au chapitre 39. Investir dans une belle abstraction pour du code temporaire est du temps perdu ; le savoir fait partie du métier.

### 36.4.4 — L'appel dans `demarrerPartie()`

Le chapitre 35 avait laissé `demarrerPartie()` presque vide. On la complète :

```dart
  Future<void> demarrerPartie() async {
    // IMPORTANT : on relance le moteur AVANT tout `await add(...)`.
    changerEtat(GameState.enJeu);

    // Repartir d'un monde propre.
    monde.removeAll(monde.children.toList());

    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = 0;

    await construireSalleDeTest();
    await ajouterJoueur();          // écrit en 36.5
  }
```

Deux lignes méritent un commentaire.

**`changerEtat(GameState.enJeu)` en premier.** Tant que le jeu est dans l'état `menu`, `changerEtat` a appelé `pauseEngine()` : la boucle est arrêtée. Or la file de montage des composants n'est traitée que dans `update()`. Un `await monde.add(...)` exécuté moteur en pause **ne se terminerait jamais**. On relance donc le moteur d'abord.

**`monde.removeAll(monde.children.toList())`.** On ne peut pas retirer des éléments d'une collection pendant qu'on la parcourt : `.toList()` fige un instantané avant la suppression. C'est la même précaution qu'au chapitre 6 avec les listes Dart.

---

## 36.5 — `lib/composants/joueur.dart` : le squelette

### 36.5.1 — Le squelette minimal

On commence par la coquille vide, celle qui compile et qui s'affiche. Les sections suivantes la remplissent.

```dart
// lib/composants/joueur.dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../core/entite.dart';

class Joueur extends Entite with KeyboardHandler, CollisionCallbacks {
  Joueur({required Vector2 position})
      : positionDepart = position.clone(),
        super(
          position: position.clone(),
          size: Vector2(Constantes.largeurJoueur, Constantes.hauteurJoueur),
          priority: 10,
        );

  /// Mémorisée pour la réapparition après une mort (36.31).
  final Vector2 positionDepart;

  double pv = Constantes.pvJoueurMax;
  bool invincible = false;
  int cles = 0;
}
```

| Élément de la signature | D'où il vient | Ce qu'il apporte |
| --- | --- | --- |
| `Entite` | 36.2 | `PositionComponent` + `HasGameReference<DonjonGame>` + vélocité + gravité |
| `KeyboardHandler` | chapitre 30 | `onKeyEvent(KeyEvent, Set<LogicalKeyboardKey>)` |
| `CollisionCallbacks` | chapitre 32 | `onCollisionStart`, `onCollision`, `onCollisionEnd` |

Rappel du chapitre 30 : `KeyboardHandler` sur un composant **n'a d'effet que si le jeu porte `HasKeyboardHandlerComponents`**. C'est le cas de `DonjonGame` depuis le chapitre 35. Et il ne faut surtout pas ajouter en plus `KeyboardEvents` sur le jeu : la documentation de Flame signale un conflit explicite entre les deux.

Le `priority: 10` est un z-index (chapitre 28). Convention du projet : plateformes 0, collectibles 5, ennemis 8, joueur 10, zone d'attaque 12.

### 36.5.2 — Le champ `positionDepart` et `clone()`

Il y a **deux** `clone()` dans le constructeur, et ce n'est pas une maladresse.

```text
  SANS clone() :
    positionDepart, component.position et la variable appelante
    designent LE MEME objet Vector2.
    Le joueur avance : position.x devient 500, donc positionDepart.x aussi.
    Au respawn, le joueur reapparait... la ou il est mort.

  AVEC clone() :
    positionDepart      -> copie 1, figee
    component.position  -> copie 2, qui bouge
```

C'est le bogue du `Vector2` mutable, listé parmi les pièges principaux de `_REF-FLAME.md`. Il ne produit aucune erreur de compilation, seulement un comportement absurde une heure plus tard.

### 36.5.3 — La création du joueur dans `DonjonGame`

```dart
// lib/donjon_game.dart — extrait
  late Joueur joueur;

  /// Position de départ dans la salle de test : colonne 3, ligne 12.
  static final Vector2 positionDepartJoueur = Vector2(
    Constantes.tailleTuile * 3,
    Constantes.tailleTuile * 12,
  );

  Future<void> ajouterJoueur() async {
    joueur = Joueur(position: positionDepartJoueur.clone());
    await monde.add(joueur);
  }
```

Encore un `clone()`. `positionDepartJoueur` est `static final` : sans copie, la première partie modifierait la constante du jeu et la deuxième partie démarrerait ailleurs.

---

## 36.6 — L'enum `EtatJoueur`

### 36.6.1 — La déclaration imposée

`_SPEC-JEU-2C.md` fixe la liste, dans cet ordre :

```dart
// lib/composants/joueur.dart — en tête de fichier
enum EtatJoueur { immobile, marche, saut, chute, attaque, touche, mort }
```

Sept états, pas un de plus. Chacun correspond à une **animation** dans un vrai jeu, et ici à une **couleur plus une déformation**.

| État | Quand | Animation typique | Notre rendu sans image |
| --- | --- | --- | --- |
| `immobile` | au sol, vitesse quasi nulle | *idle*, respiration | bleu clair, léger balancement vertical |
| `marche` | au sol, vitesse horizontale | *run* | bleu, corps qui oscille |
| `saut` | en l'air, `velocite.y < 0` | *jump* | bleu, corps étiré verticalement |
| `chute` | en l'air, `velocite.y >= 0` | *fall* | bleu foncé, corps étiré verticalement |
| `attaque` | pendant le coup | *attack* | doré, écharpe tendue |
| `touche` | juste après un dégât | *hurt* | rouge, clignotement |
| `mort` | pv à zéro | *death* | gris, aplati au sol |

### 36.6.2 — Pourquoi un enum et pas des booléens

On pourrait écrire :

```dart
// À NE PAS FAIRE
bool enSaut = false;
bool enAttaque = false;
bool estTouche = false;
bool estMort = false;
```

Avec quatre booléens indépendants, il existe 16 combinaisons, dont la plupart n'ont aucun sens : « mort et en attaque », « touché et immobile en même temps ». Vous passeriez votre temps à écrire des `if (!estMort && !estTouche && ...)`.

Un enum garantit qu'**un seul état est vrai à la fois**. C'est exactement l'argument du chapitre 11 (les enums) et du chapitre 26 (les machines à états). La question « dans quel état est le joueur ? » a toujours une réponse et une seule.

---

## 36.7 — Le corps du joueur sans image (`RectangleComponent`)

### 36.7.1 — Le principe : un groupe visuel

Le joueur ne se dessine pas lui-même. Il **contient** un sous-composant, `_visuel`, qui contient à son tour les rectangles.

```text
  Joueur                        (24 x 30, ancre topLeft)
   └── _visuel                  (PositionComponent, ancre CENTER, en (12, 15))
        ├── _corps              (RectangleComponent 24 x 30)
        ├── _oeil               (RectangleComponent 5 x 5, côté droit)
        └── _echarpe            (RectangleComponent 10 x 4)
```

Pourquoi cette couche intermédiaire ? Pour trois raisons qui reviennent tout au long du chapitre :

1. **Le retournement** (36.15) : on mettra `_visuel.scale.x = -1`. Comme l'ancre de `_visuel` est `Anchor.center`, le miroir se fait autour de son propre centre : le personnage se retourne **sur place**.
2. **La déformation** (36.27) : on étirera `_visuel.scale.y` pendant un saut sans toucher à `size`, donc sans toucher à la hitbox.
3. **Le clignotement** (36.30) : on change l'alpha des `Paint` des enfants sans affecter la logique.

> **À retenir.** Séparer le **corps physique** (position, taille, hitbox) du **visuel** (couleur, échelle, retournement) est une règle d'or. Un visuel qui déforme la hitbox produit des bogues invisibles et impossibles à reproduire.

### 36.7.2 — Le code

```dart
  late final PositionComponent _visuel;
  late final RectangleComponent _corps;
  late final RectangleComponent _oeil;
  late final RectangleComponent _echarpe;

  @override
  Future<void> onLoad() async {
    _visuel = PositionComponent(
      size: size.clone(),
      anchor: Anchor.center,
      position: Vector2(size.x / 2, size.y / 2),
    );

    _corps = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.joueur,
    );
    // L'œil est du côté droit : il indique la direction du regard.
    _oeil = RectangleComponent(
      position: Vector2(size.x - 8, 7),
      size: Vector2(5, 5),
      paint: Paint()..color = Palette.joueurOeil,
    );
    // Une écharpe qui dépasse derrière : elle donne vie au personnage.
    _echarpe = RectangleComponent(
      position: Vector2(-2, 10),
      size: Vector2(9, 4),
      paint: Paint()..color = Palette.accent,
    );

    await _visuel.addAll([_corps, _echarpe, _oeil]);
    await add(_visuel);
  }
```

L'ordre d'ajout compte : à `priority` égale, Flame dessine dans l'ordre d'insertion. Le corps passe derrière, l'œil devant. Inversez, et l'œil disparaît.

**Pourquoi 24 x 30 et non 32 x 32 ?** `Constantes.tailleTuile` vaut 32. Un joueur de 32 pixels de large remplirait exactement une tuile et se coincerait dans le moindre couloir d'une tuile, à cause des arrondis en virgule flottante. Règle empirique : donnez à votre personnage **environ 75 % de la largeur d'une tuile**. 24 sur 32, c'est exactement cela ; la hauteur de 30 le fait passer sous un plafond d'une tuile sans frotter.

---

## 36.8 — L'ancre et le point de référence (rappel chapitre 28)

### 36.8.1 — Rappel : ce qu'est une ancre

Le chapitre 28 l'a établi : `position` n'est pas « le coin haut-gauche du composant ». C'est **la position de l'ancre** du composant dans le repère du parent. L'ancre par défaut d'un `PositionComponent` est `Anchor.topLeft`.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │      LA MÊME position = Vector2(100, 100), TROIS ANCRES      │
  └──────────────────────────────────────────────────────────────┘

  Anchor.topLeft            Anchor.center           Anchor.bottomCenter

   (100,100)                                              
      ●────────┐            ┌────────┐               ┌────────┐
      │        │            │        │               │        │
      │        │            │   ●    │               │        │
      │        │            │(100,100)               │        │
      └────────┘            └────────┘               └───●────┘
                                                    (100,100)

  Le rectangle occupe :
   x de 100 a 124          x de 88 a 112            x de 88 a 112
   y de 100 a 130          y de 85 a 115            y de 70 a 100
```

### 36.8.2 — Le choix pour ce jeu : `topLeft`

`Entite` force `Anchor.topLeft`, pour trois raisons.

**Raison 1 : les collisions AABB deviennent triviales.** `Rect.fromLTWH(position.x, position.y, size.x, size.y)` suffit. Avec `Anchor.center`, il faudrait soustraire une demi-taille **partout** — une occasion d'oubli à chaque fois, et l'oubli produit un décalage qu'on met une soirée à diagnostiquer.

**Raison 2 : la grille de tuiles est en `topLeft`.** Une tuile en colonne 5, ligne 3 a son coin en `(5 x 32, 3 x 32)`. Le format de niveaux du chapitre 39 raisonne ainsi.

**Raison 3 : atterrir devient une soustraction simple**, `position.y = plateforme.boite.top - size.y`.

Le choix `topLeft` s'applique au **corps physique**. Pour le visuel, c'est l'inverse : `_visuel` a l'ancre `Anchor.center`, parce que le retournement et la déformation doivent se faire autour du **centre** du personnage. Avec une ancre `topLeft`, `scale.x = -1` déplacerait le dessin d'une largeur entière vers la gauche.

> **À retenir.** Ancre `topLeft` pour ce qui est calculé (position, hitbox, collisions). Ancre `center` pour ce qui est transformé (retournement, rotation, mise à l'échelle).

---

## 36.9 — La vélocité et l'intégration (rappel chapitre 23)

### 36.9.1 — Rappel du chapitre 23

Le mouvement se décrit par trois grandeurs : la **position** (en pixels), la **vitesse** (en pixels par seconde) et l'**accélération** (en pixels par seconde au carré). L'intégration d'Euler, employée dans tout ce cours, s'écrit à chaque frame, avec `dt` en **secondes** :

```text
    vitesse  = vitesse  + acceleration * dt
    position = position + vitesse      * dt
```

Cet ordre-là est important : ajouter la gravité **avant** de déplacer donne un mouvement plus stable qu'ajouter après (c'est l'Euler « semi-implicite »). Ne l'inversez pas.

### 36.9.2 — Pourquoi `dt` n'est pas négociable

`dt` est le temps écoulé depuis la frame précédente, **en secondes** (chapitre 20). Sur un écran 60 Hz il vaut environ `0,0167` ; sur un écran 120 Hz, environ `0,0083`.

```text
  Deplacement horizontal de 180 px/s pendant 1 seconde

  AVEC dt :
    60 fps  : 60 frames  x 180 x 0,0167 = 180,0 px   correct
    120 fps : 120 frames x 180 x 0,0083 = 180,0 px   correct

  SANS dt (position.x += 3) :
    60 fps  : 60 frames  x 3 = 180 px
    120 fps : 120 frames x 3 = 360 px   le jeu est DEUX FOIS trop rapide
```

Un jeu sans `dt` n'est pas « un peu approximatif » : il est injouable sur un autre écran que celui du développeur.

### 36.9.3 — La structure de `update()`

Voici le squelette définitif de la méthode. Toutes les sections suivantes remplissent une de ces lignes.

```dart
  @override
  void update(double dt) {
    super.update(dt);

    if (etat == EtatJoueur.mort) {
      // Mort : plus d'entrées, mais la gravité continue (36.31).
      appliquerGravite(dt);
      deplacerAvecCollisions(dt);
      return;
    }

    _mettreAJourMinuteurs(dt);   // 36.21, 36.22, 36.29, 36.30
    _mettreAJourHorizontal(dt);  // 36.13, 36.14
    _tenterLeSaut();             // 36.19 a 36.23
    appliquerGravite(dt, multiplicateur: _multiplicateurGravite);  // 36.10
    deplacerAvecCollisions(dt);  // 36.17, 36.18
    _mettreAJourEtat();          // 36.25, 36.26
    _mettreAJourApparence(dt);   // 36.27
    _verifierChuteMortelle();    // 36.31
  }
```

Cet ordre n'est pas arbitraire. Chaque étape produit ce dont la suivante a besoin :

```text
  minuteurs   -> savoir si le coyote time est encore actif
  horizontal  -> velocite.x a jour
  saut        -> velocite.y mise a -520 si le saut part
  gravite     -> velocite.y augmentee
  deplacement -> position mise a jour, auSol recalcule
  etat        -> choisi d'apres velocite ET auSol : il faut qu'ils soient a jour
  apparence   -> depend de l'etat
```

Si vous placiez `_mettreAJourEtat()` avant le déplacement, l'état serait toujours en retard d'une frame : le personnage afficherait « chute » pendant qu'il est déjà au sol.

---

## 36.10 — Appliquer la gravité

### 36.10.1 — La constante

```dart
static const double gravite = 1200.0;   // pixels / s²
```

Cette valeur vient de `_SPEC-JEU-2C.md`. Que signifie-t-elle concrètement ?

```text
  Une entite lachee sans vitesse initiale, avec g = 1200 px/s² :

  apres 0,25 s : vitesse = 300 px/s   chute de   37,5 px  (~1 tuile)
  apres 0,50 s : vitesse = 600 px/s   chute de  150,0 px  (~5 tuiles)
  apres 0,75 s : vitesse = 900 px/s   chute de  337,5 px  (~10 tuiles)
  apres 1,00 s : vitesse = 1200 px/s  chute de  600,0 px  (~19 tuiles)
```

À titre de comparaison, la gravité terrestre vaut environ 9,81 m/s². Si l'on décide qu'une tuile de 32 pixels représente 50 cm, alors 1200 px/s² correspond à 18,75 m/s², soit près du double du réel. **C'est voulu.** Presque tous les jeux de plateforme utilisent une gravité surhumaine : une chute réaliste paraît molle et lente à l'écran.

### 36.10.2 — L'axe Y pointe vers le BAS

Piège classique, déjà signalé au chapitre 21 : dans un canvas 2D, **Y augmente vers le bas**.

```text
   (0,0) ●─────────────────────> X croissant
         │      velocite.y = -520  MONTE  (saut)
         │            ▲
         │         ┌─────┐
         │         │  ●  │
         │         └─────┘
         │            ▼
         │      velocite.y = +900  DESCEND (chute)
         V   Y croissant
```

Conséquences à mémoriser : la gravité **ajoute** à `velocite.y` ; `Constantes.forceSaut` vaut `-520.0`, il est **négatif** ; « le joueur monte » se teste avec `velocite.y < 0`, « il descend » avec `velocite.y > 0`.

La gravité s'applique **toujours**, même au sol. Si vous forcez `velocite.y = 0` tant que `auSol`, le joueur qui quitte un rebord reste en lévitation. La résolution de collision (36.18) se charge d'annuler la vélocité au moment de l'impact, et `auSol` est recalculé à chaque frame.

---

## 36.11 — Limiter la vitesse de chute

### 36.11.1 — Le problème du tunneling

Sans plafond, la vélocité de chute croît indéfiniment. Or le déplacement d'une frame vaut `velocite.y * dt`. À 60 fps, une vélocité de 6000 px/s produit un bond de 100 pixels en une seule frame — alors qu'une plateforme n'en fait que 32.

```text
  frame N        frame N+1 (deplacement de 100 px)

     ██
     ██
  ████████            ████████    <- plateforme de 32 px

                          ██      <- le joueur est PASSE AU TRAVERS
                          ██         sans jamais avoir touche
```

C'est le **tunneling**, décrit au chapitre 24 : la collision n'est jamais détectée parce que les deux rectangles ne se chevauchent à aucune frame.

### 36.11.2 — La solution retenue

```dart
static const double vitesseMaxChute = 900.0;
```

À 900 px/s et 60 fps, le déplacement maximum par frame vaut `900 * 0.0167 = 15 px`. C'est bien inférieur aux 32 pixels d'une plateforme : le tunneling est impossible.

La règle générale :

```text
  vitesseMax * dt  <  epaisseur du plus fin obstacle

  Avec dt = 1/60 et une epaisseur de 32 px :
     vitesseMax < 32 x 60 = 1920 px/s

  On prend 900 : une marge de securite de plus du double.
```

---

## 36.12 — Lire le clavier (rappel chapitre 30)

### 36.12.1 — Le mixin `KeyboardHandler`

Le chapitre 30 a présenté deux niveaux de lecture du clavier : `KeyboardEvents` sur la classe `Game`, ou `KeyboardHandler` sur un composant (avec `HasKeyboardHandlerComponents` sur le jeu). Nous prenons le niveau **composant** : c'est le joueur qui sait quoi faire d'une touche, pas le jeu. Et au chapitre 40, `pauseEngine()` suffira à couper les entrées, puisqu'il gèle tout l'arbre.

La signature exacte, à recopier sans la modifier :

```dart
  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    return true;   // true : l'evenement continue de se propager
  }
```

`KeyEvent` et `LogicalKeyboardKey` viennent de `package:flutter/services.dart`.

> **Piège.** Beaucoup de tutoriels antérieurs à 2023 montrent `onKeyEvent(RawKeyEvent event, ...)`. `RawKeyEvent` appartient à l'ancienne API de Flutter ; Flame 1.38.0 utilise `KeyEvent`. Si votre éditeur signale une erreur de signature, c'est presque toujours cela.

### 36.12.2 — Deux informations très différentes

La méthode reçoit deux paramètres qui ne disent pas la même chose. Cette distinction est **la** difficulté de la section.

```text
  keysPressed : Set<LogicalKeyboardKey>
      = l'ETAT actuel du clavier, la liste des touches ENFONCEES.
      -> « la touche D est-elle maintenue ? »
      -> sert au deplacement continu (gauche / droite)

  event : KeyEvent
      = ce qui vient de CHANGER a cet instant.
        KeyDownEvent (appui) / KeyUpEvent (relachement) /
        KeyRepeatEvent (repetition automatique du systeme)
      -> « vient-on d'appuyer sur Espace ? »
      -> sert aux actions ponctuelles (saut, attaque)
```

Utiliser `keysPressed.contains(LogicalKeyboardKey.space)` pour sauter produirait un saut **à chaque frame** tant que la touche reste enfoncée. Utiliser `event is KeyDownEvent` pour se déplacer produirait un déplacement d'un pixel puis plus rien.

### 36.12.3 — `KeyRepeatEvent` : la protection gratuite

Quand vous maintenez une touche, le système d'exploitation émet, après un délai, une série de `KeyRepeatEvent`. Comme `KeyRepeatEvent` est une classe **distincte** de `KeyDownEvent`, le test `event is KeyDownEvent` est faux pour les répétitions.

Autrement dit : `event is KeyDownEvent` ne se déclenche **qu'une seule fois** par appui réel. C'est exactement ce qu'il nous faut pour le saut, et c'est gratuit.

### 36.12.4 — Le code de la lecture

```dart
  // Entrées mémorisées, lues ensuite par update().
  int _entreeHorizontale = 0;   // -1 gauche, 0 rien, +1 droite
  bool _toucheBas = false;
  bool _sautMaintenu = false;

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (etat == EtatJoueur.mort) return true;

    // --- 1. Etat continu : les directions ---
    final gauche = keysPressed.contains(LogicalKeyboardKey.arrowLeft) ||
        keysPressed.contains(LogicalKeyboardKey.keyQ) ||   // AZERTY
        keysPressed.contains(LogicalKeyboardKey.keyA);     // QWERTY
    final droite = keysPressed.contains(LogicalKeyboardKey.arrowRight) ||
        keysPressed.contains(LogicalKeyboardKey.keyD);

    _entreeHorizontale = (droite ? 1 : 0) - (gauche ? 1 : 0);

    _toucheBas = keysPressed.contains(LogicalKeyboardKey.arrowDown) ||
        keysPressed.contains(LogicalKeyboardKey.keyS);
    _sautMaintenu = keysPressed.contains(LogicalKeyboardKey.space) ||
        keysPressed.contains(LogicalKeyboardKey.arrowUp) ||
        keysPressed.contains(LogicalKeyboardKey.keyW) ||
        keysPressed.contains(LogicalKeyboardKey.keyZ);

    // --- 2. Fronts : les actions ponctuelles ---
    if (event is KeyDownEvent) {
      final t = event.logicalKey;
      if (_estToucheDeSaut(t)) sauter();
      if (t == LogicalKeyboardKey.keyE || t == LogicalKeyboardKey.keyX) {
        attaquer();
      }
      if (t == LogicalKeyboardKey.arrowDown || t == LogicalKeyboardKey.keyS) {
        descendre();
      }
    }

    if (event is KeyUpEvent && _estToucheDeSaut(event.logicalKey)) {
      _couperLeSaut();               // saut variable, 36.20
    }

    // true : l'evenement continue de se propager aux autres composants.
    // Au chapitre 40, un composant de raccourcis (touche P) doit le recevoir.
    return true;
  }
```

La ligne `(droite ? 1 : 0) - (gauche ? 1 : 0)` remplace quatre `if`. Sa table de vérité : rien enfoncé `0`, droite seule `1`, gauche seule `-1`, **les deux enfoncées `0`**. Ce dernier cas est important : sans lui, le comportement dépendrait de l'ordre des tests ; ici le joueur s'arrête, ce qu'attendent tous les joueurs.

`LogicalKeyboardKey` désigne la touche **logique**. On accepte donc `keyQ` (AZERTY) et `keyA` (QWERTY), `keyZ` et `keyW` : le jeu se joue avec ZQSD, WASD ou les flèches, sans configuration.

---

## 36.13 — Déplacement gauche / droite

### 36.13.1 — La version la plus simple

```dart
  // Version 1 : vitesse constante, sans inertie.
  void _mettreAJourHorizontal(double dt) {
    velocite.x = Constantes.vitesseJoueur * _entreeHorizontale;
  }
```

Trois lignes, et le personnage se déplace déjà. `Constantes.vitesseJoueur` vaut `180.0` : le joueur parcourt 180 pixels par seconde, soit un peu moins de six tuiles.

Testez-la. Elle marche. Et pourtant, aucun jeu commercial n'utilise cette version : le passage de 0 à 180 px/s y est **instantané**, ce qui exigerait une accélération infinie. Le cerveau du joueur le sait, même sans savoir pourquoi, et le décrit toujours avec les mêmes mots : « le personnage glisse », « c'est raide », « ça ne répond pas ». La correction est l'objet de la section 36.14.

### 36.13.2 — Mise à jour de la direction

Une ligne à ne pas oublier, en tête du calcul horizontal :

```dart
    if (_entreeHorizontale != 0) {
      direction = _entreeHorizontale;
    }
```

Le `if` compte. Sans lui, `direction` retomberait à `0` dès que le joueur lâche les touches, et le personnage regarderait « nulle part ». En mémorisant, il continue de regarder du côté où il allait.

---

## 36.14 — L'accélération et la décélération : le game feel

### 36.14.1 — Le principe

Au lieu d'imposer la vitesse, on impose une **vitesse cible** et on s'en approche progressivement.

```text
  Si une touche de direction est enfoncee :
      cible = direction x vitesseJoueur
      on approche velocite.x de cible a raison de acceleration px/s²

  Si aucune touche n'est enfoncee :
      cible = 0
      on approche velocite.x de 0 a raison de deceleration px/s²
```

Et il faut deux décélérations distinctes : au sol, la friction est forte (on s'arrête vite) ; en l'air, elle est faible (on garde son élan).

### 36.14.2 — Les constantes

```dart
  // lib/config/constantes.dart — ajouts du chapitre 36
  static const double accelerationSol = 1500.0;    // px / s²
  static const double accelerationAir = 900.0;     // px / s²
  static const double decelerationSol = 2200.0;    // px / s²
  static const double decelerationAir = 700.0;     // px / s²
```

Ce que ces valeurs signifient en temps :

```text
  Temps pour atteindre 180 px/s depuis l'arret :
    au sol : 180 / 1500 = 0,120 s   (~7 frames a 60 fps)
    en l'air : 180 / 900 = 0,200 s  (~12 frames)

  Temps pour s'arreter depuis 180 px/s :
    au sol : 180 / 2200 = 0,082 s   (~5 frames)
    en l'air : 180 / 700 = 0,257 s  (~15 frames)
```

Une règle de game feel presque universelle apparaît : **on décélère plus vite qu'on n'accélère**, et **on contrôle moins bien en l'air qu'au sol**. Le premier point rend l'arrêt net et précis ; le second récompense l'anticipation.

### 36.14.3 — Le code

```dart
  void _mettreAJourHorizontal(double dt) {
    if (_entreeHorizontale != 0) {
      direction = _entreeHorizontale;
    }

    final cible = Constantes.vitesseJoueur * _entreeHorizontale;

    final double taux;
    if (_entreeHorizontale == 0) {
      taux = auSol ? Constantes.decelerationSol : Constantes.decelerationAir;
    } else {
      taux = auSol ? Constantes.accelerationSol : Constantes.accelerationAir;
    }

    velocite.x = _approcher(velocite.x, cible, taux * dt);
  }

  /// Rapproche [valeur] de [cible] d'au plus [pas], sans jamais la dépasser.
  static double _approcher(double valeur, double cible, double pas) {
    if (valeur < cible) {
      final v = valeur + pas;
      return v > cible ? cible : v;
    }
    if (valeur > cible) {
      final v = valeur - pas;
      return v < cible ? cible : v;
    }
    return cible;
  }
```

---

## 36.15 — Retourner le personnage selon la direction

### 36.15.1 — Le principe du miroir

On ne fait pas tourner le personnage : on le **reflète** horizontalement, en donnant à l'échelle en X la valeur `-1`.

```dart
  void _mettreAJourOrientation() {
    _visuel.scale.x = direction.toDouble();
  }
```

`direction` vaut `1` ou `-1`. `scale.x = 1` laisse le visuel tel quel ; `scale.x = -1` le retourne.

### 36.15.2 — Pourquoi cela fonctionne ici et pas ailleurs

Toute la subtilité tient à l'ancre de `_visuel`, fixée à `Anchor.center` en 36.7. La mise à l'échelle d'un `PositionComponent` se fait **autour de son ancre**.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │     scale.x = -1 SELON L'ANCRE DU COMPOSANT MIS A L'ECHELLE    │
  └────────────────────────────────────────────────────────────────┘

  Ancre topLeft                        Ancre center
  (miroir autour du bord gauche)       (miroir autour du milieu)

    avant       apres                   avant       apres
   ┌──────┐   ┌──────┐                 ┌──────┐   ┌──────┐
   │██  ●│    │●  ██│                  │██  ●│    │●  ██│
   └──────┘   └──────┘                 └──────┘   └──────┘
   0    24   -24    0                  0    24    0    24
              ↑                                    ↑
        DECALE de 24 px                    RESTE EN PLACE
```

C'est la raison d'être de la couche `_visuel`. Si vous appliquiez `scale.x = -1` directement au `Joueur` (ancre `topLeft`, imposée par `Entite`), le personnage se téléporterait d'une largeur vers la gauche à chaque changement de direction, et sa hitbox ne suivrait pas.

---

## 36.16 — La hitbox du joueur (rappel chapitre 32)

### 36.16.1 — Deux systèmes de collision cohabitent

C'est le point le plus important de la section, et une source de confusion permanente.

```text
  1. RESOLUTION MANUELLE (ecrite par nous, section 36.18)
     contre les Plateforme -> EMPECHER de traverser, poser au sol
     comparaison de Rect, axe par axe, dans update()
     a la main parce qu'il faut CORRIGER la position.

  2. CollisionCallbacks DE FLAME (chapitres 32 / 37 / 38)
     contre les ennemis, collectibles, portes -> DECLENCHER un evenement
     onCollisionStart(intersectionPoints, other)
     fourni par Flame parce qu'il n'y a rien a corriger, juste a reagir.
```

Flame ne fournit **pas** de moteur physique qui repousse les corps. Ni `RectangleHitbox` ni `CollisionCallbacks` n'empêchent quoi que ce soit de se superposer : ils vous **préviennent**. Pour un vrai solveur, il faudrait `flame_forge2d` (aperçu au chapitre 34), un marteau bien trop lourd pour un jeu de plateforme à la main.

### 36.16.2 — La hitbox du joueur

```dart
    await add(
      RectangleHitbox(
        size: Vector2(size.x - 4, size.y - 2),
        position: Vector2(2, 2),
        collisionType: CollisionType.active,
      ),
    );
```

**`CollisionType.active`.** Rappel du chapitre 32 : une hitbox `active` est testée contre les `active` **et** les `passive`. Les plateformes étant `passive`, le couple fonctionne.

**Une hitbox légèrement plus petite que le corps.** C'est une pratique universelle : la hitbox de dégâts est **plus indulgente** que le sprite. Le joueur qui frôle un gobelin a l'impression de l'avoir esquivé de justesse, ce qui est bien plus satisfaisant qu'un dégât « injuste ».

**Elle n'est pas utilisée pour les plateformes.** La résolution de 36.18 se base sur `boite`, c'est-à-dire sur la taille **complète** du joueur. Utiliser deux géométries différentes est volontaire : le décor doit être précis, sinon le personnage flotte ; les dégâts doivent être indulgents.

Pendant la mise au point, `bool get debugMode => true;` sur `DonjonGame` dessine toutes les hitbox de l'arbre. Pensez à le repasser à `false` avant de livrer.

### 36.16.3 — `onCollisionStart` : la coquille à remplir

Au chapitre 37, cette méthode gérera les ennemis. Aujourd'hui, on pose la structure :

```dart
  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);

    // Les plateformes sont traitées par la résolution manuelle (36.18) :
    // on les ignore explicitement ici.
    if (other is Plateforme) return;

    // Chapitre 37 : if (other is Ennemi) subirDegats(other.degatsContact);
    // Chapitre 38 : if (other is Collectible) other.ramasser(this);
  }
```

L'appel à `super` est **obligatoire** : la méthode est annotée `@mustCallSuper` dans le code source de Flame. Sans lui, l'ensemble `activeCollisions` cesse d'être tenu à jour et `isColliding` renvoie n'importe quoi.

---

## 36.17 — Détecter le sol

### 36.17.1 — Les trois méthodes possibles

| Méthode | Principe | Verdict |
| --- | --- | --- |
| Mémoriser un booléen à l'atterrissage | `auSol = true` au contact, `false` au saut | **Mauvais.** On oublie toujours un cas : quitter un rebord, une plateforme retirée, une téléportation. |
| Sonder juste sous les pieds | tester un rectangle de 1 ou 2 px sous le joueur | Correct, mais c'est un test de plus par frame. |
| Déduire de la résolution verticale | `auSol = true` quand on a bloqué une descente | **Retenu.** Zéro test supplémentaire, toujours exact. |

### 36.17.2 — La règle retenue

```text
  A CHAQUE FRAME :

   1. auSol = false                       (on part du principe qu'on tombe)
   2. on deplace en Y
   3. si un blocage a lieu ALORS QUE velocite.y > 0 :
         -> on descendait et on a touche quelque chose
         -> c'est le sol
         -> auSol = true
```

Le point 1 est le plus important. `auSol` n'est **jamais** conservé d'une frame à l'autre : il est reconstruit. Un joueur qui quitte un rebord voit son `auSol` retomber à `false` dès la frame suivante, sans une ligne de code spécifique.

### 36.17.3 — Détecter l'instant de l'atterrissage

Pour la poussière, le son et l'écrasement visuel, il faut savoir qu'on **vient** de toucher le sol. `Entite` compare donc `auSol` avant et après le déplacement, et appelle une méthode que les sous-classes redéfinissent :

```dart
  // Dans Entite
  void deplacerAvecCollisions(double dt) {
    final basAvant = bas;
    final auSolAvant = auSol;

    _deplacerEnX(velocite.x * dt);
    _deplacerEnY(velocite.y * dt, basAvant);

    if (auSol && !auSolAvant) {
      surAtterrissage();
    }
  }

  /// Appelée à la frame exacte où l'entité vient de toucher le sol.
  void surAtterrissage() {}
```

```dart
  // Dans Joueur
  @override
  void surAtterrissage() {
    _ecrasement = 1.0;          // déformation visuelle, 36.27
    _doubleSautUtilise = false; // le crédit de double saut est rendu
  }
```

---

## 36.18 — Résoudre les collisions avec les plateformes, axe par axe

### 36.18.1 — Pourquoi axe par axe

C'est le cœur technique du chapitre. Résoudre les deux axes **en même temps** est délicat : quand deux rectangles se chevauchent, il faut deviner par où l'objet est entré. Faut-il repousser le joueur vers le haut, parce qu'il a atterri, ou vers la gauche, parce qu'il a heurté un mur ? Les deux sont géométriquement possibles, et se tromper produit un personnage qui **escalade les murs**.

Résoudre **un axe puis l'autre** supprime l'ambiguïté.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │             RESOLUTION AXE PAR AXE : sans ambiguite               │
  └──────────────────────────────────────────────────────────────────┘

  ETAPE 1 : deplacer en X SEULEMENT
           ┌───┐            ┌───┐
           │ J │  ───────>  │ J │███
           └───┘            └───┘███
    Si intersection : on venait forcement de la GAUCHE ou de la DROITE,
    puisque Y n'a pas bouge. Repousser horizontalement. velocite.x = 0.

  ETAPE 2 : deplacer en Y SEULEMENT
                            ┌───┐
                            │ J │
                            └───┘
                              V
                           ████████
    Si intersection : on venait forcement du HAUT ou du BAS.
    Repousser verticalement. velocite.y = 0. auSol si on descendait.
```

Chaque étape ne peut se tromper, car un seul axe a changé. C'est la méthode employée dans l'immense majorité des jeux de plateforme 2D à hitbox rectangulaire.

### 36.18.2 — Le squelette

```dart
  // Dans Entite (lib/core/entite.dart)
  void deplacerAvecCollisions(double dt) {
    final basAvant = bas;             // mémorisé pour les plateformes '='

    _deplacerEnX(velocite.x * dt);
    _deplacerEnY(velocite.y * dt, basAvant);
  }
```

`basAvant` est le bord inférieur **avant** tout déplacement. Il servira en 36.24 pour décider si le joueur arrive par-dessus une plateforme traversable.

### 36.18.3 — L'axe horizontal

```dart
  void _deplacerEnX(double dx) {
    if (dx == 0) return;

    position.x += dx;

    for (final p in plateformes) {
      // Une plateforme traversable ne bloque JAMAIS horizontalement.
      if (p.traversable) continue;
      if (!boite.overlaps(p.boite)) continue;

      if (dx > 0) {
        // On allait a droite : on se colle au bord GAUCHE de l'obstacle.
        position.x = p.boite.left - size.x;
      } else {
        // On allait a gauche : on se colle au bord DROIT de l'obstacle.
        position.x = p.boite.right;
      }
      velocite.x = 0;
    }
  }
```

Notez que `boite` est un **getter** : il est recalculé à chaque appel, donc après chaque correction de `position.x`. C'est indispensable, car une correction peut créer une nouvelle intersection avec une autre plateforme.

### 36.18.4 — L'axe vertical

```dart
  void _deplacerEnY(double dy, double basAvant) {
    position.y += dy;
    auSol = false;

    for (final p in plateformes) {
      if (!boite.overlaps(p.boite)) continue;

      if (dy > 0) {
        // On DESCEND : on se pose sur le dessus de l'obstacle.
        if (p.traversable) {
          // On ne se pose que si l'on venait STRICTEMENT d'au-dessus.
          if (basAvant > p.boite.top + 1.0) continue;
          if (traverseLesPlateformes) continue;
        }
        position.y = p.boite.top - size.y;
        velocite.y = 0;
        auSol = true;
      } else if (dy < 0) {
        // On MONTE : une plateforme traversable se franchit.
        if (p.traversable) continue;
        position.y = p.boite.bottom;
        velocite.y = 0;
      }
    }
  }
```

### 36.18.5 — Pourquoi `overlaps` et pas `intersects`

`Rect.overlaps(Rect other)` renvoie `true` uniquement si les deux rectangles se chevauchent **strictement**. Deux rectangles qui se touchent exactement, bord contre bord, ne se chevauchent pas.

```text
  A : x de 0 a 24        B : x de 24 a 56

  A.overlaps(B)  ->  false     (ils se TOUCHENT, ils ne se chevauchent pas)
```

C'est précisément ce qu'il faut. Après `position.y = p.boite.top - size.y`, le joueur est exactement collé : `bas == p.boite.top`. Si `overlaps` renvoyait `true` dans ce cas, la résolution se redéclencherait indéfiniment et le personnage resterait collé au plafond ou au sol de façon aléatoire.

---

## 36.19 — Le saut

### 36.19.1 — Le principe

Sauter, c'est imposer d'un coup une vélocité verticale négative. La gravité fait le reste.

```dart
static const double forceSaut = -520.0;   // pixels / s
```

### 36.19.2 — La hauteur atteinte

La physique du chapitre 23 donne la hauteur maximale et le temps de montée :

```text
  hauteur = v² / (2 g) = 520 x 520 / (2 x 1200) = 112,7 px = 3,5 tuiles
  temps   = v / g      = 520 / 1200             = 0,433 s
```

Retenez ces deux nombres : **3,5 tuiles de haut, 0,43 s de montée**. Ce sont eux qui déterminent la conception de vos niveaux. Une plateforme placée à 4 tuiles sera inatteignable ; à 3 tuiles, elle sera confortable.

La portée horizontale, elle, vaut `180 x 0,87 = 156 px`, soit près de 5 tuiles : un trou de 4 tuiles se franchit en courant, un trou de 6 tuiles ne se franchit pas.

```text
  4 tuiles ┤ ······················  hors d'atteinte
           │        ╭────╮
  3 tuiles ┤ ─────╭─╯    ╰─╮──────   confortable
           │     ╱          ╲
  1 tuile  ┤ ──╱──────────────╲───   facile
  sol   ───●──────────────────────●───
           |<--- 0,87 s au total --->|
```

### 36.19.3 — La méthode `sauter()`

`_SPEC-JEU-2C.md` impose une méthode publique `void sauter()`. Attention : elle **ne fait pas sauter directement**. Elle enregistre une **intention** de saut, que `_tenterLeSaut()` transformera (ou non) en saut réel. C'est ce qui rendra possible le jump buffering en 36.22.

```dart
  /// Demande un saut. L'exécution effective a lieu dans update().
  void sauter() {
    _bufferSaut = Constantes.bufferSaut;
  }

  void _tenterLeSaut() {
    if (_bufferSaut <= 0) return;

    if (auSol || _coyote > 0) {                  // saut au sol (ou coyote)
      _executerSaut(Constantes.forceSaut);
      _bufferSaut = 0;
      _coyote = 0;
      return;
    }

    if (Constantes.doubleSautActif && !_doubleSautUtilise) {   // 36.23
      _doubleSautUtilise = true;
      _executerSaut(Constantes.forceDoubleSaut);
      _bufferSaut = 0;
    }
  }

  void _executerSaut(double force) {
    velocite.y = force;      // ASSIGNATION, pas +=
    auSol = false;
    _etirement = 1.0;        // déformation visuelle, 36.27
  }
```

`velocite.y = force` et non `+=` : si le joueur sautait en tombant déjà à 400 px/s, l'addition donnerait `400 - 520 = -120`, un saut ridicule. L'assignation garantit que **tous les sauts font exactement la même hauteur**. La régularité est ce que le joueur attend.

---

## 36.20 — Le saut variable selon la durée d'appui

### 36.20.1 — Le problème

Avec le code précédent, tous les sauts font 3,5 tuiles. Impossible de faire un petit saut pour monter une marche. Le joueur n'a aucun contrôle après le décollage.

Tous les bons jeux du genre offrent un **saut variable** : une pression brève donne un petit saut, une pression longue le saut maximal.

### 36.20.2 — La technique : couper la montée au relâchement

```dart
static const double coupureSaut = 0.42;   // multiplicateur, entre 0 et 1
```

```dart
  /// Appelée quand la touche de saut est relâchée (36.12).
  void _couperLeSaut() {
    // On ne coupe que si l'on MONTE encore.
    if (velocite.y < 0) {
      velocite.y *= Constantes.coupureSaut;
    }
  }
```

Le test `velocite.y < 0` est indispensable : relâcher la touche pendant la descente ne doit rien faire.

```text
  ┌────────────────────────────────────────────────────────────┐
  │            SAUT COURT  vs  SAUT LONG                        │
  └────────────────────────────────────────────────────────────┘

  APPUI LONG (touche gardee)         APPUI COURT (0,1 s)

  v.y = -520                          v.y = -520
       ↓ gravite                           ↓ gravite pendant 0,1 s
  v.y = -400                          v.y = -400
       ↓                                   ↓ RELACHEMENT
  v.y = -200                          v.y = -400 x 0,42 = -168
       ↓                                   ↓
  v.y = 0  ← sommet a 112 px          v.y = 0  ← sommet a ~35 px
```

### 36.20.3 — Le second réglage : une gravité asymétrique

Une deuxième technique, complémentaire et tout aussi standard : **tomber plus vite qu'on ne monte**.

```dart
static const double multiplicateurChute = 1.55;
```

Physiquement, c'est faux : un projectile réel monte et descend symétriquement. Mais à l'écran, la montée symétrique donne une impression de flottement lunaire. En accélérant la descente d'environ 50 %, le saut devient nerveux tout en gardant sa hauteur.

```text
  hauteur          symetrique                asymetrique (retenu)
    112 ┤            ╭──╮                          ╭─╮
        │           ╱    ╲                        ╱   ╲
      0 ┼──────────╯      ╰──────────────────────╯     ╰────
                   |<0,43>|<0,43>|              |<0,43>|<0,28>|

  Meme hauteur, retour au sol 35 % plus rapide.
```

### 36.20.4 — Alléger la montée tant que la touche est tenue

Troisième raffinement, gratuit puisque `_sautMaintenu` est déjà mémorisé (36.12.4) : on réduit un peu la gravité pendant la montée si la touche reste enfoncée.

```dart
  static const double multiplicateurMontee = 0.88;

  double get _multiplicateurGravite {
    if (velocite.y < 0) {
      return _sautMaintenu ? Constantes.multiplicateurMontee : 1.0;
    }
    return Constantes.multiplicateurChute;
  }
```

L'écart entre un appui bref et un appui long s'en trouve encore accentué, sans changer la hauteur maximale de plus de 12 %.

---

## 36.21 — Le coyote time

### 36.21.1 — Le problème

Situation vécue par tout joueur : on court vers le bord d'une plateforme, on appuie sur saut... et le personnage tombe sans sauter.

```text
  frame 40 : auSol = true    ██████●     <- encore sur le bord
  frame 41 : auSol = false   ██████ ●    <- deja dans le vide
  frame 42 : APPUI SAUT      ██████  ●   <- refuse : auSol est faux
```

Il s'est écoulé deux frames, soit 33 millisecondes. Le joueur est persuadé d'avoir appuyé à temps. Il accuse le jeu, et il a raison.

### 36.21.2 — La solution

Le nom vient des dessins animés : le personnage qui court au-delà d'une falaise continue de courir dans le vide quelques instants avant de tomber. On accorde au joueur une petite fenêtre pendant laquelle il **peut encore sauter** bien qu'il ne soit plus au sol.

```dart
static const double coyoteTime = 0.10;   // secondes
```

```dart
  double _coyote = 0;

  void _mettreAJourMinuteurs(double dt) {
    if (auSol) {
      _coyote = Constantes.coyoteTime;   // rechargé en permanence
    } else {
      _coyote -= dt;                     // décompté dans le vide
    }
    // ... autres minuteurs ...
  }
```

Et la condition de saut, déjà écrite en 36.19.3 :

```dart
    final peutSauterDepuisLeSol = auSol || _coyote > 0;
```

### 36.21.3 — Ne pas oublier de le consommer

```dart
      _executerSaut(Constantes.forceSaut);
      _bufferSaut = 0;
      _coyote = 0;      // ← indispensable
```

Sans `_coyote = 0`, le joueur pourrait sauter, puis re-sauter dans les 0,1 s qui suivent, obtenant un double saut involontaire. Un crédit consommé doit être remis à zéro.

---

## 36.22 — Le jump buffering

### 36.22.1 — Le problème symétrique

Le coyote time corrige l'appui **trop tard**. Le jump buffering corrige l'appui **trop tôt**.

```text
  frame 60 : APPUI SAUT      ●        auSol = false -> refuse
  frame 61 :                  ●       auSol = false
  frame 62 : ATTERRISSAGE   ████●     auSol = true  -> plus d'appui
```

Le joueur qui enchaîne des sauts appuie systématiquement quelques frames **avant** de toucher le sol. Sans buffer, un saut sur trois est perdu.

### 36.22.2 — La solution

On ne jette pas l'appui : on le **mémorise** pendant un court instant. C'est déjà ce que fait `sauter()` (36.19.3), qui remplit `_bufferSaut` au lieu de sauter.

```dart
static const double bufferSaut = 0.12;   // secondes
```

```dart
  double _bufferSaut = 0;

  void sauter() {
    _bufferSaut = Constantes.bufferSaut;
  }

  // dans _mettreAJourMinuteurs :
    if (_bufferSaut > 0) _bufferSaut -= dt;
```

`_tenterLeSaut()` est appelée **à chaque frame**. Tant que `_bufferSaut > 0`, elle réessaie. Dès que le joueur touche le sol, le saut part.

---

## 36.23 — Le double saut (optionnel, activable par une constante)

### 36.23.1 — Activable, pas codé en dur

```dart
  // lib/config/constantes.dart
  static const bool doubleSautActif = true;
  static const double forceDoubleSaut = -430.0;
```

Un `bool` en constante permet de comparer les deux versions du jeu en changeant un seul caractère. Comme `doubleSautActif` est `const`, le compilateur supprime purement et simplement la branche morte quand il vaut `false` : le coût est nul.

### 36.23.2 — La logique

```dart
  bool _doubleSautUtilise = false;
```

Trois règles, et trois seulement :

1. le crédit est **remis** à `false` à chaque atterrissage (`_surAtterrissage`, 36.17.3) ;
2. il est **consommé** quand le double saut part ;
3. le premier saut « normal » ne le consomme pas.

```dart
    // dans _tenterLeSaut(), après l'échec du saut au sol :
    if (Constantes.doubleSautActif && !_doubleSautUtilise) {
      _doubleSautUtilise = true;
      _executerSaut(Constantes.forceDoubleSaut);
      _bufferSaut = 0;
    }
```

---

## 36.24 — Les plateformes traversables par le bas

### 36.24.1 — Ce qui est déjà en place

La résolution de 36.18.4 gère déjà deux des trois comportements :

| Situation | Code correspondant | Comportement |
| --- | --- | --- |
| On monte (`dy < 0`) | `if (p.traversable) continue;` | on traverse |
| On se déplace horizontalement | `if (p.traversable) continue;` | on traverse |
| On descend (`dy > 0`) | test sur `basAvant` | on se pose **si** on venait d'au-dessus |

### 36.24.2 — Le test décisif

```dart
        if (p.traversable) {
          if (basAvant > p.boite.top + 1.0) continue;
          if (traverseLesPlateformes) continue;
        }
```

`basAvant` est le bord inférieur du joueur **avant** le déplacement de la frame. S'il était déjà **sous** le dessus de la plateforme, c'est que le joueur remonte au travers : il ne faut pas le poser.

```text
  CAS 1 : le joueur DESCEND        CAS 2 : le joueur REMONTE au travers

    avant  ┌───┐  basAvant = 300     ═════════════  top = 320
           │ J │                     avant  ┌───┐  basAvant = 360
           └───┘                            │ J │
    ═════════════  top = 320                └───┘

    300 <= 321  -> ON SE POSE        360 > 321  -> ON TRAVERSE
```

La tolérance de `1.0` pixel absorbe les erreurs d'arrondi : un joueur posé exactement dessus a `basAvant == top`, et `top > top + 1` est faux, donc il reste posé.

### 36.24.3 — La descente volontaire

Il reste à pouvoir redescendre d'une plateforme traversable en appuyant sur Bas.

```dart
  static const double dureeTraversee = 0.25;   // dans Constantes

  double _minuteurTraversee = 0;

  void descendre() {
    if (!auSol) return;
    _minuteurTraversee = Constantes.dureeTraversee;
    traverseLesPlateformes = true;
    velocite.y = 60;            // un petit coup vers le bas pour décoller
  }

  // dans _mettreAJourMinuteurs :
    if (_minuteurTraversee > 0) _minuteurTraversee -= dt;
    traverseLesPlateformes = _minuteurTraversee > 0 || (_toucheBas && !auSol);
```

Une **durée** plutôt qu'un simple booléen relâché avec la touche : sinon un appui trop bref reposerait le joueur sur la plateforme qu'il vient à peine de quitter. La seconde condition, `_toucheBas && !auSol`, prolonge la traversée tant que le joueur maintient Bas en tombant — le comportement standard du genre.

---

## 36.25 — La machine à états d'animation (rappel chapitres 22 et 29)

### 36.25.1 — Le rappel

Le chapitre 22 a montré comment faire défiler des frames à cadence fixe ; le chapitre 29 a montré `SpriteAnimationComponent` ; le chapitre 26 a introduit les machines à états. Le lien entre les trois : **l'état du personnage décide de l'animation jouée**. Le code de jeu ne dit jamais « joue l'animation de course » ; il dit « le joueur est en état `marche` », et une seule fonction traduit l'état en apparence.

```text
   entrees clavier
        V
   velocite.x , velocite.y , auSol , minuteurs
        V
   _calculerEtat()   ─────>  EtatJoueur (une valeur parmi 7)
        V
   changerEtat()     ─────>  detecte le CHANGEMENT
        V
   _appliquerApparence()  ─> couleur, deformation, (plus tard : animation)
```

### 36.25.2 — La fonction de calcul, dans l'ordre de priorité

C'est le cœur de la machine. L'ordre des tests **est** la priorité des états.

```dart
  EtatJoueur _calculerEtat() {
    // 1. La mort gagne toujours.
    if (pv <= 0) return EtatJoueur.mort;

    // 2. L'attaque est prioritaire tant qu'elle dure.
    if (_minuteurAttaque > 0) return EtatJoueur.attaque;

    // 3. La réaction au dégât.
    if (_minuteurTouche > 0) return EtatJoueur.touche;

    // 4. En l'air : monte ou descend.
    if (!auSol) {
      return velocite.y < 0 ? EtatJoueur.saut : EtatJoueur.chute;
    }

    // 5. Au sol : bouge ou pas.
    if (velocite.x.abs() > Constantes.seuilMarche) {
      return EtatJoueur.marche;
    }

    return EtatJoueur.immobile;
  }
```

```dart
  static const double seuilMarche = 8.0;   // px/s
```

### 36.25.3 — Le tableau de priorité

| Rang | État | Condition | Peut-il être interrompu ? |
| --- | --- | --- | --- |
| 1 | `mort` | `pv <= 0` | non, jamais |
| 2 | `attaque` | `_minuteurAttaque > 0` | seulement par la mort |
| 3 | `touche` | `_minuteurTouche > 0` | par la mort et l'attaque |
| 4 | `saut` | `!auSol && velocite.y < 0` | par tout ce qui précède |
| 5 | `chute` | `!auSol && velocite.y >= 0` | par tout ce qui précède |
| 6 | `marche` | `auSol && |velocite.x| > 8` | par tout ce qui précède |
| 7 | `immobile` | sinon | par tout |

Écrire ce tableau **avant** le code est la bonne méthode. Une machine à états mal ordonnée produit des symptômes déroutants : le personnage attaque et marche en même temps, la mort est annulée par un saut, etc.

---

## 36.26 — Passer d'un état à l'autre proprement

### 36.26.1 — La méthode `changerEtat`

```dart
  void changerEtat(EtatJoueur nouvel) {
    if (etat == nouvel) return;   // rien n'a changé : on sort

    final ancien = etat;
    etat = nouvel;
    _tempsDansEtat = 0;

    _surSortieEtat(ancien);
    _surEntreeEtat(nouvel);
  }
```

Le `if (etat == nouvel) return;` est la ligne la plus importante. Sans elle, `_surEntreeEtat` serait rappelée 60 fois par seconde : le son de saut se jouerait en boucle, l'animation redémarrerait à la frame 0 en permanence et resterait figée sur sa première image.

> **À retenir.** Une transition d'état doit être détectée, pas subie. Le motif `if (nouveau == ancien) return;` est le garde-fou universel des machines à états.

### 36.26.2 — Entrée et sortie d'état

```dart
  void _surEntreeEtat(EtatJoueur e) {
    switch (e) {
      case EtatJoueur.saut:
        _etirement = 1.0;
        break;
      case EtatJoueur.mort:
        velocite.setZero();
        break;
      case EtatJoueur.immobile:
      case EtatJoueur.marche:
      case EtatJoueur.chute:
      case EtatJoueur.attaque:
      case EtatJoueur.touche:
        break;
    }
    _appliquerApparence();
  }

  void _surSortieEtat(EtatJoueur e) {
    if (e == EtatJoueur.attaque) {
      _zoneAttaque?.removeFromParent();
      _zoneAttaque = null;
    }
  }
```

`_surSortieEtat` sert au **nettoyage**. Ici, retirer la zone d'attaque si l'attaque est interrompue. Sans elle, une attaque coupée par un dégât laisserait une hitbox invisible qui blesserait les ennemis pour l'éternité.

Notez que le `switch` couvre **les sept** valeurs, y compris celles qui ne font rien. `default:` aurait été plus court, mais aurait supprimé le contrôle d'exhaustivité du compilateur (chapitre 11). Le jour où vous ajouterez `EtatJoueur.glissade`, il vous le rappellera.

---

## 36.27 — Les animations sans image : couleur et forme par état

### 36.27.1 — Le principe du blockout animé

Sans sprite, on dispose de trois leviers, et ils suffisent à rendre les sept états parfaitement lisibles :

| Levier | Propriété manipulée | Ce qu'il exprime |
| --- | --- | --- |
| La couleur | `_corps.paint.color` | l'état logique |
| La déformation | `_visuel.scale` | la dynamique (saut, atterrissage) |
| Le décalage | `_visuel.position` | le rebond, le pas de course |

### 36.27.2 — La couleur par état

```dart
  Color _couleurEtat() {
    switch (etat) {
      case EtatJoueur.immobile:
        return Palette.joueur;
      case EtatJoueur.marche:
        return Palette.joueurMarche;
      case EtatJoueur.saut:
        return Palette.joueurSaut;
      case EtatJoueur.chute:
        return Palette.joueurChute;
      case EtatJoueur.attaque:
        return Palette.joueurAttaque;
      case EtatJoueur.touche:
        return Palette.danger;
      case EtatJoueur.mort:
        return Palette.joueurMort;
    }
  }

  void _appliquerApparence() {
    _corps.paint.color = _couleurEtat();
  }
```

`Paint` est un objet **mutable** : on modifie sa couleur sans recréer le composant. C'est la manière la moins coûteuse de changer l'apparence d'une forme dans Flame.

### 36.27.3 — La déformation : squash and stretch

Le principe le plus ancien de l'animation : un corps qui accélère s'**étire** dans le sens du mouvement, un corps qui encaisse un choc s'**écrase**. Deux impulsions, `_etirement` (posée au saut) et `_ecrasement` (posée à l'atterrissage), montent à `1.0` puis retombent vers `0` en 0,15 s.

```dart
  void _mettreAJourApparence(double dt) {
    _tempsDansEtat += dt;

    _etirement = _approcher(_etirement, 0, dt / 0.15);
    _ecrasement = _approcher(_ecrasement, 0, dt / 0.15);

    var echelleX = 1.0 - 0.22 * _etirement + 0.30 * _ecrasement;
    var echelleY = 1.0 + 0.28 * _etirement - 0.26 * _ecrasement;
    var decalageY = 0.0;

    switch (etat) {
      case EtatJoueur.immobile:
        decalageY = 0.6 * sin(_tempsDansEtat * 3.0);          // respiration
        break;
      case EtatJoueur.marche:
        final cadence = velocite.x.abs() / Constantes.vitesseJoueur;
        decalageY = -1.6 * sin(_tempsDansEtat * 16.0 * cadence).abs();
        break;
      case EtatJoueur.mort:
        echelleX = 1.35;
        echelleY = 0.35;
        decalageY = size.y * 0.32;                            // aplati au sol
        break;
      // ... chute, attaque, saut, touche : voir 36.CC
      default:
        break;
    }

    _visuel
      ..scale.setValues(echelleX * direction, echelleY)
      ..position.setValues(size.x / 2, size.y / 2 + decalageY);

    _mettreAJourClignotement();     // 36.30
  }
```

`sin` vient de `dart:math` : ajoutez `import 'dart:math';` en tête du fichier. Le `default:` n'apparaît ici que pour raccourcir l'extrait ; le code livré en 36.CC énumère les sept états.

### 36.27.4 — Le retournement fusionné dans l'échelle

Observez la ligne clé :

```dart
      ..scale.setValues(echelleX * direction, echelleY)
```

Le retournement de 36.15 (`scale.x = direction`) et la déformation (`echelleX`) sont **multipliés**. Écrire les deux séparément (`scale.x = direction;` puis `scale.x = echelleX;`) ferait perdre l'un des deux : le second effacerait le premier.

Toute transformation qui s'applique au même canal doit être **combinée en une seule assignation**. C'est une source d'erreur classique dès qu'on empile des effets visuels.

---

## 36.28 — Brancher de vraies animations : le code à substituer

### 36.28.1 — Ce qu'il faut changer, et rien d'autre

Le jour où vous aurez un fichier `assets/images/heros.png`, **trois** éléments changent. Tout le reste — physique, machine à états, entrées — est inchangé.

| Élément | Version blockout (ce chapitre) | Version sprites |
| --- | --- | --- |
| Le composant visuel | `PositionComponent` + `RectangleComponent` | `SpriteAnimationGroupComponent<EtatJoueur>` |
| `_appliquerApparence()` | change `paint.color` | change `current` |
| `onLoad()` | crée trois rectangles | charge la feuille de sprites |

### 36.28.2 — Le code de remplacement

```dart
  // VARIANTE AVEC SPRITES — à substituer au visuel de 36.7.
  late final SpriteAnimationGroupComponent<EtatJoueur> _visuelAnime;

  Future<void> _chargerAnimations() async {
    final image = await game.images.load('heros.png');
    const taille = 32.0;

    SpriteAnimation ligne(int y, int nbFrames, double pas, {bool boucle = true}) {
      return SpriteAnimation.fromFrameData(
        image,
        SpriteAnimationData.sequenced(
          amount: nbFrames,
          stepTime: pas,
          textureSize: Vector2.all(taille),
          texturePosition: Vector2(0, y * taille),
          loop: boucle,
        ),
      );
    }

    _visuelAnime = SpriteAnimationGroupComponent<EtatJoueur>(
      size: Vector2.all(taille),
      anchor: Anchor.center,
      position: Vector2(size.x / 2, size.y / 2),
      current: EtatJoueur.immobile,
      animations: {
        EtatJoueur.immobile: ligne(0, 4, 0.18),
        EtatJoueur.marche: ligne(1, 6, 0.08),
        EtatJoueur.saut: ligne(2, 2, 0.12, boucle: false),
        EtatJoueur.chute: ligne(3, 2, 0.12, boucle: false),
        EtatJoueur.attaque: ligne(4, 4, 0.06, boucle: false),
        EtatJoueur.touche: ligne(5, 2, 0.10, boucle: false),
        EtatJoueur.mort: ligne(6, 5, 0.12, boucle: false),
      },
    );

    await add(_visuelAnime);
  }

  // Et _appliquerApparence() se réduit à ceci :
  void _appliquerApparence() {
    _visuelAnime.current = etat;
  }
```

### 36.28.3 — Les points de vigilance

**Le nom du fichier.** `game.images.load('heros.png')` : on passe `'heros.png'`, **pas** `'assets/images/heros.png'`. Flame ajoute le préfixe lui-même. C'est l'erreur numéro un des débutants avec les assets.

**La déclaration dans `pubspec.yaml`.**

```yaml
flutter:
  assets:
    - assets/images/
```

**La taille du sprite n'est pas la taille de la hitbox.** Le sprite fait 32 x 32 ; le corps physique fait 24 x 30. C'est normal et souhaitable : un sprite comporte presque toujours des marges transparentes. Ne redimensionnez **jamais** `size` du `Joueur` pour l'accorder au sprite ; vous casseriez toute la physique.

**Le retournement reste identique.** `SpriteAnimationGroupComponent` est un `PositionComponent` : `scale.x = -1` avec `Anchor.center` fonctionne exactement comme avec nos rectangles.

**Les animations non bouclées.** `loop: false` pour `mort` : sans cela, le personnage mourrait en boucle. On peut détecter la fin via `animationTicker`.

> **À retenir.** Le fait que le passage aux sprites ne touche que trois endroits n'est pas un hasard : c'est le résultat direct d'avoir séparé le corps physique du visuel (36.7) et la décision de l'affichage (36.25). Un code mal découpé exigerait de tout réécrire.

---

## 36.29 — `attaquer()` : la hitbox d'attaque temporaire

### 36.29.1 — `ZoneAttaque` et `attaquer()`

Attaquer, c'est faire apparaître **une hitbox devant le personnage pendant quelques dixièmes de seconde**, puis la retirer.

```text
     ┌────┐ ┌────────┐
     │ J  │ │  ZONE  │   <- 22 x 20, collee au bord droit du joueur
     └────┘ └────────┘      duree de vie : 0,25 s
```

```dart
  double _minuteurAttaque = 0;
  double _recuperationAttaque = 0;
  ZoneAttaque? _zoneAttaque;

  void attaquer() {
    if (etat == EtatJoueur.mort) return;
    if (_minuteurAttaque > 0 || _recuperationAttaque > 0) return;

    _minuteurAttaque = Constantes.dureeAttaque;            // 0,25 s
    _recuperationAttaque = Constantes.recuperationAttaque; // 0,35 s

    const largeur = Constantes.porteeAttaque;
    final x = direction > 0 ? size.x : -largeur;

    _zoneAttaque = ZoneAttaque(
      position: Vector2(x, 4),
      size: Vector2(largeur, size.y - 10),
      degats: Constantes.degatsAttaqueJoueur,
    );
    add(_zoneAttaque!);
  }
```

`ZoneAttaque` (code complet en 36.CC) est un `PositionComponent with CollisionCallbacks` qui porte un rectangle doré translucide et une `RectangleHitbox` active. Son `onCollisionStart` sera branché sur les ennemis au chapitre 37.

Elle est ajoutée **comme enfant du joueur** : elle suit donc automatiquement ses déplacements pendant toute la durée du coup, sans une ligne de code. C'est le bénéfice de l'arbre, exposé au chapitre 28.

`recuperationAttaque` empêche le joueur de marteler la touche : il ne peut attaquer qu'environ trois fois par seconde. Sans cette limite, le combat du chapitre 37 se réduirait à maintenir une touche.

### 36.29.2 — Le retrait

Deux chemins mènent au retrait, et il faut les deux :

```dart
  // dans _mettreAJourMinuteurs :
    if (_minuteurAttaque > 0) {
      _minuteurAttaque -= dt;
      if (_minuteurAttaque <= 0) {
        _zoneAttaque?.removeFromParent();
        _zoneAttaque = null;
      }
    }
    if (_recuperationAttaque > 0) _recuperationAttaque -= dt;
```

Le second chemin est `_surSortieEtat` (36.26.2), qui nettoie si l'attaque est interrompue par un dégât ou par la mort. Une hitbox oubliée dans l'arbre est un bogue redoutable : invisible, elle continue de blesser.

---

## 36.30 — `subirDegats()` et l'invincibilité temporaire clignotante (rappel chapitre 33)

### 36.30.1 — Le code

Un dégât produit trois effets : une perte de points de vie, un **recul**, et une **invincibilité temporaire** sans laquelle un joueur collé à un ennemi perdrait 100 points de vie en moins d'une seconde.

```dart
  void subirDegats(double degats) {
    if (invincible || etat == EtatJoueur.mort) return;

    pv -= degats;
    if (pv <= 0) {
      pv = 0;
      mourir();
      return;
    }

    invincible = true;
    _minuteurInvincibilite = Constantes.dureeInvincibilite;  // 1,2 s
    _minuteurTouche = Constantes.dureeTouche;                // 0,25 s

    velocite.x = Constantes.reculDegats * -direction;
    velocite.y = -Constantes.reculDegatsVertical;
    auSol = false;

    changerEtat(EtatJoueur.touche);
  }
```

Le premier `if` est le garde-fou : **toute** entrée dans la méthode est refusée si le joueur est déjà invincible ou mort. C'est ici, et nulle part ailleurs, que se décide qui peut blesser le joueur.

Remarquez que `_minuteurTouche` (0,25 s) est bien plus court que `_minuteurInvincibilite` (1,2 s). L'état `touche` ne dure qu'un quart de seconde ; l'invincibilité, elle, se poursuit pendant que le joueur a de nouveau la main. Les deux minuteurs sont décomptés dans `_mettreAJourMinuteurs`, qui remet `invincible` à `false` et rétablit l'opacité à l'expiration.

### 36.30.2 — Le clignotement

Le chapitre 33 propose `OpacityEffect`. Ici, une solution plus directe suffit : modifier l'alpha du `Paint` de chaque rectangle.

```dart
  static const double periodeClignotement = 0.12;   // ~8 clignotements / s

  void _mettreAJourClignotement() {
    if (!invincible) return;
    final phase =
        (_minuteurInvincibilite / Constantes.periodeClignotement) % 1.0;
    _appliquerAlpha(phase < 0.5 ? 90 : 255);
  }

  void _appliquerAlpha(int alpha) {
    _corps.paint.color = _corps.paint.color.withAlpha(alpha);
    _oeil.paint.color = _oeil.paint.color.withAlpha(alpha);
    _echarpe.paint.color = _echarpe.paint.color.withAlpha(alpha);
  }
```

`Color.withAlpha(int)` prend une valeur de 0 (transparent) à 255 (opaque). Nous alternons entre 90 et 255 : le personnage reste visible en permanence, ce qui est préférable à une disparition complète — un joueur invisible est un joueur qui tombe dans un trou.

> **Piège.** N'oubliez pas `_appliquerAlpha(255)` à l'expiration de l'invincibilité. Si le minuteur expire pendant une phase basse, le personnage resterait translucide définitivement.

---

## 36.31 — `mourir()` et la remise à zéro

### 36.31.1 — La méthode

```dart
  void mourir() {
    if (etat == EtatJoueur.mort) return;

    pv = 0;
    invincible = true;
    velocite.setZero();
    _zoneAttaque?.removeFromParent();
    _zoneAttaque = null;
    _appliquerAlpha(255);

    changerEtat(EtatJoueur.mort);

    add(
      TimerComponent(
        period: Constantes.delaiReapparition,   // 1,1 s
        removeOnFinish: true,
        onTick: reinitialiser,
      ),
    );
  }
```

`TimerComponent` (chapitre 33) est préférable à un champ `double` décompté à la main : il se supprime tout seul grâce à `removeOnFinish: true`, et il est gelé en même temps que le jeu quand on appelle `pauseEngine()`.

### 36.31.2 — La remise à zéro

```dart
  void reinitialiser() {
    position.setFrom(positionDepart);
    velocite.setZero();
    pv = Constantes.pvJoueurMax;
    invincible = false;
    auSol = false;
    direction = 1;
    cles = 0;
    // ... TOUS les minuteurs et drapeaux : voir 36.CC
    _appliquerAlpha(255);
    changerEtat(EtatJoueur.immobile);

    // Chapitre 37 : game.perdreUneVie();
  }
```

Une remise à zéro doit être **exhaustive**. Un minuteur oublié se traduit par un joueur qui réapparaît invincible, ou qui ne peut plus sauter. Le réflexe : quand vous ajoutez un champ d'état au joueur, ajoutez-le **immédiatement** à `reinitialiser()`.

`position.setFrom(positionDepart)` copie les composantes sans changer d'objet `Vector2` — c'est plus sûr que `position = positionDepart`, qui ferait pointer le composant sur la constante elle-même.

### 36.31.3 — La chute mortelle

```dart
  void _verifierChuteMortelle() {
    if (etat == EtatJoueur.mort) return;
    if (position.y > Constantes.limiteChute) {
      mourir();
    }
  }
```

```dart
  static const double limiteChute = 1200.0;   // pixels
```

Sans cette vérification, un joueur tombé dans le trou de la salle de test descendrait indéfiniment, hors de toute plateforme, et la caméra le suivrait dans le vide. `limiteChute` doit se situer nettement sous le point le plus bas du niveau : ici, la salle fait 18 tuiles, soit 576 pixels ; 1200 laisse une marge confortable.

---

## 36.32 — Faire suivre la caméra (rappel chapitre 31)

### 36.32.1 — Une ligne

```dart
  // dans DonjonGame.ajouterJoueur()
  camera.follow(joueur, maxSpeed: 400);
  camera.viewfinder.zoom = Constantes.zoomCamera;
```

Rappel du chapitre 31 et de `_REF-FLAME.md` : la méthode s'appelle `follow`. `followComponent` appartient à l'ancienne caméra supprimée depuis la version 1.5 de Flame ; de même, `camera.zoom = 2` n'existe pas, c'est `camera.viewfinder.zoom`.

### 36.32.2 — Le paramètre `maxSpeed`

```text
  camera.follow(joueur)                    la camera est COLLEE au joueur
                                           -> chaque a-coup est retransmis

  camera.follow(joueur, maxSpeed: 400)     la camera rattrape a 400 px/s max
                                           -> leger retard, mouvement fluide
```

Le joueur se déplace à 180 px/s au sol et jusqu'à 900 px/s en chute libre. Avec `maxSpeed: 400`, la caméra suit sans retard perceptible en course, mais **prend du retard** pendant une chute rapide — ce qui est exactement l'effet recherché : on voit arriver le sol.

### 36.32.3 — Le point suivi

`follow` suit la `position` du composant, donc son **ancre**. Comme l'ancre du joueur est `Anchor.topLeft`, la caméra centrerait le coin haut-gauche, décalant l'image d'une demi-largeur et d'une demi-hauteur — 24 et 30 pixels d'écran au zoom 2. C'est visible.

La correction tient en trois lignes : un composant vide, enfant du joueur, placé exactement en son centre. `follow` accepte tout `ReadOnlyPositionProvider`, et un `PositionComponent` enfant en est un.

```dart
  // dans Joueur.onLoad()
  cibleCamera = PositionComponent(position: Vector2(size.x / 2, size.y / 2));
  await add(cibleCamera);
```

```dart
  // dans DonjonGame.ajouterJoueur()
  camera.follow(joueur.cibleCamera, maxSpeed: 400);
```

---

## 36.33 — Régler le game feel : le tableau des valeurs à essayer

### 36.33.1 — Le tableau de référence

| Constante | Valeur retenue | Plage utile | Trop bas | Trop haut |
| --- | --- | --- | --- | --- |
| `gravite` | 1200 | 800 – 2000 | flottement lunaire | chute brutale, sauts inutilisables |
| `vitesseJoueur` | 180 | 120 – 260 | lent, ennuyeux | incontrôlable dans les couloirs |
| `forceSaut` | -520 | -400 à -700 | on ne monte plus | on sort de l'écran |
| `vitesseMaxChute` | 900 | 600 – 1400 | chute molle | risque de tunneling |
| `accelerationSol` | 1500 | 800 – 4000 | départ pâteux | équivaut à une vitesse constante |
| `decelerationSol` | 2200 | 1000 – 6000 | patinage sur glace | arrêt sec, sans poids |
| `accelerationAir` | 900 | 300 – 1800 | aucun contrôle en l'air | l'air se pilote comme le sol |
| `decelerationAir` | 700 | 200 – 1500 | on garde tout son élan | l'élan disparaît en l'air |
| `coupureSaut` | 0.42 | 0.2 – 0.7 | saut court trop court | pas de saut variable |
| `multiplicateurChute` | 1.55 | 1.0 – 2.5 | saut symétrique, mou | retombée en pierre |
| `coyoteTime` | 0.10 | 0.05 – 0.15 | sauts « volés » | personnage aimanté aux bords |
| `bufferSaut` | 0.12 | 0.08 – 0.20 | sauts « avalés » | sauts fantômes |
| `forceDoubleSaut` | -430 | -300 à -520 | inutile | vol libre |
| `dureeInvincibilite` | 1.2 | 0.8 – 2.0 | mort par contact prolongé | joueur invulnérable |
| `zoomCamera` | 2.0 | 1.5 – 3.0 | on ne voit plus le personnage | on ne voit plus le niveau |

### 36.33.2 — Trois recettes prêtes à l'emploi

| Style | `gravite` | `vitesseJoueur` | `forceSaut` | `accelerationSol` | `decelerationSol` |
| --- | --- | --- | --- | --- | --- |
| **Nerveux** (plateforme moderne) | 1800 | 220 | -620 | 4000 | 6000 |
| **Équilibré** (retenu) | 1200 | 180 | -520 | 1500 | 2200 |
| **Lourd** (jeu d'exploration) | 900 | 140 | -430 | 700 | 900 |

Copiez l'une de ces lignes dans `Constantes` et jouez. Vous entendrez la différence avant même de la comprendre.

---

## 36.34 — Ce que le projet fait à la fin de ce chapitre

Lancez `flutter run`. Le menu principal du chapitre 35 s'affiche. Cliquez sur « Jouer ».

```text
  Vous obtenez un jeu de plateforme jouable :

  - un personnage bleu de 24 x 30 pixels apparait en bas a gauche ;
  - ZQSD, WASD ou les fleches le deplacent ;
  - il accelere, decelere, se retourne et oscille en courant ;
  - Espace le fait sauter ; un appui bref saute moins haut ;
  - il peut sauter juste apres avoir quitte un rebord (coyote time) ;
  - il peut appuyer juste avant d'atterrir (jump buffering) ;
  - il dispose d'un double saut, desactivable par une constante ;
  - il traverse les plateformes claires par le bas et se pose dessus ;
  - la touche Bas le fait redescendre au travers ;
  - E ou X declenche une attaque : une zone doree apparait devant lui ;
  - sa couleur et sa forme changent selon ses sept etats ;
  - un degat le fait clignoter et le rend invincible 1,2 s ;
  - une chute dans le trou le tue, puis il reapparait au depart ;
  - la camera le suit avec un leger retard, au zoom x2 ;
  - le tout sans un seul fichier image.
```

**Ce qui manque encore, et qui arrive ensuite :**

| Manque | Chapitre |
| --- | --- |
| Des ennemis, une IA, un vrai combat | 37 |
| Des pièces, des potions, des clés, un HUD | 38 |
| De vrais niveaux, des portes, un boss | 39 |
| Du son, la pause, le Game Over, la sauvegarde | 40 |

---

## 36.35 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le personnage traverse le sol à grande vitesse | tunneling : le déplacement d'une frame dépasse l'épaisseur de la plateforme | plafonner la chute avec `vitesseMaxChute` (36.11) ; vérifier `vitesseMax * dt < épaisseur` |
| Le personnage escalade les murs | résolution des deux axes en même temps | résoudre X puis Y séparément (36.18) |
| Le personnage vibre entre `immobile` et `chute` | `velocite.y` forcée à 0 quand `auSol`, donc plus d'intersection à la frame suivante | ne jamais annuler la gravité ; laisser la résolution remettre `velocite.y = 0` (36.17.3) |
| Le personnage se téléporte à gauche quand il se retourne | `scale.x = -1` appliqué à un composant d'ancre `topLeft` | mettre le retournement sur un enfant d'ancre `Anchor.center` (36.15) |
| L'animation reste bloquée sur sa première image | `_surEntreeEtat` rappelée à chaque frame | tester `if (etat == nouvel) return;` dans `changerEtat` (36.26.1) |
| Le joueur réapparaît là où il est mort | `positionDepart` et `position` partagent le même `Vector2` | `positionDepart = position.clone()` (36.5.2) |
| Le joueur perd toute sa vie en une seconde | pas d'invincibilité après un dégât | `if (invincible) return;` en tête de `subirDegats` (36.30.1) |
| Le joueur reste translucide définitivement | le minuteur d'invincibilité expire pendant une phase basse du clignotement | `_appliquerAlpha(255)` à la fin de l'invincibilité (36.30.2) |
| Le saut part 60 fois par seconde | saut déclenché depuis `keysPressed` au lieu d'un front | `if (event is KeyDownEvent)` (36.12.2) |
| Le clavier ne répond pas du tout | `KeyboardHandler` sur le composant, mais `HasKeyboardHandlerComponents` absent du jeu | ajouter le mixin au jeu ; ne pas y mettre `KeyboardEvents` en même temps |
| Le personnage se coince dans les couloirs d'une tuile | largeur du joueur égale à la taille de tuile | viser ~75 % de la tuile : `largeurJoueur` 24 pour une tuile de 32 |
| `activeCollisions` toujours vide, `isColliding` faux | `super.onCollisionStart(...)` non appelé | appeler `super` : la méthode est `@mustCallSuper` (36.16.3) |
| Le jeu va deux fois plus vite sur un écran 120 Hz | déplacement écrit sans `dt` | toujours `position += velocite * dt` (36.9.2) |
| Une hitbox d'attaque blesse pour l'éternité | attaque interrompue sans nettoyage | retirer la zone dans `_surSortieEtat` **et** à l'expiration du minuteur (36.29.2) |
| Le joueur ne peut plus sauter après un respawn | minuteurs non remis à zéro | rendre `reinitialiser()` exhaustive (36.31.2) |
| Les plateformes traversables bloquent par le haut | `basAvant` non mémorisé avant le déplacement | conserver `bas` en début de `deplacerAvecCollisions` (36.18.2) |
| `camera.followComponent(joueur)` ne compile pas | API supprimée depuis Flame 1.5 | `camera.follow(joueur)` |
| `camera.zoom = 2` ne compile pas | propriété inexistante sur `CameraComponent` | `camera.viewfinder.zoom = 2` |
| L'image ne se charge pas malgré un chemin correct | chemin complet passé à `images.load` | passer `'heros.png'`, Flame préfixe `assets/images/` (36.28.3) |
| Le double saut se déclenche deux fois de suite | `_coyote` non consommé après un saut | `_coyote = 0` dans `_tenterLeSaut` (36.21.3) |

---

## 36.36 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `Entite` | classe abstraite : `PositionComponent` + `HasGameReference<DonjonGame>` + vélocité + gravité + collisions |
| Ancre | `Anchor.topLeft` pour le corps physique, `Anchor.center` pour le visuel transformé |
| `Plateforme` | `PositionComponent` immobile, `boite` publique, hitbox `passive`, champ `traversable` |
| Intégration | `velocite += acceleration * dt` **puis** `position += velocite * dt` |
| `dt` | toujours en secondes, toujours multiplié : sans lui le jeu dépend du framerate |
| Gravité | `velocite.y += gravite * dt`, plafonnée par `vitesseMaxChute` pour éviter le tunneling |
| Axe Y | croissant vers le **bas** : le saut est négatif, la chute positive |
| Clavier | `keysPressed` pour l'état continu, `event is KeyDownEvent` pour les actions ponctuelles |
| Accélération | approcher une vitesse cible sans la dépasser (*move towards*) : c'est ce qui donne une masse |
| Retournement | `scale.x = -1` sur un enfant d'ancre `center` ; assigner, ne jamais basculer |
| Hitbox | Flame **signale** les contacts, il ne les résout pas |
| Deux systèmes | résolution manuelle contre le décor, `CollisionCallbacks` pour les événements |
| `auSol` | recalculé à chaque frame, jamais mémorisé |
| Axe par axe | déplacer en X puis résoudre, déplacer en Y puis résoudre : plus aucune ambiguïté |
| Saut | `velocite.y = force` (assignation, pas addition) : tous les sauts sont identiques |
| Saut variable | couper la montée au relâchement + gravité asymétrique en descente |
| Coyote time | on peut encore sauter 0,10 s après avoir quitté le sol |
| Jump buffering | un appui est mémorisé 0,12 s et s'exécute dès l'atterrissage |
| Double saut | un crédit remis à zéro à chaque atterrissage, activable par une constante `const` |
| Plateforme `=` | on la franchit par le bas et sur les côtés ; on ne s'y pose que si `basAvant <= top` |
| Machine à états | une fonction qui **calcule** l'état, une fonction qui **détecte la transition** |
| `changerEtat` | commence toujours par `if (etat == nouvel) return;` |
| Apparence | couleur = état logique ; échelle = dynamique ; les deux se combinent en une assignation |
| Sprites | trois points à changer seulement, parce que physique et visuel sont séparés |
| Attaque | une hitbox enfant, temporaire, nettoyée par deux chemins distincts |
| Invincibilité | garde-fou en tête de `subirDegats`, clignotement par alpha, retour à 255 en sortie |
| `reinitialiser()` | doit être exhaustive : tout champ ajouté doit y figurer |
| Caméra | `camera.follow(cible, maxSpeed: 400)` et `camera.viewfinder.zoom` |
| Game feel | une constante à la fois, trente secondes de jeu, une grille de contrôle |

---

## 36.CC — Code complet du chapitre

Les six fichiers, dans l'ordre de dépendance. Recopiez-les tels quels : le projet compile et se lance.

### `lib/config/constantes.dart`

```dart
class Constantes {
  // --- Chapitre 35 ---
  static const double tailleTuile = 32.0;
  static const double gravite = 1200.0;
  static const double vitesseJoueur = 180.0;
  static const double forceSaut = -520.0;
  static const double vitesseMaxChute = 900.0;
  static const int viesDepart = 3;
  static const double pvJoueurMax = 100.0;
  static const double dureeInvincibilite = 1.2;
  static const double zoomCamera = 2.0;
  static const int nombreNiveaux = 3;

  // --- Chapitre 36 : gabarit du joueur ---
  static const double largeurJoueur = 24.0;
  static const double hauteurJoueur = 30.0;

  // --- Chapitre 36 : déplacement horizontal ---
  static const double accelerationSol = 1500.0;
  static const double accelerationAir = 900.0;
  static const double decelerationSol = 2200.0;
  static const double decelerationAir = 700.0;
  static const double seuilMarche = 8.0;

  // --- Chapitre 36 : saut ---
  static const double coupureSaut = 0.42;
  static const double multiplicateurChute = 1.55;
  static const double multiplicateurMontee = 0.88;
  static const double coyoteTime = 0.10;
  static const double bufferSaut = 0.12;
  static const bool doubleSautActif = true;
  static const double forceDoubleSaut = -430.0;
  static const double dureeTraversee = 0.25;

  // --- Chapitre 36 : combat et dégâts ---
  static const double dureeAttaque = 0.25;
  static const double recuperationAttaque = 0.35;
  static const double porteeAttaque = 22.0;
  static const double degatsAttaqueJoueur = 25.0;
  static const double dureeTouche = 0.25;
  static const double reculDegats = 200.0;
  static const double reculDegatsVertical = 240.0;
  static const double periodeClignotement = 0.12;

  // --- Chapitre 36 : mort ---
  static const double delaiReapparition = 1.1;
  static const double limiteChute = 1200.0;
}
```

### `lib/config/palette.dart`

```dart
import 'package:flutter/material.dart';

class Palette {
  // --- Chapitre 35 ---
  static const Color fond = Color(0xFF10131A);
  static const Color texte = Color(0xFFECEFF4);
  static const Color accent = Color(0xFFFFC107);
  static const Color danger = Color(0xFFE53935);

  // --- Chapitre 36 : décor ---
  static const Color plateforme = Color(0xFF4E342E);
  static const Color plateformeHaut = Color(0xFF8D6E63);
  static const Color plateformeLegere = Color(0xFF6D4C41);
  static const Color plateformeLegereHaut = Color(0xFFBCAAA4);

  // --- Chapitre 36 : joueur, une couleur par état ---
  static const Color joueur = Color(0xFF4FC3F7);
  static const Color joueurMarche = Color(0xFF29B6F6);
  static const Color joueurSaut = Color(0xFF26C6DA);
  static const Color joueurChute = Color(0xFF5C6BC0);
  static const Color joueurAttaque = Color(0xFFFFD54F);
  static const Color joueurMort = Color(0xFF78909C);
  static const Color joueurOeil = Color(0xFF0D1B2A);
}
```

### `lib/composants/plateforme.dart`

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';

/// Un bloc de décor. Si [traversable] vaut true, l'entité peut le franchir
/// par le bas et par les côtés, mais se pose dessus en descendant.
class Plateforme extends PositionComponent {
  Plateforme({
    required Vector2 position,
    required Vector2 size,
    this.traversable = false,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  /// Construit une plateforme à partir de coordonnées en TUILES.
  factory Plateforme.tuiles(
    int colonne,
    int ligne,
    int nombreDeColonnes, {
    int nombreDeLignes = 1,
    bool traversable = false,
  }) {
    const t = Constantes.tailleTuile;
    return Plateforme(
      position: Vector2(t * colonne, t * ligne),
      size: Vector2(t * nombreDeColonnes, t * nombreDeLignes),
      traversable: traversable,
    );
  }

  final bool traversable;

  Rect get boite => Rect.fromLTWH(position.x, position.y, size.x, size.y);

  @override
  Future<void> onLoad() async {
    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()
          ..color = traversable ? Palette.plateformeLegere : Palette.plateforme,
      ),
    );
    await add(
      RectangleComponent(
        size: Vector2(size.x, traversable ? 3 : 5),
        paint: Paint()
          ..color = traversable
              ? Palette.plateformeLegereHaut
              : Palette.plateformeHaut,
      ),
    );
    await add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}
```

### `lib/core/entite.dart`

```dart
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../composants/plateforme.dart';
import '../config/constantes.dart';
import '../donjon_game.dart';

/// Base commune de tout ce qui bouge : joueur, ennemis, boss.
abstract class Entite extends PositionComponent
    with HasGameReference<DonjonGame> {
  Entite({
    super.position,
    super.size,
    super.priority,
  }) : super(anchor: Anchor.topLeft);

  /// Vitesse en pixels par seconde. y > 0 = vers le bas.
  Vector2 velocite = Vector2.zero();

  /// Recalculé à chaque frame par [deplacerAvecCollisions].
  bool auSol = false;

  /// 1 = regarde à droite, -1 = regarde à gauche.
  int direction = 1;

  /// Vrai pendant une descente volontaire à travers une plateforme.
  bool traverseLesPlateformes = false;

  double get gauche => position.x;
  double get droite => position.x + size.x;
  double get haut => position.y;
  double get bas => position.y + size.y;

  Vector2 get centre =>
      Vector2(position.x + size.x / 2, position.y + size.y / 2);

  Rect get boite => Rect.fromLTWH(position.x, position.y, size.x, size.y);

  Iterable<Plateforme> get plateformes =>
      game.monde.children.query<Plateforme>();

  void appliquerGravite(double dt, {double multiplicateur = 1.0}) {
    velocite.y += Constantes.gravite * multiplicateur * dt;
    if (velocite.y > Constantes.vitesseMaxChute) {
      velocite.y = Constantes.vitesseMaxChute;
    }
  }

  /// Déplace l'entité et résout les collisions AXE PAR AXE.
  void deplacerAvecCollisions(double dt) {
    final basAvant = bas;
    final auSolAvant = auSol;

    _deplacerEnX(velocite.x * dt);
    _deplacerEnY(velocite.y * dt, basAvant);

    if (auSol && !auSolAvant) {
      surAtterrissage();
    }
  }

  /// Appelée à la frame exacte où l'entité vient de toucher le sol.
  void surAtterrissage() {}

  void _deplacerEnX(double dx) {
    if (dx == 0) return;
    position.x += dx;

    for (final p in plateformes) {
      if (p.traversable) continue;
      if (!boite.overlaps(p.boite)) continue;

      if (dx > 0) {
        position.x = p.boite.left - size.x;
      } else {
        position.x = p.boite.right;
      }
      velocite.x = 0;
    }
  }

  void _deplacerEnY(double dy, double basAvant) {
    position.y += dy;
    auSol = false;

    for (final p in plateformes) {
      if (!boite.overlaps(p.boite)) continue;

      if (dy > 0) {
        if (p.traversable) {
          if (basAvant > p.boite.top + 1.0) continue;
          if (traverseLesPlateformes) continue;
        }
        position.y = p.boite.top - size.y;
        velocite.y = 0;
        auSol = true;
      } else if (dy < 0) {
        if (p.traversable) continue;
        position.y = p.boite.bottom;
        velocite.y = 0;
      }
    }
  }
}
```

### `lib/composants/joueur.dart`

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../core/entite.dart';
import 'plateforme.dart';

enum EtatJoueur { immobile, marche, saut, chute, attaque, touche, mort }

class Joueur extends Entite with KeyboardHandler, CollisionCallbacks {
  Joueur({required Vector2 position})
      : positionDepart = position.clone(),
        super(
          position: position.clone(),
          size: Vector2(Constantes.largeurJoueur, Constantes.hauteurJoueur),
          priority: 10,
        );

  final Vector2 positionDepart;

  // --- Contrat de _SPEC-JEU-2C.md ---
  EtatJoueur etat = EtatJoueur.immobile;
  double pv = Constantes.pvJoueurMax;
  bool invincible = false;
  int cles = 0;

  // --- Visuel ---
  late final PositionComponent _visuel;
  late final RectangleComponent _corps;
  late final RectangleComponent _oeil;
  late final RectangleComponent _echarpe;
  late final PositionComponent cibleCamera;

  // --- Entrées mémorisées ---
  int _entreeHorizontale = 0;
  bool _toucheBas = false;
  bool _sautMaintenu = false;

  // --- Minuteurs ---
  double _coyote = 0;
  double _bufferSaut = 0;
  double _minuteurAttaque = 0;
  double _recuperationAttaque = 0;
  double _minuteurTouche = 0;
  double _minuteurInvincibilite = 0;
  double _minuteurTraversee = 0;
  double _tempsDansEtat = 0;

  bool _doubleSautUtilise = false;
  double _etirement = 0;
  double _ecrasement = 0;
  ZoneAttaque? _zoneAttaque;

  // CHARGEMENT

  @override
  Future<void> onLoad() async {
    _visuel = PositionComponent(
      size: size.clone(),
      anchor: Anchor.center,
      position: Vector2(size.x / 2, size.y / 2),
    );

    _corps = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.joueur,
    );
    _echarpe = RectangleComponent(
      position: Vector2(-2, 10),
      size: Vector2(9, 4),
      paint: Paint()..color = Palette.accent,
    );
    _oeil = RectangleComponent(
      position: Vector2(size.x - 8, 7),
      size: Vector2(5, 5),
      paint: Paint()..color = Palette.joueurOeil,
    );

    await _visuel.addAll([_corps, _echarpe, _oeil]);
    await add(_visuel);

    await add(
      RectangleHitbox(
        size: Vector2(size.x - 4, size.y - 2),
        position: Vector2(2, 2),
        collisionType: CollisionType.active,
      ),
    );

    // Point suivi par la caméra : le centre du personnage.
    cibleCamera = PositionComponent(position: Vector2(size.x / 2, size.y / 2));
    await add(cibleCamera);
  }

  // BOUCLE

  @override
  void update(double dt) {
    super.update(dt);

    if (etat == EtatJoueur.mort) {
      appliquerGravite(dt);
      deplacerAvecCollisions(dt);
      _mettreAJourApparence(dt);
      return;
    }

    _mettreAJourMinuteurs(dt);
    _mettreAJourHorizontal(dt);
    _tenterLeSaut();
    appliquerGravite(dt, multiplicateur: _multiplicateurGravite);
    deplacerAvecCollisions(dt);
    _mettreAJourEtat();
    _mettreAJourApparence(dt);
    _verifierChuteMortelle();
  }

  void _mettreAJourMinuteurs(double dt) {
    if (auSol) {
      _coyote = Constantes.coyoteTime;
    } else {
      _coyote -= dt;
    }

    if (_bufferSaut > 0) _bufferSaut -= dt;
    if (_recuperationAttaque > 0) _recuperationAttaque -= dt;
    if (_minuteurTouche > 0) _minuteurTouche -= dt;
    if (_minuteurTraversee > 0) _minuteurTraversee -= dt;

    traverseLesPlateformes =
        _minuteurTraversee > 0 || (_toucheBas && !auSol);

    if (_minuteurAttaque > 0) {
      _minuteurAttaque -= dt;
      if (_minuteurAttaque <= 0) {
        _zoneAttaque?.removeFromParent();
        _zoneAttaque = null;
      }
    }

    if (_minuteurInvincibilite > 0) {
      _minuteurInvincibilite -= dt;
      if (_minuteurInvincibilite <= 0) {
        invincible = false;
        _appliquerAlpha(255);
      }
    }
  }

  // ENTRÉES

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (etat == EtatJoueur.mort) return true;

    final gauche = keysPressed.contains(LogicalKeyboardKey.arrowLeft) ||
        keysPressed.contains(LogicalKeyboardKey.keyQ) ||
        keysPressed.contains(LogicalKeyboardKey.keyA);
    final droite = keysPressed.contains(LogicalKeyboardKey.arrowRight) ||
        keysPressed.contains(LogicalKeyboardKey.keyD);

    _entreeHorizontale = (droite ? 1 : 0) - (gauche ? 1 : 0);

    _toucheBas = keysPressed.contains(LogicalKeyboardKey.arrowDown) ||
        keysPressed.contains(LogicalKeyboardKey.keyS);

    _sautMaintenu = keysPressed.contains(LogicalKeyboardKey.space) ||
        keysPressed.contains(LogicalKeyboardKey.arrowUp) ||
        keysPressed.contains(LogicalKeyboardKey.keyW) ||
        keysPressed.contains(LogicalKeyboardKey.keyZ);

    if (event is KeyDownEvent) {
      final t = event.logicalKey;
      if (_estToucheDeSaut(t)) sauter();
      if (t == LogicalKeyboardKey.keyE || t == LogicalKeyboardKey.keyX) {
        attaquer();
      }
      if (t == LogicalKeyboardKey.arrowDown || t == LogicalKeyboardKey.keyS) {
        descendre();
      }
    }

    if (event is KeyUpEvent && _estToucheDeSaut(event.logicalKey)) {
      _couperLeSaut();
    }

    return true;
  }

  static bool _estToucheDeSaut(LogicalKeyboardKey t) =>
      t == LogicalKeyboardKey.space ||
      t == LogicalKeyboardKey.arrowUp ||
      t == LogicalKeyboardKey.keyW ||
      t == LogicalKeyboardKey.keyZ;

  // DÉPLACEMENT

  void _mettreAJourHorizontal(double dt) {
    if (_entreeHorizontale != 0) {
      direction = _entreeHorizontale;
    }

    final cible = Constantes.vitesseJoueur * _entreeHorizontale;

    final double taux;
    if (_entreeHorizontale == 0) {
      taux = auSol ? Constantes.decelerationSol : Constantes.decelerationAir;
    } else {
      taux = auSol ? Constantes.accelerationSol : Constantes.accelerationAir;
    }

    velocite.x = _approcher(velocite.x, cible, taux * dt);

    if (_entreeHorizontale == 0 && velocite.x.abs() < 1.0) {
      velocite.x = 0;
    }
  }

  /// Rapproche [valeur] de [cible] d'au plus [pas], sans jamais la dépasser.
  static double _approcher(double valeur, double cible, double pas) {
    if (valeur < cible) {
      final v = valeur + pas;
      return v > cible ? cible : v;
    }
    if (valeur > cible) {
      final v = valeur - pas;
      return v < cible ? cible : v;
    }
    return cible;
  }

  double get _multiplicateurGravite {
    if (velocite.y < 0) {
      return _sautMaintenu ? Constantes.multiplicateurMontee : 1.0;
    }
    return Constantes.multiplicateurChute;
  }

  // SAUT

  /// Demande un saut. L'exécution effective a lieu dans update().
  void sauter() {
    _bufferSaut = Constantes.bufferSaut;
  }

  void _tenterLeSaut() {
    if (_bufferSaut <= 0) return;

    if (auSol || _coyote > 0) {
      _executerSaut(Constantes.forceSaut);
      _bufferSaut = 0;
      _coyote = 0;
      return;
    }

    if (Constantes.doubleSautActif && !_doubleSautUtilise) {
      _doubleSautUtilise = true;
      _executerSaut(Constantes.forceDoubleSaut);
      _bufferSaut = 0;
    }
  }

  void _executerSaut(double force) {
    velocite.y = force;
    auSol = false;
    _etirement = 1.0;
  }

  void _couperLeSaut() {
    if (velocite.y < 0) {
      velocite.y *= Constantes.coupureSaut;
    }
  }

  void descendre() {
    if (!auSol) return;
    _minuteurTraversee = Constantes.dureeTraversee;
    traverseLesPlateformes = true;
    velocite.y = 60;
  }

  @override
  void surAtterrissage() {
    _ecrasement = 1.0;
    _doubleSautUtilise = false;
  }

  // COMBAT

  void attaquer() {
    if (etat == EtatJoueur.mort) return;
    if (_minuteurAttaque > 0 || _recuperationAttaque > 0) return;

    _minuteurAttaque = Constantes.dureeAttaque;
    _recuperationAttaque = Constantes.recuperationAttaque;

    const largeur = Constantes.porteeAttaque;
    final x = direction > 0 ? size.x : -largeur;

    _zoneAttaque = ZoneAttaque(
      position: Vector2(x, 4),
      size: Vector2(largeur, size.y - 10),
      degats: Constantes.degatsAttaqueJoueur,
    );
    add(_zoneAttaque!);
  }

  void subirDegats(double degats) {
    if (invincible || etat == EtatJoueur.mort) return;

    pv -= degats;
    if (pv <= 0) {
      pv = 0;
      mourir();
      return;
    }

    invincible = true;
    _minuteurInvincibilite = Constantes.dureeInvincibilite;
    _minuteurTouche = Constantes.dureeTouche;

    velocite.x = Constantes.reculDegats * -direction;
    velocite.y = -Constantes.reculDegatsVertical;
    auSol = false;

    changerEtat(EtatJoueur.touche);
  }

  void mourir() {
    if (etat == EtatJoueur.mort) return;

    pv = 0;
    invincible = true;
    velocite.setZero();
    _zoneAttaque?.removeFromParent();
    _zoneAttaque = null;
    _appliquerAlpha(255);

    changerEtat(EtatJoueur.mort);

    add(
      TimerComponent(
        period: Constantes.delaiReapparition,
        removeOnFinish: true,
        onTick: reinitialiser,
      ),
    );
  }

  void reinitialiser() {
    position.setFrom(positionDepart);
    velocite.setZero();
    pv = Constantes.pvJoueurMax;
    invincible = false;
    auSol = false;
    direction = 1;
    cles = 0;
    _bufferSaut = 0;
    _coyote = 0;
    _doubleSautUtilise = false;
    _minuteurAttaque = 0;
    _minuteurTouche = 0;
    _minuteurInvincibilite = 0;
    _minuteurTraversee = 0;
    _recuperationAttaque = 0;
    traverseLesPlateformes = false;
    _etirement = 0;
    _ecrasement = 0;
    _appliquerAlpha(255);
    changerEtat(EtatJoueur.immobile);

    // Chapitre 37 : game.perdreUneVie();
  }

  void _verifierChuteMortelle() {
    if (position.y > Constantes.limiteChute) {
      mourir();
    }
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);
    if (other is Plateforme) return;
    // Chapitre 37 : if (other is Ennemi) subirDegats(other.degatsContact);
    // Chapitre 38 : if (other is Collectible) other.ramasser(this);
  }

  // MACHINE À ÉTATS

  void _mettreAJourEtat() => changerEtat(_calculerEtat());

  EtatJoueur _calculerEtat() {
    if (pv <= 0) return EtatJoueur.mort;
    if (_minuteurAttaque > 0) return EtatJoueur.attaque;
    if (_minuteurTouche > 0) return EtatJoueur.touche;
    if (!auSol) {
      return velocite.y < 0 ? EtatJoueur.saut : EtatJoueur.chute;
    }
    if (velocite.x.abs() > Constantes.seuilMarche) return EtatJoueur.marche;
    return EtatJoueur.immobile;
  }

  void changerEtat(EtatJoueur nouvel) {
    if (etat == nouvel) return;

    final ancien = etat;
    etat = nouvel;
    _tempsDansEtat = 0;

    _surSortieEtat(ancien);
    _surEntreeEtat(nouvel);
  }

  void _surSortieEtat(EtatJoueur e) {
    if (e == EtatJoueur.attaque) {
      _zoneAttaque?.removeFromParent();
      _zoneAttaque = null;
    }
  }

  void _surEntreeEtat(EtatJoueur e) {
    switch (e) {
      case EtatJoueur.saut:
        _etirement = 1.0;
        break;
      case EtatJoueur.mort:
        velocite.setZero();
        break;
      case EtatJoueur.immobile:
      case EtatJoueur.marche:
      case EtatJoueur.chute:
      case EtatJoueur.attaque:
      case EtatJoueur.touche:
        break;
    }
    _appliquerApparence();
  }

  // APPARENCE

  Color _couleurEtat() {
    switch (etat) {
      case EtatJoueur.immobile:
        return Palette.joueur;
      case EtatJoueur.marche:
        return Palette.joueurMarche;
      case EtatJoueur.saut:
        return Palette.joueurSaut;
      case EtatJoueur.chute:
        return Palette.joueurChute;
      case EtatJoueur.attaque:
        return Palette.joueurAttaque;
      case EtatJoueur.touche:
        return Palette.danger;
      case EtatJoueur.mort:
        return Palette.joueurMort;
    }
  }

  void _appliquerApparence() {
    _corps.paint.color = _couleurEtat();
  }

  void _mettreAJourApparence(double dt) {
    _tempsDansEtat += dt;

    _etirement = _approcher(_etirement, 0, dt / 0.15);
    _ecrasement = _approcher(_ecrasement, 0, dt / 0.15);

    var echelleX = 1.0 - 0.22 * _etirement + 0.30 * _ecrasement;
    var echelleY = 1.0 + 0.28 * _etirement - 0.26 * _ecrasement;
    var decalageY = 0.0;

    switch (etat) {
      case EtatJoueur.immobile:
        decalageY = 0.6 * sin(_tempsDansEtat * 3.0);
        break;
      case EtatJoueur.marche:
        final cadence = velocite.x.abs() / Constantes.vitesseJoueur;
        decalageY = -1.6 * sin(_tempsDansEtat * 16.0 * cadence).abs();
        break;
      case EtatJoueur.chute:
        echelleY += 0.06;
        break;
      case EtatJoueur.attaque:
        echelleX += 0.12;
        break;
      case EtatJoueur.mort:
        echelleX = 1.35;
        echelleY = 0.35;
        decalageY = size.y * 0.32;
        break;
      case EtatJoueur.saut:
      case EtatJoueur.touche:
        break;
    }

    _visuel
      ..scale.setValues(echelleX * direction, echelleY)
      ..position.setValues(size.x / 2, size.y / 2 + decalageY);

    _mettreAJourClignotement();
  }

  void _mettreAJourClignotement() {
    if (!invincible) return;
    final phase =
        (_minuteurInvincibilite / Constantes.periodeClignotement) % 1.0;
    _appliquerAlpha(phase < 0.5 ? 90 : 255);
  }

  void _appliquerAlpha(int alpha) {
    _corps.paint.color = _corps.paint.color.withAlpha(alpha);
    _oeil.paint.color = _oeil.paint.color.withAlpha(alpha);
    _echarpe.paint.color = _echarpe.paint.color.withAlpha(alpha);
  }
}

/// Hitbox temporaire créée devant le joueur pendant une attaque.
class ZoneAttaque extends PositionComponent with CollisionCallbacks {
  ZoneAttaque({
    required Vector2 position,
    required Vector2 size,
    required this.degats,
  }) : super(
          position: position,
          size: size,
          anchor: Anchor.topLeft,
          priority: 12,
        );

  final double degats;

  @override
  Future<void> onLoad() async {
    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = Palette.accent.withAlpha(120),
      ),
    );
    await add(RectangleHitbox(collisionType: CollisionType.active));
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    // Chapitre 37 : if (other is Ennemi) other.subirDegats(degats);
  }
}
```

### `lib/donjon_game.dart`

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

import 'composants/joueur.dart';
import 'composants/plateforme.dart';
import 'config/constantes.dart';
import 'config/palette.dart';
import 'core/game_state.dart';

class Overlays {
  static const String menuPrincipal = 'menu_principal';
  static const String hud = 'hud';
  static const String pause = 'pause';
  static const String gameOver = 'game_over';
  static const String victoire = 'victoire';
  static const String chargement = 'chargement';
}

class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  /// Alias lisible du World fourni par FlameGame.
  World get monde => world;

  GameState etat = GameState.chargement;

  int score = 0;
  int vies = Constantes.viesDepart;
  int niveauCourant = 0;
  int meilleurScore = 0;

  late Joueur joueur;

  static final Vector2 positionDepartJoueur = Vector2(
    Constantes.tailleTuile * 3,
    Constantes.tailleTuile * 12,
  );

  @override
  Color backgroundColor() => Palette.fond;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder
      ..zoom = Constantes.zoomCamera
      ..anchor = Anchor.center;
    changerEtat(GameState.menu);
  }

  void changerEtat(GameState nouvelEtat) {
    if (etat == nouvelEtat) return;
    etat = nouvelEtat;
    overlays.clear();

    switch (nouvelEtat) {
      case GameState.chargement:
        overlays.add(Overlays.chargement);
        pauseEngine();
        break;
      case GameState.menu:
        overlays.add(Overlays.menuPrincipal);
        pauseEngine();
        break;
      case GameState.enJeu:
        resumeEngine();
        break;
      case GameState.pause:
        overlays.add(Overlays.pause);
        pauseEngine();
        break;
      case GameState.gameOver:
        overlays.add(Overlays.gameOver);
        pauseEngine();
        break;
      case GameState.victoire:
        overlays.add(Overlays.victoire);
        pauseEngine();
        break;
    }
  }

  Future<void> demarrerPartie() async {
    // IMPORTANT : on relance le moteur AVANT tout `await add(...)`,
    // sinon la file de montage des composants ne serait jamais traitée.
    changerEtat(GameState.enJeu);

    monde.removeAll(monde.children.toList());

    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = 0;

    await construireSalleDeTest();
    await ajouterJoueur();
  }

  /// Salle de test du chapitre 36, remplacée par un vrai niveau au chapitre 39.
  Future<void> construireSalleDeTest() async {
    // (colonne, ligne, largeur, hauteur, traversable)
    const blocs = <List<int>>[
      [0, 0, 40, 1, 0],
      [0, 0, 1, 18, 0],
      [39, 0, 1, 18, 0],
      [0, 13, 13, 1, 0],
      [30, 13, 10, 1, 0],
      [12, 14, 1, 4, 0],
      [19, 13, 1, 5, 0],
      [13, 13, 6, 1, 0],
      [5, 7, 4, 1, 0],
      [2, 10, 3, 1, 0],
      [12, 3, 4, 1, 0],
      [35, 6, 3, 1, 0],
      [1, 13, 4, 5, 0],
      [23, 5, 5, 1, 1],
      [26, 9, 5, 1, 1],
    ];

    for (final b in blocs) {
      await monde.add(
        Plateforme.tuiles(
          b[0],
          b[1],
          b[2],
          nombreDeLignes: b[3],
          traversable: b[4] == 1,
        ),
      );
    }
  }

  Future<void> ajouterJoueur() async {
    joueur = Joueur(position: positionDepartJoueur.clone());
    await monde.add(joueur);
    camera.follow(joueur.cibleCamera, maxSpeed: 400);
  }

  // Signatures réservées aux chapitres suivants.
  // void ajouterScore(int points);   // ch. 38
  // void perdreUneVie();             // ch. 37
  // Future<void> chargerNiveau(int index);  // ch. 39
  // void terminerNiveau();           // ch. 39
}
```

---

## 36.EX — Exercices

### Exercice 1 — Le mur invisible (facile)
Ajoutez à la salle de test une plateforme d'une tuile de large et de six tuiles de haut, en colonne 22, à partir de la ligne 7. Vérifiez que le joueur est bien bloqué des deux côtés et qu'il ne peut pas l'escalader.

### Exercice 2 — Un saut de hauteur choisie (facile)
Calculez la valeur de `forceSaut` qui permet d'atteindre exactement 5 tuiles (160 pixels) avec `gravite = 1200`. Appliquez-la et vérifiez en plaçant une plateforme à cette hauteur.

### Exercice 3 — Le sprint (facile)
Ajoutez une touche Maj (`LogicalKeyboardKey.shiftLeft`) qui multiplie la vitesse cible par 1,6 tant qu'elle est maintenue. La décélération ne doit pas changer.

### Exercice 4 — Compter les sauts (facile)
Ajoutez au `Joueur` un compteur `int nombreDeSauts` incrémenté à chaque saut effectif, et remis à zéro par `reinitialiser()`. Affichez-le dans la console à chaque atterrissage.

### Exercice 5 — L'état `glissade` (intermédiaire)
Ajoutez une huitième valeur `glissade` à `EtatJoueur`, activée quand le joueur est au sol, sans entrée horizontale, et que `velocite.x.abs() > 60`. Donnez-lui une couleur propre. Laissez le compilateur vous indiquer tous les `switch` à compléter.

### Exercice 6 — Le triple saut (intermédiaire)
Remplacez `bool doubleSautUtilise` par un compteur de sauts aériens et une constante `int sautsAeriensMax`. Vérifiez que la valeur `0` désactive complètement le saut en l'air et que `2` donne un triple saut.

### Exercice 7 — Le mur qui tue (intermédiaire)
Créez `PlateformePiegee extends Plateforme` avec un champ `degats`. Faites que le joueur subisse des dégâts en se posant dessus. Indice : `onCollisionStart` du joueur reçoit déjà l'objet ; il suffit de ne plus l'ignorer.

### Exercice 8 — Mettre les plateformes en cache (intermédiaire)
Remplacez le getter `plateformes` d'`Entite` par une liste `List<Plateforme>` maintenue par `DonjonGame`, remplie dans `construireSalleDeTest()`. Mesurez le gain avec 400 plateformes.

### Exercice 9 — La poussière d'atterrissage (avancé)
Dans `surAtterrissage()`, ajoutez au monde trois petits `RectangleComponent` de 3 x 3 pixels qui s'écartent du point de contact et disparaissent en 0,3 s. Utilisez `MoveByEffect` et `RemoveEffect` du chapitre 33.

### Exercice 10 — Le saut mural (avancé)
Détectez qu'un mur est collé au joueur (une sonde d'un pixel à gauche ou à droite de la boîte), affichez un état `accrocheMur` avec une chute ralentie, et permettez un saut qui repousse le joueur du mur.

---

## 36.CO — Corrections des exercices

### Correction 1

```dart
// Dans construireSalleDeTest(), ajouter à la liste `blocs` :
      [22, 7, 1, 6, 0],
```

**Explication :** `Plateforme.tuiles(22, 7, 1, nombreDeLignes: 6)` produit un bloc de 32 x 192 pixels en `(704, 224)`. La résolution horizontale de `_deplacerEnX` le traite comme n'importe quel obstacle : venant de la gauche (`dx > 0`), le joueur est recollé à `p.boite.left - size.x` ; venant de la droite, à `p.boite.right`. L'escalade est impossible parce que la correction verticale n'est jamais tentée dans la même étape : c'est tout l'intérêt de la résolution axe par axe (36.18).

### Correction 2

```dart
// hauteur = v² / (2 g)  =>  v = racine(2 g hauteur)
// v = racine(2 x 1200 x 160) = racine(384000) = 619,7
static const double forceSaut = -620.0;
```

**Explication :** on inverse la formule de la section 36.19.2. Le signe est négatif car l'axe Y croît vers le bas. Attention : avec `multiplicateurMontee = 0.88` appliqué tant que la touche est maintenue, la hauteur réelle sera légèrement supérieure ; la formule donne la borne basse.

### Correction 3

```dart
  bool _sprint = false;

  // dans onKeyEvent, avec les autres états continus :
  _sprint = keysPressed.contains(LogicalKeyboardKey.shiftLeft) ||
      keysPressed.contains(LogicalKeyboardKey.shiftRight);

  // dans _mettreAJourHorizontal :
  final facteur = _sprint ? 1.6 : 1.0;
  final cible = Constantes.vitesseJoueur * facteur * _entreeHorizontale;
```

**Explication :** le sprint modifie la **cible**, pas l'accélération ni la décélération. Le personnage met donc un peu plus longtemps à atteindre 288 px/s qu'à atteindre 180 px/s, ce qui est cohérent : accélérer davantage prend plus de temps. Relâcher Maj ramène la cible à 180 et le joueur décélère naturellement, sans à-coup.

### Correction 4

```dart
  int nombreDeSauts = 0;

  void _executerSaut(double force) {
    velocite.y = force;
    auSol = false;
    _etirement = 1.0;
    nombreDeSauts++;
  }

  @override
  void surAtterrissage() {
    _ecrasement = 1.0;
    _doubleSautUtilise = false;
    debugPrint('Sauts effectues : $nombreDeSauts');
  }

  // dans reinitialiser() :
  nombreDeSauts = 0;
```

**Explication :** l'incrémentation est placée dans `_executerSaut` et non dans `sauter()`, parce que `sauter()` enregistre seulement une **intention** (36.19.3) : un appui sans saut possible ne doit pas être compté. `debugPrint` est préférable à `print` dans une application Flutter : il régule le débit et disparaît en mode release.

### Correction 5

```dart
enum EtatJoueur {
  immobile, marche, saut, chute, attaque, touche, mort, glissade,
}

  EtatJoueur _calculerEtat() {
    if (pv <= 0) return EtatJoueur.mort;
    if (_minuteurAttaque > 0) return EtatJoueur.attaque;
    if (_minuteurTouche > 0) return EtatJoueur.touche;
    if (!auSol) return velocite.y < 0 ? EtatJoueur.saut : EtatJoueur.chute;
    if (_entreeHorizontale == 0 && velocite.x.abs() > 60) {
      return EtatJoueur.glissade;
    }
    if (velocite.x.abs() > Constantes.seuilMarche) return EtatJoueur.marche;
    return EtatJoueur.immobile;
  }

  // dans _couleurEtat() :
      case EtatJoueur.glissade:
        return const Color(0xFF80DEEA);
  // dans _surEntreeEtat() et _mettreAJourApparence() :
      case EtatJoueur.glissade:
        break;
```

**Explication :** le test de glissade est placé **avant** celui de la marche, sinon il ne serait jamais atteint (60 est supérieur à `seuilMarche`, qui vaut 8). C'est l'application directe du tableau de priorité de 36.25.3. En ajoutant la valeur à l'enum, le compilateur signale immédiatement les trois `switch` incomplets : c'est exactement le bénéfice attendu de l'exhaustivité (chapitre 11).

### Correction 6

```dart
  // Constantes
  static const int sautsAeriensMax = 2;

  // Joueur
  int _sautsAeriens = 0;

  void _tenterLeSaut() {
    if (_bufferSaut <= 0) return;

    if (auSol || _coyote > 0) {
      _executerSaut(Constantes.forceSaut);
      _bufferSaut = 0;
      _coyote = 0;
      _sautsAeriens = 0;
      return;
    }

    if (_sautsAeriens < Constantes.sautsAeriensMax) {
      _sautsAeriens++;
      _executerSaut(Constantes.forceDoubleSaut);
      _bufferSaut = 0;
    }
  }

  @override
  void surAtterrissage() {
    _ecrasement = 1.0;
    _sautsAeriens = 0;
  }
```

**Explication :** un compteur généralise proprement le booléen. `sautsAeriensMax = 0` rend la seconde condition toujours fausse et supprime le saut aérien sans autre modification ; `2` donne trois sauts au total. Le compteur est remis à zéro à **deux** endroits : à l'atterrissage et lors d'un saut depuis le sol — ce second cas couvre le saut effectué grâce au coyote time.

### Correction 7

```dart
class PlateformePiegee extends Plateforme {
  PlateformePiegee({
    required super.position,
    required super.size,
    this.degats = 15.0,
  });

  final double degats;
}

  // Dans Joueur.onCollisionStart :
  if (other is PlateformePiegee) {
    subirDegats(other.degats);
    return;
  }
  if (other is Plateforme) return;
```

**Explication :** l'ordre des tests est capital. `PlateformePiegee` **est** une `Plateforme` : si le test `other is Plateforme` venait en premier, il intercepterait aussi les plateformes piégées et la méthode sortirait sans rien faire. En programmation orientée objet, on teste toujours du type le plus spécifique vers le plus général (chapitre 10). L'invincibilité de `subirDegats` empêche par ailleurs le joueur de perdre toute sa vie en restant posé.

### Correction 8

```dart
// DonjonGame
final List<Plateforme> listePlateformes = [];

  Future<void> construireSalleDeTest() async {
    listePlateformes.clear();
    for (final b in blocs) {
      final p = Plateforme.tuiles(/* ... */);
      listePlateformes.add(p);
      await monde.add(p);
    }
  }

// Entite
  Iterable<Plateforme> get plateformes => game.listePlateformes;
```

**Explication :** `children.query<T>()` filtre l'ensemble des enfants du monde à chaque appel. Avec deux appels par entité et par frame, dix entités et 400 plateformes, cela représente environ 480 000 tests de type par seconde. La liste pré-remplie supprime entièrement ce coût. Attention : elle doit être vidée dans `demarrerPartie()` et tenue à jour si une plateforme est détruite en cours de partie, sans quoi des collisions fantômes apparaîtraient.

### Correction 9

```dart
  @override
  void surAtterrissage() {
    _ecrasement = 1.0;
    _doubleSautUtilise = false;

    for (var i = -1; i <= 1; i++) {
      final grain = RectangleComponent(
        position: Vector2(centre.x, bas - 2),
        size: Vector2.all(3),
        anchor: Anchor.center,
        paint: Paint()..color = Palette.plateformeHaut,
      );
      grain.add(MoveByEffect(Vector2(i * 14.0, -8),
          EffectController(duration: 0.3, curve: Curves.easeOut)));
      grain.add(RemoveEffect(delay: 0.3));
      game.monde.add(grain);
    }
  }
```

**Explication :** les grains sont ajoutés au **monde** et non au joueur : ils doivent rester au sol pendant que le personnage repart. `MoveByEffect` (chapitre 33) est un déplacement **relatif**, donc indépendant de la position d'apparition. `RemoveEffect(delay: 0.3)` retire le composant à la fin, sans quoi le monde se remplirait de grains invisibles à chaque atterrissage — une fuite de mémoire classique.

### Correction 10

```dart
  bool _murGauche = false;
  bool _murDroite = false;

  void _detecterMurs() {
    final sondeG = Rect.fromLTWH(gauche - 2, haut + 4, 2, size.y - 8);
    final sondeD = Rect.fromLTWH(droite, haut + 4, 2, size.y - 8);
    _murGauche = false;
    _murDroite = false;
    for (final p in plateformes) {
      if (p.traversable) continue;
      if (sondeG.overlaps(p.boite)) _murGauche = true;
      if (sondeD.overlaps(p.boite)) _murDroite = true;
    }
  }

  // dans update, apres deplacerAvecCollisions :
  _detecterMurs();
  final accroche = !auSol && velocite.y > 0 &&
      ((_murGauche && _entreeHorizontale < 0) ||
       (_murDroite && _entreeHorizontale > 0));
  if (accroche && velocite.y > 60) velocite.y = 60;   // glissade lente

  // dans _tenterLeSaut, avant le double saut :
  if (_murGauche || _murDroite) {
    final pousse = _murGauche ? 1 : -1;
    velocite
      ..x = pousse * Constantes.vitesseJoueur * 1.2
      ..y = Constantes.forceSaut * 0.9;
    direction = pousse;
    _bufferSaut = 0;
    return;
  }
```

**Explication :** les sondes sont deux rectangles de 2 pixels de large placés juste à l'extérieur de la boîte, raccourcis de 4 pixels en haut et en bas pour ne pas s'accrocher aux coins du sol ni du plafond. L'accroche exige que le joueur **pousse** vers le mur, sinon il resterait collé involontairement. Le saut mural inverse la direction : il repousse le joueur du mur, faute de quoi il se recollerait immédiatement et grimperait à la verticale.

---

## Et maintenant ?

Vous avez un personnage. Il court, il saute, il se retourne, il frappe, il encaisse et il meurt. Le jeu est jouable, mais il est vide : il n'y a rien à combattre.

Le chapitre suivant peuple le donjon. Vous y écrirez le mixin `Sante`, la classe abstraite `Ennemi`, puis deux ennemis concrets — le `Gobelin` qui patrouille et poursuit, la `Chauvesouris` qui vole en sinusoïde — ainsi que le combat complet : la zone d'attaque que vous venez d'écrire fera enfin des dégâts, et les gobelins vous en rendront.

[37-PARTIE-2C—ENNEMIS-IA-ET-COMBAT.md](./37-PARTIE-2C—ENNEMIS-IA-ET-COMBAT.md)
