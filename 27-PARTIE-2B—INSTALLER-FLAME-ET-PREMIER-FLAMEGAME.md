# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 27 — INSTALLER FLAME ET PREMIER FLAMEGAME

> **Version de Flame utilisée dans ce chapitre :** `flame` **1.38.0** (publiée le 19 juillet 2026).
> **Date de vérification des API :** 8 août 2026, sur `https://docs.flame-engine.org/latest/`,
> `https://pub.dev/packages/flame` et le dépôt `flame-engine/flame` (branche `main`).
> **Contraintes SDK déclarées par le paquet :** `sdk: ">=3.12.0 <4.0.0"`, `flutter: ">=3.44.0"`.
>
> **Niveau :** intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 19 (Flutter, `runApp`, widgets), chapitre 20 (boucle de jeu et `dt`), chapitre 21 (`Canvas`), chapitre 23 (vecteurs), chapitre 26 (architecture), et côté Dart les chapitres 12 (null safety) et 15 (asynchrone).
> **Ce que vous saurez faire à la fin :** créer un projet Flutter, y installer Flame, écrire un `FlameGame` complet avec `onLoad`, `update` et `render`, l'afficher dans un `GameWidget`, manipuler des `Vector2` sans piège, et lancer le tout sur Web, Android et Windows.

---

## 27.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est Flame, et ce qu'il n'est pas ;
- dresser la correspondance exacte entre le mini-moteur du chapitre 26 et les classes de Flame ;
- citer les paquets de l'écosystème (`flame`, `flame_audio`, `flame_tiled`, `flame_forge2d`, `flame_bloc`) et savoir lequel installer ;
- créer le projet `donjon_de_dart` avec `flutter create` en choisissant ses plateformes ;
- ajouter Flame avec `flutter pub add flame` et comprendre ce que la commande a écrit ;
- lire et commenter chaque ligne d'un `pubspec.yaml` de jeu ;
- reconnaître un tutoriel Flame périmé en moins de dix secondes ;
- écrire une classe qui hérite de `FlameGame` ;
- insérer un jeu dans une application Flutter avec `GameWidget` ;
- écrire un `main.dart` minimal qui affiche un jeu qui tourne ;
- utiliser `onLoad()` comme un constructeur asynchrone ;
- distinguer `onLoad()`, `onMount()` et `onRemove()` ;
- retrouver dans `update(double dt)` le `dt` exact du chapitre 20 ;
- retrouver dans `render(Canvas canvas)` le `Canvas` exact du chapitre 21 ;
- lire `size` et `canvasSize` sans les confondre ;
- manipuler `Vector2` et le comparer au `Vec2` que vous aviez écrit au chapitre 23 ;
- utiliser `+`, `-`, `*`, `/`, `length`, `normalized`, `distanceTo`, `dot` sur des `Vector2` ;
- éviter le piège des références partagées grâce à `.clone()` ;
- écrire un premier carré qui bouge sous Flame ;
- comparer ce code, ligne à ligne, à la version Flutter pure du chapitre 20 ;
- changer la couleur de fond avec `backgroundColor()` ;
- utiliser `GameWidget.controlled` et savoir qui possède l'instance de jeu ;
- activer le mode debug et lire ce qu'il affiche ;
- réagir proprement au redimensionnement grâce à `onGameResize` ;
- lancer le jeu sur Web, Android et Windows ;
- organiser les dossiers du projet pour les quinze chapitres suivants.

---

## 27.1 — Qu'est-ce que Flame ?

Flame est un **moteur de jeu 2D** écrit en Dart, qui s'exécute **par-dessus Flutter**.

Retenez cette dernière précision, elle est structurante. Flame n'est pas un environnement séparé, avec son propre langage et son propre rendu. C'est un **paquet Dart ordinaire**, que vous installez comme vous avez installé n'importe quel paquet au chapitre 16. Il utilise le `Canvas` de Flutter pour dessiner, le `Ticker` de Flutter pour cadencer, et le système de widgets de Flutter pour s'insérer dans une application.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                     VOTRE JEU                                │
  │       Joueur, Gobelin, Potion, Boss, HUD, niveaux            │
  └──────────────────────────────────────────────────────────────┘
                              │ utilise
                              ▼
  ┌──────────────────────────────────────────────────────────────┐
  │                      FLAME  1.38.0                           │
  │   FlameGame, Component, PositionComponent, CameraComponent,  │
  │   World, Sprite, SpriteAnimation, hitbox, effets, timers     │
  └──────────────────────────────────────────────────────────────┘
                              │ utilise
                              ▼
  ┌──────────────────────────────────────────────────────────────┐
  │                        FLUTTER                               │
  │   Widget, RenderObject, Ticker, Canvas, Paint, gestes        │
  └──────────────────────────────────────────────────────────────┘
                              │ utilise
                              ▼
  ┌──────────────────────────────────────────────────────────────┐
  │                DART + moteur graphique Impeller/Skia         │
  └──────────────────────────────────────────────────────────────┘
```

Trois conséquences pratiques découlent de cette pile.

**Vous ne perdez rien de Flutter.** Un bouton `ElevatedButton`, une `Row`, un `Navigator` restent utilisables au-dessus du jeu. Un menu principal de jeu Flame est souvent un simple widget Flutter. Vous verrez cela au chapitre 35 avec les *overlays*.

**Vous ne perdez rien de Dart.** Vos classes, vos mixins, vos enums, vos `Future`, vos tests du chapitre 18 fonctionnent tels quels. Flame n'introduit aucun nouveau langage.

**Vous héritez des cibles de Flutter.** Un jeu Flame se compile pour Android, iOS, Web, Windows, macOS et Linux, à partir du même code source.

### Ce que Flame vous donne, en une phrase

Flame vous donne **une boucle de jeu, un arbre d'objets de jeu, une caméra, et une bibliothèque d'outils 2D**, pour que vous écriviez la logique de votre jeu au lieu de réécrire la plomberie.

Vous savez précisément ce qu'est cette plomberie : vous l'avez écrite vous-même du chapitre 20 au chapitre 26. C'est exactement l'objet de la section suivante.

> **À retenir.** Flame n'est ni un langage, ni un runtime : c'est une bibliothèque Dart. Tout ce que vous savez de Dart et de Flutter reste vrai.

---

## 27.2 — Ce que Flame apporte par rapport au chapitre 26

Vous n'abordez pas Flame en terrain inconnu. Vous avez écrit, à la main, un moteur qui contient déjà les mêmes idées. Le tableau ci-dessous met en regard **votre code** et **le code de Flame**.

| Ce que vous avez écrit à la main (ch. 20 à 26) | Ce que Flame fournit | Où c'est vu |
| --- | --- | --- |
| Une classe `MoteurDeBoucle` avec un `Ticker` et un calcul de `dt` | `FlameGame` : boucle de jeu intégrée, `dt` en secondes | 27.9, 27.14 |
| Un `CustomPainter` + un `StatefulWidget` pour afficher la scène | `GameWidget` : un seul widget à poser dans l'arbre Flutter | 27.10 |
| `abstract class Entity` avec `update(dt)` et `render(canvas)` | `Component`, et sa variante géométrique `PositionComponent` | ch. 28 |
| `List<Entity> entites` + files `_aAjouter` / `_aRetirer` | Arbre de composants, `add()`, `addAll()`, `removeFromParent()` — les files sont gérées par le moteur | ch. 28 |
| Le bug de modification concurrente, corrigé à la main | Impossible : les ajouts et retraits sont différés par le moteur | 27.13 |
| Un champ `priority` et un `sort()` avant le rendu | Propriété `priority` sur `Component`, tri automatique | ch. 28 |
| `class Vec2 { double x, y; ... }` écrit au chapitre 23 | `Vector2` du paquet `vector_math`, ré-exporté par Flame | 27.17 |
| Chargement d'images avec `decodeImageFromList` et un `Completer` | `Flame.images.load()`, `Sprite.load()`, cache intégré | ch. 29 |
| Découpage manuel d'une sprite sheet, compteur de frames | `SpriteSheet`, `SpriteAnimation`, `SpriteAnimationComponent` | ch. 29 |
| `canvas.translate(-camX, -camY)` et calculs monde/écran | `CameraComponent`, `World`, `Viewfinder`, `Viewport` | ch. 31 |
| Fonction `chevauchent(...)` AABB écrite à la main | `HasCollisionDetection`, `RectangleHitbox`, `CircleHitbox`, `onCollisionStart` | ch. 32 |
| Interpolations manuelles pour un déplacement animé | `MoveEffect`, `ScaleEffect`, `OpacityEffect`, `SequenceEffect` | ch. 33 |
| Compteurs `double _restant` décrémentés dans `update` | `Timer`, `TimerComponent` | ch. 33 |
| Gestion du clavier avec `RawKeyboardListener` | `KeyboardHandler`, `HasKeyboardHandlerComponents` | ch. 30 |
| Lecture de sons : rien, vous n'en aviez pas | `flame_audio` (paquet séparé) | ch. 34 |

Et symétriquement, voici **ce que Flame ne fournit pas**. Ces briques restent votre travail, et vous les reprendrez du chapitre 26.

| Brique du chapitre 26 | Flame la fournit ? |
| --- | --- |
| Machine à états de jeu (`GameState`, transitions autorisées) | Non |
| Pile d'états (pause par-dessus le jeu) | Non |
| Gestionnaire d'entrées abstrait (`ActionJeu`, remappage) | Non |
| Bus d'événements | Non |
| Service de données (`DonneesDeJeu`, score, vies, record) | Non |
| Injection de dépendances | Non |
| Organisation des fichiers du projet | Non |
| Tests | Non |

> **À retenir.** Flame remplace la **plomberie technique** du chapitre 26, pas son **architecture**. Les sections 26.16 à 26.31 restent d'actualité et seront réutilisées telles quelles dans la PARTIE 2C.

---

## 27.3 — Flame n'est pas un moteur 3D ni un éditeur

Cette section existe pour éviter une déception, et surtout un mauvais choix technique.

### Flame est strictement 2D

Il n'y a ni caméra perspective, ni maillage, ni matériau, ni éclairage volumétrique. Toutes les coordonnées sont des `Vector2`, c'est-à-dire `(x, y)`. Un jeu isométrique reste possible, mais il est **dessiné** en 2D avec des sprites ; la troisième dimension n'existe que dans l'illusion.

Si votre projet est réellement en 3D, Flame n'est pas l'outil. Regardez plutôt Godot, Unity ou Unreal.

### Flame n'a pas d'éditeur visuel

Il n'existe pas de fenêtre où l'on fait glisser un ennemi à la souris pour le placer dans un niveau. **Tout se fait en code.**

```text
  ┌──────────────────────────┬──────────────────────────────────────┐
  │  Unity / Godot           │  Flame                               │
  ├──────────────────────────┼──────────────────────────────────────┤
  │  Fenêtre « Scene »       │  du code Dart dans onLoad()          │
  │  Inspecteur de propriétés│  des paramètres de constructeur      │
  │  Arbre de scène cliquable│  un arbre de composants en mémoire   │
  │  Fichier .scene binaire  │  vos propres classes                 │
  │  Play dans l'éditeur     │  flutter run + hot reload            │
  └──────────────────────────┴──────────────────────────────────────┘
```

Ce n'est pas seulement un manque. C'est aussi un avantage :

- tout votre jeu est du texte, donc **versionnable dans Git** avec des différences lisibles ;
- il n'y a rien à apprendre en dehors du langage ;
- deux personnes peuvent travailler sur deux fichiers sans conflit binaire.

Pour concevoir des **niveaux**, l'usage est d'employer un éditeur externe, **Tiled**, et de charger le résultat avec `flame_tiled`. Vous verrez cela au chapitre 34.

### Flame n'est pas un framework « tout compris »

Flame ne vous impose ni structure de projet, ni système de sauvegarde, ni gestion de scènes, ni système de succès, ni multijoueur. Il vous donne des briques. Le plan, c'est vous.

> **À retenir.** Flame est une **bibliothèque de jeu 2D en code**. Pas de 3D, pas d'éditeur, pas de magie. C'est exactement pour cela qu'il se comprend entièrement.

---

## 27.4 — L'écosystème : `flame`, `flame_audio`, `flame_tiled`, `flame_forge2d`, `flame_bloc`

Flame est découpé en un paquet central et des **paquets-ponts**. Un paquet-pont relie Flame à une autre bibliothèque de l'écosystème Dart.

| Paquet | Version vérifiée le 8 août 2026 | Rôle | Chapitre |
| --- | --- | --- | --- |
| `flame` | **1.38.0** | le moteur : boucle, composants, caméra, sprites, collisions, effets | 27 à 33 |
| `flame_audio` | **2.12.2** | lecture de sons et de musique, via `audioplayers` | 34 |
| `flame_tiled` | **3.1.2** | chargement de cartes créées avec l'éditeur Tiled (`.tmx`) | 34 |
| `flame_forge2d` | **0.19.3+7** | physique rigide complète (Box2D porté en Dart) | 34, aperçu |
| `flame_bloc` | version à vérifier au moment de l'installer | pont vers la gestion d'état `bloc` | non utilisé dans ce cours |

Quelques précisions importantes.

**Vous n'installez que ce dont vous avez besoin.** Ajouter `flame_forge2d` alors que vous ne faites pas de physique rigide alourdit le projet sans aucun bénéfice. Dans ce chapitre, **seul `flame` est nécessaire**.

**Chaque pont dépend d'une version précise de `flame`.** `flame_audio` 2.12.2 déclare `flame ^1.38.0`. Si vous figez `flame` sur une version ancienne, le pont refusera de s'installer. C'est la cause numéro un des messages « version solving failed ».

**`flame_forge2d` est en version 0.x.** En versionnage sémantique, cela signifie que l'API peut changer entre deux versions mineures. Traitez ce paquet avec prudence.

**`flame_bloc` n'est pas utilisé dans ce cours.** La gestion d'état de notre jeu reposera sur les briques écrites au chapitre 26 (machine à états, service de données), qui sont plus simples à comprendre et suffisantes pour un jeu 2D d'apprentissage. Le paquet existe et est parfaitement légitime dans un projet professionnel qui utilise déjà `bloc` côté application.

```text
  ┌───────────────────────────────────────────────────────────────┐
  │                            flame                              │
  │                        (obligatoire)                          │
  └───────────────────────────────────────────────────────────────┘
        │              │               │                │
        ▼              ▼               ▼                ▼
  ┌───────────┐  ┌───────────┐  ┌──────────────┐  ┌────────────┐
  │flame_audio│  │flame_tiled│  │flame_forge2d │  │ flame_bloc │
  │  sons     │  │  cartes   │  │  physique    │  │ état bloc  │
  │audioplayers│ │   tiled   │  │   forge2d    │  │    bloc    │
  └───────────┘  └───────────┘  └──────────────┘  └────────────┘
```

> **À retenir.** `flame` seul suffit pour les chapitres 27 à 33. Les ponts s'ajoutent au moment où l'on en a besoin, jamais « au cas où ».

---

## 27.5 — Créer le projet (`flutter create donjon_de_dart`)

Nous créons maintenant le projet qui nous accompagnera jusqu'au chapitre 42. Il s'appelle **`donjon_de_dart`**, comme le projet console du chapitre 18.

### Vérifier l'installation

Avant tout, contrôlez votre environnement.

```bash
flutter --version
flutter doctor
```

**Résultat attendu (les numéros peuvent différer, les contraintes non) :**

```text
Flutter 3.44.x • channel stable
Dart 3.12.x
```

Flame 1.38.0 exige **Flutter 3.44.0 au minimum** et **Dart 3.12.0 au minimum**. Si votre version est inférieure, mettez à jour avant d'aller plus loin :

```bash
flutter upgrade
```

### Créer le projet

```bash
flutter create donjon_de_dart
cd donjon_de_dart
```

**Résultat :**

```text
Creating project donjon_de_dart...
Resolving dependencies...
Got dependencies.
Wrote 127 files.

All done!
You are now ready to run your application.
```

### Nommage du projet

Le nom d'un projet Dart obéit à des règles strictes, rappelées au chapitre 16 :

| Règle | Exemple valide | Exemple refusé |
| --- | --- | --- |
| minuscules uniquement | `donjon_de_dart` | `DonjonDeDart` |
| séparateur `_` (snake_case) | `donjon_de_dart` | `donjon-de-dart` |
| ne commence pas par un chiffre | `jeu2d` | `2djeu` |
| pas de mot réservé Dart | `donjon_de_dart` | `class` |

### Choisir les plateformes

Par défaut, `flutter create` génère **toutes** les plateformes disponibles sur votre machine. C'est beaucoup de dossiers pour rien. Vous pouvez restreindre :

```bash
flutter create --platforms=web,android,windows donjon_de_dart
```

Et si plus tard vous voulez en ajouter une, relancez la commande dans le dossier existant :

```bash
flutter create --platforms=linux .
```

### Choisir l'identifiant d'organisation

L'identifiant de l'application sur Android et iOS se construit à partir de l'option `--org` :

```bash
flutter create --org com.monstudio --platforms=web,android,windows donjon_de_dart
```

Cela produit l'identifiant `com.monstudio.donjon_de_dart`. Changer cet identifiant après coup est pénible ; réfléchissez-y maintenant, même pour un projet d'apprentissage.

### Vérifier que le projet vierge démarre

```bash
flutter run -d chrome
```

Vous devez voir le compteur de démonstration de Flutter. Si oui, votre chaîne d'outils fonctionne, et toute erreur ultérieure viendra de **votre** code — c'est une information précieuse.

> **À retenir.** Créez le projet, puis vérifiez qu'il démarre **avant** d'ajouter Flame. On ne diagnostique jamais deux problèmes en même temps.

---

## 27.6 — Ajouter Flame (`flutter pub add flame`)

Une seule commande, exécutée à la racine du projet :

```bash
flutter pub add flame
```

**Résultat :**

```text
Resolving dependencies...
+ flame 1.38.0
+ ordered_set 8.0.0
+ vector_math 2.1.4
  collection 1.18.0
  meta 1.12.0
Changed 4 dependencies!
```

Trois choses viennent de se produire.

**Une ligne a été ajoutée dans `pubspec.yaml`**, sous `dependencies` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0
```

**Le fichier `pubspec.lock` a été mis à jour.** Il enregistre la version **exacte** retenue pour chaque paquet. C'est lui qui garantit que votre binôme obtiendra exactement les mêmes versions que vous.

**Les dépendances transitives ont été installées.** `vector_math` en particulier : c'est le paquet qui fournit `Vector2`, la classe centrale de la section 27.17.

### Épingler une version précise

Si vous voulez que ce cours reste reproductible dans un an, épinglez la version exacte :

```bash
flutter pub add flame:1.38.0
```

La ligne écrite est alors `flame: 1.38.0`, sans accent circonflexe. Nous verrons en 27.7 la différence exacte entre `1.38.0` et `^1.38.0`.

### Vérifier ce qui est installé

```bash
flutter pub deps --style=compact
```

**Résultat (extrait) :**

```text
Dart SDK 3.12.0
Flutter SDK 3.44.0
donjon_de_dart 1.0.0+1
- flame 1.38.0 [collection meta ordered_set vector_math]
- flutter 0.0.0
```

Une commande utile pour savoir si une mise à jour existe :

```bash
flutter pub outdated
```

### Erreurs classiques à cette étape

| Message | Cause | Correction |
| --- | --- | --- |
| `Could not find a file named "pubspec.yaml"` | vous n'êtes pas dans le dossier du projet | `cd donjon_de_dart` |
| `version solving failed` … `requires SDK version >=3.12.0` | Dart trop ancien | `flutter upgrade` |
| `flame_audio ... requires flame ^1.38.0` | vous avez épinglé une vieille version de `flame` | remonter `flame` |
| `Target of URI doesn't exist: 'package:flame/game.dart'` | `pub get` non exécuté après une modification manuelle | `flutter pub get` |

