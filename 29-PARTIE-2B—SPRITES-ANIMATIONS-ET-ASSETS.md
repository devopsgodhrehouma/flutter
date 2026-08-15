# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 29 — SPRITES, ANIMATIONS ET ASSETS AVEC FLAME

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitre 22 (sprites et sprite sheets écrits à la main), chapitre 27 (installer Flame, premier `FlameGame`), chapitre 28 (composants et cycle de vie), et, côté Dart, les chapitres 11 (enums), 12 (null safety) et 15 (asynchrone).
> **Version de Flame utilisée :** **1.38.0** (publiée le 19 juillet 2026). Toutes les signatures de ce chapitre proviennent de cette version.
> **Ce que vous saurez faire à la fin :** afficher, animer et piloter des sprites avec Flame, organiser un dossier d'assets propre, et faire tourner l'intégralité du chapitre **sans posséder le moindre fichier image**.

---

## 29.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- énumérer précisément ce que Flame automatise par rapport au code écrit à la main au chapitre 22 ;
- placer vos images dans `assets/images/`, le dossier attendu par convention ;
- déclarer un dossier d'assets dans `pubspec.yaml` avec la bonne indentation ;
- expliquer le rôle du cache `Flame.images` et de son préfixe ;
- charger une image avec `images.load()` et plusieurs avec `images.loadAll()` ;
- précharger toutes vos ressources dans `onLoad()` avant que le jeu ne démarre ;
- construire un `Sprite` à partir d'une image déjà chargée ;
- utiliser le raccourci `Sprite.load()` et savoir quand ne pas l'utiliser ;
- découper une planche avec `srcPosition` et `srcSize` ;
- afficher un sprite à l'écran avec un `SpriteComponent` ;
- utiliser `SpriteComponent.fromImage()` ;
- maîtriser la taille, l'ancre et la position d'un sprite ;
- retourner un sprite avec `flipHorizontally()` et `flipHorizontallyAroundCenter()` ;
- construire une `SpriteAnimation` ;
- décrire une séquence de frames avec `SpriteAnimationData.sequenced()` ;
- régler la cadence avec `stepTime` et le bouclage avec `loop` ;
- afficher une animation avec un `SpriteAnimationComponent` ;
- changer d'animation à l'exécution ;
- gérer plusieurs animations d'un même personnage avec `SpriteAnimationGroupComponent` et un enum ;
- manipuler la table `animations` et l'état `current` ;
- réagir à la fin d'une animation avec `onComplete` ;
- exploiter `SpriteSheet` et `createAnimation()` ;
- garder du pixel art parfaitement net avec `FilterQuality.none` et un zoom entier ;
- trouver des assets libres, lire une licence et créditer correctement ;
- organiser, nommer et regrouper vos fichiers d'assets ;
- **générer une sprite sheet complète en mémoire**, sans aucun fichier, et l'injecter dans le cache de Flame ;
- construire un cadre d'interface extensible avec `NineTileBoxComponent` ;
- installer un fond en parallaxe avec `ParallaxComponent` ;
- écrire un écran de chargement adossé à un préchargement global ;
- assembler le joueur du « Donjon de Dart » avec ses animations *idle*, *marche* et *attaque*, en 100 % code.

---

## Avertissement : ce cours ne fournit toujours aucune image

Vous connaissez déjà la règle depuis le chapitre 22 : cette formation est faite de fichiers texte. Elle ne peut pas vous livrer de PNG.

Le chapitre suit donc **deux voies en parallèle**, exactement comme le chapitre 22 :

```text
  ┌──────────────────────────────┐      ┌──────────────────────────────┐
  │  VOIE 1 — le cas normal      │      │  VOIE 2 — la solution de     │
  │  Un vrai fichier PNG posé    │      │  repli, 100 % code           │
  │  dans assets/images/         │      │  (aucun téléchargement)      │
  └──────────────┬───────────────┘      └──────────────┬───────────────┘
                 │                                      │
                 │ Flame.images.load('heros.png')       │ PictureRecorder + Canvas
                 │                                      │ puis Flame.images.add(...)
                 ▼                                      ▼
        ┌──────────────────────────────────────────────────────────┐
        │   Un ui.Image dans le cache de Flame, sous la clé        │
        │   'heros.png'  —  la SUITE DU CODE EST IDENTIQUE.        │
        └──────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                Sprite, SpriteAnimation, SpriteComponent, SpriteSheet...
```

Le point capital n'a pas changé : une image fabriquée en mémoire est **indiscernable** d'une image décodée depuis un PNG. C'est le même type `ui.Image`. Toutes les API de Flame l'acceptent sans broncher.

Vous pourrez donc suivre l'intégralité du chapitre, exécuter chaque exemple, faire les dix exercices, et remplacer plus tard une seule ligne de code le jour où vous aurez de vrais fichiers.

---

## 29.1 — Ce que Flame automatise par rapport au chapitre 22

Au chapitre 22, vous avez tout écrit à la main : le chargement, le cache, le calcul des rectangles source, la classe d'animation, l'accumulateur de temps, la machine à états. C'était le bon exercice : vous savez désormais ce qui se passe sous le capot.

Flame refait exactement ces choses, en mieux testé et en plus court. Voici la correspondance ligne par ligne.

| Tâche | Chapitre 22 — à la main | Flame 1.38.0 |
| --- | --- | --- |
| Charger une image | `rootBundle.load()` puis `decodeImageFromList()` | `Flame.images.load('heros.png')` |
| Préfixer le chemin | concaténation `'assets/images/' + nom` écrite à la main | préfixe `'assets/images/'` intégré au cache |
| Mémoriser les images chargées | une `Map<String, ui.Image>` maison | la classe `Images`, `load()` met en cache automatiquement |
| Relire une image déjà chargée | lecture dans la `Map` avec un `?` partout | `Flame.images.fromCache('heros.png')` (synchrone) |
| Découper une frame | `Rect.fromLTWH(col * 32, ligne * 32, 32, 32)` calculé à la main | `Sprite(image, srcPosition: ..., srcSize: ...)` |
| Dessiner une frame | `canvas.drawImageRect(src, dst, paint)` | `sprite.render(canvas, position: ..., size: ...)` |
| Afficher un sprite dans le jeu | dessin manuel dans `paint()` | `SpriteComponent` ajouté à l'arbre |
| Décrire une animation | classe maison avec une `List<Rect>` | `SpriteAnimation` + `SpriteAnimationData.sequenced()` |
| Faire avancer l'animation | accumulateur `_temps += dt` et calcul d'index | `SpriteAnimationTicker`, piloté automatiquement par le composant |
| Régler la cadence | champ `stepTime` maison | paramètre `stepTime` de `SpriteAnimationData` |
| Boucler ou jouer une seule fois | `if (index >= frames.length) index = frames.length - 1;` | paramètre `loop: true` / `loop: false` |
| Savoir qu'une animation est finie | booléen `fini` mis à jour à la main | `animationTicker.onComplete`, `done()`, `completed` |
| Plusieurs animations par personnage | `Map<EtatAnim, AnimationMaison>` + `switch` | `SpriteAnimationGroupComponent<T>` avec `animations` et `current` |
| Retourner un sprite | `canvas.save()`, `scale(-1, 1)`, `translate(...)`, `restore()` | `flipHorizontallyAroundCenter()` |
| Ordre de dessin | ordre des appels dans `paint()` | champ `priority` du composant |
| Planche → animation d'une ligne | boucle de calcul index → (ligne, colonne) | `SpriteSheet.createAnimation(row: 1, stepTime: 0.1)` |
| Fond en parallaxe | plusieurs boucles de `drawImageRect` | `ParallaxComponent` |
| Cadre d'interface extensible | neuf `drawImageRect` positionnés à la main | `NineTileBoxComponent` |
| Pixel art net | `Paint()..filterQuality = FilterQuality.none` | même chose, via le `paint` du composant |

> **À retenir.** Flame ne vous apprend rien de nouveau sur le plan des concepts. Il supprime le code répétitif et les occasions de se tromper d'un pixel. Tout ce que vous avez compris au chapitre 22 reste valable : c'est le vocabulaire qui change.

Une dernière colonne mérite d'être ajoutée : **ce que Flame ne fait pas à votre place**.

| Ce qui reste entièrement de votre responsabilité |
| --- |
| Déclarer les assets dans `pubspec.yaml`. Flame ne peut pas deviner vos fichiers. |
| Précharger au bon moment. Un sprite utilisé avant chargement reste un bug. |
| Choisir la taille de vos frames et la garder constante dans une planche. |
| Décider de la cadence : un `stepTime` mal réglé donne un personnage qui glisse ou qui tressaute. |
| Respecter les licences des images que vous téléchargez. |

---

## 29.2 — Le dossier `assets/images/` : la convention Flame

Flame ne cherche pas vos images n'importe où. La classe `Images` possède un **préfixe** de chemin, dont la valeur par défaut est `'assets/images/'`.

Voici l'extrait exact du code source de Flame 1.38.0 :

```dart
Images({
  this._prefix = 'assets/images/',
  AssetBundle? bundle,
}) : bundle = bundle ?? Flame.bundle;
```

La documentation du champ est sans ambiguïté :

> *« The prefix is **not** part of the keys of the images stored in this cache. For example, if you load image `player.png`, then it will be searched at location `prefix + "player.png"` but stored in the cache under the key `"player.png"`. »*

Traduisons cette phrase, parce qu'elle contient à elle seule l'erreur numéro un des débutants.

```text
  Vous écrivez :          Flame.images.load('heros.png')
                                   │
                                   ├─ fichier réellement lu :
                                   │     assets/images/heros.png
                                   │
                                   └─ clé dans le cache :
                                         'heros.png'      (SANS le préfixe)
```

Donc :

- vous passez **toujours** un chemin **relatif au dossier `assets/images/`** ;
- vous ne passez **jamais** `'assets/images/heros.png'`, sous peine de faire chercher à Flame le fichier `assets/images/assets/images/heros.png`, qui n'existe pas.

Arborescence type d'un projet « Donjon de Dart » :

```text
  donjon_de_dart/
  ├── pubspec.yaml
  ├── lib/
  │   ├── main.dart
  │   └── composants/
  │       ├── heros.dart
  │       └── gobelin.dart
  └── assets/
      ├── images/            <-- préfixe par défaut de Flame.images
      │   ├── heros.png
      │   ├── gobelin.png
      │   ├── potion.png
      │   └── decor/
      │       ├── mur.png
      │       └── sol.png
      ├── audio/             <-- préfixe par défaut de FlameAudio (chapitre 34)
      └── tiles/             <-- préfixe par défaut de flame_tiled (chapitre 34)
```

Avec cette arborescence :

```dart
Flame.images.load('heros.png');        // assets/images/heros.png
Flame.images.load('decor/mur.png');    // assets/images/decor/mur.png
```

Un sous-dossier fait donc partie de la clé. C'est normal et c'est même pratique pour ranger.

Le préfixe est modifiable, mais on y touche rarement :

```dart
Flame.images.prefix = 'assets/sprites/';   // doit se terminer par '/'
```

Le code source contient une assertion explicite :

```dart
assert(
  value.isEmpty || value.endsWith('/'),
  'Prefix must be empty or end with a "/"',
);
```

Un préfixe vide est autorisé : vous passez alors des chemins complets depuis la racine du projet. C'est déconseillé, car vous perdez la convention que tout le monde connaît.

> **Remarque.** Les préfixes des trois familles d'assets sont indépendants : `assets/images/` pour Flame, `assets/audio/` pour `flame_audio`, `assets/tiles/` pour `flame_tiled`. Ne mélangez pas les fichiers dans un seul dossier fourre-tout.

---

## 29.3 — Déclarer les assets dans `pubspec.yaml`

Poser un fichier dans `assets/images/` ne suffit pas. Flutter ne l'embarquera dans l'application que si vous l'avez **déclaré**. C'est le rôle de la section `flutter:` du `pubspec.yaml`, déjà croisée au chapitre 16 pour les paquets et au chapitre 22 pour les images.

Fichier complet et minimal pour ce chapitre :

```yaml
name: donjon_de_dart
description: Le jeu 2D de la formation, réalisé avec Flutter et Flame.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=3.12.0 <4.0.0"
  flutter: ">=3.44.0"

dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true

  assets:
    - assets/images/
```

Trois règles d'indentation, et elles font perdre des heures quand on les rate.

**Règle 1 — `assets:` est indenté de deux espaces sous `flutter:`.**

```yaml
flutter:
  assets:
    - assets/images/
```

**Règle 2 — chaque entrée est une ligne de liste, indentée de quatre espaces, commençant par un tiret et une espace.**

**Règle 3 — un chemin qui se termine par `/` déclare *tout le contenu direct* du dossier, mais PAS les sous-dossiers.**

C'est le piège classique. Reprenons l'arborescence de la section précédente :

```yaml
flutter:
  assets:
    - assets/images/          # heros.png, gobelin.png, potion.png : OUI
                              # decor/mur.png : NON
```

Il faut déclarer le sous-dossier explicitement :

```yaml
flutter:
  assets:
    - assets/images/
    - assets/images/decor/
```

Vous pouvez aussi déclarer les fichiers un par un. C'est plus verbeux, mais cela documente le contenu et évite d'embarquer des images de test oubliées :

```yaml
flutter:
  assets:
    - assets/images/heros.png
    - assets/images/gobelin.png
    - assets/images/potion.png
    - assets/images/decor/mur.png
```

Schéma de décision :

```text
  Vous ajoutez un fichier assets/images/boss.png
        │
        ├── Avez-vous déclaré « assets/images/ » (dossier entier) ?
        │       OUI  -> rien à faire, sauf pour un sous-dossier
        │       NON  -> ajoutez la ligne dans pubspec.yaml
        │
        └── Dans TOUS les cas : arrêtez l'application et relancez-la.
            Un « hot reload » ne recharge PAS le manifeste d'assets.
```

> **Erreur classique et son message.** Un asset non déclaré produit, au chargement, une exception du type :
>
> ```text
> Unable to load asset: "assets/images/heros.png"
> ```
>
> Neuf fois sur dix, la cause est l'une de ces trois-là : le fichier n'est pas déclaré, l'indentation du `pubspec.yaml` est fausse, ou l'application n'a pas été redémarrée à froid.

---

## 29.4 — `Flame.images` et le cache

`Flame.images` est une **instance globale** de la classe `Images`, fournie par Flame pour que vous n'ayez pas à en créer une.

```dart
import 'package:flame/flame.dart';

final image = await Flame.images.load('heros.png');
```

À quoi sert un cache ? À trois choses très concrètes.

**1. Ne décoder qu'une fois.** Décoder un PNG coûte du temps processeur et de la mémoire. Si trente gobelins utilisent `gobelin.png`, il serait absurde de décoder trente fois le même fichier. Le cache garantit qu'un seul `ui.Image` existe, partagé par tous.

**2. Rendre le second appel instantané.** Regardons l'implémentation réelle de `load` en 1.38.0 :

```dart
Future<Image> load(String fileName, {String? key, String? package}) {
  return (_assets[key ?? fileName] ??= _ImageAsset.future(
    _fetchToMemory(fileName, package: package),
  )).retrieveAsync();
}
```

L'opérateur `??=` (chapitre 12) est le cœur du mécanisme : si la clé existe déjà, on renvoie l'entrée existante ; sinon on lance le chargement et on l'enregistre. Un deuxième `load('heros.png')` ne touche jamais le disque.

**3. Permettre un accès synchrone plus tard.** Une fois l'image chargée, `fromCache` la rend immédiatement, sans `await` :

```dart
final image = Flame.images.fromCache('heros.png');   // ui.Image, pas Future
```

Voici l'API complète de la classe `Images` en 1.38.0.

| Membre | Signature | Rôle |
| --- | --- | --- |
| `prefix` | `String get prefix` / `set prefix` | dossier de recherche, `'assets/images/'` par défaut |
| `load` | `Future<Image> load(String fileName, {String? key, String? package})` | charge et met en cache |
| `loadAll` | `Future<List<Image>> loadAll(List<String> fileNames)` | charge plusieurs fichiers en parallèle |
| `loadAllImages` | `Future<List<Image>> loadAllImages()` | charge toutes les images trouvées sous le préfixe |
| `fromCache` | `Image fromCache(String name)` | lecture **synchrone** d'une image déjà chargée |
| `add` | `void add(String name, Image image)` | **injecte** une image déjà construite sous une clé |
| `fetchOrGenerate` | `Future<Image> fetchOrGenerate(String name, Future<Image> Function() imageGenerator)` | renvoie l'image du cache, ou la génère et la met en cache |
| `containsKey` | `bool containsKey(String key)` | la clé est-elle présente ? |
| `keys` | `List<String> get keys` | liste des clés du cache |
| `clear` | `void clear(String name)` | retire **et libère** une image |
| `clearCache` | `void clearCache()` | vide tout le cache |
| `ready` | `Future<void> ready()` | attend la fin de tous les chargements en cours |

Deux membres méritent une attention particulière, car ils sont la clé de la solution de repli de ce chapitre.

**`add(String name, Image image)`** insère dans le cache une image que vous avez fabriquée vous-même. Le commentaire du code source précise : *« The cache will assume the ownership of the image, and will properly dispose of it at the end »*. Autrement dit, Flame se charge de libérer la mémoire.

**`fetchOrGenerate(String name, Future<Image> Function() imageGenerator)`** est encore plus commode : si la clé est absente, il appelle votre générateur et enregistre le résultat ; sinon il renvoie ce qui est déjà là.

```dart
// Aucun fichier n'est lu. L'image est fabriquée par notre propre fonction.
await Flame.images.fetchOrGenerate('potion.png', creerImagePotion);

// À partir d'ici, 'potion.png' se comporte exactement comme un vrai fichier.
final image = Flame.images.fromCache('potion.png');
```

> **Attention à `clear` et `clearCache`.** Ces méthodes appellent `Image.dispose()`. Toute image encore utilisée par un `Sprite` devient inutilisable et le rendu lèvera une exception. On ne vide un cache qu'en changeant complètement de niveau ou d'écran, et après avoir retiré les composants concernés.

### Le cache du jeu plutôt que le cache global

`FlameGame` expose une propriété `images`. Par défaut, elle **pointe vers `Flame.images`**, le cache global. À l'intérieur d'un composant ou d'un jeu, écrivez simplement :

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    final image = await images.load('heros.png');   // équivalent à Flame.images
  }
}
```

Depuis un composant, la même chose passe par le mixin `HasGameReference` (chapitre 28) :

```dart
class Heros extends SpriteComponent with HasGameReference<DonjonDeDart> {
  @override
  Future<void> onLoad() async {
    final image = await game.images.load('heros.png');
    sprite = Sprite(image);
  }
}
```

---

## 29.5 — `images.load()` et `images.loadAll()`

### `load` : une image

```dart
Future<Image> load(String fileName, {String? key, String? package})
```

| Paramètre | Rôle |
| --- | --- |
| `fileName` | chemin relatif au préfixe, par exemple `'heros.png'` |
| `key` | clé de rangement dans le cache si vous voulez qu'elle diffère du nom de fichier |
| `package` | nom du paquet Dart lorsque l'image vient d'une dépendance et non de votre projet |

Le type de retour est `Future<Image>` : le chargement est **asynchrone**, comme au chapitre 15. Vous ne pouvez donc pas charger une image dans un constructeur.

```dart
// FAUX : un constructeur ne peut pas attendre.
class Heros extends SpriteComponent {
  Heros() {
    sprite = Sprite(Flame.images.load('heros.png')); // ne compile même pas
  }
}
```

```dart
// JUSTE : le chargement va dans onLoad, qui est asynchrone.
class Heros extends SpriteComponent {
  Heros() : super(size: Vector2.all(48), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    final image = await game.images.load('heros.png');
    sprite = Sprite(image);
  }
}
```

Le paramètre `key` sert quand deux fichiers portent le même nom dans deux sous-dossiers, ou quand vous voulez un nom logique plus court :

```dart
await Flame.images.load('personnages/heros_v2_final.png', key: 'heros');
final image = Flame.images.fromCache('heros');
```

### `loadAll` : plusieurs images

```dart
Future<List<Image>> loadAll(List<String> fileNames)
```

L'implémentation est un simple `Future.wait` sur `load`. Les chargements partent donc **en parallèle**, ce qui est nettement plus rapide qu'une boucle de `await`.

```dart
// LENT : les chargements s'enchaînent l'un après l'autre.
for (final nom in ['heros.png', 'gobelin.png', 'potion.png']) {
  await Flame.images.load(nom);
}

// RAPIDE : tout part en même temps, on attend la fin de l'ensemble.
await Flame.images.loadAll(['heros.png', 'gobelin.png', 'potion.png']);
```

`loadAll` renvoie la liste des images **dans l'ordre des noms fournis**. On l'ignore souvent, car on ira ensuite chercher chaque image par `fromCache`.

### `loadAllImages` : tout le dossier

```dart
Future<List<Image>> loadAllImages()
```

Cette méthode lit le manifeste des assets et charge tout ce qui, sous le préfixe, porte une extension d'image reconnue. Le motif exact, tiré du code source :

```dart
RegExp(
  r'\.(png|jpg|jpeg|svg|gif|webp|bmp|wbmp)$',
  caseSensitive: false,
)
```

C'est pratique pour un prototype. C'est dangereux pour un vrai jeu : vous chargerez en mémoire des images de menus, de crédits et de niveaux non joués. Réservez-la aux petits projets.

---

## 29.6 — Précharger dans `onLoad()`

Le chapitre 28 a établi la règle : `onLoad()` est le « constructeur asynchrone » d'un composant, appelé une seule fois, avant le premier `update`. C'est **l'endroit** du préchargement.

Rappel du cycle de vie :

```text
  constructeur
      -> onGameResize(size)
      -> onLoad()            <-- ICI on charge les images (une seule fois)
      -> onMount()
      -> boucle : update(dt) puis render(canvas)
      -> onRemove()
