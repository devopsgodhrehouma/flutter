# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 34 — AUDIO, TILED ET PHYSIQUE AVANCÉE

> **Niveau :** intermédiaire
> **Durée estimée :** 9 h
> **Pré-requis :** chapitres 27 à 33 (installation de Flame, composants, sprites, entrées, caméra, collisions, effets), chapitre 25 (tilemap en `List<String>`), chapitre 26 (services et architecture), chapitres 16 et 17 de la PARTIE 1A (`pubspec.yaml`, modélisation de données)
> **Version de Flame utilisée :** **1.38.0** — `flame_audio` **2.12.2**, `flame_tiled` **3.1.2**, `flame_forge2d` **0.19.3+7**
> **Ce que vous saurez faire à la fin :** faire sonner le « Donjon de Dart », charger un niveau dessiné dans Tiled, et décider en connaissance de cause si votre jeu a besoin ou non d'un moteur physique complet.

---

## 34.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi le son représente la moitié de la sensation de jeu ;
- installer `flame_audio` et déclarer correctement le dossier `assets/audio/` ;
- choisir entre `mp3`, `wav` et `ogg` selon la plateforme visée ;
- jouer un effet sonore avec `FlameAudio.play()` et régler son `volume` ;
- précharger les sons avec `FlameAudio.audioCache.load()` et `loadAll()` ;
- diagnostiquer et supprimer la **latence du premier son** ;
- piloter la musique de fond avec `FlameAudio.bgm` : `initialize`, `play`, `pause`, `resume`, `stop`, `dispose` ;
- réagir au passage de l'application en arrière-plan avec `lifecycleStateChange` ;
- jouer plusieurs occurrences du même son sans coupure grâce à un `AudioPool` ;
- écrire un **service audio centralisé** avec réglages séparés musique / effets ;
- trouver des sons libres de droits et vérifier leur licence ;
- écrire un jeu qui fonctionne **même sans aucun fichier audio** ;
- justifier l'usage d'un éditeur de niveaux plutôt qu'un tableau en dur ;
- installer Tiled, créer un tileset, des calques de tuiles et des calques d'objets ;
- lire un fichier `.tmx` à la main et comprendre ce qu'il contient ;
- installer `flame_tiled` et charger une carte avec `TiledComponent.load()` ;
- ajouter la carte au monde et régler la caméra en conséquence ;
- récupérer un calque avec `tileMap.getLayer<T>()` ;
- parcourir les objets d'un `ObjectGroup` pour créer le point d'apparition, les coffres, les portes et les ennemis ;
- lire les **propriétés personnalisées** d'un objet avec `properties.getValue<T>()` ;
- générer automatiquement les hitbox d'un niveau à partir d'un calque d'objets ;
- revenir à la carte en `List<String>` du chapitre 25 quand Tiled n'est pas disponible ;
- dire quand la détection de collision intégrée de Flame ne suffit plus ;
- expliquer ce qu'est Forge2D et d'où il vient ;
- installer `flame_forge2d` et écrire un `Forge2DGame` complet ;
- comprendre le facteur `zoom` et raisonner en **mètres**, pas en pixels ;
- construire un `BodyComponent` avec `BodyDef` et `FixtureDef` ;
- distinguer corps statique, dynamique et kinématique ;
- régler densité, friction et restitution ;
- citer les principaux **joints** et leur usage ;
- expliquer pourquoi le « Donjon de Dart » n'utilisera **pas** Forge2D ;
- choisir votre moteur de collision à l'aide d'un tableau de décision ;
- récapituler tout ce que la PARTIE 2B a apporté ;
- écrire le `pubspec.yaml` final du projet.

---

## 34.1 — Le son fait la moitié du jeu

Coupez le son d'un jeu que vous aimez. Jouez cinq minutes. Vous constaterez trois choses.

**Vous frappez dans le vide.** Un coup d'épée sans bruit ne donne aucune information : le joueur ne sait plus si l'attaque a porté, s'il a touché le décor ou l'ennemi. Le son est un **retour d'information**, exactement comme un clignotement rouge ou une barre de vie qui descend.

**Le rythme disparaît.** La musique de fond dicte l'énergie d'une scène. Une salle de donjon avec une nappe lente n'est pas la même salle avec une percussion rapide, alors que les pixels sont identiques.

**Le jeu paraît inachevé.** C'est le jugement le plus dur, et il est immédiat. Un prototype silencieux est perçu comme un prototype ; le même prototype avec trois bruitages est perçu comme un jeu.

Récapitulons ce que le son apporte, et à quel moment.

| Rôle du son | Exemple dans le « Donjon de Dart » | Conséquence si absent |
| --- | --- | --- |
| Confirmer une action | bruit d'épée quand le héros attaque | le joueur martèle le bouton, croyant que ça ne marche pas |
| Confirmer un succès | tintement quand une pièce est ramassée | la récompense n'est pas ressentie |
| Avertir d'un danger | grognement du gobelin qui repère le héros | le joueur est surpris sans avoir été prévenu |
| Signaler une perte | son grave quand le héros perd une vie | la perte de vie passe inaperçue en pleine action |
| Installer une ambiance | musique de fond de la salle du boss | toutes les salles se ressemblent |
| Récompenser | jingle de victoire | la fin du niveau n'est pas un événement |

> **À retenir.** Le son n'est pas une décoration ajoutée à la fin. C'est un canal de communication avec le joueur, au même titre que l'affichage. On le conçoit en même temps que le reste.

Une règle de dosage, tirée de la pratique : **un effet sonore par événement significatif, pas un de plus**. Un jeu où chaque pas produit un bruit devient épuisant en trois minutes. Choisissez cinq à huit sons pour tout un jeu de niveau débutant, et travaillez-les.

---

## 34.2 — Installer `flame_audio`

Flame ne sait pas jouer de son tout seul. Le moteur se concentre sur la boucle de jeu et le rendu. La lecture audio est déléguée à un **paquet-pont** : `flame_audio`, qui enveloppe le paquet Flutter `audioplayers`.

