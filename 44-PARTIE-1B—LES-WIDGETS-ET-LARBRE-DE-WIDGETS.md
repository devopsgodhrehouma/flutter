# PARTIE 1B — FLUTTER
# CHAPITRE 44 — LES WIDGETS ET L'ARBRE DE WIDGETS

> **Niveau :** débutant
> **Durée estimée :** 7 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), et chapitre 43 — Installer Flutter et structure d'un projet
> **Ce que vous saurez faire à la fin :** lire et écrire un arbre de widgets Flutter, construire un écran complet avec `MaterialApp`, `Scaffold`, `AppBar`, `Center` et `Text`, comprendre ce qu'est un `BuildContext`, et découper une interface en petits widgets réutilisables.

---

## 44.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer la phrase « en Flutter, tout est widget » ;
- reconnaître qu'un widget est une classe Dart ordinaire, comme celles du chapitre 08 ;
- expliquer pourquoi un widget est une **description** et non un objet dessiné à l'écran ;
- dessiner et lire un arbre de widgets ;
- expliquer pourquoi Flutter préfère la **composition** à l'héritage (rappel du chapitre 11) ;
- utiliser `runApp()` et identifier la racine de l'arbre ;
- utiliser `MaterialApp` et ses paramètres `home`, `title`, `theme`, `debugShowCheckedModeBanner` ;
- utiliser `Scaffold` et ses zones `appBar`, `body`, `floatingActionButton`, `bottomNavigationBar` ;
- écrire un premier écran complet, compilable et lisible ;
- utiliser `Center` et `Text` ;
- lire l'indentation d'un arbre de widgets imbriqués sans vous perdre ;
- expliquer ce qu'est réellement un `BuildContext` ;
- utiliser `Theme.of(context)` et comprendre la remontée dans l'arbre ;
- nommer les trois arbres de Flutter : widgets, éléments, objets de rendu ;
- expliquer pourquoi Flutter peut reconstruire son interface très souvent sans ralentir ;
- placer `const` devant un widget et dire ce que cela change ;
- expliquer à quoi sert une `Key`, brièvement ;
- extraire une portion d'interface dans sa propre classe de widget ;
- expliquer pourquoi extraire dans une méthode est moins bon qu'extraire dans une classe ;
- découper une interface en petits widgets ;
- écrire le constructeur d'un widget avec des paramètres nommés (rappel du chapitre 09) ;
- distinguer `child` et `children` ;
- ouvrir et lire l'inspecteur de widgets de VS Code ;
- lire un message d'erreur de rendu Flutter et retrouver la ligne fautive.

---

## 44.0.1 — Avertissement sur la progression

Ce chapitre est votre **première vraie rencontre avec Flutter**. Nous allons donc volontairement laisser de côté plusieurs notions :

```text
- StatefulWidget et setState()        -> chapitre 45
- Row, Column, Stack, Expanded        -> chapitre 46 (survolés ici)
- TextStyle en détail, Image, assets  -> chapitre 47
- ListView, GridView                  -> chapitre 48
- boutons interactifs et formulaires  -> chapitre 49
- navigation entre écrans             -> chapitre 50
- ThemeData en profondeur             -> chapitre 51
```

Conséquence assumée : **tous les écrans de ce chapitre sont figés**. Rien ne bouge quand on appuie sur un bouton. C'est normal, et c'est voulu : on apprend d'abord à **décrire** une interface. Faire réagir cette interface est le sujet du chapitre 45.

> Chaque bloc `dart` de ce chapitre est un fichier `lib/main.dart` **complet**. Vous pouvez le copier entièrement, écraser votre `lib/main.dart`, et lancer `flutter run`.

---

## 44.1 — En Flutter, tout est widget

C'est la première phrase que l'on entend au sujet de Flutter, et elle est exacte.

Dans la plupart des autres technologies d'interface, on distingue plusieurs familles d'objets :

```text
AUTRES TECHNOLOGIES

  une fenêtre        -> objet de type Window
  un bouton          -> objet de type Button
  une marge          -> une propriété du bouton
  un alignement      -> une propriété du conteneur
  un style de texte  -> une propriété du label
  une animation      -> un objet Animator séparé
```

Chaque chose a sa nature propre. Il faut apprendre plusieurs vocabulaires.

En Flutter, il n'y a **qu'une seule famille** :

```text
FLUTTER

  l'application entière -> un widget  (MaterialApp)
  la structure d'écran  -> un widget  (Scaffold)
  la barre du haut      -> un widget  (AppBar)
  le texte              -> un widget  (Text)
  l'icône               -> un widget  (Icon)
  le centrage           -> un widget  (Center)
  la marge intérieure   -> un widget  (Padding)
  l'espace vide         -> un widget  (SizedBox)
  la couleur de fond    -> un widget  (Container, ColoredBox)
  l'opacité             -> un widget  (Opacity)
  la rotation           -> un widget  (Transform)
  la liste défilante    -> un widget  (ListView)
```

C'est déroutant au début. Un débutant se demande : « comment ça, une marge est un widget ? »

La réponse est : oui, et c'est justement ce qui rend Flutter simple. Il n'y a **qu'un seul concept à apprendre**. Une fois que vous savez utiliser un widget, vous savez utiliser tous les widgets, parce qu'ils fonctionnent tous pareil :

```text
1. on construit un widget avec son constructeur
2. on lui passe des paramètres nommés
3. on l'emboîte dans un autre widget
```

Voici la conséquence directe, à l'écran. Pour ajouter une marge autour d'un texte, on n'écrit pas :

```text
texte.margin = 16       (ceci n'existe pas en Flutter)
```

On **enveloppe** le texte dans un widget de marge :

```dart
Padding(
  padding: EdgeInsets.all(16),
  child: Text('Score : 1200'),
)
```

Retenez cette idée dès maintenant, c'est le geste central de Flutter :

> En Flutter, on ne configure pas un objet avec des propriétés. On **entoure** un widget par un autre widget.

---

## 44.2 — Un widget est une classe Dart (rappel chapitre 08)

Il n'y a aucune magie. Un widget est une **classe Dart**, exactement comme la classe `Player` du chapitre 08.

Rappelez-vous ce que vous avez écrit au chapitre 08 :

```dart
class Player {
  String name = '';
  int health = 100;
}
```

Et au chapitre 09, avec un constructeur à paramètres nommés :

```dart
class Player {
  final String name;
  final int health;

  const Player({required this.name, this.health = 100});
}
```

Un widget Flutter, c'est **exactement la même chose** :

```dart
class ScoreLabel extends StatelessWidget {
  final int score;

  const ScoreLabel({super.key, required this.score});

  @override
  Widget build(BuildContext context) {
    return Text('Score : $score');
  }
}
```

Reconnaissez chaque morceau :

| Morceau de code | Notion vue au chapitre |
| --- | --- |
| `class ScoreLabel` | classe — chapitre 08 |
| `extends StatelessWidget` | héritage — chapitre 10 |
| `final int score;` | propriété immuable — chapitre 08 et 10 |
| `const ScoreLabel({...})` | constructeur à paramètres nommés — chapitre 09 |
| `required this.score` | paramètre nommé obligatoire — chapitre 09 |
| `super.key` | passage au constructeur parent — chapitre 10 |
| `@override` | redéfinition d'une méthode héritée — chapitre 10 |
| `Widget build(...)` | méthode qui retourne une valeur — chapitre 08 |

Vous savez déjà **tout** ce qu'il y a dans ce code. Vous n'apprenez pas un nouveau langage : vous apprenez une **bibliothèque de classes**.

Et on crée un widget comme on crée n'importe quel objet Dart :

```dart
ScoreLabel(score: 1200)
```

> En Dart, le mot-clé `new` n'est plus nécessaire depuis longtemps. `ScoreLabel(score: 1200)` crée bien un objet.

Voici un `main.dart` complet qui utilise cette classe :

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
      title: 'Mon jeu',
      home: Scaffold(
        appBar: AppBar(title: const Text('Statistiques')),
        body: const Center(
          child: ScoreLabel(score: 1200),
        ),
      ),
    );
  }
}

class ScoreLabel extends StatelessWidget {
  final int score;

  const ScoreLabel({super.key, required this.score});