```

Le patron canonique, au niveau du jeu :

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // 1. On charge TOUT ce dont le niveau a besoin.
    await images.loadAll([
      'heros.png',
      'gobelin.png',
      'potion.png',
      'cle.png',
    ]);

    // 2. Seulement ensuite, on construit les composants.
    await world.add(Heros(position: Vector2(64, 64)));
    await world.add(Gobelin(position: Vector2(220, 96)));
  }
}
```

Pourquoi ce `await super.onLoad()` en première ligne ? Parce que `FlameGame.onLoad` met en place le `world` et la `camera`. Si vous ajoutez un composant avant, vous travaillez sur un arbre incomplet.

Pourquoi précharger au niveau du jeu plutôt que dans chaque composant ? Les deux se pratiquent, et ils ne servent pas la même chose.

| Stratégie | Où | Avantage | Inconvénient |
| --- | --- | --- | --- |
| Préchargement global | `FlameGame.onLoad` | tout est prêt avant la première frame, aucun à-coup en jeu | l'écran de chargement dure plus longtemps |
| Chargement par composant | `Component.onLoad` | chaque composant est autonome et réutilisable | premier gobelin créé en pleine partie = micro-freeze |

Le compromis courant, celui que nous utiliserons dans le mini-projet : **précharger globalement** dans `FlameGame.onLoad`, puis **lire depuis le cache** dans chaque composant, avec `fromCache`, qui est synchrone et instantané.

```dart
class Gobelin extends SpriteComponent {
  Gobelin({super.position}) : super(size: Vector2.all(32), anchor: Anchor.center);

  @override
  void onLoad() {
    // Pas de await : l'image est déjà dans le cache, mise là par le jeu.
    sprite = Sprite(Flame.images.fromCache('gobelin.png'));
  }
}
```

Notez la signature : `void onLoad()` et non `Future<void> onLoad() async`. C'est autorisé, car `Component.onLoad` renvoie un `FutureOr<void>`. Quand il n'y a rien à attendre, on ne déclare pas la méthode `async`.

> **Le bug à connaître.** Si un composant appelle `fromCache('gobelin.png')` alors que le jeu n'a pas encore chargé l'image, Flame lève une assertion très explicite :
>
> ```text
> Tried to access an image "gobelin.png" that does not exist in the cache.
> Make sure to load() an image before accessing it
> ```
>
> Le correctif n'est jamais « ajouter un délai ». Il est toujours « charger avant, et attendre le chargement ».

---

## 29.7 — La classe `Sprite`

Un `ui.Image` est une image entière. Un **`Sprite`**, dans le vocabulaire de Flame, est une **portion rectangulaire** d'une image, prête à être dessinée.

```text
  ui.Image  (la planche entière, 192 x 96 pixels)
  ┌───────────────────────────────────────────────┐
  │ ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐          │
  │ │ 0  ││ 1  ││ 2  ││ 3  ││ 4  ││ 5  │          │
  │ └────┘└────┘└────┘└────┘└────┘└────┘          │
  │ ┌────┐┌────┐...                               │
  │ │ 6  ││ 7  │                                  │
  │ └────┘└────┘                                  │
  └───────────────────────────────────────────────┘
        ▲
        └── un Sprite = (image, srcPosition, srcSize)
            c'est-à-dire : « la case n° 2 de cette image »
```

Signature exacte du constructeur en 1.38.0 :

```dart
Sprite(Image image, {Vector2? srcPosition, Vector2? srcSize})
```

| Paramètre | Type | Défaut | Signification |
| --- | --- | --- | --- |
| `image` | `Image` (de `dart:ui`) | requis | l'image source, déjà chargée |
| `srcPosition` | `Vector2?` | `Vector2.zero()` | coin haut-gauche de la découpe, **en pixels de l'image source** |
| `srcSize` | `Vector2?` | taille totale de l'image | dimensions de la découpe |

Si vous omettez `srcPosition` et `srcSize`, le sprite représente **toute l'image**. C'est le cas le plus simple :

```dart
final image = await Flame.images.load('potion.png');
final potion = Sprite(image);
```

Propriétés utiles :

| Propriété | Type | Rôle |
| --- | --- | --- |
| `image` | `Image` | l'image source (modifiable) |
| `srcPosition` | `Vector2` | coin de la découpe (modifiable) |
| `srcSize` | `Vector2` | taille de la découpe (modifiable) |
| `src` | `Rect` | la découpe sous forme de `Rect` |
| `originalSize` | `Vector2` | taille de l'image **entière** (lecture seule) |

Et les méthodes de rendu :

```dart
void render(
  Canvas canvas, {
  Vector2? position,
  Vector2? size,
  Anchor anchor = Anchor.topLeft,
  Paint? overridePaint,
  double? bleed,
});

void renderRect(Canvas canvas, Rect rect, {Paint? overridePaint});
```

Exemple de dessin direct, sans passer par un composant. C'est l'équivalent Flame du `canvas.drawImageRect` du chapitre 22 :

```dart
class Decor extends Component {
  late final Sprite mur;

  @override
  Future<void> onLoad() async {
    mur = Sprite(await Flame.images.load('mur.png'));
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    mur.render(
      canvas,
      position: Vector2(0, 160),
      size: Vector2(320, 32),
    );
  }
}
```

> **Un `Sprite` n'est pas un composant.** Il ne possède ni position propre, ni cycle de vie, ni parent. C'est une simple description : « telle zone de telle image ». Pour le placer dans le jeu, on l'enveloppe dans un `SpriteComponent` (section 29.10).

Le paramètre `bleed` de `render` mérite une mention. Il agrandit très légèrement la zone source pour combattre les fines lignes parasites qui apparaissent parfois entre deux tuiles, phénomène étudié au chapitre 22 sous le nom de *texture bleeding*. Une valeur de `1.0` suffit en général :

```dart
tuile.render(canvas, position: p, size: Vector2.all(16), bleed: 1.0);
```

---

## 29.8 — `Sprite.load()`

Charger l'image puis construire le sprite en deux lignes est très courant. Flame fournit donc un raccourci statique :

```dart
static Future<Sprite> load(
  String src, {
  Vector2? srcPosition,
  Vector2? srcSize,
  Images? images,
  String? package,
});
```

Les deux écritures suivantes sont équivalentes :

```dart
// Version longue
final image = await Flame.images.load('heros.png');
final sprite = Sprite(image);

// Version courte
final sprite = await Sprite.load('heros.png');
```

`Sprite.load` accepte directement le découpage :

```dart
final tete = await Sprite.load(
  'heros.png',
  srcPosition: Vector2(0, 0),
  srcSize: Vector2(32, 32),
);
```

Le paramètre `images` permet de viser un autre cache que le cache global :

```dart
final sprite = await Sprite.load('heros.png', images: game.images);
```

### Quand ne PAS utiliser `Sprite.load`

`Sprite.load` est un `await` déguisé sur `Flame.images.load`. Il est donc **asynchrone**, avec les conséquences habituelles.

| Situation | Verdict | Pourquoi |
| --- | --- | --- |
| Dans `onLoad()` d'un composant unique | parfait | c'est exactement l'usage prévu |
| Dans `onLoad()` d'un composant créé 200 fois pendant la partie | à éviter | 200 `await` inutiles ; préchargez et utilisez `fromCache` |
| Dans un constructeur | impossible | un constructeur n'est pas `async` |
| Dans `update(dt)` | interdit | vous lanceriez un chargement à chaque frame |
| Quand l'image vient de votre code (solution de repli) | inapplicable | il n'y a pas de fichier ; utilisez `Sprite(fromCache(...))` |

> **Règle simple.** `Sprite.load` pour un décor unique ou un prototype ; préchargement global plus `Sprite(Flame.images.fromCache(...))` pour tout ce qui est créé en série.

---

## 29.9 — `srcPosition` et `srcSize` : découper une planche

C'est ici que le chapitre 22 est directement remboursé. Vous savez déjà calculer le rectangle d'une frame ; Flame vous demande exactement les deux mêmes nombres, sous forme de `Vector2`.

Reprenons la planche du héros du Donjon de Dart : cases de 32 × 32, six colonnes, trois lignes.

```text
                    colonne 0   1     2     3     4     5
                  ┌─────┬─────┬─────┬─────┬─────┬─────┐
  ligne 0  y=0    │  0  │  1  │  2  │  3  │  4  │  5  │   idle
                  ├─────┼─────┼─────┼─────┼─────┼─────┤
  ligne 1  y=32   │  6  │  7  │  8  │  9  │ 10  │ 11  │   marche
                  ├─────┼─────┼─────┼─────┼─────┼─────┤
  ligne 2  y=64   │ 12  │ 13  │ 14  │ 15  │ 16  │ 17  │   attaque
                  └─────┴─────┴─────┴─────┴─────┴─────┘
                 x=0    32    64    96   128   160   192

  Chaque case fait 32 x 32. L'image entière fait 192 x 96.
```

La formule est celle du chapitre 22 :

```text
  srcPosition = Vector2(colonne * largeurCase, ligne * hauteurCase)
  srcSize     = Vector2(largeurCase, hauteurCase)
```

Et sa traduction en code :

```dart
final planche = await Flame.images.load('heros.png');

// Case (ligne 1, colonne 2) : troisième frame de la marche.
final frameMarche2 = Sprite(
  planche,
  srcPosition: Vector2(2 * 32, 1 * 32),   // Vector2(64, 32)
  srcSize: Vector2.all(32),
);
```

Depuis un **identifiant** de case, la conversion index → (ligne, colonne) reste la même qu'au chapitre 22 :

```dart
Sprite spriteParId(ui.Image planche, int id, int colonnes, double taille) {
  final int ligne = id ~/ colonnes;      // division entière
  final int colonne = id % colonnes;     // reste
  return Sprite(
    planche,
    srcPosition: Vector2(colonne * taille, ligne * taille),
    srcSize: Vector2.all(taille),
  );
}
```

Programme complet et exécutable qui affiche trois découpes différentes de la même planche. La planche est ici fabriquée en code (voir 29.26 pour le détail du générateur) afin que l'exemple tourne sans fichier.

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

/// Fabrique un ui.Image en dessinant dedans avec un Canvas ordinaire.
Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

/// Une planche 3 cases x 1 ligne, chaque case 32 x 32, de couleurs différentes.
Future<ui.Image> creerPlancheTest() {
  const couleurs = [
    Color(0xFFE8B04B), // or
    Color(0xFF6FCF6F), // vert gobelin
    Color(0xFFE0245E), // rouge potion
  ];
  return imageDepuisDessin(96, 32, (Canvas canvas) {
    for (int i = 0; i < 3; i++) {
      canvas.drawRect(
        Rect.fromLTWH(i * 32.0 + 2, 2, 28, 28),
        Paint()..color = couleurs[i],
      );
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuDecoupe()));
}

class JeuDecoupe extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('planche.png', creerPlancheTest);
    final planche = Flame.images.fromCache('planche.png');

    for (int i = 0; i < 3; i++) {
      await world.add(
        SpriteComponent(
          sprite: Sprite(
            planche,
            srcPosition: Vector2(i * 32.0, 0),
            srcSize: Vector2.all(32),
          ),
          position: Vector2(40 + i * 90.0, 100),
          size: Vector2.all(64),
        ),
      );
    }
  }
}
```

**Résultat :**

```text
  Trois carrés de 64 x 64 alignés horizontalement sur fond sombre :
  un carré doré, un carré vert, un carré rouge.
  Une seule image source, trois découpes différentes.
```

> **Piège de dimension.** `srcPosition` et `srcSize` s'expriment en **pixels de l'image source**, jamais en pixels d'écran. Un sprite découpé en 32 × 32 peut parfaitement être affiché en 128 × 128 : c'est le `size` du composant qui décide de la taille à l'écran. Ne confondez jamais les deux.

---

## 29.10 — `SpriteComponent`

`SpriteComponent` est un `PositionComponent` (chapitre 28) qui dessine un `Sprite`. C'est le composant le plus utilisé de tout Flame.

Constructeur réel en 1.38.0 :

```dart
SpriteComponent({
  Sprite? sprite,
  bool? autoResize,
  Paint? paint,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  double nativeAngle = 0,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
  double? bleed,
  ComponentKey? key,
});
```

Il hérite de tout ce que vous savez déjà : `position`, `size`, `angle`, `anchor`, `priority`, `add`, `removeFromParent`.

Trois façons de s'en servir, de la plus directe à la plus structurée.

**1. Instance directe, sprite fourni au constructeur.**

```dart
final potion = SpriteComponent(
  sprite: Sprite(Flame.images.fromCache('potion.png')),
  position: Vector2(120, 80),
  size: Vector2.all(24),
  anchor: Anchor.center,
);
await world.add(potion);
```

**2. Sous-classe qui charge son sprite dans `onLoad`.** C'est la forme normale dès qu'un comportement s'ajoute.

```dart
class Potion extends SpriteComponent {
  Potion({super.position})
      : super(size: Vector2.all(24), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('potion.png');
  }
}
```

**3. Sous-classe avec logique dans `update`.** Le fil rouge : une potion qui flotte doucement.

```dart
// Nécessite : import 'dart:math' as math;
class PotionFlottante extends SpriteComponent {
  PotionFlottante({required Vector2 position})
      : _yInitial = position.y,
        super(
          position: position,
          size: Vector2.all(24),
          anchor: Anchor.center,
        );

  final double _yInitial;
  double _temps = 0;

  @override
  void onLoad() {
    sprite = Sprite(Flame.images.fromCache('potion.png'));
  }

  @override
  void update(double dt) {
    super.update(dt);
    _temps += dt;
    // 4 pixels d'amplitude, un aller-retour toutes les ~3,1 secondes.
    position.y = _yInitial + math.sin(_temps * 2) * 4;
  }
}
```

### Le paramètre `autoResize`

`autoResize` décide si le composant adapte automatiquement sa `size` à celle du sprite.

| Ce que vous écrivez | Comportement |
| --- | --- |
| `SpriteComponent(sprite: s)` sans `size` | `autoResize` vaut `true` : la taille suit le sprite |
| `SpriteComponent(sprite: s, size: Vector2.all(64))` | `autoResize` vaut `false` : votre taille est respectée |
| `SpriteComponent(sprite: s, autoResize: true, size: ...)` | la taille sera écrasée par celle du sprite |

En pratique : donnez toujours une `size` explicite. Un sprite de 32 pixels affiché en 32 pixels sur un téléphone moderne est invisible.

### Le paramètre `paint`

`SpriteComponent` possède un `Paint` (via le mixin `HasPaint`). Il sert à teinter, à rendre transparent, ou à régler la qualité de filtrage :

```dart
final gobelin = SpriteComponent(
  sprite: Sprite(Flame.images.fromCache('gobelin.png')),
  size: Vector2.all(32),
  paint: Paint()
    ..filterQuality = FilterQuality.none   // pixel art net (section 29.23)
    ..isAntiAlias = false,
);

// Modification après coup :
gobelin.paint.color = const Color(0x80FFFFFF);  // à moitié transparent
```

---

## 29.11 — `SpriteComponent.fromImage()`

Quand vous avez déjà l'image sous la main, ce constructeur nommé évite de construire un `Sprite` explicitement.

```dart
SpriteComponent.fromImage(
  Image image, {
  Vector2? srcPosition,
  Vector2? srcSize,
  bool? autoResize,
  Paint? paint,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  double nativeAngle = 0,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
  ComponentKey? key,
  double? bleed,
});
```

Le découpage se fait directement dans l'appel :

```dart
final planche = Flame.images.fromCache('heros.png');

final heros = SpriteComponent.fromImage(
  planche,
  srcPosition: Vector2(0, 0),
  srcSize: Vector2.all(32),
  position: Vector2(64, 64),
  size: Vector2.all(64),
  anchor: Anchor.center,
);
```

Les deux écritures suivantes produisent exactement le même composant :

```dart
// Avec Sprite explicite
SpriteComponent(
  sprite: Sprite(planche, srcPosition: Vector2(32, 0), srcSize: Vector2.all(32)),
  size: Vector2.all(64),
);

// Avec fromImage
SpriteComponent.fromImage(
  planche,
  srcPosition: Vector2(32, 0),
  srcSize: Vector2.all(32),
  size: Vector2.all(64),
);
```

Choisissez `fromImage` quand vous n'avez besoin du `Sprite` nulle part ailleurs. Choisissez `Sprite` explicite quand le même sprite sert à plusieurs composants : vous n'en construirez qu'un.

> **Attention à un constructeur qui n'existe pas.** On lit parfois `SpriteComponent.load('heros.png')` dans de vieux tutoriels. Ce constructeur **n'est pas documenté en 1.38.0**. Utilisez `Sprite.load` puis `SpriteComponent(sprite: ...)`, ou `SpriteComponent.fromImage`.

---

## 29.12 — Taille, ancre et position d'un sprite

Trois propriétés, trois sources de confusion. Reprenons-les proprement, car 90 % des « mon sprite est décalé » viennent d'ici.

### `size` : la taille à l'écran

`size` est la taille du composant **dans le monde du jeu**, exprimée en unités du monde. Elle n'a aucun rapport avec la taille de la découpe source.

```dart
// Une découpe de 16 x 16 pixels, affichée en 64 x 64 unités : x4.
SpriteComponent(
  sprite: Sprite(planche, srcPosition: Vector2.zero(), srcSize: Vector2.all(16)),
  size: Vector2.all(64),
);
```

**Ne déformez pas.** Si la découpe est carrée, gardez une taille carrée. Une découpe 16 × 24 affichée en 64 × 64 donne un personnage écrasé. Le réflexe : appliquer un facteur, pas des dimensions absolues.

```dart
const double zoom = 3;
final srcTaille = Vector2(16, 24);
SpriteComponent(
  sprite: Sprite(planche, srcSize: srcTaille),
  size: srcTaille * zoom,     // Vector2(48, 72) : proportions conservées
);
```

`Vector2` étant un vecteur de `vector_math`, la multiplication par un scalaire fonctionne directement. Rappel du chapitre 28 : `Vector2` est **mutable**, donc `srcTaille * zoom` crée bien un nouveau vecteur, mais `size = srcTaille` partagerait la même instance. Utilisez `.clone()` en cas de doute.

### `anchor` : le point de référence

`anchor` désigne le point du composant qui se trouve **exactement à `position`**. Valeur par défaut : `Anchor.topLeft`.

```text
   Anchor.topLeft                     Anchor.center
   position = (100, 100)              position = (100, 100)

   (100,100)                                  (100,100)
      ●───────────────┐                   ┌───────●───────┐
      │               │                   │       │       │
      │    sprite     │                   │    sprite     │
      │               │                   │               │
      └───────────────┘                   └───────────────┘
   Le sprite occupe                     Le sprite occupe
   [100..164] x [100..164]              [68..132] x [68..132]
```

Valeurs disponibles : `topLeft`, `topCenter`, `topRight`, `centerLeft`, `center`, `centerRight`, `bottomLeft`, `bottomCenter`, `bottomRight`.

Quelle ancre choisir ? La réponse dépend de ce que la position **signifie** dans votre jeu.

| Type d'entité | Ancre recommandée | Raison |
| --- | --- | --- |
| Personnage vu de dessus | `Anchor.center` | la rotation et les distances se calculent depuis le centre |
| Personnage de plateforme | `Anchor.bottomCenter` | la position est « les pieds », ce qui simplifie le sol |
| Tuile de décor | `Anchor.topLeft` | la grille se calcule en multiples de la taille de tuile |
| Élément de HUD | `Anchor.topLeft` ou `topRight` | l'alignement sur le bord est direct |
| Barre de vie au-dessus d'un ennemi | `Anchor.bottomCenter` | elle reste centrée quelle que soit sa largeur |

`anchor` gouverne aussi la **rotation** et la **mise à l'échelle** : un composant tourne autour de son ancre. Une épée qui pivote doit avoir son ancre sur la poignée, pas sur la lame.

```dart
final epee = SpriteComponent(
  sprite: epeeSprite,
  size: Vector2(8, 32),
  anchor: Anchor.bottomCenter,   // la poignée est en bas
  position: Vector2(100, 100),
);
epee.angle = math.pi / 4;        // pivote autour de la poignée (dart:math)
```

### `position` : dans le repère du parent

`position` est exprimée dans le repère du **parent**, pas de l'écran. Un composant enfant d'un autre composant se positionne relativement à lui.

```dart
class Heros extends SpriteComponent {
  Heros() : super(size: Vector2.all(48), anchor: Anchor.center);

