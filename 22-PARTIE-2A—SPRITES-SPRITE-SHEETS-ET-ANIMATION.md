# PARTIE 2A — JEU 2D EN FLUTTER PUR
# CHAPITRE 22 — SPRITES, SPRITE SHEETS ET ANIMATION

> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 19 (Flutter en accéléré), chapitre 20 (boucle de jeu et delta time), chapitre 21 (coordonnées et dessin `Canvas`)
> **Ce que vous saurez faire à la fin :** charger une image, la découper en frames, et afficher un personnage animé dont la vitesse d'animation ne dépend pas du framerate — avec ou sans fichier image sur le disque.

---

## 22.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est un sprite et en quoi il diffère d'une forme dessinée à la main ;
- choisir entre pixel art et illustration, et en tirer les conséquences techniques ;
- expliquer pourquoi un sprite est un PNG et jamais un JPEG ;
- trouver des sprites libres de droits et lire une licence sans vous tromper ;
- déclarer un dossier `assets/` dans `pubspec.yaml` avec la bonne indentation ;
- charger une image avec `rootBundle.load` et `decodeImageFromList` ;
- expliquer pourquoi ce chargement est asynchrone et ce que cela impose ;
- précharger toutes les images avant de démarrer la boucle de jeu ;
- dessiner une image avec `canvas.drawImage()` ;
- dessiner une portion d'image avec `canvas.drawImageRect()` ;
- redimensionner un sprite sans le déformer ;
- garder du pixel art parfaitement net grâce à `FilterQuality.none` ;
- retourner un sprite horizontalement avec `scale(-1, 1)` ;
- lire une sprite sheet et calculer le `Rect` source de n'importe quelle frame ;
- convertir un index de frame en couple (ligne, colonne) ;
- écrire une classe `SpriteSheet` réutilisable ;
- écrire une classe d'animation qui avance avec le delta time ;
- gérer les animations non bouclées (mort, attaque) ;
- organiser plusieurs animations par personnage dans une machine à états ;
- changer d'animation sans provoquer de saut visuel ;
- reconnaître et corriger le texture bleeding ;
- lire un atlas et son fichier de description ;
- **générer vous-même vos frames en code**, sans télécharger le moindre fichier ;
- réaliser un personnage animé qui marche et s'arrête.

---

## Avertissement important : ce cours ne fournit aucune image

Cette formation est constituée de fichiers texte. Elle ne peut pas vous livrer de fichiers PNG.

Ce n'est pas un handicap, c'est une contrainte pédagogique utile. Vous allez donc apprendre **deux choses en parallèle** :

1. la manière normale de travailler, avec un vrai fichier image posé dans `assets/` ;
2. une **solution de repli 100 % code** : générer les images en mémoire avec `Canvas`, exactement comme au chapitre 21, puis les traiter comme n'importe quel sprite téléchargé.

```text
  ┌─────────────────────────┐        ┌────────────────────────────┐
  │  Voie 1 : vrai fichier  │        │  Voie 2 : image générée    │
  │  assets/heros.png       │        │  par du code (repli)       │
  └───────────┬─────────────┘        └──────────────┬─────────────┘
              │                                      │
              │  rootBundle.load                     │  PictureRecorder
              │  + décodage                          │  + picture.toImage
              ▼                                      ▼
        ┌───────────────────────────────────────────────────┐
        │              un objet  ui.Image                    │
        └───────────────────────┬───────────────────────────┘
                                │
                    canvas.drawImageRect(...)
```

Le point essentiel du schéma est le rectangle du bas : **à partir du moment où vous tenez un `ui.Image`, le reste du chapitre est identique**. Découpage, animation, retournement, machine à états : tout fonctionne à l'identique sur une image téléchargée ou sur une image fabriquée en code.

Tous les exemples de ce chapitre sont donc exécutables immédiatement, sans rien télécharger.

---

## 22.1 — Qu'est-ce qu'un sprite ?

Au chapitre 21, votre héros était un rectangle bleu et votre gobelin un cercle vert. C'était parfait pour comprendre le repère et le `Canvas`, mais personne ne vend un jeu peuplé de rectangles.

Un **sprite** est une petite image bitmap que l'on affiche dans une scène de jeu, à une position donnée.

Le mot vient des jeux d'arcade des années 1970. Le matériel de l'époque savait afficher un fond fixe, puis superposer par-dessus quelques petites images mobiles, sans redessiner tout l'écran. Ces images qui « flottaient » au-dessus du décor ont été appelées *sprites* (« lutins », « esprits »).

Aujourd'hui, il n'y a plus de circuit dédié : un sprite est simplement une image que vous dessinez sur le `Canvas`. Mais le vocabulaire est resté, et le concept aussi.

```text
  Une scène de jeu = un empilement de sprites

  ┌────────────────────────────────────────────┐
  │  fond (ciel, montagnes)          couche 0  │
  ├────────────────────────────────────────────┤
  │  décor (arbres, murs)            couche 1  │
  ├────────────────────────────────────────────┤
  │  objets (potion, clé, coffre)    couche 2  │
  ├────────────────────────────────────────────┤
  │  personnages (héros, gobelin)    couche 3  │
  ├────────────────────────────────────────────┤
  │  effets (étincelles, fumée)      couche 4  │
  ├────────────────────────────────────────────┤
  │  interface (vies, score)         couche 5  │
  └────────────────────────────────────────────┘
```

Ce qui distingue un sprite d'un simple `drawRect` :

| Forme dessinée (chapitre 21) | Sprite |
| --- | --- |
| décrite par du code | décrite par des pixels |
| modifiable en changeant un nombre | modifiable en rouvrant un éditeur d'image |
| légère (rien à charger) | occupe de la mémoire vidéo |
| limitée aux formes géométriques | n'importe quel dessin |
| toujours nette à toute taille | se dégrade si mal agrandie |

Un sprite possède quatre caractéristiques que vous devez avoir en tête en permanence :

1. **une taille en pixels** (par exemple 32 × 32) ;
2. **une transparence** : les pixels autour du personnage ne doivent pas être blancs, mais transparents ;
3. **un point d'ancrage** : le coin haut-gauche par défaut, mais souvent on préfère le centre ou les pieds ;
4. **un contenu figé** : le sprite ne « sait » rien, c'est votre code qui décide où et comment le dessiner.

> **Remarque :** en Flutter pur, un sprite est un objet de type `ui.Image` (classe de la bibliothèque `dart:ui`). Ce n'est **pas** le widget `Image` de Flutter. Nous verrons en 22.6 pourquoi cette distinction va vous éviter une erreur de compilation classique.

---

## 22.2 — Pixel art contre illustration

Avant d'écrire une ligne de code, il faut trancher une question de direction artistique, car elle a des conséquences techniques directes.

**Le pixel art** consiste à dessiner à très basse résolution — 16 × 16, 32 × 32, 64 × 64 pixels — puis à agrandir l'image d'un facteur entier au moment de l'affichage. Chaque pixel est posé volontairement.

```text
  Pixel art 8x8 (un cœur), agrandi x4 à l'écran

  . X X . . X X .        ██  ██
  X X X X X X X X      ████████
  X X X X X X X X      ████████
  X X X X X X X X      ████████
  . X X X X X X .        ██████
  . . X X X X . .          ████
  . . . X X . . .            ██
  . . . . . . . .
```

**L'illustration** (ou art « HD ») consiste à dessiner à haute résolution — 512 × 512 et au-delà — avec des dégradés, des contours lissés, des ombres douces.

Comparons honnêtement :

| Critère | Pixel art | Illustration |
| --- | --- | --- |
| Temps de production | faible | élevé |
| Compétence en dessin requise | modérée | élevée |
| Poids des fichiers | très faible | élevé |
| Mémoire vidéo | très faible | élevée |
| Nombre de frames par animation | 4 à 8 suffisent | 12 à 24 attendues |
| Filtre d'affichage | `FilterQuality.none` obligatoire | lissage souhaitable |
| Redimensionnement | facteurs entiers uniquement | libre |
| Tolérance aux erreurs | élevée | faible |

Pour un premier jeu, et pour toute cette formation, **le pixel art est le bon choix**. La raison n'est pas esthétique, elle est arithmétique : une animation de marche en pixel art demande 6 images de 32 × 32 pixels, soit 6 144 pixels à dessiner. La même animation en illustration demande 18 images de 512 × 512, soit 4 718 592 pixels. C'est 768 fois plus de travail de dessin, pour un joueur qui, manette en main, ne verra pas la différence sur un écran de téléphone.

Le fil rouge de cette formation, le **Donjon de Dart**, sera donc en pixel art : héros 32 × 32, gobelin 32 × 32, potion 16 × 16, clé 16 × 16.

> **Conséquence technique à retenir dès maintenant :** si vous choisissez le pixel art, vous devrez impérativement désactiver le lissage à l'affichage. C'est l'objet de la section 22.12, et c'est l'erreur numéro un des débutants.

---

## 22.3 — Formats d'image : PNG et transparence, et pourquoi pas JPEG

Un sprite se distribue en **PNG**. Toujours. Voici pourquoi, en trois arguments.

### Argument 1 : la transparence

Un personnage n'est pas rectangulaire. Autour de sa silhouette, il faut du **vide**, pas du blanc.

```text
  Ce que contient le fichier          Ce que voit le joueur
  (T = transparent)                   sur un fond de donjon

  T T T ▓ ▓ T T T                     ░░░░░░░░░░░░░░
  T T ▓ ▓ ▓ ▓ T T                     ░░░░░░▓▓░░░░░░
  T T T ▓ ▓ T T T                     ░░░░▓▓▓▓░░░░░░
  T T ▓ ▓ ▓ ▓ T T                     ░░░░░░▓▓░░░░░░
```

Le PNG stocke, pour chaque pixel, quatre valeurs : rouge, vert, bleu et **alpha**. L'alpha est le degré d'opacité, de 0 (invisible) à 255 (opaque). C'est ce canal alpha qui permet de poser un héros sur un décor sans rectangle blanc autour de lui.

Le JPEG ne possède **aucun canal alpha**. Un personnage exporté en JPEG arrive avec un fond opaque, en général blanc ou noir.

### Argument 2 : la compression sans perte

Le PNG utilise une compression *sans perte* : les pixels ressortent exactement tels qu'ils sont entrés.

Le JPEG utilise une compression *avec perte* : il réorganise l'image par blocs de 8 × 8 pixels et jette de l'information pour gagner de la place. Sur une photo de vacances, c'est invisible. Sur du pixel art, c'est un massacre.

```text
  Pixel art d'origine        Le même après un aller-retour JPEG

  ████░░░░                   ███▓░▒░░
  ████░░░░                   ███▓▒░░░     des pixels « sales »
  ░░░░████                   ░▒░▓███▒     apparaissent aux
  ░░░░████                   ░░▒▓███▓     frontières de couleur
```

Ces salissures s'appellent des *artefacts de compression*. Sur un sprite agrandi 4 fois, elles deviennent d'énormes taches.

### Argument 3 : les aplats de couleur

Le PNG compresse très efficacement les grandes zones de couleur uniforme, précisément ce dont le pixel art est fait. Un sprite 32 × 32 en PNG pèse souvent moins de 1 kilo-octet. Le même en JPEG pèse plus lourd **et** est abîmé.

### Tableau de décision

| Format | Transparence | Perte | Verdict pour un sprite |
| --- | --- | --- | --- |
| PNG | oui | non | **à utiliser** |
| JPEG / JPG | non | oui | à proscrire |
| GIF | 1 bit seulement | non | dépassé, palette limitée à 256 couleurs |
| WebP | oui | selon le mode | acceptable, mais moins universel |
| SVG | sans objet (vectoriel) | non | ce n'est pas un bitmap, autre sujet |

> **Règle du chapitre :** vos sprites sont des `.png`. Si un site vous propose un `.jpg` pour un personnage, c'est un mauvais signe sur la qualité du reste du lot.

---

## 22.4 — Où trouver des sprites libres de droits

Vous n'êtes probablement pas dessinateur. C'est normal, et ce n'est pas bloquant : il existe des bibliothèques entières d'images utilisables gratuitement, y compris dans un jeu commercial.

### Les trois sources de référence

| Site | Adresse | Ce qu'on y trouve | Licence dominante |
| --- | --- | --- | --- |
| Kenney | `kenney.nl` | packs cohérents, très propres, 2D et interface | CC0 (domaine public) |
| OpenGameArt | `opengameart.org` | énorme catalogue communautaire | variable, à lire au cas par cas |
| itch.io | `itch.io/game-assets` | packs d'artistes indépendants, gratuits ou payants | variable, indiquée par l'auteur |

**Kenney.nl** est la source à privilégier quand vous débutez. Les packs sont en CC0, c'est-à-dire sans aucune obligation : ni citation, ni contrepartie. Le style est homogène d'un pack à l'autre, ce qui évite un jeu au graphisme incohérent.

**OpenGameArt** est plus vaste mais plus hétérogène. Chaque contribution a sa propre licence. Il faut lire.

**itch.io** héberge des packs souvent superbes. Filtrez sur « Free » et lisez la page de l'auteur.

### Lire une licence sans se tromper

Une licence répond à quatre questions. Posez-les systématiquement.

```text
  ┌──────────────────────────────────────────────────────────┐
  │  1. Puis-je utiliser cet asset dans un jeu commercial ?  │
  │  2. Dois-je citer l'auteur ? Où, et comment ?            │
  │  3. Puis-je modifier l'image (couleurs, découpe) ?       │
  │  4. Suis-je obligé de partager mon jeu sous la même      │
  │     licence ?                                            │
  └──────────────────────────────────────────────────────────┘
```

Voici les licences que vous rencontrerez, avec les réponses correspondantes :

| Licence | Commercial | Citation | Modification | Contamination |
| --- | --- | --- | --- | --- |
| **CC0** | oui | non obligatoire | oui | non |
| **CC-BY** | oui | **obligatoire** | oui | non |
| **CC-BY-SA** | oui | **obligatoire** | oui | **oui** : votre travail dérivé doit être en CC-BY-SA |
| **CC-BY-NC** | **non** | obligatoire | oui | non |
| **GPL** | oui | oui | oui | oui, contamine le code |
| « Free for personal use » | **non** | — | — | — |

Trois pièges concrets :

1. **`NC` veut dire « non commercial ».** Si un jour vous mettez une publicité dans votre jeu, ou si vous le vendez un euro, vous êtes en infraction. Évitez `NC` dès le départ : vous ne savez pas ce que deviendra votre projet.
2. **`SA` (*ShareAlike*) est contaminant.** Utiliser un seul sprite en CC-BY-SA peut vous obliger à publier des éléments de votre jeu sous la même licence.
3. **« Gratuit » n'est pas « libre ».** Un pack téléchargeable sans payer peut interdire l'usage commercial.

### La bonne pratique : le fichier CREDITS

Dès le premier asset téléchargé, créez à la racine de votre projet un fichier `CREDITS.md` :

```text
# Crédits graphiques

## Personnages
- "Tiny Dungeon" par Kenney (kenney.nl) — licence CC0 — aucune obligation
- "Goblin walk cycle" par NomDeLAuteur (opengameart.org/content/xxxx)
  — licence CC-BY 3.0 — citation obligatoire, faite ici et dans l'écran Crédits du jeu

## Interface
- "UI Pack" par Kenney (kenney.nl) — licence CC0
```

Vous croyez que vous vous souviendrez de la provenance de vos images. Six mois et deux cents fichiers plus tard, vous ne vous en souviendrez pas. Ce fichier prend trente secondes à tenir à jour et vous évite un problème juridique réel.

> **Rappel du chapitre 16 :** ce fichier vit à côté de `pubspec.yaml`, avec le `README.md`. Il fait partie du projet, pas des à-côtés.

---

## 22.5 — Déclarer un dossier `assets/` dans `pubspec.yaml`

Flutter ne lit pas votre disque au hasard. Les fichiers embarqués dans l'application doivent être **déclarés** dans `pubspec.yaml`. Un fichier non déclaré n'existe pas pour l'application, même s'il est bien présent dans le dossier.

Vous connaissez déjà `pubspec.yaml` depuis le chapitre 16, où il servait à déclarer le nom du projet et ses dépendances. Il sert aussi à déclarer les assets.

### L'arborescence

```text
  donjon_de_dart/
  ├── pubspec.yaml
  ├── lib/
  │   └── main.dart
  └── assets/
      └── images/
          ├── heros.png
          ├── gobelin.png
          └── potion.png
```

### La déclaration

```yaml
name: donjon_de_dart
description: Le jeu du Donjon de Dart
publish_to: 'none'
version: 1.0.0

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

flutter:
  uses-material-design: true
  assets:
    - assets/images/heros.png
    - assets/images/gobelin.png
    - assets/images/potion.png
```

### Déclarer un dossier entier

Lister les fichiers un par un devient vite pénible. On peut déclarer un dossier, en terminant le chemin par une barre oblique :

```yaml
flutter:
  uses-material-design: true
  assets:
    - assets/images/
```

Tous les fichiers **directement** contenus dans `assets/images/` sont alors embarqués.

> **Piège :** la déclaration d'un dossier **n'est pas récursive**. `assets/images/` n'embarque pas `assets/images/ennemis/`. Il faut déclarer chaque sous-dossier :
>
> ```yaml
>   assets:
>     - assets/images/
>     - assets/images/ennemis/
>     - assets/images/decor/
> ```

### L'indentation YAML : la source d'erreur numéro un

Le YAML ne tolère ni tabulation, ni approximation. Voici les quatre fautes classiques.

```yaml
# FAUX — assets n'est pas sous flutter
flutter:
  uses-material-design: true
assets:
  - assets/images/
```

```yaml
# FAUX — tabulation au lieu d'espaces (invisible, mais fatal)
flutter:
	assets:
		- assets/images/
```

```yaml
# FAUX — il manque le tiret de liste
flutter:
  assets:
    assets/images/
```

```yaml
# FAUX — deux clés flutter: dans le même fichier
flutter:
  uses-material-design: true

flutter:
  assets:
    - assets/images/
```

```yaml
# CORRECT
flutter:
  uses-material-design: true
  assets:
    - assets/images/
```

Retenez trois règles :

1. **Deux espaces** par niveau, jamais de tabulation.
2. `assets:` est **indenté sous** `flutter:`, au même niveau que `uses-material-design:`.
3. Chaque chemin commence par un **tiret puis un espace**.

### Après modification : redémarrage complet

C'est le second piège de cette section, et il fait perdre des heures.

> Après avoir modifié la section `assets:` de `pubspec.yaml`, un *hot reload* ne suffit pas. Il faut **arrêter l'application et la relancer** (`flutter run`, ou le bouton Stop puis Run). La liste des assets est lue à la construction du bundle, pas à chaud.

Le message d'erreur typique quand la déclaration est absente ou mal indentée :

```text
Unable to load asset: "assets/images/heros.png".
The asset does not exist or has empty data.
```

Ce message signifie presque toujours l'une de ces trois choses : chemin mal orthographié, asset non déclaré, application non redémarrée.

---

## 22.6 — Charger une image : `rootBundle.load` + `decodeImageFromList`

Passons au code. Charger une image se fait en deux temps.

```text
  assets/images/heros.png
        │
        │  (1)  rootBundle.load(...)          lecture des octets bruts
        ▼
    ByteData  →  Uint8List      (le contenu du fichier PNG, compressé)
        │
        │  (2)  décodage                       décompression
        ▼
    ui.Image                    (une grille de pixels en mémoire)
```

**Étape 1 — lire les octets.** `rootBundle` est l'objet qui donne accès aux assets déclarés dans `pubspec.yaml`. Sa méthode `load` renvoie un `Future<ByteData>` : les octets du fichier, tels quels.

**Étape 2 — décoder.** Un fichier PNG est compressé. Il faut le transformer en une image utilisable par le moteur graphique, c'est-à-dire en `ui.Image`.

### La première distinction à faire : deux classes nommées `Image`

Flutter contient deux types nommés `Image` :

| Type | Bibliothèque | Rôle |
| --- | --- | --- |
| `Image` | `package:flutter/widgets.dart` | un **widget**, à mettre dans un arbre de widgets |
| `Image` | `dart:ui` | une **image bitmap**, à dessiner sur un `Canvas` |

Dans ce chapitre, nous voulons le second. Si vous importez les deux bibliothèques sans précaution, le compilateur proteste :

```text
Error: 'Image' is imported from both 'dart:ui' and 'package:flutter/src/widgets/image.dart'.
```

La solution standard est un import préfixé :

```dart
import 'dart:ui' as ui;
```

On écrit alors `ui.Image`, sans aucune ambiguïté possible. Prenez cette habitude tout de suite.

### La fonction de chargement

```dart
import 'dart:typed_data';
import 'dart:ui' as ui;

import 'package:flutter/services.dart' show rootBundle;

Future<ui.Image> chargerImage(String chemin) async {
  final ByteData donnees = await rootBundle.load(chemin);
  final Uint8List octets = donnees.buffer.asUint8List();
  final ui.Codec codec = await ui.instantiateImageCodec(octets);
  final ui.FrameInfo premiereFrame = await codec.getNextFrame();
  return premiereFrame.image;
}
```

Ligne par ligne :

- `rootBundle.load(chemin)` renvoie un `Future<ByteData>`. Le `await` attend les octets (chapitre 15).
- `donnees.buffer.asUint8List()` convertit le `ByteData` en liste d'octets non signés, format attendu par le décodeur.
- `ui.instantiateImageCodec(octets)` crée un décodeur. Il est générique : il gère aussi les GIF animés, d'où la notion de frames.
- `codec.getNextFrame()` décode la première image et renvoie un `ui.FrameInfo`.
- `premiereFrame.image` est le `ui.Image` recherché.

### La variante `decodeImageFromList`

`dart:ui` expose aussi une fonction plus courte, mais à **callback** et non à `Future` :

```dart
import 'dart:async';
import 'dart:typed_data';
import 'dart:ui' as ui;

import 'package:flutter/services.dart' show rootBundle;

Future<ui.Image> chargerImageAvecCallback(String chemin) async {
  final ByteData donnees = await rootBundle.load(chemin);
  final Uint8List octets = donnees.buffer.asUint8List();

  final Completer<ui.Image> completer = Completer<ui.Image>();
  ui.decodeImageFromList(octets, (ui.Image resultat) {
    completer.complete(resultat);
  });
  return completer.future;
}
```

Le `Completer` (chapitre 15) sert de pont entre une API à callback et une API à `Future`. Le résultat est strictement le même. Les deux formes sont valables ; la première est plus directe, la seconde est celle que vous croiserez dans beaucoup d'exemples en ligne.

> **Note :** si vous importez `package:flutter/painting.dart`, `decodeImageFromList` existe aussi sous une forme qui renvoie déjà un `Future<ui.Image>` mais prend directement des octets. Ne mélangez pas les trois : choisissez la version `instantiateImageCodec` et tenez-vous-y.

### Lire les dimensions

Une fois l'image chargée, ses dimensions sont disponibles :

```dart
print('largeur : ${image.width}');   // en pixels, un int
print('hauteur : ${image.height}');  // en pixels, un int
```

Ces valeurs sont des `int`. Le `Canvas`, lui, travaille en `double`. Vous ferez donc souvent `image.width.toDouble()`. C'est une source d'erreurs de type fréquente.

### Libérer la mémoire

Un `ui.Image` occupe de la mémoire vidéo : environ `largeur × hauteur × 4` octets. Une image de 2048 × 2048 pèse 16 mégaoctets en mémoire, quel que soit le poids du fichier PNG.

Quand une image n'est plus utile, appelez :

```dart
image.dispose();
```

Dans un jeu, on charge les images une fois au démarrage et on les garde jusqu'à la fin : le `dispose` intervient dans le `dispose()` de votre `State`.

---

## 22.7 — Le chargement est asynchrone (rappel du chapitre 15)

Voici l'erreur qui bloque le plus de débutants, et elle mérite sa propre section.

Lire un fichier prend du temps. Pas beaucoup — quelques millisecondes — mais du temps. Pendant ce temps, l'interface doit rester réactive. Flutter refuse donc catégoriquement de bloquer le thread principal pour attendre un fichier.

Conséquence : `rootBundle.load` renvoie un `Future`. Et un `Future`, comme vous l'avez appris au chapitre 15, est **une promesse de valeur, pas une valeur**.

```text
  t = 0 ms    demande de chargement lancée
              ┌──────────────────────────────────────┐
              │  le programme CONTINUE immédiatement  │
              │  l'image n'est PAS encore disponible  │
              └──────────────────────────────────────┘
  t = 8 ms    fichier lu et décodé
              l'image devient disponible
```

Le code fautif ressemble toujours à ceci :

```dart
// NE COMPILE PAS
class _MonJeuState extends State<MonJeu> {
  ui.Image image = chargerImage('assets/images/heros.png'); // Future<ui.Image> !

  void dessiner(Canvas canvas) {
    canvas.drawImage(image, Offset.zero, Paint()); // type incompatible
  }
}
```

Le compilateur répond :

```text
Error: A value of type 'Future<Image>' can't be assigned to a variable of type 'Image'.
```

Le message est parfaitement clair une fois qu'on sait le lire : vous tenez la promesse, pas le colis.

### La bonne structure

Le champ doit être **nullable** (chapitre 12) et rempli plus tard :

```dart
class _MonJeuState extends State<MonJeu> {
  ui.Image? _heros; // null tant que le chargement n'est pas fini

  @override
  void initState() {
    super.initState();
    _chargerTout();
  }

  Future<void> _chargerTout() async {
    final ui.Image image = await chargerImage('assets/images/heros.png');
    if (!mounted) return;
    setState(() {
      _heros = image;
    });
  }
}
```

Quatre points méritent une explication :

1. **`initState` ne peut pas être `async`.** Sa signature est `void initState()`. On y appelle donc une méthode `Future<void>` séparée, sans l'attendre.
2. **Le champ est `ui.Image?`.** Tant qu'il vaut `null`, il ne faut pas dessiner.
3. **`if (!mounted) return;`** protège contre le cas où l'utilisateur quitte l'écran avant la fin du chargement. Appeler `setState` sur un `State` démonté provoque une exception (chapitre 13).
4. **`setState`** signale à Flutter que l'affichage doit être reconstruit maintenant que l'image existe.

### Le test avant de dessiner

Dans le `CustomPainter`, l'image nullable impose un test :

```dart
@override
void paint(Canvas canvas, Size size) {
  final ui.Image? img = image;
  if (img == null) {
    // Chargement en cours : on affiche autre chose.
    return;
  }
  canvas.drawImage(img, Offset.zero, Paint());
}
```

Notez la copie dans une variable locale `img`. C'est exactement la technique du chapitre 12, section 12.15 : la promotion de type ne fonctionne pas sur un champ, mais fonctionne sur une variable locale `final`.

---

## 22.8 — Précharger avant de démarrer le jeu

