# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 31 — CAMÉRA, VIEWPORT ET MONDE

> **Niveau :** intermédiaire
> **Durée estimée :** 9 h
> **Pré-requis :** chapitres 27 à 30 (installation de Flame, `FlameGame`, composants et cycle de vie, sprites, entrées), chapitre 25 (caméra écrite à la main, monde et écran, parallaxe), chapitre 28 (arbre de composants, `priority`, ancres)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** afficher un donjon plus grand que l'écran, faire suivre le héros par la caméra sans secousse, borner le regard aux murs du niveau, poser un HUD qui ne bouge jamais, et convertir un clic d'écran en position de monde.

---

## 31.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer, sans hésiter, la différence entre **coordonnées de monde** et **coordonnées d'écran** ;
- nommer les trois pièces du système de caméra de Flame : `CameraComponent`, `World`, `Viewport` ;
- dessiner l'arbre de composants d'un `FlameGame` et y placer la caméra et le monde ;
- expliquer **pourquoi** Flame a remplacé son ancienne caméra et ce que le nouveau modèle rend possible ;
- reconnaître en trois secondes un tutoriel périmé grâce à une liste de marqueurs (`BaseGame`, `followComponent`, `camera.zoom`, `worldBounds`) ;
- créer un `World`, le sous-classer, et le passer au constructeur de `FlameGame` ;
- justifier pourquoi une entité doit être ajoutée à `world` et non à `game` ;
- construire un `CameraComponent(world: ...)` à la main ;
- utiliser `CameraComponent.withFixedResolution()` et décrire l'effet de bandes noires ;
- manipuler le `viewfinder` : `position`, `zoom`, `angle`, `anchor` ;
- placer le regard de la caméra sur un point précis du monde ;
- régler un zoom et comprendre ce qu'il change au nombre de tuiles visibles ;
- déplacer le point d'ancrage pour laisser de la place devant le héros ;
- appeler `camera.follow()` et lire correctement sa signature ;
- lisser un suivi avec `maxSpeed` et choisir une valeur cohérente avec la vitesse du héros ;
- savoir que `snapTo()` **n'existe plus** et connaître ses trois remplaçants ;
- utiliser `camera.moveTo()` et `camera.moveBy()` pour une transition scriptée ;
- borner la caméra avec `setBounds()` et un `Rectangle` de `package:flame/experimental.dart` ;
- choisir entre `MaxViewport`, `FixedResolutionViewport`, `FixedAspectRatioViewport`, `FixedSizeViewport` et `CircularViewport` ;
- poser un HUD avec `camera.viewport.add()` et expliquer pourquoi il ne suit pas la caméra ;
- démontrer, code à l'appui, ce qui casse quand le HUD est mis dans le monde ;
- convertir un point d'écran en point de monde avec `camera.globalToLocal()` ;
- convertir un point de monde en point d'écran avec `camera.localToGlobal()` ;
- secouer l'écran avec un `MoveEffect` posé sur le `viewfinder` ;
- afficher deux caméras sur le même monde : minimap et écran partagé ;
- placer un fond parallaxe dans le monde ou dans le `backdrop` ;
- dire précisément ce que Flame **cull** automatiquement et ce qu'il ne cull pas, et écrire le culling manquant ;
- respecter la règle du **zoom entier** en pixel art et éviter les pixels baveux ;
- assembler un donjon plus grand que l'écran, avec caméra qui suit, barre de vie fixe et minimap.

---

## 31.1 — Rappel du chapitre 25 : monde et écran

Au chapitre 25, vous avez écrit une caméra à la main. Pas une caméra de moteur : une classe de trente lignes, avec deux `double` et une soustraction. Ce travail n'a pas été perdu. Il vous a donné le seul modèle mental qui compte, et Flame ne fait rien d'autre que l'automatiser.

Reprenons ce modèle, parce que tout le chapitre en dépend.

Un jeu manipule **deux repères différents**.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │              LE MONDE  (2400 × 1400 pixels de donjon)        │
  │                                                              │
  │   (0,0)                                                      │
  │    ┌───────────────────────────────────────────────────┐     │
  │    │                                                   │     │
  │    │        ┌═════════════════════════┐                │     │
  │    │        ║  L'ÉCRAN (800 × 450)    ║                │     │
  │    │        ║                         ║                │     │
  │    │        ║        ♦ héros          ║   ○ gobelin    │     │
  │    │        ║   (1180, 620) en monde  ║   (1900, 300)  │     │
  │    │        ║   ( 380, 205) à l'écran ║                │     │
  │    │        ╚═════════════════════════╝                │     │
  │    │                                                   │     │
  │    │   coin haut-gauche de l'écran, en monde : (800,415)     │
  │    └───────────────────────────────────────────────────┘     │
  └──────────────────────────────────────────────────────────────┘
```

| Repère | Origine | Ce qu'on y stocke | Change quand… |
| --- | --- | --- | --- |
| **Monde** | coin haut-gauche du niveau | la position des entités : héros, gobelins, potions, murs | l'entité se déplace |
| **Écran** | coin haut-gauche du `GameWidget` | rien de durable : c'est une vue | la caméra bouge, l'utilisateur redimensionne la fenêtre |

Et la formule, apprise au chapitre 25, section 25.3 :

```text
  écran = monde - caméra          (monde vers écran)
  monde = écran + caméra          (écran vers monde)
```

Avec un zoom, elle devient :

```text
  écran = (monde - caméra) × zoom
  monde =  écran / zoom + caméra
```

Voici, sous forme de tableau, ce que vous codiez à la main au chapitre 25 et ce qui devient un appel de méthode au chapitre 31.

| Chapitre 25, à la main | Chapitre 31, avec Flame |
| --- | --- |
| une classe `Camera` avec `x`, `y` | `camera.viewfinder.position` |
| `canvas.save()` puis `canvas.translate(-cam.x, -cam.y)` | fait automatiquement par `CameraComponent` |
| `canvas.scale(zoom)` | `camera.viewfinder.zoom` |
| `cam.x = joueur.x - largeur / 2` à chaque frame | `camera.follow(joueur)` |
| un `lerp` maison pour lisser le suivi | `camera.follow(joueur, maxSpeed: 300)` |
| un `clamp` sur `cam.x` et `cam.y` | `camera.setBounds(Rectangle.fromLTRB(...))` |
| `canvas.restore()` avant de dessiner le HUD | `camera.viewport.add(...)` |
| une double boucle qui saute les tuiles hors écran | `camera.canSee()` et `camera.visibleWorldRect` |
| trois `Rect` et trois vitesses pour la parallaxe | `ParallaxComponent`, ou le même calcul dans le monde |

> **À retenir.** Flame ne change pas la théorie. Il vous évite d'écrire `canvas.save()` / `canvas.translate()` / `canvas.restore()` à la main et, surtout, d'oublier l'un des trois.

Un dernier rappel avant d'attaquer. Au chapitre 25, le bug le plus fréquent était le **HUD qui se déplace avec le monde** : vous dessiniez la barre de vie après `canvas.translate()` mais avant `canvas.restore()`. Résultat : la barre de vie glissait hors de l'écran dès que le héros avançait. Ce bug existe toujours en Flame, sous une forme nouvelle : ajouter le HUD au `world` au lieu du `viewport`. La section 31.20 y est entièrement consacrée.

---

## 31.2 — Le trio `CameraComponent` / `World` / `Viewport`

Flame découpe la question « qu'est-ce que je vois ? » en trois objets. Chacun a **une seule** responsabilité. C'est cette séparation qui rend le système lisible.

```text
  ┌───────────────────────────────────────────────────────────────────┐
  │                    LES TROIS PIÈCES DE LA CAMÉRA                  │
  └───────────────────────────────────────────────────────────────────┘

   World            « CE QUI EXISTE »
                    Le donjon, le héros, les gobelins, les potions.
                    Un conteneur. Il ne se dessine PAS tout seul.
                    Coordonnées : monde.

   Viewfinder       « OÙ JE REGARDE »
                    Un point du monde, un zoom, un angle, une ancre.
                    Aucune taille, aucune forme.
                    Coordonnées : monde.

   Viewport         « PAR QUEL TROU JE REGARDE »
                    Un rectangle (ou un disque) découpé dans le canvas.
                    Il porte le HUD. Il découpe (clip) le rendu.
                    Coordonnées : écran.

   CameraComponent  « L'APPAREIL PHOTO »
                    Assemble un Viewfinder et un Viewport,
                    pointe vers un World, et dessine.
```

Et voici l'arbre de composants réel d'un `FlameGame` tout neuf, celui que Flame construit pour vous sans que vous écriviez une ligne.

```text
  FlameGame
   │
   ├── World                       priority = -0x7fffffff  (toujours en premier)
   │    ├── Donjon
   │    ├── Heros
   │    ├── Gobelin
   │    ├── Gobelin
   │    └── Potion
   │
   └── CameraComponent             priority = 0
        ├── backdrop  (Component)  dessiné DERRIÈRE le monde
        ├── Viewfinder             position / zoom / angle / anchor
        │    └── (enfants : dessinés devant le monde, avec sa transformation)
        └── Viewport               MaxViewport par défaut
             └── (enfants : le HUD, fixe à l'écran)
```

Trois remarques capitales sur ce schéma.

**Le `World` a une priorité extrêmement négative.** Sa valeur par défaut est `-0x7fffffff`, c'est-à-dire le plus petit entier signé sur 32 bits. Ce n'est pas un caprice : il faut que le monde soit **avant** la caméra dans l'arbre, pour que sa mise à jour ait lieu avant le rendu de la caméra. Vous n'avez jamais à toucher à cette valeur.

**Le `World` ne se dessine pas.** Son code source est explicite : la méthode `renderTree` du `World` est vide. Le monde n'existe visuellement que parce qu'une caméra le regarde. C'est une conséquence directe du fait que plusieurs caméras peuvent observer le même monde (section 31.24).

**L'ordre de rendu de la caméra est fixe.** D'abord le `backdrop`, puis le monde vu par le `viewfinder`, puis les enfants du `viewfinder`, puis les enfants du `viewport`. Le HUD est donc toujours devant. Vous n'avez pas à jouer avec `priority` pour ça.

```text
   ORDRE DE DESSIN D'UNE CAMÉRA, DU FOND VERS L'AVANT

   1. backdrop           ── fond fixe (ciel, dégradé, parallaxe lointaine)
   2. world              ── le donjon, transformé par le viewfinder
   3. viewfinder.children ── suivent le monde, mais dessinés devant lui
   4. viewport.children  ── le HUD, jamais transformé
```

Vérifions immédiatement que ces objets existent, avec un programme complet.

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: InspectionJeu()));
}

class InspectionJeu extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    debugPrint('world      : ${world.runtimeType}');
    debugPrint('camera     : ${camera.runtimeType}');
    debugPrint('viewport   : ${camera.viewport.runtimeType}');
    debugPrint('viewfinder : ${camera.viewfinder.runtimeType}');
    debugPrint('backdrop   : ${camera.backdrop.runtimeType}');
    debugPrint('la caméra regarde le monde : ${identical(camera.world, world)}');
    debugPrint('priorité du monde  : ${world.priority}');
    debugPrint('priorité caméra    : ${camera.priority}');
  }
}
```

**Résultat :**

```text
world      : World
camera     : CameraComponent
viewport   : MaxViewport
viewfinder : Viewfinder
backdrop   : Component
la caméra regarde le monde : true
priorité du monde  : -2147483647
priorité caméra    : 0
```

L'écran est vide et noir : c'est normal, le monde ne contient rien. Mais le câblage est déjà là.

| Objet | Type par défaut | Réglable par | Coordonnées |
| --- | --- | --- | --- |
| `game.world` | `World` | `FlameGame(world: ...)` | monde |
| `game.camera` | `CameraComponent` | `FlameGame(camera: ...)` | — |
| `camera.viewport` | `MaxViewport` | `CameraComponent(viewport: ...)` ou affectation | écran |
| `camera.viewfinder` | `Viewfinder` | `CameraComponent(viewfinder: ...)` ou affectation | monde |
| `camera.backdrop` | `Component` | `camera.backdrop = ...` ou `camera.backdrop.add(...)` | écran |

---

## 31.3 — Pourquoi Flame a changé de système de caméra, et comment reconnaître un tutoriel périmé

Cette section n'est pas de l'histoire pour le plaisir. C'est un outil de survie. La quasi-totalité des tutoriels Flame que vous trouverez en cherchant « flame camera follow player » sont écrits pour une version disparue, et leur code **ne compile pas** en 1.38.0.

### L'ancien modèle

Avant Flame 1.0, la classe de base d'un jeu s'appelait `BaseGame`. Elle possédait un champ `camera` de type `Camera` — une classe simple qui stockait une translation et l'appliquait au `Canvas` avant de dessiner les composants.

```dart
// CODE HISTORIQUE — NE COMPILE PAS EN FLAME 1.38.0
class MonJeu extends BaseGame {
  @override
  Future<void> onLoad() async {
    final joueur = Joueur();
    add(joueur);                          // au jeu, pas au monde
    camera.followComponent(joueur);       // méthode disparue
    camera.zoom = 2.0;                    // champ disparu
    camera.worldBounds = const Rect.fromLTWH(0, 0, 2400, 1400); // disparu
  }
}
```

Ce modèle marchait tant qu'on restait dans un cas simple : **un jeu, une vue, un monde**. Il s'écroulait dès qu'on sortait de ce cas.

### Les quatre limites qui ont provoqué la refonte

| Limite de l'ancien modèle | Pourquoi c'était bloquant |
| --- | --- |
| Une seule caméra par jeu | impossible de faire une minimap, un écran partagé à deux joueurs, ou une vue de recul pendant un boss |
| Le HUD n'avait pas de place définie | il fallait ruser avec `priority` et un mixin `HasWidgetsOverlay`, ou dessiner à la main après `canvas.restore()` |
| Pas de séparation « où je regarde » / « par quel trou » | on ne pouvait pas avoir un zoom fixe **et** une résolution fixe avec bandes noires |
| Pas de notion de monde | un composant ajouté au jeu était soit dans le monde, soit dans le HUD, sans que rien ne le dise ; l'ambiguïté générait des bugs muets |

Le nouveau modèle règle les quatre d'un coup, parce qu'il fait de la caméra **un composant comme un autre**. Une caméra est un composant, un monde est un composant, un viewport est un composant. On peut donc en ajouter plusieurs, les retirer, les animer avec des effets, les ranger dans l'arbre.

### Le tableau de conversion

Gardez ce tableau sous la main. Il traduit n'importe quel bout de code trouvé en ligne.

| Ce que vous lisez dans un vieux tutoriel | Statut en Flame 1.38.0 | Ce qu'il faut écrire |
| --- | --- | --- |
| `class MonJeu extends BaseGame` | classe **supprimée** depuis la 1.0 | `class MonJeu extends FlameGame` |
| `camera.followComponent(joueur)` | méthode **inexistante** | `camera.follow(joueur)` |
| `camera.followVector2(v)` | méthode **inexistante** | `camera.moveTo(v)` ou `follow` avec un `ReadOnlyPositionProvider` |
| `camera.zoom = 2` | champ **inexistant** sur `CameraComponent` | `camera.viewfinder.zoom = 2` |
| `camera.snapTo(v)` | méthode **inexistante** | `camera.moveTo(v)` (instantané par défaut) ou `follow(c, snap: true)` |
| `camera.worldBounds = Rect...` | champ **inexistant** | `camera.setBounds(Rectangle.fromLTRB(...))` |
| `camera.cameraSpeed = 1` | champ **inexistant** | `camera.follow(cible, maxSpeed: 300)` |
| `add(monEnnemi)` pour un élément du décor | compile, mais court-circuite la caméra | `world.add(monEnnemi)` |
| `Camera` importée de `package:flame/game.dart` | classe **supprimée** | `CameraComponent` de `package:flame/camera.dart` |
| `with HasGameRef<MonJeu>` + `gameRef` | **déprécié**, suppression annoncée | `with HasGameReference<MonJeu>` + `game` |
| `game.camera.viewport = ...` de l'ancien système | remplacé | `camera.viewport = FixedResolutionViewport(resolution: ...)` |

### Les six marqueurs d'un tutoriel périmé

Avant de recopier quoi que ce soit, cherchez ces six chaînes dans la page. Une seule suffit à disqualifier le texte entier.

```text
  1. BaseGame                 → antérieur à Flame 1.0 (2021)
  2. followComponent          → antérieur à la refonte de la caméra
  3. camera.zoom              → idem
  4. worldBounds              → idem
  5. HasTappables / Tappable  → antérieur au système d'événements moderne
  6. RawKeyEvent              → antérieur à l'API KeyEvent de Flutter
```

Ajoutez-y un réflexe : regardez la **date** de l'article et la version dans son `pubspec.yaml`. Si le tutoriel écrit `flame: ^1.0.0`, il a cinq ans. Si vous ne trouvez pas de version, fermez l'onglet.

### La source qui fait autorité

Une seule référence est fiable : la documentation officielle et le dépôt du projet.

```text
  https://docs.flame-engine.org/latest/flame/camera.html
  https://pub.dev/documentation/flame/latest/camera/CameraComponent-class.html
  https://github.com/flame-engine/flame/blob/main/doc/flame/camera.md
```

Notez au passage un piège : l'adresse `camera_component.html` que citent beaucoup de blogs renvoie une **erreur 404**. La page actuelle s'appelle `camera.html`. Ce détail est lui-même un bon indicateur de l'âge du texte que vous lisez.

> **À retenir.** Le nouveau système tient en une phrase : **le monde est un composant, la caméra est un composant, et la caméra regarde le monde**. Tout ce qui contredit cette phrase est périmé.

---

## 31.4 — Créer un `World`

Vous n'avez pas besoin de créer un `World` : `FlameGame` en fabrique un et vous le donne sous le nom `world`. Mais il y a trois bonnes raisons d'en créer un vous-même.

1. Y mettre du code : le chargement du niveau, la liste des ennemis, la logique commune.
2. En avoir plusieurs : un monde par niveau, ou un monde « jeu » et un monde « menu ».
3. Le typer, pour accéder à ses membres sans transtypage.

### Le constructeur réel

```dart
World({
  Iterable<Component>? children,
  int? priority = -0x7fffffff,
  ComponentKey? key,
});
```

### Un monde qui contient le donjon

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

/// Le monde du « Donjon de Dart ».
/// Il connaît sa taille, ses murs, et sait où le héros apparaît.
class MondeDonjon extends World {
  static final Vector2 taille = Vector2(2400, 1400);
  static final Vector2 apparition = Vector2(200, 200);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le sol : un grand rectangle sombre.
    await add(
      RectangleComponent(
        position: Vector2.zero(),
        size: taille,
        paint: Paint()..color = const Color(0xFF1B1A26),
      ),
    );

    // Une grille de repères, tous les 200 pixels, pour VOIR le déplacement.
    for (var x = 0.0; x <= taille.x; x += 200) {
      for (var y = 0.0; y <= taille.y; y += 200) {
        await add(
          RectangleComponent(
            position: Vector2(x, y),
            size: Vector2.all(6),
            anchor: Anchor.center,
            paint: Paint()..color = const Color(0xFF3A3850),
          ),
        );
      }
    }
  }
}

