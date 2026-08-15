# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 32 — COLLISIONS ET COLLISIONCALLBACKS

> **Niveau :** intermédiaire
> **Durée estimée :** 9 h
> **Pré-requis :** chapitres 27 à 31 (Flame, composants, sprites, entrées, caméra), chapitre 24 (collisions écrites à la main, AABB, séparation des axes, tunneling), chapitres 10 et 11 (héritage, polymorphisme, `is`, mixins)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** faire réagir les composants du « Donjon de Dart » les uns aux autres — murs infranchissables, pièces ramassées, gobelins qui blessent — en laissant Flame détecter les contacts et en écrivant vous-même la réaction.

---

## 32.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- énoncer précisément ce que Flame automatise et ce qu'il laisse à votre charge par rapport au chapitre 24 ;
- activer la détection de collision avec le mixin `HasCollisionDetection` ;
- ajouter le mixin `CollisionCallbacks` à un composant et comprendre ce qu'il apporte ;
- choisir entre `RectangleHitbox`, `CircleHitbox` et `PolygonHitbox` ;
- ajouter une hitbox à un composant, avec ou sans arguments ;
- utiliser `RectangleHitbox.relative()` pour une hitbox proportionnelle au parent ;
- décaler et redimensionner une hitbox indépendamment du visuel ;
- visualiser toutes les hitboxes grâce à `debugMode` ;
- distinguer `CollisionType.active`, `CollisionType.passive` et `CollisionType.inactive` ;
- attribuer le bon type à chaque catégorie d'objet du jeu ;
- estimer le coût en performances d'une mauvaise répartition des types ;
- écrire `onCollisionStart`, `onCollision` et `onCollisionEnd` et savoir lequel utiliser ;
- exploiter le `Set<Vector2> intersectionPoints` ;
- identifier l'autre composant avec l'opérateur `is` ;
- expliquer le patron « double dispatch » et pourquoi on teste le type des deux côtés ;
- empêcher le héros de traverser un mur ;
- calculer une profondeur de pénétration et repousser le héros ;
- séparer les axes X et Y pour éviter l'accrochage aux angles ;
- utiliser les hitboxes solides (`isSolid`) ;
- entourer la zone de jeu avec `ScreenHitbox` ;
- lancer des rayons avec `raycast`, `raycastAll` et `raytrace` ;
- construire une ligne de vue d'ennemi à l'aide d'un raycast ;
- décrire le broadphase de Flame et le remplacer par `QuadTreeCollisionDetection` ;
- décider quand le quadtree devient rentable ;
- écrire un capteur (trigger) pour le ramassage d'objets ;
- gérer les dégâts et l'invincibilité temporaire ;
- traiter le cas des projectiles rapides et du tunneling ;
- décider quand basculer sur Forge2D plutôt que sur la détection intégrée ;
- assembler un niveau jouable du « Donjon de Dart » avec murs, pièces et gobelins.

---

## 32.1 — Ce que Flame automatise par rapport au chapitre 24

Au chapitre 24 vous avez tout écrit à la main : la classe `Aabb`, la fonction `seChevauchent`, la boucle « tout le monde contre tout le monde », la grille spatiale, les couches et les masques, la résolution en deux passes, le mode debug. Ce travail n'est pas perdu. Il devient votre grille de lecture du moteur.

Voici la correspondance exacte, ligne par ligne.

| Chapitre 24 (écrit à la main) | Chapitre 32 (Flame 1.38.0) | Qui fait le travail |
| --- | --- | --- |
| classe `Aabb` avec `x`, `y`, `largeur`, `hauteur` | `RectangleHitbox` | Flame |
| classe `Cercle` avec `cx`, `cy`, `rayon` | `CircleHitbox` | Flame |
| polygone convexe écrit à la main | `PolygonHitbox` | Flame |
| fonction `seChevauchent(a, b)` | test interne du système de collision | Flame |
| double boucle `for i / for j` sur toutes les entités | broadphase *sweep and prune* | Flame |
| grille spatiale de la 24.23 | `QuadTreeCollisionDetection` (optionnel) | Flame |
| `enum Couche` + masques binaires | `CollisionType.active` / `passive` / `inactive` | Flame |
| appel manuel de `reagir(a, b)` après le test | `onCollisionStart`, `onCollision`, `onCollisionEnd` | Flame appelle, **vous** écrivez le corps |
| liste des points de contact calculée à la main | paramètre `Set<Vector2> intersectionPoints` | Flame |
| dessin des rectangles rouges du mode debug | `debugMode = true` | Flame |
| séparation hitbox / sprite | hitbox = **composant enfant** du composant visuel | Flame |
| bords de l'écran testés à la main | `ScreenHitbox` | Flame |
| **résolution** : repousser, annuler la vitesse | **rien** | **vous** |
| **anti-tunneling** : swept AABB, sous-échantillonnage | `raycast` (outil), pas de solution automatique | **vous** |
| logique de jeu : dégâts, score, mort | **rien** | **vous** |

Trois lignes de ce tableau méritent d'être encadrées.

> **Flame détecte. Flame ne résout pas.**
>
> Le moteur vous prévient qu'un contact a lieu et vous donne les points d'intersection. Il ne déplace personne. Repousser le héros hors du mur reste **votre** code, et ce sera exactement l'algorithme de la section 24.14.

C'est le point qui surprend le plus les débutants venus d'Unity ou de Godot, où un `CharacterBody2D` glisse tout seul le long des murs. Dans Flame, la détection est un service ; la physique est votre affaire. C'est aussi ce qui rend Flame léger et prévisible.

Le deuxième point encadré :

> **Une hitbox est un composant.**
>
> Elle n'est pas un champ, pas une propriété, pas une configuration. C'est un `PositionComponent` que l'on ajoute en enfant, exactement comme au chapitre 28. Elle a donc une `position`, une `size`, un `angle`, un `anchor`, et elle suit son parent automatiquement.

Et le troisième :

> **Sans hitbox, aucune collision.**
>
> Un composant qui porte le mixin `CollisionCallbacks` mais aucune hitbox ne recevra jamais le moindre appel. C'est l'erreur numéro un du chapitre.

---

## 32.2 — Activer la détection : `HasCollisionDetection` sur le jeu

Rien ne se passe tant que le jeu ne porte pas le mixin `HasCollisionDetection`. Ce mixin ajoute au jeu un objet `collisionDetection` et, à chaque tick, exécute la phase de détection après la mise à jour des composants.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/game.dart';

class DonjonDeDart extends FlameGame with HasCollisionDetection {
  // ...
}
```

Voici, en pseudo-code, ce que le mixin ajoute réellement (c'est le code source de Flame, simplifié) :

```dart
mixin HasCollisionDetection on Component {
  CollisionDetection get collisionDetection => _collisionDetection;

  @override
  void update(double dt) {
    super.update(dt);          // met à jour tout l'arbre de composants
    collisionDetection.run();  // PUIS détecte les collisions
  }
}
```

Deux conséquences importantes se lisent directement dans ces cinq lignes.

**La détection a lieu après tous les `update`.** Quand `onCollisionStart` est appelé, tous les composants ont déjà bougé pour cette frame. Vous ne pouvez donc pas « annuler » un déplacement avant qu'il ait lieu ; vous ne pouvez que le corriger après coup. Toute la section 32.19 découle de ce fait.

**La détection a lieu une fois par tick.** Un projectile qui parcourt 400 pixels en une frame n'est testé qu'aux deux extrémités de son saut. C'est le tunneling du chapitre 24.18, et Flame ne le corrige pas tout seul (section 32.29).

Le mixin peut aussi se poser sur un `World` plutôt que sur le jeu :

```dart
class MondeDuDonjon extends World with HasCollisionDetection {}

class DonjonDeDart extends FlameGame<MondeDuDonjon> {
  DonjonDeDart() : super(world: MondeDuDonjon());
}
```

**Règle de rattachement (importante).** Une hitbox est prise en charge par **l'ancêtre le plus proche** qui porte `HasCollisionDetection`. Si vous mettez le mixin sur le monde, les hitboxes placées dans le HUD (dans le `viewport`) ne seront pas détectées, puisque le viewport n'est pas un descendant du monde. C'est en général exactement ce que l'on veut : le HUD n'a rien à faire dans la physique du niveau.

```text
  QUI DÉTECTE QUOI ?

  FlameGame  with HasCollisionDetection
    ├── CameraComponent
    │     ├── Viewport
    │     │     └── BoutonAttaque  (hitbox -> détectée par le JEU)
    │     └── Viewfinder
    └── World
          ├── Heros    (hitbox -> détectée par le JEU)
          ├── Mur      (hitbox -> détectée par le JEU)
          └── Gobelin  (hitbox -> détectée par le JEU)

```

Dans ce chapitre, nous mettons le mixin sur le jeu : c'est le cas le plus courant et le plus simple à déboguer.

---

## 32.3 — Le mixin `CollisionCallbacks` sur le composant

Activer la détection ne suffit pas à être prévenu. Le composant qui veut **réagir** doit porter le mixin `CollisionCallbacks` (revoyez les mixins au chapitre 11 si le mot-clé `with` vous surprend encore).

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';

class Heros extends PositionComponent with CollisionCallbacks {
  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);
    // votre réaction
  }
}
```

Le mixin apporte trois méthodes redéfinissables et quatre membres utiles.

| Membre | Type | Rôle |
| --- | --- | --- |
| `onCollisionStart(points, other)` | `void` | appelé **au premier tick** du contact |
| `onCollision(points, other)` | `void` | appelé **à chaque tick** pendant le contact |
| `onCollisionEnd(other)` | `void` | appelé **une fois**, quand le contact cesse |
| `activeCollisions` | `Set<PositionComponent>` | tous les composants actuellement en contact |
| `isColliding` | `bool` | `true` si `activeCollisions` n'est pas vide |
| `collidingWith(other)` | `bool` | `true` si ce composant touche précisément `other` |
| `onCollisionStartCallback` etc. | champs de fonction | version « fonction » des callbacks, sans héritage |

Les trois callbacks sont annotées `@mustCallSuper` dans le code source de Flame. Concrètement :

```dart
@override
void onCollisionStart(Set<Vector2> points, PositionComponent other) {
  super.onCollisionStart(points, other); // NE JAMAIS OUBLIER
  // ...
}
```

**Pourquoi ?** Parce que c'est l'implémentation de `super` qui remplit `activeCollisions`. Si vous l'oubliez, `isColliding` restera à `false` pour toujours et `onCollisionEnd` ne fera plus le ménage. Le compilateur ne vous arrêtera pas ; il émettra un simple avertissement de l'analyseur. Prenez l'habitude d'écrire la ligne `super` **avant** d'écrire votre logique.

Il existe une variante sans héritage : affecter directement les champs `onCollisionStartCallback`, `onCollisionCallback` et `onCollisionEndCallback`. Attention, ces champs appartiennent au mixin `CollisionCallbacks` : le composant doit tout de même porter ce mixin, ce que `CircleComponent` ne fait pas par défaut. Dans le doute, écrivez une sous-classe : c'est plus long de trois lignes et infiniment plus lisible.

---

## 32.4 — Les hitboxes : `RectangleHitbox`, `CircleHitbox`, `PolygonHitbox`

Flame fournit trois formes de hitbox. Toutes héritent de `ShapeComponent`, donc de `PositionComponent` : ce sont des composants à part entière.

```text
  Component
    └── PositionComponent
          └── ShapeComponent
                ├── PolygonComponent ── RectangleComponent
                └── CircleComponent

  + mixin ShapeHitbox

  => RectangleHitbox   (rectangle, éventuellement pivoté)
  => CircleHitbox      (cercle)
  => PolygonHitbox     (polygone CONVEXE)
```

Les signatures exactes en 1.38.0 :

```dart
RectangleHitbox({
  Vector2? position,
  Vector2? size,
  double? angle,
  Anchor? anchor,
  int? priority,
  bool isSolid = false,
  CollisionType collisionType = CollisionType.active,
});

CircleHitbox({
  double? radius,
  Vector2? position,
  double? angle,
  Anchor? anchor,
  bool isSolid = false,
  CollisionType collisionType = CollisionType.active,
});

PolygonHitbox(
  List<Vector2> vertices, {
  Vector2? position,
  double? angle,
  Anchor? anchor,
  bool isSolid = false,
  CollisionType collisionType = CollisionType.active,
});
```

Comment choisir ? Le tableau suivant résume la pratique.

| Forme | Coût du test | Utiliser pour | Éviter pour |
| --- | --- | --- | --- |
| `RectangleHitbox` | faible | murs, plateformes, coffres, portes, corps de personnage | balles rondes, ennemis circulaires |
| `CircleHitbox` | le plus faible | pièces, potions, boules de feu, zones d'explosion, ennemis ronds | murs, sols |
| `PolygonHitbox` | le plus élevé | pentes, formes en losange, vaisseaux | tout ce qu'un rectangle décrit déjà correctement |

Deux contraintes à connaître par cœur.

**Un `PolygonHitbox` doit être convexe.** La détection de Flame repose sur des tests d'intersection valables uniquement pour des polygones convexes. Un polygone en forme de L ou de U produira des résultats faux, sans message d'erreur. La solution est d'en faire **plusieurs** hitboxes, réunies si besoin dans un `CompositeHitbox`.

```text
  CONVEXE (accepté)              CONCAVE (interdit)

     *-------*                      *-------*
    /         \                     |       |
   *           *                    |   *---*
    \         /                     |   |
     *-------*                      *---*

  Test : tout segment reliant     Le segment reliant les deux
  deux points de la forme reste   pointes du L sort de la forme.
  entièrement dans la forme.
```

**`PolygonHitbox.fillParent()` lève `UnsupportedError`.** Un polygone ne sait pas se redimensionner tout seul aux dimensions du parent. Pour remplir le parent, utilisez `RectangleHitbox()` sans argument.

Le constructeur `PolygonHitbox.regular` est nouveau en 1.38.0. Il crée un polygone régulier à `sides` côtés, ce qui donne un hexagone ou un octogone en une ligne :

```dart
// Un cristal hexagonal de 14 pixels de rayon.
add(PolygonHitbox.regular(sides: 6, radius: 14));
```

---

## 32.5 — Ajouter une hitbox à un composant

Le moment correct pour ajouter une hitbox est `onLoad()`, exactement comme au chapitre 28 pour n'importe quel enfant.

```dart
class Coffre extends PositionComponent with CollisionCallbacks {
  Coffre({super.position}) : super(size: Vector2(28, 22));

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox()); // remplit tout le composant
  }
}
```

**Le cas le plus courant : la hitbox sans argument.** `RectangleHitbox()` appelé sans `position` ni `size` active un drapeau interne `shouldFillParent`. La hitbox prend alors la taille de son parent, et se resynchronise automatiquement si la taille du parent change. C'est vérifiable dans le code source :

```dart
RectangleHitbox({ ... }) : shouldFillParent = size == null && position == null;
```

Attention à la conséquence : dès que vous donnez **soit** `size`, **soit** `position`, le remplissage automatique est désactivé et vous devez fournir les deux valeurs cohérentes vous-même.

`CircleHitbox()` sans argument remplit également le parent : le rayon vaut la moitié de la plus petite dimension.

Voici un premier programme complet. Il n'a besoin d'aucune image : tout est dessiné avec des rectangles.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    MaterialApp(
      home: Scaffold(
        body: GameWidget<DonjonDeDart>(game: DonjonDeDart()),
      ),
    ),
  );
}

class DonjonDeDart extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(Mur(position: Vector2(180, 60), size: Vector2(24, 200)));
    await world.add(Heros(position: Vector2(40, 140)));
  }
}

class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(32, 40));

  static final Paint _peinture = Paint()..color = const Color(0xFF4CAF50);
  double vitesse = 60; // pixels par seconde

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void update(double dt) {
    super.update(dt);
    position.x += vitesse * dt;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    debugPrint('DÉBUT de contact avec ${other.runtimeType}');
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    super.onCollisionEnd(other);
    debugPrint('FIN de contact avec ${other.runtimeType}');
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFF6D4C41);

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Résultat :**

```text
DÉBUT de contact avec Mur
FIN de contact avec Mur
```

