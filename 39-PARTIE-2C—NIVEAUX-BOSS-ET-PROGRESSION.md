# PARTIE 2C — LE JEU COMPLET « DONJON DE DART »
# CHAPITRE 39 — NIVEAUX, BOSS ET PROGRESSION

> **Niveau :** avancé
> **Durée estimée :** 12 h
> **Pré-requis :** chapitres 35 à 38 (architecture, joueur, ennemis, collectibles et HUD), chapitre 06 (`List`, `Map`, `Set`), chapitre 11 (enums, classes abstraites), chapitre 13 (exceptions), chapitre 17 (JSON et modélisation), chapitre 25 (cartes en `List<String>`, tilemap, caméra bornée), chapitre 26 (machines à états), chapitre 31 (`setBounds`, viewport), chapitre 33 (effets et fondus)
> **Version de Flame utilisée :** **1.38.0**
> **Ce que vous saurez faire à la fin :** décrire un niveau entier dans un fichier texte, le charger à la demande, enchaîner trois salles avec un fondu au noir, faire tomber un boss à trois phases et régler la difficulté — toujours sans un seul fichier image.

---

## 39.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- décrire l'état du projet à la fin du chapitre 38 et nommer précisément ce qui manque ;
- expliquer pourquoi une salle codée en dur dans `DonjonGame` est une impasse ;
- écrire un fichier `niveaux_data.dart` qui contient les cartes du jeu sous forme de `List<String>` ;
- définir une légende de caractères et la documenter dans un tableau ;
- dessiner trois niveaux en ASCII et vérifier leur validité avant de les jouer ;
- écrire une classe `Niveau` qui transforme une carte en composants Flame ;
- valider une carte et lever une `FormatException` explicite (rappel du chapitre 13) ;
- parcourir une carte caractère par caractère avec deux boucles imbriquées ;
- fusionner les tuiles voisines en une seule `Plateforme` et dire pourquoi c'est important ;
- distinguer une plateforme solide d'une plateforme traversable par le bas, et coder la différence ;
- instancier des ennemis, des collectibles et une porte à partir de la carte ;
- poser une entité « les pieds sur la tuile » quelle que soit sa taille ;
- écrire `chargerNiveau(int index)` sur `DonjonGame` ;
- vider le monde sans laisser de composant orphelin ni de caméra qui suit un fantôme ;
- expliquer pourquoi `await world.add(...)` ne se termine jamais quand le moteur est en pause ;
- borner la caméra sur les dimensions réelles du niveau chargé (rappel du chapitre 31) ;
- écrire un composant `Porte` qui consomme une clé et refuse poliment le passage ;
- écrire `terminerNiveau()` et enchaîner deux salles sans coupure visible ;
- construire un fondu au noir avec `OpacityEffect` et un `Completer` ;
- afficher un panneau de transition entre deux niveaux ;
- écrire un `Boss` qui hérite d'`Ennemi` et remplace son IA par une machine à phases ;
- coder une phase de charge, une phase de tir en éventail et une phase d'invocation ;
- calculer les directions d'un éventail de projectiles avec `atan2`, `cos` et `sin` ;
- borner le nombre de sbires vivants pour ne pas noyer le joueur ;
- afficher une barre de vie de boss dans le HUD et la faire apparaître au bon moment ;
- concevoir une fenêtre de vulnérabilité et expliquer pourquoi elle vaut mieux qu'un point faible géométrique ;
- déclencher la victoire à la mort du boss ;
- lire et écrire une courbe de difficulté sous forme de tableau ;
- implémenter trois modes de difficulté qui modifient réellement le jeu ;
- préparer la sauvegarde de la progression pour le chapitre 40 ;
- livrer un jeu jouable **de bout en bout**, du menu à l'écran de victoire.

---

## 39.1 — Où on en est et ce qu'on ajoute

### L'état du projet à la fin du chapitre 38

À la fin du chapitre 38, le « Donjon de Dart » est un vrai petit jeu. Il manque simplement… un jeu autour.

```text
  CE QUI FONCTIONNE À LA FIN DU CHAPITRE 38

  ┌─ MENU ────────────────────────────────────────────────────────┐
  │  « DONJON DE DART » + bouton Jouer          (ch. 35)          │
  └───────────────────────────────────────────────────────────────┘
                              │ demarrerPartie()
                              v
  ┌─ UNE SALLE UNIQUE ────────────────────────────────────────────┐
  │  Décor    : 6 Plateforme posées à la main dans le code        │
  │  Héros    : Joueur — marche, saut, attaque, PV, mort  (ch. 36)│
  │  Ennemis  : 2 Gobelin + 2 Chauvesouris, IA à 5 états  (ch. 37)│
  │  Objets   : 9 Piece, 2 Potion, 1 Cle                  (ch. 38)│
  │  Score    : points, combo, multiplicateur             (ch. 38)│
  │  HUD      : vie, énergie, cœurs, score, clés, objectif(ch. 38)│
  └───────────────────────────────────────────────────────────────┘
                              │
                              v
                    … et c'est tout. On tourne en rond.
```

Le joueur ramasse la clé. Le HUD lui annonce fièrement « Objectif : atteindre la porte ». Il n'y a pas de porte. Il n'y a rien après. Le chapitre 38 se terminait d'ailleurs sur cet aveu explicite.

### Ce que le chapitre 39 ajoute

Ce chapitre transforme une salle en **jeu** : trois niveaux, une porte, une transition, un boss, une victoire.

| Fichier | Statut | Contenu |
| --- | --- | --- |
| `lib/niveaux/niveaux_data.dart` | **créé** | les trois cartes du jeu en `List<String>`, la légende, les métadonnées |
| `lib/niveaux/niveau.dart` | **créé** | la classe `Niveau` : validation, analyse, fabrication des composants |
| `lib/composants/porte.dart` | **créé** | `Porte` : verrouillée, ouverte à la clé, franchie une seule fois |
| `lib/composants/boss.dart` | **créé** | `Boss` et `PhaseBoss` : la machine à phases du gardien |
| `lib/donjon_game.dart` | **modifié** | `chargerNiveau`, `viderLeMonde`, `terminerNiveau`, bornes de caméra, difficulté |
| `lib/composants/plateforme.dart` | **modifié** | le drapeau `traversable` |
| `lib/composants/joueur.dart` | **modifié** | résolution de collision sur une plateforme traversable |
| `lib/composants/ennemi.dart` | **modifié** | cible nulle tolérée, plateformes traversables, réglages de difficulté |
| `lib/composants/gobelin.dart` | **modifié** | garde sur la cible nulle |
| `lib/composants/chauvesouris.dart` | **modifié** | garde sur la cible nulle |
| `lib/hud/hud.dart` | **modifié** | la barre du boss, le voile de transition |
| `lib/hud/barre_de_vie.dart` | **modifié** | `BarreBoss` |
| `lib/hud/compteur_score.dart` | **modifié** | `CompteurVies` lit le vrai nombre de vies |
| `lib/config/constantes.dart` | **modifié** | constantes de porte, de boss, de transition, modes de difficulté |
| `lib/config/palette.dart` | **modifié** | couleurs de la porte, du boss, du voile |
| `lib/ecrans/menu_principal.dart` | **modifié** | le choix de la difficulté |
| `lib/main.dart` | **modifié** | l'overlay de victoire (provisoire, finalisé au chapitre 40) |

### La règle qui ne change pas

> **Aucun fichier image, aucun fichier son.**
>
> Les niveaux sont du texte. Les plateformes sont des rectangles. Le boss est un gros rectangle qui change de couleur. Le fondu au noir est un rectangle noir dont on anime l'opacité. Le jour où vous aurez des assets, vous remplacerez le contenu de `render()` et **rien d'autre**.

---

## 39.2 — Le problème : la salle de test est codée en dur

Voici, tel quel, le décor du chapitre 37 :

```dart
Future<void> _construireSalle() async {
  const t = Constantes.tailleTuile;

  await monde.addAll([
    Plateforme(position: Vector2(0, 6 * t), size: Vector2(16 * t, t)),
    Plateforme(position: Vector2(0, 0), size: Vector2(t, 6 * t)),
    Plateforme(position: Vector2(15 * t, 0), size: Vector2(t, 6 * t)),
    Plateforme(position: Vector2(3 * t, 4 * t), size: Vector2(3 * t, 12)),
    Plateforme(position: Vector2(9 * t, 3 * t), size: Vector2(4 * t, 12)),
    Plateforme(position: Vector2(7 * t, 5 * t), size: Vector2(t, t)),
  ]);
  // ... puis le joueur, puis les ennemis, puis (chapitre 38) les objets
}
```

Ce code marche. Il n'est pas *faux*. Il est simplement **impossible à faire grandir**. Regardons pourquoi, précisément.

### Six défauts, un par ligne de raisonnement

| Défaut | Conséquence concrète |
| --- | --- |
| On ne **voit** pas le niveau | Personne ne peut dire, en lisant ces six lignes, à quoi ressemble la salle. Il faut lancer le jeu. |
| Modifier le décor = modifier le **code** | Chaque déplacement de plateforme est un `flutter run` complet. |
| Le décor est mêlé à la **logique** | `DonjonGame` fait à la fois moteur, arbitre et décorateur. |
| Un seul niveau possible | Pour en avoir trois, il faudrait `_construireSalle1()`, `_construireSalle2()`, `_construireSalle3()` : trois fois le même code. |
| Aucune **validation** | Une plateforme oubliée fait tomber le joueur dans le vide sans le moindre message. |
| Non modifiable par un **non-programmeur** | Un game designer ne peut pas toucher au jeu. |

### La solution : les données à part du code

C'est exactement la leçon du chapitre 25. Un niveau n'est pas du code : c'est une **donnée**. Et la donnée la plus lisible pour une grille 2D, c'est un dessin.

```text
  AVANT (chapitre 37)                 APRÈS (chapitre 39)

  Plateforme(                         '################'
    position: Vector2(0, 6*t),        '#..............#'
    size: Vector2(16*t, t),           '#..o.......o...#'
  ),                                  '#....====......#'
  Plateforme(                         '#..J.......g..D#'
    position: Vector2(0, 0),          '################'
    size: Vector2(t, 6*t),
  ),                                        On VOIT la salle.
  ... 4 lignes de plus                      On la modifie au clavier.
                                            On la relit dans six mois.
  On ne voit rien.
```

Le contrat de ce chapitre tient en une phrase :

> **`DonjonGame` ne saura plus jamais où se trouve une plateforme.** Il demandera un niveau à `Niveau`, et `Niveau` lira une carte.

### L'architecture visée

```text
  niveaux_data.dart              niveau.dart                donjon_game.dart
  ─────────────────              ───────────                ────────────────
  LES DONNÉES                    LE TRADUCTEUR              LE CHEF D'ORCHESTRE

  carteNiveau1 : List<String>    Niveau(definition)         chargerNiveau(0)
  carteNiveau2 : List<String>      .valider()                 -> viderLeMonde()
  carteNiveau3 : List<String>      .analyser()                -> Niveau.numero(0)
  DefinitionNiveau x 3             .construireDans(monde)     -> construireDans()
                                                              -> camera.setBounds()
  aucun import Flame             connaît Flame              connaît tout
  aucune logique                 aucune décision de jeu     décide
```

Trois fichiers, trois responsabilités, aucun recouvrement. C'est la séparation que le chapitre 26 appelait « données / traduction / orchestration ».

---

## 39.3 — `lib/niveaux/niveaux_data.dart` : les cartes en `List<String>` (rappel chapitre 25)

### Pourquoi `List<String>` et pas autre chose

Au chapitre 25, section 25.25, vous avez comparé deux écritures d'une même salle :

```dart
// Illisible.
<int>[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
<int>[1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
```

```dart
// Lisible d'un coup d'œil.
'############',
'#..........#',
```

La conclusion n'a pas changé : **une carte est un dessin**. Une `List<String>` est le seul format qui garde le dessin visible dans le code source.

Récapitulons les autres candidats et pourquoi ils sont écartés **pour ce chapitre** :

| Format | Avantage | Pourquoi pas ici |
| --- | --- | --- |
| `List<List<int>>` | direct à indexer | illisible : on ne voit plus la salle |
| Fichier JSON externe | éditable sans recompiler | chargement asynchrone, gestion d'erreurs, `rootBundle` : hors sujet ici (chapitre 17 pour le JSON, chapitre 40 pour les fichiers) |
| Tiled (`.tmx`) | éditeur graphique complet | demande `flame_tiled`, un tileset image, et donc des assets — interdits ici (chapitre 34 pour la découverte) |
| `List<String>` | **visible, éditable, diffable, testable** | aucune limite avant ~80 colonnes de large |

### Une constante de premier niveau

Une carte ne change jamais pendant la partie. Elle est donc `const`, ce qui la place en mémoire une seule fois, au démarrage du programme.

```dart
// lib/niveaux/niveaux_data.dart
class NiveauxData {
  /// Constructeur privé : cette classe n'est qu'un porte-données.
  /// On ne l'instancie jamais. (Rappel : chapitre 09.)
  NiveauxData._();

  static const List<String> carteNiveau1 = <String>[
    '########################################',
    '#......................................#',
    // ... la suite en 39.5
  ];
}
```

Trois détails qui comptent :

1. `NiveauxData._()` — un constructeur privé nommé empêche `NiveauxData()`. La classe est un simple espace de noms, comme `Constantes` et `Palette` (chapitre 35).
2. `static const` — la carte est partagée, immuable, et ne coûte rien à chaque niveau chargé.
3. `<String>[...]` — le type explicite dans le littéral. Ce n'est pas obligatoire, mais si vous tapez un jour `0` au lieu de `'0'`, l'erreur est immédiate et lisible.

### La métadonnée : `DefinitionNiveau`

Une carte, ce n'est pas tout à fait un niveau. Un niveau a aussi un **nom**, un **sous-titre** affiché pendant la transition, et une règle : sa porte est-elle verrouillée ?

```dart
class DefinitionNiveau {
  const DefinitionNiveau({
    required this.nom,
    required this.soustitre,
    required this.carte,
    this.porteVerrouillee = true,
  });

  /// Nom affiché sur le panneau de transition.
  final String nom;

  /// Une ligne d'ambiance ou de consigne.
  final String soustitre;

  /// La carte, un caractère par tuile.
  final List<String> carte;

  /// Faux pour la salle du boss : sa porte n'attend pas une clé,
  /// elle attend un cadavre.
  final bool porteVerrouillee;

  int get colonnes => carte.isEmpty ? 0 : carte.first.length;
  int get lignes => carte.length;
}
```

C'est un objet de données immuable, exactement au sens du chapitre 09 : tous les champs sont `final`, le constructeur est `const`, il n'y a aucune méthode qui modifie quoi que ce soit.

**Résultat :**

```text
DefinitionNiveau(nom: 'Les caves', carte: 40 colonnes x 14 lignes)
```

---

## 39.4 — La légende des caractères (tableau)

C'est le tableau le plus important du chapitre. Il est fixé par le cahier des charges du jeu ; il ne change plus jusqu'à la fin de la formation.

| Caractère | Signification | Composant instancié | Bloquant ? |
| --- | --- | --- | --- |
| `#` | mur ou sol de pierre | `Plateforme(traversable: false)` | oui, dans les quatre directions |
| `.` | vide, air | *aucun* | non |
| `=` | plateforme traversable par le bas | `Plateforme(traversable: true)` | oui, **uniquement par le dessus** |
| `J` | position de départ du joueur | `Joueur` | — |
| `g` | gobelin | `Gobelin` | non |
| `c` | chauve-souris | `Chauvesouris` | non |
| `o` | pièce | `Piece` | non (capteur) |
| `p` | potion | `Potion` | non (capteur) |
| `k` | clé | `Cle` | non (capteur) |
| `D` | porte de sortie | `Porte` | non (capteur) |
| `B` | boss | `Boss` | non |

### Les trois règles de la légende

**Règle 1 — un caractère inconnu est une erreur, pas un vide.**

C'est le choix du chapitre 25 et c'est le bon. Si vous tapez `O` (la lettre) au lieu de `o` (la pièce), vous voulez un message d'erreur, pas une salle silencieusement amputée d'une pièce.

**Règle 2 — la casse compte.**

`g` est un gobelin, `B` est un boss, `D` est une porte, `J` est le joueur. La convention retenue : **minuscule = petit**, **majuscule = important**. Elle n'est pas obligatoire, elle est simplement mnémotechnique.

**Règle 3 — un caractère occupe exactement une tuile de 32 px.**

Un boss fait 48 px de large : il déborde donc de sa tuile. Ce n'est pas un problème, à condition de le **poser** correctement, ce que fait la section 39.9.

### Les constantes de légende

Écrire `'#'` en dur trente fois dans le parseur est une invitation à la faute de frappe. On nomme les caractères.

```dart
class NiveauxData {
  NiveauxData._();

  static const String mur = '#';
  static const String vide = '.';
  static const String plateforme = '=';
  static const String joueur = 'J';
  static const String gobelin = 'g';
  static const String chauvesouris = 'c';
  static const String piece = 'o';
  static const String potion = 'p';
  static const String cle = 'k';
  static const String porte = 'D';
  static const String boss = 'B';

  /// Tout caractère absent de cet ensemble fera lever une FormatException.
  static const Set<String> caracteresConnus = <String>{
    mur, vide, plateforme, joueur, gobelin,
    chauvesouris, piece, potion, cle, porte, boss,
  };
}
```

> **Pourquoi un `Set` et pas une `List` ?** Parce que la seule opération dont nous avons besoin est `contains`, appelée une fois **par tuile**. Sur une carte de 48 × 18, cela fait 864 appels par chargement. `Set.contains` est en temps constant, `List.contains` en temps linéaire. C'est le raisonnement du chapitre 06.

---

## 39.5 — Les trois niveaux du jeu, dessinés en ASCII

Voici les trois cartes. Elles sont validées : chaque ligne d'une carte a exactement la même longueur, chaque caractère appartient à la légende, chaque carte contient exactement un `J` et un `D`.

### Niveau 1 — « Les caves » (40 × 14)

Le niveau d'apprentissage. Sol continu, pas de trou, deux gobelins bien espacés, deux chauves-souris, un pilier à franchir, la clé au bout du chemin.

```text
   0         1         2         3
   0123456789012345678901234567890123456789
 0 ########################################
 1 #......................................#
 2 #.........o.o.o........................#
 3 #.......========.......................#
 4 #..................o.o.o...............#
 5 #................========..............#
 6 #......c...........................c...#
 7 #............................o.o.o.....#
 8 #..........................========....#
 9 #...........o..........................#
10 #.................p....................#
11 #.....g....###.............g...........#
12 #.J........###.................k.....D.#
13 ########################################
```

Lecture : le héros démarre en (2, 12), tout à gauche. Il croise un gobelin, saute par-dessus un pilier de deux tuiles, ramasse une potion et neuf pièces, évite deux chauves-souris, prend la clé en (31, 12) et ressort par la porte en (37, 12).

### Niveau 2 — « Les oubliettes » (48 × 18)

Le niveau de plateforme. Le sol est **une seule bande tout en bas** : tout le reste est fait de plateformes traversables. Le héros descend en sautant de perchoir en perchoir, ou tombe et remonte.

```text
   0         1         2         3         4
   012345678901234567890123456789012345678901234567
 0 ################################################
 1 #.J...........................o.o.o............#
 2 #=====.........................................#
 3 #..........o.o.o...............................#
 4 #.....========....................c............#
 5 #..............................................#
 6 #.......p..................o.o.o...............#
 7 #..========...............=========............#
 8 #..............................................#
 9 #.................o.o.o...............o.o.o....#
10 #...............========...........=========...#
11 #..............................................#
12 #....o.o.o..........g..............p......k....#
13 #...========.......................========....#
14 #..............................................#
15 #..g...................o.o.o.......c...........#
16 #......o.o......c.....========....g........D...#
17 ################################################
```

Lecture : le héros démarre en (2, 1), posé sur la plateforme `=====` de la ligne 2. Vingt-trois pièces, deux potions, trois gobelins, trois chauves-souris. La clé est en (42, 12), sur la plateforme de la ligne 13 : il faut grimper pour l'atteindre. La porte est en bas à droite, en (43, 16).

### Niveau 3 — « La salle du gardien » (40 × 14)

L'arène. Un sol plat d'un mur à l'autre pour que le boss puisse charger, quatre plateformes hautes pour esquiver, aucun ennemi ordinaire au départ — le boss en invoquera lui-même.

```text
   0         1         2         3
   0123456789012345678901234567890123456789
 0 ########################################
 1 #......................................#
 2 #....o.o..........................o.o..#
 3 #...======....................======...#
 4 #......................................#
 5 #..........p................p..........#
 6 #......................................#
 7 #.......========......========.........#
 8 #......................................#
 9 #......................................#
10 #....o....................o............#
11 #......................................#
12 #..J..............B....................#
13 ########################################
```

Lecture : le héros démarre en (3, 12), le boss l'attend en (18, 12), la porte est en (36, 12). Cette porte n'est **pas** verrouillée par une clé : elle attend que le gardien tombe.

### Le tableau de bord des trois cartes

| Niveau | Nom | Dimensions | Pièces | Gobelins | Chauves-souris | Boss | Clé |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Les caves | 40 × 14 | 10 | 2 | 2 | non | oui |
| 2 | Les oubliettes | 48 × 18 | 23 | 3 | 3 | non | oui |
| 3 | La salle du gardien | 40 × 14 | 6 | 0 (invoqués) | 0 | **oui** | non |

En pixels, avec `tailleTuile = 32` : 1280 × 448, 1536 × 576 et 1280 × 448. Avec un zoom de caméra de 2, l'écran montre à peu près 20 tuiles de large : aucun niveau ne tient à l'écran, la caméra devra donc défiler et être bornée (section 39.14).

### Comment dessiner vos propres cartes

Trois conseils qui vous éviteront des heures perdues :

1. **Commencez par le cadre.** Une ligne de `#` en haut, une en bas, un `#` au début et à la fin de chaque ligne. Vous ne tomberez jamais hors du monde.
2. **Numérotez les colonnes en commentaire** au-dessus de la carte, comme ci-dessus. Sans cela, vous ne saurez jamais si votre porte est en colonne 36 ou 37.
3. **Vérifiez les longueurs avant de lancer le jeu.** La section 39.7 écrit le validateur qui le fait pour vous, avec un message qui donne le numéro de ligne.

---

## 39.6 — `lib/niveaux/niveau.dart` : la classe `Niveau`

### Ce que `Niveau` est, et ce qu'il n'est pas

`Niveau` **n'est pas** un composant Flame. Ce n'est pas un `Component`, il ne s'ajoute à rien, il ne se dessine pas, il n'a ni `update` ni `render`.

`Niveau` est un **traducteur** : il prend une `DefinitionNiveau` (du texte) et produit une liste de composants Flame prêts à être ajoutés au monde.

```text
   DefinitionNiveau                 Niveau                    World
   ┌──────────────┐            ┌─────────────┐          ┌──────────────┐
   │ '#####'      │            │  valider()  │          │ Plateforme   │
   │ '#..o#'      │  ───────>  │  analyser() │ ───────> │ Piece        │
   │ '#J..#'      │            │ construire  │          │ Joueur       │
   │ '#####'      │            │   Dans()    │          │ ...          │
   └──────────────┘            └─────────────┘          └──────────────┘
        texte                    traduction               composants
```

Cette séparation a un bénéfice immédiat : **`Niveau` est testable sans lancer le jeu**. Vous pouvez écrire un test qui charge les trois cartes, appelle `valider()`, et vérifie qu'aucune exception n'est levée. Le chapitre 42 en fera un test unitaire.

### Le squelette

```dart
// lib/niveaux/niveau.dart
import 'package:flame/components.dart';

import '../composants/boss.dart';
import '../composants/chauvesouris.dart';
import '../composants/cle.dart';
import '../composants/gobelin.dart';
import '../composants/joueur.dart';
import '../composants/piece.dart';
import '../composants/plateforme.dart';
import '../composants/porte.dart';
import '../composants/potion.dart';
import '../config/constantes.dart';
import 'niveaux_data.dart';

/// Traduit une carte texte en composants Flame.
///
/// Un `Niveau` est à USAGE UNIQUE : il fabrique ses composants une fois,
/// les livre au monde, et n'est plus jamais réutilisé. Recharger un niveau
/// veut dire construire un NOUVEAU `Niveau`.
class Niveau {
  Niveau(this.definition);

  /// Fabrique de confort : `Niveau.numero(0)` charge le premier niveau.
  factory Niveau.numero(int index) => Niveau(NiveauxData.definition(index));

  final DefinitionNiveau definition;

  /// Raccourci de lecture.
  List<String> get carte => definition.carte;

  static const double t = Constantes.tailleTuile;

  /// Épaisseur visuelle d'une plateforme traversable, en pixels.
  static const double epaisseurPlateforme = 10.0;

  // ---- Dimensions -----------------------------------------------------

  int get colonnes => definition.colonnes;
  int get lignes => definition.lignes;
  double get largeurPixels => colonnes * t;
  double get hauteurPixels => lignes * t;

  // ---- Produits de l'analyse ------------------------------------------

  final List<Plateforme> plateformes = <Plateforme>[];
  final List<PositionComponent> ennemis = <PositionComponent>[];
  final List<PositionComponent> collectibles = <PositionComponent>[];

  Vector2 apparitionJoueur = Vector2.zero();
  Joueur? joueur;
  Porte? porte;
  Boss? boss;
  int nombreDePieces = 0;

  bool _analyse = false;

  bool get contientBoss => boss != null;
}
```

### Pourquoi trois listes plutôt qu'une

On pourrait tout mettre dans une seule `List<Component>`. Trois listes valent mieux, pour une raison qui n'a rien d'esthétique : **l'ordre d'ajout au monde compte**.

