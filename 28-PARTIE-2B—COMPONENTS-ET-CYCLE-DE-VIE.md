# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 28 — LES COMPONENTS ET LEUR CYCLE DE VIE

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitre 27 (installer Flame, `FlameGame`, `GameWidget`), chapitre 26 (entités, `priority`, files d'ajout), et les chapitres 8 à 12 de la PARTIE 1A (POO, héritage, mixins, null safety)
> **Ce que vous saurez faire à la fin :** construire un jeu entier sous forme d'arbre de composants, maîtriser leur cycle de vie, leurs coordonnées et leur ordre de rendu.
> **Version de Flame utilisée dans ce chapitre :** `flame: ^1.38.0`

---

## 28.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer la phrase « dans Flame, tout est composant » et ce qu'elle implique ;
- dessiner l'**arbre de composants** d'un jeu et nommer chacun de ses nœuds ;
- comparer l'arbre de Flame avec la `List<Entity>` que vous aviez écrite au chapitre 26 ;
- distinguer `Component` et `PositionComponent`, et choisir le bon dans chaque cas ;
- manipuler `position`, `size`, `scale`, `angle` et `anchor` sans vous tromper d'unité ;
- expliquer le piège de l'ancre par défaut `Anchor.topLeft` et le corriger ;
- passer d'un repère **local** à un repère **absolu** et inversement ;
- utiliser `absolutePosition`, `positionOf`, `absolutePositionOf` et `toAbsoluteRect` ;
- ajouter des composants avec `add()` et `addAll()` ;
- expliquer **pourquoi `add()` est asynchrone** et ce que cela change dans votre code ;
- écrire correctement `await add(...)` dans un `onLoad()` ;
- retirer un composant avec `remove()`, `removeFromParent()`, `removeAll()` et `removeWhere()` ;
- réciter le cycle de vie complet : `onLoad` → `onGameResize` → `onMount` → `update`/`render` → `onRemove` ;
- utiliser `onGameResize` pour repositionner un HUD ;
- attendre `loaded` et `mounted`, et tester `isLoaded` / `isMounted` ;
- naviguer dans l'arbre avec `parent`, `children`, `descendants()` et `ancestors()` ;
- composer un personnage complet : corps, barre de vie et étiquette de nom ;
- prévoir l'effet des transformations héritées d'un parent sur ses enfants ;
- régler l'ordre de rendu avec `priority`, y compris en cours de partie ;
- utiliser `RectangleComponent`, `CircleComponent`, `PolygonComponent` et `TextComponent` ;
- styliser du texte avec `TextPaint` ;
- écrire votre propre composant `class Joueur extends PositionComponent` ;
- accéder au jeu depuis un composant grâce à `HasGameReference<T>` ;
- reconnaître et corriger le code périmé qui utilise `HasGameRef` et `gameRef` ;
- interroger l'arbre avec `children.query<T>()`, `firstChild<T>()` et `whereType<T>()` ;
- réagir à un changement d'état avec `ComponentsNotifier` ;
- expliquer le rôle du `World` et pourquoi vos entités y vivent ;
- assembler un mini-donjon complet : un joueur, trois gobelins et un coffre, tout en composants.

---

## 28.1 — Tout est composant

Au chapitre 26, vous avez écrit à la main un moteur d'entités. Vous aviez une classe abstraite `Entity`, une `List<Entity>`, une boucle qui appelait `update(dt)` sur chaque élément puis `render(canvas)` sur chaque élément, et un champ `priority` pour trier avant de dessiner.

Flame part de la même idée, mais la pousse jusqu'au bout. Dans Flame, il n'existe **qu'un seul type d'objet manipulable** : le composant, représenté par la classe `Component`. Et absolument tout en est un.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │            CE QUI EST UN COMPOSANT DANS FLAME                 │
  └──────────────────────────────────────────────────────────────┘

  Le joueur                     -> un composant
  Un gobelin                    -> un composant
  Le coffre                     -> un composant
  La barre de vie du joueur     -> un composant
  Le texte du score             -> un composant
  Le décor du fond              -> un composant
  La CAMÉRA elle-même           -> un composant (CameraComponent)
  Le MONDE lui-même             -> un composant (World)
  Le viewport de la caméra      -> un composant (Viewport)
  Un effet de déplacement       -> un composant (MoveEffect)
  Une hitbox de collision       -> un composant (RectangleHitbox)
  Un minuteur                   -> un composant (TimerComponent)
  Un système de particules      -> un composant
  Un « gestionnaire d'ennemis » -> un composant sans apparence
```

Cette uniformité est le cœur du moteur. Elle a trois conséquences très concrètes.

**Première conséquence : une seule API à apprendre.** Que vous manipuliez un gobelin, une caméra ou un minuteur, vous utilisez les mêmes méthodes : `add`, `remove`, `onLoad`, `update`, `render`, `priority`. Il n'y a pas une façon d'ajouter une entité et une autre façon d'ajouter un effet.

**Deuxième conséquence : la composition remplace l'héritage.** Au chapitre 26, vous avez constaté qu'une hiérarchie `Entity → Character → Player` finit par coincer. Ici, un personnage n'hérite pas d'un « personnage avec barre de vie » : il **contient** un composant barre de vie. Vous ajoutez ou retirez une capacité en ajoutant ou retirant un enfant.

**Troisième conséquence : tout se range en arbre.** Un composant peut contenir d'autres composants, qui peuvent eux-mêmes en contenir d'autres. C'est l'objet de la section suivante.

> **À retenir.** Retenez la question réflexe du développeur Flame : « quel composant représente cette chose ? ». Si vous vous surprenez à écrire une `List<Gobelin>` que vous parcourez vous-même dans `update`, c'est presque toujours le signe que vous n'avez pas encore accepté l'arbre.

---

## 28.2 — L'arbre de composants

Chaque composant possède au plus **un parent** et un nombre quelconque **d'enfants**. Cette relation forme un arbre unique dont la racine est le jeu lui-même.

Voici l'arbre réel d'un `FlameGame` du « Donjon de Dart », tel qu'il existe en mémoire pendant la partie.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │             L'ARBRE DE COMPOSANTS DU DONJON DE DART              │
  └──────────────────────────────────────────────────────────────────┘

  DonjonDeDart                       (FlameGame : la racine)
  │
  ├── World                          (game.world : le monde du jeu)
  │   │
  │   ├── Sol                        (RectangleComponent gris)
  │   │
  │   ├── Joueur                     (PositionComponent)
  │   │   ├── Corps                  (RectangleComponent doré)
  │   │   ├── BarreDeVie             (PositionComponent)
  │   │   │   ├── Fond               (RectangleComponent rouge)
  │   │   │   └── Remplissage        (RectangleComponent vert)
  │   │   └── Etiquette              (TextComponent « Héros »)
  │   │
  │   ├── Gobelin                    (PositionComponent)
  │   │   ├── Corps                  (RectangleComponent vert)
  │   │   └── Oeil                   (CircleComponent noir)
  │   │
  │   ├── Gobelin                    (deuxième instance)
  │   ├── Gobelin                    (troisième instance)
  │   │
  │   └── Coffre                     (PositionComponent)
  │       ├── Caisse                 (RectangleComponent brun)
  │       └── Serrure                (RectangleComponent jaune)
  │
  └── CameraComponent                (game.camera)
      │
      ├── Viewfinder                 (zoom, angle, centrage)
      ├── Viewport                   (fenêtre d'affichage)
      │   ├── TexteScore             (TextComponent, HUD fixe)
      │   └── TexteVies              (TextComponent, HUD fixe)
      └── backdrop                   (décor derrière le monde)
```

Trois lectures de ce schéma méritent d'être faites lentement.

**Lecture verticale : la propagation.** Quand Flame appelle `update(dt)` sur `DonjonDeDart`, celui-ci appelle `update(dt)` sur ses enfants, qui appellent `update(dt)` sur les leurs, et ainsi de suite jusqu'aux feuilles. Un seul appel initial déclenche tout l'arbre. C'est très exactement ce que faisait votre boucle `for (final e in entites) e.update(dt);` du chapitre 26, mais récursivement.

**Lecture horizontale : le regroupement.** `BarreDeVie` n'est pas un enfant du monde, c'est un enfant du `Joueur`. Conséquence immédiate : quand le joueur se déplace, sa barre de vie le suit **sans une ligne de code**. Quand le joueur est retiré de l'arbre, sa barre de vie et son étiquette disparaissent avec lui. C'est le bénéfice le plus rentable de l'arbre.

**Lecture par branches : la séparation monde / interface.** Tout ce qui vit dans le `World` est soumis à la caméra : il se déplace, zoome, tourne avec elle. Tout ce qui vit dans le `Viewport` reste collé à l'écran. Le score ne bouge pas quand le héros avance. Cette séparation, vous l'aviez codée à la main au chapitre 25 avec `canvas.save()` et `canvas.translate()` ; ici, il suffit de choisir la bonne branche.

> **À retenir.** L'arbre n'est pas une structure de rangement décorative. C'est la structure qui décide **qui bouge avec qui**, **qui disparaît avec qui**, et **dans quel ordre tout est dessiné**.

---

## 28.3 — Comparaison avec la liste d'entités du chapitre 26

Mettons les deux modèles côte à côte. Voici ce que vous aviez écrit au chapitre 26, en version condensée.

```dart
// PARTIE 2A — votre moteur écrit à la main (chapitre 26).
abstract class Entity {
  double x = 0;
  double y = 0;
  int priority = 0;
  bool aRetirer = false;

  void update(double dt);
  void render(Canvas canvas);
}

class Moteur {
  final List<Entity> entites = [];
  final List<Entity> aAjouter = [];

  void update(double dt) {
    entites.addAll(aAjouter);
    aAjouter.clear();
    for (final e in entites) {
      e.update(dt);
    }
    entites.removeWhere((e) => e.aRetirer);
  }

  void render(Canvas canvas) {
    final tri = [...entites]..sort((a, b) => a.priority.compareTo(b.priority));
    for (final e in tri) {
      canvas.save();
      canvas.translate(e.x, e.y);
      e.render(canvas);
      canvas.restore();
    }
  }
}
```

Et voici l'équivalent sous Flame.

```dart
// PARTIE 2B — la même chose avec Flame.
class Moteur extends World {
  // C'est tout. Il n'y a rien d'autre à écrire.
}
```

Le tableau suivant fait la correspondance ligne à ligne. Gardez-le sous les yeux pendant tout le chapitre : il transforme des connaissances que vous avez déjà en connaissances Flame.

| Chapitre 26, écrit à la main | Flame 1.38.0 | Remarque |
| --- | --- | --- |
| `abstract class Entity` | `class Component` | Flame la fournit, elle n'est pas abstraite |
| `List<Entity> entites` | `parent.children` | un `ReadOnlyOrderedSet<Component>` trié |
| `entites.add(e)` | `parent.add(e)` | asynchrone, voir 28.11 |
| `List<Entity> aAjouter` | interne à Flame | la file d'ajout existe toujours, mais elle est cachée |
| `e.aRetirer = true` | `e.removeFromParent()` | même principe de file différée |
| `entites.removeWhere(...)` | `parent.removeWhere(...)` | même nom, même idée |
| `for (final e in entites) e.update(dt)` | `super.update(dt)` | la propagation est automatique |
| `..sort((a, b) => a.priority...)` | `priority` | le tri est maintenu en permanence |
| `canvas.save(); canvas.translate(e.x, e.y);` | `position` | la transformation est appliquée par Flame |
| `double x; double y;` | `Vector2 position` | un seul objet, pas deux champs |
| `Exception: Concurrent modification` | impossible | Flame applique les changements entre deux frames |
| une entité contenue dans une autre | `parent` / `children` | l'arbre est natif |

Deux points méritent d'être soulignés, parce qu'ils sont la vraie valeur ajoutée de Flame.

**Le bug de modification concurrente disparaît.** Au chapitre 26, vous aviez rencontré l'exception `Concurrent modification during iteration` en supprimant un gobelin pendant la boucle `update`. Vous l'aviez corrigée avec deux files. Flame utilise exactement la même technique en interne : `add()` et `remove()` ne modifient jamais l'arbre pendant qu'il est parcouru. Le bug ne peut plus se produire.

**L'arbre remplace la liste plate.** Votre `List<Entity>` était plate : une barre de vie devait connaître son propriétaire et recopier sa position à chaque frame. Dans un arbre, elle est simplement son enfant.

> **À retenir.** Vous ne découvrez pas un moteur inconnu. Vous découvrez **votre propre moteur du chapitre 26, en version complète et testée**. C'est pour cette raison que la PARTIE 2A n'était pas une perte de temps.

---

## 28.4 — La classe `Component`

`Component` est la classe de base de tout l'arbre. Elle est **concrète** : vous pouvez en instancier une directement.

Un point capital, et souvent mal compris : **`Component` n'a ni position, ni taille, ni apparence**. Elle ne sait pas se dessiner. Elle sait seulement trois choses : avoir un parent, avoir des enfants, et recevoir les appels du cycle de vie.

Cela en fait l'outil idéal pour trois usages.

**Usage 1 : un conteneur logique de rangement.**

```dart
import 'package:flame/components.dart';

// Un simple regroupement : tous les ennemis au même endroit dans l'arbre.
class GroupeEnnemis extends Component {}
```

**Usage 2 : un contrôleur invisible qui pilote le jeu.**

```dart
import 'package:flame/components.dart';

// Ce composant n'affiche rien. Il fait apparaître un gobelin toutes les 3 s.
class GenerateurDeGobelins extends Component {
  double _temps = 0;
  int _crees = 0;

  @override
  void update(double dt) {
    super.update(dt);
    _temps += dt;
    if (_temps >= 3 && _crees < 5) {
      _temps = 0;
      _crees++;
      print('Gobelin numéro $_crees invoqué.');
    }
  }
}
```

**Usage 3 : la racine d'un sous-arbre, avec des enfants fournis au constructeur.**

```dart
import 'package:flame/components.dart';

final racine = Component(
  children: [
    Component(),
    Component(),
  ],
);

// On peut aussi en ajouter après coup.
racine.addAll([Component(), Component()]);
```

Voici l'essentiel de la surface publique de `Component`, telle qu'elle existe en 1.38.0.

| Membre | Type / signature | Rôle |
| --- | --- | --- |
| `parent` | `Component?` | le parent, ou `null` si le composant n'est pas monté |
| `children` | `ReadOnlyOrderedSet<Component>` | les enfants directs, maintenus triés par `priority` |
| `priority` | `int` | ordre de rendu, modifiable à tout moment |
| `key` | `ComponentKey?` | identifiant facultatif, fixé au constructeur |
| `add(Component)` | `FutureOr<void>` | ajoute un enfant |
| `addAll(Iterable<Component>)` | `Future<void>` | ajoute plusieurs enfants |
| `remove(Component)` | `void` | retire un enfant précis |
| `removeAll(Iterable<Component>)` | `void` | retire plusieurs enfants |
| `removeWhere(bool Function(Component))` | `void` | retire les enfants qui satisfont un test |
| `removeFromParent()` | `void` | se retire soi-même de son parent |
| `onLoad()` | `FutureOr<void>` | initialisation, une seule fois |
| `onMount()` | `void` | à chaque montage dans l'arbre |
| `onRemove()` | `void` | nettoyage, une seule fois |
| `onGameResize(Vector2)` | `void` | au redimensionnement et avant le montage |
| `update(double dt)` | `void` | logique par frame, `dt` en secondes |
| `render(Canvas)` | `void` | dessin par frame |
| `isLoaded` / `isMounted` / `isRemoved` | `bool` | état courant |
| `loaded` / `mounted` / `removed` | `Future<void>` | attente de l'état |
| `findGame()` | `FlameGame<World>?` | remonte jusqu'au jeu |
| `descendants()` | `Iterable<Component>` | tous les descendants |
| `ancestors()` | `Iterable<Component>` | tous les ancêtres |

Vous n'avez pas à mémoriser cette table. Vous y reviendrez : c'est la carte du chapitre.

---

## 28.5 — `PositionComponent`

Dès qu'un composant doit **apparaître quelque part**, `Component` ne suffit plus. On utilise `PositionComponent`, qui ajoute tout ce qui concerne la géométrie.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                 LA CHAÎNE D'HÉRITAGE UTILE                   │
  └──────────────────────────────────────────────────────────────┘

  Component                       parent, enfants, cycle de vie
      │
      └── PositionComponent       position, size, scale, angle, anchor
              │
              ├── ShapeComponent          + paint
              │       ├── PolygonComponent
              │       │       └── RectangleComponent
              │       └── CircleComponent
              │
              ├── TextComponent           + text, textRenderer
              ├── SpriteComponent         + sprite      (chapitre 29)
              └── SpriteAnimationComponent + animation   (chapitre 29)
```

Voici le constructeur réel de `PositionComponent` en 1.38.0.

```dart
PositionComponent({
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  double nativeAngle = 0,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
  ComponentKey? key,
});
```

Un premier exemple complet et exécutable. Il n'utilise aucune image : un carré doré représente le héros.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le repère du monde démarre en haut à gauche : plus lisible pour débuter.
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(Heros());
  }
}

class Heros extends PositionComponent {
  Heros()
      : super(
          position: Vector2(120, 90),
          size: Vector2(48, 48),
        );

  static final Paint _or = Paint()..color = const Color(0xFFE8B04B);

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    // On dessine en coordonnées LOCALES : le coin (0, 0) est celui du composant.
    canvas.drawRect(size.toRect(), _or);
  }
}
```

**Résultat :**

```text
  Fenêtre du jeu
  ┌────────────────────────────────────────────┐
  │                                            │
  │                                            │
  │            ████████                        │
  │            ████████   <- carré doré 48x48  │
  │            ████████      coin haut-gauche  │
  │            ████████      en (120, 90)      │
  │                                            │
  └────────────────────────────────────────────┘
```

Deux remarques importantes sur ce code.

**`size.toRect()` produit `Rect.fromLTWH(0, 0, size.x, size.y)`.** On dessine bien à partir de zéro, pas à partir de `position`. Flame a déjà déplacé le `Canvas` avant d'appeler `render`.

**Ne jamais écrire `canvas.drawRect(Rect.fromLTWH(position.x, position.y, ...))`.** C'est l'erreur numéro un des débutants venant du chapitre 21 : la position serait appliquée deux fois, et le héros apparaîtrait en (240, 180).

> **À retenir.** `PositionComponent` s'occupe du **où**. Votre `render` s'occupe du **quoi**, en coordonnées locales, comme si le composant était seul au monde et posé à l'origine.

---

## 28.6 — `position`, `size`, `scale`, `angle`, `anchor`

Ces cinq propriétés décrivent entièrement la géométrie d'un `PositionComponent`. Prenons-les une par une, avec leurs unités et leurs pièges.

### `position` — un `Vector2`, exprimé dans le repère du parent

```dart
final gobelin = PositionComponent(position: Vector2(200, 140));

gobelin.position = Vector2(50, 50);   // remplacement complet
gobelin.position.x += 10;              // déplacement horizontal
gobelin.x -= 5;                        // raccourci équivalent à position.x
gobelin.position += Vector2(0, 3);     // addition vectorielle
```

`position` est de type `NotifyingVector2`, un sous-type de `Vector2`. Il est **mutable** : le modifier en place fonctionne et c'est même la façon la plus efficace de le faire.

Attention au piège vu au chapitre 23 : partager une même instance de `Vector2` entre deux composants les lie définitivement.

```dart
// PIÈGE : les deux gobelins partagent le MÊME vecteur.
final depart = Vector2(100, 100);
final g1 = PositionComponent(position: depart);
final g2 = PositionComponent(position: depart);
// Déplacer g1 déplacerait g2. (En pratique Flame recopie, mais ne comptez pas dessus.)

// CORRECT : chacun le sien.
final g3 = PositionComponent(position: depart.clone());
final g4 = PositionComponent(position: depart.clone());
```

### `size` — un `Vector2`, en pixels logiques, à l'échelle 1

```dart
final coffre = PositionComponent(size: Vector2(40, 30));

print(coffre.width);   // 40.0  -> raccourci de size.x
print(coffre.height);  // 30.0  -> raccourci de size.y
```

**Valeur par défaut : `Vector2.zero()`.** Un composant de taille nulle est invisible pour la détection de tap et pour les hitbox. C'est la cause la plus fréquente d'un bouton qui « ne réagit pas ».

### `scale` — un `Vector2`, facteur multiplicatif autour de l'ancre

```dart
final gobelin = PositionComponent(size: Vector2(32, 32));

gobelin.scale = Vector2.all(2);       // deux fois plus grand
gobelin.scale = Vector2(-1, 1);       // retourné horizontalement
gobelin.scale = Vector2(1.5, 0.5);    // aplati

print(gobelin.size);        // [32.0,32.0] -> size ne change PAS
print(gobelin.scaledSize);  // [48.0,16.0] -> taille réelle à l'écran
```

Retenez la distinction : `size` est la taille logique, `scaledSize` est la taille affichée. Un `scale` négatif retourne le composant, ce que font aussi `flipHorizontally()` et `flipVertically()`.

### `angle` — un `double`, **en radians**, autour de l'ancre

```dart
import 'dart:math';

final piece = PositionComponent();

piece.angle = pi / 2;   // 90 degrés
piece.angle = pi;       // 180 degrés
piece.angle += 0.02;    // rotation continue dans update

// Conversion si vous raisonnez en degrés :
double degresEnRadians(double d) => d * pi / 180;
piece.angle = degresEnRadians(45);
```

Écrire `angle = 45` fait tourner le composant de 45 radians, soit environ 2578 degrés. C'est un classique.

### `anchor` — un `Anchor`, point de référence du composant

```dart
final gobelin = PositionComponent(
  position: Vector2(200, 140),
  size: Vector2(32, 32),
  anchor: Anchor.center,   // (200, 140) désigne maintenant le CENTRE
);
```

Les valeurs prêtes à l'emploi sont au nombre de neuf.

```text
  topLeft ──── topCenter ──── topRight
     │             │             │
  centerLeft ─── center ──── centerRight
     │             │             │
  bottomLeft ─ bottomCenter ─ bottomRight
```

Récapitulons dans un tableau, en insistant sur l'unité, qui est la source d'erreur la plus fréquente.

| Propriété | Type | Unité | Valeur par défaut | Piège classique |
| --- | --- | --- | --- | --- |
| `position` | `NotifyingVector2` | pixels, repère du parent | `Vector2.zero()` | partager l'instance entre deux composants |
| `size` | `NotifyingVector2` | pixels, échelle 1 | `Vector2.zero()` | l'oublier, donc aucun tap et aucune hitbox |
| `scale` | `NotifyingVector2` | facteur sans unité | `Vector2.all(1)` | croire que `scale` modifie `size` |
| `angle` | `double` | **radians** | `0` | écrire des degrés |
| `anchor` | `Anchor` | fraction 0..1 | **`Anchor.topLeft`** | croire que c'est `center` |
| `nativeAngle` | `double` | radians | `0` | le confondre avec `angle` |
| `priority` | `int` | entier | `0` | croire que c'est l'ordre d'ajout |

Un mot sur `nativeAngle`, rarement utile mais parfois salvateur : il décrit l'orientation « naturelle » de votre visuel quand `angle == 0`. Si votre flèche est dessinée pointant vers le haut, `nativeAngle` vaut `0` ; si elle pointe vers la droite, `nativeAngle` vaut `pi / 2`. La méthode `angleTo(cible)` en tient compte pour orienter un composant vers un point.

---

## 28.7 — L'ancre : le piège du coin haut-gauche par défaut

C'est le piège le plus coûteux du chapitre. Prenez le temps de lire cette section deux fois.

Par défaut, `anchor` vaut **`Anchor.topLeft`**. Cela signifie que `position` désigne le **coin haut-gauche** du composant, et non son centre. Beaucoup de développeurs supposent l'inverse, parce que la plupart des moteurs 2D centrent par défaut.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │        LE MÊME position = (200, 140) AVEC DEUX ANCRES            │
  └──────────────────────────────────────────────────────────────────┘

  anchor: Anchor.topLeft (DÉFAUT)        anchor: Anchor.center

        (200,140)                                  200
           ●━━━━━━━━━━━┓                  ┏━━━━━━━━━┿━━━━━━━━━┓
           ┃           ┃                  ┃         │         ┃
           ┃  gobelin  ┃                  ┃         │         ┃
           ┃  32 x 32  ┃              140 ┼─────────●─────────┨
           ┃           ┃                  ┃      (200,140)    ┃
           ┃           ┃                  ┃         │         ┃
           ┗━━━━━━━━━━━┛                  ┗━━━━━━━━━┿━━━━━━━━━┛

   Le carré occupe                    Le carré occupe
   x de 200 à 232                     x de 184 à 216
   y de 140 à 172                     y de 124 à 156

   Décalage entre les deux : la moitié de la taille, soit (16, 16).
```

Cette différence de 16 pixels paraît anodine. Elle provoque trois bugs typiques.

**Bug 1 : la rotation part en vrille.** `angle` tourne autour de l'ancre. Avec `Anchor.topLeft`, un gobelin qui tourne décrit un grand cercle autour de son coin supérieur gauche au lieu de pivoter sur lui-même.

```dart
// Une pièce qui doit tourner sur elle-même : l'ancre DOIT être au centre.
final piece = CircleComponent(
  radius: 8,
  position: Vector2(160, 100),
  anchor: Anchor.center,   // sans cela, la pièce décrit une orbite
);
```

**Bug 2 : le centrage est faux d'une demi-taille.** Centrer un composant dans la fenêtre avec `position = size / 2` ne marche que si l'ancre est `center`.

```dart
// FAUX avec l'ancre par défaut : le composant est décalé vers le bas à droite.
final titre = TextComponent(text: 'DONJON DE DART');
titre.position = game.size / 2;

// CORRECT :
final titre2 = TextComponent(
  text: 'DONJON DE DART',
  anchor: Anchor.center,
);
titre2.position = game.size / 2;
```

**Bug 3 : la distance entre deux entités est faussée.** Comparer les `position` de deux composants d'ancres différentes compare des points qui n'ont rien à voir. Utilisez `absoluteCenter` quand vous voulez raisonner en centres.

Voici un programme complet qui met les deux ancres côte à côte, avec un point rouge marquant la position déclarée.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoAncre()));
}

class DemoAncre extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Carré A : ancre par défaut (topLeft).
    await world.add(
      RectangleComponent(
        position: Vector2(100, 100),
        size: Vector2(60, 60),
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );
    await world.add(_marqueur(Vector2(100, 100)));

    // Carré B : ancre au centre, même position déclarée.
    await world.add(
      RectangleComponent(
        position: Vector2(260, 100),
        size: Vector2(60, 60),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF2196F3),
      ),
    );
    await world.add(_marqueur(Vector2(260, 100)));
  }

  // Un petit disque rouge qui matérialise le point (x, y) déclaré.
  CircleComponent _marqueur(Vector2 p) => CircleComponent(
        radius: 4,
        position: p,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFF44336),
      );
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │        ●▓▓▓▓▓▓                        ▓▓▓▓▓▓▓▓▓              │
  │        ▓▓▓▓▓▓▓                        ▓▓▓▓▓▓▓▓▓              │
  │        ▓▓▓▓▓▓▓                        ▓▓▓▓●▓▓▓▓              │
  │        ▓▓▓▓▓▓▓                        ▓▓▓▓▓▓▓▓▓              │
  │        ▓▓▓▓▓▓▓                        ▓▓▓▓▓▓▓▓▓              │
  │                                                              │
  │   Le point rouge est               Le point rouge est        │
  │   au COIN du carré vert            au CENTRE du carré bleu   │
  └──────────────────────────────────────────────────────────────┘
```

**Une précision essentielle, souvent ignorée.** L'ancre ne change **pas** l'endroit où se placent les enfants. La documentation officielle est formelle : l'origine locale d'un composant enfant est toujours le **coin haut-gauche** de son parent, quelle que soit l'ancre du parent. Un enfant en `Vector2(0, 0)` se pose donc au coin haut-gauche du parent, même si le parent est ancré au centre.

> **À retenir.** Règle de travail : **ancre `center` pour tout ce qui tourne, ce qui se centre ou ce que l'on compare** ; ancre `topLeft` pour les décors, les sols et le HUD aligné en haut à gauche. Et fixez-la explicitement : un `anchor` écrit noir sur blanc vaut mieux qu'une valeur par défaut supposée.

---

## 28.8 — `PositionComponent` : coordonnées locales et coordonnées absolues

Un composant vit dans le repère de son parent. Quand les composants s'empilent, les repères s'empilent aussi. Il faut donc savoir distinguer deux questions différentes :

- « où est ce composant **pour son parent** ? » → `position`, coordonnées **locales** ;
- « où est ce composant **dans le monde** ? » → `absolutePosition`, coordonnées **absolues**.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                  EMPILEMENT DES REPÈRES                          │
  └──────────────────────────────────────────────────────────────────┘

  World (origine du monde)
   (0,0)
     ┌─────────────────────────────────────────────────────┐
     │                                                     │
     │        Joueur.position = (150, 80)                  │
     │              ┌────────────────────┐                 │
     │              │ (0,0) du Joueur    │                 │
     │              │                    │                 │
     │              │  BarreDeVie        │                 │
     │              │  .position = (0,-10)                 │
     │              │      ▬▬▬▬▬▬        │                 │
     │              │                    │                 │
     │              │   Remplissage      │                 │
     │              │   .position=(2, 2) │                 │
     │              └────────────────────┘                 │
     └─────────────────────────────────────────────────────┘

  Position LOCALE de Remplissage        : (2, 2)
  Position LOCALE de BarreDeVie         : (0, -10)
  Position LOCALE de Joueur             : (150, 80)

  Position ABSOLUE de Remplissage       : (150+0+2, 80-10+2) = (152, 72)
```

La règle de calcul est simple tant qu'il n'y a ni rotation ni mise à l'échelle : on additionne les positions en remontant les parents. Dès qu'un ancêtre porte un `angle` ou un `scale`, l'addition ne suffit plus, et il faut composer les transformations. C'est précisément ce que font les méthodes de la section suivante — raison pour laquelle on ne calcule jamais ces valeurs à la main.

Voici une démonstration en console, dans `onLoad`, une fois les composants montés.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoRepere()));
}

class DemoRepere extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final remplissage = RectangleComponent(
      position: Vector2(2, 2),
      size: Vector2(36, 6),
      paint: Paint()..color = const Color(0xFF4CAF50),
    );

    final barre = PositionComponent(
      position: Vector2(0, -10),
      size: Vector2(40, 10),
      children: [remplissage],
    );

    final joueur = PositionComponent(
      position: Vector2(150, 80),
      size: Vector2(40, 40),
      children: [barre],
    );

    await world.add(joueur);

    // À ce stade, tout est monté : les positions absolues sont exploitables.
    print('local  remplissage : ${remplissage.position}');
    print('absolu remplissage : ${remplissage.absolutePosition}');
    print('absolu barre       : ${barre.absolutePosition}');
    print('absolu joueur      : ${joueur.absolutePosition}');
  }
}
```

**Résultat :**

```text
local  remplissage : [2.0,2.0]
absolu remplissage : [152.0,72.0]
absolu barre       : [150.0,70.0]
absolu joueur      : [150.0,80.0]
```

> **À retenir.** `position` est **relative au parent**. Elle ne répond pas à la question « où est-ce à l'écran ». Ne comparez jamais les `position` de deux composants qui n'ont pas le même parent.

---

## 28.9 — `absolutePosition`, `positionOf`, `toAbsoluteRect`

Flame fournit une famille complète de convertisseurs. Les voici, avec leur usage réel.

| Membre | Type de retour | Réponse à la question |
| --- | --- | --- |
| `position` | `NotifyingVector2` | où est mon ancre, pour mon parent ? |
| `absolutePosition` | `Vector2` | où est mon ancre, dans le monde ? |
| `center` | `Vector2` | où est mon centre, pour mon parent ? |
| `absoluteCenter` | `Vector2` | où est mon centre, dans le monde ? |
| `topLeftPosition` | `Vector2` | où est mon coin haut-gauche, pour mon parent ? |
| `absoluteTopLeftPosition` | `Vector2` | où est mon coin haut-gauche, dans le monde ? |
| `positionOf(Vector2)` | `Vector2` | ce point local, où est-il pour mon parent ? |
| `positionOfAnchor(Anchor)` | `Vector2` | cette ancre, où est-elle pour mon parent ? |
| `absolutePositionOf(Vector2)` | `Vector2` | ce point local, où est-il dans le monde ? |
| `absolutePositionOfAnchor(Anchor)` | `Vector2` | cette ancre, où est-elle dans le monde ? |
| `toLocal(Vector2)` | `Vector2` | ce point du parent, où est-il chez moi ? |
| `absoluteToLocal(Vector2)` | `Vector2` | ce point du monde, où est-il chez moi ? |
| `toRect()` | `Rect` | mon rectangle englobant, chez mon parent |
| `toAbsoluteRect()` | `Rect` | mon rectangle englobant, dans le monde |
| `absoluteAngle` | `double` | mon angle cumulé avec ceux de mes ancêtres |
| `absoluteScale` | `Vector2` | mon échelle cumulée avec celle de mes ancêtres |
| `distance(PositionComponent)` | `double` | à quelle distance suis-je de cet autre composant ? |
| `angleTo(Vector2)` | `double` | de quel angle dois-je tourner pour viser ce point ? |

Un exemple montre l'usage de chacune sur un cas concret : la pointe de l'épée d'un héros.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoConversion()));
}

