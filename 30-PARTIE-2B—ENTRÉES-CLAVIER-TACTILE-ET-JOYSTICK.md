# PARTIE 2B — LE MOTEUR FLAME
# CHAPITRE 30 — LES ENTRÉES : CLAVIER, TACTILE ET JOYSTICK

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitres 27 à 29 (installation de Flame, `FlameGame`, composants et cycle de vie, sprites et assets), chapitre 26 (gestionnaire d'entrées, machine à états), chapitre 23 (vecteurs et normalisation)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** contrôler le héros du « Donjon de Dart » au clavier sur ordinateur, au doigt et au joystick sur mobile, avec un bouton d'attaque, sans écrire deux fois la logique de déplacement.

---

## 30.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- énumérer les périphériques d'entrée disponibles sur les trois plateformes cibles ;
- expliquer pourquoi une **intention** ne doit jamais être confondue avec une **touche** ;
- brancher `KeyboardEvents` sur une classe `FlameGame` et choisir la bonne valeur de `KeyEventResult` ;
- distinguer `KeyDownEvent`, `KeyUpEvent` et `KeyRepeatEvent` ;
- nommer une touche avec `LogicalKeyboardKey` et comprendre la différence avec `PhysicalKeyboardKey` ;
- supporter simultanément les dispositions ZQSD, WASD et les flèches ;
- lire l'ensemble `keysPressed` plutôt que réagir à l'événement, et justifier ce choix ;
- utiliser `KeyboardHandler` sur un composant et `HasKeyboardHandlerComponents` sur le jeu, sans les mélanger avec `KeyboardEvents` ;
- écrire un héros qui se déplace au clavier, code complet et exécutable ;
- corriger le bug de la diagonale trop rapide par **normalisation** ;
- rendre un composant sensible au tap avec `TapCallbacks` et réagir à `onTapDown`, `onTapUp`, `onTapCancel` ;
- savoir que `TapDetector` **n'existe plus** au niveau du jeu en Flame 1.38.0 et connaître les deux remplacements ;
- convertir les coordonnées d'un tap entre repère local, repère du canvas et repère du monde ;
- déplacer un composant au doigt avec `DragCallbacks` ;
- réagir au survol avec `HoverCallbacks` et suivre le pointeur avec `PointerMoveCallbacks` ;
- gérer le clic droit avec `SecondaryTapCallbacks` et la molette avec `ScrollCallbacks` ;
- construire un `JoystickComponent` sans aucune image, avec knob, background et **zone morte** ;
- exploiter `relativeDelta`, `intensity` et `direction` ;
- placer le joystick dans le **HUD** et non dans le monde, et dire pourquoi ;
- construire un bouton d'attaque avec `HudButtonComponent` et ancrer un élément avec `HudMarginComponent` ;
- détecter la plateforme pour choisir les contrôles à afficher ;
- offrir un remappage des touches et assembler le tout dans le « Donjon de Dart ».

---

## 30.1 — Trois plateformes, trois façons de jouer

Un jeu Flutter se compile pour Android, pour iOS, pour le Web, pour Windows, pour macOS et pour Linux. C'est un avantage énorme et un piège immédiat : **le même jeu ne se joue pas de la même manière partout**.

Regardons ce dont vous disposez réellement.

| Périphérique | Ordinateur (bureau et Web sur PC) | Mobile (Android / iOS) | Web sur tablette |
| --- | --- | --- | --- |
| Clavier | oui, complet | non (sauf clavier Bluetooth) | non |
| Souris et molette | oui | non | non |
| Survol | oui | **impossible** : le doigt ne survole rien | non fiable |
| Clic droit | oui | non | non |
| Tactile multipoint | parfois (PC hybride) | oui, jusqu'à 10 doigts | oui |
| Manette | parfois | rarement | rarement |

Trois conséquences pratiques, à graver dès maintenant.

**Le survol n'existe pas sur mobile.** Un bouton qui ne s'éclaire qu'au survol est invisible pour un joueur sur téléphone. Le survol est un **bonus**, jamais un canal d'information indispensable.

**Le clavier n'existe pas sur mobile.** Si votre seule façon d'avancer est la touche `Z`, votre jeu ne se joue pas sur téléphone. Point.

**Le tactile n'a pas d'état « appuyé en continu ».** Sur un clavier, la touche `Z` reste enfoncée tant que le doigt appuie, et le système vous le dit. Sur un écran tactile, il n'y a pas de « touche avancer » : il faut la fabriquer, avec un joystick virtuel ou des boutons directionnels.

> **À retenir.** Vous n'écrirez pas un jeu clavier puis un jeu tactile. Vous écrirez **un** jeu, dont la logique ignore totalement le périphérique, et **deux** couches d'entrée qui alimentent cette logique.

---

## 30.2 — Le principe : traduire une entrée en intention

Au chapitre 26, vous avez construit un **gestionnaire d'entrées**. L'idée y était déjà : le code du héros ne doit jamais savoir quelle touche a été pressée. Il doit savoir ce que le joueur **veut**.

Reprenons ce raisonnement, parce qu'il est la clé de tout ce chapitre.

### Version fautive

```dart
// NE FAITES PAS CELA.
class Heros extends PositionComponent with KeyboardHandler {
  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (keysPressed.contains(LogicalKeyboardKey.keyQ)) {
      position.x -= 4;
    }
    if (keysPressed.contains(LogicalKeyboardKey.keyD)) {
      position.x += 4;
    }
    return true;
  }
}
```

Ce code a quatre défauts, tous rédhibitoires.

1. Le héros **dépend du clavier**. Sur mobile, il ne bouge plus, et il faut réécrire la classe.
2. Le déplacement de 4 pixels a lieu **par événement clavier**, pas par frame : la vitesse dépend de la fréquence de répétition du système d'exploitation. C'est le bug du chapitre 20 sous un nouveau déguisement.
3. On ne peut pas **remapper** les touches : `keyQ` est écrit en dur dans la classe du héros.
4. On ne peut pas **tester** le héros sans simuler un événement Flutter.

### Version correcte : deux couches

```text
   COUCHE 1 — LECTURE DU PÉRIPHÉRIQUE
   clavier : touche Z enfoncée ?   joystick : relativeDelta ?   bouton pressé ?
        │  traduit en
        ▼
   COUCHE 2 — INTENTION (ce que le joueur VEUT)
   Intentions { Vector2 direction;  bool attaque; }
        │  lue par
        ▼
   COUCHE 3 — LOGIQUE DU JEU
   Heros.update(dt) { position += intentions.direction * vitesse * dt; }
   Ne sait pas, et ne veut pas savoir, d'où vient l'intention.
```

En Dart, cela tient en une petite classe.

```dart
/// Ce que le joueur veut faire à cet instant.
/// Aucune référence au clavier, au tactile ou au joystick.
class Intentions {
  /// Direction souhaitée, dans [-1, 1] sur chaque axe.
  final Vector2 direction = Vector2.zero();

  /// Vrai pendant la frame où l'attaque est demandée.
  bool attaque = false;
}
```

Le héros ne lit plus qu'une chose : `position.add(intentions.direction * vitesse * dt);` dans son `update`.

Ce héros fonctionne au clavier, au joystick, à la manette, et même piloté par une intelligence artificielle qui remplirait `intentions` toute seule. **Vous n'écrirez la logique de déplacement qu'une seule fois.**

Nous mettrons cette architecture en place progressivement. Commençons par la source la plus simple : le clavier.

---

## 30.3 — Le clavier : `KeyboardEvents` au niveau du jeu

Flame propose **deux niveaux** de gestion du clavier, et ils sont **mutuellement exclusifs**.

| Niveau | Mixin à poser | Sur quoi | Quand l'utiliser |
| --- | --- | --- | --- |
| Jeu | `KeyboardEvents` | la classe `FlameGame` | un seul endroit décide, jeu simple |
| Composant | `KeyboardHandler` (+ `HasKeyboardHandlerComponents` sur le jeu) | un `Component` | chaque composant gère ses propres touches |

Commençons par le niveau jeu. Le mixin s'appelle `KeyboardEvents`, il vient de `package:flame/input.dart`, et il s'applique à une classe qui hérite de `Game`.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(GameWidget(game: DonjonJeu()));
}

class DonjonJeu extends FlameGame with KeyboardEvents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
  }

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    debugPrint('Touches enfoncées : ${keysPressed.length}');
    return KeyEventResult.handled;
  }
}
```

**Résultat :** appuyez sur `Z`, puis en gardant `Z` enfoncé appuyez sur `D`, puis relâchez tout.

```text
Touches enfoncées : 1
Touches enfoncées : 2
Touches enfoncées : 1
Touches enfoncées : 0
```

Trois remarques avant d'aller plus loin.

**Le focus.** Un widget Flutter ne reçoit les touches que s'il a le focus clavier. `GameWidget` possède un paramètre `autofocus`, qui vaut **`true` par défaut** : dans le cas courant, vous n'avez rien à faire. Si votre jeu cohabite avec des champs de texte Flutter, `GameWidget` accepte aussi un `focusNode` que vous pilotez vous-même.

**Les trois imports.** `KeyEvent`, `KeyDownEvent`, `KeyUpEvent`, `KeyRepeatEvent` et `LogicalKeyboardKey` viennent de `package:flutter/services.dart`. `KeyEventResult` vient de `package:flutter/widgets.dart` (donc aussi de `material.dart`). `KeyboardEvents` vient de Flame. Oublier `services.dart` est l'erreur numéro un du chapitre.

**La signature exacte.** Les tutoriels antérieurs à 2023 écrivent `onKeyEvent(RawKeyEvent event, ...)` : `RawKeyEvent` appartient à l'ancienne API Flutter. En Flame 1.38.0 la signature est `KeyEventResult onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed)`.

---

## 30.4 — `onKeyEvent` et son retour `KeyEventResult`

`onKeyEvent` doit renvoyer une valeur de l'énumération `KeyEventResult`, définie par Flutter. Elle indique **au système de focus de Flutter** ce qu'il doit faire ensuite de la touche.

| Valeur | Signification | Effet concret |
| --- | --- | --- |
| `KeyEventResult.handled` | « J'ai traité la touche. » | La touche ne remonte plus aux autres gestionnaires de focus situés autour du `GameWidget`. |
| `KeyEventResult.ignored` | « Ce n'est pas pour moi. » | La touche continue son chemin ; si personne ne la traite, le système peut émettre un signal sonore. |
| `KeyEventResult.skipRemainingHandlers` | « Je ne la traite pas, mais que personne d'autre ne la traite non plus. » | Arrête la propagation sans marquer l'événement comme consommé. |

Concrètement, dans un jeu, la règle est simple : on consomme les touches du jeu, on laisse passer les autres.

```dart
@override
KeyEventResult onKeyEvent(
  KeyEvent event,
  Set<LogicalKeyboardKey> keysPressed,
) {
  const touchesDuJeu = {
    LogicalKeyboardKey.arrowLeft, LogicalKeyboardKey.arrowRight,
    LogicalKeyboardKey.arrowUp, LogicalKeyboardKey.arrowDown,
    LogicalKeyboardKey.space,
  };
  // Le jeu consomme la touche : la page Web ne défilera pas.
  if (touchesDuJeu.contains(event.logicalKey)) return KeyEventResult.handled;
  // Tab, F5, Ctrl+R... ne nous concernent pas : on laisse passer.
  return KeyEventResult.ignored;
}
```

> **Pourquoi c'est important.** Sur le Web, la barre d'espace fait défiler la page et les flèches déplacent l'ascenseur. Si votre `onKeyEvent` renvoie `ignored` pour l'espace, le joueur saute **et** la page saute. Renvoyer `handled` sur les touches du jeu supprime ce comportement parasite.

Réciproquement, renvoyer systématiquement `handled` pour **toutes** les touches est également une faute : la touche `Tab` ne permettrait plus de sortir du jeu au clavier, ce qui pose un vrai problème d'accessibilité.

---

## 30.5 — `KeyDownEvent`, `KeyUpEvent`, `KeyRepeatEvent`

Le paramètre `event` est de type `KeyEvent`. C'est une classe abstraite : l'objet réel est toujours l'une de ses trois sous-classes.

| Sous-classe | Émise quand | Fréquence |
| --- | --- | --- |
| `KeyDownEvent` | la touche vient d'être enfoncée | une seule fois par appui |
| `KeyRepeatEvent` | la touche est maintenue et le système émet une répétition | plusieurs fois par seconde, cadence réglée par l'OS |
| `KeyUpEvent` | la touche vient d'être relâchée | une seule fois par relâchement |

On les distingue avec l'opérateur `is`, vu au chapitre 10.

```dart
@override
KeyEventResult onKeyEvent(
  KeyEvent event,
  Set<LogicalKeyboardKey> keysPressed,
) {
  if (event is KeyDownEvent) {
    debugPrint('APPUI    sur ${event.logicalKey.keyLabel}');
  } else if (event is KeyRepeatEvent) {
    debugPrint('RÉPÉTITION de ${event.logicalKey.keyLabel}');
  } else if (event is KeyUpEvent) {
    debugPrint('RELÂCHÉ  : ${event.logicalKey.keyLabel}');
  }
  return KeyEventResult.handled;
}
```

**Résultat :** maintenez la touche `A` enfoncée une seconde, puis relâchez.

```text
APPUI    sur A
RÉPÉTITION de A
RÉPÉTITION de A
RÉPÉTITION de A
RÉPÉTITION de A
RÉPÉTITION de A
RELÂCHÉ  : A
```

Voici la chronologie sous forme de schéma.

```text
  doigt  ▁▁▁▁▁████████████████████████▁▁▁▁▁▁▁▁
              ↑    ↑   ↑   ↑   ↑   ↑  ↑
              │    └───┴───┴───┴───┘  └─ KeyUpEvent
              │      KeyRepeatEvent (cadence OS)
              └───── KeyDownEvent
```

### Le piège de `KeyRepeatEvent`

La cadence de répétition n'est **pas** la cadence du jeu. Elle est fixée dans les préférences du système d'exploitation, elle démarre après un délai d'environ 500 ms et elle diffère d'une machine à l'autre. Sur le Web, elle peut même être absente.

Un code du type :

```dart
// NE FAITES PAS CELA.
if (event is KeyDownEvent || event is KeyRepeatEvent) {
  position.x += 4; // avance de 4 pixels par répétition
}
```

produit un déplacement saccadé, avec un temps mort d'une demi-seconde au démarrage, et une vitesse qui dépend des réglages du joueur. C'est exactement ce que la section 30.9 va corriger.

### Le cas particulier des touches modificatrices

`Shift`, `Control`, `Alt` et `Meta` existent en deux exemplaires, et `keysPressed` contient la version **précise** (`shiftLeft`), pas le synonyme `shift`. Pour tester « un Shift quelconque » :

```dart
final bool courir = keysPressed.contains(LogicalKeyboardKey.shiftLeft) ||
    keysPressed.contains(LogicalKeyboardKey.shiftRight);
```

---

## 30.6 — `LogicalKeyboardKey`

`LogicalKeyboardKey` identifie une touche par **le caractère qu'elle produit**, compte tenu de la disposition clavier du système. `LogicalKeyboardKey.keyA` désigne « la touche qui écrit A », où qu'elle soit physiquement.

Il existe une seconde famille, `PhysicalKeyboardKey`, qui identifie une touche par **sa position physique**, indépendamment de la disposition.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │            LOGIQUE contre PHYSIQUE : le cas AZERTY               │
  └──────────────────────────────────────────────────────────────────┘

  Clavier QWERTY                    Clavier AZERTY
  ┌───┬───┬───┐                     ┌───┬───┬───┐
  │ Q │ W │ E │                     │ A │ Z │ E │
  ├───┼───┼───┤                     ├───┼───┼───┤
  │ A │ S │ D │                     │ Q │ S │ D │
  └───┴───┴───┘                     └───┴───┴───┘

  La touche en haut au milieu :
     PhysicalKeyboardKey.keyW  dans les DEUX cas (même position)
     LogicalKeyboardKey.keyW   sur QWERTY
     LogicalKeyboardKey.keyZ   sur AZERTY
```

Conséquence directe pour un jeu francophone : si vous testez `LogicalKeyboardKey.keyW`, un joueur en AZERTY devra appuyer sur la touche marquée `W`, c'est-à-dire en bas à gauche, à côté du `X`. Ce n'est évidemment pas ce que l'on veut.