Tester la nullité de chaque image à chaque frame est possible, mais c'est une mauvaise architecture. Imaginez vingt sprites : vingt tests, vingt fois par seconde, et un jeu qui commence à s'afficher par morceaux, un sprite après l'autre.

La bonne pratique est le **préchargement** : on charge **tout**, on attend que **tout** soit prêt, et seulement ensuite on démarre la boucle de jeu.

```text
  ┌──────────────┐   ┌─────────────────┐   ┌──────────────────┐
  │ ÉCRAN DE     │──>│ toutes les       │──>│ BOUCLE DE JEU     │
  │ CHARGEMENT   │   │ images sont      │   │ aucun test de     │
  │ (spinner)    │   │ décodées         │   │ nullité nécessaire│
  └──────────────┘   └─────────────────┘   └──────────────────┘
```

### Un gestionnaire d'assets

```dart
class BanqueImages {
  final Map<String, ui.Image> _images = <String, ui.Image>{};

  Future<void> charger(Map<String, String> aCharger) async {
    for (final MapEntry<String, String> entree in aCharger.entries) {
      _images[entree.key] = await chargerImage(entree.value);
    }
  }

  ui.Image operator [](String cle) {
    final ui.Image? image = _images[cle];
    if (image == null) {
      throw StateError('Image "$cle" non chargée. Appelez charger() avant.');
    }
    return image;
  }

  void liberer() {
    for (final ui.Image image in _images.values) {
      image.dispose();
    }
    _images.clear();
  }
}
```

Cette classe reprend plusieurs acquis de la PARTIE 1A : une `Map` (chapitre 6), une boucle `for-in` sur `entries` (chapitre 6), `async`/`await` (chapitre 15), une exception explicite (chapitre 13) et la surcharge de l'opérateur `[]` (chapitre 10).

L'usage devient limpide :

```dart
final BanqueImages banque = BanqueImages();

await banque.charger(<String, String>{
  'heros': 'assets/images/heros.png',
  'gobelin': 'assets/images/gobelin.png',
  'potion': 'assets/images/potion.png',
});

// À partir d'ici, plus aucun test de nullité :
canvas.drawImage(banque['heros'], Offset(100, 100), Paint());
```

### Charger en parallèle

La boucle ci-dessus charge les images **l'une après l'autre**. Avec vingt images, on additionne vingt attentes. `Future.wait` (chapitre 15) permet de tout lancer en même temps :

```dart
Future<void> chargerEnParallele(Map<String, String> aCharger) async {
  final List<String> cles = aCharger.keys.toList();
  final List<ui.Image> images = await Future.wait(
    cles.map((String cle) => chargerImage(aCharger[cle]!)),
  );
  for (int i = 0; i < cles.length; i++) {
    _images[cles[i]] = images[i];
  }
}
```

### L'écran de chargement

Le patron complet, côté widget :

```dart
class EcranJeu extends StatefulWidget {
  const EcranJeu({super.key});

  @override
  State<EcranJeu> createState() => _EcranJeuState();
}

class _EcranJeuState extends State<EcranJeu> {
  bool _pret = false;
  final BanqueImages _banque = BanqueImages();

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    await _banque.charger(<String, String>{
      'heros': 'assets/images/heros.png',
    });
    if (!mounted) return;
    setState(() => _pret = true);
  }

  @override
  void dispose() {
    _banque.liberer();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!_pret) {
      return const ColoredBox(
        color: Color(0xFF101018),
        child: Center(child: CircularProgressIndicator()),
      );
    }
    return Jeu(banque: _banque);
  }
}
```

**Un seul test de nullité, dans `build`.** Toute la logique de jeu, en dessous, travaille avec des images garanties non nulles. C'est la traduction directe du principe du chapitre 12 : « pousser le `null` vers les bords ».

---

## 22.8 bis — La solution de repli : fabriquer un `ui.Image` en code

Vous n'avez pas de fichier PNG. Il vous faut malgré tout un `ui.Image`. Voici la clé de voûte de tout le chapitre.

`dart:ui` fournit `PictureRecorder`, un enregistreur de commandes de dessin. On dessine dedans avec un `Canvas` ordinaire (chapitre 21), puis on convertit le résultat en image bitmap.

```text
  PictureRecorder            Canvas               Picture           ui.Image
  ┌──────────────┐      ┌──────────────┐    ┌───────────┐     ┌───────────┐
  │ on enregistre│ ───> │ drawRect     │──> │ endRecord │ ──> │  toImage  │
  │ les ordres   │      │ drawCircle   │    │  ing()    │     │  (w, h)   │
  └──────────────┘      └──────────────┘    └───────────┘     └───────────┘
```

La fonction générique :

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}
```

Le paramètre `dessin` est une fonction passée en argument : exactement le concept de fonction d'ordre supérieur du chapitre 7.

Exemple d'usage : fabriquer une potion 16 × 16 sans aucun fichier.

```dart
Future<ui.Image> creerPotion() {
  return imageDepuisDessin(16, 16, (Canvas canvas) {
    final Paint verre = Paint()..color = const Color(0xFF7FD1FF);
    final Paint liquide = Paint()..color = const Color(0xFFE0245E);
    final Paint bouchon = Paint()..color = const Color(0xFF8B5A2B);

    canvas.drawRect(const Rect.fromLTWH(6, 1, 4, 3), bouchon);
    canvas.drawRect(const Rect.fromLTWH(4, 4, 8, 11), verre);
    canvas.drawRect(const Rect.fromLTWH(5, 8, 6, 6), liquide);
  });
}
```

Le résultat est un `ui.Image` de 16 × 16 pixels, avec un **fond transparent** partout où l'on n'a rien dessiné : exactement le comportement d'un PNG bien exporté.

> **Point capital :** ce `ui.Image` est indiscernable d'un PNG chargé depuis le disque. `drawImage`, `drawImageRect`, le découpage en frames, le retournement : tout s'applique. Vous pouvez donc suivre l'intégralité du chapitre sans un seul téléchargement, puis remplacer `creerPotion()` par `chargerImage('assets/images/potion.png')` le jour où vous aurez vos vraies images. **Une seule ligne à changer.**

Voici un premier programme complet et exécutable. Il fabrique une image en code et l'affiche.

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> creerPotion() {
  return imageDepuisDessin(16, 16, (Canvas canvas) {
    final Paint verre = Paint()..color = const Color(0xFF7FD1FF);
    final Paint liquide = Paint()..color = const Color(0xFFE0245E);
    final Paint bouchon = Paint()..color = const Color(0xFF8B5A2B);

    canvas.drawRect(const Rect.fromLTWH(6, 1, 4, 3), bouchon);
    canvas.drawRect(const Rect.fromLTWH(4, 4, 8, 11), verre);
    canvas.drawRect(const Rect.fromLTWH(5, 8, 6, 6), liquide);
  });
}

void main() {
  runApp(const AppPotion());
}

class AppPotion extends StatelessWidget {
  const AppPotion({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: EcranPotion(),
      ),
    );
  }
}

class EcranPotion extends StatefulWidget {
  const EcranPotion({super.key});

  @override
  State<EcranPotion> createState() => _EcranPotionState();
}

class _EcranPotionState extends State<EcranPotion> {
  ui.Image? _potion;

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    final ui.Image image = await creerPotion();
    if (!mounted) return;
    setState(() => _potion = image);
  }

  @override
  void dispose() {
    _potion?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? potion = _potion;
    if (potion == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(
      painter: PeintrePotion(potion),
      size: Size.infinite,
    );
  }
}

class PeintrePotion extends CustomPainter {
  PeintrePotion(this.potion);

  final ui.Image potion;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint p = Paint()..filterQuality = FilterQuality.none;

    // Taille réelle : 16 x 16 pixels, en haut à gauche.
    canvas.drawImage(potion, const Offset(20, 20), p);

    // Agrandie 8 fois, au centre.
    canvas.drawImageRect(
      potion,
      const Rect.fromLTWH(0, 0, 16, 16),
      Rect.fromLTWH(size.width / 2 - 64, size.height / 2 - 64, 128, 128),
      p,
    );
  }

  @override
  bool shouldRepaint(PeintrePotion oldDelegate) => oldDelegate.potion != potion;
}
```

**Résultat :**

```text
Une fenêtre sombre.
En haut à gauche : une minuscule fiole de 16 pixels de côté.
Au centre : la même fiole, agrandie 8 fois (128 x 128), aux bords parfaitement
nets grâce à FilterQuality.none.
```

Gardez ce squelette sous la main : tous les exemples du chapitre en sont des variantes.

---

## 22.9 — `canvas.drawImage()`

La méthode la plus simple pour afficher un sprite :

```dart
canvas.drawImage(ui.Image image, Offset position, Paint paint);
```

Trois arguments, aucune surprise :

| Argument | Type | Rôle |
| --- | --- | --- |
| `image` | `ui.Image` | l'image à dessiner |
| `position` | `Offset` | où placer le **coin haut-gauche** de l'image |
| `paint` | `Paint` | options de rendu (filtre, opacité, mode de fusion) |

```text
        position = Offset(100, 60)
              │
              ▼
        (100,60) ┌────────────┐
                 │            │  l'image occupe
                 │   sprite   │  image.width x image.height
                 │            │  pixels
                 └────────────┘
```

Point crucial : **`drawImage` dessine l'image à sa taille native**, pixel pour pixel. Un sprite de 32 × 32 occupera 32 × 32 pixels logiques à l'écran, c'est-à-dire un timbre-poste sur un écran moderne. Pour l'agrandir, il faudra soit `drawImageRect` (section 22.10), soit une transformation `scale` (chapitre 21).

### L'ancrage au coin haut-gauche

`drawImage` ancre au coin haut-gauche. Or, en logique de jeu, on raisonne presque toujours par le centre (pour les rotations, les distances) ou par les pieds (pour la gravité et le sol).

Le passage se fait par une soustraction :

```dart
// Centrer le sprite sur (x, y)
final double l = image.width.toDouble();
final double h = image.height.toDouble();
canvas.drawImage(image, Offset(x - l / 2, y - h / 2), paint);

// Poser le sprite sur le sol, pieds en (x, y)
canvas.drawImage(image, Offset(x - l / 2, y - h), paint);
```

```text
   ancrage haut-gauche      ancrage centre          ancrage pieds
        (x,y)                                        
          ┌─────┐             ┌─────┐                  ┌─────┐
          │     │             │  •  │ (x,y)            │     │
          │     │             │     │                  │     │
          └─────┘             └─────┘                  └──•──┘ (x,y)
```

### Le `Paint` n'est pas optionnel

Contrairement à ce que l'on pourrait croire, le troisième argument est **obligatoire**. Un `Paint()` neuf convient dans le cas général :

```dart
canvas.drawImage(image, Offset.zero, Paint());
```

Mais ce `Paint` sert à beaucoup de choses utiles :

```dart
// Sprite à moitié transparent (personnage invincible qui clignote)
final Paint fantome = Paint()
  ..color = const Color(0xFFFFFFFF).withValues(alpha: 0.5);
canvas.drawImage(image, position, fantome);

// Sprite teinté en rouge (personnage qui vient de prendre un coup)
final Paint blesse = Paint()
  ..colorFilter = const ColorFilter.mode(Color(0xAAFF0000), BlendMode.srcATop);
canvas.drawImage(image, position, blesse);

// Pixel art : pas de lissage
final Paint net = Paint()..filterQuality = FilterQuality.none;
canvas.drawImage(image, position, net);
```

> **Optimisation :** créez vos objets `Paint` **une seule fois**, en champ de votre painter ou de votre classe de jeu. Un `Paint()` construit à chaque frame, pour chaque sprite, produit des milliers d'objets par seconde et fait travailler le ramasse-miettes pour rien.

### Exemple complet

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  return enregistreur.endRecording().toImage(largeur, hauteur);
}

/// Un héros 32x32 dessiné en code : tunique, tête, ceinture.
Future<ui.Image> creerHeros() {
  return imageDepuisDessin(32, 32, (Canvas canvas) {
    final Paint peau = Paint()..color = const Color(0xFFF2C79B);
    final Paint tunique = Paint()..color = const Color(0xFF3A7BD5);
    final Paint ceinture = Paint()..color = const Color(0xFF5A3A1A);
    final Paint bottes = Paint()..color = const Color(0xFF2A2A38);

    canvas.drawRect(const Rect.fromLTWH(11, 3, 10, 9), peau);      // tête
    canvas.drawRect(const Rect.fromLTWH(10, 12, 12, 11), tunique); // torse
    canvas.drawRect(const Rect.fromLTWH(10, 18, 12, 2), ceinture);
    canvas.drawRect(const Rect.fromLTWH(11, 23, 4, 7), bottes);    // jambe G
    canvas.drawRect(const Rect.fromLTWH(17, 23, 4, 7), bottes);    // jambe D
  });
}

void main() => runApp(const AppDrawImage());

class AppDrawImage extends StatelessWidget {
  const AppDrawImage({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF141420),
        body: EcranDrawImage(),
      ),
    );
  }
}

class EcranDrawImage extends StatefulWidget {
  const EcranDrawImage({super.key});

  @override
  State<EcranDrawImage> createState() => _EcranDrawImageState();
}

class _EcranDrawImageState extends State<EcranDrawImage> {
  ui.Image? _heros;

  @override
  void initState() {
    super.initState();
    creerHeros().then((ui.Image img) {
      if (!mounted) return;
      setState(() => _heros = img);
    });
  }

  @override
  void dispose() {
    _heros?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? heros = _heros;
    if (heros == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(painter: PeintreAncrages(heros), size: Size.infinite);
  }
}

class PeintreAncrages extends CustomPainter {
  PeintreAncrages(this.heros);

  final ui.Image heros;
  final Paint _p = Paint()..filterQuality = FilterQuality.none;
  final Paint _repere = Paint()..color = const Color(0xFFFF3B3B);

  @override
  void paint(Canvas canvas, Size size) {
    final double l = heros.width.toDouble();
    final double h = heros.height.toDouble();

    const double y = 120;

    // 1. Ancrage haut-gauche
    canvas.drawImage(heros, const Offset(60, y), _p);
    canvas.drawCircle(const Offset(60, y), 2, _repere);

    // 2. Ancrage centre
    canvas.drawImage(heros, Offset(160 - l / 2, y - h / 2), _p);
    canvas.drawCircle(const Offset(160, y), 2, _repere);

    // 3. Ancrage pieds
    canvas.drawImage(heros, Offset(260 - l / 2, y - h), _p);
    canvas.drawCircle(const Offset(260, y), 2, _repere);
  }

  @override
  bool shouldRepaint(PeintreAncrages oldDelegate) => false;
}
```

**Résultat :**

```text
Trois petits héros de 32x32 alignés horizontalement.
Un point rouge marque, pour chacun, le point (x, y) demandé :
- héros 1 : le point est à son coin haut-gauche ;
- héros 2 : le point est en plein milieu de son corps ;
- héros 3 : le point est exactement sous ses pieds.
```

---

## 22.10 — `canvas.drawImageRect()` : source et destination

`drawImage` dessine **toute** l'image, à sa **taille native**. C'est insuffisant dès qu'on veut découper ou redimensionner.

`drawImageRect` répond aux deux besoins d'un coup :

```dart
canvas.drawImageRect(
  ui.Image image,
  Rect source,       // QUOI prendre dans l'image
  Rect destination,  // OÙ le poser à l'écran, et à quelle taille
  Paint paint,
);
```

C'est **la méthode la plus importante de tout le chapitre**. Tout le reste en découle.

```text
   IMAGE SOURCE (en pixels de l'image)      ÉCRAN (en pixels logiques)

   0        32       64       96            
   ┌────────┬────────┬────────┐             ┌──────────────────┐
   │ frame0 │ frame1 │ frame2 │             │                  │
   │        │ ▓▓▓▓▓▓ │        │  32         │    ┌──────────┐  │
   │        │ ▓▓▓▓▓▓ │        │             │    │  ▓▓▓▓▓▓  │  │
   └────────┴────────┴────────┘             │    │  ▓▓▓▓▓▓  │  │
             ▲                              │    └──────────┘  │
             │                              │         ▲        │
      source = Rect.fromLTWH(32,0,32,32)    └─────────┼────────┘
                                                      │
                        destination = Rect.fromLTWH(200,150,96,96)
```

Deux idées à séparer nettement :

- le rectangle **source** est exprimé en **pixels de l'image** ; son origine est le coin haut-gauche de l'image ;
- le rectangle **destination** est exprimé en **coordonnées du canvas** ; c'est le repère du chapitre 21.

Les deux rectangles n'ont aucune obligation d'avoir la même taille. C'est justement là que se joue le redimensionnement.

| Situation | Effet |
| --- | --- |
| destination plus grande que source | agrandissement |
| destination plus petite que source | réduction |
| destination de mêmes proportions | pas de déformation |
| destination de proportions différentes | **sprite écrasé ou étiré** |

### Les constructeurs de `Rect` utiles

```dart
Rect.fromLTWH(left, top, width, height)   // le plus utilisé pour les frames
Rect.fromLTRB(left, top, right, bottom)   // par les quatre bords
Rect.fromCenter(center: Offset, width: , height: )
Rect.fromPoints(Offset a, Offset b)
```

Pour découper une sprite sheet, `Rect.fromLTWH` est presque toujours le bon choix : il décrit exactement « à partir de tel pixel, prendre telle largeur ».

### Exemple : découper une bande de trois cases

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  return enregistreur.endRecording().toImage(largeur, hauteur);
}

/// Une bande de 3 cases de 32x32 : potion, clé, pièce.
Future<ui.Image> creerBandeObjets() {
  return imageDepuisDessin(96, 32, (Canvas canvas) {
    final Paint fiole = Paint()..color = const Color(0xFFE0245E);
    final Paint verre = Paint()..color = const Color(0xFF9FE0FF);
    final Paint metal = Paint()..color = const Color(0xFFFFD447);
    final Paint sombre = Paint()..color = const Color(0xFF8A6A00);

    // Case 0 (x de 0 à 31) : potion
    canvas.drawRect(const Rect.fromLTWH(12, 6, 8, 4), sombre);
    canvas.drawRect(const Rect.fromLTWH(10, 10, 12, 16), verre);
    canvas.drawRect(const Rect.fromLTWH(12, 16, 8, 8), fiole);

    // Case 1 (x de 32 à 63) : clé
    canvas.drawCircle(const Offset(42, 12), 6, metal);
    canvas.drawCircle(const Offset(42, 12), 3, Paint()..blendMode = BlendMode.clear);
    canvas.drawRect(const Rect.fromLTWH(41, 16, 3, 12), metal);
    canvas.drawRect(const Rect.fromLTWH(44, 22, 4, 3), metal);

    // Case 2 (x de 64 à 95) : pièce
    canvas.drawCircle(const Offset(80, 16), 10, metal);
    canvas.drawCircle(const Offset(80, 16), 6, sombre);
  });
}

void main() => runApp(const AppDecoupe());

class AppDecoupe extends StatelessWidget {
  const AppDecoupe({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF141420),
        body: EcranDecoupe(),
      ),
    );
  }
}

class EcranDecoupe extends StatefulWidget {
  const EcranDecoupe({super.key});

  @override
  State<EcranDecoupe> createState() => _EcranDecoupeState();
}

class _EcranDecoupeState extends State<EcranDecoupe> {
  ui.Image? _bande;

  @override
  void initState() {
    super.initState();
    creerBandeObjets().then((ui.Image img) {
      if (!mounted) return;
      setState(() => _bande = img);
    });
  }

  @override
  void dispose() {
    _bande?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? bande = _bande;
    if (bande == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(painter: PeintreDecoupe(bande), size: Size.infinite);
  }
}

class PeintreDecoupe extends CustomPainter {
  PeintreDecoupe(this.bande);

  final ui.Image bande;
  final Paint _p = Paint()..filterQuality = FilterQuality.none;
  final Paint _cadre = Paint()
    ..color = const Color(0xFF55FF88)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 1;

  @override
  void paint(Canvas canvas, Size size) {
    // La planche entière, en haut, agrandie 2 fois.
    const Rect toutSrc = Rect.fromLTWH(0, 0, 96, 32);
    const Rect toutDst = Rect.fromLTWH(20, 20, 192, 64);
    canvas.drawImageRect(bande, toutSrc, toutDst, _p);
    canvas.drawRect(toutDst, _cadre);

    // Chaque case isolée, en dessous, agrandie 3 fois.
    for (int i = 0; i < 3; i++) {
      final Rect src = Rect.fromLTWH(i * 32.0, 0, 32, 32);
      final Rect dst = Rect.fromLTWH(20 + i * 110.0, 130, 96, 96);
      canvas.drawImageRect(bande, src, dst, _p);
      canvas.drawRect(dst, _cadre);
    }
  }

  @override
  bool shouldRepaint(PeintreDecoupe oldDelegate) => false;
}
```

**Résultat :**

```text
En haut : la planche complète (3 objets côte à côte), entourée d'un cadre vert.
En dessous : les 3 objets séparés, chacun dans son propre cadre vert, agrandis 3x.
La potion, la clé et la pièce proviennent tous du MÊME ui.Image :
seul le Rect source change.
```

C'est exactement le mécanisme d'une sprite sheet. Vous venez de le voir fonctionner avant même qu'on en donne le nom.

---

## 22.11 — Redimensionner un sprite

Un sprite 32 × 32 affiché à sa taille native est illisible sur un écran de téléphone moderne. On l'agrandit donc, souvent 2, 3 ou 4 fois.

Il y a deux façons de le faire.

### Méthode 1 : agrandir le `Rect` destination

```dart
const double echelle = 4;
final Rect src = Rect.fromLTWH(0, 0, 32, 32);
final Rect dst = Rect.fromLTWH(x, y, 32 * echelle, 32 * echelle);
canvas.drawImageRect(image, src, dst, paint);
```

C'est la méthode recommandée : explicite, locale, sans effet de bord.

### Méthode 2 : transformer le canvas (chapitre 21)

```dart
canvas.save();
canvas.translate(x, y);
canvas.scale(4);
canvas.drawImage(image, Offset.zero, paint);
canvas.restore();
```

Cette méthode est utile quand on veut agrandir **tout un groupe** de dessins d'un coup (le monde entier, par exemple). Le `save`/`restore` est obligatoire, sinon l'échelle contamine tout ce qui est dessiné ensuite.

### La règle d'or du pixel art : facteurs entiers

Un agrandissement d'un facteur non entier produit des lignes de pixels de largeurs inégales.

```text
  Facteur 3 (entier) :          Facteur 2,5 (non entier) :

  ███ ███ ███ ███               ███ ██ ███ ██ ███
  ███ ███ ███ ███               ██  ██ ███ ██ ██
  ███ ███ ███ ███               ███ ██ ███ ██ ███
  chaque pixel source =         certains pixels font 3 de large,
  exactement 3x3                d'autres 2 : le sprite « bave »
```

Utilisez donc 1, 2, 3, 4, 6, 8. Jamais 2,5 ni 3,7.

### Calculer une échelle entière adaptée à l'écran

```dart
/// Plus grand facteur entier tel que le monde tienne dans l'écran.
int echelleEntiere(Size ecran, double largeurMonde, double hauteurMonde) {
  final double parLargeur = ecran.width / largeurMonde;
  final double parHauteur = ecran.height / hauteurMonde;
  final int facteur = (parLargeur < parHauteur ? parLargeur : parHauteur).floor();
  return facteur < 1 ? 1 : facteur;
}
```

`floor()` arrondit vers le bas (chapitre 3). Le garde-fou `facteur < 1 ? 1 : facteur` évite une échelle nulle sur un écran minuscule.

### Ne jamais déformer un sprite

Le respect des proportions se contrôle avec une seule multiplication :

```dart
// CORRECT : le même facteur en largeur et en hauteur
final Rect dst = Rect.fromLTWH(x, y, 32 * e, 32 * e);

// FAUX : le personnage est étiré en largeur
final Rect dst = Rect.fromLTWH(x, y, 32 * 4, 32 * 2);
```

Un personnage écrasé est immédiatement perceptible par le joueur, même s'il ne sait pas nommer le problème.

> **Astuce de vérification :** pendant le développement, dessinez le contour du `Rect` destination avec un `Paint` en `PaintingStyle.stroke`. Si le cadre n'est pas carré alors que votre sprite l'est, vous avez une déformation.

---

## 22.12 — `FilterQuality.none` : garder le pixel art net

Voici l'erreur numéro un des débutants en pixel art, et elle est invisible tant qu'on ne l'a pas vue une fois.

Quand vous agrandissez une image, le moteur graphique doit inventer des pixels. Par défaut, il **interpole** : il calcule des couleurs intermédiaires pour adoucir la transition. Sur une photographie, c'est excellent. Sur du pixel art, c'est un désastre : le sprite devient flou, comme une image trop zoomée.

```text
  Source (4 px)      FilterQuality.none      FilterQuality.medium
                     (agrandi x4)            (agrandi x4)

  ████░░░░           ████████░░░░░░░░        ██████▓▓▒▒░░░░░░
                     ████████░░░░░░░░        ██████▓▓▒▒░░░░░░
                     ████████░░░░░░░░        █████▓▓▒▒░░░░░░░
                     ████████░░░░░░░░        ████▓▓▒▒░░░░░░░░
                     bords francs            bords dégradés = flou
```

La correction tient en une ligne :

```dart
final Paint p = Paint()..filterQuality = FilterQuality.none;
```

### Les valeurs de `FilterQuality`

| Valeur | Algorithme | Usage |
| --- | --- | --- |
| `none` | plus proche voisin | **pixel art** |
| `low` | bilinéaire | photos, illustrations réduites |
| `medium` | bilinéaire + mipmaps | réductions importantes |
| `high` | bicubique | qualité maximale, coûteux |

Pour un jeu en pixel art, la réponse est toujours `none`. Elle est en prime la **moins coûteuse** en calcul : vous gagnez en netteté et en performance.

### Le deuxième réglage : `isAntiAlias`

L'anti-crénelage lisse les bords des formes. Sur un sprite pixel art, il crée un liseré semi-transparent d'un pixel autour de l'image.

```dart
final Paint p = Paint()
  ..filterQuality = FilterQuality.none
  ..isAntiAlias = false;
```

Retenez ces deux lignes comme un bloc indissociable. Faites-en une constante de votre projet :

```dart
/// Le Paint standard pour tout sprite en pixel art du Donjon de Dart.
final Paint paintPixelArt = Paint()
  ..filterQuality = FilterQuality.none
  ..isAntiAlias = false;
```

### Programme de démonstration

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  return enregistreur.endRecording().toImage(largeur, hauteur);
}

/// Un damier 8x8 : le pire cas pour le lissage, donc le meilleur test.
Future<ui.Image> creerDamier() {
  return imageDepuisDessin(8, 8, (Canvas canvas) {
    final Paint clair = Paint()..color = const Color(0xFFFFE9A8);
    final Paint fonce = Paint()..color = const Color(0xFF7A3B12);
    for (int y = 0; y < 8; y++) {
      for (int x = 0; x < 8; x++) {
        final bool pair = (x + y) % 2 == 0;
        canvas.drawRect(
          Rect.fromLTWH(x.toDouble(), y.toDouble(), 1, 1),
          pair ? clair : fonce,
        );
      }
    }
  });
}

void main() => runApp(const AppFiltre());

class AppFiltre extends StatelessWidget {
  const AppFiltre({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF141420),
        body: EcranFiltre(),
      ),
    );
  }
}