Le héros traverse le mur de part en part : Flame a bien détecté l'entrée et la sortie, mais n'a rien empêché. Nous corrigerons cela en 32.18.

---

## 32.6 — `RectangleHitbox.relative()`

Une hitbox fixée en dur à `Vector2(24, 40)` casse dès que vous changez la taille du composant. Le constructeur `relative` exprime la hitbox **en proportion** du parent.

```dart
RectangleHitbox.relative(
  Vector2 relation, {
  required Vector2 parentSize,
  Vector2? position,
  double? angle,
  Anchor? anchor,
  bool isSolid = false,
  CollisionType collisionType = CollisionType.active,
});
```

`relation` contient deux facteurs entre 0 et 1 : la fraction de largeur et la fraction de hauteur.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(48, 64));

  @override
  Future<void> onLoad() async {
    add(
      RectangleHitbox.relative(
        Vector2(0.5, 0.8),   // 50 % de la largeur, 80 % de la hauteur
        parentSize: size,
        anchor: Anchor.center,
        position: size / 2,  // centrée dans le parent
      ),
    );
  }
}
```

```text
  size du composant : 48 x 64
  relation          : (0.5, 0.8)

  +----------------------------+  <- composant 48 x 64
  |                            |
  |        +----------+        |
  |        |          |        |
  |        |  hitbox  |        |  <- 24 x 51.2
  |        |          |        |
  |        +----------+        |
  |                            |
  +----------------------------+
```

Le calcul : `48 * 0.5 = 24` et `64 * 0.8 = 51.2`.

**Le piège de `parentSize`.** Ce paramètre est obligatoire, et il n'est pas magique : la valeur est lue **une seule fois**, au moment de la construction. Si vous écrivez `parentSize: size` dans `onLoad()`, la `size` du parent doit déjà être correcte à cet instant. Elle l'est si vous l'avez passée au constructeur ; elle ne l'est pas si vous comptez sur `autoResize` d'un `SpriteComponent` dont l'image n'est pas encore chargée. Dans ce dernier cas, chargez le sprite d'abord :

```dart
@override
Future<void> onLoad() async {
  sprite = await Sprite.load('heros.png'); // size est maintenant connue
  add(RectangleHitbox.relative(Vector2(0.6, 0.9), parentSize: size));
}
```

Les constructeurs `relative` existent aussi pour les deux autres formes :

```dart
CircleHitbox.relative(double relation, {required Vector2 parentSize, ...});
PolygonHitbox.relative(List<Vector2> relation, {required Vector2 parentSize, ...});
```

Pour `CircleHitbox.relative`, `relation` est un simple `double` : `0.5` donne un cercle inscrit dans le parent.

Pour `PolygonHitbox.relative`, chaque sommet est exprimé dans un repère où le centre du parent vaut `(0, 0)`, le bord droit `x = 1` et le bord bas `y = 1` : la liste `[Vector2(0, -1), Vector2(1, 0), Vector2(0, 1), Vector2(-1, 0)]` décrit ainsi un losange inscrit dans le composant.

---

## 32.7 — Positionner et dimensionner une hitbox indépendamment du sprite

La section 24.3 avait posé la règle : **la hitbox n'est pas le sprite**. Une image de personnage contient de l'air, une cape qui flotte, un chapeau pointu. Si la hitbox épouse l'image, le joueur meurt parce que la pointe de son chapeau a frôlé un gobelin, et il trouve le jeu injuste.

Flame ne change rien à ce principe. Il le rend simplement facile à appliquer, puisque la hitbox est un composant enfant avec sa propre `position` et sa propre `size`.

```text
  SPRITE 48 x 64                    HITBOX 24 x 44 en (12, 18)

  +----------------------------+    +----------------------------+
  |          /\                |    |          /\                |
  |         /  \   chapeau     |    |         /  \                |
  |        +----+              |    |        +----+               |
  |        |o  o|   tête       |    |     +--+----+--+  <- y = 18 |
  |     +--+----+--+           |    |     |  |    |  |            |
  |     |  | tronc |  |  bras  |    |     |  |    |  |            |
  |     +--+----+--+           |    |     |  |    |  |            |
  |        |    |              |    |     +--+----+--+  <- y = 62 |
  |        |    |   jambes     |    |        |    |               |
  |       _|    |_             |    |       _|    |_              |
  +----------------------------+    +----------------------------+
                                          ^        ^
                                       x = 12   x = 36
```

Le code correspondant :

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(48, 64));

  @override
  Future<void> onLoad() async {
    add(
      RectangleHitbox(
        size: Vector2(24, 44),
        position: Vector2(12, 18),
      ),
    );
  }
}
```

**Le repère de la hitbox est celui de son parent.** `position: Vector2(12, 18)` signifie « 12 pixels à droite et 18 pixels sous le coin haut-gauche du héros », pas « 12 pixels depuis le bord de l'écran ». C'est exactement le comportement d'un enfant vu au chapitre 28 : la hitbox suit le héros sans que vous ayez une seule ligne à écrire dans `update`.

L'ancre par défaut d'une hitbox est `Anchor.topLeft`, comme pour tout `PositionComponent`. Si vous préférez raisonner en centre :

```dart
add(
  RectangleHitbox(
    size: Vector2(24, 44),
    position: Vector2(24, 40), // centre visé dans le parent
    anchor: Anchor.center,
  ),
);
```

Trois usages très courants découlent de cette liberté.

**Réduire la hitbox de contact.** Un ennemi dont la hurtbox (zone où il encaisse) est plus grande que la hitbox (zone où il blesse) donne un jeu plus agréable. C'est la distinction de la section 24.4, et elle s'écrit ici avec deux hitboxes enfants de tailles différentes.

**Décaler vers le bas.** Dans une vue de dessus, on ne fait souvent collisionner que les *pieds* du personnage, pour qu'il puisse passer « devant » un mur bas.

```dart
// Vue de dessus : seuls les pieds bloquent.
add(
  RectangleHitbox(
    size: Vector2(28, 14),
    position: Vector2(10, 50),
  ),
);
```

**Plusieurs hitboxes sur un même composant.** Rien ne l'interdit ; chacune est un enfant indépendant, et l'on peut mélanger les formes (un `RectangleHitbox` pour le tronc, un `CircleHitbox` pour la tête). Par défaut, deux hitboxes ajoutées au **même** parent ne se testent pas entre elles : le drapeau `allowSiblingCollision` vaut `false`. C'est ce que l'on veut ici.

---

## 32.8 — Visualiser les hitboxes avec `debugMode`

Une hitbox est invisible. Tant que vous ne la voyez pas, chaque bug de collision est une devinette. Flame règle la question en une ligne.

```dart
class DonjonDeDart extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    debugMode = true; // propagé à TOUT l'arbre de composants
  }
}
```

`debugMode` est déclaré sur `Component`. Placé sur le jeu, il descend dans tout l'arbre. Chaque `PositionComponent` dessine alors son rectangle englobant, ses coordonnées, et chaque hitbox dessine son contour.

```text
  AFFICHAGE EN debugMode

    (40.0, 140.0)
    +----------------+   <- rectangle du composant, en debugColor
    | +------------+ |
    | |            | |   <- contour de la hitbox, en jaune par défaut
    | |            | |
    | +------------+ |
    +----------------+
```

On peut aussi l'activer sur un seul composant, ce qui évite un écran illisible :

```dart
heros.debugMode = true;   // seulement le héros et ses enfants
```

Deux réglages complémentaires, tous deux issus du code source de `ShapeHitbox` :

```dart
final hitbox = RectangleHitbox()
  ..debugColor = const Color(0xFFFF0000)  // rouge au lieu du jaune
  ..renderShape = true;                   // dessine la forme même hors debugMode

add(hitbox);
```

`debugColor` vaut `const Color(0xFFFFFF00)` par défaut sur une hitbox. `renderShape` vaut `false` sur une hitbox (contrairement à un `RectangleComponent` ordinaire, où il vaut `true`) : c'est pour cela qu'une hitbox est invisible en temps normal.

**Conseil de méthode.** Créez un raccourci de bascule dès le début du projet : dans `onKeyEvent`, un test sur `LogicalKeyboardKey.f1` suivi de `debugMode = !debugMode`. Vous l'utiliserez cent fois. Le mini-projet de la section 32.31 en donne le code exact.

---

## 32.9 — `CollisionType.active`, `passive`, `inactive`

Chaque hitbox porte un `CollisionType`. Il détermine **contre qui** elle est testée. Voici l'énumération telle qu'elle figure dans le code source de Flame :

```dart
enum CollisionType {
  /// Entre en collision avec les hitboxes de type active OU passive.
  active,

  /// Entre en collision uniquement avec les hitboxes de type active.
  passive,

  /// N'entre en collision avec rien.
  inactive,
}
```

La valeur par défaut est `CollisionType.active` pour les trois formes de hitbox.

La table de vérité tient en neuf cases. Apprenez-la : elle explique 90 % des collisions « qui ne se déclenchent pas ».

```text
              |  active  | passive  | inactive
    ----------+----------+----------+----------
     active   |   OUI    |   OUI    |   non
    ----------+----------+----------+----------
     passive  |   OUI    |   non    |   non
    ----------+----------+----------+----------
     inactive |   non    |   non    |   non
```

Autrement dit : **il faut au moins un `active` dans le couple.** Deux hitboxes `passive` ne se voient jamais, même superposées.

Le type se fixe au constructeur ou après coup :

```dart
// Au constructeur
add(RectangleHitbox(collisionType: CollisionType.passive));

// Plus tard, sur une référence conservée
late final RectangleHitbox hitbox;

@override
Future<void> onLoad() async {
  hitbox = RectangleHitbox();
  add(hitbox);
}

void mourir() {
  hitbox.collisionType = CollisionType.inactive; // ne blesse plus personne
}
```

**`inactive` plutôt que la suppression.** Quand un gobelin meurt et joue son animation de disparition, il ne doit plus blesser le héros mais doit rester à l'écran une seconde. Passer sa hitbox en `inactive` est plus simple et bien moins coûteux que de la retirer puis de la recréer. Le commentaire du code source de Flame le dit explicitement à propos de `CollisionDetection.remove` : « si vous voulez seulement désactiver temporairement, mettez `collisionType = CollisionType.inactive` ».

---

## 32.10 — Choisir le bon type : le décor est passif, le joueur est actif

La règle pratique tient en une phrase :

> **Ce qui bouge est `active`. Ce qui ne bouge pas est `passive`. Ce qui est mort ou désactivé est `inactive`.**

Voici la répartition pour le « Donjon de Dart ».

| Entité | Bouge ? | Type de hitbox | Justification |
| --- | --- | --- | --- |
| Héros | oui | `active` | doit voir les murs, les pièces, les gobelins |
| Gobelin | oui | `active` | doit voir les murs et le héros |
| Flèche du héros | oui | `active` | doit voir les gobelins et les murs |
| Mur | non | `passive` | deux murs n'ont rien à se dire |
| Sol, plateforme | non | `passive` | idem |
| Pièce | non | `passive` | une pièce ne se ramasse pas elle-même |
| Potion, clé | non | `passive` | idem |
| Coffre fermé | non | `passive` | idem |
| Porte de sortie | non | `passive` | idem |
| Gobelin mort | non | `inactive` | ne doit plus rien déclencher |
| Bouton du HUD | non | pas de hitbox | on utilise `TapCallbacks` (chapitre 30) |

Regardons ce que cette répartition économise. Imaginons un niveau avec 1 héros, 8 gobelins, 12 flèches en vol, 300 tuiles de mur et 40 pièces, soit 361 hitboxes.

**Tout en `active` :** le moteur doit envisager tous les couples possibles, soit `361 * 360 / 2 = 64 980` couples. Le plus gros contingent est celui des couples mur-mur : `300 * 299 / 2 = 44 850` tests parfaitement inutiles, puisque deux murs ne bougent jamais.

**Avec les murs et les pièces en `passive` :** les couples mur-mur, mur-pièce et pièce-pièce disparaissent complètement de la liste. Il reste les 21 objets mobiles entre eux (`21 * 20 / 2 = 210` couples) et les 21 mobiles contre les 340 statiques (`7 140` couples). On passe de 64 980 à 7 350 couples envisagés, soit **près de neuf fois moins**.

Ces chiffres sont ceux du cas le pire (avant broadphase). Le broadphase de Flame réduit encore fortement le nombre de tests réels, mais il travaille sur la liste que vous lui donnez : moins vous lui donnez de candidats, mieux il se porte.

Un dernier cas à connaître : **deux objets qui doivent se voir doivent avoir au moins un `active`.** Si vous mettez les pièces en `passive` et que vous décidez plus tard qu'une pièce doit tomber (donc bouger), pensez à la repasser en `active`, sinon elle traversera le sol.

---

## 32.11 — L'impact sur les performances

Le coût de la détection se décompose en deux étages, exactement comme en 24.22.

```text
  ÉTAGE 1 — BROAD PHASE  (« qui pourrait toucher qui ? »)
     Algorithme : sweep and prune (balayage et élagage)
     Travaille sur les AABB, pas sur les formes réelles.
     Sort une liste de COUPLES CANDIDATS.
                 |
                 v
  ÉTAGE 2 — NARROW PHASE  (« est-ce qu'ils se touchent vraiment ? »)
     Test géométrique exact sur chaque couple candidat.
     Calcule les points d'intersection.
                 |
                 v
     Appels de onCollisionStart / onCollision / onCollisionEnd
```

Le *sweep and prune* trie les hitboxes selon un axe et n'examine que les voisines dont les intervalles se chevauchent sur cet axe. Sur un niveau où les objets sont bien répartis, il transforme un coût en `n²` en un coût proche de `n log n`.

Les leviers dont vous disposez, du plus rentable au moins rentable :

| Levier | Gain typique | Effort |
| --- | --- | --- |
| Passer le décor en `passive` | très fort | une ligne par classe |
| Passer les objets hors écran en `inactive` | fort | quelques lignes |
| Fusionner 40 tuiles de mur en 1 grande hitbox | fort | change la conception du niveau |
| Préférer `CircleHitbox` à `PolygonHitbox` | moyen | un mot |
| Une hitbox par entité, pas cinq | moyen | discipline |
| Basculer sur le quadtree | variable | voir 32.25 |

Le troisième levier est souvent oublié et pourtant le plus efficace sur une tilemap. Un couloir horizontal de 40 tuiles de 16 pixels peut être représenté par **une seule** `RectangleHitbox` de 640 pixels de large. Vous divisez le nombre de hitboxes par 40 sans rien changer au rendu, puisque le rendu reste tuile par tuile.

```text
  40 tuiles = 40 hitboxes            1 mur logique = 1 hitbox

  [][][][][][][][][][][][]  ...      +----------------------+
   ^  ^  ^  ^  ^  ^  ^  ^            |                      |
   40 objets dans la broadphase      +----------------------+
                                      1 seul objet
```

Enfin, une mesure vaut mieux qu'une intuition. Flame fournit un composant tout prêt pour afficher le nombre d'images par seconde : `camera.viewport.add(FpsTextComponent(position: Vector2(8, 8)))`. Si les FPS ne bougent pas quand vous doublez le nombre de hitboxes, la collision n'est pas votre problème. Optimisez ce qui coûte, pas ce que vous imaginez coûteux.

---

## 32.12 — `onCollisionStart`

`onCollisionStart` est appelée **une seule fois**, au premier tick où deux hitboxes se touchent.

```dart
@override
void onCollisionStart(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  super.onCollisionStart(intersectionPoints, other);
  // ...
}
```

C'est la callback à utiliser pour tout ce qui doit se produire **une fois par contact** :

- ramasser une pièce ;
- infliger des dégâts d'un coup ;
- jouer un son d'impact ;
- déclencher l'ouverture d'un coffre ;
- démarrer une animation.