**Le paramètre `keysPressed` que Flame vous transmet est un `Set<LogicalKeyboardKey>`.** C'est donc l'identité *logique* qui vous est offerte. L'objet `event` expose les deux :

```dart
@override
KeyEventResult onKeyEvent(
  KeyEvent event,
  Set<LogicalKeyboardKey> keysPressed,
) {
  if (event is KeyDownEvent) {
    debugPrint('logique  : ${event.logicalKey.keyLabel}');
    debugPrint('physique : ${event.physicalKey.debugName}');
  }
  return KeyEventResult.handled;
}
```

**Résultat** sur un clavier AZERTY, touche en haut à gauche du bloc de lettres (marquée `A`) :

```text
logique  : A
physique : Key Q
```

### Constantes utiles

`LogicalKeyboardKey.keyA` à `keyZ` (les 26 lettres), `digit0` à `digit9`,
`arrowLeft` / `arrowRight` / `arrowUp` / `arrowDown`, `space`, `enter`,
`escape`, `tab`, `shiftLeft` / `shiftRight`, `controlLeft` / `controlRight`.

> **Bonne pratique.** Pour un jeu, la stratégie robuste consiste à accepter **plusieurs** touches pour la même intention. C'est l'objet de la section 30.7.

---

## 30.7 — Le jeu de touches ZQSD / WASD / flèches

Un joueur francophone attend `ZQSD`. Un joueur anglophone attend `WASD`. Un joueur pressé utilise les flèches. Un jeu poli accepte **les trois**.

La bonne structure de données est le `Set<LogicalKeyboardKey>`, vu au chapitre 6 : appartenance en temps constant, pas de doublon.

```dart
/// Les touches acceptées pour chaque direction.
/// ZQSD (AZERTY) + WASD (QWERTY) + flèches.
class TouchesDeDeplacement {
  static const Set<LogicalKeyboardKey> haut = {
    LogicalKeyboardKey.keyZ,     // AZERTY
    LogicalKeyboardKey.keyW,     // QWERTY
    LogicalKeyboardKey.arrowUp,
  };

  static const Set<LogicalKeyboardKey> bas = {
    LogicalKeyboardKey.keyS,
    LogicalKeyboardKey.arrowDown,
  };

  static const Set<LogicalKeyboardKey> gauche = {
    LogicalKeyboardKey.keyQ,     // AZERTY
    LogicalKeyboardKey.keyA,     // QWERTY
    LogicalKeyboardKey.arrowLeft,
  };

  static const Set<LogicalKeyboardKey> droite = {
    LogicalKeyboardKey.keyD,
    LogicalKeyboardKey.arrowRight,
  };
}
```

Le test se fait alors avec `intersection`, méthode de `Set` vue au chapitre 6.

```dart
bool estActive(Set<LogicalKeyboardKey> enfoncees, Set<LogicalKeyboardKey> voulues) {
  return enfoncees.intersection(voulues).isNotEmpty;
}
```

Ou, plus court et sans allocation intermédiaire :

```dart
bool estActive(Set<LogicalKeyboardKey> enfoncees, Set<LogicalKeyboardKey> voulues) {
  return voulues.any(enfoncees.contains);
}
```

Voici le tableau de correspondance à afficher dans l'écran d'aide de votre jeu.

| Intention | AZERTY | QWERTY | Flèches |
| --- | --- | --- | --- |
| haut | `Z` | `W` | `↑` |
| bas | `S` | `S` | `↓` |
| gauche | `Q` | `A` | `←` |
| droite | `D` | `D` | `→` |
| attaquer | `Espace` | `Espace` | `Espace` |
| pause | `Échap` | `Échap` | `Échap` |

Notez que `S` et `D` sont communs aux deux dispositions : seules deux touches diffèrent réellement.

---

## 30.8 — `keysPressed` : lire l'état plutôt que l'événement

Il y a deux façons radicalement différentes de se servir de `onKeyEvent`.

**Façon 1 — réagir à l'événement.** On regarde `event`, on agit immédiatement.

```dart
if (event is KeyDownEvent && event.logicalKey == LogicalKeyboardKey.space) {
  attaquer();
}
```

**Façon 2 — enregistrer l'état.** On regarde `keysPressed`, on stocke le résultat, et c'est `update(dt)` qui agit.

```dart
@override
KeyEventResult onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
  _vaAGauche = TouchesDeDeplacement.gauche.any(keysPressed.contains);
  _vaADroite = TouchesDeDeplacement.droite.any(keysPressed.contains);
  return KeyEventResult.handled;
}

@override
void update(double dt) {
  super.update(dt);
  if (_vaAGauche) position.x -= vitesse * dt;
  if (_vaADroite) position.x += vitesse * dt;
}
```

`keysPressed` est un **instantané** : à chaque appel, Flame vous remet l'ensemble complet des touches actuellement maintenues. Ce n'est pas une liste des touches de l'événement en cours, c'est l'état du clavier.

Vérifions-le en affichant, à chaque événement, le contenu trié de `keysPressed` :

```dart
final noms = keysPressed.map((k) => k.keyLabel).toList()..sort();
debugPrint('état du clavier : $noms');
```

**Résultat :** enfoncer `Z`, puis `D` (sans relâcher `Z`), puis relâcher `Z`, puis relâcher `D`.

```text
état du clavier : [Z]
état du clavier : [D, Z]
état du clavier : [D]
état du clavier : []
```

> **À retenir.** L'événement dit **ce qui vient de changer**. `keysPressed` dit **où on en est**. Pour un jeu, on a presque toujours besoin de la seconde information.

### Le cas où l'événement reste indispensable

Certaines actions sont **ponctuelles** et ne doivent pas se répéter tant que la touche est maintenue : ouvrir la pause, tirer une flèche, valider un dialogue. Pour celles-là, `KeyDownEvent` est la bonne source, car `keysPressed.contains(escape)` lu dans `update` ferait basculer la pause 60 fois par seconde.

```dart
if (event is KeyDownEvent && event.logicalKey == LogicalKeyboardKey.escape) {
  basculerPause(); // exactement une fois par appui
}
```

| Type d'action | Source à utiliser |
| --- | --- |
| Déplacement continu, viser, courir | `keysPressed` lu dans `update` |
| Sauter, tirer, ouvrir la pause, valider | `KeyDownEvent` |
| Relâcher une corde d'arc, arrêter de charger | `KeyUpEvent` |

---

## 30.9 — Pourquoi lire l'état est meilleur pour un déplacement continu

Cette section démontre le point le plus important du chapitre. Comparons trois implémentations du même déplacement.

### Implémentation A — dans l'événement, sans `dt`

```dart
// NE FAITES PAS CELA.
@override
KeyEventResult onKeyEvent(KeyEvent e, Set<LogicalKeyboardKey> keys) {
  if (keys.contains(LogicalKeyboardKey.keyD)) {
    heros.position.x += 4;
  }
  return KeyEventResult.handled;
}
```

Le héros n'avance **que** lorsqu'un événement clavier survient : 4 px au premier appui, puis plus rien pendant une demi-seconde, puis des sauts de 4 px à la cadence de répétition de l'OS. Le mouvement est haché et la vitesse dépend d'un réglage que vous ne contrôlez pas.

### Implémentation B — dans `update`, sans `dt`

```dart
// NE FAITES PAS CELA NON PLUS.
@override
void update(double dt) {
  super.update(dt);
  if (_vaADroite) position.x += 4;
}
```

Le mouvement est fluide. Mais 4 pixels **par frame** signifie 240 px/s à 60 FPS et 480 px/s à 120 FPS : le jeu est deux fois plus rapide sur un téléphone haut de gamme. C'est le bug du chapitre 20.

### Implémentation C — dans `update`, avec `dt`

```dart
// LA BONNE VERSION.
@override
void update(double dt) {
  super.update(dt);
  if (_vaADroite) position.x += vitesse * dt; // vitesse en pixels/seconde
}
```

Récapitulons.

| | Fluidité | Indépendant du FPS | Indépendant des réglages OS |
| --- | --- | --- | --- |
| A : événement, sans `dt` | non | non | **non** |
| B : `update`, sans `dt` | oui | **non** | oui |
| C : `update`, avec `dt` | oui | oui | oui |

Et voici la raison de fond : les événements clavier arrivent de façon **irrégulière**, imposée par le système, alors que les frames du jeu arrivent de façon **régulière**, environ soixante fois par seconde. Le mouvement appartient à la seconde ligne du temps ; le clavier ne fait que mettre à jour un booléen sur la première.

> **La règle du chapitre.** `onKeyEvent` **écrit** un état. `update(dt)` **lit** cet état et déplace. Aucun déplacement ne s'écrit jamais dans `onKeyEvent`.

---

## 30.10 — `KeyboardHandler` au niveau du composant

Le mixin `KeyboardEvents` place toute la logique clavier dans la classe du jeu. Cela fonctionne, mais cela recrée le problème du chapitre 26 : un point central qui connaît tout le monde.

Flame propose donc un second mécanisme : **le composant reçoit lui-même les touches**. Le mixin s'appelle `KeyboardHandler` et s'applique à un `Component`.

```dart
import 'package:flame/components.dart';
import 'package:flutter/services.dart';

class Heros extends PositionComponent with KeyboardHandler {
  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    // ...
    return true;
  }
}
```

Attention à la différence de signature avec le niveau jeu. Ce n'est pas un détail.

| | Niveau jeu | Niveau composant |
| --- | --- | --- |
| Mixin | `KeyboardEvents` (sur `FlameGame`) | `KeyboardHandler` (sur `Component`) |
| Signature | `KeyEventResult onKeyEvent(...)` | `bool onKeyEvent(...)` |
| Retour | `handled` / `ignored` / `skipRemainingHandlers` | `true` = « je laisse l'événement continuer », `false` = « je bloque la propagation » |
| Destinataire du retour | le **système de focus de Flutter** | **l'arbre de composants de Flame** |

C'est **l'inverse** de l'intuition héritée de `KeyEventResult.handled`. Retenez la formule : **`true` = « je continue de propager »**.

### Exemple : deux héros, un seul écoute

```dart
class Heros extends RectangleComponent with KeyboardHandler {
  Heros(this.nom) : super(size: Vector2.all(32));

  final String nom;

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (event is KeyDownEvent) {
      debugPrint('$nom voit la touche ${event.logicalKey.keyLabel}');
    }
    return false; // on bloque : le composant suivant ne verra rien
  }
}
```

**Résultat avec `return false;` :**

```text
Blanc voit la touche A
```

**Résultat avec `return true;` :**

```text
Blanc voit la touche A
Rouge voit la touche A
```

> **Remarque.** L'ordre de propagation suit l'arbre de composants, en profondeur, dans l'ordre de rendu inverse. Ne construisez jamais une logique de jeu qui dépende finement de cet ordre : si deux composants doivent réagir, renvoyez `true` partout et laissez chacun décider pour lui-même.

---

## 30.11 — `HasKeyboardHandlerComponents`

`KeyboardHandler` sur un composant ne suffit pas. Il faut encore que **quelqu'un distribue** les événements clavier dans l'arbre. Ce quelqu'un est le mixin `HasKeyboardHandlerComponents`, posé sur le jeu.

```dart
class DonjonJeu extends FlameGame with HasKeyboardHandlerComponents {}
```

Ce mixin implémente `onKeyEvent` à votre place et le fait descendre dans tout l'arbre, vers chaque composant portant `KeyboardHandler`.

```text
   Flutter  ──KeyEvent + keysPressed──▶  DonjonJeu
                                          (HasKeyboardHandlerComponents)
                                              │ propage aux KeyboardHandler
      ┌───────────────────────────────────────┴───────────────┐
      ▼                                                       ▼
   CameraComponent → Viewport → HudAttaque   → true       World
                                                            ├── Heros    → true
                                                            ├── Gobelin  (pas de mixin)
                                                            └── Coffre   → false  ⛔
                                                  (à partir d'ici, plus rien ne reçoit)
```

### La règle absolue : jamais les deux à la fois

Le code source de Flame contient cette assertion, dans le mixin `KeyboardEvents` :

```text
A keyboard event was registered by KeyboardEvents for a game also
mixed with HasKeyboardHandlerComponents. Do not mix with both,
HasKeyboardHandlerComponents removes the necessity of KeyboardEvents
```

Autrement dit :

```dart
// INTERDIT — assertion déclenchée en mode debug.
class DonjonJeu extends FlameGame
    with KeyboardEvents, HasKeyboardHandlerComponents {}
```

```dart
// CORRECT — l'un OU l'autre.
class DonjonJeu extends FlameGame with HasKeyboardHandlerComponents {}
```

### Peut-on quand même intercepter au niveau du jeu ?

Oui. `HasKeyboardHandlerComponents` fournit un `onKeyEvent` que vous pouvez **surcharger**, à condition d'appeler `super` pour ne pas casser la propagation.

```dart
class DonjonJeu extends FlameGame with HasKeyboardHandlerComponents {
  bool enPause = false;

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    // Touche globale : la pause ne concerne aucun composant en particulier.
    if (event is KeyDownEvent &&
        event.logicalKey == LogicalKeyboardKey.escape) {
      enPause = !enPause;
      enPause ? pauseEngine() : resumeEngine();
      return KeyEventResult.handled;
    }

    // Tout le reste descend vers les composants.
    return super.onKeyEvent(event, keysPressed);
  }
}
```

C'est le schéma que nous garderons pour le « Donjon de Dart » : le jeu gère les touches **globales** (pause, menu), les composants gèrent leurs touches **propres** (déplacement, attaque).

### La variante déclarative : `KeyboardListenerComponent`

Flame fournit un composant tout fait qui associe des touches à des fonctions. Il porte déjà `KeyboardHandler`, donc le jeu doit toujours porter `HasKeyboardHandlerComponents`.

```dart
await add(
  KeyboardListenerComponent(
    keyDown: {
      LogicalKeyboardKey.keyQ: (keysPressed) { direction.x = -1; return true; },
      LogicalKeyboardKey.keyD: (keysPressed) { direction.x =  1; return true; },
    },
    keyUp: {
      LogicalKeyboardKey.keyQ: (keysPressed) { direction.x = 0; return true; },
      LogicalKeyboardKey.keyD: (keysPressed) { direction.x = 0; return true; },
    },
  ),
);
```

Le type exact des fonctions attendues est, dans le code source de Flame :

```dart
typedef KeyHandlerCallback = bool Function(Set<LogicalKeyboardKey>);
```

Chaque fonction reçoit l'ensemble des touches enfoncées et renvoie un `bool` de propagation, exactement comme `KeyboardHandler.onKeyEvent`.

> **Limite de cette variante.** Confortable pour des actions ponctuelles (`Espace` = attaquer), lourde pour un déplacement : il faut écrire chaque touche deux fois, et le bug de « je relâche `D` alors que `Q` est toujours enfoncé, et le héros s'arrête » revient très vite. Pour le déplacement, préférez la lecture de `keysPressed`.

---

## 30.12 — Déplacer le joueur au clavier (code complet)

Voici le premier programme complet du chapitre. Il n'utilise **aucune image** : le héros est un rectangle, les murs sont des rectangles, le sol est un rectangle.

Créez `lib/main.dart` avec ce contenu.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: DonjonJeu()));

/// Les touches acceptées, par intention.
class Touches {
  static const haut = {LogicalKeyboardKey.keyZ, LogicalKeyboardKey.keyW,
    LogicalKeyboardKey.arrowUp};
  static const bas = {LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown};
  static const gauche = {LogicalKeyboardKey.keyQ, LogicalKeyboardKey.keyA,
    LogicalKeyboardKey.arrowLeft};
  static const droite = {LogicalKeyboardKey.keyD,
    LogicalKeyboardKey.arrowRight};
}

class DonjonJeu extends FlameGame with HasKeyboardHandlerComponents {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();

    // Le repère du monde commence en haut à gauche de l'écran.
    camera.viewfinder.anchor = Anchor.topLeft;

    // Le sol du donjon : un simple rectangle sombre.
    await world.add(
      RectangleComponent(
        position: Vector2(40, 40),
        size: Vector2(560, 360),
        paint: Paint()..color = const Color(0xFF2E2E45),
      ),
    );
    await world.add(Heros(position: Vector2(320, 220)));

