# PARTIE 2C — LE JEU COMPLET « DONJON DE DART »
# CHAPITRE 35 — ARCHITECTURE DU JEU ET MENU PRINCIPAL

> **Version de Flame utilisée dans ce chapitre :** `flame` **1.38.0** (publiée le 19 juillet 2026).
> **Date de vérification des API :** 8 août 2026, sur `https://docs.flame-engine.org/latest/`,
> `https://pub.dev/packages/flame` et le dépôt `flame-engine/flame` (branche `main`).
> **Contraintes SDK :** `sdk: ">=3.12.0 <4.0.0"`, `flutter: ">=3.44.0"`.
>
> **Niveau :** intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 19 (widgets Flutter, `StatefulWidget`), chapitre 26 (machine à états, organisation des fichiers), chapitre 27 (`FlameGame`, `GameWidget`), chapitre 28 (arbre de composants), chapitre 30 (clavier), chapitre 31 (`World` et `CameraComponent`), chapitre 32 (`HasCollisionDetection`). Côté Dart : chapitre 11 (enums et mixins), chapitre 15 (asynchrone), chapitre 16 (organisation d'un projet).
> **Ce que vous saurez faire à la fin :** créer le squelette complet du projet « Donjon de Dart », écrire la classe `DonjonGame`, piloter le jeu avec une machine à états, afficher des écrans Flutter par-dessus le canvas grâce aux overlays, et livrer une application qui se lance sur un menu principal fonctionnel — sans un seul fichier image.

---

## 35.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- décrire ce que la PARTIE 2C va construire, chapitre par chapitre ;
- rédiger et lire un cahier des charges de jeu ;
- créer l'arborescence complète du projet et justifier chaque dossier ;
- écrire le `pubspec.yaml` d'un projet de jeu Flame et déclarer ses dossiers d'assets ;
- centraliser les valeurs de réglage dans une classe `Constantes` ;
- centraliser les couleurs dans une classe `Palette` et vous passer totalement d'images ;
- définir l'enum `GameState` et dessiner la machine à états correspondante ;
- écrire la classe `DonjonGame` qui hérite de `FlameGame` ;
- choisir les mixins d'un jeu et expliquer le rôle de chacun ;
- réutiliser le `World` et la `CameraComponent` fournis par `FlameGame` ;
- implémenter `changerEtat()` comme unique point de passage entre les écrans ;
- utiliser `pauseEngine()` et `resumeEngine()` au bon moment ;
- afficher un widget Flutter au-dessus du jeu avec un overlay ;
- remplir `overlayBuilderMap` et `initialActiveOverlays` sans vous tromper de pluriel ;
- écrire un menu principal complet en Flutter pur : titre, boutons, dégradé, cadre ;
- désactiver proprement un bouton « Continuer » tant que la fonctionnalité n'existe pas ;
- ouvrir une boîte de dialogue d'options et y brancher le mode debug ;
- quitter l'application selon la plateforme ;
- écrire `demarrerPartie()` qui remet le jeu à zéro et entre dans la partie ;
- distinguer l'écran de chargement de Flame (`loadingBuilder`) et votre propre écran de chargement ;
- écrire le `main.dart` du projet et y créer l'instance de jeu une seule fois ;
- vérifier la navigation menu → jeu → menu ;
- activer le mode debug et ajouter des raccourcis clavier de développement ;
- énoncer précisément ce que le projet fait, et ne fait pas encore, à la fin de ce chapitre.

---

## 35.1 — Ce qu'on va construire dans la PARTIE 2C

Les huit chapitres de la PARTIE 2B vous ont donné huit démonstrations : un carré qui bouge, un arbre de composants, une animation, un joystick, une caméra, des collisions, des particules, un son. Chacune fonctionnait. Aucune ne parlait aux autres.

La PARTIE 2C construit **un seul programme**, du chapitre 35 au chapitre 40, sans jamais repartir de zéro. Ce programme s'appelle **« Donjon de Dart »**. C'est le même univers que le projet console du chapitre 18 : un héros, des gobelins, des potions, des clés, un boss.

### Le jeu fini, en une phrase

Un jeu de plateformes en vue de côté, dans lequel le joueur traverse trois salles de donjon, ramasse des pièces, évite ou combat des ennemis, trouve une clé, ouvre une porte, affronte un boss, et voit son meilleur score conservé d'une session à l'autre.

### Ce que chaque chapitre ajoute

| Chapitre | Ce qui apparaît à l'écran à la fin du chapitre |
| --- | --- |
| 35 | Un menu principal. Un bouton « Jouer » qui entre dans une salle vide. Un retour au menu. |
| 36 | Un héros qui court, saute, tombe, et s'arrête sur les plateformes. |
| 37 | Des gobelins qui patrouillent, des chauves-souris qui poursuivent, des dégâts, des vies. |
| 38 | Des pièces, des potions, des clés, un score, une barre de vie, un HUD. |
| 39 | Trois niveaux enchaînés, des portes verrouillées, un boss de fin. |
| 40 | Des sons, un menu pause, un écran Game Over, un écran de victoire, un meilleur score sauvegardé. |

### La règle du jeu jouable

Ce point est structurant, et il vous concerne directement : **à la fin de chaque chapitre, vous lancez `flutter run` et vous jouez**. Il n'y aura jamais de chapitre qui produit des fragments inutilisables « en attendant le suivant ».

Cette contrainte a un prix. Au chapitre 35, le jeu n'a pas encore de joueur ; il faudra donc bien que le bouton « Jouer » mène quelque part. Il mènera vers une salle de démonstration, construite en dix lignes, que le chapitre 36 remplacera. Ce genre de pièce provisoire s'appelle un *bouchon* (en anglais *stub*). Chaque fois que j'en poserai un, je l'écrirai noir sur blanc, avec le numéro du chapitre qui le retirera.

### Aucune image, aucun son

Vous n'avez aucun fichier à télécharger. Le jeu entier est dessiné avec des rectangles et des cercles colorés, comme au chapitre 28. Ce n'est pas un pis-aller : c'est un choix pédagogique qui vous garantit trois choses.

D'abord, **votre projet compile toujours**. Un chemin d'asset mal orthographié est la première cause d'abandon chez le débutant, et vous n'en aurez aucun avant le chapitre où l'on parlera précisément des assets.

Ensuite, **vous voyez la logique**. Un rectangle rouge qui recule quand il prend un coup montre le comportement sans le déguiser. Beaucoup de bugs de jeu se cachent derrière une jolie animation.

Enfin, **le branchement des vrais assets est trivial**. Toutes les entités du jeu dériveront de `PositionComponent`, exactement comme `SpriteComponent`. Remplacer une forme par un sprite, c'est changer une classe parente et une méthode `render`. Chaque chapitre vous indiquera l'endroit exact.

> **À retenir.** La PARTIE 2C est un projet unique, incrémental, jouable à chaque étape, et exécutable sans aucun fichier binaire.

---

## 35.2 — Le cahier des charges du « Donjon de Dart »

Avant d'écrire une ligne de code, on écrit ce qu'on veut. Un cahier des charges de jeu tient sur une page. Il n'a pas pour but d'être joli, mais de **trancher des questions** avant qu'elles ne coûtent cher.

Le chapitre 41 formalisera l'exercice avec un vrai *Game Design Document*. Ce qui suit en est la version courte, et elle est déjà suffisante pour six chapitres.

### 1. Identité

| Rubrique | Décision |
| --- | --- |
| Titre | Donjon de Dart |
| Genre | Plateformes 2D, vue de côté, un joueur |
| Cible | Support d'apprentissage ; parties de 2 à 5 minutes |
| Plateformes | Web et Android en priorité, Windows/Linux/macOS pour le développement |
| Orientation | Paysage |
| Durée d'une partie | 3 niveaux, environ 5 minutes |

### 2. Boucle de jeu

La *boucle de jeu* (au sens du game design, pas au sens du chapitre 20) est la séquence d'actions que le joueur répète.

```text
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   explorer la salle                                      │
  │        │                                                 │
  │        ▼                                                 │
  │   éviter ou tuer les ennemis                             │
  │        │                                                 │
  │        ▼                                                 │
  │   ramasser pièces et potions  ──► le score monte         │
  │        │                                                 │
  │        ▼                                                 │
  │   trouver la clé                                         │
  │        │                                                 │
  │        ▼                                                 │
  │   ouvrir la porte  ──────────────► niveau suivant  ──────┘
  │                                                          │
  │   (perdre ses 3 vies ──► Game Over ──► retour au menu)   │
  └──────────────────────────────────────────────────────────┘
```

### 3. Règles chiffrées

Ces valeurs ne sont pas arbitraires : elles viennent des sections de la PARTIE 2A où l'on a réglé la gravité et le saut, et elles sont figées dans `lib/config/constantes.dart` dès la section 35.5.

| Règle | Valeur | Où elle sert |
| --- | --- | --- |
| Taille d'une tuile | 32 px | grille des niveaux, ch. 39 |
| Gravité | 1200 px/s² | chute du joueur, ch. 36 |
| Vitesse du joueur | 180 px/s | déplacement horizontal, ch. 36 |
| Force de saut | −520 px/s | impulsion vers le haut, ch. 36 |
| Vitesse maximale de chute | 900 px/s | évite le tunneling, ch. 36 |
| Points de vie du joueur | 100 | barre de vie, ch. 37 et 38 |
| Vies de départ | 3 | Game Over, ch. 37 et 40 |
| Invincibilité après un coup | 1,2 s | clignotement, ch. 37 |
| Zoom caméra | 2,0 | lisibilité du pixel, ch. 39 |
| Nombre de niveaux | 3 | progression, ch. 39 |

### 4. Ce que le jeu ne fera pas

Un cahier des charges qui ne dit pas *non* ne sert à rien. Voici les fonctionnalités explicitement écartées :

- pas de multijoueur, ni local ni en ligne ;
- pas de sauvegarde de la position exacte du joueur (on sauvegarde le meilleur score et le niveau atteint, chapitre 40) ;
- pas d'inventaire complexe : seules les clés se comptent ;
- pas de dialogues, pas de scénario, pas de cinématiques ;
- pas de physique rigide : la physique est écrite à la main (chapitres 23 et 36), `flame_forge2d` n'est pas utilisé ;
- pas de génération procédurale : les trois niveaux sont écrits à la main dans `niveaux_data.dart`.

### 5. Critères d'acceptation

Le jeu sera considéré comme terminé au chapitre 40 si, et seulement si :

1. l'application se lance sur un menu et n'affiche jamais d'écran noir ;
2. le joueur peut terminer les trois niveaux sans blocage ;
3. la perte des trois vies conduit à l'écran Game Over, qui ramène au menu ;
4. le meilleur score survit à la fermeture complète de l'application ;
5. le jeu tourne à 60 images par seconde sur une machine de développement ordinaire ;
6. aucune ressource externe n'est nécessaire pour compiler.

> **À retenir.** Un cahier des charges de jeu tient sur une page et répond à trois questions : que fait le joueur en boucle, avec quels chiffres, et qu'est-ce qu'on refuse de faire.

---

## 35.3 — L'arborescence complète du projet

Voici la structure de fichiers que nous allons construire. Elle est fixée dès maintenant, jusqu'au chapitre 40 inclus. Chaque commentaire indique **le chapitre qui crée le fichier**.

```text
donjon_de_dart/
├── pubspec.yaml                      # ch. 35 — dépendances et assets
├── analysis_options.yaml             # généré par flutter create
├── assets/
│   ├── images/                       # vide pour l'instant (ch. 35)
│   └── audio/                        # vide pour l'instant (ch. 35)
└── lib/
    ├── main.dart                     # ch. 35 — point d'entrée Flutter
    ├── donjon_game.dart              # ch. 35 — la classe FlameGame
    ├── config/
    │   ├── constantes.dart           # ch. 35 — réglages chiffrés + noms d'overlays
    │   └── palette.dart              # ch. 35 — toutes les couleurs du jeu
    ├── core/
    │   ├── game_state.dart           # ch. 35 — enum GameState
    │   ├── entite.dart               # ch. 36 — PositionComponent de base
    │   └── sante.dart                # ch. 37 — mixin Sante
    ├── composants/
    │   ├── joueur.dart               # ch. 36
    │   ├── plateforme.dart           # ch. 36
    │   ├── ennemi.dart               # ch. 37 — classe abstraite
    │   ├── gobelin.dart              # ch. 37
    │   ├── chauvesouris.dart         # ch. 37
    │   ├── projectile.dart           # ch. 37
    │   ├── collectible.dart          # ch. 38 — classe abstraite
    │   ├── piece.dart                # ch. 38
    │   ├── potion.dart               # ch. 38
    │   ├── cle.dart                  # ch. 38
    │   ├── porte.dart                # ch. 39
    │   └── boss.dart                 # ch. 39
    ├── niveaux/
    │   ├── niveau.dart               # ch. 39 — chargement d'une carte
    │   └── niveaux_data.dart         # ch. 39 — les cartes en List<String>
    ├── hud/
    │   ├── hud.dart                  # ch. 38
    │   ├── barre_de_vie.dart         # ch. 38
    │   └── compteur_score.dart       # ch. 38
    ├── services/
    │   ├── audio_service.dart        # ch. 40
    │   └── sauvegarde_service.dart   # ch. 40
    └── ecrans/
        ├── menu_principal.dart       # ch. 35 — menu + écran de chargement
        ├── ecran_pause.dart          # ch. 40
        ├── ecran_game_over.dart      # ch. 40
        └── ecran_victoire.dart       # ch. 40
```

### Pourquoi cette structure et pas une autre

Le chapitre 26 (section 26.30) posait le principe : **on range par rôle technique, pas par écran**. Reprenons chaque dossier.

**`config/`** — les valeurs. Aucune logique, aucune dépendance à Flame. Deux fichiers seulement : les nombres et les couleurs. C'est le dossier qu'on ouvre quand on veut « rendre le jeu plus rapide » ou « changer l'ambiance ».

**`core/`** — les briques transversales, utilisées partout, qui ne sont ni des entités ni des écrans. L'enum d'états, la classe de base des entités, le mixin de santé. Rien dans `core/` ne connaît un gobelin en particulier.

**`composants/`** — tout ce qui vit **dans le monde du jeu**, c'est-à-dire tout ce qui sera ajouté au `World` de Flame. Un fichier par entité. C'est le dossier le plus gros, et c'est normal.

**`niveaux/`** — les données de niveaux et leur chargeur. On sépare les données (`niveaux_data.dart`) du code qui les interprète (`niveau.dart`). C'est exactement le découpage du chapitre 34 (section 34.40) : le jour où les cartes viendront de Tiled, seul `niveau.dart` change.

**`hud/`** — les composants d'interface **dessinés par Flame**, attachés au `viewport` de la caméra (chapitre 31, section 31.19). Attention à ne pas les confondre avec les écrans.

**`services/`** — les objets à longue durée de vie qui parlent au monde extérieur : audio, stockage. Ils sont testables séparément et remplaçables.

**`ecrans/`** — les interfaces **écrites en widgets Flutter**, affichées par-dessus le canvas grâce aux overlays. Un menu, une pause, un Game Over. Aucun de ces fichiers n'importe Flame autrement que pour recevoir la référence au jeu.

### La distinction la plus importante : `hud/` contre `ecrans/`

C'est la source de confusion numéro un dans un projet Flame. Retenez ce tableau.

| | `hud/` | `ecrans/` |
| --- | --- | --- |
| Technologie | composants Flame | widgets Flutter |
| Dessiné par | le `Canvas` du jeu | l'arbre de widgets, par-dessus le jeu |
| Attaché à | `camera.viewport` | `overlays` |
| Mis à jour | à chaque `update(dt)` | par `setState` / reconstruction du widget |
| Gelé par `pauseEngine()` | oui | **non** |
| Exemple | barre de vie, compteur de score | menu, pause, Game Over |
| Chapitre | 38 | 35 et 40 |

La ligne « gelé par `pauseEngine()` » explique tout le reste : un menu de pause doit rester **cliquable quand le moteur est arrêté**, donc il ne peut pas être un composant Flame. Une barre de vie doit être **collée au pixel du jeu et suivre son rythme**, donc elle n'a rien à faire dans un widget.

> **À retenir.** `config/` les valeurs, `core/` les briques, `composants/` le monde, `hud/` l'interface Flame, `ecrans/` l'interface Flutter, `services/` l'extérieur, `niveaux/` les données.

---

## 35.4 — Créer le projet et le `pubspec.yaml`

### Créer le projet

Ouvrez un terminal dans le dossier où vous rangez vos projets, puis :

```bash
flutter create --org com.exemple --platforms=web,android,windows,linux,macos donjon_de_dart
cd donjon_de_dart
```

Trois précisions sur cette commande.

`--org com.exemple` fixe l'identifiant du paquet Android (`com.exemple.donjon_de_dart`). Choisissez un domaine que vous contrôlez si vous comptez publier un jour ; sinon, `com.exemple` convient.

`--platforms=...` limite les dossiers natifs générés. Retirez de la liste les plateformes que vous ne visez pas : chaque plateforme ajoute des fichiers, du temps de compilation, et des messages d'erreur possibles. Le développement se fera surtout sur le bureau (démarrage instantané) et la vérification sur Web et Android.

Le nom du projet doit être en `snake_case`. `donjon_de_dart` est valide ; `DonjonDeDart` provoquerait une erreur, car il deviendra le nom du paquet Dart.

### Installer Flame

```bash
flutter pub add flame
```

Cette commande écrit `flame: ^1.38.0` dans le `pubspec.yaml` et lance `flutter pub get`. Vous avez fait exactement la même chose au chapitre 27.

Nous **n'installons pas** `flame_audio` maintenant. Il sera nécessaire au chapitre 40, et seulement là. Installer une dépendance « au cas où » est une mauvaise habitude : elle alourdit le projet et vous expose à des conflits de version que rien ne justifie.

### Créer les dossiers

```bash
mkdir -p assets/images assets/audio
mkdir -p lib/config lib/core lib/composants lib/niveaux lib/hud lib/services lib/ecrans
```

Les deux dossiers d'assets resteront **vides** jusqu'au bout de ce chapitre. Ce point mérite un avertissement, car il pique tout le monde une fois : Flutter refuse de compiler si le `pubspec.yaml` déclare un dossier d'assets qui **n'existe pas** sur le disque. En revanche, un dossier existant mais vide ne pose aucun problème. Créez donc les dossiers avant d'écrire le `pubspec.yaml`.

Certains outils de gestion de version (dont Git) ne conservent pas les dossiers vides. Si vous versionnez le projet, ajoutez-y un fichier vide nommé `.gitkeep` :

```bash
touch assets/images/.gitkeep assets/audio/.gitkeep
```

### Le `pubspec.yaml` complet

Remplacez le contenu du fichier `pubspec.yaml` généré par celui-ci :

```yaml
name: donjon_de_dart
description: "Jeu de plateformes 2D réalisé avec Flutter et Flame — PARTIE 2C."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=3.12.0 <4.0.0"
  flutter: ">=3.44.0"

dependencies:
  flutter:
    sdk: flutter

  # Moteur de jeu 2D. Fournit FlameGame, Component, CameraComponent,
  # les collisions, les effets et les timers.
  flame: ^1.38.0

  # AJOUTÉ AU CHAPITRE 40 seulement :
  # flame_audio: ^2.12.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  assets:
    # Dossier attendu par défaut par Flame.images et Sprite.load.
    # Vide jusqu'au jour où vous brancherez de vrais sprites.
    - assets/images/
    # Dossier attendu par défaut par FlameAudio (chapitre 40).
    - assets/audio/
```

### Lecture ligne par ligne

| Ligne | Rôle | Piège |
| --- | --- | --- |
| `name:` | nom du paquet Dart ; sert aussi aux imports `package:donjon_de_dart/...` | doit être en `snake_case` |
| `publish_to: 'none'` | interdit une publication accidentelle sur pub.dev | à ne jamais retirer pour un jeu |
| `version: 1.0.0+1` | version visible + numéro de build Android/iOS | le `+1` doit croître à chaque envoi sur un magasin |
| `environment:` | versions minimales de Dart et Flutter | doit être **au moins** aussi permissif que Flame 1.38.0 |
| `flame: ^1.38.0` | le moteur ; `^` autorise 1.38.1 mais pas 2.0.0 | ne figez pas sur `1.38.0` exactement |
| `flutter_lints` | règles d'analyse statique | version indicative, adaptez-la |
| `uses-material-design: true` | embarque la police d'icônes Material | nécessaire pour les `Icon` du menu |
| `assets:` | liste des dossiers embarqués | le `/` final est obligatoire pour un dossier |

Une remarque de fond sur les assets, déjà signalée au chapitre 29 mais qui vaut d'être répétée : **Flame préfixe automatiquement les chemins**. Vous écrirez plus tard `Sprite.load('joueur.png')`, jamais `Sprite.load('assets/images/joueur.png')`. Le préfixe `assets/images/` est celui du cache d'images de Flame ; `assets/audio/` est celui de `FlameAudio`.

### Vérifier

```bash
flutter pub get
```

**Résultat attendu :**

```text
Resolving dependencies...
+ flame 1.38.0
+ ordered_set 8.0.0
+ vector_math 2.1.4
Changed 3 dependencies!
```

Le nombre exact de lignes dépend de ce que votre cache contient déjà. Ce qui compte est l'absence du message `version solving failed`.

> **À retenir.** Créez les dossiers **avant** de les déclarer dans `pubspec.yaml`. Un dossier d'assets vide est autorisé ; un dossier d'assets inexistant fait échouer la compilation.

---

## 35.5 — `lib/config/constantes.dart`

Premier fichier de code du projet. Il ne contient **aucune logique** : uniquement des valeurs.

```dart
/// Toutes les valeurs de réglage du jeu.
///
/// Aucun nombre « magique » ne doit apparaître ailleurs dans le projet :
/// si vous écrivez `1200` dans `joueur.dart`, c'est ici qu'il aurait dû être.
class Constantes {
  // Constructeur privé : la classe ne sert qu'à porter des membres statiques.
  Constantes._();

  /// Côté d'une tuile de niveau, en pixels du monde.
  static const double tailleTuile = 32.0;

  /// Accélération verticale, en pixels par seconde au carré : au bout d'une
  /// seconde de chute libre, la vitesse vaut 1200 px/s.
  static const double gravite = 1200.0;

  /// Vitesse horizontale du joueur, en pixels par seconde.
  static const double vitesseJoueur = 180.0;

  /// Impulsion du saut. NÉGATIVE, car l'axe Y descend (chapitre 21).
  static const double forceSaut = -520.0;

  /// Vitesse de chute maximale, pour éviter le tunneling du chapitre 24.
  static const double vitesseMaxChute = 900.0;

  static const int viesDepart = 3;
  static const double pvJoueurMax = 100.0;

  /// Durée d'invincibilité après des dégâts, en secondes.
  static const double dureeInvincibilite = 1.2;

  /// Zoom de la caméra. Entier, pour un rendu net (section 31.27).
  static const double zoomCamera = 2.0;

  static const int nombreNiveaux = 3;
}
```

### Pourquoi `static const` et pas `final`

Rappel du chapitre 02. `const` signifie « connu à la compilation ». La valeur est gravée dans le binaire, elle n'occupe aucune place en mémoire au démarrage, et le compilateur peut l'utiliser pour simplifier du code. `final` signifie « affecté une seule fois, mais éventuellement au démarrage ».

Pour des nombres écrits en dur, `const` est toujours le bon choix.

### Pourquoi une classe et pas des variables globales

Vous pourriez écrire, en haut d'un fichier :

```dart
const double gravite = 1200.0;
const double vitesseJoueur = 180.0;
```

Cela fonctionne. Mais dans un fichier qui importe dix autres fichiers, `gravite` tout seul ne dit pas d'où il vient, et deux fichiers peuvent définir le même nom, ce qui provoque une erreur d'import ambigu. `Constantes.gravite` est **auto-documenté** : n'importe quel lecteur sait où aller regarder, et la complétion de l'éditeur affiche la liste complète dès que vous tapez `Constantes.`.

### Le constructeur privé

`Constantes._();` déclare un constructeur nommé privé, sans corps. Il a deux effets :

1. il empêche `Constantes()` depuis un autre fichier ;
2. il empêche Dart de générer le constructeur par défaut.

Ce n'est pas obligatoire, mais c'est la convention pour une classe qui n'est qu'un porte-valeurs. Vous verrez le même motif dans `Palette` et `Overlays`.

### La classe `Overlays`, dans le même fichier

Les noms d'overlays sont, eux aussi, des constantes de configuration. Ils vivent donc dans `constantes.dart`, sous la classe `Constantes`. La section 35.17 détaillera leur usage ; voici déjà le code, car le fichier doit être complet dès maintenant.

```dart
/// Noms des overlays Flutter affichés par-dessus le jeu.
///
/// Ce sont de simples chaînes, mais elles servent de clés dans deux endroits
/// différents (`overlayBuilderMap` et `overlays.add`). Les centraliser ici
/// transforme une faute de frappe en erreur de compilation.
class Overlays {
  Overlays._();

  static const String menuPrincipal = 'menu_principal';
  static const String hud = 'hud';
  static const String pause = 'pause';
  static const String gameOver = 'game_over';
  static const String victoire = 'victoire';
  static const String chargement = 'chargement';
}
```

> **À retenir.** Tout nombre qui décrit une règle du jeu va dans `Constantes`. Toute chaîne qui sert de clé technique va dans `Overlays`. Aucune des deux classes ne s'instancie.

---

## 35.6 — `lib/config/palette.dart`

Le jeu n'a aucune image. Toutes ses couleurs sont donc du code, et elles sont toutes ici.

```dart
import 'package:flutter/material.dart';

/// Toutes les couleurs du jeu, et quelques styles de texte.
///
/// Une seule règle : aucune valeur `Color(0xFF...)` ne doit apparaître ailleurs
/// dans le projet. Changer l'ambiance du jeu se fait dans ce fichier seul.
class Palette {
  Palette._();

  // --- Fonds -------------------------------------------------------------
  static const Color fondJeu = Color(0xFF14121C);
  static const Color fondMenu = Color(0xFF0C0A12);
  static const Color panneau = Color(0xFF1E1B2B);

  // --- Décor -------------------------------------------------------------
  static const Color mur = Color(0xFF3A3550);
  static const Color plateforme = Color(0xFF6B5B3E);

  // --- Entités et objets --------------------------------------------------
  // gobelin, chauvesouris, boss, projectile, piece, potion, cle, barreVie,
  // barreEnergie... : la liste complète figure en section 35.32.
  static const Color joueur = Color(0xFF4FC3F7);

  // --- Interface ---------------------------------------------------------
  static const Color accent = Color(0xFFFFB300);
  static const Color accentSombre = Color(0xFF8D6E00);
  static const Color texte = Color(0xFFF3F1FA);
  static const Color texteFaible = Color(0xFF9E99B5);
  static const Color danger = Color(0xFFE53935);

  // --- Styles de texte ---------------------------------------------------
  static const TextStyle titre = TextStyle(
    fontSize: 52,
    fontWeight: FontWeight.w900,
    letterSpacing: 6,
    color: texte,
  );

  static const TextStyle sousTitre = TextStyle(
    fontSize: 16,
    letterSpacing: 3,
    color: texteFaible,
  );

  /// Fabrique une `Paint` opaque de la couleur demandée.
  ///
  /// Pour un composant qui dessine à 60 images par seconde, stockez le
  /// résultat dans un champ `static final` plutôt que de rappeler la méthode.
  static Paint peinture(Color couleur) => Paint()..color = couleur;
}
```

### Le format `Color(0xFF14121C)`

Rappel du chapitre 21. Une couleur s'écrit sur 32 bits, en hexadécimal, dans l'ordre **A R G B** :

```text
   0x  FF   14   12   1C
       ▲    ▲    ▲    ▲
       │    │    │    └── bleu   : 0x1C = 28
       │    │    └─────── vert   : 0x12 = 18
       │    └──────────── rouge  : 0x14 = 20
       └───────────────── alpha  : 0xFF = 255 (opaque)
```

Une erreur classique consiste à écrire `Color(0x14121C)` en oubliant l'alpha. Le canal alpha vaut alors `0x00`, donc **totalement transparent**, et vous cherchez pendant vingt minutes pourquoi votre mur ne s'affiche pas.

### Pourquoi les styles de texte sont ici aussi

`TextStyle` est un objet Flutter, pas Flame. Le mettre dans `Palette` est un choix : cela garde tout ce qui touche à l'apparence dans un seul fichier. Un projet plus gros séparerait `palette.dart` (couleurs) et `theme.dart` (styles). Pour six chapitres, un fichier suffit, et vous n'aurez jamais à vous demander où est le style du titre.

### Une remarque sur le nom `Palette`

Flame possède une classe `BasicPalette` dans `package:flame/palette.dart`. Nous ne l'utilisons pas, et nous n'importons jamais ce fichier. Si vous voyez un jour une erreur d'import ambigu sur le mot `Palette`, c'est que vous avez ajouté `import 'package:flame/palette.dart';` par mégarde : retirez-le.

> **À retenir.** Toutes les couleurs du jeu tiennent dans un fichier de cent lignes. Le canal alpha `FF` en tête d'un `Color` n'est pas décoratif : sans lui, vous dessinez du vide.

---

## 35.7 — Pourquoi centraliser les constantes

Cette section n'apporte aucune ligne de code. Elle justifie les deux précédentes, et elle vaut d'être lue, car la centralisation est une discipline qu'on abandonne au premier moment de fatigue.

### Le problème du nombre magique

Un *nombre magique* est une valeur écrite directement dans le code, sans nom.

```dart
void update(double dt) {
  velocite.y += 1200 * dt;
  if (velocite.y > 900) {
    velocite.y = 900;
  }
}
```

Trois défauts, du plus visible au plus grave. **On ne sait pas ce que c'est** : `1200` est-ce une gravité, une largeur, un score ? **On ne peut pas le régler** : pour essayer une gravité plus faible, il faut retrouver toutes les occurrences, y compris celles qui ne devaient pas bouger. **On ne peut pas garantir la cohérence** : le jour où le joueur tombe avec 1200 et le gobelin avec 1150 parce que quelqu'un a mal recopié, vous aurez un bug que personne ne saura reproduire.

La même méthode avec les constantes :

```dart
void update(double dt) {
  velocite.y += Constantes.gravite * dt;
  if (velocite.y > Constantes.vitesseMaxChute) {
    velocite.y = Constantes.vitesseMaxChute;
  }
}
```

Le code se lit à voix haute : « la vitesse verticale augmente de la gravité fois dt, sans dépasser la vitesse maximale de chute ». Il n'y a plus rien à commenter.

### Le réglage du jeu (« game feel »)

Un jeu de plateformes se juge à des détails de sensation : la hauteur du saut, la vitesse de course, le temps de réaction. Ces détails se trouvent **par essai**, pas par calcul. Vous allez modifier `forceSaut` une trentaine de fois avant d'être satisfait.

Avec un fichier de constantes, chaque essai coûte : ouvrir `constantes.dart`, changer un nombre, sauvegarder, regarder le hot reload. Sans ce fichier, chaque essai coûte : retrouver le bon fichier, retrouver la bonne ligne, ne pas se tromper de valeur.

Cette différence de coût décide de la qualité finale du jeu. Ce n'est pas une exagération : un développeur qui met deux minutes à tester une valeur en testera cinq ; un développeur qui met cinq secondes en testera cinquante.

### Le tableau de bord

Il existe un troisième bénéfice, que vous mesurerez au chapitre 39. Ouvrez `constantes.dart` : vous lisez **les règles du jeu** en vingt lignes. C'est le document le plus utile du projet pour quelqu'un qui le découvre — plus utile que n'importe quel README.

### Ce qu'on ne met PAS dans `Constantes`

Attention à ne pas transformer le fichier en dépotoir. Trois exclusions.

| N'y mettez pas | Pourquoi | Où alors |
| --- | --- | --- |
| Les couleurs | ce sont des valeurs d'apparence, pas de règle | `Palette` |
| Les valeurs propres à **une seule** entité (ex. : le rayon de la chauve-souris) | personne d'autre ne s'en sert | champ `static const` dans la classe concernée |
| Les valeurs modifiables par le joueur (volume, difficulté) | ce sont des **réglages**, pas des constantes | `SauvegardeService`, chapitre 40 |

La ligne de partage est simple : `Constantes` contient ce qui est **fixé par le concepteur** et **partagé par plusieurs fichiers**.

> **À retenir.** Un nombre magique est un bug en attente. Le vrai bénéfice de la centralisation n'est pas l'élégance : c'est la vitesse à laquelle vous pouvez régler votre jeu.

---

## 35.8 — `lib/core/game_state.dart` : l'enum `GameState`

Le chapitre 26 (sections 26.16 à 26.20) a établi qu'un jeu est une **machine à états**, et que l'écran affiché est la conséquence de l'état courant. Nous reprenons cette idée telle quelle, avec les enums du chapitre 11.

```dart
/// Les états possibles du jeu.
///
/// À tout instant, `DonjonGame.etat` vaut exactement une de ces six valeurs.
/// L'état détermine : quel overlay est affiché, si le moteur tourne, et quelles
/// touches sont écoutées.
enum GameState {
  /// Ressources en cours de préparation, ou niveau en cours de construction.
  chargement,

  /// Menu principal. Le monde est vide, le moteur peut tourner à vide.
  menu,

  /// Une partie est en cours. C'est le seul état où le monde évolue.
  enJeu,

  /// Partie interrompue volontairement par le joueur (chapitre 40).
  pause,

  /// Le joueur a perdu sa dernière vie (chapitre 40).
  gameOver,

  /// Le joueur a terminé le dernier niveau (chapitre 40).
  victoire,
}
```

### Une extension pour les questions fréquentes

Les enums Dart acceptent des extensions (chapitre 11). Plutôt que d'écrire partout `if (etat == GameState.enJeu || etat == GameState.pause)`, on nomme la question une fois.

```dart
extension GameStateInfos on GameState {
  /// Vrai si une partie existe, qu'elle soit en cours ou suspendue.
  ///
  /// Sert par exemple à décider si le bouton « Reprendre » a un sens.
  bool get partieEnCours =>
      this == GameState.enJeu || this == GameState.pause;

  /// Vrai si le moteur de jeu doit avancer dans cet état.
  bool get moteurActif => this == GameState.enJeu;

  /// Libellé lisible, utilisé par les messages de debug et la barre de titre.
  String get libelle => switch (this) {
        GameState.chargement => 'Chargement',
        GameState.menu => 'Menu principal',
        GameState.enJeu => 'En jeu',
        GameState.pause => 'Pause',
        GameState.gameOver => 'Game Over',
        GameState.victoire => 'Victoire',
      };
}
```

Trois choses à noter.

`switch` est ici une **expression** (Dart 3), pas une instruction : elle produit une valeur, d'où le `=>`. Comme tous les cas de l'enum sont couverts, aucune branche `default` n'est nécessaire, et l'analyseur vous préviendra le jour où vous ajouterez un septième état.

`moteurActif` n'est pas utilisé tel quel dans ce chapitre, mais il documente une règle du jeu : **un seul état fait tourner le moteur**. Le code de `changerEtat()` en section 35.13 dit exactement la même chose autrement.

`this` désigne la valeur d'enum sur laquelle l'extension est appelée. `GameState.pause.libelle` renvoie `'Pause'`.

### Pourquoi un enum plutôt que des booléens

L'alternative naïve est une collection de drapeaux :

```dart
bool enPause = false;
bool dansLeMenu = true;
bool partieFinie = false;
```

Avec trois booléens, il existe **huit** combinaisons, dont six n'ont aucun sens (`enPause && dansLeMenu`, par exemple). Rien n'empêche le code d'y tomber, et le débogage devient une enquête.

Avec un enum, il existe exactement six états, tous valides, et **un seul à la fois**. C'est le même raisonnement qu'au chapitre 26, et c'est la raison pour laquelle tous les moteurs de jeu sérieux utilisent des machines à états.

> **À retenir.** Un enum ne dit pas seulement ce qui est possible : il dit surtout ce qui est **impossible**. C'est là toute sa valeur.

---

## 35.9 — La machine à états du jeu

Voici les transitions autorisées, et **uniquement** celles-là. Toute flèche absente de ce schéma est un bug si elle se produit.

```text
                        ┌────────────────┐
                        │   CHARGEMENT   │   état initial
                        └───────┬────────┘
                                │  onLoad() terminé
                                ▼
        ┌──────────────────►┌────────┐◄──────────────────────┐
        │                   │  MENU  │                       │
        │                   └───┬────┘                       │
        │                       │ bouton « Jouer »           │
        │                       ▼                            │
        │              ┌────────────────┐                    │
        │              │   CHARGEMENT   │  (construction du  │
        │              └───────┬────────┘   niveau)          │
        │                      │ niveau prêt                 │
        │                      ▼                             │
        │   Échap /     ┌────────────┐   bouton « Menu »     │
        └───────────────┤   EN JEU   ├───────────────────────┘
        │               └──┬──┬───┬──┘
        │      touche P /  │  │   │  dernier niveau terminé
        │      bouton pause │  │   │
        │                  ▼  │   ▼
        │           ┌────────┐│ ┌───────────┐
        │           │ PAUSE  ││ │ VICTOIRE  │
        │           └───┬────┘│ └─────┬─────┘
        │   « Reprendre»│     │       │ « Menu »
        │               └─────┘       │
        │                  ▲          │
        │                  │          │
        │           plus aucune vie   │
        │                  │          │
        │            ┌─────▼─────┐    │
        └────────────┤ GAME OVER │◄───┘  (non : voir note)
                     └───────────┘
```

Ce schéma en une seule figure devient vite illisible. La table de transitions, elle, reste exacte et se vérifie ligne à ligne.

### Table des transitions autorisées

| Depuis | Vers | Déclencheur | Chapitre |
| --- | --- | --- | --- |
| `chargement` | `menu` | fin de `onLoad()` | 35 |
| `menu` | `chargement` | bouton « Jouer » (`demarrerPartie`) | 35 |
| `chargement` | `enJeu` | niveau construit | 35 |
| `enJeu` | `menu` | touche Échap, ou bouton « Menu » du HUD | 35 |
| `enJeu` | `pause` | touche P, ou bouton pause du HUD | 40 |
| `pause` | `enJeu` | bouton « Reprendre » | 40 |
| `pause` | `menu` | bouton « Quitter la partie » | 40 |
| `enJeu` | `gameOver` | `vies` tombe à zéro | 40 |
| `enJeu` | `victoire` | dernier niveau terminé | 40 |
| `gameOver` | `menu` | bouton « Menu » | 40 |
| `gameOver` | `chargement` | bouton « Rejouer » | 40 |
| `victoire` | `menu` | bouton « Menu » | 40 |

### Les transitions interdites, et pourquoi

| Transition | Pourquoi elle est interdite |
| --- | --- |
| `menu` → `pause` | on ne met pas en pause ce qui ne tourne pas |
| `menu` → `gameOver` | il n'y a pas de partie à perdre |
| `pause` → `gameOver` | le moteur est arrêté, aucune vie ne peut être perdue |
| `gameOver` → `enJeu` | il faut repasser par `chargement` pour reconstruire le niveau |
| `victoire` → `enJeu` | même raison |
| `chargement` → `pause` | rien n'est encore construit |

Vous remarquerez la règle générale : **on entre toujours dans `enJeu` par `chargement`**, jamais directement, sauf depuis `pause`. Cette règle garantit qu'une partie ne démarre jamais sur les restes de la précédente.

### Ce que ce chapitre implémente

Seules trois transitions sont réellement utilisées au chapitre 35 :

```text
  chargement ──► menu ──► chargement ──► enJeu ──► menu ──► ...
```

Les états `pause`, `gameOver` et `victoire` existent dans l'enum, sont gérés par `changerEtat()`, mais ne sont **jamais atteints** avant le chapitre 40. C'est volontaire : la machine à états est écrite en entier une fois pour toutes, et les chapitres suivants n'auront qu'à déclencher les transitions.

> **À retenir.** La machine à états se dessine **avant** de coder. Une transition non prévue au tableau est un bug, pas une fonctionnalité.

---

## 35.10 — `lib/donjon_game.dart` : la classe `DonjonGame`

Voici le cœur du projet. C'est le fichier que tous les autres connaîtront, et le seul dont on peut dire qu'il « est » le jeu.

Nous allons l'écrire par morceaux dans les sections 35.10 à 35.14, puis vous en trouverez la version complète dans la section « Code complet du chapitre ».

### La déclaration

```dart
import 'package:flame/camera.dart';
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import 'config/constantes.dart';
import 'config/palette.dart';
import 'core/game_state.dart';

/// Le jeu « Donjon de Dart ».
///
/// Une seule instance existe pendant toute la vie de l'application : elle est
/// créée dans `main.dart` et confiée au `GameWidget`.
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  /// Le conteneur de tout ce qui vit dans le jeu.
  ///
  /// Simple alias du `world` fourni par `FlameGame`, affecté dans `onLoad()`.
  late final World monde;

  /// L'état courant. Ne le modifiez JAMAIS directement : passez par
  /// `changerEtat()`, qui s'occupe aussi des overlays et du moteur.
  GameState etat = GameState.chargement;

  int score = 0;                            // chapitre 38
  int vies = Constantes.viesDepart;         // chapitre 37
  int niveauCourant = 0;                    // chapitre 39
  int meilleurScore = 0;                    // chapitre 40

  /// Couleur du canvas, derrière le monde.
  @override
  Color backgroundColor() => Palette.fondJeu;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // `world` et `camera` sont créés par FlameGame : on ne les construit pas,
    // on les configure.
    monde = world;

    camera.viewfinder.anchor = Anchor.center;
    camera.viewfinder.zoom = Constantes.zoomCamera;

    // Le chargement des ressources se ferait ici (chapitres 29 et 40).
    changerEtat(GameState.menu);
  }
}
```

### Les champs, un par un

| Champ | Type | Réinitialisé quand ? | Chapitre qui s'en sert |
| --- | --- | --- | --- |
| `monde` | `World` | jamais (alias) | tous |
| `etat` | `GameState` | à chaque transition | 35 à 40 |
| `score` | `int` | au début de chaque partie | 38 |
| `vies` | `int` | au début de chaque partie | 37, 40 |
| `niveauCourant` | `int` | au début de chaque partie | 39 |
| `meilleurScore` | `int` | jamais (persistant) | 40 |

La distinction entre les quatre premiers et le dernier est importante : `meilleurScore` **survit** aux parties, et sera lu depuis le stockage du téléphone au chapitre 40. Les autres appartiennent à la partie en cours.

### Pourquoi `late final World monde` et pas un second `World`

Voilà un point où beaucoup de débutants se blessent. `FlameGame` **crée déjà** un `World` et une `CameraComponent`, les relie l'un à l'autre et les ajoute à l'arbre. C'est écrit dans le code source du moteur, et c'est ce que le chapitre 31 a montré.

Donc :

- il ne faut **pas** écrire `final World monde = World();`, ce qui créerait un second monde que la caméra ne regarde pas ;
- il ne faut **pas** redéclarer le champ `camera` : il existe déjà sur `FlameGame`, et le masquer casserait le moteur.

Le champ `monde` est donc **un alias**, affecté une fois dans `onLoad()`. Le mot-clé `late` autorise cette affectation différée, et `final` garantit qu'on ne l'affecte qu'une fois (chapitre 12 sur le null safety).

Le bénéfice est purement de lisibilité : `monde.add(gobelin)` se lit mieux que `world.add(gobelin)` dans un projet francophone, et les chapitres 36 à 39 en useront constamment. Si vous préférez écrire `world` partout, supprimez simplement l'alias : rien d'autre ne changera.

### Un seul jeu, une seule instance

Un piège classique consiste à écrire, dans le `build()` d'un widget :

```dart
// À NE PAS FAIRE
Widget build(BuildContext context) => GameWidget(game: DonjonGame());
```

`build()` peut être appelé des dizaines de fois par seconde. Chaque appel construirait un nouveau jeu, perdrait le score, relancerait `onLoad()`, et ferait clignoter l'écran. La section 35.26 montre la parade : créer l'instance dans `initState()` d'un `StatefulWidget`.

> **À retenir.** `DonjonGame` configure le `World` et la `CameraComponent` que `FlameGame` a déjà créés. On ne les recrée jamais.

---

## 35.11 — Les mixins du jeu et pourquoi chacun est là

La déclaration complète est :

```dart
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
```

Deux mixins seulement. Chacun est là pour une raison précise, et chacun a un coût.

### `HasCollisionDetection`

Vu au chapitre 32. Ce mixin installe dans le jeu un **système de détection de collisions** qui, à chaque tick, compare les hitbox de l'arbre et appelle `onCollisionStart`, `onCollision` et `onCollisionEnd` sur les composants concernés.

Sans lui, vous pouvez ajouter autant de `RectangleHitbox` que vous voulez : **rien** ne sera jamais détecté. C'est l'erreur numéro un des débutants sur Flame, et elle ne produit aucun message : simplement, le héros traverse les murs.

Nous le posons **dès maintenant**, alors qu'aucune hitbox n'existe encore, pour deux raisons : le chapitre 36 en aura besoin dès sa première ligne, et l'ajouter plus tard obligerait à modifier une classe déjà écrite, ce que la règle de cohérence de la PARTIE 2C interdit.

Rappel important du chapitre 32 : la détection est prise en charge par **l'ancêtre le plus proche** portant le mixin. Ici c'est le jeu lui-même, donc toutes les hitbox de l'arbre sont couvertes, y compris — si vous en mettiez — celles du HUD. Une variante consiste à poser le mixin sur le `World` pour exclure le HUD ; nous ne le faisons pas, car notre HUD ne contiendra jamais de hitbox.

### `HasKeyboardHandlerComponents`

Vu au chapitre 30. Ce mixin fait **descendre les événements clavier dans l'arbre de composants**, vers tous ceux qui portent `KeyboardHandler`.

Le chapitre 36 en dépend entièrement : `Joueur` portera `KeyboardHandler` et lira les touches de déplacement lui-même. Sans le mixin sur le jeu, `Joueur.onKeyEvent` ne serait jamais appelé.

Le mixin fournit également, au niveau du jeu, une méthode `onKeyEvent` que l'on peut surcharger pour les touches **globales**. C'est exactement ce que nous ferons en section 35.28 pour les raccourcis de développement.

### L'avertissement à ne pas oublier

La documentation de Flame est explicite : `HasKeyboardHandlerComponents` et `KeyboardEvents` ne doivent **jamais** être posés ensemble sur le même jeu. Le second est la version « tout au niveau du jeu », le premier la version « distribuée aux composants ». Les mélanger produit un conflit de méthodes.

```dart
// FAUX : les deux mixins déclarent onKeyEvent
class DonjonGame extends FlameGame
    with KeyboardEvents, HasKeyboardHandlerComponents {}
```

### Les mixins que nous n'utilisons pas

Il existe une petite dizaine d'autres mixins de jeu. Voici pourquoi aucun n'est ici.

| Mixin | Pourquoi il n'est pas utilisé |
| --- | --- |
| `KeyboardEvents` | remplacé par `HasKeyboardHandlerComponents` (incompatibles) |
| `HasTappables`, `HasDraggables` | **n'existent plus** : le système moderne utilise `TapCallbacks` / `DragCallbacks` sur les composants (chapitre 30) |
| `TapDetector`, `PanDetector` | détecteurs au niveau jeu ; nos boutons seront des widgets Flutter ou des `TapCallbacks` |
| `HasPerformanceTracker` | outil de mesure, utile au chapitre 42 seulement |
| `SingleGameInstance` | optimisation avancée ; inutile ici |
| `HasTimeScale` | ralenti / accéléré global ; hors sujet |

### La règle générale

Un mixin de jeu coûte du temps machine à chaque tick, même quand rien ne s'en sert. `HasCollisionDetection` parcourt les hitbox soixante fois par seconde. N'ajoutez un mixin que lorsque vous savez précisément quel composant en dépend.

> **À retenir.** Deux mixins, deux raisons : les collisions du chapitre 36, le clavier du chapitre 36. Tout autre mixin serait du poids mort.

---

## 35.12 — Le `World` et la `CameraComponent`

Rappel condensé du chapitre 31, appliqué au projet.

### Ce que `FlameGame` fournit sans qu'on demande

```text
   DonjonGame  (FlameGame)
   ├── world     : World            ← le contenu du jeu
   └── camera    : CameraComponent  ← le point de vue
        ├── viewport    : MaxViewport   (la fenêtre à l'écran)
        ├── viewfinder  : Viewfinder    (où l'on regarde, et à quel zoom)
        └── backdrop    : Component     (l'arrière-plan fixe)
```

Le `World` **ne se dessine pas lui-même**. Il est dessiné parce qu'une caméra le regarde. C'est cette séparation qui permet d'avoir plusieurs vues du même monde (minimap, écran partagé — section 31.24).

### Les deux réglages du projet

```dart
camera.viewfinder.anchor = Anchor.center;
camera.viewfinder.zoom = Constantes.zoomCamera; // 2.0
```

**`anchor = Anchor.center`** signifie : le point du monde visé par la caméra apparaît **au centre de l'écran**. C'est le réglage naturel d'un jeu de plateformes, où le héros doit rester au milieu.

Au chapitre 27, nous avions au contraire écrit `Anchor.topLeft`, pour que le coin haut-gauche du monde coïncide avec le coin haut-gauche de l'écran. C'est pratique pour une démonstration statique, inutilisable pour un jeu qui défile.

**`zoom = 2.0`** signifie : un pixel du monde occupe deux pixels à l'écran. Nos entités feront environ 24 à 32 unités de monde ; sans zoom, elles paraîtraient minuscules sur un écran moderne. La section 31.27 expliquait pourquoi il faut un zoom **entier** (2, 3, 4) et jamais 1,7 : à zoom non entier, les bords des rectangles bavent d'un demi-pixel et l'image devient floue.

### Où ajouter quoi

C'est la règle la plus utile de tout le chapitre 31, et elle sera appliquée mécaniquement jusqu'au chapitre 40.

| Ce que vous ajoutez | Où | Effet |
| --- | --- | --- |
| Joueur, ennemis, murs, pièces | `monde.add(...)` | se déplace avec la caméra, subit le zoom |
| Barre de vie, score (composants Flame) | `camera.viewport.add(...)` | **fixe à l'écran**, ignore le zoom |
| Parallaxe, ciel | `camera.backdrop.add(...)` | derrière le monde, fixe |
| Menu, pause, Game Over (widgets Flutter) | `overlays.add(...)` | par-dessus tout, cliquable même en pause |
| Écouteurs globaux, minuteurs, services | `add(...)` sur le jeu | invisible, non affecté par la caméra |

Une erreur fréquente consiste à écrire `add(joueur)` au lieu de `monde.add(joueur)`. Le code compile, le joueur s'affiche… et il ne bouge pas quand la caméra se déplace, parce qu'il n'est pas dans le monde observé. Ce bug se manifeste au chapitre 39, quand le niveau devient plus grand que l'écran, c'est-à-dire très loin de sa cause.

### Position initiale de la caméra

```dart
camera.viewfinder.position = Vector2.zero();
```

Cette ligne appartient à `demarrerPartie()` (section 35.24) et non à `onLoad()`. Elle recentre la caméra sur l'origine du monde à chaque nouvelle partie. Au chapitre 36, elle sera remplacée par `camera.follow(joueur)`.

> **À retenir.** `monde` pour ce qui vit, `camera.viewport` pour le HUD Flame, `overlays` pour les écrans Flutter. Trois destinations, jamais interchangeables.

---

## 35.13 — `changerEtat()` : le cœur de la navigation

Voici la méthode la plus importante du fichier. Toute la navigation du jeu passe par elle : il n'y a **pas** de `Navigator.push`, pas de `setState` dispersés, pas de drapeaux booléens.

### Le principe

```text
   changerEtat(nouvelEtat)
        │
        ├─ 1. si c'est déjà l'état courant : ne rien faire
        ├─ 2. mémoriser l'ancien état
        ├─ 3. affecter le nouvel état
        ├─ 4. retirer l'overlay de l'ancien état
        ├─ 5. ajouter l'overlay du nouvel état
        ├─ 6. mettre le moteur en pause ou le relancer
        └─ 7. effectuer le nettoyage propre à l'état d'arrivée
```

### La correspondance état → overlay

Un état correspond exactement à un overlay. Écrivons cette correspondance une seule fois, sous forme de fonction pure.

```dart
  /// L'overlay Flutter associé à chaque état.
  ///
  /// Fonction pure : elle ne modifie rien, elle traduit.
  static String overlayDeLEtat(GameState etat) => switch (etat) {
        GameState.chargement => Overlays.chargement,
        GameState.menu => Overlays.menuPrincipal,
        GameState.enJeu => Overlays.hud,
        GameState.pause => Overlays.pause,
        GameState.gameOver => Overlays.gameOver,
        GameState.victoire => Overlays.victoire,
      };
```

Le `switch` est **exhaustif** sur l'enum : si vous ajoutez un état, le compilateur refusera ce code tant que vous n'aurez pas donné son overlay. C'est exactement le genre de garantie qu'on veut sur une machine à états.

### La méthode

```dart
  /// Change l'état du jeu. UNIQUE point d'entrée de la navigation.
  ///
  /// Ne modifiez jamais `etat` directement : les overlays et le moteur ne
  /// suivraient pas.
  void changerEtat(GameState nouvelEtat) {
    // 1. Une transition vers soi-même n'a pas de sens : on sort.
    if (nouvelEtat == etat) {
      return;
    }

    // 2 et 3. On mémorise l'ancien état AVANT de le remplacer.
    final GameState ancienEtat = etat;
    etat = nouvelEtat;

    if (debugMode) {
      debugPrint('[DonjonGame] ${ancienEtat.libelle} -> ${nouvelEtat.libelle}');
    }

    // 4 et 5. Un seul overlay d'état est visible à la fois.
    overlays.remove(overlayDeLEtat(ancienEtat));
    overlays.add(overlayDeLEtat(nouvelEtat));

    // 6. Le moteur ne tourne que pendant le jeu.
    if (nouvelEtat == GameState.enJeu) {
      resumeEngine();
    } else if (nouvelEtat == GameState.pause ||
        nouvelEtat == GameState.gameOver ||
        nouvelEtat == GameState.victoire) {
      pauseEngine();
    }

    // 7. Nettoyage propre à l'état d'arrivée.
    if (nouvelEtat == GameState.menu) {
      viderLeMonde();
    }
  }
```

### Pourquoi ce garde-fou en première ligne

`if (nouvelEtat == etat) return;` évite un bug discret. Sans lui, un double appel à `changerEtat(GameState.menu)` retirerait l'overlay du menu (étape 4) puis le rajouterait (étape 5). Selon l'ordre de reconstruction des widgets, cela peut provoquer un clignotement, et surtout **réinitialiser l'état interne du widget de menu** (une saisie en cours, une animation). Un état stable doit rester stable.

### Pourquoi mémoriser l'ancien état

Parce que l'étape 4 en a besoin. Si l'on écrivait `etat = nouvelEtat;` puis `overlays.remove(overlayDeLEtat(etat));`, on retirerait l'overlay qu'on vient tout juste d'ajouter. C'est une faute classique, et silencieuse : l'écran devient noir sans le moindre message d'erreur.

### Pourquoi `viderLeMonde()` sur le retour au menu

Quand on revient au menu, la partie est finie. Le monde contient encore les murs, le décor, et — à partir du chapitre 36 — le joueur, les ennemis et les objets. Les laisser en place aurait deux conséquences : ils continueraient d'exister en mémoire, et la partie suivante se construirait **par-dessus** la précédente.

```dart
  /// Retire tous les composants du monde.
  ///
  /// `.toList()` est indispensable : `monde.children` est la liste vivante des
  /// enfants. Itérer dessus tout en la modifiant est le bug de modification
  /// concurrente rencontré au chapitre 26 (section 26.7).
  void viderLeMonde() {
    monde.removeAll(monde.children.toList());
  }
```

### Ce que `changerEtat()` ne fait PAS

Il ne remet **pas** le score à zéro, ni les vies. Cela appartient à `demarrerPartie()` (section 35.24). La raison est simple : on passe par `menu` dans deux situations très différentes — au démarrage de l'application, et après une partie. Dans le second cas, le chapitre 40 aura besoin de lire le score final **après** le retour au menu pour l'afficher. Une méthode qui fait une seule chose est une méthode qu'on peut appeler sans peur.

> **À retenir.** Un seul chemin pour changer d'écran. Mémoriser l'ancien état avant d'affecter le nouveau. Vider le monde en revenant au menu.

---

## 35.14 — `pauseEngine()` et `resumeEngine()`

Ces deux méthodes viennent de `FlameGame`. Elles arrêtent et relancent la **boucle de jeu**, c'est-à-dire les appels à `update(dt)` et `render(canvas)` sur l'arbre de composants.

### Ce qui s'arrête, ce qui continue

| Élément | `pauseEngine()` l'arrête ? |
| --- | --- |
| `update(dt)` de tous les composants | oui |
| `render(canvas)` de tous les composants | oui (l'image se fige) |
| Effets (`MoveEffect`, `ScaleEffect`…) | oui, ils sont pilotés par `update` |
| `Timer` et `TimerComponent` de Flame | oui, même raison |
| Détection de collisions | oui |
| Overlays Flutter | **non** : ce sont des widgets |
| `Future.delayed` / `Timer` de `dart:async` | **non** : ils vivent hors du moteur |
| Musique lancée par `flame_audio` | **non** (chapitre 40 : il faut l'arrêter à la main) |

Les trois dernières lignes expliquent la moitié des bugs de pause dans les jeux Flame. Un compteur écrit avec `Timer` de `dart:async` continue de tourner pendant la pause ; le même compteur écrit avec le `Timer` de Flame s'arrête. Depuis le chapitre 33, nous n'utilisons que le second, et c'est précisément pour cette raison.

### La règle du projet

```dart
    if (nouvelEtat == GameState.enJeu) {
      resumeEngine();
    } else if (nouvelEtat == GameState.pause ||
        nouvelEtat == GameState.gameOver ||
        nouvelEtat == GameState.victoire) {
      pauseEngine();
    }
```

Autrement dit : **le moteur tourne dans `enJeu`, il est figé dans `pause`, `gameOver` et `victoire`, et on ne s'en occupe pas dans `chargement` et `menu`**.

Pourquoi ne pas mettre en pause dans `menu` ? Parce que le monde y est vide : le moteur ne fait rien de coûteux, et le laisser tourner évite un cas particulier au tout premier appel, quand le jeu n'est pas encore attaché à son widget. Si vous tenez à figer le moteur au menu, ajoutez `GameState.menu` à la seconde condition ; le jeu fonctionnera aussi bien.

### Les états gelés doivent avoir un écran Flutter

C'est la conséquence directe de la première ligne « non » du tableau. Puisque `pauseEngine()` fige tout ce que Flame dessine, **l'écran de pause ne peut pas être un composant Flame** : il serait figé lui aussi, ses boutons ne réagiraient pas.

```text
   État : pause
   ┌─────────────────────────────────────────┐
   │  widgets Flutter (overlay)              │  ← vivant, cliquable
   │      ┌───────────────────────────┐      │
   │      │  PAUSE                    │      │
   │      │  [ Reprendre ]  [ Menu ]  │      │
   │      └───────────────────────────┘      │
   ├─────────────────────────────────────────┤
   │  canvas Flame                           │  ← figé, image dernière frame
   │      héros, ennemis, décor               │
   └─────────────────────────────────────────┘
```

C'est le raisonnement qui justifie tout le mécanisme des overlays, objet de la section suivante.

### Deux erreurs à connaître

**Appeler `pauseEngine()` deux fois** n'est pas un problème : la seconde n'a aucun effet. Idem pour `resumeEngine()`.

**Oublier `resumeEngine()`** est en revanche fatal. Si vous quittez l'état `pause` par un chemin qui n'appelle pas `resumeEngine()`, le jeu revient à l'écran… définitivement figé, sans aucun message d'erreur. C'est exactement pour éviter ce scénario que la remise en marche du moteur est écrite **une seule fois**, dans `changerEtat()`, et nulle part ailleurs.

> **À retenir.** `pauseEngine()` gèle tout ce qui est Flame et rien de ce qui est Flutter. Tout écran affiché pendant une pause doit donc être un widget.

---

## 35.15 — Les overlays Flutter par-dessus le jeu

Un *overlay* est un **widget Flutter dessiné au-dessus du canvas du jeu**, et piloté depuis le code du jeu.

### La pile d'affichage

```text
   ┌──────────────────────────────────────────────────────┐
   │  Overlays actifs (widgets Flutter)                   │  ← 3
   │   menu_principal / hud / pause / game_over ...       │
   ├──────────────────────────────────────────────────────┤
   │  Canvas du jeu : viewport de la caméra (HUD Flame)   │  ← 2
   ├──────────────────────────────────────────────────────┤
   │  Canvas du jeu : le monde vu par la caméra           │  ← 1
   ├──────────────────────────────────────────────────────┤
   │  backgroundColor() du jeu                            │  ← 0
   └──────────────────────────────────────────────────────┘
```

### Pourquoi des widgets et pas des composants Flame

Quatre raisons, dans l'ordre d'importance.

**Ils survivent à la pause.** C'est le point décisif, démontré en section 35.14.

**Vous savez déjà les écrire.** Un menu, c'est une `Column` de boutons. Vous avez tout appris au chapitre 19. Écrire le même menu en composants Flame demanderait de gérer à la main le survol, le clic, l'ordre de tabulation, la mise en page responsive et l'accessibilité.

**Ils sont accessibles.** Un `ElevatedButton` Flutter est lisible par un lecteur d'écran, réagit au clavier, respecte le thème du système. Un rectangle dessiné sur un canvas n'est rien de tout cela.

**Ils se réagencent tout seuls.** `Center`, `Column`, `Expanded`, `LayoutBuilder` gèrent le redimensionnement de la fenêtre sans une ligne de code de votre part.

### Quand ne PAS utiliser un overlay

Pour tout ce qui doit être **synchronisé au pixel et à l'image près** avec le jeu : une barre de vie au-dessus de la tête d'un ennemi, un texte de dégâts qui s'envole, un compteur qui doit s'arrêter net à la pause. Ces éléments sont des composants Flame, ils vivent dans `hud/` (chapitre 38) ou directement dans le monde.

La frontière est celle-ci :

```text
   L'élément doit-il continuer de fonctionner quand le moteur est en pause ?
        │
        ├── OUI  ──► overlay Flutter   (ecrans/)
        └── NON  ──► composant Flame   (hud/ ou composants/)
```

### L'API, côté jeu

Elle est fournie par `game.overlays`, un `OverlayManager` :

```dart
overlays.add('pause');            // affiche l'overlay ; renvoie false s'il l'était déjà
overlays.remove('pause');         // le retire ; renvoie false s'il n'y était pas
overlays.toggle('pause');         // bascule
overlays.isActive('pause');       // interroge
overlays.clear();                 // retire tout
overlays.activeOverlays;          // la liste, en lecture seule
```

Toutes ces méthodes déclenchent la reconstruction du `GameWidget`. Vous n'avez **jamais** à appeler `setState` pour un overlay.

Dans notre projet, une seule méthode appelle `overlays.add` et `overlays.remove` : `changerEtat()`. Aucun autre fichier ne touche à `overlays`. Cette discipline garantit qu'un état et un écran ne peuvent jamais se contredire.

> **À retenir.** Un overlay est un widget Flutter que le jeu allume et éteint par son nom. C'est le seul moyen d'afficher une interface pendant que le moteur est figé.

---

## 35.16 — `overlayBuilderMap` et `initialActiveOverlays`

Le jeu allume des overlays **par leur nom**. Encore faut-il que quelqu'un sache quel widget construire pour ce nom. Ce quelqu'un est le `GameWidget`, et le dictionnaire est `overlayBuilderMap`.

### La signature réelle

```dart
GameWidget<T extends Game>({
  required T game,
  Map<String, OverlayWidgetBuilder<T>>? overlayBuilderMap,
  List<String>? initialActiveOverlays,
  GameLoadingWidgetBuilder<T>? loadingBuilder,
  GameErrorWidgetBuilder? errorBuilder,
  WidgetBuilder? backgroundBuilder,
  FocusNode? focusNode,
  bool autofocus = true,
  MouseCursor? mouseCursor,
  bool addRepaintBoundary = true,
  HitTestBehavior behavior = HitTestBehavior.opaque,
  TextDirection? textDirection,
  Key? key,
});
```

### Un exemple minimal

```dart
GameWidget<DonjonGame>(
  game: jeu,
  initialActiveOverlays: const [Overlays.chargement],
  overlayBuilderMap: {
    Overlays.chargement: (BuildContext context, DonjonGame game) {
      return const EcranChargement();
    },
    Overlays.menuPrincipal: (BuildContext context, DonjonGame game) {
      return MenuPrincipal(jeu: game);
    },
  },
)
```

Chaque valeur de la map est une **fonction** qui reçoit deux arguments et renvoie un widget :

| Paramètre | Type | À quoi il sert |
| --- | --- | --- |
| `context` | `BuildContext` | thème, taille d'écran, `Navigator`, `ScaffoldMessenger` |
| `game` | `DonjonGame` | **la référence typée vers votre jeu** |

Le second est le mécanisme qui relie les deux mondes : votre menu reçoit le jeu, et peut donc appeler `game.demarrerPartie()`. C'est le seul canal dont vous avez besoin — aucun singleton, aucun `Provider`, aucune variable globale.

Le type générique du widget est important : écrivez `GameWidget<DonjonGame>` et non `GameWidget`, sans quoi le paramètre `game` des constructeurs sera typé `Game` et vous n'aurez accès à aucun de vos champs.

### `initialActiveOverlays` — attention au pluriel

Le nom du paramètre est **`initialActiveOverlays`**, avec un `s` à `Overlays`. C'est une liste, pas une chaîne. On l'écrit sans hésiter :

```dart
initialActiveOverlays: const [Overlays.chargement],
```

Ce paramètre sert **une seule fois**, à la construction du widget. Il désigne les overlays visibles avant que le code du jeu n'ait pu s'exécuter.

Pourquoi `chargement` et pas `menu_principal` ? Parce que l'état initial de `DonjonGame` est `GameState.chargement`, et qu'il faut que l'écran affiché corresponde à l'état. Dès que `onLoad()` se termine, `changerEtat(GameState.menu)` bascule vers le menu. Sur une machine rapide, cet écran de chargement s'affiche quelques images seulement ; sur le Web au premier lancement, il est bien visible, et c'est justement à cela qu'il sert.

### Les trois overlays du chapitre 35

| Nom | Constante | Widget | Statut |
| --- | --- | --- | --- |
| `chargement` | `Overlays.chargement` | `EcranChargement` | définitif |
| `menu_principal` | `Overlays.menuPrincipal` | `MenuPrincipal` | définitif |
| `hud` | `Overlays.hud` | `HudProvisoire` | **provisoire**, remplacé au chapitre 38 |

Les trois autres noms (`pause`, `game_over`, `victoire`) existent dans la classe `Overlays` et sont gérés par `changerEtat()`, mais **ne sont pas encore dans la map**. C'est sans danger tant que les états correspondants ne sont pas atteints, ce qui est le cas dans ce chapitre.

En revanche, si vous forcez `changerEtat(GameState.pause)` pour essayer, le `GameWidget` cherchera une entrée `'pause'` absente de la map et l'application plantera. Retenez la règle : **tout overlay activé doit avoir un constructeur dans `overlayBuilderMap`**. Le chapitre 40 ajoutera les trois manquants.

### Ordre d'affichage

Si deux overlays sont actifs en même temps, ils se superposent dans l'ordre des **clés de la map**, ajusté par le paramètre `priority` de `overlays.add(nom, priority: n)`.

Notre projet n'affiche jamais deux overlays d'état simultanément, puisque `changerEtat()` en retire un avant d'en ajouter un autre. La question ne se posera qu'au chapitre 40, quand le HUD restera visible derrière le menu de pause.

> **À retenir.** `overlayBuilderMap` associe un nom à un constructeur de widget ; `initialActiveOverlays` (au pluriel) dit ce qui est visible au tout premier affichage. Un nom activé sans entrée dans la map fait planter l'application.

---

## 35.17 — La classe `Overlays` et ses constantes

Revenons sur ce petit bloc écrit en section 35.5, car il illustre une habitude qui vaut pour tout le projet.

```dart
class Overlays {
  Overlays._();

  static const String menuPrincipal = 'menu_principal';
  static const String hud = 'hud';
  static const String pause = 'pause';
  static const String gameOver = 'game_over';
  static const String victoire = 'victoire';
  static const String chargement = 'chargement';
}
```

### Le problème que cela résout

Un nom d'overlay est écrit à **deux endroits** minimum : une fois comme clé dans `overlayBuilderMap`, une fois dans `overlays.add(...)`. Ce sont deux fichiers différents, souvent écrits à plusieurs jours d'intervalle.

Avec des chaînes littérales :

```dart
// dans main.dart
'menu_principal': (context, game) => MenuPrincipal(jeu: game),

// dans donjon_game.dart, trois semaines plus tard
overlays.add('menuPrincipal');   // faute de frappe : casse au lieu de tiret bas
```

Le code **compile parfaitement**. Aucun avertissement. À l'exécution, le jeu tente d'afficher un overlay inconnu. Selon la version de Flame, vous obtenez soit un écran noir, soit une exception au milieu de la pile de rendu, très loin de la ligne fautive.

Avec la classe `Overlays` :

```dart
overlays.add(Overlays.menuPrincipl);  // erreur de compilation immédiate
```

L'éditeur souligne le nom en rouge avant même que vous ayez sauvegardé. Une classe de constantes transforme une **erreur d'exécution silencieuse** en **erreur de compilation bruyante**. C'est le meilleur échange qu'un développeur puisse faire.

### Pourquoi les valeurs sont en `snake_case`

`'menu_principal'`, pas `'menuPrincipal'`. Ces chaînes n'apparaissent jamais dans le code lu par un humain — on écrit toujours `Overlays.menuPrincipal`. Elles peuvent en revanche apparaître dans des messages de debug et dans `overlays.activeOverlays`. Le `snake_case` les rend lisibles dans les logs, et les distingue au premier coup d'œil des identifiants Dart.

### Le même motif ailleurs

Vous retrouverez cette technique trois fois dans le projet :

| Classe | Fichier | Chapitre | Ce qu'elle protège |
| --- | --- | --- | --- |
| `Constantes` | `config/constantes.dart` | 35 | les nombres de réglage |
| `Overlays` | `config/constantes.dart` | 35 | les noms d'overlays |
| `Palette` | `config/palette.dart` | 35 | les couleurs |
| `SonsJeu` | `services/audio_service.dart` | 40 | les noms de fichiers audio |

Le dernier est le plus critique : un nom de fichier audio erroné ne se détecte qu'au moment où le son doit être joué, c'est-à-dire pendant un test, au pire moment.

> **À retenir.** Toute chaîne de caractères qui sert de **clé technique** doit être une constante nommée. Jamais un littéral recopié.

---

## 35.18 — `lib/ecrans/menu_principal.dart` : un widget Flutter classique

Nous quittons Flame pour une section entière. Ce fichier ne contient **aucun composant, aucun `Vector2`, aucun `Canvas`** : c'est du Flutter pur, exactement celui du chapitre 19.

### Le squelette

```dart
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/palette.dart';
import '../donjon_game.dart';

/// Le menu principal du jeu.
///
/// C'est un overlay : il est affiché PAR-DESSUS le canvas de Flame quand l'état
/// du jeu vaut `GameState.menu`.
class MenuPrincipal extends StatelessWidget {
  const MenuPrincipal({super.key, required this.jeu});

  /// La référence vers le jeu, transmise par `overlayBuilderMap`.
  /// C'est le seul lien entre l'interface et le moteur.
  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

### Pourquoi `StatelessWidget`

Le menu n'a **rien à mémoriser**. Il affiche un titre et quatre boutons ; chaque bouton délègue immédiatement au jeu ou ouvre une boîte de dialogue. Aucun champ ne change au cours de sa vie.

Le réflexe du débutant est de prendre un `StatefulWidget` « au cas où ». C'est un mauvais réflexe : un `StatefulWidget` coûte un objet `State` supplémentaire, un cycle de vie à comprendre, et surtout il masque la vraie question, qui est « où vit l'information ? ». Ici, toute l'information vit dans `DonjonGame`.

La boîte de dialogue des options, elle, a bien un état (les positions des curseurs). Elle sera donc, et elle seule, un `StatefulWidget` (section 35.21).

### Le champ `jeu`

```dart
final DonjonGame jeu;
```

C'est le point de contact entre les deux mondes. Il arrive par le constructeur, rempli dans `overlayBuilderMap` :

```dart
Overlays.menuPrincipal: (context, game) => MenuPrincipal(jeu: game),
```

Notez ce que nous **n'avons pas** fait : pas de singleton `DonjonGame.instance`, pas de variable globale, pas de `Provider`, pas d'`InheritedWidget`. Flame transmet déjà la référence ; s'en servir suffit. Le chapitre 26 (section 26.28) expliquait pourquoi un singleton rend les tests impossibles ; la règle vaut ici.

### Le `Material` en racine

```dart
  @override
  Widget build(BuildContext context) {
    return Material(
      // Transparent : le décor du menu est dessiné juste en dessous.
      color: Colors.transparent,
      child: /* ... */,
    );
  }
```

Un overlay est greffé dans l'arbre de widgets à un endroit qui n'est pas forcément sous un `Scaffold`. Or beaucoup de widgets Material (`ElevatedButton`, `TextField`, `ListTile`, `InkWell`) exigent un ancêtre `Material` pour dessiner leurs effets d'encre. Sans lui, vous obtenez l'erreur classique :

```text
No Material widget found.
ElevatedButton widgets require a Material widget ancestor.
```

Poser un `Material` transparent en racine de chaque overlay est une assurance à zéro coût. Prenez-en l'habitude.

### La structure générale du menu

```text
   Material (transparent)
   └── DecoratedBox      ← dégradé de fond (35.23)
       └── Center
           └── SingleChildScrollView   ← survie aux petits écrans
               └── Column (mainAxisSize: min)
                   ├── titre « DONJON DE DART »
                   ├── sous-titre
                   ├── bouton JOUER          (35.19)
                   ├── bouton CONTINUER      (35.20)
                   ├── bouton OPTIONS        (35.21)
                   ├── bouton QUITTER        (35.22)
                   └── mention de version
```

Le `SingleChildScrollView` mérite un mot. En mode paysage sur un téléphone, la hauteur disponible peut descendre sous 320 pixels. Sans lui, la `Column` déborde et Flutter affiche la fameuse bande rayée jaune et noire avec « BOTTOM OVERFLOWED BY 47 PIXELS ». Avec lui, le menu défile. Deux lignes de code contre un bug garanti sur la moitié des téléphones.

> **À retenir.** Un overlay est un widget ordinaire. Il reçoit le jeu par son constructeur, et pose un `Material` transparent en racine.

---

## 35.19 — Le bouton Jouer

C'est le bouton principal : celui que 95 % des joueurs presseront, et le seul qui soit mis en valeur.

### Le widget de bouton commun

Les quatre boutons du menu partagent leur apparence. On écrit donc **un** widget réutilisable, dans le même fichier.

```dart
/// Un bouton du menu principal.
///
/// `principal` met le bouton en valeur (fond plein) ; les autres sont sobres.
/// `onPressed` à `null` désactive le bouton : Flutter le grise tout seul.
class BoutonMenu extends StatelessWidget {
  const BoutonMenu({
    super.key,
    required this.libelle,
    required this.icone,
    required this.onPressed,
    this.principal = false,
    this.infoBulle,
  });

  final String libelle;
  final IconData icone;
  final VoidCallback? onPressed;
  final bool principal;
  final String? infoBulle;

  @override
  Widget build(BuildContext context) {
    final Widget bouton = SizedBox(
      width: 260,
      height: 52,
      child: principal
          ? FilledButton.icon(/* fond orange plein — voir 35.32 */)
          : OutlinedButton.icon(/* bordure sobre — voir 35.32 */),
    );

    if (infoBulle == null) {
      return bouton;
    }
    return Tooltip(message: infoBulle!, child: bouton);
  }
}
```

### Le bouton lui-même

```dart
BoutonMenu(
  libelle: 'JOUER',
  icone: Icons.play_arrow,
  principal: true,
  onPressed: () async {
    await jeu.demarrerPartie();
  },
),
```

### Trois points à comprendre

**Le rappel est asynchrone.** `demarrerPartie()` renvoie un `Future<void>` : elle construit un niveau, ce qui pourra demander des chargements au chapitre 39. Le rappel `onPressed` attend une fonction sans valeur de retour (`VoidCallback`), mais Dart accepte qu'une fonction `async` y soit passée, car `Future<void>` est compatible avec `void`. Écrire `await` à l'intérieur reste utile : sans lui, une exception levée pendant le chargement disparaîtrait silencieusement.

**Le menu ne se ferme pas lui-même.** Regardez ce qui n'est pas écrit : nulle part on ne trouve `jeu.overlays.remove(Overlays.menuPrincipal)`. C'est `demarrerPartie()` qui appelle `changerEtat(GameState.chargement)`, et c'est `changerEtat` qui retire l'overlay. Le menu **demande**, il ne décide pas. Cette discipline est ce qui rend la navigation prévisible.

**Le double clic est neutralisé.** Que se passe-t-il si le joueur presse deux fois « JOUER » très vite ? Le premier appel bascule l'état sur `chargement`, ce qui retire l'overlay du menu de l'arbre. Le second clic n'a plus de bouton sous le doigt. Et même s'il passait, le garde-fou `if (nouvelEtat == etat) return;` de `changerEtat()` absorberait la transition.

### Résultat attendu

Au clic sur « JOUER » :

```text
[DonjonGame] Menu principal -> Chargement
[DonjonGame] Chargement -> En jeu
```

L'écran passe du menu à une salle de démonstration entourée de murs, avec le HUD provisoire en haut.

> **À retenir.** Un bouton d'interface appelle **une** méthode du jeu et ne touche jamais aux overlays lui-même.

---

## 35.20 — Le bouton Continuer

Ce bouton reprendra la partie sauvegardée. Le `SauvegardeService` qui le rendra fonctionnel appartient au chapitre 40. La question de cette section est donc : **que fait-on d'un bouton dont la fonctionnalité n'existe pas encore ?**

### Les trois mauvaises réponses

**Ne pas l'afficher.** Le menu changerait de forme entre le chapitre 35 et le chapitre 40, et vous ne verriez pas venir les problèmes de mise en page.

**L'afficher actif et ne rien faire au clic.** C'est le pire choix : un bouton qui ne réagit pas donne au joueur l'impression que le jeu est cassé.

**L'afficher actif et lancer une nouvelle partie.** Un mensonge d'interface. Le joueur croira avoir repris sa partie.

### La bonne réponse : un bouton désactivé

En Flutter, un bouton dont `onPressed` vaut `null` est **automatiquement grisé et non cliquable**. Aucune logique à écrire.

```dart
  /// Une partie sauvegardée est-elle disponible ?
  ///
  /// PROVISOIRE (chapitre 35) : renvoie toujours `false`.
  /// Au chapitre 40, cette propriété interrogera `SauvegardeService`.
  bool get _peutContinuer => false;
```

```dart
BoutonMenu(
  libelle: 'CONTINUER',
  icone: Icons.history,
  onPressed: _peutContinuer
      ? () async {
          // Chapitre 40 : reprendre au niveau sauvegardé.
          await jeu.demarrerPartie();
        }
      : null,
  infoBulle: _peutContinuer
      ? 'Reprendre la partie sauvegardée'
      : 'Aucune partie sauvegardée (disponible au chapitre 40)',
),
```

### Ce que ce motif vous apporte

**La mise en page est définitive.** Le menu du chapitre 40 aura exactement la même hauteur que celui du chapitre 35.

**Le point de branchement est écrit.** Au chapitre 40, une seule ligne changera :

```dart
  bool get _peutContinuer => SauvegardeService.instance.aUneSauvegarde;
```

**Le joueur est informé.** L'info-bulle explique pourquoi le bouton est gris. Un bouton désactivé sans explication est presque aussi frustrant qu'un bouton mort.

### Un mot sur `Tooltip`

Sur ordinateur, l'info-bulle apparaît au survol de la souris. Sur mobile, elle apparaît sur un appui long. Dans les deux cas, elle ne coûte rien quand elle n'est pas sollicitée. C'est le moyen le plus économique de documenter une interface.

> **À retenir.** `onPressed: null` désactive un bouton. C'est la manière propre de montrer une fonctionnalité à venir, sans mentir ni casser la mise en page.

---

## 35.21 — Le bouton Options

Les options ouvrent une boîte de dialogue. Deux des trois réglages ne seront branchés qu'au chapitre 40 (le son) ; le troisième fonctionne dès aujourd'hui (le mode debug).

### Le stockage provisoire des réglages

```dart
/// Réglages du joueur, en mémoire uniquement.
///
/// PROVISOIRE (chapitre 35) : ces valeurs sont perdues à la fermeture de
/// l'application, et le volume n'est branché sur rien.
/// Au chapitre 40, `SauvegardeService` les rendra persistantes et
/// `AudioService` les appliquera réellement.
class ReglagesProvisoires {
  ReglagesProvisoires._();

  static double volumeMusique = 0.6;
  static double volumeEffets = 0.8;
}
```

### La boîte de dialogue

Elle a un état — la position des curseurs pendant qu'on les fait glisser — donc c'est un `StatefulWidget`.

```dart
/// Boîte de dialogue des options.
class DialogueOptions extends StatefulWidget {
  const DialogueOptions({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  State<DialogueOptions> createState() => _DialogueOptionsState();
}

class _DialogueOptionsState extends State<DialogueOptions> {
  late double _musique = ReglagesProvisoires.volumeMusique;
  late double _effets = ReglagesProvisoires.volumeEffets;
  late bool _debug = widget.jeu.debugMode;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      backgroundColor: Palette.panneau,
      title: const Text('OPTIONS', style: TextStyle(letterSpacing: 3)),
      content: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          const Text('Volume de la musique', style: Palette.sousTitre),
          Slider(
            value: _musique,
            activeColor: Palette.accent,
            onChanged: (double valeur) {
              setState(() => _musique = valeur);
              ReglagesProvisoires.volumeMusique = valeur;
              // Chapitre 40 : AudioService.instance.volumeMusique = valeur;
            },
          ),
          // Le curseur des effets est bâti sur le même modèle (voir 35.32).
          SwitchListTile(
            contentPadding: EdgeInsets.zero,
            title: const Text('Mode debug', style: Palette.sousTitre),
            value: _debug,
            onChanged: (bool valeur) {
              setState(() => _debug = valeur);
              // Celui-ci fonctionne dès maintenant.
              widget.jeu.debugMode = valeur;
            },
          ),
        ],
      ),
      actions: <Widget>[
        TextButton(
          onPressed: () => Navigator.of(context).pop(),
          child: const Text('FERMER'),
        ),
      ],
    );
  }
}
```

### L'ouverture depuis le menu

```dart
BoutonMenu(
  libelle: 'OPTIONS',
  icone: Icons.settings,
  onPressed: () {
    showDialog<void>(
      context: context,
      builder: (BuildContext context) => DialogueOptions(jeu: jeu),
    );
  },
),
```

### Quatre points de vigilance

**`showDialog` a besoin d'un `Navigator`.** Il en trouve un parce que `main.dart` place tout dans un `MaterialApp`. Si vous aviez posé le `GameWidget` seul dans `runApp`, `showDialog` échouerait. C'est l'une des raisons pour lesquelles notre `main.dart` construit une vraie application Flutter.

**Le `context` du `builder` masque celui de l'extérieur.** Le `Navigator.of(context).pop()` de la boîte utilise le contexte **du builder**, pas celui du menu. Les nommer pareil est la convention Flutter, mais sachez lequel vous utilisez.

**`late` avec initialiseur pour lire `widget`.** `late bool _debug = widget.jeu.debugMode;` fonctionne parce que `late` diffère l'évaluation au premier accès, moment où `widget` est disponible. Sans `late`, le compilateur refuse : on ne peut pas lire `widget` dans un initialiseur de champ ordinaire.

**Le dialogue n'est pas un overlay.** C'est une route Flutter ordinaire, empilée par le `Navigator` par-dessus tout, y compris par-dessus les overlays. La machine à états du jeu ne change pas : on reste dans `GameState.menu` pendant que les options sont ouvertes. C'est voulu — une boîte de dialogue n'est pas un état de jeu.

> **À retenir.** Un réglage qui n'est pas encore branché s'affiche quand même, avec le commentaire qui indique le chapitre et la ligne de branchement.

---

## 35.22 — Le bouton Quitter

« Quitter » n'a pas le même sens sur toutes les plateformes. C'est l'occasion d'écrire du code conscient de sa cible.

### Ce que dit chaque plateforme

| Plateforme | Comportement attendu | Moyen |
| --- | --- | --- |
| Android | fermer l'application | `SystemNavigator.pop()` |
| Windows, Linux, macOS | fermer la fenêtre | `SystemNavigator.pop()` |
| Web | **impossible** : un onglet ne se ferme pas lui-même | expliquer au joueur |
| iOS | possible techniquement, mais Apple le déconseille formellement dans ses recommandations d'interface | ne pas proposer le bouton |

### Le code

```dart
  /// Quitte l'application, si la plateforme le permet.
  void _quitter(BuildContext context) {
    if (kIsWeb) {
      // Un onglet de navigateur ne peut pas se fermer lui-même : on informe.
      showDialog<void>(
        context: context,
        builder: (BuildContext context) => AlertDialog(
          backgroundColor: Palette.panneau,
          title: const Text('Quitter'),
          content: const Text(
            'Dans un navigateur, le jeu ne peut pas se fermer lui-même.\n'
            'Fermez simplement cet onglet.',
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.of(context).pop(),
              child: const Text('COMPRIS'),
            ),
          ],
        ),
      );
      return;
    }

    // Android, Windows, Linux, macOS : on demande au système de fermer.
    SystemNavigator.pop();
  }