class DonjonDeDart extends FlameGame {
  DonjonDeDart() : super(world: MondeDonjon());

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    camera.viewfinder.position = Vector2.zero();
  }
}
```

**Résultat :** un fond gris très sombre avec des points plus clairs tous les 200 pixels. Seul le coin haut-gauche du donjon est visible, puisque l'écran fait moins de 2400 × 1400.

### Typer le monde

`FlameGame` est générique : sa déclaration réelle est `class FlameGame<W extends World>`. En précisant le paramètre, la propriété `world` prend directement le bon type.

```dart
class DonjonDeDart extends FlameGame<MondeDonjon> {
  DonjonDeDart() : super(world: MondeDonjon());

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Pas de transtypage : world est déjà un MondeDonjon.
    camera.viewfinder.position = world.apparition;
    debugPrint('taille du donjon : ${MondeDonjon.taille}');
  }
}
```

**Résultat :**

```text
taille du donjon : [2400.0,1400.0]
```

> **Remarque.** `MondeDonjon.taille` et `MondeDonjon.apparition` sont ici des membres **statiques**, comme au chapitre 08. C'est pratique pour les constantes du niveau, parce qu'on peut y accéder sans instance, y compris depuis le constructeur du jeu.

### Un monde par niveau

Rien n'interdit d'avoir plusieurs mondes et de changer celui que la caméra regarde. C'est même la façon la plus propre de changer de niveau.

```dart
class JeuMultiNiveaux extends FlameGame {
  final MondeDonjon salleA = MondeDonjon();
  final MondeDonjon salleB = MondeDonjon();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await add(salleA);
    await add(salleB);
    camera.world = salleA; // on regarde la salle A
  }

  void allerEnSalleB() {
    camera.world = salleB; // changement instantané, rien n'est démonté
  }
}
```

**Résultat :** l'appel à `allerEnSalleB()` bascule la vue sans recharger quoi que ce soit. Les deux mondes continuent d'être mis à jour ; si vous voulez geler la salle A, retirez-la de l'arbre ou coupez sa logique.

---

## 31.5 — Ajouter les entités au monde plutôt qu'au jeu

C'est l'erreur numéro un des débutants en Flame, et elle est sournoise parce que le code **compile** et que quelque chose **s'affiche**.

### Les deux gestes, côte à côte

```dart
add(heros);         // ajoute au JEU     → repère écran, ignore la caméra
world.add(heros);   // ajoute au MONDE   → repère monde, suit la caméra
```

### Démonstration : deux carrés, deux destinations

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuComparaison()));
}

class JeuComparaison extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Carré VERT : dans le monde. Il bougera avec la caméra.
    await world.add(
      RectangleComponent(
        position: Vector2(120, 120),
        size: Vector2.all(60),
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );

    // Carré ROUGE : dans le jeu. Il ne bougera jamais.
    await add(
      RectangleComponent(
        position: Vector2(120, 220),
        size: Vector2.all(60),
        paint: Paint()..color = const Color(0xFFE53935),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    // La caméra glisse vers la droite.
    camera.viewfinder.position = Vector2(_t * 40, 0);
  }
}
```

**Résultat :**

```text
  t = 0 s                        t = 3 s
  ┌───────────────────┐          ┌───────────────────┐
  │  ■ vert           │          │ ■ vert (parti)    │   ← le vert a glissé
  │                   │          │                   │      vers la gauche
  │  ■ rouge          │          │  ■ rouge          │   ← le rouge n'a pas
  │                   │          │                   │      bougé d'un pixel
  └───────────────────┘          └───────────────────┘
```

Le carré vert défile parce qu'il appartient au monde et que la caméra se déplace dans ce monde. Le carré rouge est peint directement sur le canevas du jeu, **après** que la caméra a fini son travail : aucune transformation ne s'y applique.

### Pourquoi c'est un piège

Le débutant écrit `add(heros)`. Le héros apparaît. Il se déplace au clavier. Tout semble parfait. Puis vient le moment d'ajouter `camera.follow(heros)` — et rien ne se passe. La caméra suit bien un point, mais ce point est dans un monde où le héros n'est pas.

| Symptôme observé | Cause réelle |
| --- | --- |
| `camera.follow()` ne fait rien | le héros est dans le jeu, pas dans le monde |
| Le décor défile mais le héros reste collé | le décor est dans le monde, le héros dans le jeu |
| Le zoom n'affecte qu'une partie des éléments | mélange `add()` / `world.add()` |
| Un composant reste visible hors des bornes de la caméra | il n'est pas soumis au viewport |
| La minimap n'affiche pas les ennemis | les ennemis ne sont pas dans le monde observé |

### La règle de décision

```text
  ┌─────────────────────────────────────────────────────────────┐
  │  OÙ AJOUTER MON COMPOSANT ?                                 │
  └─────────────────────────────────────────────────────────────┘

   Est-ce un objet qui a une position DANS LE DONJON ?
   (héros, gobelin, potion, mur, coffre, porte, projectile)
        │  oui
        └──────────────────► world.add(...)

   Est-ce un élément d'interface fixé à l'écran ?
   (barre de vie, score, joystick, bouton, minimap)
        │  oui
        └──────────────────► camera.viewport.add(...)

   Est-ce un décor fixe DERRIÈRE le monde ?
   (ciel, dégradé, brume)
        │  oui
        └──────────────────► camera.backdrop.add(...)

   Est-ce un composant sans rendu : contrôleur, minuteur, gestionnaire ?
        │  oui
        └──────────────────► add(...)  au jeu, c'est correct
```

> **À retenir.** `world.add()` pour tout ce qui a une position dans le donjon. `camera.viewport.add()` pour tout ce qui est collé à l'écran. `add()` au jeu est réservé aux composants **invisibles** : contrôleurs, minuteurs, gestionnaires d'entrées, et aux caméras supplémentaires.

---

## 31.6 — `CameraComponent(world: ...)`

`FlameGame` vous donne une caméra. Vous n'avez donc généralement pas à en construire une. Mais dès que vous voulez une minimap, un écran partagé, ou une caméra avec un viewport particulier, il faut savoir écrire le constructeur.

### Le constructeur réel

```dart
CameraComponent({
  World? world,
  Viewport? viewport,
  Viewfinder? viewfinder,
  Component? backdrop,
  List<Component>? hudComponents,
  Iterable<Component>? children,
  ComponentKey? key,
});
```

| Paramètre | Rôle | Valeur par défaut |
| --- | --- | --- |
| `world` | le monde que cette caméra observe | `null` : la caméra ne dessine rien |
| `viewport` | la fenêtre à l'écran | `MaxViewport()` |
| `viewfinder` | le point de vue | un `Viewfinder` neuf, zoom 1, ancre centre |
| `backdrop` | composant dessiné derrière le monde | un `Component` vide |
| `hudComponents` | raccourci : composants ajoutés au viewport | `null` |
| `children` | enfants directs de la caméra | `null` |

Le paramètre `hudComponents` est un simple confort : il ajoute la liste au `viewport`. Les deux écritures suivantes sont équivalentes.

```dart
// Écriture 1
final c1 = CameraComponent(
  world: monMonde,
  hudComponents: [TextComponent(text: 'Vies : 3')],
);

// Écriture 2
final c2 = CameraComponent(world: monMonde);
c2.viewport.add(TextComponent(text: 'Vies : 3'));
```

### Fournir sa propre caméra au jeu

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

class MondeDonjon extends World {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await add(
      RectangleComponent(
        position: Vector2.zero(),
        size: Vector2(1200, 800),
        paint: Paint()..color = const Color(0xFF23223A),
      ),
    );
    await add(
      CircleComponent(
        radius: 24,
        position: Vector2(300, 200),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }
}

class DonjonDeDart extends FlameGame<MondeDonjon> {
  // On construit le monde d'abord, pour pouvoir le passer à la caméra.
  DonjonDeDart._(MondeDonjon monde)
      : super(
          world: monde,
          camera: CameraComponent(world: monde),
        );

  factory DonjonDeDart() => DonjonDeDart._(MondeDonjon());

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.position = Vector2(300, 200);
  }
}
```

**Résultat :** un disque doré parfaitement centré à l'écran, sur fond violet sombre.

Le petit détour par un constructeur nommé privé et une fabrique (chapitre 09) sert à une seule chose : disposer de **la même instance** de monde des deux côtés. Écrire `super(world: MondeDonjon(), camera: CameraComponent(world: MondeDonjon()))` créerait deux mondes différents, et la caméra regarderait le mauvais.

> **Erreur classique.** Deux instances de monde. Le jeu met à jour le monde A, la caméra dessine le monde B. À l'écran : un décor figé, un héros immobile, et aucun message d'erreur.

### Remplacer la caméra après coup

C'est possible et parfois plus lisible.

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // On garde le monde fourni par FlameGame, on remplace la caméra.
    camera.removeFromParent();
    camera = CameraComponent.withFixedResolution(
      width: 640,
      height: 360,
      world: world,
    );
    await add(camera);
  }
}
```

**Résultat :** la vue passe en résolution fixe 640 × 360, avec des bandes noires si la fenêtre n'a pas exactement le rapport 16/9.

---

## 31.7 — `CameraComponent.withFixedResolution()`

C'est la fabrique la plus utile de tout le chapitre, et pourtant la moins connue. Elle répond à une question que tout jeu 2D finit par se poser : **« mon jeu doit-il montrer plus de décor sur un grand écran ? »**

### Le problème

Sans résolution fixe, un joueur sur écran 4K voit quatre fois plus de donjon qu'un joueur sur téléphone. Pour un jeu de plateforme ou un jeu d'arcade, c'est un déséquilibre : celui qui a le grand écran voit les ennemis arriver bien avant les autres.

```text
   SANS RÉSOLUTION FIXE

   Téléphone 640×360         Écran 1920×1080
   ┌──────────────┐          ┌──────────────────────────────────────┐
   │ ♦  ○         │          │ ♦  ○        ○     ▲       ▲          │
   │              │          │                                      │
   └──────────────┘          │        ▓▓▓▓       ○      ▲           │
                             └──────────────────────────────────────┘
   3 tuiles visibles          12 tuiles visibles : trop d'avantage
```

### La solution

```dart
CameraComponent.withFixedResolution({
  required double width,
  required double height,
  World? world,
  Viewfinder? viewfinder,
  Component? backdrop,
  List<Component>? hudComponents,
  Iterable<Component>? children,
  ComponentKey? key,
});
```

La caméra garantit alors que **exactement** `width × height` unités de monde sont visibles, quelle que soit la taille réelle de la fenêtre. Le rendu est mis à l'échelle, et si le rapport de forme de la fenêtre ne correspond pas, Flame ajoute des bandes noires.

```text
   AVEC withFixedResolution(width: 640, height: 360)

   Fenêtre 1920×1080 (16/9)        Fenêtre 1000×800 (5/4)
   ┌──────────────────────┐        ┌──────────────────────┐
   │                      │        │██████████████████████│ ← bande
   │  ♦  ○                │        │                      │
   │                      │        │  ♦  ○                │
   │                      │        │                      │
   └──────────────────────┘        │██████████████████████│ ← bande
   image ×3, aucune bande          └──────────────────────┘
                                    image ×1.56, bandes haut/bas
```

### Exemple complet

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuResolutionFixe()));
}

class JeuResolutionFixe extends FlameGame {
  JeuResolutionFixe()
      : super(
          camera: CameraComponent.withFixedResolution(
            width: 320,
            height: 180,
          ),
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    camera.viewfinder.anchor = Anchor.topLeft;
    camera.viewfinder.position = Vector2.zero();

    // Un cadre de 320 × 180 : il doit remplir exactement la zone visible.
    await world.add(
      RectangleComponent(
        position: Vector2.zero(),
        size: Vector2(320, 180),
        paint: Paint()..color = const Color(0xFF2E2B4A),
      ),
    );

    // Quatre coins repères, de 8 × 8 unités de monde.
    for (final coin in [
      Vector2(0, 0),
      Vector2(312, 0),
      Vector2(0, 172),
      Vector2(312, 172),
    ]) {
      await world.add(
        RectangleComponent(
          position: coin,
          size: Vector2.all(8),
          paint: Paint()..color = const Color(0xFFE8B04B),
        ),
      );
    }

    debugPrint('taille du canvas : $size');
    debugPrint('zone de monde visible : ${camera.visibleWorldRect}');
  }
}
```

**Résultat**, dans une fenêtre de 1280 × 720 :

```text
taille du canvas : [1280.0,720.0]
zone de monde visible : Rect.fromLTRB(0.0, 0.0, 320.0, 180.0)
```

Les quatre carrés dorés sont exactement aux quatre coins de l'écran, et ils sont **gros** : chaque unité de monde occupe 4 pixels réels.

Redimensionnez la fenêtre : les carrés restent aux coins. Le nombre d'unités visibles ne change jamais.

### Ce que la fabrique fait vraiment

Elle installe un `FixedResolutionViewport` et règle le `viewfinder` pour que la zone demandée remplisse ce viewport. Vous pouvez donc encore modifier le zoom **par-dessus**, mais c'est rarement une bonne idée : vous perdriez la garantie qui justifiait le choix.

| Quand l'utiliser | Quand l'éviter |
| --- | --- |
| Jeu d'arcade, plateforme, puzzle : l'équité entre joueurs compte | Jeu de stratégie où voir plus loin est un confort, pas un avantage |
| Pixel art : la résolution logique est le cœur du style | Jeu où l'on veut exploiter les grands écrans |
| Vous voulez un placement de HUD prévisible | Interface qui doit s'adapter finement à chaque taille |

---

## 31.8 — Le `viewfinder` : position, zoom, angle, ancre

Le `viewfinder` — le « viseur » — est l'objet qui répond à la question **« quelle partie du monde est à l'écran ? »**. Il n'a ni taille ni forme : c'est un point de vue.

### Les quatre propriétés

| Propriété | Type | Signification | Défaut |
| --- | --- | --- | --- |
| `position` | `Vector2` | le point du monde placé sur l'ancre du viewport | `Vector2(0, 0)` |
| `zoom` | `double` | facteur d'agrandissement ; 2 = deux fois plus gros | `1.0` |
| `angle` | `double` | rotation de la vue, **en radians** | `0.0` |
| `anchor` | `Anchor` | l'endroit du viewport considéré comme « centre logique » | `Anchor.center` |
| `visibleGameSize` | `Vector2?` | taille de monde à rendre visible ; règle le zoom pour vous | `null` |

### Le schéma qui explique tout

```text
   MONDE (2400 × 1400)                          ÉCRAN (800 × 450)
   ┌────────────────────────────────────┐       ┌──────────────────┐
   │                                    │       │                  │
   │        ┌═══════════════┐           │       │                  │
   │        ║               ║           │       │        ✚         │ ← anchor
   │        ║       ✚       ║           │  ───► │                  │   (center)
   │        ║  viewfinder   ║           │       │                  │
   │        ║  .position    ║           │       └──────────────────┘
   │        ║  = (1000,600) ║           │
   │        ╚═══════════════╝           │       Le point (1000,600)
   │         zone visible               │       du monde apparaît
   │         800/zoom × 450/zoom        │       au centre de l'écran.
   └────────────────────────────────────┘