  @override
  Widget build(BuildContext context) {
    return Text('Score : $score');
  }
}
```

**Résultat à l'écran :**

```text
┌─────────────────────────────┐
│  Statistiques               │  <- AppBar
├─────────────────────────────┤
│                             │
│                             │
│       Score : 1200          │  <- Text, centré
│                             │
│                             │
└─────────────────────────────┘
```

---

## 44.3 — Un widget est une description, pas un objet à l'écran

Voici le point que les débutants comprennent le plus tard, et qui explique presque tout le reste du chapitre.

**Un widget n'est pas ce que vous voyez à l'écran.**

Un widget est une **description** de ce que l'écran devrait montrer. C'est une fiche d'instructions. Un plan. Une recette.

Reprenons une image du chapitre 08. La classe était le moule, l'objet la pièce. Ici, il faut ajouter un troisième étage :

```text
   LE WIDGET                  CE QUI EST DESSINÉ
   (une description)          (des pixels sur l'écran)

   ┌──────────────────┐
   │ Text('Score')    │       "Score"
   │  taille 20       │  ==>  en pixels noirs
   │  couleur noire   │       sur fond blanc
   └──────────────────┘
   objet Dart léger,          géré par le moteur
   en mémoire                 de rendu de Flutter
```

Conséquences très concrètes :

**1. Un widget est immuable.** Toutes ses propriétés sont `final`. Une fois créé, il ne change plus jamais. On ne peut pas écrire :

```dart
final texte = Text('Score : 0');
texte.data = 'Score : 100';   // ERREUR : data est final
```

**2. Pour changer l'écran, on jette la description et on en crée une nouvelle.** Flutter compare l'ancienne description et la nouvelle, et ne redessine que la différence. C'est l'objet du chapitre 45.

**3. Créer un widget ne coûte presque rien.** Ce n'est qu'un petit objet Dart avec quelques champs. Flutter en crée des milliers par seconde sans broncher.

Une analogie utile :

```text
Un widget est au pixel affiché
ce qu'une recette de cuisine est au plat.

  - lire la recette ne remplit pas votre assiette ;
  - une recette ne se mange pas ;
  - écrire 100 recettes est rapide, cuisiner 100 plats est long ;
  - si vous changez un ingrédient, vous ne rejetez pas tout le plat :
    vous corrigez uniquement ce qui a changé.
```

C'est exactement le fonctionnement de Flutter. Retenez :

> Vous ne manipulez jamais l'écran. Vous décrivez à quoi il doit ressembler, et Flutter s'occupe du reste.

---

## 44.4 — L'arbre de widgets

Un widget seul ne suffit jamais. Une interface est faite de widgets **emboîtés** les uns dans les autres.

Reprenons le `main.dart` de la section 44.2 et regardons uniquement l'emboîtement :

```dart
MaterialApp(
  home: Scaffold(
    appBar: AppBar(title: Text('Statistiques')),
    body: Center(
      child: ScoreLabel(score: 1200),
    ),
  ),
)
```

Chaque widget contient un ou plusieurs autres widgets. Cela dessine une structure en **arbre** :

```text
                    MaterialApp
                         │
                         │ home
                         ▼
                     Scaffold
                    ╱         ╲
             appBar             body
               ▼                  ▼
            AppBar              Center
               │                  │
               │ title            │ child
               ▼                  ▼
        Text('Statistiques')   ScoreLabel(score: 1200)
                                  │
                                  │ build()
                                  ▼
                           Text('Score : 1200')
```

Vocabulaire, à connaître par cœur :

| Terme | Définition |
| --- | --- |
| **racine** | le widget tout en haut, celui passé à `runApp()` |
| **parent** | le widget qui en contient un autre |
| **enfant** (`child`) | le widget contenu |
| **feuille** | un widget qui ne contient aucun enfant (`Text`, `Icon`) |
| **sous-arbre** | un widget et tout ce qu'il contient |
| **descendant** | un widget situé plus bas dans la même branche |
| **ancêtre** | un widget situé plus haut dans la même branche |

Trois règles absolues de l'arbre Flutter :

```text
1. Il y a exactement UNE racine.
2. Chaque widget a AU PLUS UN parent.
3. Le parent impose des contraintes de taille à l'enfant,
   l'enfant choisit sa taille dans ces contraintes,
   le parent place l'enfant.
```

La troisième règle est le cœur du chapitre 46. Retenez pour l'instant la formule officielle de Flutter :

```text
Les contraintes descendent.
Les tailles remontent.
Le parent positionne.
```

Un arbre un peu plus fourni, pour vous habituer à lire :

```text
MaterialApp
└── Scaffold
    ├── AppBar
    │   ├── Text('Donjon — niveau 3')
    │   └── actions
    │       ├── Icon(Icons.favorite)
    │       └── Icon(Icons.settings)
    ├── body
    │   └── Center
    │       └── Column
    │           ├── Icon(Icons.shield)
    │           ├── Text('Points de vie')
    │           └── Text('87 / 100')
    └── floatingActionButton
        └── FloatingActionButton
            └── Icon(Icons.add)
```

Prenez l'habitude de dessiner cet arbre sur papier avant de coder un écran. C'est le meilleur investissement de temps que vous puissiez faire au début de Flutter.

---

## 44.5 — Composition plutôt qu'héritage : la philosophie de Flutter

Au chapitre 10 et au chapitre 11, vous avez appris l'héritage : une classe `Boss` qui `extends Enemy`, qui `extends Character`. On ajoute des comportements en **descendant** dans une hiérarchie de classes.

C'est puissant, mais cela devient vite ingérable pour une interface. Imaginez une bibliothèque construite par héritage :

```text
                    Widget
                      │
                  ┌───┴────┐
               Bouton    Texte
                  │
        ┌─────────┼──────────┐
   BoutonRouge  BoutonRond  BoutonAvecIcône
        │
   BoutonRougeRond
        │
   BoutonRougeRondAvecIcône
        │
   BoutonRougeRondAvecIcôneEtOmbre        <- et ainsi de suite...
```

C'est l'explosion combinatoire. Chaque nouvelle option double le nombre de classes.

Flutter refuse cette voie. Sa règle est la **composition** :

> Plutôt qu'une grosse classe qui sait tout faire, on emboîte plusieurs petites classes qui savent chacune faire **une seule chose**.

Le même bouton rouge, rond, avec une ombre et une marge, s'écrit ainsi :

```text
Padding            <- s'occupe UNIQUEMENT de la marge
└── DecoratedBox   <- s'occupe UNIQUEMENT du fond, de l'arrondi, de l'ombre
    └── Padding    <- s'occupe UNIQUEMENT de l'espace intérieur
        └── Row    <- s'occupe UNIQUEMENT de mettre l'icône et le texte côte à côte
            ├── Icon
            └── Text
```

Chaque widget est minuscule et n'a **qu'une responsabilité**. On obtient l'effet voulu en les empilant.

Comparons les deux approches :

| Héritage | Composition |
| --- | --- |
| « ce widget **EST** un bouton spécialisé » | « ce widget **CONTIENT** une marge qui contient un fond qui contient un texte » |
| une classe par combinaison | une classe par comportement |
| des dizaines de sous-classes | quelques widgets recombinés à l'infini |
| difficile à faire évoluer | on ajoute ou on retire une couche |

Voici la composition en vrai code, dans un `main.dart` complet :

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
      title: 'Composition',
      home: Scaffold(
        appBar: AppBar(title: const Text('Composition')),
        body: Center(
          // couche 1 : une marge extérieure
          child: Padding(
            padding: const EdgeInsets.all(24),
            // couche 2 : un fond coloré et arrondi
            child: Container(
              decoration: BoxDecoration(
                color: Colors.deepPurple,
                borderRadius: BorderRadius.circular(16),
              ),
              // couche 3 : un espace intérieur
              padding: const EdgeInsets.symmetric(
                horizontal: 32,
                vertical: 20,
              ),
              // couche 4 : le contenu
              child: const Text(
                'ATTAQUER',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
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
┌─────────────────────────────┐
│  Composition                │
├─────────────────────────────┤
│                             │
│      ╭───────────────╮      │
│      │   ATTAQUER    │      │  <- rectangle violet arrondi
│      ╰───────────────╯      │     texte blanc en gras
│                             │
└─────────────────────────────┘
```

Aucune sous-classe n'a été écrite. Nous avons seulement **empilé** des widgets existants.

> Règle à retenir : en Flutter, quand vous vous demandez « comment je fais pour que ce widget ait X ? », la réponse est presque toujours « en l'enveloppant dans un autre widget ».

---

## 44.6 — `runApp()` et la racine de l'arbre

Un programme Dart démarre toujours par `main()`. C'était vrai au chapitre 01, c'est encore vrai ici.

Le fichier `lib/main.dart` minimal d'une application Flutter est :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Text('Bonjour', textDirection: TextDirection.ltr));
}
```

Ce programme fonctionne. Il affiche `Bonjour` en haut à gauche de l'écran, sur fond noir, en texte non stylé. C'est laid, mais c'est une application Flutter valide.

Décomposons.

### La ligne `import`

```dart
import 'package:flutter/material.dart';
```

Elle importe la bibliothèque Material de Flutter. C'est elle qui apporte `runApp`, `MaterialApp`, `Scaffold`, `AppBar`, `Text`, `Colors`, `Icons`, et des centaines d'autres classes.

> Sans cette ligne, aucun widget n'est reconnu et vous obtenez « Undefined name 'MaterialApp' ».

### La fonction `runApp()`

Sa signature est simple :

```dart
void runApp(Widget app)
```

Elle prend **un seul widget** et fait trois choses :

```text
1. elle attache ce widget à l'écran comme RACINE de l'arbre ;
2. elle démarre le moteur Flutter (rendu, gestes, cycle d'images) ;
3. elle demande une première construction de l'interface.
```

Ce qu'il faut retenir :

| Question | Réponse |
| --- | --- |
| Combien de widgets passe-t-on à `runApp()` ? | exactement un |
| Combien de fois appelle-t-on `runApp()` ? | une seule fois, dans `main()` |
| Ce widget occupe quelle place ? | tout l'écran disponible |
| Que fait `runApp()` après ? | rien, elle rend la main ; le moteur prend le relais |

### Pourquoi le texte est-il en haut à gauche et sur fond noir ?

Parce que rien ne lui a dit de faire autrement. Un `Text` seul n'a ni fond, ni centrage, ni style : il n'est **qu'une description de texte**.

C'est précisément le rôle de `MaterialApp` et de `Scaffold` d'apporter tout ce décor. C'est l'objet des sections suivantes.

### La forme normale d'un `main.dart`

En pratique, on n'écrit jamais l'interface directement dans `main()`. On crée une classe racine :

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
      home: Scaffold(
        body: Center(
          child: Text('Bonjour'),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌─────────────────────────────┐
│                             │
│                             │
│         Bonjour             │  <- centré, sur fond blanc
│                             │
│                             │
└─────────────────────────────┘
```

Pourquoi passer par une classe plutôt qu'écrire l'arbre dans `main()` ? Trois raisons :

```text
1. la classe possède une méthode build() : Flutter peut la RECONSTRUIRE ;
2. le hot reload (chapitre 43) ne fonctionne bien qu'avec des classes ;
3. on obtient un BuildContext, indispensable dès la section 44.18.
```

> Convention universelle : la classe racine s'appelle `MyApp` dans les projets générés par `flutter create`. Nous l'appellerons souvent `GameApp` pour rester dans le fil rouge du jeu.

---

## 44.7 — `MaterialApp`

`MaterialApp` est presque toujours le premier widget de votre arbre, juste sous `runApp()`.

Ce n'est pas un widget qui dessine quelque chose. C'est un widget **d'infrastructure** : il installe tout ce dont une application a besoin pour fonctionner.

Ce que `MaterialApp` apporte, sans que vous ayez à l'écrire :

```text
1. le THÈME              -> couleurs, polices, formes, disponibles partout
2. le NAVIGATOR          -> le système d'écrans et de routes (chapitre 50)
3. la DIRECTIONNALITÉ    -> texte de gauche à droite (ou l'inverse)
4. la LOCALISATION       -> langue, formats de date et de nombre
5. le MEDIA QUERY        -> taille de l'écran, densité de pixels (chapitre 51)
6. les OVERLAYS          -> boîtes de dialogue, menus, info-bulles
7. le TITRE de la tâche  -> nom affiché par le système d'exploitation
```

Sans `MaterialApp`, la plupart des widgets Material refusent de fonctionner. Essayez ceci :

```dart
import 'package:flutter/material.dart';

void main() {
  // ATTENTION : ce code PLANTE volontairement.
  runApp(const Scaffold(body: Text('Bonjour')));
}
```

**Résultat dans la console :**

```text
No Directionality widget found.
Text widgets require a Directionality widget ancestor.
```

Traduction : « aucun widget `Directionality` trouvé ; les widgets `Text` ont besoin d'un ancêtre `Directionality` ». Ce widget est fourni par `MaterialApp`. Sans lui, Flutter ne sait pas dans quel sens écrire le texte.

La règle est donc simple :

> Placez toujours un `MaterialApp` au sommet de votre application, et un `Scaffold` en dessous.

L'arbre canonique de départ, à mémoriser :

```text
runApp()
   │
   ▼
MaterialApp          <- infrastructure (thème, navigation, langue)
   │
   │ home
   ▼
Scaffold             <- squelette d'un écran (barre, corps, boutons)
   │
   │ body
   ▼
votre contenu        <- ce que vous écrivez vraiment
```

> Il existe aussi `CupertinoApp` pour un rendu iOS natif, et `WidgetsApp` pour une base minimale sans design. Nous utiliserons `MaterialApp` dans toute cette formation.

---

## 44.8 — `MaterialApp` : `home`, `title`, `theme`, `debugShowCheckedModeBanner`

`MaterialApp` accepte beaucoup de paramètres nommés. Quatre suffisent pour démarrer.

### `home`

C'est le widget affiché au lancement de l'application. C'est le premier écran.

```dart
MaterialApp(
  home: Scaffold(...),
)
```

`home` attend **un widget**, presque toujours un `Scaffold`.

> `home` est un raccourci pour la route `'/'`. Les routes nommées arrivent au chapitre 50.

### `title`

C'est le nom que le **système d'exploitation** affiche pour votre application : dans la liste des tâches récentes sous Android, dans l'onglet du navigateur sur le web.

```dart
MaterialApp(
  title: 'Donjon Infini',
  home: Scaffold(...),
)
```

Attention à ne pas confondre :

| Paramètre | Où le voit-on ? |
| --- | --- |
| `MaterialApp(title: ...)` | dans le gestionnaire de tâches du système, l'onglet du navigateur |
| `AppBar(title: Text(...))` | dans la barre bleue en haut de l'écran |

C'est une confusion très fréquente chez les débutants. `title` sur `MaterialApp` **ne s'affiche pas dans votre interface**.

### `theme`

C'est la palette de couleurs et de styles appliquée à toute l'application. Elle est de type `ThemeData`.

```dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
    useMaterial3: true,
  ),
  home: Scaffold(...),
)
```

`ColorScheme.fromSeed(seedColor: ...)` est la manière moderne de faire : vous donnez **une seule couleur de départ**, et Flutter en dérive une palette complète et cohérente (couleur primaire, secondaire, de surface, d'erreur, couleurs de texte lisibles sur chacune).

`useMaterial3: true` active Material 3, le design actuel de Google. C'est déjà la valeur par défaut dans les versions récentes de Flutter, mais l'écrire rend l'intention explicite.

> Le thème est traité en profondeur au chapitre 51. Ici, retenez qu'il se déclare une fois, en haut, et qu'il descend dans tout l'arbre.

### `debugShowCheckedModeBanner`

En mode debug, Flutter affiche une bannière rouge « DEBUG » en haut à droite de l'écran :

```text
┌─────────────────────────────┐
│  Mon écran            ╱D    │
│                      ╱EBUG  │  <- bannière rouge en diagonale
├─────────────────────────────┤
```

Pour la masquer :

```dart
MaterialApp(
  debugShowCheckedModeBanner: false,
  home: Scaffold(...),
)
```

Deux précisions importantes :

```text
- la bannière n'apparaît QU'EN mode debug ;
- elle disparaît toute seule dans une version de production
  (flutter build apk --release), même sans ce paramètre.
```

On la masque surtout pour faire des captures d'écran propres.

### Les quatre paramètres réunis

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
      title: 'Donjon Infini',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Donjon Infini')),
        body: const Center(
          child: Text('Appuyez sur JOUER pour commencer'),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌─────────────────────────────┐
│  Donjon Infini              │  <- AppBar teintée de violet clair
├─────────────────────────────┤
│                             │
│  Appuyez sur JOUER pour     │
│       commencer             │
│                             │
└─────────────────────────────┘
      pas de bannière DEBUG
```

---

## 44.9 — `Scaffold`

`Scaffold` signifie « échafaudage ». C'est le **squelette d'un écran** Material.

Il ne dessine presque rien lui-même. Il définit des **emplacements** et sait les positionner correctement les uns par rapport aux autres.

```text
        ANATOMIE D'UN SCAFFOLD

┌───────────────────────────────────┐
│   appBar                          │  <- barre du haut
├───────────────────────────────────┤
│                                   │
│                                   │
│   body                            │  <- tout l'espace restant
│                                   │
│                            ╭───╮  │
│                            │ + │  │  <- floatingActionButton
│                            ╰───╯  │
├───────────────────────────────────┤
│   bottomNavigationBar             │  <- barre du bas
└───────────────────────────────────┘

  drawer      : panneau latéral gauche, caché (chapitre 50)
  endDrawer   : panneau latéral droit, caché
  bottomSheet : panneau qui monte du bas
```

Les paramètres les plus utilisés de `Scaffold` :

| Paramètre | Type attendu | Rôle |
| --- | --- | --- |
| `appBar` | `PreferredSizeWidget?` | barre en haut |
| `body` | `Widget?` | contenu principal |
| `floatingActionButton` | `Widget?` | bouton flottant |
| `bottomNavigationBar` | `Widget?` | barre en bas |
| `drawer` | `Widget?` | menu latéral gauche |
| `endDrawer` | `Widget?` | menu latéral droit |
| `backgroundColor` | `Color?` | couleur de fond de l'écran |
| `bottomSheet` | `Widget?` | panneau persistant en bas |

Tous ces paramètres sont **facultatifs**. Un `Scaffold` vide est parfaitement valide :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        backgroundColor: Colors.black,
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌─────────────────────────────┐
│                             │
│                             │
│   (écran entièrement noir)  │
│                             │
│                             │
└─────────────────────────────┘
```

Ce que `Scaffold` fait pour vous, et que vous n'avez pas à gérer :

```text
1. il évite l'encoche et la barre d'état du téléphone ;
2. il remonte le contenu quand le clavier apparaît ;
3. il place le bouton flottant au bon endroit selon la plateforme ;
4. il coordonne l'AppBar, le corps et la barre du bas ;
5. il fournit un fond opaque à la bonne couleur du thème.
```

> Un `Scaffold` par écran. Si votre application a cinq écrans, vous écrirez cinq `Scaffold`. On n'imbrique pas deux `Scaffold` l'un dans l'autre sans très bonne raison.

---

## 44.10 — `AppBar`

`AppBar` est la barre horizontale en haut de l'écran. Elle sert à trois choses : dire où l'on est, offrir un retour en arrière, et proposer des actions rapides.

```text
┌───────────────────────────────────────┐
│ ←   Inventaire            ♥    ⚙      │
│ ▲       ▲                 ▲    ▲      │
│ │       │                 └────┴─ actions │
│ │       └─ title                        │
│ └─ leading (flèche de retour auto)      │
└───────────────────────────────────────┘
```

Les paramètres les plus utiles :

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `title` | `Widget?` | contenu principal, presque toujours un `Text` |
| `actions` | `List<Widget>?` | widgets alignés à droite |
| `leading` | `Widget?` | widget à gauche du titre |
| `backgroundColor` | `Color?` | couleur de fond de la barre |
| `foregroundColor` | `Color?` | couleur du texte et des icônes |
| `centerTitle` | `bool?` | centrer le titre |
| `elevation` | `double?` | hauteur d'ombre portée |
| `toolbarHeight` | `double?` | hauteur de la barre |

Point crucial : **`title` attend un widget, pas une chaîne**.

```dart
AppBar(title: 'Inventaire')          // ERREUR de compilation
AppBar(title: const Text('Inventaire'))  // CORRECT
```

C'est l'erreur numéro un du débutant en Flutter. La raison est cohérente avec la section 44.1 : tout est widget, y compris le titre. Ce choix permet de mettre autre chose qu'un texte — une image, une barre de recherche, une ligne avec une icône.

Exemple complet avec titre centré, couleurs et actions :

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
        appBar: AppBar(
          title: const Text('Inventaire'),
          centerTitle: true,
          backgroundColor: Colors.deepPurple,
          foregroundColor: Colors.white,
          elevation: 4,
          actions: const [
            Icon(Icons.favorite),
            SizedBox(width: 16),
            Icon(Icons.settings),
            SizedBox(width: 16),
          ],
        ),
        body: const Center(
          child: Text('Votre sac est vide.'),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│        Inventaire         ♥    ⚙      │  <- fond violet, texte blanc
├───────────────────────────────────────┤
│                                       │
│         Votre sac est vide.           │
│                                       │
└───────────────────────────────────────┘
```

> `SizedBox(width: 16)` insère un espace de 16 pixels logiques. Il n'affiche rien : c'est un widget d'espacement. Nous le reverrons au chapitre 46.

Remarquez aussi que `actions` attend une **liste** de widgets (`List<Widget>`), alors que `title` attend **un seul** widget. Cette distinction est le sujet de la section 44.28.

---

## 44.11 — `body`

`body` est le paramètre le plus important de `Scaffold` : c'est **votre écran**. Tout ce que vous construisez de spécifique à votre application se trouve là.

```dart
Scaffold(
  appBar: AppBar(title: const Text('Écran')),
  body: /* votre contenu ici */,
)
```

Trois faits à connaître sur `body` :

**1. `body` reçoit exactement un widget.** Pas deux, pas une liste. Si vous voulez plusieurs éléments, vous devez les regrouper dans un widget conteneur — `Column`, `Row`, `Stack`, `ListView`. C'est le rôle du chapitre 46.

```dart
body: Text('A'), Text('B'),    // ERREUR de syntaxe
body: Column(children: [Text('A'), Text('B')]),  // CORRECT
```

**2. `body` occupe tout l'espace restant**, une fois l'`AppBar` et la `bottomNavigationBar` retirées. Il n'a pas de taille fixe : il s'adapte à l'écran.

**3. `body` ne défile pas tout seul.** Si votre contenu dépasse la hauteur disponible, vous obtenez le fameux bandeau jaune et noir de débordement. La solution est un widget de défilement (`SingleChildScrollView`, `ListView`), vu au chapitre 48.

Un `body` simple, entièrement coloré, pour bien voir l'espace qu'il occupe :

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
        appBar: AppBar(
          title: const Text('Zone body'),
          backgroundColor: Colors.indigo,
          foregroundColor: Colors.white,
        ),
        body: Container(
          color: Colors.amber,
          child: const Center(
            child: Text(
              'Tout le jaune, c\'est le body',
              style: TextStyle(fontSize: 20),
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
┌───────────────────────────────────────┐
│  Zone body                            │  <- bleu indigo
├───────────────────────────────────────┤
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░  Tout le jaune, c'est le body ░░░░│  <- fond jaune
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
└───────────────────────────────────────┘
```

Le `Container` sans dimensions prend toute la place que son parent lui offre. Ici, son parent est `body`, donc il remplit l'écran sous l'`AppBar`.

> Notez l'échappement `\'` dans `'Tout le jaune, c\'est le body'`. C'est du Dart pur, vu au chapitre 02. On peut aussi écrire la chaîne avec des guillemets doubles : `"Tout le jaune, c'est le body"`.

---

## 44.12 — `floatingActionButton`

Le bouton d'action flottante (souvent abrégé FAB) est ce cercle coloré posé au-dessus du contenu, en bas à droite.

Sa règle d'usage Material est stricte :

> Un seul FAB par écran, et il déclenche l'action **la plus importante** de cet écran.

```dart
Scaffold(
  floatingActionButton: FloatingActionButton(
    onPressed: () {
      // action ici
    },
    child: const Icon(Icons.add),
  ),
)
```

Le paramètre `onPressed` est **obligatoire**. Il est de type `VoidCallback?`, c'est-à-dire une fonction sans paramètre et sans valeur de retour — exactement les fonctions anonymes du chapitre 07.

```text
onPressed: () { ... }    -> bouton actif
onPressed: null          -> bouton grisé, non cliquable
```

Dans ce chapitre, nos boutons ne peuvent pas encore modifier l'écran : cela demande un `StatefulWidget` (chapitre 45). Nous utiliserons donc `print` pour prouver que le clic fonctionne.

Les variantes de FAB :

| Constructeur | Aspect |
| --- | --- |
| `FloatingActionButton(...)` | cercle standard |
| `FloatingActionButton.small(...)` | cercle plus petit |
| `FloatingActionButton.large(...)` | cercle plus grand |
| `FloatingActionButton.extended(...)` | pilule allongée avec `label` et `icon` |

Et le paramètre `floatingActionButtonLocation` du `Scaffold` permet de le déplacer :

```text
FloatingActionButtonLocation.endFloat        (par défaut, bas droite)
FloatingActionButtonLocation.centerFloat     (bas centre)
FloatingActionButtonLocation.startFloat      (bas gauche)
FloatingActionButtonLocation.centerDocked    (encastré dans la barre du bas)
```

Exemple complet :

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
        appBar: AppBar(
          title: const Text('Sac du héros'),
          backgroundColor: Colors.teal,
          foregroundColor: Colors.white,
        ),
        body: const Center(
          child: Text('3 potions, 1 épée'),
        ),
        floatingActionButton: FloatingActionButton.extended(
          onPressed: () {
            print('Objet ajouté au sac');
          },
          backgroundColor: Colors.teal,
          foregroundColor: Colors.white,
          icon: const Icon(Icons.add),
          label: const Text('Ajouter'),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Sac du héros                         │
├───────────────────────────────────────┤
│                                       │
│        3 potions, 1 épée              │
│                                       │
│                        ╭──────────╮   │
│                        │ + Ajouter│   │
│                        ╰──────────╯   │
└───────────────────────────────────────┘
```

**Résultat dans la console, après un clic :**

```text
Objet ajouté au sac
```

> `print` s'affiche dans le panneau **Debug Console** de VS Code, pas à l'écran du téléphone. C'est l'outil de base pour vérifier qu'un callback est bien appelé.

---

## 44.13 — `bottomNavigationBar`

C'est la barre de navigation en bas de l'écran, qui permet de passer d'une section principale à une autre.

Flutter propose deux widgets pour cela :

| Widget | Design | Usage |
| --- | --- | --- |
| `BottomNavigationBar` | Material 2 | ancien, encore très répandu |
| `NavigationBar` | Material 3 | recommandé aujourd'hui |

### `NavigationBar` (Material 3)

```dart
NavigationBar(
  selectedIndex: 0,
  onDestinationSelected: (int index) {
    print('Onglet $index');
  },
  destinations: const [
    NavigationDestination(icon: Icon(Icons.home), label: 'Accueil'),
    NavigationDestination(icon: Icon(Icons.inventory_2), label: 'Sac'),
    NavigationDestination(icon: Icon(Icons.person), label: 'Héros'),
  ],
)
```

Les trois paramètres essentiels :

```text
destinations          : la liste des onglets (au moins deux)
selectedIndex         : l'index de l'onglet actif (0 = le premier)
onDestinationSelected : la fonction appelée au clic, qui reçoit l'index
```

### `BottomNavigationBar` (Material 2)

```dart
BottomNavigationBar(
  currentIndex: 0,
  onTap: (int index) {
    print('Onglet $index');
  },
  items: const [
    BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Accueil'),
    BottomNavigationBarItem(icon: Icon(Icons.inventory_2), label: 'Sac'),
  ],
)
```

Attention à un piège classique : `BottomNavigationBar` **exige au moins deux `items`**. Avec un seul, l'application plante à l'exécution avec une assertion.

### Exemple complet

Là encore, l'onglet sélectionné ne changera pas au clic : changer `selectedIndex` demande un état, donc le chapitre 45. Nous affichons la preuve du clic dans la console.

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
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Accueil')),
        body: const Center(
          child: Text('Écran d\'accueil'),
        ),
        bottomNavigationBar: NavigationBar(
          selectedIndex: 0,
          onDestinationSelected: (int index) {
            print('Onglet sélectionné : $index');
          },
          destinations: const [
            NavigationDestination(icon: Icon(Icons.home), label: 'Accueil'),
            NavigationDestination(icon: Icon(Icons.inventory_2), label: 'Sac'),
            NavigationDestination(icon: Icon(Icons.person), label: 'Héros'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Accueil                              │
├───────────────────────────────────────┤
│                                       │
│         Écran d'accueil               │
│                                       │
├───────────────────────────────────────┤
│    ⌂          ▣          ☺            │
│ Accueil      Sac       Héros          │
└───────────────────────────────────────┘
```

**Résultat dans la console, après avoir touché « Sac » :**

```text
Onglet sélectionné : 1
```

---

## 44.14 — Un premier écran complet

Réunissons tout ce que nous avons vu depuis le début du chapitre dans un seul écran de jeu.

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
      title: 'Donjon Infini',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Donjon Infini'),
          centerTitle: true,
          backgroundColor: Colors.deepPurple,
          foregroundColor: Colors.white,
          actions: const [
            Icon(Icons.favorite),
            SizedBox(width: 8),
            Text('3'),
            SizedBox(width: 16),
          ],
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: const [
              Icon(
                Icons.shield,
                size: 96,
                color: Colors.deepPurple,
              ),
              SizedBox(height: 24),
              Text(
                'Alex, niveau 7',
                style: TextStyle(
                  fontSize: 28,
                  fontWeight: FontWeight.bold,
                ),
              ),
              SizedBox(height: 8),
              Text(
                'Points de vie : 87 / 100',
                style: TextStyle(fontSize: 18, color: Colors.black54),
              ),
              SizedBox(height: 4),
              Text(
                'Score : 12 450',
                style: TextStyle(fontSize: 18, color: Colors.black54),
              ),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            print('Le combat commence');
          },
          backgroundColor: Colors.deepPurple,
          foregroundColor: Colors.white,
          child: const Icon(Icons.play_arrow),
        ),
        bottomNavigationBar: NavigationBar(
          selectedIndex: 0,
          onDestinationSelected: (int index) {
            print('Onglet $index');
          },
          destinations: const [
            NavigationDestination(icon: Icon(Icons.home), label: 'Accueil'),
            NavigationDestination(icon: Icon(Icons.inventory_2), label: 'Sac'),
            NavigationDestination(icon: Icon(Icons.person), label: 'Héros'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│        Donjon Infini        ♥ 3       │
├───────────────────────────────────────┤
│                                       │
│                 ▲                     │
│                ███                    │  <- Icon(Icons.shield), violet
│                ███                    │
│                                       │
│         Alex, niveau 7                │  <- 28 px, gras
│                                       │
│      Points de vie : 87 / 100         │  <- 18 px, gris
│         Score : 12 450                │
│                                       │
│                            ╭───╮      │
│                            │ ▶ │      │  <- FAB violet
│                            ╰───╯      │
├───────────────────────────────────────┤
│    ⌂          ▣          ☺            │
│ Accueil      Sac       Héros          │
└───────────────────────────────────────┘
```

Et voici l'arbre de widgets correspondant :

```text
GameApp
└── MaterialApp
    └── Scaffold
        ├── appBar: AppBar
        │   ├── title: Text('Donjon Infini')
        │   └── actions: [Icon, SizedBox, Text('3'), SizedBox]
        ├── body: Center
        │   └── Column
        │       ├── Icon(Icons.shield)
        │       ├── SizedBox(height: 24)
        │       ├── Text('Alex, niveau 7')
        │       ├── SizedBox(height: 8)
        │       ├── Text('Points de vie : 87 / 100')
        │       ├── SizedBox(height: 4)
        │       └── Text('Score : 12 450')
        ├── floatingActionButton: FloatingActionButton
        │   └── Icon(Icons.play_arrow)
        └── bottomNavigationBar: NavigationBar
            └── destinations: [NavigationDestination x3]
```

Comptez : plus de vingt widgets pour un écran très simple. C'est normal en Flutter. Un écran réel en contient des centaines. Ne vous laissez pas impressionner : chaque widget fait une seule chose, et vous les lisez de haut en bas.

> `Column` place ses enfants verticalement, `mainAxisAlignment: MainAxisAlignment.center` les centre dans la hauteur disponible. C'est le chapitre 46 ; nous l'utilisons ici uniquement pour empiler quelques lignes.

---

## 44.15 — `BuildContext` : ce que c'est vraiment

Depuis le début du chapitre, vous écrivez cette ligne sans savoir ce qu'elle contient :

```dart
Widget build(BuildContext context) {
```

Il est temps de regarder dedans.

### 44.15.1 — La mauvaise intuition

Beaucoup de débutants croient que `context` est :

```text
- « l'écran »                     -> non
- « l'application »               -> non
- « le contexte Android »         -> non, rien à voir avec Android
- « un truc obligatoire de Flutter que je recopie » -> c'est ce qu'on fait, mais on peut mieux
```

### 44.15.2 — La bonne définition

La documentation officielle de Flutter dit ceci :

> `BuildContext` est **une poignée sur l'emplacement d'un widget dans l'arbre**.

Retenez ce mot : **emplacement**. Le `context` ne décrit pas le widget lui-même. Il décrit **l'endroit où ce widget se trouve** dans l'arbre.

Une comparaison. Imaginez l'arbre de widgets comme un immeuble :

```text
┌──────────────────────────────────────────────┐
│ MaterialApp        <- le hall de l'immeuble  │
│  └── Scaffold      <- étage 1                │
│       └── Center   <- étage 2                │
│            └── Text  <- appartement 2B       │
└──────────────────────────────────────────────┘
```

Le `Text` est le **locataire**. Son `BuildContext`, c'est **son adresse** : « appartement 2B, immeuble Donjon Infini ». Grâce à cette adresse, le locataire peut :

- savoir qui habite au-dessus de lui ;
- demander quelque chose à un voisin du dessus (le thème, la taille de l'écran, la barre de navigation) ;
- se situer dans l'immeuble.

Ce qu'il ne peut pas faire avec son adresse : voir ce qu'il y a en dessous de lui. Un `BuildContext` regarde **vers le haut**, jamais vers le bas.

### 44.15.3 — D'où vient ce `context` ?

Vous ne le créez jamais. C'est Flutter qui vous le donne, en paramètre de `build()` :

```dart
@override
Widget build(BuildContext context) {   // <- Flutter fournit ce context
  return const Text('Bonjour');
}
```

Quand Flutter décide de construire votre widget, il connaît déjà son emplacement dans l'arbre. Il vous passe cet emplacement sous la forme d'un `BuildContext`.

### 44.15.4 — Chaque widget a son propre `context`

C'est le point le plus important, et celui qui produit le plus d'erreurs.

```text
GameApp        -> context A  (au-dessus de MaterialApp)
MaterialApp    -> context B
Scaffold       -> context C
Center         -> context D
Text           -> context E
```

Ce sont **cinq contextes différents**. Ils ne donnent pas accès aux mêmes choses. Le `context A` ne « voit » pas `MaterialApp`, parce que `MaterialApp` est en dessous de lui, pas au-dessus.

Nous verrons en 44.16 que cette subtilité provoque une erreur classique avec `Theme.of(context)`.

### 44.15.5 — Un `BuildContext` est en réalité un `Element`

Sous le capot, un `BuildContext` **est** un objet de la classe `Element`. La documentation le dit explicitement : l'interface `BuildContext` existe pour vous décourager de manipuler directement les `Element`.

Nous parlerons des éléments en 44.17. Pour l'instant, retenez seulement :

| Ce que vous écrivez | Ce que c'est vraiment |
| --- | --- |
| `BuildContext context` | un `Element`, c'est-à-dire un nœud vivant de l'arbre |
| `context.widget` | le widget actuellement associé à ce nœud |

### 44.15.6 — Ce qu'on fait avec un `context`

Trois usages, du plus fréquent au plus rare :

```text
1. Lire une information héritée d'un widget parent
   Theme.of(context)          -> le thème
   MediaQuery.sizeOf(context) -> la taille de l'écran
   Scaffold.of(context)       -> le Scaffold parent

2. Naviguer entre les écrans (chapitre 50)
   Navigator.of(context).push(...)

3. Connaître la taille réelle du widget après le rendu
   context.size               -> rare, et seulement après le layout
```

Un exemple complet avec `MediaQuery.sizeOf(context)`, qui renvoie un objet `Size` :

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
      title: 'BuildContext',
      home: Scaffold(
        appBar: AppBar(title: const Text('Taille de l\'écran')),
        body: const EcranInfo(),
      ),
    );
  }
}

class EcranInfo extends StatelessWidget {
  const EcranInfo({super.key});

  @override
  Widget build(BuildContext context) {
    // Ce context est SOUS le MaterialApp : il voit donc le MediaQuery.
    final Size taille = MediaQuery.sizeOf(context);

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.aspect_ratio, size: 64, color: Colors.deepPurple),
          const SizedBox(height: 16),
          Text('Largeur : ${taille.width.toStringAsFixed(1)} px'),
          Text('Hauteur : ${taille.height.toStringAsFixed(1)} px'),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran (sur un téléphone 411 x 866) :**

```text
┌───────────────────────────────────────┐
│  Taille de l'écran                    │
├───────────────────────────────────────┤
│                                       │
│                 ▭                     │
│                                       │
│        Largeur : 411.0 px             │
│        Hauteur : 866.0 px             │
│                                       │
└───────────────────────────────────────┘
```

> Les valeurs affichées dépendent de votre appareil. Sur un émulateur Pixel, sur Chrome ou sur votre téléphone, elles seront différentes. C'est justement l'intérêt : le widget lit l'information à l'endroit où il se trouve.

### 44.15.7 — La règle d'or du `context`

> Un `context` ne donne accès qu'à ce qui se trouve **au-dessus** de lui dans l'arbre.

Retenez-la. Elle explique à elle seule 90 % des erreurs `No MaterialLocalizations found`, `No Scaffold widget found` et `No MediaQuery widget ancestor could be found` que vous rencontrerez.

---

## 44.16 — `Theme.of(context)` et la remontée dans l'arbre

### 44.16.1 — Le problème que cela résout

Reprenons le code de 44.14. La couleur violette y est écrite **cinq fois** :

```dart
backgroundColor: Colors.deepPurple,
color: Colors.deepPurple,
backgroundColor: Colors.deepPurple,
seedColor: Colors.deepPurple,
// ...
```

Si le client demande du vert, vous devez modifier cinq lignes. Sur une vraie application, ce serait deux cents lignes. C'est exactement le problème que résout le thème : **définir la couleur une seule fois, tout en haut, et la lire partout ailleurs**.

### 44.16.2 — La signature

La méthode officielle est :

```dart
static ThemeData of(BuildContext context)
```

Elle renvoie « les données du `Theme` le plus proche qui englobe le `context` donné ».

Lisez bien : **le plus proche**, et **qui englobe**. Deux mots qui décrivent une remontée.

### 44.16.3 — La remontée, schématisée

Quand vous écrivez `Theme.of(context)` depuis un `Text`, voici ce que Flutter fait :

```text
                              (5) trouvé -> renvoie ce ThemeData
MaterialApp(theme: ThemeData(...))  ◄──────────────┐
    └── Scaffold                                   │ (4) et là ?
         └── Center                                │ (3) et là ?
              └── Column                           │ (2) au-dessus ?
                   └── Text  ── Theme.of(context) ─┘ (1) je pars d'ici
```

Flutter part du `context` fourni, remonte de parent en parent, et **s'arrête au premier `Theme` rencontré**. Il ne descend jamais, et il ne parcourt jamais tout l'arbre : la remontée est directe.

> `MaterialApp` installe automatiquement un `Theme` en haut de votre application, à partir de ce que vous mettez dans son paramètre `theme:`. C'est pour cela que `Theme.of(context)` fonctionne partout sous un `MaterialApp`.

### 44.16.4 — Exemple complet

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
      title: 'Thème',
      debugShowCheckedModeBanner: false,
      // La couleur de base de TOUTE l'application, définie ICI et nulle part ailleurs.
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranHeros(),
    );
  }
}

class EcranHeros extends StatelessWidget {
  const EcranHeros({super.key});

  @override
  Widget build(BuildContext context) {
    // On remonte chercher le thème installé par MaterialApp.
    final ThemeData theme = Theme.of(context);
    final ColorScheme couleurs = theme.colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Fiche du héros'),
        backgroundColor: couleurs.primary,
        foregroundColor: couleurs.onPrimary,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.shield, size: 96, color: couleurs.primary),
            const SizedBox(height: 24),
            Text('Alex', style: theme.textTheme.headlineMedium),
            const SizedBox(height: 8),
            Text('Niveau 7', style: theme.textTheme.titleMedium),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Fiche du héros                       │  <- fond violet, texte clair
├───────────────────────────────────────┤
│                                       │
│                 ▲                     │
│                ███                    │  <- bouclier violet
│                                       │
│              Alex                     │  <- headlineMedium
│            Niveau 7                   │  <- titleMedium
│                                       │
└───────────────────────────────────────┘
```

Changez maintenant **une seule ligne** :

```dart
colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
```

La barre, le bouclier et l'ensemble des nuances passent au vert d'eau. Aucun autre fichier n'a été touché. C'est tout l'intérêt de la remontée.

### 44.16.5 — `colorScheme` : les rôles utiles

`ColorScheme.fromSeed()` fabrique une palette cohérente à partir d'une seule couleur. Les rôles que vous utiliserez le plus au début :

| Rôle | Usage typique |
| --- | --- |
| `primary` | la couleur principale de la marque |
| `onPrimary` | ce qu'on pose **sur** `primary` (texte, icône) |
| `secondary` | une couleur d'accentuation |
| `surface` | le fond des cartes et des feuilles |
| `onSurface` | le texte posé sur `surface` |
| `error` | les erreurs, les alertes |

> La règle de nommage est simple : `onX` est la couleur lisible **par-dessus** `X`. Un texte `onPrimary` sur un fond `primary` est toujours lisible.

### 44.16.6 — `textTheme` : les styles de texte prêts à l'emploi

Plutôt que d'écrire `TextStyle(fontSize: 28, fontWeight: FontWeight.bold)` à la main, le thème vous fournit quinze styles nommés :

```text
displayLarge   displayMedium   displaySmall
headlineLarge  headlineMedium  headlineSmall
titleLarge     titleMedium     titleSmall
bodyLarge      bodyMedium      bodySmall
labelLarge     labelMedium     labelSmall
```

Utilisation :

```dart
Text('Game Over', style: Theme.of(context).textTheme.displaySmall)
```

Avantage : si vous changez la typographie de l'application, tous les titres suivent.

> Ces styles sont `TextStyle?`, c'est-à-dire potentiellement nuls (rappel du chapitre 12). Le paramètre `style:` de `Text` accepte justement un `TextStyle?`, donc l'écriture ci-dessus compile sans point d'interrogation supplémentaire.

### 44.16.7 — L'erreur classique : le `context` trop haut

Voici le piège. Ce code **compile**, mais ne fait pas ce que vous croyez :

```dart
class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: Scaffold(
        appBar: AppBar(
          // PIÈGE : ce `context` est celui de GameApp, donc AU-DESSUS de MaterialApp.
          backgroundColor: Theme.of(context).colorScheme.primary,
          title: const Text('Piège'),
        ),
      ),
    );
  }
}
```

Le `context` utilisé ici est celui de `GameApp`. En remontant, Flutter ne trouve **aucun** `Theme` : le vôtre est plus bas, à l'intérieur de `MaterialApp`. Flutter se rabat alors sur `ThemeData.fallback()`, un thème par défaut bleu clair. Résultat : votre barre n'est pas violette, et aucun message d'erreur ne vous prévient.

Schéma du problème :

```text
GameApp                     <- context ICI. Au-dessus : rien. Aucun Theme.
└── MaterialApp(theme: ...) <- le Theme est ICI, en DESSOUS du context
    └── Scaffold
        └── AppBar          <- il aurait fallu un context d'ici
```

**La correction** : sortir l'écran dans sa propre classe, pour obtenir un `context` situé **sous** `MaterialApp`.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranAccueil(), // <- une classe séparée
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    // Ce context-ci est SOUS MaterialApp : la remontée trouve le bon Theme.
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.primary,
        foregroundColor: Theme.of(context).colorScheme.onPrimary,
        title: const Text('Corrigé'),
      ),
      body: const Center(child: Text('La barre est bien violette')),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Corrigé                              │  <- fond violet
├───────────────────────────────────────┤
│                                       │
│    La barre est bien violette         │
│                                       │
└───────────────────────────────────────┘
```

> Retenez la règle pratique : **ne jamais appeler `Theme.of(context)` dans la même méthode `build()` que celle qui crée le `MaterialApp`.** C'est aussi l'une des meilleures raisons de découper son interface en classes, sujet de la section 44.21.

### 44.16.8 — Les autres `.of(context)`

Le mécanisme n'est pas propre au thème. Vous rencontrerez la même écriture partout :

| Appel | Ce qu'il remonte chercher |
| --- | --- |
| `Theme.of(context)` | le `ThemeData` le plus proche |
| `MediaQuery.of(context)` | les informations de l'écran |
| `MediaQuery.sizeOf(context)` | uniquement la taille de l'écran |
| `Scaffold.of(context)` | le `Scaffold` parent |
| `Navigator.of(context)` | le navigateur d'écrans (chapitre 50) |
| `DefaultTextStyle.of(context)` | le style de texte hérité |

Toutes suivent la même logique : **partir du `context`, remonter, prendre le premier trouvé**.

---

## 44.17 — Les trois arbres : widgets, éléments, objets de rendu

Nous avons dit en 44.3 qu'un widget est une simple description. Une question se pose alors : **si les widgets sont jetables, qui garde la mémoire de ce qui est affiché ?**

La réponse : Flutter n'entretient pas un arbre, mais **trois**.

### 44.17.1 — Le schéma d'ensemble

```text
   ARBRE DE WIDGETS            ARBRE D'ÉLÉMENTS           ARBRE D'OBJETS DE RENDU
   (ce que VOUS écrivez)       (la mémoire de Flutter)    (ce qui mesure et dessine)
   jetable, recréé souvent     stable, conservé           coûteux, réutilisé

   MyApp                       StatelessElement
     │                              │
     ▼                              ▼
   MaterialApp        ───►      StatelessElement
     │                              │
     ▼                              ▼
   Scaffold           ───►      StatelessElement
     │                              │
     ▼                              ▼
   Center             ───►      SingleChildRenderObjectElement ───► RenderPositionedBox
     │                              │                                     │
     ▼                              ▼                                     ▼
   Padding            ───►      SingleChildRenderObjectElement ───► RenderPadding
     │                              │                                     │
     ▼                              ▼                                     ▼
   Text('Score')      ───►      (composant -> RichText)         ───► RenderParagraph
```

Observez trois choses sur ce schéma :

1. **chaque widget a un élément** ; l'élément est créé une fois, et survit aux reconstructions ;
2. **tous les éléments n'ont pas d'objet de rendu** ; `MaterialApp`, `Scaffold`, vos `StatelessWidget` n'en ont pas, ils ne font que composer d'autres widgets ;
3. l'arbre des objets de rendu est donc **plus petit** que les deux autres.

### 44.17.2 — Arbre 1 : les widgets

C'est le seul que vous écrivez.

| Caractéristique | Détail |
| --- | --- |
| Rôle | décrire la configuration voulue |
| Durée de vie | très courte, recréé à chaque `build()` |
| Coût | quasi nul, ce sont de petits objets `final` |
| Mutable ? | non, tous les champs sont `final` |

### 44.17.3 — Arbre 2 : les éléments

C'est l'arbre que Flutter gère seul, et que vous ne créez jamais à la main.

| Caractéristique | Détail |
| --- | --- |
| Rôle | faire le lien entre un widget et son objet de rendu |
| Durée de vie | longue, tant que le widget reste « du même type au même endroit » |
| Ce qu'il retient | la position dans l'arbre, le `State` associé, les dépendances au thème |
| Autre nom | c'est exactement ce que vous manipulez sous le nom de `BuildContext` |

Un élément est un **poste** dans l'organigramme. Le widget est la **fiche de poste** que l'on remplace régulièrement. Le poste, lui, reste.

### 44.17.4 — Arbre 3 : les objets de rendu

| Caractéristique | Détail |
| --- | --- |
| Rôle | mesurer, positionner, peindre, gérer les événements tactiles |
| Durée de vie | longue, réutilisé au maximum |
| Coût | élevé : c'est là que se fait le vrai calcul |
| Classe de base | `RenderObject`, et surtout `RenderBox` pour les rectangles |

### 44.17.5 — Ce qui se passe quand vous appelez `build()` à nouveau

Voici le cœur du mécanisme, étape par étape :

```text
1. Flutter appelle build() -> vous renvoyez un NOUVEL arbre de widgets

2. Flutter compare, position par position, l'ancien widget et le nouveau :

      ancien : Text('Score : 100')      nouveau : Text('Score : 250')
                     │                             │
                     └──────── même type ? ────────┘
                          même clé (Key) ?
                                 │
              ┌──────────────────┴───────────────────┐
             OUI                                    NON
              │                                      │
     l'élément est CONSERVÉ                  l'élément est DÉTRUIT
     on met juste à jour                     un nouvel élément est créé
     l'objet de rendu existant               un nouvel objet de rendu aussi
              │                                      │
        très rapide                             beaucoup plus lent
```

La comparaison est faite par la méthode `Widget.canUpdate(ancien, nouveau)`, qui renvoie `true` si les deux widgets ont **le même type d'exécution** et **la même `Key`**.

### 44.17.6 — Un exemple concret

Supposez que le score passe de 100 à 250. Vous renvoyez un tout nouvel arbre :

```text
AVANT                          APRÈS
Center                         Center
└── Text('Score : 100')        └── Text('Score : 250')
```

Ce que Flutter fait réellement :

```text
Center      : même type, même clé -> élément conservé, RenderPositionedBox conservé
Text        : même type, même clé -> élément conservé, RenderParagraph conservé
                                      la chaîne affichée est simplement remplacée
```

Aucun objet de rendu n'est recréé. Seul le texte à peindre change. C'est pour cela que Flutter peut reconstruire son interface soixante fois par seconde.

### 44.17.7 — Le tableau à retenir

| Arbre | Vous l'écrivez ? | Recréé souvent ? | Coût |
| --- | --- | --- | --- |
| Widgets | oui | oui, à chaque `build()` | très faible |
| Éléments | non | non, conservé et mis à jour | faible |
| Objets de rendu | non | non, réutilisé | élevé |

> Ne cherchez pas à manipuler les deux derniers arbres. Vous n'en aurez pas besoin avant très longtemps. Mais savoir qu'ils existent est indispensable pour comprendre `const`, les `Key`, et le chapitre 45.

---

## 44.18 — Pourquoi Flutter reconstruit vite

La phrase « Flutter recrée tout l'arbre de widgets à chaque image » inquiète toujours les débutants. Voici pourquoi elle n'est pas un problème.

### 44.18.1 — Raison 1 : un widget est un objet minuscule

Regardez ce que contient réellement un `Text` :

```dart
const Text('Score : 250', style: TextStyle(fontSize: 18))
```

Trois champs : une chaîne, un style, quelques valeurs nulles. Créer cet objet coûte quelques nanosecondes. Ce n'est pas un bouton dessiné, ce n'est pas une texture, ce n'est pas une vue Android. C'est une fiche descriptive.

```text
Créer 1 000 widgets  ->  quelques dixièmes de milliseconde
Créer 1 000 vues natives Android  ->  plusieurs secondes, et l'application rame
```

### 44.18.2 — Raison 2 : la comparaison est locale et superficielle

Flutter ne compare pas des arbres entiers. À chaque position, il pose deux questions très bon marché :

```text
runtimeType identique ?   -> une comparaison de type
key identique ?           -> une comparaison de valeur
```

Il n'inspecte pas le contenu en profondeur. Si la réponse est « oui », il conserve l'élément et l'objet de rendu, et se contente de mettre à jour la configuration.

### 44.18.3 — Raison 3 : l'algorithme est linéaire

Un algorithme naïf de comparaison d'arbres coûterait O(n³). Flutter descend à O(n) grâce à deux règles simplificatrices :

```text
Règle 1 : on ne compare jamais des widgets situés à des positions différentes.
          Un widget déplacé est détruit puis recréé (sauf s'il a une Key).

Règle 2 : on ne compare que des widgets de même type.
          Type différent -> on jette et on refait le sous-arbre.
```

Ces règles sont volontairement simplistes. Elles sont ce qui rend la reconstruction rapide.

### 44.18.4 — Raison 4 : le travail coûteux est évité

Le vrai coût d'une interface, ce n'est pas de créer des descriptions. C'est :

| Étape | Coût | Refaite à chaque build ? |
| --- | --- | --- |
| créer les widgets | très faible | oui |
| comparer les widgets | faible | oui |
| mesurer et positionner (*layout*) | moyen à élevé | seulement si nécessaire |
| peindre (*paint*) | élevé | seulement si nécessaire |
| composer les couches | élevé | seulement si nécessaire |

Un objet de rendu qui n'a pas changé n'est ni re-mesuré ni repeint. Il est simplement réutilisé tel quel.

### 44.18.5 — Ce qui rend malgré tout une application lente

Reconstruire n'est pas le problème. Les vrais coupables sont ailleurs :

```text
- un build() qui fait un calcul lourd (boucle sur 100 000 éléments)
- un build() qui lit un fichier ou appelle le réseau       -> jamais dans build()
- une image trop grande redimensionnée à chaque image      -> chapitre 47
- une liste de 10 000 éléments construite d'un coup        -> ListView.builder, chapitre 48
- reconstruire tout l'écran alors qu'un seul texte change  -> chapitres 45 et 52
```

> Règle : `build()` doit être **rapide et sans effet de bord**. Il décrit, il ne calcule pas, il ne télécharge pas, il n'écrit rien sur le disque.

### 44.18.6 — Mesurer plutôt que supposer

Un chiffre de référence : pour que l'application soit fluide à 60 images par seconde, chaque image dispose de **16,7 millisecondes**. À 120 Hz, de **8,3 ms**.

```text
budget par image à 60 fps :  16,7 ms
   dont build + comparaison :  souvent < 1 ms
   dont layout + paint      :  le reste
```

Vous verrez comment mesurer réellement ces temps avec Flutter DevTools au chapitre 42 (« Tests, performance et build »). Pour l'instant, retenez simplement : **reconstruire est bon marché, calculer dans `build()` ne l'est pas**.


---

## 44.19 — Le mot-clé `const` sur un widget et pourquoi il compte

Vous l'écrivez depuis le début du chapitre sans explication :

```dart
const Text('Bonjour')
const SizedBox(height: 16)
const Icon(Icons.shield)
```

Ce n'est ni une décoration, ni une convention de style. C'est une optimisation réelle.

### 44.19.1 — Rappel du chapitre 02 : `const` ne veut pas dire `final`

| Mot-clé | Moment où la valeur est fixée |
| --- | --- |
| `final` | à l'exécution, une seule fois |
| `const` | à la **compilation**, avant même que le programme démarre |

Un objet `const` est fabriqué par le compilateur, écrit une fois pour toutes dans le binaire, et n'est jamais reconstruit.

### 44.19.2 — La canonicalisation

Dart applique une règle très forte : **deux expressions `const` identiques donnent le même objet en mémoire**.

```dart
void main() {
  const a = [1, 2, 3];
  const b = [1, 2, 3];
  print(identical(a, b)); // true : c'est le MÊME objet

  final c = [1, 2, 3];
  final d = [1, 2, 3];
  print(identical(c, d)); // false : deux objets distincts
}
```

**Résultat :**

```text
true
false
```

Appliqué aux widgets : si votre `build()` renvoie `const Text('Score')` et qu'il est appelé mille fois, il n'y aura **jamais** mille objets `Text`. Il y en aura **un seul**, réutilisé mille fois.

### 44.19.3 — Pourquoi Flutter y gagne deux fois

**Gain 1 — aucune allocation.** L'objet existe déjà, on ne l'alloue pas, et le ramasse-miettes n'a rien à nettoyer.

**Gain 2, le plus important — le sous-arbre est court-circuité.** Souvenez-vous de 44.17.5 : Flutter compare l'ancien widget et le nouveau à chaque position. Quand il constate que ce sont **exactement le même objet**, il ne descend même pas dans le sous-arbre. Il conserve l'élément tel quel et passe à la suite.

```text
SANS const                                AVEC const
─────────                                 ──────────
build() -> nouveau Text('Titre')          build() -> LE MÊME Text('Titre')
   │                                          │
   ▼                                          ▼
Flutter compare : même type ?              Flutter compare : objet identique ?
   -> oui, on met à jour l'élément            -> oui, on ARRÊTE ICI
   -> on descend dans les enfants             -> on ne descend pas
   -> on reconfigure le RenderParagraph       -> aucun travail du tout
```

Sur un écran contenant deux cents widgets figés, marquer ces widgets `const` supprime deux cents comparaisons et deux cents reconfigurations à chaque reconstruction.

### 44.19.4 — Exemple complet, avec et sans `const`

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
      title: 'const',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Le mot-clé const')),
        // TOUT ce sous-arbre est figé : un seul const en haut suffit.
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.shield, size: 96, color: Colors.deepPurple),
              SizedBox(height: 24),
              Text(
                'Inventaire',
                style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
              ),
              SizedBox(height: 8),
              Text('3 potions, 1 épée, 12 pièces'),
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
┌───────────────────────────────────────┐
│  Le mot-clé const                     │
├───────────────────────────────────────┤
│                                       │
│                 ▲                     │
│                ███                    │
│                                       │
│           Inventaire                  │
│    3 potions, 1 épée, 12 pièces       │
│                                       │
└───────────────────────────────────────┘
```

Remarquez : le `const` est écrit **une seule fois**, devant `Center`. Tout ce qui est en dessous devient automatiquement constant. C'est pour cela qu'on ne réécrit pas `const` devant chaque `Icon` ou chaque `Text` de la liste : à l'intérieur d'un contexte déjà `const`, c'est inutile, et l'analyseur vous le signalera par la règle `unnecessary_const`.

### 44.19.5 — Quand `const` est impossible

`const` exige que **tout** soit connu à la compilation. Ces cas l'interdisent :

```dart
// 1. Une valeur calculée à l'exécution
Text('Score : $score')            // impossible : score est une variable