Exemple sur le héros du donjon :

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  int pieces = 0;
  int vies = 3;

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Piece) {
      pieces++;
      other.removeFromParent();
    } else if (other is Gobelin) {
      vies--;
    }
  }
}
```

**Pourquoi `onCollisionStart` et pas `onCollision` ici ?** Parce que `onCollision` est appelée à chaque tick. À 60 images par seconde, un héros immobile sur un gobelin perdrait 60 vies par seconde. Le compteur passerait de 3 à -57 en une seconde de contact.

**Séquence d'appels.** Voici ce que Flame appelle pour un contact qui dure quatre ticks :

```text
  tick 1 : pas de contact                -> rien
  tick 2 : contact                       -> onCollisionStart(points, other)
                                            onCollision(points, other)
  tick 3 : contact                       -> onCollision(points, other)
  tick 4 : contact                       -> onCollision(points, other)
  tick 5 : plus de contact               -> onCollisionEnd(other)
```

Notez la ligne du tick 2 : au premier tick de contact, `onCollisionStart` **et** `onCollision` sont appelées, dans cet ordre. C'est visible dans le code source de `CollisionDetection.run()`, qui appelle `handleCollisionStart` uniquement si les deux hitboxes ne se touchaient pas déjà, puis appelle `handleCollision` dans tous les cas. Ne comptez donc pas sur `onCollision` pour « ne rien faire au premier tick ».

---

## 32.13 — `onCollision`

`onCollision` est appelée **à chaque tick** tant que le contact dure.

```dart
@override
void onCollision(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  super.onCollision(intersectionPoints, other);
  // ...
}
```

Elle sert à tout ce qui est **continu** ou qui doit être **recalculé** :

- repousser le héros hors d'un mur (la position change à chaque frame, la correction aussi) ;
- infliger des dégâts par seconde dans une flaque de lave ;
- appliquer un ralentissement dans la boue ;
- maintenir un état « au sol » pour autoriser le saut.

Le cas des dégâts continus mérite une démonstration, car c'est là que les débutants oublient `dt`. Écrire `pointsDeVie -= 20` dans `onCollision` retire 20 points **par tick**, soit 1200 points par seconde à 60 images par seconde. Il faut multiplier par `dt`. Le problème : `onCollision` ne reçoit pas `dt`. Il faut donc le mémoriser dans `update`.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  double pointsDeVie = 100;
  double dernierDt = 0;

  @override
  void update(double dt) {
    super.update(dt);
    dernierDt = dt;
  }

  @override
  void onCollision(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollision(intersectionPoints, other);
    if (other is Lave) {
      pointsDeVie -= 20 * dernierDt;
    }
  }
}
```

**Cela fonctionne-t-il vraiment ?** Oui, et l'ordre le garantit : le mixin `HasCollisionDetection` appelle `super.update(dt)` (donc tout l'arbre, donc notre `update`) **avant** `collisionDetection.run()`. Quand `onCollision` s'exécute, `dernierDt` contient bien le `dt` de la frame en cours.

Une alternative plus élégante consiste à ne rien faire dans `onCollision` et à lire `isColliding` dans `update` :

```dart
@override
void update(double dt) {
  super.update(dt);
  // activeCollisions est tenu à jour par le mixin (d'où le super obligatoire).
  if (activeCollisions.any((c) => c is Lave)) pointsDeVie -= 20 * dt;
}
```

Cette version a l'avantage d'avoir `dt` sous la main. Retenez les deux formes : la première est plus directe, la seconde plus sûre.

---

## 32.14 — `onCollisionEnd`

`onCollisionEnd` est appelée **une seule fois**, au premier tick où le contact a cessé. Remarquez sa signature : elle ne reçoit **pas** de points d'intersection, ce qui est logique puisqu'il n'y a plus d'intersection.

```dart
@override
void onCollisionEnd(PositionComponent other) {
  super.onCollisionEnd(other);
  // ...
}
```

Usages typiques :

- quitter le sol : `estAuSol = false` ;
- sortir d'une zone de ralentissement : rendre sa vitesse normale au héros ;
- sortir du champ de vision d'un ennemi : repasser en patrouille ;
- fermer une porte automatique.

Exemple complet d'une plateforme et d'un héros qui saute :

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  bool estAuSol = false;

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Plateforme) estAuSol = true;
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    super.onCollisionEnd(other);
    if (other is Plateforme) estAuSol = false;
  }

  void sauter() {
    if (!estAuSol) return;
    // velocite.y = -320;
    estAuSol = false;
  }
}
```

**Le bug des deux plateformes.** Ce code contient un défaut classique. Si le héros chevauche deux plateformes voisines et quitte la première, `onCollisionEnd` met `estAuSol` à `false` alors qu'il repose toujours sur la seconde. Le joueur perd son saut sans raison.

La correction consiste à compter, ou mieux, à interroger `activeCollisions` :

```dart
@override
void onCollisionEnd(PositionComponent other) {
  super.onCollisionEnd(other);
  if (other is Plateforme) {
    // super a déjà retiré `other` de activeCollisions.
    estAuSol = activeCollisions.any((c) => c is Plateforme);
  }
}
```

Cette ligne n'est correcte **que** parce que `super.onCollisionEnd(other)` a été appelé en premier : c'est lui qui retire `other` de l'ensemble. Voilà une raison très concrète de respecter `@mustCallSuper`.

**Un cas piégeux : la disparition.** Que se passe-t-il si l'autre composant est retiré de l'arbre pendant le contact ? Depuis les versions récentes de Flame, `ShapeHitbox.onRemove()` termine proprement toutes les collisions actives avant de se retirer du système : `onCollisionEnd` est donc bien appelée. Ne comptez cependant pas sur `other` pour être encore monté à ce moment-là — n'y touchez pas, contentez-vous de mettre à jour votre propre état.

---

## 32.15 — Les points d'intersection (`Set<Vector2> intersectionPoints`)

Le premier paramètre de `onCollisionStart` et `onCollision` est un `Set<Vector2>`. Il contient les points où les **contours** des deux hitboxes se croisent, exprimés dans le repère **absolu** du monde.

```text
  DEUX RECTANGLES QUI SE CHEVAUCHENT

        +-------------+
        |  héros      |
        |        +----X------------+     X = point d'intersection
        |        |    |            |
        +--------X----+    mur     |
                 |                 |
                 +-----------------+

  intersectionPoints contient ici 2 points.
```

Le nombre de points dépend de la géométrie :

| Situation | Nombre de points typique |
| --- | --- |
| Un coin qui entre dans un rectangle | 2 |
| Deux rectangles qui se croisent en croix | 4 |
| Un cercle qui entre dans un rectangle | 2 |
| Un cercle entièrement contenu dans un autre | 0 (voir 32.21) |
| Deux bords exactement alignés | 1 ou 2, instable |

**Ne comptez jamais sur le nombre de points.** Il varie d'une frame à l'autre pendant que les objets bougent. Un test comme `if (points.length == 2)` produit des bugs impossibles à reproduire.

Ce que l'on fait raisonnablement avec ces points :

**Placer un effet visuel au contact.** C'est l'usage le plus fréquent.

```dart
@override
void onCollisionStart(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  super.onCollisionStart(intersectionPoints, other);

  if (other is Gobelin && intersectionPoints.isNotEmpty) {
    // Moyenne des points : le « centre » du contact.
    final centre = Vector2.zero();
    for (final p in intersectionPoints) {
      centre.add(p);
    }
    centre.scale(1 / intersectionPoints.length);

    parent?.add(Etincelle(position: centre));
  }
}
```

`Vector2.add(autre)` additionne en place, `Vector2.scale(f)` multiplie en place. Ces méthodes viennent de `vector_math` et évitent de créer des objets temporaires à chaque frame.

**Déduire un côté d'impact approximatif.** En comparant l'abscisse d'un point d'intersection à celle de `absoluteCenter`, on sait grossièrement si le coup vient de la gauche ou de la droite, ce qui suffit pour un effet de recul. Pour une résolution correcte, on utilise les rectangles, pas les points : c'est l'objet de la section 32.19.

**Attention à la réutilisation des objets.** Flame réutilise ses structures internes pour éviter d'allouer à chaque frame. Si vous voulez conserver un point au-delà de la callback, clonez-le :

```dart
final pointConserve = intersectionPoints.first.clone();
```

Le `.clone()` est la même précaution que celle du chapitre 27 sur le partage d'un `Vector2` entre deux composants.

---

## 32.16 — Identifier l'autre composant avec `is` (rappel chapitre 10)

Le deuxième paramètre est typé `PositionComponent`. C'est le type le plus général qui puisse porter une hitbox. Pour savoir **ce que c'est vraiment**, on utilise l'opérateur `is` du chapitre 10.

```dart
@override
void onCollisionStart(
  Set<Vector2> intersectionPoints,
  PositionComponent other,
) {
  super.onCollisionStart(intersectionPoints, other);

  if (other is Piece) {
    // Dans ce bloc, `other` est promu au type Piece.
    // On peut lire other.valeur sans aucun cast.
    pieces += other.valeur;
    other.removeFromParent();
  }
}
```

La **promotion de type** de Dart fait le travail : après `if (other is Piece)`, le compilateur sait que `other` est une `Piece` dans tout le bloc. Aucun `as Piece` n'est nécessaire.

Pour plusieurs cas, on enchaîne les `else if`. L'ordre compte, car `is` est vrai pour les sous-classes.

```dart
// Hiérarchie : Ennemi <- Gobelin <- GobelinChef

@override
void onCollisionStart(
  Set<Vector2> points,
  PositionComponent other,
) {
  super.onCollisionStart(points, other);

  if (other is GobelinChef) {
    subirDegats(3);        // le plus spécifique D'ABORD
  } else if (other is Gobelin) {
    subirDegats(1);
  } else if (other is Ennemi) {
    subirDegats(1);        // tous les autres ennemis
  }
}
```

Si vous inversez l'ordre et testez `Ennemi` en premier, le chef ne fera jamais 3 points de dégâts : le premier `if` l'aura déjà capturé.

**Une alternative : le `switch` sur le type (Dart 3).** Le filtrage par motif du chapitre 04 fonctionne aussi : `switch (other) { case GobelinChef(): ...; case Gobelin(): ...; case Piece(:final valeur): ...; default: break; }`. Les deux formes sont équivalentes ; la chaîne de `else if` reste la plus répandue dans le code Flame existant.

**Une troisième voie : une interface commune.** Plutôt que de tester dix types, on peut faire porter le comportement par l'autre objet. C'est le sujet de la section suivante.

```dart
abstract class Ramassable {
  void ramasserPar(Heros heros);
}

// puis, dans le héros :
if (other is Ramassable) {
  other.ramasserPar(this);
}
```

Le héros n'a plus besoin de connaître `Piece`, `Potion`, `Cle` ni `Gemme`. Ajouter un nouvel objet ramassable ne touche plus au code du héros. C'est le principe ouvert/fermé du chapitre 10, appliqué aux collisions.

---

## 32.17 — Le patron « double dispatch » et pourquoi on teste le type

Quand deux hitboxes se touchent, Flame appelle **les deux** composants. C'est écrit noir sur blanc dans `StandardCollisionDetection` :

```dart
void handleCollisionStart(
  Set<Vector2> intersectionPoints,
  ShapeHitbox hitboxA,
  ShapeHitbox hitboxB,
) {
  hitboxA.onCollisionStart(intersectionPoints, hitboxB);
  hitboxB.onCollisionStart(intersectionPoints, hitboxA);
}
```

Chaque hitbox transmet ensuite l'appel à son composant parent, si celui-ci porte `CollisionCallbacks`.

```text
  LE HÉROS TOUCHE UNE PIÈCE

  collisionDetection.run()
       |
       +--> heros.hitbox.onCollisionStart(points, piece.hitbox)
       |         |
       |         +--> heros.onCollisionStart(points, piece)     <-- appel 1
       |
       +--> piece.hitbox.onCollisionStart(points, heros.hitbox)
                 |
                 +--> piece.onCollisionStart(points, heros)     <-- appel 2
```

Ce mécanisme s'appelle un **double dispatch** : la réaction dépend du type des **deux** objets, et chacun décide de son côté.

Trois conséquences pratiques.

**Conséquence 1 : ne traitez la même règle qu'une seule fois.** Si `Heros.onCollisionStart` retire la pièce **et** que `Piece.onCollisionStart` incrémente le score, tout va bien : chacun fait une chose. Mais si les deux incrémentent le score, le joueur gagne deux fois. Choisissez un côté, écrivez-le en commentaire, tenez-vous-y.

**Conséquence 2 : mettez la logique du côté qui possède la donnée.** Le score appartient au héros (ou au jeu) : c'est donc le héros qui l'incrémente. La disparition appartient à la pièce : c'est donc la pièce qui se retire. Cela donne un code où chaque classe ne modifie que ses propres champs.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  int pieces = 0;

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Piece) pieces += other.valeur; // le héros gère SON compteur
  }
}

class Piece extends PositionComponent with CollisionCallbacks {
  final int valeur = 10;

  @override
  Future<void> onLoad() async {
    add(CircleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Heros) removeFromParent();    // la pièce gère SA disparition
  }
}
```

**Conséquence 3 : un seul côté suffit parfois.** Si la pièce est `passive` et que le héros est `active`, le couple est bien détecté, et **les deux** sont notifiés. Le type `passive` n'empêche pas de recevoir les callbacks ; il empêche seulement d'être testé contre d'autres `passive`. Beaucoup de débutants croient l'inverse.

**Et si l'on ne teste pas le type ?** Le code suivant est un piège classique :

```dart
// NE FAITES PAS CELA.
class Porte extends PositionComponent with CollisionCallbacks {
  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    ouvrir(); // s'ouvre pour n'importe qui !
  }
}
```

Cette porte s'ouvre pour un gobelin, pour une flèche, pour une pièce qui tombe. Le test de type n'est pas une formalité : c'est la définition même de la règle du jeu.

---

## 32.18 — Empêcher le joueur de traverser un mur

Nous y voilà. Flame a détecté le contact ; il faut maintenant écrire la réaction. Commençons par la solution la plus simple, puis raffinons.

### Solution 1 : revenir à la position précédente

On mémorise la position d'avant le déplacement et on l'y remet en cas de contact.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(32, 40));

  final Vector2 velocite = Vector2.zero();
  final Vector2 _positionPrecedente = Vector2.zero();

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    super.update(dt);
    _positionPrecedente.setFrom(position);
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is Mur) {
      position.setFrom(_positionPrecedente);
    }
  }
}
```

**Résultat :** le héros ne traverse plus le mur.

**Défaut :** il se bloque **complètement**. Poussé en diagonale contre un mur vertical, il ne glisse pas le long du mur : il s'arrête net. C'est exactement le bug décrit en 24.15, « le personnage collé au mur ». Le joueur trouve les commandes molles sans savoir pourquoi.

Cette solution reste correcte pour un jeu à déplacement uniquement horizontal ou uniquement vertical (un Pong, un shoot'em up). Pour un donjon en vue de dessus, il faut mieux.

### Solution 2 : repousser juste ce qu'il faut

L'idée de la section 24.14 : calculer de combien les deux rectangles se chevauchent, et déplacer le héros **du minimum nécessaire** pour les séparer. Le héros garde alors son mouvement sur l'autre axe : il glisse.

C'est l'objet de la section suivante.

---

## 32.19 — Calculer la pénétration et repousser

`PositionComponent` fournit `toAbsoluteRect()`, qui renvoie le `Rect` du composant en coordonnées absolues. `Rect` vient de `dart:ui` et possède une méthode `intersect` : c'est tout ce qu'il nous faut.

```text
  CALCUL DE LA PÉNÉTRATION

  heros.toAbsoluteRect()          mur.toAbsoluteRect()
  left=100 top=100                left=126 top=60
  right=132 bottom=140            right=150 bottom=260

  chevauchement = heros.intersect(mur)
     left   = max(100, 126) = 126
     top    = max(100,  60) = 100
     right  = min(132, 150) = 132
     bottom = min(140, 260) = 140

     width  = 132 - 126 = 6      <- pénétration en X
     height = 140 - 100 = 40     <- pénétration en Y

  6 < 40  =>  on corrige sur X, de 6 pixels.
  Le héros est à gauche du mur => on le pousse VERS LA GAUCHE.