```

Lisez la définition officielle de `position` mot à mot : « les coordonnées de jeu d'un point qui doit être positionné au centre du viewport ». Et celle de `anchor` : « le point à l'intérieur du viewport considéré comme le centre logique de la caméra ». Les deux ensemble donnent la règle :

```text
   Le point du MONDE  viewfinder.position
   est dessiné sur    l'ancre viewfinder.anchor du VIEWPORT.
```

### Les quatre réglages en un seul programme

```dart
import 'dart:math';

import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuViewfinder()));
}

class JeuViewfinder extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Une grille de 12 × 8 cases de 100 unités.
    for (var i = 0; i < 12; i++) {
      for (var j = 0; j < 8; j++) {
        await world.add(
          RectangleComponent(
            position: Vector2(i * 100.0, j * 100.0),
            size: Vector2.all(96),
            paint: Paint()
              ..color = (i + j).isEven
                  ? const Color(0xFF2E2B4A)
                  : const Color(0xFF3B3760),
          ),
        );
      }
    }

    // Un repère doré au centre du monde.
    await world.add(
      CircleComponent(
        radius: 20,
        position: Vector2(600, 400),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );

    // Les quatre réglages du viseur.
    camera.viewfinder
      ..position = Vector2(600, 400) // je regarde le centre du monde
      ..zoom = 1.5 // tout est 1,5 fois plus gros
      ..angle = pi / 24 // légère inclinaison, 7,5 degrés
      ..anchor = Anchor.center; // ce point est au centre de l'écran

    debugPrint('zone visible : ${camera.visibleWorldRect}');
  }
}
```

**Résultat**, dans une fenêtre de 800 × 450 :

```text
zone visible : Rect.fromLTRB(331.6, 226.7, 868.4, 573.3)
```

Le damier est incliné, le disque doré est au centre exact de l'écran, et la zone visible mesure environ 537 × 347 unités de monde au lieu de 800 × 450 : c'est l'effet du zoom 1,5 combiné à l'inclinaison.

> **Remarque.** `visibleWorldRect` renvoie le rectangle **englobant** de la zone visible. Quand la caméra est inclinée, ce rectangle est plus grand que la zone réellement dessinée : c'est voulu, il sert au culling et doit être conservateur.

### `visibleGameSize` : régler le zoom autrement

Plutôt que de calculer un zoom, on peut dire à Flame « je veux voir au moins 400 unités de large ». Il en déduit le zoom.

```dart
camera.viewfinder.visibleGameSize = Vector2(400, 225);
```

**Résultat :** sur une fenêtre 800 × 450, le zoom devient automatiquement 2. Sur une fenêtre 1600 × 900, il devient 4. Le nombre d'unités visibles ne change pas.

C'est une alternative à `withFixedResolution` quand on ne veut pas de bandes noires : ici, sur un écran de rapport différent, on verra **plus** dans une direction, jamais moins.

---

## 31.9 — `camera.viewfinder.position`

C'est le déplacement le plus direct : vous écrivez une position de monde, la caméra y saute immédiatement.

```dart
camera.viewfinder.position = Vector2(1200, 700);
```

### Un déplacement libre au clavier

Voici un programme complet où les flèches déplacent la caméra dans le donjon, sans aucun personnage.

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(GameWidget(game: JeuCameraLibre()));
}

class JeuCameraLibre extends FlameGame with KeyboardEvents {
  static final Vector2 tailleMonde = Vector2(2400, 1400);

  final Vector2 _direction = Vector2.zero();
  static const double vitesse = 400; // unités de monde par seconde

  final TextComponent _info = TextComponent(
    text: '',
    position: Vector2.all(12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
    ),
  );

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await world.add(
      RectangleComponent(
        position: Vector2.zero(),
        size: tailleMonde,
        paint: Paint()..color = const Color(0xFF1B1A26),
      ),
    );

    // Des bornes tous les 300 pixels pour se repérer.
    for (var x = 0.0; x < tailleMonde.x; x += 300) {
      for (var y = 0.0; y < tailleMonde.y; y += 300) {
        await world.add(
          RectangleComponent(
            position: Vector2(x, y),
            size: Vector2.all(40),
            anchor: Anchor.center,
            paint: Paint()..color = const Color(0xFF4A4770),
          ),
        );
      }
    }

    camera.viewfinder.anchor = Anchor.center;
    camera.viewfinder.position = tailleMonde / 2;

    await camera.viewport.add(_info);
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    _direction
      ..x = 0
      ..y = 0;
    if (keysPressed.contains(LogicalKeyboardKey.arrowLeft)) _direction.x -= 1;
    if (keysPressed.contains(LogicalKeyboardKey.arrowRight)) _direction.x += 1;
    if (keysPressed.contains(LogicalKeyboardKey.arrowUp)) _direction.y -= 1;
    if (keysPressed.contains(LogicalKeyboardKey.arrowDown)) _direction.y += 1;
    return KeyEventResult.handled;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (!_direction.isZero()) {
      // On écrit directement la position du viseur.
      camera.viewfinder.position += _direction.normalized() * vitesse * dt;
    }
    final p = camera.viewfinder.position;
    _info.text = 'caméra : (${p.x.toStringAsFixed(0)}, '
        '${p.y.toStringAsFixed(0)})';
  }
}
```

**Résultat :**

```text
caméra : (1200, 700)
caméra : (1354, 700)
caméra : (1354, 592)
```

Les flèches font défiler le donjon. Le texte, lui, ne bouge pas : il est dans le viewport.

### Le piège de la modification en place

`viewfinder.position` renvoie un `Vector2`, qui est **mutable** (chapitre 27, section 27.19). Deux écritures existent, et elles ne se valent pas toujours.

```dart
// A. Réaffectation : toujours correcte.
camera.viewfinder.position = camera.viewfinder.position + deplacement;

// B. Opérateur composé : correct aussi, Flame gère la notification.
camera.viewfinder.position += deplacement;

// C. DANGEREUX : on garde une référence sur le vecteur interne.
final p = camera.viewfinder.position;
p.x += 10;         // modifie l'objet interne sans passer par le setter
```

L'écriture C fonctionne parfois et échoue silencieusement dans d'autres cas — par exemple si un `FollowBehavior` réécrit la position juste après. Prenez l'habitude des écritures A ou B, et n'utilisez jamais une référence stockée sur `position` comme variable de travail. Si vous avez besoin d'une copie, `clone()` :

```dart
final depart = camera.viewfinder.position.clone();
```

---

## 31.10 — `camera.viewfinder.zoom`

Le zoom est un simple facteur multiplicatif appliqué entre le monde et l'écran.

```text
   pixels_écran = unités_monde × zoom
```

| `zoom` | Effet | Unités visibles sur 800 px de large |
| --- | --- | --- |
| `0.5` | dézoom : on voit deux fois plus loin | 1600 |
| `1.0` | 1 unité de monde = 1 pixel | 800 |
| `2.0` | tout est deux fois plus gros | 400 |
| `4.0` | pixel art typique | 200 |

### Un zoom animé au clavier

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(GameWidget(game: JeuZoom()));
}

class JeuZoom extends FlameGame with KeyboardEvents {
  final TextComponent _info = TextComponent(
    text: '',
    position: Vector2.all(12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
    ),
  );

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    for (var i = 0; i < 10; i++) {
      for (var j = 0; j < 10; j++) {
        await world.add(
          RectangleComponent(
            position: Vector2(i * 64.0, j * 64.0),
            size: Vector2.all(60),
            paint: Paint()
              ..color = (i + j).isEven
                  ? const Color(0xFF2E2B4A)
                  : const Color(0xFF474073),
          ),
        );
      }
    }

    camera.viewfinder
      ..anchor = Anchor.center
      ..position = Vector2(320, 320)
      ..zoom = 1.0;

    await camera.viewport.add(_info);
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is! KeyDownEvent) return KeyEventResult.ignored;

    final vf = camera.viewfinder;
    if (event.logicalKey == LogicalKeyboardKey.keyA) {
      vf.zoom = (vf.zoom * 1.25).clamp(0.25, 8.0);
    } else if (event.logicalKey == LogicalKeyboardKey.keyE) {
      vf.zoom = (vf.zoom / 1.25).clamp(0.25, 8.0);
    } else if (event.logicalKey == LogicalKeyboardKey.digit1) {
      vf.zoom = 1.0;
    }
    return KeyEventResult.handled;
  }

  @override
  void update(double dt) {
    super.update(dt);
    final r = camera.visibleWorldRect;
    _info.text = 'zoom ${camera.viewfinder.zoom.toStringAsFixed(2)}   '
        'visible ${r.width.toStringAsFixed(0)} × '
        '${r.height.toStringAsFixed(0)}';
  }
}
```

**Résultat**, touches `A` puis `E` puis `1`, dans une fenêtre 800 × 450 :

```text
zoom 1.00   visible 800 × 450
zoom 1.25   visible 640 × 360
zoom 1.00   visible 800 × 450
```

### Trois pièges du zoom

**Le zoom ne déplace pas la caméra.** Il agrandit autour du point d'ancrage. Si `anchor` vaut `Anchor.center`, on zoome sur le centre de l'écran ; si elle vaut `Anchor.topLeft`, on zoome sur le coin.

**Un zoom nul ou négatif casse tout.** `zoom = 0` provoque une division par zéro dans la matrice de transformation. Bornez toujours vos réglages, comme dans l'exemple ci-dessus avec `.clamp(0.25, 8.0)`.

**Un zoom animé image par image doit être lissé.** Multiplier par 1,25 à chaque frame donne une accélération exponentielle. Réservez cette écriture aux appuis de touche (`KeyDownEvent`), pas à `update`. Pour un zoom continu, interpolez :

```dart
@override
void update(double dt) {
  super.update(dt);
  final actuel = camera.viewfinder.zoom;
  // On rejoint _zoomCible en douceur, 6 = rapidité de rattrapage.
  camera.viewfinder.zoom = actuel + (_zoomCible - actuel) * (1 - exp(-6 * dt));
}
```

> **Remarque.** La formule `1 - exp(-k * dt)` est le lissage indépendant du framerate vu au chapitre 25, section 25.9. Un simple `actuel + (cible - actuel) * 0.1` dépendrait du nombre d'images par seconde.

---

## 31.11 — `camera.viewfinder.anchor`

L'ancre du viseur répond à la question : **« à quel endroit de l'écran le point regardé doit-il apparaître ? »**

```text
   anchor = Anchor.center           anchor = Anchor.topLeft
   ┌────────────────────┐           ┌────────────────────┐
   │                    │           │ ✚ position         │
   │                    │           │                    │
   │         ✚ position │           │                    │
   │                    │           │                    │
   │                    │           │                    │
   └────────────────────┘           └────────────────────┘
   Le monde s'étend                 Le monde s'étend
   dans les 4 directions            vers la droite et le bas


   anchor = Anchor(0.3, 0.5)        anchor = Anchor.bottomCenter
   ┌────────────────────┐           ┌────────────────────┐
   │                    │           │                    │
   │                    │           │                    │
   │   ✚ position       │           │                    │
   │                    │           │                    │
   │                    │           │         ✚ position │
   └────────────────────┘           └────────────────────┘
   Le héros est à 30 % du bord      Vue « du sol » : on voit
   gauche : on voit loin devant     surtout au-dessus du héros
```

### Les valeurs courantes

| Valeur | Usage typique |
| --- | --- |
| `Anchor.center` | valeur par défaut, jeu vu de dessus, exploration |
| `Anchor.topLeft` | quand vous voulez raisonner « comme au chapitre 21 » : le monde (0,0) au coin de l'écran |
| `Anchor(0.35, 0.5)` | jeu de plateforme orienté vers la droite : on voit loin devant |
| `Anchor.bottomCenter` | jeu de saut vertical : on voit ce qui arrive du haut |
| `Anchor(0.5, 0.65)` | jeu de tir vers le haut : le vaisseau est bas, l'action est haute |

### Démonstration : le regard qui devance le héros

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuAncre()));
}

class Heros extends RectangleComponent {
  Heros()
      : super(
          size: Vector2(40, 56),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  double vitesse = 160;

  @override
  void update(double dt) {
    super.update(dt);
    position.x += vitesse * dt;
    if (position.x > 2200) position.x = 100;
  }
}

class JeuAncre extends FlameGame {
  late final Heros heros;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await world.add(
      RectangleComponent(
        position: Vector2.zero(),
        size: Vector2(2400, 600),
        paint: Paint()..color = const Color(0xFF1B1A26),
      ),
    );

    for (var x = 0.0; x < 2400; x += 120) {
      await world.add(
        RectangleComponent(
          position: Vector2(x, 380),
          size: Vector2(100, 20),
          paint: Paint()..color = const Color(0xFF4A4770),
        ),
      );
    }

    heros = Heros()..position = Vector2(100, 340);
    await world.add(heros);

    // 35 % depuis la gauche : on voit largement devant le héros.
    camera.viewfinder.anchor = const Anchor(0.35, 0.5);
    camera.follow(heros);

    await camera.viewport.add(
      TextComponent(
        text: 'anchor = Anchor(0.35, 0.5)',
        position: Vector2.all(12),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
        ),
      ),
    );
  }
}
```

**Résultat :** le carré doré reste à environ un tiers de la largeur de l'écran, et le joueur voit deux fois plus de décor devant lui que derrière. Remplacez la ligne par `camera.viewfinder.anchor = Anchor.center` : le héros se recentre, et l'impression de vitesse diminue nettement.

> **À retenir.** Changer l'ancre ne coûte rien et améliore énormément le confort de jeu. C'est le premier réglage à tenter quand un joueur se plaint de « ne pas voir arriver les ennemis ».

---

## 31.12 — `camera.follow()`

Voici la méthode qui remplace les vingt lignes de suivi manuel du chapitre 25.

### La signature réelle

```dart
void follow(
  ReadOnlyPositionProvider target, {
  double maxSpeed = double.infinity,
  bool horizontalOnly = false,
  bool verticalOnly = false,
  bool snap = false,
});
```

| Paramètre | Effet | Défaut |
| --- | --- | --- |
| `target` | l'objet suivi ; tout `PositionComponent` convient | — |
| `maxSpeed` | vitesse maximale de la caméra, en unités de monde par seconde | `double.infinity` (suivi collé) |
| `horizontalOnly` | ne suit que l'axe X | `false` |
| `verticalOnly` | ne suit que l'axe Y | `false` |
| `snap` | saute immédiatement sur la cible avant de suivre | `false` |

### Le type `ReadOnlyPositionProvider`

Ce n'est pas une classe à connaître par cœur : c'est une interface que **tout `PositionComponent` implémente déjà**. On écrit donc directement `camera.follow(heros)`.