> **À retenir.** `flutter pub add` modifie `pubspec.yaml`, met à jour `pubspec.lock` et télécharge les paquets. Ne modifiez jamais `pubspec.lock` à la main.

---

## 27.7 — Le `pubspec.yaml` complet commenté

Voici le fichier tel qu'il doit être à la fin de ce chapitre. Chaque ligne est commentée.

```yaml
# Nom du paquet. Il devient aussi le préfixe des imports internes :
#   import 'package:donjon_de_dart/joueur.dart';
# Règles : minuscules, chiffres et underscores uniquement.
name: donjon_de_dart

# Description libre. Utilisée sur pub.dev si l'on publiait le paquet.
description: "Donjon de Dart — jeu 2D réalisé avec Flutter et Flame."

# 'none' interdit toute publication accidentelle sur pub.dev.
# À conserver pour une application ; à retirer seulement pour publier une bibliothèque.
publish_to: 'none'

# Version du jeu.
#   1.0.0  = version visible par le joueur (versionName sur Android)
#   +1     = numéro de build interne, à incrémenter à chaque envoi sur un store
version: 1.0.0+1

environment:
  # Contrainte exigée par flame 1.38.0.
  sdk: ">=3.12.0 <4.0.0"
  # Contrainte exigée par flame 1.38.0.
  flutter: ">=3.44.0"

dependencies:
  # Le SDK Flutter lui-même. Aucun numéro de version : il vient de votre installation.
  flutter:
    sdk: flutter

  # LE MOTEUR DE JEU.
  # ^1.38.0 signifie : "au moins 1.38.0, et strictement moins que 2.0.0".
  # Autrement dit : les corrections de bug et les ajouts sont acceptés,
  # les ruptures d'API majeures ne le sont pas.
  flame: ^1.38.0

  # Ponts à ajouter PLUS TARD, quand le chapitre correspondant arrive.
  # Laissez-les en commentaire pour l'instant.
  # flame_audio: ^2.12.2      # chapitre 34 — sons et musique
  # flame_tiled: ^3.1.2       # chapitre 34 — cartes Tiled
  # flame_forge2d: ^0.19.3+7  # chapitre 34 — physique rigide

dev_dependencies:
  # Outils de test. Non embarqués dans l'application finale.
  flutter_test:
    sdk: flutter
  # Règles d'analyse statique. La version dépend de votre modèle de projet.
  flutter_lints: ^6.0.0

flutter:
  # Active les polices d'icônes Material. Utile si vous mettez des widgets Flutter
  # autour du jeu (menus, boutons). Sans coût si vous ne les utilisez pas.
  uses-material-design: true

  # DÉCLARATION DES RESSOURCES.
  # Aucun fichier n'est embarqué s'il n'est pas listé ici.
  # Les dossiers ci-dessous sont ceux que Flame attend PAR DÉFAUT.
  assets:
    - assets/images/   # Flame.images.load('x.png')  -> assets/images/x.png
    - assets/audio/    # FlameAudio.play('x.mp3')    -> assets/audio/x.mp3
    - assets/tiles/    # TiledComponent.load('x.tmx')-> assets/tiles/x.tmx
```

### Le détail des contraintes de version

C'est une source d'erreurs constante. Voici la règle exacte.

| Écriture | Signification | Quand l'utiliser |
| --- | --- | --- |
| `flame: ^1.38.0` | `>=1.38.0 <2.0.0` | cas normal, recommandé |
| `flame: 1.38.0` | exactement 1.38.0 | pour reproduire un cours à l'identique |
| `flame: any` | n'importe quelle version | jamais |
| `flame: ">=1.38.0 <1.40.0"` | intervalle explicite | contournement temporaire d'un bug |

En versionnage sémantique, un numéro se lit `MAJEUR.MINEUR.CORRECTIF` :

```text
   1  .  38  .  0
   │     │      └── correctif : correction de bug, aucune API modifiée
   │     └───────── mineur : ajouts d'API, compatible avec l'existant
   └─────────────── majeur : ruptures possibles, du code peut cesser de compiler
```

Attention à l'exception documentée de Dart : pour une version `0.x.y`, l'accent circonflexe est plus strict. `^0.19.3` signifie `>=0.19.3 <0.20.0`. C'est pour cela que `flame_forge2d`, encore en `0.x`, demande une attention particulière.

### Les dossiers d'assets

Créez-les dès maintenant, même vides. Un dossier déclaré dans `pubspec.yaml` mais absent du disque fait **échouer la compilation**.

```bash
mkdir -p assets/images assets/audio assets/tiles
```

Sur certains systèmes, un dossier totalement vide n'est pas conservé par Git. Ajoutez-y un fichier vide :

```bash
touch assets/images/.gitkeep assets/audio/.gitkeep assets/tiles/.gitkeep
```

**Rappel du chapitre 16 :** l'indentation en YAML se fait **avec des espaces**, jamais avec des tabulations. Une tabulation dans un `pubspec.yaml` produit une erreur d'analyse difficile à lire.

> **À retenir.** Le `pubspec.yaml` est le contrat du projet : versions, dépendances, ressources. Trois quarts des problèmes de démarrage d'un jeu Flame se règlent dans ce fichier.

---

## 27.8 — Attention aux versions : pourquoi les tutoriels en ligne sont souvent périmés

Vous allez chercher de l'aide en ligne. C'est normal et souhaitable. Mais Flame a beaucoup changé, et **la majorité des tutoriels indexés par les moteurs de recherche ne compilent plus**.

### Une histoire rapide des ruptures

```text
  2019-2021   Flame 0.x        API instable, BaseGame, Camera simple
  2021        Flame 1.0        refonte totale : FlameGame remplace BaseGame
  2022        Flame 1.5+       nouveau système de caméra : CameraComponent
  2022-2023   Flame 1.6-1.10   TapCallbacks remplace Tappable / HasTappables
  2024        Flame 1.31       suppression de shrinkWrap sur GameWidget
  2025        Flame 1.36       HitTestBehavior, ComponentPool, correctifs hitbox
  2026        Flame 1.38       refonte de la construction des détecteurs de gestes
```

Un article écrit en 2021 utilise donc `BaseGame`, une classe **supprimée**. Un article de 2022 utilise `camera.followComponent()`, une méthode **supprimée**. Un article de 2023 utilise `HasTappables`, un mixin **supprimé**.

### Table de conversion des API périmées

Gardez cette table sous la main. Elle vous fera gagner des heures.

| Ce que vous lisez dans un vieux tutoriel | Réalité en 1.38.0 | À écrire aujourd'hui |
| --- | --- | --- |
| `class MonJeu extends BaseGame` | supprimé depuis la 1.0 | `class MonJeu extends FlameGame` |
| `game.add(monComposant)` pour un décor | fonctionne mais court-circuite la caméra | `game.world.add(monComposant)` |
| `game.camera.zoom = 2` | n'existe pas sur `CameraComponent` | `game.camera.viewfinder.zoom = 2` |
| `game.camera.followComponent(j)` | n'existe plus | `game.camera.follow(j)` |
| `game.camera.worldBounds = rect` | n'existe plus | `game.camera.setBounds(shape)` |
| `with HasTappables` sur le jeu | non exporté | rien sur le jeu ; `TapCallbacks` sur le composant |
| `with Tappable` sur un composant | non exporté | `with TapCallbacks` |
| `bool onTapDown(TapDownInfo info)` | ancienne signature | `void onTapDown(TapDownEvent event)` |
| `with TapDetector` sur le jeu | le mixin n'existe plus | `TapCallbacks` sur un composant |
| `with HasGameRef<MonJeu>` + `gameRef` | déprécié, suppression annoncée | `with HasGameReference<MonJeu>` + `game` |
| `GameWidget(game: g, shrinkWrap: true)` | supprimé en 1.31.0 | encadrer par un `SizedBox` ou `Expanded` |
| `onKeyEvent(RawKeyEvent event, ...)` | Flame suit l'API `KeyEvent` de Flutter | `onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed)` |
| `Camera` importée depuis `package:flame/game.dart` | classe supprimée | `CameraComponent` depuis `package:flame/camera.dart` |

### Comment repérer un tutoriel périmé en dix secondes

Trois réflexes, dans cet ordre.

**1. Cherchez `BaseGame` dans la page.** S'il y est, l'article a plus de cinq ans. Fermez.

**2. Cherchez `gameRef`.** S'il y est sans `HasGameReference`, l'article a au moins deux ans. Méfiance maximale.

**3. Regardez la date de publication et la version citée.** Un article qui ne cite aucun numéro de version de Flame est inutilisable, quelle que soit sa qualité.

### Les trois sources fiables

| Source | Adresse | Ce qu'on y trouve |
| --- | --- | --- |
| Documentation officielle | `https://docs.flame-engine.org/latest/` | les guides, à jour avec `main` |
| Dartdoc (API générée) | `https://pub.dev/documentation/flame/latest/` | les signatures exactes, version par version |
| Changelog | `https://pub.dev/packages/flame/changelog` | ce qui a cassé, et quand |

Le dartdoc est le plus précieux. Il est **généré à partir du code**, donc il ne peut pas mentir. Quand vous doutez d'une signature, c'est là qu'il faut aller, pas sur un forum.

### Le réflexe qui sauve

Quand un exemple ne compile pas, posez-vous **dans cet ordre** :

```text
  1. Quelle version de flame est dans MON pubspec.lock ?
       -> flutter pub deps | grep flame
  2. Cette classe / méthode existe-t-elle dans le dartdoc de CETTE version ?
       -> pub.dev/documentation/flame/1.38.0/
  3. Le changelog mentionne-t-il un "BREAKING" à son sujet ?
       -> pub.dev/packages/flame/changelog
```

> **À retenir.** En Flame, la première question devant une erreur n'est pas « qu'est-ce que j'ai mal écrit ? » mais « **quelle version ce code visait-il ?** ».

---

## 27.9 — `FlameGame` : la classe de base

`FlameGame` est **la classe que vous héritez pour écrire un jeu**. Elle joue exactement le rôle de votre `MoteurDeBoucle` du chapitre 20, en beaucoup plus complet.

### La déclaration minimale

```dart
import 'package:flame/game.dart';

class DonjonDeDart extends FlameGame {
  // Rien. Ce jeu compile et tourne : il affiche un écran noir.
}
```

Ces trois lignes suffisent. Vous avez déjà une boucle de jeu qui tourne à la cadence de l'écran.

### Ce que `FlameGame` fait pour vous

Au moment où le jeu démarre, `FlameGame` construit automatiquement trois choses.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  FlameGame  (racine de l'arbre de composants)                │
  │                                                              │
  │  ├── world   : World                                         │
  │  │     Conteneur de TOUS les objets du jeu.                  │
  │  │     Ne se dessine pas lui-même : il est REGARDÉ.          │
  │  │                                                           │
  │  └── camera  : CameraComponent                               │
  │        ├── backdrop    : décor fixe DERRIÈRE le monde        │
  │        ├── viewfinder  : quelle partie du monde on regarde   │
  │        │                 (zoom, angle, anchor)               │
  │        └── viewport    : la fenêtre par laquelle on regarde  │
  │                          (accueille le HUD, fixe à l'écran)  │
  └──────────────────────────────────────────────────────────────┘
```

C'est un modèle **caméra / monde**, exactement celui de la section 25 : le monde a ses propres coordonnées, la caméra décide de ce qu'on en voit. Vous en aviez écrit une version manuelle avec `canvas.translate(-camX, -camY)`.

La signature réelle du constructeur, vérifiée dans le code source, est :

```dart
class FlameGame<W extends World> extends ComponentTreeRoot with Game {
  FlameGame({
    super.children,
    W? world,
    CameraComponent? camera,
  });
}
```

Vous pouvez donc fournir votre propre monde ou votre propre caméra, mais **par défaut Flame les crée pour vous**.

### Les membres de `FlameGame` que vous utiliserez dès ce chapitre

| Membre | Type | Rôle |
| --- | --- | --- |
| `world` | `W` (par défaut `World`) | conteneur des objets du jeu |
| `camera` | `CameraComponent` | ce qui rend le monde visible |
| `size` | `Vector2` (lecture seule) | taille logique de la zone de jeu |
| `canvasSize` | `Vector2` (lecture seule) | taille du canvas réel |
| `paused` | `bool` | état de pause du moteur |
| `debugMode` | `bool` | affichage des boîtes de debug |
| `images` | `Images` | cache d'images (chapitre 29) |
| `assets` | `AssetsCache` | cache des autres ressources |
| `overlays` | `OverlayManager` | widgets Flutter au-dessus du jeu (chapitre 35) |
| `hasLayout` | `bool` (lecture seule) | vrai si le jeu est branché à un `GameWidget` vivant |

Et les méthodes :

| Méthode | Signature | Rôle |
| --- | --- | --- |
| `onLoad` | `FutureOr<void> onLoad()` | chargement asynchrone, une seule fois |
| `onMount` | `void onMount()` | montage dans l'arbre |
| `onRemove` | `void onRemove()` | nettoyage |
| `update` | `void update(double dt)` | logique, une fois par frame |
| `render` | `void render(Canvas canvas)` | dessin, une fois par frame |
| `onGameResize` | `void onGameResize(Vector2 size)` | redimensionnement |
| `backgroundColor` | `Color backgroundColor()` | couleur de fond |
| `pauseEngine` | `void pauseEngine()` | arrête la boucle |
| `resumeEngine` | `void resumeEngine()` | relance la boucle |
| `stepEngine` | `void stepEngine({double stepTime = 1 / 60})` | avance d'une frame, en pause |

### Un premier jeu qui affiche quelque chose

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget<DonjonDeDart>(game: DonjonDeDart()));
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le repère du monde démarre en haut à gauche de l'écran.
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(
      RectangleComponent(
        position: Vector2(60, 60),
        size: Vector2(48, 48),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }
}
```

**Résultat :**

```text
Un carré doré de 48x48 pixels, à 60 pixels du bord haut et du bord gauche,
sur un fond bleu nuit. Rien ne bouge encore.
```

Vous venez d'écrire un jeu Flame complet en vingt lignes. Comparez mentalement avec le fichier du chapitre 20 : il en faisait cent vingt pour afficher un carré immobile.

> **À retenir.** `FlameGame` est la racine. Elle possède un `world` et une `camera`. Les objets du jeu vont dans `world`, pas directement dans le jeu.

---

## 27.10 — `GameWidget` : le pont entre Flutter et Flame

Un `FlameGame` **n'est pas un widget**. C'est un objet Dart ordinaire. Pour l'afficher, il faut un widget qui sache le dessiner : c'est `GameWidget`.

```text
  ┌────────────────────────────────────────────────────────────┐
  │   MONDE FLUTTER : arbre de widgets                         │
  │                                                            │
  │   MaterialApp                                              │
  │    └── Scaffold                                            │
  │         └── Column                                         │
  │              ├── AppBar / Text / boutons  (widgets)        │
  │              └── Expanded                                  │
  │                   └── GameWidget  ◄── LA FRONTIÈRE         │
  │                        ╔═══════════════════════════════╗   │
  │                        ║ MONDE FLAME : arbre de        ║   │
  │                        ║ composants                    ║   │
  │                        ║  FlameGame                    ║   │
  │                        ║   ├── world (Joueur, Gobelin) ║   │
  │                        ║   └── camera                  ║   │
  │                        ╚═══════════════════════════════╝   │
  └────────────────────────────────────────────────────────────┘
```

Ce que fait `GameWidget`, concrètement :

- il **mesure** l'espace disponible et transmet cette taille au jeu, ce qui déclenche `onGameResize` ;
- il **démarre la boucle** de jeu (un `Ticker` Flutter) et appelle `update` puis `render` ;
- il **affiche un indicateur de chargement** tant que `onLoad` n'a pas terminé ;
- il **capte les entrées** (clavier, souris, tactile) et les transmet au jeu ;
- il **héberge les overlays**, des widgets Flutter dessinés au-dessus du canvas.

### Le constructeur réel

Vérifié dans le code source de la 1.38.0 :

```dart
GameWidget({
  required T game,
  TextDirection? textDirection,
  GameLoadingWidgetBuilder? loadingBuilder,
  GameErrorWidgetBuilder? errorBuilder,
  WidgetBuilder? backgroundBuilder,
  Map<String, OverlayWidgetBuilder<T>>? overlayBuilderMap,
  List<String>? initialActiveOverlays,
  FocusNode? focusNode,
  bool autofocus = true,
  MouseCursor? mouseCursor,
  bool addRepaintBoundary = true,
  HitTestBehavior behavior = HitTestBehavior.opaque,
  Key? key,
});
```

Les paramètres que vous emploierez dans ce chapitre :

| Paramètre | Type | Usage |
| --- | --- | --- |
| `game` | `T` | l'instance de jeu à afficher. Obligatoire. |
| `loadingBuilder` | fonction | ce qu'on affiche pendant `onLoad` |
| `errorBuilder` | fonction | ce qu'on affiche si `onLoad` lève une exception |
| `backgroundBuilder` | `WidgetBuilder` | un décor Flutter derrière le canvas |
| `autofocus` | `bool` | `true` par défaut : le jeu capte le clavier au montage |
| `focusNode` | `FocusNode?` | pour piloter le focus finement (chapitre 30) |

### Un `GameWidget` avec chargement et erreur

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(
      game: DonjonDeDart(),
      // Affiché tant que onLoad() n'a pas terminé.
      loadingBuilder: (BuildContext context) => const ColoredBox(
        color: Color(0xFF1B1B2A),
        child: Center(
          child: Text(
            'Chargement du donjon...',
            style: TextStyle(color: Color(0xFFE8B04B), fontSize: 18),
          ),
        ),
      ),
      // Affiché si onLoad() lève une exception.
      errorBuilder: (BuildContext context, Object error) => ColoredBox(
        color: const Color(0xFF3A1010),
        child: Center(
          child: Text(
            'Le donjon a refusé de s ouvrir :\n$error',
            textAlign: TextAlign.center,
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);
}
```

**Résultat :**

```text
Un écran bleu nuit. Sur une machine rapide, le message de chargement
n'apparaît pas : onLoad() ne fait rien et se termine immédiatement.
```

### Le piège de la taille

`GameWidget` prend **toute la place disponible**. Si vous le placez dans une `Column` sans contrainte de hauteur, Flutter lève une erreur de layout.

```dart
// NE FAITES PAS CELA : hauteur infinie, erreur de layout.
Column(
  children: [
    const Text('Score'),
    GameWidget(game: DonjonDeDart()),
  ],
)
```

```dart
// Version correcte : on contraint explicitement.
Column(
  children: [
    const Text('Score'),
    Expanded(child: GameWidget(game: DonjonDeDart())),
  ],
)
```

```dart
// Autre version correcte : une taille fixe.
SizedBox(
  width: 320,
  height: 240,
  child: GameWidget(game: DonjonDeDart()),
)
```

Le paramètre `shrinkWrap` que citent les tutoriels de 2023 a été **supprimé en Flame 1.31.0**. Il n'existe plus. Utilisez `Expanded` ou `SizedBox`.

> **À retenir.** `GameWidget` est la frontière. À gauche, Flutter et ses widgets. À droite, Flame et ses composants. Un seul `GameWidget` suffit pour tout un jeu.

---

## 27.11 — Le `main.dart` minimal qui affiche un jeu

Voici le fichier complet. Copiez-le dans `lib/main.dart` de votre projet `donjon_de_dart` et lancez-le : il fonctionne tel quel.

```dart
// lib/main.dart
//
// Donjon de Dart — chapitre 27, squelette minimal.
// Flame 1.38.0

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  // WidgetsFlutterBinding est initialisé par runApp : rien à faire ici
  // tant que l'on n'appelle pas d'API de plateforme avant runApp.
  runApp(const DonjonApp());
}

/// L'application Flutter qui héberge le jeu.
class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Donjon de Dart',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        // Le GameWidget occupe tout le corps du Scaffold.
        body: GameWidget<DonjonDeDart>(game: DonjonDeDart()),
      ),
    );
  }
}

