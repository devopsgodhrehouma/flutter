# PARTIE 2C — LE JEU COMPLET « DONJON DE DART »
# CHAPITRE 40 — SONS, PAUSE, GAME OVER ET SAUVEGARDE

> **Versions utilisées dans ce chapitre :** `flame` **1.38.0**, `flame_audio` **2.12.2**,
> `shared_preferences` **2.5.5** (publiée le 25 mars 2026).
> **Date de vérification des API :** 8 août 2026, sur `https://docs.flame-engine.org/latest/`,
> `https://pub.dev/packages/flame`, `https://pub.dev/packages/flame_audio`,
> `https://pub.dev/packages/shared_preferences` et le dépôt `flame-engine/flame` (branche `main`).
> **Contraintes SDK :** `sdk: ">=3.12.0 <4.0.0"`, `flutter: ">=3.44.0"`.
>
> **Niveau :** intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitres 35 à 39 (le projet « Donjon de Dart » complet), chapitre 34 (`flame_audio`, `FlameAudio.bgm`, service audio centralisé). Côté Dart : chapitre 13 (exceptions), chapitre 14 (`where`, `map`, `fold`), chapitre 15 (asynchrone), chapitre 17 (JSON).
> **Ce que vous saurez faire à la fin :** doter le jeu d'un service audio qui fonctionne même sans un seul fichier son, d'un menu pause, d'un écran Game Over, d'un écran de victoire avec statistiques, et d'une sauvegarde persistante du meilleur score, de la progression et des réglages — c'est-à-dire livrer un jeu **fini**.

---

## 40.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- énoncer précisément ce qui sépare un jeu « jouable » d'un jeu « fini » ;
- écrire un service audio en singleton, isolé du reste du code ;
- masquer complètement `flame_audio` derrière un enum d'effets logiques ;
- précharger les sons au démarrage et mesurer ce que le préchargement évite ;
- brancher un effet sonore sur chaque événement de jeu, au bon endroit ;
- lancer une musique de fond différente selon le niveau ;
- séparer le volume de la musique du volume des effets, et les appliquer à chaud ;
- couper le son proprement, sans laisser de lecteur audio orphelin ;
- faire tourner le jeu **sans aucun fichier audio**, en mode silencieux, avec un retour visuel ;
- distinguer `pauseEngine()` d'une pause logique et savoir laquelle survit à la pause ;
- écrire un écran de pause en Flutter avec Reprendre, Options et Quitter au menu ;
- mettre le jeu en pause automatiquement quand l'application perd le focus ;
- déclencher la pause avec la touche Échap **et** avec un bouton du HUD ;
- écrire un écran Game Over qui affiche le score et le meilleur score ;
- proposer Rejouer le niveau ou revenir au menu, sans fuite mémoire ;
- écrire un écran de victoire avec des statistiques de fin de partie ;
- calculer ces statistiques à partir d'un journal d'événements, avec `where`, `map` et `fold` ;
- justifier le choix de `shared_preferences` face aux autres solutions de persistance ;
- ajouter la dépendance et écrire un service de sauvegarde ;
- sauvegarder et relire un meilleur score, une progression et des réglages ;
- sérialiser une sauvegarde complète en JSON et la versionner ;
- charger la sauvegarde au démarrage et activer le bouton « Continuer » du menu ;
- détecter une sauvegarde corrompue et repartir sans planter ;
- écrire un écran d'options qui persiste réellement ses réglages ;
- dérouler une liste de vérification de polish avant de considérer un jeu comme fini ;
- lire l'arborescence finale du projet et son `pubspec.yaml` définitif.

---

## 40.1 — Où on en est et ce qui manque pour que le jeu soit fini

### Ce que le chapitre 39 vous a laissé entre les mains

À la fin du chapitre 39, « Donjon de Dart » est un vrai jeu de plateformes. Faisons l'inventaire, honnêtement, fichier par fichier.

| Ce qui existe | Chapitre | État |
| --- | --- | --- |
| `DonjonGame`, `GameState`, `changerEtat()`, overlays, menu principal | 35 | complet |
| `Joueur` : gravité, saut, machine à états, attaque | 36 | complet |
| `Ennemi`, `Gobelin`, `Chauvesouris`, dégâts, vies, invincibilité | 37 | complet |
| `Piece`, `Potion`, `Cle`, score, combo, `Hud`, barres de vie et d'énergie | 38 | complet |
| `Niveau`, `niveaux_data.dart`, `Porte`, `chargerNiveau()`, `terminerNiveau()`, `Boss`, difficulté | 39 | complet |

Concrètement : vous lancez `flutter run`, vous cliquez sur JOUER, vous traversez trois salles, vous ramassez une clé, vous ouvrez une porte, vous affrontez un boss, et vous gagnez.

Et pourtant, personne ne dirait que ce jeu est fini. Voici pourquoi.

### Les six manques

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  CE QUI MANQUE À LA FIN DU CHAPITRE 39                           │
  ├──────────────────────────────────────────────────────────────────┤
  │  1. Le jeu est MUET.                                             │
  │     Aucun retour sonore : ni saut, ni coup, ni pièce.            │
  │                                                                  │
  │  2. Le jeu ne peut PAS ÊTRE INTERROMPU.                          │
  │     Le téléphone sonne, le héros meurt.                          │
  │                                                                  │
  │  3. La MORT est un cul-de-sac.                                   │
  │     `changerEtat(GameState.gameOver)` gèle l'écran. Rien de plus.│
  │                                                                  │
  │  4. La VICTOIRE est invisible.                                   │
  │     `GameState.victoire` existe, aucun écran ne l'affiche.       │
  │                                                                  │
  │  5. RIEN NE SURVIT à la fermeture.                               │
  │     `meilleurScore` repart à 0 à chaque lancement.               │
  │                                                                  │
  │  6. Les OPTIONS sont un décor.                                   │
  │     Les curseurs du chapitre 35 ne pilotent rien.                │
  └──────────────────────────────────────────────────────────────────┘
```

Ces six points ont un dénominateur commun : ils ne concernent pas le *gameplay*. Aucun ne change la façon dont le héros saute. Ils concernent ce qu'on appelle en anglais le *game feel* et la *persistance* — le sentiment que le jeu vous répond, et le sentiment qu'il se souvient de vous.

C'est précisément ce qui sépare un projet d'école d'un jeu qu'on a envie de relancer.

### Ce que ce chapitre ajoute

| Fichier | État | Rôle |
| --- | --- | --- |
| `lib/services/audio_service.dart` | **nouveau** | `AudioService` : effets, musique, volumes, mode silencieux |
| `lib/services/sauvegarde_service.dart` | **nouveau** | `SauvegardeService` : meilleur score, progression, réglages, JSON |
| `lib/core/statistiques.dart` | **nouveau** | journal d'événements et `StatistiquesPartie` |
| `lib/ecrans/ecran_pause.dart` | **nouveau** | `EcranPause` |
| `lib/ecrans/ecran_game_over.dart` | **nouveau** | `EcranGameOver` |
| `lib/ecrans/ecran_victoire.dart` | **nouveau** | `EcranVictoire` |
| `lib/ecrans/ecran_options.dart` | **nouveau** | `EcranOptions`, partagé par le menu et la pause |
| `lib/donjon_game.dart` | **modifié** | pause, cycle de fin de partie, journal, hooks audio |
| `lib/ecrans/menu_principal.dart` | **modifié** | bouton « Continuer » réellement actif |
| `lib/hud/hud.dart` | **modifié** | bouton Pause |
| `lib/main.dart` | **modifié** | les quatre nouveaux overlays |
| `pubspec.yaml` | **modifié** | `flame_audio` et `shared_preferences` |

### La règle qui n'a pas changé

Elle tient en une phrase, et c'est la dernière fois que je l'écris : **le jeu doit tourner sans aucun fichier image ni audio**.

Pour l'audio, c'est plus subtil qu'ailleurs. On ne peut pas dessiner un son avec un `RectangleComponent`. La solution s'appelle la **dégradation gracieuse**, et vous l'avez déjà rencontrée à la section 34.15 : le service détecte l'absence de fichiers, bascule en mode silencieux, et le reste du code ne s'en aperçoit jamais. La section 40.8 y est entièrement consacrée.

> **À retenir.** Un jeu fini n'a pas plus de mécaniques qu'un jeu jouable. Il a du son, une pause, une fin, et une mémoire.

---

## 40.2 — `lib/services/audio_service.dart` : un service centralisé

### Le problème que le service résout

Imaginez la version naïve. Dans `joueur.dart`, au moment du saut :

```dart
// À NE PAS FAIRE
void sauter() {
  velocite.y = Constantes.forceSaut;
  FlameAudio.play('saut.mp3', volume: 0.8);
}
```

Ça marche. Puis viennent les questions réelles.

1. Le joueur veut baisser le volume. Où est le curseur ? Il faudrait qu'il modifie ce `0.8` dans quinze fichiers.
2. Le fichier `saut.mp3` n'existe pas encore. `FlameAudio.play` lève une exception, en pleine boucle de jeu, soixante fois par seconde.
3. Vous voulez tester `sauter()` dans un test unitaire. Le test essaie de charger un fichier audio et échoue.
4. Vous décidez de passer de `flame_audio` à autre chose. Vous ouvrez quinze fichiers.

Un service règle les quatre d'un coup. Le chapitre 34 en a montré le principe à la section 34.13 ; nous en écrivons ici la version définitive, celle qui part avec le jeu.

### Les cinq règles du service

1. **Le reste du jeu n'importe jamais `flame_audio`.** Un seul fichier du projet contient cet `import`.
2. **Le reste du jeu ne connaît jamais un nom de fichier.** Il connaît `Effet.saut`.
3. **Aucun appel audio ne peut faire planter le jeu.** Tout est gardé.
4. **Le service est un singleton.** Une seule instance, accessible de partout, y compris depuis un widget Flutter qui n'a pas de référence au jeu.
5. **Le service fonctionne sans fichier.** C'est son mode par défaut au premier lancement.

### Pourquoi un singleton, et pas un champ de `DonjonGame`

Au chapitre 34, le service était un champ : `final ServiceAudio audio = ServiceAudio();`. C'était parfait pour une démonstration d'un seul écran.

Ici, ça ne suffit plus. L'écran d'options est un widget Flutter ouvert depuis le menu principal. Le menu principal, lui, reçoit bien `jeu` par `overlayBuilderMap`… mais la boîte de dialogue des options est poussée par `showDialog`, dans un autre sous-arbre de widgets. Faire descendre la référence au jeu jusque-là, puis jusqu'au `Slider`, c'est trois niveaux de paramètres pour rien.

Le son est une ressource **globale à l'application** : il n'y a qu'une carte son. Le singleton est ici le bon outil, pas un raccourci paresseux.

```dart
class AudioService {
  /// Constructeur privé : personne ne peut faire `AudioService()`.
  AudioService._();

  /// L'unique instance.
  static final AudioService instance = AudioService._();
}
```

Le motif est exactement celui de `Constantes._()` et `Palette._()` du chapitre 35, avec une instance en plus.

### L'enum des effets

```dart
/// Identifiants logiques des effets sonores.
///
/// Le reste du jeu ne connaît QUE cet enum, jamais un nom de fichier.
enum Effet {
  saut,
  coup,       // le joueur frappe
  degat,      // le joueur encaisse
  piece,
  potion,
  cle,
  porte,
  mort,
  boss,       // apparition du boss
  victoire,
  clic,       // interface
}
```

Onze valeurs, une par événement audible du jeu. Les enums du chapitre 11 vous servent ici de contrat de compilation : si vous écrivez `Effet.saaut`, le compilateur refuse. Si vous écriviez `'saaut.mp3'`, personne ne dirait rien avant l'exécution.

### La table de correspondance

```dart
  static const Map<Effet, String> _fichiers = <Effet, String>{
    Effet.saut: 'saut.wav',
    Effet.coup: 'coup.wav',
    Effet.degat: 'degat.wav',
    Effet.piece: 'piece.wav',
    Effet.potion: 'potion.wav',
    Effet.cle: 'cle.wav',
    Effet.porte: 'porte.wav',
    Effet.mort: 'mort.wav',
    Effet.boss: 'boss.wav',
    Effet.victoire: 'victoire.wav',
    Effet.clic: 'clic.wav',
  };
```

Les chemins sont **relatifs à `assets/audio/`** : Flame ajoute le préfixe lui-même (fiche de référence, section 10.1). Écrire `'assets/audio/saut.wav'` produirait `assets/audio/assets/audio/saut.wav`. C'est l'erreur numéro un du chapitre 34, et elle reste l'erreur numéro un ici.

Le format `.wav` est choisi pour les effets courts : pas de décompression, donc pas de latence. Les musiques resteront en `.mp3` ou `.ogg`, où la taille compte davantage.

### L'équilibrage

Deux sons enregistrés par deux personnes différentes n'ont jamais le même niveau. Plutôt que de rééditer les fichiers, on corrige en code :

```dart
  /// Volume relatif de chaque effet, pour compenser des enregistrements
  /// de niveaux différents. Multiplié par `volumeEffets`.
  static const Map<Effet, double> _equilibrage = <Effet, double>{
    Effet.saut: 0.45,
    Effet.coup: 0.90,
    Effet.degat: 0.85,
    Effet.piece: 0.55,
    Effet.potion: 0.60,
    Effet.cle: 0.70,
    Effet.porte: 0.75,
    Effet.mort: 1.00,
    Effet.boss: 1.00,
    Effet.victoire: 0.90,
    Effet.clic: 0.35,
  };
```

Notez les valeurs basses pour `saut`, `piece` et `clic` : ce sont les sons les plus fréquents. Un son qu'on entend deux cents fois par partie doit être discret, sinon il devient une agression. C'est une règle de conception sonore, pas un détail technique.

### Le squelette complet du service

```dart
// lib/services/audio_service.dart
import 'package:flame_audio/flame_audio.dart';
import 'package:flutter/foundation.dart';

enum Effet { saut, coup, degat, piece, potion, cle, porte, mort, boss, victoire, clic }

class AudioService {
  AudioService._();
  static final AudioService instance = AudioService._();

  /// Vrai quand au moins un fichier audio a pu être chargé.
  /// Faux au démarrage, et faux définitivement si le préchargement échoue.
  bool disponible = false;

  /// Interrupteurs indépendants, pilotés par l'écran d'options.
  bool musiqueActivee = true;
  bool effetsActives = true;

  /// Journal des effets demandés. Sert aux tests et au mode silencieux.
  final List<Effet> journal = <Effet>[];

  /// Appelé à CHAQUE demande d'effet, que le son sorte ou non.
  /// Le jeu y branche son retour visuel (section 40.8).
  void Function(Effet effet)? surEffet;

  // ... volumes, préchargement, lecture : sections 40.3 à 40.7
}
```

Le champ `surEffet` mérite un mot. C'est un *point d'extension* : le service ne sait pas dessiner, et il ne doit pas le savoir. Il se contente de prévenir. Le jeu décide s'il fait clignoter l'écran. C'est l'inversion de dépendance du chapitre 26, en une ligne.

> **À retenir.** Un service audio réussi se reconnaît à ce qu'il rend invisible : le paquet, les noms de fichiers, les volumes, et les erreurs.

---

## 40.3 — Précharger les sons au démarrage (rappel chapitre 34)

### Ce que le préchargement évite

La section 34.8 l'a mesuré : le **premier** `FlameAudio.play('coup.wav')` d'une session doit lire le fichier depuis les assets, le décoder, et préparer un lecteur. Sur mobile, cela prend entre 50 et 300 millisecondes. Pendant ce temps, la boucle de jeu attend.

Le résultat est une saccade au moment exact où le joueur frappe pour la première fois — c'est-à-dire au moment où il juge la réactivité de votre jeu.

Le préchargement déplace ce coût dans l'écran de chargement, là où personne ne le voit.

```text
  SANS préchargement                    AVEC préchargement
  ------------------                    ------------------
  Menu       : 0 ms                     Menu       : 0 ms
  Chargement : 0 ms                     Chargement : 400 ms  (invisible)
  1er coup   : 180 ms de saccade        1er coup   : 0 ms
  2e coup    : 0 ms                     2e coup    : 0 ms
```

### `FlameAudio.audioCache.loadAll`

L'API est confirmée par la documentation de `flame_audio` 2.12.2 et par le dartdoc d'`audioplayers` :

```dart
Future<List<Uri>> loadAll(List<String> fileNames);
Future<Uri>       load(String fileName);
Future<void>      clear(String fileName);
Future<void>      clearAll();
```

> **Attention.** Certains tutoriels écrivent `FlameAudio.audioCache.clearCache()`. Cette méthode n'existe pas dans la version courante : c'est **`clearAll()`**.

### La méthode `precharger()`

```dart
  /// Charge tous les effets en mémoire.
  ///
  /// Ne lève JAMAIS : en cas d'échec, le jeu bascule en mode silencieux.
  Future<void> precharger() async {
    try {
      await FlameAudio.audioCache.loadAll(_fichiers.values.toList());
      disponible = true;
    } catch (erreur) {
      // Cas normal tant que le dossier assets/audio/ est vide.
      disponible = false;
      debugPrint(
        '[AudioService] Aucun fichier audio trouvé. '
        'Mode silencieux activé. ($erreur)',
      );
    }
  }
```

Trois points sur ce `try` / `catch`, qui reprend le chapitre 13.

**On attrape tout.** `catch (erreur)` sans type, ce qui est normalement déconseillé. Ici c'est volontaire et justifié : l'absence d'un asset ne lève pas le même type d'exception sur Android, sur le Web et sur le bureau. On ne cherche pas à diagnostiquer, on cherche à **survivre**.

**On ne relance pas.** Pas de `rethrow`. C'est la différence entre une erreur de programmation (qu'il faut faire remonter) et une condition d'environnement (qu'il faut absorber). Un jeu qui refuse de démarrer parce qu'il n'a pas de bruit de saut est un jeu mal écrit.

**On trace.** `debugPrint` écrit dans la console en mode debug et est ignoré en production. L'élève voit le message, le joueur non.

### L'initialisation complète

```dart
  /// Point d'entrée unique. À appeler UNE FOIS, depuis `DonjonGame.onLoad()`.
  Future<void> initialiser() async {
    // Enregistre l'observateur de cycle de vie de `Bgm`.
    // Doit être appelé quand un WidgetsBinding existe : onLoad convient.
    FlameAudio.bgm.initialize();

    await precharger();
  }
```

`FlameAudio.bgm.initialize()` a une signature réelle `Future<void> initialize({AudioContext? audioContext})`. Son rôle : abonner l'objet `Bgm` au cycle de vie de l'application, pour qu'il mette la musique en pause tout seul quand vous quittez l'application. Sans cet appel, la musique continue en arrière-plan. C'est le premier reproche que vous recevrez d'un testeur.

### Le branchement dans `DonjonGame`

```dart
  @override
  Future<void> onLoad() async {
    await super.onLoad();

    monde = world;
    camera.viewfinder.anchor = Anchor.center;
    camera.viewfinder.zoom = Constantes.zoomCamera;

    // AJOUT DU CHAPITRE 40 : services, dans cet ordre.
    await SauvegardeService.instance.initialiser();   // section 40.22
    await AudioService.instance.initialiser();        // section 40.3
    AudioService.instance.appliquerReglages(
      SauvegardeService.instance.donnees.reglages,    // section 40.25
    );

    meilleurScore = SauvegardeService.instance.donnees.meilleurScore;

    changerEtat(GameState.menu);
  }
```

L'ordre compte : on relit les réglages **avant** de configurer l'audio, sinon le premier son sortirait au volume par défaut.

> **À retenir.** `loadAll` dans `onLoad`, entouré d'un `try` / `catch` qui n'échoue jamais bruyamment. Et `FlameAudio.bgm.initialize()` une seule fois.

---

## 40.4 — Les effets : saut, coup, dégât, pièce, porte, mort

### La méthode `jouer`

```dart
  /// Demande la lecture d'un effet.
  ///
  /// Synchrone du point de vue de l'appelant : on ne veut pas d'`await`
  /// au milieu de `Joueur.sauter()`.
  void jouer(Effet effet) {
    // 1. Toujours journaliser, même en mode silencieux.
    journal.add(effet);
    if (journal.length > 64) {
      journal.removeAt(0);
    }

    // 2. Toujours prévenir le jeu : le retour visuel ne dépend pas du son.
    surEffet?.call(effet);

    // 3. Ensuite seulement, tenter le son.
    if (!disponible || !effetsActives || _volumeEffets <= 0) {
      return;
    }

    final double volume = _volumeEffets * (_equilibrage[effet] ?? 1.0);

    // `FlameAudio.play` renvoie un Future<AudioPlayer>. On ne l'attend pas,
    // mais on doit attraper son erreur, sinon elle devient une exception
    // non capturée (chapitre 15).
    FlameAudio.play(_fichiers[effet]!, volume: volume).catchError(
      (Object erreur) {
        disponible = false;
        debugPrint('[AudioService] Lecture impossible : $erreur');
        throw erreur;
      },
    );
  }