```

En code :

```dart
@override
void onCollision(Set<Vector2> points, PositionComponent other) {
  super.onCollision(points, other);
  if (other is! Mur) return;

  final moi = toAbsoluteRect();
  final lui = other.toAbsoluteRect();
  final chevauchement = moi.intersect(lui);

  if (chevauchement.width <= 0 || chevauchement.height <= 0) return;

  if (chevauchement.width < chevauchement.height) {
    // Le plus petit dégagement est horizontal.
    if (moi.center.dx < lui.center.dx) {
      position.x -= chevauchement.width;   // je suis à gauche : je recule
    } else {
      position.x += chevauchement.width;   // je suis à droite : j'avance
    }
    velocite.x = 0;
  } else {
    // Le plus petit dégagement est vertical.
    if (moi.center.dy < lui.center.dy) {
      position.y -= chevauchement.height;
    } else {
      position.y += chevauchement.height;
    }
    velocite.y = 0;
  }
}
```

**Pourquoi `velocite.x = 0` ?** Sans cette ligne, le héros est repoussé chaque frame mais sa vitesse le repousse à nouveau dans le mur à la frame suivante. Visuellement, il vibre. Annuler la composante de vitesse dirigée vers le mur supprime la vibration.

**Attention si `anchor` n'est pas `topLeft`.** `position` désigne la position de l'**ancre**. Le calcul ci-dessus déplace `position`, donc l'ancre, ce qui déplace bien tout le composant : il reste correct quelle que soit l'ancre. En revanche, si vous manipulez `position` en supposant qu'elle pointe le coin haut-gauche alors que l'ancre est `center`, tous vos calculs seront décalés d'une demi-taille. Fixez l'ancre une fois pour toutes au début du projet.

**Attention si la hitbox ne couvre pas tout le composant.** `toAbsoluteRect()` renvoie le rectangle du **composant**, pas celui de la hitbox. Si votre hitbox est plus petite (section 32.7), conservez-en une référence (`late final RectangleHitbox hitbox;`) et remplacez `toAbsoluteRect()` par `hitbox.toAbsoluteRect()` dans le calcul. On lit alors la hitbox mais on écrit toujours `position` du composant : décaler la hitbox elle-même la désolidariserait du visuel.

---

## 32.20 — Séparer les axes (rappel chapitre 24)

La règle du plus petit dégagement fonctionne bien dans la majorité des cas, mais elle échoue dans une situation précise : le **coin**. Quand la pénétration est presque identique sur les deux axes, la décision devient instable et le héros peut être éjecté sur le mauvais axe, ce qui produit un « accrochage » désagréable au bord des plateformes.

La section 24.16 avait donné la solution de fond : **traiter X et Y séparément**, et décider de l'axe de correction en regardant d'où l'on vient.

```text
  ENTRÉE HORIZONTALE                 ENTRÉE VERTICALE

  avant :  [H]                       avant :  [H]
                 [MUR]                          |
  après :   [H]                                 v
             ^^ chevauchement en X      après :  [H][MUR]  (superposés)

  Sur Y, le héros chevauchait          Sur X, le héros chevauchait
  DÉJÀ le mur avant de bouger.         DÉJÀ le mur avant de bouger.
  => l'entrée est horizontale          => l'entrée est verticale
  => corriger X                        => corriger Y
```

Le test s'écrit avec la position précédente, que nous mémorisons déjà.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(32, 40));

  final Vector2 velocite = Vector2.zero();
  final Vector2 _precedente = Vector2.zero();

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    super.update(dt);
    _precedente.setFrom(position);
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is! Mur) return;

    final lui = other.toAbsoluteRect();
    final moi = toAbsoluteRect();
    final chevauchement = moi.intersect(lui);
    if (chevauchement.width <= 0 || chevauchement.height <= 0) return;

    // Rectangle occupé AVANT le déplacement de cette frame.
    final avant = Rect.fromLTWH(
      _precedente.x,
      _precedente.y,
      size.x,
      size.y,
    );

    final chevauchaitDejaEnY = avant.top < lui.bottom && avant.bottom > lui.top;
    final chevauchaitDejaEnX = avant.left < lui.right && avant.right > lui.left;

    if (chevauchaitDejaEnY && !chevauchaitDejaEnX) {
      // On est entré par le côté : corriger X seulement.
      position.x = velocite.x > 0
          ? lui.left - size.x
          : lui.right;
      velocite.x = 0;
    } else if (chevauchaitDejaEnX && !chevauchaitDejaEnY) {
      // On est entré par le haut ou par le bas : corriger Y seulement.
      position.y = velocite.y > 0
          ? lui.top - size.y
          : lui.bottom;
      velocite.y = 0;
    } else {
      // Coin parfait : on retombe sur la règle du plus petit dégagement.
      if (chevauchement.width < chevauchement.height) {
        position.x += moi.center.dx < lui.center.dx
            ? -chevauchement.width
            : chevauchement.width;
        velocite.x = 0;
      } else {
        position.y += moi.center.dy < lui.center.dy
            ? -chevauchement.height
            : chevauchement.height;
        velocite.y = 0;
      }
    }
  }
}
```

Ce code suppose `anchor: Anchor.topLeft` (la valeur par défaut) et une hitbox qui remplit le composant. C'est la configuration la plus simple à raisonner, et celle que nous garderons pour le mini-projet.

**Pourquoi `position.x = lui.left - size.x` plutôt que `position.x -= chevauchement.width` ?** Les deux donnent le même résultat, mais l'affectation absolue est **idempotente** : si la callback est appelée deux fois dans la même frame (deux murs voisins), la seconde n'ajoute pas une seconde correction. C'est un petit détail qui supprime des tremblements dans les couloirs pavés de tuiles.

**Une limite honnête.** Flame n'exécute la détection qu'une fois par tick, après tous les `update`. Vous ne pouvez donc pas faire « je bouge en X, je teste, je bouge en Y, je teste » à l'intérieur d'une frame, comme au chapitre 24. Le code ci-dessus est une **reconstitution** de la séparation d'axes à partir de la position précédente. Elle est très bonne pour un jeu de donjon en vue de dessus ou un plateformer classique. Si vous avez besoin d'une résolution parfaite avec des pentes, des plateformes mobiles et des empilements, c'est le signal qu'il faut passer à Forge2D (section 32.30).

---

## 32.21 — Les hitboxes solides (`isSolid`)

Voici un piège subtil. La détection de Flame repose sur des **intersections de contours**. Si une hitbox est entièrement contenue dans une autre, les contours ne se croisent nulle part, et `intersectionPoints` est vide. Or, quand cet ensemble est vide, Flame considère qu'il n'y a **pas** de collision.

```text
  CAS 1 : LES CONTOURS SE CROISENT      CAS 2 : CONTENANCE TOTALE

     +---------+                            +---------------------+
     |    +----X-----+                      |                     |
     |    |    |     |                      |     +--------+      |
     +----X----+     |                      |     | petit  |      |
          |          |                      |     +--------+      |
          +----------+                      |                     |
                                            +---------------------+
     2 points d'intersection                0 point d'intersection
     => collision détectée                  => AUCUNE collision !
```

Le cas 2 arrive plus souvent qu'on ne croit : une petite pièce au milieu d'une grande zone de ramassage, un héros minuscule dans une salle de téléportation, un projectile lent complètement avalé par un boss.

Le drapeau `isSolid` corrige ce comportement. Voici le commentaire du code source de `ShapeComponent` :

> Si la forme est solide, les intersections se produisent même si l'autre composant est entièrement contenu dans la hitbox. Le point d'intersection est alors le centre du composant contenu. Une forme creuse entièrement contenue dans une hitbox solide produira un résultat d'intersection, mais pas l'inverse.

En clair : **c'est le contenant qui doit être `isSolid`**.

```dart
class ZoneDeTeleportation extends PositionComponent with CollisionCallbacks {
  ZoneDeTeleportation({super.position}) : super(size: Vector2(120, 90));

  @override
  Future<void> onLoad() async {
    add(
      RectangleHitbox(
        isSolid: true,                          // indispensable ici
        collisionType: CollisionType.passive,
      ),
    );
  }
}
```

Deux points à retenir :

- `isSolid` vaut `false` par défaut sur toutes les hitboxes ;
- `isSolid` n'a **rien à voir** avec `Paint.style` : il ne change pas le rendu, seulement la détection.

Le tableau de décision :

| Le petit objet peut-il être entièrement à l'intérieur du grand ? | Réglage |
| --- | --- |
| Non (murs et héros de taille comparable) | rien à faire |
| Oui, et vous voulez le détecter | `isSolid: true` sur le **grand** |
| Oui, et vous ne voulez pas le détecter | laisser `isSolid: false` |

Un test rapide pour diagnostiquer ce bug : si votre collision fonctionne quand le héros **entre** dans la zone puis cesse quand il arrive au centre, c'est `isSolid` qu'il vous manque.

---

## 32.22 — `ScreenHitbox` : les bords de l'écran

Au chapitre 24, empêcher le héros de sortir de l'écran demandait quatre tests écrits à la main. Flame fournit un composant qui matérialise les bords de la zone visible : `ScreenHitbox`.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';

class DonjonDeDart extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(ScreenHitbox());
  }
}
```

`ScreenHitbox` est un `PositionComponent` qui porte lui-même `CollisionCallbacks` et une `RectangleHitbox` remplissant sa surface. Il se redimensionne tout seul :

- ajouté au **monde**, il se cale sur `camera.visibleWorldRect` et suit la caméra, le zoom et la rotation du viewfinder ;
- ajouté directement au **jeu**, il prend la taille du jeu et se met à jour à chaque `onGameResize`.

**Ce que `ScreenHitbox` fait et ne fait pas.** Il déclenche les callbacks quand un objet touche le **bord** de la zone visible. Il ne bloque rien, comme toujours. La réaction reste à écrire.

```dart
class Fleche extends PositionComponent with CollisionCallbacks {
  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is ScreenHitbox) {
      removeFromParent(); // la flèche disparaît en sortant de l'écran
    }
  }
}
```

Pour une balle qui rebondit, on inverse la composante de vitesse correspondant au bord touché. Le bord se retrouve en comparant chaque point d'intersection aux quatre côtés du rectangle renvoyé par `other.toAbsoluteRect()` :

```dart
@override
void onCollisionStart(Set<Vector2> points, PositionComponent other) {
  super.onCollisionStart(points, other);
  if (other is! ScreenHitbox) return;

  final bords = other.toAbsoluteRect();
  for (final p in points) {
    if ((p.x - bords.left).abs() < 1 || (p.x - bords.right).abs() < 1) {
      velocite.x = -velocite.x;
    }
    if ((p.y - bords.top).abs() < 1 || (p.y - bords.bottom).abs() < 1) {
      velocite.y = -velocite.y;
    }
  }
}
```

**Limite à connaître.** Si l'objet dépasse déjà largement le bord au moment du contact, l'inversion peut le laisser coincé dehors : il rebondit vers l'extérieur à chaque frame. Ajoutez une correction de position, et préférez `onCollisionStart` (une seule fois) à `onCollision` (chaque tick).

**Pour borner un niveau plutôt qu'un écran**, `ScreenHitbox` n'est pas le bon outil : il suit la caméra. Créez quatre murs `passive` aux dimensions du niveau, ou utilisez `camera.setBounds(...)` vu au chapitre 31.

---

## 32.23 — Les raycasts : `raycast`, `raycastAll`, `raytrace`

Un **rayon** est une demi-droite : une origine et une direction. Le lancer de rayon répond à la question « qu'est-ce que je touche en premier si je pars d'ici dans cette direction ? ». C'est l'outil des lignes de vue, des lasers, des tirs instantanés et des capteurs de sol.

Le rayon se décrit avec `Ray2`, exporté par `package:flame/geometry.dart` :

```dart
final rayon = Ray2(
  origin: Vector2(100, 100),
  direction: Vector2(1, 0),
);
```

**La direction est normalisée automatiquement** par le setter de `Ray2` : `Vector2(3, 4)` devient `Vector2(0.6, 0.8)`. Vous n'avez donc pas à appeler `normalize()` vous-même, mais la direction ne doit pas être le vecteur nul.

Les trois méthodes vivent sur l'objet `collisionDetection` fourni par `HasCollisionDetection`.

### `raycast` — un seul rayon, le premier objet touché

```dart
RaycastResult<ShapeHitbox>? raycast(
  Ray2 ray, {
  double? maxDistance,
  bool Function(ShapeHitbox candidate)? hitboxFilter,
  List<ShapeHitbox>? ignoreHitboxes,
  RaycastResult<ShapeHitbox>? out,
});
```

Elle renvoie `null` si le rayon ne touche rien.

```dart
final resultat = collisionDetection.raycast(
  Ray2(origin: Vector2(0, 100), direction: Vector2(1, 0)),
  maxDistance: 400,
);

if (resultat != null) {
  debugPrint('touché : ${resultat.hitbox?.hitboxParent.runtimeType}');
  debugPrint('point  : ${resultat.intersectionPoint}');
  debugPrint('dist   : ${resultat.distance}');
}
```

Le `RaycastResult` expose (tous les membres sont **nullables**, car l'objet est réutilisé et remis à zéro) :

| Membre | Type | Contenu |
| --- | --- | --- |
| `isActive` | `bool` | `true` si le résultat contient un vrai impact |
| `hitbox` | `ShapeHitbox?` | la hitbox touchée ; `hitbox.hitboxParent` donne le composant |
| `intersectionPoint` | `Vector2?` | le point d'impact, en coordonnées absolues |
| `distance` | `double?` | la distance entre l'origine et l'impact |
| `normal` | `Vector2?` | la normale à la surface touchée |
| `reflectionRay` | `Ray2?` | le rayon réfléchi (rebond) |
| `isInsideHitbox` | `bool` | `true` si l'origine du rayon était déjà dans la hitbox |
| `clone()` | `RaycastResult` | copie indépendante |

> **Piège.** L'objet `RaycastResult` renvoyé est **réutilisé** d'un appel à l'autre pour éviter d'allouer à chaque frame. Si vous voulez conserver un résultat au-delà de la frame, appelez `clone()`.

### `raycastAll` — un éventail de rayons

```dart
List<RaycastResult<ShapeHitbox>> raycastAll(
  Vector2 origin, {
  required int numberOfRays,
  double startAngle = 0,
  double sweepAngle = tau,
  double? maxDistance,
  List<Ray2>? rays,
  bool Function(ShapeHitbox candidate)? hitboxFilter,
  List<ShapeHitbox>? ignoreHitboxes,
  List<RaycastResult<ShapeHitbox>>? out,
});
```

Elle tire `numberOfRays` rayons répartis uniformément entre `startAngle` et `startAngle + sweepAngle` (angles en radians ; la constante `tau` vaut `2 * pi` et vient de `package:flame/geometry.dart`).

```dart
// Champ de vision de 90 degrés, orienté vers la droite, 24 rayons.
final resultats = collisionDetection.raycastAll(
  absoluteCenter,
  numberOfRays: 24,
  startAngle: -pi / 4,
  sweepAngle: pi / 2,
  maxDistance: 300,
);
```

Usages : un cône de lumière, une onde de choc, un champ de vision d'ennemi, une détection de bord de plateforme.

### `raytrace` — les rebonds successifs

```dart
Iterable<RaycastResult<ShapeHitbox>> raytrace(
  Ray2 ray, {
  int maxDepth = 10,
  bool Function(ShapeHitbox candidate)? hitboxFilter,
  List<ShapeHitbox>? ignoreHitboxes,
  List<RaycastResult<ShapeHitbox>>? out,
});
```

Elle suit le rayon, puis son rayon réfléchi, puis le réfléchi du réfléchi, jusqu'à `maxDepth` rebonds. C'est le laser qui ricoche sur les murs du donjon.

```dart
final rayon = Ray2(origin: Vector2(20, 20), direction: Vector2(1, 1));