class DemoConversion extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // L'épée est un enfant du héros ; elle mesure 30x6.
    final epee = RectangleComponent(
      position: Vector2(40, 18),
      size: Vector2(30, 6),
      paint: Paint()..color = const Color(0xFFB0BEC5),
    );

    final heros = RectangleComponent(
      position: Vector2(100, 60),
      size: Vector2(40, 40),
      paint: Paint()..color = const Color(0xFFE8B04B),
      children: [epee],
    );

    await world.add(heros);

    // 1. Le point d'ancrage, dans le monde.
    print('absolutePosition epee      : ${epee.absolutePosition}');

    // 2. Un point LOCAL de l'épée (sa pointe : x = 30, y = 3),
    //    exprimé dans le repère du héros.
    print('positionOf pointe          : ${epee.positionOf(Vector2(30, 3))}');

    // 3. Le même point, dans le monde.
    print('absolutePositionOf pointe  : ${epee.absolutePositionOf(Vector2(30, 3))}');

    // 4. Une ancre remarquable, dans le monde.
    print('absolute bottomRight       : '
        '${epee.absolutePositionOfAnchor(Anchor.bottomRight)}');

    // 5. Le rectangle englobant, dans le repère du héros puis dans le monde.
    print('toRect                     : ${epee.toRect()}');
    print('toAbsoluteRect             : ${epee.toAbsoluteRect()}');

    // 6. Le chemin inverse : un point du monde ramené dans le repère de l'épée.
    print('absoluteToLocal (145,66)   : ${epee.absoluteToLocal(Vector2(145, 66))}');

    // 7. Le centre absolu, la valeur la plus utile pour comparer deux entités.
    print('absoluteCenter heros       : ${heros.absoluteCenter}');
    print('absoluteCenter epee        : ${epee.absoluteCenter}');
  }
}
```

**Résultat :**

```text
absolutePosition epee      : [140.0,78.0]
positionOf pointe          : [70.0,21.0]
absolutePositionOf pointe  : [170.0,81.0]
absolute bottomRight       : [170.0,84.0]
toRect                     : Rect.fromLTRB(40.0, 18.0, 70.0, 24.0)
toAbsoluteRect             : Rect.fromLTRB(140.0, 78.0, 170.0, 84.0)
absoluteToLocal (145,66)   : [5.0,-12.0]
absoluteCenter heros       : [120.0,80.0]
absoluteCenter epee        : [155.0,81.0]
```

Trois usages typiques de ces méthodes en situation réelle.

**Usage 1 : faire apparaître un projectile à la pointe d'une arme.** Le projectile est ajouté au monde, pas à l'arme, sinon il suivrait le héros. Il faut donc convertir.

```dart
void tirer() {
  final depart = epee.absolutePositionOf(Vector2(30, 3));
  world.add(
    CircleComponent(
      radius: 4,
      position: depart,
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFFFFEB3B),
    ),
  );
}
```

**Usage 2 : tester un chevauchement grossier entre deux entités.**

```dart
bool seTouchent(PositionComponent a, PositionComponent b) {
  return a.toAbsoluteRect().overlaps(b.toAbsoluteRect());
}
```

**Usage 3 : mesurer une distance pour une IA de poursuite.**

```dart
if (gobelin.absoluteCenter.distanceTo(heros.absoluteCenter) < 120) {
  // Le gobelin voit le héros : il le poursuit.
}
```

> **À retenir.** Ne recalculez **jamais** une position absolue à la main en additionnant les parents. Dès qu'un ancêtre tourne ou se met à l'échelle, votre addition devient fausse, alors que `absolutePositionOf` reste juste.

---

## 28.10 — Ajouter un composant : `add()` et `addAll()`

Il existe quatre façons d'ajouter un composant. Toutes produisent le même arbre ; elles diffèrent par le moment et la commodité.

### Façon 1 — `add()` sur le parent

```dart
final gobelin = Gobelin(position: Vector2(200, 120));
world.add(gobelin);
```

### Façon 2 — `addAll()` pour un lot

```dart
world.addAll([
  Gobelin(position: Vector2(200, 120)),
  Gobelin(position: Vector2(260, 150)),
  Gobelin(position: Vector2(320, 110)),
]);
```

`addAll` est strictement équivalent à une suite d'`add`, mais renvoie un unique `Future<void>` que l'on peut attendre en une fois.

### Façon 3 — le paramètre `children` du constructeur

```dart
final joueur = PositionComponent(
  position: Vector2(100, 100),
  size: Vector2(40, 40),
  children: [
    BarreDeVie(),
    Etiquette(nom: 'Héros'),
  ],
);
```

C'est la forme la plus lisible pour une composition fixe, connue dès la construction.

### Façon 4 — `add()` depuis l'intérieur du composant, dans son `onLoad`

```dart
class Gobelin extends PositionComponent {
  Gobelin({super.position}) : super(size: Vector2(30, 30));

  @override
  Future<void> onLoad() async {
    // Ici, `add` s'applique à `this` : le corps devient enfant du gobelin.
    await add(
      RectangleComponent(
        size: size,
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );
    await add(
      CircleComponent(
        radius: 4,
        position: Vector2(20, 8),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF000000),
      ),
    );
  }
}
```

C'est la forme idéale quand la composition dépend de données ou de ressources chargées.

Voici un programme complet qui utilise les quatre façons.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoAjout()));
}

class DemoAjout extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Façon 3 : enfants au constructeur.
    final sol = RectangleComponent(
      position: Vector2(0, 200),
      size: Vector2(400, 60),
      paint: Paint()..color = const Color(0xFF37474F),
      children: [
        RectangleComponent(
          position: Vector2(0, 0),
          size: Vector2(400, 4),
          paint: Paint()..color = const Color(0xFF546E7A),
        ),
      ],
    );

    // Façon 1 : un par un.
    await world.add(sol);

    // Façon 2 : un lot.
    await world.addAll([
      Gobelin(position: Vector2(120, 170)),
      Gobelin(position: Vector2(200, 170)),
      Gobelin(position: Vector2(280, 170)),
    ]);

    print('Enfants du monde : ${world.children.length}');
  }
}

class Gobelin extends PositionComponent {
  Gobelin({super.position}) : super(size: Vector2(30, 30));

  // Façon 4 : composition interne dans onLoad.
  @override
  Future<void> onLoad() async {
    await add(
      RectangleComponent(
        size: size,
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );
    await add(
      CircleComponent(
        radius: 4,
        position: Vector2(21, 9),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF212121),
      ),
    );
  }
}
```

**Résultat :**

```text
Enfants du monde : 4
```

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │                                                              │
  │        ▓▓▓▓        ▓▓▓▓        ▓▓▓▓                          │
  │        ▓▓●▓        ▓▓●▓        ▓▓●▓   <- trois gobelins      │
  │  ════════════════════════════════════════════════            │
  │  ████████████████████████████████████████████████ <- le sol  │
  └──────────────────────────────────────────────────────────────┘
```

---

## 28.11 — `add()` est asynchrone : ce que cela implique

Voici la signature réelle, telle qu'elle figure dans le code source de Flame 1.38.0 :

```dart
FutureOr<void> add(Component component);
Future<void> addAll(Iterable<Component> components);
```

Le type `FutureOr<void>` (vu au chapitre 15) signifie : « soit un résultat immédiat, soit un `Future` ». Autrement dit, **l'ajout n'est pas garanti immédiat**.

Pourquoi ? Pour deux raisons cumulées.

**Raison 1 : `onLoad()` du composant ajouté peut être asynchrone.** Charger une image, un son ou un fichier prend du temps. Flame n'insère le composant dans l'arbre qu'une fois son `onLoad` terminé.

**Raison 2 : l'arbre ne doit jamais être modifié pendant qu'il est parcouru.** C'est le bug de modification concurrente du chapitre 26. Flame place les ajouts dans une file interne et les applique **entre deux frames**, à un moment sûr.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                CE QUI SE PASSE QUAND ON APPELLE add()            │
  └──────────────────────────────────────────────────────────────────┘

   Votre code                Flame                        Arbre
  ─────────────────────────────────────────────────────────────────
   world.add(g)  ────────>  file d'attente
                            [g]
                                │
                            (fin de la frame courante)
                                │
                            appel de g.onLoad()  ...attente...
                                │
                            appel de g.onGameResize(taille)
                                │
                            insertion réelle  ─────────>  monté
                                │
                            appel de g.onMount()
                                │
                            g.isMounted == true
                                │
                            frame suivante : update(dt) puis render()
```