class EcranFiltre extends StatefulWidget {
  const EcranFiltre({super.key});

  @override
  State<EcranFiltre> createState() => _EcranFiltreState();
}

class _EcranFiltreState extends State<EcranFiltre> {
  ui.Image? _damier;

  @override
  void initState() {
    super.initState();
    creerDamier().then((ui.Image img) {
      if (!mounted) return;
      setState(() => _damier = img);
    });
  }

  @override
  void dispose() {
    _damier?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? damier = _damier;
    if (damier == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(painter: PeintreFiltre(damier), size: Size.infinite);
  }
}

class PeintreFiltre extends CustomPainter {
  PeintreFiltre(this.damier);

  final ui.Image damier;

  final Paint _net = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
  final Paint _flou = Paint()
    ..filterQuality = FilterQuality.high;

  @override
  void paint(Canvas canvas, Size size) {
    const Rect src = Rect.fromLTWH(0, 0, 8, 8);

    canvas.drawImageRect(damier, src, const Rect.fromLTWH(30, 60, 160, 160), _net);
    canvas.drawImageRect(damier, src, const Rect.fromLTWH(220, 60, 160, 160), _flou);

    final TextPainter tp = TextPainter(
      textDirection: TextDirection.ltr,
      text: const TextSpan(
        text: 'none (net)              high (flou)',
        style: TextStyle(color: Color(0xFFEEEEEE), fontSize: 14),
      ),
    )..layout();
    tp.paint(canvas, const Offset(30, 235));
  }

  @override
  bool shouldRepaint(PeintreFiltre oldDelegate) => false;
}
```

**Résultat :**

```text
Deux damiers agrandis 20 fois, côte à côte.
À gauche : carrés parfaitement carrés, arêtes tranchantes.
À droite : la même image, mais chaque case fond dans la suivante ;
l'ensemble ressemble à une photo mal redimensionnée.
```

Lancez ce programme une fois. Vous n'oublierez plus jamais `FilterQuality.none`.

---

## 22.13 — Retourner un sprite horizontalement

Votre héros marche vers la droite. Il doit aussi pouvoir marcher vers la gauche.

La mauvaise réponse est de dessiner une seconde série d'images. Cela double le travail graphique, double la mémoire, et double le risque d'incohérence.

La bonne réponse est de **retourner** le sprite au moment du dessin, avec une mise à l'échelle négative :

```dart
canvas.scale(-1, 1); // -1 en X = miroir horizontal, 1 en Y = pas de changement
```

### Le piège du repère

Une échelle négative retourne l'axe X **autour de l'origine du repère**. Si vous vous contentez d'un `scale(-1, 1)`, votre sprite part hors de l'écran, du côté des X négatifs.

```text
  AVANT scale(-1, 1)                APRÈS scale(-1, 1)

        0                                    0
   ─────┼──────────────>  X          <───────┼─────
        │ ┌─────┐                     ┌─────┐│
        │ │  ▶  │                     │  ◀  ││
        │ └─────┘                     └─────┘│
       le sprite est à droite      il est passé à GAUCHE de 0,
       de l'origine                donc hors de l'écran
```

Il faut donc **translater d'abord**, à la bonne position, puis retourner.

### La recette complète

```dart
void dessinerSprite(
  Canvas canvas,
  ui.Image image,
  Rect source,
  Rect destination,
  Paint paint, {
  bool retourne = false,
}) {
  if (!retourne) {
    canvas.drawImageRect(image, source, destination, paint);
    return;
  }

  canvas.save();
  // 1. On place l'origine au bord DROIT du rectangle de destination.
  canvas.translate(destination.right, destination.top);
  // 2. On inverse l'axe X.
  canvas.scale(-1, 1);
  // 3. On dessine en (0, 0) avec la même taille : le sprite retombe pile
  //    sur le rectangle voulu, mais en miroir.
  canvas.drawImageRect(
    image,
    source,
    Rect.fromLTWH(0, 0, destination.width, destination.height),
    paint,
  );
  canvas.restore();
}
```

Le raisonnement, étape par étape :

```text
  destination = Rect.fromLTWH(100, 50, 64, 64)   → right = 164

  translate(164, 50)   l'origine du repère est au coin haut-DROIT
  scale(-1, 1)         l'axe X part maintenant vers la gauche
  drawImageRect en (0,0,64,64)
                       le sprite s'étend de 164 vers 100 : exactement
                       la zone voulue, mais retourné
```

### Variante : retournement autour du centre

Certains préfèrent raisonner par le centre :

```dart
canvas.save();
canvas.translate(destination.center.dx, destination.center.dy);
canvas.scale(-1, 1);
canvas.translate(-destination.center.dx, -destination.center.dy);
canvas.drawImageRect(image, source, destination, paint);
canvas.restore();
```

Ce triptyque « translate au centre, transforme, translate en sens inverse » est un motif universel du dessin 2D. Vous le retrouverez pour les rotations au chapitre 23.

### Le `save`/`restore` n'est pas facultatif

Sans `restore`, l'axe X reste inversé pour **tout le reste de la frame** : le décor, les ennemis, le score. Le bug est spectaculaire et déroutant. Prenez l'habitude d'écrire `canvas.save()` et `canvas.restore()` ensemble, immédiatement, avant de remplir le bloc.

### Retenir la direction, pas l'image

Côté logique, on ne stocke pas deux jeux de sprites, mais un simple booléen :

```dart
class Heros {
  double x = 100;
  double vitesseX = 0;
  bool regardeAGauche = false;

  void majDirection() {
    if (vitesseX > 0) {
      regardeAGauche = false;
    } else if (vitesseX < 0) {
      regardeAGauche = true;
    }
    // vitesseX == 0 : on garde la dernière direction connue
  }
}
```

Le `else if` est important : quand le personnage s'arrête, il doit **continuer à regarder** du côté où il allait, pas se retourner arbitrairement. C'est un détail qui distingue immédiatement un jeu soigné d'un prototype.

---

## 22.14 — Qu'est-ce qu'une sprite sheet ?

Une **sprite sheet** (« planche de sprites », ou « feuille de sprites ») est une seule image contenant plusieurs sprites rangés en grille.

Voici la planche 4 × 3 que nous allons utiliser dans tout le reste du chapitre. Chaque case fait 32 × 32 pixels ; l'image complète fait donc 128 × 96.

```text
              0        32       64       96      128
              ├────────┼────────┼────────┼────────┤
              │        │        │        │        │
              │  idx 0 │  idx 1 │  idx 2 │  idx 3 │   ligne 0
     0 ────── │  IDLE  │  IDLE  │  IDLE  │  IDLE  │   « repos »
              │        │        │        │        │
    32 ────── ├────────┼────────┼────────┼────────┤
              │  idx 4 │  idx 5 │  idx 6 │  idx 7 │   ligne 1
              │  WALK  │  WALK  │  WALK  │  WALK  │   « marche »
              │        │        │        │        │
    64 ────── ├────────┼────────┼────────┼────────┤
              │  idx 8 │  idx 9 │ idx 10 │ idx 11 │   ligne 2
              │ ATTACK │ ATTACK │ ATTACK │ ATTACK │   « attaque »
              │        │        │        │        │
    96 ────── └────────┴────────┴────────┴────────┘

      largeur de l'image  = 4 colonnes x 32 = 128 px
      hauteur de l'image  = 3 lignes   x 32 =  96 px
      nombre total de frames = 4 x 3 = 12
```

Le vocabulaire à fixer maintenant :

| Terme | Définition |
| --- | --- |
| **frame** | une case de la planche, c'est-à-dire une image d'animation |
| **largeur de frame** (`frameWidth`) | largeur d'une case en pixels : 32 |
| **hauteur de frame** (`frameHeight`) | hauteur d'une case en pixels : 32 |
| **colonnes** | nombre de cases par ligne : 4 |
| **lignes** | nombre de rangées : 3 |
| **index** | numéro de la frame, de 0 à 11, lu de gauche à droite puis de haut en bas |

La convention d'organisation la plus répandue : **une ligne = une animation**. Ligne 0 pour le repos, ligne 1 pour la marche, ligne 2 pour l'attaque. Rien ne vous y oblige, mais c'est ce que font la plupart des packs que vous téléchargerez, et c'est ce qui rend le code le plus lisible.

Certains packs utilisent au contraire une organisation en **bande** : toutes les frames sur une seule ligne, et un fichier de description qui dit « les frames 0 à 3 sont le repos, les frames 4 à 9 la marche ». Nous verrons ce cas en 22.28.

---

## 22.15 — Pourquoi une planche plutôt que douze fichiers

Une question légitime : pourquoi ne pas simplement mettre douze fichiers `heros_00.png` à `heros_11.png` ?

Il y a quatre raisons, de la plus technique à la plus pratique.

### Raison 1 : les changements de texture coûtent cher

Le processeur graphique dessine par **lots**. Il peut enchaîner des milliers de dessins provenant de la même texture sans ralentir. En revanche, chaque fois qu'il faut changer de texture, il doit interrompre le lot en cours et en recommencer un.

```text
  12 fichiers séparés :

  drawImage(heros_00) ─┐  changement de texture
  drawImage(gobelin)  ─┤  changement de texture
  drawImage(heros_01) ─┤  changement de texture
                       └─> 3 lots de rendu

  1 planche :

  drawImageRect(planche, frame 0)  ─┐
  drawImageRect(planche, frame 5)  ─┤  même texture
  drawImageRect(planche, frame 2)  ─┘  1 seul lot de rendu
```

Sur un téléphone, la différence entre 60 lots et 5 lots par frame est parfaitement mesurable.

### Raison 2 : la mémoire vidéo est allouée par puissances de deux

Beaucoup de processeurs graphiques arrondissent la taille d'une texture à la puissance de deux supérieure. Une image de 32 × 32 peut donc occuper la même place qu'une de 64 × 64. Douze petites images gaspillent énormément ; une planche de 128 × 96 est bien plus économe.

### Raison 3 : douze chargements asynchrones au lieu d'un

Chaque `rootBundle.load` est une opération asynchrone avec son coût fixe. Un fichier au lieu de douze, c'est un `await` au lieu de douze, et un écran de chargement bien plus court.

### Raison 4 : la cohérence

Quand les douze frames sont dans la même image, le dessinateur voit immédiatement si une frame est décalée d'un pixel, si une couleur ne correspond pas, si le personnage « saute » entre deux poses. Sur douze fichiers séparés, ces défauts se découvrent une fois le jeu lancé.

### Tableau récapitulatif

| Critère | 12 fichiers | 1 planche |
| --- | --- | --- |
| Lots de rendu | jusqu'à 12 | 1 |
| Mémoire vidéo | gaspillée | optimale |
| Chargements asynchrones | 12 | 1 |
| Lignes dans `pubspec.yaml` | 12 (ou un dossier) | 1 |
| Cohérence visuelle | difficile à contrôler | immédiate |
| Complexité du code | triviale | un calcul de `Rect` |

Le seul coût de la planche est ce calcul de `Rect`. Il tient en deux lignes, et nous allons l'écrire une bonne fois pour toutes.

---

## 22.16 — Calculer le `Rect` source d'une frame

Reprenons la planche 4 × 3. Comment obtenir le rectangle source de la frame située ligne 1, colonne 2 ?

```text
   colonne :   0        1        2        3
             ┌────────┬────────┬────────┬────────┐
  ligne 0    │        │        │        │        │
             ├────────┼────────┼────────┼────────┤
  ligne 1    │        │        │ ◀ ICI  │        │
             ├────────┼────────┼────────┼────────┤
  ligne 2    │        │        │        │        │
             └────────┴────────┴────────┴────────┘

   left   = colonne x largeurFrame = 2 x 32 = 64
   top    = ligne   x hauteurFrame = 1 x 32 = 32
   width  = largeurFrame  = 32
   height = hauteurFrame  = 32
```

En code :

```dart
Rect rectFrame(int ligne, int colonne, double largeurFrame, double hauteurFrame) {
  return Rect.fromLTWH(
    colonne * largeurFrame,
    ligne * hauteurFrame,
    largeurFrame,
    hauteurFrame,
  );
}
```

Deux points de vigilance.

**Attention à l'ordre des arguments.** `Rect.fromLTWH` attend `left` en premier, c'est-à-dire la coordonnée **X**, donc la **colonne**. Inverser ligne et colonne est l'erreur la plus fréquente de cette section, et elle est difficile à repérer sur une planche carrée. Sur une planche 4 × 3, elle provoque en revanche une exception ou une frame vide, ce qui est plus facile à diagnostiquer.

**Attention aux types.** `colonne` est un `int`, `largeurFrame` un `double`. En Dart, `int * double` donne bien un `double` (chapitre 3), donc l'expression est correcte. Mais si vous écrivez `colonne * 32`, vous obtenez un `int`, et `Rect.fromLTWH` refuse un `int` là où il attend un `double` :

```text
Error: The argument type 'int' can't be assigned to the parameter type 'double'.
```

La parade : écrivez `32.0` plutôt que `32`, ou appelez `.toDouble()`.

### Vérifier ses calculs

Ajoutez temporairement un contour vert autour de chaque `Rect` destination, et un compteur d'index à l'écran. Vous verrez immédiatement si vous piochez la bonne case. Un jeu se débogue à l'œil autant qu'au `print`.

---

## 22.17 — La formule ligne / colonne à partir d'un index

Manipuler un couple (ligne, colonne) est peu pratique dans une animation : on veut un seul compteur qui avance, 0, 1, 2, 3, 4… C'est l'**index de frame**.

Les deux formules de conversion sont les suivantes :

```dart
final int ligne   = index ~/ colonnes; // division ENTIÈRE
final int colonne = index %  colonnes; // RESTE de la division
```

`~/` est la division entière et `%` le modulo, tous deux vus au chapitre 3.

### Vérification sur la planche 4 × 3

Avec `colonnes = 4` :

| index | `index ~/ 4` (ligne) | `index % 4` (colonne) | Case |
| --- | --- | --- | --- |
| 0 | 0 | 0 | ligne 0, colonne 0 |
| 1 | 0 | 1 | ligne 0, colonne 1 |
| 2 | 0 | 2 | ligne 0, colonne 2 |
| 3 | 0 | 3 | ligne 0, colonne 3 |
| 4 | 1 | 0 | ligne 1, colonne 0 |
| 5 | 1 | 1 | ligne 1, colonne 1 |
| 6 | 1 | 2 | ligne 1, colonne 2 |
| 7 | 1 | 3 | ligne 1, colonne 3 |
| 8 | 2 | 0 | ligne 2, colonne 0 |
| 9 | 2 | 1 | ligne 2, colonne 1 |
| 10 | 2 | 2 | ligne 2, colonne 2 |
| 11 | 2 | 3 | ligne 2, colonne 3 |

Le tableau se lit exactement comme le schéma de la section 22.14. L'index avance de gauche à droite, puis passe à la ligne suivante — comme une lecture.

### Le moyen mnémotechnique

```text
  Le MODULO donne la position DANS la ligne  → la COLONNE
  La DIVISION dit COMBIEN de lignes complètes → la LIGNE
```

Une image plus parlante encore : imaginez 11 pièces d'or et des sacs contenant 4 pièces chacun.

```text
  11 pièces, sacs de 4

  11 ~/ 4 = 2  →  2 sacs pleins    (= 2 lignes complètes au-dessus)
  11 %  4 = 3  →  3 pièces restantes (= 3e colonne, en partant de 0)
```

### Le sens inverse

De temps en temps, on a besoin du chemin retour :

```dart
final int index = ligne * colonnes + colonne;
```

Vérification : ligne 1, colonne 2, avec 4 colonnes donne `1 * 4 + 2 = 6`. Le tableau confirme.

### Programme de vérification

Ce petit programme s'exécute dans DartPad, sans Flutter :

```dart
void main() {
  const int colonnes = 4;
  const int lignes = 3;
  const int total = colonnes * lignes;

  for (int index = 0; index < total; index++) {
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    final int retour = ligne * colonnes + colonne;

    final double left = colonne * 32.0;
    final double top = ligne * 32.0;

    print('index $index -> ligne $ligne, colonne $colonne '
        '| Rect(l=$left, t=$top, 32, 32) | retour=$retour');
  }
}
```

**Résultat :**

```text
index 0 -> ligne 0, colonne 0 | Rect(l=0.0, t=0.0, 32, 32) | retour=0
index 1 -> ligne 0, colonne 1 | Rect(l=32.0, t=0.0, 32, 32) | retour=1
index 2 -> ligne 0, colonne 2 | Rect(l=64.0, t=0.0, 32, 32) | retour=2
index 3 -> ligne 0, colonne 3 | Rect(l=96.0, t=0.0, 32, 32) | retour=3
index 4 -> ligne 1, colonne 0 | Rect(l=0.0, t=32.0, 32, 32) | retour=4
index 5 -> ligne 1, colonne 1 | Rect(l=32.0, t=32.0, 32, 32) | retour=5
index 6 -> ligne 1, colonne 2 | Rect(l=64.0, t=32.0, 32, 32) | retour=6
index 7 -> ligne 1, colonne 3 | Rect(l=96.0, t=32.0, 32, 32) | retour=7
index 8 -> ligne 2, colonne 0 | Rect(l=0.0, t=64.0, 32, 32) | retour=8
index 9 -> ligne 2, colonne 1 | Rect(l=32.0, t=64.0, 32, 32) | retour=9
index 10 -> ligne 2, colonne 2 | Rect(l=64.0, t=64.0, 32, 32) | retour=10
index 11 -> ligne 2, colonne 3 | Rect(l=96.0, t=64.0, 32, 32) | retour=11
```

---

## 22.18 — Une classe `SpriteSheet` maison

Nous avons maintenant tous les ingrédients. Rangeons-les dans une classe, comme au chapitre 9.

### Le cahier des charges

La classe doit :

1. contenir l'image et la géométrie de la grille ;
2. calculer le nombre de colonnes et de lignes toute seule ;
3. donner le `Rect` source de n'importe quel index ;
4. donner le `Rect` source de n'importe quel couple (ligne, colonne) ;
5. extraire une liste d'index correspondant à une ligne entière ;
6. dessiner une frame à une position et une échelle données, éventuellement retournée ;
7. refuser proprement un index hors limites (chapitre 13).

### Le code

```dart
class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  /// Rectangle source de la frame d'index [index] (0 = coin haut-gauche).
  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index',
          'Index de frame hors de la planche');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * largeurFrame.toDouble(),
      ligne * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  /// Rectangle source par coordonnées de grille.
  Rect rectDeCase(int ligne, int colonne) => rectDe(ligne * colonnes + colonne);

  /// Les index d'une ligne entière : pratique pour définir une animation.
  List<int> indexLigne(int ligne, {int? depuis, int? combien}) {
    final int debut = ligne * colonnes + (depuis ?? 0);
    final int nombre = combien ?? colonnes - (depuis ?? 0);
    return List<int>.generate(nombre, (int i) => debut + i);
  }

  /// Dessine la frame [index], centrée horizontalement sur [x],
  /// pieds posés sur [y], agrandie [echelle] fois.
  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
    Paint? paint,
  }) {
    final Paint p = paint ?? _paintDefaut;
    final Rect src = rectDe(index);
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(x - l / 2, y - h, l, h);

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, p);
      return;
    }

    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, l, h), p);
    canvas.restore();
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}
```

### Ce que ce code réutilise de la PARTIE 1A

| Élément | Chapitre |
| --- | --- |
| constructeur nommé avec `required` | 9 |
| liste d'initialisation (`: colonnes = ...`) | 9 |
| champs `final` (immuabilité) | 10 |
| getter calculé (`nombreFrames`) | 8 |
| `List.generate` | 6 |
| paramètres nommés optionnels et `??` | 7 et 12 |
| `RangeError` explicite | 13 |
| membre statique privé `_paintDefaut` | 10 |

### Pourquoi `image.width ~/ largeurFrame`

Le constructeur déduit la grille de la taille de l'image. Cela évite d'écrire trois fois la même information et supprime toute possibilité d'incohérence.

Attention : si l'image ne se divise pas exactement, la division entière **tronque**. Une planche de 130 pixels de large en frames de 32 donne `130 ~/ 32 = 4` colonnes, et 2 pixels sont ignorés silencieusement. C'est souvent le signe que la planche contient une marge ou des séparateurs — voir la section 22.27.

Pour se protéger, on peut ajouter une assertion :

```dart
assert(
  image.width % largeurFrame == 0,
  'La largeur de la planche (${image.width}) n\'est pas un multiple '
  'de la largeur de frame ($largeurFrame).',
);
```

Les `assert` ne s'exécutent qu'en mode debug : ils ne coûtent rien dans le jeu publié, et ils vous sauvent pendant le développement.

### Générer une planche 4 × 3 en code

Puisque nous n'avons pas de fichier, fabriquons la planche décrite en 22.14.

```dart
/// Planche 4 colonnes x 3 lignes, frames de 32x32, soit 128x96.
/// Ligne 0 : repos (respiration)
/// Ligne 1 : marche (jambes qui bougent)
/// Ligne 2 : attaque (épée qui sort)
Future<ui.Image> creerPlancheHeros() {
  return imageDepuisDessin(128, 96, (Canvas canvas) {
    for (int ligne = 0; ligne < 3; ligne++) {
      for (int colonne = 0; colonne < 4; colonne++) {
        canvas.save();
        canvas.translate(colonne * 32.0, ligne * 32.0);
        _dessinerHeros(canvas, ligne, colonne);
        canvas.restore();
      }
    }
  });
}

void _dessinerHeros(Canvas canvas, int ligne, int colonne) {
  final Paint peau = Paint()..color = const Color(0xFFF2C79B);
  final Paint tunique = Paint()..color = const Color(0xFF3A7BD5);
  final Paint bottes = Paint()..color = const Color(0xFF2A2A38);
  final Paint acier = Paint()..color = const Color(0xFFD8DEE9);

  // Respiration : le corps monte et descend d'un pixel.
  final double souffle = (ligne == 0 && (colonne == 1 || colonne == 3)) ? 1 : 0;

  // Écartement des jambes pour la marche.
  const List<double> pas = <double>[0, 3, 0, -3];
  final double ecart = (ligne == 1) ? pas[colonne] : 0;

  canvas.drawRect(Rect.fromLTWH(11, 3 + souffle, 10, 9), peau);
  canvas.drawRect(Rect.fromLTWH(10, 12 + souffle, 12, 11), tunique);
  canvas.drawRect(Rect.fromLTWH(11 - ecart, 23, 4, 7), bottes);
  canvas.drawRect(Rect.fromLTWH(17 + ecart, 23, 4, 7), bottes);

  // Attaque : une épée qui sort progressivement vers la droite.
  if (ligne == 2) {
    const List<double> longueur = <double>[0, 5, 11, 6];
    final double lame = longueur[colonne];
    if (lame > 0) {
      canvas.drawRect(Rect.fromLTWH(22, 15, lame, 2), acier);
    }
  }
}
```

Chaque case est dessinée dans son propre repère grâce à `translate` : on écrit le héros comme s'il était seul, entre (0, 0) et (32, 32), et la translation le place dans la bonne case. C'est exactement la technique du chapitre 21, appliquée à la fabrication d'assets.

### Programme complet : afficher la planche et ses frames

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  return enregistreur.endRecording().toImage(largeur, hauteur);
}

void _dessinerHeros(Canvas canvas, int ligne, int colonne) {
  final Paint peau = Paint()..color = const Color(0xFFF2C79B);
  final Paint tunique = Paint()..color = const Color(0xFF3A7BD5);
  final Paint bottes = Paint()..color = const Color(0xFF2A2A38);
  final Paint acier = Paint()..color = const Color(0xFFD8DEE9);

  final double souffle = (ligne == 0 && (colonne == 1 || colonne == 3)) ? 1 : 0;
  const List<double> pas = <double>[0, 3, 0, -3];
  final double ecart = (ligne == 1) ? pas[colonne] : 0;

  canvas.drawRect(Rect.fromLTWH(11, 3 + souffle, 10, 9), peau);
  canvas.drawRect(Rect.fromLTWH(10, 12 + souffle, 12, 11), tunique);
  canvas.drawRect(Rect.fromLTWH(11 - ecart, 23, 4, 7), bottes);
  canvas.drawRect(Rect.fromLTWH(17 + ecart, 23, 4, 7), bottes);

  if (ligne == 2) {
    const List<double> longueur = <double>[0, 5, 11, 6];
    final double lame = longueur[colonne];
    if (lame > 0) {
      canvas.drawRect(Rect.fromLTWH(22, 15, lame, 2), acier);
    }
  }
}

Future<ui.Image> creerPlancheHeros() {
  return imageDepuisDessin(128, 96, (Canvas canvas) {
    for (int ligne = 0; ligne < 3; ligne++) {
      for (int colonne = 0; colonne < 4; colonne++) {
        canvas.save();
        canvas.translate(colonne * 32.0, ligne * 32.0);
        _dessinerHeros(canvas, ligne, colonne);
        canvas.restore();
      }
    }
  });
}

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * largeurFrame.toDouble(),
      ligne * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  Rect rectDeCase(int ligne, int colonne) => rectDe(ligne * colonnes + colonne);

  List<int> indexLigne(int ligne, {int? depuis, int? combien}) {
    final int debut = ligne * colonnes + (depuis ?? 0);
    final int nombre = combien ?? colonnes - (depuis ?? 0);
    return List<int>.generate(nombre, (int i) => debut + i);
  }

  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
    Paint? paint,
  }) {
    final Paint p = paint ?? paintDefaut;
    final Rect src = rectDe(index);
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(x - l / 2, y - h, l, h);

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, p);
      return;
    }

    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, l, h), p);
    canvas.restore();
  }

  static final Paint paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

void main() => runApp(const AppPlanche());

class AppPlanche extends StatelessWidget {
  const AppPlanche({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF141420),
        body: EcranPlanche(),
      ),
    );
  }
}

class EcranPlanche extends StatefulWidget {
  const EcranPlanche({super.key});

  @override
  State<EcranPlanche> createState() => _EcranPlancheState();
}

class _EcranPlancheState extends State<EcranPlanche> {
  SpriteSheet? _planche;

  @override
  void initState() {
    super.initState();
    creerPlancheHeros().then((ui.Image img) {
      if (!mounted) return;
      setState(() {
        _planche = SpriteSheet(image: img, largeurFrame: 32, hauteurFrame: 32);
      });
    });
  }

  @override
  void dispose() {
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(painter: PeintrePlanche(planche), size: Size.infinite);
  }
}

class PeintrePlanche extends CustomPainter {
  PeintrePlanche(this.planche);

  final SpriteSheet planche;
  final Paint _grille = Paint()
    ..color = const Color(0xFF44FF88)
    ..style = PaintingStyle.stroke
    ..strokeWidth = 1;

  @override
  void paint(Canvas canvas, Size size) {
    const double e = 2; // échelle de la planche entière

    // 1. La planche complète, agrandie 2x, avec sa grille.
    final Rect dstPlanche = Rect.fromLTWH(
      20, 20,
      planche.image.width * e,
      planche.image.height * e,
    );
    canvas.drawImageRect(
      planche.image,
      Rect.fromLTWH(0, 0, planche.image.width.toDouble(),
          planche.image.height.toDouble()),
      dstPlanche,
      SpriteSheet.paintDefaut,
    );
    for (int l = 0; l <= planche.lignes; l++) {
      final double y = 20 + l * planche.hauteurFrame * e;
      canvas.drawLine(Offset(20, y), Offset(dstPlanche.right, y), _grille);
    }
    for (int c = 0; c <= planche.colonnes; c++) {
      final double x = 20 + c * planche.largeurFrame * e;
      canvas.drawLine(Offset(x, 20), Offset(x, dstPlanche.bottom), _grille);
    }

    // 2. Les 12 frames extraites une par une, agrandies 3x.
    for (int i = 0; i < planche.nombreFrames; i++) {
      final int ligne = i ~/ planche.colonnes;
      final int colonne = i % planche.colonnes;
      final double x = 40 + colonne * 110.0;
      final double y = 260 + ligne * 110.0;
      planche.dessiner(canvas, i, x, y + 96, echelle: 3);
    }
  }

  @override
  bool shouldRepaint(PeintrePlanche oldDelegate) => false;
}
```