```

```dart
BoutonMenu(
  libelle: 'QUITTER',
  icone: Icons.exit_to_app,
  onPressed: () => _quitter(context),
),
```

### `kIsWeb`

Cette constante vient de `package:flutter/foundation.dart`. Elle vaut `true` uniquement quand le code est compilé pour le Web.

Sa particularité est d'être une **constante de compilation**. Le compilateur Dart sait, au moment de la compilation, quelle branche est morte, et il la supprime purement et simplement du binaire. Vous n'embarquez pas le code Web dans votre APK Android.

Attention à ne pas la confondre avec `Platform.isAndroid` de `dart:io` : ce dernier **plante à l'exécution sur le Web**, parce que `dart:io` n'y existe pas. La règle est : `kIsWeb` d'abord, `Platform.*` seulement à l'intérieur de la branche non-Web.

### Faut-il demander confirmation ?

Pour un jeu de cinq minutes sans sauvegarde automatique, non : cela ajoute un clic à chaque fermeture. Pour un jeu à progression longue, oui. Si vous voulez l'ajouter, c'est un `showDialog` de quatre lignes, sur le modèle exact de la branche Web ci-dessus.

> **À retenir.** `kIsWeb` est une constante de compilation : elle sert à écrire du code qui n'existe pas sur les autres plateformes. `SystemNavigator.pop()` ferme l'application partout ailleurs qu'en navigateur.

---

## 35.23 — Styliser le menu sans image

Rappel de la contrainte : **aucun fichier image**. Un menu attirant se construit pourtant très bien avec quatre outils Flutter.

### 1. Le dégradé de fond

```dart
      child: DecoratedBox(
        decoration: const BoxDecoration(
          gradient: RadialGradient(
            center: Alignment(0, -0.6),
            radius: 1.1,
            colors: [Palette.panneau, Palette.fondMenu],
          ),
        ),
        child: /* ... */,
      ),