Conséquence pratique : **la ligne juste après `add()` s'exécute avant que le composant soit dans l'arbre.**

```dart
// PIÈGE
final gobelin = Gobelin();
world.add(gobelin);
print(world.children.length);   // affiche 0, pas 1 !
print(gobelin.isMounted);       // affiche false
print(gobelin.parent);          // affiche null
```

Ce comportement en surprend beaucoup, et il produit des bugs difficiles à lire, parce que le code « marche » la plupart du temps : à la frame suivante, tout est en place. Le problème n'apparaît que lorsque la ligne suivante dépend vraiment du montage.

Voici la démonstration complète, exécutable, qui montre l'avant et l'après.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoAsync()));
}

class DemoAsync extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final g1 = Gobelin();

    // SANS await
    world.add(g1);
    print('sans await -> enfants du monde : ${world.children.length}');
    print('sans await -> g1.isMounted     : ${g1.isMounted}');

    final g2 = Gobelin();

    // AVEC await
    await world.add(g2);
    print('avec await -> enfants du monde : ${world.children.length}');
    print('avec await -> g2.isMounted     : ${g2.isMounted}');
  }
}

class Gobelin extends PositionComponent {
  Gobelin() : super(size: Vector2(30, 30));
}
```

**Résultat :**

```text
sans await -> enfants du monde : 0
sans await -> g1.isMounted     : false
avec await -> enfants du monde : 2
avec await -> g2.isMounted     : true
```

Observez la deuxième valeur : elle vaut **2**, pas 1. En attendant `g2`, on a laissé à Flame le temps de traiter aussi `g1`, qui était déjà en file. Cela illustre bien le fonctionnement par file.

Quand faut-il attendre ? Le tableau tranche.

| Situation | `await` nécessaire ? |
| --- | --- |
| Ajouter une entité et ne rien en faire ensuite | non |
| Ajouter dans `update(double dt)` | **impossible** : `update` n'est pas `async` ; ne pas attendre |
| Lire `children.length` juste après | oui |
| Appeler une méthode qui utilise `parent` ou `game` | oui |
| Ajouter un composant puis le référencer dans un autre | oui |
| Ajouter une série de composants dont l'ordre importe | oui |
| Ajouter un enfant dans `onLoad()` | oui, par sécurité |

> **À retenir.** `add()` est une **demande**, pas une exécution. Le composant est « ajouté » quand `isMounted` devient vrai, pas quand la ligne `add()` est passée.

---

## 28.12 — `await add()` dans `onLoad()`

`onLoad()` renvoie `FutureOr<void>`. Vous pouvez donc l'écrire de deux façons.

```dart
// Version synchrone : aucun await à l'intérieur.
@override
void onLoad() {
  anchor = Anchor.center;
}

// Version asynchrone : nécessaire dès qu'on attend quelque chose.
@override
Future<void> onLoad() async {
  await add(BarreDeVie());
}
```

Trois règles simples suffisent à ne jamais se tromper.

### Règle 1 — Appeler `super.onLoad()` quand la classe parente en a un

Sur `FlameGame`, `super.onLoad()` fait un travail réel : il monte le `World` et le `CameraComponent`. L'oublier laisse `world` non monté, et vos `world.add(...)` restent en file indéfiniment.

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();   // NE JAMAIS OUBLIER dans un FlameGame
    await world.add(Heros());
  }
}
```

Sur un `PositionComponent` que vous écrivez vous-même, `super.onLoad()` ne fait rien de particulier, mais l'appeler reste une bonne habitude : si vous insérez plus tard une classe intermédiaire, le code continuera de fonctionner.

### Règle 2 — Attendre les ajouts dont la suite dépend

```dart
class Joueur extends PositionComponent {
  Joueur() : super(size: Vector2(40, 40));

  late final BarreDeVie _barre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _barre = BarreDeVie();
    await add(_barre);          // attendu : on va s'en servir tout de suite

    _barre.definirRatio(0.75);  // sûr, car _barre est monté
  }
}
```

### Règle 3 — Ne jamais transformer `update` en `async`

```dart
// INTERDIT : la boucle de jeu n'attend pas les Future.
@override
Future<void> update(double dt) async {   // ne compile pas : signature invalide
  await add(Explosion());
}

// CORRECT : on ajoute sans attendre.
@override
void update(double dt) {
  super.update(dt);
  if (_doitExploser) {
    _doitExploser = false;
    add(Explosion());   // sera monté à la frame suivante, c'est très bien
  }
}
```

Voici un exemple complet où l'ordre d'initialisation compte vraiment.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoOnLoad()));
}

class DemoOnLoad extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final heros = Heros(position: Vector2(120, 90));
    await world.add(heros);

    // Sans le await ci-dessus, cet appel toucherait un composant non monté.
    heros.subirDegats(30);
  }
}

class Heros extends PositionComponent {
  Heros({super.position}) : super(size: Vector2(40, 40));

  int pointsDeVie = 100;
  late final BarreDeVie _barre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );

    _barre = BarreDeVie(largeur: size.x);
    await add(_barre);
  }

  void subirDegats(int montant) {
    pointsDeVie = (pointsDeVie - montant).clamp(0, 100);
    _barre.definirRatio(pointsDeVie / 100);
    print('Points de vie : $pointsDeVie');
  }
}

class BarreDeVie extends PositionComponent {
  BarreDeVie({required this.largeur})
      : super(position: Vector2(0, -10), size: Vector2(largeur, 6));

  final double largeur;

  late final RectangleComponent _remplissage;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size,
        paint: Paint()..color = const Color(0xFF7F1D1D),
      ),
    );

    _remplissage = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF4CAF50),
    );
    await add(_remplissage);
  }

  void definirRatio(double ratio) {
    _remplissage.size.x = largeur * ratio.clamp(0.0, 1.0);
  }
}
```

**Résultat :**

```text
Points de vie : 70
```

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │              ██████████░░░░  <- barre de vie à 70 %          │
  │              ▓▓▓▓▓▓▓▓▓▓▓▓▓▓                                  │
  │              ▓▓▓▓▓▓▓▓▓▓▓▓▓▓  <- le héros, carré doré         │
  │              ▓▓▓▓▓▓▓▓▓▓▓▓▓▓                                  │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

> **À retenir.** Dans `onLoad`, `await` chaque ajout dont vous vous servirez ensuite. Dans `update`, jamais d'`await` : appelez `add()` et laissez Flame faire.

---

## 28.13 — Retirer un composant : `remove()`, `removeFromParent()`

Il existe quatre méthodes de retrait. Toutes sont **synchrones à l'appel mais différées à l'effet**, exactement comme l'ajout.

```dart
parent.remove(enfant);                        // retirer un enfant précis
parent.removeAll([e1, e2, e3]);               // retirer un lot
parent.removeWhere((c) => c is Gobelin);      // retirer selon un test
enfant.removeFromParent();                    // se retirer soi-même
```

`removeFromParent()` est de loin la plus utilisée, parce qu'un composant sait généralement lui-même quand il doit disparaître, alors que son parent ne le sait pas.

```dart
class Fleche extends PositionComponent {
  Fleche({super.position}) : super(size: Vector2(12, 4));

  double vitesse = 260;

  @override
  void update(double dt) {
    super.update(dt);
    position.x += vitesse * dt;

    // Sortie d'écran : la flèche se supprime elle-même.
    if (position.x > 500) {
      removeFromParent();
    }
  }
}
```

Ce qui se passe exactement au retrait :

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                CE QUI SE PASSE AU RETRAIT                        │
  └──────────────────────────────────────────────────────────────────┘

   removeFromParent()
        │
        ├── mise en file de suppression
        │
        (fin de la frame courante)
        │
        ├── appel de onRemove() sur le composant
        │
        ├── appel de onRemove() sur TOUS ses descendants
        │
        ├── détachement de l'arbre : parent devient null
        │
        └── isMounted == false, isRemoved == true
            -> plus aucun update(), plus aucun render()
```

Trois conséquences à connaître.

**Conséquence 1 : les enfants partent avec le parent.** Retirer le héros retire sa barre de vie et son étiquette. Vous n'avez rien à nettoyer.

**Conséquence 2 : le composant reste vivant en mémoire.** Il est détaché de l'arbre, mais l'objet Dart existe encore tant qu'une variable le référence. Vous pouvez donc le **remettre** dans l'arbre plus tard.

```dart
final coffre = Coffre();
await world.add(coffre);

coffre.removeFromParent();     // sort de l'arbre
await world.add(coffre);       // y revient : onLoad n'est PAS rejoué,
                               // mais onMount l'est
```

**Conséquence 3 : appeler `removeFromParent()` deux fois n'est pas dramatique**, mais c'est le signe d'une logique confuse. Protégez-vous avec `isMounted` si nécessaire.

Voici un programme complet où l'on ramasse des pièces en les retirant de l'arbre.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoRetrait()));
}

class DemoRetrait extends FlameGame {
  int score = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.addAll([
      Piece(position: Vector2(80, 100)),
      Piece(position: Vector2(140, 100)),
      Piece(position: Vector2(200, 100)),
      Piece(position: Vector2(260, 100)),
    ]);

    await world.add(Ramasseur());
  }
}

class Piece extends CircleComponent {
  Piece({super.position})
      : super(
          radius: 8,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFFFC107),
        );

  @override
  void onRemove() {
    super.onRemove();
    print('Une pièce a été ramassée.');
  }
}

// Un composant sans apparence qui ramasse une pièce par seconde.
class Ramasseur extends Component {
  double _temps = 0;

  @override
  void update(double dt) {
    super.update(dt);
    _temps += dt;
    if (_temps < 1) return;
    _temps = 0;

    // parent est le World : on cherche la première pièce restante.
    final piece = parent?.firstChild<Piece>();
    if (piece != null) {
      piece.removeFromParent();
    } else {
      print('Plus aucune pièce.');
      removeFromParent();
    }
  }
}
```

**Résultat (console, une ligne par seconde) :**

```text
Une pièce a été ramassée.
Une pièce a été ramassée.
Une pièce a été ramassée.
Une pièce a été ramassée.
Plus aucune pièce.
```

---

## 28.14 — Le cycle de vie complet

Voici le schéma le plus important du chapitre. Recopiez-le, affichez-le, il répond à 80 % des questions que vous vous poserez.

```text
  ┌──────────────────────────────────────────────────────────────────────┐
  │              CYCLE DE VIE D'UN COMPOSANT FLAME 1.38.0                │
  └──────────────────────────────────────────────────────────────────────┘

   ┌────────────────────┐
   │   constructeur     │   Synchrone. Aucun accès à `parent`, ni à
   │  Gobelin(...)      │   `game`, ni à la taille de l'écran.
   └─────────┬──────────┘   On n'y fait que des affectations simples.
             │
             │   parent.add(gobelin)   -> mise en file
             ▼
   ┌────────────────────┐
   │     onLoad()       │   UNE SEULE FOIS dans la vie du composant.
   │  FutureOr<void>    │   Chargement des ressources, création des
   │                    │   enfants. Peut être asynchrone.
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │ onGameResize(size) │   Appelé AVANT onMount, puis à chaque
   │      void          │   redimensionnement de la fenêtre.
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │    onMount()       │   À CHAQUE montage dans l'arbre.
   │      void          │   Ici, `parent` est valide.
   └─────────┬──────────┘   isMounted devient true.
             │
             ▼
   ╔════════════════════════════════════════════╗
   ║           BOUCLE DE JEU (60 fois/s)        ║
   ║                                            ║
   ║    ┌──────────────┐    ┌────────────────┐  ║
   ║    │  update(dt)  │ -> │ render(canvas) │  ║
   ║    └──────────────┘    └────────────────┘  ║
   ║       logique              dessin          ║
   ║       dt en secondes       coord. locales  ║
   ╚═════════════════┬══════════════════════════╝
             │
             │   removeFromParent()   -> mise en file
             ▼
   ┌────────────────────┐
   │    onRemove()      │   UNE SEULE FOIS.
   │      void          │   Nettoyage : abonnements, minuteurs.
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │   hors de l'arbre  │   parent == null, isMounted == false
   │                    │   isRemoved == true
   └────────────────────┘

   Si on le RÉ-AJOUTE :  onLoad n'est PAS rejoué (déjà fait une fois),
                         onGameResize et onMount le sont.
```

Détaillons chaque étape avec ce qu'il est permis d'y faire.

| Étape | Fréquence | `parent` disponible ? | `game` disponible ? | Ce qu'on y met |
| --- | --- | --- | --- | --- |
| constructeur | 1 fois | non | non | affectations simples, paramètres |
| `onLoad()` | 1 fois | pas encore fiable | oui, via `findGame()` | chargement, création d'enfants |
| `onGameResize(size)` | à chaque redimension | pas garanti | oui | recalcul des positions dépendant de l'écran |
| `onMount()` | à chaque montage | **oui** | oui | branchements, abonnements, position finale |
| `update(dt)` | ~60 fois/s | oui | oui | logique, déplacement, IA |
| `render(canvas)` | ~60 fois/s | oui | oui | dessin uniquement, aucune modification d'état |
| `onRemove()` | 1 fois | encore oui | oui | désabonnement, arrêt de sons |

Le programme suivant trace le cycle de vie complet dans la console. Lancez-le, redimensionnez la fenêtre, puis attendez trois secondes.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoCycleDeVie()));
}

class DemoCycleDeVie extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Espion());
  }
}

class Espion extends PositionComponent {
  Espion() : super(position: Vector2(100, 100), size: Vector2(40, 40)) {
    print('1. constructeur   -> parent = $parent');
  }

  double _temps = 0;
  int _framesTracees = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    print('2. onLoad         -> parent = $parent');
  }

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);
    print('3. onGameResize   -> taille écran = $size');
  }

  @override
  void onMount() {
    super.onMount();
    print('4. onMount        -> parent = ${parent.runtimeType}');
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_framesTracees < 2) {
      _framesTracees++;
      print('5. update         -> dt = ${dt.toStringAsFixed(4)} s');
    }
    _temps += dt;
    if (_temps > 3) {
      removeFromParent();
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), Paint()..color = const Color(0xFF9C27B0));
  }

  @override
  void onRemove() {
    super.onRemove();
    print('7. onRemove       -> le composant quitte l\'arbre');
  }
}
```

**Résultat :**

```text
1. constructeur   -> parent = null
2. onLoad         -> parent = null
3. onGameResize   -> taille écran = [800.0,600.0]
4. onMount        -> parent = World
5. update         -> dt = 0.0166 s
5. update         -> dt = 0.0167 s
7. onRemove       -> le composant quitte l'arbre
```

Deux observations à ne pas manquer.

**`parent` vaut `null` dans `onLoad`.** C'est la raison pour laquelle tout code qui a besoin du parent doit aller dans `onMount`, pas dans `onLoad`.

**`onGameResize` arrive avant `onMount`.** Il est donc appelé au moins une fois même sans redimensionnement réel. Votre implémentation doit supporter d'être appelée alors que le composant n'est pas encore monté.

---

## 28.15 — `onGameResize`

`onGameResize(Vector2 size)` est appelé dans deux circonstances :

1. lors de l'ajout du composant à l'arbre, **avant** `onMount` ;
2. à chaque fois que la surface du jeu change de taille : rotation d'un téléphone, redimensionnement d'une fenêtre de bureau, changement de taille du navigateur.

Le paramètre `size` est la taille de la **surface du jeu**, pas celle du composant. Ne le confondez pas avec le champ `size` du `PositionComponent` : dans le corps de la méthode, le paramètre masque le champ.

```dart
class BordDroit extends PositionComponent {
  BordDroit() : super(size: Vector2(8, 400));

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);      // NE JAMAIS OUBLIER
    // `size` ici est la taille du JEU. Le champ du composant est `this.size`.
    this.size = Vector2(8, size.y);
    position = Vector2(size.x - 8, 0);
  }
}
```

Cas d'usage typique : garder un élément de HUD collé au bord.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoResize()));
}

class DemoResize extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Ajouté au viewport : reste fixe à l'écran.
    await camera.viewport.add(BandeauBas());
  }
}

class BandeauBas extends PositionComponent {
  BandeauBas() : super(size: Vector2(0, 40));

  static final Paint _fond = Paint()..color = const Color(0xCC1B1B2F);

  @override
  void onGameResize(Vector2 tailleDuJeu) {
    super.onGameResize(tailleDuJeu);
    size = Vector2(tailleDuJeu.x, 40);
    position = Vector2(0, tailleDuJeu.y - 40);
    print('Bandeau replacé : taille du jeu = $tailleDuJeu');
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _fond);
  }
}
```

**Résultat (en redimensionnant la fenêtre trois fois) :**

```text
Bandeau replacé : taille du jeu = [800.0,600.0]
Bandeau replacé : taille du jeu = [640.0,600.0]
Bandeau replacé : taille du jeu = [640.0,480.0]
```

Deux règles pour ne pas se piéger.

**Règle 1 : toujours appeler `super.onGameResize(size)`.** Sans cela, les enfants ne sont pas informés et le champ interne de taille du jeu n'est pas mis à jour.

**Règle 2 : ne rien y faire qui suppose le montage.** `onGameResize` précède `onMount`, donc `parent` peut valoir `null`. Écrire `parent!.children` ici lève une exception au premier appel.

Notez également l'existence de `onParentResize(Vector2 maxSize)`, appelé lorsque le **parent** change de taille. Il est utile pour les composants d'interface qui doivent s'adapter à leur conteneur plutôt qu'à l'écran.

---

## 28.16 — `loaded` et `mounted`

Flame expose l'état d'un composant de deux façons complémentaires : trois booléens pour tester tout de suite, trois `Future` pour attendre.

| Booléen | `Future` correspondant | Vrai quand |
| --- | --- | --- |
| `isLoaded` | `loaded` | `onLoad()` est terminé |
| `isMounted` | `mounted` | le composant est dans l'arbre, `onMount()` fait |
| `isRemoved` | `removed` | le composant a quitté l'arbre |

Les booléens servent aux garde-fous dans `update`, où l'on ne peut pas attendre.

```dart
@override
void update(double dt) {
  super.update(dt);
  if (!_cible.isMounted) {
    return;   // la cible a été détruite : on ne la poursuit plus
  }
  // ...
}
```

Les `Future` servent hors de la boucle, quand on peut attendre.

```dart
final boss = Boss();
world.add(boss);

// On attend que le boss soit réellement dans l'arbre avant de l'annoncer.
await boss.mounted;
print('Le boss est en place, on peut lancer la musique.');
```

Voici une démonstration complète de l'ordre des états.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoEtats()));
}

class DemoEtats extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final boss = Boss();

    print('avant add   : isLoaded=${boss.isLoaded} '
        'isMounted=${boss.isMounted} isRemoved=${boss.isRemoved}');

    world.add(boss);

    print('après add   : isLoaded=${boss.isLoaded} '
        'isMounted=${boss.isMounted} isRemoved=${boss.isRemoved}');

    await boss.loaded;
    print('après loaded: isLoaded=${boss.isLoaded} '
        'isMounted=${boss.isMounted} isRemoved=${boss.isRemoved}');

    await boss.mounted;
    print('après mount : isLoaded=${boss.isLoaded} '
        'isMounted=${boss.isMounted} isRemoved=${boss.isRemoved}');

    boss.removeFromParent();
    await boss.removed;
    print('après remove: isLoaded=${boss.isLoaded} '
        'isMounted=${boss.isMounted} isRemoved=${boss.isRemoved}');
  }
}

class Boss extends PositionComponent {
  Boss() : super(size: Vector2(64, 64));

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // On simule un chargement long (une image, un son...).
    await Future<void>.delayed(const Duration(milliseconds: 200));
  }
}
```

**Résultat :**

```text
avant add   : isLoaded=false isMounted=false isRemoved=false
après add   : isLoaded=false isMounted=false isRemoved=false
après loaded: isLoaded=true isMounted=false isRemoved=false
après mount : isLoaded=true isMounted=true isRemoved=false
après remove: isLoaded=true isMounted=false isRemoved=true
```