/// Le jeu proprement dit.
class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Repère du monde aligné sur le coin haut-gauche de l'écran :
    // c'est le repère du chapitre 21, celui que vous connaissez.
    camera.viewfinder.anchor = Anchor.topLeft;

    // Le sol du donjon.
    await world.add(
      RectangleComponent(
        position: Vector2(0, 300),
        size: Vector2(2000, 60),
        paint: Paint()..color = const Color(0xFF3E3A52),
      ),
    );

    // Le héros.
    await world.add(
      RectangleComponent(
        position: Vector2(80, 252),
        size: Vector2(48, 48),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );

    // Un gobelin.
    await world.add(
      RectangleComponent(
        position: Vector2(320, 268),
        size: Vector2(32, 32),
        paint: Paint()..color = const Color(0xFF6FA65A),
      ),
    );
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────────┐
  │                                                      │  fond 0xFF1B1B2A
  │                                                      │
  │      ██                                              │  héros doré 48x48
  │      ██                        ▓▓                    │  gobelin vert 32x32
  │  ████████████████████████████████████████████████    │  sol gris violet
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

### Lecture du fichier, ligne à ligne

| Ligne | Ce qu'elle fait |
| --- | --- |
| `import 'package:flame/components.dart';` | apporte `Component`, `PositionComponent`, `RectangleComponent`, `Anchor`, `Vector2` |
| `import 'package:flame/game.dart';` | apporte `FlameGame` et `GameWidget` |
| `import 'package:flutter/material.dart';` | apporte `runApp`, `MaterialApp`, `Scaffold`, `Color`, `Paint`, `Canvas` |
| `runApp(const DonjonApp())` | démarre l'application Flutter (chapitre 19) |
| `GameWidget<DonjonDeDart>(game: ...)` | insère le jeu dans l'arbre de widgets |
| `extends FlameGame` | fait de la classe un jeu, avec sa boucle |
| `backgroundColor()` | remplace le noir par défaut |
| `camera.viewfinder.anchor = Anchor.topLeft` | met l'origine du monde en haut à gauche |
| `world.add(...)` | ajoute un objet au monde, donc visible par la caméra |

### Les deux imports à connaître

Flame est découpé en fichiers d'export thématiques. Voici ceux dont vous aurez besoin dans les prochains chapitres :

```dart
import 'package:flame/game.dart';        // FlameGame, GameWidget
import 'package:flame/components.dart';  // Component, PositionComponent, Vector2, Anchor...
import 'package:flame/events.dart';      // TapCallbacks, DragCallbacks...      (ch. 30)
import 'package:flame/input.dart';       // JoystickComponent, HudButtonComponent (ch. 30)
import 'package:flame/camera.dart';      // CameraComponent, World, Viewport     (ch. 31)
import 'package:flame/collisions.dart';  // HasCollisionDetection, hitbox        (ch. 32)
import 'package:flame/effects.dart';     // MoveEffect, ScaleEffect...           (ch. 33)
import 'package:flame/geometry.dart';    // tau, utilitaires géométriques
import 'package:flame/flame.dart';       // Flame.images, Flame.assets, Flame.device
```

**Remarque.** Si vous importez à la fois `package:flutter/material.dart` et certains fichiers de Flame, le compilateur peut signaler un conflit sur `Image` ou `Animation`. La solution habituelle est de masquer les noms en trop :

```dart
import 'package:flutter/widgets.dart' hide Animation, Image;
```

> **À retenir.** Un jeu Flame minimal, c'est `runApp` + `GameWidget` + une classe qui hérite de `FlameGame`. Tout le reste est de l'enrichissement.

---

## 27.12 — `onLoad()` : le chargement asynchrone (rappel chapitre 15)

`onLoad()` est le **constructeur asynchrone** de tout composant Flame, y compris du jeu lui-même.

### Pourquoi un constructeur ne suffit pas

Un constructeur Dart est **synchrone**. Il ne peut pas attendre. Or, charger une image, un son ou une carte prend du temps et renvoie un `Future` — c'est tout le chapitre 15.

```dart
class Joueur extends PositionComponent {
  Joueur() {
    // IMPOSSIBLE : un constructeur ne peut pas être async,
    // et on ne peut donc pas y écrire await.
    // sprite = await Sprite.load('heros.png');
  }
}
```

Flame résout le problème en appelant, après la construction, une méthode que vous pouvez rendre asynchrone :

```dart
class Joueur extends PositionComponent {
  Joueur();

  @override
  Future<void> onLoad() async {
    // Ici, await est autorisé.
    // sprite = await Sprite.load('heros.png');  // chapitre 29
  }
}
```

### La signature réelle

```dart
FutureOr<void> onLoad()
```

`FutureOr<void>` signifie : « soit un `Future<void>`, soit rien du tout ». Vous avez donc **deux écritures légitimes**, selon que vous avez besoin d'`await` ou non.

```dart
// Variante 1 : rien d'asynchrone à faire.
@override
void onLoad() {
  anchor = Anchor.center;
}

// Variante 2 : il y a un await quelque part.
@override
Future<void> onLoad() async {
  await super.onLoad();
  await world.add(Joueur());
}
```

### `super.onLoad()` : quand est-ce obligatoire ?

Règle simple et sûre, à appliquer sans réfléchir : **appelez toujours `super.onLoad()` en première ligne**.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();  // TOUJOURS en premier
  // ... votre code
}
```

La raison : la classe parente peut avoir des initialisations à faire. Sur `FlameGame`, elle installe le `world` et la `camera`. Sur un composant qui hérite d'un composant de Flame (par exemple `SpriteComponent`), elle peut préparer des ressources. Oublier `super.onLoad()` produit des bugs déroutants : un composant qui n'apparaît pas, une caméra qui ne suit rien, une taille restée à zéro.

### Le retour de `add()`

C'est le point qui surprend le plus.

```dart
FutureOr<void> add(Component component);   // sur Component
Future<void>   addAll(Iterable<Component> components);
```

`add()` renvoie un `FutureOr<void>`. Le composant **n'est pas monté immédiatement** : Flame place la demande dans une file et la traite au bon moment de la frame. Deux conséquences.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  final Joueur joueur = Joueur();

  // Cas 1 : on n'a pas besoin du composant tout de suite.
  world.add(joueur);          // pas d'await : c'est correct

  // Cas 2 : la suite dépend du montage effectif.
  await world.add(joueur);    // avec await : joueur est monté après cette ligne
  camera.follow(joueur);      // maintenant sûr
}
```

Dans le doute, mettez `await`. Le coût est nul et le comportement devient déterministe.

### Les accesseurs d'état

Flame expose l'avancement du cycle de vie, pour les cas où vous en avez besoin :

| Getter | Type | Sens |
| --- | --- | --- |
| `isLoaded` | `bool` | `onLoad()` est terminé |
| `loaded` | `Future<void>` | se complète quand `onLoad()` est terminé |
| `isMounted` | `bool` | le composant est dans l'arbre |
| `mounted` | `Future<void>` | se complète au montage |
| `isRemoved` | `bool` | le composant a été retiré |
| `removed` | `Future<void>` | se complète au retrait |

### Exemple complet : un chargement volontairement lent

Ce programme montre le `loadingBuilder` en action. Il n'a pas besoin d'assets.

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(
      game: DonjonDeDart(),
      loadingBuilder: (BuildContext context) => const ColoredBox(
        color: Color(0xFF1B1B2A),
        child: Center(
          child: Text(
            'Ouverture du donjon...',
            style: TextStyle(color: Color(0xFFE8B04B), fontSize: 20),
          ),
        ),
      ),
    );
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Simulation d'un chargement long (chapitre 15 : Future.delayed).
    await Future<void>.delayed(const Duration(seconds: 2));

    await world.add(
      RectangleComponent(
        position: Vector2(60, 60),
        size: Vector2(64, 64),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }
}
```

**Résultat :**

```text
0 s -> 2 s : « Ouverture du donjon... » sur fond bleu nuit.
2 s        : le message disparaît, un carré doré apparaît en (60, 60).
```

> **À retenir.** `onLoad()` est le seul endroit où l'on a le droit d'attendre. Tout ce qui charge quelque chose y va. Et la première ligne est toujours `await super.onLoad();`.

---

## 27.13 — `onMount()` et `onRemove()`

`onLoad()` n'est pas le seul point d'entrée. Le cycle de vie complet d'un composant Flame est le suivant, tel que le décrit la documentation officielle :

```text
  constructeur
      │
      ▼
  onGameResize(Vector2 size)      ◄─ aussi à CHAQUE redimensionnement
      │
      ▼
  onLoad()                        ◄─ UNE SEULE FOIS dans la vie du composant
      │
      ▼
  onMount()                       ◄─ à CHAQUE montage dans l'arbre
      │
      ▼
  ┌──────────────────────┐
  │  update(dt)          │  ◄─ une fois par frame
  │  render(canvas)      │
  └──────────────────────┘
      │
      ▼
  onRemove()                      ◄─ avant le retrait de l'arbre
```

### Les trois différences à retenir

| Méthode | Combien de fois ? | Asynchrone ? | À quoi ça sert |
| --- | --- | --- | --- |
| `onLoad()` | une seule fois, jamais rejouée | oui (`FutureOr<void>`) | charger des ressources coûteuses |
| `onMount()` | à chaque entrée dans l'arbre | non (`void`) | (re)initialiser l'état de départ |
| `onRemove()` | une fois, à la sortie | non (`void`) | libérer, désabonner, arrêter |

Le point le plus subtil : un composant peut être **retiré puis remis** dans l'arbre. Dans ce cas, `onLoad()` n'est **pas** rejoué — la ressource est déjà chargée — mais `onMount()` l'est.

### Démonstration

Ce programme trace le cycle de vie dans la console. Il n'a besoin d'aucun asset.

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  late final Torche torche;
  double _chrono = 0;
  int _etape = 0;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    torche = Torche();
    await world.add(torche);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _chrono += dt;

    // À 2 s : on retire la torche. À 4 s : on la remet.
    if (_etape == 0 && _chrono >= 2) {
      _etape = 1;
      torche.removeFromParent();
    } else if (_etape == 1 && _chrono >= 4) {
      _etape = 2;
      world.add(torche);
    }
  }
}

class Torche extends PositionComponent {
  Torche() : super(position: Vector2(80, 80), size: Vector2(40, 90));

  static final Paint _peinture = Paint()..color = const Color(0xFFE8B04B);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    debugPrint('onLoad   : une seule fois, chargement des ressources');
  }

  @override
  void onMount() {
    super.onMount();
    debugPrint('onMount  : la torche entre dans l arbre');
  }

  @override
  void onRemove() {
    debugPrint('onRemove : la torche quitte l arbre');
    super.onRemove();
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Résultat (console) :**

```text
onLoad   : une seule fois, chargement des ressources
onMount  : la torche entre dans l arbre
onRemove : la torche quitte l arbre
onMount  : la torche entre dans l arbre
```

Observez bien : `onLoad` n'apparaît **qu'une fois**, alors que `onMount` apparaît **deux fois**. C'est exactement le comportement annoncé.

**Résultat (écran) :**

```text
0 s -> 2 s : une barre dorée verticale en (80, 80).
2 s -> 4 s : rien.
4 s -> ... : la barre dorée est de retour, au même endroit.
```

### Où mettre quoi

| Ce que vous voulez faire | Méthode |
| --- | --- |
| Charger un sprite, un son, une carte | `onLoad()` |
| Créer les sous-composants permanents | `onLoad()` |
| Remettre les points de vie à leur maximum | `onMount()` |
| Repositionner l'entité à son point de départ | `onMount()` |
| S'abonner à un bus d'événements (chapitre 26) | `onMount()` |
| Se désabonner du bus | `onRemove()` |
| Arrêter un son en boucle | `onRemove()` |
| Rendre un objet à un pool d'objets | `onRemove()` |

### `removeFromParent()`

C'est la méthode qui retire un composant de l'arbre. Elle est **différée** : le composant est retiré à un moment sûr de la frame, jamais au milieu d'une itération.

```dart
@override
void update(double dt) {
  super.update(dt);
  if (pointsDeVie <= 0) {
    removeFromParent();   // sûr, même au milieu d'un update
  }
}
```

Souvenez-vous de la section 26.7 : vous aviez dû écrire vous-même une file `_aRetirer` pour éviter l'exception `Concurrent modification during iteration`. Flame fait exactement la même chose, en interne, et vous n'avez plus à y penser.

> **À retenir.** `onLoad` charge une fois. `onMount` initialise à chaque entrée. `onRemove` nettoie. Et `removeFromParent()` est toujours sûr.

---

## 27.14 — `update(double dt)` : le `dt` est le même qu'au chapitre 20

Voici probablement la meilleure nouvelle du chapitre : **vous n'avez rien de nouveau à apprendre ici**.

```dart
void update(double dt)
```

`dt` est un `double`, il vaut le **temps écoulé depuis la frame précédente**, **en secondes**. C'est mot pour mot la définition du chapitre 20.

```text
  60 images par seconde  ->  dt ≈ 0.01667 s
 120 images par seconde  ->  dt ≈ 0.00833 s
  30 images par seconde  ->  dt ≈ 0.03333 s
```

### La règle d'or, inchangée

Toute vitesse s'exprime en **unités par seconde**, et se multiplie par `dt`.

```dart
// FAUX : dépendant du framerate. Deux fois plus rapide sur un écran 120 Hz.
position.x += 3;

// JUSTE : 180 pixels par seconde, quelle que soit la machine.
position.x += 180 * dt;
```

### Le squelette d'un `update` correct

```dart
@override
void update(double dt) {
  super.update(dt);   // OBLIGATOIRE : propage le tick aux enfants
  // ... votre logique
}
```

`super.update(dt)` parcourt les enfants du composant et appelle leur `update`. Si vous l'oubliez sur le jeu, **plus rien ne bouge** : ni le monde, ni la caméra, ni les effets, ni les timers. C'est l'erreur numéro un des débutants sous Flame, et son symptôme — « mon jeu est figé » — n'oriente pas vers la cause.

### Placement de `super.update(dt)`

Convention : en **première ligne**. Ainsi vos enfants sont à jour avant que vous ne lisiez leur état.

```dart
@override
void update(double dt) {
  super.update(dt);            // 1. les enfants avancent
  _verifierSorties();          // 2. je réagis à leur nouvel état
}
```

### Vérifier soi-même que `dt` est bien en secondes

Ce petit jeu affiche le temps total écoulé et le compare à votre montre.

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  double _tempsTotal = 0;
  int _frames = 0;

  late final TextComponent _affichage;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    _affichage = TextComponent(
      text: 'demarrage',
      position: Vector2(16, 16),
      anchor: Anchor.topLeft,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 18, color: Color(0xFFE8B04B)),
      ),
    );
    await camera.viewport.add(_affichage);
  }

  @override
  void update(double dt) {
    super.update(dt);

    _tempsTotal += dt;
    _frames++;

    final double moyenne = _frames == 0 ? 0 : _tempsTotal / _frames;
    _affichage.text = 'temps : ${_tempsTotal.toStringAsFixed(2)} s\n'
        'frames : $_frames\n'
        'dt moyen : ${(moyenne * 1000).toStringAsFixed(2)} ms\n'
        'FPS moyen : ${(1 / (moyenne == 0 ? 1 : moyenne)).toStringAsFixed(1)}';
  }
}
```

**Résultat (après une dizaine de secondes, sur un écran 60 Hz) :**

```text
temps : 10.03 s
frames : 602
dt moyen : 16.66 ms
FPS moyen : 60.0
```

Le compteur « temps » avance à la même vitesse que votre montre : `dt` est bien en secondes.

### Le pic de `dt`

Comme au chapitre 20, un `dt` anormalement grand peut apparaître : après une pause, un changement d'onglet, un chargement. Un objet rapide peut alors traverser un mur en une frame — le *tunneling* de la section 24. La parade est la même :

```dart
@override
void update(double dt) {
  // On plafonne : au-delà de 50 ms, on fait comme si la frame durait 50 ms.
  final double pas = dt > 0.05 ? 0.05 : dt;
  super.update(pas);
}
```

> **À retenir.** `dt` en Flame est le `dt` du chapitre 20 : des secondes, dans un `double`. Multipliez toujours vos vitesses par `dt`, et n'oubliez jamais `super.update(dt)`.

---

## 27.15 — `render(Canvas canvas)` : le `Canvas` est celui du chapitre 21

Deuxième bonne nouvelle : le `Canvas` de Flame **est** le `Canvas` de Flutter. Le même objet, la même classe `dart:ui`, les mêmes méthodes.

```dart
void render(Canvas canvas)
```

Tout ce que vous avez appris au chapitre 21 s'applique sans la moindre modification :

```dart
canvas.drawRect(rect, paint);
canvas.drawCircle(centre, rayon, paint);
canvas.drawLine(p1, p2, paint);
canvas.drawPath(path, paint);
canvas.drawImage(image, offset, paint);
canvas.save();
canvas.translate(dx, dy);
canvas.rotate(radians);
canvas.scale(sx, sy);
canvas.restore();
```

### La différence essentielle : le repère est déjà placé

Au chapitre 21, vous dessiniez tout dans le repère de l'écran, et vous faisiez vous-même les translations.

Sous Flame, quand `render` d'un `PositionComponent` est appelé, **le canvas a déjà été translaté, pivoté et mis à l'échelle** selon la `position`, l'`angle` et le `scale` du composant. Vous dessinez donc en **coordonnées locales**, avec l'origine sur l'ancre du composant.

```text
  CHAPITRE 21 (Flutter pur)              FLAME
  ────────────────────────────           ────────────────────────────
  canvas.drawRect(                       canvas.drawRect(
    Rect.fromLTWH(x, y, l, h),             size.toRect(),   // 0,0,l,h
    peinture,                              peinture,
  );                                     );
       ▲                                        ▲
       │                                        │
  je dois placer moi-même             Flame a déjà placé le repère
```

Concrètement, dans un `PositionComponent` d'ancre `Anchor.topLeft` :

```dart
@override
void render(Canvas canvas) {
  super.render(canvas);
  // (0, 0) est le coin haut-gauche du composant.
  // size.toRect() vaut Rect.fromLTWH(0, 0, size.x, size.y).
  canvas.drawRect(size.toRect(), _peinture);
}
```

Si vous écriviez `canvas.drawRect(Rect.fromLTWH(position.x, position.y, ...), ...)`, la position serait **appliquée deux fois** et l'objet apparaîtrait deux fois plus loin. C'est une erreur classique.

### `super.render(canvas)`

Comme pour `update`, `super.render(canvas)` dessine les **enfants** du composant. Sans lui, un composant qui contient d'autres composants n'affiche que lui-même.

Convention : `super.render(canvas)` en **première ligne** signifie que les enfants sont dessinés **avant** votre dessin, donc **derrière**. Si vous voulez le contraire, placez-le en dernier. Dans la pratique, on préfère régler l'ordre avec `priority` (chapitre 28).

### Un composant qui dessine plusieurs formes

