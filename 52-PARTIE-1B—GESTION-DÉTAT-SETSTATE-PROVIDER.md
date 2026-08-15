# PARTIE 1B — FLUTTER
# CHAPITRE 52 — LA GESTION D'ÉTAT : DE SETSTATE À PROVIDER

> **Niveau :** intermédiaire
> **Durée estimée :** 10 h
> **Pré-requis :** chapitres 43 à 51 (installation, widgets, `setState`, layouts, texte, listes, formulaires, navigation, thèmes) ; chapitres 08 à 11, 14, 15 et 16 de la PARTIE 1A (POO, collections fonctionnelles, asynchrone, organisation d'un projet)
> **Ce que vous saurez faire à la fin :** distinguer l'état éphémère de l'état d'application, comprendre le mécanisme `InheritedWidget` sur lequel repose tout Flutter, écrire une classe d'état avec `ChangeNotifier`, la partager entre plusieurs écrans avec le paquet `provider`, et choisir en connaissance de cause entre les grandes solutions du marché.

---

## 52.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- définir précisément ce qu'on appelle « gestion d'état » ;
- distinguer l'**état éphémère** de l'**état d'application** ;
- choisir l'outil adapté à chaque type d'état ;
- énoncer les trois limites concrètes de `setState()` ;
- reconnaître le **forage de propriétés** (*prop drilling*) et le mesurer ;
- expliquer pourquoi une cascade de callbacks devient ingérable ;
- décrire le rôle d'un `InheritedWidget` dans l'arbre ;
- écrire un `InheritedWidget` complet, avec `of()` et `maybeOf()` ;
- expliquer ce que fait `dependOnInheritedWidgetOfExactType` ;
- écrire un `updateShouldNotify` correct et en mesurer l'effet ;
- expliquer pourquoi `Theme.of(context)` fonctionne ;
- énoncer les limites d'un `InheritedWidget` écrit à la main ;
- écrire une classe qui étend `ChangeNotifier` ;
- appeler `notifyListeners()` au bon moment ;
- utiliser `ValueNotifier` et `ValueListenableBuilder` sans aucune dépendance ;
- utiliser `ListenableBuilder` avec n'importe quel `Listenable` ;
- installer et configurer le paquet `provider` ;
- utiliser `ChangeNotifierProvider` pour exposer un objet d'état ;
- utiliser `context.watch<T>()`, `context.read<T>()` et `context.select<T, R>()` ;
- limiter la portée des reconstructions avec `Consumer<T>` et `Selector` ;
- appliquer la règle « `watch` dans `build`, `read` dans les callbacks » ;
- regrouper plusieurs providers avec `MultiProvider` ;
- éviter le piège de `Provider.value` et de la recréation d'objet ;
- faire dépendre un état d'un autre avec `ProxyProvider` ;
- exposer un `Future` ou un `Stream` avec `FutureProvider` et `StreamProvider` ;
- séparer proprement le modèle de l'interface ;
- écrire un panier d'achat complet partagé entre plusieurs écrans ;
- tester une classe d'état sans lancer d'interface ;
- décrire honnêtement Riverpod, Bloc, GetX et les signals ;
- justifier votre choix d'architecture devant un collègue.

---

## 52.0.1 — Avertissement sur la progression

Ce chapitre est un chapitre **d'architecture**. Il ne présente presque aucun widget
nouveau : il explique comment organiser ce que vous savez déjà faire.

Nous utiliserons librement tout ce qui a été vu dans les chapitres 43 à 51 :
`Scaffold`, `AppBar`, `ListView`, `TextField`, `Navigator`, `ThemeData`. En
revanche, nous n'utiliserons pas encore :

```text
- les appels réseau (http, JSON distant)     -> chapitre 53
- la persistance locale (shared_preferences) -> chapitre 54
```

Toutes les données seront donc créées en mémoire, dans le code. C'est volontaire :
le sujet ici n'est pas d'où viennent les données, mais **qui les détient et qui est
prévenu quand elles changent**.

Chaque bloc de code est un `main.dart` **complet**. Vous pouvez le coller tel quel
dans `lib/main.dart` d'un projet créé au chapitre 43, puis lancer `flutter run`.
Les exemples qui utilisent le paquet `provider` nécessitent en plus la commande
d'installation donnée en 52.18 ; ils sont signalés explicitement.

---

## 52.1 — Qu'appelle-t-on « gestion d'état » ?

Reprenons la définition du chapitre 45 :

> **L'état d'une interface, c'est l'ensemble des informations qui peuvent changer
> pendant que l'écran est affiché, et dont l'affichage dépend.**

La **gestion d'état**, c'est la réponse à trois questions, et seulement trois :

```text
   1. OÙ vit la donnée ?
      (dans quel objet, à quel endroit de l'arbre de widgets)

   2. QUI a le droit de la modifier ?
      (quel code appelle la méthode qui change la valeur)

   3. QUI est prévenu quand elle change ?
      (quels widgets se reconstruisent, et lesquels ne bougent pas)
```

Tout le reste — `setState`, `InheritedWidget`, `ChangeNotifier`, `provider`,
Riverpod, Bloc — n'est qu'une **façon différente de répondre à ces trois questions**.

Retenez bien cela. Les débutants croient souvent qu'ils doivent « apprendre
provider ». Non : ils doivent apprendre à répondre aux trois questions, et
`provider` est un outil parmi d'autres pour y répondre.

---

## 52.1.1 — Le problème n'est pas technique, il est organisationnel

Voici une observation qui surprend souvent.

Une application de 3 écrans avec `setState` partout fonctionne parfaitement. Elle
est même plus simple qu'une application avec `provider`. Le problème n'apparaît
qu'à partir d'une certaine taille :

```text
  Nombre d'écrans qui partagent la MÊME donnée
        │
      1 │  setState suffit largement
        │
      2 │  setState + remontée d'état, encore acceptable
        │
      3 │  ça commence à faire mal
        │
      5 │  ingérable : chaque ajout casse quelque chose
        │
     10 │  personne ne comprend plus qui modifie quoi
        v
```

La gestion d'état est donc d'abord un problème de **lisibilité et de maintenance**,
pas de performance. Une application mal architecturée est rarement lente ; elle est
surtout impossible à faire évoluer.

---

## 52.1.2 — Le vocabulaire, une fois pour toutes

Ces mots reviendront cent fois. Fixons-les maintenant.

| Terme | Signification exacte |
| --- | --- |
| **État** | Une donnée qui change et dont l'affichage dépend |
| **Source de vérité** | L'unique objet qui détient la valeur officielle |
| **Reconstruction** (*rebuild*) | Un nouvel appel à `build()` d'un widget |
| **Écouteur** (*listener*) | Une fonction appelée quand la donnée change |
| **Notification** | L'acte de prévenir tous les écouteurs |
| **Consommateur** | Un widget qui lit l'état et se reconstruit avec lui |
| **Portée** (*scope*) | La partie de l'arbre où un état est accessible |
| **Injection de dépendance** | Le fait de fournir un objet à un widget sans le lui passer par constructeur |

Le terme le plus important est **source de vérité** (*single source of truth*).

> Une donnée ne doit exister qu'à **un seul endroit**. Si la même information est
> stockée dans deux objets différents, vous aurez tôt ou tard deux valeurs
> différentes, et l'utilisateur verra une incohérence.

---

## 52.2 — État éphémère et état d'application : la distinction fondamentale

Voici la distinction la plus utile de tout le chapitre. Elle vient directement de
la documentation officielle de Flutter.

**L'état éphémère** (*ephemeral state*, aussi appelé *UI state* ou état local) est
un état qu'un seul widget détient, et dont aucun autre widget n'a besoin.

**L'état d'application** (*app state*, aussi appelé état partagé) est un état dont
plusieurs parties de l'application ont besoin, ou qui doit survivre à un changement
d'écran.

Concrètement, dans une application de jeu :

```text
  ÉTAT ÉPHÉMÈRE                         ÉTAT D'APPLICATION
  (un seul widget concerné)             (plusieurs écrans concernés)
  ───────────────────────────           ─────────────────────────────
  l'onglet actif d'un TabBar            le score du joueur
  le texte tapé dans un TextField       l'inventaire
  l'animation en cours                  le personnage sélectionné
  le panneau replié / déplié            les préférences (thème, langue)
  la position du défilement             le panier d'achat
  "le mot de passe est-il visible ?"    l'utilisateur connecté
  un booléen _chargementEnCours local   la liste des ennemis vaincus
```

La règle de décision tient en une phrase :

> **Si un autre widget que celui-ci a besoin de lire ou de modifier cette donnée,
> ce n'est plus de l'état éphémère.**

---

## 52.2.1 — Le test des trois questions

Devant une donnée, posez-vous ces trois questions. Une seule réponse « oui » suffit
à en faire de l'état d'application.

```text
  1. Un autre écran affiche-t-il cette donnée ?
  2. La donnée doit-elle survivre quand on quitte cet écran ?
  3. Un autre widget peut-il la modifier ?

     ┌─────────────┬──────────────────────────────┐
     │ 3 fois non  │  état éphémère -> setState   │
     ├─────────────┼──────────────────────────────┤
     │ au moins    │  état d'application          │
     │ un oui      │  -> ChangeNotifier + provider│
     └─────────────┴──────────────────────────────┘
```

Prenons un exemple concret. Dans un jeu, le booléen « le menu pause est-il
ouvert ? » :

1. un autre écran l'affiche-t-il ? Non.
2. doit-il survivre au changement d'écran ? Non, on ferme le menu en partant.
3. un autre widget le modifie-t-il ? Non, seul le bouton pause.

Trois « non » : c'est de l'état éphémère, `setState` suffit.

Maintenant, le nombre de potions du joueur :

1. un autre écran l'affiche-t-il ? Oui, l'écran d'inventaire **et** le HUD de jeu.
2. doit-il survivre au changement d'écran ? Oui, évidemment.
3. un autre widget le modifie-t-il ? Oui, la boutique et le combat.

Trois « oui » : c'est de l'état d'application.

---

## 52.2.2 — Le piège de la mauvaise classification

L'erreur la plus coûteuse en Flutter n'est pas de mal utiliser `provider`. C'est de
classer une donnée en état éphémère alors qu'elle est en réalité partagée.

Vous écrivez un compteur de potions dans le `State` de l'écran de jeu. Trois
semaines plus tard, on vous demande d'ajouter un écran d'inventaire. Vous devez
alors :

```text
  1. sortir la variable du State
  2. la remonter dans un parent commun
  3. réécrire tous les setState en callbacks
  4. modifier les constructeurs de tous les widgets intermédiaires
  5. retester tout l'écran de jeu
```

Cinq étapes, plusieurs heures, un risque de régression. Alors que la classer
correctement dès le départ aurait coûté dix lignes.

> **En cas de doute, considérez la donnée comme de l'état d'application.** Le coût
> d'un `ChangeNotifier` inutile est de vingt lignes ; le coût d'une refonte est de
> plusieurs heures.

Attention toutefois à ne pas basculer dans l'excès inverse, décrit en 52.34 : tout
mettre dans un état global rend l'application aussi illisible que tout mettre en
local.

---

## 52.3 — Tableau : quel type d'état pour quel outil

Ce tableau est le plan de vol du chapitre. Chaque ligne sera détaillée ensuite.

| Situation | Outil recommandé | Section |
| --- | --- | --- |
| Une donnée, un seul widget | `setState()` | 52.4 |
| Une donnée, un widget et ses enfants directs | `setState()` + paramètres | 52.5 |
| Une donnée, deux widgets frères | remontée d'état (*lifting state up*) | 52.5 |
| Une valeur unique partagée, sans dépendance | `ValueNotifier` + `ValueListenableBuilder` | 52.16 |
| Un objet à plusieurs champs, sans dépendance | `ChangeNotifier` + `ListenableBuilder` | 52.17 |
| Une donnée immuable partagée par tout l'arbre | `InheritedWidget` | 52.9 |
| Un objet d'état partagé par plusieurs écrans | `ChangeNotifierProvider` | 52.19 |
| Plusieurs objets d'état indépendants | `MultiProvider` | 52.25 |
| Un état qui dépend d'un autre état | `ProxyProvider` | 52.27 |
| Un résultat asynchrone unique | `FutureProvider` | 52.28 |
| Un flux de valeurs | `StreamProvider` | 52.28 |
| Une application de très grande taille | Riverpod ou Bloc | 52.32 |

Notez qu'aucune ligne ne dit « toujours utiliser `provider` ». Les cinq premières
lignes se résolvent sans aucune dépendance externe, et elles couvrent la majorité
du code d'une application réelle.

---

## 52.4 — `setState()` : rappel et limites (chapitre 45)

Rappel du chapitre 45. `setState()` fait exactement deux choses :

```text
  setState(() { compteur++; });

     1. il exécute la fonction que vous lui passez
        (c'est vous qui modifiez la variable, pas lui)

     2. il marque l'élément associé comme "à reconstruire"
        (Flutter rappellera build() au prochain rafraîchissement)
```

Voici le compteur canonique, en version complète.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationJeu());
}

class ApplicationJeu extends StatelessWidget {
  const ApplicationJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Score local',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranScore(),
    );
  }
}

class EcranScore extends StatefulWidget {
  const EcranScore({super.key});

  @override
  State<EcranScore> createState() => _EcranScoreState();
}

class _EcranScoreState extends State<EcranScore> {
  int _score = 0;

  void _ramasserPiece() {
    setState(() {
      _score += 50;
    });
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build de EcranScore');
    return Scaffold(
      appBar: AppBar(title: const Text('Score local')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'Score : $_score',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _ramasserPiece,
              child: const Text('Ramasser une pièce'),
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
Score : 0
[ Ramasser une pièce ]

Après trois appuis :

Score : 150

Console :
build de EcranScore
build de EcranScore
build de EcranScore
build de EcranScore
```

Ce code est **correct**. Il n'y a rien à améliorer. Ne cherchez pas à y mettre
`provider` : le score n'est utilisé nulle part ailleurs.

---

## 52.4.1 — Limite 1 : `setState` reconstruit tout le `build()`

Regardez la trace ci-dessus. Chaque appui reconstruit **tout** `_EcranScoreState`,
donc le `Scaffold`, l'`AppBar`, la `Column`, le bouton — alors que seul le `Text`
du score a changé.

```text
  AVANT setState                   APRÈS setState
  ──────────────                   ──────────────
  Scaffold      (inchangé)         Scaffold      RECONSTRUIT
   └ AppBar     (inchangé)          └ AppBar     RECONSTRUIT
   └ Center     (inchangé)          └ Center     RECONSTRUIT
     └ Column   (inchangé)            └ Column   RECONSTRUIT
       └ Text   "Score : 0"             └ Text   RECONSTRUIT (utile)
       └ Button (inchangé)              └ Button RECONSTRUIT
```

Cinq reconstructions inutiles pour une utile. Sur cet écran, l'impact est nul :
reconstruire un `Text` coûte quelques microsecondes. Sur une liste de 200 éléments
avec des images, cela devient perceptible.

Le mot-clé `const` (chapitre 44) limite déjà les dégâts : un widget `const` n'est
pas reconstruit, il est réutilisé tel quel. Mais `const` ne peut rien pour un widget
qui dépend d'une variable.

---

## 52.4.2 — Limite 2 : `setState` est enfermé dans un `State`

`setState()` est une méthode de la classe `State`. Vous ne pouvez donc l'appeler
que **depuis l'intérieur** de ce `State`.

```text
        EcranJeu (State)  ──── setState() accessible ici
             │
             ├── HudScore     ──── setState() du parent INACCESSIBLE
             │
             └── BoutonAttaque ─── setState() du parent INACCESSIBLE
```

Un widget enfant ne peut donc pas modifier l'état de son parent directement. Il
faut lui passer une fonction, ce que nous appelons un **callback**. Et c'est là que
les ennuis commencent, comme nous le verrons en 52.6.

---

## 52.4.3 — Limite 3 : l'état meurt avec le widget

Un `State` est détruit quand son widget quitte l'arbre. Testez ce code : le compteur
revient à zéro dès qu'on change d'onglet, parce que `IndexedStack` est remplacé par
une reconstruction conditionnelle.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationOnglets());
}

class ApplicationOnglets extends StatelessWidget {
  const ApplicationOnglets({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
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
  int _ongletActif = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('État volatil')),
      // Attention : on RECONSTRUIT l'onglet à chaque changement.
      body: _ongletActif == 0 ? const OngletScore() : const OngletAide(),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _ongletActif,
        onDestinationSelected: (int index) {
          setState(() {
            _ongletActif = index;
          });
        },
        destinations: const <NavigationDestination>[
          NavigationDestination(icon: Icon(Icons.star), label: 'Score'),
          NavigationDestination(icon: Icon(Icons.help), label: 'Aide'),
        ],
      ),
    );
  }
}

class OngletScore extends StatefulWidget {
  const OngletScore({super.key});

  @override
  State<OngletScore> createState() => _OngletScoreState();
}

class _OngletScoreState extends State<OngletScore> {
  int _score = 0;

  @override
  void initState() {
    super.initState();
    debugPrint('initState de OngletScore : le score repart de 0');
  }

  @override
  void dispose() {
    debugPrint('dispose de OngletScore : le score $_score est PERDU');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Text('Score : $_score', style: const TextStyle(fontSize: 32)),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: () => setState(() => _score += 50),
            child: const Text('+50'),
          ),
          const SizedBox(height: 16),
          const Text('Changez d\'onglet puis revenez.'),
        ],
      ),
    );
  }
}

class OngletAide extends StatelessWidget {
  const OngletAide({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Padding(
        padding: EdgeInsets.all(24),
        child: Text(
          'Le score de l\'onglet précédent vient d\'être détruit. '
          'Revenez en arrière pour le constater.',
          textAlign: TextAlign.center,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Console, après avoir marqué 150 points puis changé d'onglet :

initState de OngletScore : le score repart de 0
dispose de OngletScore : le score 150 est PERDU
initState de OngletScore : le score repart de 0
```

L'état est mort avec le widget. C'est le comportement normal et voulu de Flutter :
un `State` appartient à une position dans l'arbre, pas à l'application.

> **Trois limites, une conclusion :** `setState` gère parfaitement l'état d'un
> widget, et ne gère pas du tout l'état d'une application.

---

## 52.5 — La remontée d'état : rappel

Le chapitre 45 a présenté la première réponse : la **remontée d'état**
(*lifting state up*).

Le principe :

> Quand deux widgets ont besoin de la même donnée, cette donnée doit vivre dans
> leur **plus proche ancêtre commun**.

```text
  AVANT (ne marche pas)              APRÈS (remontée d'état)

     EcranJeu                           EcranJeu   <- score vit ICI
      ├── HudScore  (score=?)            ├── HudScore(score: 150)
      └── BoutonPiece (score=150)        └── BoutonPiece(onRamasse: ...)
```

L'ancêtre commun détient la donnée. Il la fait **descendre** par les constructeurs,
et fait **remonter** les actions par des callbacks.

Voici l'exemple complet.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationRemontee());
}

class ApplicationRemontee extends StatelessWidget {
  const ApplicationRemontee({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const EcranJeu(),
    );
  }
}

// L'ANCÊTRE COMMUN : c'est lui qui détient l'état.
class EcranJeu extends StatefulWidget {
  const EcranJeu({super.key});

  @override
  State<EcranJeu> createState() => _EcranJeuState();
}

class _EcranJeuState extends State<EcranJeu> {
  int _score = 0;

  void _ramasser(int valeur) {
    setState(() {
      _score += valeur;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Remontée d\'état')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            // La donnée DESCEND par le constructeur.
            HudScore(score: _score),
            const SizedBox(height: 32),
            // L'action REMONTE par un callback.
            BoutonPiece(valeur: 50, onRamasse: _ramasser),
            const SizedBox(height: 12),
            BoutonPiece(valeur: 200, onRamasse: _ramasser),
          ],
        ),
      ),
    );
  }
}

// Widget d'affichage : sans état, il reçoit tout.
class HudScore extends StatelessWidget {
  const HudScore({super.key, required this.score});

  final int score;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Text(
          'Score : $score',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
      ),
    );
  }
}

// Widget d'action : sans état, il ne fait qu'appeler le callback.
class BoutonPiece extends StatelessWidget {
  const BoutonPiece({
    super.key,
    required this.valeur,
    required this.onRamasse,
  });