On peut aussi suivre un point calculé, par exemple le milieu entre deux joueurs.

```dart
/// Fournit une position calculée : le milieu entre deux composants.
class PointMilieu implements ReadOnlyPositionProvider {
  PointMilieu(this.a, this.b);

  final PositionComponent a;
  final PositionComponent b;

  @override
  Vector2 get position => (a.position + b.position) / 2;
}

// camera.follow(PointMilieu(joueur1, joueur2), maxSpeed: 200);
```

**Résultat :** la caméra reste entre les deux héros. C'est le comportement d'un jeu coopératif à écran unique.

### Exemple complet : le héros suivi dans le donjon

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuSuivi()));

class Heros extends RectangleComponent {
  Heros()
      : super(
          size: Vector2.all(40),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) position += direction.normalized() * 220 * dt;
    position.clamp(Vector2.all(20), Vector2(2380, 1380));
  }
}

class JeuSuivi extends FlameGame with KeyboardEvents {
  final Heros heros = Heros();

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await world.add(RectangleComponent(
      size: Vector2(2400, 1400),
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    final borne = Paint()..color = const Color(0xFF44406B);
    for (var i = 0; i < 84; i++) {
      await world.add(RectangleComponent(
        position: Vector2((i % 12) * 200.0, (i ~/ 12) * 200.0),
        size: Vector2.all(24),
        anchor: Anchor.center,
        paint: borne,
      ));
    }
    heros.position = Vector2(1200, 700);
    await world.add(heros);

    // Une seule ligne remplace tout le suivi manuel du chapitre 25.
    camera.follow(heros);
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    bool p(LogicalKeyboardKey a, LogicalKeyboardKey b) =>
        touches.contains(a) || touches.contains(b);
    heros.direction
      ..x = (p(LogicalKeyboardKey.keyD, LogicalKeyboardKey.arrowRight)
              ? 1.0
              : 0.0) -
          (p(LogicalKeyboardKey.keyQ, LogicalKeyboardKey.arrowLeft) ? 1.0 : 0.0)
      ..y = (p(LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown)
              ? 1.0
              : 0.0) -
          (p(LogicalKeyboardKey.keyZ, LogicalKeyboardKey.arrowUp) ? 1.0 : 0.0);
    return KeyEventResult.handled;
  }
}
```

**Résultat :** le carré doré reste au centre de l'écran, et c'est le donjon qui défile autour de lui. Le comportement est identique à celui obtenu au chapitre 25 avec quarante lignes de calcul.

### Comment ça marche à l'intérieur

`follow()` n'écrit pas la position depuis la caméra : il **ajoute un composant** au viseur, un `FollowBehavior`, dont l'`update(dt)` rapproche le viseur de la cible.

```text
   CameraComponent
    └── Viewfinder
         └── FollowBehavior      ← ajouté par follow()
              target   = heros
              maxSpeed = 250
```

Deux conséquences pratiques. **Appeler `follow()` deux fois remplace** le comportement précédent : on n'accumule pas les suiveurs. Et **un `MoveEffect` posé sur le viseur coexiste** avec le suivi, parce que les effets de Flame appliquent des modifications relatives : c'est ce qui rend possible le tremblement d'écran de la section 31.23.

### Les axes séparés

```dart
// Plateforme à défilement horizontal : hauteur de vue fixe.
camera.viewfinder.position = Vector2(0, 400);
camera.follow(heros, horizontalOnly: true);
```

**Résultat :** quand le héros saute, l'écran ne bouge pas verticalement. C'est le réglage classique des jeux de plateforme : suivre chaque petit saut donne la nausée.

## 31.13 — `maxSpeed` et le suivi lissé

Par défaut, `maxSpeed` vaut `double.infinity` : la caméra est **collée** à sa cible. C'est net, précis… et souvent désagréable.

### Pourquoi le suivi collé fatigue le joueur

Reprenez le chapitre 25, section 25.8. Quand la caméra est collée, chaque micro-mouvement du héros déplace l'écran entier. Un héros qui oscille de deux pixels fait vibrer tout le décor, alors que l'œil s'attend à un décor stable.

```text
   maxSpeed = infini (collé)        maxSpeed = 200 (lissé)

   héros  ▁▂▃▅▇▇▅▃▂▁               héros  ▁▂▃▅▇▇▅▃▂▁
   caméra ▁▂▃▅▇▇▅▃▂▁               caméra ▁▁▂▃▄▅▆▅▄▃
          identique, saccadé              en retard, doux
```

### Choisir la valeur

La règle simple : **`maxSpeed` légèrement supérieur à la vitesse maximale du héros**.

| Vitesse du héros | `maxSpeed` | Sensation |
| --- | --- | --- |
| 200 | 150 | la caméra décroche puis perd le héros : à proscrire |
| 200 | 240 | rattrapage doux, la caméra reste presque sur le héros |
| 200 | 600 | pratiquement collé, mais les micro-oscillations sont absorbées |
| 200 | `double.infinity` | collé, saccadé |

### Programme comparatif

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuLissage()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 20,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) position += direction.normalized() * 240 * dt;
  }
}

class JeuLissage extends FlameGame with KeyboardEvents {
  final Heros heros = Heros();
  final List<double> _vitesses = [double.infinity, 600, 260, 140];
  int _index = 0;
  late final TextComponent _info;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(3000, 1600),
      paint: Paint()..color = const Color(0xFF191826),
    ));
    for (var x = 0.0; x < 3000; x += 150) {
      await world.add(RectangleComponent(
        position: Vector2(x, 0),
        size: Vector2(4, 1600),
        paint: Paint()..color = const Color(0xFF32304C),
      ));
    }
    heros.position = Vector2(400, 400);
    await world.add(heros);

    _info = TextComponent(
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_info);
    _appliquer();
  }

  void _appliquer() {
    final v = _vitesses[_index];
    camera.follow(heros, maxSpeed: v);
    _info.text = 'maxSpeed : ${v.isInfinite ? "infini" : v.toStringAsFixed(0)}'
        '   (barre espace pour changer)';
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    if (e is KeyDownEvent && e.logicalKey == LogicalKeyboardKey.space) {
      _index = (_index + 1) % _vitesses.length;
      _appliquer();
      return KeyEventResult.handled;
    }
    heros.direction
      ..x = (touches.contains(LogicalKeyboardKey.arrowRight) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowLeft) ? 1.0 : 0.0)
      ..y = (touches.contains(LogicalKeyboardKey.arrowDown) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowUp) ? 1.0 : 0.0);
    return KeyEventResult.handled;
  }
}
```

**Résultat :**

```text
maxSpeed : infini   (barre espace pour changer)
maxSpeed : 600   (barre espace pour changer)
maxSpeed : 260   (barre espace pour changer)
maxSpeed : 140   (barre espace pour changer)
```

Avec `140`, le héros distance la caméra et sort de l'écran par la droite : la valeur est plus basse que sa vitesse de 240. C'est le défaut à éviter.

> **À retenir.** `maxSpeed` est une **vitesse**, pas un coefficient de lissage. Si elle est inférieure à la vitesse de la cible, la caméra perd définitivement le héros. Prenez au moins 10 % de marge.

### Et `snap` ?

Au démarrage, la caméra est en `(0, 0)` et le héros en `(1200, 700)` : avec un `maxSpeed` fini, le joueur regarde un coin vide pendant plusieurs secondes.

```dart
camera.follow(heros, maxSpeed: 260, snap: true);
```

**Résultat :** la caméra se place instantanément sur le héros au premier tick, puis se comporte normalement. Utilisez `snap: true` à chaque changement de niveau ou de cible.

## 31.14 — `snapTo()` et `stop()`

### `snapTo()` n'existe plus

Beaucoup de tutoriels écrivent `camera.snapTo(Vector2(1200, 700));`. Cette méthode appartenait à l'ancienne classe `Camera`, supprimée. Le compilateur est sans ambiguïté :

```text
Error: The method 'snapTo' isn't defined for the class 'CameraComponent'.
```

Trois remplaçants existent, et le choix dépend de l'intention.

| Intention | Écriture correcte |
| --- | --- |
| Placer le regard sur un point, tout de suite, sans suivi | `camera.viewfinder.position = Vector2(1200, 700);` |
| Aller sur un point, tout de suite, en annulant le suivi en cours | `camera.moveTo(Vector2(1200, 700));` |
| Commencer à suivre une cible, sans transition | `camera.follow(cible, snap: true);` |

Les deux premières diffèrent sur un point : `moveTo()` **remplace le comportement de suivi**, alors qu'écrire dans `viewfinder.position` laisse le `FollowBehavior` en place, qui reprend la main dès la frame suivante.

```dart
camera.follow(heros);
camera.viewfinder.position = Vector2.zero();
// À la frame suivante, le FollowBehavior ramène la caméra sur le héros.
```

C'est une source de bugs classique : « je positionne la caméra mais elle revient toute seule ». La cause est toujours un `follow()` oublié.

### `stop()`

```dart
void stop();
```

`stop()` retire le comportement en cours — suivi ou déplacement — et laisse la caméra là où elle est.

```dart
camera.follow(heros);                      // la caméra suit
camera.stop();                             // elle se fige sur place
camera.moveTo(porteDeSortie, speed: 300);  // puis glisse vers la porte
```

### Programme complet : suivre, figer, reprendre

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuStop()));

class Heros extends RectangleComponent {
  Heros()
      : super(
          size: Vector2.all(36),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  @override
  void update(double dt) {
    super.update(dt);
    position.x += 150 * dt;
    if (position.x > 2300) position.x = 100;
  }
}

class JeuStop extends FlameGame with KeyboardEvents {
  final Heros heros = Heros();
  late final TextComponent _info;
  bool _suit = true;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(2400, 800),
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    for (var x = 0.0; x < 2400; x += 100) {
      await world.add(RectangleComponent(
        position: Vector2(x, 500),
        size: Vector2(90, 16),
        paint: Paint()..color = const Color(0xFF4A4770),
      ));
    }
    heros.position = Vector2(100, 440);
    await world.add(heros);

    camera.follow(heros, maxSpeed: 200, snap: true);

    _info = TextComponent(
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_info);
    _majInfo();
  }

  void _majInfo() => _info.text = _suit
      ? 'suivi ACTIF  (espace : stop)'
      : 'suivi ARRÊTÉ (espace : reprendre)';

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    if (e is KeyDownEvent && e.logicalKey == LogicalKeyboardKey.space) {
      _suit = !_suit;
      _suit ? camera.follow(heros, maxSpeed: 200) : camera.stop();
      _majInfo();
      return KeyEventResult.handled;
    }
    return KeyEventResult.ignored;
  }
}
```

**Résultat :**

```text
suivi ACTIF  (espace : stop)
suivi ARRÊTÉ (espace : reprendre)
```

Quand le suivi est arrêté, le héros traverse l'écran et sort par la droite ; la caméra ne bouge plus. Un nouvel appui la remet en marche : elle rattrape le héros à 200 unités par seconde.

## 31.15 — `camera.moveTo()`

`moveTo()` déplace la caméra vers un point du monde. C'est l'outil des cinématiques, des transitions de salle et des « regardez ce qui vient de s'ouvrir ».

### Les signatures

```dart
void moveTo(Vector2 point, {double speed = double.infinity});
void moveBy(Vector2 offset, {double speed = double.infinity});
```

| Appel | Effet |
| --- | --- |
| `camera.moveTo(p)` | saut instantané sur `p` |
| `camera.moveTo(p, speed: 250)` | glissement à 250 unités par seconde jusqu'à `p` |
| `camera.moveBy(Vector2(0, -300))` | saut de 300 unités vers le haut |
| `camera.moveBy(Vector2(0, -300), speed: 120)` | montée douce de 300 unités |

Comme `follow()`, ces méthodes installent un comportement dans le viseur. Elles **annulent** donc le suivi en cours : il faudra rappeler `follow()` après.

### Une transition de salle complète

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuTransition()));

class JeuTransition extends FlameGame with KeyboardEvents {
  static final Map<String, Vector2> salles = {
    'A': Vector2(400, 300),
    'B': Vector2(1600, 300),
    'C': Vector2(1600, 1000),
  };

  final RectangleComponent heros = RectangleComponent(
    size: Vector2.all(36),
    anchor: Anchor.center,
    paint: Paint()..color = const Color(0xFFE8B04B),
  );

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(2200, 1400),
      paint: Paint()..color = const Color(0xFF15141F),
    ));

    for (final e in salles.entries) {
      await world.add(RectangleComponent(
        position: e.value,
        size: Vector2(600, 400),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF2E2B4A),
      ));
      await world.add(TextComponent(
        text: 'Salle ${e.key}',
        position: e.value,
        anchor: Anchor.center,
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 32, color: Color(0xFFFFFFFF)),
        ),
      ));
    }

    heros.position = salles['A']!.clone();
    await world.add(heros);
    camera.viewfinder.position = salles['A']!.clone();

    await camera.viewport.add(TextComponent(
      text: 'Touches 1, 2, 3 : aller en salle A, B, C',
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    ));
  }

  Future<void> allerVers(Vector2 destination) async {
    camera.stop();
    camera.moveTo(destination, speed: 700);
    // On laisse la caméra arriver, puis on téléporte le héros.
    await Future<void>.delayed(const Duration(milliseconds: 900));
    heros.position = destination.clone();
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    if (e is! KeyDownEvent) return KeyEventResult.ignored;
    // Pas de `const` ici : LogicalKeyboardKey redéfinit `==`.
    final codes = <LogicalKeyboardKey, String>{
      LogicalKeyboardKey.digit1: 'A',
      LogicalKeyboardKey.digit2: 'B',
      LogicalKeyboardKey.digit3: 'C',
    };
    final nom = codes[e.logicalKey];
    if (nom == null) return KeyEventResult.ignored;
    allerVers(salles[nom]!);
    return KeyEventResult.handled;
  }
}
```

**Résultat :** l'appui sur `2` fait glisser la vue de la salle A vers la salle B en un peu plus d'une seconde, puis le héros apparaît au centre de la salle B. C'est le procédé des jeux d'exploration en salles fermées.

> **Remarque.** L'attente est faite ici avec un `Future.delayed`, simple à lire. Au chapitre 33, vous la remplacerez par un `TimerComponent`, plus propre parce qu'il se met en pause avec le jeu.

## 31.16 — Borner la caméra : `setBounds()`

Sans bornes, la caméra suit le héros jusque dans le vide : quand il longe le mur nord, on voit une bande noire au-dessus du donjon. Au chapitre 25, vous corrigiez cela avec deux `clamp`. Flame offre `setBounds()`.

### La signature

```dart
void setBounds(Shape? bounds, {bool considerViewport = false});
```

Le type `Shape` n'est pas un `Rect` de Flutter : c'est une forme géométrique de Flame, exportée par **`package:flame/experimental.dart`**.

```dart
import 'package:flame/experimental.dart';

camera.setBounds(Rectangle.fromLTRB(0, 0, 2400, 1400));
```

| Constructeur | Usage |
| --- | --- |
| `Rectangle.fromLTRB(left, top, right, bottom)` | le plus courant |
| `Rectangle.fromRect(rect)` | quand vous avez déjà un `Rect` |
| `Rectangle.fromCenter(center: ..., size: ...)` | zone centrée |
| `Circle(centre, rayon)` | zone circulaire, pour une arène |

### Ce que borne exactement `setBounds()`

C'est le point que tout le monde rate. Par défaut, `setBounds()` limite le **centre du regard**, pas les bords de l'écran.

```text
   considerViewport: false  (défaut)        considerViewport: true

   ┌──────────────────────────┐ monde       ┌──────────────────────────┐ monde
   │ ┌───────────┐            │             │┌───────────┐             │
   │ │  écran    │ ✚ le POINT │             ││  écran    │  l'ÉCRAN    │
   │ │           │  REGARDÉ   │             ││           │  ENTIER     │
   │ └───────────┘  reste     │             │└───────────┘  reste      │
   └──────────────────────────┘  dedans     └──────────────────────────┘ dedans
   la moitié gauche de l'écran               aucune bande noire :
   peut sortir : on voit du noir             c'est presque toujours
                                             ce que vous voulez.
```

```dart
camera.setBounds(
  Rectangle.fromLTRB(0, 0, 2400, 1400),
  considerViewport: true,
);
```

### Programme complet