```dart
// lib/main.dart
import 'dart:math' as math;

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Coffre(position: Vector2(120, 120)));
  }
}

/// Un coffre dessiné entièrement à la main, sans aucun asset.
class Coffre extends PositionComponent {
  Coffre({super.position}) : super(size: Vector2(96, 72));

  static final Paint _bois = Paint()..color = const Color(0xFF8A5A2B);
  static final Paint _ferrure = Paint()..color = const Color(0xFF4A4A55);
  static final Paint _serrure = Paint()..color = const Color(0xFFE8B04B);
  static final Paint _contour = Paint()
    ..color = const Color(0xFF2A1A0A)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3;

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    // Corps du coffre : coordonnées LOCALES, origine au coin haut-gauche.
    final Rect corps = Rect.fromLTWH(0, size.y * 0.35, size.x, size.y * 0.65);
    canvas.drawRect(corps, _bois);
    canvas.drawRect(corps, _contour);

    // Couvercle bombé.
    final Rect couvercle = Rect.fromLTWH(0, 0, size.x, size.y * 0.5);
    canvas.drawArc(couvercle, math.pi, math.pi, true, _bois);
    canvas.drawArc(couvercle, math.pi, math.pi, true, _contour);

    // Deux ferrures verticales.
    canvas.drawRect(Rect.fromLTWH(size.x * 0.18, 0, 8, size.y), _ferrure);
    canvas.drawRect(Rect.fromLTWH(size.x * 0.74, 0, 8, size.y), _ferrure);

    // Serrure dorée, centrée horizontalement.
    canvas.drawCircle(Offset(size.x / 2, size.y * 0.52), 8, _serrure);
    canvas.drawCircle(Offset(size.x / 2, size.y * 0.52), 8, _contour);
  }
}
```

**Résultat :**

```text
Un coffre de 96x72 pixels dessiné en (120, 120) :
couvercle arrondi, corps rectangulaire brun, deux ferrures grises
verticales, une serrure dorée cerclée de noir au centre.
Aucune image n'a été chargée : tout est du Canvas.
```

### `render` ne modifie rien

La règle de la section 26.2 reste valable, et elle est encore plus importante sous Flame : **`render` lit, il n'écrit jamais**. Un `render` qui modifie une position produit un mouvement dépendant du nombre de redessins, donc imprévisible.

> **À retenir.** Le `Canvas` de Flame est celui de Flutter, mais le repère est déjà positionné sur le composant. Dessinez en local, avec `size.toRect()`.

---

## 27.16 — `size` et `canvasSize`

`FlameGame` expose deux tailles, et les confondre est une source de bugs d'affichage.

| Propriété | Type | Ce que c'est |
| --- | --- | --- |
| `size` | `Vector2` (lecture seule) | la taille **logique** de la zone de jeu, après transformation du viewport |
| `canvasSize` | `Vector2` (lecture seule) | la taille **réelle** du canvas alloué au `GameWidget` |

La documentation de `FlameGame.size` précise : « This is overwritten to consider the viewport transformation. »

### Quand les deux valeurs sont identiques

Dans le cas par défaut, la caméra utilise un `MaxViewport` qui occupe tout le canvas. Les deux valeurs sont alors **égales**.

```text
  GameWidget de 800 x 600
  ┌────────────────────────────────────┐
  │                                    │
  │       canvasSize = (800, 600)      │
  │       size       = (800, 600)      │
  │                                    │
  └────────────────────────────────────┘
```

### Quand elles diffèrent

Dès que vous employez une caméra à résolution fixe (chapitre 31), les deux se séparent.

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

```text
  GameWidget de 1280 x 720
  ┌──────────────────────────────────────────────┐
  │                                              │
  │   canvasSize = (1280, 720)   pixels réels    │
  │   size       = ( 320, 180)   unités de jeu   │
  │                                              │
  │   Chaque unité de jeu occupe 4 pixels.       │
  └──────────────────────────────────────────────┘
```

C'est la technique du pixel art : vous raisonnez dans une petite grille, Flame agrandit.

### Lequel utiliser ?

**Utilisez `size` pour la logique du jeu.** Placer un objet au centre, détecter une sortie d'écran, positionner un HUD : `size`.

**Utilisez `canvasSize` uniquement pour des besoins liés aux pixels réels**, par exemple ajuster une épaisseur de trait ou diagnostiquer une mise à l'échelle.

```dart
// Centrer un objet : on raisonne en unités de jeu.
composant.position = size / 2;

// Détecter une sortie par la droite.
if (composant.position.x > size.x) {
  composant.removeFromParent();
}
```

### Attention : `size` n'existe pas dans le constructeur

Avant que le `GameWidget` n'ait mesuré l'espace disponible, la taille du jeu est indéfinie. Lire `size` dans le constructeur du jeu lève une erreur.

```dart
class DonjonDeDart extends FlameGame {
  DonjonDeDart() {
    // ERREUR : le jeu n'a pas encore de taille.
    // final Vector2 centre = size / 2;
  }

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // Correct : à ce stade, onGameResize a déjà été appelé.
    final Vector2 centre = size / 2;
    await world.add(
      RectangleComponent(
        position: centre,
        size: Vector2.all(40),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }
}
```

Le getter `hasLayout` permet de vérifier explicitement si le jeu est branché à un `GameWidget` vivant, donc si `size` est exploitable.

### Programme de démonstration

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  late final TextComponent _infos;
  late final RectangleComponent _cadre;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    _cadre = RectangleComponent(
      position: Vector2.zero(),
      size: size.clone(),
      paint: Paint()
        ..color = const Color(0xFFE8B04B)
        ..style = PaintingStyle.stroke
        ..strokeWidth = 4,
    );
    await world.add(_cadre);

    _infos = TextComponent(
      text: '',
      position: Vector2(16, 16),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_infos);

    _rafraichir();
  }

  void _rafraichir() {
    _infos.text = 'size       : ${size.x.toStringAsFixed(1)}'
        ' x ${size.y.toStringAsFixed(1)}\n'
        'canvasSize : ${canvasSize.x.toStringAsFixed(1)}'
        ' x ${canvasSize.y.toStringAsFixed(1)}';
  }

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);
    // isLoaded protège l'accès aux champs late avant onLoad.
    if (isLoaded) {
      _cadre.size = size.clone();
      _rafraichir();
    }
  }
}
```

**Résultat (fenêtre de 900 x 640) :**

```text
size       : 900.0 x 640.0
canvasSize : 900.0 x 640.0
```

Un cadre doré épouse exactement les bords, et suit la fenêtre quand on la redimensionne.

> **À retenir.** `size` est la taille dans laquelle vous raisonnez ; `canvasSize` est la taille en pixels. Elles ne sont pas lisibles avant `onLoad`.

---

## 27.17 — `Vector2` : le vecteur de Flame (comparaison avec le `Vec2` du chapitre 23)

Au chapitre 23, vous avez écrit votre propre classe de vecteur. Elle ressemblait à ceci :

```dart
// Votre Vec2 du chapitre 23.
class Vec2 {
  Vec2(this.x, this.y);
  double x;
  double y;

  Vec2 operator +(Vec2 autre) => Vec2(x + autre.x, y + autre.y);
  Vec2 operator *(double k) => Vec2(x * k, y * k);
  double get longueur => math.sqrt(x * x + y * y);
}
```

Flame n'utilise pas cette classe : il utilise `Vector2`, qui vient du paquet **`vector_math`** (dépendance `vector_math ^2.1.4` de `flame`) et qui est **ré-exportée** par Flame. Vous n'avez donc rien à importer de plus :

```dart
import 'package:flame/components.dart';  // Vector2 est disponible
```

### Comparaison terme à terme

| Votre `Vec2` (ch. 23) | `Vector2` (Flame) | Remarque |
| --- | --- | --- |
| `Vec2(3, 4)` | `Vector2(3, 4)` | identique |
| aucun équivalent | `Vector2.all(5)` | `(5, 5)` |
| `Vec2(0, 0)` | `Vector2.zero()` | identique |
| aucun équivalent | `Vector2.copy(autre)` | copie |
| `v.x`, `v.y` | `v.x`, `v.y` | **modifiables** dans les deux cas |
| `a + b` | `a + b` | renvoie un nouveau vecteur |
| `v * 2` | `v * 2` | renvoie un nouveau vecteur |
| `v.longueur` | `v.length` | `double` |
| aucun équivalent | `v.length2` | longueur au carré, sans racine |
| `v.normalise()` (que vous aviez écrite) | `v.normalized()` | renvoie une **copie** normalisée |
| aucun équivalent | `v.normalize()` | normalise **sur place**, renvoie l'ancienne longueur |
| `distance(a, b)` | `a.distanceTo(b)` | `double` |
| `produitScalaire(a, b)` | `a.dot(b)` | `double` |
| aucun équivalent | `a.angleTo(b)` | angle en radians |
| copie manuelle | `v.clone()` | **essentiel**, voir 27.19 |

### La différence structurante : `Vector2` est mutable

Comme votre `Vec2`, `Vector2` est **mutable** : ses champs `x` et `y` peuvent être réassignés.

```dart
final Vector2 v = Vector2(10, 20);
v.x = 99;              // autorisé, même sur un final
v.setValues(3, 4);     // remplace les deux composantes d'un coup
v.setFrom(autre);      // recopie les composantes de autre
```

Le mot-clé `final` interdit de remplacer **la variable**, pas de modifier **l'objet** qu'elle désigne. C'est le rappel du chapitre 2, et c'est ici lourd de conséquences (section 27.19).

### `Vector2` n'est pas `Offset`

C'est la confusion la plus fréquente pour qui arrive de Flutter.

| | `Vector2` (vector_math) | `Offset` (dart:ui) |
| --- | --- | --- |
| Mutable ? | **oui** | non, immuable |
| Origine | paquet `vector_math` | Flutter, `dart:ui` |
| Attendu par | toutes les API de Flame | `canvas.drawCircle`, `canvas.drawLine`… |
| Champs | `x`, `y` (modifiables) | `dx`, `dy` (en lecture seule) |

Vous croiserez donc les deux dans le même fichier : `Vector2` pour positionner un composant, `Offset` pour dessiner sur le `Canvas`. Le paquet `package:flame/extensions.dart` fournit des conversions entre les deux ; en pratique, on écrit souvent la conversion à la main :

```dart
final Vector2 v = Vector2(10, 20);
final Offset o = Offset(v.x, v.y);
```

### `Vector2` est utilisé partout dans Flame

```dart
PositionComponent(
  position: Vector2(100, 50),   // position
  size: Vector2(48, 48),        // taille
  scale: Vector2.all(2),        // échelle
);

camera.moveTo(Vector2(400, 300));
final Vector2 tailleDuJeu = game.size;
```

> **À retenir.** `Vector2` est votre `Vec2` du chapitre 23, en plus complet et en mutable. Il représente aussi bien un point, une taille, une vitesse qu'une direction.

---

## 27.18 — Les opérations sur `Vector2`

Voici les opérations que vous utiliserez réellement, classées par usage. Toutes les signatures ci-dessous sont celles du dartdoc de `vector_math`.

### Opérateurs (renvoient un NOUVEAU vecteur)

```dart
Vector2 operator +(Vector2 other)
Vector2 operator -(Vector2 other)
Vector2 operator *(double scale)
Vector2 operator /(double scale)
Vector2 operator -()          // opposé
```

```dart
final Vector2 a = Vector2(3, 4);
final Vector2 b = Vector2(1, 2);

final Vector2 somme = a + b;      // (4, 6)   a et b inchangés
final Vector2 diff = a - b;       // (2, 2)
final Vector2 double_ = a * 2;    // (6, 8)
final Vector2 moitie = a / 2;     // (1.5, 2)
final Vector2 oppose = -a;        // (-3, -4)
```

### Mutateurs (modifient le vecteur SUR PLACE, renvoient `void`)

```dart
void add(Vector2 arg)
void sub(Vector2 arg)
void multiply(Vector2 arg)   // composante par composante
void scale(double arg)
void setValues(double x_, double y_)
void setFrom(Vector2 other)
```

```dart
final Vector2 p = Vector2(10, 10);
p.add(Vector2(5, 0));       // p vaut maintenant (15, 10)
p.scale(2);                 // p vaut maintenant (30, 20)
p.setValues(0, 0);          // p vaut maintenant (0, 0)
```

Le tableau ci-dessous résume la distinction, qui est la clé pour ne pas se tromper :

| Vous écrivez | Le vecteur d'origine est… | Vous obtenez… |
| --- | --- | --- |
| `a + b` | inchangé | un nouveau vecteur |
| `a.add(b)` | **modifié** | `void` |
| `a * 2` | inchangé | un nouveau vecteur |
| `a.scale(2)` | **modifié** | `void` |
| `a.normalized()` | inchangé | un nouveau vecteur unitaire |
| `a.normalize()` | **modifié** | l'ancienne longueur (`double`) |

### Mesures (ne modifient rien)

```dart
double get length          // longueur euclidienne
double get length2         // longueur au carré
double distanceTo(Vector2 arg)
double dot(Vector2 other)
double angleTo(Vector2 other)
```

```dart
final Vector2 v = Vector2(3, 4);
print(v.length);                        // 5.0
print(v.length2);                       // 25.0
print(v.distanceTo(Vector2(0, 0)));     // 5.0
print(Vector2(1, 0).dot(Vector2(0, 1))); // 0.0  (perpendiculaires)
```

**Optimisation classique.** `length` calcule une racine carrée, `length2` non. Pour **comparer** deux distances, comparez les carrés :

```dart
// Lent : deux racines carrées par frame et par ennemi.
if (joueur.position.distanceTo(gobelin.position) < 100) { /* ... */ }

// Rapide : aucune racine carrée.
final Vector2 ecart = joueur.position - gobelin.position;
if (ecart.length2 < 100 * 100) { /* ... */ }
```

### Direction et normalisation

Un vecteur **normalisé** a une longueur de 1 : il ne porte plus que la direction.

```dart
final Vector2 depart = Vector2(100, 100);
final Vector2 cible = Vector2(400, 300);

final Vector2 direction = (cible - depart).normalized();  // longueur 1
final Vector2 vitesse = direction * 180;                  // 180 px/s vers la cible
```

Attention : normaliser le vecteur nul est indéfini. Protégez-vous.

```dart
final Vector2 ecart = cible - depart;
if (ecart.length2 > 0.0001) {
  position += ecart.normalized() * 180 * dt;
}
```

### Programme de démonstration, exécutable en console

Ce programme n'utilise ni Flutter ni Flame : il ne dépend que de `vector_math`, disponible grâce à Flame.

```dart
// Exécutable avec : dart run bin/vecteurs.dart
import 'package:vector_math/vector_math_64.dart';

void main() {
  final Vector2 heros = Vector2(100, 200);
  final Vector2 gobelin = Vector2(340, 360);

  print('heros   : $heros');
  print('gobelin : $gobelin');

  final Vector2 ecart = gobelin - heros;
  print('ecart          : $ecart');
  print('distance       : ${ecart.length.toStringAsFixed(2)}');
  print('distance au 2  : ${ecart.length2.toStringAsFixed(2)}');
  print('direction      : ${ecart.normalized()}');

  final Vector2 vitesse = ecart.normalized() * 180;
  print('vitesse 180px/s: $vitesse');

  // Un pas de simulation de 1/60 s.
  final Vector2 apresUneFrame = heros + vitesse * (1 / 60);
  print('apres 1 frame  : $apresUneFrame');

  // Mutateur : heros est modifie sur place.
  heros.add(vitesse * (1 / 60));
  print('heros modifie  : $heros');
}
```

**Résultat :**

```text
heros   : [100.0,200.0]
gobelin : [340.0,360.0]
ecart          : [240.0,160.0]
distance       : 288.44
distance au 2  : 83200.00
direction      : [0.8320502943378437,0.5547001962252291]
vitesse 180px/s: [149.76905298081186,99.84603532054124]
apres 1 frame  : [102.49615088301353,201.6641000588676]
heros modifie  : [102.49615088301353,201.6641000588676]
```

> **À retenir.** Les **opérateurs** créent, les **méthodes** modifient. Dans un `update`, préférez les mutateurs : ils évitent d'allouer un objet par frame.

---

## 27.19 — `Vector2.zero()`, `Vector2.all()`, `.clone()` : le piège des références partagées

Nous arrivons au piège qui coûte le plus de temps aux débutants sous Flame. Il découle directement du fait que `Vector2` est **mutable**.

### Le bug

```dart
// NE FAITES PAS CELA.
final Vector2 tailleCommune = Vector2(48, 48);

final Joueur joueur = Joueur(position: Vector2(0, 0), size: tailleCommune);
final Gobelin gobelin = Gobelin(position: Vector2(200, 0), size: tailleCommune);

// Plus tard, le joueur ramasse une potion de croissance :
joueur.size.setValues(96, 96);
```

**Le gobelin grandit aussi.** Les deux composants partagent **la même instance** de `Vector2`. Modifier l'un modifie l'autre, car ce sont le même objet en mémoire.

```text
  MÉMOIRE

     joueur.size ────────┐
                         ├──────►  Vector2 { x: 96, y: 96 }
     gobelin.size ───────┘
                          UNE SEULE instance, DEUX références
```

### La correction : `.clone()`

```dart
final Vector2 tailleCommune = Vector2(48, 48);

final Joueur joueur = Joueur(position: Vector2(0, 0), size: tailleCommune.clone());
final Gobelin gobelin = Gobelin(position: Vector2(200, 0), size: tailleCommune.clone());
```

```text
  MÉMOIRE

     joueur.size  ──────►  Vector2 { x: 96, y: 96 }
     gobelin.size ──────►  Vector2 { x: 48, y: 48 }
                            DEUX instances distinctes
```

### Les constructeurs créent bien un objet neuf à chaque appel

Une confusion fréquente : croire que `Vector2.zero()` renvoie une constante partagée. **Ce n'est pas le cas.** Chaque appel construit un objet neuf.

```dart
final Vector2 a = Vector2.zero();
final Vector2 b = Vector2.zero();
a.x = 100;
print(b.x);   // 0.0  -> ce sont bien deux objets différents
```

Le danger ne vient donc **jamais** de `Vector2.zero()` ou `Vector2.all(3)` appelés dans une expression. Il vient toujours d'une **variable** de type `Vector2` réutilisée à plusieurs endroits.

### Les trois constructeurs à connaître

| Écriture | Résultat | Usage courant |
| --- | --- | --- |
| `Vector2(x, y)` | `(x, y)` | position, vitesse |
| `Vector2.all(v)` | `(v, v)` | taille carrée, échelle uniforme |
| `Vector2.zero()` | `(0, 0)` | vitesse initiale, accumulateur |
| `Vector2.copy(a)` | copie de `a` | équivalent de `a.clone()` |

### Le cas le plus vicieux : `position` d'un composant

```dart
// NE FAITES PAS CELA.
final Vector2 depart = Vector2(50, 50);
world.add(Torche(position: depart));
world.add(Torche(position: depart));   // même instance !
```

Les deux torches partagent leur position. Déplacer la première déplace la seconde. Elles resteront superposées à jamais, et vous chercherez longtemps pourquoi.

### Et l'affectation directe ?

```dart
final Vector2 cible = Vector2(300, 120);

composant.position = cible;      // DANGER : partage de référence
composant.position.setFrom(cible); // SÛR : recopie les valeurs
composant.position = cible.clone(); // SÛR : nouvelle instance
```

Dans un `update`, préférez `setFrom` : aucune allocation, donc aucun travail pour le ramasse-miettes.

### Programme de démonstration

```dart
// Exécutable avec : dart run bin/partage.dart
import 'package:vector_math/vector_math_64.dart';

class Entite {
  Entite(this.nom, this.position, this.taille);
  final String nom;
  final Vector2 position;
  final Vector2 taille;

  @override
  String toString() => '$nom pos=$position taille=$taille';
}