  @override
  Future<void> onLoad() async {
    sprite = Sprite(Flame.images.fromCache('heros.png'));

    // Enfant : une petite barre de vie AU-DESSUS du héros.
    // (0, -6) est relatif au héros, pas à l'écran.
    await add(
      RectangleComponent(
        position: Vector2(size.x / 2, -6),
        size: Vector2(40, 4),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFF4CAF50),
      ),
    );
  }
}
```

Quand le héros bouge, la barre de vie suit toute seule. C'est l'un des bénéfices majeurs de l'arbre de composants vu au chapitre 28.

> **Rappel du chapitre 28.** `position` est un `NotifyingVector2`. Écrire `position.x += 100 * dt;` fonctionne et notifie Flame du changement. Remplacer l'instance entière (`position = Vector2(...)`) fonctionne aussi.

---

## 29.13 — Retourner un sprite

Un personnage qui marche vers la gauche est le plus souvent le même sprite que celui qui marche vers la droite, retourné horizontalement. Dessiner deux jeux de frames serait un gaspillage.

Au chapitre 22, vous l'avez fait à la main :

```dart
// Chapitre 22, à la main
canvas.save();
canvas.translate(x + largeur, y);
canvas.scale(-1, 1);
canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, largeur, hauteur), paint);
canvas.restore();
```

Flame fournit quatre méthodes sur **tout `PositionComponent`** :

```dart
void flipHorizontally();
void flipVertically();
void flipHorizontallyAroundCenter();
void flipVerticallyAroundCenter();
```

La documentation officielle précise leur portée :

> *« you can also use `flipHorizontally()` and `flipVertically()` to flip anything drawn to canvas during `render(Canvas canvas)`, around the anchor point. »*

### `flipHorizontally()` — autour de l'ancre

Le retournement s'opère autour du **point d'ancrage**. Conséquence pratique :

```text
  anchor = Anchor.center                anchor = Anchor.topLeft
  ┌───────┬───────┐                     ●───────────────┐
  │       ●       │                     │               │
  └───────┴───────┘                     └───────────────┘
  flip -> le sprite reste               flip -> le sprite bascule
  exactement au même endroit            à GAUCHE de sa position
```

Avec `Anchor.center`, `flipHorizontally()` est visuellement parfait. Avec `Anchor.topLeft`, le sprite se décale de sa propre largeur, ce qui surprend toujours au premier essai.

### `flipHorizontallyAroundCenter()` — autour du centre géométrique

Cette variante retourne le composant autour de son centre **quelle que soit l'ancre**. C'est la méthode à privilégier quand vous n'êtes pas sûr, ou quand l'ancre est en bas (personnage de plateforme).

### Comment ça marche réellement

Ces méthodes agissent sur `scale` : elles multiplient `scale.x` par `-1`. Deux conséquences à connaître.

**Conséquence 1 : appeler deux fois annule.** `flipHorizontally()` puis `flipHorizontally()` remet le sprite à l'endroit. Ce n'est donc **pas** une méthode « regarder à gauche », mais une méthode « basculer ».

**Conséquence 2 : il faut mémoriser l'état.** Le code fautif que tout le monde écrit une fois :

```dart
// FAUX : à chaque frame où l'on va à gauche, on bascule. Le sprite clignote.
@override
void update(double dt) {
  super.update(dt);
  if (vitesse.x < 0) {
    flipHorizontally();
  }
}
```

Le code juste, avec un booléen d'état :

```dart
class Heros extends SpriteComponent {
  Heros() : super(size: Vector2.all(48), anchor: Anchor.center);

  final Vector2 vitesse = Vector2.zero();
  bool _regardeAGauche = false;

  @override
  void update(double dt) {
    super.update(dt);
    position += vitesse * dt;

    if (vitesse.x < 0 && !_regardeAGauche) {
      _regardeAGauche = true;
      flipHorizontallyAroundCenter();
    } else if (vitesse.x > 0 && _regardeAGauche) {
      _regardeAGauche = false;
      flipHorizontallyAroundCenter();
    }
  }
}
```

Variante encore plus explicite, sans `flip` : on pilote `scale.x` directement.

```dart
@override
void update(double dt) {
  super.update(dt);
  position += vitesse * dt;

  if (vitesse.x != 0) {
    // -1 : regarde à gauche ; +1 : regarde à droite.
    scale.x = vitesse.x < 0 ? -1 : 1;
  }
}
```

Cette seconde forme est idempotente : la réécrire à chaque frame ne pose aucun problème. C'est celle que nous utiliserons dans le mini-projet.

> **Attention aux hitbox (chapitre 32).** Retourner un composant retourne aussi ses enfants, hitbox comprises. Une hitbox symétrique n'en souffre pas. Une hitbox décalée (une épée devant le personnage) doit être repositionnée. Le correctif de Flame 1.36.0 sur les hitbox de parents mis à l'échelle rend ce cas fiable ; les contournements des vieux tutoriels sont devenus inutiles.

---

## 29.14 — `SpriteAnimation`

Une **animation** est une liste ordonnée de sprites, chacun affiché pendant une durée donnée. C'est exactement la définition du chapitre 22, et Flame la reprend telle quelle.

```text
  SpriteAnimation
  ┌──────────┬──────────┬──────────┬──────────┐
  │ frame 0  │ frame 1  │ frame 2  │ frame 3  │
  │  0,12 s  │  0,12 s  │  0,12 s  │  0,12 s  │
  └──────────┴──────────┴──────────┴──────────┘
      ▲
      └─ chaque frame = un Sprite + un stepTime
```

Constructeurs disponibles en 1.38.0 :

```dart
SpriteAnimation(List<SpriteAnimationFrame> frames, {bool loop = true});

SpriteAnimation.spriteList(
  List<Sprite> sprites, {
  required double stepTime,
  bool loop = true,
});

SpriteAnimation.variableSpriteList(
  List<Sprite> sprites, {
  required List<double> stepTimes,
  bool loop = true,
});

SpriteAnimation.fromFrameData(Image image, SpriteAnimationData data);

SpriteAnimation.fromAsepriteData(Image image, Map<String, dynamic> jsonData);

static Future<SpriteAnimation> load(
  String src,
  SpriteAnimationData data, {
  Images? images,
  String? package,
});
```

Vous en utiliserez deux, presque toujours les deux mêmes.

### `SpriteAnimation.spriteList` — quand vous avez déjà les sprites

C'est le constructeur le plus lisible, et celui qui convient à des frames provenant de fichiers séparés ou d'un découpage fait à la main.

```dart
final planche = Flame.images.fromCache('heros.png');

// Les quatre frames de la ligne 0 (idle).
final sprites = List.generate(
  4,
  (int i) => Sprite(
    planche,
    srcPosition: Vector2(i * 32.0, 0),
    srcSize: Vector2.all(32),
  ),
);

final idle = SpriteAnimation.spriteList(sprites, stepTime: 0.18);
```

`List.generate` vient du chapitre 6. La fonction anonyme passée en second argument est une fonction d'ordre supérieur du chapitre 7.

Depuis des fichiers séparés (`heros_0.png`, `heros_1.png`, `heros_2.png`) :

```dart
final futurs = [0, 1, 2].map((int i) => Sprite.load('heros_$i.png'));
final animation = SpriteAnimation.spriteList(
  await Future.wait(futurs),
  stepTime: 0.1,
);
```

`Future.wait` vient du chapitre 15 : il attend une liste de futures en parallèle.

### `SpriteAnimation.fromFrameData` — quand tout est dans une planche

C'est la voie la plus courte pour une sprite sheet régulière. Elle demande une description, objet `SpriteAnimationData`, détaillé à la section suivante.

```dart
final marche = SpriteAnimation.fromFrameData(
  planche,
  SpriteAnimationData.sequenced(
    amount: 6,
    stepTime: 0.1,
    textureSize: Vector2.all(32),
    texturePosition: Vector2(0, 32),   // ligne 1 de la planche
    amountPerRow: 6,
  ),
);
```

### Propriétés et méthodes

| Membre | Type | Rôle |
| --- | --- | --- |
| `frames` | `List<SpriteAnimationFrame>` | les frames et leurs durées |
| `loop` | `bool` | l'animation reboucle-t-elle après la dernière frame ? |
| `stepTime` | `double` (écriture seule) | fixe la même durée pour toutes les frames |
| `variableStepTimes` | `List<double>` (écriture seule) | fixe une durée par frame |
| `createTicker()` | `SpriteAnimationTicker` | crée un lecteur pour cette animation |
| `reversed()` | `SpriteAnimation` | une nouvelle animation jouée à l'envers |
| `clone()` | `SpriteAnimation` | une copie indépendante |

> **Point crucial : une `SpriteAnimation` ne contient AUCUN état de lecture.** Elle ne sait pas quelle frame est affichée, ni depuis combien de temps. Elle est une simple **description**. C'est le `SpriteAnimationTicker` qui joue.
>
> Conséquence heureuse : vous pouvez partager la même `SpriteAnimation` entre trente gobelins. Chacun aura son propre ticker, donc sa propre progression, et ils ne seront pas tous synchronisés.

### `SpriteAnimationTicker`

Le lecteur, justement. Vous le manipulerez rarement directement, car les composants d'animation en créent un pour vous, mais il faut connaître son API.

```dart
SpriteAnimationTicker(SpriteAnimation spriteAnimation);
```

| Membre | Type | Rôle |
| --- | --- | --- |
| `currentIndex` | `int` | index de la frame affichée |
| `clock` | `double` | temps total écoulé, en secondes |
| `elapsed` | `double` | temps écoulé depuis le début ou le dernier `reset()` |
| `isFirstFrame` / `isLastFrame` | `bool` | position dans la séquence |
| `done()` | `bool` | l'animation est-elle terminée ? |
| `completed` | `Future<void>` | future qui se complète à la fin |
| `reset()` | `void` | revient à la frame 0 |
| `setToLast()` | `void` | saute à la dernière frame |
| `update(double dt)` | `void` | fait avancer le temps |
| `getSprite()` | `Sprite` | le sprite courant |
| `onStart` | `void Function()?` | callback de démarrage |
| `onComplete` | `void Function()?` | callback de fin |
| `onFrame` | `void Function(int)?` | callback à chaque changement de frame |

Usage manuel, sans composant, pour bien voir le mécanisme :

```dart
class RendezVousAnime extends Component {
  late final SpriteAnimationTicker ticker;

  @override
  Future<void> onLoad() async {
    final animation = SpriteAnimation.spriteList(sprites, stepTime: 0.1);
    ticker = animation.createTicker();
  }

  @override
  void update(double dt) {
    super.update(dt);
    ticker.update(dt);          // c'est le ticker qui avance, pas l'animation
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    ticker.getSprite().render(canvas, size: Vector2.all(64));
  }
}
```

---

## 29.15 — `SpriteAnimationData.sequenced()`

`SpriteAnimationData` décrit **où** se trouvent les frames dans une planche. C'est l'équivalent Flame de la boucle de calcul de `Rect` que vous écriviez au chapitre 22.

Signature exacte :

```dart
SpriteAnimationData.sequenced({
  required int amount,
  required double stepTime,
  required Vector2 textureSize,
  int? amountPerRow,
  Vector2? texturePosition,
  bool loop = true,
});
```

| Paramètre | Type | Défaut | Signification |
| --- | --- | --- | --- |
| `amount` | `int` | requis | nombre total de frames de l'animation |
| `stepTime` | `double` | requis | durée d'affichage d'une frame, **en secondes** |
| `textureSize` | `Vector2` | requis | taille d'une case dans l'image source, en pixels |
| `amountPerRow` | `int?` | `amount` | nombre de cases par ligne avant de passer à la ligne suivante |
| `texturePosition` | `Vector2?` | `Vector2.zero()` | coin haut-gauche de la **première** frame dans l'image |
| `loop` | `bool` | `true` | l'animation reboucle-t-elle ? |

Trois cas de figure suffisent à tout couvrir.

### Cas 1 — Une planche d'une seule ligne

```text
  ┌─────┬─────┬─────┬─────┬─────┬─────┐
  │  0  │  1  │  2  │  3  │  4  │  5  │      image 192 x 32
  └─────┴─────┴─────┴─────┴─────┴─────┘
```

```dart
SpriteAnimationData.sequenced(
  amount: 6,
  stepTime: 0.1,
  textureSize: Vector2.all(32),
);
```

`amountPerRow` vaut `amount` par défaut : les six frames sont lues sur une seule ligne. Rien d'autre à préciser.

### Cas 2 — Une ligne précise d'une planche à plusieurs lignes

C'est le cas de notre planche du héros. On veut la ligne 1 (marche), six frames.

```text
                  ┌─────┬─────┬─────┬─────┬─────┬─────┐
  y = 0   idle    │     │     │     │     │     │     │
                  ├─────┼─────┼─────┼─────┼─────┼─────┤
  y = 32  marche  │  X  │  X  │  X  │  X  │  X  │  X  │   <-- celle-ci
                  ├─────┼─────┼─────┼─────┼─────┼─────┤
  y = 64  attaque │     │     │     │     │     │     │
                  └─────┴─────┴─────┴─────┴─────┴─────┘
```

```dart
SpriteAnimationData.sequenced(
  amount: 6,
  stepTime: 0.1,
  textureSize: Vector2.all(32),
  texturePosition: Vector2(0, 32),   // on démarre à la ligne 1
  amountPerRow: 6,                   // et on ne déborde pas dessus
);
```

**`amountPerRow` est indispensable ici.** Sans lui, il vaudrait `amount`, ce qui donnerait le même résultat par chance. Mais si l'animation ne comptait que 4 frames sur une planche de 6 colonnes, l'omettre ferait lire les frames 0 à 3 de la ligne… puis passer à la ligne suivante pour la cinquième. Prenez l'habitude de toujours l'écrire : il décrit la **planche**, pas l'animation.

### Cas 3 — Une animation qui déborde sur plusieurs lignes

```text
  ┌─────┬─────┬─────┬─────┐
  │  0  │  1  │  2  │  3  │      amountPerRow = 4
  ├─────┼─────┼─────┼─────┤
  │  4  │  5  │  6  │  7  │      amount = 10
  ├─────┼─────┼─────┼─────┤
  │  8  │  9  │     │     │      les cases vides sont ignorées
  └─────┴─────┴─────┴─────┘
```

```dart
SpriteAnimationData.sequenced(
  amount: 10,
  stepTime: 0.08,
  textureSize: Vector2.all(32),
  amountPerRow: 4,
);
```

### Les variantes

**Durées variables** — utile pour une attaque où l'on veut « tenir » la frame d'impact :

```dart
SpriteAnimationData.variable(
  amount: 4,
  stepTimes: [0.05, 0.05, 0.25, 0.10],   // autant de valeurs que de frames
  textureSize: Vector2.all(32),
  texturePosition: Vector2(0, 64),
  amountPerRow: 6,
  loop: false,
);
```

**Constructeur générique** — vous fournissez la liste des frames :

```dart
SpriteAnimationData(List<SpriteAnimationFrameData> frames, {bool loop = true});
```

C'est ce que renvoie `SpriteSheet.createFrameData` (section 29.22), ce qui permet de composer une animation à partir de cases éparpillées dans la planche.

> **Erreur fréquente : `stepTimes` de mauvaise longueur.** `SpriteAnimationData.variable` exige que `stepTimes.length == amount`. Une liste plus courte lève une exception au chargement.

---

## 29.16 — `stepTime` et `loop`

Deux paramètres, deux erreurs de débutant très répandues.

### `stepTime` est en SECONDES

C'est un `double`, exprimé en **secondes**, comme le `dt` du chapitre 20 et comme la `period` d'un `TimerComponent`.

| Ce que vous voulez | `stepTime` |
| --- | --- |
| 10 images par seconde | `0.1` |
| 12 images par seconde (cadence d'animation classique) | `1 / 12`, soit environ `0.0833` |
| 8 images par seconde (marche lourde) | `0.125` |
| 24 images par seconde (animation fluide, coûteuse en frames) | `1 / 24`, soit environ `0.0417` |
| 2 images par seconde (respiration très lente) | `0.5` |

L'erreur classique consiste à raisonner en millisecondes :

```dart
// FAUX : 100 secondes par frame. Le personnage semble figé.
SpriteAnimationData.sequenced(amount: 6, stepTime: 100, textureSize: ...);

// JUSTE : 100 millisecondes = 0,1 seconde.
SpriteAnimationData.sequenced(amount: 6, stepTime: 0.1, textureSize: ...);
```

Le symptôme est reconnaissable : « mon animation ne bouge pas ». Elle bouge, mais une frame toutes les cent secondes.

Le symptôme inverse existe aussi :

```dart
// FAUX : 1 milliseconde par frame. Le personnage vibre.
SpriteAnimationData.sequenced(amount: 6, stepTime: 0.001, textureSize: ...);
```

Une formule utile pour raisonner en « images par seconde » :

```dart
double cadence(double imagesParSeconde) => 1 / imagesParSeconde;

// Lisible et sans ambiguïté :
SpriteAnimationData.sequenced(
  amount: 6,
  stepTime: cadence(12),
  textureSize: Vector2.all(32),
);
```

### Durée totale d'une animation

```text
  duree_totale = amount * stepTime