| Groupe | Doit être ajouté | Pourquoi |
| --- | --- | --- |
| `plateformes` | en premier | pour que les rayons de `solDevant()` (chapitre 37) trouvent quelque chose dès la première frame |
| `joueur` | ensuite | parce que chaque `Ennemi` lit `game.joueur` dès son montage |
| `collectibles` et `ennemis` | en dernier | ils dépendent des deux précédents, personne ne dépend d'eux |

L'ordre est une **contrainte du domaine**, pas un détail. Trois listes séparées le rendent visible dans le code.

### Le drapeau `_analyse`

`analyser()` fabrique des composants. L'appeler deux fois fabriquerait deux jeux de composants et le second effacerait les références du premier. Le drapeau garantit qu'on ne le fait qu'une fois :

```dart
  /// Valide la carte puis fabrique tous les composants.
  /// Idempotente : le deuxième appel ne fait rien.
  void analyser() {
    if (_analyse) {
      return;
    }
    valider();
    _construireDecor();
    _construireEntites();
    _analyse = true;
  }
```

C'est le patron « initialisation paresseuse » du chapitre 12 : on ne fait le travail qu'au premier besoin, et jamais deux fois.

---

## 39.7 — Parser la carte caractère par caractère

### D'abord valider, ensuite analyser

La règle du chapitre 13 s'applique intégralement : **échouer tôt, échouer clairement**. Un niveau mal formé doit exploser au chargement avec un message qui donne la ligne et la colonne, pas produire une salle bancale que vous déboguerez pendant une heure.

Le validateur vérifie quatre choses, dans cet ordre. Son code intégral figure en section 39.34 ; voici sa charpente et ses messages.

```dart
  /// Vérifie la carte. Lève une [FormatException] au premier défaut.
  void valider() {
    // 1. La carte n'est pas vide.
    //    -> 'la carte ne contient aucune ligne.'

    final int largeur = carte.first.length;

    for (int l = 0; l < carte.length; l++) {
      final String ligne = carte[l];

      // 2. Toutes les lignes ont la même longueur.
      if (ligne.length != largeur) {
        throw FormatException(
          'Niveau "${definition.nom}" : ligne $l de longueur '
          '${ligne.length}, attendu $largeur.',
        );
      }

      for (int c = 0; c < ligne.length; c++) {
        // 3. Chaque caractère appartient à la légende.
        if (!NiveauxData.caracteresConnus.contains(ligne[c])) {
          throw FormatException(
            'Niveau "${definition.nom}" : caractère "${ligne[c]}" inconnu '
            'en ligne $l, colonne $c.',
          );
        }
        // ... et l'on compte au passage les 'J' et les 'D'
      }
    }

    // 4. Exactement un 'J' et exactement un 'D'.
    //    -> 'il faut exactement un "J", trouvé 2.'
  }
```

### Les quatre erreurs que ce validateur attrape

| Faute | Message obtenu |
| --- | --- |
| Un espace en trop en fin de ligne | `ligne 7 de longueur 41, attendu 40.` |
| Un `O` majuscule au lieu d'un `o` | `caractère "O" inconnu en ligne 4, colonne 12.` |
| Deux `J` (copier-coller d'une ligne) | `il faut exactement un "J", trouvé 2.` |
| Porte oubliée | `il faut exactement une porte "D", trouvé 0.` |

Sans ce validateur, ces quatre fautes donnent respectivement : un niveau décalé d'une colonne à partir de la ligne 7, une pièce manquante, deux héros qui se marchent dessus, et un niveau dont on ne sort jamais. Aucune des quatre ne produit d'erreur à l'exécution. Ce sont exactement les bugs les plus longs à trouver.

### Le parcours : deux boucles imbriquées

Le parcours d'une grille 2D est toujours le même depuis le chapitre 05 :

```text
  pour chaque LIGNE l de 0 à lignes-1
      pour chaque COLONNE c de 0 à colonnes-1
          traiter carte[l][c]
```

Attention au piège de l'ordre : **la ligne d'abord, la colonne ensuite**. `carte[l]` est une `String` — la ligne. `carte[l][c]` est un caractère de cette ligne — la colonne. Écrire `carte[c][l]` compile parfaitement et produit un niveau transposé, ou une `RangeError` si la carte n'est pas carrée.

```text
  carte[l][c]
        │  └── c : COLONNE, l'axe X, de 0 à colonnes-1
        └───── l : LIGNE,   l'axe Y, de 0 à lignes-1

  En pixels :  x = c * 32      y = l * 32
```

### De la tuile aux pixels

Deux fonctions d'aide résolvent une fois pour toutes la conversion. Elles sont privées, courtes, et utilisées partout ensuite.

```dart
  /// Coin haut-gauche de la tuile (colonne, ligne), en pixels du monde.
  static Vector2 coinDe(int colonne, int ligne) =>
      Vector2(colonne * t, ligne * t);

  /// Centre de la tuile (colonne, ligne), en pixels du monde.
  static Vector2 centreDe(int colonne, int ligne) =>
      Vector2(colonne * t + t / 2, ligne * t + t / 2);

  /// Position `topLeft` d'une entité de [taille] posée AU SOL de la tuile,
  /// centrée horizontalement dessus.
  ///
  /// Fonctionne quelle que soit la taille : un gobelin de 24x30 et un boss
  /// de 48x56 ont tous les deux les pieds exactement sur la tuile.
  static Vector2 poserSurTuile(int colonne, int ligne, Vector2 taille) =>
      Vector2(
        colonne * t + (t - taille.x) / 2,
        (ligne + 1) * t - taille.y,
      );
```

**Résultat**, avec `t = 32` :

```text
coinDe(3, 12)                          -> (96, 384)
centreDe(3, 12)                        -> (112, 400)
poserSurTuile(3, 12, Vector2(24, 30))  -> (100, 386)   gobelin : pieds en y=416
poserSurTuile(3, 12, Vector2(48, 56))  -> (88, 360)    boss    : pieds en y=416
```

Les deux entités ont bien les pieds sur la même ligne, `y = 416`, c'est-à-dire le haut de la tuile 13. C'est tout l'intérêt de `poserSurTuile` : la formule `(ligne + 1) * t - taille.y` aligne les **pieds**, pas les têtes.

---

## 39.8 — Instancier les plateformes

### Le piège du composant par tuile

La méthode naïve crée une `Plateforme` par caractère `#`. Comptons ce que cela donne sur nos trois cartes :

| Niveau | Caractères `#` | Composants créés |
| --- | --- | --- |
| 1 | 110 | 110 |
| 2 | 128 | 128 |
| 3 | 104 | 104 |

Chacun porte un `RectangleComponent` et une `RectangleHitbox`. Cela fait donc **342 composants** et **342 hitbox** pour trois salles vides. Or, la détection de collision de Flame teste chaque hitbox active contre les hitbox voisines : plus il y en a, plus la broadphase travaille.

Pire : les jointures. Deux plateformes collées côte à côte créent une arête interne. Le code de résolution du chapitre 37 (« je repousse selon le plus petit chevauchement ») peut, sur cette arête, repousser le joueur vers le haut alors qu'il courait à droite. C'est le fameux bug du « héros qui accroche entre deux dalles ».

### La fusion des tuiles voisines

La solution est un classique du jeu 2D : **fusionner les suites horizontales de tuiles identiques en une seule plateforme large**.

```text
  AVANT — 8 plateformes de 32 px           APRÈS — 1 plateforme de 256 px

  ┌──┬──┬──┬──┬──┬──┬──┬──┐                ┌───────────────────────┐
  │##│##│##│##│##│##│##│##│                │#######################│
  └──┴──┴──┴──┴──┴──┴──┴──┘                └───────────────────────┘
   7 arêtes internes                        0 arête interne
   8 hitbox                                 1 hitbox
```

L'algorithme est un simple parcours avec avance :

```dart
  void _construireDecor() {
    for (int l = 0; l < lignes; l++) {
      final String ligne = carte[l];
      int c = 0;

      while (c < colonnes) {
        final String caractere = ligne[c];

        // Ni mur ni plateforme : on avance d'une case.
        if (caractere != NiveauxData.mur &&
            caractere != NiveauxData.plateforme) {
          c++;
          continue;
        }

        // On étend tant que le caractère est le MÊME.
        int fin = c;
        while (fin + 1 < colonnes && ligne[fin + 1] == caractere) {
          fin++;
        }

        final int nombreDeTuiles = fin - c + 1;
        final bool traversable = caractere == NiveauxData.plateforme;

        plateformes.add(
          Plateforme(
            position: coinDe(c, l),
            size: Vector2(
              nombreDeTuiles * t,
              traversable ? epaisseurPlateforme : t,
            ),
            traversable: traversable,
          ),
        );

        // On saute par-dessus toute la suite fusionnée.
        c = fin + 1;
      }
    }
  }
```

**Résultat sur le niveau 1 :**

```text
110 caractères '#'  ->  22 Plateforme solides
 24 caractères '='  ->   3 Plateforme traversables
                        ─────────────────────────
                        25 composants au lieu de 134
```

Un point de détail qui n'en est pas un : la condition d'extension est `ligne[fin + 1] == caractere`, pas `ligne[fin + 1] != vide`. Un `#` et un `=` collés ne doivent **jamais** fusionner : ils n'ont ni la même hauteur ni le même comportement.

### La plateforme traversable

Le cahier des charges dit : `=` est « traversable par le bas ». Concrètement :

```text
   Le héros MONTE                Le héros DESCEND

        ╱                              ╲
   ====▓====  il passe             ====▓====  il se pose
       ▓                               ▓
```

Cela demande deux changements. D'abord un drapeau sur la plateforme :

```dart
// lib/composants/plateforme.dart — MODIFIÉ AU CHAPITRE 39
class Plateforme extends PositionComponent {
  Plateforme({
    required Vector2 position,
    required Vector2 size,
    this.traversable = false,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  /// Vrai pour les tuiles '=' : on ne les heurte que par le dessus.
  final bool traversable;

  @override
  Future<void> onLoad() async {
    add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()
          ..color = traversable
              ? Palette.plateformeTraversable
              : Palette.plateforme,
      ),
    );
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}
```

Ensuite une règle de résolution différente, à ajouter dans `Joueur` **et** dans `Ennemi` (le code est identique dans les deux, on le recopie) :

```dart
  /// Chevauchement maximal toléré pour un atterrissage sur plateforme.
  /// Au-delà, on considère que l'entité est DÉJÀ passée au travers.
  static const double _seuilAtterrissage = 14.0;

  void _resoudreCollision(Plateforme plateforme) {
    final moi = toAbsoluteRect();
    final lui = plateforme.toAbsoluteRect();
    final inter = moi.intersect(lui);
    if (inter.width <= 0 || inter.height <= 0) {
      return;
    }

    // NOUVEAU : une plateforme traversable ne bloque QUE par le dessus.
    if (plateforme.traversable) {
      _resoudreTraversable(moi, lui, inter);
      return;
    }

    // ... le code solide du chapitre 37, inchangé
  }

  void _resoudreTraversable(Rect moi, Rect lui, Rect inter) {
    if (velocite.y <= 0) {
      return;                       // il monte ou il stagne : il traverse
    }
    if (inter.width < inter.height) {
      return;                       // contact latéral : il traverse
    }
    if (moi.center.dy >= lui.center.dy) {
      return;                       // il arrive par en dessous : il traverse
    }
    if (inter.height > _seuilAtterrissage) {
      return;                       // trop enfoncé : trop tard pour se poser
    }

    position.y -= inter.height;
    velocite.y = 0;
    auSol = true;
  }
```

Les quatre gardes se lisent comme une phrase : **on ne se pose que si l'on descend, par le dessus, à plat, et pas trop tard**.

> **Pourquoi 14 px ?** À la vitesse de chute maximale (`vitesseMaxChute = 900` px/s) et à 60 images par seconde, une frame déplace le joueur de 15 px. Un seuil un peu inférieur garantit qu'un joueur en pleine chute libre ne « raccroche » pas une plateforme qu'il a déjà dépassée d'une frame entière. Un seuil trop grand ferait remonter le héros brutalement ; un seuil trop petit le laisserait passer au travers. Réglez-le en jouant.

### Les rayons des ennemis doivent en tenir compte

Les ennemis du chapitre 37 sondent le décor avec `raycast` (`solDevant()` et `murDevant()`). Il faut leur apprendre la nuance :

- **`solDevant()`** : une plateforme traversable **est** du sol. Le filtre ne change pas.
- **`murDevant()`** : une plateforme traversable n'est **pas** un mur. Un gobelin ne doit pas faire demi-tour parce qu'un perchoir lui passe devant le nez.

```dart
// lib/composants/ennemi.dart — MODIFIÉ AU CHAPITRE 39
  bool murDevant() {
    final origine = absoluteCenter + Vector2(0, size.y * 0.25);
    final resultat = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: Vector2(sens.toDouble(), 0)),
      maxDistance: size.x * 0.5 + 4,
      hitboxFilter: (candidate) {
        final parent = candidate.parent;
        // NOUVEAU : on ignore les plateformes traversables.
        return parent is Plateforme && !parent.traversable;
      },
    );
    return resultat != null;
  }
```

La promotion de type fonctionne ici parce que `parent` est une **variable locale** : `parent is Plateforme` autorise `parent.traversable` juste après. Sur un champ d'instance, ce ne serait pas le cas — c'est la règle du chapitre 12.

---

## 39.9 — Instancier les ennemis

### Le principe : un caractère, un constructeur

```dart
  void _construireEntites() {
    for (int l = 0; l < lignes; l++) {
      final String ligne = carte[l];

      for (int c = 0; c < colonnes; c++) {
        switch (ligne[c]) {
          case NiveauxData.gobelin:
            ennemis.add(
              Gobelin(position: poserSurTuile(c, l, tailleGobelin)),
            );
            break;

          case NiveauxData.chauvesouris:
            ennemis.add(
              Chauvesouris(
                position: centreDe(c, l) - tailleChauvesouris / 2,
              ),
            );
            break;

          case NiveauxData.boss:
            final Boss gardien =
                Boss(position: poserSurTuile(c, l, tailleBoss));
            boss = gardien;
            ennemis.add(gardien);
            break;

          // ... les autres cas en 39.10 et 39.11
        }
      }
    }
  }
```

Avec, en tête de classe, les tailles exactes déclarées par les composants des chapitres 37 et 39 :

```dart
  static final Vector2 tailleJoueur = Vector2(24, 32);
  static final Vector2 tailleGobelin = Vector2(24, 30);
  static final Vector2 tailleChauvesouris = Vector2(22, 16);
  static final Vector2 tailleBoss = Vector2(48, 56);
```

> **Attention aux `Vector2` partagés.** Ces quatre constantes sont `static final`, donc une seule instance pour tout le programme. `Vector2` est **mutable** (piège rappelé au chapitre 27) : ne les passez jamais directement comme `position` ou comme `size` d'un composant. Ici elles ne servent qu'à **calculer** une position : `poserSurTuile` renvoie un `Vector2` neuf. C'est sûr.

### Terrien contre volant : deux façons de poser

C'est la seule vraie subtilité de cette section.

```text
  GOBELIN (terrien)                 CHAUVE-SOURIS (volante)
  poserSurTuile(c, l, taille)       centreDe(c, l) - taille / 2

  ┌────┬────┐                       ┌────┬────┐
  │    │    │                       │  ▓▓▓▓   │  <- centrée dans la tuile
  │  ▓▓│▓▓  │                       │  ▓▓▓▓   │
  ├──▓▓┼▓▓──┤ <- pieds sur l'arête  ├────┼────┤
  │####│####│                       │    │    │
```

Un gobelin qui n'est pas posé au sol tombe d'une demi-tuile au démarrage, traverse éventuellement le sol si la vitesse de chute initiale est mal gérée, et se retrouve dans le vide. Une chauve-souris posée au sol reste collée au sol : elle ne subit pas la gravité (`subitGravite => false` au chapitre 37) et son ondulation se calcule autour de son **point d'apparition**, qui deviendrait le plancher.

### Le rappel du chapitre 37 : l'ancre de patrouille

Souvenez-vous du champ `ancre` d'`Ennemi` :

```dart
  @override
  Future<void> onLoad() async {
    initialiserSante(_pvDepart);
    ancre = position.clone();      // <- ici
    // ...
  }
```

`ancre` est le point d'apparition mémorisé. Le gobelin patrouille entre `ancre.x - demiPatrouille` et `ancre.x + demiPatrouille` ; la chauve-souris ondule autour de `ancre.y`. Autrement dit : **la position que vous donnez dans la carte détermine tout le comportement de l'ennemi**. Un `g` mal placé, à cheval sur un trou, produit un gobelin qui fait demi-tour toutes les deux frames.

### Le cas du boss : une seule instance

`boss` est un champ de `Niveau`, pas une liste. Deux `B` sur une carte donneraient deux boss, deux barres de vie qui se battent pour le même emplacement de HUD, et une porte qui ne sait plus qui attendre. Le validateur de la section 39.7 n'interdit pas explicitement le double `B` ; l'exercice 3 vous demandera de l'ajouter.

Le boss est ajouté **à `ennemis`** en plus d'être mémorisé dans `boss` : c'est un ennemi comme un autre pour le monde, et un objet particulier pour le jeu.

---

## 39.10 — Instancier les collectibles

Les trois collectibles du chapitre 38 partagent une caractéristique : leur `anchor` est `Anchor.center`. Ils se posent donc **au centre de la tuile**, sans calcul.

```dart
          case NiveauxData.piece:
            collectibles.add(Piece(position: centreDe(c, l)));
            nombreDePieces++;
            break;

          case NiveauxData.potion:
            collectibles.add(Potion(position: centreDe(c, l)));
            break;

          case NiveauxData.cle:
            collectibles.add(Cle(position: centreDe(c, l)));
            break;
```

### Compter les pièces au bon moment

Le compteur `nombreDePieces` est incrémenté **pendant l'analyse**, c'est-à-dire avant tout ajout au monde. C'est la correction du piège signalé au chapitre 38 :

> « `piecesDuNiveau` vaut 0 alors qu'il y a des pièces : on compte `world.children` juste après `add()`, qui est asynchrone. »

En comptant dans le parseur, le problème disparaît par construction. Quand `chargerNiveau` récupère `niveau.nombreDePieces`, la valeur est exacte depuis longtemps, et aucun `await` n'est nécessaire.

### Le flottement se désynchronise tout seul

Rien à faire ici. `Collectible.onLoad` du chapitre 38 ajoute déjà un `MoveEffect` de flottement avec un `startDelay` tiré au hasard :

```dart
        startDelay: hasard.nextDouble() * dureeFlottement,
```

Les 23 pièces du niveau 2 sont créées dans la même milliseconde ; elles flotteront pourtant chacune à son rythme. C'est le genre de détail qui fait la différence entre « une grille de pièces » et « une salle vivante ».

### Le tableau récapitulatif des ancrages

| Composant | `anchor` | Fonction de placement | Raison |
| --- | --- | --- | --- |
| `Plateforme` | `topLeft` | `coinDe(c, l)` | elle remplit la tuile |
| `Joueur` | `topLeft` | `poserSurTuile` | il tombe, il doit être au sol |
| `Gobelin` | `topLeft` | `poserSurTuile` | il tombe |
| `Boss` | `topLeft` | `poserSurTuile` | il tombe |
| `Chauvesouris` | `topLeft` | `centreDe - taille / 2` | elle vole |
| `Piece`, `Potion`, `Cle` | `center` | `centreDe` | ils flottent et pulsent autour de leur centre |
| `Porte` | `topLeft` | `poserSurTuile` | elle est posée sur le sol |

Retenez la règle générale : **un composant qui grossit ou rétrécit (`ScaleEffect`) doit être ancré au centre**, sinon il grossit vers le bas-droite. Un composant qui doit reposer sur une surface est ancré en haut à gauche et positionné par calcul.

---

## 39.11 — Le point d'apparition du joueur

### Le cas `J`

```dart
          case NiveauxData.joueur:
            apparitionJoueur = poserSurTuile(c, l, tailleJoueur);
            joueur = Joueur(position: apparitionJoueur.clone());
            break;
```

Deux lignes, deux pièges évités.

**Piège 1 — le `clone()`.** `apparitionJoueur` sera conservé par `DonjonGame` comme point de réapparition après une mort. Si on passait la **même** instance de `Vector2` au constructeur du `Joueur`, alors le champ `position` du joueur et le point de réapparition seraient le même objet. Le héros bouge, `position` change, le point de réapparition suit… et le joueur réapparaît là où il vient de mourir. Boucle infinie de morts.

```text
  SANS clone()                          AVEC clone()

  apparitionJoueur ─┐                   apparitionJoueur ──> (100, 386)
                    ├──> Vector2(100,386)
  joueur.position ──┘                   joueur.position  ──> (100, 386) copie

  Le héros marche : LES DEUX bougent.   Le héros marche : seule sa position bouge.
```

**Piège 2 — l'ordre.** `apparitionJoueur` est calculé **avant** de construire le `Joueur`, pas déduit de `joueur!.position` après coup. Cela garde la valeur correcte même si le joueur n'a pas encore été monté.

### Le point de réapparition à l'échelle du jeu

`DonjonGame` possède depuis le chapitre 37 :

```dart
  Vector2 pointDeReapparition = Vector2(48, 150);
```

`chargerNiveau` va simplement l'écraser :

```dart
    pointDeReapparition = niveau.apparitionJoueur.clone();
```

Encore un `clone()`. Oui, deux `clone()` pour la même valeur, c'est volontaire : chaque propriétaire d'un `Vector2` doit posséder **sa** copie. Le prix est de deux `double` en mémoire ; le bénéfice est de ne jamais avoir à se demander qui modifie quoi.

### Assembler : `construireDans`

```dart
  /// Ajoute tout le niveau au monde, DANS L'ORDRE.
  Future<void> construireDans(World monde) async {
    analyser();

    // 1. Le décor d'abord : les rayons des ennemis doivent le trouver.
    await monde.addAll(plateformes);

    // 2. Le héros ensuite : chaque Ennemi lit game.joueur à son montage.
    final Joueur? heros = joueur;
    if (heros != null) {
      await monde.add(heros);
    }

    // 3. La porte, avant les objets : elle est en arrière-plan.
    final Porte? sortie = porte;
    if (sortie != null) {
      await monde.add(sortie);
    }

    // 4. Les objets et les ennemis, qui dépendent de tout le reste.
    await monde.addAll(collectibles);
    await monde.addAll(ennemis);
  }
```

**Résultat attendu au chargement du niveau 1 :**

```text
Niveau "Les caves" : 40 x 14 tuiles (1280 x 448 px)
  25 plateformes  (22 solides, 3 traversables)
   1 joueur en (100, 386)
   1 porte en (1188, 386)
   4 ennemis  (2 gobelins, 2 chauves-souris)
  12 collectibles (10 pièces, 1 potion, 1 clé)
```

---

## 39.12 — `chargerNiveau(int index)` sur `DonjonGame`

### La signature imposée

Le cahier des charges du jeu la fixe depuis le chapitre 35 :

```dart
  Future<void> chargerNiveau(int index);    // ch. 39
```

`Future<void>` parce que l'ajout de composants est asynchrone (chapitre 28). `int index` parce que les niveaux sont numérotés de 0 à `Constantes.nombreNiveaux - 1`.

### Le code

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 39

  /// Le niveau en cours, ou null entre deux niveaux.
  Niveau? niveauEnCours;

  /// La porte de sortie du niveau courant.
  Porte? porte;

  /// Le boss du niveau courant, s'il y en a un.
  Boss? boss;

  /// Le plus haut niveau atteint dans cette partie (sauvegardé au ch. 40).
  int niveauMaxAtteint = 0;

  /// Charge le niveau [index] et remplace complètement le monde.
  Future<void> chargerNiveau(int index) async {
    // 0. Le moteur DOIT tourner : voir l'encadré ci-dessous.
    resumeEngine();

    final int cible = index.clamp(0, Constantes.nombreNiveaux - 1);
    niveauCourant = cible;
    if (cible > niveauMaxAtteint) {
      niveauMaxAtteint = cible;
    }

    // 1. Faire le vide (section 39.13).
    await viderLeMonde();

    // 2. Traduire la carte (sections 39.6 à 39.11).
    final Niveau niveau = Niveau.numero(cible);
    await niveau.construireDans(monde);

    // 3. Mémoriser ce dont le jeu aura besoin.
    niveauEnCours = niveau;
    porte = niveau.porte;
    boss = niveau.boss;
    pointDeReapparition = niveau.apparitionJoueur.clone();
    piecesDuNiveau = niveau.nombreDePieces;
    piecesRamassees = 0;
    reinitialiserCombo();

    // 4. Recaler la caméra (section 39.14).
    final Joueur? heros = niveau.joueur;
    if (heros != null) {
      camera.follow(heros, snap: true);
    }
    bornerLaCamera(niveau);
  }
```

### `resumeEngine()` en première ligne : le piège mortel

C'est probablement le piège le plus vicieux de tout le chapitre, et il touchait déjà le code des chapitres 35 à 38 sans qu'on s'en aperçoive.

```text
  changerEtat(GameState.menu)  ->  pauseEngine()
                                        │
                                        v
                              la boucle de jeu s'arrête
                                        │
                                        v
                       processLifecycleEvents() n'est plus appelée
                                        │
                                        v
                          await monde.add(joueur)  ...  n'aboutit JAMAIS
```

`Component.add()` ne monte pas le composant immédiatement : il **empile un événement de cycle de vie**, traité au tour suivant de la boucle de jeu. Le `Future` renvoyé se complète au montage. Si le moteur est en pause, la boucle ne tourne pas, l'événement n'est jamais traité, et votre `await` reste suspendu pour l'éternité. Le jeu affiche le menu, vous cliquez sur « Jouer », et **rien ne se passe** — sans erreur, sans exception, sans message.

La règle est simple :

> **On ne charge jamais un niveau moteur en pause.** `resumeEngine()` d'abord, `await` ensuite.

### `demarrerPartie()` réécrite

```dart
  Future<void> demarrerPartie({Difficulte? mode}) async {
    if (mode != null) {
      difficulte = mode;
    }

    resumeEngine();

    score = 0;
    viesMax = reglages.vies;
    vies = viesMax;
    niveauCourant = 0;
    niveauMaxAtteint = 0;
    transitionEnCours = false;
    reinitialiserCombo();

    await chargerNiveau(0);
    await preparerHud();

    changerEtat(GameState.enJeu);
  }
```

### Le HUD ne doit être créé qu'une fois

Le chapitre 38 écrivait :

```dart
  late final Hud hud;
  // ...
  hud = Hud();
  await camera.viewport.add(hud);
```

Cela fonctionne **une seule fois**. Au deuxième appel de `demarrerPartie` — c'est-à-dire dès que le joueur clique sur « Rejouer » — Dart lève :

```text
LateInitializationError: Field 'hud' has already been initialized.
```

Le correctif, à appliquer maintenant :

```dart
  Hud? _hud;

  /// Le HUD. Ne l'appelez qu'après `preparerHud()`.
  Hud get hud => _hud!;

  /// Crée le HUD au premier appel, le recale aux suivants.
  Future<void> preparerHud() async {
    if (_hud == null) {
      final Hud nouveau = Hud();
      _hud = nouveau;
      await camera.viewport.add(nouveau);   // LE HUD VIT DANS LE VIEWPORT
    }
    hud.compteurScore.synchroniserImmediatement();
  }
```

Le HUD vit dans le **viewport**, pas dans le monde (chapitre 38.14) : c'est précisément pour cela qu'il survit intact au vidage du monde. On le crée une fois, il reste pour toute la durée de vie du jeu.

### Ce que `chargerNiveau` ne fait PAS

| Ne fait pas | Qui s'en charge |
| --- | --- |
| remettre le score à zéro | `demarrerPartie` — le score se cumule d'un niveau à l'autre |
| remettre les vies à zéro | `demarrerPartie` — les vies aussi se conservent |
| changer l'état du jeu | `demarrerPartie` ou `terminerNiveau` |
| jouer une musique | chapitre 40 |
| afficher la transition | `terminerNiveau`, section 39.17 |

`chargerNiveau` remplace le contenu du monde. Rien de plus. Cette discipline permet de l'appeler depuis trois endroits différents (démarrage, transition, et plus tard reprise d'une sauvegarde) sans effet de bord surprise.

---

## 39.13 — Vider le monde avant de recharger : le piège des composants orphelins

### Ce qui reste quand on croit avoir tout retiré

Le chapitre 37 vidait le monde comme ceci :

```dart
    monde.removeAll(monde.children.toList());
```

La ligne est correcte, mais elle ne suffit plus. Quatre choses survivent au vidage :

```text
  ┌──────────────────────── APRÈS monde.removeAll(...) ─────────────────────┐
  │                                                                         │
  │  RETIRÉS                          SURVIVANTS (les orphelins)            │
  │  ────────                         ───────────────────────────           │
  │  Plateforme x 25                  camera.follow(ancien joueur)   (1)    │
  │  Joueur                           game.joueur -> ancien joueur   (2)    │
  │  Gobelin x 2                      game.boss   -> ancien boss     (3)    │
  │  Piece x 10                       camera.setBounds(ancien monde) (4)    │
  │  Porte                            Hud (dans le viewport : normal)       │
  │                                                                         │
  └─────────────────────────────────────────────────────────────────────────┘
```

Détaillons chacun.

**(1) La caméra suit un fantôme.** `camera.follow(joueur)` installe un `FollowBehavior` qui garde une référence vers le composant cible. Retirer le composant du monde ne retire pas le comportement de la caméra. Résultat : la caméra continue de viser les dernières coordonnées connues d'un composant mort, et le nouveau joueur est hors champ. Correctif : `camera.stop()`.

**(2) et (3) Les références du jeu.** `game.joueur` et `game.boss` sont réaffectés par les `onMount` des nouveaux composants — mais entre le vidage et le montage, ils pointent sur des cadavres. Tout code qui s'exécute dans cet intervalle (le HUD, par exemple, qui tourne toujours dans le viewport) lira des valeurs obsolètes. Correctif : les mettre à `null` explicitement.

**(4) Les bornes de caméra.** Elles décrivent l'ancien niveau. Si le suivant est plus petit, la caméra pourra sortir du monde. Correctif : `bornerLaCamera` après chaque chargement (section 39.14).

### Le code complet de `viderLeMonde`

```dart
  /// Retire tout le contenu du monde et coupe toutes les références
  /// qui pointeraient vers des composants retirés.
  Future<void> viderLeMonde() async {
    // 1. La caméra ne doit plus suivre personne.
    camera.stop();

    // 2. Les références du jeu.
    joueur = null;
    boss = null;
    porte = null;
    niveauEnCours = null;

    // 3. Le contenu du monde.
    //    `.toList()` fait une COPIE : on ne parcourt pas une collection
    //    qu'on est en train de modifier (chapitre 06).
    monde.removeAll(monde.children.toList());

    // 4. On rend la main au moteur avant de reconstruire.
    await Future<void>.delayed(Duration.zero);
  }
```

> **Sur la ligne 4.** Elle n'est pas strictement obligatoire : les retraits et les ajouts sont empilés dans **la même file**, dans l'ordre, et Flame traite cette file dans l'ordre. Les retraits qui précèdent sont donc traités avant les ajouts qui suivent, même sans attente explicite. Le `await` rend simplement l'intention lisible et laisse une frame au moteur. N'essayez pas de vérifier `monde.children.isEmpty` juste après `removeAll` : la collection est encore pleine, exactement comme `add()` ne monte pas immédiatement (chapitre 28).

### `game.joueur` peut désormais être `null`

C'est la conséquence directe du vidage, et elle touche tout le code écrit au chapitre 37. Pendant la transition entre deux niveaux, `game.joueur` vaut `null`. Toute IA qui écrit `game.joueur.absoluteCenter` plantera avec :

```text
The property 'absoluteCenter' can't be unconditionally accessed
because the receiver can be 'null'.
```

On ajoute donc un accesseur unique sur `Ennemi` et on l'utilise partout :

```dart
// lib/composants/ennemi.dart — AJOUT DU CHAPITRE 39
  /// Le héros, ou null pendant une transition de niveau.
  Joueur? get cible => game.joueur;
```

Puis on protège les quatre méthodes concernées :

```dart
// lib/composants/ennemi.dart — MODIFIÉ
  double get distanceAuJoueur {
    final Joueur? heros = cible;
    if (heros == null || !heros.isMounted) {
      return double.infinity;
    }
    return (heros.absoluteCenter - absoluteCenter).length;
  }

  bool voitLeJoueur() {
    final Joueur? heros = cible;
    if (heros == null || !heros.isMounted || !heros.estVivant) {
      return false;
    }
    final vers = heros.absoluteCenter - absoluteCenter;
    // ... la suite du chapitre 37, inchangée
  }
```

```dart
// lib/composants/gobelin.dart — MODIFIÉ
  void _poursuivre(double dt) {
    final Joueur? heros = cible;
    if (heros == null) {
      etat = EtatEnnemi.retour;
      return;
    }
    final ecartX = heros.absoluteCenter.x - absoluteCenter.x;
    // ... la suite du chapitre 37, inchangée
  }
```

```dart
// lib/composants/chauvesouris.dart — MODIFIÉ
  void _poursuivreEnVol(double dt) {
    final Joueur? heros = cible;
    if (heros == null) {
      etat = EtatEnnemi.retour;
      return;
    }
    final vers = heros.absoluteCenter - absoluteCenter;
    // ... la suite du chapitre 37, inchangée
  }
```

Le même traitement s'applique à `_declencherPique()` et à `tirer()`.

> **Le motif à retenir.** On ne teste jamais `if (game.joueur != null)` puis on utilise `game.joueur.pv` : Dart ne promeut pas un champ d'instance, parce qu'un autre bout de code pourrait le remettre à `null` entre les deux lignes. On **copie dans une variable locale**, on teste la locale, on utilise la locale. C'est la règle du chapitre 12, et le chapitre 38 la rappelait déjà pour le HUD.

### La liste de contrôle du vidage

| Question | Réponse attendue |
| --- | --- |
| La caméra suit-elle encore quelqu'un ? | non — `camera.stop()` |
| `game.joueur` pointe-t-il sur un composant retiré ? | non — remis à `null` |
| `game.boss` et `game.porte` ? | non — remis à `null` |
| Le HUD a-t-il été détruit ? | non, et c'est voulu : il est dans le viewport |
| Les bornes de caméra sont-elles à jour ? | pas encore — section suivante |
| Reste-t-il un `Projectile` de l'ancien niveau ? | non : il était dans le monde |

---

## 39.14 — Borner la caméra au niveau (rappel chapitre 31)

### Le problème, en une image

```text
   SANS BORNES                              AVEC BORNES

   ████ bande noire ██████                  ┌─────────────────┐
   ┌─────────────────┐                      │#################│
   │#################│                      │#...............#│
   │#....J..........#│                      │#.J.............#│
   │#################│                      │#################│
   └─────────────────┘                      └─────────────────┘
   Le héros longe le mur gauche :           La caméra bute sur le bord
   la caméra le centre et montre            du monde et laisse le héros
   du vide hors du monde.                   se décaler à l'écran.
```

Au chapitre 25 vous corrigiez cela avec deux `clamp` écrits à la main. Le chapitre 31 a montré l'outil de Flame :

```dart
void setBounds(Shape? bounds, {bool considerViewport = false});
```

Le type `Shape` vient de `package:flame/experimental.dart`, et le constructeur qui nous intéresse est `Rectangle.fromLTRB`.

### Le code

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 39
import 'package:flame/experimental.dart';

  /// Empêche la caméra de sortir du niveau chargé.
  void bornerLaCamera(Niveau niveau) {
    // Taille de la portion de monde visible, zoom compris.
    final Vector2 vue = camera.viewport.size / camera.viewfinder.zoom;

    // Un niveau plus petit que l'écran donnerait des bornes inversées.
    final double largeur = max(niveau.largeurPixels, vue.x);
    final double hauteur = max(niveau.hauteurPixels, vue.y);

    camera.setBounds(
      Rectangle.fromLTRB(0, 0, largeur, hauteur),
      considerViewport: true,
    );
  }
```

### Les trois points à ne pas rater

**1. `considerViewport: true`.** Sans lui, `setBounds` limite le **centre du regard**, pas les bords de l'écran : la moitié gauche de l'écran peut sortir du monde et vous voyez du noir. Avec lui, c'est l'écran entier qui reste dedans. C'est presque toujours ce que vous voulez. Le schéma du chapitre 31.16 vaut la peine d'être relu.

**2. Le `max` de sécurité.** Si le niveau est plus petit que la zone visible, `considerViewport: true` produit un rectangle de bornes de largeur négative. Selon les versions, cela donne une assertion ou un comportement erratique. Le `max` garantit des bornes toujours au moins aussi grandes que l'écran.

**3. Rappeler `bornerLaCamera` après tout changement de zoom.** Les bornes sont calculées à partir du zoom courant. Si vous ajoutez un jour un effet de dézoom pendant le combat de boss, il faudra recalculer. Le chapitre 31.16 le disait déjà.

### Vérification chiffrée

Avec une fenêtre de 1280 × 720 et `zoomCamera = 2.0` :

```text
vue = viewport.size / zoom = (1280, 720) / 2 = (640, 360) px de monde visibles

Niveau 1 : 1280 x 448 px    -> bornes (0, 0, 1280, 448)    défilement horizontal
Niveau 2 : 1536 x 576 px    -> bornes (0, 0, 1536, 576)    défilement dans les 2 axes
Niveau 3 : 1280 x 448 px    -> bornes (0, 0, 1280, 448)    défilement horizontal

Sur une petite fenêtre de 500 x 300 :
vue = (250, 150)            -> aucun niveau n'est plus petit : le max ne sert pas
Sur une immense fenêtre de 4000 x 1200 :
vue = (2000, 600)           -> le max relève les bornes du niveau 1 à (2000, 600)
```

---

## 39.15 — `lib/composants/porte.dart`

### Le rôle de la porte

C'est le seul composant du jeu qui **termine** quelque chose. Ses responsabilités :

1. se dessiner (un battant, un cadre, une serrure) ;
2. savoir si elle est verrouillée, et par quoi : une clé ou un boss ;
3. réagir au contact du joueur ;
4. appeler `game.terminerNiveau()` **une seule fois**.

### Une porte est un capteur, pas un mur

Comme les collectibles du chapitre 38, la porte porte une hitbox **passive** : elle ne bloque pas le joueur, elle le détecte. Un joueur bloqué par la porte qu'il vient d'ouvrir serait absurde.

```dart
    add(RectangleHitbox(collisionType: CollisionType.passive));
```

Rappel de la règle de collision du chapitre 32 : deux hitbox `passive` ne se voient **jamais**. La porte n'est donc vue que par le joueur, qui est `active`. Un projectile de chauve-souris ne la déclenchera pas — ce qui est exactement le comportement voulu.

### L'état de la porte

Le fichier complet figure en section 39.34. Voici les champs et l'entête, qui portent toute la logique.

```dart
// lib/composants/porte.dart
class Porte extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Porte({required Vector2 position, this.verrouillee = true})
      : super(
          position: position,
          size: Vector2(Constantes.largeurPorte, Constantes.hauteurPorte),
          anchor: Anchor.topLeft,
          priority: -1,          // derrière le joueur
        );

  /// Verrouillée : il faut une clé.
  bool verrouillee;

  /// Bloquée tant que le boss du niveau est vivant.
  /// Positionné par `Niveau` APRÈS analyse complète de la carte.
  bool attendLeBoss = false;

  bool _ouverte = false;     // le battant est-il ouvert ?
  bool _franchie = false;    // verrou : on ne finit un niveau qu'une fois
  double _delaiRefus = 0;    // throttle du message de refus

  bool get ouverte => _ouverte;

  @override
  Future<void> onLoad() async {
    // trois RectangleComponent : battant, cadre, serrure
    add(RectangleHitbox(collisionType: CollisionType.passive));

    // Une porte ni verrouillée ni gardée est ouverte dès le départ.
    if (!verrouillee && !attendLeBoss) {
      ouvrir(silencieux: true);
    }
  }
}
```

### Le rendu sans image

Trois rectangles suffisent :

```text
  ┌────────────┐  <- cadre : RectangleComponent en PaintingStyle.stroke
  │▓▓▓▓▓▓▓▓▓▓▓▓│
  │▓▓▓▓▓▓▓▓▓▓▓▓│  <- battant : rectangle plein, couleur bois
  │▓▓▓▓▓▓▓▓▓█▓▓│  <- serrure : petit rectangle doré, retiré à l'ouverture
  │▓▓▓▓▓▓▓▓▓▓▓▓│
  └────────────┘
```

Le `PaintingStyle.stroke` sur un `RectangleComponent` dessine seulement le contour : c'est la technique du chapitre 21, appliquée à un composant Flame. `strokeWidth = 3` donne un cadre épais et lisible même au zoom 2.

Le jour où vous aurez des sprites, ces trois enfants deviennent un `SpriteAnimationComponent` à deux animations (`fermee`, `ouverte`) et **le reste du fichier ne bouge pas**.

---

## 39.16 — La porte verrouillée et la clé (rappel chapitre 38)

### Ce que le chapitre 38 a déjà écrit

Le joueur possède depuis le chapitre 38 :

```dart
  /// Clés en poche. Consommées par les portes verrouillées (chapitre 39).
  int cles = 0;

  bool get possedeUneCle => cles > 0;

  /// Consomme une clé. Renvoie faux s'il n'y en avait pas.
  bool consommerUneCle() {
    if (cles <= 0) {
      return false;
    }
    cles--;
    return true;
  }
```

`consommerUneCle()` fait deux choses en une : elle **teste** et elle **consomme**, atomiquement. C'est ce qui empêche le grand classique du « il reste une clé, mais deux portes se sont ouvertes ». La porte n'a donc rien à décider : elle appelle, elle lit le booléen.

### Les trois verrous possibles

| Verrou | Condition d'ouverture | Niveau |
| --- | --- | --- |
| `verrouillee = true` | le joueur possède une clé | 1 et 2 |
| `attendLeBoss = true` | le boss du niveau est mort | 3 |
| aucun | ouverte dès l'apparition | (aucun pour l'instant) |