void main() {
  print('--- SANS clone() : les deux entites partagent la taille ---');
  final Vector2 tailleCommune = Vector2(48, 48);
  final Entite a1 = Entite('heros  ', Vector2(0, 0), tailleCommune);
  final Entite b1 = Entite('gobelin', Vector2(200, 0), tailleCommune);

  a1.taille.setValues(96, 96);
  print(a1);
  print(b1);

  print('');
  print('--- AVEC clone() : chacune a la sienne ---');
  final Vector2 modele = Vector2(48, 48);
  final Entite a2 = Entite('heros  ', Vector2(0, 0), modele.clone());
  final Entite b2 = Entite('gobelin', Vector2(200, 0), modele.clone());

  a2.taille.setValues(96, 96);
  print(a2);
  print(b2);

  print('');
  print('--- Vector2.zero() cree bien un objet neuf a chaque appel ---');
  final Vector2 z1 = Vector2.zero();
  final Vector2 z2 = Vector2.zero();
  z1.x = 100;
  print('z1 = $z1');
  print('z2 = $z2');
  print('identiques ? ${identical(z1, z2)}');
}
```

**Résultat :**

```text
--- SANS clone() : les deux entites partagent la taille ---
heros   pos=[0.0,0.0] taille=[96.0,96.0]
gobelin pos=[200.0,0.0] taille=[96.0,96.0]

--- AVEC clone() : chacune a la sienne ---
heros   pos=[0.0,0.0] taille=[96.0,96.0]
gobelin pos=[200.0,0.0] taille=[48.0,48.0]

--- Vector2.zero() cree bien un objet neuf a chaque appel ---
z1 = [100.0,0.0]
z2 = [0.0,0.0]
identiques ? false
```

> **À retenir.** Une variable `Vector2` réutilisée est une bombe à retardement. La règle : **une variable `Vector2` ne sert qu'à un seul objet**, sinon `.clone()`.

---

## 27.20 — Premier carré qui bouge, en Flame

Nous y sommes. Voici le programme complet : un carré doré qui traverse l'écran de gauche à droite, rebondit sur les bords, et reste indépendant du framerate.

```dart
// lib/main.dart
//
// Donjon de Dart — chapitre 27, section 27.20.
// Un carré qui bouge, en Flame 1.38.0.

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: GameWidget<DonjonDeDart>(game: DonjonDeDart()),
      ),
    );
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Repère du monde = repère de l'écran, origine en haut à gauche.
    camera.viewfinder.anchor = Anchor.topLeft;

    await world.add(
      Heros(
        position: Vector2(40, 120),
        size: Vector2(48, 48),
      ),
    );
  }
}

/// Le héros : un carré doré qui va et vient horizontalement.
class Heros extends PositionComponent with HasGameReference<DonjonDeDart> {
  Heros({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFFE8B04B);

  /// Vitesse en PIXELS PAR SECONDE. Positive = vers la droite.
  Vector2 vitesse = Vector2(180, 0);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    anchor = Anchor.topLeft;
  }

  @override
  void update(double dt) {
    super.update(dt);

    // Déplacement indépendant du framerate (chapitre 20).
    position += vitesse * dt;

    // Rebond sur le bord droit.
    if (position.x + size.x >= game.size.x) {
      position.x = game.size.x - size.x;
      vitesse.x = -vitesse.x;
    }

    // Rebond sur le bord gauche.
    if (position.x <= 0) {
      position.x = 0;
      vitesse.x = -vitesse.x;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    // Coordonnées LOCALES : le canvas est déjà placé sur le composant.
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Résultat :**

```text
Un carré doré de 48x48 sur fond bleu nuit, à 120 pixels du haut.
Il traverse l'écran à 180 pixels par seconde, rebondit sur le bord
droit, revient, rebondit sur le bord gauche, indéfiniment.

Sur un écran 60 Hz comme sur un écran 120 Hz, il met exactement le
même temps à traverser.
```

### Lecture détaillée

**`with HasGameReference<DonjonDeDart>`** donne au composant une propriété `game`, typée `DonjonDeDart`. C'est ainsi qu'on accède à `game.size`. Attention : c'est ce mixin qu'il faut employer, et **non** l'ancien `HasGameRef` avec `gameRef`, qui est déprécié depuis plusieurs versions.

**`position += vitesse * dt`** fonctionne parce que `Vector2` définit `+` et `*`. Notez que `position += x` crée un nouveau vecteur et le réaffecte ; la variante sans allocation serait `position.add(vitesse * dt)`. À l'échelle d'un carré, la différence est invisible ; à l'échelle de mille projectiles, elle compte.

**`vitesse.x = -vitesse.x`** inverse la direction. Comme `Vector2` est mutable, on modifie le champ directement, sans reconstruire le vecteur.

**`game.size.x`** est la largeur logique du jeu, et elle est correcte même si l'utilisateur redimensionne la fenêtre : `onGameResize` met `size` à jour à chaque frame concernée.

**`size.toRect()`** produit `Rect.fromLTWH(0, 0, size.x, size.y)`. C'est l'idiome standard pour dessiner un `PositionComponent` en coordonnées locales.

### Variante : rebond dans les deux axes

Changez une seule ligne du constructeur et ajoutez le test vertical :

```dart
  Vector2 vitesse = Vector2(180, 130);

  @override
  void update(double dt) {
    super.update(dt);
    position += vitesse * dt;

    if (position.x <= 0) {
      position.x = 0;
      vitesse.x = -vitesse.x;
    } else if (position.x + size.x >= game.size.x) {
      position.x = game.size.x - size.x;
      vitesse.x = -vitesse.x;
    }

    if (position.y <= 0) {
      position.y = 0;
      vitesse.y = -vitesse.y;
    } else if (position.y + size.y >= game.size.y) {
      position.y = game.size.y - size.y;
      vitesse.y = -vitesse.y;
    }
  }
```

**Résultat :**

```text
Le carré rebondit maintenant sur les quatre bords, comme l'économiseur
d'écran d'un lecteur DVD.
```

> **À retenir.** Le mouvement en Flame, c'est `position += vitesse * dt` dans `update`, et un dessin en coordonnées locales dans `render`. Rien d'autre.

---

## 27.21 — Comparaison ligne à ligne avec la version Flutter pure du chapitre 20

Mettons les deux programmes côte à côte. À gauche, ce que vous aviez écrit. À droite, ce que Flame demande.

### Version chapitre 20 (Flutter pur)

```dart
// lib/main.dart — VERSION CHAPITRE 20, Flutter pur.
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const DonjonApp());

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(backgroundColor: Color(0xFF1B1B2A), body: VueDeJeu()),
    );
  }
}

class VueDeJeu extends StatefulWidget {
  const VueDeJeu({super.key});

  @override
  State<VueDeJeu> createState() => _VueDeJeuState();
}

class _VueDeJeuState extends State<VueDeJeu>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _x = 40;
  final double _y = 120;
  final double _taille = 48;
  double _vitesseX = 180;

  Size _tailleEcran = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surFrame)..start();
  }

  void _surFrame(Duration horodatage) {
    // 1. Calcul manuel du delta time.
    final double dt = _precedent == Duration.zero
        ? 0
        : (horodatage - _precedent).inMicroseconds / 1000000.0;
    _precedent = horodatage;

    // 2. Logique.
    _x += _vitesseX * dt;
    if (_x + _taille >= _tailleEcran.width) {
      _x = _tailleEcran.width - _taille;
      _vitesseX = -_vitesseX;
    }
    if (_x <= 0) {
      _x = 0;
      _vitesseX = -_vitesseX;
    }

    // 3. Demande de redessin manuelle.
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
      builder: (BuildContext context, BoxConstraints contraintes) {
        _tailleEcran = Size(contraintes.maxWidth, contraintes.maxHeight);
        return CustomPaint(
          size: Size.infinite,
          painter: _PeintreDeJeu(x: _x, y: _y, taille: _taille),
        );
      },
    );
  }
}

class _PeintreDeJeu extends CustomPainter {
  _PeintreDeJeu({required this.x, required this.y, required this.taille});

  final double x;
  final double y;
  final double taille;

  static final Paint _peinture = Paint()..color = const Color(0xFFE8B04B);

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(Rect.fromLTWH(x, y, taille, taille), _peinture);
  }

  @override
  bool shouldRepaint(_PeintreDeJeu ancien) =>
      ancien.x != x || ancien.y != y || ancien.taille != taille;
}
```

**Total : 96 lignes.**

### Version Flame

C'est le programme de la section 27.20. **Total : 74 lignes**, dont une bonne part de commentaires.

### Le tableau de correspondance

| Chapitre 20, à la main | Flame | Ce que vous n'écrivez plus |
| --- | --- | --- |
| `with SingleTickerProviderStateMixin` | intégré à `FlameGame` | la fourniture du ticker |
| `createTicker(_surFrame)..start()` | intégré | le démarrage de la boucle |
| `_ticker.dispose()` dans `dispose()` | intégré | l'arrêt propre |
| `Duration _precedent` + soustraction | `update(double dt)` | le calcul du delta time |
| `.inMicroseconds / 1000000.0` | `dt` déjà en secondes | la conversion d'unité |
| `setState(() {})` à chaque frame | intégré | la demande de redessin |
| `LayoutBuilder` pour connaître la taille | `game.size` | la mesure du parent |
| `CustomPainter` + `shouldRepaint` | `render(Canvas)` | la classe de peintre |
| `Rect.fromLTWH(x, y, l, h)` | `size.toRect()` | le placement manuel |
| champs `_x`, `_y` séparés | `position` (`Vector2`) | la gestion de deux `double` |
| `_vitesseX`, `_vitesseY` séparés | `vitesse` (`Vector2`) | idem |
| une classe `_PeintreDeJeu` par visuel | un `Component` par objet | la séparation artificielle |
| liste d'entités à gérer soi-même | `world.add(...)` | les files d'ajout et de retrait |

### Ce qui n'a pas changé du tout

Et c'est le point le plus important de cette section :

```text
  ┌─────────────────────────────────────────────────────────────┐
  │  IDENTIQUE entre le chapitre 20 et Flame                    │
  ├─────────────────────────────────────────────────────────────┤
  │  la structure  entrées -> update(dt) -> render(canvas)      │
  │  dt en secondes, dans un double                             │
  │  position += vitesse * dt                                   │
  │  le Canvas de Flutter et ses méthodes                       │
  │  Paint, Color, Rect, Offset                                 │
  │  le repère : x vers la droite, y vers le BAS                │
  │  la règle : render lit, update écrit                        │
  └─────────────────────────────────────────────────────────────┘
```

C'est précisément parce que vous avez écrit le chapitre 20 à la main que Flame vous paraît lisible aujourd'hui. Un élève qui commencerait directement par Flame verrait de la magie là où vous voyez du code que vous avez déjà écrit.

### Le vrai gain n'est pas le nombre de lignes

Comparez ce qu'il faut faire pour **ajouter un deuxième carré**.

En Flutter pur : ajouter `_x2`, `_y2`, `_vitesseX2`, dupliquer la logique dans `_surFrame`, dupliquer le dessin dans `paint`, ajouter les champs au `shouldRepaint`. Cinq endroits.

En Flame :

```dart
await world.add(Heros(position: Vector2(40, 220), size: Vector2(32, 32)));
```

Une ligne. C'est cela, le gain réel : Flame vous donne une **unité de composition**, le composant, que vous pouvez multiplier sans toucher au reste.

> **À retenir.** Flame ne change pas les concepts du jeu : il supprime la plomberie et vous donne une unité réutilisable, le composant.

---

## 27.22 — `backgroundColor()`

Par défaut, un `FlameGame` dessine un fond **noir**. La documentation de la méthode héritée est explicite : « Returns the game background color. By default it will return a black color. »

Pour changer cette couleur, on **surcharge une méthode**, on n'affecte pas une propriété :

```dart
class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);
}
```

Signature exacte :

```dart
Color backgroundColor()
```

### Rappel sur les couleurs (chapitre 21)

Le format est `0xAARRGGBB`, en hexadécimal :

```text
   0x FF 1B 1B 2A
      ▲  ▲  ▲  ▲
      │  │  │  └── bleu   : 0x2A = 42
      │  │  └───── vert   : 0x1B = 27
      │  └──────── rouge  : 0x1B = 27
      └─────────── alpha  : 0xFF = 255 (opaque)
```

### Un fond transparent

Si vous voulez qu'un décor Flutter apparaisse **derrière** le canvas du jeu, rendez le fond transparent et fournissez un `backgroundBuilder` au `GameWidget`.

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(
      game: DonjonDeDart(),
      // Un dégradé Flutter, dessiné DERRIÈRE le canvas du jeu.
      backgroundBuilder: (BuildContext context) => const DecoratedBox(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: <Color>[Color(0xFF2B1B4A), Color(0xFF0B0B12)],
          ),
        ),
      ),
    );
  }
}

class DonjonDeDart extends FlameGame {
  // Alpha = 0x00 : totalement transparent, le dégradé reste visible.
  @override
  Color backgroundColor() => const Color(0x00000000);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(
      CircleComponent(
        radius: 30,
        position: Vector2(160, 160),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );
  }
}
```

**Résultat :**

```text
Un dégradé violet vers noir, dessiné par Flutter, occupe tout l'écran.
Un disque doré de rayon 30 est centré en (160, 160), dessiné par Flame.
```

### Changer la couleur en cours de partie

`backgroundColor()` est une **méthode**, donc rien n'empêche de faire dépendre son résultat d'un champ.

```dart
class DonjonDeDart extends FlameGame {
  Color _fond = const Color(0xFF1B1B2A);

  @override
  Color backgroundColor() => _fond;

  void entrerDansLaSalleDuBoss() {
    _fond = const Color(0xFF3A0F14);   // rouge sombre
  }
}
```

> **À retenir.** `backgroundColor()` se **surcharge**. Un alpha à `0x00` laisse voir ce que Flutter dessine derrière.

---

## 27.23 — `GameWidget.controlled` et la gestion du cycle de vie

Il existe deux façons de fournir un jeu à un `GameWidget`, et la différence porte sur **qui possède l'instance**.

### Le constructeur ordinaire : c'est vous qui possédez le jeu

```dart
GameWidget<DonjonDeDart>(game: DonjonDeDart());
```

Vous créez l'instance, vous la passez. Danger : si ce `GameWidget` est construit dans la méthode `build` d'un widget, **une nouvelle instance de jeu est créée à chaque reconstruction du widget**. Or `build` peut être appelé très souvent.

```dart
// NE FAITES PAS CELA : un nouveau jeu à chaque build.
@override
Widget build(BuildContext context) {
  return GameWidget<DonjonDeDart>(game: DonjonDeDart());
}
```

La parade classique consiste à stocker l'instance dans un `StatefulWidget` :

```dart
class EcranDeJeu extends StatefulWidget {
  const EcranDeJeu({super.key});

  @override
  State<EcranDeJeu> createState() => _EcranDeJeuState();
}

class _EcranDeJeuState extends State<EcranDeJeu> {
  // Créé UNE FOIS, conservé pour toute la vie du State.
  late final DonjonDeDart _jeu = DonjonDeDart();

  @override
  Widget build(BuildContext context) => GameWidget<DonjonDeDart>(game: _jeu);
}
```

### `GameWidget.controlled` : c'est le widget qui possède le jeu

Signature exacte, vérifiée dans le dartdoc de la 1.38.0 :

```dart
const GameWidget.controlled({
  required GameFactory<T> gameFactory,
  TextDirection? textDirection,
  GameLoadingWidgetBuilder? loadingBuilder,
  GameErrorWidgetBuilder? errorBuilder,
  WidgetBuilder? backgroundBuilder,
  Map<String, OverlayWidgetBuilder<T>>? overlayBuilderMap,
  List<String>? initialActiveOverlays,
  FocusNode? focusNode,
  bool autofocus = true,
  MouseCursor? mouseCursor,
  bool addRepaintBoundary = true,
  HitTestBehavior behavior = HitTestBehavior.opaque,
  Key? key,
});
```

Vous ne passez plus une instance, mais une **fabrique** : une fonction qui sait créer le jeu. Le widget l'appelle **une seule fois**, au moment où son `State` est créé.

```dart
// Une seule instance de jeu, même si build() est rappelé cent fois.
GameWidget<DonjonDeDart>.controlled(
  gameFactory: DonjonDeDart.new,
);
```

`DonjonDeDart.new` est la référence au constructeur, notation vue au chapitre 7. On peut aussi écrire une lambda si le constructeur prend des arguments :

```dart
GameWidget<DonjonDeDart>.controlled(
  gameFactory: () => DonjonDeDart(difficulte: 2),
);
```

### Le tableau de décision

| Situation | Constructeur à employer |
| --- | --- |
| Le jeu est créé dans un `build()` | `GameWidget.controlled` |
| Le jeu est stocké dans un `State` | l'un ou l'autre |
| Vous devez appeler des méthodes du jeu depuis Flutter | `GameWidget(game: _jeu)`, en gardant la référence |
| Le jeu est passé à plusieurs widgets | `GameWidget(game: _jeu)` |
| Cas simple, un seul écran de jeu | `GameWidget.controlled` |

### Le cycle de vie complet

```text
   Le GameWidget est monté dans l'arbre Flutter
        │
        ▼
   création du jeu (constructeur, ou gameFactory)
        │
        ▼
   onGameResize(size)   -> le jeu apprend sa taille
        │
        ▼
   onLoad()             -> loadingBuilder affiché pendant ce temps
        │
        ▼
   onMount()
        │
        ▼
   boucle : update(dt) / render(canvas)  ... tant que le widget est vivant
        │
        ▼
   Le GameWidget est retiré de l'arbre
        │
        ▼
   onRemove()
```

Deux points annexes, utiles à connaître :

- **Mise en arrière-plan.** Un jeu Flame se met automatiquement en pause quand l'application passe en arrière-plan. Ce comportement se désactive avec `pauseWhenBackgrounded = false`.
- **Pause manuelle.** `pauseEngine()` et `resumeEngine()` arrêtent et relancent la boucle. La propriété `paused` donne et modifie l'état.

### Programme complet avec pause

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatefulWidget {
  const EcranDeJeu({super.key});

  @override
  State<EcranDeJeu> createState() => _EcranDeJeuState();
}

class _EcranDeJeuState extends State<EcranDeJeu> {
  late final DonjonDeDart _jeu = DonjonDeDart();

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        // Le GameWidget doit être contraint : ici par Expanded.
        Expanded(child: GameWidget<DonjonDeDart>(game: _jeu)),
        Padding(
          padding: const EdgeInsets.all(8),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              ElevatedButton(
                onPressed: () => setState(_jeu.pauseEngine),
                child: const Text('Pause'),
              ),
              const SizedBox(width: 12),
              ElevatedButton(
                onPressed: () => setState(_jeu.resumeEngine),
                child: const Text('Reprendre'),
              ),
              const SizedBox(width: 12),
              ElevatedButton(
                // stepEngine n'a d'effet que si le moteur est en pause.
                onPressed: () => _jeu.stepEngine(),
                child: const Text('Une frame'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Heros(position: Vector2(20, 100), size: Vector2(40, 40)));
  }
}

class Heros extends PositionComponent with HasGameReference<DonjonDeDart> {
  Heros({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFFE8B04B);
  Vector2 vitesse = Vector2(140, 0);

  @override
  void update(double dt) {
    super.update(dt);
    position += vitesse * dt;
    if (position.x <= 0 || position.x + size.x >= game.size.x) {
      vitesse.x = -vitesse.x;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Résultat :**

```text
Un carré doré va et vient. Le bouton « Pause » l'immobilise,
« Reprendre » le relance, « Une frame » le fait avancer d'un seul pas
quand il est en pause : idéal pour observer un bug au ralenti.
```

> **À retenir.** `GameWidget.controlled` garantit une seule instance de jeu. `pauseEngine`, `resumeEngine` et `stepEngine` sont vos outils de débogage temporel.

---

## 27.24 — Le mode debug (`debugMode`)

Flame dispose d'un mode d'affichage de diagnostic. La documentation précise ce qu'il fait : « each `PositionComponent` will be rendered with their bounding size, and have their positions written on the screen ».

Autrement dit : **la boîte englobante et la position** de chaque `PositionComponent` apparaissent à l'écran.

### Activer le mode debug

Sur tout le jeu :

```dart
class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    debugMode = true;   // s'applique au jeu ET à toute sa descendance
  }
}
```

Sur un seul composant :

```dart
class Gobelin extends PositionComponent {
  Gobelin() {
    debugMode = true;   // seulement ce composant
  }
}
```

`debugMode` est un `bool` avec getter et setter, hérité de `Component`.

### N'activer le debug qu'en développement

Dart fournit la constante `kDebugMode` dans `package:flutter/foundation.dart`. Elle vaut `true` en mode debug, `false` en `flutter build release`. C'est le réflexe professionnel :

```dart
import 'package:flutter/foundation.dart' show kDebugMode;