  final int valeur;
  final void Function(int valeur) onRamasse;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => onRamasse(valeur),
      child: Text('Ramasser $valeur points'),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────┐
│ Remontée d'état              │
├──────────────────────────────┤
│                              │
│      ┌──────────────┐        │
│      │ Score : 250  │        │
│      └──────────────┘        │
│                              │
│   [ Ramasser 50 points ]     │
│   [ Ramasser 200 points ]    │
│                              │
└──────────────────────────────┘
```

C'est propre, c'est explicite, c'est testable. Sur deux niveaux, la remontée d'état
est **la bonne solution**. Ne cherchez pas plus compliqué.

---

## 52.5.1 — Ce que la remontée d'état garantit

Trois qualités qu'aucune autre solution ne dépasse :

| Qualité | Explication |
| --- | --- |
| **Explicite** | En lisant le constructeur, vous savez exactement de quoi le widget dépend |
| **Réutilisable** | `HudScore` peut afficher n'importe quel entier, il ne connaît aucun état |
| **Testable** | `HudScore(score: 42)` se teste sans aucun contexte particulier |

Un widget qui reçoit tout par son constructeur est appelé un **widget de
présentation** (*presentational widget*) ou **widget bête** (*dumb widget*). C'est
un compliment : c'est le meilleur type de widget qui existe.

Gardez ce principe même quand vous utiliserez `provider` : dans l'idéal, seul un
petit nombre de widgets lisent l'état partagé, et ils passent les valeurs à des
widgets de présentation.

---

## 52.6 — Le forage de propriétés (*prop drilling*) : le problème illustré sur cinq niveaux

Maintenant, poussons la remontée d'état jusqu'à son point de rupture.

Imaginons une application de jeu avec cette structure réelle :

```text
  EcranJeu                       <- le score vit ici
   └── CorpsJeu
        └── ZoneBasse
             └── BarreOutils
                  └── HudScore   <- le score est affiché ici
```

Cinq niveaux. Le score doit traverser `CorpsJeu`, `ZoneBasse` et `BarreOutils`,
alors qu'**aucun de ces trois widgets ne s'en sert**. Ils le transportent, c'est
tout. C'est ce qu'on appelle le **forage de propriétés** (*prop drilling*) : on
force un tuyau à travers des couches qui n'ont rien demandé.

Voici le code. Comptez le nombre de fois où le mot `score` apparaît.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationForage());
}

class ApplicationForage extends StatelessWidget {
  const ApplicationForage({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
        useMaterial3: true,
      ),
      home: const EcranJeu(),
    );
  }
}

// NIVEAU 1 : propriétaire de l'état.
class EcranJeu extends StatefulWidget {
  const EcranJeu({super.key});

  @override
  State<EcranJeu> createState() => _EcranJeuState();
}

class _EcranJeuState extends State<EcranJeu> {
  int _score = 0;
  int _vies = 3;

  void _ramasser() => setState(() => _score += 50);
  void _perdreUneVie() => setState(() => _vies--);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Forage de propriétés')),
      body: CorpsJeu(
        score: _score,
        vies: _vies,
        onRamasse: _ramasser,
        onPerdVie: _perdreUneVie,
      ),
    );
  }
}

// NIVEAU 2 : ne se sert de RIEN, transporte TOUT.
class CorpsJeu extends StatelessWidget {
  const CorpsJeu({
    super.key,
    required this.score,
    required this.vies,
    required this.onRamasse,
    required this.onPerdVie,
  });

  final int score;
  final int vies;
  final VoidCallback onRamasse;
  final VoidCallback onPerdVie;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        const Expanded(child: Center(child: Text('Zone de jeu'))),
        ZoneBasse(
          score: score,
          vies: vies,
          onRamasse: onRamasse,
          onPerdVie: onPerdVie,
        ),
      ],
    );
  }
}

// NIVEAU 3 : ne se sert de RIEN, transporte TOUT.
class ZoneBasse extends StatelessWidget {
  const ZoneBasse({
    super.key,
    required this.score,
    required this.vies,
    required this.onRamasse,
    required this.onPerdVie,
  });

  final int score;
  final int vies;
  final VoidCallback onRamasse;
  final VoidCallback onPerdVie;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Theme.of(context).colorScheme.surfaceContainerHighest,
      padding: const EdgeInsets.all(12),
      child: BarreOutils(
        score: score,
        vies: vies,
        onRamasse: onRamasse,
        onPerdVie: onPerdVie,
      ),
    );
  }
}

// NIVEAU 4 : ne se sert de RIEN, transporte TOUT.
class BarreOutils extends StatelessWidget {
  const BarreOutils({
    super.key,
    required this.score,
    required this.vies,
    required this.onRamasse,
    required this.onPerdVie,
  });

  final int score;
  final int vies;
  final VoidCallback onRamasse;
  final VoidCallback onPerdVie;

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: <Widget>[
        HudScore(score: score, vies: vies),
        Row(
          children: <Widget>[
            IconButton(
              onPressed: onRamasse,
              icon: const Icon(Icons.star),
              tooltip: 'Ramasser',
            ),
            IconButton(
              onPressed: onPerdVie,
              icon: const Icon(Icons.heart_broken),
              tooltip: 'Perdre une vie',
            ),
          ],
        ),
      ],
    );
  }
}

// NIVEAU 5 : enfin quelqu'un qui S'EN SERT.
class HudScore extends StatelessWidget {
  const HudScore({super.key, required this.score, required this.vies});

  final int score;
  final int vies;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Text('Score : $score'),
        Text('Vies  : $vies'),
      ],
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────┐
│ Forage de propriétés         │
├──────────────────────────────┤
│                              │
│         Zone de jeu          │
│                              │
├──────────────────────────────┤
│ Score : 100     (etoile)(coeur)│
│ Vies  : 2                    │
└──────────────────────────────┘
```

Le résultat visuel est modeste. Le code, lui, ne l'est pas : **le mot `score`
apparaît 17 fois**, dont 12 dans des widgets qui ne l'utilisent pas.

---

## 52.6.1 — Le coût réel du forage, mesuré

Mettons des chiffres sur le problème.

```text
  Pour transporter UNE donnée sur N niveaux, il faut écrire :

     N - 1  déclarations de champ  (final int score;)
     N - 1  paramètres nommés      (required this.score,)
     N - 1  passages au constructeur (score: score,)
     ─────
     3 x (N - 1) lignes purement mécaniques
```

Pour notre exemple : 4 données (`score`, `vies`, `onRamasse`, `onPerdVie`) sur 5
niveaux :

```text
     4 données x 3 lignes x 4 niveaux intermédiaires = 48 lignes
```

Quarante-huit lignes de code qui ne font **rien**. Elles ne calculent rien,
n'affichent rien, ne décident rien. Elles transportent.

Et maintenant, ajoutez une cinquième donnée, `energie`. Vous devez modifier **quatre
fichiers** pour afficher une valeur. Puis on vous demande de déplacer `HudScore`
dans l'`AppBar` : vous devez démonter puis remonter tout le tuyau.

---

## 52.6.2 — Les trois symptômes du forage

Vous êtes en train de forer si vous constatez l'un de ces symptômes :

```text
  SYMPTÔME 1 : un widget déclare un champ qu'il ne lit jamais,
               sinon pour le repasser à son enfant.

  SYMPTÔME 2 : ajouter une donnée d'affichage impose de modifier
               trois fichiers ou plus.

  SYMPTÔME 3 : déplacer un widget dans l'arbre casse la compilation
               à cause de paramètres manquants.
```

Ces trois symptômes se soignent tous de la même manière : au lieu de faire
**descendre** la donnée niveau par niveau, on la met **à disposition** de tout le
sous-arbre, et chaque widget la lit directement là où il en a besoin.

C'est exactement ce que fait `InheritedWidget`, que nous voyons en 52.8.

---

## 52.7 — Pourquoi les callbacks ne suffisent plus

Le forage descendant (les données) a un jumeau montant : le forage des
**callbacks** (les actions). Et celui-là est encore pire.

Reprenons l'exemple. Le bouton « Ramasser » est au niveau 4. Il doit modifier une
variable du niveau 1. La fonction `_ramasser` a donc dû traverser quatre
constructeurs.

```text
  DESCENTE DES DONNÉES            REMONTÉE DES ACTIONS

  EcranJeu  score=100             EcranJeu  _ramasser()
     │  score: 100                    ^  onRamasse: _ramasser
     v                                │
  CorpsJeu                        CorpsJeu
     │  score: 100                    ^  onRamasse: onRamasse
     v                                │
  ZoneBasse                       ZoneBasse
     │  score: 100                    ^  onRamasse: onRamasse
     v                                │
  BarreOutils                     BarreOutils
     │  score: 100                    ^  onRamasse: onRamasse
     v                                │
  HudScore  "Score : 100"         IconButton  onPressed: onRamasse
```

Deux tuyaux parallèles à maintenir en permanence.

---

## 52.7.1 — L'explosion combinatoire

Le vrai problème apparaît quand les actions se multiplient. Une application de jeu
réaliste a facilement une dizaine d'actions :

```text
  onRamasse         onPerdVie        onUtilisePotion
  onEquipeArme      onChangeNiveau   onOuvreInventaire
  onSauvegarde      onPause          onReprend
  onAchete
```

Dix callbacks x 4 niveaux x 3 lignes = **120 lignes** de tuyauterie. Et chaque
signature de constructeur devient illisible :

```dart
// Ce genre de constructeur est un signal d'alarme.
class BarreOutils extends StatelessWidget {
  const BarreOutils({
    super.key,
    required this.score,
    required this.vies,
    required this.energie,
    required this.niveau,
    required this.inventaire,
    required this.onRamasse,
    required this.onPerdVie,
    required this.onUtilisePotion,
    required this.onEquipeArme,
    required this.onChangeNiveau,
    required this.onOuvreInventaire,
    required this.onSauvegarde,
    required this.onPause,
    required this.onReprend,
    required this.onAchete,
  });
  // ... quinze champs final, quinze lignes de plus
}
```

---

## 52.7.2 — Le problème des frères éloignés

Il existe un cas où les callbacks ne sont pas seulement pénibles, mais **absurdes**.

Imaginez deux écrans distincts, atteints par `Navigator.push` (chapitre 50) :

```text
  MaterialApp
   ├── EcranBoutique    (achète une potion)
   └── EcranInventaire  (affiche les potions)
```

Ces deux écrans ne sont pas parent et enfant. Ils sont poussés l'un après l'autre
sur la pile de navigation. Leur ancêtre commun est le `MaterialApp` lui-même.

Pour partager les potions par remontée d'état, il faudrait :

```text
  1. mettre l'état des potions dans un StatefulWidget au-dessus de MaterialApp
  2. passer la valeur ET le callback à CHAQUE route
  3. les repasser à chaque écran poussé depuis ces routes
  4. gérer le retour de valeur de Navigator.pop pour resynchroniser
```

L'étape 4 est le coup de grâce : `Navigator.pop(context, resultat)` permet bien de
renvoyer une valeur, mais il faut alors que chaque écran pense à la traiter. Si un
écran oublie, l'état se désynchronise silencieusement.

> **La remontée d'état ne fonctionne bien que dans un sous-arbre profond de 2 ou 3
> niveaux, sans navigation entre les participants.** Au-delà, il faut un autre
> mécanisme.

---

## 52.7.3 — Ce qu'il nous faudrait idéalement

Écrivons le cahier des charges de la solution que nous cherchons :

```text
  1. La donnée vit à UN seul endroit, au-dessus de tous ses utilisateurs.

  2. N'IMPORTE QUEL descendant peut la LIRE, sans que les widgets
     intermédiaires aient à la connaître.

  3. N'IMPORTE QUEL descendant peut demander à la MODIFIER,
     sans chaîne de callbacks.

  4. Quand elle change, SEULS les widgets qui la lisent se reconstruisent.

  5. La donnée survit à la navigation entre écrans.
```

Les points 1, 2 et 4 sont exactement ce que fournit `InheritedWidget`, le mécanisme
natif de Flutter. Les points 3 et 5 demanderont un peu plus de travail.

Nous allons donc étudier `InheritedWidget` en détail. Non pas parce que vous en
écrirez souvent à la main — vous n'en écrirez presque jamais — mais parce que
**tout le reste repose dessus** : `Theme`, `MediaQuery`, `Navigator`, `provider`,
Riverpod. Comprendre `InheritedWidget`, c'est comprendre Flutter.

---

## 52.8 — `InheritedWidget` : le mécanisme sous-jacent de Flutter

Un `InheritedWidget` est un widget qui **met une valeur à disposition de tout son
sous-arbre**, de manière que n'importe quel descendant puisse la récupérer en une
ligne, quelle que soit sa profondeur.

Sa déclaration officielle, dans `package:flutter/widgets.dart` :

```text
abstract class InheritedWidget extends ProxyWidget
```

Son constructeur ne prend que deux choses :

```text
InheritedWidget({Key? key, required Widget child})
```

Une sous-classe ajoute ses propres champs — la donnée à partager — et implémente
une seule méthode obligatoire :

```text
bool updateShouldNotify(covariant InheritedWidget oldWidget)
```

---

## 52.8.1 — L'idée : un raccourci dans l'arbre

Sans `InheritedWidget`, une donnée voyage de parent en enfant :

```text
  A ──> B ──> C ──> D ──> E        chaque flèche = un paramètre de constructeur
```

Avec un `InheritedWidget` placé en A, la donnée est **posée** dans l'arbre. N'importe
quel descendant remonte directement jusqu'à elle :

```text
  A (InheritedWidget, score = 100)
  │
  ├─ B
  │  └─ C
  │     └─ D
  │        └─ E ──────┐
  │                   │  E remonte l'arbre et trouve A
  └───────────────────┘  en une seule opération
```

B, C et D ne savent rien du score. Ils ne le déclarent pas, ne le transportent pas,
ne se reconstruisent pas quand il change.

---

## 52.8.2 — Comment Flutter retrouve l'ancêtre si vite

Un débutant imagine que Flutter remonte l'arbre widget par widget jusqu'à trouver
le bon type. Ce serait lent sur un arbre profond.

En réalité, chaque `Element` de l'arbre (chapitre 44) maintient une **table de
correspondance** des `InheritedWidget` visibles depuis sa position :

```text
  Element de E, champ _inheritedElements :

     { Theme        -> Element du Theme
       MediaQuery   -> Element du MediaQuery
       ScoreHeritee -> Element du ScoreHeritee
       Directionality -> ... }
```

Cette table est **héritée du parent et copiée** lors du montage de l'élément. La
recherche est donc une simple lecture dans une table de hachage :

```text
  Coût de la recherche : O(1), quelle que soit la profondeur de l'arbre.
```

C'est pour cette raison que `Theme.of(context)` est utilisable des milliers de fois
dans une application sans aucun impact mesurable.

---

## 52.8.3 — Le double rôle : lecture et abonnement

Voici le point que les débutants ratent systématiquement.

Quand un widget appelle `dependOnInheritedWidgetOfExactType`, il fait **deux**
choses :

```text
  1. il LIT la valeur portée par l'InheritedWidget ;
  2. il S'ABONNE : Flutter note que ce widget dépend de cet InheritedWidget.
```

Le point 2 est le mécanisme central. Grâce à cet abonnement, quand
l'`InheritedWidget` est remplacé par une nouvelle instance portant une valeur
différente, Flutter sait **exactement** quels widgets reconstruire — et seulement
ceux-là.

```text
  Le score passe de 100 à 150 :

      A (nouvel InheritedWidget, score = 150)
      │
      ├─ B    pas abonné  ->  PAS reconstruit
      │  └─ C pas abonné  ->  PAS reconstruit
      │     └─ D pas abonné -> PAS reconstruit
      │        └─ E ABONNÉ ->  RECONSTRUIT
      │
      └─ F ABONNÉ         ->  RECONSTRUIT
```

C'est la réponse au point 4 de notre cahier des charges de 52.7.3 : seuls les
widgets qui lisent la donnée se reconstruisent.

---

## 52.9 — Écrire son propre `InheritedWidget`

Passons à la pratique. Nous allons partager un score entre plusieurs widgets
profonds, sans aucun paramètre de constructeur intermédiaire.

Un `InheritedWidget` s'écrit toujours en quatre parties :

```text
  1. la classe étend InheritedWidget
  2. elle déclare ses champs FINAL (la donnée partagée)
  3. elle expose une méthode statique of(context) et maybeOf(context)
  4. elle implémente updateShouldNotify
```

Voici le patron officiel, repris de la documentation de Flutter (exemple
`FrogColor`) et adapté à notre fil rouge.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationHeritee());
}

class ApplicationHeritee extends StatelessWidget {
  const ApplicationHeritee({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      home: const EcranJeu(),
    );
  }
}

// ─────────────────────────────────────────────────────────────
// 1. L'InheritedWidget lui-même.
// ─────────────────────────────────────────────────────────────
class ScoreHeritee extends InheritedWidget {
  const ScoreHeritee({
    super.key,
    required this.score,
    required this.vies,
    required super.child,
  });

  // 2. Les champs partagés : TOUJOURS final.
  final int score;
  final int vies;

  // 3a. maybeOf : renvoie null si l'ancêtre n'existe pas.
  static ScoreHeritee? maybeOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ScoreHeritee>();
  }

  // 3b. of : lève une erreur claire si l'ancêtre n'existe pas.
  static ScoreHeritee of(BuildContext context) {
    final ScoreHeritee? resultat = maybeOf(context);
    assert(resultat != null, 'Aucun ScoreHeritee trouvé dans le contexte');
    return resultat!;
  }

  // 4. Faut-il prévenir les abonnés ?
  @override
  bool updateShouldNotify(ScoreHeritee oldWidget) {
    return score != oldWidget.score || vies != oldWidget.vies;
  }
}

// ─────────────────────────────────────────────────────────────
// Le propriétaire de l'état : il reconstruit l'InheritedWidget.
// ─────────────────────────────────────────────────────────────
class EcranJeu extends StatefulWidget {
  const EcranJeu({super.key});

  @override
  State<EcranJeu> createState() => _EcranJeuState();
}

class _EcranJeuState extends State<EcranJeu> {
  int _score = 0;
  int _vies = 3;

  @override
  Widget build(BuildContext context) {
    return ScoreHeritee(
      score: _score,
      vies: _vies,
      child: Scaffold(
        appBar: AppBar(title: const Text('InheritedWidget')),
        body: const CorpsJeu(),
        floatingActionButton: Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: <Widget>[
            FloatingActionButton(
              heroTag: 'piece',
              onPressed: () => setState(() => _score += 50),
              child: const Icon(Icons.star),
            ),
            const SizedBox(width: 12),
            FloatingActionButton(
              heroTag: 'vie',
              onPressed: () => setState(() {
                if (_vies > 0) _vies--;
              }),
              child: const Icon(Icons.heart_broken),
            ),
          ],
        ),
      ),
    );
  }
}

// ─────────────────────────────────────────────────────────────
// Les niveaux intermédiaires : AUCUN paramètre, tous const.
// ─────────────────────────────────────────────────────────────
class CorpsJeu extends StatelessWidget {
  const CorpsJeu({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build CorpsJeu');
    return const Column(
      children: <Widget>[
        Expanded(child: Center(child: Text('Zone de jeu'))),
        ZoneBasse(),
        SizedBox(height: 80),
      ],
    );
  }
}

class ZoneBasse extends StatelessWidget {
  const ZoneBasse({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build ZoneBasse');
    return const Padding(
      padding: EdgeInsets.all(12),
      child: BarreOutils(),
    );
  }
}

class BarreOutils extends StatelessWidget {
  const BarreOutils({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BarreOutils');
    return const Row(
      mainAxisAlignment: MainAxisAlignment.spaceAround,
      children: <Widget>[
        AfficheScore(),
        AfficheVies(),
      ],
    );
  }
}

// ─────────────────────────────────────────────────────────────
// Les consommateurs : ils lisent directement l'ancêtre.
// ─────────────────────────────────────────────────────────────
class AfficheScore extends StatelessWidget {
  const AfficheScore({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build AfficheScore');
    final ScoreHeritee donnees = ScoreHeritee.of(context);
    return Text(
      'Score : ${donnees.score}',
      style: Theme.of(context).textTheme.titleLarge,
    );
  }
}

class AfficheVies extends StatelessWidget {
  const AfficheVies({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build AfficheVies');
    final ScoreHeritee donnees = ScoreHeritee.of(context);
    return Text(
      'Vies : ${donnees.vies}',
      style: Theme.of(context).textTheme.titleLarge,
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────┐
│ InheritedWidget              │
├──────────────────────────────┤
│                              │
│         Zone de jeu          │
│                              │
│   Score : 100   Vies : 3     │
│                     (+) (-)  │
└──────────────────────────────┘

Console, au tout premier affichage :
build CorpsJeu
build ZoneBasse
build BarreOutils
build AfficheScore
build AfficheVies

Console, après un appui sur l'étoile :
build AfficheScore
build AfficheVies
```

Regardez bien la seconde trace. `CorpsJeu`, `ZoneBasse` et `BarreOutils` ne se
reconstruisent **pas**. Ils sont `const`, donc Flutter réutilise les instances
existantes ; et surtout ils ne sont pas abonnés.

Comparez avec le forage de 52.6 : là-bas, les quatre niveaux se reconstruisaient à
chaque changement.

---

## 52.9.1 — Pourquoi les champs doivent être `final`

Un `InheritedWidget` est un widget. Comme tout widget, il est **immuable** : on ne
modifie jamais ses champs, on en crée une nouvelle instance.

```dart
// FAUX : ne compile même pas si le champ est final,
// et ne redessine rien s'il ne l'est pas.
ScoreHeritee.of(context).score = 150;
```

Le cycle correct est toujours le même :

```text
  1. le State appelle setState()
  2. build() est rappelé
  3. build() crée une NOUVELLE instance de ScoreHeritee, avec score = 150
  4. Flutter compare l'ancienne et la nouvelle via updateShouldNotify
  5. si true, tous les abonnés sont reconstruits
```

Un `InheritedWidget` ne **stocke** donc rien de mutable : il **transporte** une
valeur produite par un `State` situé au-dessus de lui.

---

## 52.10 — `dependOnInheritedWidgetOfExactType`

C'est la méthode centrale du mécanisme. Sa signature, définie sur `BuildContext` :

```text
T? dependOnInheritedWidgetOfExactType<T extends InheritedWidget>({Object? aspect})
```

Trois points à retenir sur son nom, qui décrit exactement son comportement.

**`dependOn`** : elle crée une **dépendance**. Le widget appelant sera désormais
reconstruit à chaque notification de cet ancêtre. Ce n'est pas une simple lecture.

**`OfExactType`** : la correspondance porte sur le type **exact**, pas sur les
sous-classes. Si vous demandez `<ScoreHeritee>` et que l'arbre contient une classe
`ScoreHeriteeAvancee extends ScoreHeritee`, elle ne sera **pas** trouvée.

**`<T>`** : le type est un paramètre générique. C'est ce qui permet à Flutter
d'aller chercher directement la bonne entrée dans la table de correspondance
décrite en 52.8.2.

---

## 52.10.1 — La variante sans abonnement

Il existe une seconde méthode, moins connue :

```text
InheritedElement? getElementForInheritedWidgetOfExactType<T extends InheritedWidget>()
```

Celle-ci **lit sans s'abonner**. Elle est utile dans les rares cas où vous voulez
consulter une valeur depuis `initState()` ou depuis un callback, sans provoquer de
reconstruction.

```dart
// Lecture SANS abonnement : le widget ne sera pas reconstruit
// si la valeur change ensuite.
final ScoreHeritee? donnees = context
    .getElementForInheritedWidgetOfExactType<ScoreHeritee>()
    ?.widget as ScoreHeritee?;
```

Retenez la correspondance, elle vous servira pour `provider` :

| Méthode Flutter | Équivalent `provider` | Effet |
| --- | --- | --- |
| `dependOnInheritedWidgetOfExactType` | `context.watch<T>()` | lit **et** s'abonne |
| `getElementForInheritedWidgetOfExactType` | `context.read<T>()` | lit **sans** s'abonner |

---

## 52.10.2 — L'erreur classique : appeler depuis `initState()`

Ce code déclenche une erreur au lancement :

```dart
@override
void initState() {
  super.initState();
  // ERREUR : dépendance créée trop tôt.
  final donnees = ScoreHeritee.of(context);
}
```

**Message réel :**

```text
dependOnInheritedWidgetOfExactType<ScoreHeritee>() or
dependOnInheritedElement() was called before _MonEtatState.initState()
completed.
```

La raison : au moment d'`initState()`, l'élément n'est pas encore complètement
monté dans l'arbre, la table des `InheritedWidget` n'est pas prête.

La solution est `didChangeDependencies()`, la méthode du cycle de vie vue au
chapitre 45 qui est appelée **juste après** `initState()` et **à chaque fois**
qu'une dépendance héritée change :

```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  final ScoreHeritee donnees = ScoreHeritee.of(context);
  debugPrint('Score courant : ${donnees.score}');
}
```

---

## 52.11 — `updateShouldNotify`

C'est la méthode que vous devez implémenter, et la seule que Flutter vous impose.

```text
bool updateShouldNotify(covariant InheritedWidget oldWidget)
```

Flutter l'appelle chaque fois qu'une **nouvelle instance** de votre
`InheritedWidget` remplace l'ancienne au même endroit de l'arbre. Elle reçoit
l'ancienne instance et doit répondre à une question simple :

> « Les abonnés doivent-ils être reconstruits ? »

```text
  return true   ->  tous les widgets abonnés sont reconstruits
  return false  ->  aucun widget n'est reconstruit
```

L'implémentation habituelle compare les champs :

```dart
@override
bool updateShouldNotify(ScoreHeritee oldWidget) {
  return score != oldWidget.score || vies != oldWidget.vies;
}
```

Notez le mot-clé `covariant` dans la signature de base : il autorise votre
redéfinition à typer le paramètre par votre propre classe plutôt que par
`InheritedWidget`. C'est un mécanisme de Dart vu au chapitre 10.

---

## 52.11.1 — Mesurer l'effet de `updateShouldNotify`

Voici un programme complet qui rend le phénomène visible. Un bouton change le score
(la valeur partagée), un autre change une couleur locale qui provoque un
`setState` **sans** modifier le score.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationNotify());
}

class ApplicationNotify extends StatelessWidget {
  const ApplicationNotify({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const EcranDemo(),
    );
  }
}

class ScoreHeritee extends InheritedWidget {
  const ScoreHeritee({
    super.key,
    required this.score,
    required super.child,
  });

  final int score;

  static ScoreHeritee of(BuildContext context) {
    final ScoreHeritee? r =
        context.dependOnInheritedWidgetOfExactType<ScoreHeritee>();
    assert(r != null, 'Aucun ScoreHeritee dans le contexte');
    return r!;
  }

  @override
  bool updateShouldNotify(ScoreHeritee oldWidget) {
    final bool doitNotifier = score != oldWidget.score;
    debugPrint(
      'updateShouldNotify : ${oldWidget.score} -> $score '
      '=> $doitNotifier',
    );
    return doitNotifier;
  }
}

class EcranDemo extends StatefulWidget {
  const EcranDemo({super.key});

  @override
  State<EcranDemo> createState() => _EcranDemoState();
}

class _EcranDemoState extends State<EcranDemo> {
  int _score = 0;
  Color _fond = Colors.white;
  int _compteurRebuildLocal = 0;

  @override
  Widget build(BuildContext context) {
    _compteurRebuildLocal++;
    return ScoreHeritee(
      score: _score,
      child: Scaffold(
        backgroundColor: _fond,
        appBar: AppBar(title: const Text('updateShouldNotify')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const AbonneAuScore(),
              const SizedBox(height: 24),
              Text('build de l\'écran : $_compteurRebuildLocal fois'),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () => setState(() => _score += 50),
                child: const Text('Changer le score (notifie)'),
              ),
              const SizedBox(height: 12),
              ElevatedButton(
                onPressed: () => setState(() {
                  _fond = _fond == Colors.white
                      ? const Color(0xFFF3E5F5)
                      : Colors.white;
                }),
                child: const Text('Changer le fond (ne notifie pas)'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class AbonneAuScore extends StatelessWidget {
  const AbonneAuScore({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  >>> build de AbonneAuScore');
    final ScoreHeritee donnees = ScoreHeritee.of(context);
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Text(
          'Score partagé : ${donnees.score}',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
Console, appui sur "Changer le score (notifie)" :
updateShouldNotify : 0 -> 50 => true
  >>> build de AbonneAuScore

Console, appui sur "Changer le fond (ne notifie pas)" :
updateShouldNotify : 50 -> 50 => false
(pas de build de AbonneAuScore)
```

La démonstration est nette. Le second bouton reconstruit tout l'écran (le compteur
`build de l'écran` augmente), mais `AbonneAuScore` n'est pas reconstruit, parce que
`updateShouldNotify` a renvoyé `false`.

---

## 52.11.2 — Les deux erreurs sur `updateShouldNotify`

**Erreur 1 : renvoyer toujours `true`.**

```dart
@override
bool updateShouldNotify(ScoreHeritee oldWidget) => true; // paresseux
```

Cela fonctionne, mais annule tout le bénéfice : chaque reconstruction du parent
reconstruit tous les abonnés, même quand la valeur n'a pas bougé.

**Erreur 2 : renvoyer toujours `false`.**

```dart
@override
bool updateShouldNotify(ScoreHeritee oldWidget) => false; // bug silencieux
```

Cela ne provoque aucune erreur, aucun avertissement. Simplement, l'affichage ne se
met **jamais** à jour. C'est un des bugs les plus déroutants de Flutter : le
`setState` fonctionne, la valeur change bien en mémoire, mais l'écran reste figé.

**Erreur 3, plus subtile : comparer des objets mutables.**

```dart
// Piège : si vous MODIFIEZ la liste au lieu d'en créer une nouvelle,
// oldWidget.inventaire et inventaire sont le MÊME objet.
// La comparaison renvoie donc toujours false.
@override
bool updateShouldNotify(InventaireHerite oldWidget) {
  return inventaire != oldWidget.inventaire;
}
```

Nous reviendrons sur ce piège en 52.29 : c'est exactement la même erreur qui frappe
les utilisateurs de `provider` quand ils font `liste.add(...)` sans créer de
nouvelle liste.

---

## 52.12 — Pourquoi `Theme.of(context)` fonctionne (rappel chapitre 51)

Vous utilisez `InheritedWidget` depuis le chapitre 44 sans le savoir. Toutes les
écritures suivantes en sont des applications directes :

```dart
Theme.of(context).colorScheme.primary
MediaQuery.of(context).size.width
Navigator.of(context).push(...)
ScaffoldMessenger.of(context).showSnackBar(...)
DefaultTextStyle.of(context).style
Directionality.of(context)
```

Le suffixe `.of(context)` est **la signature visuelle** d'un accès à un
`InheritedWidget`. Chaque fois que vous voyez `.of(context)`, dites-vous :

> « Ce widget remonte l'arbre pour trouver un ancêtre d'un type précis. »

---

## 52.12.1 — Le chemin exact de `Theme.of(context)`

Voici ce qui se passe réellement quand vous écrivez `Theme.of(context)` :

```text
  1. MaterialApp construit un widget Theme quelque part au-dessus de vous.

  2. Theme, en interne, construit un _InheritedTheme, qui est un
     InheritedWidget portant un objet ThemeData.

  3. Theme.of(context) appelle (en simplifiant) :
        context.dependOnInheritedWidgetOfExactType<_InheritedTheme>()

  4. Votre widget est désormais ABONNÉ au thème.

  5. Si vous changez de thème (mode sombre, chapitre 51), une nouvelle
     instance de _InheritedTheme est créée, updateShouldNotify renvoie true,
     et TOUS les widgets qui ont appelé Theme.of se reconstruisent.
```

C'est pour cela que basculer en mode sombre met à jour l'application entière sans
que vous ayez écrit une seule ligne de propagation.

---

## 52.12.2 — La conséquence pratique à connaître

Ce mécanisme a un effet de bord que beaucoup de développeurs subissent sans le
comprendre : **appeler `Theme.of(context)` abonne votre widget au thème**.

```dart
@override
Widget build(BuildContext context) {
  // Ce widget sera reconstruit à CHAQUE changement de thème,
  // même s'il n'affiche qu'un texte fixe.
  final Color couleur = Theme.of(context).colorScheme.primary;
  return Text('Bonjour', style: TextStyle(color: couleur));
}
```

C'est en général souhaitable. Mais si vous appelez `MediaQuery.of(context)` pour
lire seulement la largeur, votre widget sera reconstruit à **chaque** changement de
`MediaQuery` — y compris l'ouverture du clavier, qui modifie `viewInsets`.

Flutter fournit pour cela des accesseurs ciblés :

```dart
// Abonnement à TOUT le MediaQuery : beaucoup de reconstructions.
final double largeur = MediaQuery.of(context).size.width;

// Abonnement à la SEULE taille : beaucoup moins de reconstructions.
final double largeurCiblee = MediaQuery.sizeOf(context).width;
```

Retenez ce principe, il reviendra en 52.22 avec `context.select` : **plus l'abonnement
est étroit, moins il y a de reconstructions**.

---

## 52.13 — Les limites de l'`InheritedWidget` écrit à la main

Notre `ScoreHeritee` de 52.9 fonctionne. Pourquoi ne pas s'arrêter là ?

Parce qu'il souffre de quatre limites sérieuses.

**Limite 1 : il ne stocke pas l'état, il le transporte.**

L'état vit toujours dans un `State` au-dessus. Vous devez donc écrire **deux**
classes : le `StatefulWidget` propriétaire et l'`InheritedWidget` transporteur. Cela
fait beaucoup de code pour partager un entier.

**Limite 2 : les descendants ne peuvent pas le modifier.**

`ScoreHeritee.of(context).score = 150` est impossible : le champ est `final`. Pour
permettre la modification, il faut exposer aussi les callbacks :

```dart
class ScoreHeritee extends InheritedWidget {
  const ScoreHeritee({
    super.key,
    required this.score,
    required this.onRamasse,   // il faut ajouter ça
    required super.child,
  });

  final int score;
  final VoidCallback onRamasse;
  // ...
}
```

Le problème du point 3 de notre cahier des charges (52.7.3) n'est donc résolu qu'à
moitié : on ne fore plus, mais il faut toujours écrire et maintenir les callbacks.

**Limite 3 : tout ou rien.**

`updateShouldNotify` renvoie un seul booléen pour **tous** les abonnés. Si votre
`InheritedWidget` porte cinq champs, un widget qui n'utilise que `score` sera quand
même reconstruit quand `vies` change.

```text
  ScoreHeritee { score, vies, energie, niveau, or }

  energie change  ->  updateShouldNotify renvoie true
                  ->  AfficheScore reconstruit  (inutile)
                  ->  AfficheVies reconstruit   (inutile)
                  ->  AfficheOr reconstruit     (inutile)
                  ->  AfficheEnergie reconstruit (utile)
```

Il existe bien `InheritedModel`, une variante qui permet de découper la notification
en « aspects », mais son écriture est nettement plus technique.

**Limite 4 : il ne survit pas à la navigation, sauf à être placé très haut.**

Pour qu'un état soit accessible depuis deux routes différentes, l'`InheritedWidget`
doit être placé **au-dessus** du `MaterialApp`, ou au moins au-dessus du
`Navigator`. C'est faisable, mais on se retrouve alors avec un `StatefulWidget`
géant qui détient l'état de toute l'application.

---

## 52.13.1 — Le tableau des besoins non couverts

| Besoin | `InheritedWidget` à la main | Ce qu'il nous faudrait |
| --- | --- | --- |
| Stocker l'état | non (il faut un `State` en plus) | un objet qui détient ses données |
| Le modifier depuis un descendant | non (callbacks obligatoires) | des méthodes appelables directement |
| Notifier finement | non (tout ou rien) | un abonnement par champ |
| Libérer les ressources | à la main | automatique |
| Créer paresseusement | non | à la première utilisation |

La suite du chapitre comble ces cinq cases. D'abord avec `ChangeNotifier`, qui est
un objet **capable de détenir des données et de prévenir des écouteurs**. Puis avec
`provider`, qui fait le lien entre un `ChangeNotifier` et l'arbre de widgets.

---

## 52.14 — `ChangeNotifier`

`ChangeNotifier` est une classe fournie par `package:flutter/foundation.dart`
(réexportée par `material.dart`). Elle implémente l'interface `Listenable`.

Son rôle est simple : **tenir une liste d'écouteurs et pouvoir les appeler tous**.

Ses membres publics, vérifiés dans l'API officielle :

| Membre | Signature | Rôle |
| --- | --- | --- |
| `hasListeners` | `bool` | Y a-t-il des écouteurs enregistrés ? |
| `addListener` | `void addListener(VoidCallback listener)` | Enregistrer une fonction |
| `removeListener` | `void removeListener(VoidCallback listener)` | Retirer une fonction |
| `notifyListeners` | `void notifyListeners()` | Appeler tous les écouteurs |
| `dispose` | `void dispose()` | Libérer les ressources |

C'est tout. Cinq membres. `ChangeNotifier` ne sait **rien** de Flutter, des widgets
ou de l'affichage. C'est un objet Dart ordinaire, ce qui aura une conséquence très
agréable en 52.31 : il se teste sans interface.

---

## 52.14.1 — Écrire sa première classe d'état

Le patron est toujours le même, en quatre points :

```text
  1. class MonEtat extends ChangeNotifier
  2. des champs PRIVÉS (avec _)
  3. des accesseurs (getters) PUBLICS en lecture seule
  4. des MÉTHODES qui modifient, puis appellent notifyListeners()
```

Appliquons-le à un joueur de jeu vidéo.

```dart
class Joueur extends ChangeNotifier {
  // 2. Champs privés : personne ne peut les modifier de l'extérieur.
  int _score = 0;
  int _vies = 3;
  final List<String> _inventaire = <String>[];

  // 3. Accesseurs publics en lecture seule.
  int get score => _score;
  int get vies => _vies;
  List<String> get inventaire => List<String>.unmodifiable(_inventaire);
  bool get estMort => _vies <= 0;

  // 4. Méthodes de modification.
  void ramasserPiece(int valeur) {
    _score += valeur;
    notifyListeners();
  }

  void perdreUneVie() {
    if (_vies > 0) {
      _vies--;
      notifyListeners();
    }
  }

  void ajouterObjet(String objet) {
    _inventaire.add(objet);
    notifyListeners();
  }

  void reinitialiser() {
    _score = 0;
    _vies = 3;
    _inventaire.clear();
    notifyListeners();
  }
}
```

Trois détails importants dans ce code.

**Les champs sont privés.** Le tiret bas est essentiel : sans lui, n'importe qui
pourrait écrire `joueur.score = 9999` sans déclencher la moindre notification, et
l'interface resterait figée.

**`inventaire` renvoie une copie non modifiable.** `List.unmodifiable` (chapitre 06)
garantit qu'un appelant ne peut pas faire `joueur.inventaire.add('triche')`. La
seule porte d'entrée est la méthode `ajouterObjet`.

**`estMort` est un accesseur calculé.** Il n'est pas stocké. Il ne peut donc jamais
se désynchroniser du nombre de vies. C'est un réflexe à prendre : **tout ce qui peut
être calculé ne doit pas être stocké**.

---

## 52.14.2 — Utiliser un `ChangeNotifier` sans aucune dépendance

`ChangeNotifier` fait partie de Flutter. Vous pouvez l'utiliser dès maintenant, sans
installer quoi que ce soit, en vous abonnant à la main dans un `State`.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationNotifier());
}

// ─────────────────────────────────────────────────────────────
// LE MODÈLE : du Dart pur, aucun widget.
// ─────────────────────────────────────────────────────────────
class Joueur extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;