Les deux verrous sont **indépendants** et cumulables. Une porte pourrait exiger une clé **et** la mort du boss ; nos trois niveaux n'utilisent qu'un verrou à la fois.

### La logique de franchissement

```dart
  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);
    if (other is Joueur) {
      _essayerDeFranchir(other);
    }
  }

  @override
  void onCollision(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollision(intersectionPoints, other);
    // On ré-essaie à chaque frame UNIQUEMENT si la porte s'est ouverte
    // pendant que le joueur était déjà dedans (cas du boss vaincu).
    if (other is Joueur && _ouverte) {
      _essayerDeFranchir(other);
    }
  }

  void _essayerDeFranchir(Joueur heros) {
    if (_franchie) {
      return;                          // verrou définitif
    }

    if (attendLeBoss) {
      _refuser('Le gardien veille encore');
      return;
    }

    if (verrouillee) {
      if (!heros.consommerUneCle()) {
        _refuser('Il faut une clé');
        return;
      }
      ouvrir();
    }

    if (!_ouverte) {
      return;
    }

    _franchie = true;
    game.terminerNiveau();
  }
```

### Pourquoi `onCollision` en plus de `onCollisionStart`

`onCollisionStart` n'est appelée **qu'une fois**, à l'instant du contact. Or il existe un cas où le joueur est déjà en contact quand la porte s'ouvre : il attend devant la porte du niveau 3 pendant qu'un allié imaginaire… non, plus simplement : il tue le boss en étant collé à la porte. `onCollisionStart` est passée depuis longtemps, la porte s'ouvre, et rien ne se produit tant qu'il ne ressort pas pour revenir.

`onCollision`, appelée à chaque frame de contact, corrige ce cas — mais seulement si la porte est déjà ouverte, pour ne pas relancer la logique de refus soixante fois par seconde.

### Le refus poli

```dart
  void _refuser(String message) {
    if (_delaiRefus > 0) {
      return;                          // pas plus d'un refus toutes les 1,5 s
    }
    _delaiRefus = 1.5;

    add(
      MoveEffect.by(
        Vector2(3, 0),
        EffectController(duration: 0.05, alternate: true, repeatCount: 3),
      ),
    );
    game.afficherTexteFlottant(
      absoluteCenter,
      message,
      Palette.serrure,
      taille: 10,
    );
  }
```

Trois éléments de conception, tous délibérés :

1. **Un throttle.** Sans `_delaiRefus`, `onCollision` créerait un texte flottant par frame : soixante « Il faut une clé » superposés. Le chapitre 38 signalait déjà ce piège pour les dégâts continus.
2. **Une secousse.** Un `MoveEffect.by` avec `alternate: true` revient exactement à sa position de départ. Il n'y a aucune dérive à craindre, contrairement à un `MoveEffect.to`.
3. **Un message, pas un silence.** Un joueur qui pousse une porte sans réaction croit à un bug. Un joueur qui lit « Il faut une clé » comprend l'objectif.

### L'ouverture

```dart
  void ouvrir({bool silencieux = false}) {
    if (_ouverte) {
      return;
    }
    _ouverte = true;
    verrouillee = false;
    attendLeBoss = false;

    _serrure.removeFromParent();
    _battant.paint.color = Palette.porteOuverte;

    if (!silencieux) {
      add(
        ScaleEffect.by(
          Vector2(1.0, 1.12),
          EffectController(duration: 0.12, alternate: true),
        ),
      );
      game.afficherTexteFlottant(
        absoluteCenter,
        'Ouverte !',
        Palette.serrure,
      );
    }
  }

  /// Appelée par `DonjonGame` quand le boss du niveau tombe.
  void deverrouillerParLeBoss() {
    attendLeBoss = false;
    if (!verrouillee) {
      ouvrir();
    }
  }
```

Le paramètre `silencieux` évite un « Ouverte ! » au chargement d'une porte déjà déverrouillée. C'est le genre de détail qu'on n'ajoute qu'après l'avoir vu à l'écran.

---

## 39.17 — `terminerNiveau()` et la transition

### La signature imposée

```dart
  void terminerNiveau();                    // ch. 39
```

`void`, et non `Future<void>`. C'est un choix du cahier des charges, et c'est le bon : la porte appelle cette méthode depuis un callback de collision. Un callback de collision **ne doit jamais attendre**. `terminerNiveau` déclenche, elle n'attend pas.

### Le code

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 39
  /// Vrai pendant tout le fondu entre deux niveaux.
  bool transitionEnCours = false;

  /// Le niveau courant est réussi : bonus, puis niveau suivant ou victoire.
  void terminerNiveau() {
    if (transitionEnCours) {
      return;                              // garde de ré-entrance
    }
    transitionEnCours = true;

    _accorderLesBonus();

    final int suivant = niveauCourant + 1;
    if (suivant >= Constantes.nombreNiveaux) {
      _terminerLAventure();
      return;
    }

    // On lance la transition sans l'attendre : `unawaited` le dit
    // explicitement au lecteur et à l'analyseur (import 'dart:async').
    unawaited(_enchainerVers(suivant));
  }

  void _accorderLesBonus() {
    int bonus = Constantes.bonusFinNiveau;
    if (piecesDuNiveau > 0 && piecesRamassees >= piecesDuNiveau) {
      bonus += Constantes.bonusSalleVidee;
    }
    // Un bonus n'alimente pas le combo : addition directe.
    score += bonus;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  void _terminerLAventure() {
    transitionEnCours = false;
    niveauMaxAtteint = Constantes.nombreNiveaux - 1;
    changerEtat(GameState.victoire);
  }
```

### La garde de ré-entrance

`if (transitionEnCours) return;` n'est pas décoratif. Sans elle :

```text
  frame N     : le joueur touche la porte -> terminerNiveau() -> fondu lancé
  frame N + 1 : le joueur est TOUJOURS dans la porte -> onCollision
                -> terminerNiveau() -> DEUXIÈME fondu lancé
  frame N + 2 : troisième fondu…
```

Trois fondus simultanés, trois chargements du même niveau, trois joueurs dans le monde. La porte a déjà son propre verrou `_franchie`, mais un jeu qui compte sur un seul verrou finit toujours par en manquer un. **Deux verrous indépendants, chacun chez son propriétaire.**

### Bonus de fin de niveau

| Bonus | Valeur | Condition |
| --- | --- | --- |
| `bonusFinNiveau` | 200 | toujours, à la sortie du niveau |
| `bonusSalleVidee` | 300 | toutes les pièces du niveau ramassées |

Ils sont ajoutés **sans passer par `ajouterScore`** : ce ne sont pas des ramassages, ils ne doivent ni nourrir le combo ni relancer sa fenêtre. C'est une distinction que le chapitre 38 avait préparée en isolant `ajouterScore`.

### L'enchaînement

```dart
  Future<void> _enchainerVers(int suivant) async {
    final VoileTransition voile = VoileTransition();
    await camera.viewport.add(voile);

    // 1. Écran au noir.
    await voile.fermer();

    // 2. Pendant que l'écran est noir : on remplace tout le monde.
    await chargerNiveau(suivant);

    // 3. On annonce le niveau.
    final DefinitionNiveau def = NiveauxData.definition(suivant);
    await voile.annoncer('NIVEAU ${suivant + 1}', def.nom, def.soustitre);

    // 4. Écran de nouveau visible.
    await voile.ouvrir();
    voile.removeFromParent();

    transitionEnCours = false;
  }
```

Six lignes utiles, et une chronologie qu'on peut lire à voix haute. C'est le bénéfice d'avoir mis chaque responsabilité à sa place : le voile sait fondre, `chargerNiveau` sait charger, `_enchainerVers` sait seulement dans quel ordre.

```text
   t=0.00 s   la porte est franchie
   t=0.00 s   VoileTransition ajoutée au viewport, opacité 0
   t=0.45 s   opacité 1 : écran noir
   t=0.45 s   viderLeMonde() + Niveau.numero(1).construireDans(monde)
   t=0.50 s   panneau « NIVEAU 2 — Les oubliettes »
   t=1.70 s   panneau effacé
   t=2.15 s   opacité 0 : le niveau 2 apparaît
   t=2.15 s   voile retirée, transitionEnCours = false
```

### Pourquoi le moteur continue de tourner

On aurait pu appeler `pauseEngine()` pendant la transition. **Surtout pas** :

- un moteur en pause ne joue plus les effets — donc plus de fondu ;
- un moteur en pause ne traite plus la file de cycle de vie — donc `await monde.addAll(...)` ne se termine jamais (section 39.12).

Le prix à payer est que le joueur garde le contrôle pendant les 0,45 seconde du fondu. C'est imperceptible, et c'est même agréable : le personnage finit son mouvement au lieu de se figer net.

---

## 39.18 — L'écran de transition entre deux niveaux

### Où le placer : viewport, monde, ou overlay ?

Trois emplacements possibles, trois conséquences.

| Emplacement | Effet | Verdict |
| --- | --- | --- |
| dans le **monde** | le voile subit le zoom et le défilement : il ne couvre qu'un morceau d'écran | à exclure |
| en **overlay Flutter** | un widget par-dessus le canvas ; puissant, mais on perd `OpacityEffect` et l'on doit synchroniser deux boucles d'animation | possible, plus lourd |
| dans le **viewport** | coordonnées écran, priorité élevée, effets Flame disponibles | **retenu** |

C'est exactement le raisonnement du chapitre 38.14 pour le HUD, avec une exigence supplémentaire : le voile doit couvrir **le HUD lui-même**. Il suffit de lui donner une priorité supérieure (`Hud` est à 10, le voile sera à 1000).

### Le composant

Le code intégral figure en section 39.34. Voici sa structure et les deux points délicats.

```dart
// lib/hud/hud.dart — AJOUT DU CHAPITRE 39

/// Le rideau noir des transitions, plus le panneau d'annonce.
/// Vit dans le viewport : coordonnées écran, insensible au zoom.
class VoileTransition extends PositionComponent
    with HasGameReference<DonjonGame> {
  VoileTransition() : super(priority: 1000, anchor: Anchor.topLeft);

  late final RectangleComponent _rideau;      // le noir, opacité animée
  late final TextComponent _titre;            // « NIVEAU 2 »
  late final TextComponent _nom;              // « Les oubliettes »
  late final TextComponent _soustitre;        // la consigne

  @override
  Future<void> onLoad() async {
    size = game.size.clone();                 // POINT 1
    _rideau = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.voile,
    )..opacity = 0;
    // ... création des trois TextComponent, puis :
    await addAll(<Component>[_rideau, _titre, _nom, _soustitre]);
    _recentrer();
  }

  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    size = taille.clone();
    if (isLoaded) {                           // POINT 2
      _rideau.size = taille.clone();
      _recentrer();
    }
  }

  void _recentrer() {
    _titre.position = Vector2(size.x / 2, size.y / 2 - 42);
    _nom.position = Vector2(size.x / 2, size.y / 2 - 4);
    _soustitre.position = Vector2(size.x / 2, size.y / 2 + 32);
  }
}
```

Deux points méritent qu'on s'y arrête.

**`size = game.size.clone()`.** Le voile doit couvrir tout l'écran. Avec le `MaxViewport` par défaut (chapitre 31), la taille du viewport est celle du jeu ; on lit donc `game.size`. Le `clone()` est obligatoire : `game.size` est un `Vector2` vivant que le moteur met à jour au redimensionnement. Sans copie, le rideau et le jeu partageraient le même objet, et une écriture sur l'un modifierait l'autre.

**`onGameResize`.** Redimensionner la fenêtre pendant une transition doit redimensionner le rideau. Le garde `if (isLoaded)` est indispensable : `onGameResize` est appelée **avant** `onLoad` la première fois (chapitre 28), et `_rideau` n'existe pas encore.

### L'annonce

```dart
  /// Affiche le panneau, attend, l'efface. À appeler écran noir.
  Future<void> annoncer(String titre, String nom, String soustitre) async {
    _titre.text = titre;
    _nom.text = nom;
    _soustitre.text = soustitre;
    _recentrer();

    _nom.add(
      ScaleEffect.by(
        Vector2.all(1.08),
        EffectController(duration: 0.35, alternate: true),
      ),
    );

    await Future<void>.delayed(
      Duration(
        milliseconds: (Constantes.dureePanneauTransition * 1000).round(),
      ),
    );

    _titre.text = '';
    _nom.text = '';
    _soustitre.text = '';
  }