for (final r in collisionDetection.raytrace(rayon, maxDepth: 5)) {
  final p = r.intersectionPoint;
  if (p == null || p.distanceTo(rayon.origin) > 600) break;
  debugPrint('rebond sur ${r.hitbox?.hitboxParent.runtimeType} en $p');
}
```

**`raytrace` est paresseuse.** Elle renvoie un `Iterable` : rien n'est calculé tant que vous n'itérez pas. Un `break` dans la boucle économise réellement les calculs suivants. C'est aussi pour cela qu'il ne faut pas la stocker et l'itérer plus tard : les résultats sont recyclés.

**Coût.** Chaque rayon parcourt la liste des hitboxes. `raycastAll` avec 100 rayons et 300 hitboxes, exécuté à chaque frame, est un excellent moyen de faire tomber le jeu à 10 FPS. Deux réflexes : limiter avec `maxDistance`, et ne pas lancer les rayons à chaque frame (une fois toutes les 5 frames suffit très souvent pour une IA).

---

## 32.24 — Un raycast pour la ligne de vue d'un ennemi

Voici l'application concrète. Un gobelin doit poursuivre le héros **seulement s'il le voit**. Un héros caché derrière un mur ne doit pas être détecté.

```text
  SANS RAYCAST                          AVEC RAYCAST

  gobelin ---- distance 120 ----> héros  gobelin - - -X       héros
              (mur ignoré)                            |
       [MUR]                                        [MUR]
                                        le rayon touche le mur AVANT
  Le gobelin poursuit à travers          le héros => pas de ligne de vue
  le mur : effet ridicule.
```

L'algorithme tient en quatre étapes :

1. calculer le vecteur du gobelin vers le héros ;
2. si la distance dépasse la portée de vue, abandonner ;
3. lancer un rayon dans cette direction, limité à cette distance ;
4. la ligne de vue existe si le premier objet touché est le héros.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/geometry.dart';

class Gobelin extends PositionComponent
    with CollisionCallbacks, HasGameReference<DonjonDeDart> {
  Gobelin({super.position}) : super(size: Vector2(26, 30));

  late final RectangleHitbox hitbox;
  double porteeDeVue = 260;
  double vitesse = 55;
  bool voitLeHeros = false;

  // On ne relance pas le rayon à chaque frame : 6 fois par seconde suffit.
  double _cumul = 0;
  static const double _periodeDeVue = 1 / 6;

  @override
  Future<void> onLoad() async {
    hitbox = RectangleHitbox();
    add(hitbox);
  }

  @override
  void update(double dt) {
    super.update(dt);

    _cumul += dt;
    if (_cumul >= _periodeDeVue) {
      _cumul = 0;
      voitLeHeros = _testerLigneDeVue();
    }

    if (voitLeHeros) {
      final vers = game.heros.absoluteCenter - absoluteCenter;
      if (vers.length2 > 1) position += vers.normalized() * vitesse * dt;
    }
  }

  bool _testerLigneDeVue() {
    final origine = absoluteCenter;
    final vers = game.heros.absoluteCenter - origine;
    final distance = vers.length;
    if (distance > porteeDeVue || distance < 0.001) return false;

    final resultat = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: vers),
      maxDistance: distance,
      ignoreHitboxes: [hitbox], // sinon le gobelin se voit lui-même
    );

    return resultat?.hitbox?.hitboxParent is Heros;
  }
}
```

Trois détails valent qu'on s'y arrête.

**`ignoreHitboxes: [hitbox]`.** Le rayon part du centre du gobelin, donc de l'intérieur de sa propre hitbox. Sans cette ligne, le premier objet touché serait le gobelin lui-même et la ligne de vue serait toujours fausse. C'est la première chose à vérifier quand un raycast « ne voit jamais rien ».

**`maxDistance: distance`.** Inutile de tester au-delà du héros. Cela limite le travail et évite qu'un mur situé derrière le héros ne fasse échouer le test.

**`resultat?.hitbox?.hitboxParent is Heros`.** On remonte de la hitbox à son composant propriétaire avec `hitboxParent`. Les deux `?` sont nécessaires : `raycast` peut renvoyer `null`, et `hitbox` est nullable quand le résultat n'est pas actif.

Pour visualiser la ligne de vue pendant le développement, dessinez-la dans `render` avec `canvas.drawLine`, entre `(size / 2).toOffset()` et `((size / 2) + vers).toOffset()` : le canvas d'un composant est déjà translaté à sa position, on dessine donc en coordonnées **locales**. La correction 8 en donne le code complet.

**Variante : un cône de vision.** En remplaçant `raycast` par `raycastAll` avec `numberOfRays: 16`, `startAngle: -pi / 6` et `sweepAngle: pi / 3`, on obtient un vrai cône de 60 degrés : la ligne de vue existe si `resultats.any((r) => r.hitbox?.hitboxParent is Heros)`. Seize rayons six fois par seconde, c'est 96 raycasts par seconde et par gobelin : avec huit gobelins on approche du millier, mesurez avant de généraliser.

---

## 32.25 — Le broadphase de Flame (`QuadTreeCollisionDetection`)

Par défaut, Flame utilise un broadphase nommé `Sweep` (« balayage et élagage »). Il trie les AABB des hitboxes le long d'un axe et ne compare que celles dont les intervalles se recouvrent.

```text
  SWEEP AND PRUNE, sur l'axe X

  A [====]
  B      [======]
  C            [====]
  D                          [======]
  ---------------------------------------> X

  Couples examinés : A-B, B-C.
  D est trop loin : il n'est comparé à personne.
```

C'est efficace quand les objets sont répartis sur l'axe de tri. Cela l'est moins quand des centaines d'objets statiques se superposent sur cet axe, par exemple un mur vertical de 200 tuiles empilées.

Flame propose donc un second broadphase, fondé sur un **quadtree** : l'espace est découpé récursivement en quatre quadrants, et chaque hitbox n'est comparée qu'aux hitboxes du même quadrant.

```text
  QUADTREE, profondeur 2

  +-----------------+-----------------+
  |  .   .          |                 |
  |    .    .       |        .        |
  +--------+--------+                 |
  |  ..    | .  .   |                 |
  | .   .  |  .     |          .      |
  +--------+--------+-----------------+
  |                 |                 |
  |        .        |    .        .   |
  |                 |                 |
  +-----------------+-----------------+

  Un quadrant se divise dès qu'il contient plus de maxObjects hitboxes,
  jusqu'à une profondeur de maxLevels.
```

L'activation se fait avec un autre mixin de jeu, `HasQuadTreeCollisionDetection`, et un appel obligatoire à `initializeCollisionDetection` dans `onLoad`.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

class DonjonDeDart extends FlameGame with HasQuadTreeCollisionDetection {
  static const double largeurNiveau = 3000;
  static const double hauteurNiveau = 2000;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    initializeCollisionDetection(
      mapDimensions: const Rect.fromLTWH(0, 0, largeurNiveau, hauteurNiveau),
      minimumDistance: 10,
      maxObjects: 25,
      maxLevels: 10,
    );

    // ... ajout des composants APRÈS l'initialisation
  }
}
```

Signature réelle, telle qu'elle figure dans le code source de Flame 1.38.0 :

```dart
void initializeCollisionDetection({
  required Rect mapDimensions,
  double? minimumDistance,
  int maxObjects = 25,
  int maxLevels = 10,
});
```

| Paramètre | Rôle | Conseil |
| --- | --- | --- |
| `mapDimensions` | rectangle couvrant tout le niveau | doit **englober** toutes les positions possibles |
| `minimumDistance` | distance en dessous de laquelle deux objets sont considérés comme candidats | vaut `null` par défaut (test désactivé) |
| `maxObjects` | nombre d'objets avant qu'un quadrant ne se divise | 25 convient presque toujours |
| `maxLevels` | profondeur maximale de division | 10 convient presque toujours |

Trois précautions propres au quadtree.

**Les objets hors de `mapDimensions` posent problème.** Un composant qui sort du rectangle déclaré n'a plus de quadrant. Prévoyez une marge, ou supprimez les objets qui sortent du niveau.

**Une optimisation manuelle est disponible** après avoir peuplé le niveau. Elle réorganise l'arbre une fois que tous les objets statiques sont en place :

```dart
(collisionDetection as QuadTreeCollisionDetection)
    .quadBroadphase
    .tree
    .optimize();
```

**`onComponentTypeCheck` ne fonctionne qu'avec le quadtree.** Cette méthode du mixin `CollisionCallbacks` permet d'exclure définitivement des couples de types.

```dart
class Fleche extends PositionComponent with CollisionCallbacks {
  @override
  bool onComponentTypeCheck(PositionComponent other) {
    // Une flèche du héros ignore le héros et les autres flèches.
    if (other is Heros || other is Fleche) return false;
    return super.onComponentTypeCheck(other);
  }
}
```

Le résultat est **mis en cache** : n'y testez que des types, jamais un état variable.

---

## 32.26 — Quand passer au quadtree

La documentation officielle de Flame est prudente sur ce point, et nous le serons aussi :

> `HasQuadTreeCollisionDetection` est utile si vous avez beaucoup d'entités entrant en collision, dont la plupart sont statiques (plateformes, murs, arbres, bâtiments). Faites toujours des essais avant de choisir : il n'est pas rare d'observer de meilleures performances avec le mixin `HasCollisionDetection` par défaut.

Autrement dit : **le quadtree n'est pas une amélioration automatique**. C'est un compromis.

| Critère | `HasCollisionDetection` (sweep) | `HasQuadTreeCollisionDetection` |
| --- | --- | --- |
| Nombre de hitboxes | jusqu'à quelques centaines | plusieurs milliers |
| Proportion d'objets statiques | indifférente | l'avantage vient des statiques |
| Objets très mobiles et dispersés | bon | l'arbre se réorganise sans cesse |
| Taille du niveau | indifférente | doit être connue à l'avance |
| Mise en place | zéro configuration | `initializeCollisionDetection` obligatoire |
| Objets hors des limites | sans effet | comportement indéfini |
| `onComponentTypeCheck` | ignoré | actif |

Une méthode de décision honnête, en quatre étapes :

1. **Écrivez le jeu avec `HasCollisionDetection`.** C'est le défaut, il fonctionne, il ne demande rien.
2. **Mesurez.** Ajoutez `FpsTextComponent`. Notez le nombre d'images par seconde sur le vrai appareil cible, pas sur votre ordinateur de développement.
3. **Essayez d'abord les leviers gratuits** de la section 32.11 : décor en `passive`, fusion des tuiles de mur, hitboxes plus simples. Dans la grande majorité des projets de niveau débutant à intermédiaire, cela suffit.
4. **Alors seulement**, essayez le quadtree et mesurez à nouveau. S'il n'apporte rien, revenez en arrière.

Pour le « Donjon de Dart » du chapitre 39, avec une centaine de hitboxes par salle, le broadphase par défaut est largement suffisant. Le quadtree deviendra pertinent si vous décidez un jour de faire un monde ouvert d'un seul tenant.

---

## 32.27 — Les capteurs (triggers) : ramassage d'objets

Un **capteur** — ou *trigger* — est une zone qui détecte sans bloquer. Vous en connaissez déjà le principe depuis la section 24.4. Dans Flame, un capteur n'est pas un type particulier : c'est simplement une hitbox dont la réaction ne repousse personne.

```text
  MUR (solide)                    CAPTEUR (trigger)

  onCollision -> je repousse      onCollisionStart -> je déclenche
                                                       et je ne bouge personne
```

Le ramassage d'une pièce est le capteur le plus simple.

```dart
class Piece extends PositionComponent with CollisionCallbacks {
  Piece({super.position}) : super(size: Vector2.all(14));

  static final Paint _peinture = Paint()..color = const Color(0xFFFFC107);
  final int valeur = 10;
  bool _prise = false;

  @override
  Future<void> onLoad() async {
    add(CircleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawCircle((size / 2).toOffset(), size.x / 2, _peinture);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (_prise || other is! Heros) return;   // garde-fou : voir plus bas
    _prise = true;
    other.pieces += valeur;
    removeFromParent();
  }
}
```

**Pourquoi le drapeau `_prise` ?** Parce que `removeFromParent()` n'est pas immédiat. Comme tout ajout ou retrait dans Flame, la suppression est mise en file d'attente et appliquée à la fin de la frame. Si deux hitboxes du héros touchent la pièce dans la même frame, `onCollisionStart` peut être appelée deux fois avant la suppression effective, et le joueur gagne 20 pièces au lieu de 10. Le drapeau règle définitivement la question. Prenez-en l'habitude sur tous vos objets ramassables.

**Le cas de la porte à clé** montre un capteur qui interroge l'état du héros :

```dart
class PorteDeSortie extends PositionComponent with CollisionCallbacks {
  PorteDeSortie({super.position}) : super(size: Vector2(28, 40));

  static final Paint _fermee = Paint()..color = const Color(0xFF795548);
  static final Paint _ouverte = Paint()..color = const Color(0xFF4DD0E1);
  bool ouverte = false;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(isSolid: true, collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), ouverte ? _ouverte : _fermee);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Heros && other.aLaCle) ouverte = true;
  }
}
```

**Un capteur peut aussi mesurer une durée de séjour.** Une plaque de pression qui n'ouvre la herse qu'après une seconde utilise les trois callbacks à la fois : `onCollisionStart` lève un drapeau `_heroDessus`, `onCollisionEnd` le rabaisse et remet le chronomètre à zéro, et `update` accumule `dt` tant que le drapeau est levé. Retenez la répartition : la détection est dans les callbacks, la mesure du temps est dans `update`, là où `dt` est disponible.

---

## 32.28 — Les dégâts et l'invincibilité temporaire

Sans précaution, un héros qui reste au contact d'un gobelin perd une vie **par tick**. Même en utilisant `onCollisionStart`, un gobelin qui entre et sort trois fois par seconde vide la barre de vie en un instant.

La solution universelle, déjà vue en 24.27, est l'**invincibilité temporaire** : après un coup, le héros ignore les dégâts pendant un court instant, et clignote pour que le joueur comprenne pourquoi.

```dart
class Heros extends PositionComponent with CollisionCallbacks {
  Heros({super.position}) : super(size: Vector2(28, 34));

  static final Paint _normal = Paint()..color = const Color(0xFF4CAF50);
  static final Paint _blesse = Paint()..color = const Color(0xFFEF5350);

  int vies = 3;
  double _invincibiliteRestante = 0;
  static const double dureeInvincibilite = 1.2;

  bool get estInvincible => _invincibiliteRestante > 0;

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void update(double dt) {
    super.update(dt);
    if (_invincibiliteRestante > 0) {
      _invincibiliteRestante -= dt;
      if (_invincibiliteRestante < 0) _invincibiliteRestante = 0;
    }
  }

  void subirDegats(int montant) {
    if (estInvincible) return;
    vies -= montant;
    _invincibiliteRestante = dureeInvincibilite;
    if (vies <= 0) {
      // gameOver();
    }
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Gobelin) subirDegats(1);
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    // Contact prolongé : on retente à chaque tick, subirDegats filtre.
    if (other is Gobelin) subirDegats(1);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    // Clignotement : visible une frame sur deux pendant l'invincibilité.
    if (estInvincible && (_invincibiliteRestante * 12).floor().isEven) return;
    canvas.drawRect(size.toRect(), estInvincible ? _blesse : _normal);
  }
}
```

Trois choix de conception à commenter.

**Pourquoi appeler `subirDegats` dans les deux callbacks ?** `onCollisionStart` traite le premier contact, `onCollision` traite le contact prolongé au-delà de la fin de l'invincibilité. Comme `subirDegats` refuse net si le héros est invincible, il n'y a aucun risque de double décompte. La logique de filtrage est **au même endroit**, ce qui est la vraie qualité de ce code.

**Pourquoi un compteur plutôt qu'un `Timer` ?** Un simple `double` décrémenté dans `update` est lisible, sans allocation, et se remet à zéro trivialement. Le `TimerComponent` du chapitre 33 sera préférable dès que vous aurez plusieurs délais à gérer, mais ici il serait surdimensionné.

**Pourquoi le clignotement dans `render` et non un effet ?** Parce qu'un simple `return` avant le dessin coûte zéro. Le chapitre 33 vous montrera la version élégante avec `OpacityEffect` et un `EffectController` qui alterne.

**Variante : l'invincibilité par attaquant.** Dans certains jeux, on veut pouvoir être touché par deux ennemis différents sans délai. Remplacez alors le `double` unique par une `Map<PositionComponent, double>` associant un délai à chaque source de dégâts, décrémentée dans `update`. Pensez à purger les entrées nulles, sinon la `Map` grossit indéfiniment.

---

## 32.29 — Les projectiles rapides et le tunneling (rappel chapitre 24)

Reprenons le raisonnement de la section 24.18, appliqué à Flame.

La détection tourne **une fois par tick**. Entre deux ticks, un composant se téléporte de sa position précédente à sa nouvelle position. Si un mur tient dans l'intervalle, personne ne le remarque.

```text
  Flèche à 900 px/s, dt = 1/60 s  =>  15 px par tick
  Mur de 12 px d'épaisseur.

  tick n     : [F]        |M|              pas de contact
  tick n+1   :            |M|      [F]     pas de contact
                           ^
                    la flèche a sauté par-dessus le mur