  int get score => _score;
  int get vies => _vies;
  bool get estMort => _vies <= 0;

  void ramasserPiece(int valeur) {
    _score += valeur;
    notifyListeners();
  }

  void perdreUneVie() {
    if (_vies > 0) {
      _vies--;
      notifyListeners();
    }
  }
}

// ─────────────────────────────────────────────────────────────
// L'INTERFACE
// ─────────────────────────────────────────────────────────────
class ApplicationNotifier extends StatelessWidget {
  const ApplicationNotifier({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      home: const EcranJoueur(),
    );
  }
}

class EcranJoueur extends StatefulWidget {
  const EcranJoueur({super.key});

  @override
  State<EcranJoueur> createState() => _EcranJoueurState();
}

class _EcranJoueurState extends State<EcranJoueur> {
  // On crée le modèle une seule fois.
  final Joueur _joueur = Joueur();

  @override
  void initState() {
    super.initState();
    // On s'abonne : à chaque notification, on redessine.
    _joueur.addListener(_surChangement);
  }

  void _surChangement() {
    debugPrint('Le joueur a changé : score=${_joueur.score}');
    setState(() {});
  }

  @override
  void dispose() {
    // OBLIGATOIRE : on se désabonne, puis on libère le modèle.
    _joueur.removeListener(_surChangement);
    _joueur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ChangeNotifier à la main')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'Score : ${_joueur.score}',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            Text(
              'Vies : ${_joueur.vies}',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            if (_joueur.estMort)
              const Padding(
                padding: EdgeInsets.all(16),
                child: Text(
                  'PARTIE TERMINÉE',
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                    color: Colors.red,
                  ),
                ),
              ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () => _joueur.ramasserPiece(50),
              child: const Text('Ramasser 50 points'),
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: _joueur.perdreUneVie,
              child: const Text('Perdre une vie'),
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
Score : 100
Vies : 1
[ Ramasser 50 points ]
[ Perdre une vie ]

Console :
Le joueur a changé : score=50
Le joueur a changé : score=100
Le joueur a changé : score=100
Le joueur a changé : score=100

Après deux pertes de vie de plus :
Vies : 0
PARTIE TERMINÉE
```

Notez le gain, même sans `provider` : la logique du jeu (`Joueur`) est complètement
séparée de l'interface. Vous pouvez la tester, la réutiliser, la déplacer dans un
autre fichier.

---

## 52.14.3 — Le contrat de libération

Un `ChangeNotifier` est un objet vivant. Il **doit** être libéré, sinon vous créez
une fuite mémoire.

```text
  Cycle de vie complet :

     création       final Joueur _joueur = Joueur();
        │
     abonnement     _joueur.addListener(_surChangement);
        │
     utilisation    _joueur.ramasserPiece(50);   -> notifyListeners()
        │
     désabonnement  _joueur.removeListener(_surChangement);
        │
     libération     _joueur.dispose();
```

Si vous oubliez `removeListener` **et** que le notifier survit au widget, l'écouteur
continue de pointer vers un `State` mort. Le premier `setState()` déclenché lèvera
alors :

```text
setState() called after dispose(): _EcranJoueurState#a1b2c(lifecycle state:
defunct, not mounted)
```

Et si vous utilisez un notifier après l'avoir libéré :

```text
A Joueur was used after being disposed.
Once you have called dispose() on a Joueur, it can no longer be used.
```

Ce dernier message vient de `ChangeNotifier.debugAssertNotDisposed`, présent
uniquement en mode debug. Retenez-le : il indique toujours un `dispose()` appelé
trop tôt.

---

## 52.15 — `notifyListeners()`

`notifyListeners()` est le déclencheur. Sans lui, rien ne se passe.

```text
void notifyListeners()
```

Son fonctionnement est direct : elle parcourt la liste des écouteurs enregistrés et
appelle chacun d'eux, de façon **synchrone**, dans l'ordre d'enregistrement.

```text
  joueur.ramasserPiece(50)
     │
     ├─ _score += 50                    (la donnée change)
     │
     └─ notifyListeners()
           ├─ écouteur 1 : setState() du HUD
           ├─ écouteur 2 : setState() de l'inventaire
           └─ écouteur 3 : journalisation
```

---

## 52.15.1 — Les quatre règles de `notifyListeners()`

**Règle 1 : l'appeler APRÈS la modification, jamais avant.**

```dart
// FAUX : les écouteurs lisent l'ancienne valeur.
void ramasserPiece(int valeur) {
  notifyListeners();
  _score += valeur;
}

// CORRECT
void ramasserPiece(int valeur) {
  _score += valeur;
  notifyListeners();
}
```

**Règle 2 : un seul appel par méthode, à la fin.**

```dart
// INEFFICACE : trois notifications, donc trois séries de reconstructions.
void finDeNiveau() {
  _score += 1000;
  notifyListeners();
  _niveau++;
  notifyListeners();
  _vies++;
  notifyListeners();
}

// CORRECT : une seule notification pour un changement cohérent.
void finDeNiveau() {
  _score += 1000;
  _niveau++;
  _vies++;
  notifyListeners();
}
```

**Règle 3 : ne pas notifier si rien n'a changé.**

```dart
void perdreUneVie() {
  if (_vies <= 0) {
    return; // rien n'a changé, on ne notifie pas
  }
  _vies--;
  notifyListeners();
}
```

**Règle 4 : ne jamais l'appeler depuis un accesseur.**

```dart
// CATASTROPHE : boucle infinie.
// build() lit score -> notifie -> reconstruit -> lit score -> ...
int get score {
  notifyListeners();
  return _score;
}
```

Un accesseur **lit**, il ne notifie jamais.

---

## 52.15.2 — La notification pendant une phase de construction

Voici l'erreur la plus fréquente avec `ChangeNotifier`, et la plus mal comprise.

```dart
@override
Widget build(BuildContext context) {
  // ERREUR : on modifie l'état PENDANT la construction de l'arbre.
  _joueur.ramasserPiece(50);
  return Text('Score : ${_joueur.score}');
}
```

**Message réel :**

```text
setState() or markNeedsBuild() called during build.

This ChangeNotifier widget cannot be marked as needing to build because the
framework is already in the process of building widgets.
```

La cause est structurelle : Flutter construit l'arbre en une passe. Marquer un
widget « à reconstruire » alors que la passe est en cours créerait une boucle.

Les deux solutions :

```dart
// Solution 1 : déplacer l'appel dans un callback (le cas normal).
ElevatedButton(
  onPressed: () => _joueur.ramasserPiece(50),
  child: const Text('Ramasser'),
)

// Solution 2 : différer après la construction (cas rare, chargement initial).
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((Duration _) {
    _joueur.chargerDonneesInitiales();
  });
}
```

---

## 52.16 — `ValueNotifier` et `ValueListenableBuilder` : la solution sans dépendance

Avant d'installer quoi que ce soit, sachez qu'une grande partie des besoins de
partage d'état se règle avec **deux classes déjà présentes dans Flutter**.

`ValueNotifier<T>` est un `ChangeNotifier` qui enveloppe **une seule valeur** :

```dart
final ValueNotifier<int> score = ValueNotifier<int>(0);

score.value = 150;          // affecte ET notifie automatiquement
debugPrint('${score.value}'); // 150
```

Le point clé : l'affectation de `value` appelle `notifyListeners()` **toute seule**,
mais seulement si la nouvelle valeur est différente de l'ancienne (comparaison avec
`==`).

---

## 52.16.1 — `ValueListenableBuilder`

Pour afficher un `ValueNotifier`, Flutter fournit un widget dédié. Sa signature
officielle :

```text
const ValueListenableBuilder({
  Key? key,
  required ValueListenable<T> valueListenable,
  required ValueWidgetBuilder<T> builder,
  Widget? child,
})
```

Et le type du constructeur :

```text
typedef ValueWidgetBuilder<T> =
    Widget Function(BuildContext context, T value, Widget? child);
```

Ce widget s'abonne au `ValueListenable`, et reconstruit **uniquement son
`builder`** à chaque changement de valeur. Le reste de l'écran ne bouge pas.

Voici un programme complet.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationValueNotifier());
}

class ApplicationValueNotifier extends StatelessWidget {
  const ApplicationValueNotifier({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.amber),
        useMaterial3: true,
      ),
      home: const EcranValeur(),
    );
  }
}

class EcranValeur extends StatefulWidget {
  const EcranValeur({super.key});

  @override
  State<EcranValeur> createState() => _EcranValeurState();
}

class _EcranValeurState extends State<EcranValeur> {
  final ValueNotifier<int> _score = ValueNotifier<int>(0);
  final ValueNotifier<int> _vies = ValueNotifier<int>(3);

  @override
  void dispose() {
    _score.dispose();
    _vies.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build de EcranValeur (une seule fois attendue)');
    return Scaffold(
      appBar: AppBar(title: const Text('ValueNotifier')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            // Ce bloc-ci ne se reconstruit QUE si _score change.
            ValueListenableBuilder<int>(
              valueListenable: _score,
              builder: (BuildContext context, int valeur, Widget? enfant) {
                debugPrint('  build du bloc SCORE');
                return Row(
                  mainAxisSize: MainAxisSize.min,
                  children: <Widget>[
                    enfant!, // l'icône, construite une seule fois
                    const SizedBox(width: 8),
                    Text('Score : $valeur', style: const TextStyle(fontSize: 28)),
                  ],
                );
              },
              // child est construit UNE FOIS et réutilisé à chaque rebuild.
              child: const Icon(Icons.star, size: 32, color: Colors.amber),
            ),
            const SizedBox(height: 16),
            // Ce bloc-là ne se reconstruit QUE si _vies change.
            ValueListenableBuilder<int>(
              valueListenable: _vies,
              builder: (BuildContext context, int valeur, Widget? enfant) {
                debugPrint('  build du bloc VIES');
                return Text(
                  'Vies : $valeur',
                  style: const TextStyle(fontSize: 28),
                );
              },
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              // Pas de setState : on modifie directement la valeur.
              onPressed: () => _score.value += 50,
              child: const Text('Ramasser 50 points'),
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: () {
                if (_vies.value > 0) _vies.value--;
              },
              child: const Text('Perdre une vie'),
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
Console au démarrage :
build de EcranValeur (une seule fois attendue)
  build du bloc SCORE
  build du bloc VIES

Console après trois appuis sur "Ramasser 50 points" :
  build du bloc SCORE
  build du bloc SCORE
  build du bloc SCORE

Console après un appui sur "Perdre une vie" :
  build du bloc VIES
```

C'est un résultat remarquable : `build de EcranValeur` n'apparaît **qu'une seule
fois**. L'écran entier n'est jamais reconstruit. Seul le petit bloc concerné l'est.

Comparez avec la version `setState` de 52.4, où chaque appui reconstruisait tout.

---

## 52.16.2 — Le paramètre `child` : l'optimisation gratuite

Regardez ce détail du code précédent :

```dart
ValueListenableBuilder<int>(
  valueListenable: _score,
  builder: (BuildContext context, int valeur, Widget? enfant) {
    return Row(children: <Widget>[enfant!, Text('Score : $valeur')]);
  },
  child: const Icon(Icons.star, size: 32, color: Colors.amber),
)
```

Le widget passé à `child:` est construit **une seule fois**, en dehors du `builder`.
Il est ensuite transmis au `builder` sous le nom `enfant` à chaque reconstruction.

```text
  SANS child :                       AVEC child :

  reconstruction 1 : Row + Icon      reconstruction 1 : Row (Icon réutilisée)
  reconstruction 2 : Row + Icon      reconstruction 2 : Row (Icon réutilisée)
  reconstruction 3 : Row + Icon      reconstruction 3 : Row (Icon réutilisée)
```

Sur une icône, le gain est négligeable. Sur un sous-arbre lourd — une image, une
liste, une carte — il devient significatif. Ce paramètre `child` existe sur
`ValueListenableBuilder`, `ListenableBuilder`, `AnimatedBuilder` et `Consumer` de
`provider` : c'est toujours la même optimisation.

---

## 52.16.3 — Le piège du `ValueNotifier` de liste

Attention à ce cas, il piège tout le monde.

```dart
final ValueNotifier<List<String>> inventaire =
    ValueNotifier<List<String>>(<String>[]);

// NE MARCHE PAS : on modifie la liste, mais value ne change pas d'identité.
inventaire.value.add('Potion');   // aucune notification !
```

`ValueNotifier` compare l'ancienne et la nouvelle valeur avec `==`. Pour une liste,
`==` compare les **références**, pas le contenu. Comme c'est le même objet, aucune
notification n'est émise.

La solution est d'affecter une **nouvelle liste** :

```dart
// CORRECT : nouvelle liste, donc nouvelle référence, donc notification.
inventaire.value = <String>[...inventaire.value, 'Potion'];
```

L'opérateur de propagation `...` (chapitre 06) crée bien une nouvelle liste.

> **Règle générale :** `ValueNotifier` convient parfaitement aux valeurs simples
> (`int`, `String`, `bool`, `double`, `enum`). Pour un objet complexe ou une
> collection, préférez `ChangeNotifier`, où **vous** décidez quand notifier.

---

## 52.17 — `ListenableBuilder`

`ValueListenableBuilder` ne fonctionne qu'avec un `ValueListenable`. Pour un
`ChangeNotifier` quelconque, Flutter fournit `ListenableBuilder`.

Sa signature officielle :

```text
const ListenableBuilder({
  Key? key,
  required Listenable listenable,
  required TransitionBuilder builder,
  Widget? child,
})
```

Avec :

```text
typedef TransitionBuilder = Widget Function(BuildContext context, Widget? child);
```

Notez la différence avec `ValueListenableBuilder` : le `builder` ne reçoit **pas**
de valeur. C'est logique, un `Listenable` quelconque n'a pas de « valeur » unique.
Vous lisez donc directement les propriétés de votre objet.

---

## 52.17.1 — Un exemple complet avec `ListenableBuilder`

Ce programme n'utilise **aucune dépendance externe**. C'est déjà une architecture
propre : un modèle séparé, des reconstructions ciblées.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationListenable());
}

// ─────────────────────────────────────────────────────────────
// LE MODÈLE
// ─────────────────────────────────────────────────────────────
class Joueur extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;
  final List<String> _inventaire = <String>[];

  int get score => _score;
  int get vies => _vies;
  List<String> get inventaire => List<String>.unmodifiable(_inventaire);

  void ramasserPiece(int valeur) {
    _score += valeur;
    notifyListeners();
  }

  void perdreUneVie() {
    if (_vies <= 0) return;
    _vies--;
    notifyListeners();
  }

  void ajouterObjet(String objet) {
    _inventaire.add(objet);
    notifyListeners();
  }
}

// ─────────────────────────────────────────────────────────────
// L'INTERFACE
// ─────────────────────────────────────────────────────────────
class ApplicationListenable extends StatelessWidget {
  const ApplicationListenable({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.cyan),
        useMaterial3: true,
      ),
      home: const EcranListenable(),
    );
  }
}

class EcranListenable extends StatefulWidget {
  const EcranListenable({super.key});

  @override
  State<EcranListenable> createState() => _EcranListenableState();
}

class _EcranListenableState extends State<EcranListenable> {
  final Joueur _joueur = Joueur();