// 2. Une fonction anonyme
ElevatedButton(onPressed: () {}, child: const Text('OK'))  // le bouton n'est pas const

// 3. Un objet non constant reçu en paramètre
Icon(monIcone)                    // impossible si monIcone n'est pas const

// 4. Une couleur calculée
Container(color: Color(valeurLue))  // impossible
```

Dans tous ces cas, on met `const` **le plus bas possible** :

```dart
Column(
  children: [
    Text('Score : $score'),        // pas const : dépend d'une variable
    const SizedBox(height: 8),     // const : figé
    const Text('Bonne chance'),    // const : figé
  ],
)
```

### 44.19.6 — Les erreurs classiques autour de `const`

**Erreur A — `const` sur une valeur non constante :**

```dart
final int vies = 3;
const Text('Vies : $vies');   // ERREUR de compilation
```

Message de l'analyseur :

```text
Const variables must be initialized with a constant value.
Invalid constant value.
```

Correction : retirer le `const`.

**Erreur B — oublier `const` sur le constructeur du widget lui-même :**

```dart
class CarteHeros extends StatelessWidget {
  CarteHeros({super.key});      // pas de const -> on ne pourra jamais écrire const CarteHeros()
  // ...
}
```

Correction :

```dart
class CarteHeros extends StatelessWidget {
  const CarteHeros({super.key});   // maintenant const CarteHeros() est possible
  // ...
}
```

> Prenez l'habitude d'écrire `const` devant le constructeur de **tous** vos widgets. Cela ne coûte rien et laisse la porte ouverte à l'optimisation. C'est d'ailleurs pour cela que `flutter create` génère `const MyApp({super.key});`.

**Erreur C — un champ non `final` empêche le constructeur `const` :**

```dart
class CarteHeros extends StatelessWidget {
  String nom;                    // ERREUR : champ modifiable
  const CarteHeros({super.key, required this.nom});
  // ...
}
```

Message :

```text
Can't define a const constructor for a class with non-final fields.
```

Correction : `final String nom;`.

### 44.19.7 — Laisser l'outil travailler pour vous

Le fichier `analysis_options.yaml` (vu au chapitre 43) active deux règles utiles :

```yaml
linter:
  rules:
    - prefer_const_constructors
    - prefer_const_literals_to_create_immutables