```

Un `RadialGradient` centré en haut simule une torche qui éclaire le titre. Le coût est nul et l'effet est immédiat : un fond uni fait « prototype », un dégradé fait « jeu ».

Pour un dégradé du haut vers le bas, `LinearGradient` avec `begin: Alignment.topCenter` et `end: Alignment.bottomCenter`.

### 2. Le titre travaillé

```dart
Text(
  'DONJON',
  style: Palette.titre.copyWith(
    shadows: const <Shadow>[
      Shadow(color: Palette.accent, blurRadius: 24),   // lueur
      Shadow(color: Palette.fondMenu, offset: Offset(3, 3)), // relief
    ],
  ),
),
Text(
  'DE DART',
  style: Palette.titre.copyWith(
    fontSize: 34,
    color: Palette.accent,
    letterSpacing: 12,
  ),
),
const Text('UN JEU DE PLATEFORMES EN FLAME', style: Palette.sousTitre),
```

Trois techniques y sont à l'œuvre :

- **`letterSpacing`** : écarter les lettres donne instantanément un aspect « logo ». C'est l'effet le plus rentable de tout le menu.
- **`shadows`** : une ombre floue et colorée (`blurRadius: 24`) imite une lueur ; une ombre nette et sombre décalée de trois pixels détache le texte du fond.
- **deux tailles, deux couleurs** : hiérarchiser le titre le rend lisible en un coup d'œil.

### 3. Le cadre

```dart
Container(
  padding: const EdgeInsets.symmetric(horizontal: 48, vertical: 36),
  decoration: BoxDecoration(
    color: Palette.fondMenu.withValues(alpha: 0.72),
    border: Border.all(color: Palette.mur, width: 3),
    borderRadius: BorderRadius.circular(10),
  ),
  child: /* la colonne de boutons */,
)
```

Un cadre semi-transparent avec une bordure épaisse suffit à donner l'impression d'un panneau de pierre.

`withValues(alpha: 0.72)` est la méthode moderne de `Color` ; l'ancienne, `withOpacity(0.72)`, est dépréciée dans les versions récentes de Flutter. Si votre éditeur signale `withValues` comme inconnue, votre Flutter est antérieur à 3.27 : utilisez `withOpacity`.

### 4. Les icônes Material

`Icons.play_arrow`, `Icons.settings`, `Icons.exit_to_app`, `Icons.history` sont des **polices vectorielles**, pas des images. Elles sont embarquées par la ligne `uses-material-design: true` du `pubspec.yaml`, se colorent, se redimensionnent sans perte, et ne coûtent rien à charger.

### Ce qu'on ne fera pas

Pas d'animation d'entrée, pas de particules derrière le titre, pas de musique de menu. Ces éléments sont plaisants et faciles à ajouter — l'exercice 8 vous y invite — mais ils n'apprennent rien de neuf et alourdiraient le chapitre.

### Résultat attendu

```text
   ┌──────────────────────────────────────────────────────┐
   │                                                      │
   │                    D O N J O N                       │
   │                  D E   D A R T                       │
   │        UN JEU DE PLATEFORMES EN FLAME                │
   │                                                      │
   │      ┌──────────────────────────────────────┐        │
   │      │        >  JOUER                      │        │
   │      │        >  CONTINUER      (grisé)     │        │
   │      │        >  OPTIONS                    │        │
   │      │        >  QUITTER                    │        │
   │      └──────────────────────────────────────┘        │
   │                                                      │
   │                 Chapitre 35 — v1.0.0                 │
   └──────────────────────────────────────────────────────┘