**Résultat :**

```text
En haut : la planche 128x96 agrandie 2x, quadrillée en vert : 4 colonnes, 3 lignes.
En dessous : les 12 frames extraites, disposées dans la même grille, agrandies 3x.
Ligne 0 : le héros immobile, qui « respire » (frames 1 et 3 décalées d'un pixel).
Ligne 1 : le héros avec les jambes plus ou moins écartées.
Ligne 2 : le héros dont l'épée sort progressivement vers la droite.
```

---

## 22.19 — Qu'est-ce qu'une animation ?

Une animation est une **liste ordonnée de frames** que l'on affiche successivement, assez vite pour que l'œil perçoive un mouvement continu.

```text
  temps  ──────────────────────────────────────────────>

  frame  │ 4 │ 5 │ 6 │ 7 │ 4 │ 5 │ 6 │ 7 │ 4 │ 5 │ ...
         └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
           marche : les index 4 à 7, en boucle
```

Une animation n'est donc **pas** une image : c'est une donnée composée de trois éléments.

| Élément | Type | Rôle |
| --- | --- | --- |
| les frames | `List<int>` | les index de la planche à afficher, dans l'ordre |
| la cadence | `double` | combien de temps chaque frame reste affichée |
| le bouclage | `bool` | recommencer au début, ou rester sur la dernière |

Notez que les frames sont une **liste**, et non un intervalle. Cela permet des séquences non contiguës et des allers-retours :

```dart
final List<int> marche = <int>[4, 5, 6, 7];        // cycle simple
final List<int> respire = <int>[0, 1, 2, 1];       // aller-retour (ping-pong)
final List<int> clignote = <int>[0, 0, 0, 0, 1];   // pause longue, puis un clin d'œil
```

Ce dernier exemple montre une astuce courante : répéter un index pour le faire durer plus longtemps, sans changer la cadence globale.

---

## 22.20 — La cadence : `stepTime`

Le **`stepTime`** (« temps par pas ») est la durée d'affichage d'une frame, exprimée **en secondes**.

```text
  stepTime = 0.1 s

  0.0s    0.1s    0.2s    0.3s    0.4s
   │       │       │       │       │
   ├───────┼───────┼───────┼───────┤
   │frame 4│frame 5│frame 6│frame 7│frame 4 ...
```

Les relations à connaître :

```text
  stepTime            = durée d'une frame, en secondes
  1 / stepTime        = nombre de frames d'animation par seconde
  frames x stepTime   = durée d'un cycle complet
```

Exemple : 4 frames à 0,1 s font un cycle de 0,4 seconde, soit 2,5 cycles de marche par seconde.

### Valeurs usuelles

| Action | `stepTime` | Effet |
| --- | --- | --- |
| respiration au repos | 0,20 à 0,30 s | lent, presque imperceptible |
| marche | 0,10 à 0,15 s | rythme naturel |
| course | 0,05 à 0,08 s | rapide, énergique |
| attaque | 0,05 à 0,10 s | sec |
| mort | 0,15 à 0,25 s | lourd, appuyé |
| flamme de torche | 0,08 s | nerveux |

### Ne confondez jamais `stepTime` et FPS

C'est une confusion classique, et elle vaut d'être posée noir sur blanc.

| Notion | Ce que c'est | Valeur typique |
| --- | --- | --- |
| **FPS du jeu** | nombre de fois par seconde où l'écran est redessiné | 60 |
| **FPS de l'animation** | nombre de changements de frame par seconde | 8 à 12 |

Une animation en pixel art à 60 changements de frame par seconde serait illisible : l'œil ne suivrait rien. Les jeux 2D classiques tournent entre 8 et 12 frames d'animation par seconde, tout en affichant l'écran 60 fois par seconde.

```text
  Écran   : █ █ █ █ █ █ █ █ █ █ █ █   (60 fois par seconde)
  Frame   : 4 4 4 4 4 4 5 5 5 5 5 5   (change toutes les 6 images)
```

Un `stepTime` de 0,1 s à 60 FPS signifie que chaque frame est affichée pendant 6 rafraîchissements d'écran. C'est normal, et c'est même souhaitable.

---

## 22.21 — Faire avancer l'animation avec le delta time

Au chapitre 20, vous avez appris la règle d'or de la boucle de jeu :

> Ne comptez jamais en nombre de frames. Comptez toujours en secondes écoulées.

Cette règle s'applique intégralement aux animations. Le code fautif ressemble à ceci :

```dart
// MAUVAIS : dépend du framerate
compteur++;
if (compteur >= 6) {
  compteur = 0;
  indexFrame++;
}
```

Ce code change de frame toutes les 6 images affichées. Sur un écran 60 Hz, cela fait 10 changements par seconde. Sur un écran 120 Hz — un téléphone récent, une tablette de jeu —, cela en fait 20 : **le personnage marche deux fois plus vite**. Sur une machine qui rame à 30 FPS, il marche deux fois moins vite.

La solution est un **accumulateur de temps** :

```dart
// BON : indépendant du framerate
tempsAccumule += dt; // dt en secondes, fourni par la boucle de jeu
while (tempsAccumule >= stepTime) {
  tempsAccumule -= stepTime;
  indexCourant++;
}
```

### Pourquoi un `while` et non un `if`

Le `while` traite le cas où plusieurs frames doivent défiler pendant le même intervalle.

```text
  stepTime = 0.1 s

  Cas normal :  dt = 0.016 s  →  la boucle ne tourne pas toujours
  Cas de lag :  dt = 0.35 s   →  la boucle tourne 3 fois
                                  (0.35 = 3 x 0.1 + 0.05)
```

Avec un simple `if`, une frame de 0,35 seconde ne ferait avancer l'animation que d'un cran : l'animation prendrait du retard et se désynchroniserait du reste du jeu, définitivement.

### Pourquoi `-=` et non `= 0`

Remettre l'accumulateur à zéro jetterait le reste. Avec `tempsAccumule -= stepTime`, les 0,05 seconde restantes sont conservées et comptent pour la frame suivante. C'est ce qui garantit qu'à long terme, l'animation tourne exactement à la bonne vitesse.

```text
  dt = 0.016 s à chaque frame, stepTime = 0.1 s

  frame 1 : acc = 0.016
  frame 2 : acc = 0.032
  ...
  frame 7 : acc = 0.112  →  0.112 >= 0.1  →  index++, acc = 0.012
  frame 8 : acc = 0.028
  ...
  Le reliquat de 0.012 n'est pas perdu : il avance la suite.
```

### Le garde-fou du gros `dt`

Si l'application est mise en arrière-plan puis revient, `dt` peut valoir plusieurs secondes. Le `while` tournerait alors des dizaines de fois. Il est prudent de plafonner `dt` dans la boucle de jeu :

```dart
final double dtSecurise = dt > 0.05 ? 0.05 : dt;
```

C'est le même plafonnement que celui recommandé au chapitre 20 pour la physique.

---

## 22.22 — Une classe d'animation maison

Écrivons maintenant la classe qui encapsule tout cela.

> **Attention au nom.** Flutter possède déjà une classe `Animation<T>` (dans `package:flutter/animation.dart`, réexportée par `material.dart`). Si vous nommez votre classe `Animation`, le compilateur signalera un conflit dès que vous importerez `material.dart` :
>
> ```text
> Error: 'Animation' is imported from both 'package:flutter/src/animation/animation.dart' and 'main.dart'.
> ```
>
> Nous l'appellerons donc **`SpriteAnimation`**. C'est aussi le nom que Flame utilise, ce qui vous préparera au chapitre 29.

### Le code

```dart
class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  /// Les index de frames dans la planche, dans l'ordre d'affichage.
  final List<int> frames;

  /// Durée d'affichage d'une frame, en secondes.
  final double stepTime;

  /// L'animation recommence-t-elle au début après la dernière frame ?
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  /// L'index de frame à dessiner MAINTENANT (index dans la planche).
  int get frameCourante => frames[_index];

  /// La position dans la séquence (0 pour la première frame).
  int get positionCourante => _index;

  /// Vrai uniquement pour une animation non bouclée arrivée au bout.
  bool get estTerminee => _terminee;

  /// Durée totale d'un cycle, en secondes.
  double get duree => frames.length * stepTime;

  /// Fait avancer l'animation de [dt] secondes.
  void avancer(double dt) {
    if (_terminee) return;

    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;

      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  /// Remet l'animation à sa première frame.
  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}
```

### Les points de conception à comprendre

**Les champs privés.** `_index`, `_accumulateur` et `_terminee` sont privés (chapitre 10). Personne, à l'extérieur, ne doit les modifier directement : l'état de l'animation n'évolue que par `avancer` et `reinitialiser`.

**Les getters en lecture seule.** `frameCourante`, `estTerminee` et `duree` exposent l'information utile sans exposer le mécanisme. C'est de l'encapsulation, pas de la coquetterie : si vous changez un jour la façon de compter, aucun code appelant ne casse.

**Le `assert` dans le constructeur.** Une animation vide provoquerait un `RangeError` sur `frames[_index]`, à l'exécution, loin de la cause. L'`assert` signale l'erreur au moment de la création, ce qui est bien plus facile à corriger.

**Le retour immédiat si `_terminee`.** Une animation de mort, une fois finie, ne doit plus rien faire. Ce `return` évite tout calcul inutile et fige la dernière frame.

### Utilisation

```dart
final SpriteAnimation marche = SpriteAnimation(
  frames: <int>[4, 5, 6, 7],
  stepTime: 0.12,
);

// Dans la boucle de jeu :
marche.avancer(dt);

// Dans le rendu :
planche.dessiner(canvas, marche.frameCourante, x, y, echelle: 4);
```

### Vérifier son comportement sans interface

On peut tester la classe en pur Dart, dans DartPad, en simulant une boucle de jeu :

```dart
void main() {
  final SpriteAnimation marche =
      SpriteAnimation(frames: <int>[4, 5, 6, 7], stepTime: 0.1);

  const double dt = 1 / 60; // 60 FPS
  double temps = 0;

  for (int i = 0; i < 40; i++) {
    marche.avancer(dt);
    temps += dt;
    if (i % 5 == 0) {
      print('t=${temps.toStringAsFixed(3)}s  frame=${marche.frameCourante}');
    }
  }
}
```

**Résultat :**

```text
t=0.017s  frame=4
t=0.100s  frame=5
t=0.183s  frame=5
t=0.267s  frame=6
t=0.350s  frame=7
t=0.433s  frame=8
```

La dernière ligne est fausse : `frame=8` n'existe pas dans la liste `[4, 5, 6, 7]`.

C'est volontaire : lisez de nouveau la classe. `frameCourante` renvoie `frames[_index]`, donc jamais 8. En réalité, `t=0.433s` correspond à `_index = 0` après bouclage, donc `frames[0] = 4`. La sortie réelle est :

```text
t=0.017s  frame=4
t=0.100s  frame=5
t=0.183s  frame=5
t=0.267s  frame=6
t=0.350s  frame=7
t=0.433s  frame=4
```

L'animation a bouclé après 0,4 seconde, exactement comme prévu par `duree = 4 x 0,1 = 0,4`.

> **Leçon de méthode :** ne croyez pas une sortie annoncée sans la vérifier. Écrivez la boucle de test, exécutez-la, et confrontez ce que vous voyez à ce que vous attendiez. C'est ainsi qu'on débogue une animation.

---

## 22.23 — Animations non bouclées : mort et attaque

Toutes les animations ne tournent pas en rond. Certaines se jouent **une seule fois**, puis s'arrêtent.

| Type | Exemples | `loop` |
| --- | --- | --- |
| cyclique | repos, marche, course, nage, flamme | `true` |
| unique | attaque, saut, dégât reçu, mort, ouverture de coffre | `false` |

Le comportement en fin de séquence diffère radicalement :

```text
  loop = true             loop = false

  4 5 6 7 4 5 6 7 ...     8 9 10 11 11 11 11 11 ...
  recommence               reste figé sur la DERNIÈRE frame
```

Rester sur la dernière frame est le comportement attendu : un personnage mort doit rester au sol, pas revenir à sa pose debout.

### Le drapeau `estTerminee`

Il permet de déclencher une suite. C'est la brique de base de toute la logique de combat :

```dart
void mettreAJour(double dt) {
  animationCourante.avancer(dt);

  if (etat == EtatHeros.attaque && animationCourante.estTerminee) {
    changerEtat(EtatHeros.repos);
  }

  if (etat == EtatHeros.mort && animationCourante.estTerminee) {
    afficherEcranGameOver();
  }
}
```

### Le piège de la réutilisation

Une animation non bouclée est **à usage unique** tant qu'on ne la réinitialise pas. Si le héros attaque une seconde fois, il faut appeler `reinitialiser()`, sinon `estTerminee` reste `true` et l'animation ne repart jamais.

```dart
void attaquer() {
  if (etat == EtatHeros.attaque) return; // déjà en train d'attaquer
  attaque.reinitialiser();               // INDISPENSABLE
  etat = EtatHeros.attaque;
}
```

Oublier ce `reinitialiser()` produit un bug typique : « mon héros n'attaque qu'une seule fois de toute la partie ». Vous savez maintenant où regarder.

### Chaîner deux animations

Un enchaînement classique : dégât reçu, puis mort si les points de vie tombent à zéro.

```dart
void subirDegats(int degats) {
  pointsDeVie -= degats;
  if (pointsDeVie <= 0) {
    pointsDeVie = 0;
    mort.reinitialiser();
    etat = EtatHeros.mort;
  } else {
    blesse.reinitialiser();
    etat = EtatHeros.blesse;
  }
}
```

---

## 22.24 — Plusieurs animations pour un personnage

Un personnage de plateforme complet possède en général six animations. Voici celles du héros du Donjon de Dart.

| Nom | Rôle | Frames typiques | `stepTime` | `loop` |
| --- | --- | --- | --- | --- |
| `idle` | debout, immobile, respiration | 4 | 0,20 | oui |
| `walk` | déplacement au sol | 6 | 0,12 | oui |
| `jump` | en l'air | 2 | 0,15 | non |
| `attack` | coup d'épée | 4 | 0,07 | non |
| `hurt` | dégât reçu | 2 | 0,10 | non |
| `die` | mort | 5 | 0,18 | non |

Sur une planche, on leur consacre en général une ligne chacune :

```text
              0        32       64       96      128      160      192
             ┌────────┬────────┬────────┬────────┬────────┬────────┐
  ligne 0    │ idle 0 │ idle 1 │ idle 2 │ idle 3 │  ····  │  ····  │
             ├────────┼────────┼────────┼────────┼────────┼────────┤
  ligne 1    │ walk 0 │ walk 1 │ walk 2 │ walk 3 │ walk 4 │ walk 5 │
             ├────────┼────────┼────────┼────────┼────────┼────────┤
  ligne 2    │ jump 0 │ jump 1 │  ····  │  ····  │  ····  │  ····  │
             ├────────┼────────┼────────┼────────┼────────┼────────┤
  ligne 3    │ atk 0  │ atk 1  │ atk 2  │ atk 3  │  ····  │  ····  │
             ├────────┼────────┼────────┼────────┼────────┼────────┤
  ligne 4    │ hurt 0 │ hurt 1 │  ····  │  ····  │  ····  │  ····  │
             ├────────┼────────┼────────┼────────┼────────┼────────┤
  ligne 5    │ die 0  │ die 1  │ die 2  │ die 3  │ die 4  │  ····  │
             └────────┴────────┴────────┴────────┴────────┴────────┘
              (« ···· » = case vide, laissée transparente)
```

Les lignes n'ont pas toutes la même longueur utile. C'est normal : on remplit la grille au format du plus long cycle, et on laisse les cases inutilisées transparentes. La méthode `indexLigne` de notre `SpriteSheet` accepte justement un paramètre `combien` pour ne prendre que les frames réellement dessinées :

```dart
final SpriteAnimation idle = SpriteAnimation(
  frames: planche.indexLigne(0, combien: 4), // [0, 1, 2, 3]
  stepTime: 0.20,
);
final SpriteAnimation walk = SpriteAnimation(
  frames: planche.indexLigne(1, combien: 6), // [6, 7, 8, 9, 10, 11]
  stepTime: 0.12,
);
final SpriteAnimation die = SpriteAnimation(
  frames: planche.indexLigne(5, combien: 5),
  stepTime: 0.18,
  loop: false,
);
```

Avec une planche de 6 colonnes, `indexLigne(1)` part de `1 * 6 = 6`. Vérifiez toujours ce calcul sur votre propre planche : c'est là que se logent les décalages d'une case.

---

## 22.25 — Une machine à états d'animation

Six animations, mais **une seule** à afficher à un instant donné. Comment choisir ?

La réponse est une **machine à états** : le personnage est toujours dans exactement un état, et chaque état a son animation.

Vous connaissez déjà l'outil : les `enum` du chapitre 11.

```dart
enum EtatHeros { repos, marche, saut, attaque, blesse, mort }
```

### Le diagramme des transitions

```text
                    ┌──────────────────────────────────┐
                    │                                  │
                    ▼           vitesse != 0           │
              ┌──────────┐ ─────────────────> ┌──────────────┐
              │  REPOS   │                    │    MARCHE    │
              │  (idle)  │ <───────────────── │    (walk)    │
              └──┬───┬───┘     vitesse == 0   └───┬──────────┘
                 │   │                            │
        touche   │   │  touche saut               │  touche saut
        attaque  │   └────────────┐   ┌───────────┘
                 │                ▼   ▼
                 │            ┌──────────────┐
                 │            │     SAUT     │
                 │            │    (jump)    │
                 │            └──────┬───────┘
                 ▼                   │ au sol
          ┌──────────────┐           │
          │   ATTAQUE    │ <─────────┘
          │   (attack)   │
          └──────┬───────┘
                 │ animation terminée
                 ▼
              (retour à REPOS ou MARCHE)

          ┌──────────────┐  pv <= 0   ┌──────────────┐
          │   BLESSÉ     │ ─────────> │     MORT     │
          │   (hurt)     │            │     (die)    │
          └──────────────┘            └──────────────┘
              animation                  état FINAL :
              terminée                   aucune sortie
              → retour                   possible
```

### Les priorités

Toutes les transitions ne se valent pas. La mort l'emporte sur tout, l'attaque l'emporte sur la marche. On code donc les tests **du plus prioritaire au moins prioritaire** :

```dart
void choisirEtat() {
  if (pointsDeVie <= 0) {
    _passerA(EtatHeros.mort);
    return;
  }
  if (etat == EtatHeros.blesse && !blesse.estTerminee) return;
  if (etat == EtatHeros.attaque && !attaque.estTerminee) return;
  if (!auSol) {
    _passerA(EtatHeros.saut);
    return;
  }
  if (vitesseX.abs() > 1) {
    _passerA(EtatHeros.marche);
  } else {
    _passerA(EtatHeros.repos);
  }
}
```

Les deux lignes du milieu se lisent ainsi : « si je suis en train de subir un dégât ou d'attaquer et que l'animation n'est pas finie, je ne change rien ». C'est ce qui empêche une attaque d'être interrompue au premier pas.

### Associer un état à une animation

Une `Map` (chapitre 6) fait le lien :

```dart
final Map<EtatHeros, SpriteAnimation> animations = <EtatHeros, SpriteAnimation>{
  EtatHeros.repos: idle,
  EtatHeros.marche: walk,
  EtatHeros.saut: jump,
  EtatHeros.attaque: attack,
  EtatHeros.blesse: hurt,
  EtatHeros.mort: die,
};

SpriteAnimation get animationCourante => animations[etat]!;
```

Le `!` est ici légitime (chapitre 12) : la `Map` contient une entrée pour **chaque** valeur de l'énumération. On peut d'ailleurs le garantir mécaniquement :

```dart
assert(
  EtatHeros.values.every(animations.containsKey),
  'Il manque une animation pour au moins un état',
);
```

`every` vient du chapitre 14. Cet `assert` vous évitera un crash le jour où vous ajouterez un septième état sans penser à son animation.

---

## 22.26 — Changer d'animation sans casser la frame courante

Il reste un détail, invisible sur le papier mais très visible à l'écran.

Que se passe-t-il si vous exécutez, à chaque frame :

```dart
// MAUVAIS
etat = nouvelEtat;
animationCourante.reinitialiser();
```

Tant que l'état ne change pas, vous réinitialisez l'animation **soixante fois par seconde**. Elle reste donc bloquée sur sa première frame : votre héros marche sans bouger les jambes.

### La règle

> On ne réinitialise une animation **que** lorsque l'état change **réellement**.

```dart
void _passerA(EtatHeros nouveau) {
  if (etat == nouveau) return; // rien à faire : c'est le point clé
  etat = nouveau;
  animations[nouveau]!.reinitialiser();
}
```

Cette unique ligne `if (etat == nouveau) return;` est la différence entre une animation qui fonctionne et une animation figée. C'est le bug le plus courant de toute cette partie du cours.

### Faut-il toujours réinitialiser ?

Non, et c'est une question de goût qui a des conséquences visuelles.

| Choix | Comportement | Quand l'utiliser |
| --- | --- | --- |
| réinitialiser | l'animation repart de sa frame 0 | attaque, mort, dégât : le geste doit être complet |
| conserver la position | on reprend au même rang dans la nouvelle séquence | marche ↔ course : la foulée reste continue |

Pour un enchaînement marche vers course, la continuité est plus jolie :

```dart
void passerAvecContinuite(EtatHeros nouveau) {
  if (etat == nouveau) return;
  final int position = animationCourante.positionCourante;
  etat = nouveau;
  animationCourante.reglerPosition(position);
}
```

Cela suppose d'ajouter une méthode à `SpriteAnimation` :

```dart
/// Positionne l'animation sur le rang [position] de sa séquence.
/// Le modulo évite tout dépassement si la nouvelle séquence est plus courte.
void reglerPosition(int position) {
  _index = position % frames.length;
  _accumulateur = 0;
  _terminee = false;
}
```

Le modulo est indispensable : passer d'une animation de 8 frames à une animation de 4 frames avec `_index = 6` provoquerait un `RangeError`.

### Le cas de l'attaque prioritaire

Dernier point : une attaque doit se terminer avant tout autre changement. On le garantit dans `choisirEtat` (section 22.25), pas dans `_passerA`. Séparer ces deux responsabilités — « quel état dois-je viser ? » d'un côté, « comment j'y passe ? » de l'autre — rend le code beaucoup plus facile à faire évoluer.

---

## 22.27 — Padding et artefacts de bord (texture bleeding)

Voici un bug qui n'apparaît que sur certaines machines, à certaines échelles, et qui rend fou parce qu'il semble aléatoire.

### Le symptôme

Une fine ligne de pixels étrangers apparaît sur un bord de votre sprite : un liseré de la frame voisine, ou une ligne transparente.

```text
  Ce que vous attendez        Ce que vous obtenez

  ┌────────────┐              ┌────────────┐
  │            │              │▏           │  ← une colonne de pixels
  │   héros    │              │▏  héros    │    venus de la frame d'à côté
  │            │              │▏           │
  └────────────┘              └────────────┘
```

### La cause

Le processeur graphique travaille en nombres à virgule flottante. Quand votre `Rect` source vaut `Rect.fromLTWH(32, 0, 32, 32)` et que l'échelle produit un facteur non entier, l'échantillonnage peut tomber pile sur la frontière entre deux frames. Le processeur, hésitant, va chercher un demi-pixel dans la case voisine.

C'est ce qu'on appelle le **texture bleeding** (« bavure de texture »).

### Les trois remèdes

**Remède 1 : rétrécir légèrement le rectangle source.** C'est la correction la plus rapide, à appliquer côté code :

```dart
final Rect src = planche.rectDe(index).deflate(0.5);
```

`deflate(0.5)` rétrécit le rectangle d'un demi-pixel sur ses quatre côtés. On perd un demi-pixel du sprite, invisible à l'œil, et la bavure disparaît.

**Remède 2 : `FilterQuality.none`.** Le plus proche voisin n'interpole pas : il ne va donc pas chercher le pixel d'à côté. C'est une raison de plus de l'utiliser systématiquement en pixel art.

**Remède 3 : le padding dans la planche.** C'est la vraie solution, côté image. On sépare les frames par 1 ou 2 pixels transparents.

```text
  SANS padding (frames collées)      AVEC padding de 2 px

  ┌────┬────┬────┐                   ┌────┐ ┌────┐ ┌────┐
  │ f0 │ f1 │ f2 │                   │ f0 │ │ f1 │ │ f2 │
  └────┴────┴────┘                   └────┘ └────┘ └────┘
   risque de bavure                   ░ = 2 px transparents
```

Le calcul du `Rect` change alors :

```dart
Rect rectAvecPadding(int index, int colonnes, double lf, double hf, double pad) {
  final int ligne = index ~/ colonnes;
  final int colonne = index % colonnes;
  return Rect.fromLTWH(
    pad + colonne * (lf + pad),
    pad + ligne * (hf + pad),
    lf,
    hf,
  );
}
```

Le `pad +` initial tient compte de la marge extérieure de la planche.

