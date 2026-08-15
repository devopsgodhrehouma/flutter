# PARTIE 1B — FLUTTER
# CHAPITRE 50 — NAVIGATION ET ROUTES

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** chapitre 49 — Boutons, formulaires et validation
> **Ce que vous saurez faire à la fin :** construire une application à plusieurs écrans, passer des données dans les deux sens entre ces écrans, organiser la navigation par onglets et par tiroir, et remplacer le `Navigator` par `go_router` quand l'application devient sérieuse.

---

## 50.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi une application réelle possède plusieurs écrans ;
- décrire la pile de navigation et dessiner son état à tout instant ;
- ouvrir un écran avec `Navigator.push()` ;
- construire une route avec `MaterialPageRoute` ;
- fermer un écran avec `Navigator.pop()` ;
- expliquer ce que fait le bouton retour du système et pourquoi il fonctionne tout seul ;
- passer des données à l'écran suivant par le constructeur ;
- récupérer une valeur au retour avec `await Navigator.push` ;
- renvoyer une valeur avec `Navigator.pop(context, valeur)` ;
- typer correctement le résultat d'une navigation, en réutilisant les `Future` du chapitre 15 ;
- déclarer des routes nommées dans `MaterialApp.routes` ;
- naviguer avec `Navigator.pushNamed()` ;
- transmettre des `arguments` et les lire avec `ModalRoute.of(context)` ;
- générer des routes dynamiquement avec `onGenerateRoute` ;
- afficher une page 404 avec `onUnknownRoute` ;
- choisir l'écran de démarrage avec `initialRoute` ;
- remplacer l'écran courant avec `pushReplacement` ;
- vider la pile avec `pushAndRemoveUntil` ;
- remonter à un écran précis avec `popUntil` ;
- tester la pile avec `canPop` et `maybePop` ;
- bloquer le retour avec `PopScope` ;
- afficher une confirmation « voulez-vous vraiment quitter ? » ;
- animer une image d'un écran à l'autre avec `Hero` ;
- écrire votre propre transition avec `PageRouteBuilder` ;
- construire des onglets avec `TabBar`, `TabBarView` et `DefaultTabController` ;
- construire une barre de navigation basse avec `NavigationBar` (Material 3) ;
- conserver l'état de chaque onglet avec `IndexedStack` ;
- ajouter un tiroir latéral avec `Drawer` et `NavigationDrawer` ;
- ajouter un tiroir de droite avec `endDrawer` ;
- énoncer les trois limites du `Navigator` 1.0 ;
- expliquer pourquoi `go_router` existe ;
- installer et configurer `go_router` ;
- déclarer des routes, des sous-routes, des paramètres de chemin et de requête ;
- protéger une zone privée avec une redirection d'authentification ;
- comprendre ce que devient l'URL du navigateur sur le Web ;
- choisir en connaissance de cause entre `Navigator` et `go_router`.

---

## 50.0.1 — Où nous en sommes

Depuis le chapitre 43, toutes vos applications tenaient dans **un seul écran**.

Vous avez appris à composer des widgets (chapitre 44), à les faire changer d'état (chapitre 45), à les placer (chapitre 46), à les habiller (chapitre 47), à en afficher des listes (chapitre 48) et à saisir des données (chapitre 49).

Un seul écran, c'est suffisant pour une calculatrice. Ce n'est pas suffisant pour :

```text
- une liste de joueurs -> la fiche d'un joueur
- un catalogue d'armes -> le détail d'une arme -> le panier
- un écran de connexion -> le menu principal -> une partie
- un menu -> les réglages -> les réglages audio
```

Dès qu'il y a une flèche `->` dans votre maquette, il y a **navigation**.

Ce chapitre est celui qui transforme vos maquettes en applications.

---

## 50.1 — Une application a plusieurs écrans

Ouvrez n'importe quelle application de votre téléphone. Comptez les écrans. Vous en trouverez rarement moins de dix.

Un **écran** (on dit aussi une **page**, ou en Flutter une **route**) est un widget qui occupe la totalité de la surface visible et qui a sa propre `AppBar`, son propre contenu, son propre rôle.

Prenons un exemple de fil rouge que nous garderons tout le chapitre : une petite application de jeu vidéo.

```text
Écran 1 : la liste des joueurs de la guilde
Écran 2 : la fiche détaillée d'un joueur
Écran 3 : le formulaire d'ajout d'un joueur
```

Trois écrans. Trois classes Dart. Trois `Scaffold`.

Un débutant est tenté de simuler tout cela avec un booléen et un `setState` :

```dart
// Version « faux » : un seul écran qui change de contenu.
@override
Widget build(BuildContext context) {
  if (_surLeDetail) {
    return Scaffold(
      appBar: AppBar(title: const Text('Détail')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => setState(() => _surLeDetail = false),
          child: const Text('Revenir'),
        ),
      ),
    );
  }
  return Scaffold(
    appBar: AppBar(title: const Text('Guilde')),
    body: Center(
      child: ElevatedButton(
        onPressed: () => setState(() => _surLeDetail = true),
        child: const Text('Voir Aria'),
      ),
    ),
  );
}
```

**Résultat :**

```text
L'écran affiche « Voir Aria ».
On appuie : l'écran affiche « Aria, niveau 42 ».
On appuie sur « Revenir » : on retourne au premier écran.
```

Cela **fonctionne**. Et pourtant c'est faux. Voici pourquoi.

---

## 50.1.1 — Pourquoi un booléen ne suffit pas

Testez l'application ci-dessus sur un téléphone Android et appuyez sur le **bouton retour du système** pendant que vous êtes sur le détail.

L'application se **ferme**.

Le système ne sait pas que vous êtes « sur le détail ». Pour lui, il n'y a qu'un écran, donc revenir en arrière signifie quitter l'application.

Ajoutez maintenant :

```text
- une animation de glissement entre les deux écrans
- un titre d'AppBar avec une flèche de retour automatique
- un troisième écran, puis un quatrième
- le retour d'une valeur du détail vers la liste
- une URL /joueur/42 quand l'application tourne dans un navigateur
```

Avec des booléens, chacun de ces points est un cauchemar. Avec le `Navigator`, chacun de ces points est gratuit ou coûte une ligne.

> **Règle :** on ne simule jamais la navigation avec `setState`. On utilise le `Navigator`.

---

## 50.2 — La pile de navigation

Flutter gère les écrans avec une **pile** (*stack*). C'est exactement la structure de données que vous connaissez depuis la PARTIE 1A : on empile au sommet, on dépile au sommet.

L'écran visible est **toujours celui du sommet**. Les autres existent encore, en mémoire, mais sont cachés dessous.

```text
        ÉTAT INITIAL — l'application vient de démarrer

        ┌───────────────────────────────┐
        │                               │
        │                               │
        │                               │
        │   sommet ->  [ Liste ]        │  <- écran visible
        │                               │
        └───────────────────────────────┘
              base de la pile

        La pile contient 1 route.
```

L'utilisateur appuie sur « Aria ». On **empile** l'écran de détail :

```text
        APRÈS Navigator.push(DetailJoueur)

        ┌───────────────────────────────┐
        │   sommet ->  [ Détail ]       │  <- écran visible
        │                               │
        │              [ Liste ]        │     caché, mais vivant
        │                               │
        └───────────────────────────────┘

        La pile contient 2 routes.
```

L'utilisateur appuie sur la flèche retour. On **dépile** :

```text
        APRÈS Navigator.pop()

        ┌───────────────────────────────┐
        │                               │
        │                               │
        │   sommet ->  [ Liste ]        │  <- écran visible à nouveau
        │                               │
        └───────────────────────────────┘

        La pile contient 1 route.
        L'objet Détail est détruit ; dispose() est appelé.
```

Trois idées à retenir dès maintenant :

1. **Le sommet est visible, le reste est caché.** Pas détruit : caché.
2. **`push` ajoute au sommet, `pop` retire le sommet.** Il n'y a pas d'autre façon de faire.
3. **Quand la pile est vide, l'application se ferme.** C'est pour cela qu'on ne peut pas dépiler la dernière route.

---

## 50.2.1 — Un parcours complet dessiné

Voici une session utilisateur entière, écran par écran.

```text
1. Démarrage
   [ Liste ]

2. Tape sur « Aria »          -> push(Détail Aria)
   [ Détail Aria ]
   [ Liste       ]

3. Tape sur « Inventaire »    -> push(Inventaire)
   [ Inventaire  ]
   [ Détail Aria ]
   [ Liste       ]

4. Tape sur « Épée longue »   -> push(Détail Arme)
   [ Détail Arme ]
   [ Inventaire  ]
   [ Détail Aria ]
   [ Liste       ]

5. Bouton retour              -> pop()
   [ Inventaire  ]
   [ Détail Aria ]
   [ Liste       ]

6. Bouton retour              -> pop()
   [ Détail Aria ]
   [ Liste       ]

7. Bouton retour              -> pop()
   [ Liste       ]

8. Bouton retour              -> la pile ne contient qu'une route
                                 l'application passe en arrière-plan
```

Gardez ce schéma sous les yeux : toutes les méthodes du chapitre se décrivent comme une transformation de cette pile.

---

## 50.2.2 — Qui possède la pile ?

La pile est gérée par un widget appelé `Navigator`. Vous ne l'écrivez presque jamais vous-même, parce que `MaterialApp` en crée un automatiquement à la racine de votre application.

```text
MaterialApp
  └── Navigator                <- créé automatiquement
        ├── Route 1  (Liste)
        ├── Route 2  (Détail)
        └── Route 3  (Inventaire)
```

C'est pour cela que le premier écran s'appelle `home` : `MaterialApp` le place tout seul au fond de la pile du `Navigator`.

---

## 50.3 — `Navigator.push()`

`push` empile une nouvelle route.

Signature officielle :

```dart
static Future<T?> push<T extends Object?>(
  BuildContext context,
  Route<T> route,
)
```

Lisez-la lentement, elle contient déjà deux informations importantes :

- il faut un `BuildContext` : le `Navigator` est retrouvé **en remontant l'arbre de widgets** depuis ce `context` ;
- elle retourne un `Future<T?>` : nous nous en servirons en 50.8 pour récupérer un résultat.

Voici le premier exemple complet de navigation réelle. Il remplace le booléen de 50.1.

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
      title: 'Guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => const EcranDetail(),
              ),
            );
          },
          child: const Text('Voir Aria'),
        ),
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Détail')),
      body: const Center(
        child: Text('Aria, niveau 42'),
      ),
    );
  }
}
```

**Résultat :**

```text
Écran 1 : une AppBar « Guilde », un bouton « Voir Aria ».
On appuie : l'écran de détail arrive en glissant depuis la droite.
Son AppBar affiche « Détail » ET une flèche de retour à gauche,
que nous n'avons pas écrite.
On appuie sur la flèche : retour à la liste, animation inverse.
Le bouton retour physique d'Android fonctionne aussi.
```

Trois choses gratuites que le booléen ne donnait pas : l'animation, la flèche de retour, le bouton système.

---

## 50.3.1 — `Navigator.of(context).push(...)`

Vous rencontrerez très souvent cette écriture :

```dart
Navigator.of(context).push(
  MaterialPageRoute(builder: (context) => const EcranDetail()),
);
```

`Navigator.of(context)` retrouve l'objet `NavigatorState` et on appelle `push` dessus.

C'est **exactement équivalent** à `Navigator.push(context, ...)`. La forme statique est un raccourci qui appelle la forme longue.

Utilisez celle que vous voulez, mais soyez cohérent dans un projet. Dans ce chapitre nous utiliserons `Navigator.push(context, ...)`, plus court.

Il existe un cas où la forme longue est utile : l'option `rootNavigator`.

```dart
Navigator.of(context, rootNavigator: true).push(...);
```

Elle sert quand plusieurs `Navigator` sont imbriqués (onglets avec pile propre, par exemple) et qu'on veut viser celui de la racine, pour couvrir toute la fenêtre. Nous n'en aurons pas besoin avant la section 50.30.

---

## 50.3.2 — Le piège du `context`

`Navigator.push(context, ...)` a besoin d'un `context` situé **sous** le `MaterialApp` dans l'arbre.

Ce code échoue :

```dart
class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: ElevatedButton(
          // ERREUR : ce `context` est celui de MonApp,
          // il est AU-DESSUS de MaterialApp.
          onPressed: () => Navigator.push(context, /* ... */),
          child: const Text('Aller'),
        ),
      ),
    );
  }
}
```

**Message d'erreur :**

```text
Navigator operation requested with a context that does not include a Navigator.
The context used to push or pop routes from the Navigator must be that of a
widget that is a descendant of a Navigator widget.
```

La correction est toujours la même : **extraire l'écran dans sa propre classe**, comme dans l'exemple de 50.3. Chaque écran étant un widget séparé, son `build` reçoit un `context` déjà situé sous le `Navigator`.

C'est la première raison, très concrète, pour laquelle on crée une classe par écran.

---

## 50.4 — `MaterialPageRoute`

`Navigator.push` ne prend pas un widget. Il prend une **route**.

Une route est un objet qui sait :

- construire le widget de l'écran ;
- comment l'animer à l'entrée et à la sortie ;
- s'il est opaque (cache-t-il ce qu'il y a dessous ?) ;
- quel résultat il renverra quand il sera dépilé.

`MaterialPageRoute` est l'implémentation standard, avec l'animation de la plateforme : glissement latéral sur Android, glissement iOS avec geste de bord sur iOS.

```dart
MaterialPageRoute(
  builder: (context) => const EcranDetail(),
)
```

Le paramètre `builder` est une **fonction**, pas un widget. Pourquoi ?

Parce que l'écran ne doit être construit qu'au moment où la route est réellement affichée, et surtout parce que le `context` fourni au `builder` est **celui de la nouvelle route**, pas celui de l'appelant. C'est ce `context` qui permettra à l'écran de faire son propre `pop`.

---

## 50.4.1 — Les paramètres utiles de `MaterialPageRoute`

| Paramètre | Type | Rôle |
| --- | --- | --- |
| `builder` | `WidgetBuilder` | Construit le contenu de l'écran. Obligatoire. |
| `settings` | `RouteSettings?` | Nom et arguments de la route (voir 50.13). |
| `fullscreenDialog` | `bool` | `true` : animation de bas en haut, et la flèche devient une croix. |
| `maintainState` | `bool` | `false` : l'écran caché est détruit pour économiser la mémoire. |

L'option `fullscreenDialog` est la plus visible. Elle sert pour les écrans de type « formulaire modal » : ajouter un joueur, écrire un message.

```dart
// Écran classique : entre par la droite, flèche de retour.
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const EcranSecond()),
);

// Écran modal : monte depuis le bas, croix de fermeture.
Navigator.push(
  context,
  MaterialPageRoute(
    fullscreenDialog: true,
    builder: (context) => const EcranSecond(),
  ),
);
```

**Résultat :**

```text
Bouton 1 : l'écran entre par la droite, la flèche de retour est une flèche.
Bouton 2 : l'écran monte depuis le bas, la flèche de retour est une croix (X).
```

---

## 50.4.2 — Les autres routes fournies

| Classe | Effet |
| --- | --- |
| `MaterialPageRoute` | Transition Material de la plateforme. Le choix par défaut. |
| `CupertinoPageRoute` | Transition iOS explicite, même sur Android. Nécessite `package:flutter/cupertino.dart`. |
| `PageRouteBuilder` | Transition entièrement personnalisée (voir 50.24). |
| `DialogRoute` | Utilisée en interne par `showDialog`. |
| `ModalBottomSheetRoute` | Utilisée en interne par `showModalBottomSheet`. |

Retenez que `showDialog` et `showModalBottomSheet`, vus au chapitre 49, **empilent eux aussi une route**. C'est pour cela qu'on les ferme avec `Navigator.pop(context)`.

---

## 50.5 — `Navigator.pop()`

`pop` retire la route du sommet.

```dart
static void pop<T extends Object?>(
  BuildContext context, [
  T? result,
])
```

Le second paramètre est optionnel : c'est la valeur renvoyée à l'écran précédent (section 50.9).

Exemple avec un bouton de fermeture explicite :

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: Center(
        child: FilledButton(
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const EcranDetail()),
            );
          },
          child: const Text('Voir Aria'),
        ),
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Détail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Aria, niveau 42'),
            const SizedBox(height: 24),
            OutlinedButton.icon(
              onPressed: () {
                Navigator.pop(context);
              },
              icon: const Icon(Icons.arrow_back),
              label: const Text('Retour à la guilde'),
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
On ouvre le détail, on appuie sur « Retour à la guilde ».
On revient à la liste, avec l'animation inverse,
exactement comme si on avait appuyé sur la flèche de l'AppBar.
```

---

## 50.5.1 — Ne dépilez jamais la dernière route

```dart
// Sur l'écran d'accueil, alors que la pile ne contient que lui :
Navigator.pop(context);
```

Selon la plateforme, vous obtiendrez un écran noir, ou une exception. Sur Android le comportement est particulièrement désagréable : l'application reste ouverte mais n'affiche plus rien.

La protection s'écrit en une ligne, avec `canPop` (section 50.20) :

```dart
if (Navigator.canPop(context)) {
  Navigator.pop(context);
}
```

---

## 50.5.2 — Où trouver le bon `context` pour `pop`

Le `context` passé à `pop` doit appartenir à l'écran **que l'on veut fermer**.

Dans l'exemple ci-dessus, le bouton est dans le `build` de `EcranDetail` : son `context` est bien sous la route du détail. Tout va bien.

Un piège classique : un `showDialog` ouvert depuis le détail.

```text
Pile réelle :
  [ Dialogue   ]   <- route empilée par showDialog
  [ Détail     ]
  [ Liste      ]
```

Si dans le `builder` du dialogue vous appelez `Navigator.pop(context)` avec le `context` **du dialogue**, vous fermez le dialogue. C'est ce que vous voulez.

Si vous appelez `pop` avec le `context` **du détail** (capturé plus haut), vous fermez... le détail, et le dialogue reste orphelin.

> **Règle :** utilisez toujours le `context` que le `builder` le plus proche vous donne.

---

## 50.6 — Le bouton retour du système

Sur Android, il existe un bouton (ou un geste) « retour ». Sur iOS, il existe un geste de balayage depuis le bord gauche. Sur le Web, il existe le bouton « précédent » du navigateur.

Bonne nouvelle : **vous n'avez rien à écrire**. Le `Navigator` reçoit l'événement du système et appelle `pop` tout seul.

Quand la pile ne contient plus qu'une route et que l'utilisateur appuie sur retour :

| Plateforme | Comportement |
| --- | --- |
| Android | L'application passe en arrière-plan. |
| iOS | Le geste ne fait rien (pas de bouton retour système). |
| Web | Le navigateur revient à la page précédente de son historique. |
| Desktop | Rien. |

Ce comportement automatique est **exactement** ce qui manquait à la version « booléen » de 50.1. C'est la raison numéro un d'utiliser le `Navigator`.

Nous verrons en 50.21 comment intercepter ce bouton pour poser une question à l'utilisateur.

---

## 50.6.1 — La flèche de retour de l'`AppBar`

L'`AppBar` d'un `Scaffold` affiche automatiquement une flèche de retour **si et seulement si** la route peut être dépilée.

Ce comportement est piloté par `automaticallyImplyLeading`, qui vaut `true` par défaut.

```dart
AppBar(
  title: const Text('Détail'),
  automaticallyImplyLeading: false, // supprime la flèche
)
```

Pour remplacer la flèche par autre chose :

```dart
AppBar(
  title: const Text('Détail'),
  leading: IconButton(
    icon: const Icon(Icons.close),
    tooltip: 'Fermer',
    onPressed: () => Navigator.pop(context),
  ),
)
```

Attention : supprimer la flèche ne supprime pas le bouton retour du système. Pour cela, il faut `PopScope` (section 50.21).

---

## 50.7 — Passer des données à l'écran suivant

Voici la question qui arrive immédiatement : « mon écran de détail doit afficher **quel** joueur ? »

La réponse est d'une simplicité désarmante : **par le constructeur**. Un écran est un widget, un widget se configure par son constructeur, comme au chapitre 45.

Commençons par le modèle. Nous réutilisons les classes du chapitre 09.

```dart
class Joueur {
  const Joueur({
    required this.nom,
    required this.classe,
    required this.niveau,
    required this.pointsDeVie,
  });

  final String nom;
  final String classe;
  final int niveau;
  final int pointsDeVie;
}
```

Puis l'écran de détail déclare ce dont il a besoin :

```dart
class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key, required this.joueur});

  final Joueur joueur;
  // ...
}
```

Et l'appelant le fournit :

```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => EcranDetail(joueur: unJoueur)),
);
```

Voici le programme complet.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Joueur {
  const Joueur({
    required this.nom,
    required this.classe,
    required this.niveau,
    required this.pointsDeVie,
  });

  final String nom;
  final String classe;
  final int niveau;
  final int pointsDeVie;
}