```

> **À retenir.** `letterSpacing`, `shadows`, un dégradé et une bordure : quatre propriétés suffisent à faire un menu présentable sans le moindre fichier.

---

## 35.24 — `demarrerPartie()`

Retour dans `donjon_game.dart`. Cette méthode est le pont entre le menu et le jeu.

```dart
  /// Démarre une nouvelle partie depuis le début.
  ///
  /// Remet à zéro le score, les vies et le niveau, reconstruit le monde,
  /// puis entre dans l'état `enJeu`.
  Future<void> demarrerPartie() async {
    // 1. On affiche l'écran de chargement AVANT tout travail.
    changerEtat(GameState.chargement);

    // 2. Remise à zéro de la partie. `meilleurScore` n'est PAS touché.
    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = 0;

    // 3. On repart d'un monde vide, quoi qu'il s'y trouve.
    viderLeMonde();

    // 4. Construction du niveau.
    //    PROVISOIRE (chapitre 35) : une salle de démonstration.
    //    Au chapitre 39 : await chargerNiveau(niveauCourant);
    await monde.addAll(_salleDeDemonstration());

    // 5. La caméra regarde le centre du monde.
    //    Au chapitre 36 : camera.follow(joueur);
    camera.viewfinder.position = Vector2.zero();

    // 6. Petite pause artificielle pour que l'écran de chargement soit vu.
    //    À RETIRER au chapitre 39, quand le chargement prendra du temps.
    await Future<void>.delayed(const Duration(milliseconds: 350));

    // 7. On entre en jeu.
    changerEtat(GameState.enJeu);
  }