@override
Future<void> onLoad() async {
  await super.onLoad();
  debugMode = kDebugMode;   // jamais actif dans la version distribuée
}
```

### Le compteur d'images

Flame fournit deux composants dédiés :

- `FpsComponent` — mesure le nombre d'images par seconde ; s'ajoute n'importe où dans l'arbre ;
- `FpsTextComponent` — un `TextComponent` qui affiche la valeur mesurée par un `FpsComponent`.

La documentation précise que le chiffre affiché peut être légèrement inférieur à celui des DevTools Flutter, et que c'est **celui de Flame** qui fait foi, puisque c'est lui qui borne la boucle de jeu.

```dart
await camera.viewport.add(
  FpsTextComponent(position: Vector2(8, 8)),
);
```

### Programme complet de diagnostic

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/foundation.dart' show kDebugMode;
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Actif en développement seulement.
    debugMode = kDebugMode;

    await world.addAll(<Component>[
      Heros(position: Vector2(60, 60), size: Vector2(48, 48)),
      Heros(position: Vector2(180, 140), size: Vector2(32, 64)),
      Heros(position: Vector2(300, 90), size: Vector2(64, 32)),
    ]);

    // Compteur d'images, fixe à l'écran (dans le viewport).
    await camera.viewport.add(FpsTextComponent(position: Vector2(8, 8)));
  }
}

class Heros extends PositionComponent {
  Heros({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFFE8B04B);

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}
```

**Résultat :**

```text
Trois rectangles dorés, chacun entouré d'un cadre de debug portant
ses coordonnées. En haut à gauche, le nombre d'images par seconde.

En mode release (flutter build ... --release), les cadres disparaissent
automatiquement, puisque kDebugMode vaut false.
```

### À quoi le mode debug sert vraiment

| Symptôme | Ce que le mode debug révèle |
| --- | --- |
| Un composant est invisible | sa `size` vaut `Vector2.zero()` : aucun cadre n'apparaît |
| Un composant apparaît deux fois trop loin | vous avez ajouté `position` dans `render` |
| Un tap ne fonctionne pas | la boîte est nulle ou décalée |
| Un objet est hors écran | ses coordonnées affichées le confirment |
| Une hitbox ne correspond pas au sprite | les deux cadres se superposent mal (chapitre 32) |

> **À retenir.** `debugMode = kDebugMode;` dans `onLoad`. Vous voyez les boîtes en développement, le joueur ne les voit jamais.

---

## 27.25 — Redimensionnement de la fenêtre et `onGameResize`

Un jeu Flutter s'exécute dans une fenêtre redimensionnable sur bureau, dans un onglet redimensionnable sur Web, et sur un écran qui pivote sur mobile. La taille du jeu **change en cours d'exécution**.

### La méthode

```dart
void onGameResize(Vector2 size)
```

Elle est appelée :

1. **avant `onLoad`**, au tout premier calcul de mise en page ;
2. **à chaque changement de taille** ensuite.

C'est pour cela qu'elle figure en tête du cycle de vie de la section 27.13. Sa documentation précise qu'elle « passes the new size along to every component in the tree via their onGameResize method » : la propagation à toute la descendance est automatique.

### Le piège du `late final`

Comme `onGameResize` est appelée **avant** `onLoad`, les champs initialisés dans `onLoad` n'existent pas encore au premier appel. Y accéder lève une `LateInitializationError`.

```dart
class DonjonDeDart extends FlameGame {
  late final RectangleComponent _sol;

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);
    // _sol n'existe pas au premier appel : il faut se protéger.
    if (isLoaded) {
      _sol.size.x = size.x;
    }
  }
}
```

Le getter `isLoaded` est exactement fait pour cela. C'est le garde-fou standard.

### Trois stratégies de redimensionnement

| Stratégie | Comment | Quand |
| --- | --- | --- |
| Adapter le contenu | recalculer positions et tailles dans `onGameResize` | interfaces, HUD, jeux de plateau |
| Fixer la résolution | `CameraComponent.withFixedResolution(...)` | pixel art, jeux d'action (chapitre 31) |
| Ignorer | ne rien faire | prototypes |

La deuxième est la plus confortable pour un jeu d'action : vous raisonnez toujours dans la même grille, et Flame met à l'échelle.

### Programme complet

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(home: Scaffold(body: EcranDeJeu())));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<DonjonDeDart>(game: DonjonDeDart());
  }
}

class DonjonDeDart extends FlameGame {
  late final RectangleComponent _sol;
  late final RectangleComponent _plafond;
  late final CircleComponent _lune;
  late final TextComponent _infos;

  int _nombreDeResize = 0;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    _sol = RectangleComponent(
      size: Vector2(size.x, 40),
      position: Vector2(0, size.y - 40),
      paint: Paint()..color = const Color(0xFF3E3A52),
    );

    _plafond = RectangleComponent(
      size: Vector2(size.x, 24),
      position: Vector2.zero(),
      paint: Paint()..color = const Color(0xFF2A2740),
    );

    _lune = CircleComponent(
      radius: 26,
      anchor: Anchor.center,
      position: Vector2(size.x - 60, 70),
      paint: Paint()..color = const Color(0xFFE8B04B),
    );

    _infos = TextComponent(
      text: '',
      position: Vector2(12, 36),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFFFFFFF)),
      ),
    );

    await world.addAll(<Component>[_plafond, _sol, _lune]);
    await camera.viewport.add(_infos);

    _replacer(size);
  }

  @override
  void onGameResize(Vector2 size) {
    super.onGameResize(size);
    _nombreDeResize++;
    // Garde-fou : onGameResize est appelé AVANT onLoad.
    if (isLoaded) {
      _replacer(size);
    }
  }

  void _replacer(Vector2 taille) {
    _sol.size.x = taille.x;
    _sol.position.setValues(0, taille.y - 40);

    _plafond.size.x = taille.x;

    // La lune reste collée en haut à droite, à 60 px du bord.
    _lune.position.setValues(taille.x - 60, 70);

    _infos.text = 'taille : ${taille.x.toStringAsFixed(0)}'
        ' x ${taille.y.toStringAsFixed(0)}\n'
        'redimensionnements : $_nombreDeResize';
  }
}
```

**Résultat (fenêtre agrandie de 600x400 à 1000x700) :**

```text
taille : 1000 x 700
redimensionnements : 14

Le sol reste collé en bas et occupe toute la largeur.
Le plafond occupe toute la largeur.
La lune reste à 60 pixels du bord droit.
```

Le compteur monte vite : pendant un glissement de souris, Flutter appelle la mise en page à chaque frame. **Ne faites donc jamais de calcul coûteux dans `onGameResize`** : pas de chargement de fichier, pas de reconstruction de niveau.

> **À retenir.** `onGameResize` est appelée **avant** `onLoad` et à chaque changement de taille. Protégez-vous avec `isLoaded` et gardez la méthode légère.

---

## 27.26 — Lancer sur Web, Android et Windows

Un même code source, trois cibles. Voici les commandes et les points d'attention.

### Lister les appareils disponibles

```bash
flutter devices
```

**Résultat :**

```text
Windows (desktop) • windows • windows-x64 • Microsoft Windows
Chrome (web)      • chrome  • web-javascript • Google Chrome
sdk gphone64 x86_64 (mobile) • emulator-5554 • android-x64 • Android 15
```

### Web

```bash
flutter run -d chrome
flutter build web --release
```

Le résultat de la compilation se trouve dans `build/web/`. Il faut le servir par **HTTP** : ouvrir `index.html` directement depuis le disque ne fonctionne pas, à cause des règles de sécurité du navigateur.

```bash
cd build/web
python3 -m http.server 8080
```

| Point d'attention Web | Détail |
| --- | --- |
| Premier chargement | plusieurs mégaoctets ; prévoyez un écran d'attente |
| Son | la plupart des navigateurs exigent une **interaction utilisateur** avant de jouer un son (chapitre 34) |
| Clavier | ne fonctionne qu'après un clic dans le canvas, sauf si `autofocus` est actif |
| Performances | inférieures au natif ; visez des scènes simples |

### Android

```bash
flutter run -d emulator-5554
flutter build apk --release
flutter build appbundle --release
```

L'APK est produit dans `build/app/outputs/flutter-apk/app-release.apk`.

| Point d'attention Android | Détail |
| --- | --- |
| Identifiant | défini par `--org` à la création, modifiable dans `android/app/build.gradle` |
| Orientation | à verrouiller dans `main()` avec `SystemChrome.setPreferredOrientations` |
| Plein écran | `SystemChrome.setEnabledSystemUIMode(SystemUiMode.immersiveSticky)` |
| Taille de l'APK | `--split-per-abi` produit un APK par architecture |

Exemple de `main()` verrouillant l'orientation en paysage :

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

Future<void> main() async {
  // Obligatoire avant tout appel à une API de plateforme.
  WidgetsFlutterBinding.ensureInitialized();

  await SystemChrome.setPreferredOrientations(<DeviceOrientation>[
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight,
  ]);
  await SystemChrome.setEnabledSystemUIMode(SystemUiMode.immersiveSticky);

  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(body: GameWidget<DonjonDeDart>(game: DonjonDeDart())),
    ),
  );
}

class DonjonDeDart extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);
}
```

### Windows

```bash
flutter config --enable-windows-desktop
flutter run -d windows
flutter build windows --release
```

L'exécutable et ses DLL se trouvent dans `build/windows/x64/runner/Release/`. **Distribuez le dossier entier**, pas seulement le `.exe`.

| Point d'attention Windows | Détail |
| --- | --- |
| Outils requis | Visual Studio avec la charge « Développement Desktop en C++ » |
| Fenêtre | redimensionnable par défaut : `onGameResize` sera très sollicité |
| Clavier | pleinement disponible, contrairement au mobile |

### Tableau récapitulatif

| Cible | Lancer | Compiler | Sortie |
| --- | --- | --- | --- |
| Web | `flutter run -d chrome` | `flutter build web --release` | `build/web/` |
| Android | `flutter run -d <id>` | `flutter build apk --release` | `build/app/outputs/flutter-apk/` |
| Windows | `flutter run -d windows` | `flutter build windows --release` | `build/windows/x64/runner/Release/` |

### Le conseil qui fait gagner du temps

**Développez sur bureau, vérifiez régulièrement sur la cible réelle.** Le hot reload est le plus rapide sur bureau. Mais un jeu jamais essayé sur téléphone réserve toujours des surprises : taille des boutons, framerate, latence du son. Prenez l'habitude d'un essai sur appareil réel à la fin de chaque chapitre.

> **À retenir.** Un code, trois plateformes. Les différences ne portent pas sur Flame mais sur l'environnement : son, clavier, orientation, taille d'écran.

---

## 27.27 — L'arborescence recommandée du projet

Voici la structure que nous utiliserons jusqu'au chapitre 42. Créez-la maintenant : il est beaucoup plus coûteux de ranger un projet après coup.

```text
donjon_de_dart/
├── pubspec.yaml              # dépendances et déclaration des assets
├── pubspec.lock             # versions exactes — versionné dans Git
├── analysis_options.yaml    # règles de style
├── README.md
│
├── assets/
│   ├── images/              # PNG des sprites   -> Flame.images.load('x.png')
│   ├── audio/               # MP3, OGG, WAV     -> FlameAudio.play('x.mp3')
│   └── tiles/               # cartes Tiled .tmx -> TiledComponent.load('x.tmx')
│
├── lib/
│   ├── main.dart            # runApp + GameWidget UNIQUEMENT
│   │
│   ├── jeu/
│   │   ├── donjon_de_dart.dart   # la classe FlameGame
│   │   └── constantes.dart       # gravité, vitesses, couleurs, tailles
│   │
│   ├── composants/
│   │   ├── heros.dart
│   │   ├── gobelin.dart
│   │   ├── potion.dart
│   │   ├── cle.dart
│   │   ├── coffre.dart
│   │   └── boss.dart
│   │
│   ├── hud/
│   │   ├── barre_de_vie.dart
│   │   └── affichage_score.dart
│   │
│   ├── ecrans/                   # overlays Flutter (chapitre 35)
│   │   ├── menu_principal.dart
│   │   ├── menu_pause.dart
│   │   └── ecran_game_over.dart
│   │
│   ├── donnees/
│   │   ├── donnees_de_jeu.dart   # score, vies, record (chapitre 26)
│   │   └── machine_a_etats.dart  # GameState (chapitre 26)
│   │
│   └── outils/
│       ├── entrees.dart          # gestionnaire d'entrées (chapitre 26)
│       └── bus_evenements.dart   # bus d'événements (chapitre 26)
│
├── test/
│   ├── donnees_de_jeu_test.dart
│   └── machine_a_etats_test.dart
│
├── android/  ios/  web/  windows/    # généré par flutter create
└── build/                            # généré — jamais versionné
```

### Les quatre règles de rangement

**Règle 1 — `main.dart` reste minuscule.** Il contient `main()`, l'application Flutter et le `GameWidget`. Une trentaine de lignes, jamais plus. Toute la logique va ailleurs.

**Règle 2 — un fichier par classe de composant.** `heros.dart` contient `Heros`. Ce n'est pas une contrainte de Dart, c'est une convention qui rend le projet navigable.

**Règle 3 — les dossiers portent des noms de rôles, pas de types.** `composants/`, `hud/`, `donnees/` disent ce que font les fichiers. Un dossier `classes/` ou `fichiers/` n'apprend rien.

**Règle 4 — aucune valeur magique dans le code.** Toutes les constantes de réglage vont dans `constantes.dart`.

```dart
// lib/jeu/constantes.dart
import 'package:flutter/painting.dart' show Color;

/// Physique
const double kGravite = 980;          // pixels par seconde au carré
const double kVitesseHeros = 180;     // pixels par seconde
const double kForceDeSaut = 420;      // pixels par seconde

/// Règles
const int kViesInitiales = 3;
const int kPointsParPiece = 10;
const double kDureeInvincibilite = 1.2;  // secondes

/// Couleurs du donjon
const Color kFondDonjon = Color(0xFF1B1B2A);
const Color kCouleurHeros = Color(0xFFE8B04B);
const Color kCouleurGobelin = Color(0xFF6FA65A);
const Color kCouleurSol = Color(0xFF3E3A52);
```

Le jour où le héros vous paraît trop lent, vous changez **un chiffre, à un seul endroit**.

### Ce qui va dans Git, et ce qui n'y va pas

Le `.gitignore` généré par `flutter create` est correct. Retenez seulement :

| Chemin | Versionné ? | Pourquoi |
| --- | --- | --- |
| `lib/`, `assets/`, `test/` | oui | c'est votre travail |
| `pubspec.yaml` | oui | le contrat du projet |
| `pubspec.lock` | oui pour une application | garantit des versions identiques dans l'équipe |
| `build/` | non | régénéré à chaque compilation |
| `.dart_tool/` | non | cache local |
| `android/`, `web/`, `windows/` | oui | ils contiennent votre configuration |

> **À retenir.** Un projet bien rangé dès le premier jour est un projet qu'on peut encore modifier au chapitre 42.

---

## 27.28 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `super.onLoad()` oublié | la classe parente n'a pas pu s'initialiser : `world` et `camera` peuvent être incomplets | mettre `await super.onLoad();` en **première ligne** de tout `onLoad` |
| `super.update(dt)` oublié sur le jeu | les enfants ne reçoivent plus le tick : le jeu paraît figé alors que la boucle tourne | `super.update(dt);` en première ligne de `update` |
| `super.render(canvas)` oublié | les composants enfants ne sont pas dessinés | `super.render(canvas);` en première ligne de `render` |
| `await` manquant devant `world.add(...)` puis usage immédiat du composant | `add()` renvoie un `FutureOr<void>` : le montage est différé | `await world.add(c);` avant toute opération dépendant du montage |
| Un même `Vector2` réutilisé pour deux composants | `Vector2` est **mutable** : les deux références désignent le même objet | `.clone()`, ou construire un `Vector2` neuf pour chacun |
| `composant.position = autrePosition;` | partage de référence | `position.setFrom(autre)` ou `position = autre.clone()` |
| `class MonJeu extends BaseGame` | API d'avant Flame 1.0, tirée d'un tutoriel périmé | `extends FlameGame` |
| `camera.zoom = 2` ou `camera.followComponent(j)` | API de l'ancien système de caméra, supprimée | `camera.viewfinder.zoom = 2` et `camera.follow(j)` |
| `with HasGameRef<MonJeu>` puis `gameRef` | mixin déprécié, suppression annoncée | `with HasGameReference<MonJeu>` puis `game` |
| `GameWidget(game: g, shrinkWrap: true)` | paramètre supprimé en Flame 1.31.0 | encadrer le `GameWidget` d'un `SizedBox` ou d'un `Expanded` |
| `position.x += 3;` sans `dt` | vitesse dépendante du framerate : deux fois plus rapide en 120 Hz | `position.x += vitesse * dt;` |
| Lecture de `size` dans le constructeur du jeu | la taille n'est connue qu'après `onGameResize` | lire `size` dans `onLoad` ou plus tard |
| `LateInitializationError` dans `onGameResize` | `onGameResize` est appelée **avant** `onLoad` | protéger avec `if (isLoaded) { ... }` |
| Un composant reste invisible | `size` vaut `Vector2.zero()` par défaut | donner une `size` explicite |
| `canvas.drawRect(Rect.fromLTWH(position.x, position.y, ...))` dans un composant | le canvas est **déjà** translaté : la position est appliquée deux fois | `canvas.drawRect(size.toRect(), peinture);` |
| `game.add(monDecor)` au lieu de `world.add(monDecor)` | l'objet échappe à la caméra et ne défile pas | `world.add(...)` pour tout ce qui appartient au monde |
| `angle = 45` pour incliner un composant | `angle` est en **radians**, pas en degrés | `angle = 45 * math.pi / 180;` |
| `Vector2` passé là où `Offset` est attendu | `canvas.drawCircle` attend un `Offset` de `dart:ui` | `Offset(v.x, v.y)` |
| `version solving failed` après `flutter pub add flame_audio` | version de `flame` incompatible avec le pont | aligner les versions sur la table de la section 27.4 |
| `Unable to load asset: assets/images/x.png` | dossier non déclaré dans `pubspec.yaml`, ou chemin préfixé à la main | déclarer `assets/images/` et passer `'x.png'` |
| Le jeu se recrée sans arrêt et repart de zéro | `GameWidget(game: MonJeu())` construit dans un `build()` | `GameWidget.controlled(gameFactory: MonJeu.new)` |
| Le clavier ne répond pas sur Web | le canvas n'a pas le focus | cliquer dans le canvas, ou laisser `autofocus: true` |

---