```

Notez le `_recentrer()` **après** l'affectation des textes. `TextComponent` recalcule sa `size` à chaque écriture de `text` (chapitre 38.21) : recentrer avant l'écriture centrerait un texte vide.

**Résultat à l'écran :**

```text
  ┌──────────────────────────────────────────┐
  │                                          │
  │              N I V E A U   2             │   <- _titre, doré, espacé
  │                                          │
  │           Les oubliettes                 │   <- _nom, grand, blanc
  │                                          │
  │   Descendez. La clé est tout en bas.     │   <- _soustitre, gris
  │                                          │
  └──────────────────────────────────────────┘
```

---

## 39.19 — Le fondu au noir avec un effet (rappel chapitre 33)

### Le principe

Un fondu au noir, c'est un rectangle noir qui couvre l'écran et dont on anime l'opacité de 0 à 1. Rien de plus. Le chapitre 33 a fourni l'outil :

```dart
OpacityEffect.to(1.0, EffectController(duration: 0.45, curve: Curves.easeIn));
```

`RectangleComponent` porte le mixin `HasPaint` : il possède donc `opacity`, et `OpacityEffect` fonctionne dessus sans configuration.

### Le problème : savoir quand c'est fini

`add(effet)` ne renvoie rien d'utile. Mais `_enchainerVers` a besoin d'attendre la fin du fondu avant de vider le monde, sinon le joueur voit le niveau disparaître.

La solution est un `Completer` (chapitre 15) : un objet qui fabrique un `Future` que **vous** décidez de compléter.

```text
   Completer<void> fini = Completer<void>();

   ┌──────────────┐  onComplete: fini.complete   ┌──────────────┐
   │ OpacityEffect│ ───────────────────────────> │  fini.future │
   └──────────────┘                              └──────────────┘
        Flame                                    await côté appelant
```

### Le code

```dart
  /// Fait passer le rideau à l'opacité [cible] et attend la fin.
  Future<void> _fondre(double cible, Curve courbe) {
    final Completer<void> fini = Completer<void>();

    _rideau.add(
      OpacityEffect.to(
        cible,
        EffectController(
          duration: Constantes.dureeFonduTransition,
          curve: courbe,
        ),
        onComplete: fini.complete,
      ),
    );

    return fini.future;
  }

  /// Écran au noir.
  Future<void> fermer() => _fondre(1.0, Curves.easeIn);

  /// Écran de nouveau visible.
  Future<void> ouvrir() => _fondre(0.0, Curves.easeOut);
```

`onComplete` attend un `void Function()?`. `fini.complete` a la signature `void complete([FutureOr<void>? value])` : une fonction dont **tous** les paramètres sont optionnels est bien assignable à `void Function()`. On peut donc écrire `onComplete: fini.complete` sans lambda.

### Le choix des courbes

Le chapitre 33.14 donnait la règle :

| Moment | Courbe | Raison |
| --- | --- | --- |
| fermer (l'écran s'assombrit) | `Curves.easeIn` | démarre doucement, finit vite : on « tombe » dans le noir |
| ouvrir (l'écran réapparaît) | `Curves.easeOut` | démarre vite, finit doucement : le niveau se pose |

Inversez les deux et le fondu paraît mou à la fermeture et brutal à l'ouverture. C'est subtil, mais c'est ce qui distingue un fondu fait à la main d'un fondu qui « respire ».

### Le piège de l'opacité qui reste coincée

Le chapitre 33 le signalait déjà : **l'opacité appartient au composant, pas à l'effet**. Si le fondu est interrompu — par exemple parce que le joueur relance une partie depuis le menu au milieu d'une transition —, le rideau reste à mi-noir pour toujours.

Deux protections, toutes deux appliquées ici :

1. La voile est **créée à chaque transition et retirée à la fin** : chaque transition part d'une opacité 0 garantie.
2. `terminerNiveau` refuse de lancer une deuxième transition tant que la première n'est pas finie.

### Variante : la fermeture en iris

Pour le plaisir, et parce que le chapitre 31 a présenté les viewports : Flame fournit un `CircularViewport`. En animer le rayon donne une fermeture « en iris » comme dans les dessins animés. C'est un excellent exercice une fois le chapitre terminé — mais un rideau rectangulaire reste le choix le plus sûr, parce qu'il est indépendant du viewport et donc du reste de l'architecture.

---

## 39.20 — `lib/composants/boss.dart` : le boss du niveau 3

### Un boss est un ennemi, pas un cas particulier

La tentation est d'écrire le boss de zéro. Ce serait une faute : il a des PV, il subit des dégâts, il inflige des dégâts au contact, il meurt en donnant du score, il tombe. Tout cela est déjà écrit dans `Ennemi` (chapitre 37).

```text
   PositionComponent
        │
        ├── mixin Sante              pv, pvMax, subirDegats, soigner   (ch. 37)
        ├── mixin CollisionCallbacks
        ├── mixin HasGameReference
        │
        └── Ennemi (abstraite)       gravité, knockback, raycast,      (ch. 37)
             │                       mort, particules, contact
             ├── Gobelin             IA à 5 états
             ├── Chauvesouris        IA à 5 états, vol
             └── Boss                IA à 6 PHASES                     (ch. 39)
```

Le boss n'hérite donc **que** d'une chose nouvelle : sa manière de décider. C'est précisément le rôle de la méthode abstraite `mettreAJourIA(double dt)` déclarée au chapitre 37.

### Ce que le boss redéfinit, et ce qu'il garde

| Membre | Origine | Boss |
| --- | --- | --- |
| gravité, collision avec le décor | `Ennemi.update` | **gardé tel quel** |
| dégâts de contact au joueur | `Ennemi.onCollision` | **gardé tel quel** |
| flash blanc quand il encaisse | `Ennemi.onDegatsRecus` | **gardé tel quel** |
| particules et score à la mort | `Ennemi.mourir` | **gardé**, avec un appel en plus |
| `mettreAJourIA(dt)` | abstraite | **implémentée** : machine à phases |
| `subirDegats(double)` | `Sante` | **redéfinie** : armure et vulnérabilité |
| `etat` (`EtatEnnemi`) | `Ennemi` | inutilisé sauf pour `mort` |

Ce dernier point mérite une explication franche : le boss possède `etat` par héritage, mais sa vraie machine est `phase`. On ne s'en sert plus que pour signaler la mort à `Ennemi.update`, qui court-circuite l'IA quand `etat == EtatEnnemi.mort`. Deux machines dans un même objet seraient une faute de conception ; ici la première est **réduite à un drapeau**, ce qui est acceptable et documenté.

### La déclaration

```dart
// lib/composants/boss.dart
enum PhaseBoss { sommeil, charge, eventail, invocation, etourdi, mort }

class Boss extends Ennemi {
  Boss({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(48, 56),
          vitesse: Constantes.vitesseBoss,
          degatsContact: Constantes.degatsContactBoss,
          pointsScore: Constantes.scoreBoss,
          couleur: Palette.boss,
          pvDepart: Constantes.pvBoss,
        );

  PhaseBoss phase = PhaseBoss.sommeil;

  bool get enrage => ratioPv <= Constantes.seuilEnrageBoss;
  bool get vulnerable => phase == PhaseBoss.etourdi;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = Constantes.rayonReveilBoss;
    rayonAbandon = double.infinity;   // un boss n'abandonne jamais
    demiPatrouille = 0;              // il ne patrouille pas
  }

  @override
  void onMount() {
    super.onMount();
    game.boss = this;
  }

  @override
  void onRemove() {
    if (game.boss == this) {
      game.boss = null;
    }
    super.onRemove();
  }
}
```

`ratioPv` vient du mixin `Sante` du chapitre 37 : `pv / pvMax`, borné entre 0 et 1. Il servira trois fois : la rage, la barre de vie et le réglage de difficulté.

---

## 39.21 — Les phases d'un boss : la machine à états

### Le schéma

```text
                    joueur à moins de 220 px
        ┌─ SOMMEIL ────────────────────────┐
        │                                   v
        │                            ┌──> CHARGE ──┐
        │                            │      │       │ mur touché
        │                            │      │ temps │
        │                            │      v       v
        │                            │  EVENTAIL  ETOURDI
        │                            │      │       │
        │                            │      │       │ 2,4 s
        │                            │      v       │
        │                            │   CHARGE <───┘
        │                            │      │
        │                            │      v
        │                            └── INVOCATION
        │
        └─ (à tout moment) pv <= 0 ──> MORT
```

Le cycle nominal est une **liste**, ce qui rend l'équilibrage trivial : pour changer le rythme du combat, on change la liste, pas le code.

```dart
  static const List<PhaseBoss> cycle = <PhaseBoss>[
    PhaseBoss.charge,
    PhaseBoss.eventail,
    PhaseBoss.charge,
    PhaseBoss.invocation,
  ];

  int _indiceCycle = 0;

  void _phaseSuivante() {
    _entrerDans(cycle[_indiceCycle % cycle.length]);
    _indiceCycle++;
  }
```

`ETOURDI` n'est pas dans la liste : il s'insère **hors cycle** quand le boss se cogne, puis rend la main à `_phaseSuivante()`. C'est un état d'exception, et sa place dans le code le dit.

### Entrée, mise à jour, sortie

Toute machine à états propre distingue trois moments (chapitre 26) :

| Moment | Ce qui s'y passe | Où |
| --- | --- | --- |
| **entrée** | armer les minuteries, viser, changer de couleur | `_entrerDans(phase)` |
| **mise à jour** | agir, image par image | `mettreAJourIA(dt)` |
| **sortie** | décider de la phase suivante | fin de chaque `_faireX()` |

```dart
  double _minuterie = 0;      // temps restant dans la phase
  double _cadence = 0;        // temps avant la prochaine action de la phase
  int _sbiresInvoques = 0;

  void _entrerDans(PhaseBoss nouvelle) {
    phase = nouvelle;
    velocite.setZero();
    _sbiresInvoques = 0;
    _cadence = 0;

    if (nouvelle == PhaseBoss.charge) {
      _minuterie = Constantes.dureeChargeBoss / facteurEnrage;
      _viser();
    } else if (nouvelle == PhaseBoss.eventail) {
      _minuterie = Constantes.dureeEventailBoss;
    } else if (nouvelle == PhaseBoss.invocation) {
      _minuterie = Constantes.dureeInvocationBoss;
    } else if (nouvelle == PhaseBoss.etourdi) {
      _minuterie = Constantes.dureeEtourdiBoss;
    } else {
      _minuterie = 0;
    }

    _teindreCorps();     // doré si étourdi, rouge vif si enragé, sinon sombre
  }

  double get facteurEnrage => enrage ? 1.4 : 1.0;
```

`_teindreCorps()` est appelée à chaque entrée de phase : le joueur **voit** l'état interne du boss. Une machine à états invisible est une machine à états injouable.

### Le répartiteur

```dart
  @override
  void mettreAJourIA(double dt) {
    if (_cadence > 0) {
      _cadence -= dt;
    }
    _minuterie -= dt;

    switch (phase) {
      case PhaseBoss.sommeil:
        _dormir();
        break;
      case PhaseBoss.charge:
        _charger();
        break;
      case PhaseBoss.eventail:
        _tirerEnEventail();
        break;
      case PhaseBoss.invocation:
        _invoquer();
        break;
      case PhaseBoss.etourdi:
        _rester();
        break;
      case PhaseBoss.mort:
        velocite.setZero();
        break;
    }
  }

  void _dormir() {
    velocite.x = 0;
    if (distanceAuJoueur <= Constantes.rayonReveilBoss) {
      _rugir();
      _phaseSuivante();
    }
  }

  void _rester() {
    velocite.x = 0;
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }
```

Le `switch` sur un `enum` sans `default` est un allié : si vous ajoutez une septième phase, l'analyseur signale immédiatement que le `switch` ne la traite pas. C'est l'argument du chapitre 11 en faveur des enums.

---

## 39.22 — Phase 1 : charge

### L'idée

Le boss vise le joueur, puis fonce en ligne droite à 2,6 fois sa vitesse de base. S'il rencontre un mur, il s'assomme — et devient vulnérable. C'est le mécanisme de boss le plus ancien du monde, et le plus lisible : le joueur comprend en une seconde qu'il faut esquiver au dernier moment.

```text
   1. Le boss vise            2. Il fonce             3. Le joueur esquive
      ▓          @               ▓───────>@              @   ▓───────>│MUR
                                                                       │
                                                       4. ÉTOURDI : frappez !