```

### L'ordre des étapes n'est pas négociable

**Le chargement d'abord.** Si l'on construisait le niveau avant de changer d'état, le joueur verrait le menu se figer pendant la construction. Passer d'abord en `chargement` donne un retour visuel immédiat.

**La remise à zéro avant la construction.** Au chapitre 39, la construction du niveau lira `niveauCourant`. S'il n'était pas encore remis à zéro, on reconstruirait le dernier niveau joué.

**Le vidage avant l'ajout.** Sans lui, une seconde partie superposerait ses murs à ceux de la première. Le bug est spectaculaire et facile à provoquer : jouez, revenez au menu, rejouez.

**L'entrée en jeu en dernier.** Tant que `changerEtat(GameState.enJeu)` n'est pas appelé, le moteur est libre et rien ne peut bouger dans un monde à moitié construit.

### La salle de démonstration

C'est le bouchon du chapitre 35. Il disparaîtra au chapitre 39, remplacé par `chargerNiveau()`.

```dart
  /// Décor provisoire du chapitre 35.
  ///
  /// Quatre murs, deux plateformes, un rectangle à l'emplacement du futur
  /// joueur. Remplacé par `chargerNiveau(index)` au chapitre 39.
  List<Component> _salleDeDemonstration() {
    const double t = Constantes.tailleTuile;
    const double largeur = 20 * t; // 640 unités de monde
    const double hauteur = 12 * t; // 384 unités de monde
    const double gauche = -largeur / 2;
    const double haut = -hauteur / 2;

    RectangleComponent bloc(double x, double y, double l, double h, Color c) {
      return RectangleComponent(
        position: Vector2(x, y),
        size: Vector2(l, h),
        paint: Palette.peinture(c),
      );
    }

    return <Component>[
      bloc(gauche, haut, largeur, t, Palette.mur), // plafond
      bloc(gauche, haut + hauteur - t, largeur, t, Palette.mur), // sol
      bloc(gauche, haut, t, hauteur, Palette.mur), // mur gauche
      bloc(gauche + largeur - t, haut, t, hauteur, Palette.mur), // mur droit
      bloc(gauche + 4 * t, haut + 8 * t, 4 * t, t / 2, Palette.plateforme),
      bloc(gauche + 12 * t, haut + 6 * t, 4 * t, t / 2, Palette.plateforme),
      // Emplacement du héros : un simple rectangle bleu (chapitre 36).
      bloc(gauche + 2 * t, haut + 9 * t, t * 0.75, t * 1.5, Palette.joueur),
      // Un TextComponent complète le décor : voir la section 35.32.
    ];
  }
```

Deux remarques de lecture.

**La fonction `bloc` est locale.** Dart autorise les fonctions imbriquées (chapitre 07). Elle n'a de sens que dans cette méthode, elle n'a donc rien à faire ailleurs, et elle évite sept fois la répétition de `RectangleComponent(...)`.

**Le monde est centré sur l'origine.** Les coordonnées vont de −320 à +320 en X, et de −192 à +192 en Y. Combiné à `camera.viewfinder.anchor = Anchor.center` et `position = Vector2.zero()`, la salle apparaît exactement au centre de l'écran. Les niveaux du chapitre 39 partiront au contraire de (0, 0) en haut à gauche, comme toute grille de tuiles ; ce sera alors la caméra qui suivra le joueur.

### `await monde.addAll(...)`

`addAll` renvoie un `Future<void>` (contrairement à `add`, qui renvoie `FutureOr<void>`). L'`await` garantit que **tous les composants sont montés** avant la suite. Ici, la suite n'en dépend pas ; au chapitre 39, elle en dépendra (positionner le joueur exige que le joueur existe). Prendre l'habitude dès maintenant coûte quatre lettres.

### La pause artificielle

`await Future<void>.delayed(const Duration(milliseconds: 350));` n'a **aucune utilité fonctionnelle**. Sans elle, l'écran de chargement apparaîtrait et disparaîtrait en une image, produisant un clignotement désagréable.

C'est une décision d'ergonomie assumée, et elle est marquée comme telle dans le commentaire. Au chapitre 39, la construction d'un niveau prendra un temps réel et la ligne sera supprimée.

> **À retenir.** `demarrerPartie()` fait sept choses dans un ordre précis : afficher le chargement, remettre à zéro, vider, construire, cadrer, patienter, entrer en jeu.

---

## 35.25 — L'écran de chargement

Il existe **deux** écrans de chargement dans un jeu Flame, et ils ne servent pas à la même chose. Les confondre est fréquent.

### 1. `loadingBuilder` : pendant `onLoad()` du jeu

C'est un paramètre du `GameWidget`. Flame l'affiche **entre le premier affichage du widget et la fin de `onLoad()`** de la classe de jeu.

```dart
GameWidget<DonjonGame>(
  game: jeu,
  loadingBuilder: (BuildContext context) => const EcranChargement(
    message: 'Préparation du donjon…',
  ),
  // ...
)
```

Vous ne le contrôlez pas : Flame le montre et le retire tout seul. Il ne dure que le temps de `onLoad()`, soit quelques millisecondes dans notre projet — mais plusieurs secondes au chapitre 40, quand les sons seront préchargés, et surtout au premier lancement d'une version Web.

### 2. L'overlay `chargement` : pendant vos propres chargements

C'est le nôtre, celui que `changerEtat(GameState.chargement)` allume. Il couvre la construction d'un niveau, c'est-à-dire un moment où le jeu **est déjà chargé** mais où le monde est en travaux.

```text
   Lancement de l'application
        │
        ▼
   [ loadingBuilder ]  ← Flame, pendant DonjonGame.onLoad()
        │
        ▼
   [ overlay menu_principal ]
        │  clic sur JOUER
        ▼
   [ overlay chargement ]   ← à nous, pendant demarrerPartie()
        │
        ▼
   [ overlay hud ] + le monde
```

### Le widget, commun aux deux

Nous écrivons un seul widget, réutilisé par les deux mécanismes. Il vit dans `ecrans/menu_principal.dart`, aux côtés du menu.

```dart
/// Écran de chargement, utilisé à deux endroits :
/// - par `loadingBuilder` du GameWidget, pendant `DonjonGame.onLoad()` ;
/// - par l'overlay `Overlays.chargement`, pendant `demarrerPartie()`.
class EcranChargement extends StatelessWidget {
  const EcranChargement({super.key, this.message = 'Chargement…'});

  final String message;

  @override
  Widget build(BuildContext context) {
    return Material(
      color: Palette.fondMenu, // opaque : on masque le monde en travaux
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const SizedBox(
              width: 44,
              height: 44,
              child: CircularProgressIndicator(
                color: Palette.accent,
                strokeWidth: 3,
              ),
            ),
            const SizedBox(height: 24),
            Text(message, style: Palette.sousTitre),
          ],
        ),
      ),
    );
  }
}
```

Notez que le `Material` est ici **opaque** (`color: Palette.fondMenu`), contrairement à celui du menu. Un écran de chargement doit masquer ce qu'il y a derrière : pendant `demarrerPartie()`, le monde est à moitié détruit, et il n'y a aucune raison de le montrer.

`CircularProgressIndicator` sans paramètre `value` tourne indéfiniment : c'est un indicateur **indéterminé**, adapté quand on ne sait pas combien de temps l'opération prendra. Si un jour vous chargez cent ressources et que vous voulez une vraie barre de progression, donnez-lui `value: chargees / total`.

### Pourquoi ne pas simplement afficher un écran noir

Parce qu'un écran noir immobile est indiscernable d'un plantage. Un indicateur qui tourne dit au joueur « le programme travaille ». C'est la différence entre attendre et se demander s'il faut redémarrer.

> **À retenir.** `loadingBuilder` couvre le chargement **du jeu** ; l'overlay `chargement` couvre le chargement **d'un niveau**. Un seul widget, deux usages.

---

## 35.26 — `lib/main.dart` complet

Le point d'entrée. Il est court, et chacune de ses lignes compte. Voici ses deux
morceaux décisifs ; le fichier entier, avec `DonjonApp`, le thème et
`HudProvisoire`, figure en section 35.32.

```dart
Future<void> main() async {
  // Obligatoire dès qu'on appelle une API de plateforme avant runApp().
  WidgetsFlutterBinding.ensureInitialized();

  // Le jeu est conçu pour le paysage. Sans effet sur le Web et le bureau.
  await SystemChrome.setPreferredOrientations(<DeviceOrientation>[
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight,
  ]);

  runApp(const DonjonApp());
}
```

```dart
class _PageDeJeuState extends State<PageDeJeu> {
  late final DonjonGame jeu;

  @override
  void initState() {
    super.initState();
    jeu = DonjonGame(); // une seule fois dans la vie du widget
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Palette.fondJeu,
      body: GameWidget<DonjonGame>(
        game: jeu,
        // Affiché par Flame pendant DonjonGame.onLoad().
        loadingBuilder: (BuildContext context) =>
            const EcranChargement(message: 'Préparation du donjon…'),
        // Affiché par Flame si onLoad() lève une exception.
        errorBuilder: (BuildContext context, Object erreur) => Material(
          color: Palette.fondMenu,
          child: Center(child: Text('$erreur')),
        ),
        // L'état initial du jeu est GameState.chargement : l'overlay affiché
        // au premier rendu doit correspondre.
        initialActiveOverlays: const <String>[Overlays.chargement],
        overlayBuilderMap: <String, Widget Function(BuildContext, DonjonGame)>{
          Overlays.chargement: (BuildContext context, DonjonGame game) =>
              const EcranChargement(message: 'Construction du niveau…'),
          Overlays.menuPrincipal: (BuildContext context, DonjonGame game) =>
              MenuPrincipal(jeu: game),
          // PROVISOIRE : remplacé par le vrai HUD au chapitre 38.
          Overlays.hud: (BuildContext context, DonjonGame game) =>
              HudProvisoire(jeu: game),
        },
      ),
    );
  }
}
```

### `WidgetsFlutterBinding.ensureInitialized()`

Cette ligne initialise le pont entre Dart et la plateforme. Elle est **obligatoire** dès qu'on appelle une API de plateforme (ici `SystemChrome`) avant `runApp()`. Sans elle, vous obtenez à l'exécution :

```text
Unhandled Exception: ServicesBinding.defaultBinaryMessenger was accessed
before the binding was initialized.
```

Au chapitre 40, elle sera également indispensable pour lire le stockage local avant l'affichage.

### Pourquoi `MaterialApp` et pas seulement `GameWidget`

Au chapitre 27, nous écrivions `runApp(GameWidget(game: MonJeu()))`. C'était suffisant pour une démonstration. Ici, `MaterialApp` apporte quatre choses dont nous avons besoin :

| Apport | Utilisé par |
| --- | --- |
| Un `Navigator` | `showDialog` des options (35.21) et de la sortie Web (35.22) |
| Un `Theme` | la couleur des boutons, des curseurs, du texte par défaut |
| Un `Directionality` | tout widget qui affiche du texte |
| Un `ScaffoldMessenger` | les `SnackBar`, si vous en ajoutez |

Sans `MaterialApp`, le premier `showDialog` déclenche `Navigator operation requested with a context that does not include a Navigator`.

### Pourquoi `StatefulWidget` pour la page

C'est le point le plus important du fichier, et une erreur classique.

```dart
// FAUX
class PageDeJeu extends StatelessWidget {
  @override
  Widget build(BuildContext context) =>
      GameWidget<DonjonGame>(game: DonjonGame());  // nouveau jeu à chaque build
}
```

`build()` est rappelé à chaque changement de thème, à chaque rotation, à chaque redimensionnement de fenêtre, à chaque `setState` d'un ancêtre. Chaque appel construirait un `DonjonGame` neuf : `onLoad()` relancé, score perdu, monde vidé, écran qui clignote.

La solution est de créer l'instance dans `initState()`, qui n'est appelé qu'une fois :

```dart
  late final DonjonGame jeu;

  @override
  void initState() {
    super.initState();
    jeu = DonjonGame();
  }