**L'extrusion** est une variante plus poussée, utilisée dans les tilemaps (chapitre 25) : au lieu de séparer par du transparent, on duplique le pixel de bord vers l'extérieur. Ainsi, même si le processeur déborde, il lit une copie de la bonne couleur.

### Le tableau de diagnostic

| Symptôme | Cause probable | Correction |
| --- | --- | --- |
| liseré de la frame voisine | échantillonnage sur la frontière | `deflate(0.5)` ou padding |
| liseré transparent au bord | `Rect` destination non aligné sur des pixels entiers | arrondir les coordonnées destination |
| sprite flou en plus du liseré | filtre d'interpolation actif | `FilterQuality.none` |
| bavure uniquement en mouvement | positions en `double` non arrondies | `x.roundToDouble()` avant de dessiner |

Cette dernière ligne mérite un mot : en pixel art, on arrondit souvent la position de dessin à l'entier le plus proche.

```dart
canvas.drawImageRect(image, src, Rect.fromLTWH(
  x.roundToDouble(), y.roundToDouble(), l, h), paint);
```

La logique de jeu, elle, continue à travailler en `double` : seul l'affichage est arrondi. On garde ainsi la précision du mouvement et la netteté du rendu.

---

## 22.28 — Atlas et fichiers de description

Une sprite sheet en grille régulière est simple à découper, mais elle gaspille de la place dès que les sprites ont des tailles différentes.

Un **atlas** (ou *texture atlas*) est une planche où les images sont rangées **de manière compacte**, sans grille, chacune à la position et à la taille que le logiciel de packing a trouvées.

```text
  Sprite sheet régulière            Atlas compacté

  ┌─────┬─────┬─────┐               ┌──────────┬─────┬───┐
  │héros│ ··· │ ··· │               │  héros   │ pot │clé│
  ├─────┼─────┼─────┤               │          ├─────┴───┤
  │potion│ ··· │ ··· │              ├──────────┤ gobelin │
  ├─────┼─────┼─────┤               │  coffre  │         │
  │gobelin│ ··· │ ··· │             └──────────┴─────────┘
  └─────┴─────┴─────┘
   beaucoup de vide                  presque aucun vide
```

Comme les positions ne se calculent plus, il faut les **décrire**. C'est le rôle du **fichier de description**, en général un JSON portant le même nom que l'image.

```text
  assets/images/
  ├── donjon.png      ← l'atlas
  └── donjon.json     ← la description
```

Exemple de description, au format produit par la plupart des outils :

```text
{
  "frames": {
    "heros_idle_0": { "frame": { "x": 0,  "y": 0,  "w": 32, "h": 32 } },
    "heros_idle_1": { "frame": { "x": 32, "y": 0,  "w": 32, "h": 32 } },
    "potion":       { "frame": { "x": 64, "y": 0,  "w": 16, "h": 16 } },
    "cle":          { "frame": { "x": 80, "y": 0,  "w": 16, "h": 12 } },
    "gobelin_0":    { "frame": { "x": 0,  "y": 32, "w": 24, "h": 28 } }
  },
  "meta": { "image": "donjon.png", "size": { "w": 128, "h": 96 } }
}
```

Vous savez déjà lire ce fichier : c'est exactement le JSON du chapitre 17.

```dart
class Atlas {
  Atlas(this.image, this.regions);

  final ui.Image image;
  final Map<String, Rect> regions;

  static Future<Atlas> charger(String cheminImage, String cheminJson) async {
    final ui.Image image = await chargerImage(cheminImage);
    final String texte = await rootBundle.loadString(cheminJson);
    final Map<String, dynamic> racine =
        jsonDecode(texte) as Map<String, dynamic>;
    final Map<String, dynamic> frames =
        racine['frames'] as Map<String, dynamic>;

    final Map<String, Rect> regions = <String, Rect>{};
    frames.forEach((String nom, dynamic valeur) {
      final Map<String, dynamic> f =
          (valeur as Map<String, dynamic>)['frame'] as Map<String, dynamic>;
      regions[nom] = Rect.fromLTWH(
        (f['x'] as num).toDouble(),
        (f['y'] as num).toDouble(),
        (f['w'] as num).toDouble(),
        (f['h'] as num).toDouble(),
      );
    });
    return Atlas(image, regions);
  }

  Rect operator [](String nom) {
    final Rect? r = regions[nom];
    if (r == null) {
      throw ArgumentError('Région "$nom" absente de l\'atlas');
    }
    return r;
  }
}
```

Notez `rootBundle.loadString` pour un fichier texte, là où l'image utilisait `rootBundle.load`. Le JSON doit lui aussi être déclaré dans `pubspec.yaml`.

Notez également le `(f['x'] as num).toDouble()`. Un JSON peut contenir `32` (un `int`) ou `32.0` (un `double`). Passer par `num` puis `toDouble()` traite les deux cas. C'est un réflexe du chapitre 17 qui évite une exception de cast bien désagréable.

### Faut-il un atlas dès maintenant ?

Non. Pour votre premier jeu, une planche en grille régulière est plus simple, plus lisible et amplement suffisante. Retenez seulement que l'atlas existe, que c'est ce que produisent les outils de packing, et que vous savez le lire le jour où vous en croiserez un.

---

## 22.29 — Solution de repli sans image : générer ses frames en code

C'est la section qui rend tout ce chapitre exécutable sans un seul téléchargement. Nous en avons utilisé le principe depuis 22.8 bis ; formalisons-le.

### Le principe

```text
  ┌─────────────────────────────────────────────────────────┐
  │  Pour chaque frame :                                     │
  │    1. placer l'origine dans la case (translate)          │
  │    2. dessiner le personnage dans sa pose n° i           │
  │  Puis convertir toute la planche en un ui.Image          │
  └─────────────────────────────────────────────────────────┘
```

La seule difficulté est artistique : comment faire varier la pose selon la frame ? Voici quatre recettes qui suffisent à produire des animations lisibles.

### Recette 1 : le décalage vertical (respiration, flottement)

```dart
final double y = (index % 2 == 0) ? 0 : 1;
```

Un pixel de haut en bas, deux frames : le personnage semble respirer. C'est peu, et c'est exactement ce que font beaucoup de jeux professionnels.

### Recette 2 : le balancement (marche)

```dart
// 4 frames : jambes serrées, écartées, serrées, écartées dans l'autre sens
const List<double> ecart = <double>[0, 3, 0, -3];
```

Une table de valeurs indexée par la frame. C'est la technique la plus utile : elle marche pour les jambes, les bras, une queue, une cape.

### Recette 3 : la sinusoïde (mouvements fluides)

```dart
import 'dart:math' as math;

final double angle = 2 * math.pi * index / nombreFrames;
final double hauteur = math.sin(angle) * 3; // oscillation de -3 à +3
```

Une sinusoïde donne un cycle parfaitement continu : la dernière frame enchaîne naturellement avec la première. Idéal pour une pièce qui flotte, une torche, une potion qui pulse.

### Recette 4 : la progression (attaque, mort)

```dart
final double avancement = index / (nombreFrames - 1); // de 0.0 à 1.0
final double longueurLame = 12 * avancement;
```

Pour une animation non bouclée, on interpole linéairement du début vers la fin.

### Un générateur complet de planche

```dart
/// Fabrique une planche de [colonnes] x [lignes] frames de [taille] px,
/// en appelant [dessinFrame] une fois par case.
Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}
```

Le `clipRect` est une sécurité précieuse : il empêche qu'un dessin qui déborde de sa case vienne polluer la case voisine. Sans lui, une épée trop longue apparaîtrait dans la frame suivante — un texture bleeding que vous auriez créé vous-même.

### Les avantages de cette approche pédagogique

| Avantage | Détail |
| --- | --- |
| Rien à télécharger | tout tourne immédiatement |
| Rien à déclarer | pas de `pubspec.yaml` à modifier |
| Modification instantanée | changer une couleur, c'est changer un `Color` |
| Prototypage | on valide la mécanique de jeu avant de commander les graphismes |
| Transition indolore | remplacer `genererPlanche()` par `chargerImage()` : une ligne |

### Sa limite

Un `PictureRecorder` produit une image en mémoire, à chaque lancement. Pour dix planches, cela reste négligeable. Pour un vrai jeu avec cinquante personnages, chargez de vrais PNG : c'est plus rapide et plus joli.

Cette solution de repli est un **échafaudage**, pas une architecture définitive. Mais tant que l'échafaudage tient, vous pouvez construire.

---

## 22.30 — Mini-projet : un personnage animé qui marche et s'arrête

Rassemblons tout. Objectif : un héros du Donjon de Dart qui traverse l'écran, se retourne aux bords, et que l'on peut arrêter et relancer d'une simple pression.

### Cahier des charges

| Exigence | Moyen |
| --- | --- |
| Aucun fichier image | planche générée par `PictureRecorder` (22.29) |
| Deux animations | `idle` (ligne 0) et `walk` (ligne 1) |
| Vitesse indépendante du framerate | accumulateur de `dt` (22.21) et déplacement en px/s |
| Personnage net | `FilterQuality.none` (22.12) |
| Retournement selon la direction | `scale(-1, 1)` (22.13) |
| Changement d'état propre | `_passerA` avec test d'égalité (22.26) |
| Interaction | une pression sur l'écran démarre ou arrête la marche |

### Structure du programme

```text
  main
   └── AppDonjon (StatelessWidget)
        └── EcranJeu (StatefulWidget)
             ├── initState : génère la planche, puis démarre le Ticker
             ├── _boucle(dt) : met à jour le héros
             └── build : GestureDetector + CustomPaint
                            └── PeintreJeu : décor + héros
   Modèle :
     SpriteSheet      (image + géométrie + dessin)
     SpriteAnimation  (frames + cadence + accumulateur)
     Heros            (position, direction, état, animations)
```

### Le programme complet

```dart
import 'dart:math' as math;
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

// ---------------------------------------------------------------------------
// 1. Fabrication de l'image (solution de repli sans fichier)
// ---------------------------------------------------------------------------

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  return enregistreur.endRecording().toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// Une frame du héros, dessinée dans un repère local de 32x32.
/// Ligne 0 = repos (4 frames utiles), ligne 1 = marche (6 frames).
void dessinerHeros(Canvas canvas, int ligne, int colonne) {
  if (ligne == 0 && colonne >= 4) return; // cases inutilisées

  final Paint peau = Paint()..color = const Color(0xFFF2C79B);
  final Paint tunique = Paint()..color = const Color(0xFF3A7BD5);
  final Paint ceinture = Paint()..color = const Color(0xFF5A3A1A);
  final Paint bottes = Paint()..color = const Color(0xFF2A2A38);
  final Paint cheveux = Paint()..color = const Color(0xFF6B4423);

  double souffle = 0;
  double jambeAvant = 0;
  double bras = 0;

  if (ligne == 0) {
    // Repos : le buste monte d'un pixel une frame sur deux.
    souffle = (colonne == 1 || colonne == 2) ? 1 : 0;
  } else {
    // Marche : oscillation sinusoïdale sur 6 frames (cycle continu).
    final double phase = 2 * math.pi * colonne / 6;
    jambeAvant = math.sin(phase) * 3.5;
    bras = -math.sin(phase) * 3.0;
    souffle = (math.cos(phase) > 0.5) ? 1 : 0;
  }

  // Jambes (dessinées avant le corps pour passer dessous).
  canvas.drawRect(Rect.fromLTWH(12 + jambeAvant, 23, 4, 7), bottes);
  canvas.drawRect(Rect.fromLTWH(16 - jambeAvant, 23, 4, 7), bottes);

  // Bras arrière.
  canvas.drawRect(Rect.fromLTWH(8 + bras, 14 + souffle, 3, 8), peau);

  // Corps.
  canvas.drawRect(Rect.fromLTWH(10, 12 + souffle, 12, 11), tunique);
  canvas.drawRect(Rect.fromLTWH(10, 19 + souffle, 12, 2), ceinture);

  // Tête.
  canvas.drawRect(Rect.fromLTWH(11, 3 + souffle, 10, 9), peau);
  canvas.drawRect(Rect.fromLTWH(11, 3 + souffle, 10, 3), cheveux);
  canvas.drawRect(Rect.fromLTWH(18, 7 + souffle, 2, 2), bottes); // œil

  // Bras avant.
  canvas.drawRect(Rect.fromLTWH(21 - bras, 14 + souffle, 3, 8), peau);
}

// ---------------------------------------------------------------------------
// 2. Les outils du chapitre
// ---------------------------------------------------------------------------

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    return Rect.fromLTWH(
      (index % colonnes) * largeurFrame.toDouble(),
      (index ~/ colonnes) * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  List<int> indexLigne(int ligne, {int? combien}) {
    final int debut = ligne * colonnes;
    return List<int>.generate(combien ?? colonnes, (int i) => debut + i);
  }

  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
  }) {
    final Rect src = rectDe(index).deflate(0.5); // anti-bavure (22.27)
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(
      (x - l / 2).roundToDouble(),
      (y - h).roundToDouble(),
      l,
      h,
    );

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, paintPixelArt);
      return;
    }
    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(
        image, src, Rect.fromLTWH(0, 0, l, h), paintPixelArt);
    canvas.restore();
  }

  static final Paint paintPixelArt = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

// ---------------------------------------------------------------------------
// 3. Le héros
// ---------------------------------------------------------------------------

enum EtatHeros { repos, marche }

class Heros {
  Heros(this.planche)
      : _animations = <EtatHeros, SpriteAnimation>{
          EtatHeros.repos: SpriteAnimation(
            frames: planche.indexLigne(0, combien: 4),
            stepTime: 0.22,
          ),
          EtatHeros.marche: SpriteAnimation(
            frames: planche.indexLigne(1, combien: 6),
            stepTime: 0.11,
          ),
        };

  final SpriteSheet planche;
  final Map<EtatHeros, SpriteAnimation> _animations;

  double x = 80;
  double y = 260;          // le sol : le héros a les pieds ici
  double vitesse = 90;     // pixels par seconde
  bool versLaDroite = true;
  bool enMarche = false;
  EtatHeros etat = EtatHeros.repos;

  SpriteAnimation get animation => _animations[etat]!;

  void basculerMarche() {
    enMarche = !enMarche;
  }

  void _passerA(EtatHeros nouveau) {
    if (etat == nouveau) return; // LA ligne qui évite l'animation figée
    etat = nouveau;
    _animations[nouveau]!.reinitialiser();
  }

  void mettreAJour(double dt, double largeurEcran) {
    const double marge = 60;

    if (enMarche) {
      x += (versLaDroite ? vitesse : -vitesse) * dt;
      if (x > largeurEcran - marge) {
        x = largeurEcran - marge;
        versLaDroite = false;
      } else if (x < marge) {
        x = marge;
        versLaDroite = true;
      }
      _passerA(EtatHeros.marche);
    } else {
      _passerA(EtatHeros.repos);
    }

    animation.avancer(dt);
  }

  void dessiner(Canvas canvas) {
    planche.dessiner(
      canvas,
      animation.frameCourante,
      x,
      y,
      echelle: 4,
      retourne: !versLaDroite,
    );
  }
}

// ---------------------------------------------------------------------------
// 4. L'application
// ---------------------------------------------------------------------------

void main() => runApp(const AppDonjon());

class AppDonjon extends StatelessWidget {
  const AppDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(backgroundColor: Color(0xFF1B1B2A), body: EcranJeu()),
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
  Ticker? _ticker;
  Duration _precedent = Duration.zero;
  Heros? _heros;
  Size _taille = const Size(400, 600);

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    final ui.Image image = await genererPlanche(
      colonnes: 6,
      lignes: 2,
      taille: 32,
      dessinFrame: dessinerHeros,
    );
    if (!mounted) return;

    final SpriteSheet planche =
        SpriteSheet(image: image, largeurFrame: 32, hauteurFrame: 32);
    setState(() => _heros = Heros(planche));

    _ticker = createTicker(_boucle)..start();
  }

  void _boucle(Duration elapsed) {
    final double dtBrut =
        (elapsed - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = elapsed;
    if (dtBrut <= 0) return;

    final double dt = dtBrut > 0.05 ? 0.05 : dtBrut; // garde-fou (22.21)
    _heros?.mettreAJour(dt, _taille.width);
    setState(() {});
  }

  @override
  void dispose() {
    _ticker?.dispose();
    _heros?.planche.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final Heros? heros = _heros;
    if (heros == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints contraintes) {
        _taille = Size(contraintes.maxWidth, contraintes.maxHeight);
        return GestureDetector(
          behavior: HitTestBehavior.opaque,
          onTap: () => heros.basculerMarche(),
          child: CustomPaint(
            painter: PeintreJeu(heros),
            size: Size.infinite,
          ),
        );
      },
    );
  }
}

class PeintreJeu extends CustomPainter {
  PeintreJeu(this.heros);

  final Heros heros;

  final Paint _sol = Paint()..color = const Color(0xFF2E2E45);
  final Paint _pierre = Paint()..color = const Color(0xFF3A3A58);

  @override
  void paint(Canvas canvas, Size size) {
    heros.y = size.height * 0.65;

    // Décor : un sol de donjon en dalles.
    canvas.drawRect(
      Rect.fromLTWH(0, heros.y, size.width, size.height - heros.y),
      _sol,
    );
    for (double x = 0; x < size.width; x += 40) {
      canvas.drawRect(Rect.fromLTWH(x + 2, heros.y + 4, 36, 12), _pierre);
    }

    heros.dessiner(canvas);

    // Information à l'écran.
    final TextPainter tp = TextPainter(
      textDirection: TextDirection.ltr,
      text: TextSpan(
        text: 'État : ${heros.etat.name}   '
            'Frame : ${heros.animation.frameCourante}\n'
            'Touchez l\'écran pour marcher ou vous arrêter.',
        style: const TextStyle(color: Color(0xFFCFCFE8), fontSize: 14),
      ),
    )..layout();
    tp.paint(canvas, const Offset(16, 16));
  }

  @override
  bool shouldRepaint(PeintreJeu oldDelegate) => true;
}
```

**Résultat :**

```text
Un donjon sombre avec un sol dallé.
Au démarrage, le héros est immobile : son buste monte et descend d'un pixel,
au rythme lent de l'animation « repos ».
Une pression sur l'écran le met en marche : ses jambes et ses bras oscillent,
et il avance à 90 pixels par seconde.
Arrivé au bord droit, il se retourne (sprite en miroir) et repart vers la gauche.
Une nouvelle pression l'arrête : il repasse en animation « repos », sans saut
visuel, et continue à regarder du côté où il allait.
En haut à gauche, l'état courant et l'index de frame s'affichent en direct.
```

### Ce qu'il faut retenir de ce mini-projet

1. **Le rendu et la logique sont séparés.** `Heros.mettreAJour` ne connaît pas le `Canvas` ; `PeintreJeu` ne décide de rien. C'est l'architecture que le chapitre 26 généralisera.
2. **Le `dt` circule partout.** Déplacement et animation utilisent le même `dt`. Sur un écran 120 Hz, le héros marchera à la même vitesse et ses jambes bougeront au même rythme.
3. **Un seul `_passerA`.** Toute la protection contre l'animation figée tient dans une ligne.
4. **Zéro fichier image.** Remplacez `genererPlanche(...)` par `chargerImage('assets/images/heros.png')` et le reste du programme est inchangé.

---

## 22.31 — Erreurs fréquentes