    await camera.viewport.add(
      TextComponent(
        text: 'ZQSD / WASD / flèches pour vous déplacer',
        position: Vector2(12, 12),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 14, color: Color(0xFFBBBBCC)),
        ),
      ),
    );
  }
}

class Heros extends RectangleComponent with KeyboardHandler {
  Heros({required Vector2 position})
      : super(
          position: position,
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  /// Vitesse en PIXELS PAR SECONDE.
  static const double vitesse = 180;

  /// Direction voulue par le joueur : mise à jour par onKeyEvent,
  /// utilisée par update.
  final Vector2 _direction = Vector2.zero();

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    // On LIT L'ÉTAT du clavier. On ne déplace rien ici.
    final bool haut = Touches.haut.any(keysPressed.contains);
    final bool bas = Touches.bas.any(keysPressed.contains);
    final bool gauche = Touches.gauche.any(keysPressed.contains);
    final bool droite = Touches.droite.any(keysPressed.contains);

    _direction
      ..x = (droite ? 1 : 0) - (gauche ? 1 : 0)
      ..y = (bas ? 1 : 0) - (haut ? 1 : 0);

    return true; // on laisse l'événement continuer sa route
  }

  @override
  void update(double dt) {
    super.update(dt);

    // Le déplacement a lieu ICI, et nulle part ailleurs.
    if (!_direction.isZero()) {
      position.add(_direction * vitesse * dt);
    }
  }
}
```

**Résultat :** une fenêtre sombre, un carré doré au centre, qui se déplace tant qu'une touche de direction est maintenue et s'arrête immédiatement au relâchement.

### Ce qu'il faut noter dans ce code

**`camera.viewfinder.anchor = Anchor.topLeft;`** place l'origine du monde en haut à gauche, comme le `Canvas` du chapitre 21. Sans cette ligne, l'origine est au centre de l'écran.

**`world.add` pour le décor et le héros, `camera.viewport.add` pour le texte.** Le texte reste collé à l'écran ; le reste appartient au monde (chapitre 31).

**L'astuce `(droite ? 1 : 0) - (gauche ? 1 : 0)`** transforme deux booléens en une composante valant `-1`, `0` ou `+1`. Si les deux touches opposées sont enfoncées, le résultat vaut `0`.

**`_direction` est un champ, pas une variable locale.** Il survit entre `onKeyEvent` et `update` : c'est le gestionnaire d'entrées du chapitre 26, réduit à sa plus simple expression.

**Aucune image, aucun `assets/`.** `RectangleComponent` et `TextComponent` suffisent à tout tester.

---

## 30.13 — Les diagonales et la normalisation

Lancez le programme précédent et maintenez `D` **et** `S` en même temps. Le héros part en diagonale. Il part surtout **trop vite**.

Voici pourquoi, avec les vecteurs du chapitre 23.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │           LE BUG DE LA DIAGONALE, EN UNE FIGURE                  │
  └──────────────────────────────────────────────────────────────────┘

   Touche D seule            Touches D + S
   direction = (1, 0)        direction = (1, 1)
   longueur  = 1             longueur  = √(1² + 1²) = √2 ≈ 1.414

        ────▶                     ────▶
        1.0                       │  ╲
                                  ▼   ╲ 1.414
                                       ╲

   À vitesse = 180 px/s :
      horizontalement : 180 × 1.000 = 180 px/s
      en diagonale    : 180 × 1.414 = 254 px/s     <-- 41 % plus rapide
```

Le joueur qui découvre cela se déplace en permanence en diagonale, parce que c'est plus rapide. Le jeu est cassé.

### La correction : normaliser

**Normaliser** un vecteur, c'est le ramener à une longueur de 1 en conservant sa direction. Le paquet `vector_math` fournit deux méthodes :

| Méthode | Effet | Renvoie |
| --- | --- | --- |
| `v.normalize()` | **modifie** `v` sur place | l'ancienne longueur (`double`) |
| `v.normalized()` | ne touche pas à `v` | un **nouveau** `Vector2` de longueur 1 |

Attention : normaliser le vecteur nul provoque une division par zéro et produit des `NaN`. Il faut donc toujours tester avant.

```dart
@override
void update(double dt) {
  super.update(dt);

  if (_direction.isZero()) {
    return; // pas de normalisation possible, et rien à faire
  }

  // normalized() renvoie une COPIE de longueur 1 : _direction est préservé.
  final Vector2 pas = _direction.normalized() * vitesse * dt;
  position.add(pas);
}
```

**Résultat :** la vitesse est identique dans les huit directions — 180 px/s en haut, en bas, à gauche, à droite **et** en diagonale, au lieu de 254 px/s en diagonale sans normalisation.

### Le piège inverse : ne normalisez pas une entrée analogique

Un joystick virtuel ou une manette renvoient une direction dont la longueur porte une information : **l'intensité de la poussée**. Un joueur qui pousse le joystick à moitié veut marcher, pas courir.

```dart
// Clavier : direction binaire → il FAUT normaliser.
position.add(_direction.normalized() * vitesse * dt);

// Joystick : direction analogique → il NE FAUT PAS normaliser.
position.add(joystick.relativeDelta * vitesse * dt);
```

Nous reviendrons sur ce point en 30.25. Retenez la règle :

> **Normalisez ce qui vient d'un clavier. Ne normalisez jamais ce qui vient d'un joystick.**

---

## 30.14 — Le tactile : `TapCallbacks` sur un composant

Passons au doigt. En Flame 1.38.0, le tactile fonctionne **au niveau du composant**, avec le mixin `TapCallbacks`, importé de `package:flame/events.dart`.

Point capital, et c'est une bonne nouvelle : **aucun mixin n'est nécessaire sur la classe du jeu**. Vous posez `TapCallbacks` sur le composant, et c'est tout.

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';

class Coffre extends RectangleComponent with TapCallbacks {
  Coffre({required Vector2 position})
      : super(
          position: position,
          size: Vector2(48, 36),          // OBLIGATOIRE : sans taille, pas de tap
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF8A5A2B),
        );

  bool ouvert = false;

  @override
  void onTapDown(TapDownEvent event) {
    ouvert = !ouvert;
    paint.color = ouvert
        ? const Color(0xFFF0C36B)
        : const Color(0xFF8A5A2B);
  }
}
```

**Résultat :** un rectangle brun devient doré au premier tap, redevient brun au second.

### Comment Flame décide quel composant est touché

Le mécanisme repose sur la méthode `containsLocalPoint(Vector2 point)` du composant. `PositionComponent` en fournit déjà une implémentation : elle teste si le point est dans le rectangle `size`.

```text
  1. Flutter transmet la position du doigt à Flame.
  2. Flame parcourt l'arbre du PREMIER PLAN vers le FOND.
  3. Pour chaque composant portant TapCallbacks, il convertit la position
     dans le repère LOCAL du composant puis appelle containsLocalPoint().
  4. Le PREMIER composant qui répond vrai reçoit onTapDown ; la propagation
     s'arrête là.
  5. Sauf si ce composant écrit event.continuePropagation = true : les
     composants situés en dessous reçoivent alors l'événement à leur tour.
```

> **L'erreur numéro un du tactile.** Un `PositionComponent` a par défaut une `size` de `Vector2.zero()`. Un rectangle de largeur 0 et de hauteur 0 ne contient **aucun** point. Le composant ne recevra donc **jamais** de tap, sans le moindre message d'erreur. Si votre `onTapDown` ne se déclenche pas, vérifiez la `size` avant toute chose.

### Le jeu lui-même peut recevoir les taps

`FlameGame` est un `Component` et implémente `containsLocalPoint`. On peut donc écrire :

```dart
class DonjonJeu extends FlameGame with TapCallbacks {
  @override
  void onTapDown(TapDownEvent event) {
    debugPrint('tap sur le fond, en ${event.canvasPosition}');
  }
}
```

C'est le moyen propre de réagir à un tap « n'importe où sur l'écran », par exemple pour passer un écran-titre.

---

## 30.15 — `onTapDown`, `onTapUp`, `onTapCancel`

Le mixin `TapCallbacks` propose quatre méthodes. Aucune n'est obligatoire : celles que vous ne surchargez pas ne font rien.

```dart
mixin TapCallbacks on Component {
  void onTapDown(TapDownEvent event) {}
  void onLongTapDown(TapDownEvent event) {}
  void onTapUp(TapUpEvent event) {}
  void onTapCancel(TapCancelEvent event) {}
}
```

| Méthode | Déclenchée quand | Événement reçu |
| --- | --- | --- |
| `onTapDown` | le doigt se pose sur le composant | `TapDownEvent` |
| `onLongTapDown` | le doigt reste posé ≈ 300 ms (réglable par `TapConfig.longTapDelay`) | `TapDownEvent` |
| `onTapUp` | le doigt se lève **sur** le composant, tap réussi | `TapUpEvent` |
| `onTapCancel` | le tap est abandonné : le doigt glisse et le geste devient un drag, ou un autre détecteur gagne | `TapCancelEvent` |

Voici la chronologie des trois scénarios.

```text
  TAP RÉUSSI    doigt posé ──────────── doigt levé
                onTapDown               onTapUp

  TAP ANNULÉ    doigt posé ── glisse ── doigt levé ailleurs
                onTapDown    onTapCancel     (onTapUp jamais appelé)

  TAP LONG      doigt posé ── 300 ms ── doigt levé
                onTapDown   onLongTapDown    onTapUp
```

### Un bouton correct utilise les trois

Un bouton qui ne surcharge que `onTapDown` reste « enfoncé » pour toujours si le joueur fait glisser son doigt hors du bouton.

```dart
class BoutonAttaque extends RectangleComponent with TapCallbacks {
  BoutonAttaque({required super.position})
      : super(size: Vector2.all(64), anchor: Anchor.center);

  static final Paint _repos = Paint()..color = const Color(0xFFAA3344);
  static final Paint _presse = Paint()..color = const Color(0xFFFF6677);

  @override
  void onTapDown(TapDownEvent event) => paint = _presse;

  @override
  void onTapUp(TapUpEvent event) {
    paint = _repos;
    debugPrint('Attaque !'); // l'action a lieu au RELÂCHEMENT
  }

  @override
  void onTapCancel(TapCancelEvent event) => paint = _repos; // annulation propre
}
```

**Résultat :** posez le doigt, le bouton s'éclaire ; glissez hors du bouton et relâchez, il reprend sa couleur **sans** déclencher l'attaque.

> **Question de conception.** Déclencher au `down` ou au `up` ? Pour un bouton d'action de jeu (tirer, sauter), déclenchez au **`down`** : la réactivité prime. Pour un bouton d'interface (valider, quitter), déclenchez au **`up`** : le joueur peut se raviser en glissant le doigt à côté. Nous appliquerons la première règle au bouton d'attaque en 30.27.

### `onTapCancel` ne porte presque aucune information

`TapCancelEvent` n'expose qu'un `pointerId`. Il n'a pas de position : le geste est déjà parti ailleurs. Ne cherchez pas `event.localPosition` dessus, il n'existe pas.

---

## 30.16 — `TapDetector` au niveau du jeu

Vous trouverez, dans de très nombreux tutoriels et sur des pages de documentation non mises à jour, ce code :

```dart
// CE CODE NE COMPILE PLUS EN FLAME 1.38.0.
class MonJeu extends FlameGame with TapDetector {
  @override
  bool onTapDown(TapDownInfo info) {
    // ...
    return true;
  }
}
```

**Le mixin `TapDetector` n'existe plus.** Le fichier `package:flame/src/gestures/detectors.dart` de Flame 1.38.0 ne déclare plus que sept mixins de niveau jeu : `VerticalDragDetector`, `HorizontalDragDetector`, `ForcePressDetector`, `PanDetector`, `ScaleDetector`, `MouseMovementDetector` et `ScrollDetector`. Aucun détecteur de tap simple n'y figure, et la documentation officielle ajoute cet avertissement général : « Detectors will be deprecated in the future. Prefer `Callbacks` instead. »

### Les deux remplacements corrects

**Solution 1 — `TapCallbacks` sur le jeu lui-même.** C'est la solution recommandée, déjà vue en 30.14.

```dart
class DonjonJeu extends FlameGame with TapCallbacks {
  @override
  void onTapDown(TapDownEvent event) {
    debugPrint('tap n° ${event.pointerId} en ${event.canvasPosition}');
  }
}
```

**Solution 2 — `MultiTouchTapDetector`, si vous avez besoin du multipoint.** Ce mixin de niveau jeu, lui, existe bel et bien. Il numérote chaque doigt.

```dart
class DonjonJeu extends FlameGame with MultiTouchTapDetector {
  @override
  void onTapDown(int pointerId, TapDownInfo info) {
    debugPrint('doigt $pointerId posé en ${info.eventPosition.global}');
  }

  @override
  void onTapUp(int pointerId, TapUpInfo info) => debugPrint('doigt $pointerId levé');

  @override
  void onTapCancel(int pointerId) => debugPrint('doigt $pointerId annulé');
}
```

Notez les différences avec `TapCallbacks` :

| | `TapCallbacks` (composant) | `MultiTouchTapDetector` (jeu) |
| --- | --- | --- |
| Où | sur un `Component` | sur la classe `Game` |
| Signature | `void onTapDown(TapDownEvent)` | `void onTapDown(int pointerId, TapDownInfo)` |
| Type d'info | `TapDownEvent` | `TapDownInfo` (ancienne famille `*Info`) |
| Multipoint | oui, via `event.pointerId` | oui, via le paramètre `pointerId` |
| Recommandé | **oui** | seulement pour des besoins multipoints particuliers |

Et un avertissement de la documentation : **on ne mélange pas** les détecteurs avancés (`MultiTouch*`) avec les détecteurs simples de même nature. Flutter déclenche une assertion, parce que les deux se disputent l'« arène de gestes ».

---

## 30.17 — Coordonnées locales et globales d'un tap

Un tap a lieu à un endroit. Mais « à quel endroit » dépend du repère dont on parle. `TapDownEvent` en expose trois.

| Propriété | Repère | Origine | Utile pour |
| --- | --- | --- | --- |
| `event.devicePosition` | l'appareil | coin de la fenêtre / de l'écran | presque jamais |
| `event.canvasPosition` | le `GameWidget` | coin haut-gauche du canvas du jeu | placer un élément de HUD |
| `event.localPosition` | le composant touché | coin haut-gauche du composant | savoir **où** dans le composant |

```text
  (0,0) appareil
   ┌──────────────────────────────────────────┐
   │  barre de titre / encoche                │
   │  ┌───────────────────────────────────┐   │
   │  │ (0,0) canvas du GameWidget        │   │
   │  │        ┌──────────────────┐       │   │
   │  │        │(0,0) le composant│       │   │
   │  │        │         ✱ le tap │       │   │
   │  │        └──────────────────┘       │   │
   │  └───────────────────────────────────┘   │
   └──────────────────────────────────────────┘

   devicePosition ≈ (330, 275)
   canvasPosition ≈ (330, 235)
   localPosition  ≈ ( 50,  35)
```

### Démonstration

```dart
@override
void onTapDown(TapDownEvent event) {
  debugPrint('device : ${event.devicePosition}');
  debugPrint('canvas : ${event.canvasPosition}');
  debugPrint('local  : ${event.localPosition}');
  // On peut décider différemment selon la moitié touchée.
  debugPrint(event.localPosition.x < size.x / 2 ? 'moitié : gauche'
                                                : 'moitié : droite');
}
```

**Résultat** pour un clic proche du bord gauche d'un coffre de 120 × 80 :

```text
device : [212.0,187.0]
canvas : [212.0,187.0]
local  : [12.0,27.0]
moitié : gauche
```

### Et les coordonnées du monde ?

`canvasPosition` est en pixels d'écran. Si votre caméra a un zoom ou suit le héros, l'écran et le monde ne coïncident plus. Pour convertir, `CameraComponent` fournit :

```dart
// Écran → monde
final Vector2 dansLeMonde = camera.globalToLocal(event.canvasPosition);