```

`late final` exprime exactement l'intention : affecté plus tard, une seule fois, jamais nul ensuite.

Il existe une alternative fournie par Flame, `GameWidget.controlled(gameFactory: DonjonGame.new)`, où c'est le widget qui possède l'instance. Elle convient quand personne d'autre n'a besoin de la référence. Nous préférons la version explicite : elle rend visible qui possède le jeu, et le chapitre 40 aura besoin d'accéder à `jeu` depuis l'extérieur du `GameWidget`.

### Le type de `overlayBuilderMap`

```dart
overlayBuilderMap: <String, Widget Function(BuildContext, DonjonGame)>{
```

Le type est écrit explicitement. Ce n'est pas obligatoire — l'inférence y arrive dans la plupart des cas — mais dès qu'une entrée de la map renvoie un type légèrement différent, le message d'erreur devient obscur. L'annoter fait apparaître l'erreur sur la bonne ligne.

### `errorBuilder`

Peu de tutoriels le mentionnent, et c'est dommage. Si `onLoad()` lève une exception, sans `errorBuilder` vous obtenez un écran rouge illisible ou, en production, un écran gris. Avec lui, vous affichez le message d'erreur, ce qui vous fait gagner un temps considérable sur mobile où la console n'est pas toujours sous la main.

> **À retenir.** L'instance de jeu se crée dans `initState()`, jamais dans `build()`. `MaterialApp` n'est pas décoratif : le `Navigator` et le `Theme` sont nécessaires.

---

## 35.27 — Lancer le jeu et vérifier la navigation

Tout est écrit. Vérifions.

### Lancer

Le plus rapide pendant le développement est la cible bureau :

```bash
flutter run -d windows
```

```bash
flutter run -d linux
```

```bash
flutter run -d macos
```

Sur le Web :

```bash
flutter run -d chrome
```

Sur un téléphone Android branché :

```bash
flutter devices
flutter run -d <identifiant-du-téléphone>
```

### Ce que vous devez voir, dans l'ordre

```text
1. Un fond très sombre avec un indicateur circulaire orange
   et le texte « Préparation du donjon… »          ← loadingBuilder

2. Le menu : titre DONJON / DE DART, quatre boutons,
   « CONTINUER » grisé                              ← overlay menu_principal

3. Après le clic sur JOUER : l'indicateur circulaire
   et « Construction du niveau… »                   ← overlay chargement

4. Une salle rectangulaire, murs violets, deux plateformes
   brunes, un rectangle bleu, un texte gris.
   En haut : « Score 0   Vies 3   Niveau 1/3 » et un bouton MENU

5. Après le clic sur MENU (ou la touche Échap) :
   retour au menu, la salle a disparu
```

### La console, avec le mode debug activé

```text
[DonjonGame] Chargement -> Menu principal
[DonjonGame] Menu principal -> Chargement
[DonjonGame] Chargement -> En jeu
[DonjonGame] En jeu -> Menu principal
```

Ces quatre lignes sont la preuve que la machine à états fonctionne. Si l'une manque, ou si une transition inattendue apparaît, vous savez immédiatement où chercher.

### La liste de vérification

Passez-la point par point. Chaque ligne teste une chose précise du chapitre.

| # | Test | Attendu | Section |
| --- | --- | --- | --- |
| 1 | Lancer l'application | menu affiché, pas d'écran noir | 35.26 |
| 2 | Redimensionner la fenêtre | le menu reste centré, aucun débordement | 35.18 |
| 3 | Cliquer sur JOUER | chargement puis salle | 35.24 |
| 4 | Cliquer sur MENU | retour au menu, salle disparue | 35.13 |
| 5 | Rejouer, puis revenir | **aucun** doublon de murs | 35.13 |
| 6 | Touche Échap en jeu | retour au menu | 35.28 |
| 7 | Touche F1 en jeu | contours blancs et coordonnées affichés | 35.28 |
| 8 | Bouton CONTINUER | grisé, info-bulle au survol | 35.20 |
| 9 | Bouton OPTIONS | boîte de dialogue, curseurs mobiles | 35.21 |
| 10 | Interrupteur « Mode debug » | agit immédiatement | 35.21 |
| 11 | Bouton QUITTER (bureau) | l'application se ferme | 35.22 |
| 12 | Bouton QUITTER (Web) | message explicatif | 35.22 |

### Le test numéro 5 mérite une explication

C'est le test qui échoue chez la plupart des gens la première fois. Enchaînez : JOUER, MENU, JOUER, MENU, JOUER. Puis activez F1.

Si `viderLeMonde()` était absent ou mal placé, vous verriez, au bout de trois parties, **trois salles superposées** — invisible à l'œil nu puisqu'elles sont identiques, mais parfaitement visible en mode debug, où les contours se superposent. Et au chapitre 37, ce seraient trois gobelins au même endroit.

Une manière de le vérifier sans le mode debug : ajoutez temporairement, à la fin de `demarrerPartie()` :

```dart
debugPrint('Composants dans le monde : ${monde.children.length}');
```

**Résultat attendu, identique à chaque partie :**

```text
Composants dans le monde : 8
```

Si le nombre grimpe à 16 puis 24, votre monde n'est pas vidé.

### Si rien ne s'affiche

Un écran noir vient presque toujours d'un overlay activé sans entrée dans `overlayBuilderMap` ; un jeu qui se relance en boucle vient d'un `DonjonGame()` construit dans `build()` ; un `No Navigator` au clic sur OPTIONS vient d'un `MaterialApp` manquant. Le tableau de la section 35.30 recense les autres cas.

> **À retenir.** Une liste de vérification écrite vaut mieux qu'un « ça a l'air de marcher ». Le test numéro 5 est celui qui attrape le bug le plus coûteux.

---

## 35.28 — Le mode debug et les raccourcis de développement

### `debugMode`

Tout composant Flame possède une propriété `debugMode`. Quand elle vaut `true`, le composant dessine par-dessus lui-même son contour et ses coordonnées.

La propriété est **héritée** : la poser sur le jeu l'active pour tout l'arbre.

```dart
jeu.debugMode = true;
```

Ce qui apparaît :

- le rectangle englobant de chaque `PositionComponent`, en blanc ;
- sa position, en petit, près de son ancre ;
- les hitbox, dès le chapitre 36, en couleur.

C'est l'outil de diagnostic numéro un pour tout ce qui touche à la position et à la collision. Prenez l'habitude de l'activer au moindre doute.

### Les raccourcis clavier

Le mixin `HasKeyboardHandlerComponents` fournit au jeu une méthode `onKeyEvent` que l'on peut surcharger pour les touches **globales**, c'est-à-dire celles qui ne concernent aucun composant en particulier. C'est le schéma annoncé au chapitre 30.

```dart
  /// Touches globales du jeu.
  ///
  /// Les touches propres au joueur (déplacement, saut, attaque) seront gérées
  /// par le composant `Joueur` lui-même au chapitre 36 : il suffira de ne pas
  /// les intercepter ici.
  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent) {
      // Échap : quitter la partie et revenir au menu.
      if (event.logicalKey == LogicalKeyboardKey.escape) {
        if (etat == GameState.enJeu) {
          retournerAuMenu();
          return KeyEventResult.handled;
        }
      }

      // F1 : afficher ou masquer les informations de debug.
      if (event.logicalKey == LogicalKeyboardKey.f1) {
        debugMode = !debugMode;
        debugPrint('[DonjonGame] debugMode = $debugMode');
        return KeyEventResult.handled;
      }

      // F2 : imprimer un état complet dans la console (voir 35.32).
    }

    // Tout le reste descend vers les composants (chapitre 36).
    return super.onKeyEvent(event, keysPressed);
  }
```

Et la méthode de retour au menu, appelée aussi par le bouton du HUD :

```dart
  /// Abandonne la partie en cours et revient au menu principal.
  void retournerAuMenu() {
    changerEtat(GameState.menu);
  }
```

### Trois précisions sur ce code

**`event is KeyDownEvent`.** Sans ce test, chaque appui déclencherait l'action au moins deux fois : une fois à l'enfoncement (`KeyDownEvent`), une fois au relâchement (`KeyUpEvent`), et une fois par répétition automatique (`KeyRepeatEvent`). Un `debugMode` qui bascule deux fois donne l'impression que la touche ne fait rien.

**`return super.onKeyEvent(event, keysPressed);`.** C'est la ligne à ne pas oublier. Sans elle, aucun composant ne recevrait jamais d'événement clavier, et le joueur du chapitre 36 serait paralysé.

**`KeyEventResult.handled` arrête la propagation.** Une touche consommée par le jeu ne descend pas aux composants. C'est ce que l'on veut pour Échap et F1 : on ne veut pas que le héros saute en même temps qu'on ouvre le menu.

### Un tableau des raccourcis, à tenir à jour

| Touche | Effet | Disponible dans | Chapitre |
| --- | --- | --- | --- |
| Échap | retour au menu | `enJeu` | 35 |
| F1 | bascule `debugMode` | partout | 35 |
| F2 | état complet dans la console | partout | 35 |
| P | pause | `enJeu` | 40 |
| Flèches / QSDZ / WASD | déplacement | `enJeu` | 36 |
| Espace | saut | `enJeu` | 36 |
| E | attaque | `enJeu` | 37 |

### Faut-il retirer ces raccourcis avant publication ?

Non, à deux conditions : qu'ils ne donnent aucun avantage (F1 et F2 n'en donnent pas), et qu'ils n'existent qu'en mode debug si c'est le cas. Pour n'activer un raccourci qu'en développement, entourez-le de `kDebugMode` :

```dart
if (kDebugMode && event.logicalKey == LogicalKeyboardKey.f3) {
  vies = 99; // triche de développement, absente de la version publiée
  return KeyEventResult.handled;
}
```

`kDebugMode` étant une constante de compilation, le code de triche **disparaît** du binaire de production.

> **À retenir.** Le jeu intercepte les touches globales et fait descendre le reste avec `super.onKeyEvent(...)`. Oublier cette ligne paralyse tous les composants.

---

## 35.29 — Ce que le projet fait à la fin de ce chapitre

Faisons le point honnêtement, en deux colonnes.

### Ce qui fonctionne

| Fonctionnalité | Fichier | Section |
| --- | --- | --- |
| L'application se lance et affiche un menu | `main.dart` | 35.26 |
| Un écran de chargement pendant `onLoad()` | `main.dart` | 35.25 |
| Un menu stylisé sans aucune image | `ecrans/menu_principal.dart` | 35.23 |
| Un bouton « Jouer » qui démarre une partie | `donjon_game.dart` | 35.24 |
| Un bouton « Continuer » désactivé et documenté | `ecrans/menu_principal.dart` | 35.20 |
| Une boîte d'options avec un réglage actif (debug) | `ecrans/menu_principal.dart` | 35.21 |
| Un bouton « Quitter » conscient de la plateforme | `ecrans/menu_principal.dart` | 35.22 |
| Une machine à états à six états | `core/game_state.dart` | 35.8 |
| Des transitions centralisées dans `changerEtat()` | `donjon_game.dart` | 35.13 |
| Des overlays qui suivent l'état sans jamais diverger | `donjon_game.dart` | 35.13 |
| Un monde vidé entre deux parties | `donjon_game.dart` | 35.13 |
| Une caméra centrée, zoom 2 | `donjon_game.dart` | 35.12 |
| Une salle de démonstration en rectangles | `donjon_game.dart` | 35.24 |
| Un HUD provisoire avec retour au menu | `main.dart` | 35.26 |
| Les raccourcis Échap, F1, F2 | `donjon_game.dart` | 35.28 |
| Toutes les valeurs de réglage centralisées | `config/constantes.dart` | 35.5 |
| Toutes les couleurs centralisées | `config/palette.dart` | 35.6 |

### Ce qui ne fonctionne pas encore

| Manque | Conséquence visible | Chapitre |
| --- | --- | --- |
| Aucun joueur | le rectangle bleu ne bouge pas | 36 |
| Aucune gravité, aucun saut | rien ne tombe | 36 |
| Aucune collision réelle | rien ne s'arrête sur le sol | 36 |
| Aucun ennemi | la salle est inoffensive | 37 |
| `vies` n'est jamais décrémenté | on ne peut pas perdre | 37 |
| Aucun objet à ramasser | `score` reste à 0 | 38 |
| Le HUD est un widget provisoire | il ne se met pas à jour tout seul | 38 |
| Un seul décor, en dur | `niveauCourant` ne sert à rien | 39 |
| Aucune porte, aucun boss | pas de fin | 39 |
| Aucun son | jeu muet | 40 |
| États `pause`, `gameOver`, `victoire` inatteignables | leurs overlays n'existent pas | 40 |
| `meilleurScore` toujours à 0 | rien n'est sauvegardé | 40 |

### Le décompte

Sept fichiers, environ 600 lignes de code, dont **zéro** sera jeté. Chacun des cinq chapitres suivants ajoutera des fichiers et complétera ceux-ci ; aucun ne les réécrira.

Trois bouchons seulement sont posés, et tous sont marqués dans le code :

| Bouchon | Où | Retiré au chapitre |
| --- | --- | --- |
| `_salleDeDemonstration()` | `donjon_game.dart` | 39 |
| `HudProvisoire` | `main.dart` | 38 |
| `ReglagesProvisoires` et `_peutContinuer` | `ecrans/menu_principal.dart` | 40 |

### Le vrai résultat du chapitre

Il n'est pas visuel. Vous avez une **architecture** :

- un seul endroit qui change d'écran ;
- un seul endroit qui définit les règles chiffrées ;
- un seul endroit qui définit les couleurs ;
- une frontière nette entre l'interface Flutter et le monde Flame ;
- une place réservée, nommée et documentée pour chaque fonctionnalité à venir.

C'est exactement ce qui manquait aux huit démonstrations de la PARTIE 2B, et c'est ce qui permettra aux cinq chapitres suivants de n'être que des ajouts.

> **À retenir.** À la fin du chapitre 35, le jeu ne se joue pas encore, mais il est **prêt à être rempli**. C'est le seul chapitre de la PARTIE 2C dont ce sera le cas.

---

## 35.30 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Écran noir au lancement, aucun message | un overlay actif n'a pas d'entrée dans `overlayBuilderMap` | ajouter le constructeur dans la map, ou ne pas activer cet état tant qu'il n'existe pas |
| `initialActiveOverlay: 'chargement'` — paramètre inconnu | le paramètre est au pluriel et attend une **liste** | `initialActiveOverlays: const ['chargement']` |
| Le jeu se relance en boucle, le score repart à zéro | `DonjonGame()` est construit dans `build()` | créer l'instance dans `initState()` d'un `StatefulWidget` |
| `No Material widget found. ElevatedButton requires a Material ancestor` | l'overlay n'a pas de `Material` au-dessus de lui | poser `Material(color: Colors.transparent, child: ...)` en racine de chaque overlay |
| `Navigator operation requested with a context that does not include a Navigator` | `showDialog` appelé sans `MaterialApp` | envelopper l'application dans un `MaterialApp` |
| Le décor s'accumule à chaque partie | le monde n'est pas vidé entre deux parties | appeler `viderLeMonde()` dans `changerEtat` (retour menu) **et** dans `demarrerPartie()` |
| `Concurrent modification during iteration` en vidant le monde | on itère sur `monde.children` tout en le modifiant | `monde.removeAll(monde.children.toList())` |
| L'écran devient noir après un changement d'état | l'ancien état a été écrasé avant de retirer son overlay | mémoriser `ancienEtat` **avant** d'affecter `etat` |
| Le jeu reste figé après une pause | un chemin de sortie n'appelle pas `resumeEngine()` | ne jamais appeler `pauseEngine`/`resumeEngine` ailleurs que dans `changerEtat()` |
| Les boutons du menu de pause ne réagissent pas | l'écran de pause a été écrit en composants Flame, donc figé par `pauseEngine()` | tout écran affiché pendant une pause doit être un overlay Flutter |
| Un mur ne s'affiche pas, aucune erreur | couleur écrite sans canal alpha : `Color(0x3A3550)` | `Color(0xFF3A3550)` |
| `Undefined name 'Palette'` ou import ambigu sur `Palette` | `package:flame/palette.dart` importé par erreur | retirer cet import ; nous n'utilisons que notre propre `Palette` |
| Le joueur ne bouge pas quand la caméra se déplace | composant ajouté avec `add(...)` au lieu de `monde.add(...)` | ajouter au monde tout ce qui appartient au niveau |
| Aucune collision détectée, le héros traverse les murs | `HasCollisionDetection` absent du jeu | ajouter le mixin sur `DonjonGame` |
| Aucun composant ne reçoit les touches | `super.onKeyEvent(...)` non appelé dans la surcharge du jeu | terminer la surcharge par `return super.onKeyEvent(event, keysPressed);` |
| Une touche déclenche son action deux ou trois fois | absence de test `event is KeyDownEvent` | filtrer le type d'événement avant d'agir |
| `KeyboardEvents` et `HasKeyboardHandlerComponents` ensemble | les deux mixins déclarent `onKeyEvent` | n'en garder qu'un : `HasKeyboardHandlerComponents` |
| `Target of URI doesn't exist: 'assets/images/'` au build | dossier déclaré dans `pubspec.yaml` mais absent du disque | créer le dossier (même vide) avant de le déclarer |
| `ServicesBinding.defaultBinaryMessenger was accessed before the binding was initialized` | `SystemChrome` appelé avant l'initialisation | `WidgetsFlutterBinding.ensureInitialized();` en première ligne de `main()` |
| `BOTTOM OVERFLOWED BY n PIXELS` sur le menu | la colonne dépasse en paysage sur petit écran | envelopper dans un `SingleChildScrollView` |
| Le zoom rend l'image floue | facteur de zoom non entier (1,5 ; 1,7) | utiliser un entier : 2, 3, 4 |
| `withOpacity` signalé comme déprécié | méthode remplacée dans les Flutter récents | `couleur.withValues(alpha: 0.72)` |
| `Platform.isAndroid` plante sur le Web | `dart:io` n'existe pas en navigateur | tester `kIsWeb` en premier, `Platform.*` seulement ensuite |

---

## 35.31 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Projet de la PARTIE 2C | un seul jeu, « Donjon de Dart », construit du chapitre 35 au 40, jouable à chaque étape |
| Cahier des charges | une page : identité, boucle de jeu, chiffres, refus explicites, critères d'acceptation |
| Arborescence | `config/`, `core/`, `composants/`, `niveaux/`, `hud/`, `services/`, `ecrans/` |
| `hud/` contre `ecrans/` | composants Flame figés par la pause, contre widgets Flutter qui y survivent |
| `pubspec.yaml` | `flame: ^1.38.0` ; les dossiers d'assets doivent exister avant d'être déclarés |
| `Constantes` | toutes les valeurs de règle, `static const`, constructeur privé |
| `Palette` | toutes les couleurs et les styles ; alpha `FF` obligatoire en tête d'un `Color` |
| Nombre magique | un nombre sans nom est un bug en attente et un réglage impossible |
| `GameState` | six états exclusifs ; un enum interdit les combinaisons absurdes |
| Machine à états | les transitions se dessinent avant de coder ; toute flèche absente du tableau est un bug |
| `DonjonGame` | hérite de `FlameGame` ; porte l'état, le score, les vies, le niveau |
| `monde` | alias de `world`, affecté dans `onLoad()` ; on ne recrée jamais le `World` |
| `camera` | fournie par `FlameGame` ; on règle `viewfinder.anchor` et `viewfinder.zoom` |
| Mixins du jeu | `HasCollisionDetection` (ch. 36) et `HasKeyboardHandlerComponents` (ch. 36), rien d'autre |
| `changerEtat()` | unique point de navigation : overlay retiré, overlay ajouté, moteur, nettoyage |
| `pauseEngine()` | fige tout ce qui est Flame, rien de ce qui est Flutter |
| Overlays | widgets Flutter allumés par leur nom depuis le jeu |
| `overlayBuilderMap` | associe un nom à `(context, game) => Widget` ; un nom activé sans entrée fait planter |
| `initialActiveOverlays` | au pluriel, liste, appliquée au tout premier rendu |
| Classe `Overlays` | transforme une faute de frappe en erreur de compilation |
| `MenuPrincipal` | `StatelessWidget` qui reçoit `DonjonGame` par son constructeur |
| Bouton désactivé | `onPressed: null` grise le bouton ; la façon propre d'annoncer une fonctionnalité |
| Style sans image | `letterSpacing`, `shadows`, dégradé, bordure, icônes Material |
| `demarrerPartie()` | chargement, remise à zéro, vidage, construction, cadrage, entrée en jeu |
| Écrans de chargement | `loadingBuilder` pour le jeu, overlay `chargement` pour le niveau |
| `main.dart` | `MaterialApp` pour le `Navigator` et le `Theme` ; instance de jeu dans `initState()` |
| `debugMode` | contours et positions ; hérité par tout l'arbre |
| Raccourcis | Échap (menu), F1 (debug), F2 (état) ; `super.onKeyEvent` obligatoire à la fin |
| État du projet | l'architecture est complète, le contenu ne l'est pas |

---

## 35.32 — Code complet du chapitre

Sept fichiers. Recopiez-les dans cet ordre.

### `pubspec.yaml`

```yaml
name: donjon_de_dart
description: "Jeu de plateformes 2D réalisé avec Flutter et Flame — PARTIE 2C."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=3.12.0 <4.0.0"
  flutter: ">=3.44.0"

dependencies:
  flutter:
    sdk: flutter

  # Moteur de jeu 2D.
  flame: ^1.38.0

  # AJOUTÉ AU CHAPITRE 40 seulement :
  # flame_audio: ^2.12.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/audio/
```

### `lib/config/constantes.dart`

```dart
/// Toutes les valeurs de réglage du jeu.
///
/// Aucun nombre « magique » ne doit apparaître ailleurs dans le projet.
class Constantes {
  Constantes._();

  /// Côté d'une tuile de niveau, en pixels du monde.
  static const double tailleTuile = 32.0;

  /// Accélération verticale, en pixels par seconde au carré.
  static const double gravite = 1200.0;

  /// Vitesse horizontale du joueur, en pixels par seconde.
  static const double vitesseJoueur = 180.0;

  /// Impulsion de saut. NÉGATIVE : l'axe Y descend.
  static const double forceSaut = -520.0;

  /// Vitesse de chute maximale, pour éviter le tunneling.
  static const double vitesseMaxChute = 900.0;

  /// Nombre de vies au début d'une partie.
  static const int viesDepart = 3;

  /// Points de vie maximum du joueur.
  static const double pvJoueurMax = 100.0;

  /// Durée d'invincibilité après des dégâts, en secondes.
  static const double dureeInvincibilite = 1.2;

  /// Facteur de zoom de la caméra (entier, pour un rendu net).
  static const double zoomCamera = 2.0;

  /// Nombre de niveaux de la campagne.
  static const int nombreNiveaux = 3;
}

/// Noms des overlays Flutter affichés par-dessus le jeu.
///
/// Ces chaînes servent de clés dans `overlayBuilderMap` et dans
/// `overlays.add`. Les centraliser transforme une faute de frappe en
/// erreur de compilation.
class Overlays {
  Overlays._();

  static const String menuPrincipal = 'menu_principal';
  static const String hud = 'hud';
  static const String pause = 'pause';
  static const String gameOver = 'game_over';
  static const String victoire = 'victoire';
  static const String chargement = 'chargement';
}
```

### `lib/config/palette.dart`

```dart
import 'package:flutter/material.dart';

/// Toutes les couleurs du jeu, et quelques styles de texte.
///
/// Aucune valeur `Color(0xFF...)` ne doit apparaître ailleurs dans le projet.
class Palette {
  Palette._();

  // --- Fonds -------------------------------------------------------------
  static const Color fondJeu = Color(0xFF14121C);
  static const Color fondMenu = Color(0xFF0C0A12);
  static const Color panneau = Color(0xFF1E1B2B);

  // --- Décor -------------------------------------------------------------
  static const Color mur = Color(0xFF3A3550);
  static const Color murClair = Color(0xFF4C4668);
  static const Color plateforme = Color(0xFF6B5B3E);
  static const Color porte = Color(0xFF8C6B2F);

  // --- Entités -----------------------------------------------------------
  static const Color joueur = Color(0xFF4FC3F7);
  static const Color joueurTouche = Color(0xFFFF8A80);
  static const Color gobelin = Color(0xFF7CB342);
  static const Color chauvesouris = Color(0xFF9575CD);
  static const Color boss = Color(0xFFD32F2F);
  static const Color projectile = Color(0xFFFFD54F);

  // --- Objets ------------------------------------------------------------
  static const Color piece = Color(0xFFFFC107);
  static const Color potion = Color(0xFFEF5350);
  static const Color cle = Color(0xFFFFEE58);

  // --- Interface ---------------------------------------------------------
  static const Color accent = Color(0xFFFFB300);
  static const Color accentSombre = Color(0xFF8D6E00);
  static const Color texte = Color(0xFFF3F1FA);
  static const Color texteFaible = Color(0xFF9E99B5);
  static const Color danger = Color(0xFFE53935);
  static const Color succes = Color(0xFF66BB6A);

  // --- Barres ------------------------------------------------------------
  static const Color barreFond = Color(0xFF2A2636);
  static const Color barreVie = Color(0xFFE53935);
  static const Color barreEnergie = Color(0xFF29B6F6);

  // --- Styles de texte ---------------------------------------------------
  static const TextStyle titre = TextStyle(
    fontSize: 52,
    fontWeight: FontWeight.w900,
    letterSpacing: 6,
    color: texte,
  );

  static const TextStyle sousTitre = TextStyle(
    fontSize: 16,
    letterSpacing: 3,
    color: texteFaible,
  );

  static const TextStyle bouton = TextStyle(
    fontSize: 18,
    fontWeight: FontWeight.w600,
    letterSpacing: 1.5,
  );

  /// Fabrique une `Paint` opaque de la couleur demandée.
  ///
  /// Pour un composant qui dessine à 60 images par seconde, stockez le
  /// résultat dans un champ `static final` plutôt que de rappeler la méthode.
  static Paint peinture(Color couleur) => Paint()..color = couleur;
}
```