Notez la dernière ligne : après retrait, `isLoaded` reste `true`. C'est cohérent avec la règle « `onLoad` ne s'exécute qu'une fois » : si l'on remet le composant dans l'arbre, il ne rechargera pas ses ressources.

---

## 28.17 — Parent et enfants : `parent`, `children`

Chaque composant connaît son entourage immédiat.

```dart
Component? parent;                        // null si non monté
ReadOnlyOrderedSet<Component> children;   // enfants directs, triés
```

`children` est un ensemble **ordonné** et en **lecture seule** : on ne l'utilise pas pour ajouter ou retirer (on passe par `add` / `remove`), mais on peut le parcourir, le compter, le filtrer.

```dart
print(joueur.children.length);
for (final enfant in joueur.children) {
  print(enfant.runtimeType);
}
```

Pour explorer plus loin que les enfants directs, deux méthodes existent.

```dart
// Tous les descendants, en profondeur.
for (final c in world.descendants()) {
  print(c.runtimeType);
}

// Tous les ancêtres, en remontant vers la racine.
for (final a in barreDeVie.ancestors()) {
  print(a.runtimeType);
}
```

Il existe enfin `findGame()`, qui remonte l'arbre jusqu'au `FlameGame` et le renvoie (ou `null` si le composant n'est pas monté).

Programme complet d'exploration de l'arbre.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoArbre()));
}

class DemoArbre extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final barre = PositionComponent(size: Vector2(40, 6));
    final joueur = PositionComponent(
      position: Vector2(100, 100),
      size: Vector2(40, 40),
      children: [barre],
    );

    await world.add(joueur);
    await world.addAll([
      PositionComponent(position: Vector2(200, 100), size: Vector2(30, 30)),
      PositionComponent(position: Vector2(250, 100), size: Vector2(30, 30)),
    ]);

    print('--- enfants directs du monde ---');
    for (final c in world.children) {
      print('  ${c.runtimeType}');
    }

    print('--- ancêtres de la barre de vie ---');
    for (final a in barre.ancestors()) {
      print('  ${a.runtimeType}');
    }

    print('--- le jeu vu depuis la barre ---');
    print('  ${barre.findGame().runtimeType}');
  }
}
```

**Résultat :**

```text
--- enfants directs du monde ---
  PositionComponent
  PositionComponent
  PositionComponent
--- ancêtres de la barre de vie ---
  PositionComponent
  World
  DemoArbre
--- le jeu vu depuis la barre ---
  DemoArbre
```

Deux mixins facilitent l'accès typé à l'entourage, et évitent les transtypages :

```dart
// Le parent est garanti du type indiqué.
class BarreDeVie extends PositionComponent with ParentIsA<Joueur> {
  @override
  void onMount() {
    super.onMount();
    print('Vie du porteur : ${parent.pointsDeVie}');   // parent typé Joueur
  }
}

// Un ancêtre quelconque, pas forcément le parent direct.
class Etincelle extends PositionComponent with HasAncestor<Niveau> {
  void faire() {
    ancestor.signaler();   // ancestor est typé Niveau
  }
}
```

---

## 28.18 — Composer un personnage : corps + barre de vie + nom

Mettons l'arbre au travail. Un personnage complet du Donjon de Dart se compose de trois enfants : un corps, une barre de vie et une étiquette de nom.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │              ANATOMIE D'UN PERSONNAGE                        │
  └──────────────────────────────────────────────────────────────┘

  Personnage (PositionComponent, 40 x 40, anchor: center)
  │
  ├── TextComponent    « Héros »       position (20, -26)
  ├── BarreDeVie       (40 x 6)        position (0, -14)
  │   ├── RectangleComponent rouge     (fond, 40 x 6)
  │   └── RectangleComponent vert      (remplissage, largeur variable)
  └── RectangleComponent doré          (corps, 40 x 40)


   Rendu à l'écran (le parent est ancré au centre) :

              Héros
            ██████░░░░        <- barre de vie
            ▓▓▓▓▓▓▓▓▓▓
            ▓▓▓▓▓▓▓▓▓▓        <- corps
            ▓▓▓▓▓▓▓▓▓▓
```

Voici le programme complet. Il fonctionne sans aucune image.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoPersonnage()));
}

class DemoPersonnage extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final heros = Personnage(
      nom: 'Héros',
      couleur: const Color(0xFFE8B04B),
      position: Vector2(120, 120),
    );

    final gobelin = Personnage(
      nom: 'Gobelin',
      couleur: const Color(0xFF4CAF50),
      position: Vector2(260, 120),
    );

    await world.addAll([heros, gobelin]);

    // Le héros a déjà pris des coups.
    heros.subirDegats(35);
  }
}

class Personnage extends PositionComponent {
  Personnage({
    required this.nom,
    required this.couleur,
    super.position,
  }) : super(size: Vector2(40, 40), anchor: Anchor.center);

  final String nom;
  final Color couleur;

  int pointsDeVieMax = 100;
  late int pointsDeVie = pointsDeVieMax;

  late final BarreDeVie _barre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // 1. Le corps. Enfant en (0, 0) : coin haut-gauche du parent.
    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = couleur,
      ),
    );

    // 2. La barre de vie, juste au-dessus du corps.
    _barre = BarreDeVie(largeur: size.x)..position = Vector2(0, -14);
    await add(_barre);

    // 3. L'étiquette de nom, centrée au-dessus de la barre.
    await add(
      TextComponent(
        text: nom,
        position: Vector2(size.x / 2, -20),
        anchor: Anchor.bottomCenter,
        textRenderer: TextPaint(
          style: const TextStyle(
            fontSize: 12,
            color: Color(0xFFFFFFFF),
          ),
        ),
      ),
    );
  }

  void subirDegats(int montant) {
    pointsDeVie = (pointsDeVie - montant).clamp(0, pointsDeVieMax);
    _barre.definirRatio(pointsDeVie / pointsDeVieMax);
    if (pointsDeVie == 0) {
      removeFromParent();
    }
  }
}

class BarreDeVie extends PositionComponent {
  BarreDeVie({required this.largeur}) : super(size: Vector2(largeur, 6));

  final double largeur;

  late final RectangleComponent _remplissage;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0xFF7F1D1D),
      ),
    );

    _remplissage = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF4CAF50),
    );
    await add(_remplissage);
  }

  void definirRatio(double ratio) {
    _remplissage.size.x = largeur * ratio.clamp(0.0, 1.0);
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │          Héros                     Gobelin                   │
  │        ██████░░░░                ██████████                  │
  │        ▓▓▓▓▓▓▓▓▓▓                ▒▒▒▒▒▒▒▒▒▒                  │
  │        ▓▓▓▓▓▓▓▓▓▓                ▒▒▒▒▒▒▒▒▒▒                  │
  │        ▓▓▓▓▓▓▓▓▓▓                ▒▒▒▒▒▒▒▒▒▒                  │
  │                                                              │
  │      vie 65 %                  vie 100 %                     │
  └──────────────────────────────────────────────────────────────┘
```

Ce que cette composition vous offre gratuitement, et qu'il aurait fallu coder à la main au chapitre 26 :

- déplacer `heros.position` déplace corps, barre et nom d'un seul mouvement ;
- `heros.removeFromParent()` supprime les quatre composants d'un coup ;
- `heros.scale = Vector2.all(2)` agrandit l'ensemble, texte compris ;
- `heros.angle = 0.3` fait pivoter l'ensemble autour du centre du personnage.

> **À retenir.** Ne recopiez jamais la position d'un composant dans un autre à chaque frame. Si B doit suivre A, **faites de B un enfant de A**.

---

## 28.19 — Les transformations héritées du parent

Un enfant hérite de **toutes** les transformations de ses ancêtres : position, rotation, échelle. C'est puissant, et c'est parfois surprenant.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │          CE QUE LE PARENT IMPOSE À SES ENFANTS                    │
  └──────────────────────────────────────────────────────────────────┘

  parent.position = (100, 50)   -> tous les enfants sont décalés de (100, 50)
  parent.angle    = pi / 4      -> tous les enfants pivotent avec lui,
                                   autour de l'ANCRE DU PARENT
  parent.scale    = (2, 2)      -> tous les enfants sont deux fois plus grands,
                                   y compris l'épaisseur de leurs traits
  parent.anchor   = center      -> ne change RIEN pour les enfants :
                                   leur origine reste le coin haut-gauche
                                   du parent
```

Le dernier point mérite un exemple, parce qu'il contredit l'intuition de presque tout le monde.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoHeritage()));
}

class DemoHeritage extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Parent ancré au CENTRE.
    final parentCentre = RectangleComponent(
      position: Vector2(200, 130),
      size: Vector2(100, 100),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0x552196F3),
      children: [
        // Enfant en (0, 0) : il se pose au COIN HAUT-GAUCHE du parent,
        // pas au centre. L'ancre du parent n'y change rien.
        CircleComponent(
          radius: 6,
          position: Vector2(0, 0),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFF44336),
        ),
        // Pour poser un enfant au centre du parent, il faut le dire.
        CircleComponent(
          radius: 6,
          position: Vector2(50, 50),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFFFEB3B),
        ),
      ],
    );

    await world.add(parentCentre);
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │            ●░░░░░░░░░░░░░░░░░░                               │
  │            ░░░░░░░░░░░░░░░░░░░░  <- point ROUGE au coin,     │
  │            ░░░░░░░●░░░░░░░░░░░░     bien que le parent soit  │
  │            ░░░░░░░░░░░░░░░░░░░░     ancré au centre          │
  │            ░░░░░░░░░░░░░░░░░░░░                              │
  │                                                              │
  │            point JAUNE en (50, 50) = vrai centre du parent   │
  └──────────────────────────────────────────────────────────────┘
```

Voyons maintenant la rotation et l'échelle héritées, avec une roue à trois rayons.

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoRotation()));
}

class DemoRotation extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Roue(position: Vector2(200, 140)));
  }
}

class Roue extends PositionComponent {
  Roue({super.position})
      : super(size: Vector2(80, 80), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le moyeu, au centre du parent.
    await add(
      CircleComponent(
        radius: 8,
        position: size / 2,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF795548),
      ),
    );

    // Trois rayons. Ils tourneront avec le parent, sans code supplémentaire.
    for (var i = 0; i < 3; i++) {
      await add(
        RectangleComponent(
          size: Vector2(36, 4),
          position: size / 2,
          anchor: Anchor.centerLeft,
          angle: i * 2 * pi / 3,
          paint: Paint()..color = const Color(0xFFBCAAA4),
        ),
      );
    }
  }

  @override
  void update(double dt) {
    super.update(dt);
    // Un seul angle modifié : toute la roue tourne.
    angle += 1.2 * dt;
    // Une respiration d'échelle, héritée elle aussi.
    scale = Vector2.all(1 + 0.2 * sin(angle * 2));
  }
}
```

**Résultat :** une roue à trois rayons qui tourne régulièrement et « respire » légèrement. Aucune ligne ne touche à la position des rayons : ils sont enfants, donc ils suivent.

Le tableau récapitule qui hérite de quoi.

| Propriété du parent | Effet sur les enfants | Contournement si non voulu |
| --- | --- | --- |
| `position` | décalage identique | rattacher l'enfant à un autre parent |
| `angle` | rotation autour de l'ancre du parent | appliquer `-angle` à l'enfant |
| `scale` | mise à l'échelle proportionnelle | appliquer l'inverse à l'enfant |
| `anchor` | **aucun effet** sur les enfants | positionner l'enfant explicitement |
| `priority` | définit l'ordre du parent parmi ses frères | régler la `priority` de l'enfant entre frères |
| retrait de l'arbre | les enfants sont retirés aussi | déplacer l'enfant avant de retirer le parent |

Le cas classique du contournement : une étiquette de nom qui doit rester horizontale alors que le personnage tourne.

```dart
@override
void update(double dt) {
  super.update(dt);
  angle += dt;                 // le personnage tourne
  _etiquette.angle = -angle;   // le texte annule la rotation et reste droit
}
```

---

## 28.20 — `priority` : l'ordre de rendu

`priority` est un entier, `0` par défaut. Il détermine l'ordre de rendu **entre frères**, c'est-à-dire entre enfants d'un même parent.

Règle unique : **plus la priorité est élevée, plus le composant est dessiné tard, donc devant.**

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                     ORDRE DE RENDU                               │
  └──────────────────────────────────────────────────────────────────┘

   priority = -10   Fond du donjon        dessiné en premier
   priority =   0   Sol, murs
   priority =  10   Coffres, potions
   priority =  20   Gobelins
   priority =  30   Le joueur
   priority =  40   Projectiles, effets
   priority = 100   HUD                   dessiné en dernier, devant tout

           ARRIÈRE-PLAN  ────────────────────>  PREMIER PLAN
```

Deux façons de la fixer.

```dart
// Au constructeur :
final fond = RectangleComponent(priority: -10, /* ... */);

// Ou plus tard, à tout moment :
fond.priority = -20;
```

Une bonne pratique consiste à centraliser les valeurs dans une classe de constantes, comme au chapitre 26.

```dart
abstract final class Couches {
  static const int fond = -10;
  static const int sol = 0;
  static const int objets = 10;
  static const int ennemis = 20;
  static const int joueur = 30;
  static const int effets = 40;
  static const int hud = 100;
}
```

Programme complet montrant l'effet de `priority` sur trois carrés qui se chevauchent.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoPriorite()));
}

class DemoPriorite extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Ajoutés dans l'ordre rouge, vert, bleu.
    // Mais les priorités décident du rendu : bleu, vert, rouge.
    await world.addAll([
      RectangleComponent(
        position: Vector2(100, 100),
        size: Vector2(80, 80),
        priority: 30,
        paint: Paint()..color = const Color(0xFFF44336),
      ),
      RectangleComponent(
        position: Vector2(140, 130),
        size: Vector2(80, 80),
        priority: 20,
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
      RectangleComponent(
        position: Vector2(180, 160),
        size: Vector2(80, 80),
        priority: 10,
        paint: Paint()..color = const Color(0xFF2196F3),
      ),
    ]);

    for (final c in world.children) {
      print('${c.runtimeType} priority=${c.priority}');
    }
  }
}
```

**Résultat :**

```text
RectangleComponent priority=10
RectangleComponent priority=20
RectangleComponent priority=30
```

```text
  ┌──────────────────────────────────────────────────────────────┐
  │        ██████████                                            │
  │        ██████████         Le ROUGE (priority 30) est devant  │
  │        ████▓▓▓▓▓▓▓▓▓▓     le VERT (20), lui-même devant      │
  │            ▓▓▓▓▓▓▓▓▓▓     le BLEU (10).                      │
  │            ▓▓▓▓▒▒▒▒▒▒▒▒▒▒                                    │
  │                ▒▒▒▒▒▒▒▒▒▒                                    │
  │                ▒▒▒▒▒▒▒▒▒▒                                    │
  └──────────────────────────────────────────────────────────────┘
```

Notez que le parcours de `world.children` sort les composants **triés par priorité croissante**, et non dans l'ordre d'ajout. L'ensemble est maintenu trié en permanence : c'est le rôle du `ReadOnlyOrderedSet`.

Deux précisions importantes.

**La priorité ne joue qu'entre frères.** Un enfant avec `priority = 1000` ne passera jamais devant un composant appartenant à une autre branche dessinée plus tard. La branche entière est dessinée d'un bloc.

**En cas d'égalité, l'ordre d'insertion départage.** Deux composants de priorité identique sont dessinés dans l'ordre où ils ont été ajoutés. Ne comptez pas dessus pour du rendu critique : fixez des priorités distinctes.

---

## 28.21 — Changer la priorité à l'exécution

`priority` est modifiable à tout moment. Flame réordonne l'ensemble concerné, et le changement prend effet **avant le rendu de la frame en cours**.

Le cas d'usage classique d'un jeu vu de dessus : un personnage plus bas à l'écran doit être dessiné devant un personnage plus haut, afin de simuler la profondeur. C'est le tri « en Y ».

```dart
class Personnage extends PositionComponent {
  @override
  void update(double dt) {
    super.update(dt);
    // Plus le personnage est bas, plus il est devant.
    priority = position.y.round();
  }
}
```

Programme complet : deux personnages se croisent, et celui du dessous passe automatiquement devant.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoTriEnY()));
}

class DemoTriEnY extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.addAll([
      Marcheur(
        depart: Vector2(60, 60),
        arrivee: Vector2(260, 200),
        couleur: const Color(0xFFE8B04B),
      ),
      Marcheur(
        depart: Vector2(260, 60),
        arrivee: Vector2(60, 200),
        couleur: const Color(0xFF4CAF50),
      ),
    ]);
  }
}

class Marcheur extends PositionComponent {
  Marcheur({
    required Vector2 depart,
    required this.arrivee,
    required this.couleur,
  })  : _depart = depart.clone(),
        super(
          position: depart.clone(),
          size: Vector2(50, 70),
          anchor: Anchor.bottomCenter,
        );

  final Vector2 _depart;
  final Vector2 arrivee;
  final Color couleur;

  double _t = 0;
  int _sens = 1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = couleur,
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    _t += _sens * dt * 0.4;
    if (_t >= 1) {
      _t = 1;
      _sens = -1;
    } else if (_t <= 0) {
      _t = 0;
      _sens = 1;
    }

    position = _depart + (arrivee - _depart) * _t;

    // Tri en profondeur : le plus bas est dessiné devant.
    priority = position.y.round();
  }
}
```

**Résultat :** deux rectangles font des allers-retours en diagonale. À chaque croisement, celui dont le pied est le plus bas passe devant l'autre, sans aucun `if` explicite.

Autres cas d'usage courants du changement de priorité :

| Situation | Action |
| --- | --- |
| Le joueur passe derrière un arbre | `arbre.priority = joueur.priority + 1` selon la position |
| Une carte que l'on saisit doit passer au-dessus | `carte.priority = 999` dans `onDragStart` |
| Un menu de pause s'ouvre | `menu.priority = 1000` |
| Un ennemi entre en phase « enragée » | `ennemi.priority = Couches.effets` |

> **À retenir.** `priority` est un vrai champ de gameplay, pas un réglage figé. Le modifier dans `update` coûte très peu et remplace des tonnes de code de tri.

---

## 28.22 — Les composants intégrés

Flame fournit un jeu de composants géométriques qui suffisent à prototyper un jeu entier **sans une seule image**. C'est exactement ce dont vous avez besoin dans ce cours.

Tous héritent de `ShapeComponent`, donc tous acceptent un `Paint`.

### `RectangleComponent`

```dart
// Constructeur principal.
RectangleComponent(
  position: Vector2(50, 50),
  size: Vector2(80, 40),
  paint: Paint()..color = const Color(0xFF795548),
);

// Un carré, plus court à écrire.
RectangleComponent.square(
  size: 40,
  position: Vector2(10, 10),
  paint: Paint()..color = const Color(0xFF9C27B0),
);

// À partir d'un Rect de dart:ui.
RectangleComponent.fromRect(
  const Rect.fromLTWH(0, 0, 120, 20),
  paint: Paint()..color = const Color(0xFF37474F),
);

// Proportionnel à la taille du parent : ici la moitié en largeur,
// le quart en hauteur.
RectangleComponent.relative(
  Vector2(0.5, 0.25),
  parentSize: Vector2(200, 200),
  paint: Paint()..color = const Color(0xFF00BCD4),
);
```

### `CircleComponent`

```dart
CircleComponent(
  radius: 12,
  position: Vector2(100, 100),
  anchor: Anchor.center,
  paint: Paint()..color = const Color(0xFFFFC107),
);

CircleComponent.relative(
  0.5,
  parentSize: Vector2(60, 60),
  paint: Paint()..color = const Color(0xFFE91E63),
);
```

`CircleComponent` expose `radius` en lecture et en écriture (le modifier ajuste `size`) et `scaledRadius` en lecture seule.

### `PolygonComponent`

```dart
// Au moins 3 sommets, sinon une assertion se déclenche.
PolygonComponent(
  [
    Vector2(0, -20),
    Vector2(18, 12),
    Vector2(-18, 12),
  ],
  position: Vector2(150, 120),
  paint: Paint()..color = const Color(0xFFF44336),
);

// Polygone régulier : ajouté en Flame 1.38.0.
PolygonComponent.regular(
  // voir le dartdoc pour les paramètres exacts de votre version
);