const List<Joueur> guilde = [
  Joueur(nom: 'Aria', classe: 'Archère', niveau: 42, pointsDeVie: 310),
  Joueur(nom: 'Borin', classe: 'Guerrier', niveau: 37, pointsDeVie: 480),
  Joueur(nom: 'Célia', classe: 'Mage', niveau: 45, pointsDeVie: 260),
  Joueur(nom: 'Dorn', classe: 'Voleur', niveau: 29, pointsDeVie: 295),
];

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: ListView.separated(
        itemCount: guilde.length,
        separatorBuilder: (context, index) => const Divider(height: 1),
        itemBuilder: (context, index) {
          final joueur = guilde[index];
          return ListTile(
            leading: CircleAvatar(child: Text(joueur.nom[0])),
            title: Text(joueur.nom),
            subtitle: Text('${joueur.classe} — niveau ${joueur.niveau}'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => EcranDetail(joueur: joueur),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key, required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(joueur.nom)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            CircleAvatar(
              radius: 40,
              child: Text(
                joueur.nom[0],
                style: const TextStyle(fontSize: 32),
              ),
            ),
            const SizedBox(height: 24),
            Text(joueur.nom, style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 8),
            Text('Classe : ${joueur.classe}'),
            Text('Niveau : ${joueur.niveau}'),
            Text('Points de vie : ${joueur.pointsDeVie}'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Une liste de quatre joueurs.
On tape sur « Célia » : l'écran de détail affiche
  Célia
  Classe : Mage
  Niveau : 45
  Points de vie : 260
La flèche de retour ramène à la liste.
```

---

## 50.7.1 — Pourquoi le constructeur et pas autre chose

Trois raisons, dans l'ordre d'importance :

1. **Le compilateur vérifie.** Si vous oubliez `joueur:`, le code ne compile pas. Aucun risque de faute de frappe silencieuse.
2. **Le type est connu.** `joueur` est un `Joueur`, pas un `Object?` qu'il faudrait caster.
3. **C'est lisible.** En lisant `EcranDetail(joueur: joueur)`, on sait tout.

Nous verrons en 50.13 une autre méthode, avec `arguments`, qui perd ces trois garanties. Nous verrons alors quand elle reste néanmoins utile.

---

## 50.7.2 — Faut-il passer l'objet ou l'identifiant ?

Deux styles existent.

```dart
// Style A : on passe l'objet complet.
EcranDetail(joueur: joueur)

// Style B : on passe seulement l'identifiant, l'écran recharge les données.
EcranDetail(idJoueur: joueur.id)
```

| Style | Avantage | Inconvénient |
| --- | --- | --- |
| A — l'objet | Affichage immédiat, aucun chargement | Données figées : si elles changent ailleurs, le détail ne le voit pas |
| B — l'identifiant | Toujours à jour, compatible avec les liens profonds et les URL | Nécessite un chargement, donc un état de chargement à gérer |

Pour l'instant, utilisez le style A : c'est plus simple et suffisant. Nous reviendrons au style B en 50.34 avec `go_router`, où il devient obligatoire (une URL ne peut transporter qu'un identifiant, pas un objet Dart).

---

## 50.8 — Récupérer un résultat au retour

Regardez à nouveau la signature de `push` :

```dart
static Future<T?> push<T extends Object?>(BuildContext context, Route<T> route)
```

Elle retourne un `Future`. Vous connaissez les `Future` depuis le chapitre 15.

Ce `Future` se **complète au moment où la route est dépilée**, et sa valeur est celle passée à `pop`.

Autrement dit :

```text
   push(...)          -----> le Future est créé, non complété
      |
      |  l'utilisateur travaille sur le second écran
      |
   pop(context, 42)   -----> le Future se complète avec la valeur 42
      |
      v
   await rend la main avec 42
```

Un exemple minimal. L'écran de détail permet de « recruter » un joueur, et la liste veut savoir si le recrutement a eu lieu.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranAccueil(),
    );
  }
}

class EcranAccueil extends StatefulWidget {
  const EcranAccueil({super.key});

  @override
  State<EcranAccueil> createState() => _EcranAccueilState();
}

class _EcranAccueilState extends State<EcranAccueil> {
  String _message = 'Aucune décision prise.';

  Future<void> _ouvrirRecrutement() async {
    final bool? recrute = await Navigator.push<bool>(
      context,
      MaterialPageRoute(builder: (context) => const EcranRecrutement()),
    );

    if (!mounted) return;

    setState(() {
      if (recrute == null) {
        _message = 'Écran fermé sans décision.';
      } else if (recrute) {
        _message = 'Aria a été recrutée.';
      } else {
        _message = 'Aria a été refusée.';
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(_message, style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _ouvrirRecrutement,
              child: const Text('Examiner la candidature d\'Aria'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranRecrutement extends StatelessWidget {
  const EcranRecrutement({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Candidature')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Aria, archère de niveau 42'),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                OutlinedButton(
                  onPressed: () => Navigator.pop(context, false),
                  child: const Text('Refuser'),
                ),
                const SizedBox(width: 16),
                FilledButton(
                  onPressed: () => Navigator.pop(context, true),
                  child: const Text('Recruter'),
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
Départ : « Aucune décision prise. »
On ouvre, on tape « Recruter »  -> « Aria a été recrutée. »
On ouvre, on tape « Refuser »   -> « Aria a été refusée. »
On ouvre, on tape la flèche de retour -> « Écran fermé sans décision. »
```

Ce dernier cas est capital : quand l'utilisateur revient sans choisir, la valeur reçue est **`null`**. C'est pour cela que le type est `bool?` et non `bool`.

---

## 50.8.1 — Le contrôle `if (!mounted) return;`

Regardez cette ligne :

```dart
final bool? recrute = await Navigator.push<bool>(/* ... */);

if (!mounted) return;

setState(() { /* ... */ });
```

Que se passe-t-il entre le `await` et le `setState` ? **Du temps.** Potentiellement plusieurs minutes, si l'utilisateur laisse l'écran ouvert.

Pendant ce temps, le widget qui a lancé la navigation peut avoir été **démonté** : l'utilisateur a pu quitter cette partie de l'application autrement, ou l'écran a pu être détruit.

Appeler `setState` sur un `State` démonté déclenche :

```text
This widget has been unmounted, so the State no longer has a context
(and should be considered defunct).
Consider canceling any active work during "dispose" or using the "mounted"
getter to determine if the State is still active.
```

`mounted` est un booléen fourni par `State` : il vaut `true` tant que le widget est dans l'arbre.

> **Règle absolue :** après chaque `await`, avant tout `setState`, `Navigator`, `ScaffoldMessenger` ou `Theme.of`, vérifiez `mounted`.

L'analyseur Dart a une règle dédiée, `use_build_context_synchronously`, qui vous avertit quand vous utilisez un `BuildContext` après un `await` sans vérification. Ne l'ignorez pas.

---

## 50.8.2 — La variante avec `.then()`

Si vous préférez le style « callback » du chapitre 15 :

```dart
Navigator.push<bool>(
  context,
  MaterialPageRoute(builder: (context) => const EcranRecrutement()),
).then((recrute) {
  if (!mounted) return;
  setState(() { /* ... */ });
});
```

C'est correct, mais `async` / `await` reste plus lisible dès qu'il y a deux étapes. Utilisez `.then()` seulement quand la fonction appelante ne peut pas être `async`.

---

## 50.9 — `Navigator.pop(context, valeur)`

Le second argument de `pop` est la valeur renvoyée.

```dart
Navigator.pop(context, true);          // renvoie un bool
Navigator.pop(context, 42);            // renvoie un int
Navigator.pop(context, 'Aria');        // renvoie une String
Navigator.pop(context, unJoueur);      // renvoie un objet
Navigator.pop(context);                // renvoie null
```

Il n'y a aucune restriction sur le type : tout objet Dart convient. Vous pouvez renvoyer une liste, une `Map`, une instance de vos classes du chapitre 09.

Exemple avec un objet complet : un formulaire d'ajout de joueur qui renvoie le joueur créé.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Joueur {
  const Joueur({required this.nom, required this.classe, required this.niveau});

  final String nom;
  final String classe;
  final int niveau;
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatefulWidget {
  const EcranListe({super.key});

  @override
  State<EcranListe> createState() => _EcranListeState();
}

class _EcranListeState extends State<EcranListe> {
  final List<Joueur> _joueurs = [
    const Joueur(nom: 'Aria', classe: 'Archère', niveau: 42),
    const Joueur(nom: 'Borin', classe: 'Guerrier', niveau: 37),
  ];

  Future<void> _ajouter() async {
    final Joueur? nouveau = await Navigator.push<Joueur>(
      context,
      MaterialPageRoute(
        fullscreenDialog: true,
        builder: (context) => const EcranFormulaire(),
      ),
    );

    if (!mounted) return;
    if (nouveau == null) return;

    setState(() => _joueurs.add(nouveau));

    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('${nouveau.nom} a rejoint la guilde.')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Guilde (${_joueurs.length})')),
      body: ListView.builder(
        itemCount: _joueurs.length,
        itemBuilder: (context, index) {
          final joueur = _joueurs[index];
          return ListTile(
            leading: CircleAvatar(child: Text(joueur.nom[0])),
            title: Text(joueur.nom),
            subtitle: Text('${joueur.classe} — niveau ${joueur.niveau}'),
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

class EcranFormulaire extends StatefulWidget {
  const EcranFormulaire({super.key});

  @override
  State<EcranFormulaire> createState() => _EcranFormulaireState();
}

class _EcranFormulaireState extends State<EcranFormulaire> {
  final _cleFormulaire = GlobalKey<FormState>();
  final _nomControleur = TextEditingController();
  final _niveauControleur = TextEditingController();
  String _classe = 'Guerrier';

  @override
  void dispose() {
    _nomControleur.dispose();
    _niveauControleur.dispose();
    super.dispose();
  }

  void _valider() {
    if (!_cleFormulaire.currentState!.validate()) return;

    final joueur = Joueur(
      nom: _nomControleur.text.trim(),
      classe: _classe,
      niveau: int.parse(_niveauControleur.text),
    );

    Navigator.pop(context, joueur);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Nouveau joueur')),
      body: Form(
        key: _cleFormulaire,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            TextFormField(
              controller: _nomControleur,
              decoration: const InputDecoration(
                labelText: 'Nom',
                border: OutlineInputBorder(),
              ),
              validator: (valeur) {
                if (valeur == null || valeur.trim().isEmpty) {
                  return 'Le nom est obligatoire.';
                }
                return null;
              },
            ),
            const SizedBox(height: 16),
            DropdownButtonFormField<String>(
              initialValue: _classe,
              decoration: const InputDecoration(
                labelText: 'Classe',
                border: OutlineInputBorder(),
              ),
              items: const [
                DropdownMenuItem(value: 'Guerrier', child: Text('Guerrier')),
                DropdownMenuItem(value: 'Archère', child: Text('Archère')),
                DropdownMenuItem(value: 'Mage', child: Text('Mage')),
                DropdownMenuItem(value: 'Voleur', child: Text('Voleur')),
              ],
              onChanged: (valeur) {
                if (valeur != null) setState(() => _classe = valeur);
              },
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _niveauControleur,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Niveau',
                border: OutlineInputBorder(),
              ),
              validator: (valeur) {
                final n = int.tryParse(valeur ?? '');
                if (n == null) return 'Entrez un nombre entier.';
                if (n < 1 || n > 99) return 'Le niveau va de 1 à 99.';
                return null;
              },
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _valider,
              child: const Text('Recruter'),
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
La liste affiche « Guilde (2) ».
On appuie sur +, on remplit « Célia / Mage / 45 », on valide.
Retour à la liste : « Guilde (3) », Célia est en bas,
et une SnackBar annonce « Célia a rejoint la guilde. ».
Si on ferme le formulaire par la croix, rien ne change.
```

Remarquez le `if (nouveau == null) return;` : il traite le cas « l'utilisateur a annulé ».

---

## 50.10 — Le typage du résultat

Reprenons le chapitre 15. Un `Future<T>` est une promesse de valeur de type `T`.

Ici, `push` est **générique** :

```dart
Future<T?> push<T extends Object?>(BuildContext context, Route<T> route)
```

Le `T` apparaît à deux endroits : dans le type de retour et dans le type de la route. Il y a donc **deux façons** de le préciser.

```dart
// Façon 1 : sur push
final bool? r = await Navigator.push<bool>(
  context,
  MaterialPageRoute(builder: (context) => const EcranChoix()),
);

// Façon 2 : sur MaterialPageRoute
final bool? r = await Navigator.push(
  context,
  MaterialPageRoute<bool>(builder: (context) => const EcranChoix()),
);
```

Les deux sont équivalentes. Choisissez-en une et tenez-vous-y.

---

## 50.10.1 — Ce qui se passe si vous ne typez rien

```dart
final resultat = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const EcranChoix()),
);
```

Dart infère `T = dynamic`, donc `resultat` est de type `dynamic`.

`dynamic` désactive toute vérification à la compilation. Ce code compile :

```dart
final resultat = await Navigator.push(/* ... */);
final int score = resultat;      // compile !
print(score + 10);               // plante à l'exécution si c'était un bool
```

**Erreur à l'exécution :**

```text
type 'bool' is not a subtype of type 'int'
```

> **Règle :** typez toujours le résultat d'un `push`. C'est une seule paire de chevrons et cela vous économise une heure de débogage.

---

## 50.10.2 — Le résultat est toujours nullable

Même si vous écrivez `Navigator.push<bool>`, le type reçu est `bool?`.

C'est logique : Flutter ne peut pas garantir qu'un `pop` avec valeur aura lieu. L'utilisateur peut toujours :

- appuyer sur la flèche de l'`AppBar` ;
- appuyer sur le bouton retour du système ;
- balayer depuis le bord sur iOS.

Dans ces trois cas, `pop` est appelé **sans** valeur, donc le `Future` se complète avec `null`.

Deux façons de gérer ce `null`, vues au chapitre 12 :

```dart
// A : valeur par défaut
final bool recrute = await Navigator.push<bool>(/* ... */) ?? false;

// B : sortie anticipée
final bool? recrute = await Navigator.push<bool>(/* ... */);
if (recrute == null) return;
```

La forme B est préférable quand « annulé » et « refusé » ne veulent pas dire la même chose — ce qui est le cas la plupart du temps.

---

## 50.10.3 — Un type de retour riche

Rien n'oblige à renvoyer un type primitif. Pour un écran de filtres, on renvoie un objet :

```dart
class Filtre {
  const Filtre({required this.niveauMin, required this.classes});

  final int niveauMin;
  final Set<String> classes;
}
```

```dart
final Filtre? filtre = await Navigator.push<Filtre>(
  context,
  MaterialPageRoute(builder: (context) => EcranFiltres(initial: _filtre)),
);
if (!mounted || filtre == null) return;
setState(() => _filtre = filtre);
```

C'est le schéma standard d'un écran de réglages : on passe l'état actuel par le constructeur, on récupère le nouvel état par le résultat.

---

## 50.11 — Les routes nommées : `routes` de `MaterialApp`

Jusqu'ici, chaque navigation citait la classe de l'écran cible :

```dart
Navigator.push(context, MaterialPageRoute(builder: (_) => const EcranReglages()));
```

Cela crée un **couplage** : l'écran de départ doit importer l'écran d'arrivée. Dans une application de trente écrans, cela finit par ressembler à un plat de spaghettis.

Les **routes nommées** proposent une autre approche : on donne un nom textuel à chaque écran, une seule fois, dans `MaterialApp`.

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const EcranAccueil(),
    '/reglages': (context) => const EcranReglages(),
    '/credits': (context) => const EcranCredits(),
  },
)
```

Le type de `routes` est `Map<String, WidgetBuilder>`. Les clés sont des chaînes, par convention préfixées par `/` comme des chemins d'URL.

Ensuite, on navigue par le nom :

```dart
Navigator.pushNamed(context, '/reglages');
```

Programme complet :

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
      title: 'Guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      initialRoute: '/',
      routes: {
        '/': (context) => const EcranAccueil(),
        '/reglages': (context) => const EcranReglages(),
        '/credits': (context) => const EcranCredits(),
      },
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu principal')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FilledButton(
              onPressed: () => Navigator.pushNamed(context, '/reglages'),
              child: const Text('Réglages'),
            ),
            const SizedBox(height: 12),
            FilledButton.tonal(
              onPressed: () => Navigator.pushNamed(context, '/credits'),
              child: const Text('Crédits'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranReglages extends StatelessWidget {
  const EcranReglages({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Réglages')),
      body: const Center(child: Text('Volume, difficulté, langue...')),
    );
  }
}

class EcranCredits extends StatelessWidget {
  const EcranCredits({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Crédits')),
      body: const Center(child: Text('Développé pendant la PARTIE 1B.')),
    );
  }
}
```

**Résultat :**

```text
Le menu affiche deux boutons.
Chacun ouvre son écran, avec flèche de retour et animation.
Aucun `MaterialPageRoute` n'apparaît dans le code des écrans.
```

---

## 50.11.1 — `home` et `'/'` sont incompatibles

Une erreur très fréquente :

```dart
MaterialApp(
  home: const EcranAccueil(),      // définit implicitement '/'
  routes: {
    '/': (context) => const EcranAccueil(),   // redéfinit '/'
  },
)
```

**Erreur :**

```text
'package:flutter/src/material/app.dart': Failed assertion:
'routes == null || !routes.containsKey(Navigator.defaultRouteName) || home == null'
If the home property is specified, the routes table cannot include an entry
for "/", since it would be redundant.
```

Choisissez : soit `home`, soit une entrée `'/'` dans `routes`. Jamais les deux.

---

## 50.11.2 — Avantages et limites des routes nommées

| Avantage | Limite |
| --- | --- |
| Tous les écrans listés au même endroit | Les données passent par `Object?`, donc sans typage |
| Aucun import croisé entre écrans | Aucune vérification à la compilation d'un nom mal écrit |
| Les noms ressemblent à des URL | Pas de paramètres de chemin (`/joueur/42`) sans travail supplémentaire |
| Facile à consigner (journalisation) | La documentation officielle les déconseille pour les applications complexes |

La documentation Flutter est explicite : **« We don't recommend using named routes for most applications »**. Les raisons sont détaillées en 50.30. Vous devez néanmoins les connaître, car elles apparaissent dans quantité de projets existants.

---

## 50.12 — `Navigator.pushNamed()`

```dart
static Future<T?> pushNamed<T extends Object?>(
  BuildContext context,
  String routeName, {
  Object? arguments,
})
```

Trois remarques :

1. Le retour est un `Future<T?>` : le mécanisme de résultat de 50.8 fonctionne à l'identique.
2. `routeName` est une simple `String` : **aucune vérification à la compilation**.
3. `arguments` est un `Object?` : on peut y mettre n'importe quoi (section 50.13).

### Le nom mal écrit

```dart
Navigator.pushNamed(context, '/reglage');   // il manque le « s »
```

**Erreur à l'exécution :**

```text
Could not find a generator for route RouteSettings("/reglage", null)
in the _WidgetsAppState.
Make sure your root app widget has provided a way to generate this route.
```

Le code compile, l'application démarre, et l'erreur n'apparaît qu'au moment du clic. C'est la faiblesse principale des routes nommées.

**Parade obligatoire :** centralisez les noms dans des constantes.

```dart
class Routes {
  const Routes._();

  static const String accueil = '/';
  static const String reglages = '/reglages';
  static const String credits = '/credits';
}
```

```dart
routes: {
  Routes.accueil: (context) => const EcranAccueil(),
  Routes.reglages: (context) => const EcranReglages(),
  Routes.credits: (context) => const EcranCredits(),
},
```

```dart
Navigator.pushNamed(context, Routes.reglages);
```

Maintenant, une faute de frappe est une **erreur de compilation**. Vous avez récupéré la sécurité perdue. Cette astuce coûte dix lignes et doit être systématique.

---

## 50.12.1 — Récupérer un résultat d'une route nommée

Le mécanisme de 50.8 fonctionne à l'identique :

```dart
Future<void> _choisir() async {
  final String? choix =
      await Navigator.pushNamed<String>(context, Routes.difficulte);
  if (!mounted || choix == null) return;
  setState(() => _difficulte = choix);
}
```

Et sur l'écran de difficulté, chaque `ListTile` fait :

```dart
onTap: () => Navigator.pop(context, niveau),
```

---

## 50.13 — `arguments` et `ModalRoute.of(context)`

Une route nommée est un `WidgetBuilder` sans paramètre. Comment lui passer un joueur ?

Par le paramètre `arguments` de `pushNamed`, qui est de type `Object?`.

**Côté émetteur :**

```dart
Navigator.pushNamed(
  context,
  Routes.detail,
  arguments: joueur,
);
```

**Côté récepteur :**

```dart
final joueur = ModalRoute.of(context)!.settings.arguments as Joueur;
```

Décortiquons cette ligne, car elle contient quatre concepts :

```text
ModalRoute.of(context)          -> retrouve la route qui contient ce widget
                                   (type : ModalRoute? — nullable)
        !                       -> on affirme qu'elle existe
         .settings              -> l'objet RouteSettings (name + arguments)
                  .arguments    -> l'Object? transmis
                             as Joueur  -> cast explicite obligatoire
```

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Routes {
  const Routes._();
  static const String liste = '/';
  static const String detail = '/detail';
}

class Joueur {
  const Joueur({required this.nom, required this.classe, required this.niveau});

  final String nom;
  final String classe;
  final int niveau;
}

const List<Joueur> guilde = [
  Joueur(nom: 'Aria', classe: 'Archère', niveau: 42),
  Joueur(nom: 'Borin', classe: 'Guerrier', niveau: 37),
  Joueur(nom: 'Célia', classe: 'Mage', niveau: 45),
];

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      initialRoute: Routes.liste,
      routes: {
        Routes.liste: (context) => const EcranListe(),
        Routes.detail: (context) => const EcranDetail(),
      },
    );
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: ListView.builder(
        itemCount: guilde.length,
        itemBuilder: (context, index) {
          final joueur = guilde[index];
          return ListTile(
            leading: CircleAvatar(child: Text(joueur.nom[0])),
            title: Text(joueur.nom),
            subtitle: Text(joueur.classe),
            onTap: () => Navigator.pushNamed(
              context,
              Routes.detail,
              arguments: joueur,
            ),
          );
        },
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key});

  @override
  Widget build(BuildContext context) {
    final argument = ModalRoute.of(context)?.settings.arguments;

    if (argument is! Joueur) {
      return Scaffold(
        appBar: AppBar(title: const Text('Erreur')),
        body: const Center(
          child: Text('Aucun joueur fourni à cet écran.'),
        ),
      );
    }

    final joueur = argument;

    return Scaffold(
      appBar: AppBar(title: Text(joueur.nom)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(joueur.nom, style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 8),
            Text('Classe : ${joueur.classe}'),
            Text('Niveau : ${joueur.niveau}'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
On tape sur « Célia » : l'écran de détail affiche Célia, Mage, 45.
Si on ouvrait /detail sans argument, l'écran afficherait
« Aucun joueur fourni à cet écran. » au lieu de planter.
```

---

## 50.13.1 — Pourquoi `is!` plutôt que `as`

Comparez :

```dart
// Version fragile
final joueur = ModalRoute.of(context)!.settings.arguments as Joueur;

// Version robuste
final argument = ModalRoute.of(context)?.settings.arguments;
if (argument is! Joueur) {
  return const EcranErreur();
}
final joueur = argument;   // Dart sait maintenant que c'est un Joueur
```

La version fragile plante avec :

```text
type 'Null' is not a subtype of type 'Joueur' in type cast
```

dès que quelqu'un ouvre `/detail` sans argument — ce qui arrive dès qu'un lien profond ou une URL Web pointe vers cet écran.

La version robuste utilise la **promotion de type** de Dart, vue au chapitre 12 : après le `if (argument is! Joueur) return ...`, le compilateur sait que `argument` est un `Joueur`.

---

## 50.13.2 — Passer plusieurs valeurs

`arguments` n'accepte qu'un seul objet. Pour en passer plusieurs, trois solutions.

**Solution A — une `Map` (déconseillée) :**

```dart
arguments: {'nom': 'Aria', 'niveau': 42}
```

```dart
final map = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
final nom = map['nom'] as String;
final niveau = map['niveau'] as int;
```

Aucun typage, des clés en chaînes de caractères : tous les défauts cumulés.

**Solution B — une classe d'arguments (recommandée) :**

```dart
class DetailArguments {
  const DetailArguments({required this.joueur, required this.modeEdition});

  final Joueur joueur;
  final bool modeEdition;
}
```

```dart
arguments: DetailArguments(joueur: joueur, modeEdition: false)
```

```dart
final args = ModalRoute.of(context)!.settings.arguments as DetailArguments;
```

On retrouve le typage, au prix d'une classe. C'est le bon compromis.

**Solution C — abandonner les routes nommées** et revenir au constructeur (50.7), ou passer à `go_router` (50.31).

---

## 50.13.3 — `ModalRoute.of` renvoie autre chose d'utile

`ModalRoute.of(context)` donne accès à la route courante. On peut y lire :

| Propriété | Type | Utilité |
| --- | --- | --- |
| `settings.name` | `String?` | Le nom de la route, pour la journalisation |
| `settings.arguments` | `Object?` | Les arguments |
| `isCurrent` | `bool` | La route est-elle au sommet de la pile ? |
| `isFirst` | `bool` | La route est-elle au fond de la pile ? |
| `animation` | `Animation<double>?` | La progression de la transition d'entrée |

Exemple pratique : masquer un bouton « retour » quand on est déjà au fond de la pile.

```dart
final estPremier = ModalRoute.of(context)?.isFirst ?? true;
// ...
if (!estPremier)
  TextButton(
    onPressed: () => Navigator.pop(context),
    child: const Text('Retour'),
  ),
```

---

## 50.14 — `onGenerateRoute`

La table `routes` est statique : une clé, un écran. Elle ne sait pas traiter `/joueur/42`, parce qu'il faudrait une entrée par joueur.

`onGenerateRoute` résout cela. C'est une **fonction** appelée par le `Navigator` chaque fois qu'un nom n'est pas trouvé dans `routes` (ou systématiquement si `routes` est absent).

Son type est `RouteFactory` :

```dart
typedef RouteFactory = Route<dynamic>? Function(RouteSettings settings);
```

Elle reçoit les `RouteSettings` (nom + arguments) et doit renvoyer une `Route`, ou `null` si elle ne sait pas gérer ce nom.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Joueur {
  const Joueur({required this.id, required this.nom, required this.niveau});

  final int id;
  final String nom;
  final int niveau;
}

const List<Joueur> guilde = [
  Joueur(id: 1, nom: 'Aria', niveau: 42),
  Joueur(id: 2, nom: 'Borin', niveau: 37),
  Joueur(id: 3, nom: 'Célia', niveau: 45),
];

Joueur? chercherJoueur(int? id) {
  for (final joueur in guilde) {
    if (joueur.id == id) return joueur;
  }
  return null;
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      initialRoute: '/',
      onGenerateRoute: _genererRoute,
    );
  }

  static Route<dynamic>? _genererRoute(RouteSettings settings) {
    final uri = Uri.parse(settings.name ?? '/');

    // '/'  -> segments == []
    if (uri.pathSegments.isEmpty) {
      return MaterialPageRoute(
        settings: settings,
        builder: (context) => const EcranListe(),
      );
    }

    // '/joueur/2'  -> segments == ['joueur', '2']
    if (uri.pathSegments.length == 2 && uri.pathSegments.first == 'joueur') {
      final id = int.tryParse(uri.pathSegments[1]);
      final joueur = chercherJoueur(id);

      if (joueur != null) {
        return MaterialPageRoute(
          settings: settings,
          builder: (context) => EcranDetail(joueur: joueur),
        );
      }
    }

    // Nom inconnu : on rend null, onUnknownRoute prendra le relais.
    return null;
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: ListView(
        children: [
          for (final joueur in guilde)
            ListTile(
              leading: CircleAvatar(child: Text('${joueur.id}')),
              title: Text(joueur.nom),
              subtitle: Text('/joueur/${joueur.id}'),
              onTap: () =>
                  Navigator.pushNamed(context, '/joueur/${joueur.id}'),
            ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.warning_amber),
            title: const Text('Ouvrir /joueur/999 (inexistant)'),
            onTap: () => Navigator.pushNamed(context, '/joueur/999'),
          ),
        ],
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key, required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(joueur.nom)),
      body: Center(
        child: Text(
          '${joueur.nom} — niveau ${joueur.niveau}',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
La liste montre trois joueurs et une entrée piégée.
Taper sur « Borin » ouvre /joueur/2 -> « Borin — niveau 37 ».
Taper sur l'entrée piégée provoque une exception,
car aucune route n'a été générée et onUnknownRoute n'est pas défini.
Nous corrigeons cela à la section suivante.
```

> La recherche est écrite avec une petite fonction `chercherJoueur` plutôt qu'avec
> `firstWhere`, qui lève une exception quand rien ne correspond. Ici, l'absence de
> résultat est un cas normal : c'est ce qui déclenche la page 404.

---

## 50.14.1 — Pourquoi transmettre `settings`

Notez `settings: settings` dans chaque `MaterialPageRoute`. Ne l'oubliez jamais.

Sans lui, la route générée n'a plus de nom : `ModalRoute.of(context)?.settings.name` renvoie `null`, la journalisation ne fonctionne plus, `popUntil(ModalRoute.withName(...))` (section 50.19) ne trouve plus rien, et sur le Web l'URL n'est pas correcte.

---

## 50.14.2 — L'ordre de résolution d'un nom

Quand vous appelez `pushNamed(context, '/x')`, Flutter procède dans cet ordre :

```text
1. '/x' est-il une clé de `routes` ?          -> oui : on l'utilise, fin.
2. onGenerateRoute('/x') renvoie-t-il une Route ? -> oui : on l'utilise, fin.
3. onUnknownRoute('/x') renvoie-t-il une Route ?  -> oui : on l'utilise, fin.
4. Sinon : exception.
```

On peut donc combiner `routes` (pour les écrans fixes) et `onGenerateRoute` (pour les écrans paramétrés). C'est même la configuration la plus courante.

---

## 50.15 — `onUnknownRoute` et la page 404

`onUnknownRoute` est le filet de sécurité. Même type que `onGenerateRoute` : `RouteFactory`.

```dart
MaterialApp(
  onGenerateRoute: _genererRoute,
  onUnknownRoute: (settings) => MaterialPageRoute(
    settings: settings,
    builder: (context) => EcranIntrouvable(nom: settings.name),
  ),
)
```

Voici le programme complet, avec la page 404.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      initialRoute: '/',
      routes: {
        '/': (context) => const EcranAccueil(),
        '/boutique': (context) => const EcranBoutique(),
      },
      onUnknownRoute: (settings) => MaterialPageRoute(
        settings: settings,
        builder: (context) => EcranIntrouvable(nom: settings.name),
      ),
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FilledButton(
              onPressed: () => Navigator.pushNamed(context, '/boutique'),
              child: const Text('Ouvrir la boutique'),
            ),
            const SizedBox(height: 12),
            OutlinedButton(
              onPressed: () => Navigator.pushNamed(context, '/donjon-secret'),
              child: const Text('Ouvrir une route inexistante'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranBoutique extends StatelessWidget {
  const EcranBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Boutique')),
      body: const Center(child: Text('Potions, épées, boucliers.')),
    );
  }
}

class EcranIntrouvable extends StatelessWidget {
  const EcranIntrouvable({super.key, this.nom});

  final String? nom;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Page introuvable')),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                Icons.explore_off,
                size: 96,
                color: Theme.of(context).colorScheme.error,
              ),
              const SizedBox(height: 24),
              Text(
                '404',
                style: Theme.of(context).textTheme.displaySmall,
              ),
              const SizedBox(height: 8),
              Text(
                'La route « ${nom ?? 'inconnue'} » n\'existe pas.',
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 24),
              FilledButton.icon(
                onPressed: () => Navigator.pop(context),
                icon: const Icon(Icons.arrow_back),
                label: const Text('Revenir'),
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
Le second bouton ouvre l'écran 404 :
une icône de boussole barrée, le texte « 404 »,
« La route « /donjon-secret » n'existe pas. »
et un bouton de retour.
```

> Sur le Web, `onUnknownRoute` est ce qui répond quand l'utilisateur tape une adresse à la main. C'est donc un écran que vos utilisateurs verront réellement : soignez-le.

---

## 50.16 — `initialRoute`

`initialRoute` désigne le nom de la route affichée au démarrage.

```dart
MaterialApp(
  initialRoute: '/',
  routes: { /* ... */ },
)
```

Par défaut, `initialRoute` vaut `Navigator.defaultRouteName`, c'est-à-dire `'/'`.

Une subtilité importante : si `initialRoute` contient plusieurs segments, Flutter tente de **construire une pile** en cherchant chaque préfixe dans `routes`.

```dart
MaterialApp(
  initialRoute: '/guilde/detail',
  routes: {
    '/': (context) => const EcranAccueil(),
    '/guilde': (context) => const EcranGuilde(),
    '/guilde/detail': (context) => const EcranDetail(),
  },
)
```

**Pile obtenue au démarrage :**

```text
[ EcranDetail  ]   <- visible
[ EcranGuilde  ]
[ EcranAccueil ]
```

L'utilisateur arrive directement sur le détail, mais peut revenir en arrière normalement. C'est exactement ce qu'on attend d'un lien profond.

Attention : ce mécanisme ne fonctionne que si **chaque préfixe** existe dans `routes`. Si `/guilde` manque, la pile générée est incomplète et Flutter journalise un avertissement puis retombe sur `/`.

---

## 50.16.1 — Quand `initialRoute` est ignoré

`initialRoute` est ignoré si :

- `home` est défini (car `home` occupe déjà `'/'`) ;
- la plateforme fournit une route initiale différente (ouverture par lien profond) ;
- vous utilisez `MaterialApp.router` avec `go_router` (l'équivalent est alors `initialLocation`).

---

## 50.17 — `pushReplacement`

Parfois, on ne veut **pas** que l'utilisateur puisse revenir.

Cas d'école : l'écran de connexion. Une fois connecté, revenir à l'écran de connexion n'a aucun sens.

```dart
static Future<T?> pushReplacement<T extends Object?, TO extends Object?>(
  BuildContext context,
  Route<T> newRoute, {
  TO? result,
})
```

Effet sur la pile :

```text
AVANT                          APRÈS pushReplacement(Menu)

[ Connexion ]  <- sommet       [ Menu ]  <- sommet
                               
La route Connexion est retirée ET détruite.
La pile garde la même hauteur.
```

Il existe aussi `pushReplacementNamed(context, '/menu')` pour les routes nommées.

Programme complet : un écran de démarrage (*splash*) suivi du menu.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranDemarrage(),
    );
  }
}

class EcranDemarrage extends StatefulWidget {
  const EcranDemarrage({super.key});

  @override
  State<EcranDemarrage> createState() => _EcranDemarrageState();
}

class _EcranDemarrageState extends State<EcranDemarrage> {
  @override
  void initState() {
    super.initState();
    _charger();
  }

  Future<void> _charger() async {
    // Simule le chargement des sauvegardes (chapitre 15).
    await Future<void>.delayed(const Duration(seconds: 2));

    if (!mounted) return;

    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => const EcranMenu()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.primaryContainer,
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.shield_moon, size: 96),
            const SizedBox(height: 24),
            Text(
              'CHRONIQUES DE LA GUILDE',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 32),
            const CircularProgressIndicator(),
          ],
        ),
      ),
    );
  }
}

class EcranMenu extends StatelessWidget {
  const EcranMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu principal')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Chargement terminé.'),
            const SizedBox(height: 16),
            Text(
              'Le bouton retour quitte l\'application :\n'
              'l\'écran de démarrage a été remplacé, pas empilé.',
              textAlign: TextAlign.center,
              style: Theme.of(context).textTheme.bodySmall,
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
Deux secondes d'écran violet avec un indicateur circulaire.
Puis le menu principal, SANS flèche de retour dans l'AppBar.
Le bouton retour du système ferme l'application : la pile n'a qu'une route.
```

Remarquez à nouveau le `if (!mounted) return;` après le `await`. Sans lui, si l'utilisateur ferme l'application pendant les deux secondes, vous obtenez une exception.

---

## 50.17.1 — `pushReplacement` et le résultat de l'ancienne route

Le paramètre nommé `result` de `pushReplacement` sert à compléter le `Future` de la route **remplacée**.

```dart
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => const EcranMenu()),
  result: 'connexion terminée',
);
```

Si quelqu'un attendait le résultat de l'écran de connexion via `await Navigator.push<String>(...)`, il reçoit `'connexion terminée'`. C'est rare mais utile pour un flux d'inscription en plusieurs pages.

---

## 50.18 — `pushAndRemoveUntil`

`pushReplacement` ne retire qu'une route. Parfois il faut **tout vider**.

Cas d'école : la fin d'une partie. La pile ressemble à :

```text
[ Résultat  ]
[ Combat 3  ]
[ Combat 2  ]
[ Combat 1  ]
[ Menu      ]
```

Depuis l'écran de résultat, l'utilisateur appuie sur « Retour au menu ». On ne veut évidemment pas qu'il traverse les trois combats à reculons.

```dart
static Future<T?> pushAndRemoveUntil<T extends Object?>(
  BuildContext context,
  Route<T> newRoute,
  RoutePredicate predicate,
)
```

Le `predicate` est de type `bool Function(Route<dynamic>)`. Flutter dépile **tant que** le prédicat renvoie `false`.

Deux prédicats couvrent 95 % des besoins :

```dart
// A : tout supprimer
(route) => false

// B : supprimer jusqu'à retrouver la route nommée '/menu'
ModalRoute.withName('/menu')
```

```dart
// Vider complètement la pile et poser le menu au fond :
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const EcranMenu()),
  (route) => false,
);
```

**Effet :**

```text
AVANT                     APRÈS

[ Résultat  ]             [ Menu ]  <- sommet, et aussi fond
[ Combat 3  ]
[ Combat 2  ]
[ Combat 1  ]
[ Menu      ]

La pile ne contient plus qu'une route.
Le bouton retour quitte l'application.
```

Programme complet :

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
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
      body: Center(
        child: FilledButton(
          onPressed: () => Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => const EcranCombat(numero: 1),
            ),
          ),
          child: const Text('Commencer la campagne'),
        ),
      ),
    );
  }
}

class EcranCombat extends StatelessWidget {
  const EcranCombat({super.key, required this.numero});

  final int numero;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Combat $numero')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Vous affrontez l\'ennemi n° $numero.'),
            const SizedBox(height: 24),
            if (numero < 3)
              FilledButton(
                onPressed: () => Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => EcranCombat(numero: numero + 1),
                  ),
                ),
                child: const Text('Combat suivant'),
              )
            else
              FilledButton(
                onPressed: () => Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const EcranResultat()),
                ),
                child: const Text('Affronter le boss'),
              ),
          ],
        ),
      ),
    );
  }
}

class EcranResultat extends StatelessWidget {
  const EcranResultat({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Victoire'),
        automaticallyImplyLeading: false,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.emoji_events, size: 96),
            const SizedBox(height: 16),
            Text(
              'Campagne terminée',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 32),
            FilledButton(
              onPressed: () {
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(builder: (context) => const EcranMenu()),
                  (route) => false,
                );
              },
              child: const Text('Retour au menu'),
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
Menu -> Combat 1 -> Combat 2 -> Combat 3 -> Victoire   (5 routes empilées)
On appuie sur « Retour au menu ».
On arrive au menu, SANS flèche de retour.
Le bouton système quitte l'application : la pile ne contient qu'une route.
```

---

## 50.18.1 — `pushNamedAndRemoveUntil`

L'équivalent pour les routes nommées :

```dart
Navigator.pushNamedAndRemoveUntil(
  context,
  '/menu',
  (route) => false,
);
```

C'est la ligne standard d'un bouton « se déconnecter » : on part sur l'écran de connexion et on efface tout l'historique de la session précédente.

---

## 50.18.2 — Conserver un écran au fond

Si vous voulez garder la racine et effacer le reste :

```dart
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const EcranResultat()),
  ModalRoute.withName('/'),
);
```

**Effet :**

```text
AVANT                     APRÈS

[ Combat 3  ]             [ Résultat ]
[ Combat 2  ]             [ Menu     ]   <- conservé, car nommé '/'
[ Combat 1  ]
[ Menu ('/')]
```

`ModalRoute.withName('/')` fabrique un prédicat qui renvoie `true` quand `route.settings.name == '/'`. Cela suppose donc que vos routes **aient un nom**, d'où l'importance de `settings:` vue en 50.14.1.

---

## 50.19 — `popUntil`

`popUntil` dépile sans rien empiler.

```dart
static void popUntil(BuildContext context, RoutePredicate predicate)
```

Le prédicat suit la même logique : on dépile **tant qu'il renvoie `false`**, on s'arrête dès qu'il renvoie `true`.

```dart
// Remonter à la racine de la pile :
Navigator.popUntil(context, (route) => route.isFirst);

// Remonter à la route nommée '/guilde' :
Navigator.popUntil(context, ModalRoute.withName('/guilde'));
```

**Effet de `(route) => route.isFirst` :**

```text
AVANT                     APRÈS

[ Détail arme ]           [ Liste ]
[ Inventaire  ]
[ Détail Aria ]
[ Liste       ]
```

Programme complet, avec compteur de profondeur :

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      home: const EcranNiveau(profondeur: 0),
    );
  }
}