  @override
  void dispose() {
    _joueur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build de EcranListenable');
    return Scaffold(
      appBar: AppBar(title: const Text('ListenableBuilder')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            // Bloc 1 : le HUD, reconstruit à chaque notification.
            ListenableBuilder(
              listenable: _joueur,
              builder: (BuildContext context, Widget? enfant) {
                debugPrint('  build du HUD');
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      children: <Widget>[
                        Text('Score : ${_joueur.score}',
                            style: const TextStyle(fontSize: 24)),
                        Text('Vies : ${_joueur.vies}',
                            style: const TextStyle(fontSize: 24)),
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            // Bloc 2 : l'inventaire, reconstruit lui aussi.
            Expanded(
              child: ListenableBuilder(
                listenable: _joueur,
                builder: (BuildContext context, Widget? enfant) {
                  debugPrint('  build de l\'inventaire');
                  final List<String> objets = _joueur.inventaire;
                  if (objets.isEmpty) {
                    return const Center(child: Text('Inventaire vide'));
                  }
                  return ListView.builder(
                    itemCount: objets.length,
                    itemBuilder: (BuildContext context, int index) {
                      return ListTile(
                        leading: const Icon(Icons.inventory_2),
                        title: Text(objets[index]),
                      );
                    },
                  );
                },
              ),
            ),
            // Bloc 3 : les boutons, JAMAIS reconstruits.
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                ElevatedButton(
                  onPressed: () => _joueur.ramasserPiece(50),
                  child: const Text('+50'),
                ),
                ElevatedButton(
                  onPressed: _joueur.perdreUneVie,
                  child: const Text('-1 vie'),
                ),
                ElevatedButton(
                  onPressed: () => _joueur.ajouterObjet('Potion'),
                  child: const Text('Potion'),
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
Console au démarrage :
build de EcranListenable
  build du HUD
  build de l'inventaire

Console après un appui sur "+50" :
  build du HUD
  build de l'inventaire
```

`build de EcranListenable` n'apparaît qu'une fois. Les boutons ne sont jamais
reconstruits.

Remarquez toutefois que **les deux blocs** se reconstruisent, alors que seul le
score a changé. `ListenableBuilder` s'abonne à l'objet entier, pas à un champ. C'est
la limite 3 de 52.13, à nouveau. `context.select` (52.22) la résoudra.

---

## 52.17.2 — Ce qu'il reste à résoudre

Faisons le point. Avec `ChangeNotifier` + `ListenableBuilder`, sans aucune
dépendance, nous avons déjà :

```text
  [OK]  la donnée vit à un seul endroit          (l'objet Joueur)
  [OK]  les descendants la modifient directement (joueur.ramasserPiece)
  [OK]  seuls les blocs concernés se reconstruisent
  [--]  la donnée n'est PAS accessible sans la passer par constructeur
  [--]  elle ne survit PAS à la navigation entre écrans
  [--]  l'abonnement porte sur l'objet entier, pas sur un champ
```

Les deux premières croix sont exactement le problème du forage (52.6) : pour donner
`_joueur` à un widget profond, il faut encore le passer de constructeur en
constructeur.

Il nous manque donc **le pont entre l'objet `ChangeNotifier` et l'arbre de
widgets** : quelque chose qui pose l'objet dans l'arbre comme le fait un
`InheritedWidget`, et qui s'abonne automatiquement.

Ce pont s'appelle `provider`.

---

## 52.18 — Le paquet `provider` : installation

`provider` est un paquet écrit par Rémi Rousselet, publié par
`dash-overflow.net`, sous licence MIT. Ce n'est pas un framework : c'est une
**enveloppe autour d'`InheritedWidget`** qui automatise ce que nous avons écrit à la
main en 52.9.

Installation, depuis la racine du projet :

```text
flutter pub add provider
```

La commande ajoute la ligne adéquate dans `pubspec.yaml` et lance `flutter pub get`.

```text
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.5
```

> **Ne figez jamais un numéro de version à la main.** Utilisez
> `flutter pub add provider` : le gestionnaire écrira la version courante au moment
> de l'installation. Au moment de la rédaction de ce chapitre, la dernière version
> publiée est `6.1.5+1`.

Import dans le code :

```dart
import 'package:provider/provider.dart';
```

---

## 52.18.1 — Ce que `provider` apporte exactement

| Fonction | Sans `provider` | Avec `provider` |
| --- | --- | --- |
| Poser un objet dans l'arbre | écrire un `InheritedWidget` | `ChangeNotifierProvider` |
| S'abonner | `addListener` + `setState` | `context.watch<T>()` |
| Lire sans s'abonner | `getElementForInherited...` | `context.read<T>()` |
| S'abonner à un seul champ | impossible simplement | `context.select<T, R>()` |
| Libérer le notifier | `dispose()` à la main | automatique |
| Créer paresseusement | à la main | par défaut |
| Erreur si absent | `assert` à écrire | `ProviderNotFoundException` |

`provider` n'invente rien. Il supprime du code répétitif et rend les erreurs
lisibles.

---

## 52.19 — `ChangeNotifierProvider`

C'est le provider que vous utiliserez dans 90 % des cas. Sa documentation officielle
le décrit ainsi :

> « Listens to a ChangeNotifier, expose it to its descendants and rebuilds
> dependents whenever ChangeNotifier.notifyListeners is called. »

Ses deux constructeurs, vérifiés dans les sources :

```text
ChangeNotifierProvider({
  Key? key,
  required Create<T> create,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
})

ChangeNotifierProvider.value({
  Key? key,
  required T value,
  TransitionBuilder? builder,
  Widget? child,
})
```

Le paramètre `create` est une fonction qui **construit** l'objet :

```dart
ChangeNotifierProvider<Joueur>(
  create: (BuildContext context) => Joueur(),
  child: const MonApplication(),
)
```

Trois comportements offerts gratuitement :

```text
  1. l'objet est créé PARESSEUSEMENT, à la première lecture (lazy: true par défaut)
  2. dispose() est appelé automatiquement quand le provider quitte l'arbre
  3. les widgets abonnés sont reconstruits à chaque notifyListeners()
```

---

## 52.19.1 — Où placer le provider

La règle est simple : **le plus bas possible, mais au-dessus de tous les widgets qui
en ont besoin**.

```text
  Pour un état global (utilisateur, thème, panier) :

     runApp(
       ChangeNotifierProvider(          <- AU-DESSUS de MaterialApp
         create: (_) => Joueur(),
         child: const MonApp(),
       ),
     );

  Pour un état limité à un écran et ses enfants :

     Navigator.push(context, MaterialPageRoute(
       builder: (_) => ChangeNotifierProvider(   <- dans la route
         create: (_) => FiltreRecherche(),
         child: const EcranRecherche(),
       ),
     ));
```

Placer le provider au-dessus de `MaterialApp` garantit que l'état survit à toutes
les navigations. Le placer dans une route le limite à cette route : c'est le
comportement « scopé » évoqué dans le message d'erreur de `ProviderNotFoundException`.

---

## 52.19.2 — Premier programme complet avec `provider`

Ce programme reprend l'exemple de 52.6 — cinq niveaux de profondeur — mais sans
aucun forage.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Joueur>(
      create: (BuildContext context) => Joueur(),
      child: const ApplicationProvider(),
    ),
  );
}

// ─────────────────────────────────────────────────────────────
// LE MODÈLE : Dart pur, aucune référence à Flutter hors ChangeNotifier.
// ─────────────────────────────────────────────────────────────
class Joueur extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;

  int get score => _score;
  int get vies => _vies;
  bool get estMort => _vies <= 0;

  void ramasserPiece(int valeur) {
    _score += valeur;
    notifyListeners();
  }

  void perdreUneVie() {
    if (_vies <= 0) return;
    _vies--;
    notifyListeners();
  }
}

// ─────────────────────────────────────────────────────────────
// L'INTERFACE : plus AUCUN paramètre transporté.
// ─────────────────────────────────────────────────────────────
class ApplicationProvider extends StatelessWidget {
  const ApplicationProvider({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranJeu(),
    );
  }
}

class EcranJeu extends StatelessWidget {
  const EcranJeu({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build EcranJeu');
    return const Scaffold(
      body: SafeArea(child: CorpsJeu()),
    );
  }
}

class CorpsJeu extends StatelessWidget {
  const CorpsJeu({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build CorpsJeu');
    return const Column(
      children: <Widget>[
        Expanded(child: Center(child: Text('Zone de jeu'))),
        ZoneBasse(),
      ],
    );
  }
}

class ZoneBasse extends StatelessWidget {
  const ZoneBasse({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build ZoneBasse');
    return const Padding(padding: EdgeInsets.all(12), child: BarreOutils());
  }
}

class BarreOutils extends StatelessWidget {
  const BarreOutils({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BarreOutils');
    return const Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: <Widget>[HudScore(), BoutonsAction()],
    );
  }
}

// Ce widget LIT l'état : il s'abonne.
class HudScore extends StatelessWidget {
  const HudScore({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  build HudScore (abonné)');
    final Joueur joueur = context.watch<Joueur>();
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Text('Score : ${joueur.score}'),
        Text('Vies  : ${joueur.vies}'),
        if (joueur.estMort)
          const Text('PARTIE TERMINÉE',
              style: TextStyle(color: Colors.red, fontWeight: FontWeight.bold)),
      ],
    );
  }
}

// Ce widget MODIFIE l'état : il ne s'abonne pas.
class BoutonsAction extends StatelessWidget {
  const BoutonsAction({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  build BoutonsAction (non abonné)');
    return Row(
      children: <Widget>[
        IconButton(
          tooltip: 'Ramasser',
          icon: const Icon(Icons.star),
          onPressed: () => context.read<Joueur>().ramasserPiece(50),
        ),
        IconButton(
          tooltip: 'Perdre une vie',
          icon: const Icon(Icons.heart_broken),
          onPressed: () => context.read<Joueur>().perdreUneVie(),
        ),
      ],
    );
  }
}
```

**Résultat :**

```text
Console au démarrage :
build EcranJeu
build CorpsJeu
build ZoneBasse
build BarreOutils
  build HudScore (abonné)
  build BoutonsAction (non abonné)

Console après un appui sur l'étoile :
  build HudScore (abonné)
```

Une seule ligne. Un seul widget reconstruit sur six. Comparez avec les 48 lignes de
tuyauterie de 52.6 : elles ont toutes disparu.

---

## 52.20 — `context.watch<T>()`

`watch` est une méthode d'extension sur `BuildContext`, fournie par l'extension
`WatchContext` du paquet.

```text
T watch<T>()
```

Elle fait deux choses, exactement comme `dependOnInheritedWidgetOfExactType` :

```text
  1. elle renvoie l'objet de type T fourni par le provider le plus proche ;
  2. elle ABONNE le widget appelant : il sera reconstruit à chaque notification.
```

```dart
@override
Widget build(BuildContext context) {
  final Joueur joueur = context.watch<Joueur>();
  return Text('Score : ${joueur.score}');
}
```

Trois règles :

**Elle ne s'utilise que dans `build()`.** Appelée depuis `initState()` ou depuis un
`onPressed`, elle lève une erreur ou crée un abonnement absurde.

**Elle reconstruit tout le `build()` du widget appelant.** Placez-la donc dans le
plus petit widget possible.

**Si aucun provider n'est trouvé, elle lève `ProviderNotFoundException`** (voir
52.21.2).

---

## 52.20.1 — L'équivalent historique : `Provider.of<T>(context)`

Avant l'arrivée des extensions, on écrivait :

```dart
// Ancienne syntaxe, strictement équivalente à context.watch<Joueur>()
final Joueur joueur = Provider.of<Joueur>(context);

// Strictement équivalente à context.read<Joueur>()
final Joueur joueur = Provider.of<Joueur>(context, listen: false);
```

Vous rencontrerez cette forme dans beaucoup de code et de tutoriels. Elle
fonctionne toujours. Préférez cependant `watch` et `read` : le nom dit ce que le
code fait, alors que `listen: false` est facile à oublier.

---

## 52.21 — `context.read<T>()`

```text
T read<T>()
```

Son implémentation réelle, dans les sources de `provider`, tient en une ligne :

```dart
T read<T>() {
  return Provider.of<T>(this, listen: false);
}
```

Elle **lit sans s'abonner**. Le widget appelant ne sera donc jamais reconstruit à
cause de cette lecture.

Son usage normal est dans un callback :

```dart
ElevatedButton(
  onPressed: () => context.read<Joueur>().ramasserPiece(50),
  child: const Text('Ramasser'),
)
```

Pourquoi `read` et non `watch` ici ? Parce que le bouton n'affiche pas le score. Il
n'a aucune raison de se reconstruire quand le score change. `watch` fonctionnerait,
mais provoquerait des reconstructions inutiles du bouton.

---

## 52.21.1 — L'erreur : `read` dans `build` pour afficher

```dart
// FAUX : le texte ne se mettra JAMAIS à jour.
@override
Widget build(BuildContext context) {
  final Joueur joueur = context.read<Joueur>();
  return Text('Score : ${joueur.score}');
}
```

Aucune erreur, aucun avertissement. L'écran affiche `Score : 0` et n'en bouge plus,
alors que le score augmente bien en mémoire. C'est le bug le plus fréquent chez les
débutants de `provider`.

Il existe un cas légitime de `read` dans `build` : lorsque vous passez l'objet à un
widget enfant qui, lui, ne l'affiche pas — par exemple pour brancher un callback.
Mais si le résultat apparaît à l'écran, c'est `watch`.

---

## 52.21.2 — `ProviderNotFoundException`

Si vous demandez un type qu'aucun ancêtre ne fournit, `provider` lève une exception
dédiée. En mode release, le message est court :

```text
Provider<Joueur> not found for HudScore
```

En mode debug, il est long et pédagogique. Voici son début exact :

```text
Error: Could not find the correct Provider<Joueur> above this HudScore Widget

This happens because you used a `BuildContext` that does not include the provider
of your choice. There are a few common scenarios:

- You added a new provider in your `main.dart` and performed a hot-reload.
  To fix, perform a hot-restart.

- The provider you are trying to read is in a different route.

  Providers are "scoped". So if you insert of provider inside a route, then
  other routes will not be able to access that provider.

- You used a `BuildContext` that is an ancestor of the provider you are trying to read.
```

Les trois causes sont listées par ordre de fréquence réelle. Retenez surtout la
première : **après avoir ajouté un provider, faites un hot restart, pas un hot
reload.**

---

## 52.21.3 — Le piège du `context` ancêtre

La troisième cause du message mérite une démonstration, car elle est contre-intuitive.

```dart
// FAUX : le `context` de build() est le contexte du PARENT du provider.
@override
Widget build(BuildContext context) {
  return ChangeNotifierProvider<Joueur>(
    create: (_) => Joueur(),
    child: Text('${context.watch<Joueur>().score}'), // ProviderNotFoundException
  );
}
```

Le `context` passé à `build` désigne la position **du widget courant**, donc
au-dessus du `ChangeNotifierProvider` qu'il vient de créer. La recherche remonte
vers le haut, et ne trouve rien.

Deux corrections. La première consiste à extraire un widget enfant :

```dart
return ChangeNotifierProvider<Joueur>(
  create: (_) => Joueur(),
  child: const HudScore(), // HudScore a son PROPRE context, sous le provider
);
```

La seconde utilise le paramètre `builder`, prévu exactement pour cela :

```dart
return ChangeNotifierProvider<Joueur>(
  create: (_) => Joueur(),
  builder: (BuildContext context, Widget? child) {
    // ce `context` est SOUS le provider
    return Text('${context.watch<Joueur>().score}');
  },
);
```

---

## 52.22 — `context.select<T, R>()`

C'est l'outil de précision. Sa signature, vérifiée dans les sources :

```text
R select<T, R>(R Function(T value) selector)
```

Elle s'abonne à **une valeur dérivée** de l'objet, et ne reconstruit le widget que
si cette valeur change.

```dart
// S'abonne au SEUL score. Si vies change, ce widget ne bouge pas.
final int score = context.select<Joueur, int>((Joueur j) => j.score);
return Text('Score : $score');
```

Le mécanisme : après chaque notification, `provider` rappelle votre fonction et
compare le résultat au précédent avec `==`. Si les deux valeurs sont égales, aucune
reconstruction.

```text
  joueur.perdreUneVie()  ->  notifyListeners()
      │
      ├─ widget abonné par watch    : selector inexistant  -> RECONSTRUIT
      ├─ widget select(j => j.score): 100 == 100           -> pas reconstruit
      └─ widget select(j => j.vies) : 3 != 2               -> RECONSTRUIT
```

---

## 52.22.1 — Démonstration chiffrée

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Joueur>(
      create: (_) => Joueur(),
      child: const ApplicationSelect(),
    ),
  );
}

class Joueur extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;

  int get score => _score;
  int get vies => _vies;

  void ramasserPiece() {
    _score += 50;
    notifyListeners();
  }

  void perdreUneVie() {
    if (_vies <= 0) return;
    _vies--;
    notifyListeners();
  }
}

class ApplicationSelect extends StatelessWidget {
  const ApplicationSelect({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.purple),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('watch contre select')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const BlocWatch(),
              const SizedBox(height: 12),
              const BlocSelectScore(),
              const SizedBox(height: 12),
              const BlocSelectVies(),
              const SizedBox(height: 32),
              Builder(
                builder: (BuildContext context) => Column(
                  children: <Widget>[
                    ElevatedButton(
                      onPressed: () => context.read<Joueur>().ramasserPiece(),
                      child: const Text('Ramasser (change le score)'),
                    ),
                    const SizedBox(height: 8),
                    ElevatedButton(
                      onPressed: () => context.read<Joueur>().perdreUneVie(),
                      child: const Text('Perdre une vie (change les vies)'),
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

class BlocWatch extends StatelessWidget {
  const BlocWatch({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocWatch');
    final Joueur j = context.watch<Joueur>();
    return Text('watch  -> score ${j.score}, vies ${j.vies}');
  }
}

class BlocSelectScore extends StatelessWidget {
  const BlocSelectScore({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocSelectScore');
    final int score = context.select<Joueur, int>((Joueur j) => j.score);
    return Text('select score -> $score');
  }
}

class BlocSelectVies extends StatelessWidget {
  const BlocSelectVies({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocSelectVies');
    final int vies = context.select<Joueur, int>((Joueur j) => j.vies);
    return Text('select vies -> $vies');
  }
}
```

**Résultat :**

```text
Console au démarrage :
build BlocWatch
build BlocSelectScore
build BlocSelectVies

Console après "Ramasser (change le score)" :
build BlocWatch
build BlocSelectScore

Console après "Perdre une vie (change les vies)" :
build BlocWatch
build BlocSelectVies
```

Le bloc `watch` se reconstruit à chaque fois. Les blocs `select` ne se reconstruisent
que lorsque **leur** valeur change. Sur un écran complexe, la différence se compte
en dizaines de reconstructions évitées par seconde.

---

## 52.22.2 — Les deux contraintes de `select`

**Contrainte 1 : uniquement dans `build()`.** Le paquet en fait une assertion
explicite :

```text
Tried to use `context.select` outside of the `build` method of a widget.

Any usage other than inside the `build` method of a widget are not supported.
```

**Contrainte 2 : la valeur sélectionnée doit être comparable avec `==`.**

```dart
// INUTILE : la liste est le même objet, == renvoie toujours true,
// donc le widget ne se reconstruit jamais.
final List<String> objets =
    context.select<Joueur, List<String>>((Joueur j) => j.inventaire);

// CORRECT : on sélectionne une valeur scalaire.
final int nombre =
    context.select<Joueur, int>((Joueur j) => j.inventaire.length);
```

Sélectionnez toujours un `int`, un `String`, un `bool`, un `double`, un `enum`, ou
une classe qui redéfinit `==` (chapitre 10).

Le paquet interdit d'ailleurs `select` à l'intérieur d'une `SliverList` :

```text
Tried to use context.select inside a SliverList/SliderGridView.
```

---

## 52.23 — `Consumer<T>` et la portée des reconstructions

`Consumer<T>` est un widget qui fait la même chose que `watch`, mais en **délimitant
visuellement** la zone reconstruite.

Sa signature, vérifiée dans les sources :

```text
Consumer({
  Key? key,
  required Widget Function(BuildContext context, T value, Widget? child) builder,
  Widget? child,
})
```

```dart
Consumer<Joueur>(
  builder: (BuildContext context, Joueur joueur, Widget? enfant) {
    return Text('Score : ${joueur.score}');
  },
)
```

Le `builder` reçoit un `context` situé **sous** le provider — ce qui règle au passage
le piège de 52.21.3.

---

## 52.23.1 — Quand `Consumer` est indispensable

Comparez ces deux écritures. Elles affichent la même chose, mais ne reconstruisent
pas la même quantité de widgets.

```dart
// VERSION A : watch en haut de build.
// Tout le Scaffold est reconstruit à chaque notification.
@override
Widget build(BuildContext context) {
  final Joueur joueur = context.watch<Joueur>();
  return Scaffold(
    appBar: AppBar(title: const Text('Jeu')),
    body: Column(
      children: <Widget>[
        const ImageLourde(),
        Text('Score : ${joueur.score}'),
      ],
    ),
  );
}

// VERSION B : Consumer autour du seul Text.
// Seul le Text est reconstruit.
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: const Text('Jeu')),
    body: Column(
      children: <Widget>[
        const ImageLourde(),
        Consumer<Joueur>(
          builder: (BuildContext context, Joueur joueur, Widget? enfant) {
            return Text('Score : ${joueur.score}');
          },
        ),
      ],
    ),
  );
}
```

```text
  VERSION A                        VERSION B
  ─────────                        ─────────
  Scaffold      RECONSTRUIT        Scaffold      inchangé
   ├ AppBar     RECONSTRUIT         ├ AppBar     inchangé
   └ Column     RECONSTRUIT         └ Column     inchangé
     ├ ImageLourde RECONSTRUITE       ├ ImageLourde inchangée
     └ Text     RECONSTRUIT           └ Consumer  RECONSTRUIT
```

> **Règle pratique :** `context.watch` en haut d'un `build()` est acceptable si le
> widget est petit. Dès que le widget contient autre chose que la donnée observée,
> utilisez `Consumer` ou extrayez un sous-widget.

Le paramètre `child` de `Consumer` joue le même rôle qu'en 52.16.2 : il permet de
sortir un sous-arbre coûteux de la zone reconstruite.

---

## 52.23.2 — `Selector`, la version filtrée de `Consumer`

Il existe aussi un widget `Selector<A, S>`, qui est à `Consumer` ce que `select` est
à `watch` :

```text
Selector({
  Key? key,
  required ValueWidgetBuilder<S> builder,
  required S Function(BuildContext, A) selector,
  ShouldRebuild<S>? shouldRebuild,
  Widget? child,
})
```

```dart
Selector<Joueur, int>(
  selector: (BuildContext context, Joueur joueur) => joueur.vies,
  builder: (BuildContext context, int vies, Widget? enfant) {
    return Text('Vies : $vies');
  },
)
```

Le paquet fournit également `Consumer2` à `Consumer6` et `Selector2` à `Selector6`
pour consommer plusieurs providers à la fois. Utilisez-les avec parcimonie : au-delà
de deux, il est presque toujours plus lisible d'imbriquer.

---

## 52.24 — `watch` ou `read` : la règle simple

Voici la règle. Apprenez-la par cœur, elle règle 95 % des hésitations.

```text
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   Je suis dans build() et j'AFFICHE la valeur            │
  │        ->  context.watch<T>()   (ou Consumer)            │
  │                                                          │
  │   Je suis dans build() et je n'affiche qu'UN champ       │
  │        ->  context.select<T, R>()                        │
  │                                                          │
  │   Je suis dans un callback (onPressed, onTap, onChanged) │
  │        ->  context.read<T>()                             │
  │                                                          │
  │   Je suis dans initState / dispose / un timer            │
  │        ->  context.read<T>()                             │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

Formulée autrement, en une phrase :

> **`watch` pour afficher, `read` pour agir.**

---

## 52.24.1 — Les deux erreurs symétriques

**Erreur A : `watch` dans un callback.**

```dart
onPressed: () {
  // Erreur : on s'abonne depuis un callback.
  context.watch<Joueur>().ramasserPiece(50);
}
```

Selon le contexte, `provider` lève :

```text
Tried to listen to a value exposed with provider, from outside of the widget tree.

This is likely caused by an event handler (like a button's onPressed) that called
watch/create.
```

**Erreur B : `read` dans `build` pour afficher.** Vue en 52.21.1 : aucune erreur,
mais l'écran ne se met jamais à jour.

Notez l'asymétrie : l'erreur A **crie**, l'erreur B **se tait**. C'est pourquoi
l'erreur B est plus dangereuse.

---

## 52.24.2 — Le cas de `initState`

Depuis `initState()`, seul `read` est possible :

```dart
@override
void initState() {
  super.initState();
  // CORRECT : lecture ponctuelle, sans abonnement.
  final Joueur joueur = context.read<Joueur>();
  debugPrint('Score initial : ${joueur.score}');
}
```

Si vous utilisez `watch` ou `Provider.of` avec `listen: true` depuis `initState`,
`provider` lève une erreur explicite :

```text
Tried to listen to an InheritedWidget in a life-cycle that will never be called
again.

This error typically happens when calling Provider.of with `listen` to `true`,
in a situation where listening to the provider doesn't make sense, such as:
- initState of a StatefulWidget
- the "create" callback of a provider
```

La raison est logique : `initState` n'est appelée qu'une fois. S'abonner depuis
cette méthode n'aurait aucun effet.

---

## 52.25 — `MultiProvider`

Une application réelle a plusieurs objets d'état. L'imbrication devient vite
illisible :

```dart
// Imbrication manuelle : le "triangle de la mort".
ChangeNotifierProvider<Joueur>(
  create: (_) => Joueur(),
  child: ChangeNotifierProvider<Inventaire>(
    create: (_) => Inventaire(),
    child: ChangeNotifierProvider<Parametres>(
      create: (_) => Parametres(),
      child: const MonApplication(),
    ),
  ),
)
```

`MultiProvider` aplatit cette structure :

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider<Joueur>(create: (_) => Joueur()),
    ChangeNotifierProvider<Inventaire>(create: (_) => Inventaire()),
    ChangeNotifierProvider<Parametres>(create: (_) => Parametres()),
  ],
  child: const MonApplication(),
)
```

Le résultat dans l'arbre est **strictement identique** : `MultiProvider` se contente
d'imbriquer les providers pour vous.

---

> **Remarque de syntaxe.** Ailleurs dans ce cours, nous écrivons toujours le type
> de liste explicitement (`children: <Widget>[...]`). Ici, nous écrivons
> `providers: [...]` sans annotation, car le type attendu est
> `List<SingleChildWidget>`, et cette classe n'est pas exportée par
> `package:provider/provider.dart` mais par
> `package:provider/single_child_widget.dart`. L'inférence de type de Dart fait
> parfaitement le travail : n'ajoutez pas d'annotation, et n'ajoutez pas d'import.

---

## 52.25.1 — L'ordre compte

Point important : dans la liste, chaque provider peut lire ceux qui le **précèdent**,
jamais ceux qui le suivent.

```text
  providers: [
    A,     <- A ne voit rien
    B,     <- B voit A
    C,     <- C voit A et B
  ]
```

Si `C` a besoin de `A`, tout va bien. Si `A` a besoin de `C`, vous obtiendrez une
`ProviderNotFoundException`. Placez donc les dépendances **en premier**.

---

## 52.25.2 — Un provider sans `child` hors de `MultiProvider`

Notez que le paramètre `child` est absent dans la liste ci-dessus. C'est autorisé
uniquement à l'intérieur d'un `MultiProvider`. Ailleurs, l'oubli déclenche :

```text
ChangeNotifierProvider<Joueur> used outside of MultiProvider must specify a child
```

---

## 52.26 — `Provider.value` et le piège de la recréation

Il existe deux façons de fournir un objet :

```text
  create: (_) => Joueur()    le provider CRÉE l'objet, et le libère (dispose)
  value: monJoueur           le provider EXPOSE un objet existant, sans le libérer
```

La règle officielle est nette :

> **Utilisez `create` quand vous instanciez l'objet. Utilisez `.value` uniquement
> pour exposer un objet qui existe déjà et dont le cycle de vie est géré ailleurs.**

---

## 52.26.1 — Le piège : `.value` avec une instanciation

Voici l'erreur classique :

```dart
// FAUX, et grave.
@override
Widget build(BuildContext context) {
  return ChangeNotifierProvider<Joueur>.value(
    value: Joueur(),          // nouvelle instance à CHAQUE build !
    child: const EcranJeu(),
  );
}
```

Que se passe-t-il ?

```text
  build 1  ->  Joueur#a  (score 0)
  l'utilisateur ramasse 50 points -> notifyListeners -> rebuild du parent
  build 2  ->  Joueur#b  (score 0)   <- l'état est PERDU
  build 3  ->  Joueur#c  (score 0)
```

L'état est réinitialisé à chaque reconstruction du widget parent, et les anciennes
instances ne sont jamais libérées : c'est à la fois un bug fonctionnel et une fuite
mémoire.

Avec `create`, le problème n'existe pas : `provider` n'appelle la fonction
**qu'une seule fois**, et conserve l'objet tant que le provider reste dans l'arbre.

```dart
// CORRECT : l'objet est créé une seule fois.
ChangeNotifierProvider<Joueur>(
  create: (BuildContext context) => Joueur(),
  child: const EcranJeu(),
)
```

---

## 52.26.2 — Le cas légitime de `.value`

`.value` est le bon outil dans deux situations.

**Situation 1 : exposer un élément de liste à un écran de détail.**

```dart
// L'objet vient d'une liste déjà gérée ailleurs : on l'expose, on ne le crée pas.
Navigator.push(
  context,
  MaterialPageRoute<void>(
    builder: (BuildContext _) => ChangeNotifierProvider<Article>.value(
      value: article,
      child: const EcranDetailArticle(),
    ),
  ),
);
```

**Situation 2 : exposer un objet détenu par un `State`.**

```dart
class _MonEcranState extends State<MonEcran> {
  final Joueur _joueur = Joueur();   // créé et libéré par le State

  @override
  void dispose() {
    _joueur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Joueur>.value(
      value: _joueur,     // ici .value est correct : _joueur est stable
      child: const CorpsEcran(),
    );
  }
}
```

Retenez la question de contrôle : **« qui appelle `dispose()` ? »**

```text
  create : c'est le provider qui appelle dispose()
  .value : c'est VOUS qui devez appeler dispose()
```

---

## 52.26.3 — Le piège du contexte de navigation

Un dernier piège très fréquent, lié à 52.19.1. Ce code échoue :

```dart
// FAUX : la nouvelle route n'est pas sous le provider de l'écran courant.
Navigator.push(
  context,
  MaterialPageRoute<void>(builder: (BuildContext _) => const EcranDetail()),
);
```

Si le provider a été déclaré **dans** l'écran courant, la route poussée est un
sous-arbre du `Navigator`, donc **au-dessus** du provider. Vous obtiendrez :

```text
Error: Could not find the correct Provider<Joueur> above this EcranDetail Widget
[...]
- The provider you are trying to read is in a different route.
```

Deux corrections possibles :

```dart
// Correction 1 : déclarer le provider au-dessus de MaterialApp (le plus courant).

// Correction 2 : réexposer l'objet dans la nouvelle route.
final Joueur joueur = context.read<Joueur>();
Navigator.push(
  context,
  MaterialPageRoute<void>(
    builder: (BuildContext _) => ChangeNotifierProvider<Joueur>.value(
      value: joueur,
      child: const EcranDetail(),
    ),
  ),
);
```

Notez qu'on lit `joueur` **avant** le `push`, avec le `context` de l'écran courant.
Lire depuis le `builder` de la route ne fonctionnerait pas.

---

## 52.27 — `ProxyProvider`

`ProxyProvider` sert à construire un objet **à partir** d'un autre provider, et à le
reconstruire quand celui-ci change.

Sa signature :

```text
ProxyProvider<T, R>({
  Key? key,
  Create<R>? create,
  required ProxyProviderBuilder<T, R> update,
  UpdateShouldNotify<R>? updateShouldNotify,
  Dispose<R>? dispose,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
})
```

Le paramètre clé est `update`, de type :

```text
R Function(BuildContext context, T value, R? previous)
```

Il reçoit la valeur du provider amont (`T`), l'ancienne valeur produite
(`R? previous`), et renvoie la nouvelle valeur (`R`).

---

## 52.27.1 — Un exemple : le prix du panier dépend du niveau du joueur

Supposons que les objets de la boutique soient moins chers quand le joueur monte en
niveau. Le service de tarification dépend donc du joueur.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        // 1. Le provider amont.
        ChangeNotifierProvider<Joueur>(create: (_) => Joueur()),
        // 2. Le provider aval : reconstruit dès que Joueur notifie.
        ProxyProvider<Joueur, Tarification>(
          update: (BuildContext context, Joueur joueur, Tarification? ancien) {
            return Tarification(niveau: joueur.niveau);
          },
        ),
      ],
      child: const ApplicationProxy(),
    ),
  );
}

class Joueur extends ChangeNotifier {
  int _niveau = 1;
  int get niveau => _niveau;

  void monterDeNiveau() {
    _niveau++;
    notifyListeners();
  }
}

// Objet SIMPLE (pas un ChangeNotifier) : il est recalculé, pas modifié.
class Tarification {
  const Tarification({required this.niveau});

  final int niveau;

  // 5 % de remise par niveau, plafonnée à 50 %.
  double get remise => (niveau - 1) * 0.05 > 0.5 ? 0.5 : (niveau - 1) * 0.05;

  int prixPotion() => (100 * (1 - remise)).round();
}

class ApplicationProxy extends StatelessWidget {
  const ApplicationProxy({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.brown),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('ProxyProvider')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Builder(
                builder: (BuildContext context) {
                  final int niveau =
                      context.select<Joueur, int>((Joueur j) => j.niveau);
                  final Tarification tarif = context.watch<Tarification>();
                  return Column(
                    children: <Widget>[
                      Text('Niveau : $niveau',
                          style: const TextStyle(fontSize: 24)),
                      Text('Remise : ${(tarif.remise * 100).round()} %',
                          style: const TextStyle(fontSize: 24)),
                      Text('Prix de la potion : ${tarif.prixPotion()} or',
                          style: const TextStyle(fontSize: 24)),
                    ],
                  );
                },
              ),
              const SizedBox(height: 32),
              Builder(
                builder: (BuildContext context) => ElevatedButton(
                  onPressed: () => context.read<Joueur>().monterDeNiveau(),
                  child: const Text('Monter de niveau'),
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
Niveau : 1          Niveau : 4
Remise : 0 %   ->   Remise : 15 %
Prix : 100 or       Prix : 85 or
```

`Tarification` est recréée automatiquement à chaque `notifyListeners()` de `Joueur`.
Vous n'écrivez aucune synchronisation.

---

## 52.27.2 — La variante `ChangeNotifierProxyProvider`

Si l'objet aval est lui-même un `ChangeNotifier` que vous ne voulez pas recréer,
utilisez `ChangeNotifierProxyProvider` et **mettez à jour** l'instance existante :

```dart
ChangeNotifierProxyProvider<Joueur, Panier>(
  create: (BuildContext context) => Panier(),
  update: (BuildContext context, Joueur joueur, Panier? panier) {
    // On réutilise l'instance et on lui injecte la nouvelle dépendance.
    return (panier ?? Panier())..mettreAJourNiveau(joueur.niveau);
  },
)
```

Attention : ne recréez **jamais** l'objet dans `update` s'il porte de l'état, sinon
vous perdez son contenu à chaque notification amont — c'est la même erreur qu'en
52.26.1.

---

## 52.28 — `FutureProvider` et `StreamProvider`

Ces deux providers exposent un résultat asynchrone (chapitre 15). Leurs signatures
exactes :

```text
FutureProvider<T>({
  Key? key,
  required Create<Future<T>?> create,
  required T initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
})

StreamProvider<T>({
  Key? key,
  required Create<Stream<T>?> create,
  required T initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
})
```

Notez que `initialData` est **obligatoire**. C'est un changement de la version 5.0.0
du paquet : avant, il était facultatif. La valeur fournie est celle qui est visible
tant que le `Future` n'a pas abouti ou que le `Stream` n'a rien émis.

---

## 52.28.1 — Exemple complet

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        FutureProvider<String>(
          initialData: 'Chargement du monde...',
          create: (BuildContext context) => chargerNomDuMonde(),
        ),
        StreamProvider<int>(
          initialData: 0,
          create: (BuildContext context) => fluxEnergie(),
        ),
      ],
      child: const ApplicationAsync(),
    ),
  );
}

Future<String> chargerNomDuMonde() async {
  await Future<void>.delayed(const Duration(seconds: 2));
  return 'Vallée des Ombres';
}

Stream<int> fluxEnergie() async* {
  int energie = 100;
  while (energie > 0) {
    await Future<void>.delayed(const Duration(milliseconds: 700));
    energie -= 5;
    yield energie;
  }
}

class ApplicationAsync extends StatelessWidget {
  const ApplicationAsync({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.lightGreen),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Future et Stream')),
        body: Center(
          child: Builder(
            builder: (BuildContext context) {
              final String monde = context.watch<String>();
              final int energie = context.watch<int>();
              return Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Text(monde, style: const TextStyle(fontSize: 26)),
                  const SizedBox(height: 24),
                  Text('Énergie : $energie %',
                      style: const TextStyle(fontSize: 26)),
                  const SizedBox(height: 16),
                  SizedBox(
                    width: 240,
                    child: LinearProgressIndicator(value: energie / 100),
                  ),
                ],
              );
            },
          ),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
t = 0 s  : "Chargement du monde..."   Énergie : 0 %
t = 0.7 s: "Chargement du monde..."   Énergie : 95 %
t = 2 s  : "Vallée des Ombres"        Énergie : 90 %
t = 5 s  : "Vallée des Ombres"        Énergie : 70 %
```

---

## 52.28.2 — Quand ne PAS les utiliser

Ces deux providers sont pratiques mais limités.

```text
  Ils exposent une VALEUR, pas un ÉTAT de chargement.
  Vous ne savez pas distinguer :
     - "en cours de chargement"
     - "chargé, résultat vide"
     - "en erreur"
  sauf à encoder ces cas dans le type T lui-même.
```

Pour un chargement réseau réel, avec un indicateur de progression et une gestion
d'erreur, préférez `FutureBuilder` et `StreamBuilder`, qui donnent accès à un
`AsyncSnapshot` complet. C'est le sujet du chapitre 53.

Un piège à connaître : deux providers du **même type** dans le même
`MultiProvider` se masquent l'un l'autre. `context.watch<String>()` ne renverra que
le plus proche. Enveloppez toujours vos données dans des types dédiés
(`class NomDuMonde { ... }`) plutôt que d'exposer des `String` ou des `int` bruts.

---

## 52.29 — Séparer l'état de l'interface : le modèle et le widget

Vous avez maintenant tous les outils. Reste la question la plus importante :
**comment organiser les fichiers ?**

L'organisation recommandée, dans la continuité du chapitre 16 :

```text
  lib/
   ├── main.dart                  point d'entrée, MultiProvider
   ├── modeles/
   │    ├── article.dart          classes de données PURES
   │    └── panier.dart           classes ChangeNotifier
   ├── ecrans/
   │    ├── ecran_catalogue.dart
   │    ├── ecran_panier.dart
   │    └── ecran_detail.dart
   └── widgets/
        ├── carte_article.dart    widgets réutilisables
        └── badge_panier.dart
```

Deux règles de dépendance, qu'il faut tenir strictement :

```text
  RÈGLE 1 : un fichier de modeles/ n'importe JAMAIS
            'package:flutter/material.dart'.
            Au plus 'package:flutter/foundation.dart' pour ChangeNotifier.

  RÈGLE 2 : un fichier de widgets/ ne contient AUCUNE logique métier.
            Il affiche, il appelle des méthodes du modèle, c'est tout.
```

La règle 1 est la plus importante. Si votre modèle importe `material.dart`, c'est
qu'il manipule des `Widget`, des `Color` ou du `BuildContext` — et il devient
alors impossible à tester sans interface (52.31).

---

## 52.29.1 — Deux sortes de classes dans `modeles/`

Ne confondez pas les deux.

**Les classes de données**, immuables, sans état :

```dart
// modeles/article.dart — aucune notification, aucun état.
class Article {
  const Article({
    required this.id,
    required this.nom,
    required this.prix,
    required this.icone,
  });

  final String id;
  final String nom;
  final int prix;
  final int icone; // code point d'une icône Material

  @override
  bool operator ==(Object other) =>
      other is Article && other.id == id;

  @override
  int get hashCode => id.hashCode;
}
```

**Les classes d'état**, mutables, qui notifient :

```dart
// modeles/panier.dart — l'état partagé.
class Panier extends ChangeNotifier {
  final Map<Article, int> _lignes = <Article, int>{};
  // ...
}
```

Redéfinir `==` et `hashCode` sur les classes de données (chapitre 10) est très
utile : cela permet à `context.select` et à `Selector` de comparer correctement, et
cela permet d'utiliser l'objet comme clé de `Map`, comme ci-dessus.

---

## 52.29.2 — Le piège de la mutation sans notification

Voici l'erreur la plus répandue avec `ChangeNotifier`. Elle mérite une section
entière.

```dart
class Panier extends ChangeNotifier {
  final List<Article> _articles = <Article>[];

  // Accesseur DANGEREUX : il expose la liste interne.
  List<Article> get articles => _articles;

  void ajouter(Article a) {
    _articles.add(a);
    notifyListeners();
  }
}
```

Le problème est l'accesseur. N'importe qui peut écrire :

```dart
// La liste est modifiée, mais AUCUNE notification n'est émise.
context.read<Panier>().articles.add(article);
```

L'article est bien ajouté en mémoire. L'interface, elle, ne bouge pas. Vous
chercherez le bug pendant une heure.

Les deux protections :

```dart
// Protection 1 : renvoyer une vue non modifiable.
List<Article> get articles => List<Article>.unmodifiable(_articles);

// Protection 2 : ne jamais exposer la liste, seulement ce qui est utile.
int get nombreArticles => _articles.length;
int get total => _articles.fold(0, (int s, Article a) => s + a.prix);
bool contient(Article a) => _articles.contains(a);
```

Avec la protection 1, la tentative de modification lève une erreur immédiate :

```text
Unsupported operation: Cannot add to an unmodifiable list
```

Une erreur bruyante vaut infiniment mieux qu'un bug silencieux.

---

## 52.30 — Un panier d'achat avec `provider` (exemple complet)

Voici l'exemple canonique, complet et exécutable. Il met en pratique tout ce qui
précède : un modèle séparé, `MultiProvider`, `select` pour le badge, `Consumer` pour
la liste, `read` pour les actions, et la navigation entre deux écrans.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        Provider<Catalogue>(create: (_) => Catalogue()),
        ChangeNotifierProvider<Panier>(create: (_) => Panier()),
      ],
      child: const ApplicationBoutique(),
    ),
  );
}

// ═════════════════════════════════════════════════════════════
// MODÈLES
// ═════════════════════════════════════════════════════════════

class Article {
  const Article({
    required this.id,
    required this.nom,
    required this.prix,
    required this.icone,
  });

  final String id;
  final String nom;
  final int prix;
  final IconData icone;

  @override
  bool operator ==(Object other) => other is Article && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

// Catalogue : données FIXES, donc pas de ChangeNotifier.
class Catalogue {
  final List<Article> articles = const <Article>[
    Article(id: 'a1', nom: 'Potion de soin', prix: 50, icone: Icons.local_drink),
    Article(id: 'a2', nom: 'Épée courte', prix: 120, icone: Icons.hardware),
    Article(id: 'a3', nom: 'Bouclier de bois', prix: 90, icone: Icons.shield),
    Article(id: 'a4', nom: 'Arc long', prix: 210, icone: Icons.gps_fixed),
    Article(id: 'a5', nom: 'Grimoire', prix: 300, icone: Icons.menu_book),
  ];
}

// Panier : état PARTAGÉ, donc ChangeNotifier.
class Panier extends ChangeNotifier {
  final Map<Article, int> _lignes = <Article, int>{};

  // Vue non modifiable : personne ne peut contourner les méthodes.
  Map<Article, int> get lignes => Map<Article, int>.unmodifiable(_lignes);

  int get nombreArticles =>
      _lignes.values.fold(0, (int s, int q) => s + q);

  int get total => _lignes.entries
      .fold(0, (int s, MapEntry<Article, int> e) => s + e.key.prix * e.value);

  bool get estVide => _lignes.isEmpty;

  int quantiteDe(Article a) => _lignes[a] ?? 0;

  void ajouter(Article a) {
    _lignes[a] = (_lignes[a] ?? 0) + 1;
    notifyListeners();
  }

  void retirer(Article a) {
    final int actuel = _lignes[a] ?? 0;
    if (actuel <= 1) {
      _lignes.remove(a);
    } else {
      _lignes[a] = actuel - 1;
    }
    notifyListeners();
  }

  void vider() {
    if (_lignes.isEmpty) return;
    _lignes.clear();
    notifyListeners();
  }
}

// ═════════════════════════════════════════════════════════════
// INTERFACE
// ═════════════════════════════════════════════════════════════

class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Boutique',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
        useMaterial3: true,
      ),
      home: const EcranCatalogue(),
    );
  }
}

class EcranCatalogue extends StatelessWidget {
  const EcranCatalogue({super.key});

  @override
  Widget build(BuildContext context) {
    // Provider (non ChangeNotifier) : read suffit, il ne change jamais.
    final Catalogue catalogue = context.read<Catalogue>();
    return Scaffold(
      appBar: AppBar(
        title: const Text('Boutique'),
        actions: const <Widget>[BadgePanier()],
      ),
      body: ListView.separated(
        itemCount: catalogue.articles.length,
        separatorBuilder: (BuildContext context, int i) =>
            const Divider(height: 1),
        itemBuilder: (BuildContext context, int index) {
          return LigneCatalogue(article: catalogue.articles[index]);
        },
      ),
    );
  }
}

// Ce widget ne s'abonne qu'au NOMBRE d'articles.
class BadgePanier extends StatelessWidget {
  const BadgePanier({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BadgePanier');
    final int nombre =
        context.select<Panier, int>((Panier p) => p.nombreArticles);
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: Badge(
        isLabelVisible: nombre > 0,
        label: Text('$nombre'),
        child: IconButton(
          icon: const Icon(Icons.shopping_cart),
          tooltip: 'Voir le panier',
          onPressed: () {
            // Le provider est au-dessus de MaterialApp : la route y a accès.
            Navigator.of(context).push(
              MaterialPageRoute<void>(
                builder: (BuildContext _) => const EcranPanier(),
              ),
            );
          },
        ),
      ),
    );
  }
}

// Ce widget ne s'abonne qu'à SA propre quantité.
class LigneCatalogue extends StatelessWidget {
  const LigneCatalogue({super.key, required this.article});

  final Article article;

  @override
  Widget build(BuildContext context) {
    debugPrint('build LigneCatalogue ${article.nom}');
    final int quantite =
        context.select<Panier, int>((Panier p) => p.quantiteDe(article));
    return ListTile(
      leading: CircleAvatar(child: Icon(article.icone)),
      title: Text(article.nom),
      subtitle: Text('${article.prix} pièces d\'or'),
      trailing: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          if (quantite > 0) ...<Widget>[
            IconButton(
              icon: const Icon(Icons.remove_circle_outline),
              onPressed: () => context.read<Panier>().retirer(article),
            ),
            Text('$quantite', style: const TextStyle(fontSize: 18)),
          ],
          IconButton(
            icon: const Icon(Icons.add_circle),
            onPressed: () => context.read<Panier>().ajouter(article),
          ),
        ],
      ),
    );
  }
}

class EcranPanier extends StatelessWidget {
  const EcranPanier({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Mon panier'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            tooltip: 'Vider le panier',
            onPressed: () => context.read<Panier>().vider(),
          ),
        ],
      ),
      body: Consumer<Panier>(
        builder: (BuildContext context, Panier panier, Widget? enfant) {
          if (panier.estVide) {
            return const Center(child: Text('Votre panier est vide.'));
          }
          final List<MapEntry<Article, int>> lignes =
              panier.lignes.entries.toList();
          return ListView.builder(
            itemCount: lignes.length,
            itemBuilder: (BuildContext context, int index) {
              final MapEntry<Article, int> ligne = lignes[index];
              return ListTile(
                leading: CircleAvatar(child: Icon(ligne.key.icone)),
                title: Text(ligne.key.nom),
                subtitle: Text('${ligne.value} x ${ligne.key.prix} or'),
                trailing: Text(
                  '${ligne.value * ligne.key.prix} or',
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
              );
            },
          );
        },
      ),
      bottomNavigationBar: const BarreTotal(),
    );
  }
}

class BarreTotal extends StatelessWidget {
  const BarreTotal({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BarreTotal');
    final int total = context.select<Panier, int>((Panier p) => p.total);
    return Material(
      color: Theme.of(context).colorScheme.primaryContainer,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: <Widget>[
            const Text('TOTAL', style: TextStyle(fontSize: 18)),
            Text('$total or',
                style: const TextStyle(
                    fontSize: 22, fontWeight: FontWeight.bold)),
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
│ Boutique                    (2) cart │
├──────────────────────────────────────┤
│ (o) Potion de soin       - 1 +       │
│     50 pièces d'or                   │
│ (o) Épée courte              +       │
│     120 pièces d'or                  │
│ (o) Bouclier de bois     - 1 +       │
│     90 pièces d'or                   │
└──────────────────────────────────────┘

Console après un appui sur "+" de la potion :
build BadgePanier
build LigneCatalogue Potion de soin
```

Deux widgets reconstruits sur huit. Les quatre autres lignes du catalogue ne bougent
pas, parce que `context.select` a comparé leur quantité et l'a trouvée inchangée.

---

## 52.30.1 — Analyse des choix faits dans cet exemple

| Décision | Raison |
| --- | --- |
| `Provider<Catalogue>` et non `ChangeNotifierProvider` | Le catalogue ne change jamais |
| `context.read<Catalogue>()` dans `build` | Il ne change jamais, l'abonnement serait inutile |
| `context.select` sur le badge | Le badge n'affiche qu'un entier |
| `context.select` sur chaque ligne | Chaque ligne n'affiche que sa quantité |
| `Consumer<Panier>` dans l'écran panier | La liste entière dépend du panier |
| `context.read` dans tous les `onPressed` | Les boutons n'affichent rien |
| `Map<Article, int>` plutôt que `List<Article>` | Le regroupement par quantité est naturel |
| `Map.unmodifiable` dans l'accesseur | Empêche les mutations sans notification |
| Provider au-dessus de `MaterialApp` | L'écran panier est une autre route |

---

## 52.31 — Tester une classe d'état sans interface (rappel chapitre 16)

Voici la récompense de la séparation. La classe `Panier` n'utilise aucun widget :
elle se teste comme n'importe quelle classe Dart.

Aucune installation n'est nécessaire : `flutter create` (chapitre 43) place déjà
`flutter_test` dans les `dev_dependencies` de votre `pubspec.yaml`.

```text
dev_dependencies:
  flutter_test:
    sdk: flutter
```

Créez `test/panier_test.dart`. Nous reprenons ici les classes de 52.29.1, où
`Article.icone` est un simple `int` (un code de caractère), afin que
`modeles/article.dart` n'ait rien à importer de Flutter :

```dart
import 'package:flutter_test/flutter_test.dart';

import 'package:ma_boutique/modeles/article.dart';
import 'package:ma_boutique/modeles/panier.dart';

void main() {
  const Article potion =
      Article(id: 'a1', nom: 'Potion', prix: 50, icone: 0xe000);
  const Article epee =
      Article(id: 'a2', nom: 'Épée', prix: 120, icone: 0xe001);

  group('Panier', () {
    test('un panier neuf est vide', () {
      final Panier panier = Panier();
      expect(panier.estVide, isTrue);
      expect(panier.nombreArticles, 0);
      expect(panier.total, 0);
    });

    test('ajouter deux fois le même article incrémente la quantité', () {
      final Panier panier = Panier();
      panier.ajouter(potion);
      panier.ajouter(potion);
      expect(panier.quantiteDe(potion), 2);
      expect(panier.nombreArticles, 2);
      expect(panier.total, 100);
    });

    test('le total additionne les articles différents', () {
      final Panier panier = Panier();
      panier.ajouter(potion);
      panier.ajouter(epee);
      expect(panier.total, 170);
    });

    test('retirer le dernier exemplaire supprime la ligne', () {
      final Panier panier = Panier();
      panier.ajouter(potion);
      panier.retirer(potion);
      expect(panier.estVide, isTrue);
    });

    test('ajouter émet exactement une notification', () {
      final Panier panier = Panier();
      int notifications = 0;
      panier.addListener(() => notifications++);

      panier.ajouter(potion);
      expect(notifications, 1);

      panier.ajouter(epee);
      expect(notifications, 2);
    });

    test('vider un panier déjà vide ne notifie pas', () {
      final Panier panier = Panier();
      int notifications = 0;
      panier.addListener(() => notifications++);

      panier.vider();
      expect(notifications, 0);
    });

    test('la liste exposée est non modifiable', () {
      final Panier panier = Panier();
      panier.ajouter(potion);
      expect(() => panier.lignes[epee] = 1, throwsUnsupportedError);
    });
  });
}
```

**Résultat :**

```text
flutter test

00:02 +7: All tests passed!
```

Sept tests, aucune interface, une exécution en deux secondes. C'est là tout
l'intérêt d'un modèle qui n'importe pas `material.dart`.

> **Le test le plus utile est celui du nombre de notifications.** Il attrape à la
> fois le `notifyListeners()` oublié (0 au lieu de 1) et le `notifyListeners()`
> appelé en boucle (3 au lieu de 1).

---

## 52.32 — Panorama honnête des autres solutions : Riverpod, Bloc, GetX, signals

La documentation officielle de Flutter ne recommande aucun paquet en particulier.
Elle présente `setState`, `ValueNotifier`, `InheritedWidget` et `InheritedModel`
comme les approches natives, puis renvoie vers le sujet `#state-management` de
pub.dev, en précisant que « le meilleur choix dépend souvent de la complexité de
l'application, des préférences de l'équipe et des problèmes à résoudre ».

Voici une présentation honnête des principales solutions. Aucune n'est
« la bonne » : elles font des compromis différents.

---

## 52.32.1 — Riverpod

Écrit par le même auteur que `provider`, Riverpod en est une réécriture qui corrige
ses défauts structurels.

```text
  Ce qu'il apporte :
    - pas de BuildContext : un provider est une variable globale typée
    - donc pas de ProviderNotFoundException : les erreurs sont à la COMPILATION
    - plusieurs providers du même type possibles
    - gestion native de l'asynchrone (AsyncValue : loading / data / error)
    - génération de code optionnelle avec riverpod_generator

  Ce qu'il coûte :
    - un vocabulaire plus riche à apprendre (ref, Notifier, AsyncNotifier)
    - une migration non triviale depuis provider
```

C'est aujourd'hui le choix par défaut d'une grande partie de la communauté pour un
nouveau projet de taille moyenne ou grande.

---

## 52.32.2 — Bloc / Cubit

Le paquet `flutter_bloc` applique une architecture stricte, inspirée de Redux.

```text
  Cubit  : état + méthodes qui appellent emit(nouvelEtat)
  Bloc   : état + ÉVÉNEMENTS ; chaque événement produit un nouvel état

  Ce qu'il apporte :
    - une discipline forte : un état IMMUABLE, remplacé, jamais muté
    - une traçabilité complète (chaque transition est observable)
    - excellent en grande équipe, sur un projet de longue durée

  Ce qu'il coûte :
    - beaucoup de code pour un petit besoin
    - une classe d'état, une classe d'événement, un bloc, un builder
```

Un compteur en `Cubit` fait 20 lignes contre 5 en `ChangeNotifier`. Sur une
application bancaire de 200 écrans, ces 15 lignes supplémentaires par
fonctionnalité sont un investissement rentable. Sur une application de 5 écrans,
c'est du poids mort.

---

## 52.32.3 — GetX

`get` est un paquet « tout-en-un » : état, navigation, injection de dépendances,
internationalisation, et davantage.

```text
  Ce qu'il apporte :
    - une syntaxe très courte (Obx(() => Text('${c.score}')))
    - une navigation sans BuildContext (Get.to(...))
    - une prise en main immédiate

  Ce qu'il coûte :
    - il s'écarte des conventions de Flutter, ce qui rend le code difficile
      à relire pour un développeur venant d'ailleurs
    - un couplage fort : sortir de GetX impose une réécriture large
    - une utilisation de variables globales que beaucoup jugent risquée
```

GetX est populaire, notamment pour les prototypes rapides. Il est aussi la solution
la plus controversée. Sachez qu'elle existe ; en contexte professionnel, vérifiez
que l'équipe est d'accord avant de l'imposer.

---

## 52.32.4 — Les signals

Les *signals* sont un modèle de réactivité fine venu du monde web (SolidJS, Angular,
Vue). Plusieurs paquets Dart les proposent, dont `signals`.

```text
  Principe :
     final score = signal(0);
     final rang  = computed(() => score.value ~/ 100);

     score.value = 250;   -> rang se recalcule TOUT SEUL, puis l'UI se met à jour

  Ce qu'ils apportent :
    - des valeurs dérivées automatiques (computed)
    - une granularité de reconstruction très fine
    - une syntaxe légère

  Ce qu'ils coûtent :
    - un écosystème plus récent et plus petit dans le monde Flutter
    - moins de ressources et d'exemples que provider ou bloc
```

Remarquez que `ValueNotifier` (52.16) est déjà une forme rudimentaire de signal :
une valeur observable. Les paquets signals ajoutent surtout le `computed`.

---

## 52.32.5 — Les autres, en une ligne chacun

| Paquet | Idée |
| --- | --- |
| `flutter_redux` | Un unique magasin global, des actions, des réducteurs purs |
| `mobx` | Observables et réactions automatiques, avec génération de code |
| `get_it` | Localisateur de services : injection de dépendances, sans réactivité |
| `states_rebuilder` | Modèle réactif complet, avec injection et navigation |
| `flutter_command` | Chaque action est un objet `Command` observable |

---

## 52.33 — Tableau comparatif et conseils de choix

| Critère | `setState` | `ValueNotifier` | `provider` | Riverpod | Bloc | GetX |
| --- | --- | --- | --- | --- | --- | --- |
| Dépendance externe | non | non | oui | oui | oui | oui |
| Lignes pour un compteur | 5 | 8 | 12 | 12 | 20 | 8 |
| Partage entre écrans | non | difficile | oui | oui | oui | oui |
| Erreurs à la compilation | — | — | non | oui | partiel | non |
| Testabilité du modèle | faible | bonne | bonne | très bonne | excellente | moyenne |
| Traçabilité des changements | nulle | faible | faible | bonne | excellente | faible |
| Courbe d'apprentissage | nulle | faible | faible | moyenne | forte | faible |
| Suit les conventions Flutter | oui | oui | oui | oui | oui | non |
| Adapté à une grande équipe | non | non | moyen | oui | oui | non |

---

## 52.33.1 — L'arbre de décision

```text
  Votre donnée est-elle utilisée par un seul widget ?
     OUI -> setState. Arrêtez ici.
     NON -> continuez.

  Est-elle utilisée par 2 ou 3 widgets d'un même sous-arbre ?
     OUI -> remontée d'état, ou ValueNotifier + ValueListenableBuilder.
     NON -> continuez.

  Est-ce un projet personnel ou une application de moins de 15 écrans ?
     OUI -> provider. C'est le meilleur rapport simplicité / puissance.
     NON -> continuez.

  L'équipe compte-t-elle plus de 3 développeurs, ou l'application
  dépasse-t-elle 30 écrans ?
     OUI -> Riverpod (souple) ou Bloc (discipliné).
     NON -> provider convient encore très bien.
```

---

## 52.33.2 — Trois conseils qui valent pour toutes les solutions

**Conseil 1 : ne mélangez pas deux solutions de gestion d'état dans un même
projet.** Un projet moitié `provider`, moitié GetX est un projet que personne ne
peut relire. Choisissez, et tenez.

**Conseil 2 : la solution la plus importante reste `setState`.** Dans une
application bien architecturée avec `provider`, la majorité des `StatefulWidget`
gardent leur `setState` local pour l'état éphémère : animation, champ replié,
onglet actif. `provider` ne remplace pas `setState`, il le complète.

**Conseil 3 : ce qui compte, c'est la séparation modèle / interface.** Si vos
classes d'état sont propres, testées, et sans import de `material.dart`, changer de
solution de gestion d'état est un travail d'une journée. Si elles sont mélangées à
l'interface, c'est un travail d'un mois.

---

## 52.34 — Ce qu'il faut retenir : commencez simple

Le message central de ce chapitre tient en une phrase :

> **Commencez avec `setState`. Passez à `provider` quand `setState` ne suffit plus,
> pas avant.**

Le sur-dimensionnement est un piège plus courant que le sous-dimensionnement. Voici
les trois symptômes d'une architecture trop lourde :

```text
  1. Un ChangeNotifier "AppState" unique qui contient 30 champs,
     et que tous les widgets écoutent en entier.

  2. Un provider pour une donnée utilisée par un seul widget.

  3. Une fonctionnalité de 10 lignes qui demande de modifier
     5 fichiers d'architecture.
```

Le symptôme 1 est le plus commun. Un `ChangeNotifier` géant, souvent appelé
`AppState` ou `GlobalState`, reconstruit toute l'application dès qu'un champ change.
On appelle cela un **objet-dieu** (*god object*), et c'est exactement le problème
que `provider` était censé résoudre.

La bonne granularité :

```text
  MAUVAIS                        BON
  ───────                        ───
  AppState                       Utilisateur      (session, profil)
   ├ utilisateur                 Panier           (achats en cours)
   ├ panier                      Parametres       (thème, langue)
   ├ theme                       Notifications    (messages non lus)
   ├ langue
   ├ notifications
   ├ historique
   └ ... 25 autres champs
```

Découpez par **domaine fonctionnel**, pas par écran. Un objet d'état qui contient
« tout ce dont l'écran d'accueil a besoin » est mal découpé : le jour où l'écran
d'accueil change, tout casse.

---

## 52.34.1 — La checklist finale

Avant de valider une architecture d'état, vérifiez ces huit points :

```text
  [ ] Chaque donnée a UNE source de vérité, et une seule.
  [ ] Les classes d'état n'importent pas material.dart.
  [ ] Les champs sont privés, les accesseurs en lecture seule.
  [ ] Toute méthode qui modifie l'état appelle notifyListeners() UNE fois.
  [ ] Les collections exposées sont non modifiables.
  [ ] watch/select dans build, read dans les callbacks.
  [ ] Les objets d'état sont découpés par domaine, pas en un objet-dieu.
  [ ] Les classes d'état ont des tests unitaires.
```

Si les huit cases sont cochées, votre application tiendra la charge, quelle que soit
la bibliothèque choisie.

---

## 52.35 — Mini-projet : une application de gestion d'inventaire avec état partagé entre trois écrans

Nous allons construire une application complète de gestion d'inventaire pour un jeu
de rôle, avec trois écrans qui partagent le même état.

**Cahier des charges :**

```text
  ÉCRAN 1 — Inventaire
     liste des objets possédés, avec quantité
     poids total / capacité maximale
     bouton pour équiper ou déséquiper
     bouton pour jeter un objet

  ÉCRAN 2 — Boutique
     liste des objets achetables
     achat si l'or et la capacité le permettent
     revente d'un objet possédé à la moitié du prix

  ÉCRAN 3 — Personnage
     or disponible
     poids porté
     objets équipés et bonus d'attaque et de défense

  ÉTAT PARTAGÉ
     l'or, les objets possédés et les objets équipés
     doivent être identiques sur les trois écrans à tout instant
```

**Architecture retenue :**

```text
  Personnage (ChangeNotifier)   -> or, objets possédés, objets équipés
  Boutique   (Provider simple)  -> catalogue figé

  MaterialApp
   └── EcranPrincipal (NavigationBar, IndexedStack)
        ├── OngletInventaire
        ├── OngletBoutique
        └── OngletPersonnage
```

Nous utilisons un `IndexedStack` (chapitre 46) plutôt qu'une reconstruction
conditionnelle : les trois onglets restent montés, ce qui préserve leur état
éphémère — mais l'état partagé, lui, vient du `ChangeNotifier` et serait préservé de
toute façon.

Voici le programme complet.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        Provider<Boutique>(create: (_) => Boutique()),
        ChangeNotifierProvider<Personnage>(create: (_) => Personnage()),
      ],
      child: const ApplicationInventaire(),
    ),
  );
}

// ═════════════════════════════════════════════════════════════
// MODÈLES
// ═════════════════════════════════════════════════════════════

enum Categorie { arme, armure, consommable }

class Objet {
  const Objet({
    required this.id,
    required this.nom,
    required this.prix,
    required this.poids,
    required this.categorie,
    required this.icone,
    this.attaque = 0,
    this.defense = 0,
  });

  final String id;
  final String nom;
  final int prix;
  final int poids;
  final Categorie categorie;
  final IconData icone;
  final int attaque;
  final int defense;

  bool get estEquipable => categorie != Categorie.consommable;

  @override
  bool operator ==(Object other) => other is Objet && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

// Catalogue figé : aucune notification nécessaire.
class Boutique {
  final List<Objet> catalogue = const <Objet>[
    Objet(
      id: 'o1',
      nom: 'Dague rouillée',
      prix: 40,
      poids: 2,
      categorie: Categorie.arme,
      icone: Icons.hardware,
      attaque: 3,
    ),
    Objet(
      id: 'o2',
      nom: 'Épée longue',
      prix: 180,
      poids: 6,
      categorie: Categorie.arme,
      icone: Icons.gavel,
      attaque: 12,
    ),
    Objet(
      id: 'o3',
      nom: 'Tunique de cuir',
      prix: 90,
      poids: 5,
      categorie: Categorie.armure,
      icone: Icons.checkroom,
      defense: 6,
    ),
    Objet(
      id: 'o4',
      nom: 'Cotte de mailles',
      prix: 260,
      poids: 14,
      categorie: Categorie.armure,
      icone: Icons.shield,
      defense: 15,
    ),
    Objet(
      id: 'o5',
      nom: 'Potion de soin',
      prix: 25,
      poids: 1,
      categorie: Categorie.consommable,
      icone: Icons.local_drink,
    ),
    Objet(
      id: 'o6',
      nom: 'Élixir de force',
      prix: 75,
      poids: 1,
      categorie: Categorie.consommable,
      icone: Icons.science,
    ),
  ];
}

// L'ÉTAT PARTAGÉ.
class Personnage extends ChangeNotifier {
  Personnage({this.capacite = 40});

  final int capacite;

  int _or = 300;
  final Map<Objet, int> _possedes = <Objet, int>{};
  final Set<Objet> _equipes = <Objet>{};

  // ── Lecture seule ────────────────────────────────────────
  int get or => _or;
  int get capaciteMax => capacite;
  Map<Objet, int> get possedes => Map<Objet, int>.unmodifiable(_possedes);
  Set<Objet> get equipes => Set<Objet>.unmodifiable(_equipes);

  int get poidsPorte => _possedes.entries
      .fold(0, (int s, MapEntry<Objet, int> e) => s + e.key.poids * e.value);

  int get poidsRestant => capacite - poidsPorte;

  int get attaque =>
      _equipes.fold(0, (int s, Objet o) => s + o.attaque);

  int get defense =>
      _equipes.fold(0, (int s, Objet o) => s + o.defense);

  int quantiteDe(Objet o) => _possedes[o] ?? 0;
  bool estEquipe(Objet o) => _equipes.contains(o);

  bool peutAcheter(Objet o) => _or >= o.prix && poidsRestant >= o.poids;

  // ── Modification ─────────────────────────────────────────
  bool acheter(Objet o) {
    if (!peutAcheter(o)) return false;
    _or -= o.prix;
    _possedes[o] = (_possedes[o] ?? 0) + 1;
    notifyListeners();
    return true;
  }

  bool revendre(Objet o) {
    if (quantiteDe(o) == 0) return false;
    _or += o.prix ~/ 2;
    _retirerUnExemplaire(o);
    notifyListeners();
    return true;
  }

  bool jeter(Objet o) {
    if (quantiteDe(o) == 0) return false;
    _retirerUnExemplaire(o);
    notifyListeners();
    return true;
  }

  void basculerEquipement(Objet o) {
    if (!o.estEquipable || quantiteDe(o) == 0) return;
    if (_equipes.contains(o)) {
      _equipes.remove(o);
    } else {
      _equipes.add(o);
    }
    notifyListeners();
  }

  // Méthode privée : elle ne notifie PAS, l'appelant s'en charge.
  void _retirerUnExemplaire(Objet o) {
    final int actuel = _possedes[o] ?? 0;
    if (actuel <= 1) {
      _possedes.remove(o);
      _equipes.remove(o);
    } else {
      _possedes[o] = actuel - 1;
    }
  }
}

// ═════════════════════════════════════════════════════════════
// INTERFACE
// ═════════════════════════════════════════════════════════════

class ApplicationInventaire extends StatelessWidget {
  const ApplicationInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Inventaire',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
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
  // État ÉPHÉMÈRE : l'onglet actif. setState suffit.
  int _onglet = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Aventurier'),
        actions: const <Widget>[BandeauOr(), SizedBox(width: 8)],
      ),
      body: IndexedStack(
        index: _onglet,
        children: const <Widget>[
          OngletInventaire(),
          OngletBoutique(),
          OngletPersonnage(),
        ],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _onglet,
        onDestinationSelected: (int i) => setState(() => _onglet = i),
        destinations: const <NavigationDestination>[
          NavigationDestination(
              icon: Icon(Icons.backpack), label: 'Inventaire'),
          NavigationDestination(icon: Icon(Icons.store), label: 'Boutique'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Personnage'),
        ],
      ),
    );
  }
}

// S'abonne au SEUL or.
class BandeauOr extends StatelessWidget {
  const BandeauOr({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BandeauOr');
    final int or = context.select<Personnage, int>((Personnage p) => p.or);
    return Center(
      child: Row(
        children: <Widget>[
          const Icon(Icons.monetization_on),
          const SizedBox(width: 4),
          Text('$or', style: const TextStyle(fontSize: 18)),
        ],
      ),
    );
  }
}

// ───────────────────────── ONGLET 1 ──────────────────────────
class OngletInventaire extends StatelessWidget {
  const OngletInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        const JaugePoids(),
        Expanded(
          child: Consumer<Personnage>(
            builder:
                (BuildContext context, Personnage perso, Widget? enfant) {
              debugPrint('build liste inventaire');
              final List<MapEntry<Objet, int>> lignes =
                  perso.possedes.entries.toList();
              if (lignes.isEmpty) {
                return const Center(
                  child: Text('Sac vide. Passez à la boutique.'),
                );
              }
              return ListView.separated(
                itemCount: lignes.length,
                separatorBuilder: (BuildContext c, int i) =>
                    const Divider(height: 1),
                itemBuilder: (BuildContext context, int index) {
                  final Objet objet = lignes[index].key;
                  final int quantite = lignes[index].value;
                  final bool equipe = perso.estEquipe(objet);
                  return ListTile(
                    leading: CircleAvatar(
                      backgroundColor: equipe
                          ? Theme.of(context).colorScheme.primaryContainer
                          : null,
                      child: Icon(objet.icone),
                    ),
                    title: Text('${objet.nom}  x$quantite'),
                    subtitle: Text(
                      'Poids ${objet.poids} '
                      '${objet.attaque > 0 ? '| ATT +${objet.attaque} ' : ''}'
                      '${objet.defense > 0 ? '| DEF +${objet.defense}' : ''}',
                    ),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: <Widget>[
                        if (objet.estEquipable)
                          IconButton(
                            tooltip: equipe ? 'Déséquiper' : 'Équiper',
                            icon: Icon(
                              equipe ? Icons.check_circle : Icons.circle_outlined,
                            ),
                            onPressed: () => context
                                .read<Personnage>()
                                .basculerEquipement(objet),
                          ),
                        IconButton(
                          tooltip: 'Jeter',
                          icon: const Icon(Icons.delete_outline),
                          onPressed: () =>
                              context.read<Personnage>().jeter(objet),
                        ),
                      ],
                    ),
                  );
                },
              );
            },
          ),
        ),
      ],
    );
  }
}