// Monde → écran
final Vector2 aLEcran = camera.localToGlobal(heros.position);
```

Exemple : faire apparaître une potion à l'endroit exact du clic, dans le monde.

```dart
@override
void onTapDown(TapDownEvent event) {
  world.add(
    CircleComponent(
      radius: 8,
      position: camera.globalToLocal(event.canvasPosition),
      anchor: Anchor.center,
      paint: Paint()..color = const Color(0xFF66DD88),
    ),
  );
}
```

**Résultat :** un petit disque vert apparaît sous le doigt et reste à sa place dans le donjon, même si la caméra se déplace ensuite.

Le chapitre 31 approfondira ces conversions. Retenez pour l'instant : **`localPosition` pour l'intérieur d'un composant, `canvasPosition` pour le HUD, `camera.globalToLocal` pour le monde.**

---

## 30.18 — `DragCallbacks` : `onDragStart`, `onDragUpdate`, `onDragEnd`

Glisser le doigt est un geste distinct du tap. Le mixin s'appelle `DragCallbacks` et vient également de `package:flame/events.dart`.

```dart
mixin DragCallbacks on Component {
  bool get isDragged;                            // vrai pendant le glissement
  void onDragStart(DragStartEvent event) {}      // @mustCallSuper
  void onDragUpdate(DragUpdateEvent event) {}
  void onDragEnd(DragEndEvent event) {}          // @mustCallSuper
  void onDragCancel(DragCancelEvent event) {}    // @mustCallSuper
}
```

Trois des quatre méthodes sont annotées `@mustCallSuper` dans le code source de Flame : elles tiennent à jour le drapeau interne `isDragged`. **Si vous les surchargez, appelez `super`.** `onDragUpdate` fait exception : elle est vide, vous pouvez la surcharger librement.

| Méthode | Déclenchée quand |
| --- | --- |
| `onDragStart` | le doigt se pose **et** commence à bouger, sur ce composant |
| `onDragUpdate` | le doigt bouge, à chaque nouvelle position |
| `onDragEnd` | le doigt se lève ; l'événement porte une vitesse de fin |
| `onDragCancel` | le geste est interrompu par le système ou par un autre détecteur ; **aucune vitesse** |

### Les propriétés de `DragUpdateEvent`

C'est ici que les débutants se trompent le plus. `DragUpdateEvent` expose **trois déplacements** et **six positions**.

| Propriété | Type | Sens |
| --- | --- | --- |
| `localDelta` | `Vector2` | déplacement depuis la mise à jour précédente, repère **local** du composant |
| `canvasDelta` / `deviceDelta` | `Vector2` | le même déplacement, repère du canvas / de l'appareil |
| `localStartPosition` / `localEndPosition` | `Vector2` | positions dans le repère local |
| `canvasStartPosition` / `canvasEndPosition` | `Vector2` | positions dans le repère du canvas |
| `deviceStartPosition` / `deviceEndPosition` | `Vector2` | positions dans le repère de l'appareil |
| `pointerId` / `timestamp` | `int` / `Duration` | numéro du doigt, horodatage |
| `continuePropagation` | `bool` | fait redescendre l'événement aux composants inférieurs |

> **Point d'attention documenté.** Une fois le glissement commencé, les événements continuent d'être livrés au composant **même si le doigt sort de ses limites**. Dans ce cas, `localPosition` contient des `NaN`, alors que `canvasPosition` et `devicePosition` restent valides. Ne testez donc jamais une position locale pendant un drag sans vérifier `isNaN`.

---

## 30.19 — Déplacer un composant au doigt

Voici le programme complet. Un coffre que l'on traîne dans le donjon.

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: DonjonJeu()));

class DonjonJeu extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Coffre(position: Vector2(150, 150)));
    await world.add(Coffre(position: Vector2(350, 250)));
  }
}

class Coffre extends RectangleComponent with DragCallbacks {
  Coffre({required Vector2 position})
      : super(
          position: position,
          size: Vector2(64, 48),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF8A5A2B),
        );

  static final Paint _repos = Paint()..color = const Color(0xFF8A5A2B);
  static final Paint _traine = Paint()..color = const Color(0xFFF0C36B);

  @override
  void onDragStart(DragStartEvent event) {
    super.onDragStart(event); // OBLIGATOIRE : @mustCallSuper
    paint = _traine;
    priority = 10;            // passe au premier plan pendant le glissement
  }

  @override
  void onDragUpdate(DragUpdateEvent event) {
    // localDelta : le déplacement depuis la dernière mise à jour.
    position.add(event.localDelta);
  }

  @override
  void onDragEnd(DragEndEvent event) {
    super.onDragEnd(event);   // OBLIGATOIRE
    paint = _repos;
    priority = 0;
  }

  @override
  void onDragCancel(DragCancelEvent event) {
    super.onDragCancel(event); // OBLIGATOIRE
    paint = _repos;
    priority = 0;
  }
}
```

**Résultat :** deux rectangles bruns. Posez le doigt sur l'un, il devient doré et suit le doigt. Relâchez, il reprend sa couleur et reste où vous l'avez laissé.

### Pourquoi `localDelta` et pas `localEndPosition` ?

Tentation naturelle : « le doigt est là, mets le coffre là ». Si l'on écrit `position = event.canvasStartPosition;`, le coffre se recentre brutalement sous le doigt dès le premier mouvement, alors qu'ajouter `localDelta` conserve l'écart initial entre le doigt et le centre du coffre. Le glissement devient naturel.

### Le piège du drag sur un composant qui bouge

Si le composant traîné suit une logique dans `update` (gravité, vélocité), les deux se combattent. On désactive la physique pendant le glissement grâce au getter du mixin :

```dart
@override
void update(double dt) {
  super.update(dt);
  if (isDragged) return; // le doigt commande, pas la gravité
  velocite.y += gravite * dt;
  position.add(velocite * dt);
}
```

Enfin, `DragCallbacks` gère naturellement plusieurs doigts : chaque composant reçoit les événements du doigt qui l'a saisi, identifié par `event.pointerId`. Deux joueurs peuvent traîner deux coffres en même temps, sans une ligne de code supplémentaire.

---

## 30.20 — `HoverCallbacks` et la souris

Sur ordinateur, la souris peut **survoler** sans cliquer. C'est une information précieuse : mettre un bouton en surbrillance, afficher le nom d'un ennemi, changer le curseur.

Le mixin `HoverCallbacks` fournit trois méthodes et un getter.

| Membre | Rôle |
| --- | --- |
| `isHovered` | vrai tant que le pointeur est au-dessus du composant |
| `onHoverEnter()` | le pointeur vient d'entrer |
| `onHoverExit()` | le pointeur vient de sortir |
| `onHoverCancel()` | le survol est interrompu par l'appui d'un bouton de souris (nouveauté 1.38.0) |

Notez qu'aucune de ces trois méthodes ne reçoit d'événement : elles sont sans paramètre.

```dart
class Gobelin extends CircleComponent with HoverCallbacks {
  Gobelin({required Vector2 position})
      : super(
          radius: 20,
          position: position,
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF3FA34D),
        );

  late final TextComponent _etiquette = TextComponent(
    text: 'Gobelin',
    position: Vector2(radius, -6),
    anchor: Anchor.bottomCenter,
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 12, color: Color(0xFFFFFFFF)),
    ),
  );

  @override
  void onHoverEnter() {
    paint.color = const Color(0xFF7FE08D);
    add(_etiquette);
  }

  @override
  void onHoverExit() {
    paint.color = const Color(0xFF3FA34D);
    _etiquette.removeFromParent();
  }

  @override
  void onHoverCancel() => onHoverExit(); // un bouton a été enfoncé
}
```

**Résultat :** sur ordinateur, passer la souris sur le disque vert l'éclaircit et fait apparaître le mot « Gobelin » au-dessus. Sur mobile, rien ne se produit jamais — et c'est normal.

### `onHoverCancel`, la nouveauté de la 1.38.0

Flutter n'émet pas d'événements de survol pendant qu'un bouton de souris est maintenu : l'état de survol se termine dès le début de l'appui. Sans `onHoverCancel`, un composant survolé puis cliqué restait « éclairé » indéfiniment, car `onHoverExit` n'était jamais appelé.

> **Rappel de conception.** Le survol est un **confort**. Aucune information vitale ne doit passer par lui, sinon votre jeu devient injouable sur téléphone.

---

## 30.21 — `PointerMoveCallbacks`

`HoverCallbacks` répond à la question « suis-je survolé, oui ou non ? ». Elle ne dit pas **où** est le pointeur. Pour cela, il faut le mixin de plus bas niveau, dont `HoverCallbacks` est d'ailleurs construit.

```dart
mixin PointerMoveCallbacks on Component {
  void onPointerMove(PointerMoveEvent event) {}
  void onPointerMoveStop(PointerMoveEvent event) {}
}
```

| Méthode | Déclenchée quand |
| --- | --- |
| `onPointerMove` | le pointeur bouge **au-dessus** du composant |
| `onPointerMoveStop` | le composant était survolé et le pointeur vient de partir |

L'événement `PointerMoveEvent` de Flame (attention à ne pas le confondre avec la classe du même nom dans `package:flutter/gestures.dart`) expose les positions habituelles : `localPosition`, `canvasPosition`, `devicePosition`.

Exemple : un canon qui vise le curseur. Le composant parent est une zone carrée invisible ; l'enfant `_canon` pivote.

```dart
import 'dart:math';

class Tourelle extends RectangleComponent with PointerMoveCallbacks {
  Tourelle({required Vector2 position})
      : super(
          position: position,
          size: Vector2(160, 160),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0x22FFFFFF),
        );

  late final RectangleComponent _canon = RectangleComponent(
    position: size / 2,
    size: Vector2(48, 8),
    anchor: Anchor.centerLeft,
    paint: Paint()..color = const Color(0xFFDD5544),
  );

  @override
  Future<void> onLoad() async => add(_canon);

  @override
  void onPointerMove(PointerMoveEvent event) {
    final Vector2 vers = event.localPosition - size / 2;
    if (vers.length2 > 0.01) {
      _canon.angle = atan2(vers.y, vers.x); // angle EN RADIANS
    }
  }

  @override
  void onPointerMoveStop(PointerMoveEvent event) => _canon.angle = 0;
}
```

**Résultat :** le petit rectangle rouge pivote pour pointer vers la souris tant qu'elle survole la zone claire, et revient à l'horizontale quand elle en sort.

> **Rappel du chapitre 21.** `angle` est en **radians**, jamais en degrés. `atan2(y, x)` renvoie déjà des radians, il n'y a donc aucune conversion à faire.

Il existe également un détecteur au niveau du jeu, `MouseMovementDetector`, avec la méthode `void onMouseMove(PointerHoverInfo info)`. Il appartient à l'ancienne famille des détecteurs, appelée à être dépréciée : préférez `PointerMoveCallbacks`.

---

## 30.22 — Le clic droit et la molette

Deux entrées propres à l'ordinateur, très utiles pour un jeu de stratégie ou un éditeur de niveau.

### Le clic droit : `SecondaryTapCallbacks`

```dart
mixin SecondaryTapCallbacks on Component {
  void onSecondaryTapDown(SecondaryTapDownEvent event) {}
  void onSecondaryTapUp(SecondaryTapUpEvent event) {}
  void onSecondaryTapCancel(SecondaryTapCancelEvent event) {}
}
```

Il existe un troisième bouton, celui de la molette enfoncée, géré par `TertiaryTapCallbacks` avec `onTertiaryTapDown`, `onTertiaryTapUp` et `onTertiaryTapCancel`.

```dart
class Coffre extends RectangleComponent
    with TapCallbacks, SecondaryTapCallbacks {
  Coffre({required super.position})
      : super(size: Vector2(64, 48), anchor: Anchor.center);

  @override
  void onTapDown(TapDownEvent event) => debugPrint('clic gauche : ouvrir');

  @override
  void onSecondaryTapDown(SecondaryTapDownEvent event) =>
      debugPrint('clic droit : inspecter');
}
```

Un même composant peut porter plusieurs mixins d'entrée : ils sont indépendants et ne se gênent pas.

### La molette : `ScrollCallbacks`

```dart
mixin ScrollCallbacks on Component {
  void onScroll(ScrollEvent event) {}
}
```

`ScrollEvent` expose `scrollDelta` (un `Vector2` en pixels logiques), plus `localPosition`, `canvasPosition` et `devicePosition`.

Usage classique : le zoom de la caméra.

```dart
class DonjonJeu extends FlameGame with ScrollCallbacks {
  @override
  void onScroll(ScrollEvent event) {
    // scrollDelta.y est POSITIF quand on fait défiler vers le bas.
    final double facteur = event.scrollDelta.y > 0 ? 0.9 : 1.1;
    camera.viewfinder.zoom =
        (camera.viewfinder.zoom * facteur).clamp(0.5, 3.0);
  }
}
```

**Résultat :** trois crans de molette vers le haut puis deux vers le bas donnent les zooms 1.10, 1.21, 1.33, 1.20, 1.08.

`clamp`, vu au chapitre 3, empêche le joueur de zoomer à l'infini ou de retourner l'image.

Tous ces mixins s'importent depuis `package:flame/events.dart` et se posent sur un `Component`. Aucun ne demande quoi que ce soit à la classe du jeu. Le tableau complet figure dans le résumé du chapitre.

---

## 30.23 — `JoystickComponent`

Sur mobile, il n'y a pas de clavier. Le standard du jeu d'action tactile est le **joystick virtuel** : un disque fixe sur lequel on pose le pouce, et un disque plus petit qui suit le doigt.

Flame en fournit un tout fait : `JoystickComponent`, importé de `package:flame/input.dart`.

Voici son constructeur réel.

```dart
JoystickComponent({
  PositionComponent? knob,        // le disque mobile
  PositionComponent? background,  // le disque fixe
  Vector2? position,
  EdgeInsets? margin,
  double? size,
  double? knobRadius,
  Anchor anchor = Anchor.center,
  Iterable<Component>? children,
  int? priority,
  ComponentKey? key,
});
```

Deux assertions sont vérifiées à la construction, directement dans le code source de Flame :

1. `size != null || background != null` — il faut **soit** une taille, **soit** un fond, sinon le joystick n'a aucune dimension.
2. Les positions du `knob` et du `background` doivent rester à zéro — le joystick les place lui-même, dans `onMount`.

### Un joystick sans aucune image

Les exemples officiels utilisent des sprites. Nous n'en avons pas, et nous n'en avons pas besoin : `knob` et `background` acceptent **n'importe quel `PositionComponent`**, donc un `CircleComponent`.

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: DonjonJeu()));

class DonjonJeu extends FlameGame {
  late final JoystickComponent joystick;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    joystick = JoystickComponent(
      knob: CircleComponent(
        radius: 26,
        paint: Paint()..color = const Color(0xCCE8B04B),
      ),
      background: CircleComponent(
        radius: 64,
        paint: Paint()..color = const Color(0x55FFFFFF),
      ),
      margin: const EdgeInsets.only(left: 32, bottom: 32),
    );

    await world.add(Heros(joystick: joystick));

    // Le joystick va dans le HUD : il ne bouge pas avec la caméra.
    await camera.viewport.add(joystick);
  }
}

class Heros extends RectangleComponent {
  Heros({required this.joystick})
      : super(
          position: Vector2(300, 200),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final JoystickComponent joystick;

  static const double vitesse = 200; // pixels/seconde

  @override
  void update(double dt) {
    super.update(dt);
    if (joystick.direction != JoystickDirection.idle) {
      position.add(joystick.relativeDelta * vitesse * dt);
    }
  }
}
```

**Résultat :** un grand disque translucide en bas à gauche, avec un disque doré au centre. Posez le doigt dessus et tirez : le disque doré suit, et le carré doré se déplace dans la même direction.

### `margin` ou `position` ?

`JoystickComponent` porte le mixin `ComponentViewportMargin`. Il accepte donc deux modes de placement, exclusifs : `margin: const EdgeInsets.only(left: 32, bottom: 32)` maintient le joystick à 32 px du bord gauche et du bas quelle que soit la taille de la fenêtre, tandis que `position: Vector2(96, 400)` fait calculer une fois pour toutes la marge correspondante. Sur mobile, l'orientation peut changer et la barre de navigation apparaître : **utilisez toujours `margin`**.

Attention à la règle exacte du calcul, lisible dans le code source :

```text
   si margin.left != 0  →  la position en X part du bord GAUCHE
   sinon                →  la position en X part du bord DROIT (margin.right)
   si margin.top  != 0  →  la position en Y part du bord HAUT
   sinon                →  la position en Y part du bord BAS (margin.bottom)
```

Donc `EdgeInsets.only(left: 32, bottom: 32)` place bien le joystick en bas à gauche. Et `EdgeInsets.all(32)` le placerait en **haut** à gauche, puisque `left` et `top` sont tous deux non nuls.

---

## 30.24 — Le knob, le background et la zone morte

### Anatomie

```text
                 knobRadius
            │◀───────────────▶│
      ╭─────┼─────────────────┼─────╮
     ╱      │      ╭───╮      │      ╲   ← background (fixe)
    │       │     │  ●  │◀────┼───────┼── knob (suit le doigt)
     ╲      │      ╰───╯      │      ╱
      ╰─────┼───────▲─────────┼─────╯
            │   épicentre     │