Ce tableau rassemble les erreurs que vous rencontrerez réellement. Relisez-le chaque fois qu'un sprite refuse de s'afficher ou qu'une animation reste figée.

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: assets/images/heros.png` | le dossier n'est pas déclaré dans `pubspec.yaml`, ou le fichier n'existe pas à ce chemin exact | ajouter la ligne sous `flutter: assets:`, vérifier le chemin **caractère par caractère**, puis relancer `flutter run` (pas seulement un hot reload) |
| L'asset est bien déclaré, mais l'erreur persiste | `pubspec.yaml` mal indenté : `assets:` doit être aligné sous `flutter:` avec deux espaces, et chaque chemin avec quatre espaces et un tiret | ré-indenter en **espaces uniquement** (jamais de tabulation), puis `flutter pub get` |
| `LateInitializationError: Field '_planche' has not been initialized` | on dessine avant la fin du chargement asynchrone | déclarer `ui.Image? _planche;` (chapitre 12) et afficher un écran d'attente tant que la valeur est `null` |
| Écran noir au démarrage, puis le sprite apparaît | le `Future` de chargement n'est pas attendu avant de lancer la boucle de jeu | tout précharger dans un `Future.wait` (chapitre 15), et ne démarrer le `Ticker` qu'après |
| `setState() called after dispose()` | l'image finit de charger après la fermeture de l'écran | tester `if (!mounted) return;` juste après le `await` |
| Le pixel art est flou, baveux | interpolation bilinéaire par défaut à l'agrandissement | `Paint()..filterQuality = FilterQuality.none` et `..isAntiAlias = false` |
| Le sprite est net mais déformé (écrasé ou étiré) | le rectangle de destination n'a pas le même rapport largeur/hauteur que la source | calculer la destination à partir d'une seule échelle : `l = largeurFrame * echelle`, `h = hauteurFrame * echelle` |
| L'animation va deux fois plus vite sur un écran 120 Hz | l'index de frame avance d'une unité **par frame de rendu** au lieu d'avancer avec le temps | accumuler le `dt` et comparer à `stepTime` (chapitre 20) |
| Le personnage marche mais ses jambes ne bougent pas | `reinitialiser()` est appelé à chaque frame parce que l'état est réaffecté sans test | dans `_passerA`, commencer par `if (etat == nouveau) return;` |
| `RangeError (index): Invalid value: Not in inclusive range 0..11: 12` | index de frame calculé au-delà de la planche, souvent après avoir changé le nombre de colonnes | utiliser `SpriteSheet.rectDe` qui contrôle les bornes, et un `%` sur l'index d'animation |
| Une fine ligne de la frame voisine apparaît sur le bord du sprite | texture bleeding : le rectangle source tombe entre deux pixels après mise à l'échelle | `FilterQuality.none`, coordonnées source entières, et un padding d'un pixel dans la planche |
| L'image est retournée mais elle a disparu de l'écran | `canvas.scale(-1, 1)` sans `translate` préalable : le dessin part dans les x négatifs | `save()`, `translate(dst.right, dst.top)`, `scale(-1, 1)`, dessiner en `(0, 0)`, `restore()` |
| Les transformations s'accumulent et tout part de travers | un `save()` sans `restore()` correspondant | toujours écrire les deux dans la même méthode, `restore()` en dernier |
| Le fond du sprite est un carré blanc ou noir | l'image est un JPEG : ce format n'a pas de canal alpha | réexporter en PNG 32 bits avec transparence |
| La planche est coupée : la dernière colonne manque | `image.width ~/ largeurFrame` a tronqué à cause d'une marge ou d'un séparateur | tenir compte de `marge` et `espacement` dans le calcul du `Rect` source (exercice 9) |
| L'animation de mort recommence en boucle | `loop` laissé à `true` | passer `loop: false` et lire `estTerminee` pour déclencher la suite |
| L'attaque est interrompue dès qu'on bouge | la transition vers `marche` est testée avant la fin de l'animation d'attaque | ordonner les tests du plus prioritaire au moins prioritaire dans `choisirEtat` |
| Le jeu rame après quelques minutes | des `ui.Image` créés à chaque frame par `PictureRecorder` et jamais libérés | générer les planches **une seule fois** au chargement, et appeler `dispose()` sur les images dans `dispose()` |
| `Concurrent modification during iteration` en changeant d'arme | la liste des entités est modifiée pendant le `for` du rendu | collecter les changements et les appliquer après la boucle (chapitre 6) |
| Le sprite « tremble » d'un pixel en marchant | la position en `double` est envoyée telle quelle au `Rect` de destination | arrondir la position de destination (`x.roundToDouble()`) pour un rendu pixel-perfect |

---

## 22.32 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Sprite | une image 2D dessinée telle quelle à l'écran, avec un fond transparent |
| Pixel art | peu de pixels, agrandis à l'affichage ; impose `FilterQuality.none` |
| PNG | seul format raisonnable pour un sprite : compression sans perte **et** canal alpha |
| JPEG | à proscrire : pas de transparence, artefacts autour des contours |
| Licence | CC0 = libre sans condition ; CC-BY = attribution obligatoire ; toujours lire avant d'intégrer |
| `pubspec.yaml` | déclarer `assets:` sous `flutter:`, en espaces, deux puis quatre |
| Chargement | `rootBundle.load(chemin)` puis `decodeImageFromList(bytes)` |
| Asynchronisme | le chargement renvoie un `Future` : rien ne peut être dessiné avant sa résolution |
| Préchargement | `Future.wait` sur toutes les images, puis démarrage du `Ticker` |
| `drawImage` | dessine l'image entière à un `Offset` donné, en taille réelle |
| `drawImageRect` | dessine une **portion** (`src`) dans un **rectangle** (`dst`) : c'est la méthode du jeu |
| Redimensionner | changer `dst` seulement, en gardant le rapport largeur/hauteur de `src` |
| `FilterQuality.none` | garde les carrés nets à l'agrandissement ; indispensable en pixel art |
| Miroir | `save()`, `translate(dst.right, dst.top)`, `scale(-1, 1)`, dessiner en `(0, 0)`, `restore()` |
| Sprite sheet | une seule image contenant une grille de frames de taille identique |
| Pourquoi une planche | un seul chargement, une seule texture, moins de changements d'état GPU |
| Index vers grille | `ligne = index ~/ colonnes`, `colonne = index % colonnes` |
| `Rect` source | `Rect.fromLTWH(colonne * l, ligne * h, l, h)` |
| `SpriteSheet` | classe maison : image, géométrie, `rectDe`, `indexLigne`, `dessiner` |
| Animation | une liste d'index de frames parcourue au fil du temps |
| `stepTime` | durée d'affichage d'une frame, en secondes ; `fps = 1 / stepTime` |
| Avancer | accumuler le `dt`, et tant que l'accumulateur dépasse `stepTime`, passer à la frame suivante |
| `loop` | `true` pour repos, marche, course ; `false` pour attaque, dégât, mort |
| `estTerminee` | signal de fin d'une animation non bouclée : sert à enchaîner un état |
| États | un `enum` (chapitre 11) plus une `Map` état vers animation (chapitre 6) |
| `_passerA` | `if (etat == nouveau) return;` avant toute réinitialisation : la ligne la plus importante du chapitre |
| Padding | un pixel de marge autour de chaque frame évite le texture bleeding |
| Atlas | planche à frames de tailles variées, accompagnée d'un fichier de description JSON (chapitre 17) |
| Repli sans image | `PictureRecorder` plus `Canvas` plus `picture.toImage()` produit un `ui.Image` indiscernable d'un PNG |
| Migration | remplacer `genererPlanche(...)` par `chargerImage(...)` : le reste du code ne change pas |

---

## 22.33 — Exercices

Les dix exercices sont classés du plus simple au plus complet. **Aucun ne nécessite de télécharger la moindre image :** toutes les frames sont fabriquées en code, selon la méthode de la section 22.29.

Les exercices 1 et 2 sont du Dart pur : ils s'exécutent dans DartPad en mode Dart. Les exercices 3 à 10 sont des applications Flutter complètes.

Travaillez d'abord seul. Les corrections viennent ensuite, en code intégral.

---

### Exercice 1 — La géométrie d'une planche (facile)

Sans Flutter, écrivez une classe `GeometriePlanche` qui décrit une planche de sprites : nombre de colonnes, nombre de lignes, largeur et hauteur d'une frame.

Elle doit fournir :

- un getter `nombreFrames` ;
- `ligneDe(int index)` et `colonneDe(int index)` ;
- `gaucheDe(int index)` et `hautDe(int index)`, les coordonnées du coin haut-gauche de la frame dans la planche ;
- `indexDe(int ligne, int colonne)`, l'opération inverse ;
- `decrire(int index)`, qui renvoie une ligne de texte lisible, et qui lève une `RangeError` si l'index sort de la planche (chapitre 13).

Dans `main`, décrivez les douze frames d'une planche 4 colonnes sur 3 lignes en frames de 32 sur 32, vérifiez l'aller-retour index vers grille, puis provoquez volontairement l'erreur d'index et attrapez-la.

---

### Exercice 2 — Une animation qui ne dépend pas du framerate (facile)

Toujours en Dart pur. Écrivez la classe `SpriteAnimation` de la section 22.22 (`frames`, `stepTime`, `loop`, `avancer(dt)`, `frameCourante`, `estTerminee`, `reinitialiser()`).

Créez deux animations sur les mêmes frames `[0, 1, 2, 3]` avec `stepTime` à `0.1` : l'une bouclée, l'autre non.

Simulez douze pas de `dt = 0.05` seconde et affichez à chaque pas le temps écoulé, la frame de chaque animation et l'état `estTerminee` de la seconde.

Vérifiez sur la sortie que la frame change bien toutes les deux itérations, que la bouclée revient à `0` et que la non bouclée se fige sur `3`.

---

### Exercice 3 — Une pièce d'or fabriquée en code (facile)

Application Flutter. Sans aucun fichier, fabriquez avec `PictureRecorder` une image de 16 sur 16 pixels représentant une pièce d'or du Donjon de Dart : un disque doré, un liseré plus sombre, un petit reflet clair.

Affichez cette même image trois fois sur le `Canvas` :

- en taille réelle (16 sur 16) ;
- agrandie 6 fois avec `FilterQuality.none` ;
- agrandie 6 fois avec `FilterQuality.high`.

Écrivez sous chaque version un libellé avec `TextPainter`, et concluez sur la différence visuelle.

---

### Exercice 4 — Une planche de rotation de pièce (moyen)

Fabriquez une planche de 4 colonnes sur 1 ligne, en frames de 16 sur 16, représentant la pièce vue sous quatre angles : large, moyenne, de profil (presque un trait), moyenne.

Affichez les quatre frames côte à côte, agrandies 6 fois, en utilisant `drawImageRect` avec un rectangle source calculé par la formule de la section 22.16. Numérotez chaque frame.

Vérifiez que vous voyez bien quatre pièces distinctes et non quatre fois la même.

---

### Exercice 5 — Animer la pièce (moyen)

Reprenez la planche de l'exercice 4. Ajoutez un `Ticker` (chapitre 20) et la classe `SpriteAnimation`.

Faites tourner la pièce au centre de l'écran avec un `stepTime` de `0.12` seconde, agrandie 8 fois.

Affichez en haut de l'écran le `dt` de la frame courante, les FPS et l'index de frame affiché, afin de vérifier de vos yeux que l'index avance au rythme du temps et non au rythme du rendu.

---

### Exercice 6 — Un coffre qui s'ouvre une seule fois (moyen)

Fabriquez une planche de 5 frames de 24 sur 24 : coffre fermé, entrouvert, à demi ouvert, grand ouvert, grand ouvert avec une lueur.

Animez-la avec `loop: false` et un `stepTime` de `0.15` seconde.

Une pression sur l'écran réinitialise l'animation et rejoue l'ouverture. Tant que l'animation n'est pas terminée, une nouvelle pression est ignorée. Affichez l'état à l'écran : « ouverture en cours » ou « coffre ouvert, touchez pour refermer ».

---

### Exercice 7 — Un gobelin qui fait des allers-retours (moyen)

Fabriquez une planche de 4 frames de 24 sur 24 pour la marche d'un gobelin (recette 2 de la section 22.29 : une table de valeurs pour l'écartement des jambes).

Le gobelin se déplace à 80 pixels par seconde. Arrivé à 40 pixels d'un bord, il change de sens.

Quand il va vers la gauche, le sprite est retourné avec `scale(-1, 1)`. Assurez-vous que la position à l'écran ne saute pas au moment du retournement.

---

### Exercice 8 — Une machine à états à trois états (difficile)

Reprenez le gobelin et donnez-lui trois états dans un `enum` : `repos`, `marche`, `attaque`.

Règles :

- l'attaque dure une animation complète et **ne peut pas être interrompue** ;
- une pression sur l'écran déclenche l'attaque ;
- entre deux attaques, le gobelin alterne repos et marche selon sa vitesse ;
- la vitesse passe de `0` à `80` toutes les deux secondes, automatiquement.

Vous devez implémenter `choisirEtat()` (tests du plus prioritaire au moins prioritaire) et `_passerA()` (avec le test d'égalité qui évite l'animation figée). Affichez l'état courant et l'index de frame à l'écran.

---

### Exercice 9 — Une planche avec marge et espacement (difficile)

Beaucoup de planches téléchargées ont un pixel de marge sur le pourtour et un pixel de séparation entre les cases (section 22.27).

Écrivez une classe `SpriteSheetPad` dont le constructeur prend `marge` et `espacement` en plus de la taille de frame, et dont `rectDe(index)` applique la formule complète :

```text
x = marge + colonne * (largeurFrame + espacement)
y = marge + ligne   * (hauteurFrame + espacement)
```

Le nombre de colonnes et de lignes doit être **calculé** à partir de la taille de l'image, pas passé en paramètre.

Générez une planche 4 sur 2 avec `marge: 2` et `espacement: 2`, en peignant volontairement les zones de séparation en rouge vif. Affichez les huit frames agrandies 5 fois : si votre formule est juste, **aucun rouge ne doit apparaître**.

---

### Exercice 10 — Une scène du Donjon de Dart (difficile)

Assemblez tout dans une seule application.

Une planche unique de 4 colonnes sur 3 lignes, en frames de 24 sur 24 :

- ligne 0 : le héros au repos (4 frames) ;
- ligne 1 : le héros en marche (4 frames) ;
- ligne 2 : la potion qui pulse (4 frames).

La scène contient un sol, le héros qui marche de gauche à droite en se retournant aux bords, et trois potions posées au sol qui pulsent, chacune avec un décalage de phase différent afin qu'elles ne battent pas à l'unisson.

Un appui sur l'écran arrête ou relance le héros. Un panneau d'information affiche les FPS, l'état du héros et l'index de frame de chaque potion.

Contrainte d'architecture : la logique (`mettreAJour(dt)`) et le rendu (`dessiner(canvas)`) doivent être séparés, comme au chapitre 21.

---

## 22.34 — Corrections des exercices

---

### Correction 1

```dart
/// Décrit la géométrie d'une planche de sprites, sans aucune image.
class GeometriePlanche {
  const GeometriePlanche({
    required this.colonnes,
    required this.lignes,
    required this.largeurFrame,
    required this.hauteurFrame,
  });

  final int colonnes;
  final int lignes;
  final int largeurFrame;
  final int hauteurFrame;

  /// Nombre total de cases de la grille.
  int get nombreFrames => colonnes * lignes;

  /// Largeur totale de la planche, en pixels.
  int get largeurPlanche => colonnes * largeurFrame;

  /// Hauteur totale de la planche, en pixels.
  int get hauteurPlanche => lignes * hauteurFrame;

  int ligneDe(int index) => index ~/ colonnes;
  int colonneDe(int index) => index % colonnes;

  int gaucheDe(int index) => colonneDe(index) * largeurFrame;
  int hautDe(int index) => ligneDe(index) * hauteurFrame;

  /// Opération inverse : de (ligne, colonne) vers l'index.
  int indexDe(int ligne, int colonne) => ligne * colonnes + colonne;

  String decrire(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    return 'index $index -> ligne ${ligneDe(index)}, '
        'colonne ${colonneDe(index)} -> '
        'Rect(${gaucheDe(index)}, ${hautDe(index)}, '
        '$largeurFrame, $hauteurFrame)';
  }
}

void main() {
  const GeometriePlanche planche = GeometriePlanche(
    colonnes: 4,
    lignes: 3,
    largeurFrame: 32,
    hauteurFrame: 32,
  );

  print('Planche : ${planche.colonnes} x ${planche.lignes} '
      '= ${planche.nombreFrames} frames');
  print('Taille  : ${planche.largeurPlanche} x ${planche.hauteurPlanche} px');
  print('');

  for (int i = 0; i < planche.nombreFrames; i++) {
    print(planche.decrire(i));
  }

  print('');
  print('Aller-retour : indexDe(2, 3) = ${planche.indexDe(2, 3)}');
  print('Aller-retour : ligneDe(11) = ${planche.ligneDe(11)}, '
      'colonneDe(11) = ${planche.colonneDe(11)}');

  print('');
  try {
    print(planche.decrire(12));
  } on RangeError catch (e) {
    print('Erreur attendue : $e');
  }
}
```

**Résultat :**

```text
Planche : 4 x 3 = 12 frames
Taille  : 128 x 96 px

index 0 -> ligne 0, colonne 0 -> Rect(0, 0, 32, 32)
index 1 -> ligne 0, colonne 1 -> Rect(32, 0, 32, 32)
index 2 -> ligne 0, colonne 2 -> Rect(64, 0, 32, 32)
index 3 -> ligne 0, colonne 3 -> Rect(96, 0, 32, 32)
index 4 -> ligne 1, colonne 0 -> Rect(0, 32, 32, 32)
index 5 -> ligne 1, colonne 1 -> Rect(32, 32, 32, 32)
index 6 -> ligne 1, colonne 2 -> Rect(64, 32, 32, 32)
index 7 -> ligne 1, colonne 3 -> Rect(96, 32, 32, 32)
index 8 -> ligne 2, colonne 0 -> Rect(0, 64, 32, 32)
index 9 -> ligne 2, colonne 1 -> Rect(32, 64, 32, 32)
index 10 -> ligne 2, colonne 2 -> Rect(64, 64, 32, 32)
index 11 -> ligne 2, colonne 3 -> Rect(96, 64, 32, 32)

Aller-retour : indexDe(2, 3) = 11
Aller-retour : ligneDe(11) = 2, colonneDe(11) = 3

Erreur attendue : RangeError (index): Invalid value: Not in inclusive range 0..11: 12
```

**Explication :** tout le découpage d'une sprite sheet tient dans deux opérations vues au chapitre 3 : la division entière `~/` donne la ligne, le modulo `%` donne la colonne. Le reste n'est qu'une multiplication par la taille de frame. La classe est `const` et tous ses champs sont `final` (chapitre 10) : une géométrie de planche ne change jamais en cours de partie, autant l'interdire au compilateur. Le contrôle de bornes de `decrire` transforme un bug silencieux — un `Rect` source hors de l'image, qui donne un sprite vide sans message — en une exception explicite (chapitre 13). Remarquez enfin que `indexDe` et le couple `ligneDe` / `colonneDe` sont exactement inverses l'un de l'autre : c'est la vérification la plus simple pour s'assurer que la formule est juste.

---

### Correction 2

```dart
/// Une animation : une liste d'index de frames parcourue au fil du temps.
class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  int get positionCourante => _index;
  bool get estTerminee => _terminee;
  double get duree => frames.length * stepTime;

  void avancer(double dt) {
    if (_terminee) return;

    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;

      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

void main() {
  final SpriteAnimation bouclee = SpriteAnimation(
    frames: <int>[0, 1, 2, 3],
    stepTime: 0.1,
  );
  final SpriteAnimation unique = SpriteAnimation(
    frames: <int>[0, 1, 2, 3],
    stepTime: 0.1,
    loop: false,
  );

  print('Durée d\'un cycle : ${bouclee.duree} s');
  print('');

  const double dt = 0.05;
  double temps = 0;

  for (int i = 0; i < 12; i++) {
    bouclee.avancer(dt);
    unique.avancer(dt);
    temps += dt;

    print('t=${temps.toStringAsFixed(2)}s  '
        'bouclee=${bouclee.frameCourante}  '
        'unique=${unique.frameCourante}  '
        'terminee=${unique.estTerminee}');
  }
}
```

**Résultat :**

```text
Durée d'un cycle : 0.4 s

t=0.05s  bouclee=0  unique=0  terminee=false
t=0.10s  bouclee=1  unique=1  terminee=false
t=0.15s  bouclee=1  unique=1  terminee=false
t=0.20s  bouclee=2  unique=2  terminee=false
t=0.25s  bouclee=2  unique=2  terminee=false
t=0.30s  bouclee=3  unique=3  terminee=false
t=0.35s  bouclee=3  unique=3  terminee=false
t=0.40s  bouclee=0  unique=3  terminee=true
t=0.45s  bouclee=0  unique=3  terminee=true
t=0.50s  bouclee=1  unique=3  terminee=true
t=0.55s  bouclee=1  unique=3  terminee=true
t=0.60s  bouclee=2  unique=3  terminee=true
```

**Explication :** avec `dt = 0.05` et `stepTime = 0.1`, il faut exactement deux pas pour franchir un cran : la sortie montre bien chaque frame affichée deux fois de suite. À `t = 0.40 s`, soit la durée annoncée par `duree`, les deux animations divergent : la bouclée revient à `0`, la non bouclée se fige sur sa dernière frame et lève son drapeau `estTerminee`. C'est ce drapeau qui servira, dans les exercices suivants, à enchaîner « fin de l'attaque » vers « retour au repos ».

Le point décisif est le `while` plutôt qu'un `if`. Si une frame de rendu dure anormalement longtemps — chargement, retour de veille — l'accumulateur peut dépasser plusieurs fois `stepTime`. Le `while` rattrape alors le retard d'un coup et l'animation reste synchronisée avec l'horloge, ce qu'un `if` ne ferait pas. C'est exactement le raisonnement du chapitre 20 sur le delta time, appliqué non plus à une position mais à un index.

---

### Correction 3

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

/// Fabrique un ui.Image de [largeur] x [hauteur] pixels
/// en exécutant [dessin] sur un Canvas enregistré.
Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

/// Une pièce d'or du Donjon de Dart, 16 x 16 pixels.
Future<ui.Image> creerPiece() {
  return imageDepuisDessin(16, 16, (Canvas canvas) {
    final Paint bord = Paint()
      ..color = const Color(0xFF8A5A00)
      ..isAntiAlias = false;
    final Paint or = Paint()
      ..color = const Color(0xFFFFC42E)
      ..isAntiAlias = false;
    final Paint reflet = Paint()
      ..color = const Color(0xFFFFF3B0)
      ..isAntiAlias = false;

    canvas.drawCircle(const Offset(8, 8), 7.5, bord);
    canvas.drawCircle(const Offset(8, 8), 6, or);
    canvas.drawCircle(const Offset(6, 6), 2, reflet);
  });
}

void main() {
  runApp(const AppPiece());
}

class AppPiece extends StatelessWidget {
  const AppPiece({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF12121A),
        body: EcranPiece(),
      ),
    );
  }
}

class EcranPiece extends StatefulWidget {
  const EcranPiece({super.key});

  @override
  State<EcranPiece> createState() => _EcranPieceState();
}

class _EcranPieceState extends State<EcranPiece> {
  ui.Image? _piece;

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    final ui.Image image = await creerPiece();
    if (!mounted) return;
    setState(() => _piece = image);
  }

  @override
  void dispose() {
    _piece?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? piece = _piece;
    if (piece == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(
      painter: PeintrePiece(piece),
      size: Size.infinite,
    );
  }
}

class PeintrePiece extends CustomPainter {
  PeintrePiece(this.piece);

  final ui.Image piece;

  static final Paint _net = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;

  static final Paint _lisse = Paint()..filterQuality = FilterQuality.high;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 13),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  @override
  void paint(Canvas canvas, Size size) {
    const Rect src = Rect.fromLTWH(0, 0, 16, 16);

    // 1. Taille réelle : 16 x 16 pixels.
    canvas.drawImage(piece, const Offset(40, 60), _net);
    _texte(canvas, 'taille reelle (16 x 16)', const Offset(40, 84));

    // 2. Agrandie 6 fois, sans interpolation.
    canvas.drawImageRect(
      piece,
      src,
      const Rect.fromLTWH(40, 130, 96, 96),
      _net,
    );
    _texte(canvas, 'x6 : FilterQuality.none', const Offset(40, 234));

    // 3. Agrandie 6 fois, avec interpolation.
    canvas.drawImageRect(
      piece,
      src,
      const Rect.fromLTWH(180, 130, 96, 96),
      _lisse,
    );
    _texte(canvas, 'x6 : FilterQuality.high', const Offset(180, 234));
  }

  @override
  bool shouldRepaint(PeintrePiece oldDelegate) => oldDelegate.piece != piece;
}
```

**Résultat :**

```text
Une fenêtre sombre.
En haut à gauche : une minuscule pièce de 16 pixels de côté, à peine lisible.
En dessous, deux pièces de 96 pixels côte à côte :
  - celle de gauche montre des marches d'escalier franches ;
  - celle de droite a des bords adoucis, presque cotonneux.
Trois libellés blancs accompagnent les trois versions.
```

**Explication :** l'image fabriquée par `PictureRecorder` est un vrai `ui.Image`. Elle se comporte exactement comme un PNG chargé depuis le disque, et tout ce qui suit dans le chapitre s'y applique sans changement.

Trois détails méritent attention. D'abord `isAntiAlias = false` **à la fabrication** : sans lui, `drawCircle` produirait des pixels de bord semi-transparents, et l'agrandissement les révélerait comme un halo sale. Ensuite `FilterQuality.none` **à l'affichage** : c'est ce qui donne l'escalier net, caractéristique du pixel art assumé. Enfin la comparaison côte à côte : le rendu lissé n'est pas objectivement mauvais, il est simplement inadapté au pixel art. Pour une illustration haute résolution réduite à l'écran, `FilterQuality.medium` serait au contraire le bon choix.

Notez aussi `_piece?.dispose()` dans `dispose()`. Une `ui.Image` occupe de la mémoire graphique que le ramasse-miettes de Dart ne libère pas seul : oublier cet appel, sur un jeu qui crée beaucoup d'images, se paye en saccades.

---

### Correction 4

```dart
import 'dart:math' as math;
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

/// Fabrique une planche de [colonnes] x [lignes] frames carrées de [taille] px.
Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// La pièce vue sous quatre angles : large, moyenne, de profil, moyenne.
Future<ui.Image> creerPlanchePiece() {
  const List<double> demiLargeurs = <double>[7, 4.5, 1.5, 4.5];

  return genererPlanche(
    colonnes: 4,
    lignes: 1,
    taille: 16,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      final Paint bord = Paint()
        ..color = const Color(0xFF8A5A00)
        ..isAntiAlias = false;
      final Paint or = Paint()
        ..color = const Color(0xFFFFC42E)
        ..isAntiAlias = false;

      final double rx = demiLargeurs[colonne];
      final double rxInterieur = math.max(0.4, rx - 1.2);

      canvas.drawOval(
        Rect.fromCenter(
          center: const Offset(8, 8),
          width: rx * 2,
          height: 15,
        ),
        bord,
      );
      canvas.drawOval(
        Rect.fromCenter(
          center: const Offset(8, 8),
          width: rxInterieur * 2,
          height: 12.5,
        ),
        or,
      );
    },
  );
}

void main() {
  runApp(const AppPlanchePiece());
}

class AppPlanchePiece extends StatelessWidget {
  const AppPlanchePiece({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF12121A),
        body: EcranPlanche(),
      ),
    );
  }
}

class EcranPlanche extends StatefulWidget {
  const EcranPlanche({super.key});

  @override
  State<EcranPlanche> createState() => _EcranPlancheState();
}

class _EcranPlancheState extends State<EcranPlanche> {
  ui.Image? _planche;

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    final ui.Image image = await creerPlanchePiece();
    if (!mounted) return;
    setState(() => _planche = image);
  }

  @override
  void dispose() {
    _planche?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ui.Image? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(
      painter: PeintrePlanche(planche),
      size: Size.infinite,
    );
  }
}

class PeintrePlanche extends CustomPainter {
  PeintrePlanche(this.planche);

  final ui.Image planche;

  static const int taille = 16;
  static const int colonnes = 4;
  static const double echelle = 6;

  static final Paint _net = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 13),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  /// La formule de la section 22.16, appliquée à une planche d'une seule ligne.
  Rect rectDe(int index) {
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * taille.toDouble(),
      ligne * taille.toDouble(),
      taille.toDouble(),
      taille.toDouble(),
    );
  }

  @override
  void paint(Canvas canvas, Size size) {
    _texte(canvas, 'Planche 4 x 1, frames de 16 x 16', const Offset(24, 30));
    _texte(
      canvas,
      'planche complete : ${planche.width} x ${planche.height} px',
      const Offset(24, 50),
    );

    const double cote = taille * echelle; // 96
    const double y = 100;

    for (int index = 0; index < colonnes; index++) {
      final double x = 24 + index * (cote + 12);
      canvas.drawImageRect(
        planche,
        rectDe(index),
        Rect.fromLTWH(x, y, cote, cote),
        _net,
      );
      _texte(canvas, 'frame $index', Offset(x + 18, y + cote + 8));
    }

    // La planche entière, en taille réelle, pour comparaison.
    _texte(canvas, 'la meme planche en taille reelle :', const Offset(24, 240));
    canvas.drawImage(planche, const Offset(24, 262), _net);
  }

  @override
  bool shouldRepaint(PeintrePlanche oldDelegate) =>
      oldDelegate.planche != planche;
}
```

**Résultat :**

```text
Planche 4 x 1, frames de 16 x 16
planche complete : 64 x 16 px

Quatre pièces alignées, chacune de 96 pixels de côté :
  frame 0 : disque large, vu de face
  frame 1 : ellipse plus étroite
  frame 2 : quasiment un trait vertical (la pièce vue par la tranche)
  frame 3 : ellipse identique à la frame 1
Sous chaque pièce, son numéro de frame.
Tout en bas, la planche complète en taille réelle : une bande de 64 x 16 px.
```

**Explication :** la planche entière ne fait que 64 pixels sur 16. C'est un seul objet en mémoire, une seule texture pour le GPU, quatre poses disponibles. Le `clipRect` posé par `genererPlanche` autour de chaque case garantit qu'aucune ellipse ne déborde sur sa voisine, ce qui est précisément le texture bleeding que l'on cherche à éviter (section 22.27).

La méthode `rectDe` est la formule de la section 22.16, écrite ici en clair pour la voir à l'œuvre : `colonne * taille` pour l'abscisse, `ligne * taille` pour l'ordonnée. Sur une planche d'une seule ligne, `ligne` vaut toujours `0`, mais garder la formule générale évite d'avoir à la réécrire à l'exercice 10.

Enfin, la séquence `[0, 1, 2, 3]` puis retour à `0` produit un cycle visuellement correct : la frame 3 étant identique à la frame 1, la pièce semble tourner indéfiniment dans le même sens sans à-coup. Cette économie — quatre dessins pour un tour complet — est un réflexe classique du pixel art.

---

### Correction 5

```dart
import 'dart:math' as math;
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

Future<ui.Image> creerPlanchePiece() {
  const List<double> demiLargeurs = <double>[7, 4.5, 1.5, 4.5];

  return genererPlanche(
    colonnes: 4,
    lignes: 1,
    taille: 16,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      final Paint bord = Paint()
        ..color = const Color(0xFF8A5A00)
        ..isAntiAlias = false;
      final Paint or = Paint()
        ..color = const Color(0xFFFFC42E)
        ..isAntiAlias = false;

      final double rx = demiLargeurs[colonne];
      final double rxInterieur = math.max(0.4, rx - 1.2);

      canvas.drawOval(
        Rect.fromCenter(center: const Offset(8, 8), width: rx * 2, height: 15),
        bord,
      );
      canvas.drawOval(
        Rect.fromCenter(
          center: const Offset(8, 8),
          width: rxInterieur * 2,
          height: 12.5,
        ),
        or,
      );
    },
  );
}

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * largeurFrame.toDouble(),
      ligne * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
  }) {
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    canvas.drawImageRect(
      image,
      rectDe(index),
      Rect.fromLTWH(x - l / 2, y - h / 2, l, h),
      _paintDefaut,
    );
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  int get positionCourante => _index;
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

void main() {
  runApp(const AppPieceAnimee());
}

class AppPieceAnimee extends StatelessWidget {
  const AppPieceAnimee({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF12121A),
        body: EcranPieceAnimee(),
      ),
    );
  }
}

class EcranPieceAnimee extends StatefulWidget {
  const EcranPieceAnimee({super.key});

  @override
  State<EcranPieceAnimee> createState() => _EcranPieceAnimeeState();
}

class _EcranPieceAnimeeState extends State<EcranPieceAnimee>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  SpriteSheet? _planche;
  final SpriteAnimation _rotation =
      SpriteAnimation(frames: <int>[0, 1, 2, 3], stepTime: 0.12);

  double _dt = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick);
    _demarrer();
  }

  Future<void> _demarrer() async {
    final ui.Image image = await creerPlanchePiece();
    if (!mounted) return;
    setState(() {
      _planche = SpriteSheet(
        image: image,
        largeurFrame: 16,
        hauteurFrame: 16,
      );
    });
    _ticker.start(); // La boucle ne démarre qu'APRÈS le chargement.
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;

    _rotation.avancer(dt);
    setState(() => _dt = dt);
  }

  @override
  void dispose() {
    _ticker.dispose();
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(
      painter: PeintrePieceAnimee(
        planche: planche,
        animation: _rotation,
        dt: _dt,
      ),
      size: Size.infinite,
    );
  }
}

class PeintrePieceAnimee extends CustomPainter {
  PeintrePieceAnimee({
    required this.planche,
    required this.animation,
    required this.dt,
  });

  final SpriteSheet planche;
  final SpriteAnimation animation;
  final double dt;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 14),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  @override
  void paint(Canvas canvas, Size size) {
    final double fps = dt > 0 ? 1 / dt : 0;

    _texte(canvas, 'dt   : ${(dt * 1000).toStringAsFixed(2)} ms',
        const Offset(20, 24));
    _texte(canvas, 'FPS  : ${fps.toStringAsFixed(1)}', const Offset(20, 44));
    _texte(
      canvas,
      'frame: ${animation.frameCourante} '
      '(position ${animation.positionCourante})',
      const Offset(20, 64),
    );
    _texte(canvas, 'stepTime : ${animation.stepTime} s', const Offset(20, 84));

    planche.dessiner(
      canvas,
      animation.frameCourante,
      size.width / 2,
      size.height / 2,
      echelle: 8,
    );
  }

  @override
  bool shouldRepaint(PeintrePieceAnimee oldDelegate) => true;
}
```

**Résultat :**

```text
dt   : 16.67 ms
FPS  : 60.0
frame: 2 (position 2)
stepTime : 0.12 s

Au centre de l'écran, une pièce d'or de 128 pixels qui tourne sur elle-même,
environ deux tours par seconde. Le compteur de frame change toutes les
120 millisecondes, quel que soit le nombre d'images par seconde affiché.
```

**Explication :** le panneau d'information est ici l'outil pédagogique principal. Sur un écran 60 Hz, `dt` vaut environ 16,7 ms ; sur un écran 120 Hz, environ 8,3 ms et les FPS doublent. Or l'index de frame, lui, change toujours toutes les 120 ms. C'est la preuve visuelle que l'animation est pilotée par le temps et non par le rendu.

Trois points d'architecture méritent d'être notés.

`_ticker.start()` est appelé **après** le `await`. Tant que la planche n'est pas prête, la boucle de jeu n'existe pas : impossible de dessiner une image nulle.

Le garde-fou `if (dt <= 0 || dt > 0.25) return;` protège de deux cas réels. La toute première frame, où `_precedent` vaut encore `Duration.zero`, donnerait un `dt` énorme. Et un retour de veille produirait un `dt` de plusieurs secondes, qui ferait défiler des dizaines de frames d'un coup dans le `while` de `avancer`.

Enfin `shouldRepaint` renvoie `true` sans condition. C'est volontaire dans une boucle de jeu : l'état change à chaque frame, et tenter de le détecter coûterait plus cher que de repeindre.

---

### Correction 6

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// Cinq frames de 24 x 24 : le coffre du Donjon de Dart qui s'ouvre.
/// Recette 4 de la section 22.29 : une progression linéaire de 0.0 à 1.0.
Future<ui.Image> creerPlancheCoffre() {
  return genererPlanche(
    colonnes: 5,
    lignes: 1,
    taille: 24,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      final double avancement = colonne / 4; // 0.00, 0.25, 0.50, 0.75, 1.00

      final Paint bois = Paint()
        ..color = const Color(0xFF7A4A1E)
        ..isAntiAlias = false;
      final Paint boisClair = Paint()
        ..color = const Color(0xFF9A6430)
        ..isAntiAlias = false;
      final Paint ferrure = Paint()
        ..color = const Color(0xFFC9A227)
        ..isAntiAlias = false;
      final Paint interieur = Paint()
        ..color = const Color(0xFF2A1A0A)
        ..isAntiAlias = false;

      // L'intérieur sombre, visible dès que le couvercle se soulève.
      canvas.drawRect(const Rect.fromLTWH(4, 10, 16, 6), interieur);

      // Le corps du coffre.
      canvas.drawRect(const Rect.fromLTWH(3, 13, 18, 9), bois);
      canvas.drawRect(const Rect.fromLTWH(3, 16, 18, 2), ferrure);

      // Le couvercle : il pivote autour de sa charnière arrière gauche.
      canvas.save();
      canvas.translate(3, 13);
      canvas.rotate(-1.2 * avancement); // jusqu'à environ 69 degrés
      canvas.drawRect(const Rect.fromLTWH(0, -6, 18, 6), boisClair);
      canvas.drawRect(const Rect.fromLTWH(0, -6, 18, 1), ferrure);
      canvas.restore();

      // La lueur du trésor, sur la dernière frame uniquement.
      if (colonne == 4) {
        final Paint lueur = Paint()
          ..color = const Color(0x88FFE066)
          ..isAntiAlias = false;
        canvas.drawCircle(const Offset(12, 12), 5, lueur);
      }
    },
  );
}

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * largeurFrame.toDouble(),
      ligne * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
  }) {
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    canvas.drawImageRect(
      image,
      rectDe(index),
      Rect.fromLTWH(x - l / 2, y - h / 2, l, h),
      _paintDefaut,
    );
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

void main() {
  runApp(const AppCoffre());
}

class AppCoffre extends StatelessWidget {
  const AppCoffre({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF161020),
        body: EcranCoffre(),
      ),
    );
  }
}

