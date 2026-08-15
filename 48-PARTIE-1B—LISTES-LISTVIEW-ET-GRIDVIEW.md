# PARTIE 1B — FLUTTER
# CHAPITRE 48 — LISTES : LISTVIEW, GRIDVIEW ET DÉFILEMENT

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** chapitres 43 à 47 (installation, widgets, état, layouts, texte et images) ; chapitres 06, 08, 09 et 14 de la PARTIE 1A (collections, classes, constructeurs, `map`/`where`/`sort`)
> **Ce que vous saurez faire à la fin :** afficher n'importe quelle collection Dart à l'écran sous forme de liste ou de grille performante, la filtrer, la trier, la rafraîchir, la réordonner et en supprimer des éléments.

---

## 48.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi une `Column` ne suffit pas pour afficher une collection ;
- utiliser `SingleChildScrollView` et en connaître la limite exacte ;
- construire une `ListView` avec `children` ;
- construire une `ListView.builder` avec `itemCount` et `itemBuilder` ;
- expliquer, chiffres à l'appui, pourquoi `.builder` est indispensable ;
- séparer les éléments avec `ListView.separated` ;
- utiliser `ListTile` et ses cinq zones (`leading`, `title`, `subtitle`, `trailing`, `onTap`) ;
- envelopper une tuile dans une `Card` ;
- écrire votre propre tuile personnalisée ;
- afficher une `List<Joueur>` construite avec les classes du chapitre 08 ;
- décider quand utiliser `shrinkWrap` et `physics`, et quand ne surtout pas les utiliser ;
- diagnostiquer et corriger le message `Vertical viewport was given unbounded height.` ;
- piloter le défilement avec un `ScrollController` ;
- détecter l'arrivée en bas de liste pour charger la page suivante ;
- ajouter un bouton « remonter en haut » ;
- construire une grille avec `GridView.count` ;
- construire une grille performante avec `GridView.builder` et un `SliverGridDelegate` ;
- régler `crossAxisCount`, `childAspectRatio`, `mainAxisSpacing` et `crossAxisSpacing` ;
- rendre une grille responsive avec `SliverGridDelegateWithMaxCrossAxisExtent` ;
- supprimer un élément par glissement avec `Dismissible` ;
- expliquer pourquoi la `Key` d'un `Dismissible` est obligatoire ;
- proposer une annulation avec un `SnackBar` et son `SnackBarAction` ;
- rafraîchir une liste avec `RefreshIndicator` ;
- réordonner une liste avec `ReorderableListView` ;
- soigner l'état vide de vos écrans ;
- filtrer et trier une liste affichée ;
- brancher une barre de recherche sur une liste ;
- introduire `CustomScrollView` et les slivers ;
- replier un en-tête avec `SliverAppBar` ;
- appliquer les bonnes pratiques de performance (`const`, `itemExtent`, cache d'images) ;
- réaliser un écran complet « liste d'ennemis » filtrable, triable et supprimable.

---

## 48.0.1 — Ce que le chapitre suppose déjà connu

Ce chapitre s'appuie directement sur des notions vues auparavant. Un rappel rapide vous évitera de bloquer.

```text
PARTIE 1A
  chapitre 06 : List, Set, Map, boucle for-in, .length, .add, .removeAt
  chapitre 08 : class, propriétés, méthodes
  chapitre 09 : constructeurs, paramètres nommés, required
  chapitre 14 : map, where, toList, sort, fold, any, every
  chapitre 15 : Future, async, await          (pour 48.18 et 48.27)

PARTIE 1B
  chapitre 44 : arbre de widgets, BuildContext, MaterialApp, Scaffold
  chapitre 45 : StatelessWidget, StatefulWidget, setState, initState, dispose
  chapitre 46 : Column, Row, Expanded, Container, Padding, SizedBox, contraintes
  chapitre 47 : Text, TextStyle, Icon, CircleAvatar, Image.network
```

Si l'un de ces points est flou, relisez-le. Le chapitre 46 (les contraintes) est le plus important des quatre : les erreurs de listes sont presque toujours des erreurs de contraintes.

---

## 48.1 — Afficher une collection à l'écran (rappel chapitre 06)

Au chapitre 06, vous avez appris à ranger des données dans une `List` :

```dart
void main() {
  List<String> inventaire = ['Épée', 'Bouclier', 'Potion', 'Arc', 'Torche'];

  for (String objet in inventaire) {
    print(objet);
  }
}
```

**Résultat :**

```text
Épée
Bouclier
Potion
Arc
Torche
```

En console, une boucle suffit. Dans une application Flutter, il n'y a pas de console : il faut **transformer chaque élément de la liste en widget**.

Le problème est donc toujours le même, quelle que soit l'application :

```text
DONNÉES                             INTERFACE

List<String>                        ┌──────────────────┐
  [0] 'Épée'          ────────>     │  Épée            │
  [1] 'Bouclier'      ────────>     │  Bouclier        │
  [2] 'Potion'        ────────>     │  Potion          │
  [3] 'Arc'           ────────>     │  Arc             │
  [4] 'Torche'        ────────>     │  Torche          │
                                    └──────────────────┘

     1 élément  =  1 widget
```

Tout ce chapitre répond à une seule question : **comment fabriquer ces widgets, efficacement, et les faire défiler ?**

---

## 48.1.1 — La conversion naïve : `map` puis `toList`

Le chapitre 14 vous a donné l'outil exact pour transformer une liste en une autre liste :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Inventaire',
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = <String>[
      'Épée',
      'Bouclier',
      'Potion',
      'Arc',
      'Torche',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: inventaire.map((String objet) => Text(objet)).toList(),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Inventaire               │
├──────────────────────────┤
│ Épée                     │
│ Bouclier                 │
│ Potion                   │
│ Arc                      │
│ Torche                   │
│                          │
└──────────────────────────┘
```

Ce code fonctionne. Il est même parfaitement correct pour cinq éléments.

Décomposons la ligne clé :

```text
inventaire                          List<String>
  .map((objet) => Text(objet))      Iterable<Text>
  .toList()                         List<Text>   <- ce que children attend
```

`children` attend un `List<Widget>`. Comme `Text` est un `Widget`, `List<Text>` est accepté.

> **Remarque.** N'oubliez jamais le `.toList()`. `map` renvoie un `Iterable`, pas une `List`. Sans lui, le compilateur refuse : `The argument type 'Iterable<Text>' can't be assigned to the parameter type 'List<Widget>'`.

---

## 48.2 — `Column` + `SingleChildScrollView` : la solution naïve et sa limite

Ajoutons des éléments. Passons de 5 à 40 objets.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = List<String>.generate(
      40,
      (int i) => 'Objet numéro ${i + 1}',
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: inventaire.map((String objet) => Text(objet)).toList(),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Inventaire               │
├──────────────────────────┤
│ Objet numéro 1           │
│ Objet numéro 2           │
│ ...                      │
│ Objet numéro 23          │
│▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│  <- rayures jaunes et noires
└──────────────────────────┘

Console :
A RenderFlex overflowed by 214 pixels on the bottom.
```

Une `Column` ne défile pas. Elle empile ses enfants et, quand la somme des hauteurs dépasse la place disponible, elle déborde et affiche la barre rayée jaune et noire.

---

## 48.2.1 — Première correction : `SingleChildScrollView`

`SingleChildScrollView` est un widget qui prend **un seul enfant** et le rend défilant.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = List<String>.generate(
      40,
      (int i) => 'Objet numéro ${i + 1}',
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: inventaire.map((String objet) => Text(objet)).toList(),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Inventaire               │
├──────────────────────────┤
│ Objet numéro 1           │
│ Objet numéro 2           │
│ Objet numéro 3           │  <- on peut maintenant faire défiler
│ ...                      │     jusqu'à « Objet numéro 40 »
└──────────────────────────┘

Aucune erreur en console.
```

Le débordement disparaît. L'écran défile.

---

## 48.2.2 — La limite : tout est construit, tout est monté

Voici pourquoi cette solution ne passe pas à l'échelle.

`SingleChildScrollView` **place son enfant dans une hauteur infinie**, puis fait glisser une fenêtre devant lui. Conséquence : la `Column` doit être **entièrement construite et mesurée**, même la partie invisible.

```text
        SingleChildScrollView + Column
        =====================================

        ╔══════════════════╗ <- ce que l'utilisateur voit
        ║ Objet 1          ║
        ║ Objet 2          ║      widgets CONSTRUITS   : 40 000
        ║ Objet 3          ║      widgets AFFICHÉS     :     12
        ╚══════════════════╝      widgets INUTILES     : 39 988
          Objet 4
          Objet 5
          ...                     mémoire : tout
          Objet 39 999
          Objet 40 000
```

Avec 40 objets, personne ne s'en aperçoit. Avec 40 000, l'application se fige à l'ouverture de l'écran, ou plante avec une erreur mémoire.

Le tableau suivant résume le comportement observé (ordre de grandeur, appareil de milieu de gamme) :

| Nombre d'éléments | `SingleChildScrollView` + `Column` | `ListView.builder` |
| --- | --- | --- |
| 20 | instantané | instantané |
| 500 | ~150 ms de latence à l'ouverture | instantané |
| 5 000 | 1 à 3 s de gel | instantané |
| 100 000 | inutilisable, souvent un plantage | instantané |

**Règle à retenir dès maintenant :**

> `SingleChildScrollView` + `Column` convient pour un contenu **fini et court** que l'on connaît à l'avance : un formulaire, une page « À propos », un écran de réglages de dix lignes.
> Pour une **collection**, on utilise une `ListView`.

---

## 48.3 — `ListView`

`ListView` est le widget de liste défilante de Flutter. C'est un **viewport** : une fenêtre qui montre une portion d'un contenu plus grand.

Trois différences majeures avec `SingleChildScrollView` + `Column` :

```text
1. ListView défile TOUTE SEULE.
   Pas besoin de l'envelopper dans quoi que ce soit.

2. ListView occupe TOUT l'espace disponible dans la direction du défilement.
   Elle ne s'ajuste pas au contenu : elle prend tout, puis fait défiler.

3. ListView peut construire ses enfants À LA DEMANDE (avec .builder),
   c'est-à-dire seulement ceux qui sont visibles ou presque.
```

Le premier exemple, en une ligne de moins que la version précédente :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = List<String>.generate(
      40,
      (int i) => 'Objet numéro ${i + 1}',
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: ListView(
        children: inventaire.map((String objet) => Text(objet)).toList(),
      ),
    );
  }
}
```

**Résultat :** identique à la version `SingleChildScrollView`, mais sans le widget englobant.

---

## 48.3.1 — Le vocabulaire du défilement

Trois mots reviendront tout au long du chapitre.

```text
  ┌───────────────────────────────┐  ← le VIEWPORT (la fenêtre)
  │  Objet 5                      │     hauteur = viewportDimension
  │  Objet 6                      │
  │  Objet 7                      │
  └───────────────────────────────┘

  Objet 1    ┐
  Objet 2    │  extentBefore : ce qui est passé au-dessus
  Objet 3    │
  Objet 4    ┘
  ...
  Objet 38   ┐
  Objet 39   │  extentAfter : ce qui reste en-dessous
  Objet 40   ┘

  pixels          : position actuelle du défilement (0 = tout en haut)
  maxScrollExtent : position maximale atteignable (tout en bas)
```

| Terme | Signification |
| --- | --- |
| viewport | la zone visible, la « fenêtre » |
| axe principal (`mainAxis`) | l'axe du défilement : vertical par défaut |
| axe transversal (`crossAxis`) | l'axe perpendiculaire : horizontal par défaut |
| `pixels` | la position de défilement courante, en pixels logiques |
| `maxScrollExtent` | la position de défilement maximale |

---

## 48.4 — `ListView` avec `children`

Le constructeur par défaut de `ListView` prend une liste de widgets, exactement comme `Column` :

```dart
ListView({
  Key? key,
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController? controller,
  ScrollPhysics? physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry? padding,
  double? itemExtent,
  List<Widget> children = const <Widget>[],
  // ... et d'autres paramètres vus plus loin
})
```

Exemple complet avec des enfants écrits à la main :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationMenu());
}

class ApplicationMenu extends StatelessWidget {
  const ApplicationMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranMenu(),
    );
  }
}

class EcranMenu extends StatelessWidget {
  const EcranMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu principal')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: const <Widget>[
          Text('Nouvelle partie', style: TextStyle(fontSize: 22)),
          SizedBox(height: 12),
          Text('Charger une partie', style: TextStyle(fontSize: 22)),
          SizedBox(height: 12),
          Text('Options', style: TextStyle(fontSize: 22)),
          SizedBox(height: 12),
          Text('Quitter', style: TextStyle(fontSize: 22)),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Menu principal           │
├──────────────────────────┤
│                          │
│  Nouvelle partie         │
│                          │
│  Charger une partie      │
│                          │
│  Options                 │
│                          │
│  Quitter                 │
│                          │
└──────────────────────────┘
```

---

## 48.4.1 — Quand utiliser `children` ?

`ListView(children: ...)` construit **immédiatement tous** les widgets de la liste. C'est acceptable dans un cas précis :

> Utilisez `ListView(children: ...)` quand le nombre d'enfants est **petit, fixe et connu à l'écriture du code**.

Typiquement : un menu, un écran de réglages, un panneau d'aide.

Dans **tous les autres cas** — dès que les enfants viennent d'une collection dont la taille varie — utilisez `ListView.builder`.

---

## 48.4.2 — Le défilement horizontal

Un paramètre suffit pour coucher la liste :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationArmes());
}

class ApplicationArmes extends StatelessWidget {
  const ApplicationArmes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranArmes(),
    );
  }
}

