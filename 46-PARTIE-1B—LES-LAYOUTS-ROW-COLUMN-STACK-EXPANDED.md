# PARTIE 1B — FLUTTER
# CHAPITRE 46 — LES LAYOUTS : ROW, COLUMN, STACK ET LES CONTRAINTES

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 10 h
> **Pré-requis :** chapitre 45 — `StatelessWidget` et `StatefulWidget`
> **Ce que vous saurez faire à la fin :** placer n'importe quel widget exactement où vous le voulez à l'écran, comprendre et corriger les erreurs de mise en page de Flutter, et reproduire une maquette complète sans tâtonner.

---

## 46.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- énoncer la règle de mise en page de Flutter en une phrase ;
- expliquer ce qu'est une contrainte et pourquoi elle descend dans l'arbre ;
- utiliser `Container` en connaissant ses pièges ;
- écrire un `EdgeInsets` sous ses quatre formes ;
- décorer une boîte avec `BoxDecoration` : couleur, bordure, coins arrondis, ombre ;
- choisir entre `Padding`, `SizedBox`, `Center` et `Align` ;
- empiler des widgets verticalement avec `Column` ;
- aligner des widgets horizontalement avec `Row` ;
- distinguer axe principal et axe transversal ;
- utiliser les six valeurs de `MainAxisAlignment` ;
- utiliser les valeurs de `CrossAxisAlignment` ;
- régler `mainAxisSize` et savoir pourquoi ;
- lire une erreur `RenderFlex overflowed` et la corriger en trois façons différentes ;
- répartir l'espace avec `Expanded`, `Flexible` et `Spacer` ;
- expliquer la différence exacte entre `Expanded` et `Flexible` ;
- passer à la ligne automatiquement avec `Wrap` ;
- superposer des widgets avec `Stack` et les placer avec `Positioned` ;
- garder plusieurs pages en mémoire avec `IndexedStack` ;
- imposer un rapport largeur/hauteur avec `AspectRatio` ;
- dimensionner en pourcentage avec `FractionallySizedBox` ;
- forcer des contraintes avec `ConstrainedBox` et `BoxConstraints` ;
- comprendre le coût de `IntrinsicHeight` ;
- rendre une page défilante avec `SingleChildScrollView` ;
- éviter l'encoche et la barre système avec `SafeArea` ;
- déboguer visuellement une mise en page ;
- reconstruire une maquette de haut en bas avec une méthode reproductible.

---

## 46.0.1 — Pourquoi ce chapitre est le plus important de la PARTIE 1B

Aux chapitres 44 et 45, vous avez appris **ce qu'est** un widget et **comment il se reconstruit**.

Vous savez donc écrire un `Text`, un `Icon`, un `Scaffold`.

Mais dès que vous voulez écrire une vraie page, vous vous heurtez à une autre question, complètement différente :

```text
« J'ai six widgets. Comment je les place ? »
« Pourquoi mon bouton est collé en haut à gauche ? »
« Pourquoi j'ai une bande jaune et noire en travers de l'écran ? »
« Pourquoi mon Container ne fait pas 200 pixels alors que je l'ai écrit ? »
```

Ce ne sont pas des questions de débutant maladroit. Ce sont **les** questions de Flutter. La mise en page est le sujet sur lequel tout le monde bloque, y compris des développeurs expérimentés venus d'autres technologies.

La raison est simple : Flutter ne fonctionne **pas** comme le CSS du web, ni comme les `LayoutParams` d'Android, ni comme l'Auto Layout d'iOS. Il a son propre modèle, très simple une fois compris, mais qui doit être compris **explicitement**.

Ce chapitre ne se contente donc pas de lister des widgets. Il commence par le modèle, et tout le reste en découle.

> Si vous ne devez retenir qu'une seule section de tout ce chapitre, retenez la section 46.1.

---

## 46.1 — Le modèle de mise en page de Flutter en une phrase

Voici la phrase. Apprenez-la par cœur.

> **Les contraintes descendent. Les tailles remontent. Le parent place l'enfant.**

Trois temps, dans cet ordre, toujours.

**Temps 1 — Les contraintes descendent.**
Chaque widget reçoit de son parent une **contrainte** : « tu as le droit de faire entre telle et telle largeur, et entre telle et telle hauteur ». Le widget ne choisit pas sa contrainte : il la subit.

**Temps 2 — Les tailles remontent.**
Le widget regarde sa contrainte, regarde ce qu'il contient, et **choisit sa taille** à l'intérieur de ce qui est autorisé. Puis il annonce cette taille à son parent.

**Temps 3 — Le parent place l'enfant.**
Le parent connaît maintenant la taille de son enfant. Il décide **où** le poser dans sa propre zone. L'enfant n'a pas son mot à dire sur sa position.

Une conséquence immédiate, et surprenante pour un débutant :

> Un widget ne connaît jamais sa position à l'écran. Il connaît seulement sa taille, et encore : seulement celle qu'on lui a autorisée.

---

## 46.1.1 — Qu'est-ce qu'une contrainte, concrètement ?

Une contrainte, en Flutter, est un objet `BoxConstraints`. Il contient quatre nombres :

```text
BoxConstraints(
  minWidth  : largeur minimale autorisée
  maxWidth  : largeur maximale autorisée
  minHeight : hauteur minimale autorisée
  maxHeight : hauteur maximale autorisée
)
```

C'est tout. Quatre nombres.

Exemple. L'écran d'un téléphone de 400 x 800 logical pixels donne au widget racine :

```text
BoxConstraints(
  minWidth  : 400.0
  maxWidth  : 400.0
  minHeight : 800.0
  maxHeight : 800.0
)
```

Minimum égal maximum : le widget racine n'a **aucun choix**, il fera exactement 400 x 800. On appelle cela une contrainte **serrée** (*tight*).

À l'inverse :

```text
BoxConstraints(
  minWidth  : 0.0
  maxWidth  : 400.0
  minHeight : 0.0
  maxHeight : 800.0
)
```

Ici le widget peut choisir n'importe quelle taille entre 0 x 0 et 400 x 800. On appelle cela une contrainte **lâche** (*loose*).

Et enfin, le cas qui provoque le plus d'erreurs :

```text
BoxConstraints(
  minWidth  : 0.0
  maxWidth  : 400.0
  minHeight : 0.0
  maxHeight : Infinity      <-- hauteur non bornée
)
```

Une contrainte **non bornée** (*unbounded*). Le widget a le droit d'être aussi haut qu'il veut. Cela arrive dans une zone défilante : la hauteur d'un `ListView` n'est pas limitée puisqu'on peut faire défiler. Nous y reviendrons longuement en 46.23 et 46.37.

---

## 46.1.2 — Les trois mots de vocabulaire à retenir

| Mot | Signification | Exemple |
| --- | --- | --- |
| Contrainte **serrée** (*tight*) | `min == max` : la taille est imposée | Le widget racine, un `SizedBox(width: 100, height: 50)` |
| Contrainte **lâche** (*loose*) | `min == 0`, `max` fini : le widget choisit | Ce que `Center` donne à son enfant |
| Contrainte **non bornée** (*unbounded*) | `max == double.infinity` | Un enfant de `ListView`, dans le sens du défilement |

Retenez surtout : **une contrainte lâche autorise à être petit, une contrainte serrée oblige à être grand.**

---

## 46.2 — Le schéma du modèle

Voici le voyage complet d'une passe de layout. Lisez-le de haut en bas, puis de bas en haut.

```text
                    PHASE 1 : LES CONTRAINTES DESCENDENT
                    ────────────────────────────────────

        ┌──────────────────────────────────────────────┐
        │  Écran  (400 x 800)                          │
        │                                              │
        │   « Tu fais exactement 400 x 800 »           │
        │                    │                         │
        │                    v                         │
        │   ┌──────────────────────────────────────┐   │
        │   │  Center                              │   │
        │   │                                      │   │
        │   │  « Tu peux faire de 0x0 à 400x800 »  │   │
        │   │                 │                    │   │
        │   │                 v                    │   │
        │   │   ┌──────────────────────────────┐   │   │
        │   │   │  Container(width: 100,       │   │   │
        │   │   │            height: 50)       │   │   │
        │   │   └──────────────────────────────┘   │   │
        │   └──────────────────────────────────────┘   │
        └──────────────────────────────────────────────┘


                    PHASE 2 : LES TAILLES REMONTENT
                    ───────────────────────────────

        ┌──────────────────────────────────────────────┐
        │  Écran                                       │
        │                    ^                         │
        │        « Je fais 400 x 800 »                 │
        │   ┌────────────────┴─────────────────────┐   │
        │   │  Center                              │   │
        │   │                 ^                    │   │
        │   │     « Je fais 100 x 50 »             │   │
        │   │   ┌─────────────┴────────────────┐   │   │
        │   │   │  Container                   │   │   │
        │   │   │  (choisit 100 x 50 :         │   │   │
        │   │   │   c'est autorisé)            │   │   │
        │   │   └──────────────────────────────┘   │   │
        │   └──────────────────────────────────────┘   │
        └──────────────────────────────────────────────┘


                    PHASE 3 : LE PARENT PLACE
                    ─────────────────────────

        ┌──────────────────────────────────────────────┐
        │  Écran                                       │
        │   ┌──────────────────────────────────────┐   │
        │   │  Center sait que l'enfant fait       │   │
        │   │  100 x 50 et que lui fait 400 x 800. │   │
        │   │  Il calcule :                        │   │
        │   │      x = (400 - 100) / 2 = 150       │   │
        │   │      y = (800 -  50) / 2 = 375       │   │
        │   │                                      │   │
        │   │            ┌───────────┐             │   │
        │   │            │ Container │             │   │
        │   │            └───────────┘             │   │
        │   │                                      │   │
        │   └──────────────────────────────────────┘   │
        └──────────────────────────────────────────────┘
```