class JaugePoids extends StatelessWidget {
  const JaugePoids({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build JaugePoids');
    final int porte =
        context.select<Personnage, int>((Personnage p) => p.poidsPorte);
    final int max =
        context.select<Personnage, int>((Personnage p) => p.capaciteMax);
    final double taux = max == 0 ? 0 : porte / max;
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Text('Charge : $porte / $max'),
          const SizedBox(height: 6),
          LinearProgressIndicator(
            value: taux > 1 ? 1 : taux,
            color: taux > 0.9 ? Colors.red : null,
          ),
        ],
      ),
    );
  }
}

// ───────────────────────── ONGLET 2 ──────────────────────────
class OngletBoutique extends StatelessWidget {
  const OngletBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    final Boutique boutique = context.read<Boutique>();
    return ListView.separated(
      itemCount: boutique.catalogue.length,
      separatorBuilder: (BuildContext c, int i) => const Divider(height: 1),
      itemBuilder: (BuildContext context, int index) {
        return LigneBoutique(objet: boutique.catalogue[index]);
      },
    );
  }
}

class LigneBoutique extends StatelessWidget {
  const LigneBoutique({super.key, required this.objet});

  final Objet objet;

  @override
  Widget build(BuildContext context) {
    debugPrint('build LigneBoutique ${objet.nom}');
    // Deux sélections ciblées : cette ligne ne se reconstruit
    // que si SA quantité ou SA capacité d'achat change.
    final int possede =
        context.select<Personnage, int>((Personnage p) => p.quantiteDe(objet));
    final bool achetable = context
        .select<Personnage, bool>((Personnage p) => p.peutAcheter(objet));

    return ListTile(
      leading: CircleAvatar(child: Icon(objet.icone)),
      title: Text(objet.nom),
      subtitle: Text('${objet.prix} or | poids ${objet.poids}'
          '${possede > 0 ? ' | possédé x$possede' : ''}'),
      trailing: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          if (possede > 0)
            TextButton(
              onPressed: () => context.read<Personnage>().revendre(objet),
              child: Text('Vendre ${objet.prix ~/ 2}'),
            ),
          FilledButton(
            onPressed: achetable
                ? () {
                    context.read<Personnage>().acheter(objet);
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(
                        content: Text('${objet.nom} acheté.'),
                        duration: const Duration(milliseconds: 900),
                      ),
                    );
                  }
                : null,
            child: const Text('Acheter'),
          ),
        ],
      ),
    );
  }
}