```

Le seuil est simple à retenir :

> **Si `vitesse * dt` dépasse l'épaisseur de l'obstacle, le tunneling est possible.**

Flame ne propose pas de détection continue. Trois remèdes, du plus simple au plus robuste.

### Remède 1 : épaissir la hitbox de l'obstacle

Le moins élégant, mais parfois le plus raisonnable. Un mur dessiné sur 12 pixels peut porter une hitbox de 40 pixels si le côté « invisible » du mur n'est pas accessible au joueur.

### Remède 2 : le sous-échantillonnage manuel

On découpe le déplacement de la frame en plusieurs petits pas de quelques pixels, comme en 24.20. Dans Flame, ce remède **seul ne suffit pas** : la détection n'a lieu qu'une fois, après tous les `update`, donc les positions intermédiaires ne sont jamais testées. Il n'a d'intérêt qu'associé à un test explicite, ce qui nous amène au troisième remède.

### Remède 3 : le raycast (recommandé)

C'est la bonne solution. Avant de déplacer la flèche, on tire un rayon de la position actuelle vers la position visée, limité à la distance parcourue pendant la frame. Si le rayon touche quelque chose, on place la flèche au point d'impact et on traite la collision soi-même.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/geometry.dart';

class Fleche extends PositionComponent
    with CollisionCallbacks, HasGameReference<DonjonDeDart> {
  Fleche({super.position, required this.direction})
      : super(size: Vector2(10, 4), anchor: Anchor.center);

  static final Paint _peinture = Paint()..color = const Color(0xFFFFF176);

  final Vector2 direction;
  double vitesse = 900;

  late final RectangleHitbox hitbox;

  @override
  Future<void> onLoad() async {
    hitbox = RectangleHitbox();
    add(hitbox);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final distance = vitesse * dt;

    final resultat = game.collisionDetection.raycast(
      Ray2(origin: absoluteCenter, direction: direction),
      maxDistance: distance,
      ignoreHitboxes: [hitbox],
    );

    final cible = resultat?.hitbox?.hitboxParent;

    if (cible is Mur || cible is Gobelin) {
      position.setFrom(resultat!.intersectionPoint!);
      if (cible is Gobelin) cible.subirDegats(1);
      removeFromParent();
      return;
    }

    position += direction * distance;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Pourquoi cela marche.** Le rayon teste **tout le segment** parcouru pendant la frame, pas seulement les deux extrémités. Aucun mur ne peut passer entre les mailles, quelle que soit la vitesse.

**Points d'attention :**

- `direction` doit être normalisée avant d'être passée au constructeur, car on l'utilise aussi dans `position += direction * distance`. `Ray2` la normalise de son côté, mais votre copie ne l'est pas ;
- `ignoreHitboxes: [hitbox]` évite que la flèche ne se détecte elle-même ;
- la flèche garde tout de même une `RectangleHitbox`, ce qui permet aux **autres** objets de la détecter normalement ;
- ce composant n'utilise pas `onCollisionStart` du tout : le raycast fait tout le travail.

Un tableau récapitulatif pour choisir :

| Vitesse du projectile | Solution |
| --- | --- |
| moins de 300 px/s | rien à faire, `onCollisionStart` suffit |
| 300 à 800 px/s | hitbox d'obstacle épaissie, ou raycast |
| plus de 800 px/s | raycast obligatoire |
| tir instantané (laser, fusil) | raycast, sans composant projectile du tout |

---

## 32.30 — Quand utiliser Forge2D plutôt que la détection intégrée (annonce du chapitre 34)

`flame_forge2d` est le pont vers Forge2D, portage Dart du célèbre moteur physique Box2D. Ce n'est **pas** une version améliorée de la détection intégrée : c'est un outil différent, avec un modèle de pensée différent.

| Aspect | Détection intégrée de Flame | `flame_forge2d` |
| --- | --- | --- |
| Rôle | détecter les contacts | simuler une physique complète |
| Résolution des contacts | à votre charge | automatique |
| Masse, densité, restitution, friction | absentes | présentes |
| Forces, impulsions, couples | absents | présents |
| Articulations (joints), moteurs, ressorts | absents | présents |
| Empilements d'objets | instables si écrits à la main | gérés |
| Unité de mesure | le pixel | le mètre (conversion nécessaire) |
| Contrôle direct de la position | total | déconseillé (on applique des forces) |
| Coût processeur | faible | notable |
| Courbe d'apprentissage | douce | raide |
| Version du paquet | inclus dans `flame` | `flame_forge2d ^0.19.3+7` |

**Choisissez la détection intégrée si** votre jeu ressemble à ceci : un personnage qui marche et saute, des ennemis qui patrouillent, des objets à ramasser, des projectiles, des murs immobiles. C'est-à-dire la quasi-totalité des jeux de plateforme, des jeux de donjon, des shoot'em up, des runners, des puzzles à grille. Le « Donjon de Dart » est exactement dans cette catégorie.

**Choisissez Forge2D si** votre jeu a besoin de : caisses que l'on pousse et qui s'empilent, ragdolls, véhicules à roues et suspensions, chaînes et cordes, ponts articulés, billards, boulets de démolition, ou d'une gravité crédible entre plusieurs corps.

**Le signal d'alarme.** Si vous vous surprenez à écrire vous-même une gestion de masse, de rebond élastique, de frottement de contact et d'empilement stable, vous êtes en train de réécrire Box2D. Arrêtez-vous et passez à Forge2D : vous perdrez deux jours d'apprentissage et vous en gagnerez trente.

**Un piège fréquent chez les débutants** consiste à choisir Forge2D « parce que c'est plus sérieux ». Le résultat habituel est un personnage impossible à contrôler : dans un moteur physique, on ne fixe pas la position d'un corps, on lui applique des forces, et un personnage de plateforme réactif demande alors beaucoup de réglages. Un platformer se code presque toujours mieux **sans** moteur physique.

Le chapitre 34 vous donnera un aperçu concret de `flame_forge2d` : `Forge2DGame`, `BodyComponent`, `BodyDef`, `FixtureDef` et la fameuse conversion pixels/mètres.

---

## 32.31 — Mini-projet : le « Donjon de Dart » qui se cogne, ramasse et souffre

Tout ce chapitre converge ici. Le programme suivant est **complet, exécutable, et n'a besoin d'aucune image**. Il tient dans un seul `lib/main.dart`.

Ce qu'il fait :

- le héros se déplace aux flèches ou en `Z Q S D` ;
- il ne traverse pas les murs (résolution par séparation d'axes, section 32.20) ;
- il ramasse les pièces (capteur, section 32.27) ;
- il perd une vie au contact d'un gobelin, avec invincibilité et clignotement (section 32.28) ;
- les gobelins patrouillent et rebondissent sur les murs ;
- le HUD affiche vies et pièces ;
- `F1` bascule l'affichage des hitboxes.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(
    MaterialApp(
      home: Scaffold(
        backgroundColor: const Color(0xFF12100E),
        body: GameWidget<DonjonDeDart>(game: DonjonDeDart()),
      ),
    ),
  );
}

class DonjonDeDart extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  late final Heros heros;
  late final TextComponent hud;

  int pieces = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Enceinte de la salle : 4 murs passifs.
    await world.addAll([
      Mur(position: Vector2(0, 0), size: Vector2(560, 20)),
      Mur(position: Vector2(0, 380), size: Vector2(560, 20)),
      Mur(position: Vector2(0, 0), size: Vector2(20, 400)),
      Mur(position: Vector2(540, 0), size: Vector2(20, 400)),
      // Deux piliers intérieurs.
      Mur(position: Vector2(160, 120), size: Vector2(24, 160)),
      Mur(position: Vector2(360, 120), size: Vector2(24, 160)),
    ]);

    for (final p in const [
      [70.0, 60.0], [250.0, 60.0], [460.0, 60.0],
      [70.0, 330.0], [250.0, 330.0], [460.0, 330.0],
      [270.0, 200.0],
    ]) {
      await world.add(Piece(position: Vector2(p[0], p[1])));
    }

    await world.add(Gobelin(position: Vector2(250, 120), vitesseX: 70));
    await world.add(Gobelin(position: Vector2(430, 260), vitesseX: -55));

    heros = Heros(position: Vector2(60, 190));
    await world.add(heros);

    hud = TextComponent(
      text: '',
      position: Vector2(12, 8),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(hud);
  }

  @override
  void update(double dt) {
    super.update(dt);
    hud.text = 'Vies : ${heros.vies}   Pièces : $pieces';
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent && event.logicalKey == LogicalKeyboardKey.f1) {
      debugMode = !debugMode;
      return KeyEventResult.handled;
    }
    return super.onKeyEvent(event, keysPressed);
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}

class Piece extends PositionComponent with CollisionCallbacks {
  Piece({super.position}) : super(size: Vector2.all(14));

  static final Paint _peinture = Paint()..color = const Color(0xFFFFC107);

  final int valeur = 1;
  bool _prise = false;

  @override
  Future<void> onLoad() async {
    add(CircleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawCircle((size / 2).toOffset(), size.x / 2, _peinture);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (_prise || other is! Heros) return;
    _prise = true;
    other.gagnerPiece(valeur);
    removeFromParent();
  }
}

class Gobelin extends PositionComponent with CollisionCallbacks {
  Gobelin({super.position, required this.vitesseX})
      : super(size: Vector2(26, 30));

  static final Paint _peinture = Paint()..color = const Color(0xFF8BC34A);

  double vitesseX;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    super.update(dt);
    position.x += vitesseX * dt;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is! Mur) return;

    final moi = toAbsoluteRect();
    final lui = other.toAbsoluteRect();
    final chevauchement = moi.intersect(lui);
    if (chevauchement.width <= 0) return;

    // Demi-tour : on dégage d'abord, on inverse ensuite.
    position.x += moi.center.dx < lui.center.dx
        ? -chevauchement.width
        : chevauchement.width;
    vitesseX = -vitesseX;
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2(26, 30));

  static final Paint _normal = Paint()..color = const Color(0xFF4FC3F7);
  static final Paint _blesse = Paint()..color = const Color(0xFFEF5350);

  final Vector2 velocite = Vector2.zero();
  final Vector2 _precedente = Vector2.zero();
  double vitesse = 150;

  int vies = 3;
  double _invincible = 0;
  static const double dureeInvincibilite = 1.2;

  bool get estInvincible => _invincible > 0;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    final gauche = keysPressed.contains(LogicalKeyboardKey.arrowLeft) ||
        keysPressed.contains(LogicalKeyboardKey.keyQ);
    final droite = keysPressed.contains(LogicalKeyboardKey.arrowRight) ||
        keysPressed.contains(LogicalKeyboardKey.keyD);
    final haut = keysPressed.contains(LogicalKeyboardKey.arrowUp) ||
        keysPressed.contains(LogicalKeyboardKey.keyZ);
    final bas = keysPressed.contains(LogicalKeyboardKey.arrowDown) ||
        keysPressed.contains(LogicalKeyboardKey.keyS);

    velocite.setValues(
      (droite ? 1 : 0) - (gauche ? 1 : 0),
      (bas ? 1 : 0) - (haut ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(vitesse);
    }
    return true;
  }

  void gagnerPiece(int valeur) {
    final jeu = findParent<FlameGame>();
    if (jeu is DonjonDeDart) jeu.pieces += valeur;
  }

  void subirDegats(int montant) {
    if (estInvincible) return;
    vies -= montant;
    _invincible = dureeInvincibilite;
    if (vies < 0) vies = 0;
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (_invincible > 0) {
      _invincible -= dt;
      if (_invincible < 0) _invincible = 0;
    }

    _precedente.setFrom(position);
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);

    if (other is Gobelin) {
      subirDegats(1);
      return;
    }
    if (other is! Mur) return;

    final lui = other.toAbsoluteRect();
    final moi = toAbsoluteRect();
    final chevauchement = moi.intersect(lui);
    if (chevauchement.width <= 0 || chevauchement.height <= 0) return;

    final avant = Rect.fromLTWH(_precedente.x, _precedente.y, size.x, size.y);
    final dejaY = avant.top < lui.bottom && avant.bottom > lui.top;
    final dejaX = avant.left < lui.right && avant.right > lui.left;

    if (dejaY && !dejaX) {
      position.x = velocite.x > 0 ? lui.left - size.x : lui.right;
      velocite.x = 0;
    } else if (dejaX && !dejaY) {
      position.y = velocite.y > 0 ? lui.top - size.y : lui.bottom;
      velocite.y = 0;
    } else if (chevauchement.width < chevauchement.height) {
      position.x += moi.center.dx < lui.center.dx
          ? -chevauchement.width
          : chevauchement.width;
      velocite.x = 0;
    } else {
      position.y += moi.center.dy < lui.center.dy
          ? -chevauchement.height
          : chevauchement.height;
      velocite.y = 0;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    if (estInvincible && (_invincible * 12).floor().isEven) return;
    canvas.drawRect(size.toRect(), estInvincible ? _blesse : _normal);
  }
}
```

**Résultat :**

```text
Vies : 3   Pièces : 0
Vies : 3   Pièces : 3        (après ramassage de trois pièces)
Vies : 2   Pièces : 3        (après un contact avec un gobelin)
```

Quelques points de lecture.

**Le héros est le seul objet `active` du décor.** Les murs et les pièces sont `passive`. Les gobelins sont `active` (valeur par défaut), sans quoi ils ne verraient pas les murs passifs et sortiraient de la salle.

**Le gobelin résout ses propres collisions.** Il applique la version simple de la section 32.19, qui suffit puisqu'il ne se déplace que sur X.

**`gagnerPiece` remonte au jeu avec `findParent`.** On aurait pu utiliser le mixin `HasGameReference<DonjonDeDart>` du chapitre 27, plus direct. Les deux formes sont correctes ; la seconde est préférable dans un vrai projet.

**Le HUD est dans le `viewport`.** C'est le point du chapitre 31 : ajouté au monde, il défilerait avec la caméra.

**Pistes d'extension**, dans l'ordre de difficulté :

1. ajouter une potion qui rend une vie ;
2. ajouter une clé et une porte de sortie (section 32.27) ;
3. faire tirer une flèche au héros avec la barre d'espace (section 32.29) ;
4. donner aux gobelins une ligne de vue par raycast (section 32.24) ;
5. remplacer les rectangles par des sprites (chapitre 29) sans toucher aux hitboxes — c'est le test ultime de la séparation hitbox/visuel.

---