// Défini proportionnellement au parent.
PolygonComponent.relative(
  [Vector2(0, -1), Vector2(1, 1), Vector2(-1, 1)],
  parentSize: Vector2(40, 40),
  paint: Paint()..color = const Color(0xFF8BC34A),
);
```

Attention : pour la détection de collision (chapitre 32), un polygone doit être **convexe**. Pour du rendu pur, cette contrainte ne s'applique pas.

### Peintures multiples

Tous ces composants acceptent aussi `paintLayers`, une `List<Paint>` appliquée en couches successives. Cela permet un contour peu coûteux.

```dart
RectangleComponent(
  size: Vector2(60, 60),
  paintLayers: [
    Paint()..color = const Color(0xFF3E2723),                 // remplissage
    Paint()
      ..color = const Color(0xFFFFC107)
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3,                                       // contour
  ],
);
```

Programme complet : une galerie des formes disponibles.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Galerie()));
}

class Galerie extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.addAll([
      // Un coffre : rectangle brun avec un contour doré.
      RectangleComponent(
        position: Vector2(40, 60),
        size: Vector2(60, 44),
        paintLayers: [
          Paint()..color = const Color(0xFF6D4C41),
          Paint()
            ..color = const Color(0xFFFFC107)
            ..style = PaintingStyle.stroke
            ..strokeWidth = 3,
        ],
      ),

      // Une potion : cercle rouge.
      CircleComponent(
        radius: 16,
        position: Vector2(160, 82),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE53935),
      ),

      // Une pointe de piège : triangle.
      PolygonComponent(
        [Vector2(0, -22), Vector2(20, 14), Vector2(-20, 14)],
        position: Vector2(250, 82),
        paint: Paint()..color = const Color(0xFF90A4AE),
      ),

      // Un carré, forme raccourcie.
      RectangleComponent.square(
        size: 34,
        position: Vector2(320, 64),
        paint: Paint()..color = const Color(0xFF3F51B5),
      ),

      // Une étiquette.
      TextComponent(
        text: 'Salle 1 — le corridor',
        position: Vector2(40, 20),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
        ),
      ),
    ]);
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Salle 1 — le corridor                                       │
  │                                                              │
  │   ┏━━━━━━━┓        ●●●         ▲          ███                │
  │   ┃▓▓▓▓▓▓▓┃       ●●●●●       ▲▲▲         ███                │
  │   ┗━━━━━━━┛        ●●●       ▲▲▲▲▲                           │
  │    coffre        potion      piège       bloc                │
  └──────────────────────────────────────────────────────────────┘
```

---

## 28.23 — `TextComponent` et `TextPaint`

Le texte suit exactement les mêmes règles que le reste : c'est un `PositionComponent`.

```dart
TextComponent({
  String? text,
  T? textRenderer,
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

Le style passe par `textRenderer`. L'implémentation intégrée est `TextPaint`, bâtie sur le moteur de texte de Flutter.

```dart
final score = TextComponent(
  text: 'Score : 0',
  position: Vector2.all(16),
  anchor: Anchor.topLeft,
  textRenderer: TextPaint(
    style: const TextStyle(
      fontSize: 24,
      color: Color(0xFFFFFFFF),
      fontWeight: FontWeight.bold,
    ),
  ),
);
```

Trois particularités à connaître.

**La `size` est calculée automatiquement.** Elle est recalculée à chaque changement de `text` ou de `textRenderer`. Ne la fixez pas vous-même.

**Changer le texte est instantané et bon marché.**

```dart
score.text = 'Score : $points';
```

**Réutilisez le `TextPaint`.** Construire un `TextPaint` à chaque frame gaspille du travail. Créez-le une fois, en `static final`.

```dart
class Hud extends PositionComponent {
  static final TextPaint styleNormal = TextPaint(
    style: const TextStyle(fontSize: 18, color: Color(0xFFFFFFFF)),
  );
  static final TextPaint styleAlerte = TextPaint(
    style: const TextStyle(
      fontSize: 18,
      color: Color(0xFFFF5252),
      fontWeight: FontWeight.bold,
    ),
  );
}
```

Programme complet : un HUD fixé au viewport, avec score et vies qui évoluent.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoTexte()));
}

class DemoTexte extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Un décor quelconque dans le monde.
    await world.add(
      RectangleComponent(
        position: Vector2(0, 200),
        size: Vector2(600, 80),
        paint: Paint()..color = const Color(0xFF37474F),
      ),
    );

    // Le HUD vit dans le viewport : il ne bouge pas avec la caméra.
    await camera.viewport.add(Hud());
  }
}

class Hud extends Component {
  static final TextPaint _style = TextPaint(
    style: const TextStyle(fontSize: 20, color: Color(0xFFFFFFFF)),
  );
  static final TextPaint _styleAlerte = TextPaint(
    style: const TextStyle(
      fontSize: 20,
      color: Color(0xFFFF5252),
      fontWeight: FontWeight.bold,
    ),
  );

  late final TextComponent _score;
  late final TextComponent _vies;

  int points = 0;
  int vies = 3;
  double _temps = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _score = TextComponent(
      text: 'Score : 0',
      position: Vector2(16, 14),
      textRenderer: _style,
    );

    _vies = TextComponent(
      text: 'Vies : 3',
      position: Vector2(16, 40),
      textRenderer: _style,
    );

    await addAll([_score, _vies]);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _temps += dt;
    if (_temps < 1) return;
    _temps = 0;

    points += 10;
    _score.text = 'Score : $points';

    if (points % 50 == 0 && vies > 0) {
      vies--;
      _vies.text = 'Vies : $vies';
      _vies.textRenderer = vies <= 1 ? _styleAlerte : _style;
    }
  }
}
```

**Résultat (après six secondes) :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Score : 60                                                  │
  │  Vies : 2                                                    │
  │                                                              │
  │                                                              │
  │  ██████████████████████████████████████████████████████████  │
  └──────────────────────────────────────────────────────────────┘
```

Mentionnons enfin `TextBoxComponent`, cousin de `TextComponent` pour le texte multi-lignes. Il gère un `boxConfig` de type `TextBoxConfig`, une boîte qui grandit avec `growingBox`, un effet machine à écrire via `boxConfig.timePerChar`, ainsi que les méthodes `skip()` et `resetAnimation()`. C'est le composant des dialogues de PNJ ; vous l'emploierez dans la PARTIE 2C.

---

## 28.24 — Créer son propre composant

Vous avez maintenant tout pour écrire un vrai composant de jeu. La recette tient en cinq points.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │              RECETTE D'UN COMPOSANT PERSONNALISÉ                 │
  └──────────────────────────────────────────────────────────────────┘

  1. Choisir la classe de base
       Component            -> pas d'apparence (contrôleur, système)
       PositionComponent    -> position et taille, rendu personnalisé
       RectangleComponent   -> un rectangle coloré, rien de plus
       SpriteComponent      -> une image                (chapitre 29)

  2. Constructeur
       ne fait QUE transmettre les paramètres à super
       et affecter des champs simples

  3. onLoad()
       créer les enfants, charger les ressources
       await sur chaque add() dont on se sert ensuite

  4. update(double dt)
       super.update(dt) EN PREMIER
       toute la logique, multipliée par dt

  5. render(Canvas canvas)   [facultatif]
       super.render(canvas) EN PREMIER
       dessin en coordonnées LOCALES seulement
```

Voici un `Joueur` complet, sans image, qui applique la recette.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(
      RectangleComponent(
        position: Vector2(0, 220),
        size: Vector2(600, 60),
        priority: -1,
        paint: Paint()..color = const Color(0xFF263238),
      ),
    );

    await world.add(Joueur(position: Vector2(80, 190)));
  }
}

// 1. Classe de base : PositionComponent, car on veut un rendu à nous.
class Joueur extends PositionComponent {
  // 2. Constructeur : transmission et affectations simples uniquement.
  Joueur({super.position})
      : super(size: Vector2(36, 48), anchor: Anchor.bottomLeft);

  static final Paint _corps = Paint()..color = const Color(0xFFE8B04B);
  static final Paint _ceinture = Paint()..color = const Color(0xFF5D4037);

  double vitesse = 90;      // pixels par seconde
  int _sens = 1;
  int pointsDeVie = 100;

  late final TextComponent _etiquette;

  // 3. onLoad : création des enfants.
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _etiquette = TextComponent(
      text: 'Héros',
      position: Vector2(size.x / 2, -6),
      anchor: Anchor.bottomCenter,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 12, color: Color(0xFFFFFFFF)),
      ),
    );
    await add(_etiquette);
  }

  // 4. update : logique, toujours multipliée par dt.
  @override
  void update(double dt) {
    super.update(dt);

    position.x += vitesse * _sens * dt;

    if (position.x > 420) {
      position.x = 420;
      _sens = -1;
    } else if (position.x < 40) {
      position.x = 40;
      _sens = 1;
    }
  }

  // 5. render : coordonnées locales exclusivement.
  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _corps);
    canvas.drawRect(
      Rect.fromLTWH(0, size.y * 0.55, size.x, 6),
      _ceinture,
    );
  }

  void subirDegats(int montant) {
    pointsDeVie -= montant;
    if (pointsDeVie <= 0) {
      removeFromParent();
    }
  }
}
```

**Résultat :** un personnage doré à ceinture brune, surmonté de son nom, qui fait des allers-retours au-dessus du sol.

Quelques erreurs classiques à ne pas reproduire dans un composant personnalisé.

```dart
// FAUTE 1 : dessiner à la position absolue.
canvas.drawRect(Rect.fromLTWH(position.x, position.y, 36, 48), _corps);
// -> la position est appliquée deux fois. Utiliser size.toRect().

// FAUTE 2 : oublier super.update(dt).
@override
void update(double dt) {
  position.x += 1;   // les enfants ne sont plus mis à jour
}

// FAUTE 3 : oublier dt.
position.x += 2;     // vitesse dépendante du framerate (chapitre 20)

// FAUTE 4 : créer un Paint dans render.
canvas.drawRect(size.toRect(), Paint()..color = const Color(0xFFE8B04B));
// -> 60 objets Paint par seconde et par composant. Le mettre en static final.

// FAUTE 5 : accéder au parent dans le constructeur.
Joueur() { print(parent!.children.length); }   // parent vaut null ici
```

---

## 28.25 — Accéder au jeu depuis un composant

Un composant a souvent besoin de son jeu : pour lire la taille de l'écran, un score global, ou pour appeler une méthode comme `terminerLaPartie()`.

La façon moderne est le mixin `HasGameReference<T>`, qui fournit une propriété `game` **typée**.

```dart
mixin HasGameReference<T extends FlameGame> on Component {
  T get game;
  set game(T? value);
}
```

Utilisation :

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

class DonjonDeDart extends FlameGame {
  int score = 0;

  void ajouterPoints(int n) {
    score += n;
    print('Score : $score');
  }

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Piece(position: Vector2(100, 100)));
    await world.add(Piece(position: Vector2(160, 100)));
  }
}

// Le mixin donne accès à `game`, typé DonjonDeDart.
class Piece extends CircleComponent with HasGameReference<DonjonDeDart> {
  Piece({super.position})
      : super(
          radius: 10,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFFFC107),
        );

  double _vie = 2;

  @override
  void update(double dt) {
    super.update(dt);

    _vie -= dt;
    if (_vie <= 0) {
      // Accès direct et typé au jeu, sans transtypage.
      game.ajouterPoints(50);

      // On peut aussi lire la taille de la surface de jeu.
      if (position.x > game.size.x) {
        // hors écran
      }

      removeFromParent();
    }
  }
}
```

**Résultat :**

```text
Score : 50
Score : 100
```

Trois variantes existent selon ce que l'on veut atteindre.

| Mixin | Propriété fournie | Quand l'utiliser |
| --- | --- | --- |
| `HasGameReference<MonJeu>` | `game` | accéder au jeu racine |
| `HasWorldReference<MonMonde>` | `world` | accéder au monde typé |
| `HasAncestor<MonNiveau>` | `ancestor` | accéder à un ancêtre quelconque |
| `ParentIsA<MonParent>` | `parent` typé | le parent direct, garanti d'un type |

Sans mixin, il reste `findGame()`, qui renvoie un `FlameGame<World>?` non typé et nullable :

```dart
final jeu = findGame();
if (jeu != null) {
  print(jeu.size);
}
```

Deux règles de prudence.

**Règle 1 : jamais dans le constructeur.** `game` n'est fiable qu'à partir de `onLoad`, et vraiment sûr à partir de `onMount`.

**Règle 2 : ne pas en abuser.** Un composant qui appelle `game.` partout est fortement couplé au jeu. Préférez lui passer ce dont il a besoin par son constructeur, comme au chapitre 26 avec l'injection de dépendances. `game` reste utile pour la taille de l'écran et pour les événements globaux.

---

## 28.26 — Le piège des anciens tutoriels : `HasGameRef` / `gameRef`

Presque tous les tutoriels Flame antérieurs à 2023 écrivent ceci :

```dart
// CODE PÉRIMÉ — ne l'écrivez plus.
class Joueur extends PositionComponent with HasGameRef<MonJeu> {
  @override
  void update(double dt) {
    if (position.x > gameRef.size.x) {
      removeFromParent();
    }
  }
}
```

En Flame 1.38.0, le fichier `has_game_ref.dart` porte une annotation `@Deprecated` explicite : « Use HasGameReference instead. This mixin will be removed in a future version of Flame. » La propriété `gameRef` y est décrite comme « Equivalent to the [game] property ».

Autrement dit : cela **compile encore**, avec un avertissement, et cela **disparaîtra**.

La correction est mécanique.

```dart
// CODE ACTUEL
class Joueur extends PositionComponent with HasGameReference<MonJeu> {
  @override
  void update(double dt) {
    super.update(dt);
    if (position.x > game.size.x) {
      removeFromParent();
    }
  }
}
```

| Ancien | Actuel |
| --- | --- |
| `with HasGameRef<MonJeu>` | `with HasGameReference<MonJeu>` |
| `gameRef.size` | `game.size` |
| `gameRef.add(c)` | `game.world.add(c)` (voir 28.29) |
| `gameRef.camera.zoom = 2` | `game.camera.viewfinder.zoom = 2` |

Profitons-en pour lister les autres pièges d'un tutoriel périmé, que vous rencontrerez inévitablement en cherchant de l'aide en ligne.

| Ce qu'on lit dans un vieux tutoriel | Réalité en Flame 1.38.0 | Correction |
| --- | --- | --- |
| `class MonJeu extends BaseGame` | `BaseGame` n'existe plus depuis la 1.0 | `extends FlameGame` |
| `with HasTappables` sur le jeu | non exporté | `TapCallbacks` sur le composant |
| `with Tappable` sur un composant | non exporté | `TapCallbacks` |
| `bool onTapDown(TapDownInfo info)` | ancienne signature | `void onTapDown(TapDownEvent event)` |
| `game.camera.followComponent(j)` | méthode inexistante | `game.camera.follow(j)` |
| `game.camera.zoom = 2` | inexistant sur `CameraComponent` | `game.camera.viewfinder.zoom = 2` |
| `game.camera.worldBounds = rect` | inexistant | `game.camera.setBounds(shape)` |
| `add(monComposant)` pour une entité | fonctionne, mais court-circuite la caméra | `world.add(monComposant)` |
| `GameWidget(game: g, shrinkWrap: true)` | supprimé en 1.31.0 | encadrer d'un `SizedBox` |

> **À retenir.** Devant un exemple trouvé en ligne, votre premier réflexe doit être de chercher `gameRef`. S'il y est, l'exemple a au moins trois ans, et le reste du code est probablement périmé aussi.

---

## 28.27 — Requêtes dans l'arbre

Trouver un composant dans l'arbre est une opération courante : « tous les gobelins vivants », « le premier coffre », « le joueur ». Flame offre plusieurs outils, du plus simple au plus rapide.

### `whereType<T>()` — le plus simple

`children` est un `Iterable`, donc toutes les méthodes du chapitre 14 s'appliquent, dont `whereType`.

```dart
final gobelins = world.children.whereType<Gobelin>();
print('Gobelins restants : ${gobelins.length}');

for (final g in world.children.whereType<Gobelin>()) {
  g.reveiller();
}
```

C'est direct, toujours disponible, et parfaitement acceptable pour quelques dizaines d'éléments.

### `firstChild<T>()` et `lastChild<T>()`

```dart
final joueur = world.firstChild<Joueur>();
if (joueur != null) {
  print('Joueur trouvé en ${joueur.position}');
}

final dernierGobelin = world.lastChild<Gobelin>();
```

Les deux renvoient `T?` : pensez à traiter le `null` (chapitre 12).

### `children.query<T>()` — le plus rapide

`children` est un ensemble « interrogeable ». Si l'on **enregistre** un type au préalable, Flame maintient en permanence une liste dédiée, et la requête devient quasi instantanée.

```dart
class Niveau extends World {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // On déclare les types que l'on interrogera souvent.
    children.register<Gobelin>();
  }

  @override
  void update(double dt) {
    super.update(dt);
    // Requête sans coût de filtrage.
    final gobelins = children.query<Gobelin>();
    if (gobelins.isEmpty) {
      print('Salle nettoyée.');
    }
  }
}
```

### `descendants()` — chercher dans tout le sous-arbre

```dart
// Toutes les barres de vie du niveau, à n'importe quelle profondeur.
final barres = world.descendants().whereType<BarreDeVie>();
```

### `ComponentKey` — retrouver un composant nommé

```dart
// À la création :
final boss = Boss(key: ComponentKey.named('boss'));

// Plus tard, depuis n'importe où dans le jeu :
final leBoss = game.findByKeyName<Boss>('boss');
```

Programme complet comparant les approches.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoRequetes()));
}

class DemoRequetes extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    world.children.register<Gobelin>();

    await world.addAll([
      Joueur(position: Vector2(60, 120)),
      Gobelin(position: Vector2(160, 120)),
      Gobelin(position: Vector2(220, 120)),
      Gobelin(position: Vector2(280, 120)),
      Coffre(position: Vector2(360, 120), key: ComponentKey.named('coffre')),
    ]);

    print('whereType   : ${world.children.whereType<Gobelin>().length} gobelins');
    print('query       : ${world.children.query<Gobelin>().length} gobelins');
    print('firstChild  : ${world.firstChild<Joueur>()?.position}');
    print('descendants : ${world.descendants().length} descendants');
    print('par clé     : ${findByKeyName<Coffre>('coffre')?.position}');
  }
}

class Joueur extends RectangleComponent {
  Joueur({super.position})
      : super(
          size: Vector2(30, 40),
          paint: Paint()..color = const Color(0xFFE8B04B),
        );
}

class Gobelin extends RectangleComponent {
  Gobelin({super.position})
      : super(
          size: Vector2(26, 30),
          paint: Paint()..color = const Color(0xFF4CAF50),
        );
}

class Coffre extends RectangleComponent {
  Coffre({super.position, super.key})
      : super(
          size: Vector2(34, 26),
          paint: Paint()..color = const Color(0xFF6D4C41),
        );
}
```

**Résultat :**

```text
whereType   : 3 gobelins
query       : 3 gobelins
firstChild  : [60.0,120.0]
descendants : 5 descendants
par clé     : [360.0,120.0]
```

Le tableau aide à choisir.

| Besoin | Outil recommandé |
| --- | --- |
| Quelques éléments, code lisible | `children.whereType<T>()` |
| Requête à chaque frame sur beaucoup d'éléments | `children.register<T>()` puis `children.query<T>()` |
| Un seul élément unique | `firstChild<T>()` |
| Chercher en profondeur | `descendants().whereType<T>()` |
| Un composant précis, connu par son nom | `ComponentKey.named` + `findByKeyName<T>` |
| Le composant sous le doigt | `componentsAtPoint(point)` |

> **À retenir.** Interroger l'arbre reste plus lent que garder une référence directe. Si vous accédez au joueur à chaque frame, stockez-le dans un champ `late final Joueur joueur;` plutôt que de le rechercher soixante fois par seconde.

---

## 28.28 — `ComponentsNotifier` et les changements d'état

Il arrive qu'un élément d'interface doive réagir à l'apparition ou à la disparition d'un composant : « quand le joueur meurt, afficher Game Over », « quand il ne reste plus de gobelin, ouvrir la porte ».

La solution naïve consiste à tester à chaque frame :

```dart
// Fonctionne, mais interroge l'arbre 60 fois par seconde pour rien.
@override
void update(double dt) {
  super.update(dt);
  if (world.children.whereType<Gobelin>().isEmpty) {
    ouvrirLaPorte();
  }
}
```

Flame propose une solution réactive : le mixin `Notifier` sur le composant surveillé, et `componentsNotifier<T>()` côté jeu.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoNotifier()));
}

class DemoNotifier extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // On s'abonne aux changements concernant les Gobelin.
    final notificateur = componentsNotifier<Gobelin>()
      ..addListener(() {
        final restants = componentsNotifier<Gobelin>().components.length;
        print('Gobelins encore en vie : $restants');
      });

    // La variable est conservée pour la lisibilité de l'exemple.
    print('Écouteurs installés : ${notificateur.components.length} gobelins');

    await world.addAll([
      Gobelin(position: Vector2(100, 120)),
      Gobelin(position: Vector2(160, 120)),
      Gobelin(position: Vector2(220, 120)),
    ]);

    await world.add(Faucheuse());
  }
}

// Le mixin Notifier rend ce composant observable.
class Gobelin extends RectangleComponent with Notifier {
  Gobelin({super.position})
      : super(
          size: Vector2(26, 30),
          paint: Paint()..color = const Color(0xFF4CAF50),
        );
}

// Retire un gobelin par seconde.
class Faucheuse extends Component {
  double _t = 0;

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    if (_t < 1) return;
    _t = 0;
    parent?.firstChild<Gobelin>()?.removeFromParent();
  }
}
```