// ───────────────────────── ONGLET 3 ──────────────────────────
class OngletPersonnage extends StatelessWidget {
  const OngletPersonnage({super.key});

  @override
  Widget build(BuildContext context) {
    return Consumer<Personnage>(
      builder: (BuildContext context, Personnage perso, Widget? enfant) {
        debugPrint('build OngletPersonnage');
        return ListView(
          padding: const EdgeInsets.all(16),
          children: <Widget>[
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text('Caractéristiques',
                        style: Theme.of(context).textTheme.titleLarge),
                    const SizedBox(height: 12),
                    _Ligne(libelle: 'Or', valeur: '${perso.or}'),
                    _Ligne(
                        libelle: 'Charge',
                        valeur: '${perso.poidsPorte} / ${perso.capaciteMax}'),
                    _Ligne(libelle: 'Attaque', valeur: '${perso.attaque}'),
                    _Ligne(libelle: 'Défense', valeur: '${perso.defense}'),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),
            Text('Équipement porté',
                style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 8),
            if (perso.equipes.isEmpty)
              const Padding(
                padding: EdgeInsets.all(16),
                child: Text('Aucun équipement porté.'),
              )
            else
              ...perso.equipes.map(
                (Objet o) => ListTile(
                  leading: Icon(o.icone),
                  title: Text(o.nom),
                  subtitle: Text(
                    'ATT +${o.attaque} | DEF +${o.defense}',
                  ),
                ),
              ),
          ],
        );
      },
    );
  }
}