```

Six frames à 0,1 s font 0,6 seconde. C'est le chiffre à garder en tête pour synchroniser une animation d'attaque avec un délai de dégâts.

### `loop` : boucler ou pas

| Type d'animation | `loop` |
| --- | --- |
| idle (respiration) | `true` |
| marche, course | `true` |
| flottement d'une potion | `true` |
| attaque | `false` |
| saut, atterrissage | `false` |
| mort | `false` |
| ouverture de coffre | `false` |

Une animation `loop: false` s'arrête sur sa **dernière frame** et y reste. Elle ne disparaît pas, elle ne revient pas au début. C'est le comportement souhaité pour une mort : le cadavre reste au sol.

```dart
final mort = SpriteAnimation.fromFrameData(
  planche,
  SpriteAnimationData.sequenced(
    amount: 5,
    stepTime: 0.12,
    textureSize: Vector2.all(32),
    texturePosition: Vector2(0, 96),
    amountPerRow: 6,
    loop: false,          // reste sur la dernière frame
  ),
);
```

`loop` est aussi accessible en écriture sur l'animation elle-même :

```dart
animation.loop = false;
```

> **Piège du composant.** `loop` appartient à l'**animation**, pas au composant. `SpriteAnimationComponent` possède par ailleurs un champ `playing` (booléen) qui met la lecture en pause sans changer `loop`, et un champ `removeOnFinish` qui supprime le composant quand une animation non bouclée se termine. Ne confondez pas les trois.

---

## 29.17 — `SpriteAnimationComponent`

C'est le pendant animé de `SpriteComponent`. Constructeurs réels en 1.38.0 :

```dart
SpriteAnimationComponent({
  SpriteAnimation? animation,
  bool? autoResize,
  bool removeOnFinish = false,
  bool playing = true,
  bool resetOnRemove = false,
  Paint? paint,
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

SpriteAnimationComponent.fromFrameData(
  Image image,
  SpriteAnimationData data, {
  // mêmes paramètres nommés que ci-dessus
});
```

| Paramètre spécifique | Rôle |
| --- | --- |
| `animation` | l'animation à jouer |
| `playing` | `true` : l'animation avance ; `false` : elle est figée |
| `removeOnFinish` | supprime le composant à la fin d'une animation `loop: false` |
| `resetOnRemove` | remet le ticker à zéro quand le composant est retiré |
| `autoResize` | adapte la `size` à celle des frames |

Propriétés en lecture / écriture : `animation`, `playing`. En lecture seule : `animationTicker`.

Exemple complet et exécutable : une pièce d'or qui tourne, entièrement générée en code.

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

/// Une planche de 6 frames de 16 x 16 : une pièce vue de face,
/// de plus en plus fine, puis de nouveau large. Effet de rotation.
Future<ui.Image> creerPlanchePiece() {
  const largeurs = [12.0, 9.0, 5.0, 2.0, 5.0, 9.0];
  return imageDepuisDessin(16 * 6, 16, (Canvas canvas) {
    final or = Paint()..color = const Color(0xFFE8B04B);
    final ombre = Paint()..color = const Color(0xFFB07C22);
    for (int i = 0; i < 6; i++) {
      final double cx = i * 16 + 8;
      final double l = largeurs[i];
      canvas.drawOval(
        Rect.fromCenter(center: Offset(cx, 8), width: l, height: 12),
        or,
      );
      canvas.drawOval(
        Rect.fromCenter(center: Offset(cx, 9), width: l * 0.5, height: 6),
        ombre,
      );
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuPiece()));
}

class JeuPiece extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('piece.png', creerPlanchePiece);
    final planche = Flame.images.fromCache('piece.png');

    await world.add(
      SpriteAnimationComponent.fromFrameData(
        planche,
        SpriteAnimationData.sequenced(
          amount: 6,
          stepTime: 0.09,
          textureSize: Vector2.all(16),
        ),
        position: Vector2(140, 100),
        size: Vector2.all(64),
        anchor: Anchor.center,
      ),
    );
  }
}
```

**Résultat :**

```text
  Une pièce dorée qui tourne sur elle-même, au centre de l'écran,
  à environ 11 images par seconde. Aucun fichier image n'a été lu.
```

### Mettre en pause, reprendre

```dart
piece.playing = false;   // figée sur la frame courante
piece.playing = true;    // repart
```

C'est utile pour geler tous les ennemis pendant une pause sans passer par `pauseEngine()`.

### Remplacer l'animation

```dart
piece.animation = autreAnimation;
```

Attention : cette affectation crée un **nouveau ticker**, donc l'animation repart de la frame 0. C'est presque toujours ce que l'on veut ; la section 29.18 traite du cas contraire.

---

## 29.18 — Changer d'animation à l'exécution

Un personnage a plusieurs animations. Le premier réflexe est d'écrire :

```dart
// Fonctionne, mais mal.
if (vitesse.x != 0) {
  composant.animation = marche;
} else {
  composant.animation = idle;
}
```

Ce code est appelé **à chaque frame**. À 60 images par seconde, on remplace donc 60 fois par seconde une animation par elle-même, et le ticker repart de zéro à chaque fois. Résultat : le personnage reste bloqué sur la frame 0. C'est exactement le bug décrit au chapitre 22, section 22.26.

Le correctif minimal consiste à ne réaffecter que si l'animation change :

```dart
void _appliquer(SpriteAnimation voulue) {
  if (composant.animation != voulue) {
    composant.animation = voulue;
  }
}
```

Cette comparaison fonctionne parce que l'on compare des **identités d'objet** : `marche` et `idle` sont deux instances distinctes, conservées dans des champs.

```dart
class Heros extends SpriteAnimationComponent {
  Heros({super.position})
      : super(size: Vector2.all(64), anchor: Anchor.center);

  late final SpriteAnimation idle;
  late final SpriteAnimation marche;

  final Vector2 vitesse = Vector2.zero();

  @override
  void onLoad() {
    final planche = Flame.images.fromCache('heros.png');

    idle = SpriteAnimation.fromFrameData(
      planche,
      SpriteAnimationData.sequenced(
        amount: 4,
        stepTime: 0.18,
        textureSize: Vector2.all(32),
        amountPerRow: 6,
      ),
    );

    marche = SpriteAnimation.fromFrameData(
      planche,
      SpriteAnimationData.sequenced(
        amount: 6,
        stepTime: 0.09,
        textureSize: Vector2.all(32),
        texturePosition: Vector2(0, 32),
        amountPerRow: 6,
      ),
    );

    animation = idle;
  }

  @override
  void update(double dt) {
    super.update(dt);
    position += vitesse * dt;

    final SpriteAnimation voulue = vitesse.x == 0 ? idle : marche;
    if (animation != voulue) {
      animation = voulue;
    }
    if (vitesse.x != 0) {
      scale.x = vitesse.x < 0 ? -1 : 1;
    }
  }
}
```

Ce code fonctionne. Il a toutefois deux défauts, qui vont motiver la section suivante.

**Défaut 1 : la comparaison est fragile.** Rien n'empêche d'écrire par mégarde `animation = SpriteAnimation.fromFrameData(...)` dans `update`, ce qui recréerait une instance différente à chaque frame et ramènerait le bug.

**Défaut 2 : chaque changement remet le compteur à zéro.** Passer de marche à idle puis revenir à marche redémarre la marche à la frame 0. Ce n'est pas grave pour un personnage, mais c'est visible sur une animation lente.

Flame propose une solution dédiée à ce problème exact : `SpriteAnimationGroupComponent`.

---

## 29.19 — `SpriteAnimationGroupComponent` et les états

La documentation officielle le présente ainsi :

> *« `SpriteAnimationGroupComponent` is a simple wrapper around `SpriteAnimationComponent` which enables your component to hold several animations and change the current playing animation at runtime. »*

L'idée : au lieu d'affecter une animation, on affecte un **état**, et le composant retrouve l'animation correspondante dans une table. C'est très exactement la machine à états d'animation que vous avez écrite à la main au chapitre 22, section 22.25.

### L'enum d'états (rappel du chapitre 11)

Les enums du chapitre 11 sont l'outil idéal : un ensemble fermé de valeurs nommées, vérifiées par le compilateur.

```dart
enum EtatHeros { idle, marche, attaque, touche, mort }
```

Trois bénéfices immédiats, tous hérités du chapitre 11 :

1. **Impossible d'écrire une faute de frappe.** `EtatHeros.marhce` ne compile pas, alors que `'marhce'` en chaîne de caractères compilerait très bien et échouerait à l'exécution.
2. **Le `switch` est exhaustif.** Ajoutez `EtatHeros.saut` et l'analyseur signalera tous les `switch` incomplets.
3. **Le type générique est contrôlé.** `SpriteAnimationGroupComponent<EtatHeros>` refuse toute clé qui ne serait pas un `EtatHeros`.

### Le composant

Constructeurs réels en 1.38.0 :

```dart
SpriteAnimationGroupComponent({
  Map<T, SpriteAnimation>? animations,
  T? current,
  bool? autoResize,
  bool playing = true,
  Map<T, bool> removeOnFinish = const {},
  bool autoResetTicker = true,
  Paint? paint,
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

SpriteAnimationGroupComponent.fromFrameData(
  Image image,
  Map<T, SpriteAnimationData> data, {
  T? current,
  // ... mêmes paramètres nommés
});
```

Exemple officiel, adapté au Donjon de Dart :

```dart
enum EtatHeros { idle, marche }

final heros = SpriteAnimationGroupComponent<EtatHeros>(
  animations: {
    EtatHeros.idle: idle,
    EtatHeros.marche: marche,
  },
  current: EtatHeros.idle,
  size: Vector2.all(64),
  anchor: Anchor.center,
);

// Plus tard, n'importe où :
heros.current = EtatHeros.marche;
```

Le constructeur `fromFrameData` évite même de construire les animations une par une :

```dart
final heros = SpriteAnimationGroupComponent<EtatHeros>.fromFrameData(
  planche,
  {
    EtatHeros.idle: SpriteAnimationData.sequenced(
      amount: 4,
      stepTime: 0.18,
      textureSize: Vector2.all(32),
      amountPerRow: 6,
    ),
    EtatHeros.marche: SpriteAnimationData.sequenced(
      amount: 6,
      stepTime: 0.09,
      textureSize: Vector2.all(32),
      texturePosition: Vector2(0, 32),
      amountPerRow: 6,
    ),
  },
  current: EtatHeros.idle,
  size: Vector2.all(64),
  anchor: Anchor.center,
);
```

### Pourquoi c'est mieux que la section 29.18

```text
  SANS le groupe                        AVEC le groupe
  ─────────────────────────────         ─────────────────────────────
  champ SpriteAnimation idle            Map<EtatHeros, SpriteAnimation>
  champ SpriteAnimation marche
  champ SpriteAnimation attaque
  ...un champ par animation             une seule table

  if (animation != voulue) {            current = EtatHeros.marche;
    animation = voulue;                 (réaffecter le même état
  }                                      ne coûte rien)

  un seul ticker, remis à zéro          un ticker PAR animation,
  à chaque changement                   conservé entre les changements
```

Le champ `autoResetTicker` (par défaut `true`) contrôle ce dernier point : à `true`, revenir sur un état redémarre son animation ; à `false`, elle reprend là où elle en était.

---

## 29.20 — `animations` et `current`

Les deux propriétés centrales du composant de groupe.

### `animations`

```dart
Map<T, SpriteAnimation>? animations;
```

C'est une table `Map` (chapitre 6) dont les clés sont vos états et les valeurs vos animations. Elle est **modifiable après construction** :

```dart
heros.animations = {
  ...heros.animations!,
  EtatHeros.attaque: attaque,     // on ajoute un état
};
```

L'opérateur de propagation `...` vient du chapitre 6. Notez le `!` : `animations` est nullable, car un composant peut exister avant que ses animations ne soient renseignées.

### `current`

```dart
T? current;
```

L'état courant. Lire cette propriété indique quelle animation joue ; l'écrire change d'animation.

```dart
if (heros.current == EtatHeros.attaque) {
  // Pas d'interruption : on ignore les ordres de déplacement.
  return;
}
heros.current = EtatHeros.marche;
```

**Deux règles à respecter.**

**Règle 1 — chaque valeur de `current` doit exister dans `animations`.** Sinon le composant n'a rien à dessiner. Le réflexe défensif :

```dart
void changerEtat(EtatHeros nouvel) {
  assert(
    animations!.containsKey(nouvel),
    'Aucune animation enregistrée pour $nouvel',
  );
  current = nouvel;
}
```

**Règle 2 — ne pilotez `current` que depuis un seul endroit.** Si le clavier écrit `current = marche` et que la détection de collision écrit `current = touche` dans la même frame, le résultat dépend de l'ordre d'exécution. Concentrez la décision dans une seule méthode, appelée en fin d'`update` :

```dart
@override
void update(double dt) {
  super.update(dt);
  _deplacer(dt);
  _mettreAJourEtat();     // décide de current, une seule fois par frame
}

void _mettreAJourEtat() {
  // Les états prioritaires ne sont jamais interrompus.
  if (current == EtatHeros.mort) return;
  if (current == EtatHeros.attaque) return;

  current = vitesse.x == 0 ? EtatHeros.idle : EtatHeros.marche;
}
```

Cette structure « garde d'abord, décide ensuite » est la même que celle du chapitre 26 pour les états de jeu. Elle évite l'entrelacement de conditions qui rend un personnage impossible à déboguer.

### `animationTickers`

```dart
Map<T, SpriteAnimationTicker>? animationTickers;
```

Un ticker par état. C'est par là qu'on accroche les callbacks, sujet de la section 29.21.

### `removeOnFinish`

```dart
Map<T, bool> removeOnFinish = const {};
```

Une table qui indique, état par état, si le composant doit se retirer de l'arbre à la fin de l'animation. Idéal pour un ennemi qui meurt :

```dart
final gobelin = SpriteAnimationGroupComponent<EtatGobelin>(
  animations: {
    EtatGobelin.marche: marche,
    EtatGobelin.mort: mort,        // mort a loop: false
  },
  current: EtatGobelin.marche,
  removeOnFinish: {EtatGobelin.mort: true},
  size: Vector2.all(48),
  anchor: Anchor.center,
);

// Quand le gobelin est tué :
gobelin.current = EtatGobelin.mort;
// L'animation de mort se joue, puis le composant disparaît tout seul.
```

---

## 29.21 — Le callback de fin d'animation (`onComplete`)

Une animation d'attaque dure 0,4 seconde. Que fait le héros ensuite ? Il revient à l'état *idle*. Encore faut-il savoir **quand** l'attaque est terminée.

Trois façons de le savoir, de la moins bonne à la meilleure.

### Méthode 1 (déconseillée) — un compteur maison

```dart
// Fonctionne, mais duplique une information que Flame possède déjà.
double _restantAttaque = 0;

void attaquer() {
  current = EtatHeros.attaque;
  _restantAttaque = 4 * 0.1;    // amount * stepTime, recopié à la main
}

@override
void update(double dt) {
  super.update(dt);
  if (_restantAttaque > 0) {
    _restantAttaque -= dt;
    if (_restantAttaque <= 0) current = EtatHeros.idle;
  }
}
```

Le défaut est évident : le jour où vous changez `stepTime`, il faut penser à changer aussi cette constante. C'est le genre de duplication qui produit des bugs six mois plus tard.

### Méthode 2 — interroger le ticker

```dart
@override
void update(double dt) {
  super.update(dt);
  if (current == EtatHeros.attaque &&
      animationTickers![EtatHeros.attaque]!.done()) {
    current = EtatHeros.idle;
  }
}
```

C'est correct et sans duplication. Le seul reproche : on interroge à chaque frame ce que l'on pourrait apprendre par notification.

### Méthode 3 (recommandée) — `onComplete`

`SpriteAnimationTicker` expose trois callbacks :

```dart
void Function()? onStart;
void Function()? onComplete;
void Function(int)? onFrame;
```

Sur un `SpriteAnimationGroupComponent`, on les branche via `animationTickers` :

```dart
@override
void onLoad() {
  // ... construction des animations ...

  animationTickers?[EtatHeros.attaque]?.onComplete = () {
    current = EtatHeros.idle;
  };

  animationTickers?[EtatHeros.attaque]?.onFrame = (int index) {
    // La frame 2 est celle où l'épée touche : c'est là qu'on inflige les dégâts.
    if (index == 2) {
      _infligerDegats();
    }
  };
}
```

Sur un `SpriteAnimationComponent` simple, on passe par `animationTicker` (au singulier) :

```dart
final explosion = SpriteAnimationComponent(
  animation: animationExplosion,   // loop: false
  size: Vector2.all(48),
  anchor: Anchor.center,
  removeOnFinish: true,
);
explosion.animationTicker?.onComplete = () {
  print('Explosion terminée');
};
```

Enfin, la variante asynchrone, avec le future `completed` :

```dart
Future<void> jouerLaMort() async {
  current = EtatGobelin.mort;
  await animationTickers![EtatGobelin.mort]!.completed;
  removeFromParent();
}
```

`await` sur un future vient du chapitre 15. C'est élégant, mais gardez en tête que la méthode ne doit pas être appelée deux fois de suite.

### Tableau de synthèse

| Besoin | Outil |
| --- | --- |
| Revenir à *idle* après une attaque | `onComplete` du ticker de l'état attaque |
| Infliger des dégâts sur une frame précise | `onFrame` avec test d'index |
| Jouer un son au démarrage de l'animation | `onStart` |
| Supprimer le composant à la fin | `removeOnFinish: {etat: true}` |
| Savoir dans `update` si c'est fini | `animationTickers![etat]!.done()` |
| Attendre la fin en `async` | `await animationTickers![etat]!.completed` |

> **Piège du rebouclage.** `onComplete` n'est jamais appelé sur une animation `loop: true`, puisqu'elle ne se termine jamais. Si votre callback ne se déclenche pas, vérifiez d'abord que l'animation concernée a bien été construite avec `loop: false`.

---

## 29.22 — `SpriteSheet` et `createAnimation`

`SpriteSheet` est une couche de confort au-dessus d'une image découpée en grille régulière. Elle calcule les lignes et les colonnes à votre place.

Constructeurs réels :

```dart
SpriteSheet({
  required Image image,
  required Vector2 srcSize,
  double margin = 0,
  double spacing = 0,
});

SpriteSheet.fromColumnsAndRows({
  required Image image,
  required int columns,
  required int rows,
  double spacing = 0,
  double margin = 0,
});
```

| Paramètre | Rôle |
| --- | --- |
| `image` | la planche |
| `srcSize` | taille d'une case |
| `margin` | marge autour de la grille, en pixels |
| `spacing` | espace entre deux cases, en pixels |

Le calcul du nombre de colonnes, tiré du code source :

```text
  columns = (image.width  - 2 * margin + spacing) ~/ (srcSize.x + spacing)
  rows    = (image.height - 2 * margin + spacing) ~/ (srcSize.y + spacing)
```

Les paramètres `margin` et `spacing` répondent au problème de *texture bleeding* du chapitre 22 : de nombreuses planches téléchargées séparent les cases d'un ou deux pixels transparents.

```text
  spacing = 2, margin = 4

  ┌──────────────────────────────────────────┐
  │    ← margin 4                            │
  │    ┌────┐  ┌────┐  ┌────┐                │
  │    │ 0  │  │ 1  │  │ 2  │                │
  │    └────┘  └────┘  └────┘                │
  │          ↑ spacing 2                     │
  └──────────────────────────────────────────┘
```

### Les méthodes

```dart
Sprite getSprite(int row, int column);
Sprite getSpriteById(int spriteId);

SpriteAnimation createAnimation({
  required int row,
  required double stepTime,
  bool loop = true,
  int from = 0,
  int? to,
});

SpriteAnimation createAnimationWithVariableStepTimes({
  required int row,
  required List<double> stepTimes,
  bool loop = true,
  int from = 0,
  int? to,
});

SpriteAnimationFrameData createFrameData(int row, int column, {required double stepTime});
SpriteAnimationFrameData createFrameDataFromId(int spriteId, {required double stepTime});
```

`createAnimation` est la méthode vedette. Elle construit l'animation d'une **ligne entière**, ou d'une portion de ligne :

```dart
final planche = Flame.images.fromCache('heros.png');

final feuille = SpriteSheet(
  image: planche,
  srcSize: Vector2.all(32),
);

// Ligne 0 en entier (6 frames de notre planche) :
final idleComplet = feuille.createAnimation(row: 0, stepTime: 0.18);

// Ligne 0, colonnes 0 à 3 seulement (4 frames) :
final idle = feuille.createAnimation(row: 0, stepTime: 0.18, from: 0, to: 4);

// Ligne 1 en entier, la marche :
final marche = feuille.createAnimation(row: 1, stepTime: 0.09);

// Ligne 2, colonnes 0 à 3, une seule fois :
final attaque = feuille.createAnimation(
  row: 2,
  stepTime: 0.08,
  from: 0,
  to: 4,
  loop: false,
);
```

> **Sémantique de `from` et `to` :** `from` est inclus, `to` est **exclu**. `from: 0, to: 4` donne donc les colonnes 0, 1, 2 et 3, soit quatre frames. C'est la même convention que `sublist` du chapitre 6.

Par défaut, `from` vaut 0 et `to` vaut le nombre de colonnes : l'animation couvre toute la ligne.

### Ranger des cases éparpillées

Quand les frames d'une animation ne sont pas contiguës, on compose la liste soi-même avec `createFrameData` :

```dart
final animationCousue = SpriteAnimation.fromFrameData(
  planche,
  SpriteAnimationData([
    feuille.createFrameDataFromId(1, stepTime: 0.1),
    feuille.createFrameData(2, 3, stepTime: 0.3),   // ligne 2, colonne 3
    feuille.createFrameDataFromId(4, stepTime: 0.1),
  ]),
);
```

### `SpriteSheet` ou `SpriteAnimationData.sequenced` ?

| Situation | Choix |
| --- | --- |
| Une planche régulière, une animation par ligne | `SpriteSheet.createAnimation` — le plus lisible |
| Une planche avec `spacing` ou `margin` | `SpriteSheet` — il fait les calculs |
| Une animation qui déborde sur plusieurs lignes | `SpriteAnimationData.sequenced` avec `amountPerRow` |
| Des durées différentes par frame | `createAnimationWithVariableStepTimes` ou `SpriteAnimationData.variable` |
| Des frames dispersées dans la planche | `SpriteAnimationData` + `createFrameData` |
| Récupérer un sprite fixe dans une planche | `feuille.getSpriteById(7)` |

> **Attention à un exemple périmé.** La documentation en ligne montre parfois `spriteSheet.createAnimation(0, stepTime: 0.1)`, avec la ligne en argument positionnel. En 1.38.0, le paramètre `row` est **nommé et requis** : écrivez `createAnimation(row: 0, stepTime: 0.1)`.

---

## 29.23 — Le pixel art net : `FilterQuality` et le zoom entier

Une case de 16 × 16 pixels affichée en 64 × 64 est agrandie quatre fois. Comment Flutter fabrique-t-il les pixels manquants ? C'est le rôle du **filtrage**, déjà rencontré au chapitre 22.

```text
  Pixel source          FilterQuality.low         FilterQuality.none
  (agrandi x4)          (interpolation)           (plus proche voisin)

    ██                    ▓▒░ dégradé               ████
                          flou entre les            ████
                          couleurs                  ████ franc
```

Par défaut, Flutter interpole. Sur une photo, c'est souhaitable. Sur du pixel art, c'est une catastrophe : le résultat est flou et le style est détruit.

### Le réglage

`FilterQuality` est une énumération de `dart:ui` : `none`, `low`, `medium`, `high`. Pour du pixel art, une seule valeur convient : `FilterQuality.none`.

Sur un `SpriteComponent` ou un `SpriteAnimationComponent`, on le règle via le `Paint` du composant :

```dart
final heros = SpriteAnimationComponent(
  animation: marche,
  size: Vector2.all(64),
  paint: Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false,
);
```

Ou après construction, puisque `paint` est accessible :

```dart
heros.paint.filterQuality = FilterQuality.none;
heros.paint.isAntiAlias = false;
```

`isAntiAlias = false` complète le réglage : l'anticrénelage lisse les bords, ce que l'on ne veut pas non plus.

Pour l'appliquer partout sans le répéter, on fabrique une fonction :

```dart
Paint peinturePixelArt() => Paint()
  ..filterQuality = FilterQuality.none
  ..isAntiAlias = false;
```

### Le zoom entier

Le filtrage ne suffit pas. Si vous agrandissez d'un facteur non entier — 2,37 par exemple — certains pixels source occupent 2 pixels écran et d'autres 3. Le personnage semble irrégulier, et les irrégularités **se déplacent** quand il bouge : c'est le scintillement.

```text
  Zoom x3 (entier)                   Zoom x2,5 (non entier)
  ┌───┬───┬───┐                      ┌──┬───┬──┬───┐
  │███│███│███│  chaque pixel        │██│███│██│███│  largeurs
  │███│███│███│  source = 3x3        │██│███│██│███│  inégales :
  │███│███│███│  pixels écran        └──┴───┴──┴───┘  ça scintille
  └───┴───┴───┘
```

**Règle : le facteur d'agrandissement doit être un entier.** Deux façons de le garantir.

**Façon 1 — une taille multiple de la taille source.**

```dart
const double zoomPixel = 4;
final srcTaille = Vector2.all(16);

SpriteComponent(
  sprite: Sprite(planche, srcSize: srcTaille),
  size: srcTaille * zoomPixel,     // 64 x 64 : facteur exactement 4
  paint: peinturePixelArt(),
);
```

**Façon 2 — une caméra à zoom entier.** C'est la méthode propre : on travaille en unités de « pixels du jeu » et la caméra fait l'agrandissement une bonne fois pour toutes.

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder
      ..anchor = Anchor.topLeft
      ..zoom = 4;                  // entier : 3, 4, 5... jamais 3,7
  }
}
```

Avec `zoom = 4`, un sprite de `size: Vector2.all(16)` occupe 64 pixels à l'écran, et toutes vos coordonnées de jeu restent de petits nombres entiers. C'est beaucoup plus confortable pour concevoir un niveau.

L'alternative, si vous voulez une résolution logique fixe quel que soit l'écran :

```dart
class DonjonDeDart extends FlameGame {
  DonjonDeDart()
      : super(
          camera: CameraComponent.withFixedResolution(
            width: 320,
            height: 180,
          ),
        );
}
```

Le zoom est alors calculé par Flame en fonction de la taille de la fenêtre. Il ne sera pas forcément entier, mais l'image entière est cohérente, ce qui est déjà un net progrès.

### Récapitulatif

| Symptôme | Cause | Correctif |
| --- | --- | --- |
| Sprite flou, contours baveux | filtrage par interpolation | `paint.filterQuality = FilterQuality.none` |
| Bords légèrement translucides | anticrénelage | `paint.isAntiAlias = false` |
| Pixels de largeur inégale | zoom non entier | taille multiple de la source, ou `zoom` entier |
| Scintillement pendant les déplacements | zoom non entier + position fractionnaire | zoom entier ; éventuellement arrondir la position au pixel |
| Lignes parasites entre les tuiles | *texture bleeding* | `spacing` dans `SpriteSheet`, ou paramètre `bleed` au rendu |

---

## 29.24 — Où trouver des assets libres et lire une licence

Ce cours ne fournit pas d'images, mais l'internet en regorge. Encore faut-il avoir le droit de s'en servir.

### Trois sources sérieuses

| Site | Contenu | Licence habituelle |
| --- | --- | --- |
| **kenney.nl** | packs très complets et cohérents (personnages, tuiles, interface, sons) | CC0 : domaine public, aucune contrainte |
| **itch.io** (rubrique *Game assets*) | packs gratuits et payants, styles très variés | variable, indiquée pack par pack |
| **opengameart.org** | vaste dépôt communautaire | variable : CC0, CC-BY, CC-BY-SA, GPL |

Deux autres, utiles pour compléter : **craftpix.net** (une section gratuite) et **fontstruct.org** ou **dafont.com** pour les polices, en vérifiant là aussi la licence.

### Lire une licence en trois questions

Avant de télécharger, répondez à ces trois questions. Elles suffisent dans 95 % des cas.

```text
  1. Puis-je l'utiliser dans un projet COMMERCIAL ?
  2. Dois-je CITER l'auteur ?
  3. Suis-je obligé de publier mon jeu sous la MÊME licence ?