class EcranNiveau extends StatelessWidget {
  const EcranNiveau({super.key, required this.profondeur});

  final int profondeur;

  @override
  Widget build(BuildContext context) {
    final estRacine = profondeur == 0;

    return Scaffold(
      appBar: AppBar(
        title: Text(estRacine ? 'Surface' : 'Sous-sol $profondeur'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Profondeur : $profondeur',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) =>
                      EcranNiveau(profondeur: profondeur + 1),
                ),
              ),
              icon: const Icon(Icons.arrow_downward),
              label: const Text('Descendre'),
            ),
            const SizedBox(height: 12),
            if (!estRacine)
              OutlinedButton.icon(
                onPressed: () =>
                    Navigator.popUntil(context, (route) => route.isFirst),
                icon: const Icon(Icons.vertical_align_top),
                label: const Text('Remonter à la surface'),
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
On descend cinq fois : « Profondeur : 5 ».
On appuie sur « Remonter à la surface ».
On arrive directement à « Profondeur : 0 », en une animation.
Le bouton « Remonter » disparaît car on est à la racine.
```

---

## 50.19.1 — Le piège du prédicat toujours faux

```dart
// NE FAITES PAS CELA
Navigator.popUntil(context, (route) => false);
```

Le prédicat n'est jamais vrai, donc Flutter dépile tout... y compris la dernière route. L'application se retrouve avec une pile vide et un écran noir.

`popUntil` doit **toujours** avoir une condition d'arrêt atteignable. Les deux sûres sont `(route) => route.isFirst` et `ModalRoute.withName(...)` avec un nom qui existe réellement dans la pile.

---

## 50.20 — `canPop` et `maybePop`

### `canPop`

```dart
static bool canPop(BuildContext context)
```

Renvoie `true` si la pile contient au moins deux routes, autrement dit si un `pop` est possible sans vider la pile.

```dart
if (Navigator.canPop(context)) {
  Navigator.pop(context);
} else {
  // On est à la racine : proposer autre chose.
}
```

### `maybePop`

```dart
static Future<bool> maybePop<T extends Object?>(
  BuildContext context, [
  T? result,
])
```

`maybePop` est plus intelligent que `canPop` + `pop`. Il :

1. consulte les `PopScope` de la route courante (section 50.21) ;
2. si tous autorisent le retour et que la pile le permet, dépile ;
3. renvoie un `Future<bool>` indiquant si le `pop` a effectivement eu lieu.

C'est **exactement** ce que fait le bouton retour du système. Utiliser `maybePop` dans un bouton personnalisé, c'est garantir le même comportement que le bouton système.

```dart
// Sur un écran racine, canPop vaut false ; sur un écran empilé, true.
Text('canPop : ${Navigator.canPop(context)}'),

// Un bouton de retour qui se comporte comme le bouton système :
OutlinedButton(
  onPressed: () async {
    final aFerme = await Navigator.maybePop(context);
    if (!context.mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('maybePop a renvoyé $aFerme')),
    );
  },
  child: const Text('Tenter un retour'),
),
```

**Résultat :**

```text
Sur A : « canPop : false ».
On appuie sur « Tenter un retour » : une SnackBar annonce
« maybePop a renvoyé false », et rien ne bouge.
Sur B : « canPop : true ». « Tenter un retour » ramène à A.
```

> Notez `if (!context.mounted) return;` : sur un `StatelessWidget`, il n'y a pas de `State`, donc pas de `mounted` de `State`. Depuis Flutter 3.7, `BuildContext` expose lui-même un getter `mounted`. Utilisez `context.mounted` dans un `StatelessWidget` et `mounted` dans un `State`.

---

## 50.21 — Empêcher le retour : `PopScope`

Il arrive qu'on veuille **bloquer** le retour : un formulaire à moitié rempli, une partie en cours, un envoi en cours.

Le widget s'appelle `PopScope`.

```dart
PopScope<T>({
  Key? key,
  required Widget child,
  bool canPop = true,
  PopInvokedWithResultCallback<T>? onPopInvokedWithResult,
})
```

avec :

```dart
typedef PopInvokedWithResultCallback<T> = void Function(bool didPop, T? result);
```

> **Attention à l'historique :** avant Flutter 3.12, ce widget s'appelait `WillPopScope` et son rappel s'appelait `onWillPop`. Il est **obsolète**. Ensuite, `PopScope` a eu un rappel `onPopInvoked`, lui aussi **obsolète** depuis Flutter 3.22 au profit de `onPopInvokedWithResult`. Beaucoup de tutoriels en ligne sont périmés : utilisez `PopScope` + `onPopInvokedWithResult`.

### Le blocage total

```dart
PopScope(
  canPop: false,
  child: Scaffold(/* ... */),
)
```

Ici, le bouton retour du système ne fait plus rien du tout. C'est brutal et rarement souhaitable seul : sans message, l'utilisateur croit que l'application est bloquée.

Exemple d'usage légitime : pendant un envoi réseau.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranAccueil(),
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: Center(
        child: FilledButton(
          onPressed: () => Navigator.push(
            context,
            MaterialPageRoute(builder: (context) => const EcranEnvoi()),
          ),
          child: const Text('Sauvegarder la partie'),
        ),
      ),
    );
  }
}

class EcranEnvoi extends StatefulWidget {
  const EcranEnvoi({super.key});

  @override
  State<EcranEnvoi> createState() => _EcranEnvoiState();
}

class _EcranEnvoiState extends State<EcranEnvoi> {
  bool _envoiEnCours = true;

  @override
  void initState() {
    super.initState();
    _envoyer();
  }

  Future<void> _envoyer() async {
    await Future<void>.delayed(const Duration(seconds: 4));
    if (!mounted) return;
    setState(() => _envoiEnCours = false);
  }

  @override
  Widget build(BuildContext context) {
    return PopScope(
      canPop: !_envoiEnCours,
      onPopInvokedWithResult: (didPop, result) {
        if (didPop) return;
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('Sauvegarde en cours, veuillez patienter.'),
          ),
        );
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Sauvegarde'),
          automaticallyImplyLeading: !_envoiEnCours,
        ),
        body: Center(
          child: _envoiEnCours
              ? const Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    CircularProgressIndicator(),
                    SizedBox(height: 16),
                    Text('Envoi vers le serveur...'),
                  ],
                )
              : Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.cloud_done, size: 72),
                    const SizedBox(height: 16),
                    const Text('Partie sauvegardée.'),
                    const SizedBox(height: 24),
                    FilledButton(
                      onPressed: () => Navigator.pop(context),
                      child: const Text('Terminer'),
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
Pendant 4 secondes : un indicateur, aucune flèche de retour.
Appuyer sur le bouton retour affiche
« Sauvegarde en cours, veuillez patienter. » et ne ferme rien.
Après 4 secondes : la flèche revient, le retour fonctionne.
```

---

## 50.21.1 — Lire correctement `onPopInvokedWithResult`

Le rappel est appelé **après** que Flutter a décidé, pas avant. Le premier paramètre dit ce qui s'est passé :

| `didPop` | Signification | Que faire |
| --- | --- | --- |
| `true` | Le `pop` a eu lieu, l'écran est en train de se fermer | Ne rien faire de bloquant. L'écran part. |
| `false` | Le `pop` a été **empêché** par `canPop: false` | C'est ici qu'on affiche un message ou une confirmation. |

D'où le motif standard :

```dart
onPopInvokedWithResult: (didPop, result) {
  if (didPop) return;    // rien à faire, l'écran se ferme
  // ici : canPop valait false, on réagit
}
```

Oublier le `if (didPop) return;` est l'erreur numéro un avec `PopScope` : vous affichez une confirmation alors que l'écran est déjà en train de disparaître.

---

## 50.22 — La boîte de dialogue « voulez-vous vraiment quitter ? »

C'est le cas d'usage réel de `PopScope` : bloquer, demander, puis fermer soi-même si l'utilisateur confirme.

L'algorithme est en trois temps :

```text
1. canPop: false            -> le retour est intercepté
2. onPopInvokedWithResult   -> on affiche la boîte de dialogue
3. si l'utilisateur confirme -> on appelle Navigator.pop() nous-mêmes
```

Le point 3 mérite une explication : puisque `canPop` est `false`, un `pop` ordinaire serait de nouveau bloqué. Il faut donc appeler le `pop` sur le `NavigatorState` **après** que l'utilisateur a confirmé — et à ce moment-là, on utilise `Navigator.of(context).pop()`, qui n'est pas repassé par le `PopScope` puisque nous ne passons pas par le canal du système.

Voici le programme complet, un éditeur de fiche de personnage avec modifications non enregistrées.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatefulWidget {
  const EcranListe({super.key});

  @override
  State<EcranListe> createState() => _EcranListeState();
}

class _EcranListeState extends State<EcranListe> {
  String _nom = 'Aria';

  Future<void> _editer() async {
    final String? nouveau = await Navigator.push<String>(
      context,
      MaterialPageRoute(builder: (context) => EcranEdition(nomInitial: _nom)),
    );
    if (!mounted || nouveau == null) return;
    setState(() => _nom = nouveau);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Fiche')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(_nom, style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 24),
            FilledButton(onPressed: _editer, child: const Text('Modifier')),
          ],
        ),
      ),
    );
  }
}

class EcranEdition extends StatefulWidget {
  const EcranEdition({super.key, required this.nomInitial});

  final String nomInitial;

  @override
  State<EcranEdition> createState() => _EcranEditionState();
}

