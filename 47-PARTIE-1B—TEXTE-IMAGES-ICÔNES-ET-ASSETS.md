# PARTIE 1B — FLUTTER
# CHAPITRE 47 — TEXTE, IMAGES, ICÔNES ET ASSETS

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), chapitre 43 (installation), chapitre 44 (widgets), chapitre 45 (état), chapitre 46 (layouts)
> **Ce que vous saurez faire à la fin :** styliser n'importe quel texte, déclarer et utiliser une police, afficher des icônes, déclarer un dossier d'assets dans `pubspec.yaml` sans erreur d'indentation, afficher une image locale ou distante avec gestion du chargement et des erreurs, maîtriser les sept valeurs de `BoxFit`, et construire une carte de profil complète sans posséder le moindre fichier image.

---

## 47.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- afficher un texte avec le widget `Text` et connaître ses paramètres principaux ;
- construire un `TextStyle` complet ;
- régler la taille, la graisse, l'italique et la couleur d'un texte ;
- utiliser `letterSpacing`, `wordSpacing`, `height` et `decoration` ;
- aligner un texte avec `textAlign` et comprendre pourquoi cela ne marche parfois « pas » ;
- limiter un texte à N lignes avec `maxLines` et le tronquer proprement avec `overflow` ;
- expliquer le rôle de `softWrap` ;
- écrire un texte multi-styles avec `Text.rich` et `TextSpan` ;
- mettre en valeur un mot au milieu d'une phrase ;
- rendre un texte sélectionnable avec `SelectableText` ;
- lire et utiliser les quinze styles de `Theme.of(context).textTheme` ;
- expliquer pourquoi on ne code jamais les tailles de police en dur ;
- trouver une police libre de droits et lire sa licence ;
- déclarer une police dans `pubspec.yaml` avec plusieurs graisses et variantes ;
- utiliser le package `google_fonts` sans télécharger un seul fichier ;
- afficher une icône avec `Icon` et le catalogue `Icons` ;
- régler la taille et la couleur d'une icône ;
- créer un bouton-icône avec `IconButton` et ses variantes Material 3 ;
- distinguer les icônes Material des icônes Cupertino ;
- déclarer un dossier d'assets dans `pubspec.yaml` ;
- diagnostiquer une erreur d'indentation YAML ;
- afficher une image locale avec `Image.asset` ;
- organiser les variantes de résolution `2.0x` et `3.0x` ;
- afficher une image distante avec `Image.network` ;
- gérer le chargement avec `loadingBuilder` et l'échec avec `errorBuilder` ;
- mettre une image réseau en cache avec `cached_network_image` ;
- choisir la bonne valeur de `BoxFit` parmi les sept possibles ;
- arrondir les coins d'une image avec `ClipRRect` ;
- afficher un avatar circulaire avec `CircleAvatar` ;
- afficher une image depuis des octets (`Image.memory`) ou un fichier (`Image.file`) ;
- poser une image en fond d'un `Container` avec `DecorationImage` ;
- assombrir, éclaircir ou teinter une image avec `Opacity` et `ColorFiltered` ;
- construire une interface visuellement riche **sans aucun fichier image** ;
- assembler une carte de profil complète.

---

## 47.0.1 — Avertissement sur la progression

Ce chapitre suppose acquis les chapitres 43 à 46. Nous réutiliserons sans les réexpliquer :

```text
- MaterialApp, Scaffold, AppBar          -> chapitre 44
- StatelessWidget, StatefulWidget        -> chapitre 45
- Row, Column, Container, Padding        -> chapitre 46
- Expanded, SizedBox, Stack              -> chapitre 46
```

Et nous laisserons volontairement de côté :

```text
- ListView et GridView                   -> chapitre 48
- les boutons de formulaire et la saisie -> chapitre 49
- la navigation entre écrans             -> chapitre 50
- ThemeData en profondeur, mode sombre   -> chapitre 51
- le chargement de données distantes     -> chapitre 53
```

**Règle du chapitre :** aucun fichier image ne vous est fourni. Tous les exemples fonctionnent tels quels, soit parce qu'ils n'utilisent que des couleurs, des icônes et des initiales, soit parce qu'ils utilisent l'URL publique `https://picsum.photos`. Quand un exemple parle d'un fichier local, le code réel est donné et signalé comme **non exécutable en l'état**.

---

## 47.1 — `Text` : le widget de base

Vous connaissez déjà `Text` depuis le chapitre 44. Nous allons maintenant l'étudier sérieusement.

`Text` est un `StatelessWidget` qui reçoit une chaîne de caractères en **premier argument positionnel** :

```dart
Text('Bonjour')
```

C'est le seul argument obligatoire. Tous les autres sont nommés.