class EcranCoffre extends StatefulWidget {
  const EcranCoffre({super.key});

  @override
  State<EcranCoffre> createState() => _EcranCoffreState();
}

class _EcranCoffreState extends State<EcranCoffre>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  SpriteSheet? _planche;
  final SpriteAnimation _ouverture = SpriteAnimation(
    frames: <int>[0, 1, 2, 3, 4],
    stepTime: 0.15,
    loop: false, // Le point clé de l'exercice.
  );

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick);
    _demarrer();
  }

  Future<void> _demarrer() async {
    final ui.Image image = await creerPlancheCoffre();
    if (!mounted) return;
    setState(() {
      _planche = SpriteSheet(
        image: image,
        largeurFrame: 24,
        hauteurFrame: 24,
      );
    });
    _ticker.start();
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;

    _ouverture.avancer(dt);
    setState(() {});
  }

  void _auTouche() {
    // Une pression pendant l'ouverture est purement et simplement ignorée.
    if (!_ouverture.estTerminee) return;
    _ouverture.reinitialiser();
  }

  @override
  void dispose() {
    _ticker.dispose();
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return GestureDetector(
      behavior: HitTestBehavior.opaque,
      onTap: _auTouche,
      child: CustomPaint(
        painter: PeintreCoffre(planche: planche, animation: _ouverture),
        size: Size.infinite,
      ),
    );
  }
}

class PeintreCoffre extends CustomPainter {
  PeintreCoffre({required this.planche, required this.animation});

  final SpriteSheet planche;
  final SpriteAnimation animation;

  void _texte(Canvas canvas, String contenu, Offset position, Color couleur) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: TextStyle(color: couleur, fontSize: 16),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  @override
  void paint(Canvas canvas, Size size) {
    // Le sol du donjon.
    final Paint sol = Paint()..color = const Color(0xFF2A2438);
    canvas.drawRect(
      Rect.fromLTWH(0, size.height / 2 + 60, size.width, size.height),
      sol,
    );

    planche.dessiner(
      canvas,
      animation.frameCourante,
      size.width / 2,
      size.height / 2,
      echelle: 6,
    );

    final bool fini = animation.estTerminee;
    _texte(
      canvas,
      fini
          ? 'coffre ouvert, touchez pour refermer'
          : 'ouverture en cours...',
      const Offset(24, 30),
      fini ? const Color(0xFFFFE066) : const Color(0xFFB8B8C8),
    );
    _texte(
      canvas,
      'frame ${animation.frameCourante} sur 4',
      const Offset(24, 56),
      const Color(0xFF8888A0),
    );
  }

  @override
  bool shouldRepaint(PeintreCoffre oldDelegate) => true;
}
```

**Résultat :**

```text
ouverture en cours...
frame 2 sur 4

Au centre, un coffre de 144 pixels dont le couvercle se relève en cinq étapes,
en 0,75 seconde. Une lueur dorée apparaît à la dernière frame.
Puis plus rien ne bouge : le coffre reste ouvert.

Une pression referme le coffre d'un coup et rejoue l'ouverture.
Une pression PENDANT l'ouverture ne fait rien.
```

**Explication :** `loop: false` change entièrement le contrat de la classe. Au lieu de tourner indéfiniment, l'animation s'arrête sur sa dernière frame et lève `estTerminee`. Le `return` placé en tête de `avancer` fait alors du calcul une opération nulle : le coffre ouvert ne consomme plus rien.

Le drapeau `estTerminee` sert ici à deux choses, et c'est tout son intérêt. Il pilote l'affichage du message, et il **filtre l'entrée utilisateur** : `if (!_ouverture.estTerminee) return;`. Sans cette ligne, un joueur qui tapote l'écran remettrait l'animation à zéro en boucle et le couvercle ne se lèverait jamais. C'est la version « entrée » du bug de l'animation figée décrit en 22.26.

Sur le dessin lui-même, notez la superposition volontaire : l'intérieur sombre est peint **avant** le corps du coffre, donc il n'apparaît que dans l'espace libéré par le couvercle qui pivote. Le `canvas.rotate` est encadré par `save()` et `restore()` (chapitre 21), faute de quoi la rotation s'appliquerait aussi à la lueur, puis à toutes les frames suivantes.

---

### Correction 7

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// Quatre frames de marche pour un gobelin, recette 2 de la section 22.29 :
/// une table de valeurs indexée par la frame.
Future<ui.Image> creerPlancheGobelin() {
  const List<double> ecartJambes = <double>[0, 3, 0, -3];
  const List<double> hauteurBuste = <double>[0, -1, 0, -1];

  return genererPlanche(
    colonnes: 4,
    lignes: 1,
    taille: 24,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      final Paint peau = Paint()
        ..color = const Color(0xFF5FA847)
        ..isAntiAlias = false;
      final Paint pagne = Paint()
        ..color = const Color(0xFF7A4A1E)
        ..isAntiAlias = false;
      final Paint oeil = Paint()
        ..color = const Color(0xFFFFE066)
        ..isAntiAlias = false;

      final double e = ecartJambes[colonne];
      final double dy = hauteurBuste[colonne];

      // Jambes : elles s'écartent selon la frame.
      canvas.drawRect(Rect.fromLTWH(9 - e, 17, 3, 6), peau);
      canvas.drawRect(Rect.fromLTWH(12 + e, 17, 3, 6), peau);

      // Buste et pagne, très légèrement remontés une frame sur deux.
      canvas.drawRect(Rect.fromLTWH(8, 14 + dy, 8, 4), pagne);
      canvas.drawRect(Rect.fromLTWH(8, 8 + dy, 8, 6), peau);

      // Bras : ils balancent à l'inverse des jambes.
      canvas.drawRect(Rect.fromLTWH(6 + e, 9 + dy, 2, 5), peau);
      canvas.drawRect(Rect.fromLTWH(16 - e, 9 + dy, 2, 5), peau);

      // Tête, oreille pointue vers la droite, et oeil.
      canvas.drawRect(Rect.fromLTWH(8, 2 + dy, 8, 6), peau);
      canvas.drawRect(Rect.fromLTWH(16, 3 + dy, 3, 2), peau);
      canvas.drawRect(Rect.fromLTWH(13, 4 + dy, 2, 2), oeil);
    },
  );
}

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      colonne * largeurFrame.toDouble(),
      ligne * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  /// [x] est le centre horizontal, [y] la ligne des pieds.
  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
  }) {
    final Rect src = rectDe(index);
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(x - l / 2, y - h, l, h);

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, _paintDefaut);
      return;
    }

    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, l, h), _paintDefaut);
    canvas.restore();
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

/// Le gobelin : logique pure, aucun Canvas ici.
class Gobelin {
  Gobelin({required this.x, required this.y});

  double x;
  double y;
  double vitesse = 80; // pixels par seconde
  int direction = 1; // 1 vers la droite, -1 vers la gauche

  final SpriteAnimation marche =
      SpriteAnimation(frames: <int>[0, 1, 2, 3], stepTime: 0.14);

  bool get regardeAGauche => direction < 0;

  void mettreAJour(double dt, double largeurMonde) {
    x += vitesse * direction * dt;

    const double marge = 40;
    if (x > largeurMonde - marge) {
      x = largeurMonde - marge;
      direction = -1;
    } else if (x < marge) {
      x = marge;
      direction = 1;
    }

    marche.avancer(dt);
  }
}

void main() {
  runApp(const AppGobelin());
}

class AppGobelin extends StatelessWidget {
  const AppGobelin({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14121C),
        body: EcranGobelin(),
      ),
    );
  }
}

class EcranGobelin extends StatefulWidget {
  const EcranGobelin({super.key});

  @override
  State<EcranGobelin> createState() => _EcranGobelinState();
}

class _EcranGobelinState extends State<EcranGobelin>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  SpriteSheet? _planche;
  final Gobelin _gobelin = Gobelin(x: 60, y: 0);
  Size _monde = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick);
    _demarrer();
  }

  Future<void> _demarrer() async {
    final ui.Image image = await creerPlancheGobelin();
    if (!mounted) return;
    setState(() {
      _planche = SpriteSheet(
        image: image,
        largeurFrame: 24,
        hauteurFrame: 24,
      );
    });
    _ticker.start();
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;
    if (_monde.width == 0) return; // La taille n'est pas encore connue.

    _gobelin.y = _monde.height * 0.7;
    _gobelin.mettreAJour(dt, _monde.width);
    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose();
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints contraintes) {
        _monde = Size(contraintes.maxWidth, contraintes.maxHeight);
        return CustomPaint(
          painter: PeintreGobelin(planche: planche, gobelin: _gobelin),
          size: Size.infinite,
        );
      },
    );
  }
}

class PeintreGobelin extends CustomPainter {
  PeintreGobelin({required this.planche, required this.gobelin});

  final SpriteSheet planche;
  final Gobelin gobelin;

  @override
  void paint(Canvas canvas, Size size) {
    final double sol = size.height * 0.7;

    // Le sol du donjon.
    canvas.drawRect(
      Rect.fromLTWH(0, sol, size.width, size.height - sol),
      Paint()..color = const Color(0xFF2A2438),
    );
    canvas.drawRect(
      Rect.fromLTWH(0, sol, size.width, 2),
      Paint()..color = const Color(0xFF4A4260),
    );

    planche.dessiner(
      canvas,
      gobelin.marche.frameCourante,
      gobelin.x,
      sol,
      echelle: 4,
      retourne: gobelin.regardeAGauche,
    );

    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: 'x = ${gobelin.x.toStringAsFixed(1)}   '
            'direction = ${gobelin.direction}   '
            'frame = ${gobelin.marche.frameCourante}',
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 14),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, const Offset(20, 28));
  }

  @override
  bool shouldRepaint(PeintreGobelin oldDelegate) => true;
}
```

**Résultat :**

```text
x = 213.4   direction = 1   frame = 2

Un gobelin vert de 96 pixels marche sur le sol du donjon, de gauche à droite,
à 80 pixels par seconde. Ses jambes s'écartent et se resserrent, ses bras
balancent en opposition, son buste monte et descend d'un pixel.
Arrivé à 40 pixels du bord droit, il se retourne d'un coup et repart vers la
gauche, son oeil et son oreille pointant maintenant du bon côté.
```

**Explication :** trois mécanismes se combinent ici, et il est important de voir qu'ils sont indépendants.

Le **déplacement** utilise `x += vitesse * direction * dt` : c'est la formule du chapitre 20, en pixels par seconde. L'**animation** utilise l'accumulateur de `SpriteAnimation` : elle avance au rythme de `stepTime`, sans aucun lien avec la vitesse. Le **retournement** n'est qu'un booléen dérivé de `direction`, appliqué au moment du dessin.

Le point délicat de l'exercice était le saut de position au retournement. Il est évité parce que `dessiner` prend `x` comme **centre horizontal** et calcule `dst` symétriquement autour de lui, puis retourne le sprite autour de ce même rectangle : `translate(dst.right, dst.top)` puis `scale(-1, 1)`. Le rectangle occupé à l'écran est rigoureusement le même dans les deux sens. Si l'on avait pris `x` comme bord gauche, le sprite se serait décalé de sa largeur entière au moment du demi-tour, avec un saut de 96 pixels bien visible.

Notez aussi la classe `Gobelin` : elle ne connaît ni `Canvas`, ni `SpriteSheet`, seulement des nombres et une animation. C'est la séparation logique / rendu que le chapitre 26 systématisera. Elle a un effet immédiat : on peut tester `mettreAJour` en pur Dart, sans lancer l'application.

Enfin, `_monde` est renseigné par le `LayoutBuilder`. Tant que la taille n'est pas connue, `_tick` sort immédiatement : c'est plus sûr que de deviner une largeur d'écran.

---

### Correction 8

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// Planche 4 x 3 : ligne 0 repos, ligne 1 marche, ligne 2 attaque.
Future<ui.Image> creerPlancheGobelin() {
  const List<double> ecartRepos = <double>[0, 0, 0, 0];
  const List<double> ecartMarche = <double>[0, 3, 0, -3];
  const List<double> respiration = <double>[0, -1, 0, -1];

  return genererPlanche(
    colonnes: 4,
    lignes: 3,
    taille: 24,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      final Paint peau = Paint()
        ..color = const Color(0xFF5FA847)
        ..isAntiAlias = false;
      final Paint pagne = Paint()
        ..color = const Color(0xFF7A4A1E)
        ..isAntiAlias = false;
      final Paint oeil = Paint()
        ..color = const Color(0xFFFFE066)
        ..isAntiAlias = false;
      final Paint lame = Paint()
        ..color = const Color(0xFFCFD6E6)
        ..isAntiAlias = false;

      final double e = (ligne == 1) ? ecartMarche[colonne] : ecartRepos[colonne];
      final double dy = respiration[colonne];

      // Jambes.
      canvas.drawRect(Rect.fromLTWH(9 - e, 17, 3, 6), peau);
      canvas.drawRect(Rect.fromLTWH(12 + e, 17, 3, 6), peau);

      // Buste, pagne, tête.
      canvas.drawRect(Rect.fromLTWH(8, 14 + dy, 8, 4), pagne);
      canvas.drawRect(Rect.fromLTWH(8, 8 + dy, 8, 6), peau);
      canvas.drawRect(Rect.fromLTWH(8, 2 + dy, 8, 6), peau);
      canvas.drawRect(Rect.fromLTWH(16, 3 + dy, 3, 2), peau);
      canvas.drawRect(Rect.fromLTWH(13, 4 + dy, 2, 2), oeil);

      if (ligne == 2) {
        // Attaque : la lame sort progressivement (recette 4, section 22.29).
        final double avancement = colonne / 3; // 0.00 -> 1.00
        final double longueur = 2 + 10 * avancement;
        canvas.drawRect(Rect.fromLTWH(16, 10 + dy, 2, 2), peau); // le bras
        canvas.drawRect(Rect.fromLTWH(18, 10 + dy, longueur, 1), lame);
      } else {
        // Bras au repos ou en balancement.
        canvas.drawRect(Rect.fromLTWH(6 + e, 9 + dy, 2, 5), peau);
        canvas.drawRect(Rect.fromLTWH(16 - e, 9 + dy, 2, 5), peau);
      }
    },
  );
}

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    return Rect.fromLTWH(
      (index % colonnes) * largeurFrame.toDouble(),
      (index ~/ colonnes) * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  List<int> indexLigne(int ligne) =>
      List<int>.generate(colonnes, (int i) => ligne * colonnes + i);

  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
  }) {
    final Rect src = rectDe(index);
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(x - l / 2, y - h, l, h);

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, _paintDefaut);
      return;
    }
    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, l, h), _paintDefaut);
    canvas.restore();
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  int get positionCourante => _index;
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }
}

/// Les trois états possibles du gobelin (enum du chapitre 11).
enum EtatGobelin { repos, marche, attaque }

class Gobelin {
  Gobelin({required this.planche, required this.x}) {
    animations = <EtatGobelin, SpriteAnimation>{
      EtatGobelin.repos: SpriteAnimation(
        frames: planche.indexLigne(0),
        stepTime: 0.25,
      ),
      EtatGobelin.marche: SpriteAnimation(
        frames: planche.indexLigne(1),
        stepTime: 0.14,
      ),
      EtatGobelin.attaque: SpriteAnimation(
        frames: planche.indexLigne(2),
        stepTime: 0.09,
        loop: false,
      ),
    };
    assert(
      EtatGobelin.values.every(animations.containsKey),
      'Il manque une animation pour au moins un état',
    );
  }

  final SpriteSheet planche;
  double x;

  late final Map<EtatGobelin, SpriteAnimation> animations;

  EtatGobelin etat = EtatGobelin.repos;
  double vitesse = 0;
  int direction = 1;

  double _horloge = 0;
  bool _attaqueDemandee = false;

  SpriteAnimation get animationCourante => animations[etat]!;
  bool get regardeAGauche => direction < 0;

  void demanderAttaque() {
    if (etat == EtatGobelin.attaque) return; // déjà en train d'attaquer
    _attaqueDemandee = true;
  }

  void mettreAJour(double dt, double largeurMonde) {
    // Toutes les deux secondes, le gobelin s'arrête ou repart.
    _horloge += dt;
    if (_horloge >= 2) {
      _horloge -= 2;
      vitesse = (vitesse == 0) ? 80 : 0;
    }

    // On ne se déplace pas pendant une attaque.
    if (etat != EtatGobelin.attaque) {
      x += vitesse * direction * dt;
      const double marge = 50;
      if (x > largeurMonde - marge) {
        x = largeurMonde - marge;
        direction = -1;
      } else if (x < marge) {
        x = marge;
        direction = 1;
      }
    }

    choisirEtat();
    animationCourante.avancer(dt);
  }

  /// Du plus prioritaire au moins prioritaire (section 22.25).
  void choisirEtat() {
    // 1. Une attaque en cours ne s'interrompt pas.
    if (etat == EtatGobelin.attaque && !animationCourante.estTerminee) {
      return;
    }
    // 2. Une attaque demandée démarre immédiatement.
    if (_attaqueDemandee) {
      _attaqueDemandee = false;
      _passerA(EtatGobelin.attaque);
      return;
    }
    // 3. Sinon, la vitesse décide.
    if (vitesse.abs() > 1) {
      _passerA(EtatGobelin.marche);
    } else {
      _passerA(EtatGobelin.repos);
    }
  }

  /// La ligne la plus importante du chapitre est la première.
  void _passerA(EtatGobelin nouveau) {
    if (etat == nouveau) return;
    etat = nouveau;
    animations[nouveau]!.reinitialiser();
  }
}

void main() {
  runApp(const AppEtats());
}

class AppEtats extends StatelessWidget {
  const AppEtats({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF14121C),
        body: EcranEtats(),
      ),
    );
  }
}

class EcranEtats extends StatefulWidget {
  const EcranEtats({super.key});

  @override
  State<EcranEtats> createState() => _EcranEtatsState();
}

class _EcranEtatsState extends State<EcranEtats>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  SpriteSheet? _planche;
  Gobelin? _gobelin;
  Size _monde = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick);
    _demarrer();
  }

  Future<void> _demarrer() async {
    final ui.Image image = await creerPlancheGobelin();
    if (!mounted) return;
    final SpriteSheet planche = SpriteSheet(
      image: image,
      largeurFrame: 24,
      hauteurFrame: 24,
    );
    setState(() {
      _planche = planche;
      _gobelin = Gobelin(planche: planche, x: 120);
    });
    _ticker.start();
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;
    if (_monde.width == 0) return;

    _gobelin?.mettreAJour(dt, _monde.width);
    setState(() {});
  }

  @override
  void dispose() {
    _ticker.dispose();
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    final Gobelin? gobelin = _gobelin;
    if (planche == null || gobelin == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return GestureDetector(
      behavior: HitTestBehavior.opaque,
      onTap: gobelin.demanderAttaque,
      child: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints contraintes) {
          _monde = Size(contraintes.maxWidth, contraintes.maxHeight);
          return CustomPaint(
            painter: PeintreEtats(planche: planche, gobelin: gobelin),
            size: Size.infinite,
          );
        },
      ),
    );
  }
}

class PeintreEtats extends CustomPainter {
  PeintreEtats({required this.planche, required this.gobelin});

  final SpriteSheet planche;
  final Gobelin gobelin;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 14),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  @override
  void paint(Canvas canvas, Size size) {
    final double sol = size.height * 0.72;

    canvas.drawRect(
      Rect.fromLTWH(0, sol, size.width, size.height - sol),
      Paint()..color = const Color(0xFF2A2438),
    );

    planche.dessiner(
      canvas,
      gobelin.animationCourante.frameCourante,
      gobelin.x,
      sol,
      echelle: 4,
      retourne: gobelin.regardeAGauche,
    );

    _texte(canvas, 'etat    : ${gobelin.etat.name}', const Offset(20, 28));
    _texte(
      canvas,
      'frame   : ${gobelin.animationCourante.frameCourante} '
      '(position ${gobelin.animationCourante.positionCourante})',
      const Offset(20, 50),
    );
    _texte(
      canvas,
      'vitesse : ${gobelin.vitesse.toStringAsFixed(0)} px/s',
      const Offset(20, 72),
    );
    _texte(canvas, 'touchez l\'ecran pour attaquer', const Offset(20, 100));
  }

  @override
  bool shouldRepaint(PeintreEtats oldDelegate) => true;
}
```

**Résultat :**

```text
etat    : marche
frame   : 6 (position 2)
vitesse : 80 px/s

touchez l'ecran pour attaquer

Le gobelin alterne automatiquement : deux secondes immobile en respiration
lente, deux secondes de marche à 80 px/s, et ainsi de suite.
Une pression sur l'écran interrompt tout : il se plante sur place, sort sa
lame en quatre frames (0,36 seconde), puis reprend exactement l'activité
qu'il avait avant. Pendant l'attaque, aucune autre pression n'a d'effet et
il ne se déplace pas.
```

**Explication :** cette correction est le cœur du chapitre. Trois responsabilités y sont soigneusement séparées.

`mettreAJour` fait avancer le **monde** : l'horloge, la position, puis l'animation. Elle ne décide de rien.

`choisirEtat` répond à une seule question : « quel état devrais-je avoir maintenant ? ». L'ordre des tests **est** la table des priorités. Le premier `if` est le garde-fou qui rend l'attaque non interruptible : tant que l'animation d'attaque n'est pas terminée, la méthode sort sans rien changer. Si vous déplaciez ce test après celui de la vitesse, l'attaque serait annulée dès le pas suivant et vous ne verriez jamais la lame sortir.

`_passerA` répond à l'autre question : « comment j'y passe ? ». Son `if (etat == nouveau) return;` est indispensable. Sans lui, `choisirEtat` appellerait `_passerA(EtatGobelin.marche)` soixante fois par seconde, donc `reinitialiser()` soixante fois par seconde, et le gobelin marcherait avec les jambes bloquées sur la frame 4.

Deux détails valent d'être relevés. `animations` est `late final` : la `Map` est construite dans le corps du constructeur parce qu'elle a besoin de `planche`, mais elle ne sera jamais remplacée (chapitre 12). Et `indexLigne(0)`, `indexLigne(1)`, `indexLigne(2)` évitent d'écrire les listes `[0,1,2,3]`, `[4,5,6,7]`, `[8,9,10,11]` à la main : le jour où la planche passe à 6 colonnes, rien à corriger.

---

### Correction 9

```dart
import 'dart:ui' as ui;