class _EcranEditionState extends State<EcranEdition> {
  late final TextEditingController _controleur;
  bool _modifie = false;

  @override
  void initState() {
    super.initState();
    _controleur = TextEditingController(text: widget.nomInitial);
    _controleur.addListener(_surModification);
  }

  void _surModification() {
    final modifie = _controleur.text != widget.nomInitial;
    if (modifie != _modifie) {
      setState(() => _modifie = modifie);
    }
  }

  @override
  void dispose() {
    _controleur.removeListener(_surModification);
    _controleur.dispose();
    super.dispose();
  }

  Future<bool> _confirmerAbandon() async {
    final bool? reponse = await showDialog<bool>(
      context: context,
      builder: (dialogContext) => AlertDialog(
        title: const Text('Quitter sans enregistrer ?'),
        content: const Text(
          'Vos modifications seront perdues.',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(dialogContext, false),
            child: const Text('Rester'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(dialogContext, true),
            child: const Text('Quitter'),
          ),
        ],
      ),
    );
    return reponse ?? false;
  }

  @override
  Widget build(BuildContext context) {
    return PopScope<String>(
      canPop: !_modifie,
      onPopInvokedWithResult: (didPop, result) async {
        if (didPop) return;

        final quitter = await _confirmerAbandon();
        if (!context.mounted) return;
        if (quitter) {
          Navigator.pop(context);
        }
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Modifier la fiche'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, _controleur.text),
              child: const Text('Enregistrer'),
            ),
          ],
        ),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              TextField(
                controller: _controleur,
                decoration: const InputDecoration(
                  labelText: 'Nom du personnage',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 16),
              Text(
                _modifie
                    ? 'Modifications non enregistrées.'
                    : 'Aucune modification.',
                style: TextStyle(
                  color: _modifie
                      ? Theme.of(context).colorScheme.error
                      : Theme.of(context).colorScheme.outline,
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
On ouvre l'éditeur : « Aucune modification. »
Le bouton retour fonctionne normalement.

On tape une lettre : « Modifications non enregistrées. » en rouge.
Le bouton retour ouvre la boîte :
  « Quitter sans enregistrer ? »
  [Rester] [Quitter]
« Rester » ferme la boîte, on reste dans l'éditeur.
« Quitter » ferme la boîte PUIS l'éditeur, sans rien renvoyer.

« Enregistrer » renvoie le texte : la fiche est mise à jour.
```

---

## 50.22.1 — Les points délicats de cet exemple

**Le `dialogContext`.** Dans le `builder` de `showDialog`, le paramètre s'appelle `dialogContext` et non `context`. C'est volontaire : `Navigator.pop(dialogContext, false)` ferme **le dialogue**. Si on écrivait `Navigator.pop(context, false)` avec le `context` de l'écran, on fermerait le mauvais élément.

**Le `if (!context.mounted) return;`.** Entre le `await _confirmerAbandon()` et le `Navigator.pop(context)`, l'utilisateur a pu quitter l'application. Le contrôle est obligatoire.

**Le `canPop` dynamique.** `canPop: !_modifie` est recalculé à chaque `build`. Tant que rien n'est modifié, le retour est libre : aucune boîte de dialogue inutile. C'est ce qui distingue une bonne application d'une application pénible.

---

## 50.23 — `Hero` : l'animation partagée entre deux écrans

Quand on passe d'une liste à un détail, l'avatar du joueur est présent des deux côtés. Plutôt que de le faire disparaître puis réapparaître, on peut le faire **voler** de sa position dans la liste à sa position dans le détail.

Ce widget s'appelle `Hero`. Son usage tient en une règle :

> Placez le **même** `Hero`, avec le **même** `tag`, sur les deux écrans. Flutter fait le reste.

```dart
Hero(
  tag: 'avatar-Aria',
  child: CircleAvatar(child: Text('A')),
)
```

Le `tag` est un `Object` : une chaîne, un entier, n'importe quoi qui se compare avec `==`. Il doit être **unique sur un écran** et **identique entre les deux écrans**.

Programme complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Joueur {
  const Joueur({required this.nom, required this.classe, required this.couleur});

  final String nom;
  final String classe;
  final Color couleur;
}

const List<Joueur> guilde = [
  Joueur(nom: 'Aria', classe: 'Archère', couleur: Colors.green),
  Joueur(nom: 'Borin', classe: 'Guerrier', couleur: Colors.brown),
  Joueur(nom: 'Célia', classe: 'Mage', couleur: Colors.purple),
  Joueur(nom: 'Dorn', classe: 'Voleur', couleur: Colors.blueGrey),
];

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranListe(),
    );
  }
}

class EcranListe extends StatelessWidget {
  const EcranListe({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Guilde')),
      body: GridView.count(
        crossAxisCount: 2,
        padding: const EdgeInsets.all(16),
        mainAxisSpacing: 16,
        crossAxisSpacing: 16,
        children: [
          for (final joueur in guilde)
            InkWell(
              onTap: () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => EcranDetail(joueur: joueur),
                ),
              ),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Hero(
                    tag: 'avatar-${joueur.nom}',
                    child: CircleAvatar(
                      radius: 36,
                      backgroundColor: joueur.couleur,
                      child: Text(
                        joueur.nom[0],
                        style: const TextStyle(
                          fontSize: 28,
                          color: Colors.white,
                        ),
                      ),
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(joueur.nom),
                ],
              ),
            ),
        ],
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key, required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(joueur.nom)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: 'avatar-${joueur.nom}',
              child: CircleAvatar(
                radius: 90,
                backgroundColor: joueur.couleur,
                child: Text(
                  joueur.nom[0],
                  style: const TextStyle(fontSize: 72, color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 24),
            Text(joueur.nom, style: Theme.of(context).textTheme.headlineMedium),
            Text(joueur.classe),
          ],
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Une grille de quatre avatars colorés.
On tape sur « Célia » : le petit avatar violet grandit
et se déplace en une animation continue vers le centre du détail.
Le retour rejoue l'animation en sens inverse.
```

---

## 50.23.1 — Les trois erreurs classiques avec `Hero`

**Erreur 1 — deux `Hero` avec le même tag sur le même écran.**

```text
There are multiple heroes that share the same tag within a subtree.
```

C'est pour cela que le tag contient le nom du joueur : `'avatar-${joueur.nom}'`. Un tag constant `'avatar'` planterait dès que la liste contient deux éléments.

**Erreur 2 — un tag qui change entre les deux écrans.** Aucune erreur affichée : l'animation ne se produit tout simplement pas. Vérifiez que les deux chaînes sont rigoureusement identiques.

**Erreur 3 — un `Hero` autour d'un widget trop complexe.** Pendant le vol, le `Hero` est retiré des deux arbres et placé dans une couche superposée. Un `Hero` autour d'une `Card` entière donne souvent une déformation disgracieuse, parce que la largeur change brutalement. Limitez-vous à des formes simples : avatar, image, icône.

---

## 50.24 — Personnaliser la transition (`PageRouteBuilder`)

`MaterialPageRoute` impose la transition de la plateforme. Pour écrire la vôtre, utilisez `PageRouteBuilder`.

```dart
PageRouteBuilder({
  RouteSettings? settings,
  required RoutePageBuilder pageBuilder,
  RouteTransitionsBuilder transitionsBuilder = /* transition par défaut */,
  Duration transitionDuration = const Duration(milliseconds: 300),
  Duration reverseTransitionDuration = const Duration(milliseconds: 300),
  bool opaque = true,
  bool fullscreenDialog = false,
})
```

Deux fonctions à fournir :

```text
pageBuilder(context, animation, secondaryAnimation)
    -> construit le CONTENU de l'écran

transitionsBuilder(context, animation, secondaryAnimation, child)
    -> enveloppe ce contenu dans une animation
```

`animation` va de 0.0 à 1.0 quand l'écran entre, et de 1.0 à 0.0 quand il sort. `secondaryAnimation` fait la même chose pour l'écran **suivant**, ce qui permet d'animer aussi la sortie de l'écran courant.

Programme complet, avec trois transitions au choix.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranAccueil(),
    );
  }
}

/// Fondu simple.
Route<void> routeFondu(Widget page) {
  return PageRouteBuilder<void>(
    transitionDuration: const Duration(milliseconds: 400),
    pageBuilder: (context, animation, secondaryAnimation) => page,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  );
}

/// Glissement depuis le bas, avec une courbe d'accélération.
Route<void> routeDepuisLeBas(Widget page) {
  return PageRouteBuilder<void>(
    transitionDuration: const Duration(milliseconds: 400),
    pageBuilder: (context, animation, secondaryAnimation) => page,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      final tween = Tween<Offset>(
        begin: const Offset(0, 1),
        end: Offset.zero,
      ).chain(CurveTween(curve: Curves.easeOutCubic));

      return SlideTransition(position: animation.drive(tween), child: child);
    },
  );
}

/// Zoom combiné à un fondu.
Route<void> routeZoom(Widget page) {
  return PageRouteBuilder<void>(
    transitionDuration: const Duration(milliseconds: 450),
    pageBuilder: (context, animation, secondaryAnimation) => page,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      final courbe = CurvedAnimation(parent: animation, curve: Curves.easeOut);
      return FadeTransition(
        opacity: courbe,
        child: ScaleTransition(
          scale: Tween<double>(begin: 0.85, end: 1).animate(courbe),
          child: child,
        ),
      );
    },
  );
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Transitions')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FilledButton(
              onPressed: () => Navigator.push(
                context,
                routeFondu(const EcranCible(titre: 'Fondu')),
              ),
              child: const Text('Fondu'),
            ),
            const SizedBox(height: 12),
            FilledButton(
              onPressed: () => Navigator.push(
                context,
                routeDepuisLeBas(const EcranCible(titre: 'Depuis le bas')),
              ),
              child: const Text('Depuis le bas'),
            ),
            const SizedBox(height: 12),
            FilledButton(
              onPressed: () => Navigator.push(
                context,
                routeZoom(const EcranCible(titre: 'Zoom')),
              ),
              child: const Text('Zoom'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranCible extends StatelessWidget {
  const EcranCible({super.key, required this.titre});

  final String titre;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(titre)),
      body: Center(
        child: Text(titre, style: Theme.of(context).textTheme.displaySmall),
      ),
    );
  }
}
```

**Résultat :**

```text
Bouton 1 : l'écran apparaît en fondu, sans glisser.
Bouton 2 : l'écran monte depuis le bas de la fenêtre en 400 ms.
Bouton 3 : l'écran grandit de 85 % à 100 % en s'opacifiant.
Chaque retour rejoue l'animation à l'envers.
```

---

## 50.24.1 — Rendre la transition réutilisable

Notez que les trois transitions sont des **fonctions** qui renvoient une `Route`. C'est la bonne pratique : la transition devient un objet nommé qu'on réutilise partout.

```dart
Navigator.push(context, routeFondu(const EcranReglages()));
```

Pour l'appliquer à **toutes** les routes de l'application, on passe par le thème :

```dart
ThemeData(
  pageTransitionsTheme: const PageTransitionsTheme(
    builders: {
      TargetPlatform.android: FadeUpwardsPageTransitionsBuilder(),
      TargetPlatform.iOS: CupertinoPageTransitionsBuilder(),
    },
  ),
)
```

Nous détaillerons `ThemeData` au chapitre 51.

---

## 50.25 — `TabBar`, `TabBarView` et `DefaultTabController`

Les onglets ne sont **pas** de la navigation empilée. Ils affichent plusieurs vues **au même niveau**, sans empiler ni dépiler.

Trois widgets travaillent ensemble :

| Widget | Rôle |
| --- | --- |
| `TabBar` | La rangée d'onglets cliquables |
| `TabBarView` | La zone qui affiche le contenu de l'onglet actif |
| `TabController` | L'objet qui garde l'index courant et synchronise les deux |

Le `TabController` peut être créé à la main (avec un `State` et un `TickerProviderStateMixin`), mais dans 90 % des cas `DefaultTabController` suffit : il le crée et le fournit à ses descendants.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranPersonnage(),
    );
  }
}

class EcranPersonnage extends StatelessWidget {
  const EcranPersonnage({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Aria — niveau 42'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.person), text: 'Fiche'),
              Tab(icon: Icon(Icons.backpack), text: 'Sac'),
              Tab(icon: Icon(Icons.auto_awesome), text: 'Sorts'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            _OngletFiche(),
            _OngletSac(),
            _OngletSorts(),
          ],
        ),
      ),
    );
  }
}

class _OngletFiche extends StatelessWidget {
  const _OngletFiche();

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(16),
      children: const [
        ListTile(leading: Icon(Icons.favorite), title: Text('Points de vie'), trailing: Text('310')),
        ListTile(leading: Icon(Icons.bolt), title: Text('Énergie'), trailing: Text('120')),
        ListTile(leading: Icon(Icons.shield), title: Text('Armure'), trailing: Text('48')),
      ],
    );
  }
}

class _OngletSac extends StatelessWidget {
  const _OngletSac();

  static const List<String> objets = [
    'Arc long elfique',
    'Potion de soin x3',
    'Corde de 15 mètres',
    'Carte du donjon',
  ];

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: objets.length,
      itemBuilder: (context, index) => ListTile(
        leading: const Icon(Icons.inventory_2),
        title: Text(objets[index]),
      ),
    );
  }
}

class _OngletSorts extends StatelessWidget {
  const _OngletSorts();

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Text('Aria ne connaît aucun sort.'),
    );
  }
}
```

**Résultat :**

```text
Une AppBar avec trois onglets sous le titre.
On tape « Sac » : la vue glisse vers la gauche et montre l'inventaire.
On peut aussi balayer horizontalement pour changer d'onglet.
L'onglet actif est souligné.
```

---

## 50.25.1 — Les points à connaître

- **`length` doit être égal au nombre d'onglets ET au nombre d'enfants du `TabBarView`.** Sinon :

```text
Controller's length property (3) does not match the number of children
present in TabBarView (2)
```

- **Les onglets ne modifient pas la pile de navigation.** Le bouton retour du système ferme l'application, il ne revient pas à l'onglet précédent. C'est le comportement attendu.
- **`TabBarView` reconstruit l'onglet quand on y revient.** Si vous voulez conserver la position de défilement d'un onglet, ajoutez `AutomaticKeepAliveClientMixin` à son `State`, ou utilisez `IndexedStack` (section 50.27).
- Pour lire ou changer l'onglet courant depuis un descendant : `DefaultTabController.of(context)`.

```dart
final controleur = DefaultTabController.of(context);
controleur.animateTo(2);          // aller à l'onglet « Sorts »
print(controleur.index);          // l'index courant
```

---

## 50.26 — `BottomNavigationBar` et `NavigationBar`

Une barre basse permet de basculer entre les **sections principales** d'une application : Accueil, Guilde, Boutique, Profil.

Deux widgets existent :

| Widget | Version | À utiliser ? |
| --- | --- | --- |
| `BottomNavigationBar` | Material 2 | Existant, encore supporté |
| `NavigationBar` | Material 3 | **Oui**, c'est le widget actuel |

Comme `useMaterial3: true` est le comportement par défaut, utilisez `NavigationBar`.

```dart
NavigationBar({
  int selectedIndex = 0,
  required List<Widget> destinations,
  ValueChanged<int>? onDestinationSelected,
  NavigationDestinationLabelBehavior? labelBehavior,
  double? height,
  Color? backgroundColor,
  Color? indicatorColor,
})
```

et chaque destination :

```dart
NavigationDestination({
  required Widget icon,
  Widget? selectedIcon,
  required String label,
  String? tooltip,
  bool enabled = true,
})
```

Le principe est celui du chapitre 45 : **la barre ne mémorise rien**. C'est votre `State` qui garde l'index et le repasse à la barre.

```text
     appui sur une destination
              |
              v
   onDestinationSelected(index)
              |
              v
   setState(() => _index = index)
              |
              v
   build() repasse selectedIndex: _index
```

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranPrincipal(),
    );
  }
}

class EcranPrincipal extends StatefulWidget {
  const EcranPrincipal({super.key});

  @override
  State<EcranPrincipal> createState() => _EcranPrincipalState();
}

class _EcranPrincipalState extends State<EcranPrincipal> {
  int _index = 0;

  static const List<String> _titres = ['Accueil', 'Guilde', 'Boutique'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_titres[_index])),
      body: Center(
        child: Text(
          'Section : ${_titres[_index]}',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: (index) => setState(() => _index = index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'Accueil',
          ),
          NavigationDestination(
            icon: Icon(Icons.groups_outlined),
            selectedIcon: Icon(Icons.groups),
            label: 'Guilde',
          ),
          NavigationDestination(
            icon: Icon(Icons.storefront_outlined),
            selectedIcon: Icon(Icons.storefront),
            label: 'Boutique',
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Une barre basse à trois destinations.
La destination active reçoit une pastille colorée derrière son icône,
et son icône passe de la version « contour » à la version pleine.
Le titre de l'AppBar suit la section choisie.
```

---

## 50.26.1 — L'ancienne `BottomNavigationBar`

Vous la rencontrerez dans des projets existants :

```dart
BottomNavigationBar(
  currentIndex: _index,
  onTap: (index) => setState(() => _index = index),
  type: BottomNavigationBarType.fixed,
  items: const [
    BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Accueil'),
    BottomNavigationBarItem(icon: Icon(Icons.groups), label: 'Guilde'),
    BottomNavigationBarItem(icon: Icon(Icons.storefront), label: 'Boutique'),
  ],
)
```

Deux différences de vocabulaire à retenir : `currentIndex` au lieu de `selectedIndex`, `onTap` au lieu de `onDestinationSelected`, et `items` au lieu de `destinations`.

Le piège célèbre : avec plus de trois entrées, `BottomNavigationBar` bascule en mode `shifting` et **masque les libellés** des onglets inactifs. D'où le `type: BottomNavigationBarType.fixed` presque toujours nécessaire. `NavigationBar` n'a pas ce problème.

---

## 50.27 — Conserver l'état des onglets avec `IndexedStack`

Dans l'exemple de 50.26, chaque section est reconstruite de zéro à chaque changement d'onglet. Conséquence :

```text
- la position de défilement d'une liste est perdue
- le texte tapé dans un champ est perdu
- une requête réseau est relancée
```

Une solution naïve consiste à écrire :

```dart
body: [
  const SectionAccueil(),
  const SectionGuilde(),
  const SectionBoutique(),
][_index],
```

Cela construit **uniquement** la section visible, donc l'état des deux autres n'existe pas.

`IndexedStack` (vu au chapitre 46) résout le problème : il construit **tous** ses enfants, les garde vivants, et n'en affiche qu'un.

```dart
body: IndexedStack(
  index: _index,
  children: const [
    SectionAccueil(),
    SectionGuilde(),
    SectionBoutique(),
  ],
),
```

```text
        IndexedStack(index: 1)

        [ SectionAccueil  ]  construite, vivante, invisible
        [ SectionGuilde   ]  construite, vivante, VISIBLE
        [ SectionBoutique ]  construite, vivante, invisible
```

Programme complet, avec une preuve visible : un compteur par section.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranPrincipal(),
    );
  }
}

class EcranPrincipal extends StatefulWidget {
  const EcranPrincipal({super.key});

  @override
  State<EcranPrincipal> createState() => _EcranPrincipalState();
}

class _EcranPrincipalState extends State<EcranPrincipal> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('État conservé')),
      body: IndexedStack(
        index: _index,
        children: const [
          Compteur(titre: 'Ennemis vaincus', couleur: Colors.red),
          Compteur(titre: 'Potions bues', couleur: Colors.green),
          Compteur(titre: 'Pièces d\'or', couleur: Colors.amber),
        ],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: (index) => setState(() => _index = index),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.dangerous), label: 'Ennemis'),
          NavigationDestination(icon: Icon(Icons.local_drink), label: 'Potions'),
          NavigationDestination(icon: Icon(Icons.paid), label: 'Or'),
        ],
      ),
    );
  }
}

class Compteur extends StatefulWidget {
  const Compteur({super.key, required this.titre, required this.couleur});

  final String titre;
  final Color couleur;

  @override
  State<Compteur> createState() => _CompteurState();
}

class _CompteurState extends State<Compteur> {
  int _valeur = 0;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(widget.titre, style: Theme.of(context).textTheme.titleLarge),
          const SizedBox(height: 12),
          Text(
            '$_valeur',
            style: TextStyle(fontSize: 72, color: widget.couleur),
          ),
          const SizedBox(height: 24),
          FilledButton(
            onPressed: () => setState(() => _valeur++),
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
Section « Ennemis » : on appuie 5 fois sur +1  -> 5
On passe à « Potions », on appuie 2 fois       -> 2
On revient à « Ennemis »                       -> 5 (conservé)
```

Sans `IndexedStack`, le compteur serait revenu à 0.

---

## 50.27.1 — Le coût d'`IndexedStack`

`IndexedStack` construit **tout**, tout le temps. C'est un compromis :

| Avantage | Coût |
| --- | --- |
| L'état est conservé | Toutes les sections occupent de la mémoire |
| Le changement d'onglet est instantané | Le premier `build` est plus lent |
| Aucun rechargement réseau | Une section invisible peut consommer des ressources |

Pour trois ou quatre sections légères, c'est le bon choix. Pour dix sections lourdes, préférez `AutomaticKeepAliveClientMixin` appliqué seulement aux sections qui en ont besoin.

Notez aussi que `IndexedStack` prend la taille de son **plus grand** enfant, ce qui peut surprendre si une section est beaucoup plus haute que les autres.

---

## 50.28 — `Drawer` et `NavigationDrawer`

Un **tiroir** est le panneau latéral qui glisse depuis le bord gauche. Il sert aux sections secondaires : réglages, aide, déconnexion.

`Scaffold` a un paramètre `drawer`. Dès qu'il est renseigné, l'`AppBar` affiche automatiquement l'icône « hamburger » et le geste de balayage depuis le bord gauche est activé.

Deux widgets possibles :

- `Drawer` : le conteneur générique, dont vous remplissez le `child` avec ce que vous voulez ;
- `NavigationDrawer` (Material 3) : un `Drawer` déjà organisé en destinations, avec sélection et pastille.

```dart
NavigationDrawer({
  required List<Widget> children,
  ValueChanged<int>? onDestinationSelected,
  int? selectedIndex = 0,
  Color? backgroundColor,
  Color? indicatorColor,
})
```

```dart
NavigationDrawerDestination({
  required Widget icon,
  Widget? selectedIcon,
  required Widget label,
  bool enabled = true,
})
```

Attention à un détail comptable : `onDestinationSelected` renvoie l'index **parmi les destinations seulement**. Les `Divider`, `Padding` et `Text` placés dans `children` ne sont **pas** comptés.

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranPrincipal(),
    );
  }
}

class EcranPrincipal extends StatefulWidget {
  const EcranPrincipal({super.key});

  @override
  State<EcranPrincipal> createState() => _EcranPrincipalState();
}

class _EcranPrincipalState extends State<EcranPrincipal> {
  int _index = 0;