   delta         = vecteur épicentre → knob, en pixels
   knobRadius    = distance maximale parcourable par le knob
   relativeDelta = delta / knobRadius  →  longueur de 0.0 à 1.0
```

| Élément | Type | Rôle | Valeur par défaut |
| --- | --- | --- | --- |
| `background` | `PositionComponent?` | disque fixe ; sa `size` devient celle du joystick | aucun |
| `knob` | `PositionComponent?` | disque mobile ; **obligatoire** au montage | aucun |
| `knobRadius` | `double` | rayon de déplacement maximal du knob | `size.x / 2` |
| `size` | `double?` | taille du joystick si aucun `background` | — |

Le code source place le knob dans `onMount` :

```text
knob.anchor   = Anchor.center;
knob.position = size / 2;        // exactement au centre du joystick
```

C'est pour cela que vous ne devez pas fixer vous-même la position du knob : une assertion vous en empêche.

### La zone morte

Un pouce posé sur un écran n'est jamais parfaitement immobile. Sans précaution, le héros dérive lentement même quand le joueur croit ne rien faire. On corrige cela avec une **zone morte** (*dead zone*) : en dessous d'un certain seuil, on considère que le joystick est au repos.

**`JoystickComponent` n'a pas de zone morte intégrée.** C'est à vous de l'écrire. Elle tient en trois lignes.

```dart
class Heros extends RectangleComponent {
  Heros({required this.joystick})
      : super(
          position: Vector2(300, 200),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final JoystickComponent joystick;

  static const double vitesse = 200;

  /// En dessous de 15 % de poussée, on ignore le joystick.
  static const double zoneMorte = 0.15;

  @override
  void update(double dt) {
    super.update(dt);

    final Vector2 pousse = joystick.relativeDelta;
    if (pousse.length < zoneMorte) {
      return; // repos
    }
    position.add(pousse * vitesse * dt);
  }
}
```

```text
   0.00 ────────── 0.15 ──────────────────── 1.00
   repos      seuil (zone morte)     poussée maximale
   |<-- rien ne se passe -->|<-- le héros avance -->|
```

Une zone morte trop grande rend le joystick mou, trop petite le rend nerveux. Entre **0,10 et 0,20**, on est presque toujours bien.

---

## 30.25 — `joystick.relativeDelta` et `joystick.direction`

Le joystick expose quatre valeurs de lecture. Il faut savoir précisément ce que chacune signifie, parce qu'elles se ressemblent.

| Propriété | Type | Contenu |
| --- | --- | --- |
| `delta` | `Vector2` | déplacement du knob depuis son épicentre, **en pixels**, limité à `knobRadius` |
| `relativeDelta` | `Vector2` | `delta / knobRadius` : direction **et** intensité, longueur de 0 à 1 |
| `intensity` | `double` | mesure de poussée dans `[0, 1]` |
| `direction` | `JoystickDirection` | direction **discrète** parmi neuf valeurs |

### `relativeDelta` : la valeur à utiliser pour se déplacer

C'est la propriété centrale. Sa longueur porte l'intensité de la poussée, sa direction porte l'orientation.

```dart
// Déplacement analogique : pousser à moitié = avancer à mi-vitesse.
position.add(joystick.relativeDelta * vitesseMax * dt);
```

Exemples chiffrés, avec `knobRadius = 64` :

| Position du knob | `delta` | `relativeDelta` | Vitesse à 200 px/s |
| --- | --- | --- | --- |
| au centre | `(0, 0)` | `(0.00, 0.00)` | 0 px/s |
| 32 px à droite | `(32, 0)` | `(0.50, 0.00)` | 100 px/s |
| 64 px à droite (butée) | `(64, 0)` | `(1.00, 0.00)` | 200 px/s |
| butée en bas à droite | `(45.3, 45.3)` | `(0.71, 0.71)` | 200 px/s |

Remarquez la dernière ligne : **`delta` est déjà borné au cercle**, donc la longueur de `relativeDelta` ne dépasse jamais 1, y compris en diagonale. Le bug de la diagonale trop rapide de la section 30.13 **n'existe pas** avec un joystick. C'est précisément pour cela qu'il ne faut **pas** normaliser `relativeDelta`.

```dart
// FAUX : on détruit l'information d'intensité.
position.add(joystick.relativeDelta.normalized() * vitesse * dt);

// JUSTE.
position.add(joystick.relativeDelta * vitesse * dt);
```

### `intensity`

`intensity` est documentée comme « le pourcentage `[0.0, 1.0]` dont le knob est tiré du centre vers le bord ». Dans l'implémentation actuelle, elle est calculée ainsi :

```text
intensity = (longueur de delta)² / knobRadius²
```

C'est donc le **carré** du rapport de poussée. Elle croît lentement au début et vite à la fin. Pour un seuil de zone morte ou une jauge affichée à l'écran, la valeur la plus prévisible reste `joystick.relativeDelta.length`.

### `direction` : les huit secteurs

`JoystickDirection` est une énumération de neuf valeurs :

```dart
enum JoystickDirection {
  up, upLeft, upRight, right, down, downRight, downLeft, left, idle,
}
```

Le cercle est découpé en huit secteurs de 45 degrés, plus l'état `idle` lorsque `delta` est nul.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │              LES HUIT SECTEURS DE JoystickDirection              │
  └──────────────────────────────────────────────────────────────────┘

                        up
              upLeft  ╲  │  ╱  upRight
                       ╲ │ ╱
          left ─────────╳╳╳───────── right
                       ╱ │ ╲
            downLeft  ╱  │  ╲  downRight
                        down

           au centre exact : JoystickDirection.idle
```

`direction` est très utile pour choisir une **animation** (le héros regarde à gauche, à droite, en haut, en bas), là où `relativeDelta` sert au **déplacement**.

```dart
@override
void update(double dt) {
  super.update(dt);
  position.add(joystick.relativeDelta * vitesse * dt); // analogique

  switch (joystick.direction) {                        // discret
    case JoystickDirection.left:
    case JoystickDirection.upLeft:
    case JoystickDirection.downLeft:
      regardVersLaGauche = true;
    case JoystickDirection.right:
    case JoystickDirection.upRight:
    case JoystickDirection.downRight:
      regardVersLaGauche = false;
    default:
      break; // on garde l'orientation précédente
  }
}
```

### Affichage de contrôle

Pour comprendre le joystick, rien ne vaut un affichage en direct, mis à jour dans le `update` du jeu (l'exercice 8 en fait un programme complet) :

```dart
final d = joystick.relativeDelta;
sonde.text = 'relativeDelta = (${d.x.toStringAsFixed(2)}, '
    '${d.y.toStringAsFixed(2)})   longueur = ${d.length.toStringAsFixed(2)}   '
    'direction = ${joystick.direction.name}';
```

**Résultat** en tirant le joystick vers le haut à droite, à fond :

```text
relativeDelta = (0.71, -0.71)   longueur = 1.00   direction = upRight
```

Notez le signe : `y` est **négatif** vers le haut, comme sur le `Canvas` du chapitre 21.

---

## 30.26 — Placer le joystick dans le HUD, pas dans le monde

C'est l'erreur la plus fréquente, et la plus spectaculaire.

```dart
// NE FAITES PAS CELA.
await world.add(joystick);
```

```dart
// CORRECT.
await camera.viewport.add(joystick);
```

### Pourquoi

Le `World` est l'univers du jeu : le donjon, les murs, les gobelins. Il est **observé** par la caméra. Quand la caméra suit le héros, tout ce qui est dans le monde défile.

Le `Viewport` de la caméra est la **fenêtre** à travers laquelle on regarde. Ce qui y est ajouté reste collé à l'écran.

```text
  DANS LE MONDE (faux)          DANS LE VIEWPORT (juste)
  le héros avance ...           le héros avance ...

  ┌──────────────┐              ┌──────────────┐
  │      ▓▓      │              │      ▓▓      │
  │              │              │              │
  │              │              │  ╭──╮        │
  │              │              │ │ ● │        │
  └──────────────┘              └──╰──╯────────┘
   le joystick est SORTI          le joystick n'a PAS bougé
   de l'écran : il est resté
   dans le donjon
```

Un joystick placé dans le monde subit en plus le **zoom** de la caméra : à `zoom = 2`, il devient deux fois plus gros ; à `zoom = 0.5`, il devient minuscule.

### Où placer quoi

| Élément | Destination | Écriture |
| --- | --- | --- |
| Héros, ennemis, murs, potions | le monde | `world.add(...)` |
| Décor de fond fixe, derrière le monde | l'arrière-plan de la caméra | `camera.backdrop.add(...)` |
| Joystick, boutons, barre de vie, score | le HUD | `camera.viewport.add(...)` |
| Éléments solidaires du zoom de la caméra | le viseur | `camera.viewfinder.add(...)` |

Le constructeur de `CameraComponent` offre aussi le raccourci `hudComponents: [joystick, boutonAttaque]`.

### Vérification pratique

Ajoutez `camera.follow(heros);` au jeu de la section 30.23. Avec `world.add(joystick)`, le joystick s'échappe dès le premier pas ; avec `camera.viewport.add(joystick)`, il reste en bas à gauche. Testez les deux : ce genre de bug se retient bien mieux après l'avoir vu.

---

## 30.27 — `HudButtonComponent` : le bouton d'attaque

Il nous manque une action. Sur mobile, elle prend la forme d'un bouton posé en bas à droite, sous le pouce droit.

Flame fournit `HudButtonComponent`, importé de `package:flame/input.dart`.

```dart
HudButtonComponent({
  PositionComponent? button,       // apparence au repos
  PositionComponent? buttonDown,   // apparence pendant l'appui (optionnel)
  EdgeInsets? margin,
  Function()? onPressed,
  Function()? onReleased,
  Function()? onCancelled,
  Vector2? position,
  Vector2? size,
  Vector2? scale,
  double? angle,
  Anchor? anchor,
  Iterable<Component>? children,
  int? priority,
});
```

`HudButtonComponent` hérite de `ButtonComponent`, qui porte déjà `TapCallbacks`. Vous n'avez donc rien à ajouter : le bouton est cliquable dès sa construction.

| Callback | Déclenché par |
| --- | --- |
| `onPressed` | `onTapDown` — le doigt se pose |
| `onReleased` | `onTapUp` — le doigt se lève sur le bouton |
| `onCancelled` | `onTapCancel` — le doigt glisse ailleurs |

### Un bouton d'attaque sans image

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: DonjonJeu()));

class DonjonJeu extends FlameGame {
  final Heros heros = Heros();

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(heros);

    await camera.viewport.add(
      HudButtonComponent(
        button: CircleComponent(
          radius: 34,
          paint: Paint()..color = const Color(0xAAAA3344),
        ),
        buttonDown: CircleComponent(
          radius: 34,
          paint: Paint()..color = const Color(0xFFFF6677),
        ),
        margin: const EdgeInsets.only(right: 32, bottom: 32),
        onPressed: heros.attaquer,
      ),
    );
  }
}

class Heros extends RectangleComponent {
  Heros()
      : super(
          position: Vector2(280, 200),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  int coups = 0;

  void attaquer() => debugPrint('Coup d\'épée n° ${++coups}');
}
```

**Résultat :** un disque rouge sombre en bas à droite, qui devient rouge vif à l'appui.

```text
Coup d'épée n° 1
Coup d'épée n° 2
Coup d'épée n° 3
```

### Le comportement de `buttonDown`

Le code source de `ButtonComponent` retire l'enfant `button` et met `buttonDown` à sa place pendant l'appui, puis fait l'inverse au relâchement, à l'annulation comprise. Vous n'avez donc rien à gérer : aucun risque de bouton resté « enfoncé ».

Si vous ne fournissez pas de `buttonDown`, l'apparence ne change pas. Sur mobile, c'est un mauvais choix : le joueur n'a **aucun** retour visuel de son appui. Fournissez-en toujours un.

### Variante : surcharger plutôt que passer des callbacks

Pour un bouton qui porte un état, mieux vaut une sous-classe qui surcharge `onTapDown`, `onTapUp` et `onTapCancel`. Ces trois méthodes de `ButtonComponent` sont annotées `@mustCallSuper` : **oublier `super` casse l'échange des apparences**, et le bouton reste bloqué dans son état enfoncé. La section 30.31 en donne un exemple complet.

### Les composants de bouton disponibles

`ButtonComponent` (deux `PositionComponent`, position absolue), `HudButtonComponent` (le même, placé par marges), `SpriteButtonComponent` (deux `Sprite`), `AdvancedButtonComponent` (plusieurs « peaux » : `defaultSkin`, `downSkin`, `hoverSkin`, `disabledSkin`…) et `ToggleButtonComponent` (avec les états « sélectionné »). Pour un jeu 100 % code, `HudButtonComponent` avec des `CircleComponent` ou des `RectangleComponent` couvre tous les besoins.

---

## 30.28 — `HudMarginComponent` et le placement relatif aux bords

`JoystickComponent` et `HudButtonComponent` se placent par marges. Comment faire de même avec **votre** composant, par exemple une barre de vie ?

Réponse : `HudMarginComponent`, exporté par `package:flame/input.dart`.

```dart
class HudMarginComponent extends PositionComponent {
  HudMarginComponent({
    EdgeInsets? margin,
    Vector2? position,
    Vector2? size,
    Vector2? scale,
    double? angle,
    Anchor? anchor,
    Iterable<Component>? children,
    int? priority,
    ComponentKey? key,
  });
}
```

Une assertion impose : `margin != null || position != null`. Il faut l'un des deux.

### Le problème résolu

Un écran de téléphone peut mesurer 360 × 640 en portrait, puis 640 × 360 après rotation. Une barre de vie placée à `position: Vector2(240, 20)` sera hors de l'écran en portrait.

```text
  position: Vector2(560, 20)          margin: only(right: 16, top: 16)