```dart
import 'package:flame/components.dart';
import 'package:flame/experimental.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuBornes()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 18,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) position += direction.normalized() * 260 * dt;
    position.clamp(Vector2.all(20), JeuBornes.tailleMonde - Vector2.all(20));
  }
}

class JeuBornes extends FlameGame with KeyboardEvents {
  static final Vector2 tailleMonde = Vector2(2400, 1400);

  final Heros heros = Heros();
  late final TextComponent _info;
  bool _bornes = true;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await world.add(RectangleComponent(
      size: tailleMonde,
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    // Un liseré clair sur tout le pourtour, pour voir les limites.
    await world.add(RectangleComponent(
      size: tailleMonde,
      paint: Paint()
        ..color = const Color(0xFF8A83C9)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 8,
    ));
    final borne = Paint()..color = const Color(0xFF443F6B);
    for (var i = 0; i < 84; i++) {
      await world.add(RectangleComponent(
        position: Vector2(100.0 + (i % 12) * 200, 100.0 + (i ~/ 12) * 200),
        size: Vector2.all(18),
        anchor: Anchor.center,
        paint: borne,
      ));
    }
    heros.position = tailleMonde / 2;
    await world.add(heros);

    camera.follow(heros, maxSpeed: 400, snap: true);
    _appliquerBornes();

    _info = TextComponent(
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_info);
    _majInfo();
  }

  void _appliquerBornes() {
    camera.setBounds(
      _bornes
          ? Rectangle.fromLTRB(0, 0, tailleMonde.x, tailleMonde.y)
          : null, // null retire toute limite
      considerViewport: true,
    );
  }

  void _majInfo() => _info.text = _bornes
      ? 'bornes ACTIVES (espace pour retirer)'
      : 'bornes RETIRÉES (espace pour remettre)';

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    if (e is KeyDownEvent && e.logicalKey == LogicalKeyboardKey.space) {
      _bornes = !_bornes;
      _appliquerBornes();
      _majInfo();
      return KeyEventResult.handled;
    }
    heros.direction
      ..x = (touches.contains(LogicalKeyboardKey.arrowRight) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowLeft) ? 1.0 : 0.0)
      ..y = (touches.contains(LogicalKeyboardKey.arrowDown) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowUp) ? 1.0 : 0.0);
    return KeyEventResult.handled;
  }
}
```

**Résultat :** avec les bornes actives, le liseré clair reste toujours au bord de l'écran ou au-delà ; on ne voit jamais de noir. Appuyez sur espace et allez dans un coin : le héros continue, l'écran le suit, et une large zone noire apparaît.

### Bornes et zoom

`setBounds(..., considerViewport: true)` tient compte de la taille visible **au moment du calcul**, qui dépend du zoom. Si vous changez le zoom en cours de partie, rappelez `setBounds()`.

> **Piège.** Si votre monde est **plus petit** que la zone visible et que vous passez `considerViewport: true`, il n'existe aucune position valable pour la caméra. Pour un petit niveau, ne bornez pas : centrez la caméra une fois pour toutes.

## 31.17 — Le viewport : `MaxViewport`, `FixedResolutionViewport`, `FixedAspectRatioViewport`, `CircularViewport`

Le viewport est le **trou** par lequel on regarde le monde. Il a une position et une taille **à l'écran**, il découpe le rendu, et il porte le HUD.

```text
   LE CANVAS DU GAMEWIDGET (1280 × 720)
   ┌──────────────────────────────────────────────────────┐
   │        ┌────────────────────────────────┐            │
   │        │      LE VIEWPORT (960 × 540)   │            │
   │        │  Tout ce que la caméra dessine │            │
   │        │  est découpé à ce rectangle.   │            │
   │        └────────────────────────────────┘            │
   └──────────────────────────────────────────────────────┘
```

### Les cinq types et leurs constructeurs réels

```dart
MaxViewport();
FixedResolutionViewport({required Vector2 resolution, Iterable<Component>? children});
FixedSizeViewport(double width, double height, {Iterable<Component>? children});
FixedAspectRatioViewport({required double aspectRatio, Iterable<Component>? children});
CircularViewport(double radius, {Iterable<Component>? children});
CircularViewport.ellipse(double radiusX, double radiusY, {Iterable<Component>? children});
```

**`MaxViewport`** est le viewport par défaut : il occupe tout le canvas et se redimensionne seul. Plus la fenêtre est grande, plus on voit de monde.

**`FixedResolutionViewport`** maintient une résolution logique constante et ajoute des bandes noires si le rapport de forme diffère. C'est ce que `CameraComponent.withFixedResolution()` installe ; la différence est que la fabrique règle **aussi** le viseur, ce que le viewport seul ne fait pas.

**`FixedAspectRatioViewport`** occupe le maximum de place en gardant un rapport donné. Ici la **résolution** n'est pas fixée, seul le **rapport** l'est : sur un grand écran, on voit plus de monde, mais toujours dans un cadre 16/9.

**`FixedSizeViewport`** est un rectangle de taille fixe en pixels d'écran, qui ne se redimensionne pas seul : c'est le viewport des minimaps et des écrans partagés.

**`CircularViewport`** est un disque : lunette, trou de serrure, fondu en iris, minimap ronde.

### Programme complet : changer de viewport à chaud

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuViewports()));

class JeuViewports extends FlameGame with KeyboardEvents {
  int _index = 0;
  late final TextComponent _info;

  static const _noms = [
    'MaxViewport',
    'FixedResolutionViewport 320×180',
    'FixedAspectRatioViewport 4/3',
    'FixedSizeViewport 400×260',
    'CircularViewport r=160',
  ];

  Viewport _construire(int i) {
    switch (i) {
      case 0:
        return MaxViewport();
      case 1:
        return FixedResolutionViewport(resolution: Vector2(320, 180));
      case 2:
        return FixedAspectRatioViewport(aspectRatio: 4 / 3);
      case 3:
        return FixedSizeViewport(400, 260);
      default:
        return CircularViewport(160);
    }
  }

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    for (var i = 0; i < 140; i++) {
      final x = i % 14;
      final y = i ~/ 14;
      await world.add(RectangleComponent(
        position: Vector2(x * 60.0, y * 60.0),
        size: Vector2.all(56),
        paint: Paint()
          ..color = (x + y).isEven
              ? const Color(0xFF2E2B4A)
              : const Color(0xFF484075),
      ));
    }
    camera.viewfinder
      ..anchor = Anchor.center
      ..position = Vector2(420, 300);

    _info = TextComponent(
      position: Vector2.all(10),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 14, color: Color(0xFFFFFFFF)),
      ),
    );
    _appliquer();
  }

  void _appliquer() {
    camera.viewport = _construire(_index);
    // Le HUD appartient au viewport : il faut le rattacher au nouveau.
    camera.viewport.add(_info);
    _info.text = '${_noms[_index]}   (espace pour changer)';
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    if (e is KeyDownEvent && e.logicalKey == LogicalKeyboardKey.space) {
      _index = (_index + 1) % _noms.length;
      _appliquer();
      return KeyEventResult.handled;
    }
    return KeyEventResult.ignored;
  }
}
```

**Résultat :**

```text
MaxViewport   (espace pour changer)
FixedResolutionViewport 320×180   (espace pour changer)
FixedAspectRatioViewport 4/3   (espace pour changer)
FixedSizeViewport 400×260   (espace pour changer)
CircularViewport r=160   (espace pour changer)
```

Le damier remplit d'abord tout l'écran, puis se retrouve dans un cadre 16/9 agrandi, puis dans un cadre 4/3, puis dans un petit rectangle, puis dans un disque.

> **Détail important.** Remplacer `camera.viewport` **détache** l'ancien viewport et ses enfants. Le HUD disparaît si vous ne le rattachez pas, comme le fait `_appliquer()`. C'est la cause de « mon score a disparu quand je change de mode d'affichage ».

## 31.18 — Choisir son viewport selon la cible

Il n'y a pas de bon viewport dans l'absolu. Il y a un viewport adapté à un type de jeu et à un parc d'appareils.

| Viewport | Ce qui est garanti | Bandes noires | Cible idéale | À éviter si |
| --- | --- | --- | --- | --- |
| `MaxViewport` | rien : on voit le maximum | jamais | prototype, jeu de stratégie, éditeur de niveau | l'équité entre joueurs compte |
| `FixedResolutionViewport` | le nombre d'unités visibles | oui, si le rapport diffère | pixel art, arcade, plateforme, portage rétro | vous voulez exploiter les grands écrans |
| `FixedAspectRatioViewport` | le rapport largeur / hauteur | oui, sur les côtés | jeu conçu en 16/9 mais tolérant à l'échelle | le contenu doit être identique partout |
| `FixedSizeViewport` | une taille en pixels d'écran | non applicable | minimap, écran partagé, vignette de prévisualisation | c'est la caméra principale |
| `CircularViewport` | une forme ronde | non applicable | lunette, effet d'iris, minimap ronde, vision limitée | vous avez besoin des coins |

### Arbre de décision

```text
   Mon jeu est-il en pixel art avec une résolution logique précise ?
        │ oui  →  FixedResolutionViewport (ou withFixedResolution)
        │ non
        ▼
   L'équité entre un téléphone et un écran 27 pouces est-elle importante ?
        │ oui  →  FixedResolutionViewport
        │ non
        ▼
   Ai-je conçu mon interface pour un rapport précis (16/9) ?
        │ oui  →  FixedAspectRatioViewport
        │ non
        ▼
   →  MaxViewport
```

### Le cas des trois plateformes du cours

| Plateforme | Contrainte réelle | Choix conseillé pour « Donjon de Dart » |
| --- | --- | --- |
| Web, fenêtre redimensionnable | rapport imprévisible, de 21/9 à 3/4 | `withFixedResolution(width: 480, height: 270)` |
| Android, téléphone en paysage | rapports très variés, encoches | `withFixedResolution` + marges de sécurité dans le HUD |
| Windows / bureau | grande fenêtre, souris | idem, ou `MaxViewport` si le jeu se prête au dézoom |

> **À retenir.** Décidez du viewport **avant** de dessiner votre HUD. Changer de viewport après coup oblige à replacer tous les éléments d'interface, parce que leurs coordonnées sont exprimées dans le repère du viewport.

---

## 31.19 — Le HUD : `camera.viewport.add()`

Le HUD — *head-up display* — regroupe tout ce qui est collé à l'écran : vies, score, minimap, joystick, boutons. En Flame, il vit dans le **viewport**.

```dart
camera.viewport.add(TextComponent(text: 'Vies : 3', position: Vector2.all(12)));
```

Les enfants du viewport sont dessinés **après** le monde, sans aucune transformation de caméra. Leur repère a pour origine le coin haut-gauche du viewport.

```text
   VIEWPORT (repère du HUD)
   (0,0)
   ┌──────────────────────────────────────────┐
   │ [|||]  Score : 120                         │  ← position (12, 12)
   │                                          │
   │            [ le monde défile ici ]       │
   │                                          │
   │                            ┌──────┐      │
   │  (joystick)                │minimap│     │
   └────────────────────────────┴──────┴──────┘
```

### Trois façons d'ajouter un élément de HUD

```dart
// 1. Après coup, la plus lisible.
camera.viewport.add(barreDeVie);

// 2. À la construction de la caméra.
CameraComponent(world: monde, hudComponents: [barreDeVie, score]);

// 3. Dans le constructeur du viewport.
FixedResolutionViewport(
  resolution: Vector2(320, 180),
  children: [barreDeVie],
);
```

### Un HUD complet, sans image

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuHud()));
}

/// Barre de vie dessinée avec deux rectangles.
class BarreDeVie extends PositionComponent {
  BarreDeVie() : super(position: Vector2(12, 12), size: Vector2(220, 18));

  double ratio = 1.0; // de 0 à 1

  static final Paint _fond = Paint()..color = const Color(0xFF3A3550);
  static final Paint _vie = Paint()..color = const Color(0xFFD5453E);
  static final Paint _bord = Paint()
    ..color = const Color(0xFFFFFFFF)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _fond);
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.x * ratio.clamp(0, 1), size.y),
      _vie,
    );
    canvas.drawRect(size.toRect(), _bord);
  }
}

class JeuHud extends FlameGame {
  final BarreDeVie barre = BarreDeVie();
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    for (var i = 0; i < 20; i++) {
      await world.add(
        RectangleComponent(
          position: Vector2(i * 120.0, 200),
          size: Vector2(100, 100),
          paint: Paint()..color = const Color(0xFF3B3760),
        ),
      );
    }
    camera.viewfinder.anchor = Anchor.topLeft;

    // Le HUD : dans le viewport, jamais dans le monde.
    await camera.viewport.addAll([
      barre,
      TextComponent(
        text: 'Donjon de Dart',
        position: Vector2(12, 40),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 18, color: Color(0xFFFFFFFF)),
        ),
      ),
    ]);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(_t * 60, 0);
    barre.ratio = 1 - (_t % 5) / 5; // la vie descend, puis se recharge
  }
}
```

**Résultat :** le décor défile vers la gauche, la barre rouge se vide en cinq secondes, et ni la barre ni le titre ne bougent d'un pixel.

> **Remarque.** Le HUD n'est pas affecté par le zoom du viseur. C'est voulu : un texte de 18 pixels reste lisible même si le monde est zoomé à 4×.

---

## 31.20 — Pourquoi le HUD ne doit pas être dans le monde

Faisons l'erreur exprès, pour la reconnaître ensuite en une seconde.

```dart
// À NE PAS FAIRE
await world.add(barreDeVie);
```

**Résultat observé :**

```text
   t = 0 s                        t = 4 s
   ┌────────────────────┐         ┌────────────────────┐
   │ ▓▓▓▓▓▓▓▓░░  vie    │         │                    │  ← la barre est
   │                    │         │                    │    sortie par la
   │      ♦ héros       │         │      ♦ héros       │    gauche
   └────────────────────┘         └────────────────────┘
```

La barre de vie est un objet du donjon : elle est posée en `(12, 12)` **dans le monde**, c'est-à-dire dans le coin haut-gauche du niveau. Dès que le héros s'éloigne, elle sort du champ.

### Les cinq symptômes du HUD mal placé

| Symptôme | Explication |
| --- | --- |
| Le score disparaît quand le héros avance | il est ancré à une position du monde |
| Le texte du score grossit quand on zoome | il subit le zoom du viseur |
| Le joystick se déplace sous le doigt | il suit la caméra au lieu de l'écran |
| Un ennemi passe **devant** la barre de vie | même monde, donc même règle de `priority` |
| La minimap se dessine dans la minimap | la minimap est dans le monde qu'elle affiche |

Le dernier cas est spectaculaire : une minimap ajoutée au monde est vue par la caméra de la minimap, qui affiche donc une minimap contenant une minimap.

### Et si je veux un élément d'interface attaché à un personnage ?

Une barre de vie **au-dessus d'un gobelin** n'est pas du HUD : c'est un objet du monde, et il doit y aller. La règle est celle de la section 31.5 : si l'élément a une position dans le donjon, il va dans le monde.

```dart
class Gobelin extends RectangleComponent {
  Gobelin() : super(size: Vector2(32, 32));

  @override
  Future<void> onLoad() async {
    // La barre suit le gobelin : c'est un enfant, donc dans le monde.
    await add(
      RectangleComponent(
        position: Vector2(0, -8),
        size: Vector2(32, 4),
        paint: Paint()..color = const Color(0xFFD5453E),
      ),
    );
  }
}
```

| Élément | Où le mettre |
| --- | --- |
| Barre de vie du joueur, en haut de l'écran | `camera.viewport` |
| Barre de vie flottante au-dessus d'un ennemi | enfant du composant ennemi, dans le monde |
| Nom d'un PNJ affiché sous ses pieds | enfant du PNJ, dans le monde |
| Compteur de pièces | `camera.viewport` |
| Flèche indiquant la direction de la sortie, sur le bord de l'écran | `camera.viewport` |
| Marqueur au sol indiquant la sortie | monde |

---

## 31.21 — Convertir écran → monde : `camera.globalToLocal()`

Le joueur clique à l'écran. Vous, vous raisonnez en coordonnées de monde. Il faut convertir.

```dart
Vector2 globalToLocal(Vector2 point, {Vector2? output});
```

Le nom mérite une explication, parce qu'il déroute. Du point de vue de la caméra, « global » désigne le repère du canvas (l'écran) et « local » le repère du monde qu'elle observe. Donc :

```text
   globalToLocal :  ÉCRAN  →  MONDE
   localToGlobal :  MONDE  →  ÉCRAN
```