```

L'analyseur souligne alors chaque endroit où un `const` manque. Dans VS Code, le raccourci de correction rapide (`Ctrl+.` ou `Cmd+.`) propose « Add 'const' modifier ». Un `dart fix --apply` corrige même tout le projet d'un coup :

```text
Computing fixes in mon_jeu...
Applying fixes...
  lib/main.dart
    prefer_const_constructors  - 12 fixes
12 fixes made in 1 file.
```

### 44.19.8 — Faut-il en mettre partout ?

Oui, chaque fois que c'est possible. Le coût est nul, le gain est réel, et l'habitude vous évitera de chercher plus tard pourquoi une liste de cent lignes est saccadée.

> Mais ne tordez jamais votre code pour rendre quelque chose `const`. Si une valeur dépend de l'exécution, elle ne peut pas être constante, et c'est très bien ainsi.

---

## 44.20 — `Key` : à quoi ça sert, brièvement

Vous avez écrit `super.key` dans chaque constructeur sans savoir ce que c'était. Voici l'essentiel, en version courte : les `Key` deviennent vraiment utiles au chapitre 45 et surtout au chapitre 48.

### 44.20.1 — Le rôle d'une clé

Rappel de 44.17.5 : pour décider s'il conserve un élément, Flutter appelle `Widget.canUpdate()`, qui compare **le type d'exécution** et **la clé**.

```text
même type + même key      -> l'élément et son état sont CONSERVÉS
même type + key différente -> l'élément est DÉTRUIT, un nouveau est créé
```

Une `Key` est donc une **étiquette d'identité** qui permet à Flutter de reconnaître un widget même s'il a changé de place dans une liste.

### 44.20.2 — Quand cela n'a aucune importance

Dans tout ce chapitre, aucune clé n'a été nécessaire, pour deux raisons :

- nos widgets sont figés, ils n'ont pas d'état à préserver ;
- rien n'est réordonné, inséré ou supprimé dans une liste.

C'est le cas de la grande majorité des interfaces. **Ne mettez pas de clés partout « au cas où ».**

### 44.20.3 — Quand cela devient indispensable

Le cas d'école : une liste de widgets qui possèdent un état, que l'on réordonne ou dont on supprime un élément.

```text
SANS clé, après suppression du premier élément

  avant :  [ Gobelin(pv=10) , Orc(pv=30) , Dragon(pv=99) ]
  après  :  [ Orc            , Dragon                    ]

  Flutter compare par POSITION :
    position 0 : Gobelin -> Orc     même type -> il garde l'état pv=10 !
    position 1 : Orc     -> Dragon  même type -> il garde l'état pv=30 !
    position 2 : Dragon  -> rien    supprimé

  Résultat : les barres de vie sont décalées. Bug classique.
```

Avec une clé qui identifie chaque ennemi, Flutter reconnaît que l'`Orc` de la position 0 est bien l'ancien `Orc` de la position 1, et déplace son état avec lui.

### 44.20.4 — Les types de clés

| Type | Utilisation |
| --- | --- |
| `ValueKey<T>` | une valeur simple qui identifie l'objet : `ValueKey(ennemi.id)` |
| `ObjectKey` | l'objet lui-même sert d'identité |
| `UniqueKey` | une clé différente à chaque construction, force la recréation |
| `GlobalKey` | identité unique dans toute l'application, permet d'accéder à un état ailleurs ; coûteuse, à réserver aux cas rares (formulaires, chapitre 49) |

### 44.20.5 — Un exemple minimal, uniquement pour la syntaxe

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    const List<String> ennemis = ['Gobelin', 'Orc', 'Dragon'];

    return MaterialApp(
      title: 'Key',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Les clés')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              for (final String nom in ennemis)
                // Chaque ligne reçoit une identité stable, indépendante de sa position.
                LigneEnnemi(key: ValueKey<String>(nom), nom: nom),
            ],
          ),
        ),
      ),
    );
  }
}

class LigneEnnemi extends StatelessWidget {
  const LigneEnnemi({super.key, required this.nom});

  final String nom;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.bug_report, color: Colors.redAccent),
          const SizedBox(width: 12),
          Text(nom, style: const TextStyle(fontSize: 20)),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Les clés                             │
├───────────────────────────────────────┤
│                                       │
│           ✳  Gobelin                  │
│           ✳  Orc                      │
│           ✳  Dragon                   │
│                                       │
└───────────────────────────────────────┘
```

> Le paramètre `key` est accepté par **tous** les widgets, parce qu'il est déclaré sur la classe `Widget` elle-même. C'est ce que fait `super.key` dans vos constructeurs : transmettre la clé reçue à la classe mère.

### 44.20.6 — Ce qu'il faut retenir pour l'instant

```text
- Une Key est une étiquette d'identité, pas un identifiant de base de données.
- Sans état à préserver, elle ne sert à rien.
- Elle sert quand on RÉORDONNE, INSÈRE ou SUPPRIME des widgets qui ont un état.
- Écrivez toujours `const MonWidget({super.key, ...})` : cela ne coûte rien.
- Le sujet est repris en profondeur aux chapitres 45 et 48.
```

---

## 44.21 — Extraire un widget dans sa propre classe

Voici la compétence la plus rentable de tout ce chapitre.

### 44.21.1 — Le point de départ : un `build()` trop long

Reprenez mentalement le code de 44.14 : plus de cent lignes dans une seule méthode `build()`, avec une indentation qui descend jusqu'à sept niveaux. Ce code fonctionne, mais il est déjà difficile à lire. Ajoutez trois cartes de statistiques et il devient illisible.