  static const List<String> _titres = ['Guilde', 'Boutique', 'Réglages'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_titres[_index])),
      drawer: NavigationDrawer(
        selectedIndex: _index,
        onDestinationSelected: (index) {
          setState(() => _index = index);
          Navigator.pop(context); // referme le tiroir
        },
        children: [
          Padding(
            padding: const EdgeInsets.fromLTRB(28, 24, 16, 12),
            child: Row(
              children: [
                const CircleAvatar(child: Text('A')),
                const SizedBox(width: 12),
                Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Aria',
                        style: Theme.of(context).textTheme.titleMedium),
                    Text('Niveau 42',
                        style: Theme.of(context).textTheme.bodySmall),
                  ],
                ),
              ],
            ),
          ),
          const Divider(),
          const NavigationDrawerDestination(
            icon: Icon(Icons.groups_outlined),
            selectedIcon: Icon(Icons.groups),
            label: Text('Guilde'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.storefront_outlined),
            selectedIcon: Icon(Icons.storefront),
            label: Text('Boutique'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: Text('Réglages'),
          ),
        ],
      ),
      body: Center(
        child: Text(
          'Section : ${_titres[_index]}',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
L'AppBar affiche une icône hamburger à gauche.
On tape dessus (ou on balaie depuis le bord gauche) :
un panneau s'ouvre avec l'avatar d'Aria puis trois destinations.
On choisit « Boutique » : le tiroir se referme,
le titre et le contenu changent.
```

---

## 50.28.1 — Le `Drawer` générique

Quand la structure ne se réduit pas à une liste de destinations, utilisez le `Drawer` classique :

```dart
drawer: Drawer(
  child: ListView(
    padding: EdgeInsets.zero,
    children: [
      const DrawerHeader(
        decoration: BoxDecoration(color: Colors.indigo),
        child: Text('Chroniques de la guilde',
            style: TextStyle(color: Colors.white, fontSize: 22)),
      ),
      ListTile(
        leading: const Icon(Icons.groups),
        title: const Text('Guilde'),
        onTap: () {
          Navigator.pop(context);               // referme le tiroir
          Navigator.pushNamed(context, '/guilde');
        },
      ),
      const Divider(),
      ListTile(
        leading: const Icon(Icons.logout),
        title: const Text('Se déconnecter'),
        onTap: () {
          Navigator.pop(context);
          Navigator.pushNamedAndRemoveUntil(
            context, '/connexion', (route) => false);
        },
      ),
    ],
  ),
),
```

Retenez l'ordre : **`Navigator.pop(context)` d'abord** (le tiroir est une route), **la navigation ensuite**. L'inverse laisse le tiroir ouvert par-dessus le nouvel écran.

---

## 50.28.2 — Ouvrir et fermer le tiroir par programme

```dart
Scaffold.of(context).openDrawer();
Scaffold.of(context).closeDrawer();
```

`Scaffold.of(context)` exige un `context` situé **sous** le `Scaffold`. Depuis le `build` qui crée le `Scaffold`, il faut donc un `Builder` :

```dart
Scaffold(
  appBar: AppBar(
    leading: Builder(
      builder: (context) => IconButton(
        icon: const Icon(Icons.menu_open),
        onPressed: () => Scaffold.of(context).openDrawer(),
      ),
    ),
  ),
  drawer: const Drawer(child: SizedBox()),
  body: const SizedBox(),
)
```

C'est le même piège de `context` qu'en 50.3.2, avec la même famille de solutions.

---

## 50.29 — `endDrawer`

`Scaffold` accepte un second tiroir, ouvert depuis le bord **droit** : `endDrawer`.

```dart
Scaffold(
  appBar: AppBar(title: const Text('Donjon')),
  drawer: const Drawer(child: Center(child: Text('Menu'))),
  endDrawer: const Drawer(child: Center(child: Text('Filtres'))),
  body: const SizedBox(),
)
```

Différences avec `drawer` :

| Point | `drawer` | `endDrawer` |
| --- | --- | --- |
| Bord d'ouverture | Gauche (droite en langue RTL) | Droite (gauche en RTL) |
| Icône automatique | Dans `leading` | Dans `actions`, à droite |
| Ouverture par code | `openDrawer()` | `openEndDrawer()` |
| Usage habituel | Navigation principale | Filtres, options, panier |

Programme complet :

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
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const EcranCatalogue(),
    );
  }
}

class EcranCatalogue extends StatefulWidget {
  const EcranCatalogue({super.key});

  @override
  State<EcranCatalogue> createState() => _EcranCatalogueState();
}

class _EcranCatalogueState extends State<EcranCatalogue> {
  final Set<String> _categories = {'Armes'};

  static const List<String> _toutesCategories = [
    'Armes',
    'Armures',
    'Potions',
    'Grimoires',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Boutique')),
      drawer: const Drawer(
        child: Center(child: Text('Navigation principale')),
      ),
      endDrawer: Drawer(
        child: SafeArea(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text('Filtres',
                    style: Theme.of(context).textTheme.titleLarge),
              ),
              const Divider(height: 1),
              for (final categorie in _toutesCategories)
                CheckboxListTile(
                  title: Text(categorie),
                  value: _categories.contains(categorie),
                  onChanged: (coche) {
                    setState(() {
                      if (coche ?? false) {
                        _categories.add(categorie);
                      } else {
                        _categories.remove(categorie);
                      }
                    });
                  },
                ),
              const Spacer(),
              Padding(
                padding: const EdgeInsets.all(16),
                child: SizedBox(
                  width: double.infinity,
                  child: FilledButton(
                    onPressed: () => Navigator.pop(context),
                    child: const Text('Appliquer'),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
      body: Center(
        child: Text(
          _categories.isEmpty
              ? 'Aucune catégorie sélectionnée.'
              : 'Catégories : ${_categories.join(', ')}',
          textAlign: TextAlign.center,
          style: Theme.of(context).textTheme.titleMedium,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
L'AppBar a un hamburger à gauche ET une icône de tiroir à droite.
On ouvre celui de droite, on coche « Potions » et « Grimoires ».
Le corps affiche « Catégories : Armes, Potions, Grimoires ».
« Appliquer » referme le tiroir avec Navigator.pop.
```

---

## 50.30 — Les limites de `Navigator` 1.0

Tout ce que vous avez vu jusqu'ici s'appelle le `Navigator` **impératif**, ou `Navigator` 1.0 : on donne des ordres (« empile », « dépile ») et l'état résulte de la suite des ordres.

Cela fonctionne parfaitement pour une application mobile simple. Cela atteint ses limites dans quatre situations.

**Limite 1 — l'état de navigation n'est pas déclaratif.**

Vous ne pouvez pas écrire « voici la pile que je veux ». Vous ne pouvez qu'écrire une suite d'opérations qui, espérez-vous, produira cette pile. Quand l'état de l'application change (l'utilisateur se déconnecte, une session expire), il faut retrouver toutes les opérations à effectuer pour ramener la pile dans un état cohérent.

**Limite 2 — les liens profonds et les URL.**

Sur le Web, l'utilisateur peut taper `/joueur/42` dans la barre d'adresse. Avec `Navigator` 1.0 et les routes nommées, Flutter **empile** simplement cette route sur ce qui existait. Résultat : pas d'écran parent en dessous, donc pas de flèche de retour cohérente. La documentation officielle le dit noir sur blanc : « When a deep link is received, Flutter always pushes a new Route regardless of where the user currently is ».

**Limite 3 — le bouton « suivant » du navigateur.**

`Navigator` 1.0 ne le gère pas. Sur le Web, l'utilisateur qui revient en arrière puis appuie sur « suivant » obtient un comportement incohérent.

**Limite 4 — les navigateurs imbriqués.**

Une application à onglets où chaque onglet garde sa propre pile (comportement standard sur iOS) demande un `Navigator` par onglet, avec des clés globales, une gestion manuelle du bouton retour, et beaucoup de code fragile.

Flutter a répondu à ces limites avec `Navigator` 2.0, aussi appelé l'API `Router` : `RouterDelegate`, `RouteInformationParser`, `BackButtonDispatcher`. Cette API est **puissante et pénible**. Personne ne l'écrit à la main.

C'est exactement le vide que `go_router` vient combler.

---

## 50.31 — `go_router` : pourquoi il existe

`go_router` est un paquet **officiel**, maintenu par l'équipe Flutter (éditeur vérifié `flutter.dev` sur pub.dev). Ce n'est pas un paquet communautaire de plus.

Son idée tient en une phrase :

> Vous décrivez votre application comme un **arbre d'URL**. `go_router` calcule la pile de navigation correspondante.

Comparez les deux modèles :

```text
NAVIGATOR 1.0 (impératif)
  « empile le détail du joueur 42 »
  -> la pile dépend de l'historique des ordres

GO_ROUTER (déclaratif)
  « affiche /guilde/42 »
  -> la pile est déduite de l'arbre de routes :
       [ /guilde/42 ]
       [ /guilde    ]
```

Ce que cela apporte concrètement :

| Besoin | `Navigator` 1.0 | `go_router` |
| --- | --- | --- |
| URL propre sur le Web | À faire soi-même | Automatique |
| Lien profond avec pile cohérente | Difficile | Automatique |
| Bouton « suivant » du navigateur | Non géré | Géré |
| Paramètres de chemin `/joueur/:id` | `onGenerateRoute` + parsing manuel | Intégré |
| Redirection d'authentification | Éparpillée dans chaque écran | Centralisée en une fonction |
| Barre de navigation persistante | `IndexedStack` bricolé | `StatefulShellRoute` |

`go_router` n'interdit rien : il construit un `Navigator` par-dessous, et vous pouvez toujours faire un `Navigator.push` ponctuel pour une boîte de dialogue plein écran.

---

## 50.32 — Installer et configurer `go_router`

### Installation

Dans le dossier du projet :

```text
flutter pub add go_router
```

La commande écrit la dépendance dans `pubspec.yaml` avec la dernière version compatible, puis lance `flutter pub get`. Ne recopiez jamais un numéro de version trouvé dans un tutoriel : `go_router` évolue vite (la version majeure courante est la 17).

Votre `pubspec.yaml` contient alors :

```text
dependencies:
  flutter:
    sdk: flutter
  go_router: ^17.5.0
```

### Configuration minimale

Trois changements par rapport à ce que vous connaissez :

1. On crée un objet `GoRouter` avec la liste des routes.
2. On remplace `MaterialApp(...)` par **`MaterialApp.router(...)`**.
3. On passe le routeur via `routerConfig:`.

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MonApp());
}

final GoRouter _routeur = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const EcranAccueil(),
    ),
    GoRoute(
      path: '/boutique',
      builder: (context, state) => const EcranBoutique(),
    ),
  ],
);

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: _routeur,
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Accueil')),
      body: Center(
        child: FilledButton(
          onPressed: () => context.go('/boutique'),
          child: const Text('Ouvrir la boutique'),
        ),
      ),
    );
  }
}

class EcranBoutique extends StatelessWidget {
  const EcranBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Boutique')),
      body: Center(
        child: FilledButton(
          onPressed: () => context.go('/'),
          child: const Text('Revenir à l\'accueil'),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
L'application démarre sur « Accueil ».
Le bouton ouvre « Boutique ».
Sur le Web, l'URL du navigateur devient .../boutique.
```

---

## 50.32.1 — `go` ou `push` ?

`go_router` ajoute des méthodes d'extension sur `BuildContext`.

| Appel | Effet sur la pile |
| --- | --- |
| `context.go('/x')` | **Remplace** la pile par celle que décrit `/x` |
| `context.push('/x')` | **Empile** `/x` par-dessus la pile actuelle |
| `context.pop()` | Dépile la route du sommet |
| `context.replace('/x')` | Remplace la route du sommet par `/x` |
| `context.goNamed('nom')` | Comme `go`, mais par nom de route |
| `context.pushNamed('nom')` | Comme `push`, mais par nom de route |

La règle simple :

> **`go` pour changer de section** (une entrée de barre de navigation, un menu).
> **`push` pour ouvrir un écran par-dessus** (un détail, un formulaire, une confirmation).

`context.push` renvoie un `Future` : le mécanisme de résultat de la section 50.8 fonctionne, et `context.pop(valeur)` renvoie la valeur.

---

## 50.33 — Définir des routes et des sous-routes

Le paramètre `routes` de `GoRoute` permet d'imbriquer.

```dart
GoRoute(
  path: '/guilde',
  builder: (context, state) => const EcranGuilde(),
  routes: [
    GoRoute(
      path: 'membre',                 // pas de '/' au début !
      builder: (context, state) => const EcranMembre(),
    ),
  ],
),
```

Le chemin complet de la sous-route est `/guilde/membre`.

**Règle de syntaxe :** une route de premier niveau commence par `/`, une sous-route **ne commence pas** par `/`. Écrire `path: '/membre'` à l'intérieur de `/guilde` crée une route racine `/membre`, et non `/guilde/membre`. C'est l'erreur numéro un des débutants avec `go_router`.

L'imbrication a une conséquence essentielle : quand on va directement sur `/guilde/membre`, `go_router` construit **les deux routes**.

```text
context.go('/guilde/membre')

Pile construite :
   [ EcranMembre ]   <- visible
   [ EcranGuilde ]   <- parent, empilé automatiquement

La flèche de retour ramène à /guilde, même si
l'utilisateur est arrivé par un lien profond.
```

C'est précisément la limite 2 de la section 50.30, résolue gratuitement.

---

## 50.34 — Les paramètres de chemin et de requête

### Paramètres de chemin

On déclare un segment variable avec `:` :

```dart
GoRoute(
  path: '/joueur/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];   // String?
    return EcranDetail(id: id);
  },
),
```

`state` est un `GoRouterState`. Ses propriétés utiles :

| Propriété | Type | Contenu |
| --- | --- | --- |
| `pathParameters` | `Map<String, String>` | `{'id': '42'}` pour `/joueur/42` |
| `uri` | `Uri` | l'URI complète, avec la requête |
| `uri.queryParameters` | `Map<String, String>` | `{'tri': 'niveau'}` |
| `extra` | `Object?` | un objet Dart transmis hors URL |
| `matchedLocation` | `String` | le chemin apparié jusqu'ici |
| `name` | `String?` | le nom de la route |

Notez que **les paramètres sont toujours des `String`**. Une URL ne transporte que du texte. La conversion est à votre charge, et elle peut échouer :

```dart
final id = int.tryParse(state.pathParameters['id'] ?? '');
if (id == null) {
  return const EcranIntrouvable();
}
```

### Paramètres de requête

Ils ne se déclarent pas : ils sont lus depuis l'URI.

```dart
// URL appelée : /guilde?tri=niveau&ordre=desc
final tri = state.uri.queryParameters['tri'] ?? 'nom';
final ordre = state.uri.queryParameters['ordre'] ?? 'asc';
```

### Le paramètre `extra`

Pour passer un objet Dart complet sans le mettre dans l'URL :

```dart
context.go('/joueur/42', extra: monJoueur);
```

```dart
final joueur = state.extra as Joueur?;
```

Attention : `extra` **ne survit pas** à un rechargement de page Web ni à un lien profond, puisqu'il n'est pas dans l'URL. Traitez-le comme une optimisation d'affichage, jamais comme la seule source de données.

Programme complet couvrant les trois mécanismes :

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MonApp());
}

class Joueur {
  const Joueur({required this.id, required this.nom, required this.niveau});

  final int id;
  final String nom;
  final int niveau;
}

const List<Joueur> guilde = [
  Joueur(id: 1, nom: 'Aria', niveau: 42),
  Joueur(id: 2, nom: 'Borin', niveau: 37),
  Joueur(id: 3, nom: 'Célia', niveau: 45),
  Joueur(id: 4, nom: 'Dorn', niveau: 29),
];

Joueur? chercher(int? id) {
  for (final joueur in guilde) {
    if (joueur.id == id) return joueur;
  }
  return null;
}

final GoRouter _routeur = GoRouter(
  initialLocation: '/guilde',
  routes: [
    GoRoute(
      path: '/guilde',
      builder: (context, state) {
        final tri = state.uri.queryParameters['tri'] ?? 'nom';
        return EcranGuilde(tri: tri);
      },
      routes: [
        GoRoute(
          path: 'joueur/:id',
          builder: (context, state) {
            final id = int.tryParse(state.pathParameters['id'] ?? '');
            final joueur = chercher(id);
            if (joueur == null) {
              return EcranIntrouvable(chemin: state.uri.toString());
            }
            return EcranDetail(joueur: joueur);
          },
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) =>
      EcranIntrouvable(chemin: state.uri.toString()),
);

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: _routeur,
    );
  }
}

class EcranGuilde extends StatelessWidget {
  const EcranGuilde({super.key, required this.tri});

  final String tri;

  @override
  Widget build(BuildContext context) {
    final liste = [...guilde];
    if (tri == 'niveau') {
      liste.sort((a, b) => b.niveau.compareTo(a.niveau));
    } else {
      liste.sort((a, b) => a.nom.compareTo(b.nom));
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('Guilde'),
        actions: [
          PopupMenuButton<String>(
            icon: const Icon(Icons.sort),
            onSelected: (valeur) => context.go('/guilde?tri=$valeur'),
            itemBuilder: (context) => const [
              PopupMenuItem(value: 'nom', child: Text('Trier par nom')),
              PopupMenuItem(value: 'niveau', child: Text('Trier par niveau')),
            ],
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(12),
            child: Text('Tri courant : $tri'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: liste.length,
              itemBuilder: (context, index) {
                final joueur = liste[index];
                return ListTile(
                  leading: CircleAvatar(child: Text('${joueur.id}')),
                  title: Text(joueur.nom),
                  subtitle: Text('Niveau ${joueur.niveau}'),
                  onTap: () => context.go('/guilde/joueur/${joueur.id}'),
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(12),
            child: OutlinedButton(
              onPressed: () => context.go('/guilde/joueur/999'),
              child: const Text('Ouvrir un joueur inexistant'),
            ),
          ),
        ],
      ),
    );
  }
}

class EcranDetail extends StatelessWidget {
  const EcranDetail({super.key, required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(joueur.nom)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(radius: 48, child: Text(joueur.nom[0])),
            const SizedBox(height: 16),
            Text('Niveau ${joueur.niveau}',
                style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 8),
            Text('URL : /guilde/joueur/${joueur.id}'),
          ],
        ),
      ),
    );
  }
}

class EcranIntrouvable extends StatelessWidget {
  const EcranIntrouvable({super.key, required this.chemin});

  final String chemin;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Introuvable')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.search_off, size: 72),
            const SizedBox(height: 16),
            Text('Aucun contenu pour « $chemin ».'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => context.go('/guilde'),
              child: const Text('Retour à la guilde'),
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
La liste s'affiche triée par nom, « Tri courant : nom ».
Le menu de tri passe à /guilde?tri=niveau : la liste se réordonne.
Taper sur « Célia » ouvre /guilde/joueur/3, avec une flèche de retour
qui ramène à /guilde même si on est arrivé par un lien profond.
Le bouton piégé ouvre /guilde/joueur/999 -> écran « Introuvable ».
```

---

## 50.35 — La redirection (garde d'authentification)

Le paramètre `redirect` du `GoRouter` est appelé **avant chaque navigation**. Il reçoit l'état demandé et renvoie :

- `null` : « la navigation demandée est acceptée » ;
- une chaîne : « non, va plutôt là ».

```dart
typedef GoRouterRedirect = FutureOr<String?> Function(
  BuildContext context,
  GoRouterState state,
);
```

C'est **le** point unique où l'on protège les zones privées. Plus de test d'authentification éparpillé dans chaque écran.

Le motif standard comporte deux tests et un piège :

```dart
redirect: (context, state) {
  final connecte = Session.connecte;
  final surConnexion = state.matchedLocation == '/connexion';

  // 1. Non connecté et pas déjà sur la page de connexion -> on y va.
  if (!connecte && !surConnexion) return '/connexion';

  // 2. Connecté mais encore sur la page de connexion -> on en sort.
  if (connecte && surConnexion) return '/';

  // 3. Sinon, on laisse passer.
  return null;
},
```

Le piège est le test `surConnexion` : sans lui, aller sur `/connexion` renverrait vers `/connexion`, qui renverrait vers `/connexion`... `go_router` interrompt la boucle après `redirectLimit` (5 par défaut) et lève :

```text
GoException: too many redirects
```

Programme complet :

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MonApp());
}

/// État d'authentification très simple, observable par le routeur.
class Session extends ChangeNotifier {
  bool _connecte = false;
  String _pseudo = '';

  bool get connecte => _connecte;
  String get pseudo => _pseudo;

  void connecter(String pseudo) {
    _connecte = true;
    _pseudo = pseudo;
    notifyListeners();
  }

  void deconnecter() {
    _connecte = false;
    _pseudo = '';
    notifyListeners();
  }
}

final Session session = Session();

final GoRouter _routeur = GoRouter(
  initialLocation: '/',
  refreshListenable: session,
  redirect: (context, state) {
    final surConnexion = state.matchedLocation == '/connexion';

    if (!session.connecte && !surConnexion) return '/connexion';
    if (session.connecte && surConnexion) return '/';
    return null;
  },
  routes: [
    GoRoute(
      path: '/connexion',
      builder: (context, state) => const EcranConnexion(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => const EcranMenu(),
      routes: [
        GoRoute(
          path: 'coffre',
          builder: (context, state) => const EcranCoffre(),
        ),
      ],
    ),
  ],
);

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: _routeur,
    );
  }
}

class EcranConnexion extends StatefulWidget {
  const EcranConnexion({super.key});

  @override
  State<EcranConnexion> createState() => _EcranConnexionState();
}

class _EcranConnexionState extends State<EcranConnexion> {
  final _controleur = TextEditingController();

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Connexion')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _controleur,
              decoration: const InputDecoration(
                labelText: 'Pseudonyme',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: () {
                final pseudo = _controleur.text.trim();
                if (pseudo.isEmpty) return;
                session.connecter(pseudo);
              },
              child: const Text('Entrer dans le jeu'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranMenu extends StatelessWidget {
  const EcranMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Bonjour ${session.pseudo}'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            tooltip: 'Se déconnecter',
            onPressed: session.deconnecter,
          ),
        ],
      ),
      body: Center(
        child: FilledButton(
          onPressed: () => context.go('/coffre'),
          child: const Text('Ouvrir le coffre'),
        ),
      ),
    );
  }
}

class EcranCoffre extends StatelessWidget {
  const EcranCoffre({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Coffre')),
      body: const Center(child: Text('1 240 pièces d\'or.')),
    );
  }
}
```

**Résultat :**

```text
Au démarrage, on demande '/' mais la redirection envoie sur /connexion.
On saisit « Aria » et on valide :
  notifyListeners() -> refreshListenable -> le routeur réévalue
  -> la redirection renvoie '/' -> le menu s'affiche.
Sur /coffre, on appuie sur l'icône de déconnexion :
  la redirection ramène immédiatement sur /connexion.
```

---

## 50.35.1 — Le rôle de `refreshListenable`

Sans `refreshListenable`, la fonction `redirect` ne serait évaluée qu'au moment d'une navigation. Se déconnecter depuis un écran privé ne provoquerait rien : l'utilisateur resterait sur l'écran privé jusqu'à sa prochaine navigation.

`refreshListenable: session` demande au routeur de **réévaluer les redirections** chaque fois que la `Session` appelle `notifyListeners()`.

`ChangeNotifier` est vu en détail au chapitre 52. Retenez pour l'instant : un objet qui prévient ses abonnés quand il change.

---

## 50.35.2 — Redirection au niveau d'une route

`GoRoute` a aussi un paramètre `redirect`, appliqué à cette seule route :

```dart
GoRoute(
  path: '/admin',
  redirect: (context, state) => session.estAdmin ? null : '/',
  builder: (context, state) => const EcranAdmin(),
),
```

Utilisez la redirection globale pour les règles transversales (être connecté) et la redirection locale pour les règles propres à une branche (être administrateur).

---

## 50.36 — La navigation par URL sur le Web

Quand vous compilez pour le Web (`flutter run -d chrome`), `go_router` synchronise la barre d'adresse du navigateur avec l'état de navigation. Vous obtenez, sans écrire une ligne de plus :

| Fonction | Comportement |
| --- | --- |
| L'URL change quand on navigue | `context.go('/guilde/42')` écrit `.../guilde/42` |
| Le bouton « précédent » | Dépile, comme la flèche de l'`AppBar` |
| Le bouton « suivant » | Ré-avance correctement |
| Taper une URL à la main | Reconstruit la pile complète, parents compris |
| Actualiser la page (F5) | Reconstruit le même écran |
| Partager un lien | Le destinataire arrive au bon endroit |

### Le dièse dans l'URL

Par défaut, Flutter Web utilise la stratégie de hachage :

```text
https://exemple.com/#/guilde/42
```

Le `#` évite d'avoir à configurer le serveur. Pour des URL propres :

```text
https://exemple.com/guilde/42
```

il faut appeler, avant `runApp` :