  PAYSAGE 640×360   PORTRAIT 360×640  PAYSAGE      PORTRAIT
  ┌───────────┐     ┌──────┐          ┌───────────┐  ┌──────┐
  │      ▤▤▤▤ │     │      │  ▤▤▤▤    │      ▤▤▤▤ │  │ ▤▤▤▤ │
  │           │     │      │  ↑perdu  │           │  │      │
  └───────────┘     └──────┘          └───────────┘  └──────┘
```

### Une barre de vie ancrée en haut à droite

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: DonjonJeu()));

class DonjonJeu extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    final barre = BarreDeVie();
    await camera.viewport.add(barre);
    // Démonstration : on perd 5 points de vie par seconde.
    await add(TimerComponent(
      period: 1,
      repeat: true,
      onTick: () => barre.vie = (barre.vie - 5).clamp(0, 100),
    ));
  }
}

class BarreDeVie extends HudMarginComponent {
  BarreDeVie()
      : super(
          margin: const EdgeInsets.only(right: 16, top: 16),
          size: Vector2(160, 18),
          anchor: Anchor.topRight,
        );

  late final RectangleComponent _remplissage;

  int _vie = 100;
  int get vie => _vie;
  set vie(int valeur) {
    _vie = valeur;
    _remplissage.size.x = size.x * (_vie / 100);
  }

  @override
  Future<void> onLoad() async {
    _remplissage = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = const Color(0xFFCC3344),
    );
    await addAll([
      RectangleComponent(
        size: size.clone(),
        paint: Paint()..color = const Color(0x66000000),
      ),
      _remplissage,
    ]);
  }
}
```

**Résultat :** une barre rouge en haut à droite, qui se vide de 5 % par seconde et reste collée au coin même si vous redimensionnez la fenêtre.

### Le rappel important sur les marges

Deux points que le code source rend explicites, et qui surprennent. D'abord, **la marge est mesurée jusqu'à l'`anchor`, pas jusqu'au bord du composant** : avec `anchor: Anchor.topRight` et `right: 16`, c'est le coin haut-droit qui se trouve à 16 px du bord. Ensuite, **une marge à zéro bascule le calcul sur le bord opposé** : n'écrivez donc que les deux marges qui vous intéressent.

Enfin, une contrainte de montage vérifiée par une assertion : le parent d'un `HudMarginComponent` doit **fournir une taille**. `camera.viewport` en fournit une ; un simple `Component` non.

---

## 30.29 — Détecter la plateforme pour choisir les contrôles

Nous avons maintenant deux jeux de contrôles. Reste à décider lequel afficher.

Flutter fournit deux constantes dans `package:flutter/foundation.dart` :

| Constante | Type | Contenu |
| --- | --- | --- |
| `kIsWeb` | `bool` | vrai si l'application tourne dans un navigateur |
| `defaultTargetPlatform` | `TargetPlatform` | `android`, `iOS`, `fuchsia`, `linux`, `macOS`, `windows` |

```dart
import 'package:flutter/foundation.dart';

/// Vrai si l'appareil est un mobile tactile.
/// Sur le Web, defaultTargetPlatform renvoie le système hôte :
/// un iPad renvoie iOS, un PC renvoie windows.
bool get estMobile =>
    defaultTargetPlatform == TargetPlatform.android ||
    defaultTargetPlatform == TargetPlatform.iOS;
```

> **N'utilisez pas `dart:io`.** L'écriture `Platform.isAndroid` provoque une erreur de compilation sur le Web, car `dart:io` n'y existe pas. `defaultTargetPlatform` fonctionne partout.

### Application au jeu

```dart
class DonjonJeu extends FlameGame with HasKeyboardHandlerComponents {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    final heros = Heros();
    await world.add(heros);

    // Sur ordinateur, on n'affiche rien : le clavier suffit.
    if (!estMobile) return;

    await camera.viewport.add(
      JoystickComponent(
        knob: CircleComponent(
          radius: 26,
          paint: Paint()..color = const Color(0xCCE8B04B),
        ),
        background: CircleComponent(
          radius: 64,
          paint: Paint()..color = const Color(0x55FFFFFF),
        ),
        margin: const EdgeInsets.only(left: 32, bottom: 32),
      ),
    );
    await camera.viewport.add(
      HudButtonComponent(
        button: CircleComponent(
          radius: 34,
          paint: Paint()..color = const Color(0xAAAA3344),
        ),
        buttonDown: CircleComponent(
          radius: 34,
          paint: Paint()..color = const Color(0xFFFF6677),
        ),
        margin: const EdgeInsets.only(right: 32, bottom: 32),
        onPressed: heros.attaquer,
      ),
    );
  }
}
```

### La meilleure stratégie n'est pas la détection

La détection de plateforme a une limite connue : un PC hybride avec écran tactile, une tablette Android avec clavier Bluetooth, un joueur qui branche une manette. Elle donne une réponse binaire à une question qui ne l'est pas.

La stratégie robuste tient en deux points.

1. **Toujours brancher les deux sources d'entrée.** Le clavier ne coûte rien s'il n'est pas utilisé ; le joystick ne coûte presque rien s'il est là.
2. **Afficher les contrôles tactiles à la demande.** Soit par détection, comme ci-dessus, soit par une option du menu (« Afficher les contrôles tactiles »), soit automatiquement : on affiche le joystick au premier événement tactile et on le masque au premier événement clavier, en jouant sur `opacity` (fourni par le mixin `HasPaint`) ou en ajoutant et retirant le composant de l'arbre.

---

## 30.30 — Le remappage des touches

Au chapitre 26, vous avez séparé la **touche** de l'**action** dans le gestionnaire d'entrées. Reprenons cette idée dans Flame.

### La structure

```dart
/// Les actions du jeu, indépendantes de tout périphérique.
enum ActionJeu { haut, bas, gauche, droite, attaquer, pause }

/// La table qui associe une action à un ensemble de touches.
class Commandes {
  Commandes(this._table);

  final Map<ActionJeu, Set<LogicalKeyboardKey>> _table;

  /// La configuration par défaut : ZQSD + WASD + flèches.
  factory Commandes.parDefaut() => Commandes({
        ActionJeu.haut: {
          LogicalKeyboardKey.keyZ,
          LogicalKeyboardKey.keyW,
          LogicalKeyboardKey.arrowUp,
        },
        ActionJeu.bas: {LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown},
        ActionJeu.gauche: {
          LogicalKeyboardKey.keyQ,
          LogicalKeyboardKey.keyA,
          LogicalKeyboardKey.arrowLeft,
        },
        ActionJeu.droite: {
          LogicalKeyboardKey.keyD,
          LogicalKeyboardKey.arrowRight,
        },
        ActionJeu.attaquer: {LogicalKeyboardKey.space},
        ActionJeu.pause: {LogicalKeyboardKey.escape},
      });

  /// Vrai si au moins une touche de l'action est enfoncée.
  bool estActive(ActionJeu action, Set<LogicalKeyboardKey> enfoncees) =>
      _table[action]?.any(enfoncees.contains) ?? false;

  /// Remplace toutes les touches d'une action.
  void remapper(ActionJeu action, Set<LogicalKeyboardKey> nouvelles) {
    _table[action] = nouvelles;
  }

  /// Le libellé à afficher dans l'écran des options.
  String libelle(ActionJeu action) =>
      _table[action]?.map((t) => t.keyLabel).join(' / ') ?? 'non assignée';
}
```

### L'utilisation

```dart
@override
bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
  _direction
    ..x = (commandes.estActive(ActionJeu.droite, keysPressed) ? 1 : 0) -
        (commandes.estActive(ActionJeu.gauche, keysPressed) ? 1 : 0)
    ..y = (commandes.estActive(ActionJeu.bas, keysPressed) ? 1 : 0) -
        (commandes.estActive(ActionJeu.haut, keysPressed) ? 1 : 0);
  return true;
}
```

Le héros ne contient **plus une seule constante de touche**. Changer les commandes se fait de l'extérieur :

```dart
final commandes = Commandes.parDefaut();
commandes.remapper(ActionJeu.attaquer, {LogicalKeyboardKey.keyE});
```

### L'écran de configuration

Capturer la prochaine touche pressée pour l'assigner à une action est un cas d'usage classique.

```dart
class CapteurDeTouche extends Component with KeyboardHandler {
  CapteurDeTouche({required this.action, required this.commandes});

  final ActionJeu action;
  final Commandes commandes;

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    if (event is! KeyDownEvent) return true;
    if (event.logicalKey != LogicalKeyboardKey.escape) {
      commandes.remapper(action, {event.logicalKey});
    }
    removeFromParent();
    return false; // on consomme : personne d'autre ne voit cette touche
  }
}
```

**Résultat :** dans l'écran des options, cliquez sur « Attaquer », appuyez sur `E`, et le libellé devient :

```text
Attaquer : E
```

### Rappel de conception

Dans le code du héros, on ne veut lire que des intentions (« je vais à gauche », « j'attaque ») et ses propres constantes (« ma vitesse est de 180 px/s ») — jamais `LogicalKeyboardKey.keyQ`, `event is KeyDownEvent` ni `keysPressed.contains(...)`. Si vous respectez cette règle, ajouter le support d'une manette au chapitre 42 vous demandera une classe de plus, et **zéro modification** dans le héros.

---

## 30.31 — Mini-projet : les contrôles du « Donjon de Dart »

Assemblons tout. Le héros se déplace au clavier sur ordinateur **et** au joystick sur mobile, avec un bouton d'attaque tactile et la barre d'espace sur ordinateur. La logique de déplacement n'est écrite **qu'une fois**.

### L'architecture

```text
   ControleClavier (KeyboardHandler)     Joystick + BoutonAttaque
   ajouté au jeu                         dans camera.viewport
            │  écrit                              │  lu par
            ▼                                     ▼
        ┌──────────────────────────────────────────────┐
        │  Intentions                                  │
        │    Vector2 direction   (déjà normalisée)     │
        │    bool    attaque                           │
        └──────────────────────────────────────────────┘
                            │  lu par
                            ▼
                  Heros.update(dt) — une seule logique de déplacement
```

### Le programme complet

Aucun fichier image n'est nécessaire. Copiez ce contenu dans `lib/main.dart`.

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(GameWidget(game: DonjonDeDart()));
}

// ───────────────────────────────────────────── 1. LES INTENTIONS

/// Ce que le joueur veut, sans savoir d'où ça vient.
class Intentions {
  final Vector2 direction = Vector2.zero();
  bool attaque = false;
}

// ───────────────────────────────────────────── 2. LA SOURCE CLAVIER

/// Composant invisible : il traduit le clavier en intentions.
/// priority negative = mis à jour AVANT le héros.
class ControleClavier extends Component with KeyboardHandler {
  ControleClavier(this.intentions) : super(priority: -2);

  final Intentions intentions;

  static const _haut = {LogicalKeyboardKey.keyZ, LogicalKeyboardKey.keyW,
    LogicalKeyboardKey.arrowUp};
  static const _bas = {LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown};
  static const _gauche = {LogicalKeyboardKey.keyQ, LogicalKeyboardKey.keyA,
    LogicalKeyboardKey.arrowLeft};
  static const _droite = {LogicalKeyboardKey.keyD,
    LogicalKeyboardKey.arrowRight};
  static const _attaque = {LogicalKeyboardKey.space};

  /// État du clavier, mémorisé entre deux événements.
  final Vector2 _lecture = Vector2.zero();
  bool _espaceAvant = false;

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    // onKeyEvent ÉCRIT un état, il ne déplace rien.
    _lecture
      ..x = (_droite.any(keysPressed.contains) ? 1 : 0) -
          (_gauche.any(keysPressed.contains) ? 1 : 0)
      ..y = (_bas.any(keysPressed.contains) ? 1 : 0) -
          (_haut.any(keysPressed.contains) ? 1 : 0);

    // Attaque : front montant seulement, une fois par appui.
    final bool espace = _attaque.any(keysPressed.contains);
    if (espace && !_espaceAvant) intentions.attaque = true;
    _espaceAvant = espace;

    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_lecture.isZero()) return;
    // Clavier : direction binaire → NORMALISATION obligatoire.
    intentions.direction.setFrom(_lecture.normalized());
  }
}

// ───────────────────────────────────────────── 3. LA SOURCE JOYSTICK

/// Composant invisible : il traduit le joystick en intentions.
class ControleJoystick extends Component {
  ControleJoystick(this.intentions, this.joystick) : super(priority: -1);

  final Intentions intentions;
  final JoystickComponent joystick;

  static const double zoneMorte = 0.15;

  @override
  void update(double dt) {
    super.update(dt);
    final Vector2 pousse = joystick.relativeDelta;
    if (pousse.length < zoneMorte) {
      return; // on ne touche pas aux intentions : le clavier peut les remplir
    }
    // Joystick : direction analogique → PAS de normalisation.
    intentions.direction.setFrom(pousse);
  }
}

// ───────────────────────────────────────────── 4. LE BOUTON D'ATTAQUE

class BoutonAttaque extends HudButtonComponent {
  BoutonAttaque(this.intentions)
      : super(
          button: CircleComponent(
            radius: 34,
            paint: Paint()..color = const Color(0xAAAA3344),
          ),
          buttonDown: CircleComponent(
            radius: 34,
            paint: Paint()..color = const Color(0xFFFF6677),
          ),
          margin: const EdgeInsets.only(right: 32, bottom: 32),
        );

  final Intentions intentions;

  @override
  void onTapDown(TapDownEvent event) {
    super.onTapDown(event);
    intentions.attaque = true; // action immédiate au posé du doigt
  }
}

// ───────────────────────────────────────────── 5. LE HÉROS

class Heros extends RectangleComponent {
  Heros(this.intentions)
      : super(
          position: Vector2(300, 220),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Intentions intentions;

  static const double vitesse = 190;

  int coups = 0;
  double _eclat = 0;

  @override
  void update(double dt) {
    super.update(dt);

    // UNE SEULE logique de déplacement, quelle que soit la source.
    if (!intentions.direction.isZero()) {
      position.add(intentions.direction * vitesse * dt);
      position.x = position.x.clamp(56.0, 584.0);
      position.y = position.y.clamp(56.0, 384.0);
    }

    if (intentions.attaque) {
      intentions.attaque = false; // consommée
      coups++;
      _eclat = 0.15;
    }

    if (_eclat > 0) {
      _eclat -= dt;
      paint.color = const Color(0xFFFFFFFF);
    } else {
      paint.color = const Color(0xFFE8B04B);
    }

    // Le clavier remet la direction à zéro tout seul (au relâchement).
    // Le joystick, lui, ne produit rien quand il est au repos : on efface
    // donc la direction ici, chaque source la réécrira à la frame suivante.
    intentions.direction.setZero();
  }
}

// ───────────────────────────────────────────── 6. LE JEU

bool get estMobile =>
    defaultTargetPlatform == TargetPlatform.android ||
    defaultTargetPlatform == TargetPlatform.iOS;

class DonjonDeDart extends FlameGame with HasKeyboardHandlerComponents {
  final Intentions intentions = Intentions();

  late final Heros heros;
  late final TextComponent _compteur;

  @override
  Color backgroundColor() => const Color(0xFF16162A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;

    // Le sol du donjon.
    await world.add(
      RectangleComponent(
        position: Vector2(40, 40),
        size: Vector2(560, 360),
        paint: Paint()..color = const Color(0xFF262640),
      ),
    );

    heros = Heros(intentions);
    await world.add(heros);

    // Source clavier : toujours branchée, elle ne coûte rien.
    await add(ControleClavier(intentions));

    // Sources tactiles : seulement sur mobile.
    if (estMobile) {
      final joystick = JoystickComponent(
        knob: CircleComponent(
          radius: 26,
          paint: Paint()..color = const Color(0xCCE8B04B),
        ),
        background: CircleComponent(
          radius: 64,
          paint: Paint()..color = const Color(0x44FFFFFF),
        ),
        margin: const EdgeInsets.only(left: 32, bottom: 32),
      );
      await camera.viewport.add(joystick);
      await add(ControleJoystick(intentions, joystick));
      await camera.viewport.add(BoutonAttaque(intentions));
    }

    _compteur = TextComponent(
      text: 'Coups : 0',
      position: Vector2(14, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDEE)),
      ),
    );
    await camera.viewport.add(_compteur);

    await camera.viewport.add(
      TextComponent(
        text: estMobile
            ? 'Joystick pour marcher, bouton rouge pour frapper'
            : 'ZQSD / WASD / flèches, Espace pour frapper',
        position: Vector2(14, 34),
        textRenderer: TextPaint(
          style: const TextStyle(fontSize: 13, color: Color(0xFF9999AA)),
        ),
      ),
    );
  }

  @override
  void update(double dt) {
    super.update(dt);
    _compteur.text = 'Coups : ${heros.coups}';
  }
}
```

**Résultat sur ordinateur :**

```text
  ┌──────────────────────────────────────────────┐
  │ Coups : 3                                    │
  │ ZQSD / WASD / flèches, Espace pour frapper   │
  │   ┌────────────────────────────────────────┐ │
  │   │                  ▓▓                    │ │
  │   └────────────────────────────────────────┘ │
  └──────────────────────────────────────────────┘