Voici une version encore plus parlante du problème, avec trois blocs presque identiques :

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
      title: 'Avant extraction',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Statistiques')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // ---- bloc 1 ----
              Container(
                width: 260,
                padding: const EdgeInsets.all(16),
                margin: const EdgeInsets.symmetric(vertical: 8),
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.favorite, color: Colors.red),
                    const SizedBox(width: 12),
                    const Text('Vies'),
                    const Spacer(),
                    const Text('3', style: TextStyle(fontWeight: FontWeight.bold)),
                  ],
                ),
              ),
              // ---- bloc 2 : le même, avec d'autres valeurs ----
              Container(
                width: 260,
                padding: const EdgeInsets.all(16),
                margin: const EdgeInsets.symmetric(vertical: 8),
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.bolt, color: Colors.orange),
                    const SizedBox(width: 12),
                    const Text('Énergie'),
                    const Spacer(),
                    const Text('87', style: TextStyle(fontWeight: FontWeight.bold)),
                  ],
                ),
              ),
              // ---- bloc 3 : encore le même ----
              Container(
                width: 260,
                padding: const EdgeInsets.all(16),
                margin: const EdgeInsets.symmetric(vertical: 8),
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  children: [
                    const Icon(Icons.star, color: Colors.amber),
                    const SizedBox(width: 12),
                    const Text('Score'),
                    const Spacer(),
                    const Text('12450', style: TextStyle(fontWeight: FontWeight.bold)),
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
┌───────────────────────────────────────┐
│  Statistiques                         │
├───────────────────────────────────────┤
│      ╭─────────────────────────╮      │
│      │ ♥  Vies             3   │      │
│      ╰─────────────────────────╯      │
│      ╭─────────────────────────╮      │
│      │ ⚡ Énergie          87   │      │
│      ╰─────────────────────────╯      │
│      ╭─────────────────────────╮      │
│      │ ★  Score        12450   │      │
│      ╰─────────────────────────╯      │
└───────────────────────────────────────┘
```

Le code marche. Mais il contient trois fois la même chose. Si le client demande des coins plus arrondis, vous devez modifier trois endroits — et vous en oublierez un.

### 44.21.2 — La solution : une classe par bloc réutilisable

On identifie ce qui change entre les trois blocs :

```text
ce qui CHANGE          : l'icône, sa couleur, le libellé, la valeur
ce qui NE CHANGE PAS   : la largeur, les marges, le fond, l'arrondi, la disposition
```

Ce qui change devient des **paramètres**. Ce qui ne change pas devient le **corps du widget**.

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
      title: 'Après extraction',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranStatistiques(),
    );
  }
}

class EcranStatistiques extends StatelessWidget {
  const EcranStatistiques({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Statistiques')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CarteStat(icone: Icons.favorite, couleur: Colors.red, libelle: 'Vies', valeur: '3'),
            CarteStat(icone: Icons.bolt, couleur: Colors.orange, libelle: 'Énergie', valeur: '87'),
            CarteStat(icone: Icons.star, couleur: Colors.amber, libelle: 'Score', valeur: '12450'),
          ],
        ),
      ),
    );
  }
}

/// Une ligne de statistique : une icône, un libellé, une valeur.
class CarteStat extends StatelessWidget {
  const CarteStat({
    super.key,
    required this.icone,
    required this.couleur,
    required this.libelle,
    required this.valeur,
  });

  final IconData icone;
  final Color couleur;
  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 260,
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.symmetric(vertical: 8),
      decoration: BoxDecoration(
        color: Colors.deepPurple.shade50,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: [
          Icon(icone, color: couleur),
          const SizedBox(width: 12),
          Text(libelle),
          const Spacer(),
          Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :** exactement le même que ci-dessus.

Comparons :

| | Avant | Après |
| --- | --- | --- |
| Lignes dans `build()` de l'écran | environ 70 | 12 |
| Description d'une carte | répétée 3 fois | écrite 1 fois |
| Ajouter une 4e carte | copier 20 lignes | ajouter 1 ligne |
| Changer l'arrondi | 3 modifications | 1 modification |
| Testable isolément | non | oui |
| `const` possible sur la liste | non | oui |

### 44.21.3 — Le raccourci de VS Code

Vous n'avez pas besoin d'écrire cette classe à la main. Dans VS Code :

```text
1. Placez le curseur sur le widget à extraire (par exemple sur `Container`)
2. Appuyez sur Ctrl+.   (Cmd+. sur macOS)
3. Choisissez « Extract Widget »
4. Tapez le nom de la classe : CarteStat
```

VS Code crée la classe, déplace le code, et remplace l'original par un appel. Il ne devine pas les paramètres : c'est à vous de transformer les valeurs codées en dur en champs `final`, comme ci-dessus.

### 44.21.4 — Les cinq bénéfices de l'extraction en classe

```text
1. LISIBILITÉ    : build() devient une table des matières de l'écran
2. RÉUTILISATION : la carte s'utilise sur trois écrans différents
3. const         : une classe avec constructeur const peut être instanciée en const
4. RECONSTRUCTION: Flutter peut sauter le sous-arbre inchangé (voir 44.22)
5. INSPECTEUR    : votre classe apparaît sous son nom dans l'inspecteur de widgets
```

Le point 3 mérite qu'on s'y arrête. Dans la version extraite, la ligne suivante est possible :

```dart
body: const Center(
  child: Column(
    children: [
      CarteStat(icone: Icons.favorite, ...),
```

Un seul `const`, et les trois cartes entières deviennent constantes. Dans la version d'origine, c'était impossible parce que `Colors.deepPurple.shade50` et `BorderRadius.circular(12)` étaient calculés au milieu du `build()` de l'écran.

### 44.21.5 — Où mettre la classe ?

Tant que vous apprenez, laissez toutes les classes dans `lib/main.dart` : les exemples de ce chapitre restent copiables d'un bloc. Dans un vrai projet, on suit la structure vue au chapitre 43 :

```text
lib/
├── main.dart                  <- runApp() et MaterialApp uniquement
├── ecrans/
│   ├── ecran_accueil.dart
│   └── ecran_statistiques.dart
└── widgets/
    ├── carte_stat.dart
    └── barre_energie.dart
```

Une classe par fichier, le nom du fichier en `snake_case`, le nom de la classe en `PascalCase`.

---

## 44.22 — Extraire dans une méthode : pourquoi c'est moins bon

Il existe une autre façon de raccourcir un `build()` : déplacer un morceau dans une méthode privée qui renvoie un `Widget`. C'est tentant, et c'est ce que font beaucoup de débutants. Voyons pourquoi c'est une moins bonne idée.

### 44.22.1 — La version « méthode »

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
      title: 'Extraction en méthode',
      debugShowCheckedModeBanner: false,
      home: const EcranStatistiques(),
    );
  }
}

class EcranStatistiques extends StatelessWidget {
  const EcranStatistiques({super.key});

  // Une MÉTHODE qui renvoie un widget. Ce n'est PAS un widget.
  Widget _construireCarte(IconData icone, Color couleur, String libelle, String valeur) {
    return Container(
      width: 260,
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.symmetric(vertical: 8),
      decoration: BoxDecoration(
        color: Colors.deepPurple.shade50,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: [
          Icon(icone, color: couleur),
          const SizedBox(width: 12),
          Text(libelle),
          const Spacer(),
          Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Statistiques')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _construireCarte(Icons.favorite, Colors.red, 'Vies', '3'),
            _construireCarte(Icons.bolt, Colors.orange, 'Énergie', '87'),
            _construireCarte(Icons.star, Colors.amber, 'Score', '12450'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :** rigoureusement identique aux deux versions précédentes.

Le `build()` est court, le code n'est pas dupliqué. Alors, où est le problème ?

### 44.22.2 — Différence 1 : aucun nœud n'est créé dans l'arbre

C'est la différence fondamentale.

```text
AVEC UNE CLASSE                       AVEC UNE MÉTHODE
───────────────                       ────────────────
EcranStatistiques                     EcranStatistiques
└── Scaffold                          └── Scaffold
    └── Center                            └── Center
        └── Column                            └── Column
            ├── CarteStat   ◄── un nœud           ├── Container   ◄── pas de nœud
            │   └── Container                     │   └── Row
            ├── CarteStat                         ├── Container
            │   └── Container                     │   └── Row
            └── CarteStat                         └── Container
                └── Container                         └── Row
```

Une méthode ne crée **rien** dans l'arbre. Son résultat est simplement recopié à l'endroit de l'appel, comme si vous aviez écrit le code sur place. Vous n'avez rien découpé du point de vue de Flutter : vous avez seulement déplacé du texte dans votre fichier.

### 44.22.3 — Différence 2 : pas de reconstruction indépendante

Un widget de classe possède son propre élément. Flutter peut donc décider de le reconstruire, ou **de ne pas le reconstruire**, indépendamment de son parent.

Une méthode n'a pas d'élément. Elle est ré-exécutée **systématiquement** dès que le `build()` du parent est appelé, même si rien n'a changé pour elle.

```text
Le parent se reconstruit (par exemple parce que le score change)

  version CLASSE  : Flutter compare CarteStat ancien / nouveau.
                    Si les paramètres sont identiques (et si c'est const),
                    il conserve tout et ne redescend pas.

  version MÉTHODE : la méthode est rappelée, un Container tout neuf est créé,
                    la comparaison redescend dans tout le sous-arbre.
                    À chaque fois. Sans exception.
```

Sur un écran figé, la différence est invisible. Sur une liste qui se reconstruit à chaque frappe au clavier, elle devient mesurable.

### 44.22.4 — Différence 3 : pas de `const` possible

Une méthode est appelée à l'exécution. Son résultat ne peut donc jamais être constant :

```dart
const _construireCarte(...)   // n'a aucun sens, ne compile pas
const CarteStat(...)          // parfaitement valide
```

Vous perdez d'un coup tout le bénéfice de la section 44.19.

### 44.22.5 — Différence 4 : l'inspecteur ne montre rien

Dans l'inspecteur de widgets (section 44.26), la version en classe affiche :

```text
Column
├── CarteStat
├── CarteStat
└── CarteStat
```

La version en méthode affiche :

```text
Column
├── Container
├── Container
└── Container
```

Trois `Container` anonymes, impossibles à distinguer les uns des autres. Sur un écran réel qui en contient quarante, le débogage devient pénible.

### 44.22.6 — Différence 5 : pas de paramètres nommés, pas de valeurs par défaut

Comparez les deux appels :

```dart
// méthode : quatre paramètres positionnels, l'ordre est à retenir
_construireCarte(Icons.star, Colors.amber, 'Score', '12450')

// classe : chaque paramètre est nommé, l'ordre est libre, on ne peut pas se tromper
CarteStat(
  icone: Icons.star,
  couleur: Colors.amber,
  libelle: 'Score',
  valeur: '12450',
)
```

Rien n'interdit d'écrire une méthode à paramètres nommés, mais en pratique on ne le fait pas, et les erreurs d'ordre arrivent : `_construireCarte(Icons.star, Colors.amber, '12450', 'Score')` compile parfaitement et affiche n'importe quoi.

### 44.22.7 — Le tableau de décision

| Critère | Méthode `Widget _xxx()` | Classe `class Xxx extends StatelessWidget` |
| --- | --- | --- |
| Crée un nœud dans l'arbre | non | oui |
| Peut être `const` | non | oui |
| Reconstruction indépendante | non | oui |
| Visible dans l'inspecteur | non | oui |
| Réutilisable dans un autre fichier | difficilement | oui |
| Paramètres nommés naturels | rarement | oui |
| Rapidité d'écriture | légèrement plus rapide | 5 lignes de plus |

### 44.22.8 — La règle pratique

> Par défaut, **extrayez toujours dans une classe**. La méthode reste acceptable pour un fragment court, utilisé une seule fois, purement décoratif, et qui ne se reconstruit jamais. Dans le doute, faites une classe : vous ne le regretterez jamais, l'inverse si.


---

## 44.23 — Découper son interface en petits widgets

Vous savez maintenant extraire une classe. Reste la question difficile : **où couper ?**

### 44.23.1 — La méthode du crayon

Prenez la maquette de votre écran et entourez les zones. Chaque zone entourée est un widget candidat.

```text
┌───────────────────────────────────────┐
│  Donjon Infini              ♥ 3       │  <- (1) barre : déjà AppBar
├───────────────────────────────────────┤
│  ╭─────────────────────────────────╮  │
│  │  ▲  Alex — niveau 7             │  │  <- (2) EnTeteHeros
│  │     Guerrier du Nord            │  │
│  ╰─────────────────────────────────╯  │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │ ♥ Vies                       3  │  │  <- (3) CarteStat
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ⚡ Énergie                  87  │  │  <- (3) CarteStat
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ★ Score                  12450  │  │  <- (3) CarteStat
│  ╰─────────────────────────────────╯  │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │        COMMENCER LA PARTIE      │  │  <- (4) BoutonPrincipal
│  ╰─────────────────────────────────╯  │
└───────────────────────────────────────┘
```

Quatre zones, donc trois classes à écrire (la barre est déjà un widget de Flutter). Le `build()` de l'écran ne contiendra plus que l'assemblage.

### 44.23.2 — Les quatre signaux qui disent « il faut extraire »

```text
SIGNAL 1 : le code est répété au moins deux fois
           -> extraire, avec les différences en paramètres

SIGNAL 2 : le build() dépasse ~50 lignes ou 5 niveaux d'indentation
           -> extraire les blocs les plus profonds

SIGNAL 3 : vous pouvez donner un NOM MÉTIER au bloc
           « ça, c'est la carte du héros » -> c'est un widget

SIGNAL 4 : une petite partie change souvent, le reste jamais
           -> isoler la partie mouvante (déterminant à partir du chapitre 45)
```

### 44.23.3 — Les trois signaux qui disent « n'extrayez pas »

```text
CONTRE-SIGNAL 1 : le bloc fait deux lignes et n'a pas de nom évident
                  -> un Padding autour d'un Text n'est pas un widget métier

CONTRE-SIGNAL 2 : vous devez passer huit paramètres pour que ça marche
                  -> la découpe est mal placée, recoupez ailleurs

CONTRE-SIGNAL 3 : vous extrayez « au cas où », sans besoin réel
                  -> vingt classes d'une ligne sont pires qu'un build() de 60 lignes
```

### 44.23.4 — L'écran complet, entièrement découpé

Voici l'aboutissement du chapitre. Lisez d'abord le `build()` de `EcranHeros` : il se lit comme un sommaire.

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
      title: 'Donjon Infini',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranHeros(),
    );
  }
}

/// L'écran principal : il ne fait qu'assembler des widgets.
class EcranHeros extends StatelessWidget {
  const EcranHeros({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Donjon Infini'),
        centerTitle: true,
        backgroundColor: Theme.of(context).colorScheme.primary,
        foregroundColor: Theme.of(context).colorScheme.onPrimary,
      ),
      body: const Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            EnTeteHeros(nom: 'Alex', classe: 'Guerrier du Nord', niveau: 7),
            SizedBox(height: 24),
            CarteStat(icone: Icons.favorite, couleur: Colors.red, libelle: 'Vies', valeur: '3'),
            CarteStat(icone: Icons.bolt, couleur: Colors.orange, libelle: 'Énergie', valeur: '87'),
            CarteStat(icone: Icons.star, couleur: Colors.amber, libelle: 'Score', valeur: '12450'),
            Spacer(),
            BoutonPrincipal(libelle: 'COMMENCER LA PARTIE'),
            SizedBox(height: 16),
          ],
        ),
      ),
    );
  }
}

/// Zone 2 : l'en-tête, avec l'avatar, le nom, la classe et le niveau.
class EnTeteHeros extends StatelessWidget {
  const EnTeteHeros({
    super.key,
    required this.nom,
    required this.classe,
    required this.niveau,
  });

  final String nom;
  final String classe;
  final int niveau;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Row(
      children: [
        Icon(Icons.shield, size: 64, color: couleurs.primary),
        const SizedBox(width: 16),
        Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '$nom — niveau $niveau',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            Text(classe, style: Theme.of(context).textTheme.bodyMedium),
          ],
        ),
      ],
    );
  }
}

/// Zone 3 : une ligne de statistique, réutilisée trois fois.
class CarteStat extends StatelessWidget {
  const CarteStat({
    super.key,
    required this.icone,
    required this.couleur,
    required this.libelle,
    required this.valeur,
  });

  final IconData icone;
  final Color couleur;
  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.symmetric(vertical: 6),
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surfaceContainerHighest,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: [
          Icon(icone, color: couleur),
          const SizedBox(width: 12),
          Text(libelle),
          const Spacer(),
          Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}

/// Zone 4 : le bouton d'action principal, sur toute la largeur.
class BoutonPrincipal extends StatelessWidget {
  const BoutonPrincipal({super.key, required this.libelle});

  final String libelle;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Container(
      width: double.infinity,
      padding: const EdgeInsets.symmetric(vertical: 18),
      decoration: BoxDecoration(
        color: couleurs.primary,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Text(
        libelle,
        textAlign: TextAlign.center,
        style: TextStyle(
          color: couleurs.onPrimary,
          fontSize: 18,
          fontWeight: FontWeight.bold,
          letterSpacing: 1.2,
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│           Donjon Infini               │  <- fond violet
├───────────────────────────────────────┤
│  ▲   Alex — niveau 7                  │
│ ███  Guerrier du Nord                 │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │ ♥ Vies                       3  │  │
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ⚡ Énergie                  87  │  │
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ★ Score                  12450  │  │
│  ╰─────────────────────────────────╯  │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │      COMMENCER LA PARTIE        │  │  <- fond violet, texte clair
│  ╰─────────────────────────────────╯  │
└───────────────────────────────────────┘
```

Et l'arbre correspondant :

```text
GameApp
└── MaterialApp
    └── EcranHeros
        └── Scaffold
            ├── AppBar
            └── Padding
                └── Column
                    ├── EnTeteHeros
                    ├── SizedBox
                    ├── CarteStat  (Vies)
                    ├── CarteStat  (Énergie)
                    ├── CarteStat  (Score)
                    ├── Spacer
                    ├── BoutonPrincipal
                    └── SizedBox
```

Neuf lignes lisibles là où il y en avait cent vingt.

> `Spacer` occupe tout l'espace vertical restant dans la `Column` et pousse le bouton vers le bas. C'est un widget du chapitre 46 ; il est utilisé ici parce qu'il rend l'exemple réaliste.

> `surfaceContainerHighest` est l'un des rôles de couleur de Material 3, légèrement plus contrasté que `surface`. Il donne au bloc un fond discret qui s'adapte automatiquement au mode clair et au mode sombre.

### 44.23.5 — Le principe de responsabilité unique

C'est le même principe qu'au chapitre 08 pour les classes Dart :

> Un widget fait **une seule chose**, et son nom doit le dire.

Un widget qui s'appellerait `CarteStatEtBoutonEtAvatar` viole cette règle. Un widget qui s'appelle `CarteStat` la respecte.

---

## 44.24 — Les constructeurs de widget et les paramètres nommés (rappel chapitre 09)

Toutes les classes de widget que vous avez écrites suivent exactement le même moule. Détaillons-le, en reliant chaque élément au chapitre 09.

### 44.24.1 — Le moule complet

```dart
class CarteStat extends StatelessWidget {
  //  1        2                  3
  const CarteStat({
    super.key,                 // 4
    required this.libelle,     // 5
    required this.valeur,      // 5
    this.couleur = Colors.grey,// 6
  });

  final String libelle;        // 7
  final String valeur;         // 7
  final Color couleur;         // 7

  @override                    // 8
  Widget build(BuildContext context) {
    return Text('$libelle : $valeur', style: TextStyle(color: couleur));
  }
}
```

| Repère | Élément | Explication |
| --- | --- | --- |
| 1 | `class` | c'est une classe Dart ordinaire (chapitre 08) |
| 2 | `extends StatelessWidget` | héritage (chapitre 10) : on hérite du comportement de widget |
| 3 | `const` devant le constructeur | rend possible `const CarteStat(...)` (section 44.19) |
| 4 | `super.key` | passe la clé à la classe mère (chapitre 09, constructeur de la superclasse) |
| 5 | `required this.xxx` | paramètre nommé obligatoire, initialise directement le champ |
| 6 | `this.xxx = valeur` | paramètre nommé avec valeur par défaut, donc facultatif |
| 7 | `final` | obligatoire : un widget est immuable |
| 8 | `@override` | on redéfinit `build()`, déclarée dans la classe mère |

### 44.24.2 — Pourquoi des paramètres nommés et pas positionnels ?

Rappel du chapitre 09 : les accolades `{ }` dans la liste des paramètres les rendent **nommés**.

```dart
// positionnel : l'ordre compte, et rien ne dit ce que veut dire chaque valeur
CarteStat('Vies', '3', Colors.red)

// nommé : l'ordre est libre, chaque valeur est étiquetée
CarteStat(libelle: 'Vies', valeur: '3', couleur: Colors.red)
```

Flutter utilise systématiquement les paramètres nommés, pour trois raisons :

```text
1. Un widget a souvent 5, 10, parfois 30 paramètres.
   Personne ne peut retenir l'ordre de 30 valeurs.

2. La plupart sont facultatifs.
   En positionnel, sauter le 3e pour donner le 7e est impossible.

3. Le code s'auto-documente.
   `centerTitle: true` se comprend seul ; `true` tout court, non.
```

Regardez `Container` : il accepte `key`, `alignment`, `padding`, `color`, `decoration`, `foregroundDecoration`, `width`, `height`, `constraints`, `margin`, `transform`, `transformAlignment`, `child`, `clipBehavior`. Tous nommés, tous facultatifs sauf aucun. En positionnel, ce serait inutilisable.

### 44.24.3 — `required`, valeur par défaut, ou nullable ?

Trois façons de traiter un paramètre nommé, à choisir selon le sens (rappel du chapitre 12 sur le null safety) :

```dart
class BarreVie extends StatelessWidget {
  const BarreVie({
    super.key,
    required this.pointsDeVie,        // A : obligatoire, sans lui le widget n'a pas de sens
    this.pointsMax = 100,             // B : facultatif avec une valeur par défaut
    this.libelle,                     // C : facultatif, peut rester nul
  });

  final int pointsDeVie;              // non nullable, garanti par `required`
  final int pointsMax;                // non nullable, garanti par la valeur par défaut
  final String? libelle;              // NULLABLE : il faut le point d'interrogation

  @override
  Widget build(BuildContext context) {
    return Text(libelle ?? 'Vie : $pointsDeVie / $pointsMax');
  }
}
```

| Cas | Écriture | Type du champ |
| --- | --- | --- |
| A | `required this.x` | non nullable |
| B | `this.x = valeur` | non nullable |
| C | `this.x` | **nullable** (`Type?`) |

> Erreur fréquente : écrire `this.libelle` avec un champ `final String libelle;` non nullable. Le compilateur refuse, car rien ne garantit que le champ sera initialisé. Il faut choisir entre `required`, une valeur par défaut, ou `String?`.

### 44.24.4 — Exemple complet et exécutable

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
      title: 'Constructeurs',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Paramètres nommés')),
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Seul le paramètre obligatoire est fourni.
              BarreVie(pointsDeVie: 87),
              SizedBox(height: 12),
              // On surcharge la valeur par défaut.
              BarreVie(pointsDeVie: 240, pointsMax: 300),
              SizedBox(height: 12),
              // On fournit le paramètre facultatif nullable, dans un autre ordre.
              BarreVie(libelle: 'Boss : presque mort', pointsDeVie: 12, pointsMax: 500),
            ],
          ),
        ),
      ),
    );
  }
}

class BarreVie extends StatelessWidget {
  const BarreVie({
    super.key,
    required this.pointsDeVie,
    this.pointsMax = 100,
    this.libelle,
  });

  final int pointsDeVie;
  final int pointsMax;
  final String? libelle;