Voici le même exemple en code, complet et copiable :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.grey.shade200,
        body: Center(
          child: Container(
            width: 100,
            height: 50,
            color: Colors.blue,
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────┐
│                                │
│                                │
│                                │
│                                │
│           ┌────────┐           │
│           │ bleu   │           │
│           └────────┘           │
│                                │
│                                │
│                                │
└────────────────────────────────┘
   rectangle bleu 100 x 50,
   parfaitement au centre
```

---

## 46.3 — Ce que cela implique concrètement

Le modèle a des conséquences pratiques qui expliquent 90 % des surprises du débutant. Les voici, une par une.

---

### 46.3.1 — Conséquence 1 : un widget ne peut pas dépasser sa contrainte

Écrivez ceci :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Container(
          width: 10000,
          height: 10000,
          color: Colors.red,
        ),
      ),
    );
  }
}
```

Vous demandez 10 000 x 10 000 pixels. Vous obtenez... l'écran entier, et rien de plus.

**Résultat :**

```text
┌────────────────────────────────┐
│                                │
│                                │
│         tout en rouge          │
│      (400 x 800, pas plus)     │
│                                │
│                                │
└────────────────────────────────┘
```

Pourquoi ? Parce que `Scaffold` donne à son `body` une contrainte serrée de la taille de l'écran. `Container` demande 10 000, la contrainte répond « maximum 400 ». Le `Container` fait 400.

> **Règle :** votre `width` et votre `height` sont des **souhaits**, pas des ordres. La contrainte du parent gagne toujours.

---

### 46.3.2 — Conséquence 2 : un widget ne choisit pas sa position

Vous ne trouverez jamais de propriété `x` ou `y` sur un widget Flutter. C'est volontaire.

Pour déplacer un widget, vous ne le déplacez pas : vous **changez son parent**.

| Vous voulez | Vous n'écrivez pas | Vous écrivez |
| --- | --- | --- |
| centrer | `widget.center = true` | `Center(child: widget)` |
| décaler de 16 px | `widget.marginLeft = 16` | `Padding(padding: EdgeInsets.only(left: 16), child: widget)` |
| coller en bas à droite | `widget.position = ...` | `Align(alignment: Alignment.bottomRight, child: widget)` |

C'est la **composition** que vous avez vue au chapitre 44 : en Flutter, on enveloppe.

---

### 46.3.3 — Conséquence 3 : le parent ne connaît pas l'intention de l'enfant

Un widget ne sait pas ce que son parent veut. Il connaît quatre nombres, pas plus.

Conséquence directe : un même widget peut avoir un rendu totalement différent selon son parent.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Exactement le même widget, deux fois.
    const boite = ColoredBox(child: SizedBox.expand(), color: Colors.orange);

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Column(
          children: [
            // Parent 1 : contrainte serrée de 100 de haut.
            SizedBox(height: 100, child: boite),
            const SizedBox(height: 20),
            // Parent 2 : contrainte serrée de 200 de haut.
            SizedBox(height: 200, child: boite),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────┐
│████████████████████████████████│  100 de haut
│████████████████████████████████│
├────────────────────────────────┤  (20 de vide)
│████████████████████████████████│
│████████████████████████████████│  200 de haut
│████████████████████████████████│
│████████████████████████████████│
└────────────────────────────────┘
```

Le widget `boite` est le même. Son rendu dépend entièrement de qui l'enveloppe.

---

### 46.3.4 — Conséquence 4 : parfois l'information manque, et Flutter plante

Si un parent donne une contrainte non bornée en hauteur (« sois aussi haut que tu veux ») et que l'enfant répond « je veux être aussi haut que possible », personne ne peut trancher. Flutter lève alors une erreur du type :

```text
BoxConstraints forces an infinite height.
```

ou :

```text
RenderBox was not laid out: RenderFlex#a1b2c NEEDS-LAYOUT NEEDS-PAINT
```

Ces messages ne sont pas des bugs de Flutter. Ce sont des questions sans réponse. Nous verrons en 46.23 et dans la section « Erreurs fréquentes » comment les lire et les corriger.

---

### 46.3.5 — Résumé du modèle en un tableau

| Question | Réponse |
| --- | --- |
| Qui décide de la contrainte ? | Le parent |
| Qui décide de la taille ? | L'enfant, dans les limites de la contrainte |
| Qui décide de la position ? | Le parent |
| Un enfant peut-il dépasser sa contrainte ? | Non |
| Un enfant connaît-il sa position ? | Non |
| Un parent connaît-il le contenu de l'enfant ? | Non, seulement sa taille finale |

---

## 46.4 — `Container` : le couteau suisse

`Container` est le widget que vous utiliserez le plus au début, et celui que vous abandonnerez progressivement.

Ce n'est pas un widget « de base ». C'est un **assemblage** : Flutter compose pour vous plusieurs widgets plus simples selon les paramètres que vous fournissez.

```text
Container(
  margin:     ...  ->  ajoute un  Padding  (extérieur)
  decoration: ...  ->  ajoute un  DecoratedBox
  width/height ->      ajoute un  ConstrainedBox
  padding:    ...  ->  ajoute un  Padding  (intérieur)
  alignment:  ...  ->  ajoute un  Align
  child:      ...  ->  votre widget
)
```

C'est pour cela qu'on l'appelle « le couteau suisse » : il fait beaucoup de choses, mais chacune existe aussi en version simple et dédiée.

Signature simplifiée des paramètres que nous allons voir :

```dart
Container({
  Key? key,
  AlignmentGeometry? alignment,
  EdgeInsetsGeometry? padding,
  Color? color,
  Decoration? decoration,
  Decoration? foregroundDecoration,
  double? width,
  double? height,
  BoxConstraints? constraints,
  EdgeInsetsGeometry? margin,
  Matrix4? transform,
  AlignmentGeometry? transformAlignment,
  Widget? child,
  Clip clipBehavior = Clip.none,
})
```

> **À savoir tout de suite :** on ne peut PAS fournir `color` **et** `decoration` en même temps. Nous verrons pourquoi en 46.8.

---

## 46.5 — `color`, `width`, `height`

Les trois paramètres les plus utilisés.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('color, width, height')),
        body: Center(
          child: Container(
            width: 200,
            height: 120,
            color: Colors.teal,
            child: const Center(
              child: Text(
                'Boîte',
                style: TextStyle(color: Colors.white, fontSize: 24),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│  color, width, height            │  <- AppBar
├──────────────────────────────────┤
│                                  │
│                                  │
│        ┌────────────────┐        │
│        │                │        │
│        │     Boîte      │        │  200 x 120, vert canard
│        │                │        │
│        └────────────────┘        │
│                                  │
│                                  │
└──────────────────────────────────┘
```

---

### 46.5.1 — Les couleurs en Flutter

Trois façons de désigner une couleur, toutes valides :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // 1. Une constante Material.
              Container(width: 200, height: 60, color: Colors.indigo),
              const SizedBox(height: 12),
              // 2. Une nuance d'une couleur Material (50 -> 900).
              Container(width: 200, height: 60, color: Colors.indigo.shade200),
              const SizedBox(height: 12),
              // 3. Une valeur ARGB explicite : 0xAARRGGBB.
              Container(width: 200, height: 60, color: const Color(0xFF3F51B5)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│                                  │
│      ████████████████████        │  indigo (fonce)
│      ████████████████████        │  indigo.shade200 (clair)
│      ████████████████████        │  0xFF3F51B5 (= indigo)
│                                  │
└──────────────────────────────────┘
```

Décomposition de `0xFF3F51B5` :

```text
   0x  FF   3F   51   B5
       ──   ──   ──   ──
       A    R    G    B
       │    │    │    └── bleu   = 0xB5 = 181
       │    │    └─────── vert   = 0x51 =  81
       │    └──────────── rouge  = 0x3F =  63
       └───────────────── alpha  = 0xFF = 255 (totalement opaque)
```

> `0x00` en alpha = totalement transparent. `0xFF` = totalement opaque. Un oubli fréquent : écrire `Color(0x3F51B5)` sans les deux `FF` du début donne une couleur **invisible**.

Pour rendre une couleur semi-transparente à partir d'une couleur existante :

```dart
Colors.black.withValues(alpha: 0.25)   // noir à 25 %
```

> Historiquement on écrivait `withOpacity(0.25)`. Cette méthode est dépréciée depuis Flutter 3.27 au profit de `withValues(alpha: ...)`. Utilisez la nouvelle forme.

---

### 46.5.2 — Si vous ne donnez ni `width` ni `height`

Un `Container` sans dimension prend la taille de son enfant :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Container(
            color: Colors.amber,
            child: const Text('Juste ce texte', style: TextStyle(fontSize: 28)),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│                                  │
│     ┌──────────────────────┐     │
│     │   Juste ce texte     │     │  la boîte jaune épouse
│     └──────────────────────┘     │  exactement le texte
│                                  │
└──────────────────────────────────┘
```

Et **sans enfant du tout**, il prend toute la place disponible. C'est le piège de la section 46.11.

---

## 46.6 — `padding` et `margin`

Deux mots qui se ressemblent, deux effets opposés.

```text
                     margin (à L'EXTÉRIEUR de la boîte)
       ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
       │                                          │
       │   ┌──────────────────────────────────┐   │
       │   │  ← padding (à L'INTÉRIEUR) →     │   │
       │   │   ┌──────────────────────────┐   │   │
       │   │   │                          │   │   │
       │   │   │        l'enfant          │   │   │
       │   │   │                          │   │   │
       │   │   └──────────────────────────┘   │   │
       │   │                                  │   │
       │   └──────────────────────────────────┘   │
       │       ^                                  │
       └ ─ ─ ─ │─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
               │
        la COULEUR de fond s'arrête ici :
        elle couvre le padding, jamais le margin
```

Points essentiels :

- le **padding** est **dedans** : la couleur de fond le recouvre ;
- le **margin** est **dehors** : la couleur de fond ne le touche pas ;
- le padding **agrandit** la boîte ;
- le margin **éloigne** la boîte de ses voisines.

Exemple qui montre les deux :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.grey.shade300,
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(
                color: Colors.blue,
                padding: const EdgeInsets.all(30),
                child: const Text('padding 30'),
              ),
              Container(
                color: Colors.red,
                margin: const EdgeInsets.all(30),
                child: const Text('margin 30'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  (fond gris)                             │
│                                          │
│        ┌────────────────────────┐        │
│        │████████████████████████│        │
│        │████  padding 30   █████│        │  bleu partout,
│        │████████████████████████│        │  y compris autour du texte
│        └────────────────────────┘        │
│                                          │
│        (30 px de gris : le margin)       │
│                                          │
│             ┌────────────┐               │
│             │ margin 30  │               │  rouge collé au texte
│             └────────────┘               │
│                                          │
│        (30 px de gris : le margin)       │
└──────────────────────────────────────────┘
```

La boîte bleue est grande : le padding l'a agrandie.
La boîte rouge est petite : le margin est en dehors d'elle.

---

### 46.6.1 — Quand utiliser l'un ou l'autre

| Objectif | Utilisez |
| --- | --- |
| Éloigner le texte du bord de sa propre carte colorée | `padding` |
| Espacer deux cartes l'une de l'autre | `margin` (ou un `SizedBox` entre les deux) |
| Agrandir la zone cliquable d'un bouton | `padding` |
| Décaler un bloc par rapport au bord de l'écran | `margin` ou un `Padding` autour |

> En pratique, beaucoup de développeurs Flutter n'utilisent jamais `margin` : ils mettent un `SizedBox` entre les éléments, ou un `Padding` autour. C'est plus lisible, et cela évite d'introduire un `Container` uniquement pour espacer.

---

## 46.7 — `EdgeInsets` : `all`, `symmetric`, `only`, `fromLTRB`

`padding` et `margin` n'acceptent pas un nombre. Ils acceptent un objet `EdgeInsets`, qui décrit **quatre** marges : gauche, haut, droite, bas.

Quatre constructeurs, du plus court au plus précis.

---

### 46.7.1 — `EdgeInsets.all`

La même valeur partout.

```dart
EdgeInsets.all(16)
```

```text
        16
   ┌────────────┐
16 │   contenu  │ 16
   └────────────┘
        16
```

---

### 46.7.2 — `EdgeInsets.symmetric`

Une valeur horizontale, une valeur verticale.

```dart
EdgeInsets.symmetric(horizontal: 24, vertical: 8)
```

```text
         8
   ┌────────────┐
24 │   contenu  │ 24
   └────────────┘
         8
```

C'est de loin le plus utilisé pour un bouton ou un champ de saisie : on veut du souffle sur les côtés, peu en hauteur.

Les deux paramètres valent `0.0` par défaut, on peut donc n'en donner qu'un :

```dart
EdgeInsets.symmetric(horizontal: 24)   // vertical: 0
```

---

### 46.7.3 — `EdgeInsets.only`

Uniquement les côtés que vous nommez. Les autres valent `0.0`.

```dart
EdgeInsets.only(left: 16, top: 8)
```

```text
         8
   ┌────────────┐
16 │   contenu  │ 0
   └────────────┘
         0
```

Les quatre noms sont `left`, `top`, `right`, `bottom`.

---

### 46.7.4 — `EdgeInsets.fromLTRB`

Les quatre valeurs positionnelles, dans l'ordre **L**eft, **T**op, **R**ight, **B**ottom.

```dart
EdgeInsets.fromLTRB(16, 8, 16, 24)
```

```text
         8
   ┌────────────┐
16 │   contenu  │ 16
   └────────────┘
        24
```

> **Piège classique :** l'ordre est L, T, R, B — pas T, R, B, L comme en CSS. Retenez le mot **LTRB**, il est dans le nom de la méthode.

---

### 46.7.5 — Tableau récapitulatif

| Écriture | Gauche | Haut | Droite | Bas |
| --- | --- | --- | --- | --- |
| `EdgeInsets.all(16)` | 16 | 16 | 16 | 16 |
| `EdgeInsets.symmetric(horizontal: 24)` | 24 | 0 | 24 | 0 |
| `EdgeInsets.symmetric(vertical: 8)` | 0 | 8 | 0 | 8 |
| `EdgeInsets.symmetric(horizontal: 24, vertical: 8)` | 24 | 8 | 24 | 8 |
| `EdgeInsets.only(left: 16)` | 16 | 0 | 0 | 0 |
| `EdgeInsets.only(top: 4, bottom: 4)` | 0 | 4 | 0 | 4 |
| `EdgeInsets.fromLTRB(1, 2, 3, 4)` | 1 | 2 | 3 | 4 |
| `EdgeInsets.zero` | 0 | 0 | 0 | 0 |

---

### 46.7.6 — Exemple complet comparant les quatre

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _boite(String titre, EdgeInsets marges, Color couleur) {
    return Container(
      color: couleur,
      padding: marges,
      child: Container(
        color: Colors.white,
        child: Text(titre),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('EdgeInsets')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              _boite('all(20)', const EdgeInsets.all(20), Colors.blue),
              const SizedBox(height: 10),
              _boite(
                'symmetric(h:40, v:5)',
                const EdgeInsets.symmetric(horizontal: 40, vertical: 5),
                Colors.green,
              ),
              const SizedBox(height: 10),
              _boite(
                'only(left: 50)',
                const EdgeInsets.only(left: 50),
                Colors.orange,
              ),
              const SizedBox(height: 10),
              _boite(
                'fromLTRB(5,20,5,40)',
                const EdgeInsets.fromLTRB(5, 20, 5, 40),
                Colors.purple,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────────┐
│  EdgeInsets                                │
├────────────────────────────────────────────┤
│                                            │
│      ┌──────────────────────────┐          │
│      │██████████████████████████│          │
│      │██│      all(20)      │███│          │  bleu épais partout
│      │██████████████████████████│          │
│      └──────────────────────────┘          │
│                                            │
│  ┌────────────────────────────────────┐    │
│  │████│ symmetric(h:40, v:5) │████████│    │  vert large, fin en haut
│  └────────────────────────────────────┘    │
│                                            │
│      ┌──────────────────────────────┐      │
│      │██████████│ only(left: 50)    │      │  orange uniquement à gauche
│      └──────────────────────────────┘      │
│                                            │
│      ┌──────────────────────────┐          │
│      │██████████████████████████│          │
│      │█│  fromLTRB(5,20,5,40) │█│          │  violet : 20 en haut,
│      │██████████████████████████│          │  40 en bas
│      │██████████████████████████│          │
│      └──────────────────────────┘          │
└────────────────────────────────────────────┘
```

---

## 46.8 — `decoration` et `BoxDecoration`

`color` ne sait faire qu'une chose : remplir. Pour tout le reste — bordure, coins arrondis, ombre, dégradé — il faut `decoration`.

```dart
Container(
  decoration: BoxDecoration(
    color: Colors.blue,
    // et bien plus...
  ),
)
```

---

### 46.8.1 — La règle absolue : `color` ou `decoration`, jamais les deux

Ce code plante :

```dart
// NE COMPILE PAS À L'EXÉCUTION
Container(
  color: Colors.blue,
  decoration: BoxDecoration(borderRadius: BorderRadius.circular(8)),
)
```

Message réel :

```text
Cannot provide both a color and a decoration
To provide both, use "decoration: BoxDecoration(color: color)".
```

Le message donne lui-même la solution : la couleur passe **dans** la décoration.

```dart
// CORRECT
Container(
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
  ),
)
```

> **Mémo :** dès que vous ajoutez une `decoration`, déplacez la ligne `color:` à l'intérieur.

---

### 46.8.2 — Les paramètres de `BoxDecoration`

```dart
BoxDecoration({
  Color? color,
  DecorationImage? image,
  BoxBorder? border,
  BorderRadiusGeometry? borderRadius,
  List<BoxShadow>? boxShadow,
  Gradient? gradient,
  BlendMode? backgroundBlendMode,
  BoxShape shape = BoxShape.rectangle,
})
```

Nous utiliserons `color`, `border`, `borderRadius`, `boxShadow`, `gradient` et `shape`. Nous laissons `image` de côté : le chapitre 47 traite les images.

---

### 46.8.3 — Un dégradé

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Container(
            width: 260,
            height: 140,
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(16),
              gradient: const LinearGradient(
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
                colors: [Colors.deepPurple, Colors.pinkAccent],
              ),
            ),
            child: const Center(
              child: Text(
                'Dégradé',
                style: TextStyle(color: Colors.white, fontSize: 26),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│                                    │
│                                    │
│      ╭──────────────────────╮      │
│      │▓▓▓▓▓▓▒▒▒▒▒░░░░░░░░░░░│      │  violet en haut à gauche,
│      │▓▓▓▓▒▒▒ Dégradé ░░░░░░│      │  rose en bas à droite
│      │▓▓▒▒▒▒░░░░░░░░░░░░░░░░│      │
│      ╰──────────────────────╯      │
│                                    │
│                                    │
└────────────────────────────────────┘
```

---

### 46.8.4 — Un cercle avec `shape`

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Container(
            width: 120,
            height: 120,
            decoration: const BoxDecoration(
              color: Colors.orange,
              shape: BoxShape.circle,
            ),
            child: const Center(
              child: Icon(Icons.person, size: 64, color: Colors.white),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│                                    │
│              ╭──────╮              │
│            ╭─┘      └─╮            │
│            │  (icone) │            │  disque orange de 120,
│            ╰─┐      ┌─╯            │  icône blanche au centre
│              ╰──────╯              │
│                                    │
└────────────────────────────────────┘
```

> **Attention :** avec `shape: BoxShape.circle`, on ne peut pas utiliser `borderRadius`. Flutter lève : `A borderRadius can only be given on boxes with rectangular shapes.`

---

## 46.9 — Bordures, coins arrondis, ombres

Les trois décorations que vous écrirez le plus souvent.

---

### 46.9.1 — Les bordures avec `Border`

```dart
// Une bordure identique sur les 4 côtés.
Border.all(color: Colors.black, width: 2)

// Une bordure uniquement en bas.
Border(bottom: BorderSide(color: Colors.grey, width: 1))

// Des bordures différentes selon les côtés.
Border(
  top: BorderSide(color: Colors.red, width: 4),
  left: BorderSide(color: Colors.blue, width: 4),
)
```

Chaque côté est un `BorderSide` :

```dart
BorderSide({
  Color color = const Color(0xFF000000),
  double width = 1.0,
  BorderStyle style = BorderStyle.solid,
  double strokeAlign = BorderSide.strokeAlignInside,
})
```

---

### 46.9.2 — Les coins arrondis avec `BorderRadius`

```dart
// Les 4 coins au même rayon.
BorderRadius.circular(12)

// Uniquement certains coins.
BorderRadius.only(
  topLeft: Radius.circular(20),
  topRight: Radius.circular(20),
)

// Haut arrondi, bas droit.
BorderRadius.vertical(top: Radius.circular(20))

// Gauche arrondie, droite droite.
BorderRadius.horizontal(left: Radius.circular(20))
```

---

### 46.9.3 — Les ombres avec `BoxShadow`

`boxShadow` prend une **liste** : on peut superposer plusieurs ombres.

```dart
BoxShadow({
  Color color = const Color(0xFF000000),
  Offset offset = Offset.zero,   // décalage (dx, dy)
  double blurRadius = 0.0,       // flou
  double spreadRadius = 0.0,     // grossissement
})
```

Schéma des trois nombres :

```text
    offset: Offset(4, 6)          blurRadius: 10        spreadRadius: 3

    ┌────────┐                    ┌────────┐            ┌────────┐
    │ boîte  │                    │ boîte  │            │ boîte  │
    └────────┘                    └────────┘            └────────┘
        └──╌╌╌╌╌╌┐               ░░░░░░░░░░░░          ▒▒▒▒▒▒▒▒▒▒▒▒
           ombre │               ░ ombre floue        ▒ ombre plus ▒
           décalée               ░░░░░░░░░░░░          ▒▒ grande ▒▒▒
           de 4 à droite         (bords adoucis)      (agrandie de 3
           et 6 vers le bas                            avant le flou)
```

---

### 46.9.4 — Exemple complet : une carte

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.grey.shade200,
        body: Center(
          child: Container(
            width: 280,
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(16),
              border: Border.all(color: Colors.grey.shade300, width: 1),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withValues(alpha: 0.15),
                  offset: const Offset(0, 6),
                  blurRadius: 12,
                  spreadRadius: 0,
                ),
              ],
            ),
            child: const Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Épée de flamme',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                SizedBox(height: 8),
                Text('Dégâts : 45'),
                Text('Prix : 150 pièces'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  (fond gris clair)                       │
│                                          │
│      ╭──────────────────────────╮        │
│      │                          │        │
│      │  Épée de flamme          │        │  carte blanche,
│      │                          │        │  coins arrondis 16,
│      │  Dégâts : 45             │        │  bordure grise fine,
│      │  Prix : 150 pièces       │        │  ombre portée en dessous
│      │                          │        │
│      ╰──────────────────────────╯        │
│        ░░░░░░░░░░░░░░░░░░░░░░░░          │  <- l'ombre
│                                          │
└──────────────────────────────────────────┘
```

> **Remarque :** Flutter fournit aussi le widget `Card`, qui produit une carte Material toute prête (élévation, coins, couleur du thème). Nous le verrons au chapitre 48. `Container` + `BoxDecoration` reste utile quand vous voulez un contrôle total.

---

## 46.10 — `alignment`

Le paramètre `alignment` d'un `Container` place **l'enfant à l'intérieur** de la boîte.

Il n'a d'effet que si la boîte est **plus grande** que son enfant. Sinon, il n'y a rien à aligner.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Container(
            width: 250,
            height: 250,
            color: Colors.lightBlue.shade100,
            alignment: Alignment.bottomRight,
            child: Container(width: 80, height: 40, color: Colors.red),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│                                      │
│      ┌────────────────────────┐      │
│      │                        │      │
│      │                        │      │
│      │       (bleu clair)     │      │
│      │                        │      │
│      │                        │      │
│      │              ┌────────┐│      │
│      │              │ rouge  ││      │  <- bottomRight
│      └──────────────┴────────┴┘      │
│                                      │
└──────────────────────────────────────┘
```

Les neuf valeurs prédéfinies de `Alignment` :

```text
   topLeft        topCenter       topRight
      ┌───────────────┬───────────────┐
      │ ●             │             ● │
      │               │               │
      ├───────────────┼───────────────┤
      │ ●          ●  │  ●          ● │   centerLeft
      │            center             │   center
      ├───────────────┼───────────────┤   centerRight
      │               │               │
      │ ●             │             ● │
      └───────────────┴───────────────┘
   bottomLeft   bottomCenter   bottomRight
```

Et vous pouvez inventer la vôtre :

```dart
Alignment(0.0, 0.0)    // == Alignment.center
Alignment(-1.0, -1.0)  // == Alignment.topLeft
Alignment(1.0, 1.0)    // == Alignment.bottomRight
Alignment(0.5, -0.8)   // à droite du centre, très haut
```

Le repère `Alignment` :

```text
                   y = -1
                     ▲
                     │
        (-1,-1) ┌────┼────┐ (1,-1)
                │    │    │
     x = -1 ────┼────●────┼──── x = 1
                │  (0,0)  │
        (-1, 1) └────┼────┘ (1, 1)
                     │
                     ▼
                   y = 1
```

> **Attention :** `y = -1` est le HAUT et `y = 1` est le BAS. C'est l'inverse de l'intuition mathématique, mais c'est cohérent avec toutes les coordonnées écran.

---

## 46.11 — Le piège : `Container` sans contrainte prend toute la place, ou aucune

Voici LE piège du `Container`, celui qui fait perdre des heures.

**La règle réelle :**

- `Container` **sans enfant** et **sans dimensions** prend **tout l'espace disponible** ;
- `Container` **avec un enfant** et **sans dimensions** prend **la taille de l'enfant**.

Le même widget, deux comportements opposés. Démonstration :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Row(
          children: [
            // Moitié gauche : Container SANS enfant.
            Expanded(
              child: Container(color: Colors.red),
            ),
            // Moitié droite : Container AVEC enfant.
            Expanded(
              child: Center(
                child: Container(
                  color: Colors.blue,
                  child: const Text('petit'),
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

**Résultat :**

```text
┌────────────────────┬────────────────────┐
│████████████████████│                    │
│████████████████████│                    │
│████████████████████│                    │
│██████ rouge ███████│      ┌───────┐     │
│████████████████████│      │ petit │     │  bleu minuscule
│████████████████████│      └───────┘     │
│████████████████████│                    │
│████████████████████│                    │
└────────────────────┴────────────────────┘
   sans enfant :          avec enfant :
   prend tout             prend la taille
                          du texte
```

---

### 46.11.1 — Pourquoi ce comportement ?

Ce n'est pas arbitraire. Reprenez le modèle de 46.1.

`Container` construit un `ConstrainedBox` avec les contraintes reçues. Sans enfant, il n'a aucune raison d'être petit : il prend le maximum autorisé. Avec un enfant, il mesure l'enfant et adopte sa taille.

Formulé autrement :

```text
   Container sans width/height :

       a-t-il un enfant ?
              │
      ┌───────┴────────┐
      │                │
     NON              OUI
      │                │
      v                v
   prend le        prend la
   MAXIMUM         taille de
   autorisé        l'enfant
```

---

### 46.11.2 — Le troisième cas : la contrainte est non bornée

Il existe un troisième cas, plus rare mais très déroutant. Si la contrainte reçue est **non bornée** et que le `Container` n'a pas d'enfant, il ne peut pas prendre « tout l'espace » puisque l'espace est infini. Il prend alors **zéro**.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Column(
          children: [
            const Text('Avant'),
            // Ici la hauteur disponible est bornée : le Container
            // ne prend PAS toute la place car Column mesure les
            // enfants un par un, et le Container sans enfant
            // reçoit une hauteur non bornée -> il fait 0.
            Container(color: Colors.red),
            const Text('Après'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────┐
│              Avant                 │
│              Après                 │  <- le rouge est INVISIBLE
│                                    │     (hauteur 0)
│                                    │
└────────────────────────────────────┘
```

Le débutant conclut « `Container` ne marche pas ». En réalité, le `Container` a fait exactement ce qu'on lui a demandé : dans une `Column`, la hauteur disponible pour chaque enfant est non bornée, donc un `Container` vide vaut 0 de haut.

**La correction** consiste à donner une contrainte explicite :

```dart
Container(color: Colors.red, height: 50)
```

ou à l'envelopper dans un `Expanded` (46.24) pour qu'il prenne le reste de la place.

---

### 46.11.3 — Les trois corrections possibles

| Situation | Correction |
| --- | --- |
| Je veux une taille fixe | `Container(width: ..., height: ...)` |
| Je veux le reste de la place dans un `Row`/`Column` | `Expanded(child: Container(...))` |
| Je veux la taille du contenu | Donner un `child` |

---

## 46.12 — `Padding`

`Padding` fait une seule chose : ajouter de l'espace autour de son enfant.

```dart
Padding({
  Key? key,
  required EdgeInsetsGeometry padding,
  Widget? child,
})
```

C'est exactement le `padding` du `Container`, mais isolé.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Padding')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Container(
            color: Colors.green,
            child: const Center(child: Text('24 px de marge partout')),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│  Padding                             │
├──────────────────────────────────────┤
│                                      │  <- 24 px
│   ┌──────────────────────────────┐   │
│   │██████████████████████████████│   │
│   │████ 24 px de marge partout ██│   │  le vert commence
│   │██████████████████████████████│   │  à 24 px des bords
│   │██████████████████████████████│   │
│   └──────────────────────────────┘   │
│                                      │  <- 24 px
└──────────────────────────────────────┘
```

---

### 46.12.1 — `Padding` ou `Container(padding: ...)` ?

Les deux donnent le même résultat visuel. Alors pourquoi choisir ?

| Critère | `Padding` | `Container(padding:)` |
| --- | --- | --- |
| Lisibilité de l'intention | Explicite : « j'espace » | Ambigu : que fait ce Container ? |
| Coût | Un seul widget | Plusieurs widgets composés |
| `const` possible | Oui, souvent | Plus rarement |

> **Règle pratique :** si vous n'utilisez d'un `Container` que son `padding`, remplacez-le par un `Padding`. Votre code sera plus court et plus clair.

---

### 46.12.2 — Comment `Padding` transmet la contrainte

C'est important pour la suite. `Padding` **réduit** la contrainte avant de la transmettre.

```text
   Contrainte reçue par Padding :        0 .. 400 de large
   padding: EdgeInsets.all(24)           -> 24 à gauche + 24 à droite = 48

   Contrainte transmise à l'enfant :     0 .. 352 de large
                                              ^^^
                                          400 - 48
```

Puis, quand la taille remonte :

```text
   L'enfant répond : « je fais 200 de large »
   Padding répond à son parent : « je fais 200 + 48 = 248 de large »
```

Un `Padding` s'agrandit donc de la valeur du padding par rapport à son enfant.

---

## 46.13 — `SizedBox`

`SizedBox` impose une taille. C'est le widget le plus simple et l'un des plus utilisés.

```dart
SizedBox({
  Key? key,
  double? width,
  double? height,
  Widget? child,
})
```

Trois usages très différents selon les paramètres.

---

### 46.13.1 — Usage 1 : créer un espace vide

Sans `child`, `SizedBox` est un trou d'une taille donnée. C'est l'usage n° 1 en pratique.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(width: 200, height: 40, color: Colors.red),
              const SizedBox(height: 40),   // <- espace vertical
              Container(width: 200, height: 40, color: Colors.blue),
              const SizedBox(height: 8),    // <- petit espace
              Container(width: 200, height: 40, color: Colors.green),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│                                      │
│         ████████████████████         │  rouge
│                                      │
│                                      │  40 px de vide
│                                      │
│         ████████████████████         │  bleu
│                                      │  8 px de vide
│         ████████████████████         │  vert
│                                      │
└──────────────────────────────────────┘
```

> Dans une `Column`, seul `height` compte pour espacer. Dans une `Row`, seul `width` compte. Se tromper est l'erreur la plus fréquente du chapitre : un `SizedBox(height: 20)` dans une `Row` ne fait rien de visible.

---

### 46.13.2 — Usage 2 : forcer la taille d'un enfant

Avec un `child`, `SizedBox` impose une contrainte **serrée** à son enfant.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: SizedBox(
            width: 250,
            height: 60,
            child: ElevatedButton(
              onPressed: () {},
              child: const Text('Bouton large'),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│                                      │
│      ╭──────────────────────────╮    │
│      │      Bouton large        │    │  250 x 60 exactement
│      ╰──────────────────────────╯    │
│                                      │
└──────────────────────────────────────┘
```

Sans le `SizedBox`, le bouton aurait la taille de son texte.

---

### 46.13.3 — Usage 3 : ne fixer qu'une dimension

`width` et `height` sont indépendants. On peut n'en donner qu'un.

```dart
SizedBox(
  height: 200,          // hauteur imposée
  child: monWidget,     // largeur : l'enfant décide
)
```

Schéma :

```text
   SizedBox(height: 200)

   contrainte reçue :   largeur 0..400,  hauteur 0..800
   contrainte donnée :  largeur 0..400,  hauteur 200..200
                                                  ^^^^^^^
                                                  serrée
```

---

## 46.14 — `SizedBox.shrink()` et `SizedBox.expand()`

Deux constructeurs nommés, très pratiques.

---

### 46.14.1 — `SizedBox.shrink()`

Un `SizedBox` de taille **zéro**. Équivalent à `SizedBox(width: 0, height: 0)`.

Usage typique : « afficher un widget, ou rien ».

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatefulWidget {
  const MonApp({super.key});

  @override
  State<MonApp> createState() => _MonAppState();
}

class _MonAppState extends State<MonApp> {
  bool _afficherAlerte = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('SizedBox.shrink')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Si false : rien du tout, hauteur 0.
              _afficherAlerte
                  ? Container(
                      padding: const EdgeInsets.all(16),
                      color: Colors.red.shade100,
                      child: const Text('Attention : vie faible !'),
                    )
                  : const SizedBox.shrink(),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {
                  setState(() => _afficherAlerte = !_afficherAlerte);
                },
                child: const Text('Basculer l\'alerte'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat (alerte masquée) :**

```text
┌──────────────────────────────────────┐
│  SizedBox.shrink                     │
├──────────────────────────────────────┤
│                                      │
│        ╭────────────────────╮        │
│        │ Basculer l'alerte  │        │
│        ╰────────────────────╯        │
│                                      │
└──────────────────────────────────────┘
```

**Résultat (alerte affichée) :**

```text
┌──────────────────────────────────────┐
│  SizedBox.shrink                     │
├──────────────────────────────────────┤
│   ┌──────────────────────────────┐   │
│   │  Attention : vie faible !    │   │
│   └──────────────────────────────┘   │
│                                      │
│        ╭────────────────────╮        │
│        │ Basculer l'alerte  │        │
│        ╰────────────────────╯        │
└──────────────────────────────────────┘
```

> **Nuance :** `SizedBox.shrink()` avec un enfant ne le supprime pas, il le contraint à 0 x 0 (l'enfant peut alors déborder). Pour « ne rien afficher », utilisez-le **sans** enfant.

---

### 46.14.2 — `SizedBox.expand()`

Un `SizedBox` qui prend **tout l'espace disponible**. Équivalent à `SizedBox(width: double.infinity, height: double.infinity)`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: SizedBox.expand(
          child: ColoredBox(
            color: Colors.deepOrange.shade200,
            child: const Center(child: Text('Je remplis tout')),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│██████████████████████████████████████│
│██████████████████████████████████████│
│█████████ Je remplis tout ████████████│
│██████████████████████████████████████│
│██████████████████████████████████████│
└──────────────────────────────────────┘
```

> `double.infinity` en `width` ou `height` signifie « le maximum autorisé par la contrainte », pas « l'infini ». Si la contrainte est non bornée dans cette direction, en revanche, vous obtenez l'erreur `BoxConstraints forces an infinite height.`

---

### 46.14.3 — Les autres constructeurs

| Constructeur | Effet |
| --- | --- |
| `SizedBox(width: w, height: h)` | Taille explicite |
| `SizedBox.square(dimension: 40)` | 40 x 40 |
| `SizedBox.shrink()` | 0 x 0 |
| `SizedBox.expand()` | Le maximum autorisé |
| `SizedBox.fromSize(size: Size(30, 50))` | Depuis un objet `Size` |

---

## 46.15 — `Center`

`Center` centre son enfant, horizontalement et verticalement.

```dart
Center({
  Key? key,
  double? widthFactor,
  double? heightFactor,
  Widget? child,
})
```

C'est en réalité un raccourci pour `Align(alignment: Alignment.center)`.

---

### 46.15.1 — Le comportement par défaut

Sans `widthFactor` ni `heightFactor`, `Center` prend **tout l'espace disponible** et place l'enfant au milieu.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Container(
          color: Colors.yellow.shade200,
          child: Center(
            child: Container(
              width: 100,
              height: 100,
              color: Colors.brown,
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│  jaune = le Center
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│  (il prend tout)
│░░░░░░░░░░░░░████████░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░████████░░░░░░░░░░░░░░░░░│  marron = l'enfant
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
└──────────────────────────────────────┘
```

---

### 46.15.2 — `widthFactor` et `heightFactor`

Ces deux paramètres changent tout : au lieu de prendre tout l'espace, `Center` prend un **multiple de la taille de son enfant**.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Container(
            color: Colors.yellow,
            child: Center(
              widthFactor: 2.0,    // 2 fois la largeur de l'enfant
              heightFactor: 1.5,   // 1,5 fois la hauteur de l'enfant
              child: Container(
                width: 100,
                height: 100,
                color: Colors.brown,
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│                                      │
│      ┌──────────────────────┐        │
│      │░░░░░░░░░░░░░░░░░░░░░░│        │  jaune : 200 x 150
│      │░░░░░░████████░░░░░░░░│        │  (2 x 100 et 1,5 x 100)
│      │░░░░░░████████░░░░░░░░│        │
│      │░░░░░░░░░░░░░░░░░░░░░░│        │  marron : 100 x 100
│      └──────────────────────┘        │
│                                      │
└──────────────────────────────────────┘
```

C'est une manière élégante d'obtenir un espacement proportionnel au contenu.

---

## 46.16 — `Align` et `Alignment`

`Align` est la version générale de `Center`.

```dart
Align({
  Key? key,
  AlignmentGeometry alignment = Alignment.center,
  double? widthFactor,
  double? heightFactor,
  Widget? child,
})
```

Neuf positions prêtes à l'emploi, plus n'importe quelle position sur mesure.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Align')),
        body: Container(
          color: Colors.grey.shade200,
          child: Align(
            alignment: Alignment.topRight,
            child: Container(
              width: 90,
              height: 90,
              color: Colors.purple,
              alignment: Alignment.center,
              child: const Text(
                'TR',
                style: TextStyle(color: Colors.white, fontSize: 22),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│  Align                               │
├──────────────────────────────────────┤
│                          ┌─────────┐ │
│                          │   TR    │ │  <- collé en haut à droite
│                          └─────────┘ │
│                                      │
│                                      │
│                                      │
└──────────────────────────────────────┘
```

---

### 46.16.1 — Les neuf constantes

| Constante | Position | Coordonnées |
| --- | --- | --- |
| `Alignment.topLeft` | haut gauche | `(-1, -1)` |
| `Alignment.topCenter` | haut centre | `(0, -1)` |
| `Alignment.topRight` | haut droite | `(1, -1)` |
| `Alignment.centerLeft` | milieu gauche | `(-1, 0)` |
| `Alignment.center` | centre | `(0, 0)` |
| `Alignment.centerRight` | milieu droite | `(1, 0)` |
| `Alignment.bottomLeft` | bas gauche | `(-1, 1)` |
| `Alignment.bottomCenter` | bas centre | `(0, 1)` |
| `Alignment.bottomRight` | bas droite | `(1, 1)` |

---

### 46.16.2 — `Alignment` sur mesure

Rien n'oblige à rester dans les neuf constantes, ni même dans l'intervalle `[-1, 1]`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Container(
          color: Colors.blueGrey.shade100,
          child: Stack(
            children: [
              _point(const Alignment(-0.5, -0.5), 'A'),
              _point(const Alignment(0.0, 0.0), 'B'),
              _point(const Alignment(0.8, 0.3), 'C'),
              _point(const Alignment(0.0, 0.9), 'D'),
            ],
          ),
        ),
      ),
    );
  }

  Widget _point(Alignment a, String nom) {
    return Align(
      alignment: a,
      child: Container(
        width: 40,
        height: 40,
        decoration: const BoxDecoration(
          color: Colors.deepPurple,
          shape: BoxShape.circle,
        ),
        alignment: Alignment.center,
        child: Text(nom, style: const TextStyle(color: Colors.white)),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│                                      │
│           (A)                        │  (-0.5, -0.5)
│                                      │
│                  (B)                 │  ( 0.0,  0.0) = centre
│                             (C)      │  ( 0.8,  0.3)
│                                      │
│                  (D)                 │  ( 0.0,  0.9)
└──────────────────────────────────────┘
```

> `AlignmentDirectional` existe aussi : il utilise `start`/`end` au lieu de `left`/`right`, ce qui s'adapte automatiquement aux langues écrites de droite à gauche (arabe, hébreu). Pour une application francophone, `Alignment` suffit.

---

## 46.17 — `Column`

`Column` empile ses enfants **verticalement**, de haut en bas.

```dart
Column({
  Key? key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline? textBaseline,
  List<Widget> children = const <Widget>[],
})
```

Le premier exemple :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Column')),
        body: Column(
          children: [
            Container(height: 80, color: Colors.red),
            Container(height: 80, color: Colors.green),
            Container(height: 80, color: Colors.blue),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│  Column                              │
├──────────────────────────────────────┤
│██████████████████████████████████████│  rouge, 80 de haut
│██████████████████████████████████████│
│██████████████████████████████████████│  vert, 80 de haut
│██████████████████████████████████████│
│██████████████████████████████████████│  bleu, 80 de haut
│██████████████████████████████████████│
│                                      │
│         (le reste est vide)          │
│                                      │
└──────────────────────────────────────┘
```

Remarquez : les `Container` n'ont pas de `width`, et ils occupent pourtant toute la largeur. Pourquoi ? Parce que `crossAxisAlignment` vaut `center` par défaut... mais un `Container` sans enfant prend le maximum autorisé, et sur l'axe transversal la contrainte est bornée par la largeur de l'écran. Le `Container` prend donc toute la largeur.

---

### 46.17.1 — L'ordre compte

Les enfants sont posés dans l'ordre du tableau `children`, de haut en bas.

```text
   children: [ A, B, C ]

   ┌───────┐
   │   A   │   ← premier élément du tableau
   ├───────┤
   │   B   │
   ├───────┤
   │   C   │   ← dernier élément
   └───────┘
```

Pour inverser sans toucher au tableau :

```dart
Column(
  verticalDirection: VerticalDirection.up,
  children: [A, B, C],   // affichés C, B, A de haut en bas
)
```

---

## 46.18 — `Row`

`Row` aligne ses enfants **horizontalement**, de gauche à droite. Ses paramètres sont les mêmes que `Column`.

```dart
Row({
  Key? key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline? textBaseline,
  List<Widget> children = const <Widget>[],
})
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Row')),
        body: Row(
          children: [
            Container(width: 80, color: Colors.red),
            Container(width: 80, color: Colors.green),
            Container(width: 80, color: Colors.blue),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────┐
│  Row                                 │
├──────────────────────────────────────┤
│████│████│████│                       │
│████│████│████│                       │
│████│████│████│    (vide)             │
│████│████│████│                       │
│████│████│████│                       │
│ R    V    B                          │
└──────────────────────────────────────┘
  80   80   80
```

Symétriquement au cas de `Column` : les `Container` prennent toute la hauteur parce que la hauteur (axe transversal ici) est bornée.

---

### 46.18.1 — `Row` et `Column` sont la même classe

Techniquement, `Row` et `Column` héritent tous les deux de `Flex`. Un `Flex` est une boîte qui distribue ses enfants sur un axe :

```dart
Flex(direction: Axis.horizontal, children: [...])  // == Row
Flex(direction: Axis.vertical, children: [...])    // == Column
```

Retenez le mot **Flex** : il apparaît dans les messages d'erreur (`RenderFlex overflowed`), et il désigne aussi bien un `Row` qu'une `Column`.

---

## 46.19 — `mainAxisAlignment` (les six valeurs)

`mainAxisAlignment` décide **comment répartir les enfants le long de l'axe principal**, quand il reste de la place.

Pour une `Row`, l'axe principal est horizontal. Nous prendrons donc une `Row` avec trois carrés, et nous ferons varier la valeur.

Voici le code de base, identique pour les six cas :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  // Changez cette valeur pour observer les six comportements.
  static const MainAxisAlignment alignement = MainAxisAlignment.start;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('mainAxisAlignment')),
        body: Container(
          color: Colors.grey.shade300,
          height: 120,
          child: Row(
            mainAxisAlignment: alignement,
            children: [
              Container(width: 60, height: 60, color: Colors.red),
              Container(width: 60, height: 60, color: Colors.green),
              Container(width: 60, height: 60, color: Colors.blue),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### 46.19.1 — `MainAxisAlignment.start` (valeur par défaut)

Tout au début. Le vide est à la fin.

```text
┌────────────────────────────────────────────┐
│ ███ ███ ███                                │
│ ███ ███ ███                                │
│  R   V   B                                 │
└────────────────────────────────────────────┘
  ^^^^^^^^^^^^                ^^^^^^^^^^^^^^
  les 3 carrés                tout le vide
  collés à gauche
```

---

### 46.19.2 — `MainAxisAlignment.end`

Tout à la fin. Le vide est au début.

```text
┌────────────────────────────────────────────┐
│                                ███ ███ ███ │
│                                ███ ███ ███ │
│                                 R   V   B  │
└────────────────────────────────────────────┘
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  tout le vide
```

---

### 46.19.3 — `MainAxisAlignment.center`

Groupés au milieu. Le vide est réparti à parts égales aux deux extrémités.

```text
┌────────────────────────────────────────────┐
│               ███ ███ ███                  │
│               ███ ███ ███                  │
│                R   V   B                   │
└────────────────────────────────────────────┘
  ^^^^^^^^^^^^^^^           ^^^^^^^^^^^^^^^^
   moitié du vide            moitié du vide
```

---

### 46.19.4 — `MainAxisAlignment.spaceBetween`

Le premier collé au début, le dernier collé à la fin, le vide réparti **entre** les éléments.

```text
┌────────────────────────────────────────────┐
│ ███              ███              ███      │
│ ███              ███              ███      │
│  R                V                B       │
└────────────────────────────────────────────┘
      ^^^^^^^^^^^^     ^^^^^^^^^^^^
        espace 1         espace 2
   (les 2 espaces sont égaux, rien aux extrémités)
```

Avec 3 enfants, il y a 2 intervalles. Le vide est divisé par 2.

> C'est la valeur idéale pour une barre « titre à gauche, bouton à droite ».

---

### 46.19.5 — `MainAxisAlignment.spaceAround`

Chaque enfant reçoit la même quantité d'espace **autour de lui**. Conséquence : les espaces aux extrémités valent la **moitié** des espaces intérieurs.

```text
┌────────────────────────────────────────────┐
│     ███         ███         ███            │
│     ███         ███         ███            │
│      R           V           B             │
└────────────────────────────────────────────┘
  ^^^^     ^^^^^^^     ^^^^^^^     ^^^^
   1u        2u          2u         1u
```

Calcul : le vide est divisé par 3 (le nombre d'enfants), puis chaque part est coupée en deux, une moitié devant, une moitié derrière. Les moitiés du milieu se collent et forment un espace double.

---

### 46.19.6 — `MainAxisAlignment.spaceEvenly`

Tous les espaces sont **égaux**, y compris aux extrémités.

```text
┌────────────────────────────────────────────┐
│      ███       ███       ███               │
│      ███       ███       ███               │
│       R         V         B                │
└────────────────────────────────────────────┘
  ^^^^^     ^^^^^     ^^^^^     ^^^^^
    1u        1u        1u        1u
```

Calcul : le vide est divisé par 4 (nombre d'enfants + 1).

---

### 46.19.7 — Les trois « space » côte à côte

C'est la confusion la plus courante. Voici les trois superposés, avec 3 enfants :

```text
spaceBetween   │███              ███              ███│
                 ^ rien aux extrémités

spaceAround    │     ███         ███         ███     │
                 ^^^^ demi-espace aux extrémités

spaceEvenly    │      ███       ███       ███        │
                 ^^^^^^ espace plein aux extrémités
```

| Valeur | Espace aux extrémités | Espace entre |
| --- | --- | --- |
| `spaceBetween` | 0 | vide / (n - 1) |
| `spaceAround` | (vide / n) / 2 | vide / n |
| `spaceEvenly` | vide / (n + 1) | vide / (n + 1) |

---

### 46.19.8 — Exemple pratique : une barre d'en-tête

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: SafeArea(
          child: Column(
            children: [
              Container(
                padding: const EdgeInsets.symmetric(
                  horizontal: 16,
                  vertical: 12,
                ),
                color: Colors.indigo,
                child: const Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      'Inventaire',
                      style: TextStyle(color: Colors.white, fontSize: 20),
                    ),
                    Icon(Icons.settings, color: Colors.white),
                  ],
                ),
              ),
              Container(
                padding: const EdgeInsets.symmetric(vertical: 16),
                color: Colors.indigo.shade50,
                width: double.infinity,
                child: const Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    Icon(Icons.shield, size: 32),
                    Icon(Icons.local_fire_department, size: 32),
                    Icon(Icons.bolt, size: 32),
                    Icon(Icons.favorite, size: 32),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Inventaire                     (icone) │  spaceBetween
├──────────────────────────────────────────┤
│   (icone)   (icone)   (icone)   (icone) │  spaceEvenly
├──────────────────────────────────────────┤
│                                          │
│                                          │
└──────────────────────────────────────────┘
```

---

## 46.20 — `crossAxisAlignment`

`crossAxisAlignment` décide de l'alignement sur l'axe **transversal** (perpendiculaire à l'axe principal).

Pour une `Row` : l'axe transversal est **vertical**.
Pour une `Column` : l'axe transversal est **horizontal**.

Cinq valeurs.

Code de base, avec trois carrés de hauteurs différentes :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  // Faites varier cette valeur.
  static const CrossAxisAlignment alignement = CrossAxisAlignment.center;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('crossAxisAlignment')),
        body: Container(
          height: 200,
          color: Colors.grey.shade300,
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            crossAxisAlignment: alignement,
            children: [
              Container(width: 60, height: 40, color: Colors.red),
              Container(width: 60, height: 90, color: Colors.green),
              Container(width: 60, height: 140, color: Colors.blue),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### 46.20.1 — `CrossAxisAlignment.center` (valeur par défaut)

Chaque enfant est centré verticalement.

```text
┌──────────────────────────────────────────┐
│                             ███          │
│                ███          ███          │
│     ███        ███          ███          │  200 de haut
│     ███        ███          ███          │
│                ███          ███          │
│                             ███          │
└──────────────────────────────────────────┘
      40         90          140
   (tous centrés sur la même ligne médiane)
```

---

### 46.20.2 — `CrossAxisAlignment.start`

Chaque enfant est collé en haut (début de l'axe transversal).

```text
┌──────────────────────────────────────────┐
│     ███        ███          ███          │  <- tous alignés en haut
│     ███        ███          ███          │
│                ███          ███          │
│                ███          ███          │
│                             ███          │
│                             ███          │
└──────────────────────────────────────────┘
```

---

### 46.20.3 — `CrossAxisAlignment.end`

Chaque enfant est collé en bas.

```text
┌──────────────────────────────────────────┐
│                             ███          │
│                             ███          │
│                ███          ███          │
│                ███          ███          │
│     ███        ███          ███          │
│     ███        ███          ███          │  <- tous alignés en bas
└──────────────────────────────────────────┘
```

---

### 46.20.4 — `CrossAxisAlignment.stretch`

Chaque enfant est **étiré** sur toute la hauteur. Les `height` des enfants sont ignorées : la contrainte transversale devient serrée.

```text
┌──────────────────────────────────────────┐
│     ███        ███          ███          │
│     ███        ███          ███          │
│     ███        ███          ███          │  tous étirés
│     ███        ███          ███          │  sur 200
│     ███        ███          ███          │
│     ███        ███          ███          │
└──────────────────────────────────────────┘
```

> `stretch` est très utile dans une `Column` pour que tous les boutons aient la même largeur, sans écrire de `width`.

---

### 46.20.5 — `CrossAxisAlignment.baseline`

Aligne les enfants sur la **ligne de base du texte**. Ne fonctionne qu'avec des enfants contenant du texte, et **exige** le paramètre `textBaseline`.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.baseline,
            textBaseline: TextBaseline.alphabetic,   // OBLIGATOIRE
            mainAxisAlignment: MainAxisAlignment.center,
            children: const [
              Text('42', style: TextStyle(fontSize: 48)),
              SizedBox(width: 6),
              Text('points', style: TextStyle(fontSize: 18)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│                                          │
│                                          │
│           ██  ██                         │
│           ██  ██                         │
│           ██  ██  points                 │
│         ──────────────────  <- ligne de base commune
│                                          │
└──────────────────────────────────────────┘
```

Sans `textBaseline`, Flutter lève :

```text
'package:flutter/src/rendering/flex.dart': Failed assertion:
'textBaseline != null': is not true.
```

---

### 46.20.6 — Tableau récapitulatif

| Valeur | Effet sur l'axe transversal | Ignore la taille de l'enfant ? |
| --- | --- | --- |
| `start` | collé au début | non |
| `end` | collé à la fin | non |
| `center` | centré | non |
| `stretch` | étiré au maximum | **oui** |
| `baseline` | aligné sur la base du texte | non (exige `textBaseline`) |

---

## 46.21 — `mainAxisSize`

`mainAxisSize` répond à une question différente : **quelle taille prend le `Row` ou la `Column` lui-même** sur l'axe principal ?

Deux valeurs seulement.

| Valeur | Signification |
| --- | --- |
| `MainAxisSize.max` (défaut) | Prend tout l'espace disponible sur l'axe principal |
| `MainAxisSize.min` | Prend juste la place nécessaire à ses enfants |

---

### 46.21.1 — Démonstration

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            // max : le Row (fond jaune) prend toute la largeur.
            ColoredBox(
              color: Colors.yellow,
              child: Row(
                mainAxisSize: MainAxisSize.max,
                children: [
                  Container(width: 50, height: 50, color: Colors.red),
                  Container(width: 50, height: 50, color: Colors.blue),
                ],
              ),
            ),
            // min : le Row (fond jaune) épouse ses enfants.
            ColoredBox(
              color: Colors.yellow,
              child: Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Container(width: 50, height: 50, color: Colors.red),
                  Container(width: 50, height: 50, color: Colors.blue),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│                                          │
│ ███ ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │  max : jaune partout
│ ███ ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│                                          │
│                                          │
│ ███ ███                                  │  min : pas de jaune
│ ███ ███                                  │
│                                          │
└──────────────────────────────────────────┘
```

> Le fond jaune révèle la taille réelle du `Row`. C'est une technique de débogage à retenir : entourez temporairement un widget d'une couleur voyante pour voir sa vraie taille.

---

### 46.21.2 — Quand il faut absolument `MainAxisSize.min`

Deux situations très concrètes.

**Situation 1 : une `Column` dans une carte de taille libre.**
Sans `min`, la `Column` prend toute la hauteur disponible et la carte devient immense.

```dart
Container(
  padding: const EdgeInsets.all(16),
  color: Colors.white,
  child: const Column(
    mainAxisSize: MainAxisSize.min,   // sinon la carte est géante
    children: [Text('Titre'), Text('Sous-titre')],
  ),
)
```

**Situation 2 : un `Row` dans un bouton.**
Sans `min`, le contenu du bouton essaie de s'étaler à l'infini.

```dart
ElevatedButton(
  onPressed: () {},
  child: const Row(
    mainAxisSize: MainAxisSize.min,
    children: [
      Icon(Icons.save),
      SizedBox(width: 8),
      Text('Enregistrer'),
    ],
  ),
)
```

---

### 46.21.3 — Attention : `mainAxisAlignment` devient inutile avec `min`

Si le `Row` fait exactement la taille de ses enfants, il n'y a plus d'espace libre à répartir. `mainAxisAlignment` n'a alors aucun effet visible.

```text
   MainAxisSize.max + spaceBetween  ->  ███              ███
   MainAxisSize.min + spaceBetween  ->  ██████            (rien à répartir)
```

---

## 46.22 — L'axe principal selon `Row` ou `Column`

C'est la source de confusion n° 1 du chapitre. Voici le schéma à garder sous les yeux.

```text
                          ROW
              ┌──────────────────────────────┐
              │                              │
   axe        │   ┌───┐  ┌───┐  ┌───┐        │
   transversal│   │ A │  │ B │  │ C │        │
      ▲       │   └───┘  └───┘  └───┘        │
      │       │                              │
      │       └──────────────────────────────┘
      │        ─────────────────────────────▶
      │                axe PRINCIPAL
      ▼

   Row :  mainAxisAlignment  -> gauche / droite
          crossAxisAlignment -> haut / bas
          mainAxisSize       -> largeur du Row


                        COLUMN
              ┌──────────────────────────────┐
              │   ┌───┐                      │  │
              │   │ A │                      │  │
              │   └───┘                      │  │  axe
              │   ┌───┐                      │  │  PRINCIPAL
              │   │ B │                      │  │
              │   └───┘                      │  ▼
              │   ┌───┐                      │
              │   │ C │                      │
              │   └───┘                      │
              └──────────────────────────────┘
               ──────────────────────────────▶
                     axe transversal

   Column : mainAxisAlignment  -> haut / bas
            crossAxisAlignment -> gauche / droite
            mainAxisSize       -> hauteur de la Column
```

---

### 46.22.1 — Le tableau de conversion

Gardez-le à portée de main les premières semaines.

| Je veux... | Dans une `Row` | Dans une `Column` |
| --- | --- | --- |
| Centrer horizontalement | `mainAxisAlignment: center` | `crossAxisAlignment: center` |
| Centrer verticalement | `crossAxisAlignment: center` | `mainAxisAlignment: center` |
| Coller à gauche | `mainAxisAlignment: start` | `crossAxisAlignment: start` |
| Coller en haut | `crossAxisAlignment: start` | `mainAxisAlignment: start` |
| Coller à droite | `mainAxisAlignment: end` | `crossAxisAlignment: end` |
| Coller en bas | `crossAxisAlignment: end` | `mainAxisAlignment: end` |
| Espacer régulièrement à l'horizontale | `mainAxisAlignment: spaceEvenly` | impossible directement |
| Que tous fassent la même largeur | impossible directement | `crossAxisAlignment: stretch` |

---

### 46.22.2 — La phrase mnémotechnique

> **Le « main » suit le sens du widget.**
> `Row` avance à l'horizontale, donc `main` = horizontal.
> `Column` descend à la verticale, donc `main` = vertical.

Et par conséquent, `cross` est toujours l'autre.

---

### 46.22.3 — Exemple imbriqué : `Column` dans `Row`

En pratique on imbrique en permanence. Il faut alors raisonner niveau par niveau.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Column dans Row')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Colonne 1
              Container(
                width: 60,
                height: 60,
                decoration: const BoxDecoration(
                  color: Colors.deepPurple,
                  shape: BoxShape.circle,
                ),
                alignment: Alignment.center,
                child: const Text(
                  'AL',
                  style: TextStyle(color: Colors.white, fontSize: 20),
                ),
              ),
              const SizedBox(width: 16),
              // Colonne 2 : la Column interne
              const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text(
                    'Alex',
                    style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 4),
                  Text('Niveau 12 — Guerrier'),
                  SizedBox(height: 4),
                  Text('Score : 4 820'),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Column dans Row                         │
├──────────────────────────────────────────┤
│                                          │
│   ╭────╮   Alex                          │
│   │ AL │   Niveau 12 — Guerrier          │
│   ╰────╯   Score : 4 820                 │
│                                          │
│                                          │
└──────────────────────────────────────────┘
```

Décomposition :

```text
   Row (crossAxisAlignment: start)
     │                                 <- colle les 2 enfants EN HAUT
     ├── Container (le cercle AL)
     ├── SizedBox(width: 16)           <- dans un Row, on écarte avec width
     └── Column (crossAxisAlignment: start, mainAxisSize: min)
           │                           <- colle les textes À GAUCHE
           │                              et n'occupe que le nécessaire
           ├── Text('Alex')
           ├── SizedBox(height: 4)     <- dans une Column, on écarte avec height
           ├── Text('Niveau 12...')
           ├── SizedBox(height: 4)
           └── Text('Score : ...')
```

Notez le changement de vocabulaire d'un niveau à l'autre : `crossAxisAlignment: start` signifie « en haut » dans le `Row` et « à gauche » dans la `Column`. C'est exactement le tableau de 46.22.1.

---

## 46.23 — L'erreur « RenderFlex overflowed » : la lire et la corriger

C'est l'erreur la plus fréquente de tout Flutter. Vous la verrez des dizaines de fois. Apprenez à la lire une bonne fois.

---

### 46.23.1 — Provoquons-la volontairement

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Débordement')),
        body: Row(
          children: [
            Container(width: 200, height: 100, color: Colors.red),
            Container(width: 200, height: 100, color: Colors.green),
            Container(width: 200, height: 100, color: Colors.blue),
          ],
        ),
      ),
    );
  }
}
```

Trois enfants de 200 = 600 pixels demandés. L'écran en fait 400. Il manque 200.

**Résultat à l'écran :**

```text
┌──────────────────────────────────────────┐
│  Débordement                             │
├──────────────────────────────────────────┤
│████████████│████████████│███████████▨▨▨▨▨│
│████ rouge ▐│████ vert ▐ │████ bleu ▐▨▨▨▨▨│
│████████████│████████████│███████████▨▨▨▨▨│
│                                       ^^^│
│                          bandes jaunes et│
│                          noires en travers│
└──────────────────────────────────────────┘
```

**Message dans la console :**

```text
════════ Exception caught by rendering library ════════════════════
A RenderFlex overflowed by 200 pixels on the right.

The relevant error-causing widget was:
    Row Row:file:///.../main.dart:14:15

The overflowing RenderFlex has an orientation of Axis.horizontal.
The edge of the RenderFlex that is overflowing has been marked in
the rendering with a yellow and black striped pattern. This is
usually caused by the contents being too big for the RenderFlex.
═══════════════════════════════════════════════════════════════════
```

---

### 46.23.2 — Décoder le message ligne par ligne

| Extrait | Ce que cela vous dit |
| --- | --- |
| `A RenderFlex overflowed by 200 pixels` | Il manque exactement 200 pixels |
| `on the right` | Le débordement est à droite -> c'est un `Row` |
| `on the bottom` | Le débordement est en bas -> c'est une `Column` |
| `The relevant error-causing widget was: Row ...main.dart:14:15` | Le fautif est le `Row` ligne 14, colonne 15 |
| `orientation of Axis.horizontal` | Confirmation : axe horizontal |

> **Le réflexe :** lisez le nombre de pixels et le côté. Le côté vous dit si c'est un `Row` (right/left) ou une `Column` (bottom/top). Le fichier et la ligne vous donnent le coupable exact.

---

### 46.23.3 — Correction 1 : rendre les enfants flexibles

Si les enfants peuvent rétrécir, `Expanded` (section 46.24) résout tout.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Corrigé : Expanded')),
        body: Row(
          children: [
            Expanded(child: Container(height: 100, color: Colors.red)),
            Expanded(child: Container(height: 100, color: Colors.green)),
            Expanded(child: Container(height: 100, color: Colors.blue)),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Corrigé : Expanded                      │
├──────────────────────────────────────────┤
│█████████████│█████████████│██████████████│
│███ rouge ███│███ vert ████│███ bleu █████│  3 x 133 px
│█████████████│█████████████│██████████████│
└──────────────────────────────────────────┘
```

---

### 46.23.4 — Correction 2 : rendre la zone défilante

Si les enfants doivent garder leur taille, il faut pouvoir faire défiler.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Corrigé : défilement')),
        body: SingleChildScrollView(
          scrollDirection: Axis.horizontal,
          child: Row(
            children: [
              Container(width: 200, height: 100, color: Colors.red),
              Container(width: 200, height: 100, color: Colors.green),
              Container(width: 200, height: 100, color: Colors.blue),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Corrigé : défilement                    │
├──────────────────────────────────────────┤
│████████████████████│█████████████████████│
│█████ rouge ████████│██████ vert █████████│  -> on fait glisser
│████████████████████│█████████████████████│     vers la gauche
│                                          │     pour voir le bleu
└──────────────────────────────────────────┘
```

---

### 46.23.5 — Correction 3 : passer à la ligne

Avec `Wrap` (section 46.28), ce qui ne rentre pas descend d'une ligne.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Corrigé : Wrap')),
        body: Wrap(
          children: [
            Container(width: 200, height: 100, color: Colors.red),
            Container(width: 200, height: 100, color: Colors.green),
            Container(width: 200, height: 100, color: Colors.blue),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Corrigé : Wrap                          │
├──────────────────────────────────────────┤
│████████████████████│█████████████████████│
│█████ rouge ████████│██████ vert █████████│
│████████████████████│█████████████████████│
│████████████████████│                     │
│█████ bleu █████████│   <- descendu       │
│████████████████████│                     │
└──────────────────────────────────────────┘
```

---

### 46.23.6 — Le cas particulier du texte trop long

Un `Text` dans un `Row` ne rétrécit pas tout seul : il demande la largeur nécessaire à son contenu, et déborde.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // 1. Sans protection : DÉBORDE.
              // Row(children: [Icon(Icons.info), Text('Un texte ...')]),

              // 2. Avec Expanded : le texte revient à la ligne.
              const Row(
                children: [
                  Icon(Icons.info),
                  SizedBox(width: 8),
                  Expanded(
                    child: Text(
                      'Un texte très long qui ne tient absolument pas '
                      'sur une seule ligne dans cette rangée.',
                    ),
                  ),
                ],
              ),
              const SizedBox(height: 30),
              // 3. Avec Expanded + ellipsis : coupé avec des points.
              const Row(
                children: [
                  Icon(Icons.warning),
                  SizedBox(width: 8),
                  Expanded(
                    child: Text(
                      'Un texte très long qui ne tient absolument pas '
                      'sur une seule ligne dans cette rangée.',
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│                                          │
│  (i)  Un texte très long qui ne tient    │
│       absolument pas sur une seule ligne │
│       dans cette rangée.                 │
│                                          │
│  (!)  Un texte très long qui ne tie...   │
│                                          │
└──────────────────────────────────────────┘
```

> **Règle à retenir :** dans un `Row`, un `Text` de longueur inconnue doit **toujours** être enveloppé dans un `Expanded` ou un `Flexible`.

---

### 46.23.7 — L'arbre de décision

```text
   « RenderFlex overflowed by N pixels »
                  │
        Les enfants peuvent-ils rétrécir ?
                  │
        ┌─────────┴──────────┐
       OUI                  NON
        │                    │
        v                    │
    Expanded /               │
    Flexible                 │
                   Doit-on tout voir en même temps ?
                             │
                   ┌─────────┴─────────┐
                  NON                 OUI
                   │                   │
                   v                   v
          SingleChildScrollView      Wrap
          ou ListView             (passage à la ligne)
```

---

## 46.24 — `Expanded`

`Expanded` force son enfant à **remplir tout l'espace restant** sur l'axe principal d'un `Row`, d'une `Column` ou d'un `Flex`.

```dart
Expanded({
  Key? key,
  int flex = 1,
  required Widget child,
})
```

---

### 46.24.1 — Comment cela fonctionne

`Row` et `Column` procèdent en deux temps :

```text
   ÉTAPE 1 : mesurer les enfants NON flexibles
   ───────────────────────────────────────────
   Largeur totale : 400
   Enfant fixe de 100 -> il reste 300


   ÉTAPE 2 : partager le reste entre les Expanded
   ───────────────────────────────────────────────
   Il reste 300, il y a 2 Expanded de flex 1
   -> chacun reçoit 150 (contrainte SERRÉE)
```

---

### 46.24.2 — Exemple

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Expanded')),
        body: Row(
          children: [
            Container(width: 100, height: 150, color: Colors.grey),
            Expanded(child: Container(height: 150, color: Colors.red)),
            Expanded(child: Container(height: 150, color: Colors.blue)),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Expanded                                │
├──────────────────────────────────────────┤
│██████████│███████████████│███████████████│
│██ gris ██│███  rouge  ███│███   bleu  ███│
│██████████│███████████████│███████████████│
│   100         150             150        │
│  (fixe)      (reste/2)      (reste/2)    │
└──────────────────────────────────────────┘
```

---

### 46.24.3 — `Expanded` dans une `Column` : le grand classique

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: SafeArea(
          child: Column(
            children: [
              // En-tête : hauteur fixe
              Container(
                height: 70,
                color: Colors.indigo,
                alignment: Alignment.center,
                child: const Text(
                  'EN-TÊTE',
                  style: TextStyle(color: Colors.white, fontSize: 20),
                ),
              ),
              // Contenu : tout le reste
              Expanded(
                child: Container(
                  color: Colors.indigo.shade50,
                  alignment: Alignment.center,
                  child: const Text('CONTENU (prend tout le reste)'),
                ),
              ),
              // Pied : hauteur fixe
              Container(
                height: 60,
                color: Colors.indigo.shade700,
                alignment: Alignment.center,
                child: const Text(
                  'PIED DE PAGE',
                  style: TextStyle(color: Colors.white),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│              EN-TÊTE                     │  70 fixes
├──────────────────────────────────────────┤
│                                          │
│                                          │
│      CONTENU (prend tout le reste)       │  tout le reste
│                                          │
│                                          │
├──────────────────────────────────────────┤
│            PIED DE PAGE                  │  60 fixes
└──────────────────────────────────────────┘
```

C'est le squelette de la moitié des applications que vous écrirez.

---

### 46.24.4 — `Expanded` impose une contrainte SERRÉE

Point crucial : `Expanded` ne dit pas « tu peux aller jusqu'à 150 ». Il dit « **tu fais exactement 150** ».

Conséquence : la `width` (dans un `Row`) ou la `height` (dans une `Column`) de l'enfant est **ignorée**.

```dart
// La width: 20 est IGNORÉE : Expanded impose la largeur.
Expanded(child: Container(width: 20, color: Colors.red))
```

Si vous voulez que l'enfant puisse rester plus petit, il faut `Flexible` (section 46.26).

---

## 46.25 — `flex` et le partage proportionnel

Le paramètre `flex` d'un `Expanded` indique **combien de parts** il prend dans le partage.

```text
   Espace restant : 300

   Expanded(flex: 1)  ┐
   Expanded(flex: 2)  ├─ total des flex = 1 + 2 + 3 = 6
   Expanded(flex: 3)  ┘

   -> 1 part = 300 / 6 = 50

   flex 1 -> 1 x 50 =  50
   flex 2 -> 2 x 50 = 100
   flex 3 -> 3 x 50 = 150
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('flex')),
        body: Column(
          children: [
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.red,
                alignment: Alignment.center,
                child: const Text('flex: 1'),
              ),
            ),
            Expanded(
              flex: 2,
              child: Container(
                color: Colors.green,
                alignment: Alignment.center,
                child: const Text('flex: 2'),
              ),
            ),
            Expanded(
              flex: 3,
              child: Container(
                color: Colors.blue,
                alignment: Alignment.center,
                child: const Text('flex: 3'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  flex                                    │
├──────────────────────────────────────────┤
│               flex: 1                    │  1/6 de la hauteur
├──────────────────────────────────────────┤
│                                          │
│               flex: 2                    │  2/6 de la hauteur
│                                          │
├──────────────────────────────────────────┤
│                                          │
│                                          │
│               flex: 3                    │  3/6 de la hauteur
│                                          │
│                                          │
└──────────────────────────────────────────┘
```

---

### 46.25.1 — Les proportions courantes

| Objectif | Écriture |
| --- | --- |
| Moitié / moitié | `flex: 1` et `flex: 1` |
| Un tiers / deux tiers | `flex: 1` et `flex: 2` |
| 25 % / 75 % | `flex: 1` et `flex: 3` |
| 20 % / 30 % / 50 % | `flex: 2`, `flex: 3`, `flex: 5` |

> Astuce : pour raisonner en pourcentages, choisissez des `flex` dont la somme fait 10 ou 100.

---

### 46.25.2 — Attention : `flex` ne compte que l'espace RESTANT

```dart
Row(
  children: [
    Container(width: 300, color: Colors.grey),   // fixe
    Expanded(flex: 1, child: ...),
    Expanded(flex: 1, child: ...),
  ],
)
```

Sur un écran de 400, l'espace restant est de 100, pas 400. Chaque `Expanded` reçoit 50, pas 200. Le `flex` s'applique après déduction des enfants fixes.

---

## 46.26 — `Flexible` et la différence avec `Expanded`

`Flexible` ressemble à `Expanded`, avec un paramètre de plus :

```dart
Flexible({
  Key? key,
  int flex = 1,
  FlexFit fit = FlexFit.loose,
  required Widget child,
})
```

Et là est toute la différence : `fit`.

| Widget | `fit` | Contrainte donnée à l'enfant |
| --- | --- | --- |
| `Flexible` | `FlexFit.loose` (défaut) | `0 .. part` -> l'enfant **peut** être plus petit |
| `Expanded` | `FlexFit.tight` | `part .. part` -> l'enfant **doit** faire cette taille |

Autrement dit :

```dart
Expanded(child: x)                                 // équivaut à
Flexible(fit: FlexFit.tight, child: x)
```

---

### 46.26.1 — La démonstration côte à côte

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Flexible vs Expanded')),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            const Text('Avec Expanded (fit: tight)'),
            Row(
              children: [
                Expanded(
                  child: Container(height: 60, color: Colors.red),
                ),
                Expanded(
                  child: Container(height: 60, color: Colors.blue),
                ),
              ],
            ),
            const Text('Avec Flexible (fit: loose) et enfants petits'),
            Row(
              children: [
                Flexible(
                  child: Container(width: 50, height: 60, color: Colors.red),
                ),
                Flexible(
                  child: Container(width: 50, height: 60, color: Colors.blue),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Flexible vs Expanded                    │
├──────────────────────────────────────────┤
│      Avec Expanded (fit: tight)          │
│█████████████████████│████████████████████│  chacun remplit
│█████████████████████│████████████████████│  sa moitié
│                                          │
│  Avec Flexible (fit: loose) ...          │
│████│████                                 │  chacun garde
│████│████                                 │  ses 50 px
└──────────────────────────────────────────┘
```

Avec `Expanded`, le `width: 50` serait ignoré et chaque boîte ferait 200. Avec `Flexible`, la boîte a le **droit** d'aller jusqu'à 200, mais elle choisit 50.

---

### 46.26.2 — Quand `Flexible` est indispensable

Quand un enfant doit pouvoir rétrécir **si nécessaire**, mais rester à sa taille naturelle sinon. Le cas type : un texte de longueur variable suivi d'une icône.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _ligne(String texte) {
    return Container(
      margin: const EdgeInsets.all(8),
      padding: const EdgeInsets.all(12),
      color: Colors.amber.shade100,
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Flexible(
            child: Text(
              texte,
              maxLines: 1,
              overflow: TextOverflow.ellipsis,
            ),
          ),
          const SizedBox(width: 8),
          const Icon(Icons.check_circle, color: Colors.green, size: 20),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              _ligne('Court'),
              _ligne('Un titre de longueur moyenne'),
              _ligne('Un titre vraiment très très très long qui ne tient pas'),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│                                          │
│   ┌──────────────┐                       │
│   │ Court    (v) │   <- la boîte est     │
│   └──────────────┘      petite           │
│                                          │
│   ┌──────────────────────────────┐       │
│   │ Un titre de longueur ... (v) │       │
│   └──────────────────────────────┘       │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ Un titre vraiment très trè... (v)  │  │
│  └────────────────────────────────────┘  │
│                                          │
└──────────────────────────────────────────┘
```

Avec `Expanded` à la place de `Flexible`, les trois boîtes feraient toutes la largeur maximale : le texte court serait suivi d'un grand vide.

---

### 46.26.3 — Le tableau de choix

| Situation | Widget |
| --- | --- |
| Remplir tout le reste, coûte que coûte | `Expanded` |
| Diviser l'espace en parts égales | `Expanded` avec le même `flex` |
| Autoriser à rétrécir sans forcer à grandir | `Flexible` |
| Un texte de longueur inconnue dans un `Row` | `Flexible` (ou `Expanded` si on veut la largeur pleine) |
| Créer un vide élastique | `Spacer` (section 46.27) |

---

### 46.26.4 — Les deux ne fonctionnent QUE dans un `Flex`

`Expanded`, `Flexible` et `Spacer` sont des `ParentDataWidget` : ils écrivent une information destinée au parent. Si le parent n'est pas un `Row`, une `Column` ou un `Flex`, l'information n'a aucun sens et Flutter plante :

```text
════════ Exception caught by widgets library ═══════════════════════
Incorrect use of ParentDataWidget.

The ParentDataWidget Expanded(flex: 1) wants to apply ParentData of
type FlexParentData to a RenderObject, which has been set up to
accept ParentData of type StackParentData.

Usually, this means that the Expanded widget has the wrong ancestor
RenderObjectWidget. Typically, Expanded widgets are placed directly
inside Flex widgets.
═══════════════════════════════════════════════════════════════════
```

Traduction : « vous avez mis un `Expanded` dans un `Stack` (ou un `Container`, ou un `Center`...) ». Corrigez en supprimant l'`Expanded`, ou en ajoutant le `Row`/`Column` manquant.

> Attention aussi : « directement à l'intérieur ». Un `Expanded` enveloppé dans un `Padding` lui-même dans un `Row` provoque la même erreur. Inversez : `Expanded(child: Padding(...))`.

---

## 46.27 — `Spacer`

`Spacer` est un vide flexible. C'est littéralement `Expanded(child: SizedBox.shrink())`.

```dart
Spacer({Key? key, int flex = 1})
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Spacer')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              Row(
                children: [
                  const Icon(Icons.person, size: 32),
                  const SizedBox(width: 8),
                  const Text('Alex', style: TextStyle(fontSize: 20)),
                  const Spacer(),                       // pousse la suite
                  const Text('4 820 pts'),
                  const SizedBox(width: 8),
                  Icon(Icons.star, color: Colors.amber.shade700),
                ],
              ),
              const SizedBox(height: 24),
              Row(
                children: [
                  Container(width: 40, height: 40, color: Colors.red),
                  const Spacer(flex: 1),
                  Container(width: 40, height: 40, color: Colors.green),
                  const Spacer(flex: 3),                // 3 fois plus large
                  Container(width: 40, height: 40, color: Colors.blue),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Spacer                                  │
├──────────────────────────────────────────┤
│  (o) Alex              4 820 pts  (*)    │
│      ^^^^^^^^^^^^^^^^^^                  │
│      le Spacer occupe tout le milieu     │
│                                          │
│  ███      ███                       ███  │
│     ^^^^^^   ^^^^^^^^^^^^^^^^^^^^^^^     │
│     1 part          3 parts              │
└──────────────────────────────────────────┘
```

---

### 46.27.1 — `Spacer` ou `mainAxisAlignment` ?

Les deux font parfois la même chose :

```dart
// Version A
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [const Text('Gauche'), const Text('Droite')],
)

// Version B — résultat identique
Row(
  children: [const Text('Gauche'), const Spacer(), const Text('Droite')],
)
```

Utilisez `Spacer` quand vous avez besoin de contrôle **inégal** : pousser seulement certains éléments, ou avec des proportions.

> **Piège documenté :** mettre un `Spacer` dans un `Row` dont le `mainAxisAlignment` vaut `spaceBetween`, `spaceAround` ou `spaceEvenly` rend ces valeurs sans effet. Le `Spacer` a déjà mangé tout l'espace libre : il n'en reste plus à répartir.

> **Autre piège :** `Spacer` ne fonctionne pas dans un `Row` ou une `Column` en `MainAxisSize.min`, ni dans une zone défilante. Il lui faut un espace libre borné à consommer. Sinon : `RenderFlex children have non-zero flex but incoming height constraints are unbounded.`

---

## 46.28 — `Wrap`

`Wrap` place ses enfants comme un `Row`, mais **passe à la ligne** quand la place manque.

```dart
Wrap({
  Key? key,
  Axis direction = Axis.horizontal,
  WrapAlignment alignment = WrapAlignment.start,
  double spacing = 0.0,
  WrapAlignment runAlignment = WrapAlignment.start,
  double runSpacing = 0.0,
  WrapCrossAlignment crossAxisAlignment = WrapCrossAlignment.start,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  Clip clipBehavior = Clip.none,
  List<Widget> children = const <Widget>[],
})
```

Vocabulaire propre à `Wrap` : une **run** est une ligne. `spacing` sépare les éléments **dans** une ligne, `runSpacing` sépare les **lignes** entre elles.

```text
     spacing                  spacing
        ↕                        ↕
   ┌────┐ ┌────┐ ┌────┐ ┌────┐
   │ A  │ │ B  │ │ C  │ │ D  │      <- run 1
   └────┘ └────┘ └────┘ └────┘
                                    <- runSpacing
   ┌────┐ ┌────┐
   │ E  │ │ F  │                    <- run 2
   └────┘ └────┘
```

---

### 46.28.1 — Exemple : des étiquettes

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _tag(String texte, Color couleur) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 8),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(20),
      ),
      child: Text(texte, style: const TextStyle(color: Colors.white)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Wrap')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Wrap(
            spacing: 10,        // écart horizontal
            runSpacing: 10,     // écart vertical entre les lignes
            children: [
              _tag('Épée', Colors.red),
              _tag('Bouclier légendaire', Colors.blue),
              _tag('Potion', Colors.green),
              _tag('Arc elfique', Colors.purple),
              _tag('Or', Colors.orange),
              _tag('Grimoire ancien', Colors.teal),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Wrap                                    │
├──────────────────────────────────────────┤
│                                          │
│  (Épée) (Bouclier légendaire)            │
│                                          │
│  (Potion) (Arc elfique) (Or)             │
│                                          │
│  (Grimoire ancien)                       │
│                                          │
└──────────────────────────────────────────┘
```

Le même code dans un `Row` déborderait immédiatement.

---

### 46.28.2 — `alignment` et `runAlignment`

- `alignment` aligne les éléments **à l'intérieur d'une ligne** (axe principal) ;
- `runAlignment` aligne les **lignes entre elles** (axe transversal).

Les valeurs de `WrapAlignment` sont les mêmes noms que `MainAxisAlignment` : `start`, `end`, `center`, `spaceBetween`, `spaceAround`, `spaceEvenly`.

```text
   alignment: WrapAlignment.center

   ┌──────────────────────────────────┐
   │      [A] [B] [C] [D]             │  <- centré dans la ligne
   │          [E] [F]                 │
   └──────────────────────────────────┘
```

---

### 46.28.3 — `Wrap` vertical

`direction: Axis.vertical` empile de haut en bas et passe à la **colonne** suivante.

```dart
Wrap(
  direction: Axis.vertical,
  spacing: 8,
  runSpacing: 8,
  children: [...],
)
```

```text
   ┌──────────────────────────────────┐
   │  [A]   [D]                       │
   │  [B]   [E]                       │  colonne 1, puis colonne 2
   │  [C]                             │
   └──────────────────────────────────┘
```

> **Important :** `Expanded`, `Flexible` et `Spacer` ne fonctionnent PAS dans un `Wrap`. `Wrap` n'est pas un `Flex`. Vous obtiendriez l'erreur `Incorrect use of ParentDataWidget`.

---

## 46.29 — `Stack`

`Stack` **superpose** ses enfants, comme des feuilles empilées.

```dart
Stack({
  Key? key,
  AlignmentGeometry alignment = AlignmentDirectional.topStart,
  TextDirection? textDirection,
  StackFit fit = StackFit.loose,
  Clip clipBehavior = Clip.hardEdge,
  List<Widget> children = const <Widget>[],
})
```

L'ordre du tableau `children` est l'ordre de dessin : **le premier est au fond, le dernier est au-dessus**.

```text
   children: [ A, B, C ]

              ┌───────────┐
              │     C     │   dessiné en DERNIER  -> au-dessus
              └───────────┘
           ┌───────────┐
           │     B     │
           └───────────┘
        ┌───────────┐
        │     A     │      dessiné en PREMIER  -> au fond
        └───────────┘
```

---

### 46.29.1 — Exemple

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Stack')),
        body: Center(
          child: Stack(
            children: [
              Container(width: 250, height: 250, color: Colors.red),
              Container(width: 180, height: 180, color: Colors.green),
              Container(width: 110, height: 110, color: Colors.blue),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Stack                                   │
├──────────────────────────────────────────┤
│                                          │
│      ┌──────────────────────┐            │
│      │████████████│rouge    │            │
│      │███ bleu ███│         │            │
│      │████████████│         │            │
│      │██ vert ████│         │            │
│      │████████████┘         │            │
│      │                      │            │
│      └──────────────────────┘            │
│                                          │
└──────────────────────────────────────────┘
```

Les trois carrés sont alignés en haut à gauche : c'est la valeur par défaut de `alignment` (`AlignmentDirectional.topStart`).

---

### 46.29.2 — Quelle taille prend un `Stack` ?

C'est la question importante.

```text
   Le Stack prend la taille du plus GRAND de ses enfants
   NON POSITIONNÉS.

   Si TOUS les enfants sont Positioned :
       -> il prend tout l'espace disponible
       -> et si l'espace est non borné, c'est une erreur
```

Dans l'exemple ci-dessus, le plus grand enfant non positionné fait 250 x 250 : le `Stack` fait 250 x 250.

---

### 46.29.3 — `alignment` et `fit`

`alignment` place tous les enfants **non positionnés**.

```dart
Stack(
  alignment: Alignment.center,   // tout est centré
  children: [...],
)
```

`fit` décide de la contrainte donnée aux enfants non positionnés :

| Valeur | Effet |
| --- | --- |
| `StackFit.loose` (défaut) | Les enfants peuvent choisir leur taille |
| `StackFit.expand` | Les enfants sont forcés à remplir le `Stack` |
| `StackFit.passthrough` | La contrainte du parent est transmise telle quelle |

---

### 46.29.4 — Exemple pratique : une bulle de notification

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: Stack(
            clipBehavior: Clip.none,   // pour ne pas couper la bulle
            children: [
              const Icon(Icons.inventory_2, size: 72, color: Colors.brown),
              Positioned(
                top: -4,
                right: -4,
                child: Container(
                  padding: const EdgeInsets.all(6),
                  decoration: const BoxDecoration(
                    color: Colors.red,
                    shape: BoxShape.circle,
                  ),
                  child: const Text(
                    '7',
                    style: TextStyle(color: Colors.white, fontSize: 12),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│                                          │
│                             (7)          │  <- bulle rouge
│              ┌────────────┐              │     débordant du coin
│              │            │              │
│              │   caisse   │              │
│              │            │              │
│              └────────────┘              │
│                                          │
└──────────────────────────────────────────┘
```

> `clipBehavior: Clip.none` est indispensable ici : par défaut (`Clip.hardEdge`), tout ce qui dépasse du `Stack` est **coupé**, et la bulle serait rognée.

---

## 46.30 — `Positioned`

`Positioned` place précisément un enfant **dans un `Stack`**.

```dart
Positioned({
  Key? key,
  double? left,
  double? top,
  double? right,
  double? bottom,
  double? width,
  double? height,
  required Widget child,
})
```

Chaque valeur est une distance **depuis le bord correspondant du `Stack`**.

```text
   ┌───────────── Stack ──────────────────┐
   │        ↕ top                         │
   │  ←──→  ┌────────────┐                │
   │  left  │            │  ←──────────→  │
   │        │   child    │     right      │
   │        └────────────┘                │
   │        ↕ bottom                      │
   └──────────────────────────────────────┘
```

---

### 46.30.1 — Les combinaisons

Vous ne pouvez pas donner les trois valeurs d'un même axe. Les combinaisons valides sont :

| Écriture | Effet horizontal |
| --- | --- |
| `left: 10` | collé à 10 du bord gauche, largeur libre |
| `right: 10` | collé à 10 du bord droit, largeur libre |
| `left: 10, right: 10` | **étiré** entre les deux bords |
| `left: 10, width: 80` | à 10 du bord gauche, largeur 80 |
| rien | position définie par le `alignment` du `Stack` |

Si vous donnez `left`, `right` **et** `width` en même temps :

```text
'package:flutter/src/widgets/basic.dart': Failed assertion:
'left == null || right == null || width == null': is not true.
```

---

### 46.30.2 — Exemple complet

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Positioned')),
        body: Center(
          child: Container(
            width: 300,
            height: 300,
            color: Colors.grey.shade300,
            child: Stack(
              children: [
                Positioned(
                  top: 10,
                  left: 10,
                  child: Container(width: 60, height: 60, color: Colors.red),
                ),
                Positioned(
                  top: 10,
                  right: 10,
                  child: Container(width: 60, height: 60, color: Colors.green),
                ),
                Positioned(
                  bottom: 10,
                  left: 10,
                  child: Container(width: 60, height: 60, color: Colors.blue),
                ),
                Positioned(
                  bottom: 10,
                  right: 10,
                  child: Container(width: 60, height: 60, color: Colors.purple),
                ),
                // Étiré horizontalement : left ET right.
                Positioned(
                  left: 20,
                  right: 20,
                  top: 130,
                  height: 40,
                  child: Container(
                    color: Colors.black87,
                    alignment: Alignment.center,
                    child: const Text(
                      'left + right = étiré',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Positioned                              │
├──────────────────────────────────────────┤
│    ┌────────────────────────────────┐    │
│    │ ███                       ███  │    │  rouge / vert
│    │ ███                       ███  │    │
│    │                                │    │
│    │  ████████████████████████████  │    │  barre noire étirée
│    │                                │    │
│    │ ███                       ███  │    │  bleu / violet
│    │ ███                       ███  │    │
│    └────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

---

### 46.30.3 — `Positioned.fill`

Raccourci pour « occupe tout le `Stack` » :

```dart
Positioned.fill(child: monWidget)
// équivaut à
Positioned(left: 0, top: 0, right: 0, bottom: 0, child: monWidget)
```

Très utile pour poser un voile sombre par-dessus une image ou une couleur :

```dart
Stack(
  children: [
    Container(width: 300, height: 200, color: Colors.orange),
    Positioned.fill(
      child: ColoredBox(color: Colors.black.withValues(alpha: 0.4)),
    ),
    const Positioned(
      bottom: 12,
      left: 12,
      child: Text('Titre', style: TextStyle(color: Colors.white)),
    ),
  ],
)
```

---

### 46.30.4 — Erreur : `Positioned` hors d'un `Stack`

```text
════════ Exception caught by widgets library ═══════════════════════
Incorrect use of ParentDataWidget.

The ParentDataWidget Positioned(left: 10.0, top: 10.0) wants to
apply ParentData of type StackParentData to a RenderObject, which
has been set up to accept ParentData of type FlexParentData.

Usually, this means that the Positioned widget has the wrong
ancestor RenderObjectWidget. Typically, Positioned widgets are
placed directly inside Stack widgets.
═══════════════════════════════════════════════════════════════════
```

Même famille d'erreur qu'avec `Expanded` : un `ParentDataWidget` dans le mauvais parent.

---

## 46.31 — `Align` dans un `Stack`

`Positioned` travaille en pixels. `Align` travaille en **proportions**. Dans un `Stack`, les deux cohabitent.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _pastille(String t) => Container(
        width: 46,
        height: 46,
        decoration: const BoxDecoration(
          color: Colors.deepPurple,
          shape: BoxShape.circle,
        ),
        alignment: Alignment.center,
        child: Text(t, style: const TextStyle(color: Colors.white)),
      );

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Align dans Stack')),
        body: Center(
          child: Container(
            width: 300,
            height: 300,
            color: Colors.grey.shade200,
            child: Stack(
              children: [
                Align(alignment: Alignment.topCenter, child: _pastille('N')),
                Align(alignment: Alignment.centerRight, child: _pastille('E')),
                Align(alignment: Alignment.bottomCenter, child: _pastille('S')),
                Align(alignment: Alignment.centerLeft, child: _pastille('O')),
                Align(alignment: Alignment.center, child: _pastille('C')),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Align dans Stack                        │
├──────────────────────────────────────────┤
│    ┌────────────────────────────────┐    │
│    │              (N)               │    │
│    │                                │    │
│    │                                │    │
│    │  (O)         (C)         (E)   │    │
│    │                                │    │
│    │                                │    │
│    │              (S)               │    │
│    └────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

---

### 46.31.1 — `Positioned` ou `Align` ?

| Besoin | Widget |
| --- | --- |
| « à 12 pixels du bord bas » | `Positioned(bottom: 12, ...)` |
| « collé en bas au milieu » | `Align(alignment: Alignment.bottomCenter, ...)` |
| « au tiers de la largeur » | `Align(alignment: Alignment(-0.33, 0), ...)` |
| « étiré d'un bord à l'autre » | `Positioned(left: 0, right: 0, ...)` |

Point important : un enfant `Align` (non `Positioned`) **compte dans la taille du `Stack`**. Un `Positioned` ne compte pas. Si tous vos enfants sont `Positioned`, le `Stack` n'a plus de taille naturelle.

---

## 46.32 — `IndexedStack`

`IndexedStack` est un `Stack` qui n'affiche **qu'un seul** de ses enfants, celui de l'index donné. Les autres restent dans l'arbre : leur état est conservé.

```dart
IndexedStack({
  Key? key,
  AlignmentGeometry alignment = AlignmentDirectional.topStart,
  TextDirection? textDirection,
  StackFit sizing = StackFit.loose,
  Clip clipBehavior = Clip.hardEdge,
  int? index = 0,
  List<Widget> children = const <Widget>[],
})
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatefulWidget {
  const MonApp({super.key});

  @override
  State<MonApp> createState() => _MonAppState();
}

class _MonAppState extends State<MonApp> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('IndexedStack')),
        body: IndexedStack(
          index: _index,
          children: const [
            _Page(titre: 'Inventaire', couleur: Colors.brown),
            _Page(titre: 'Carte', couleur: Colors.teal),
            _Page(titre: 'Réglages', couleur: Colors.indigo),
          ],
        ),
        bottomNavigationBar: BottomNavigationBar(
          currentIndex: _index,
          onTap: (i) => setState(() => _index = i),
          items: const [
            BottomNavigationBarItem(
              icon: Icon(Icons.inventory_2),
              label: 'Inventaire',
            ),
            BottomNavigationBarItem(icon: Icon(Icons.map), label: 'Carte'),
            BottomNavigationBarItem(
              icon: Icon(Icons.settings),
              label: 'Réglages',
            ),
          ],
        ),
      ),
    );
  }
}

class _Page extends StatefulWidget {
  const _Page({required this.titre, required this.couleur});

  final String titre;
  final Color couleur;

  @override
  State<_Page> createState() => _PageState();
}

class _PageState extends State<_Page> {
  int _compteur = 0;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: widget.couleur.withValues(alpha: 0.15),
      alignment: Alignment.center,
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Text(widget.titre, style: const TextStyle(fontSize: 28)),
          const SizedBox(height: 16),
          Text('Compteur : $_compteur', style: const TextStyle(fontSize: 20)),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: () => setState(() => _compteur++),
            child: const Text('+1'),
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
│  IndexedStack                            │
├──────────────────────────────────────────┤
│                                          │
│              Inventaire                  │
│                                          │
│            Compteur : 3                  │
│                                          │
│               [ +1 ]                     │
│                                          │
├──────────────────────────────────────────┤
│  Inventaire  |   Carte   |   Réglages    │
└──────────────────────────────────────────┘

Vous cliquez 3 fois sur +1, vous allez sur « Carte »,
puis vous revenez : le compteur affiche toujours 3.
```

---

### 46.32.1 — Pourquoi c'est utile

Sans `IndexedStack`, en remplaçant simplement le widget affiché, l'ancienne page est détruite : son `State` disparaît, la position de défilement est perdue, le compteur repart à zéro.

| Approche | État conservé ? | Coût mémoire |
| --- | --- | --- |
| Remplacer le widget | Non | Faible |
| `IndexedStack` | **Oui** | Toutes les pages restent construites |

> `IndexedStack` construit **tous** ses enfants dès le départ. Avec dix pages lourdes, le démarrage est ralenti. Utilisez-le pour trois ou quatre onglets, pas plus.

> `index: null` masque tous les enfants.

---

## 46.33 — `AspectRatio`

`AspectRatio` impose un **rapport largeur / hauteur** à son enfant.

```dart
AspectRatio({
  Key? key,
  required double aspectRatio,   // largeur / hauteur
  Widget? child,
})
```

| Valeur | Forme |
| --- | --- |
| `1.0` | carré |
| `16 / 9` | format vidéo paysage |
| `4 / 3` | format photo classique |
| `9 / 16` | portrait (story) |
| `0.5` | deux fois plus haut que large |

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('AspectRatio')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              AspectRatio(
                aspectRatio: 16 / 9,
                child: Container(
                  color: Colors.blueGrey,
                  alignment: Alignment.center,
                  child: const Text(
                    '16 / 9',
                    style: TextStyle(color: Colors.white, fontSize: 24),
                  ),
                ),
              ),
              const SizedBox(height: 16),
              AspectRatio(
                aspectRatio: 1,
                child: Container(
                  color: Colors.deepOrange,
                  alignment: Alignment.center,
                  child: const Text(
                    'carré',
                    style: TextStyle(color: Colors.white, fontSize: 24),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  AspectRatio                             │
├──────────────────────────────────────────┤
│  ┌────────────────────────────────────┐  │
│  │            16 / 9                  │  │  largeur 368
│  └────────────────────────────────────┘  │  hauteur 207
│                                          │
│  ┌────────────────────────────────────┐  │
│  │                                    │  │
│  │              carré                 │  │  368 x 368
│  │                                    │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

---

### 46.33.1 — Comment `AspectRatio` choisit sa taille

Il essaie d'abord la **largeur maximale** autorisée, puis calcule la hauteur correspondante. Si cette hauteur dépasse la contrainte, il repart de la hauteur maximale et recalcule la largeur.

```text
   Contrainte : 0..400 large, 0..300 haut,  aspectRatio = 16/9

   Essai 1 : largeur = 400 -> hauteur = 400 * 9/16 = 225   -> 225 <= 300 : OK
   Résultat : 400 x 225


   Contrainte : 0..400 large, 0..150 haut,  aspectRatio = 16/9

   Essai 1 : largeur = 400 -> hauteur = 225                -> 225 > 150 : refusé
   Essai 2 : hauteur = 150 -> largeur = 150 * 16/9 = 267
   Résultat : 267 x 150
```

> Si la contrainte est **non bornée** dans les deux directions, `AspectRatio` ne peut rien décider. Placez-le toujours dans une zone bornée (par exemple à l'intérieur d'un `SizedBox` ou d'une `Column` de largeur connue).

---

## 46.34 — `FractionallySizedBox`

`FractionallySizedBox` dimensionne son enfant en **fraction de l'espace disponible**.

```dart
FractionallySizedBox({
  Key? key,
  AlignmentGeometry alignment = Alignment.center,
  double? widthFactor,
  double? heightFactor,
  Widget? child,
})
```

- `widthFactor: 0.5` -> la moitié de la largeur disponible ;
- `heightFactor: 0.25` -> le quart de la hauteur disponible ;
- un facteur `null` -> la dimension n'est pas contrainte par ce widget.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('FractionallySizedBox')),
        body: Column(
          children: [
            SizedBox(
              height: 80,
              child: FractionallySizedBox(
                widthFactor: 1.0,
                alignment: Alignment.centerLeft,
                child: Container(color: Colors.red),
              ),
            ),
            SizedBox(
              height: 80,
              child: FractionallySizedBox(
                widthFactor: 0.6,
                alignment: Alignment.centerLeft,
                child: Container(color: Colors.orange),
              ),
            ),
            SizedBox(
              height: 80,
              child: FractionallySizedBox(
                widthFactor: 0.25,
                alignment: Alignment.centerLeft,
                child: Container(color: Colors.green),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  FractionallySizedBox                    │
├──────────────────────────────────────────┤
│██████████████████████████████████████████│  100 %
│██████████████████████████████████████████│
│██████████████████████████                │  60 %
│██████████████████████████                │
│██████████                                │  25 %
│██████████                                │
└──────────────────────────────────────────┘
```

C'est l'outil idéal pour une barre de progression, une barre de vie, une jauge.

---

### 46.34.1 — Exemple : une barre de vie

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatefulWidget {
  const MonApp({super.key});

  @override
  State<MonApp> createState() => _MonAppState();
}

class _MonAppState extends State<MonApp> {
  double _vie = 0.75;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Barre de vie')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(
                height: 26,
                decoration: BoxDecoration(
                  color: Colors.grey.shade400,
                  borderRadius: BorderRadius.circular(13),
                ),
                child: FractionallySizedBox(
                  widthFactor: _vie,
                  alignment: Alignment.centerLeft,
                  child: Container(
                    decoration: BoxDecoration(
                      color: _vie > 0.3 ? Colors.green : Colors.red,
                      borderRadius: BorderRadius.circular(13),
                    ),
                  ),
                ),
              ),
              const SizedBox(height: 24),
              Text('Vie : ${(_vie * 100).round()} %'),
              const SizedBox(height: 16),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    onPressed: () => setState(() {
                      _vie = (_vie - 0.1).clamp(0.0, 1.0);
                    }),
                    child: const Text('- 10'),
                  ),
                  const SizedBox(width: 16),
                  ElevatedButton(
                    onPressed: () => setState(() {
                      _vie = (_vie + 0.1).clamp(0.0, 1.0);
                    }),
                    child: const Text('+ 10'),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  Barre de vie                            │
├──────────────────────────────────────────┤
│                                          │
│  (████████████████████████░░░░░░░░░)     │  75 % vert
│                                          │
│              Vie : 75 %                  │
│                                          │
│         [ - 10 ]    [ + 10 ]             │
│                                          │
└──────────────────────────────────────────┘
```

---

### 46.34.2 — Un facteur supérieur à 1

Rien n'interdit `widthFactor: 1.5` : l'enfant fera une fois et demie la largeur disponible et **débordera**. Combiné à `alignment`, cela permet des effets, mais attention au rognage du parent.

---

## 46.35 — `ConstrainedBox` et `BoxConstraints`

`ConstrainedBox` **ajoute** des contraintes à celles déjà reçues.

```dart
ConstrainedBox({
  Key? key,
  required BoxConstraints constraints,
  Widget? child,
})
```

```dart
BoxConstraints({
  double minWidth = 0.0,
  double maxWidth = double.infinity,
  double minHeight = 0.0,
  double maxHeight = double.infinity,
})
```

---

### 46.35.1 — Exemple : une largeur minimale et maximale

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _carte(String texte) {
    return Container(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: ConstrainedBox(
        constraints: const BoxConstraints(minWidth: 120, maxWidth: 260),
        child: Container(
          padding: const EdgeInsets.all(12),
          color: Colors.lightGreen.shade200,
          child: Text(texte),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('ConstrainedBox')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              _carte('Court'),
              _carte('Un texte de longueur moyenne pour voir le maximum'),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  ConstrainedBox                          │
├──────────────────────────────────────────┤
│           ┌────────────┐                 │
│           │ Court      │                 │  120 (le minimum)
│           └────────────┘                 │
│      ┌──────────────────────────┐        │
│      │ Un texte de longueur     │        │  260 (le maximum),
│      │ moyenne pour voir le max │        │  le texte passe à la ligne
│      └──────────────────────────┘        │
└──────────────────────────────────────────┘
```

---

### 46.35.2 — Le piège : `ConstrainedBox` ne peut qu'ajouter

`ConstrainedBox` ne **remplace** pas la contrainte du parent : il la **restreint encore**.

```text
   Contrainte du parent :         200 .. 200   (serrée)
   ConstrainedBox(maxWidth: 100)

   Contrainte finale :            200 .. 200
                                  ^^^^^^^^^^
                          le ConstrainedBox est SANS EFFET :
                          il ne peut pas descendre sous le min du parent
```

C'est la raison n° 1 pour laquelle « mon `ConstrainedBox` ne fait rien ». Vérifiez ce que le parent impose. Si le parent est un `Row`, ajoutez d'abord un `Align` ou un `Center` pour desserrer la contrainte.

---

### 46.35.3 — Les constructeurs raccourcis de `BoxConstraints`

| Écriture | Signification |
| --- | --- |
| `BoxConstraints.tight(Size(100, 50))` | exactement 100 x 50 |
| `BoxConstraints.tightFor(width: 100)` | largeur exacte, hauteur libre |
| `BoxConstraints.loose(Size(100, 50))` | de 0 x 0 à 100 x 50 |
| `BoxConstraints.expand()` | le maximum dans les deux sens |
| `BoxConstraints.tightForFinite(width: 100)` | serrée si la valeur est finie |

---

### 46.35.4 — `UnconstrainedBox` et `LimitedBox`

Deux voisins utiles :

- `UnconstrainedBox` **retire** la contrainte de son enfant : l'enfant prend sa taille naturelle, quitte à déborder (Flutter affiche alors un avertissement de débordement) ;
- `LimitedBox` n'applique ses limites que si la contrainte reçue est **non bornée**. C'est exactement ce que fait `Container` en interne (46.11.2).

```dart
// Dans un ListView (hauteur non bornée), limite la hauteur à 200.
LimitedBox(maxHeight: 200, child: monWidget)
```

---

## 46.36 — `IntrinsicHeight` et son coût

`IntrinsicHeight` force tous les enfants d'un `Row` à avoir la **même hauteur** : celle du plus grand.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('IntrinsicHeight')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: IntrinsicHeight(
            child: Row(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                Container(width: 8, color: Colors.red),
                const SizedBox(width: 12),
                const Expanded(
                  child: Text(
                    'Un texte court.',
                  ),
                ),
                const VerticalDivider(width: 24, thickness: 1),
                const Expanded(
                  child: Text(
                    'Un texte beaucoup plus long qui occupe plusieurs '
                    'lignes et impose donc la hauteur de toute la rangée.',
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  IntrinsicHeight                         │
├──────────────────────────────────────────┤
│  █  Un texte     │  Un texte beaucoup    │
│  █  court.       │  plus long qui occupe │
│  █               │  plusieurs lignes et  │
│  █               │  impose donc la       │
│  █               │  hauteur de toute la  │
│  █               │  rangée.              │
└──────────────────────────────────────────┘
   ^                ^
   la barre rouge   le séparateur vertical
   descend jusqu'en bas, alors qu'on ne
   lui a donné aucune hauteur
```

Sans `IntrinsicHeight`, `CrossAxisAlignment.stretch` étirerait la barre rouge et le séparateur sur **toute la hauteur disponible** de la page, et non sur la seule hauteur du texte. `IntrinsicHeight` mesure d'abord la hauteur naturelle du contenu, puis impose cette hauteur au `Row` : les enfants étirés s'arrêtent exactement au bas du texte le plus long.

---

### 46.36.1 — Le coût

C'est le point important de cette section.

Pour connaître la hauteur intrinsèque, Flutter doit **interroger chaque enfant une fois de plus**, en dehors de la passe normale de layout.

```text
   Layout normal :          1 passe sur chaque enfant       -> O(n)
   Avec IntrinsicHeight :   2 passes (mesure + layout)      -> O(n) x 2

   IntrinsicHeight imbriqué dans un autre IntrinsicHeight :
                            -> O(n²), voire pire
```

La documentation officielle le dit explicitement : ce widget est **relativement coûteux** parce que son coût est « proportionnel à la taille de la passe de layout, dans le meilleur des cas, et exponentiel dans le pire des cas ».

**Règles pratiques :**

- ne mettez jamais un `IntrinsicHeight` dans un élément de `ListView` répété des centaines de fois ;
- n'imbriquez jamais deux `IntrinsicHeight` ;
- si vous connaissez la hauteur, écrivez un `SizedBox(height: ...)` : c'est gratuit.

`IntrinsicWidth` existe symétriquement, avec le même coût, pour égaliser les largeurs dans une `Column`.

---

## 46.37 — `SingleChildScrollView` pour éviter le débordement

Le débordement vertical d'une `Column` est aussi fréquent que le débordement horizontal d'un `Row`, surtout quand le clavier apparaît.

```dart
SingleChildScrollView({
  Key? key,
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  EdgeInsetsGeometry? padding,
  ScrollPhysics? physics,
  ScrollController? controller,
  Widget? child,
})
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('SingleChildScrollView')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              for (int i = 1; i <= 12; i++)
                Container(
                  height: 90,
                  margin: const EdgeInsets.only(bottom: 12),
                  decoration: BoxDecoration(
                    color: Colors.blue.shade100,
                    borderRadius: BorderRadius.circular(10),
                  ),
                  alignment: Alignment.center,
                  child: Text(
                    'Bloc $i',
                    style: const TextStyle(fontSize: 20),
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

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  SingleChildScrollView                   │
├──────────────────────────────────────────┤
│  ┌────────────────────────────────────┐  │
│  │              Bloc 1                │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │  <- on fait défiler
│  │              Bloc 2                │  │     pour voir les
│  └────────────────────────────────────┘  │     blocs 3 à 12
│  ┌────────────────────────────────────┐  │
│  │              Bloc 3                │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

---

### 46.37.1 — Le piège : `Expanded` dans une `Column` scrollable

Dans un `SingleChildScrollView`, la `Column` reçoit une hauteur **non bornée**. Un `Expanded` demande alors « donne-moi une part de l'infini », ce qui n'a pas de sens.

```dart
// NE FONCTIONNE PAS
SingleChildScrollView(
  child: Column(
    children: [
      Expanded(child: Container(color: Colors.red)),
    ],
  ),
)
```

Message :

```text
════════ Exception caught by rendering library ════════════════════
RenderFlex children have non-zero flex but incoming height
constraints are unbounded.

When a column is in a parent that does not provide a finite height
constraint, for example if it is in a vertical scrollable, it will
try to shrink-wrap its children along the vertical axis. Setting a
flex on a child (e.g. using Expanded) indicates that the child is to
expand to fill the remaining space in the vertical direction.
═══════════════════════════════════════════════════════════════════
```

**Corrections possibles :**

| Intention | Solution |
| --- | --- |
| Une hauteur fixe | `SizedBox(height: 300, child: ...)` |
| Occuper au moins l'écran | `ConstrainedBox(constraints: BoxConstraints(minHeight: ...))` + `IntrinsicHeight` |
| Une vraie liste | Utiliser `ListView` (chapitre 48) plutôt qu'une `Column` |

La solution « occuper au moins l'écran, et défiler si le contenu dépasse » s'écrit ainsi :

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Scroll + Expanded')),
        body: LayoutBuilder(
          builder: (context, constraints) {
            return SingleChildScrollView(
              child: ConstrainedBox(
                constraints: BoxConstraints(
                  minHeight: constraints.maxHeight,
                ),
                child: IntrinsicHeight(
                  child: Column(
                    children: [
                      Container(height: 120, color: Colors.orange),
                      // Prend le reste QUAND il y en a.
                      Expanded(child: Container(color: Colors.yellow)),
                      Container(height: 80, color: Colors.deepOrange),
                    ],
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

`LayoutBuilder` donne accès aux contraintes reçues : nous les verrons plus en détail au chapitre 51.

---

### 46.37.2 — `SingleChildScrollView` ou `ListView` ?

| Critère | `SingleChildScrollView` | `ListView` |
| --- | --- | --- |
| Nombre d'éléments | petit et connu | grand ou inconnu |
| Construction | tout, tout de suite | à la demande (`.builder`) |
| Performance sur 1 000 éléments | mauvaise | bonne |
| Contenu hétérogène (formulaire) | idéal | possible mais moins pratique |

> Règle : un formulaire ou une page de détail -> `SingleChildScrollView`. Une liste d'éléments répétés -> `ListView` (chapitre 48).

---

## 46.38 — `SafeArea`

`SafeArea` ajoute automatiquement le padding nécessaire pour éviter les zones occupées par le système : encoche, barre d'état, barre de navigation, coins arrondis.

```dart
SafeArea({
  Key? key,
  bool left = true,
  bool top = true,
  bool right = true,
  bool bottom = true,
  EdgeInsets minimum = EdgeInsets.zero,
  bool maintainBottomViewPadding = false,
  required Widget child,
})
```

```text
   SANS SafeArea                     AVEC SafeArea
   ┌──────────▁▁▁▁──────────┐        ┌──────────▁▁▁▁──────────┐
   │ Mon titr█████ohé       │        │▒▒▒▒▒▒▒▒▒▒████▒▒▒▒▒▒▒▒▒▒│
   │          ▔▔▔▔          │        │▒▒▒▒▒▒▒▒▒▒▔▔▔▔▒▒▒▒▒▒▒▒▒▒│
   │                        │        │  Mon titre lisible     │
   │                        │        │                        │
   │                        │        │                        │
   │  contenu               │        │  contenu               │
   │                        │        │                        │
   │▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬│        │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
   └────────────────────────┘        └────────────────────────┘
     le texte passe SOUS               ▒ = zone réservée,
     l'encoche et la barre               le contenu commence après
```

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.indigo,
        body: SafeArea(
          child: Container(
            color: Colors.white,
            width: double.infinity,
            child: const Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Ce texte n\'est jamais sous l\'encoche.'),
                Spacer(),
                Text('Ni au-dessus de la barre de navigation.'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

---

### 46.38.1 — Quand en avez-vous besoin ?

| Situation | `SafeArea` nécessaire ? |
| --- | --- |
| `Scaffold` avec une `AppBar` | Non en haut : l'`AppBar` s'en charge |
| `Scaffold` sans `AppBar` | **Oui** |
| Contenu en plein écran (image de fond) | Oui autour du contenu, non autour de l'image |
| `bottomNavigationBar` | Non : le `Scaffold` gère |
| Une page avec juste un `body` et du texte en haut | **Oui** |

Vous pouvez désactiver certains côtés :

```dart
SafeArea(
  top: true,
  bottom: false,   // le contenu peut aller jusqu'en bas
  child: ...,
)
```

> `MediaQuery.of(context).padding` donne les valeurs brutes de ces zones si vous voulez les gérer vous-même. Nous y reviendrons au chapitre 51.

---

## 46.39 — Déboguer une mise en page

Trois outils, du plus simple au plus complet.

---

### 46.39.1 — Outil 1 : la couleur temporaire

Le plus rapide et le plus efficace. Entourez le widget suspect d'un `ColoredBox` voyant.

```dart
ColoredBox(
  color: Colors.red.withValues(alpha: 0.3),
  child: monWidgetSuspect,
)
```

Vous voyez immédiatement sa taille réelle. Si vous ne voyez rien, le widget fait 0 x 0 : c'est le diagnostic.

---

### 46.39.2 — Outil 2 : `debugPaintSizeEnabled`

Flutter peut dessiner les contours de **toutes** les boîtes.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';   // <- indispensable

void main() {
  debugPaintSizeEnabled = true;   // <- AVANT runApp
  runApp(const MonApp());
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('debugPaintSizeEnabled')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              Container(height: 60, color: Colors.red.shade200),
              const SizedBox(height: 16),
              const Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [Text('Gauche'), Text('Droite')],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│  debugPaintSizeEnabled                   │
├──────────────────────────────────────────┤
│ ╔══════════════════════════════════════╗ │
│ ║ ← ← ← ← padding hachuré → → → → →    ║ │
│ ║ ┌──────────────────────────────────┐ ║ │
│ ║ │ Container rouge                  │ ║ │  bords bleu clair
│ ║ └──────────────────────────────────┘ ║ │
│ ║ ┌──────────────────────────────────┐ ║ │
│ ║ │Gauche  →→→ espace →→→     Droite │ ║ │  flèches = espace
│ ║ └──────────────────────────────────┘ ║ │  distribué
│ ╚══════════════════════════════════════╝ │
└──────────────────────────────────────────┘
```

Lecture des couleurs :

| Élément dessiné | Signification |
| --- | --- |
| Rectangle bleu clair | Les limites d'une `RenderBox` |
| Zone hachurée avec des flèches | Un `padding` ou une marge |
| Flèches jaunes | L'espace distribué dans un `Flex` |
| Bandes jaunes et noires | Un débordement |

> N'oubliez pas de repasser la variable à `false` avant de livrer : elle ralentit le rendu et pollue l'écran. Elle n'a de toute façon aucun effet en mode `release`.

Deux variantes utiles :

```dart
debugPaintBaselinesEnabled = true;   // lignes de base du texte
debugPaintPointersEnabled = true;    // zones tactiles touchées
debugRepaintRainbowEnabled = true;   // ce qui est repeint change de couleur
```

---

### 46.39.3 — Outil 3 : l'inspecteur de widgets

C'est l'outil professionnel. Il est intégré à Flutter DevTools.

**Comment l'ouvrir :**

- dans VS Code : palette de commandes, puis `Flutter: Open DevTools`, onglet `Flutter Inspector` ;
- dans Android Studio : onglet `Flutter Inspector` sur le côté droit ;
- en ligne de commande : l'URL de DevTools s'affiche au lancement de `flutter run`.

**Ce qu'il vous montre :**

```text
   ┌── ARBRE DES WIDGETS ────────┬── DÉTAILS ──────────────────┐
   │ MaterialApp                 │  Row                        │
   │ └ Scaffold                  │                             │
   │   └ Column                  │  renderObject: RenderFlex   │
   │     ├ Container             │  size: Size(400.0, 56.0)    │
   │     └ Row              <──  │  constraints:               │
   │       ├ Text                │    BoxConstraints(          │
   │       └ Icon                │      w=400.0,               │
   │                             │      0.0<=h<=744.0)         │
   │                             │  direction: horizontal      │
   │                             │  mainAxisAlignment:         │
   │                             │    spaceBetween             │
   └─────────────────────────────┴─────────────────────────────┘
```

**Les trois fonctions à connaître :**

1. **Select Widget Mode** : vous cliquez sur un élément de l'application, l'inspecteur sélectionne le widget correspondant dans l'arbre.
2. **Layout Explorer** : sélectionnez un `Row` ou une `Column`, l'explorateur dessine les axes, montre les `flex` de chaque enfant et permet de modifier `mainAxisAlignment` en direct pour tester.
3. **Le panneau de détails** : il affiche `size` et surtout `constraints`. C'est la réponse directe à la question « quelle contrainte ce widget a-t-il reçue ? ».

> Si vous ne deviez retenir qu'un réflexe : quand une mise en page vous échappe, ouvrez l'inspecteur et lisez la ligne `constraints` du widget fautif. Neuf fois sur dix, la réponse est là.

---

### 46.39.4 — La méthode de diagnostic en quatre questions

```text
   Mon widget n'est pas où je veux.
             │
   1. Est-il seulement visible ?
      -> ColoredBox rouge. Rien ? Sa taille est 0.
             │
   2. Quelle contrainte reçoit-il ?
      -> Inspecteur, ligne "constraints".
             │
   3. Qui est son parent direct ?
      -> C'est LUI qui place. Pas le widget lui-même.
             │
   4. Le parent a-t-il de la place à distribuer ?
      -> mainAxisSize: min ? Alors non, il n'y a rien à répartir.
```

---

## 46.40 — Méthode : reconstruire une maquette de haut en bas

Voici la méthode complète, appliquée à une vraie maquette. C'est la synthèse du chapitre.

---

### 46.40.1 — La maquette à reproduire

```text
┌────────────────────────────────────────────┐
│  ←   Fiche de personnage             ⋮     │   AppBar
├────────────────────────────────────────────┤
│                                            │
│   ╭────────╮   Alaric le Sombre            │
│   │        │   Guerrier — Niveau 27        │
│   │   AS   │   ┌──────┐ ┌──────┐           │
│   │        │   │ Élite│ │ Clan │           │
│   ╰────────╯   └──────┘ └──────┘           │
│                                            │
│   Vie                                      │
│   (████████████████████░░░░░░░░)  78 %     │
│                                            │
│   Énergie                                  │
│   (███████████░░░░░░░░░░░░░░░░░)  42 %     │
│                                            │
│  ┌──────────┬──────────┬──────────┐        │
│  │   145    │    38    │   1 204  │        │
│  │  Force   │ Défense  │  Score   │        │
│  └──────────┴──────────┴──────────┘        │
│                                            │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │            ÉQUIPER                   │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

---

### 46.40.2 — Étape 1 : identifier la structure verticale

On ne code rien. On trace des lignes horizontales entre les blocs.

```text
┌────────────────────────────────────────────┐
│  AppBar                                    │  <- Scaffold.appBar
├════════════════════════════════════════════┤
│  BLOC 1 : en-tête (avatar + infos)         │
├════════════════════════════════════════════┤
│  BLOC 2 : les deux jauges                  │
├════════════════════════════════════════════┤
│  BLOC 3 : les trois statistiques           │
├════════════════════════════════════════════┤
│  (espace élastique)                        │  <- Spacer
├════════════════════════════════════════════┤
│  BLOC 4 : le bouton                        │
└────────────────────────────────────────────┘
```

Quatre blocs empilés verticalement, plus un vide élastique.

> **Conclusion de l'étape 1 : la racine est une `Column`.**

---

### 46.40.3 — Étape 2 : décomposer chaque bloc

**Bloc 1** : deux éléments côte à côte -> un `Row`.

```text
   Row
   ├── Container (cercle avatar, 90x90)
   ├── SizedBox(width: 16)
   └── Expanded
       └── Column (crossAxisAlignment: start)
           ├── Text('Alaric le Sombre')
           ├── Text('Guerrier — Niveau 27')
           └── Row
               ├── étiquette 'Élite'
               ├── SizedBox(width: 8)
               └── étiquette 'Clan'
```

**Bloc 2** : deux jauges empilées -> une `Column` de deux sous-blocs identiques.

```text
   Column
   ├── jauge('Vie', 0.78, vert)
   ├── SizedBox(height: 16)
   └── jauge('Énergie', 0.42, bleu)

   et chaque jauge :
   Column (crossAxisAlignment: start)
   ├── Text(label)
   ├── SizedBox(height: 6)
   └── Row
       ├── Expanded -> Container gris -> FractionallySizedBox -> Container coloré
       ├── SizedBox(width: 12)
       └── SizedBox(width: 46) -> Text('78 %')
```

**Bloc 3** : trois colonnes égales -> un `Row` de trois `Expanded`.

**Bloc 4** : un bouton pleine largeur -> `SizedBox(width: double.infinity)`.

---

### 46.40.4 — Étape 3 : nommer les répétitions

Trois motifs se répètent : l'étiquette, la jauge, la statistique. On en fait des méthodes privées (ou des `StatelessWidget` dédiés, comme au chapitre 44).

```text
   _etiquette(String texte)
   _jauge(String label, double valeur, Color couleur)
   _stat(String valeur, String label)
```

---

### 46.40.5 — Étape 4 : écrire le squelette avec des couleurs

Avant d'écrire le contenu réel, on pose les blocs avec des `Container` colorés pour vérifier la structure. C'est l'étape que les débutants sautent, et c'est une erreur.

```dart
// Squelette de vérification
Column(
  children: [
    Container(height: 120, color: Colors.red),      // bloc 1
    Container(height: 140, color: Colors.green),    // bloc 2
    Container(height: 90, color: Colors.blue),      // bloc 3
    const Spacer(),
    Container(height: 56, color: Colors.orange),    // bloc 4
  ],
)
```

Si le squelette est bon, le remplissage ne réserve plus de surprise.

---

### 46.40.6 — Étape 5 : le code final complet

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true),
      home: const FichePersonnage(),
    );
  }
}

class FichePersonnage extends StatelessWidget {
  const FichePersonnage({super.key});

  // --- Motif répété 1 : une étiquette arrondie ---
  Widget _etiquette(String texte, Color couleur) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Text(
        texte,
        style: const TextStyle(color: Colors.white, fontSize: 12),
      ),
    );
  }

  // --- Motif répété 2 : une jauge ---
  Widget _jauge(String label, double valeur, Color couleur) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(label, style: const TextStyle(fontWeight: FontWeight.w600)),
        const SizedBox(height: 6),
        Row(
          children: [
            Expanded(
              child: Container(
                height: 18,
                decoration: BoxDecoration(
                  color: Colors.grey.shade300,
                  borderRadius: BorderRadius.circular(9),
                ),
                child: FractionallySizedBox(
                  widthFactor: valeur,
                  alignment: Alignment.centerLeft,
                  child: Container(
                    decoration: BoxDecoration(
                      color: couleur,
                      borderRadius: BorderRadius.circular(9),
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(width: 12),
            SizedBox(
              width: 46,
              child: Text(
                '${(valeur * 100).round()} %',
                textAlign: TextAlign.right,
              ),
            ),
          ],
        ),
      ],
    );
  }

  // --- Motif répété 3 : une statistique ---
  Widget _stat(String valeur, String label) {
    return Column(
      children: [
        Text(
          valeur,
          style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 4),
        Text(label, style: TextStyle(color: Colors.grey.shade700)),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Fiche de personnage'),
        leading: const BackButton(),
        actions: const [Icon(Icons.more_vert), SizedBox(width: 12)],
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ---------- BLOC 1 : en-tête ----------
            Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Container(
                  width: 90,
                  height: 90,
                  decoration: const BoxDecoration(
                    color: Colors.deepPurple,
                    shape: BoxShape.circle,
                  ),
                  alignment: Alignment.center,
                  child: const Text(
                    'AS',
                    style: TextStyle(color: Colors.white, fontSize: 28),
                  ),
                ),
                const SizedBox(width: 16),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'Alaric le Sombre',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 4),
                      Text(
                        'Guerrier — Niveau 27',
                        style: TextStyle(color: Colors.grey.shade700),
                      ),
                      const SizedBox(height: 10),
                      Row(
                        children: [
                          _etiquette('Élite', Colors.amber.shade800),
                          const SizedBox(width: 8),
                          _etiquette('Clan', Colors.teal),
                        ],
                      ),
                    ],
                  ),
                ),
              ],
            ),

            const SizedBox(height: 28),

            // ---------- BLOC 2 : les jauges ----------
            _jauge('Vie', 0.78, Colors.green),
            const SizedBox(height: 16),
            _jauge('Énergie', 0.42, Colors.blue),

            const SizedBox(height: 28),

            // ---------- BLOC 3 : les statistiques ----------
            Container(
              padding: const EdgeInsets.symmetric(vertical: 16),
              decoration: BoxDecoration(
                color: Colors.grey.shade100,
                borderRadius: BorderRadius.circular(12),
                border: Border.all(color: Colors.grey.shade300),
              ),
              child: Row(
                children: [
                  Expanded(child: _stat('145', 'Force')),
                  Container(width: 1, height: 40, color: Colors.grey.shade300),
                  Expanded(child: _stat('38', 'Défense')),
                  Container(width: 1, height: 40, color: Colors.grey.shade300),
                  Expanded(child: _stat('1 204', 'Score')),
                ],
              ),
            ),

            // ---------- L'espace élastique ----------
            const Spacer(),

            // ---------- BLOC 4 : le bouton ----------
            SizedBox(
              width: double.infinity,
              height: 52,
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('ÉQUIPER', style: TextStyle(fontSize: 16)),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :** la maquette de 46.40.1, à l'identique.

---

### 46.40.7 — La méthode en six règles

| # | Règle |
| --- | --- |
| 1 | Tracez d'abord les lignes **horizontales** : elles donnent la `Column` racine |
| 2 | Dans chaque bande, tracez les lignes **verticales** : elles donnent les `Row` |
| 3 | Un élément qui doit « prendre le reste » -> `Expanded` ; un vide qui pousse -> `Spacer` |
| 4 | Un motif qui apparaît deux fois ou plus -> une méthode ou un widget dédié |
| 5 | Codez le squelette en `Container` colorés AVANT le contenu |
| 6 | Les espacements se font avec `SizedBox`, jamais avec des `Container` vides |

---

## 46.41 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `A RenderFlex overflowed by 137 pixels on the right.` | Les enfants d'un `Row` demandent plus que la largeur disponible | Envelopper l'enfant élastique dans `Expanded` / `Flexible`, ou utiliser `Wrap`, ou `SingleChildScrollView(scrollDirection: Axis.horizontal)` |
| `A RenderFlex overflowed by 42 pixels on the bottom.` | Une `Column` est plus haute que l'écran (souvent quand le clavier s'ouvre) | Envelopper la `Column` dans un `SingleChildScrollView`, ou remplacer par un `ListView` |
| `Incorrect use of ParentDataWidget. The ParentDataWidget Expanded(flex: 1) wants to apply ParentData of type FlexParentData to a RenderObject, which has been set up to accept ParentData of type BoxParentData.` | Un `Expanded` (ou `Flexible`, ou `Spacer`) n'est pas **directement** dans un `Row`/`Column`/`Flex` | Supprimer l'`Expanded`, ou ajouter le `Row`/`Column` manquant. Si un `Padding` s'est glissé entre les deux, inverser : `Expanded(child: Padding(...))` |
| `Incorrect use of ParentDataWidget. ... wants to apply ParentData of type StackParentData ... Typically, Positioned widgets are placed directly inside Stack widgets.` | Un `Positioned` est utilisé hors d'un `Stack` | Ajouter le `Stack`, ou remplacer `Positioned` par `Align`/`Padding` |
| `RenderFlex children have non-zero flex but incoming height constraints are unbounded.` | Un `Expanded`/`Spacer` dans une `Column` placée dans un `SingleChildScrollView` ou un `ListView` | Retirer l'`Expanded`, fixer une hauteur avec `SizedBox`, ou combiner `LayoutBuilder` + `ConstrainedBox(minHeight:)` + `IntrinsicHeight` |
| `Vertical viewport was given unbounded height.` | Un `ListView` (ou `GridView`) placé directement dans une `Column` | Envelopper le `ListView` dans un `Expanded`, ou lui donner une hauteur avec `SizedBox`, ou mettre `shrinkWrap: true` |
| `BoxConstraints forces an infinite height.` | `height: double.infinity` alors que la contrainte reçue est non bornée | Donner une hauteur finie, ou utiliser `Expanded` |
| `BoxConstraints forces an infinite width.` | `width: double.infinity` dans une zone à défilement horizontal | Donner une largeur finie, ou utiliser `Expanded` |
| `RenderBox was not laid out: RenderFlex#3d4e9 NEEDS-LAYOUT NEEDS-PAINT` | Un widget n'a pas pu être mesuré : conséquence d'une erreur de contrainte survenue plus haut | Ignorer ce message et corriger l'**erreur précédente** dans la console : c'est elle la vraie cause |
| `Cannot provide both a color and a decoration. To provide both, use "decoration: BoxDecoration(color: color)".` | `Container(color: ..., decoration: ...)` | Déplacer la couleur dans le `BoxDecoration` |
| `A borderRadius can only be given on boxes with rectangular shapes.` | `BoxDecoration(shape: BoxShape.circle, borderRadius: ...)` | Choisir : soit `shape: BoxShape.circle`, soit `borderRadius` |
| `'left == null \|\| right == null \|\| width == null': is not true.` | `Positioned` avec `left`, `right` et `width` en même temps | N'en donner que deux sur les trois |
| `'textBaseline != null': is not true.` | `CrossAxisAlignment.baseline` sans `textBaseline` | Ajouter `textBaseline: TextBaseline.alphabetic` |
| `Failed assertion: 'child.hasSize'` lors d'un `IntrinsicHeight` | Un enfant refuse de se mesurer (souvent une zone défilante à l'intérieur) | Retirer l'`IntrinsicHeight` ou sortir la zone défilante |
| Le `Container` est **invisible** (aucune erreur) | `Container` sans enfant ni dimensions dans une `Column` : la hauteur non bornée le réduit à 0 | Ajouter `height:`, ou un `child`, ou envelopper dans `Expanded` |
| Le `Container` prend **tout l'écran** (aucune erreur) | `Container` sans enfant ni dimensions, dans une contrainte bornée | Ajouter `width`/`height`, ou un `child` |
| Le `Stack` ne fait rien / est invisible | Tous les enfants sont `Positioned` : le `Stack` n'a plus de taille naturelle | Ajouter un enfant non positionné, ou envelopper le `Stack` dans un `SizedBox`/`Container` dimensionné |
| Un enfant du `Stack` est **coupé** | `clipBehavior: Clip.hardEdge` (défaut) rogne ce qui dépasse | `Stack(clipBehavior: Clip.none, ...)` |
| `mainAxisAlignment` semble sans effet | Le `Flex` est en `MainAxisSize.min`, ou un `Spacer` a déjà mangé tout l'espace | Passer en `MainAxisSize.max`, ou retirer le `Spacer` |
| `crossAxisAlignment: stretch` ne fait rien | Le `Flex` lui-même n'a pas de taille sur l'axe transversal | Donner une taille au parent (`SizedBox`, `Container`, `Expanded`) |
| `ConstrainedBox` semble ignoré | Le parent impose déjà une contrainte serrée | Desserrer d'abord avec `Center`, `Align` ou `UnconstrainedBox` |
| `SizedBox(height: 20)` dans un `Row` ne sépare rien | Dans un `Row`, l'écart se fait sur la **largeur** | Écrire `SizedBox(width: 20)` |
| Le texte passe sous l'encoche du téléphone | Pas de `SafeArea` sur un `Scaffold` sans `AppBar` | Envelopper le `body` dans `SafeArea` |
| Un `Text` long déborde d'un `Row` | Un `Text` demande toute la largeur de son contenu | `Expanded(child: Text(..., overflow: TextOverflow.ellipsis))` |
| `Expanded` dans un `Wrap` : `Incorrect use of ParentDataWidget` | `Wrap` n'est pas un `Flex` | Retirer l'`Expanded` ; utiliser `SizedBox(width:)` pour dimensionner |

---

## 46.42 — Résumé du chapitre

### 46.42.1 — Les widgets de ce chapitre

| Widget | Ce qu'il fait | Quand l'utiliser |
| --- | --- | --- |
| `Container` | Boîte polyvalente : taille, couleur, marges, décoration | Quand il faut au moins deux de ces effets à la fois |
| `Padding` | Ajoute de l'espace autour de l'enfant | Quand seul l'espacement intérieur vous intéresse |
| `SizedBox` | Impose une taille, ou crée un vide | Espacer deux widgets ; forcer la taille d'un bouton |
| `SizedBox.shrink()` | Taille 0 x 0 | Afficher « rien » dans une expression conditionnelle |
| `SizedBox.expand()` | Prend tout l'espace disponible | Remplir une zone entière |
| `Center` | Centre l'enfant | Un contenu unique au milieu de l'écran |
| `Align` | Place l'enfant à une position relative | Coller un élément dans un coin ; placer dans un `Stack` |
| `Column` | Empile verticalement | Une page structurée de haut en bas |
| `Row` | Aligne horizontalement | Une barre, une ligne d'icônes, avatar + texte |
| `MainAxisAlignment` | Répartit sur l'axe principal | Centrer, coller, espacer les enfants |
| `CrossAxisAlignment` | Aligne sur l'axe transversal | Aligner en haut/bas ou étirer |
| `MainAxisSize` | Taille du `Flex` lui-même | `min` pour une carte ou un bouton |
| `Expanded` | Force l'enfant à remplir le reste | Zone de contenu entre un en-tête et un pied |
| `flex` | Nombre de parts dans le partage | Colonnes 1/3 - 2/3 |
| `Flexible` | Autorise à rétrécir sans forcer à grandir | Texte de longueur variable dans un `Row` |
| `Spacer` | Vide élastique | Pousser un élément à l'opposé |
| `Wrap` | Comme un `Row`, mais passe à la ligne | Étiquettes, tags, filtres |
| `Stack` | Superpose les enfants | Badge, voile, texte sur fond coloré |
| `Positioned` | Place précisément dans un `Stack` | Coin supérieur droit d'une carte |
| `Positioned.fill` | Remplit tout le `Stack` | Un voile sombre par-dessus |
| `IndexedStack` | Affiche un seul enfant, garde les autres | Onglets qui conservent leur état |
| `AspectRatio` | Impose un rapport largeur/hauteur | Vignette 16/9, tuile carrée |
| `FractionallySizedBox` | Dimensionne en pourcentage | Barre de vie, jauge, barre de progression |
| `ConstrainedBox` | Ajoute des contraintes min/max | Largeur maximale d'une carte sur tablette |
| `BoxConstraints` | Les quatre nombres de la contrainte | Décrire ce qu'un parent autorise |
| `IntrinsicHeight` | Égalise la hauteur des enfants d'un `Row` | Séparateurs verticaux ; **coûteux** |
| `SingleChildScrollView` | Rend le contenu défilant | Formulaire, page de détail |
| `SafeArea` | Évite encoche et barres système | `Scaffold` sans `AppBar` |
| `BoxDecoration` | Couleur, bordure, coins, ombre, dégradé | Toute carte personnalisée |
| `EdgeInsets` | Décrit quatre marges | Tous les `padding` et `margin` |

---

### 46.42.2 — Les cinq idées à ne jamais oublier

```text
1.  Les contraintes descendent, les tailles remontent, le parent place.

2.  width et height sont des SOUHAITS. La contrainte du parent gagne.

3.  Pour déplacer un widget, on change son PARENT (Center, Align,
    Padding, Positioned), jamais le widget lui-même.

4.  Row  -> main = horizontal, cross = vertical
    Column -> main = vertical,  cross = horizontal

5.  Expanded = contrainte SERRÉE (tu fais cette taille)
    Flexible = contrainte LÂCHE  (tu peux aller jusqu'à cette taille)
```

---

### 46.42.3 — Le mémo des espacements

| Contexte | Comment espacer |
| --- | --- |
| Entre deux enfants d'une `Column` | `SizedBox(height: n)` |
| Entre deux enfants d'un `Row` | `SizedBox(width: n)` |
| Autour du contenu d'une carte | `padding` du `Container`, ou `Padding` |
| Entre plusieurs cartes | `margin`, ou `SizedBox` |
| Espace qui pousse à l'opposé | `Spacer()` |
| Espaces égaux automatiques | `mainAxisAlignment: spaceEvenly` |

---

## 46.43 — Exercices

### Exercice 1 — Trois blocs empilés (facile)

Écrivez un `main.dart` complet qui affiche trois rectangles de 250 x 70, de couleurs différentes, centrés horizontalement et groupés au centre vertical de l'écran, séparés de 20 pixels.

---

### Exercice 2 — Une carte décorée (facile)

Affichez au centre de l'écran une carte de 280 pixels de large contenant, alignés à gauche : un titre en gras de taille 20, un espace de 8, et deux lignes de texte. La carte doit avoir un fond blanc, un coin arrondi de 16, une bordure grise de 1 pixel et une ombre portée.

Le fond de l'écran doit être gris clair pour que l'ombre soit visible.

---

### Exercice 3 — Une barre d'en-tête (facile)

Réalisez une barre horizontale de 60 pixels de haut, de couleur indigo, occupant toute la largeur, contenant :

- à gauche : une icône `Icons.menu` blanche ;
- au centre : rien ;
- à droite : le texte « 1 250 or » en blanc, suivi d'un espace de 8 et de l'icône `Icons.monetization_on` en ambre.

La barre doit être sous une `SafeArea` et avoir 16 pixels de padding horizontal.

---

### Exercice 4 — Corriger un débordement (moyen)

Le code suivant provoque `A RenderFlex overflowed by 260 pixels on the right.`

```dart
Row(
  children: [
    Container(width: 220, height: 120, color: Colors.red),
    Container(width: 220, height: 120, color: Colors.green),
    Container(width: 220, height: 120, color: Colors.blue),
  ],
)
```

Écrivez **trois** versions corrigées dans un seul `main.dart`, empilées dans une `Column`, chacune précédée d'un `Text` qui nomme la technique employée :

1. avec `Expanded` ;
2. avec `SingleChildScrollView` ;
3. avec `Wrap`.

---

### Exercice 5 — Partage proportionnel (moyen)

Réalisez un écran divisé verticalement en trois zones :

- une zone bleue occupant 20 % de la hauteur ;
- une zone blanche occupant 60 % ;
- une zone grise occupant 20 %.

Chaque zone affiche son pourcentage en son centre. Utilisez `Expanded` et `flex`.

---

### Exercice 6 — Reproduire cette maquette (moyen)

```text
┌────────────────────────────────────────────┐
│  Tableau de bord                           │  AppBar
├────────────────────────────────────────────┤
│                                            │
│  ┌──────────────────┐ ┌──────────────────┐ │
│  │                  │ │                  │ │
│  │       12         │ │       47         │ │
│  │    Ennemis       │ │     Pièces       │ │
│  │                  │ │                  │ │
│  └──────────────────┘ └──────────────────┘ │
│                                            │
│  ┌──────────────────┐ ┌──────────────────┐ │
│  │                  │ │                  │ │
│  │        3         │ │      1 204       │ │
│  │     Niveaux      │ │      Score       │ │
│  │                  │ │                  │ │
│  └──────────────────┘ └──────────────────┘ │
│                                            │
└────────────────────────────────────────────┘
```

Contraintes :

- les quatre tuiles sont **carrées** (utilisez `AspectRatio`) ;
- elles sont de taille identique et remplissent la largeur disponible ;
- l'écart entre les tuiles est de 16 pixels, la marge extérieure de 16 également ;
- chaque tuile a un fond coloré clair et des coins arrondis de 12 ;
- n'utilisez pas `GridView` (il sera vu au chapitre 48).

---

### Exercice 7 — Des étiquettes qui passent à la ligne (moyen)

Affichez huit étiquettes arrondies de longueurs très différentes, séparées de 10 pixels horizontalement et de 12 verticalement, qui passent automatiquement à la ligne. Ajoutez un titre au-dessus.

---

### Exercice 8 — Une bulle de notification (moyen)

Affichez au centre de l'écran une icône `Icons.mail` de taille 80, surmontée en haut à droite d'une bulle circulaire rouge contenant le nombre 12 en blanc. La bulle doit **dépasser** légèrement de l'icône.

Ajoutez un bouton qui incrémente le nombre.

---

### Exercice 9 — Reproduire cette maquette (difficile)

```text
┌──────────────────────────────────────────────┐
│  Boutique                                    │  AppBar
├──────────────────────────────────────────────┤
│                                              │
│ ┌──────────────────────────────────────────┐ │
│ │┌────────┐                          ┌───┐ │ │
│ ││        │ Épée de flamme           │ -3│ │ │
│ ││ (icone)│ Arme légendaire          └───┘ │ │
│ ││        │                                │ │
│ │└────────┘ ┌─────┐  150 pièces            │ │
│ │           │ RARE│                        │ │
│ │           └─────┘                        │ │
│ └──────────────────────────────────────────┘ │
│                                              │
│ ┌──────────────────────────────────────────┐ │
│ │┌────────┐                          ┌───┐ │ │
│ ││        │ Potion de soin           │+12│ │ │
│ ││ (icone)│ Consommable              └───┘ │ │
│ ││        │                                │ │
│ │└────────┘ ┌───────┐  25 pièces           │ │
│ │           │COMMUN │                      │ │
│ │           └───────┘                      │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

Contraintes :

- chaque carte est un `Container` blanc, coins arrondis 14, ombre légère, margin vertical 10 ;
- la vignette de gauche est un carré de 70 avec un fond coloré, des coins arrondis 10 et une icône blanche ;
- le badge en haut à droite (`-3` / `+12`) est un petit rectangle arrondi coloré ;
- le nom, le sous-titre, l'étiquette de rareté et le prix sont alignés à gauche ;
- l'étiquette de rareté et le prix sont sur la même ligne.

---

### Exercice 10 — Deux jauges et un total (difficile)

Réalisez un `StatefulWidget` affichant :

- deux jauges horizontales (« Vie » et « Mana ») dessinées avec `FractionallySizedBox` ;
- à droite de chaque jauge, le pourcentage sur une largeur fixe de 50 pixels, aligné à droite ;
- sous les jauges, quatre boutons dans un `Row` en `spaceEvenly` : `- Vie`, `+ Vie`, `- Mana`, `+ Mana`, qui modifient les valeurs par pas de 0,1 en restant entre 0 et 1 ;
- la couleur de la jauge de vie passe au rouge sous 30 %.

---

### Exercice 11 — Reproduire cette maquette (difficile)

```text
┌────────────────────────────────────────────┐
│                                            │
│                                            │
│                                            │
│              PARTIE TERMINÉE               │
│                                            │
│                  1 204                     │
│                  points                    │
│                                            │
│      ┌───────────────────────────────┐     │
│      │  Meilleur score      2 480     │     │
│      ├───────────────────────────────┤     │
│      │  Ennemis vaincus       37      │     │
│      ├───────────────────────────────┤     │
│      │  Temps               04:12     │     │
│      └───────────────────────────────┘     │
│                                            │
│                                            │
│                                            │
│  ┌──────────────┐    ┌──────────────────┐  │
│  │    MENU      │    │     REJOUER      │  │
│  └──────────────┘    └──────────────────┘  │
└────────────────────────────────────────────┘
```

Contraintes :

- fond dégradé du violet foncé (haut) au noir (bas), sur tout l'écran ;
- pas d'`AppBar`, mais une `SafeArea` ;
- les textes sont blancs, le score en taille 48 ;
- le tableau central a un fond blanc translucide (`withValues(alpha: ...)`) et des coins arrondis ;
- chaque ligne du tableau utilise `spaceBetween` ;
- les deux boutons du bas se partagent la largeur : `MENU` avec `flex: 2`, `REJOUER` avec `flex: 3`, séparés de 16 ;
- le bloc central est centré verticalement, les boutons sont collés en bas.

---

### Exercice 12 — Une page défilante complète (difficile)

Assemblez une page de réglages qui :

- utilise une `SafeArea` et un `SingleChildScrollView` ;
- affiche un en-tête (avatar circulaire de 80 + nom + sous-titre) dans un `Row` ;
- puis six « lignes de réglage » identiques, chacune composée d'une icône, d'un titre extensible et d'une icône `Icons.chevron_right`, séparées par un `Divider` ;
- puis un bouton « Déconnexion » pleine largeur ;
- l'ensemble doit défiler correctement même sur un très petit écran.

Le titre de chaque ligne doit être tronqué avec des points de suspension s'il est trop long : testez avec un titre volontairement très long.

---

## 46.44 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 1')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(width: 250, height: 70, color: Colors.red),
              const SizedBox(height: 20),
              Container(width: 250, height: 70, color: Colors.green),
              const SizedBox(height: 20),
              Container(width: 250, height: 70, color: Colors.blue),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** `Center` donne à la `Column` une contrainte lâche, mais la `Column` étant en `MainAxisSize.max` par défaut, elle prend toute la hauteur. C'est donc `mainAxisAlignment: center` qui groupe les trois blocs au milieu vertical, et `crossAxisAlignment: center` (valeur par défaut, écrite ici pour être explicite) qui les centre horizontalement. Les deux `SizedBox(height: 20)` créent les écarts : dans une `Column`, on espace avec `height`.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.grey.shade200,
        body: Center(
          child: Container(
            width: 280,
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(16),
              border: Border.all(color: Colors.grey.shade400, width: 1),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withValues(alpha: 0.18),
                  offset: const Offset(0, 6),
                  blurRadius: 14,
                ),
              ],
            ),
            child: const Column(
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Bouclier du gardien',
                  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                SizedBox(height: 8),
                Text('Défense : +38'),
                Text('Durabilité : 92 / 100'),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**Explication :** trois points sont décisifs. D'abord, `color` est **dans** le `BoxDecoration` : le mettre aussi sur le `Container` lèverait `Cannot provide both a color and a decoration`. Ensuite, `mainAxisSize: MainAxisSize.min` empêche la `Column` de prendre toute la hauteur disponible ; sans lui, la carte s'étirerait sur tout l'écran. Enfin, `crossAxisAlignment: start` aligne les textes à gauche à l'intérieur de la carte : c'est bien `cross` puisque nous sommes dans une `Column`.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: SafeArea(
          child: Column(
            children: [
              Container(
                height: 60,
                width: double.infinity,
                color: Colors.indigo,
                padding: const EdgeInsets.symmetric(horizontal: 16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Icon(Icons.menu, color: Colors.white),
                    Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        const Text(
                          '1 250 or',
                          style: TextStyle(color: Colors.white, fontSize: 16),
                        ),
                        const SizedBox(width: 8),
                        Icon(
                          Icons.monetization_on,
                          color: Colors.amber.shade400,
                        ),
                      ],
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

**Explication :** la barre est un `Row` en `spaceBetween` avec exactement **deux** enfants : l'icône de menu et un groupe. Le groupe de droite est lui-même un `Row` en `MainAxisSize.min` : sans ce `min`, il essaierait d'occuper tout l'espace restant et le `spaceBetween` du parent n'aurait plus rien à répartir. La `SafeArea` évite que la barre passe sous la barre d'état. `width: double.infinity` force la barre à prendre toute la largeur, car sans lui le `Container` prendrait la largeur de son enfant.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _titre(String t) => Padding(
        padding: const EdgeInsets.fromLTRB(12, 16, 12, 8),
        child: Text(t, style: const TextStyle(fontWeight: FontWeight.bold)),
      );

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 4')),
        body: SingleChildScrollView(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // ---- 1. Expanded : les enfants rétrécissent ----
              _titre('1. Expanded'),
              Row(
                children: [
                  Expanded(child: Container(height: 120, color: Colors.red)),
                  Expanded(child: Container(height: 120, color: Colors.green)),
                  Expanded(child: Container(height: 120, color: Colors.blue)),
                ],
              ),

              // ---- 2. SingleChildScrollView : on fait défiler ----
              _titre('2. SingleChildScrollView horizontal'),
              SingleChildScrollView(
                scrollDirection: Axis.horizontal,
                child: Row(
                  children: [
                    Container(width: 220, height: 120, color: Colors.red),
                    Container(width: 220, height: 120, color: Colors.green),
                    Container(width: 220, height: 120, color: Colors.blue),
                  ],
                ),
              ),

              // ---- 3. Wrap : on passe à la ligne ----
              _titre('3. Wrap'),
              Wrap(
                children: [
                  Container(width: 220, height: 120, color: Colors.red),
                  Container(width: 220, height: 120, color: Colors.green),
                  Container(width: 220, height: 120, color: Colors.blue),
                ],
              ),
              const SizedBox(height: 24),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** les trois techniques répondent à trois intentions différentes. `Expanded` supprime le `width: 220` et impose une contrainte serrée d'un tiers de la largeur : les enfants rétrécissent. `SingleChildScrollView(scrollDirection: Axis.horizontal)` donne au `Row` une largeur **non bornée** : plus rien ne déborde puisque la zone visible glisse. `Wrap` mesure les enfants un par un et démarre une nouvelle ligne dès que la place manque. Notez que la page entière est elle-même dans un `SingleChildScrollView` vertical, sans quoi les trois démonstrations empilées déborderaient en bas.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _zone(String texte, Color fond, Color encre, int part) {
    return Expanded(
      flex: part,
      child: Container(
        width: double.infinity,
        color: fond,
        alignment: Alignment.center,
        child: Text(
          texte,
          style: TextStyle(color: encre, fontSize: 24),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Column(
          children: [
            _zone('20 %', Colors.blue, Colors.white, 2),
            _zone('60 %', Colors.white, Colors.black, 6),
            _zone('20 %', Colors.grey.shade600, Colors.white, 2),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** la somme des `flex` vaut 2 + 6 + 2 = 10. Une part vaut donc un dixième de la hauteur, ce qui donne directement 20 %, 60 % et 20 %. Choisir des `flex` dont la somme fait 10 (ou 100) permet de raisonner en pourcentages sans calcul. Le `Container` reçoit d'`Expanded` une contrainte **serrée** en hauteur, il n'a donc pas besoin de `height` ; en revanche il faut `width: double.infinity` pour qu'il occupe toute la largeur, car sur l'axe transversal d'une `Column` la contrainte reste lâche.

---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _tuile(String valeur, String label, Color fond) {
    return AspectRatio(
      aspectRatio: 1,
      child: Container(
        decoration: BoxDecoration(
          color: fond,
          borderRadius: BorderRadius.circular(12),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              valeur,
              style: const TextStyle(
                fontSize: 34,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 6),
            Text(label, style: const TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Tableau de bord')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Expanded(
                    child: _tuile('12', 'Ennemis', Colors.red.shade100),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: _tuile('47', 'Pièces', Colors.amber.shade100),
                  ),
                ],
              ),
              const SizedBox(height: 16),
              Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Expanded(
                    child: _tuile('3', 'Niveaux', Colors.green.shade100),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: _tuile('1 204', 'Score', Colors.blue.shade100),
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

**Explication :** la structure est une `Column` de deux `Row`, chaque `Row` contenant deux `Expanded` séparés par un `SizedBox(width: 16)`. `Expanded` impose à chaque tuile une largeur serrée égale à la moitié de l'espace restant. `AspectRatio(aspectRatio: 1)` calcule alors la hauteur : elle vaut la largeur, la tuile est carrée. Le `crossAxisAlignment: CrossAxisAlignment.start` sur les `Row` explicite l'intention et protège d'un accident : avec `CrossAxisAlignment.stretch`, la contrainte verticale deviendrait serrée et `AspectRatio` ne pourrait plus imposer son rapport. Enfin, `GridView` n'est pas utilisé : quatre tuiles se composent très bien à la main.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _etiquette(String texte, Color couleur) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(20),
      ),
      child: Text(
        texte,
        style: const TextStyle(color: Colors.white, fontSize: 14),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 7')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                'Objets de l\'inventaire',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 16),
              Wrap(
                spacing: 10,
                runSpacing: 12,
                children: [
                  _etiquette('Or', Colors.amber.shade800),
                  _etiquette('Épée courte', Colors.red),
                  _etiquette('Bouclier de fer renforcé', Colors.blueGrey),
                  _etiquette('Potion', Colors.green),
                  _etiquette('Arc', Colors.brown),
                  _etiquette('Grimoire des anciens sorts', Colors.deepPurple),
                  _etiquette('Clé', Colors.teal),
                  _etiquette('Cape d\'invisibilité légendaire', Colors.indigo),
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

**Explication :** `Wrap` remplit une ligne, puis en commence une nouvelle dès qu'une étiquette ne rentre plus. `spacing: 10` sépare les étiquettes **à l'intérieur** d'une ligne, `runSpacing: 12` sépare les **lignes** entre elles : ne confondez pas les deux. La `Column` qui entoure le tout est en `crossAxisAlignment: start` pour que le titre et le bloc d'étiquettes commencent au même bord gauche. Aucun `Expanded` ne doit apparaître ici : `Wrap` n'est pas un `Flex` et lèverait `Incorrect use of ParentDataWidget`.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatefulWidget {
  const MonApp({super.key});

  @override
  State<MonApp> createState() => _MonAppState();
}

class _MonAppState extends State<MonApp> {
  int _messages = 12;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 8')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Stack(
                clipBehavior: Clip.none,
                children: [
                  const Icon(Icons.mail, size: 80, color: Colors.blueGrey),
                  Positioned(
                    top: -6,
                    right: -10,
                    child: Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 8,
                        vertical: 5,
                      ),
                      decoration: BoxDecoration(
                        color: Colors.red,
                        borderRadius: BorderRadius.circular(14),
                      ),
                      constraints: const BoxConstraints(minWidth: 28),
                      child: Text(
                        '$_messages',
                        textAlign: TextAlign.center,
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 13,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                ],
              ),
              const SizedBox(height: 40),
              ElevatedButton(
                onPressed: () => setState(() => _messages++),
                child: const Text('Nouveau message'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** l'icône est le seul enfant **non positionné** : c'est donc elle qui donne sa taille au `Stack` (80 x 80). La bulle est un enfant `Positioned` avec des valeurs **négatives** (`top: -6`, `right: -10`), ce qui la fait sortir du `Stack`. Sans `clipBehavior: Clip.none`, la valeur par défaut `Clip.hardEdge` rognerait la partie qui dépasse. Le `constraints: BoxConstraints(minWidth: 28)` garantit que la bulle reste ronde même avec un seul chiffre, tout en s'élargissant à trois chiffres.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _badge(String texte, Color couleur) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
      decoration: BoxDecoration(
        color: couleur,
        borderRadius: BorderRadius.circular(8),
      ),
      child: Text(
        texte,
        style: const TextStyle(
          color: Colors.white,
          fontSize: 12,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }

  Widget _carte({
    required IconData icone,
    required Color couleur,
    required String nom,
    required String type,
    required String rarete,
    required Color couleurRarete,
    required String prix,
    required String variation,
    required Color couleurVariation,
  }) {
    return Container(
      margin: const EdgeInsets.symmetric(vertical: 10),
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(14),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withValues(alpha: 0.12),
            offset: const Offset(0, 4),
            blurRadius: 10,
          ),
        ],
      ),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // La vignette carrée.
          Container(
            width: 70,
            height: 70,
            decoration: BoxDecoration(
              color: couleur,
              borderRadius: BorderRadius.circular(10),
            ),
            child: Icon(icone, color: Colors.white, size: 36),
          ),
          const SizedBox(width: 14),
          // Toute la partie texte.
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              mainAxisSize: MainAxisSize.min,
              children: [
                Row(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            nom,
                            maxLines: 1,
                            overflow: TextOverflow.ellipsis,
                            style: const TextStyle(
                              fontSize: 17,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          const SizedBox(height: 2),
                          Text(
                            type,
                            style: TextStyle(color: Colors.grey.shade600),
                          ),
                        ],
                      ),
                    ),
                    const SizedBox(width: 8),
                    _badge(variation, couleurVariation),
                  ],
                ),
                const SizedBox(height: 12),
                Row(
                  children: [
                    _badge(rarete, couleurRarete),
                    const SizedBox(width: 12),
                    Text(
                      prix,
                      style: const TextStyle(fontWeight: FontWeight.w600),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Colors.grey.shade100,
        appBar: AppBar(title: const Text('Boutique')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
          child: Column(
            children: [
              _carte(
                icone: Icons.local_fire_department,
                couleur: Colors.deepOrange,
                nom: 'Épée de flamme',
                type: 'Arme légendaire',
                rarete: 'RARE',
                couleurRarete: Colors.purple,
                prix: '150 pièces',
                variation: '-3',
                couleurVariation: Colors.red,
              ),
              _carte(
                icone: Icons.local_drink,
                couleur: Colors.green,
                nom: 'Potion de soin',
                type: 'Consommable',
                rarete: 'COMMUN',
                couleurRarete: Colors.blueGrey,
                prix: '25 pièces',
                variation: '+12',
                couleurVariation: Colors.green,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** la carte se lit de l'extérieur vers l'intérieur. Le `Container` porte le fond, les coins et l'ombre. Son enfant est un `Row` en `crossAxisAlignment: start`, qui colle la vignette et le texte en haut. La vignette a une taille fixe, la partie texte est dans un `Expanded` pour occuper tout le reste : sans lui, le `Text` du nom demanderait sa largeur naturelle et déborderait sur les noms longs. À l'intérieur, un second niveau : un `Row` qui met le bloc nom/type (encore dans un `Expanded`) à gauche et le badge de variation à droite, puis un `Row` qui pose la rareté et le prix côte à côte. Le `maxLines: 1` avec `overflow: TextOverflow.ellipsis` protège définitivement du débordement.

---

### Correction 10

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatefulWidget {
  const MonApp({super.key});

  @override
  State<MonApp> createState() => _MonAppState();
}

class _MonAppState extends State<MonApp> {
  double _vie = 0.8;
  double _mana = 0.45;

  void _modifier(bool estVie, double delta) {
    setState(() {
      if (estVie) {
        _vie = (_vie + delta).clamp(0.0, 1.0);
      } else {
        _mana = (_mana + delta).clamp(0.0, 1.0);
      }
    });
  }

  Widget _jauge(String label, double valeur, Color couleur) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(label, style: const TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 6),
        Row(
          children: [
            Expanded(
              child: Container(
                height: 22,
                decoration: BoxDecoration(
                  color: Colors.grey.shade300,
                  borderRadius: BorderRadius.circular(11),
                ),
                child: FractionallySizedBox(
                  widthFactor: valeur,
                  alignment: Alignment.centerLeft,
                  child: Container(
                    decoration: BoxDecoration(
                      color: couleur,
                      borderRadius: BorderRadius.circular(11),
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(width: 12),
            SizedBox(
              width: 50,
              child: Text(
                '${(valeur * 100).round()} %',
                textAlign: TextAlign.right,
              ),
            ),
          ],
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Exercice 10')),
        body: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              _jauge('Vie', _vie, _vie < 0.3 ? Colors.red : Colors.green),
              const SizedBox(height: 24),
              _jauge('Mana', _mana, Colors.blue),
              const SizedBox(height: 40),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  ElevatedButton(
                    onPressed: () => _modifier(true, -0.1),
                    child: const Text('- Vie'),
                  ),
                  ElevatedButton(
                    onPressed: () => _modifier(true, 0.1),
                    child: const Text('+ Vie'),
                  ),
                  ElevatedButton(
                    onPressed: () => _modifier(false, -0.1),
                    child: const Text('- Mana'),
                  ),
                  ElevatedButton(
                    onPressed: () => _modifier(false, 0.1),
                    child: const Text('+ Mana'),
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

**Explication :** la jauge est faite de deux `Container` superposés par imbrication : le gris donne la piste, l'enfant coloré donne le remplissage. C'est `FractionallySizedBox(widthFactor: valeur)` qui traduit la valeur `0.0`–`1.0` en largeur, et `alignment: Alignment.centerLeft` qui fait partir le remplissage de la gauche (sans lui, il serait centré). La `Row` de la jauge combine un `Expanded` (la piste prend tout le reste) et un `SizedBox(width: 50)` (le pourcentage garde toujours la même largeur, donc la piste ne saute pas quand on passe de 9 % à 100 %). `clamp(0.0, 1.0)`, vu au chapitre 03, empêche de sortir de l'intervalle.

---

### Correction 11

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  Widget _ligne(String label, String valeur, {bool dernier = false}) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 18, vertical: 14),
      decoration: BoxDecoration(
        border: dernier
            ? null
            : Border(
                bottom: BorderSide(
                  color: Colors.white.withValues(alpha: 0.25),
                ),
              ),
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: const TextStyle(color: Colors.white70)),
          Text(
            valeur,
            style: const TextStyle(
              color: Colors.white,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Container(
          decoration: const BoxDecoration(
            gradient: LinearGradient(
              begin: Alignment.topCenter,
              end: Alignment.bottomCenter,
              colors: [Color(0xFF3B1E6E), Colors.black],
            ),
          ),
          child: SafeArea(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                children: [
                  const Spacer(flex: 2),
                  const Text(
                    'PARTIE TERMINÉE',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                      letterSpacing: 2,
                    ),
                  ),
                  const SizedBox(height: 28),
                  const Text(
                    '1 204',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 48,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const Text(
                    'points',
                    style: TextStyle(color: Colors.white70, fontSize: 16),
                  ),
                  const SizedBox(height: 32),
                  Container(
                    decoration: BoxDecoration(
                      color: Colors.white.withValues(alpha: 0.12),
                      borderRadius: BorderRadius.circular(14),
                    ),
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        _ligne('Meilleur score', '2 480'),
                        _ligne('Ennemis vaincus', '37'),
                        _ligne('Temps', '04:12', dernier: true),
                      ],
                    ),
                  ),
                  const Spacer(flex: 3),
                  Row(
                    children: [
                      Expanded(
                        flex: 2,
                        child: SizedBox(
                          height: 52,
                          child: OutlinedButton(
                            onPressed: () {},
                            style: OutlinedButton.styleFrom(
                              foregroundColor: Colors.white,
                              side: const BorderSide(color: Colors.white54),
                            ),
                            child: const Text('MENU'),
                          ),
                        ),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        flex: 3,
                        child: SizedBox(
                          height: 52,
                          child: ElevatedButton(
                            onPressed: () {},
                            style: ElevatedButton.styleFrom(
                              backgroundColor: Colors.deepPurpleAccent,
                              foregroundColor: Colors.white,
                            ),
                            child: const Text('REJOUER'),
                          ),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Explication :** l'ordre d'imbrication est décisif. Le `Container` du dégradé est **au-dessus** de la `SafeArea` : le dégradé couvre donc tout l'écran, encoche comprise, tandis que le contenu, lui, reste dans la zone sûre. Les deux `Spacer` de `flex: 2` et `flex: 3` répartissent le vide : le bloc central est légèrement au-dessus du milieu et les boutons sont poussés vers le bas. Le tableau est une `Column` en `MainAxisSize.min` pour ne pas s'étirer, chaque ligne étant un `Row` en `spaceBetween` avec exactement deux enfants. Enfin, `Expanded(flex: 2)` et `Expanded(flex: 3)` partagent la largeur des boutons dans un rapport 40 / 60.

---

### Correction 12

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true),
      home: const PageReglages(),
    );
  }
}

class PageReglages extends StatelessWidget {
  const PageReglages({super.key});

  Widget _ligne(IconData icone, String titre) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 14),
      child: Row(
        children: [
          Icon(icone, color: Colors.indigo),
          const SizedBox(width: 16),
          Expanded(
            child: Text(
              titre,
              maxLines: 1,
              overflow: TextOverflow.ellipsis,
              style: const TextStyle(fontSize: 16),
            ),
          ),
          const SizedBox(width: 8),
          Icon(Icons.chevron_right, color: Colors.grey.shade500),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(20),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // ---------- En-tête ----------
              Row(
                children: [
                  Container(
                    width: 80,
                    height: 80,
                    decoration: const BoxDecoration(
                      color: Colors.indigo,
                      shape: BoxShape.circle,
                    ),
                    alignment: Alignment.center,
                    child: const Text(
                      'AL',
                      style: TextStyle(color: Colors.white, fontSize: 26),
                    ),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        const Text(
                          'Alaric le Sombre',
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                          style: TextStyle(
                            fontSize: 22,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const SizedBox(height: 4),
                        Text(
                          'Guerrier — Niveau 27',
                          style: TextStyle(color: Colors.grey.shade600),
                        ),
                      ],
                    ),
                  ),
                ],
              ),

              const SizedBox(height: 28),

              // ---------- Les lignes de réglage ----------
              _ligne(Icons.person, 'Profil'),
              const Divider(height: 1),
              _ligne(Icons.notifications, 'Notifications'),
              const Divider(height: 1),
              _ligne(Icons.volume_up, 'Sons et musique'),
              const Divider(height: 1),
              _ligne(
                Icons.language,
                'Langue de l\'interface et des sous-titres du jeu complet',
              ),
              const Divider(height: 1),
              _ligne(Icons.lock, 'Confidentialité'),
              const Divider(height: 1),
              _ligne(Icons.help_outline, 'Aide et support'),

              const SizedBox(height: 32),

              // ---------- Le bouton ----------
              SizedBox(
                width: double.infinity,
                height: 52,
                child: ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.red.shade50,
                    foregroundColor: Colors.red.shade700,
                    elevation: 0,
                  ),
                  child: const Text('Déconnexion'),
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

**Explication :** quatre décisions structurent cette page. `SafeArea` d'abord, parce qu'il n'y a pas d'`AppBar` : sans elle, l'avatar passerait sous la barre d'état. `SingleChildScrollView` ensuite : la `Column` reçoit une hauteur non bornée, elle ne peut donc plus déborder, quelle que soit la taille de l'écran. Notez qu'aucun `Expanded` ni `Spacer` vertical n'apparaît dans cette `Column` : ce serait `RenderFlex children have non-zero flex but incoming height constraints are unbounded.` En revanche, les `Expanded` **horizontaux** à l'intérieur des `Row` sont parfaitement valides, puisque la largeur, elle, reste bornée. Enfin, chaque titre est protégé par `maxLines: 1` et `overflow: TextOverflow.ellipsis` : la quatrième ligne, volontairement très longue, se termine par des points de suspension au lieu de faire déborder le `Row`.

---

## Et maintenant ?

Vous savez désormais placer n'importe quel widget à l'endroit exact que vous voulez, et surtout vous savez **pourquoi** il s'y trouve. La règle « les contraintes descendent, les tailles remontent, le parent place » n'est plus une phrase abstraite : vous l'avez vue à l'œuvre dans `Container`, dans `Expanded`, dans `Stack`, et dans chaque message d'erreur du chapitre.

Il vous manque encore le contenu de ces boîtes. Jusqu'ici, vos écrans étaient faits de rectangles colorés et de textes bruts. Le chapitre suivant s'occupe de ce qui remplit la mise en page : mettre en forme le texte avec `Text` et `TextStyle`, choisir et charger des polices, utiliser la bibliothèque d'icônes de Flutter, afficher des images locales ou distantes, déclarer des assets dans le `pubspec.yaml`, et construire des avatars.

Rendez-vous au chapitre 47 : [47-PARTIE-1B—TEXTE-IMAGES-ICÔNES-ET-ASSETS.md](./47-PARTIE-1B—TEXTE-IMAGES-ICÔNES-ET-ASSETS.md)