```dart
import 'package:flutter_web_plugins/url_strategy.dart';

void main() {
  usePathUrlStrategy();
  runApp(const MonApp());
}
```

et **configurer le serveur** pour renvoyer `index.html` sur toutes les routes inconnues. Sans cette configuration serveur, actualiser `/guilde/42` renvoie une erreur 404 du serveur, avant même que Flutter ne démarre.

### Ce que le Web change dans votre code

Trois conséquences pratiques :

1. **Tout écran doit pouvoir se construire à partir de son URL seule.** Le style « on passe l'objet complet » de 50.7.2 ne suffit plus : il faut pouvoir recharger depuis l'identifiant.
2. **`extra` n'est pas fiable.** Après un F5, il vaut `null`.
3. **Il faut une page d'erreur.** L'utilisateur peut taper n'importe quoi : `errorBuilder` est obligatoire, pas optionnel.

---

## 50.37 — Choisir : `Navigator` ou `go_router`

| Critère | `Navigator` 1.0 seul | `go_router` |
| --- | --- | --- |
| Dépendance externe | Aucune | Un paquet (officiel) |
| Courbe d'apprentissage | Très courte | Une demi-journée |
| Nombre d'écrans conseillé | Jusqu'à 5 ou 6 | Au-delà |
| Cible Web | Déconseillé | Recommandé |
| Liens profonds (mobile) | Manuel et fragile | Intégré |
| Bouton « suivant » du navigateur | Non géré | Géré |
| Passage de données typé | Excellent (constructeur) | Bon (`pathParameters` + `extra`) |
| Authentification | Vérification par écran | Une fonction `redirect` centrale |
| Barre de navigation persistante | `IndexedStack` manuel | `StatefulShellRoute` |
| Retour d'une valeur | `await Navigator.push` | `await context.push` |
| Tests unitaires de navigation | Moyens | Bons (on teste des URL) |
| Lisibilité de l'architecture | Diffuse | Un fichier de routes |

**Recommandation pratique :**

```text
Application de moins de 6 écrans, mobile uniquement, pas de lien profond
    -> Navigator.push / pop. N'ajoutez rien.

Application avec authentification, onglets persistants, ou cible Web
    -> go_router dès le premier jour.

Application existante en Navigator 1.0 qui grandit
    -> migrez, mais en une fois : mélanger longtemps les deux modèles
       produit des incohérences de pile difficiles à déboguer.
```

Notez que la documentation officielle déconseille explicitement les **routes nommées** (`MaterialApp.routes`) pour la plupart des applications. Le vrai choix est donc entre `Navigator.push` avec constructeurs, et `go_router`. Les routes nommées sont surtout un patrimoine à savoir lire.

---

## 50.38 — Mini-projet : une application à trois écrans

Il est temps de tout assembler.

### Cahier des charges

Une application de gestion de guilde, en `Navigator` 1.0 (le mini-projet en `go_router` sera l'exercice 10).

```text
ÉCRAN PRINCIPAL — trois onglets dans une NavigationBar
  Onglet 1 « Guilde »     : la liste des membres
  Onglet 2 « Boutique »   : la liste des objets
  Onglet 3 « Profil »     : les statistiques du joueur

FONCTIONS
  - taper un membre ouvre son détail (push)
  - le détail permet d'entraîner le membre (+1 niveau) et
    renvoie le membre modifié au retour (pop avec valeur)
  - le bouton + ouvre un formulaire modal d'ajout de membre
  - le formulaire bloque le retour si des données sont saisies (PopScope)
  - l'état de chaque onglet est conservé (IndexedStack)
  - l'avatar vole de la liste vers le détail (Hero)
  - un endDrawer permet de filtrer la liste par classe
```

### Étape 1 — le modèle

Un `Membre` immuable, avec une méthode `copyAvec` pour produire une version modifiée. C'est le style vu au chapitre 09.

```dart
class Membre {
  const Membre({
    required this.id,
    required this.nom,
    required this.classe,
    required this.niveau,
  });

  final int id;
  final String nom;
  final String classe;
  final int niveau;

  Membre copyAvec({String? nom, String? classe, int? niveau}) {
    return Membre(
      id: id,
      nom: nom ?? this.nom,
      classe: classe ?? this.classe,
      niveau: niveau ?? this.niveau,
    );
  }
}
```

### Étape 2 — la coque à onglets

`IndexedStack` conserve l'état des trois sections, `NavigationBar` pilote l'index.

### Étape 3 — le détail et le retour de données

L'écran de détail renvoie un `Membre` (le membre entraîné) ou `null`.

### Étape 4 — le formulaire protégé

`PopScope` avec `canPop: !_saisiCommencee`.

### Le programme complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

// ---------------------------------------------------------------- MODÈLE

class Membre {
  const Membre({
    required this.id,
    required this.nom,
    required this.classe,
    required this.niveau,
  });

  final int id;
  final String nom;
  final String classe;
  final int niveau;

  Membre copyAvec({String? nom, String? classe, int? niveau}) {
    return Membre(
      id: id,
      nom: nom ?? this.nom,
      classe: classe ?? this.classe,
      niveau: niveau ?? this.niveau,
    );
  }
}

const List<String> classes = ['Guerrier', 'Archer', 'Mage', 'Voleur'];

Color couleurDeClasse(String classe) {
  switch (classe) {
    case 'Guerrier':
      return Colors.brown;
    case 'Archer':
      return Colors.green;
    case 'Mage':
      return Colors.purple;
    default:
      return Colors.blueGrey;
  }
}

// ------------------------------------------------------------ APPLICATION

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Chroniques de la guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranPrincipal(),
    );
  }
}

// ------------------------------------------------------- ÉCRAN PRINCIPAL

class EcranPrincipal extends StatefulWidget {
  const EcranPrincipal({super.key});

  @override
  State<EcranPrincipal> createState() => _EcranPrincipalState();
}

class _EcranPrincipalState extends State<EcranPrincipal> {
  int _index = 0;

  static const List<String> _titres = ['Guilde', 'Boutique', 'Profil'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_titres[_index])),
      body: IndexedStack(
        index: _index,
        children: const [
          SectionGuilde(),
          SectionBoutique(),
          SectionProfil(),
        ],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: (index) => setState(() => _index = index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.groups_outlined),
            selectedIcon: Icon(Icons.groups),
            label: 'Guilde',
          ),
          NavigationDestination(
            icon: Icon(Icons.storefront_outlined),
            selectedIcon: Icon(Icons.storefront),
            label: 'Boutique',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'Profil',
          ),
        ],
      ),
    );
  }
}

// --------------------------------------------------------- SECTION GUILDE

class SectionGuilde extends StatefulWidget {
  const SectionGuilde({super.key});

  @override
  State<SectionGuilde> createState() => _SectionGuildeState();
}

class _SectionGuildeState extends State<SectionGuilde> {
  final List<Membre> _membres = [
    const Membre(id: 1, nom: 'Aria', classe: 'Archer', niveau: 42),
    const Membre(id: 2, nom: 'Borin', classe: 'Guerrier', niveau: 37),
    const Membre(id: 3, nom: 'Célia', classe: 'Mage', niveau: 45),
    const Membre(id: 4, nom: 'Dorn', classe: 'Voleur', niveau: 29),
  ];

  final Set<String> _filtres = {...classes};
  int _prochainId = 5;

  List<Membre> get _visibles =>
      _membres.where((m) => _filtres.contains(m.classe)).toList();