**Résultat :**

```text
Écouteurs installés : 0 gobelins
Gobelins encore en vie : 1
Gobelins encore en vie : 2
Gobelins encore en vie : 3
Gobelins encore en vie : 2
Gobelins encore en vie : 1
Gobelins encore en vie : 0
```

Trois points sur ce mécanisme.

**Le notificateur suit les montages et les retraits.** Chaque ajout et chaque suppression d'un `Gobelin` déclenche les écouteurs. C'est pour cela que l'on voit d'abord la liste se remplir, puis se vider.

**Un composant peut aussi notifier manuellement.** Appelez `notify()` depuis le composant lorsque son état interne change (perte de points de vie, changement de phase), et les écouteurs seront prévenus.

**`componentsNotifier<T>()` est un `ValueNotifier` de Flutter.** Il s'intègre donc directement avec les widgets, ce qui sera très utile pour les overlays du chapitre 35.

> **À retenir.** Pour trois gobelins, un test dans `update` suffit. Pour un HUD complet qui doit refléter l'état du monde, le notificateur évite des dizaines de sondages inutiles par seconde.

---

## 28.29 — Le `World` et pourquoi les entités y vivent

Vous avez vu depuis le début du chapitre que l'on écrit `world.add(...)` et non `add(...)`. Il est temps d'expliquer pourquoi.

`FlameGame` crée automatiquement deux composants et les relie :

- `game.world`, une instance de `World` ;
- `game.camera`, une instance de `CameraComponent` qui observe ce monde.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │        world.add(x)   CONTRE   game.add(x)                       │
  └──────────────────────────────────────────────────────────────────┘

   FlameGame
   ├── World  ◄────── observé par la caméra
   │   └── x            x subit le zoom, le déplacement, la rotation
   │                    de la caméra. C'est une ENTITÉ DU JEU.
   │
   ├── CameraComponent
   │   └── Viewport
   │       └── y        y est collé à l'écran. C'est du HUD.
   │
   └── z                z est dessiné hors caméra, en coordonnées
                        écran brutes. Cas rare et déconseillé.
```

Les conséquences en trois lignes de code :

```dart
world.add(gobelin);              // entité de jeu : suit la caméra
camera.viewport.add(scoreTexte); // HUD : fixe à l'écran
camera.backdrop.add(ciel);       // décor derrière le monde, fixe
add(quelqueChose);               // possible, mais court-circuite la caméra
```

Un exemple qui rend la différence visible : la caméra zoome, et l'on voit qui suit et qui ne suit pas.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DemoMonde()));
}

class DemoMonde extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Dans le MONDE : suit la caméra.
    await world.add(
      RectangleComponent(
        position: Vector2(100, 100),
        size: Vector2(60, 60),
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );

    // Dans le VIEWPORT : ne bouge jamais.
    await camera.viewport.add(
      TextComponent(
        text: 'HUD fixe',
        position: Vector2(16, 16),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 20, color: Color(0xFFFFFFFF)),
        ),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    // Zoom oscillant entre 0.6 et 1.4.
    camera.viewfinder.zoom = 1 + 0.4 * (_t % 4 < 2 ? _t % 2 : 2 - _t % 2);
  }
}
```

**Résultat :** le carré vert grandit et rétrécit avec le zoom ; le texte « HUD fixe » ne bouge pas d'un pixel.

Vous pouvez aussi définir votre **propre classe de monde**, ce qui est la bonne pratique dès qu'un niveau a une logique propre.

```dart
class Niveau1 extends World {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await addAll([
      Joueur(position: Vector2(60, 180)),
      Gobelin(position: Vector2(200, 180)),
    ]);
  }
}

class DonjonDeDart extends FlameGame<Niveau1> {
  DonjonDeDart() : super(world: Niveau1());
}
```

Changer de niveau revient alors à remplacer le monde de la caméra, ce qui retire d'un coup toutes les entités de l'ancien niveau. Vous pouvez également appliquer `HasCollisionDetection` à un `World` plutôt qu'au jeu, pour isoler un système de collision.

Le chapitre 31 traitera en détail de `CameraComponent`, `Viewfinder`, `Viewport`, `follow`, `zoom`, des différents types de viewport et de la construction complète d'un HUD. Pour l'instant, retenez la règle.

> **À retenir.** **Entités du jeu → `world`. Interface → `camera.viewport`. Décor fixe derrière → `camera.backdrop`.** Ajouter directement au `FlameGame` est réservé à des systèmes sans apparence.

---

## 28.30 — Mini-projet : le joueur, trois gobelins et un coffre, en composants

Assemblons tout le chapitre en un programme unique et exécutable. Il n'utilise **aucune image** : uniquement des rectangles, des cercles et du texte.

### Cahier des charges

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │            DONJON DE DART — SALLE 1, EN COMPOSANTS                │
  └──────────────────────────────────────────────────────────────────┘

  * Un sol, un fond et deux torches (décor, priorités négatives).
  * Un joueur qui fait des allers-retours, avec barre de vie et nom.
  * Trois gobelins qui patrouillent, chacun avec sa barre de vie.
  * Un coffre fermé qui s'ouvre quand tous les gobelins sont morts.
  * Un HUD fixe : score, gobelins restants, état du coffre.
  * Le joueur inflige des dégâts au gobelin qu'il croise.
  * Tri en profondeur : le personnage le plus bas est dessiné devant.
```

### L'arbre visé

```text
  DonjonDeDart (FlameGame)
  │
  ├── Salle (World)
  │   ├── Fond              priority -20
  │   ├── Sol               priority -10
  │   ├── Torche x2         priority -5
  │   ├── Coffre            priority  10
  │   │   ├── Caisse (RectangleComponent)
  │   │   ├── Serrure (RectangleComponent)
  │   │   └── Etiquette (TextComponent)
  │   ├── Joueur            priority = position.y
  │   │   ├── Corps (RectangleComponent)
  │   │   ├── Ceinture (RectangleComponent)
  │   │   ├── BarreDeVie
  │   │   │   ├── Fond (RectangleComponent)
  │   │   │   └── Remplissage (RectangleComponent)
  │   │   └── Etiquette (TextComponent)
  │   ├── Gobelin x3        priority = position.y
  │   │   ├── Corps, Oeil, BarreDeVie, Etiquette
  │   └── Arbitre (Component sans apparence)
  │
  └── CameraComponent
      └── Viewport
          └── Hud
              ├── TexteScore
              ├── TexteGobelins
              └── TexteCoffre
```

### Le programme complet

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(body: EcranDeJeu()),
    ),
  );
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

// ---------------------------------------------------------------------------
// 1. LES COUCHES DE RENDU
// ---------------------------------------------------------------------------

abstract final class Couches {
  static const int fond = -20;
  static const int sol = -10;
  static const int decor = -5;
  static const int objets = 10;
  // Les personnages reçoivent une priorité calculée à partir de leur y.
  static const int hud = 1000;
}

// ---------------------------------------------------------------------------
// 2. LE JEU
// ---------------------------------------------------------------------------

class DonjonDeDart extends FlameGame<Salle> {
  DonjonDeDart() : super(world: Salle());

  int score = 0;

  void ajouterPoints(int n) {
    score += n;
  }

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    camera.viewfinder.anchor = Anchor.topLeft;

    // Le HUD est un enfant du viewport : il reste fixe à l'écran.
    await camera.viewport.add(Hud()..priority = Couches.hud);
  }
}

// ---------------------------------------------------------------------------
// 3. LE MONDE
// ---------------------------------------------------------------------------

class Salle extends World {
  static const double largeur = 520;
  static const double hauteurSol = 250;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // On enregistre les types que l'on interrogera souvent.
    children.register<Gobelin>();
    children.register<Joueur>();

    // Décor.
    await addAll([
      RectangleComponent(
        position: Vector2.zero(),
        size: Vector2(largeur, 320),
        priority: Couches.fond,
        paint: Paint()..color = const Color(0xFF14141F),
      ),
      RectangleComponent(
        position: Vector2(0, hauteurSol),
        size: Vector2(largeur, 70),
        priority: Couches.sol,
        paint: Paint()..color = const Color(0xFF2E2A22),
      ),
      Torche(position: Vector2(70, 90)),
      Torche(position: Vector2(430, 90)),
    ]);

    // Le coffre.
    await add(Coffre(position: Vector2(430, hauteurSol)));

    // Le joueur.
    await add(Joueur(position: Vector2(80, hauteurSol)));

    // Trois gobelins.
    await addAll([
      Gobelin(
        nom: 'Grik',
        position: Vector2(200, hauteurSol),
        gauche: 170,
        droite: 300,
      ),
      Gobelin(
        nom: 'Snarl',
        position: Vector2(320, hauteurSol - 30),
        gauche: 260,
        droite: 390,
      ),
      Gobelin(
        nom: 'Vok',
        position: Vector2(150, hauteurSol - 60),
        gauche: 120,
        droite: 260,
      ),
    ]);

    // L'arbitre : composant sans apparence qui gère les combats.
    await add(Arbitre());
  }
}

// ---------------------------------------------------------------------------
// 4. LE DÉCOR
// ---------------------------------------------------------------------------

class Torche extends PositionComponent {
  Torche({super.position})
      : super(size: Vector2(10, 26), priority: Couches.decor);

  late final CircleComponent _flamme;
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0xFF4E342E),
      ),
    );

    _flamme = CircleComponent(
      radius: 8,
      position: Vector2(size.x / 2, 0),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFFFF9800),
    );
    await add(_flamme);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt * 4;
    // Une onde triangulaire qui monte de 0 à 1 puis redescend.
    final phase = _t % 2;
    final onde = phase < 1 ? phase : 2 - phase;
    // Une flamme qui vacille : le rayon oscille entre 6 et 9.
    _flamme.radius = 6 + 3 * onde;
  }
}

// ---------------------------------------------------------------------------
// 5. LA BARRE DE VIE (réutilisée par le joueur et les gobelins)
// ---------------------------------------------------------------------------

class BarreDeVie extends PositionComponent {
  BarreDeVie({required double largeur, super.position})
      : _largeur = largeur,
        super(size: Vector2(largeur, 5));

  final double _largeur;
  late final RectangleComponent _remplissage;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0xFF7F1D1D),
      ),
    );

    _remplissage = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF43A047),
    );
    await add(_remplissage);
  }

  void definirRatio(double ratio) {
    final r = ratio.clamp(0.0, 1.0);
    _remplissage.size.x = _largeur * r;
    _remplissage.paint.color =
        r > 0.5 ? const Color(0xFF43A047) : const Color(0xFFFFB300);
  }
}

// ---------------------------------------------------------------------------
// 6. LA BASE COMMUNE DES PERSONNAGES
// ---------------------------------------------------------------------------

abstract class Personnage extends PositionComponent {
  Personnage({
    required this.nom,
    required this.pointsDeVieMax,
    required Vector2 taille,
    super.position,
  }) : super(size: taille, anchor: Anchor.bottomLeft) {
    pointsDeVie = pointsDeVieMax;
  }

  final String nom;
  final int pointsDeVieMax;
  late int pointsDeVie;

  late final BarreDeVie barre;

  Color get couleur;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = couleur,
      ),
    );

    barre = BarreDeVie(
      largeur: size.x,
      position: Vector2(0, -10),
    );
    await add(barre);

    await add(
      TextComponent(
        text: nom,
        position: Vector2(size.x / 2, -14),
        anchor: Anchor.bottomCenter,
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 11, color: Color(0xFFECEFF1)),
        ),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    // Tri en profondeur : le plus bas est dessiné devant.
    priority = position.y.round();
  }

  bool get estVivant => pointsDeVie > 0;

  void subirDegats(int montant) {
    if (!estVivant) return;
    pointsDeVie = (pointsDeVie - montant).clamp(0, pointsDeVieMax);
    barre.definirRatio(pointsDeVie / pointsDeVieMax);
    if (pointsDeVie == 0) {
      mourir();
    }
  }

  void mourir() {
    removeFromParent();
  }
}

// ---------------------------------------------------------------------------
// 7. LE JOUEUR
// ---------------------------------------------------------------------------

class Joueur extends Personnage with HasGameReference<DonjonDeDart> {
  Joueur({super.position})
      : super(
          nom: 'Héros',
          pointsDeVieMax: 120,
          taille: Vector2(30, 44),
        );

  @override
  Color get couleur => const Color(0xFFE8B04B);

  double vitesse = 70;
  int _sens = 1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Une ceinture, pour reconnaître le héros au premier coup d'oeil.
    await add(
      RectangleComponent(
        position: Vector2(0, size.y * 0.55),
        size: Vector2(size.x, 5),
        paint: Paint()..color = const Color(0xFF5D4037),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    position.x += vitesse * _sens * dt;
    if (position.x > Salle.largeur - 90) {
      position.x = Salle.largeur - 90;
      _sens = -1;
    } else if (position.x < 40) {
      position.x = 40;
      _sens = 1;
    }
  }

  @override
  void mourir() {
    print('Le héros est tombé. Score final : ${game.score}');
    super.mourir();
  }
}

// ---------------------------------------------------------------------------
// 8. LES GOBELINS
// ---------------------------------------------------------------------------

class Gobelin extends Personnage with HasGameReference<DonjonDeDart> {
  Gobelin({
    required super.nom,
    required this.gauche,
    required this.droite,
    super.position,
  }) : super(pointsDeVieMax: 40, taille: Vector2(24, 28));

  @override
  Color get couleur => const Color(0xFF4CAF50);

  final double gauche;
  final double droite;

  double vitesse = 45;
  int _sens = 1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Un oeil, pour l'ambiance.
    await add(
      CircleComponent(
        radius: 3,
        position: Vector2(size.x * 0.7, size.y * 0.3),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF1B1B1B),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    position.x += vitesse * _sens * dt;
    if (position.x > droite) {
      position.x = droite;
      _sens = -1;
    } else if (position.x < gauche) {
      position.x = gauche;
      _sens = 1;
    }
  }

  @override
  void mourir() {
    game.ajouterPoints(25);
    print('$nom est vaincu. Score : ${game.score}');
    super.mourir();
  }
}

// ---------------------------------------------------------------------------
// 9. LE COFFRE
// ---------------------------------------------------------------------------

class Coffre extends PositionComponent {
  Coffre({super.position})
      : super(
          size: Vector2(44, 32),
          anchor: Anchor.bottomLeft,
          priority: Couches.objets,
        );

  bool ouvert = false;

  late final RectangleComponent _caisse;
  late final RectangleComponent _serrure;
  late final TextComponent _etiquette;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _caisse = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF6D4C41),
    );
    await add(_caisse);

    _serrure = RectangleComponent(
      position: Vector2(size.x / 2 - 4, size.y / 2 - 4),
      size: Vector2(8, 8),
      paint: Paint()..color = const Color(0xFFB0BEC5),
    );
    await add(_serrure);

    _etiquette = TextComponent(
      text: 'fermé',
      position: Vector2(size.x / 2, -6),
      anchor: Anchor.bottomCenter,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 11, color: Color(0xFF90A4AE)),
      ),
    );
    await add(_etiquette);
  }

  void ouvrir() {
    if (ouvert) return;
    ouvert = true;
    _caisse.paint.color = const Color(0xFFFFC107);
    _serrure.paint.color = const Color(0xFF4CAF50);
    _etiquette.text = 'OUVERT';
    print('Le coffre s\'ouvre.');
  }
}

// ---------------------------------------------------------------------------
// 10. L'ARBITRE : combats et condition de victoire
// ---------------------------------------------------------------------------

class Arbitre extends Component with HasGameReference<DonjonDeDart> {
  double _cadence = 0;

  @override
  void update(double dt) {
    super.update(dt);

    final salle = parent! as Salle;

    final joueur = salle.firstChild<Joueur>();
    final gobelins = salle.children.query<Gobelin>();

    // Ouverture du coffre.
    if (gobelins.isEmpty) {
      salle.firstChild<Coffre>()?.ouvrir();
      return;
    }

    if (joueur == null) return;

    // Un échange de coups toutes les 0,5 seconde au maximum.
    _cadence -= dt;
    if (_cadence > 0) return;

    for (final g in gobelins) {
      final distance = (g.absoluteCenter - joueur.absoluteCenter).length;
      if (distance < 34) {
        g.subirDegats(10);
        joueur.subirDegats(4);
        _cadence = 0.5;
        break;
      }
    }
  }
}

// ---------------------------------------------------------------------------
// 11. LE HUD
// ---------------------------------------------------------------------------

class Hud extends Component with HasGameReference<DonjonDeDart> {
  static final TextPaint _style = TextPaint(
    style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
  );

  late final TextComponent _score;
  late final TextComponent _restants;
  late final TextComponent _coffre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _score = TextComponent(
      text: 'Score : 0',
      position: Vector2(14, 12),
      textRenderer: _style,
    );
    _restants = TextComponent(
      text: 'Gobelins : 3',
      position: Vector2(14, 34),
      textRenderer: _style,
    );
    _coffre = TextComponent(
      text: 'Coffre : fermé',
      position: Vector2(14, 56),
      textRenderer: _style,
    );

    await addAll([_score, _restants, _coffre]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final salle = game.world;
    final nb = salle.children.query<Gobelin>().length;
    final coffreOuvert = salle.firstChild<Coffre>()?.ouvert ?? false;

    _score.text = 'Score : ${game.score}';
    _restants.text = 'Gobelins : $nb';
    _coffre.text = coffreOuvert ? 'Coffre : OUVERT' : 'Coffre : fermé';
  }
}
```

Relisez le bloc numéro 2 : c'est là que le HUD est branché. Il est ajouté à `camera.viewport`, et non au monde, ce qui le rend insensible aux mouvements de la caméra. Le `Hud`, lui, est déclaré tout à la fin du fichier, dans le bloc numéro 11 : en Dart, l'ordre de déclaration des classes dans un fichier n'a aucune importance.

**Résultat :**

```text
  ┌────────────────────────────────────────────────────────────────────┐
  │ Score : 50                                                         │
  │ Gobelins : 1                       ▓          ▓                    │
  │ Coffre : fermé                     ▒          ▒   <- torches       │
  │                                                                    │
  │                     Vok                                            │
  │                   ████░░                                           │
  │                   ▒▒▒▒▒▒                                           │
  │      Héros                                                         │
  │    ████████░░                                    fermé             │
  │    ▓▓▓▓▓▓▓▓▓▓                                   ▓▓▓▓▓▓             │
  │    ▓▓▓▓▓▓▓▓▓▓                                   ▓▓██▓▓             │
  │  ████████████████████████████████████████████████████████████████  │
  └────────────────────────────────────────────────────────────────────┘
```

**Console :**

```text
Grik est vaincu. Score : 25
Snarl est vaincu. Score : 50
Vok est vaincu. Score : 75
Le coffre s'ouvre.
```

### Ce que ce programme démontre

Reprenez le tableau, section par section, et vérifiez que vous voyez où chaque notion est employée.

| Notion du chapitre | Où elle apparaît dans le mini-projet |
| --- | --- |
| 28.1 tout est composant | même l'arbitre et le HUD sont des `Component` |
| 28.2 arbre | `Salle` contient les personnages, qui contiennent leurs pièces |
| 28.4 `Component` sans apparence | `Arbitre`, `Hud` |
| 28.5 `PositionComponent` | `Personnage`, `Coffre`, `Torche`, `BarreDeVie` |
| 28.6 propriétés | `size`, `position`, `anchor` sur chaque personnage |
| 28.7 ancre | `Anchor.bottomLeft` sur les personnages : les pieds au sol |
| 28.9 coordonnées absolues | `absoluteCenter` dans `Arbitre` pour la distance |
| 28.10 `add` / `addAll` | partout dans les `onLoad` |
| 28.12 `await add` | `barre` est attendue car utilisée par `subirDegats` |
| 28.13 retrait | `removeFromParent()` dans `mourir()` |
| 28.14 cycle de vie | `onLoad` puis `update` sur chaque classe |
| 28.17 parent et enfants | `parent! as Salle` dans `Arbitre` |
| 28.18 composition | `Personnage` = corps + barre + nom |
| 28.19 transformations héritées | déplacer un gobelin déplace sa barre et son nom |
| 28.20 `priority` | `Couches`, et le décor en priorités négatives |
| 28.21 priorité dynamique | `priority = position.y.round()` |
| 28.22 formes intégrées | rectangles, cercles, aucun fichier image |
| 28.23 `TextComponent` | noms, étiquette du coffre, HUD |
| 28.24 composant personnalisé | `Joueur`, `Gobelin`, `Coffre`, `Torche` |
| 28.25 `HasGameReference` | `Joueur`, `Gobelin`, `Arbitre`, `Hud` |
| 28.27 requêtes | `children.query<Gobelin>()`, `firstChild<Coffre>()` |
| 28.29 `World` | `Salle extends World`, HUD dans le viewport |