  @override
  Widget build(BuildContext context) {
    final double ratio = pointsDeVie / pointsMax;
    final Color couleur = ratio > 0.5
        ? Colors.green
        : (ratio > 0.2 ? Colors.orange : Colors.red);

    return Column(
      children: [
        Text(libelle ?? 'Vie : $pointsDeVie / $pointsMax'),
        const SizedBox(height: 4),
        SizedBox(
          width: 240,
          height: 16,
          child: LinearProgressIndicator(
            value: ratio,
            color: couleur,
            backgroundColor: Colors.black12,
          ),
        ),
      ],
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Paramètres nommés                    │
├───────────────────────────────────────┤
│         Vie : 87 / 100                │
│      ████████████████████░░░          │  <- vert (87 %)
│                                       │
│        Vie : 240 / 300                │
│      ██████████████████░░░░░          │  <- vert (80 %)
│                                       │
│     Boss : presque mort               │
│      █░░░░░░░░░░░░░░░░░░░░░           │  <- rouge (2 %)
└───────────────────────────────────────┘
```

Trois appels, trois combinaisons de paramètres différentes, une seule classe.

> `LinearProgressIndicator` attend un `value` compris entre 0.0 et 1.0. Au-delà, la barre est simplement pleine. Le calcul de `ratio` et le choix de la couleur sont du Dart pur : ce sont les opérateurs et l'opérateur ternaire du chapitre 03.

---

## 44.25 — `child` et `children`

Deux paramètres que vous croiserez dans presque tous les widgets. La différence est simple, mais l'erreur est très fréquente.

### 44.25.1 — La règle

```text
child    -> UN SEUL widget          -> type Widget
children -> PLUSIEURS widgets       -> type List<Widget>, on écrit des crochets [ ]
```

### 44.25.2 — Qui utilise quoi

| Prend `child` | Prend `children` |
| --- | --- |
| `Center` | `Column` |
| `Padding` | `Row` |
| `Container` | `Stack` |
| `SizedBox` | `ListView` |
| `Align` | `Wrap` |
| `Expanded` | `GridView` |
| `Card` | `CustomScrollView` (via `slivers`) |

La logique est limpide : un widget qui **positionne ou décore** n'a besoin que d'un enfant. Un widget qui **dispose plusieurs éléments** a besoin d'une liste.

### 44.25.3 — Schéma

```text
        child (1 enfant)                     children (n enfants)

        Center                               Column
          │                                    ├── Text('Vies')
          ▼                                    ├── Text('Énergie')
        Text('Score')                          └── Text('Score')
```

### 44.25.4 — Les deux en même temps

Un `Scaffold` a `body:` (un seul widget) mais aussi `actions:` sur son `AppBar` (une liste). Un `Container` n'a que `child`. Rien n'interdit de combiner :

```dart
Center(                 // child : un seul
  child: Column(        // children : plusieurs
    children: [
      Text('Vies'),
      Text('Énergie'),
    ],
  ),
)
```

C'est l'écriture la plus courante de tout Flutter : **`Center` + `Column`**. Retenez-la.

### 44.25.5 — Exemple complet

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
      title: 'child et children',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(
          title: const Text('child et children'),
          // actions : une LISTE
          actions: const [
            Icon(Icons.favorite),
            SizedBox(width: 16),
          ],
        ),
        // body : UN SEUL widget
        body: const Padding(
          // padding a un child : UN SEUL widget
          padding: EdgeInsets.all(24),
          child: Column(
            // Column a des children : une LISTE
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Inventaire', style: TextStyle(fontSize: 24)),
              SizedBox(height: 16),
              Row(
                // Row aussi : une LISTE
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  Icon(Icons.local_drink, size: 40, color: Colors.pink),
                  Icon(Icons.hardware, size: 40, color: Colors.blueGrey),
                  Icon(Icons.paid, size: 40, color: Colors.amber),
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

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  child et children             ♥      │
├───────────────────────────────────────┤
│                                       │
│            Inventaire                 │
│                                       │
│      ▽          ⚒          ◉          │
│    potion      épée      pièces       │
│                                       │
└───────────────────────────────────────┘
```

### 44.25.6 — Les trois erreurs classiques

**Erreur 1 — une liste là où un seul enfant est attendu :**

```dart
Center(
  child: [                      // ERREUR
    Text('A'),
    Text('B'),
  ],
)
```

Message :

```text
The argument type 'List<Text>' can't be assigned to the parameter type 'Widget?'.
```

Correction : envelopper dans une `Column` ou une `Row`.

```dart
Center(
  child: Column(
    children: [
      Text('A'),
      Text('B'),
    ],
  ),
)
```

**Erreur 2 — `child` au lieu de `children` :**

```dart
Column(
  child: Text('A'),             // ERREUR
)
```

Message :

```text
The named parameter 'child' isn't defined.
```

Correction : `children: [Text('A')]`. Oui, même pour un seul enfant, les crochets sont obligatoires.

**Erreur 3 — oublier les crochets :**

```dart
Column(
  children: Text('A'),          // ERREUR
)
```

Message :

```text
The argument type 'Text' can't be assigned to the parameter type 'List<Widget>'.
```

Correction : `children: [Text('A')]`.

### 44.25.7 — Le moyen mnémotechnique

> `children` finit par un **s** comme **liste au pluriel**, et un pluriel s'écrit toujours entre crochets `[ ]`.

---

## 44.26 — L'inspecteur de widgets de VS Code

Lire son arbre dans le code est une chose. Le voir vivre dans l'application en est une autre. C'est le rôle de l'inspecteur de widgets.

### 44.26.1 — L'ouvrir

L'application doit être **lancée en mode debug** (`flutter run`, ou F5 dans VS Code). Ensuite :

```text
Méthode 1 : la palette de commandes
   Ctrl+Shift+P  (Cmd+Shift+P sur macOS)
   > Flutter: Open DevTools
   > Open Widget Inspector Page

Méthode 2 : la barre d'état, en bas à droite de VS Code
   Cliquez sur le nom de l'appareil, puis sur l'icône DevTools

Méthode 3 : depuis le terminal où tourne `flutter run`
   L'URL de DevTools est affichée au démarrage :
   The Flutter DevTools debugger and profiler on Chrome is available at:
   http://127.0.0.1:9101?uri=http://127.0.0.1:52341/xxxx=/
```

> L'inspecteur n'existe **qu'en mode debug**. En mode `--release`, tous les outils de débogage sont retirés du binaire.

### 44.26.2 — Ce que vous voyez

L'inspecteur affiche trois panneaux :

```text
┌──────────────────────────┬────────────────────┬──────────────────────┐
│  ARBRE DE WIDGETS        │  DÉTAILS           │  APERÇU DE L'ÉCRAN   │
│                          │                    │                      │
│  MaterialApp             │  Text              │  ┌────────────────┐  │
│   └ EcranHeros           │   data: "Alex"     │  │  Donjon Infini │  │
│      └ Scaffold          │   style: TextStyle │  ├────────────────┤  │
│         ├ AppBar         │     fontSize: 22   │  │   ▲ Alex       │  │
│         └ Padding        │   renderObject:    │  │                │  │
│            └ Column      │     RenderParagraph│  │  ♥ Vies     3  │  │
│               ├ EnTete…  │     size: 96 x 28  │  │  ⚡ Énergie 87  │  │
│               ├ CarteStat│   constraints:     │  │                │  │
│               ├ CarteStat│     0 <= w <= 343  │  └────────────────┘  │
│               └ CarteStat│                    │                      │
└──────────────────────────┴────────────────────┴──────────────────────┘
```

Notez la ligne `size: 96 x 28` et la ligne `constraints:`. Ce sont les informations de l'arbre des objets de rendu (44.17). C'est là que vous verrez pourquoi un widget déborde ou reste invisible.

### 44.26.3 — Le mode « sélection de widget »

C'est la fonction la plus utile.

```text
1. Cliquez sur le bouton « Select Widget Mode » (icône de curseur)
2. Cliquez sur un élément DANS l'application qui tourne
3. L'inspecteur sélectionne le widget correspondant dans l'arbre
4. VS Code ouvre le fichier et place le curseur sur la bonne ligne
```

Vous passez ainsi de « ce texte est mal placé à l'écran » à « voici la ligne 87 de `carte_stat.dart` » en un clic.

C'est aussi ici que l'extraction en classe (44.21) paye : votre arbre affiche `CarteStat`, `EnTeteHeros`, `BoutonPrincipal`. Avec des méthodes, il n'afficherait que des `Container` anonymes.

### 44.26.4 — Les interrupteurs utiles

| Interrupteur | Ce qu'il fait |
| --- | --- |
| Select Widget Mode | cliquer dans l'application pour sélectionner le widget |
| Show Guidelines | dessine les bordures et les axes de tous les widgets |
| Show Baselines | dessine la ligne de base des textes |
| Highlight Repaints | colore les zones repeintes ; utile pour traquer les reconstructions |
| Highlight Oversized Images | signale les images trop grandes (chapitre 47) |
| Slow Animations | ralentit les animations pour les observer |

`Show Guidelines` correspond à l'option `debugPaintSizeEnabled` du code. Elle transforme votre écran en plan d'architecte : chaque rectangle est tracé, chaque marge est visible. C'est l'outil numéro un pour comprendre un layout qui ne fait pas ce qu'on attend.

### 44.26.5 — L'équivalent en ligne de commande : `debugDumpApp()`

Sans interface graphique, vous pouvez demander à Flutter d'écrire l'arbre dans la console :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
  // Affiche l'arbre complet dans la console, une fois construit.
  WidgetsBinding.instance.addPostFrameCallback((_) {
    debugDumpApp();
  });
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(child: Text('Arbre')),
      ),
    );
  }
}
```

**Résultat dans la console (extrait très raccourci) :**

```text
[root](renderObject: RenderView#1a2b3)
└─GameApp
  └─MaterialApp
    └─ScrollConfiguration
      └─HeroControllerScope
        └─Focus
          └─...
            └─Scaffold
              └─Center
                └─Text("Arbre")
```

Vous découvrez au passage que `MaterialApp` insère lui-même une bonne dizaine de widgets. C'est normal : gestion du focus, du défilement, des transitions, de la localisation.

> `debugPrint()` est également préférable à `print()` pour les longues sorties : il découpe le texte pour éviter que la console Android ne le tronque.

### 44.26.6 — Une méthode de débogage en trois étapes

```text
1. Le widget est-il là ?        -> inspecteur, panneau de gauche
2. Quelle taille a-t-il ?       -> panneau du milieu, ligne `size`
3. Quelles contraintes reçoit-il ? -> panneau du milieu, ligne `constraints`
```

Quatre-vingt-dix pour cent des « mon widget ne s'affiche pas » se résolvent avec ces trois questions. Le plus souvent, la réponse est « il est là, mais sa taille vaut 0 x 0 ».

---

## 44.27 — Lire un message d'erreur de rendu Flutter

Les messages d'erreur de Flutter sont longs. Beaucoup de débutants les referment sans les lire. C'est dommage : ils sont écrits pour être lus, et ils contiennent presque toujours la solution.

### 44.27.1 — L'anatomie d'un message

```text
════════ Exception caught by rendering library ═══════════════════   <- (1) qui a échoué
The following assertion was thrown during layout:                    <- (2) quelle phase
A RenderFlex overflowed by 1146 pixels on the right.                 <- (3) LE MESSAGE

The relevant error-causing widget was:                               <- (4) VOTRE ligne
    Row Row:file:///home/alex/jeu/lib/main.dart:42:16

The overflowing RenderFlex has an orientation of Axis.horizontal.    <- (5) le détail
The edge of the RenderFlex that is overflowing has been marked in
the rendering with a yellow and black striped pattern. ...

Consider applying a flex factor (e.g. using an Expanded widget) ...  <- (6) LA PISTE
════════════════════════════════════════════════════════════════════
```

Les deux lignes à lire en premier sont **(3)** et **(4)**. La (3) dit ce qui s'est passé ; la (4) donne le fichier, la ligne et la colonne de **votre** code. Tout le reste est du contexte.

> Ne cherchez jamais votre bug dans la pile d'appels de Flutter (`package:flutter/src/rendering/…`). L'erreur est presque toujours dans votre fichier, indiqué à la ligne « The relevant error-causing widget was ».

### 44.27.2 — Les trois phases où une erreur peut survenir

```text
build   -> vous avez décrit quelque chose d'impossible ou de nul
layout  -> les tailles ne sont pas calculables (débordement, contrainte infinie)
paint   -> rare : le dessin lui-même échoue
```

Le message vous dit toujours laquelle : « thrown during layout », « thrown building », « thrown during paint ».

### 44.27.3 — Erreur numéro 1 : `A RenderFlex overflowed by N pixels`

**Le message exact :**

```text
A RenderFlex overflowed by 1146 pixels on the right.
```

**Ce qu'il signifie :** une `Row` ou une `Column` contient plus de contenu que la place disponible. À l'écran, une bande jaune et noire apparaît sur le bord qui déborde.

**Le code fautif :**

```dart
Row(
  children: [
    Icon(Icons.shield),
    Text('Un très très très long texte de description du héros qui ne tient pas'),
  ],
)
```

**Les corrections possibles :**

```dart
// A. laisser le texte occuper la place restante et revenir à la ligne
Row(
  children: [
    const Icon(Icons.shield),
    Expanded(
      child: Text('Un très très long texte...'),
    ),
  ],
)

// B. couper le texte avec des points de suspension
Expanded(
  child: Text('Un très long texte...', overflow: TextOverflow.ellipsis),
)

// C. pour une Column trop haute : la rendre défilante
SingleChildScrollView(
  child: Column(children: [ /* ... */ ]),
)
```

> `Expanded` et `SingleChildScrollView` sont détaillés aux chapitres 46 et 48. Retenez pour l'instant le réflexe : **débordement horizontal dans une `Row` -> `Expanded`**.

### 44.27.4 — Erreur numéro 2 : `Vertical viewport was given unbounded height`

**Le message exact :**

```text
Vertical viewport was given unbounded height.
```

**Ce qu'il signifie :** vous avez mis une `ListView` (qui veut prendre toute la hauteur disponible) à l'intérieur d'une `Column` (qui donne une hauteur illimitée à ses enfants). Flutter ne peut pas calculer une hauteur infinie.

**Le code fautif :**

```dart
Column(
  children: [
    Text('Inventaire'),
    ListView(children: [ /* ... */ ]),   // hauteur inconnue
  ],
)
```

**Les corrections :**

```dart
// A. donner explicitement la hauteur restante
Column(
  children: [
    const Text('Inventaire'),
    Expanded(child: ListView(children: const [ /* ... */ ])),
  ],
)

// B. donner une hauteur fixe
SizedBox(height: 200, child: ListView(children: const [ /* ... */ ]))

// C. dire à la liste de ne prendre que la place de son contenu
ListView(shrinkWrap: true, children: const [ /* ... */ ])
```

### 44.27.5 — Erreur numéro 3 : `RenderBox was not laid out`

**Le message exact :**

```text
RenderBox was not laid out: RenderViewport#5a477 NEEDS-LAYOUT NEEDS-PAINT
```

**Ce qu'il signifie :** presque toujours une **conséquence** de l'erreur précédente. Un widget n'a pas pu être mesuré, donc tout ce qui en dépend échoue.

**La méthode :** faites défiler la console vers le **haut**. La première erreur affichée est la vraie cause ; celle-ci n'en est que l'écho.

> Règle générale : dans une avalanche d'erreurs Flutter, seule **la première** compte.

### 44.27.6 — Erreur numéro 4 : `Incorrect use of ParentDataWidget`

**Le message exact :**

```text
Incorrect use of ParentDataWidget.
```

**Ce qu'il signifie :** vous avez utilisé `Expanded`, `Flexible` ou `Positioned` ailleurs que directement sous le widget qui les comprend.

```text
Expanded / Flexible  -> uniquement enfant DIRECT d'une Row, d'une Column ou d'un Flex
Positioned           -> uniquement enfant DIRECT d'un Stack
```

**Le code fautif :**

```dart
Center(
  child: Expanded(          // Center n'est pas une Row ni une Column
    child: Text('Score'),
  ),
)
```

**La correction :** retirer `Expanded`, ou remettre un `Row`/`Column` comme parent direct.

### 44.27.7 — Erreur numéro 5 : `Null check operator used on a null value`

**Le message exact :**

```text
Null check operator used on a null value
```

**Ce qu'il signifie :** rappel du chapitre 12. Vous avez écrit `!` sur une valeur nulle.

```dart
final String? nom = null;
Text(nom!);      // plantage
```

**La correction :**

```dart
Text(nom ?? 'Héros sans nom');
```

### 44.27.8 — Erreur numéro 6 : `No MaterialLocalizations found`

**Le message exact commence par :**

```text
No MaterialLocalizations found.
```

**Ce qu'il signifie :** un widget Material (souvent un `Scaffold`, un `AppBar` ou une boîte de dialogue) est utilisé **en dehors** d'un `MaterialApp`.

**Le code fautif :**

```dart
void main() {
  runApp(Scaffold(body: Center(child: Text('Bonjour'))));  // pas de MaterialApp
}
```

**La correction :**

```dart
void main() {
  runApp(const MaterialApp(
    home: Scaffold(body: Center(child: Text('Bonjour'))),
  ));
}
```

C'est encore une application de la règle du `context` (44.15.7) : le widget cherche au-dessus de lui quelque chose qui n'y est pas.

### 44.27.9 — L'écran rouge

En mode debug, une erreur de construction affiche un rectangle rouge à l'endroit fautif, avec le message écrit dessus :

```text
┌───────────────────────────────────────┐
│  Donjon Infini                        │
├───────────────────────────────────────┤
│▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│
│▓ A RenderFlex overflowed by         ▓│
│▓ 1146 pixels on the right.          ▓│
│▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│
└───────────────────────────────────────┘
```

Ce n'est pas un plantage de l'application : c'est un widget d'erreur, affiché à la place du widget qui a échoué. Le reste de l'écran continue de fonctionner. En mode `--release`, ce rectangle est remplacé par une zone grise vide.

### 44.27.10 — La méthode de lecture, en cinq étapes

```text
1. Remonter à la PREMIÈRE erreur de la console.
2. Lire la ligne « The following assertion was thrown ... » -> quelle phase.
3. Lire le message principal -> quoi.
4. Lire « The relevant error-causing widget was » -> OÙ, dans VOTRE fichier.
5. Lire le paragraphe « Consider ... » -> Flutter propose souvent la solution.
```

Appliquez ces cinq étapes systématiquement. En quelques semaines, vous lirez ces messages en dix secondes.


---

## 44.28 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `The named parameter 'child' isn't defined` sur une `Column` | `Column` attend une liste, pas un enfant unique | écrire `children: [ ... ]`, avec les crochets |
| `The argument type 'Text' can't be assigned to the parameter type 'List<Widget>'` | crochets oubliés après `children:` | `children: [Text('A')]` |
| `The argument type 'List<Text>' can't be assigned to the parameter type 'Widget?'` | une liste donnée à `child:` | envelopper dans une `Column` ou une `Row` |
| La barre reste bleue alors que le thème est violet | `Theme.of(context)` appelé dans le même `build()` que le `MaterialApp` : le `context` est au-dessus du thème | extraire l'écran dans une classe, comme `home: const EcranAccueil()` |
| `No MaterialLocalizations found` | un `Scaffold` ou un `AppBar` utilisé hors d'un `MaterialApp` | envelopper la racine dans `MaterialApp` |
| `No MediaQuery widget ancestor could be found` | `MediaQuery.sizeOf(context)` appelé au-dessus du `MaterialApp` | déplacer l'appel dans un widget enfant |
| `A RenderFlex overflowed by N pixels` | le contenu d'une `Row` ou d'une `Column` dépasse la place disponible | `Expanded`, `overflow: TextOverflow.ellipsis` ou `SingleChildScrollView` |
| `Incorrect use of ParentDataWidget` | `Expanded` ou `Positioned` placé ailleurs que sous une `Row`/`Column`/`Stack` | supprimer le widget ou corriger le parent direct |
| `Vertical viewport was given unbounded height` | une `ListView` directement dans une `Column` | envelopper la liste dans `Expanded` ou fixer une hauteur |
| `Invalid constant value` | `const` placé devant une valeur calculée à l'exécution (`'Score : $score'`) | retirer le `const` sur cette ligne et le descendre plus bas |
| `Can't define a const constructor for a class with non-final fields` | un champ de widget déclaré sans `final` | déclarer tous les champs `final` |
| `The parameter 'libelle' can't have a value of 'null'` | champ non nullable déclaré sans `required` ni valeur par défaut | choisir entre `required this.libelle`, `this.libelle = '...'` ou `String? libelle` |
| `Null check operator used on a null value` | opérateur `!` appliqué à une valeur nulle | utiliser `??` avec une valeur de repli |
| Le widget extrait n'apparaît pas dans l'inspecteur | l'extraction a été faite dans une **méthode**, pas dans une classe | créer une classe `extends StatelessWidget` |
| Rien ne se passe quand on appuie sur le bouton | aucun état n'est géré dans ce chapitre : l'écran est figé | c'est normal, voir le chapitre 45 |
| `RenderBox was not laid out` | conséquence d'une erreur de layout survenue plus tôt | remonter à la **première** erreur de la console |
| L'inspecteur de widgets refuse de s'ouvrir | l'application tourne en mode `--release` | relancer avec `flutter run` en mode debug |
| Le texte est écrit mais rien ne s'affiche | le widget a une taille de 0 x 0, ou il est hors de l'écran | activer `Show Guidelines` et lire `size` dans l'inspecteur |
| `Another exception was thrown` répété des dizaines de fois | une seule erreur se répète à chaque image | corriger la première erreur, les autres disparaissent |
| `super.key` oublié dans un constructeur de widget | la clé n'est plus transmise à la classe mère | écrire `const MonWidget({super.key, ...})` |

---

## 44.29 — Résumé du chapitre

### Les notions fondamentales

| Notion | À retenir |
| --- | --- |
| Widget | une classe Dart immuable qui **décrit** une portion d'interface |
| « Tout est widget » | le texte, la marge, l'alignement, l'écran entier et l'application elle-même |
| Description, pas objet | un widget n'est pas ce qui est dessiné ; il dit à Flutter quoi dessiner |
| Arbre de widgets | l'imbrication des widgets, du plus englobant au plus profond |
| Composition | on empile des widgets simples plutôt que d'hériter de widgets compliqués |
| `runApp()` | installe le widget racine et démarre l'application |

### La structure d'un écran

| Widget | Rôle |
| --- | --- |
| `MaterialApp` | racine Material : thème, titre, écran d'accueil, navigation |
| `Scaffold` | squelette d'un écran : `appBar`, `body`, `floatingActionButton`, `bottomNavigationBar` |
| `AppBar` | la barre du haut : `title`, `actions`, `centerTitle` |
| `Center` | centre son unique enfant |
| `Text` | affiche une chaîne |

### Le contexte et le thème

| Notion | À retenir |
| --- | --- |
| `BuildContext` | une poignée sur **l'emplacement** du widget dans l'arbre |
| Règle d'or | un `context` ne voit que ce qui est **au-dessus** de lui |
| `Theme.of(context)` | remonte l'arbre et renvoie le `ThemeData` le plus proche |
| Piège classique | appeler `Theme.of(context)` dans le `build()` qui crée le `MaterialApp` |
| `colorScheme` | `primary`, `onPrimary`, `surface`, `onSurface`, `error` |
| `textTheme` | `headlineMedium`, `titleLarge`, `bodyMedium`... quinze styles nommés |

### Les trois arbres

| Arbre | Rôle | Recréé souvent ? |
| --- | --- | --- |
| Widgets | décrire | oui, à chaque `build()` |
| Éléments | mémoriser la position et l'état ; c'est le `BuildContext` | non |
| Objets de rendu | mesurer, positionner, peindre | non |

### Performance

| Notion | À retenir |
| --- | --- |
| Pourquoi c'est rapide | les widgets sont minuscules, la comparaison est superficielle, l'algorithme est en O(n) |
| `Widget.canUpdate` | même type + même `Key` -> l'élément est conservé |
| `const` | objet créé à la compilation, partagé, et sous-arbre court-circuité |
| Où mettre `const` | le plus haut possible, tant que rien ne dépend de l'exécution |
| `Key` | une étiquette d'identité, utile seulement quand on réordonne des widgets à état |
| Ce qui ralentit vraiment | un calcul lourd, une image trop grande ou une liste non paresseuse dans `build()` |

### Organiser son code

| Notion | À retenir |
| --- | --- |
| Extraire en classe | crée un vrai nœud, autorise `const`, apparaît dans l'inspecteur |
| Extraire en méthode | ne crée aucun nœud, interdit `const`, invisible dans l'inspecteur |
| Quand extraire | code répété, `build()` trop long, bloc qui porte un nom métier |
| Constructeur type | `const MonWidget({super.key, required this.x, this.y = 0, this.z});` |
| Champs | toujours `final` |
| `child` | **un seul** widget |
| `children` | une **liste** de widgets, entre crochets |

### Déboguer

| Outil | Usage |
| --- | --- |
| Inspecteur de widgets | voir l'arbre vivant, la taille et les contraintes de chaque widget |
| Select Widget Mode | cliquer dans l'application pour sauter à la ligne de code |
| Show Guidelines | tracer toutes les bordures pour comprendre un layout |
| `debugDumpApp()` | écrire l'arbre complet dans la console |
| Lire une erreur | première erreur, phase, message, « relevant error-causing widget », « Consider... » |

---

## 44.30 — Exercices

Pour chaque exercice, créez un projet ou réutilisez le vôtre, et écrivez un `lib/main.dart` complet. Vérifiez avec `flutter run`, puis avec `flutter analyze` : aucun avertissement ne doit rester.

### Exercice 1 — L'écran de titre (facile)

Écrivez une application Flutter complète qui affiche :

- une `AppBar` dont le titre est `Donjon Infini` ;
- au centre du corps, le texte `Appuyez pour commencer`.

Contraintes : la bannière `DEBUG` doit être masquée, et tous les widgets qui peuvent être `const` doivent l'être.

### Exercice 2 — Le `const` manquant (facile)

Voici un code qui compile mais que l'analyseur critique. Ajoutez tous les `const` possibles, et **uniquement** ceux qui sont possibles.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(GameApp());
}