import 'package:flutter/material.dart';

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

/// Génère une planche AVEC marge extérieure et séparateurs entre les cases.
/// Le fond est peint en rouge vif : ce rouge ne doit JAMAIS apparaître
/// à l'écran si la formule de découpe est correcte.
Future<ui.Image> genererPlancheAvecMarge({
  required int colonnes,
  required int lignes,
  required int taille,
  required int marge,
  required int espacement,
  required void Function(Canvas canvas, int index) dessinFrame,
}) {
  final int largeur =
      2 * marge + colonnes * taille + (colonnes - 1) * espacement;
  final int hauteur = 2 * marge + lignes * taille + (lignes - 1) * espacement;

  return imageDepuisDessin(largeur, hauteur, (Canvas canvas) {
    canvas.drawRect(
      Rect.fromLTWH(0, 0, largeur.toDouble(), hauteur.toDouble()),
      Paint()
        ..color = const Color(0xFFFF0044)
        ..isAntiAlias = false,
    );

    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        final double x = (marge + c * (taille + espacement)).toDouble();
        final double y = (marge + l * (taille + espacement)).toDouble();

        canvas.save();
        canvas.translate(x, y);
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l * colonnes + c);
        canvas.restore();
      }
    }
  });
}

/// Une planche de sprites qui tient compte de la marge et de l'espacement.
class SpriteSheetPad {
  SpriteSheetPad({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
    this.marge = 0,
    this.espacement = 0,
  })  : colonnes = (image.width - 2 * marge + espacement) ~/
            (largeurFrame + espacement),
        lignes = (image.height - 2 * marge + espacement) ~/
            (hauteurFrame + espacement);

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int marge;
  final int espacement;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  /// La formule complète :
  ///   x = marge + colonne * (largeurFrame + espacement)
  ///   y = marge + ligne   * (hauteurFrame + espacement)
  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    final int ligne = index ~/ colonnes;
    final int colonne = index % colonnes;
    return Rect.fromLTWH(
      (marge + colonne * (largeurFrame + espacement)).toDouble(),
      (marge + ligne * (hauteurFrame + espacement)).toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  void dessiner(Canvas canvas, int index, Rect destination) {
    canvas.drawImageRect(image, rectDe(index), destination, _paintDefaut);
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

/// Huit tuiles du donjon, une par index.
void dessinerTuile(Canvas canvas, int index) {
  const List<Color> fonds = <Color>[
    Color(0xFF3B3550),
    Color(0xFF4A4260),
    Color(0xFF2A5E3A),
    Color(0xFF5E4A2A),
    Color(0xFF2A3E5E),
    Color(0xFF5E2A3E),
    Color(0xFF3E5E2A),
    Color(0xFF52526A),
  ];

  final Paint fond = Paint()
    ..color = fonds[index]
    ..isAntiAlias = false;
  final Paint clair = Paint()
    ..color = const Color(0xFFE8E8F0)
    ..isAntiAlias = false;

  // Le fond couvre TOUTE la case : aucun pixel transparent ne doit rester.
  canvas.drawRect(const Rect.fromLTWH(0, 0, 16, 16), fond);

  // Un motif qui dépend de l'index, pour distinguer les huit tuiles.
  for (int i = 0; i <= index; i++) {
    canvas.drawRect(Rect.fromLTWH(2 + i.toDouble(), 12, 1, 2), clair);
  }
  canvas.drawRect(const Rect.fromLTWH(0, 0, 16, 1), clair);
}

void main() {
  runApp(const AppPadding());
}

class AppPadding extends StatelessWidget {
  const AppPadding({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF12121A),
        body: EcranPadding(),
      ),
    );
  }
}

class EcranPadding extends StatefulWidget {
  const EcranPadding({super.key});

  @override
  State<EcranPadding> createState() => _EcranPaddingState();
}

class _EcranPaddingState extends State<EcranPadding> {
  SpriteSheetPad? _planche;

  @override
  void initState() {
    super.initState();
    _preparer();
  }

  Future<void> _preparer() async {
    final ui.Image image = await genererPlancheAvecMarge(
      colonnes: 4,
      lignes: 2,
      taille: 16,
      marge: 2,
      espacement: 2,
      dessinFrame: dessinerTuile,
    );
    if (!mounted) return;
    setState(() {
      _planche = SpriteSheetPad(
        image: image,
        largeurFrame: 16,
        hauteurFrame: 16,
        marge: 2,
        espacement: 2,
      );
    });
  }

  @override
  void dispose() {
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheetPad? planche = _planche;
    if (planche == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return CustomPaint(
      painter: PeintrePadding(planche),
      size: Size.infinite,
    );
  }
}

class PeintrePadding extends CustomPainter {
  PeintrePadding(this.planche);

  final SpriteSheetPad planche;

  static final Paint _net = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFE8E8F0), fontSize: 13),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  @override
  void paint(Canvas canvas, Size size) {
    _texte(
      canvas,
      'planche : ${planche.image.width} x ${planche.image.height} px, '
      'grille calculee : ${planche.colonnes} x ${planche.lignes}',
      const Offset(20, 24),
    );

    // 1. La planche entière, agrandie 3 fois : le rouge des séparateurs
    //    est bien visible ici.
    canvas.drawImageRect(
      planche.image,
      Rect.fromLTWH(
        0,
        0,
        planche.image.width.toDouble(),
        planche.image.height.toDouble(),
      ),
      Rect.fromLTWH(
        20,
        50,
        planche.image.width * 3,
        planche.image.height * 3,
      ),
      _net,
    );
    _texte(canvas, 'la planche brute (marge et separateurs en rouge)',
        Offset(20, 50 + planche.image.height * 3 + 8));

    // 2. Les huit frames découpées : plus aucun rouge.
    const double cote = 64; // 16 x 4
    final double yBase = 50 + planche.image.height * 3 + 40;

    for (int index = 0; index < planche.nombreFrames; index++) {
      final int ligne = index ~/ 4;
      final int colonne = index % 4;
      planche.dessiner(
        canvas,
        index,
        Rect.fromLTWH(
          20 + colonne * (cote + 8),
          yBase + ligne * (cote + 8),
          cote,
          cote,
        ),
      );
    }

    _texte(canvas, 'les 8 frames decoupees : aucun rouge',
        Offset(20, yBase + 2 * (cote + 8) + 4));
  }

  @override
  bool shouldRepaint(PeintrePadding oldDelegate) =>
      oldDelegate.planche != planche;
}
```

**Résultat :**

```text
planche : 74 x 38 px, grille calculee : 4 x 2

En haut, la planche brute agrandie 3 fois : un quadrillage rouge vif entoure
et sépare huit tuiles colorées.
En dessous, les huit tuiles découpées et agrandies 4 fois : elles sont
parfaitement propres, sans le moindre pixel rouge sur les bords.
```

**Explication :** l'intérêt de cet exercice est d'être **auto-vérifiant**. Le rouge joue le rôle de traceur : s'il apparaît sur un bord de frame, c'est que la formule de découpe est fausse. Vous pouvez le constater en changeant `marge: 2` en `marge: 0` dans le constructeur de `SpriteSheetPad` — sans toucher au générateur : un liseré rouge surgit immédiatement en haut et à gauche de chaque tuile.

La formule inverse mérite qu'on s'y arrête :

```text
largeur = 2 * marge + colonnes * largeurFrame + (colonnes - 1) * espacement
```

Pour retrouver `colonnes`, on ajoute un espacement virtuel à droite de la dernière case, ce qui rend la division exacte :

```text
colonnes = (largeur - 2 * marge + espacement) / (largeurFrame + espacement)
```

Sur nos chiffres : `(74 - 4 + 2) / (16 + 2) = 72 / 18 = 4`. Et pour les lignes : `(38 - 4 + 2) / 18 = 2`. Le calcul est fait par le constructeur, pas fourni par l'appelant : impossible de déclarer une grille incohérente avec l'image.

Une remarque de méthode pour finir. Dans un vrai projet, la marge sert à autre chose : on y recopie le pixel du bord de la frame (technique dite d'*extrusion*), de sorte que si le GPU déborde d'un demi-pixel, il lise une couleur identique au bord au lieu d'une couleur voisine. Combinée à `FilterQuality.none`, cette technique fait disparaître définitivement le texture bleeding de la section 22.27.

---

### Correction 10

```dart
import 'dart:math' as math;
import 'dart:ui' as ui;

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

// ---------------------------------------------------------------------------
// 1. Fabrication des images, sans aucun fichier
// ---------------------------------------------------------------------------

Future<ui.Image> imageDepuisDessin(
  int largeur,
  int hauteur,
  void Function(Canvas canvas) dessin,
) async {
  final ui.PictureRecorder enregistreur = ui.PictureRecorder();
  final Canvas canvas = Canvas(enregistreur);
  dessin(canvas);
  final ui.Picture picture = enregistreur.endRecording();
  return picture.toImage(largeur, hauteur);
}

Future<ui.Image> genererPlanche({
  required int colonnes,
  required int lignes,
  required int taille,
  required void Function(Canvas canvas, int ligne, int colonne) dessinFrame,
}) {
  return imageDepuisDessin(colonnes * taille, lignes * taille, (Canvas canvas) {
    for (int l = 0; l < lignes; l++) {
      for (int c = 0; c < colonnes; c++) {
        canvas.save();
        canvas.translate(c * taille.toDouble(), l * taille.toDouble());
        canvas.clipRect(
          Rect.fromLTWH(0, 0, taille.toDouble(), taille.toDouble()),
        );
        dessinFrame(canvas, l, c);
        canvas.restore();
      }
    }
  });
}

/// Planche 4 x 3, frames de 24 x 24 :
///   ligne 0 : heros au repos
///   ligne 1 : heros en marche
///   ligne 2 : potion qui pulse
Future<ui.Image> creerPlancheDonjon() {
  const List<double> ecartMarche = <double>[0, 3, 0, -3];
  const List<double> respiration = <double>[0, -1, 0, -1];

  return genererPlanche(
    colonnes: 4,
    lignes: 3,
    taille: 24,
    dessinFrame: (Canvas canvas, int ligne, int colonne) {
      if (ligne <= 1) {
        _dessinerHeros(
          canvas,
          ecart: ligne == 1 ? ecartMarche[colonne] : 0,
          dy: respiration[colonne],
        );
      } else {
        _dessinerPotion(canvas, colonne);
      }
    },
  );
}

void _dessinerHeros(Canvas canvas, {required double ecart, required double dy}) {
  final Paint peau = Paint()
    ..color = const Color(0xFFF2C9A0)
    ..isAntiAlias = false;
  final Paint tunique = Paint()
    ..color = const Color(0xFF3E6BD8)
    ..isAntiAlias = false;
  final Paint bottes = Paint()
    ..color = const Color(0xFF5A3A1E)
    ..isAntiAlias = false;
  final Paint cheveux = Paint()
    ..color = const Color(0xFF6B3A1E)
    ..isAntiAlias = false;

  // Jambes.
  canvas.drawRect(Rect.fromLTWH(9 - ecart, 18, 3, 5), bottes);
  canvas.drawRect(Rect.fromLTWH(12 + ecart, 18, 3, 5), bottes);

  // Tunique et bras.
  canvas.drawRect(Rect.fromLTWH(8, 11 + dy, 8, 7), tunique);
  canvas.drawRect(Rect.fromLTWH(6 + ecart, 12 + dy, 2, 5), peau);
  canvas.drawRect(Rect.fromLTWH(16 - ecart, 12 + dy, 2, 5), peau);

  // Tête et cheveux.
  canvas.drawRect(Rect.fromLTWH(9, 5 + dy, 6, 6), peau);
  canvas.drawRect(Rect.fromLTWH(9, 4 + dy, 6, 2), cheveux);
}

void _dessinerPotion(Canvas canvas, int colonne) {
  // Recette 3 de la section 22.29 : une sinusoide pour un cycle continu.
  final double angle = 2 * math.pi * colonne / 4;
  final double pulse = math.sin(angle); // -1 .. +1

  final Paint verre = Paint()
    ..color = const Color(0xFF9FE0FF)
    ..isAntiAlias = false;
  final Paint liquide = Paint()
    ..color = const Color(0xFFE0245E)
    ..isAntiAlias = false;
  final Paint bouchon = Paint()
    ..color = const Color(0xFF8B5A2B)
    ..isAntiAlias = false;

  final double hauteurLiquide = 6 + pulse;
  final double hautLiquide = 18 - hauteurLiquide;

  canvas.drawRect(const Rect.fromLTWH(10, 6, 4, 3), bouchon);
  canvas.drawRect(const Rect.fromLTWH(8, 9, 8, 10), verre);
  canvas.drawRect(Rect.fromLTWH(9, hautLiquide, 6, hauteurLiquide), liquide);
}

// ---------------------------------------------------------------------------
// 2. Les outils du chapitre
// ---------------------------------------------------------------------------

class SpriteSheet {
  SpriteSheet({
    required this.image,
    required this.largeurFrame,
    required this.hauteurFrame,
  })  : colonnes = image.width ~/ largeurFrame,
        lignes = image.height ~/ hauteurFrame;

  final ui.Image image;
  final int largeurFrame;
  final int hauteurFrame;
  final int colonnes;
  final int lignes;

  int get nombreFrames => colonnes * lignes;

  Rect rectDe(int index) {
    if (index < 0 || index >= nombreFrames) {
      throw RangeError.range(index, 0, nombreFrames - 1, 'index');
    }
    return Rect.fromLTWH(
      (index % colonnes) * largeurFrame.toDouble(),
      (index ~/ colonnes) * hauteurFrame.toDouble(),
      largeurFrame.toDouble(),
      hauteurFrame.toDouble(),
    );
  }

  List<int> indexLigne(int ligne) =>
      List<int>.generate(colonnes, (int i) => ligne * colonnes + i);

  /// [x] centre horizontal, [y] ligne des pieds.
  void dessiner(
    Canvas canvas,
    int index,
    double x,
    double y, {
    double echelle = 1,
    bool retourne = false,
  }) {
    final Rect src = rectDe(index);
    final double l = largeurFrame * echelle;
    final double h = hauteurFrame * echelle;
    final Rect dst = Rect.fromLTWH(
      (x - l / 2).roundToDouble(),
      (y - h).roundToDouble(),
      l,
      h,
    );

    if (!retourne) {
      canvas.drawImageRect(image, src, dst, _paintDefaut);
      return;
    }
    canvas.save();
    canvas.translate(dst.right, dst.top);
    canvas.scale(-1, 1);
    canvas.drawImageRect(image, src, Rect.fromLTWH(0, 0, l, h), _paintDefaut);
    canvas.restore();
  }

  static final Paint _paintDefaut = Paint()
    ..filterQuality = FilterQuality.none
    ..isAntiAlias = false;
}

class SpriteAnimation {
  SpriteAnimation({
    required this.frames,
    this.stepTime = 0.1,
    this.loop = true,
  }) : assert(frames.length > 0, 'Une animation doit avoir au moins une frame');

  final List<int> frames;
  final double stepTime;
  final bool loop;

  int _index = 0;
  double _accumulateur = 0;
  bool _terminee = false;

  int get frameCourante => frames[_index];
  int get positionCourante => _index;
  bool get estTerminee => _terminee;

  void avancer(double dt) {
    if (_terminee) return;
    _accumulateur += dt;
    while (_accumulateur >= stepTime) {
      _accumulateur -= stepTime;
      _index++;
      if (_index >= frames.length) {
        if (loop) {
          _index = 0;
        } else {
          _index = frames.length - 1;
          _terminee = true;
          _accumulateur = 0;
          return;
        }
      }
    }
  }

  void reinitialiser() {
    _index = 0;
    _accumulateur = 0;
    _terminee = false;
  }

  /// Positionne l'animation sur le rang [position] de sa séquence.
  void reglerPosition(int position) {
    _index = position % frames.length;
    _accumulateur = 0;
    _terminee = false;
  }
}

// ---------------------------------------------------------------------------
// 3. Les entités : logique pure, aucun Canvas
// ---------------------------------------------------------------------------

enum EtatHeros { repos, marche }

class Heros {
  Heros({required this.planche, required this.x}) {
    animations = <EtatHeros, SpriteAnimation>{
      EtatHeros.repos:
          SpriteAnimation(frames: planche.indexLigne(0), stepTime: 0.30),
      EtatHeros.marche:
          SpriteAnimation(frames: planche.indexLigne(1), stepTime: 0.13),
    };
  }

  final SpriteSheet planche;
  double x;

  late final Map<EtatHeros, SpriteAnimation> animations;

  EtatHeros etat = EtatHeros.repos;
  bool enMarche = false;
  double vitesse = 90;
  int direction = 1;

  SpriteAnimation get animationCourante => animations[etat]!;
  bool get regardeAGauche => direction < 0;

  void basculerMarche() => enMarche = !enMarche;

  void mettreAJour(double dt, double largeurMonde) {
    if (enMarche) {
      x += vitesse * direction * dt;
      const double marge = 40;
      if (x > largeurMonde - marge) {
        x = largeurMonde - marge;
        direction = -1;
      } else if (x < marge) {
        x = marge;
        direction = 1;
      }
    }

    _passerA(enMarche ? EtatHeros.marche : EtatHeros.repos);
    animationCourante.avancer(dt);
  }

  void _passerA(EtatHeros nouveau) {
    if (etat == nouveau) return; // sans cette ligne, animation figée
    etat = nouveau;
    animations[nouveau]!.reinitialiser();
  }
}

class Potion {
  Potion({
    required SpriteSheet planche,
    required this.x,
    required int dephasage,
  }) : animation = SpriteAnimation(
          frames: planche.indexLigne(2),
          stepTime: 0.18,
        ) {
    // Chaque potion démarre à un rang différent : elles ne battent pas
    // à l'unisson.
    animation.reglerPosition(dephasage);
  }

  final double x;
  final SpriteAnimation animation;

  void mettreAJour(double dt) => animation.avancer(dt);
}

// ---------------------------------------------------------------------------
// 4. L'application
// ---------------------------------------------------------------------------

void main() {
  runApp(const AppDonjon());
}

class AppDonjon extends StatelessWidget {
  const AppDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0E0C16),
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
  Duration _precedent = Duration.zero;

  SpriteSheet? _planche;
  Heros? _heros;
  List<Potion> _potions = <Potion>[];
  Size _monde = Size.zero;
  double _dt = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_tick);
    _demarrer();
  }

  Future<void> _demarrer() async {
    // Préchargement : tout est prêt AVANT que la boucle ne démarre.
    final ui.Image image = await creerPlancheDonjon();
    if (!mounted) return;

    final SpriteSheet planche = SpriteSheet(
      image: image,
      largeurFrame: 24,
      hauteurFrame: 24,
    );

    setState(() {
      _planche = planche;
      _heros = Heros(planche: planche, x: 80);
      _potions = <Potion>[
        Potion(planche: planche, x: 120, dephasage: 0),
        Potion(planche: planche, x: 220, dephasage: 1),
        Potion(planche: planche, x: 320, dephasage: 2),
      ];
    });

    _ticker.start();
  }

  void _tick(Duration horodatage) {
    final double dt = (horodatage - _precedent).inMicroseconds / 1000000;
    _precedent = horodatage;
    if (dt <= 0 || dt > 0.25) return;
    if (_monde.width == 0) return;

    _heros?.mettreAJour(dt, _monde.width);
    for (final Potion potion in _potions) {
      potion.mettreAJour(dt);
    }

    setState(() => _dt = dt);
  }

  @override
  void dispose() {
    _ticker.dispose();
    _planche?.image.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final SpriteSheet? planche = _planche;
    final Heros? heros = _heros;
    if (planche == null || heros == null) {
      return const Center(child: CircularProgressIndicator());
    }
    return GestureDetector(
      behavior: HitTestBehavior.opaque,
      onTap: heros.basculerMarche,
      child: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints contraintes) {
          _monde = Size(contraintes.maxWidth, contraintes.maxHeight);
          return CustomPaint(
            painter: PeintreDonjon(
              planche: planche,
              heros: heros,
              potions: _potions,
              dt: _dt,
            ),
            size: Size.infinite,
          );
        },
      ),
    );
  }
}

class PeintreDonjon extends CustomPainter {
  PeintreDonjon({
    required this.planche,
    required this.heros,
    required this.potions,
    required this.dt,
  });

  final SpriteSheet planche;
  final Heros heros;
  final List<Potion> potions;
  final double dt;

  void _texte(Canvas canvas, String contenu, Offset position) {
    final TextPainter tp = TextPainter(
      text: TextSpan(
        text: contenu,
        style: const TextStyle(color: Color(0xFFD8D8E8), fontSize: 13),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    tp.paint(canvas, position);
  }

  void _dessinerDecor(Canvas canvas, Size size, double sol) {
    canvas.drawRect(
      Rect.fromLTWH(0, sol, size.width, size.height - sol),
      Paint()..color = const Color(0xFF241E30),
    );
    canvas.drawRect(
      Rect.fromLTWH(0, sol, size.width, 2),
      Paint()..color = const Color(0xFF463C5C),
    );

    // Les dalles du donjon.
    final Paint joint = Paint()..color = const Color(0xFF1C1728);
    for (double x = 0; x < size.width; x += 48) {
      canvas.drawRect(Rect.fromLTWH(x, sol + 2, 2, size.height - sol), joint);
    }
  }

  @override
  void paint(Canvas canvas, Size size) {
    final double sol = size.height * 0.75;

    _dessinerDecor(canvas, size, sol);

    // Les potions, posées au sol, derrière le héros.
    for (final Potion potion in potions) {
      planche.dessiner(
        canvas,
        potion.animation.frameCourante,
        potion.x,
        sol,
        echelle: 3,
      );
    }

    // Le héros.
    planche.dessiner(
      canvas,
      heros.animationCourante.frameCourante,
      heros.x,
      sol,
      echelle: 4,
      retourne: heros.regardeAGauche,
    );

    // Le panneau d'information.
    final double fps = dt > 0 ? 1 / dt : 0;
    final String framesPotions =
        potions.map((Potion p) => p.animation.positionCourante).join(', ');

    _texte(canvas, 'DONJON DE DART', const Offset(20, 24));
    _texte(canvas, 'FPS            : ${fps.toStringAsFixed(0)}',
        const Offset(20, 46));
    _texte(canvas, 'etat du heros  : ${heros.etat.name}', const Offset(20, 64));
    _texte(
      canvas,
      'frame du heros : ${heros.animationCourante.frameCourante}',
      const Offset(20, 82),
    );
    _texte(canvas, 'potions (rang) : $framesPotions', const Offset(20, 100));
    _texte(canvas, 'touchez pour marcher / s\'arreter',
        const Offset(20, 124));
  }

  @override
  bool shouldRepaint(PeintreDonjon oldDelegate) => true;
}
```

**Résultat :**

```text
DONJON DE DART
FPS            : 60
etat du heros  : marche
frame du heros : 6
potions (rang) : 2, 3, 0

touchez pour marcher / s'arreter

Un sol de donjon dallé occupe le quart inférieur de l'écran.
Trois potions rouges y sont posées : le niveau de liquide monte et descend
dans chaque fiole, mais décalé d'une potion à l'autre.
Le héros en tunique bleue traverse l'écran à 90 pixels par seconde, jambes
et bras en mouvement. Arrivé au bord, il se retourne et repart.
Une pression l'arrête : il repasse en respiration lente, sans saut visuel.
Une nouvelle pression le relance.
```

**Explication :** cette dernière correction assemble tout le chapitre en une seule planche de 96 sur 72 pixels — soit moins de 7 000 pixels pour animer quatre entités.

**Une planche, plusieurs personnages.** Les trois lignes ne sont pas trois images : `indexLigne(0)`, `indexLigne(1)` et `indexLigne(2)` découpent la même texture. Le héros et les potions partagent le même `ui.Image` et le même `Paint`. C'est exactement l'argument de la section 22.15, et c'est ce qui rendra le passage à Flame (chapitre 29) naturel.

**Le déphasage.** `reglerPosition(dephasage)` place chaque potion à un rang différent de la même séquence. Trois potions, trois rangs, aucun code d'animation supplémentaire. Sans cela, les trois fioles battraient au même instant et l'œil verrait un clignotement collectif, très artificiel.

**L'ordre de dessin fait la profondeur.** Les potions sont peintes avant le héros, donc il passe devant elles. Le `Canvas` n'a aucune notion de plan : c'est l'ordre des appels qui décide, comme au chapitre 21.

**L'arrondi de position.** `dessiner` applique `roundToDouble()` au rectangle de destination. Le héros avance de 1,5 pixel par frame ; sans arrondi, ses colonnes de pixels tomberaient entre deux pixels écran et sembleraient vibrer. C'est un détail que l'on ne voit qu'en mouvement, et qui distingue un rendu pixel art propre d'un rendu approximatif.

**La séparation logique / rendu.** `Heros.mettreAJour` et `Potion.mettreAJour` ne connaissent pas le `Canvas`. `PeintreDonjon` ne décide de rien : il lit un état et l'affiche. Vous pouvez, dès maintenant, écrire un test unitaire sur `Heros` sans lancer Flutter. C'est cette discipline qui rendra les chapitres 23 à 26 confortables.

**Et le jour où vous aurez de vraies images ?** Une seule ligne change :

```dart
final ui.Image image = await creerPlancheDonjon();
```

devient :

```dart
final ui.Image image = await chargerImage('assets/images/donjon.png');
```

Tout le reste — `SpriteSheet`, `SpriteAnimation`, les états, le rendu — demeure identique. C'est la promesse faite au début du chapitre, et elle est tenue.

---

## Et maintenant ?

Votre personnage sait s'afficher, se découper, s'animer et changer d'état. Mais il ne se déplace encore que sur commande directe : `x += vitesse * dt`, et une inversion brutale de direction au bord de l'écran.

Un vrai jeu réclame davantage. Un héros accélère, il ne passe pas instantanément de 0 à 90 pixels par seconde. Il retombe quand il saute. Il glisse un peu sur la glace et s'arrête vite sur la pierre. Il est repoussé quand un gobelin le frappe.

Tout cela s'exprime avec une seule notion : le **vecteur vitesse**, et une seule règle : la position change selon la vitesse, la vitesse change selon l'accélération.

Le chapitre 23 pose ces bases. Vous y écrirez une classe `Vecteur2`, vous appliquerez la gravité, vous ferez sauter le héros du Donjon de Dart, et vous comprendrez pourquoi l'ordre des trois lignes de l'intégration d'Euler change tout. L'animation de saut que vous savez déjà déclencher trouvera enfin le mouvement qui lui correspond.

Chapitre suivant : [23-PARTIE-2A—MOUVEMENT-VÉLOCITÉ-ET-PHYSIQUE-SIMPLE.md](./23-PARTIE-2A—MOUVEMENT-VÉLOCITÉ-ET-PHYSIQUE-SIMPLE.md)