```

### Le code

```dart
  void _viser() {
    final Joueur? heros = cible;
    if (heros == null) {
      return;
    }
    sens = heros.absoluteCenter.x >= absoluteCenter.x ? 1 : -1;
  }

  void _charger() {
    velocite.x =
        sens * vitesse * Constantes.facteurChargeBoss * facteurEnrage;

    if (murDevant()) {
      _entrerDans(PhaseBoss.etourdi);
      _impact();
      return;
    }
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  void _impact() {
    game.monde.add(
      ParticleSystemComponent(
        position: absoluteCenter.clone(),
        particle: _poussiere(),
      ),
    );
    game.afficherTexteFlottant(
      absoluteCenter,
      'Étourdi !',
      Palette.bossEtourdi,
      taille: 12,
    );
    corps.add(
      ScaleEffect.by(
        Vector2(1.25, 0.78),
        EffectController(duration: 0.1, alternate: true),
      ),
    );
  }
```

### Trois décisions de conception

**1. Le boss vise à l'entrée de la phase, pas à chaque frame.** Une charge qui se corrige en cours de route est impossible à esquiver. Une charge qui s'engage une fois est **lisible** : le joueur apprend à attendre le dernier moment. C'est la différence entre un boss difficile et un boss injuste.

**2. `murDevant()` a été corrigé en 39.8** pour ignorer les plateformes traversables. Sans cette correction, le boss du niveau 3 s'assommerait sur les perchoirs de la ligne 7. Un bug d'un seul caractère de filtre, invisible en lisant le code du boss.

**3. La charge a une durée maximale.** `_minuterie` sort de la phase même si aucun mur n'est touché. Sinon, un boss qui charge dans une salle sans mur (ou dont le rayon rate le mur d'un pixel) chargerait pour l'éternité. **Toute phase d'un boss doit avoir une sortie par le temps**, en plus de sa sortie logique.

---

## 39.23 — Phase 2 : tir en éventail

### La géométrie

Un éventail de `n` projectiles, d'ouverture totale `ouverture` radians, centré sur la direction du joueur.

```text
                       ouverture = 1,1 rad (63 degrés)
                    ╲   ╲   │   ╱   ╱
                     ╲   ╲  │  ╱   ╱
                      ╲   ╲ │ ╱   ╱          n = 5 projectiles
                       ╲   ╲│╱   ╱
                        ▓ BOSS ▓
                             │
                             v  angle de base = atan2(dy, dx)
                             @  joueur
```

`atan2(dy, dx)` donne l'angle de la direction joueur-boss, en radians, dans le bon quadrant — c'est tout l'intérêt d'`atan2` par rapport à `atan` (chapitre 23).

### Le code

```dart
  void _tirerEnEventail() {
    velocite.x = 0;
    if (_cadence <= 0) {
      _salve();
      _cadence = enrage ? 0.45 : 0.70;
    }
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  void _salve() {
    final Joueur? heros = cible;
    final Vector2 vers = heros == null
        ? Vector2(sens.toDouble(), 0)
        : heros.absoluteCenter - absoluteCenter;
    if (vers.length2 < 1) {
      return;
    }

    final double base = atan2(vers.y, vers.x);
    final int nombre =
        Constantes.projectilesEventailBoss + (enrage ? 2 : 0);
    const double ouverture = Constantes.ouvertureEventailBoss;

    for (int i = 0; i < nombre; i++) {
      // fraction de 0 à 1 sur toute l'ouverture
      final double fraction = nombre == 1 ? 0.5 : i / (nombre - 1);
      final double angle = base - ouverture / 2 + ouverture * fraction;

      game.monde.add(
        Projectile(
          position: absoluteCenter.clone(),
          direction: Vector2(cos(angle), sin(angle)),
          vitesse: Constantes.vitesseProjectile * 1.15,
        ),
      );
    }

    corps.add(
      ScaleEffect.by(
        Vector2(0.85, 1.2),
        EffectController(duration: 0.09, alternate: true),
      ),
    );
  }
```

### Le détail qui compte : `nombre - 1`

```text
  n = 5, ouverture = 1,1 rad, base = 0

  i = 0 -> fraction 0,00 -> angle = -0,550
  i = 1 -> fraction 0,25 -> angle = -0,275
  i = 2 -> fraction 0,50 -> angle =  0,000   <- pile sur le joueur
  i = 3 -> fraction 0,75 -> angle = +0,275
  i = 4 -> fraction 1,00 -> angle = +0,550
```

En divisant par `nombre` au lieu de `nombre - 1`, le dernier projectile s'arrêterait à 0,44 rad et l'éventail serait décentré vers la gauche. La garde `nombre == 1` évite la division par zéro dans le cas dégénéré — le genre de cas qui n'arrive jamais, sauf le jour où vous réglez `projectilesEventailBoss` à 1 pour tester.

### La réutilisation de `Projectile`

Aucune nouvelle classe. Le `Projectile` du chapitre 37 fait déjà tout :

- il avance en ligne droite le long de sa `direction` normalisée ;
- il blesse le joueur et disparaît ;
- il disparaît au contact d'une `Plateforme` ;
- il s'auto-détruit au bout de `dureeVieProjectile` secondes.

Il ne blesse pas les ennemis, donc le boss ne se tire pas dessus, et les projectiles nés au centre du boss le traversent sans effet. C'est le bénéfice d'un composant écrit proprement deux chapitres plus tôt.

---

## 39.24 — Phase 3 : invocation de gobelins

### L'idée et le danger

Le boss appelle des renforts. Le danger est évident : un boss qui invoque sans limite noie le joueur sous vingt gobelins en dix secondes, le framerate s'effondre, et le combat devient une loterie.

**Deux plafonds, pas un :**

1. un plafond par phase : `_sbiresInvoques` ne dépasse pas les places disponibles ;
2. un plafond global : au plus `sbiresMaxBoss` gobelins **vivants** en même temps.

### Le code

```dart
  void _invoquer() {
    velocite.x = 0;

    if (_cadence <= 0 && _placesDisponibles() > 0) {
      _invoquerUnGobelin();
      _sbiresInvoques++;
      _cadence = 0.4;
    }

    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  /// Combien de gobelins peut-on encore ajouter sans dépasser le plafond ?
  int _placesDisponibles() {
    final int vivants = game.monde.children
        .whereType<Gobelin>()
        .where((Gobelin g) => g.estVivant)
        .length;
    return Constantes.sbiresMaxBoss - vivants;
  }

  void _invoquerUnGobelin() {
    // Un sbire à gauche, un à droite, de plus en plus loin.
    final double ecart =
        Constantes.tailleTuile * (2 + _sbiresInvoques);
    final double decalage = _sbiresInvoques.isEven ? -ecart : ecart;

    final Vector2 point = Vector2(
      absoluteCenter.x + decalage - 12,
      absoluteCenter.y - Constantes.tailleTuile,
    );

    game.monde.add(Gobelin(position: point));
    game.monde.add(
      ParticleSystemComponent(
        position: point.clone()..add(Vector2(12, 15)),
        particle: _poussiere(),
      ),
    );
  }
```

### Le comptage par `whereType`

```dart
    game.monde.children.whereType<Gobelin>().where((g) => g.estVivant).length
```

C'est le chapitre 14 dans le monde du jeu : `whereType<T>()` filtre **et** type en une opération. Sans lui, il faudrait un `if (c is Gobelin)` et un transtypage manuel.

Le filtre `estVivant` n'est pas cosmétique : `Ennemi.mourir()` lance une animation de rétrécissement de 0,24 s avant de retirer le composant. Pendant ce quart de seconde, un gobelin mort est toujours dans `children`. Sans le filtre, le boss croirait la salle pleine alors qu'elle se vide.

### Où faire apparaître les sbires

Le calcul place les gobelins alternativement à gauche et à droite du boss, à 2, 3 puis 4 tuiles, **une tuile au-dessus du sol** pour qu'ils tombent en place. Ce petit saut a deux vertus : il est lisible à l'écran, et il évite qu'un gobelin apparaisse coincé dans le décor.

> **Attention.** Le calcul ne vérifie pas que la position d'apparition est libre. Sur une carte plus étroite, un sbire pourrait naître dans un mur. La solution propre — lire la carte du niveau courant pour choisir une tuile vide — est l'exercice 8.

### Les particules de poussière

```dart
  Particle _poussiere() {
    final Random hasard = Random();
    return Particle.generate(
      count: 12,
      lifespan: 0.45,
      generator: (int i) => AcceleratedParticle(
        speed: Vector2(
          hasard.nextDouble() * 140 - 70,
          hasard.nextDouble() * -90 - 20,
        ),
        acceleration: Vector2(0, 340),
        child: CircleParticle(
          radius: 1.5 + hasard.nextDouble() * 2.5,
          paint: Paint()..color = Palette.bossEtourdi,
        ),
      ),
    );
  }
```

Même recette qu'au chapitre 37 pour la gerbe de mort : `AcceleratedParticle` pour la gravité, `CircleParticle` pour le grain, `Particle.generate` pour la variété.

---

## 39.25 — La barre de vie du boss dans le HUD

### Où elle vit

Dans le **viewport**, comme tout le HUD, mais pas dans un `HudMarginComponent` : on la veut centrée horizontalement, et le placement par marges du chapitre 38 ancre à gauche ou à droite, jamais au centre. On la positionne donc à la main dans `onGameResize`.

```dart
// lib/hud/barre_de_vie.dart — AJOUT DU CHAPITRE 39

/// Barre de vie du boss, centrée en bas de l'écran.
/// Invisible tant qu'aucun boss n'est éveillé.
class BarreBoss extends PositionComponent with HasGameReference<DonjonGame> {
  BarreBoss()
      : super(size: Vector2(largeur, hauteur), anchor: Anchor.topCenter);

  static const double largeur = 320;
  static const double hauteur = 12;
  static const double bordure = 2;
  static const double margeBas = 52;

  static double get _utile => largeur - bordure * 2;

  late final RectangleComponent _cadre;
  late final RectangleComponent _creux;
  late final RectangleComponent _remplissage;
  late final TextComponent _titre;

  double _ratioAffiche = 1;

  @override
  Future<void> onLoad() async {
    _cadre = RectangleComponent(
      size: Vector2(largeur, hauteur),
      paint: Paint()..color = const Color(0xFF15151E),
    );
    _creux = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = const Color(0xFF3A1E24),
    );
    _remplissage = RectangleComponent(
      position: Vector2.all(bordure),
      size: Vector2(_utile, hauteur - bordure * 2),
      paint: Paint()..color = Palette.boss,
    );
    _titre = TextComponent(
      text: 'LE GARDIEN',
      anchor: Anchor.bottomCenter,
      position: Vector2(largeur / 2, -3),
      textRenderer: TextPaint(
        style: const TextStyle(
          fontSize: 12,
          color: Palette.texte,
          fontWeight: FontWeight.bold,
          letterSpacing: 2,
          shadows: <Shadow>[
            Shadow(color: Color(0xCC000000), offset: Offset(1, 1)),
          ],
        ),
      ),
    );

    await addAll(<Component>[_cadre, _creux, _remplissage, _titre]);
  }

  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    position = Vector2(taille.x / 2, taille.y - margeBas);
  }

  @override
  void update(double dt) {
    super.update(dt);

    final Boss? gardien = game.boss;
    final bool visible = gardien != null &&
        gardien.estVivant &&
        gardien.phase != PhaseBoss.sommeil;

    // Astuce du chapitre 38 : on cache par l'échelle, pas par le retrait.
    scale = visible ? Vector2.all(1) : Vector2.all(0.001);
    if (!visible || gardien == null) {
      return;
    }

    // Rattrapage progressif : la barre glisse, elle ne saute pas.
    final double cible = gardien.ratioPv;
    _ratioAffiche += (cible - _ratioAffiche) * min(1.0, 6 * dt);
    if ((cible - _ratioAffiche).abs() < 0.002) {
      _ratioAffiche = cible;
    }

    final double l = (_utile * _ratioAffiche).clamp(0.01, _utile);
    if ((_remplissage.size.x - l).abs() > 0.05) {
      _remplissage.size.setValues(l, _remplissage.size.y);
    }

    _remplissage.paint.color =
        gardien.vulnerable ? Palette.bossEtourdi : Palette.boss;

    final String voulu =
        gardien.vulnerable ? 'LE GARDIEN — ÉTOURDI' : 'LE GARDIEN';
    if (_titre.text != voulu) {
      _titre.text = voulu;
    }
  }
}
```

### Les quatre règles reprises du chapitre 38

| Règle | Application ici |
| --- | --- |
| ne jamais laisser une largeur exactement nulle | `clamp(0.01, _utile)` |
| n'écrire `text` que si la valeur change | `if (_titre.text != voulu)` |
| borner le facteur d'interpolation | `min(1.0, 6 * dt)` |
| cacher par l'échelle, pas par le retrait | `scale = Vector2.all(0.001)` |

Cette dernière mérite un mot. Retirer et rajouter le composant à chaque changement d'état déclencherait `onLoad`, réinitialiserait `_ratioAffiche`, et ferait clignoter la barre. Une échelle quasi nulle rend le composant invisible sans toucher à son cycle de vie. On n'utilise pas exactement `0` parce qu'une matrice d'échelle nulle n'est pas inversible (piège signalé au chapitre 38).

### Le branchement dans le HUD

```dart
// lib/hud/hud.dart — MODIFIÉ AU CHAPITRE 39
  late final BarreBoss barreBoss;

  @override
  Future<void> onLoad() async {
    // ... les six éléments du chapitre 38
    barreBoss = BarreBoss();

    await addAll(<Component>[
      barreDeVie, barreEnergie, compteurVies,
      compteurScore, compteurCles, indicateurObjectif,
      barreBoss,                                  // AJOUT DU CHAPITRE 39
    ]);
  }
```

---

## 39.26 — Les points faibles et les fenêtres d'attaque

### Deux façons de rendre un boss vulnérable

| Approche | Principe | Coût |
| --- | --- | --- |
| **spatiale** | une zone du corps encaisse plus (la tête, le cœur, le dos) | une hitbox supplémentaire, et une refonte de la détection : `other is Ennemi` ne suffit plus |
| **temporelle** | le boss est vulnérable **pendant** certaines phases | un booléen |

Nous retenons la **fenêtre temporelle**, pour trois raisons.

1. **Lisibilité.** Sans sprite, un point faible géométrique est invisible : le joueur ne peut pas deviner qu'il faut frapper le coin supérieur gauche d'un rectangle rouge. Une phase `ETOURDI` qui change la couleur du boss **et** le titre de sa barre de vie se comprend instantanément.
2. **Compatibilité.** Tout le code de combat du chapitre 37 s'appuie sur `other is Ennemi` dans `ZoneAttaque.onCollisionStart`. Ajouter une seconde hitbox sur le boss demanderait de distinguer les hitbox entre elles, donc de changer la signature de toute la chaîne de dégâts.
3. **Rythme.** Une fenêtre temporelle impose un **cycle** : esquiver, attendre l'ouverture, frapper, reculer. C'est ce cycle qui fait un combat de boss, pas la géométrie.

### Le code de l'armure

```dart
  @override
  void subirDegats(double degats) {
    if (phase == PhaseBoss.sommeil || phase == PhaseBoss.mort) {
      return;                       // on ne frappe pas un boss endormi
    }

    final double facteur = vulnerable
        ? Constantes.bonusDegatsEtourdi     // 2,0 : fenêtre ouverte
        : Constantes.reductionDegatsBoss;   // 0,5 : armure

    final bool etaitEnrage = enrage;
    super.subirDegats(degats * facteur);

    // La rage se déclenche UNE FOIS, au passage du seuil.
    if (!etaitEnrage && enrage && estVivant) {
      _entrerEnRage();
    }
  }

  void _entrerEnRage() {
    game.afficherTexteFlottant(
      absoluteCenter,
      'ENRAGÉ',
      Palette.bossEnrage,
      taille: 13,
    );
    _entrerDans(PhaseBoss.charge);   // la rage interrompt la phase en cours
  }
```

### Le calcul du combat

L'attaque du joueur inflige `degatsAttaqueJoueur = 25` (chapitre 37), avec un cooldown de 0,45 s.

```text
  Boss : 320 PV

  HORS fenêtre (armure 0,5)          25 x 0,5 = 12,5 PV par coup
                                     -> 26 coups, soit ~12 secondes de martelage

  DANS la fenêtre (bonus 2,0)        25 x 2,0 = 50 PV par coup
                                     -> 7 coups seulement

  Une fenêtre ETOURDI dure 2,4 s, soit 5 coups possibles (cooldown 0,45 s)
  -> environ 2 fenêtres bien exploitées suffisent à tuer le boss.
```

Le message envoyé au joueur est donc parfaitement clair : **taper au hasard prend une éternité, provoquer la charge et frapper l'étourdissement prend vingt secondes**. C'est l'équilibrage qui enseigne le mécanisme, sans une ligne de tutoriel.

### La lisibilité de la fenêtre

Trois signaux simultanés annoncent la fenêtre, parce qu'un seul serait raté par la moitié des joueurs :

| Signal | Où | Section |
| --- | --- | --- |
| le corps du boss vire au doré | dans le monde | `_teindreCorps` (39.21) |
| le texte « Étourdi ! » monte au-dessus de lui | dans le monde | `_impact` (39.22) |
| la barre de vie change de couleur et de titre | dans le HUD | `BarreBoss.update` (39.25) |

---

## 39.27 — La mort du boss et la victoire

### La chaîne complète

```text
  ZoneAttaque touche le boss
        │
        v
  Boss.subirDegats(25 x 2,0)
        │
        v
  Sante.subirDegats -> pv <= 0 -> onMort()
        │
        v
  Ennemi.onMort() => mourir()   ... redéfinie par Boss
        │
        ├──> game.bossVaincu(this)  -> déverrouille la porte, bonus de score
        │
        └──> super.mourir()         -> score, particules, rétrécissement, retrait
```

### Côté boss

```dart
  @override
  void mourir() {
    if (phase == PhaseBoss.mort) {
      return;
    }
    phase = PhaseBoss.mort;
    _minuterie = 0;

    game.bossVaincu(this);

    // Une gerbe plus généreuse que celle d'un gobelin.
    for (int i = 0; i < 3; i++) {
      game.monde.add(
        ParticleSystemComponent(
          position: absoluteCenter.clone(),
          priority: 30,
          particle: _poussiere(),
        ),
      );
    }

    super.mourir();     // score, effet de disparition, removeFromParent
  }
```

L'ordre est important : `game.bossVaincu(this)` **avant** `super.mourir()`, parce que `super.mourir()` déclenche l'effet qui finira par retirer le composant, et donc par déclencher `onRemove()` qui met `game.boss` à `null`.

### Côté jeu

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 39
  /// Appelée par le boss au moment de sa mort.
  void bossVaincu(Boss vaincu) {
    porte?.deverrouillerParLeBoss();

    score += Constantes.bonusBoss;
    if (score > meilleurScore) {
      meilleurScore = score;
    }

    afficherTexteFlottant(
      vaincu.absoluteCenter,
      'LE GARDIEN TOMBE',
      Palette.accent,
      taille: 14,
    );
  }
```

`porte?.deverrouillerParLeBoss()` — l'opérateur `?.` du chapitre 12. Si un jour vous écrivez une carte de boss sans porte, le jeu ne plantera pas ; il sera simplement impossible d'en sortir, ce que le validateur de la section 39.7 interdit déjà.

### La victoire

Le joueur franchit la porte, `terminerNiveau()` constate que `niveauCourant + 1 >= Constantes.nombreNiveaux`, et bascule l'état :

```dart
  void _terminerLAventure() {
    transitionEnCours = false;
    niveauMaxAtteint = Constantes.nombreNiveaux - 1;
    changerEtat(GameState.victoire);
  }
```

`changerEtat(GameState.victoire)` — écrite au chapitre 35 — met le moteur en pause et affiche l'overlay `Overlays.victoire`. Il faut donc lui donner un widget dans `main.dart`, faute de quoi Flame cherchera un constructeur d'overlay inexistant. Le chapitre 40 fera un véritable écran ; en attendant, voici la version provisoire :

```dart
// lib/main.dart — AJOUT DU CHAPITRE 39
            Overlays.victoire: (context, game) => Container(
                  color: Palette.fond.withValues(alpha: 0.92),
                  child: Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: <Widget>[
                        const Text(
                          'VICTOIRE',
                          style: TextStyle(
                            color: Palette.accent,
                            fontSize: 44,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const SizedBox(height: 10),
                        Text(
                          'Score final : ${game.score}',
                          style: const TextStyle(
                            color: Palette.texte,
                            fontSize: 20,
                          ),
                        ),
                        const SizedBox(height: 20),
                        ElevatedButton(
                          onPressed: game.demarrerPartie,
                          child: const Text('Rejouer'),
                        ),
                      ],
                    ),
                  ),
                ),
```

**Résultat, la boucle de jeu complète :**

```text
   MENU  ──Jouer──>  NIVEAU 1  ──porte──>  NIVEAU 2  ──porte──>  NIVEAU 3
     ^                   │                     │                     │
     │                   │ 0 vie               │ 0 vie               │ boss mort
     │                   v                     v                     │ + porte
     └── Rejouer ──── GAME OVER ───────────────┘                     v
                                                                  VICTOIRE
```

---

## 39.28 — La courbe de difficulté : tableau niveau par niveau

### Ce qu'une courbe de difficulté doit faire

Elle doit **enseigner**, puis **tester**, puis **couronner**. Un niveau qui introduit trois mécanismes à la fois n'enseigne rien.

| | Niveau 1 — Les caves | Niveau 2 — Les oubliettes | Niveau 3 — La salle du gardien |
| --- | --- | --- | --- |
| Rôle | apprendre | tester | couronner |
| Dimensions | 40 × 14 | 48 × 18 | 40 × 14 |
| Sol | continu, aucun trou | une seule bande tout en bas | continu |
| Plateformes traversables | 3, décoratives | 8, **indispensables** | 4, pour esquiver |
| Gobelins | 2, espacés | 3, dont un sur perchoir | 0 au départ, jusqu'à 3 invoqués |
| Chauves-souris | 2 | 3 | 0 |
| Boss | — | — | 1 |
| PV d'ennemis à affronter | 90 | 135 | 320 (+ 90 de sbires) |
| Pièces | 10 | 23 | 6 |
| Potions | 1 | 2 | 2 |
| Objectif | trouver la clé | **descendre**, puis grimper à la clé | tuer le gardien |
| Danger de chute | nul | permanent | nul |
| Durée visée | 45 s | 2 min | 2 min |

### Les trois principes appliqués

**1. Un mécanisme nouveau par niveau.** Le niveau 1 n'enseigne que « courir, sauter, frapper, ramasser ». Le niveau 2 ajoute la verticalité et les plateformes traversables. Le niveau 3 n'ajoute rien de nouveau au joueur : il lui demande d'appliquer tout ce qu'il sait contre un adversaire à motif.

**2. La densité monte, la surface aussi.** Passer de 4 à 6 ennemis en doublant presque la surface ne rend pas le jeu plus dur — cela le rend plus long. La vraie montée du niveau 2 est le **risque de chute**, gratuit en termes de composants.

**3. Le boss récapitule.** Sa charge exige l'esquive du niveau 1. Son éventail exige les plateformes du niveau 2. Son invocation ramène les gobelins que le joueur sait déjà tuer. Un bon boss ne demande rien qu'on n'ait déjà appris.

### Les nombres à retoucher en premier

Quand un testeur vous dit « c'est trop dur », voici l'ordre dans lequel toucher les réglages :

| Rang | Constante | Effet ressenti |
| --- | --- | --- |
| 1 | `dureeInvincibilite` | change tout le confort du combat |
| 2 | `degatsContactGobelin` / `...Chauvesouris` | change la punition d'une erreur |
| 3 | `pvBoss` | change uniquement la **longueur** du combat de boss |
| 4 | `dureeEtourdiBoss` | change la générosité des fenêtres |
| 5 | nombre d'ennemis dans la carte | change la densité, mais coûte un rechargement |

Ne touchez **jamais** à `vitesseJoueur` ni à `forceSaut` pour équilibrer : ce sont les constantes qui définissent la sensation du jeu, et le joueur les a apprises dans les dix premières secondes.

---

## 39.29 — Les modes de difficulté (facile, normal, difficile)

### Ce qu'un mode de difficulté doit changer

Un mode qui ne modifie qu'un nombre de vies est un faux mode. Nous en modifions quatre choses, toutes multiplicatives sauf les vies.

```dart
// lib/config/constantes.dart — AJOUT DU CHAPITRE 39
enum Difficulte { facile, normal, difficile }

/// Les réglages associés à un mode de difficulté.
class ReglagesDifficulte {
  const ReglagesDifficulte({
    required this.libelle,
    required this.vies,
    required this.pvEnnemis,
    required this.degatsEnnemis,
    required this.bonusScore,
  });

  final String libelle;
  final int vies;
  final double pvEnnemis;
  final double degatsEnnemis;
  final double bonusScore;

  static const Map<Difficulte, ReglagesDifficulte> table =
      <Difficulte, ReglagesDifficulte>{
    Difficulte.facile: ReglagesDifficulte(
      libelle: 'Facile',
      vies: 5,
      pvEnnemis: 0.75,
      degatsEnnemis: 0.6,
      bonusScore: 0.75,
    ),
    Difficulte.normal: ReglagesDifficulte(
      libelle: 'Normal',
      vies: 3,
      pvEnnemis: 1.0,
      degatsEnnemis: 1.0,
      bonusScore: 1.0,
    ),
    Difficulte.difficile: ReglagesDifficulte(
      libelle: 'Difficile',
      vies: 1,
      pvEnnemis: 1.4,
      degatsEnnemis: 1.5,
      bonusScore: 1.5,
    ),
  };

  static ReglagesDifficulte de(Difficulte mode) => table[mode]!;
}
```

| Mode | Vies | PV ennemis | Dégâts subis | Score | Coups pour tuer un gobelin |
| --- | --- | --- | --- | --- | --- |
| Facile | 5 | ×0,75 | ×0,6 | ×0,75 | 1 |
| Normal | 3 | ×1,0 | ×1,0 | ×1,0 | 2 |
| Difficile | 1 | ×1,4 | ×1,5 | ×1,5 | 2 |

Le `bonusScore` est la contrepartie : jouer en facile est confortable **et** rapporte moins. Sans lui, le tableau des meilleurs scores du chapitre 40 n'aurait aucun sens.

### Les trois branchements

**1. Sur le jeu :**

```dart
// lib/donjon_game.dart
  Difficulte difficulte = Difficulte.normal;
  ReglagesDifficulte get reglages => ReglagesDifficulte.de(difficulte);
  int viesMax = Constantes.viesDepart;
```

**2. Sur les ennemis**, à leur chargement — deux lignes dans `Ennemi.onLoad` :

```dart
// lib/composants/ennemi.dart — MODIFIÉ AU CHAPITRE 39
  @override
  Future<void> onLoad() async {
    final ReglagesDifficulte reglages = game.reglages;
    initialiserSante(_pvDepart * reglages.pvEnnemis);   // MODIFIÉ
    degatsContact *= reglages.degatsEnnemis;            // AJOUT
    ancre = position.clone();
    // ... le reste du chapitre 37, inchangé
  }
```

C'est le bon endroit : `onLoad` s'exécute une fois par ennemi, après que `game` est disponible, et avant la première frame. Faire le calcul dans le constructeur serait impossible — `game` n'y est pas encore accessible (piège du chapitre 27).

**3. Sur le score :**

```dart
// lib/donjon_game.dart — MODIFIÉ (version du chapitre 38)
  void ajouterScore(int points) {
    if (points <= 0) {
      return;
    }
    score += (points * multiplicateur * reglages.bonusScore).round();
    combo++;
    _tempsRestantCombo = dureeCombo;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }
```

### Le HUD doit suivre

`CompteurVies` du chapitre 38 dessine `Constantes.viesDepart` cœurs, soit toujours 3. En mode facile, le joueur en a 5 : deux seraient invisibles.

```dart
// lib/hud/compteur_score.dart — MODIFIÉ AU CHAPITRE 39
class CompteurVies extends HudMarginComponent
    with HasGameReference<DonjonGame> {
  CompteurVies({EdgeInsets? margin})
      : super(
          margin: margin,
          // On réserve la place du mode le plus généreux.
          size: Vector2(maxCoeurs * (tailleCoeur + ecart), tailleCoeur),
          anchor: Anchor.topLeft,
        );

  static const int maxCoeurs = 5;

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    for (int i = 0; i < game.viesMax; i++) {       // MODIFIÉ
      final double x = i * (tailleCoeur + ecart);
      _dessinerCoeur(canvas, x, 0, tailleCoeur,
          i < game.vies ? _plein : _vide);
    }
  }
  // ... _dessinerCoeur inchangé
}
```

### Le choix dans le menu

```dart
// lib/ecrans/menu_principal.dart — MODIFIÉ AU CHAPITRE 39
            Wrap(
              spacing: 10,
              children: Difficulte.values.map((Difficulte mode) {
                final ReglagesDifficulte r = ReglagesDifficulte.de(mode);
                final bool actif = game.difficulte == mode;
                return ElevatedButton(
                  onPressed: () => game.demarrerPartie(mode: mode),
                  style: ElevatedButton.styleFrom(
                    backgroundColor:
                        actif ? Palette.accent : null,
                  ),
                  child: Text('${r.libelle}  (${r.vies} vies)'),
                );
              }).toList(),
            ),
```

`Difficulte.values` — la liste automatique de toutes les valeurs d'un enum (chapitre 11). Ajouter un quatrième mode ajoutera un quatrième bouton **sans toucher au menu**.

---

## 39.30 — Sauvegarder le niveau atteint (préparation du chapitre 40)

### Ce que l'on sauvegardera, et ce que l'on ne sauvegardera pas

| Donnée | Sauvegardée ? | Pourquoi |
| --- | --- | --- |
| `meilleurScore` | **oui** | c'est la récompense de long terme |
| `niveauMaxAtteint` | **oui** | pour proposer « Continuer » |
| `difficulte` | **oui** | on ne redemande pas à chaque lancement |
| `score`, `vies`, `pv`, positions | non | une partie en cours ne se reprend pas : le jeu se rejoue |

C'est un choix de conception assumé : sauvegarder une partie en cours demanderait de sérialiser le monde entier — positions, PV et états de tous les composants. Sauvegarder une **progression** demande trois nombres.

### Le format : une `Map<String, dynamic>`, comme au chapitre 17

Le chapitre 40 écrira le fichier ; ce chapitre écrit ce qu'on lui donnera. Deux méthodes symétriques suffisent, exactement dans l'esprit `toJson` / `fromJson` du chapitre 17.

```dart
// lib/donjon_game.dart — AJOUT DU CHAPITRE 39

  /// Ce que le chapitre 40 écrira sur le disque.
  Map<String, dynamic> etatDeProgression() => <String, dynamic>{
        'meilleurScore': meilleurScore,
        'niveauMaxAtteint': niveauMaxAtteint,
        'difficulte': difficulte.name,
      };

  /// Ce que le chapitre 40 relira au démarrage.
  /// Tolère une donnée absente, corrompue ou d'une ancienne version.
  void appliquerProgression(Map<String, dynamic> donnees) {
    meilleurScore = (donnees['meilleurScore'] as int?) ?? 0;

    niveauMaxAtteint = ((donnees['niveauMaxAtteint'] as int?) ?? 0)
        .clamp(0, Constantes.nombreNiveaux - 1);

    final String? nomDuMode = donnees['difficulte'] as String?;
    difficulte = Difficulte.values.firstWhere(
      (Difficulte d) => d.name == nomDuMode,
      orElse: () => Difficulte.normal,
    );
  }
```

### Les trois défenses de `appliquerProgression`

1. **`as int?` puis `?? 0`.** Une valeur absente ou d'un autre type ne fait pas planter le jeu ; elle vaut zéro. C'est la lecture défensive du chapitre 17.
2. **`clamp`.** Un fichier trafiqué contenant `niveauMaxAtteint: 47` provoquerait un `RangeError` au premier `chargerNiveau`. Le `clamp` le ramène dans les bornes réelles.
3. **`orElse`.** Une difficulté nommée `'impossible'` — parce que vous aurez renommé un mode entre deux versions — retombe sur `normal` au lieu de lever une `StateError`.

> **`Difficulte.name`.** L'accesseur `name` d'un enum donne le nom de la constante sous forme de chaîne : `Difficulte.facile.name == 'facile'`. C'est bien plus robuste que sauvegarder l'**index** : réordonner l'enum casserait toutes les sauvegardes existantes.

### Le point d'entrée pour le chapitre 40

```dart
  Future<void> continuerLaPartie() async {
    resumeEngine();
    score = 0;
    viesMax = reglages.vies;
    vies = viesMax;
    reinitialiserCombo();
    await chargerNiveau(niveauMaxAtteint);
    await preparerHud();
    changerEtat(GameState.enJeu);
  }
```

Cette méthode fonctionne dès aujourd'hui : le bouton « Continuer » du chapitre 40 n'aura qu'à l'appeler.

---

## 39.31 — Ce que le projet fait à la fin de ce chapitre

### La liste de contrôle

Lancez `flutter run`. Voici ce qui doit se produire, dans l'ordre.

| # | Ce qui doit se produire | Section |
| --- | --- | --- |
| 1 | Le menu propose trois boutons de difficulté | 39.29 |
| 2 | « Normal » démarre le niveau 1, le HUD affiche 3 cœurs | 39.12 |
| 3 | Le décor correspond exactement au dessin ASCII de la section 39.5 | 39.5, 39.8 |
| 4 | Une même bande de sol est **une seule** plateforme, sans accroche | 39.8 |
| 5 | On saute à travers un `=` par le bas et on se pose dessus par le haut | 39.8 |
| 6 | La caméra suit le héros et **ne montre jamais de bande noire** | 39.14 |
| 7 | Le gobelin patrouille autour de sa tuile de la carte | 39.9 |
| 8 | Pousser la porte sans clé secoue la porte et affiche « Il faut une clé » | 39.16 |
| 9 | Ramasser la clé puis pousser la porte : « Ouverte ! » puis fondu au noir | 39.16, 39.19 |
| 10 | Le panneau « NIVEAU 2 — Les oubliettes » s'affiche sur écran noir | 39.18 |
| 11 | Le niveau 2 apparaît, le score est **conservé**, les vies aussi | 39.12 |
| 12 | Le niveau 2 est plus haut : la caméra défile aussi verticalement | 39.14 |
| 13 | Tomber du sommet du niveau 2 ne fait que perdre du temps | 39.5 |
| 14 | Au niveau 3, approcher le boss le réveille : « LE GARDIEN S'ÉVEILLE » | 39.21 |
| 15 | La barre du boss apparaît en bas de l'écran | 39.25 |
| 16 | Le boss charge, se cogne, devient doré : la barre affiche « ÉTOURDI » | 39.22, 39.25 |
| 17 | Frapper un boss étourdi fait chuter sa barre deux fois plus vite | 39.26 |
| 18 | Le boss tire un éventail de 5 projectiles centré sur le héros | 39.23 |
| 19 | Le boss invoque au plus 3 gobelins vivants à la fois | 39.24 |
| 20 | Sous 40 % de PV, le boss devient rouge vif et accélère | 39.26 |
| 21 | À sa mort, « LE GARDIEN TOMBE » et la porte s'ouvre | 39.27 |
| 22 | Franchir la porte affiche l'écran de victoire avec le score final | 39.27 |
| 23 | « Rejouer » relance depuis le niveau 1 **sans plantage** | 39.12 |
| 24 | En mode difficile, une seule vie et des gobelins qui tapent à 18 | 39.29 |
| 25 | Aucune image n'est chargée : `assets/images/` est toujours vide | — |

### L'arborescence

```text
  donjon_de_dart/  ── ÉTAT À LA FIN DU CHAPITRE 39
  │
  └── lib/
      ├── main.dart                     ← MODIFIÉ : overlay de victoire
      ├── donjon_game.dart              ← MODIFIÉ : chargerNiveau, terminerNiveau
      ├── config/
      │   ├── constantes.dart           ← MODIFIÉ : porte, boss, difficulté
      │   └── palette.dart              ← MODIFIÉ : porte, boss, voile
      ├── core/
      │   ├── game_state.dart
      │   ├── entite.dart
      │   └── sante.dart
      ├── composants/
      │   ├── joueur.dart               ← MODIFIÉ : plateformes traversables
      │   ├── plateforme.dart           ← MODIFIÉ : drapeau traversable
      │   ├── ennemi.dart               ← MODIFIÉ : cible nulle, difficulté
      │   ├── gobelin.dart              ← MODIFIÉ : cible nulle
      │   ├── chauvesouris.dart         ← MODIFIÉ : cible nulle
      │   ├── projectile.dart
      │   ├── collectible.dart
      │   ├── piece.dart
      │   ├── potion.dart
      │   ├── cle.dart
      │   ├── texte_flottant.dart
      │   ├── porte.dart                ← NOUVEAU
      │   └── boss.dart                 ← NOUVEAU
      ├── niveaux/                      ← NOUVEAU DOSSIER
      │   ├── niveau.dart               ← NOUVEAU
      │   └── niveaux_data.dart         ← NOUVEAU
      ├── hud/
      │   ├── hud.dart                  ← MODIFIÉ : BarreBoss, VoileTransition
      │   ├── barre_de_vie.dart         ← MODIFIÉ : BarreBoss
      │   └── compteur_score.dart       ← MODIFIÉ : viesMax
      └── ecrans/
          └── menu_principal.dart       ← MODIFIÉ : choix de la difficulté
```

### Ce qui manque encore

| Manque | Chapitre |
| --- | --- |
| Le moindre son | 40 |
| La pause | 40 |
| Un vrai écran de Game Over et de victoire | 40 |
| Le meilleur score conservé entre deux lancements | 40 |
| Un bouton « Continuer » au menu | 40 |
| Des tests automatisés et un build Android / Web | 42 |

Le jeu est **fini du point de vue du gameplay**. Ce qui reste relève de la finition, et c'est exactement l'objet du chapitre 40.

---

## 39.32 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le clic sur « Jouer » ne fait rien, aucune exception | `await monde.add(...)` pendant que le moteur est en pause : la file de cycle de vie n'est jamais traitée | `resumeEngine()` **avant** tout `await` d'ajout de composant |
| `LateInitializationError: Field 'hud' has already been initialized` | `late final Hud hud` réaffecté au deuxième `demarrerPartie` | `Hud? _hud` + un getter, et ne créer le HUD qu'une fois |
| `FormatException: ligne 7 de longueur 41, attendu 40` | un espace en trop en fin de ligne dans la carte | c'est le validateur qui fait son travail : corrigez la carte |
| Une pièce a disparu de la carte sans erreur | vous avez tapé `O` (lettre) au lieu de `o` (chiffre… non, la lettre minuscule) | le validateur l'attrape : ne le désactivez jamais |
| Le niveau est transposé ou lève une `RangeError` | `carte[c][l]` au lieu de `carte[l][c]` | ligne d'abord, colonne ensuite |
| Le héros réapparaît là où il vient de mourir | `pointDeReapparition` et `joueur.position` partagent le même `Vector2` | `clone()` des deux côtés |
| Le héros accroche entre deux dalles de sol | une plateforme par tuile : les arêtes internes trompent la résolution par plus petit chevauchement | fusionner les suites horizontales en une seule `Plateforme` |
| Le héros ne peut plus sauter à travers un `=` | la résolution traite les plateformes traversables comme des murs | la garde `if (plateforme.traversable) { ... }` avec ses quatre conditions |
| Le héros traverse une plateforme sur laquelle il devrait se poser | le seuil `_seuilAtterrissage` est plus petit que le déplacement d'une frame en chute libre | seuil ≥ `vitesseMaxChute / 60`, soit 15 px |
| Le gobelin fait demi-tour devant chaque perchoir | `murDevant()` compte les plateformes traversables comme des murs | filtrer `!parent.traversable` dans `hitboxFilter` |
| Le boss s'assomme sur une plateforme au lieu du mur | même cause | même correction |
| `The property 'absoluteCenter' can't be unconditionally accessed` | `game.joueur` est devenu nullable ; un champ d'instance n'est pas promu par un `if` | copier dans une variable locale, tester la locale |
| La caméra reste bloquée sur l'ancien niveau | `camera.follow` pointe encore sur le joueur retiré | `camera.stop()` dans `viderLeMonde()` |
| Bandes noires sur les bords du niveau | bornes absentes, ou `considerViewport` à `false` | `setBounds(Rectangle.fromLTRB(...), considerViewport: true)` |
| Assertion sur les bornes dans une grande fenêtre | le niveau est plus petit que la zone visible : rectangle de largeur négative | `max(niveau.largeurPixels, vue.x)` |
| `The setter 'worldBounds' isn't defined` | API supprimée depuis longtemps | `camera.setBounds(...)`, `Rectangle` venant de `package:flame/experimental.dart` |
| Le niveau se charge trois fois de suite | `terminerNiveau()` rappelée à chaque frame de contact avec la porte | la garde `if (transitionEnCours) return;` **et** le verrou `_franchie` de la porte |
| Soixante « Il faut une clé » superposés | un texte flottant créé à chaque frame par `onCollision` | un throttle : `_delaiRefus` de 1,5 s |
| La porte s'ouvre mais rien ne se passe si le héros est déjà dedans | `onCollisionStart` est déjà passée | traiter aussi `onCollision`, uniquement quand la porte est ouverte |
| Le fondu ne se termine jamais | le `Completer` n'est jamais complété parce que `onComplete` a été oublié | `onComplete: fini.complete` sur l'effet |
| L'écran reste gris à moitié pour toujours | un `OpacityEffect` interrompu laisse l'opacité en l'état | créer et retirer la voile à chaque transition |
| Le voile ne couvre que le coin haut-gauche | `size` non initialisée, ou non mise à jour au redimensionnement | `size = game.size.clone()` dans `onLoad` **et** dans `onGameResize` |
| Plantage au redimensionnement pendant le chargement | `onGameResize` s'exécute avant `onLoad` | garder par `if (isLoaded)` |
| Le voile passe derrière le HUD | priorité insuffisante | `priority: 1000` contre 10 pour le `Hud` |
| Le boss invoque vingt gobelins et le jeu rame | aucun plafond global | compter les `Gobelin` **vivants** et borner à `sbiresMaxBoss` |
| Le boss croit la salle pleine alors qu'elle se vide | les gobelins morts restent dans `children` pendant leur animation | filtrer par `estVivant` |
| L'éventail est décentré vers la gauche | division par `nombre` au lieu de `nombre - 1` | `i / (nombre - 1)`, avec la garde `nombre == 1` |
| Le boss charge éternellement | aucune sortie par le temps, et le rayon de mur rate le mur | toute phase doit avoir **deux** sorties : logique et temporelle |
| La barre du boss clignote | on retire et rajoute le composant à chaque changement | cacher par `scale = Vector2.all(0.001)` |
| La rage se redéclenche à chaque coup | on teste un **état** (`enrage`) au lieu d'une **transition** | mémoriser `etaitEnrage` avant `super.subirDegats` |
| Deux cœurs manquent en mode facile | `CompteurVies` dessine `Constantes.viesDepart` cœurs | dessiner `game.viesMax` cœurs, et réserver la place de 5 |
| Une sauvegarde trafiquée fait planter le jeu | `niveauMaxAtteint` hors bornes | `clamp(0, Constantes.nombreNiveaux - 1)` |
| Renommer un mode de difficulté casse les sauvegardes | on avait sauvegardé l'`index` de l'enum | sauvegarder `.name`, relire avec `firstWhere(..., orElse: ...)` |

---

## 39.33 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Données contre code | Un niveau est une **donnée**. `List<String>` garde le dessin visible dans le source, éditable, comparable en diff. |
| `DefinitionNiveau` | Objet immuable : nom, sous-titre, carte, verrou de porte. Tout `final`, constructeur `const`. |
| Légende | Un `Set<String>` de caractères connus. Un caractère inconnu est une **erreur**, jamais un vide. |
| `Niveau` | N'est **pas** un composant Flame : c'est un traducteur texte → composants, testable sans lancer le jeu. |
| Valider d'abord | `FormatException` avec le nom du niveau, la ligne et la colonne. Quatre fautes classiques attrapées avant la première frame. |
| `carte[l][c]` | Ligne d'abord, colonne ensuite. `x = c * 32`, `y = l * 32`. |
| `poserSurTuile` | `(ligne + 1) * t - taille.y` aligne les **pieds**, quelle que soit la taille de l'entité. |
| Fusion des tuiles | Une suite horizontale de `#` devient **une** `Plateforme`. Moins de hitbox, et surtout plus d'arêtes internes. |
| `traversable` | Un `=` ne bloque que par le dessus : on descend, à plat, par le haut, et pas trop tard. Quatre gardes. |
| Ordre d'ajout | Décor, puis joueur, puis porte, puis objets et ennemis. C'est une contrainte du domaine, pas un détail. |
| `resumeEngine()` d'abord | Moteur en pause = file de cycle de vie gelée = `await add(...)` qui ne revient jamais. |
| `viderLeMonde` | Retirer les composants **et** couper les références : `camera.stop()`, `joueur`, `boss`, `porte` à `null`. |
| Cible nullable | Entre deux niveaux, `game.joueur` vaut `null`. Copier dans une locale, tester la locale, utiliser la locale. |
| `setBounds` | `Rectangle.fromLTRB` de `package:flame/experimental.dart`, `considerViewport: true`, et un `max` de sécurité. |
| `Porte` | Hitbox **passive**, deux verrous indépendants (clé, boss), un verrou de franchissement, un refus throttlé. |
| `consommerUneCle()` | Teste et consomme en une opération : impossible d'ouvrir deux portes avec une clé. |
| `terminerNiveau()` | `void`, jamais `Future` : elle est appelée depuis un callback de collision. Elle déclenche, elle n'attend pas. |
| Garde de ré-entrance | `transitionEnCours` côté jeu **plus** `_franchie` côté porte. Deux verrous, deux propriétaires. |
| Transition | Voile dans le **viewport**, `priority: 1000`, créée et retirée à chaque transition. |
| Fondu | `OpacityEffect` + `Completer` : `onComplete: fini.complete` transforme un effet en `Future` attendable. |
| `easeIn` / `easeOut` | On tombe dans le noir (`easeIn`), le niveau se pose (`easeOut`). |
| `Boss extends Ennemi` | Il n'hérite qu'une nouveauté : sa façon de décider. Tout le reste est déjà écrit. |
| Machine à phases | Entrée (armer, viser, teindre), mise à jour (agir), sortie (décider). Le cycle est une `List`, pas du code. |
| Charge | On vise **à l'entrée**, pas à chaque frame : c'est ce qui rend l'esquive apprenable. |
| Éventail | `atan2(dy, dx)` pour la base, `i / (nombre - 1)` pour la répartition. |
| Invocation | Deux plafonds : par phase, et global sur les sbires **vivants**. |
| Fenêtre de vulnérabilité | Préférée au point faible géométrique : lisible sans sprite, compatible avec la chaîne de dégâts existante. |
| Signaler l'état | Trois signaux simultanés pour la fenêtre : couleur du corps, texte flottant, barre de HUD. |
| Courbe de difficulté | Enseigner, tester, couronner. Un mécanisme nouveau par niveau. Le boss ne demande rien de neuf. |
| Modes de difficulté | Vies, PV ennemis, dégâts subis **et** multiplicateur de score : sans le dernier, les scores ne se comparent plus. |
| Progression | Trois nombres suffisent. `enum.name` et jamais l'index. Lecture défensive : `as T?`, `??`, `clamp`, `orElse`. |

---

## 39.34 — Code complet du chapitre

Quatre fichiers sont **nouveaux** : copiez-les tels quels. Les autres blocs sont des **ajouts et des modifications** de fichiers écrits aux chapitres 35 à 38 : insérez-les, ne remplacez pas les fichiers entiers.

### `lib/niveaux/niveaux_data.dart` (nouveau)

```dart
/// Les données des niveaux du « Donjon de Dart ».
/// Aucun import : ce fichier ne connaît ni Flame ni Flutter.

class DefinitionNiveau {
  const DefinitionNiveau({
    required this.nom,
    required this.soustitre,
    required this.carte,
    this.porteVerrouillee = true,
  });

  final String nom;
  final String soustitre;
  final List<String> carte;

  /// Faux pour la salle du boss : sa porte n'attend pas une clé.
  final bool porteVerrouillee;

  int get colonnes => carte.isEmpty ? 0 : carte.first.length;
  int get lignes => carte.length;
}

class NiveauxData {
  NiveauxData._();

  // ---- Légende (voir le tableau de la section 39.4) -------------------
  static const String mur = '#';
  static const String vide = '.';
  static const String plateforme = '=';
  static const String joueur = 'J';
  static const String gobelin = 'g';
  static const String chauvesouris = 'c';
  static const String piece = 'o';
  static const String potion = 'p';
  static const String cle = 'k';
  static const String porte = 'D';
  static const String boss = 'B';

  static const Set<String> caracteresConnus = <String>{
    mur, vide, plateforme, joueur, gobelin,
    chauvesouris, piece, potion, cle, porte, boss,
  };

  // ---- Niveau 1 : 40 x 14 --------------------------------------------
  static const List<String> carteNiveau1 = <String>[
    '########################################',
    '#......................................#',
    '#.........o.o.o........................#',
    '#.......========.......................#',
    '#..................o.o.o...............#',
    '#................========..............#',
    '#......c...........................c...#',
    '#............................o.o.o.....#',
    '#..........................========....#',
    '#...........o..........................#',
    '#.................p....................#',
    '#.....g....###.............g...........#',
    '#.J........###.................k.....D.#',
    '########################################',
  ];

  // ---- Niveau 2 : 48 x 18 --------------------------------------------
  static const List<String> carteNiveau2 = <String>[
    '################################################',
    '#.J...........................o.o.o............#',
    '#=====.........................................#',
    '#..........o.o.o...............................#',
    '#.....========....................c............#',
    '#..............................................#',
    '#.......p..................o.o.o...............#',
    '#..========...............=========............#',
    '#..............................................#',
    '#.................o.o.o...............o.o.o....#',
    '#...............========...........=========...#',
    '#..............................................#',
    '#....o.o.o..........g..............p......k....#',
    '#...========.......................========....#',
    '#..............................................#',
    '#..g...................o.o.o.......c...........#',
    '#......o.o......c.....========....g........D...#',
    '################################################',
  ];

  // ---- Niveau 3 : 40 x 14 --------------------------------------------
  static const List<String> carteNiveau3 = <String>[
    '########################################',
    '#......................................#',
    '#....o.o..........................o.o..#',
    '#...======....................======...#',
    '#......................................#',
    '#..........p................p..........#',
    '#......................................#',
    '#.......========......========.........#',
    '#......................................#',
    '#......................................#',
    '#....o....................o............#',
    '#......................................#',
    '#..J..............B....................#',
    '########################################',
  ];

  static const List<DefinitionNiveau> niveaux = <DefinitionNiveau>[
    DefinitionNiveau(
      nom: 'Les caves',
      soustitre: 'Trouvez la clé, ouvrez la porte.',
      carte: carteNiveau1,
    ),
    DefinitionNiveau(
      nom: 'Les oubliettes',
      soustitre: 'Descendez. La clé est tout en bas.',
      carte: carteNiveau2,
    ),
    DefinitionNiveau(
      nom: 'La salle du gardien',
      soustitre: 'La porte ne s\'ouvrira qu\'à sa chute.',
      carte: carteNiveau3,
      porteVerrouillee: false,
    ),
  ];

  static int get nombre => niveaux.length;

  /// Toujours bornée : jamais de RangeError, même sur un index trafiqué.
  static DefinitionNiveau definition(int index) =>
      niveaux[index.clamp(0, niveaux.length - 1)];
}
```

### `lib/niveaux/niveau.dart` (nouveau)

```dart
import 'package:flame/components.dart';

import '../composants/boss.dart';
import '../composants/chauvesouris.dart';
import '../composants/cle.dart';
import '../composants/gobelin.dart';
import '../composants/joueur.dart';
import '../composants/piece.dart';
import '../composants/plateforme.dart';
import '../composants/porte.dart';
import '../composants/potion.dart';
import '../config/constantes.dart';
import 'niveaux_data.dart';

/// Traduit une carte texte en composants Flame. À USAGE UNIQUE.
class Niveau {
  Niveau(this.definition);

  factory Niveau.numero(int index) => Niveau(NiveauxData.definition(index));

  final DefinitionNiveau definition;

  List<String> get carte => definition.carte;

  static const double t = Constantes.tailleTuile;
  static const double epaisseurPlateforme = 10.0;

  static final Vector2 tailleJoueur = Vector2(24, 32);
  static final Vector2 tailleGobelin = Vector2(24, 30);
  static final Vector2 tailleChauvesouris = Vector2(22, 16);
  static final Vector2 tailleBoss = Vector2(48, 56);

  int get colonnes => definition.colonnes;
  int get lignes => definition.lignes;
  double get largeurPixels => colonnes * t;
  double get hauteurPixels => lignes * t;

  final List<Plateforme> plateformes = <Plateforme>[];
  final List<PositionComponent> ennemis = <PositionComponent>[];
  final List<PositionComponent> collectibles = <PositionComponent>[];

  Vector2 apparitionJoueur = Vector2.zero();
  Joueur? joueur;
  Porte? porte;
  Boss? boss;
  int nombreDePieces = 0;

  bool _analyse = false;

  bool get contientBoss => boss != null;

  // ---- Conversion tuile -> pixels --------------------------------------

  static Vector2 coinDe(int colonne, int ligne) =>
      Vector2(colonne * t, ligne * t);

  static Vector2 centreDe(int colonne, int ligne) =>
      Vector2(colonne * t + t / 2, ligne * t + t / 2);

  /// Position `topLeft` d'une entité posée AU SOL de la tuile.
  static Vector2 poserSurTuile(int colonne, int ligne, Vector2 taille) =>
      Vector2(colonne * t + (t - taille.x) / 2, (ligne + 1) * t - taille.y);

  // ---- Validation ------------------------------------------------------

  void valider() {
    if (carte.isEmpty) {
      throw FormatException(
        'Niveau "${definition.nom}" : la carte ne contient aucune ligne.',
      );
    }

    final int largeur = carte.first.length;
    if (largeur == 0) {
      throw FormatException(
        'Niveau "${definition.nom}" : la première ligne est vide.',
      );
    }

    int nombreDeJ = 0;
    int nombreDeD = 0;

    for (int l = 0; l < carte.length; l++) {
      final String ligne = carte[l];

      if (ligne.length != largeur) {
        throw FormatException(
          'Niveau "${definition.nom}" : ligne $l de longueur '
          '${ligne.length}, attendu $largeur.',
        );
      }

      for (int c = 0; c < ligne.length; c++) {
        final String caractere = ligne[c];

        if (!NiveauxData.caracteresConnus.contains(caractere)) {
          throw FormatException(
            'Niveau "${definition.nom}" : caractère "$caractere" inconnu '
            'en ligne $l, colonne $c.',
          );
        }

        if (caractere == NiveauxData.joueur) {
          nombreDeJ++;
        }
        if (caractere == NiveauxData.porte) {
          nombreDeD++;
        }
      }
    }

    if (nombreDeJ != 1) {
      throw FormatException(
        'Niveau "${definition.nom}" : il faut exactement un "J", '
        'trouvé $nombreDeJ.',
      );
    }
    if (nombreDeD != 1) {
      throw FormatException(
        'Niveau "${definition.nom}" : il faut exactement une porte "D", '
        'trouvé $nombreDeD.',
      );
    }
  }

  // ---- Analyse ---------------------------------------------------------

  void analyser() {
    if (_analyse) {
      return;
    }
    valider();
    _construireDecor();
    _construireEntites();
    porte?.attendLeBoss = boss != null;
    _analyse = true;
  }

  void _construireDecor() {
    for (int l = 0; l < lignes; l++) {
      final String ligne = carte[l];
      int c = 0;

      while (c < colonnes) {
        final String caractere = ligne[c];

        if (caractere != NiveauxData.mur &&
            caractere != NiveauxData.plateforme) {
          c++;
          continue;
        }

        int fin = c;
        while (fin + 1 < colonnes && ligne[fin + 1] == caractere) {
          fin++;
        }

        final int nombreDeTuiles = fin - c + 1;
        final bool traversable = caractere == NiveauxData.plateforme;

        plateformes.add(
          Plateforme(
            position: coinDe(c, l),
            size: Vector2(
              nombreDeTuiles * t,
              traversable ? epaisseurPlateforme : t,
            ),
            traversable: traversable,
          ),
        );

        c = fin + 1;
      }
    }
  }

  void _construireEntites() {
    for (int l = 0; l < lignes; l++) {
      final String ligne = carte[l];

      for (int c = 0; c < colonnes; c++) {
        switch (ligne[c]) {
          case NiveauxData.joueur:
            apparitionJoueur = poserSurTuile(c, l, tailleJoueur);
            joueur = Joueur(position: apparitionJoueur.clone());
            break;

          case NiveauxData.gobelin:
            ennemis.add(Gobelin(position: poserSurTuile(c, l, tailleGobelin)));
            break;

          case NiveauxData.chauvesouris:
            ennemis.add(
              Chauvesouris(
                position: centreDe(c, l) - tailleChauvesouris / 2,
              ),
            );
            break;

          case NiveauxData.boss:
            final Boss gardien = Boss(position: poserSurTuile(c, l, tailleBoss));
            boss = gardien;
            ennemis.add(gardien);
            break;

          case NiveauxData.piece:
            collectibles.add(Piece(position: centreDe(c, l)));
            nombreDePieces++;
            break;

          case NiveauxData.potion:
            collectibles.add(Potion(position: centreDe(c, l)));
            break;

          case NiveauxData.cle:
            collectibles.add(Cle(position: centreDe(c, l)));
            break;

          case NiveauxData.porte:
            porte = Porte(
              position: poserSurTuile(
                c,
                l,
                Vector2(Constantes.largeurPorte, Constantes.hauteurPorte),
              ),
              verrouillee: definition.porteVerrouillee,
            );
            break;

          default:
            break;   // '#', '=' et '.' : déjà traités ou sans effet
        }
      }
    }
  }

  // ---- Construction ----------------------------------------------------

  /// Ajoute tout le niveau au monde, DANS L'ORDRE.
  Future<void> construireDans(World monde) async {
    analyser();

    await monde.addAll(plateformes);

    final Joueur? heros = joueur;
    if (heros != null) {
      await monde.add(heros);
    }

    final Porte? sortie = porte;
    if (sortie != null) {
      await monde.add(sortie);
    }

    await monde.addAll(collectibles);
    await monde.addAll(ennemis);
  }
}
```

### `lib/composants/porte.dart` (nouveau)

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../donjon_game.dart';
import 'joueur.dart';

/// La sortie d'un niveau. Capteur passif : elle ne bloque jamais le joueur.
class Porte extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Porte({required Vector2 position, this.verrouillee = true})
      : super(
          position: position,
          size: Vector2(Constantes.largeurPorte, Constantes.hauteurPorte),
          anchor: Anchor.topLeft,
          priority: -1,
        );

  bool verrouillee;
  bool attendLeBoss = false;

  bool _ouverte = false;
  bool _franchie = false;
  double _delaiRefus = 0;

  bool get ouverte => _ouverte;

  late final RectangleComponent _battant;
  late final RectangleComponent _serrure;

  @override
  Future<void> onLoad() async {
    _battant = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.porte,
    );
    final RectangleComponent cadre = RectangleComponent(
      size: size.clone(),
      paint: Paint()
        ..color = Palette.porteCadre
        ..style = PaintingStyle.stroke
        ..strokeWidth = 3,
    );
    _serrure = RectangleComponent(
      size: Vector2(6, 9),
      position: Vector2(size.x - 10, size.y / 2 - 4),
      paint: Paint()..color = Palette.serrure,
    );

    await addAll(<Component>[_battant, cadre, _serrure]);
    add(RectangleHitbox(collisionType: CollisionType.passive));

    if (!verrouillee && !attendLeBoss) {
      ouvrir(silencieux: true);
    }
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_delaiRefus > 0) {
      _delaiRefus -= dt;
    }
  }

  // ---- Ouverture -------------------------------------------------------

  void ouvrir({bool silencieux = false}) {
    if (_ouverte) {
      return;
    }
    _ouverte = true;
    verrouillee = false;
    attendLeBoss = false;

    _serrure.removeFromParent();
    _battant.paint.color = Palette.porteOuverte;

    if (!silencieux) {
      add(
        ScaleEffect.by(
          Vector2(1.0, 1.12),
          EffectController(duration: 0.12, alternate: true),
        ),
      );
      game.afficherTexteFlottant(absoluteCenter, 'Ouverte !', Palette.serrure);
    }
  }

  /// Appelée par `DonjonGame` quand le boss du niveau tombe.
  void deverrouillerParLeBoss() {
    attendLeBoss = false;
    if (!verrouillee) {
      ouvrir();
    }
  }

  // ---- Franchissement --------------------------------------------------

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);
    if (other is Joueur) {
      _essayerDeFranchir(other);
    }
  }

  @override
  void onCollision(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollision(intersectionPoints, other);
    if (other is Joueur && _ouverte) {
      _essayerDeFranchir(other);
    }
  }

  void _essayerDeFranchir(Joueur heros) {
    if (_franchie) {
      return;
    }

    if (attendLeBoss) {
      _refuser('Le gardien veille encore');
      return;
    }

    if (verrouillee) {
      if (!heros.consommerUneCle()) {
        _refuser('Il faut une clé');
        return;
      }
      ouvrir();
    }

    if (!_ouverte) {
      return;
    }

    _franchie = true;
    game.terminerNiveau();
  }

  void _refuser(String message) {
    if (_delaiRefus > 0) {
      return;
    }
    _delaiRefus = 1.5;

    add(
      MoveEffect.by(
        Vector2(3, 0),
        EffectController(duration: 0.05, alternate: true, repeatCount: 3),
      ),
    );
    game.afficherTexteFlottant(
      absoluteCenter,
      message,
      Palette.serrure,
      taille: 10,
    );
  }
}
```

### `lib/composants/boss.dart` (nouveau)

```dart
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/particles.dart';
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import 'ennemi.dart';
import 'gobelin.dart';
import 'joueur.dart';
import 'projectile.dart';