  Future<void> _ouvrirDetail(Membre membre) async {
    final Membre? modifie = await Navigator.push<Membre>(
      context,
      MaterialPageRoute(builder: (context) => EcranDetail(membre: membre)),
    );

    if (!mounted || modifie == null) return;

    setState(() {
      final index = _membres.indexWhere((m) => m.id == modifie.id);
      if (index != -1) _membres[index] = modifie;
    });

    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('${modifie.nom} est niveau ${modifie.niveau}.')),
    );
  }

  Future<void> _ajouter() async {
    final Membre? nouveau = await Navigator.push<Membre>(
      context,
      MaterialPageRoute(
        fullscreenDialog: true,
        builder: (context) => EcranFormulaire(idPropose: _prochainId),
      ),
    );

    if (!mounted || nouveau == null) return;

    setState(() {
      _membres.add(nouveau);
      _prochainId++;
      _filtres.add(nouveau.classe);
    });
  }

  @override
  Widget build(BuildContext context) {
    final visibles = _visibles;

    return Scaffold(
      endDrawer: Drawer(
        child: SafeArea(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text('Filtrer par classe',
                    style: Theme.of(context).textTheme.titleLarge),
              ),
              const Divider(height: 1),
              for (final classe in classes)
                CheckboxListTile(
                  title: Text(classe),
                  value: _filtres.contains(classe),
                  onChanged: (coche) {
                    setState(() {
                      if (coche ?? false) {
                        _filtres.add(classe);
                      } else {
                        _filtres.remove(classe);
                      }
                    });
                  },
                ),
              const Spacer(),
              Padding(
                padding: const EdgeInsets.all(16),
                child: SizedBox(
                  width: double.infinity,
                  child: FilledButton(
                    onPressed: () => Navigator.pop(context),
                    child: const Text('Fermer'),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
      body: visibles.isEmpty
          ? const Center(child: Text('Aucun membre ne correspond au filtre.'))
          : ListView.separated(
              itemCount: visibles.length,
              separatorBuilder: (context, index) => const Divider(height: 1),
              itemBuilder: (context, index) {
                final membre = visibles[index];
                return ListTile(
                  leading: Hero(
                    tag: 'membre-${membre.id}',
                    child: CircleAvatar(
                      backgroundColor: couleurDeClasse(membre.classe),
                      child: Text(
                        membre.nom[0],
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                  title: Text(membre.nom),
                  subtitle: Text('${membre.classe} — niveau ${membre.niveau}'),
                  trailing: const Icon(Icons.chevron_right),
                  onTap: () => _ouvrirDetail(membre),
                );
              },
            ),
      floatingActionButton: Row(
        mainAxisSize: MainAxisSize.min,
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          Builder(
            builder: (context) => FloatingActionButton.small(
              heroTag: 'fab-filtre',
              tooltip: 'Filtrer',
              onPressed: () => Scaffold.of(context).openEndDrawer(),
              child: const Icon(Icons.filter_alt),
            ),
          ),
          const SizedBox(width: 12),
          FloatingActionButton(
            heroTag: 'fab-ajout',
            tooltip: 'Recruter',
            onPressed: _ajouter,
            child: const Icon(Icons.person_add),
          ),
        ],
      ),
    );
  }
}

// ----------------------------------------------------------- ÉCRAN DÉTAIL

class EcranDetail extends StatefulWidget {
  const EcranDetail({super.key, required this.membre});

  final Membre membre;

  @override
  State<EcranDetail> createState() => _EcranDetailState();
}

class _EcranDetailState extends State<EcranDetail> {
  late Membre _membre = widget.membre;

  bool get _modifie => _membre.niveau != widget.membre.niveau;

  void _entrainer() {
    if (_membre.niveau >= 99) return;
    setState(() => _membre = _membre.copyAvec(niveau: _membre.niveau + 1));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_membre.nom),
        actions: [
          if (_modifie)
            TextButton(
              onPressed: () => Navigator.pop(context, _membre),
              child: const Text('Valider'),
            ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: 'membre-${_membre.id}',
              child: CircleAvatar(
                radius: 64,
                backgroundColor: couleurDeClasse(_membre.classe),
                child: Text(
                  _membre.nom[0],
                  style: const TextStyle(fontSize: 48, color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 24),
            Text(_membre.nom,
                style: Theme.of(context).textTheme.headlineMedium),
            Text(_membre.classe),
            const SizedBox(height: 24),
            Text('Niveau ${_membre.niveau}',
                style: Theme.of(context).textTheme.displaySmall),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: _entrainer,
              icon: const Icon(Icons.fitness_center),
              label: const Text('Entraîner (+1)'),
            ),
            const SizedBox(height: 8),
            Text(
              _modifie
                  ? 'Appuyez sur « Valider » pour enregistrer.'
                  : 'Aucun entraînement effectué.',
              style: Theme.of(context).textTheme.bodySmall,
            ),
          ],
        ),
      ),
    );
  }
}

// ------------------------------------------------------- ÉCRAN FORMULAIRE

class EcranFormulaire extends StatefulWidget {
  const EcranFormulaire({super.key, required this.idPropose});

  final int idPropose;

  @override
  State<EcranFormulaire> createState() => _EcranFormulaireState();
}

class _EcranFormulaireState extends State<EcranFormulaire> {
  final _cle = GlobalKey<FormState>();
  final _nom = TextEditingController();
  String _classe = classes.first;

  bool get _saisiCommencee => _nom.text.trim().isNotEmpty;

  @override
  void initState() {
    super.initState();
    _nom.addListener(() => setState(() {}));
  }

  @override
  void dispose() {
    _nom.dispose();
    super.dispose();
  }

  Future<bool> _confirmerAbandon() async {
    final reponse = await showDialog<bool>(
      context: context,
      builder: (dialogContext) => AlertDialog(
        title: const Text('Abandonner le recrutement ?'),
        content: const Text('Les informations saisies seront perdues.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(dialogContext, false),
            child: const Text('Continuer'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(dialogContext, true),
            child: const Text('Abandonner'),
          ),
        ],
      ),
    );
    return reponse ?? false;
  }

  void _valider() {
    if (!_cle.currentState!.validate()) return;
    Navigator.pop(
      context,
      Membre(
        id: widget.idPropose,
        nom: _nom.text.trim(),
        classe: _classe,
        niveau: 1,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return PopScope<Membre>(
      canPop: !_saisiCommencee,
      onPopInvokedWithResult: (didPop, result) async {
        if (didPop) return;
        final quitter = await _confirmerAbandon();
        if (!context.mounted) return;
        if (quitter) Navigator.pop(context);
      },
      child: Scaffold(
        appBar: AppBar(title: const Text('Recruter un membre')),
        body: Form(
          key: _cle,
          child: ListView(
            padding: const EdgeInsets.all(16),
            children: [
              TextFormField(
                controller: _nom,
                decoration: const InputDecoration(
                  labelText: 'Nom',
                  border: OutlineInputBorder(),
                ),
                validator: (v) => (v == null || v.trim().isEmpty)
                    ? 'Le nom est obligatoire.'
                    : null,
              ),
              const SizedBox(height: 16),
              DropdownButtonFormField<String>(
                initialValue: _classe,
                decoration: const InputDecoration(
                  labelText: 'Classe',
                  border: OutlineInputBorder(),
                ),
                items: [
                  for (final c in classes)
                    DropdownMenuItem(value: c, child: Text(c)),
                ],
                onChanged: (v) {
                  if (v != null) setState(() => _classe = v);
                },
              ),
              const SizedBox(height: 24),
              FilledButton(
                onPressed: _valider,
                child: const Text('Recruter'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ------------------------------------------------------- AUTRES SECTIONS

class SectionBoutique extends StatelessWidget {
  const SectionBoutique({super.key});

  static const List<(String, int)> articles = [
    ('Potion de soin', 25),
    ('Arc long', 320),
    ('Grimoire de feu', 780),
    ('Bouclier de fer', 210),
  ];

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: articles.length,
      itemBuilder: (context, index) {
        final (nom, prix) = articles[index];
        return ListTile(
          leading: const Icon(Icons.shopping_bag_outlined),
          title: Text(nom),
          trailing: Text('$prix or'),
        );
      },
    );
  }
}

class SectionProfil extends StatefulWidget {
  const SectionProfil({super.key});

  @override
  State<SectionProfil> createState() => _SectionProfilState();
}

class _SectionProfilState extends State<SectionProfil> {
  int _parties = 0;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const CircleAvatar(radius: 48, child: Text('MJ')),
          const SizedBox(height: 16),
          Text('Maître de guilde',
              style: Theme.of(context).textTheme.titleLarge),
          const SizedBox(height: 24),
          Text('Parties jouées : $_parties',
              style: Theme.of(context).textTheme.headlineSmall),
          const SizedBox(height: 16),
          FilledButton(
            onPressed: () => setState(() => _parties++),
            child: const Text('Jouer une partie'),
          ),
          const SizedBox(height: 8),
          Text(
            'Changez d\'onglet et revenez :\nle compteur est conservé.',
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.bodySmall,
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Onglet Guilde : quatre membres, avatars colorés par classe.
On tape « Célia » : l'avatar violet vole vers le centre du détail.
On appuie 3 fois sur « Entraîner » : niveau 48, « Valider » apparaît.
On valide : retour à la liste, Célia est niveau 48,
et une SnackBar l'annonce.

Le bouton filtre ouvre le tiroir de droite ;
décocher « Mage » retire Célia de la liste.

Le bouton « recruter » ouvre le formulaire modal.
On tape « Elen » puis on appuie sur retour :
  « Abandonner le recrutement ? » [Continuer] [Abandonner].
On choisit Continuer, on complète, on recrute :
  Elen apparaît en bas de la liste, niveau 1.

Onglet Profil : on incrémente le compteur, on passe à Boutique
et on revient : le compteur est conservé (IndexedStack).
```

---

## 50.38.1 — Ce que ce projet démontre

| Section du chapitre | Où c'est utilisé dans le projet |
| --- | --- |
| 50.3 / 50.4 | `Navigator.push` + `MaterialPageRoute` pour le détail |
| 50.4.1 | `fullscreenDialog: true` pour le formulaire |
| 50.5 | `Navigator.pop(context)` pour fermer le tiroir |
| 50.7 | `EcranDetail(membre: membre)` : passage par le constructeur |
| 50.8 / 50.9 / 50.10 | `await Navigator.push<Membre>` puis `pop(context, _membre)` |
| 50.21 / 50.22 | `PopScope` + boîte de confirmation du formulaire |
| 50.23 | `Hero(tag: 'membre-${membre.id}')` |
| 50.26 | `NavigationBar` à trois destinations |
| 50.27 | `IndexedStack` qui conserve le compteur du profil |
| 50.29 | `endDrawer` de filtres |

Deux détails techniques valent d'être notés.

**Le `heroTag` des `FloatingActionButton`.** Deux `FloatingActionButton` sur le même écran partagent par défaut le même tag `Hero`, ce qui déclenche l'erreur « multiple heroes share the same tag ». D'où les `heroTag: 'fab-filtre'` et `heroTag: 'fab-ajout'` explicites.

**Le `Scaffold` imbriqué de `SectionGuilde`.** Un `endDrawer` appartient à un `Scaffold`. Comme la section est à l'intérieur du `Scaffold` principal, elle a son propre `Scaffold` sans `appBar`, ce qui permet d'avoir un tiroir propre à cet onglet. C'est un usage légitime des `Scaffold` imbriqués.

---

## 50.39 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Navigator operation requested with a context that does not include a Navigator` | Le `context` utilisé est celui du widget qui crée `MaterialApp`, donc au-dessus du `Navigator` | Extraire l'écran dans sa propre classe, ou insérer un `Builder` |
| `setState() called after dispose()` | Un `setState` après un `await Navigator.push` alors que le widget a été démonté | Ajouter `if (!mounted) return;` juste après le `await` (`context.mounted` dans un `StatelessWidget`) |
| Écran noir après un `pop` | On a dépilé la dernière route de la pile | Tester `Navigator.canPop(context)` avant, ou utiliser `maybePop` |
| `Could not find a generator for route RouteSettings("/xxx", null)` | Route nommée absente de `routes`, ou faute de frappe | Déclarer la route, centraliser les noms dans des constantes, prévoir `onUnknownRoute` |
| `type 'Null' is not a subtype of type 'Joueur' in type cast` | `arguments` non fourni et casté avec `as` | Lire dans une variable, tester avec `is!` et rendre un écran d'erreur |
| `type 'bool' is not a subtype of type 'int'` | Résultat de `push` non typé, donc `dynamic` | Écrire `Navigator.push<bool>(...)` et déclarer `final bool? r = ...` |
| Le résultat attendu vaut toujours `null` | L'écran a été fermé par la flèche ou le bouton système, pas par `pop(context, valeur)` | Traiter explicitement le cas `null` (annulation) |
| `There are multiple heroes that share the same tag within a subtree` | Deux `Hero` (ou deux `FloatingActionButton`) avec le même tag sur le même écran | Rendre le tag unique : `'avatar-${joueur.id}'`, ou fixer `heroTag:` |
| L'animation `Hero` ne se produit pas | Les tags des deux écrans ne sont pas identiques | Comparer les deux chaînes caractère par caractère ; utiliser une constante partagée |
| `If the home property is specified, the routes table cannot include an entry for "/"` | `home:` et `routes: {'/': ...}` coexistent | Choisir l'un des deux |
| `Controller's length property (3) does not match the number of children present in TabBarView (2)` | `DefaultTabController(length:)` incohérent avec les enfants | Aligner `length`, le nombre de `Tab` et le nombre d'enfants du `TabBarView` |
| Le tiroir reste ouvert par-dessus le nouvel écran | On a navigué sans fermer le `Drawer` | `Navigator.pop(context)` d'abord, navigation ensuite |
| L'état d'un onglet est perdu à chaque changement | Le corps est reconstruit par un `[...][index]` | Utiliser `IndexedStack`, ou `AutomaticKeepAliveClientMixin` |
| La boîte « voulez-vous quitter ? » s'affiche alors que l'écran se ferme | `if (didPop) return;` oublié dans `onPopInvokedWithResult` | Sortir immédiatement quand `didPop` vaut `true` |
| `WillPopScope`/`onPopInvoked` signalés comme obsolètes | Ancienne API antérieure à Flutter 3.12 puis 3.22 | Utiliser `PopScope` avec `onPopInvokedWithResult` |
| Le dialogue ne se ferme pas, l'écran se ferme à sa place | `Navigator.pop` appelé avec le `context` de l'écran au lieu de celui du dialogue | Nommer le paramètre du `builder` `dialogContext` et l'utiliser |
| `GoException: too many redirects` | La fonction `redirect` renvoie la page vers laquelle elle redirige déjà | Ajouter le test `state.matchedLocation == '/connexion'` |
| Une sous-route `go_router` devient une route racine | Le `path` d'une sous-route commence par `/` | Retirer le `/` initial des sous-routes |
| Sur le Web, F5 sur `/guilde/42` renvoie une 404 du serveur | `usePathUrlStrategy()` sans configuration serveur | Configurer le serveur pour servir `index.html`, ou garder la stratégie de hachage |
| `state.extra` vaut `null` après un rechargement Web | `extra` n'est pas dans l'URL | Ne jamais dépendre d'`extra` : recharger depuis l'identifiant du chemin |
| `popUntil` produit un écran noir | Prédicat `(route) => false`, jamais satisfait | Utiliser `(route) => route.isFirst` ou `ModalRoute.withName('/x')` |
| `ModalRoute.withName('/menu')` ne trouve jamais rien | Les routes créées dans `onGenerateRoute` n'ont pas reçu `settings:` | Passer `settings: settings` à chaque route générée |

---

## 50.40 — Résumé du chapitre

### Effet de chaque méthode sur la pile

| Méthode | Effet sur la pile |
| --- | --- |
| `Navigator.push(context, route)` | Ajoute une route au sommet. Hauteur +1. Renvoie un `Future` complété au `pop`. |
| `Navigator.pushNamed(context, '/x')` | Identique, en résolvant `/x` via `routes`, `onGenerateRoute` puis `onUnknownRoute`. |
| `Navigator.pop(context)` | Retire la route du sommet. Hauteur −1. Le `Future` du `push` se complète avec `null`. |
| `Navigator.pop(context, valeur)` | Identique, mais le `Future` se complète avec `valeur`. |
| `Navigator.maybePop(context)` | Retire le sommet **si** les `PopScope` l'autorisent et si la pile le permet. Hauteur −1 ou inchangée. |
| `Navigator.canPop(context)` | Ne modifie rien. Renvoie `true` si la hauteur est ≥ 2. |
| `Navigator.pushReplacement(context, route)` | Retire le sommet et empile la nouvelle route. Hauteur inchangée. |
| `Navigator.pushReplacementNamed(context, '/x')` | Idem, par nom. |
| `Navigator.pushAndRemoveUntil(context, route, (r) => false)` | Vide toute la pile puis empile la route. Hauteur = 1. |
| `Navigator.pushAndRemoveUntil(context, route, ModalRoute.withName('/'))` | Dépile jusqu'à `/` inclus dans la pile, puis empile. Hauteur = 2. |
| `Navigator.pushNamedAndRemoveUntil(context, '/x', (r) => false)` | Idem, par nom. Hauteur = 1. |
| `Navigator.popUntil(context, (r) => r.isFirst)` | Dépile jusqu'à la route du fond. Hauteur = 1. Rien n'est empilé. |
| `Navigator.popUntil(context, ModalRoute.withName('/x'))` | Dépile jusqu'à retrouver `/x` au sommet. |
| `showDialog` / `showModalBottomSheet` | Empilent une route modale. Hauteur +1 ; `pop` les referme. |
| `context.go('/x')` (`go_router`) | **Remplace** la pile par celle que décrit l'arbre de routes pour `/x`. |
| `context.push('/x')` (`go_router`) | Empile `/x` par-dessus la pile actuelle. Hauteur +1. |
| `context.pop()` (`go_router`) | Dépile le sommet. Hauteur −1. |
| `context.replace('/x')` (`go_router`) | Remplace la route du sommet. Hauteur inchangée. |

### Les notions clés

| Notion | À retenir |
| --- | --- |
| Pile de navigation | Le sommet est visible, le reste est caché mais vivant. Pile vide = application fermée. |
| `MaterialPageRoute` | La route standard. `builder` est une fonction, `fullscreenDialog` change l'animation et l'icône. |
| Passage de données | Par le **constructeur** : typé, vérifié à la compilation. C'est la méthode recommandée. |
| Retour de données | `await Navigator.push<T>` puis `Navigator.pop(context, valeur)`. Le résultat est toujours `T?`. |
| `mounted` | À tester après chaque `await` avant d'utiliser `context` ou `setState`. |
| Routes nommées | `Map<String, WidgetBuilder>` dans `MaterialApp.routes`. Pratiques mais non typées, et déconseillées par la documentation. |
| `arguments` | Un seul `Object?`. À vérifier avec `is!` avant usage, jamais avec un `as` nu. |
| `onGenerateRoute` | Génère des routes dynamiques ; toujours transmettre `settings:`. |
| `onUnknownRoute` | La page 404. Obligatoire sur le Web. |
| `PopScope` | `canPop: false` bloque le retour ; `onPopInvokedWithResult` réagit, et commence par `if (didPop) return;`. |
| `Hero` | Même tag des deux côtés, tag unique par écran, formes simples. |
| `PageRouteBuilder` | Transition personnalisée : `pageBuilder` construit, `transitionsBuilder` anime. |
| Onglets | `DefaultTabController` + `TabBar` + `TabBarView`, avec `length` cohérent. Ne touchent pas la pile. |
| `NavigationBar` | Le widget Material 3 de la barre basse. L'index est dans votre `State`. |
| `IndexedStack` | Conserve l'état de toutes les sections, au prix de leur construction permanente. |
| `Drawer` / `endDrawer` | Deux tiroirs par `Scaffold`. Toujours fermer le tiroir avant de naviguer. |
| Limites de `Navigator` 1.0 | Impératif, liens profonds bancals, bouton « suivant » ignoré, navigateurs imbriqués pénibles. |
| `go_router` | Paquet officiel, déclaratif, URL-centré : `MaterialApp.router` + `routerConfig`. |
| Paramètres `go_router` | `state.pathParameters` (chemin), `state.uri.queryParameters` (requête), `state.extra` (non persistant). |
| `redirect` | Le point unique de contrôle d'accès, associé à `refreshListenable`. |
| Choix | Moins de six écrans et mobile seul : `Navigator`. Authentification, onglets persistants ou Web : `go_router`. |

---

## 50.41 — Exercices

Faites-les dans l'ordre : chacun réutilise le précédent. Écrivez le code avant de lire la correction.

### Exercice 1 — Le donjon à deux salles (facile)

Écrivez une application de deux écrans.

- L'écran `SalleEntree` affiche « Entrée du donjon » et un bouton « Descendre ».
- Le bouton empile l'écran `SalleTresor`, qui affiche « Salle du trésor » et un bouton « Remonter ».
- « Remonter » dépile l'écran.

Contrainte : le bouton retour du système doit fonctionner sans code supplémentaire.

### Exercice 2 — La fiche d'ennemi (facile)

Créez une classe `Ennemi` avec `nom`, `type`, `pointsDeVie` et `degats`.

Affichez une `ListView` de quatre ennemis. Taper un ennemi ouvre un écran de détail qui affiche ses quatre propriétés.

Contrainte : les données passent **par le constructeur**, pas par `arguments`.

### Exercice 3 — Le choix de l'arme (facile)

Un écran principal affiche « Arme équipée : aucune ».

Un bouton ouvre un écran listant cinq armes. Taper une arme renvoie son nom à l'écran principal, qui l'affiche.

Contraintes :

- typez le résultat avec `Navigator.push<String>` ;
- si l'utilisateur revient par la flèche, l'arme équipée ne change pas ;
- vérifiez `mounted` après le `await`.

### Exercice 4 — Les routes nommées et la page 404 (moyen)

Reprenez trois écrans : `/` (menu), `/inventaire` et `/carte`.

Contraintes :

- déclarez-les dans `MaterialApp.routes` ;
- centralisez les noms dans une classe `Routes` avec des `static const String` ;
- ajoutez un bouton « Ouvrir /donjon-secret » qui déclenche `onUnknownRoute` ;
- la page 404 affiche le nom demandé et un bouton de retour.

### Exercice 5 — Les arguments d'une route nommée (moyen)

Reprenez l'exercice 2, mais faites passer l'ennemi par `arguments` d'une route nommée `/ennemi`.

Contraintes :

- l'écran de détail lit `ModalRoute.of(context)?.settings.arguments` ;
- il **ne plante pas** si l'argument est absent ou du mauvais type : il affiche un message ;
- ajoutez un bouton qui ouvre `/ennemi` **sans** argument, pour tester ce cas.

### Exercice 6 — Le cycle complet d'une partie (moyen)

Enchaînez : écran de chargement → menu → combat 1 → combat 2 → écran de victoire.

Contraintes :

- l'écran de chargement dure 2 secondes puis appelle `pushReplacement` vers le menu ;
- depuis la victoire, un bouton « Retour au menu » utilise `pushAndRemoveUntil` avec `(route) => false` ;
- après ce retour, le bouton système doit quitter l'application ;
- affichez sur chaque écran la valeur de `Navigator.canPop(context)`.

### Exercice 7 — Le formulaire protégé (moyen)

Un écran ouvre un formulaire de création de personnage (un `TextField` pour le nom, un `Slider` pour la force).

Contraintes :

- tant que le champ nom est vide et le curseur inchangé, le retour est libre ;
- dès qu'une valeur est modifiée, le retour ouvre une boîte « Abandonner la création ? » ;
- « Continuer » reste sur le formulaire, « Abandonner » ferme sans rien renvoyer ;
- le bouton « Créer » renvoie un objet `Personnage` à l'écran appelant.

### Exercice 8 — Onglets et animation partagée (moyen)

Un écran à trois onglets (`DefaultTabController`) : « Armes », « Armures », « Potions ».

Chaque onglet est une grille d'objets colorés. Taper un objet ouvre son détail avec une animation `Hero`.

Contraintes :

- le tag `Hero` doit être unique pour chaque objet de l'application entière ;
- le détail utilise une transition personnalisée en fondu (`PageRouteBuilder`).

### Exercice 9 — La coque complète (difficile)

Construisez une coque d'application avec :

- une `NavigationBar` à trois sections ;
- un `IndexedStack` qui conserve l'état de chaque section ;
- un `Drawer` à gauche qui permet aussi de changer de section, puis se referme ;
- un `endDrawer` à droite contenant un réglage (un `Switch` « mode difficile ») ;
- la section 1 contient un compteur ; changer d'onglet et revenir doit le conserver.

### Exercice 10 — La même application en `go_router` (difficile)

Réécrivez une application de guilde avec `go_router` :

- `/connexion` : un champ pseudonyme et un bouton ;
- `/guilde` : la liste des membres, accessible seulement si connecté ;
- `/guilde/membre/:id` : le détail d'un membre ;
- une redirection globale qui protège `/guilde` ;
- un `errorBuilder` pour les URL inconnues ;
- un bouton de déconnexion qui ramène automatiquement à `/connexion`.

Contraintes :

- utilisez `refreshListenable` avec un `ChangeNotifier` ;
- le détail se reconstruit à partir du seul `:id`, pas d'un objet passé en `extra`.

---

## 50.42 — Corrections des exercices

### Correction 1

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
      title: 'Donjon',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.brown),
        useMaterial3: true,
      ),
      home: const SalleEntree(),
    );
  }
}

class SalleEntree extends StatelessWidget {
  const SalleEntree({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Donjon')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.door_front_door, size: 96),
            const SizedBox(height: 16),
            Text(
              'Entrée du donjon',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 32),
            FilledButton.icon(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const SalleTresor(),
                  ),
                );
              },
              icon: const Icon(Icons.arrow_downward),
              label: const Text('Descendre'),
            ),
          ],
        ),
      ),
    );
  }
}

class SalleTresor extends StatelessWidget {
  const SalleTresor({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Salle du trésor')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.diamond, size: 96, color: Colors.amber),
            const SizedBox(height: 16),
            Text(
              'Salle du trésor',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 32),
            OutlinedButton.icon(
              onPressed: () => Navigator.pop(context),
              icon: const Icon(Icons.arrow_upward),
              label: const Text('Remonter'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** deux classes, deux `Scaffold`, une seule ligne de navigation dans chaque sens. `Navigator.push` empile `SalleTresor` (hauteur 2), `Navigator.pop` la retire (hauteur 1). La flèche de retour dans l'`AppBar` et la prise en charge du bouton système sont fournies automatiquement par le `Navigator` : c'est exactement ce que la version « booléen » de la section 50.1 ne savait pas faire. Notez que chaque écran est une classe séparée, ce qui garantit un `context` situé sous le `Navigator` (piège de la section 50.3.2).

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Ennemi {
  const Ennemi({
    required this.nom,
    required this.type,
    required this.pointsDeVie,
    required this.degats,
  });

  final String nom;
  final String type;
  final int pointsDeVie;
  final int degats;
}

const List<Ennemi> bestiaire = [
  Ennemi(nom: 'Gobelin', type: 'Humanoïde', pointsDeVie: 45, degats: 8),
  Ennemi(nom: 'Loup des glaces', type: 'Bête', pointsDeVie: 80, degats: 14),
  Ennemi(nom: 'Golem de pierre', type: 'Élémentaire', pointsDeVie: 260, degats: 22),
  Ennemi(nom: 'Liche', type: 'Mort-vivant', pointsDeVie: 340, degats: 41),
];

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bestiaire',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      home: const EcranBestiaire(),
    );
  }
}

class EcranBestiaire extends StatelessWidget {
  const EcranBestiaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bestiaire')),
      body: ListView.separated(
        itemCount: bestiaire.length,
        separatorBuilder: (context, index) => const Divider(height: 1),
        itemBuilder: (context, index) {
          final ennemi = bestiaire[index];
          return ListTile(
            leading: const Icon(Icons.pest_control),
            title: Text(ennemi.nom),
            subtitle: Text(ennemi.type),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => EcranFicheEnnemi(ennemi: ennemi),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class EcranFicheEnnemi extends StatelessWidget {
  const EcranFicheEnnemi({super.key, required this.ennemi});

  final Ennemi ennemi;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(ennemi.nom)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          ListTile(
            leading: const Icon(Icons.badge),
            title: const Text('Nom'),
            trailing: Text(ennemi.nom),
          ),
          ListTile(
            leading: const Icon(Icons.category),
            title: const Text('Type'),
            trailing: Text(ennemi.type),
          ),
          ListTile(
            leading: const Icon(Icons.favorite),
            title: const Text('Points de vie'),
            trailing: Text('${ennemi.pointsDeVie}'),
          ),
          ListTile(
            leading: const Icon(Icons.whatshot),
            title: const Text('Dégâts'),
            trailing: Text('${ennemi.degats}'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** `EcranFicheEnnemi` déclare `required this.ennemi` dans son constructeur. Le compilateur refuse alors toute navigation qui oublierait la donnée, et le type est connu partout : aucun cast, aucune vérification à l'exécution. C'est la méthode recommandée de la section 50.7. La liste est un `ListView.separated` du chapitre 48 : chaque `itemBuilder` capture son propre `ennemi`, qui est passé tel quel au `MaterialPageRoute`.

---

### Correction 3

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
      title: 'Armurerie',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
        useMaterial3: true,
      ),
      home: const EcranPrincipal(),
    );
  }
}

class EcranPrincipal extends StatefulWidget {
  const EcranPrincipal({super.key});

  @override
  State<EcranPrincipal> createState() => _EcranPrincipalState();
}

class _EcranPrincipalState extends State<EcranPrincipal> {
  String? _arme;

  Future<void> _choisirArme() async {
    final String? choix = await Navigator.push<String>(
      context,
      MaterialPageRoute(builder: (context) => const EcranArmurerie()),
    );

    // Contrôle obligatoire après un await (section 50.8.1).
    if (!mounted) return;

    // L'utilisateur est revenu par la flèche : on ne change rien.
    if (choix == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Aucun changement.')),
      );
      return;
    }

    setState(() => _arme = choix);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Équipement')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.person, size: 96),
            const SizedBox(height: 16),
            Text(
              'Arme équipée : ${_arme ?? 'aucune'}',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 32),
            FilledButton.icon(
              onPressed: _choisirArme,
              icon: const Icon(Icons.hardware),
              label: const Text('Choisir une arme'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranArmurerie extends StatelessWidget {
  const EcranArmurerie({super.key});

  static const List<String> armes = [
    'Épée courte',
    'Hache de guerre',
    'Arc long',
    'Bâton de mage',
    'Dague empoisonnée',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Armurerie')),
      body: ListView.builder(
        itemCount: armes.length,
        itemBuilder: (context, index) {
          final arme = armes[index];
          return ListTile(
            leading: const Icon(Icons.hardware),
            title: Text(arme),
            onTap: () => Navigator.pop(context, arme),
          );
        },
      ),
    );
  }
}
```

**Explication :** `Navigator.push<String>` annonce que la route renverra une `String`, donc `choix` est de type `String?`. Le `?` est incontournable (section 50.10.2) : la flèche de retour appelle `pop` sans valeur, et le `Future` se complète alors avec `null`. Le code distingue explicitement les deux cas — une valeur reçue provoque un `setState`, un `null` affiche simplement une `SnackBar`. Le `if (!mounted) return;` précède toute utilisation de `context`, conformément à la règle de la section 50.8.1.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Routes {
  const Routes._();

  static const String menu = '/';
  static const String inventaire = '/inventaire';
  static const String carte = '/carte';
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Routes nommées',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      initialRoute: Routes.menu,
      routes: {
        Routes.menu: (context) => const EcranMenu(),
        Routes.inventaire: (context) => const EcranInventaire(),
        Routes.carte: (context) => const EcranCarte(),
      },
      onUnknownRoute: (settings) => MaterialPageRoute(
        settings: settings,
        builder: (context) => EcranIntrouvable(nom: settings.name),
      ),
    );
  }
}

class EcranMenu extends StatelessWidget {
  const EcranMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FilledButton.icon(
              onPressed: () =>
                  Navigator.pushNamed(context, Routes.inventaire),
              icon: const Icon(Icons.backpack),
              label: const Text('Inventaire'),
            ),
            const SizedBox(height: 12),
            FilledButton.icon(
              onPressed: () => Navigator.pushNamed(context, Routes.carte),
              icon: const Icon(Icons.map),
              label: const Text('Carte'),
            ),
            const SizedBox(height: 24),
            OutlinedButton(
              onPressed: () =>
                  Navigator.pushNamed(context, '/donjon-secret'),
              child: const Text('Ouvrir /donjon-secret'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranInventaire extends StatelessWidget {
  const EcranInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventaire')),
      body: ListView(
        children: const [
          ListTile(leading: Icon(Icons.local_drink), title: Text('Potion x3')),
          ListTile(leading: Icon(Icons.hardware), title: Text('Épée courte')),
          ListTile(leading: Icon(Icons.key), title: Text('Clé rouillée')),
        ],
      ),
    );
  }
}

class EcranCarte extends StatelessWidget {
  const EcranCarte({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Carte')),
      body: const Center(child: Text('Trois régions découvertes sur douze.')),
    );
  }
}

class EcranIntrouvable extends StatelessWidget {
  const EcranIntrouvable({super.key, this.nom});

  final String? nom;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Page introuvable')),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                Icons.explore_off,
                size: 96,
                color: Theme.of(context).colorScheme.error,
              ),
              const SizedBox(height: 16),
              Text('404', style: Theme.of(context).textTheme.displaySmall),
              const SizedBox(height: 8),
              Text(
                'La route « ${nom ?? 'inconnue'} » n\'existe pas.',
                textAlign: TextAlign.center,
              ),
              const SizedBox(height: 24),
              FilledButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('Revenir'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** la classe `Routes` a un constructeur privé `const Routes._()` pour empêcher toute instanciation : elle ne sert que de porte-noms. Grâce à elle, une faute de frappe sur `Routes.inventaire` devient une erreur de compilation, alors que `'/inventaire'` écrit à la main ne serait détecté qu'au clic (section 50.12). Le bouton piégé utilise volontairement une chaîne littérale absente de la table : Flutter ne trouve rien dans `routes`, il n'y a pas d'`onGenerateRoute`, donc `onUnknownRoute` est appelé et rend la page 404. Le `settings: settings` transmis conserve le nom demandé, que la page affiche.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Routes {
  const Routes._();

  static const String bestiaire = '/';
  static const String ennemi = '/ennemi';
}

class Ennemi {
  const Ennemi({
    required this.nom,
    required this.type,
    required this.pointsDeVie,
    required this.degats,
  });

  final String nom;
  final String type;
  final int pointsDeVie;
  final int degats;
}

const List<Ennemi> bestiaire = [
  Ennemi(nom: 'Gobelin', type: 'Humanoïde', pointsDeVie: 45, degats: 8),
  Ennemi(nom: 'Loup des glaces', type: 'Bête', pointsDeVie: 80, degats: 14),
  Ennemi(nom: 'Golem de pierre', type: 'Élémentaire', pointsDeVie: 260, degats: 22),
  Ennemi(nom: 'Liche', type: 'Mort-vivant', pointsDeVie: 340, degats: 41),
];

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      initialRoute: Routes.bestiaire,
      routes: {
        Routes.bestiaire: (context) => const EcranBestiaire(),
        Routes.ennemi: (context) => const EcranFicheEnnemi(),
      },
    );
  }
}

class EcranBestiaire extends StatelessWidget {
  const EcranBestiaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bestiaire')),
      body: ListView(
        children: [
          for (final ennemi in bestiaire)
            ListTile(
              leading: const Icon(Icons.pest_control),
              title: Text(ennemi.nom),
              subtitle: Text(ennemi.type),
              onTap: () => Navigator.pushNamed(
                context,
                Routes.ennemi,
                arguments: ennemi,
              ),
            ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.warning_amber),
            title: const Text('Ouvrir /ennemi SANS argument'),
            onTap: () => Navigator.pushNamed(context, Routes.ennemi),
          ),
          ListTile(
            leading: const Icon(Icons.warning_amber),
            title: const Text('Ouvrir /ennemi avec un mauvais type'),
            onTap: () => Navigator.pushNamed(
              context,
              Routes.ennemi,
              arguments: 'ceci est une chaîne',
            ),
          ),
        ],
      ),
    );
  }
}

class EcranFicheEnnemi extends StatelessWidget {
  const EcranFicheEnnemi({super.key});

  @override
  Widget build(BuildContext context) {
    final argument = ModalRoute.of(context)?.settings.arguments;

    if (argument is! Ennemi) {
      return Scaffold(
        appBar: AppBar(title: const Text('Fiche indisponible')),
        body: Center(
          child: Padding(
            padding: const EdgeInsets.all(32),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.help_outline, size: 72),
                const SizedBox(height: 16),
                Text(
                  'Cette page attend un objet Ennemi.\n'
                  'Reçu : ${argument.runtimeType}.',
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 24),
                FilledButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('Revenir au bestiaire'),
                ),
              ],
            ),
          ),
        ),
      );
    }

    // Ici, Dart a promu `argument` en Ennemi.
    final ennemi = argument;

    return Scaffold(
      appBar: AppBar(title: Text(ennemi.nom)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          ListTile(title: const Text('Type'), trailing: Text(ennemi.type)),
          ListTile(
            title: const Text('Points de vie'),
            trailing: Text('${ennemi.pointsDeVie}'),
          ),
          ListTile(
            title: const Text('Dégâts'),
            trailing: Text('${ennemi.degats}'),
          ),
        ],
      ),
    );
  }
}
```

**Explication :** l'écran ne fait **jamais** `arguments as Ennemi`. Il stocke la valeur, teste `is! Ennemi` et rend un écran de repli. Ce seul test couvre les trois cas défaillants : argument absent (`Null`), argument d'un autre type (`String`), et route ouverte directement par un lien profond. Après le `return`, la promotion de type de Dart rend `argument` utilisable comme un `Ennemi` sans cast (section 50.13.1). Comparez avec la correction 2 : le constructeur donnait cette garantie gratuitement, alors qu'ici il faut la reconstruire à la main. C'est exactement pourquoi la documentation déconseille les routes nommées.

---
### Correction 6

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
      title: 'Cycle de partie',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranChargement(),
    );
  }
}

/// Affiche l'état de la pile sur chaque écran.
class IndicateurPile extends StatelessWidget {
  const IndicateurPile({super.key});

  @override
  Widget build(BuildContext context) {
    return Text(
      'canPop : ${Navigator.canPop(context)}',
      style: Theme.of(context).textTheme.bodySmall,
    );
  }
}

class EcranChargement extends StatefulWidget {
  const EcranChargement({super.key});

  @override
  State<EcranChargement> createState() => _EcranChargementState();
}

class _EcranChargementState extends State<EcranChargement> {
  @override
  void initState() {
    super.initState();
    _demarrer();
  }

  Future<void> _demarrer() async {
    await Future<void>.delayed(const Duration(seconds: 2));
    if (!mounted) return;

    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => const EcranMenu()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.primaryContainer,
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.shield_moon, size: 96),
            SizedBox(height: 24),
            Text('Chargement des sauvegardes...'),
            SizedBox(height: 24),
            CircularProgressIndicator(),
          ],
        ),
      ),
    );
  }
}

class EcranMenu extends StatelessWidget {
  const EcranMenu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Menu principal')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const IndicateurPile(),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => const EcranCombat(numero: 1),
                ),
              ),
              child: const Text('Commencer la campagne'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranCombat extends StatelessWidget {
  const EcranCombat({super.key, required this.numero});

  final int numero;

  @override
  Widget build(BuildContext context) {
    final dernier = numero == 2;

    return Scaffold(
      appBar: AppBar(title: Text('Combat $numero')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const IndicateurPile(),
            const SizedBox(height: 16),
            Text('Ennemi n° $numero vaincu.'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => dernier
                      ? const EcranVictoire()
                      : const EcranCombat(numero: 2),
                ),
              ),
              child: Text(dernier ? 'Terminer la campagne' : 'Combat suivant'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranVictoire extends StatelessWidget {
  const EcranVictoire({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Victoire'),
        automaticallyImplyLeading: false,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const IndicateurPile(),
            const SizedBox(height: 16),
            const Icon(Icons.emoji_events, size: 96, color: Colors.amber),
            const SizedBox(height: 16),
            Text(
              'Campagne terminée',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 32),
            FilledButton(
              onPressed: () {
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(builder: (context) => const EcranMenu()),
                  (route) => false,
                );
              },
              child: const Text('Retour au menu'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** trois mécanismes se répondent. `pushReplacement` retire l'écran de chargement en même temps qu'il empile le menu : la hauteur reste à 1, donc `canPop` affiche `false` sur le menu et le bouton système quitte l'application. Les deux combats et la victoire sont empilés normalement (hauteur 4, `canPop` vaut `true`). Enfin `pushAndRemoveUntil` avec le prédicat `(route) => false` ne trouve jamais de route à conserver : il vide toute la pile et pose le menu au fond. On revient donc à `canPop : false`. Le `if (!mounted) return;` après le délai de deux secondes protège le cas où l'utilisateur ferme l'application pendant le chargement.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Personnage {
  const Personnage({required this.nom, required this.force});

  final String nom;
  final int force;
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranAccueil(),
    );
  }
}

class EcranAccueil extends StatefulWidget {
  const EcranAccueil({super.key});

  @override
  State<EcranAccueil> createState() => _EcranAccueilState();
}

class _EcranAccueilState extends State<EcranAccueil> {
  Personnage? _personnage;

  Future<void> _creer() async {
    final Personnage? cree = await Navigator.push<Personnage>(
      context,
      MaterialPageRoute(
        fullscreenDialog: true,
        builder: (context) => const EcranCreation(),
      ),
    );
    if (!mounted || cree == null) return;
    setState(() => _personnage = cree);
  }

  @override
  Widget build(BuildContext context) {
    final p = _personnage;

    return Scaffold(
      appBar: AppBar(title: const Text('Personnage')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (p == null)
              const Text('Aucun personnage créé.')
            else ...[
              CircleAvatar(radius: 40, child: Text(p.nom[0])),
              const SizedBox(height: 12),
              Text(p.nom, style: Theme.of(context).textTheme.headlineSmall),
              Text('Force : ${p.force}'),
            ],
            const SizedBox(height: 32),
            FilledButton(
              onPressed: _creer,
              child: const Text('Créer un personnage'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranCreation extends StatefulWidget {
  const EcranCreation({super.key});

  @override
  State<EcranCreation> createState() => _EcranCreationState();
}

class _EcranCreationState extends State<EcranCreation> {
  static const int forceInitiale = 10;

  final TextEditingController _nom = TextEditingController();
  double _force = forceInitiale.toDouble();

  bool get _modifie =>
      _nom.text.trim().isNotEmpty || _force.round() != forceInitiale;

  @override
  void initState() {
    super.initState();
    _nom.addListener(() => setState(() {}));
  }

  @override
  void dispose() {
    _nom.dispose();
    super.dispose();
  }

  Future<bool> _confirmerAbandon() async {
    final reponse = await showDialog<bool>(
      context: context,
      builder: (dialogContext) => AlertDialog(
        title: const Text('Abandonner la création ?'),
        content: const Text('Le personnage ne sera pas enregistré.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(dialogContext, false),
            child: const Text('Continuer'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(dialogContext, true),
            child: const Text('Abandonner'),
          ),
        ],
      ),
    );
    return reponse ?? false;
  }

  void _creer() {
    final nom = _nom.text.trim();
    if (nom.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Le nom est obligatoire.')),
      );
      return;
    }
    Navigator.pop(context, Personnage(nom: nom, force: _force.round()));
  }

  @override
  Widget build(BuildContext context) {
    return PopScope<Personnage>(
      canPop: !_modifie,
      onPopInvokedWithResult: (didPop, result) async {
        if (didPop) return;

        final quitter = await _confirmerAbandon();
        if (!context.mounted) return;
        if (quitter) Navigator.pop(context);
      },
      child: Scaffold(
        appBar: AppBar(title: const Text('Nouveau personnage')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextField(
                controller: _nom,
                decoration: const InputDecoration(
                  labelText: 'Nom',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 24),
              Text('Force : ${_force.round()}'),
              Slider(
                value: _force,
                min: 1,
                max: 20,
                divisions: 19,
                label: '${_force.round()}',
                onChanged: (v) => setState(() => _force = v),
              ),
              const SizedBox(height: 16),
              Text(
                _modifie
                    ? 'Modifications en cours : le retour demandera confirmation.'
                    : 'Rien de saisi : le retour est libre.',
                style: Theme.of(context).textTheme.bodySmall,
              ),
              const Spacer(),
              FilledButton(onPressed: _creer, child: const Text('Créer')),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** `canPop: !_modifie` est recalculé à chaque `build`. Tant que rien n'a bougé, Flutter laisse le retour se faire normalement et `onPopInvokedWithResult` reçoit `didPop == true` : le `if (didPop) return;` sort immédiatement. Dès qu'un caractère est tapé ou que le curseur bouge, `canPop` devient `false` : le retour est intercepté, `didPop` vaut `false`, et on affiche la boîte. Si l'utilisateur confirme, on appelle nous-mêmes `Navigator.pop(context)` — sans valeur, donc l'écran appelant reçoit `null` et ne modifie rien. Notez le `dialogContext` dans le `showDialog` : c'est lui qui ferme la boîte, pas l'écran (piège de la section 50.22.1).

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApp());
}

class Objet {
  const Objet({
    required this.id,
    required this.nom,
    required this.categorie,
    required this.couleur,
    required this.icone,
  });

  final String id;
  final String nom;
  final String categorie;
  final Color couleur;
  final IconData icone;
}

const List<Objet> catalogue = [
  Objet(id: 'a1', nom: 'Épée courte', categorie: 'Armes', couleur: Colors.blueGrey, icone: Icons.hardware),
  Objet(id: 'a2', nom: 'Arc long', categorie: 'Armes', couleur: Colors.green, icone: Icons.gps_fixed),
  Objet(id: 'a3', nom: 'Bâton runique', categorie: 'Armes', couleur: Colors.purple, icone: Icons.auto_awesome),
  Objet(id: 'b1', nom: 'Cotte de mailles', categorie: 'Armures', couleur: Colors.brown, icone: Icons.shield),
  Objet(id: 'b2', nom: 'Robe de mage', categorie: 'Armures', couleur: Colors.indigo, icone: Icons.checkroom),
  Objet(id: 'c1', nom: 'Potion de soin', categorie: 'Potions', couleur: Colors.red, icone: Icons.local_drink),
  Objet(id: 'c2', nom: 'Élixir de force', categorie: 'Potions', couleur: Colors.orange, icone: Icons.science),
];

/// Transition en fondu réutilisable (section 50.24).
Route<void> routeFondu(Widget page) {
  return PageRouteBuilder<void>(
    transitionDuration: const Duration(milliseconds: 350),
    reverseTransitionDuration: const Duration(milliseconds: 350),
    pageBuilder: (context, animation, secondaryAnimation) => page,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  );
}

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const EcranCatalogue(),
    );
  }
}

class EcranCatalogue extends StatelessWidget {
  const EcranCatalogue({super.key});

  static const List<String> categories = ['Armes', 'Armures', 'Potions'];

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: categories.length,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Catalogue'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.hardware), text: 'Armes'),
              Tab(icon: Icon(Icons.shield), text: 'Armures'),
              Tab(icon: Icon(Icons.local_drink), text: 'Potions'),
            ],
          ),
        ),
        body: TabBarView(
          children: [
            for (final categorie in categories)
              GrilleObjets(categorie: categorie),
          ],
        ),
      ),
    );
  }
}

class GrilleObjets extends StatelessWidget {
  const GrilleObjets({super.key, required this.categorie});

  final String categorie;

  @override
  Widget build(BuildContext context) {
    final objets =
        catalogue.where((o) => o.categorie == categorie).toList();

    return GridView.count(
      crossAxisCount: 2,
      padding: const EdgeInsets.all(16),
      mainAxisSpacing: 16,
      crossAxisSpacing: 16,
      children: [
        for (final objet in objets)
          InkWell(
            onTap: () => Navigator.push(
              context,
              routeFondu(EcranObjet(objet: objet)),
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Hero(
                  tag: 'objet-${objet.id}',
                  child: CircleAvatar(
                    radius: 36,
                    backgroundColor: objet.couleur,
                    child: Icon(objet.icone, color: Colors.white, size: 32),
                  ),
                ),
                const SizedBox(height: 8),
                Text(objet.nom, textAlign: TextAlign.center),
              ],
            ),
          ),
      ],
    );
  }
}

class EcranObjet extends StatelessWidget {
  const EcranObjet({super.key, required this.objet});

  final Objet objet;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(objet.nom)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: 'objet-${objet.id}',
              child: CircleAvatar(
                radius: 90,
                backgroundColor: objet.couleur,
                child: Icon(objet.icone, color: Colors.white, size: 80),
              ),
            ),
            const SizedBox(height: 24),
            Text(objet.nom, style: Theme.of(context).textTheme.headlineMedium),
            Text('Catégorie : ${objet.categorie}'),
            Text('Identifiant : ${objet.id}'),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** le `tag` du `Hero` est `'objet-${objet.id}'`. Comme les identifiants sont uniques dans tout le catalogue, deux `Hero` ne peuvent jamais partager un tag, y compris entre deux onglets — c'est important, car `TabBarView` construit l'onglet voisin pendant le glissement. `DefaultTabController(length: 3)` correspond exactement aux trois `Tab` et aux trois enfants du `TabBarView` : toute divergence lèverait l'assertion vue en 50.25.1. La transition en fondu est extraite dans la fonction `routeFondu`, ce qui la rend réutilisable et laisse le code d'appel lisible. Notez que l'animation `Hero` fonctionne parfaitement avec une transition personnalisée : les deux mécanismes sont indépendants.

---
### Correction 9

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
      title: 'Coque complète',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const Coque(),
    );
  }
}

class Coque extends StatefulWidget {
  const Coque({super.key});

  @override
  State<Coque> createState() => _CoqueState();
}

class _CoqueState extends State<Coque> {
  int _index = 0;
  bool _modeDifficile = false;

  static const List<String> _titres = ['Aventure', 'Journal', 'Réglages'];

  void _changerSection(int index) {
    setState(() => _index = index);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_titres[_index])),

      // ----- tiroir de gauche : navigation
      drawer: NavigationDrawer(
        selectedIndex: _index,
        onDestinationSelected: (index) {
          _changerSection(index);
          Navigator.pop(context); // referme le tiroir AVANT toute autre action
        },
        children: [
          Padding(
            padding: const EdgeInsets.fromLTRB(28, 24, 16, 12),
            child: Text(
              'Chroniques de la guilde',
              style: Theme.of(context).textTheme.titleMedium,
            ),
          ),
          const Divider(),
          const NavigationDrawerDestination(
            icon: Icon(Icons.explore_outlined),
            selectedIcon: Icon(Icons.explore),
            label: Text('Aventure'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.menu_book_outlined),
            selectedIcon: Icon(Icons.menu_book),
            label: Text('Journal'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: Text('Réglages'),
          ),
        ],
      ),

      // ----- tiroir de droite : réglage rapide
      endDrawer: Drawer(
        child: SafeArea(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text(
                  'Options rapides',
                  style: Theme.of(context).textTheme.titleLarge,
                ),
              ),
              const Divider(height: 1),
              SwitchListTile(
                title: const Text('Mode difficile'),
                subtitle: const Text('Les ennemis frappent deux fois plus fort.'),
                value: _modeDifficile,
                onChanged: (v) => setState(() => _modeDifficile = v),
              ),
              const Spacer(),
              Padding(
                padding: const EdgeInsets.all(16),
                child: SizedBox(
                  width: double.infinity,
                  child: FilledButton(
                    onPressed: () => Navigator.pop(context),
                    child: const Text('Fermer'),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),

      // ----- corps : IndexedStack, donc état conservé
      body: IndexedStack(
        index: _index,
        children: [
          const SectionAventure(),
          const SectionJournal(),
          SectionReglages(modeDifficile: _modeDifficile),
        ],
      ),

      floatingActionButton: Builder(
        builder: (context) => FloatingActionButton(
          tooltip: 'Options rapides',
          onPressed: () => Scaffold.of(context).openEndDrawer(),
          child: const Icon(Icons.tune),
        ),
      ),

      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: _changerSection,
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.explore_outlined),
            selectedIcon: Icon(Icons.explore),
            label: 'Aventure',
          ),
          NavigationDestination(
            icon: Icon(Icons.menu_book_outlined),
            selectedIcon: Icon(Icons.menu_book),
            label: 'Journal',
          ),
          NavigationDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: 'Réglages',
          ),
        ],
      ),
    );
  }
}