```

Le point délicat est le troisième. Relisez-le.

`FlameAudio.play` a pour signature réelle :

```dart
static Future<AudioPlayer> play(
  String file, {
  double volume = 1.0,
  AudioContext? audioContext,
  String? package,
});
```

C'est un `Future`. Si vous l'ignorez avec un simple `FlameAudio.play(...)` sans `await` et qu'il échoue, Dart signale une **erreur asynchrone non capturée**. Selon la plateforme, cela va du message rouge dans la console au crash de la zone. Le chapitre 15 a nommé ce piège. Ici, `.catchError` l'attrape.

> **Variante plus lisible.** Si vous préférez, extrayez une méthode privée `Future<void> _jouerFichier(...) async { try { await FlameAudio.play(...); } catch (_) { disponible = false; } }` et appelez-la sans `await`. Le comportement est identique ; le code complet du chapitre utilise cette forme, plus facile à lire pour un débutant.

### Où brancher chaque effet

C'est la vraie question de cette section. Un son mal placé est pire qu'une absence de son.

| Effet | Fichier | Endroit exact | Pourquoi là |
| --- | --- | --- | --- |
| `Effet.saut` | `composants/joueur.dart` | dans `sauter()`, **après** le test `auSol` | sinon le son sort même quand le saut est refusé |
| `Effet.coup` | `composants/joueur.dart` | dans `attaquer()`, au déclenchement | le son accompagne le geste, pas le contact |
| `Effet.degat` | `composants/joueur.dart` | dans `subirDegats()`, **après** le test `invincible` | un joueur invincible ne doit pas grésiller |
| `Effet.piece` | `composants/piece.dart` | dans `ramasser()` | une seule fois : le verrou du chapitre 38 s'en charge |
| `Effet.potion` | `composants/potion.dart` | dans `ramasser()` | même si le soin est inutile : le joueur a agi |
| `Effet.cle` | `composants/cle.dart` | dans `ramasser()` | |
| `Effet.porte` | `composants/porte.dart` | dans `ouvrir()`, quand la clé est acceptée | pas quand la porte reste verrouillée |
| `Effet.mort` | `donjon_game.dart` | dans `perdreUneVie()` | le joueur peut mourir sans avoir été touché (chute) |
| `Effet.boss` | `composants/boss.dart` | dans `onMount()` | l'arrivée du boss est un événement |
| `Effet.victoire` | `donjon_game.dart` | dans `changerEtat(victoire)` | |
| `Effet.clic` | `ecrans/*.dart` | dans les `onPressed` | |

### Les diffs

Trois exemples suffisent, le reste suit le même modèle.

```dart
// lib/composants/joueur.dart — dans sauter()

  void sauter() {
    if (!auSol) {
      return;                                  // saut refusé : pas de son
    }
    velocite.y = Constantes.forceSaut;
    auSol = false;
    etat = EtatJoueur.saut;
    AudioService.instance.jouer(Effet.saut);   // AJOUT DU CHAPITRE 40
  }
```

```dart
// lib/composants/joueur.dart — dans subirDegats()

  void subirDegats(double degats) {
    if (invincible || pv <= 0) {
      return;                                  // invincible : pas de son
    }
    AudioService.instance.jouer(Effet.degat);  // AJOUT DU CHAPITRE 40
    pv = (pv - degats).clamp(0.0, Constantes.pvJoueurMax);
    game.reinitialiserCombo();
    // ... suite du chapitre 38
  }
```

```dart
// lib/composants/piece.dart — dans ramasser()

  @override
  void ramasser(Joueur joueur) {
    AudioService.instance.jouer(Effet.piece);  // AJOUT DU CHAPITRE 40
    game.ajouterScore(valeur);
    game.piecesRamassees++;
    game.enregistrerEvenement(TypeEvenement.pieceRamassee, valeur);  // 40.18
  }
```

### Le piège du son en rafale

Ramassez huit pièces alignées en une seconde. Huit `FlameAudio.play` simultanés : selon la plateforme, vous obtenez un mur de bruit, ou pire, une saturation.

Deux parades, par ordre de simplicité.

**Parade 1 — le délai minimal entre deux occurrences du même effet.** Quinze lignes, aucune API nouvelle :

```dart
  /// Dernier instant de lecture de chaque effet, en millisecondes.
  final Map<Effet, int> _dernierJeu = <Effet, int>{};

  /// Délai minimal entre deux lectures du même effet, en millisecondes.
  static const Map<Effet, int> _delaiMinimal = <Effet, int>{
    Effet.piece: 60,
    Effet.saut: 80,
    Effet.degat: 120,
  };

  bool _tropTot(Effet effet) {
    final int? delai = _delaiMinimal[effet];
    if (delai == null) {
      return false;
    }
    final int maintenant = DateTime.now().millisecondsSinceEpoch;
    final int? precedent = _dernierJeu[effet];
    if (precedent != null && maintenant - precedent < delai) {
      return true;
    }
    _dernierJeu[effet] = maintenant;
    return false;
  }
```

Appelée en tête de `jouer`, juste après la journalisation, elle supprime le mur de bruit sans supprimer le son.

**Parade 2 — `AudioPool`.** Le paquet fournit une fabrique dont la signature réelle est, vérification faite le 8 août 2026 :

```dart
static Future<AudioPool> createPool(
  String sound, {
  required int maxPlayers,
  int minPlayers = 1,
  AudioContext? audioContext,
  String? package,
});
```

Le pool garde `maxPlayers` lecteurs prêts et renvoie, à chaque `start()`, une fonction d'arrêt. C'est la bonne solution pour un jeu de tir où l'on entend cent balles par seconde. Pour « Donjon de Dart », la parade 1 suffit largement, et elle ne coûte aucune mémoire. Nous nous en tenons là.

> **À retenir.** Un effet se déclenche là où la décision est prise, jamais avant le test qui peut l'annuler.

---

## 40.5 — La musique de fond par niveau

### `FlameAudio.bgm`, et pourquoi ce n'est pas `play`

Une musique dure trois minutes. Un effet dure trois dixièmes de seconde. Ce ne sont pas les mêmes objets techniques.

| | Effet (`FlameAudio.play`) | Musique (`FlameAudio.bgm`) |
| --- | --- | --- |
| Durée | < 2 s | plusieurs minutes |
| Nombre simultané | plusieurs | **un seul** |
| Boucle | non | oui |
| Préchargement en mémoire | oui | non (flux) |
| Pause automatique en arrière-plan | non | **oui**, après `initialize()` |

L'objet `Bgm` de `flame_audio` gère un lecteur unique et un observateur de cycle de vie. Son API réelle (dartdoc `flame_audio` 2.12.2) :

```dart
Future<void> initialize({AudioContext? audioContext});
Future<void> play(String fileName, {double volume = 1, String? package});
Future<void> stop();
Future<void> pause();
Future<void> resume();
Future<void> dispose();
bool        isPlaying;      // getter/setter
AudioPlayer audioPlayer;    // getter/setter
```

Retenez `audioPlayer` : c'est par lui que nous changerons le volume **à chaud**, à la section 40.6.

### Une musique par ambiance

Le jeu compte cinq ambiances, dont trois niveaux :

```dart
  /// Musique de chaque contexte. `null` = silence volontaire.
  static const Map<String, String> _musiques = <String, String>{
    'menu': 'musique_menu.mp3',
    'niveau_0': 'musique_donjon.mp3',
    'niveau_1': 'musique_donjon.mp3',
    'niveau_2': 'musique_crypte.mp3',
    'boss': 'musique_boss.mp3',
    'victoire': 'musique_victoire.mp3',
  };
```

Les niveaux 0 et 1 partagent volontairement la même piste. C'est un choix de production : trois musiques originales pour un jeu de cinq minutes, c'est disproportionné. On réserve le changement pour le moment où il raconte quelque chose — l'entrée dans la crypte, puis le boss.

### La méthode

```dart
  /// Piste en cours, pour ne pas relancer la même à chaque transition.
  String? _musiqueCourante;

  /// Lance la musique associée à une ambiance.
  ///
  /// Relancer la même piste ne fait RIEN : c'est ce qui évite le hoquet
  /// à chaque changement d'état.
  Future<void> jouerMusique(String ambiance) async {
    final String? fichier = _musiques[ambiance];

    if (fichier == null) {
      await arreterMusique();
      return;
    }
    if (fichier == _musiqueCourante && FlameAudio.bgm.isPlaying) {
      return;                                  // déjà en cours : on sort
    }
    _musiqueCourante = fichier;

    if (!disponible || !musiqueActivee || _volumeMusique <= 0) {
      return;
    }

    try {
      await FlameAudio.bgm.play(fichier, volume: _volumeMusique);
    } catch (erreur) {
      disponible = false;
      debugPrint('[AudioService] Musique indisponible : $erreur');
    }
  }

  /// Musique du niveau d'index [index], ou du boss.
  void musiqueDuNiveau(int index, {bool boss = false}) {
    jouerMusique(boss ? 'boss' : 'niveau_$index');
  }
```

Le test `fichier == _musiqueCourante` n'est pas une micro-optimisation. Sans lui, chaque passage par la pause redémarrerait la musique à zéro. Vous entendriez les trois premières notes en boucle chaque fois que le joueur appuie sur Échap. C'est le genre de bug qu'on ne voit jamais en lisant le code, et qu'on entend immédiatement.

### Le branchement dans le cycle de vie du jeu

```dart
// lib/donjon_game.dart — dans changerEtat(), AJOUT DU CHAPITRE 40

  void _appliquerAudio(GameState ancien, GameState nouveau) {
    final AudioService audio = AudioService.instance;

    switch (nouveau) {
      case GameState.menu:
        audio.jouerMusique('menu');
      case GameState.enJeu:
        if (ancien == GameState.pause) {
          audio.repriseMusique();              // on REPREND, on ne relance pas
        } else {
          audio.musiqueDuNiveau(niveauCourant);
        }
      case GameState.pause:
        audio.pauseMusique();
      case GameState.gameOver:
        audio.arreterMusique();
        audio.jouer(Effet.mort);
      case GameState.victoire:
        audio.arreterMusique();
        audio.jouer(Effet.victoire);
        audio.jouerMusique('victoire');
      case GameState.chargement:
        break;                                 // on ne touche à rien
    }
  }
```

Le `switch` exhaustif sur un enum, sans `default`, est celui du chapitre 11 : si vous ajoutez un septième état, le compilateur vous signale ce `switch`.

Distinguez bien les deux branches de `enJeu` :

- venant de `pause`, on **reprend** la piste là où elle s'était arrêtée ;
- venant d'ailleurs (chargement d'un niveau), on **lance** la piste du niveau.

Sans cette distinction, sortir de pause relancerait la musique depuis le début. Encore un bug inaudible à la lecture.

### Le changement de musique au boss

Le chapitre 39 fait apparaître le boss au troisième niveau. Une ligne suffit :

```dart
// lib/composants/boss.dart — AJOUT DU CHAPITRE 40

  @override
  void onMount() {
    super.onMount();
    AudioService.instance.jouer(Effet.boss);
    AudioService.instance.musiqueDuNiveau(game.niveauCourant, boss: true);
  }
```

> **À retenir.** Une musique se change avec `bgm.play`, se suspend avec `bgm.pause`, et ne se relance jamais si c'est déjà la bonne.

---

## 40.6 — Le volume musique et le volume effets

### Pourquoi deux réglages, et pas un

Parce que les joueurs ne coupent pas le son : ils coupent la **musique**. Ils veulent entendre les pas de l'ennemi qui approche, pas la boucle de guitare qu'ils connaissent par cœur depuis vingt minutes.

Un seul curseur « Volume » est le signe d'un jeu qui n'a pas été testé par un joueur.

### Des champs privés et des setters

```dart
  double _volumeMusique = 0.5;
  double _volumeEffets = 0.8;

  double get volumeMusique => _volumeMusique;
  double get volumeEffets => _volumeEffets;

  /// Le volume est appliqué IMMÉDIATEMENT à la musique en cours.
  set volumeMusique(double valeur) {
    _volumeMusique = valeur.clamp(0.0, 1.0);
    _appliquerVolumeMusique();
  }

  /// Les effets prennent le nouveau volume au prochain `jouer()`.
  set volumeEffets(double valeur) {
    _volumeEffets = valeur.clamp(0.0, 1.0);
  }

  void _appliquerVolumeMusique() {
    if (!disponible) {
      return;
    }
    try {
      // `audioPlayer` est une propriété publique de `Bgm`.
      FlameAudio.bgm.audioPlayer.setVolume(
        musiqueActivee ? _volumeMusique : 0.0,
      );
    } catch (erreur) {
      debugPrint('[AudioService] Volume musique : $erreur');
    }
  }
```

Le `clamp` du chapitre 38 revient : un `Slider` Flutter est borné, mais une valeur relue d'une sauvegarde corrompue ne l'est pas. Un volume de `12.0` ne fait pas planter le jeu, il le rend inaudible autrement — c'est-à-dire saturé.

### La différence de traitement

Elle est importante et souvent mal comprise.

```text
  Volume MUSIQUE                     Volume EFFETS
  --------------                     -------------
  un seul lecteur, qui joue          des lecteurs éphémères, créés
  en ce moment                       à chaque son

         │                                  │
         ▼                                  ▼
  il faut le modifier À CHAUD        il suffit de changer la valeur
  audioPlayer.setVolume(v)           utilisée au prochain play()
```

Si vous n'appelez pas `setVolume`, le joueur bouge le curseur de la musique et… rien ne se passe, jusqu'au prochain niveau. Il conclura que votre curseur est cassé. Il aura raison.

### Le retour visuel du curseur des effets

Un curseur de volume d'effets qui ne produit aucun son pendant qu'on le bouge est un curseur qu'on règle à l'aveugle. La solution universelle : jouer un son témoin quand on relâche le curseur.

```dart
Slider(
  value: _effets,
  activeColor: Palette.accent,
  onChanged: (double v) => setState(() => _effets = v),
  onChangeEnd: (double v) {
    AudioService.instance.volumeEffets = v;
    AudioService.instance.jouer(Effet.piece);   // son témoin
  },
),
```

`onChanged` met à jour l'affichage soixante fois par seconde ; `onChangeEnd` n'est appelé qu'une fois, au relâchement. Jouer le son dans `onChanged` produirait deux cents sons pendant le glissement.

### Les interrupteurs

```dart
  void basculerMusique() {
    musiqueActivee = !musiqueActivee;
    if (musiqueActivee) {
      repriseMusique();
    } else {
      pauseMusique();
    }
  }

  void basculerEffets() {
    effetsActives = !effetsActives;
  }
```

Un interrupteur n'est pas un volume à zéro : il retient la position du curseur. Le joueur qui coupe la musique puis la rallume retrouve son réglage. C'est ce que fait `basculerMusique` : le champ `_volumeMusique` n'est jamais touché.

> **À retenir.** Le volume de la musique s'applique à chaud sur `bgm.audioPlayer` ; celui des effets s'applique au prochain son.

---

## 40.7 — Couper le son proprement

### Ce qu'on oublie toujours

Un lecteur audio est une **ressource système**. Sur Android, c'est un `MediaPlayer` ; sur le Web, un élément `<audio>`. Si vous ne le relâchez pas, il reste.

Les trois symptômes classiques :

1. la musique du menu continue par-dessus celle du niveau ;
2. le jeu, relancé après un *hot restart*, joue deux musiques superposées ;
3. sur mobile, l'application garde le focus audio et empêche la musique du système de reprendre.

### Les trois niveaux de coupure

```dart
  /// Suspend la musique. Elle reprendra à la même seconde.
  Future<void> pauseMusique() async {
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.pause();
    } catch (_) {
      // Rien à faire : couper un son déjà coupé n'est pas une erreur.
    }
  }

  /// Reprend la musique suspendue, si le joueur ne l'a pas désactivée.
  Future<void> repriseMusique() async {
    if (!disponible || !musiqueActivee) {
      return;
    }
    try {
      await FlameAudio.bgm.resume();
    } catch (_) {}
  }

  /// Arrête la musique et oublie la piste courante.
  Future<void> arreterMusique() async {
    _musiqueCourante = null;                   // INDISPENSABLE
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.stop();
    } catch (_) {}
  }
```

L'oubli de `_musiqueCourante = null` dans `arreterMusique` est un bug subtil : après un arrêt, `jouerMusique` avec la même piste croirait qu'elle joue déjà et ne relancerait rien. Le jeu deviendrait silencieux sans erreur.

### `couperTout` : le bouton panique

```dart
  /// Coupe absolument tout. Utilisé par le raccourci M et par l'écran
  /// d'options en cas de doute.
  Future<void> couperTout() async {
    musiqueActivee = false;
    effetsActives = false;
    await arreterMusique();
  }
```

### `liberer` : la libération définitive

```dart
  /// Libère le lecteur de musique et vide le cache des effets.
  /// À appeler quand le jeu est détruit, PAS à chaque changement d'écran.
  Future<void> liberer() async {
    _musiqueCourante = null;
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.dispose();
      await FlameAudio.audioCache.clearAll();
    } catch (erreur) {
      debugPrint('[AudioService] Libération : $erreur');
    } finally {
      disponible = false;
    }
  }
```

Le `finally` du chapitre 13 garantit que `disponible` retombe à `false` même si `dispose()` échoue. Sans lui, un appel ultérieur à `jouer()` tenterait d'utiliser un lecteur détruit.

### Où appeler `liberer`

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  @override
  void onRemove() {
    AudioService.instance.liberer();
    super.onRemove();
  }
```

`onRemove` est appelé une seule fois, avant le retrait du composant racine (fiche de référence, section 4.1). C'est le bon endroit.

> **Attention au hot restart.** En développement, un *hot restart* recrée l'instance de jeu sans forcément passer par `onRemove`. Si vous entendez deux musiques superposées après un `r` dans la console, ce n'est pas votre code qui est faux : arrêtez et relancez l'application (`R` majuscule, ou `flutter run` à nouveau). Le comportement est correct en production.

> **À retenir.** `pause` suspend, `stop` arrête et oublie, `dispose` détruit. Trois verbes, trois moments.

---

## 40.8 — Faire tourner le jeu sans aucun fichier audio : le service en mode silencieux

### Le contrat

Vous n'avez rien téléchargé. Le dossier `assets/audio/` est vide. Et pourtant :

- le jeu se lance ;
- aucune exception n'apparaît dans la console, sauf un message d'information ;
- chaque événement sonore produit un **retour visuel** ;
- le jour où vous déposez un fichier `piece.wav` dans `assets/audio/`, il se met à sonner sans qu'une ligne de code change.

C'est la dégradation gracieuse de la section 34.15, appliquée au jeu complet.

### Le mécanisme, en une image

```text
              joueur.sauter()
                     │
                     ▼
        AudioService.instance.jouer(Effet.saut)
                     │
        ┌────────────┴─────────────┐
        │                          │
   journal.add(...)          surEffet?.call(...)
        │                          │
        │                          ▼
        │              DonjonGame._retourVisuel(effet)
        │                          │
        ▼                          ▼
  disponible == false ?      flash / texte flottant
        │                    TOUJOURS affiché
   ┌────┴─────┐
  oui        non
   │          │
 on sort   FlameAudio.play(...)
```

Notez bien : le retour visuel est déclenché **avant** le test `disponible`. Il n'est donc pas une compensation de l'absence de son ; il est là dans les deux cas. C'est une règle d'accessibilité, pas un bricolage : un joueur sourd, ou un joueur dans le train sans écouteurs, joue avec les mêmes informations que les autres.

### Le retour visuel côté jeu

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  /// Les effets qui méritent un retour visible quand le son est absent.
  static const Map<Effet, String> _symbolesAudio = <Effet, String>{
    Effet.coup: '!',
    Effet.degat: '!!',
    Effet.porte: 'CLIC',
    Effet.cle: 'CLÉ',
    Effet.boss: 'BOSS',
  };

  /// Branché sur `AudioService.surEffet` dans `onLoad()`.
  void _retourVisuel(Effet effet) {
    // Rien à faire si le son sort vraiment ET que le jeu n'est pas en pause.
    if (AudioService.instance.disponible) {
      return;
    }
    final String? symbole = _symbolesAudio[effet];
    if (symbole == null || joueur == null) {
      return;
    }
    afficherTexteFlottant(         // méthode du chapitre 38
      joueur!.position,
      symbole,
      Palette.accent,
      taille: 9,
    );
  }
```

Et le branchement, dans `onLoad` :

```dart
    AudioService.instance.surEffet = _retourVisuel;
```

Une seule ligne. Le service ne connaît toujours pas `DonjonGame`, `Palette`, ni `TexteFlottant`. Il ne connaît qu'une fonction. Si demain vous écrivez un jeu différent avec le même service, vous branchez une autre fonction.

### Le bandeau du menu

Un élève qui lance le jeu pour la première fois doit comprendre pourquoi il n'entend rien. Un message discret dans le menu suffit :

```dart
// lib/ecrans/menu_principal.dart — AJOUT DU CHAPITRE 40

if (!AudioService.instance.disponible)
  const Padding(
    padding: EdgeInsets.only(top: 12),
    child: Text(
      'Mode silencieux : aucun fichier dans assets/audio/',
      style: TextStyle(color: Palette.texteFaible, fontSize: 11),
    ),
  ),
```

### Tester le service sans Flutter et sans son

C'est le bénéfice caché du `journal`. Le service peut être vérifié dans un test unitaire ordinaire, sans moteur de jeu, sans plugin de plateforme :

```dart
// test/audio_service_test.dart
import 'package:donjon_de_dart/services/audio_service.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('le journal enregistre même en mode silencieux', () {
    final AudioService audio = AudioService.instance;
    audio.disponible = false;         // on force le mode silencieux
    audio.journal.clear();

    audio.jouer(Effet.saut);
    audio.jouer(Effet.piece);
    audio.jouer(Effet.piece);

    expect(audio.journal.length, 3);
    expect(audio.journal.where((Effet e) => e == Effet.piece).length, 2);
  });

  test('surEffet est appelé même sans fichier', () {
    final AudioService audio = AudioService.instance;
    audio.disponible = false;
    final List<Effet> vus = <Effet>[];
    audio.surEffet = vus.add;

    audio.jouer(Effet.degat);

    expect(vus, <Effet>[Effet.degat]);
    audio.surEffet = null;            // on nettoie : c'est un singleton
  });
}
```

Le `audio.surEffet = null` final n'est pas cosmétique. Un singleton garde son état entre les tests ; oublier de le nettoyer produit des échecs qui dépendent de l'ordre d'exécution — l'un des bugs les plus pénibles à diagnostiquer.

### Passer en mode sonore

Le jour où vous voulez du son, la procédure tient en trois lignes.

1. Déposez vos fichiers dans `assets/audio/`, avec **exactement** les noms de `_fichiers`.
2. Vérifiez que `pubspec.yaml` déclare bien `- assets/audio/` (le dossier, pas chaque fichier).
3. `flutter clean` puis `flutter run`.

Vous n'avez modifié aucune ligne de code Dart. Si un seul fichier manque, `loadAll` échoue globalement et vous repassez en silencieux. Pour un préchargement fichier par fichier, tolérant aux manques, voyez l'exercice 2.

> **À retenir.** Le mode silencieux n'est pas un mode dégradé de secours : c'est le mode par défaut du projet, et il est complet.

---

## 40.9 — La pause : rappel de `pauseEngine()` (chapitre 35)

### Ce que `pauseEngine` fait exactement

La section 35.14 l'a introduite. Rappel de la signature réelle, sur le mixin `Game` :

```dart
void pauseEngine();
void resumeEngine();
bool paused;          // getter/setter
```

`pauseEngine()` arrête la boucle de jeu. Concrètement :

| Ce qui s'arrête | Ce qui continue |
| --- | --- |
| `update(dt)` de tous les composants | le rendu Flutter du dernier état |
| les `Effect` de Flame (`MoveEffect`, `OpacityEffect`…) | les widgets des overlays, avec leurs animations |
| les `Timer` et `TimerComponent` | les `AnimationController` de Flutter |
| la détection de collision | le lecteur audio, qui n'est pas dans la boucle |
| les `SpriteAnimationTicker` | les gestes Flutter |

Cette table est la clé de tout le chapitre. Relisez la colonne de droite : **votre écran de pause doit être un widget Flutter**, parce qu'un composant Flame ne bougerait plus.

### Le corollaire audio

La musique **ne s'arrête pas** avec `pauseEngine()`. Le lecteur audio vit dans le système, pas dans la boucle de jeu. C'est pour cela que `_appliquerAudio` de la section 40.5 appelle explicitement `audio.pauseMusique()` en entrant dans l'état `pause`.

C'est un test facile à faire et très parlant : commentez cette ligne, mettez le jeu en pause, la musique continue. Beaucoup de jeux d'étudiants ont ce défaut.

### La pause est déjà à moitié écrite

Regardez le `changerEtat` du chapitre 35 :

```dart
    if (nouvelEtat == GameState.enJeu) {
      resumeEngine();
    } else if (nouvelEtat == GameState.pause ||
        nouvelEtat == GameState.gameOver ||
        nouvelEtat == GameState.victoire) {
      pauseEngine();
    }
```

Tout est là. `changerEtat(GameState.pause)` gèle déjà le monde et affiche déjà l'overlay `Overlays.pause`. Il ne manque que deux choses : **le widget** derrière cet overlay, et **les déclencheurs**.

### `basculerPause()`

Une seule méthode publique, qui protège contre les états où la pause n'a aucun sens :

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  /// Bascule entre `enJeu` et `pause`.
  ///
  /// Ne fait rien dans les autres états : on ne met pas en pause un
  /// menu, un Game Over ou un écran de chargement.
  void basculerPause() {
    switch (etat) {
      case GameState.enJeu:
        changerEtat(GameState.pause);
      case GameState.pause:
        changerEtat(GameState.enJeu);
      case GameState.chargement:
      case GameState.menu:
      case GameState.gameOver:
      case GameState.victoire:
        break;
    }
  }

  /// Met en pause si, et seulement si, une partie est en cours.
  /// Utilisée par la perte de focus (section 40.12) : elle ne REPREND jamais.
  void mettreEnPause() {
    if (etat == GameState.enJeu) {
      changerEtat(GameState.pause);
    }
  }
```

La distinction entre `basculerPause` et `mettreEnPause` est essentielle et vous la retrouverez à la section 40.12. Une bascule appelée par la perte de focus **sortirait** de la pause si le joueur avait déjà mis en pause avant de changer de fenêtre. Ce serait exactement le contraire de ce qu'il attend.

### Le piège du `Future` en attente

Un point que la documentation ne souligne pas assez. Ceci est dangereux :

```dart
// À NE PAS FAIRE
Future<void> terminerNiveau() async {
  changerEtat(GameState.chargement);
  await Future<void>.delayed(const Duration(milliseconds: 400));
  await chargerNiveau(niveauCourant + 1);
  changerEtat(GameState.enJeu);
}
```

Si le joueur met en pause pendant ces 400 millisecondes, le `changerEtat(GameState.enJeu)` final écrasera sa pause. `pauseEngine()` n'arrête pas les `Future` : ils vivent dans la boucle d'événements de Dart, pas dans celle de Flame (chapitre 15).

La parade est simple : à la fin d'une opération asynchrone, on vérifie que le contexte n'a pas changé.

```dart
  Future<void> terminerNiveau() async {
    final int cible = niveauCourant + 1;
    changerEtat(GameState.chargement);

    if (cible >= Constantes.nombreNiveaux) {
      declarerVictoire();                       // section 40.17
      return;
    }

    await chargerNiveau(cible);

    // Le joueur a pu revenir au menu entre-temps.
    if (etat != GameState.chargement) {
      return;
    }
    changerEtat(GameState.enJeu);
  }
```

> **À retenir.** `pauseEngine()` gèle Flame, pas Flutter, pas l'audio, pas les `Future`. Chacun des trois se gère à la main.

---

## 40.10 — `lib/ecrans/ecran_pause.dart`

### Le cahier des charges d'un écran de pause

| Exigence | Conséquence technique |
| --- | --- |
| Le jeu reste visible derrière | fond semi-transparent, pas opaque |
| Le joueur voit où il en est | rappel du niveau, du score et des vies |
| Reprendre est l'action évidente | bouton principal, mis en valeur, en premier |
| Quitter ne doit pas être une catastrophe | confirmation avant de perdre la partie |
| La touche Échap referme l'écran | gestion du clavier côté widget aussi |

### Le fond semi-transparent

C'est ce qui distingue une pause d'un menu. Le joueur doit voir sa position gelée derrière le panneau : cela lui rappelle qu'une partie est en cours, et cela lui donne le temps de réfléchir à sa prochaine action.

```dart
Material(
  color: Palette.fondJeu.withValues(alpha: 0.78),
  child: /* ... */,
)
```

`withValues(alpha:)` est la méthode courante de `Color` ; l'ancienne `withOpacity()` est dépréciée dans les versions récentes de Flutter. Le chapitre 35 utilise déjà `withValues` dans le menu principal.

### Le widget complet

```dart
// lib/ecrans/ecran_pause.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../core/game_state.dart';
import '../donjon_game.dart';
import '../services/audio_service.dart';
import 'ecran_options.dart';