enum PhaseBoss { sommeil, charge, eventail, invocation, etourdi, mort }

/// Le gardien du niveau 3. Hérite de tout `Ennemi` sauf sa façon de décider.
class Boss extends Ennemi {
  Boss({required Vector2 position})
      : super(
          position: position,
          taille: Vector2(48, 56),
          vitesse: Constantes.vitesseBoss,
          degatsContact: Constantes.degatsContactBoss,
          pointsScore: Constantes.scoreBoss,
          couleur: Palette.boss,
          pvDepart: Constantes.pvBoss,
        );

  /// Le cycle nominal. Pour ré-équilibrer, on change cette liste.
  static const List<PhaseBoss> cycle = <PhaseBoss>[
    PhaseBoss.charge,
    PhaseBoss.eventail,
    PhaseBoss.charge,
    PhaseBoss.invocation,
  ];

  PhaseBoss phase = PhaseBoss.sommeil;

  int _indiceCycle = 0;
  double _minuterie = 0;
  double _cadence = 0;
  int _sbiresInvoques = 0;

  bool get enrage => ratioPv <= Constantes.seuilEnrageBoss;
  bool get vulnerable => phase == PhaseBoss.etourdi;
  double get facteurEnrage => enrage ? 1.4 : 1.0;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    rayonAggro = Constantes.rayonReveilBoss;
    rayonAbandon = double.infinity;
    demiPatrouille = 0;
  }

  @override
  void onMount() {
    super.onMount();
    game.boss = this;
  }

  @override
  void onRemove() {
    if (game.boss == this) {
      game.boss = null;
    }
    super.onRemove();
  }

  // ---- Machine à phases -------------------------------------------------

  @override
  void mettreAJourIA(double dt) {
    if (_cadence > 0) {
      _cadence -= dt;
    }
    _minuterie -= dt;

    switch (phase) {
      case PhaseBoss.sommeil:
        _dormir();
        break;
      case PhaseBoss.charge:
        _charger();
        break;
      case PhaseBoss.eventail:
        _tirerEnEventail();
        break;
      case PhaseBoss.invocation:
        _invoquer();
        break;
      case PhaseBoss.etourdi:
        _rester();
        break;
      case PhaseBoss.mort:
        velocite.setZero();
        break;
    }
  }

  void _phaseSuivante() {
    _entrerDans(cycle[_indiceCycle % cycle.length]);
    _indiceCycle++;
  }

  void _entrerDans(PhaseBoss nouvelle) {
    phase = nouvelle;
    velocite.setZero();
    _sbiresInvoques = 0;
    _cadence = 0;

    if (nouvelle == PhaseBoss.charge) {
      _minuterie = Constantes.dureeChargeBoss / facteurEnrage;
      _viser();
    } else if (nouvelle == PhaseBoss.eventail) {
      _minuterie = Constantes.dureeEventailBoss;
    } else if (nouvelle == PhaseBoss.invocation) {
      _minuterie = Constantes.dureeInvocationBoss;
    } else if (nouvelle == PhaseBoss.etourdi) {
      _minuterie = Constantes.dureeEtourdiBoss;
    } else {
      _minuterie = 0;
    }

    _teindreCorps();
  }

  void _teindreCorps() {
    if (phase == PhaseBoss.etourdi) {
      corps.paint.color = Palette.bossEtourdi;
    } else if (enrage) {
      corps.paint.color = Palette.bossEnrage;
    } else {
      corps.paint.color = Palette.boss;
    }
  }

  // ---- Phase 0 : sommeil ------------------------------------------------

  void _dormir() {
    velocite.x = 0;
    if (distanceAuJoueur <= Constantes.rayonReveilBoss) {
      _rugir();
      _phaseSuivante();
    }
  }

  void _rugir() {
    corps.add(
      ScaleEffect.by(
        Vector2.all(1.25),
        EffectController(duration: 0.18, alternate: true),
      ),
    );
    game.afficherTexteFlottant(
      absoluteCenter,
      'LE GARDIEN S\'ÉVEILLE',
      Palette.bossEnrage,
      taille: 13,
    );
  }

  // ---- Phase 1 : charge -------------------------------------------------

  void _viser() {
    final Joueur? heros = cible;
    if (heros == null) {
      return;
    }
    sens = heros.absoluteCenter.x >= absoluteCenter.x ? 1 : -1;
  }

  void _charger() {
    velocite.x = sens * vitesse * Constantes.facteurChargeBoss * facteurEnrage;

    if (murDevant()) {
      _entrerDans(PhaseBoss.etourdi);
      _impact();
      return;
    }
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  void _impact() {
    game.monde.add(
      ParticleSystemComponent(
        position: absoluteCenter.clone(),
        particle: _poussiere(),
      ),
    );
    game.afficherTexteFlottant(
      absoluteCenter,
      'Étourdi !',
      Palette.bossEtourdi,
      taille: 12,
    );
    corps.add(
      ScaleEffect.by(
        Vector2(1.25, 0.78),
        EffectController(duration: 0.1, alternate: true),
      ),
    );
  }

  void _rester() {
    velocite.x = 0;
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  // ---- Phase 2 : éventail -----------------------------------------------

  void _tirerEnEventail() {
    velocite.x = 0;
    if (_cadence <= 0) {
      _salve();
      _cadence = enrage ? 0.45 : 0.70;
    }
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  void _salve() {
    final Joueur? heros = cible;
    final Vector2 vers = heros == null
        ? Vector2(sens.toDouble(), 0)
        : heros.absoluteCenter - absoluteCenter;
    if (vers.length2 < 1) {
      return;
    }

    final double base = atan2(vers.y, vers.x);
    final int nombre = Constantes.projectilesEventailBoss + (enrage ? 2 : 0);
    const double ouverture = Constantes.ouvertureEventailBoss;

    for (int i = 0; i < nombre; i++) {
      final double fraction = nombre == 1 ? 0.5 : i / (nombre - 1);
      final double angle = base - ouverture / 2 + ouverture * fraction;

      game.monde.add(
        Projectile(
          position: absoluteCenter.clone(),
          direction: Vector2(cos(angle), sin(angle)),
          vitesse: Constantes.vitesseProjectile * 1.15,
        ),
      );
    }

    corps.add(
      ScaleEffect.by(
        Vector2(0.85, 1.2),
        EffectController(duration: 0.09, alternate: true),
      ),
    );
  }

  // ---- Phase 3 : invocation ---------------------------------------------

  void _invoquer() {
    velocite.x = 0;

    if (_cadence <= 0 && _placesDisponibles() > 0) {
      _invoquerUnGobelin();
      _sbiresInvoques++;
      _cadence = 0.4;
    }

    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }

  int _placesDisponibles() {
    final int vivants = game.monde.children
        .whereType<Gobelin>()
        .where((Gobelin g) => g.estVivant)
        .length;
    return Constantes.sbiresMaxBoss - vivants;
  }

  void _invoquerUnGobelin() {
    final double ecart = Constantes.tailleTuile * (2 + _sbiresInvoques);
    final double decalage = _sbiresInvoques.isEven ? -ecart : ecart;

    final Vector2 point = Vector2(
      absoluteCenter.x + decalage - 12,
      absoluteCenter.y - Constantes.tailleTuile,
    );

    game.monde.add(Gobelin(position: point));
    game.monde.add(
      ParticleSystemComponent(
        position: point.clone()..add(Vector2(12, 15)),
        particle: _poussiere(),
      ),
    );
  }

  // ---- Dégâts et mort ----------------------------------------------------

  @override
  void subirDegats(double degats) {
    if (phase == PhaseBoss.sommeil || phase == PhaseBoss.mort) {
      return;
    }

    final double facteur = vulnerable
        ? Constantes.bonusDegatsEtourdi
        : Constantes.reductionDegatsBoss;

    final bool etaitEnrage = enrage;
    super.subirDegats(degats * facteur);

    if (!etaitEnrage && enrage && estVivant) {
      _entrerEnRage();
    }
  }

  void _entrerEnRage() {
    game.afficherTexteFlottant(
      absoluteCenter,
      'ENRAGÉ',
      Palette.bossEnrage,
      taille: 13,
    );
    _entrerDans(PhaseBoss.charge);
  }

  @override
  void mourir() {
    if (phase == PhaseBoss.mort) {
      return;
    }
    phase = PhaseBoss.mort;
    _minuterie = 0;

    game.bossVaincu(this);

    for (int i = 0; i < 3; i++) {
      game.monde.add(
        ParticleSystemComponent(
          position: absoluteCenter.clone(),
          priority: 30,
          particle: _poussiere(),
        ),
      );
    }

    super.mourir();
  }

  // ---- Décor ------------------------------------------------------------

  Particle _poussiere() {
    final Random hasard = Random();
    return Particle.generate(
      count: 12,
      lifespan: 0.45,
      generator: (int i) => AcceleratedParticle(
        speed: Vector2(
          hasard.nextDouble() * 140 - 70,
          hasard.nextDouble() * -90 - 20,
        ),
        acceleration: Vector2(0, 340),
        child: CircleParticle(
          radius: 1.5 + hasard.nextDouble() * 2.5,
          paint: Paint()..color = Palette.bossEtourdi,
        ),
      ),
    );
  }
}
```

### `lib/config/constantes.dart` (modifié — ajouts du chapitre 39)

```dart
class Constantes {
  // ... tout le contenu des chapitres 35 et 37, inchangé

  // ---- Chapitre 39 : porte et transitions ----
  static const double largeurPorte = 26.0;
  static const double hauteurPorte = 46.0;
  static const double dureeFonduTransition = 0.45;
  static const double dureePanneauTransition = 1.2;
  static const int bonusFinNiveau = 200;
  static const int bonusSalleVidee = 300;
  static const int bonusBoss = 1000;

  // ---- Chapitre 39 : boss ----
  static const double pvBoss = 320.0;
  static const double vitesseBoss = 70.0;
  static const double degatsContactBoss = 20.0;
  static const int scoreBoss = 750;
  static const double rayonReveilBoss = 220.0;
  static const double dureeChargeBoss = 2.2;
  static const double facteurChargeBoss = 2.6;
  static const double dureeEtourdiBoss = 2.4;
  static const double dureeEventailBoss = 2.0;
  static const double dureeInvocationBoss = 1.6;
  static const int projectilesEventailBoss = 5;
  static const double ouvertureEventailBoss = 1.1;   // radians
  static const int sbiresMaxBoss = 3;
  static const double seuilEnrageBoss = 0.4;
  static const double reductionDegatsBoss = 0.5;
  static const double bonusDegatsEtourdi = 2.0;
}

// ---- Chapitre 39 : difficulté (hors de la classe Constantes) ----
enum Difficulte { facile, normal, difficile }

class ReglagesDifficulte {
  const ReglagesDifficulte({
    required this.libelle,
    required this.vies,
    required this.pvEnnemis,
    required this.degatsEnnemis,
    required this.bonusScore,
  });

  final String libelle;
  final int vies;
  final double pvEnnemis;
  final double degatsEnnemis;
  final double bonusScore;

  static const Map<Difficulte, ReglagesDifficulte> table =
      <Difficulte, ReglagesDifficulte>{
    Difficulte.facile: ReglagesDifficulte(
      libelle: 'Facile',
      vies: 5,
      pvEnnemis: 0.75,
      degatsEnnemis: 0.6,
      bonusScore: 0.75,
    ),
    Difficulte.normal: ReglagesDifficulte(
      libelle: 'Normal',
      vies: 3,
      pvEnnemis: 1.0,
      degatsEnnemis: 1.0,
      bonusScore: 1.0,
    ),
    Difficulte.difficile: ReglagesDifficulte(
      libelle: 'Difficile',
      vies: 1,
      pvEnnemis: 1.4,
      degatsEnnemis: 1.5,
      bonusScore: 1.5,
    ),
  };

  static ReglagesDifficulte de(Difficulte mode) => table[mode]!;
}
```

### `lib/config/palette.dart` (modifié — ajouts du chapitre 39)

```dart
class Palette {
  // ... chapitres 35 et 37, inchangés

  // ---- Chapitre 39 ----
  static const Color plateformeTraversable = Color(0xFF7A6A4A);
  static const Color porte = Color(0xFF7A4B22);
  static const Color porteCadre = Color(0xFF4A2C12);
  static const Color porteOuverte = Color(0xFF14141C);
  static const Color serrure = Color(0xFFF2E27A);
  static const Color boss = Color(0xFF9C2B3A);
  static const Color bossEnrage = Color(0xFFE0432F);
  static const Color bossEtourdi = Color(0xFFF2C14E);
  static const Color voile = Color(0xFF05050A);
}
```

### `lib/composants/plateforme.dart` (modifié — fichier complet)

```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flutter/material.dart';

import '../config/palette.dart';

class Plateforme extends PositionComponent {
  Plateforme({
    required Vector2 position,
    required Vector2 size,
    this.traversable = false,
  }) : super(position: position, size: size, anchor: Anchor.topLeft);

  /// Vrai pour les tuiles '=' : on ne les heurte que par le dessus.
  final bool traversable;

  @override
  Future<void> onLoad() async {
    add(
      RectangleComponent(
        size: size.clone(),
        paint: Paint()
          ..color = traversable
              ? Palette.plateformeTraversable
              : Palette.plateforme,
      ),
    );
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }
}
```

### `lib/composants/joueur.dart` (modifié — extraits)

```dart
// ---- À AJOUTER DANS LA CLASSE Joueur --------------------------------

  /// Chevauchement maximal toléré pour se poser sur une plateforme.
  static const double seuilAtterrissage = 14.0;

// ---- À MODIFIER : _resoudreCollision (chapitre 37) ------------------

  void _resoudreCollision(Plateforme plateforme) {
    final moi = toAbsoluteRect();
    final lui = plateforme.toAbsoluteRect();
    final inter = moi.intersect(lui);
    if (inter.width <= 0 || inter.height <= 0) {
      return;
    }

    if (plateforme.traversable) {           // AJOUT DU CHAPITRE 39
      _resoudreTraversable(moi, lui, inter);
      return;
    }

    // ... le code du chapitre 37, inchangé
  }

// ---- À AJOUTER : la résolution des plateformes traversables ---------

  void _resoudreTraversable(Rect moi, Rect lui, Rect inter) {
    if (velocite.y <= 0) {
      return;                       // il monte : il traverse
    }
    if (inter.width < inter.height) {
      return;                       // contact latéral : il traverse
    }
    if (moi.center.dy >= lui.center.dy) {
      return;                       // il vient d'en dessous : il traverse
    }
    if (inter.height > seuilAtterrissage) {
      return;                       // trop enfoncé : trop tard pour se poser
    }

    position.y -= inter.height;
    velocite.y = 0;
    auSol = true;
  }
```

### `lib/composants/ennemi.dart` (modifié — extraits)

```dart
// ---- À AJOUTER DANS LA CLASSE Ennemi --------------------------------

  /// Le héros, ou null pendant une transition de niveau.
  Joueur? get cible => game.joueur;

  static const double seuilAtterrissage = 14.0;

// ---- À MODIFIER : onLoad (difficulté, chapitre 39) ------------------

  @override
  Future<void> onLoad() async {
    final ReglagesDifficulte reglages = game.reglages;   // AJOUT
    initialiserSante(_pvDepart * reglages.pvEnnemis);    // MODIFIÉ
    degatsContact *= reglages.degatsEnnemis;             // AJOUT
    ancre = position.clone();
    // ... le reste du chapitre 37, inchangé
  }

// ---- À MODIFIER : les capteurs --------------------------------------

  double get distanceAuJoueur {
    final Joueur? heros = cible;
    if (heros == null || !heros.isMounted) {
      return double.infinity;
    }
    return (heros.absoluteCenter - absoluteCenter).length;
  }

  bool voitLeJoueur() {
    final Joueur? heros = cible;
    if (heros == null || !heros.isMounted || !heros.estVivant) {
      return false;
    }
    final vers = heros.absoluteCenter - absoluteCenter;
    // ... le reste du chapitre 37, inchangé
  }

  bool murDevant() {
    final origine = absoluteCenter + Vector2(0, size.y * 0.25);
    final resultat = game.collisionDetection.raycast(
      Ray2(origin: origine, direction: Vector2(sens.toDouble(), 0)),
      maxDistance: size.x * 0.5 + 4,
      hitboxFilter: (candidate) {
        final parent = candidate.parent;
        return parent is Plateforme && !parent.traversable;   // MODIFIÉ
      },
    );
    return resultat != null;
  }

// ---- À MODIFIER : _resoudreCollision --------------------------------
// Copiez ici le même bloc `if (plateforme.traversable) { ... }` et la
// même méthode `_resoudreTraversable` que dans `Joueur`.
```

### `lib/composants/gobelin.dart` et `lib/composants/chauvesouris.dart` (modifiés — extraits)

```dart
// ---- gobelin.dart : à modifier dans _poursuivre ---------------------

  void _poursuivre(double dt) {
    final Joueur? heros = cible;
    if (heros == null) {
      etat = EtatEnnemi.retour;
      return;
    }
    final ecartX = heros.absoluteCenter.x - absoluteCenter.x;
    // ... le reste du chapitre 37, inchangé
  }

// ---- chauvesouris.dart : même garde dans les TROIS méthodes ---------
// _poursuivreEnVol, _declencherPique et tirer commencent désormais par :

    final Joueur? heros = cible;
    if (heros == null) {
      etat = EtatEnnemi.retour;
      return;
    }
    final vers = heros.absoluteCenter - absoluteCenter;
```

### `lib/donjon_game.dart` (modifié — extraits)

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import 'dart:async';
// import 'package:flame/experimental.dart';
// import 'composants/boss.dart';
// import 'composants/porte.dart';
// import 'niveaux/niveau.dart';
// import 'niveaux/niveaux_data.dart';

// ---- À AJOUTER DANS LA CLASSE DonjonGame ----------------------------

  Niveau? niveauEnCours;
  Porte? porte;
  Boss? boss;

  int niveauMaxAtteint = 0;
  int viesMax = Constantes.viesDepart;
  bool transitionEnCours = false;

  Difficulte difficulte = Difficulte.normal;
  ReglagesDifficulte get reglages => ReglagesDifficulte.de(difficulte);

  Hud? _hud;
  Hud get hud => _hud!;

  Future<void> preparerHud() async {
    if (_hud == null) {
      final Hud nouveau = Hud();
      _hud = nouveau;
      await camera.viewport.add(nouveau);
    }
    hud.compteurScore.synchroniserImmediatement();
  }

// ---- À REMPLACER : demarrerPartie -----------------------------------

  Future<void> demarrerPartie({Difficulte? mode}) async {
    if (mode != null) {
      difficulte = mode;
    }
    resumeEngine();                 // INDISPENSABLE avant tout `await add`

    score = 0;
    viesMax = reglages.vies;
    vies = viesMax;
    niveauCourant = 0;
    niveauMaxAtteint = 0;
    transitionEnCours = false;
    reinitialiserCombo();

    await chargerNiveau(0);
    await preparerHud();

    changerEtat(GameState.enJeu);
  }

  Future<void> continuerLaPartie() async {
    resumeEngine();
    score = 0;
    viesMax = reglages.vies;
    vies = viesMax;
    reinitialiserCombo();
    await chargerNiveau(niveauMaxAtteint);
    await preparerHud();
    changerEtat(GameState.enJeu);
  }

// ---- À AJOUTER : le chargement de niveau ----------------------------

  Future<void> chargerNiveau(int index) async {
    resumeEngine();

    final int cible = index.clamp(0, Constantes.nombreNiveaux - 1);
    niveauCourant = cible;
    if (cible > niveauMaxAtteint) {
      niveauMaxAtteint = cible;
    }

    await viderLeMonde();

    final Niveau niveau = Niveau.numero(cible);
    await niveau.construireDans(monde);

    niveauEnCours = niveau;
    porte = niveau.porte;
    boss = niveau.boss;
    pointDeReapparition = niveau.apparitionJoueur.clone();
    piecesDuNiveau = niveau.nombreDePieces;
    piecesRamassees = 0;
    reinitialiserCombo();

    final Joueur? heros = niveau.joueur;
    if (heros != null) {
      camera.follow(heros, snap: true);
    }
    bornerLaCamera(niveau);
  }

  Future<void> viderLeMonde() async {
    camera.stop();

    joueur = null;
    boss = null;
    porte = null;
    niveauEnCours = null;

    monde.removeAll(monde.children.toList());
    await Future<void>.delayed(Duration.zero);
  }

  void bornerLaCamera(Niveau niveau) {
    final Vector2 vue = camera.viewport.size / camera.viewfinder.zoom;
    final double largeur = max(niveau.largeurPixels, vue.x);
    final double hauteur = max(niveau.hauteurPixels, vue.y);

    camera.setBounds(
      Rectangle.fromLTRB(0, 0, largeur, hauteur),
      considerViewport: true,
    );
  }

// ---- À AJOUTER : la fin de niveau -----------------------------------

  void terminerNiveau() {
    if (transitionEnCours) {
      return;
    }
    transitionEnCours = true;

    _accorderLesBonus();

    final int suivant = niveauCourant + 1;
    if (suivant >= Constantes.nombreNiveaux) {
      _terminerLAventure();
      return;
    }
    unawaited(_enchainerVers(suivant));
  }

  void _accorderLesBonus() {
    int bonus = Constantes.bonusFinNiveau;
    if (piecesDuNiveau > 0 && piecesRamassees >= piecesDuNiveau) {
      bonus += Constantes.bonusSalleVidee;
    }
    score += bonus;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

  void _terminerLAventure() {
    transitionEnCours = false;
    niveauMaxAtteint = Constantes.nombreNiveaux - 1;
    changerEtat(GameState.victoire);
  }

  Future<void> _enchainerVers(int suivant) async {
    final VoileTransition voile = VoileTransition();
    await camera.viewport.add(voile);

    await voile.fermer();
    await chargerNiveau(suivant);

    final DefinitionNiveau def = NiveauxData.definition(suivant);
    await voile.annoncer('NIVEAU ${suivant + 1}', def.nom, def.soustitre);

    await voile.ouvrir();
    voile.removeFromParent();

    transitionEnCours = false;
  }

  void bossVaincu(Boss vaincu) {
    porte?.deverrouillerParLeBoss();

    score += Constantes.bonusBoss;
    if (score > meilleurScore) {
      meilleurScore = score;
    }

    afficherTexteFlottant(
      vaincu.absoluteCenter,
      'LE GARDIEN TOMBE',
      Palette.accent,
      taille: 14,
    );
  }

// ---- À MODIFIER : ajouterScore (chapitre 38) ------------------------

  void ajouterScore(int points) {
    if (points <= 0) {
      return;
    }
    score += (points * multiplicateur * reglages.bonusScore).round();  // MODIFIÉ
    combo++;
    _tempsRestantCombo = dureeCombo;
    if (score > meilleurScore) {
      meilleurScore = score;
    }
  }

// ---- À MODIFIER : perdreUneVie --------------------------------------

  void perdreUneVie() {
    if (transitionEnCours) {
      return;
    }
    vies--;
    reinitialiserCombo();
    _hud?.barreDeVie.secouer();

    if (vies <= 0) {
      vies = 0;
      changerEtat(GameState.gameOver);
      return;
    }
    joueur?.reapparaitre(pointDeReapparition);
  }

// ---- À AJOUTER : la progression (utilisée au chapitre 40) -----------

  Map<String, dynamic> etatDeProgression() => <String, dynamic>{
        'meilleurScore': meilleurScore,
        'niveauMaxAtteint': niveauMaxAtteint,
        'difficulte': difficulte.name,
      };

  void appliquerProgression(Map<String, dynamic> donnees) {
    meilleurScore = (donnees['meilleurScore'] as int?) ?? 0;
    niveauMaxAtteint = ((donnees['niveauMaxAtteint'] as int?) ?? 0)
        .clamp(0, Constantes.nombreNiveaux - 1);

    final String? nomDuMode = donnees['difficulte'] as String?;
    difficulte = Difficulte.values.firstWhere(
      (Difficulte d) => d.name == nomDuMode,
      orElse: () => Difficulte.normal,
    );
  }
```

### `lib/hud/hud.dart` (modifié — ajouts)

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import 'dart:async';
// import 'package:flame/effects.dart';
// import 'package:flame/text.dart';
// import '../config/constantes.dart';
// import '../config/palette.dart';

// ---- À AJOUTER DANS LA CLASSE Hud -----------------------------------

  late final BarreBoss barreBoss;

  // dans onLoad, après la création des six éléments du chapitre 38 :
  //   barreBoss = BarreBoss();
  //   await addAll([... , barreBoss]);

// ---- À AJOUTER À LA FIN DU FICHIER ----------------------------------

/// Le rideau noir des transitions, plus le panneau d'annonce.
class VoileTransition extends PositionComponent
    with HasGameReference<DonjonGame> {
  VoileTransition() : super(priority: 1000, anchor: Anchor.topLeft);

  late final RectangleComponent _rideau;
  late final TextComponent _titre;
  late final TextComponent _nom;
  late final TextComponent _soustitre;

  @override
  Future<void> onLoad() async {
    size = game.size.clone();

    _rideau = RectangleComponent(
      size: size.clone(),
      paint: Paint()..color = Palette.voile,
    )..opacity = 0;

    _titre = TextComponent(
      text: '',
      anchor: Anchor.center,
      textRenderer: TextPaint(
        style: const TextStyle(
          fontSize: 15,
          color: Palette.accent,
          fontWeight: FontWeight.bold,
          letterSpacing: 4,
        ),
      ),
    );
    _nom = TextComponent(
      text: '',
      anchor: Anchor.center,
      textRenderer: TextPaint(
        style: const TextStyle(
          fontSize: 30,
          color: Palette.texte,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
    _soustitre = TextComponent(
      text: '',
      anchor: Anchor.center,
      textRenderer: TextPaint(
        style: const TextStyle(fontSize: 13, color: Color(0xFF9C948A)),
      ),
    );

    await addAll(<Component>[_rideau, _titre, _nom, _soustitre]);
    _recentrer();
  }

  @override
  void onGameResize(Vector2 taille) {
    super.onGameResize(taille);
    size = taille.clone();
    if (isLoaded) {
      _rideau.size = taille.clone();
      _recentrer();
    }
  }

  void _recentrer() {
    _titre.position = Vector2(size.x / 2, size.y / 2 - 42);
    _nom.position = Vector2(size.x / 2, size.y / 2 - 4);
    _soustitre.position = Vector2(size.x / 2, size.y / 2 + 32);
  }

  Future<void> fermer() => _fondre(1.0, Curves.easeIn);

  Future<void> ouvrir() => _fondre(0.0, Curves.easeOut);

  Future<void> _fondre(double cible, Curve courbe) {
    final Completer<void> fini = Completer<void>();
    _rideau.add(
      OpacityEffect.to(
        cible,
        EffectController(
          duration: Constantes.dureeFonduTransition,
          curve: courbe,
        ),
        onComplete: fini.complete,
      ),
    );
    return fini.future;
  }

  Future<void> annoncer(String titre, String nom, String soustitre) async {
    _titre.text = titre;
    _nom.text = nom;
    _soustitre.text = soustitre;
    _recentrer();

    _nom.add(
      ScaleEffect.by(
        Vector2.all(1.08),
        EffectController(duration: 0.35, alternate: true),
      ),
    );

    await Future<void>.delayed(
      Duration(
        milliseconds: (Constantes.dureePanneauTransition * 1000).round(),
      ),
    );

    _titre.text = '';
    _nom.text = '';
    _soustitre.text = '';
  }
}
```

### `lib/hud/barre_de_vie.dart` (modifié — ajout)

```dart
// ---- IMPORTS À AJOUTER ----------------------------------------------
// import '../composants/boss.dart';
// import '../config/palette.dart';

// ---- À AJOUTER À LA FIN DU FICHIER ----------------------------------
// La classe `BarreBoss` figure intégralement en section 39.25.
// Elle se colle telle quelle à la suite de `BarreEnergie`.
```

### `lib/hud/compteur_score.dart` (modifié — extrait)

```dart
// ---- À MODIFIER DANS CompteurVies -----------------------------------

  static const int maxCoeurs = 5;

  CompteurVies({EdgeInsets? margin})
      : super(
          margin: margin,
          size: Vector2(maxCoeurs * (tailleCoeur + ecart), tailleCoeur),
          anchor: Anchor.topLeft,
        );

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    for (int i = 0; i < game.viesMax; i++) {         // MODIFIÉ
      final double x = i * (tailleCoeur + ecart);
      _dessinerCoeur(canvas, x, 0, tailleCoeur,
          i < game.vies ? _plein : _vide);
    }
  }
```

### `lib/ecrans/menu_principal.dart` (modifié — extrait)

```dart
// ---- IMPORT À AJOUTER -----------------------------------------------
// import '../config/constantes.dart';   // Difficulte, ReglagesDifficulte

// ---- À REMPLACER : le bouton « Jouer » unique -----------------------

            const Text(
              'Choisissez votre difficulté',
              style: TextStyle(color: Palette.texte, fontSize: 14),
            ),
            const SizedBox(height: 12),
            Wrap(
              spacing: 10,
              children: Difficulte.values.map((Difficulte mode) {
                final ReglagesDifficulte r = ReglagesDifficulte.de(mode);
                return ElevatedButton(
                  onPressed: () => game.demarrerPartie(mode: mode),
                  style: ElevatedButton.styleFrom(
                    backgroundColor:
                        game.difficulte == mode ? Palette.accent : null,
                  ),
                  child: Text('${r.libelle}  (${r.vies} vies)'),
                );
              }).toList(),
            ),
```

### `lib/main.dart` (modifié — extrait)

```dart
// ---- À AJOUTER DANS overlayBuilderMap -------------------------------

            Overlays.victoire: (context, game) => Container(
                  color: Palette.fond.withValues(alpha: 0.92),
                  child: Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: <Widget>[
                        const Text(
                          'VICTOIRE',
                          style: TextStyle(
                            color: Palette.accent,
                            fontSize: 44,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const SizedBox(height: 10),
                        Text(
                          'Score final : ${game.score}',
                          style: const TextStyle(
                            color: Palette.texte,
                            fontSize: 20,
                          ),
                        ),
                        const SizedBox(height: 20),
                        ElevatedButton(
                          onPressed: game.demarrerPartie,
                          child: const Text('Rejouer'),
                        ),
                      ],
                    ),
                  ),
                ),
```

**Commandes de vérification :**

```text
flutter pub get
dart analyze
flutter run -d chrome
```

**Contrôles :**

```text
Q / A / flèche gauche    aller à gauche
D / flèche droite        aller à droite
Z / W / espace / haut    sauter
E / J                    attaquer
```

---

## 39.35 — Exercices

### Exercice 1 — Une quatrième salle (facile)
Ajoutez à `niveaux_data.dart` une carte `carteNiveau4` de 30 colonnes sur 12 lignes, avec un cadre de murs, un `J`, un `D`, une clé et trois pièces. Ajoutez la `DefinitionNiveau` correspondante et faites en sorte que le jeu compte désormais quatre niveaux.

### Exercice 2 — Un compte rendu de niveau (facile)
Ajoutez à `Niveau` une méthode `String resume()` qui renvoie une ligne du type
`Les caves : 40x14, 25 plateformes, 4 ennemis, 12 objets, boss: non`.
Appelez-la depuis `chargerNiveau` derrière un `if (kDebugMode)`.

### Exercice 3 — Renforcer le validateur (facile)
Faites lever une `FormatException` si la carte contient plus d'un `B`, et une autre si elle contient à la fois un `B` et un `k` (une salle de boss ne demande pas de clé).

### Exercice 4 — Descendre volontairement d'une plateforme (intermédiaire)
Ajoutez au `Joueur` la possibilité de traverser une plateforme `=` vers le bas en maintenant la flèche du bas. Indice : un champ `double _tempsIgnorePlateformes` remis à 0,3 s à l'appui, décrémenté dans `update`, et testé en tête de `_resoudreTraversable`.

### Exercice 5 — Un caractère de pointes (intermédiaire)
Ajoutez à la légende le caractère `^` : un piège fixe qui inflige 20 dégâts au contact. Créez `lib/composants/pointes.dart` sur le modèle de `Porte` (hitbox passive), instanciez-le dans `Niveau._construireEntites`, et posez-en trois dans le niveau 2.

### Exercice 6 — Fusionner aussi verticalement (intermédiaire)
La fusion actuelle est horizontale. Écrivez une seconde passe qui fusionne les colonnes de murs restées seules — par exemple les bords gauche et droit d'un niveau, aujourd'hui découpés en une plateforme par ligne. Comparez le nombre de plateformes avant et après sur le niveau 2.

### Exercice 7 — Un compte à rebours par niveau (intermédiaire)
Ajoutez à `DefinitionNiveau` un champ `int secondes` et à `DonjonGame` un compte à rebours affiché dans le HUD. À zéro, le joueur perd une vie et réapparaît. Le bonus de fin de niveau devient proportionnel au temps restant.

### Exercice 8 — Invoquer sur une tuile libre (difficile)
Le boss invoque aujourd'hui à une position calculée sans vérifier le décor. Ajoutez à `Niveau` une méthode `bool tuileLibre(int c, int l)` et à `DonjonGame` un accès au niveau courant, puis faites choisir au boss la tuile vide la plus proche de sa position idéale.

### Exercice 9 — Une quatrième phase de boss (difficile)
Ajoutez `PhaseBoss.pluie` : le boss saute au plafond et fait tomber huit projectiles verticaux répartis sur toute la largeur de l'arène, puis retombe. Insérez-la dans le cycle **uniquement** quand le boss est enragé.

### Exercice 10 — Un éditeur de niveau minimal (difficile)
Écrivez un écran Flutter séparé qui affiche la carte du niveau courant dans un `GridView`, permet de changer le caractère d'une case au clic parmi la légende, valide la carte en direct avec `Niveau.valider()` en affichant l'erreur, et recopie la `List<String>` résultante dans le presse-papier.

---

## 39.36 — Corrections des exercices

### Correction 1

```dart
// lib/niveaux/niveaux_data.dart
  static const List<String> carteNiveau4 = <String>[
    '##############################',
    '#............................#',
    '#....o.o.o...................#',
    '#...========.................#',
    '#............................#',
    '#....................o.......#',
    '#..................=====.....#',
    '#............................#',
    '#.......g...........g........#',
    '#............................#',
    '#.J...............k........D.#',
    '##############################',
  ];

  static const List<DefinitionNiveau> niveaux = <DefinitionNiveau>[
    // ... les trois précédentes
    DefinitionNiveau(
      nom: 'Le corridor',
      soustitre: 'Une dernière porte avant la sortie.',
      carte: carteNiveau4,
    ),
  ];
```

```dart
// lib/config/constantes.dart
  static const int nombreNiveaux = 4;    // était 3
```

**Explication :** les douze lignes font toutes exactement 30 caractères — c'est la première chose que `valider()` vérifiera. `Constantes.nombreNiveaux` doit suivre, car c'est lui qui décide quand `terminerNiveau()` déclenche la victoire. Un désaccord entre `nombreNiveaux` et `NiveauxData.niveaux.length` produirait soit un niveau inatteignable, soit une victoire prématurée : c'est exactement le genre d'incohérence que l'exercice 2 rend visible.

### Correction 2

```dart
// lib/niveaux/niveau.dart
  String resume() {
    analyser();
    final int solides = plateformes.where((p) => !p.traversable).length;
    return '${definition.nom} : ${colonnes}x$lignes, '
        '${plateformes.length} plateformes ($solides solides), '
        '${ennemis.length} ennemis, ${collectibles.length} objets, '
        'boss: ${contientBoss ? "oui" : "non"}';
  }
```

```dart
// lib/donjon_game.dart, dans chargerNiveau, après construireDans
    if (kDebugMode) {
      // ignore: avoid_print
      print(niveau.resume());
    }
```

**Résultat :**

```text
Les caves : 40x14, 25 plateformes (22 solides), 4 ennemis, 12 objets, boss: non
```

**Explication :** `resume()` commence par `analyser()`, qui est idempotente : la méthode est donc appelable avant ou après la construction, sans risque de double fabrication. `kDebugMode` (de `package:flutter/foundation.dart`) est une constante de compilation : en mode release, le compilateur supprime purement et simplement le bloc. C'est la bonne façon de laisser des traces de débogage dans un jeu.

### Correction 3

```dart
// lib/niveaux/niveau.dart, dans valider(), en comptant aussi B et k
    int nombreDeB = 0;
    int nombreDeCles = 0;
    // ... dans la double boucle :
    //   if (caractere == NiveauxData.boss) nombreDeB++;
    //   if (caractere == NiveauxData.cle) nombreDeCles++;

    if (nombreDeB > 1) {
      throw FormatException(
        'Niveau "${definition.nom}" : un seul boss autorisé, trouvé $nombreDeB.',
      );
    }
    if (nombreDeB == 1 && nombreDeCles > 0) {
      throw FormatException(
        'Niveau "${definition.nom}" : une salle de boss ne doit pas '
        'contenir de clé (trouvé $nombreDeCles).',
      );
    }
```

**Explication :** la première règle protège le HUD et la porte, qui supposent tous deux un boss unique. La seconde est une règle de **conception**, pas de syntaxe : elle interdit un niveau jouable mais incohérent, où le joueur pourrait sortir sans affronter le gardien. Un validateur utile encode les règles du jeu, pas seulement celles du format.

### Correction 4

```dart
// lib/composants/joueur.dart
  double _tempsIgnorePlateformes = 0;

  // dans onKeyEvent, sur KeyDownEvent :
      if (touche == LogicalKeyboardKey.arrowDown ||
          touche == LogicalKeyboardKey.keyS) {
        descendre();
      }

  void descendre() {
    if (!auSol) {
      return;                       // on ne descend que depuis un appui
    }
    _tempsIgnorePlateformes = 0.30;
    position.y += 2;                // décollage : on sort de la hitbox
    auSol = false;
  }

  // dans update, avec les autres minuteries :
    if (_tempsIgnorePlateformes > 0) {
      _tempsIgnorePlateformes -= dt;
    }

  // en TÊTE de _resoudreTraversable :
    if (_tempsIgnorePlateformes > 0) {
      return;
    }
```

**Explication :** trois détails font la différence entre un mécanisme agréable et un mécanisme cassé. Le `if (!auSol) return;` empêche de déclencher la descente en plein saut. Le `position.y += 2` décolle le joueur de la plateforme : sans lui, le chevauchement reste nul et la chute ne démarre pas. Enfin, 0,30 s est le temps qu'il faut pour traverser une plateforme de 10 px d'épaisseur ; un délai plus long ferait traverser aussi la plateforme du dessous.

### Correction 5

```dart
// lib/composants/pointes.dart
class Pointes extends PositionComponent
    with HasGameReference<DonjonGame>, CollisionCallbacks {
  Pointes({required Vector2 position})
      : super(position: position, size: Vector2(32, 12),
              anchor: Anchor.topLeft);

  static const double degats = 20.0;

  @override
  Future<void> onLoad() async {
    add(RectangleHitbox(collisionType: CollisionType.passive));
  }

  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    super.onCollision(points, other);
    if (other is Joueur) {
      other.subirDegats(degats, direction: Vector2(0, -1));
    }
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    final Paint p = Paint()..color = const Color(0xFFB9BFCE);
    for (int i = 0; i < 4; i++) {
      final double x = i * 8.0;
      canvas.drawPath(
        Path()
          ..moveTo(x, size.y)
          ..lineTo(x + 4, 0)
          ..lineTo(x + 8, size.y)
          ..close(),
        p,
      );
    }
  }
}
```

```dart
// lib/niveaux/niveaux_data.dart
  static const String pointes = '^';
  // ... et l'ajouter à caracteresConnus

// lib/niveaux/niveau.dart, dans _construireEntites :
          case NiveauxData.pointes:
            collectibles.add(
              Pointes(position: Vector2(c * t, (l + 1) * t - 12)),
            );
            break;
```

**Explication :** on utilise `onCollision` et non `onCollisionStart`, parce qu'un joueur immobile sur des pointes doit continuer de souffrir. C'est sans danger : `Joueur.subirDegats` refuse les coups pendant l'invincibilité temporaire (chapitre 37), qui sert ici de cadence naturelle. La direction `Vector2(0, -1)` projette le joueur vers le haut, ce qui l'aide à se dégager — un piège qui bloque le joueur dedans est un piège raté.

### Correction 6

```dart
// lib/niveaux/niveau.dart — deuxième passe, sur les murs restés seuls
  void _fusionnerVerticalement() {
    final List<Plateforme> isolees = plateformes
        .where((p) => !p.traversable && p.size.x == t)
        .toList();

    for (final Plateforme haute in List<Plateforme>.of(isolees)) {
      if (!plateformes.contains(haute)) {
        continue;
      }
      Plateforme courante = haute;
      Plateforme? dessous = _plateformeEn(courante.position.x,
          courante.position.y + courante.size.y);

      while (dessous != null && dessous.size.x == t) {
        courante.size.y += dessous.size.y;
        plateformes.remove(dessous);
        dessous = _plateformeEn(
            courante.position.x, courante.position.y + courante.size.y);
      }
    }
  }

  Plateforme? _plateformeEn(double x, double y) {
    for (final Plateforme p in plateformes) {
      if (!p.traversable && p.position.x == x && p.position.y == y) {
        return p;
      }
    }
    return null;
  }
```

**Résultat sur le niveau 2 :**

```text
avant : 34 plateformes solides
après : 12 plateformes solides   (les deux bords de 16 tuiles fusionnent)
```

**Explication :** on n'agrandit que des plateformes de largeur exactement une tuile, pour ne jamais fusionner deux bandes de largeurs différentes — le rectangle résultant couvrirait des cases vides. `List<Plateforme>.of(isolees)` fait une copie : on modifie `plateformes` pendant le parcours, ce qui lèverait une `ConcurrentModificationError` sans copie (chapitre 06). La comparaison de `double` par `==` est ici sûre parce que toutes les valeurs sont des multiples exacts de 32 calculés de la même façon.

### Correction 7

```dart
// lib/niveaux/niveaux_data.dart
  const DefinitionNiveau({ /* ... */ this.secondes = 120 });
  final int secondes;
```

```dart
// lib/donjon_game.dart
  double tempsRestant = 0;

  // dans chargerNiveau :
    tempsRestant = NiveauxData.definition(cible).secondes.toDouble();

  // dans update :
    if (etat == GameState.enJeu && !transitionEnCours && tempsRestant > 0) {
      tempsRestant -= dt;
      if (tempsRestant <= 0) {
        tempsRestant = 0;
        joueur?.subirDegats(Constantes.pvJoueurMax);
      }
    }

  // dans _accorderLesBonus :
    bonus += tempsRestant.floor() * 5;
```

**Explication :** on ne rappelle pas `perdreUneVie()` directement : infliger les dégâts laisse le joueur mourir par le chemin normal — animation de mort, délai, puis `perdreUneVie()` par `Joueur.update`. Toute mort passe ainsi par le même code, ce qui évite les états incohérents. La garde `!transitionEnCours` empêche le compte à rebours de courir pendant le fondu au noir, où le joueur ne contrôle plus rien.

### Correction 8

```dart
// lib/niveaux/niveau.dart
  bool tuileLibre(int colonne, int ligne) {
    if (ligne < 0 || ligne >= lignes || colonne < 0 || colonne >= colonnes) {
      return false;
    }
    final String ch = carte[ligne][colonne];
    return ch != NiveauxData.mur && ch != NiveauxData.plateforme;
  }

  static int colonneDe(double x) => (x / t).floor();
  static int ligneDe(double y) => (y / t).floor();
```

```dart
// lib/composants/boss.dart
  Vector2 _pointDInvocation() {
    final Niveau? niveau = game.niveauEnCours;
    final double ecart = Constantes.tailleTuile * (2 + _sbiresInvoques);
    final double vise =
        absoluteCenter.x + (_sbiresInvoques.isEven ? -ecart : ecart);
    final double y = absoluteCenter.y - Constantes.tailleTuile;

    if (niveau == null) {
      return Vector2(vise - 12, y);
    }

    final int ligne = Niveau.ligneDe(y);
    final int colonneVisee = Niveau.colonneDe(vise);

    // On s'écarte de la position idéale, un pas à la fois, des deux côtés.
    for (int d = 0; d <= 6; d++) {
      for (final int c in <int>[colonneVisee - d, colonneVisee + d]) {
        if (niveau.tuileLibre(c, ligne)) {
          return Vector2(c * Niveau.t + 4, y);
        }
      }
    }
    return Vector2(vise - 12, y);
  }
```

**Explication :** la recherche « en spirale » à une dimension — d = 0, puis 1 à gauche et à droite, puis 2… — trouve toujours la tuile libre la plus proche de la position idéale, ce qui garde l'invocation lisible : le gobelin apparaît là où le joueur regarde. Le repli final garantit qu'aucun sbire ne manque à l'appel même si toute la ligne est pleine ; mieux vaut un gobelin mal placé qu'une phase de boss qui ne fait rien.

### Correction 9

```dart
// lib/composants/boss.dart
enum PhaseBoss { sommeil, charge, eventail, invocation, pluie, etourdi, mort }

  static const List<PhaseBoss> cycleEnrage = <PhaseBoss>[
    PhaseBoss.charge,
    PhaseBoss.pluie,
    PhaseBoss.eventail,
    PhaseBoss.invocation,
  ];

  void _phaseSuivante() {
    final List<PhaseBoss> liste = enrage ? cycleEnrage : cycle;
    _entrerDans(liste[_indiceCycle % liste.length]);
    _indiceCycle++;
  }

  // dans _entrerDans :
    } else if (nouvelle == PhaseBoss.pluie) {
      _minuterie = 1.6;
      _cadence = 0.25;
      velocite.y = -420;             // le boss saute
    }

  void _faireTomberLaPluie() {
    if (_cadence <= 0 && _sbiresInvoques < 8) {
      final Niveau? niveau = game.niveauEnCours;
      final double largeur = niveau?.largeurPixels ?? 1280;
      final double x = 40 + (largeur - 80) * (_sbiresInvoques / 7);

      game.monde.add(
        Projectile(
          position: Vector2(x, 32),
          direction: Vector2(0, 1),
          vitesse: Constantes.vitesseProjectile,
        ),
      );
      _sbiresInvoques++;
      _cadence = 0.12;
    }
    if (_minuterie <= 0) {
      _phaseSuivante();
    }
  }
```

**Explication :** on réutilise `_sbiresInvoques` comme compteur générique de la phase courante, puisqu'il est remis à zéro à chaque `_entrerDans`. La répartition `x = 40 + (largeur - 80) * (i / 7)` étale exactement huit projectiles d'un bord à l'autre — même formule que l'éventail, avec `i / (n - 1)`. Le passage par une seconde liste de cycle plutôt que par un `if` dans la phase garde la machine à états lisible : le rythme du combat reste **une donnée**.

### Correction 10

```dart
// lib/ecrans/editeur_niveau.dart (esquisse)
class EditeurNiveau extends StatefulWidget {
  const EditeurNiveau({super.key, required this.depart});
  final List<String> depart;

  @override
  State<EditeurNiveau> createState() => _EditeurNiveauState();
}

class _EditeurNiveauState extends State<EditeurNiveau> {
  late List<List<String>> _grille;
  String _pinceau = NiveauxData.mur;
  String? _erreur;

  @override
  void initState() {
    super.initState();
    _grille = widget.depart.map((l) => l.split('')).toList();
  }

  List<String> get _carte => _grille.map((l) => l.join()).toList();

  void _valider() {
    try {
      Niveau(DefinitionNiveau(
        nom: 'Brouillon',
        soustitre: '',
        carte: _carte,
      )).valider();
      setState(() => _erreur = null);
    } on FormatException catch (e) {
      setState(() => _erreur = e.message);
    }
  }

  @override
  Widget build(BuildContext context) {
    final int colonnes = _grille.first.length;
    return Scaffold(
      appBar: AppBar(
        title: Text(_erreur ?? 'Carte valide'),
        backgroundColor: _erreur == null ? Colors.green : Colors.red,
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.copy),
            onPressed: () => Clipboard.setData(
              ClipboardData(
                text: _carte.map((l) => "  '$l',").join('\n'),
              ),
            ),
          ),
        ],
      ),
      body: Column(
        children: <Widget>[
          Wrap(
            children: NiveauxData.caracteresConnus
                .map((String ch) => ChoiceChip(
                      label: Text(ch),
                      selected: _pinceau == ch,
                      onSelected: (_) => setState(() => _pinceau = ch),
                    ))
                .toList(),
          ),
          Expanded(
            child: GridView.builder(
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: colonnes,
              ),
              itemCount: colonnes * _grille.length,
              itemBuilder: (BuildContext context, int i) {
                final int l = i ~/ colonnes;
                final int c = i % colonnes;
                return GestureDetector(
                  onTap: () {
                    setState(() => _grille[l][c] = _pinceau);
                    _valider();
                  },
                  child: Container(
                    margin: const EdgeInsets.all(0.5),
                    color: const Color(0xFF22222C),
                    child: Center(
                      child: Text(
                        _grille[l][c],
                        style: const TextStyle(color: Colors.white70),
                      ),
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** l'éditeur travaille sur une `List<List<String>>` mutable et ne reconstruit la `List<String>` immuable qu'à la demande — c'est la séparation entre le modèle de travail et le format de stockage. `Niveau(...).valider()` est réutilisé tel quel : la validation écrite en section 39.7 sert dans le jeu **et** dans l'outil, sans une ligne dupliquée. C'est la récompense d'avoir fait de `Niveau` une classe ordinaire, indépendante de Flame. Enfin, le bouton de copie produit directement les lignes au format Dart, prêtes à être collées dans `niveaux_data.dart`.

---

## Et maintenant ?

Le « Donjon de Dart » se joue désormais du début à la fin : trois salles, une clé par porte, un gardien à trois phases, un écran de victoire. Il lui manque encore tout ce qui transforme un prototype jouable en jeu fini — le son, la pause, de vrais écrans de fin, et une mémoire entre deux lancements.

C'est exactement le programme du dernier chapitre de la PARTIE 2C : effets sonores, musique de fond avec `flame_audio`, écran de pause qui ne casse pas les minuteries, écrans de Game Over et de victoire soignés, et sauvegarde persistante du meilleur score et de la progression que la section 39.30 a préparée.

[40-PARTIE-2C—SONS-PAUSE-GAME-OVER-ET-SAUVEGARDE.md](./40-PARTIE-2C—SONS-PAUSE-GAME-OVER-ET-SAUVEGARDE.md)