class GameApp extends StatelessWidget {
  GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    int vies = 3;
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Statut')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.favorite, color: Colors.red, size: 48),
              SizedBox(height: 12),
              Text('Vies restantes : $vies'),
              SizedBox(height: 12),
              Text('Bonne chance'),
            ],
          ),
        ),
      ),
    );
  }
}
```

Indiquez aussi, en commentaire, pourquoi une des lignes ne peut pas être `const`.

### Exercice 3 — Le thème qui ne s'applique pas (facile)

Le code suivant définit un thème vert, mais la barre reste bleue. Trouvez la cause et corrigez-la, en expliquant en commentaire pourquoi votre correction fonctionne.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const GameApp());

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(
          backgroundColor: Theme.of(context).colorScheme.primary,
          title: const Text('Forêt maudite'),
        ),
        body: const Center(child: Text('Niveau 3')),
      ),
    );
  }
}
```

### Exercice 4 — Dessiner l'arbre (facile)

Écrivez une application qui affiche, centrés verticalement :

- une icône `Icons.local_fire_department` orange de 72 pixels ;
- un espace de 16 pixels ;
- le texte `Salle du dragon` en 24 pixels et en gras ;
- le texte `Danger : extrême` en rouge.

Puis, dans un commentaire en fin de fichier, dessinez l'arbre de widgets complet de votre écran, en respectant l'indentation.

### Exercice 5 — Extraire une classe (moyen)

Le code ci-dessous répète trois fois la même structure. Extrayez-la dans une classe `LigneObjet` avec des paramètres nommés, puis réécrivez le `build()` de l'écran.

```dart
Column(
  children: [
    Row(children: [Icon(Icons.local_drink), SizedBox(width: 8), Text('Potion x3')]),
    Row(children: [Icon(Icons.hardware), SizedBox(width: 8), Text('Épée x1')]),
    Row(children: [Icon(Icons.paid), SizedBox(width: 8), Text('Pièces x12')]),
  ],
)
```

Contraintes : la classe doit avoir un constructeur `const`, deux paramètres nommés obligatoires, et être instanciable en `const`.

### Exercice 6 — `child` ou `children` (moyen)

Ce code contient trois erreurs liées à `child` et `children`. Corrigez-les et donnez le message d'erreur que produisait chacune.

```dart
Scaffold(
  body: Center(
    child: [
      Text('Inventaire'),
      Text('3 objets'),
    ],
  ),
  bottomNavigationBar: Row(
    child: Text('Barre du bas'),
  ),
)
```

Puis ajoutez une `Row` correcte contenant trois icônes.

### Exercice 7 — Les trois sortes de paramètres (moyen)

Écrivez un widget `BadgeNiveau` qui affiche un cercle coloré contenant un numéro de niveau, avec :

- `niveau` : paramètre nommé **obligatoire**, de type `int` ;
- `couleur` : paramètre nommé **facultatif avec valeur par défaut** `Colors.deepPurple` ;
- `titre` : paramètre nommé **facultatif nullable**, affiché sous le cercle s'il est fourni, et rien sinon.

Affichez trois badges dans un écran complet, chacun utilisant une combinaison différente de paramètres.

### Exercice 8 — L'écran découpé (difficile)

Construisez un écran « Fiche du boss » entièrement découpé en widgets, comportant :

- un `MaterialApp` avec un thème rouge (`seedColor: Colors.red`) ;
- une classe `EcranBoss` pour l'écran ;
- une classe `EnTeteBoss` affichant une icône, le nom et le titre du boss ;
- une classe `LigneCaracteristique` réutilisée au moins quatre fois (force, vitesse, armure, dégâts) ;
- une classe `PiedDePage` affichant un avertissement centré.

Contraintes : aucune couleur codée en dur en dehors du thème (utilisez `Theme.of(context).colorScheme`), et le `build()` de `EcranBoss` doit tenir en moins de vingt lignes.

### Exercice 9 — Le débordement (difficile)

Le code suivant produit une erreur `A RenderFlex overflowed by ... pixels on the right`.

```dart
Row(
  children: [
    const Icon(Icons.warning, size: 48),
    const Text(
      'Attention : le dragon ancestral du donjon infini approche de la salle du trône',
      style: TextStyle(fontSize: 20),
    ),
    const Icon(Icons.warning, size: 48),
  ],
)
```

Produisez trois versions corrigées dans un même écran, l'une sous l'autre :

1. avec `Expanded` ;
2. avec `Expanded` et `TextOverflow.ellipsis` ;
3. avec `Expanded` et `maxLines: 2`.

Expliquez en commentaire la différence visuelle entre les trois.

### Exercice 10 — Projet : la fiche de personnage (projet)

Réalisez un `lib/main.dart` complet et unique produisant une fiche de personnage complète, figée, qui réunit tout le chapitre :

- un `MaterialApp` avec thème `ColorScheme.fromSeed` et bannière de debug masquée ;
- un `Scaffold` avec `AppBar` (titre centré, une icône dans `actions`), `body`, `floatingActionButton` et `bottomNavigationBar` à trois destinations ;
- une classe `EcranFiche` dont le `build()` ne fait qu'assembler ;
- une classe `EnTetePersonnage` (icône, nom, classe, niveau) ;
- une classe `BarreStatistique` (libellé, valeur, valeur maximale, couleur) affichant une barre de progression ;
- une classe `PuceCompetence` (icône + libellé) utilisée dans une `Row` ;
- la liste des compétences construite par une boucle `for` sur une `List<String>`, chaque puce recevant une `ValueKey` ;
- au moins un `Theme.of(context)` utilisé pour les couleurs ;
- tous les `const` possibles.

Contraintes : aucun fichier image, aucune interactivité, `flutter analyze` doit être totalement silencieux.


---

## 44.31 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Donjon Infini',
      debugShowCheckedModeBanner: false,
      home: EcranTitre(),
    );
  }
}

class EcranTitre extends StatelessWidget {
  const EcranTitre({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Donjon Infini')),
      body: const Center(
        child: Text('Appuyez pour commencer'),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Donjon Infini                        │
├───────────────────────────────────────┤
│                                       │
│      Appuyez pour commencer           │
│                                       │
└───────────────────────────────────────┘
```

**Explication :** l'arbre est le plus court possible : `GameApp` → `MaterialApp` → `EcranTitre` → `Scaffold` → `Center` → `Text`. Trois points méritent un commentaire. **Premièrement**, `debugShowCheckedModeBanner: false` retire la bannière rouge affichée en haut à droite en mode debug ; elle n'apparaît de toute façon jamais en `--release`, mais elle gêne les captures d'écran. **Deuxièmement**, le `MaterialApp` entier est `const` : c'est possible parce que ses trois paramètres (`title`, `debugShowCheckedModeBanner`, `home`) sont eux-mêmes constants, et que `EcranTitre` possède un constructeur `const`. **Troisièmement**, le `Scaffold` n'est pas `const` : son paramètre `appBar` contient un `AppBar` dont le constructeur n'est pas `const` (il calcule des valeurs à partir du thème). En revanche, le `Center` et son `Text` le sont. C'est exactement la règle de 44.19.5 : on met `const` le plus haut possible, et là où c'est impossible, on le descend d'un cran. Enfin, séparer `EcranTitre` du `GameApp` n'était pas obligatoire ici, mais cette habitude évite le piège du `context` trop haut qui fait l'objet de l'exercice 3.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key}); // const ajouté : sans lui, `const GameApp()` serait impossible

  @override
  Widget build(BuildContext context) {
    int vies = 3;
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Statut')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.favorite, color: Colors.red, size: 48),
              const SizedBox(height: 12),
              // PAS const : `vies` est une variable locale, sa valeur n'est
              // connue qu'à l'exécution. Une chaîne interpolée avec une
              // variable non constante ne peut pas être une constante.
              Text('Vies restantes : $vies'),
              const SizedBox(height: 12),
              const Text('Bonne chance'),
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
┌───────────────────────────────────────┐
│  Statut                               │
├───────────────────────────────────────┤
│                                       │
│                 ♥                     │
│                                       │
│        Vies restantes : 3             │
│                                       │
│           Bonne chance                │
│                                       │
└───────────────────────────────────────┘
```

**Explication :** six `const` ont été ajoutés. Le plus important est celui du **constructeur** : `const GameApp({super.key})`. Sans lui, `runApp(const GameApp())` refuserait de compiler, et toute la classe deviendrait non constante. Ensuite, `Icon`, les deux `SizedBox` et le dernier `Text` sont figés : leurs paramètres sont tous des littéraux ou des constantes de la bibliothèque (`Icons.favorite` et `Colors.red` sont des `static const`). Le seul qui résiste est `Text('Vies restantes : $vies')`, car `vies` est une variable locale ordinaire. **Détail important** : ni la `Column`, ni le `Center`, ni le `Scaffold` ne peuvent être `const`, précisément parce que ce `Text` non constant est dans leur sous-arbre. Un `const` placé en haut exige que **tout** ce qui est en dessous soit constant. C'est pourquoi on descend le `const` jusqu'au niveau des feuilles. **Variante à connaître** : si l'on écrivait `const int vies = 3;`, alors `const Text('Vies restantes : $vies')` deviendrait légal, car Dart autorise l'interpolation dans une constante quand la valeur interpolée est elle-même une constante de compilation de type `num`, `String`, `bool` ou `null`. Dans une vraie application, `vies` viendra d'un état qui change : le cas non constant est donc le cas normal.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() => runApp(const GameApp());

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      // CORRECTION : l'écran devient une classe séparée.
      // Son build() reçoit un context situé SOUS le MaterialApp,
      // donc la remontée trouve bien le Theme installé ci-dessus.
      home: const EcranForet(),
    );
  }
}

class EcranForet extends StatelessWidget {
  const EcranForet({super.key});

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(
        backgroundColor: couleurs.primary,
        foregroundColor: couleurs.onPrimary,
        title: const Text('Forêt maudite'),
      ),
      body: const Center(child: Text('Niveau 3')),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Forêt maudite                        │  <- fond vert, texte clair
├───────────────────────────────────────┤
│                                       │
│              Niveau 3                 │
│                                       │
└───────────────────────────────────────┘
```

**Explication :** dans le code d'origine, `Theme.of(context)` était appelé dans le `build()` de `GameApp`. Or ce `context` désigne l'emplacement de `GameApp`, c'est-à-dire un point situé **au-dessus** du `MaterialApp`. Flutter remonte donc l'arbre à partir de là, ne rencontre aucun `Theme`, et se rabat silencieusement sur `ThemeData.fallback()`, dont la couleur primaire est bleue. Aucune exception n'est levée : c'est ce qui rend l'erreur difficile à trouver. Le schéma de la situation est celui de 44.16.7 : le thème est **plus bas** que le `context`, et un `context` ne regarde jamais vers le bas. La correction consiste à créer un point d'observation situé sous le `MaterialApp`. Extraire `EcranForet` dans sa propre classe crée exactement cela : un nouvel élément, donc un nouveau `BuildContext`, dont les ancêtres incluent cette fois le `Theme` posé par `MaterialApp`. On en profite pour stocker `Theme.of(context).colorScheme` dans une variable locale `couleurs`, ce qui évite trois remontées identiques et rend le code plus lisible. Deux alternatives existent — envelopper l'écran dans un `Builder`, ou utiliser le paramètre `builder:` de `MaterialApp` — mais l'extraction en classe est de loin la plus courante et la plus utile, puisqu'elle résout aussi les problèmes de lisibilité.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Salle du dragon',
      debugShowCheckedModeBanner: false,
      home: EcranSalle(),
    );
  }
}

class EcranSalle extends StatelessWidget {
  const EcranSalle({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Salle du dragon')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.local_fire_department,
              size: 72,
              color: Colors.orange,
            ),
            SizedBox(height: 16),
            Text(
              'Salle du dragon',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            Text(
              'Danger : extrême',
              style: TextStyle(color: Colors.red),
            ),
          ],
        ),
      ),
    );
  }
}

// Arbre de widgets de cet écran :
//
// GameApp
// └── MaterialApp
//     └── EcranSalle
//         └── Scaffold
//             ├── appBar: AppBar
//             │   └── title: Text('Salle du dragon')
//             └── body: Center
//                 └── Column
//                     ├── Icon(Icons.local_fire_department)
//                     ├── SizedBox(height: 16)
//                     ├── Text('Salle du dragon')
//                     └── Text('Danger : extrême')
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Salle du dragon                      │
├───────────────────────────────────────┤
│                                       │
│                ▲▲▲                    │  <- flamme orange, 72 px
│                                       │
│          Salle du dragon              │  <- 24 px, gras
│          Danger : extrême             │  <- rouge
│                                       │
└───────────────────────────────────────┘
```

**Explication :** l'exercice porte sur la lecture de l'indentation. Chaque niveau d'indentation du code correspond exactement à un niveau de profondeur dans l'arbre : le `Center` est enfant du `Scaffold` par son paramètre `body`, la `Column` est l'unique enfant du `Center` via `child`, et les quatre widgets suivants sont les éléments de la liste `children`, donc frères entre eux. Notez que `SizedBox(height: 16)` est un **vrai widget**, au même titre que les autres : en Flutter, un espace vide se décrit, il ne se devine pas. Notez aussi que `Center` centre à la fois horizontalement et verticalement son enfant, tandis que `mainAxisAlignment: MainAxisAlignment.center` centre les enfants de la `Column` sur son axe principal, qui est vertical. Les deux se combinent sans se contredire. Enfin, tout le corps est un unique `const` posé sur `Center` : rien dans ce sous-arbre ne dépend de l'exécution, donc l'ensemble est fabriqué une fois pour toutes à la compilation.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Inventaire',
      debugShowCheckedModeBanner: false,
      home: EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      // Un seul const en haut : les trois lignes deviennent constantes.
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            LigneObjet(icone: Icons.local_drink, libelle: 'Potion x3'),
            LigneObjet(icone: Icons.hardware, libelle: 'Épée x1'),
            LigneObjet(icone: Icons.paid, libelle: 'Pièces x12'),
          ],
        ),
      ),
    );
  }
}

/// Une ligne d'inventaire : une icône, un espace, un libellé.
class LigneObjet extends StatelessWidget {
  const LigneObjet({
    super.key,
    required this.icone,
    required this.libelle,
  });

  final IconData icone;
  final String libelle;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 6),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icone),
          const SizedBox(width: 8),
          Text(libelle),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Inventaire                           │
├───────────────────────────────────────┤
│                                       │
│      ▽  Potion x3                     │
│      ⚒  Épée x1                       │
│      ◉  Pièces x12                    │
│                                       │
└───────────────────────────────────────┘
```

**Explication :** la démarche est celle de 44.21.2. On sépare ce qui varie — l'icône et le libellé — de ce qui ne varie pas — la `Row`, l'espacement de 8 pixels, la marge verticale. Ce qui varie devient deux champs `final` alimentés par des paramètres nommés `required` ; ce qui ne varie pas devient le corps de `build()`. Trois points techniques méritent attention. **Premièrement**, le type de `icone` est `IconData`, et non `Icon` : `Icons.paid` est une donnée décrivant un glyphe, que le widget `Icon` sait afficher. Confondre les deux est une erreur classique. **Deuxièmement**, le constructeur est déclaré `const`, ce qui rend possible le `const` unique posé sur `Center` : les trois `LigneObjet` sont alors fabriquées à la compilation, et Flutter pourra court-circuiter leur comparaison à chaque reconstruction. **Troisièmement**, `mainAxisSize: MainAxisSize.min` demande à la `Row` de ne prendre que la largeur de son contenu, au lieu de s'étirer sur toute la largeur ; combiné à `crossAxisAlignment: CrossAxisAlignment.start` sur la `Column`, cela aligne proprement les trois lignes à gauche les unes sous les autres. Bénéfice concret de l'extraction : ajouter un quatrième objet ne demande plus qu'une ligne.


---

### Correction 6

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'child et children',
      debugShowCheckedModeBanner: false,
      home: EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: const Center(
        // ERREUR 1 corrigée : Center prend UN SEUL enfant.
        // On enveloppe les deux textes dans une Column.
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Inventaire'),
            SizedBox(height: 8),
            Text('3 objets'),
            SizedBox(height: 24),
            // La Row demandée en fin d'énoncé.
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Icon(Icons.local_drink, size: 36, color: Colors.pink),
                Icon(Icons.hardware, size: 36, color: Colors.blueGrey),
                Icon(Icons.paid, size: 36, color: Colors.amber),
              ],
            ),
          ],
        ),
      ),
      // ERREURS 2 et 3 corrigées : Row prend `children`, et c'est une LISTE.
      bottomNavigationBar: const Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Padding(
            padding: EdgeInsets.all(12),
            child: Text('Barre du bas'),
          ),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Inventaire                           │
├───────────────────────────────────────┤
│                                       │
│             Inventaire                │
│              3 objets                 │
│                                       │
│      ▽          ⚒          ◉          │
│                                       │
├───────────────────────────────────────┤
│            Barre du bas               │
└───────────────────────────────────────┘
```

**Explication :** les trois erreurs et leurs messages exacts.

| Erreur d'origine | Message de l'analyseur | Correction |
| --- | --- | --- |
| `Center(child: [Text(...), Text(...)])` | `The argument type 'List<Text>' can't be assigned to the parameter type 'Widget?'` | envelopper la liste dans une `Column` |
| `Row(child: ...)` | `The named parameter 'child' isn't defined` | `Row` n'expose pas `child`, mais `children` |
| `Row(children: Text(...))` (après la correction naïve) | `The argument type 'Text' can't be assigned to the parameter type 'List<Widget>'` | ajouter les crochets : `children: [Text(...)]` |

La logique est toujours la même, et elle est purement typée : `child` est de type `Widget?`, `children` est de type `List<Widget>`. Le compilateur Dart vérifie donc l'erreur avant même l'exécution, ce qui est une bonne nouvelle : ces trois fautes ne passeront jamais en production. Deux remarques complémentaires. **Un enfant unique dans `children` s'écrit quand même entre crochets** : `children: [Text('A')]`. Il n'existe aucune exception. **Un `bottomNavigationBar` accepte n'importe quel widget**, pas seulement une `NavigationBar` : ici une simple `Row` fait l'affaire, et le `Padding` évite que le texte ne colle au bord de l'écran.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Badges',
      debugShowCheckedModeBanner: false,
      home: EcranBadges(),
    );
  }
}

class EcranBadges extends StatelessWidget {
  const EcranBadges({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Badges de niveau')),
      body: const Center(
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            // A : seul le paramètre obligatoire.
            BadgeNiveau(niveau: 1),
            // B : on remplace la couleur par défaut.
            BadgeNiveau(niveau: 7, couleur: Colors.orange),
            // C : les trois paramètres, dans un ordre libre.
            BadgeNiveau(titre: 'Boss', couleur: Colors.red, niveau: 42),
          ],
        ),
      ),
    );
  }
}

class BadgeNiveau extends StatelessWidget {
  const BadgeNiveau({
    super.key,
    required this.niveau,              // A : obligatoire
    this.couleur = Colors.deepPurple,  // B : facultatif, valeur par défaut
    this.titre,                        // C : facultatif, nullable
  });