```

**Résultat sur mobile :** le même écran, avec le joystick en bas à gauche et le bouton rouge en bas à droite, tous deux fixes quel que soit le déplacement du héros.

### Les cinq décisions de conception à retenir

**Le héros ne connaît ni le clavier, ni le joystick.** Il lit `intentions`. Ajouter une manette au chapitre 42 consistera à écrire un `ControleManette` de vingt lignes.

**La normalisation est faite par la source, pas par le héros.** Le clavier normalise (direction binaire), le joystick ne normalise pas (direction analogique).

**`intentions.direction` est effacée à la fin de `update` du héros.** Chaque source la réécrit à la frame suivante ; sans cet effacement, le héros continuerait d'avancer après un relâchement du joystick.

**L'attaque est un drapeau consommé.** Le héros met `intentions.attaque = false` dès qu'il l'a traité, sinon il frapperait 60 fois par seconde.

**Le joystick et le bouton vont dans `camera.viewport`, les contrôleurs dans le jeu.** `ControleClavier` et `ControleJoystick` ne dessinent rien : ils sont ajoutés avec `add`, pas au monde, et reçoivent une `priority` négative pour être mis à jour **avant** le héros — sinon celui-ci lirait une intention déjà effacée.

### Pour aller plus loin

- Ajoutez un `RectangleHitbox` au héros et des gobelins : c'est le chapitre 32.
- Faites suivre le héros par la caméra avec `camera.follow(heros)` : c'est le chapitre 31.
- Ajoutez un effet visuel à l'attaque avec `ScaleEffect` : c'est le chapitre 33.

---

## 30.32 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le clavier ne fait rien, aucun message | Aucun mixin clavier sur le jeu | Ajouter `KeyboardEvents` **ou** `HasKeyboardHandlerComponents` sur la classe `FlameGame` |
| Assertion « Do not mix with both » au premier appui | `KeyboardEvents` **et** `HasKeyboardHandlerComponents` sur le même jeu | N'en garder qu'un seul ; `HasKeyboardHandlerComponents` rend `KeyboardEvents` inutile |
| `KeyboardHandler` sur un composant, mais rien ne se passe | Le jeu ne porte pas `HasKeyboardHandlerComponents` | L'ajouter sur la classe du jeu |
| `onTapDown` n'est jamais appelé | Le composant a une `size` de `Vector2.zero()` | Donner une `size` explicite : `super(size: Vector2(64, 48))` |
| La barre d'espace fait défiler la page Web en plus de faire sauter | `onKeyEvent` renvoie `KeyEventResult.ignored` | Renvoyer `KeyEventResult.handled` pour les touches du jeu |
| Le joystick disparaît dès que le héros avance | Joystick ajouté avec `world.add(joystick)` | `camera.viewport.add(joystick)` |
| Le héros va 41 % plus vite en diagonale | Direction non normalisée : `(1,1)` a une longueur de 1,414 | `direction.normalized() * vitesse * dt` |
| Le héros se déplace par à-coups avec un temps mort au départ | Le déplacement est écrit dans `onKeyEvent` et suit `KeyRepeatEvent` | Écrire l'état dans `onKeyEvent`, déplacer dans `update(dt)` |
| Le jeu est deux fois plus rapide sur un écran 120 Hz | Déplacement sans `dt` | Multiplier par `dt` : la vitesse est en pixels par seconde |
| Le héros dérive tout seul, joystick au repos | Aucune zone morte | Ignorer `relativeDelta` si `length < 0.15` |
| Le joystick ne pousse qu'à vitesse maximale | `relativeDelta` a été normalisé | Ne **jamais** normaliser une entrée analogique |
| Le bouton reste allumé après un glissement hors de sa zone | `onTapCancel` non traité | Surcharger `onTapCancel` et y remettre l'apparence de repos |
| Le bouton `HudButtonComponent` reste bloqué enfoncé | `super.onTapUp(event)` ou `super.onTapCancel(event)` oublié | Appeler `super` : ces méthodes sont `@mustCallSuper` |
| Le composant traîné saute sous le doigt | Utilisation d'une position absolue dans `onDragUpdate` | Utiliser `event.localDelta` et `position.add(...)` |
| `TapDetector` : « The name 'TapDetector' isn't defined » | Ce mixin n'existe plus en Flame 1.38.0 | `TapCallbacks` sur un composant (ou sur le jeu), ou `MultiTouchTapDetector` |
| `onKeyEvent(RawKeyEvent event, ...)` ne compile pas | Ancienne API Flutter | Signature actuelle : `onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed)` |
| La touche `W` ne fait rien sur clavier AZERTY | `LogicalKeyboardKey` dépend de la disposition : en AZERTY la touche du haut est `keyZ` | Accepter l'ensemble ZQSD **et** WASD |
| La pause clignote tant que `Échap` est enfoncée | La bascule est faite depuis `keysPressed` dans `update` | La faire sur `KeyDownEvent`, une seule fois par appui |
| `Platform.isAndroid` casse la compilation Web | `dart:io` n'existe pas sur le Web | Utiliser `defaultTargetPlatform` de `package:flutter/foundation.dart` |
| `HudMarginComponent` déclenche une assertion au montage | Le parent ne fournit pas de taille | L'ajouter à `camera.viewport`, pas à un `Component` nu |
| Le joystick refuse de se construire : assertion sur la position | Une `position` a été donnée au `knob` ou au `background` | Les laisser à zéro : le joystick les place lui-même |

---

## 30.33 — Résumé du chapitre

| Besoin | Mixin / composant | Méthode ou propriété clé |
| --- | --- | --- |
| Clavier au niveau du jeu | `KeyboardEvents` sur `FlameGame` | `KeyEventResult onKeyEvent(KeyEvent, Set<LogicalKeyboardKey>)` |
| Clavier au niveau du composant | `KeyboardHandler` sur `Component` | `bool onKeyEvent(KeyEvent, Set<LogicalKeyboardKey>)` |
| Distribuer le clavier dans l'arbre | `HasKeyboardHandlerComponents` sur `FlameGame` | remplace `KeyboardEvents` |
| Associer des touches à des fonctions | `KeyboardListenerComponent` | `keyDown: {...}`, `keyUp: {...}` |
| Savoir si une touche est maintenue | — | `keysPressed.contains(...)` |
| Réagir une seule fois à un appui | — | `event is KeyDownEvent` |
| Nommer une touche | `LogicalKeyboardKey` | `.keyZ`, `.space`, `.arrowUp`, `.escape` |
| Tap sur un composant | `TapCallbacks` | `onTapDown`, `onTapUp`, `onTapCancel` |
| Tap n'importe où | `TapCallbacks` sur `FlameGame` | `onTapDown` |
| Tap multipoint au niveau jeu | `MultiTouchTapDetector` | `onTapDown(int pointerId, TapDownInfo)` |
| Glisser un composant | `DragCallbacks` | `onDragUpdate` + `event.localDelta` |
| Survol de la souris | `HoverCallbacks` | `onHoverEnter`, `onHoverExit`, `isHovered` |
| Position du pointeur | `PointerMoveCallbacks` | `onPointerMove(PointerMoveEvent)` |
| Clic droit | `SecondaryTapCallbacks` | `onSecondaryTapDown` |
| Molette | `ScrollCallbacks` | `onScroll(ScrollEvent)` + `event.scrollDelta` |
| Joystick virtuel | `JoystickComponent` | `relativeDelta`, `intensity`, `direction` |
| Bouton d'action tactile | `HudButtonComponent` | `onPressed`, `onReleased`, `onCancelled` |
| Élément d'interface ancré à un bord | `HudMarginComponent` | `margin: EdgeInsets.only(...)` |
| Placer un contrôle à l'écran | — | `camera.viewport.add(...)` |
| Placer un élément dans le donjon | — | `world.add(...)` |
| Convertir un tap en coordonnées du monde | `CameraComponent` | `camera.globalToLocal(event.canvasPosition)` |
| Connaître la plateforme | `foundation.dart` | `defaultTargetPlatform`, `kIsWeb` |
| Éviter la diagonale trop rapide | `Vector2` | `direction.normalized()` |
| Éviter la dérive du joystick | — | zone morte : `relativeDelta.length < 0.15` |

---

## 30.34 — Exercices

### Exercice 1 — La sonde clavier (facile)
Écrivez un `FlameGame` avec `KeyboardEvents` qui affiche dans la console, à chaque événement, le type de l'événement (`APPUI`, `RÉPÉTITION`, `RELÂCHÉ`), le libellé de la touche, et le nombre de touches actuellement enfoncées.

### Exercice 2 — Le carré qui glisse (facile)
Un `RectangleComponent` se déplace avec les flèches, à 150 pixels par seconde, en utilisant `KeyboardHandler`. Le déplacement doit avoir lieu dans `update`, jamais dans `onKeyEvent`.

### Exercice 3 — Diagonale corrigée (facile)
Reprenez l'exercice 2, ajoutez ZQSD et WASD, et normalisez la direction. Affichez en haut de l'écran la vitesse réelle en pixels par seconde, pour prouver qu'elle est identique dans les huit directions.

### Exercice 4 — Le coffre au trésor (facile)
Trois coffres (`RectangleComponent` avec `TapCallbacks`) sont posés dans le donjon. Un tap ouvre un coffre : il change de couleur et un compteur de coffres ouverts s'incrémente dans le HUD. Un coffre déjà ouvert ne compte pas deux fois.

### Exercice 5 — Le bouton propre (intermédiaire)
Écrivez un bouton qui gère les trois états : repos, enfoncé, annulé. Il doit s'éclaircir à l'appui, revenir au repos si le doigt glisse dehors (`onTapCancel`) et n'exécuter son action qu'au relâchement **sur** le bouton.

### Exercice 6 — Le rangement de l'inventaire (intermédiaire)
Quatre `CircleComponent` avec `DragCallbacks` peuvent être traînés. Celui que l'on traîne passe au premier plan (`priority`) et grossit légèrement. Il retrouve sa taille au relâchement.

### Exercice 7 — Le déplacement au clic (intermédiaire)
Un tap n'importe où dans le monde fait avancer le héros vers le point cliqué, à vitesse constante, jusqu'à l'atteindre. Utilisez `camera.globalToLocal` pour convertir la position du tap.

### Exercice 8 — Joystick et jauge (intermédiaire)
Construisez un joystick sans image. Affichez en permanence dans le HUD sa direction (`JoystickDirection`) et l'intensité de poussée sous forme de barre horizontale dont la largeur vaut `relativeDelta.length × 200` pixels.

### Exercice 9 — Zone morte réglable (difficile)
Reprenez l'exercice 8. Ajoutez deux `HudButtonComponent` (« − » et « + ») qui font varier la zone morte de 0,00 à 0,50 par pas de 0,05. Affichez sa valeur, et faites changer la couleur du héros dès que le joystick franchit le seuil.

### Exercice 10 — Le double contrôle (difficile)
Écrivez un `Donjon de Dart` réduit où le héros se contrôle **simultanément** au clavier et au joystick, avec la classe `Intentions`. Le joystick doit toujours être visible, et un texte dans le HUD doit indiquer la source active : `clavier`, `joystick` ou `aucune`.

---

## 30.35 — Corrections des exercices

### Correction 1

```dart
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: SondeClavier()));

class SondeClavier extends FlameGame with KeyboardEvents {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    final String type = switch (event) {
      KeyDownEvent() => 'APPUI     ',
      KeyRepeatEvent() => 'RÉPÉTITION',
      KeyUpEvent() => 'RELÂCHÉ   ',
      _ => 'AUTRE     ',
    };
    debugPrint('$type ${event.logicalKey.keyLabel} '
        '| enfoncées : ${keysPressed.length}');
    return KeyEventResult.handled;
  }
}
```

**Résultat :**

```text
APPUI      Z | enfoncées : 1
RÉPÉTITION Z | enfoncées : 1
APPUI      D | enfoncées : 2
RELÂCHÉ    Z | enfoncées : 1
RELÂCHÉ    D | enfoncées : 0
```

**Explication :** `KeyEvent` est une classe abstraite dont l'objet réel est toujours l'une des trois sous-classes ; le `switch` sur motif de type, vu au chapitre 11, les distingue proprement. `keysPressed` est l'état complet du clavier, et non le contenu de l'événement : on le vérifie en voyant le compteur monter à 2 quand deux touches sont maintenues. Renvoyer `handled` évite que le navigateur ne réagisse aux flèches en plus du jeu.

---

### Correction 2

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuFleches()));

class JeuFleches extends FlameGame with HasKeyboardHandlerComponents {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Carre());
  }
}

class Carre extends RectangleComponent with KeyboardHandler {
  Carre()
      : super(
          position: Vector2(300, 200),
          size: Vector2.all(30),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  static const double vitesse = 150; // pixels par seconde
  final Vector2 _direction = Vector2.zero();

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    _direction
      ..x = (keysPressed.contains(LogicalKeyboardKey.arrowRight) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowLeft) ? 1 : 0)
      ..y = (keysPressed.contains(LogicalKeyboardKey.arrowDown) ? 1 : 0) -
          (keysPressed.contains(LogicalKeyboardKey.arrowUp) ? 1 : 0);
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (!_direction.isZero()) {
      position.add(_direction * vitesse * dt);
    }
  }
}
```

**Explication :** `onKeyEvent` ne fait qu'**écrire** le champ `_direction` ; tout le mouvement est dans `update`, multiplié par `dt`. Le jeu porte `HasKeyboardHandlerComponents`, sans quoi le `KeyboardHandler` du carré ne recevrait jamais rien. L'expression `(droite ? 1 : 0) - (gauche ? 1 : 0)` donne `0` quand les deux flèches opposées sont enfoncées, ce qui est le comportement attendu. Le carré s'arrête net au relâchement, car `keysPressed` ne contient alors plus rien.

---

### Correction 3

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: JeuDiagonale()));

const Set<LogicalKeyboardKey> kHaut = {
  LogicalKeyboardKey.keyZ, LogicalKeyboardKey.keyW, LogicalKeyboardKey.arrowUp,
};
const Set<LogicalKeyboardKey> kBas = {
  LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown,
};
const Set<LogicalKeyboardKey> kGauche = {
  LogicalKeyboardKey.keyQ, LogicalKeyboardKey.keyA, LogicalKeyboardKey.arrowLeft,
};
const Set<LogicalKeyboardKey> kDroite = {
  LogicalKeyboardKey.keyD, LogicalKeyboardKey.arrowRight,
};

class JeuDiagonale extends FlameGame with HasKeyboardHandlerComponents {
  final Carre carre = Carre();
  late final TextComponent sonde;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(carre);
    sonde = TextComponent(
      position: Vector2(12, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDEE)),
      ),
    );
    await camera.viewport.add(sonde);
  }

  @override
  void update(double dt) {
    super.update(dt);
    sonde.text =
        'vitesse réelle : ${carre.vitesseReelle.toStringAsFixed(1)} px/s';
  }
}

class Carre extends RectangleComponent with KeyboardHandler {
  Carre()
      : super(
          position: Vector2(300, 200),
          size: Vector2.all(30),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Vector2 _direction = Vector2.zero();
  double vitesseReelle = 0;

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    _direction
      ..x = (kDroite.any(keysPressed.contains) ? 1 : 0) -
          (kGauche.any(keysPressed.contains) ? 1 : 0)
      ..y = (kBas.any(keysPressed.contains) ? 1 : 0) -
          (kHaut.any(keysPressed.contains) ? 1 : 0);
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_direction.isZero()) {
      vitesseReelle = 0;
      return;
    }
    final Vector2 pas = _direction.normalized() * 150 * dt;
    position.add(pas);
    vitesseReelle = pas.length / dt; // reconstitution de la vitesse
  }
}
```

**Résultat** en maintenant `D` seule, puis `D` et `S` :

```text
vitesse réelle : 150.0 px/s
vitesse réelle : 150.0 px/s
```

**Explication :** `normalized()` renvoie une **copie** de longueur 1 : `_direction` reste intact et conserve ses valeurs `-1/0/1`. Sans normalisation, la seconde ligne afficherait 212,1 px/s, soit `150 × √2`. Les ensembles constants `kHaut`, `kGauche`… permettent de couvrir AZERTY, QWERTY et les flèches sans dupliquer les tests. Le test `_direction.isZero()` est indispensable : normaliser le vecteur nul produirait des `NaN`.

---

### Correction 4

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuCoffres()));

class JeuCoffres extends FlameGame {
  int ouverts = 0;

  final TextComponent compteur = TextComponent(
    text: 'Coffres ouverts : 0 / 3',
    position: Vector2(12, 12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 16, color: Color(0xFFDDDDEE)),
    ),
  );

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    for (final x in [140.0, 300.0, 460.0]) {
      await world.add(Coffre(position: Vector2(x, 220)));
    }
    await camera.viewport.add(compteur);
  }

  void signalerOuverture() {
    ouverts++;
    compteur.text = 'Coffres ouverts : $ouverts / 3';
  }
}

class Coffre extends RectangleComponent
    with TapCallbacks, HasGameReference<JeuCoffres> {
  Coffre({required Vector2 position})
      : super(
          position: position,
          size: Vector2(70, 52), // sans size, aucun tap ne serait reçu
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFF8A5A2B),
        );

  bool ouvert = false;

  @override
  void onTapDown(TapDownEvent event) {
    if (ouvert) return; // un coffre ne s'ouvre qu'une fois
    ouvert = true;
    paint.color = const Color(0xFFF0C36B);
    game.signalerOuverture();
  }
}
```