### `lib/core/game_state.dart`

```dart
/// Les états possibles du jeu.
///
/// À tout instant, `DonjonGame.etat` vaut exactement une de ces six valeurs.
enum GameState {
  /// Ressources en cours de préparation, ou niveau en construction.
  chargement,

  /// Menu principal. Le monde est vide.
  menu,

  /// Une partie est en cours : le seul état où le monde évolue.
  enJeu,

  /// Partie suspendue par le joueur (chapitre 40).
  pause,

  /// Le joueur a perdu sa dernière vie (chapitre 40).
  gameOver,

  /// Le joueur a terminé le dernier niveau (chapitre 40).
  victoire,
}

extension GameStateInfos on GameState {
  /// Vrai si une partie existe, qu'elle soit en cours ou suspendue.
  bool get partieEnCours =>
      this == GameState.enJeu || this == GameState.pause;

  /// Vrai si le moteur de jeu doit avancer dans cet état.
  bool get moteurActif => this == GameState.enJeu;

  /// Libellé lisible, pour les messages de debug.
  String get libelle => switch (this) {
        GameState.chargement => 'Chargement',
        GameState.menu => 'Menu principal',
        GameState.enJeu => 'En jeu',
        GameState.pause => 'Pause',
        GameState.gameOver => 'Game Over',
        GameState.victoire => 'Victoire',
      };
}
```

### `lib/donjon_game.dart`

```dart
import 'package:flame/camera.dart';
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import 'config/constantes.dart';
import 'config/palette.dart';
import 'core/game_state.dart';

/// Le jeu « Donjon de Dart ».
///
/// Une seule instance existe pendant toute la vie de l'application : elle est
/// créée dans `main.dart` et confiée au `GameWidget`.
class DonjonGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  /// Alias du `World` fourni par `FlameGame`, affecté dans `onLoad()`.
  late final World monde;

  /// L'état courant. Ne le modifiez jamais directement : passez par
  /// `changerEtat()`.
  GameState etat = GameState.chargement;

  /// Score de la partie en cours (chapitre 38).
  int score = 0;

  /// Vies restantes (chapitre 37).
  int vies = Constantes.viesDepart;

  /// Index du niveau courant (chapitre 39).
  int niveauCourant = 0;

  /// Meilleur score, chargé et sauvegardé au chapitre 40.
  int meilleurScore = 0;

  @override
  Color backgroundColor() => Palette.fondJeu;

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // `world` et `camera` existent déjà : on les configure, on ne les crée pas.
    monde = world;

    camera.viewfinder.anchor = Anchor.center;
    camera.viewfinder.zoom = Constantes.zoomCamera;

    // Chargement des ressources : rien pour l'instant (chapitres 29 et 40).
    changerEtat(GameState.menu);
  }

  // --- Machine à états ----------------------------------------------------

  /// L'overlay Flutter associé à chaque état. Fonction pure.
  static String overlayDeLEtat(GameState etat) => switch (etat) {
        GameState.chargement => Overlays.chargement,
        GameState.menu => Overlays.menuPrincipal,
        GameState.enJeu => Overlays.hud,
        GameState.pause => Overlays.pause,
        GameState.gameOver => Overlays.gameOver,
        GameState.victoire => Overlays.victoire,
      };

  /// Change l'état du jeu. UNIQUE point d'entrée de la navigation.
  void changerEtat(GameState nouvelEtat) {
    if (nouvelEtat == etat) {
      return;
    }

    final GameState ancienEtat = etat;
    etat = nouvelEtat;

    if (debugMode) {
      debugPrint('[DonjonGame] ${ancienEtat.libelle} -> ${nouvelEtat.libelle}');
    }

    // Un seul overlay d'état visible à la fois.
    overlays.remove(overlayDeLEtat(ancienEtat));
    overlays.add(overlayDeLEtat(nouvelEtat));

    // Le moteur ne tourne que pendant le jeu.
    if (nouvelEtat == GameState.enJeu) {
      resumeEngine();
    } else if (nouvelEtat == GameState.pause ||
        nouvelEtat == GameState.gameOver ||
        nouvelEtat == GameState.victoire) {
      pauseEngine();
    }

    // Nettoyage propre à l'état d'arrivée.
    if (nouvelEtat == GameState.menu) {
      viderLeMonde();
    }
  }

  /// Abandonne la partie en cours et revient au menu principal.
  void retournerAuMenu() {
    changerEtat(GameState.menu);
  }

  /// Retire tous les composants du monde.
  ///
  /// `.toList()` est indispensable : `monde.children` est la liste vivante des
  /// enfants (bug de modification concurrente, chapitre 26).
  void viderLeMonde() {
    monde.removeAll(monde.children.toList());
  }

  // --- Cycle d'une partie -------------------------------------------------

  /// Démarre une nouvelle partie depuis le début.
  Future<void> demarrerPartie() async {
    // 1. Écran de chargement avant tout travail.
    changerEtat(GameState.chargement);

    // 2. Remise à zéro. `meilleurScore` n'est PAS touché.
    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = 0;

    // 3. On repart d'un monde vide.
    viderLeMonde();

    // 4. PROVISOIRE (chapitre 35) : une salle de démonstration.
    //    Chapitre 39 : await chargerNiveau(niveauCourant);
    await monde.addAll(_salleDeDemonstration());

    // 5. Cadrage. Chapitre 36 : camera.follow(joueur);
    camera.viewfinder.position = Vector2.zero();

    // 6. Pause artificielle pour que l'écran de chargement soit visible.
    //    À RETIRER au chapitre 39.
    await Future<void>.delayed(const Duration(milliseconds: 350));

    // 7. En jeu.
    changerEtat(GameState.enJeu);
  }

  /// Décor provisoire du chapitre 35.
  ///
  /// Quatre murs, deux plateformes, un rectangle à l'emplacement du futur
  /// joueur. Remplacé par `chargerNiveau(index)` au chapitre 39.
  List<Component> _salleDeDemonstration() {
    const double t = Constantes.tailleTuile;
    const double largeur = 20 * t; // 640 unités de monde
    const double hauteur = 12 * t; // 384 unités de monde
    const double gauche = -largeur / 2;
    const double haut = -hauteur / 2;

    RectangleComponent bloc(double x, double y, double l, double h, Color c) {
      return RectangleComponent(
        position: Vector2(x, y),
        size: Vector2(l, h),
        paint: Palette.peinture(c),
      );
    }

    return <Component>[
      bloc(gauche, haut, largeur, t, Palette.mur), // plafond
      bloc(gauche, haut + hauteur - t, largeur, t, Palette.mur), // sol
      bloc(gauche, haut, t, hauteur, Palette.mur), // mur gauche
      bloc(gauche + largeur - t, haut, t, hauteur, Palette.mur), // mur droit
      bloc(gauche + 4 * t, haut + 8 * t, 4 * t, t / 2, Palette.plateforme),
      bloc(gauche + 12 * t, haut + 6 * t, 4 * t, t / 2, Palette.plateforme),
      // Emplacement du héros : un simple rectangle bleu (chapitre 36).
      bloc(gauche + 2 * t, haut + 9 * t, t * 0.75, t * 1.5, Palette.joueur),
      TextComponent(
        text: 'Salle de démonstration — chapitre 35',
        position: Vector2(0, haut + 2.5 * t),
        anchor: Anchor.center,
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 12, color: Palette.texteFaible),
        ),
      ),
    ];
  }

  // --- Entrées globales ---------------------------------------------------

  /// Touches globales du jeu.
  ///
  /// Les touches propres au joueur seront gérées par le composant `Joueur`
  /// lui-même au chapitre 36 : il suffit de ne pas les intercepter ici.
  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent) {
      // Échap : quitter la partie.
      if (event.logicalKey == LogicalKeyboardKey.escape) {
        if (etat == GameState.enJeu) {
          retournerAuMenu();
          return KeyEventResult.handled;
        }
      }

      // F1 : contours et positions.
      if (event.logicalKey == LogicalKeyboardKey.f1) {
        debugMode = !debugMode;
        debugPrint('[DonjonGame] debugMode = $debugMode');
        return KeyEventResult.handled;
      }

      // F2 : état complet dans la console.
      if (event.logicalKey == LogicalKeyboardKey.f2) {
        debugPrint(
          '[DonjonGame] etat=${etat.libelle} score=$score vies=$vies '
          'niveau=$niveauCourant composants=${monde.children.length} '
          'overlays=${overlays.activeOverlays}',
        );
        return KeyEventResult.handled;
      }
    }

    // Tout le reste descend vers les composants (chapitre 36).
    return super.onKeyEvent(event, keysPressed);
  }
}
```

### `lib/ecrans/menu_principal.dart`

```dart
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/palette.dart';
import '../donjon_game.dart';

/// Réglages du joueur, en mémoire uniquement.
///
/// PROVISOIRE (chapitre 35) : perdus à la fermeture, et le volume n'est
/// branché sur rien. Chapitre 40 : `SauvegardeService` + `AudioService`.
class ReglagesProvisoires {
  ReglagesProvisoires._();

  static double volumeMusique = 0.6;
  static double volumeEffets = 0.8;
}

/// Le menu principal du jeu, affiché en overlay quand l'état vaut
/// `GameState.menu`.
class MenuPrincipal extends StatelessWidget {
  const MenuPrincipal({super.key, required this.jeu});

  /// La référence vers le jeu, transmise par `overlayBuilderMap`.
  final DonjonGame jeu;

  /// Une partie sauvegardée est-elle disponible ?
  ///
  /// PROVISOIRE (chapitre 35) : toujours `false`.
  /// Chapitre 40 : `SauvegardeService.instance.aUneSauvegarde`.
  bool get _peutContinuer => false;

  @override
  Widget build(BuildContext context) {
    return Material(
      color: Colors.transparent,
      child: DecoratedBox(
        decoration: const BoxDecoration(
          gradient: RadialGradient(
            center: Alignment(0, -0.6),
            radius: 1.1,
            colors: <Color>[Palette.panneau, Palette.fondMenu],
          ),
        ),
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.symmetric(vertical: 24),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                const TitreDuJeu(),
                const SizedBox(height: 28),
                Container(
                  padding: const EdgeInsets.symmetric(
                    horizontal: 40,
                    vertical: 28,
                  ),
                  decoration: BoxDecoration(
                    color: Palette.fondMenu.withValues(alpha: 0.72),
                    border: Border.all(color: Palette.mur, width: 3),
                    borderRadius: BorderRadius.circular(10),
                  ),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: <Widget>[
                      BoutonMenu(
                        libelle: 'JOUER',
                        icone: Icons.play_arrow,
                        principal: true,
                        onPressed: () async {
                          await jeu.demarrerPartie();
                        },
                      ),
                      const SizedBox(height: 12),
                      BoutonMenu(
                        libelle: 'CONTINUER',
                        icone: Icons.history,
                        onPressed: _peutContinuer
                            ? () async {
                                // Chapitre 40 : reprendre au niveau sauvegardé.
                                await jeu.demarrerPartie();
                              }
                            : null,
                        infoBulle: _peutContinuer
                            ? 'Reprendre la partie sauvegardée'
                            : 'Aucune partie sauvegardée '
                                '(disponible au chapitre 40)',
                      ),
                      const SizedBox(height: 12),
                      BoutonMenu(
                        libelle: 'OPTIONS',
                        icone: Icons.settings,
                        onPressed: () {
                          showDialog<void>(
                            context: context,
                            builder: (BuildContext context) =>
                                DialogueOptions(jeu: jeu),
                          );
                        },
                      ),
                      const SizedBox(height: 12),
                      BoutonMenu(
                        libelle: 'QUITTER',
                        icone: Icons.exit_to_app,
                        onPressed: () => _quitter(context),
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 20),
                const Text(
                  'Chapitre 35 — v1.0.0',
                  style: TextStyle(color: Palette.texteFaible, fontSize: 12),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }

  /// Quitte l'application, si la plateforme le permet.
  void _quitter(BuildContext context) {
    if (kIsWeb) {
      showDialog<void>(
        context: context,
        builder: (BuildContext context) => AlertDialog(
          backgroundColor: Palette.panneau,
          title: const Text('Quitter'),
          content: const Text(
            'Dans un navigateur, le jeu ne peut pas se fermer lui-même.\n'
            'Fermez simplement cet onglet.',
          ),
          actions: <Widget>[
            TextButton(
              onPressed: () => Navigator.of(context).pop(),
              child: const Text('COMPRIS'),
            ),
          ],
        ),
      );
      return;
    }

    SystemNavigator.pop();
  }
}

/// Le titre du jeu, dessiné uniquement avec du texte.
class TitreDuJeu extends StatelessWidget {
  const TitreDuJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text(
          'DONJON',
          style: Palette.titre.copyWith(
            shadows: const <Shadow>[
              Shadow(color: Palette.accent, blurRadius: 24),
              Shadow(color: Palette.fondMenu, offset: Offset(3, 3)),
            ],
          ),
        ),
        Text(
          'DE DART',
          style: Palette.titre.copyWith(
            fontSize: 34,
            color: Palette.accent,
            letterSpacing: 12,
          ),
        ),
        const SizedBox(height: 12),
        const Text('UN JEU DE PLATEFORMES EN FLAME', style: Palette.sousTitre),
      ],
    );
  }
}

/// Un bouton du menu principal.
///
/// `principal` met le bouton en valeur ; `onPressed: null` le désactive.
class BoutonMenu extends StatelessWidget {
  const BoutonMenu({
    super.key,
    required this.libelle,
    required this.icone,
    required this.onPressed,
    this.principal = false,
    this.infoBulle,
  });

  final String libelle;
  final IconData icone;
  final VoidCallback? onPressed;
  final bool principal;
  final String? infoBulle;

  @override
  Widget build(BuildContext context) {
    final Widget bouton = SizedBox(
      width: 260,
      height: 52,
      child: principal
          ? FilledButton.icon(
              onPressed: onPressed,
              icon: Icon(icone),
              label: Text(libelle, style: Palette.bouton),
              style: FilledButton.styleFrom(
                backgroundColor: Palette.accent,
                foregroundColor: Palette.fondMenu,
                disabledBackgroundColor: Palette.accentSombre,
                shape: const RoundedRectangleBorder(
                  borderRadius: BorderRadius.all(Radius.circular(6)),
                ),
              ),
            )
          : OutlinedButton.icon(
              onPressed: onPressed,
              icon: Icon(icone),
              label: Text(libelle, style: Palette.bouton),
              style: OutlinedButton.styleFrom(
                foregroundColor: Palette.texte,
                disabledForegroundColor: Palette.texteFaible,
                side: const BorderSide(color: Palette.mur, width: 2),
                shape: const RoundedRectangleBorder(
                  borderRadius: BorderRadius.all(Radius.circular(6)),
                ),
              ),
            ),
    );

    if (infoBulle == null) {
      return bouton;
    }
    return Tooltip(message: infoBulle!, child: bouton);
  }
}

/// Boîte de dialogue des options.
class DialogueOptions extends StatefulWidget {
  const DialogueOptions({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  State<DialogueOptions> createState() => _DialogueOptionsState();
}

class _DialogueOptionsState extends State<DialogueOptions> {
  late double _musique = ReglagesProvisoires.volumeMusique;
  late double _effets = ReglagesProvisoires.volumeEffets;
  late bool _debug = widget.jeu.debugMode;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      backgroundColor: Palette.panneau,
      title: const Text('OPTIONS', style: TextStyle(letterSpacing: 3)),
      content: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          const Text('Volume de la musique', style: Palette.sousTitre),
          Slider(
            value: _musique,
            activeColor: Palette.accent,
            onChanged: (double valeur) {
              setState(() => _musique = valeur);
              ReglagesProvisoires.volumeMusique = valeur;
              // Chapitre 40 : AudioService.instance.volumeMusique = valeur;
            },
          ),
          const Text('Volume des effets', style: Palette.sousTitre),
          Slider(
            value: _effets,
            activeColor: Palette.accent,
            onChanged: (double valeur) {
              setState(() => _effets = valeur);
              ReglagesProvisoires.volumeEffets = valeur;
              // Chapitre 40 : AudioService.instance.volumeEffets = valeur;
            },
          ),
          const SizedBox(height: 8),
          SwitchListTile(
            contentPadding: EdgeInsets.zero,
            title: const Text('Mode debug', style: Palette.sousTitre),
            subtitle: const Text(
              'Affiche les contours et les positions',
              style: TextStyle(fontSize: 12, color: Palette.texteFaible),
            ),
            value: _debug,
            onChanged: (bool valeur) {
              setState(() => _debug = valeur);
              // Celui-ci fonctionne dès maintenant.
              widget.jeu.debugMode = valeur;
            },
          ),
        ],
      ),
      actions: <Widget>[
        TextButton(
          onPressed: () => Navigator.of(context).pop(),
          child: const Text('FERMER'),
        ),
      ],
    );
  }
}

/// Écran de chargement, utilisé à deux endroits :
/// - `loadingBuilder` du GameWidget, pendant `DonjonGame.onLoad()` ;
/// - overlay `Overlays.chargement`, pendant `demarrerPartie()`.
class EcranChargement extends StatelessWidget {
  const EcranChargement({super.key, this.message = 'Chargement…'});

  final String message;

  @override
  Widget build(BuildContext context) {
    return Material(
      // Opaque : on masque un monde en cours de reconstruction.
      color: Palette.fondMenu,
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const SizedBox(
              width: 44,
              height: 44,
              child: CircularProgressIndicator(
                color: Palette.accent,
                strokeWidth: 3,
              ),
            ),
            const SizedBox(height: 24),
            Text(message, style: Palette.sousTitre),
          ],
        ),
      ),
    );
  }
}
```

### `lib/main.dart`

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import 'config/constantes.dart';
import 'config/palette.dart';
import 'donjon_game.dart';
import 'ecrans/menu_principal.dart';

Future<void> main() async {
  // Obligatoire dès qu'on appelle une API de plateforme avant runApp().
  WidgetsFlutterBinding.ensureInitialized();

  // Le jeu est conçu pour le paysage. Sans effet sur le Web et le bureau.
  await SystemChrome.setPreferredOrientations(<DeviceOrientation>[
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight,
  ]);

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
      theme: ThemeData(
        useMaterial3: true,
        brightness: Brightness.dark,
        scaffoldBackgroundColor: Palette.fondMenu,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Palette.accent,
          brightness: Brightness.dark,
        ),
      ),
      home: const PageDeJeu(),
    );
  }
}

/// La page unique de l'application : le jeu et ses overlays.
///
/// StatefulWidget parce que l'instance de `DonjonGame` doit être créée UNE
/// SEULE FOIS et survivre à toutes les reconstructions du widget.
class PageDeJeu extends StatefulWidget {
  const PageDeJeu({super.key});

  @override
  State<PageDeJeu> createState() => _PageDeJeuState();
}

class _PageDeJeuState extends State<PageDeJeu> {
  late final DonjonGame jeu;

  @override
  void initState() {
    super.initState();
    jeu = DonjonGame();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Palette.fondJeu,
      body: GameWidget<DonjonGame>(
        game: jeu,
        // Affiché par Flame pendant DonjonGame.onLoad().
        loadingBuilder: (BuildContext context) => const EcranChargement(
          message: 'Préparation du donjon…',
        ),
        // Affiché par Flame si onLoad() lève une exception.
        errorBuilder: (BuildContext context, Object erreur) => Material(
          color: Palette.fondMenu,
          child: Center(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Text(
                'Le jeu n\'a pas pu démarrer.\n\n$erreur',
                style: const TextStyle(color: Palette.danger),
                textAlign: TextAlign.center,
              ),
            ),
          ),
        ),
        // L'état initial du jeu est GameState.chargement.
        initialActiveOverlays: const <String>[Overlays.chargement],
        overlayBuilderMap: <String, Widget Function(BuildContext, DonjonGame)>{
          Overlays.chargement: (BuildContext context, DonjonGame game) {
            return const EcranChargement(message: 'Construction du niveau…');
          },
          Overlays.menuPrincipal: (BuildContext context, DonjonGame game) {
            return MenuPrincipal(jeu: game);
          },
          // PROVISOIRE : remplacé par le vrai HUD au chapitre 38.
          Overlays.hud: (BuildContext context, DonjonGame game) {
            return HudProvisoire(jeu: game);
          },
        },
      ),
    );
  }
}