```

| Licence | Usage commercial | Attribution | Contamination |
| --- | --- | --- | --- |
| **CC0** | oui | non exigée (mais élégante) | non |
| **CC-BY** | oui | **obligatoire** | non |
| **CC-BY-SA** | oui | obligatoire | **oui** : votre œuvre dérivée doit être en CC-BY-SA |
| **CC-BY-NC** | **non** | obligatoire | non |
| **GPL** | oui | obligatoire | **oui**, y compris sur le code dans certains montages |
| **MIT / Apache 2.0** | oui | obligatoire (mention du copyright) | non |
| « Free for personal use » | **non** | variable | — |
| Aucune licence indiquée | **considérez que c'est non** | — | — |

Deux pièges reviennent constamment.

**Piège 1 — « gratuit » ne veut pas dire « libre ».** Un pack téléchargeable sans payer peut interdire l'usage commercial. Lisez le fichier `license.txt` ou `readme.txt` fourni dans l'archive : il fait foi.

**Piège 2 — le « NC » vous rattrape plus tard.** Un jeu publié gratuitement mais contenant des publicités est considéré comme commercial. Si vous envisagez un jour de publier, écartez d'emblée les licences non commerciales.

### Créditer proprement

Même quand ce n'est pas obligatoire, un fichier `CREDITS.md` à la racine du projet vous protège et rend service.

```text
  # Crédits

  ## Graphismes
  - « Tiny Dungeon » par Kenney (kenney.nl) — CC0
  - « Goblin Pack » par NomDeLAuteur (opengameart.org/content/xxxx) — CC-BY 3.0

  ## Polices
  - « Press Start 2P » par CodeMan38 — SIL Open Font License 1.1
```

Notez **au moment du téléchargement**, pas six mois plus tard. Retrouver l'auteur d'un PNG anonyme dans un dossier `assets/` est une tâche décourageante.

> **Conseil pratique.** Pour un premier jeu, prenez **un seul pack, chez un seul auteur**. Mélanger trois styles de pixel art donne un résultat visuellement incohérent, même si chaque asset est joli pris isolément. Kenney est idéal pour cela : ses packs partagent une palette et une taille de tuile.

---

## 29.25 — Organiser ses assets

Un projet de jeu accumule vite deux cents fichiers. Trois décisions prises au début vous éviteront un grand désordre.

### Décision 1 — le nommage

Adoptez une convention et n'en changez jamais.

```text
  BON                              MAUVAIS
  heros_idle.png                   Heros Idle FINAL (2).png
  heros_marche.png                 herosMarcheV3.png
  gobelin_idle.png                 gob.png
  ui_bouton.png                    bouton nouveau.PNG
  decor/mur_pierre.png             mur3.png
```

Les règles :

- **minuscules uniquement** ; certains systèmes de fichiers distinguent la casse, d'autres non, et un jeu qui marche sur votre machine peut échouer à la compilation Android ;
- **pas d'espaces ni d'accents** dans les noms de fichiers ;
- **le tiret bas** comme séparateur ;
- **du général au particulier** : `heros_attaque_1.png`, pas `attaque_heros_1.png`. Le tri alphabétique regroupe alors tout ce qui concerne le héros.

### Décision 2 — les tailles

Fixez une **taille de tuile** dès le premier jour et faites-en la base de tout.

| Taille de tuile | Style | Remarque |
| --- | --- | --- |
| 8 × 8 | rétro extrême | très peu de détail, très rapide à dessiner |
| 16 × 16 | pixel art classique | le meilleur compromis pour un premier jeu |
| 32 × 32 | pixel art détaillé | demande un vrai travail de dessin |
| 64 × 64 et plus | illustration | ce n'est plus du pixel art |

Un personnage peut être plus grand qu'une tuile : un héros de 16 × 24 sur une grille de 16 est très courant. Ce qui compte, c'est que la **grille du décor** soit constante.

### Décision 3 — les atlas

Un **atlas** (ou *sprite sheet*) regroupe plusieurs images dans un seul fichier. Le chapitre 22 en a exposé la raison : moins de fichiers à charger, moins de changements de texture au rendu, donc de meilleures performances.

```text
  DISPERSÉ                          REGROUPÉ EN ATLAS
  heros_idle_0.png                  heros.png  (une planche 6 x 4)
  heros_idle_1.png                    ligne 0 : idle
  heros_idle_2.png                    ligne 1 : marche
  heros_idle_3.png                    ligne 2 : attaque
  heros_marche_0.png                  ligne 3 : mort
  ... 20 fichiers ...
  = 20 chargements                  = 1 chargement
```

Le regroupement obéit à une logique simple : **un atlas par entité, ou par thème**.

```text
  assets/images/
  ├── heros.png          (toutes les animations du héros)
  ├── gobelin.png        (toutes les animations du gobelin)
  ├── boss.png
  ├── objets.png         (potion, clé, coffre, pièce)
  ├── ui.png             (boutons, cadres, icônes de vie)
  └── decor/
      ├── tuiles.png     (murs, sols, portes)
      └── fond.png       (couches de parallaxe)
```

Ne fabriquez **pas** un atlas géant unique. Il serait chargé en entier même pour afficher le menu, et un changement d'un seul sprite obligerait à tout réexporter.

### Petit tableau de décision

| Question | Réponse |
| --- | --- |
| Un fichier ou un atlas ? | atlas dès que deux images partagent un usage |
| Quelle taille de case ? | une seule par atlas, constante |
| Une animation par ligne ? | oui : `SpriteSheet.createAnimation(row: ...)` devient trivial |
| Des cases vides en fin de ligne ? | acceptable ; utilisez `from` et `to` |
| Espacer les cases ? | 1 ou 2 pixels si vous constatez du *bleeding* ; renseignez `spacing` |
| Où ranger les sous-dossiers ? | par thème, et déclarez-les tous dans `pubspec.yaml` |

---

## 29.26 — Solution de repli sans image : générer une sprite sheet en mémoire

Voici la section centrale du chapitre pour qui ne possède aucun fichier. Nous allons **fabriquer une véritable sprite sheet** en dessinant avec un `Canvas`, puis l'injecter dans le cache de Flame sous un nom de fichier fictif. Tout le reste du chapitre s'appliquera sans changer une ligne.

### Le principe

```text
  PictureRecorder      Canvas             Picture           ui.Image
  ┌────────────┐   ┌──────────────┐   ┌───────────┐   ┌──────────────┐
  │ enregistre │──>│ drawRect     │──>│ endRecor- │──>│ toImage(w,h) │
  │ les ordres │   │ drawCircle   │   │  ding()   │   │              │
  └────────────┘   └──────────────┘   └───────────┘   └──────┬───────┘
                                                             │
                                          Flame.images.add('heros.png', img)
                                                             │
                                                             ▼
                                          le cache contient 'heros.png'
```

C'est exactement le mécanisme du chapitre 22, section 22.8 bis, auquel on ajoute une seule étape : l'injection dans le cache de Flame.

### L'atelier d'images

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

/// Fabrique un ui.Image de [largeur] x [hauteur] pixels en exécutant [dessin].
Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}
```

Le paramètre `dessin` est une fonction passée en argument : une fonction d'ordre supérieur du chapitre 7. Tout ce qui n'est pas dessiné reste **transparent**, comme dans un PNG bien exporté.

### La planche du héros, ligne par ligne

Nous construisons une planche de **6 colonnes × 3 lignes**, cases de 32 × 32, soit 192 × 96 pixels.

```text
                colonne 0     1       2       3       4       5
              ┌───────┬───────┬───────┬───────┬───────┬───────┐
  ligne 0     │ idle  │ idle  │ idle  │ idle  │ vide  │ vide  │
  IDLE        │   0   │   1   │   2   │   3   │       │       │
              ├───────┼───────┼───────┼───────┼───────┼───────┤
  ligne 1     │ mar-  │ mar-  │ mar-  │ mar-  │ mar-  │ mar-  │
  MARCHE      │ che 0 │ che 1 │ che 2 │ che 3 │ che 4 │ che 5 │
              ├───────┼───────┼───────┼───────┼───────┼───────┤
  ligne 2     │ att.  │ att.  │ att.  │ att.  │ vide  │ vide  │
  ATTAQUE     │   0   │   1   │   2   │   3   │       │       │
              └───────┴───────┴───────┴───────┴───────┴───────┘

  idle    : le héros respire, il monte et descend de 1 pixel
  marche  : les jambes s'écartent, le corps oscille
  attaque : l'épée se lève puis s'abat
```

### Le code complet du générateur

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

const double kCase = 32;      // taille d'une case de la planche
const int kColonnes = 6;
const int kLignes = 3;

// Palette du Donjon de Dart.
const Color kPeau = Color(0xFFF0C89A);
const Color kTunique = Color(0xFF3E7CB1);
const Color kCeinture = Color(0xFF6B4423);
const Color kJambes = Color(0xFF2C4A6B);
const Color kEpee = Color(0xFFD8D8E0);
const Color kCheveux = Color(0xFF6B4423);

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

/// Dessine un héros dans une case, en coordonnées LOCALES (0..32).
/// [oscillation] décale le corps verticalement.
/// [ecartJambes] écarte les pieds.
/// [angleEpee] positionne l'épée : 0 = au repos, 1 = levée, 2 = abattue.
void dessinerHeros(
  Canvas canvas, {
  required double oscillation,
  required double ecartJambes,
  required int poseEpee,
}) {
  final peau = Paint()..color = kPeau;
  final tunique = Paint()..color = kTunique;
  final ceinture = Paint()..color = kCeinture;
  final jambes = Paint()..color = kJambes;
  final epee = Paint()..color = kEpee;
  final cheveux = Paint()..color = kCheveux;

  final double y = oscillation;

  // Jambes
  canvas.drawRect(Rect.fromLTWH(11 - ecartJambes, 24, 4, 6), jambes);
  canvas.drawRect(Rect.fromLTWH(17 + ecartJambes, 24, 4, 6), jambes);

  // Corps
  canvas.drawRect(Rect.fromLTWH(10, 14 + y, 12, 11), tunique);
  canvas.drawRect(Rect.fromLTWH(10, 21 + y, 12, 2), ceinture);

  // Tête
  canvas.drawRect(Rect.fromLTWH(11, 5 + y, 10, 9), peau);
  canvas.drawRect(Rect.fromLTWH(11, 4 + y, 10, 3), cheveux);

  // Yeux
  final noir = Paint()..color = const Color(0xFF201820);
  canvas.drawRect(Rect.fromLTWH(13, 9 + y, 2, 2), noir);
  canvas.drawRect(Rect.fromLTWH(17, 9 + y, 2, 2), noir);

  // Épée, selon la pose
  switch (poseEpee) {
    case 1: // levée au-dessus de la tête
      canvas.drawRect(Rect.fromLTWH(22, 1 + y, 3, 14), epee);
      break;
    case 2: // abattue devant
      canvas.drawRect(Rect.fromLTWH(22, 15 + y, 9, 3), epee);
      break;
    default: // au repos, le long du corps
      canvas.drawRect(Rect.fromLTWH(22, 14 + y, 3, 11), epee);
  }
}

/// Construit la planche complète : 6 x 3 cases de 32 x 32.
Future<ui.Image> creerPlancheHeros() {
  return imageDepuisDessin(
    (kCase * kColonnes).toInt(),
    (kCase * kLignes).toInt(),
    (Canvas canvas) {
      // --- Ligne 0 : IDLE (4 frames) ---
      const oscillationsIdle = [0.0, 1.0, 0.0, -1.0];
      for (int i = 0; i < 4; i++) {
        canvas.save();
        canvas.translate(i * kCase, 0);
        dessinerHeros(
          canvas,
          oscillation: oscillationsIdle[i],
          ecartJambes: 0,
          poseEpee: 0,
        );
        canvas.restore();
      }

      // --- Ligne 1 : MARCHE (6 frames) ---
      const ecarts = [0.0, 2.0, 3.0, 0.0, -2.0, -3.0];
      const oscillationsMarche = [0.0, -1.0, 0.0, 0.0, -1.0, 0.0];
      for (int i = 0; i < 6; i++) {
        canvas.save();
        canvas.translate(i * kCase, kCase);
        dessinerHeros(
          canvas,
          oscillation: oscillationsMarche[i],
          ecartJambes: ecarts[i],
          poseEpee: 0,
        );
        canvas.restore();
      }

      // --- Ligne 2 : ATTAQUE (4 frames) ---
      const posesEpee = [0, 1, 2, 2];
      for (int i = 0; i < 4; i++) {
        canvas.save();
        canvas.translate(i * kCase, kCase * 2);
        dessinerHeros(
          canvas,
          oscillation: 0,
          ecartJambes: i >= 2 ? 2 : 0,
          poseEpee: posesEpee[i],
        );
        canvas.restore();
      }
    },
  );
}
```

`canvas.save()` et `canvas.restore()` encadrent la translation vers la case courante : c'est la technique du chapitre 21. Chaque case est ensuite dessinée en coordonnées locales de 0 à 32, ce qui rend la fonction `dessinerHeros` totalement indépendante de sa position dans la planche.

### L'injection dans le cache de Flame

```dart
import 'package:flame/flame.dart';

// Méthode 1 : add, quand vous avez déjà l'image.
final planche = await creerPlancheHeros();
Flame.images.add('heros.png', planche);

// Méthode 2 : fetchOrGenerate, qui ne génère qu'une fois. Préférez celle-ci.
await Flame.images.fetchOrGenerate('heros.png', creerPlancheHeros);
```

`fetchOrGenerate` est la meilleure option : si la clé existe déjà, votre générateur n'est pas rappelé. Rappel de son implémentation réelle :

```dart
Future<Image> fetchOrGenerate(
  String name,
  Future<Image> Function() imageGenerator,
) {
  return (_assets[name] ??= _ImageAsset.future(
    imageGenerator(),
  )).retrieveAsync();
}
```

### Programme complet et exécutable

Assemblons le tout : une planche générée, une `SpriteSheet`, trois animations, un héros qui alterne automatiquement entre elles.

```dart
// (reprendre les constantes, imageDepuisDessin, dessinerHeros
//  et creerPlancheHeros de la section précédente)

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flame/sprite.dart';

void main() {
  runApp(GameWidget(game: DemoPlanche()));
}

class DemoPlanche extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF181420);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('heros.png', creerPlancheHeros);
    final feuille = SpriteSheet(
      image: Flame.images.fromCache('heros.png'),
      srcSize: Vector2.all(kCase),
    );

    // Trois animations, une par ligne.
    final animations = <String, SpriteAnimation>{
      'idle': feuille.createAnimation(row: 0, stepTime: 0.20, from: 0, to: 4),
      'marche': feuille.createAnimation(row: 1, stepTime: 0.10),
      'attaque': feuille.createAnimation(
        row: 2,
        stepTime: 0.09,
        from: 0,
        to: 4,
        loop: false,
      ),
    };

    // On les affiche côte à côte pour comparer.
    int i = 0;
    for (final entree in animations.entries) {
      await world.add(
        SpriteAnimationComponent(
          animation: entree.value,
          position: Vector2(60 + i * 100.0, 120),
          size: Vector2.all(96),
          anchor: Anchor.center,
          paint: Paint()
            ..filterQuality = FilterQuality.none
            ..isAntiAlias = false,
        ),
      );
      await world.add(
        TextComponent(
          text: entree.key,
          position: Vector2(60 + i * 100.0, 190),
          anchor: Anchor.topCenter,
          textRenderer: TextPaint(
            style: const TextStyle(fontSize: 14, color: Color(0xFFE8E8F0)),
          ),
        ),
      );
      i++;
    }
  }
}
```

**Résultat :**

```text
  Trois héros de 96 x 96 alignés, chacun sous son libellé :
    - « idle »    : il respire, boucle indéfiniment
    - « marche »  : ses jambes bougent, boucle indéfiniment
    - « attaque » : il lève puis abat son épée, et reste sur la dernière frame
  Aucun fichier PNG n'est présent dans le projet.
```

> **Le jour où vous aurez de vrais fichiers.** Remplacez la ligne
> `await Flame.images.fetchOrGenerate('heros.png', creerPlancheHeros);`
> par `await Flame.images.load('heros.png');`
> et supprimez le générateur. **Rien d'autre ne change.** C'est tout l'intérêt d'être passé par le cache.

---

## 29.27 — `NineTileBoxComponent` pour les cadres d'interface

Un cadre de boîte de dialogue doit s'adapter au texte qu'il contient. L'étirer déforme ses coins et ses bordures. La technique du **neuf-tuiles** résout le problème depuis trente ans.

```text
  Sprite source : une grille 3 x 3          Rendu à n'importe quelle taille
  ┌────┬────┬────┐                          ┌───┬─────────────────┬───┐
  │ HG │ H  │ HD │   HG = haut-gauche       │ HG│  H  H  H  H  H  │HD │
  ├────┼────┼────┤   H  = haut              ├───┼─────────────────┼───┤
  │ G  │ C  │ D  │   C  = centre            │ G │                 │ D │
  ├────┼────┼────┤                          │ G │   C  C  C  C    │ D │
  │ BG │ B  │ BD │                          │ G │                 │ D │
  └────┴────┴────┘                          ├───┼─────────────────┼───┤
                                            │ BG│  B  B  B  B  B  │BD │
  Les 4 COINS ne sont jamais étirés.        └───┴─────────────────┴───┘
  Les 4 CÔTÉS sont étirés dans UN sens.
  Le CENTRE est étiré dans les deux.
```

Flame fournit deux classes.

**`NineTileBox`**, l'objet de dessin :

```dart
NineTileBox(Sprite sprite, {int? tileSize, int? destTileSize});