## 32.32 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Aucune collision ne se déclenche, nulle part | le mixin `HasCollisionDetection` n'est pas sur le jeu (ou sur le monde) | `class MonJeu extends FlameGame with HasCollisionDetection` |
| Un composant précis ne réagit jamais | il porte `CollisionCallbacks` mais aucune hitbox n'a été ajoutée | `add(RectangleHitbox());` dans `onLoad()` |
| La hitbox existe mais reste de taille nulle | le composant a une `size` de `Vector2.zero()` | donner une `size` au constructeur, ou après le chargement du sprite |
| Deux objets superposés ne se voient pas | les deux hitboxes sont en `CollisionType.passive` | mettre au moins l'un des deux en `active` |
| Le jeu rame avec beaucoup de murs | tout est en `CollisionType.active` | décor en `passive` ; fusionner les tuiles voisines |
| `isColliding` reste toujours `false` | `super.onCollisionStart(...)` a été oublié | appeler `super` en première ligne des trois callbacks |
| Le joueur perd 60 vies par seconde | la logique de dégâts est dans `onCollision` sans filtre | utiliser `onCollisionStart`, ou une invincibilité temporaire |
| Le score augmente de deux crans par pièce | la règle est écrite des deux côtés (double dispatch) | ne l'écrire que d'un côté, plus un drapeau `_prise` |
| Le héros se bloque net contre un mur au lieu de glisser | on remet simplement la position précédente sur les deux axes | repousser sur le plus petit axe, puis séparer X et Y (32.20) |
| Le héros vibre contre le mur | la vitesse le repousse dans le mur à chaque frame | annuler la composante de vitesse : `velocite.x = 0` |
| Le héros s'accroche aux angles des tuiles | correction appliquée sur les deux axes en même temps | séparer les axes à l'aide de la position précédente |
| La hitbox est calée sur le sprite entier | on a laissé `RectangleHitbox()` sans argument sur un sprite avec des marges | dimensionner et décaler la hitbox (32.7) ou utiliser `.relative()` |
| Un petit objet au centre d'une grande zone n'est pas détecté | les contours ne se croisent pas | `isSolid: true` sur la **grande** hitbox |
| Un `PolygonHitbox` donne des résultats absurdes | le polygone est concave | le découper en plusieurs hitboxes convexes |
| `PolygonHitbox` lève `UnsupportedError` | on a demandé un remplissage du parent | utiliser `RectangleHitbox()` sans argument |
| Un raycast ne détecte jamais rien | l'origine est à l'intérieur de la hitbox du tireur | `ignoreHitboxes: [maHitbox]` |
| Un projectile rapide traverse les murs | tunneling : `vitesse * dt` dépasse l'épaisseur du mur | raycast le long du déplacement (32.29) |
| Le résultat d'un raycast devient faux à la frame suivante | le `RaycastResult` est recyclé par Flame | `clone()` si vous le conservez |
| Une pièce donne deux fois son score | `removeFromParent()` n'est pas immédiat | ajouter un drapeau booléen `_prise` |
| `estAuSol` devient `false` alors qu'on est sur une autre plateforme | `onCollisionEnd` ne regarde qu'un seul contact | recalculer depuis `activeCollisions` après `super` |
| Le quadtree se comporte de façon erratique | des objets sortent de `mapDimensions` | agrandir le rectangle ou supprimer les objets hors limites |
| `initializeCollisionDetection` non appelé avec le quadtree | le mixin l'exige dans `onLoad` | l'appeler avant d'ajouter les composants |

---

## 32.33 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `HasCollisionDetection` | mixin sur le jeu (ou le monde) ; sans lui, rien n'est détecté |
| Ordre d'exécution | `update` de tout l'arbre, **puis** détection, une fois par tick |
| `CollisionCallbacks` | mixin sur le composant qui veut réagir |
| `@mustCallSuper` | les trois callbacks doivent appeler `super` en premier |
| Hitbox | un **composant enfant**, avec `position`, `size`, `angle`, `anchor` |
| `RectangleHitbox()` sans argument | remplit le parent et suit ses changements de taille |
| `.relative(...)` | hitbox exprimée en fraction de `parentSize` |
| Hitbox et visuel | indépendants : c'est la hitbox qui définit la règle du jeu |
| `debugMode = true` | affiche toutes les hitboxes de l'arbre |
| `CollisionType.active` | testé contre `active` et `passive` |
| `CollisionType.passive` | testé uniquement contre `active` |
| `CollisionType.inactive` | jamais testé ; idéal pour désactiver temporairement |
| Règle de choix | ce qui bouge est `active`, le décor est `passive` |
| `onCollisionStart` | une fois, au début du contact : ramassage, dégâts, son |
| `onCollision` | à chaque tick : repoussement, dégâts continus (avec `dt`) |
| `onCollisionEnd` | une fois, à la fin : quitter le sol, sortir d'une zone |
| `intersectionPoints` | points de croisement des contours, en coordonnées absolues |
| Identification | opérateur `is` ; du type le plus spécifique au plus général |
| Double dispatch | les **deux** composants sont notifiés ; n'écrire la règle qu'une fois |
| Résolution | **Flame ne résout rien** : le repoussement est votre code |
| Pénétration | `toAbsoluteRect().intersect(...)` puis correction du plus petit axe |
| Séparation des axes | décider l'axe à corriger d'après la position précédente |
| `isSolid` | à mettre sur le **contenant** pour détecter la contenance totale |
| `ScreenHitbox` | matérialise les bords de la zone visible ; suit la caméra |
| `raycast` | premier objet touché ; `hitbox`, `intersectionPoint`, `distance`, `normal` |
| `raycastAll` | éventail de rayons : cône de vision, onde de choc |
| `raytrace` | suit les réflexions ; **paresseux** |
| `ignoreHitboxes` | indispensable pour qu'un tireur ne se détecte pas lui-même |
| Broadphase | *sweep and prune* par défaut ; `HasQuadTreeCollisionDetection` en option |
| Quadtree | pour des milliers de hitboxes majoritairement statiques ; à mesurer |
| Capteur (trigger) | hitbox `passive` qui déclenche sans repousser |
| Invincibilité | un compteur en secondes décrémenté dans `update` |
| Tunneling | `vitesse * dt` > épaisseur : la seule vraie solution est le raycast |
| Forge2D | pour la vraie physique (masse, joints, empilements), pas pour un platformer |

---

## 32.34 — Exercices

### Exercice 1 — Activer et observer (facile)
Écrivez un jeu Flame avec un carré vert immobile au centre et un carré marron qui traverse l'écran de gauche à droite. Ajoutez `HasCollisionDetection`, `CollisionCallbacks` et une `RectangleHitbox` à chacun. Affichez un message dans la console au début et à la fin du contact. Activez `debugMode`.

### Exercice 2 — Hitbox décalée (facile)
Reprenez l'exercice 1. Donnez au carré vert une taille de 60 x 60, mais une hitbox de 20 x 20 centrée. Vérifiez visuellement en `debugMode` que le message n'apparaît que lorsque les deux hitboxes se touchent, et non les deux carrés.

### Exercice 3 — `passive` et `inactive` (facile)
Créez trois carrés immobiles et superposés deux à deux : un `active`, un `passive`, un `inactive`. Faites-les tous porter `CollisionCallbacks` et affichez chaque contact détecté. Vérifiez expérimentalement la table de vérité de la section 32.9.

### Exercice 4 — Ramassage de pièces (moyen)
Écrivez un héros contrôlé aux flèches et cinq pièces disposées au hasard. Chaque pièce ramassée incrémente un compteur affiché dans le HUD. Utilisez un drapeau pour interdire le double comptage.

### Exercice 5 — Le mur infranchissable (moyen)
Ajoutez au héros de l'exercice 4 quatre murs formant une pièce fermée, plus un pilier central. Implémentez la résolution par plus petit dégagement de la section 32.19. Le héros doit glisser le long des murs, pas s'y coller.

### Exercice 6 — Séparation des axes (moyen)
Remplacez la résolution de l'exercice 5 par celle de la section 32.20. Comparez le comportement dans un couloir formé de plusieurs tuiles de mur adjacentes : le héros ne doit plus s'accrocher aux jointures.

### Exercice 7 — Le capteur solide (moyen)
Créez une grande zone de 160 x 120 et une petite pièce de 10 x 10 placée en plein centre. Constatez qu'aucune collision n'est détectée, puis corrigez avec `isSolid`. Affichez le nombre de points d'intersection reçus dans les deux cas.

### Exercice 8 — Ligne de vue (difficile)
Écrivez un gobelin qui poursuit le héros seulement s'il le voit. Un mur placé entre les deux doit interrompre la poursuite. Utilisez `raycast` avec `maxDistance` et `ignoreHitboxes`. Dessinez la ligne de vue en `debugMode`.

### Exercice 9 — Flèche rapide sans tunneling (difficile)
Faites tirer au héros une flèche à 1000 pixels par seconde avec la barre d'espace. Sans raycast, montrez qu'elle traverse un mur de 10 pixels d'épaisseur. Corrigez avec un raycast le long du déplacement de chaque frame.

### Exercice 10 — Dégâts, invincibilité et Game Over (difficile)
Assemblez : héros à 3 vies, deux gobelins qui patrouillent, invincibilité de 1,5 seconde avec clignotement, HUD, et un overlay Flutter « Game Over » affiché quand les vies tombent à zéro. Le jeu doit se mettre en pause à ce moment-là.

---

## 32.35 — Corrections des exercices

### Correction 1

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex1>(game: Ex1()))));
}

class Ex1 extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;
    await world.add(Cible(position: Vector2(220, 160)));
    await world.add(Mobile(position: Vector2(0, 170)));
  }
}

class Cible extends PositionComponent with CollisionCallbacks {
  Cible({super.position}) : super(size: Vector2.all(60));
  static final Paint _p = Paint()..color = const Color(0xFF4CAF50);

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    debugPrint('DÉBUT : ${other.runtimeType}, ${points.length} point(s)');
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    super.onCollisionEnd(other);
    debugPrint('FIN   : ${other.runtimeType}');
  }
}

class Mobile extends PositionComponent with CollisionCallbacks {
  Mobile({super.position}) : super(size: Vector2.all(40));
  static final Paint _p = Paint()..color = const Color(0xFF6D4C41);

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void update(double dt) {
    super.update(dt);
    position.x += 90 * dt;
    if (position.x > 520) position.x = -40;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :**

```text
DÉBUT : Mobile, 2 point(s)
FIN   : Mobile
DÉBUT : Mobile, 2 point(s)
FIN   : Mobile
```

**Explication :** le mixin `HasCollisionDetection` sur `Ex1` lance la détection à chaque tick. Chaque composant porte `CollisionCallbacks` et une `RectangleHitbox()` sans argument, qui remplit son parent. `debugMode = true` posé sur le jeu descend dans tout l'arbre et affiche les contours. `onCollisionStart` et `onCollisionEnd` sont appelés une fois chacun par passage : le cycle se répète à chaque tour de boucle du carré marron.

---

### Correction 2

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex2>(game: Ex2()))));
}

class Ex2 extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;
    await world.add(Cible(position: Vector2(220, 150)));
    await world.add(Mobile(position: Vector2(0, 170)));
  }
}

class Cible extends PositionComponent with CollisionCallbacks {
  Cible({super.position}) : super(size: Vector2.all(60));
  static final Paint _p = Paint()..color = const Color(0xFF4CAF50);

  @override
  Future<void> onLoad() async {
    // 20 x 20 centrée dans un composant de 60 x 60.
    add(
      RectangleHitbox(
        size: Vector2.all(20),
        position: Vector2.all(30),
        anchor: Anchor.center,
      ),
    );
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    debugPrint('contact hitbox à x = ${other.position.x.toStringAsFixed(1)}');
  }
}

class Mobile extends PositionComponent with CollisionCallbacks {
  Mobile({super.position}) : super(size: Vector2.all(20));
  static final Paint _p = Paint()..color = const Color(0xFF6D4C41);

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void update(double dt) {
    super.update(dt);
    position.x += 90 * dt;
    if (position.x > 520) position.x = -20;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :**

```text
contact hitbox à x = 230.0
contact hitbox à x = 230.0
```

**Explication :** le carré vert occupe de 220 à 280 en X, mais sa hitbox n'occupe que de 250 à 270. Le contact commence donc quand le bord droit du mobile (largeur 20) atteint 250, c'est-à-dire quand sa position vaut 230, et non 200. En `debugMode`, le contour jaune de la hitbox est nettement plus petit que le carré vert : c'est exactement la séparation hitbox/visuel de la section 32.7.

---

### Correction 3

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex3>(game: Ex3()))));
}

class Ex3 extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;

    // A chevauche B ; B chevauche C.
    await world.add(
      Bloc('A-active', CollisionType.active,
          Vector2(60, 100), const Color(0xFF4CAF50)),
    );
    await world.add(
      Bloc('B-passive', CollisionType.passive,
          Vector2(100, 100), const Color(0xFF2196F3)),
    );
    await world.add(
      Bloc('C-passive', CollisionType.passive,
          Vector2(140, 100), const Color(0xFF9C27B0)),
    );
    await world.add(
      Bloc('D-inactive', CollisionType.inactive,
          Vector2(180, 100), const Color(0xFF9E9E9E)),
    );
  }
}

class Bloc extends PositionComponent with CollisionCallbacks {
  Bloc(this.nom, this.type, Vector2 pos, this.couleur)
      : super(position: pos, size: Vector2.all(60));

  final String nom;
  final CollisionType type;
  final Color couleur;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: type));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), Paint()..color = couleur.withAlpha(140));
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Bloc) debugPrint('$nom voit ${other.nom}');
  }
}
```

**Résultat :**

```text
A-active voit B-passive
B-passive voit A-active
```

**Explication :** seuls A et B se détectent, car il faut au moins un `active` dans le couple. B et C sont tous deux `passive` : ils se chevauchent pourtant, mais ne sont jamais testés l'un contre l'autre. D est `inactive` : il n'est testé contre personne, même contre A. Notez que les deux lignes de sortie illustrent le double dispatch de la section 32.17 : chaque composant du couple reçoit son propre appel.

---

### Correction 4

```dart
import 'dart:math';

import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex4>(game: Ex4()))));
}

class Ex4 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  late final TextComponent hud;
  int pieces = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final hasard = Random(7);
    for (var i = 0; i < 5; i++) {
      await world.add(
        Piece(
          position: Vector2(
            40 + hasard.nextDouble() * 400,
            40 + hasard.nextDouble() * 280,
          ),
        ),
      );
    }
    await world.add(Heros(position: Vector2(30, 30)));

    hud = TextComponent(
      text: 'Pièces : 0',
      position: Vector2(10, 10),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 18, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(hud);
  }

  @override
  void update(double dt) {
    super.update(dt);
    hud.text = 'Pièces : $pieces';
  }
}

class Piece extends PositionComponent with CollisionCallbacks {
  Piece({super.position}) : super(size: Vector2.all(16));
  static final Paint _p = Paint()..color = const Color(0xFFFFC107);
  bool _prise = false;

  @override
  Future<void> onLoad() async {
    add(CircleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawCircle((size / 2).toOffset(), size.x / 2, _p);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (_prise || other is! Heros) return;
    _prise = true;
    final jeu = findParent<FlameGame>();
    if (jeu is Ex4) jeu.pieces++;
    removeFromParent();
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _p = Paint()..color = const Color(0xFF4FC3F7);
  final Vector2 velocite = Vector2.zero();

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    velocite.setValues(
      (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0),
      (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(160);
    }
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    position += velocite * dt;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :**

```text
Pièces : 0
Pièces : 1
Pièces : 2
```

**Explication :** les pièces sont `passive`, le héros est `active` : le couple est testé. La règle est écrite d'un seul côté, celui de la pièce, ce qui évite le double comptage du double dispatch. Le drapeau `_prise` protège du second appel possible avant que `removeFromParent()` ne prenne effet à la fin de la frame. Le HUD est ajouté au `camera.viewport` pour rester fixe.

---
### Correction 5

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex5>(game: Ex5()))));
}

class Ex5 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.addAll([
      Mur(position: Vector2(0, 0), size: Vector2(480, 20)),
      Mur(position: Vector2(0, 300), size: Vector2(480, 20)),
      Mur(position: Vector2(0, 0), size: Vector2(20, 320)),
      Mur(position: Vector2(460, 0), size: Vector2(20, 320)),
      Mur(position: Vector2(220, 110), size: Vector2(40, 100)),
    ]);
    await world.add(Heros(position: Vector2(60, 60)));
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});
  static final Paint _p = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _p = Paint()..color = const Color(0xFF4FC3F7);
  final Vector2 velocite = Vector2.zero();

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    velocite.setValues(
      (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0),
      (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(170);
    }
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is! Mur) return;

    final moi = toAbsoluteRect();
    final lui = other.toAbsoluteRect();
    final c = moi.intersect(lui);
    if (c.width <= 0 || c.height <= 0) return;

    if (c.width < c.height) {
      position.x += moi.center.dx < lui.center.dx ? -c.width : c.width;
      velocite.x = 0;
    } else {
      position.y += moi.center.dy < lui.center.dy ? -c.height : c.height;
      velocite.y = 0;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :** le héros reste enfermé dans la salle et glisse le long des murs quand on avance en diagonale.

**Explication :** la résolution compare les deux dimensions du rectangle de chevauchement et corrige **le plus petit axe**, celui qui demande le moins de déplacement. Le mouvement sur l'autre axe est préservé : c'est ce qui produit le glissement. L'annulation de la composante de vitesse (`velocite.x = 0`) empêche le héros de repousser dans le mur à la frame suivante et supprime la vibration. Les murs sont `passive` : ils n'ont pas besoin de se tester entre eux.

---

### Correction 6

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex6>(game: Ex6()))));
}