Comparez maintenant ce fichier avec celui de la section 26.32. Le nombre de lignes est comparable, mais toute la mécanique — file d'ajout, file de suppression, tri par profondeur, propagation de `update`, transformations imbriquées — a disparu de votre code. Elle est dans Flame.

---

## 28.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Rien ne s'affiche, l'écran reste noir | `super.onLoad()` oublié dans le `FlameGame` : le `World` et la caméra ne sont jamais montés | `await super.onLoad();` en première ligne de `onLoad` |
| Les enfants ne bougent plus | `super.update(dt)` oublié : la propagation s'arrête | appeler `super.update(dt)` en premier dans `update` |
| Les enfants ne sont plus dessinés | `super.render(canvas)` oublié | appeler `super.render(canvas)` en premier dans `render` |
| `children.length` vaut 0 juste après `add()` | `add()` est asynchrone : le montage a lieu entre deux frames | `await parent.add(c);` quand la suite en dépend |
| `LateInitializationError` sur un champ `late final` | le champ est utilisé avant que `await add(...)` l'ait monté | attendre l'ajout dans `onLoad` avant de s'en servir |
| Le composant est décalé d'une demi-taille | ancre par défaut `Anchor.topLeft` alors qu'on raisonnait en centre | fixer `anchor: Anchor.center` explicitement |
| Un objet tourne en décrivant une orbite | `angle` pivote autour de l'ancre, restée en haut à gauche | `anchor: Anchor.center` |
| Le composant part à 2578 degrés | `angle` est en radians, pas en degrés | `angle = degres * pi / 180` |
| `Null check operator used on a null value` dans le constructeur | `parent` et `game` valent `null` avant le montage | déplacer le code dans `onLoad` ou `onMount` |
| Avertissement de dépréciation sur `gameRef` | `HasGameRef` est déprécié depuis plusieurs versions | `with HasGameReference<MonJeu>` et `game` |
| Le composant ne réagit pas au tap | `size` vaut `Vector2.zero()` : aucun point n'est contenu | donner une `size` explicite |
| Le HUD se déplace avec le héros | il a été ajouté au `world` | l'ajouter à `camera.viewport` |
| Le décor passe devant le joueur | priorités égales, l'ordre d'ajout décide | fixer des `priority` distinctes |
| `Concurrent modification during iteration` | on modifie soi-même une liste pendant `update` | utiliser `add` / `removeFromParent`, jamais de liste parallèle |
| La position absolue calculée à la main est fausse | un ancêtre porte un `angle` ou un `scale` | utiliser `absolutePosition` / `absolutePositionOf` |
| Deux composants bougent ensemble sans raison | ils partagent la même instance de `Vector2` | `position: depart.clone()` |
| Les FPS s'effondrent | un `Paint` ou un `TextPaint` est créé dans `render` | les déclarer `static final` |
| `onLoad` semble ne pas se rejouer après un ré-ajout | c'est le comportement attendu : `onLoad` est unique | mettre le code de remontage dans `onMount` |
| `parent` vaut `null` dans `onGameResize` | `onGameResize` précède `onMount` | ne rien y faire qui suppose le montage |
| La barre de vie ne suit pas le personnage | elle a été ajoutée au `world`, pas au personnage | `personnage.add(barre)` |
| Un enfant en `(0, 0)` n'est pas au centre du parent centré | l'ancre du parent n'affecte pas ses enfants | positionner l'enfant à `size / 2` |
| `query<T>()` renvoie toujours vide | le type n'a pas été enregistré | `children.register<T>()` dans `onLoad` |

---

## 28.32 — Résumé du chapitre

### Les composants

| Composant | À quoi il sert | Propriétés clés |
| --- | --- | --- |
| `Component` | brique de base, sans apparence : contrôleurs, systèmes, regroupements | `parent`, `children`, `priority`, `key` |
| `PositionComponent` | tout ce qui a une place à l'écran | `position`, `size`, `scale`, `angle`, `anchor`, `nativeAngle` |
| `RectangleComponent` | rectangle coloré, brique de prototypage | `size`, `paint`, `paintLayers` |
| `CircleComponent` | disque coloré | `radius`, `scaledRadius`, `paint` |
| `PolygonComponent` | forme libre à au moins 3 sommets | `vertices`, `paint` |
| `TextComponent` | une ligne de texte | `text`, `textRenderer`, `anchor` |
| `TextBoxComponent` | texte multi-lignes, dialogues | `boxConfig`, `growingBox`, `skip()` |
| `World` | conteneur des entités observées par la caméra | `children` |
| `CameraComponent` | ce qui regarde le monde | `viewfinder`, `viewport`, `backdrop` |
| `TextPaint` | style de rendu du texte (ce n'est pas un composant) | `style` |

### Le cycle de vie

| Méthode | Fréquence | Ce qu'on y met |
| --- | --- | --- |
| constructeur | une fois | affectations simples uniquement |
| `onLoad()` | une fois | chargement, création des enfants, `await add(...)` |
| `onGameResize(size)` | avant le montage, puis à chaque redimension | repositionnement dépendant de l'écran |
| `onMount()` | à chaque montage | code qui a besoin de `parent` |
| `update(dt)` | ~60 fois/s | logique, toujours multipliée par `dt` |
| `render(canvas)` | ~60 fois/s | dessin en coordonnées locales, sans modifier l'état |
| `onRemove()` | une fois | nettoyage, désabonnements |

### Les opérations sur l'arbre

| Opération | Méthode | Remarque |
| --- | --- | --- |
| ajouter | `add`, `addAll`, paramètre `children` | asynchrone |
| retirer | `remove`, `removeAll`, `removeWhere`, `removeFromParent` | différé d'une frame |
| explorer | `parent`, `children`, `descendants()`, `ancestors()` | `children` est trié par `priority` |
| chercher | `whereType<T>()`, `firstChild<T>()`, `lastChild<T>()`, `query<T>()` | `query` exige `register<T>()` |
| identifier | `ComponentKey.named`, `findByKeyName<T>` | pratique pour un boss unique |
| observer | `Notifier`, `componentsNotifier<T>()` | réactif, sans sondage |
| accéder au jeu | `HasGameReference<T>`, `findGame()` | jamais dans le constructeur |

### Les conversions de coordonnées

| Question | Membre |
| --- | --- |
| où suis-je pour mon parent ? | `position` |
| où suis-je dans le monde ? | `absolutePosition` |
| où est mon centre dans le monde ? | `absoluteCenter` |
| où est ce point local, chez mon parent ? | `positionOf(p)` |
| où est ce point local, dans le monde ? | `absolutePositionOf(p)` |
| quel est mon rectangle dans le monde ? | `toAbsoluteRect()` |
| où est ce point du monde, chez moi ? | `absoluteToLocal(p)` |

### Les six règles à ne jamais oublier

```text
  1. super.onLoad(), super.update(dt), super.render(canvas) : toujours.
  2. add() est asynchrone : await dès que la suite en dépend.
  3. anchor vaut topLeft par défaut : fixez-la explicitement.
  4. angle est en radians.
  5. Les entités vont dans world, l'interface dans camera.viewport.
  6. render dessine en coordonnées LOCALES, jamais à position.x / position.y.
```

---

## 28.33 — Exercices

### Exercice 1 — Le premier composant (facile)

Écrivez un `FlameGame` nommé `Exercice1` qui affiche un unique carré bleu de 50 sur 50, positionné en (100, 80), ancré au centre. Utilisez `RectangleComponent`, sans créer de classe personnalisée.

### Exercice 2 — Les deux ancres (facile)

Affichez deux cercles rouges de rayon 20 à la même position déclarée `Vector2(150, 120)`, l'un ancré en `topLeft`, l'autre en `center`. Ajoutez, pour chacun, un petit carré blanc de 4 sur 4 ancré au centre et placé exactement à `Vector2(150, 120)`, afin de matérialiser le point déclaré. Observez le décalage.

### Exercice 3 — Le composant qui se déplace (facile)

Créez une classe `Potion extends CircleComponent` de rayon 10, de couleur violette, ancrée au centre. Elle descend de 60 pixels par seconde et se replace en haut (`y = 0`) dès qu'elle dépasse `y = 300`. Ajoutez-en cinq à des `x` différents.

### Exercice 4 — La composition parent-enfant (intermédiaire)

Créez une classe `Bouclier extends PositionComponent` de 60 sur 60, ancrée au centre, contenant deux enfants : un `CircleComponent` gris de rayon 30 centré, et un `RectangleComponent` doré de 8 sur 44 centré. Ajoutez un `Bouclier` au monde et faites-le tourner à raison d'un tour toutes les quatre secondes. Vérifiez que les deux enfants tournent ensemble.

### Exercice 5 — Le cycle de vie tracé (intermédiaire)

Écrivez un composant `Tracé extends PositionComponent` qui affiche dans la console, avec un préfixe numéroté, chacune des étapes suivantes : constructeur, `onLoad`, `onGameResize`, `onMount`, la première frame d'`update`, et `onRemove`. Le composant se retire tout seul après deux secondes. Vérifiez l'ordre obtenu.

### Exercice 6 — La barre d'énergie (intermédiaire)

Créez une classe `BarreEnergie extends PositionComponent` de 80 sur 8, composée d'un fond gris foncé et d'un remplissage bleu. Elle expose une méthode `definirRatio(double)`. Ajoutez-la à un `Joueur` carré, et faites baisser l'énergie de 15 % par seconde dans le `update` du joueur ; quand elle atteint zéro, elle remonte à 100 %.

### Exercice 7 — Le tri en profondeur (intermédiaire)

Créez cinq carrés de couleurs différentes, de taille 40 sur 60, ancrés en `bottomCenter`, placés à des `y` variés entre 100 et 220. Dans le `update` de chacun, réglez `priority = position.y.round()`. Faites osciller leur `y` de plus ou moins 40 pixels et vérifiez que l'ordre de rendu se réorganise tout seul.

### Exercice 8 — Le compteur de gobelins (difficile)

Écrivez un jeu contenant un `World` avec cinq `Gobelin` et un composant `Compteur extends Component` sans apparence. Le `Compteur` retire un gobelin toutes les 1,5 seconde, met à jour un `TextComponent` du HUD affichant « Gobelins : N », et affiche « SALLE NETTOYÉE » quand il n'en reste plus. Utilisez `children.register<Gobelin>()` et `children.query<Gobelin>()`.

### Exercice 9 — Les coordonnées absolues (difficile)

Construisez un `Bras` composé d'un `Epaule` (rectangle 40 sur 10, ancré `centerLeft`) contenant un `AvantBras` (rectangle 30 sur 8, ancré `centerLeft`, positionné à l'extrémité de l'épaule). Faites tourner les deux à des vitesses différentes. Toutes les secondes, affichez dans la console la position absolue de la pointe de l'avant-bras, obtenue avec `absolutePositionOf`.

### Exercice 10 — La salle du trésor (difficile)

Assemblez un programme complet : un sol, un `Joueur` ancré `bottomLeft` qui fait des allers-retours, trois `Coffre` alignés, et un composant `Ouvreur` sans apparence. Quand la distance entre le centre absolu du joueur et celui d'un coffre passe sous 40 pixels, le coffre s'ouvre (changement de couleur et de texte) et le score augmente de 100. Un HUD fixe affiche le score et le nombre de coffres encore fermés.

---

## 28.34 — Corrections des exercices

### Correction 1

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice1()));
}

class Exercice1 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Sans cette ligne, l'origine du monde serait au centre de l'écran.
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(
      RectangleComponent(
        position: Vector2(100, 80),
        size: Vector2(50, 50),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF2196F3),
      ),
    );
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │          ▓▓▓▓▓▓▓▓                                            │
  │          ▓▓▓▓▓▓▓▓   <- centre du carré exactement en (100,80)│
  │          ▓▓▓▓▓▓▓▓                                            │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

**Explication :** trois éléments sont obligatoires et suffisent. `await super.onLoad()` monte le `World` et la caméra ; sans lui, `world.add` resterait sans effet visible. `camera.viewfinder.anchor = Anchor.topLeft` place l'origine du monde en haut à gauche, ce qui rend les coordonnées lisibles pour un débutant. Enfin, `anchor: Anchor.center` fait que `position` désigne le centre du carré, et non son coin haut-gauche : le carré occupe donc `x` de 75 à 125 et `y` de 55 à 105. Aucune classe personnalisée n'est nécessaire : `RectangleComponent` est un composant complet, prêt à l'emploi.

---

### Correction 2

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice2()));
}

class Exercice2 extends FlameGame {
  static final Paint _rouge = Paint()..color = const Color(0xFFE53935);
  static final Paint _blanc = Paint()..color = const Color(0xFFFFFFFF);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Groupe 1 : ancre par défaut (topLeft).
    await world.addAll([
      CircleComponent(
        radius: 20,
        position: Vector2(150, 120),
        paint: _rouge,
      ),
      _marqueur(Vector2(150, 120)),
    ]);

    // Groupe 2 : ancre au centre, décalé de 150 px en x pour comparer.
    await world.addAll([
      CircleComponent(
        radius: 20,
        position: Vector2(300, 120),
        anchor: Anchor.center,
        paint: _rouge,
      ),
      _marqueur(Vector2(300, 120)),
    ]);

    // Légendes.
    final style = TextPaint(
      style: const TextStyle(fontSize: 13, color: Color(0xFFFFFFFF)),
    );
    await world.addAll([
      TextComponent(
        text: 'anchor: topLeft',
        position: Vector2(150, 200),
        anchor: Anchor.center,
        textRenderer: style,
      ),
      TextComponent(
        text: 'anchor: center',
        position: Vector2(300, 200),
        anchor: Anchor.center,
        textRenderer: style,
      ),
    ]);
  }

  // Un carré blanc de 4x4 centré sur le point déclaré.
  RectangleComponent _marqueur(Vector2 p) => RectangleComponent(
        position: p,
        size: Vector2.all(4),
        anchor: Anchor.center,
        priority: 10,
        paint: _blanc,
      );
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │              ▪●●●●●                    ●●●●●                 │
  │              ●●●●●●●                  ●●●●●●●                │
  │              ●●●●●●●                  ●●●▪●●●                │
  │              ●●●●●●●                  ●●●●●●●                │
  │               ●●●●●                    ●●●●●                 │
  │                                                              │
  │           anchor: topLeft           anchor: center           │
  └──────────────────────────────────────────────────────────────┘
```

**Explication :** le carré blanc matérialise le point `(150, 120)` ou `(300, 120)` déclaré dans `position`. Sur le disque de gauche, ce point tombe au coin haut-gauche de la boîte englobante du cercle — le disque est donc entièrement décalé vers le bas à droite. Sur celui de droite, il tombe en plein centre. Le décalage vaut exactement le rayon dans chaque direction, soit `(20, 20)`. Notez le `priority: 10` sur les marqueurs : sans lui, le carré blanc et le disque rouge auraient la même priorité, et le rendu dépendrait de l'ordre d'ajout. On rend ici le résultat déterministe.

---

### Correction 3

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice3()));
}

class Exercice3 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Cinq potions, à des abscisses différentes et à des hauteurs décalées
    // pour éviter qu'elles ne tombent toutes en même temps.
    for (var i = 0; i < 5; i++) {
      await world.add(
        Potion(position: Vector2(60.0 + i * 70, i * 45.0)),
      );
    }
  }
}

class Potion extends CircleComponent {
  Potion({super.position})
      : super(
          radius: 10,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF9C27B0),
        );

  static const double vitesse = 60; // pixels par seconde

  @override
  void update(double dt) {
    super.update(dt);

    // dt est en SECONDES : la vitesse est indépendante du framerate.
    position.y += vitesse * dt;

    if (position.y > 300) {
      position.y = 0;
    }
  }
}
```

**Résultat :** cinq disques violets descendent régulièrement, chacun réapparaissant en haut de l'écran dès qu'il dépasse `y = 300`.

**Explication :** on hérite directement de `CircleComponent`, ce qui évite d'écrire un `render` : la forme et la peinture sont déjà gérées. Le constructeur transmet `position` grâce à `super.position` (syntaxe des paramètres super vue au chapitre 9) et fixe le reste en dur. Deux points de vigilance : `super.update(dt)` en première ligne, faute de quoi d'éventuels enfants et effets ne seraient plus mis à jour ; et la multiplication par `dt`, sans laquelle la vitesse dépendrait du nombre d'images par seconde — le bug fondateur du chapitre 20. Enfin, `Anchor.center` fait que `position.y` désigne le centre du disque, donc le test `> 300` porte sur le centre.

---

### Correction 4

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice4()));
}

class Exercice4 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Bouclier(position: Vector2(180, 140)));
  }
}

class Bouclier extends PositionComponent {
  Bouclier({super.position})
      : super(size: Vector2(60, 60), anchor: Anchor.center);

  // Un tour complet en 4 secondes : 2*pi radians / 4 s.
  static const double vitesseAngulaire = 2 * pi / 4;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le disque gris. size / 2 = le centre du parent, car l'origine
    // locale d'un enfant est TOUJOURS le coin haut-gauche du parent.
    await add(
      CircleComponent(
        radius: 30,
        position: size / 2,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF90A4AE),
      ),
    );

    // La barre dorée, centrée elle aussi.
    await add(
      RectangleComponent(
        size: Vector2(8, 44),
        position: size / 2,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFFFC107),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    angle += vitesseAngulaire * dt;
  }
}
```

**Résultat :** un disque gris avec une barre dorée verticale, l'ensemble tournant régulièrement sur lui-même, un tour toutes les quatre secondes.

**Explication :** trois décisions font que cela fonctionne. D'abord `anchor: Anchor.center` sur le `Bouclier` : la rotation se fait autour du centre du composant, et non autour de son coin, ce qui donnerait une orbite. Ensuite `position: size / 2` sur les deux enfants : l'origine locale d'un enfant est le coin haut-gauche du parent, indépendamment de l'ancre du parent — c'est le piège de la section 28.19. Enfin, la rotation est appliquée **une seule fois**, sur le parent : les deux enfants héritent de la transformation sans une ligne de code. Notez aussi que `vitesseAngulaire` est exprimée en radians par seconde et multipliée par `dt`, comme toute grandeur de mouvement.

---

### Correction 5

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice5()));
}

class Exercice5 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Trace());
  }
}

class Trace extends PositionComponent {
  Trace() : super(position: Vector2(120, 100), size: Vector2(60, 60)) {
    print('[1] constructeur   parent=$parent isMounted=$isMounted');
  }

  double _temps = 0;
  bool _premiereFrame = true;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    print('[2] onLoad         parent=$parent isMounted=$isMounted');
  }

  @override
  void onGameResize(Vector2 tailleDuJeu) {
    super.onGameResize(tailleDuJeu);
    print('[3] onGameResize   tailleDuJeu=$tailleDuJeu isMounted=$isMounted');
  }

  @override
  void onMount() {
    super.onMount();
    print('[4] onMount        parent=${parent.runtimeType} isMounted=$isMounted');
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (_premiereFrame) {
      _premiereFrame = false;
      print('[5] update (1re)   dt=${dt.toStringAsFixed(4)} s');
    }

    _temps += dt;
    if (_temps >= 2) {
      removeFromParent();
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), Paint()..color = const Color(0xFF00BCD4));
  }

  @override
  void onRemove() {
    super.onRemove();
    print('[6] onRemove       isRemoved=$isRemoved');
  }
}
```

**Résultat :**

```text
[1] constructeur   parent=null isMounted=false
[2] onLoad         parent=null isMounted=false
[3] onGameResize   tailleDuJeu=[800.0,600.0] isMounted=false
[4] onMount        parent=World isMounted=true
[5] update (1re)   dt=0.0166 s
[6] onRemove       isRemoved=false
```