  final int niveau;
  final Color couleur;
  final String? titre;   // le `?` est OBLIGATOIRE pour le cas C

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Container(
          width: 72,
          height: 72,
          alignment: Alignment.center,
          decoration: BoxDecoration(
            color: couleur,
            shape: BoxShape.circle,
          ),
          child: Text(
            '$niveau',
            style: const TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        // Si `titre` est nul, on n'ajoute rien du tout.
        if (titre != null) ...[
          const SizedBox(height: 8),
          Text(titre!, style: const TextStyle(fontWeight: FontWeight.w600)),
        ],
      ],
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Badges de niveau                     │
├───────────────────────────────────────┤
│                                       │
│    ╭───╮      ╭───╮      ╭───╮        │
│    │ 1 │      │ 7 │      │42 │        │
│    ╰───╯      ╰───╯      ╰───╯        │
│    violet     orange      rouge       │
│                           Boss        │
│                                       │
└───────────────────────────────────────┘
```

**Explication :** l'exercice fait travailler les trois formes de paramètre nommé du chapitre 09, revues en 44.24.3. **`required this.niveau`** garantit que la valeur sera toujours fournie : le champ peut donc être déclaré `final int niveau;`, non nullable, et l'oubli de ce paramètre est une erreur de compilation. **`this.couleur = Colors.deepPurple`** rend le paramètre facultatif tout en gardant un champ non nullable, puisque la valeur par défaut est toujours appliquée ; notez que cette valeur par défaut doit obligatoirement être une constante de compilation, ce que `Colors.deepPurple` est bien. **`this.titre`** sans `required` ni valeur par défaut impose un champ nullable `String? titre;` — sans le point d'interrogation, l'analyseur refuserait le code. Le troisième appel montre en outre que l'ordre des paramètres nommés est totalement libre : `titre`, `couleur` puis `niveau` fonctionne aussi bien que l'ordre de déclaration. Enfin, la construction `if (titre != null) ...[ ... ]` combine un `if` de collection et l'opérateur de diffusion `...` du chapitre 06 : elle insère **deux** widgets dans la liste, ou aucun. À l'intérieur du bloc, `titre!` est légitime puisque le test qui précède garantit la non-nullité, ce que l'analyseur de flux de Dart ne peut pas toujours prouver seul sur un champ de classe.

---

### Correction 8

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
      title: 'Fiche du boss',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      home: const EcranBoss(),
    );
  }
}

/// L'écran : il assemble, il ne décore pas. Moins de vingt lignes.
class EcranBoss extends StatelessWidget {
  const EcranBoss({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Fiche du boss'),
        backgroundColor: Theme.of(context).colorScheme.primary,
        foregroundColor: Theme.of(context).colorScheme.onPrimary,
      ),
      body: const Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            EnTeteBoss(nom: 'Gorthar', titre: 'Seigneur des cendres'),
            SizedBox(height: 24),
            LigneCaracteristique(icone: Icons.fitness_center, libelle: 'Force', valeur: '95'),
            LigneCaracteristique(icone: Icons.speed, libelle: 'Vitesse', valeur: '40'),
            LigneCaracteristique(icone: Icons.security, libelle: 'Armure', valeur: '80'),
            LigneCaracteristique(icone: Icons.whatshot, libelle: 'Dégâts', valeur: '120'),
            Spacer(),
            PiedDePage(message: 'Ne combattez jamais ce boss seul.'),
          ],
        ),
      ),
    );
  }
}

/// L'en-tête : icône, nom, titre.
class EnTeteBoss extends StatelessWidget {
  const EnTeteBoss({super.key, required this.nom, required this.titre});

  final String nom;
  final String titre;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Row(
      children: [
        Icon(Icons.local_fire_department, size: 64, color: theme.colorScheme.primary),
        const SizedBox(width: 16),
        Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(nom, style: theme.textTheme.headlineSmall),
            Text(titre, style: theme.textTheme.bodyMedium),
          ],
        ),
      ],
    );
  }
}

/// Une caractéristique, réutilisée quatre fois.
class LigneCaracteristique extends StatelessWidget {
  const LigneCaracteristique({
    super.key,
    required this.icone,
    required this.libelle,
    required this.valeur,
  });

  final IconData icone;
  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
      margin: const EdgeInsets.symmetric(vertical: 5),
      decoration: BoxDecoration(
        color: couleurs.surfaceContainerHighest,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: [
          Icon(icone, color: couleurs.primary),
          const SizedBox(width: 12),
          Text(libelle),
          const Spacer(),
          Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}

/// Le pied de page : un avertissement centré.
class PiedDePage extends StatelessWidget {
  const PiedDePage({super.key, required this.message});

  final String message;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Container(
      width: double.infinity,
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: couleurs.errorContainer,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Text(
        message,
        textAlign: TextAlign.center,
        style: TextStyle(color: couleurs.onErrorContainer),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│  Fiche du boss                        │  <- fond rouge
├───────────────────────────────────────┤
│  ▲▲▲  Gorthar                         │
│       Seigneur des cendres            │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │ ⚖ Force                     95  │  │
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ➤ Vitesse                   40  │  │
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ⛨ Armure                    80  │  │
│  ╰─────────────────────────────────╯  │
│  ╭─────────────────────────────────╮  │
│  │ ✷ Dégâts                   120  │  │
│  ╰─────────────────────────────────╯  │
│                                       │
│  ╭─────────────────────────────────╮  │
│  │ Ne combattez jamais ce boss     │  │
│  │            seul.                │  │
│  ╰─────────────────────────────────╯  │
└───────────────────────────────────────┘
```

**Explication :** cet exercice met en pratique les sections 44.21 à 44.23 d'un seul coup. Le `build()` de `EcranBoss` tient en dix-sept lignes et se lit comme un sommaire : un en-tête, quatre caractéristiques, un espace élastique, un pied de page. Personne n'a besoin de lire les autres classes pour comprendre la structure de l'écran, et c'est précisément le but. **Sur les couleurs** : aucune n'est codée en dur. `colorScheme.primary` pour les accents, `surfaceContainerHighest` pour le fond des blocs, `errorContainer` et `onErrorContainer` pour l'avertissement. Changer `seedColor: Colors.red` en `Colors.indigo` retinte l'écran entier de façon cohérente, sans toucher une seule autre ligne. **Sur `const`** : le `Padding` du corps est `const`, ce qui rend constants l'en-tête, les quatre caractéristiques, le `Spacer` et le pied de page — soit la quasi-totalité de l'arbre. C'est possible parce que ces widgets ne reçoivent que des littéraux, et parce que la lecture du thème se fait **à l'intérieur** de leur `build()`, donc à l'exécution, ce qui n'empêche nullement leur construction constante. **Sur `Spacer`** : il absorbe tout l'espace vertical restant et pousse le pied de page en bas de l'écran, sans qu'il soit nécessaire de connaître la hauteur de l'appareil. **Sur `width: double.infinity`** : il force le pied de page à occuper toute la largeur disponible, ce qui rend le `textAlign: TextAlign.center` visible.


---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Débordement',
      debugShowCheckedModeBanner: false,
      home: EcranDebordement(),
    );
  }
}

class EcranDebordement extends StatelessWidget {
  const EcranDebordement({super.key});

  static const String texte =
      'Attention : le dragon ancestral du donjon infini approche de la salle du trône';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Trois corrections')),
      body: const Padding(
        padding: EdgeInsets.all(12),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            // 1. Expanded seul : le texte occupe la place restante
            //    et revient à la ligne autant de fois que nécessaire.
            Row(
              children: [
                Icon(Icons.warning, size: 48),
                Expanded(
                  child: Text(texte, style: TextStyle(fontSize: 20)),
                ),
                Icon(Icons.warning, size: 48),
              ],
            ),
            Divider(),
            // 2. Expanded + ellipsis : une seule ligne, coupée par « ... ».
            Row(
              children: [
                Icon(Icons.warning, size: 48),
                Expanded(
                  child: Text(
                    texte,
                    style: TextStyle(fontSize: 20),
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
                Icon(Icons.warning, size: 48),
              ],
            ),
            Divider(),
            // 3. Expanded + maxLines : deux lignes au maximum, puis « ... ».
            Row(
              children: [
                Icon(Icons.warning, size: 48),
                Expanded(
                  child: Text(
                    texte,
                    style: TextStyle(fontSize: 20),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
                Icon(Icons.warning, size: 48),
              ],
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
┌───────────────────────────────────────┐
│  Trois corrections                    │
├───────────────────────────────────────┤
│ ⚠ Attention : le dragon ancestral  ⚠ │
│   du donjon infini approche de la     │  <- 1 : tout le texte, sur 3 lignes
│   salle du trône                      │
│ ───────────────────────────────────── │
│ ⚠ Attention : le dragon ances...   ⚠ │  <- 2 : une seule ligne, coupée
│ ───────────────────────────────────── │
│ ⚠ Attention : le dragon ancestral  ⚠ │
│   du donjon infini approche de...     │  <- 3 : deux lignes, puis coupé
└───────────────────────────────────────┘
```

**Explication :** l'erreur d'origine venait d'un `Text` non contraint dans une `Row`. Une `Row` propose à chacun de ses enfants une largeur **illimitée**, puis constate que la somme des largeurs demandées dépasse la sienne : c'est le message `A RenderFlex overflowed by N pixels on the right`, matérialisé à l'écran par la bande jaune et noire. Le `Text` ne peut pas deviner seul qu'il doit revenir à la ligne, puisqu'on ne lui a donné aucune limite. **`Expanded` est la correction de fond** : il transforme la contrainte « prends la largeur que tu veux » en « prends exactement la largeur restante ». Le `Text` connaît alors sa largeur et se coupe naturellement en plusieurs lignes. Les deux icônes de 48 pixels sont mesurées d'abord ; ce qui reste est attribué au texte. **Les différences entre les trois versions ne portent que sur le traitement du dépassement vertical.** La version 1 affiche l'intégralité du texte, sur autant de lignes que nécessaire, ce qui fait grandir la hauteur de la ligne — parfait pour un message important. La version 2 impose implicitement une seule ligne et remplace la fin par des points de suspension : c'est le comportement des listes, où toutes les lignes doivent avoir la même hauteur. La version 3 est le compromis : au plus deux lignes, puis des points de suspension. Notez que `maxLines` sans `overflow` couperait le texte net, sans indiquer visuellement qu'il manque quelque chose : les deux paramètres vont ensemble. **Le réflexe à retenir** : débordement horizontal dans une `Row` → `Expanded` sur l'enfant élastique. Débordement vertical dans une `Column` → `Expanded` également, ou `SingleChildScrollView` si le contenu doit rester intégralement lisible.

---

### Correction 10 — Projet : la fiche de personnage

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const GameApp());
}

// ---------------------------------------------------------------------------
// 1. La racine de l'application : thème et écran d'accueil.
// ---------------------------------------------------------------------------
class GameApp extends StatelessWidget {
  const GameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Donjon Infini',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranFiche(),
    );
  }
}

// ---------------------------------------------------------------------------
// 2. L'écran : il assemble, rien de plus.
// ---------------------------------------------------------------------------
class EcranFiche extends StatelessWidget {
  const EcranFiche({super.key});

  /// Les compétences du héros. La liste est constante : rien ne bouge encore.
  static const List<String> competences = ['Épée', 'Feu', 'Soin', 'Vol'];

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Fiche du héros'),
        centerTitle: true,
        backgroundColor: couleurs.primary,
        foregroundColor: couleurs.onPrimary,
        actions: const [
          Icon(Icons.favorite),
          SizedBox(width: 16),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const EnTetePersonnage(
              nom: 'Alex',
              classe: 'Guerrier du Nord',
              niveau: 7,
            ),
            const SizedBox(height: 24),
            const BarreStatistique(
              libelle: 'Points de vie',
              valeur: 87,
              valeurMax: 100,
              couleur: Colors.red,
            ),
            const BarreStatistique(
              libelle: 'Énergie',
              valeur: 40,
              valeurMax: 60,
              couleur: Colors.orange,
            ),
            const BarreStatistique(
              libelle: 'Expérience',
              valeur: 12450,
              valeurMax: 20000,
              couleur: Colors.blue,
            ),
            const SizedBox(height: 24),
            Text('Compétences', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                // Une puce par compétence, avec une identité stable.
                for (final String nom in competences)
                  PuceCompetence(key: ValueKey<String>(nom), libelle: nom),
              ],
            ),
            const Spacer(),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          print('Le combat commence');
        },
        backgroundColor: couleurs.primary,
        foregroundColor: couleurs.onPrimary,
        child: const Icon(Icons.play_arrow),
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: 2,
        onDestinationSelected: (int index) {
          print('Onglet $index');
        },
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Accueil'),
          NavigationDestination(icon: Icon(Icons.inventory_2), label: 'Sac'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Héros'),
        ],
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// 3. L'en-tête : avatar, nom, classe, niveau.
// ---------------------------------------------------------------------------
class EnTetePersonnage extends StatelessWidget {
  const EnTetePersonnage({
    super.key,
    required this.nom,
    required this.classe,
    required this.niveau,
  });

  final String nom;
  final String classe;
  final int niveau;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Row(
      children: [
        Container(
          width: 72,
          height: 72,
          alignment: Alignment.center,
          decoration: BoxDecoration(
            color: theme.colorScheme.primaryContainer,
            shape: BoxShape.circle,
          ),
          child: Icon(
            Icons.shield,
            size: 40,
            color: theme.colorScheme.onPrimaryContainer,
          ),
        ),
        const SizedBox(width: 16),
        Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(nom, style: theme.textTheme.headlineSmall),
            Text(classe, style: theme.textTheme.bodyMedium),
            const SizedBox(height: 4),
            Text(
              'Niveau $niveau',
              style: theme.textTheme.labelLarge?.copyWith(
                color: theme.colorScheme.primary,
              ),
            ),
          ],
        ),
      ],
    );
  }
}

// ---------------------------------------------------------------------------
// 4. Une barre de statistique, réutilisée trois fois.
// ---------------------------------------------------------------------------
class BarreStatistique extends StatelessWidget {
  const BarreStatistique({
    super.key,
    required this.libelle,
    required this.valeur,
    required this.couleur,
    this.valeurMax = 100,
  });

  final String libelle;
  final int valeur;
  final int valeurMax;
  final Color couleur;

  @override
  Widget build(BuildContext context) {
    final double ratio = valeur / valeurMax;

    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              Text(libelle),
              const Spacer(),
              Text(
                '$valeur / $valeurMax',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ],
          ),
          const SizedBox(height: 6),
          ClipRRect(
            borderRadius: BorderRadius.circular(8),
            child: LinearProgressIndicator(
              value: ratio,
              minHeight: 12,
              color: couleur,
              backgroundColor: Colors.black12,
            ),
          ),
        ],
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// 5. Une puce de compétence : icône + libellé.
// ---------------------------------------------------------------------------
class PuceCompetence extends StatelessWidget {
  const PuceCompetence({
    super.key,
    required this.libelle,
    this.icone = Icons.auto_awesome,
  });

  final String libelle;
  final IconData icone;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: couleurs.secondaryContainer,
        borderRadius: BorderRadius.circular(20),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icone, size: 16, color: couleurs.onSecondaryContainer),
          const SizedBox(width: 6),
          Text(
            libelle,
            style: TextStyle(color: couleurs.onSecondaryContainer),
          ),
        ],
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌───────────────────────────────────────┐
│          Fiche du héros        ♥      │  <- fond violet, titre centré
├───────────────────────────────────────┤
│  ╭───╮  Alex                          │
│  │ ▲ │  Guerrier du Nord              │
│  ╰───╯  Niveau 7                      │
│                                       │
│  Points de vie             87 / 100   │
│  ████████████████████████░░░░         │  <- rouge
│                                       │
│  Énergie                    40 / 60   │
│  █████████████████░░░░░░░░░░░         │  <- orange
│                                       │
│  Expérience           12450 / 20000   │
│  ██████████████████░░░░░░░░░░         │  <- bleu
│                                       │
│  Compétences                          │
│  (✦ Épée) (✦ Feu) (✦ Soin) (✦ Vol)    │
│                                       │
│                            ╭───╮      │
│                            │ ▶ │      │
│                            ╰───╯      │
├───────────────────────────────────────┤
│    ⌂          ▣          ☺            │
│ Accueil      Sac       Héros          │
└───────────────────────────────────────┘
```

**Résultat dans la console, après avoir touché le bouton puis l'onglet « Sac » :**

```text
Le combat commence
Onglet 1
```

**Explication :** ce projet réunit l'intégralité du chapitre. Reprenons les points dans l'ordre.

**L'architecture.** Quatre classes de widgets, plus la racine. Le `build()` de `EcranFiche` ne contient aucune décoration : il place un en-tête, trois barres, un titre de section, une rangée de puces et un espace élastique. Chaque classe porte un nom métier — `EnTetePersonnage`, `BarreStatistique`, `PuceCompetence` — et ces noms apparaîtront tels quels dans l'inspecteur de widgets, ce qui rendra tout débogage immédiat. C'est le bénéfice décrit en 44.22.5.

**L'arbre obtenu.**

```text
GameApp
└── MaterialApp
    └── EcranFiche
        └── Scaffold
            ├── AppBar
            ├── Padding
            │   └── Column
            │       ├── EnTetePersonnage
            │       ├── BarreStatistique (Points de vie)
            │       ├── BarreStatistique (Énergie)
            │       ├── BarreStatistique (Expérience)
            │       ├── Text('Compétences')
            │       ├── Row
            │       │   ├── PuceCompetence(key: ValueKey('Épée'))
            │       │   ├── PuceCompetence(key: ValueKey('Feu'))
            │       │   ├── PuceCompetence(key: ValueKey('Soin'))
            │       │   └── PuceCompetence(key: ValueKey('Vol'))
            │       └── Spacer
            ├── FloatingActionButton
            └── NavigationBar
```

**Le thème.** Aucune couleur d'interface n'est écrite en dur : `primary` et `onPrimary` pour la barre et le bouton, `primaryContainer` et `onPrimaryContainer` pour l'avatar, `secondaryContainer` et `onSecondaryContainer` pour les puces. Seules les trois couleurs de barres de statistique sont explicites, parce qu'elles portent un sens de jeu — le rouge pour la vie, l'orange pour l'énergie — et non un rôle de marque. Remplacez `seedColor: Colors.deepPurple` par `Colors.teal` : tout l'écran change de tonalité, et les trois barres gardent volontairement leur signification.

**Les `const`.** L'en-tête, les trois barres et les `SizedBox` sont `const`. En revanche, la `Column` ne l'est pas, car elle contient un `Text` dont le style est lu dans le thème à l'exécution, ainsi qu'une boucle `for`. C'est exactement la règle de 44.19.5 : on descend le `const` jusqu'au niveau où tout redevient constant.

**La boucle et les clés.** `for (final String nom in competences)` est un `for` **de collection** (chapitre 06) : il produit directement des éléments dans la liste `children`, sans `add()` ni variable intermédiaire. Chaque puce reçoit `key: ValueKey<String>(nom)`. Dans cet écran figé, la clé ne change rien au rendu ; elle prépare le terrain pour le jour où les compétences seront ajoutées, retirées ou réordonnées, cas dans lequel Flutter doit pouvoir suivre chaque puce individuellement (44.20.3).

**Les paramètres.** `BarreStatistique` illustre les trois formes du chapitre 09 : `libelle`, `valeur` et `couleur` sont `required`, tandis que `valeurMax` possède la valeur par défaut `100`. `PuceCompetence` a une valeur par défaut pour son icône, ce qui permet d'écrire `PuceCompetence(libelle: 'Feu')` sans plus.

**Les détails d'affichage.** `ClipRRect` arrondit les coins de la barre de progression, qui est rectangulaire par défaut. `?.copyWith(...)` part d'un style du thème et n'en modifie qu'une propriété ; le `?.` est indispensable car `labelLarge` est de type `TextStyle?` (chapitre 12). `mainAxisSize: MainAxisSize.min` empêche chaque puce de s'étirer sur toute la largeur.

**Une limite assumée.** La rangée de puces est une `Row` : avec quatre libellés courts, elle tient sur un téléphone standard. Ajoutez-en trois de plus, ou passez l'appareil en très petite largeur, et vous obtiendrez un `A RenderFlex overflowed by N pixels on the right`. La solution propre est le widget `Wrap`, qui passe automatiquement à la ligne suivante ; il est présenté au chapitre 46. Reproduire volontairement ce débordement, puis lire le message d'erreur avec la méthode de 44.27.10, est le meilleur exercice supplémentaire que vous puissiez vous donner.

**Ce qui manque encore.** Rien ne réagit. Le bouton écrit une ligne dans la console, la barre de navigation aussi, et l'écran reste identique. Aucune des valeurs affichées ne peut changer, parce que tous nos champs sont `final` et que nos widgets sont immuables. C'est précisément le mur que le chapitre suivant fait tomber.

---

## Et maintenant ?

Vous savez désormais lire et écrire un arbre de widgets. Vous connaissez la structure d'un écran Material — `MaterialApp`, `Scaffold`, `AppBar`, `body` —, vous comprenez ce qu'est réellement un `BuildContext` et pourquoi il ne regarde que vers le haut, vous savez remonter chercher un thème avec `Theme.of(context)`, et vous avez découvert les trois arbres que Flutter entretient en coulisses.

Vous savez surtout organiser votre code : extraire un widget dans sa propre classe plutôt que dans une méthode, écrire un constructeur `const` à paramètres nommés, distinguer `child` de `children`, et découper un écran de cent vingt lignes en cinq classes lisibles. Vous savez ouvrir l'inspecteur de widgets et lire un message d'erreur de rendu sans paniquer.

Mais tous vos écrans sont morts. Le score reste à 12 450 pour l'éternité, les points de vie ne descendent jamais, et le bouton « Commencer la partie » se contente d'écrire une ligne dans la console. La raison est claire, et vous la connaissez maintenant : un widget est immuable, tous ses champs sont `final`, et rien dans ce que nous avons écrit ne peut changer après la construction.

Le chapitre suivant lève cette limite. Vous y découvrirez pourquoi Flutter distingue deux familles de widgets, comment un `StatefulWidget` conserve des données entre deux reconstructions grâce à un objet `State` qui, lui, survit, ce que fait vraiment `setState()`, quand `initState()` et `dispose()` sont appelés, et comment faire remonter un état vers un parent commun. C'est le chapitre qui transforme vos maquettes en applications.

Rendez-vous au chapitre 45 : [45-PARTIE-1B—STATELESSWIDGET-ET-STATEFULWIDGET.md](./45-PARTIE-1B—STATELESSWIDGET-ET-STATEFULWIDGET.md)