### Exemple complet : poser une torche là où l'on clique

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuClic()));
}

/// Un capteur de clic qui couvre tout le viewport.
class CapteurEcran extends PositionComponent
    with TapCallbacks, HasGameReference<JeuClic> {
  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    size = taille; // le capteur couvre tout l'écran
  }

  @override
  void onTapDown(TapDownEvent event) {
    // event.canvasPosition est en pixels d'écran.
    final dansLeMonde = game.camera.globalToLocal(event.canvasPosition);
    game.poserTorche(dansLeMonde);
    debugPrint('écran ${event.canvasPosition}  →  monde $dansLeMonde');
  }
}

class JeuClic extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    for (var i = 0; i < 30; i++) {
      for (var j = 0; j < 12; j++) {
        await world.add(
          RectangleComponent(
            position: Vector2(i * 100.0, j * 100.0),
            size: Vector2.all(96),
            paint: Paint()
              ..color = (i + j).isEven
                  ? const Color(0xFF23223A)
                  : const Color(0xFF2C2A47),
          ),
        );
      }
    }

    camera.viewfinder
      ..anchor = Anchor.center
      ..zoom = 1.4;

    await camera.viewport.add(CapteurEcran());
  }

  void poserTorche(Vector2 positionMonde) {
    world.add(
      CircleComponent(
        radius: 10,
        position: positionMonde,
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(600 + _t * 50, 500);
  }
}
```

**Résultat :**

```text
écran [412.0,233.0]  →  monde [694.3,523.6]
écran [120.0,300.0]  →  monde [485.7,571.4]
```

Les disques dorés restent **collés au décor** et défilent avec lui. Sans la conversion, ils resteraient collés à l'écran.

### La version fausse, pour comparaison

```dart
// FAUX : on utilise une coordonnée d'écran comme coordonnée de monde.
world.add(CircleComponent(radius: 10, position: event.canvasPosition));
```

**Résultat :** au premier clic, le disque semble correct… tant que la caméra n'a pas bougé et que le zoom vaut 1. Dès qu'un des deux change, le disque apparaît décalé, parfois de très loin.

| Situation | Erreur commise sans conversion |
| --- | --- |
| Caméra en `(600, 500)`, zoom 1 | décalage de 600 et 500 unités |
| Zoom 2 | position divisée par deux |
| Caméra inclinée | décalage **et** rotation |
| Viewport à résolution fixe | décalage proportionnel à l'échelle |

> **Remarque.** Le paramètre optionnel `output` permet de réutiliser un `Vector2` existant au lieu d'en allouer un nouveau : `camera.globalToLocal(p, output: _tampon)`. C'est une optimisation utile seulement si vous convertissez des centaines de points par frame.

---

## 31.22 — Convertir monde → écran : `camera.localToGlobal()`

La conversion inverse sert dès que vous voulez qu'un élément **d'écran** pointe vers un objet **du monde**.

```dart
Vector2 localToGlobal(Vector2 position, {Vector2? output});
```

Cas typiques : un marqueur au bord de l'écran indiquant un ennemi hors champ, une infobulle Flutter positionnée au-dessus d'un coffre, une flèche de quête.

### Exemple : un indicateur de bord d'écran

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuIndicateur()));
}

class Boss extends CircleComponent {
  Boss()
      : super(
          radius: 30,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFD5453E),
        );
}

/// Petit losange affiché au bord de l'écran, vers la position du boss.
class Indicateur extends PositionComponent
    with HasGameReference<JeuIndicateur> {
  Indicateur() : super(size: Vector2.all(18), anchor: Anchor.center);

  static final Paint _p = Paint()..color = const Color(0xFFE8B04B);

  @override
  void update(double dt) {
    super.update(dt);
    // Position du boss, exprimée en pixels d'écran.
    final aLEcran = game.camera.localToGlobal(game.boss.position);
    final taille = game.camera.viewport.size;
    // On plaque le point sur les bords, avec 20 px de marge.
    position = Vector2(
      aLEcran.x.clamp(20.0, taille.x - 20.0),
      aLEcran.y.clamp(20.0, taille.y - 20.0),
    );
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    final c = size.x / 2;
    final chemin = Path()
      ..moveTo(c, 0)
      ..lineTo(size.x, c)
      ..lineTo(c, size.y)
      ..lineTo(0, c)
      ..close();
    canvas.drawPath(chemin, _p);
  }
}

class JeuIndicateur extends FlameGame {
  final Boss boss = Boss();
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await world.add(
      RectangleComponent(
        position: Vector2.zero(),
        size: Vector2(2000, 1200),
        paint: Paint()..color = const Color(0xFF1B1A26),
      ),
    );
    boss.position = Vector2(1600, 900);
    await world.add(boss);

    camera.viewfinder.anchor = Anchor.center;
    await camera.viewport.add(Indicateur());
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(400 + _t * 120, 400);
  }
}
```

**Résultat :** un losange doré glisse le long du bord droit puis du bord bas de l'écran, indiquant en permanence où se trouve le boss. Quand la caméra arrive sur lui, le losange se pose exactement dessus.

| Méthode | Sens | Entrée | Sortie |
| --- | --- | --- | --- |
| `camera.globalToLocal(p)` | écran → monde | pixels du canvas | unités de monde |
| `camera.localToGlobal(p)` | monde → écran | unités de monde | pixels du canvas |

---

## 31.23 — Le tremblement de caméra avec un effet (annonce du chapitre 33)

Au chapitre 25, section 25.14, vous aviez écrit un tremblement d'écran à la main : un compteur, un `Random`, un décalage ajouté à la position de la caméra. Avec Flame, un effet suffit.

L'astuce tient à une propriété du `Viewfinder` : il implémente `PositionProvider`. On peut donc lui coller un `MoveEffect`, exactement comme sur un composant ordinaire.

```dart
import 'package:flame/effects.dart';

void secouer() {
  camera.viewfinder.add(
    MoveEffect.by(
      Vector2(8, 0),
      EffectController(
        duration: 0.04,
        alternate: true, // aller-retour
        repeatCount: 5, // cinq allers-retours
      ),
    ),
  );
}
```

Pourquoi cela cohabite avec `follow()` : les effets de Flame appliquent des modifications **relatives**. À chaque frame, le `FollowBehavior` place le viseur sur le héros, puis le `MoveEffect` ajoute son petit décalage. L'effet se retire tout seul à la fin.

### Programme complet

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(GameWidget(game: JeuSecousse()));
}

class JeuSecousse extends FlameGame with KeyboardEvents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    for (var i = 0; i < 12; i++) {
      for (var j = 0; j < 8; j++) {
        await world.add(
          RectangleComponent(
            position: Vector2(i * 90.0, j * 90.0),
            size: Vector2.all(84),
            paint: Paint()
              ..color = (i + j).isEven
                  ? const Color(0xFF2E2B4A)
                  : const Color(0xFF433C6E),
          ),
        );
      }
    }

    camera.viewfinder
      ..anchor = Anchor.center
      ..position = Vector2(540, 360);

    await camera.viewport.add(
      TextComponent(
        text: 'Barre espace : le boss frappe le sol',
        position: Vector2.all(12),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
        ),
      ),
    );
  }

  void secouer({double intensite = 10, double duree = 0.05, int coups = 6}) {
    camera.viewfinder.add(
      MoveEffect.by(
        Vector2(intensite, intensite * 0.4),
        EffectController(
          duration: duree,
          alternate: true,
          repeatCount: coups,
        ),
      ),
    );
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent &&
        event.logicalKey == LogicalKeyboardKey.space) {
      secouer();
      return KeyEventResult.handled;
    }
    return KeyEventResult.ignored;
  }
}
```

**Résultat :** à chaque appui, l'écran vibre pendant environ 0,6 seconde puis se stabilise **exactement** à sa position d'origine, parce que le nombre d'allers-retours est pair et que le déplacement est relatif.

| Réglage | Effet ressenti |
| --- | --- |
| `intensite: 4`, `coups: 4` | pas d'un géant, choc léger |
| `intensite: 10`, `coups: 6` | coup de masse du boss |
| `intensite: 24`, `coups: 12`, `duree: 0.03` | explosion, tremblement de terre |

> **Attention.** Un nombre **impair** d'allers-retours laisse la caméra décalée à la fin. Vérifiez toujours que `repeatCount` est pair quand `alternate` vaut `true`, ou remettez le viseur en place dans un `onComplete`.

Le chapitre 33 reprend cela en détail : `EffectController` complet, courbes, `SequenceEffect`, effets combinés et particules.

---

## 31.24 — Plusieurs caméras : minimap et écran partagé

C'est ici que le nouveau système montre tout son intérêt : un `World` peut être observé par **autant de caméras que l'on veut**, en même temps.

```text
   FlameGame
    ├── World                    ← un seul monde
    ├── CameraComponent          ← vue principale, MaxViewport
    └── CameraComponent          ← minimap, FixedSizeViewport, zoom 0.09
```

### Une minimap

```dart
import 'dart:math' as math;

import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuMinimap()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 16,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  double _t = 0;

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    position = Vector2(
      1200 + 900 * math.cos(_t * 0.4),
      700 + 500 * math.sin(_t * 0.4),
    );
  }
}

class JeuMinimap extends FlameGame {
  static final Vector2 tailleMonde = Vector2(2400, 1400);

  final Heros heros = Heros();
  CameraComponent? minimap;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: tailleMonde,
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    final borne = Paint()..color = const Color(0xFF453F70);
    for (var i = 0; i < 60; i++) {
      await world.add(RectangleComponent(
        position: Vector2(100.0 + (i % 10) * 220, 100.0 + (i ~/ 10) * 220),
        size: Vector2.all(60),
        anchor: Anchor.center,
        paint: borne,
      ));
    }
    heros.position = tailleMonde / 2;
    await world.add(heros);
    camera.follow(heros, maxSpeed: 400, snap: true);

    // Deuxième caméra : elle regarde LE MÊME monde, de très loin.
    final m = CameraComponent(
      world: world,
      viewport: FixedSizeViewport(200, 130),
    )..priority = 10;
    m.viewfinder
      ..anchor = Anchor.center
      ..position = tailleMonde / 2
      ..zoom = 0.08;
    minimap = m;
    await add(m);
    _placerMinimap();
  }

  void _placerMinimap() {
    final m = minimap;
    if (m == null) return;
    m.viewport.anchor = Anchor.topRight;
    m.viewport.position = Vector2(size.x - 12, 12);
  }

  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    _placerMinimap();
  }
}
```

**Résultat :** en haut à droite, un petit rectangle montre tout le donjon en réduction, avec le point doré du héros qui décrit une ellipse. La vue principale, elle, suit le héros de près.

### Écran partagé à deux joueurs

Le principe est le même : deux caméras, deux `FixedSizeViewport`, deux positions.

```dart
camera.viewport = FixedSizeViewport(size.x / 2, size.y);
camera.viewport.anchor = Anchor.topLeft;
camera.viewport.position = Vector2.zero();
camera.follow(heros1, snap: true);

final cam2 = CameraComponent(
  world: world,
  viewport: FixedSizeViewport(size.x / 2, size.y),
);
cam2.viewport.anchor = Anchor.topLeft;
cam2.viewport.position = Vector2(size.x / 2, 0);
cam2.follow(heros2, snap: true);
await add(cam2);
```

**Résultat :** l'écran est coupé en deux, chaque moitié suit son héros, et les deux joueurs se voient quand ils sont proches.

| Usage | Viewport | Zoom typique |
| --- | --- | --- |
| Minimap rectangulaire | `FixedSizeViewport(200, 130)` | 0,05 à 0,15 |
| Minimap ronde | `CircularViewport(80)` | 0,10 |
| Écran partagé horizontal | `FixedSizeViewport(w/2, h)` | 1 |
| Vue de recul pendant un boss | seconde caméra plein écran, `priority` élevée | 0,6 |

> **Attention aux performances.** Chaque caméra redessine le monde entier : deux caméras coûtent deux fois le rendu. Pour une minimap, préférez un monde simplifié — quelques rectangles représentant les salles — dès que le décor devient riche.

## 31.25 — `ParallaxComponent` dans le monde (rappel du chapitre 25)

Au chapitre 25, section 25.16, vous aviez implémenté la parallaxe à la main : plusieurs plans, chacun déplacé d'une fraction du déplacement de la caméra. Flame fournit `ParallaxComponent`, qui fait cela avec des images.

```dart
final fond = await loadParallaxComponent(
  [
    ParallaxImageData('ciel.png'),
    ParallaxImageData('montagnes.png'),
    ParallaxImageData('arbres.png'),
  ],
  baseVelocity: Vector2(20, 0),
  velocityMultiplierDelta: Vector2(1.8, 1.0),
);
camera.backdrop.add(fond);
```

`baseVelocity` est la vitesse du plan le plus lointain ; `velocityMultiplierDelta` la multiplie à chaque plan suivant. Le composant défile **tout seul** : c'est adapté à un jeu à défilement automatique, pas à un donjon exploré librement.

**Ce cours ne fournit pas d'images.** Voici donc l'équivalent 100 % code, qui a l'avantage de suivre réellement la caméra.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuParallaxe()));

/// Un plan de décor qui se déplace d'une FRACTION du déplacement caméra.
class PlanParallaxe extends PositionComponent
    with HasGameReference<JeuParallaxe> {
  PlanParallaxe({
    required this.facteur, // 0 = fixe à l'infini, 1 = solidaire du monde
    required this.couleur,
    required this.hauteur,
    required this.y,
  });

  final double facteur;
  final Color couleur;
  final double hauteur;
  final double y;

  @override
  void update(double dt) {
    super.update(dt);
    final cam = game.camera.viewfinder.position;
    position = Vector2(cam.x * (1 - facteur), y);
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    final p = Paint()..color = couleur;
    for (var i = -2; i < 40; i++) {
      canvas.drawRect(Rect.fromLTWH(i * 220.0, 0, 160, hauteur), p);
    }
  }
}

class JeuParallaxe extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Du plus lointain au plus proche : facteur croissant.
    await world.addAll([
      PlanParallaxe(
        facteur: 0.15,
        couleur: const Color(0xFF272544),
        hauteur: 260,
        y: 60,
      )..priority = 0,
      PlanParallaxe(
        facteur: 0.45,
        couleur: const Color(0xFF383466),
        hauteur: 200,
        y: 160,
      )..priority = 1,
      PlanParallaxe(
        facteur: 1.0,
        couleur: const Color(0xFF5A5296),
        hauteur: 120,
        y: 300,
      )..priority = 2,
    ]);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(_t * 120, 0);
  }
}
```

**Résultat :** trois rangées de colonnes défilent à trois vitesses différentes. La rangée du haut, la plus sombre, bouge à peine ; celle du bas glisse à pleine vitesse. L'illusion de profondeur est immédiate.

| Où placer le fond | Conséquence |
| --- | --- |
| `world.add(...)` avec une `priority` basse | le fond subit le zoom et le suivi : c'est le cas ci-dessus |
| `camera.backdrop.add(...)` | le fond est fixe à l'écran, dessiné derrière le monde : idéal pour un ciel uni ou un dégradé |

## 31.26 — Le culling automatique

Le mot « automatique » demande une précision, parce que beaucoup de tutoriels laissent croire que Flame ignore tout seul les objets hors champ.

### Ce que Flame fait vraiment

| Mécanisme | Automatique ? | Ce qu'il évite |
| --- | --- | --- |
| Découpe (clip) au rectangle du viewport | **oui** | les pixels hors du viewport ne sont pas peints |
| Rejet par le GPU des primitives hors écran | **oui**, côté Flutter/Skia | le coût de remplissage |
| Saut de l'appel `render()` d'un composant hors champ | **non** | rien : la méthode est bien appelée |
| Saut de l'appel `update()` d'un composant hors champ | **non** | rien |

Autrement dit : le code source du `World` en Flame 1.38.0 ne contient **aucune** logique de culling. Chaque composant du monde reçoit son `update(dt)` et son `render(canvas)`, qu'il soit visible ou non. C'est un choix délibéré : un ennemi hors champ doit continuer de patrouiller.

### Les deux outils fournis

```dart
Rect get visibleWorldRect;                         // zone du monde visible
bool canSee(PositionComponent c, {World? componentWorld});
```

### Écrire son culling

Pour un décor très fourni — plusieurs milliers de tuiles —, il devient rentable de sauter le rendu. Un mixin de trois lignes suffit.