class EcranArmes extends StatelessWidget {
  const EcranArmes({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> armes = <String>[
      'Épée',
      'Hache',
      'Arc',
      'Dague',
      'Masse',
      'Lance',
      'Bâton',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Armes disponibles')),
      body: Column(
        children: <Widget>[
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('Faites glisser horizontalement'),
          ),
          SizedBox(
            height: 120,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 12),
              children: armes.map((String arme) {
                return Container(
                  width: 100,
                  margin: const EdgeInsets.all(8),
                  alignment: Alignment.center,
                  decoration: BoxDecoration(
                    color: Colors.indigo.shade100,
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Text(arme),
                );
              }).toList(),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────┐
│ Armes disponibles              │
├────────────────────────────────┤
│ Faites glisser horizontalement │
│                                │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌───  │
│  │Épée │ │Hache│ │ Arc │ │Dag  │ →
│  └─────┘ └─────┘ └─────┘ └───  │
│                                │
└────────────────────────────────┘
```

> **Point crucial.** Le `SizedBox(height: 120)` n'est pas décoratif. Une `ListView` horizontale placée dans une `Column` reçoit une hauteur **non contrainte** et provoque l'erreur `RenderBox was not laid out`. Nous détaillons ce mécanisme en 48.16.

---

## 48.5 — `ListView.builder`

`ListView.builder` est le constructeur que vous utiliserez dans 90 % des cas.

Au lieu de recevoir une liste de widgets déjà construits, il reçoit une **recette** :

```text
ListView(children: [...])          ListView.builder(...)
=========================          ======================

« Voici les 40 000 widgets,        « Il y a 40 000 éléments.
  débrouille-toi. »                  Quand tu as besoin du
                                     numéro i, appelle cette
                                     fonction et elle te le
                                     fabriquera. »

Construction : immédiate,          Construction : à la demande,
tout, d'un coup.                   uniquement le visible.
```

Signature (paramètres essentiels) :

```dart
ListView.builder({
  Key? key,
  required Widget? Function(BuildContext context, int index) itemBuilder,
  int? itemCount,
  Axis scrollDirection = Axis.vertical,
  ScrollController? controller,
  ScrollPhysics? physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry? padding,
  double? itemExtent,
  Widget? prototypeItem,
})
```

Premier exemple complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = <String>[
      'Épée courte',
      'Bouclier de bois',
      'Potion de soin',
      'Arc long',
      'Torche',
      'Corde',
      'Clé rouillée',
      'Grimoire',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: ListView.builder(
        itemCount: inventaire.length,
        itemBuilder: (BuildContext context, int index) {
          return Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
            child: Text('${index + 1}. ${inventaire[index]}'),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Inventaire               │
├──────────────────────────┤
│  1. Épée courte          │
│  2. Bouclier de bois     │
│  3. Potion de soin       │
│  4. Arc long             │
│  5. Torche               │
│  6. Corde                │
│  7. Clé rouillée         │
│  8. Grimoire             │
└──────────────────────────┘
```

---

## 48.5.1 — Ce que Flutter fait réellement

Suivons l'exécution pas à pas.

```text
1. Flutter mesure le viewport : 600 pixels de haut.

2. Il appelle itemBuilder(context, 0)  -> widget de 44 px. Total : 44
   Il appelle itemBuilder(context, 1)  -> widget de 44 px. Total : 88
   ...
   Il appelle itemBuilder(context, 13) -> widget de 44 px. Total : 616

3. 616 > 600 : le viewport est rempli. Il s'arrête.
   itemBuilder n'a JAMAIS été appelé pour l'index 14, 15, ... 39 999.

4. L'utilisateur fait défiler de 100 pixels.
   Flutter appelle itemBuilder(context, 14), (context, 15)
   et DÉTRUIT les widgets 0 et 1 sortis par le haut.
```

Conséquence directe et souvent surprenante pour un débutant :

> `itemBuilder` est appelé **plusieurs fois** pour le même index au cours de la vie de l'écran, et il n'est **jamais** appelé pour les index jamais atteints.

`itemBuilder` doit donc être une fonction **pure et rapide** : elle construit un widget, point. On n'y écrit jamais de `print` de débogage définitif, jamais d'appel réseau, jamais de calcul lourd.

---

## 48.6 — `itemCount` et `itemBuilder`

Ces deux paramètres méritent une section à eux seuls.

### `itemCount`

`itemCount` est le **nombre d'éléments de la liste**, pas le dernier index.

```text
List<String> inventaire = ['Épée', 'Arc', 'Potion'];

inventaire.length  =  3        <- c'est itemCount
index valides      =  0, 1, 2  <- ce que reçoit itemBuilder
dernier index      =  2        <- length - 1
```

Écrivez donc **toujours** :

```dart
itemCount: inventaire.length,
```

et jamais un nombre écrit en dur. Un nombre en dur devient faux dès qu'on ajoute ou supprime un élément.

### `itemCount` est facultatif — et c'est un piège

`itemCount` est de type `int?`. Il peut donc valoir `null`, ce qui signifie **liste infinie**.

```dart
ListView.builder(
  // pas de itemCount : liste infinie
  itemBuilder: (BuildContext context, int index) {
    return ListTile(title: Text('Ligne $index'));
  },
)
```

C'est utile pour un calendrier perpétuel ou un défilement sans fin. Mais si vous oubliez `itemCount` sur une liste finie, `itemBuilder` sera appelé avec des index de plus en plus grands et vous obtiendrez :

```text
RangeError (index): Index out of range: index should be less than 8: 8
```

### `itemBuilder`

Sa signature exacte est `Widget? Function(BuildContext context, int index)`.

Trois règles :

1. **Elle reçoit un `index` compris entre `0` et `itemCount - 1`.**
2. **Elle doit retourner un widget** (le type autorise `null` mais retourner `null` signale « plus rien à construire », ce qui n'a de sens que sans `itemCount`).
3. **Elle ne doit rien faire d'autre** que construire.

```dart
itemBuilder: (BuildContext context, int index) {
  final String objet = inventaire[index];   // 1. je lis la donnée
  return Text(objet);                       // 2. je fabrique le widget
},
```

Deux lignes. C'est le modèle mental à garder.

---

## 48.6.1 — Le paramètre `context` de `itemBuilder`

Le `context` reçu par `itemBuilder` **n'est pas** le même que celui de `build`. C'est le contexte de la ligne en cours de construction, situé plus bas dans l'arbre.

Cela a une conséquence pratique : si vous devez appeler `Theme.of(...)` ou `MediaQuery.of(...)` **dans** la ligne, utilisez le `context` de `itemBuilder`.

```dart
itemBuilder: (BuildContext context, int index) {
  return Container(
    color: Theme.of(context).colorScheme.surfaceContainerHighest,
    padding: const EdgeInsets.all(12),
    child: Text(inventaire[index]),
  );
},
```

> **Convention.** Beaucoup de développeurs nomment ce paramètre `context` sans réfléchir. Si vous préférez éviter toute confusion, nommez-le `itemContext`. Les deux sont corrects.

---

## 48.7 — Pourquoi `.builder` est indispensable sur une longue liste

Faisons l'expérience. Le programme suivant compte le nombre d'appels à `itemBuilder` et l'affiche.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationCompteur());
}

class ApplicationCompteur extends StatelessWidget {
  const ApplicationCompteur({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranCompteur(),
    );
  }
}

class EcranCompteur extends StatefulWidget {
  const EcranCompteur({super.key});

  @override
  State<EcranCompteur> createState() => _EcranCompteurState();
}

class _EcranCompteurState extends State<EcranCompteur> {
  int _appels = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('itemBuilder appelé $_appels fois')),
      body: ListView.builder(
        itemCount: 100000,
        itemBuilder: (BuildContext context, int index) {
          _appels++;
          return Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
            child: Text('Ennemi n°$index'),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() {}),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

**Résultat :**

```text
Au premier affichage, appuyez sur le bouton de rafraîchissement :

┌──────────────────────────────────┐
│ itemBuilder appelé 18 fois       │  <- pour 100 000 éléments !
├──────────────────────────────────┤
│  Ennemi n°0                      │
│  Ennemi n°1                      │
│  ...                             │
└──────────────────────────────────┘

Après avoir fait défiler jusqu'en bas puis rafraîchi :

┌──────────────────────────────────┐
│ itemBuilder appelé 100000+ fois  │  <- seulement si l'on a tout parcouru
└──────────────────────────────────┘
```

Le chiffre exact varie selon la taille de l'écran (c'est le viewport plus le `cacheExtent`, une marge de 250 pixels par défaut). L'ordre de grandeur, lui, est toujours le même : **quelques dizaines**, pas cent mille.

---

## 48.7.1 — Le tableau de décision

| Situation | Constructeur |
| --- | --- |
| Menu de 4 entrées écrites en dur | `ListView(children: ...)` |
| Écran de réglages, 10 lignes fixes | `ListView(children: ...)` |
| Liste d'ennemis venant d'une `List<Ennemi>` | `ListView.builder` |
| Résultats d'une recherche | `ListView.builder` |
| Réponse d'une API (chapitre 53) | `ListView.builder` |
| Fil d'actualité infini | `ListView.builder` sans `itemCount` |
| Liste avec un trait entre chaque ligne | `ListView.separated` |

**Le doute doit toujours pencher vers `.builder`.** Il n'y a aucun inconvénient à l'utiliser sur une liste de trois éléments.

---

## 48.7.2 — Ce que `.builder` ne fait PAS

Une confusion fréquente : `.builder` ne construit pas les widgets à la demande **si vous lui donnez déjà des widgets construits**.

```dart
// MAUVAIS : la liste de widgets est construite AVANT le builder
final List<Widget> lignes = inventaire.map((String o) => Text(o)).toList();

ListView.builder(
  itemCount: lignes.length,
  itemBuilder: (BuildContext context, int index) => lignes[index],
)
```

Ici, `lignes` a déjà fabriqué les 40 000 `Text`. Le `.builder` n'apporte rien : le travail est déjà fait.

```dart
// BON : le widget est fabriqué à l'intérieur du builder
ListView.builder(
  itemCount: inventaire.length,
  itemBuilder: (BuildContext context, int index) => Text(inventaire[index]),
)
```

> **Règle.** Le `itemBuilder` doit partir de la **donnée**, pas d'un widget déjà construit.

---

## 48.8 — `ListView.separated`

Quand chaque ligne doit être séparée de la suivante par un trait, un espace ou n'importe quel widget, on utilise `ListView.separated`.

Sa signature ajoute un paramètre à celle de `.builder` :

```dart
ListView.separated({
  Key? key,
  required Widget? Function(BuildContext context, int index) itemBuilder,
  required Widget Function(BuildContext context, int index) separatorBuilder,
  required int itemCount,
  // ... mêmes autres paramètres
})
```

Deux différences avec `.builder` :

1. `separatorBuilder` est **obligatoire** ;
2. `itemCount` est **obligatoire** (il n'est plus `int?` mais `int`). Une liste séparée est forcément finie.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSorts());
}

class ApplicationSorts extends StatelessWidget {
  const ApplicationSorts({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranSorts(),
    );
  }
}

class EcranSorts extends StatelessWidget {
  const EcranSorts({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> sorts = <String>[
      'Boule de feu',
      'Bouclier de glace',
      'Éclair',
      'Soin mineur',
      'Invisibilité',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Grimoire')),
      body: ListView.separated(
        itemCount: sorts.length,
        separatorBuilder: (BuildContext context, int index) {
          return const Divider(height: 1);
        },
        itemBuilder: (BuildContext context, int index) {
          return Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),
            child: Text(sorts[index]),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Grimoire                 │
├──────────────────────────┤
│  Boule de feu            │
│ ──────────────────────── │
│  Bouclier de glace       │
│ ──────────────────────── │
│  Éclair                  │
│ ──────────────────────── │
│  Soin mineur             │
│ ──────────────────────── │
│  Invisibilité            │
└──────────────────────────┘
```

---

## 48.8.1 — Le compte exact des séparateurs

Pour `n` éléments, il y a `n - 1` séparateurs. Flutter s'en occupe : vous donnez `itemCount = n` et il appelle `separatorBuilder` `n - 1` fois.

```text
itemCount = 5

itemBuilder(0)       ─┐
separatorBuilder(0)   │
itemBuilder(1)        │
separatorBuilder(1)   │  5 éléments
itemBuilder(2)        │  4 séparateurs
separatorBuilder(2)   │
itemBuilder(3)        │  index de séparateur : 0 à 3
separatorBuilder(3)   │
itemBuilder(4)       ─┘
```

Il n'y a donc **jamais** de séparateur après le dernier élément, ni avant le premier. C'est exactement ce qu'on veut.

---

## 48.8.2 — Un séparateur qui varie selon l'index

`separatorBuilder` reçoit l'index de l'élément **au-dessus** du séparateur. On peut donc en tirer parti.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationClassement());
}

class ApplicationClassement extends StatelessWidget {
  const ApplicationClassement({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranClassement(),
    );
  }
}

class EcranClassement extends StatelessWidget {
  const EcranClassement({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> joueurs = <String>[
      'Alex',
      'Sophie',
      'Samir',
      'Maria',
      'Léo',
      'Nina',
      'Tom',
      'Inès',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Classement')),
      body: ListView.separated(
        itemCount: joueurs.length,
        separatorBuilder: (BuildContext context, int index) {
          // Un trait épais après le podium (index 2), fin ailleurs.
          if (index == 2) {
            return const Divider(thickness: 3, color: Colors.orange);
          }
          return const Divider(height: 1);
        },
        itemBuilder: (BuildContext context, int index) {
          return Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
            child: Text('${index + 1}. ${joueurs[index]}'),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Classement               │
├──────────────────────────┤
│  1. Alex                 │
│ ──────────────────────── │
│  2. Sophie               │
│ ──────────────────────── │
│  3. Samir                │
│ ════════════════════════ │  <- trait orange épais
│  4. Maria                │
│ ──────────────────────── │
│  5. Léo                  │
│  ...                     │
└──────────────────────────┘
```

---

## 48.8.3 — `Divider` et son piège de hauteur

`Divider` n'est pas un simple trait : c'est une boîte qui contient un trait.

```dart
Divider({
  double? height,      // hauteur TOTALE de la boîte (16 par défaut)
  double? thickness,   // épaisseur du trait lui-même (1 par défaut)
  double? indent,      // retrait à gauche
  double? endIndent,   // retrait à droite
  Color? color,
})
```

```text
Divider()                          Divider(height: 1)
┌──────────────────────┐           ┌──────────────────────┐
│                      │  8 px     │──────────────────────│  1 px au total
│──────────────────────│  1 px     └──────────────────────┘
│                      │  7 px
└──────────────────────┘
     total : 16 px
```

Si vos lignes semblent trop espacées avec `ListView.separated`, la cause est presque toujours la hauteur par défaut de `Divider`. Utilisez `Divider(height: 1)`.

---

## 48.9 — `ListTile`

Écrire soi-même la mise en page de chaque ligne devient vite fastidieux. Material fournit un widget prêt à l'emploi : `ListTile`.

`ListTile` est une ligne standardisée, avec des marges, des tailles de police et des couleurs conformes aux règles Material 3.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInventaire());
}

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> inventaire = <String>[
      'Épée courte',
      'Bouclier de bois',
      'Potion de soin',
      'Arc long',
      'Torche',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: ListView.builder(
        itemCount: inventaire.length,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            title: Text(inventaire[index]),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────┐
│ Inventaire               │
├──────────────────────────┤
│  Épée courte             │
│  Bouclier de bois        │
│  Potion de soin          │
│  Arc long                │
│  Torche                  │
└──────────────────────────┘

Chaque ligne fait 56 pixels de haut, avec 16 pixels de
marge à gauche et à droite : la norme Material.
```

---

## 48.9.1 — Où peut-on utiliser un `ListTile` ?

Contrairement à ce que son nom suggère, `ListTile` n'est pas réservé aux listes. C'est un widget de mise en page comme un autre. On le trouve :

- dans une `ListView` (le cas le plus courant) ;
- dans une `Column` ;
- dans un `Drawer` (chapitre 50) ;
- dans une `Card` (section 48.11) ;
- dans une boîte de dialogue.

Une seule contrainte : `ListTile` a besoin d'une **largeur bornée**. Dans une `Row`, il faut donc l'envelopper dans `Expanded` ou `SizedBox(width: ...)`.

---

## 48.10 — `leading`, `title`, `subtitle`, `trailing`, `onTap`

Voici l'anatomie complète d'un `ListTile`.

```text
┌──────────────────────────────────────────────────────┐
│  ┌────┐                                      ┌────┐  │
│  │lead│   title                              │trai│  │
│  │ing │   subtitle                           │ling│  │
│  └────┘                                      └────┘  │
└──────────────────────────────────────────────────────┘
   ▲         ▲                                   ▲
   │         │                                   │
 icône,    texte principal                   icône, texte,
 avatar,   + texte secondaire                interrupteur,
 image                                       flèche
```

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `leading` | `Widget?` | à gauche : icône, `CircleAvatar`, image |
| `title` | `Widget?` | la ligne principale |
| `subtitle` | `Widget?` | la ligne secondaire, plus petite et grisée |
| `trailing` | `Widget?` | à droite : flèche, `Switch`, prix, bouton |
| `onTap` | `VoidCallback?` | appelé au clic ; ajoute l'effet d'encre |
| `onLongPress` | `VoidCallback?` | appelé sur appui long |
| `isThreeLine` | `bool` | autorise deux lignes de `subtitle` |
| `dense` | `bool` | version compacte |
| `selected` | `bool` | colore le texte et l'icône en couleur primaire |
| `tileColor` | `Color?` | fond de la tuile |
| `contentPadding` | `EdgeInsetsGeometry?` | marges internes |
| `shape` | `ShapeBorder?` | forme (coins arrondis, bordure) |

Exemple complet utilisant les cinq zones :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationBoutique());
}

class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranBoutique(),
    );
  }
}

class EcranBoutique extends StatelessWidget {
  const EcranBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Boutique')),
      body: ListView(
        children: <Widget>[
          ListTile(
            leading: const Icon(Icons.shield, color: Colors.blueGrey),
            title: const Text('Bouclier de fer'),
            subtitle: const Text('Défense +12'),
            trailing: const Text('150 or'),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Bouclier de fer sélectionné')),
              );
            },
          ),
          ListTile(
            leading: const Icon(Icons.local_drink, color: Colors.red),
            title: const Text('Potion de soin'),
            subtitle: const Text('Rend 50 points de vie'),
            trailing: const Text('25 or'),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Potion de soin sélectionnée')),
              );
            },
          ),
          ListTile(
            leading: const Icon(Icons.bolt, color: Colors.amber),
            title: const Text('Parchemin d\'éclair'),
            subtitle: const Text('Dégâts 80, zone'),
            trailing: const Text('320 or'),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Parchemin sélectionné')),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│ Boutique                             │
├──────────────────────────────────────┤
│ [S]  Bouclier de fer         150 or  │
│      Défense +12                     │
│                                      │
│ [P]  Potion de soin           25 or  │
│      Rend 50 points de vie           │
│                                      │
│ [E]  Parchemin d'éclair      320 or  │
│      Dégâts 80, zone                 │
└──────────────────────────────────────┘

Au clic sur une ligne, une bannière apparaît en bas :
┌──────────────────────────────────────┐
│ Bouclier de fer sélectionné          │
└──────────────────────────────────────┘
```

---

## 48.10.1 — `onTap` : l'effet d'encre est offert

Sans `onTap`, un `ListTile` est un simple bloc de texte. Avec `onTap`, il devient cliquable **et** il affiche l'onde de propagation (l'effet « ripple ») au toucher. Vous n'avez rien à faire de plus : pas de `GestureDetector`, pas de `InkWell`.

```dart
ListTile(
  title: const Text('Options'),
  onTap: () { /* rien ici : la ligne reste cliquable */ },
)
```

> **Attention.** Si `onTap` vaut `null` **et** `onLongPress` vaut `null`, la tuile est considérée comme non interactive. Ce n'est pas un bug.

---

## 48.10.2 — `trailing` interactif : attention au conflit de clic

On met souvent un `Switch` ou une `Checkbox` dans `trailing`. Le widget du `trailing` capte alors le clic à sa place, mais le reste de la ligne déclenche toujours `onTap`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationOptions());
}

class ApplicationOptions extends StatelessWidget {
  const ApplicationOptions({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranOptions(),
    );
  }
}

class EcranOptions extends StatefulWidget {
  const EcranOptions({super.key});

  @override
  State<EcranOptions> createState() => _EcranOptionsState();
}

class _EcranOptionsState extends State<EcranOptions> {
  bool _sonActive = true;
  bool _vibrationActive = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Options')),
      body: ListView(
        children: <Widget>[
          SwitchListTile(
            secondary: const Icon(Icons.volume_up),
            title: const Text('Sons du jeu'),
            subtitle: const Text('Musique et bruitages'),
            value: _sonActive,
            onChanged: (bool valeur) {
              setState(() => _sonActive = valeur);
            },
          ),
          SwitchListTile(
            secondary: const Icon(Icons.vibration),
            title: const Text('Vibrations'),
            value: _vibrationActive,
            onChanged: (bool valeur) {
              setState(() => _vibrationActive = valeur);
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│ Options                              │
├──────────────────────────────────────┤
│ [s]  Sons du jeu             ( ●══)  │
│      Musique et bruitages            │
│                                      │
│ [≈]  Vibrations              (══○ )  │
└──────────────────────────────────────┘

Le clic n'importe où sur la ligne bascule l'interrupteur.
```

> **Astuce.** `SwitchListTile` (et ses cousins `CheckboxListTile`, `RadioListTile`) rendent **toute la ligne** cliquable. C'est préférable à un `ListTile` avec un `Switch` dans `trailing`, où seule la petite zone de l'interrupteur réagit. Notez que le paramètre s'appelle `secondary` et non `leading` sur ces trois widgets.

---

## 48.11 — `Card`

Une `Card` est une surface surélevée, avec des coins arrondis et une ombre. Elle sert à regrouper visuellement des informations qui vont ensemble.

Paramètres utiles :

```dart
Card({
  Color? color,
  double? elevation,          // hauteur de l'ombre
  ShapeBorder? shape,         // coins, bordure
  EdgeInsetsGeometry? margin, // marge EXTÉRIEURE (4 par défaut)
  Clip? clipBehavior,         // rogner l'enfant à la forme de la carte
  Widget? child,
})
```

Combinée à un `ListTile`, elle donne le rendu le plus courant des applications mobiles modernes :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationQuetes());
}

class ApplicationQuetes extends StatelessWidget {
  const ApplicationQuetes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranQuetes(),
    );
  }
}

class EcranQuetes extends StatelessWidget {
  const EcranQuetes({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> quetes = <String>[
      'Retrouver l\'épée du roi',
      'Libérer le village de Thorn',
      'Vaincre le dragon des cimes',
      'Livrer le message au mage',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Journal de quêtes')),
      body: ListView.builder(
        padding: const EdgeInsets.all(12),
        itemCount: quetes.length,
        itemBuilder: (BuildContext context, int index) {
          return Card(
            elevation: 2,
            margin: const EdgeInsets.symmetric(vertical: 6),
            child: ListTile(
              leading: CircleAvatar(child: Text('${index + 1}')),
              title: Text(quetes[index]),
              subtitle: Text('Récompense : ${(index + 1) * 100} or'),
              trailing: const Icon(Icons.chevron_right),
              onTap: () {},
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│ Journal de quêtes                      │
├────────────────────────────────────────┤
│  ╭────────────────────────────────────╮│
│  │ (1) Retrouver l'épée du roi     >  ││
│  │     Récompense : 100 or            ││
│  ╰────────────────────────────────────╯│
│  ╭────────────────────────────────────╮│
│  │ (2) Libérer le village de Thorn >  ││
│  │     Récompense : 200 or            ││
│  ╰────────────────────────────────────╯│
│  ╭────────────────────────────────────╮│
│  │ (3) Vaincre le dragon des cimes >  ││
│  │     Récompense : 300 or            ││
│  ╰────────────────────────────────────╯│
└────────────────────────────────────────┘
```

---

## 48.11.1 — L'ordre `Card` puis `ListTile`, jamais l'inverse

```dart
// BON
Card(
  child: ListTile(title: Text('Quête')),
)

// MAUVAIS : un ListTile ne prend pas de Card comme contenu
ListTile(
  title: Card(child: Text('Quête')),
)
```

La `Card` est le **contenant**. Le `ListTile` est le **contenu**.

---

## 48.11.2 — Coins arrondis et rognage

Par défaut, la `Card` ne rogne pas son enfant : un `Container` coloré à l'intérieur dépassera des coins arrondis.

```dart
Card(
  clipBehavior: Clip.antiAlias,   // <- indispensable pour rogner
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(16),
  ),
  child: Column(
    children: <Widget>[
      Container(height: 80, color: Colors.indigo),  // suit les coins
      const ListTile(title: Text('Donjon des glaces')),
    ],
  ),
)
```

```text
Sans clipBehavior              Avec Clip.antiAlias

┌────────────────┐             ╭────────────────╮
│████████████████│  <- carré   │████████████████│  <- arrondi
│████████████████│             │████████████████│
╰────────────────╯             ╰────────────────╯
 Donjon des glaces              Donjon des glaces
```

---

## 48.12 — Construire une tuile personnalisée

`ListTile` couvre 80 % des besoins. Pour les 20 % restants, on écrit son propre widget.

La règle est celle du chapitre 44 : **une tuile est un widget comme un autre**. On en fait une classe `StatelessWidget` séparée, jamais une méthode privée qui retourne un widget.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationEnnemis());
}

class ApplicationEnnemis extends StatelessWidget {
  const ApplicationEnnemis({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranEnnemis(),
    );
  }
}

/// Une tuile personnalisée : nom, niveau, et barre de vie.
class TuileEnnemi extends StatelessWidget {
  const TuileEnnemi({
    super.key,
    required this.nom,
    required this.niveau,
    required this.vie,
    required this.vieMax,
  });

  final String nom;
  final int niveau;
  final int vie;
  final int vieMax;

  @override
  Widget build(BuildContext context) {
    final double ratio = vie / vieMax;

    return Container(
      margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surfaceContainerHighest,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: <Widget>[
          CircleAvatar(
            backgroundColor: Colors.red.shade100,
            child: Text('$niveau'),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text(
                  nom,
                  style: const TextStyle(
                    fontSize: 16,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 6),
                ClipRRect(
                  borderRadius: BorderRadius.circular(4),
                  child: LinearProgressIndicator(
                    value: ratio,
                    minHeight: 8,
                    backgroundColor: Colors.grey.shade300,
                    color: ratio > 0.5
                        ? Colors.green
                        : (ratio > 0.2 ? Colors.orange : Colors.red),
                  ),
                ),
                const SizedBox(height: 4),
                Text('$vie / $vieMax PV',
                    style: const TextStyle(fontSize: 12)),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class EcranEnnemis extends StatelessWidget {
  const EcranEnnemis({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Ennemis en vue')),
      body: ListView(
        padding: const EdgeInsets.symmetric(vertical: 8),
        children: const <Widget>[
          TuileEnnemi(nom: 'Gobelin', niveau: 2, vie: 28, vieMax: 30),
          TuileEnnemi(nom: 'Orc', niveau: 5, vie: 40, vieMax: 90),
          TuileEnnemi(nom: 'Squelette', niveau: 3, vie: 8, vieMax: 45),
          TuileEnnemi(nom: 'Dragon', niveau: 20, vie: 480, vieMax: 500),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│ Ennemis en vue                           │
├──────────────────────────────────────────┤
│ ╭──────────────────────────────────────╮ │
│ │ (2)  Gobelin                         │ │
│ │      ████████████████████░  28/30 PV │ │  vert
│ ╰──────────────────────────────────────╯ │
│ ╭──────────────────────────────────────╮ │
│ │ (5)  Orc                             │ │
│ │      █████████░░░░░░░░░░░  40/90 PV  │ │  orange
│ ╰──────────────────────────────────────╯ │
│ ╭──────────────────────────────────────╮ │
│ │ (3)  Squelette                       │ │
│ │      ███░░░░░░░░░░░░░░░░░   8/45 PV  │ │  rouge
│ ╰──────────────────────────────────────╯ │
│ ╭──────────────────────────────────────╮ │
│ │(20)  Dragon                          │ │
│ │      ███████████████████░ 480/500 PV │ │  vert
│ ╰──────────────────────────────────────╯ │
└──────────────────────────────────────────┘
```

---

## 48.12.1 — Pourquoi une classe et pas une fonction ?

On voit souvent :

```dart
// À ÉVITER
Widget _construireTuile(String nom, int niveau) {
  return Row(children: <Widget>[ /* ... */ ]);
}
```

Cela fonctionne, mais :

| Classe `StatelessWidget` | Fonction qui retourne un widget |
| --- | --- |
| peut être `const` | ne peut jamais être `const` |
| a sa propre place dans l'arbre | fusionne dans le widget parent |
| Flutter peut la reconstruire seule | force la reconstruction du parent entier |
| apparaît dans le Widget Inspector | invisible dans l'inspecteur |
| réutilisable dans un autre fichier | souvent liée à son écran |

Sur une liste de 500 lignes, la différence est mesurable. **Faites-en une classe.**

---

## 48.12.2 — Le `Expanded` obligatoire dans une `Row`

Dans l'exemple ci-dessus, la `Column` du texte est enveloppée dans `Expanded`. Sans lui :

```text
RenderFlex children have non-zero flex but incoming width constraints are unbounded.
```

ou, plus fréquemment :

```text
A RenderFlex overflowed by 87 pixels on the right.
```

Rappel du chapitre 46 : dans une `Row`, un enfant qui doit « prendre la place restante » se met dans `Expanded`. C'est particulièrement vrai dans une tuile de liste, où la largeur du texte varie d'une ligne à l'autre.

---

## 48.13 — Une liste d'objets Dart (rappel chapitres 08-09)

Jusqu'ici nos listes contenaient des `String`. Dans une vraie application, elles contiennent des **objets**.

Reprenons la classe du chapitre 08, écrite avec le constructeur du chapitre 09 :

```dart
class Joueur {
  Joueur({
    required this.nom,
    required this.niveau,
    required this.score,
    required this.classe,
  });

  final String nom;
  final int niveau;
  final int score;
  final String classe;
}
```

Quatre propriétés `final`, un constructeur à paramètres nommés `required`. C'est le modèle standard d'une classe de données en Flutter.

La liste devient :

```dart
final List<Joueur> joueurs = <Joueur>[
  Joueur(nom: 'Alex', niveau: 12, score: 4820, classe: 'Guerrier'),
  Joueur(nom: 'Sophie', niveau: 15, score: 6310, classe: 'Mage'),
  Joueur(nom: 'Samir', niveau: 9, score: 2740, classe: 'Voleur'),
];
```

Et l'accès à un élément se fait comme au chapitre 08 :

```text
joueurs[1]              -> l'objet Joueur « Sophie »
joueurs[1].nom          -> 'Sophie'
joueurs[1].niveau       -> 15
joueurs.length          -> 3
```

---

## 48.13.1 — Où déclarer la liste ?

C'est une question que tous les débutants se posent. La réponse dépend du chapitre 45.

```text
La liste ne change JAMAIS
  -> StatelessWidget, liste déclarée en const ou en champ final

La liste change (ajout, suppression, tri, filtre)
  -> StatefulWidget, liste déclarée dans le State,
     modifiée uniquement à l'intérieur d'un setState
```

**Erreur classique** : déclarer la liste **dans** la méthode `build`.

```dart
@override
Widget build(BuildContext context) {
  final List<Joueur> joueurs = <Joueur>[ /* ... */ ];  // recréée à chaque build !
  return ListView.builder( /* ... */ );
}
```

Sur un `StatelessWidget` de démonstration, c'est sans conséquence. Sur un `StatefulWidget` qui se reconstruit souvent, la liste est reconstruite à chaque frame et toute modification est perdue. Placez-la dans le `State` :

```dart
class _EcranJoueursState extends State<EcranJoueurs> {
  final List<Joueur> _joueurs = <Joueur>[ /* ... */ ];   // créée UNE fois
  // ...
}
```

---

## 48.14 — Afficher une liste de `Joueur`

Assemblons tout : la classe, la liste, la tuile personnalisée et la `ListView.builder`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationClassement());
}

// ---------------------------------------------------------------------------
// MODÈLE (chapitres 08 et 09)
// ---------------------------------------------------------------------------

class Joueur {
  const Joueur({
    required this.nom,
    required this.niveau,
    required this.score,
    required this.classe,
  });

  final String nom;
  final int niveau;
  final int score;
  final String classe;

  /// Les initiales, pour l'avatar. Aucun fichier image n'est nécessaire.
  String get initiales => nom.isEmpty ? '?' : nom[0].toUpperCase();
}

// ---------------------------------------------------------------------------
// DONNÉES
// ---------------------------------------------------------------------------

const List<Joueur> donneesJoueurs = <Joueur>[
  Joueur(nom: 'Alex', niveau: 12, score: 4820, classe: 'Guerrier'),
  Joueur(nom: 'Sophie', niveau: 15, score: 6310, classe: 'Mage'),
  Joueur(nom: 'Samir', niveau: 9, score: 2740, classe: 'Voleur'),
  Joueur(nom: 'Maria', niveau: 18, score: 9150, classe: 'Paladin'),
  Joueur(nom: 'Léo', niveau: 7, score: 1980, classe: 'Archer'),
  Joueur(nom: 'Nina', niveau: 14, score: 5600, classe: 'Druide'),
  Joueur(nom: 'Tom', niveau: 11, score: 3410, classe: 'Guerrier'),
  Joueur(nom: 'Inès', niveau: 16, score: 7020, classe: 'Mage'),
];

// ---------------------------------------------------------------------------
// APPLICATION
// ---------------------------------------------------------------------------

class ApplicationClassement extends StatelessWidget {
  const ApplicationClassement({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Classement',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.indigo,
      ),
      home: const EcranClassement(),
    );
  }
}

class TuileJoueur extends StatelessWidget {
  const TuileJoueur({super.key, required this.joueur, required this.rang});

  final Joueur joueur;
  final int rang;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 5),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Theme.of(context).colorScheme.primaryContainer,
          child: Text(joueur.initiales),
        ),
        title: Text(joueur.nom),
        subtitle: Text('${joueur.classe} — niveau ${joueur.niveau}'),
        trailing: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.end,
          children: <Widget>[
            Text(
              '${joueur.score}',
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            Text('rang $rang', style: const TextStyle(fontSize: 12)),
          ],
        ),
        onTap: () {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('${joueur.nom} : ${joueur.score} points'),
              duration: const Duration(seconds: 1),
            ),
          );
        },
      ),
    );
  }
}

class EcranClassement extends StatelessWidget {
  const EcranClassement({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Classement')),
      body: ListView.builder(
        padding: const EdgeInsets.symmetric(vertical: 8),
        itemCount: donneesJoueurs.length,
        itemBuilder: (BuildContext context, int index) {
          return TuileJoueur(
            joueur: donneesJoueurs[index],
            rang: index + 1,
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────────┐
│ Classement                                 │
├────────────────────────────────────────────┤
│ ╭────────────────────────────────────────╮ │
│ │ (A) Alex                        4820   │ │
│ │     Guerrier — niveau 12       rang 1  │ │
│ ╰────────────────────────────────────────╯ │
│ ╭────────────────────────────────────────╮ │
│ │ (S) Sophie                      6310   │ │
│ │     Mage — niveau 15           rang 2  │ │
│ ╰────────────────────────────────────────╯ │
│ ╭────────────────────────────────────────╮ │
│ │ (S) Samir                       2740   │ │
│ │     Voleur — niveau 9          rang 3  │ │
│ ╰────────────────────────────────────────╯ │
│ ...                                        │
└────────────────────────────────────────────┘
```

---

## 48.14.1 — L'enchaînement complet, en trois couches

Retenez cette structure : elle est valable pour toute application Flutter affichant des données.

```text
┌─────────────────────────────────────────────────────────┐
│  COUCHE MODÈLE                                          │
│  class Joueur { nom, niveau, score, classe }            │
│  Aucun import de Flutter. Testable seul. Chapitre 08.   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  COUCHE DONNÉES                                         │
│  List<Joueur> joueurs = [...]                           │
│  En dur ici ; viendra d'une API au chapitre 53          │
│  ou d'une base locale au chapitre 54.                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  COUCHE INTERFACE                                       │
│  ListView.builder(                                      │
│    itemCount: joueurs.length,                           │
│    itemBuilder: (c, i) => TuileJoueur(joueur: joueurs[i]│
│  )                                                      │
└─────────────────────────────────────────────────────────┘
```

Changer la source des données (API, base, fichier) ne touche **que la couche du milieu**. C'est tout l'intérêt de cette séparation.

---

## 48.15 — `shrinkWrap` et `physics` : quand et pourquoi

Ces deux paramètres sont les plus mal utilisés de tout Flutter. Voici la règle avant l'explication.

> `shrinkWrap: true` et `physics: NeverScrollableScrollPhysics()` ne servent **que** dans un cas : une liste **courte** placée à l'intérieur d'un **autre** élément défilant.
> Dans tous les autres cas, ne les écrivez pas.

### `shrinkWrap`

Par défaut, `shrinkWrap` vaut `false` : la `ListView` prend **toute** la hauteur disponible.

Avec `shrinkWrap: true`, elle mesure ses enfants et prend **exactement** la hauteur nécessaire.

```text
shrinkWrap: false (défaut)          shrinkWrap: true
┌──────────────────────┐            ┌──────────────────────┐
│ Ligne 1              │            │ Ligne 1              │
│ Ligne 2              │            │ Ligne 2              │
│ Ligne 3              │            │ Ligne 3              │
│                      │            └──────────────────────┘
│   (espace vide,      │              La liste s'arrête ici.
│    la liste occupe   │              Le reste de l'écran est
│    tout l'écran)     │              libre pour d'autres widgets.
└──────────────────────┘
```

Le coût : pour connaître sa hauteur, la `ListView` doit **construire tous ses enfants**. Elle perd donc l'avantage principal de `.builder`.

| | `shrinkWrap: false` | `shrinkWrap: true` |
| --- | --- | --- |
| Hauteur | tout l'espace disponible | juste le contenu |
| Enfants construits | seulement les visibles | **tous** |
| Convient à | une liste de 1 à 1 000 000 | une liste de 1 à 30 |

### `physics`

`physics` décide de la manière dont la liste réagit au geste de l'utilisateur.

| Valeur | Comportement |
| --- | --- |
| `null` (défaut) | la plateforme décide (rebond sur iOS, lueur sur Android) |
| `NeverScrollableScrollPhysics()` | la liste **ne défile pas** ; le geste passe au parent |
| `AlwaysScrollableScrollPhysics()` | la liste défile toujours, même si le contenu tient à l'écran |
| `BouncingScrollPhysics()` | rebond façon iOS, sur toutes les plateformes |
| `ClampingScrollPhysics()` | arrêt net façon Android, sur toutes les plateformes |

`AlwaysScrollableScrollPhysics` a un usage précis : rendre un `RefreshIndicator` (section 48.27) utilisable même quand la liste est vide ou très courte.

### Les deux ensemble

```dart
ListView.builder(
  shrinkWrap: true,
  physics: const NeverScrollableScrollPhysics(),
  itemCount: 5,
  itemBuilder: (BuildContext context, int index) => ListTile(
    title: Text('Objet $index'),
  ),
)
```

Traduction en français : « Prends la hauteur de ton contenu, et ne capte aucun geste : c'est mon parent qui défile. »

C'est la combinaison à utiliser pour une liste imbriquée. **Jamais l'une sans l'autre** : `shrinkWrap` seul donne une liste qui défile à l'intérieur d'une autre liste, ce qui est très désagréable à l'usage.

---

## 48.16 — Le piège de la liste dans une liste

Voici l'erreur la plus fréquente du chapitre. Vous la rencontrerez, garanti.

```dart
Scaffold(
  body: Column(
    children: <Widget>[
      const Text('Mon équipe'),
      ListView.builder(                 // <- ERREUR
        itemCount: 5,
        itemBuilder: (BuildContext c, int i) => Text('Membre $i'),
      ),
    ],
  ),
)
```

**Résultat :**

```text
════════ Exception caught by rendering library ════════
The following assertion was thrown during performLayout():
Vertical viewport was given unbounded height.

Viewports expand in the scrolling direction to fill their container.
In this case, a vertical viewport was given an unlimited amount of
vertical space in which to expand. This situation typically happens
when a scrollable widget is nested inside another scrollable widget.
```

et souvent, juste après :

```text
RenderBox was not laid out: RenderViewport#3f1e4 NEEDS-LAYOUT NEEDS-PAINT
'package:flutter/src/rendering/box.dart':
Failed assertion: line 1965 pos 12: 'hasSize'
```

### Pourquoi ?

Une `Column` donne à ses enfants une hauteur **infinie** : « prends la place qu'il te faut, je m'arrangerai ». Une `ListView` répond : « je prends tout l'espace disponible ». Tout de l'infini vaut l'infini. Flutter ne sait pas dessiner un widget de hauteur infinie, il s'arrête.

```text
  Column dit :  "hauteur maximale = infini"
                          │
                          ▼
  ListView répond : "alors je fais infini de haut"
                          │
                          ▼
              Vertical viewport was given unbounded height.
```

### Les quatre corrections possibles

**1. Supprimer la `Column` (la meilleure quand c'est possible)**

Si la `Column` ne servait qu'à mettre un titre au-dessus, mettez le titre dans l'`AppBar` ou en premier élément de la liste.

**2. `Expanded` — la solution par défaut**

```dart
Column(
  children: <Widget>[
    const Text('Mon équipe'),
    Expanded(
      child: ListView.builder(
        itemCount: 5,
        itemBuilder: (BuildContext c, int i) => Text('Membre $i'),
      ),
    ),
  ],
)
```

`Expanded` dit : « la hauteur restante, exactement ». La `ListView` reçoit une contrainte finie et fonctionne, en gardant la construction à la demande.

**3. `SizedBox` — quand la hauteur est connue**

```dart
SizedBox(
  height: 240,
  child: ListView.builder(/* ... */),
)
```

Indispensable pour une liste **horizontale** dans une `Column` (voir 48.4.2).

**4. `shrinkWrap: true` — le dernier recours**

```dart
ListView.builder(
  shrinkWrap: true,
  physics: const NeverScrollableScrollPhysics(),
  itemCount: 5,
  itemBuilder: (BuildContext c, int i) => Text('Membre $i'),
)
```

Attention : si la `Column` elle-même n'est pas dans un `SingleChildScrollView`, un contenu trop long provoquera `A RenderFlex overflowed by ... pixels on the bottom.`

### L'arbre de décision

```text
J'ai une ListView dans une Column
              │
              ├─ La ListView doit occuper le reste de l'écran ?
              │       └─> Expanded          (le plus courant)
              │
              ├─ Elle doit faire exactement N pixels ?
              │       └─> SizedBox(height: N)
              │
              └─ Elle est courte ET le parent défile déjà ?
                      └─> shrinkWrap: true
                          + physics: NeverScrollableScrollPhysics()
```

Exemple complet, correct, à conserver comme référence :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationEquipe());
}

class ApplicationEquipe extends StatelessWidget {
  const ApplicationEquipe({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranEquipe(),
    );
  }
}

class EcranEquipe extends StatelessWidget {
  const EcranEquipe({super.key});

  @override
  Widget build(BuildContext context) {
    final List<String> membres = <String>[
      'Alex', 'Sophie', 'Samir', 'Maria', 'Léo', 'Nina', 'Tom',
    ];
    final List<String> favoris = <String>['Épée', 'Potion', 'Arc'];

    return Scaffold(
      appBar: AppBar(title: const Text('Équipe')),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('Objets favoris',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),
          // Liste COURTE imbriquée : shrinkWrap + physics.
          ListView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            itemCount: favoris.length,
            itemBuilder: (BuildContext context, int index) {
              return ListTile(
                dense: true,
                leading: const Icon(Icons.star, size: 20),
                title: Text(favoris[index]),
              );
            },
          ),
          const Divider(),
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('Membres',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),
          // Liste LONGUE : Expanded, pas de shrinkWrap.
          Expanded(
            child: ListView.builder(
              itemCount: membres.length,
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  leading: CircleAvatar(child: Text(membres[index][0])),
                  title: Text(membres[index]),
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

**Résultat :**

```text
┌────────────────────────────────┐
│ Équipe                         │
├────────────────────────────────┤
│ Objets favoris                 │
│  * Épée                        │  <- shrinkWrap : juste 3 lignes
│  * Potion                      │
│  * Arc                         │
│ ────────────────────────────── │
│ Membres                        │
│ (A) Alex                       │  <- Expanded : occupe le reste,
│ (S) Sophie                     │     défile si nécessaire
│ (S) Samir                      │
│ (M) Maria                      │
└────────────────────────────────┘
```

---

## 48.17 — `ScrollController`

Un `ScrollController` est un objet qui donne accès à la position de défilement d'une liste : le **lire** et la **modifier**.

Cycle de vie obligatoire (chapitre 45) :

```text
StatefulWidget requis
  │
  ├─ champ      : final ScrollController _controleur = ScrollController();
  ├─ ListView   : controller: _controleur
  └─ dispose()  : _controleur.dispose();     <- OBLIGATOIRE
```

Oublier `dispose()` provoque une fuite de mémoire, signalée en mode debug par :

```text
A ScrollController was used after being disposed.
```

ou par un avertissement de l'analyseur : `close_sinks` / `Don't forget to dispose`.

### Ce qu'on peut lire

| Expression | Valeur |
| --- | --- |
| `_controleur.offset` | position actuelle en pixels (raccourci de `position.pixels`) |
| `_controleur.position.pixels` | idem |
| `_controleur.position.maxScrollExtent` | position maximale |
| `_controleur.position.minScrollExtent` | position minimale, `0.0` en général |
| `_controleur.position.viewportDimension` | hauteur visible |
| `_controleur.position.extentAfter` | pixels restants sous le viewport |
| `_controleur.hasClients` | `true` si le contrôleur est attaché à une liste |

> **`hasClients` est un garde-fou indispensable.** Tant que la liste n'est pas construite, `offset` et `position` lèvent :
> `'package:flutter/src/widgets/scroll_controller.dart': Failed assertion: line 116 pos 12: '_positions.isNotEmpty': ScrollController not attached to any scroll views.`

### Ce qu'on peut commander

```dart
// Saut instantané
_controleur.jumpTo(0);

// Déplacement animé
_controleur.animateTo(
  0,
  duration: const Duration(milliseconds: 400),
  curve: Curves.easeOut,
);
```

`animateTo` retourne un `Future<void>` qui se termine à la fin de l'animation (chapitre 15).

Exemple : afficher la position en direct.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationPosition());
}

class ApplicationPosition extends StatelessWidget {
  const ApplicationPosition({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranPosition(),
    );
  }
}

class EcranPosition extends StatefulWidget {
  const EcranPosition({super.key});

  @override
  State<EcranPosition> createState() => _EcranPositionState();
}

class _EcranPositionState extends State<EcranPosition> {
  final ScrollController _controleur = ScrollController();
  double _position = 0;
  double _maximum = 0;

  @override
  void initState() {
    super.initState();
    _controleur.addListener(_surDefilement);
  }

  void _surDefilement() {
    setState(() {
      _position = _controleur.offset;
      _maximum = _controleur.position.maxScrollExtent;
    });
  }

  @override
  void dispose() {
    _controleur.removeListener(_surDefilement);
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('${_position.round()} / ${_maximum.round()} px'),
      ),
      body: ListView.builder(
        controller: _controleur,
        itemCount: 100,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(title: Text('Ennemi n°${index + 1}'));
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────┐
│ 0 / 5044 px                  │   au démarrage
├──────────────────────────────┤
│  Ennemi n°1                  │
│  Ennemi n°2                  │
└──────────────────────────────┘

Après avoir fait défiler :

┌──────────────────────────────┐
│ 1832 / 5044 px               │
├──────────────────────────────┤
│  Ennemi n°34                 │
│  Ennemi n°35                 │
└──────────────────────────────┘
```

> **Attention à la performance.** Appeler `setState` à chaque pixel de défilement reconstruit tout l'écran 60 fois par seconde. Ici c'est acceptable car l'écran est simple. En production, préférez un `ValueNotifier` et un `ValueListenableBuilder`, ou n'appelez `setState` que lorsque la valeur affichée change réellement.

---

## 48.18 — Détecter la fin de liste et charger la suite

C'est le mécanisme du « défilement infini » : on charge 20 éléments, et quand l'utilisateur approche du bas, on en charge 20 de plus.

### La condition de déclenchement

```text
  ┌───────────────────┐
  │                   │
  │   viewport        │  <- pixels
  │                   │
  └───────────────────┘
  ░░░░░░░░░░░░░░░░░░░░░  <- extentAfter : ce qui reste dessous

  Déclencher quand :  extentAfter < 300

  ou, formulation équivalente :
     offset >= maxScrollExtent - 300
```

On ne teste **jamais** `offset == maxScrollExtent`. Sur mobile, l'égalité exacte n'est pratiquement jamais atteinte à cause de l'inertie et du rebond.

### Le drapeau anti-doublon

Le listener est appelé des dizaines de fois par seconde. Sans protection, dix chargements partiraient en même temps. Un booléen `_enChargement` règle le problème.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationInfinie());
}

class ApplicationInfinie extends StatelessWidget {
  const ApplicationInfinie({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInfini(),
    );
  }
}

class EcranInfini extends StatefulWidget {
  const EcranInfini({super.key});

  @override
  State<EcranInfini> createState() => _EcranInfiniState();
}

class _EcranInfiniState extends State<EcranInfini> {
  final ScrollController _controleur = ScrollController();
  final List<String> _ennemis = <String>[];

  bool _enChargement = false;
  bool _finAtteinte = false;
  int _page = 0;

  static const int _tailleDePage = 20;
  static const int _nombreTotal = 85;

  @override
  void initState() {
    super.initState();
    _controleur.addListener(_surDefilement);
    _chargerPageSuivante();
  }

  @override
  void dispose() {
    _controleur.removeListener(_surDefilement);
    _controleur.dispose();
    super.dispose();
  }

  void _surDefilement() {
    if (_controleur.position.extentAfter < 300) {
      _chargerPageSuivante();
    }
  }

  /// Simule un appel réseau (chapitre 15 ; le vrai http arrive au chapitre 53).
  Future<void> _chargerPageSuivante() async {
    if (_enChargement || _finAtteinte) {
      return;
    }
    setState(() => _enChargement = true);

    await Future<void>.delayed(const Duration(milliseconds: 800));

    final int debut = _page * _tailleDePage;
    final int fin = (debut + _tailleDePage) > _nombreTotal
        ? _nombreTotal
        : debut + _tailleDePage;

    final List<String> nouveaux = <String>[
      for (int i = debut; i < fin; i++) 'Ennemi n°${i + 1}',
    ];

    if (!mounted) {
      return;
    }
    setState(() {
      _ennemis.addAll(nouveaux);
      _page++;
      _enChargement = false;
      _finAtteinte = fin >= _nombreTotal;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Bestiaire (${_ennemis.length})')),
      body: ListView.builder(
        controller: _controleur,
        // Une ligne de plus pour le pied de liste.
        itemCount: _ennemis.length + 1,
        itemBuilder: (BuildContext context, int index) {
          if (index < _ennemis.length) {
            return ListTile(
              leading: const Icon(Icons.pest_control),
              title: Text(_ennemis[index]),
            );
          }
          // Dernière ligne : indicateur ou message de fin.
          return Padding(
            padding: const EdgeInsets.symmetric(vertical: 24),
            child: Center(
              child: _finAtteinte
                  ? const Text('Tous les ennemis sont chargés.')
                  : const CircularProgressIndicator(),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────┐
│ Bestiaire (20)                 │
├────────────────────────────────┤
│  Ennemi n°18                   │
│  Ennemi n°19                   │
│  Ennemi n°20                   │
│         ( ◌ )                  │  <- chargement de la page 2
└────────────────────────────────┘

Après quelques secondes de défilement :

┌────────────────────────────────┐
│ Bestiaire (85)                 │
├────────────────────────────────┤
│  Ennemi n°84                   │
│  Ennemi n°85                   │
│  Tous les ennemis sont chargés.│
└────────────────────────────────┘
```

### Les trois points à ne pas manquer

1. **`itemCount: _ennemis.length + 1`** — la ligne supplémentaire porte l'indicateur. Le `itemBuilder` doit alors tester `index < _ennemis.length` avant d'accéder au tableau, sinon : `RangeError (index): Index out of range: index should be less than 20: 20`.
2. **`if (!mounted) return;`** — entre le `await` et le `setState`, l'utilisateur a pu quitter l'écran. Sans ce test : `setState() called after dispose()`.
3. **`_enChargement`** — sans lui, dix requêtes partent en parallèle et la liste contient des doublons.

---

## 48.19 — Remonter en haut avec un bouton

Un classique : un bouton flottant apparaît dès que l'on a défilé, et ramène en haut de la liste.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationRetourHaut());
}

class ApplicationRetourHaut extends StatelessWidget {
  const ApplicationRetourHaut({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranRetourHaut(),
    );
  }
}

class EcranRetourHaut extends StatefulWidget {
  const EcranRetourHaut({super.key});

  @override
  State<EcranRetourHaut> createState() => _EcranRetourHautState();
}

class _EcranRetourHautState extends State<EcranRetourHaut> {
  final ScrollController _controleur = ScrollController();
  bool _boutonVisible = false;

  @override
  void initState() {
    super.initState();
    _controleur.addListener(_surDefilement);
  }

  void _surDefilement() {
    final bool doitEtreVisible = _controleur.offset > 400;
    // On n'appelle setState QUE si l'état change réellement.
    if (doitEtreVisible != _boutonVisible) {
      setState(() => _boutonVisible = doitEtreVisible);
    }
  }

  @override
  void dispose() {
    _controleur.removeListener(_surDefilement);
    _controleur.dispose();
    super.dispose();
  }

  Future<void> _remonter() async {
    await _controleur.animateTo(
      0,
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeOutCubic,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bestiaire')),
      body: ListView.builder(
        controller: _controleur,
        itemCount: 120,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            leading: CircleAvatar(child: Text('${index + 1}')),
            title: Text('Ennemi n°${index + 1}'),
          );
        },
      ),
      floatingActionButton: _boutonVisible
          ? FloatingActionButton(
              onPressed: _remonter,
              tooltip: 'Remonter en haut',
              child: const Icon(Icons.arrow_upward),
            )
          : null,
    );
  }
}
```

**Résultat :**

```text
Au démarrage (offset = 0) :        Après avoir défilé (offset > 400) :

┌────────────────────────┐         ┌────────────────────────┐
│ Bestiaire              │         │ Bestiaire              │
├────────────────────────┤         ├────────────────────────┤
│ (1) Ennemi n°1         │         │ (23) Ennemi n°23       │
│ (2) Ennemi n°2         │         │ (24) Ennemi n°24       │
│ (3) Ennemi n°3         │         │ (25) Ennemi n°25       │
│                        │         │                  ╭───╮ │
│                        │         │                  │ ↑ │ │
└────────────────────────┘         └──────────────────╰───╯─┘
```

> **Le test `if (doitEtreVisible != _boutonVisible)` est essentiel.** Sans lui, `setState` serait appelé à chaque pixel de défilement, soit des milliers de reconstructions inutiles.

---

## 48.20 — `GridView.count`

Une grille est une liste à plusieurs colonnes. Tout ce que vous savez sur `ListView` s'y applique : elle défile, elle a un `controller`, un `padding`, un `shrinkWrap`, un `physics`.

`GridView.count` est le constructeur le plus simple : on lui dit combien de colonnes on veut.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSacADos());
}

class ApplicationSacADos extends StatelessWidget {
  const ApplicationSacADos({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranSacADos(),
    );
  }
}

class EcranSacADos extends StatelessWidget {
  const EcranSacADos({super.key});

  @override
  Widget build(BuildContext context) {
    final List<IconData> objets = <IconData>[
      Icons.shield, Icons.local_drink, Icons.bolt, Icons.key,
      Icons.diamond, Icons.book, Icons.lightbulb, Icons.pets,
      Icons.coffee, Icons.star, Icons.security, Icons.whatshot,
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Sac à dos')),
      body: GridView.count(
        crossAxisCount: 3,
        mainAxisSpacing: 12,
        crossAxisSpacing: 12,
        padding: const EdgeInsets.all(12),
        children: objets.map((IconData icone) {
          return Container(
            decoration: BoxDecoration(
              color: Colors.indigo.shade50,
              borderRadius: BorderRadius.circular(12),
              border: Border.all(color: Colors.indigo.shade200),
            ),
            child: Icon(icone, size: 36, color: Colors.indigo),
          );
        }).toList(),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│ Sac à dos                        │
├──────────────────────────────────┤
│ ╭──────╮ ╭──────╮ ╭──────╮       │
│ │  S   │ │  P   │ │  E   │       │
│ ╰──────╯ ╰──────╯ ╰──────╯       │
│ ╭──────╮ ╭──────╮ ╭──────╮       │
│ │  K   │ │  D   │ │  B   │       │
│ ╰──────╯ ╰──────╯ ╰──────╯       │
│ ╭──────╮ ╭──────╮ ╭──────╮       │
│ │  L   │ │  A   │ │  C   │       │
│ ╰──────╯ ╰──────╯ ╰──────╯       │
│  ...                             │
└──────────────────────────────────┘
```

> **Même mise en garde que pour `ListView(children: ...)`.** `GridView.count` construit toutes ses cases immédiatement. Réservez-le aux grilles courtes et fixes.

---

## 48.21 — `GridView.builder` et `SliverGridDelegate`

Pour une grille de taille variable, on utilise `GridView.builder`. Il exige un paramètre supplémentaire : le `gridDelegate`.

Le **delegate** est l'objet qui répond à la question « où et de quelle taille est la case numéro `i` ? ». Flutter en fournit deux :

| Delegate | Question posée | Usage |
| --- | --- | --- |
| `SliverGridDelegateWithFixedCrossAxisCount` | « combien de colonnes ? » | mise en page fixe |
| `SliverGridDelegateWithMaxCrossAxisExtent` | « quelle largeur maximale par case ? » | mise en page responsive |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationCartes());
}

class ApplicationCartes extends StatelessWidget {
  const ApplicationCartes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranCartes(),
    );
  }
}

class EcranCartes extends StatelessWidget {
  const EcranCartes({super.key});

  @override
  Widget build(BuildContext context) {
    const List<String> raretes = <String>[
      'Commune', 'Rare', 'Épique', 'Légendaire',
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Collection de cartes')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        itemCount: 60,
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
          childAspectRatio: 0.72,
        ),
        itemBuilder: (BuildContext context, int index) {
          final String rarete = raretes[index % raretes.length];
          return Card(
            clipBehavior: Clip.antiAlias,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: <Widget>[
                Expanded(
                  child: Container(
                    color: Colors.primaries[index % Colors.primaries.length]
                        .shade200,
                    child: Center(
                      child: Text(
                        '#${index + 1}',
                        style: const TextStyle(
                          fontSize: 28,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.all(8),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: <Widget>[
                      Text('Carte n°${index + 1}',
                          style: const TextStyle(
                              fontWeight: FontWeight.bold)),
                      Text(rarete,
                          style: const TextStyle(fontSize: 12)),
                    ],
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│ Collection de cartes               │
├────────────────────────────────────┤
│ ╭──────────────╮ ╭──────────────╮  │
│ │              │ │              │  │
│ │      #1      │ │      #2      │  │
│ │              │ │              │  │
│ ├──────────────┤ ├──────────────┤  │
│ │ Carte n°1    │ │ Carte n°2    │  │
│ │ Commune      │ │ Rare         │  │
│ ╰──────────────╯ ╰──────────────╯  │
│ ╭──────────────╮ ╭──────────────╮  │
│ │      #3      │ │      #4      │  │
│  ...                               │
└────────────────────────────────────┘
```

Comme pour `ListView.builder`, seules les cases visibles sont construites. `itemCount: 60` ou `itemCount: 60000`, le coût d'affichage est le même.

---

## 48.22 — `crossAxisCount`, `childAspectRatio`, les espacements

Quatre nombres suffisent à décrire une grille. Voici précisément ce que chacun fait.

```text
             crossAxisSpacing
                    │
   ┌────────────────┼────────────────┐
   │  ╭─────────╮   │   ╭─────────╮  │
   │  │         │ ◄─┴─► │         │  │
   │  │  case   │       │  case   │  │  largeur = L
   │  │         │       │         │  │  hauteur = H
   │  ╰─────────╯       ╰─────────╯  │
   │        ▲                        │  childAspectRatio = L / H
   │        │  mainAxisSpacing       │
   │        ▼                        │
   │  ╭─────────╮       ╭─────────╮  │
   │  │  case   │       │  case   │  │
   │  ╰─────────╯       ╰─────────╯  │
   └─────────────────────────────────┘
        crossAxisCount = 2
```

| Paramètre | Type | Défaut | Effet |
| --- | --- | --- | --- |
| `crossAxisCount` | `int` | requis | nombre de colonnes (grille verticale) |
| `mainAxisSpacing` | `double` | `0.0` | espace **vertical** entre deux rangées |
| `crossAxisSpacing` | `double` | `0.0` | espace **horizontal** entre deux colonnes |
| `childAspectRatio` | `double` | `1.0` | largeur divisée par hauteur d'une case |
| `mainAxisExtent` | `double?` | `null` | hauteur fixe d'une case, **prioritaire** sur `childAspectRatio` |

### Le calcul exact de la largeur d'une case

```text
largeurCase = (largeurGrille
               - padding.gauche - padding.droit
               - crossAxisSpacing * (crossAxisCount - 1))
              / crossAxisCount

hauteurCase = largeurCase / childAspectRatio
```

Exemple chiffré, écran de 400 pixels de large, `padding: EdgeInsets.all(12)`, `crossAxisCount: 2`, `crossAxisSpacing: 12` :

```text
largeurCase = (400 - 12 - 12 - 12 * 1) / 2 = 364 / 2 = 182 px

childAspectRatio: 1.0   ->  hauteurCase = 182 px  (carré)
childAspectRatio: 0.72  ->  hauteurCase = 253 px  (portrait, format carte)
childAspectRatio: 1.6   ->  hauteurCase = 114 px  (paysage)
```

### Le sens de `childAspectRatio`, une fois pour toutes

```text
childAspectRatio < 1   ->  case PLUS HAUTE que large   ╭──╮
                                                       │  │
                                                       │  │
                                                       ╰──╯

childAspectRatio = 1   ->  case CARRÉE                 ╭────╮
                                                       │    │
                                                       ╰────╯

childAspectRatio > 1   ->  case PLUS LARGE que haute   ╭────────╮
                                                       ╰────────╯
```

Beaucoup se trompent de sens. Moyen mnémotechnique : **c'est une fraction, largeur sur hauteur**. Une case deux fois plus large que haute vaut `2.0`.

### L'erreur de débordement en grille

Si le contenu de la case est plus haut que la case :

```text
A RenderFlex overflowed by 23 pixels on the bottom.
```

Trois corrections, dans l'ordre de préférence :

1. **diminuer `childAspectRatio`** (cases plus hautes) ;
2. **utiliser `mainAxisExtent`** pour fixer la hauteur en pixels, indépendamment de la largeur ;
3. réduire le contenu de la case (`maxLines`, police plus petite).

---

## 48.23 — Une grille responsive avec `maxCrossAxisExtent`

`crossAxisCount: 2` donne deux colonnes sur un téléphone... et deux colonnes énormes sur une tablette ou dans un navigateur.

`SliverGridDelegateWithMaxCrossAxisExtent` inverse le raisonnement : au lieu de fixer le nombre de colonnes, on fixe la **largeur maximale d'une case**, et Flutter calcule le nombre de colonnes.

```text
maxCrossAxisExtent: 200

Écran 360 px  ->  360 / 200 = 1,8  ->  2 colonnes de 180 px
Écran 800 px  ->  800 / 200 = 4,0  ->  4 colonnes de 200 px
Écran 1400 px -> 1400 / 200 = 7,0  ->  7 colonnes de 200 px
```

La formule appliquée par Flutter est : `nombreDeColonnes = (largeurDisponible / maxCrossAxisExtent).ceil()`, puis la largeur réelle est répartie équitablement. Une case ne dépasse donc **jamais** `maxCrossAxisExtent`, mais elle peut être plus étroite.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationGrilleResponsive());
}

class ApplicationGrilleResponsive extends StatelessWidget {
  const ApplicationGrilleResponsive({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranGrilleResponsive(),
    );
  }
}

class EcranGrilleResponsive extends StatelessWidget {
  const EcranGrilleResponsive({super.key});

  @override
  Widget build(BuildContext context) {
    final double largeur = MediaQuery.sizeOf(context).width;

    return Scaffold(
      appBar: AppBar(title: Text('Grille — écran ${largeur.round()} px')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        itemCount: 40,
        gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
          maxCrossAxisExtent: 200,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
          childAspectRatio: 1,
        ),
        itemBuilder: (BuildContext context, int index) {
          return Container(
            decoration: BoxDecoration(
              color: Colors.teal.shade100,
              borderRadius: BorderRadius.circular(12),
            ),
            alignment: Alignment.center,
            child: Text(
              'Case ${index + 1}',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
Téléphone (360 px)              Tablette (900 px)

┌────────────────────┐          ┌──────────────────────────────────────┐
│ Grille — 360 px    │          │ Grille — 900 px                      │
├────────────────────┤          ├──────────────────────────────────────┤
│ ╭─────╮ ╭─────╮    │          │ ╭─────╮ ╭─────╮ ╭─────╮ ╭─────╮ ╭──╮ │
│ │ 1   │ │ 2   │    │          │ │ 1   │ │ 2   │ │ 3   │ │ 4   │ │5 │ │
│ ╰─────╯ ╰─────╯    │          │ ╰─────╯ ╰─────╯ ╰─────╯ ╰─────╯ ╰──╯ │
│ ╭─────╮ ╭─────╮    │          │ ╭─────╮ ╭─────╮ ╭─────╮ ╭─────╮ ╭──╮ │
│ │ 3   │ │ 4   │    │          │ │ 6   │ │ 7   │ │ 8   │ │ 9   │ │10│ │
│ ╰─────╯ ╰─────╯    │          │ ╰─────╯ ╰─────╯ ╰─────╯ ╰─────╯ ╰──╯ │
└────────────────────┘          └──────────────────────────────────────┘

2 colonnes automatiques         5 colonnes automatiques
```

Aucune ligne de code conditionnel. C'est la façon la plus simple de rendre une grille responsive.

> **Rappel.** `MediaQuery.sizeOf(context)` est la forme moderne, plus performante que `MediaQuery.of(context).size` car elle ne reconstruit le widget que lorsque la **taille** change. Le chapitre 51 revient en détail sur le responsive.

---

## 48.24 — `Dismissible` : glisser pour supprimer

`Dismissible` enveloppe une ligne et la rend « glissable ». Quand l'utilisateur la fait sortir de l'écran, elle disparaît avec une animation et `onDismissed` est appelé.

Paramètres principaux :

```dart
Dismissible({
  required Key key,                       // OBLIGATOIRE (voir 48.25)
  required Widget child,
  Widget? background,                     // fond révélé en glissant vers la droite
  Widget? secondaryBackground,            // fond révélé en glissant vers la gauche
  DismissDirection direction = DismissDirection.horizontal,
  void Function(DismissDirection)? onDismissed,
  Future<bool?> Function(DismissDirection)? confirmDismiss,
  Map<DismissDirection, double> dismissThresholds = const {},
})
```

Valeurs de `DismissDirection` : `horizontal`, `vertical`, `endToStart`, `startToEnd`, `up`, `down`, `none`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSac());
}

class ApplicationSac extends StatelessWidget {
  const ApplicationSac({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranSac(),
    );
  }
}

class EcranSac extends StatefulWidget {
  const EcranSac({super.key});

  @override
  State<EcranSac> createState() => _EcranSacState();
}

class _EcranSacState extends State<EcranSac> {
  final List<String> _objets = <String>[
    'Épée courte',
    'Bouclier de bois',
    'Potion de soin',
    'Arc long',
    'Torche',
    'Clé rouillée',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Sac (${_objets.length})')),
      body: ListView.builder(
        itemCount: _objets.length,
        itemBuilder: (BuildContext context, int index) {
          final String objet = _objets[index];
          return Dismissible(
            key: ValueKey<String>(objet),
            direction: DismissDirection.endToStart,
            background: Container(
              color: Colors.red,
              alignment: Alignment.centerRight,
              padding: const EdgeInsets.only(right: 20),
              child: const Icon(Icons.delete, color: Colors.white),
            ),
            onDismissed: (DismissDirection direction) {
              setState(() => _objets.removeAt(index));
            },
            child: ListTile(
              leading: const Icon(Icons.inventory_2),
              title: Text(objet),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
Pendant le glissement vers la gauche :

┌────────────────────────────────────┐
│ Sac (6)                            │
├────────────────────────────────────┤
│  Épée courte                       │
│      Bouclier de bois     ███ X ██ │  <- fond rouge révélé
│  Potion de soin                    │
└────────────────────────────────────┘

Après le relâchement :

┌────────────────────────────────────┐
│ Sac (5)                            │
├────────────────────────────────────┤
│  Épée courte                       │
│  Potion de soin                    │
└────────────────────────────────────┘
```

### Deux fonds pour deux directions

```dart
Dismissible(
  key: ValueKey<String>(objet),
  background: Container(          // glissement vers la DROITE
    color: Colors.green,
    alignment: Alignment.centerLeft,
    padding: const EdgeInsets.only(left: 20),
    child: const Icon(Icons.check, color: Colors.white),
  ),
  secondaryBackground: Container( // glissement vers la GAUCHE
    color: Colors.red,
    alignment: Alignment.centerRight,
    padding: const EdgeInsets.only(right: 20),
    child: const Icon(Icons.delete, color: Colors.white),
  ),
  onDismissed: (DismissDirection direction) {
    if (direction == DismissDirection.startToEnd) {
      // équipé
    } else {
      // supprimé
    }
  },
  child: ListTile(title: Text(objet)),
)
```

### `confirmDismiss` : demander avant de supprimer

`confirmDismiss` est appelé **avant** l'animation de sortie. S'il retourne `false`, la ligne revient en place.

```dart
confirmDismiss: (DismissDirection direction) async {
  final bool? reponse = await showDialog<bool>(
    context: context,
    builder: (BuildContext context) => AlertDialog(
      title: const Text('Supprimer cet objet ?'),
      actions: <Widget>[
        TextButton(
          onPressed: () => Navigator.of(context).pop(false),
          child: const Text('Annuler'),
        ),
        TextButton(
          onPressed: () => Navigator.of(context).pop(true),
          child: const Text('Supprimer'),
        ),
      ],
    ),
  );
  return reponse ?? false;
},
```

---

## 48.24.1 — L'erreur à connaître par cœur

Si vous oubliez de retirer réellement l'élément de la liste dans `onDismissed` :

```text
════════ Exception caught by widgets library ════════
A dismissed Dismissible widget is still part of the tree.

Make sure to implement the onDismissed handler and to immediately
remove the Dismissible widget from the application once that
handler has fired.
```

La cause est toujours la même : `onDismissed` a été appelé, mais la donnée est encore dans la liste, donc `itemBuilder` reconstruit la ligne supprimée.

```dart
// MAUVAIS : on n'enlève rien
onDismissed: (_) {
  ScaffoldMessenger.of(context).showSnackBar(
    const SnackBar(content: Text('Supprimé')),
  );
},

// BON : on enlève la donnée ET on prévient Flutter
onDismissed: (_) {
  setState(() => _objets.removeAt(index));
},
```

---

## 48.25 — La `Key` d'un `Dismissible` : pourquoi elle est obligatoire

`key` est le seul paramètre `required` de `Dismissible` avec `child`. Ce n'est pas un caprice de l'API.

### Comment Flutter identifie les enfants d'une liste

Sans clé, Flutter apparie les anciens et les nouveaux widgets **par position et par type**.

```text
AVANT suppression        APRÈS suppression de l'index 1

position 0 : 'Épée'      position 0 : 'Épée'
position 1 : 'Bouclier'  position 1 : 'Potion'
position 2 : 'Potion'    position 2 : 'Arc'
position 3 : 'Arc'
```

Flutter voit : « position 0 : ListTile, toujours ListTile — je le garde. Position 1 : ListTile, toujours ListTile — je le garde. » Il ne comprend pas qu'un élément a été **retiré du milieu**. Il croit que les textes ont changé.

Or l'état interne du `Dismissible` (sa position d'animation, son fait d'être « déjà glissé ») reste attaché à la position, pas à la donnée. Résultat : **la mauvaise ligne disparaît**, ou une ligne reste bloquée à moitié glissée.

### Ce que la clé change

```text
AVEC ValueKey<String>

AVANT                          APRÈS
key 'Épée'     ─────────────>  key 'Épée'
key 'Bouclier' ─────  X        (absente)
key 'Potion'   ─────────────>  key 'Potion'
key 'Arc'      ─────────────>  key 'Arc'
```

Flutter apparie **par clé**. Il voit que « Bouclier » a disparu et détruit exactement cet élément-là.

### Quelle clé utiliser ?

| Clé | Quand |
| --- | --- |
| `ValueKey(objet.id)` | idéal : un identifiant unique et stable |
| `ValueKey(objet.nom)` | acceptable si les noms sont uniques |
| `ObjectKey(objet)` | bon si l'objet lui-même est unique en mémoire |
| `UniqueKey()` | **jamais** : elle change à chaque `build`, tout est détruit et reconstruit |
| `ValueKey(index)` | **jamais** : l'index d'un élément change quand on supprime |

Les deux dernières lignes sont les pièges classiques.

```dart
// CATASTROPHIQUE : la clé change à chaque reconstruction
key: UniqueKey(),

// FAUX : après suppression de l'index 1, l'ancien index 2 devient 1
key: ValueKey<int>(index),

// CORRECT
key: ValueKey<String>(objet.id),
```

### Si deux éléments partagent la même clé

```text
Duplicate keys found.
If multiple keyed nodes exist as children of another node,
they must have unique keys.
```

C'est le signe que vous avez utilisé un champ non unique (par exemple le nom, avec deux « Potion » dans la liste). Ajoutez un identifiant à votre modèle.

---

## 48.26 — Annuler une suppression avec un `SnackBar`

Supprimer par glissement est rapide... et donc facile à déclencher par accident. La bonne pratique est de proposer une annulation.

Le principe :

```text
1. Retirer l'élément de la liste, MAIS en garder une copie
   ainsi que son index d'origine.
2. Afficher un SnackBar avec une SnackBarAction « Annuler ».
3. Si l'utilisateur appuie : réinsérer la copie à son index.
4. Sinon, le SnackBar disparaît et la suppression devient définitive.
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationAnnulation());
}

class ApplicationAnnulation extends StatelessWidget {
  const ApplicationAnnulation({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranAnnulation(),
    );
  }
}

class Objet {
  const Objet({required this.id, required this.nom});
  final String id;
  final String nom;
}

class EcranAnnulation extends StatefulWidget {
  const EcranAnnulation({super.key});

  @override
  State<EcranAnnulation> createState() => _EcranAnnulationState();
}

class _EcranAnnulationState extends State<EcranAnnulation> {
  final List<Objet> _objets = <Objet>[
    const Objet(id: 'o1', nom: 'Épée courte'),
    const Objet(id: 'o2', nom: 'Bouclier de bois'),
    const Objet(id: 'o3', nom: 'Potion de soin'),
    const Objet(id: 'o4', nom: 'Arc long'),
    const Objet(id: 'o5', nom: 'Torche'),
  ];

  void _supprimer(int index) {
    final Objet retire = _objets[index];
    final int indexOrigine = index;

    setState(() => _objets.removeAt(index));

    // On mémorise le messenger AVANT tout await éventuel.
    final ScaffoldMessengerState messenger = ScaffoldMessenger.of(context);
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      SnackBar(
        content: Text('${retire.nom} supprimé'),
        duration: const Duration(seconds: 4),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () {
            setState(() => _objets.insert(indexOrigine, retire));
          },
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Sac (${_objets.length})')),
      body: ListView.builder(
        itemCount: _objets.length,
        itemBuilder: (BuildContext context, int index) {
          final Objet objet = _objets[index];
          return Dismissible(
            key: ValueKey<String>(objet.id),
            direction: DismissDirection.endToStart,
            background: Container(
              color: Colors.red,
              alignment: Alignment.centerRight,
              padding: const EdgeInsets.only(right: 20),
              child: const Icon(Icons.delete, color: Colors.white),
            ),
            onDismissed: (DismissDirection direction) => _supprimer(index),
            child: ListTile(
              leading: const Icon(Icons.inventory_2),
              title: Text(objet.nom),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│ Sac (4)                            │
├────────────────────────────────────┤
│  Épée courte                       │
│  Bouclier de bois                  │
│  Arc long                          │
│  Torche                            │
│                                    │
│ ┌────────────────────────────────┐ │
│ │ Potion de soin supprimé ANNULER│ │
│ └────────────────────────────────┘ │
└────────────────────────────────────┘

Après appui sur ANNULER : la potion revient à sa place d'origine.
```

### Points de vigilance

- `hideCurrentSnackBar()` évite qu'une file de bannières s'accumule quand on supprime rapidement plusieurs éléments.
- `insert(indexOrigine, retire)` remet l'élément **à sa place**, pas à la fin.
- Si `indexOrigine` peut dépasser la longueur actuelle (parce que d'autres suppressions ont eu lieu), protégez-vous :
  `_objets.insert(indexOrigine.clamp(0, _objets.length), retire);`
- `ScaffoldMessenger.of(context)` doit être appelé pendant que le widget est encore monté. Si un `await` sépare l'appel de son usage, capturez la référence avant, comme ci-dessus.

---

## 48.27 — `RefreshIndicator` : tirer pour rafraîchir

`RefreshIndicator` ajoute le geste « tirer vers le bas pour recharger ».

```dart
RefreshIndicator({
  required Widget child,               // doit contenir un widget défilant
  required Future<void> Function() onRefresh,
  double displacement = 40.0,
  Color? color,
  Color? backgroundColor,
  double edgeOffset = 0.0,
})
```

Deux exigences absolues :

1. l'enfant (direct ou non) doit être un widget **défilant** : `ListView`, `GridView`, `CustomScrollView` ;
2. `onRefresh` doit retourner un `Future` qui **se termine** ; sinon l'animation tourne indéfiniment.

```dart
import 'dart:math';

import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationRafraichir());
}

class ApplicationRafraichir extends StatelessWidget {
  const ApplicationRafraichir({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranRafraichir(),
    );
  }
}

class EcranRafraichir extends StatefulWidget {
  const EcranRafraichir({super.key});

  @override
  State<EcranRafraichir> createState() => _EcranRafraichirState();
}

class _EcranRafraichirState extends State<EcranRafraichir> {
  final Random _alea = Random();
  List<int> _scores = <int>[];

  @override
  void initState() {
    super.initState();
    _scores = _genererScores();
  }

  List<int> _genererScores() {
    return List<int>.generate(15, (int i) => _alea.nextInt(9000) + 1000);
  }

  Future<void> _rafraichir() async {
    // Ici, un vrai appel réseau au chapitre 53.
    await Future<void>.delayed(const Duration(seconds: 1));
    if (!mounted) {
      return;
    }
    setState(() => _scores = _genererScores());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Scores en ligne')),
      body: RefreshIndicator(
        onRefresh: _rafraichir,
        child: ListView.builder(
          // Indispensable si la liste est courte : sans cela, le geste
          // de tirage n'est pas capté quand il n'y a rien à faire défiler.
          physics: const AlwaysScrollableScrollPhysics(),
          itemCount: _scores.length,
          itemBuilder: (BuildContext context, int index) {
            return ListTile(
              leading: CircleAvatar(child: Text('${index + 1}')),
              title: Text('Joueur ${index + 1}'),
              trailing: Text('${_scores[index]} pts'),
            );
          },
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Pendant le geste de tirage :

┌────────────────────────────────┐
│ Scores en ligne                │
├────────────────────────────────┤
│             ( ◌ )              │  <- roue de chargement
│  (1) Joueur 1        4821 pts  │
│  (2) Joueur 2        2317 pts  │
└────────────────────────────────┘

Une seconde plus tard, les scores ont changé :

┌────────────────────────────────┐
│ Scores en ligne                │
├────────────────────────────────┤
│  (1) Joueur 1        7104 pts  │
│  (2) Joueur 2        1553 pts  │
└────────────────────────────────┘
```

> **Erreur fréquente.** Écrire `onRefresh: () { _rafraichir(); }` (avec des accolades et sans `return`) donne une fonction qui retourne `void`, incompatible avec le type attendu : `The return type 'void' isn't a 'Future<void>', as required by the closure's context.` Écrivez `onRefresh: _rafraichir` ou `onRefresh: () async { await _rafraichir(); }`.

---

## 48.28 — `ReorderableListView`

`ReorderableListView` permet de réordonner les éléments par glisser-déposer.

Elle fournit automatiquement une poignée de déplacement (`buildDefaultDragHandles: true` par défaut) et appelle `onReorder(oldIndex, newIndex)` quand l'utilisateur relâche.

Deux règles impératives :

1. **chaque enfant doit avoir une `key` unique**, exactement pour la raison vue en 48.25 ;
2. **le calcul d'index doit compenser le décalage** : quand on déplace un élément vers le bas, `newIndex` est calculé *avant* le retrait de l'élément.

```text
Liste : [A, B, C, D]     on déplace A (index 0) après C

Flutter fournit : oldIndex = 0, newIndex = 3

Si l'on fait bêtement :
  removeAt(0)     -> [B, C, D]
  insert(3, A)    -> [B, C, D, A]     <- A est allé trop loin

Correction :
  if (newIndex > oldIndex) newIndex -= 1;
  removeAt(0)     -> [B, C, D]
  insert(2, A)    -> [B, C, A, D]     <- correct
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationOrdreEquipe());
}

class ApplicationOrdreEquipe extends StatelessWidget {
  const ApplicationOrdreEquipe({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranOrdreEquipe(),
    );
  }
}

class EcranOrdreEquipe extends StatefulWidget {
  const EcranOrdreEquipe({super.key});

  @override
  State<EcranOrdreEquipe> createState() => _EcranOrdreEquipeState();
}

class _EcranOrdreEquipeState extends State<EcranOrdreEquipe> {
  final List<String> _equipe = <String>[
    'Alex (Guerrier)',
    'Sophie (Mage)',
    'Samir (Voleur)',
    'Maria (Paladin)',
    'Léo (Archer)',
  ];

  void _reordonner(int ancienIndex, int nouvelIndex) {
    setState(() {
      if (nouvelIndex > ancienIndex) {
        nouvelIndex -= 1;
      }
      final String membre = _equipe.removeAt(ancienIndex);
      _equipe.insert(nouvelIndex, membre);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Ordre de combat')),
      body: ReorderableListView.builder(
        padding: const EdgeInsets.all(8),
        itemCount: _equipe.length,
        onReorder: _reordonner,
        itemBuilder: (BuildContext context, int index) {
          return Card(
            key: ValueKey<String>(_equipe[index]),
            child: ListTile(
              leading: CircleAvatar(child: Text('${index + 1}')),
              title: Text(_equipe[index]),
              trailing: const Icon(Icons.drag_handle),
            ),
          );
        },
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│ Ordre de combat                    │
├────────────────────────────────────┤
│ ╭────────────────────────────────╮ │
│ │ (1) Alex (Guerrier)         ≡  │ │
│ ╰────────────────────────────────╯ │
│ ╭────────────────────────────────╮ │
│ │ (2) Sophie (Mage)           ≡  │ │
│ ╰────────────────────────────────╯ │
│ ...                                │
└────────────────────────────────────┘

Après avoir déplacé Sophie en première position :

│ (1) Sophie (Mage)           ≡      │
│ (2) Alex (Guerrier)         ≡      │
```

> **La `key` va sur l'enfant direct**, ici la `Card`, pas sur le `ListTile` à l'intérieur. Si vous l'oubliez :
> `Every item of ReorderableListView must have a key.`

> **Note de version.** Les versions les plus récentes de Flutter déprécient `onReorder` au profit de `onReorderItem`, dont la signature est identique (`void Function(int oldIndex, int newIndex)`). Si votre éditeur signale `'onReorder' is deprecated`, remplacez simplement le nom du paramètre.

---

## 48.29 — L'état vide : quoi afficher quand la liste est vide

Une liste vide qui affiche un écran blanc est un défaut d'interface, pas un cas particulier. L'utilisateur ne sait pas si l'application est cassée, si elle charge encore, ou s'il n'y a réellement rien.

Trois états à distinguer :

```text
┌──────────────────┬───────────────────────────────────────────┐
│ CHARGEMENT       │ CircularProgressIndicator                 │
│                  │ « on ne sait pas encore »                 │
├──────────────────┼───────────────────────────────────────────┤
│ VIDE             │ icône + phrase + action proposée          │
│                  │ « on sait, et il n'y a rien »             │
├──────────────────┼───────────────────────────────────────────┤
│ VIDE APRÈS       │ « aucun résultat pour "xyz" »             │
│ FILTRAGE         │ + bouton « effacer le filtre »            │
└──────────────────┴───────────────────────────────────────────┘
```

Le troisième cas est le plus souvent oublié : « aucun ennemi » et « aucun ennemi ne correspond à votre recherche » ne se traitent pas de la même façon.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationEtatVide());
}

class ApplicationEtatVide extends StatelessWidget {
  const ApplicationEtatVide({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranEtatVide(),
    );
  }
}

/// Widget réutilisable d'état vide.
class EtatVide extends StatelessWidget {
  const EtatVide({
    super.key,
    required this.icone,
    required this.titre,
    required this.message,
    this.libelleAction,
    this.onAction,
  });

  final IconData icone;
  final String titre;
  final String message;
  final String? libelleAction;
  final VoidCallback? onAction;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Icon(icone, size: 72, color: Colors.grey),
            const SizedBox(height: 16),
            Text(titre,
                style: Theme.of(context).textTheme.titleLarge,
                textAlign: TextAlign.center),
            const SizedBox(height: 8),
            Text(message,
                style: const TextStyle(color: Colors.grey),
                textAlign: TextAlign.center),
            if (libelleAction != null && onAction != null) ...<Widget>[
              const SizedBox(height: 24),
              FilledButton(onPressed: onAction, child: Text(libelleAction!)),
            ],
          ],
        ),
      ),
    );
  }
}

class EcranEtatVide extends StatefulWidget {
  const EcranEtatVide({super.key});

  @override
  State<EcranEtatVide> createState() => _EcranEtatVideState();
}

class _EcranEtatVideState extends State<EcranEtatVide> {
  final List<String> _objets = <String>[];

  void _ajouter() {
    setState(() => _objets.add('Objet n°${_objets.length + 1}'));
  }

  void _vider() {
    setState(_objets.clear);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Sac (${_objets.length})'),
        actions: <Widget>[
          IconButton(onPressed: _vider, icon: const Icon(Icons.clear_all)),
        ],
      ),
      body: _objets.isEmpty
          ? EtatVide(
              icone: Icons.backpack_outlined,
              titre: 'Votre sac est vide',
              message: 'Ramassez des objets pendant vos aventures, '
                  'ou ajoutez-en un pour tester.',
              libelleAction: 'Ajouter un objet',
              onAction: _ajouter,
            )
          : ListView.builder(
              itemCount: _objets.length,
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  leading: const Icon(Icons.inventory_2),
                  title: Text(_objets[index]),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: _ajouter,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**Résultat :**

```text
Liste vide :                        Après trois ajouts :

┌────────────────────────┐          ┌────────────────────────┐
│ Sac (0)            [X] │          │ Sac (3)            [X] │
├────────────────────────┤          ├────────────────────────┤
│                        │          │  Objet n°1             │
│         ╭───╮          │          │  Objet n°2             │
│         │ []│          │          │  Objet n°3             │
│         ╰───╯          │          │                        │
│  Votre sac est vide    │          │                        │
│  Ramassez des objets   │          │                        │
│  pendant vos aventures │          │                        │
│                        │          │                        │
│  ╭──────────────────╮  │          │                        │
│  │ Ajouter un objet │  │          │                  ╭───╮ │
│  ╰──────────────────╯  │          │                  │ + │ │
└────────────────────────┘          └──────────────────╰───╯─┘
```

> **Remarque sur `...` dans une liste de widgets.** L'opérateur de diffusion conditionnel `if (condition) ...<Widget>[a, b]` vient du chapitre 06. Il permet d'inclure zéro, un ou plusieurs widgets selon une condition, directement dans `children`.

---

## 48.30 — Filtrer et trier une liste affichée (rappel chapitre 14)

Le chapitre 14 vous a donné `where`, `map`, `sort` et `toList`. Ce sont exactement les outils du filtrage et du tri à l'écran.

### La règle d'or : ne jamais modifier la liste source

```text
_tousLesEnnemis   <- la liste COMPLÈTE, jamais modifiée par un filtre
       │
       │  .where(...)      filtre
       ▼
_ennemisAffiches  <- la liste DÉRIVÉE, recalculée à chaque changement
       │
       ▼
ListView.builder(itemCount: _ennemisAffiches.length, ...)
```

Si vous filtrez en supprimant des éléments de la liste source, les éléments filtrés sont perdus : impossible de revenir en arrière.

### Filtrer

```dart
List<Ennemi> get _ennemisAffiches {
  return _tousLesEnnemis
      .where((Ennemi e) => e.niveau >= _niveauMinimum)
      .toList();
}
```

Un `getter` (chapitre 10) est parfait ici : la liste dérivée est recalculée automatiquement à chaque appel, sans qu'on ait à penser à la mettre à jour.

### Trier — et le piège de `sort`

`sort` **modifie la liste sur laquelle il est appelé** et retourne `void`.

```dart
// PIÈGE : sort modifie _tousLesEnnemis en place !
final List<Ennemi> tries = _tousLesEnnemis..sort(...);

// PIÈGE : sort retourne void
final List<Ennemi> tries = _tousLesEnnemis.sort(...);
// -> A value of type 'void' can't be assigned to a variable of type 'List<Ennemi>'.

// CORRECT : on trie une COPIE
final List<Ennemi> tries = List<Ennemi>.of(_tousLesEnnemis)
  ..sort((Ennemi a, Ennemi b) => a.niveau.compareTo(b.niveau));
```

`List.of(...)` fabrique une copie. L'opérateur en cascade `..sort(...)` applique le tri et retourne quand même la liste (rappel du chapitre 08).

### La fonction de comparaison

```text
compare(a, b) doit retourner :
  un nombre NÉGATIF  si a doit venir AVANT b
  ZÉRO               si l'ordre est indifférent
  un nombre POSITIF  si a doit venir APRÈS b

Croissant  : a.score.compareTo(b.score)
Décroissant: b.score.compareTo(a.score)     <- on inverse a et b
Par texte  : a.nom.toLowerCase().compareTo(b.nom.toLowerCase())
```

Exemple complet combinant filtre et tri :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationFiltreTri());
}

class Ennemi {
  const Ennemi({required this.nom, required this.niveau, required this.type});
  final String nom;
  final int niveau;
  final String type;
}

const List<Ennemi> bestiaire = <Ennemi>[
  Ennemi(nom: 'Gobelin', niveau: 2, type: 'Humanoïde'),
  Ennemi(nom: 'Loup des neiges', niveau: 6, type: 'Bête'),
  Ennemi(nom: 'Squelette', niveau: 3, type: 'Mort-vivant'),
  Ennemi(nom: 'Orc berserk', niveau: 9, type: 'Humanoïde'),
  Ennemi(nom: 'Liche', niveau: 22, type: 'Mort-vivant'),
  Ennemi(nom: 'Ours des cavernes', niveau: 11, type: 'Bête'),
  Ennemi(nom: 'Dragon rouge', niveau: 30, type: 'Dragon'),
  Ennemi(nom: 'Zombie', niveau: 1, type: 'Mort-vivant'),
];

class ApplicationFiltreTri extends StatelessWidget {
  const ApplicationFiltreTri({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranFiltreTri(),
    );
  }
}

class EcranFiltreTri extends StatefulWidget {
  const EcranFiltreTri({super.key});

  @override
  State<EcranFiltreTri> createState() => _EcranFiltreTriState();
}

class _EcranFiltreTriState extends State<EcranFiltreTri> {
  String _typeChoisi = 'Tous';
  bool _croissant = true;

  List<String> get _types {
    final Set<String> ensemble = bestiaire.map((Ennemi e) => e.type).toSet();
    return <String>['Tous', ...ensemble];
  }

  List<Ennemi> get _affiches {
    final List<Ennemi> filtres = _typeChoisi == 'Tous'
        ? List<Ennemi>.of(bestiaire)
        : bestiaire.where((Ennemi e) => e.type == _typeChoisi).toList();

    filtres.sort((Ennemi a, Ennemi b) => _croissant
        ? a.niveau.compareTo(b.niveau)
        : b.niveau.compareTo(a.niveau));

    return filtres;
  }

  @override
  Widget build(BuildContext context) {
    final List<Ennemi> affiches = _affiches;

    return Scaffold(
      appBar: AppBar(
        title: Text('Bestiaire (${affiches.length})'),
        actions: <Widget>[
          IconButton(
            tooltip: _croissant ? 'Tri croissant' : 'Tri décroissant',
            icon: Icon(_croissant ? Icons.arrow_upward : Icons.arrow_downward),
            onPressed: () => setState(() => _croissant = !_croissant),
          ),
        ],
      ),
      body: Column(
        children: <Widget>[
          SizedBox(
            height: 56,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 8),
              children: _types.map((String type) {
                return Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 4),
                  child: ChoiceChip(
                    label: Text(type),
                    selected: _typeChoisi == type,
                    onSelected: (bool _) =>
                        setState(() => _typeChoisi = type),
                  ),
                );
              }).toList(),
            ),
          ),
          const Divider(height: 1),
          Expanded(
            child: ListView.builder(
              itemCount: affiches.length,
              itemBuilder: (BuildContext context, int index) {
                final Ennemi ennemi = affiches[index];
                return ListTile(
                  leading: CircleAvatar(child: Text('${ennemi.niveau}')),
                  title: Text(ennemi.nom),
                  subtitle: Text(ennemi.type),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│ Bestiaire (3)                       [↑]  │
├──────────────────────────────────────────┤
│ (Tous) (Humanoïde) (Bête) [Mort-vivant]  │
│ ──────────────────────────────────────── │
│ (1)  Zombie                              │
│      Mort-vivant                         │
│ (3)  Squelette                           │
│      Mort-vivant                         │
│ (22) Liche                               │
│      Mort-vivant                         │
└──────────────────────────────────────────┘
```

---

## 48.31 — La barre de recherche

Une recherche textuelle est un filtre dont le critère est saisi par l'utilisateur. Il faut donc un `TextField` et un `TextEditingController` — vus en détail au chapitre 49, utilisés ici dans leur forme la plus simple.

Les trois règles d'une recherche correcte :

```text
1. Comparer en MINUSCULES des deux côtés :
   e.nom.toLowerCase().contains(_requete.toLowerCase())

2. Supprimer les espaces parasites : .trim()

3. Une requête vide affiche TOUT, pas rien.
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationRecherche());
}

class Ennemi {
  const Ennemi({required this.nom, required this.niveau, required this.type});
  final String nom;
  final int niveau;
  final String type;
}

const List<Ennemi> bestiaire = <Ennemi>[
  Ennemi(nom: 'Gobelin', niveau: 2, type: 'Humanoïde'),
  Ennemi(nom: 'Loup des neiges', niveau: 6, type: 'Bête'),
  Ennemi(nom: 'Squelette', niveau: 3, type: 'Mort-vivant'),
  Ennemi(nom: 'Orc berserk', niveau: 9, type: 'Humanoïde'),
  Ennemi(nom: 'Liche', niveau: 22, type: 'Mort-vivant'),
  Ennemi(nom: 'Ours des cavernes', niveau: 11, type: 'Bête'),
  Ennemi(nom: 'Dragon rouge', niveau: 30, type: 'Dragon'),
  Ennemi(nom: 'Zombie', niveau: 1, type: 'Mort-vivant'),
];

class ApplicationRecherche extends StatelessWidget {
  const ApplicationRecherche({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranRecherche(),
    );
  }
}

class EcranRecherche extends StatefulWidget {
  const EcranRecherche({super.key});

  @override
  State<EcranRecherche> createState() => _EcranRechercheState();
}

class _EcranRechercheState extends State<EcranRecherche> {
  final TextEditingController _controleurTexte = TextEditingController();
  String _requete = '';

  @override
  void dispose() {
    _controleurTexte.dispose();
    super.dispose();
  }

  List<Ennemi> get _resultats {
    final String requete = _requete.trim().toLowerCase();
    if (requete.isEmpty) {
      return bestiaire;
    }
    return bestiaire.where((Ennemi e) {
      return e.nom.toLowerCase().contains(requete) ||
          e.type.toLowerCase().contains(requete);
    }).toList();
  }

  @override
  Widget build(BuildContext context) {
    final List<Ennemi> resultats = _resultats;

    return Scaffold(
      appBar: AppBar(title: const Text('Rechercher un ennemi')),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(12),
            child: TextField(
              controller: _controleurTexte,
              decoration: InputDecoration(
                hintText: 'Nom ou type...',
                prefixIcon: const Icon(Icons.search),
                border: const OutlineInputBorder(),
                suffixIcon: _requete.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _controleurTexte.clear();
                          setState(() => _requete = '');
                        },
                      ),
              ),
              onChanged: (String valeur) => setState(() => _requete = valeur),
            ),
          ),
          Expanded(
            child: resultats.isEmpty
                ? Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: <Widget>[
                        const Icon(Icons.search_off,
                            size: 64, color: Colors.grey),
                        const SizedBox(height: 12),
                        Text('Aucun ennemi ne correspond à "$_requete"'),
                        TextButton(
                          onPressed: () {
                            _controleurTexte.clear();
                            setState(() => _requete = '');
                          },
                          child: const Text('Effacer la recherche'),
                        ),
                      ],
                    ),
                  )
                : ListView.builder(
                    itemCount: resultats.length,
                    itemBuilder: (BuildContext context, int index) {
                      final Ennemi ennemi = resultats[index];
                      return ListTile(
                        leading: CircleAvatar(child: Text('${ennemi.niveau}')),
                        title: Text(ennemi.nom),
                        subtitle: Text(ennemi.type),
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

**Résultat :**

```text
Recherche « or » :                  Recherche « xyz » :

┌────────────────────────────┐      ┌────────────────────────────┐
│ Rechercher un ennemi       │      │ Rechercher un ennemi       │
├────────────────────────────┤      ├────────────────────────────┤
│ ┌────────────────────────┐ │      │ ┌────────────────────────┐ │
│ │ [q] or              x  │ │      │ │ [q] xyz             x  │ │
│ └────────────────────────┘ │      │ └────────────────────────┘ │
│ (3)  Squelette             │      │           ╭───╮            │
│      Mort-vivant           │      │           │ ? │            │
│ (9)  Orc berserk           │      │           ╰───╯            │
│      Humanoïde             │      │ Aucun ennemi ne correspond │
│ (22) Liche                 │      │ à "xyz"                    │
│      Mort-vivant           │      │   Effacer la recherche     │
│ (1)  Zombie                │      │                            │
│      Mort-vivant           │      │                            │
└────────────────────────────┘      └────────────────────────────┘
```

> **Pourquoi « Squelette » apparaît-il alors que son nom ne contient pas « or » ?** Parce que la recherche porte aussi sur le **type** : `mort-vivant` contient bien `or`. C'est le comportement voulu ici, mais il surprend souvent l'utilisateur. Si vous ne voulez chercher que dans le nom, retirez la seconde condition du `where`.

---

## 48.32 — `CustomScrollView` et les slivers : introduction

Jusqu'ici, un écran ne contenait qu'**une** zone défilante. Que faire quand on veut, dans un **seul** mouvement de défilement :

```text
┌────────────────────────────┐
│  une grande image d'en-tête│  <- se replie en défilant
├────────────────────────────┤
│  un bandeau de statistiques│  <- bloc unique
├────────────────────────────┤
│  une grille de 3 objets    │  <- grille
├────────────────────────────┤
│  une liste de 200 lignes   │  <- liste
└────────────────────────────┘
```

Empiler une `GridView` et une `ListView` dans une `Column` avec des `shrinkWrap` fonctionne mal : deux zones défilantes indépendantes, des performances catastrophiques et un défilement qui « saute ».

La réponse de Flutter est le **sliver**.

### Qu'est-ce qu'un sliver ?

> Un **sliver** est un morceau de contenu défilant. Ce n'est pas un widget ordinaire : il ne sait pas se placer tout seul. Il doit vivre dans la liste `slivers` d'un `CustomScrollView`.

```text
CustomScrollView          = LE moteur de défilement, unique
  slivers: [
    SliverAppBar          = un en-tête qui réagit au défilement
    SliverToBoxAdapter    = « je transforme UN widget normal en sliver »
    SliverList            = l'équivalent de ListView
    SliverGrid            = l'équivalent de GridView
    SliverPadding         = l'équivalent de Padding
    SliverFillRemaining   = « je remplis ce qui reste de l'écran »
  ]
```

### La table de conversion

| Widget ordinaire | Sliver équivalent |
| --- | --- |
| `ListView(children: ...)` | `SliverList.list(children: ...)` |
| `ListView.builder` | `SliverList.builder(itemBuilder:, itemCount:)` |
| `ListView.separated` | `SliverList.separated(...)` |
| `GridView.builder` | `SliverGrid.builder(gridDelegate:, itemBuilder:)` |
| `Padding` | `SliverPadding(sliver: ...)` |
| n'importe quel widget | `SliverToBoxAdapter(child: ...)` |
| `AppBar` | `SliverAppBar` |

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSlivers());
}

class ApplicationSlivers extends StatelessWidget {
  const ApplicationSlivers({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranSlivers(),
    );
  }
}

class EcranSlivers extends StatelessWidget {
  const EcranSlivers({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: <Widget>[
          const SliverAppBar(
            title: Text('Donjon'),
            pinned: true,
          ),

          // Un widget ordinaire, transformé en sliver.
          SliverToBoxAdapter(
            child: Container(
              height: 90,
              color: Colors.indigo.shade50,
              alignment: Alignment.center,
              child: const Text('Progression : 12 / 40 salles'),
            ),
          ),

          // Une grille.
          SliverPadding(
            padding: const EdgeInsets.all(12),
            sliver: SliverGrid.builder(
              itemCount: 6,
              gridDelegate:
                  const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                mainAxisSpacing: 8,
                crossAxisSpacing: 8,
              ),
              itemBuilder: (BuildContext context, int index) {
                return Container(
                  decoration: BoxDecoration(
                    color: Colors.teal.shade100,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  alignment: Alignment.center,
                  child: Text('Clé ${index + 1}'),
                );
              },
            ),
          ),

          // Une liste, dans le MÊME défilement.
          SliverList.builder(
            itemCount: 40,
            itemBuilder: (BuildContext context, int index) {
              return ListTile(
                leading: CircleAvatar(child: Text('${index + 1}')),
                title: Text('Salle n°${index + 1}'),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│ Donjon                             │  <- SliverAppBar (pinned)
├────────────────────────────────────┤
│   Progression : 12 / 40 salles     │  <- SliverToBoxAdapter
├────────────────────────────────────┤
│ ╭────╮ ╭────╮ ╭────╮               │
│ │Clé1│ │Clé2│ │Clé3│               │  <- SliverGrid
│ ╰────╯ ╰────╯ ╰────╯               │
│ ╭────╮ ╭────╮ ╭────╮               │
│ │Clé4│ │Clé5│ │Clé6│               │
│ ╰────╯ ╰────╯ ╰────╯               │
│ (1) Salle n°1                      │  <- SliverList
│ (2) Salle n°2                      │
└────────────────────────────────────┘

Un SEUL geste de défilement fait tout bouger ensemble.
```

> **Erreur classique.** Mettre un widget ordinaire directement dans `slivers` :
> `A RenderViewport expected a child of type RenderSliver but received a child of type RenderConstrainedBox.`
> Enveloppez-le dans un `SliverToBoxAdapter`.

---

## 48.33 — `SliverAppBar` et l'en-tête qui se replie

`SliverAppBar` est l'`AppBar` du monde des slivers. Elle réagit au défilement.

| Paramètre | Effet |
| --- | --- |
| `expandedHeight` | hauteur quand elle est totalement déployée |
| `pinned` | la barre de titre reste visible en haut, repliée |
| `floating` | la barre réapparaît dès qu'on défile vers le haut, même au milieu |
| `snap` | avec `floating: true`, elle se déploie d'un coup |
| `stretch` | elle s'étire quand on tire au-delà du haut |
| `flexibleSpace` | le contenu qui se replie, généralement un `FlexibleSpaceBar` |

```text
pinned: true, floating: false          pinned: false, floating: false
(le plus courant)

défilement 0    ┌──────────────┐       ┌──────────────┐
                │              │       │              │
                │  EN-TÊTE     │       │  EN-TÊTE     │
                │              │       │              │
                └──────────────┘       └──────────────┘

défilement 150  ┌──────────────┐       (l'en-tête a totalement
                │ Titre        │        disparu de l'écran)
                └──────────────┘
                la barre reste
```

`FlexibleSpaceBar` gère l'animation du titre et du fond :

```dart
FlexibleSpaceBar({
  Widget? title,
  Widget? background,
  bool? centerTitle,
  EdgeInsetsGeometry? titlePadding,
  CollapseMode collapseMode = CollapseMode.parallax,
  List<StretchMode> stretchModes = const <StretchMode>[StretchMode.zoomBackground],
})
```

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationEnTete());
}

class ApplicationEnTete extends StatelessWidget {
  const ApplicationEnTete({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranEnTete(),
    );
  }
}

class EcranEnTete extends StatelessWidget {
  const EcranEnTete({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: <Widget>[
          SliverAppBar(
            expandedHeight: 220,
            pinned: true,
            stretch: true,
            backgroundColor: Colors.deepPurple,
            foregroundColor: Colors.white,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('Forêt maudite'),
              centerTitle: true,
              stretchModes: const <StretchMode>[
                StretchMode.zoomBackground,
                StretchMode.fadeTitle,
              ],
              background: DecoratedBox(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topCenter,
                    end: Alignment.bottomCenter,
                    colors: <Color>[
                      Colors.deepPurple.shade300,
                      Colors.deepPurple.shade900,
                    ],
                  ),
                ),
                child: const Center(
                  child: Icon(Icons.forest, size: 90, color: Colors.white24),
                ),
              ),
            ),
          ),
          SliverList.builder(
            itemCount: 40,
            itemBuilder: (BuildContext context, int index) {
              return ListTile(
                leading: const Icon(Icons.pest_control),
                title: Text('Rencontre n°${index + 1}'),
                subtitle: Text('Niveau ${index % 12 + 1}'),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
En haut de la liste :               Après avoir défilé :

┌────────────────────────────┐      ┌────────────────────────────┐
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░│      │      Forêt maudite         │ <- repliée
│░░░░░░░░░ /\\ ░░░░░░░░░░░░░░░│      ├────────────────────────────┤
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░│      │ >  Rencontre n°14          │
│░░░░░ Forêt maudite ░░░░░░░░│      │    Niveau 2                │
├────────────────────────────┤      │ >  Rencontre n°15          │
│ >  Rencontre n°1           │      │    Niveau 3                │
│    Niveau 1                │      │ >  Rencontre n°16          │
└────────────────────────────┘      └────────────────────────────┘
```

> **Attention.** Un `Scaffold` ne doit pas avoir à la fois `appBar:` et un `SliverAppBar` dans son `body`. Vous auriez deux barres empilées. Avec `CustomScrollView`, la barre vit **dans** les slivers.

---

## 48.34 — Performance : `const`, `itemExtent`, images en cache

Une liste qui « accroche » au défilement vient presque toujours de l'une de ces cinq causes.

### 1. Des widgets non `const`

Un widget `const` est construit **une seule fois** pour toute la durée du programme. Flutter le reconnaît et ne le reconstruit jamais.

```dart
// 500 objets Icon créés, un par ligne
leading: Icon(Icons.star),

// 1 seul objet Icon, partagé par les 500 lignes
leading: const Icon(Icons.star),
```

Activez le lint `prefer_const_constructors` dans `analysis_options.yaml` : l'éditeur vous signalera automatiquement chaque `const` manquant.

### 2. Une hauteur de ligne inconnue

Par défaut, Flutter doit **mesurer** chaque ligne pour savoir où elle commence. Si toutes vos lignes font la même hauteur, dites-le :

```dart
ListView.builder(
  itemExtent: 72,          // toutes les lignes font 72 pixels
  itemCount: 100000,
  itemBuilder: (BuildContext context, int index) => const TuileEnnemi(),
)
```

Avec `itemExtent`, Flutter calcule la position par une multiplication au lieu de mesurer. La barre de défilement devient exacte immédiatement et `jumpTo` est instantané, même sur 100 000 éléments.

Variante quand la hauteur est constante mais inconnue à l'écriture :

```dart
prototypeItem: const ListTile(title: Text('exemple')),
```

Flutter mesure **une seule fois** ce prototype et applique sa hauteur à toutes les lignes.

| Paramètre | Utiliser quand |
| --- | --- |
| `itemExtent: 72` | vous connaissez la hauteur exacte en pixels |
| `prototypeItem: ...` | toutes les lignes ont la même hauteur, mais vous ne la connaissez pas |
| ni l'un ni l'autre | les lignes ont des hauteurs différentes |

Ces deux paramètres sont **exclusifs** : `Failed assertion: itemExtent == null || prototypeItem == null`.

### 3. Des images rechargées à chaque défilement

`Image.network` recharge l'image dès que la ligne sort puis revient dans le viewport. Sur une liste, c'est visible et coûteux.

La solution standard est le paquet `cached_network_image` :

```text
flutter pub add cached_network_image
```

```dart
CachedNetworkImage(
  imageUrl: 'https://picsum.photos/seed/$index/200/200',
  width: 56,
  height: 56,
  fit: BoxFit.cover,
  placeholder: (BuildContext context, String url) =>
      const SizedBox(width: 56, height: 56),
  errorWidget: (BuildContext context, String url, Object error) =>
      const Icon(Icons.broken_image),
)
```

Sans paquet supplémentaire, deux réflexes aident déjà beaucoup :

- donner une **taille fixe** à l'image (`width`, `height`), pour éviter que la ligne change de hauteur au chargement ;
- utiliser `cacheWidth` / `cacheHeight` sur `Image.network` pour décoder l'image à la taille d'affichage plutôt qu'en pleine résolution.

```dart
Image.network(
  'https://picsum.photos/seed/$index/400/400',
  width: 56,
  height: 56,
  cacheWidth: 112,   // 56 * 2 pour un écran haute densité
  fit: BoxFit.cover,
)
```

### 4. Du calcul dans `itemBuilder`

```dart
// MAUVAIS : le tri est refait à CHAQUE ligne construite
itemBuilder: (BuildContext context, int index) {
  final List<Ennemi> tries = List<Ennemi>.of(bestiaire)..sort(...);
  return Text(tries[index].nom);
}

// BON : le tri est fait une fois, avant la ListView
final List<Ennemi> tries = List<Ennemi>.of(bestiaire)..sort(...);
// ...
itemBuilder: (BuildContext context, int index) => Text(tries[index].nom),
```

### 5. Une liste imbriquée avec `shrinkWrap` sur beaucoup d'éléments

Rappel de 48.15 : `shrinkWrap: true` construit **tous** les enfants. Sur 500 lignes, c'est le pire choix possible. Utilisez `CustomScrollView` et des slivers (48.32).

### Le récapitulatif

| Symptôme | Cause probable | Correction |
| --- | --- | --- |
| saccades constantes | widgets non `const` | ajouter `const` |
| barre de défilement qui saute | hauteurs mesurées | `itemExtent` ou `prototypeItem` |
| images qui clignotent | `Image.network` sans cache | `cached_network_image` |
| gel à l'ouverture | `shrinkWrap` sur une longue liste | `Expanded` ou slivers |
| ralentissement progressif | calcul dans `itemBuilder` | sortir le calcul |

---

## 48.35 — Mini-projet : une liste d'ennemis filtrable, triable et supprimable

Ce projet réunit toutes les notions du chapitre : modèle, `ListView.builder`, `ListTile`, `Card`, recherche, filtre, tri, `Dismissible`, `SnackBar` d'annulation, `RefreshIndicator`, état vide et `ScrollController`.

### Cahier des charges

```text
1. Afficher un bestiaire d'ennemis (nom, type, niveau, points de vie).
2. Une barre de recherche filtre par nom et par type.
3. Des puces filtrent par type.
4. Un menu permet de trier par nom, par niveau ou par points de vie.
5. Un glissement vers la gauche supprime un ennemi.
6. Un SnackBar permet d'annuler la suppression.
7. Tirer vers le bas restaure le bestiaire complet.
8. Un état vide distinct selon qu'il n'y a rien ou que le filtre ne donne rien.
9. Un bouton flottant remonte en haut de la liste.
```

### Le code complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationBestiaire());
}

// ---------------------------------------------------------------------------
// MODÈLE
// ---------------------------------------------------------------------------

class Ennemi {
  const Ennemi({
    required this.id,
    required this.nom,
    required this.type,
    required this.niveau,
    required this.pointsDeVie,
  });

  final String id;
  final String nom;
  final String type;
  final int niveau;
  final int pointsDeVie;
}

enum CritereTri { nom, niveau, pointsDeVie }

extension LibelleTri on CritereTri {
  String get libelle => switch (this) {
        CritereTri.nom => 'Nom',
        CritereTri.niveau => 'Niveau',
        CritereTri.pointsDeVie => 'Points de vie',
      };
}

const List<Ennemi> bestiaireInitial = <Ennemi>[
  Ennemi(id: 'e1', nom: 'Gobelin', type: 'Humanoïde', niveau: 2, pointsDeVie: 30),
  Ennemi(id: 'e2', nom: 'Loup des neiges', type: 'Bête', niveau: 6, pointsDeVie: 55),
  Ennemi(id: 'e3', nom: 'Squelette', type: 'Mort-vivant', niveau: 3, pointsDeVie: 45),
  Ennemi(id: 'e4', nom: 'Orc berserk', type: 'Humanoïde', niveau: 9, pointsDeVie: 120),
  Ennemi(id: 'e5', nom: 'Liche', type: 'Mort-vivant', niveau: 22, pointsDeVie: 300),
  Ennemi(id: 'e6', nom: 'Ours des cavernes', type: 'Bête', niveau: 11, pointsDeVie: 180),
  Ennemi(id: 'e7', nom: 'Dragon rouge', type: 'Dragon', niveau: 30, pointsDeVie: 500),
  Ennemi(id: 'e8', nom: 'Zombie', type: 'Mort-vivant', niveau: 1, pointsDeVie: 25),
  Ennemi(id: 'e9', nom: 'Troll des marais', type: 'Humanoïde', niveau: 14, pointsDeVie: 210),
  Ennemi(id: 'e10', nom: 'Wyverne', type: 'Dragon', niveau: 18, pointsDeVie: 260),
  Ennemi(id: 'e11', nom: 'Rat géant', type: 'Bête', niveau: 1, pointsDeVie: 12),
  Ennemi(id: 'e12', nom: 'Spectre', type: 'Mort-vivant', niveau: 16, pointsDeVie: 140),
];

// ---------------------------------------------------------------------------
// APPLICATION
// ---------------------------------------------------------------------------

class ApplicationBestiaire extends StatelessWidget {
  const ApplicationBestiaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bestiaire',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.deepPurple),
      home: const EcranBestiaire(),
    );
  }
}

// ---------------------------------------------------------------------------
// TUILE
// ---------------------------------------------------------------------------

class TuileEnnemi extends StatelessWidget {
  const TuileEnnemi({super.key, required this.ennemi});

  final Ennemi ennemi;

  Color _couleurType(BuildContext context) => switch (ennemi.type) {
        'Humanoïde' => Colors.brown,
        'Bête' => Colors.green,
        'Mort-vivant' => Colors.blueGrey,
        'Dragon' => Colors.red,
        _ => Colors.grey,
      };

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: _couleurType(context).withValues(alpha: 0.2),
          child: Text(
            '${ennemi.niveau}',
            style: TextStyle(
              color: _couleurType(context),
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        title: Text(ennemi.nom),
        subtitle: Text('${ennemi.type} — ${ennemi.pointsDeVie} PV'),
        trailing: const Icon(Icons.chevron_right),
        onTap: () {
          showDialog<void>(
            context: context,
            builder: (BuildContext context) => AlertDialog(
              title: Text(ennemi.nom),
              content: Text(
                'Type : ${ennemi.type}\n'
                'Niveau : ${ennemi.niveau}\n'
                'Points de vie : ${ennemi.pointsDeVie}',
              ),
              actions: <Widget>[
                TextButton(
                  onPressed: () => Navigator.of(context).pop(),
                  child: const Text('Fermer'),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// ÉCRAN
// ---------------------------------------------------------------------------

class EcranBestiaire extends StatefulWidget {
  const EcranBestiaire({super.key});

  @override
  State<EcranBestiaire> createState() => _EcranBestiaireState();
}

class _EcranBestiaireState extends State<EcranBestiaire> {
  final TextEditingController _controleurTexte = TextEditingController();
  final ScrollController _controleurDefilement = ScrollController();

  List<Ennemi> _bestiaire = List<Ennemi>.of(bestiaireInitial);
  String _requete = '';
  String _typeChoisi = 'Tous';
  CritereTri _critere = CritereTri.niveau;
  bool _croissant = true;
  bool _boutonHautVisible = false;

  @override
  void initState() {
    super.initState();
    _controleurDefilement.addListener(_surDefilement);
  }

  @override
  void dispose() {
    _controleurDefilement.removeListener(_surDefilement);
    _controleurDefilement.dispose();
    _controleurTexte.dispose();
    super.dispose();
  }

  void _surDefilement() {
    final bool visible = _controleurDefilement.offset > 300;
    if (visible != _boutonHautVisible) {
      setState(() => _boutonHautVisible = visible);
    }
  }

  List<String> get _types {
    final Set<String> ensemble =
        _bestiaire.map((Ennemi e) => e.type).toSet();
    final List<String> liste = ensemble.toList()..sort();
    return <String>['Tous', ...liste];
  }

  List<Ennemi> get _affiches {
    final String requete = _requete.trim().toLowerCase();

    final List<Ennemi> resultat = _bestiaire.where((Ennemi e) {
      final bool bonType = _typeChoisi == 'Tous' || e.type == _typeChoisi;
      final bool bonTexte = requete.isEmpty ||
          e.nom.toLowerCase().contains(requete) ||
          e.type.toLowerCase().contains(requete);
      return bonType && bonTexte;
    }).toList();

    resultat.sort((Ennemi a, Ennemi b) {
      final int comparaison = switch (_critere) {
        CritereTri.nom => a.nom.toLowerCase().compareTo(b.nom.toLowerCase()),
        CritereTri.niveau => a.niveau.compareTo(b.niveau),
        CritereTri.pointsDeVie => a.pointsDeVie.compareTo(b.pointsDeVie),
      };
      return _croissant ? comparaison : -comparaison;
    });

    return resultat;
  }

  void _supprimer(Ennemi ennemi) {
    final int indexOrigine = _bestiaire.indexOf(ennemi);
    setState(() => _bestiaire.remove(ennemi));

    final ScaffoldMessengerState messenger = ScaffoldMessenger.of(context);
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      SnackBar(
        content: Text('${ennemi.nom} retiré du bestiaire'),
        duration: const Duration(seconds: 4),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () {
            setState(() {
              _bestiaire.insert(
                indexOrigine.clamp(0, _bestiaire.length),
                ennemi,
              );
            });
          },
        ),
      ),
    );
  }

  Future<void> _restaurer() async {
    await Future<void>.delayed(const Duration(milliseconds: 600));
    if (!mounted) {
      return;
    }
    setState(() {
      _bestiaire = List<Ennemi>.of(bestiaireInitial);
      _typeChoisi = 'Tous';
      _requete = '';
      _controleurTexte.clear();
    });
  }

  void _effacerRecherche() {
    _controleurTexte.clear();
    setState(() => _requete = '');
  }

  @override
  Widget build(BuildContext context) {
    final List<Ennemi> affiches = _affiches;
    final bool filtreActif = _requete.trim().isNotEmpty || _typeChoisi != 'Tous';

    return Scaffold(
      appBar: AppBar(
        title: Text('Bestiaire (${affiches.length})'),
        actions: <Widget>[
          IconButton(
            tooltip: _croissant ? 'Croissant' : 'Décroissant',
            icon: Icon(_croissant ? Icons.arrow_upward : Icons.arrow_downward),
            onPressed: () => setState(() => _croissant = !_croissant),
          ),
          PopupMenuButton<CritereTri>(
            tooltip: 'Trier par',
            icon: const Icon(Icons.sort),
            initialValue: _critere,
            onSelected: (CritereTri valeur) =>
                setState(() => _critere = valeur),
            itemBuilder: (BuildContext context) {
              return CritereTri.values.map((CritereTri critere) {
                return PopupMenuItem<CritereTri>(
                  value: critere,
                  child: Text(critere.libelle),
                );
              }).toList();
            },
          ),
        ],
      ),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.fromLTRB(12, 12, 12, 8),
            child: TextField(
              controller: _controleurTexte,
              decoration: InputDecoration(
                hintText: 'Rechercher un ennemi...',
                prefixIcon: const Icon(Icons.search),
                border: const OutlineInputBorder(),
                isDense: true,
                suffixIcon: _requete.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: _effacerRecherche,
                      ),
              ),
              onChanged: (String valeur) => setState(() => _requete = valeur),
            ),
          ),
          SizedBox(
            height: 48,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 8),
              children: _types.map((String type) {
                return Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 4),
                  child: ChoiceChip(
                    label: Text(type),
                    selected: _typeChoisi == type,
                    onSelected: (bool _) =>
                        setState(() => _typeChoisi = type),
                  ),
                );
              }).toList(),
            ),
          ),
          const Divider(height: 1),
          Expanded(
            child: RefreshIndicator(
              onRefresh: _restaurer,
              child: affiches.isEmpty
                  ? _construireEtatVide(filtreActif)
                  : ListView.builder(
                      controller: _controleurDefilement,
                      physics: const AlwaysScrollableScrollPhysics(),
                      padding: const EdgeInsets.symmetric(vertical: 8),
                      itemCount: affiches.length,
                      itemBuilder: (BuildContext context, int index) {
                        final Ennemi ennemi = affiches[index];
                        return Dismissible(
                          key: ValueKey<String>(ennemi.id),
                          direction: DismissDirection.endToStart,
                          background: Container(
                            margin: const EdgeInsets.symmetric(
                                horizontal: 10, vertical: 4),
                            decoration: BoxDecoration(
                              color: Colors.red,
                              borderRadius: BorderRadius.circular(12),
                            ),
                            alignment: Alignment.centerRight,
                            padding: const EdgeInsets.only(right: 24),
                            child: const Icon(Icons.delete,
                                color: Colors.white),
                          ),
                          onDismissed: (DismissDirection direction) =>
                              _supprimer(ennemi),
                          child: TuileEnnemi(ennemi: ennemi),
                        );
                      },
                    ),
            ),
          ),
        ],
      ),
      floatingActionButton: _boutonHautVisible
          ? FloatingActionButton.small(
              tooltip: 'Remonter',
              onPressed: () => _controleurDefilement.animateTo(
                0,
                duration: const Duration(milliseconds: 450),
                curve: Curves.easeOut,
              ),
              child: const Icon(Icons.arrow_upward),
            )
          : null,
    );
  }

  Widget _construireEtatVide(bool filtreActif) {
    // ListView pour que le geste de tirage fonctionne même sur l'état vide.
    return ListView(
      physics: const AlwaysScrollableScrollPhysics(),
      children: <Widget>[
        const SizedBox(height: 80),
        Icon(
          filtreActif ? Icons.search_off : Icons.pest_control_outlined,
          size: 72,
          color: Colors.grey,
        ),
        const SizedBox(height: 16),
        Center(
          child: Text(
            filtreActif
                ? 'Aucun ennemi ne correspond'
                : 'Le bestiaire est vide',
            style: Theme.of(context).textTheme.titleLarge,
          ),
        ),
        const SizedBox(height: 8),
        Center(
          child: Text(
            filtreActif
                ? 'Modifiez la recherche ou le filtre de type.'
                : 'Tirez vers le bas pour restaurer le bestiaire.',
            style: const TextStyle(color: Colors.grey),
            textAlign: TextAlign.center,
          ),
        ),
        const SizedBox(height: 24),
        Center(
          child: FilledButton.tonal(
            onPressed: filtreActif
                ? () {
                    _effacerRecherche();
                    setState(() => _typeChoisi = 'Tous');
                  }
                : _restaurer,
            child: Text(filtreActif ? 'Réinitialiser' : 'Restaurer'),
          ),
        ),
      ],
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────────┐
│ Bestiaire (4)                    [↑] [≡]   │
├────────────────────────────────────────────┤
│ ┌────────────────────────────────────────┐ │
│ │ [q] Rechercher un ennemi...             │ │
│ └────────────────────────────────────────┘ │
│ (Tous) (Bête) (Dragon) [Mort-vivant] (Hu.. │
│ ────────────────────────────────────────── │
│ ╭────────────────────────────────────────╮ │
│ │ (1)  Zombie                        >   │ │
│ │      Mort-vivant — 25 PV               │ │
│ ╰────────────────────────────────────────╯ │
│ ╭────────────────────────────────────────╮ │
│ │ (3)  Squelette                     >   │ │
│ │      Mort-vivant — 45 PV               │ │
│ ╰────────────────────────────────────────╯ │
│ ╭────────────────────────────────────────╮ │
│ │ (16) Spectre                       >   │ │
│ │      Mort-vivant — 140 PV              │ │
│ ╰────────────────────────────────────────╯ │
│ ╭────────────────────────────────────────╮ │
│ │ (22) Liche                         >   │ │
│ │      Mort-vivant — 300 PV              │ │
│ ╰────────────────────────────────────────╯ │
└────────────────────────────────────────────┘
```

### Ce qu'il faut retenir de l'architecture

```text
_bestiaire          la SOURCE. Modifiée seulement par
                    suppression et restauration.

_affiches           un GETTER. Recalculé à chaque build à
                    partir de _requete, _typeChoisi, _critere
                    et _croissant. Jamais stocké.

Dismissible.key     ValueKey(ennemi.id), donc stable même
                    quand le tri change l'ordre d'affichage.

_supprimer          retire de _bestiaire (la source), pas de
                    _affiches (la vue dérivée).
```

Ce dernier point est la clé : **on agit toujours sur la source, jamais sur la liste affichée**. Si vous supprimiez de `_affiches`, la modification disparaîtrait au prochain `build`.

---

## 48.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Vertical viewport was given unbounded height.` | une `ListView` verticale placée directement dans une `Column`, un `SingleChildScrollView` ou une autre `ListView` | envelopper dans `Expanded`, ou fixer la hauteur avec `SizedBox`, ou en dernier recours `shrinkWrap: true` + `physics: NeverScrollableScrollPhysics()` |
| `Horizontal viewport was given unbounded width.` | une `ListView` horizontale dans une `Row` sans contrainte de largeur | `Expanded` ou `SizedBox(width: ...)` |
| `RenderBox was not laid out: RenderViewport#... NEEDS-LAYOUT NEEDS-PAINT` | conséquence de l'erreur précédente : la liste n'a pas pu être mesurée | corriger la contrainte, pas le symptôme |
| `A RenderFlex overflowed by 214 pixels on the bottom.` | une `Column` (ou une case de grille) dont le contenu dépasse | remplacer la `Column` par une `ListView`, ou baisser `childAspectRatio`, ou utiliser `mainAxisExtent` |
| application qui gèle à l'ouverture d'un écran | `ListView(children: ...)` ou `shrinkWrap: true` sur une longue collection | `ListView.builder` sans `shrinkWrap` |
| `RangeError (index): Index out of range: index should be less than 20: 20` | `itemCount` supérieur au nombre réel d'éléments, ou `itemCount` oublié, ou ligne supplémentaire non testée dans `itemBuilder` | `itemCount: liste.length` (jamais un nombre en dur) ; tester `if (index < liste.length)` quand on ajoute un pied de liste |
| une ligne sur deux disparaît en supprimant | `Dismissible` sans `Key`, ou avec `ValueKey(index)` | `key: ValueKey(objet.id)` — un identifiant **stable** |
| `A dismissed Dismissible widget is still part of the tree.` | `onDismissed` n'a pas retiré la donnée de la liste, ou l'a fait hors d'un `setState` | `setState(() => _liste.removeAt(index));` dans `onDismissed` |
| `Duplicate keys found. If multiple keyed nodes exist as children of another node, they must have unique keys.` | deux éléments partagent la même `ValueKey` (nom identique) | ajouter un champ `id` unique au modèle |
| `Every item of ReorderableListView must have a key.` | la `key` a été mise sur un widget interne au lieu de l'enfant direct | placer la `key` sur le widget retourné par `itemBuilder` |
| après un glisser-déposer, l'élément atterrit une case trop loin | oubli de la correction d'index dans `onReorder` | `if (newIndex > oldIndex) newIndex -= 1;` avant `removeAt` / `insert` |
| `'package:flutter/src/widgets/scroll_controller.dart': Failed assertion: line 116 pos 12: '_positions.isNotEmpty': ScrollController not attached to any scroll views.` | lecture de `offset` ou `position` avant que la liste ne soit construite | tester `if (_controleur.hasClients)` avant |
| `A ScrollController was used after being disposed.` | `dispose()` appelé alors qu'un listener tourne encore | `removeListener(...)` puis `dispose()` dans `dispose()` du `State` |
| `setState() called after dispose()` | `setState` appelé après un `await`, alors que l'écran a été quitté | `if (!mounted) return;` juste après chaque `await` |
| la roue du `RefreshIndicator` tourne indéfiniment | `onRefresh` retourne un `Future` qui ne se termine jamais, ou retourne `void` | `Future<void> _rafraichir() async { ... }` et `onRefresh: _rafraichir` |
| le geste « tirer pour rafraîchir » ne se déclenche pas | la liste est trop courte pour défiler | `physics: const AlwaysScrollableScrollPhysics()` sur la liste |
| `A RenderViewport expected a child of type RenderSliver but received a child of type RenderConstrainedBox.` | un widget ordinaire placé dans `slivers:` | l'envelopper dans `SliverToBoxAdapter(child: ...)` |
| `The argument type 'Iterable<Widget>' can't be assigned to the parameter type 'List<Widget>'.` | `.toList()` oublié après un `map` | ajouter `.toList()` |
| `A value of type 'void' can't be assigned to a variable of type 'List<Ennemi>'.` | `sort` retourne `void` | `final tries = List<Ennemi>.of(source)..sort(...);` |
| le filtre supprime définitivement des éléments | filtrage par `removeWhere` sur la liste source | garder la source intacte et calculer une liste dérivée avec `where(...).toList()` |
| liste saccadée au défilement | widgets non `const`, hauteurs mesurées, images non mises en cache, calcul dans `itemBuilder` | `const`, `itemExtent`/`prototypeItem`, `cached_network_image`, sortir les calculs du builder |
| `Failed assertion: itemExtent == null || prototypeItem == null` | `itemExtent` et `prototypeItem` fournis en même temps | n'en garder qu'un seul |
| deux barres d'application empilées | `Scaffold(appBar: ...)` **et** `SliverAppBar` dans le `body` | supprimer le `appBar:` du `Scaffold` |

---

## 48.37 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| `SingleChildScrollView` + `Column` | pour un contenu **court et fixe** uniquement ; construit tout |
| `ListView(children: ...)` | pour 1 à 30 éléments écrits en dur ; construit tout |
| `ListView.builder` | le choix par défaut : construit **à la demande**, coût indépendant de la taille |
| `itemCount` | **toujours** `liste.length` ; `null` signifie « liste infinie » |
| `itemBuilder` | reçoit `(context, index)`, lit la donnée, retourne un widget, rien d'autre |
| `ListView.separated` | ajoute `separatorBuilder` ; `n` éléments, `n - 1` séparateurs |
| `Divider(height: 1)` | `Divider()` fait 16 pixels de haut par défaut, pas 1 |
| `ListTile` | ligne Material standard : `leading`, `title`, `subtitle`, `trailing`, `onTap` |
| `SwitchListTile` | rend toute la ligne cliquable ; le paramètre gauche s'appelle `secondary` |
| `Card` | contenant surélevé ; `clipBehavior: Clip.antiAlias` pour rogner aux coins |
| Tuile personnalisée | toujours une **classe** `StatelessWidget`, jamais une fonction |
| Liste d'objets | modèle (classe) → données (`List<T>`) → interface (`ListView.builder`) |
| `shrinkWrap: true` | la liste prend la hauteur de son contenu, mais construit **tout** |
| `physics: NeverScrollableScrollPhysics()` | la liste ne capte plus le geste ; à coupler avec `shrinkWrap` |
| `physics: AlwaysScrollableScrollPhysics()` | rend le tirage possible même sur une liste courte |
| Liste dans une `Column` | `Expanded` dans 90 % des cas ; `SizedBox` si la hauteur est connue |
| `ScrollController` | à créer dans le `State` et à `dispose()` obligatoirement |
| `hasClients` | à tester avant toute lecture de `offset` ou `position` |
| Défilement infini | déclencher sur `extentAfter < 300`, protéger par un booléen `_enChargement` |
| `animateTo` | déplacement animé ; retourne un `Future<void>` |
| `GridView.count` | grille simple à nombre de colonnes fixe ; construit tout |
| `GridView.builder` | grille performante ; exige un `gridDelegate` |
| `SliverGridDelegateWithFixedCrossAxisCount` | `crossAxisCount` colonnes fixes |
| `SliverGridDelegateWithMaxCrossAxisExtent` | largeur maximale par case : grille responsive sans condition |
| `childAspectRatio` | **largeur / hauteur** ; `< 1` = case haute, `> 1` = case large |
| `mainAxisExtent` | hauteur de case en pixels, prioritaire sur `childAspectRatio` |
| `Dismissible` | glisser pour supprimer ; `background` et `secondaryBackground` |
| `Key` d'un `Dismissible` | **obligatoire et stable** : `ValueKey(objet.id)`, jamais l'index ni `UniqueKey()` |
| `confirmDismiss` | demande confirmation avant l'animation ; retourne `Future<bool?>` |
| Annulation | garder l'objet et son index, puis `SnackBarAction` qui réinsère |
| `RefreshIndicator` | `onRefresh` doit retourner un `Future<void>` qui se termine |
| `ReorderableListView` | `onReorder` + correction `if (newIndex > oldIndex) newIndex -= 1;` |
| État vide | distinguer « rien du tout » et « rien qui corresponde au filtre » |
| Filtrer / trier | ne jamais modifier la source ; calculer une liste dérivée |
| `sort` | modifie en place et retourne `void` ; trier une copie (`List.of(...)..sort(...)`) |
| `CustomScrollView` | un seul moteur de défilement pour plusieurs zones (slivers) |
| `SliverToBoxAdapter` | transforme un widget ordinaire en sliver |
| `SliverAppBar` | en-tête défilant ; `pinned`, `floating`, `expandedHeight`, `flexibleSpace` |
| Performance | `const`, `itemExtent`/`prototypeItem`, cache d'images, pas de calcul dans le builder |

---

## 48.38 — Exercices

### Exercice 1 — La liste de potions (facile)

Affichez une liste de six potions (`'Potion de soin'`, `'Potion de mana'`, `'Élixir de force'`, `'Antidote'`, `'Potion de vitesse'`, `'Philtre de vie'`) avec `ListView.builder` et `ListTile`.
Chaque ligne doit avoir une icône `Icons.local_drink` en `leading` et son numéro d'ordre (à partir de 1) en `trailing`.

### Exercice 2 — Le grimoire séparé (facile)

Reprenez une liste de cinq sorts. Affichez-la avec `ListView.separated`.
Le séparateur est un `Divider(height: 1)`, **sauf** après le troisième sort où il doit être un `Divider(thickness: 3, color: Colors.deepPurple)`.
Le titre de l'`AppBar` affiche le nombre de sorts.

### Exercice 3 — Les armes en cartes (facile)

Créez une classe `Arme` avec `nom` (`String`), `degats` (`int`) et `portee` (`String`).
Créez une liste de cinq armes, puis affichez-les dans des `Card` contenant un `ListTile` :
`leading` = un `CircleAvatar` avec la première lettre du nom, `title` = le nom, `subtitle` = la portée, `trailing` = `'<dégâts> dgt'`.

### Exercice 4 — Le carrousel de compétences (moyen)

Dans une `Column`, affichez :
1. un titre `'Compétences'` ;
2. une `ListView` **horizontale** de six compétences, chaque case faisant 120 pixels de large ;
3. sous celle-ci, une `ListView` verticale de dix objets qui occupe le reste de l'écran.

Aucune erreur de contrainte ne doit apparaître dans la console.

### Exercice 5 — La grille de sorts (moyen)

Affichez douze sorts dans un `GridView.builder` à **trois** colonnes.
Chaque case contient une icône et le nom du sort, sur fond coloré et coins arrondis.
Réglez `childAspectRatio` pour que les cases soient légèrement plus hautes que larges, sans aucun débordement.

### Exercice 6 — La grille responsive (moyen)

Reprenez l'exercice 5 avec `SliverGridDelegateWithMaxCrossAxisExtent` et `maxCrossAxisExtent: 180`.
Affichez dans l'`AppBar` la largeur de l'écran et le nombre de colonnes calculé par la formule `(largeurDisponible / 180).ceil()`.

### Exercice 7 — Le sac avec annulation (moyen)

Créez une classe `Objet` avec `id` et `nom`. Affichez huit objets.
Un glissement de la droite vers la gauche supprime l'objet, avec un fond rouge et une icône de corbeille.
Un `SnackBar` propose `ANNULER` pendant quatre secondes et réinsère l'objet à sa place d'origine.

### Exercice 8 — La position de défilement (difficile)

Affichez 200 ennemis.
1. L'`AppBar` affiche le pourcentage de progression du défilement, arrondi à l'entier.
2. Un bouton flottant apparaît au-delà de 500 pixels et remonte en haut en 400 ms.
3. Un second bouton descend tout en bas.

`setState` ne doit être appelé que lorsque la valeur affichée change réellement.

### Exercice 9 — Recherche et tri (difficile)

Reprenez la classe `Arme` de l'exercice 3 avec dix armes.
1. Une barre de recherche filtre par nom.
2. Trois `ChoiceChip` trient par nom, par dégâts croissants ou par dégâts décroissants.
3. Quand aucun résultat ne correspond, affichez un état vide avec un bouton « Effacer ».

La liste source ne doit jamais être modifiée.

### Exercice 10 — L'écran de donjon (difficile)

Construisez un écran unique avec un `CustomScrollView` contenant, dans cet ordre :
1. un `SliverAppBar` de 200 pixels de haut, `pinned`, avec un `FlexibleSpaceBar` à dégradé et le titre `'Donjon du gouffre'` ;
2. un `SliverToBoxAdapter` affichant `'Salles explorées : X / 30'` ;
3. un `SliverPadding` contenant un `SliverGrid.builder` de six clés (3 colonnes) ;
4. un `SliverList.builder` de trente salles, chacune supprimable par `Dismissible`.

Le compteur de l'en-tête doit se mettre à jour après chaque suppression.

---

## 48.39 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationPotions());
}

class ApplicationPotions extends StatelessWidget {
  const ApplicationPotions({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranPotions(),
    );
  }
}

class EcranPotions extends StatelessWidget {
  const EcranPotions({super.key});

  static const List<String> potions = <String>[
    'Potion de soin',
    'Potion de mana',
    'Élixir de force',
    'Antidote',
    'Potion de vitesse',
    'Philtre de vie',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Potions')),
      body: ListView.builder(
        itemCount: potions.length,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            leading: const Icon(Icons.local_drink, color: Colors.pink),
            title: Text(potions[index]),
            trailing: Text('${index + 1}'),
          );
        },
      ),
    );
  }
}
```

**Explication :** `itemCount: potions.length` garantit que le nombre de lignes suit toujours la liste. `itemBuilder` reçoit un `index` de `0` à `5` : on affiche donc `index + 1` pour numéroter à partir de 1. L'icône est déclarée `const` car elle est identique sur toutes les lignes : Flutter la construit une seule fois et la réutilise, ce qui évite six allocations inutiles.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationGrimoire());
}

class ApplicationGrimoire extends StatelessWidget {
  const ApplicationGrimoire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranGrimoire(),
    );
  }
}

class EcranGrimoire extends StatelessWidget {
  const EcranGrimoire({super.key});

  static const List<String> sorts = <String>[
    'Boule de feu',
    'Bouclier de glace',
    'Éclair enchaîné',
    'Soin mineur',
    'Invisibilité',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Grimoire (${sorts.length} sorts)')),
      body: ListView.separated(
        itemCount: sorts.length,
        separatorBuilder: (BuildContext context, int index) {
          // index = position de l'élément AU-DESSUS du séparateur.
          // Le 3e sort est à l'index 2.
          if (index == 2) {
            return const Divider(thickness: 3, color: Colors.deepPurple);
          }
          return const Divider(height: 1);
        },
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            leading: const Icon(Icons.auto_awesome),
            title: Text(sorts[index]),
            subtitle: Text('Sort n°${index + 1}'),
          );
        },
      ),
    );
  }
}
```

**Explication :** avec `ListView.separated`, `separatorBuilder` est appelé `itemCount - 1` fois, soit quatre fois ici, avec des index de `0` à `3`. L'index reçu est celui de l'élément **précédant** le séparateur : le trait après le troisième sort correspond donc à `index == 2`. `Divider(height: 1)` évite l'espacement de 16 pixels du `Divider` par défaut.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationArmes());
}

class Arme {
  const Arme({
    required this.nom,
    required this.degats,
    required this.portee,
  });

  final String nom;
  final int degats;
  final String portee;

  String get initiale => nom[0].toUpperCase();
}

const List<Arme> armes = <Arme>[
  Arme(nom: 'Épée longue', degats: 24, portee: 'Corps à corps'),
  Arme(nom: 'Arc court', degats: 15, portee: 'Distance moyenne'),
  Arme(nom: 'Hache de guerre', degats: 32, portee: 'Corps à corps'),
  Arme(nom: 'Dague empoisonnée', degats: 11, portee: 'Corps à corps'),
  Arme(nom: 'Arbalète lourde', degats: 28, portee: 'Longue distance'),
];

class ApplicationArmes extends StatelessWidget {
  const ApplicationArmes({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranArmes(),
    );
  }
}

class EcranArmes extends StatelessWidget {
  const EcranArmes({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Armurerie')),
      body: ListView.builder(
        padding: const EdgeInsets.symmetric(vertical: 8),
        itemCount: armes.length,
        itemBuilder: (BuildContext context, int index) {
          final Arme arme = armes[index];
          return Card(
            margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 5),
            child: ListTile(
              leading: CircleAvatar(
                backgroundColor: Theme.of(context).colorScheme.primaryContainer,
                child: Text(arme.initiale),
              ),
              title: Text(arme.nom),
              subtitle: Text(arme.portee),
              trailing: Text(
                '${arme.degats} dgt',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

**Explication :** la classe `Arme` est un modèle pur : aucune dépendance à Flutter, des champs `final` et un constructeur `const` à paramètres nommés `required` (chapitre 09). Le getter `initiale` déplace la logique d'affichage dans le modèle plutôt que dans le `itemBuilder`. La `Card` est le contenant, le `ListTile` le contenu : jamais l'inverse.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationCompetences());
}

class ApplicationCompetences extends StatelessWidget {
  const ApplicationCompetences({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranCompetences(),
    );
  }
}

class EcranCompetences extends StatelessWidget {
  const EcranCompetences({super.key});

  static const List<String> competences = <String>[
    'Force', 'Agilité', 'Endurance', 'Magie', 'Furtivité', 'Charisme',
  ];

  @override
  Widget build(BuildContext context) {
    final List<String> objets =
        List<String>.generate(10, (int i) => 'Objet n°${i + 1}');

    return Scaffold(
      appBar: AppBar(title: const Text('Personnage')),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          const Padding(
            padding: EdgeInsets.fromLTRB(16, 16, 16, 8),
            child: Text(
              'Compétences',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ),
          // Une ListView horizontale DOIT recevoir une hauteur bornée.
          SizedBox(
            height: 110,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 12),
              itemCount: competences.length,
              itemBuilder: (BuildContext context, int index) {
                return Container(
                  width: 120,
                  margin: const EdgeInsets.symmetric(horizontal: 6),
                  decoration: BoxDecoration(
                    color: Colors.indigo.shade50,
                    borderRadius: BorderRadius.circular(12),
                    border: Border.all(color: Colors.indigo.shade200),
                  ),
                  alignment: Alignment.center,
                  child: Text(
                    competences[index],
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                );
              },
            ),
          ),
          const Divider(),
          const Padding(
            padding: EdgeInsets.fromLTRB(16, 8, 16, 8),
            child: Text(
              'Inventaire',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ),
          // Expanded donne à la liste verticale « tout ce qui reste ».
          Expanded(
            child: ListView.builder(
              itemCount: objets.length,
              itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  leading: const Icon(Icons.inventory_2),
                  title: Text(objets[index]),
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

**Explication :** deux contraintes différentes pour deux listes différentes. La liste horizontale reçoit une hauteur explicite via `SizedBox(height: 110)` : sans elle, `Vertical viewport was given unbounded height.` apparaîtrait, car la `Column` offre une hauteur infinie. La liste verticale, elle, doit occuper l'espace restant : `Expanded` transforme la contrainte infinie de la `Column` en contrainte finie exacte. Aucune des deux listes n'utilise `shrinkWrap`, donc la construction reste à la demande.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationGrilleSorts());
}

class Sort {
  const Sort({required this.nom, required this.icone, required this.couleur});
  final String nom;
  final IconData icone;
  final Color couleur;
}

const List<Sort> sorts = <Sort>[
  Sort(nom: 'Feu', icone: Icons.local_fire_department, couleur: Colors.red),
  Sort(nom: 'Glace', icone: Icons.ac_unit, couleur: Colors.lightBlue),
  Sort(nom: 'Foudre', icone: Icons.bolt, couleur: Colors.amber),
  Sort(nom: 'Soin', icone: Icons.favorite, couleur: Colors.pink),
  Sort(nom: 'Bouclier', icone: Icons.shield, couleur: Colors.blueGrey),
  Sort(nom: 'Poison', icone: Icons.science, couleur: Colors.green),
  Sort(nom: 'Vent', icone: Icons.air, couleur: Colors.teal),
  Sort(nom: 'Terre', icone: Icons.landscape, couleur: Colors.brown),
  Sort(nom: 'Lumière', icone: Icons.wb_sunny, couleur: Colors.orange),
  Sort(nom: 'Ombre', icone: Icons.dark_mode, couleur: Colors.deepPurple),
  Sort(nom: 'Temps', icone: Icons.hourglass_bottom, couleur: Colors.indigo),
  Sort(nom: 'Vie', icone: Icons.eco, couleur: Colors.lightGreen),
];

class ApplicationGrilleSorts extends StatelessWidget {
  const ApplicationGrilleSorts({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranGrilleSorts(),
    );
  }
}

class EcranGrilleSorts extends StatelessWidget {
  const EcranGrilleSorts({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Livre de sorts')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        itemCount: sorts.length,
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 3,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
          childAspectRatio: 0.85,
        ),
        itemBuilder: (BuildContext context, int index) {
          final Sort sort = sorts[index];
          return Container(
            decoration: BoxDecoration(
              color: sort.couleur.withValues(alpha: 0.15),
              borderRadius: BorderRadius.circular(14),
              border: Border.all(color: sort.couleur.withValues(alpha: 0.4)),
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Icon(sort.icone, size: 36, color: sort.couleur),
                const SizedBox(height: 8),
                Text(
                  sort.nom,
                  textAlign: TextAlign.center,
                  maxLines: 1,
                  overflow: TextOverflow.ellipsis,
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

**Explication :** `childAspectRatio: 0.85` signifie « largeur = 0,85 × hauteur », donc une case plus haute que large, ce qui laisse la place à l'icône et au texte empilés. `maxLines: 1` et `overflow: TextOverflow.ellipsis` garantissent qu'un nom long ne provoquera jamais `A RenderFlex overflowed by ... pixels on the bottom.`. Si vous voulez une hauteur strictement fixe indépendante de la largeur de l'écran, remplacez `childAspectRatio` par `mainAxisExtent: 130`.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationGrilleResponsive());
}

class ApplicationGrilleResponsive extends StatelessWidget {
  const ApplicationGrilleResponsive({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranGrilleResponsive(),
    );
  }
}

class EcranGrilleResponsive extends StatelessWidget {
  const EcranGrilleResponsive({super.key});

  static const double extentMax = 180;
  static const double marge = 12;

  @override
  Widget build(BuildContext context) {
    final double largeurEcran = MediaQuery.sizeOf(context).width;
    // La grille dispose de la largeur de l'écran moins le padding.
    final double largeurUtile = largeurEcran - marge * 2;
    final int colonnes = (largeurUtile / extentMax).ceil();

    return Scaffold(
      appBar: AppBar(
        title: Text('${largeurEcran.round()} px — $colonnes colonnes'),
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(marge),
        itemCount: 24,
        gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
          maxCrossAxisExtent: extentMax,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
          childAspectRatio: 0.85,
        ),
        itemBuilder: (BuildContext context, int index) {
          return Container(
            decoration: BoxDecoration(
              color: Colors.primaries[index % Colors.primaries.length]
                  .withValues(alpha: 0.2),
              borderRadius: BorderRadius.circular(14),
            ),
            alignment: Alignment.center,
            child: Text(
              'Sort ${index + 1}',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          );
        },
      ),
    );
  }
}
```

**Explication :** `SliverGridDelegateWithMaxCrossAxisExtent` inverse le raisonnement : on impose une largeur maximale par case et Flutter en déduit le nombre de colonnes avec `(largeurDisponible / maxCrossAxisExtent).ceil()`. Sur un téléphone de 360 pixels, la largeur utile vaut 336 et l'on obtient deux colonnes ; sur une tablette de 1024 pixels, la largeur utile vaut 1000 et l'on obtient six colonnes. Aucune condition `if` sur la taille de l'écran n'est nécessaire. `MediaQuery.sizeOf` est préféré à `MediaQuery.of(context).size` car il ne redéclenche le `build` que si la taille change.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationSac());
}

class Objet {
  const Objet({required this.id, required this.nom});
  final String id;
  final String nom;
}

class ApplicationSac extends StatelessWidget {
  const ApplicationSac({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranSac(),
    );
  }
}

class EcranSac extends StatefulWidget {
  const EcranSac({super.key});

  @override
  State<EcranSac> createState() => _EcranSacState();
}

class _EcranSacState extends State<EcranSac> {
  final List<Objet> _objets = <Objet>[
    const Objet(id: 'o1', nom: 'Épée courte'),
    const Objet(id: 'o2', nom: 'Bouclier de bois'),
    const Objet(id: 'o3', nom: 'Potion de soin'),
    const Objet(id: 'o4', nom: 'Arc long'),
    const Objet(id: 'o5', nom: 'Torche'),
    const Objet(id: 'o6', nom: 'Corde de chanvre'),
    const Objet(id: 'o7', nom: 'Clé rouillée'),
    const Objet(id: 'o8', nom: 'Grimoire ancien'),
  ];

  void _supprimer(Objet objet) {
    final int indexOrigine = _objets.indexOf(objet);
    setState(() => _objets.remove(objet));

    final ScaffoldMessengerState messenger = ScaffoldMessenger.of(context);
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      SnackBar(
        content: Text('${objet.nom} jeté'),
        duration: const Duration(seconds: 4),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () {
            setState(() {
              _objets.insert(indexOrigine.clamp(0, _objets.length), objet);
            });
          },
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Sac (${_objets.length})')),
      body: _objets.isEmpty
          ? const Center(child: Text('Le sac est vide.'))
          : ListView.builder(
              itemCount: _objets.length,
              itemBuilder: (BuildContext context, int index) {
                final Objet objet = _objets[index];
                return Dismissible(
                  key: ValueKey<String>(objet.id),
                  direction: DismissDirection.endToStart,
                  background: Container(
                    color: Colors.red,
                    alignment: Alignment.centerRight,
                    padding: const EdgeInsets.only(right: 24),
                    child: const Icon(Icons.delete, color: Colors.white),
                  ),
                  onDismissed: (DismissDirection direction) =>
                      _supprimer(objet),
                  child: ListTile(
                    leading: const Icon(Icons.inventory_2),
                    title: Text(objet.nom),
                  ),
                );
              },
            ),
    );
  }
}
```

**Explication :** la clé est `ValueKey(objet.id)`, c'est-à-dire un identifiant **stable** qui ne dépend pas de la position. Avec `ValueKey(index)`, la suppression du deuxième élément ferait « glisser » toutes les clés suivantes et Flutter animerait la mauvaise ligne. `_supprimer` reçoit l'objet et non l'index : on calcule `indexOrigine` avant le retrait, ce qui permet de le réinsérer exactement à sa place. Le `.clamp(0, _objets.length)` protège contre le cas où plusieurs suppressions rapides auraient raccourci la liste entre-temps.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationProgression());
}

class ApplicationProgression extends StatelessWidget {
  const ApplicationProgression({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranProgression(),
    );
  }
}

class EcranProgression extends StatefulWidget {
  const EcranProgression({super.key});

  @override
  State<EcranProgression> createState() => _EcranProgressionState();
}

class _EcranProgressionState extends State<EcranProgression> {
  final ScrollController _controleur = ScrollController();

  int _pourcentage = 0;
  bool _boutonsVisibles = false;

  @override
  void initState() {
    super.initState();
    _controleur.addListener(_surDefilement);
  }

  @override
  void dispose() {
    _controleur.removeListener(_surDefilement);
    _controleur.dispose();
    super.dispose();
  }

  void _surDefilement() {
    if (!_controleur.hasClients) {
      return;
    }
    final double maximum = _controleur.position.maxScrollExtent;
    final int pourcentage =
        maximum <= 0 ? 0 : ((_controleur.offset / maximum) * 100).round();
    final bool visibles = _controleur.offset > 500;

    // On ne reconstruit QUE si une valeur affichée a réellement changé.
    if (pourcentage != _pourcentage || visibles != _boutonsVisibles) {
      setState(() {
        _pourcentage = pourcentage.clamp(0, 100);
        _boutonsVisibles = visibles;
      });
    }
  }

  Future<void> _allerA(double cible) {
    return _controleur.animateTo(
      cible,
      duration: const Duration(milliseconds: 400),
      curve: Curves.easeOut,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Bestiaire — $_pourcentage %')),
      body: ListView.builder(
        controller: _controleur,
        itemExtent: 64,
        itemCount: 200,
        itemBuilder: (BuildContext context, int index) {
          return ListTile(
            leading: CircleAvatar(child: Text('${index + 1}')),
            title: Text('Ennemi n°${index + 1}'),
          );
        },
      ),
      floatingActionButton: _boutonsVisibles
          ? Column(
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                FloatingActionButton.small(
                  heroTag: 'haut',
                  tooltip: 'Remonter en haut',
                  onPressed: () => _allerA(0),
                  child: const Icon(Icons.arrow_upward),
                ),
                const SizedBox(height: 12),
                FloatingActionButton.small(
                  heroTag: 'bas',
                  tooltip: 'Descendre en bas',
                  onPressed: () =>
                      _allerA(_controleur.position.maxScrollExtent),
                  child: const Icon(Icons.arrow_downward),
                ),
              ],
            )
          : null,
    );
  }
}
```

**Explication :** trois précautions cohabitent ici. `hasClients` évite l'assertion `ScrollController not attached to any scroll views.` lors du tout premier appel. La comparaison `pourcentage != _pourcentage || visibles != _boutonsVisibles` limite les `setState` à un par pourcent au lieu d'un par pixel, soit une centaine de reconstructions au total plutôt que plusieurs milliers. `itemExtent: 64` indique que toutes les lignes ont la même hauteur : Flutter calcule alors `maxScrollExtent` par multiplication (`200 × 64`), donc le pourcentage est exact dès le premier frame. Enfin, deux `FloatingActionButton` sur le même écran exigent des `heroTag` distincts, sans quoi l'animation de transition lève une exception de tags dupliqués.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationArmurerie());
}

class Arme {
  const Arme({required this.nom, required this.degats, required this.portee});
  final String nom;
  final int degats;
  final String portee;
}

const List<Arme> armurerie = <Arme>[
  Arme(nom: 'Épée longue', degats: 24, portee: 'Corps à corps'),
  Arme(nom: 'Arc court', degats: 15, portee: 'Distance moyenne'),
  Arme(nom: 'Hache de guerre', degats: 32, portee: 'Corps à corps'),
  Arme(nom: 'Dague empoisonnée', degats: 11, portee: 'Corps à corps'),
  Arme(nom: 'Arbalète lourde', degats: 28, portee: 'Longue distance'),
  Arme(nom: 'Marteau de siège', degats: 41, portee: 'Corps à corps'),
  Arme(nom: 'Fronde', degats: 7, portee: 'Distance moyenne'),
  Arme(nom: 'Lance de cavalerie', degats: 26, portee: 'Allonge'),
  Arme(nom: 'Bâton runique', degats: 19, portee: 'Corps à corps'),
  Arme(nom: 'Katana spectral', degats: 35, portee: 'Corps à corps'),
];

enum ModeTri { nom, degatsCroissants, degatsDecroissants }

class ApplicationArmurerie extends StatelessWidget {
  const ApplicationArmurerie({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranArmurerie(),
    );
  }
}

class EcranArmurerie extends StatefulWidget {
  const EcranArmurerie({super.key});

  @override
  State<EcranArmurerie> createState() => _EcranArmurerieState();
}

class _EcranArmurerieState extends State<EcranArmurerie> {
  final TextEditingController _controleurTexte = TextEditingController();
  String _requete = '';
  ModeTri _mode = ModeTri.nom;

  @override
  void dispose() {
    _controleurTexte.dispose();
    super.dispose();
  }

  /// Liste DÉRIVÉE : `armurerie` n'est jamais modifiée.
  List<Arme> get _resultats {
    final String requete = _requete.trim().toLowerCase();

    final List<Arme> filtrees = requete.isEmpty
        ? List<Arme>.of(armurerie)
        : armurerie
            .where((Arme a) => a.nom.toLowerCase().contains(requete))
            .toList();

    switch (_mode) {
      case ModeTri.nom:
        filtrees.sort((Arme a, Arme b) =>
            a.nom.toLowerCase().compareTo(b.nom.toLowerCase()));
      case ModeTri.degatsCroissants:
        filtrees.sort((Arme a, Arme b) => a.degats.compareTo(b.degats));
      case ModeTri.degatsDecroissants:
        filtrees.sort((Arme a, Arme b) => b.degats.compareTo(a.degats));
    }
    return filtrees;
  }

  void _effacer() {
    _controleurTexte.clear();
    setState(() => _requete = '');
  }

  @override
  Widget build(BuildContext context) {
    final List<Arme> resultats = _resultats;

    return Scaffold(
      appBar: AppBar(title: Text('Armurerie (${resultats.length})')),
      body: Column(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(12),
            child: TextField(
              controller: _controleurTexte,
              decoration: InputDecoration(
                hintText: 'Rechercher une arme...',
                prefixIcon: const Icon(Icons.search),
                border: const OutlineInputBorder(),
                isDense: true,
                suffixIcon: _requete.isEmpty
                    ? null
                    : IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: _effacer,
                      ),
              ),
              onChanged: (String valeur) => setState(() => _requete = valeur),
            ),
          ),
          Wrap(
            spacing: 8,
            children: <Widget>[
              ChoiceChip(
                label: const Text('Nom'),
                selected: _mode == ModeTri.nom,
                onSelected: (bool _) => setState(() => _mode = ModeTri.nom),
              ),
              ChoiceChip(
                label: const Text('Dégâts +'),
                selected: _mode == ModeTri.degatsCroissants,
                onSelected: (bool _) =>
                    setState(() => _mode = ModeTri.degatsCroissants),
              ),
              ChoiceChip(
                label: const Text('Dégâts -'),
                selected: _mode == ModeTri.degatsDecroissants,
                onSelected: (bool _) =>
                    setState(() => _mode = ModeTri.degatsDecroissants),
              ),
            ],
          ),
          const SizedBox(height: 8),
          const Divider(height: 1),
          Expanded(
            child: resultats.isEmpty
                ? Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: <Widget>[
                        const Icon(Icons.search_off,
                            size: 64, color: Colors.grey),
                        const SizedBox(height: 12),
                        Text('Aucune arme pour "$_requete"'),
                        const SizedBox(height: 8),
                        FilledButton.tonal(
                          onPressed: _effacer,
                          child: const Text('Effacer'),
                        ),
                      ],
                    ),
                  )
                : ListView.builder(
                    itemCount: resultats.length,
                    itemBuilder: (BuildContext context, int index) {
                      final Arme arme = resultats[index];
                      return ListTile(
                        leading: CircleAvatar(child: Text('${arme.degats}')),
                        title: Text(arme.nom),
                        subtitle: Text(arme.portee),
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

**Explication :** `armurerie` est une constante de niveau supérieur : elle n'est jamais modifiée. Le getter `_resultats` reconstruit à chaque `build` une liste dérivée, d'abord filtrée par `where`, puis triée. Comme `where(...).toList()` produit déjà une nouvelle liste, `sort` peut travailler dessus sans risque ; dans la branche « requête vide », `List<Arme>.of(armurerie)` fabrique explicitement la copie qui manquerait sinon — sans elle, `sort` modifierait la source. Le `switch` sur `enum` utilise la syntaxe sans `break` de Dart 3 (chapitre 11).

---

### Correction 10

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationDonjon());
}

class Salle {
  const Salle({required this.id, required this.nom, required this.danger});
  final String id;
  final String nom;
  final int danger;
}

class ApplicationDonjon extends StatelessWidget {
  const ApplicationDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.deepPurple),
      home: const EcranDonjon(),
    );
  }
}

class EcranDonjon extends StatefulWidget {
  const EcranDonjon({super.key});

  @override
  State<EcranDonjon> createState() => _EcranDonjonState();
}

class _EcranDonjonState extends State<EcranDonjon> {
  static const int totalSalles = 30;

  late List<Salle> _salles = List<Salle>.generate(
    totalSalles,
    (int i) => Salle(
      id: 's$i',
      nom: 'Salle n°${i + 1}',
      danger: (i % 5) + 1,
    ),
  );

  void _supprimer(Salle salle) {
    final int indexOrigine = _salles.indexOf(salle);
    setState(() => _salles.remove(salle));

    final ScaffoldMessengerState messenger = ScaffoldMessenger.of(context);
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      SnackBar(
        content: Text('${salle.nom} explorée'),
        duration: const Duration(seconds: 3),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () => setState(() {
            _salles.insert(indexOrigine.clamp(0, _salles.length), salle);
          }),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    final int explorees = totalSalles - _salles.length;

    return Scaffold(
      body: CustomScrollView(
        slivers: <Widget>[
          // 1. En-tête repliable.
          SliverAppBar(
            expandedHeight: 200,
            pinned: true,
            backgroundColor: Colors.deepPurple,
            foregroundColor: Colors.white,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('Donjon du gouffre'),
              centerTitle: true,
              background: DecoratedBox(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topCenter,
                    end: Alignment.bottomCenter,
                    colors: <Color>[
                      Colors.deepPurple.shade200,
                      Colors.deepPurple.shade900,
                    ],
                  ),
                ),
                child: const Center(
                  child: Icon(Icons.castle, size: 88, color: Colors.white24),
                ),
              ),
            ),
          ),

          // 2. Compteur : un widget ordinaire rendu « sliver ».
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              color: Theme.of(context).colorScheme.surfaceContainerHighest,
              child: Text(
                'Salles explorées : $explorees / $totalSalles',
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),

          // 3. Grille des clés.
          SliverPadding(
            padding: const EdgeInsets.all(12),
            sliver: SliverGrid.builder(
              itemCount: 6,
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                mainAxisSpacing: 10,
                crossAxisSpacing: 10,
                childAspectRatio: 1.4,
              ),
              itemBuilder: (BuildContext context, int index) {
                return Container(
                  decoration: BoxDecoration(
                    color: Colors.amber.shade100,
                    borderRadius: BorderRadius.circular(10),
                  ),
                  alignment: Alignment.center,
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      const Icon(Icons.vpn_key, color: Colors.amber),
                      Text('Clé ${index + 1}'),
                    ],
                  ),
                );
              },
            ),
          ),

          // 4. Liste des salles, supprimables.
          if (_salles.isEmpty)
            const SliverToBoxAdapter(
              child: Padding(
                padding: EdgeInsets.all(48),
                child: Center(
                  child: Text('Donjon entièrement exploré.'),
                ),
              ),
            )
          else
            SliverList.builder(
              itemCount: _salles.length,
              itemBuilder: (BuildContext context, int index) {
                final Salle salle = _salles[index];
                return Dismissible(
                  key: ValueKey<String>(salle.id),
                  direction: DismissDirection.endToStart,
                  background: Container(
                    color: Colors.green,
                    alignment: Alignment.centerRight,
                    padding: const EdgeInsets.only(right: 24),
                    child: const Icon(Icons.check, color: Colors.white),
                  ),
                  onDismissed: (DismissDirection direction) =>
                      _supprimer(salle),
                  child: ListTile(
                    leading: CircleAvatar(child: Text('${salle.danger}')),
                    title: Text(salle.nom),
                    subtitle: Text('Danger : ${salle.danger} / 5'),
                  ),
                );
              },
            ),
        ],
      ),
    );
  }
}
```

**Explication :** un seul `CustomScrollView` fait défiler quatre zones de natures différentes dans un mouvement unique. Le `Scaffold` n'a **pas** de paramètre `appBar:` : la barre est le `SliverAppBar`, sinon deux barres se superposeraient. Le compteur est un widget ordinaire, donc il doit être enveloppé dans `SliverToBoxAdapter` ; sans cela, Flutter lève `A RenderViewport expected a child of type RenderSliver`. Le `SliverPadding` remplace le `Padding` habituel car un `Padding` ordinaire n'est pas un sliver. Enfin, `explorees` est **calculé** à partir de `_salles.length` plutôt que stocké dans une variable d'état : il ne peut donc jamais se désynchroniser de la liste, et il se met à jour automatiquement à chaque `setState`, y compris lors d'une annulation.

---

## Et maintenant ?

Vous savez désormais afficher n'importe quelle collection : en liste, en grille, avec séparateurs, avec suppression par glissement, avec recherche, tri et défilement infini. C'est la moitié de toute interface mobile.

L'autre moitié, c'est la **saisie** : comment l'utilisateur crée et modifie les données que vous affichez. Dans ce chapitre, la barre de recherche a introduit `TextField` et `TextEditingController` sans les expliquer. Le chapitre suivant les reprend depuis le début, et va beaucoup plus loin : les différents boutons Material 3, le clavier et ses types, `Form` et `TextFormField`, les validateurs, les messages d'erreur, `GlobalKey<FormState>` et la soumission d'un formulaire complet.

À la fin du chapitre 49, vous saurez ajouter un ennemi au bestiaire que vous venez de construire.

Rendez-vous au chapitre 49 : [49-PARTIE-1B—BOUTONS-FORMULAIRES-ET-VALIDATION.md](./49-PARTIE-1B—BOUTONS-FORMULAIRES-ET-VALIDATION.md)