## 27.29 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Flame | moteur de jeu **2D** en Dart, posé **sur** Flutter ; ni 3D, ni éditeur visuel |
| Version | `flame` **1.38.0**, exige Dart `>=3.12.0` et Flutter `>=3.44.0` |
| Écosystème | `flame`, plus les ponts `flame_audio`, `flame_tiled`, `flame_forge2d`, `flame_bloc` |
| Installation | `flutter create donjon_de_dart` puis `flutter pub add flame` |
| `pubspec.yaml` | `flame: ^1.38.0` et déclaration de `assets/images/`, `assets/audio/`, `assets/tiles/` |
| Tutoriels périmés | vérifier `BaseGame`, `gameRef`, `camera.zoom` : trois marqueurs d'obsolescence |
| `FlameGame` | classe de base d'un jeu ; possède `world` et `camera` |
| `GameWidget` | pont Flutter ↔ Flame ; prend toute la place disponible |
| `GameWidget.controlled` | prend une `gameFactory` : garantit **une seule** instance de jeu |
| `onLoad()` | `FutureOr<void>`, une seule fois, seul endroit où l'on peut `await` |
| `onMount()` | à chaque entrée dans l'arbre ; réinitialisation |
| `onRemove()` | à la sortie ; désabonnements et nettoyage |
| `update(double dt)` | `dt` en **secondes**, exactement comme au chapitre 20 |
| `render(Canvas canvas)` | le `Canvas` du chapitre 21, mais en coordonnées **locales** |
| `super.` | à appeler systématiquement dans `onLoad`, `update`, `render`, `onGameResize` |
| `size` | taille **logique** du jeu ; indisponible avant `onLoad` |
| `canvasSize` | taille **réelle** du canvas en pixels |
| `Vector2` | vecteur **mutable** de `vector_math` ; `Vector2(x, y)`, `.all()`, `.zero()` |
| Opérateurs | `+ - * /` créent ; `add`, `scale`, `setValues`, `setFrom` modifient |
| `.clone()` | indispensable dès qu'un `Vector2` risque d'être partagé |
| `length2` | comparer des distances sans calculer de racine carrée |
| `.normalized()` | direction pure de longueur 1 ; ne jamais l'appeler sur le vecteur nul |
| `backgroundColor()` | méthode à **surcharger** ; alpha `0x00` pour laisser voir Flutter derrière |
| `debugMode` | affiche boîtes et positions ; à piloter par `kDebugMode` |
| `pauseEngine` / `resumeEngine` / `stepEngine` | contrôle et débogage temporel de la boucle |
| `onGameResize` | appelée **avant** `onLoad` puis à chaque redimensionnement ; garder légère |
| `removeFromParent()` | retrait différé et sûr, même pendant un `update` |
| Plateformes | `flutter run -d chrome / windows / <android>` ; même code source |
| Arborescence | `main.dart` minuscule, `composants/`, `hud/`, `donnees/`, `constantes.dart` |

---

## 27.30 — Exercices

Tous les exercices se font dans le projet `donjon_de_dart`. Sauf mention contraire, ils ne demandent **aucun asset** : tout est dessiné en code.

### Exercice 1 — La salle vide (facile)

Créez un jeu `SalleVide` qui hérite de `FlameGame` et affiche un fond bleu nuit `0xFF1B1B2A`. Le `main.dart` doit contenir `runApp`, un `MaterialApp`, un `Scaffold` et un `GameWidget`. Rien d'autre ne doit apparaître à l'écran.

### Exercice 2 — Les trois habitants (facile)

Reprenez l'exercice 1 et ajoutez au **monde** trois `RectangleComponent` : un sol gris violet `0xFF3E3A52` de 40 pixels de haut collé au bas de l'écran, un héros doré `0xFFE8B04B` de 48 x 48 posé sur le sol à x = 60, et un gobelin vert `0xFF6FA65A` de 32 x 32 posé sur le sol à x = 260. L'origine du monde doit être en haut à gauche.

### Exercice 3 — Le calcul du guetteur (facile)

Écrivez un programme **console** (`dart run`) qui, à partir de `heros = Vector2(120, 340)` et `gobelin = Vector2(480, 180)`, affiche : le vecteur d'écart, la distance, la distance au carré, la direction normalisée, et la position du gobelin après une seconde de déplacement vers le héros à 90 pixels par seconde.

### Exercice 4 — Le sortilège de croissance (facile)

Le code suivant est fautif : quand le héros grandit, le gobelin grandit aussi.

```dart
final Vector2 taille = Vector2(48, 48);
final Heros heros = Heros(position: Vector2(60, 200), size: taille);
final Gobelin gobelin = Gobelin(position: Vector2(260, 200), size: taille);
heros.size.setValues(96, 96);
```

Expliquez la cause en une phrase, puis écrivez un programme console qui démontre le bug **et** sa correction.

### Exercice 5 — Le fantôme du donjon (moyen)

Écrivez un `PositionComponent` nommé `Fantome` qui se déplace en ligne droite et rebondit sur les **quatre** bords de l'écran. Sa vitesse initiale est `Vector2(200, 150)`. Le mouvement doit être indépendant du framerate, et le rebond doit rester correct si l'on redimensionne la fenêtre.

### Exercice 6 — Le gobelin traqueur (moyen)

Écrivez un composant `Gobelin` qui se dirige vers une cible fixe située au centre de l'écran, à 120 pixels par seconde, en utilisant `normalized()`. Il doit s'arrêter net quand il est à moins de 4 pixels de la cible, sans osciller. Protégez-vous du vecteur nul.

### Exercice 7 — La potion et son cycle de vie (moyen)

Écrivez un composant `Potion` qui affiche dans la console les trois messages `onLoad`, `onMount` et `onRemove`. Le jeu doit retirer la potion au bout de 3 secondes, puis la remettre 2 secondes plus tard. Vérifiez dans la console que `onLoad` n'apparaît **qu'une seule fois**.

### Exercice 8 — Le tableau de bord (moyen)

Ajoutez au jeu un HUD fixe, placé dans `camera.viewport`, qui affiche sur quatre lignes : le temps écoulé en secondes, le nombre de frames, `size`, et `canvasSize`. Activez le mode debug uniquement en développement, grâce à `kDebugMode`, et ajoutez un compteur d'images `FpsTextComponent`.

### Exercice 9 — Les quatre torches (difficile)

Placez quatre `CircleComponent` dorés de rayon 14, un dans chaque coin de l'écran, à 30 pixels des bords. Ils doivent **rester dans les coins** quand la fenêtre est redimensionnée. Affichez également le nombre de redimensionnements reçus depuis le démarrage. Attention au piège de l'ordre `onGameResize` / `onLoad`.

### Exercice 10 — La première salle du Donjon de Dart (difficile)

Assemblez tout le chapitre en un seul programme :

- `GameWidget.controlled` avec une `gameFactory` ;
- un `loadingBuilder` et un chargement volontairement retardé d'une seconde ;
- un sol, un héros qui rebondit horizontalement, trois gobelins de vitesses différentes ;
- un HUD dans le viewport affichant le temps écoulé et le nombre de gobelins encore présents ;
- chaque gobelin s'auto-supprime avec `removeFromParent()` après une durée de vie propre (4, 7 et 10 secondes) ;
- deux boutons Flutter sous le jeu : « Pause » et « Reprendre » ;
- toutes les constantes de réglage regroupées en tête de fichier.

---

## 27.31 — Corrections des exercices

### Correction 1

```dart
// lib/main.dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(body: EcranDeJeu()),
    );
  }
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<SalleVide>(game: SalleVide());
  }
}

class SalleVide extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);
}
```

**Résultat :**

```text
Un écran entièrement bleu nuit (0xFF1B1B2A). Aucun contenu.
La boucle de jeu tourne pourtant déjà à la cadence de l'écran.
```

**Explication :** trois éléments suffisent pour un jeu Flame. `runApp` démarre Flutter (chapitre 19). `GameWidget` est la frontière entre l'arbre de widgets et l'arbre de composants. `SalleVide extends FlameGame` fournit la boucle de jeu, le `world` et la `camera`, sans que nous ayons rien à écrire. `backgroundColor()` est une **méthode surchargée**, pas une propriété affectée : c'est la seule façon prévue par Flame de changer le noir par défaut. Notez `debugShowCheckedModeBanner: false`, qui retire le ruban « DEBUG » : il gêne la lecture d'un jeu.

---

### Correction 2

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

const Color kFond = Color(0xFF1B1B2A);
const Color kSol = Color(0xFF3E3A52);
const Color kHeros = Color(0xFFE8B04B);
const Color kGobelin = Color(0xFF6FA65A);
const double kHauteurSol = 40;

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return GameWidget<SalleVide>(game: SalleVide());
  }
}

class SalleVide extends FlameGame {
  @override
  Color backgroundColor() => kFond;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Origine du monde en haut à gauche : le repère du chapitre 21.
    camera.viewfinder.anchor = Anchor.topLeft;

    final double yDuSol = size.y - kHauteurSol;

    await world.addAll(<Component>[
      // Le sol, sur toute la largeur.
      RectangleComponent(
        position: Vector2(0, yDuSol),
        size: Vector2(size.x, kHauteurSol),
        paint: Paint()..color = kSol,
      ),
      // Le héros, posé SUR le sol : son bas touche yDuSol.
      RectangleComponent(
        position: Vector2(60, yDuSol - 48),
        size: Vector2(48, 48),
        paint: Paint()..color = kHeros,
      ),
      // Le gobelin, posé sur le sol lui aussi.
      RectangleComponent(
        position: Vector2(260, yDuSol - 32),
        size: Vector2(32, 32),
        paint: Paint()..color = kGobelin,
      ),
    ]);
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────┐
  │                                              │
  │                                              │
  │    ██                                        │  héros doré 48x48
  │    ██          ▓▓                            │  gobelin vert 32x32
  │  ██████████████████████████████████████████  │  sol 40 px
  └──────────────────────────────────────────────┘
```

**Explication :** trois points méritent attention. D'abord, `size` est lisible dans `onLoad` mais pas dans le constructeur : `onGameResize` a déjà été appelée à ce stade. Ensuite, `position` désigne l'**ancre** du composant, et l'ancre par défaut d'un `PositionComponent` est `Anchor.topLeft` : pour poser un objet de 48 pixels de haut sur le sol, on écrit donc `yDuSol - 48`. Enfin, `addAll` prend un `Iterable<Component>` et renvoie un vrai `Future<void>` : un seul `await` suffit pour les trois composants. Les couleurs sont sorties du code sous forme de constantes, conformément à la règle 4 de la section 27.27.

---

### Correction 3

```dart
// bin/guetteur.dart — exécutable avec : dart run bin/guetteur.dart
import 'package:vector_math/vector_math_64.dart';

void main() {
  final Vector2 heros = Vector2(120, 340);
  final Vector2 gobelin = Vector2(480, 180);

  print('heros   : $heros');
  print('gobelin : $gobelin');

  // Vecteur qui va DU gobelin VERS le héros.
  final Vector2 ecart = heros - gobelin;
  print('ecart              : $ecart');
  print('distance           : ${ecart.length.toStringAsFixed(3)}');
  print('distance au carre  : ${ecart.length2.toStringAsFixed(1)}');

  final Vector2 direction = ecart.normalized();
  print('direction (norme 1): $direction');
  print('verification norme : ${direction.length.toStringAsFixed(6)}');

  const double vitesse = 90; // pixels par seconde
  const double dt = 1.0;     // une seconde
  final Vector2 apres = gobelin + direction * vitesse * dt;
  print('gobelin apres 1 s  : $apres');
  print('distance restante  : '
      '${heros.distanceTo(apres).toStringAsFixed(3)}');
}
```

**Résultat :**

```text
heros   : [120.0,340.0]
gobelin : [480.0,180.0]
ecart              : [-360.0,160.0]
distance           : 393.955
distance au carre  : 155200.0
direction (norme 1): [-0.9137011148834558,0.4060893006148693]
verification norme : 1.000000
gobelin apres 1 s  : [397.7671003204889,216.5480370553382]
distance restante  : 303.955
```

**Explication :** l'ordre de la soustraction porte le sens. `heros - gobelin` produit le vecteur qui **part du gobelin et va vers le héros** : c'est celui qu'il faut pour faire avancer le gobelin. `normalized()` renvoie une **copie** de longueur 1 sans modifier `ecart` ; la ligne de vérification le confirme. La distance restante vaut exactement `393.955 - 90`, ce qui prouve que le gobelin s'est déplacé de 90 unités le long de la droite qui le sépare du héros : c'est la propriété fondamentale d'un vecteur normalisé multiplié par une vitesse. Enfin, `length2` évite une racine carrée ; on l'utilise dès qu'il s'agit seulement de **comparer** des distances.

---

### Correction 4

**Cause en une phrase :** `Vector2` est une classe **mutable**, donc les deux composants reçoivent une **référence vers le même objet** ; modifier la taille de l'un modifie mécaniquement celle de l'autre.

```dart
// bin/croissance.dart — exécutable avec : dart run bin/croissance.dart
import 'package:vector_math/vector_math_64.dart';

class Creature {
  Creature(this.nom, this.position, this.size);

  final String nom;
  final Vector2 position;
  final Vector2 size;

  @override
  String toString() =>
      '${nom.padRight(8)} position=$position size=$size';
}

void main() {
  print('=== VERSION FAUTIVE : une seule instance partagee ===');
  final Vector2 taillePartagee = Vector2(48, 48);
  final Creature heros1 = Creature('heros', Vector2(60, 200), taillePartagee);
  final Creature gobelin1 =
      Creature('gobelin', Vector2(260, 200), taillePartagee);

  heros1.size.setValues(96, 96); // sortilege de croissance sur le heros
  print(heros1);
  print(gobelin1);
  print('meme objet ? ${identical(heros1.size, gobelin1.size)}');

  print('');
  print('=== VERSION CORRIGEE : clone() pour chacun ===');
  final Vector2 modele = Vector2(48, 48);
  final Creature heros2 = Creature('heros', Vector2(60, 200), modele.clone());
  final Creature gobelin2 =
      Creature('gobelin', Vector2(260, 200), modele.clone());

  heros2.size.setValues(96, 96);
  print(heros2);
  print(gobelin2);
  print('meme objet ? ${identical(heros2.size, gobelin2.size)}');
  print('modele intact : $modele');
}
```

**Résultat :**

```text
=== VERSION FAUTIVE : une seule instance partagee ===
heros    position=[60.0,200.0] size=[96.0,96.0]
gobelin  position=[260.0,200.0] size=[96.0,96.0]
meme objet ? true

=== VERSION CORRIGEE : clone() pour chacun ===
heros    position=[60.0,200.0] size=[96.0,96.0]
gobelin  position=[260.0,200.0] size=[48.0,48.0]
meme objet ? false
modele intact : [48.0,48.0]
```

**Explication :** `identical()` répond à la question exacte « est-ce le même objet en mémoire ? », et sa réponse passe de `true` à `false` entre les deux versions : c'est la preuve du diagnostic. Remarquez que `final` n'a rien empêché : `final` interdit de réassigner la **variable**, pas de muter l'**objet**. Le réflexe à installer est simple : dès qu'une variable `Vector2` sert à plus d'un objet, on écrit `.clone()`. La dernière ligne montre un bénéfice annexe : le modèle reste intact et peut resservir indéfiniment.

---

### Correction 5

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) => GameWidget<Donjon>(game: Donjon());
}

class Donjon extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Fantome(position: Vector2(60, 60), size: Vector2(44, 44)));
  }
}

class Fantome extends PositionComponent with HasGameReference<Donjon> {
  Fantome({super.position, super.size});

  static final Paint _peinture = Paint()..color = const Color(0xFFBFC7E8);

  /// Vitesse en pixels par seconde, sur les deux axes.
  final Vector2 vitesse = Vector2(200, 150);

  @override
  void update(double dt) {
    super.update(dt);

    // 1. Déplacement indépendant du framerate.
    position += vitesse * dt;

    // 2. Rebond horizontal : on replace PUIS on inverse.
    if (position.x <= 0) {
      position.x = 0;
      vitesse.x = -vitesse.x;
    } else if (position.x + size.x >= game.size.x) {
      position.x = game.size.x - size.x;
      vitesse.x = -vitesse.x;
    }

    // 3. Rebond vertical.
    if (position.y <= 0) {
      position.y = 0;
      vitesse.y = -vitesse.y;
    } else if (position.y + size.y >= game.size.y) {
      position.y = game.size.y - size.y;
      vitesse.y = -vitesse.y;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRRect(
      RRect.fromRectAndRadius(size.toRect(), const Radius.circular(10)),
      _peinture,
    );
  }
}
```

**Résultat :**

```text
Un carré arrondi gris bleuté rebondit sur les quatre bords de la fenêtre.
Sur un écran 60 Hz comme sur un écran 120 Hz, il parcourt la même
distance dans le même temps.
Après un redimensionnement, il rebondit sur les NOUVEAUX bords.
```

**Explication :** trois décisions font la qualité de ce code. **Repositionner avant d'inverser** (`position.x = 0;` puis `vitesse.x = -vitesse.x;`) évite l'oscillation : sans cette remise en place, un fantôme sorti de deux pixels pourrait déclencher l'inversion à chaque frame et rester collé au bord en vibrant. **Lire `game.size` à chaque frame**, plutôt que de mémoriser la taille dans `onLoad`, rend le rebond automatiquement correct après un redimensionnement. Enfin, `vitesse` est déclaré `final` mais **muté** par `vitesse.x = -vitesse.x` : c'est exactement la mutabilité de `Vector2` mise à profit, et cela évite d'allouer un vecteur par rebond.

---

### Correction 6

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

const double kVitesseGobelin = 120; // pixels par seconde
const double kSeuilArret = 4;       // pixels

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) => GameWidget<Donjon>(game: Donjon());
}

class Donjon extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // La cible : le centre de l'écran.
    // size / 2 renvoie DÉJÀ un nouveau vecteur, mais on reste explicite.
    final Vector2 cible = Vector2(size.x / 2, size.y / 2);

    // Repère visuel de la cible.
    await world.add(
      CircleComponent(
        radius: 6,
        position: cible.clone(),
        anchor: Anchor.center,
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
    );

    await world.add(
      Gobelin(
        position: Vector2(40, 40),
        size: Vector2(32, 32),
        cible: cible.clone(),
      ),
    );
  }
}

class Gobelin extends PositionComponent {
  Gobelin({required this.cible, super.position, super.size});

  /// Point visé, en coordonnées du monde.
  final Vector2 cible;

  static final Paint _peinture = Paint()..color = const Color(0xFF6FA65A);

  /// Vecteur de travail réutilisé : évite une allocation par frame.
  final Vector2 _ecart = Vector2.zero();

  bool arrive = false;