class _Ligne extends StatelessWidget {
  const _Ligne({required this.libelle, required this.valeur});

  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: <Widget>[
          Text(libelle),
          Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│ Aventurier                  (or) 300   │
├────────────────────────────────────────┤
│ Charge : 0 / 40                        │
│ [░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░]       │
│                                        │
│      Sac vide. Passez à la boutique.   │
│                                        │
├────────────────────────────────────────┤
│  Inventaire    Boutique    Personnage  │
└────────────────────────────────────────┘

Après achat d'une Épée longue et d'une Tunique de cuir :

┌────────────────────────────────────────┐
│ Aventurier                  (or) 30    │
├────────────────────────────────────────┤
│ Charge : 11 / 40                       │
│ [████████░░░░░░░░░░░░░░░░░░░░░░]       │
│ (o) Épée longue  x1     (v)  (suppr)   │
│     Poids 6 | ATT +12                  │
│ (o) Tunique de cuir x1  ( )  (suppr)   │
│     Poids 5 | DEF +6                   │
└────────────────────────────────────────┘

Onglet Personnage, avec l'épée équipée :

Or       : 30
Charge   : 11 / 40
Attaque  : 12
Défense  : 0
```

---

## 52.35.1 — Ce que ce projet démontre

| Point | Où le voir |
| --- | --- |
| Un état partagé par trois écrans, sans forage | `Personnage` lu depuis chaque onglet |
| État éphémère et état d'application cohabitent | `_onglet` en `setState`, le reste en provider |
| Un catalogue figé n'a pas besoin de `ChangeNotifier` | `Provider<Boutique>` |
| `select` limite les reconstructions | `BandeauOr`, `JaugePoids`, `LigneBoutique` |
| `Consumer` délimite une zone | liste d'inventaire, onglet personnage |
| `read` pour agir | tous les `onPressed` |
| Les collections sont protégées | `Map.unmodifiable`, `Set.unmodifiable` |
| Les valeurs dérivées sont calculées | `attaque`, `defense`, `poidsPorte`, `poidsRestant` |
| Une méthode privée qui ne notifie pas | `_retirerUnExemplaire` |

Ce dernier point mérite une explication. `_retirerUnExemplaire` est appelée par
`revendre` et par `jeter`. Si elle appelait elle-même `notifyListeners()`, chaque
revente déclencherait **deux** notifications. En faisant notifier l'appelant, on
garantit une notification par action utilisateur.

---

## 52.35.2 — Extensions à réaliser seul

1. Ajouter un objet ne peut se faire que si `peutAcheter` est vrai. Ajoutez un
   `SnackBar` explicite en cas de refus, en distinguant « pas assez d'or » et
   « trop lourd ».
2. Empêchez d'équiper deux armes en même temps : équiper une arme doit déséquiper
   l'arme précédente.
3. Ajoutez un onglet « Journal » qui affiche l'historique des dix dernières actions
   (achat, vente, équipement), avec un second `ChangeNotifier`.
4. Ajoutez un `ProxyProvider` qui calcule une classe `Puissance` à partir du
   `Personnage`, avec un score `attaque * 2 + defense`.
5. Écrivez les tests unitaires de `Personnage` : achat impossible sans or, achat
   impossible si trop lourd, revente à la moitié du prix, déséquipement automatique
   quand le dernier exemplaire est jeté.

---

## 52.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `context.read<T>()` dans `build()` pour afficher une valeur : l'écran reste figé, aucun message | `read` ne crée aucun abonnement | Utiliser `context.watch<T>()`, `context.select<T, R>()` ou `Consumer<T>` |
| `context.watch<T>()` dans un `onPressed` : `Tried to listen to a value exposed with provider, from outside of the widget tree.` | On ne peut s'abonner que pendant la construction | Utiliser `context.read<T>()` dans les callbacks |
| `context.select` hors de `build` : `Tried to use context.select outside of the build method of a widget.` | `select` a besoin de la phase de construction | Déplacer l'appel dans `build`, ou utiliser `read` |
| `notifyListeners()` oublié : la donnée change en mémoire, l'écran ne bouge pas | Aucun écouteur n'est prévenu | Ajouter `notifyListeners()` à la fin de chaque méthode qui modifie l'état |
| `panier.articles.add(a)` depuis l'extérieur : rien ne se met à jour | La collection interne est mutée sans notification | Exposer `List.unmodifiable(...)` et passer par une méthode du modèle |
| `Unsupported operation: Cannot add to an unmodifiable list` | Tentative de mutation d'une vue non modifiable | C'est le comportement voulu : appeler la méthode du modèle |
| `Error: Could not find the correct Provider<Joueur> above this HudScore Widget` | Le widget n'est pas sous le provider, ou le provider est dans une autre route | Remonter le provider au-dessus de `MaterialApp`, ou réexposer avec `.value` dans la route |
| Même message après avoir ajouté un provider et fait un hot reload | Un provider ajouté n'est pas pris en compte par le hot reload | Faire un **hot restart** (`R` majuscule) |
| `context.watch<T>()` juste sous le `child:` du provider qu'on vient de créer : `ProviderNotFoundException` | Le `context` de `build` est l'ancêtre du provider | Extraire un widget enfant, ou utiliser le paramètre `builder:` du provider |
| `ChangeNotifierProvider<Joueur> used outside of MultiProvider must specify a child` | Provider déclaré sans `child` ni `builder`, hors d'un `MultiProvider` | Ajouter `child:` ou le placer dans la liste `providers:` d'un `MultiProvider` |
| `ChangeNotifierProvider.value(value: Joueur())` : l'état repart de zéro à chaque reconstruction | `.value` reçoit une nouvelle instance à chaque `build` | Utiliser `create: (_) => Joueur()`, qui n'instancie qu'une fois |
| `A Joueur was used after being disposed. Once you have called dispose() on a Joueur, it can no longer be used.` | Le notifier a été libéré alors qu'il est encore utilisé | Ne pas appeler `dispose()` soi-même sur un objet géré par `create:` |
| `setState() called after dispose(): _EcranJoueurState#a1b2c(lifecycle state: defunct, not mounted)` | Un écouteur pointe encore vers un `State` détruit | Appeler `removeListener` dans `dispose()`, ou tester `if (!mounted) return;` |
| `setState() or markNeedsBuild() called during build.` | Une méthode du modèle est appelée depuis `build()` | Déplacer l'appel dans un callback, ou dans `addPostFrameCallback` |
| `dependOnInheritedWidgetOfExactType<ScoreHeritee>() or dependOnInheritedElement() was called before _MonEtatState.initState() completed.` | Lecture d'un `InheritedWidget` trop tôt | Lire dans `didChangeDependencies()`, ou utiliser `context.read` |
| `updateShouldNotify` qui renvoie toujours `false` : l'affichage ne se met jamais à jour, sans erreur | Les abonnés ne sont jamais prévenus | Comparer réellement les champs : `return score != oldWidget.score;` |
| `context.select<Joueur, List<String>>((j) => j.inventaire)` ne déclenche jamais de reconstruction | La liste est le même objet, `==` renvoie `true` | Sélectionner une valeur scalaire (`.length`) ou une classe qui redéfinit `==` |
| Deux `Provider<String>` dans le même `MultiProvider` : une seule valeur est visible | Le provider le plus proche masque l'autre | Envelopper chaque donnée dans un type dédié |
| `Provider.of<T>(context)` dans `initState` : `Tried to listen to an InheritedWidget in a life-cycle that will never be called again.` | `initState` n'est appelée qu'une fois, l'abonnement n'aurait aucun effet | Utiliser `context.read<T>()` |
| L'application entière se reconstruit à chaque changement | Un unique `ChangeNotifier` « AppState » écouté partout avec `watch` | Découper par domaine, et utiliser `select` ou `Consumer` |

---

## 52.37 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Gestion d'état | Répondre à trois questions : où vit la donnée, qui la modifie, qui est prévenu |
| État éphémère | Un seul widget concerné : `setState` suffit |
| État d'application | Plusieurs widgets ou plusieurs écrans concernés : il faut le partager |
| Source de vérité | Une donnée n'existe qu'à un seul endroit |
| Limites de `setState` | Reconstruit tout le `build`, inaccessible aux enfants, meurt avec le widget |
| Remontée d'état | La donnée vit dans le plus proche ancêtre commun |
| Forage de propriétés | 3 x (N - 1) lignes inutiles pour traverser N niveaux |
| `InheritedWidget` | Pose une valeur dans l'arbre ; recherche en O(1) par table de hachage |
| `dependOnInheritedWidgetOfExactType` | Lit **et** abonne ; type exact, pas les sous-classes |
| `updateShouldNotify` | Décide si les abonnés doivent être reconstruits |
| `Theme.of(context)` | Un `InheritedWidget` : `.of(context)` est la signature du mécanisme |
| `ChangeNotifier` | Cinq membres : `hasListeners`, `addListener`, `removeListener`, `notifyListeners`, `dispose` |
| `notifyListeners()` | Après la modification, une seule fois, jamais dans un accesseur |
| `ValueNotifier` | Une seule valeur ; l'affectation notifie si `==` diffère |
| `ValueListenableBuilder` | Reconstruit uniquement son `builder` ; `child` sort le sous-arbre coûteux |
| `ListenableBuilder` | Même principe pour n'importe quel `Listenable` |
| `provider` | Une enveloppe autour d'`InheritedWidget` ; version courante `6.1.5+1` |
| `ChangeNotifierProvider` | `create:` instancie et libère automatiquement |
| `context.watch<T>()` | Lit et s'abonne ; uniquement dans `build` |
| `context.read<T>()` | Lit sans s'abonner ; dans les callbacks |
| `context.select<T, R>()` | S'abonne à une valeur dérivée comparable par `==` |
| `Consumer<T>` | Délimite la zone reconstruite ; son `context` est sous le provider |
| Règle d'or | `watch` pour afficher, `read` pour agir |
| `MultiProvider` | Aplatit l'imbrication ; l'ordre compte, les dépendances d'abord |
| `Provider.value` | Uniquement pour un objet existant ; c'est vous qui appelez `dispose()` |
| `ProxyProvider` | Reconstruit un objet quand un autre provider change |
| `FutureProvider` / `StreamProvider` | `initialData` obligatoire ; pas d'état de chargement ni d'erreur |
| Séparation modèle / interface | Un modèle n'importe jamais `material.dart` |
| Tests | Un `ChangeNotifier` se teste sans interface ; comptez les notifications |
| Choix d'outil | `setState` par défaut, `provider` quand il ne suffit plus, Riverpod ou Bloc en grande équipe |
| Objet-dieu | Un `ChangeNotifier` de 30 champs reconstruit toute l'application : découpez par domaine |

---

## 52.38 — Exercices

### Exercice 1 — La barre d'énergie (facile)

Écrivez un `main.dart` complet qui affiche une énergie de 100 %, une
`LinearProgressIndicator`, et deux boutons « Courir » (-10) et « Se reposer »
(+20, plafonné à 100).

Contraintes : utilisez un `ValueNotifier<int>` et un `ValueListenableBuilder<int>`.
Le `build()` de l'écran ne doit s'exécuter **qu'une seule fois** ; prouvez-le avec
un `debugPrint`.

### Exercice 2 — Du `setState` au `ChangeNotifier` (facile)

Écrivez une classe `Heros extends ChangeNotifier` avec un score, des vies et une
méthode `subirDegats(int)` qui retire une vie si les dégâts dépassent 50.

Affichez le tout avec un `ListenableBuilder`, sans aucune dépendance externe. Les
boutons ne doivent pas être reconstruits.

### Exercice 3 — Un `InheritedWidget` maison (facile)

Écrivez un `InheritedWidget` nommé `NomDuJoueur` qui porte un `String`. Placez-le
au sommet d'un arbre de trois niveaux, et affichez le nom depuis le niveau le plus
profond, sans aucun paramètre de constructeur intermédiaire.

Ajoutez un champ de saisie dans un `StatefulWidget` parent pour changer le nom, et
vérifiez que seul le widget profond se reconstruit.

### Exercice 4 — Premier provider (moyen)

Reprenez l'exercice 2 avec `provider` : `ChangeNotifierProvider` au-dessus de
`MaterialApp`, `context.watch` pour afficher, `context.read` pour agir.

Ajoutez volontairement un `context.read` à la place du `watch` dans l'affichage,
constatez le bug, puis corrigez-le.

### Exercice 5 — `watch` contre `select` (moyen)

Créez un `ChangeNotifier` avec trois champs (`or`, `bois`, `pierre`) et trois
boutons qui les incrémentent séparément.

Affichez chaque ressource dans un widget distinct, l'un avec `watch`, les deux
autres avec `select`. Ajoutez un `debugPrint` dans chaque `build` et notez dans un
commentaire le nombre de reconstructions observées pour dix appuis.

### Exercice 6 — `Consumer` et son `child` (moyen)

Écrivez un écran contenant un widget « lourd » (une `Column` de 50 `Text` générés
par une boucle) et un compteur.

Version A : `context.watch` en haut de `build`.
Version B : `Consumer` avec le widget lourd passé en `child`.

Mettez un `debugPrint` dans le constructeur du widget lourd et comparez.

### Exercice 7 — `MultiProvider` (moyen)

Créez deux modèles : `Joueur` (score) et `Preferences` (booléen `modeSombre`).

Fournissez-les avec un `MultiProvider`, et faites en sorte que `MaterialApp` change
réellement de thème quand `modeSombre` bascule. Le bouton de bascule doit être dans
un écran enfant, pas dans le `MaterialApp`.

### Exercice 8 — `ProxyProvider` (difficile)

Créez `Joueur` (avec un `niveau`) et une classe simple `Difficulte` qui, à partir du
niveau, calcule le nombre de points de vie des ennemis
(`50 + niveau * 25`) et leur vitesse (`1.0 + niveau * 0.15`).

Reliez-les avec un `ProxyProvider<Joueur, Difficulte>` et affichez les trois valeurs.

### Exercice 9 — Panier et navigation (difficile)

Écrivez une application à deux écrans : un catalogue de quatre objets et un écran de
détail atteint par `Navigator.push`.

L'écran de détail doit afficher la quantité déjà au panier et permettre d'ajouter.
Le badge du catalogue doit se mettre à jour en revenant. Utilisez `Provider.value`
pour exposer l'objet sélectionné à l'écran de détail.

### Exercice 10 — Asynchrone et état combinés (difficile)

Écrivez une application qui :

- charge le nom d'un monde avec un `FutureProvider` (2 secondes de délai) ;
- reçoit un flux de dégâts avec un `StreamProvider` (un entier toutes les
  1,5 seconde) ;
- possède un `ChangeNotifier` `Heros` avec des points de vie ;
- applique automatiquement chaque dégât reçu au héros, sans jamais appeler
  `notifyListeners()` depuis un `build()`.

Indication : enveloppez les valeurs asynchrones dans des types dédiés, et utilisez
`ChangeNotifierProxyProvider` ou un `ListenableBuilder` combiné à un
`addPostFrameCallback`.

---

## 52.39 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationEnergie());
}

class ApplicationEnergie extends StatelessWidget {
  const ApplicationEnergie({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      home: const EcranEnergie(),
    );
  }
}

class EcranEnergie extends StatefulWidget {
  const EcranEnergie({super.key});

  @override
  State<EcranEnergie> createState() => _EcranEnergieState();
}

class _EcranEnergieState extends State<EcranEnergie> {
  final ValueNotifier<int> _energie = ValueNotifier<int>(100);

  void _courir() {
    final int nouvelle = _energie.value - 10;
    _energie.value = nouvelle < 0 ? 0 : nouvelle;
  }

  void _seReposer() {
    final int nouvelle = _energie.value + 20;
    _energie.value = nouvelle > 100 ? 100 : nouvelle;
  }