**Explication :** l'ordre confirme le schéma de la section 28.14. Trois observations méritent d'être retenues. `parent` vaut `null` dans le constructeur **et** dans `onLoad` : tout code qui a besoin du parent doit aller dans `onMount`. `onGameResize` est appelé même sans redimensionnement réel, simplement parce que le composant entre dans l'arbre, et il précède `onMount` — donc `isMounted` y vaut encore `false`. Enfin, dans `onRemove`, `isRemoved` vaut encore `false` : le composant est en train de partir, il n'est pas encore parti. Le drapeau `_premiereFrame` évite d'inonder la console de soixante lignes par seconde.

---

### Correction 6

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice6()));
}

class Exercice6 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Joueur(position: Vector2(140, 120)));
  }
}

class BarreEnergie extends PositionComponent {
  BarreEnergie({super.position}) : super(size: Vector2(80, 8));

  late final RectangleComponent _remplissage;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le fond, gris foncé.
    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0xFF37474F),
      ),
    );

    // Le remplissage, bleu, dont seule la largeur variera.
    _remplissage = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF29B6F6),
    );
    await add(_remplissage);
  }

  void definirRatio(double ratio) {
    _remplissage.size.x = size.x * ratio.clamp(0.0, 1.0);
  }
}

class Joueur extends PositionComponent {
  Joueur({super.position}) : super(size: Vector2(80, 60));

  double energie = 1.0;             // 100 %
  static const double perte = 0.15; // 15 % par seconde

  late final BarreEnergie _barre;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );

    _barre = BarreEnergie(position: Vector2(0, -14));
    // On ATTEND l'ajout, car definirRatio sera appelé dès la première frame.
    await add(_barre);
  }

  @override
  void update(double dt) {
    super.update(dt);

    energie -= perte * dt;
    if (energie <= 0) {
      energie = 1.0;
      print('Énergie rechargée.');
    }

    _barre.definirRatio(energie);
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │          ████████████░░░░░░░░   <- énergie à 60 %            │
  │          ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                                │
  │          ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓   <- le joueur                 │
  │          ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                                │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

**Console (toutes les ~6,7 secondes) :**

```text
Énergie rechargée.
Énergie rechargée.
```

**Explication :** la barre est composée de deux rectangles superposés : un fond qui ne change jamais, et un remplissage dont on modifie uniquement `size.x`. C'est bien plus économique que de redessiner à la main dans un `render`. Le point critique est le `await add(_barre)` : `_barre` est un champ `late final` utilisé dès la première frame d'`update`, donc il doit être monté avant. Sans le `await`, la première frame lèverait une `LateInitializationError`. Notez enfin que le ratio est **calculé** (`energie`), pas stocké en pixels : la barre ne connaît pas les règles du jeu, elle sait seulement afficher un pourcentage. C'est la séparation données / rendu du chapitre 26.

---

### Correction 7

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice7()));
}

class Exercice7 extends FlameGame {
  static const List<Color> _couleurs = [
    Color(0xFFE53935),
    Color(0xFF43A047),
    Color(0xFF1E88E5),
    Color(0xFFFDD835),
    Color(0xFF8E24AA),
  ];

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    for (var i = 0; i < 5; i++) {
      await world.add(
        Personnage(
          couleur: _couleurs[i],
          position: Vector2(80.0 + i * 55, 100.0 + i * 30),
          dephasage: i * 0.7,
        ),
      );
    }
  }
}

class Personnage extends RectangleComponent {
  Personnage({
    required Color couleur,
    required Vector2 position,
    required this.dephasage,
  })  : _yBase = position.y,
        super(
          position: position,
          size: Vector2(40, 60),
          anchor: Anchor.bottomCenter,
          paint: Paint()..color = couleur,
        );

  final double dephasage;
  final double _yBase;

  double _t = 0;

  @override
  void update(double dt) {
    super.update(dt);

    _t += dt;
    // Oscillation verticale de plus ou moins 40 pixels.
    position.y = _yBase + 40 * sin(_t + dephasage);

    // Tri en profondeur : plus le pied est bas, plus on est devant.
    priority = position.y.round();
  }
}
```

**Résultat :** cinq rectangles colorés montent et descendent en se croisant. À chaque croisement, celui dont la base est la plus basse passe devant l'autre, sans aucune condition écrite à la main.

**Explication :** l'astuce tient en une ligne, `priority = position.y.round()`. Comme `priority` est un `int` et que `position.y` est un `double`, l'arrondi est obligatoire. Flame maintient l'ensemble des enfants trié en permanence : dès que la priorité change, le composant est réinséré au bon endroit, et le changement s'applique **avant** le rendu de la frame en cours. L'ancre `Anchor.bottomCenter` est essentielle ici : elle fait que `position.y` désigne les **pieds** du personnage, ce qui est exactement le critère de profondeur recherché dans un jeu vu de dessus. Avec l'ancre par défaut, on trierait par le sommet de la tête, ce qui donnerait un résultat faux pour des personnages de tailles différentes.

---

### Correction 8

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice8()));
}

class Exercice8 extends FlameGame<Salle> {
  Exercice8() : super(world: Salle());

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await camera.viewport.add(Hud());
  }
}

class Salle extends World {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Enregistrement du type : rend children.query<Gobelin>() immédiat.
    children.register<Gobelin>();

    for (var i = 0; i < 5; i++) {
      await add(Gobelin(position: Vector2(60.0 + i * 70, 150)));
    }

    await add(Compteur());
  }
}

class Gobelin extends RectangleComponent {
  Gobelin({super.position})
      : super(
          size: Vector2(30, 36),
          anchor: Anchor.bottomLeft,
          paint: Paint()..color = const Color(0xFF4CAF50),
        );
}

// Composant sans apparence : il ne fait que de la logique.
class Compteur extends Component {
  static const double periode = 1.5;

  double _t = 0;
  bool _annonceFaite = false;

  @override
  void update(double dt) {
    super.update(dt);

    if (_annonceFaite) return;

    _t += dt;
    if (_t < periode) return;
    _t = 0;

    final salle = parent! as Salle;
    final gobelins = salle.children.query<Gobelin>();

    if (gobelins.isEmpty) {
      _annonceFaite = true;
      print('SALLE NETTOYÉE');
      return;
    }

    gobelins.first.removeFromParent();
  }
}

class Hud extends Component with HasGameReference<Exercice8> {
  static final TextPaint _style = TextPaint(
    style: const TextStyle(fontSize: 18, color: Color(0xFFFFFFFF)),
  );

  late final TextComponent _texte;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    _texte = TextComponent(
      text: 'Gobelins : 5',
      position: Vector2(14, 12),
      textRenderer: _style,
    );
    await add(_texte);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final nb = game.world.children.query<Gobelin>().length;
    _texte.text = nb == 0 ? 'SALLE NETTOYÉE' : 'Gobelins : $nb';
  }
}
```

**Résultat (console, une ligne toutes les 1,5 seconde après le dernier retrait) :**

```text
SALLE NETTOYÉE
```

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Gobelins : 3                                                │
  │                                                              │
  │                       ▓▓▓   ▓▓▓   ▓▓▓                        │
  │                       ▓▓▓   ▓▓▓   ▓▓▓                        │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

**Explication :** quatre points structurent cette correction. `children.register<Gobelin>()` est appelé dans le `onLoad` du monde, avant tout ajout : Flame maintient dès lors une liste dédiée aux `Gobelin`, et `query<Gobelin>()` la renvoie sans filtrage. Le `Compteur` hérite de `Component`, pas de `PositionComponent`, parce qu'il n'a rigoureusement aucune apparence — c'est le cas d'usage canonique de la section 28.4. Le HUD est ajouté à `camera.viewport` et non au monde, donc il reste fixe. Enfin, `game.world` est typé `Salle` grâce à `FlameGame<Salle>` : on peut appeler `children.query` dessus sans transtypage. Le drapeau `_annonceFaite` empêche de répéter le message à l'infini.

---

### Correction 9

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: Exercice9()));
}

class Exercice9 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Epaule(position: Vector2(200, 150)));
  }
}

class Epaule extends RectangleComponent {
  Epaule({super.position})
      : super(
          size: Vector2(40, 10),
          anchor: Anchor.centerLeft,
          paint: Paint()..color = const Color(0xFF8D6E63),
        );

  late final AvantBras avantBras;

  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // L'origine locale d'un enfant est le coin haut-gauche du parent.
    // L'extrémité droite de l'épaule est donc en (40, 5).
    avantBras = AvantBras(position: Vector2(40, 5));
    await add(avantBras);
  }

  @override
  void update(double dt) {
    super.update(dt);

    angle += 0.6 * dt;   // l'épaule tourne lentement

    _t += dt;
    if (_t >= 1) {
      _t = 0;
      // La pointe de l'avant-bras est son point local (30, 4).
      final pointe = avantBras.absolutePositionOf(Vector2(30, 4));
      print('pointe absolue : '
          '(${pointe.x.toStringAsFixed(1)}, ${pointe.y.toStringAsFixed(1)})'
          '   angle épaule=${angle.toStringAsFixed(2)} rad'
          '   angle absolu avant-bras=${avantBras.absoluteAngle.toStringAsFixed(2)} rad');
    }
  }
}

class AvantBras extends RectangleComponent {
  AvantBras({super.position})
      : super(
          size: Vector2(30, 8),
          anchor: Anchor.centerLeft,
          paint: Paint()..color = const Color(0xFFD7CCC8),
        );

  @override
  void update(double dt) {
    super.update(dt);
    angle += 1.8 * dt;   // trois fois plus vite que l'épaule
  }
}
```

**Résultat (console, une ligne par seconde) :**

```text
pointe absolue : (255.8, 209.2)   angle épaule=0.60 rad   angle absolu avant-bras=2.40 rad
pointe absolue : (216.1, 189.1)   angle épaule=1.20 rad   angle absolu avant-bras=4.80 rad
pointe absolue : (177.0, 158.6)   angle épaule=1.80 rad   angle absolu avant-bras=7.20 rad
```

**Explication :** cet exercice montre pourquoi il ne faut jamais additionner les positions à la main. La pointe de l'avant-bras subit deux rotations successives : celle de l'avant-bras autour de son propre point d'ancrage, puis celle de l'épaule autour du sien. Une simple addition `epaule.position + avantBras.position + Vector2(30, 4)` donnerait un résultat faux dès que l'un des deux angles est non nul. `absolutePositionOf(Vector2(30, 4))` compose correctement toute la chaîne de transformations depuis la racine du monde. On vérifie au passage que `absoluteAngle` cumule bien les angles : l'avant-bras tourne trois fois plus vite, donc son angle absolu vaut quatre fois celui de l'épaule (le sien plus celui du parent). Les valeurs exactes affichées dépendent de la cadence réelle des frames ; seul l'ordre de grandeur importe.

---

### Correction 10

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: SalleDuTresor()));
}

// ---------------------------------------------------------------------------
// LE JEU
// ---------------------------------------------------------------------------

class SalleDuTresor extends FlameGame<Tresorerie> {
  SalleDuTresor() : super(world: Tresorerie());

  int score = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await camera.viewport.add(Hud());
  }
}

// ---------------------------------------------------------------------------
// LE MONDE
// ---------------------------------------------------------------------------

class Tresorerie extends World {
  static const double niveauSol = 220;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    children.register<Coffre>();

    // Le sol, en arrière-plan.
    await add(
      RectangleComponent(
        position: Vector2(0, niveauSol),
        size: Vector2(520, 60),
        priority: -10,
        paint: Paint()..color = const Color(0xFF2E2A22),
      ),
    );

    // Trois coffres alignés.
    await addAll([
      Coffre(position: Vector2(140, niveauSol)),
      Coffre(position: Vector2(260, niveauSol)),
      Coffre(position: Vector2(380, niveauSol)),
    ]);

    // Le joueur.
    await add(Joueur(position: Vector2(40, niveauSol)));

    // L'ouvreur : logique pure, aucune apparence.
    await add(Ouvreur());
  }
}

// ---------------------------------------------------------------------------
// LE JOUEUR
// ---------------------------------------------------------------------------

class Joueur extends PositionComponent {
  Joueur({super.position})
      : super(size: Vector2(30, 44), anchor: Anchor.bottomLeft);

  static final Paint _corps = Paint()..color = const Color(0xFFE8B04B);

  double vitesse = 80;
  int _sens = 1;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await add(
      RectangleComponent(size: size.clone(), paint: _corps),
    );

    await add(
      TextComponent(
        text: 'Héros',
        position: Vector2(size.x / 2, -4),
        anchor: Anchor.bottomCenter,
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 11, color: Color(0xFFECEFF1)),
        ),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    position.x += vitesse * _sens * dt;
    if (position.x > 460) {
      position.x = 460;
      _sens = -1;
    } else if (position.x < 20) {
      position.x = 20;
      _sens = 1;
    }
  }
}

// ---------------------------------------------------------------------------
// LES COFFRES
// ---------------------------------------------------------------------------

class Coffre extends PositionComponent {
  Coffre({super.position})
      : super(size: Vector2(40, 30), anchor: Anchor.bottomLeft);

  bool ouvert = false;

  late final RectangleComponent _caisse;
  late final TextComponent _etiquette;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _caisse = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFF6D4C41),
    );
    await add(_caisse);

    _etiquette = TextComponent(
      text: 'fermé',
      position: Vector2(size.x / 2, -4),
      anchor: Anchor.bottomCenter,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 11, color: Color(0xFF90A4AE)),
      ),
    );
    await add(_etiquette);
  }

  // Renvoie true seulement lors de la PREMIÈRE ouverture.
  bool ouvrir() {
    if (ouvert) return false;
    ouvert = true;
    _caisse.paint.color = const Color(0xFFFFC107);
    _etiquette.text = 'OUVERT';
    return true;
  }
}

// ---------------------------------------------------------------------------
// L'OUVREUR
// ---------------------------------------------------------------------------

class Ouvreur extends Component with HasGameReference<SalleDuTresor> {
  static const double portee = 40;

  @override
  void update(double dt) {
    super.update(dt);

    final monde = parent! as Tresorerie;
    final joueur = monde.firstChild<Joueur>();
    if (joueur == null) return;

    for (final coffre in monde.children.query<Coffre>()) {
      if (coffre.ouvert) continue;

      // On compare des CENTRES ABSOLUS : les ancres n'ont plus d'importance.
      final distance =
          coffre.absoluteCenter.distanceTo(joueur.absoluteCenter);

      if (distance < portee && coffre.ouvrir()) {
        game.score += 100;
        print('Coffre ouvert. Score : ${game.score}');
      }
    }
  }
}

// ---------------------------------------------------------------------------
// LE HUD
// ---------------------------------------------------------------------------

class Hud extends Component with HasGameReference<SalleDuTresor> {
  static final TextPaint _style = TextPaint(
    style: const TextStyle(fontSize: 17, color: Color(0xFFFFFFFF)),
  );

  late final TextComponent _score;
  late final TextComponent _restants;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    _score = TextComponent(
      text: 'Score : 0',
      position: Vector2(14, 12),
      textRenderer: _style,
    );
    _restants = TextComponent(
      text: 'Coffres fermés : 3',
      position: Vector2(14, 34),
      textRenderer: _style,
    );

    await addAll([_score, _restants]);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final fermes = game.world.children
        .query<Coffre>()
        .where((c) => !c.ouvert)
        .length;

    _score.text = 'Score : ${game.score}';
    _restants.text = 'Coffres fermés : $fermes';
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │ Score : 200                                                      │
  │ Coffres fermés : 1                                               │
  │                                                                  │
  │                                                                  │
  │            OUVERT      OUVERT       fermé                        │
  │   Héros    ██████      ██████       ▓▓▓▓▓▓                       │
  │   ▓▓▓▓     ██████      ██████       ▓▓▓▓▓▓                       │
  │   ▓▓▓▓                                                           │
  │ ████████████████████████████████████████████████████████████████ │
  └──────────────────────────────────────────────────────────────────┘
```

**Console :**

```text
Coffre ouvert. Score : 100
Coffre ouvert. Score : 200
Coffre ouvert. Score : 300
```

**Explication :** ce programme réunit huit notions du chapitre, et chacune répond à un besoin précis.

**L'ancre `bottomLeft`** sur le joueur et les coffres fait que `position.y` désigne le sol : poser un objet au sol devient une simple affectation `position.y = niveauSol`.

**Les centres absolus** sont indispensables pour la distance. Le joueur mesure 30 sur 44, les coffres 40 sur 30, et tous sont ancrés en bas à gauche : comparer directement les `position` reviendrait à comparer des coins, et la portée mesurée changerait selon la taille des objets. `absoluteCenter.distanceTo(...)` compare deux centres, indépendamment des ancres et de la profondeur dans l'arbre.

**Le retour booléen de `ouvrir()`** garantit que le score n'augmente qu'une seule fois par coffre, même si le joueur repasse cinquante fois à portée. C'est un garde-fou classique, à ne jamais oublier dans une boucle de jeu qui tourne soixante fois par seconde.

**Le `continue`** en tête de boucle évite de recalculer une distance inutile pour un coffre déjà ouvert.

**Le composant `Ouvreur`**, sans apparence, isole toute la règle du jeu. Ni le joueur ni les coffres ne savent qu'un score existe : on pourrait remplacer la règle d'ouverture sans toucher à une seule ligne des autres classes.

**`children.register<Coffre>()`** rend `query<Coffre>()` gratuit, alors qu'il est appelé soixante fois par seconde, à la fois par l'`Ouvreur` et par le `Hud`.

**Le HUD dans `camera.viewport`** reste fixe même si l'on ajoute plus tard un `camera.follow(joueur)`.

**`FlameGame<Tresorerie>`** donne un `game.world` typé, ce qui supprime tout transtypage dans le `Hud`.

---

## Et maintenant ?

Faisons le point. Vous savez désormais construire un jeu entier sous forme d'arbre de composants.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │                    CE QUE VOUS MAÎTRISEZ                         │
  └──────────────────────────────────────────────────────────────────┘

  L'ARBRE          Component, PositionComponent, parent, children,
                   descendants, ancestors, World, viewport

  LA GÉOMÉTRIE     position, size, scale, angle, anchor,
                   coordonnées locales et absolues, conversions

  LE CYCLE DE VIE  onLoad, onGameResize, onMount, update, render,
                   onRemove, loaded, mounted, removed

  LES OPÉRATIONS   add, addAll, remove, removeFromParent, removeWhere,
                   priority statique et dynamique

  LES OUTILS       RectangleComponent, CircleComponent,
                   PolygonComponent, TextComponent, TextPaint

  L'ACCÈS          HasGameReference, ParentIsA, HasAncestor,
                   query, firstChild, ComponentKey, ComponentsNotifier
```

Il manque encore une chose, et elle saute aux yeux : **votre donjon est fait de rectangles**. Le héros est un carré doré, les gobelins des carrés verts, le coffre un carré brun. C'était voulu : aucune image n'était nécessaire pour comprendre l'arbre de composants, et vous avez pu tout exécuter sans rien télécharger.

Le chapitre 29 remplace ces formes par de vraies images. Vous y découvrirez :

- le cache d'images de Flame et la méthode `images.load` ;
- le dossier `assets/images/` et sa déclaration dans `pubspec.yaml` ;
- `SpriteComponent`, qui affiche une image à la place de votre `render` ;
- `SpriteAnimationComponent`, qui enchaîne les images d'une sprite sheet ;
- `SpriteAnimationData.sequenced`, pour découper une planche en frames ;
- les atlas et le regroupement d'images pour les performances ;
- où trouver des sprites libres de droits, sur Kenney.nl, itch.io et OpenGameArt ;
- et, comme toujours, une solution de repli entièrement en code, pour que le chapitre reste exécutable sans le moindre fichier.

Vous y retrouverez tout ce chapitre-ci, sans exception. `SpriteComponent` **est** un `PositionComponent` : il a une `position`, une `size`, une `anchor`, une `priority`, un `onLoad` et des enfants. Vous ne réapprendrez rien : vous changerez seulement la classe de base de vos personnages et supprimerez leur `render`.

Deux conseils avant d'y aller.

**Gardez votre mini-projet de la section 28.30.** Le chapitre 29 s'en servira comme point de départ : vous remplacerez les `RectangleComponent` du corps par des sprites, et rien d'autre ne bougera. C'est la meilleure démonstration possible de l'intérêt de la composition.

**Refaites l'exercice 10 sans regarder la correction.** S'il sort du premier coup, l'arbre de composants est acquis, et la suite de la PARTIE 2B se déroulera sans accroc.

Rendez-vous au chapitre suivant : [29-PARTIE-2B—SPRITES-ANIMATIONS-ET-ASSETS.md](./29-PARTIE-2B—SPRITES-ANIMATIONS-ET-ASSETS.md)