NineTileBox.withGrid(
  Sprite sprite, {
  double leftWidth = 0.0,
  double rightWidth = 0.0,
  double topHeight = 0.0,
  double bottomHeight = 0.0,
});
```

| Membre | Rôle |
| --- | --- |
| `sprite` | le sprite source, qui **doit** être une grille 3 × 3 de tuiles carrées |
| `tileSize` | taille d'une tuile dans l'image source |
| `destTileSize` | taille de la tuile au rendu, pour agrandir le cadre |
| `setGrid({leftWidth, rightWidth, topHeight, bottomHeight})` | définit des tailles différentes par bord |

**`NineTileBoxComponent`**, le composant :

```dart
NineTileBoxComponent({
  NineTileBox? nineTileBox,
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

Exemple complet, avec un cadre généré en code : une image de 24 × 24, soit neuf tuiles de 8 × 8.

```dart
/// Un cadre de parchemin 24 x 24 : 9 tuiles de 8 x 8.
Future<ui.Image> creerCadre() {
  const double t = 8;
  return imageDepuisDessin(24, 24, (Canvas canvas) {
    final bordure = Paint()..color = const Color(0xFF8B6F3E);
    final interieur = Paint()..color = const Color(0xFF241E2C);
    final liseret = Paint()..color = const Color(0xFFD8B45C);

    // Fond général : la bordure.
    canvas.drawRect(const Rect.fromLTWH(0, 0, 24, 24), bordure);
    // Liseré clair d'un pixel.
    canvas.drawRect(const Rect.fromLTWH(1, 1, 22, 22), liseret);
    canvas.drawRect(const Rect.fromLTWH(2, 2, 20, 20), bordure);
    // Tuile centrale : l'intérieur du cadre.
    canvas.drawRect(const Rect.fromLTWH(t, t, t, t), interieur);
  });
}
```

Et son utilisation :

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();
  camera.viewfinder.anchor = Anchor.topLeft;

  await Flame.images.fetchOrGenerate('cadre.png', creerCadre);

  final boite = NineTileBoxComponent(
    nineTileBox: NineTileBox(
      Sprite(Flame.images.fromCache('cadre.png')),
      tileSize: 8,
      destTileSize: 24,     // les tuiles sont agrandies x3
    ),
    position: Vector2(20, 20),
    size: Vector2(260, 120),   // n'importe quelle taille : le cadre s'adapte
  );

  await camera.viewport.add(boite);

  await boite.add(
    TextBoxComponent(
      text: 'Vous entrez dans le Donjon de Dart. '
          'Le gobelin vous a repéré.',
      position: Vector2(28, 28),
      boxConfig: const TextBoxConfig(maxWidth: 210),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 13, color: Color(0xFFE8E8F0)),
      ),
    ),
  );
}
```

**Résultat :**

```text
  Un cadre doré de 260 x 120 pixels dans le coin haut-gauche, avec
  ses coins nets et son intérieur sombre, contenant deux lignes de texte.
  Changer size en Vector2(400, 200) donne un cadre plus grand
  SANS déformer les coins.
```

> **Deux contraintes à respecter.** Le sprite source doit être une grille **3 × 3 de tuiles carrées** : une image de 24 × 24 pour des tuiles de 8, de 48 × 48 pour des tuiles de 16. Et le composant est ajouté au `camera.viewport` (chapitre 31) pour rester fixe à l'écran : un cadre d'interface ne doit pas défiler avec le monde.

---

## 29.28 — `ParallaxComponent` pour les fonds

Le chapitre 25 a posé le principe : plusieurs couches de fond défilant à des vitesses différentes créent une illusion de profondeur. Les couches lointaines bougent lentement, les proches vite.

```text
  Caméra qui se déplace vers la droite ->

  couche 3 (ciel)        vitesse x1     ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
  couche 2 (montagnes)   vitesse x1,8   ▁▁/\▁▁▁/\▁▁▁▁/\▁▁▁
  couche 1 (arbres)      vitesse x3,2   ▁♣▁♣▁▁♣▁♣▁▁♣▁♣▁▁♣▁
  couche 0 (sol)         vitesse x5,8   ══════════════════
```

Flame fournit `ParallaxComponent` et, sur `FlameGame`, une méthode d'aide qui construit tout.

```dart
Future<ParallaxComponent<FlameGame<World>>> loadParallaxComponent(
  Iterable<ParallaxData> dataList, {
  Vector2? baseVelocity,
  Vector2? velocityMultiplierDelta,
  ImageRepeat repeat = ImageRepeat.repeatX,
  Alignment alignment = Alignment.bottomLeft,
  LayerFill fill = LayerFill.height,
  Images? images,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  Anchor? anchor,
  int? priority,
  FilterQuality? filterQuality,
  ComponentKey? key,
  String? package,
});
```

| Paramètre | Rôle |
| --- | --- |
| `dataList` | la liste des couches, de la plus **lointaine** à la plus **proche** |
| `baseVelocity` | vitesse de défilement de la première couche, en pixels par seconde |
| `velocityMultiplierDelta` | facteur multiplicatif appliqué d'une couche à la suivante |
| `repeat` | `ImageRepeat.repeatX`, `repeatY`, `repeat` ou `noRepeat` |
| `alignment` | alignement de l'image dans la couche |
| `fill` | `LayerFill.height`, `width` ou `none` |
| `filterQuality` | filtrage, à mettre à `FilterQuality.none` pour du pixel art |

Le comportement par défaut, indiqué par la documentation : les images sont alignées en bas à gauche, répétées sur l'axe X, et mises à l'échelle proportionnellement pour couvrir la hauteur de l'écran.

Exemple complet, avec des couches générées en code :

```dart
// Nécessite : import 'dart:math' as math;
Future<ui.Image> creerCouche(Color couleur, double hauteurRelative, int graine) {
  final rnd = math.Random(graine);
  return imageDepuisDessin(160, 90, (Canvas canvas) {
    final p = Paint()..color = couleur;
    double x = 0;
    while (x < 160) {
      final double l = 12 + rnd.nextDouble() * 20;
      final double h = 90 * hauteurRelative * (0.6 + rnd.nextDouble() * 0.4);
      canvas.drawRect(Rect.fromLTWH(x, 90 - h, l, h), p);
      x += l;
    }
  });
}

class JeuParallaxe extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await Flame.images.fetchOrGenerate(
      'fond_loin.png',
      () => creerCouche(const Color(0xFF2A2440), 0.55, 1),
    );
    await Flame.images.fetchOrGenerate(
      'fond_moyen.png',
      () => creerCouche(const Color(0xFF3A3358), 0.40, 2),
    );
    await Flame.images.fetchOrGenerate(
      'fond_proche.png',
      () => creerCouche(const Color(0xFF4C4470), 0.25, 3),
    );

    final fond = await loadParallaxComponent(
      [
        ParallaxImageData('fond_loin.png'),
        ParallaxImageData('fond_moyen.png'),
        ParallaxImageData('fond_proche.png'),
      ],
      baseVelocity: Vector2(14, 0),
      velocityMultiplierDelta: Vector2(2.0, 1.0),
      filterQuality: FilterQuality.none,
    );

    // Le fond est statique par rapport à l'écran : on l'ajoute au backdrop.
    camera.backdrop.add(fond);
  }
}
```

**Résultat :**

```text
  Trois rangées de créneaux violets défilant vers la gauche,
  la plus foncée lentement, la plus claire quatre fois plus vite.
```

Modification en cours de partie, par exemple quand le héros accélère :

```dart
fond.parallax?.baseVelocity = Vector2(60, 0);
```

> **Où ajouter le fond ?** Trois emplacements, trois comportements.
>
> | Ajouté à | Comportement |
> | --- | --- |
> | `camera.backdrop` | fixe par rapport à l'écran, dessiné derrière le monde — le choix normal |
> | `world` | fait partie du monde, donc défile avec la caméra ; le parallaxe fait alors doublon |
> | `camera.viewport` | fixe à l'écran, mais **devant** le monde : réservé au HUD |

---

## 29.29 — Le préchargement global et l'écran de chargement

Le chargement des images prend du temps. Sur un téléphone modeste, une trentaine d'images peut demander une seconde. Pendant ce temps, l'utilisateur ne doit pas voir un écran noir sans explication.

### Le point de départ : `loadingBuilder`

`GameWidget` accepte un constructeur de widget affiché tant que le `onLoad` du jeu n'est pas terminé.

```dart
GameWidget<DonjonDeDart>(
  game: DonjonDeDart(),
  loadingBuilder: (BuildContext context) {
    return const ColoredBox(
      color: Color(0xFF101018),
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            CircularProgressIndicator(color: Color(0xFFE8B04B)),
            SizedBox(height: 16),
            Text(
              'Chargement du donjon...',
              style: TextStyle(color: Color(0xFFE8E8F0)),
            ),
          ],
        ),
      ),
    );
  },
  errorBuilder: (BuildContext context, Object erreur) {
    return ColoredBox(
      color: const Color(0xFF3A1020),
      child: Center(
        child: Text(
          'Erreur de chargement : $erreur',
          style: const TextStyle(color: Color(0xFFFFD0D0)),
        ),
      ),
    );
  },
);
```

`errorBuilder` mérite d'être renseigné dès le premier jour : sans lui, un asset manquant produit un écran rouge peu lisible. Avec lui, vous lisez le nom du fichier fautif immédiatement.

### Centraliser la liste des assets

Écrire la liste des images en dur dans `onLoad` est un mauvais réflexe : elle finit dupliquée. Centralisez-la.

```dart
/// Toutes les images du jeu, en un seul endroit.
class Assets {
  const Assets._();

  static const String heros = 'heros.png';
  static const String gobelin = 'gobelin.png';
  static const String potion = 'potion.png';
  static const String cle = 'cle.png';
  static const String coffre = 'coffre.png';

  static const List<String> toutes = [heros, gobelin, potion, cle, coffre];
}
```

Le constructeur privé `Assets._()` empêche l'instanciation : c'est le patron « classe de constantes » du chapitre 16.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();
  await images.loadAll(Assets.toutes);
  await world.add(Heros());
}
```

Les constantes remplacent alors les chaînes disséminées :

```dart
sprite = Sprite(Flame.images.fromCache(Assets.gobelin));
```

Une faute de frappe devient une erreur de compilation au lieu d'un plantage à l'exécution. C'est exactement l'argument qui justifiait les enums au chapitre 11.

### Version « repli 100 % code »

Quand les images sont générées, la même centralisation s'applique, avec une table nom → générateur :

```dart
typedef GenerateurImage = Future<ui.Image> Function();

const Map<String, GenerateurImage> generateurs = {
  Assets.heros: creerPlancheHeros,
  Assets.gobelin: creerPlancheGobelin,
  Assets.potion: creerPotion,
};

Future<void> preparerLesImages() async {
  for (final entree in generateurs.entries) {
    await Flame.images.fetchOrGenerate(entree.key, entree.value);
  }
}
```

`typedef` pour un type de fonction vient du chapitre 7.

### Un écran de chargement avec progression

Pour afficher une barre de progression réelle, on charge les images une par une et on publie l'avancement. Un `ValueNotifier` (chapitre 19) suffit.

```dart
class DonjonDeDart extends FlameGame {
  final ValueNotifier<double> progression = ValueNotifier<double>(0);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    final noms = Assets.toutes;
    for (int i = 0; i < noms.length; i++) {
      await images.load(noms[i]);
      progression.value = (i + 1) / noms.length;
    }

    await world.add(Heros());
  }
}
```

Le widget associé :

```dart
ValueListenableBuilder<double>(
  valueListenable: jeu.progression,
  builder: (BuildContext context, double valeur, Widget? _) {
    return LinearProgressIndicator(value: valeur);
  },
);
```

> **Compromis à connaître.** Cette boucle charge les images **en série** : c'est plus lent qu'un `loadAll` parallèle. On l'accepte parce que la barre de progression a une valeur perçue supérieure à quelques dizaines de millisecondes gagnées. Pour un jeu à trois images, gardez `loadAll`.

### Récapitulatif des stratégies

| Taille du jeu | Stratégie |
| --- | --- |
| Prototype, moins de 10 images | `loadAll` dans `onLoad`, `loadingBuilder` simple |
| Jeu complet, un seul niveau | `loadAll` + écran de chargement soigné + `errorBuilder` |
| Jeu à plusieurs niveaux | chargement du commun au démarrage, du spécifique à l'entrée du niveau, `clear` à la sortie |
| Jeu web | limiter le poids total ; le chargement passe par le réseau |

---

## 29.30 — Mini-projet : le joueur du « Donjon de Dart »

Objectif : un héros complet, avec ses animations *idle*, *marche* et *attaque*, piloté au clavier, **sans aucun fichier image**. C'est la brique que les chapitres 30 et 36 reprendront.

### Cahier des charges

| Exigence | Moyen |
| --- | --- |
| Aucun asset externe | planche générée par `PictureRecorder`, injectée avec `fetchOrGenerate` |
| Trois animations | `SpriteSheet.createAnimation` sur les trois lignes |
| Changement d'état propre | `SpriteAnimationGroupComponent<EtatHeros>` |
| L'attaque ne s'interrompt pas | garde sur `current` + `onComplete` |
| Le héros regarde où il va | `scale.x = ±1` |
| Pixel art net | `FilterQuality.none` et zoom entier |
| Déplacement indépendant du framerate | `position += vitesse * dt` |

### La machine à états

```text
                 ┌──────────────────────────────────────┐
                 │                                      │
                 ▼                                      │
            ┌─────────┐   touche de direction   ┌─────────────┐
            │  IDLE   │ ──────────────────────> │   MARCHE    │
            │ boucle  │ <────────────────────── │   boucle    │
            └────┬────┘   plus aucune touche    └──────┬──────┘
                 │                                     │
                 │  ESPACE                     ESPACE  │
                 ▼                                     ▼
            ┌───────────────────────────────────────────────┐
            │                  ATTAQUE                      │
            │  loop: false — aucune interruption possible    │
            │  onComplete -> retour à IDLE                   │
            └───────────────────────────────────────────────┘
```

### Le code complet

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flame/sprite.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// ───────────────────────────────────────────────────────────────
// 1. ATELIER D'IMAGES : aucune ressource externe
// ───────────────────────────────────────────────────────────────

const double kCase = 32;
const int kColonnes = 6;
const int kLignes = 3;

const Color kPeau = Color(0xFFF0C89A);
const Color kTunique = Color(0xFF3E7CB1);
const Color kCeinture = Color(0xFF6B4423);
const Color kJambes = Color(0xFF2C4A6B);
const Color kEpee = Color(0xFFD8D8E0);
const Color kCheveux = Color(0xFF6B4423);
const Color kNoir = Color(0xFF201820);

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

void dessinerHeros(
  Canvas canvas, {
  required double oscillation,
  required double ecartJambes,
  required int poseEpee,
}) {
  final double y = oscillation;

  canvas.drawRect(
    Rect.fromLTWH(11 - ecartJambes, 24, 4, 6),
    Paint()..color = kJambes,
  );
  canvas.drawRect(
    Rect.fromLTWH(17 + ecartJambes, 24, 4, 6),
    Paint()..color = kJambes,
  );
  canvas.drawRect(
    Rect.fromLTWH(10, 14 + y, 12, 11),
    Paint()..color = kTunique,
  );
  canvas.drawRect(
    Rect.fromLTWH(10, 21 + y, 12, 2),
    Paint()..color = kCeinture,
  );
  canvas.drawRect(Rect.fromLTWH(11, 5 + y, 10, 9), Paint()..color = kPeau);
  canvas.drawRect(Rect.fromLTWH(11, 4 + y, 10, 3), Paint()..color = kCheveux);
  canvas.drawRect(Rect.fromLTWH(13, 9 + y, 2, 2), Paint()..color = kNoir);
  canvas.drawRect(Rect.fromLTWH(17, 9 + y, 2, 2), Paint()..color = kNoir);

  final epee = Paint()..color = kEpee;
  switch (poseEpee) {
    case 1:
      canvas.drawRect(Rect.fromLTWH(22, 1 + y, 3, 14), epee);
      break;
    case 2:
      canvas.drawRect(Rect.fromLTWH(22, 15 + y, 9, 3), epee);
      break;
    default:
      canvas.drawRect(Rect.fromLTWH(22, 14 + y, 3, 11), epee);
  }
}

Future<ui.Image> creerPlancheHeros() {
  return imageDepuisDessin(
    (kCase * kColonnes).toInt(),
    (kCase * kLignes).toInt(),
    (Canvas canvas) {
      const oscillationsIdle = [0.0, 1.0, 0.0, -1.0];
      for (int i = 0; i < 4; i++) {
        canvas.save();
        canvas.translate(i * kCase, 0);
        dessinerHeros(
          canvas,
          oscillation: oscillationsIdle[i],
          ecartJambes: 0,
          poseEpee: 0,
        );
        canvas.restore();
      }

      const ecarts = [0.0, 2.0, 3.0, 0.0, -2.0, -3.0];
      const oscMarche = [0.0, -1.0, 0.0, 0.0, -1.0, 0.0];
      for (int i = 0; i < 6; i++) {
        canvas.save();
        canvas.translate(i * kCase, kCase);
        dessinerHeros(
          canvas,
          oscillation: oscMarche[i],
          ecartJambes: ecarts[i],
          poseEpee: 0,
        );
        canvas.restore();
      }

      const poses = [0, 1, 2, 2];
      for (int i = 0; i < 4; i++) {
        canvas.save();
        canvas.translate(i * kCase, kCase * 2);
        dessinerHeros(
          canvas,
          oscillation: 0,
          ecartJambes: i >= 2 ? 2 : 0,
          poseEpee: poses[i],
        );
        canvas.restore();
      }
    },
  );
}

// ───────────────────────────────────────────────────────────────
// 2. LES ÉTATS (enum, chapitre 11)
// ───────────────────────────────────────────────────────────────

enum EtatHeros { idle, marche, attaque }

// ───────────────────────────────────────────────────────────────
// 3. LE HÉROS
// ───────────────────────────────────────────────────────────────

class Heros extends SpriteAnimationGroupComponent<EtatHeros>
    with HasGameReference<DonjonDeDart>, KeyboardHandler {
  Heros({required Vector2 position})
      : super(
          position: position,
          size: Vector2.all(kCase),
          anchor: Anchor.center,
          paint: Paint()
            ..filterQuality = FilterQuality.none
            ..isAntiAlias = false,
        );

  static const double vitesseMax = 60; // pixels du monde par seconde

  final Vector2 vitesse = Vector2.zero();
  bool _demandeAttaque = false;

  @override
  void onLoad() {
    final feuille = SpriteSheet(
      image: Flame.images.fromCache('heros.png'),
      srcSize: Vector2.all(kCase),
    );

    animations = {
      EtatHeros.idle:
          feuille.createAnimation(row: 0, stepTime: 0.20, from: 0, to: 4),
      EtatHeros.marche: feuille.createAnimation(row: 1, stepTime: 0.10),
      EtatHeros.attaque: feuille.createAnimation(
        row: 2,
        stepTime: 0.08,
        from: 0,
        to: 4,
        loop: false,
      ),
    };
    current = EtatHeros.idle;

    // Fin de l'attaque : on revient au repos.
    animationTickers?[EtatHeros.attaque]?.onComplete = () {
      current = EtatHeros.idle;
    };

    // Frame d'impact : c'est ici que les dégâts seront infligés (chapitre 32).
    animationTickers?[EtatHeros.attaque]?.onFrame = (int index) {
      if (index == 2) {
        game.messages.value = 'Coup d\'épée !';
      }
    };
  }

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    final double dx = (keysPressed.contains(LogicalKeyboardKey.arrowRight) ||
                keysPressed.contains(LogicalKeyboardKey.keyD)
            ? 1.0
            : 0.0) -
        (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ||
                keysPressed.contains(LogicalKeyboardKey.keyA)
            ? 1.0
            : 0.0);
    final double dy = (keysPressed.contains(LogicalKeyboardKey.arrowDown) ||
                keysPressed.contains(LogicalKeyboardKey.keyS)
            ? 1.0
            : 0.0) -
        (keysPressed.contains(LogicalKeyboardKey.arrowUp) ||
                keysPressed.contains(LogicalKeyboardKey.keyW)
            ? 1.0
            : 0.0);

    vitesse.setValues(dx, dy);
    if (vitesse.length2 > 0) {
      vitesse.normalize();
      vitesse.scale(vitesseMax);
    }

    if (keysPressed.contains(LogicalKeyboardKey.space)) {
      _demandeAttaque = true;
    }
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);

    // 1. L'attaque bloque tout : ni déplacement, ni changement d'état.
    if (current == EtatHeros.attaque) {
      _demandeAttaque = false;
      return;
    }

    // 2. Déclenchement d'une attaque.
    if (_demandeAttaque) {
      _demandeAttaque = false;
      animationTickers?[EtatHeros.attaque]?.reset();
      current = EtatHeros.attaque;
      return;
    }

    // 3. Déplacement, indépendant du framerate (chapitre 20).
    position += vitesse * dt;

    // 4. Orientation : idempotent, donc sans clignotement.
    if (vitesse.x != 0) {
      scale.x = vitesse.x < 0 ? -1 : 1;
    }

    // 5. État d'animation : une seule décision par frame.
    current = vitesse.length2 == 0 ? EtatHeros.idle : EtatHeros.marche;
  }
}

// ───────────────────────────────────────────────────────────────
// 4. LE JEU
// ───────────────────────────────────────────────────────────────

class DonjonDeDart extends FlameGame with HasKeyboardHandlerComponents {
  final ValueNotifier<String> messages = ValueNotifier<String>('');

  late final Heros heros;

  @override
  Color backgroundColor() => const Color(0xFF181420);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Zoom ENTIER : le pixel art reste net (section 29.23).
    camera.viewfinder.zoom = 4;

    await Flame.images.fetchOrGenerate('heros.png', creerPlancheHeros);

    heros = Heros(position: Vector2.zero());
    await world.add(heros);
    camera.follow(heros);

    await camera.viewport.add(
      TextComponent(
        text: 'Flèches ou ZQSD pour marcher, ESPACE pour attaquer',
        position: Vector2.all(12),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 13, color: Color(0xFF8A86A0)),
        ),
      ),
    );
  }
}

// ───────────────────────────────────────────────────────────────
// 5. POINT D'ENTRÉE
// ───────────────────────────────────────────────────────────────