/// L'écran de pause, affiché en overlay quand `etat == GameState.pause`.
///
/// C'est un widget FLUTTER : le moteur Flame est gelé, un composant Flame
/// ne serait plus mis à jour (section 40.9).
class EcranPause extends StatelessWidget {
  const EcranPause({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    return Focus(
      autofocus: true,
      onKeyEvent: (FocusNode node, KeyEvent event) {
        // Échap referme la pause, même si le focus a quitté le canvas.
        if (event is KeyDownEvent &&
            event.logicalKey == LogicalKeyboardKey.escape) {
          jeu.basculerPause();
          return KeyEventResult.handled;
        }
        return KeyEventResult.ignored;
      },
      child: Material(
        // Semi-transparent : le monde gelé reste visible derrière.
        color: Palette.fondJeu.withValues(alpha: 0.78),
        child: Center(
          child: SingleChildScrollView(
            child: Container(
              width: 340,
              padding: const EdgeInsets.symmetric(
                horizontal: 28,
                vertical: 26,
              ),
              decoration: BoxDecoration(
                color: Palette.panneau,
                border: Border.all(color: Palette.mur, width: 3),
                borderRadius: BorderRadius.circular(10),
              ),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: <Widget>[
                  const Text(
                    'PAUSE',
                    style: TextStyle(
                      fontSize: 34,
                      fontWeight: FontWeight.w900,
                      letterSpacing: 8,
                      color: Palette.texte,
                    ),
                  ),
                  const SizedBox(height: 18),
                  _LigneEtat(jeu: jeu),
                  const SizedBox(height: 22),
                  BoutonEcran(
                    libelle: 'REPRENDRE',
                    icone: Icons.play_arrow,
                    principal: true,
                    onPressed: () {
                      AudioService.instance.jouer(Effet.clic);
                      jeu.basculerPause();
                    },
                  ),
                  const SizedBox(height: 10),
                  BoutonEcran(
                    libelle: 'OPTIONS',
                    icone: Icons.settings,
                    onPressed: () {
                      AudioService.instance.jouer(Effet.clic);
                      showDialog<void>(
                        context: context,
                        builder: (BuildContext c) => EcranOptions(jeu: jeu),
                      );
                    },
                  ),
                  const SizedBox(height: 10),
                  BoutonEcran(
                    libelle: 'QUITTER AU MENU',
                    icone: Icons.home,
                    onPressed: () => _confirmerAbandon(context),
                  ),
                  const SizedBox(height: 16),
                  const Text(
                    'Échap : reprendre',
                    style: TextStyle(
                      color: Palette.texteFaible,
                      fontSize: 11,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  /// Abandonner une partie en cours est destructif : on confirme.
  void _confirmerAbandon(BuildContext context) {
    AudioService.instance.jouer(Effet.clic);
    showDialog<void>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        backgroundColor: Palette.panneau,
        title: const Text('Abandonner la partie ?'),
        content: Text(
          'Vous perdrez votre progression dans le niveau '
          '${jeu.niveauCourant + 1}.\n'
          'Votre meilleur score, lui, est conservé.',
          style: const TextStyle(color: Palette.texteFaible),
        ),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(c).pop(),
            child: const Text('CONTINUER À JOUER'),
          ),
          FilledButton(
            style: FilledButton.styleFrom(backgroundColor: Palette.danger),
            onPressed: () {
              Navigator.of(c).pop();
              jeu.abandonnerPartie();          // section 40.11
            },
            child: const Text('ABANDONNER'),
          ),
        ],
      ),
    );
  }
}

/// Rappel compact de l'état de la partie.
class _LigneEtat extends StatelessWidget {
  const _LigneEtat({required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text(
          'NIVEAU ${jeu.niveauCourant + 1} / ${Constantes.nombreNiveaux}',
          style: Palette.sousTitre,
        ),
        const SizedBox(height: 6),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: <Widget>[
            _Pastille(libelle: 'SCORE', valeur: '${jeu.score}'),
            _Pastille(libelle: 'VIES', valeur: '${jeu.vies}'),
            _Pastille(libelle: 'PIÈCES',
                valeur: '${jeu.piecesRamassees}/${jeu.piecesDuNiveau}'),
          ],
        ),
      ],
    );
  }
}

class _Pastille extends StatelessWidget {
  const _Pastille({required this.libelle, required this.valeur});

  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text(
          valeur,
          style: const TextStyle(
            color: Palette.accent,
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          libelle,
          style: const TextStyle(color: Palette.texteFaible, fontSize: 10),
        ),
      ],
    );
  }
}

/// Bouton commun aux trois écrans de fin. Volontairement proche de
/// `BoutonMenu` (chapitre 35), mais dimensionné pour un panneau.
class BoutonEcran extends StatelessWidget {
  const BoutonEcran({
    super.key,
    required this.libelle,
    required this.icone,
    required this.onPressed,
    this.principal = false,
  });

  final String libelle;
  final IconData icone;
  final VoidCallback? onPressed;
  final bool principal;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      height: 48,
      child: principal
          ? FilledButton.icon(
              onPressed: onPressed,
              icon: Icon(icone),
              label: Text(libelle, style: Palette.bouton),
              style: FilledButton.styleFrom(
                backgroundColor: Palette.accent,
                foregroundColor: Palette.fondMenu,
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
                side: const BorderSide(color: Palette.mur, width: 2),
                shape: const RoundedRectangleBorder(
                  borderRadius: BorderRadius.all(Radius.circular(6)),
                ),
              ),
            ),
    );
  }
}
```

### Pourquoi `Focus` et pas un `KeyboardListener`

Quand un overlay Flutter est affiché, il peut capter le focus clavier. Le `GameWidget` ne reçoit alors plus les touches, et l'`onKeyEvent` de `DonjonGame` (chapitre 35) ne voit plus rien : Échap ne ferait plus sortir de la pause.

Le widget `Focus` avec `autofocus: true` règle le problème à la source. Sa signature de rappel, en Flutter récent, est :

```dart
KeyEventResult Function(FocusNode node, KeyEvent event) onKeyEvent;
```

`KeyEventResult.handled` arrête la propagation, `KeyEventResult.ignored` la laisse continuer. Ce sont les mêmes valeurs que celles de Flame (fiche de référence, section 6.1), et ce n'est pas un hasard : Flame s'appuie sur le système de focus de Flutter.

> **À retenir.** L'écran de pause est un widget Flutter, semi-transparent, dont le bouton principal est Reprendre, et qui gère Échap lui-même.

---

## 40.11 — Reprendre, Options, Quitter au menu

### Les trois actions, et ce qu'elles touchent

```text
  REPRENDRE          →  changerEtat(enJeu)
                        resumeEngine() + reprise de la musique
                        RIEN d'autre n'est modifié

  OPTIONS            →  showDialog(EcranOptions)
                        le jeu reste en pause pendant le réglage
                        les réglages sont ENREGISTRÉS à la fermeture

  QUITTER AU MENU    →  confirmation
                        sauvegarde du meilleur score et de la progression
                        vidage du monde
                        changerEtat(menu)
```

### Reprendre

`jeu.basculerPause()` suffit. Toute la mécanique est dans `changerEtat` : `resumeEngine()`, retrait de l'overlay `pause`, ajout de l'overlay `hud`, reprise de la musique.

Notez ce qu'on ne fait **pas** : on ne recharge pas le niveau, on ne réinitialise pas la position du joueur, on ne remet pas le score à zéro. Une pause ne modifie rien. Si votre pause change quoi que ce soit à l'état du jeu, elle est fausse.

### Options depuis la pause

Le point subtil : `showDialog` empile une route par-dessus l'overlay. Le jeu, lui, reste en `GameState.pause`. Quand la boîte se ferme, on retombe sur l'écran de pause, pas sur le jeu. C'est le comportement attendu.

Attention toutefois à un détail : si le joueur coupe la musique depuis les options **pendant la pause**, la musique est déjà suspendue. Au retour en jeu, `repriseMusique()` testera `musiqueActivee` et ne reprendra rien. Le comportement est correct sans code supplémentaire — mais uniquement parce que `repriseMusique` teste bien `musiqueActivee`. C'est le genre de garde qui paie deux sections plus loin.

### Quitter au menu : `abandonnerPartie()`

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  /// Abandonne la partie en cours et revient au menu principal.
  ///
  /// Sauvegarde ce qui mérite de l'être AVANT de tout effacer.
  void abandonnerPartie() {
    // 1. Ce qui survit à la partie.
    _sauvegarderProgression();     // section 40.24

    // 2. Retour au menu : changerEtat vide le monde (chapitre 35).
    changerEtat(GameState.menu);
  }

  /// Persiste meilleur score et niveau atteint. Ne bloque jamais l'appelant.
  void _sauvegarderProgression() {
    final SauvegardeService s = SauvegardeService.instance;

    // `unawaited` : la sauvegarde est asynchrone, la navigation ne l'est pas.
    unawaited(s.enregistrerMeilleurScore(meilleurScore));
    unawaited(s.enregistrerProgression(niveauCourant));
  }
```

`unawaited` vient de `dart:async` et documente une intention : « je sais que ce `Future` n'est pas attendu, c'est volontaire ». Sans lui, l'analyseur signale `unawaited_futures` si vous l'avez activé, et surtout un relecteur se demande si c'est un oubli.

### Le piège du monde vidé

`changerEtat(GameState.menu)` appelle `viderLeMonde()`, qui fait :

```dart
  void viderLeMonde() {
    monde.removeAll(monde.children.toList());
  }
```

Le `.toList()` du chapitre 35 est indispensable : `monde.children` est la liste vivante des enfants ; la parcourir en la modifiant lève une `ConcurrentModificationError` (chapitre 6).

Conséquence pour nous : après un retour au menu, `game.joueur` vaut `null` — c'est le `onRemove` du chapitre 38 qui s'en charge. Tout code d'écran qui lit `jeu.joueur!.pv` plantera. Les écrans de ce chapitre lisent donc `jeu.score` et `jeu.vies`, qui sont des champs du jeu, jamais des champs du joueur.

### Et le HUD ?

Le `Hud` du chapitre 38 vit dans `camera.viewport`, pas dans le monde. `viderLeMonde()` ne le retire donc pas. Il faut l'enlever explicitement, sinon le menu principal s'affiche avec une barre de vie par-dessus.

```dart
  /// Retire le HUD du viewport, s'il y est.
  void retirerHud() {
    camera.viewport.children.whereType<Hud>().toList().forEach(
      (Hud h) => h.removeFromParent(),
    );
  }
```

Et dans `changerEtat`, à l'entrée dans `menu` :

```dart
    if (nouvelEtat == GameState.menu) {
      retirerHud();          // AJOUT DU CHAPITRE 40
      viderLeMonde();
    }
```

`whereType<Hud>()` est du chapitre 14 : il filtre et retype en une opération. `.toList()` pour la même raison que plus haut.

> **À retenir.** Reprendre ne modifie rien. Quitter sauvegarde d'abord, efface ensuite, et n'oublie pas le HUD qui vit hors du monde.

---

## 40.12 — La pause sur perte de focus (`AppLifecycleState`)

### Le scénario

Le joueur est à trois pixels du boss. Un appel arrive. L'application passe en arrière-plan.

Dans un jeu naïf, deux choses se produisent, toutes deux mauvaises :

1. la boucle de jeu continue de tourner en arrière-plan, ou reprend brutalement avec un `dt` énorme au retour ;
2. le héros meurt pendant l'appel.

Le second point mérite un mot. Flutter limite le `dt` transmis, mais la logique de votre jeu, elle, ne le sait pas. Un jeu correct **se met en pause tout seul**.

### `AppLifecycleState`

L'enum vient de `package:flutter/widgets.dart` :

| Valeur | Signification |
| --- | --- |
| `resumed` | l'application est visible et reçoit les entrées |
| `inactive` | visible mais ne reçoit pas les entrées (appel entrant, panneau système) |
| `paused` | invisible, en arrière-plan |
| `detached` | plus attachée à une vue |
| `hidden` | masquée (ajoutée aux versions récentes de Flutter) |

### La bonne méthode : `lifecycleStateChange`

Vous pourriez ajouter un `WidgetsBindingObserver` sur votre `State`. C'est inutile : Flame fournit déjà le point d'accroche, sur le mixin `Game` (dartdoc vérifié le 8 août 2026) :

```dart
void lifecycleStateChange(AppLifecycleState state);
```

Documentation officielle : « This is the lifecycle state change hook; every time the game is resumed, paused or suspended, this is called. »

Il suffit de la redéfinir.

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  @override
  void lifecycleStateChange(AppLifecycleState state) {
    super.lifecycleStateChange(state);

    switch (state) {
      case AppLifecycleState.inactive:
      case AppLifecycleState.paused:
      case AppLifecycleState.hidden:
      case AppLifecycleState.detached:
        // On MET en pause. On ne bascule pas : si le joueur avait déjà
        // mis en pause, il doit retrouver son écran de pause.
        mettreEnPause();
        AudioService.instance.pauseMusique();

      case AppLifecycleState.resumed:
        // On ne REPREND PAS le jeu automatiquement.
        // Le joueur reprend quand il est prêt, en appuyant sur REPRENDRE.
        if (etat != GameState.pause) {
          AudioService.instance.repriseMusique();
        }
    }
  }
```

### Les deux décisions de conception

Elles sont plus importantes que le code.

**Décision 1 : on met en pause, on ne bascule pas.** D'où `mettreEnPause()` et non `basculerPause()`. Le raisonnement est à la section 40.9 : une bascule sortirait de la pause si le joueur y était déjà.

**Décision 2 : on ne reprend jamais tout seul.** C'est contre-intuitif et c'est pourtant la règle de tous les bons jeux mobiles. Quand vous revenez dans l'application, votre pouce est encore en train de fermer une notification. Si le jeu redémarre à l'instant précis où l'écran s'affiche, vous mourez. On laisse donc le joueur appuyer sur REPRENDRE.

### Le cas du Web

Sur le Web, changer d'onglet déclenche bien `AppLifecycleState.hidden` puis `resumed` dans les versions récentes de Flutter. Le comportement est donc correct.

Notez toutefois que la plupart des navigateurs suspendent d'eux-mêmes les *timers* d'un onglet inactif : votre jeu s'arrête même sans ce code. Le code reste utile, parce qu'il affiche l'écran de pause, ce que le navigateur ne fait pas.

### `Bgm` fait déjà la moitié du travail

Si vous avez appelé `FlameAudio.bgm.initialize()` (section 40.3), l'objet `Bgm` suspend et reprend la musique tout seul selon le cycle de vie. Les appels explicites de notre `lifecycleStateChange` sont donc partiellement redondants — et c'est très bien : ils rendent le comportement lisible, et ils fonctionnent même si vous remplacez `bgm` par autre chose.

Le seul cas où la redondance serait gênante serait un double `resume` qui redémarrerait la piste. Ce n'est pas le cas : `resume()` sur un lecteur déjà en lecture ne fait rien.

> **À retenir.** Redéfinissez `lifecycleStateChange`, mettez en pause à la perte de focus, et ne reprenez jamais automatiquement.

---

## 40.13 — La touche Échap et le bouton Pause du HUD

### Le nouveau rôle d'Échap

Au chapitre 35, Échap revenait directement au menu. C'était un bouchon assumé. Le comportement définitif est celui de tous les jeux :

| Contexte | Échap |
| --- | --- |
| `enJeu` | met en pause |
| `pause` | reprend |
| `menu` | ne fait rien |
| `gameOver`, `victoire` | ne fait rien (les boutons sont explicites) |

Le diff sur l'`onKeyEvent` de `DonjonGame` :

```dart
// lib/donjon_game.dart — MODIFIÉ AU CHAPITRE 40

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent) {
      // Échap : PAUSE (chapitre 40). Avant : retour au menu (chapitre 35).
      if (event.logicalKey == LogicalKeyboardKey.escape) {
        basculerPause();
        return KeyEventResult.handled;
      }

      // P : synonyme d'Échap, habitude des joueurs PC.
      if (event.logicalKey == LogicalKeyboardKey.keyP) {
        basculerPause();
        return KeyEventResult.handled;
      }

      // M : couper ou rétablir la musique (chapitre 40).
      if (event.logicalKey == LogicalKeyboardKey.keyM) {
        AudioService.instance.basculerMusique();
        return KeyEventResult.handled;
      }

      // F1 et F2 : debug (chapitre 35), inchangés.
      // ...
    }

    return super.onKeyEvent(event, keysPressed);
  }
```

Remarquez qu'on ne teste plus `if (etat == GameState.enJeu)` : c'est `basculerPause()` qui contient la logique d'état. Une seule méthode décide, trois déclencheurs l'appellent. C'est la règle du point de passage unique du chapitre 35, appliquée à la pause.

### Le retour de touche pendant la pause

Un détail qui perturbe beaucoup d'élèves : quand l'overlay de pause est affiché et qu'il a pris le focus, l'`onKeyEvent` de `DonjonGame` **ne reçoit plus rien**. C'est pour cela que l'`EcranPause` de la section 40.10 gère Échap de son côté, avec un `Focus`.

Vous avez donc deux gestionnaires d'Échap. Ce n'est pas une duplication maladroite : ce sont deux mondes différents (Flame et Flutter), et ils appellent tous deux la même méthode `basculerPause()`. La logique, elle, n'est écrite qu'une fois.

### Le bouton Pause du HUD

Sur mobile, il n'y a pas de touche Échap. Il faut un bouton, et il doit être :

- assez grand pour un pouce (48 points de côté au minimum) ;
- en haut à droite, loin des commandes de déplacement (chapitre 38, section 38.29) ;
- visible sur n'importe quel fond.

Flame fournit `HudButtonComponent`, dont la signature réelle est rappelée dans la fiche de référence (section 6.5) :

```dart
HudButtonComponent({
  PositionComponent? button,
  PositionComponent? buttonDown,
  EdgeInsets? margin,
  void Function()? onPressed,
  void Function()? onReleased,
  void Function()? onCancelled,
  Vector2? position,
  Vector2? size,
  // ...
});
```

```dart
// lib/hud/hud.dart — AJOUT DU CHAPITRE 40
import 'package:flame/input.dart';

/// Bouton Pause du HUD, en haut à droite.
///
/// Il vit dans le VIEWPORT (chapitre 38) : il ne bouge pas avec la caméra.
class BoutonPause extends HudButtonComponent with HasGameReference<DonjonGame> {
  BoutonPause()
      : super(
          size: Vector2.all(30),
          margin: const EdgeInsets.only(top: 10, right: 10),
          button: RectangleComponent(
            size: Vector2.all(30),
            paint: Paint()..color = Palette.panneau,
            children: <Component>[
              // Deux barres verticales : le symbole « pause ».
              RectangleComponent(
                position: Vector2(9, 8),
                size: Vector2(4, 14),
                paint: Paint()..color = Palette.texte,
              ),
              RectangleComponent(
                position: Vector2(17, 8),
                size: Vector2(4, 14),
                paint: Paint()..color = Palette.texte,
              ),
            ],
          ),
          buttonDown: RectangleComponent(
            size: Vector2.all(30),
            paint: Paint()..color = Palette.accent,
          ),
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // `onPressed` ne peut pas référencer `game` dans le constructeur :
    // la référence n'est fiable qu'à partir d'onLoad (fiche, section 12.2).
    onPressed = () {
      AudioService.instance.jouer(Effet.clic);
      game.basculerPause();
    };
  }
}
```

Et son ajout, dans `Hud.onLoad()` :

```dart
    await add(BoutonPause());        // AJOUT DU CHAPITRE 40
```

### Le piège du bouton pendant la pause

Le bouton Pause est un composant Flame. Quand le moteur est en pause, ses `update` ne tournent plus. **Il ne faut donc pas compter dessus pour sortir de la pause.**

La sortie se fait par le bouton REPRENDRE de l'écran Flutter, ou par Échap. Le bouton du HUD n'a qu'un seul rôle : entrer en pause. C'est suffisant, et c'est cohérent avec ce qu'on voit dans les jeux réels.

Si vous voulez malgré tout un bouton unique qui fasse les deux, mettez-le dans un overlay Flutter, pas dans le HUD Flame. L'exercice 5 vous fait écrire cette variante.

### Récapitulatif des déclencheurs

| Déclencheur | Plateforme | Entre en pause | Sort de pause |
| --- | --- | --- | --- |
| Touche Échap (Flame) | bureau, Web | oui | — |
| Touche Échap (widget `Focus`) | bureau, Web | — | oui |
| Touche P | bureau, Web | oui | oui, si le focus est au jeu |
| `BoutonPause` du HUD | tactile | oui | non |
| Bouton REPRENDRE | toutes | — | oui |
| Perte de focus | toutes | oui | jamais |

> **À retenir.** Un seul `basculerPause()`, plusieurs déclencheurs. Entrer en pause peut venir de Flame ; en sortir doit toujours pouvoir venir de Flutter.

---

## 40.14 — `lib/ecrans/ecran_game_over.dart`

### Ce qu'un bon Game Over fait

Un écran de fin ne sert pas à punir. Il sert à **relancer**. Les trois règles :

1. il dit ce qui s'est passé, en un mot ;
2. il donne un chiffre à battre ;
3. il propose de recommencer en un seul geste.

Le pire écran de Game Over est celui qui renvoie au menu principal en obligeant à recliquer sur JOUER. Chaque clic supplémentaire fait perdre des joueurs.

### Le déclenchement

Il existe déjà, depuis le chapitre 37 :

```dart
  void perdreUneVie() {
    vies--;
    reinitialiserCombo();
    hud.barreDeVie.secouer();
    AudioService.instance.jouer(Effet.mort);   // AJOUT DU CHAPITRE 40

    if (vies <= 0) {
      vies = 0;
      declarerGameOver();                      // AJOUT DU CHAPITRE 40
      return;
    }
    joueur!.reapparaitre(pointDeReapparition);
  }
```

Et la méthode dédiée :

```dart
  /// Fin de partie perdue. Point de passage unique.
  void declarerGameOver() {
    dureePartie = _chronometre;                // section 40.18
    _sauvegarderProgression();                 // section 40.24
    changerEtat(GameState.gameOver);
  }
```

Pourquoi une méthode plutôt qu'un `changerEtat` direct ? Parce qu'il y a trois choses à faire, et qu'on veut être certain qu'elles se font toutes, depuis n'importe quel appelant. C'est le même raisonnement que pour `basculerPause`.

### Le widget

```dart
// lib/ecrans/ecran_game_over.dart
import 'package:flutter/material.dart';

import '../config/palette.dart';
import '../donjon_game.dart';
import '../services/audio_service.dart';
import 'ecran_pause.dart' show BoutonEcran;