La chaîne complète est donc : votre jeu → `flame_audio` (conventions Flame : dossier `assets/audio/`, cache global, musique de fond) → `audioplayers` (ouverture d'un lecteur, décodage) → système d'exploitation. La comprendre évite beaucoup de confusion : quand un son ne sort pas, la panne est presque toujours **en haut** (mauvais chemin d'asset) ou **en bas** (format non supporté par la plateforme), rarement au milieu.

L'installation se fait en une ligne dans le `pubspec.yaml`.

```yaml
dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0
  flame_audio: ^2.12.2
```

Puis, dans un terminal placé à la racine du projet :

```text
flutter pub get
```

**Résultat attendu :**

```text
Resolving dependencies...
+ audioplayers 6.2.0
+ flame_audio 2.12.2
...
Changed 7 dependencies!
```

Une seule ligne d'import suffit ensuite dans le code :

```dart
import 'package:flame_audio/flame_audio.dart';
```

> **Remarque.** `flame_audio` déclare `flame: ^1.38.0`. Si vous voyez une erreur de résolution de version, c'est que votre `flame` est trop ancien. Alignez les deux : `flame: ^1.38.0` et `flame_audio: ^2.12.2`.

---

## 34.3 — Le dossier `assets/audio/` et le `pubspec.yaml`

`flame_audio` cherche vos fichiers dans un dossier précis : **`assets/audio/`**. C'est la valeur par défaut du préfixe utilisé par `FlameAudio`. Comme pour les images, vous ne passez **jamais** le chemin complet aux méthodes.

Arborescence attendue :

```text
  donjon_de_dart/
  ├── assets/
  │   ├── audio/
  │   │   ├── epee.mp3
  │   │   ├── piece.mp3
  │   │   ├── degat.mp3
  │   │   ├── porte.mp3
  │   │   └── musique_donjon.mp3
  │   ├── images/
  │   │   └── ...
  │   └── tiles/
  │       └── ...
  ├── lib/
  │   └── main.dart
  └── pubspec.yaml
```

Déclaration dans le `pubspec.yaml` :

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/audio/
```

Deux écritures sont possibles : `- assets/audio/` embarque tout le dossier (sans les sous-dossiers), tandis que la liste fichier par fichier (`- assets/audio/epee.mp3`, …) est plus verbeuse mais n'embarque que ce que vous utilisez, ce qui réduit la taille de l'application finale.

Attention à un détail syntaxique qui fait perdre du temps : la barre oblique finale est **obligatoire** pour un dossier.

| Écriture | Effet |
| --- | --- |
| `- assets/audio/` | tout le dossier `assets/audio` est embarqué |
| `- assets/audio` | interprété comme un **fichier** nommé `audio` : erreur de build |
| `- assets/audio/*` | l'étoile n'est pas supportée : erreur |

Et voici la règle d'appel, la source d'erreur numéro un du chapitre :

```dart
// CORRECT : chemin relatif à assets/audio/
FlameAudio.play('epee.mp3');

// FAUX : Flame ajoute son préfixe, le chemin devient
// assets/audio/assets/audio/epee.mp3
FlameAudio.play('assets/audio/epee.mp3');
```

**Résultat de la version fautive :**

```text
Unable to load asset: "assets/audio/assets/audio/epee.mp3"
```

Ce message est explicite : lisez-le en entier, le chemin dupliqué saute aux yeux.

---

## 34.4 — Formats : mp3, wav, ogg, et la compatibilité Web

La documentation de `flame_audio` recommande trois formats : **MP3**, **OGG** et **WAV**. Ils ne servent pas au même usage.

| Format | Compression | Taille typique (1 s) | Latence de décodage | Usage conseillé |
| --- | --- | --- | --- | --- |
| `wav` | aucune (PCM brut) | ~ 176 Ko | quasi nulle | effets très courts sur ordinateur |
| `mp3` | avec pertes | ~ 16 Ko | faible | usage général, musique et effets |
| `ogg` | avec pertes | ~ 14 Ko | faible | alternative libre au mp3 |

Le raisonnement à tenir est simple.

**Pour la musique**, vous voulez de la compression : un morceau de deux minutes en `wav` pèse 20 Mo, contre 2 Mo en `mp3`. Personne ne téléchargera votre jeu avec 60 Mo de musique non compressée.

**Pour les effets courts**, le `wav` évite le décodage, donc réduit la latence. Mais un effet de 0,3 seconde en `mp3` ne pèse que quelques kilo-octets et se décode instantanément une fois en cache. En pratique, le `mp3` suffit partout.

Le point délicat est le **Web**. Le navigateur, et non votre code, décide des formats qu'il accepte. La situation actuelle, à vérifier sur vos navigateurs cibles :

| Format | Chrome | Firefox | Safari |
| --- | --- | --- | --- |
| `mp3` | oui | oui | oui |
| `wav` | oui | oui | oui |
| `ogg` (Vorbis) | oui | oui | variable selon les versions |

Conclusion pratique pour ce cours : **utilisez le `mp3`**. C'est le seul format accepté partout sans discussion, y compris sur les vieux Safari.

Le second piège du Web n'est pas un format mais une **politique**. Les navigateurs interdisent de jouer un son avant que l'utilisateur ait interagi avec la page. C'est une protection contre les publicités sonores.

```text
  Chargement de la page
        │
        ▼
  Le jeu appelle FlameAudio.play('musique.mp3')
        │
        ▼
  Le navigateur BLOQUE  →  aucune erreur visible, aucun son
        │
        ▼
  Le joueur clique sur « Jouer »
        │
        ▼
  FlameAudio.play('musique.mp3')  →  le son sort
```

La parade est architecturale et vous la connaissez déjà : **la musique démarre sur l'appui du bouton « Jouer » du menu, pas dans `onLoad`**. Vous mettrez cela en place au chapitre 35, quand vous construirez le menu principal.

---

## 34.5 — `FlameAudio.play()`

Voici la méthode la plus utilisée du paquet.

```dart
FlameAudio.play('epee.mp3');
```

Elle joue le fichier une fois et rend la main immédiatement. Elle renvoie un `Future<AudioPlayer>` : le lecteur du paquet `audioplayers`, que vous pouvez conserver si vous voulez arrêter le son en cours de route.

Les quatre méthodes de lecture disponibles :

```dart
// Effet court, joué une fois.
FlameAudio.play('epee.mp3');

// Effet court, joué en boucle sans blanc entre les répétitions.
FlameAudio.loop('torche.mp3');

// Fichier long, joué une fois.
FlameAudio.playLongAudio('cinematique.mp3');

// Fichier long, joué en boucle.
FlameAudio.loopLongAudio('musique_donjon.mp3');
```

La différence entre `play` et `playLongAudio` mérite une explication, car le nom prête à confusion. `play` charge tout le fichier en mémoire avant de le jouer : c'est instantané mais coûteux pour un fichier de plusieurs minutes. `playLongAudio` diffuse le fichier au fil de la lecture : c'est économe en mémoire, mais le démarrage peut provoquer une saccade visible dans le framerate.

Règle simple : **effets courts avec `play`, musique avec `FlameAudio.bgm`** (section 34.9), et `playLongAudio` seulement pour une cinématique ponctuelle.

Voici un premier exemple complet et exécutable. Il fonctionne même si vous n'avez aucun fichier audio : le `try` intercepte l'échec de chargement et le jeu continue.

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame_audio/flame_audio.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuSon()));
}

class JeuSon extends FlameGame {
  late final TextComponent journal;
  int coups = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    journal = TextComponent(
      text: 'Touchez le carré pour frapper.',
      position: Vector2(12, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 16, color: Color(0xFFEEEEEE)),
      ),
    );
    await camera.viewport.add(journal);
    await world.add(CibleSonore());
  }

  Future<void> frapper() async {
    coups++;
    try {
      await FlameAudio.play('epee.mp3');
      journal.text = 'Coups portés : $coups';
    } catch (e) {
      journal.text = 'Coups portés : $coups (son indisponible)';
    }
  }
}

class CibleSonore extends RectangleComponent
    with TapCallbacks, HasGameReference<JeuSon> {
  CibleSonore()
      : super(
          position: Vector2(120, 140),
          size: Vector2.all(96),
          paint: Paint()..color = const Color(0xFFB5563C),
        );

  @override
  void onTapDown(TapDownEvent event) => game.frapper();
}
```

**Résultat (avec le fichier `assets/audio/epee.mp3`) :**

```text
Coups portés : 1
Coups portés : 2
Coups portés : 3
```

**Résultat (sans le fichier) :**

```text
Coups portés : 1 (son indisponible)
```

Le jeu ne plante pas. C'est exactement le comportement que vous devez viser tout au long de ce chapitre.

---

## 34.6 — Le volume

`play` et `loop` acceptent un paramètre nommé `volume`, de type `double`, dont la valeur par défaut est `1.0`.

```dart
FlameAudio.play('epee.mp3', volume: 0.8);
FlameAudio.play('piece.mp3', volume: 0.5);
FlameAudio.loop('torche.mp3', volume: 0.15);
```

L'échelle va de `0.0` (silence) à `1.0` (volume du fichier tel qu'il a été enregistré). Vous ne pouvez pas monter au-delà de `1.0` de façon fiable : si un son est trop faible, corrigez le fichier, pas le code.

Trois erreurs de débutant sur le volume.

**Erreur 1 : tout jouer à `1.0`.** Vos sons ne sont pas normalisés entre eux. Un bruitage de pièce téléchargé sur un site et un bruitage d'épée téléchargé sur un autre n'ont aucune raison d'avoir la même intensité perçue. Réglez chaque son individuellement, à l'oreille.

**Erreur 2 : la musique au même niveau que les effets.** La musique doit rester **sous** les effets, sinon elle les masque. Un rapport de départ raisonnable :

| Catégorie | Volume conseillé |
| --- | --- |
| Musique de fond | `0.25` à `0.40` |
| Effets d'action (épée, saut) | `0.70` à `1.00` |
| Effets d'ambiance (torche, vent) | `0.10` à `0.25` |
| Effets d'interface (bouton) | `0.40` à `0.60` |

**Erreur 3 : appliquer le réglage utilisateur à la main partout.** Si le joueur baisse le volume dans les options, il ne faut pas multiplier à la main dans chaque appel. Vous centraliserez cela en 34.13.

Petit calcul à retenir : le volume perçu ne suit pas une échelle linéaire. Passer de `1.0` à `0.5` ne divise pas la sensation par deux, mais la réduit d'environ un tiers. C'est pour cela qu'un curseur de volume doit descendre assez bas pour être utile.

```dart
// Le volume final combine le réglage du joueur et celui du son.
final double volumeJoueur = 0.6;   // curseur des options
final double volumeDuSon = 0.8;    // équilibrage de ce bruitage précis

FlameAudio.play('epee.mp3', volume: volumeJoueur * volumeDuSon); // 0.48
```

---

## 34.7 — Précharger avec `FlameAudio.audioCache.loadAll()`

Un fichier audio doit être lu depuis le paquet de l'application, décodé, puis envoyé au système. Tant que ce travail n'est pas fait, aucun son ne sort. `flame_audio` conserve les fichiers déjà chargés dans un **cache** accessible par `FlameAudio.audioCache`.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  // Un seul fichier.
  await FlameAudio.audioCache.load('epee.mp3');

  // Plusieurs fichiers d'un coup.
  await FlameAudio.audioCache.loadAll([
    'epee.mp3',
    'piece.mp3',
    'degat.mp3',
    'porte.mp3',
  ]);
}
```

Et la libération, quand un niveau se termine et que ses sons ne serviront plus :

```dart
// Libérer un fichier précis.
FlameAudio.audioCache.clear('porte.mp3');

// Vider tout le cache audio.
FlameAudio.audioCache.clearCache();
```

Le `await` sur `loadAll` est important. Sans lui, `onLoad` se termine avant la fin du chargement, et le premier son du jeu subit la latence que vous cherchiez précisément à éviter.

C'est le même raisonnement que pour `Sprite.load` au chapitre 29, et que pour les `Future` du chapitre 15 de la PARTIE 1A.

> **À retenir.** Tout ce qui doit être prêt avant la première frame se charge dans `onLoad`, avec `await`. Le reste se charge au moment où on en a besoin, et provoque une saccade.

---

## 34.8 — La latence du premier son et comment l'éviter

Voici le symptôme, décrit tel que les élèves le rapportent : « le premier coup d'épée ne fait pas de bruit, les suivants oui ». Ou : « le son arrive une demi-seconde après le coup, seulement la première fois ».

La cause est mécanique.

```text
  SANS PRÉCHARGEMENT              AVEC PRÉCHARGEMENT DANS onLoad
  ────────────────────────        ──────────────────────────────
  appui           t = 0 ms        (lecture + décodage à l'écran
  lecture fichier     +2 ms         de chargement)
  décodage mp3       +58 ms       appui           t = 0 ms
  création lecteur   +30 ms       ► le son sort       +4 ms
  ► le son sort     140 ms
```

Un retard de 140 millisecondes est parfaitement audible. Le seuil au-delà duquel un joueur perçoit une désynchronisation entre l'action et le son se situe autour de **50 ms**.

Trois parades, à combiner.

**Parade 1 — précharger dans `onLoad`.** C'est la section précédente. Elle règle 90 % des cas.

**Parade 2 — jouer un son inaudible au démarrage.** Sur certaines plateformes, l'ouverture du canal audio du système coûte elle aussi du temps, indépendamment du fichier. Jouer un son à volume nul « réveille » ce canal.

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();
  await FlameAudio.audioCache.loadAll(['epee.mp3', 'piece.mp3']);

  // Réveille la chaîne audio du système sans être entendu.
  await FlameAudio.play('epee.mp3', volume: 0.0);
}
```

**Parade 3 — utiliser un `AudioPool` pour les sons très fréquents.** Un pool garde des lecteurs déjà ouverts et prêts. Voir 34.12.

Résumons les symptômes et leurs causes, parce que vous les rencontrerez tous.

| Symptôme | Cause probable | Correction |
| --- | --- | --- |
| Le premier son est en retard | fichier non préchargé | `audioCache.loadAll` dans `onLoad` |
| Le premier son ne sort pas du tout (Web) | politique d'interaction du navigateur | démarrer le son après un clic |
| Tous les sons sont en retard | fichier trop long joué avec `play` | `playLongAudio` ou fichier plus court |
| Le son se coupe quand on le rejoue | un seul lecteur réutilisé | `AudioPool` |
| Le framerate chute au premier son | décodage pendant une frame | préchargement |

---

## 34.9 — `FlameAudio.bgm` : la musique de fond

La musique de fond n'est pas un effet sonore long. Elle a des besoins propres : elle doit boucler, survivre aux changements d'écran, se mettre en pause quand le joueur reçoit un appel téléphonique, et reprendre ensuite. `flame_audio` fournit un objet dédié : **`FlameAudio.bgm`**, de type `Bgm`.

Le point le plus important, et celui qu'on oublie : **`initialize()` doit être appelé avant tout usage**, à un moment où l'application Flutter existe déjà. La documentation recommande explicitement de le faire dans `onLoad`.

```dart
import 'package:flame/game.dart';
import 'package:flame_audio/flame_audio.dart';

class DonjonDeDart extends FlameGame {
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // 1. Enregistre l'observateur du cycle de vie de l'application.
    FlameAudio.bgm.initialize();

    // 2. Démarre la musique, en boucle, à volume réduit.
    FlameAudio.bgm.play('musique_donjon.mp3', volume: 0.25);
  }

  @override
  void onRemove() {
    // 3. Libère les ressources quand le jeu disparaît.
    FlameAudio.bgm.dispose();
    super.onRemove();
  }
}
```

Que fait `initialize()` exactement ? Il inscrit `Bgm` comme observateur du cycle de vie de l'application Flutter. C'est grâce à cela que la musique se met automatiquement en pause quand l'application passe en arrière-plan, et reprend au retour.

Sans `initialize()`, la musique joue quand même, mais l'observateur n'est pas branché : votre jeu continuera de chanter dans la poche du joueur.

| Appel manquant | Conséquence |
| --- | --- |
| `initialize()` oublié | la musique continue en arrière-plan |
| `dispose()` oublié | les ressources audio ne sont pas libérées |
| `play()` sans `initialize()` | la musique sort, mais sans gestion du cycle de vie |

> **Remarque.** `FlameAudio.bgm` est un objet **global**. Il n'y a qu'une seule musique de fond à la fois dans toute l'application. Appeler `play` avec un autre fichier remplace la musique en cours ; c'est exactement ce que vous voudrez pour passer de la musique du menu à celle du donjon.

---

## 34.10 — `bgm.play()`, `pause()`, `resume()`, `stop()`

Les cinq méthodes de `Bgm` couvrent tout le cycle de vie d'une musique.

```dart
// Démarrer (ou remplacer) la musique de fond.
FlameAudio.bgm.play('musique_donjon.mp3', volume: 0.25);

// Suspendre : la position de lecture est conservée.
FlameAudio.bgm.pause();

// Reprendre là où on s'était arrêté.
FlameAudio.bgm.resume();

// Arrêter : la position est perdue, le prochain play repart du début.
FlameAudio.bgm.stop();

// Libérer définitivement les ressources.
FlameAudio.bgm.dispose();
```

La différence entre `pause` et `stop` est la seule chose à mémoriser : `pause()` conserve la position de lecture, donc `resume()` reprend à 0:42 ; `stop()` la perd, donc le prochain `play()` repart de 0:00.

Traduction en règles de jeu :

| Situation | Méthode |
| --- | --- |
| Le joueur ouvre le menu pause | `pause()` |
| Le joueur ferme le menu pause | `resume()` |
| Le joueur meurt et revient au menu | `stop()` puis `play('musique_menu.mp3')` |
| Le joueur entre dans la salle du boss | `play('musique_boss.mp3')` |
| Le jeu est retiré de l'arbre | `dispose()` |

Voici le branchement sur la machine à états du chapitre 26, directement réutilisable au chapitre 35.

```dart
enum EtatJeu { menu, enJeu, pause, gameOver }

class DonjonDeDart extends FlameGame {
  EtatJeu _etat = EtatJeu.menu;

  void changerEtat(EtatJeu nouvel) {
    if (nouvel == _etat) return;
    final ancien = _etat;
    _etat = nouvel;

    switch (nouvel) {
      case EtatJeu.menu:
        FlameAudio.bgm.stop();
        FlameAudio.bgm.play('musique_menu.mp3', volume: 0.25);
      case EtatJeu.enJeu:
        if (ancien == EtatJeu.pause) {
          FlameAudio.bgm.resume();
        } else {
          FlameAudio.bgm.stop();
          FlameAudio.bgm.play('musique_donjon.mp3', volume: 0.25);
        }
      case EtatJeu.pause:
        FlameAudio.bgm.pause();
      case EtatJeu.gameOver:
        FlameAudio.bgm.stop();
    }
  }
}
```

**Explication du cas subtil.** Le passage `pause → enJeu` appelle `resume()`, alors que `menu → enJeu` appelle `stop()` puis `play()`. Sans distinguer les deux, revenir de la pause redémarrerait la musique au début à chaque fois, ce qui est très désagréable. C'est pour cela que `_adapterMusique` reçoit l'**ancien** état en plus du nouveau.

---

## 34.11 — Mettre la musique en pause quand l'application passe en arrière-plan

`FlameAudio.bgm` gère déjà ce cas, à condition d'avoir appelé `initialize()`. Mais vos **effets sonores**, votre `AudioPool` et surtout votre **boucle de jeu** ne sont pas concernés. Un jeu qui continue de tourner quand le joueur consulte un message perdra des vies tout seul.

Flame expose un point d'entrée dédié sur la classe `Game` :

```dart
void lifecycleStateChange(AppLifecycleState state);
```

Il est appelé à chaque changement d'état de l'application. Les valeurs d'`AppLifecycleState` viennent de `package:flutter/widgets.dart`.

| Valeur | Signification | Ce qu'il faut faire |
| --- | --- | --- |
| `resumed` | l'application est au premier plan et interactive | reprendre le jeu et la musique |
| `inactive` | transition, l'application perd le focus | ne rien faire de définitif |
| `paused` | l'application est en arrière-plan | mettre en pause jeu et son |
| `hidden` | l'application n'est plus visible | comme `paused` |
| `detached` | la vue est détachée du moteur | libérer les ressources |

Voici l'implémentation complète pour le « Donjon de Dart ».

```dart
import 'package:flame/game.dart';
import 'package:flame_audio/flame_audio.dart';
import 'package:flutter/widgets.dart';

class DonjonDeDart extends FlameGame {
  bool _pauseParLeJoueur = false;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    FlameAudio.bgm.initialize();
    FlameAudio.bgm.play('musique_donjon.mp3', volume: 0.25);
  }

  /// Pause déclenchée par le joueur (bouton PAUSE du HUD).
  void basculerPause() {
    _pauseParLeJoueur = !_pauseParLeJoueur;
    if (_pauseParLeJoueur) {
      pauseEngine();
      FlameAudio.bgm.pause();
    } else {
      resumeEngine();
      FlameAudio.bgm.resume();
    }
  }

  @override
  void lifecycleStateChange(AppLifecycleState state) {
    super.lifecycleStateChange(state);

    switch (state) {
      case AppLifecycleState.paused:
      case AppLifecycleState.hidden:
      case AppLifecycleState.inactive:
        pauseEngine();
        FlameAudio.bgm.pause();
      case AppLifecycleState.resumed:
        // On ne reprend PAS si le joueur avait lui-même mis en pause.
        if (!_pauseParLeJoueur) {
          resumeEngine();
          FlameAudio.bgm.resume();
        }
      case AppLifecycleState.detached:
        FlameAudio.bgm.stop();
    }
  }

  @override
  void onRemove() {
    FlameAudio.bgm.dispose();
    super.onRemove();
  }
}
```

**Explication du drapeau `_pauseParLeJoueur`.** Imaginez la séquence : le joueur ouvre le menu pause, puis répond à un message, puis revient au jeu. Sans le drapeau, le retour au premier plan appellerait `resumeEngine()` et relancerait le jeu **alors que le menu pause est encore affiché**. Le héros se ferait tuer derrière un menu. Le drapeau distingue « pause voulue » et « pause subie ».

| pause joueur | arrière-plan | moteur |
| --- | --- | --- |
| non | non | actif |
| non | oui | en pause |
| oui | non | en pause |
| oui | oui | en pause |

Le moteur ne tourne que si **aucune** des deux sources ne demande la pause. C'est un ET logique, et c'est exactement ce que le drapeau implémente.

---

## 34.12 — Plusieurs sons simultanés (`AudioPool`)

Testez ceci : appelez `FlameAudio.play('epee.mp3')` cinq fois en une demi-seconde. Selon la plateforme, vous obtiendrez soit cinq sons superposés, soit un son coupé net et relancé, soit une latence croissante. Le comportement n'est pas garanti, car chaque appel doit obtenir un lecteur du système.

`AudioPool` résout le problème : il maintient un **groupe de lecteurs déjà ouverts et préchargés avec le même son**. Quand vous demandez une lecture, il vous en prête un ; quand la lecture s'achève, le lecteur retourne dans le groupe.

```text
  AudioPool('epee.mp3', minPlayers: 3, maxPlayers: 6)

  ┌──────────────────────────────────────────────┐
  │  lecteur 1  lecteur 2  lecteur 3   (au repos) │
  └──────────────────────────────────────────────┘
        │
   start()  ──►  lecteur 1 joue
   start()  ──►  lecteur 2 joue
   start()  ──►  lecteur 3 joue
   start()  ──►  aucun libre : un 4e lecteur est créé
        │
   fin du son du lecteur 1  ──►  lecteur 1 revient au repos
```

Trois façons de créer un pool. La plus simple utilise le cache global de Flame :

```dart
import 'package:flame_audio/flame_audio.dart';

late AudioPool poolEpee;

Future<void> chargerSons() async {
  poolEpee = await FlameAudio.createPool(
    'epee.mp3',
    minPlayers: 3,
    maxPlayers: 6,
  );
}
```

Deux autres fabriques existent si vous devez préciser la source ou le cache : `AudioPool.create(source: AssetSource('epee.mp3'), minPlayers: 1, maxPlayers: 4, audioCache: FlameAudio.audioCache)` et `AudioPool.createFromAsset(path: 'epee.mp3', minPlayers: 1, maxPlayers: 4)`.

L'usage se fait avec `start()`, qui renvoie une **fonction d'arrêt** :

```dart
// Lecture au volume par défaut (1.0).
final stop = await poolEpee.start();

// Lecture à volume réglé.
await poolEpee.start(volume: 0.7);

// Arrêt anticipé, si nécessaire.
await stop();
```

Et la libération :

```dart
await poolEpee.dispose();
```

Voici l'intégration dans un jeu. Le pool est optionnel : si le fichier manque, le jeu tourne quand même.

```dart
class JeuPool extends FlameGame {
  AudioPool? _poolEpee;
  int coups = 0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    try {
      _poolEpee = await FlameAudio.createPool(
        'epee.mp3',
        minPlayers: 3,
        maxPlayers: 6,
      );
    } catch (_) {
      _poolEpee = null; // aucun fichier : le jeu reste jouable
    }
    await world.add(BoutonFrappe());
  }

  Future<void> frapper() async {
    coups++;
    await _poolEpee?.start(volume: 0.7);
  }

  @override
  void onRemove() {
    _poolEpee?.dispose();
    super.onRemove();
  }
}

class BoutonFrappe extends RectangleComponent
    with TapCallbacks, HasGameReference<JeuPool> {
  BoutonFrappe()
      : super(
          position: Vector2(100, 140),
          size: Vector2(140, 90),
          paint: Paint()..color = const Color(0xFF6A8CAF),
        );

  @override
  void onTapDown(TapDownEvent event) => game.frapper();
}
```

**Quand faut-il un pool ?** Réponse courte : quand le même son peut se déclencher plus d'une fois par seconde. Tir, pas de course, pièces ramassées en série, impacts. Pour un son de porte ou de fin de niveau, `FlameAudio.play` suffit largement.

---

## 34.13 — Un gestionnaire de son centralisé

Vous avez maintenant tous les outils. Les disperser dans le code serait une erreur d'architecture, et le chapitre 26 vous a montré pourquoi : un jour vous voudrez couper le son, et vous devrez chercher les appels dans quarante fichiers.

Créez un **service**, exactement comme le service de données du chapitre 26.

Cahier des charges :

1. deux réglages indépendants : volume musique et volume effets ;
2. deux interrupteurs : musique activée / désactivée, effets activés / désactivés ;
3. un point d'entrée unique par événement de jeu (`jouerEpee()`, `jouerPiece()`, …) ;
4. **fonctionner sans aucun fichier audio** ;
5. aucune dépendance à `FlameGame` : le service doit être testable en console.

```dart
// lib/services/service_audio.dart
import 'package:flame_audio/flame_audio.dart';

/// Identifiants logiques des effets du jeu.
/// Le reste du code ne connaît QUE cet enum, jamais un nom de fichier.
enum Effet { epee, piece, degat, porte, boss, victoire }

class ServiceAudio {
  ServiceAudio({this.actif = true});

  /// Passe à false si un chargement échoue : le jeu continue en silence.
  bool actif;

  bool musiqueActivee = true;
  bool effetsActives = true;

  double volumeMusique = 0.30;
  double volumeEffets = 0.80;

  /// Table de correspondance effet -> fichier.
  static const Map<Effet, String> _fichiers = {
    Effet.epee: 'epee.mp3',
    Effet.piece: 'piece.mp3',
    Effet.degat: 'degat.mp3',
    Effet.porte: 'porte.mp3',
    Effet.boss: 'boss.mp3',
    Effet.victoire: 'victoire.mp3',
  };

  /// Volume relatif de chaque effet, pour équilibrer des fichiers
  /// enregistrés à des niveaux différents.
  static const Map<Effet, double> _equilibrage = {
    Effet.epee: 1.00,
    Effet.piece: 0.60,
    Effet.degat: 0.90,
    Effet.porte: 0.70,
    Effet.boss: 1.00,
    Effet.victoire: 0.85,
  };

  /// Journal des sons demandés : sert aux tests et au mode sans audio.
  final List<Effet> journal = [];

  Future<void> precharger() async {
    if (!actif) return;
    try {
      await FlameAudio.audioCache.loadAll(_fichiers.values.toList());
    } catch (_) {
      actif = false; // un fichier manque : mode silencieux
    }
  }

  Future<void> jouer(Effet effet) async {
    journal.add(effet);
    if (!actif || !effetsActives) return;
    final volume = volumeEffets * (_equilibrage[effet] ?? 1.0);
    try {
      await FlameAudio.play(_fichiers[effet]!, volume: volume);
    } catch (_) {
      actif = false;
    }
  }

  void demarrerMusique(String fichier) {
    if (!actif || !musiqueActivee) return;
    FlameAudio.bgm.play(fichier, volume: volumeMusique);
  }

  void pauseMusique() => actif ? FlameAudio.bgm.pause() : null;
  void arreterMusique() => actif ? FlameAudio.bgm.stop() : null;

  void repriseMusique() {
    if (!actif || !musiqueActivee) return;
    FlameAudio.bgm.resume();
  }

  void basculerMusique() {
    musiqueActivee = !musiqueActivee;
    musiqueActivee ? repriseMusique() : pauseMusique();
  }

  void basculerEffets() => effetsActives = !effetsActives;

  void liberer() {
    if (!actif) return;
    FlameAudio.bgm.dispose();
  }
}
```

Le branchement dans le jeu tient en quelques lignes.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame_audio/flame_audio.dart';

class DonjonDeDart extends FlameGame {
  final ServiceAudio audio = ServiceAudio();

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    FlameAudio.bgm.initialize();
    await audio.precharger();
    audio.demarrerMusique('musique_donjon.mp3');
  }

  @override
  void onRemove() {
    audio.liberer();
    super.onRemove();
  }
}

class Heros extends PositionComponent with HasGameReference<DonjonDeDart> {
  void attaquer() {
    game.audio.jouer(Effet.epee);
  }

  void ramasserPiece() {
    game.audio.jouer(Effet.piece);
  }
}
```

Le bénéfice principal apparaît quand on teste : le `journal` rend le service vérifiable **sans Flutter et sans son**, en construisant un `ServiceAudio(actif: false)` et en comptant les effets demandés. C'est l'objet de l'exercice 3.

> **À retenir.** Un service audio bien conçu se reconnaît à ceci : le reste du jeu n'importe jamais `flame_audio`. Il appelle `audio.jouer(Effet.epee)`. Le jour où vous changez de bibliothèque audio, un seul fichier change.

---

## 34.14 — Où trouver des sons libres de droits

Vous ne pouvez pas prendre un son sur une vidéo ni dans un autre jeu. Même pour un projet scolaire, la règle est la même : on utilise ce qu'on a le droit d'utiliser.

Voici les sources habituelles du monde du jeu indépendant.

| Source | Contenu | Licence courante | Remarque |
| --- | --- | --- | --- |
| `freesound.org` | banque collaborative très large | variable, à lire par fichier | filtrez par licence avant de télécharger |
| `opengameart.org` | assets de jeu, sons et musiques | CC0, CC-BY, GPL selon le fichier | rubrique « Art Type: Sound Effect » |
| `kenney.nl` | packs d'effets d'interface et d'action | CC0 (domaine public) | qualité homogène, idéal pour un premier jeu |
| `incompetech.com` | musiques d'ambiance | CC-BY | attribution obligatoire |
| `sfxr` / `jsfxr` / `bfxr` | générateurs de sons rétro | vous créez le fichier, il est à vous | parfait pour un donjon 8 bits |
| `mixkit.co` | effets et musiques | licence propre, gratuite | lire les conditions |

Trois licences à savoir distinguer :

| Licence | Ce que vous devez faire |
| --- | --- |
| **CC0** | rien : le fichier est dans le domaine public |
| **CC-BY** | citer l'auteur dans les crédits du jeu |
| **CC-BY-SA** | citer l'auteur **et** publier votre travail sous la même licence |
| **CC-BY-NC** | citer l'auteur et ne pas faire d'usage commercial |

Pour le « Donjon de Dart », la recommandation est claire : **cherchez du CC0**, et créez vos propres bruitages avec un générateur `sfxr`. Un donjon rétro sonne très bien avec des sons générés, et vous n'aurez aucune question de licence.

Prévoyez dès maintenant un fichier `assets/audio/CREDITS.txt` :

```text
CREDITS AUDIO — Donjon de Dart
------------------------------
epee.mp3      — généré avec bfxr, domaine public
piece.mp3     — généré avec bfxr, domaine public
degat.mp3     — « Hit 03 » par Kenney (kenney.nl), licence CC0
musique_donjon.mp3 — « Dungeon Ambient » par X. Y. (opengameart.org), CC-BY 3.0
```

> **Remarque.** Le fichier `CREDITS.txt` placé dans `assets/audio/` sera embarqué avec le reste du dossier. Ce n'est pas gênant : il pèse quelques centaines d'octets et il prouve votre bonne foi.

---

## 34.15 — Solution de repli sans fichier audio

C'est la contrainte permanente de ce cours : **tout doit s'exécuter sans télécharger quoi que ce soit**. Le son ne fait pas exception.

Le principe est celui de la **dégradation gracieuse** : le jeu détecte l'absence de son et remplace le retour sonore par un retour visuel, sans changer une ligne de sa logique.

```text
  ┌───────────────────────────────────────────────────────────┐
  │  Le héros frappe                                          │
  └───────────────────────────────────────────────────────────┘
              │
              ▼
      audio.jouer(Effet.epee)
              │
      ┌───────┴────────┐
      │                │
   actif = true    actif = false
      │                │
      ▼                ▼
  FlameAudio.play  retour VISUEL : flash, texte, particule
```

Voici un jeu complet qui applique cette règle. Il fonctionne tel quel, sans aucun fichier dans `assets/audio/`, et se met à sonner dès que vous en déposez un. Le `ServiceAudio` de la section 34.13 fait déjà le plus gros du travail : il bascule `actif` à `false` au premier échec de chargement. Il ne reste qu'à brancher le retour visuel.

```dart
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

import 'services/service_audio.dart'; // ServiceAudio et Effet, section 34.13

void main() {
  runApp(GameWidget(game: DonjonSonore()));
}

class DonjonSonore extends FlameGame {
  final ServiceAudio audio = ServiceAudio();
  late final TextComponent bandeau;
  late final RectangleComponent flash;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await audio.precharger();

    // Le flash sert de substitut visuel quand le son est absent.
    flash = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0x00FFFFFF),
      priority: 100,
    );
    bandeau = TextComponent(
      position: Vector2(12, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFEEEEEE)),
      ),
    );
    await camera.viewport.addAll([flash, bandeau]);
    _majBandeau('prêt');

    await world.addAll([
      Cible(Effet.epee, Vector2(40, 120), const Color(0xFFB5563C), 'ÉPÉE'),
      Cible(Effet.piece, Vector2(160, 120), const Color(0xFFE8B04B), 'PIÈCE'),
      Cible(Effet.degat, Vector2(280, 120), const Color(0xFF7A4E9E), 'DÉGÂT'),
    ]);
  }

  void _majBandeau(String message) {
    final mode = audio.actif ? 'son activé' : 'SANS SON (retour visuel)';
    bandeau.text = '$mode — $message';
  }

  Future<void> declencher(Effet effet, String libelle) async {
    await audio.jouer(effet);
    _majBandeau(libelle);

    // Retour visuel : toujours affiché, indispensable en mode silencieux.
    flash.paint.color = const Color(0x33FFFFFF);
    flash.add(OpacityEffect.fadeOut(EffectController(duration: 0.18)));
  }
}

class Cible extends RectangleComponent
    with TapCallbacks, HasGameReference<DonjonSonore> {
  Cible(this.effet, Vector2 position, Color couleur, this.libelle)
      : super(position: position, size: Vector2.all(96),
              paint: Paint()..color = couleur);

  final Effet effet;
  final String libelle;

  @override
  void onTapDown(TapDownEvent event) => game.declencher(effet, libelle);
}
```

**Résultat sans fichier audio :**

```text
SANS SON (retour visuel) — ÉPÉE
SANS SON (retour visuel) — PIÈCE
```

**Résultat avec les fichiers :**

```text
son activé — ÉPÉE
son activé — PIÈCE
```

**Explication.** Le service tente le préchargement dans `onLoad` ; s'il échoue, `actif` passe à `false` et plus aucun appel audio n'est tenté. Le reste du jeu ne change pas d'un caractère : il appelle `declencher`, qui joue le son **et** affiche le flash. Le flash n'est pas un pis-aller, d'ailleurs : dans un jeu final, il reste utile pour les joueurs sourds et pour ceux qui jouent sans son dans les transports.

> **À retenir.** Tout retour sonore doit avoir un jumeau visuel. Ce n'est pas seulement une solution de repli technique : c'est une règle d'accessibilité.

---

## 34.16 — Pourquoi un éditeur de niveaux

Au chapitre 25, vous avez construit un niveau avec un tableau de chaînes :

```dart
const List<String> niveau1 = [
  '####################',
  '#..........#.......#',
  '#..P.......#...C...#',
  '#..........#.......#',
  '#....####..#.......#',
  '#....#..#..........#',
  '#....#..#...G......#',
  '#..........#########',
  '#..K.......#.......#',
  '####################',
];
```

Cette technique est excellente pour apprendre, et elle reste valable pour un petit jeu. Mais elle atteint ses limites très vite.

**Limite 1 — vous dessinez à l'aveugle.** Pour savoir à quoi ressemble le niveau, il faut lancer le jeu. Corriger un mur mal placé demande un aller-retour de trente secondes ; multipliez par deux cents corrections.

**Limite 2 — un seul caractère par case.** Comment poser une torche **sur** un mur ? Un tapis **sous** un coffre ? Il faudrait deux tableaux, puis trois.

**Limite 3 — pas de données attachées.** Le caractère `C` désigne un coffre, mais quel coffre ? Contenant quoi ? Verrouillé par quelle clé ? Il faut une seconde structure, tenue à jour à la main.

**Limite 4 — le grand niveau devient illisible.** Un niveau de 100 × 60 cases fait 60 lignes de 100 caractères.

**Limite 5 — vous êtes le seul à pouvoir travailler dessus.** Un game designer non programmeur ne touchera jamais à un tableau Dart.

Un **éditeur de niveaux** règle les cinq points d'un coup.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │        SANS ÉDITEUR                 AVEC ÉDITEUR (Tiled)     │
  ├──────────────────────────────────────────────────────────────┤
  │  écrire des caractères           dessiner à la souris        │
  │  compiler pour voir              voir immédiatement          │
  │  1 information par case          N calques superposés        │
  │  données en dur ailleurs         propriétés sur chaque objet │
  │  fichier .dart                   fichier .tmx (XML)          │
  │  relire le code pour modifier    rouvrir le fichier          │
  └──────────────────────────────────────────────────────────────┘
```

**Tiled** est l'éditeur de niveaux 2D le plus répandu. Il est gratuit, libre, multiplateforme, et Flame fournit un paquet-pont officiel pour lire ses fichiers : `flame_tiled`.

---

## 34.17 — Installer et prendre en main Tiled

Tiled se télécharge sur `mapeditor.org`. Il existe pour Windows, macOS et Linux. Il ne s'agit pas d'un paquet Dart : c'est une application autonome, que vous installez comme n'importe quel logiciel.

> **Si vous ne pouvez pas l'installer**, ne sautez pas cette partie. Les sections qui suivent expliquent le contenu du fichier `.tmx` ligne par ligne, et vous pourrez l'écrire à la main dans un éditeur de texte. La section 34.28 fournit en plus une solution de repli complète.

Le flux de travail, une fois Tiled lancé :

```text
  1. Fichier > Nouvelle carte
        Orientation : Orthogonale
        Format des calques : CSV       (le plus lisible)
        Taille de la carte : 40 x 24 tuiles
        Taille des tuiles  : 16 x 16 pixels

  2. Carte > Nouveau tileset
        Type : basé sur une image de tuiles
        Image : assets/tiles/donjon_tileset.png
        Largeur/hauteur de tuile : 16 x 16

  3. Dessiner sur le calque « Sol »
  4. Ajouter un calque « Murs » et dessiner dessus
  5. Ajouter un calque d'objets « Entites » et y placer des rectangles
  6. Fichier > Enregistrer sous… > assets/tiles/donjon1.tmx
```

Deux réglages méritent une attention particulière au moment de la création.

**Le format des calques.** Tiled peut compresser les données de tuiles (`zlib`, `gzip`, `zstd`) ou les écrire en clair. Choisissez **CSV** : le fichier reste lisible dans un éditeur de texte, ce qui est précieux pour apprendre et pour déboguer.

**Le chemin de l'image du tileset.** Tiled enregistre un chemin **relatif** au fichier `.tmx`. Si votre `.tmx` et votre image sont tous les deux dans `assets/tiles/`, le chemin sera simplement `donjon_tileset.png`. C'est ce que vous voulez. Si vous rangez l'image ailleurs, vous obtiendrez un chemin du type `../images/donjon_tileset.png`, qui fonctionnera à condition que le fichier soit déclaré dans le `pubspec.yaml`.

Vocabulaire de l'interface, en français et en anglais, parce que la documentation est anglophone :

| Français | Anglais | Définition |
| --- | --- | --- |
| Carte | Map | le niveau entier |
| Tuile | Tile | une case du tileset, ici 16 × 16 pixels |
| Jeu de tuiles | Tileset | l'image découpée qui fournit les tuiles |
| Calque de tuiles | Tile Layer | une grille de tuiles superposable |
| Calque d'objets | Object Layer / Object Group | des formes libres, non alignées sur la grille |
| Propriété personnalisée | Custom Property | une donnée nommée attachée à un objet, une tuile ou un calque |

---

## 34.18 — Tileset, calques, tuiles

Trois notions, et une seule à la fois.

### Le tileset

Un tileset est une **image unique** contenant toutes les tuiles, rangées en grille. Chaque tuile reçoit un identifiant, appelé **GID** (global id), attribué de gauche à droite puis de haut en bas, en commençant à **1**. La valeur **0** est réservée : elle signifie « case vide ».

```text
  donjon_tileset.png   (64 x 32 pixels, tuiles de 16 x 16)

  ┌────┬────┬────┬────┐
  │ 1  │ 2  │ 3  │ 4  │     1 = sol de pierre
  ├────┼────┼────┼────┤     2 = mur
  │ 5  │ 6  │ 7  │ 8  │     3 = dalle fissurée
  └────┴────┴────┴────┘     4 = eau
                             5 = escalier
                             6 = porte fermée
                             7 = torche
                             8 = herbe
```

Une seule image pour tout le décor : c'est le même principe que la sprite sheet du chapitre 22, et c'est efficace pour la même raison. Le moteur charge une texture, puis découpe.

### Les calques de tuiles

Un calque est une grille de la taille de la carte. Chaque case contient un GID, ou `0`. Les calques se superposent dans l'ordre où ils apparaissent dans le fichier : le premier est au fond.

```text
  ┌──────────────────────────────────────────────────────┐
  │  Calque 3 : « Decor »    torches, tapis, colonnes    │  ← devant
  ├──────────────────────────────────────────────────────┤
  │  Calque 2 : « Murs »     murs et obstacles           │
  ├──────────────────────────────────────────────────────┤
  │  Calque 1 : « Sol »      pierre, herbe, eau          │  ← au fond
  └──────────────────────────────────────────────────────┘
```

Découper le décor en calques résout la limite 2 de la section 34.16 : une torche peut être posée sur un mur, parce qu'elle est sur un autre calque.

Nommez vos calques avec rigueur : **vous les retrouverez par leur nom depuis le code**. `Sol`, `Murs`, `Decor`, `Entites`. Évitez les accents et les espaces, non pas parce que c'est interdit, mais parce que vous écrirez ces chaînes en dur dans votre Dart.

### Les tuiles

Une tuile est simplement une case du tileset. Elle peut recevoir des propriétés personnalisées dans Tiled (par exemple `solide = true`), mais nous ne nous appuierons pas sur cette possibilité : nous utiliserons des calques d'objets, plus explicites et mieux documentés côté `flame_tiled`.

---

## 34.19 — Le format `.tmx` et le dossier `assets/tiles/`

Un fichier `.tmx` est du **XML**. Vous pouvez l'ouvrir avec n'importe quel éditeur de texte. Le comprendre vous rendra autonome le jour où Tiled produira quelque chose d'inattendu.

Voici une carte minuscule mais complète : 8 × 6 tuiles de 16 pixels, deux calques de tuiles et un calque d'objets.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<map version="1.10" tiledversion="1.11.0" orientation="orthogonal"
     renderorder="right-down" width="8" height="6"
     tilewidth="16" tileheight="16" infinite="0" nextlayerid="4" nextobjectid="5">

  <tileset firstgid="1" name="donjon" tilewidth="16" tileheight="16"
           tilecount="8" columns="4">
    <image source="donjon_tileset.png" width="64" height="32"/>
  </tileset>

  <layer id="1" name="Sol" width="8" height="6">
    <data encoding="csv">
1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1
    </data>
  </layer>

  <layer id="2" name="Murs" width="8" height="6">
    <data encoding="csv">
2,2,2,2,2,2,2,2,
2,0,0,0,0,0,0,2,
2,0,0,2,2,0,0,2,
2,0,0,0,0,0,0,2,
2,0,0,0,0,0,0,2,
2,2,2,2,2,2,2,2
    </data>
  </layer>

  <objectgroup id="3" name="Entites">
    <object id="1" name="depart" type="apparition" x="32" y="32" width="16" height="16"/>
    <object id="2" name="coffre1" type="coffre" x="96" y="64" width="16" height="16">
      <properties>
        <property name="contenu" value="potion"/>
        <property name="verrouille" type="bool" value="true"/>
        <property name="or" type="int" value="25"/>
      </properties>
    </object>
    <object id="3" name="gobelin1" type="ennemi" x="64" y="48" width="16" height="16">
      <properties>
        <property name="pv" type="int" value="12"/>
        <property name="vitesse" type="float" value="45.5"/>
      </properties>
    </object>
    <object id="4" name="sortie" type="porte" x="112" y="48" width="16" height="16">
      <properties>
        <property name="niveauSuivant" value="donjon2.tmx"/>
      </properties>
    </object>
  </objectgroup>

</map>
```

Décryptage des attributs qui comptent :

| Attribut | Où | Signification |
| --- | --- | --- |
| `width` / `height` sur `<map>` | carte | taille **en tuiles**, pas en pixels |
| `tilewidth` / `tileheight` | carte | taille d'une tuile **en pixels** |
| `firstgid` | tileset | GID de la première tuile de ce tileset |
| `source` sur `<image>` | tileset | chemin de l'image, relatif au `.tmx` |
| `encoding="csv"` | data | les GID sont écrits en clair, séparés par des virgules |
| `x` / `y` sur `<object>` | objet | position **en pixels**, pas en tuiles |
| `type` sur `<object>` | objet | catégorie libre, choisie par vous |

Un point à bien saisir : la carte fait 8 × 6 **tuiles**, soit 128 × 96 **pixels**. Les objets, eux, sont positionnés en pixels : l'objet `depart` est à `x=32, y=32`, c'est-à-dire à la tuile de colonne 2, ligne 2. La conversion s'écrit `colonne = x / tilewidth` dans un sens, `x = colonne * tilewidth` dans l'autre.

Le fichier `.tmx` **et** l'image du tileset se rangent dans `assets/tiles/`, déclaré dans le `pubspec.yaml` :

```yaml
flutter:
  assets:
    - assets/images/
    - assets/audio/
    - assets/tiles/
```

> **Piège classique.** On déclare `assets/tiles/` mais on oublie que l'image du tileset y est aussi. Si vous rangez l'image dans `assets/images/`, il faut que Tiled écrive `../images/donjon_tileset.png` **et** que `assets/images/` soit déclaré. Le plus simple reste de tout mettre dans `assets/tiles/`.

---

## 34.20 — Installer `flame_tiled`

Comme pour l'audio, une ligne dans le `pubspec.yaml` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0
  flame_audio: ^2.12.2
  flame_tiled: ^3.1.2
```

```text
flutter pub get
```

**Résultat :**

```text
Resolving dependencies...
+ flame_tiled 3.1.2
+ tiled 0.11.0
+ xml 6.3.0
...
Changed 4 dependencies!
```

Notez la dépendance `tiled 0.11.0`. C'est un paquet Dart **indépendant de Flame**, qui sait analyser un fichier `.tmx` et produire des objets Dart (`TiledMap`, `TileLayer`, `ObjectGroup`, `TiledObject`). `flame_tiled` ajoute par-dessus le rendu et l'intégration au système de composants.

Un seul import est nécessaire, et il ré-exporte les types du paquet `tiled` dont vous aurez besoin :

```dart
import 'package:flame_tiled/flame_tiled.dart';
```

---

## 34.21 — `TiledComponent.load()`

Voici la signature réelle de la méthode statique en `flame_tiled` 3.1.2 :

```dart
static Future<TiledComponent<FlameGame<World>>> load(
  String fileName,
  Vector2 destTileSize, {
  double? atlasMaxX,
  double? atlasMaxY,
  String prefix = 'assets/tiles/',
  int? priority,
  bool? ignoreFlip,
  AssetBundle? bundle,
  Images? images,
  bool Function(Tileset)? tsxPackingFilter,
  bool useAtlas = true,
  Paint Function(double opacity)? layerPaintFactory,
  double atlasPackingSpacingX = 0,
  double atlasPackingSpacingY = 0,
  ComponentKey? key,
  String? package,
});
```

Deux paramètres sont obligatoires, et le second est celui qui surprend.

**`fileName`** est le nom du fichier, **relatif à `assets/tiles/`**. Comme pour l'audio et les images, ne mettez jamais le chemin complet.

**`destTileSize`** est la taille **de destination** d'une tuile, en pixels du monde de jeu. Ce n'est pas la taille de la tuile dans le tileset. Les deux sont souvent identiques, mais pas toujours.

```text
  Tuile SOURCE (dans le tileset)  :  16 x 16 pixels
  destTileSize                    :  16 x 16   → rendu 1:1
  destTileSize                    :  32 x 32   → chaque tuile occupe
                                                 32 pixels dans le monde
```

Autrement dit, `destTileSize` vous permet d'agrandir toute la carte sans toucher aux images. En pratique, on met la même valeur que dans Tiled et on règle l'agrandissement avec le `zoom` de la caméra, ce qui est plus souple.

```dart
// Cas standard : tuiles de 16 px, rendu à 16 px.
final carte = await TiledComponent.load('donjon1.tmx', Vector2.all(16));

// Carte au fond de tout, et fichier rangé ailleurs que dans assets/tiles/
final carte2 = await TiledComponent.load(
  'donjon1.tmx',
  Vector2.all(16),
  priority: -10,
  prefix: 'assets/niveaux/',
);
```

Le paramètre `useAtlas` (vrai par défaut) regroupe les images de tuiles dans une seule grande texture pour accélérer le rendu. Laissez-le à `true`. Si vous observez de fines lignes entre les tuiles — un défaut connu et documenté, appelé *seams* —, c'est de ce mécanisme qu'elles viennent ; l'option `bleed` de `SpriteBatch` ajoutée en Flame 1.38.0 vise précisément ce problème.

---

## 34.22 — Placer la carte dans le monde

Le `TiledComponent` est un `PositionComponent` comme les autres. Il s'ajoute au **monde**, jamais directement au jeu, pour que la caméra puisse le déplacer.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame_tiled/flame_tiled.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: JeuTiled()));
}

class JeuTiled extends FlameGame {
  late final TiledComponent carte;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    carte = await TiledComponent.load('donjon1.tmx', Vector2.all(16));
    await world.add(carte);

    // Le repère du monde démarre en haut à gauche de la carte.
    camera.viewfinder.anchor = Anchor.topLeft;
    camera.viewfinder.zoom = 3.0;
  }
}
```

**Résultat :** la carte s'affiche, agrandie trois fois, coin supérieur gauche à l'origine.

Deux réglages de caméra méritent d'être expliqués.

**`anchor = Anchor.topLeft`.** Par défaut, le `Viewfinder` est centré : le point `(0, 0)` du monde apparaît au **centre** de l'écran, et la moitié gauche de votre carte se retrouve hors champ. Avec `Anchor.topLeft`, le point `(0, 0)` du monde est placé en haut à gauche de l'écran, ce qui correspond à l'intuition d'une carte.

**`zoom`.** Un tileset de 16 pixels affiché à l'échelle 1 sur un téléphone moderne est minuscule. Un zoom de 3 ou 4 est habituel pour du pixel art 16 × 16.

L'autre approche, utilisée par l'exemple officiel de `flame_tiled`, consiste à fixer la résolution logique du jeu et à laisser Flame calculer l'agrandissement, en passant une caméra au constructeur :

```dart
class JeuTiled extends FlameGame {
  JeuTiled()
      : super(
          camera: CameraComponent.withFixedResolution(
            width: 16 * 28,   // 28 tuiles visibles en largeur
            height: 16 * 14,  // 14 tuiles visibles en hauteur
          ),
        );
}
```

Cette seconde forme garantit que le joueur voit **exactement** 28 × 14 tuiles, quel que soit l'écran. C'est la meilleure option pour un donjon, où le champ de vision fait partie de la difficulté.

Enfin, la taille de la carte est disponible après le chargement, ce qui permet de borner la caméra (chapitre 31) :

```dart
@override
Future<void> onLoad() async {
  await super.onLoad();

  carte = await TiledComponent.load('donjon1.tmx', Vector2.all(16));
  await world.add(carte);

  // carte.size vaut (largeur en tuiles * 16, hauteur en tuiles * 16)
  print('Taille de la carte : ${carte.size}');
}
```

**Résultat pour la carte d'exemple de 8 × 6 tuiles :**

```text
Taille de la carte : [128.0,96.0]
```

---

## 34.23 — Les calques de tuiles

Une fois la carte chargée, tout passe par la propriété `tileMap`, de type `RenderableTiledMap`. C'est elle qui donne accès aux calques.

```dart
// Récupérer un calque de tuiles par son nom.
final TileLayer? sol = carte.tileMap.getLayer<TileLayer>('Sol');
final TileLayer? murs = carte.tileMap.getLayer<TileLayer>('Murs');
```

`getLayer<T>(String name)` renvoie `null` si aucun calque de ce nom et de ce type n'existe. Vérifiez toujours, car une faute de frappe dans le nom est l'erreur la plus fréquente de cette partie.

```dart
final murs = carte.tileMap.getLayer<TileLayer>('Murs');
if (murs == null) {
  throw StateError(
    'Le calque « Murs » est introuvable. '
    'Vérifiez son nom exact dans Tiled (sensible à la casse).',
  );
}
```

Un `TileLayer` expose les propriétés suivantes :

| Propriété | Type | Contenu |
| --- | --- | --- |
| `name` | `String` | le nom donné dans Tiled |
| `width` / `height` | `int` | taille du calque **en tuiles** |
| `visible` | `bool` | calque visible ou masqué |
| `opacity` | `double` | opacité, de 0 à 1 |
| `offsetX` / `offsetY` | `double` | décalage du calque en pixels |
| `properties` | `CustomProperties` | propriétés personnalisées du calque |
| `tileData` | `List<List<Gid>>?` | la grille des identifiants de tuiles |

`tileData` est un tableau à deux dimensions **indexé ligne puis colonne** : `tileData[y][x]`. Chaque case est un objet `Gid`, dont le champ `tile` contient l'identifiant global de la tuile, ou `0` pour une case vide.

```dart
final donnees = murs.tileData!;
for (var y = 0; y < murs.height; y++) {
  final ligne = StringBuffer();
  for (var x = 0; x < murs.width; x++) {
    ligne.write(donnees[y][x].tile == 0 ? '.' : '#');
  }
  print(ligne);
}
```

**Résultat pour le calque « Murs » de notre carte d'exemple :**

```text
########
#......#
#..##..#
#......#
#......#
########
```

Vous reconnaissez la carte du chapitre 25. C'est exactement la même information, produite par un éditeur graphique au lieu d'être tapée à la main.

Deux méthodes utiles de `RenderableTiledMap` permettent de modifier la carte en cours de partie :

```dart
// Lire une case précise.
final Gid? g = carte.tileMap.getTileData(layerId: 1, x: 3, y: 2);

// Masquer un calque (par exemple un brouillard de guerre).
carte.tileMap.setLayerVisibility(2, visible: false);

// Changer l'opacité d'un calque.
carte.tileMap.setLayerOpacity(2, opacity: 0.5);
```

> **Remarque.** `layerId` est l'identifiant du calque tel qu'il figure dans le `.tmx` (`<layer id="1" ...>`), pas son index dans la liste. Les deux coïncident souvent, mais pas après une suppression de calque dans Tiled.

---

## 34.24 — Les calques d'objets

Un calque de tuiles décrit le décor. Il ne peut pas décrire **ce qui vit dedans**. C'est le rôle du calque d'objets, appelé `objectgroup` dans le `.tmx` et `ObjectGroup` dans le code.

Différences fondamentales avec un calque de tuiles :

| | Calque de tuiles | Calque d'objets |
| --- | --- | --- |
| Structure | grille fixe | liste libre |
| Position | alignée sur la grille | n'importe quel pixel |
| Contenu d'une case | un GID | une forme (rectangle, point, ellipse, polygone) |
| Données attachées | via la tuile | **propriétés par objet** |
| Usage typique | sol, murs, décor | apparition, ennemis, coffres, portes, zones |

L'accès se fait avec le même `getLayer`, en changeant le type générique :

```dart
final ObjectGroup? entites =
    carte.tileMap.getLayer<ObjectGroup>('Entites');

if (entites != null) {
  print('Nombre d\'objets : ${entites.objects.length}');
  for (final objet in entites.objects) {
    print('${objet.type} « ${objet.name} » '
        'en (${objet.x}, ${objet.y}) '
        'taille ${objet.width} x ${objet.height}');
  }
}
```

**Résultat pour la carte d'exemple :**

```text
Nombre d'objets : 4
apparition « depart » en (32.0, 32.0) taille 16.0 x 16.0
coffre « coffre1 » en (96.0, 64.0) taille 16.0 x 16.0
ennemi « gobelin1 » en (64.0, 48.0) taille 16.0 x 16.0
porte « sortie » en (112.0, 48.0) taille 16.0 x 16.0
```

Chaque élément est un `TiledObject`. Voici ses champs utiles :

| Champ | Type | Usage |
| --- | --- | --- |
| `id` | `int` | identifiant unique dans la carte |
| `name` | `String` | nom libre saisi dans Tiled |
| `type` | `String` | catégorie libre : `ennemi`, `coffre`, `porte`… |
| `class_` | `String` | équivalent moderne de `type` selon la version de Tiled |
| `x`, `y` | `double` | position en pixels, coin **supérieur gauche** pour un rectangle |
| `width`, `height` | `double` | taille en pixels (0 pour un point) |
| `rotation` | `double` | rotation en degrés |
| `visible` | `bool` | objet visible dans l'éditeur |
| `point`, `ellipse`, `rectangle` | `bool` | nature de la forme |
| `polygon`, `polyline` | `List<Point>` | sommets, pour les formes libres |
| `properties` | `CustomProperties` | les données personnalisées |

Il existe aussi les getters `isPoint`, `isEllipse`, `isPolygon`, `isPolyline` et `isRectangle`, plus lisibles qu'un test sur les booléens bruts.

> **Attention à `type` et `class_`.** Tiled a renommé le champ « Type » en « Class » à partir de la version 1.9. Le paquet `tiled` expose les deux. Selon la version de Tiled qui a produit votre fichier, l'un des deux sera rempli et l'autre vide. Le code robuste teste les deux :

```dart
String categorie(TiledObject o) =>
    o.class_.isNotEmpty ? o.class_ : o.type;
```

---

## 34.25 — Lire les objets : apparition, coffres, portes, ennemis

Nous y sommes : transformer un calque d'objets en entités de jeu. Le schéma est toujours le même.

```text
  ObjectGroup « Entites »
        │
        │  pour chaque TiledObject
        ▼
    switch sur la catégorie
        │
        ├── 'apparition' ──► positionner le héros
        ├── 'ennemi'     ──► world.add(Gobelin(...))
        ├── 'coffre'     ──► world.add(Coffre(...))
        ├── 'porte'      ──► world.add(Porte(...))
        └── autre        ──► avertissement dans la console
```

Voici le code complet, écrit dans le style du « Donjon de Dart ». Il utilise des `RectangleComponent` colorés pour rester exécutable sans aucune image.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame_tiled/flame_tiled.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(GameWidget(game: DonjonTiled()));
}

class DonjonTiled extends FlameGame {
  late final TiledComponent carte;
  late final Heros heros;
  final List<String> journal = [];

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder
      ..anchor = Anchor.topLeft
      ..zoom = 3.0;

    carte = await TiledComponent.load('donjon1.tmx', Vector2.all(16));
    await world.add(carte);

    await _construireEntites();
  }

  Future<void> _construireEntites() async {
    final groupe = carte.tileMap.getLayer<ObjectGroup>('Entites');
    if (groupe == null) {
      journal.add('Calque « Entites » absent : niveau vide.');
      return;
    }

    for (final objet in groupe.objects) {
      final categorie =
          objet.class_.isNotEmpty ? objet.class_ : objet.type;
      final position = Vector2(objet.x, objet.y);
      final taille = Vector2(
        objet.width == 0 ? 16 : objet.width,
        objet.height == 0 ? 16 : objet.height,
      );

      switch (categorie) {
        case 'apparition':
          heros = Heros(position: position, size: taille);
          await world.add(heros);
          camera.follow(heros);
          journal.add('Héros placé en (${objet.x}, ${objet.y})');

        case 'ennemi':
          await world.add(Gobelin(position: position, size: taille));
          journal.add('Gobelin « ${objet.name} » ajouté');

        case 'coffre':
          await world.add(Coffre(position: position, size: taille));
          journal.add('Coffre « ${objet.name} » ajouté');

        case 'porte':
          await world.add(Porte(position: position, size: taille));
          journal.add('Porte « ${objet.name} » ajoutée');

        default:
          journal.add('Catégorie inconnue : « $categorie » '
              '(objet « ${objet.name} »)');
      }
    }

    for (final ligne in journal) {
      print(ligne);
    }
  }
}

/// Quatre entités, toutes de simples rectangles colorés :
/// aucun asset n'est nécessaire pour exécuter cet exemple.
class Heros extends RectangleComponent {
  Heros({super.position, super.size})
      : super(paint: Paint()..color = const Color(0xFF4CAF50));
}

class Gobelin extends RectangleComponent {
  Gobelin({super.position, super.size})
      : super(paint: Paint()..color = const Color(0xFFB5563C));
}

class Coffre extends RectangleComponent {
  Coffre({super.position, super.size})
      : super(paint: Paint()..color = const Color(0xFFE8B04B));
}

class Porte extends RectangleComponent {
  Porte({super.position, super.size})
      : super(paint: Paint()..color = const Color(0xFF7A4E9E));
}
```

**Résultat :**

```text
Héros placé en (32.0, 32.0)
Gobelin « gobelin1 » ajouté
Coffre « coffre1 » ajouté
Porte « sortie » ajoutée
```

Trois détails du code méritent un commentaire.

**Le repli sur `16` pour la taille.** Un objet « point » dans Tiled a une largeur et une hauteur nulles. Un composant de taille nulle ne se dessine pas et ne peut recevoir aucun tap (chapitre 30). Le repli garantit une taille utilisable.

**Le `default` qui journalise au lieu de planter.** Un game designer ajoutera un jour un objet de type `piege` que votre code ne connaît pas. Il vaut mieux un avertissement lisible qu'une exception au lancement du niveau.

**L'appel à `camera.follow(heros)` juste après la création.** La caméra ne peut suivre le héros qu'une fois celui-ci créé, et le héros est créé par la lecture de la carte. L'ordre est donc imposé : charger la carte, lire les objets, puis brancher la caméra.

---

## 34.26 — Les propriétés personnalisées

Un objet `coffre` sans données n'est qu'un rectangle jaune. Ce qui fait un coffre, c'est son contenu, sa serrure, sa récompense. Tiled permet d'attacher à chaque objet un nombre libre de **propriétés personnalisées**, typées.

Dans l'interface, on les ajoute en bas du panneau « Propriétés » de l'objet sélectionné. Dans le `.tmx`, elles apparaissent ainsi :

```xml
<object id="2" name="coffre1" type="coffre" x="96" y="64" width="16" height="16">
  <properties>
    <property name="contenu" value="potion"/>
    <property name="verrouille" type="bool" value="true"/>
    <property name="or" type="int" value="25"/>
    <property name="poids" type="float" value="3.5"/>
  </properties>
</object>
```

Notez que `contenu` n'a pas d'attribut `type` : dans le format Tiled, l'absence de type signifie **chaîne de caractères**.

Côté Dart, ces propriétés arrivent dans un objet `CustomProperties` accessible par `objet.properties`. Son API :

| Méthode | Signature | Rôle |
| --- | --- | --- |
| `getValue<T>` | `T? getValue<T>(String name)` | la **valeur** typée, ou `null` |
| `getProperty<T>` | `T? getProperty<T>(String name)` | l'**objet propriété** complet |
| `has` | `bool has(String name)` | la propriété existe-t-elle |
| `operator []` | `Property<Object>? operator [](String name)` | accès brut |
| `byName` | `Map<String, Property<Object>>` | la table complète |

L'usage courant se limite à `getValue` et `has` :

```dart
final contenu = objet.properties.getValue<String>('contenu');
final verrouille = objet.properties.getValue<bool>('verrouille');
final or = objet.properties.getValue<int>('or');
final poids = objet.properties.getValue<double>('poids');

print('contenu    : $contenu');
print('verrouille : $verrouille');
print('or         : $or');
print('poids      : $poids');
```

**Résultat :**

```text
contenu    : potion
verrouille : true
or         : 25
poids      : 3.5
```

`getValue<T>` renvoie `null` dans deux cas : la propriété n'existe pas, ou son type ne correspond pas à `T`. C'est du null safety du chapitre 12 : traitez systématiquement le `null` avec `??`.

```dart
// Toujours donner une valeur par défaut.
final or = objet.properties.getValue<int>('or') ?? 0;
final verrouille = objet.properties.getValue<bool>('verrouille') ?? false;
final contenu = objet.properties.getValue<String>('contenu') ?? 'rien';
```

Une extension rend le code beaucoup plus lisible dès que vous lisez dix propriétés :

```dart
extension LectureProprietes on TiledObject {
  String texte(String nom, {String defaut = ''}) =>
      properties.getValue<String>(nom) ?? defaut;

  int entier(String nom, {int defaut = 0}) =>
      properties.getValue<int>(nom) ?? defaut;

  double reel(String nom, {double defaut = 0}) =>
      properties.getValue<double>(nom) ?? defaut;

  bool booleen(String nom, {bool defaut = false}) =>
      properties.getValue<bool>(nom) ?? defaut;
}

// Usage dans la construction des entités :
//   contenu:    objet.texte('contenu', defaut: 'or'),
//   verrouille: objet.booleen('verrouille'),
//   pv:         objet.entier('pv', defaut: 10),
//   vitesse:    objet.reel('vitesse', defaut: 40),
```

> **À retenir.** Les propriétés personnalisées suppriment tout le code de configuration écrit en dur. Régler l'équilibrage d'un niveau ne demande plus de recompiler : on modifie le `.tmx` et on relance.

Le piège classique est le couple int/float : une propriété déclarée `float` avec la valeur `10` doit être lue en `double`, jamais en `int`.

---

## 34.27 — Générer les collisions à partir d'un calque d'objets

Au chapitre 32, vous avez appris à poser des `RectangleHitbox` sur vos composants et à réagir avec `onCollisionStart`. Il reste un problème pratique : écrire à la main les cinquante murs d'un niveau est aussi pénible que le tableau du chapitre 25.

La solution consiste à **dessiner les murs dans Tiled**, sur un calque d'objets dédié, et à les transformer en composants de collision au chargement.

Dans Tiled, créez un calque d'objets nommé `Collisions`, puis tracez des rectangles par-dessus vos murs. Vous n'êtes pas obligé de suivre la grille au pixel près : un long rectangle valant dix tuiles de mur remplace dix hitbox.

```xml
<objectgroup id="4" name="Collisions">
  <object id="10" x="0"   y="0"  width="128" height="16"/>
  <object id="11" x="0"   y="80" width="128" height="16"/>
  <object id="12" x="0"   y="16" width="16"  height="64"/>
  <object id="13" x="112" y="16" width="16"  height="64"/>
  <object id="14" x="48"  y="32" width="32"  height="16"/>
</objectgroup>
```

Cinq rectangles au lieu de trente-six tuiles de mur : le moteur a cinq fois moins de tests à faire par frame.

Voici la classe de mur et le générateur.

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame_tiled/flame_tiled.dart';
import 'package:flutter/material.dart';

/// Un mur invisible généré à partir d'un objet Tiled.
class Mur extends PositionComponent {
  Mur({super.position, super.size});

  @override
  Future<void> onLoad() async {
    // Passive : les murs ne se testent pas entre eux ; seuls les
    // composants « active » (héros, gobelins) les percutent.
    await add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}

class DonjonCollisions extends FlameGame with HasCollisionDetection {
  late final TiledComponent carte;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder
      ..anchor = Anchor.topLeft
      ..zoom = 3.0;

    carte = await TiledComponent.load('donjon1.tmx', Vector2.all(16));
    await world.add(carte);
    await _genererCollisions();
  }

  Future<void> _genererCollisions() async {
    final groupe = carte.tileMap.getLayer<ObjectGroup>('Collisions');
    if (groupe == null) {
      print('Aucun calque « Collisions » : niveau sans murs.');
      return;
    }

    final murs = <Mur>[];
    for (final objet in groupe.objects) {
      if (objet.width <= 0 || objet.height <= 0) continue;
      murs.add(
        Mur(
          position: Vector2(objet.x, objet.y),
          size: Vector2(objet.width, objet.height),
        ),
      );
    }

    await world.addAll(murs);
    print('Murs générés : ${murs.length}');
  }
}
```

**Résultat :**

```text
Murs générés : 5
```

Les murs sont **invisibles** : `PositionComponent` ne dessine rien par lui-même. C'est voulu, puisque le décor est déjà dessiné par le calque de tuiles. Pendant la mise au point, activez le mode debug pour les voir :

```dart
class DonjonCollisions extends FlameGame with HasCollisionDetection {
  DonjonCollisions() {
    debugMode = true; // dessine les contours de toutes les hitbox
  }
}
```

Si vous préférez générer les collisions à partir du **calque de tuiles** plutôt que d'un calque d'objets, c'est possible aussi : on parcourt `tileData` et on crée un mur par case non vide.

```dart
Future<void> _collisionsDepuisLesTuiles() async {
  final murs = carte.tileMap.getLayer<TileLayer>('Murs');
  final donnees = murs?.tileData;
  if (murs == null || donnees == null) return;

  final aAjouter = <Mur>[];
  for (var y = 0; y < murs.height; y++) {
    for (var x = 0; x < murs.width; x++) {
      if (donnees[y][x].tile == 0) continue; // case vide
      aAjouter.add(
        Mur(position: Vector2(x * 16.0, y * 16.0), size: Vector2.all(16)),
      );
    }
  }
  await world.addAll(aAjouter);
  print('Murs générés depuis les tuiles : ${aAjouter.length}');
}
```

**Résultat pour la carte d'exemple :**

```text
Murs générés depuis les tuiles : 30
```

Trente hitbox au lieu de cinq. Comparons honnêtement les deux méthodes.

| Critère | Calque d'objets | Calque de tuiles |
| --- | --- | --- |
| Travail dans Tiled | tracer des rectangles à la main | rien de plus à faire |
| Nombre de hitbox | faible | une par tuile |
| Coût par frame | faible | élevé sur un grand niveau |
| Précision | rectangles libres | alignée sur la grille |
| Risque d'oubli | oui, un mur peut ne pas être couvert | non |
| Recommandation | **niveaux publiés** | prototypes et petits niveaux |

Une optimisation intermédiaire consiste à **fusionner les tuiles voisines** en bandes horizontales, ce qui divise le nombre de hitbox par cinq ou dix sans travail supplémentaire dans Tiled. Vous l'écrirez en exercice.

---

## 34.28 — Solution de repli sans Tiled

Vous n'avez pas installé Tiled, ou vous n'avez pas de tileset. Le cours doit rester exécutable. Retour à la carte en `List<String>` du chapitre 25, mais avec une amélioration décisive : **la même interface que la version Tiled**.

L'idée est celle du chapitre 26 : définir une abstraction, deux implémentations, et un jeu qui ne sait pas laquelle il utilise.

```text
  SourceDeNiveau  (abstraction : Future<Niveau> charger())
        ▲                                   ▲
   SourceTiled (lit un .tmx)     SourceTexte (lit une List<String>)
```

Le contrat et la source de repli, en Dart :

```dart
import 'package:flame/components.dart';

/// Description d'une entité, indépendante de la source.
class DescriptionEntite {
  const DescriptionEntite({
    required this.categorie,
    required this.position,
    required this.taille,
    this.donnees = const {},
  });

  final String categorie;
  final Vector2 position;
  final Vector2 taille;
  final Map<String, Object> donnees;
}

/// Un niveau chargé, quelle que soit sa provenance.
class Niveau {
  const Niveau({required this.murs, required this.entites});

  final List<DescriptionEntite> murs;
  final List<DescriptionEntite> entites;
}

/// Contrat commun aux deux sources.
abstract class SourceDeNiveau {
  Future<Niveau> charger();
}

/// Source de repli : la carte en List<String> du chapitre 25.
class SourceTexte implements SourceDeNiveau {
  const SourceTexte(this.lignes, {this.tailleTuile = 16});

  final List<String> lignes;
  final double tailleTuile;

  static const Map<String, String> _categories = {
    'P': 'apparition',
    'G': 'ennemi',
    'C': 'coffre',
    'K': 'cle',
    'D': 'porte',
  };

  @override
  Future<Niveau> charger() async {
    final murs = <DescriptionEntite>[];
    final entites = <DescriptionEntite>[];

    for (var y = 0; y < lignes.length; y++) {
      for (var x = 0; x < lignes[y].length; x++) {
        final c = lignes[y][x];
        final position = Vector2(x * tailleTuile, y * tailleTuile);
        final taille = Vector2.all(tailleTuile);

        if (c == '#') {
          murs.add(DescriptionEntite(
              categorie: 'mur', position: position, taille: taille));
          continue;
        }
        final categorie = _categories[c];
        if (categorie != null) {
          entites.add(DescriptionEntite(
              categorie: categorie, position: position, taille: taille));
        }
      }
    }
    return Niveau(murs: murs, entites: entites);
  }
}
```

Le jeu, lui, ne connaît que `SourceDeNiveau` : il parcourt `niveau.murs` puis `niveau.entites` et ajoute au monde un `RectangleComponent` par description, coloré selon `categorie`.

**Résultat pour la carte 20 × 10 du chapitre 25 :**

```text
Murs : 92, entités : 5
```

Le jour où vous installerez Tiled, vous écrirez une classe `SourceTiled implements SourceDeNiveau` qui lit un `.tmx` et remplit les mêmes `Niveau` et `DescriptionEntite`. Le corps de `construire` ne changera pas d'une ligne, et l'unique modification sera :

```dart
// Avant
final SourceDeNiveau source = const SourceTexte(carteTexte);

// Après
final SourceDeNiveau source = const SourceTiled('donjon1.tmx');
```

La correction de l'exercice 10 assemble cette architecture au complet, avec la fusion des murs en bandes et les propriétés par défaut.

> **À retenir.** Une solution de repli bien conçue n'est pas une version dégradée du code : c'est une **seconde implémentation du même contrat**. Le reste du jeu ne voit pas la différence.

---

## 34.29 — Quand la détection intégrée ne suffit plus

Le système de collision de Flame, vu au chapitre 32, répond à une question et une seule : **ces deux formes se touchent-elles ?** Il vous donne les points d'intersection et vous laisse décider de la suite.

C'est parfait pour la grande majorité des jeux 2D : un héros qui touche une potion, elle disparaît ; un héros qui touche un gobelin, il perd une vie. Aucun calcul physique n'est nécessaire. Cela cesse de suffire dès que vous voulez une **réponse physique réaliste**, c'est-à-dire calculée et non scriptée.

| Ce que vous voulez | Flame seul | Moteur physique |
| --- | --- | --- |
| Détecter un contact | oui | oui |
| Bloquer un héros contre un mur | oui, à la main | automatique |
| Faire rebondir une balle avec un angle correct | à écrire vous-même | automatique |
| Empiler des caisses qui restent stables | très difficile | automatique |
| Une chaîne, un pont de cordes, un pendule | non | joints |
| Un véhicule avec des roues et une suspension | non | joints |
| Une explosion qui pousse dix objets | à écrire | `applyLinearImpulse` |
| Des objets qui glissent différemment selon le sol | à écrire | friction |
| Un objet qui tourne parce qu'il a été frappé de biais | non | automatique |

La ligne de partage est nette : **si vos objets doivent réagir les uns aux autres avec des forces, des masses et des rotations, il vous faut un moteur physique.** Sinon, non.

Un exemple concret dans notre univers. Un gobelin qui vous fonce dessus, c'est de la logique de jeu : `position += direction * vitesse * dt`. Un tonneau que le héros pousse, qui dévale un escalier, rebondit sur un mur et écrase le gobelin, c'est de la physique.

> **Avertissement.** Ajouter un moteur physique « au cas où » est une erreur classique. Il ajoute une dépendance, un vocabulaire, un système d'unités, et une classe de bugs entièrement nouvelle. On l'ajoute quand on ne peut pas faire autrement.

---

## 34.30 — Qu'est-ce que Forge2D

**Box2D** est un moteur physique 2D écrit en C++ par Erin Catto en 2007. Il équipe une part considérable des jeux 2D commerciaux des vingt dernières années, dont *Angry Birds*. Il est libre.

**Forge2D** est le portage de Box2D en Dart, maintenu par Blue Fire, l'équipe qui développe Flame. Le code est traduit, pas ré-inventé : les noms de classes, les concepts et les unités sont ceux de Box2D.

**`flame_forge2d`** est le paquet-pont entre les deux mondes : il fournit `Forge2DGame`, `Forge2DWorld` et `BodyComponent`, qui permettent de manipuler des corps physiques comme des composants Flame ordinaires.

Le vocabulaire, hérité de Box2D, est à connaître avant d'écrire une ligne :

| Terme | Définition |
| --- | --- |
| **World** | l'univers physique ; il possède une gravité et fait avancer la simulation |
| **Body** | un corps rigide : une masse, une position, une vitesse, une rotation |
| **Shape** | une forme géométrique pure : cercle, polygone, segment, chaîne |
| **Fixture** | l'attache d'une `Shape` à un `Body`, avec densité, friction et restitution |
| **Joint** | une contrainte reliant deux corps (charnière, ressort, glissière…) |
| **Contact** | l'événement produit quand deux fixtures se touchent |

Le point crucial : **un `Body` n'a pas de forme**. Il a une masse, une position, une vitesse. Ce sont ses `Fixture` qui portent les formes. Un même corps peut porter plusieurs fixtures, ce qui permet de composer un personnage à partir de plusieurs rectangles.

```text
   Body « tonneau »
   ├── position (x, y), angle, vitesse, masse
   └── Fixture
         ├── Shape : CircleShape(radius = 0.5)
         ├── density     = 1.0
         ├── friction    = 0.3
         └── restitution = 0.4
```

---

## 34.31 — Installer `flame_forge2d`

```yaml
dependencies:
  flutter:
    sdk: flutter
  flame: ^1.38.0
  flame_forge2d: ^0.19.3+7
```

```text
flutter pub get
```

**Résultat :**

```text
Resolving dependencies...
+ flame_forge2d 0.19.3+7
+ forge2d 0.14.2
...
Changed 2 dependencies!
```

Un seul import, qui ré-exporte les types de `forge2d` :

```dart
import 'package:flame_forge2d/flame_forge2d.dart';
```

> **Remarque sur le numéro de version.** `flame_forge2d` est en `0.19.x`, donc avant la version 1.0. En Dart, `^0.19.3+7` autorise les mises à jour jusqu'à `0.20.0` exclu, mais pas au-delà : un paquet en `0.x` peut casser son API à chaque montée de version mineure. Fixez la version dans votre `pubspec.lock` et relisez le changelog avant toute mise à jour.

---

## 34.32 — `Forge2DGame` et le facteur d'échelle (`zoom`)

`Forge2DGame` remplace `FlameGame`. Sa signature réelle :

```dart
Forge2DGame({
  Forge2DWorld? world,
  CameraComponent? camera,
  Vector2? gravity,
  ContactListener? contactListener,
  double zoom = 10,
});
```

Trois choses à comprendre, dont une est la source de toutes les déceptions des débutants.

**1. Le monde est un `Forge2DWorld`.** `game.world` n'est plus un `World` ordinaire mais un `Forge2DWorld`, capable d'héberger des `BodyComponent` et des composants Flame normaux.

**2. La gravité est inversée par rapport à Box2D.** Dans Box2D, l'axe Y monte. Dans Flame, l'axe Y descend. `flame_forge2d` a choisi de suivre Flame : **une gravité positive en Y tire vers le bas**.

```dart
// Gravité terrestre, vers le bas.
Forge2DGame(gravity: Vector2(0, 10));

// Apesanteur : un jeu spatial.
Forge2DGame(gravity: Vector2.zero());

// Vers le haut : des bulles qui remontent.
Forge2DGame(gravity: Vector2(0, -3));
```

**3. Le `zoom` vaut 10 par défaut, et ce n'est pas un caprice.**

Voici le point décisif. Forge2D ne raisonne **pas en pixels**. Il raisonne en **mètres**, en kilogrammes et en secondes, comme la physique réelle. Le moteur est réglé pour des objets dont la taille va d'environ 0,1 m à 10 m, et il impose une limite de vitesse interne d'environ 2 mètres par pas de simulation.

Si vous décidiez qu'un pixel vaut un mètre, un personnage de 64 pixels serait un géant de 64 mètres, tombant à des vitesses absurdes, et le moteur deviendrait instable.

Le `zoom` est le facteur de conversion entre les deux mondes.

```text
  zoom = 10   signifie   1 mètre physique  =  10 pixels à l'écran

  balle de 50 px  =  5.0 m      héros de 16 px  =  1.6 m
```

Et voici la faute que tout le monde commet une fois :

```dart
// FAUX : on croit écrire « une balle de 25 pixels de rayon ».
// On écrit en réalité « une balle de 25 MÈTRES de rayon ».
final shape = CircleShape()..radius = 25;

// CORRECT avec zoom = 10 : une balle de 25 pixels à l'écran.
final shape = CircleShape()..radius = 2.5;
```

**Symptôme de l'erreur :** les objets se déplacent au ralenti, traversent les murs, ou restent figés. C'est le signe que vos valeurs sont dix à cent fois trop grandes.

Deux façons de régler le zoom :

```dart
// Dans le constructeur
class MonJeuPhysique extends Forge2DGame {
  MonJeuPhysique() : super(zoom: 20, gravity: Vector2(0, 10));
}

// Plus tard, comme n'importe quelle caméra Flame
game.camera.viewfinder.zoom = 20;
```

> **À retenir.** En Forge2D, on ne pense jamais en pixels. On décide d'abord de la taille de son héros **en mètres** (1,6 m est raisonnable), puis on choisit un zoom qui le rend visible à l'écran.

---

## 34.33 — `BodyComponent`, `BodyDef`, `FixtureDef`

`BodyComponent` est le composant Flame qui enveloppe un `Body` de Forge2D. Sa signature réelle :

```dart
BodyComponent({
  Paint? paint,
  Iterable<Component>? children,
  int? priority,
  bool renderBody = true,
  BodyDef? bodyDef,
  List<FixtureDef>? fixtureDefs,
  ComponentKey? key,
});
```

Il y a deux manières de définir le corps, et il faut connaître les deux.

### Manière 1 — passer `bodyDef` et `fixtureDefs` au constructeur

C'est la plus courte, et celle qu'emploie l'exemple officiel de `flame_forge2d`.

```dart
class Balle extends BodyComponent {
  Balle({Vector2? initialPosition})
      : super(
          fixtureDefs: [
            FixtureDef(
              CircleShape()..radius = 5,
              restitution: 0.8,
              friction: 0.4,
            ),
          ],
          bodyDef: BodyDef(
            angularDamping: 0.8,
            position: initialPosition ?? Vector2.zero(),
            type: BodyType.dynamic,
          ),
        );
}
```

### Manière 2 — redéfinir `createBody()`

Nécessaire dès que la construction dépend de données calculées.

```dart
class Mur extends BodyComponent {
  Mur(this._debut, this._fin);

  final Vector2 _debut;
  final Vector2 _fin;

  @override
  Body createBody() {
    final shape = EdgeShape()..set(_debut, _fin);
    final fixtureDef = FixtureDef(shape, friction: 0.3);
    final bodyDef = BodyDef(position: Vector2.zero());

    return world.createBody(bodyDef)..createFixture(fixtureDef);
  }
}
```

### `BodyDef` : ce que le corps **est**

Signature réelle de `forge2d` 0.14.2 :

```dart
BodyDef({
  BodyType type = BodyType.static,
  Object? userData,
  Vector2? position,
  double angle = 0.0,
  Vector2? linearVelocity,
  double angularVelocity = 0.0,
  double linearDamping = 0.0,
  double angularDamping = 0.0,
  bool allowSleep = true,
  bool isAwake = true,
  bool fixedRotation = false,
  bool bullet = false,
  bool active = true,
  Vector2? gravityOverride,
  Vector2? gravityScale,
});
```

Les paramètres qui servent vraiment :

| Paramètre | Usage typique |
| --- | --- |
| `type` | statique, dynamique ou kinématique (section 34.34) |
| `position` | position initiale, **en mètres** |
| `userData` | l'objet Dart associé, indispensable aux contacts |
| `angularDamping` | freine la rotation ; une balle qui tourne sans fin est irréaliste |
| `linearDamping` | freine le déplacement ; simule une résistance de l'air |
| `fixedRotation` | empêche toute rotation : **indispensable pour un personnage** |
| `bullet` | active la détection continue ; contre le tunneling (chapitre 24) |

### `FixtureDef` : ce que le corps **touche**

```dart
FixtureDef(
  Shape shape, {
  Object? userData,
  double friction = 0,
  double restitution = 0,
  double density = 1,
  bool isSensor = false,
  Filter? filter,
});
```

Le premier paramètre est positionnel : la forme. Les quatre formes disponibles :

| Forme | Constructeur | Usage |
| --- | --- | --- |
| `CircleShape` | `CircleShape()..radius = 0.5` | balles, roues, projectiles |
| `PolygonShape` | `PolygonShape()..setAsBoxXY(1, 2)` | caisses, plateformes, corps |
| `EdgeShape` | `EdgeShape()..set(a, b)` | murs et sols **statiques** |
| `ChainShape` | suite de segments | terrain découpé |

`isSensor: true` mérite une mention : une fixture capteur **détecte** les contacts mais ne produit **aucune** réponse physique. C'est ainsi qu'on modélise une zone de déclenchement, un point de passage ou un ramassage d'objet.

Voici un jeu complet et exécutable, directement dérivé de l'exemple officiel. Il ne nécessite aucun asset.

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/extensions.dart';
import 'package:flame/game.dart';
import 'package:flame_forge2d/flame_forge2d.dart';
import 'package:flutter/widgets.dart';

void main() {
  runApp(const GameWidget.managed(gameFactory: BacASable.new));
}

class BacASable extends Forge2DGame {
  BacASable() : super(gravity: Vector2(0, 10), zoom: 10);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    world.add(Balle(initialPosition: Vector2(0, -10)));

    final r = camera.visibleWorldRect;
    world.addAll([
      Bordure(r.topLeft.toVector2(), r.topRight.toVector2()),
      Bordure(r.topRight.toVector2(), r.bottomRight.toVector2()),
      Bordure(r.bottomLeft.toVector2(), r.bottomRight.toVector2()),
      Bordure(r.topLeft.toVector2(), r.bottomLeft.toVector2()),
    ]);
  }
}

class Balle extends BodyComponent with TapCallbacks {
  Balle({Vector2? initialPosition})
      : super(
          fixtureDefs: [
            FixtureDef(
              CircleShape()..radius = 2, // 2 MÈTRES, pas 2 pixels
              restitution: 0.8,
              friction: 0.4,
              density: 1.0,
            ),
          ],
          bodyDef: BodyDef(
            angularDamping: 0.8,
            position: initialPosition ?? Vector2.zero(),
            type: BodyType.dynamic,
          ),
        );

  @override
  void onTapDown(TapDownEvent event) {
    body.applyLinearImpulse(Vector2(0, -300)); // la balle saute
  }
}

class Bordure extends BodyComponent {
  Bordure(this._debut, this._fin);

  final Vector2 _debut;
  final Vector2 _fin;

  @override
  Body createBody() {
    final shape = EdgeShape()..set(_debut, _fin);
    final fixtureDef = FixtureDef(shape, friction: 0.3);
    return world.createBody(BodyDef(position: Vector2.zero()))
      ..createFixture(fixtureDef);
  }
}
```

**Résultat :** une balle tombe, rebondit sur le sol de plus en plus bas, puis s'immobilise. Un appui sur la balle la relance vers le haut.

Notez `renderBody`, qui vaut `true` par défaut : `BodyComponent` dessine le contour de ses fixtures. C'est pour cela que l'exemple est visible sans le moindre sprite. Dans un vrai jeu, on met `renderBody = false` et on ajoute un `SpriteComponent` en enfant.

Enfin, une règle absolue héritée de Box2D :

> **Un corps physique n'est jamais l'enfant d'un autre corps.** Forge2D ne connaît pas la notion de corps imbriqués. Tous les `BodyComponent` s'ajoutent directement au `world` : `world.add(Arme())`, jamais `joueur.add(Arme())`.

---

## 34.34 — Corps statique, dynamique, kinématique

`BodyType` compte exactement trois valeurs, et le choix conditionne tout le comportement.

| Type | Masse | Vitesse | Déplacé par | Collisionne avec |
| --- | --- | --- | --- | --- |
| `static` | nulle | nulle | vous, à la main | dynamiques uniquement |
| `kinematic` | nulle | fixée par vous | sa propre vitesse | dynamiques uniquement |
| `dynamic` | positive | calculée | les forces et la gravité | tout |

Trois conséquences pratiques à mémoriser. **Deux corps statiques ne se détectent jamais** : un mur et un sol peuvent se recouvrir. **Un corps kinématique ne peut pas être arrêté** : il traverse les murs statiques, mais pousse les corps dynamiques, ce qui est exactement le comportement attendu d'un ascenseur. **Un corps statique déplacé à la main ne réveille pas ses voisins** : les caisses posées dessus peuvent rester en l'air, car elles sont endormies (`allowSleep`).

```dart
// Le sol : ne bouge pas, ne subit rien.
final sol = BodyDef(type: BodyType.static, position: Vector2(0, 20));

// L'ascenseur : monte à vitesse constante, indifférent aux chocs.
final ascenseur = BodyDef(
  type: BodyType.kinematic,
  position: Vector2(5, 15),
  linearVelocity: Vector2(0, -2), // 2 mètres par seconde vers le haut
);

// Le tonneau : subit tout.
final tonneau = BodyDef(
  type: BodyType.dynamic,
  position: Vector2(3, 0),
  angularDamping: 0.5,
);

// Le héros : dynamique, mais il ne doit JAMAIS basculer sur le côté.
final heros = BodyDef(
  type: BodyType.dynamic,
  position: Vector2(1, 1),
  fixedRotation: true,
);
```

`fixedRotation: true` sur le héros est la première chose que l'on oublie, et le symptôme est mémorable : le personnage tombe, roule sur le dos et continue la partie couché.

---

## 34.35 — Densité, friction, restitution

Trois nombres suffisent à caractériser la matière d'un objet. Ils se règlent sur la `FixtureDef`.

### `density` — la densité

Unité : kilogrammes par mètre carré. Valeur par défaut : `1`. La masse du corps est calculée par le moteur : **masse = densité × surface**. On ne fixe donc jamais une masse directement.

```dart
FixtureDef(shape, density: 0.2);  // du bois flottant, léger
FixtureDef(shape, density: 1.0);  // valeur neutre
FixtureDef(shape, density: 7.8);  // de l'acier
```

Une densité de `0` est autorisée : le corps a alors une masse nulle et se comporte comme un corps statique, même déclaré dynamique. C'est presque toujours une erreur.

### `friction` — le frottement

Plage utile : `0` à `1`. Valeur par défaut : `0`. C'est la résistance au glissement le long d'une surface. Le moteur combine les frictions des deux fixtures en contact.

```dart
FixtureDef(shape, friction: 0.0);  // glace : rien ne s'arrête
FixtureDef(shape, friction: 0.3);  // valeur générale
FixtureDef(shape, friction: 0.9);  // caoutchouc sur béton
```

Attention : la valeur par défaut est `0`. Un sol créé sans préciser la friction est une patinoire, et vos caisses glisseront indéfiniment. Précisez toujours une friction sur les sols.

### `restitution` — l'élasticité

Plage : `0` à `1`. Valeur par défaut : `0`. C'est la proportion de vitesse restituée après un choc.

```text
  restitution = 0.0   la balle tombe et reste au sol  (argile)
  restitution = 0.5   elle remonte à la moitié        (balle de tennis)
  restitution = 0.8   elle remonte aux 4/5            (balle rebondissante)
  restitution = 1.0   elle remonte à la même hauteur, indéfiniment
```

Une restitution supérieure à `1` ferait gagner de l'énergie à chaque rebond : la simulation exploserait. Ne le faites pas.

Récapitulatif des matériaux du « Donjon de Dart » :

| Matière | `density` | `friction` | `restitution` |
| --- | --- | --- | --- |
| Sol de pierre | statique | `0.6` | `0.0` |
| Caisse de bois | `0.6` | `0.5` | `0.1` |
| Tonneau métallique | `2.0` | `0.4` | `0.2` |
| Balle de feu | `0.3` | `0.1` | `0.85` |
| Glace | statique | `0.02` | `0.0` |
| Toile d'araignée | statique | `0.98` | `0.0` |

---

## 34.36 — Les joints

Un **joint** est une contrainte permanente entre deux corps. Le moteur la maintient à chaque pas de simulation, quoi qu'il arrive. C'est ce que Flame seul ne saura jamais faire.

Les joints disponibles dans `forge2d` 0.14.2, avec leur classe de définition associée :

| Joint | Classe de définition | Ce qu'il contraint | Exemple de jeu |
| --- | --- | --- | --- |
| `RevoluteJoint` | `RevoluteJointDef` | rotation autour d'un point commun | charnière de porte, coude |
| `PrismaticJoint` | `PrismaticJointDef` | translation le long d'un axe | piston, plateforme sur rail |
| `DistanceJoint` | `DistanceJointDef` | distance fixe entre deux points | barre rigide, ressort |
| `RopeJoint` | `RopeJointDef` | distance **maximale** | corde, chaîne |
| `WheelJoint` | `WheelJointDef` | rotation + suspension | roue de véhicule |
| `WeldJoint` | `WeldJointDef` | soudure complète | objet cassable en morceaux |
| `MotorJoint` | `MotorJointDef` | déplacement relatif imposé | objet téléguidé |
| `MouseJoint` | `MouseJointDef` | attire un point vers une cible | attraper un objet à la souris |
| `PulleyJoint` | `PulleyJointDef` | poulie : ce qui monte d'un côté descend de l'autre | contrepoids |
| `GearJoint` | `GearJointDef` | engrenage entre deux joints | mécanisme d'horlogerie |
| `FrictionJoint` | `FrictionJointDef` | freine translation et rotation | vue de dessus, sol rugueux |
| `ConstantVolumeJoint` | `ConstantVolumeJointDef` | volume constant d'un ensemble | corps mou, bulle |

Le principe de création est identique pour tous : on remplit une définition, on la donne au monde.

```text
  1. créer les deux corps
  2. créer un XxxJointDef
  3. renseigner bodyA, bodyB et les points d'ancrage
  4. world.createJoint(def)
```

> **Remarque de prudence.** Les signatures exactes des classes `XxxJointDef` varient d'une version de `forge2d` à l'autre, et elles ne sont pas documentées dans la documentation de Flame. Si vous utilisez des joints, lisez le dartdoc de la version **exacte** que votre `pubspec.lock` a installée. Ce cours ne s'appuiera pas sur eux.

---

## 34.37 — Pourquoi le « Donjon de Dart » n'utilisera PAS Forge2D

Décision assumée, et voici les six raisons.

**1. Le jeu n'en a pas besoin.** Le héros marche, saute, frappe. Les gobelins patrouillent. Les potions se ramassent. Aucun de ces comportements ne demande une simulation de corps rigides. Tout se fait avec la vélocité du chapitre 23 et les hitbox du chapitre 32.

**2. Le contrôle du personnage devient plus difficile, pas plus facile.** Un héros piloté par la physique n'a pas les réponses nettes qu'attend un joueur : il glisse, il rebondit, il met du temps à s'arrêter. Les jeux de plateforme commerciaux pilotent presque toujours leur personnage **hors** du moteur physique, précisément pour garder ce contrôle.

**3. Le changement d'unités coûte cher.** Toutes les valeurs du cours — vitesses en pixels par seconde, tailles en pixels, positions issues de Tiled — devraient être converties en mètres. Chaque chapitre précédent serait à réécrire.

**4. Le débogage devient opaque.** Quand un héros traverse un mur en Flame, vous inspectez deux rectangles. Quand un corps traverse un mur en Forge2D, il faut raisonner sur le pas de simulation, la vitesse maximale, le mode `bullet` et la densité. Ce n'est pas un savoir de débutant.

**5. La dépendance est en version `0.x`.** Une montée de version mineure peut casser l'API. Pour un cours qui doit rester valable, c'est un risque inutile.

**6. Le poids et les performances.** Forge2D simule des contacts et résout des contraintes à chaque frame, même quand rien ne bouge. Sur un téléphone d'entrée de gamme, un donjon avec deux cents corps coûte cher pour un bénéfice nul.

Ce que le « Donjon de Dart » utilisera à la place :

| Besoin | Solution retenue | Chapitre |
| --- | --- | --- |
| Gravité et saut | vélocité + accélération constante | 23 |
| Blocage contre les murs | résolution AABB avec correction de position | 24 |
| Détection des contacts | `HasCollisionDetection` + hitbox | 32 |
| Recul après un coup | vélocité imposée pendant 0,2 s | 37 |
| Objets qui tombent | vélocité verticale simple | 23 |

> **À retenir.** Choisir un outil, c'est aussi savoir refuser ceux qui coûtent plus qu'ils ne rapportent. Vous savez maintenant que Forge2D existe, ce qu'il fait, et pourquoi ce jeu-là s'en passe.

---

## 34.38 — Comment choisir : tableau de décision

Posez-vous les questions dans cet ordre. La première réponse « oui » décide.

```text
  1. Ai-je besoin de piles d'objets stables, de véhicules,
     de cordes, de pendules, de ragdolls ?
        oui ──► FORGE2D
        non ──► question 2

  2. Ai-je besoin de rebonds calculés avec des angles
     et des masses réalistes ?
        oui ──► FORGE2D
        non ──► question 3

  3. Ai-je plus de 30 objets qui s'entrechoquent
     en permanence entre eux ?
        oui ──► FORGE2D
        non ──► question 4

  4. Mon jeu est-il un jeu de plateforme, un jeu de tir,
     un jeu de rôle ou un jeu de puzzle sur grille ?
        oui ──► FLAME SEUL
        non ──► question 5

  5. Suis-je capable d'expliquer ce qu'est une impulsion
     linéaire et pourquoi la densité vaut 1 par défaut ?
        non ──► FLAME SEUL (revenez plus tard)
        oui ──► FORGE2D si vous y tenez
```

Et le tableau de synthèse, à garder :

| Critère | Flame seul | Forge2D |
| --- | --- | --- |
| Courbe d'apprentissage | douce | raide |
| Unités | pixels | mètres, kilogrammes, secondes |
| Contrôle du personnage | total et immédiat | indirect, via forces |
| Rebonds réalistes | à coder | fournis |
| Empilements stables | quasi impossible | fournis |
| Cordes, ponts, véhicules | impossible | joints |
| Coût processeur | faible | modéré à élevé |
| Débogage | simple | difficile |
| Stabilité de l'API | `flame` en 1.x | `flame_forge2d` en 0.x |
| Convient à | plateforme, tir, RPG, puzzle | bac à sable physique, billard, *Angry Birds* |

---

## 34.39 — Bilan de la PARTIE 2B

Huit chapitres, un moteur. Récapitulons ce que chacun a apporté au « Donjon de Dart ».

| Ch. | Titre | Apports principaux | Ce que cela remplace (PARTIE 2A) |
| --- | --- | --- | --- |
| 27 | Installer Flame et premier `FlameGame` | `flame` dans le `pubspec.yaml`, `FlameGame`, `GameWidget`, `onLoad`, `update`, `render` | la boucle `Ticker` écrite à la main (ch. 20) |
| 28 | Components et cycle de vie | `Component`, `PositionComponent`, arbre de composants, `add` / `removeFromParent`, `priority`, `anchor` | la liste d'entités et les files d'ajout / suppression (ch. 26) |
| 29 | Sprites, animations et assets | `SpriteComponent`, `SpriteAnimationComponent`, `Flame.images`, `assets/images/` | le découpage manuel de sprite sheet (ch. 22) |
| 30 | Entrées : clavier, tactile, joystick | `KeyboardHandler`, `TapCallbacks`, `DragCallbacks`, `JoystickComponent`, `HudButtonComponent` | l'écoute clavier brute et le gestionnaire d'entrées (ch. 26) |
| 31 | Caméra, viewport et monde | `CameraComponent`, `World`, `Viewport`, `Viewfinder`, `follow`, `zoom`, HUD | la caméra `canvas.translate` faite main (ch. 25) |
| 32 | Collisions et `CollisionCallbacks` | `HasCollisionDetection`, `RectangleHitbox`, `CircleHitbox`, `onCollisionStart` | les tests AABB écrits à la main (ch. 24) |
| 33 | Effets, particules et timers | `MoveEffect`, `ScaleEffect`, `OpacityEffect`, `SequenceEffect`, `ParticleSystemComponent`, `TimerComponent` | les interpolations et compteurs manuels (ch. 20, 23) |
| 34 | Audio, Tiled et physique avancée | `flame_audio`, `FlameAudio.bgm`, `AudioPool`, `flame_tiled`, `TiledComponent`, aperçu de `flame_forge2d` | l'absence de son, la carte en `List<String>` (ch. 25) |

> **À retenir.** Flame ne vous a rien appris de nouveau sur le fond : boucle de jeu, delta time, sprites, collisions, caméra sont exactement les notions de la PARTIE 2A. Il vous a évité de les réécrire. C'est parce que vous les aviez écrites à la main que vous savez ce que Flame fait à votre place, et ce qu'il ne fait pas.

---

## 34.40 — Le `pubspec.yaml` final du projet

Voici le fichier complet du « Donjon de Dart », tel qu'il sera à l'ouverture de la PARTIE 2C.

```yaml
name: donjon_de_dart
description: Un jeu de donjon 2D réalisé avec Flutter et Flame.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=3.12.0 <4.0.0"
  flutter: ">=3.44.0"

dependencies:
  flutter:
    sdk: flutter

  # Moteur de jeu
  flame: ^1.38.0

  # Paquets-ponts réellement utilisés par le projet
  flame_audio: ^2.12.2
  flame_tiled: ^3.1.2

  # Sauvegarde du meilleur score (chapitre 40)
  shared_preferences: ^2.3.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  assets:
    # Images : dossier attendu par défaut par Flame.images et Sprite.load
    - assets/images/
    # Audio : dossier attendu par défaut par FlameAudio
    - assets/audio/
    # Cartes Tiled : dossier attendu par défaut par TiledComponent.load
    - assets/tiles/
```

Trois remarques sur ce fichier.

**`flame_forge2d` n'y figure pas**, conformément à la décision de la section 34.37. Si un jour vous en avez besoin, ajoutez `flame_forge2d: ^0.19.3+7`.

**Les trois dossiers d'assets sont déclarés, même vides.** Un dossier déclaré mais inexistant provoque une erreur au build. Créez-les, et déposez-y au minimum un fichier `.gitkeep` si vous versionnez le projet.

**Les versions sont volontairement en `^`.** Cela autorise les correctifs (`1.38.1`) mais interdit les ruptures (`2.0.0`). Le fichier `pubspec.lock`, lui, fige les versions exactes : versionnez-le pour qu'un camarade obtienne le même environnement que vous.

L'arborescence, elle, suit la séparation du chapitre 26 : `lib/jeu/` pour la classe de jeu et les états, `lib/composants/` pour le héros, le gobelin, le coffre et la porte, `lib/niveaux/` pour `source_de_niveau.dart`, `source_tiled.dart` et `source_texte.dart`, `lib/services/` pour `service_audio.dart` et `service_donnees.dart`, `lib/ecrans/` pour les menus, et `test/` pour les tests des services.

Cette structure est celle que le chapitre 35 mettra en place fichier par fichier.

---

## 34.41 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: "assets/audio/assets/audio/epee.mp3"` | chemin complet passé à `FlameAudio.play` | passer `'epee.mp3'` : Flame ajoute le préfixe `assets/audio/` |
| Le premier son sort avec 150 ms de retard | fichier non préchargé | `await FlameAudio.audioCache.loadAll([...])` dans `onLoad` |
| Aucun son sur le Web au chargement | politique d'interaction du navigateur | démarrer la musique après un clic du joueur, pas dans `onLoad` |
| La musique continue quand l'application passe en arrière-plan | `FlameAudio.bgm.initialize()` jamais appelé | l'appeler dans `onLoad`, avant tout `bgm.play` |
| Revenir de la pause redémarre la musique au début | `stop()` puis `play()` au lieu de `resume()` | `pause()` / `resume()` pour une pause, `stop()` / `play()` pour un changement de morceau |
| Le jeu reprend tout seul derrière le menu pause | `lifecycleStateChange` appelle `resumeEngine()` sans condition | mémoriser une pause voulue par le joueur dans un booléen |
| Le son se coupe quand on le rejoue vite | un seul lecteur réutilisé | `FlameAudio.createPool(...)` puis `pool.start()` |
| `Unable to load asset` sur le `.tmx` | dossier `assets/tiles/` non déclaré dans le `pubspec.yaml` | ajouter `- assets/tiles/` sous `flutter: assets:` |
| L'image du tileset ne s'affiche pas, la carte est vide | l'image n'est pas dans `assets/tiles/` ou n'est pas déclarée | placer le `.png` à côté du `.tmx` et redémarrer complètement l'application |
| `getLayer<ObjectGroup>('Entites')` renvoie `null` | nom de calque erroné ou mauvais type générique | vérifier le nom **exact** dans Tiled (sensible à la casse) et le type demandé |
| `objet.type` est vide alors que Tiled affiche une classe | Tiled ≥ 1.9 remplit `class_`, pas `type` | tester `o.class_.isNotEmpty ? o.class_ : o.type` |
| `properties.getValue<int>('vitesse')` renvoie `null` | la propriété est déclarée `float` dans Tiled | lire en `double`, ou changer le type dans Tiled |
| Seule la moitié droite de la carte est visible | `Viewfinder` centré par défaut | `camera.viewfinder.anchor = Anchor.topLeft` |
| Le héros traverse tous les murs Tiled | calque `Collisions` non converti en hitbox | générer un `Mur` par objet du calque et l'ajouter au monde |
| En Forge2D, les objets se déplacent au ralenti et traversent tout | valeurs saisies en pixels au lieu de mètres | diviser par le `zoom` : un rayon de 25 px vaut `2.5` avec `zoom: 10` |
| Le personnage Forge2D tombe et roule sur le dos | rotation non bloquée | `BodyDef(fixedRotation: true)` |
| Les caisses glissent sans jamais s'arrêter | `friction` vaut `0` par défaut | préciser `FixtureDef(shape, friction: 0.5)` sur le sol et les objets |
| `beginContact` n'est jamais appelé | `userData` non renseigné | `BodyDef(userData: this)` sur les deux corps concernés |
| Les corps physiques ne bougent pas ensemble | `BodyComponent` ajouté comme enfant d'un autre | tous les corps s'ajoutent à `world`, jamais imbriqués |

---

## 34.42 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `flame_audio` | paquet-pont vers `audioplayers` ; dossier par défaut `assets/audio/` |
| `FlameAudio.play` | effet court, paramètre `volume` (`0.0` à `1.0`), renvoie un `AudioPlayer` |
| `FlameAudio.loop` | même chose, en boucle sans blanc |
| `playLongAudio` | fichier long diffusé en flux ; peut faire chuter le framerate |
| `audioCache.loadAll` | préchargement, **avec `await`**, dans `onLoad` ; supprime la latence du premier son |
| `FlameAudio.bgm` | musique de fond globale ; `initialize()` obligatoire dans `onLoad` |
| `pause` / `resume` | conservent la position ; `stop` / `play` repartent du début |
| `lifecycleStateChange` | point d'entrée du cycle de vie de l'application ; `paused`, `hidden`, `resumed`, `detached` |
| `AudioPool` | plusieurs lecteurs préchargés du même son ; `FlameAudio.createPool`, puis `start()` |
| Service audio | un enum d'effets, deux volumes, deux interrupteurs, un mode silencieux |
| Repli sans son | tout retour sonore doit avoir un jumeau visuel |
| Tiled | éditeur de niveaux 2D ; produit un fichier `.tmx` en XML |
| Tileset | une image, des tuiles numérotées à partir de 1 ; `0` signifie « vide » |
| Calque de tuiles | grille de GID ; `tileData[y][x].tile` |
| Calque d'objets | liste libre de `TiledObject` avec `x`, `y`, `type` / `class_` et `properties` |
| `flame_tiled` | `TiledComponent.load(fichier, destTileSize)` ; préfixe `assets/tiles/` |
| `getLayer<T>` | récupère un calque par son nom ; renvoie `null` si absent |
| `properties.getValue<T>` | lit une propriété personnalisée typée ; toujours doubler d'un `??` |
| Collisions Tiled | un calque d'objets `Collisions` → un `Mur` avec `RectangleHitbox` passive |
| Repli sans Tiled | une seconde implémentation du même contrat `SourceDeNiveau` |
| Forge2D | portage Dart de Box2D ; `World`, `Body`, `Shape`, `Fixture`, `Joint` |
| `Forge2DGame` | `gravity` en Y positif vers le bas, `zoom = 10` par défaut |
| Unités | Forge2D raisonne en **mètres**, jamais en pixels |
| `BodyType` | `static`, `kinematic`, `dynamic` |
| `density` / `friction` / `restitution` | masse par surface, glissement, rebond |
| Joints | contraintes entre deux corps : charnière, glissière, corde, roue… |
| Décision | Flame seul pour plateforme, tir, RPG, puzzle ; Forge2D pour empilements et véhicules |

---

## 34.43 — Exercices

Tous les exercices sont conçus pour être faisables **sans fichier audio et sans Tiled installé**.

### Exercice 1 — Le catalogue de sons (facile)
Écrivez un enum `Effet` avec cinq valeurs et une constante `Map<Effet, String>` associant chaque effet à un nom de fichier. Écrivez une fonction `String? fichierDe(Effet e)` et affichez les cinq correspondances.

### Exercice 2 — Le calcul du volume final (facile)
Écrivez une fonction `double volumeFinal(double reglageJoueur, double equilibrage)` qui renvoie le produit des deux, borné entre `0.0` et `1.0`. Testez-la avec `(0.6, 0.8)`, `(1.0, 1.5)` et `(-0.2, 0.5)`.

### Exercice 3 — Le journal audio (facile)
Écrivez une classe `ServiceAudio` en mode silencieux (`actif = false`) qui enregistre chaque appel à `jouer(Effet)` dans une liste. Jouez six sons et affichez le nombre d'occurrences de chaque effet.

### Exercice 4 — La machine à états musicale (moyen)
Écrivez une fonction `List<String> actionsMusique(EtatJeu ancien, EtatJeu nouvel)` qui renvoie la liste des appels à effectuer (`'pause'`, `'resume'`, `'stop'`, `'play:menu'`, `'play:donjon'`). Vérifiez que `pause → enJeu` produit `['resume']` et que `menu → enJeu` produit `['stop', 'play:donjon']`.

### Exercice 5 — Le double verrou de pause (moyen)
Écrivez une classe `EtatMoteur` avec deux booléens `pauseJoueur` et `arrierePlan`, et un getter `bool get actif`. Affichez la table de vérité des quatre combinaisons.

### Exercice 6 — Lire un calque de tuiles à la main (moyen)
Sans `flame_tiled`, écrivez une fonction qui reçoit une chaîne CSV de GID, une largeur et une hauteur, et renvoie une `List<List<int>>`. Affichez la grille avec `.` pour `0` et `#` sinon.

### Exercice 7 — Les propriétés typées (moyen)
Écrivez une classe `ProprietesSimples` qui enveloppe une `Map<String, Object>` et expose `T? getValue<T>(String nom)`, `bool has(String nom)`, plus des aides `texte`, `entier`, `reel`, `booleen` avec valeurs par défaut. Testez les cas de type incorrect.

### Exercice 8 — Le convertisseur pixels / mètres (moyen)
Écrivez une classe `Echelle` avec un champ `zoom`, et les méthodes `double versMetres(double pixels)` et `double versPixels(double metres)`. Affichez un tableau de conversion pour un zoom de 10 et de 20.

### Exercice 9 — Fusionner les murs en bandes (difficile)
À partir d'une carte en `List<String>`, produisez la liste des rectangles de collision en **fusionnant horizontalement** les `#` consécutifs d'une même ligne. Comparez le nombre de rectangles obtenus avec le nombre de `#`.

### Exercice 10 — Le chargeur de niveau complet (difficile)
Assemblez : une abstraction `SourceDeNiveau`, une implémentation `SourceTexte` qui lit une `List<String>`, la fusion horizontale de l'exercice 9, la lecture des entités avec leurs propriétés par défaut, et un rapport imprimé. Le tout doit s'exécuter en console, sans Flutter.

---

## 34.44 — Corrections des exercices

### Correction 1

```dart
enum Effet { epee, piece, degat, porte, victoire }

const Map<Effet, String> fichiers = {
  Effet.epee: 'epee.mp3',
  Effet.piece: 'piece.mp3',
  Effet.degat: 'degat.mp3',
  Effet.porte: 'porte.mp3',
  Effet.victoire: 'victoire.mp3',
};

String? fichierDe(Effet e) => fichiers[e];

void main() {
  for (final e in Effet.values) {
    print('${e.name.padRight(10)} -> ${fichierDe(e)}');
  }
}
```

**Résultat :**

```text
epee       -> epee.mp3
piece      -> piece.mp3
degat      -> degat.mp3
porte      -> porte.mp3
victoire   -> victoire.mp3
```

**Explication :** l'enum est le seul identifiant connu du reste du jeu ; la `Map` est la seule à connaître les noms de fichiers. Le jour où `epee.mp3` devient `coup_epee.ogg`, une seule ligne change. `fichierDe` renvoie un `String?` parce qu'une `Map` interrogée avec une clé absente renvoie `null` (chapitre 12).

---

### Correction 2

```dart
double volumeFinal(double reglageJoueur, double equilibrage) {
  final brut = reglageJoueur * equilibrage;
  return brut.clamp(0.0, 1.0);
}

void main() {
  print(volumeFinal(0.6, 0.8));
  print(volumeFinal(1.0, 1.5));
  print(volumeFinal(-0.2, 0.5));
}
```

**Résultat :**

```text
0.48
1.0
0.0
```

**Explication :** `clamp` borne la valeur sans écrire de `if`. Le bornage n'est pas cosmétique : un volume supérieur à `1.0` n'est pas garanti par toutes les plateformes, et un volume négatif est une valeur invalide. Comme `reglageJoueur` viendra d'un curseur manipulé par l'utilisateur, on ne fait jamais confiance à son intervalle.

---

### Correction 3

```dart
enum Effet { epee, piece, degat }

class ServiceAudio {
  ServiceAudio({this.actif = true});

  final bool actif;
  final List<Effet> journal = [];

  void jouer(Effet e) {
    journal.add(e);
    if (!actif) return;
    // Ici viendrait FlameAudio.play(...)
  }

  Map<Effet, int> compter() {
    final resultat = <Effet, int>{};
    for (final e in journal) {
      resultat[e] = (resultat[e] ?? 0) + 1;
    }
    return resultat;
  }
}

void main() {
  final audio = ServiceAudio(actif: false);

  audio.jouer(Effet.epee);
  audio.jouer(Effet.piece);
  audio.jouer(Effet.epee);
  audio.jouer(Effet.degat);
  audio.jouer(Effet.epee);
  audio.jouer(Effet.piece);

  print('Journal : ${audio.journal.length} événements');
  audio.compter().forEach((e, n) => print('${e.name.padRight(8)} : $n'));
}
```

**Résultat :**

```text
Journal : 6 événements
epee     : 3
piece    : 2
degat    : 1
```

**Explication :** le journal est ajouté **avant** le test `actif`, ce qui rend la classe testable en mode silencieux : c'est tout l'intérêt. `compter` applique le motif d'accumulation dans une `Map` vu au chapitre 6, avec `(resultat[e] ?? 0) + 1` pour gérer la première occurrence.

---

### Correction 4

```dart
enum EtatJeu { menu, enJeu, pause, gameOver }

List<String> actionsMusique(EtatJeu ancien, EtatJeu nouvel) {
  if (ancien == nouvel) return const [];

  switch (nouvel) {
    case EtatJeu.menu:
      return ['stop', 'play:menu'];
    case EtatJeu.enJeu:
      if (ancien == EtatJeu.pause) return ['resume'];
      return ['stop', 'play:donjon'];
    case EtatJeu.pause:
      return ['pause'];
    case EtatJeu.gameOver:
      return ['stop'];
  }
}

void main() {
  const transitions = [
    [EtatJeu.menu, EtatJeu.enJeu],
    [EtatJeu.enJeu, EtatJeu.pause],
    [EtatJeu.pause, EtatJeu.enJeu],
    [EtatJeu.enJeu, EtatJeu.gameOver],
    [EtatJeu.gameOver, EtatJeu.menu],
    [EtatJeu.menu, EtatJeu.menu],
  ];

  for (final t in transitions) {
    final actions = actionsMusique(t[0], t[1]);
    print('${t[0].name} -> ${t[1].name} : $actions');
  }
}
```

**Résultat :**

```text
menu -> enJeu : [stop, play:donjon]
enJeu -> pause : [pause]
pause -> enJeu : [resume]
enJeu -> gameOver : [stop]
gameOver -> menu : [stop, play:menu]
menu -> menu : []
```

**Explication :** renvoyer une **liste d'actions** au lieu d'appeler directement `FlameAudio` rend la logique musicale testable en console, sans Flutter ni fichier audio. Le cas `pause → enJeu` est traité à part : c'est le seul qui exige `resume` plutôt que `stop` + `play`. Le garde `ancien == nouvel` évite de relancer la musique lors d'un changement d'état sans effet.

---

### Correction 5

```dart
class EtatMoteur {
  bool pauseJoueur = false;
  bool arrierePlan = false;

  /// Le moteur ne tourne que si AUCUNE source ne demande la pause.
  bool get actif => !pauseJoueur && !arrierePlan;
}

void main() {
  final m = EtatMoteur();

  print('pauseJoueur | arrierePlan | moteur');
  print('------------+-------------+--------');
  for (final pj in [false, true]) {
    for (final ap in [false, true]) {
      m.pauseJoueur = pj;
      m.arrierePlan = ap;
      final etat = m.actif ? 'actif' : 'en pause';
      print('${pj.toString().padRight(11)} | '
          '${ap.toString().padRight(11)} | $etat');
    }
  }
}
```

**Résultat :**

```text
pauseJoueur | arrierePlan | moteur
------------+-------------+--------
false       | false       | actif
false       | true        | en pause
true        | false       | en pause
true        | true        | en pause
```

**Explication :** le bug classique consiste à appeler `resumeEngine()` au retour au premier plan sans vérifier la pause volontaire du joueur. Modéliser les deux sources séparément, avec un ET logique, supprime le bug par construction : il n'existe plus de chemin de code capable de reprendre le jeu alors que le menu pause est affiché.

---

### Correction 6

```dart
List<List<int>> lireCsv(String csv, int largeur, int hauteur) {
  final valeurs = csv
      .split(',')
      .map((s) => s.trim())
      .where((s) => s.isNotEmpty)
      .map(int.parse)
      .toList();

  if (valeurs.length != largeur * hauteur) {
    throw FormatException(
      'Attendu ${largeur * hauteur} valeurs, reçu ${valeurs.length}.',
    );
  }

  return List.generate(
    hauteur,
    (y) => List.generate(largeur, (x) => valeurs[y * largeur + x]),
  );
}

void afficher(List<List<int>> grille) {
  for (final ligne in grille) {
    print(ligne.map((g) => g == 0 ? '.' : '#').join());
  }
}

void main() {
  const csv = '''
2,2,2,2,2,2,2,2,
2,0,0,0,0,0,0,2,
2,0,0,2,2,0,0,2,
2,0,0,0,0,0,0,2,
2,0,0,0,0,0,0,2,
2,2,2,2,2,2,2,2''';

  final grille = lireCsv(csv, 8, 6);
  afficher(grille);
  print('Tuiles non vides : '
      '${grille.expand((l) => l).where((g) => g != 0).length}');
}
```

**Résultat :**

```text
########
#......#
#..##..#
#......#
#......#
########
Tuiles non vides : 30
```

**Explication :** c'est exactement ce que fait le paquet `tiled` en interne pour un calque encodé en CSV. La conversion d'un index plat en coordonnées `(x, y)` s'écrit `valeurs[y * largeur + x]` : c'est la formule du chapitre 25, et elle réapparaîtra chaque fois que vous manipulerez une grille. Le contrôle de longueur transforme une carte corrompue en `FormatException` lisible plutôt qu'en `RangeError` obscur.

---

### Correction 7

```dart
class ProprietesSimples {
  const ProprietesSimples(this._valeurs);

  final Map<String, Object> _valeurs;

  bool has(String nom) => _valeurs.containsKey(nom);

  T? getValue<T>(String nom) {
    final v = _valeurs[nom];
    return v is T ? v : null;
  }

  String texte(String nom, {String defaut = ''}) =>
      getValue<String>(nom) ?? defaut;

  int entier(String nom, {int defaut = 0}) => getValue<int>(nom) ?? defaut;

  double reel(String nom, {double defaut = 0}) =>
      getValue<double>(nom) ?? defaut;

  bool booleen(String nom, {bool defaut = false}) =>
      getValue<bool>(nom) ?? defaut;
}

void main() {
  const p = ProprietesSimples({
    'contenu': 'potion',
    'or': 25,
    'poids': 3.5,
    'verrouille': true,
  });

  print(p.texte('contenu'));
  print(p.entier('or'));
  print(p.reel('poids'));
  print(p.booleen('verrouille'));

  // Type incorrect : la propriété existe mais n'est pas un int.
  print('poids en int : ${p.getValue<int>('poids')}');
  // Propriété absente : valeur par défaut.
  print('pv          : ${p.entier('pv', defaut: 10)}');
  print('has(pv)     : ${p.has('pv')}');
}
```

**Résultat :**

```text
potion
25
3.5
true
poids en int : null
pv          : 10
has(pv)     : false
```

**Explication :** `v is T ? v : null` reproduit fidèlement le comportement de `CustomProperties.getValue<T>` du paquet `tiled` : la méthode renvoie `null` aussi bien pour une propriété absente que pour une propriété de type incompatible. La ligne `poids en int : null` illustre le piège int/float de la section 34.26. Les quatre aides typées suppriment tous les `??` du code appelant.

---

### Correction 8

```dart
class Echelle {
  const Echelle(this.zoom);

  /// Nombre de pixels par mètre.
  final double zoom;

  double versMetres(double pixels) => pixels / zoom;
  double versPixels(double metres) => metres * zoom;
}

void main() {
  const tailles = {
    'héros': 32.0,
    'gobelin': 24.0,
    'caisse': 16.0,
    'boss': 96.0,
  };

  for (final zoom in [10.0, 20.0]) {
    final e = Echelle(zoom);
    print('--- zoom = ${zoom.toInt()} px/m ---');
    tailles.forEach((nom, px) {
      final m = e.versMetres(px);
      print('${nom.padRight(8)} $px px  =  $m m  '
          '(retour : ${e.versPixels(m)} px)');
    });
  }
}
```

**Résultat :**

```text
--- zoom = 10 px/m ---
héros    32.0 px  =  3.2 m  (retour : 32.0 px)
gobelin  24.0 px  =  2.4 m  (retour : 24.0 px)
caisse   16.0 px  =  1.6 m  (retour : 16.0 px)
boss     96.0 px  =  9.6 m  (retour : 96.0 px)
--- zoom = 20 px/m ---
héros    32.0 px  =  1.6 m  (retour : 32.0 px)
gobelin  24.0 px  =  1.2 m  (retour : 24.0 px)
caisse   16.0 px  =  1.2000000000000002 m  (retour : 24.0 px)
boss     96.0 px  =  4.8 m  (retour : 96.0 px)
```

**Explication :** avec un zoom de 10, un héros de 32 pixels mesurerait 3,2 mètres, ce qui est trop grand pour Forge2D et rendrait la simulation lourde. Avec un zoom de 20, il fait 1,6 mètre : une taille humaine, dans la plage où le moteur est stable. Notez au passage `1.2000000000000002` : c'est l'arithmétique flottante du chapitre 2, et c'est une raison de plus de ne jamais comparer deux `double` avec `==`.

---

### Correction 9

```dart
class RectMur {
  const RectMur(this.x, this.y, this.largeur, this.hauteur);
  final double x, y, largeur, hauteur;

  @override
  String toString() => 'Rect(x: $x, y: $y, l: $largeur, h: $hauteur)';
}

List<RectMur> fusionnerBandes(List<String> carte, {double tuile = 16}) {
  final murs = <RectMur>[];

  for (var y = 0; y < carte.length; y++) {
    final ligne = carte[y];
    var debut = -1;

    // La boucle va jusqu'à length INCLUS pour fermer une bande
    // qui toucherait le bord droit.
    for (var x = 0; x <= ligne.length; x++) {
      final estMur = x < ligne.length && ligne[x] == '#';

      if (estMur && debut == -1) {
        debut = x;                                   // début de bande
      } else if (!estMur && debut != -1) {
        murs.add(RectMur(debut * tuile, y * tuile,
            (x - debut) * tuile, tuile));            // fin de bande
        debut = -1;
      }
    }
  }
  return murs;
}

void main() {
  const carte = [
    '####################',
    '#..........#.......#',
    '#..P.......#...C...#',
    '#....####..#.......#',
    '#....#..#..........#',
    '####################',
  ];

  final murs = fusionnerBandes(carte);
  final nbDieses =
      carte.map((l) => l.split('#').length - 1).reduce((a, b) => a + b);

  print('Caractères # : $nbDieses');
  print('Rectangles   : ${murs.length}');
  print('Gain         : '
      '${(100 - murs.length * 100 / nbDieses).toStringAsFixed(1)} %');
  print(murs.first);
}
```

**Résultat :**

```text
Caractères # : 76
Rectangles   : 22
Gain         : 71.1 %
Rect(x: 0.0, y: 0.0, l: 320.0, h: 16.0)
```

**Explication :** la boucle interne va jusqu'à `ligne.length` **inclus** pour fermer une bande qui toucherait le bord droit ; sans cette astuce, le dernier mur de chaque ligne serait perdu. Le gain de 71 % n'est pas anecdotique : le moteur teste 22 hitbox au lieu de 76 à chaque frame. Sur un niveau de 100 × 60, l'écart devient décisif. Une fusion verticale supplémentaire serait possible, mais son rapport gain / complexité est bien moins favorable.

---

### Correction 10

```dart
// ---------- Modèle commun, sans Flame ni Tiled ----------

class RectMur {
  const RectMur(this.x, this.y, this.largeur, this.hauteur);
  final double x, y, largeur, hauteur;
}

class DescriptionEntite {
  const DescriptionEntite({
    required this.categorie,
    required this.x,
    required this.y,
    this.donnees = const {},
  });

  final String categorie;
  final double x, y;
  final Map<String, Object> donnees;

  T? valeur<T>(String nom) {
    final v = donnees[nom];
    return v is T ? v : null;
  }
}

class Niveau {
  const Niveau({required this.murs, required this.entites});
  final List<RectMur> murs;
  final List<DescriptionEntite> entites;
}

abstract class SourceDeNiveau {
  Future<Niveau> charger();
}

// ---------- Implémentation texte ----------

class SourceTexte implements SourceDeNiveau {
  const SourceTexte(this.lignes, {this.tuile = 16});

  final List<String> lignes;
  final double tuile;

  static const Map<String, String> _categories = {
    'P': 'apparition', 'G': 'ennemi', 'C': 'coffre',
    'K': 'cle', 'D': 'porte',
  };

  static const Map<String, Map<String, Object>> _defauts = {
    'ennemi': {'pv': 10, 'vitesse': 40.0},
    'coffre': {'contenu': 'or', 'or': 15},
    'porte': {'niveauSuivant': 'donjon2'},
  };

  @override
  Future<Niveau> charger() async =>
      Niveau(murs: _fusionner(), entites: _entites());

  /// Fusion horizontale des '#' : exercice 9.
  List<RectMur> _fusionner() {
    final murs = <RectMur>[];
    for (var y = 0; y < lignes.length; y++) {
      final ligne = lignes[y];
      var debut = -1;
      for (var x = 0; x <= ligne.length; x++) {
        final estMur = x < ligne.length && ligne[x] == '#';
        if (estMur && debut == -1) {
          debut = x;
        } else if (!estMur && debut != -1) {
          murs.add(RectMur(debut * tuile, y * tuile,
              (x - debut) * tuile, tuile));
          debut = -1;
        }
      }
    }
    return murs;
  }

  List<DescriptionEntite> _entites() {
    final entites = <DescriptionEntite>[];
    for (var y = 0; y < lignes.length; y++) {
      for (var x = 0; x < lignes[y].length; x++) {
        final c = _categories[lignes[y][x]];
        if (c == null) continue;
        entites.add(DescriptionEntite(
          categorie: c,
          x: x * tuile,
          y: y * tuile,
          donnees: _defauts[c] ?? const {},
        ));
      }
    }
    return entites;
  }
}

// ---------- Programme ----------

Future<void> main() async {
  const carte = [
    '####################',
    '#..........#.......#',
    '#..P.......#...C...#',
    '#....####..#.......#',
    '#....#..#...G......#',
    '#..K............D..#',
    '####################',
  ];

  final SourceDeNiveau source = const SourceTexte(carte);
  final n = await source.charger();

  print('=== NIVEAU CHARGÉ ===');
  print('Rectangles de collision : ${n.murs.length}');
  print('Entités : ${n.entites.length}');
  for (final e in n.entites) {
    final pv = e.valeur<int>('pv');
    print('  ${e.categorie.padRight(12)} '
        '(${e.x.toInt()}, ${e.y.toInt()})'
        '${pv != null ? " pv:$pv" : ""}');
  }
}
```

**Résultat :**

```text
=== NIVEAU CHARGÉ ===
Rectangles de collision : 22
Entités : 5
  apparition   (48, 32)
  coffre       (240, 32)
  ennemi       (192, 64) pv:10
  cle          (48, 80)
  porte        (256, 80)
```

**Explication :** ce programme est le squelette exact du chargeur de niveaux de la PARTIE 2C. Trois points le rendent réutilisable. D'abord, `Niveau`, `RectMur` et `DescriptionEntite` ne dépendent ni de Flame ni de Tiled : ce sont des données pures, testables en console. Ensuite, `charger()` est `async` alors que `SourceTexte` n'a rien d'asynchrone à faire ; c'est volontaire, car `SourceTiled` devra faire un `await TiledComponent.load(...)` et doit respecter le même contrat. Enfin, `_defauts` joue le rôle des propriétés personnalisées de Tiled : le jour où vous passerez au `.tmx`, elles seront remplacées par `objet.properties`, et rien d'autre ne changera.

---

## Et maintenant ?

La PARTIE 2B est terminée. Faites le compte de ce que vous savez faire.

Vous savez installer Flame et lancer un `FlameGame`. Vous savez construire un arbre de composants et maîtriser leur cycle de vie. Vous savez afficher des sprites et des animations. Vous savez lire le clavier, le doigt et un joystick. Vous savez piloter une caméra sur un monde plus grand que l'écran. Vous savez détecter des collisions et y réagir. Vous savez animer avec des effets, des particules et des minuteurs. Et depuis ce chapitre, vous savez faire sonner votre jeu, charger un niveau dessiné dans un éditeur, et décider en connaissance de cause si un moteur physique vous est nécessaire.

Il vous manque une seule chose, et c'est la plus importante : **un jeu**. Vous avez huit chapitres de démonstrations, chacune exécutable, aucune reliée aux autres.

La PARTIE 2C répare cela. Elle construit le « Donjon de Dart » du premier écran au dernier, en huit chapitres, sans jamais repartir de zéro.

Le chapitre 35 pose les fondations. Vous y créerez la structure de fichiers annoncée en 34.40, vous écrirez la classe `DonjonDeDart` qui portera le jeu entier, vous mettrez en place la machine à états du chapitre 26 avec les `overlays` de Flutter, et vous construirez un vrai menu principal : titre, bouton « Jouer », bouton « Options » avec les réglages de volume de ce chapitre, et navigation entre les écrans. À la fin du chapitre 35, vous aurez une application qui se lance, affiche un menu, entre dans le jeu, se met en pause et revient au menu.

Rendez-vous au chapitre 35 : [35-PARTIE-2C—ARCHITECTURE-DU-JEU-ET-MENU-PRINCIPAL.md](./35-PARTIE-2C—ARCHITECTURE-DU-JEU-ET-MENU-PRINCIPAL.md)