void main() {
  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: GameWidget<DonjonDeDart>(
          game: DonjonDeDart(),
          loadingBuilder: (BuildContext context) => const ColoredBox(
            color: Color(0xFF181420),
            child: Center(
              child: Text(
                'Préparation du donjon...',
                style: TextStyle(color: Color(0xFFE8E8F0)),
              ),
            ),
          ),
          errorBuilder: (BuildContext context, Object erreur) => ColoredBox(
            color: const Color(0xFF3A1020),
            child: Center(
              child: Text(
                'Erreur : $erreur',
                style: const TextStyle(color: Color(0xFFFFD0D0)),
              ),
            ),
          ),
        ),
      ),
    ),
  );
}
```

**Résultat :**

```text
  Un héros de 32 unités affiché en x4, au centre de l'écran.
  - Sans touche : il respire (idle, 4 frames à 0,20 s).
  - Flèche gauche : il marche vers la gauche, sprite retourné.
  - Flèche droite : il marche vers la droite.
  - ESPACE : il lève et abat son épée ; pendant l'attaque
    les flèches sont ignorées ; à la fin il revient au repos.
  Aucun fichier image dans le projet.
```

### Ce que ce mini-projet démontre

| Point | Où le lire dans le code |
| --- | --- |
| Le cache accepte une image fabriquée en code | `fetchOrGenerate('heros.png', creerPlancheHeros)` |
| `SpriteSheet` lit une planche générée comme un vrai PNG | construction de `feuille` |
| Un enum pilote proprement les animations | `SpriteAnimationGroupComponent<EtatHeros>` |
| Une animation non bouclée signale sa fin | `animationTickers?[...]?.onComplete` |
| Une frame précise déclenche un événement | `onFrame` avec `index == 2` |
| Un état prioritaire n'est jamais interrompu | le `return` au début d'`update` |
| L'orientation est idempotente | `scale.x = vitesse.x < 0 ? -1 : 1` |
| Le pixel art reste net | `FilterQuality.none` + `zoom = 4` |

### Pistes d'extension

- Ajouter `EtatHeros.touche` avec un clignotement (`OpacityEffect`, chapitre 33).
- Ajouter `EtatHeros.mort` avec `removeOnFinish: {EtatHeros.mort: true}`.
- Générer une planche de gobelin et le faire patrouiller (chapitre 37).
- Remplacer le clavier par un `JoystickComponent` (chapitre 30).

---

## 29.31 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: "assets/images/heros.png"` | le fichier n'est pas déclaré dans `pubspec.yaml`, ou l'indentation est fausse, ou l'application n'a pas été redémarrée à froid | déclarer `- assets/images/` sous `flutter: assets:` et **relancer** l'application (un hot reload ne suffit pas) |
| `Unable to load asset: "assets/images/assets/images/heros.png"` | chemin dupliqué : vous avez passé le préfixe à `load` | passer `'heros.png'`, jamais `'assets/images/heros.png'` |
| Une image d'un sous-dossier reste introuvable | `- assets/images/` ne déclare **pas** les sous-dossiers | ajouter `- assets/images/decor/` |
| `Tried to access an image "x.png" that does not exist in the cache` | `fromCache` appelé avant tout `load` | précharger dans `FlameGame.onLoad` et attendre avec `await` |
| `LateInitializationError` sur un champ `late final Sprite` | le sprite est lu avant la fin de `onLoad` | ne rien dessiner tant que `isLoaded` est faux, ou charger en amont |
| Le sprite ne s'affiche pas du tout | `size` vaut `Vector2.zero()` | donner une `size` explicite au composant |
| Le pixel art est flou | filtrage par interpolation, valeur par défaut | `paint.filterQuality = FilterQuality.none` et `paint.isAntiAlias = false` |
| Le pixel art scintille pendant les déplacements | zoom non entier | zoom entier (`camera.viewfinder.zoom = 4`) ou `size` multiple de `srcSize` |
| L'animation semble figée | `stepTime` exprimé en millisecondes (`stepTime: 100`) | `stepTime` est en **secondes** : `0.1` pour 10 images/s |
| L'animation vibre à toute vitesse | `stepTime` trop petit (`0.001`) | viser 0,06 à 0,20 selon l'effet |
| Le personnage reste bloqué sur la frame 0 | `animation = ...` réaffecté à chaque frame dans `update` | tester `if (animation != voulue)`, ou passer à `SpriteAnimationGroupComponent` |
| `onComplete` ne se déclenche jamais | l'animation a `loop: true` | construire l'animation avec `loop: false` |
| `Null check operator used on a null value` sur `animationTickers!` | branchement des callbacks avant que `animations` ne soit renseigné | renseigner `animations` puis `current`, et seulement ensuite les tickers |
| Le sprite clignote quand on va à gauche | `flipHorizontally()` appelé à chaque frame | mémoriser l'orientation, ou piloter `scale.x` (idempotent) |
| Le sprite se décale de sa largeur au retournement | `flipHorizontally()` avec `Anchor.topLeft` | `flipHorizontallyAroundCenter()` |
| Le personnage est écrasé ou étiré | `size` ne respecte pas les proportions de `srcSize` | `size = srcSize * facteur` |
| Fines lignes parasites entre les tuiles | *texture bleeding* | `spacing` dans `SpriteSheet`, ou paramètre `bleed` au rendu |
| `createAnimation(0, stepTime: 0.1)` ne compile pas | `row` est un paramètre **nommé requis** en 1.38.0 | `createAnimation(row: 0, stepTime: 0.1)` |
| L'animation lit des frames de la ligne suivante | `amountPerRow` omis alors que `amount` < nombre de colonnes | renseigner `amountPerRow` avec le nombre **de colonnes de la planche** |
| Le jeu rame quand un ennemi apparaît | `Sprite.load` appelé dans le `onLoad` de chaque ennemi | précharger globalement, puis `Sprite(Flame.images.fromCache(...))` |
| Exception au rendu après un changement de niveau | `clearCache()` a libéré des images encore utilisées | retirer les composants avant de vider le cache |
| `SpriteComponent.load(...)` : méthode introuvable | ce constructeur n'existe pas en 1.38.0 | `Sprite.load` puis `SpriteComponent(sprite: ...)`, ou `SpriteComponent.fromImage` |

---

## 29.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `assets/images/` | dossier attendu par défaut ; c'est le `prefix` du cache `Images` |
| `pubspec.yaml` | tout asset doit être déclaré ; un `/` final ne couvre pas les sous-dossiers |
| `Flame.images` | cache global ; `game.images` pointe dessus par défaut |
| `load(nom)` | asynchrone, met en cache ; le second appel est instantané |
| `loadAll(liste)` | charge en parallèle ; à préférer à une boucle de `await` |
| `fromCache(nom)` | **synchrone** ; exige que l'image soit déjà chargée |
| `add(nom, image)` | injecte une image fabriquée en code dans le cache |
| `fetchOrGenerate(nom, gen)` | génère une seule fois ; base de la solution de repli |
| `onLoad()` | l'unique bon endroit pour charger ; appelé une seule fois |
| `Sprite` | une **portion** d'image : `(image, srcPosition, srcSize)` ; sans état ni position |
| `Sprite.load(nom)` | raccourci `images.load` + `Sprite` ; à éviter en série |
| `srcPosition` / `srcSize` | en **pixels de l'image source**, jamais en pixels d'écran |
| `SpriteComponent` | `PositionComponent` qui dessine un `Sprite` |
| `SpriteComponent.fromImage` | découpage directement dans le constructeur |
| `size` | taille dans le monde ; indépendante de `srcSize` ; respectez les proportions |
| `anchor` | point placé exactement à `position` ; gouverne rotation et retournement |
| `flipHorizontallyAroundCenter()` | retournement fiable quelle que soit l'ancre ; **bascule**, ne fixe pas |
| `SpriteAnimation` | description sans état ; partageable entre plusieurs composants |
| `SpriteAnimationTicker` | le lecteur : `currentIndex`, `done()`, `onStart`, `onComplete`, `onFrame` |
| `SpriteAnimationData.sequenced` | `amount`, `stepTime`, `textureSize`, `amountPerRow`, `texturePosition`, `loop` |
| `stepTime` | en **secondes** ; `1 / imagesParSeconde` |
| `loop` | `true` pour idle et marche ; `false` pour attaque, saut et mort |
| `SpriteAnimationComponent` | joue une animation ; `playing`, `removeOnFinish`, `animationTicker` |
| Changer d'animation | ne jamais réaffecter la même animation à chaque frame |
| `SpriteAnimationGroupComponent<T>` | `animations` (table), `current` (état), un ticker par état |
| `removeOnFinish: {etat: true}` | supprime le composant à la fin de l'animation de cet état |
| `SpriteSheet` | grille régulière ; `srcSize`, `margin`, `spacing`, `rows`, `columns` |
| `createAnimation(row:, stepTime:, from:, to:)` | `from` inclus, `to` **exclu** |
| Pixel art net | `FilterQuality.none`, `isAntiAlias = false`, zoom **entier** |
| Licences | CC0 sans contrainte ; CC-BY impose l'attribution ; NC interdit le commercial |
| Organisation | minuscules, tiret bas, une taille de tuile, un atlas par entité |
| `PictureRecorder` | fabrique un `ui.Image` en mémoire, indiscernable d'un PNG |
| `NineTileBoxComponent` | cadre extensible à partir d'une grille 3 × 3 |
| `ParallaxComponent` | fonds à plusieurs couches ; à ajouter au `camera.backdrop` |
| `loadingBuilder` / `errorBuilder` | écran de chargement et diagnostic d'asset manquant |

---

## 29.33 — Exercices

Les dix exercices sont réalisables **sans télécharger la moindre image**. Chacun repose sur le générateur `imageDepuisDessin` de la section 29.26, reproduit dans le préambule commun ci-dessous.

### Préambule commun

Tous les exercices commencent par ces lignes. Recopiez-les en tête de votre `lib/main.dart`.

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flame/sprite.dart';
import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}
```

### Exercice 1 — La potion du donjon (facile)

Écrivez une fonction `creerPotion()` qui fabrique une image de 16 × 16 représentant une potion (bouchon, flacon, liquide rouge). Injectez-la dans le cache de Flame sous la clé `'potion.png'` avec `fetchOrGenerate`, puis affichez-la au centre de l'écran dans un `SpriteComponent` de 64 × 64, ancré au centre.

### Exercice 2 — Découper une planche (facile)

Fabriquez une planche de 4 cases de 16 × 16 côte à côte (une potion rouge, une potion bleue, une potion verte, une potion dorée). Affichez **uniquement la troisième case** (index 2), agrandie quatre fois. Vous n'avez droit qu'à un seul `Sprite`.

### Exercice 3 — Comprendre l'ancre (facile)

Affichez trois fois le même sprite de 32 × 32, tous à la position `Vector2(80 + i * 90, 100)`, avec respectivement `Anchor.topLeft`, `Anchor.center` et `Anchor.bottomRight`. Dessinez sous chacun un petit `CircleComponent` rouge de rayon 2 à la position exacte du sprite, afin de visualiser où se trouve l'ancre.

### Exercice 4 — Première animation (intermédiaire)

Fabriquez une planche de 6 cases de 16 × 16 représentant une clé qui tourne (la largeur du panneton varie). Créez l'animation avec `SpriteAnimationData.sequenced` et affichez-la. Le nombre d'images par seconde doit être défini par une constante `imagesParSeconde = 12`, et `stepTime` calculé à partir d'elle.

### Exercice 5 — Deux animations avec `SpriteSheet` (intermédiaire)

Fabriquez une planche de 4 colonnes × 2 lignes, cases de 16 × 16 : la ligne 0 est une torche qui vacille en orange, la ligne 1 la même torche en bleu. Utilisez `SpriteSheet` et `createAnimation` pour créer les deux animations, et affichez-les côte à côte avec des cadences différentes (0,08 s et 0,16 s).

### Exercice 6 — Une animation qui ne boucle pas (intermédiaire)

Fabriquez une planche de 5 cases de 16 × 16 représentant un coffre qui s'ouvre progressivement. Créez l'animation avec `loop: false`. Le coffre doit démarrer figé sur la première frame (`playing = false`) et ne s'animer qu'après un tap. À la fin de l'animation, affichez le texte « Le coffre est ouvert ! » sous le coffre, grâce à `onComplete`.

### Exercice 7 — Retourner sans clignoter (intermédiaire)

Faites parcourir à un sprite un aller-retour horizontal entre `x = 40` et `x = 260`. Le sprite doit se retourner à chaque changement de sens, **sans clignoter**. Interdiction d'appeler `flipHorizontally()` dans `update` sans condition : pilotez `scale.x`.

### Exercice 8 — Le gobelin et ses trois états (difficile)

Fabriquez une planche de 4 colonnes × 3 lignes (cases de 16 × 16) : ligne 0 *idle*, ligne 1 *marche*, ligne 2 *mort*. Créez un `SpriteAnimationGroupComponent<EtatGobelin>` avec un enum à trois valeurs. Le gobelin patrouille horizontalement (état *marche*). Un tap sur le gobelin le fait passer en *mort* : l'animation ne boucle pas, et le composant se retire de l'arbre à la fin grâce à `removeOnFinish`.

### Exercice 9 — Le comparateur de netteté (difficile)

Fabriquez un damier de 8 × 8 pixels très contrasté. Affichez-le deux fois côte à côte, en 128 × 128 : à gauche avec `FilterQuality.none` et `isAntiAlias = false`, à droite avec `FilterQuality.low`. Ajoutez sous chacun un libellé. Ajoutez enfin un troisième exemplaire affiché en 100 × 100 (facteur non entier) pour visualiser l'irrégularité des pixels.

### Exercice 10 — Écran de chargement et assets centralisés (difficile)

Créez une classe `Assets` contenant les noms de trois images (`heros.png`, `gobelin.png`, `potion.png`) et une table `Map<String, Future<ui.Image> Function()>` associant chaque nom à son générateur. Dans `onLoad`, générez-les **une par une** en publiant l'avancement dans un `ValueNotifier<double>`. Affichez une barre de progression pendant le chargement, puis les trois sprites une fois terminé. Ajoutez un `errorBuilder`.

---

## 29.34 — Corrections des exercices

Chaque correction est un fichier `lib/main.dart` complet. Le préambule commun (imports et `imageDepuisDessin`) est rappelé en tête du premier programme et sous-entendu dans les suivants, où seule la ligne des imports change si nécessaire.

### Correction 1

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

Future<ui.Image> creerPotion() {
  return imageDepuisDessin(16, 16, (Canvas canvas) {
    canvas.drawRect(
      const Rect.fromLTWH(6, 1, 4, 3),
      Paint()..color = const Color(0xFF8B5A2B), // bouchon
    );
    canvas.drawRect(
      const Rect.fromLTWH(4, 4, 8, 11),
      Paint()..color = const Color(0xFF7FD1FF), // verre
    );
    canvas.drawRect(
      const Rect.fromLTWH(5, 8, 6, 6),
      Paint()..color = const Color(0xFFE0245E), // liquide
    );
  });
}

void main() {
  runApp(GameWidget(game: JeuPotion()));
}

class JeuPotion extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await Flame.images.fetchOrGenerate('potion.png', creerPotion);

    await world.add(
      SpriteComponent(
        sprite: Sprite(Flame.images.fromCache('potion.png')),
        position: Vector2.zero(),
        size: Vector2.all(64),
        anchor: Anchor.center,
        paint: Paint()
          ..filterQuality = FilterQuality.none
          ..isAntiAlias = false,
      ),
    );
  }
}
```

**Explication :** `imageDepuisDessin` enregistre les ordres de dessin dans un `PictureRecorder`, puis les convertit en `ui.Image` de 16 × 16. Ce qui n'est pas dessiné reste transparent. `fetchOrGenerate` place l'image dans le cache sous la clé `'potion.png'` : à partir de là, Flame la traite comme un fichier ordinaire. `fromCache` la relit de façon **synchrone**, ce qui est légal ici puisque l'`await` précédent garantit qu'elle est présente. La position `Vector2.zero()` correspond au centre de l'écran, car le `viewfinder` de la caméra est centré par défaut. `FilterQuality.none` garde les pixels nets malgré l'agrandissement de 16 à 64, soit un facteur entier de 4.

### Correction 2

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).

const List<Color> kCouleurs = [
  Color(0xFFE0245E), // rouge
  Color(0xFF4B8BE0), // bleu
  Color(0xFF5FBF5F), // vert
  Color(0xFFE8B04B), // doré
];

Future<ui.Image> creerPlanchePotions() {
  return imageDepuisDessin(64, 16, (Canvas canvas) {
    for (int i = 0; i < 4; i++) {
      final double x = i * 16.0;
      canvas.drawRect(
        Rect.fromLTWH(x + 6, 1, 4, 3),
        Paint()..color = const Color(0xFF8B5A2B),
      );
      canvas.drawRect(
        Rect.fromLTWH(x + 4, 4, 8, 11),
        Paint()..color = const Color(0xFF7FD1FF),
      );
      canvas.drawRect(
        Rect.fromLTWH(x + 5, 8, 6, 6),
        Paint()..color = kCouleurs[i],
      );
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuDecoupe()));
}

class JeuDecoupe extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await Flame.images.fetchOrGenerate('potions.png', creerPlanchePotions);

    const int index = 2; // troisième case : la potion verte
    const double taille = 16;

    await world.add(
      SpriteComponent(
        sprite: Sprite(
          Flame.images.fromCache('potions.png'),
          srcPosition: Vector2(index * taille, 0),
          srcSize: Vector2.all(taille),
        ),
        size: Vector2.all(taille * 4),
        anchor: Anchor.center,
        paint: Paint()..filterQuality = FilterQuality.none,
      ),
    );
  }
}
```

**Explication :** la planche fait 64 × 16, soit quatre cases de 16 × 16. Le rectangle source de la case d'index 2 commence à `x = 2 * 16 = 32`, d'où `srcPosition: Vector2(32, 0)`. `srcSize` reste `Vector2.all(16)` : c'est la taille d'**une** case, jamais celle de la planche. Le `size` du composant vaut `16 * 4 = 64` : le facteur d'agrandissement est entier, condition du pixel art net vue en 29.23. Un seul `Sprite` est construit ; les trois autres potions existent dans l'image mais ne sont jamais dessinées.

### Correction 3

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).

Future<ui.Image> creerCarre() {
  return imageDepuisDessin(32, 32, (Canvas canvas) {
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 32, 32),
      Paint()..color = const Color(0xFF3E7CB1),
    );
    canvas.drawRect(
      const Rect.fromLTWH(2, 2, 28, 28),
      Paint()..color = const Color(0xFF7FB2E0),
    );
  });
}

void main() {
  runApp(GameWidget(game: JeuAncres()));
}

class JeuAncres extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('carre.png', creerCarre);
    final image = Flame.images.fromCache('carre.png');

    const ancres = <String, Anchor>{
      'topLeft': Anchor.topLeft,
      'center': Anchor.center,
      'bottomRight': Anchor.bottomRight,
    };

    int i = 0;
    for (final entree in ancres.entries) {
      final Vector2 point = Vector2(80 + i * 90.0, 100);

      await world.add(
        SpriteComponent.fromImage(
          image,
          position: point,
          size: Vector2.all(32),
          anchor: entree.value,
        ),
      );

      // Marqueur rouge : la position exacte demandée.
      await world.add(
        CircleComponent(
          radius: 2,
          position: point,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFFF3B3B),
        ),
      );

      await world.add(
        TextComponent(
          text: entree.key,
          position: Vector2(80 + i * 90.0, 160),
          anchor: Anchor.topCenter,
          textRenderer: TextPaint(
            style: const TextStyle(fontSize: 12, color: Color(0xFFE8E8F0)),
          ),
        ),
      );
      i++;
    }
  }
}
```

**Explication :** les trois sprites reçoivent **la même** `position`, seul l'`anchor` change. Le point rouge matérialise cette position commune. Avec `Anchor.topLeft`, le carré s'étend vers le bas et la droite du point ; avec `Anchor.center`, il est centré dessus ; avec `Anchor.bottomRight`, il s'étend vers le haut et la gauche. `camera.viewfinder.anchor = Anchor.topLeft` place l'origine du monde en haut à gauche de l'écran, ce qui rend les coordonnées lisibles. Notez que le `CircleComponent` a lui aussi besoin de `Anchor.center`, sans quoi le marqueur serait décalé de son rayon.

### Correction 4

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).

const int imagesParSeconde = 12;