/// HUD provisoire du chapitre 35.
///
/// REMPLACÉ au chapitre 38 par `hud/hud.dart` (composants Flame).
class HudProvisoire extends StatelessWidget {
  const HudProvisoire({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    return Material(
      color: Colors.transparent,
      child: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Text(
                'Score ${jeu.score}   Vies ${jeu.vies}   '
                'Niveau ${jeu.niveauCourant + 1}/${Constantes.nombreNiveaux}',
                style: const TextStyle(
                  color: Palette.texte,
                  fontSize: 16,
                  letterSpacing: 1.2,
                ),
              ),
              const Spacer(),
              const Text(
                'Échap : menu    F1 : debug',
                style: TextStyle(color: Palette.texteFaible, fontSize: 12),
              ),
              const SizedBox(width: 16),
              OutlinedButton(
                onPressed: () => jeu.retournerAuMenu(),
                style: OutlinedButton.styleFrom(
                  foregroundColor: Palette.texte,
                  side: const BorderSide(color: Palette.mur, width: 2),
                ),
                child: const Text('MENU'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 35.33 — Exercices

Les exercices s'enchaînent : plusieurs réutilisent le résultat des précédents. Travaillez sur le projet du chapitre, pas sur une copie.

### Exercice 1 — Une constante de plus (facile)

La durée de la pause artificielle de `demarrerPartie()` est écrite en dur : `Duration(milliseconds: 350)`. C'est un nombre magique.

Ajoutez à `Constantes` une valeur `dureeTransition` exprimée en millisecondes, et utilisez-la dans `demarrerPartie()`. Réglez-la à 350, puis essayez 1200 pour vérifier que l'écran de chargement s'affiche bien plus longtemps.

### Exercice 2 — Le meilleur score au menu (facile)

Sous les boutons du menu, affichez `Meilleur score : 0` en utilisant la valeur `jeu.meilleurScore`.

Le texte doit utiliser `Palette.texteFaible` et n'apparaître **que** si `jeu.meilleurScore > 0`. Comme la valeur vaut toujours 0 au chapitre 35, vérifiez votre travail en modifiant temporairement l'initialisation du champ dans `DonjonGame`.

### Exercice 3 — La table des transitions (facile)

Dans `core/game_state.dart`, écrivez une fonction de niveau supérieur :

```dart
bool estTransitionAutorisee(GameState depuis, GameState vers)
```

Elle renvoie `true` si la transition figure dans le tableau de la section 35.9, `false` sinon. Utilisez une `Map<GameState, Set<GameState>>` constante.

Vérifiez : `estTransitionAutorisee(GameState.menu, GameState.pause)` doit valoir `false`.

### Exercice 4 — Un bouton Crédits (facile)

Ajoutez un cinquième bouton `CRÉDITS` au menu, entre `OPTIONS` et `QUITTER`. Il ouvre une `AlertDialog` contenant le titre du jeu, votre nom, la version de Flame utilisée et l'année.

Réutilisez `BoutonMenu` sans le modifier.

### Exercice 5 — Compteur de parties (moyen)

Ajoutez à `DonjonGame` un champ `int partiesJouees = 0;`, incrémenté à chaque appel de `demarrerPartie()`.

Affichez sa valeur dans le message de la touche F2, et dans le `HudProvisoire`.

### Exercice 6 — Recommencer depuis le HUD (moyen)

Ajoutez un bouton `RECOMMENCER` dans `HudProvisoire`, à gauche du bouton `MENU`. Il relance une partie depuis le début, sans repasser par le menu.

Attention : `demarrerPartie()` commence par `changerEtat(GameState.chargement)`. Vérifiez que la transition depuis `enJeu` fonctionne, et que le monde ne contient pas deux salles superposées après avoir recommencé trois fois (touche F2).

### Exercice 7 — Un garde-fou sur les transitions (moyen)

Utilisez la fonction de l'exercice 3 dans `changerEtat()`. Si la transition n'est pas autorisée, écrivez un message d'alerte dans la console avec `debugPrint` et **ne changez rien**.

Testez en appelant volontairement `changerEtat(GameState.pause)` depuis le menu : le jeu doit rester au menu et vous prévenir.

### Exercice 8 — Le titre qui apparaît (moyen)

Faites apparaître le titre du jeu en fondu sur 900 millisecondes à chaque affichage du menu.

Indication : transformez `TitreDuJeu` en `StatefulWidget` et utilisez `AnimatedOpacity`, ou bien restez en `StatelessWidget` et utilisez `TweenAnimationBuilder<double>`.

Vérifiez que l'animation se rejoue quand vous revenez au menu depuis une partie.

### Exercice 9 — L'écran de pause (difficile)

Rendez l'état `GameState.pause` réellement atteignable, en avance sur le chapitre 40.

1. Écrivez un widget `EcranPause` dans `ecrans/menu_principal.dart` : un fond semi-transparent, le mot PAUSE, un bouton `REPRENDRE` et un bouton `MENU`.
2. Enregistrez-le dans `overlayBuilderMap` sous `Overlays.pause`.
3. Dans `onKeyEvent` de `DonjonGame`, faites basculer la touche P entre `enJeu` et `pause`.

Vérifiez le point essentiel du chapitre : pendant la pause, l'image du jeu est **figée** mais les boutons de l'overlay **répondent**.

### Exercice 10 — Tester la machine à états (difficile)

Créez `test/machine_etats_test.dart` et écrivez des tests `flutter_test` qui vérifient, sans lancer le jeu :

1. que `GameState.enJeu.moteurActif` vaut `true` et que celui de `pause` vaut `false` ;
2. que `DonjonGame.overlayDeLEtat` renvoie un nom différent pour chacun des six états ;
3. que la fonction `estTransitionAutorisee` de l'exercice 3 accepte `menu → chargement` et refuse `menu → gameOver`.

Lancez `flutter test`.

---

## 35.34 — Corrections des exercices

### Correction 1

```dart
// lib/config/constantes.dart — ajout dans la classe Constantes
  /// Durée de l'écran de chargement entre deux écrans, en millisecondes.
  /// Purement esthétique : évite un clignotement.
  static const int dureeTransition = 350;
```

```dart
// lib/donjon_game.dart — dans demarrerPartie()
    await Future<void>.delayed(
      const Duration(milliseconds: Constantes.dureeTransition),
    );
```

**Explication :** `Duration` accepte une valeur constante, donc `const Duration(milliseconds: Constantes.dureeTransition)` reste une expression constante et ne coûte rien à l'exécution. La valeur est un `int` parce que le constructeur de `Duration` attend des entiers ; c'est le seul cas du projet où l'on ne stocke pas une durée en secondes. Passez la constante à 1200 et relancez : l'indicateur de chargement reste visible plus d'une seconde, ce qui prouve que l'ordre des opérations de `demarrerPartie()` est bien celui annoncé en 35.24.

### Correction 2

```dart
// lib/ecrans/menu_principal.dart — après le Container des boutons
                const SizedBox(height: 20),
                if (jeu.meilleurScore > 0)
                  Text(
                    'Meilleur score : ${jeu.meilleurScore}',
                    style: const TextStyle(
                      color: Palette.texteFaible,
                      fontSize: 14,
                      letterSpacing: 2,
                    ),
                  ),
                const SizedBox(height: 8),
```

**Explication :** `if` dans une liste de widgets est une *collection if* (chapitre 06). Le widget n'est pas construit du tout quand la condition est fausse — c'est mieux qu'un `Visibility` ou qu'un `Opacity`, qui construisent puis masquent. Pour vérifier, remplacez temporairement `int meilleurScore = 0;` par `int meilleurScore = 1250;` dans `DonjonGame` : la ligne apparaît. Le chapitre 40 remplira ce champ depuis le stockage local, et cette ligne fonctionnera sans être touchée.

### Correction 3

```dart
// lib/core/game_state.dart — après l'extension

/// Transitions autorisées, telles que définies au chapitre 35, section 35.9.
const Map<GameState, Set<GameState>> transitionsAutorisees =
    <GameState, Set<GameState>>{
  GameState.chargement: <GameState>{GameState.menu, GameState.enJeu},
  GameState.menu: <GameState>{GameState.chargement},
  GameState.enJeu: <GameState>{
    GameState.menu,
    GameState.pause,
    GameState.gameOver,
    GameState.victoire,
    GameState.chargement,
  },
  GameState.pause: <GameState>{GameState.enJeu, GameState.menu},
  GameState.gameOver: <GameState>{GameState.menu, GameState.chargement},
  GameState.victoire: <GameState>{GameState.menu, GameState.chargement},
};

/// La transition `depuis -> vers` figure-t-elle dans la table ?
bool estTransitionAutorisee(GameState depuis, GameState vers) {
  return transitionsAutorisees[depuis]?.contains(vers) ?? false;
}
```

**Explication :** la table est `const`, donc construite à la compilation et partagée par tout le programme. L'opérateur `?.` suivi de `?? false` (chapitre 12) traite proprement le cas d'un état absent de la table : plutôt que de planter, la fonction refuse la transition, ce qui est le comportement prudent. Notez que `enJeu → chargement` est autorisé : c'est le chemin utilisé par le bouton « Recommencer » de l'exercice 6. Une table de transitions vaut mieux qu'une cascade de `if` : elle se lit comme le tableau de la section 35.9, et on peut la comparer visuellement à la documentation.

### Correction 4

```dart
// lib/ecrans/menu_principal.dart — dans la colonne des boutons
                      const SizedBox(height: 12),
                      BoutonMenu(
                        libelle: 'CRÉDITS',
                        icone: Icons.info_outline,
                        onPressed: () {
                          showDialog<void>(
                            context: context,
                            builder: (BuildContext context) => AlertDialog(
                              backgroundColor: Palette.panneau,
                              title: const Text('DONJON DE DART'),
                              content: const Text(
                                'Un jeu de plateformes 2D.\n\n'
                                'Développement : votre nom\n'
                                'Moteur : Flame 1.38.0 sur Flutter\n'
                                'Formation : PARTIE 2C, chapitres 35 à 40\n'
                                'Année : 2026',
                              ),
                              actions: <Widget>[
                                TextButton(
                                  onPressed: () => Navigator.of(context).pop(),
                                  child: const Text('FERMER'),
                                ),
                              ],
                            ),
                          );
                        },
                      ),
```

**Explication :** aucun code nouveau n'est nécessaire, ce qui est précisément le but de l'exercice : `BoutonMenu` avait été écrit pour être réutilisable, et il l'est. Le bouton n'est pas `principal`, il prend donc l'apparence sobre. Remarquez la concaténation implicite des chaînes littérales adjacentes : `'ligne 1\n' 'ligne 2'` est un seul littéral, ce qui permet de respecter la limite de 80 colonnes sans utiliser d'opérateur `+`.

### Correction 5

```dart
// lib/donjon_game.dart — nouveau champ
  /// Nombre de parties lancées depuis le démarrage de l'application.
  /// Remis à zéro seulement à la fermeture. Utile en développement.
  int partiesJouees = 0;
```

```dart
// lib/donjon_game.dart — dans demarrerPartie(), après la remise à zéro
    partiesJouees++;
```

```dart
// lib/donjon_game.dart — dans le message de la touche F2
        debugPrint(
          '[DonjonGame] etat=${etat.libelle} score=$score vies=$vies '
          'niveau=$niveauCourant parties=$partiesJouees '
          'composants=${monde.children.length} '
          'overlays=${overlays.activeOverlays}',
        );
```

```dart
// lib/main.dart — dans HudProvisoire, dans le premier Text
                'Score ${jeu.score}   Vies ${jeu.vies}   '
                'Niveau ${jeu.niveauCourant + 1}/${Constantes.nombreNiveaux}'
                '   Partie ${jeu.partiesJouees}',
```

**Explication :** `partiesJouees` n'est **pas** remis à zéro par `demarrerPartie()`, contrairement à `score`, `vies` et `niveauCourant` : c'est un compteur de session, pas un compteur de partie. Cette distinction est la même que pour `meilleurScore`. Le champ s'incrémente après la remise à zéro pour que sa valeur ne soit jamais écrasée. Le `HudProvisoire` étant un widget construit à chaque activation de l'overlay, il lit la valeur au bon moment ; il ne se mettrait pas à jour en cours de partie, mais `partiesJouees` ne change jamais pendant une partie.

### Correction 6

```dart
// lib/main.dart — dans HudProvisoire, avant le bouton MENU
              OutlinedButton(
                onPressed: () async {
                  await jeu.demarrerPartie();
                },
                style: OutlinedButton.styleFrom(
                  foregroundColor: Palette.texte,
                  side: const BorderSide(color: Palette.mur, width: 2),
                ),
                child: const Text('RECOMMENCER'),
              ),
              const SizedBox(width: 8),
```

**Explication :** aucune modification de `DonjonGame` n'est nécessaire, et c'est le signe que `demarrerPartie()` a été correctement écrite : elle est **idempotente**, c'est-à-dire qu'on peut l'appeler depuis n'importe quel état sans laisser de résidu. Elle vide le monde avant de le remplir (étape 3), remet les compteurs à zéro (étape 2) et passe par `chargement` (étape 1), ce qui retire l'overlay du HUD et évite que le joueur ne clique deux fois. Vérification par la touche F2 : `composants=8` après une partie comme après dix.

### Correction 7

```dart
// lib/donjon_game.dart — au début de changerEtat()
  void changerEtat(GameState nouvelEtat) {
    if (nouvelEtat == etat) {
      return;
    }

    // Garde-fou : la transition doit figurer dans la table du chapitre 35.
    if (!estTransitionAutorisee(etat, nouvelEtat)) {
      debugPrint(
        '[DonjonGame] TRANSITION REFUSÉE : '
        '${etat.libelle} -> ${nouvelEtat.libelle}',
      );
      return;
    }

    final GameState ancienEtat = etat;
    // ... la suite est inchangée
```

**Résultat en appelant `changerEtat(GameState.pause)` depuis le menu :**

```text
[DonjonGame] TRANSITION REFUSÉE : Menu principal -> Pause
```

**Explication :** ce garde-fou transforme une erreur de logique en message immédiat, à l'endroit exact où elle se produit. Sans lui, la même erreur se manifesterait bien plus loin, sous la forme d'un plantage du `GameWidget` cherchant un overlay `'pause'` absent de la map. Notez que la fonction **ne lève pas d'exception** : dans un jeu, faire planter le programme pour une transition invalide serait disproportionné. On refuse, on trace, on continue. En développement, vous pouvez remplacer le `debugPrint` par un `assert(false, '...')`, qui plante en debug et disparaît en production.

### Correction 8

```dart
// lib/ecrans/menu_principal.dart — TitreDuJeu revu
class TitreDuJeu extends StatelessWidget {
  const TitreDuJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder<double>(
      tween: Tween<double>(begin: 0, end: 1),
      duration: const Duration(milliseconds: 900),
      curve: Curves.easeOut,
      builder: (BuildContext context, double valeur, Widget? enfant) {
        return Opacity(
          opacity: valeur,
          child: Transform.translate(
            offset: Offset(0, (1 - valeur) * -24),
            child: enfant,
          ),
        );
      },
      // La colonne du titre, inchangée, passée en `child` :
      child: Column(children: <Widget>[/* ... voir 35.32 ... */]),
    );
  }
}
```

**Explication :** `TweenAnimationBuilder` anime une valeur de 0 à 1 dès que le widget est construit, sans qu'on ait à gérer un `AnimationController` ni un `StatefulWidget` (chapitre 19). Le paramètre `child` est l'optimisation clé : la colonne est construite **une seule fois** et passée telle quelle au `builder` à chaque image ; seuls `Opacity` et `Transform.translate` sont reconstruits. L'animation se rejoue à chaque retour au menu, car `changerEtat()` retire l'overlay puis le rajoute, ce qui détruit et reconstruit tout le sous-arbre. Le léger déplacement vertical (`-24` pixels au début) rend l'apparition plus vivante qu'un simple fondu.

### Correction 9

```dart
// lib/ecrans/menu_principal.dart — nouveau widget
/// Écran de pause (avancé du chapitre 40).
///
/// Il DOIT être un widget Flutter : `pauseEngine()` fige tout ce que Flame
/// dessine, donc un composant Flame ne répondrait plus aux clics.
class EcranPause extends StatelessWidget {
  const EcranPause({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    return Material(
      // Semi-transparent : on voit le jeu figé derrière.
      color: Palette.fondMenu.withValues(alpha: 0.75),
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const Text('PAUSE', style: Palette.titre),
            const SizedBox(height: 32),
            BoutonMenu(
              libelle: 'REPRENDRE',
              icone: Icons.play_arrow,
              principal: true,
              onPressed: () => jeu.changerEtat(GameState.enJeu),
            ),
            const SizedBox(height: 12),
            BoutonMenu(
              libelle: 'MENU',
              icone: Icons.home,
              onPressed: () => jeu.retournerAuMenu(),
            ),
          ],
        ),
      ),
    );
  }
}
```

Il faut alors importer `../core/game_state.dart` dans ce fichier, enregistrer l'overlay :

```dart
// lib/main.dart — dans overlayBuilderMap
          Overlays.pause: (BuildContext context, DonjonGame game) {
            return EcranPause(jeu: game);
          },
```

et gérer la touche :

```dart
// lib/donjon_game.dart — dans onKeyEvent, à l'intérieur du bloc KeyDownEvent
      if (event.logicalKey == LogicalKeyboardKey.keyP) {
        if (etat == GameState.enJeu) {
          changerEtat(GameState.pause);
          return KeyEventResult.handled;
        }
        if (etat == GameState.pause) {
          changerEtat(GameState.enJeu);
          return KeyEventResult.handled;
        }
      }
```

**Explication :** les trois pièces sont indissociables, et c'est la leçon de l'exercice : un état de jeu n'existe vraiment que si la transition **et** l'overlay **et** le déclencheur sont en place. Si vous oubliez l'entrée dans `overlayBuilderMap`, l'application plante à la première pression sur P — c'est l'erreur décrite en section 35.16. Vérifiez le point essentiel : pendant la pause, le décor est immobile (le moteur est figé par `changerEtat`) alors que les boutons répondent instantanément, parce qu'ils appartiennent à l'arbre de widgets. Au chapitre 40, ce widget déménagera dans `ecrans/ecran_pause.dart` et gagnera les réglages de volume.

### Correction 10

```dart
// test/machine_etats_test.dart
import 'package:donjon_de_dart/config/constantes.dart';
import 'package:donjon_de_dart/core/game_state.dart';
import 'package:donjon_de_dart/donjon_game.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('GameState', () {
    test('le moteur ne tourne que dans enJeu', () {
      expect(GameState.enJeu.moteurActif, isTrue);
      expect(GameState.pause.moteurActif, isFalse);
      expect(GameState.menu.moteurActif, isFalse);
      expect(GameState.gameOver.moteurActif, isFalse);
    });

    test('partieEnCours couvre enJeu et pause', () {
      expect(GameState.enJeu.partieEnCours, isTrue);
      expect(GameState.pause.partieEnCours, isTrue);
      expect(GameState.menu.partieEnCours, isFalse);
    });

  });
  group('overlayDeLEtat', () {
    test('chaque état a un overlay distinct', () {
      final Set<String> noms = GameState.values
          .map(DonjonGame.overlayDeLEtat)
          .toSet();
      expect(noms.length, equals(GameState.values.length));
    });

    test('les noms viennent bien de la classe Overlays', () {
      expect(DonjonGame.overlayDeLEtat(GameState.menu),
          equals(Overlays.menuPrincipal));
      expect(DonjonGame.overlayDeLEtat(GameState.enJeu), equals(Overlays.hud));
    });
  });

  group('estTransitionAutorisee', () {
    test('menu -> chargement est autorisé', () {
      expect(
        estTransitionAutorisee(GameState.menu, GameState.chargement),
        isTrue,
      );
    });

    test('menu -> gameOver est refusé', () {
      expect(
        estTransitionAutorisee(GameState.menu, GameState.gameOver),
        isFalse,
      );
    });

    test('aucun état ne peut aller vers lui-même', () {
      for (final GameState etat in GameState.values) {
        expect(estTransitionAutorisee(etat, etat), isFalse);
      }
    });
  });
}
```

**Résultat :**

```text
00:01 +7: All tests passed!
```

**Explication :** ces tests ne construisent **aucune** instance de `DonjonGame` et ne lancent aucun rendu : ils portent uniquement sur des fonctions pures et des données constantes. C'est exactement ce que le chapitre 26 (section 26.31) recommandait de tester en priorité dans un jeu — la logique d'état, pas l'affichage. C'est aussi la raison pour laquelle `overlayDeLEtat` a été déclarée `static` : une méthode statique se teste sans instance. Tester `changerEtat()` demanderait un jeu monté dans un widget, donc `flutter_test` avec un `GameWidget` ou le paquet `flame_test` ; c'est possible, plus lent, et hors du champ de ce chapitre. Le test « aucun état ne peut aller vers lui-même » documente le garde-fou de la première ligne de `changerEtat()`.

---

## Et maintenant ?

Votre projet se lance, affiche un menu, entre dans une salle, en ressort. Sept fichiers, une architecture complète, et une seule chose qui manque : **quelqu'un à qui appuyer sur les touches**.

Le rectangle bleu posé dans la salle de démonstration n'est qu'un décor. Au chapitre 36, il devient le héros du jeu.

Vous y écrirez `core/entite.dart`, la classe de base de tout ce qui vit dans le donjon, puis `composants/joueur.dart` : un `PositionComponent` qui lit le clavier, accumule une vélocité, subit la gravité de `Constantes.gravite`, saute avec `Constantes.forceSaut`, s'arrête sur les plateformes grâce aux hitbox du chapitre 32, et change d'apparence selon son état — immobile, marche, saut, chute. Vous y écrirez aussi `composants/plateforme.dart`, et la caméra se mettra enfin à suivre quelqu'un.

Tout ce que vous avez posé aujourd'hui va servir dès la première section : `Constantes` pour les chiffres, `Palette` pour les couleurs, `monde` pour l'accueillir, `HasKeyboardHandlerComponents` pour lui transmettre les touches, `HasCollisionDetection` pour l'empêcher de traverser le sol, et `demarrerPartie()` pour le faire naître.

Rendez-vous au chapitre 36 : [36-PARTIE-2C—LE-JOUEUR-MOUVEMENT-ET-ANIMATIONS.md](./36-PARTIE-2C—LE-JOUEUR-MOUVEMENT-ET-ANIMATIONS.md)