  @override
  void update(double dt) {
    super.update(dt);
    if (arrive) return;

    // Centre du gobelin -> cible.
    _ecart
      ..setFrom(cible)
      ..sub(position + size / 2);

    // Protection : ne jamais normaliser un vecteur nul.
    final double distanceAuCarre = _ecart.length2;
    if (distanceAuCarre <= kSeuilArret * kSeuilArret) {
      arrive = true;
      return;
    }

    position += _ecart.normalized() * kVitesseGobelin * dt;
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
    if (arrive) {
      canvas.drawRect(
        size.toRect(),
        Paint()
          ..color = const Color(0xFFE8B04B)
          ..style = PaintingStyle.stroke
          ..strokeWidth = 3,
      );
    }
  }
}
```

**Résultat :**

```text
Un carré vert part du coin haut-gauche, file en ligne droite vers le
disque doré au centre, et s'immobilise net dès qu'il est à moins de
4 pixels. Il se cercle alors d'or. Il ne vibre pas et ne dépasse pas.
```

**Explication :** quatre points techniques. **La protection contre le vecteur nul** est indispensable : `normalized()` sur `(0, 0)` produit des `NaN` qui contaminent ensuite toute la position, et l'objet disparaît sans message d'erreur. **Le seuil d'arrêt** empêche l'oscillation : sans lui, le gobelin dépasserait la cible d'une fraction de pixel à chaque frame et repartirait en sens inverse indéfiniment. **La comparaison sur `length2`** évite une racine carrée par frame et par ennemi ; sur cent ennemis, la différence se mesure. Enfin, `_ecart` est un vecteur de travail créé une seule fois et rempli avec `setFrom` puis `sub` : aucune allocation par frame, donc aucun travail supplémentaire pour le ramasse-miettes.

---

### Correction 7

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) => GameWidget<Donjon>(game: Donjon());
}

class Donjon extends FlameGame {
  late final Potion _potion;
  double _chrono = 0;
  int _etape = 0;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    _potion = Potion(position: Vector2(120, 120));
    await world.add(_potion);
    debugPrint('--- t=0 s : la potion est dans le monde ---');
  }

  @override
  void update(double dt) {
    super.update(dt);
    _chrono += dt;

    if (_etape == 0 && _chrono >= 3) {
      _etape = 1;
      debugPrint('--- t=3 s : le heros ramasse la potion ---');
      _potion.removeFromParent();
    } else if (_etape == 1 && _chrono >= 5) {
      _etape = 2;
      debugPrint('--- t=5 s : la potion reapparait ---');
      world.add(_potion);
    }
  }
}

class Potion extends PositionComponent {
  Potion({super.position}) : super(size: Vector2(28, 40));

  static final Paint _verre = Paint()..color = const Color(0xFFBFC7E8);
  static final Paint _liquide = Paint()..color = const Color(0xFFC0392B);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    debugPrint('onLoad   : UNE SEULE FOIS — chargement des ressources');
  }

  @override
  void onMount() {
    super.onMount();
    debugPrint('onMount  : la potion entre dans l arbre');
  }

  @override
  void onRemove() {
    debugPrint('onRemove : la potion quitte l arbre');
    super.onRemove();
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    // Flacon.
    canvas.drawRect(size.toRect(), _verre);
    // Liquide : les deux tiers du bas.
    canvas.drawRect(
      Rect.fromLTWH(3, size.y * 0.35, size.x - 6, size.y * 0.65 - 3),
      _liquide,
    );
  }
}
```

**Résultat (console) :**

```text
onLoad   : UNE SEULE FOIS — chargement des ressources
onMount  : la potion entre dans l arbre
--- t=0 s : la potion est dans le monde ---
--- t=3 s : le heros ramasse la potion ---
onRemove : la potion quitte l arbre
--- t=5 s : la potion reapparait ---
onMount  : la potion entre dans l arbre
```

**Résultat (écran) :**

```text
0 s -> 3 s : un flacon rouge et gris en (120, 120).
3 s -> 5 s : rien.
5 s -> ... : le flacon est de retour, identique et au même endroit.
```

**Explication :** la console démontre la règle centrale de la section 27.13 : `onLoad` apparaît **une seule fois** alors que `onMount` apparaît **deux fois**. Flame considère qu'une ressource chargée le reste, même si le composant sort de l'arbre. C'est ce qui rend la réutilisation d'un composant peu coûteuse — un ennemi mis de côté puis remis en jeu ne recharge pas son sprite. La conséquence pratique est celle du tableau de la section 27.13 : ce qui doit être **remis à zéro** à chaque apparition, comme des points de vie ou une position de départ, va dans `onMount`, jamais dans `onLoad`. Notez enfin l'ordre des appels à `super` : `super.onMount()` en premier, `super.onRemove()` en dernier, afin que le message soit toujours affiché avant que le composant ne soit réellement détaché.

---

### Correction 8

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/foundation.dart' show kDebugMode;
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) => GameWidget<Donjon>(game: Donjon());
}

class Donjon extends FlameGame {
  late final TextComponent _tableauDeBord;

  double _temps = 0;
  int _frames = 0;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Boîtes de debug en développement uniquement.
    debugMode = kDebugMode;

    // Un décor pour que le mode debug ait quelque chose à encadrer.
    await world.addAll(<Component>[
      RectangleComponent(
        position: Vector2(80, 140),
        size: Vector2(48, 48),
        paint: Paint()..color = const Color(0xFFE8B04B),
      ),
      RectangleComponent(
        position: Vector2(220, 180),
        size: Vector2(32, 32),
        paint: Paint()..color = const Color(0xFF6FA65A),
      ),
    ]);

    // HUD : ajouté au VIEWPORT, donc fixe à l'écran.
    _tableauDeBord = TextComponent(
      text: '',
      position: Vector2(12, 40),
      anchor: Anchor.topLeft,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_tableauDeBord);

    // Compteur d'images fourni par Flame.
    await camera.viewport.add(FpsTextComponent(position: Vector2(12, 12)));
  }

  @override
  void update(double dt) {
    super.update(dt);
    _temps += dt;
    _frames++;

    _tableauDeBord.text = 'temps      : ${_temps.toStringAsFixed(2)} s\n'
        'frames     : $_frames\n'
        'size       : ${size.x.toStringAsFixed(0)}'
        ' x ${size.y.toStringAsFixed(0)}\n'
        'canvasSize : ${canvasSize.x.toStringAsFixed(0)}'
        ' x ${canvasSize.y.toStringAsFixed(0)}';
  }
}
```

**Résultat (fenêtre de 800 x 600, après 12 secondes) :**

```text
60.0
temps      : 12.03 s
frames     : 722
size       : 800 x 600
canvasSize : 800 x 600

Les deux rectangles sont entourés d'un cadre de debug portant leurs
coordonnées. En build release, les cadres disparaissent.
```

**Explication :** le HUD est ajouté à `camera.viewport` et non à `world`. C'est la différence qui compte : un composant du **monde** défile avec la caméra, un composant du **viewport** reste collé à l'écran. Ici, la caméra ne bouge pas encore, mais l'habitude doit être prise dès maintenant, sous peine de voir le score partir hors champ au chapitre 31. `debugMode = kDebugMode` est le réglage professionnel : les boîtes vous aident pendant le développement et n'atteignent jamais le joueur, sans qu'il faille penser à les désactiver avant de publier. Enfin, `size` et `canvasSize` sont ici identiques, ce qui est normal avec la caméra par défaut : la section 27.16 explique dans quel cas ils divergent.

---

### Correction 9

```dart
// lib/main.dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

const double kMarge = 30;
const double kRayonTorche = 14;

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) => GameWidget<Donjon>(game: Donjon());
}

class Donjon extends FlameGame {
  final List<CircleComponent> _torches = <CircleComponent>[];
  late final TextComponent _compteur;

  int _redimensionnements = 0;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Quatre torches, créées une fois pour toutes.
    for (int i = 0; i < 4; i++) {
      final CircleComponent torche = CircleComponent(
        radius: kRayonTorche,
        anchor: Anchor.center,
        position: Vector2.zero(),
        paint: Paint()..color = const Color(0xFFE8B04B),
      );
      _torches.add(torche);
      await world.add(torche);
    }

    _compteur = TextComponent(
      text: '',
      position: Vector2(12, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFFFFFFF)),
      ),
    );
    await camera.viewport.add(_compteur);

    _placerLesTorches(size);
  }

  @override
  void onGameResize(Vector2 size) {
    // super EN PREMIER : la nouvelle taille est propagée à tout l'arbre.
    super.onGameResize(size);
    _redimensionnements++;

    // GARDE-FOU : onGameResize est appelée AVANT onLoad.
    // Sans ce test, _torches et _compteur lèveraient LateInitializationError.
    if (isLoaded) {
      _placerLesTorches(size);
    }
  }

  void _placerLesTorches(Vector2 taille) {
    final double gauche = kMarge;
    final double droite = taille.x - kMarge;
    final double haut = kMarge;
    final double bas = taille.y - kMarge;

    _torches[0].position.setValues(gauche, haut);
    _torches[1].position.setValues(droite, haut);
    _torches[2].position.setValues(gauche, bas);
    _torches[3].position.setValues(droite, bas);

    _compteur.text = 'taille : ${taille.x.toStringAsFixed(0)}'
        ' x ${taille.y.toStringAsFixed(0)}\n'
        'redimensionnements : $_redimensionnements';
  }
}
```

**Résultat (fenêtre passée de 640 x 480 à 1100 x 720) :**

```text
taille : 1100 x 720
redimensionnements : 23

Quatre disques dorés, un dans chaque coin, à 30 pixels des bords.
Ils suivent la fenêtre pendant tout le glissement de la souris.
```

**Explication :** l'exercice porte sur un seul piège, mais c'est l'un des plus fréquents. `onGameResize` est appelée **avant** `onLoad` : au premier appel, les champs `late final` n'existent pas encore, et y accéder lève une `LateInitializationError` dont le message ne désigne pas la vraie cause. Le garde-fou `if (isLoaded)` règle le problème définitivement. Deux autres décisions comptent. Les torches sont créées **une seule fois** dans `onLoad` puis simplement **repositionnées** : recréer les composants à chaque redimensionnement produirait des dizaines de créations par seconde pendant un glissement. Et `position.setValues(x, y)` mute le vecteur existant au lieu d'en allouer un nouveau, ce qui suit la même logique d'économie. `Anchor.center` place la position sur le centre du disque, ce qui rend le calcul de marge direct.

---

### Correction 10

```dart
// lib/main.dart
//
// Donjon de Dart — chapitre 27, exercice 10.
// Première salle : GameWidget.controlled, chargement, HUD, pause.

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

// ---------------------------------------------------------------------------
// CONSTANTES DE RÉGLAGE (règle 4 de la section 27.27)
// ---------------------------------------------------------------------------
const Color kFond = Color(0xFF1B1B2A);
const Color kCouleurSol = Color(0xFF3E3A52);
const Color kCouleurHeros = Color(0xFFE8B04B);
const Color kCouleurGobelin = Color(0xFF6FA65A);
const Color kCouleurTexte = Color(0xFFFFFFFF);

const double kHauteurSol = 40;
const double kVitesseHeros = 180;      // pixels par seconde
const Duration kDureeChargement = Duration(seconds: 1);

// ---------------------------------------------------------------------------
// APPLICATION FLUTTER
// ---------------------------------------------------------------------------
void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: Scaffold(body: EcranDeJeu()),
  ));
}

class EcranDeJeu extends StatefulWidget {
  const EcranDeJeu({super.key});

  @override
  State<EcranDeJeu> createState() => _EcranDeJeuState();
}

class _EcranDeJeuState extends State<EcranDeJeu> {
  // On garde une référence pour piloter la pause depuis Flutter.
  late final Donjon _jeu = Donjon();

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        // Le GameWidget DOIT être contraint : Expanded s'en charge.
        Expanded(
          child: GameWidget<Donjon>.controlled(
            gameFactory: () => _jeu,
            loadingBuilder: (BuildContext context) => const ColoredBox(
              color: kFond,
              child: Center(
                child: Text(
                  'Ouverture de la premiere salle...',
                  style: TextStyle(color: kCouleurHeros, fontSize: 18),
                ),
              ),
            ),
          ),
        ),
        Padding(
          padding: const EdgeInsets.symmetric(vertical: 8),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              ElevatedButton(
                onPressed: _jeu.pauseEngine,
                child: const Text('Pause'),
              ),
              const SizedBox(width: 16),
              ElevatedButton(
                onPressed: _jeu.resumeEngine,
                child: const Text('Reprendre'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// ---------------------------------------------------------------------------
// LE JEU
// ---------------------------------------------------------------------------
class Donjon extends FlameGame {
  late final TextComponent _hud;
  double _temps = 0;

  @override
  Color backgroundColor() => kFond;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Chargement volontairement retardé : le loadingBuilder devient visible.
    await Future<void>.delayed(kDureeChargement);

    final double yDuSol = size.y - kHauteurSol;

    await world.add(
      RectangleComponent(
        position: Vector2(0, yDuSol),
        size: Vector2(size.x, kHauteurSol),
        paint: Paint()..color = kCouleurSol,
      ),
    );

    await world.add(
      Heros(
        position: Vector2(40, yDuSol - 48),
        size: Vector2(48, 48),
      ),
    );

    // Trois gobelins : vitesses et durées de vie différentes.
    const List<double> vitesses = <double>[70, 110, 150];
    const List<double> durees = <double>[4, 7, 10];
    for (int i = 0; i < 3; i++) {
      await world.add(
        Gobelin(
          position: Vector2(140.0 + i * 110, yDuSol - 32),
          size: Vector2(32, 32),
          vitesseX: vitesses[i],
          dureeDeVie: durees[i],
        ),
      );
    }

    // HUD fixe à l'écran.
    _hud = TextComponent(
      text: '',
      position: Vector2(12, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: kCouleurTexte),
      ),
    );
    await camera.viewport.add(_hud);
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (!isLoaded) return;

    _temps += dt;

    final int restants = world.children.whereType<Gobelin>().length;
    _hud.text = 'temps    : ${_temps.toStringAsFixed(1)} s\n'
        'gobelins : $restants';
  }
}

// ---------------------------------------------------------------------------
// COMPOSANTS
// ---------------------------------------------------------------------------
class Heros extends PositionComponent with HasGameReference<Donjon> {
  Heros({super.position, super.size});

  static final Paint _peinture = Paint()..color = kCouleurHeros;
  final Vector2 vitesse = Vector2(kVitesseHeros, 0);

  @override
  void update(double dt) {
    super.update(dt);
    position += vitesse * dt;

    if (position.x <= 0) {
      position.x = 0;
      vitesse.x = -vitesse.x;
    } else if (position.x + size.x >= game.size.x) {
      position.x = game.size.x - size.x;
      vitesse.x = -vitesse.x;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);
  }
}

class Gobelin extends PositionComponent with HasGameReference<Donjon> {
  Gobelin({
    required this.vitesseX,
    required this.dureeDeVie,
    super.position,
    super.size,
  });

  final double vitesseX;
  final double dureeDeVie;

  static final Paint _peinture = Paint()..color = kCouleurGobelin;

  double _age = 0;
  double _sens = 1;

  @override
  void update(double dt) {
    super.update(dt);

    _age += dt;
    if (_age >= dureeDeVie) {
      // Retrait DIFFÉRÉ et sûr, même au milieu d'un update.
      removeFromParent();
      return;
    }

    position.x += vitesseX * _sens * dt;
    if (position.x <= 0 || position.x + size.x >= game.size.x) {
      _sens = -_sens;
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    canvas.drawRect(size.toRect(), _peinture);

    // Jauge de vie restante, au-dessus du gobelin.
    final double reste = (1 - _age / dureeDeVie).clamp(0.0, 1.0);
    canvas.drawRect(
      Rect.fromLTWH(0, -8, size.x * reste, 4),
      Paint()..color = kCouleurHeros,
    );
  }
}
```

**Résultat :**

```text
0 s -> 1 s : « Ouverture de la premiere salle... » sur fond bleu nuit.

Puis :
  ┌──────────────────────────────────────────────┐
  │ temps    : 5.4 s                             │
  │ gobelins : 2                                 │
  │                                              │
  │    ██          ▔▔▔        ▔▔                 │  jauges de vie
  │    ██          ▓▓         ▓▓                 │  gobelins
  │  ██████████████████████████████████████████  │  sol
  └──────────────────────────────────────────────┘
       [ Pause ]   [ Reprendre ]

t = 4 s  : le premier gobelin disparaît, le HUD affiche « gobelins : 2 »
t = 7 s  : le deuxième disparaît
t = 10 s : le dernier disparaît, « gobelins : 0 »
```

**Explication :** ce programme rassemble sept notions du chapitre. `GameWidget.controlled` reçoit une `gameFactory` qui renvoie l'instance conservée dans le `State` : le jeu n'est donc créé qu'une fois, même si `build` est rappelé à chaque `setState`. Le `loadingBuilder` devient visible grâce au `Future.delayed` d'une seconde placé dans `onLoad` — c'est exactement le mécanisme de la section 27.12. Le `Expanded` contraint le `GameWidget`, sans quoi la `Column` lèverait une erreur de layout. Le HUD est ajouté au `camera.viewport`, donc il reste fixe. Le comptage `world.children.whereType<Gobelin>().length` réutilise `whereType` du chapitre 14 : c'est du Dart ordinaire appliqué à l'arbre de composants. `removeFromParent()` est appelé **au milieu d'un `update`**, ce qui aurait produit une `Concurrent modification during iteration` avec la liste écrite à la main au chapitre 26 ; Flame diffère le retrait, et le `return` qui suit évite simplement de continuer à déplacer un objet condamné. Enfin, la jauge de vie est dessinée en coordonnées **locales négatives** (`-8` en ordonnée), ce qui la place juste au-dessus du composant : une illustration directe du fait que le canvas est déjà translaté sur le composant.

---

## Et maintenant ?

Faisons le point sur ce que vous avez acquis dans ce chapitre.

```text
  ┌───────────────────────────────────────────────────────────────┐
  │                     ACQUIS DU CHAPITRE 27                     │
  └───────────────────────────────────────────────────────────────┘

  Installation     flutter create, flutter pub add flame,
                   pubspec.yaml, versions, tutoriels périmés

  Le jeu           FlameGame, world, camera, backgroundColor(),
                   size, canvasSize, debugMode, pauseEngine

  Le pont          GameWidget, GameWidget.controlled,
                   loadingBuilder, errorBuilder, contraintes de taille

  Le cycle de vie  onGameResize -> onLoad -> onMount ->
                   update / render -> onRemove

  Les vecteurs     Vector2, opérateurs, mutateurs, mesures,
                   normalized, length2, et le piège du clone

  Le mouvement     position += vitesse * dt, rebonds, poursuite

  Le déploiement   Web, Android, Windows ; arborescence du projet
```

Vous savez faire tourner un jeu Flame. Vous savez y mettre un carré, le faire bouger, l'afficher, le déboguer et le compiler pour trois plateformes.

Ce que vous ne savez pas encore, c'est **construire une scène**. Pour l'instant, chacun de vos composants est un objet isolé, ajouté à plat dans `world`. Or un jeu réel est un **arbre** : un héros porte une arme, l'arme porte une flamme, la flamme porte un halo. Quand le héros se déplace, tout le suit. Quand le héros disparaît, tout disparaît avec lui.

C'est l'objet du chapitre 28. Vous y découvrirez :

- la classe `Component` et sa spécialisation `PositionComponent` ;
- l'**arbre de composants**, les parents, les enfants, et ce que l'on hérite d'un parent ;
- `add`, `addAll`, `remove`, `removeFromParent`, `removeAll`, et la question du moment exact où l'action prend effet ;
- la propriété `priority`, c'est-à-dire l'ordre de dessin, celui que vous aviez implémenté à la main en 26.9 ;
- les **ancres** (`Anchor`) et le rôle exact de `anchor` dans la position, la rotation et la mise à l'échelle ;
- `angle`, `scale`, `nativeAngle`, et les repères locaux emboîtés ;
- les composants tout faits : `RectangleComponent`, `CircleComponent`, `PolygonComponent`, `TextComponent` ;
- comment retrouver un composant dans l'arbre, et comment le faire communiquer avec le jeu.

Deux conseils avant d'y aller.

**Gardez le projet `donjon_de_dart` intact.** Tous les chapitres suivants s'y ajoutent. Ne repartez pas d'un projet neuf à chaque fois : vous perdriez la continuité du fil rouge.

**Refaites l'exercice 10 sans regarder la correction.** Il condense l'essentiel du chapitre. Si vous le réécrivez de mémoire, le chapitre 28 sera une simple extension de ce que vous savez déjà.

Rendez-vous au chapitre suivant : [28-PARTIE-2B—COMPONENTS-ET-CYCLE-DE-VIE.md](./28-PARTIE-2B—COMPONENTS-ET-CYCLE-DE-VIE.md)