Future<ui.Image> creerPlancheCle() {
  const largeursPanneton = [6.0, 4.5, 2.5, 1.0, 2.5, 4.5];
  return imageDepuisDessin(16 * 6, 16, (Canvas canvas) {
    final or = Paint()..color = const Color(0xFFE8B04B);
    final ombre = Paint()..color = const Color(0xFFB07C22);
    for (int i = 0; i < 6; i++) {
      final double cx = i * 16 + 8;
      final double l = largeursPanneton[i];
      // Anneau
      canvas.drawOval(
        Rect.fromCenter(center: Offset(cx, 5), width: l + 2, height: 7),
        or,
      );
      canvas.drawOval(
        Rect.fromCenter(center: Offset(cx, 5), width: (l + 2) * 0.4, height: 3),
        ombre,
      );
      // Tige
      canvas.drawRect(Rect.fromLTWH(cx - 1, 8, 2, 6), or);
      // Dents
      canvas.drawRect(Rect.fromLTWH(cx - 1, 12, 3, 2), or);
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuCle()));
}

class JeuCle extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await Flame.images.fetchOrGenerate('cle.png', creerPlancheCle);

    await world.add(
      SpriteAnimationComponent.fromFrameData(
        Flame.images.fromCache('cle.png'),
        SpriteAnimationData.sequenced(
          amount: 6,
          stepTime: 1 / imagesParSeconde, // 0,0833 s : 12 images par seconde
          textureSize: Vector2.all(16),
        ),
        size: Vector2.all(96),
        anchor: Anchor.center,
        paint: Paint()..filterQuality = FilterQuality.none,
      ),
    );
  }
}
```

**Explication :** `1 / imagesParSeconde` traduit une cadence en `stepTime`, qui est exprimé en **secondes**. Écrire `stepTime: 12` afficherait une frame toutes les douze secondes, l'erreur la plus fréquente du chapitre. `SpriteAnimationData.sequenced` n'a besoin ni de `texturePosition` (la première frame est en `(0, 0)`) ni de `amountPerRow` (la planche n'a qu'une ligne, donc la valeur par défaut, égale à `amount`, convient). La durée totale d'un tour de clé est `6 / 12 = 0,5` seconde.

### Correction 5

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).
// Ajouter : import 'package:flame/sprite.dart';

Future<ui.Image> creerPlancheTorches() {
  const hauteurs = [7.0, 9.0, 6.0, 8.0];
  const decalages = [0.0, -1.0, 1.0, 0.0];
  const flammes = [Color(0xFFFF9B3B), Color(0xFF4BC8FF)];

  return imageDepuisDessin(16 * 4, 16 * 2, (Canvas canvas) {
    for (int ligne = 0; ligne < 2; ligne++) {
      for (int col = 0; col < 4; col++) {
        final double ox = col * 16.0;
        final double oy = ligne * 16.0;
        // Manche
        canvas.drawRect(
          Rect.fromLTWH(ox + 7, oy + 9, 2, 6),
          Paint()..color = const Color(0xFF6B4423),
        );
        // Flamme
        canvas.drawOval(
          Rect.fromCenter(
            center: Offset(ox + 8 + decalages[col], oy + 6),
            width: 6,
            height: hauteurs[col],
          ),
          Paint()..color = flammes[ligne],
        );
      }
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuTorches()));
}

class JeuTorches extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('torches.png', creerPlancheTorches);

    final feuille = SpriteSheet(
      image: Flame.images.fromCache('torches.png'),
      srcSize: Vector2.all(16),
    );

    final animations = <SpriteAnimation>[
      feuille.createAnimation(row: 0, stepTime: 0.08),
      feuille.createAnimation(row: 1, stepTime: 0.16),
    ];

    for (int i = 0; i < animations.length; i++) {
      await world.add(
        SpriteAnimationComponent(
          animation: animations[i],
          position: Vector2(90 + i * 110.0, 100),
          size: Vector2.all(80),
          anchor: Anchor.center,
          paint: Paint()..filterQuality = FilterQuality.none,
        ),
      );
    }
  }
}
```

**Explication :** `SpriteSheet` déduit lui-même la grille : `columns = 64 ~/ 16 = 4` et `rows = 32 ~/ 16 = 2`. `createAnimation(row: 0, ...)` prend donc les quatre cases de la première ligne, sans qu'aucun rectangle ne soit calculé à la main. Les paramètres `from` et `to` sont omis : ils valent 0 et 4, soit la ligne entière. Les deux `stepTime` différents montrent que la cadence appartient à l'animation, pas à la planche : la torche bleue vacille deux fois plus lentement.

### Correction 6

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).
// Ajouter : import 'package:flame/events.dart';
//           import 'package:flame/sprite.dart';

Future<ui.Image> creerPlancheCoffre() {
  const ouvertures = [0.0, 2.0, 4.0, 6.0, 8.0];
  return imageDepuisDessin(16 * 5, 16, (Canvas canvas) {
    final bois = Paint()..color = const Color(0xFF8B5A2B);
    final ferrure = Paint()..color = const Color(0xFFE8B04B);
    final interieur = Paint()..color = const Color(0xFF201820);

    for (int i = 0; i < 5; i++) {
      final double ox = i * 16.0;
      // Cuve
      canvas.drawRect(Rect.fromLTWH(ox + 2, 8, 12, 6), bois);
      canvas.drawRect(Rect.fromLTWH(ox + 3, 9, 10, 4), interieur);
      // Couvercle, qui se soulève
      canvas.save();
      canvas.translate(ox + 8, 8 - ouvertures[i]);
      canvas.drawRect(const Rect.fromLTWH(-6, -4, 12, 4), bois);
      canvas.drawRect(const Rect.fromLTWH(-6, -2, 12, 1), ferrure);
      canvas.restore();
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuCoffre()));
}

class Coffre extends SpriteAnimationComponent with TapCallbacks {
  Coffre({required this.surOuverture})
      : super(
          size: Vector2.all(96),
          anchor: Anchor.center,
          playing: false, // figé sur la première frame
          paint: Paint()..filterQuality = FilterQuality.none,
        );

  final void Function() surOuverture;

  @override
  void onLoad() {
    final feuille = SpriteSheet(
      image: Flame.images.fromCache('coffre.png'),
      srcSize: Vector2.all(16),
    );
    animation = feuille.createAnimation(row: 0, stepTime: 0.12, loop: false);
    animationTicker?.onComplete = surOuverture;
  }

  @override
  void onTapDown(TapDownEvent event) {
    if (!playing) {
      playing = true;
    }
  }
}

class JeuCoffre extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    await Flame.images.fetchOrGenerate('coffre.png', creerPlancheCoffre);

    final message = TextComponent(
      text: 'Touchez le coffre',
      position: Vector2(0, 80),
      anchor: Anchor.topCenter,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFE8E8F0)),
      ),
    );

    await world.add(message);
    await world.add(
      Coffre(surOuverture: () => message.text = 'Le coffre est ouvert !'),
    );
  }
}
```

**Explication :** trois mécanismes se combinent. `playing: false` fige l'animation sur la frame 0 tant que personne n'a touché le coffre. `TapCallbacks` (chapitre 30) fournit `onTapDown` ; le composant a une `size` non nulle, condition sans laquelle il ne recevrait aucun tap. Enfin `loop: false` fait s'arrêter l'animation sur la dernière frame — le coffre reste ouvert — et déclenche `onComplete`, branché ici sur une fonction passée au constructeur. Ce callback ne serait **jamais** appelé avec `loop: true`.

### Correction 7

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).

Future<ui.Image> creerFleche() {
  return imageDepuisDessin(16, 16, (Canvas canvas) {
    final corps = Paint()..color = const Color(0xFFE8B04B);
    final pointe = Paint()..color = const Color(0xFFFF6B3B);
    canvas.drawRect(const Rect.fromLTWH(2, 6, 9, 4), corps);
    final chemin = Path()
      ..moveTo(10, 3)
      ..lineTo(15, 8)
      ..lineTo(10, 13)
      ..close();
    canvas.drawPath(chemin, pointe);
  });
}

class Navette extends SpriteComponent {
  Navette()
      : super(
          position: Vector2(40, 100),
          size: Vector2.all(64),
          anchor: Anchor.center,
          paint: Paint()..filterQuality = FilterQuality.none,
        );

  static const double xMin = 40;
  static const double xMax = 260;

  double _vx = 90; // pixels par seconde

  @override
  void onLoad() {
    sprite = Sprite(Flame.images.fromCache('fleche.png'));
  }

  @override
  void update(double dt) {
    super.update(dt);

    position.x += _vx * dt;

    if (position.x >= xMax) {
      position.x = xMax;
      _vx = -_vx;
    } else if (position.x <= xMin) {
      position.x = xMin;
      _vx = -_vx;
    }

    // Idempotent : réécrire la même valeur ne provoque aucun clignotement.
    scale.x = _vx < 0 ? -1 : 1;
  }
}

void main() {
  runApp(GameWidget(game: JeuNavette()));
}

class JeuNavette extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await Flame.images.fetchOrGenerate('fleche.png', creerFleche);
    await world.add(Navette());
  }
}
```

**Explication :** la clé est la dernière ligne d'`update`. `scale.x = _vx < 0 ? -1 : 1` **fixe** une valeur, alors que `flipHorizontally()` **bascule** : appelée soixante fois par seconde, la seconde inverserait le sprite à chaque frame et produirait un clignotement. Le recadrage `position.x = xMax` avant l'inversion évite que le sprite ne dépasse la borne d'une fraction de pixel selon la valeur de `dt`. L'ancre au centre garantit que le retournement se fait sur place.

### Correction 8

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).
// Ajouter : import 'package:flame/events.dart';
//           import 'package:flame/sprite.dart';

enum EtatGobelin { idle, marche, mort }

Future<ui.Image> creerPlancheGobelin() {
  const vert = Color(0xFF6FBF4F);
  const vertFonce = Color(0xFF3E7A2E);
  const noir = Color(0xFF201820);

  void corps(Canvas c, double ox, double oy, double dy, double ecart) {
    c.drawRect(Rect.fromLTWH(ox + 5 - ecart, oy + 12, 2, 3),
        Paint()..color = vertFonce);
    c.drawRect(Rect.fromLTWH(ox + 9 + ecart, oy + 12, 2, 3),
        Paint()..color = vertFonce);
    c.drawRect(Rect.fromLTWH(ox + 4, oy + 6 + dy, 8, 7), Paint()..color = vert);
    c.drawRect(Rect.fromLTWH(ox + 5, oy + 8 + dy, 2, 2), Paint()..color = noir);
    c.drawRect(Rect.fromLTWH(ox + 9, oy + 8 + dy, 2, 2), Paint()..color = noir);
  }

  return imageDepuisDessin(16 * 4, 16 * 3, (Canvas canvas) {
    // Ligne 0 : idle
    const dyIdle = [0.0, 1.0, 0.0, -1.0];
    for (int i = 0; i < 4; i++) {
      corps(canvas, i * 16.0, 0, dyIdle[i], 0);
    }
    // Ligne 1 : marche
    const ecarts = [0.0, 2.0, 0.0, -2.0];
    for (int i = 0; i < 4; i++) {
      corps(canvas, i * 16.0, 16, 0, ecarts[i]);
    }
    // Ligne 2 : mort, le gobelin s'affaisse et pâlit
    for (int i = 0; i < 4; i++) {
      canvas.save();
      canvas.translate(i * 16.0, 32);
      canvas.drawRect(
        Rect.fromLTWH(3, 13 - i * 0.0, 10, 2 + i.toDouble()),
        Paint()..color = Color.lerp(vert, const Color(0xFF60604F), i / 3)!,
      );
      canvas.restore();
    }
  });
}

class Gobelin extends SpriteAnimationGroupComponent<EtatGobelin>
    with TapCallbacks {
  Gobelin({required Vector2 position})
      : super(
          position: position,
          size: Vector2.all(64),
          anchor: Anchor.center,
          removeOnFinish: {EtatGobelin.mort: true},
          paint: Paint()..filterQuality = FilterQuality.none,
        );

  double _vx = 50;

  @override
  void onLoad() {
    final feuille = SpriteSheet(
      image: Flame.images.fromCache('gobelin.png'),
      srcSize: Vector2.all(16),
    );

    animations = {
      EtatGobelin.idle: feuille.createAnimation(row: 0, stepTime: 0.20),
      EtatGobelin.marche: feuille.createAnimation(row: 1, stepTime: 0.12),
      EtatGobelin.mort:
          feuille.createAnimation(row: 2, stepTime: 0.14, loop: false),
    };
    current = EtatGobelin.marche;
  }

  @override
  void onTapDown(TapDownEvent event) {
    if (current == EtatGobelin.mort) return;
    current = EtatGobelin.mort;
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (current == EtatGobelin.mort) return; // plus aucun déplacement

    position.x += _vx * dt;
    if (position.x > 250 || position.x < 60) {
      _vx = -_vx;
    }
    scale.x = _vx < 0 ? -1 : 1;
  }
}

void main() {
  runApp(GameWidget(game: JeuGobelin()));
}

class JeuGobelin extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('gobelin.png', creerPlancheGobelin);
    await world.add(Gobelin(position: Vector2(150, 110)));
  }
}
```

**Explication :** trois animations sont rangées dans la table `animations`, indexée par l'enum `EtatGobelin` du chapitre 11 : une faute de frappe sur un état devient une erreur de compilation. `removeOnFinish: {EtatGobelin.mort: true}` demande à Flame de retirer le composant de l'arbre dès que l'animation de mort, non bouclée, se termine — aucun `removeFromParent()` n'est écrit. Les deux `return` en tête de `onTapDown` et d'`update` implémentent la garde d'état : une fois mort, le gobelin ne bouge plus et ne peut pas mourir deux fois. L'ordre de construction compte : on renseigne `animations` **avant** `current`.

### Correction 9

```dart
// Préambule commun : imports + imageDepuisDessin (voir correction 1).

Future<ui.Image> creerDamier() {
  return imageDepuisDessin(8, 8, (Canvas canvas) {
    const clair = Color(0xFFF4F4FA);
    const fonce = Color(0xFF201830);
    for (int y = 0; y < 8; y++) {
      for (int x = 0; x < 8; x++) {
        canvas.drawRect(
          Rect.fromLTWH(x.toDouble(), y.toDouble(), 1, 1),
          Paint()..color = (x + y).isEven ? clair : fonce,
        );
      }
    }
  });
}

void main() {
  runApp(GameWidget(game: JeuNettete()));
}

class JeuNettete extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    await Flame.images.fetchOrGenerate('damier.png', creerDamier);
    final image = Flame.images.fromCache('damier.png');

    final exemples = <List<Object>>[
      [
        'none, x16',
        Vector2.all(128),
        Paint()
          ..filterQuality = FilterQuality.none
          ..isAntiAlias = false,
      ],
      [
        'low, x16',
        Vector2.all(128),
        Paint()..filterQuality = FilterQuality.low,
      ],
      [
        'none, x12,5',
        Vector2.all(100),
        Paint()
          ..filterQuality = FilterQuality.none
          ..isAntiAlias = false,
      ],
    ];

    for (int i = 0; i < exemples.length; i++) {
      final String titre = exemples[i][0] as String;
      final Vector2 taille = exemples[i][1] as Vector2;

      await world.add(
        SpriteComponent.fromImage(
          image,
          position: Vector2(40 + i * 150.0, 40),
          size: taille,
          paint: exemples[i][2] as Paint,
        ),
      );

      await world.add(
        TextComponent(
          text: titre,
          position: Vector2(40 + i * 150.0 + taille.x / 2, 180),
          anchor: Anchor.topCenter,
          textRenderer: TextPaint(
            style: const TextStyle(fontSize: 12, color: Color(0xFFE8E8F0)),
          ),
        ),
      );
    }
  }
}
```

**Explication :** l'image source ne fait que 8 × 8 pixels, ce qui rend les différences évidentes. À gauche, `FilterQuality.none` associé à un facteur entier de 16 donne des carrés parfaitement francs. Au centre, `FilterQuality.low` interpole : les frontières deviennent des dégradés gris, et le damier perd son identité. À droite, le facteur `100 / 8 = 12,5` n'est pas entier : certains carrés occupent 12 pixels et d'autres 13, ce qui produit des colonnes visiblement inégales même avec `FilterQuality.none`. Les deux conditions du pixel art net — filtrage désactivé **et** facteur entier — sont donc bien indépendantes.

### Correction 10

```dart
import 'dart:ui' as ui;

import 'package:flame/components.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final recorder = ui.PictureRecorder();
  final canvas = Canvas(recorder);
  dessin(canvas);
  return recorder.endRecording().toImage(largeur, hauteur);
}

// --- Générateurs simples : un pictogramme 16 x 16 par entité ---

Future<ui.Image> creerHeros() => imageDepuisDessin(16, 16, (Canvas c) {
      c.drawRect(const Rect.fromLTWH(5, 2, 6, 5),
          Paint()..color = const Color(0xFFF0C89A));
      c.drawRect(const Rect.fromLTWH(4, 7, 8, 6),
          Paint()..color = const Color(0xFF3E7CB1));
      c.drawRect(const Rect.fromLTWH(12, 4, 2, 9),
          Paint()..color = const Color(0xFFD8D8E0));
    });

Future<ui.Image> creerGobelin() => imageDepuisDessin(16, 16, (Canvas c) {
      c.drawRect(const Rect.fromLTWH(4, 5, 8, 7),
          Paint()..color = const Color(0xFF6FBF4F));
      c.drawRect(const Rect.fromLTWH(5, 7, 2, 2),
          Paint()..color = const Color(0xFF201820));
      c.drawRect(const Rect.fromLTWH(9, 7, 2, 2),
          Paint()..color = const Color(0xFF201820));
    });

Future<ui.Image> creerPotion() => imageDepuisDessin(16, 16, (Canvas c) {
      c.drawRect(const Rect.fromLTWH(6, 1, 4, 3),
          Paint()..color = const Color(0xFF8B5A2B));
      c.drawRect(const Rect.fromLTWH(4, 4, 8, 11),
          Paint()..color = const Color(0xFF7FD1FF));
      c.drawRect(const Rect.fromLTWH(5, 8, 6, 6),
          Paint()..color = const Color(0xFFE0245E));
    });

// --- Centralisation des assets ---

typedef GenerateurImage = Future<ui.Image> Function();

class Assets {
  const Assets._();

  static const String heros = 'heros.png';
  static const String gobelin = 'gobelin.png';
  static const String potion = 'potion.png';

  static const Map<String, GenerateurImage> generateurs = {
    heros: creerHeros,
    gobelin: creerGobelin,
    potion: creerPotion,
  };
}

// --- Le jeu ---

class DonjonDeDart extends FlameGame {
  final ValueNotifier<double> progression = ValueNotifier<double>(0);

  @override
  Color backgroundColor() => const Color(0xFF101018);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final entrees = Assets.generateurs.entries.toList();
    for (int i = 0; i < entrees.length; i++) {
      await Flame.images.fetchOrGenerate(entrees[i].key, entrees[i].value);
      progression.value = (i + 1) / entrees.length;
      // Petite pause : rend la barre de progression visible en démonstration.
      await Future<void>.delayed(const Duration(milliseconds: 250));
    }

    for (int i = 0; i < entrees.length; i++) {
      await world.add(
        SpriteComponent(
          sprite: Sprite(Flame.images.fromCache(entrees[i].key)),
          position: Vector2(70 + i * 90.0, 90),
          size: Vector2.all(64),
          anchor: Anchor.center,
          paint: Paint()..filterQuality = FilterQuality.none,
        ),
      );
      await world.add(
        TextComponent(
          text: entrees[i].key,
          position: Vector2(70 + i * 90.0, 130),
          anchor: Anchor.topCenter,
          textRenderer: TextPaint(
            style: const TextStyle(fontSize: 11, color: Color(0xFFE8E8F0)),
          ),
        ),
      );
    }
  }
}

void main() {
  final jeu = DonjonDeDart();

  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: GameWidget<DonjonDeDart>(
          game: jeu,
          loadingBuilder: (BuildContext context) => ColoredBox(
            color: const Color(0xFF101018),
            child: Center(
              child: SizedBox(
                width: 220,
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    const Text(
                      'Préparation du donjon...',
                      style: TextStyle(color: Color(0xFFE8E8F0)),
                    ),
                    const SizedBox(height: 12),
                    ValueListenableBuilder<double>(
                      valueListenable: jeu.progression,
                      builder: (BuildContext c, double v, Widget? _) {
                        return LinearProgressIndicator(
                          value: v,
                          color: const Color(0xFFE8B04B),
                          backgroundColor: const Color(0xFF2A2440),
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),
          ),
          errorBuilder: (BuildContext context, Object erreur) => ColoredBox(
            color: const Color(0xFF3A1020),
            child: Center(
              child: Text(
                'Erreur de chargement : $erreur',
                style: const TextStyle(color: Color(0xFFFFD0D0)),
              ),
            ),
          ),
        ),
      ),
    ),
  );
}
```

**Explication :** la classe `Assets` centralise les noms **et** les générateurs. Son constructeur privé `Assets._()` interdit l'instanciation : c'est une simple boîte à constantes. Toute faute de frappe sur `Assets.gobelin` devient une erreur de compilation, alors qu'une chaîne `'gobelin.png'` écrite à la main ne serait détectée qu'à l'exécution. Le chargement se fait **en série** afin de pouvoir publier l'avancement dans le `ValueNotifier` après chaque image ; c'est plus lent qu'un `loadAll` parallèle, mais c'est le prix d'une barre de progression honnête. Le `ValueListenableBuilder` du chapitre 19 reconstruit uniquement la barre, pas tout l'écran. Enfin, `errorBuilder` transforme un plantage rouge illisible en un message exploitable : c'est le premier réflexe à prendre quand on manipule des assets.

---

## Et maintenant ?

Votre héros existe, il respire, il marche et il frappe. Il ne vous obéit encore qu'au clavier, et de façon rudimentaire : une seule touche à la fois, aucun support tactile, aucune manette.

Le chapitre suivant est consacré aux **entrées**. Vous y verrez le mixin `KeyboardHandler` en détail, les taps avec `TapCallbacks`, les glissements avec `DragCallbacks`, puis les deux composants qui rendent un jeu jouable sur téléphone : le `JoystickComponent` et le `HudButtonComponent`. Vous y brancherez enfin proprement les états d'animation construits ici.

[30-PARTIE-2B—ENTRÉES-CLAVIER-TACTILE-ET-JOYSTICK.md](./30-PARTIE-2B—ENTRÉES-CLAVIER-TACTILE-ET-JOYSTICK.md)