Voici un premier programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Chapitre 47',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.1 — Le widget Text')),
        body: const Center(
          child: Text('Le joueur Alex entre dans le donjon.'),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.1 — Le widget Text                │  <- AppBar
├──────────────────────────────────────┤
│                                      │
│  Le joueur Alex entre dans le donjon.│  <- centré
│                                      │
└──────────────────────────────────────┘
```

Trois remarques importantes.

**Premièrement**, un `Text` sans style n'est pas « sans style ». Il hérite du style par défaut de l'application, fourni par le thème. Nous verrons cela en 47.11.

**Deuxièmement**, un `Text` prend exactement la place dont il a besoin : ni plus, ni moins. Si le texte est trop long pour la largeur disponible, il passe à la ligne tout seul.

**Troisièmement**, `Text` refuse une valeur `null`. Ceci ne compile pas :

```dart
String? nom;
Text(nom) // ERREUR : The argument type 'String?' can't be assigned to 'String'
```

Le null safety du chapitre 12 s'applique ici comme partout. Écrivez :

```dart
Text(nom ?? 'Inconnu')
```

---

## 47.1.1 — Le tableau des paramètres de `Text`

Voici les paramètres que nous allons couvrir dans ce chapitre :

| Paramètre | Type | Rôle | Section |
| --- | --- | --- | --- |
| `data` | `String` | le texte (positionnel) | 47.1 |
| `style` | `TextStyle?` | toute la mise en forme | 47.2 |
| `textAlign` | `TextAlign?` | alignement horizontal | 47.5 |
| `maxLines` | `int?` | nombre maximal de lignes | 47.6 |
| `overflow` | `TextOverflow?` | que faire du texte en trop | 47.6 |
| `softWrap` | `bool?` | autoriser le retour à la ligne | 47.7 |
| `semanticsLabel` | `String?` | texte lu par les lecteurs d'écran | 47.1.2 |
| `textScaler` | `TextScaler?` | mise à l'échelle système | 47.3.5 |

---

## 47.1.2 — `semanticsLabel` : le texte pour l'accessibilité

Un cas concret. Vous affichez un prix abrégé :

```dart
const Text('1,2 k', semanticsLabel: 'mille deux cents pièces d\'or')
```

L'écran affiche `1,2 k`. Le lecteur d'écran d'un utilisateur malvoyant, lui, prononce « mille deux cents pièces d'or ».

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.1.2 — semanticsLabel')),
        body: const Center(
          child: Text(
            '1,2 k',
            semanticsLabel: 'mille deux cents pieces d or',
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.1.2 — semanticsLabel              │
├──────────────────────────────────────┤
│                                      │
│                1,2 k                 │
│                                      │
└──────────────────────────────────────┘

Lecteur d'écran : « mille deux cents pieces d or »
```

> Prenez l'habitude de renseigner `semanticsLabel` dès qu'un texte est abrégé, symbolique ou purement visuel. Cela coûte une ligne et rend votre application utilisable par tout le monde.

---

## 47.2 — `TextStyle`

Toute la mise en forme d'un texte passe par un seul objet : `TextStyle`.

```dart
Text(
  'Boss vaincu',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.red,
  ),
)
```

`TextStyle` est une classe ordinaire (chapitre 08) avec un constructeur à paramètres nommés (chapitre 09). Tous ses paramètres sont optionnels et nullables.

Voici la carte du territoire :

```text
                         TextStyle
                             │
   ┌──────────────┬──────────┴──────────┬──────────────────┐
   │              │                     │                  │
 COULEUR       TAILLE               ESPACEMENT        DÉCORATION
 color         fontSize             letterSpacing     decoration
 backgroundColor  fontWeight        wordSpacing       decorationColor
                fontStyle           height            decorationStyle
                fontFamily                            decorationThickness
                                                      shadows
```

Un programme complet qui met tout cela en scène :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.2 — TextStyle')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('Texte par defaut'),
              SizedBox(height: 12),
              Text(
                'Texte stylise',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                  color: Colors.deepPurple,
                  letterSpacing: 1.5,
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.2 — TextStyle                     │
├──────────────────────────────────────┤
│ Texte par defaut                     │  <- 14 px environ, noir
│                                      │
│ T e x t e   s t y l i s e            │  <- 24 px, gras, violet, espacé
└──────────────────────────────────────┘
```

---

## 47.2.1 — `TextStyle` est immuable : `copyWith`

Comme la plupart des classes de Flutter, `TextStyle` est **immuable**. On ne modifie pas un style existant, on en fabrique une copie modifiée avec `copyWith` :

```dart
const base = TextStyle(fontSize: 16, color: Colors.black);
final titre = base.copyWith(fontSize: 28, fontWeight: FontWeight.bold);
```

`titre` est un nouveau `TextStyle` avec `fontSize: 28`, `fontWeight: bold` et `color: Colors.black` (hérité de `base`).

Ceci sera votre outil quotidien avec le thème (section 47.11) :

```dart
Theme.of(context).textTheme.titleLarge?.copyWith(color: Colors.red)
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const TextStyle kBase = TextStyle(fontSize: 16, color: Colors.black87);

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    final TextStyle titre = kBase.copyWith(
      fontSize: 28,
      fontWeight: FontWeight.bold,
    );
    final TextStyle alerte = kBase.copyWith(color: Colors.red);

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.2.1 — copyWith')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text('Inventaire', style: titre),
              const SizedBox(height: 8),
              Text('3 potions, 1 epee', style: kBase),
              const SizedBox(height: 8),
              Text('Sac presque plein', style: alerte),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.2.1 — copyWith                    │
├──────────────────────────────────────┤
│ Inventaire                           │  <- 28 px gras noir
│ 3 potions, 1 epee                    │  <- 16 px noir
│ Sac presque plein                    │  <- 16 px rouge
└──────────────────────────────────────┘
```

> `copyWith` ne modifie jamais l'objet d'origine. `kBase` reste noir et en 16 px après ces deux appels.

---

## 47.3 — Taille, graisse, style, couleur

Les quatre réglages que vous utiliserez dans 90 % des cas.

---

### 47.3.1 — `fontSize`

`fontSize` est un `double`, exprimé en **pixels logiques**. La valeur par défaut d'un `Text` sans style dépend du thème ; elle vaut 14 pour `bodyMedium` en Material 3.

```dart
Text('Petit', style: TextStyle(fontSize: 12))
Text('Normal', style: TextStyle(fontSize: 16))
Text('Grand', style: TextStyle(fontSize: 32))
```

Repères utiles :

| Taille | Usage typique |
| --- | --- |
| 11 – 12 | mention légale, horodatage |
| 14 | corps de texte secondaire |
| 16 | corps de texte principal |
| 20 – 22 | titre de carte, titre de section |
| 28 – 36 | titre d'écran |
| 45 et plus | chiffre mis en avant, score |

---

### 47.3.2 — `fontWeight` : la graisse

`FontWeight` propose neuf valeurs numériques, plus deux alias :

```text
FontWeight.w100  Thin        (le plus fin)
FontWeight.w200  ExtraLight
FontWeight.w300  Light
FontWeight.w400  Regular     == FontWeight.normal
FontWeight.w500  Medium
FontWeight.w600  SemiBold
FontWeight.w700  Bold        == FontWeight.bold
FontWeight.w800  ExtraBold
FontWeight.w900  Black       (le plus gras)
```

**Point crucial :** une graisse ne s'affiche réellement que si la police contient ce fichier. Si vous demandez `w300` à une police qui ne fournit que `w400` et `w700`, le moteur choisit la graisse la plus proche. Vous verrez peut-être exactement le même rendu qu'en `w400`. Ce n'est pas un bug ; c'est une police incomplète. Nous y reviendrons en 47.15.

---

### 47.3.3 — `fontStyle` : romain ou italique

Deux valeurs seulement :

```dart
FontStyle.normal  // droit (par défaut)
FontStyle.italic  // italique
```

Même remarque : si la police n'a pas de fichier italique déclaré, le moteur peut **simuler** l'italique en inclinant les lettres, ou ne rien faire du tout selon la plateforme. Une vraie italique est toujours plus belle qu'une italique simulée.

---

### 47.3.4 — `color` et `backgroundColor`

```dart
TextStyle(
  color: Colors.white,
  backgroundColor: Colors.black,
)
```

`backgroundColor` peint un rectangle **derrière les glyphes uniquement**, pas derrière tout le widget. Pour un fond qui déborde du texte, utilisez un `Container` avec `color` (chapitre 46).

Programme complet qui montre les quatre réglages :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.3 — Taille, graisse, style')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('fontSize 12', style: TextStyle(fontSize: 12)),
              Text('fontSize 20', style: TextStyle(fontSize: 20)),
              Text('fontSize 32', style: TextStyle(fontSize: 32)),
              SizedBox(height: 16),
              Text('w300 Light',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.w300)),
              Text('w400 Regular',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.w400)),
              Text('w700 Bold',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.w700)),
              Text('w900 Black',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.w900)),
              SizedBox(height: 16),
              Text('Italique',
                  style: TextStyle(fontSize: 20, fontStyle: FontStyle.italic)),
              SizedBox(height: 16),
              Text(
                'Couleur et fond',
                style: TextStyle(
                  fontSize: 20,
                  color: Colors.white,
                  backgroundColor: Colors.indigo,
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.3 — Taille, graisse, style        │
├──────────────────────────────────────┤
│ fontSize 12                          │
│ fontSize 20                          │
│ fontSize 32                          │
│                                      │
│ w300 Light                           │
│ w400 Regular                         │
│ w700 Bold                            │
│ w900 Black                           │
│                                      │
│ Italique                             │
│                                      │
│ ███Couleur et fond███                │  <- glyphes blancs sur bandeau indigo
└──────────────────────────────────────┘
```

---

### 47.3.5 — Le texte grossit tout seul : `textScaler`

Sur Android comme sur iOS, l'utilisateur peut agrandir la taille des textes dans les réglages système. Flutter respecte ce choix automatiquement : un `fontSize: 16` peut être rendu en 24 px si l'utilisateur a demandé 150 %.

C'est **voulu** et c'est **bien**. Ne le désactivez pas sans raison sérieuse. En revanche, testez votre interface avec un grossissement fort : c'est là que les débordements apparaissent (voir 47.6).

Si vous devez vraiment figer une taille — cas rare, par exemple un compteur dans un jeu — vous pouvez écrire :

```dart
Text('99', textScaler: TextScaler.noScaling, style: TextStyle(fontSize: 40))
```

> L'ancien paramètre `textScaleFactor` est déprécié. Utilisez `textScaler`.

---

## 47.4 — `letterSpacing`, `height`, `decoration`

Trois réglages qui séparent une interface amateur d'une interface soignée.

---

### 47.4.1 — `letterSpacing` et `wordSpacing`

`letterSpacing` ajoute (ou retire, si négatif) des pixels logiques **entre chaque lettre**. `wordSpacing` fait la même chose **entre les mots**.

```dart
TextStyle(letterSpacing: 2.0)   // lettres aérées
TextStyle(letterSpacing: -0.5)  // lettres resserrées
TextStyle(wordSpacing: 8.0)     // mots très espacés
```

Usage typique : un titre en majuscules devient beaucoup plus lisible avec `letterSpacing: 1.5`. À l'inverse, un très grand titre (40 px et plus) gagne souvent à être resserré avec `letterSpacing: -1`.

---

### 47.4.2 — `height` : l'interligne

`height` est le réglage le plus mal compris de `TextStyle`.

Ce n'est **pas** une hauteur en pixels. C'est un **multiplicateur de `fontSize`** qui définit la hauteur totale de chaque ligne.

```text
fontSize: 16, height: 1.0   ->  hauteur de ligne = 16 px  (très serré)
fontSize: 16, height: 1.5   ->  hauteur de ligne = 24 px  (confortable)
fontSize: 16, height: 2.0   ->  hauteur de ligne = 32 px  (très aéré)
```

Schéma :

```text
height: 1.0                     height: 1.6

┌────────────────────┐          ┌────────────────────┐
│Le joueur avance    │          │Le joueur avance    │
│dans le couloir     │          │                    │
│sombre du donjon    │          │dans le couloir     │
└────────────────────┘          │                    │
   lignes collées               │sombre du donjon    │
                                └────────────────────┘
                                   lignes respirantes
```

Règle pratique : pour un paragraphe, `height: 1.4` à `1.6`. Pour un titre sur une seule ligne, laissez `height` à `null`.

---

### 47.4.3 — `decoration` : souligné, barré, surligné

```dart
TextDecoration.none        // rien (par défaut)
TextDecoration.underline   // souligné
TextDecoration.lineThrough // barré
TextDecoration.overline    // trait au-dessus
```

On peut en combiner plusieurs :

```dart
TextDecoration.combine([
  TextDecoration.underline,
  TextDecoration.lineThrough,
])
```

Et on peut styliser le trait lui-même :

```dart
TextStyle(
  decoration: TextDecoration.underline,
  decorationColor: Colors.red,
  decorationStyle: TextDecorationStyle.wavy,
  decorationThickness: 2.0,
)
```

Les cinq styles de trait :

```text
TextDecorationStyle.solid    ────────────
TextDecorationStyle.double   ════════════
TextDecorationStyle.dotted   ············
TextDecorationStyle.dashed   ─ ─ ─ ─ ─ ─
TextDecorationStyle.wavy     ~~~~~~~~~~~~
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.4 — Espacement et decoration')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('DONJON DE PIERRE',
                  style: TextStyle(fontSize: 18, letterSpacing: 4)),
              SizedBox(height: 4),
              Text('DONJON DE PIERRE', style: TextStyle(fontSize: 18)),
              SizedBox(height: 20),
              Text(
                'Le joueur avance dans le couloir sombre du donjon et '
                'entend un bruit metallique derriere la porte.',
                style: TextStyle(fontSize: 16, height: 1.0),
              ),
              SizedBox(height: 20),
              Text(
                'Le joueur avance dans le couloir sombre du donjon et '
                'entend un bruit metallique derriere la porte.',
                style: TextStyle(fontSize: 16, height: 1.6),
              ),
              SizedBox(height: 20),
              Text('150 pieces',
                  style: TextStyle(
                    fontSize: 18,
                    decoration: TextDecoration.lineThrough,
                    decorationColor: Colors.red,
                    decorationThickness: 2,
                  )),
              Text('90 pieces',
                  style: TextStyle(
                    fontSize: 18,
                    decoration: TextDecoration.underline,
                    decorationStyle: TextDecorationStyle.wavy,
                    decorationColor: Colors.green,
                  )),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.4 — Espacement et decoration      │
├──────────────────────────────────────┤
│ D O N J O N   D E   P I E R R E      │
│ DONJON DE PIERRE                     │
│                                      │
│ Le joueur avance dans le couloir     │
│ sombre du donjon et entend un bruit  │  <- lignes serrées
│ metallique derriere la porte.        │
│                                      │
│ Le joueur avance dans le couloir     │
│                                      │  <- lignes aérées
│ sombre du donjon et entend un bruit  │
│                                      │
│ metallique derriere la porte.        │
│                                      │
│ 150 pieces  (barré rouge)            │
│ 90 pieces (souligné ondulé vert)     │
└──────────────────────────────────────┘
```

> Le cas « ancien prix barré / nouveau prix souligné » est le grand classique des applications de vente. Vous venez de l'écrire.

---

## 47.5 — `textAlign`

`textAlign` définit l'alignement horizontal du texte **à l'intérieur de la largeur du widget `Text`**.

Les valeurs :

| Valeur | Effet |
| --- | --- |
| `TextAlign.left` | collé à gauche |
| `TextAlign.right` | collé à droite |
| `TextAlign.center` | centré |
| `TextAlign.justify` | justifié (bords gauche et droit alignés) |
| `TextAlign.start` | début de ligne selon la langue (gauche en français) |
| `TextAlign.end` | fin de ligne selon la langue (droite en français) |

> Préférez `start` et `end` à `left` et `right` : votre application restera correcte si elle est un jour traduite en arabe ou en hébreu, où l'écriture va de droite à gauche.

---

## 47.5.1 — Le piège numéro un de `textAlign`

Beaucoup de débutants écrivent ceci et concluent que `textAlign` « ne marche pas » :

```dart
Column(
  children: const [
    Text('Titre', textAlign: TextAlign.center),
  ],
)
```

Le texte reste collé à gauche. Pourquoi ?

Parce que `textAlign` aligne le texte **dans la boîte du `Text`**, et que la boîte du `Text` fait exactement la largeur du texte. Aligner un texte au centre d'une boîte qui épouse ce texte ne produit évidemment aucun effet.

```text
CE QUE VOUS CROYEZ                 CE QUI SE PASSE

┌────────────────────────┐         ┌────────────────────────┐
│         Titre          │         │┌─────┐                 │
│                        │         ││Titre│                 │
└────────────────────────┘         │└─────┘                 │
  boîte = tout l'écran             └────────────────────────┘
                                     boîte du Text = 5 lettres
                                     "centré" DANS ces 5 lettres
```

**La solution** : donner de la largeur au `Text`, ou utiliser un widget d'alignement.

Trois façons correctes :

```dart
// 1. Occuper toute la largeur, puis aligner le texte dedans
SizedBox(
  width: double.infinity,
  child: Text('Titre', textAlign: TextAlign.center),
)

// 2. Centrer le widget lui-même (pas son contenu)
Center(child: Text('Titre'))

// 3. Étirer tous les enfants de la Column
Column(
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: const [Text('Titre', textAlign: TextAlign.center)],
)
```

Programme complet qui montre le problème et les solutions :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.5 — textAlign')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Container(
                color: Colors.amber.shade100,
                child: const Text('KO : dans une Column simple',
                    textAlign: TextAlign.center),
              ),
              const SizedBox(height: 12),
              Container(
                color: Colors.lightGreen.shade100,
                width: double.infinity,
                child: const Text('OK : SizedBox infinity',
                    textAlign: TextAlign.center),
              ),
              const SizedBox(height: 12),
              Container(
                color: Colors.lightBlue.shade100,
                child: const Text(
                  'Ce paragraphe est justifie. Les bords gauche et droit '
                  'sont alignes, sauf sur la derniere ligne, ce qui est '
                  'la regle typographique normale.',
                  textAlign: TextAlign.justify,
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.5 — textAlign                     │
├──────────────────────────────────────┤
│ ████ KO : dans une Column simple ████│  <- stretch appliqué, donc centré
│                                      │
│ ████ OK : SizedBox infinity     █████│
│                                      │
│ ████Ce paragraphe est justifie. Les██│
│ ████bords gauche et droit sont██████ │
│ ████alignes, sauf sur la derniere████│
│ ████ligne, ce qui est la regle.██████│
└──────────────────────────────────────┘
```

> Dans cet exemple, la première ligne fonctionne quand même : parce que la `Column` utilise `CrossAxisAlignment.stretch`, qui force chaque enfant à occuper toute la largeur. Retirez cette ligne et le premier texte se recolle à gauche. Faites l'essai : c'est la meilleure façon de comprendre.

---

## 47.6 — `maxLines` et `overflow`

Le problème : un texte trop long déborde et Flutter affiche des rayures jaunes et noires.

```text
┌──────────────────────────────────────┐
│ Un nom de joueur vraiment tres long q│▨▨▨▨▨
│                                      │▨▨▨▨▨  <- overflow
└──────────────────────────────────────┘
```

Ces rayures signifient : « le contenu ne tient pas ». En production, l'utilisateur ne doit jamais les voir.

---

### 47.6.1 — `maxLines`

`maxLines` limite le nombre de lignes affichées :

```dart
Text('Une longue description...', maxLines: 2)
```

Au-delà de 2 lignes, le reste est **coupé net**, en plein milieu d'un mot. Ce n'est pas satisfaisant.

---

### 47.6.2 — `overflow`

`overflow` décide de ce qui arrive au texte en trop :

| Valeur | Effet |
| --- | --- |
| `TextOverflow.clip` | coupe net (comportement par défaut) |
| `TextOverflow.ellipsis` | remplace la fin par `…` |
| `TextOverflow.fade` | fait disparaître la fin en dégradé |
| `TextOverflow.visible` | laisse déborder (les rayures apparaissent) |

Schéma :

```text
Texte source : "Epee legendaire du dragon de glace"
Largeur disponible : 20 caractères

clip     : |Epee legendaire du d|
ellipsis : |Epee legendaire d…  |
fade     : |Epee legendaire du ▒|
visible  : |Epee legendaire du dragon de glace|  -> déborde
```

`maxLines` et `overflow` s'utilisent **ensemble** :

```dart
Text(
  'Epee legendaire du dragon de glace',
  maxLines: 1,
  overflow: TextOverflow.ellipsis,
)
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kTexte =
    'Epee legendaire du dragon de glace forgee dans les montagnes du nord '
    'par un maitre artisan disparu depuis trois siecles.';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.6 — maxLines et overflow')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('1 ligne + clip :',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Text(kTexte, maxLines: 1, overflow: TextOverflow.clip),
              SizedBox(height: 16),
              Text('1 ligne + ellipsis :',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Text(kTexte, maxLines: 1, overflow: TextOverflow.ellipsis),
              SizedBox(height: 16),
              Text('2 lignes + ellipsis :',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Text(kTexte, maxLines: 2, overflow: TextOverflow.ellipsis),
              SizedBox(height: 16),
              Text('1 ligne + fade :',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Text(kTexte, maxLines: 1, overflow: TextOverflow.fade),
              SizedBox(height: 16),
              Text('Sans limite :',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Text(kTexte),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.6 — maxLines et overflow          │
├──────────────────────────────────────┤
│ 1 ligne + clip :                     │
│ Epee legendaire du dragon de glace f │
│                                      │
│ 1 ligne + ellipsis :                 │
│ Epee legendaire du dragon de glace…  │
│                                      │
│ 2 lignes + ellipsis :                │
│ Epee legendaire du dragon de glace   │
│ forgee dans les montagnes du nord p… │
│                                      │
│ 1 ligne + fade :                     │
│ Epee legendaire du dragon de glace ▒ │
│                                      │
│ Sans limite :                        │
│ Epee legendaire du dragon de glace   │
│ forgee dans les montagnes du nord    │
│ par un maitre artisan disparu depuis │
│ trois siecles.                       │
└──────────────────────────────────────┘
```

> Remarquez la constante `kTexte` déclarée **hors** de toute classe, au premier niveau du fichier. C'est parfaitement légal en Dart (chapitre 02) et cela évite de réécrire trois fois la même longue chaîne. La convention de nommage `kNomDeLaConstante` vient du code source de Flutter lui-même.

---

### 47.6.3 — Le cas du `Text` dans une `Row`

Un `Text` long placé dans une `Row` (chapitre 46) déborde **même avec `overflow: ellipsis`**. Pourquoi ? Parce qu'une `Row` donne à ses enfants une largeur illimitée : le `Text` croit disposer d'une place infinie et ne coupe jamais.

La solution est celle du chapitre 46 : `Expanded`.

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    Expanded(
      child: Text(
        'Un nom de joueur vraiment tres long',
        maxLines: 1,
        overflow: TextOverflow.ellipsis,
      ),
    ),
    const Icon(Icons.star),
  ],
)
```

Programme complet, avec le cas cassé et le cas correct :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kNom = 'Alexandra la Vaillante Porteuse de Flamme Eternelle';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.6.3 — Text dans une Row')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('Correct : avec Expanded',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              Container(
                color: Colors.lightGreen.shade100,
                child: Row(
                  children: const [
                    Icon(Icons.person),
                    SizedBox(width: 8),
                    Expanded(
                      child: Text(kNom,
                          maxLines: 1, overflow: TextOverflow.ellipsis),
                    ),
                    Icon(Icons.star, color: Colors.amber),
                  ],
                ),
              ),
              const SizedBox(height: 24),
              const Text('A eviter : sans Expanded, le texte deborde',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              Container(
                color: Colors.red.shade50,
                child: Row(
                  children: const [
                    Icon(Icons.person),
                    SizedBox(width: 8),
                    Flexible(
                      child: Text(kNom,
                          maxLines: 1, overflow: TextOverflow.ellipsis),
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.6.3 — Text dans une Row           │
├──────────────────────────────────────┤
│ Correct : avec Expanded              │
│ [ic] Alexandra la Vaillante Port… * │
│                                      │
│ A eviter : sans Expanded...          │
│ [ic] Alexandra la Vaillante Porteu…  │
└──────────────────────────────────────┘
```

> `Flexible` fonctionne aussi : il laisse l'enfant prendre moins que la place offerte, alors qu'`Expanded` l'oblige à tout prendre. Pour un texte à tronquer, les deux conviennent. Sans l'un des deux, le débordement est garanti.

---

## 47.7 — `softWrap`

`softWrap` répond à une seule question : **le texte a-t-il le droit de passer à la ligne tout seul ?**

```dart
Text('...', softWrap: true)   // par défaut : oui
Text('...', softWrap: false)  // non : une seule ligne, quoi qu'il arrive
```

Avec `softWrap: false`, le texte reste sur une ligne unique et déborde horizontalement — sauf si vous ajoutez `overflow: TextOverflow.ellipsis`, qui reprend alors la main.

```text
softWrap: true (défaut)          softWrap: false

┌──────────────────┐             ┌──────────────────┐
│Le joueur avance  │             │Le joueur avance d│▨▨▨▨
│dans le couloir   │             └──────────────────┘
│sombre            │               une seule ligne
└──────────────────┘
```

Attention à ne pas confondre :

| Réglage | Question à laquelle il répond |
| --- | --- |
| `softWrap` | le texte a-t-il le droit d'aller à la ligne ? |
| `maxLines` | combien de lignes au maximum ? |
| `overflow` | que faire du texte qui ne rentre pas ? |

`softWrap: false` équivaut presque à `maxLines: 1`, avec une différence : un `\n` explicite dans la chaîne crée quand même une nouvelle ligne avec `softWrap: false`, alors que `maxLines: 1` coupe tout.

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kTexte =
    'Le joueur avance dans le couloir sombre et humide du donjon oublie.';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.7 — softWrap')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('softWrap: true (defaut)',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                color: Colors.lightGreen.shade100,
                child: const Text(kTexte, softWrap: true),
              ),
              const SizedBox(height: 24),
              const Text('softWrap: false + ellipsis',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                color: Colors.amber.shade100,
                child: const Text(
                  kTexte,
                  softWrap: false,
                  overflow: TextOverflow.ellipsis,
                ),
              ),
              const SizedBox(height: 24),
              const Text('Avec un saut de ligne explicite',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                color: Colors.lightBlue.shade100,
                child: const Text(
                  'Ligne un\nLigne deux\nLigne trois',
                  softWrap: false,
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.7 — softWrap                      │
├──────────────────────────────────────┤
│ softWrap: true (defaut)              │
│ ███Le joueur avance dans le couloir██│
│ ███sombre et humide du donjon oublie.│
│                                      │
│ softWrap: false + ellipsis           │
│ ███Le joueur avance dans le couloi…██│
│                                      │
│ Avec un saut de ligne explicite      │
│ ███Ligne un██████████████████████████│
│ ███Ligne deux████████████████████████│
│ ███Ligne trois███████████████████████│
└──────────────────────────────────────┘
```

> Les `\n` sont toujours respectés. `softWrap: false` n'interdit que le retour à la ligne **automatique**.

---

## 47.8 — `Text.rich` et `TextSpan`

Un `Text` ordinaire applique **un seul style** à toute la chaîne. Comment écrire une phrase dont un seul mot est en gras et rouge ?

Première idée, mauvaise : découper la phrase en trois `Text` dans une `Row`.

```dart
Row(
  children: const [
    Text('Vous avez perdu '),
    Text('50', style: TextStyle(color: Colors.red)),
    Text(' points de vie.'),
  ],
)
```

Cela « marche » tant que la phrase tient sur une ligne. Dès qu'il faut passer à la ligne, tout casse : une `Row` ne sait pas revenir à la ligne, et la phrase déborde.

La bonne solution s'appelle `Text.rich`.

---

### 47.8.1 — Le principe

`Text.rich` reçoit non pas une `String`, mais un **arbre de fragments** appelés `InlineSpan`. Le type concret que vous utiliserez est `TextSpan`.

```text
Text.rich( TextSpan racine )
              │
              ├─ text  : 'Vous avez perdu '   (style hérité)
              └─ children :
                    ├─ TextSpan('50', style: rouge gras)
                    └─ TextSpan(' points de vie.')
```

Un `TextSpan` possède trois propriétés principales :

| Propriété | Type | Rôle |
| --- | --- | --- |
| `text` | `String?` | le texte de ce fragment |
| `style` | `TextStyle?` | son style |
| `children` | `List<InlineSpan>?` | les fragments suivants |

Règle d'héritage : un enfant **hérite** du style de son parent, puis applique le sien par-dessus. C'est exactement la logique d'un `copyWith` automatique.

---

### 47.8.2 — Premier exemple complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.8 — Text.rich')),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Text.rich(
            TextSpan(
              style: TextStyle(fontSize: 18, color: Colors.black87),
              children: [
                TextSpan(text: 'Vous avez perdu '),
                TextSpan(
                  text: '50',
                  style: TextStyle(
                    color: Colors.red,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                TextSpan(text: ' points de vie face au '),
                TextSpan(
                  text: 'Gobelin des cavernes',
                  style: TextStyle(fontStyle: FontStyle.italic),
                ),
                TextSpan(text: '.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.8 — Text.rich                     │
├──────────────────────────────────────┤
│ Vous avez perdu 50 points de vie     │
│ face au Gobelin des cavernes.        │
│                                      │
│  "50" est rouge et gras              │
│  "Gobelin des cavernes" est italique │
│  la phrase revient à la ligne toute  │
│  seule, comme un vrai paragraphe     │
└──────────────────────────────────────┘
```

Remarquez : le `TextSpan` racine n'a **pas** de `text`, seulement un `style` et des `children`. C'est le montage le plus lisible.

---

### 47.8.3 — L'imbrication et l'héritage des styles

Les `TextSpan` peuvent s'imbriquer sur plusieurs niveaux. Chaque niveau ajoute son style au style hérité.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.8.3 — Imbrication')),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Text.rich(
            TextSpan(
              style: TextStyle(fontSize: 20),
              children: [
                TextSpan(text: 'normal '),
                TextSpan(
                  text: 'bleu ',
                  style: TextStyle(color: Colors.blue),
                  children: [
                    TextSpan(text: 'bleu gras ',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    TextSpan(
                      text: 'bleu gras italique',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontStyle: FontStyle.italic,
                      ),
                    ),
                  ],
                ),
                TextSpan(text: ' et retour au normal.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.8.3 — Imbrication                 │
├──────────────────────────────────────┤
│ normal bleu bleu gras bleu gras      │
│ italique et retour au normal.        │
└──────────────────────────────────────┘

Chaîne d'héritage :
  racine  -> fontSize 20
  "bleu"  -> fontSize 20 + couleur bleue
  enfant  -> fontSize 20 + bleu + gras
  enfant  -> fontSize 20 + bleu + gras + italique
  dernier -> fontSize 20 seulement (il est enfant de la RACINE)
```

> Attention : `' et retour au normal.'` est enfant de la **racine**, pas du span bleu. Il n'hérite donc pas du bleu. Si vous vous trompez de niveau d'imbrication, tout le reste de la phrase change de couleur. C'est l'erreur classique avec `Text.rich`.

---

### 47.8.4 — `WidgetSpan` : glisser un widget dans une phrase

`TextSpan` n'est pas le seul `InlineSpan`. Il existe aussi `WidgetSpan`, qui insère **un widget entier** au fil du texte.

```dart
WidgetSpan(child: Icon(Icons.star, size: 18, color: Colors.amber))
```

C'est ainsi que l'on écrit « Vous avez gagné [icône] 3 étoiles » avec une vraie icône alignée sur la ligne de base du texte.

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.8.4 — WidgetSpan')),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Text.rich(
            TextSpan(
              style: TextStyle(fontSize: 18, height: 1.6),
              children: [
                TextSpan(text: 'Niveau termine. Vous avez gagne '),
                WidgetSpan(
                  alignment: PlaceholderAlignment.middle,
                  child: Icon(Icons.star, size: 20, color: Colors.amber),
                ),
                WidgetSpan(
                  alignment: PlaceholderAlignment.middle,
                  child: Icon(Icons.star, size: 20, color: Colors.amber),
                ),
                WidgetSpan(
                  alignment: PlaceholderAlignment.middle,
                  child: Icon(Icons.star_border, size: 20, color: Colors.grey),
                ),
                TextSpan(text: ' et '),
                TextSpan(
                  text: '250 pieces',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                TextSpan(text: '.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.8.4 — WidgetSpan                  │
├──────────────────────────────────────┤
│ Niveau termine. Vous avez gagne      │
│ [etoile][etoile][etoile vide] et     │
│ 250 pieces.                          │
└──────────────────────────────────────┘
```

`PlaceholderAlignment` propose entre autres `baseline`, `middle`, `top`, `bottom`. Sans ce réglage, l'icône se cale sur le bas de la ligne et paraît décalée vers le bas.

---

## 47.9 — Mettre en forme une partie d'un texte

Voici un cas très fréquent : vous recevez une phrase et un mot à mettre en valeur, et vous devez construire les `TextSpan` **dynamiquement**.

C'est un exercice de manipulation de chaînes (chapitre 02) et de listes (chapitre 06) autant que de Flutter.

---

### 47.9.1 — Surligner un mot recherché

L'algorithme :

```text
1. trouver l'indice du mot dans la phrase (indexOf)
2. si -1 : renvoyer un seul TextSpan avec toute la phrase
3. sinon : trois TextSpan
     - avant  : phrase.substring(0, indice)
     - le mot : phrase.substring(indice, indice + mot.length)  -> stylisé
     - apres  : phrase.substring(indice + mot.length)
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

/// Construit une liste de TextSpan qui met [motCle] en valeur dans [phrase].
List<TextSpan> surligner(String phrase, String motCle) {
  if (motCle.isEmpty) {
    return [TextSpan(text: phrase)];
  }

  final int indice = phrase.toLowerCase().indexOf(motCle.toLowerCase());
  if (indice == -1) {
    return [TextSpan(text: phrase)];
  }

  return [
    TextSpan(text: phrase.substring(0, indice)),
    TextSpan(
      text: phrase.substring(indice, indice + motCle.length),
      style: const TextStyle(
        fontWeight: FontWeight.bold,
        color: Colors.deepOrange,
        backgroundColor: Color(0xFFFFF3E0),
      ),
    ),
    TextSpan(text: phrase.substring(indice + motCle.length)),
  ];
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    const String phrase =
        'Le gobelin garde le coffre du donjon depuis mille ans.';

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.9 — Surligner un mot')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('Recherche : "coffre"',
                  style: TextStyle(color: Colors.grey)),
              const SizedBox(height: 8),
              Text.rich(
                TextSpan(
                  style: const TextStyle(fontSize: 18, height: 1.5),
                  children: surligner(phrase, 'coffre'),
                ),
              ),
              const SizedBox(height: 24),
              const Text('Recherche : "dragon" (absent)',
                  style: TextStyle(color: Colors.grey)),
              const SizedBox(height: 8),
              Text.rich(
                TextSpan(
                  style: const TextStyle(fontSize: 18, height: 1.5),
                  children: surligner(phrase, 'dragon'),
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.9 — Surligner un mot              │
├──────────────────────────────────────┤
│ Recherche : "coffre"                 │
│ Le gobelin garde le coffre du donjon │
│                       ^^^^^^ orange  │
│ depuis mille ans.                    │
│                                      │
│ Recherche : "dragon" (absent)        │
│ Le gobelin garde le coffre du donjon │
│ depuis mille ans.  (aucun surlignage)│
└──────────────────────────────────────┘
```

> La fonction `surligner` est une fonction Dart pure : elle ne connaît rien à Flutter à part `TextSpan`. Elle est donc **testable unitairement** sans lancer d'interface. C'est une très bonne habitude.

---

### 47.9.2 — Un lien cliquable dans une phrase

Un `TextSpan` accepte un `recognizer`, qui détecte les gestes. Le plus courant est `TapGestureRecognizer`, importé depuis `package:flutter/gestures.dart`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatefulWidget {
  const GameApp({super.key});

  @override
  State<GameApp> createState() => _GameAppState();
}

class _GameAppState extends State<GameApp> {
  late final TapGestureRecognizer _recognizer;
  int _clics = 0;

  @override
  void initState() {
    super.initState();
    _recognizer = TapGestureRecognizer()
      ..onTap = () {
        setState(() => _clics++);
      };
  }

  @override
  void dispose() {
    _recognizer.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.9.2 — Lien cliquable')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text.rich(
                TextSpan(
                  style: const TextStyle(fontSize: 18, height: 1.5),
                  children: [
                    const TextSpan(text: 'Pour equiper une arme, consultez '),
                    TextSpan(
                      text: 'le manuel du joueur',
                      style: const TextStyle(
                        color: Colors.blue,
                        decoration: TextDecoration.underline,
                      ),
                      recognizer: _recognizer,
                    ),
                    const TextSpan(text: ' avant de partir.'),
                  ],
                ),
              ),
              const SizedBox(height: 24),
              Text('Lien clique $_clics fois',
                  style: const TextStyle(fontSize: 16)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran (après 3 clics sur le lien) :**

```text
┌──────────────────────────────────────┐
│ 47.9.2 — Lien cliquable              │
├──────────────────────────────────────┤
│ Pour equiper une arme, consultez le  │
│ manuel du joueur avant de partir.    │
│      (bleu souligné, cliquable)      │
│                                      │
│ Lien clique 3 fois                   │
└──────────────────────────────────────┘
```

> Le `TapGestureRecognizer` doit être libéré dans `dispose()` (chapitre 45). C'est un objet qui écoute les gestes ; l'oublier crée une fuite mémoire. Ne le créez donc **jamais** directement dans `build()`.

---

## 47.10 — `SelectableText`

Le texte affiché par `Text` n'est **pas sélectionnable**. L'utilisateur ne peut ni le surligner ni le copier. Sur mobile c'est souvent normal ; sur le web et le bureau, c'est frustrant.

`SelectableText` corrige cela. Il s'utilise exactement comme `Text` :

```dart
SelectableText('Code de la partie : AX-7741-KLM')
```

Il existe aussi `SelectableText.rich`, qui prend un `TextSpan` comme `Text.rich`.

Paramètres utiles :

| Paramètre | Rôle |
| --- | --- |
| `style` | identique à `Text` |
| `textAlign`, `maxLines` | identiques à `Text` |
| `showCursor` | affiche un curseur de saisie (faux par défaut) |
| `cursorColor` | couleur du curseur |
| `onTap` | callback au simple appui |

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.10 — SelectableText')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('Text ordinaire (non selectionnable) :',
                  style: TextStyle(color: Colors.grey)),
              SizedBox(height: 4),
              Text('AX-7741-KLM', style: TextStyle(fontSize: 22)),
              SizedBox(height: 32),
              Text('SelectableText :',
                  style: TextStyle(color: Colors.grey)),
              SizedBox(height: 4),
              SelectableText(
                'AX-7741-KLM',
                style: TextStyle(fontSize: 22),
                showCursor: true,
                cursorColor: Colors.deepPurple,
              ),
              SizedBox(height: 32),
              Text('SelectableText.rich :',
                  style: TextStyle(color: Colors.grey)),
              SizedBox(height: 4),
              SelectableText.rich(
                TextSpan(
                  style: TextStyle(fontSize: 18),
                  children: [
                    TextSpan(text: 'Serveur : '),
                    TextSpan(
                      text: 'eu-west-3.jeu.example',
                      style: TextStyle(fontWeight: FontWeight.bold),
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.10 — SelectableText               │
├──────────────────────────────────────┤
│ Text ordinaire (non selectionnable) :│
│ AX-7741-KLM                          │
│   -> appui long : rien ne se passe   │
│                                      │
│ SelectableText :                     │
│ AX-7741-KLM|                         │
│   -> appui long : poignées de        │
│      sélection + menu Copier         │
│                                      │
│ SelectableText.rich :                │
│ Serveur : eu-west-3.jeu.example      │
└──────────────────────────────────────┘
```

---

### 47.10.1 — `SelectionArea` : rendre toute une zone sélectionnable

Remplacer chaque `Text` par un `SelectableText` est fastidieux. Flutter propose mieux : enveloppez une portion de l'arbre dans un `SelectionArea` et **tous** les `Text` qu'elle contient deviennent sélectionnables d'un seul geste.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.10.1 — SelectionArea')),
        body: const SelectionArea(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Fiche du personnage',
                    style: TextStyle(
                        fontSize: 24, fontWeight: FontWeight.bold)),
                SizedBox(height: 12),
                Text('Nom : Alex'),
                Text('Classe : Rodeuse'),
                Text('Niveau : 12'),
                Text('Points de vie : 340 / 400'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.10.1 — SelectionArea              │
├──────────────────────────────────────┤
│ Fiche du personnage                  │
│ Nom : Alex                           │
│ Classe : Rodeuse                     │  <- un seul glissement
│ Niveau : 12                          │     sélectionne plusieurs
│ Points de vie : 340 / 400            │     lignes d'un coup
└──────────────────────────────────────┘
```

> `SelectionArea` est presque toujours le bon choix pour une application web ou bureau. Placez-le haut dans l'arbre, par exemple juste sous le `Scaffold`.

---

## 47.11 — Les styles du thème (`Theme.of(context).textTheme`)

Vous savez maintenant styliser un texte à la main. Nous allons voir pourquoi il ne faut **presque jamais** le faire.

---

### 47.11.1 — Les quinze styles de Material 3

Chaque application Flutter possède un `TextTheme` : une collection de quinze `TextStyle` prêts à l'emploi, cohérents entre eux, définis par les règles de Material Design.

```text
                    LES 15 STYLES DU TEXTTHEME

  DISPLAY   displayLarge     57 px    écran d'accueil, chiffre géant
            displayMedium    45 px
            displaySmall     36 px

  HEADLINE  headlineLarge    32 px    titre d'écran
            headlineMedium   28 px
            headlineSmall    24 px

  TITLE     titleLarge       22 px    titre de carte, d'AppBar
            titleMedium      16 px
            titleSmall       14 px

  BODY      bodyLarge        16 px    paragraphe important
            bodyMedium       14 px    texte par défaut
            bodySmall        12 px    mention secondaire

  LABEL     labelLarge       14 px    texte de bouton
            labelMedium      12 px
            labelSmall       11 px    étiquette, badge
```

Les tailles indiquées sont celles de la référence Material 3 ; elles peuvent varier légèrement selon la version de Flutter et le thème appliqué. **N'apprenez pas les chiffres, apprenez les noms.**

---

### 47.11.2 — Comment y accéder

```dart
Theme.of(context).textTheme.headlineMedium
```

Décomposons cette expression, que vous écrirez des milliers de fois :

```text
Theme.of(context)          remonte l'arbre jusqu'au thème le plus proche
      .textTheme           récupère l'objet TextTheme
      .headlineMedium      récupère un TextStyle?  (attention : nullable)
```

Le type retourné est `TextStyle?`, avec un point d'interrogation. Il peut donc être `null` si le thème ne le définit pas. En pratique, avec `MaterialApp`, il ne l'est jamais — mais Dart vous oblige à le traiter (chapitre 12) :

```dart
Text('Titre', style: Theme.of(context).textTheme.headlineMedium)
```

Cela compile, car `style` accepte `TextStyle?`.

Pour le modifier, `?.copyWith` :

```dart
Text(
  'Alerte',
  style: Theme.of(context).textTheme.headlineMedium?.copyWith(
        color: Colors.red,
      ),
)
```

> Rappel du chapitre 12 : `?.` n'appelle `copyWith` que si la valeur n'est pas nulle. Si elle l'est, l'expression vaut `null` et `Text` retombe sur le style par défaut. Aucun plantage.

---

### 47.11.3 — Programme complet : les quinze styles à l'écran

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: const _EcranStyles(),
    );
  }
}

class _EcranStyles extends StatelessWidget {
  const _EcranStyles();

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;

    return Scaffold(
      appBar: AppBar(title: const Text('47.11 — textTheme')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Text('displayLarge', style: t.displayLarge),
          Text('displayMedium', style: t.displayMedium),
          Text('displaySmall', style: t.displaySmall),
          const Divider(),
          Text('headlineLarge', style: t.headlineLarge),
          Text('headlineMedium', style: t.headlineMedium),
          Text('headlineSmall', style: t.headlineSmall),
          const Divider(),
          Text('titleLarge', style: t.titleLarge),
          Text('titleMedium', style: t.titleMedium),
          Text('titleSmall', style: t.titleSmall),
          const Divider(),
          Text('bodyLarge', style: t.bodyLarge),
          Text('bodyMedium', style: t.bodyMedium),
          Text('bodySmall', style: t.bodySmall),
          const Divider(),
          Text('labelLarge', style: t.labelLarge),
          Text('labelMedium', style: t.labelMedium),
          Text('labelSmall', style: t.labelSmall),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran (haut de la liste) :**

```text
┌──────────────────────────────────────┐
│ 47.11 — textTheme                    │
├──────────────────────────────────────┤
│ displayLar…                          │  <- très grand
│ displayMedium                        │
│ displaySmall                         │
│ ──────────────────────────────────── │
│ headlineLarge                        │
│ headlineMedium                       │
│ headlineSmall                        │
│ ──────────────────────────────────── │
│ titleLarge                           │
│ titleMedium                          │
│ titleSmall                           │
│ ──────────────────────────────────── │
│ bodyLarge                            │
│ bodyMedium                           │
│ bodySmall                            │
│ ...  (faites défiler)                │
└──────────────────────────────────────┘
```

> `ListView` sera étudié au chapitre 48. Ici, il sert seulement à rendre l'écran défilant, car les quinze styles ne tiennent pas sur un téléphone.

---

### 47.11.4 — Le tableau de correspondance « quel style pour quoi ? »

| Vous voulez afficher | Style à utiliser |
| --- | --- |
| le titre d'un écran | `headlineMedium` ou `headlineSmall` |
| le titre d'une carte | `titleLarge` |
| le sous-titre d'une carte | `titleMedium` |
| un paragraphe | `bodyLarge` ou `bodyMedium` |
| une date, une mention | `bodySmall` |
| le texte d'un bouton | `labelLarge` |
| un badge, une étiquette | `labelSmall` |
| un score énorme, un chrono | `displayMedium` |

---

## 47.12 — Pourquoi passer par le thème plutôt que coder les tailles en dur

Voici la section la plus importante de la première moitié de ce chapitre. Elle ne contient presque pas de code, mais elle vous fera gagner des semaines.

---

### 47.12.1 — Le problème, en trois écrans

Imaginons une application de trois écrans. Sur chacun, vous écrivez le titre à la main :

```dart
// écran 1
Text('Inventaire', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold))

// écran 2
Text('Boutique', style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold))

// écran 3
Text('Reglages', style: TextStyle(fontSize: 24, fontWeight: FontWeight.w600))
```

Trois écrans, trois styles **différents**. Personne ne l'a voulu ; c'est juste que vous avez tapé les chiffres de mémoire. Le résultat est une application qui « fait bricolage » sans qu'on sache dire pourquoi.

Puis le client demande : « les titres sont trop petits, agrandissez-les ». Vous devez maintenant retrouver et modifier tous les titres, dans tous les fichiers. Et vous en oublierez un.

---

### 47.12.2 — La même chose avec le thème

```dart
// écran 1
Text('Inventaire', style: Theme.of(context).textTheme.headlineSmall)

// écran 2
Text('Boutique', style: Theme.of(context).textTheme.headlineSmall)

// écran 3
Text('Reglages', style: Theme.of(context).textTheme.headlineSmall)
```

Les trois titres sont **identiques par construction**. Et pour tous les agrandir, on modifie **un seul endroit** : le thème, dans `MaterialApp`.

```text
                    SANS THÈME                    AVEC THÈME

  écran 1  ──> fontSize: 24  ┐          écran 1 ──┐
  écran 2  ──> fontSize: 22  ├─ 3 lieux  écran 2 ──┼──> headlineSmall ──> 1 lieu
  écran 3  ──> fontSize: 24  ┘           écran 3 ──┘         (le thème)

  changer la taille :                    changer la taille :
  3 fichiers à modifier                  1 fichier à modifier
  risque d'oubli : élevé                 risque d'oubli : nul
```

---

### 47.12.3 — Les quatre bénéfices concrets

1. **Cohérence.** Deux titres de même niveau se ressemblent forcément.
2. **Maintenance.** Un changement de charte graphique = un fichier.
3. **Mode sombre.** Les couleurs du thème s'adaptent automatiquement au mode sombre. Un `color: Colors.black` codé en dur reste noir sur fond noir : illisible.
4. **Accessibilité.** Les styles du thème respectent les réglages de taille du système.

Le point 3 mérite un exemple. Comparez :

```dart
// À ÉVITER : reste noir en mode sombre, donc invisible
Text('Score', style: TextStyle(color: Colors.black))

// CORRECT : suit le thème, devient clair en mode sombre
Text('Score', style: Theme.of(context).textTheme.titleLarge)

// CORRECT AUSSI : couleur sémantique tirée du ColorScheme
Text('Score', style: TextStyle(color: Theme.of(context).colorScheme.primary))
```

---

### 47.12.4 — Personnaliser le thème : un aperçu

Le chapitre 51 est entièrement consacré aux thèmes. Voici tout de même de quoi comprendre le mécanisme dès maintenant.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
        textTheme: const TextTheme(
          headlineSmall: TextStyle(
            fontSize: 26,
            fontWeight: FontWeight.bold,
            letterSpacing: 0.5,
          ),
          bodyMedium: TextStyle(fontSize: 16, height: 1.5),
        ),
      ),
      home: const _Ecran(),
    );
  }
}

class _Ecran extends StatelessWidget {
  const _Ecran();

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;

    return Scaffold(
      appBar: AppBar(title: const Text('47.12 — Le theme centralise')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Inventaire', style: t.headlineSmall),
            const SizedBox(height: 12),
            Text(
              'Votre sac contient trois potions de soin, une epee courte '
              'et une carte du donjon partiellement effacee.',
              style: t.bodyMedium,
            ),
            const SizedBox(height: 24),
            Text('Boutique', style: t.headlineSmall),
            const SizedBox(height: 12),
            Text(
              'Le marchand propose aujourd hui une armure de cuir renforce.',
              style: t.bodyMedium,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.12 — Le theme centralise          │
├──────────────────────────────────────┤
│ I n v e n t a i r e                  │  <- 26 px, gras, espacé
│                                      │
│ Votre sac contient trois potions de  │
│                                      │  <- 16 px, interligne 1.5
│ soin, une epee courte et une carte   │
│                                      │
│ du donjon partiellement effacee.     │
│                                      │
│ B o u t i q u e                      │  <- exactement le même style
│                                      │
│ Le marchand propose aujourd hui une  │
│                                      │
│ armure de cuir renforce.             │
└──────────────────────────────────────┘
```

Changez `fontSize: 26` en `fontSize: 34` dans le `TextTheme` : **les deux titres** changent d'un coup. C'est là tout l'intérêt.

> Attention : fournir un `TextTheme` partiel comme ci-dessus **remplace** les styles listés et laisse les autres tels quels. Pour partir du thème existant et n'en modifier qu'une partie proprement, on utilise `ThemeData().textTheme.copyWith(...)`. Le chapitre 51 détaille tout cela.

---

## 47.13 — Les polices : où les trouver, quelles licences

Une police (en anglais *font*) est un fichier. Deux formats sont acceptés par Flutter :

| Extension | Nom | Remarque |
| --- | --- | --- |
| `.ttf` | TrueType Font | le plus courant, recommandé |
| `.otf` | OpenType Font | fonctionne également |

Les formats `.woff` et `.woff2`, destinés au web, ne sont **pas** utilisables tels quels par Flutter.

---

### 47.13.1 — Où trouver des polices utilisables

| Source | Adresse | Licence habituelle |
| --- | --- | --- |
| Google Fonts | `https://fonts.google.com` | Open Font License, usage commercial libre |
| Font Squirrel | `https://www.fontsquirrel.com` | filtre « 100 % free for commercial use » |
| The League of Moveable Type | `https://www.theleagueofmoveabletype.com` | Open Font License |
| Fontshare | `https://www.fontshare.com` | gratuit, usage commercial |

Google Fonts est le point de départ évident : plusieurs milliers de familles, toutes sous licence libre, téléchargeables en `.ttf`.

---

### 47.13.2 — Lire une licence en trois questions

Avant d'embarquer une police dans une application, posez-vous ces trois questions :

```text
1. Ai-je le droit de l'utiliser dans un produit COMMERCIAL ?
2. Ai-je le droit de l'INCLURE dans un fichier distribué (APK, IPA) ?
3. Dois-je citer l'auteur quelque part dans l'application ?
```

Les licences que vous rencontrerez :

| Licence | Commercial | Embarquer | Citation |
| --- | --- | --- | --- |
| SIL Open Font License (OFL) | oui | oui | non obligatoire, mais courtoise |
| Apache 2.0 | oui | oui | mention conseillée |
| « Free for personal use » | **non** | non | interdit en commercial |
| Police achetée (desktop) | dépend | souvent **non** | lire le contrat |

> Le piège classique : une police marquée « free » sur un site généraliste est souvent « free for **personal** use ». L'embarquer dans une application vendue sur un magasin d'applications est une violation de licence. Vérifiez toujours le fichier `LICENSE.txt` fourni dans l'archive.

Bonne pratique : ajoutez un écran « Licences » dans votre application. Flutter en fournit un tout fait avec `showLicensePage(context: context)`.

---

## 47.14 — Déclarer une police dans `pubspec.yaml`

Trois étapes, dans cet ordre.

**Étape 1 — Ranger les fichiers.** Créez un dossier `fonts/` à la racine du projet :

```text
mon_projet/
├── lib/
│   └── main.dart
├── fonts/
│   ├── Raleway-Regular.ttf
│   └── Raleway-Bold.ttf
└── pubspec.yaml
```

**Étape 2 — Déclarer dans `pubspec.yaml`.**

```yaml
flutter:
  uses-material-design: true

  fonts:
    - family: Raleway
      fonts:
        - asset: fonts/Raleway-Regular.ttf
        - asset: fonts/Raleway-Bold.ttf
          weight: 700
```

**Étape 3 — Utiliser.** La valeur de `fontFamily` doit être **exactement** celle de `family` :

```dart
Text('Bonjour', style: TextStyle(fontFamily: 'Raleway'))
```

Pour appliquer la police à toute l'application :

```dart
MaterialApp(
  theme: ThemeData(fontFamily: 'Raleway'),
  home: const MonEcran(),
)
```

> Après toute modification de `pubspec.yaml`, un **hot reload ne suffit pas**. Il faut arrêter l'application et la relancer (hot restart, voire relance complète). C'est la cause numéro un du « ma police ne s'applique pas ».

---

## 47.15 — Les graisses et les variantes

Un fichier `.ttf` classique contient **une seule** graisse et **un seul** style. Une famille complète, c'est donc plusieurs fichiers.

```text
Raleway-Light.ttf        -> weight 300, droit
Raleway-Regular.ttf      -> weight 400, droit
Raleway-Bold.ttf         -> weight 700, droit
Raleway-Italic.ttf       -> weight 400, italique
Raleway-BoldItalic.ttf   -> weight 700, italique
```

La déclaration complète associe à chaque fichier son `weight` et son `style` :

```yaml
flutter:
  uses-material-design: true

  fonts:
    - family: Raleway
      fonts:
        - asset: fonts/Raleway-Light.ttf
          weight: 300
        - asset: fonts/Raleway-Regular.ttf
          weight: 400
        - asset: fonts/Raleway-Bold.ttf
          weight: 700
        - asset: fonts/Raleway-Italic.ttf
          weight: 400
          style: italic
        - asset: fonts/Raleway-BoldItalic.ttf
          weight: 700
          style: italic

    - family: RobotoMono
      fonts:
        - asset: fonts/RobotoMono-Regular.ttf
```

Les valeurs autorisées :

| Clé | Valeurs autorisées | Défaut |
| --- | --- | --- |
| `weight` | 100, 200, 300, 400, 500, 600, 700, 800, 900 | 400 |
| `style` | `normal`, `italic` | `normal` |

Ce que Flutter fait ensuite :

```text
Vous écrivez         Flutter cherche              Résultat
─────────────────────────────────────────────────────────────────
fontWeight: w700  -> le fichier weight: 700   ->  vrai gras
fontWeight: w300  -> le fichier weight: 300   ->  vraie graisse light
fontWeight: w600  -> AUCUN fichier 600        ->  approximation
fontStyle: italic -> le fichier style: italic ->  vraie italique
                     (si absent)              ->  penché simulé ou ignoré
```

> Règle : n'utilisez dans votre code que les graisses que vous avez réellement déclarées. Trois fichiers (300, 400, 700) suffisent pour presque toutes les applications, et gardent l'APK léger. Chaque `.ttf` pèse entre 30 et 300 ko.

**Le cas des polices variables.** Certaines polices modernes (fichiers `.ttf` dits *variables*) contiennent toutes les graisses dans un seul fichier. Flutter les gère : déclarez le fichier une fois, sans `weight`, et toutes les valeurs de `fontWeight` fonctionnent.

---

## 47.16 — `google_fonts` : l'alternative sans fichier

Télécharger, ranger, déclarer, relancer : c'est fastidieux, surtout pendant les essais. Le package `google_fonts` supprime toutes ces étapes.

**Installation** (dans le dossier du projet) :

```text
flutter pub add google_fonts
```

Cette commande écrit elle-même la bonne ligne dans `pubspec.yaml`, avec la version courante. C'est toujours préférable à recopier un numéro de version trouvé sur Internet.

**Utilisation** :

```dart
import 'package:google_fonts/google_fonts.dart';

Text('Bonjour', style: GoogleFonts.lato())
```

Chaque famille de Google Fonts a sa méthode : `GoogleFonts.roboto()`, `GoogleFonts.oswald()`, `GoogleFonts.pressStart2P()`, etc. Toutes acceptent les mêmes paramètres qu'un `TextStyle`, plus un paramètre `textStyle` pour partir d'un style existant :

```dart
GoogleFonts.lato(
  textStyle: Theme.of(context).textTheme.headlineSmall,
  fontWeight: FontWeight.w700,
)
```

**Pour toute l'application** :

```dart
MaterialApp(
  theme: ThemeData(
    textTheme: GoogleFonts.latoTextTheme(),
  ),
  home: const MonEcran(),
)
```

Programme complet (nécessite `flutter pub add google_fonts`) :

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.deepPurple,
        textTheme: GoogleFonts.latoTextTheme(),
      ),
      home: const _Ecran(),
    );
  }
}

class _Ecran extends StatelessWidget {
  const _Ecran();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('47.16 — google_fonts')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Lato via le theme',
                style: Theme.of(context).textTheme.headlineSmall),
            const SizedBox(height: 16),
            Text('Oswald ponctuel',
                style: GoogleFonts.oswald(fontSize: 26)),
            const SizedBox(height: 16),
            Text('SCORE 004250',
                style: GoogleFonts.pressStart2P(fontSize: 14)),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.16 — google_fonts                 │
├──────────────────────────────────────┤
│ Lato via le theme                    │  <- Lato, 24 px
│                                      │
│ Oswald ponctuel                      │  <- Oswald, condensé
│                                      │
│ SCORE 004250                         │  <- Press Start 2P, style arcade
└──────────────────────────────────────┘
```

---

### 47.16.1 — Le point à connaître absolument sur `google_fonts`

Par défaut, `google_fonts` **télécharge la police au premier lancement** et la met en cache sur l'appareil. Conséquences :

```text
+ APK plus léger : aucun .ttf embarqué
+ essais très rapides : on change de police en une ligne
- il faut une connexion au premier affichage
- un très bref clignotement peut apparaître le temps du téléchargement
- sans réseau, la police de secours est utilisée
```

Pour une application destinée à fonctionner hors ligne, la bonne pratique est donc :

```text
PENDANT LE DÉVELOPPEMENT   ->  google_fonts (rapide à essayer)
POUR LA PRODUCTION         ->  embarquer les .ttf dans pubspec.yaml
```

`google_fonts` sait aussi utiliser des fichiers embarqués : si vous placez les `.ttf` dans le dossier `google_fonts/` déclaré en asset, le package les utilise au lieu de les télécharger.

> Pensez enfin aux licences (47.13). La documentation du package recommande d'enregistrer les licences des polices retenues avec `LicenseRegistry.addLicense()` afin qu'elles apparaissent dans l'écran des licences de l'application.

---

## 47.17 — `Icon` et `Icons`

Une icône, dans Flutter, n'est pas une image. C'est un **glyphe de police**, exactement comme une lettre. C'est pour cela qu'elle est nette à toutes les tailles et qu'elle se colorie comme un texte.

```dart
Icon(Icons.favorite)
```

Deux éléments distincts :

| Élément | Nature | Rôle |
| --- | --- | --- |
| `Icon` | un widget | affiche un glyphe |
| `Icons` | une classe de constantes | le catalogue des glyphes disponibles |
| `IconData` | une classe | décrit un glyphe (code + police) |

`Icons.favorite` est donc une constante de type `IconData`, et `Icon` est le widget qui la dessine.

Le catalogue complet est consultable sur `https://fonts.google.com/icons`. Il contient plusieurs milliers d'entrées. Quelques-unes que vous utiliserez tout le temps :

```text
Icons.home            Icons.search          Icons.settings
Icons.person          Icons.favorite        Icons.star
Icons.add             Icons.remove          Icons.close
Icons.check           Icons.edit            Icons.delete
Icons.arrow_back      Icons.arrow_forward   Icons.menu
Icons.more_vert       Icons.share           Icons.shopping_cart
Icons.visibility      Icons.visibility_off  Icons.refresh
```

Beaucoup d'icônes existent en plusieurs variantes, repérables au suffixe :

```text
Icons.star            plein
Icons.star_border     contour
Icons.star_outlined   contour (autre jeu)
Icons.star_rounded    coins arrondis
Icons.star_sharp      coins nets
```

> `uses-material-design: true` doit être présent dans `pubspec.yaml` pour que les icônes Material soient embarquées. C'est le cas par défaut dans tout projet créé par `flutter create`. Si vos icônes apparaissent comme des carrés vides, vérifiez cette ligne en premier.

---

## 47.18 — Taille et couleur d'une icône

```dart
Icon(
  Icons.favorite,
  size: 48,
  color: Colors.red,
)
```

| Paramètre | Type | Effet |
| --- | --- | --- |
| `size` | `double?` | largeur = hauteur, en pixels logiques (24 par défaut) |
| `color` | `Color?` | couleur du glyphe |
| `semanticLabel` | `String?` | description pour les lecteurs d'écran |
| `shadows` | `List<Shadow>?` | ombres portées |

Sans `color`, l'icône prend la couleur définie par l'`IconTheme` ambiant — c'est-à-dire, en pratique, la couleur du thème. C'est le comportement souhaitable dans la majorité des cas.

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.18 — Taille et couleur')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                crossAxisAlignment: CrossAxisAlignment.end,
                children: const [
                  Icon(Icons.favorite, size: 16, color: Colors.red),
                  SizedBox(width: 12),
                  Icon(Icons.favorite, size: 24, color: Colors.red),
                  SizedBox(width: 12),
                  Icon(Icons.favorite, size: 48, color: Colors.red),
                  SizedBox(width: 12),
                  Icon(Icons.favorite, size: 72, color: Colors.red),
                ],
              ),
              const SizedBox(height: 24),
              Row(
                children: const [
                  Icon(Icons.shield, size: 40, color: Colors.blueGrey),
                  SizedBox(width: 16),
                  Icon(Icons.local_fire_department,
                      size: 40, color: Colors.deepOrange),
                  SizedBox(width: 16),
                  Icon(Icons.water_drop, size: 40, color: Colors.lightBlue),
                  SizedBox(width: 16),
                  Icon(Icons.bolt, size: 40, color: Colors.amber),
                ],
              ),
              const SizedBox(height: 24),
              const Text('Sans couleur : suit le theme'),
              const SizedBox(height: 8),
              const Icon(Icons.settings, size: 40),
              const SizedBox(height: 24),
              const Text('Barre de vie en icones'),
              const SizedBox(height: 8),
              Row(
                children: List.generate(
                  5,
                  (int i) => Icon(
                    i < 3 ? Icons.favorite : Icons.favorite_border,
                    color: Colors.red,
                    size: 28,
                  ),
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.18 — Taille et couleur            │
├──────────────────────────────────────┤
│ [c] [C]  [ C ]   [  C  ]             │  coeurs 16/24/48/72 px, rouges
│                                      │
│ [bouclier] [flamme] [goutte] [eclair]│
│                                      │
│ Sans couleur : suit le theme         │
│ [engrenage]                          │
│                                      │
│ Barre de vie en icones               │
│ [plein][plein][plein][vide][vide]    │
└──────────────────────────────────────┘
```

> La dernière construction, avec `List.generate` (chapitre 06), est le moyen le plus court d'afficher une jauge de points de vie. Retenez-la : elle revient dans tous les jeux.

---

## 47.19 — `IconButton`

Une `Icon` n'est **pas** cliquable. Pour obtenir un bouton, utilisez `IconButton`.

```dart
IconButton(
  icon: const Icon(Icons.favorite),
  onPressed: () { /* action */ },
)
```

Deux paramètres sont obligatoires : `icon` et `onPressed`. Passer `onPressed: null` désactive le bouton (il devient gris et n'appelle rien) : c'est le mécanisme standard pour griser un bouton.

Paramètres utiles :

| Paramètre | Rôle |
| --- | --- |
| `icon` | le widget affiché (généralement une `Icon`) |
| `onPressed` | l'action ; `null` = désactivé |
| `iconSize` | taille de l'icône |
| `color` | couleur de l'icône active |
| `tooltip` | bulle d'aide au survol / appui long |
| `isSelected` + `selectedIcon` | bouton à deux états |

Material 3 ajoute trois constructeurs nommés, qui changent l'apparence du fond :

```text
IconButton(...)             fond transparent
IconButton.filled(...)      fond plein, couleur primaire
IconButton.filledTonal(...) fond doux, teinte secondaire
IconButton.outlined(...)    contour, fond transparent
```

Programme complet et interactif :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatefulWidget {
  const GameApp({super.key});

  @override
  State<GameApp> createState() => _GameAppState();
}

class _GameAppState extends State<GameApp> {
  int _vies = 3;
  bool _favori = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('47.19 — IconButton'),
          actions: [
            IconButton(
              icon: const Icon(Icons.refresh),
              tooltip: 'Reinitialiser',
              onPressed: () => setState(() {
                _vies = 3;
                _favori = false;
              }),
            ),
          ],
        ),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text('Vies : $_vies',
                  style: const TextStyle(fontSize: 24)),
              const SizedBox(height: 8),
              Row(
                children: [
                  IconButton(
                    icon: const Icon(Icons.remove),
                    tooltip: 'Perdre une vie',
                    onPressed:
                        _vies > 0 ? () => setState(() => _vies--) : null,
                  ),
                  IconButton(
                    icon: const Icon(Icons.add),
                    tooltip: 'Gagner une vie',
                    onPressed:
                        _vies < 5 ? () => setState(() => _vies++) : null,
                  ),
                ],
              ),
              const SizedBox(height: 24),
              const Text('Bouton a deux etats'),
              IconButton(
                isSelected: _favori,
                icon: const Icon(Icons.favorite_border),
                selectedIcon: const Icon(Icons.favorite, color: Colors.red),
                tooltip: 'Ajouter aux favoris',
                onPressed: () => setState(() => _favori = !_favori),
              ),
              const SizedBox(height: 24),
              const Text('Les quatre variantes Material 3'),
              const SizedBox(height: 8),
              Row(
                children: [
                  IconButton(
                      icon: const Icon(Icons.share), onPressed: () {}),
                  const SizedBox(width: 8),
                  IconButton.filled(
                      icon: const Icon(Icons.share), onPressed: () {}),
                  const SizedBox(width: 8),
                  IconButton.filledTonal(
                      icon: const Icon(Icons.share), onPressed: () {}),
                  const SizedBox(width: 8),
                  IconButton.outlined(
                      icon: const Icon(Icons.share), onPressed: () {}),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran (état initial) :**

```text
┌──────────────────────────────────────┐
│ 47.19 — IconButton          [refresh]│
├──────────────────────────────────────┤
│ Vies : 3                             │
│ [ - ]  [ + ]                         │
│                                      │
│ Bouton a deux etats                  │
│ [coeur vide]                         │
│                                      │
│ Les quatre variantes Material 3      │
│ (partage) (●partage) (○partage) (□)  │
└──────────────────────────────────────┘

Après 3 clics sur [ - ] : "Vies : 0" et le bouton [ - ] devient gris.
```

> Le passage de `onPressed` à `null` lorsque `_vies == 0` est idiomatique en Flutter. On n'écrit pas une condition dans le callback ; on désactive le bouton. L'utilisateur voit immédiatement que l'action est indisponible.

---

## 47.20 — Les icônes Material et Cupertino

Flutter fournit deux catalogues d'icônes.

| Catalogue | Classe | Import | Style |
| --- | --- | --- | --- |
| Material (Android, web) | `Icons` | `package:flutter/material.dart` | géométrique, plein |
| Cupertino (iOS) | `CupertinoIcons` | `package:flutter/cupertino.dart` | fin, contour |

Les noms diffèrent parfois :

| Concept | Material | Cupertino |
| --- | --- | --- |
| accueil | `Icons.home` | `CupertinoIcons.home` |
| retour | `Icons.arrow_back` | `CupertinoIcons.back` |
| partager | `Icons.share` | `CupertinoIcons.share` |
| réglages | `Icons.settings` | `CupertinoIcons.settings` |
| corbeille | `Icons.delete` | `CupertinoIcons.delete` |
| plus | `Icons.add` | `CupertinoIcons.add` |

Programme complet qui compare les deux jeux et choisit selon la plateforme :

```dart
import 'dart:io' show Platform;

import 'package:flutter/cupertino.dart';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

/// Renvoie true si l'on doit adopter le style iOS.
bool get styleIOS {
  if (kIsWeb) return false;
  return Platform.isIOS || Platform.isMacOS;
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.20 — Material vs Cupertino')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('Material'),
              const SizedBox(height: 8),
              Row(
                children: const [
                  Icon(Icons.home, size: 36),
                  SizedBox(width: 16),
                  Icon(Icons.share, size: 36),
                  SizedBox(width: 16),
                  Icon(Icons.settings, size: 36),
                  SizedBox(width: 16),
                  Icon(Icons.delete, size: 36),
                ],
              ),
              const SizedBox(height: 24),
              const Text('Cupertino'),
              const SizedBox(height: 8),
              Row(
                children: const [
                  Icon(CupertinoIcons.home, size: 36),
                  SizedBox(width: 16),
                  Icon(CupertinoIcons.share, size: 36),
                  SizedBox(width: 16),
                  Icon(CupertinoIcons.settings, size: 36),
                  SizedBox(width: 16),
                  Icon(CupertinoIcons.delete, size: 36),
                ],
              ),
              const SizedBox(height: 24),
              Text('Choix automatique (styleIOS = $styleIOS)'),
              const SizedBox(height: 8),
              Icon(
                styleIOS ? CupertinoIcons.back : Icons.arrow_back,
                size: 36,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran (sur Android) :**

```text
┌──────────────────────────────────────┐
│ 47.20 — Material vs Cupertino        │
├──────────────────────────────────────┤
│ Material                             │
│ [maison] [partage] [roue] [poubelle] │  <- traits épais, pleins
│                                      │
│ Cupertino                            │
│ [maison] [partage] [roue] [poubelle] │  <- traits fins, contours
│                                      │
│ Choix automatique (styleIOS = false) │
│ [fleche gauche Material]             │
└──────────────────────────────────────┘
```

> `Icon` accepte indifféremment un `Icons.x` ou un `CupertinoIcons.x` : les deux sont des `IconData`. Vous pouvez donc mélanger les deux catalogues, même si ce n'est pas recommandé du point de vue du design.

> Attention à `dart:io` : ce paquet n'existe pas sur le web. C'est pourquoi le code teste `kIsWeb` **avant** de toucher à `Platform`. Sans cela, l'application plante à la compilation web.

---

## 47.21 — Les assets : déclarer un dossier dans `pubspec.yaml`

Un **asset** est un fichier embarqué dans l'application : image, police, son, fichier JSON, texte. Il est copié dans l'APK ou l'IPA au moment de la compilation.

Flutter n'embarque **que** ce que vous déclarez. Un fichier posé dans un dossier sans déclaration n'existe tout simplement pas pour l'application.

**Étape 1 — Ranger.** La convention est un dossier `assets/` à la racine, avec des sous-dossiers par type :

```text
mon_projet/
├── assets/
│   ├── images/
│   │   ├── logo.png
│   │   └── fond.jpg
│   ├── icons/
│   │   └── epee.png
│   └── data/
│       └── niveaux.json
├── lib/
└── pubspec.yaml
```

**Étape 2 — Déclarer.** Deux syntaxes possibles.

Fichier par fichier :

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/logo.png
    - assets/images/fond.jpg
```

Dossier entier (le slash final est **obligatoire**) :

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/icons/
    - assets/data/
```

**Point capital :** déclarer `assets/images/` n'inclut **que** les fichiers directement dans ce dossier. Les sous-dossiers ne sont pas parcourus récursivement. Chaque sous-dossier doit être déclaré à son tour.

```text
assets:
  - assets/images/          -> inclut assets/images/logo.png
                            -> N'INCLUT PAS assets/images/boss/dragon.png

assets:
  - assets/images/
  - assets/images/boss/     -> maintenant dragon.png est inclus
```

**Étape 3 — Relancer.** Comme pour les polices, un hot reload ne prend pas en compte un nouvel asset. Arrêtez et relancez l'application.

**Quelle syntaxe choisir ?**

| Situation | Recommandation |
| --- | --- |
| projet en cours d'écriture, images qui bougent | déclarer le dossier |
| projet livré, taille de l'APK critique | déclarer fichier par fichier |
| beaucoup de fichiers | déclarer le dossier |

---

## 47.22 — L'indentation YAML : la source d'erreur numéro un

`pubspec.yaml` est écrit en YAML. En YAML, **l'indentation porte le sens**. Une espace en trop ou en moins change la structure du fichier.

Trois règles, non négociables :

```text
1. On indente avec des ESPACES. Jamais de tabulation.
2. Deux espaces par niveau. Toujours le même nombre.
3. Un tiret "- " introduit un élément de liste, et compte dans l'indentation.
```

Le fichier correct, commenté niveau par niveau :

```yaml
name: mon_projet
description: Un projet Flutter.

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter

flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/icons/

  fonts:
    - family: Raleway
      fonts:
        - asset: fonts/Raleway-Regular.ttf
        - asset: fonts/Raleway-Bold.ttf
          weight: 700
```

Lecture de l'arborescence :

```text
flutter:                          colonne 0  -> section racine
  uses-material-design: true      colonne 2  -> clé de "flutter"
  assets:                         colonne 2  -> clé de "flutter"
    - assets/images/              colonne 4  -> élément de la liste "assets"
  fonts:                          colonne 2  -> clé de "flutter"
    - family: Raleway             colonne 4  -> élément de la liste "fonts"
      fonts:                      colonne 6  -> clé DE CET ÉLÉMENT
        - asset: fonts/...        colonne 8  -> élément de la sous-liste
          weight: 700             colonne 10 -> clé DE CET élément de sous-liste
```

Remarquez la subtilité de la dernière ligne : `weight` est aligné sur `asset`, pas sur le tiret. Le tiret et l'espace qui le suit comptent pour deux colonnes.

---

### 47.22.1 — Les cinq erreurs YAML classiques

**Erreur 1 — `assets` mis au mauvais niveau.**

```yaml
flutter:
  uses-material-design: true

assets:                    # <- ERREUR : colonne 0, hors de "flutter"
  - assets/images/
```

Symptôme : l'application compile, mais toute image locale déclenche une exception `Unable to load asset`. Il faut deux espaces devant `assets:`.

**Erreur 2 — tabulation au lieu d'espaces.**

```yaml
flutter:
	assets:                  # <- ERREUR : une tabulation
	  - assets/images/
```

Symptôme : `flutter pub get` échoue avec un message du type « Error on line 12: mapping values are not allowed here » ou « found character that cannot start any token ». Configurez votre éditeur pour convertir les tabulations en espaces.

**Erreur 3 — slash final oublié sur un dossier.**

```yaml
  assets:
    - assets/images        # <- ERREUR : pas de slash final
```

Symptôme : Flutter cherche un **fichier** nommé `images`, ne le trouve pas, et n'embarque rien.

**Erreur 4 — chemin relatif au fichier Dart.**

```yaml
  assets:
    - ../assets/images/    # <- ERREUR
```

Tous les chemins sont relatifs à la **racine du projet**, celle où se trouve `pubspec.yaml`. Jamais à `lib/`.

**Erreur 5 — deux sections `flutter:`.**

```yaml
flutter:
  uses-material-design: true

dependencies:
  http: ^1.2.0

flutter:                   # <- ERREUR : la clé existe déjà
  assets:
    - assets/images/
```

Symptôme : `Error on line 20: Duplicate mapping key`. Une clé ne peut apparaître qu'une fois par niveau. Fusionnez les deux blocs.

---

### 47.22.2 — Vérifier son YAML

Trois réflexes, dans l'ordre :

```text
1. Lancer "flutter pub get" : toute erreur de syntaxe est signalée avec le numéro de ligne.
2. Ouvrir pubspec.yaml dans VS Code : l'extension Flutter souligne les erreurs en rouge.
3. Afficher les espaces (VS Code : Affichage > Rendu des espaces) pour repérer une tabulation.
```

> Le message d'erreur d'exécution le plus fréquent de tout Flutter est celui-ci :
> `Unable to load asset: "assets/images/logo.png"`
> Dans 90 % des cas, la cause est l'une des cinq erreurs ci-dessus, ou l'oubli du redémarrage complet de l'application.

---

## 47.23 — `Image.asset`

Une fois l'asset déclaré, l'afficher tient en une ligne :

```dart
Image.asset('assets/images/logo.png')
```

Le chemin est celui écrit dans `pubspec.yaml`, à la lettre près, en respectant la casse. Sur Android et iOS, `Logo.png` et `logo.png` ne sont pas la même chose.

Paramètres les plus utilisés :

| Paramètre | Rôle |
| --- | --- |
| `width`, `height` | dimensions imposées |
| `fit` | comment remplir l'espace (section 47.28) |
| `color` + `colorBlendMode` | teinter l'image |
| `alignment` | position de l'image dans sa boîte |
| `errorBuilder` | widget de repli si le fichier manque |
| `semanticLabel` | description pour l'accessibilité |

Exemple type (**non exécutable** sans le fichier correspondant) :

```dart
Image.asset(
  'assets/images/logo.png',
  width: 120,
  height: 120,
  fit: BoxFit.contain,
  semanticLabel: 'Logo du jeu',
  errorBuilder: (BuildContext context, Object error, StackTrace? stack) {
    return const Icon(Icons.broken_image, size: 120, color: Colors.grey);
  },
)
```

> `errorBuilder` sur un `Image.asset` est un excellent réflexe pendant le développement : au lieu d'un écran rouge d'exception, vous voyez une icône « image cassée » et le reste de l'écran continue de fonctionner.

Il existe une écriture équivalente, plus verbeuse, avec un `ImageProvider` :

```dart
const Image(image: AssetImage('assets/images/logo.png'))
```

`Image.asset(...)` est un raccourci pour cette forme. On utilise `AssetImage` directement lorsqu'un widget attend un `ImageProvider` plutôt qu'un widget : c'est le cas de `CircleAvatar` (47.30) et de `DecorationImage` (47.32).

---

## 47.24 — Les résolutions (`2.0x`, `3.0x`)

Un téléphone récent affiche 3 pixels physiques pour 1 pixel logique. Une image prévue pour 1 pixel logique par pixel physique y apparaît floue.

La solution de Flutter : fournir plusieurs versions du **même** fichier, rangées dans des sous-dossiers nommés d'après le facteur d'échelle.

```text
assets/images/
├── logo.png            <- version 1x   (par exemple 100 x 100 px)
├── 2.0x/
│   └── logo.png        <- version 2x   (200 x 200 px)
└── 3.0x/
    └── logo.png        <- version 3x   (300 x 300 px)
```

Correspondances usuelles :

| Dossier | Densité Android | Exemple d'appareil |
| --- | --- | --- |
| (racine) | mdpi | anciens appareils, web |
| `1.5x/` | hdpi | anciens appareils |
| `2.0x/` | xhdpi | téléphones milieu de gamme |
| `3.0x/` | xxhdpi | téléphones récents |
| `4.0x/` | xxxhdpi | tablettes haut de gamme |

**Ce que vous écrivez dans le code ne change pas :**

```dart
Image.asset('assets/images/logo.png')
```

Flutter choisit tout seul la variante adaptée à l'écran. Vous ne mentionnez **jamais** `2.0x` dans le code Dart.

**Ce que vous devez déclarer dans `pubspec.yaml` non plus :**

```yaml
flutter:
  assets:
    - assets/images/logo.png
```

Déclarer le fichier principal suffit : Flutter embarque automatiquement ses variantes de résolution.

```text
   Appareil                Flutter charge

   ratio 1.0     ─────>    assets/images/logo.png
   ratio 2.0     ─────>    assets/images/2.0x/logo.png
   ratio 2.75    ─────>    assets/images/3.0x/logo.png  (le plus proche au-dessus)
   ratio 3.0     ─────>    assets/images/3.0x/logo.png
```

> Si une variante manque, Flutter prend celle qui existe et la met à l'échelle. L'image sera correcte, simplement moins nette. Ne considérez pas les variantes comme obligatoires : c'est une optimisation de qualité, pas une exigence.

---

## 47.25 — `Image.network`

Pour afficher une image distante :

```dart
Image.network('https://picsum.photos/300/200')
```

C'est tout. Flutter télécharge, décode et affiche l'image, puis la garde en cache **mémoire** pour la session en cours.

Programme complet et exécutable (nécessite une connexion) :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.25 — Image.network')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Image.network(
                'https://picsum.photos/id/1043/300/200',
                width: 300,
                height: 200,
                fit: BoxFit.cover,
              ),
              const SizedBox(height: 16),
              const Text('Foret brumeuse — picsum.photos'),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.25 — Image.network                │
├──────────────────────────────────────┤
│    ┌────────────────────────────┐    │
│    │                            │    │
│    │      photo 300 x 200       │    │
│    │                            │    │
│    └────────────────────────────┘    │
│     Foret brumeuse — picsum.photos   │
└──────────────────────────────────────┘
```

---

### 47.25.1 — Le service `picsum.photos`

`https://picsum.photos` fournit des photos libres pour les maquettes. Les formes d'URL utiles :

| URL | Effet |
| --- | --- |
| `https://picsum.photos/300/200` | photo **aléatoire** 300 x 200 |
| `https://picsum.photos/id/1043/300/200` | la photo n° 1043, **toujours la même** |
| `https://picsum.photos/seed/alex/200/200` | photo tirée du mot-clé `alex`, stable |
| `https://picsum.photos/200/200?grayscale` | en noir et blanc |
| `https://picsum.photos/200/200?blur=4` | floutée |

> Préférez la forme `/id/` ou `/seed/` dans un cours ou une démonstration : l'image reste la même à chaque lancement, ce qui rend les captures d'écran reproductibles. La forme aléatoire, elle, provoque un rechargement à chaque reconstruction du widget et donne une impression de clignotement.

---

### 47.25.2 — Les trois obstacles d'`Image.network`

```text
1. PERMISSION ANDROID
   En release, Android exige la permission INTERNET :
   android/app/src/main/AndroidManifest.xml
   <uses-permission android:name="android.permission.INTERNET"/>
   Elle est présente d'office en debug, absente en release. D'où le
   classique « ça marchait en debug ».

2. HTTP EN CLAIR
   Depuis Android 9 et iOS 9, les URL en http:// sont bloquées.
   Utilisez https:// systématiquement.

3. CORS SUR LE WEB
   En Flutter web, le serveur distant doit autoriser votre origine.
   Sinon l'image ne s'affiche pas, avec une erreur dans la console
   du navigateur. picsum.photos autorise toutes les origines.
```

---

## 47.26 — `loadingBuilder` et `errorBuilder`

Une image distante met du temps à arriver, et peut ne jamais arriver. Sans traitement, l'utilisateur voit un trou blanc, puis éventuellement une exception.

`Image.network` propose deux fonctions pour couvrir ces deux cas.

---

### 47.26.1 — `loadingBuilder`

Signature exacte :

```dart
typedef ImageLoadingBuilder = Widget Function(
  BuildContext context,
  Widget child,
  ImageChunkEvent? loadingProgress,
);
```

Comment la lire :

| Paramètre | Signification |
| --- | --- |
| `child` | l'image elle-même, prête à être affichée |
| `loadingProgress` | `null` **quand le chargement est terminé** |
| `loadingProgress.cumulativeBytesLoaded` | octets reçus (`int`) |
| `loadingProgress.expectedTotalBytes` | taille totale (`int?`, souvent `null`) |

La règle d'or : **si `loadingProgress` est `null`, retournez `child`**. Sinon, retournez votre indicateur.

```dart
loadingBuilder: (BuildContext context, Widget child,
    ImageChunkEvent? progress) {
  if (progress == null) return child;
  return const Center(child: CircularProgressIndicator());
}
```

Pour un indicateur de progression réel, il faut calculer un ratio — mais `expectedTotalBytes` peut valoir `null` si le serveur n'envoie pas d'en-tête `Content-Length` :

```dart
final double? ratio = progress.expectedTotalBytes != null
    ? progress.cumulativeBytesLoaded / progress.expectedTotalBytes!
    : null;
return Center(child: CircularProgressIndicator(value: ratio));
```

Un `CircularProgressIndicator` dont `value` vaut `null` tourne indéfiniment : c'est exactement le comportement voulu quand la taille est inconnue.

---

### 47.26.2 — `errorBuilder`

Signature exacte :

```dart
typedef ImageErrorWidgetBuilder = Widget Function(
  BuildContext context,
  Object error,
  StackTrace? stackTrace,
);
```

Retournez le widget de repli de votre choix : une icône, une couleur, un texte.

```dart
errorBuilder: (BuildContext context, Object error, StackTrace? stack) {
  return const ColoredBox(
    color: Color(0xFFEEEEEE),
    child: Center(child: Icon(Icons.broken_image, color: Colors.grey)),
  );
}
```

---

### 47.26.3 — Programme complet : les trois états

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

/// Une image reseau robuste : chargement, erreur, dimensions imposees.
class ImageReseau extends StatelessWidget {
  const ImageReseau({
    super.key,
    required this.url,
    this.width = 300,
    this.height = 180,
  });

  final String url;
  final double width;
  final double height;

  @override
  Widget build(BuildContext context) {
    return Image.network(
      url,
      width: width,
      height: height,
      fit: BoxFit.cover,
      loadingBuilder: (BuildContext context, Widget child,
          ImageChunkEvent? progress) {
        if (progress == null) return child;
        final int? total = progress.expectedTotalBytes;
        final double? ratio =
            total != null ? progress.cumulativeBytesLoaded / total : null;
        return SizedBox(
          width: width,
          height: height,
          child: Center(child: CircularProgressIndicator(value: ratio)),
        );
      },
      errorBuilder:
          (BuildContext context, Object error, StackTrace? stack) {
        return Container(
          width: width,
          height: height,
          color: Colors.grey.shade200,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: const [
              Icon(Icons.broken_image, size: 40, color: Colors.grey),
              SizedBox(height: 8),
              Text('Image indisponible',
                  style: TextStyle(color: Colors.grey)),
            ],
          ),
        );
      },
    );
  }
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.26 — loading et error')),
        body: ListView(
          padding: const EdgeInsets.all(16),
          children: const [
            Text('URL valide'),
            SizedBox(height: 8),
            ImageReseau(url: 'https://picsum.photos/id/1015/300/180'),
            SizedBox(height: 24),
            Text('URL invalide'),
            SizedBox(height: 8),
            ImageReseau(url: 'https://picsum.photos/ceci-nexiste-pas.jpg'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
Pendant le chargement          Après chargement / après échec

┌──────────────────────┐       ┌──────────────────────┐
│ URL valide           │       │ URL valide           │
│ ┌──────────────────┐ │       │ ┌──────────────────┐ │
│ │      (o)         │ │  ->   │ │   photo 300x180  │ │
│ └──────────────────┘ │       │ └──────────────────┘ │
│ URL invalide         │       │ URL invalide         │
│ ┌──────────────────┐ │       │ ┌──────────────────┐ │
│ │      (o)         │ │  ->   │ │  [image cassee]  │ │
│ └──────────────────┘ │       │ │Image indisponible│ │
└──────────────────────┘       │ └──────────────────┘ │
                               └──────────────────────┘
```

> Le widget `ImageReseau` extrait ci-dessus est réutilisable partout. C'est exactement la démarche du chapitre 44 : dès qu'un montage se répète, on lui donne une classe et un nom.

---

## 47.27 — `cached_network_image`

Le cache d'`Image.network` est un cache **mémoire** : il disparaît à la fermeture de l'application. Sur une liste de cent vignettes, l'utilisateur retélécharge tout à chaque lancement.

Le package `cached_network_image` ajoute un cache **disque** persistant.

**Installation :**

```text
flutter pub add cached_network_image
```

**Utilisation :**

```dart
import 'package:cached_network_image/cached_network_image.dart';

CachedNetworkImage(
  imageUrl: 'https://picsum.photos/id/1015/300/180',
  placeholder: (BuildContext context, String url) =>
      const Center(child: CircularProgressIndicator()),
  errorWidget: (BuildContext context, String url, Object error) =>
      const Icon(Icons.error),
)
```

Correspondance avec `Image.network` :

| `Image.network` | `CachedNetworkImage` |
| --- | --- |
| `src` (positionnel) | `imageUrl` (nommé) |
| `loadingBuilder` | `placeholder` ou `progressIndicatorBuilder` |
| `errorBuilder` | `errorWidget` |
| `fit`, `width`, `height` | identiques |

Avec une barre de progression réelle :

```dart
CachedNetworkImage(
  imageUrl: url,
  progressIndicatorBuilder: (BuildContext context, String url,
          DownloadProgress progress) =>
      Center(child: CircularProgressIndicator(value: progress.progress)),
  errorWidget: (BuildContext context, String url, Object error) =>
      const Icon(Icons.error),
)
```

Le package fournit aussi un `ImageProvider` utilisable partout où un provider est attendu :

```dart
CircleAvatar(
  backgroundImage: CachedNetworkImageProvider(url),
)
```

Quand l'utiliser :

```text
Image.network            -> une ou deux images, écran ponctuel
CachedNetworkImage       -> listes, galeries, avatars, catalogue produits
                            + application censée fonctionner hors ligne
```

> Le cache disque est géré automatiquement : durée de vie par défaut de 30 jours, nettoyage automatique. Vous n'avez rien à écrire. Le chapitre 54 reviendra sur la persistance et le fonctionnement hors ligne.

---

## 47.28 — `BoxFit` : les sept valeurs, avec un schéma pour chacune

Le problème : la boîte que vous imposez (`width` x `height`) a rarement les mêmes proportions que l'image. Que faire de la différence ?

`BoxFit` répond à cette question. Il s'utilise avec `Image.asset`, `Image.network`, `DecorationImage`, `FittedBox`.

Pour tous les schémas qui suivent, la situation de départ est la même :

```text
IMAGE SOURCE (large)          BOÎTE CIBLE (carrée)

┌──────────────────────┐      ┌────────────┐
│                      │      │            │
│   400 x 200 px       │      │  200 x 200 │
│                      │      │            │
└──────────────────────┘      │            │
                              └────────────┘
```

---

### 47.28.1 — `BoxFit.fill`

Étire l'image pour remplir exactement la boîte. **Les proportions ne sont pas respectées.**

```text
┌────────────┐
│┌──────────┐│   L'image est compressée horizontalement
││ image    ││   et étirée verticalement.
││ DÉFORMÉE ││
│└──────────┘│   Aucun vide, aucune coupe, mais des visages
└────────────┘   écrasés. À réserver aux textures et aux motifs.
```

---

### 47.28.2 — `BoxFit.contain`

L'image est agrandie ou réduite pour tenir **entièrement** dans la boîte, sans déformation. Il reste du vide.

```text
┌────────────┐
│░░░░░░░░░░░░│   ░ = espace vide (transparent)
│┌──────────┐│
││  image   ││   Toute l'image est visible.
│└──────────┘│   Rien n'est coupé.
│░░░░░░░░░░░░│
└────────────┘
```

Usage : logos, illustrations, photos de produit sur fond neutre. C'est la valeur par défaut de `Image` lorsque `width` et `height` sont tous deux fournis.

---

### 47.28.3 — `BoxFit.cover`

L'image est agrandie jusqu'à **couvrir** toute la boîte, sans déformation. Ce qui dépasse est coupé.

```text
┌────────────┐
│░│ image  │░│   ░ = partie coupée (invisible)
│░│        │░│
│░│        │░│   Aucun vide, aucune déformation,
│░│        │░│   mais les bords sont perdus.
└────────────┘
```

Usage : photos de couverture, bannières, avatars, vignettes de liste. **C'est la valeur que vous utiliserez le plus souvent.**

---

### 47.28.4 — `BoxFit.fitWidth`

La largeur de l'image est ajustée à celle de la boîte. La hauteur suit, quitte à déborder ou à laisser du vide.

```text
┌────────────┐
│░░░░░░░░░░░░│
│┌──────────┐│   La largeur est exactement remplie.
││  image   ││   Ici l'image est moins haute : il reste
│└──────────┘│   du vide en haut et en bas.
│░░░░░░░░░░░░│
└────────────┘
```

---

### 47.28.5 — `BoxFit.fitHeight`

La hauteur de l'image est ajustée à celle de la boîte. La largeur suit.

```text
┌────────────┐
│░│        │░│   La hauteur est exactement remplie.
│░│ image  │░│   Ici l'image est plus large : les côtés
│░│        │░│   sont coupés.
│░│        │░│
└────────────┘
```

---

### 47.28.6 — `BoxFit.none`

Aucun redimensionnement. L'image est affichée à sa taille réelle, centrée, et ce qui dépasse est coupé.

```text
┌────────────┐
│░░░░░░░░░░░░│   Si l'image est plus petite que la boîte :
│░░ image ░░░│   elle apparaît petite, au centre.
│░░░░░░░░░░░░│
│░░░░░░░░░░░░│   Si elle est plus grande : seul le centre
└────────────┘   est visible.
```

---

### 47.28.7 — `BoxFit.scaleDown`

Comme `contain`, mais **jamais d'agrandissement**. Si l'image est plus petite que la boîte, elle reste à sa taille réelle.

```text
image PLUS GRANDE que la boîte   ->  comportement de contain
image PLUS PETITE que la boîte   ->  comportement de none
```

Usage : afficher des vignettes d'origines variées sans jamais dégrader les plus petites.

---

### 47.28.8 — Le tableau de décision

| Valeur | Déforme ? | Coupe ? | Laisse du vide ? |
| --- | --- | --- | --- |
| `fill` | **oui** | non | non |
| `contain` | non | non | **oui** |
| `cover` | non | **oui** | non |
| `fitWidth` | non | possible | possible |
| `fitHeight` | non | possible | possible |
| `none` | non | possible | possible |
| `scaleDown` | non | non | **oui** |

Comment choisir :

```text
Je veux TOUT voir, quitte à avoir du vide          -> contain
Je veux REMPLIR, quitte à couper les bords         -> cover
Je veux remplir exactement, la forme m'est égale   -> fill
Je ne veux jamais agrandir une petite image        -> scaleDown
```

---

### 47.28.9 — Programme complet : les sept côte à côte

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kUrl = 'https://picsum.photos/id/1025/400/200';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    const List<BoxFit> valeurs = <BoxFit>[
      BoxFit.fill,
      BoxFit.contain,
      BoxFit.cover,
      BoxFit.fitWidth,
      BoxFit.fitHeight,
      BoxFit.none,
      BoxFit.scaleDown,
    ];

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.28 — BoxFit')),
        body: ListView.builder(
          padding: const EdgeInsets.all(16),
          itemCount: valeurs.length,
          itemBuilder: (BuildContext context, int i) {
            final BoxFit fit = valeurs[i];
            return Padding(
              padding: const EdgeInsets.only(bottom: 20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'BoxFit.${fit.name}',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 6),
                  Container(
                    width: 200,
                    height: 200,
                    color: Colors.grey.shade300,
                    child: Image.network(kUrl, fit: fit),
                  ),
                ],
              ),
            );
          },
        ),
      ),
    );
  }
}
```

**Résultat à l'écran (trois premières entrées) :**

```text
┌──────────────────────────────────────┐
│ 47.28 — BoxFit                       │
├──────────────────────────────────────┤
│ BoxFit.fill                          │
│ ┌──────────┐                         │
│ │ deformee │  image étirée en hauteur│
│ └──────────┘                         │
│ BoxFit.contain                       │
│ ┌──────────┐                         │
│ │▒▒▒▒▒▒▒▒▒▒│  bandes grises en haut  │
│ │  image   │  et en bas              │
│ │▒▒▒▒▒▒▒▒▒▒│                         │
│ └──────────┘                         │
│ BoxFit.cover                         │
│ ┌──────────┐                         │
│ │  image   │  aucun gris visible,    │
│ │  image   │  côtés coupés           │
│ └──────────┘                         │
└──────────────────────────────────────┘
```

> Le `Container` gris derrière l'image sert de révélateur : partout où vous voyez du gris, l'image ne remplit pas la boîte. C'est l'astuce de débogage à retenir.

> `fit.name` fonctionne parce que `BoxFit` est un `enum` et que tout `enum` Dart expose `.name` depuis Dart 2.15 (chapitre 11).

---

## 47.29 — `ClipRRect` et les coins arrondis

Une image est toujours rectangulaire à angles vifs. Pour arrondir ses coins, on la **découpe** avec `ClipRRect` (*Clip Rounded Rectangle*).

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: Image.network(url, width: 200, height: 200, fit: BoxFit.cover),
)
```

L'ordre est essentiel : `ClipRRect` est le **parent**, l'image est l'**enfant**. L'inverse n'a aucun sens.

La famille des découpeurs :

| Widget | Forme obtenue |
| --- | --- |
| `ClipRRect` | rectangle à coins arrondis |
| `ClipOval` | ellipse (cercle si la boîte est carrée) |
| `ClipRect` | rectangle net (utile pour couper un débordement) |
| `ClipPath` | forme libre définie par un `CustomClipper` |

`BorderRadius` accepte plusieurs constructions :

```dart
BorderRadius.circular(16)                       // les 4 coins
BorderRadius.only(topLeft: Radius.circular(24)) // un seul coin
BorderRadius.vertical(top: Radius.circular(16)) // haut seulement
BorderRadius.horizontal(left: Radius.circular(16))
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kUrl = 'https://picsum.photos/id/1062/200/200';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.29 — ClipRRect')),
        body: Center(
          child: Wrap(
            spacing: 20,
            runSpacing: 20,
            children: [
              Image.network(kUrl, width: 120, height: 120, fit: BoxFit.cover),
              ClipRRect(
                borderRadius: BorderRadius.circular(16),
                child: Image.network(kUrl,
                    width: 120, height: 120, fit: BoxFit.cover),
              ),
              ClipRRect(
                borderRadius: const BorderRadius.vertical(
                  top: Radius.circular(40),
                ),
                child: Image.network(kUrl,
                    width: 120, height: 120, fit: BoxFit.cover),
              ),
              ClipOval(
                child: Image.network(kUrl,
                    width: 120, height: 120, fit: BoxFit.cover),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.29 — ClipRRect                    │
├──────────────────────────────────────┤
│   ┌────────┐      ╭────────╮         │
│   │ carre  │      │arrondi │         │
│   └────────┘      ╰────────╯         │
│                                      │
│   ╭────────╮      ╭────────╮         │
│   │demi-    │     │ cercle │         │
│   │rond    │      ╰────────╯         │
│   └────────┘                         │
└──────────────────────────────────────┘
```

> `Wrap` (chapitre 46) place ses enfants côte à côte et passe automatiquement à la ligne quand la place manque. C'est une `Row` qui sait revenir à la ligne.

---

### 47.29.1 — L'alternative sans découpage

`ClipRRect` a un coût : Flutter doit appliquer un masque à chaque image. Pour un fond d'image simple, `Container` + `BoxDecoration` est plus efficace :

```dart
Container(
  width: 120,
  height: 120,
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(16),
    image: const DecorationImage(
      image: NetworkImage('https://picsum.photos/id/1062/200/200'),
      fit: BoxFit.cover,
    ),
  ),
)
```

Nous détaillons `DecorationImage` en 47.32.

---

## 47.30 — `CircleAvatar`

`CircleAvatar` est le widget prêt à l'emploi pour une photo de profil ronde.

```dart
CircleAvatar(
  radius: 40,
  backgroundImage: NetworkImage('https://picsum.photos/id/64/200/200'),
)
```

Paramètres :

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `radius` | `double?` | rayon en pixels logiques (diamètre = 2 x rayon) |
| `backgroundColor` | `Color?` | couleur du disque |
| `backgroundImage` | `ImageProvider?` | image du disque |
| `foregroundImage` | `ImageProvider?` | image au-dessus du fond |
| `foregroundColor` | `Color?` | couleur par défaut du `child` |
| `child` | `Widget?` | contenu centré (texte, icône) |
| `onBackgroundImageError` | callback | appelé si l'image échoue |

Points à connaître :

```text
1. backgroundImage attend un ImageProvider, PAS un widget Image.
     NetworkImage(url)          et non Image.network(url)
     AssetImage('assets/x.png') et non Image.asset('assets/x.png')

2. Si backgroundImage échoue, le child s'affiche à la place.
   D'où le montage idéal : image en fond, initiales en secours.

3. radius: 40 donne un cercle de 80 px de diamètre.
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

/// Renvoie les initiales d'un nom complet : "Alex Dupont" -> "AD".
String initiales(String nomComplet) {
  final List<String> mots = nomComplet
      .trim()
      .split(RegExp(r'\s+'))
      .where((String m) => m.isNotEmpty)
      .toList();
  if (mots.isEmpty) return '?';
  if (mots.length == 1) return mots.first[0].toUpperCase();
  return (mots.first[0] + mots.last[0]).toUpperCase();
}

/// Couleur stable deduite du nom : le meme nom donne toujours la meme couleur.
Color couleurDepuisNom(String nom) {
  const List<Color> palette = <Color>[
    Color(0xFF1E88E5),
    Color(0xFF43A047),
    Color(0xFFE53935),
    Color(0xFF8E24AA),
    Color(0xFFF4511E),
    Color(0xFF00897B),
  ];
  int somme = 0;
  for (int i = 0; i < nom.length; i++) {
    somme += nom.codeUnitAt(i);
  }
  return palette[somme % palette.length];
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    const List<String> joueurs = <String>[
      'Alex Dupont',
      'Sophie Martin',
      'Samir Bennani',
      'Maria Silva',
    ];

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.30 — CircleAvatar')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('Avec une image reseau'),
              const SizedBox(height: 8),
              const CircleAvatar(
                radius: 40,
                backgroundImage:
                    NetworkImage('https://picsum.photos/id/64/200/200'),
              ),
              const SizedBox(height: 24),
              const Text('Image + initiales de secours'),
              const SizedBox(height: 8),
              CircleAvatar(
                radius: 40,
                backgroundColor: couleurDepuisNom('Alex Dupont'),
                foregroundColor: Colors.white,
                backgroundImage: const NetworkImage(
                    'https://picsum.photos/url-invalide.jpg'),
                onBackgroundImageError:
                    (Object error, StackTrace? stack) {},
                child: Text(initiales('Alex Dupont'),
                    style: const TextStyle(fontSize: 28)),
              ),
              const SizedBox(height: 24),
              const Text('Initiales seules, sans aucun fichier'),
              const SizedBox(height: 8),
              Row(
                children: joueurs.map((String nom) {
                  return Padding(
                    padding: const EdgeInsets.only(right: 12),
                    child: CircleAvatar(
                      radius: 28,
                      backgroundColor: couleurDepuisNom(nom),
                      child: Text(
                        initiales(nom),
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  );
                }).toList(),
              ),
              const SizedBox(height: 24),
              const Text('Avec une icone'),
              const SizedBox(height: 8),
              const CircleAvatar(
                radius: 28,
                backgroundColor: Colors.blueGrey,
                child: Icon(Icons.person, color: Colors.white, size: 30),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.30 — CircleAvatar                 │
├──────────────────────────────────────┤
│ Avec une image reseau                │
│  ( photo ronde )                     │
│                                      │
│ Image + initiales de secours         │
│  ( AD )   fond colore, image echouee │
│                                      │
│ Initiales seules, sans aucun fichier │
│  (AD) (SM) (SB) (MS)                 │
│  bleu  vert rouge violet             │
│                                      │
│ Avec une icone                       │
│  ( [personne] )                      │
└──────────────────────────────────────┘
```

> Les fonctions `initiales` et `couleurDepuisNom` sont du Dart pur (chapitres 07 et 14). Elles règlent à elles seules le problème « je n'ai pas de photos de profil » : chaque utilisateur obtient un avatar coloré, stable et reconnaissable. Toutes les grandes applications de messagerie font exactement cela.

---

## 47.31 — `Image.memory` et `Image.file`

Deux constructeurs pour deux situations particulières.

---

### 47.31.1 — `Image.memory`

Affiche une image contenue dans un tableau d'octets (`Uint8List`). Cas typiques : une image reçue en base64 depuis une API (chapitre 53), une photo prise par l'appareil et non encore enregistrée, une image générée à la volée.

```dart
Image.memory(mesOctets)
```

Programme complet et exécutable — il n'utilise **aucun fichier** : les octets d'un petit PNG sont écrits en base64 dans le code source.

```dart
import 'dart:convert';
import 'dart:typed_data';

import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

/// Un PNG de 1 pixel rouge, encode en base64.
const String kPngBase64 =
    'iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8'
    'z8BQDwAEhQGAhKmMIQAAAABJRU5ErkJggg==';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    final Uint8List octets = base64Decode(kPngBase64);

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.31 — Image.memory')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('${octets.length} octets decodes'),
              const SizedBox(height: 16),
              Image.memory(
                octets,
                width: 150,
                height: 150,
                fit: BoxFit.fill,
                filterQuality: FilterQuality.none,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.31 — Image.memory                 │
├──────────────────────────────────────┤
│          70 octets decodes           │
│        ┌──────────────┐              │
│        │              │              │
│        │  carre rouge │  1 pixel     │
│        │              │  agrandi     │
│        └──────────────┘              │
└──────────────────────────────────────┘
```

> `filterQuality: FilterQuality.none` désactive le lissage. Sans lui, agrandir un pixel unique donne un dégradé flou. C'est aussi le réglage à utiliser pour un jeu en pixel art (voir la PARTIE 2).

---

### 47.31.2 — `Image.file`

Affiche une image lue depuis le système de fichiers de l'appareil.

```dart
import 'dart:io';

Image.file(File('/chemin/vers/photo.jpg'))
```

Contraintes :

```text
1. dart:io N'EXISTE PAS sur le web. Image.file est donc inutilisable en Flutter web.
2. Il faut un chemin réel : on l'obtient avec un sélecteur (image_picker,
   file_picker) ou avec path_provider (chapitre 54).
3. Les permissions d'accès au stockage doivent être accordées.
```

Exemple type (**non exécutable** sans fichier ni package) :

```dart
import 'dart:io';

import 'package:flutter/material.dart';

class Apercu extends StatelessWidget {
  const Apercu({super.key, required this.chemin});

  final String chemin;

  @override
  Widget build(BuildContext context) {
    final File fichier = File(chemin);
    if (!fichier.existsSync()) {
      return const Icon(Icons.image_not_supported, size: 80);
    }
    return Image.file(
      fichier,
      width: 200,
      height: 200,
      fit: BoxFit.cover,
      errorBuilder: (BuildContext c, Object e, StackTrace? s) =>
          const Icon(Icons.broken_image, size: 80),
    );
  }
}
```

Récapitulatif des quatre sources d'image :

| Constructeur | Source | Disponible sur le web |
| --- | --- | --- |
| `Image.asset` | fichier embarqué dans l'application | oui |
| `Image.network` | URL distante | oui |
| `Image.memory` | tableau d'octets en mémoire | oui |
| `Image.file` | fichier du système local | **non** |

---

## 47.32 — `DecorationImage` dans un `Container`

Jusqu'ici, l'image était un widget. Avec `DecorationImage`, elle devient le **fond** d'un `Container`, et l'on peut poser du contenu par-dessus.

```dart
Container(
  height: 200,
  decoration: const BoxDecoration(
    image: DecorationImage(
      image: NetworkImage('https://picsum.photos/id/1039/600/400'),
      fit: BoxFit.cover,
    ),
  ),
  child: const Center(child: Text('Par-dessus')),
)
```

Paramètres de `DecorationImage` :

| Paramètre | Rôle |
| --- | --- |
| `image` | un `ImageProvider` (obligatoire) |
| `fit` | comme partout (47.28) |
| `alignment` | position dans la boîte |
| `opacity` | opacité de 0.0 à 1.0 |
| `colorFilter` | teinte ou assombrissement |
| `repeat` | répétition en mosaïque |
| `onError` | callback en cas d'échec |

**Le problème du texte illisible.** Un texte blanc sur une photo claire est illisible. La solution professionnelle est un `colorFilter` qui assombrit la photo :

```dart
DecorationImage(
  image: NetworkImage(url),
  fit: BoxFit.cover,
  colorFilter: ColorFilter.mode(
    Colors.black.withValues(alpha: 0.45),
    BlendMode.darken,
  ),
)
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kUrl = 'https://picsum.photos/id/1039/600/400';

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.32 — DecorationImage')),
        body: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            const Text('Sans filtre : texte peu lisible'),
            const SizedBox(height: 8),
            Container(
              height: 140,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(12),
                image: const DecorationImage(
                  image: NetworkImage(kUrl),
                  fit: BoxFit.cover,
                ),
              ),
              child: const Center(
                child: Text(
                  'NIVEAU 3',
                  style: TextStyle(
                      color: Colors.white,
                      fontSize: 30,
                      fontWeight: FontWeight.bold),
                ),
              ),
            ),
            const SizedBox(height: 24),
            const Text('Avec assombrissement : lisible'),
            const SizedBox(height: 8),
            Container(
              height: 140,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(12),
                image: DecorationImage(
                  image: const NetworkImage(kUrl),
                  fit: BoxFit.cover,
                  colorFilter: ColorFilter.mode(
                    Colors.black.withValues(alpha: 0.45),
                    BlendMode.darken,
                  ),
                ),
              ),
              child: const Center(
                child: Text(
                  'NIVEAU 3',
                  style: TextStyle(
                      color: Colors.white,
                      fontSize: 30,
                      fontWeight: FontWeight.bold),
                ),
              ),
            ),
            const SizedBox(height: 24),
            const Text('Fond en mosaique (repeat)'),
            const SizedBox(height: 8),
            Container(
              height: 140,
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: NetworkImage(
                      'https://picsum.photos/id/1084/60/60'),
                  repeat: ImageRepeat.repeat,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.32 — DecorationImage              │
├──────────────────────────────────────┤
│ Sans filtre : texte peu lisible      │
│ ╭──────────────────────────────────╮ │
│ │      NIVEAU 3   (blanc sur clair)│ │
│ ╰──────────────────────────────────╯ │
│ Avec assombrissement : lisible       │
│ ╭──────────────────────────────────╮ │
│ │      NIVEAU 3   (blanc sur sombre│ │
│ ╰──────────────────────────────────╯ │
│ Fond en mosaique (repeat)            │
│ ┌──────────────────────────────────┐ │
│ │ motif repete en damier           │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

> `Color.withValues(alpha: 0.45)` remplace l'ancienne méthode `withOpacity(0.45)`, dépréciée dans les versions récentes de Flutter. Les deux donnent le même résultat visuel ; utilisez `withValues`.

---

## 47.33 — `Opacity` et `ColorFiltered`

Deux widgets pour modifier l'apparence de **n'importe quel** enfant, image ou non.

---

### 47.33.1 — `Opacity`

```dart
Opacity(
  opacity: 0.4,   // 0.0 = invisible, 1.0 = opaque
  child: Image.network(url),
)
```

`Opacity` est simple mais **coûteux** : Flutter dessine l'enfant dans un tampon séparé, puis le compose. Sur une liste qui défile, cela se sent.

Les alternatives, dans l'ordre de préférence :

```text
1. Une couleur déjà semi-transparente : Colors.black.withValues(alpha: 0.4)
2. Le paramètre "opacity" de DecorationImage
3. Le paramètre "opacity" (Animation<double>) de Image
4. Opacity, en dernier recours
```

---

### 47.33.2 — `ColorFiltered`

`ColorFiltered` applique un filtre de couleur à son enfant :

```dart
ColorFiltered(
  colorFilter: const ColorFilter.mode(Colors.deepPurple, BlendMode.modulate),
  child: Image.network(url),
)
```

Deux constructions de filtre à connaître :

| Filtre | Effet |
| --- | --- |
| `ColorFilter.mode(couleur, blendMode)` | mélange une couleur selon un mode |
| `ColorFilter.matrix(List<double>)` | transformation libre, 20 coefficients |

Les `BlendMode` les plus utiles :

```text
BlendMode.srcIn       remplace TOUTE l'image par la couleur (silhouette)
BlendMode.modulate    multiplie : teinte l'image
BlendMode.darken      garde le plus sombre : assombrit
BlendMode.lighten     garde le plus clair : éclaircit
BlendMode.saturation  agit sur la saturation
BlendMode.color       applique la teinte en gardant la luminosité (sépia)
```

Le noir et blanc s'obtient avec une matrice de saturation nulle :

```dart
const ColorFilter kNoirEtBlanc = ColorFilter.matrix(<double>[
  0.2126, 0.7152, 0.0722, 0, 0,
  0.2126, 0.7152, 0.0722, 0, 0,
  0.2126, 0.7152, 0.0722, 0, 0,
  0,      0,      0,      1, 0,
]);
```

Les trois premières lignes calculent la luminance perçue et la copient sur les canaux rouge, vert et bleu. La quatrième laisse l'alpha inchangé.

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

const String kUrl = 'https://picsum.photos/id/1024/200/200';

const ColorFilter kNoirEtBlanc = ColorFilter.matrix(<double>[
  0.2126, 0.7152, 0.0722, 0, 0,
  0.2126, 0.7152, 0.0722, 0, 0,
  0.2126, 0.7152, 0.0722, 0, 0,
  0, 0, 0, 1, 0,
]);

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    Widget etiquette(String texte, Widget enfant) {
      return Column(
        children: [
          SizedBox(width: 110, height: 110, child: enfant),
          const SizedBox(height: 4),
          Text(texte, style: const TextStyle(fontSize: 12)),
        ],
      );
    }

    const Widget photo =
        Image(image: NetworkImage(kUrl), fit: BoxFit.cover);

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('47.33 — Opacity, ColorFiltered')),
        body: Center(
          child: Wrap(
            spacing: 16,
            runSpacing: 16,
            children: [
              etiquette('normale', photo),
              etiquette('opacity 0.35',
                  const Opacity(opacity: 0.35, child: photo)),
              etiquette(
                'noir et blanc',
                const ColorFiltered(
                    colorFilter: kNoirEtBlanc, child: photo),
              ),
              etiquette(
                'teinte violette',
                const ColorFiltered(
                  colorFilter: ColorFilter.mode(
                      Colors.deepPurple, BlendMode.modulate),
                  child: photo,
                ),
              ),
              etiquette(
                'assombrie',
                ColorFiltered(
                  colorFilter: ColorFilter.mode(
                      Colors.black.withValues(alpha: 0.5),
                      BlendMode.darken),
                  child: photo,
                ),
              ),
              etiquette(
                'silhouette',
                const ColorFiltered(
                  colorFilter:
                      ColorFilter.mode(Colors.indigo, BlendMode.srcIn),
                  child: photo,
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

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.33 — Opacity, ColorFiltered       │
├──────────────────────────────────────┤
│ [photo]   [photo pale]  [photo n&b]  │
│ normale   opacity 0.35  noir et blanc│
│                                      │
│ [violet]  [sombre]      [aplat bleu] │
│ teinte    assombrie     silhouette   │
└──────────────────────────────────────┘
```

> `BlendMode.srcIn` avec une couleur pleine transforme n'importe quelle image en silhouette monochrome. C'est ainsi que l'on colorie une icône fournie en PNG blanc sur fond transparent.

---

## 47.34 — Travailler sans aucun fichier image

Vous n'avez pas de graphiste. Vous n'avez aucun fichier. Vous devez malgré tout livrer une interface qui ne ressemble pas à un brouillon. Voici les cinq techniques, par ordre de rendu.

---

### 47.34.1 — Technique 1 : les dégradés

Un `Container` avec un `LinearGradient` remplace avantageusement une photo de fond :

```dart
Container(
  decoration: const BoxDecoration(
    gradient: LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: [Color(0xFF3F51B5), Color(0xFF9C27B0)],
    ),
  ),
)
```

---

### 47.34.2 — Technique 2 : icône dans une pastille colorée

```dart
Container(
  padding: const EdgeInsets.all(14),
  decoration: BoxDecoration(
    color: Colors.teal.withValues(alpha: 0.15),
    shape: BoxShape.circle,
  ),
  child: const Icon(Icons.shield, color: Colors.teal, size: 28),
)
```

---

### 47.34.3 — Technique 3 : initiales colorées

C'est le montage de la section 47.30 : `CircleAvatar` + `Text` + couleur déduite du nom.

---

### 47.34.4 — Technique 4 : images de remplissage distantes

`https://picsum.photos` fournit des photos réalistes et stables (voir 47.25.1). Pour un jeu, cette maquette suffit largement le temps d'obtenir les vrais visuels.

---

### 47.34.5 — Technique 5 : formes et ombres

Une carte avec `borderRadius`, `boxShadow` et deux couleurs paraît soignée sans la moindre image.

Programme complet regroupant les cinq techniques :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: Scaffold(
        appBar: AppBar(title: const Text('47.34 — Sans aucun fichier')),
        body: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            Container(
              height: 120,
              alignment: Alignment.center,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(16),
                gradient: const LinearGradient(
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                  colors: [Color(0xFF3F51B5), Color(0xFF9C27B0)],
                ),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withValues(alpha: 0.2),
                    blurRadius: 12,
                    offset: const Offset(0, 6),
                  ),
                ],
              ),
              child: const Text(
                'DONJON DE CRISTAL',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 22,
                  fontWeight: FontWeight.bold,
                  letterSpacing: 2,
                ),
              ),
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: const [
                _Pastille(Icons.shield, Colors.blue, 'Defense'),
                _Pastille(Icons.local_fire_department, Colors.deepOrange,
                    'Attaque'),
                _Pastille(Icons.bolt, Colors.amber, 'Vitesse'),
                _Pastille(Icons.favorite, Colors.pink, 'Vie'),
              ],
            ),
            const SizedBox(height: 20),
            Row(
              children: const [
                CircleAvatar(
                  radius: 24,
                  backgroundColor: Color(0xFF1E88E5),
                  child: Text('AD',
                      style: TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold)),
                ),
                SizedBox(width: 12),
                CircleAvatar(
                  radius: 24,
                  backgroundColor: Color(0xFF43A047),
                  child: Text('SM',
                      style: TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold)),
                ),
                SizedBox(width: 12),
                CircleAvatar(
                  radius: 24,
                  backgroundImage:
                      NetworkImage('https://picsum.photos/seed/hero/100/100'),
                ),
              ],
            ),
            const SizedBox(height: 20),
            ClipRRect(
              borderRadius: BorderRadius.circular(16),
              child: Image.network(
                'https://picsum.photos/seed/donjon/600/200',
                height: 140,
                fit: BoxFit.cover,
                errorBuilder: (BuildContext c, Object e, StackTrace? s) =>
                    Container(
                  height: 140,
                  color: Colors.indigo.shade100,
                  alignment: Alignment.center,
                  child: const Icon(Icons.landscape,
                      size: 48, color: Colors.indigo),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _Pastille extends StatelessWidget {
  const _Pastille(this.icone, this.couleur, this.libelle);

  final IconData icone;
  final Color couleur;
  final String libelle;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(
          padding: const EdgeInsets.all(14),
          decoration: BoxDecoration(
            color: couleur.withValues(alpha: 0.15),
            shape: BoxShape.circle,
          ),
          child: Icon(icone, color: couleur, size: 28),
        ),
        const SizedBox(height: 6),
        Text(libelle, style: const TextStyle(fontSize: 12)),
      ],
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ 47.34 — Sans aucun fichier           │
├──────────────────────────────────────┤
│ ╭──────────────────────────────────╮ │
│ │   D O N J O N  D E  C R I S T A L│ │  <- dégradé bleu-violet + ombre
│ ╰──────────────────────────────────╯ │
│                                      │
│  (o)     (o)      (o)      (o)       │
│Defense Attaque Vitesse    Vie        │  <- pastilles colorées
│                                      │
│ (AD) (SM) (photo)                    │  <- avatars
│                                      │
│ ╭──────────────────────────────────╮ │
│ │      paysage picsum.photos       │ │  <- ou icône de repli
│ ╰──────────────────────────────────╯ │
└──────────────────────────────────────┘
```

> Retenez la structure du dernier bloc : `ClipRRect` + `Image.network` + `errorBuilder`. Sans réseau, l'écran reste présentable au lieu d'afficher une exception. C'est le minimum professionnel.

---

## 47.35 — Mini-projet : une carte de profil complète

Nous réunissons tout le chapitre dans un seul écran : une carte de profil de joueur.

**Cahier des charges :**

```text
1. une bannière en dégradé, avec le nom du donjon en surimpression ;
2. un avatar circulaire qui chevauche la bannière ;
3. un nom, un titre, une localisation avec icône ;
4. une biographie limitée à 3 lignes avec points de suspension ;
5. trois statistiques alignées, séparées par des traits verticaux ;
6. une ligne de badges (icône + libellé) ;
7. une galerie de trois vignettes à coins arrondis ;
8. aucun fichier local : tout est couleur, icône, initiales ou picsum ;
9. tous les textes passent par le textTheme.
```

**Le code complet :**

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ProfilApp());
}

/// Le modele de donnees (chapitres 08 et 09).
class Joueur {
  const Joueur({
    required this.nom,
    required this.titre,
    required this.ville,
    required this.bio,
    required this.niveau,
    required this.victoires,
    required this.pieces,
  });

  final String nom;
  final String titre;
  final String ville;
  final String bio;
  final int niveau;
  final int victoires;
  final int pieces;

  String get initiales {
    final List<String> mots = nom.split(' ');
    if (mots.length < 2) return nom.substring(0, 1).toUpperCase();
    return (mots.first[0] + mots.last[0]).toUpperCase();
  }
}

const Joueur kJoueur = Joueur(
  nom: 'Alex Dupont',
  titre: 'Rodeuse des Terres Grises',
  ville: 'Val-des-Brumes',
  bio: 'Exploratrice depuis douze ans, specialiste des donjons souterrains '
      'et des pieges a mecanisme. A survecu trois fois au Gouffre Noir, '
      'ce qui reste un record non homologue par la guilde des aventuriers.',
  niveau: 27,
  victoires: 184,
  pieces: 12450,
);

class ProfilApp extends StatelessWidget {
  const ProfilApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Carte de profil',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: const Color(0xFF3F51B5),
      ),
      home: const EcranProfil(joueur: kJoueur),
    );
  }
}

class EcranProfil extends StatelessWidget {
  const EcranProfil({super.key, required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;
    final ColorScheme c = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Profil'),
        actions: [
          IconButton(
            icon: const Icon(Icons.share),
            tooltip: 'Partager',
            onPressed: () {},
          ),
        ],
      ),
      body: ListView(
        children: [
          // 1 et 2 : banniere + avatar chevauchant
          SizedBox(
            height: 200,
            child: Stack(
              clipBehavior: Clip.none,
              children: [
                Container(
                  height: 150,
                  decoration: const BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.topLeft,
                      end: Alignment.bottomRight,
                      colors: [Color(0xFF303F9F), Color(0xFF7B1FA2)],
                    ),
                  ),
                  alignment: Alignment.topRight,
                  padding: const EdgeInsets.all(16),
                  child: Text(
                    'GOUFFRE NOIR',
                    style: t.labelLarge?.copyWith(
                      color: Colors.white70,
                      letterSpacing: 3,
                    ),
                  ),
                ),
                Positioned(
                  left: 20,
                  bottom: 0,
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: BoxDecoration(
                      color: c.surface,
                      shape: BoxShape.circle,
                    ),
                    child: CircleAvatar(
                      radius: 46,
                      backgroundColor: const Color(0xFF1E88E5),
                      foregroundColor: Colors.white,
                      backgroundImage: const NetworkImage(
                          'https://picsum.photos/seed/alex/200/200'),
                      onBackgroundImageError:
                          (Object e, StackTrace? s) {},
                      child: Text(joueur.initiales,
                          style: t.headlineSmall
                              ?.copyWith(color: Colors.white)),
                    ),
                  ),
                ),
              ],
            ),
          ),

          // 3 : identite
          Padding(
            padding: const EdgeInsets.fromLTRB(20, 12, 20, 0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(joueur.nom, style: t.headlineSmall),
                const SizedBox(height: 2),
                Text(joueur.titre,
                    style: t.titleMedium?.copyWith(color: c.primary)),
                const SizedBox(height: 6),
                Row(
                  children: [
                    Icon(Icons.place, size: 16, color: c.outline),
                    const SizedBox(width: 4),
                    Text(joueur.ville,
                        style: t.bodySmall?.copyWith(color: c.outline)),
                    const SizedBox(width: 16),
                    Icon(Icons.military_tech, size: 16, color: c.outline),
                    const SizedBox(width: 4),
                    Text('Niveau ${joueur.niveau}',
                        style: t.bodySmall?.copyWith(color: c.outline)),
                  ],
                ),
                const SizedBox(height: 14),

                // 4 : biographie tronquee
                Text(
                  joueur.bio,
                  style: t.bodyMedium?.copyWith(height: 1.5),
                  maxLines: 3,
                  overflow: TextOverflow.ellipsis,
                ),
                const SizedBox(height: 18),
              ],
            ),
          ),

          // 5 : statistiques
          IntrinsicHeight(
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _Stat(valeur: '${joueur.niveau}', libelle: 'Niveau'),
                const VerticalDivider(thickness: 1),
                _Stat(valeur: '${joueur.victoires}', libelle: 'Victoires'),
                const VerticalDivider(thickness: 1),
                _Stat(
                  valeur: '${(joueur.pieces / 1000).toStringAsFixed(1)} k',
                  libelle: 'Pieces',
                ),
              ],
            ),
          ),
          const SizedBox(height: 18),

          // 6 : badges
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 20),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Badges', style: t.titleMedium),
                const SizedBox(height: 10),
                Wrap(
                  spacing: 10,
                  runSpacing: 10,
                  children: const [
                    _Badge(Icons.shield, 'Gardienne', Colors.blue),
                    _Badge(Icons.local_fire_department, 'Bruleuse',
                        Colors.deepOrange),
                    _Badge(Icons.bolt, 'Rapide', Colors.amber),
                    _Badge(Icons.explore, 'Exploratrice', Colors.teal),
                  ],
                ),
                const SizedBox(height: 24),
                Text('Derniers donjons', style: t.titleMedium),
                const SizedBox(height: 10),
              ],
            ),
          ),

          // 7 : galerie
          SizedBox(
            height: 110,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 20),
              children: const [
                _Vignette('donjon1', 'Crypte'),
                _Vignette('donjon2', 'Marais'),
                _Vignette('donjon3', 'Tour'),
                _Vignette('donjon4', 'Mine'),
              ],
            ),
          ),
          const SizedBox(height: 30),
        ],
      ),
    );
  }
}

class _Stat extends StatelessWidget {
  const _Stat({required this.valeur, required this.libelle});

  final String valeur;
  final String libelle;

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;
    return Column(
      children: [
        Text(valeur, style: t.headlineSmall),
        const SizedBox(height: 2),
        Text(libelle,
            style: t.labelMedium
                ?.copyWith(color: Theme.of(context).colorScheme.outline)),
      ],
    );
  }
}

class _Badge extends StatelessWidget {
  const _Badge(this.icone, this.libelle, this.couleur);

  final IconData icone;
  final String libelle;
  final Color couleur;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: couleur.withValues(alpha: 0.12),
        borderRadius: BorderRadius.circular(20),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icone, size: 18, color: couleur),
          const SizedBox(width: 6),
          Text(
            libelle,
            style: Theme.of(context)
                .textTheme
                .labelLarge
                ?.copyWith(color: couleur),
          ),
        ],
      ),
    );
  }
}

class _Vignette extends StatelessWidget {
  const _Vignette(this.graine, this.libelle);

  final String graine;
  final String libelle;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(right: 12),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          ClipRRect(
            borderRadius: BorderRadius.circular(12),
            child: Image.network(
              'https://picsum.photos/seed/$graine/160/160',
              width: 110,
              height: 80,
              fit: BoxFit.cover,
              loadingBuilder: (BuildContext c, Widget child,
                  ImageChunkEvent? p) {
                if (p == null) return child;
                return Container(
                  width: 110,
                  height: 80,
                  color: Colors.black12,
                  child: const Center(
                    child: SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    ),
                  ),
                );
              },
              errorBuilder:
                  (BuildContext c, Object e, StackTrace? s) => Container(
                width: 110,
                height: 80,
                color: Colors.black12,
                child: const Icon(Icons.image_not_supported,
                    color: Colors.black38),
              ),
            ),
          ),
          const SizedBox(height: 4),
          Text(libelle,
              style: Theme.of(context).textTheme.labelMedium),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌──────────────────────────────────────┐
│ Profil                      [partage]│
├──────────────────────────────────────┤
│▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ G O U F F R E ▓▓▓│  <- dégradé
│▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ N O I R ▓▓▓▓▓▓│
│  ( photo )                           │  <- avatar à cheval
├──────────────────────────────────────┤
│ Alex Dupont                          │  headlineSmall
│ Rodeuse des Terres Grises            │  titleMedium, couleur primaire
│ [lieu] Val-des-Brumes [medaille] Niv…│  bodySmall gris
│                                      │
│ Exploratrice depuis douze ans,       │
│ specialiste des donjons souterrains  │  3 lignes max
│ et des pieges a mecanisme. A survec… │
│                                      │
│    27     │    184    │   12.5 k     │
│  Niveau   │ Victoires │  Pieces      │
│                                      │
│ Badges                               │
│ (bouclier Gardienne) (flamme Bruleu…)│
│ (eclair Rapide) (boussole Explorat…) │
│                                      │
│ Derniers donjons                     │
│ ╭─────╮ ╭─────╮ ╭─────╮ ╭─────╮      │
│ │Crypt│ │Marai│ │Tour │ │Mine │  ->  │  <- défilement horizontal
│ ╰─────╯ ╰─────╯ ╰─────╯ ╰─────╯      │
└──────────────────────────────────────┘
```

**Ce que ce projet mobilise :**

| Notion | Section | Où dans le code |
| --- | --- | --- |
| `textTheme` | 47.11 | tous les `Text` |
| `copyWith` | 47.2.1 | couleurs et espacements |
| `maxLines` + `overflow` | 47.6 | la biographie |
| `letterSpacing` | 47.4 | « GOUFFRE NOIR » |
| `Icon` | 47.17 | lieu, médaille, badges |
| `IconButton` | 47.19 | partage dans l'AppBar |
| `CircleAvatar` + repli | 47.30 | avatar |
| `Image.network` | 47.25 | vignettes |
| `loadingBuilder` / `errorBuilder` | 47.26 | vignettes |
| `BoxFit.cover` | 47.28 | vignettes |
| `ClipRRect` | 47.29 | vignettes |
| dégradé, ombres, pastilles | 47.34 | bannière, badges |
| classe modèle | chapitres 08-09 | `Joueur` |

> `Stack` avec `clipBehavior: Clip.none` autorise l'avatar à déborder de la bannière. Sans cela, la partie basse du cercle serait rognée. C'est le détail qui fait toute la différence sur ce type de carte.

---

## 47.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: "assets/images/x.png"` | l'asset n'est pas déclaré dans `pubspec.yaml` | ajouter le fichier ou le dossier sous `flutter: assets:` puis relancer l'application |
| L'asset est déclaré mais reste introuvable | `assets:` est à la colonne 0 au lieu d'être indenté sous `flutter:` | remettre deux espaces devant `assets:` (section 47.22) |
| `flutter pub get` échoue avec « mapping values are not allowed here » | une tabulation dans `pubspec.yaml` | remplacer toutes les tabulations par des espaces |
| Un dossier déclaré n'embarque rien | slash final oublié : `- assets/images` | écrire `- assets/images/` |
| Les images d'un sous-dossier manquent | la déclaration d'un dossier n'est pas récursive | déclarer aussi le sous-dossier : `- assets/images/boss/` |
| La police déclarée ne s'applique pas | hot reload au lieu d'un redémarrage complet | arrêter et relancer l'application après toute modification de `pubspec.yaml` |
| `fontWeight: FontWeight.w300` ne change rien | le fichier de graisse 300 n'est pas déclaré | ajouter le `.ttf` correspondant avec `weight: 300`, ou n'utiliser que les graisses disponibles |
| Rayures jaunes et noires sous un `Text` | le texte déborde de la place disponible | ajouter `maxLines` + `overflow: TextOverflow.ellipsis` |
| Un `Text` déborde toujours dans une `Row`, malgré `ellipsis` | la `Row` offre une largeur infinie à ses enfants | entourer le `Text` d'un `Expanded` ou d'un `Flexible` |
| `textAlign: TextAlign.center` ne centre rien | la boîte du `Text` fait la largeur du texte | utiliser `Center`, un `SizedBox(width: double.infinity)` ou `CrossAxisAlignment.stretch` |
| L'image réseau ne s'affiche pas en release Android | permission `INTERNET` absente du manifeste | ajouter `<uses-permission android:name="android.permission.INTERNET"/>` |
| Écran rouge d'exception quand le réseau est coupé | aucun `errorBuilder` sur `Image.network` | fournir systématiquement `errorBuilder`, et de préférence `loadingBuilder` |
| L'image est déformée, les visages écrasés | `BoxFit.fill` sur une boîte de proportions différentes | utiliser `BoxFit.cover` (remplir en coupant) ou `BoxFit.contain` (tout voir) |
| L'image dépasse et cache le reste de l'écran | pas de contrainte de taille et pas de `fit` | imposer `width`/`height`, ou envelopper dans un `SizedBox` |
| `CircleAvatar(backgroundImage: Image.network(url))` ne compile pas | `backgroundImage` attend un `ImageProvider` | écrire `NetworkImage(url)` ou `AssetImage('...')` |
| Le texte blanc est illisible sur la photo de fond | pas de filtre d'assombrissement | ajouter un `colorFilter: ColorFilter.mode(Colors.black.withValues(alpha: 0.45), BlendMode.darken)` |
| L'image de `picsum.photos` change à chaque reconstruction | l'URL aléatoire est rechargée à chaque `build` | utiliser la forme `/id/1015/...` ou `/seed/mot/...` |
| `Image.file` provoque une erreur de compilation web | `dart:io` n'existe pas sur le web | utiliser `Image.network` ou `Image.memory` sur le web |
| Le défilement est saccadé sur une liste d'images | `Opacity` ou `ClipRRect` sur chaque élément | préférer une couleur semi-transparente ou `BoxDecoration(borderRadius:)` |
| Les icônes s'affichent comme des carrés vides | `uses-material-design: true` absent de `pubspec.yaml` | ajouter cette ligne sous `flutter:` |
| Le texte est noir sur fond noir en mode sombre | `color: Colors.black` codé en dur | passer par `Theme.of(context).textTheme` ou `colorScheme` |
| `TapGestureRecognizer` provoque une fuite mémoire | créé dans `build()` et jamais libéré | le créer dans `initState()` et le libérer dans `dispose()` |

---

## 47.37 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `Text` | premier argument positionnel obligatoire, tout le reste est nommé |
| `TextStyle` | objet immuable ; on le dérive avec `copyWith` |
| `fontSize` | en pixels logiques ; 14 pour `bodyMedium` par défaut |
| `fontWeight` | `w100` à `w900` ; ne rend que si le fichier existe |
| `letterSpacing` | espace entre les lettres, en pixels |
| `height` | multiplicateur de `fontSize`, pas une hauteur absolue ; 1.4 à 1.6 pour un paragraphe |
| `decoration` | `underline`, `lineThrough`, `overline`, combinables |
| `textAlign` | aligne **dans** la boîte du `Text` ; sans largeur, aucun effet |
| `maxLines` + `overflow` | le duo à écrire dès qu'un texte peut être long |
| `Text` dans une `Row` | toujours dans un `Expanded` ou un `Flexible` |
| `softWrap` | autorise ou interdit le retour à la ligne automatique |
| `Text.rich` / `TextSpan` | plusieurs styles dans un seul paragraphe, avec héritage |
| `WidgetSpan` | insère un widget (icône) au fil du texte |
| `SelectableText` | texte copiable ; `SelectionArea` pour une zone entière |
| `textTheme` | 15 styles nommés ; `Theme.of(context).textTheme.titleLarge` |
| Thème plutôt que valeurs en dur | cohérence, maintenance, mode sombre, accessibilité |
| Polices | `.ttf` ou `.otf` ; vérifier la licence avant d'embarquer |
| `pubspec.yaml` fonts | `family`, puis liste de `asset` avec `weight` et `style` |
| `google_fonts` | aucune déclaration, mais téléchargement au premier lancement |
| `Icon` / `Icons` | une icône est un glyphe de police, pas une image |
| `IconButton` | `onPressed: null` désactive le bouton |
| Cupertino | second catalogue d'icônes, style iOS |
| Assets | rien n'est embarqué sans déclaration explicite |
| Indentation YAML | espaces uniquement, 2 par niveau, slash final sur un dossier |
| `Image.asset` | chemin exact, casse comprise |
| Résolutions | dossiers `2.0x/`, `3.0x/` ; le code ne change pas |
| `Image.network` | prévoir `loadingBuilder` et `errorBuilder` |
| `cached_network_image` | cache disque persistant, indispensable pour les listes |
| `BoxFit` | `cover` remplit en coupant, `contain` montre tout avec du vide |
| `ClipRRect` | coins arrondis ; parent de l'image |
| `CircleAvatar` | attend un `ImageProvider` ; le `child` sert de repli |
| `Image.memory` / `Image.file` | octets en mémoire / fichier local (pas sur le web) |
| `DecorationImage` | image en fond d'un `Container`, avec `colorFilter` |
| `ColorFiltered` | noir et blanc, teinte, silhouette |
| Sans fichier image | dégradés, pastilles, initiales, `picsum.photos` |

---

## 47.38 — Exercices

Tous ces exercices sont réalisables **sans aucun fichier image local**. Écrivez chaque solution dans un `main.dart` complet.

### Exercice 1 — Fiche d'objet (facile)

Affichez, dans une `Column`, le nom d'un objet en 24 px gras, sa rareté en italique et en violet, et son prix en 16 px. Les trois textes sont alignés à gauche, avec 8 pixels d'écart.

### Exercice 2 — Prix barré (facile)

Affichez sur une même ligne l'ancien prix `150 pieces` barré en rouge, puis le nouveau prix `90 pieces` en gras et vert, séparés par 12 pixels.

### Exercice 3 — Description tronquée (facile)

Affichez une description de 400 caractères limitée à 3 lignes, avec des points de suspension, un interligne de 1.5 et une largeur de 280 pixels.

### Exercice 4 — Phrase multi-styles (moyen)

Avec `Text.rich`, affichez : « Vous avez trouvé **une potion de soin** et gagné [icône étoile] **120 XP**. » Le nom de l'objet est en gras bleu, l'XP en gras orange, et l'étoile est une vraie icône insérée dans le texte.

### Exercice 5 — Barre de vie en icônes (moyen)

Écrivez un widget `BarreDeVie` qui reçoit `vies` et `viesMax` et affiche `viesMax` cœurs, dont les `vies` premiers sont pleins et rouges, les autres vides et gris. Testez-le avec 3 sur 5, 0 sur 5 et 5 sur 5.

### Exercice 6 — Compteur avec `IconButton` (moyen)

Affichez un score et deux `IconButton` (moins et plus) qui le modifient de 10. Le bouton moins est désactivé à 0, le bouton plus à 100. Ajoutez un `IconButton` de remise à zéro dans l'`AppBar`.

### Exercice 7 — Comparateur de `BoxFit` (moyen)

Affichez la même image `https://picsum.photos/id/1025/400/200` dans quatre boîtes de 150 x 150 pixels, sur fond gris, avec `fill`, `contain`, `cover` et `scaleDown`. Étiquetez chaque boîte.

### Exercice 8 — Avatar robuste (difficile)

Écrivez un widget `AvatarJoueur` qui reçoit un nom et une URL éventuellement nulle. S'il y a une URL, il affiche la photo dans un cercle ; sinon, ou en cas d'échec, il affiche les initiales sur un fond de couleur déduite du nom. Testez avec une URL valide, une URL invalide et `null`.

### Exercice 9 — Carte de niveau (difficile)

Créez une carte de 340 x 180 pixels, coins arrondis à 16, avec une image `picsum` en fond assombrie par un `colorFilter`, le titre du niveau en blanc 26 px gras en bas à gauche, et un badge de difficulté en haut à droite. Prévoyez un repli visuel si l'image échoue.

### Exercice 10 — Écran de fin de partie (difficile)

Assemblez un écran complet : un titre `VICTOIRE` en `displaySmall`, trois étoiles dont deux pleines, une ligne de trois statistiques (score, temps, pièces) séparées par des `VerticalDivider`, une phrase `Text.rich` où le score est mis en valeur, et deux boutons-icônes (rejouer, partager). Tous les textes doivent venir du `textTheme`.

---

## 47.39 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 1')),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Epee de givre',
                style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              ),
              SizedBox(height: 8),
              Text(
                'Legendaire',
                style: TextStyle(
                  fontStyle: FontStyle.italic,
                  color: Colors.deepPurple,
                ),
              ),
              SizedBox(height: 8),
              Text('450 pieces', style: TextStyle(fontSize: 16)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** `CrossAxisAlignment.start` aligne les enfants de la `Column` à gauche ; par défaut ils seraient centrés horizontalement. Les `SizedBox(height: 8)` créent les écarts. Comme rien ne dépend de l'état, tout l'arbre est `const`, ce qui permet à Flutter de ne jamais le reconstruire (chapitre 44).

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 2')),
        body: const Center(
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(
                '150 pieces',
                style: TextStyle(
                  fontSize: 18,
                  color: Colors.grey,
                  decoration: TextDecoration.lineThrough,
                  decorationColor: Colors.red,
                  decorationThickness: 2,
                ),
              ),
              SizedBox(width: 12),
              Text(
                '90 pieces',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                  color: Colors.green,
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

**Explication :** `decoration: TextDecoration.lineThrough` barre le texte, et `decorationColor` colore le trait indépendamment du texte, qui reste gris. `mainAxisSize: MainAxisSize.min` empêche la `Row` de s'étirer sur toute la largeur, ce qui permet à `Center` de la centrer réellement.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

const String kDescription =
    'Cette lame a ete forgee dans les profondeurs du mont Kharas par les '
    'derniers artisans de la guilde du fer noir. Sa garde porte encore les '
    'runes de protection gravees a la main, et son tranchant n a jamais '
    'ete emousse malgre trois siecles de batailles continues contre les '
    'creatures du gouffre.';

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 3')),
        body: Center(
          child: Container(
            width: 280,
            color: Colors.amber.shade50,
            padding: const EdgeInsets.all(12),
            child: const Text(
              kDescription,
              maxLines: 3,
              overflow: TextOverflow.ellipsis,
              style: TextStyle(fontSize: 15, height: 1.5),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Explication :** la largeur de 280 pixels est imposée par le `Container`, ce qui donne au `Text` une contrainte connue. `maxLines: 3` limite l'affichage et `TextOverflow.ellipsis` remplace la fin par des points de suspension. `height: 1.5` multiplie la hauteur de ligne par 1,5, soit 22,5 pixels pour une police de 15.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 4')),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Text.rich(
            TextSpan(
              style: TextStyle(fontSize: 18, height: 1.5, color: Colors.black87),
              children: [
                TextSpan(text: 'Vous avez trouve '),
                TextSpan(
                  text: 'une potion de soin',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Colors.blue,
                  ),
                ),
                TextSpan(text: ' et gagne '),
                WidgetSpan(
                  alignment: PlaceholderAlignment.middle,
                  child: Icon(Icons.star, size: 20, color: Colors.amber),
                ),
                TextSpan(text: ' '),
                TextSpan(
                  text: '120 XP',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Colors.deepOrange,
                  ),
                ),
                TextSpan(text: '.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Explication :** le `TextSpan` racine porte le style commun (taille, interligne, couleur de base) ; chaque enfant n'ajoute que ce qui le distingue. `WidgetSpan` insère une véritable `Icon` dans le flux du texte, et `PlaceholderAlignment.middle` la centre verticalement sur la ligne. Sans cet alignement, l'icône paraîtrait posée trop bas.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class BarreDeVie extends StatelessWidget {
  const BarreDeVie({
    super.key,
    required this.vies,
    required this.viesMax,
    this.taille = 28,
  });

  final int vies;
  final int viesMax;
  final double taille;

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: List<Widget>.generate(viesMax, (int i) {
        final bool plein = i < vies;
        return Icon(
          plein ? Icons.favorite : Icons.favorite_border,
          color: plein ? Colors.red : Colors.grey,
          size: taille,
          semanticLabel: plein ? 'vie restante' : 'vie perdue',
        );
      }),
    );
  }
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 5')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: const [
              Text('3 sur 5'),
              BarreDeVie(vies: 3, viesMax: 5),
              SizedBox(height: 20),
              Text('0 sur 5'),
              BarreDeVie(vies: 0, viesMax: 5),
              SizedBox(height: 20),
              Text('5 sur 5'),
              BarreDeVie(vies: 5, viesMax: 5),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** `List<Widget>.generate(viesMax, ...)` construit la liste d'icônes en une expression (chapitre 06). La condition `i < vies` détermine si le cœur est plein. Le widget est paramétré par `vies`, `viesMax` et `taille` : il est donc réutilisable pour les points de vie du joueur comme pour ceux d'un ennemi. Le `semanticLabel` rend la jauge compréhensible pour un lecteur d'écran.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class App extends StatefulWidget {
  const App({super.key});

  @override
  State<App> createState() => _AppState();
}

class _AppState extends State<App> {
  int _score = 50;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.teal),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Exercice 6'),
          actions: [
            IconButton(
              icon: const Icon(Icons.restart_alt),
              tooltip: 'Remettre a zero',
              onPressed: () => setState(() => _score = 0),
            ),
          ],
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('$_score',
                  style: Theme.of(context).textTheme.displayMedium),
              const SizedBox(height: 8),
              Text('points',
                  style: Theme.of(context).textTheme.labelLarge),
              const SizedBox(height: 24),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  IconButton.filledTonal(
                    iconSize: 32,
                    icon: const Icon(Icons.remove),
                    tooltip: 'Retirer 10',
                    onPressed: _score >= 10
                        ? () => setState(() => _score -= 10)
                        : null,
                  ),
                  const SizedBox(width: 24),
                  IconButton.filled(
                    iconSize: 32,
                    icon: const Icon(Icons.add),
                    tooltip: 'Ajouter 10',
                    onPressed: _score <= 90
                        ? () => setState(() => _score += 10)
                        : null,
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** l'état `_score` vit dans le `State` et chaque modification passe par `setState` (chapitre 45). Aux bornes, `onPressed` reçoit `null` : Flutter grise alors le bouton automatiquement, ce qui informe l'utilisateur sans aucune ligne d'affichage supplémentaire. Les tailles de texte viennent du `textTheme`, conformément à la section 47.12.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

const String kUrl = 'https://picsum.photos/id/1025/400/200';

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    const List<BoxFit> fits = <BoxFit>[
      BoxFit.fill,
      BoxFit.contain,
      BoxFit.cover,
      BoxFit.scaleDown,
    ];

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 7')),
        body: Center(
          child: Wrap(
            spacing: 16,
            runSpacing: 16,
            children: fits.map((BoxFit fit) {
              return Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Container(
                    width: 150,
                    height: 150,
                    color: Colors.grey.shade300,
                    child: Image.network(
                      kUrl,
                      fit: fit,
                      errorBuilder: (BuildContext c, Object e,
                              StackTrace? s) =>
                          const Icon(Icons.broken_image, size: 40),
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text('BoxFit.${fit.name}',
                      style: const TextStyle(fontSize: 12)),
                ],
              );
            }).toList(),
          ),
        ),
      ),
    );
  }
}
```

**Explication :** `fits.map(...).toList()` transforme la liste d'énumérations en liste de widgets (chapitre 14). Le `Container` gris agit comme révélateur : le gris visible correspond exactement à la partie de la boîte que l'image ne remplit pas. `fit.name` donne le nom de la valeur d'énumération sous forme de chaîne, sans écrire de `switch`.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class AvatarJoueur extends StatelessWidget {
  const AvatarJoueur({
    super.key,
    required this.nom,
    this.urlPhoto,
    this.rayon = 32,
  });

  final String nom;
  final String? urlPhoto;
  final double rayon;

  static const List<Color> _palette = <Color>[
    Color(0xFF1E88E5),
    Color(0xFF43A047),
    Color(0xFFE53935),
    Color(0xFF8E24AA),
    Color(0xFFF4511E),
    Color(0xFF00897B),
  ];

  String get _initiales {
    final List<String> mots = nom
        .trim()
        .split(RegExp(r'\s+'))
        .where((String m) => m.isNotEmpty)
        .toList();
    if (mots.isEmpty) return '?';
    if (mots.length == 1) return mots.first[0].toUpperCase();
    return (mots.first[0] + mots.last[0]).toUpperCase();
  }

  Color get _couleur {
    int somme = 0;
    for (int i = 0; i < nom.length; i++) {
      somme += nom.codeUnitAt(i);
    }
    return _palette[somme % _palette.length];
  }

  @override
  Widget build(BuildContext context) {
    final Widget secours = Text(
      _initiales,
      style: TextStyle(
        color: Colors.white,
        fontSize: rayon * 0.7,
        fontWeight: FontWeight.bold,
      ),
    );

    return CircleAvatar(
      radius: rayon,
      backgroundColor: _couleur,
      backgroundImage:
          urlPhoto == null ? null : NetworkImage(urlPhoto!),
      onBackgroundImageError:
          urlPhoto == null ? null : (Object e, StackTrace? s) {},
      child: secours,
    );
  }
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 8')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: const [
              AvatarJoueur(
                nom: 'Alex Dupont',
                urlPhoto: 'https://picsum.photos/seed/alex/200/200',
              ),
              SizedBox(height: 8),
              Text('URL valide'),
              SizedBox(height: 24),
              AvatarJoueur(
                nom: 'Sophie Martin',
                urlPhoto: 'https://picsum.photos/introuvable.jpg',
              ),
              SizedBox(height: 8),
              Text('URL invalide'),
              SizedBox(height: 24),
              AvatarJoueur(nom: 'Samir Bennani'),
              SizedBox(height: 8),
              Text('Aucune URL'),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** le `child` d'un `CircleAvatar` reste visible tant que `backgroundImage` n'a pas été chargée avec succès ; il joue donc naturellement le rôle de repli. `onBackgroundImageError` doit être fourni dès qu'il y a une image, sinon l'échec remonte comme une exception non traitée. Les deux accesseurs `_initiales` et `_couleur` sont des getters (chapitre 10) : ils calculent leur valeur à la demande, sans stockage.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class CarteNiveau extends StatelessWidget {
  const CarteNiveau({
    super.key,
    required this.titre,
    required this.difficulte,
    required this.graine,
  });

  final String titre;
  final String difficulte;
  final String graine;

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;

    return ClipRRect(
      borderRadius: BorderRadius.circular(16),
      child: SizedBox(
        width: 340,
        height: 180,
        child: Stack(
          fit: StackFit.expand,
          children: [
            Image.network(
              'https://picsum.photos/seed/$graine/680/360',
              fit: BoxFit.cover,
              color: Colors.black.withValues(alpha: 0.45),
              colorBlendMode: BlendMode.darken,
              errorBuilder: (BuildContext c, Object e, StackTrace? s) =>
                  const ColoredBox(color: Color(0xFF37474F)),
            ),
            Positioned(
              left: 16,
              bottom: 16,
              child: Text(
                titre,
                style: t.headlineSmall?.copyWith(
                  color: Colors.white,
                  fontSize: 26,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            Positioned(
              right: 12,
              top: 12,
              child: Container(
                padding:
                    const EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                decoration: BoxDecoration(
                  color: Colors.white.withValues(alpha: 0.9),
                  borderRadius: BorderRadius.circular(20),
                ),
                child: Text(difficulte,
                    style: t.labelMedium
                        ?.copyWith(fontWeight: FontWeight.bold)),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 9')),
        body: ListView(
          padding: const EdgeInsets.all(16),
          children: const [
            CarteNiveau(
                titre: 'Crypte oubliee',
                difficulte: 'FACILE',
                graine: 'crypte'),
            SizedBox(height: 16),
            CarteNiveau(
                titre: 'Gouffre noir',
                difficulte: 'DIFFICILE',
                graine: 'gouffre'),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** l'assombrissement est obtenu ici par `color` + `colorBlendMode` directement sur l'`Image`, sans passer par un `ColorFiltered` supplémentaire : c'est équivalent et moins coûteux. `Stack` avec `StackFit.expand` fait remplir toute la carte par l'image, puis les deux `Positioned` placent le titre et le badge. `ClipRRect` enveloppe l'ensemble pour que l'image aussi soit arrondie.

---

### Correction 10

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: const EcranFin(score: 12480, secondes: 214, pieces: 340),
    );
  }
}

class EcranFin extends StatelessWidget {
  const EcranFin({
    super.key,
    required this.score,
    required this.secondes,
    required this.pieces,
  });

  final int score;
  final int secondes;
  final int pieces;

  String get _temps {
    final int m = secondes ~/ 60;
    final int s = secondes % 60;
    return '$m:${s.toString().padLeft(2, '0')}';
  }

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;
    final ColorScheme c = Theme.of(context).colorScheme;

    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('VICTOIRE',
                  style: t.displaySmall?.copyWith(
                    color: c.primary,
                    fontWeight: FontWeight.bold,
                    letterSpacing: 3,
                  )),
              const SizedBox(height: 16),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: List<Widget>.generate(3, (int i) {
                  return Icon(
                    i < 2 ? Icons.star : Icons.star_border,
                    size: 48,
                    color: i < 2 ? Colors.amber : Colors.grey,
                  );
                }),
              ),
              const SizedBox(height: 32),
              IntrinsicHeight(
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    _Stat(valeur: '$score', libelle: 'Score'),
                    const VerticalDivider(thickness: 1),
                    _Stat(valeur: _temps, libelle: 'Temps'),
                    const VerticalDivider(thickness: 1),
                    _Stat(valeur: '$pieces', libelle: 'Pieces'),
                  ],
                ),
              ),
              const SizedBox(height: 32),
              Text.rich(
                TextSpan(
                  style: t.bodyLarge?.copyWith(height: 1.5),
                  children: [
                    const TextSpan(text: 'Vous terminez avec '),
                    TextSpan(
                      text: '$score points',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        color: c.primary,
                      ),
                    ),
                    const TextSpan(
                        text: ', soit votre meilleur resultat sur ce '
                            'niveau.'),
                  ],
                ),
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 32),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  IconButton.filled(
                    iconSize: 32,
                    icon: const Icon(Icons.replay),
                    tooltip: 'Rejouer',
                    onPressed: () {},
                  ),
                  const SizedBox(width: 24),
                  IconButton.outlined(
                    iconSize: 32,
                    icon: const Icon(Icons.share),
                    tooltip: 'Partager',
                    onPressed: () {},
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class _Stat extends StatelessWidget {
  const _Stat({required this.valeur, required this.libelle});

  final String valeur;
  final String libelle;

  @override
  Widget build(BuildContext context) {
    final TextTheme t = Theme.of(context).textTheme;
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Text(valeur, style: t.headlineSmall),
        const SizedBox(height: 2),
        Text(libelle,
            style: t.labelMedium
                ?.copyWith(color: Theme.of(context).colorScheme.outline)),
      ],
    );
  }
}
```

**Explication :** tous les styles proviennent du `textTheme` et toutes les couleurs du `colorScheme` : l'écran s'adaptera automatiquement à un changement de thème ou au mode sombre (chapitre 51). `IntrinsicHeight` est indispensable pour que les `VerticalDivider` connaissent leur hauteur, sinon ils ne s'affichent pas. Le getter `_temps` convertit les secondes en `m:ss` avec `padLeft` (chapitre 02). `List<Widget>.generate(3, ...)` produit les trois étoiles.

---

## Et maintenant ?

Vous savez désormais habiller une interface : texte maîtrisé jusqu'à l'interligne, polices déclarées ou téléchargées, icônes, assets déclarés sans erreur YAML, images locales et distantes avec gestion du chargement et de l'échec, cadrage, découpe et avatars. Vous savez surtout produire un rendu propre **sans posséder le moindre fichier image**.

Il manque une chose : jusqu'ici, vos écrans affichaient un nombre fixe d'éléments, écrits à la main. Or une application réelle affiche des listes : cent objets d'inventaire, cinquante quêtes, une grille de niveaux. Les construire une par une dans une `Column` est impossible, et surtout catastrophique pour les performances.

Le chapitre suivant répond à ce problème avec `ListView`, `ListView.builder`, `ListView.separated`, `GridView`, `ListTile`, `Card` et `Dismissible`. Vous y réutiliserez directement tout ce chapitre : chaque élément de liste est une composition de texte, d'icônes et d'images.

Chapitre suivant : [48-PARTIE-1B—LISTES-LISTVIEW-ET-GRIDVIEW.md](./48-PARTIE-1B—LISTES-LISTVIEW-ET-GRIDVIEW.md)