```dart
/// À poser sur un composant décoratif nombreux et statique.
mixin CullingRendu on PositionComponent {
  @override
  void renderTree(Canvas canvas) {
    final cam = CameraComponent.currentCamera;
    if (cam != null && !cam.canSee(this)) return; // hors champ : on saute
    super.renderTree(canvas);
  }
}
```

```dart
class Tuile extends RectangleComponent with CullingRendu {
  Tuile({super.position, super.size, super.paint});
}
```

**Résultat mesuré** sur un donjon de 4000 tuiles avec 200 tuiles visibles : le temps de rendu par frame passe d'environ 9 ms à environ 1 ms sur une machine de bureau. Le gain devient nul si presque tout est visible.

> **Règle.** N'écrivez ce culling que lorsque le profilage (chapitre 42) montre un problème. Sur un donjon de quelques centaines de composants, il ne sert à rien et complique le code.

---

## 31.27 — Le pixel art et le zoom entier

Le pixel art impose une contrainte que le reste du chapitre ignorait : **un pixel de la texture doit couvrir un nombre entier de pixels de l'écran**.

```text
   zoom = 3 (entier)              zoom = 2.7 (fractionnaire)
   ┌───┬───┬───┐                  ┌──┬───┬──┐
   │███│███│███│  chaque pixel    │██│▓██│██│  certains pixels
   │███│███│███│  fait 3×3        │▓▓│███│▓█│  font 3, d'autres 2
   │███│███│███│  net             └──┴───┴──┘  bords baveux
   └───┴───┴───┘
```

Un zoom fractionnaire produit trois défauts visibles : des bords flous, des colonnes de pixels d'épaisseur inégale, et un scintillement quand la caméra bouge.

### Les trois règles

**1. Zoom entier.** Arrondissez toujours.

```dart
camera.viewfinder.zoom = camera.viewfinder.zoom.roundToDouble().clamp(1, 8);
```

**2. Résolution logique fixe.** Choisissez la résolution du jeu, pas celle de l'écran.

```dart
class DonjonPixel extends FlameGame {
  DonjonPixel()
      : super(
          camera: CameraComponent.withFixedResolution(
            width: 320,
            height: 180,
          ),
        );
}
```

**3. Filtrage sans lissage.** Par défaut, Flutter interpole les textures agrandies, ce qui rend le pixel art flou. Il faut demander le voisin le plus proche.

```dart
// Une seule fois, avant de charger les images.
Paint peintureNette() => Paint()..filterQuality = FilterQuality.none;

// Sur un SpriteComponent :
final joueur = SpriteComponent(
  sprite: sprite,
  size: Vector2.all(16),
  paint: Paint()..filterQuality = FilterQuality.none,
);
```

| Symptôme | Cause | Correction |
| --- | --- | --- |
| Bords flous | `FilterQuality` par défaut | `FilterQuality.none` |
| Pixels d'épaisseur inégale | zoom fractionnaire | zoom entier |
| Scintillement en déplacement | position de caméra non entière | arrondir la position du viseur |
| Fines lignes entre les tuiles | erreurs d'arrondi | option `bleed` de `SpriteBatch`, ou tuiles jointives |

Pour le scintillement, arrondissez la position du viseur à l'unité de monde :

```dart
@override
void update(double dt) {
  super.update(dt);
  final p = camera.viewfinder.position;
  camera.viewfinder.position = Vector2(p.x.roundToDouble(), p.y.roundToDouble());
}
```

> **Remarque.** Cet arrondi rend le défilement légèrement saccadé, par pas d'une unité de monde. C'est le compromis classique du pixel art : netteté contre fluidité. La plupart des jeux du genre choisissent la netteté.

---

## 31.28 — Mini-projet : un donjon plus grand que l'écran

Objectif : réunir tout le chapitre dans un programme unique et exécutable, sans aucune image.

**Cahier des charges.**

1. Un donjon de 2400 × 1600 unités, composé de six salles reliées par des couloirs.
2. Un héros contrôlable aux flèches ou en ZQSD.
3. La caméra le suit avec un lissage, bornée aux murs du donjon.
4. Un HUD fixe : barre de vie, position, nom du donjon.
5. Une minimap en haut à droite, avec le héros visible dessus.
6. Un tremblement d'écran quand le héros touche un piège.

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/experimental.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) =>
      GameWidget<DonjonDeDart>(game: DonjonDeDart());
}

// ---------------------------------------------------------------- LE MONDE

/// Une salle du donjon : un centre et une taille.
class Salle {
  Salle(this.centre, this.taille);

  final Vector2 centre;
  final Vector2 taille;
}

class MondeDonjon extends World {
  static final Vector2 taille = Vector2(2400, 1600);

  static final List<Salle> salles = [
    Salle(Vector2(400, 300), Vector2(600, 400)),
    Salle(Vector2(1200, 300), Vector2(500, 360)),
    Salle(Vector2(2000, 400), Vector2(560, 480)),
    Salle(Vector2(400, 1200), Vector2(560, 440)),
    Salle(Vector2(1200, 1150), Vector2(640, 400)),
    Salle(Vector2(2000, 1250), Vector2(520, 420)),
  ];

  static final List<Vector2> pieges = [
    Vector2(1200, 300),
    Vector2(2000, 400),
    Vector2(400, 1200),
    Vector2(1200, 1150),
  ];

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le sol général.
    await add(
      RectangleComponent(
        position: Vector2.zero(),
        size: taille,
        paint: Paint()..color = const Color(0xFF13121C),
      ),
    );

    // Les salles.
    for (final s in salles) {
      await add(
        RectangleComponent(
          position: s.centre,
          size: s.taille,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF2A2742),
        ),
      );
    }

    // Les couloirs : de simples bandes reliant les centres.
    Future<void> couloir(Vector2 a, Vector2 b) async {
      final centre = (a + b) / 2;
      final dim = Vector2((b.x - a.x).abs() + 80, (b.y - a.y).abs() + 80);
      await add(
        RectangleComponent(
          position: centre,
          size: dim,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF232038),
        ),
      );
    }

    await couloir(salles[0].centre, salles[1].centre);
    await couloir(salles[1].centre, salles[2].centre);
    await couloir(salles[0].centre, salles[3].centre);
    await couloir(salles[3].centre, salles[4].centre);
    await couloir(salles[4].centre, salles[5].centre);
    await couloir(salles[2].centre, salles[5].centre);

    // Les pièges : des losanges rouges.
    for (final p in pieges) {
      await add(Piege()..position = p.clone());
    }

    // Le liseré des murs extérieurs.
    await add(
      RectangleComponent(
        position: Vector2.zero(),
        size: taille,
        paint: Paint()
          ..color = const Color(0xFF6F67B8)
          ..style = PaintingStyle.stroke
          ..strokeWidth = 10,
      ),
    );
  }
}

class Piege extends RectangleComponent {
  Piege()
      : super(
          size: Vector2.all(46),
          anchor: Anchor.center,
          angle: 0.7854, // pi/4 : un carré tourné = un losange
          paint: Paint()..color = const Color(0xFFD5453E),
        );
}

// ---------------------------------------------------------------- LE HÉROS

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 18,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();
  static const double vitesse = 260;

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) {
      position += direction.normalized() * vitesse * dt;
    }
    position.clamp(Vector2.all(30), MondeDonjon.taille - Vector2.all(30));
  }
}

// ------------------------------------------------------------------- LE HUD

class BarreDeVie extends PositionComponent {
  BarreDeVie() : super(position: Vector2(14, 14), size: Vector2(240, 20));

  double ratio = 1;

  static final Paint _fond = Paint()..color = const Color(0xFF35304E);
  static final Paint _vie = Paint()..color = const Color(0xFF56C271);
  static final Paint _bord = Paint()
    ..color = const Color(0xFFE9E7F5)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2;

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _fond);
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.x * ratio.clamp(0, 1), size.y),
      _vie,
    );
    canvas.drawRect(size.toRect(), _bord);
  }
}

// -------------------------------------------------------------------- LE JEU

class DonjonDeDart extends FlameGame<MondeDonjon> with KeyboardEvents {
  DonjonDeDart._(MondeDonjon monde)
      : super(world: monde, camera: CameraComponent(world: monde));

  factory DonjonDeDart() => DonjonDeDart._(MondeDonjon());

  final Heros heros = Heros();
  final BarreDeVie barre = BarreDeVie();
  late final TextComponent _info;

  CameraComponent? minimap;
  double vies = 1.0;
  double _invincible = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    heros.position = Vector2(400, 300);
    await world.add(heros);

    // Caméra principale : suivi lissé, bornes, ancre légèrement haute.
    camera.viewfinder.anchor = const Anchor(0.5, 0.45);
    camera.follow(heros, maxSpeed: 320, snap: true);
    camera.setBounds(
      Rectangle.fromLTRB(0, 0, MondeDonjon.taille.x, MondeDonjon.taille.y),
      considerViewport: true,
    );

    // HUD : dans le viewport.
    _info = TextComponent(
      text: '',
      position: Vector2(14, 42),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFE9E7F5)),
      ),
    );
    await camera.viewport.addAll([barre, _info]);

    // Minimap : seconde caméra sur le même monde.
    final m = CameraComponent(
      world: world,
      viewport: FixedSizeViewport(220, 150),
    )..priority = 10;
    m.viewfinder
      ..anchor = Anchor.center
      ..position = MondeDonjon.taille / 2
      ..zoom = 0.09;
    minimap = m;
    await add(m);
    _placerMinimap();
  }

  void _placerMinimap() {
    final m = minimap;
    if (m == null) return;
    m.viewport.anchor = Anchor.topRight;
    m.viewport.position = Vector2(size.x - 14, 14);
  }

  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    _placerMinimap();
  }

  void secouer() {
    camera.viewfinder.add(
      MoveEffect.by(
        Vector2(12, 5),
        EffectController(duration: 0.045, alternate: true, repeatCount: 6),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (_invincible > 0) _invincible -= dt;

    // Collision simple avec les pièges (distance au centre).
    if (_invincible <= 0) {
      for (final p in world.children.whereType<Piege>()) {
        if (p.position.distanceTo(heros.position) < 34) {
          vies = (vies - 0.2).clamp(0, 1);
          _invincible = 1.2;
          secouer();
          break;
        }
      }
    }

    barre.ratio = vies;
    final p = heros.position;
    _info.text = 'Donjon de Dart   —   héros '
        '(${p.x.toStringAsFixed(0)}, ${p.y.toStringAsFixed(0)})   '
        'vie ${(vies * 100).toStringAsFixed(0)} %';
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    heros.direction
      ..x = 0
      ..y = 0;
    if (keysPressed.contains(LogicalKeyboardKey.keyQ) ||
        keysPressed.contains(LogicalKeyboardKey.arrowLeft)) {
      heros.direction.x -= 1;
    }
    if (keysPressed.contains(LogicalKeyboardKey.keyD) ||
        keysPressed.contains(LogicalKeyboardKey.arrowRight)) {
      heros.direction.x += 1;
    }
    if (keysPressed.contains(LogicalKeyboardKey.keyZ) ||
        keysPressed.contains(LogicalKeyboardKey.arrowUp)) {
      heros.direction.y -= 1;
    }
    if (keysPressed.contains(LogicalKeyboardKey.keyS) ||
        keysPressed.contains(LogicalKeyboardKey.arrowDown)) {
      heros.direction.y += 1;
    }
    return KeyEventResult.handled;
  }
}
```

**Résultat :**

```text
Donjon de Dart   —   héros (400, 300)   vie 100 %
Donjon de Dart   —   héros (1204, 302)  vie 80 %      ← piège touché, l'écran vibre
Donjon de Dart   —   héros (1998, 401)  vie 60 %
```

Le donjon défile, le héros reste presque au centre, la vue s'arrête net au liseré violet des murs, la barre de vie et le texte ne bougent jamais, et la minimap en haut à droite montre les six salles avec le point doré du héros.

| Élément du cahier des charges | Section du chapitre |
| --- | --- |
| Monde plus grand que l'écran | 31.4, 31.5 |
| Caméra qui suit avec lissage | 31.12, 31.13 |
| Bornes aux murs | 31.16 |
| HUD fixe | 31.19, 31.20 |
| Minimap | 31.24 |
| Tremblement | 31.23 |

---

## 31.29 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `camera.follow()` ne fait rien | l'entité a été ajoutée avec `add()` au jeu, pas avec `world.add()` | `await world.add(heros);` |
| Le HUD glisse hors de l'écran | le HUD a été ajouté au monde | `camera.viewport.add(barre);` |
| `The method 'followComponent' isn't defined` | API supprimée depuis la refonte de la caméra | `camera.follow(cible)` |
| `The setter 'zoom' isn't defined for 'CameraComponent'` | `zoom` appartient au viseur | `camera.viewfinder.zoom = 2;` |
| `The method 'snapTo' isn't defined` | méthode de l'ancienne classe `Camera` | `camera.moveTo(p)` ou `follow(c, snap: true)` |
| `The setter 'worldBounds' isn't defined` | API supprimée | `camera.setBounds(Rectangle.fromLTRB(...))` |
| `class MonJeu extends BaseGame` ne compile pas | `BaseGame` a disparu en Flame 1.0 | `extends FlameGame` |
| Un clic pose l'objet au mauvais endroit | coordonnée d'écran utilisée comme coordonnée de monde | `camera.globalToLocal(event.canvasPosition)` |
| Le pixel art est flou et scintille | zoom fractionnaire et filtrage par défaut | zoom entier + `FilterQuality.none` |
| La caméra revient toute seule après un repositionnement | un `follow()` est toujours actif | `camera.stop()` avant de repositionner |
| Bandes noires en bord de niveau | bornes absentes ou `considerViewport: false` | `setBounds(..., considerViewport: true)` |
| Le décor bouge, le héros non (ou l'inverse) | mélange de `add()` et `world.add()` | tout ce qui a une position de monde va dans `world` |
| Le score disparaît après un changement de viewport | remplacer `camera.viewport` détache ses enfants | rattacher le HUD au nouveau viewport |
| La caméra ne rattrape jamais le héros | `maxSpeed` inférieur à la vitesse du héros | `maxSpeed` supérieur d'au moins 10 % |
| La minimap s'affiche dans la minimap | la minimap a été ajoutée au monde | l'ajouter au jeu, avec son propre viewport |
| Le décor est figé alors que la caméra bouge | deux instances de `World` : une mise à jour, une observée | une seule instance, passée aux deux endroits |
| L'écran reste décalé après un tremblement | `repeatCount` impair avec `alternate: true` | nombre pair d'allers-retours |

---

## 31.30 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `World` | conteneur des entités ; ne se dessine pas lui-même ; `priority` très négative |
| `CameraComponent` | assemble un viseur et un viewport, et dessine un monde |
| `Viewfinder` | où l'on regarde : `position`, `zoom`, `angle`, `anchor` |
| `Viewport` | par quel trou on regarde ; porte le HUD ; découpe le rendu |
| `backdrop` | composants fixes dessinés **derrière** le monde |
| `world.add()` | tout ce qui a une position dans le donjon |
| `camera.viewport.add()` | tout ce qui est collé à l'écran |
| `CameraComponent.withFixedResolution()` | garantit le nombre d'unités visibles, avec bandes noires |
| `camera.follow(cible)` | suivi ; `maxSpeed` pour lisser, `snap` pour démarrer sur la cible |
| `camera.moveTo(p, speed:)` | déplacement scripté ; annule le suivi |
| `camera.stop()` | fige la caméra sur place |
| `camera.setBounds(shape)` | limite le regard ; `considerViewport: true` évite les bandes noires |
| `Rectangle.fromLTRB` | forme de bornes, importée de `package:flame/experimental.dart` |
| `camera.globalToLocal()` | écran → monde |
| `camera.localToGlobal()` | monde → écran |
| `visibleWorldRect` / `canSee()` | outils de culling ; le culling n'est **pas** automatique |
| Plusieurs caméras | minimap, écran partagé ; chaque caméra redessine le monde |
| Tremblement | `MoveEffect` relatif posé sur le `viewfinder` |
| Pixel art | zoom entier, résolution logique fixe, `FilterQuality.none` |
| API périmées | `BaseGame`, `followComponent`, `camera.zoom`, `snapTo`, `worldBounds` |

---

## 31.31 — Exercices

### Exercice 1 — Le monde et l'écran (facile)
Affichez un carré vert dans le monde et un carré rouge dans le jeu. Faites glisser la caméra vers la droite et vérifiez lequel des deux bouge.

### Exercice 2 — Point de vue fixe (facile)
Construisez un damier de 10 × 10 cases de 64 unités. Réglez le viseur pour que le centre du damier soit au centre de l'écran, avec un zoom de 1,5.

### Exercice 3 — Résolution fixe (facile)
Écrivez un jeu en `withFixedResolution(width: 320, height: 180)` qui place un carré doré exactement à chaque coin de la zone visible. Vérifiez que le redimensionnement ne change rien.

### Exercice 4 — Suivi simple (moyen)
Un héros traverse le donjon de gauche à droite tout seul. Faites-le suivre par la caméra, avec un `maxSpeed` de 200 et un `snap` au démarrage.

### Exercice 5 — Ancre de plateforme (moyen)
Reprenez l'exercice 4 et placez l'ancre du viseur à `Anchor(0.3, 0.5)`. Ajoutez un texte de HUD indiquant l'ancre utilisée.

### Exercice 6 — Bornes de niveau (moyen)
Un monde de 1600 × 900 avec un liseré. Le héros se déplace aux flèches. Bornez la caméra de façon qu'aucune zone noire n'apparaisse jamais.

### Exercice 7 — Clic vers le monde (moyen)
Chaque clic doit poser un disque à l'endroit cliqué **dans le monde**, alors que la caméra défile en permanence et que le zoom vaut 2.

### Exercice 8 — Indicateur de sortie (difficile)
Une sortie est placée en `(1500, 1000)`. Affichez dans le HUD une pastille plaquée au bord de l'écran indiquant sa direction, en utilisant `localToGlobal`.

### Exercice 9 — Minimap ronde (difficile)
Ajoutez à un donjon une seconde caméra en `CircularViewport(80)`, placée en bas à droite, avec un zoom de 0,1.

### Exercice 10 — Secousse au contact (difficile)
Un héros contrôlé aux flèches, un piège au centre. Quand il entre dans le piège, l'écran tremble et un compteur de dégâts du HUD s'incrémente, une fois par entrée seulement.

---

## 31.32 — Corrections des exercices

Tous les programmes ci-dessous sont complets et s'exécutent tels quels, sans aucun fichier image.

### Correction 1

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex1()));