**Explication :** `TapCallbacks` se pose sur le composant et ne demande **aucun** mixin sur le jeu. La `size` explicite est vitale : `containsLocalPoint` s'appuie dessus, et un composant de taille nulle ne reçoit jamais de tap. `HasGameReference<JeuCoffres>` donne au coffre l'accès typé `game`, remplaçant le `HasGameRef` déprécié. Le drapeau `ouvert` garantit qu'un second tap ne compte pas.

---

### Correction 5

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuBouton()));

class JeuBouton extends FlameGame {
  final TextComponent journal = TextComponent(
    text: 'Posez le doigt, puis glissez dehors avant de relâcher.',
    position: Vector2(12, 12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 14, color: Color(0xFFDDDDEE)),
    ),
  );

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    await camera.viewport.addAll([journal, BoutonPropre(journal: journal)]);
  }
}

class BoutonPropre extends RectangleComponent with TapCallbacks {
  BoutonPropre({required this.journal})
      : super(
          position: Vector2(200, 180),
          size: Vector2(180, 70),
          anchor: Anchor.center,
          paint: Paint()..color = _couleurRepos,
        );

  final TextComponent journal;

  static const Color _couleurRepos = Color(0xFF3A6EA5);
  static const Color _couleurPressee = Color(0xFF7FB2E5);

  @override
  void onTapDown(TapDownEvent event) {
    paint.color = _couleurPressee;
    journal.text = 'état : ENFONCÉ';
  }

  @override
  void onTapUp(TapUpEvent event) {
    paint.color = _couleurRepos;
    journal.text = 'état : RELÂCHÉ — action exécutée';
  }

  @override
  void onTapCancel(TapCancelEvent event) {
    paint.color = _couleurRepos;
    journal.text = 'état : ANNULÉ — aucune action';
  }
}
```

**Résultat :**

```text
état : ENFONCÉ
état : ANNULÉ — aucune action
```

**Explication :** les trois méthodes forment un triangle complet. Sans `onTapCancel`, glisser le doigt hors du bouton le laisserait éclairé pour toujours, car `onTapUp` n'est **jamais** appelé après une annulation. Exécuter l'action dans `onTapUp` et non dans `onTapDown` laisse au joueur la possibilité de se raviser : c'est la convention des boutons d'interface. Le bouton est ajouté au `viewport` pour rester fixe à l'écran.

---

### Correction 6

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuInventaire()));

class JeuInventaire extends FlameGame {
  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    const couleurs = [0xFFCC4444, 0xFF44CC66, 0xFF4466CC, 0xFFCCAA33];
    for (var i = 0; i < 4; i++) {
      await world.add(Objet(
        position: Vector2(120.0 + i * 110, 200),
        couleur: Color(couleurs[i]),
      ));
    }
  }
}

class Objet extends CircleComponent with DragCallbacks {
  Objet({required Vector2 position, required Color couleur})
      : super(
          radius: 28,
          position: position,
          anchor: Anchor.center,
          paint: Paint()..color = couleur,
        );

  @override
  void onDragStart(DragStartEvent event) {
    super.onDragStart(event); // @mustCallSuper : met isDragged à true
    priority = 10;            // au premier plan
    scale = Vector2.all(1.2); // légèrement grossi
  }

  @override
  void onDragUpdate(DragUpdateEvent event) {
    position.add(event.localDelta);
  }

  @override
  void onDragEnd(DragEndEvent event) {
    super.onDragEnd(event);
    _reposer();
  }

  @override
  void onDragCancel(DragCancelEvent event) {
    super.onDragCancel(event);
    _reposer();
  }

  void _reposer() {
    priority = 0;
    scale = Vector2.all(1.0);
  }
}
```

**Explication :** `super` est obligatoire dans `onDragStart`, `onDragEnd` et `onDragCancel` : ces trois méthodes sont `@mustCallSuper` et tiennent à jour le getter `isDragged`. `event.localDelta` est le **déplacement** depuis la mise à jour précédente ; l'ajouter à `position` préserve l'écart initial entre le doigt et le centre, alors qu'affecter une position ferait sauter l'objet. `priority` est un z-index : une valeur plus élevée passe devant. Traiter `onDragCancel` comme `onDragEnd` évite qu'un objet reste grossi si le système interrompt le geste.

---

### Correction 7

```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuClicDeplacement()));

class JeuClicDeplacement extends FlameGame with TapCallbacks {
  final Heros heros = Heros();

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(heros);
  }

  @override
  void onTapDown(TapDownEvent event) {
    // canvasPosition est en pixels d'écran ; on la convertit en monde.
    heros.cible = camera.globalToLocal(event.canvasPosition);
  }
}

class Heros extends RectangleComponent {
  Heros()
      : super(
          position: Vector2(300, 220),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  static const double vitesse = 200;

  Vector2? cible;

  @override
  void update(double dt) {
    super.update(dt);
    final Vector2? but = cible;
    if (but == null) return;

    final Vector2 vers = but - position;
    final double pas = vitesse * dt;

    if (vers.length <= pas) {
      position.setFrom(but); // on est arrivé : pas de dépassement
      cible = null;
      return;
    }
    position.add(vers.normalized() * pas);
  }
}
```

**Explication :** `TapCallbacks` posé sur `FlameGame` fonctionne parce que `FlameGame` est un `Component` qui implémente `containsLocalPoint`. `camera.globalToLocal` convertit une position d'écran en position de monde : sans elle, le héros irait au mauvais endroit dès qu'un zoom ou un déplacement de caméra intervient. Le test `vers.length <= pas` évite le classique tremblement autour de la cible, où le héros dépasse puis revient à l'infini. `cible` est nullable : `null` signifie « aucune destination ».

---

### Correction 8

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuJoystick()));

class JeuJoystick extends FlameGame {
  final JoystickComponent joystick = JoystickComponent(
    knob: CircleComponent(
      radius: 26,
      paint: Paint()..color = const Color(0xCCE8B04B),
    ),
    background: CircleComponent(
      radius: 64,
      paint: Paint()..color = const Color(0x44FFFFFF),
    ),
    margin: const EdgeInsets.only(left: 32, bottom: 32),
  );

  final TextComponent info = TextComponent(
    position: Vector2(14, 12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDEE)),
    ),
  );

  final RectangleComponent jauge = RectangleComponent(
    position: Vector2(14, 40),
    size: Vector2(0, 14),
    paint: Paint()..color = const Color(0xFF44CC88),
  );

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await camera.viewport.addAll([joystick, info, jauge]);
  }

  @override
  void update(double dt) {
    super.update(dt);
    final double poussee = joystick.relativeDelta.length;
    info.text = 'direction : ${joystick.direction.name}   '
        'poussée : ${poussee.toStringAsFixed(2)}';
    jauge.size.x = poussee * 200;
  }
}
```

**Résultat** en tirant le joystick à fond vers la gauche :

```text
direction : left   poussée : 1.00
```

**Explication :** `knob` et `background` acceptent n'importe quel `PositionComponent` : deux `CircleComponent` suffisent, aucune image n'est nécessaire. Le joystick va dans `camera.viewport` pour rester fixe. `relativeDelta.length` vaut 0 au repos et 1 à la butée, y compris en diagonale, car `delta` est borné au cercle de rayon `knobRadius`. On préfère cette longueur à `intensity`, qui vaut le carré du rapport de poussée et progresse donc de façon non linéaire.

---

### Correction 9

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() => runApp(GameWidget(game: JeuZoneMorte()));

class JeuZoneMorte extends FlameGame {
  final RectangleComponent heros = RectangleComponent(
    position: Vector2(300, 200),
    size: Vector2.all(30),
    anchor: Anchor.center,
    paint: Paint()..color = const Color(0xFFE8B04B),
  );

  final JoystickComponent joystick = JoystickComponent(
    knob: CircleComponent(
      radius: 24,
      paint: Paint()..color = const Color(0xCCE8B04B),
    ),
    background: CircleComponent(
      radius: 60,
      paint: Paint()..color = const Color(0x44FFFFFF),
    ),
    margin: const EdgeInsets.only(left: 28, bottom: 28),
  );

  final TextComponent info = TextComponent(
    position: Vector2(14, 12),
    textRenderer: TextPaint(
      style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDEE)),
    ),
  );

  double zoneMorte = 0.15;

  @override
  Color backgroundColor() => const Color(0xFF1B1B2A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(heros);
    await camera.viewport.addAll([
      joystick,
      info,
      _bouton('-', 150, () => _regler(-0.05)),
      _bouton('+', 80, () => _regler(0.05)),
    ]);
  }

  HudButtonComponent _bouton(String texte, double marge, VoidCallback action) =>
      HudButtonComponent(
        button: RectangleComponent(
          size: Vector2.all(52),
          paint: Paint()..color = const Color(0xAA3A6EA5),
          children: [
            TextComponent(
              text: texte,
              position: Vector2.all(26),
              anchor: Anchor.center,
              textRenderer: TextPaint(
                style: const TextStyle(fontSize: 22, color: Color(0xFFFFFFFF)),
              ),
            ),
          ],
        ),
        buttonDown: RectangleComponent(
          size: Vector2.all(52),
          paint: Paint()..color = const Color(0xFF7FB2E5),
        ),
        margin: EdgeInsets.only(right: marge, bottom: 28),
        onPressed: action,
      );

  void _regler(double delta) =>
      zoneMorte = (zoneMorte + delta).clamp(0.0, 0.5);

  @override
  void update(double dt) {
    super.update(dt);

    final Vector2 poussee = joystick.relativeDelta;
    final bool actif = poussee.length >= zoneMorte;

    if (actif) {
      heros.position.add(poussee * 200 * dt);
    }
    heros.paint.color =
        actif ? const Color(0xFF66DD88) : const Color(0xFFE8B04B);

    info.text = 'zone morte : ${zoneMorte.toStringAsFixed(2)}   '
        'poussée : ${poussee.length.toStringAsFixed(2)}   '
        '${actif ? "ACTIF" : "repos"}';
  }
}
```

**Explication :** la zone morte n'existe pas dans `JoystickComponent` : elle se code toujours à la main, par comparaison de `relativeDelta.length` avec un seuil. `clamp(0.0, 0.5)` empêche des réglages absurdes. Les boutons sont placés par marges droite décroissantes (150 puis 80 px), donc côte à côte en bas à droite ; un `TextComponent` enfant du rectangle sert d'étiquette, sans police externe. Réglez la zone morte à 0,00 puis lâchez le joystick : vous verrez le héros dériver, ce qui démontre l'intérêt du seuil.

---

### Correction 10

```dart
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(GameWidget(game: DoubleControle()));

class Intentions {
  final Vector2 direction = Vector2.zero();
  String source = 'aucune';
}

class ControleClavier extends Component with KeyboardHandler {
  ControleClavier(this.intentions) : super(priority: -2);

  final Intentions intentions;
  final Vector2 lecture = Vector2.zero();

  static const _haut = {LogicalKeyboardKey.keyZ, LogicalKeyboardKey.keyW,
    LogicalKeyboardKey.arrowUp};
  static const _bas = {LogicalKeyboardKey.keyS, LogicalKeyboardKey.arrowDown};
  static const _gauche = {LogicalKeyboardKey.keyQ, LogicalKeyboardKey.keyA,
    LogicalKeyboardKey.arrowLeft};
  static const _droite = {LogicalKeyboardKey.keyD,
    LogicalKeyboardKey.arrowRight};

  @override
  bool onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keysPressed) {
    lecture
      ..x = (_droite.any(keysPressed.contains) ? 1 : 0) -
          (_gauche.any(keysPressed.contains) ? 1 : 0)
      ..y = (_bas.any(keysPressed.contains) ? 1 : 0) -
          (_haut.any(keysPressed.contains) ? 1 : 0);
    return true;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (lecture.isZero()) return;
    // Clavier : direction binaire -> normalisation.
    intentions.direction.setFrom(lecture.normalized());
    intentions.source = 'clavier';
  }
}

class ControleJoystick extends Component {
  ControleJoystick(this.intentions, this.joystick) : super(priority: -1);

  final Intentions intentions;
  final JoystickComponent joystick;

  @override
  void update(double dt) {
    super.update(dt);
    final Vector2 poussee = joystick.relativeDelta;
    if (poussee.length < 0.15) return; // zone morte
    // Joystick : direction analogique -> PAS de normalisation.
    intentions.direction.setFrom(poussee);
    intentions.source = 'joystick';
  }
}

class Heros extends RectangleComponent {
  Heros(this.intentions)
      : super(
          position: Vector2(300, 220),
          size: Vector2.all(28),
          anchor: Anchor.center,
          paint: Paint()..color = const Color(0xFFE8B04B),
        );

  final Intentions intentions;

  @override
  void update(double dt) {
    super.update(dt);
    if (!intentions.direction.isZero()) {
      position.add(intentions.direction * 190 * dt);
    }
    // On efface : chaque source réécrira à la frame suivante.
    intentions.direction.setZero();
    intentions.source = 'aucune';
  }
}

class DoubleControle extends FlameGame with HasKeyboardHandlerComponents {
  final Intentions intentions = Intentions();
  late final TextComponent info;
  String _derniere = 'aucune';

  @override
  Color backgroundColor() => const Color(0xFF16162A);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    await world.add(Heros(intentions));

    final joystick = JoystickComponent(
      knob: CircleComponent(
        radius: 26,
        paint: Paint()..color = const Color(0xCCE8B04B),
      ),
      background: CircleComponent(
        radius: 64,
        paint: Paint()..color = const Color(0x44FFFFFF),
      ),
      margin: const EdgeInsets.only(left: 32, bottom: 32),
    );
    await camera.viewport.add(joystick);

    // Les deux sources sont branchées EN MÊME TEMPS.
    await addAll([
      ControleClavier(intentions),
      ControleJoystick(intentions, joystick),
    ]);

    info = TextComponent(
      position: Vector2(14, 12),
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 15, color: Color(0xFFDDDDEE)),
      ),
    );
    await camera.viewport.add(info);
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (intentions.source != 'aucune') _derniere = intentions.source;
    info.text = 'source active : ${intentions.source}   '
        '(dernière : $_derniere)';
  }
}
```

**Résultat :**

```text
source active : clavier   (dernière : clavier)
source active : aucune    (dernière : clavier)
source active : joystick  (dernière : joystick)
```

**Explication :** les deux sources écrivent dans le même objet `Intentions` ; le héros lit puis **efface**, ce qui garantit qu'une source qui cesse d'émettre cesse aussi de faire avancer le héros. Chaque source applique la règle qui lui convient : normalisation pour le clavier, valeur brute pour le joystick. Les deux contrôleurs reçoivent une `priority` négative pour être mis à jour **avant** le héros, qui est dans le monde. Ajouter une manette au chapitre 42 reviendra à écrire une troisième classe de ce type, sans toucher ni au héros ni au jeu.

---

## Et maintenant ?

Votre héros obéit. Il se déplace au clavier, au doigt, au joystick, il frappe sur commande, et sa logique ne connaît aucun périphérique. Il reste cependant enfermé dans un écran fixe : dès que le donjon dépassera la taille de la fenêtre, vous ne verrez plus rien.

Le chapitre suivant règle ce problème. Vous y verrez comment `CameraComponent` observe un `World`, comment le `Viewport` définit la fenêtre de vue, comment faire suivre le héros avec `follow`, comment régler le `zoom` du `Viewfinder`, comment borner la caméra aux limites du niveau, et pourquoi le HUD que vous avez commencé à construire dans ce chapitre appartient précisément au viewport.

Rendez-vous au chapitre 31 : [31-PARTIE-2B—CAMÉRA-VIEWPORT-ET-MONDE.md](./31-PARTIE-2B—CAMÉRA-VIEWPORT-ET-MONDE.md)