class Ex6 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    // Un couloir pavé de tuiles de 32 px : le cas qui révèle l'accrochage.
    for (var i = 0; i < 14; i++) {
      await world.add(Mur(position: Vector2(i * 32.0, 240)));
      await world.add(Mur(position: Vector2(i * 32.0, 40)));
    }
    await world.add(Heros(position: Vector2(60, 120)));
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position}) : super(size: Vector2.all(32));
  static final Paint _p = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _p = Paint()..color = const Color(0xFF4FC3F7);
  final Vector2 velocite = Vector2.zero();
  final Vector2 _precedente = Vector2.zero();

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    velocite.setValues(
      (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0),
      (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(190);
    }
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    _precedente.setFrom(position);
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is! Mur) return;

    final lui = other.toAbsoluteRect();
    final moi = toAbsoluteRect();
    final c = moi.intersect(lui);
    if (c.width <= 0 || c.height <= 0) return;

    final avant = Rect.fromLTWH(_precedente.x, _precedente.y, size.x, size.y);
    final dejaY = avant.top < lui.bottom && avant.bottom > lui.top;
    final dejaX = avant.left < lui.right && avant.right > lui.left;

    if (dejaY && !dejaX) {
      position.x = velocite.x > 0 ? lui.left - size.x : lui.right;
      velocite.x = 0;
    } else if (dejaX && !dejaY) {
      position.y = velocite.y > 0 ? lui.top - size.y : lui.bottom;
      velocite.y = 0;
    } else if (c.width < c.height) {
      position.x += moi.center.dx < lui.center.dx ? -c.width : c.width;
      velocite.x = 0;
    } else {
      position.y += moi.center.dy < lui.center.dy ? -c.height : c.height;
      velocite.y = 0;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :** le héros longe la rangée de tuiles sans jamais s'arrêter sur une jointure.

**Explication :** le rectangle `avant` reconstitue la position occupée avant le déplacement de la frame. Si le héros chevauchait déjà la tuile en Y sans la chevaucher en X, c'est qu'il est entré **par le côté** : seule la coordonnée X est corrigée, le mouvement vertical est conservé. Avec la règle du plus petit dégagement seule (correction 5), une jointure entre deux tuiles pouvait produire une correction verticale parasite qui bloquait la course. L'affectation absolue (`position.x = lui.left - size.x`) rend la correction idempotente : deux tuiles voisines qui déclenchent la callback dans la même frame ne cumulent pas leurs corrections.

---

### Correction 7

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex7>(game: Ex7()))));
}

class Ex7 extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;

    // Zone A : creuse (isSolid: false) -> ne détecte pas la pièce au centre.
    await world.add(Zone(nom: 'creuse', solide: false,
        position: Vector2(30, 40)));
    await world.add(Piece(position: Vector2(105, 95)));

    // Zone B : solide -> détecte la pièce au centre.
    await world.add(Zone(nom: 'solide', solide: true,
        position: Vector2(240, 40)));
    await world.add(Piece(position: Vector2(315, 95)));
  }
}

class Zone extends PositionComponent with CollisionCallbacks {
  Zone({required this.nom, required this.solide, super.position})
      : super(size: Vector2(160, 120));

  final String nom;
  final bool solide;
  static final Paint _p = Paint()
    ..color = const Color(0x554DD0E1)
    ..style = PaintingStyle.fill;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(isSolid: solide));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    super.onCollisionStart(points, other);
    if (other is Piece) {
      debugPrint('zone $nom : détectée, ${points.length} point(s)');
    }
  }
}

class Piece extends PositionComponent with CollisionCallbacks {
  Piece({super.position}) : super(size: Vector2.all(10));
  static final Paint _p = Paint()..color = const Color(0xFFFFC107);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :**

```text
zone solide : détectée, 1 point(s)
```

**Explication :** la zone creuse ne produit aucune ligne. La pièce de 10 x 10 est entièrement contenue dans la zone de 160 x 120 : leurs contours ne se croisent nulle part, donc `intersectionPoints` est vide, donc Flame conclut qu'il n'y a pas de collision. Avec `isSolid: true` sur la **zone** (le contenant), Flame accepte la contenance totale et renvoie un unique point : le centre de la forme contenue. C'est le seul cas où l'on peut prédire le nombre de points d'intersection.

---

### Correction 8

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/geometry.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex8>(game: Ex8()))));
}

class Ex8 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  late final Heros heros;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;

    await world.add(Mur(position: Vector2(230, 60), size: Vector2(24, 200)));
    heros = Heros(position: Vector2(60, 150));
    await world.add(heros);
    await world.add(Gobelin(position: Vector2(400, 150)));
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});
  static final Paint _p = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _p = Paint()..color = const Color(0xFF4FC3F7);
  final Vector2 velocite = Vector2.zero();

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    velocite.setValues(
      (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0),
      (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(150);
    }
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    position += velocite * dt;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Gobelin extends PositionComponent
    with CollisionCallbacks, HasGameReference<Ex8> {
  Gobelin({super.position}) : super(size: Vector2(26, 30));
  static final Paint _p = Paint()..color = const Color(0xFF8BC34A);
  static final Paint _ligne = Paint()
    ..color = const Color(0xFFFF5252)
    ..strokeWidth = 1;

  late final RectangleHitbox hitbox;
  double portee = 300;
  double vitesse = 60;
  bool voit = false;
  double _cumul = 0;

  @override
  Future<void> onLoad() async {
    hitbox = RectangleHitbox();
    add(hitbox);
  }

  bool _ligneDeVue() {
    final origine = absoluteCenter;
    final vers = game.heros.absoluteCenter - origine;
    final distance = vers.length;
    if (distance > portee || distance < 0.001) return false;

    final r = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: vers),
      maxDistance: distance,
      ignoreHitboxes: [hitbox],
    );
    return r?.hitbox?.hitboxParent is Heros;
  }

  @override
  void update(double dt) {
    super.update(dt);
    _cumul += dt;
    if (_cumul >= 1 / 6) {
      _cumul = 0;
      voit = _ligneDeVue();
    }
    if (voit) {
      final vers = game.heros.absoluteCenter - absoluteCenter;
      if (vers.length2 > 1) position += vers.normalized() * vitesse * dt;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
    if (debugMode && voit) {
      final vers = game.heros.absoluteCenter - absoluteCenter;
      canvas.drawLine(
        (size / 2).toOffset(),
        ((size / 2) + vers).toOffset(),
        _ligne,
      );
    }
  }
}
```

**Résultat :** le gobelin avance vers le héros à découvert, et s'immobilise dès que le mur s'interpose ; le trait rouge n'apparaît que lorsque la vue est dégagée.

**Explication :** le rayon part du centre du gobelin en direction du héros, limité à la distance qui les sépare. `ignoreHitboxes: [hitbox]` est indispensable : sans lui, la première hitbox rencontrée serait celle du gobelin lui-même et le test échouerait toujours. `hitboxParent` remonte de la hitbox au composant, ce qui permet le test `is Heros`. Le rayon n'est relancé que six fois par seconde : une IA n'a pas besoin de la précision d'une frame, et 60 raycasts par seconde et par ennemi coûtent cher pour rien.

---

### Correction 9

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/geometry.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: GameWidget<Ex9>(game: Ex9()))));
}

class Ex9 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    debugMode = true;
    await world.add(Mur(position: Vector2(320, 60), size: Vector2(10, 220)));
    await world.add(Heros(position: Vector2(50, 150)));
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});
  static final Paint _p = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _p = Paint()..color = const Color(0xFF4FC3F7);

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (event is KeyDownEvent &&
        event.logicalKey == LogicalKeyboardKey.space) {
      parent?.add(
        Fleche(
          position: absoluteCenter.clone(),
          direction: Vector2(1, 0),
        ),
      );
      return false;
    }
    return true;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Fleche extends PositionComponent
    with CollisionCallbacks, HasGameReference<Ex9> {
  Fleche({super.position, required this.direction})
      : super(size: Vector2(12, 4), anchor: Anchor.center);

  static final Paint _p = Paint()..color = const Color(0xFFFFF176);
  final Vector2 direction;
  double vitesse = 1000;
  late final RectangleHitbox hitbox;

  @override
  Future<void> onLoad() async {
    hitbox = RectangleHitbox();
    add(hitbox);
  }

  @override
  void update(double dt) {
    super.update(dt);
    final distance = vitesse * dt;

    final r = game.collisionDetection.raycast(
      Ray2(origin: absoluteCenter, direction: direction),
      maxDistance: distance,
      ignoreHitboxes: [hitbox],
    );

    if (r?.hitbox?.hitboxParent is Mur) {
      position.setFrom(r!.intersectionPoint!);
      debugPrint('impact à ${position.x.toStringAsFixed(1)}');
      removeFromParent();
      return;
    }

    position += direction * distance;
    if (position.x > 600) removeFromParent();
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}
```

**Résultat :**

```text
impact à 320.0
```

**Explication :** à 1000 px/s et 60 images par seconde, la flèche parcourt environ 16,7 pixels par tick, soit bien plus que les 10 pixels d'épaisseur du mur : sans raycast, elle sautait par-dessus une frame sur deux, et l'impact n'était détecté qu'au hasard. Le rayon, lui, teste **tout le segment** parcouru pendant la frame : aucun obstacle ne peut passer entre deux positions. On place ensuite la flèche exactement au point d'impact avant de la retirer, ce qui permettra d'y faire naître une gerbe d'étincelles au chapitre 33. La flèche conserve tout de même une hitbox afin que les autres objets puissent la détecter normalement.

---

### Correction 10

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(
    MaterialApp(
      home: Scaffold(
        body: GameWidget<Ex10>(
          game: Ex10(),
          overlayBuilderMap: {
            'GameOver': (context, game) => Center(
                  child: Container(
                    padding: const EdgeInsets.all(24),
                    color: const Color(0xCC000000),
                    child: const Text(
                      'GAME OVER',
                      style: TextStyle(fontSize: 32, color: Colors.white),
                    ),
                  ),
                ),
          },
        ),
      ),
    ),
  );
}

class Ex10 extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  late final Heros heros;
  late final TextComponent hud;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.addAll([
      Mur(position: Vector2(0, 0), size: Vector2(480, 20)),
      Mur(position: Vector2(0, 300), size: Vector2(480, 20)),
      Mur(position: Vector2(0, 0), size: Vector2(20, 320)),
      Mur(position: Vector2(460, 0), size: Vector2(20, 320)),
    ]);
    await world.add(Gobelin(position: Vector2(200, 80), vitesseX: 90));
    await world.add(Gobelin(position: Vector2(300, 220), vitesseX: -70));

    heros = Heros(position: Vector2(60, 150));
    await world.add(heros);

    hud = TextComponent(
      text: '',
      position: Vector2(10, 10),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 18, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(hud);
  }

  @override
  void update(double dt) {
    super.update(dt);
    hud.text = 'Vies : ${heros.vies}';
    if (heros.vies <= 0 && !overlays.isActive('GameOver')) {
      overlays.add('GameOver');
      pauseEngine();
    }
  }
}

class Mur extends PositionComponent with CollisionCallbacks {
  Mur({super.position, super.size});
  static final Paint _p = Paint()..color = const Color(0xFF5D4037);

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Gobelin extends PositionComponent with CollisionCallbacks {
  Gobelin({super.position, required this.vitesseX})
      : super(size: Vector2(26, 30));
  static final Paint _p = Paint()..color = const Color(0xFF8BC34A);
  double vitesseX;

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  void update(double dt) {
    super.update(dt);
    position.x += vitesseX * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is! Mur) return;
    final moi = toAbsoluteRect();
    final lui = other.toAbsoluteRect();
    final c = moi.intersect(lui);
    if (c.width <= 0) return;
    position.x += moi.center.dx < lui.center.dx ? -c.width : c.width;
    vitesseX = -vitesseX;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _p);
  }
}

class Heros extends PositionComponent with CollisionCallbacks, KeyboardHandler {
  Heros({super.position}) : super(size: Vector2.all(26));
  static final Paint _normal = Paint()..color = const Color(0xFF4FC3F7);
  static final Paint _blesse = Paint()..color = const Color(0xFFEF5350);

  final Vector2 velocite = Vector2.zero();
  int vies = 3;
  double _invincible = 0;
  static const double duree = 1.5;

  bool get estInvincible => _invincible > 0;

  @override
  Future<void> onLoad() async => add(RectangleHitbox());

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    velocite.setValues(
      (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0),
      (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0),
    );
    if (velocite.length2 > 0) {
      velocite.normalize();
      velocite.scale(160);
    }
    return true;
  }

  void subirDegats(int montant) {
    if (estInvincible) return;
    vies -= montant;
    _invincible = duree;
    if (vies < 0) vies = 0;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_invincible > 0) {
      _invincible -= dt;
      if (_invincible < 0) _invincible = 0;
    }
    position += velocite * dt;
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);

    if (other is Gobelin) {
      subirDegats(1);
      return;
    }
    if (other is! Mur) return;

    final moi = toAbsoluteRect();
    final lui = other.toAbsoluteRect();
    final c = moi.intersect(lui);
    if (c.width <= 0 || c.height <= 0) return;
    if (c.width < c.height) {
      position.x += moi.center.dx < lui.center.dx ? -c.width : c.width;
      velocite.x = 0;
    } else {
      position.y += moi.center.dy < lui.center.dy ? -c.height : c.height;
      velocite.y = 0;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    if (estInvincible && (_invincible * 12).floor().isEven) return;
    canvas.drawRect(size.toRect(), estInvincible ? _blesse : _normal);
  }
}
```

**Résultat :**

```text
Vies : 3
Vies : 2      (premier contact, puis clignotement pendant 1,5 s)
Vies : 1
Vies : 0      -> overlay « GAME OVER » et moteur en pause
```

**Explication :** toute la logique de filtrage des dégâts est concentrée dans `subirDegats`, qui refuse net tant que `_invincible` est positif. On peut donc appeler cette méthode depuis `onCollision`, c'est-à-dire à chaque tick, sans risque de vider la barre de vie : le contact prolongé ne fera perdre une vie que toutes les 1,5 seconde. Le clignotement se fait par un simple `return` avant le dessin, une frame sur deux, ce qui ne coûte rien. Le passage en Game Over est géré dans `update` du jeu : `overlays.add('GameOver')` affiche le widget Flutter déclaré dans `overlayBuilderMap`, et `pauseEngine()` fige la boucle de jeu. Le test `!overlays.isActive('GameOver')` évite d'ajouter l'overlay à chaque frame.

---

## Et maintenant ?

Votre donjon réagit. Le héros bute contre la pierre, ramasse l'or, encaisse les coups des gobelins et clignote pendant qu'il récupère. Vous savez désormais où s'arrête le moteur et où commence votre code : Flame vous dit **qu'il y a contact**, vous décidez **ce que cela signifie**.

Il manque encore tout ce qui rend un contact satisfaisant. Une pièce ramassée devrait grandir puis disparaître dans une gerbe d'étincelles. Un héros touché devrait être projeté en arrière et sa barre de vie devrait vibrer. Une porte devrait s'ouvrir en glissant, pas changer de couleur d'un seul coup. Un gobelin devrait apparaître toutes les trois secondes sans que vous ayez à compter les frames à la main.

Le chapitre suivant apporte ces outils : les effets (`MoveEffect`, `ScaleEffect`, `OpacityEffect`, `SequenceEffect`) pilotés par un `EffectController`, les systèmes de particules avec `ParticleSystemComponent`, et la temporisation propre avec `Timer` et `TimerComponent`. Vous y remplacerez notamment le clignotement bricolé de la section 32.28 par sa version élégante.

Rendez-vous au chapitre 33 : [33-PARTIE-2B—EFFETS-PARTICULES-ET-TIMERS.md](./33-PARTIE-2B—EFFETS-PARTICULES-ET-TIMERS.md)