class Ex1 extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(RectangleComponent(
      position: Vector2(120, 100),
      size: Vector2.all(60),
      paint: Paint()..color = const Color(0xFF4CAF50),
    ));
    await add(RectangleComponent(
      position: Vector2(120, 200),
      size: Vector2.all(60),
      paint: Paint()..color = const Color(0xFFE53935),
    ));
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(_t * 50, 0);
  }
}
```

**Explication :** le carré vert appartient au `world` : la transformation de la caméra lui est appliquée, il défile donc vers la gauche à 50 unités par seconde. Le carré rouge est un enfant direct du jeu ; il est dessiné après le travail de la caméra, dans le repère de l'écran, et reste immobile. C'est la démonstration littérale de la règle de la section 31.5.

---

### Correction 2

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex2()));

class Ex2 extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    for (var i = 0; i < 100; i++) {
      final x = i % 10;
      final y = i ~/ 10;
      await world.add(RectangleComponent(
        position: Vector2(x * 64.0, y * 64.0),
        size: Vector2.all(60),
        paint: Paint()
          ..color = (x + y).isEven
              ? const Color(0xFF2E2B4A)
              : const Color(0xFF484075),
      ));
    }
    // Le damier occupe de 0 à 640 : son centre est en (320, 320).
    camera.viewfinder
      ..anchor = Anchor.center
      ..position = Vector2(320, 320)
      ..zoom = 1.5;
    debugPrint('zone visible : ${camera.visibleWorldRect}');
  }
}
```

**Explication :** `Anchor.center` signifie que `viewfinder.position` est dessinée au centre du viewport ; il suffit d'y mettre le centre géométrique du damier. Le zoom de 1,5 réduit la zone visible d'un tiers : sur 800 pixels de large, on ne voit plus que 533 unités de monde.

---

### Correction 3

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex3()));

class Ex3 extends FlameGame {
  Ex3()
      : super(
          camera:
              CameraComponent.withFixedResolution(width: 320, height: 180),
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder
      ..anchor = Anchor.topLeft
      ..position = Vector2.zero();
    await world.add(RectangleComponent(
      size: Vector2(320, 180),
      paint: Paint()..color = const Color(0xFF2E2B4A),
    ));
    for (final coin in [
      Vector2(0, 0),
      Vector2(310, 0),
      Vector2(0, 170),
      Vector2(310, 170),
    ]) {
      await world.add(RectangleComponent(
        position: coin,
        size: Vector2.all(10),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ));
    }
    debugPrint('visible : ${camera.visibleWorldRect}');
  }
}
```

**Explication :** `withFixedResolution` installe un `FixedResolutionViewport` de 320 × 180 et règle le viseur pour que cette zone remplisse le viewport. Quelle que soit la fenêtre, `visibleWorldRect` vaut toujours `Rect.fromLTRB(0, 0, 320, 180)` : les carrés dorés restent collés aux coins, et des bandes noires apparaissent si le rapport n'est pas 16/9.

---

### Correction 4

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex4()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 18,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  @override
  void update(double dt) {
    super.update(dt);
    position.x += 180 * dt;
    if (position.x > 2300) position.x = 100;
  }
}

class Ex4 extends FlameGame {
  final Heros heros = Heros();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(2400, 800),
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    for (var x = 0.0; x < 2400; x += 120) {
      await world.add(RectangleComponent(
        position: Vector2(x, 460),
        size: Vector2(100, 18),
        paint: Paint()..color = const Color(0xFF4A4770),
      ));
    }
    heros.position = Vector2(100, 400);
    await world.add(heros);
    camera.follow(heros, maxSpeed: 200, snap: true);
  }
}
```

**Explication :** le héros avance à 180 unités par seconde, la caméra est autorisée à en faire 200 : elle le rattrape toujours, avec un léger retard qui donne une impression de poids. `snap: true` évite le long travelling initial depuis `(0, 0)`. Avec `maxSpeed: 150`, le héros aurait distancé la caméra et serait sorti de l'écran.

---

### Correction 5

Le programme est celui de la correction 4, avec deux ajouts dans `onLoad` :

```dart
    camera.viewfinder.anchor = const Anchor(0.3, 0.5);
    camera.follow(heros, maxSpeed: 220, snap: true);

    await camera.viewport.add(TextComponent(
      text: 'anchor = Anchor(0.3, 0.5) : on voit loin devant',
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    ));
```

**Explication :** l'ancre déplace le point de l'écran sur lequel le regard est posé. À 30 % de la largeur, le héros n'est plus au centre : 70 % de l'écran montre ce qui l'attend. Le texte est ajouté au **viewport**, donc il reste en haut à gauche même quand le décor défile ; ajouté au monde, il serait parti avec le décor.

---

### Correction 6

```dart
import 'package:flame/components.dart';
import 'package:flame/experimental.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: Ex6()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 16,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) position += direction.normalized() * 250 * dt;
    position.clamp(Vector2.all(20), Ex6.monde - Vector2.all(20));
  }
}

class Ex6 extends FlameGame with KeyboardEvents {
  static final Vector2 monde = Vector2(1600, 900);
  final Heros heros = Heros();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: monde,
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    await world.add(RectangleComponent(
      size: monde,
      paint: Paint()
        ..color = const Color(0xFF8A83C9)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 8,
    ));
    heros.position = monde / 2;
    await world.add(heros);
    camera.follow(heros, maxSpeed: 350, snap: true);
    camera.setBounds(
      Rectangle.fromLTRB(0, 0, monde.x, monde.y),
      considerViewport: true,
    );
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    heros.direction
      ..x = (touches.contains(LogicalKeyboardKey.arrowRight) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowLeft) ? 1.0 : 0.0)
      ..y = (touches.contains(LogicalKeyboardKey.arrowDown) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowUp) ? 1.0 : 0.0);
    return KeyEventResult.handled;
  }
}
```

**Explication :** `Rectangle` vient de `package:flame/experimental.dart`, pas de `dart:ui`. Le paramètre `considerViewport: true` garantit l'absence de zone noire : sans lui, seule la position du regard serait limitée, et la moitié de l'écran pourrait sortir du monde quand le héros longe un mur.

---

### Correction 7

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex7()));

class Capteur extends PositionComponent
    with TapCallbacks, HasGameReference<Ex7> {
  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    size = taille;
  }

  @override
  void onTapDown(TapDownEvent event) {
    final p = game.camera.globalToLocal(event.canvasPosition);
    game.world.add(CircleComponent(
      radius: 8,
      position: p,
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFF66DD88),
    ));
    debugPrint('écran ${event.canvasPosition} → monde $p');
  }
}

class Ex7 extends FlameGame {
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    for (var i = 0; i < 360; i++) {
      final x = i % 30;
      final y = i ~/ 30;
      await world.add(RectangleComponent(
        position: Vector2(x * 100.0, y * 100.0),
        size: Vector2.all(96),
        paint: Paint()
          ..color = (x + y).isEven
              ? const Color(0xFF23223A)
              : const Color(0xFF2C2A47),
      ));
    }
    camera.viewfinder
      ..anchor = Anchor.center
      ..zoom = 2;
    await camera.viewport.add(Capteur());
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(400 + _t * 60, 400);
  }
}
```

**Explication :** le capteur est un composant du **viewport** dimensionné à l'écran ; il reçoit donc tous les taps. `event.canvasPosition` est en pixels d'écran : sans `globalToLocal`, le disque serait posé à cette coordonnée interprétée comme une position de monde, donc décalée de plusieurs centaines d'unités et divisée par le zoom de 2.

---

### Correction 8

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex8()));

class Pastille extends CircleComponent with HasGameReference<Ex8> {
  Pastille()
      : super(
          radius: 9,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF56C271),
        );

  @override
  void update(double dt) {
    super.update(dt);
    final aLEcran = game.camera.localToGlobal(Ex8.sortie);
    final t = game.camera.viewport.size;
    position = Vector2(
      aLEcran.x.clamp(16.0, t.x - 16.0),
      aLEcran.y.clamp(16.0, t.y - 16.0),
    );
  }
}

class Ex8 extends FlameGame {
  static final Vector2 sortie = Vector2(1500, 1000);
  double _t = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(2000, 1400),
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    await world.add(RectangleComponent(
      position: sortie,
      size: Vector2.all(50),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFF56C271),
    ));
    camera.viewfinder.anchor = Anchor.center;
    await camera.viewport.add(Pastille());
  }

  @override
  void update(double dt) {
    super.update(dt);
    _t += dt;
    camera.viewfinder.position = Vector2(300 + _t * 100, 400 + _t * 60);
  }
}
```

**Explication :** `localToGlobal` transforme la position de monde de la sortie en pixels d'écran ; le `clamp` plaque ce point sur le bord du viewport quand il est hors champ. Comme la pastille vit dans le viewport, ses coordonnées sont déjà des coordonnées d'écran : aucune conversion supplémentaire n'est nécessaire.

---

### Correction 9

```dart
import 'package:flame/camera.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: Ex9()));

class Ex9 extends FlameGame {
  static final Vector2 monde = Vector2(2000, 1400);
  CameraComponent? mini;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: monde,
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    final borne = Paint()..color = const Color(0xFF453F70);
    for (var i = 0; i < 70; i++) {
      await world.add(RectangleComponent(
        position: Vector2(100.0 + (i % 10) * 200, 100.0 + (i ~/ 10) * 200),
        size: Vector2.all(70),
        anchor: Anchor.center,
        paint: borne,
      ));
    }
    camera.viewfinder
      ..anchor = Anchor.center
      ..position = monde / 2;

    final m = CameraComponent(world: world, viewport: CircularViewport(80))
      ..priority = 10;
    m.viewfinder
      ..anchor = Anchor.center
      ..position = monde / 2
      ..zoom = 0.1;
    mini = m;
    await add(m);
    _placer();
  }

  void _placer() {
    final m = mini;
    if (m == null) return;
    m.viewport.anchor = Anchor.bottomRight;
    m.viewport.position = Vector2(size.x - 16, size.y - 16);
  }

  @override
  void onGameResize(Vector2 t) {
    super.onGameResize(t);
    _placer();
  }
}
```

**Explication :** la seconde caméra reçoit **la même instance** de `world` que la principale : c'est ce qui permet d'observer le même donjon deux fois. Son `CircularViewport(80)` découpe le rendu en disque, et le zoom de 0,1 fait tenir les 2000 unités du monde dedans. La `priority` de 10 la fait dessiner par-dessus la caméra principale. Le placement est refait dans `onGameResize`, car un viewport de taille fixe ne se replace pas tout seul.

---

### Correction 10

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: Ex10()));

class Heros extends CircleComponent {
  Heros()
      : super(
          radius: 16,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 direction = Vector2.zero();

  @override
  void update(double dt) {
    super.update(dt);
    if (!direction.isZero()) position += direction.normalized() * 240 * dt;
  }
}

class Ex10 extends FlameGame with KeyboardEvents {
  static final Vector2 piege = Vector2(600, 400);
  final Heros heros = Heros();
  late final TextComponent _info;
  int degats = 0;
  bool _dedans = false;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await world.add(RectangleComponent(
      size: Vector2(1200, 800),
      paint: Paint()..color = const Color(0xFF1B1A26),
    ));
    await world.add(RectangleComponent(
      position: piege,
      size: Vector2.all(60),
      anchor: Anchor.center,
      angle: 0.7854, // pi/4 : un carré tourné devient un losange
      paint: Paint()..color = const Color(0xFFD5453E),
    ));
    heros.position = Vector2(200, 200);
    await world.add(heros);
    camera.follow(heros, maxSpeed: 320, snap: true);

    _info = TextComponent(
      text: 'dégâts : 0',
      position: Vector2.all(12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_info);
  }

  @override
  void update(double dt) {
    super.update(dt);
    final proche = heros.position.distanceTo(piege) < 40;
    if (proche && !_dedans) {
      degats += 10;
      _info.text = 'dégâts : $degats';
      camera.viewfinder.add(MoveEffect.by(
        Vector2(12, 5),
        EffectController(duration: 0.045, alternate: true, repeatCount: 6),
      ));
    }
    _dedans = proche; // mémorise l'état pour ne compter qu'une entrée
  }

  @override
  KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> touches) {
    heros.direction
      ..x = (touches.contains(LogicalKeyboardKey.arrowRight) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowLeft) ? 1.0 : 0.0)
      ..y = (touches.contains(LogicalKeyboardKey.arrowDown) ? 1.0 : 0.0) -
          (touches.contains(LogicalKeyboardKey.arrowUp) ? 1.0 : 0.0);
    return KeyEventResult.handled;
  }
}
```

**Explication :** le booléen `_dedans` mémorise l'état de la frame précédente ; sans lui, le compteur augmenterait soixante fois par seconde tant que le héros reste sur le piège. Le tremblement est un `MoveEffect` **relatif** posé sur le viseur : il coexiste avec le `FollowBehavior` installé par `follow()`, et le nombre pair d'allers-retours garantit le retour exact à la position d'origine. La détection par distance sera remplacée au chapitre 32 par de vraies hitbox et `onCollisionStart`.

## Et maintenant ?

Votre donjon est enfin plus grand que l'écran. La caméra suit le héros sans le lâcher ni donner la nausée, elle s'arrête aux murs, le HUD reste vissé à l'écran et une minimap montre où l'on en est. Vous savez aussi reconnaître, en une seconde, un tutoriel écrit pour une version de Flame disparue.

Il manque pourtant l'essentiel d'un jeu : rien ne se touche. Le héros traverse les murs, les pièges se détectent avec un calcul de distance approximatif, et les gobelins ne mordent pas. Le chapitre suivant remplace tout cela par le vrai système de collision de Flame : le mixin `HasCollisionDetection`, les hitbox `RectangleHitbox` et `CircleHitbox`, les rappels `onCollisionStart`, `onCollision` et `onCollisionEnd`, et les trois valeurs de `CollisionType`.

Rendez-vous au chapitre 32 : [32-PARTIE-2B—COLLISIONS-ET-COLLISIONCALLBACKS.md](./32-PARTIE-2B—COLLISIONS-ET-COLLISIONCALLBACKS.md)