class SectionAventure extends StatefulWidget {
  const SectionAventure({super.key});

  @override
  State<SectionAventure> createState() => _SectionAventureState();
}

class _SectionAventureState extends State<SectionAventure> {
  int _ennemisVaincus = 0;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.dangerous, size: 72),
          const SizedBox(height: 16),
          Text(
            'Ennemis vaincus : $_ennemisVaincus',
            style: Theme.of(context).textTheme.headlineSmall,
          ),
          const SizedBox(height: 24),
          FilledButton(
            onPressed: () => setState(() => _ennemisVaincus++),
            child: const Text('Combattre'),
          ),
          const SizedBox(height: 16),
          Text(
            'Changez de section et revenez :\nle compteur est conservé.',
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.bodySmall,
          ),
        ],
      ),
    );
  }
}

class SectionJournal extends StatelessWidget {
  const SectionJournal({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView(
      children: const [
        ListTile(
          leading: Icon(Icons.circle, size: 12),
          title: Text('Jour 1 — arrivée à la citadelle'),
        ),
        ListTile(
          leading: Icon(Icons.circle, size: 12),
          title: Text('Jour 4 — rencontre avec la guilde'),
        ),
        ListTile(
          leading: Icon(Icons.circle, size: 12),
          title: Text('Jour 9 — descente dans les catacombes'),
        ),
      ],
    );
  }
}

class SectionReglages extends StatelessWidget {
  const SectionReglages({super.key, required this.modeDifficile});

  final bool modeDifficile;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            modeDifficile ? Icons.local_fire_department : Icons.spa,
            size: 72,
            color: modeDifficile
                ? Theme.of(context).colorScheme.error
                : Theme.of(context).colorScheme.primary,
          ),
          const SizedBox(height: 16),
          Text(
            modeDifficile ? 'Mode difficile activé' : 'Mode normal',
            style: Theme.of(context).textTheme.headlineSmall,
          ),
          const SizedBox(height: 8),
          const Text('Réglable depuis le tiroir de droite.'),
        ],
      ),
    );
  }
}
```

**Explication :** un seul `State` détient `_index` et `_modeDifficile` ; tout le reste en découle. `NavigationBar` et `NavigationDrawer` appellent la même méthode `_changerSection`, ce qui garantit que les deux restent synchronisés — c'est pour cela que le tiroir affiche `selectedIndex: _index`. Le `Navigator.pop(context)` du tiroir vient **après** le changement de section mais avant que quoi que ce soit d'autre ne se produise : un tiroir est une route empilée, il faut la dépiler (section 50.28.1). Le `Builder` autour du `FloatingActionButton` est indispensable : `Scaffold.of(context)` a besoin d'un `context` situé sous le `Scaffold`, or le `context` du `build` de `Coque` est celui qui **crée** ce `Scaffold` (même piège qu'en 50.3.2). Enfin, `IndexedStack` construit les trois sections en permanence : le compteur d'ennemis de `SectionAventure` survit à tous les changements d'onglet (section 50.27).

---

### Correction 10

Ajoutez d'abord la dépendance :

```text
flutter pub add go_router
```

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MonApp());
}

// ---------------------------------------------------------------- MODÈLE

class Membre {
  const Membre({
    required this.id,
    required this.nom,
    required this.classe,
    required this.niveau,
  });

  final int id;
  final String nom;
  final String classe;
  final int niveau;
}

const List<Membre> guilde = [
  Membre(id: 1, nom: 'Aria', classe: 'Archer', niveau: 42),
  Membre(id: 2, nom: 'Borin', classe: 'Guerrier', niveau: 37),
  Membre(id: 3, nom: 'Célia', classe: 'Mage', niveau: 45),
  Membre(id: 4, nom: 'Dorn', classe: 'Voleur', niveau: 29),
];

/// Recharge un membre à partir de son seul identifiant.
/// C'est la contrainte imposée par les URL : pas d'objet en `extra`.
Membre? chercherMembre(int? id) {
  for (final membre in guilde) {
    if (membre.id == id) return membre;
  }
  return null;
}

// ------------------------------------------------------------- SESSION

class Session extends ChangeNotifier {
  bool _connecte = false;
  String _pseudo = '';

  bool get connecte => _connecte;
  String get pseudo => _pseudo;

  void connecter(String pseudo) {
    _connecte = true;
    _pseudo = pseudo;
    notifyListeners();
  }

  void deconnecter() {
    _connecte = false;
    _pseudo = '';
    notifyListeners();
  }
}

final Session session = Session();

// -------------------------------------------------------------- ROUTEUR

final GoRouter routeur = GoRouter(
  initialLocation: '/guilde',
  refreshListenable: session,
  debugLogDiagnostics: true,
  redirect: (context, state) {
    final surConnexion = state.matchedLocation == '/connexion';

    if (!session.connecte && !surConnexion) return '/connexion';
    if (session.connecte && surConnexion) return '/guilde';
    return null;
  },
  routes: [
    GoRoute(
      path: '/connexion',
      builder: (context, state) => const EcranConnexion(),
    ),
    GoRoute(
      path: '/guilde',
      builder: (context, state) => const EcranGuilde(),
      routes: [
        // Sous-route : PAS de '/' au début.
        GoRoute(
          path: 'membre/:id',
          builder: (context, state) {
            final id = int.tryParse(state.pathParameters['id'] ?? '');
            final membre = chercherMembre(id);

            if (membre == null) {
              return EcranErreur(message: 'Aucun membre d\'identifiant $id.');
            }
            return EcranMembre(membre: membre);
          },
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) =>
      EcranErreur(message: 'Adresse inconnue : ${state.uri}'),
);

// ---------------------------------------------------------- APPLICATION

class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Guilde',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: routeur,
    );
  }
}

// -------------------------------------------------------------- ÉCRANS

class EcranConnexion extends StatefulWidget {
  const EcranConnexion({super.key});

  @override
  State<EcranConnexion> createState() => _EcranConnexionState();
}

class _EcranConnexionState extends State<EcranConnexion> {
  final TextEditingController _pseudo = TextEditingController();

  @override
  void dispose() {
    _pseudo.dispose();
    super.dispose();
  }

  void _entrer() {
    final pseudo = _pseudo.text.trim();
    if (pseudo.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Entrez un pseudonyme.')),
      );
      return;
    }
    // La redirection fera le reste, grâce à refreshListenable.
    session.connecter(pseudo);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Connexion')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.lock_outline, size: 72),
            const SizedBox(height: 24),
            TextField(
              controller: _pseudo,
              decoration: const InputDecoration(
                labelText: 'Pseudonyme',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) => _entrer(),
            ),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: _entrer,
              child: const Text('Entrer dans la guilde'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranGuilde extends StatelessWidget {
  const EcranGuilde({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Guilde — ${session.pseudo}'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            tooltip: 'Se déconnecter',
            onPressed: session.deconnecter,
          ),
        ],
      ),
      body: ListView(
        children: [
          for (final membre in guilde)
            ListTile(
              leading: CircleAvatar(child: Text('${membre.id}')),
              title: Text(membre.nom),
              subtitle: Text('${membre.classe} — niveau ${membre.niveau}'),
              trailing: const Icon(Icons.chevron_right),
              // push : on empile le détail par-dessus la liste.
              onTap: () => context.push('/guilde/membre/${membre.id}'),
            ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.warning_amber),
            title: const Text('Ouvrir /guilde/membre/999'),
            onTap: () => context.push('/guilde/membre/999'),
          ),
          ListTile(
            leading: const Icon(Icons.warning_amber),
            title: const Text('Ouvrir /adresse-inconnue'),
            onTap: () => context.push('/adresse-inconnue'),
          ),
        ],
      ),
    );
  }
}

class EcranMembre extends StatelessWidget {
  const EcranMembre({super.key, required this.membre});

  final Membre membre;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(membre.nom)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 56,
              child: Text(
                membre.nom[0],
                style: const TextStyle(fontSize: 40),
              ),
            ),
            const SizedBox(height: 24),
            Text(membre.nom, style: Theme.of(context).textTheme.headlineMedium),
            Text(membre.classe),
            const SizedBox(height: 16),
            Text(
              'Niveau ${membre.niveau}',
              style: Theme.of(context).textTheme.displaySmall,
            ),
            const SizedBox(height: 24),
            Text('URL : /guilde/membre/${membre.id}'),
            const SizedBox(height: 24),
            OutlinedButton(
              onPressed: () => context.go('/guilde'),
              child: const Text('Retour à la liste'),
            ),
          ],
        ),
      ),
    );
  }
}

class EcranErreur extends StatelessWidget {
  const EcranErreur({super.key, required this.message});

  final String message;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Erreur')),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                Icons.search_off,
                size: 96,
                color: Theme.of(context).colorScheme.error,
              ),
              const SizedBox(height: 16),
              Text(message, textAlign: TextAlign.center),
              const SizedBox(height: 24),
              FilledButton(
                onPressed: () => context.go('/guilde'),
                child: const Text('Revenir à la guilde'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Explication :** cinq mécanismes de `go_router` se combinent ici.

1. **`MaterialApp.router` + `routerConfig`** remplacent `MaterialApp` + `home`. C'est le seul changement structurel exigé par le paquet.
2. **La redirection globale** est le point unique de contrôle d'accès. Ses deux tests sont symétriques : rediriger vers `/connexion` quand on n'est pas connecté, et **en sortir** quand on l'est. Sans le second test — ou sans le garde `surConnexion` du premier — on obtiendrait `GoException: too many redirects`.
3. **`refreshListenable: session`** fait réévaluer la redirection à chaque `notifyListeners()`. C'est ce qui rend la déconnexion instantanée : aucun écran n'a besoin de savoir qu'il doit se fermer.
4. **La sous-route `'membre/:id'`** ne commence pas par `/`. Son chemin complet est donc `/guilde/membre/3`, et `go_router` empile automatiquement `EcranGuilde` sous `EcranMembre` : même en arrivant par un lien profond ou une URL tapée à la main, la flèche de retour ramène à la liste.
5. **`errorBuilder`** attrape les adresses hors de l'arbre (`/adresse-inconnue`), tandis qu'un identifiant syntaxiquement valide mais introuvable (`999`) est traité dans le `builder` lui-même, puisque la route existe bien : la distinction est importante.

Enfin, remarquez que `EcranMembre` reçoit un `Membre` reconstruit par `chercherMembre(id)` à partir du seul paramètre de chemin. C'est la contrainte imposée par le Web (section 50.36) : après un rechargement de page, `state.extra` serait `null`, mais l'URL contient toujours l'identifiant.

---

## Et maintenant ?

Vous savez désormais faire circuler l'utilisateur dans votre application : empiler un détail, revenir avec une valeur, protéger un formulaire, organiser des onglets et un tiroir, et passer à `go_router` quand l'application le mérite.

Vos écrans se comportent bien. En revanche, ils se ressemblent tous : la même couleur de départ, la même taille de texte, la même mise en page quelle que soit la fenêtre. Une application vue sur un téléphone en mode sombre, sur une tablette en paysage et dans une fenêtre de bureau redimensionnée pose des problèmes que ni les widgets ni la navigation ne résolvent.

C'est l'objet du chapitre suivant : construire un thème cohérent avec `ThemeData` et Material 3, gérer le mode sombre, et adapter la mise en page à toutes les tailles d'écran avec `MediaQuery` et `LayoutBuilder`.

Rendez-vous au chapitre 51 : [51-PARTIE-1B—THÈMES-STYLES-ET-RESPONSIVE.md](./51-PARTIE-1B—THÈMES-STYLES-ET-RESPONSIVE.md)