  @override
  void dispose() {
    _energie.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build de EcranEnergie');
    return Scaffold(
      appBar: AppBar(title: const Text('Énergie')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ValueListenableBuilder<int>(
              valueListenable: _energie,
              builder: (BuildContext context, int valeur, Widget? enfant) {
                debugPrint('  build du bloc énergie : $valeur');
                return Column(
                  children: <Widget>[
                    Text('Énergie : $valeur %',
                        style: const TextStyle(fontSize: 28)),
                    const SizedBox(height: 12),
                    LinearProgressIndicator(
                      value: valeur / 100,
                      minHeight: 12,
                      color: valeur < 30 ? Colors.red : Colors.green,
                    ),
                  ],
                );
              },
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                ElevatedButton(
                  onPressed: _courir,
                  child: const Text('Courir (-10)'),
                ),
                ElevatedButton(
                  onPressed: _seReposer,
                  child: const Text('Se reposer (+20)'),
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

**Explication :** `build de EcranEnergie` n'apparaît qu'une fois dans la console,
au démarrage. Les boutons n'appellent jamais `setState` : ils affectent
`_energie.value`, ce qui déclenche la notification interne du `ValueNotifier`. Seul
le `builder` du `ValueListenableBuilder` est réexécuté. Notez le plafonnement : sans
lui, l'énergie dépasserait 100 et `LinearProgressIndicator` recevrait une `value`
supérieure à 1, ce qui n'est pas accepté. Le `dispose()` du notifier est
obligatoire, sinon l'objet reste enregistré et fuit.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationHeros());
}

// ── MODÈLE ────────────────────────────────────────────────────
class Heros extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;
  String _dernierMessage = 'Prêt au combat.';

  int get score => _score;
  int get vies => _vies;
  String get dernierMessage => _dernierMessage;
  bool get estMort => _vies <= 0;

  void ramasser(int points) {
    if (estMort) return;
    _score += points;
    _dernierMessage = '+$points points';
    notifyListeners();
  }

  void subirDegats(int degats) {
    if (estMort) return;
    if (degats > 50) {
      _vies--;
      _dernierMessage = 'Coup critique de $degats : une vie perdue !';
    } else {
      _dernierMessage = 'Coup encaissé de $degats.';
    }
    notifyListeners();
  }
}

// ── INTERFACE ─────────────────────────────────────────────────
class ApplicationHeros extends StatelessWidget {
  const ApplicationHeros({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      home: const EcranHeros(),
    );
  }
}

class EcranHeros extends StatefulWidget {
  const EcranHeros({super.key});

  @override
  State<EcranHeros> createState() => _EcranHerosState();
}

class _EcranHerosState extends State<EcranHeros> {
  final Heros _heros = Heros();

  @override
  void dispose() {
    _heros.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build de EcranHeros');
    return Scaffold(
      appBar: AppBar(title: const Text('ChangeNotifier')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ListenableBuilder(
              listenable: _heros,
              builder: (BuildContext context, Widget? enfant) {
                debugPrint('  build du HUD');
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(20),
                    child: Column(
                      children: <Widget>[
                        Text('Score : ${_heros.score}',
                            style: const TextStyle(fontSize: 24)),
                        Text('Vies : ${_heros.vies}',
                            style: const TextStyle(fontSize: 24)),
                        const SizedBox(height: 8),
                        Text(_heros.dernierMessage),
                        if (_heros.estMort)
                          const Text('PARTIE TERMINÉE',
                              style: TextStyle(
                                  color: Colors.red,
                                  fontWeight: FontWeight.bold)),
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 32),
            Wrap(
              spacing: 12,
              runSpacing: 12,
              alignment: WrapAlignment.center,
              children: <Widget>[
                ElevatedButton(
                  onPressed: () => _heros.ramasser(50),
                  child: const Text('Ramasser 50'),
                ),
                ElevatedButton(
                  onPressed: () => _heros.subirDegats(30),
                  child: const Text('Dégâts 30'),
                ),
                ElevatedButton(
                  onPressed: () => _heros.subirDegats(80),
                  child: const Text('Dégâts 80'),
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

**Explication :** la console affiche `build de EcranHeros` une seule fois, puis
`build du HUD` à chaque action. Les boutons, situés hors du `ListenableBuilder`, ne
sont jamais reconstruits. La méthode `subirDegats` illustre la règle 3 de 52.15.1 :
elle notifie dans les deux branches parce que `_dernierMessage` change toujours ;
si seul le cas critique modifiait l'état, il faudrait notifier uniquement dans ce
cas. Le garde `if (estMort) return;` évite les modifications après la mort du héros
et, accessoirement, les notifications inutiles.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationNom());
}

// ── L'InheritedWidget ─────────────────────────────────────────
class NomDuJoueur extends InheritedWidget {
  const NomDuJoueur({
    super.key,
    required this.nom,
    required super.child,
  });

  final String nom;

  static NomDuJoueur? maybeOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<NomDuJoueur>();
  }

  static NomDuJoueur of(BuildContext context) {
    final NomDuJoueur? resultat = maybeOf(context);
    assert(resultat != null, 'Aucun NomDuJoueur dans le contexte');
    return resultat!;
  }

  @override
  bool updateShouldNotify(NomDuJoueur oldWidget) => nom != oldWidget.nom;
}

class ApplicationNom extends StatelessWidget {
  const ApplicationNom({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
        useMaterial3: true,
      ),
      home: const EcranRacine(),
    );
  }
}

class EcranRacine extends StatefulWidget {
  const EcranRacine({super.key});

  @override
  State<EcranRacine> createState() => _EcranRacineState();
}

class _EcranRacineState extends State<EcranRacine> {
  final TextEditingController _controleur =
      TextEditingController(text: 'Alex');
  String _nom = 'Alex';

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build EcranRacine');
    return NomDuJoueur(
      nom: _nom,
      child: Scaffold(
        appBar: AppBar(title: const Text('InheritedWidget maison')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: <Widget>[
              TextField(
                controller: _controleur,
                decoration: const InputDecoration(
                  labelText: 'Nom du joueur',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 12),
              ElevatedButton(
                onPressed: () => setState(() => _nom = _controleur.text),
                child: const Text('Valider'),
              ),
              const SizedBox(height: 24),
              const Expanded(child: Niveau1()),
            ],
          ),
        ),
      ),
    );
  }
}

// ── Trois niveaux, aucun paramètre ────────────────────────────
class Niveau1 extends StatelessWidget {
  const Niveau1({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  build Niveau1');
    return const Card(child: Padding(padding: EdgeInsets.all(8), child: Niveau2()));
  }
}

class Niveau2 extends StatelessWidget {
  const Niveau2({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  build Niveau2');
    return const Padding(padding: EdgeInsets.all(8), child: Niveau3());
  }
}

class Niveau3 extends StatelessWidget {
  const Niveau3({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('  build Niveau3 (abonné)');
    final NomDuJoueur donnees = NomDuJoueur.of(context);
    return Center(
      child: Text(
        'Bienvenue, ${donnees.nom} !',
        style: Theme.of(context).textTheme.headlineSmall,
      ),
    );
  }
}
```

**Explication :** au démarrage, les cinq `build` apparaissent. Après validation d'un
nouveau nom, la console n'affiche que `build EcranRacine` puis
`build Niveau3 (abonné)`. `Niveau1` et `Niveau2` ne se reconstruisent pas : ils sont
`const`, donc Flutter réutilise leurs instances, et ils n'ont pas appelé
`dependOnInheritedWidgetOfExactType`. Si vous retirez les `const`, ils seront
reconstruits — mais toujours pas parce qu'ils sont abonnés, seulement parce que leur
parent l'est. C'est la démonstration que le `const` du chapitre 44 et l'abonnement
du chapitre 52 sont deux optimisations complémentaires.

---

### Correction 4

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Heros>(
      create: (BuildContext context) => Heros(),
      child: const ApplicationProvider4(),
    ),
  );
}

class Heros extends ChangeNotifier {
  int _score = 0;
  int _vies = 3;

  int get score => _score;
  int get vies => _vies;
  bool get estMort => _vies <= 0;

  void ramasser(int points) {
    if (estMort) return;
    _score += points;
    notifyListeners();
  }

  void subirDegats(int degats) {
    if (estMort) return;
    if (degats > 50) {
      _vies--;
      notifyListeners();
    }
  }
}

class ApplicationProvider4 extends StatelessWidget {
  const ApplicationProvider4({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const EcranHeros4(),
    );
  }
}

class EcranHeros4 extends StatelessWidget {
  const EcranHeros4({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Premier provider')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            HudHeros(),
            SizedBox(height: 32),
            BoutonsHeros(),
          ],
        ),
      ),
    );
  }
}

// AFFICHE -> watch
class HudHeros extends StatelessWidget {
  const HudHeros({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build HudHeros');
    // ERREUR À NE PAS FAIRE :
    // final Heros heros = context.read<Heros>();  -> l'écran resterait figé.
    final Heros heros = context.watch<Heros>();
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: <Widget>[
            Text('Score : ${heros.score}', style: const TextStyle(fontSize: 24)),
            Text('Vies : ${heros.vies}', style: const TextStyle(fontSize: 24)),
            if (heros.estMort)
              const Text('PARTIE TERMINÉE',
                  style: TextStyle(
                      color: Colors.red, fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}

// AGIT -> read
class BoutonsHeros extends StatelessWidget {
  const BoutonsHeros({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BoutonsHeros');
    return Wrap(
      spacing: 12,
      alignment: WrapAlignment.center,
      children: <Widget>[
        ElevatedButton(
          onPressed: () => context.read<Heros>().ramasser(50),
          child: const Text('Ramasser 50'),
        ),
        ElevatedButton(
          onPressed: () => context.read<Heros>().subirDegats(30),
          child: const Text('Dégâts 30'),
        ),
        ElevatedButton(
          onPressed: () => context.read<Heros>().subirDegats(80),
          child: const Text('Dégâts 80'),
        ),
      ],
    );
  }
}
```

**Explication :** la console affiche `build HudHeros` à chaque action, et
`build BoutonsHeros` **une seule fois**, au démarrage. C'est la preuve visible de la
règle de 52.24 : `watch` abonne, `read` non. Si vous remplacez le `watch` de
`HudHeros` par un `read`, l'application compile, s'exécute, ne produit aucune erreur,
et le score reste bloqué à zéro. Notez aussi que le bouton « Dégâts 30 » ne provoque
aucune reconstruction : `subirDegats(30)` ne modifie rien et ne notifie donc pas.

---

### Correction 5

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Ressources>(
      create: (_) => Ressources(),
      child: const ApplicationRessources(),
    ),
  );
}

class Ressources extends ChangeNotifier {
  int _or = 0;
  int _bois = 0;
  int _pierre = 0;

  int get or => _or;
  int get bois => _bois;
  int get pierre => _pierre;

  void miner() {
    _or += 10;
    notifyListeners();
  }

  void couper() {
    _bois += 5;
    notifyListeners();
  }

  void tailler() {
    _pierre += 3;
    notifyListeners();
  }
}

class ApplicationRessources extends StatelessWidget {
  const ApplicationRessources({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.amber),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('watch contre select')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const BlocOrWatch(),
              const BlocBoisSelect(),
              const BlocPierreSelect(),
              const SizedBox(height: 32),
              Builder(
                builder: (BuildContext context) => Wrap(
                  spacing: 12,
                  children: <Widget>[
                    ElevatedButton(
                      onPressed: () => context.read<Ressources>().miner(),
                      child: const Text('Miner'),
                    ),
                    ElevatedButton(
                      onPressed: () => context.read<Ressources>().couper(),
                      child: const Text('Couper'),
                    ),
                    ElevatedButton(
                      onPressed: () => context.read<Ressources>().tailler(),
                      child: const Text('Tailler'),
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

// Pour 10 appuis répartis, ce bloc est reconstruit 10 fois.
class BlocOrWatch extends StatelessWidget {
  const BlocOrWatch({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocOrWatch');
    final Ressources r = context.watch<Ressources>();
    return Text('Or (watch) : ${r.or}', style: const TextStyle(fontSize: 22));
  }
}

// Reconstruit uniquement quand le bois change.
class BlocBoisSelect extends StatelessWidget {
  const BlocBoisSelect({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocBoisSelect');
    final int bois = context.select<Ressources, int>((Ressources r) => r.bois);
    return Text('Bois (select) : $bois', style: const TextStyle(fontSize: 22));
  }
}

// Reconstruit uniquement quand la pierre change.
class BlocPierreSelect extends StatelessWidget {
  const BlocPierreSelect({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocPierreSelect');
    final int pierre =
        context.select<Ressources, int>((Ressources r) => r.pierre);
    return Text('Pierre (select) : $pierre',
        style: const TextStyle(fontSize: 22));
  }
}
```

**Explication :** pour dix appuis (quatre « Miner », trois « Couper », trois
« Tailler »), la console montre `build BlocOrWatch` **dix** fois, alors que
`build BlocBoisSelect` n'apparaît que **trois** fois et `build BlocPierreSelect`
**trois** fois. Le bloc `watch` est reconstruit à chaque notification, quelle que
soit la ressource concernée ; les blocs `select` comparent leur valeur et se
taisent quand elle n'a pas bougé. Sur cet écran, `select` divise par plus de trois
le nombre total de reconstructions.

---

### Correction 6

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Compteur>(
      create: (_) => Compteur(),
      child: const ApplicationLourde(),
    ),
  );
}

class Compteur extends ChangeNotifier {
  int _valeur = 0;
  int get valeur => _valeur;

  void incrementer() {
    _valeur++;
    notifyListeners();
  }
}

// Widget "lourd" : on trace sa CONSTRUCTION.
class ListeLourde extends StatelessWidget {
  ListeLourde({super.key}) {
    debugPrint('  >>> CONSTRUCTION de ListeLourde');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('  >>> build de ListeLourde');
    return Column(
      children: List<Widget>.generate(
        50,
        (int i) => Text('Ligne ${i + 1}'),
      ),
    );
  }
}

class ApplicationLourde extends StatelessWidget {
  const ApplicationLourde({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.pink),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Consumer et child')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: <Widget>[
              const Text('VERSION A : watch en haut de build'),
              const VersionA(),
              const Divider(height: 32),
              const Text('VERSION B : Consumer avec child'),
              const VersionB(),
            ],
          ),
        ),
        floatingActionButton: Builder(
          builder: (BuildContext context) => FloatingActionButton(
            onPressed: () => context.read<Compteur>().incrementer(),
            child: const Icon(Icons.add),
          ),
        ),
      ),
    );
  }
}

class VersionA extends StatelessWidget {
  const VersionA({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build VersionA');
    final Compteur c = context.watch<Compteur>();
    return Column(
      children: <Widget>[
        Text('A -> ${c.valeur}', style: const TextStyle(fontSize: 22)),
        ListeLourde(), // reconstruite à chaque incrément
      ],
    );
  }
}

class VersionB extends StatelessWidget {
  const VersionB({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build VersionB');
    return Consumer<Compteur>(
      builder: (BuildContext context, Compteur c, Widget? enfant) {
        return Column(
          children: <Widget>[
            Text('B -> ${c.valeur}', style: const TextStyle(fontSize: 22)),
            enfant!, // réutilisé, jamais reconstruit
          ],
        );
      },
      child: ListeLourde(), // construite UNE SEULE FOIS
    );
  }
}
```

**Explication :** au démarrage, `CONSTRUCTION de ListeLourde` apparaît deux fois,
une par version. À chaque appui sur le bouton, elle réapparaît **une seule fois** :
celle de la version A. La version B ne reconstruit que le `Text`, parce que le
widget lourd, passé en `child:`, est capturé une fois pour toutes hors du `builder`
et retransmis sous le nom `enfant`. Remarquez que `ListeLourde` n'a pas de
constructeur `const` : c'est volontaire, afin que la trace de construction soit
visible. Dans du vrai code, un widget passé en `child` devrait être `const` chaque
fois que c'est possible.

---

### Correction 7

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider<Preferences>(create: (_) => Preferences()),
        ChangeNotifierProvider<Joueur>(create: (_) => Joueur()),
      ],
      child: const ApplicationPreferences(),
    ),
  );
}

class Preferences extends ChangeNotifier {
  bool _modeSombre = false;
  bool get modeSombre => _modeSombre;

  void basculerTheme() {
    _modeSombre = !_modeSombre;
    notifyListeners();
  }
}

class Joueur extends ChangeNotifier {
  int _score = 0;
  int get score => _score;

  void ramasser() {
    _score += 50;
    notifyListeners();
  }
}

class ApplicationPreferences extends StatelessWidget {
  const ApplicationPreferences({super.key});

  @override
  Widget build(BuildContext context) {
    // MaterialApp s'abonne au SEUL booléen du thème.
    final bool sombre =
        context.select<Preferences, bool>((Preferences p) => p.modeSombre);
    debugPrint('build MaterialApp (thème sombre : $sombre)');
    return MaterialApp(
      title: 'Thème partagé',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.dark,
        ),
        useMaterial3: true,
      ),
      themeMode: sombre ? ThemeMode.dark : ThemeMode.light,
      home: const EcranAccueil(),
    );
  }
}

class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('MultiProvider'),
        actions: const <Widget>[InterrupteurTheme()],
      ),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[AfficheScore(), SizedBox(height: 24), BoutonScore()],
        ),
      ),
    );
  }
}

class InterrupteurTheme extends StatelessWidget {
  const InterrupteurTheme({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build InterrupteurTheme');
    final bool sombre =
        context.select<Preferences, bool>((Preferences p) => p.modeSombre);
    return Row(
      children: <Widget>[
        Icon(sombre ? Icons.dark_mode : Icons.light_mode),
        Switch(
          value: sombre,
          onChanged: (bool _) => context.read<Preferences>().basculerTheme(),
        ),
      ],
    );
  }
}

class AfficheScore extends StatelessWidget {
  const AfficheScore({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build AfficheScore');
    final int score = context.select<Joueur, int>((Joueur j) => j.score);
    return Text('Score : $score', style: const TextStyle(fontSize: 32));
  }
}

class BoutonScore extends StatelessWidget {
  const BoutonScore({super.key});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.read<Joueur>().ramasser(),
      child: const Text('Ramasser 50 points'),
    );
  }
}
```

**Explication :** le `MultiProvider` est placé **au-dessus** de `MaterialApp`, ce qui
est indispensable ici : c'est `MaterialApp` lui-même qui doit lire le thème. Le
bouton de bascule se trouve pourtant dans l'`AppBar`, cinq niveaux plus bas, et il
n'a aucun paramètre. La bascule reconstruit `MaterialApp` — donc toute
l'application — ce qui est normal pour un changement de thème. En revanche, un
appui sur « Ramasser » ne reconstruit que `AfficheScore` : les deux états sont
totalement indépendants, alors qu'ils vivent dans le même `MultiProvider`.

---

### Correction 8

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider<Joueur>(create: (_) => Joueur()),
        ProxyProvider<Joueur, Difficulte>(
          update: (BuildContext context, Joueur joueur, Difficulte? ancienne) {
            debugPrint('update de Difficulte pour le niveau ${joueur.niveau}');
            return Difficulte(niveau: joueur.niveau);
          },
        ),
      ],
      child: const ApplicationDifficulte(),
    ),
  );
}

class Joueur extends ChangeNotifier {
  int _niveau = 1;
  int get niveau => _niveau;

  void monter() {
    _niveau++;
    notifyListeners();
  }

  void descendre() {
    if (_niveau <= 1) return;
    _niveau--;
    notifyListeners();
  }
}

// Objet dérivé, immuable : il est RECALCULÉ, jamais modifié.
class Difficulte {
  const Difficulte({required this.niveau});

  final int niveau;

  int get pointsDeVieEnnemi => 50 + niveau * 25;
  double get vitesseEnnemi => 1.0 + niveau * 0.15;
  String get libelle {
    if (niveau <= 2) return 'Facile';
    if (niveau <= 5) return 'Normal';
    if (niveau <= 9) return 'Difficile';
    return 'Cauchemar';
  }
}

class ApplicationDifficulte extends StatelessWidget {
  const ApplicationDifficulte({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('ProxyProvider')),
        body: const Center(child: TableauDifficulte()),
        bottomNavigationBar: const BarreNiveau(),
      ),
    );
  }
}

class TableauDifficulte extends StatelessWidget {
  const TableauDifficulte({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build TableauDifficulte');
    final Difficulte d = context.watch<Difficulte>();
    return Card(
      margin: const EdgeInsets.all(24),
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Text('Niveau ${d.niveau} — ${d.libelle}',
                style: Theme.of(context).textTheme.headlineSmall),
            const SizedBox(height: 16),
            Text('PV des ennemis : ${d.pointsDeVieEnnemi}',
                style: const TextStyle(fontSize: 20)),
            Text('Vitesse : ${d.vitesseEnnemi.toStringAsFixed(2)}',
                style: const TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}

class BarreNiveau extends StatelessWidget {
  const BarreNiveau({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: <Widget>[
          OutlinedButton(
            onPressed: () => context.read<Joueur>().descendre(),
            child: const Text('Niveau -'),
          ),
          FilledButton(
            onPressed: () => context.read<Joueur>().monter(),
            child: const Text('Niveau +'),
          ),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
Niveau 1 — Facile          Niveau 6 — Difficile
PV des ennemis : 75   ->   PV des ennemis : 200
Vitesse : 1.15             Vitesse : 1.90

Console à chaque changement de niveau :
update de Difficulte pour le niveau 6
build TableauDifficulte
```

**Explication :** `Difficulte` n'est pas un `ChangeNotifier`, et c'est volontaire :
elle ne contient aucun état propre, elle **dérive** entièrement du niveau du joueur.
`ProxyProvider` recrée l'objet à chaque notification amont, et `context.watch<Difficulte>()`
reconstruit le tableau. Aucune synchronisation n'est écrite à la main : c'est
exactement ce qu'apporte `ProxyProvider`. Notez l'ordre dans `providers:` :
`Joueur` doit précéder `ProxyProvider`, sinon le second ne le trouverait pas.

---

### Correction 9

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider<Panier>(
      create: (_) => Panier(),
      child: const ApplicationCatalogue(),
    ),
  );
}

class Objet {
  const Objet({
    required this.id,
    required this.nom,
    required this.prix,
    required this.icone,
    required this.description,
  });

  final String id;
  final String nom;
  final int prix;
  final IconData icone;
  final String description;

  @override
  bool operator ==(Object other) => other is Objet && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

const List<Objet> catalogue = <Objet>[
  Objet(
      id: 'x1',
      nom: 'Potion de soin',
      prix: 30,
      icone: Icons.local_drink,
      description: 'Restaure 40 points de vie instantanément.'),
  Objet(
      id: 'x2',
      nom: 'Torche',
      prix: 12,
      icone: Icons.local_fire_department,
      description: 'Éclaire les cavernes pendant 10 minutes.'),
  Objet(
      id: 'x3',
      nom: 'Corde',
      prix: 20,
      icone: Icons.link,
      description: 'Vingt mètres de corde solide.'),
  Objet(
      id: 'x4',
      nom: 'Parchemin',
      prix: 95,
      icone: Icons.description,
      description: 'Contient un sort de téléportation à usage unique.'),
];

class Panier extends ChangeNotifier {
  final Map<Objet, int> _lignes = <Objet, int>{};

  Map<Objet, int> get lignes => Map<Objet, int>.unmodifiable(_lignes);
  int get nombre => _lignes.values.fold(0, (int s, int q) => s + q);
  int get total => _lignes.entries
      .fold(0, (int s, MapEntry<Objet, int> e) => s + e.key.prix * e.value);
  int quantiteDe(Objet o) => _lignes[o] ?? 0;

  void ajouter(Objet o) {
    _lignes[o] = (_lignes[o] ?? 0) + 1;
    notifyListeners();
  }

  void retirer(Objet o) {
    final int actuel = _lignes[o] ?? 0;
    if (actuel == 0) return;
    if (actuel == 1) {
      _lignes.remove(o);
    } else {
      _lignes[o] = actuel - 1;
    }
    notifyListeners();
  }
}

class ApplicationCatalogue extends StatelessWidget {
  const ApplicationCatalogue({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.brown),
        useMaterial3: true,
      ),
      home: const EcranCatalogue9(),
    );
  }
}

class EcranCatalogue9 extends StatelessWidget {
  const EcranCatalogue9({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Échoppe'),
        actions: <Widget>[
          Builder(
            builder: (BuildContext context) {
              debugPrint('build badge');
              final int nombre =
                  context.select<Panier, int>((Panier p) => p.nombre);
              final int total =
                  context.select<Panier, int>((Panier p) => p.total);
              return Padding(
                padding: const EdgeInsets.only(right: 16),
                child: Center(
                  child: Text('$nombre article(s) — $total or'),
                ),
              );
            },
          ),
        ],
      ),
      body: ListView.separated(
        itemCount: catalogue.length,
        separatorBuilder: (BuildContext c, int i) => const Divider(height: 1),
        itemBuilder: (BuildContext context, int index) {
          final Objet objet = catalogue[index];
          return ListTile(
            leading: CircleAvatar(child: Icon(objet.icone)),
            title: Text(objet.nom),
            subtitle: Text('${objet.prix} or'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {
              // On lit le panier AVANT le push, avec le context courant.
              final Panier panier = context.read<Panier>();
              Navigator.of(context).push(
                MaterialPageRoute<void>(
                  builder: (BuildContext _) => MultiProvider(
                    providers: [
                      // .value : le panier existe déjà, on l'expose.
                      ChangeNotifierProvider<Panier>.value(value: panier),
                      Provider<Objet>.value(value: objet),
                    ],
                    child: const EcranDetail(),
                  ),
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
  const EcranDetail({super.key});

  @override
  Widget build(BuildContext context) {
    final Objet objet = context.read<Objet>();
    final int quantite =
        context.select<Panier, int>((Panier p) => p.quantiteDe(objet));
    return Scaffold(
      appBar: AppBar(title: Text(objet.nom)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Icon(objet.icone, size: 72),
            const SizedBox(height: 16),
            Text(objet.description,
                style: Theme.of(context).textTheme.bodyLarge),
            const SizedBox(height: 16),
            Text('Prix : ${objet.prix} or',
                style: const TextStyle(fontSize: 20)),
            const SizedBox(height: 8),
            Text('Déjà au panier : $quantite',
                style: const TextStyle(fontSize: 20)),
            const Spacer(),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: <Widget>[
                OutlinedButton.icon(
                  onPressed: quantite == 0
                      ? null
                      : () => context.read<Panier>().retirer(objet),
                  icon: const Icon(Icons.remove),
                  label: const Text('Retirer'),
                ),
                FilledButton.icon(
                  onPressed: () => context.read<Panier>().ajouter(objet),
                  icon: const Icon(Icons.add_shopping_cart),
                  label: const Text('Ajouter'),
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

**Explication :** le point délicat est le `Navigator.push`. Ici, le provider du
panier est déclaré au-dessus de `MaterialApp`, donc la route y aurait accès sans
rien faire. On réexpose malgré tout avec `ChangeNotifierProvider.value` pour montrer
le motif correct dans le cas général — celui où le provider serait scopé à un écran.
Notez `final Panier panier = context.read<Panier>();` écrit **avant** le `push` :
le `builder` de la `MaterialPageRoute` reçoit un `context` rattaché au `Navigator`,
qui ne descend pas de l'écran courant. `Provider<Objet>.value` est ici légitime :
`objet` est une constante du catalogue, personne ne doit la libérer. Enfin, le badge
utilise deux `select` distincts, ce qui est plus fin qu'un seul `watch`.

---

### Correction 10

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        // 1. Le nom du monde, chargé une fois.
        FutureProvider<NomDuMonde>(
          initialData: const NomDuMonde('Chargement...'),
          create: (BuildContext context) => chargerMonde(),
        ),
        // 2. Le flux de dégâts.
        StreamProvider<Degat>(
          initialData: const Degat(0),
          create: (BuildContext context) => fluxDegats(),
        ),
        // 3. Le héros, dont les PV dépendent des dégâts reçus.
        ChangeNotifierProxyProvider<Degat, Heros>(
          create: (BuildContext context) => Heros(),
          update: (BuildContext context, Degat degat, Heros? heros) {
            // On MET À JOUR l'instance existante, on ne la recrée pas.
            return (heros ?? Heros())..appliquer(degat);
          },
        ),
      ],
      child: const ApplicationAsync10(),
    ),
  );
}

// ── Types dédiés : deux String bruts se masqueraient ──────────
class NomDuMonde {
  const NomDuMonde(this.valeur);
  final String valeur;
}

class Degat {
  const Degat(this.points);
  final int points;
}

Future<NomDuMonde> chargerMonde() async {
  await Future<void>.delayed(const Duration(seconds: 2));
  return const NomDuMonde('Marais de Cendre');
}

Stream<Degat> fluxDegats() async* {
  int compteur = 0;
  while (compteur < 20) {
    await Future<void>.delayed(const Duration(milliseconds: 1500));
    compteur++;
    yield Degat(5 + (compteur % 4) * 5);
  }
}

// ── Le modèle ─────────────────────────────────────────────────
class Heros extends ChangeNotifier {
  int _pv = 100;
  int _degatsRecus = 0;
  int _dernierDegat = 0;

  int get pv => _pv;
  int get degatsRecus => _degatsRecus;
  int get dernierDegat => _dernierDegat;
  bool get estMort => _pv <= 0;

  // Appelée par le ProxyProvider, PAS depuis un build().
  void appliquer(Degat degat) {
    if (degat.points == 0 || estMort) return;
    _dernierDegat = degat.points;
    _degatsRecus += degat.points;
    _pv -= degat.points;
    if (_pv < 0) _pv = 0;
    notifyListeners();
  }
}

class ApplicationAsync10 extends StatelessWidget {
  const ApplicationAsync10({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blueGrey),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Asynchrone et état')),
        body: const Padding(
          padding: EdgeInsets.all(24),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              TitreMonde(),
              SizedBox(height: 32),
              BlocSante(),
              SizedBox(height: 24),
              JournalDegats(),
            ],
          ),
        ),
      ),
    );
  }
}

class TitreMonde extends StatelessWidget {
  const TitreMonde({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build TitreMonde');
    final NomDuMonde monde = context.watch<NomDuMonde>();
    return Text(monde.valeur,
        style: Theme.of(context).textTheme.headlineMedium);
  }
}

class BlocSante extends StatelessWidget {
  const BlocSante({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build BlocSante');
    final int pv = context.select<Heros, int>((Heros h) => h.pv);
    return Column(
      children: <Widget>[
        Text('Points de vie : $pv', style: const TextStyle(fontSize: 28)),
        const SizedBox(height: 12),
        LinearProgressIndicator(
          value: pv / 100,
          minHeight: 14,
          color: pv < 30 ? Colors.red : Colors.green,
        ),
        if (pv == 0)
          const Padding(
            padding: EdgeInsets.only(top: 16),
            child: Text('LE HÉROS EST TOMBÉ',
                style: TextStyle(
                    fontSize: 22,
                    color: Colors.red,
                    fontWeight: FontWeight.bold)),
          ),
      ],
    );
  }
}

class JournalDegats extends StatelessWidget {
  const JournalDegats({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('build JournalDegats');
    return Consumer<Heros>(
      builder: (BuildContext context, Heros heros, Widget? enfant) {
        return Card(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: <Widget>[
                Text('Dernier coup reçu : ${heros.dernierDegat}'),
                Text('Total encaissé : ${heros.degatsRecus}'),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

**Résultat :**

```text
t = 0 s   : "Chargement..."         PV 100
t = 1,5 s : "Chargement..."         PV  90   dernier coup 10
t = 2 s   : "Marais de Cendre"      PV  90
t = 3 s   : "Marais de Cendre"      PV  75   dernier coup 15
t = 4,5 s : "Marais de Cendre"      PV  55   dernier coup 20
...
```

**Explication :** trois providers coopèrent. `FutureProvider` et `StreamProvider`
exposent des **types dédiés** (`NomDuMonde`, `Degat`) et non des `String` ou des
`int` bruts : deux providers du même type se masqueraient l'un l'autre, comme
signalé en 52.28.2. Le `ChangeNotifierProxyProvider` est la pièce centrale : son
`update` est appelé par `provider` **entre** deux constructions, jamais pendant,
ce qui évite l'erreur `setState() or markNeedsBuild() called during build.`. Notez
la forme `(heros ?? Heros())..appliquer(degat)` avec l'opérateur de cascade
(chapitre 09) : on réutilise l'instance existante pour ne pas perdre les points de
vie déjà retirés, contrairement au piège de 52.26.1. Le garde
`if (degat.points == 0 || estMort) return;` évite d'appliquer la valeur initiale du
flux et de notifier inutilement après la mort du héros.

---

## Et maintenant ?

Vous savez désormais répondre aux trois questions de la gestion d'état : où vit la
donnée, qui la modifie, qui est prévenu. Vous connaissez le mécanisme natif
(`InheritedWidget`), les outils intégrés (`ChangeNotifier`, `ValueNotifier`,
`ListenableBuilder`), et l'outil de référence de l'écosystème (`provider`). Vous
savez aussi quand ne rien utiliser du tout et vous contenter de `setState`.

Il reste une lacune importante. Depuis le chapitre 43, toutes vos données sont
écrites en dur dans le code : des listes d'objets, des catalogues, des noms de
monde produits par un `Future.delayed`. Aucune application réelle ne fonctionne
ainsi.

Le chapitre suivant branche votre interface sur le monde extérieur : le protocole
HTTP, le paquet `http`, la transformation du JSON en objets Dart (en réutilisant
tout le chapitre 17), et surtout les deux widgets qui font le lien entre
l'asynchrone du chapitre 15 et l'arbre de widgets : `FutureBuilder` et
`StreamBuilder`. Vous y apprendrez à afficher un indicateur de chargement, à
traiter une erreur réseau proprement, et à ne plus jamais laisser un écran blanc à
l'utilisateur.

À la fin du chapitre 53, l'inventaire que vous venez de construire pourra être
chargé depuis un serveur.

Rendez-vous au chapitre 53 : [53-PARTIE-1B—API-REST-HTTP-ET-FUTUREBUILDER.md](./53-PARTIE-1B—API-REST-HTTP-ET-FUTUREBUILDER.md)