/// L'écran affiché quand le joueur a perdu sa dernière vie.
class EcranGameOver extends StatelessWidget {
  const EcranGameOver({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    final bool record = jeu.score > 0 && jeu.score >= jeu.meilleurScore;

    return Material(
      // Opaque et rouge sombre : on ne joue plus.
      color: const Color(0xFF1A0C0C),
      child: Center(
        child: SingleChildScrollView(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              const Text(
                'GAME OVER',
                style: TextStyle(
                  fontSize: 48,
                  fontWeight: FontWeight.w900,
                  letterSpacing: 8,
                  color: Palette.danger,
                ),
              ),
              const SizedBox(height: 8),
              Text(
                'Le donjon vous a eu au niveau ${jeu.niveauCourant + 1}.',
                style: Palette.sousTitre,
              ),
              const SizedBox(height: 26),
              TableauScores(
                score: jeu.score,
                meilleurScore: jeu.meilleurScore,
                record: record,
              ),
              const SizedBox(height: 26),
              SizedBox(
                width: 300,
                child: Column(
                  children: <Widget>[
                    BoutonEcran(
                      libelle: 'REJOUER LE NIVEAU',
                      icone: Icons.replay,
                      principal: true,
                      onPressed: () {
                        AudioService.instance.jouer(Effet.clic);
                        jeu.rejouerLeNiveau();
                      },
                    ),
                    const SizedBox(height: 10),
                    BoutonEcran(
                      libelle: 'NOUVELLE PARTIE',
                      icone: Icons.restart_alt,
                      onPressed: () {
                        AudioService.instance.jouer(Effet.clic);
                        jeu.demarrerPartie();
                      },
                    ),
                    const SizedBox(height: 10),
                    BoutonEcran(
                      libelle: 'MENU PRINCIPAL',
                      icone: Icons.home,
                      onPressed: () {
                        AudioService.instance.jouer(Effet.clic);
                        jeu.retournerAuMenu();
                      },
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Le fond opaque

Contrairement à la pause, le Game Over est **opaque**. La raison est psychologique autant que technique : montrer le cadavre du héros gelé derrière un panneau semi-transparent est désagréable, et laisse croire que la partie continue. On coupe net.

> **À retenir.** Trois boutons, le plus utile en premier, et un fond opaque.

---

## 40.15 — Afficher le score et le meilleur score

### Le tableau des scores

```dart
/// Bloc « SCORE / MEILLEUR », partagé par le Game Over et la victoire.
class TableauScores extends StatelessWidget {
  const TableauScores({
    super.key,
    required this.score,
    required this.meilleurScore,
    required this.record,
  });

  final int score;
  final int meilleurScore;
  final bool record;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 20),
      decoration: BoxDecoration(
        color: Palette.panneau,
        border: Border.all(color: Palette.mur, width: 2),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: <Widget>[
          const Text('VOTRE SCORE', style: Palette.sousTitre),
          Text(
            '$score'.padLeft(6, '0'),
            style: const TextStyle(
              fontSize: 42,
              fontWeight: FontWeight.bold,
              color: Palette.accent,
              fontFeatures: <FontFeature>[FontFeature.tabularFigures()],
            ),
          ),
          const SizedBox(height: 12),
          const Divider(color: Palette.mur, height: 1),
          const SizedBox(height: 12),
          if (record)
            const Text(
              'NOUVEAU RECORD',
              style: TextStyle(
                color: Palette.succes,
                fontSize: 16,
                letterSpacing: 3,
                fontWeight: FontWeight.bold,
              ),
            )
          else
            Text(
              'MEILLEUR : ${'$meilleurScore'.padLeft(6, '0')}',
              style: const TextStyle(
                color: Palette.texteFaible,
                fontSize: 15,
                letterSpacing: 2,
                fontFeatures: <FontFeature>[FontFeature.tabularFigures()],
              ),
            ),
        ],
      ),
    );
  }
}
```

### `padLeft` et `tabularFigures`

Deux détails de typographie qui font une grosse différence.

`'$score'.padLeft(6, '0')` affiche `001240` au lieu de `1240`. Le chapitre 38 l'utilise déjà dans le HUD : la largeur du texte ne change plus, donc le nombre ne « saute » plus quand il grandit.

`FontFeature.tabularFigures()` demande à la police d'utiliser des chiffres de largeur **fixe**. Sans lui, dans la plupart des polices, un `1` est plus étroit qu'un `8`, et un score qui change fait vibrer tout le bloc.

`FontFeature` vient de `dart:ui`, ré-exporté par `package:flutter/material.dart` : aucun import supplémentaire.

### Quand y a-t-il record ?

```dart
    final bool record = jeu.score > 0 && jeu.score >= jeu.meilleurScore;
```

Le `>=` et non `>` peut surprendre. La raison est dans `ajouterScore` du chapitre 38 :

```dart
    if (score > meilleurScore) {
      meilleurScore = score;
    }
```

Le meilleur score est mis à jour **pendant** la partie. À la fin, si le joueur a battu son record, `score == meilleurScore`. Avec un `>` strict, l'écran n'annoncerait jamais un record. C'est un bug classique, et il est invisible tant qu'on n'a pas joué deux fois.

Le `score > 0` évite d'annoncer un record à un joueur qui meurt sans avoir ramassé une seule pièce, au tout premier lancement.

### La cohérence avec la sauvegarde

Le `meilleurScore` affiché ici est celui du jeu en mémoire. Il a été initialisé au démarrage depuis la sauvegarde (section 40.3) et mis à jour pendant la partie. La persistance, elle, se fait dans `declarerGameOver` via `_sauvegarderProgression()`.

L'ordre est important :

```text
  1. ajouterScore()      met à jour game.meilleurScore  (chapitre 38)
  2. declarerGameOver()  persiste game.meilleurScore    (chapitre 40)
  3. EcranGameOver       LIT game.meilleurScore
```

Si vous persistiez après l'affichage, un plantage entre les deux perdrait le record. Si vous affichiez la valeur relue du disque, vous afficheriez l'ancienne. On persiste, puis on affiche la valeur en mémoire.

> **À retenir.** `padLeft` plus `tabularFigures` pour l'affichage, `>=` pour le test de record.

---

## 40.16 — Rejouer le niveau ou revenir au menu

### Trois sorties, trois méthodes

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  /// Recommence le niveau courant, avec des vies neuves.
  /// Le score n'est PAS remis à zéro : le joueur garde ses points.
  Future<void> rejouerLeNiveau() async {
    changerEtat(GameState.chargement);

    vies = Constantes.viesDepart;
    reinitialiserCombo();
    if (joueur != null) {
      joueur!.pv = Constantes.pvJoueurMax;
    }
    _demarrerChronometre();

    await chargerNiveau(niveauCourant);        // chapitre 39

    if (etat != GameState.chargement) {
      return;                                  // le joueur a changé d'avis
    }
    changerEtat(GameState.enJeu);
  }
```

Le choix « le score n'est pas remis à zéro » est discutable, et c'est justement pour cela qu'il faut le documenter. Deux écoles :

| Choix | Argument | Jeux qui le font |
| --- | --- | --- |
| Le score repart de zéro | le score mesure une performance d'un bout à l'autre | jeux de score pur, arcade |
| Le score est conservé | rejouer un niveau n'est pas tricher, c'est apprendre | jeux de plateformes narratifs |

Nous prenons le second, parce que « Donjon de Dart » a trois niveaux et vise l'apprentissage. Le bouton NOUVELLE PARTIE, lui, remet tout à zéro : le joueur qui veut un score « propre » l'utilise.

### `demarrerPartie` révisée

```dart
  /// Démarre une partie, éventuellement à un niveau donné.
  Future<void> demarrerPartie({int niveau = 0}) async {
    changerEtat(GameState.chargement);

    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = niveau.clamp(0, Constantes.nombreNiveaux - 1);
    reinitialiserCombo();
    journalPartie.clear();                     // section 40.18
    _demarrerChronometre();

    await chargerNiveau(niveauCourant);

    if (etat != GameState.chargement) {
      return;
    }
    changerEtat(GameState.enJeu);
  }
```

Le paramètre `niveau` sert au bouton « Continuer » du menu (section 40.27). Le `clamp` protège d'une sauvegarde corrompue qui contiendrait `niveauAtteint: 47`.

### Le piège de l'accumulation de composants

Voici le bug le plus fréquent de cette section, et il ne se voit qu'après cinq parties.

```text
  Partie 1 : 1 joueur, 12 ennemis, 30 pièces      →  43 composants
  Partie 2 : 2 joueurs, 24 ennemis, 60 pièces     →  86 composants
  Partie 3 : 3 joueurs, 36 ennemis, 90 pièces     → 129 composants
  ...
  Partie 8 : le jeu tombe à 20 images par seconde
```

La cause : `chargerNiveau` ajoute au monde sans avoir vidé le monde. Chaque `HudButtonComponent` en double, chaque `Joueur` fantôme continue à consommer de la mémoire et du temps de calcul, et le HUD lit le mauvais joueur.

La vérification est immédiate, avec le raccourci F2 du chapitre 35 :

```text
[DonjonGame] etat=En jeu score=0 vies=3 niveau=0 composants=43 overlays=[hud]
```

Relancez trois parties. Si `composants` grandit, votre `chargerNiveau` ne vide pas. Le chapitre 39 le fait ; ce rappel vous évitera de casser cette garantie en modifiant le code.

### Retourner au menu

`retournerAuMenu()` existe depuis le chapitre 35. On lui ajoute la sauvegarde :

```dart
  void retournerAuMenu() {
    _sauvegarderProgression();                 // AJOUT DU CHAPITRE 40
    changerEtat(GameState.menu);
  }
```

> **À retenir.** Rejouer recharge le niveau et rend les vies ; nouvelle partie remet tout à zéro. Dans les deux cas, on vide le monde d'abord.

---

## 40.17 — `lib/ecrans/ecran_victoire.dart`

### Le déclenchement

Le chapitre 39 a écrit `terminerNiveau()`. Il ne lui manque que la branche finale :

```dart
  /// Le joueur a franchi la porte du dernier niveau.
  void declarerVictoire() {
    dureePartie = _chronometre;
    statistiques = StatistiquesPartie.calculer(   // section 40.19
      journalPartie,
      duree: dureePartie,
      score: score,
      meilleurScore: meilleurScore,
    );
    _sauvegarderProgression();
    unawaited(
      SauvegardeService.instance.enregistrerProgression(
        Constantes.nombreNiveaux,                 // campagne terminée
      ),
    );
    changerEtat(GameState.victoire);
  }
```

Notez le `Constantes.nombreNiveaux` passé à `enregistrerProgression` : la progression enregistrée vaut 3 sur un jeu de 3 niveaux, ce qui signifie « campagne terminée ». Le menu s'en servira pour proposer autre chose que « Continuer » (section 40.27).

### Le widget

```dart
// lib/ecrans/ecran_victoire.dart
import 'package:flutter/material.dart';

import '../config/palette.dart';
import '../core/statistiques.dart';
import '../donjon_game.dart';
import '../services/audio_service.dart';
import 'ecran_game_over.dart' show TableauScores;
import 'ecran_pause.dart' show BoutonEcran;

/// L'écran affiché quand le joueur termine le dernier niveau.
class EcranVictoire extends StatelessWidget {
  const EcranVictoire({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    final StatistiquesPartie stats = jeu.statistiques;

    return Material(
      color: const Color(0xFF0E1A12),
      child: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.symmetric(vertical: 24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              const Text(
                'VICTOIRE',
                style: TextStyle(
                  fontSize: 48,
                  fontWeight: FontWeight.w900,
                  letterSpacing: 10,
                  color: Palette.succes,
                ),
              ),
              const SizedBox(height: 6),
              const Text(
                'LE DONJON EST NETTOYÉ',
                style: Palette.sousTitre,
              ),
              const SizedBox(height: 22),
              TableauScores(
                score: jeu.score,
                meilleurScore: jeu.meilleurScore,
                record: jeu.score >= jeu.meilleurScore,
              ),
              const SizedBox(height: 18),
              TableauStatistiques(stats: stats),   // section 40.18
              const SizedBox(height: 22),
              SizedBox(
                width: 300,
                child: Column(
                  children: <Widget>[
                    BoutonEcran(
                      libelle: 'REJOUER',
                      icone: Icons.restart_alt,
                      principal: true,
                      onPressed: () {
                        AudioService.instance.jouer(Effet.clic);
                        jeu.demarrerPartie();
                      },
                    ),
                    const SizedBox(height: 10),
                    BoutonEcran(
                      libelle: 'MENU PRINCIPAL',
                      icone: Icons.home,
                      onPressed: () {
                        AudioService.instance.jouer(Effet.clic);
                        jeu.retournerAuMenu();
                      },
                    ),
                  ],
                ),
              ),
              const SizedBox(height: 16),
              const Text(
                'Merci d\'avoir joué — Donjon de Dart, chapitre 40',
                style: TextStyle(color: Palette.texteFaible, fontSize: 11),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Pourquoi pas de « niveau suivant »

Parce qu'il n'y en a pas. Un bouton désactivé « NIVEAU SUIVANT » serait une frustration gratuite. Quand vous ajouterez des niveaux (exercice 8), c'est `Constantes.nombreNiveaux` qui changera, pas cet écran.

> **À retenir.** L'écran de victoire réutilise `TableauScores` et ajoute les statistiques. Aucun code dupliqué entre les deux écrans de fin.

---

## 40.18 — Les statistiques de fin de partie (temps, pièces, ennemis vaincus)

### Le mauvais réflexe : un compteur par statistique

```dart
// À NE PAS FAIRE
int nbPiecesRamassees = 0;
int nbEnnemisVaincus = 0;
int nbPotionsBues = 0;
int nbDegatsSubis = 0;
int nbClesRamassees = 0;
// ... et six lignes de remise à zéro dans demarrerPartie
```

Chaque nouvelle statistique demande : un champ, une incrémentation, une remise à zéro, une ligne d'affichage. Quatre endroits à ne pas oublier. Vous en oublierez un — probablement la remise à zéro, et vos statistiques cumuleront d'une partie à l'autre.

### Le bon réflexe : un journal d'événements

Une seule liste. On y ajoute ce qui se passe. On calcule à la fin.

```dart
// lib/core/statistiques.dart

/// Ce qui peut arriver de notable pendant une partie.
enum TypeEvenement {
  pieceRamassee,
  potionBue,
  cleRamassee,
  ennemiVaincu,
  bossVaincu,
  degatSubi,
  viePerdue,
  niveauTermine,
}

/// Un fait de jeu, daté et situé.
class EvenementPartie {
  const EvenementPartie({
    required this.type,
    required this.valeur,
    required this.instant,
    required this.niveau,
  });

  final TypeEvenement type;

  /// Grandeur associée : points, PV, index. 1 quand elle n'a pas de sens.
  final int valeur;

  /// Secondes écoulées depuis le début de la partie.
  final double instant;

  /// Index du niveau où l'événement s'est produit.
  final int niveau;
}
```

Quatre champs, une seule liste, et une propriété précieuse : **le journal est une trace**. Vous pouvez l'afficher pour déboguer, le rejouer, en tirer un graphique de progression, ou le sauvegarder. Aucun compteur ne vous donne cela.

### L'enregistrement

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 40

  /// Journal de la partie en cours. Vidé par `demarrerPartie`.
  final List<EvenementPartie> journalPartie = <EvenementPartie>[];

  /// Chronomètre de la partie, en secondes.
  double _chronometre = 0;
  double dureePartie = 0;

  /// Statistiques de la dernière partie terminée.
  StatistiquesPartie statistiques = StatistiquesPartie.vide();

  void _demarrerChronometre() {
    _chronometre = 0;
    dureePartie = 0;
  }

  /// Consigne un fait de jeu. Appelée par les composants.
  void enregistrerEvenement(TypeEvenement type, [int valeur = 1]) {
    journalPartie.add(
      EvenementPartie(
        type: type,
        valeur: valeur,
        instant: _chronometre,
        niveau: niveauCourant,
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);

    // Le chronomètre n'avance QUE pendant le jeu.
    // `update` n'est pas appelé quand le moteur est en pause, mais on
    // garde le test : il documente l'intention et protège d'un futur
    // changement de conception.
    if (etat == GameState.enJeu) {
      _chronometre += dt;
    }

    // ... la fenêtre de combo du chapitre 38
  }
```

Le chronomètre s'incrémente avec `dt`, jamais avec `DateTime.now()`. C'est la règle du chapitre 20 : un temps de jeu doit être un temps **de jeu**, pas un temps de montre. Les cinq minutes passées dans l'écran de pause ne comptent pas.

### Les points d'appel

| Événement | Fichier | Emplacement |
| --- | --- | --- |
| `pieceRamassee` | `piece.dart` | dans `ramasser`, avec la valeur en points |
| `potionBue` | `potion.dart` | dans `ramasser`, avec les PV rendus |
| `cleRamassee` | `cle.dart` | dans `ramasser` |
| `ennemiVaincu` | `ennemi.dart` | dans `mourir`, avec `pointsScore` |
| `bossVaincu` | `boss.dart` | dans `mourir` |
| `degatSubi` | `joueur.dart` | dans `subirDegats`, avec les dégâts arrondis |
| `viePerdue` | `donjon_game.dart` | dans `perdreUneVie` |
| `niveauTermine` | `donjon_game.dart` | dans `terminerNiveau`, avec l'index |

Une seule ligne à chaque endroit :

```dart
    game.enregistrerEvenement(TypeEvenement.ennemiVaincu, pointsScore);
```

### La classe `StatistiquesPartie`

```dart
/// Résumé chiffré d'une partie, calculé à partir du journal.
class StatistiquesPartie {
  const StatistiquesPartie({
    required this.duree,
    required this.score,
    required this.meilleurScore,
    required this.pieces,
    required this.pointsDesPieces,
    required this.potions,
    required this.cles,
    required this.ennemisVaincus,
    required this.bossVaincus,
    required this.degatsSubis,
    required this.viesPerdues,
    required this.niveauxTermines,
  });

  factory StatistiquesPartie.vide() => const StatistiquesPartie(
        duree: 0, score: 0, meilleurScore: 0,
        pieces: 0, pointsDesPieces: 0, potions: 0, cles: 0,
        ennemisVaincus: 0, bossVaincus: 0,
        degatsSubis: 0, viesPerdues: 0, niveauxTermines: 0,
      );

  final double duree;
  final int score;
  final int meilleurScore;
  final int pieces;
  final int pointsDesPieces;
  final int potions;
  final int cles;
  final int ennemisVaincus;
  final int bossVaincus;
  final int degatsSubis;
  final int viesPerdues;
  final int niveauxTermines;

  /// Durée formatée en `m:ss`.
  String get dureeFormatee {
    final int total = duree.round();
    final int minutes = total ~/ 60;
    final String secondes = (total % 60).toString().padLeft(2, '0');
    return '$minutes:$secondes';
  }

  /// Note de fin, purement indicative.
  String get rang {
    if (viesPerdues == 0 && duree < 180) return 'S';
    if (viesPerdues <= 1) return 'A';
    if (viesPerdues <= 3) return 'B';
    return 'C';
  }
}
```

### Le tableau d'affichage

```dart
/// Affichage à deux colonnes des statistiques de fin de partie.
class TableauStatistiques extends StatelessWidget {
  const TableauStatistiques({super.key, required this.stats});

  final StatistiquesPartie stats;

  @override
  Widget build(BuildContext context) {
    final List<(String, String)> lignes = <(String, String)>[
      ('Temps', stats.dureeFormatee),
      ('Pièces ramassées', '${stats.pieces}'),
      ('Ennemis vaincus', '${stats.ennemisVaincus}'),
      ('Boss vaincus', '${stats.bossVaincus}'),
      ('Potions bues', '${stats.potions}'),
      ('Dégâts subis', '${stats.degatsSubis}'),
      ('Vies perdues', '${stats.viesPerdues}'),
      ('Rang', stats.rang),
    ];

    return Container(
      width: 320,
      padding: const EdgeInsets.symmetric(horizontal: 22, vertical: 16),
      decoration: BoxDecoration(
        color: Palette.panneau.withValues(alpha: 0.7),
        border: Border.all(color: Palette.mur, width: 2),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: <Widget>[
          for (final (String libelle, String valeur) in lignes)
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 3),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: <Widget>[
                  Text(
                    libelle,
                    style: const TextStyle(
                      color: Palette.texteFaible,
                      fontSize: 13,
                    ),
                  ),
                  Text(
                    valeur,
                    style: const TextStyle(
                      color: Palette.texte,
                      fontSize: 13,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
        ],
      ),
    );
  }
}
```

Le type `(String, String)` est un **enregistrement** (*record*) de Dart 3 : un couple anonyme, sans classe à écrire. La déstructuration `for (final (String libelle, String valeur) in lignes)` en extrait les deux membres directement.

> **À retenir.** Un journal d'événements plutôt que dix compteurs : une seule remise à zéro, et des statistiques qu'on peut ajouter après coup sans toucher au jeu.

---

## 40.19 — Calculer ces statistiques avec `where`, `map`, `fold` (rappel chapitre 14)

### La fabrique `calculer`

C'est la section la plus « Dart » du chapitre. Tout le calcul tient en une méthode, sans une seule boucle `for`.

```dart
  /// Calcule les statistiques à partir du journal d'une partie.
  factory StatistiquesPartie.calculer(
    List<EvenementPartie> journal, {
    required double duree,
    required int score,
    required int meilleurScore,
  }) {
    // COMBIEN de fois un type est-il apparu ?
    int compter(TypeEvenement type) =>
        journal.where((EvenementPartie e) => e.type == type).length;

    // SOMME des valeurs associées à un type.
    int cumuler(TypeEvenement type) => journal
        .where((EvenementPartie e) => e.type == type)
        .map((EvenementPartie e) => e.valeur)
        .fold<int>(0, (int total, int v) => total + v);

    return StatistiquesPartie(
      duree: duree,
      score: score,
      meilleurScore: meilleurScore,
      pieces: compter(TypeEvenement.pieceRamassee),
      pointsDesPieces: cumuler(TypeEvenement.pieceRamassee),
      potions: compter(TypeEvenement.potionBue),
      cles: compter(TypeEvenement.cleRamassee),
      ennemisVaincus: compter(TypeEvenement.ennemiVaincu),
      bossVaincus: compter(TypeEvenement.bossVaincu),
      degatsSubis: cumuler(TypeEvenement.degatSubi),
      viesPerdues: compter(TypeEvenement.viePerdue),
      niveauxTermines: compter(TypeEvenement.niveauTermine),
    );
  }
```

### Les trois opérations, une par une

Le chapitre 14 les a introduites ; voici ce qu'elles font ici, sur un journal de six événements.

```text
  journal (6 événements)
  ┌───────────────────────────────────────────────────────────────┐
  │ pieceRamassee  v=10  t=2.1  n=0                               │
  │ ennemiVaincu   v=50  t=5.4  n=0                               │
  │ pieceRamassee  v=10  t=6.0  n=0                               │
  │ degatSubi      v=20  t=8.2  n=0                               │
  │ pieceRamassee  v=25  t=9.9  n=0                               │
  │ viePerdue      v=1   t=12.5 n=0                               │
  └───────────────────────────────────────────────────────────────┘

  .where((e) => e.type == pieceRamassee)
  ┌───────────────────────────────────────────────────────────────┐
  │ pieceRamassee v=10 │ pieceRamassee v=10 │ pieceRamassee v=25   │
  └───────────────────────────────────────────────────────────────┘
                       .length  ──────────────────────────────►  3

  .map((e) => e.valeur)
  ┌───────────────────────────────────────────────────────────────┐
  │        10          │         10         │         25          │
  └───────────────────────────────────────────────────────────────┘

  .fold<int>(0, (total, v) => total + v)
      0 + 10 = 10 ; 10 + 10 = 20 ; 20 + 25 = 45  ────────────────►  45
```

`where` **filtre** sans changer le type. `map` **transforme** et change le type (`EvenementPartie` devient `int`). `fold` **réduit** une collection à une seule valeur, en partant d'une graine.

### Pourquoi `fold<int>(0, ...)` et pas `reduce`

`reduce` existe aussi, et paraît plus court :

```dart
    journal.map((e) => e.valeur).reduce((a, b) => a + b);   // DANGEREUX
```

Mais `reduce` sur une collection **vide** lève une `StateError` (« No element »). Et une collection d'événements est vide au tout premier écran de victoire d'un joueur qui n'aurait subi aucun dégât.

`fold` prend une valeur initiale : sur une collection vide, il renvoie cette valeur. Zéro dégât subi affiche `0`, pas un plantage.

C'est exactement la distinction du chapitre 14, et c'est ici qu'elle coûte cher si on l'ignore.

### La paresse des itérables

```dart
final Iterable<EvenementPartie> pieces =
    journal.where((EvenementPartie e) => e.type == TypeEvenement.pieceRamassee);
```

`where` ne construit **aucune liste**. Il renvoie un `Iterable` paresseux, qui ne fait le travail qu'au moment où on le parcourt. Chaîner `where().map().fold()` ne crée donc pas deux listes intermédiaires : le journal n'est parcouru qu'une fois.

Conséquence pratique : appeler `compter()` neuf fois parcourt le journal neuf fois. Sur cent événements, c'est neuf cents comparaisons — soit une fraction de milliseconde, une seule fois par partie, sur un écran de fin. C'est parfaitement acceptable, et la lisibilité vaut mieux ici que la micro-optimisation.

Si vous vouliez malgré tout un seul passage, `fold` sait le faire :

```dart
    final Map<TypeEvenement, int> occurrences =
        journal.fold<Map<TypeEvenement, int>>(
      <TypeEvenement, int>{},
      (Map<TypeEvenement, int> acc, EvenementPartie e) {
        acc[e.type] = (acc[e.type] ?? 0) + 1;
        return acc;
      },
    );
```

C'est l'objet de l'exercice 6.

### Quelques statistiques dérivées

Une fois le journal en main, les statistiques « intelligentes » ne coûtent presque rien.

```dart
  /// Points par minute. Évite la division par zéro.
  double get scoreParMinute => duree < 1 ? 0 : score * 60 / duree;

  /// Instant du premier événement d'un type donné, ou null.
  static double? premierInstant(
    List<EvenementPartie> journal,
    TypeEvenement type,
  ) {
    final Iterable<EvenementPartie> filtres =
        journal.where((EvenementPartie e) => e.type == type);
    return filtres.isEmpty ? null : filtres.first.instant;
  }

  /// Le niveau où le joueur a le plus souffert.
  static int? niveauLePlusDur(List<EvenementPartie> journal) {
    final Map<int, int> parNiveau = <int, int>{};
    for (final EvenementPartie e
        in journal.where((EvenementPartie e) =>
            e.type == TypeEvenement.viePerdue)) {
      parNiveau[e.niveau] = (parNiveau[e.niveau] ?? 0) + 1;
    }
    if (parNiveau.isEmpty) {
      return null;
    }
    return parNiveau.entries
        .reduce((MapEntry<int, int> a, MapEntry<int, int> b) =>
            a.value >= b.value ? a : b)
        .key;
  }
```

Ici `reduce` est sûr : on a testé `isEmpty` juste avant.

> **À retenir.** `where` filtre, `map` transforme, `fold` réduit avec une graine. Sur une collection potentiellement vide, `fold` et jamais `reduce`.

---

## 40.20 — La persistance : pourquoi `shared_preferences`

### Le besoin, chiffré

Ce que « Donjon de Dart » doit conserver entre deux lancements :

| Donnée | Type | Taille |
| --- | --- | --- |
| Meilleur score | `int` | 4 octets |
| Niveau atteint | `int` | 4 octets |
| Parties jouées | `int` | 4 octets |
| Volumes, interrupteurs, difficulté | 5 valeurs | quelques octets |
| Touches personnalisées | 5 chaînes | quelques dizaines d'octets |

Total : **moins de 500 octets**. Aucune requête, aucune relation, aucune recherche. C'est le cas d'usage exact d'un magasin clé-valeur.

### Les quatre solutions, et pourquoi trois sont écartées

| Solution | Verdict | Raison |
| --- | --- | --- |
| Fichier via `dart:io` (`File`) | **écarté** | `dart:io` n'existe pas sur le Web. Notre jeu vise le Web en priorité (cahier des charges, chapitre 35). |
| `sqflite` (SQLite) | **écarté** | une base relationnelle pour trois entiers ; pas de support Web sans greffon. |
| `hive` / base NoSQL locale | **écarté** | excellent, mais il faut générer des adaptateurs et apprendre une API entière pour 500 octets. |
| `shared_preferences` | **retenu** | officiel (équipe Flutter), toutes plateformes y compris le Web, API de dix méthodes. |

### Ce que `shared_preferences` est vraiment

C'est une **façade unique** sur le magasin clé-valeur natif de chaque plateforme :

```text
  votre code Dart
        │
  shared_preferences
        │
  ┌─────┼─────┬───────────┬─────────┬──────────┐
  ▼     ▼     ▼           ▼         ▼          ▼
Android iOS  macOS      Linux    Windows      Web
Shared  User  User    fichier   fichier   localStorage
Prefs  Defaults Defaults .json    .json
```

Conséquences directes, à connaître avant de l'utiliser :

1. **Ce n'est pas sécurisé.** Un utilisateur averti peut modifier les valeurs. N'y mettez jamais de mot de passe. Pour un meilleur score, c'est sans importance — et de toute façon, il triche sur sa machine.
2. **Ce n'est pas transactionnel.** Deux écritures ne sont pas atomiques entre elles.
3. **Les types sont limités** à `int`, `double`, `bool`, `String` et `List<String>`. Tout le reste passe par du JSON dans une `String` (section 40.26).
4. **Sur le Web, l'effacement des données du site efface la sauvegarde.** C'est normal, et il faut le dire au joueur si votre jeu vise le Web.

> **À retenir.** `shared_preferences` pour des réglages et des compteurs ; une vraie base seulement quand vous avez des listes à interroger.

---

## 40.21 — Ajouter la dépendance

### La commande

```text
flutter pub add shared_preferences
```

### Le résultat dans `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter

  flame: ^1.38.0
  flame_audio: ^2.12.2          # AJOUTÉ AU CHAPITRE 40
  shared_preferences: ^2.5.5    # AJOUTÉ AU CHAPITRE 40
```

Version courante vérifiée le 8 août 2026 sur `https://pub.dev/packages/shared_preferences` : **2.5.5**, publiée le 25 mars 2026. Plateformes prises en charge : Android (SDK 24+), iOS 13+, Linux, macOS 10.15+, Web, Windows.

`flame_audio: ^2.12.2` était en commentaire dans le `pubspec.yaml` du chapitre 35. Décommentez-le : c'est ce chapitre qui l'utilise enfin.

### Les deux API du paquet

C'est le point qui perd le plus de monde, parce que la majorité des tutoriels en ligne montrent l'ancienne.

| API | Statut en 2.5.5 | Forme |
| --- | --- | --- |
| `SharedPreferences.getInstance()` | **héritée**, en cours de dépréciation | cache en mémoire, lectures synchrones |
| `SharedPreferencesAsync` | **recommandée** | tout asynchrone, pas de cache |
| `SharedPreferencesWithCache` | recommandée | asynchrone avec cache explicite |

Nous utilisons **`SharedPreferencesAsync`**. Signatures réelles, vérifiées dans le dartdoc le 8 août 2026 :

```dart
SharedPreferencesAsync({SharedPreferencesOptions options = const SharedPreferencesOptions()});

Future<int?>           getInt(String key);
Future<void>           setInt(String key, int value);
Future<String?>        getString(String key);
Future<void>           setString(String key, String value);
Future<bool?>          getBool(String key);
Future<void>           setBool(String key, bool value);
Future<double?>        getDouble(String key);
Future<void>           setDouble(String key, double value);
Future<List<String>?>  getStringList(String key);
Future<void>           setStringList(String key, List<String> value);
Future<void>           remove(String key);
Future<void>           clear({Set<String>? allowList});
Future<bool>           containsKey(String key);
Future<Set<String>>    getKeys({Set<String>? allowList});
Future<Map<String, Object?>> getAll({Set<String>? allowList});
```

Notez les `?` : **toutes les lectures peuvent renvoyer `null`**, au premier lancement notamment. Le Null Safety du chapitre 12 n'est pas une formalité ici : `await prefs.getInt('score')` est un `int?`, et l'oublier est l'erreur la plus fréquente de la section.

### `WidgetsFlutterBinding.ensureInitialized()`

`shared_preferences` est un greffon de plateforme : il communique avec du code natif. Cette communication exige que la liaison Flutter existe. Le `main()` du chapitre 35 appelle déjà `WidgetsFlutterBinding.ensureInitialized()` : rien à changer.

Si vous l'oubliez, l'erreur est explicite : *« Binding has not yet been initialized »*.

> **À retenir.** `shared_preferences: ^2.5.5`, et l'API moderne `SharedPreferencesAsync`, dont toutes les lectures sont nullables.

---

## 40.22 — `lib/services/sauvegarde_service.dart`

### La forme du service

Même motif que `AudioService` : singleton, jamais d'exception vers l'extérieur, et une seule classe du projet importe `shared_preferences`.

```dart
class SauvegardeService {
  SauvegardeService._();
  static final SauvegardeService instance = SauvegardeService._();

  static const String _cleSauvegarde = 'donjon_de_dart.sauvegarde.v1';

  final SharedPreferencesAsync _prefs = SharedPreferencesAsync();

  /// Les données en mémoire. TOUJOURS valides, même avant `initialiser()`.
  Sauvegarde donnees = Sauvegarde.parDefaut();

  /// Vrai une fois la lecture du disque terminée.
  bool pret = false;

  /// Vrai si la dernière lecture a trouvé des données illisibles.
  bool sauvegardeCorrompue = false;
}
```

Le champ `donnees` est **toujours utilisable**, y compris avant l'appel à `initialiser()`. C'est une décision de conception importante : aucun appelant n'a besoin de tester `if (pret)`. Au pire, il lit les valeurs par défaut. Un jeu qui affiche `0` comme meilleur score pendant 40 millisecondes est bien préférable à un jeu qui lève une `LateInitializationError`.

### Le nom des clés

```dart
  static const String _cleSauvegarde = 'donjon_de_dart.sauvegarde.v1';
```

Trois règles :

1. **Préfixez par le nom de l'application.** Sur le Web, `localStorage` est partagé par tout le domaine : une clé `score` entrerait en collision avec un autre jeu hébergé au même endroit.
2. **Versionnez la clé** (`.v1`). Le jour où le format change de façon incompatible, `.v2` cohabite avec `.v1` sans conflit.
3. **Centralisez-les en constantes.** Une clé écrite à la main dans deux fichiers finit toujours par diverger d'une lettre.

### Le cycle de vie

```text
  DonjonGame.onLoad()
        │
        ▼
  SauvegardeService.initialiser()          ── lecture, une seule fois
        │
        ├─ succès          → donnees = Sauvegarde.fromJson(...)
        ├─ rien de stocké  → donnees = Sauvegarde.parDefaut()
        └─ illisible       → donnees = Sauvegarde.parDefaut()
                             sauvegardeCorrompue = true    (section 40.28)
        │
        ▼
  pret = true
        │
   pendant le jeu : lectures en mémoire, sans I/O
        │
        ▼
  enregistrerMeilleurScore / enregistrerProgression / enregistrerReglages
        │
        ▼
  _ecrire()  ── une seule écriture, la sauvegarde entière en JSON
```

Une seule lecture au démarrage, une écriture aux moments choisis. Jamais d'accès disque dans `update()` : ce serait soixante écritures par seconde.

### Quand écrire

| Moment | Pourquoi |
| --- | --- |
| Fin de partie (Game Over, victoire) | c'est là que le score est définitif |
| Fin de niveau | pour ne pas perdre la progression d'un joueur qui ferme l'application |
| Fermeture de l'écran d'options | les réglages sont explicites |
| Abandon vers le menu | le joueur a décidé d'arrêter |

Jamais : à chaque pièce ramassée. Une écriture coûte quelques millisecondes et use la mémoire flash.

---

## 40.23 — Sauvegarder le meilleur score

### La méthode

```dart
  /// Enregistre un nouveau meilleur score, s'il est meilleur.
  ///
  /// Ne fait rien si le score proposé est inférieur ou égal : cela évite
  /// qu'un appel malencontreux n'écrase le record.
  Future<void> enregistrerMeilleurScore(int score) async {
    if (score <= donnees.meilleurScore) {
      return;
    }
    donnees.meilleurScore = score;
    await _ecrire();
  }
```

La garde `score <= donnees.meilleurScore` est le cœur de la méthode. Sans elle, un appel avec `game.score` (et non `game.meilleurScore`) après une mauvaise partie écraserait le record par une valeur plus faible. Le service refuse structurellement de régresser.

### La lecture

Elle n'existe pas comme méthode : on lit `SauvegardeService.instance.donnees.meilleurScore`, en mémoire, gratuitement. Le jeu le fait une fois, dans `onLoad`.

### Vérifier que ça marche

La procédure de test manuel, en cinq étapes :

```text
  1. flutter run
  2. Jouer, mourir avec un score non nul. Noter le score.
  3. FERMER COMPLÈTEMENT l'application (pas un hot restart).
  4. flutter run à nouveau.
  5. Menu → JOUER → mourir immédiatement.
     L'écran Game Over doit afficher le score de l'étape 2 en MEILLEUR.
```

L'étape 3 est celle qu'on rate. Un *hot restart* garde le processus vivant, et une variable statique conserve sa valeur : vous croiriez que la sauvegarde marche alors qu'elle ne fait rien du tout.

### La commande de remise à zéro

Pendant le développement, vous voudrez repartir de zéro. Ajoutez un raccourci de debug :

```dart
      // F9 : efface la sauvegarde (mode debug uniquement).
      if (event.logicalKey == LogicalKeyboardKey.f9 && debugMode) {
        unawaited(SauvegardeService.instance.reinitialiser());
        meilleurScore = 0;
        debugPrint('[DonjonGame] Sauvegarde effacée.');
        return KeyEventResult.handled;
      }
```

---

## 40.24 — Sauvegarder la progression (niveau atteint)

### Ce que « niveau atteint » veut dire

Une ambiguïté à trancher tout de suite : `niveauAtteint = 2` signifie-t-il « j'ai fini le niveau 2 » ou « je peux commencer au niveau 2 » ?

Nous choisissons : **`niveauAtteint` est l'index du niveau le plus avancé que le joueur a le droit de commencer.** Donc :

| Situation | `niveauAtteint` |
| --- | --- |
| Premier lancement | 0 |
| A terminé le niveau 1 (index 0) | 1 |
| A terminé le niveau 2 (index 1) | 2 |
| A terminé la campagne | 3 (= `Constantes.nombreNiveaux`) |

Écrivez cette table dans un commentaire du code. Une convention non écrite est une convention qui sera violée.

### La méthode

```dart
  /// Enregistre la progression. Ne recule jamais.
  Future<void> enregistrerProgression(int niveauAtteint) async {
    final int borne = niveauAtteint.clamp(0, Constantes.nombreNiveaux);
    if (borne <= donnees.niveauAtteint) {
      return;
    }
    donnees.niveauAtteint = borne;
    await _ecrire();
  }
```

Même principe que le score : monotone croissant, et borné. Le `clamp` protège contre une valeur aberrante venue d'une sauvegarde modifiée à la main.

### Le point d'appel

Dans `terminerNiveau()`, écrit au chapitre 39 :

```dart
  Future<void> terminerNiveau() async {
    enregistrerEvenement(TypeEvenement.niveauTermine, niveauCourant);

    final int cible = niveauCourant + 1;

    // AJOUT DU CHAPITRE 40 : on débloque le niveau suivant.
    unawaited(SauvegardeService.instance.enregistrerProgression(cible));

    if (cible >= Constantes.nombreNiveaux) {
      declarerVictoire();
      return;
    }

    changerEtat(GameState.chargement);
    niveauCourant = cible;
    await chargerNiveau(niveauCourant);
    if (etat != GameState.chargement) {
      return;
    }
    changerEtat(GameState.enJeu);
  }
```

### Le compteur de parties

Deux lignes de plus, et vous pouvez répondre à « combien de fois ai-je joué ? » :

```dart
  Future<void> incrementerPartiesJouees() async {
    donnees.partiesJouees++;
    await _ecrire();
  }
```

Appelé dans `demarrerPartie`. C'est le genre de chiffre qui ne sert à rien pendant six mois, puis devient très utile le jour où vous voulez équilibrer votre jeu.

---

## 40.25 — Sauvegarder les réglages (volumes, difficulté, touches)

### La classe `Reglages`

```dart
/// Tout ce que le joueur peut régler, et qui doit lui survivre.
class Reglages {
  Reglages({
    this.volumeMusique = 0.5,
    this.volumeEffets = 0.8,
    this.musiqueActivee = true,
    this.effetsActives = true,
    this.difficulte = 1.0,
    Map<String, String>? touches,
  }) : touches = touches ?? Map<String, String>.from(touchesParDefaut);

  double volumeMusique;
  double volumeEffets;
  bool musiqueActivee;
  bool effetsActives;

  /// Multiplicateur appliqué aux PV et aux dégâts des ennemis (chapitre 37).
  double difficulte;

  /// Action logique -> nom de touche. Voir section « touches ».
  final Map<String, String> touches;

  static const Map<String, String> touchesParDefaut = <String, String>{
    'gauche': 'a',
    'droite': 'd',
    'saut': 'space',
    'attaque': 'j',
    'pause': 'escape',
  };
}
```

### Les volumes et la difficulté

Rien de neuf : deux `double` bornés et un `double` de difficulté qui alimente le champ `game.difficulte` du chapitre 37.

```dart
  /// Applique des réglages relus au démarrage.
  void appliquerReglages(Reglages r) {
    musiqueActivee = r.musiqueActivee;
    effetsActives = r.effetsActives;
    volumeMusique = r.volumeMusique;      // passe par le setter : clamp + à chaud
    volumeEffets = r.volumeEffets;
  }
```

Cette méthode d'`AudioService` est celle appelée dans `onLoad` (section 40.3). Elle passe par les **setters**, donc par les `clamp` : une sauvegarde contenant `volumeMusique: 8.5` est ramenée à `1.0` sans un mot.

### Les touches : pourquoi une `Map<String, String>`

On ne peut pas sauvegarder un `LogicalKeyboardKey` : ce n'est ni un nombre stable entre les versions de Flutter, ni une chaîne. On sauvegarde donc un **nom logique**, et on le retraduit au chargement.

```dart
  /// Table de traduction nom -> touche. Volontairement courte :
  /// on n'autorise que des touches sûres.
  static const Map<String, LogicalKeyboardKey> _touchesConnues =
      <String, LogicalKeyboardKey>{
    'a': LogicalKeyboardKey.keyA,
    'd': LogicalKeyboardKey.keyD,
    'j': LogicalKeyboardKey.keyJ,
    'k': LogicalKeyboardKey.keyK,
    'space': LogicalKeyboardKey.space,
    'escape': LogicalKeyboardKey.escape,
    'left': LogicalKeyboardKey.arrowLeft,
    'right': LogicalKeyboardKey.arrowRight,
    'up': LogicalKeyboardKey.arrowUp,
  };

  /// Touche associée à une action, ou la valeur par défaut si le nom
  /// enregistré est inconnu (sauvegarde d'une version antérieure).
  static LogicalKeyboardKey touchePour(Reglages r, String action) {
    final String nom = r.touches[action] ?? Reglages.touchesParDefaut[action]!;
    return _touchesConnues[nom] ??
        _touchesConnues[Reglages.touchesParDefaut[action]!]!;
  }
```

Deux `??` en cascade : nom absent, puis nom inconnu. C'est le Null Safety du chapitre 12 employé pour ce qu'il vaut — décrire une chaîne de replis, pas juste éviter un plantage.

### La difficulté, et sa conséquence sur le score

Si le joueur peut baisser la difficulté, son meilleur score n'est plus comparable. Deux réponses possibles :

1. **ne rien faire** : c'est un jeu solo, le joueur se compare à lui-même ;
2. **pondérer** : `score * difficulte` au moment d'enregistrer le record.

Nous prenons la première pour rester simple, mais nous stockons la difficulté dans la sauvegarde, ce qui laisse la porte ouverte (exercice 9).

---

## 40.26 — Sérialiser une sauvegarde complète en JSON (rappel chapitre 17)

### Pourquoi une seule chaîne JSON plutôt que douze clés

Comparons.

| Approche | Écriture | Lecture | Évolution |
| --- | --- | --- | --- |
| Une clé par valeur | 12 appels `setX` | 12 appels `getX`, 12 `??` | ajouter un champ = toucher 3 endroits |
| Une clé, du JSON | 1 appel `setString` | 1 appel `getString` + `jsonDecode` | ajouter un champ = toucher `toJson`/`fromJson` |

Le JSON gagne dès que les données forment un **objet cohérent**. Et surtout, il permet le **versionnement** : un champ `version` dans le document, et vous savez toujours à quoi vous avez affaire.

### `toJson` et `fromJson`

Le chapitre 17 a posé le motif : `Map<String, dynamic> toJson()` et un constructeur nommé `fromJson`.

```dart
  Map<String, dynamic> toJson() => <String, dynamic>{
        'version': version,
        'meilleurScore': meilleurScore,
        'niveauAtteint': niveauAtteint,
        'partiesJouees': partiesJouees,
        'reglages': reglages.toJson(),
      };

  factory Sauvegarde.fromJson(Map<String, dynamic> json) {
    return Sauvegarde(
      version: (json['version'] as num?)?.toInt() ?? 1,
      meilleurScore: (json['meilleurScore'] as num?)?.toInt() ?? 0,
      niveauAtteint: (json['niveauAtteint'] as num?)?.toInt() ?? 0,
      partiesJouees: (json['partiesJouees'] as num?)?.toInt() ?? 0,
      reglages: Reglages.fromJson(
        (json['reglages'] as Map<String, dynamic>?) ?? <String, dynamic>{},
      ),
    );
  }
```

Le motif `(json['x'] as num?)?.toInt() ?? 0` mérite d'être décortiqué, car c'est celui qui rend une désérialisation **incassable** :

```text
  json['meilleurScore']            dynamic  (peut être absent, null, "abc", 12.0)
  as num?                          échoue proprement si ce n'est pas un nombre
  ?.toInt()                        null si la valeur était null
  ?? 0                             valeur par défaut
```

Pourquoi `num?` et pas `int?` ? Parce que le JSON ne distingue pas les entiers des réels : `jsonDecode('{"a": 5}')` donne bien un `int`, mais `jsonDecode('{"a": 5.0}')` donne un `double`. Un cast direct `as int?` sur `5.0` lève une `TypeError`. `num` couvre les deux, et `.toInt()` tranche.

### Le document produit

```json
{
  "version": 1,
  "meilleurScore": 4820,
  "niveauAtteint": 2,
  "partiesJouees": 17,
  "reglages": {
    "volumeMusique": 0.35,
    "volumeEffets": 0.8,
    "musiqueActivee": true,
    "effetsActives": true,
    "difficulte": 1.0,
    "touches": {
      "gauche": "a",
      "droite": "d",
      "saut": "space",
      "attaque": "j",
      "pause": "escape"
    }
  }
}
```

Moins de 400 octets. Lisible par un humain, ce qui est très pratique pour déboguer : le raccourci F3 de l'exercice 4 l'affiche dans la console.

### L'écriture et la lecture

```dart
  Future<void> _ecrire() async {
    try {
      await _prefs.setString(_cleSauvegarde, jsonEncode(donnees.toJson()));
    } catch (erreur) {
      debugPrint('[Sauvegarde] Écriture impossible : $erreur');
    }
  }
```

`jsonEncode` et `jsonDecode` viennent de `dart:convert` (chapitre 17). Une écriture qui échoue — disque plein, mode privé du navigateur — ne doit pas interrompre le jeu.

### La migration de version

```dart
  /// Adapte un document ancien au format courant.
  static Map<String, dynamic> migrer(Map<String, dynamic> json) {
    final int version = (json['version'] as num?)?.toInt() ?? 1;

    // Exemple : la v1 n'avait pas de champ `partiesJouees`.
    if (version < 2) {
      json['partiesJouees'] = json['partiesJouees'] ?? 0;
      json['version'] = 2;
    }
    return json;
  }
```

Ce code ne sert à rien aujourd'hui — la version courante est 1. Il sert le jour où vous publierez une mise à jour. Écrivez la fonction maintenant, vide : le jour venu, vous saurez où mettre le code, et vous ne casserez pas les sauvegardes de vos joueurs.

---

## 40.27 — Charger au démarrage et le bouton Continuer du menu (chapitre 35)

### `initialiser()`

```dart
  /// Lit la sauvegarde. Ne lève jamais. À appeler une seule fois.
  Future<void> initialiser() async {
    try {
      final String? brut = await _prefs.getString(_cleSauvegarde);

      if (brut == null || brut.isEmpty) {
        donnees = Sauvegarde.parDefaut();      // premier lancement
      } else {
        final Object? decode = jsonDecode(brut);
        if (decode is! Map<String, dynamic>) {
          throw const FormatException('La sauvegarde n\'est pas un objet.');
        }
        donnees = Sauvegarde.fromJson(Sauvegarde.migrer(decode));
        sauvegardeCorrompue = false;
      }
    } catch (erreur) {
      // Section 40.28.
      donnees = Sauvegarde.parDefaut();
      sauvegardeCorrompue = true;
      debugPrint('[Sauvegarde] Illisible, valeurs par défaut. ($erreur)');
    } finally {
      pret = true;
    }
  }
```

Le `finally` garantit `pret = true` quoi qu'il arrive. Un service qui reste « en cours d'initialisation » pour toujours est pire qu'un service en erreur.

### Le bouton Continuer, enfin actif

Le chapitre 35 avait laissé un bouchon explicite :

```dart
  /// PROVISOIRE (chapitre 35) : toujours `false`.
  bool get _peutContinuer => false;
```

On le remplace :

```dart
// lib/ecrans/menu_principal.dart — MODIFIÉ AU CHAPITRE 40

  /// Une progression existe-t-elle ?
  bool get _peutContinuer {
    final Sauvegarde s = SauvegardeService.instance.donnees;
    return s.niveauAtteint > 0 && s.niveauAtteint < Constantes.nombreNiveaux;
  }

  /// Index du niveau proposé par « Continuer ».
  int get _niveauDeReprise =>
      SauvegardeService.instance.donnees.niveauAtteint;
```

Le `< Constantes.nombreNiveaux` est important : un joueur qui a terminé la campagne n'a rien à « continuer ». Le bouton se désactive alors de lui-même, et le libellé change :

```dart
  BoutonMenu(
    libelle: 'CONTINUER',
    icone: Icons.history,
    onPressed: _peutContinuer
        ? () async {
            AudioService.instance.jouer(Effet.clic);
            await jeu.demarrerPartie(niveau: _niveauDeReprise);
          }
        : null,
    infoBulle: _peutContinuer
        ? 'Reprendre au niveau ${_niveauDeReprise + 1}'
        : 'Aucune progression enregistrée',
  ),
```

### Le problème du menu qui ne se rafraîchit pas

`MenuPrincipal` est un `StatelessWidget`. Il est reconstruit chaque fois que l'overlay est ajouté — donc à chaque retour au menu. Le bouton Continuer reflète donc toujours l'état de la sauvegarde.

Mais au **tout premier affichage**, `initialiser()` a-t-il fini ? Oui : il est `await` dans `onLoad()`, et Flame n'affiche l'overlay du menu qu'après `onLoad`. Pendant ce temps, le `loadingBuilder` du `GameWidget` est visible (chapitre 35, section 35.25). L'enchaînement est correct par construction.

Si vous déplacez un jour `initialiser()` hors de `onLoad`, il faudra un `FutureBuilder`. C'est exactement le genre de dépendance qu'il faut noter en commentaire.

### Le rappel de progression

Un détail qui plaît beaucoup aux joueurs :

```dart
  if (SauvegardeService.instance.donnees.partiesJouees > 0)
    Text(
      'Meilleur score : ${SauvegardeService.instance.donnees.meilleurScore}'
      '   ·   ${SauvegardeService.instance.donnees.partiesJouees} parties',
      style: const TextStyle(color: Palette.texteFaible, fontSize: 12),
    ),
```

---

## 40.28 — Gérer une sauvegarde corrompue (rappel chapitre 13)

### Comment une sauvegarde se corrompt

Ce n'est pas rare, et ce n'est presque jamais de votre faute.

| Cause | Fréquence |
| --- | --- |
| Le joueur a modifié le `localStorage` du navigateur | occasionnel |
| Une version antérieure de votre jeu écrivait un autre format | **fréquent** |
| L'application a été tuée pendant l'écriture | rare |
| Une extension de navigateur a tronqué la valeur | rare |
| Vous avez changé un nom de champ sans y penser | **très fréquent** |

### Les exceptions à attendre

```dart
  try {
    donnees = Sauvegarde.fromJson(jsonDecode(brut) as Map<String, dynamic>);
  } on FormatException catch (e) {
    // jsonDecode : la chaîne n'est pas du JSON valide.
  } on TypeError catch (e) {
    // Un `as` a échoué : le JSON est valide mais le type ne correspond pas.
  } catch (e) {
    // Tout le reste.
  }
```

Le chapitre 13 a insisté sur ce point : `FormatException` et `TypeError` ne sont pas de la même famille (`TypeError` descend de `Error`, pas d'`Exception`). Une clause `on Exception` seule ne les attrape pas toutes. C'est pour cela que le `catch (erreur)` nu du `initialiser()` de la section précédente est ici le bon outil.

### La stratégie : repartir, pas planter

Trois comportements possibles face à une sauvegarde illisible :

| Stratégie | Effet | Verdict |
| --- | --- | --- |
| Laisser l'exception remonter | l'application ne démarre pas | **inacceptable** |
| Effacer la sauvegarde immédiatement | le joueur perd tout sans savoir pourquoi | brutal |
| Repartir sur les valeurs par défaut, garder l'ancienne, prévenir | rien n'est perdu, le joueur est informé | **retenu** |

```dart
  Future<void> reinitialiser() async {
    donnees = Sauvegarde.parDefaut();
    sauvegardeCorrompue = false;
    try {
      await _prefs.remove(_cleSauvegarde);
    } catch (erreur) {
      debugPrint('[Sauvegarde] Effacement impossible : $erreur');
    }
  }
```

Notez ce que `initialiser()` **ne fait pas** : il n'appelle pas `reinitialiser()`. La donnée illisible reste sur le disque. Tant que le joueur ne provoque pas d'écriture, elle est récupérable — et vous pouvez l'inspecter pour comprendre. C'est la première écriture réussie qui l'écrasera.

### Prévenir le joueur

```dart
// lib/ecrans/menu_principal.dart — AJOUT DU CHAPITRE 40

if (SauvegardeService.instance.sauvegardeCorrompue)
  Container(
    margin: const EdgeInsets.only(top: 14),
    padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 8),
    decoration: BoxDecoration(
      border: Border.all(color: Palette.danger, width: 1),
      borderRadius: BorderRadius.circular(4),
    ),
    child: const Text(
      'Sauvegarde illisible : le jeu est reparti des réglages par défaut.',
      style: TextStyle(color: Palette.danger, fontSize: 11),
    ),
  ),
```

### Tester le cas, à la main

Ne comptez pas sur le hasard pour vérifier ce code. Provoquez la panne.

```dart
  /// Écrit volontairement une sauvegarde invalide. MODE DEBUG UNIQUEMENT.
  Future<void> corrompreVolontairement() async {
    await _prefs.setString(_cleSauvegarde, '{ ceci n\'est pas du JSON');
  }
```

Appelez-la depuis un raccourci de debug, redémarrez, et vérifiez que le menu s'affiche avec le bandeau rouge. Un chemin d'erreur non testé est un chemin d'erreur qui ne marche pas.

> **À retenir.** Une sauvegarde illisible ne doit jamais empêcher le jeu de démarrer : valeurs par défaut, drapeau, message, et surtout on n'efface pas tout de suite.

---

## 40.29 — L'écran Options

### Un seul écran, deux points d'entrée

Le menu principal et l'écran de pause ouvrent **le même widget**. Écrire deux écrans d'options est le meilleur moyen d'en avoir un qui persiste et un qui ne persiste pas.

```dart
// lib/ecrans/ecran_options.dart — extrait
class EcranOptions extends StatefulWidget {
  const EcranOptions({super.key, required this.jeu});
  final DonjonGame jeu;

  @override
  State<EcranOptions> createState() => _EcranOptionsState();
}
```

### La règle : appliquer tout de suite, enregistrer à la fermeture

```text
  onChanged      → on modifie l'état local ET AudioService (effet immédiat)
  onChangeEnd    → son témoin pour les effets
  FERMER         → SauvegardeService.enregistrerReglages(...)
```

Appliquer immédiatement, c'est ce qui permet au joueur d'entendre ce qu'il règle. Enregistrer à la fermeture, c'est ce qui évite quarante écritures disque pendant le glissement d'un curseur.

### Le contenu

| Réglage | Widget | Effet immédiat |
| --- | --- | --- |
| Volume musique | `Slider` | `AudioService.volumeMusique` (à chaud) |
| Volume effets | `Slider` + son témoin | `AudioService.volumeEffets` |
| Musique activée | `SwitchListTile` | `basculerMusique()` |
| Effets activés | `SwitchListTile` | `basculerEffets()` |
| Difficulté | `SegmentedButton` | `jeu.difficulte` |
| Mode debug | `SwitchListTile` | `jeu.debugMode` |
| Effacer la sauvegarde | `TextButton` rouge | confirmation puis `reinitialiser()` |

### Le bouton dangereux

```dart
  void _confirmerEffacement(BuildContext context) {
    showDialog<void>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        backgroundColor: Palette.panneau,
        title: const Text('Effacer la sauvegarde ?'),
        content: const Text(
          'Le meilleur score, la progression et les réglages seront perdus. '
          'Cette action est définitive.',
          style: TextStyle(color: Palette.texteFaible),
        ),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(c).pop(),
            child: const Text('ANNULER'),
          ),
          FilledButton(
            style: FilledButton.styleFrom(backgroundColor: Palette.danger),
            onPressed: () async {
              await SauvegardeService.instance.reinitialiser();
              widget.jeu.meilleurScore = 0;
              if (c.mounted) {
                Navigator.of(c).pop();
              }
              if (context.mounted) {
                setState(() {});
              }
            },
            child: const Text('EFFACER'),
          ),
        ],
      ),
    );
  }
```

Le `if (c.mounted)` et le `if (context.mounted)` sont obligatoires : entre l'`await` et l'utilisation du `BuildContext`, le widget a pu être démonté. L'analyseur Dart signale ce cas (`use_build_context_synchronously`) et il a raison : c'est un plantage fréquent en production.

### La difficulté avec `SegmentedButton`

```dart
  SegmentedButton<double>(
    segments: const <ButtonSegment<double>>[
      ButtonSegment<double>(value: 0.75, label: Text('FACILE')),
      ButtonSegment<double>(value: 1.0, label: Text('NORMAL')),
      ButtonSegment<double>(value: 1.4, label: Text('DIFFICILE')),
    ],
    selected: <double>{_difficulte},
    onSelectionChanged: (Set<double> choix) {
      setState(() => _difficulte = choix.first);
      widget.jeu.difficulte = _difficulte;
    },
  ),
```

Le champ `difficulte` de `DonjonGame` vient du chapitre 37 : chaque ennemi créé multiplie ses PV par `difficulte`. Le changement ne prend donc effet qu'au **prochain niveau chargé**, ce qu'il faut écrire à l'écran :

```dart
  const Text(
    'La difficulté s\'applique au prochain niveau.',
    style: TextStyle(color: Palette.texteFaible, fontSize: 11),
  ),
```

Un réglage dont l'effet est différé et qui ne le dit pas passe pour un réglage cassé.

---

## 40.30 — Le polish final : liste de vérification

Le *polish* est le travail des dernières heures, celui qui ne change aucune mécanique et qui change tout. Voici la liste que j'utilise, dans l'ordre.

### 1. Aucun cul-de-sac

| Vérification | Comment tester |
| --- | --- |
| Chaque écran a une sortie | ouvrir chaque écran, chercher un bouton de sortie |
| Le retour arrière Android ne quitte pas brutalement | appuyer sur Retour en jeu |
| Aucun bouton ne mène à un écran noir | cliquer sur tout, deux fois |

### 2. Aucun état incohérent

| Vérification | Symptôme si c'est faux |
| --- | --- |
| Le monde est vidé entre deux parties | le nombre de composants grandit (F2) |
| Le HUD est retiré au menu | une barre de vie sur le menu |
| Le combo est remis à zéro | un multiplicateur x4 au démarrage |
| Le chronomètre repart de zéro | des statistiques absurdes |
| `joueur` vaut `null` au menu | plantage à l'affichage |

### 3. Le son

- couper la musique et vérifier qu'elle ne revient pas toute seule ;
- mettre en pause : la musique doit se taire ;
- passer l'application en arrière-plan : la musique doit se taire ;
- régler le volume des effets pendant qu'un son joue.

### 4. La sauvegarde

- fermer complètement l'application et relancer : le record est là ;
- effacer la sauvegarde et relancer : aucun plantage ;
- corrompre volontairement la sauvegarde : le bandeau s'affiche.

### 5. Les performances

Le compteur d'images de Flame, en une ligne :

```dart
    // Dans DonjonGame.onLoad(), en mode debug uniquement.
    if (debugMode) {
      camera.viewport.add(
        FpsTextComponent(position: Vector2(8, 8)),
      );
    }
```

`FpsTextComponent` est fourni par `package:flame/components.dart`. Objectif : 60 images par seconde stables sur trois parties enchaînées.

### 6. Le confort

| Point | Pourquoi |
| --- | --- |
| Le bouton principal de chaque écran est le plus utile | un clic de moins par partie |
| Aucun texte ne dépasse en 16:9 comme en 4:3 | tester en redimensionnant la fenêtre |
| Le jeu démarre en moins de deux secondes | l'écran de chargement doit être bref |
| Les actions destructives sont confirmées | abandonner, effacer |
| Un texte explique le mode silencieux | l'élève ne croit pas à un bug |

### 7. Le code

- aucun `print` restant (utilisez `debugPrint`) ;
- `flutter analyze` ne renvoie **aucun** avertissement ;
- aucun nombre magique hors de `Constantes` ;
- aucune couleur hors de `Palette` ;
- aucun `import 'package:flame_audio/...'` en dehors d'`audio_service.dart` ;
- aucun `import 'package:shared_preferences/...'` en dehors de `sauvegarde_service.dart`.

Les deux derniers points se vérifient en une commande :

```text
grep -rn "flame_audio" lib/ | grep -v "lib/services/audio_service.dart"
```

Une sortie vide signifie que votre architecture tient.

---

## 40.31 — Le jeu complet : arborescence finale et `pubspec.yaml` final

### L'arborescence

```text
  donjon_de_dart/  ── ÉTAT FINAL, FIN DE LA PARTIE 2C
  │
  ├── pubspec.yaml                       ← MODIFIÉ ch. 40
  ├── analysis_options.yaml
  ├── assets/
  │   ├── images/                        (vide : le jeu n'en a pas besoin)
  │   └── audio/                         (vide : mode silencieux)
  ├── test/
  │   ├── audio_service_test.dart        ← NOUVEAU ch. 40
  │   └── statistiques_test.dart         ← NOUVEAU ch. 40
  └── lib/
      ├── main.dart                      ← MODIFIÉ ch. 40 (4 overlays)
      ├── donjon_game.dart               ← MODIFIÉ ch. 40
      ├── config/
      │   ├── constantes.dart            ch. 35
      │   └── palette.dart               ch. 35
      ├── core/
      │   ├── game_state.dart            ch. 35
      │   ├── entite.dart                ch. 36
      │   ├── sante.dart                 ch. 37
      │   └── statistiques.dart          ← NOUVEAU ch. 40
      ├── composants/
      │   ├── joueur.dart                ch. 36  (+ sons ch. 40)
      │   ├── plateforme.dart            ch. 36
      │   ├── ennemi.dart                ch. 37  (+ journal ch. 40)
      │   ├── gobelin.dart               ch. 37
      │   ├── chauvesouris.dart          ch. 37
      │   ├── projectile.dart            ch. 37
      │   ├── collectible.dart           ch. 38
      │   ├── piece.dart                 ch. 38  (+ sons ch. 40)
      │   ├── potion.dart                ch. 38  (+ sons ch. 40)
      │   ├── cle.dart                   ch. 38  (+ sons ch. 40)
      │   ├── texte_flottant.dart        ch. 38
      │   ├── porte.dart                 ch. 39  (+ sons ch. 40)
      │   └── boss.dart                  ch. 39  (+ musique ch. 40)
      ├── niveaux/
      │   ├── niveau.dart                ch. 39
      │   └── niveaux_data.dart          ch. 39
      ├── hud/
      │   ├── hud.dart                   ch. 38  (+ BoutonPause ch. 40)
      │   ├── barre_de_vie.dart          ch. 38
      │   └── compteur_score.dart        ch. 38
      ├── services/                      ← NOUVEAU DOSSIER ch. 40
      │   ├── audio_service.dart         ← NOUVEAU ch. 40
      │   └── sauvegarde_service.dart    ← NOUVEAU ch. 40
      └── ecrans/
          ├── menu_principal.dart        ch. 35  (+ Continuer ch. 40)
          ├── ecran_pause.dart           ← NOUVEAU ch. 40
          ├── ecran_game_over.dart       ← NOUVEAU ch. 40
          ├── ecran_victoire.dart        ← NOUVEAU ch. 40
          └── ecran_options.dart         ← NOUVEAU ch. 40
```

Trente fichiers Dart. Aucun fichier binaire. Un projet complet tient dans un dépôt de moins de 300 kilo-octets.

### Le `pubspec.yaml` final

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

  # Son : effets et musique de fond (chapitre 40).
  flame_audio: ^2.12.2

  # Persistance clé-valeur : meilleur score, progression, réglages (ch. 40).
  shared_preferences: ^2.5.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  assets:
    # Ces deux dossiers peuvent rester VIDES : le jeu fonctionne sans.
    # Déposez-y vos fichiers et ils seront pris en compte sans changer
    # une ligne de code (sections 29.x pour les images, 40.8 pour l'audio).
    - assets/images/
    - assets/audio/
```

> **Attention.** Un dossier d'assets **totalement vide** peut faire échouer `flutter pub get` sur certaines versions de l'outillage : Flutter refuse de déclarer un dossier inexistant. Placez-y un fichier `.gitkeep` ou le `CREDITS.txt` de la section 34.14, et le problème disparaît.

### Les commandes de vérification

```text
flutter pub get
flutter analyze          → No issues found!
flutter test             → All tests passed!
flutter run -d chrome
flutter build apk --release
flutter build web --release
```

Le chapitre 42 détaillera les deux dernières.

---

## 40.32 — Bilan de la PARTIE 2C

### Ce que vous avez construit en six chapitres

```text
  ch. 35 ──► squelette, machine à états, overlays, menu           ~800 lignes
  ch. 36 ──► joueur, gravité, saut, animations                    ~600 lignes
  ch. 37 ──► ennemis, IA, combat, vies                            ~700 lignes
  ch. 38 ──► collectibles, score, HUD                             ~900 lignes
  ch. 39 ──► niveaux, portes, boss, progression                   ~800 lignes
  ch. 40 ──► son, pause, fins, sauvegarde                         ~900 lignes
                                                        ─────────────────────
                                                        environ 4700 lignes
```

Ce n'est pas un chiffre à retenir. C'est un ordre de grandeur à connaître : **un petit jeu complet, c'est quelques milliers de lignes**, pas quelques centaines et pas cent mille.

### Les critères d'acceptation du chapitre 35

Reprenons-les un par un. C'est le moment de vérité.

| Critère (section 35.2) | État |
| --- | --- |
| 1. L'application se lance sur un menu, jamais d'écran noir | **atteint** (ch. 35, `loadingBuilder`) |
| 2. Le joueur peut terminer les trois niveaux sans blocage | **atteint** (ch. 39) |
| 3. La perte des trois vies conduit au Game Over, qui ramène au menu | **atteint** (ch. 40) |
| 4. Le meilleur score survit à la fermeture complète | **atteint** (ch. 40) |
| 5. Le jeu tourne à 60 images par seconde | **à vérifier chez vous** (`FpsTextComponent`) |
| 6. Aucune ressource externe n'est nécessaire | **atteint** depuis le chapitre 35 |

Six sur six, dont un à mesurer sur votre machine. Le cahier des charges est rempli.

### Les notions Dart et Flutter réellement employées

| Notion | Chapitre d'origine | Où elle sert dans le jeu |
| --- | --- | --- |
| Enums et `switch` exhaustif | 11 | `GameState`, `Effet`, `TypeEvenement` |
| Mixins | 11 | `Sante`, `HasGameReference`, `CollisionCallbacks` |
| Null Safety, `??`, `?.` | 12 | lectures de sauvegarde, `joueur?` |
| Exceptions, `try`/`catch`/`finally` | 13 | audio absent, sauvegarde corrompue |
| `where`, `map`, `fold`, `whereType` | 14 | statistiques, retrait du HUD |
| `Future`, `async`/`await`, `unawaited` | 15 | chargement, persistance |
| Organisation en paquets et services | 16 | `services/`, `config/`, `core/` |
| JSON, `toJson`/`fromJson` | 17 | `Sauvegarde` |
| Widgets, `StatefulWidget`, `setState` | 19 | tous les écrans |
| Boucle de jeu et `dt` | 20 | chronomètre, combo |
| Machine à états | 26 | `changerEtat` |
| Composants Flame et cycle de vie | 28 | tout le jeu |
| Overlays | 27 et 35 | six écrans |

Aucune notion de la PARTIE 1A n'a été apprise pour rien.

### Ce que le jeu ne fait toujours pas

Soyons précis, pour que vous sachiez quoi construire ensuite :

- pas de sprites ni de musiques (par choix ; le branchement est prêt) ;
- pas de sauvegarde de la position exacte du joueur dans un niveau ;
- pas de tableau des meilleurs scores en ligne ;
- pas de manette ;
- pas de traduction ;
- pas de tests automatisés du gameplay (chapitre 42).

Les quatre premiers sont des exercices. Les deux derniers sont la PARTIE 2D.

---

## 40.33 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: assets/audio/assets/audio/saut.wav` | chemin complet passé à `FlameAudio.play` | passer `'saut.wav'` : Flame ajoute `assets/audio/` lui-même |
| Exception non capturée à chaque son | `FlameAudio.play(...)` est un `Future` ignoré | `await` dans une méthode `async` gardée, ou `.catchError(...)` |
| `clearCache()` n'existe pas | méthode inventée par de vieux tutoriels | `FlameAudio.audioCache.clearAll()` |
| La musique continue quand on quitte l'application | `FlameAudio.bgm.initialize()` n'a jamais été appelé | l'appeler une fois dans `onLoad()` |
| La musique redémarre à zéro à chaque sortie de pause | on appelle `bgm.play` au lieu de `bgm.resume` | tester la provenance de l'état dans `_appliquerAudio` |
| Le curseur du volume musique ne fait rien | le volume n'est appliqué qu'au prochain `play` | `FlameAudio.bgm.audioPlayer.setVolume(v)` dans le setter |
| Deux musiques superposées | `arreterMusique()` sans remise à `null` de `_musiqueCourante` | remettre `_musiqueCourante = null` dans `arreterMusique` |
| L'écran de pause est figé et ne réagit pas | il a été écrit en composants Flame | l'écrire en widgets Flutter : `pauseEngine()` gèle Flame, pas Flutter |
| Échap ne sort pas de la pause | l'overlay a pris le focus, `DonjonGame.onKeyEvent` ne reçoit plus rien | envelopper l'écran de pause dans un `Focus(autofocus: true, onKeyEvent: ...)` |
| Revenir de l'arrière-plan relance le jeu et tue le héros | on a utilisé `basculerPause()` dans `lifecycleStateChange` | utiliser `mettreEnPause()` et ne jamais reprendre automatiquement |
| Le jeu ralentit après cinq parties | le monde n'est pas vidé entre deux chargements | `viderLeMonde()` avant `chargerNiveau`, vérifier avec F2 |
| Une barre de vie s'affiche par-dessus le menu | le `Hud` vit dans `camera.viewport`, que `viderLeMonde()` ne touche pas | `retirerHud()` à l'entrée dans `GameState.menu` |
| « NOUVEAU RECORD » ne s'affiche jamais | test `score > meilleurScore` alors que `ajouterScore` a déjà mis à jour le record | tester `score >= meilleurScore` |
| Le meilleur score retombe à zéro | `enregistrerMeilleurScore(score)` appelé avec le score courant, sans garde | garde `if (score <= donnees.meilleurScore) return;` |
| `Binding has not yet been initialized` | `shared_preferences` utilisé avant `runApp` sans liaison | `WidgetsFlutterBinding.ensureInitialized()` en tête de `main()` |
| `type 'double' is not a subtype of type 'int'` à la relecture | `as int?` sur une valeur JSON écrite `5.0` | `(json['x'] as num?)?.toInt() ?? 0` |
| `StateError: No element` sur l'écran de victoire | `reduce` sur une collection vide | `fold<int>(0, ...)` avec une valeur initiale |
| Le jeu ne démarre plus après une mise à jour | format de sauvegarde changé, exception au chargement | `try`/`catch` global dans `initialiser()` + valeurs par défaut |
| `use_build_context_synchronously` | `BuildContext` utilisé après un `await` | tester `if (context.mounted)` avant |
| Les statistiques cumulent d'une partie à l'autre | `journalPartie` non vidé dans `demarrerPartie` | `journalPartie.clear()` au démarrage |
| Le chronomètre compte le temps passé en pause | incrémenté avec `DateTime.now()` | l'incrémenter avec `dt`, uniquement si `etat == GameState.enJeu` |
| La sauvegarde marche… jusqu'au vrai redémarrage | testée avec un *hot restart* | fermer complètement l'application avant de retester |

---

## 40.34 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `AudioService` | singleton ; un seul fichier du projet importe `flame_audio` |
| `Effet` | enum d'identifiants logiques ; le jeu ne connaît aucun nom de fichier |
| Préchargement | `FlameAudio.audioCache.loadAll(...)` dans `onLoad`, sous `try`/`catch` |
| `FlameAudio.play` | renvoie un `Future<AudioPlayer>` : ne jamais l'ignorer sans `catchError` |
| `FlameAudio.bgm` | un seul lecteur ; `initialize()` une fois ; `play`/`pause`/`resume`/`stop`/`dispose` |
| Volume musique | s'applique à chaud via `bgm.audioPlayer.setVolume(v)` |
| Volume effets | s'applique au prochain `jouer()` |
| Mode silencieux | `disponible == false` ; le retour visuel passe par le hook `surEffet` |
| `pauseEngine()` | gèle Flame ; ni Flutter, ni l'audio, ni les `Future` |
| Écrans de pause et de fin | widgets **Flutter**, jamais des composants Flame |
| `basculerPause` / `mettreEnPause` | bascule pour le joueur, mise en pause seule pour la perte de focus |
| `lifecycleStateChange` | hook fourni par le mixin `Game` ; on met en pause, on ne reprend jamais |
| `EcranGameOver` | fond opaque, trois sorties, la plus utile en premier |
| Test de record | `score >= meilleurScore`, parce que `ajouterScore` met déjà à jour |
| `EcranVictoire` | réutilise `TableauScores` et ajoute `TableauStatistiques` |
| Journal d'événements | une `List<EvenementPartie>` plutôt que dix compteurs |
| `where` / `map` / `fold` | filtrer, transformer, réduire ; `fold` et jamais `reduce` sur du vide |
| `shared_preferences` | 2.5.5 ; API moderne `SharedPreferencesAsync` ; toutes les lectures nullables |
| Clés de préférences | préfixées par le nom du jeu et versionnées |
| `Sauvegarde` | un seul document JSON, avec un champ `version` |
| Désérialisation robuste | `(json['x'] as num?)?.toInt() ?? 0` |
| Sauvegarde corrompue | valeurs par défaut, drapeau, message ; on n'efface pas tout de suite |
| Écriture | à la fin d'une partie, d'un niveau ou d'un réglage — jamais dans `update` |
| Écran Options | un seul widget, deux points d'entrée ; appliquer tout de suite, enregistrer à la fermeture |

---

## 40.35 — Code complet du chapitre

Cinq fichiers sont **nouveaux** : copiez-les tels quels. Les autres blocs sont des **ajouts** à des fichiers écrits aux chapitres 35 à 39 : insérez-les, ne remplacez pas les fichiers.

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
  flame: ^1.38.0
  flame_audio: ^2.12.2
  shared_preferences: ^2.5.5

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

### `lib/services/audio_service.dart`

```dart
import 'package:flame_audio/flame_audio.dart';
import 'package:flutter/foundation.dart';

/// Identifiants logiques des effets sonores du jeu.
/// Le reste du code ne connaît QUE cet enum, jamais un nom de fichier.
enum Effet {
  saut, coup, degat, piece, potion, cle, porte, mort, boss, victoire, clic,
}

/// Service audio unique du jeu.
///
/// Seul fichier du projet à importer `flame_audio`. Fonctionne intégralement
/// sans aucun fichier dans `assets/audio/` (mode silencieux).
class AudioService {
  AudioService._();
  static final AudioService instance = AudioService._();

  // ---- État ------------------------------------------------------------

  /// Vrai quand les fichiers audio ont pu être chargés.
  bool disponible = false;

  bool musiqueActivee = true;
  bool effetsActives = true;

  double _volumeMusique = 0.5;
  double _volumeEffets = 0.8;

  String? _musiqueCourante;

  /// Journal des effets demandés, pour les tests et le débogage.
  final List<Effet> journal = <Effet>[];

  /// Appelé à CHAQUE demande d'effet, que le son sorte ou non.
  void Function(Effet effet)? surEffet;

  // ---- Tables ----------------------------------------------------------

  static const Map<Effet, String> _fichiers = <Effet, String>{
    Effet.saut: 'saut.wav',
    Effet.coup: 'coup.wav',
    Effet.degat: 'degat.wav',
    Effet.piece: 'piece.wav',
    Effet.potion: 'potion.wav',
    Effet.cle: 'cle.wav',
    Effet.porte: 'porte.wav',
    Effet.mort: 'mort.wav',
    Effet.boss: 'boss.wav',
    Effet.victoire: 'victoire.wav',
    Effet.clic: 'clic.wav',
  };

  static const Map<Effet, double> _equilibrage = <Effet, double>{
    Effet.saut: 0.45, Effet.coup: 0.90, Effet.degat: 0.85,
    Effet.piece: 0.55, Effet.potion: 0.60, Effet.cle: 0.70,
    Effet.porte: 0.75, Effet.mort: 1.00, Effet.boss: 1.00,
    Effet.victoire: 0.90, Effet.clic: 0.35,
  };

  static const Map<String, String> _musiques = <String, String>{
    'menu': 'musique_menu.mp3',
    'niveau_0': 'musique_donjon.mp3',
    'niveau_1': 'musique_donjon.mp3',
    'niveau_2': 'musique_crypte.mp3',
    'boss': 'musique_boss.mp3',
    'victoire': 'musique_victoire.mp3',
  };

  /// Délai minimal entre deux lectures du même effet, en millisecondes.
  static const Map<Effet, int> _delaiMinimal = <Effet, int>{
    Effet.piece: 60, Effet.saut: 80, Effet.degat: 120,
  };

  final Map<Effet, int> _dernierJeu = <Effet, int>{};

  // ---- Volumes ---------------------------------------------------------

  double get volumeMusique => _volumeMusique;
  double get volumeEffets => _volumeEffets;

  set volumeMusique(double valeur) {
    _volumeMusique = valeur.clamp(0.0, 1.0);
    _appliquerVolumeMusique();
  }

  set volumeEffets(double valeur) {
    _volumeEffets = valeur.clamp(0.0, 1.0);
  }

  void _appliquerVolumeMusique() {
    if (!disponible) {
      return;
    }
    try {
      FlameAudio.bgm.audioPlayer
          .setVolume(musiqueActivee ? _volumeMusique : 0.0);
    } catch (erreur) {
      debugPrint('[AudioService] Volume musique : $erreur');
    }
  }

  // ---- Cycle de vie ----------------------------------------------------

  /// À appeler UNE FOIS, depuis `DonjonGame.onLoad()`.
  Future<void> initialiser() async {
    FlameAudio.bgm.initialize();
    await precharger();
  }

  /// Charge tous les effets. Ne lève jamais.
  Future<void> precharger() async {
    try {
      await FlameAudio.audioCache.loadAll(_fichiers.values.toList());
      disponible = true;
    } catch (erreur) {
      disponible = false;
      debugPrint(
        '[AudioService] Aucun fichier audio trouvé, mode silencieux. ($erreur)',
      );
    }
  }

  /// Applique des réglages relus depuis la sauvegarde.
  void appliquerReglages(Reglages r) {
    musiqueActivee = r.musiqueActivee;
    effetsActives = r.effetsActives;
    volumeMusique = r.volumeMusique;
    volumeEffets = r.volumeEffets;
  }

  Future<void> liberer() async {
    _musiqueCourante = null;
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.dispose();
      await FlameAudio.audioCache.clearAll();
    } catch (erreur) {
      debugPrint('[AudioService] Libération : $erreur');
    } finally {
      disponible = false;
    }
  }

  // ---- Effets ----------------------------------------------------------

  bool _tropTot(Effet effet) {
    final int? delai = _delaiMinimal[effet];
    if (delai == null) {
      return false;
    }
    final int maintenant = DateTime.now().millisecondsSinceEpoch;
    final int? precedent = _dernierJeu[effet];
    if (precedent != null && maintenant - precedent < delai) {
      return true;
    }
    _dernierJeu[effet] = maintenant;
    return false;
  }

  /// Demande la lecture d'un effet. Synchrone du point de vue de l'appelant.
  void jouer(Effet effet) {
    journal.add(effet);
    if (journal.length > 64) {
      journal.removeAt(0);
    }
    surEffet?.call(effet);

    if (!disponible || !effetsActives || _volumeEffets <= 0) {
      return;
    }
    if (_tropTot(effet)) {
      return;
    }
    _jouerFichier(effet);
  }

  Future<void> _jouerFichier(Effet effet) async {
    final double volume = _volumeEffets * (_equilibrage[effet] ?? 1.0);
    try {
      await FlameAudio.play(_fichiers[effet]!, volume: volume);
    } catch (erreur) {
      disponible = false;
      debugPrint('[AudioService] Lecture impossible : $erreur');
    }
  }

  // ---- Musique ---------------------------------------------------------

  Future<void> jouerMusique(String ambiance) async {
    final String? fichier = _musiques[ambiance];
    if (fichier == null) {
      await arreterMusique();
      return;
    }
    if (fichier == _musiqueCourante && FlameAudio.bgm.isPlaying) {
      return;
    }
    _musiqueCourante = fichier;

    if (!disponible || !musiqueActivee || _volumeMusique <= 0) {
      return;
    }
    try {
      await FlameAudio.bgm.play(fichier, volume: _volumeMusique);
    } catch (erreur) {
      disponible = false;
      debugPrint('[AudioService] Musique indisponible : $erreur');
    }
  }

  void musiqueDuNiveau(int index, {bool boss = false}) {
    jouerMusique(boss ? 'boss' : 'niveau_$index');
  }

  Future<void> pauseMusique() async {
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.pause();
    } catch (_) {}
  }

  Future<void> repriseMusique() async {
    if (!disponible || !musiqueActivee) {
      return;
    }
    try {
      await FlameAudio.bgm.resume();
    } catch (_) {}
  }

  Future<void> arreterMusique() async {
    _musiqueCourante = null;
    if (!disponible) {
      return;
    }
    try {
      await FlameAudio.bgm.stop();
    } catch (_) {}
  }

  void basculerMusique() {
    musiqueActivee = !musiqueActivee;
    if (musiqueActivee) {
      repriseMusique();
    } else {
      pauseMusique();
    }
  }

  void basculerEffets() {
    effetsActives = !effetsActives;
  }

  Future<void> couperTout() async {
    musiqueActivee = false;
    effetsActives = false;
    await arreterMusique();
  }
}
```

> **Import à ajouter en tête du fichier :** `import 'sauvegarde_service.dart';`
> pour le type `Reglages` utilisé par `appliquerReglages`.

### `lib/services/sauvegarde_service.dart`

```dart
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../config/constantes.dart';

/// Tout ce que le joueur peut régler, et qui doit lui survivre.
class Reglages {
  Reglages({
    this.volumeMusique = 0.5,
    this.volumeEffets = 0.8,
    this.musiqueActivee = true,
    this.effetsActives = true,
    this.difficulte = 1.0,
    Map<String, String>? touches,
  }) : touches = touches ?? Map<String, String>.from(touchesParDefaut);

  factory Reglages.fromJson(Map<String, dynamic> json) => Reglages(
        volumeMusique:
            ((json['volumeMusique'] as num?)?.toDouble() ?? 0.5).clamp(0.0, 1.0),
        volumeEffets:
            ((json['volumeEffets'] as num?)?.toDouble() ?? 0.8).clamp(0.0, 1.0),
        musiqueActivee: json['musiqueActivee'] as bool? ?? true,
        effetsActives: json['effetsActives'] as bool? ?? true,
        difficulte:
            ((json['difficulte'] as num?)?.toDouble() ?? 1.0).clamp(0.5, 2.0),
        touches: (json['touches'] as Map<String, dynamic>?)?.map(
          (String k, dynamic v) => MapEntry<String, String>(k, '$v'),
        ),
      );

  double volumeMusique;
  double volumeEffets;
  bool musiqueActivee;
  bool effetsActives;
  double difficulte;
  final Map<String, String> touches;

  static const Map<String, String> touchesParDefaut = <String, String>{
    'gauche': 'a',
    'droite': 'd',
    'saut': 'space',
    'attaque': 'j',
    'pause': 'escape',
  };

  static const Map<String, LogicalKeyboardKey> touchesConnues =
      <String, LogicalKeyboardKey>{
    'a': LogicalKeyboardKey.keyA,
    'd': LogicalKeyboardKey.keyD,
    'j': LogicalKeyboardKey.keyJ,
    'k': LogicalKeyboardKey.keyK,
    'space': LogicalKeyboardKey.space,
    'escape': LogicalKeyboardKey.escape,
    'left': LogicalKeyboardKey.arrowLeft,
    'right': LogicalKeyboardKey.arrowRight,
    'up': LogicalKeyboardKey.arrowUp,
  };

  /// Touche associée à une action, avec double repli.
  LogicalKeyboardKey touchePour(String action) {
    final String nom = touches[action] ?? touchesParDefaut[action]!;
    return touchesConnues[nom] ?? touchesConnues[touchesParDefaut[action]!]!;
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'volumeMusique': volumeMusique,
        'volumeEffets': volumeEffets,
        'musiqueActivee': musiqueActivee,
        'effetsActives': effetsActives,
        'difficulte': difficulte,
        'touches': touches,
      };
}

/// L'ensemble des données persistées du jeu.
///
/// `niveauAtteint` = index du niveau le plus avancé que le joueur a le droit
/// de commencer. 0 au premier lancement, `Constantes.nombreNiveaux` quand la
/// campagne est terminée.
class Sauvegarde {
  Sauvegarde({
    this.version = versionCourante,
    this.meilleurScore = 0,
    this.niveauAtteint = 0,
    this.partiesJouees = 0,
    Reglages? reglages,
  }) : reglages = reglages ?? Reglages();

  factory Sauvegarde.parDefaut() => Sauvegarde();

  factory Sauvegarde.fromJson(Map<String, dynamic> json) => Sauvegarde(
        version: (json['version'] as num?)?.toInt() ?? 1,
        meilleurScore: (json['meilleurScore'] as num?)?.toInt() ?? 0,
        niveauAtteint: ((json['niveauAtteint'] as num?)?.toInt() ?? 0)
            .clamp(0, Constantes.nombreNiveaux),
        partiesJouees: (json['partiesJouees'] as num?)?.toInt() ?? 0,
        reglages: Reglages.fromJson(
          (json['reglages'] as Map<String, dynamic>?) ?? <String, dynamic>{},
        ),
      );

  static const int versionCourante = 1;

  int version;
  int meilleurScore;
  int niveauAtteint;
  int partiesJouees;
  Reglages reglages;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'version': version,
        'meilleurScore': meilleurScore,
        'niveauAtteint': niveauAtteint,
        'partiesJouees': partiesJouees,
        'reglages': reglages.toJson(),
      };

  /// Adapte un document ancien au format courant.
  static Map<String, dynamic> migrer(Map<String, dynamic> json) {
    final int version = (json['version'] as num?)?.toInt() ?? 1;
    if (version < 1) {
      json['version'] = 1;
    }
    return json;
  }
}

/// Persistance du jeu. Seul fichier à importer `shared_preferences`.
class SauvegardeService {
  SauvegardeService._();
  static final SauvegardeService instance = SauvegardeService._();

  static const String _cleSauvegarde = 'donjon_de_dart.sauvegarde.v1';

  final SharedPreferencesAsync _prefs = SharedPreferencesAsync();

  /// Toujours valide, même avant `initialiser()`.
  Sauvegarde donnees = Sauvegarde.parDefaut();

  bool pret = false;
  bool sauvegardeCorrompue = false;

  bool get aUneSauvegarde =>
      donnees.niveauAtteint > 0 &&
      donnees.niveauAtteint < Constantes.nombreNiveaux;

  /// Lit la sauvegarde. Ne lève jamais.
  Future<void> initialiser() async {
    try {
      final String? brut = await _prefs.getString(_cleSauvegarde);
      if (brut == null || brut.isEmpty) {
        donnees = Sauvegarde.parDefaut();
      } else {
        final Object? decode = jsonDecode(brut);
        if (decode is! Map<String, dynamic>) {
          throw const FormatException('La sauvegarde n\'est pas un objet.');
        }
        donnees = Sauvegarde.fromJson(Sauvegarde.migrer(decode));
        sauvegardeCorrompue = false;
      }
    } catch (erreur) {
      donnees = Sauvegarde.parDefaut();
      sauvegardeCorrompue = true;
      debugPrint('[Sauvegarde] Illisible, valeurs par défaut. ($erreur)');
    } finally {
      pret = true;
    }
  }

  Future<void> _ecrire() async {
    try {
      await _prefs.setString(_cleSauvegarde, jsonEncode(donnees.toJson()));
    } catch (erreur) {
      debugPrint('[Sauvegarde] Écriture impossible : $erreur');
    }
  }

  /// N'écrase jamais un meilleur score par un moins bon.
  Future<void> enregistrerMeilleurScore(int score) async {
    if (score <= donnees.meilleurScore) {
      return;
    }
    donnees.meilleurScore = score;
    await _ecrire();
  }

  /// La progression ne recule jamais.
  Future<void> enregistrerProgression(int niveauAtteint) async {
    final int borne = niveauAtteint.clamp(0, Constantes.nombreNiveaux);
    if (borne <= donnees.niveauAtteint) {
      return;
    }
    donnees.niveauAtteint = borne;
    await _ecrire();
  }

  Future<void> enregistrerReglages(Reglages reglages) async {
    donnees.reglages = reglages;
    await _ecrire();
  }

  Future<void> incrementerPartiesJouees() async {
    donnees.partiesJouees++;
    await _ecrire();
  }

  Future<void> reinitialiser() async {
    donnees = Sauvegarde.parDefaut();
    sauvegardeCorrompue = false;
    try {
      await _prefs.remove(_cleSauvegarde);
    } catch (erreur) {
      debugPrint('[Sauvegarde] Effacement impossible : $erreur');
    }
  }

  /// MODE DEBUG UNIQUEMENT : provoque une sauvegarde illisible.
  Future<void> corrompreVolontairement() async {
    await _prefs.setString(_cleSauvegarde, '{ ceci n\'est pas du JSON');
  }
}
```

### `lib/core/statistiques.dart`

```dart
/// Ce qui peut arriver de notable pendant une partie.
enum TypeEvenement {
  pieceRamassee,
  potionBue,
  cleRamassee,
  ennemiVaincu,
  bossVaincu,
  degatSubi,
  viePerdue,
  niveauTermine,
}

/// Un fait de jeu, daté et situé.
class EvenementPartie {
  const EvenementPartie({
    required this.type,
    required this.valeur,
    required this.instant,
    required this.niveau,
  });

  final TypeEvenement type;

  /// Grandeur associée : points, PV, index. 1 quand elle n'a pas de sens.
  final int valeur;

  /// Secondes de JEU écoulées depuis le début de la partie.
  final double instant;

  final int niveau;

  @override
  String toString() =>
      '${instant.toStringAsFixed(1)}s n$niveau ${type.name} ($valeur)';
}

/// Résumé chiffré d'une partie, calculé à partir du journal.
class StatistiquesPartie {
  const StatistiquesPartie({
    required this.duree,
    required this.score,
    required this.meilleurScore,
    required this.pieces,
    required this.pointsDesPieces,
    required this.potions,
    required this.cles,
    required this.ennemisVaincus,
    required this.bossVaincus,
    required this.degatsSubis,
    required this.viesPerdues,
    required this.niveauxTermines,
  });

  factory StatistiquesPartie.vide() => const StatistiquesPartie(
        duree: 0, score: 0, meilleurScore: 0, pieces: 0, pointsDesPieces: 0,
        potions: 0, cles: 0, ennemisVaincus: 0, bossVaincus: 0,
        degatsSubis: 0, viesPerdues: 0, niveauxTermines: 0,
      );

  /// Calcule les statistiques à partir du journal. Aucune boucle `for`.
  factory StatistiquesPartie.calculer(
    List<EvenementPartie> journal, {
    required double duree,
    required int score,
    required int meilleurScore,
  }) {
    int compter(TypeEvenement type) =>
        journal.where((EvenementPartie e) => e.type == type).length;

    int cumuler(TypeEvenement type) => journal
        .where((EvenementPartie e) => e.type == type)
        .map((EvenementPartie e) => e.valeur)
        .fold<int>(0, (int total, int v) => total + v);

    return StatistiquesPartie(
      duree: duree,
      score: score,
      meilleurScore: meilleurScore,
      pieces: compter(TypeEvenement.pieceRamassee),
      pointsDesPieces: cumuler(TypeEvenement.pieceRamassee),
      potions: compter(TypeEvenement.potionBue),
      cles: compter(TypeEvenement.cleRamassee),
      ennemisVaincus: compter(TypeEvenement.ennemiVaincu),
      bossVaincus: compter(TypeEvenement.bossVaincu),
      degatsSubis: cumuler(TypeEvenement.degatSubi),
      viesPerdues: compter(TypeEvenement.viePerdue),
      niveauxTermines: compter(TypeEvenement.niveauTermine),
    );
  }

  final double duree;
  final int score;
  final int meilleurScore;
  final int pieces;
  final int pointsDesPieces;
  final int potions;
  final int cles;
  final int ennemisVaincus;
  final int bossVaincus;
  final int degatsSubis;
  final int viesPerdues;
  final int niveauxTermines;

  String get dureeFormatee {
    final int total = duree.round();
    final int minutes = total ~/ 60;
    final String secondes = (total % 60).toString().padLeft(2, '0');
    return '$minutes:$secondes';
  }

  double get scoreParMinute => duree < 1 ? 0 : score * 60 / duree;

  String get rang {
    if (viesPerdues == 0 && duree < 180) {
      return 'S';
    }
    if (viesPerdues <= 1) {
      return 'A';
    }
    if (viesPerdues <= 3) {
      return 'B';
    }
    return 'C';
  }

  /// Instant du premier événement d'un type donné, ou `null`.
  static double? premierInstant(
    List<EvenementPartie> journal,
    TypeEvenement type,
  ) {
    final Iterable<EvenementPartie> filtres =
        journal.where((EvenementPartie e) => e.type == type);
    return filtres.isEmpty ? null : filtres.first.instant;
  }

  /// Le niveau où le joueur a perdu le plus de vies, ou `null`.
  static int? niveauLePlusDur(List<EvenementPartie> journal) {
    final Map<int, int> parNiveau = <int, int>{};
    for (final EvenementPartie e in journal.where(
      (EvenementPartie e) => e.type == TypeEvenement.viePerdue,
    )) {
      parNiveau[e.niveau] = (parNiveau[e.niveau] ?? 0) + 1;
    }
    if (parNiveau.isEmpty) {
      return null;
    }
    return parNiveau.entries
        .reduce((MapEntry<int, int> a, MapEntry<int, int> b) =>
            a.value >= b.value ? a : b)
        .key;
  }
}
```

### `lib/ecrans/ecran_pause.dart`

Le fichier complet est celui de la section 40.10. Il contient `EcranPause`,
`_LigneEtat`, `_Pastille` et le bouton partagé `BoutonEcran`.

### `lib/ecrans/ecran_game_over.dart`

Le fichier complet est celui des sections 40.14 et 40.15 : `EcranGameOver`
puis `TableauScores`, tous deux dans le même fichier.

### `lib/ecrans/ecran_victoire.dart`

Le fichier complet est celui de la section 40.17, suivi de
`TableauStatistiques` (section 40.18) placé dans le même fichier.

### `lib/ecrans/ecran_options.dart`

```dart
import 'package:flutter/material.dart';

import '../config/palette.dart';
import '../donjon_game.dart';
import '../services/audio_service.dart';
import '../services/sauvegarde_service.dart';

/// Écran d'options, ouvert depuis le menu principal ET depuis la pause.
///
/// Règle : appliquer immédiatement, enregistrer à la fermeture.
class EcranOptions extends StatefulWidget {
  const EcranOptions({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  State<EcranOptions> createState() => _EcranOptionsState();
}

class _EcranOptionsState extends State<EcranOptions> {
  late final Reglages _reglages =
      SauvegardeService.instance.donnees.reglages;

  late double _musique = _reglages.volumeMusique;
  late double _effets = _reglages.volumeEffets;
  late bool _musiqueOn = _reglages.musiqueActivee;
  late bool _effetsOn = _reglages.effetsActives;
  late double _difficulte = _reglages.difficulte;
  late bool _debug = widget.jeu.debugMode;

  /// Recopie l'état local dans les réglages, puis persiste.
  Future<void> _enregistrerEtFermer() async {
    _reglages
      ..volumeMusique = _musique
      ..volumeEffets = _effets
      ..musiqueActivee = _musiqueOn
      ..effetsActives = _effetsOn
      ..difficulte = _difficulte;

    await SauvegardeService.instance.enregistrerReglages(_reglages);

    if (mounted) {
      Navigator.of(context).pop();
    }
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      backgroundColor: Palette.panneau,
      title: const Text('OPTIONS', style: TextStyle(letterSpacing: 3)),
      content: SizedBox(
        width: 340,
        child: SingleChildScrollView(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              const Text('Volume de la musique', style: Palette.sousTitre),
              Slider(
                value: _musique,
                activeColor: Palette.accent,
                onChanged: (double v) {
                  setState(() => _musique = v);
                  AudioService.instance.volumeMusique = v;  // à chaud
                },
              ),
              const Text('Volume des effets', style: Palette.sousTitre),
              Slider(
                value: _effets,
                activeColor: Palette.accent,
                onChanged: (double v) => setState(() => _effets = v),
                onChangeEnd: (double v) {
                  AudioService.instance.volumeEffets = v;
                  AudioService.instance.jouer(Effet.piece);  // son témoin
                },
              ),
              SwitchListTile(
                contentPadding: EdgeInsets.zero,
                title: const Text('Musique', style: Palette.sousTitre),
                value: _musiqueOn,
                onChanged: (bool v) {
                  setState(() => _musiqueOn = v);
                  AudioService.instance.basculerMusique();
                },
              ),
              SwitchListTile(
                contentPadding: EdgeInsets.zero,
                title: const Text('Effets sonores', style: Palette.sousTitre),
                value: _effetsOn,
                onChanged: (bool v) {
                  setState(() => _effetsOn = v);
                  AudioService.instance.basculerEffets();
                },
              ),
              const SizedBox(height: 10),
              const Text('Difficulté', style: Palette.sousTitre),
              const SizedBox(height: 6),
              SegmentedButton<double>(
                segments: const <ButtonSegment<double>>[
                  ButtonSegment<double>(value: 0.75, label: Text('FACILE')),
                  ButtonSegment<double>(value: 1.0, label: Text('NORMAL')),
                  ButtonSegment<double>(value: 1.4, label: Text('DIFFICILE')),
                ],
                selected: <double>{_difficulte},
                onSelectionChanged: (Set<double> choix) {
                  setState(() => _difficulte = choix.first);
                  widget.jeu.difficulte = _difficulte;
                },
              ),
              const Padding(
                padding: EdgeInsets.only(top: 6),
                child: Text(
                  'La difficulté s\'applique au prochain niveau.',
                  style: TextStyle(color: Palette.texteFaible, fontSize: 11),
                ),
              ),
              const SizedBox(height: 8),
              SwitchListTile(
                contentPadding: EdgeInsets.zero,
                title: const Text('Mode debug', style: Palette.sousTitre),
                value: _debug,
                onChanged: (bool v) {
                  setState(() => _debug = v);
                  widget.jeu.debugMode = v;
                },
              ),
              if (!AudioService.instance.disponible)
                const Padding(
                  padding: EdgeInsets.only(top: 8),
                  child: Text(
                    'Mode silencieux : aucun fichier dans assets/audio/.',
                    style: TextStyle(
                      color: Palette.texteFaible,
                      fontSize: 11,
                    ),
                  ),
                ),
              const SizedBox(height: 12),
              const Divider(color: Palette.mur, height: 1),
              TextButton.icon(
                onPressed: () => _confirmerEffacement(context),
                icon: const Icon(Icons.delete_forever, color: Palette.danger),
                label: const Text(
                  'EFFACER LA SAUVEGARDE',
                  style: TextStyle(color: Palette.danger),
                ),
              ),
            ],
          ),
        ),
      ),
      actions: <Widget>[
        TextButton(
          onPressed: _enregistrerEtFermer,
          child: const Text('FERMER'),
        ),
      ],
    );
  }

  void _confirmerEffacement(BuildContext context) {
    showDialog<void>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        backgroundColor: Palette.panneau,
        title: const Text('Effacer la sauvegarde ?'),
        content: const Text(
          'Le meilleur score, la progression et les réglages seront perdus. '
          'Cette action est définitive.',
          style: TextStyle(color: Palette.texteFaible),
        ),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(c).pop(),
            child: const Text('ANNULER'),
          ),
          FilledButton(
            style: FilledButton.styleFrom(backgroundColor: Palette.danger),
            onPressed: () async {
              await SauvegardeService.instance.reinitialiser();
              widget.jeu.meilleurScore = 0;
              if (c.mounted) {
                Navigator.of(c).pop();
              }
              if (mounted) {
                setState(() {});
              }
            },
            child: const Text('EFFACER'),
          ),
        ],
      ),
    );
  }
}
```

### `lib/donjon_game.dart` — ajouts et modifications du chapitre 40

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import 'dart:async';                       // unawaited
// import 'package:flutter/widgets.dart';     // AppLifecycleState
// import 'core/statistiques.dart';
// import 'services/audio_service.dart';
// import 'services/sauvegarde_service.dart';

// ---- CHAMPS À AJOUTER DANS LA CLASSE DonjonGame ---------------------

  /// Journal de la partie en cours. Vidé par `demarrerPartie`.
  final List<EvenementPartie> journalPartie = <EvenementPartie>[];

  /// Statistiques de la dernière partie terminée.
  StatistiquesPartie statistiques = StatistiquesPartie.vide();

  double _chronometre = 0;
  double dureePartie = 0;

  /// Symboles affichés à la place des sons, en mode silencieux.
  static const Map<Effet, String> _symbolesAudio = <Effet, String>{
    Effet.coup: '!',
    Effet.degat: '!!',
    Effet.porte: 'CLIC',
    Effet.cle: 'CLÉ',
    Effet.boss: 'BOSS',
  };

// ---- onLoad : AJOUTS ------------------------------------------------

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    monde = world;
    camera.viewfinder.anchor = Anchor.center;
    camera.viewfinder.zoom = Constantes.zoomCamera;

    // AJOUTS DU CHAPITRE 40, dans cet ordre.
    await SauvegardeService.instance.initialiser();
    await AudioService.instance.initialiser();
    AudioService.instance
        .appliquerReglages(SauvegardeService.instance.donnees.reglages);
    AudioService.instance.surEffet = _retourVisuel;

    meilleurScore = SauvegardeService.instance.donnees.meilleurScore;
    difficulte = SauvegardeService.instance.donnees.reglages.difficulte;

    changerEtat(GameState.menu);
  }

  @override
  void onRemove() {
    AudioService.instance.liberer();
    super.onRemove();
  }

// ---- Retour visuel du mode silencieux --------------------------------

  void _retourVisuel(Effet effet) {
    if (AudioService.instance.disponible) {
      return;
    }
    final String? symbole = _symbolesAudio[effet];
    if (symbole == null || joueur == null) {
      return;
    }
    afficherTexteFlottant(joueur!.position, symbole, Palette.accent, taille: 9);
  }

// ---- changerEtat : AJOUT de l'audio ----------------------------------

  void changerEtat(GameState nouvelEtat) {
    if (nouvelEtat == etat) {
      return;
    }
    final GameState ancienEtat = etat;
    etat = nouvelEtat;

    overlays.remove(overlayDeLEtat(ancienEtat));
    overlays.add(overlayDeLEtat(nouvelEtat));

    if (nouvelEtat == GameState.enJeu) {
      resumeEngine();
    } else if (nouvelEtat == GameState.pause ||
        nouvelEtat == GameState.gameOver ||
        nouvelEtat == GameState.victoire) {
      pauseEngine();
    }

    if (nouvelEtat == GameState.menu) {
      retirerHud();        // AJOUT DU CHAPITRE 40
      viderLeMonde();
    }

    _appliquerAudio(ancienEtat, nouvelEtat);   // AJOUT DU CHAPITRE 40
  }

  void _appliquerAudio(GameState ancien, GameState nouveau) {
    final AudioService audio = AudioService.instance;
    switch (nouveau) {
      case GameState.menu:
        audio.jouerMusique('menu');
      case GameState.enJeu:
        if (ancien == GameState.pause) {
          audio.repriseMusique();
        } else {
          audio.musiqueDuNiveau(niveauCourant);
        }
      case GameState.pause:
        audio.pauseMusique();
      case GameState.gameOver:
        audio.arreterMusique();
        audio.jouer(Effet.mort);
      case GameState.victoire:
        audio.arreterMusique();
        audio.jouer(Effet.victoire);
        audio.jouerMusique('victoire');
      case GameState.chargement:
        break;
    }
  }

  /// Le HUD vit dans le viewport : `viderLeMonde()` ne le retire pas.
  void retirerHud() {
    camera.viewport.children
        .whereType<Hud>()
        .toList()
        .forEach((Hud h) => h.removeFromParent());
  }

// ---- Pause ------------------------------------------------------------

  void basculerPause() {
    switch (etat) {
      case GameState.enJeu:
        changerEtat(GameState.pause);
      case GameState.pause:
        changerEtat(GameState.enJeu);
      case GameState.chargement:
      case GameState.menu:
      case GameState.gameOver:
      case GameState.victoire:
        break;
    }
  }

  /// Met en pause sans jamais reprendre. Utilisée par la perte de focus.
  void mettreEnPause() {
    if (etat == GameState.enJeu) {
      changerEtat(GameState.pause);
    }
  }

  @override
  void lifecycleStateChange(AppLifecycleState state) {
    super.lifecycleStateChange(state);
    switch (state) {
      case AppLifecycleState.inactive:
      case AppLifecycleState.paused:
      case AppLifecycleState.hidden:
      case AppLifecycleState.detached:
        mettreEnPause();
        AudioService.instance.pauseMusique();
      case AppLifecycleState.resumed:
        if (etat != GameState.pause) {
          AudioService.instance.repriseMusique();
        }
    }
  }

// ---- Journal et chronomètre -------------------------------------------

  void _demarrerChronometre() {
    _chronometre = 0;
    dureePartie = 0;
  }

  void enregistrerEvenement(TypeEvenement type, [int valeur = 1]) {
    journalPartie.add(
      EvenementPartie(
        type: type,
        valeur: valeur,
        instant: _chronometre,
        niveau: niveauCourant,
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (etat == GameState.enJeu) {
      _chronometre += dt;      // AJOUT DU CHAPITRE 40
    }
    // ... fenêtre de combo du chapitre 38
  }

// ---- Cycle d'une partie -----------------------------------------------

  Future<void> demarrerPartie({int niveau = 0}) async {
    changerEtat(GameState.chargement);

    score = 0;
    vies = Constantes.viesDepart;
    niveauCourant = niveau.clamp(0, Constantes.nombreNiveaux - 1);
    reinitialiserCombo();
    journalPartie.clear();
    _demarrerChronometre();
    unawaited(SauvegardeService.instance.incrementerPartiesJouees());

    await chargerNiveau(niveauCourant);

    if (etat != GameState.chargement) {
      return;
    }
    changerEtat(GameState.enJeu);
  }

  Future<void> rejouerLeNiveau() async {
    changerEtat(GameState.chargement);

    vies = Constantes.viesDepart;
    reinitialiserCombo();
    _demarrerChronometre();

    await chargerNiveau(niveauCourant);

    if (etat != GameState.chargement) {
      return;
    }
    changerEtat(GameState.enJeu);
  }

  void perdreUneVie() {
    vies--;
    reinitialiserCombo();
    hud.barreDeVie.secouer();
    enregistrerEvenement(TypeEvenement.viePerdue);     // AJOUT ch. 40
    AudioService.instance.jouer(Effet.mort);           // AJOUT ch. 40

    if (vies <= 0) {
      vies = 0;
      declarerGameOver();
      return;
    }
    joueur!.reapparaitre(pointDeReapparition);
  }

  void declarerGameOver() {
    dureePartie = _chronometre;
    statistiques = StatistiquesPartie.calculer(
      journalPartie,
      duree: dureePartie,
      score: score,
      meilleurScore: meilleurScore,
    );
    _sauvegarderProgression();
    changerEtat(GameState.gameOver);
  }

  void declarerVictoire() {
    dureePartie = _chronometre;
    statistiques = StatistiquesPartie.calculer(
      journalPartie,
      duree: dureePartie,
      score: score,
      meilleurScore: meilleurScore,
    );
    _sauvegarderProgression();
    unawaited(
      SauvegardeService.instance
          .enregistrerProgression(Constantes.nombreNiveaux),
    );
    changerEtat(GameState.victoire);
  }

  void abandonnerPartie() {
    _sauvegarderProgression();
    changerEtat(GameState.menu);
  }

  void retournerAuMenu() {
    _sauvegarderProgression();
    changerEtat(GameState.menu);
  }

  void _sauvegarderProgression() {
    final SauvegardeService s = SauvegardeService.instance;
    unawaited(s.enregistrerMeilleurScore(meilleurScore));
    unawaited(s.enregistrerProgression(niveauCourant));
  }

// ---- Clavier : MODIFIÉ ------------------------------------------------

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent) {
      if (event.logicalKey == LogicalKeyboardKey.escape ||
          event.logicalKey == LogicalKeyboardKey.keyP) {
        basculerPause();                       // MODIFIÉ ch. 40
        return KeyEventResult.handled;
      }
      if (event.logicalKey == LogicalKeyboardKey.keyM) {
        AudioService.instance.basculerMusique();
        return KeyEventResult.handled;
      }
      if (event.logicalKey == LogicalKeyboardKey.f9 && debugMode) {
        unawaited(SauvegardeService.instance.reinitialiser());
        meilleurScore = 0;
        debugPrint('[DonjonGame] Sauvegarde effacée.');
        return KeyEventResult.handled;
      }
      // F1 et F2 : inchangés (chapitre 35).
    }
    return super.onKeyEvent(event, keysPressed);
  }
```

### `lib/hud/hud.dart` — ajouts du chapitre 40

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import 'package:flame/input.dart';
// import 'package:flutter/widgets.dart' show EdgeInsets;
// import '../services/audio_service.dart';

/// Bouton Pause du HUD, en haut à droite du viewport.
class BoutonPause extends HudButtonComponent
    with HasGameReference<DonjonGame> {
  BoutonPause()
      : super(
          size: Vector2.all(30),
          margin: const EdgeInsets.only(top: 10, right: 10),
          button: RectangleComponent(
            size: Vector2.all(30),
            paint: Paint()..color = Palette.panneau,
            children: <Component>[
              RectangleComponent(
                position: Vector2(9, 8),
                size: Vector2(4, 14),
                paint: Paint()..color = Palette.texte,
              ),
              RectangleComponent(
                position: Vector2(17, 8),
                size: Vector2(4, 14),
                paint: Paint()..color = Palette.texte,
              ),
            ],
          ),
          buttonDown: RectangleComponent(
            size: Vector2.all(30),
            paint: Paint()..color = Palette.accent,
          ),
        );

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // `game` n'est fiable qu'à partir d'onLoad.
    onPressed = () {
      AudioService.instance.jouer(Effet.clic);
      game.basculerPause();
    };
  }
}

// ---- À AJOUTER DANS Hud.onLoad() -------------------------------------
//     await add(BoutonPause());
```

### `lib/ecrans/menu_principal.dart` — modifications du chapitre 40

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import '../config/constantes.dart';
// import '../services/audio_service.dart';
// import '../services/sauvegarde_service.dart';
// import 'ecran_options.dart';

// ---- REMPLACER le bouchon du chapitre 35 -----------------------------

  bool get _peutContinuer => SauvegardeService.instance.aUneSauvegarde;

  int get _niveauDeReprise =>
      SauvegardeService.instance.donnees.niveauAtteint;

// ---- REMPLACER le bouton CONTINUER -----------------------------------

  BoutonMenu(
    libelle: 'CONTINUER',
    icone: Icons.history,
    onPressed: _peutContinuer
        ? () async {
            AudioService.instance.jouer(Effet.clic);
            await jeu.demarrerPartie(niveau: _niveauDeReprise);
          }
        : null,
    infoBulle: _peutContinuer
        ? 'Reprendre au niveau ${_niveauDeReprise + 1}'
        : 'Aucune progression enregistrée',
  ),

// ---- REMPLACER le bouton OPTIONS -------------------------------------

  BoutonMenu(
    libelle: 'OPTIONS',
    icone: Icons.settings,
    onPressed: () {
      AudioService.instance.jouer(Effet.clic);
      showDialog<void>(
        context: context,
        builder: (BuildContext c) => EcranOptions(jeu: jeu),
      );
    },
  ),

// ---- AJOUTER sous le bloc des boutons --------------------------------

  if (SauvegardeService.instance.donnees.partiesJouees > 0)
    Text(
      'Meilleur score : '
      '${SauvegardeService.instance.donnees.meilleurScore}   ·   '
      '${SauvegardeService.instance.donnees.partiesJouees} parties',
      style: const TextStyle(color: Palette.texteFaible, fontSize: 12),
    ),

  if (!AudioService.instance.disponible)
    const Padding(
      padding: EdgeInsets.only(top: 10),
      child: Text(
        'Mode silencieux : aucun fichier dans assets/audio/',
        style: TextStyle(color: Palette.texteFaible, fontSize: 11),
      ),
    ),

  if (SauvegardeService.instance.sauvegardeCorrompue)
    Container(
      margin: const EdgeInsets.only(top: 14),
      padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 8),
      decoration: BoxDecoration(
        border: Border.all(color: Palette.danger, width: 1),
        borderRadius: BorderRadius.circular(4),
      ),
      child: const Text(
        'Sauvegarde illisible : le jeu est reparti des réglages par défaut.',
        style: TextStyle(color: Palette.danger, fontSize: 11),
      ),
    ),

// ---- SUPPRIMER ------------------------------------------------------
// La classe `ReglagesProvisoires` et la classe `DialogueOptions` du
// chapitre 35 sont remplacées par `EcranOptions` et `Reglages`.
```

### `lib/main.dart` (final)

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

import 'config/constantes.dart';
import 'config/palette.dart';
import 'donjon_game.dart';
import 'ecrans/ecran_game_over.dart';
import 'ecrans/ecran_pause.dart';
import 'ecrans/ecran_victoire.dart';
import 'ecrans/menu_principal.dart';

Future<void> main() async {
  // Obligatoire : shared_preferences est un greffon de plateforme.
  WidgetsFlutterBinding.ensureInitialized();

  await SystemChrome.setPreferredOrientations(<DeviceOrientation>[
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight,
  ]);

  runApp(const DonjonApp());
}

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
        loadingBuilder: (BuildContext context) =>
            const EcranChargement(message: 'Préparation du donjon…'),
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
        initialActiveOverlays: const <String>[Overlays.chargement],
        overlayBuilderMap: <String, Widget Function(BuildContext, DonjonGame)>{
          Overlays.chargement: (BuildContext c, DonjonGame g) =>
              const EcranChargement(message: 'Construction du niveau…'),
          Overlays.menuPrincipal: (BuildContext c, DonjonGame g) =>
              MenuPrincipal(jeu: g),
          // Le HUD est un composant Flame (chapitre 38) : cet overlay est
          // volontairement vide. Il sert de « place » à l'état enJeu.
          Overlays.hud: (BuildContext c, DonjonGame g) =>
              const SizedBox.shrink(),
          Overlays.pause: (BuildContext c, DonjonGame g) => EcranPause(jeu: g),
          Overlays.gameOver: (BuildContext c, DonjonGame g) =>
              EcranGameOver(jeu: g),
          Overlays.victoire: (BuildContext c, DonjonGame g) =>
              EcranVictoire(jeu: g),
        },
      ),
    );
  }
}
```

> **Note.** `HudProvisoire`, écrit au chapitre 35, disparaît : le HUD Flame du
> chapitre 38 l'a remplacé. L'overlay `Overlays.hud` reste déclaré parce que
> `overlayDeLEtat` l'associe à `GameState.enJeu` ; il ne dessine plus rien.

---

## 40.36 — Exercices

Les exercices se font dans le projet « Donjon de Dart » tel qu'il est à la fin de ce chapitre. Ils vont du plus simple au plus complet.

### Exercice 1 — Le son du combo (facile)
Ajoutez une valeur `Effet.combo` à l'enum, son fichier et son équilibrage. Jouez-la dans `ajouterScore()` uniquement quand le multiplicateur **change** de valeur, jamais à chaque pièce.

### Exercice 2 — Le préchargement tolérant (facile)
`loadAll` échoue globalement si un seul fichier manque. Écrivez une méthode `prechargerUnParUn()` qui charge chaque fichier séparément, retire de `_fichiers` ceux qui manquent, et met `disponible` à `true` s'il en reste au moins un.

### Exercice 3 — Le raccourci de coupure totale (facile)
Ajoutez la touche `N` : elle appelle `AudioService.instance.couperTout()` et affiche un texte flottant « SON COUPÉ » au-dessus du joueur.

### Exercice 4 — Le dump de la sauvegarde (facile)
Ajoutez le raccourci `F3` en mode debug : il affiche dans la console la sauvegarde courante, formatée en JSON indenté avec `JsonEncoder.withIndent('  ')`.

### Exercice 5 — Le bouton pause en overlay (moyen)
Le `BoutonPause` du HUD ne peut pas sortir de la pause. Écrivez à la place un overlay Flutter `Overlays.hud` contenant un unique bouton en haut à droite, qui appelle `basculerPause()` et change d'icône selon `jeu.etat`.

### Exercice 6 — Les statistiques en un seul passage (moyen)
Remplacez les neuf appels à `compter`/`cumuler` par un unique `fold` qui construit une `Map<TypeEvenement, (int, int)>` — occurrences et somme des valeurs — puis lisez cette table. Vérifiez que les chiffres sont identiques.

### Exercice 7 — Le temps par niveau (moyen)
À partir du journal, calculez le temps passé dans chaque niveau et affichez-le dans `TableauStatistiques`. Indice : les événements `niveauTermine` portent l'instant de fin de chaque niveau.

### Exercice 8 — Le tableau des cinq meilleurs scores (difficile)
Ajoutez à `Sauvegarde` un champ `List<int> meilleursScores` (cinq valeurs maximum, triées par ordre décroissant). Sérialisez-le, mettez-le à jour en fin de partie, et affichez le classement sur l'écran de Game Over. Gérez la migration depuis une sauvegarde v1 qui ne contient pas ce champ.

### Exercice 9 — Le score pondéré par la difficulté (difficile)
Enregistrez le meilleur score en le multipliant par la difficulté (`0.75`, `1.0` ou `1.4`) et affichez, sur l'écran de fin, à la fois le score brut et le score pondéré. Vérifiez qu'un joueur en mode facile ne peut pas battre le record d'un joueur en mode difficile à performance égale.

### Exercice 10 — Le service de sauvegarde testable (difficile)
`SauvegardeService` dépend directement de `SharedPreferencesAsync`, donc d'un greffon de plateforme : impossible à tester en console. Introduisez une interface `MagasinCleValeur` avec deux implémentations — l'une sur `SharedPreferencesAsync`, l'autre en mémoire — et écrivez trois tests : premier lancement, relecture, sauvegarde corrompue.

---

## 40.37 — Corrections des exercices

### Correction 1

```dart
// lib/services/audio_service.dart
enum Effet { saut, coup, degat, piece, potion, cle, porte, mort, boss,
             victoire, clic, combo }

// _fichiers  : Effet.combo: 'combo.wav',
// _equilibrage : Effet.combo: 0.65,

// lib/donjon_game.dart
  void ajouterScore(int points) {
    if (points <= 0) {
      return;
    }
    final int avant = multiplicateur;          // AVANT modification

    score += points * multiplicateur;
    combo++;
    _tempsRestantCombo = dureeCombo;
    if (score > meilleurScore) {
      meilleurScore = score;
    }

    if (multiplicateur > avant) {              // le palier a changé
      AudioService.instance.jouer(Effet.combo);
    }
  }
```

**Explication :** le multiplicateur du chapitre 38 vaut `min(1 + combo ~/ 3, 4)`. Il ne change donc qu'un ramassage sur trois. On mémorise sa valeur **avant** d'incrémenter `combo`, et on ne joue le son que si la comparaison montre une progression. Jouer le son à chaque ramassage produirait un doublon avec `Effet.piece` — deux sons simultanés qui se masquent l'un l'autre.

### Correction 2

```dart
  /// Charge les fichiers un par un et oublie ceux qui manquent.
  Future<void> prechargerUnParUn() async {
    final List<Effet> manquants = <Effet>[];

    for (final MapEntry<Effet, String> entree in _fichiers.entries) {
      try {
        await FlameAudio.audioCache.load(entree.value);
      } catch (_) {
        manquants.add(entree.key);
      }
    }

    _indisponibles
      ..clear()
      ..addAll(manquants);

    disponible = manquants.length < _fichiers.length;

    if (manquants.isNotEmpty) {
      debugPrint(
        '[AudioService] ${manquants.length} effet(s) manquant(s) : '
        '${manquants.map((Effet e) => e.name).join(', ')}',
      );
    }
  }

  /// Effets dont le fichier est absent.
  final Set<Effet> _indisponibles = <Effet>{};

// Et dans `jouer`, après le test `disponible` :
    if (_indisponibles.contains(effet)) {
      return;
    }
```

**Explication :** `_fichiers` est `const` : on ne peut pas en retirer d'entrée. On tient donc à côté un `Set<Effet>` des manquants, et `jouer` le consulte. `disponible` devient « au moins un son fonctionne » au lieu de « tout fonctionne ». C'est plus tolérant, au prix d'un champ et de trois lignes — un bon compromis pour un projet où l'on ajoute les sons progressivement.

### Correction 3

```dart
// lib/donjon_game.dart — dans onKeyEvent
      if (event.logicalKey == LogicalKeyboardKey.keyN) {
        unawaited(AudioService.instance.couperTout());
        if (joueur != null) {
          afficherTexteFlottant(
            joueur!.position,
            'SON COUPÉ',
            Palette.texteFaible,
            taille: 9,
          );
        }
        return KeyEventResult.handled;
      }
```

**Explication :** `couperTout()` renvoie un `Future` parce qu'il attend `arreterMusique()`. `unawaited` documente qu'on ne l'attend pas volontairement : `onKeyEvent` doit rendre un `KeyEventResult` tout de suite. Le test `joueur != null` est indispensable : la touche peut être pressée au menu, où le monde est vide.

### Correction 4

```dart
// lib/services/sauvegarde_service.dart
  /// Représentation lisible de la sauvegarde. Débogage uniquement.
  String enJsonLisible() =>
      const JsonEncoder.withIndent('  ').convert(donnees.toJson());

// lib/donjon_game.dart — dans onKeyEvent
      if (event.logicalKey == LogicalKeyboardKey.f3 && debugMode) {
        debugPrint(SauvegardeService.instance.enJsonLisible());
        return KeyEventResult.handled;
      }
```

**Explication :** `JsonEncoder.withIndent` produit un JSON indenté, contrairement à `jsonEncode` qui écrit tout sur une ligne. Le constructeur est `const`, on peut donc le déclarer ainsi sans allouer. Le garde `&& debugMode` évite d'écrire l'état du joueur dans les journaux d'une version publiée.

### Correction 5

```dart
// lib/ecrans/bouton_pause_overlay.dart
class BoutonPauseOverlay extends StatelessWidget {
  const BoutonPauseOverlay({super.key, required this.jeu});

  final DonjonGame jeu;

  @override
  Widget build(BuildContext context) {
    final bool enPause = jeu.etat == GameState.pause;
    return Material(
      color: Colors.transparent,
      child: SafeArea(
        child: Align(
          alignment: Alignment.topRight,
          child: Padding(
            padding: const EdgeInsets.all(10),
            child: IconButton.filled(
              iconSize: 26,
              style: IconButton.styleFrom(
                backgroundColor: Palette.panneau.withValues(alpha: 0.85),
                foregroundColor: Palette.texte,
                minimumSize: const Size(48, 48),
              ),
              icon: Icon(enPause ? Icons.play_arrow : Icons.pause),
              onPressed: () {
                AudioService.instance.jouer(Effet.clic);
                jeu.basculerPause();
              },
            ),
          ),
        ),
      ),
    );
  }
}

// lib/main.dart
  Overlays.hud: (BuildContext c, DonjonGame g) => BoutonPauseOverlay(jeu: g),
```

**Explication :** un widget Flutter continue de recevoir les gestes quand le moteur Flame est en pause : le même bouton peut donc entrer **et** sortir de la pause. La taille minimale de 48 par 48 points respecte les recommandations d'accessibilité tactile. Attention : l'overlay `hud` est retiré quand l'état passe à `pause` — pour que ce bouton reste visible pendant la pause, il faut soit l'ajouter aussi à l'écran de pause, soit forcer `overlays.add(Overlays.hud)` dans l'état `pause`. La seconde solution est celle qu'attend l'énoncé.

### Correction 6

```dart
  factory StatistiquesPartie.calculerEnUnPassage(
    List<EvenementPartie> journal, {
    required double duree,
    required int score,
    required int meilleurScore,
  }) {
    // type -> (nombre d'occurrences, somme des valeurs)
    final Map<TypeEvenement, (int, int)> table =
        journal.fold<Map<TypeEvenement, (int, int)>>(
      <TypeEvenement, (int, int)>{},
      (Map<TypeEvenement, (int, int)> acc, EvenementPartie e) {
        final (int n, int somme) = acc[e.type] ?? (0, 0);
        acc[e.type] = (n + 1, somme + e.valeur);
        return acc;
      },
    );

    int n(TypeEvenement t) => table[t]?.$1 ?? 0;
    int s(TypeEvenement t) => table[t]?.$2 ?? 0;

    return StatistiquesPartie(
      duree: duree,
      score: score,
      meilleurScore: meilleurScore,
      pieces: n(TypeEvenement.pieceRamassee),
      pointsDesPieces: s(TypeEvenement.pieceRamassee),
      potions: n(TypeEvenement.potionBue),
      cles: n(TypeEvenement.cleRamassee),
      ennemisVaincus: n(TypeEvenement.ennemiVaincu),
      bossVaincus: n(TypeEvenement.bossVaincu),
      degatsSubis: s(TypeEvenement.degatSubi),
      viesPerdues: n(TypeEvenement.viePerdue),
      niveauxTermines: n(TypeEvenement.niveauTermine),
    );
  }
```

**Explication :** le journal n'est parcouru qu'une fois. L'accumulateur est une `Map` dont la valeur est un enregistrement `(int, int)` : `$1` est le nombre d'occurrences, `$2` la somme. Les deux fonctions locales `n` et `s` rendent la construction aussi lisible qu'avant. Sur cent événements le gain est indétectable ; sur un journal de dix mille lignes — un mode infini, par exemple — il devient réel.

### Correction 7

```dart
  /// Temps passé dans chaque niveau, en secondes.
  static Map<int, double> tempsParNiveau(
    List<EvenementPartie> journal,
    double dureeTotale,
  ) {
    final List<EvenementPartie> fins = journal
        .where((EvenementPartie e) => e.type == TypeEvenement.niveauTermine)
        .toList();

    final Map<int, double> resultat = <int, double>{};
    double debut = 0;

    for (final EvenementPartie fin in fins) {
      resultat[fin.niveau] = fin.instant - debut;
      debut = fin.instant;
    }

    // Le niveau en cours au moment de la fin de partie n'a pas d'événement
    // `niveauTermine` : on lui attribue le reste du temps.
    final int dernier = fins.isEmpty ? 0 : fins.last.niveau + 1;
    if (dureeTotale > debut) {
      resultat[dernier] = dureeTotale - debut;
    }
    return resultat;
  }
```

**Explication :** les instants du journal sont **absolus** depuis le début de la partie. La durée d'un niveau est donc la différence entre deux instants consécutifs de fin. Le dernier niveau demande un traitement à part : le joueur y est mort ou y a gagné sans qu'un `niveauTermine` soit émis pour lui — sauf en cas de victoire, où le `terminerNiveau` final l'émet ; le test `dureeTotale > debut` couvre les deux cas sans doublon.

### Correction 8

```dart
// Dans Sauvegarde
  List<int> meilleursScores = <int>[];

// toJson :        'meilleursScores': meilleursScores,
// fromJson :
        meilleursScores: ((json['meilleursScores'] as List<dynamic>?) ??
                <dynamic>[])
            .map((dynamic v) => (v as num?)?.toInt() ?? 0)
            .toList(),

// Migration v1 -> v2 : le champ n'existait pas.
  static Map<String, dynamic> migrer(Map<String, dynamic> json) {
    final int version = (json['version'] as num?)?.toInt() ?? 1;
    if (version < 2) {
      final int ancien = (json['meilleurScore'] as num?)?.toInt() ?? 0;
      // On amorce le classement avec l'unique score connu.
      json['meilleursScores'] = ancien > 0 ? <int>[ancien] : <int>[];
      json['version'] = 2;
    }
    return json;
  }

// Dans SauvegardeService
  Future<void> enregistrerAuClassement(int score) async {
    if (score <= 0) {
      return;
    }
    final List<int> liste = <int>[...donnees.meilleursScores, score]
      ..sort((int a, int b) => b.compareTo(a));

    donnees.meilleursScores = liste.take(5).toList();
    if (score > donnees.meilleurScore) {
      donnees.meilleurScore = score;
    }
    await _ecrire();
  }
```

**Explication :** trois points méritent attention. D'abord, la migration : elle amorce le classement avec l'ancien `meilleurScore`, ce qui évite qu'un joueur de longue date voie son classement vide. Ensuite, `sort((a, b) => b.compareTo(a))` trie en ordre **décroissant** — l'inversion des opérandes est la façon idiomatique de le faire en Dart. Enfin, `take(5).toList()` tronque : `take` renvoie un `Iterable` paresseux, et sans `toList()` on stockerait une vue qui recalcule à chaque lecture.

### Correction 9

```dart
// Dans DonjonGame
  /// Score pondéré par la difficulté choisie.
  int get scorePondere => (score * difficulte).round();

  void _sauvegarderProgression() {
    final SauvegardeService s = SauvegardeService.instance;
    unawaited(s.enregistrerMeilleurScore(scorePondere));   // pondéré
    unawaited(s.enregistrerProgression(niveauCourant));
  }

// Dans TableauScores, sous le score brut
  Text(
    'Pondéré (x${difficulte.toStringAsFixed(2)}) : $scorePondere',
    style: const TextStyle(color: Palette.texteFaible, fontSize: 13),
  ),
```

**Explication :** le record persisté devient le score pondéré ; le score affiché en grand reste le score brut, qui est celui que le joueur a vu monter pendant la partie. À performance égale, un joueur en mode facile obtient `score * 0.75` et ne peut donc pas battre le record d'un joueur en mode difficile, qui obtient `score * 1.4`. Attention : `game.meilleurScore` est mis à jour par `ajouterScore` avec le score **brut** ; il faut donc soit pondérer aussi à cet endroit, soit ne comparer que les valeurs persistées. La seconde option est plus simple et c'est celle-ci qui est montrée.

### Correction 10

```dart
// lib/services/magasin_cle_valeur.dart
abstract interface class MagasinCleValeur {
  Future<String?> lire(String cle);
  Future<void> ecrire(String cle, String valeur);
  Future<void> effacer(String cle);
}

class MagasinPreferences implements MagasinCleValeur {
  final SharedPreferencesAsync _prefs = SharedPreferencesAsync();

  @override
  Future<String?> lire(String cle) => _prefs.getString(cle);

  @override
  Future<void> ecrire(String cle, String valeur) =>
      _prefs.setString(cle, valeur);

  @override
  Future<void> effacer(String cle) => _prefs.remove(cle);
}

/// Implémentation en mémoire, pour les tests. Aucun greffon de plateforme.
class MagasinMemoire implements MagasinCleValeur {
  MagasinMemoire([Map<String, String>? initial])
      : _donnees = <String, String>{...?initial};

  final Map<String, String> _donnees;

  @override
  Future<String?> lire(String cle) async => _donnees[cle];

  @override
  Future<void> ecrire(String cle, String valeur) async =>
      _donnees[cle] = valeur;

  @override
  Future<void> effacer(String cle) async => _donnees.remove(cle);
}
```

```dart
// SauvegardeService devient injectable
class SauvegardeService {
  SauvegardeService(this._magasin);
  static SauvegardeService instance = SauvegardeService(MagasinPreferences());

  final MagasinCleValeur _magasin;
  // ... le reste : _magasin.lire / _magasin.ecrire / _magasin.effacer
}
```

```dart
// test/sauvegarde_service_test.dart
void main() {
  test('premier lancement : valeurs par défaut', () async {
    final SauvegardeService s = SauvegardeService(MagasinMemoire());
    await s.initialiser();

    expect(s.pret, isTrue);
    expect(s.donnees.meilleurScore, 0);
    expect(s.donnees.niveauAtteint, 0);
    expect(s.sauvegardeCorrompue, isFalse);
  });

  test('relecture d\'une sauvegarde valide', () async {
    final MagasinMemoire magasin = MagasinMemoire();
    final SauvegardeService ecrivain = SauvegardeService(magasin);
    await ecrivain.initialiser();
    await ecrivain.enregistrerMeilleurScore(4820);
    await ecrivain.enregistrerProgression(2);

    final SauvegardeService lecteur = SauvegardeService(magasin);
    await lecteur.initialiser();

    expect(lecteur.donnees.meilleurScore, 4820);
    expect(lecteur.donnees.niveauAtteint, 2);
  });

  test('sauvegarde corrompue : valeurs par défaut et drapeau', () async {
    final MagasinMemoire magasin = MagasinMemoire(<String, String>{
      'donjon_de_dart.sauvegarde.v1': '{ pas du JSON',
    });
    final SauvegardeService s = SauvegardeService(magasin);
    await s.initialiser();

    expect(s.sauvegardeCorrompue, isTrue);
    expect(s.donnees.meilleurScore, 0);
    expect(s.pret, isTrue);              // le service reste utilisable
  });
}
```

**Explication :** c'est l'**injection de dépendance**, le motif d'architecture le plus rentable de tout le cours. Trois bénéfices immédiats. Les tests s'exécutent en millisecondes, sans émulateur ni `TestWidgetsFlutterBinding`. Le troisième test vérifie un chemin d'erreur qu'on ne peut pas provoquer autrement de façon fiable. Et le jour où vous remplacerez `shared_preferences` par un stockage en ligne, vous écrirez une quatrième implémentation de `MagasinCleValeur` sans toucher à `SauvegardeService`. Le prix à payer : une interface de trois méthodes et un paramètre de constructeur. Notez que `instance` n'est plus `final` — c'est ce qui permet à un test d'intégration de la remplacer.

---

## Et maintenant ?

**Le jeu est fini.**

Pas « presque fini », pas « fini côté code » : fini au sens où l'on peut le donner à quelqu'un. Il se lance sur un menu, il sonne — ou il s'en passe proprement —, il se met en pause quand le téléphone sonne, il annonce la défaite et la victoire, il compte ce que vous avez fait, et il se souvient de votre record. Les six critères d'acceptation du chapitre 35 sont remplis.

Vous avez écrit, en six chapitres, environ 4 700 lignes de Dart réparties sur trente fichiers, sans un seul octet d'image ni de son. Vous savez maintenant ce qu'il y a dans un jeu 2D, et surtout ce qu'il y a **autour** : les services, les états, la persistance, les écrans de fin. C'est cette couche-là qui manque à presque tous les projets d'étudiants, et c'est elle qui fait la différence à la soutenance.

La PARTIE 2D change de registre. On ne code plus le jeu : on l'encadre et on le livre.

Le chapitre 41 remonte d'un cran, avant le code. Vous y écrirez un vrai **cahier des charges** et un **Game Design Document** : le document qui décrit un jeu avant qu'il existe, qui tranche les questions coûteuses, qui organise les assets et le planning. Vous ferez l'exercice sur un jeu neuf — et vous verrez, rétrospectivement, tout ce que « Donjon de Dart » a coûté faute d'avoir eu ce document dès le premier jour.

Le chapitre 42 fermera la boucle : tests, profilage, optimisation, `flutter build apk`, `flutter build web`, et la présentation du projet.

Rendez-vous au chapitre suivant :

[41-PARTIE-2D—CAHIER-DES-CHARGES-ET-GAME-DESIGN-DOCUMENT.md](./41-PARTIE-2D—CAHIER-DES-CHARGES-ET-GAME-DESIGN-DOCUMENT.md)
