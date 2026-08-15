# PARTIE 1B — FLUTTER
# CHAPITRE 45 — STATELESSWIDGET, STATEFULWIDGET ET CYCLE DE VIE

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 7 h
> **Pré-requis :** chapitre 44 — Les widgets et l'arbre de widgets
> **Ce que vous saurez faire à la fin :** décider si un widget doit être `StatelessWidget` ou `StatefulWidget`, stocker un état dans une classe `State`, le modifier proprement avec `setState()`, initialiser et libérer des ressources aux bons moments du cycle de vie, et remonter un état vers un parent commun.

---

## 45.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est l'« état » d'une interface ;
- distinguer ce qui change dans une interface de ce qui ne change pas ;
- définir précisément un `StatelessWidget` ;
- écrire un `StatelessWidget` de zéro ;
- expliquer le rôle exact de la méthode `build()` ;
- expliquer pourquoi toutes les propriétés d'un widget sans état sont `final` ;
- passer des données à un widget par son constructeur ;
- expliquer pourquoi un `StatefulWidget` est fait de **deux** classes ;
- écrire une classe `State<T>` ;
- placer les variables d'état au bon endroit ;
- utiliser `setState()` correctement ;
- décrire ce que `setState()` fait réellement dans le framework ;
- écrire le compteur, exemple canonique de Flutter ;
- reconnaître le bug silencieux de l'état modifié hors de `setState()` ;
- mesurer le coût d'un `setState()` inutile ;
- accéder aux propriétés du widget depuis l'état avec `widget.propriete` ;
- lire et dessiner le cycle de vie complet d'un `State` ;
- utiliser `createState()`, `initState()`, `didChangeDependencies()`, `build()`, `didUpdateWidget()`, `deactivate()` et `dispose()` ;
- libérer un contrôleur dans `dispose()` ;
- utiliser `mounted` pour éviter un `setState()` après démontage ;
- choisir entre état local et état remonté ;
- pratiquer la remontée d'état (*lifting state up*) ;
- faire redescendre une action par un callback ;
- énoncer les limites de cette approche et savoir ce que le chapitre 52 apportera ;
- appliquer un arbre de décision « stateless ou stateful ? ».

---

## 45.0.1 — Avertissement sur la progression

Ce chapitre est le cœur de Flutter. Tout le reste de la PARTIE 1B en dépend.

Nous n'utiliserons volontairement PAS encore :

```text
- les layouts avancés (Row, Column, Stack, Expanded)   -> chapitre 46
- les images et les assets                             -> chapitre 47
- les listes défilantes (ListView, GridView)           -> chapitre 48
- les formulaires validés (Form, TextFormField)        -> chapitre 49
- la navigation entre écrans (Navigator)               -> chapitre 50
- les thèmes personnalisés                             -> chapitre 51
- provider et ChangeNotifier                           -> chapitre 52
```

Les exemples resteront donc visuellement simples : une `Column`, un `Text`, un
`ElevatedButton`. Ce n'est pas un manque de soin, c'est un choix : ici, le sujet
n'est pas la beauté de l'écran, c'est **la mécanique de l'état**.

Chaque bloc de code est un `main.dart` **complet**. Vous pouvez le coller tel quel
dans `lib/main.dart` d'un projet créé au chapitre 43 (`flutter create mon_appli`),
puis lancer `flutter run`.

---

## 45.1 — Qu'est-ce que l'état d'une interface ?

Au chapitre 44, nous avons appris qu'un widget **décrit** une portion d'interface.
Nous avons construit des écrans figés : un titre, un texte, une icône.

Un écran figé, c'est rare. Une vraie application **change**.

Prenons un écran de jeu :

```text
┌──────────────────────────────┐
│  AVENTURE                    │   <- ne change jamais
├──────────────────────────────┤
│                              │
│  Joueur : Alex               │   <- ne change pas de la partie
│                              │
│  Vies   : 3                  │   <- CHANGE quand le joueur meurt
│  Score  : 1250               │   <- CHANGE à chaque pièce ramassée
│  Énergie: 68 %               │   <- CHANGE en permanence
│                              │
│      [ ATTAQUER ]            │   <- ne change jamais
│                              │
└──────────────────────────────┘
```

Trois informations bougent : les vies, le score, l'énergie.

> **L'état d'une interface, c'est l'ensemble des informations qui peuvent changer
> pendant que l'écran est affiché, et dont l'affichage dépend.**

Retenez les deux conditions, elles comptent toutes les deux :

1. l'information **peut changer** pendant la vie de l'écran ;
2. **l'affichage dépend** de cette information.

Si une information change mais que personne ne l'affiche, ce n'est pas de l'état
d'interface. Si une information est affichée mais ne change jamais, ce n'est pas
de l'état non plus.

---

## 45.1.1 — L'état vu comme une photographie

Une manière très utile de se représenter les choses :

```text
     ÉTAT (des données)                ÉCRAN (des pixels)

     vies    = 3           ─────>      "Vies : 3"
     score   = 1250        ─────>      "Score : 1250"
     energie = 68          ─────>      "Énergie : 68 %"

     ------------ le joueur ramasse une pièce ------------

     vies    = 3           ─────>      "Vies : 3"
     score   = 1300        ─────>      "Score : 1300"
     energie = 68          ─────>      "Énergie : 68 %"
```

L'écran est toujours **une photographie de l'état à un instant donné**.

Vous ne modifierez jamais l'écran directement. Vous ne direz jamais à Flutter
« remplace le texte 1250 par 1300 ». Vous direz :

> « le score vaut maintenant 1300, refais la photo ».

C'est la différence fondamentale entre Flutter et les anciennes façons de
programmer une interface. On appelle cela une interface **déclarative** :
on déclare à quoi l'écran doit ressembler **pour un état donné**, et le framework
se charge du reste.

---

## 45.1.2 — Ce que l'état n'est pas

Attention à trois confusions classiques.

**L'état n'est pas le contenu du serveur.** Les données qui viennent d'une API
(chapitre 53) sont des données. Elles deviennent de l'état seulement une fois
chargées dans l'écran et affichées.

**L'état n'est pas forcément visible.** Un booléen `_chargementEnCours` ne s'affiche
pas lui-même, mais il décide d'afficher un indicateur de chargement ou la liste.
Il fait donc partie de l'état.

**L'état n'est pas une variable globale.** Une variable déclarée en dehors de toute
classe est accessible partout, mais Flutter ne la surveille pas. La modifier ne
redessine rien. Nous verrons ce piège en 45.14.

---

## 45.2 — Ce qui change et ce qui ne change pas

Avant d'écrire une seule ligne, prenez l'habitude suivante : devant une maquette,
faites deux colonnes.

| Ce qui ne change pas | Ce qui change |
| --- | --- |
| le titre « AVENTURE » | le score |
| le libellé du bouton « ATTAQUER » | le nombre de vies |
| les couleurs du thème | la barre d'énergie |
| l'icône d'épée | le texte « Boss vaincu ! » qui apparaît |
| la mise en page générale | le contenu de l'inventaire |

Cette liste vous donne directement la réponse :

```text
Aucune ligne dans la colonne "ce qui change"  ->  StatelessWidget
Au moins une ligne dans "ce qui change"       ->  StatefulWidget
```

C'est aussi simple que cela. Toute la suite du chapitre ne fait qu'expliquer
**comment** on gère la colonne de droite.

---

## 45.2.1 — Un piège fréquent : « ça change, donc c'est stateful »

Attention à une nuance importante.

Une donnée qui change **avant** la construction du widget, et qui est ensuite
donnée au widget par son parent, ne rend PAS ce widget stateful.

```text
        ┌──────────────────────────────────┐
        │  ÉcranJeu   (StatefulWidget)     │   score = 1250   <- l'état vit ICI
        │                                  │
        │   ┌──────────────────────────┐   │
        │   │ CarteScore (Stateless)   │   │   reçoit 1250    <- pas d'état
        │   │  "Score : 1250"          │   │
        │   └──────────────────────────┘   │
        └──────────────────────────────────┘
```

`CarteScore` affiche un nombre qui change… mais **elle** ne le change pas. Elle le
reçoit. Chaque fois que le score change, le parent reconstruit une **nouvelle**
`CarteScore` avec la nouvelle valeur.

> Un widget est `StatefulWidget` s'il doit **se souvenir** d'une information entre
> deux reconstructions, et la **modifier lui-même**. Pas simplement s'il affiche
> quelque chose de variable.

Nous reviendrons longuement là-dessus en 45.27 et 45.28.

---

## 45.3 — `StatelessWidget` : la définition

Un `StatelessWidget` est un widget **sans état interne**.

Sa définition tient en une phrase :

> Un `StatelessWidget` est un widget dont l'apparence est entièrement déterminée
> par les informations qu'il reçoit à sa construction, et qui n'a aucun moyen de
> changer d'avis tout seul.

Regardons ce que cela implique concrètement.

```text
┌─────────────────────────────────────────────────────────┐
│  StatelessWidget                                        │
│                                                         │
│   ENTRÉES                                               │
│     - les propriétés passées au constructeur            │
│     - le BuildContext (thème, taille d'écran…)          │
│                                                         │
│                     │                                   │
│                     v                                   │
│               build(context)                            │
│                     │                                   │
│                     v                                   │
│   SORTIE                                                │
│     - un arbre de widgets                               │
│                                                         │
│  Mêmes entrées  =>  même sortie. Toujours.              │
└─────────────────────────────────────────────────────────┘
```

C'est exactement le comportement d'une fonction pure du chapitre 07 : mêmes
arguments, même résultat, aucun effet de bord.

Un `StatelessWidget` peut être reconstruit dix fois par seconde sans que rien ne
change à l'écran, tant que ses entrées sont les mêmes. C'est ce qui le rend
parfaitement prévisible.

---

## 45.3.1 — La forme minimale

En Dart, `StatelessWidget` est une **classe abstraite** (chapitre 11). On en hérite
avec `extends` (chapitre 10) et on est obligé de redéfinir `build()` :

```dart
class MonWidget extends StatelessWidget {
  const MonWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Bonjour');
  }
}
```

Quatre éléments, tous obligatoires ou fortement recommandés :

| Élément | Rôle |
| --- | --- |
| `extends StatelessWidget` | déclare que cette classe est un widget sans état |
| `const MonWidget({super.key})` | constructeur `const` (chapitre 09) et clé transmise au parent |
| `@override` | signale qu'on redéfinit une méthode héritée |
| `Widget build(BuildContext context)` | la seule méthode que le framework appellera |

> `super.key` transmet le paramètre `key` à la classe parente. La `Key` sert à
> Flutter pour identifier un widget entre deux reconstructions. Nous l'avons
> croisée au chapitre 44 ; elle deviendra vraiment utile au chapitre 48, avec les
> listes. Écrivez-la systématiquement, c'est une bonne habitude et l'analyseur
> Dart la réclame.

---

## 45.4 — Écrire son premier `StatelessWidget`

Écrivons une carte de statistique de jeu : un libellé, une valeur, une couleur.

Voici un `main.dart` complet et exécutable.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Statistiques',
      theme: ThemeData(useMaterial3: true),
      home: const EcranStatistiques(),
    );
  }
}

class EcranStatistiques extends StatelessWidget {
  const EcranStatistiques({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Fiche du joueur')),
      body: const Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            CarteStatistique(libelle: 'Vies', valeur: '3'),
            CarteStatistique(libelle: 'Score', valeur: '1250'),
            CarteStatistique(libelle: 'Énergie', valeur: '68 %'),
          ],
        ),
      ),
    );
  }
}

class CarteStatistique extends StatelessWidget {
  const CarteStatistique({
    super.key,
    required this.libelle,
    required this.valeur,
  });

  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        title: Text(libelle),
        trailing: Text(valeur),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────┐
│ Fiche du joueur                  │
├──────────────────────────────────┤
│ ┌──────────────────────────────┐ │
│ │ Vies                       3 │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ Score                   1250 │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ Énergie                 68 % │ │
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

Trois classes, trois rôles :

- `MonApplication` installe le `MaterialApp` (le cadre de l'application) ;
- `EcranStatistiques` décrit l'écran ;
- `CarteStatistique` est un composant **réutilisable**, utilisé trois fois.

Aucune de ces trois classes ne contient d'état. Les valeurs `'3'`, `'1250'`,
`'68 %'` sont écrites en dur. L'écran est une photo figée.

> Remarquez le `const` devant `Padding` dans `EcranStatistiques`. Il est possible
> parce que **tout** le sous-arbre est constant : `Column`, `CarteStatistique`,
> les chaînes. Flutter construira ces objets une seule fois pour toute la durée de
> l'application. C'est l'optimisation vue au chapitre 44.

---

## 45.4.1 — Pourquoi extraire `CarteStatistique` ?

On aurait pu écrire trois `Card` à la main dans `EcranStatistiques`. Extraire une
classe apporte quatre choses :

1. **Moins de répétition.** La mise en forme est écrite une fois.
2. **Un nom.** `CarteStatistique` dit ce que la chose est ; `Card > ListTile > Text`
   ne dit rien.
3. **Un point de modification unique.** Changer l'apparence des cartes se fait à
   un seul endroit.
4. **De meilleures performances.** Un widget extrait peut être `const` et peut être
   sauté lors des reconstructions. Une **fonction** qui retournerait le même arbre
   n'offre pas cet avantage.

> Règle à retenir dès maintenant : pour découper une interface, créez des **classes
> de widgets**, pas des méthodes `Widget _construireCarte()`. La documentation
> officielle de Flutter est explicite là-dessus dans la section performance de
> `StatefulWidget`.

---

## 45.5 — La méthode `build()`

`build()` est la seule méthode obligatoire d'un `StatelessWidget`. Sa signature
exacte est :

```dart
Widget build(BuildContext context)
```

Elle prend le contexte, elle retourne un widget. C'est tout.

### Ce que `build()` doit faire

- lire les propriétés du widget ;
- lire éventuellement le `context` (thème, dimensions) ;
- **retourner** un arbre de widgets.

### Ce que `build()` ne doit JAMAIS faire

| Interdit dans `build()` | Pourquoi |
| --- | --- |
| modifier une variable d'état | `build()` peut être appelé très souvent |
| appeler `setState()` | boucle infinie garantie |
| lancer une requête réseau | une requête par reconstruction |
| démarrer un `Timer` | un timer par reconstruction, jamais annulé |
| créer un contrôleur | fuite mémoire à chaque reconstruction |
| écrire dans un fichier | effet de bord imprévisible |

La règle est simple : **`build()` ne fait que décrire, jamais agir**.

---

## 45.5.1 — `build()` est appelé souvent, et ce n'est pas grave

Beaucoup de débutants s'inquiètent : « si `build()` est rappelé à chaque image,
c'est catastrophique pour les performances ? »

Non, et voici pourquoi.

```text
   build() crée des OBJETS DE DESCRIPTION (widgets)
                    │
                    v
   Flutter COMPARE la nouvelle description à l'ancienne
                    │
                    v
   Il ne redessine que ce qui a réellement changé
```

Un widget est un objet Dart minuscule : quelques champs, aucun pixel. En créer des
milliers coûte très peu. Ce qui coûte cher, c'est la mise en page (layout) et le
dessin (paint), et Flutter les évite quand la description est identique.

Cela dit, `build()` doit rester **rapide** :

- pas de boucle sur 100 000 éléments ;
- pas de calcul lourd ;
- pas d'appel bloquant.

Si un calcul est coûteux, faites-le **une fois** dans `initState()` (section 45.19)
et stockez le résultat.

---

## 45.5.2 — Combien de fois `build()` est-il appelé ?

Voici un exemple qui vous le montre concrètement. Il compte les appels et les
affiche dans la console.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranCompteurDeBuild(),
    );
  }
}

class EcranCompteurDeBuild extends StatefulWidget {
  const EcranCompteurDeBuild({super.key});

  @override
  State<EcranCompteurDeBuild> createState() => _EcranCompteurDeBuildState();
}

class _EcranCompteurDeBuildState extends State<EcranCompteurDeBuild> {
  int _pieces = 0;
  int _nombreDeBuild = 0;

  @override
  Widget build(BuildContext context) {
    _nombreDeBuild++;
    debugPrint('build() appelé $_nombreDeBuild fois');

    return Scaffold(
      appBar: AppBar(title: const Text('Compteur de build')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Pièces : $_pieces'),
            Text('Nombre de build : $_nombreDeBuild'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _pieces++;
                });
              },
              child: const Text('Ramasser une pièce'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat dans la console après trois appuis :**

```text
build() appelé 1 fois
build() appelé 2 fois
build() appelé 3 fois
build() appelé 4 fois
```

> **Attention :** incrémenter une variable dans `build()` est une très mauvaise
> pratique. Nous le faisons ici **uniquement** pour observer le mécanisme.
> N'écrivez jamais cela dans du vrai code.

Notez que `_nombreDeBuild` augmente mais que le texte affiché est celui **du build
en cours** : la valeur qui s'affiche est donc cohérente. Si vous tournez l'appareil
ou redimensionnez la fenêtre, vous verrez le compteur grimper sans que vous ayez
touché au bouton : Flutter reconstruit aussi quand les contraintes changent.

---

## 45.6 — Les propriétés `final` d'un widget sans état (rappel chapitre 02)

Au chapitre 02, nous avons vu `final` : une variable qu'on ne peut affecter qu'une
seule fois.

Dans un widget, **toutes** les propriétés doivent être `final`. Ce n'est pas une
convention de style, c'est une exigence du framework, et l'analyseur Dart vous
avertira si vous l'oubliez.

```dart
class CarteStatistique extends StatelessWidget {
  const CarteStatistique({super.key, required this.libelle, required this.valeur});

  final String libelle;   // final : correct
  final String valeur;    // final : correct

  @override
  Widget build(BuildContext context) => Text('$libelle : $valeur');
}
```

---

## 45.6.1 — Pourquoi cette obligation ?

Trois raisons, de la plus simple à la plus profonde.

**Raison 1 : un widget est jeté après usage.**

Modifier un champ d'un widget ne servirait à rien : à la reconstruction suivante,
le parent crée un **nouvel** objet widget, et votre modification disparaît.

```text
   Instant T    : CarteStatistique(valeur: '1250')   <- objet A
   vous écrivez : objetA.valeur = '1300'             <- interdit, et inutile
   Instant T+1  : CarteStatistique(valeur: '1250')   <- objet B, tout neuf
                  votre modification n'existe plus
```

**Raison 2 : `const` exige l'immuabilité.**

Un constructeur `const` n'est possible que si tous les champs sont `final`. Sans
`const`, vous perdez l'optimisation la plus rentable de Flutter.

**Raison 3 : la comparaison devient fiable.**

Flutter compare l'ancien widget et le nouveau pour décider quoi refaire. Si les
champs pouvaient changer dans le dos du framework, cette comparaison deviendrait
fausse et l'écran afficherait des données périmées.

---

## 45.6.2 — Ce qui arrive si vous oubliez `final`

```dart
class MauvaiseCarte extends StatelessWidget {
  MauvaiseCarte({super.key, required this.valeur});

  String valeur;   // PAS final -> avertissement de l'analyseur

  @override
  Widget build(BuildContext context) => Text(valeur);
}
```

**Résultat (analyseur) :**

```text
info: This class (or a class that this class inherits from) is marked as
'@immutable', but one or more of its instance fields aren't final.
must_be_immutable
```

Le code compile quand même. C'est un piège : vous croyez pouvoir écrire
`monWidget.valeur = 'x'`, l'écran ne bouge pas, et vous cherchez le bug pendant
une heure. Mettez `final` partout.

---

## 45.7 — Passer des données à un widget par son constructeur

C'est le mécanisme de communication numéro un de Flutter : **le parent donne, l'enfant reçoit**.

```text
        ┌─────────────────────────────┐
        │  EcranJeu                   │
        │                             │
        │   BarreDeVie(               │   les données descendent
        │     vies: 3,                │   ─────────────────────>
        │     viesMax: 5,             │
        │   )                         │
        └──────────────┬──────────────┘
                       │
                       v
        ┌─────────────────────────────┐
        │  BarreDeVie                 │
        │    final int vies;    = 3   │
        │    final int viesMax; = 5   │
        └─────────────────────────────┘
```

Voici un exemple complet avec plusieurs types de paramètres : requis, optionnels,
avec valeur par défaut.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

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
    return Scaffold(
      appBar: AppBar(title: const Text('Équipe')),
      body: const Column(
        children: [
          FicheHeros(nom: 'Alex', vies: 3, classe: 'Guerrier'),
          FicheHeros(nom: 'Sophie', vies: 5, classe: 'Mage'),
          FicheHeros(nom: 'Samir', vies: 4),
        ],
      ),
    );
  }
}

class FicheHeros extends StatelessWidget {
  const FicheHeros({
    super.key,
    required this.nom,          // obligatoire
    required this.vies,         // obligatoire
    this.classe = 'Aventurier', // optionnel avec valeur par défaut
    this.estBoss = false,       // optionnel avec valeur par défaut
  });

  final String nom;
  final int vies;
  final String classe;
  final bool estBoss;

  @override
  Widget build(BuildContext context) {
    return Card(
      color: estBoss ? Colors.red.shade100 : null,
      child: ListTile(
        leading: CircleAvatar(child: Text(nom[0])),
        title: Text('$nom — $classe'),
        subtitle: Text('Vies : $vies'),
      ),
    );
  }
}
```

**Résultat :**

```text
┌────────────────────────────────────────┐
│ Équipe                                 │
├────────────────────────────────────────┤
│ ( A )  Alex — Guerrier                 │
│        Vies : 3                        │
│ ( S )  Sophie — Mage                   │
│        Vies : 5                        │
│ ( S )  Samir — Aventurier              │
│        Vies : 4                        │
└────────────────────────────────────────┘
```

Points à noter :

- `required` (chapitre 09) rend le paramètre obligatoire : oublier `nom` est une
  erreur de compilation, pas un bug à l'exécution ;
- `this.classe = 'Aventurier'` donne une valeur par défaut, utilisée pour Samir ;
- les paramètres nommés rendent l'appel lisible : `FicheHeros(nom: ..., vies: ...)`
  est bien plus clair que `FicheHeros('Alex', 3, 'Guerrier', false)` ;
- `nom[0]` extrait la première lettre (chapitre 02) pour l'avatar. Aucun fichier
  image n'est nécessaire.

---

## 45.7.1 — Le sens unique de la communication

Retenez bien ce schéma, il explique la moitié des difficultés des débutants :

```text
   PARENT  ───── données (constructeur) ─────>  ENFANT      OUI, naturel

   PARENT  <──── données (?????????????) ─────  ENFANT      NON, impossible
                                                            directement
```

Un enfant ne peut pas « pousser » une valeur vers son parent. Il ne le connaît même
pas. La solution s'appelle **le callback**, et nous la verrons en 45.29.

---

## 45.8 — `StatefulWidget` : pourquoi deux classes

Passons à ce qui change.

Un `StatefulWidget` s'écrit **toujours** en deux classes :

```dart
class Compteur extends StatefulWidget {          // classe 1 : la configuration
  const Compteur({super.key});

  @override
  State<Compteur> createState() => _CompteurState();
}

class _CompteurState extends State<Compteur> {   // classe 2 : l'état
  int _valeur = 0;

  @override
  Widget build(BuildContext context) => Text('$_valeur');
}
```

La question que tout le monde se pose : **pourquoi deux classes ? Pourquoi pas
une seule avec une variable dedans ?**

La réponse est la clé de tout Flutter. Prenez le temps de la lire deux fois.

---

## 45.8.1 — Le widget est jetable, l'état ne l'est pas

Rappel du chapitre 44 : Flutter jette et recrée les objets widgets en permanence.
À chaque reconstruction du parent, un **nouvel** objet `Compteur` est créé.

Si le compteur était stocké dans le widget, il repartirait à zéro à chaque
reconstruction :

```text
   SANS séparation (ce qui n'existe pas en Flutter)

   t=0   new Compteur()     valeur = 0
   t=1   appui  ->  valeur = 1
   t=2   le parent se reconstruit
         new Compteur()     valeur = 0     <- PERDU
```

Avec la séparation en deux classes :

```text
   AVEC séparation (Flutter réel)

   t=0   new Compteur()  ──crée──> _CompteurState   _valeur = 0
   t=1   appui  ->  _valeur = 1
   t=2   le parent se reconstruit
         new Compteur()  ──────>   le MÊME _CompteurState est conservé
                                    _valeur = 1     <- CONSERVÉ
```

> **Le widget est une configuration jetable. Le `State` est un objet durable,
> conservé par le framework tant que le widget reste au même endroit de l'arbre,
> avec le même type et la même `Key`.**

---

## 45.8.2 — Les trois arbres de Flutter

Pour bien comprendre, il faut savoir que Flutter maintient trois arbres parallèles.

```text
  ARBRE DES WIDGETS        ARBRE DES ÉLÉMENTS        ARBRE DES OBJETS DE RENDU
  (description)            (le lien durable)         (les pixels)

  Compteur          ───>   StatefulElement    ───>   RenderObject
   (recréé souvent)         (garde le State)          (mesure et dessine)
                                 │
                                 └─> _CompteurState
                                       _valeur = 1
```

- l'**arbre des widgets** est jeté et recréé constamment : c'est du texte, une
  recette, une description ;
- l'**arbre des éléments** est stable : chaque `Element` est le point fixe qui
  survit aux reconstructions et qui tient le `State` ;
- l'**arbre de rendu** fait la mise en page et le dessin.

Vous ne manipulerez jamais les deux derniers directement. Mais savoir qu'ils
existent explique pourquoi l'état survit alors que le widget est jeté.

---

## 45.8.3 — Le nom du `State` commence par un `_`

Convention universelle en Flutter :

```dart
class Compteur extends StatefulWidget { ... }
class _CompteurState extends State<Compteur> { ... }
```

Le tiret bas rend la classe **privée au fichier** (chapitre 10, encapsulation).
C'est voulu : personne d'autre que le widget ne doit toucher à son état interne.
Le nom suit toujours le motif `_<NomDuWidget>State`.

> Dans VS Code et Android Studio, tapez `stful` puis Tab : l'éditeur génère les
> deux classes d'un coup. Tapez `stless` pour un `StatelessWidget`.

---

## 45.9 — La classe `State<T>`

`State<T>` est une classe générique (chapitre 11). Le `T` est le type du widget
associé.

```dart
class _CompteurState extends State<Compteur>
//                             ─────┬──────
//                                  └─ le widget auquel cet état appartient
```

Grâce à ce `T`, la propriété `widget` sera typée `Compteur`, et vous pourrez écrire
`widget.valeurInitiale` avec l'autocomplétion et la vérification de types.

### Ce que `State<T>` vous fournit

| Membre | Type | Rôle |
| --- | --- | --- |
| `widget` | `T` | la configuration courante (le widget actuel) |
| `context` | `BuildContext` | la position dans l'arbre |
| `mounted` | `bool` | vrai tant que l'état est dans l'arbre |
| `setState(fn)` | méthode | signale un changement d'état |
| `initState()` | méthode | appelée une fois, à l'insertion |
| `didChangeDependencies()` | méthode | appelée quand une dépendance change |
| `build(context)` | méthode | décrit l'interface (obligatoire) |
| `didUpdateWidget(old)` | méthode | appelée quand la configuration change |
| `deactivate()` | méthode | appelée au retrait de l'arbre |
| `activate()` | méthode | appelée en cas de réinsertion |
| `dispose()` | méthode | appelée à la destruction définitive |

Ces noms et signatures viennent directement de la documentation de la classe
`State` sur `api.flutter.dev`. Nous les détaillerons une par une de 45.18 à 45.24.

---

## 45.9.1 — La seule méthode obligatoire

Dans un `State`, une seule méthode doit obligatoirement être redéfinie : `build()`.

```dart
class _CompteurState extends State<Compteur> {
  @override
  Widget build(BuildContext context) {
    return const Text('Bonjour');
  }
}
```

Toutes les autres sont optionnelles. Vous ne redéfinissez `initState()` ou
`dispose()` que si vous en avez besoin.

---

## 45.10 — Où vivent les variables d'état

Question simple, réponse capitale :

> **Les variables d'état vont dans la classe `State`, jamais dans le
> `StatefulWidget`.**

### Correct

```dart
class Compteur extends StatefulWidget {
  const Compteur({super.key});

  @override
  State<Compteur> createState() => _CompteurState();
}

class _CompteurState extends State<Compteur> {
  int _valeur = 0;   // BON ENDROIT : dans le State
  // ...
}
```

### Incorrect

```dart
class Compteur extends StatefulWidget {
  Compteur({super.key});

  int valeur = 0;    // MAUVAIS ENDROIT : dans le widget

  @override
  State<Compteur> createState() => _CompteurState();
}
```

Le second code **compile** mais produit un bug parfaitement silencieux : la valeur
repart à zéro dès que le parent se reconstruit, et l'analyseur signale seulement
un `must_be_immutable`.

---

## 45.10.1 — Le tableau de répartition

Utilisez ce tableau comme référence :

| Donnée | Où la mettre |
| --- | --- |
| une valeur reçue du parent (`nom`, `couleur`, `viesMax`) | dans le **widget**, en `final` |
| une valeur que ce widget modifie lui-même (`_score`, `_estOuvert`) | dans le **State** |
| un `TextEditingController` | dans le **State** |
| un `AnimationController` | dans le **State** |
| un `Timer` | dans le **State** |
| une valeur calculée à partir de l'état, utilisée seulement dans `build()` | variable locale de `build()` |
| une constante partagée par toutes les instances | `static const` dans le widget |

---

## 45.10.2 — Convention de nommage de l'état

Les variables d'état sont préfixées par `_` : elles sont privées, personne ne doit
y toucher de l'extérieur.

```dart
class _EcranJeuState extends State<EcranJeu> {
  int _score = 0;
  int _vies = 3;
  bool _partieTerminee = false;
  List<String> _inventaire = [];
}
```

Une variable d'état sans `_` n'est pas une erreur, mais c'est un signal que vous
n'avez pas encore intégré le principe d'encapsulation du chapitre 10.

---

## 45.11 — `setState()`

Voici la méthode la plus importante de ce chapitre.

Sa signature exacte est :

```dart
void setState(VoidCallback fn)
```

Elle prend une fonction sans paramètre et sans retour (chapitre 07), et ne
retourne rien.

### L'usage correct

```dart
setState(() {
  _score = _score + 100;
});
```

On lit cela ainsi :

> « Modifie le score **et** préviens le framework que l'écran doit être refait. »

---

## 45.11.1 — Ce qu'on met dans `setState()` et ce qu'on laisse dehors

La fonction passée à `setState()` doit contenir **uniquement les modifications
d'état**. Rien d'autre.

### Correct

```dart
void _ramasserPiece() {
  setState(() {
    _score += 10;
    _pieces += 1;
  });
}
```

### Acceptable mais moins propre

```dart
void _ramasserPiece() {
  setState(() {
    _score += 10;
    debugPrint('score = $_score');   // effet de bord dans setState
  });
}
```

### À éviter absolument

```dart
void _ramasserPiece() {
  setState(() async {                // setState ne doit JAMAIS être async
    await _sauvegarderSurServeur();
    _score += 10;
  });
}
```

`setState()` attend un `VoidCallback`, donc une fonction **synchrone**. Passer une
fonction `async` produit un `Future` que Flutter ignore : la reconstruction a lieu
**avant** que votre `await` ne se termine. La bonne façon :

```dart
Future<void> _ramasserPiece() async {
  await _sauvegarderSurServeur();    // d'abord le travail asynchrone
  if (!mounted) return;              // voir 45.26
  setState(() {
    _score += 10;                    // puis la modification d'état
  });
}
```

---

## 45.11.2 — Une variante qui marche mais qu'il ne faut pas prendre

Ceci fonctionne :

```dart
_score += 10;
setState(() {});    // setState vide
```

L'état est modifié avant, et `setState()` vide déclenche quand même la
reconstruction. Alors pourquoi est-ce déconseillé ?

Parce que le code perd son intention. Le bloc `setState()` sert aussi de
**documentation** : il montre au lecteur exactement quelles variables constituent
l'état. Un `setState(() {})` vide oblige à chercher dans tout le fichier.

> Mettez les modifications **dans** le bloc. Toujours.

---

## 45.12 — Ce que `setState()` fait vraiment

Beaucoup de débutants pensent que `setState()` redessine l'écran immédiatement.
C'est faux, et comprendre ce qui se passe vraiment évite plusieurs bugs.

Voici la séquence réelle :

```text
   1. Vous appelez setState(() { _score += 10; })
              │
   2. Flutter EXÉCUTE votre fonction immédiatement
      -> _score vaut maintenant 1260
              │
   3. Flutter MARQUE l'Element comme "sale" (dirty)
              │
   4. setState() RETOURNE. La ligne suivante de votre code s'exécute.
              │
      ...  votre méthode se termine  ...
              │
   5. À la prochaine image (frame), Flutter parcourt les Elements sales
              │
   6. Il appelle build() sur chacun
              │
   7. Il compare l'ancien arbre et le nouveau
              │
   8. Il met à jour uniquement les RenderObject concernés
              │
   9. L'écran change
```

Trois conséquences pratiques :

**Conséquence 1 : votre variable est modifiée tout de suite.**

```dart
setState(() {
  _score = 100;
});
debugPrint('$_score');   // affiche 100, immédiatement
```

**Conséquence 2 : l'écran, lui, n'a pas encore changé.**

À la ligne suivant `setState()`, l'affichage est toujours l'ancien. Il changera à
la prochaine image, quelques millisecondes plus tard.

**Conséquence 3 : plusieurs `setState()` de suite ne coûtent qu'un seul build.**

```dart
setState(() { _score += 10; });
setState(() { _pieces += 1; });
setState(() { _niveau = 2; });
// -> UN SEUL build() sera exécuté
```

Flutter regroupe. C'est pourquoi un `setState()` n'est pas aussi coûteux qu'on le
craint. Mais écrire trois `setState()` reste inutilement verbeux : regroupez les
modifications dans un seul bloc.

---

## 45.12.1 — Le périmètre exact d'un `setState()`

Question fréquente : « `setState()` reconstruit-il toute l'application ? »

Non. Il ne marque comme sale que **l'Element du `State` sur lequel il est appelé**.

```text
        MaterialApp                 non reconstruit
             │
        EcranJeu                    non reconstruit
             │
        ┌────┴─────────────────┐
        │                      │
    PanneauScore           ZoneDeJeu          <- setState() ici
    non reconstruit             │
                        ┌───────┴────────┐
                        │                │
                 BarreDeVie        SpriteJoueur   <- reconstruits (enfants)
```

`setState()` reconstruit le `build()` du `State` concerné, donc tout ce que ce
`build()` retourne : le widget lui-même et ses descendants directs décrits dans
cette méthode.

Les sous-arbres marqués `const` sont sautés, car Flutter reconnaît qu'il s'agit du
**même objet en mémoire** et ne redescend pas dedans.

> C'est la raison numéro un d'écrire `const` partout où c'est possible, et de
> placer l'état **le plus bas possible** dans l'arbre. Nous y reviendrons en 45.27.

---

## 45.13 — Le compteur : l'exemple canonique

Voici le programme que Flutter génère lorsque vous créez un projet, réécrit dans
notre univers de jeu. C'est l'exemple à connaître par cœur.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Compteur de pièces',
      theme: ThemeData(useMaterial3: true),
      home: const EcranPieces(),
    );
  }
}

class EcranPieces extends StatefulWidget {
  const EcranPieces({super.key});

  @override
  State<EcranPieces> createState() => _EcranPiecesState();
}

class _EcranPiecesState extends State<EcranPieces> {
  int _pieces = 0;

  void _ramasserUnePiece() {
    setState(() {
      _pieces++;
    });
  }

  void _toutDepenser() {
    setState(() {
      _pieces = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bourse du joueur')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Pièces ramassées :'),
            Text(
              '$_pieces',
              style: Theme.of(context).textTheme.displayMedium,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _ramasserUnePiece,
              child: const Text('Ramasser une pièce'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _pieces == 0 ? null : _toutDepenser,
              child: const Text('Tout dépenser'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après trois appuis sur « Ramasser une pièce » :**

```text
┌──────────────────────────────────┐
│ Bourse du joueur                 │
├──────────────────────────────────┤
│                                  │
│      Pièces ramassées :          │
│               3                  │
│                                  │
│    [ Ramasser une pièce ]        │
│    [   Tout dépenser    ]        │
│                                  │
└──────────────────────────────────┘
```

Détaillons les points importants.

**`onPressed: _ramasserUnePiece`** — on passe la **référence** de la méthode, sans
parenthèses. Écrire `onPressed: _ramasserUnePiece()` appellerait la méthode
immédiatement pendant `build()` : bug classique, boucle infinie assurée.

**`onPressed: _pieces == 0 ? null : _toutDepenser`** — passer `null` à `onPressed`
**désactive** le bouton, qui devient grisé automatiquement. C'est le mécanisme
standard de Flutter, pas besoin d'une propriété `enabled`.

**`Theme.of(context).textTheme.displayMedium`** — lecture d'un style dans le thème
via le `context`. Nous approfondirons au chapitre 51.

**Les méthodes `_ramasserUnePiece` et `_toutDepenser`** sont extraites plutôt
qu'écrites en ligne dans `onPressed`. Avec une ou deux instructions, une fonction
anonyme en ligne est acceptable ; au-delà, extrayez une méthode nommée.

---

## 45.13.1 — Un compteur borné, plus réaliste

Un état ne doit jamais devenir incohérent. Ajoutons des règles métier : maximum
10 vies, minimum 0.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranVies(),
    );
  }
}

class EcranVies extends StatefulWidget {
  const EcranVies({super.key});

  @override
  State<EcranVies> createState() => _EcranViesState();
}

class _EcranViesState extends State<EcranVies> {
  static const int viesMax = 10;

  int _vies = 3;

  bool get _peutSoigner => _vies < viesMax;
  bool get _peutBlesser => _vies > 0;

  void _soigner() {
    if (!_peutSoigner) return;
    setState(() {
      _vies++;
    });
  }

  void _blesser() {
    if (!_peutBlesser) return;
    setState(() {
      _vies--;
    });
  }

  @override
  Widget build(BuildContext context) {
    final String coeurs = '#' * _vies + '.' * (viesMax - _vies);

    return Scaffold(
      appBar: AppBar(title: const Text('Points de vie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('$_vies / $viesMax'),
            const SizedBox(height: 8),
            Text(coeurs, style: const TextStyle(fontSize: 24, letterSpacing: 4)),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _peutBlesser ? _blesser : null,
                  child: const Text('Subir un coup'),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: _peutSoigner ? _soigner : null,
                  child: const Text('Boire une potion'),
                ),
              ],
            ),
            if (_vies == 0)
              const Padding(
                padding: EdgeInsets.only(top: 24),
                child: Text(
                  'GAME OVER',
                  style: TextStyle(fontSize: 32, color: Colors.red),
                ),
              ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat avec 3 vies :**

```text
┌──────────────────────────────────┐
│ Points de vie                    │
├──────────────────────────────────┤
│                                  │
│            3 / 10                │
│      # # # . . . . . . .         │
│                                  │
│  [Subir un coup] [Boire potion]  │
│                                  │
└──────────────────────────────────┘
```

**Résultat à 0 vie :**

```text
┌──────────────────────────────────┐
│ Points de vie                    │
├──────────────────────────────────┤
│            0 / 10                │
│      . . . . . . . . . .         │
│                                  │
│  [Subir un coup] [Boire potion]  │
│      (le premier est grisé)      │
│                                  │
│          GAME OVER               │
└──────────────────────────────────┘
```

Trois techniques nouvelles et très utiles :

**Les getters dérivés.** `_peutSoigner` et `_peutBlesser` ne sont pas de l'état :
ce sont des valeurs **calculées** à partir de `_vies` (chapitre 10). Ne stockez
jamais ce que vous pouvez calculer, sous peine d'incohérence.

**Le calcul dans `build()`.** `coeurs` est une variable **locale**, recalculée à
chaque build. C'est correct : c'est un calcul instantané qui ne dépend que de
l'état.

**Le `if` dans une liste de widgets.** `if (_vies == 0) ...` est un *collection if*
(chapitre 06). Si la condition est fausse, aucun widget n'est ajouté. C'est la
manière idiomatique d'afficher un élément conditionnellement.

---

## 45.14 — Modifier l'état sans `setState()` : le bug silencieux

Voici l'erreur numéro un des débutants Flutter. Vous la ferez. Autant la voir
maintenant.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranBugue(),
    );
  }
}

class EcranBugue extends StatefulWidget {
  const EcranBugue({super.key});

  @override
  State<EcranBugue> createState() => _EcranBugueState();
}

class _EcranBugueState extends State<EcranBugue> {
  int _score = 0;

  void _marquerDesPoints() {
    _score += 10;                       // ERREUR : pas de setState
    debugPrint('score interne = $_score');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build() avec score = $_score');
    return Scaffold(
      appBar: AppBar(title: const Text('Bug silencieux')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Score affiché : $_score'),
            ElevatedButton(
              onPressed: _marquerDesPoints,
              child: const Text('Marquer 10 points'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat dans la console après trois appuis :**

```text
build() avec score = 0
score interne = 10
score interne = 20
score interne = 30
```

**Résultat à l'écran :**

```text
Score affiché : 0        <- inchangé, malgré les trois appuis
```

La variable **a bien changé**. L'écran, non. Aucune erreur, aucun message rouge,
aucun avertissement de l'analyseur.

C'est exactement ce que dit la section 45.1 : Flutter ne surveille pas vos
variables. Il ne peut pas savoir que `_score` a changé. `setState()` est le signal
que vous lui envoyez.

---

## 45.14.1 — Comment reconnaître ce bug

Symptôme caractéristique :

> « Je clique, rien ne se passe. Mais si je tourne le téléphone, la bonne valeur
> apparaît d'un coup. »

Explication : la rotation change les contraintes, ce qui provoque un `build()`
pour une autre raison. Ce `build()` lit la variable, qui est à jour depuis
longtemps. La valeur « apparaît » subitement.

Idem avec le hot reload : il relance `build()` et « répare » l'affichage.

> **Si un changement n'apparaît qu'après une rotation ou un hot reload, vous avez
> oublié un `setState()`.**

---

## 45.14.2 — La correction

```dart
  void _marquerDesPoints() {
    setState(() {
      _score += 10;
    });
  }
```

**Résultat dans la console après trois appuis :**

```text
build() avec score = 0
build() avec score = 10
build() avec score = 20
build() avec score = 30
```

L'écran suit désormais la variable.

---

## 45.14.3 — La variante avec les collections

Le même piège existe, en plus vicieux, avec les listes du chapitre 06.

```dart
  // NE FONCTIONNE PAS
  void _ajouterAuSac(String objet) {
    _inventaire.add(objet);      // la liste change, l'écran non
  }

  // FONCTIONNE
  void _ajouterAuSac(String objet) {
    setState(() {
      _inventaire.add(objet);
    });
  }
```

C'est plus vicieux car il n'y a même pas d'affectation : on modifie l'objet liste
en place. Beaucoup de débutants pensent qu'un `setState()` n'est utile que pour
`=`. Faux : **toute** modification de l'état, y compris `add`, `remove`, `clear`,
`sort`, doit être encadrée par `setState()`.

---

## 45.15 — `setState()` appelé pour rien : le coût

L'erreur inverse existe aussi : appeler `setState()` alors que rien ne change.

```dart
  void _selectionnerArme(String arme) {
    setState(() {
      _armeChoisie = arme;   // si c'est déjà la même arme, on rebuild pour rien
    });
  }
```

Si l'utilisateur retouche l'arme déjà sélectionnée, on refait tout le `build()`
pour aboutir à un écran identique.

### Ce que cela coûte réellement

```text
   setState() inutile
        │
        ├─ build() ré-exécuté            (création d'objets Dart : léger)
        ├─ comparaison des arbres        (léger)
        ├─ layout si la taille change    (peut être lourd)
        └─ paint si les pixels changent  (peut être lourd)
```

Si rien ne change réellement, Flutter s'arrête tôt dans la chaîne : le surcoût est
faible. Un `setState()` inutile de temps en temps n'est **pas** un drame.

Cela devient un vrai problème dans deux cas :

1. **`setState()` très fréquent** — dans un `Timer` à 60 Hz, dans un capteur, dans
   un défilement ;
2. **`build()` coûteux** — une liste longue construite sans `.builder`, un calcul
   lourd, un formatage complexe.

### La protection : ne rien faire si rien ne change

```dart
  void _selectionnerArme(String arme) {
    if (_armeChoisie == arme) return;   // garde en début de méthode
    setState(() {
      _armeChoisie = arme;
    });
  }
```

Ce test s'appelle une **garde** (*guard clause*). C'est un réflexe à prendre :
toute méthode qui appelle `setState()` peut commencer par vérifier que le
changement est réel.

---

## 45.15.1 — Ne jamais appeler `setState()` dans `build()`

Il existe un cas de `setState()` inutile qui, lui, est fatal.

```dart
  @override
  Widget build(BuildContext context) {
    setState(() { _compteur++; });   // CATASTROPHE
    return Text('$_compteur');
  }
```

Enchaînement :

```text
   build()  ->  setState()  ->  marque sale  ->  build()  ->  setState()  -> ...
```

Boucle infinie. L'application se fige, la console se remplit, et Flutter finit par
lever une exception :

```text
setState() or markNeedsBuild() called during build.
This Overlay widget cannot be marked as needing to build because the framework
is already in the process of building widgets.
```

Le message est explicite. Si vous le voyez, cherchez un `setState()` appelé
directement — ou indirectement, via une méthode — depuis un `build()`.

> Le même message apparaît si vous appelez `setState()` depuis `initState()` :
> l'état est déjà en train d'être construit. Dans `initState()`, il suffit
> d'affecter la variable directement, sans `setState()` (voir 45.19).

---

## 45.16 — Accéder au widget depuis l'état : `widget.propriete`

Le `State` a besoin de lire les données passées au widget. La propriété `widget`
du `State` est là pour cela.

```text
    Compteur(valeurInitiale: 5, pas: 10)          <- le widget (configuration)
                     │
                     │  accessible depuis le State via  widget.
                     v
    _CompteurState
      widget.valeurInitiale  -> 5
      widget.pas             -> 10
```

Exemple complet :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

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
      body: const Column(
        children: [
          CompteurAchat(nomObjet: 'Potion de soin', prix: 25),
          CompteurAchat(nomObjet: 'Épée en fer', prix: 150, quantiteMax: 1),
        ],
      ),
    );
  }
}

class CompteurAchat extends StatefulWidget {
  const CompteurAchat({
    super.key,
    required this.nomObjet,
    required this.prix,
    this.quantiteMax = 99,
  });

  final String nomObjet;
  final int prix;
  final int quantiteMax;

  @override
  State<CompteurAchat> createState() => _CompteurAchatState();
}

class _CompteurAchatState extends State<CompteurAchat> {
  int _quantite = 0;

  void _ajouter() {
    if (_quantite >= widget.quantiteMax) return;   // lecture du widget
    setState(() {
      _quantite++;
    });
  }

  void _retirer() {
    if (_quantite == 0) return;
    setState(() {
      _quantite--;
    });
  }

  @override
  Widget build(BuildContext context) {
    final int total = _quantite * widget.prix;     // lecture du widget

    return Card(
      child: ListTile(
        title: Text(widget.nomObjet),
        subtitle: Text('${widget.prix} pièces — total : $total'),
        trailing: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            IconButton(
              onPressed: _quantite == 0 ? null : _retirer,
              icon: const Icon(Icons.remove),
            ),
            Text('$_quantite'),
            IconButton(
              onPressed: _quantite >= widget.quantiteMax ? null : _ajouter,
              icon: const Icon(Icons.add),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après 2 potions et 1 épée :**

```text
┌────────────────────────────────────────────┐
│ Boutique                                   │
├────────────────────────────────────────────┤
│ Potion de soin                             │
│ 25 pièces — total : 50      [-]  2  [+]    │
├────────────────────────────────────────────┤
│ Épée en fer                                │
│ 150 pièces — total : 150    [-]  1  [+]    │
│                                  (grisé)   │
└────────────────────────────────────────────┘
```

Deux instances du même widget, deux états **indépendants**. Chaque
`CompteurAchat` a son propre `_CompteurAchatState`, donc sa propre `_quantite`.

---

## 45.16.1 — Trois règles sur `widget.`

**Règle 1 : `widget` est toujours à jour.**

Quand le parent reconstruit avec de nouvelles valeurs, Flutter remplace l'objet
pointé par `widget` **avant** d'appeler `build()`. Vous lisez donc toujours la
configuration courante.

**Règle 2 : ne copiez pas `widget.x` dans un champ du `State`.**

```dart
class _MonState extends State<MonWidget> {
  late String _nom = widget.nom;    // PIÈGE

  @override
  Widget build(BuildContext context) => Text(_nom);
}
```

`_nom` est fixé une seule fois, à la création du `State`. Si le parent envoie un
nouveau `nom`, l'écran affichera toujours l'ancien. Lisez `widget.nom` directement
dans `build()`.

La seule exception légitime : quand la valeur du parent n'est qu'une **valeur
initiale** que l'utilisateur pourra ensuite modifier. Dans ce cas, nommez-la
explicitement `valeurInitiale` et gérez la mise à jour avec `didUpdateWidget()`
(section 45.22).

**Règle 3 : `widget` n'est pas modifiable.**

```dart
widget.nom = 'Sophie';   // erreur de compilation : le champ est final
```

C'est la conséquence directe de la section 45.6.

---

## 45.17 — Le cycle de vie complet

Un `State` n'apparaît pas et ne disparaît pas au hasard. Le framework l'accompagne
en appelant une série de méthodes dans un ordre garanti.

Voici le schéma complet. Gardez-le sous les yeux pour les sections 45.18 à 45.26.

```text
                    LE WIDGET EST INSÉRÉ DANS L'ARBRE
                                  │
                                  v
                      ┌───────────────────────┐
                      │    createState()      │   crée l'objet State
                      └───────────┬───────────┘
                                  │
                                  v
                      ┌───────────────────────┐
                      │      initState()      │   UNE SEULE FOIS
                      │  super.initState()    │   abonnements, contrôleurs
                      └───────────┬───────────┘
                                  │
                                  v
                      ┌───────────────────────┐
                      │ didChangeDependencies()│  1re fois, puis à chaque
                      └───────────┬───────────┘  changement d'InheritedWidget
                                  │
                                  v
                      ┌───────────────────────┐
                      │        build()        │<──────────────┐
                      └───────────┬───────────┘               │
                                  │                           │
                                  │                           │
              ┌───────────────────┼───────────────────┐       │
              │                   │                   │       │
              v                   v                   v       │
     ┌────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
     │  setState()    │  │ didUpdateWidget │  │didChangeDep.│ │
     │ (vous l'appelez│  │ (le parent a    │  │(un héritage │ │
     │  vous-même)    │  │  reconstruit)   │  │ a changé)   │ │
     └────────┬───────┘  └────────┬────────┘  └──────┬──────┘ │
              │                   │                  │        │
              └───────────────────┴──────────────────┴────────┘
                                  │
                                  │  LE WIDGET EST RETIRÉ DE L'ARBRE
                                  v
                      ┌───────────────────────┐
                      │     deactivate()      │   retiré (peut-être
                      └───────────┬───────────┘   temporairement)
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
       réinséré ailleurs                  définitivement retiré
       dans la même frame                        │
                    │                            v
                    v                ┌───────────────────────┐
          ┌──────────────────┐       │      dispose()        │  UNE SEULE FOIS
          │   activate()     │       │  libérer les          │
          └────────┬─────────┘       │  ressources           │
                   │                 └───────────┬───────────┘
                   │                             │
                   v                             v
              retour au build()              FIN. mounted == false
```

Trois zones à distinguer :

| Zone | Méthodes | Fréquence |
| --- | --- | --- |
| **Naissance** | `createState`, `initState`, `didChangeDependencies` | une fois (sauf la dernière) |
| **Vie** | `build`, `didUpdateWidget`, `didChangeDependencies` | souvent |
| **Mort** | `deactivate`, `dispose` | une fois |

---

## 45.17.1 — Observer le cycle de vie en direct

Le meilleur moyen de comprendre est de le voir. Ce programme affiche chaque étape
dans la console.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranParent(),
    );
  }
}

class EcranParent extends StatefulWidget {
  const EcranParent({super.key});

  @override
  State<EcranParent> createState() => _EcranParentState();
}

class _EcranParentState extends State<EcranParent> {
  bool _enfantVisible = true;
  int _niveau = 1;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cycle de vie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (_enfantVisible) Espion(niveau: _niveau) else const Text('(absent)'),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _enfantVisible = !_enfantVisible;
                });
              },
              child: Text(_enfantVisible ? 'Retirer l\'enfant' : 'Ajouter l\'enfant'),
            ),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _niveau++;
                });
              },
              child: const Text('Niveau suivant'),
            ),
          ],
        ),
      ),
    );
  }
}

class Espion extends StatefulWidget {
  const Espion({super.key, required this.niveau});

  final int niveau;

  @override
  State<Espion> createState() {
    debugPrint('>> createState()');
    return _EspionState();
  }
}

class _EspionState extends State<Espion> {
  @override
  void initState() {
    super.initState();
    debugPrint('>> initState()');
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    debugPrint('>> didChangeDependencies()');
  }

  @override
  void didUpdateWidget(covariant Espion oldWidget) {
    super.didUpdateWidget(oldWidget);
    debugPrint('>> didUpdateWidget() : ${oldWidget.niveau} -> ${widget.niveau}');
  }

  @override
  void deactivate() {
    debugPrint('>> deactivate()');
    super.deactivate();
  }

  @override
  void activate() {
    super.activate();
    debugPrint('>> activate()');
  }

  @override
  void dispose() {
    debugPrint('>> dispose()');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('>> build() : niveau ${widget.niveau}');
    return Text('Niveau ${widget.niveau}', style: const TextStyle(fontSize: 32));
  }
}
```

**Résultat au lancement :**

```text
>> createState()
>> initState()
>> didChangeDependencies()
>> build() : niveau 1
```

**Résultat après un appui sur « Niveau suivant » :**

```text
>> didUpdateWidget() : 1 -> 2
>> build() : niveau 2
```

Notez que `createState()` et `initState()` ne sont **pas** rappelés : le même
`State` est réutilisé, seule la configuration a changé.

**Résultat après un appui sur « Retirer l'enfant » :**

```text
>> deactivate()
>> dispose()
```

**Résultat après « Ajouter l'enfant » :**

```text
>> createState()
>> initState()
>> didChangeDependencies()
>> build() : niveau 2
```

Un **nouveau** `State` est créé. Tout ce que l'ancien contenait est perdu.

> Faites tourner ce programme. Appuyez dans tous les ordres, faites un hot reload,
> tournez l'appareil. Vingt minutes passées ici valent trois chapitres de lecture.

---

## 45.18 — `createState()`

Signature exacte :

```dart
@override
State<MonWidget> createState() => _MonWidgetState();
```

C'est la seule méthode qu'un `StatefulWidget` doit redéfinir. Son rôle est unique :
**fabriquer l'objet `State`**.

### Quand est-elle appelée ?

Une fois par insertion du widget dans l'arbre. Pas à chaque reconstruction du
parent : uniquement quand Flutter crée un nouvel `Element` pour ce widget.

### Ce qu'il ne faut PAS y faire

```dart
  @override
  State<MonWidget> createState() {
    final etat = _MonWidgetState();
    etat.chargerDesDonnees();      // NON : trop tôt
    return etat;
  }
```

À cet instant, le `State` n'est pas encore rattaché à l'arbre : `context` n'est
pas utilisable, `widget` n'est pas encore affecté, et `setState()` lèverait une
exception.

> **Règle : `createState()` retourne le `State` et rien d'autre.** Toute
> initialisation va dans `initState()`.

### Une forme historique que vous rencontrerez

```dart
  @override
  _MonWidgetState createState() => _MonWidgetState();   // ancien style
```

Le type de retour était l'implémentation privée. La forme moderne, recommandée par
l'analyseur, utilise le type public `State<MonWidget>` :

```dart
  @override
  State<MonWidget> createState() => _MonWidgetState();  // style actuel
```

Utilisez la seconde. C'est celle que génèrent les modèles de code de l'IDE.

---

## 45.19 — `initState()`

Signature exacte :

```dart
@override
void initState() {
  super.initState();
  // votre code
}
```

`initState()` est appelée **une seule fois**, juste après la création du `State`
et juste avant le premier `build()`.

### À quoi elle sert

| Usage | Exemple |
| --- | --- |
| initialiser un état à partir du widget | `_pseudo = widget.pseudoInitial;` |
| créer un contrôleur | `_controleur = TextEditingController();` |
| démarrer un `Timer` | `_timer = Timer.periodic(...)` |
| lancer un chargement réseau | `_chargement = _recupererDonnees();` |
| s'abonner à un flux | `_abonnement = flux.listen(...)` |

### La règle du `super`

```dart
  @override
  void initState() {
    super.initState();   // TOUJOURS EN PREMIER
    _score = 0;
  }
```

Appelez `super.initState()` **avant** votre propre code. La classe parente y
effectue des opérations dont dépend le reste.

---

## 45.19.1 — Ce qu'il ne faut PAS faire dans `initState()`

**Interdit : `setState()`.**

```dart
  @override
  void initState() {
    super.initState();
    setState(() { _score = 10; });   // exception
  }
```

Le premier `build()` n'a pas encore eu lieu ; il aura donc forcément la bonne
valeur. Affectez directement :

```dart
  @override
  void initState() {
    super.initState();
    _score = 10;                     // correct
  }
```

**Fortement déconseillé : dépendre du `context` de façon héritée.**

```dart
  @override
  void initState() {
    super.initState();
    final couleur = Theme.of(context).primaryColor;   // à éviter
  }
```

Techniquement le `context` existe déjà, mais les dépendances héritées ne sont pas
encore établies proprement à cet instant. Pour tout ce qui dépend de
`Theme.of()`, `MediaQuery.of()` ou d'un `InheritedWidget`, utilisez
`didChangeDependencies()` (section 45.20) ou lisez la valeur dans `build()`.

**À encadrer : le travail asynchrone.**

`initState()` ne peut pas être `async`. On lance le travail sans l'attendre :

```dart
  @override
  void initState() {
    super.initState();
    _chargerLeProfil();     // pas de await ici
  }

  Future<void> _chargerLeProfil() async {
    final profil = await _service.recuperer();
    if (!mounted) return;   // indispensable, voir 45.26
    setState(() {
      _profil = profil;
    });
  }
```

---

## 45.19.2 — Exemple complet : un chronomètre de partie

`initState()` prend tout son sens avec un `Timer`. Voici une partie chronométrée.

```dart
import 'dart:async';

import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranChrono(),
    );
  }
}

class EcranChrono extends StatefulWidget {
  const EcranChrono({super.key});

  @override
  State<EcranChrono> createState() => _EcranChronoState();
}

class _EcranChronoState extends State<EcranChrono> {
  static const int dureeInitiale = 30;

  Timer? _timer;
  int _secondesRestantes = dureeInitiale;

  @override
  void initState() {
    super.initState();
    _demarrer();
  }

  void _demarrer() {
    _timer?.cancel();
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      if (_secondesRestantes == 0) {
        timer.cancel();
        return;
      }
      setState(() {
        _secondesRestantes--;
      });
    });
  }

  void _reinitialiser() {
    setState(() {
      _secondesRestantes = dureeInitiale;
    });
    _demarrer();
  }

  @override
  void dispose() {
    _timer?.cancel();     // INDISPENSABLE, voir 45.24
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final bool termine = _secondesRestantes == 0;

    return Scaffold(
      appBar: AppBar(title: const Text('Temps de la partie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              '$_secondesRestantes',
              style: TextStyle(
                fontSize: 72,
                color: termine ? Colors.red : Colors.black,
              ),
            ),
            Text(termine ? 'Temps écoulé' : 'secondes restantes'),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _reinitialiser,
              child: const Text('Recommencer'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après 5 secondes :**

```text
┌──────────────────────────────────┐
│ Temps de la partie               │
├──────────────────────────────────┤
│                                  │
│              25                  │
│      secondes restantes          │
│                                  │
│        [ Recommencer ]           │
│                                  │
└──────────────────────────────────┘
```

Trois choses à observer :

- le `Timer` est créé dans `initState()`, donc **une seule fois** ;
- il est annulé dans `dispose()`, donc jamais orphelin ;
- `_timer?.cancel()` dans `_demarrer()` évite d'avoir deux timers en parallèle si
  l'utilisateur appuie plusieurs fois sur « Recommencer ».

---

## 45.20 — `didChangeDependencies()`

Signature exacte :

```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  // votre code
}
```

Cette méthode est appelée :

1. **une première fois** juste après `initState()`, avant le premier `build()` ;
2. **puis à chaque fois** qu'un `InheritedWidget` dont ce `State` dépend change.

### Qu'est-ce qu'une « dépendance » ?

Vous créez une dépendance chaque fois que vous écrivez `QuelqueChose.of(context)` :

```dart
Theme.of(context)          // dépend du thème
MediaQuery.of(context)     // dépend de la taille de l'écran, de l'orientation
Localizations.of(...)      // dépend de la langue
```

Ces appels ne se contentent pas de lire une valeur : ils **inscrivent** votre
widget sur une liste d'abonnés. Quand la valeur change, Flutter appelle
`didChangeDependencies()` puis `build()`.

Les `InheritedWidget` seront étudiés en détail au chapitre 52.

### Quand la redéfinir ?

Rarement. Dans deux cas concrets :

**Cas 1 : une initialisation coûteuse qui dépend du contexte.**

```dart
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    final Locale langue = Localizations.localeOf(context);
    _dictionnaire = ChargeurDeTextes.pour(langue);   // recalculé si la langue change
  }
```

**Cas 2 : réagir à un changement d'orientation ou de taille pour recalculer autre
chose que du dessin.**

Dans l'immense majorité des cas, il suffit de lire `Theme.of(context)` directement
dans `build()`, et vous n'avez pas besoin de cette méthode.

---

## 45.20.1 — Exemple observable

Ce programme affiche la largeur disponible et journalise chaque appel.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranDependances(),
    );
  }
}

class EcranDependances extends StatefulWidget {
  const EcranDependances({super.key});

  @override
  State<EcranDependances> createState() => _EcranDependancesState();
}

class _EcranDependancesState extends State<EcranDependances> {
  double _largeurMemorisee = 0;

  @override
  void initState() {
    super.initState();
    debugPrint('initState()');
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    _largeurMemorisee = MediaQuery.sizeOf(context).width;
    debugPrint('didChangeDependencies() : largeur = $_largeurMemorisee');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dépendances')),
      body: Center(
        child: Text('Largeur mémorisée : ${_largeurMemorisee.toStringAsFixed(0)}'),
      ),
    );
  }
}
```

**Résultat au lancement puis après un redimensionnement de la fenêtre :**

```text
initState()
didChangeDependencies() : largeur = 400.0
didChangeDependencies() : largeur = 720.0
```

> `MediaQuery.sizeOf(context)` est la forme moderne et plus efficace de
> `MediaQuery.of(context).size` : elle n'abonne le widget qu'à la taille, pas à
> l'ensemble des données de `MediaQuery`. Nous y reviendrons au chapitre 51.

---

## 45.21 — `build()`

Nous l'avons déjà vue en 45.5 pour le `StatelessWidget`. Dans un `State`, elle a
exactement la même signature et les mêmes règles :

```dart
@override
Widget build(BuildContext context) {
  return ...;
}
```

### Quand est-elle appelée ?

- après `didChangeDependencies()`, pour le premier affichage ;
- après chaque `setState()` ;
- après chaque `didUpdateWidget()` ;
- après chaque `didChangeDependencies()` ;
- quand un ancêtre se reconstruit et recrée ce widget ;
- après un hot reload.

Autrement dit : **souvent, et à des moments que vous ne contrôlez pas tous**.

### La règle d'or, répétée

`build()` doit être **pure** : elle lit `widget`, elle lit les variables d'état,
elle lit le `context`, elle retourne un arbre. Elle ne modifie rien.

Un test mental infaillible :

> Si j'appelle `build()` cent fois de suite sans rien changer d'autre, l'application
> se comporte-t-elle exactement pareil ?

Si la réponse est non, `build()` contient quelque chose qui ne devrait pas y être.

---

## 45.21.1 — Ce que `build()` peut lire sans risque

```dart
  @override
  Widget build(BuildContext context) {
    // 1. les propriétés du widget
    final String titre = widget.titre;

    // 2. les variables d'état
    final int score = _score;

    // 3. le contexte
    final ThemeData theme = Theme.of(context);

    // 4. des valeurs dérivées calculées ici
    final String mention = score > 1000 ? 'Expert' : 'Novice';

    return Text('$titre — $score — $mention', style: theme.textTheme.bodyLarge);
  }
```

Les quatre sont légitimes. La variable `mention` en particulier : elle est
**dérivée** de l'état, elle ne doit surtout pas être stockée dans un champ, sinon
il faudrait penser à la mettre à jour à chaque changement de score.

---

## 45.22 — `didUpdateWidget()`

Signature exacte :

```dart
@override
void didUpdateWidget(covariant MonWidget oldWidget) {
  super.didUpdateWidget(oldWidget);
  // votre code
}
```

Cette méthode est appelée quand **le parent reconstruit ce widget avec une nouvelle
configuration**, alors que le `State` est conservé.

```text
   Le parent fait setState()
              │
              v
   Il crée un NOUVEAU widget Espion(niveau: 2)
              │
              v
   Flutter voit : même type, même Key  ->  je garde le State
              │
              v
   Il remplace la propriété widget du State
              │
              v
   didUpdateWidget(oldWidget)   <- oldWidget = Espion(niveau: 1)
                                    widget    = Espion(niveau: 2)
              │
              v
   build()
```

Le mot-clé `covariant` autorise à typer le paramètre avec votre type précis
(`Espion`) au lieu du type générique. Écrivez-le, l'analyseur le réclame.

### À quoi elle sert

À **réagir au changement d'une propriété**, typiquement pour reconfigurer une
ressource créée dans `initState()`.

---

## 45.22.1 — Le cas classique : une durée qui change

```dart
import 'dart:async';

import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranReglages(),
    );
  }
}

class EcranReglages extends StatefulWidget {
  const EcranReglages({super.key});

  @override
  State<EcranReglages> createState() => _EcranReglagesState();
}

class _EcranReglagesState extends State<EcranReglages> {
  int _cadence = 1;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Régénération d\'énergie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            RegenerationEnergie(secondesParPoint: _cadence),
            const SizedBox(height: 32),
            Text('Cadence actuelle : 1 point / $_cadence s'),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _cadence = _cadence == 1 ? 3 : 1;
                });
              },
              child: const Text('Changer la cadence'),
            ),
          ],
        ),
      ),
    );
  }
}

class RegenerationEnergie extends StatefulWidget {
  const RegenerationEnergie({super.key, required this.secondesParPoint});

  final int secondesParPoint;

  @override
  State<RegenerationEnergie> createState() => _RegenerationEnergieState();
}

class _RegenerationEnergieState extends State<RegenerationEnergie> {
  Timer? _timer;
  int _energie = 0;

  @override
  void initState() {
    super.initState();
    _programmerLeTimer();
  }

  @override
  void didUpdateWidget(covariant RegenerationEnergie oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.secondesParPoint != widget.secondesParPoint) {
      debugPrint('cadence modifiée, on reprogramme le timer');
      _programmerLeTimer();
    }
  }

  void _programmerLeTimer() {
    _timer?.cancel();
    _timer = Timer.periodic(
      Duration(seconds: widget.secondesParPoint),
      (_) {
        setState(() {
          _energie = (_energie + 1) % 101;
        });
      },
    );
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text('Énergie : $_energie %', style: const TextStyle(fontSize: 28));
  }
}
```

**Résultat dans la console après un appui sur « Changer la cadence » :**

```text
cadence modifiée, on reprogramme le timer
```

Sans `didUpdateWidget()`, le timer resterait à son ancienne cadence pour toujours :
`initState()` n'est plus rappelé, et rien n'aurait pris en compte la nouvelle
valeur.

---

## 45.22.2 — Le test de comparaison est obligatoire

```dart
  @override
  void didUpdateWidget(covariant RegenerationEnergie oldWidget) {
    super.didUpdateWidget(oldWidget);
    _programmerLeTimer();     // MAUVAIS : sans test
  }
```

`didUpdateWidget()` est appelée à **chaque** reconstruction du parent, même si la
propriété n'a pas bougé. Sans le test `oldWidget.x != widget.x`, vous détruisez et
recréez le timer sans arrêt, et le compte à rebours ne progresse jamais.

> **Toujours comparer l'ancienne et la nouvelle valeur avant d'agir.**

---

## 45.23 — `deactivate()`

Signature exacte :

```dart
@override
void deactivate() {
  // votre code
  super.deactivate();
}
```

Cette méthode est appelée quand le `State` est **retiré** de l'arbre. Deux
scénarios possibles :

```text
   deactivate()
        │
        ├── le widget est réinséré ailleurs dans la MÊME frame
        │        -> activate() est appelée, la vie continue
        │
        └── le widget n'est pas réinséré
                 -> dispose() est appelée, c'est fini
```

Le premier cas se produit quand un widget change de place dans l'arbre en
conservant sa `GlobalKey`. C'est rare.

### Quand la redéfinir ?

Presque jamais. Elle sert essentiellement à se désabonner de quelque chose qui
dépend de la position dans l'arbre, et à annuler un enregistrement auprès d'un
ancêtre.

> **Pour un débutant : ignorez `deactivate()`.** Mettez votre nettoyage dans
> `dispose()`. Sachez simplement qu'elle existe et qu'elle passe avant `dispose()`.

Notez l'ordre inversé du `super` : dans `initState()` et `didUpdateWidget()`, on
appelle `super` **en premier** ; dans `deactivate()` et `dispose()`, on l'appelle
**en dernier**. La logique est simple : on construit de l'extérieur vers
l'intérieur, on détruit de l'intérieur vers l'extérieur.

---

## 45.24 — `dispose()`

Signature exacte :

```dart
@override
void dispose() {
  // libérer vos ressources ici
  super.dispose();
}
```

`dispose()` est appelée **une seule fois**, quand le `State` est retiré
définitivement. Après elle, `mounted` vaut `false` et l'objet ne sera plus jamais
utilisé.

### C'est le moment de tout rendre

| Ressource créée | À faire dans `dispose()` |
| --- | --- |
| `TextEditingController` | `_controleur.dispose();` |
| `AnimationController` | `_animation.dispose();` |
| `ScrollController` | `_defilement.dispose();` |
| `PageController` | `_pages.dispose();` |
| `FocusNode` | `_focus.dispose();` |
| `Timer` | `_timer?.cancel();` |
| `StreamSubscription` | `_abonnement.cancel();` |
| écouteur ajouté avec `addListener` | `.removeListener(...)` |

### Pourquoi c'est indispensable

Une ressource non libérée continue d'exister après la disparition de l'écran.
Conséquences, dans l'ordre de gravité croissante :

1. **Fuite mémoire** : l'objet reste, et il retient tout le `State`, donc tout
   l'écran ;
2. **Travail inutile** : un `Timer` continue de tourner, un flux continue d'émettre ;
3. **Plantage** : le rappel appelle `setState()` sur un `State` démonté, et Flutter
   lève une exception (voir 45.26).

Multipliez par vingt écrans ouverts et fermés dans une session, et l'application
devient lente puis instable.

---

## 45.25 — Libérer un contrôleur dans `dispose()`

Voici l'exemple le plus fréquent en pratique : un `TextEditingController`.

Nous anticipons légèrement sur le chapitre 49, qui traitera les formulaires en
détail. Ici, seule la gestion du cycle de vie nous intéresse.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranNomDuHeros(),
    );
  }
}

class EcranNomDuHeros extends StatefulWidget {
  const EcranNomDuHeros({super.key});

  @override
  State<EcranNomDuHeros> createState() => _EcranNomDuHerosState();
}

class _EcranNomDuHerosState extends State<EcranNomDuHeros> {
  late final TextEditingController _controleurNom;
  String _nomValide = 'Sans nom';

  @override
  void initState() {
    super.initState();
    _controleurNom = TextEditingController(text: 'Alex');
  }

  @override
  void dispose() {
    _controleurNom.dispose();   // LIBÉRATION
    super.dispose();
  }

  void _valider() {
    final String saisie = _controleurNom.text.trim();
    setState(() {
      _nomValide = saisie.isEmpty ? 'Sans nom' : saisie;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Nom du héros')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _controleurNom,
              decoration: const InputDecoration(
                labelText: 'Nom du personnage',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: _valider,
              child: const Text('Valider'),
            ),
            const SizedBox(height: 24),
            Text('Héros enregistré : $_nomValide',
                style: const TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après avoir saisi « Sophie » et validé :**

```text
┌────────────────────────────────────────┐
│ Nom du héros                           │
├────────────────────────────────────────┤
│ ┌────────────────────────────────────┐ │
│ │ Nom du personnage                  │ │
│ │ Sophie                             │ │
│ └────────────────────────────────────┘ │
│            [ Valider ]                 │
│                                        │
│ Héros enregistré : Sophie              │
└────────────────────────────────────────┘
```

Trois points de méthode :

**`late final`** — le contrôleur ne peut pas être créé à la déclaration si l'on
veut y mettre `widget.quelqueChose`. `late` (chapitre 12) diffère l'initialisation
jusqu'à `initState()`, `final` interdit de le remplacer ensuite.

**Le contrôleur n'est PAS de l'état déclaré avec `setState()`** — il gère
lui-même son texte et notifie le `TextField`. On l'utilise pour **lire** la saisie
au moment voulu.

**`dispose()` appelle `_controleurNom.dispose()` avant `super.dispose()`.**

---

## 45.25.1 — Ce qui se passe si vous oubliez

Retirez la méthode `dispose()` de l'exemple précédent, puis, en mode debug,
ouvrez et fermez l'écran une centaine de fois. Flutter finit par signaler la fuite.
Avec certains contrôleurs, vous obtenez directement :

```text
A TextEditingController was used after being disposed.
Once you have called dispose() on a TextEditingController, it can no longer
be used.
```

ou, en cas d'oubli inverse (contrôleur jamais libéré) :

```text
An instance of TextEditingController was leaked. Instances of this class
should be disposed when no longer in use.
```

> **Réflexe à automatiser : dès que vous écrivez `= SomethingController()`,
> écrivez immédiatement le `dispose()` correspondant. Avant même de continuer.**

---

## 45.25.2 — Le modèle complet à recopier

```dart
class _MonEcranState extends State<MonEcran> {
  late final TextEditingController _texte;
  late final ScrollController _defilement;
  Timer? _timer;
  StreamSubscription<int>? _abonnement;

  @override
  void initState() {
    super.initState();
    _texte = TextEditingController();
    _defilement = ScrollController();
    _timer = Timer.periodic(const Duration(seconds: 1), (_) {});
    // _abonnement = monFlux.listen((v) { ... });
  }

  @override
  void dispose() {
    _abonnement?.cancel();
    _timer?.cancel();
    _defilement.dispose();
    _texte.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => const SizedBox.shrink();
}
```

Notez la symétrie : ce qui est créé en premier dans `initState()` est libéré en
dernier dans `dispose()`. Ce n'est pas obligatoire, mais c'est une bonne habitude
qui vous évitera des oublis.

---

## 45.26 — `mounted` et le `setState()` après démontage

`mounted` est un booléen fourni par `State` :

```dart
bool get mounted   // vrai tant que ce State est dans l'arbre
```

- il vaut `false` avant `initState()` ;
- il vaut `true` de `initState()` jusqu'à `dispose()` ;
- il vaut `false` après `dispose()`, définitivement.

### Le problème qu'il résout

Reprenons le scénario asynchrone du chapitre 15 :

```text
   t=0    l'utilisateur ouvre l'écran
   t=0    initState() lance une requête réseau (2 secondes)
   t=1    l'utilisateur revient en arrière -> dispose(), mounted = false
   t=2    la requête se termine et appelle setState()
              │
              v
          EXCEPTION
```

Message affiché :

```text
setState() called after dispose(): _EcranProfilState#a1b2c(lifecycle state:
defunct, not mounted)

This error happens if you call setState() on a State object for a widget that
no longer appears in the widget tree.
```

### La solution

```dart
  Future<void> _chargerLeProfil() async {
    final profil = await _service.recuperer();   // opération longue
    if (!mounted) return;                        // LE GARDE-FOU
    setState(() {
      _profil = profil;
    });
  }
```

La règle est mécanique :

> **Après chaque `await`, si la ligne suivante utilise `setState()` ou le
> `context`, testez `mounted` d'abord.**

---

## 45.26.1 — Exemple complet et vérifiable

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranChargement(),
    );
  }
}

class EcranChargement extends StatefulWidget {
  const EcranChargement({super.key});

  @override
  State<EcranChargement> createState() => _EcranChargementState();
}

class _EcranChargementState extends State<EcranChargement> {
  bool _enCours = true;
  String _resultat = '';

  @override
  void initState() {
    super.initState();
    _chargerLeClassement();
  }

  Future<void> _chargerLeClassement() async {
    // simulation d'un appel réseau de 3 secondes (chapitre 15)
    await Future<void>.delayed(const Duration(seconds: 3));

    if (!mounted) {
      debugPrint('écran déjà fermé : on abandonne proprement');
      return;
    }

    setState(() {
      _enCours = false;
      _resultat = '1. Sophie 4200\n2. Alex 3150\n3. Samir 2980';
    });
  }

  @override
  void dispose() {
    debugPrint('dispose() : mounted passe à false');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Classement')),
      body: Center(
        child: _enCours
            ? const CircularProgressIndicator()
            : Text(_resultat, style: const TextStyle(fontSize: 20)),
      ),
    );
  }
}
```

**Résultat pendant les 3 premières secondes :**

```text
┌──────────────────────────────────┐
│ Classement                       │
├──────────────────────────────────┤
│                                  │
│              (o)                 │   indicateur circulaire
│                                  │
└──────────────────────────────────┘
```

**Résultat après 3 secondes :**

```text
┌──────────────────────────────────┐
│ Classement                       │
├──────────────────────────────────┤
│      1. Sophie   4200            │
│      2. Alex     3150            │
│      3. Samir    2980            │
└──────────────────────────────────┘
```

Si l'écran est fermé avant la fin (au chapitre 50, avec `Navigator.pop`), la
console affiche :

```text
dispose() : mounted passe à false
écran déjà fermé : on abandonne proprement
```

Aucune exception. C'est exactement le comportement voulu.

---

## 45.26.2 — `mounted` et le `context` après un `await`

Le même problème existe pour le `context`. Utiliser un `BuildContext` après un
`await` est signalé par l'analyseur Dart :

```text
warning: Don't use 'BuildContext's across async gaps.
use_build_context_synchronously
```

La correction est la même :

```dart
  Future<void> _sauvegarderPuisAvertir() async {
    await _service.sauvegarder();
    if (!mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Partie sauvegardée')),
    );
  }
```

> **Le test `if (!mounted) return;` juste après un `await` doit devenir un
> automatisme.** Il ne coûte rien et supprime toute une famille de plantages.

---

## 45.27 — État local ou état remonté : où le placer

Vous savez maintenant **comment** gérer un état. Reste la question la plus
importante en pratique : **où** le placer dans l'arbre.

La règle générale :

> **Placez l'état au niveau du plus proche ancêtre commun à tous les widgets qui
> en ont besoin. Ni plus haut, ni plus bas.**

Deux forces s'opposent :

```text
   ÉTAT TROP BAS                        ÉTAT TROP HAUT
   ─────────────                        ──────────────
   les autres widgets                   tout l'écran se reconstruit
   ne peuvent pas y accéder             à chaque petit changement

   -> il faut le remonter               -> il faut le redescendre
```

---

## 45.27.1 — Cas 1 : un seul widget concerné, gardez l'état local

Un panneau dépliable : personne d'autre n'a besoin de savoir s'il est ouvert.

```text
        EcranJeu  (stateless)
             │
        ┌────┴────────────────────┐
        │                         │
   PanneauInventaire        PanneauQuetes
   _estOuvert = true        _estOuvert = false
   (état LOCAL)             (état LOCAL)
```

Chaque panneau gère son propre `_estOuvert`. Aucun des deux ne se soucie de
l'autre. C'est le meilleur cas : `setState()` ne reconstruit que le panneau
concerné, et l'écran parent n'est même pas stateful.

---

## 45.27.2 — Cas 2 : deux widgets concernés, remontez l'état

Un compteur de pièces et un bouton d'achat qui doit se griser quand il n'y a plus
assez d'argent.

```text
        EcranBoutique  (STATEFUL)
        _pieces = 120                <- l'état vit ICI
             │
        ┌────┴────────────────────┐
        │                         │
   PorteMonnaie              BoutonAchat
   (stateless)               (stateless)
   reçoit 120                reçoit 120 et prix 150
                             -> désactivé
```

Ni `PorteMonnaie` ni `BoutonAchat` ne peuvent détenir `_pieces` : ils en ont
besoin tous les deux. On le remonte au premier ancêtre commun.

---

## 45.27.3 — Cas 3 : tout l'écran concerné, restez au niveau de l'écran

Un mode « pause » qui change l'apparence de tout l'écran : l'état va dans le
`State` de l'écran. C'est légitime, et cela ne pose pas de problème tant que
l'écran n'est pas énorme.

---

## 45.27.4 — Le contre-exemple à éviter

```text
        MonApplication  (STATEFUL)
        _quantitePotions = 3        <- BEAUCOUP TROP HAUT
             │
        MaterialApp
             │
        EcranJeu
             │
        ... 8 niveaux ...
             │
        CompteurDePotions
```

Chaque `+1` sur les potions provoquerait la reconstruction de `MaterialApp` et de
tout ce qu'il contient. C'est le symptôme d'une application mal découpée, et c'est
précisément ce que le chapitre 52 corrigera avec `provider`.

---

## 45.28 — La remontée d'état (*lifting state up*)

La **remontée d'état** est l'opération qui consiste à déplacer une variable d'état
d'un enfant vers un ancêtre commun. C'est la technique la plus importante de cette
section.

### Le problème, en code

Voici une version **cassée** : deux widgets, chacun avec son propre état, qui
devraient partager la même information.

```dart
// CE CODE NE FONCTIONNE PAS COMME VOULU
class _PorteMonnaieState extends State<PorteMonnaie> {
  int _pieces = 120;              // premier exemplaire de la donnée
  // ...
}

class _BoutonAchatState extends State<BoutonAchat> {
  int _pieces = 120;              // deuxième exemplaire, indépendant
  // ...
}
```

Deux variables, deux vérités. Acheter une potion décrémente l'une, pas l'autre.
L'écran devient incohérent.

> **Une information ne doit exister qu'à un seul endroit.** C'est la règle de la
> source unique de vérité.

---

## 45.28.1 — La remontée, étape par étape

```text
   AVANT                                APRÈS

   EcranBoutique (stateless)            EcranBoutique (STATEFUL)
        │                                    │  _pieces = 120
        ├── PorteMonnaie                     ├── PorteMonnaie(pieces: 120)
        │     _pieces = 120  <- ici          │     (stateless)
        │                                    │
        └── BoutonAchat                      └── BoutonAchat(
              _pieces = 120  <- et ici             pieces: 120,
                                                   prix: 150,
                                                   onAcheter: _acheter,
                                                 )   (stateless)
```

Quatre étapes, toujours les mêmes :

1. identifier le plus proche ancêtre commun ;
2. transformer cet ancêtre en `StatefulWidget` s'il ne l'est pas ;
3. y déplacer la variable d'état ;
4. la redistribuer aux enfants par leurs constructeurs, et remonter les actions
   par des callbacks (section 45.29).

---

## 45.28.2 — Exemple complet de remontée d'état

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranBoutique(),
    );
  }
}

/// ÉTAPE 2 et 3 : l'ancêtre commun devient stateful et détient l'état.
class EcranBoutique extends StatefulWidget {
  const EcranBoutique({super.key});

  @override
  State<EcranBoutique> createState() => _EcranBoutiqueState();
}

class _EcranBoutiqueState extends State<EcranBoutique> {
  int _pieces = 120;
  int _potions = 0;

  static const int prixPotion = 50;

  void _acheterUnePotion() {
    if (_pieces < prixPotion) return;
    setState(() {
      _pieces -= prixPotion;
      _potions += 1;
    });
  }

  void _vendreUnePotion() {
    if (_potions == 0) return;
    setState(() {
      _pieces += prixPotion ~/ 2;
      _potions -= 1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Boutique du village')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // ÉTAPE 4 : on redistribue la donnée...
            PorteMonnaie(pieces: _pieces),
            const SizedBox(height: 8),
            SacAObjets(nombreDePotions: _potions),
            const SizedBox(height: 24),
            // ...et on remonte les actions par callbacks.
            BoutonAchat(
              libelle: 'Acheter une potion ($prixPotion pièces)',
              actif: _pieces >= prixPotion,
              onAppui: _acheterUnePotion,
            ),
            const SizedBox(height: 8),
            BoutonAchat(
              libelle: 'Revendre une potion (${prixPotion ~/ 2} pièces)',
              actif: _potions > 0,
              onAppui: _vendreUnePotion,
            ),
          ],
        ),
      ),
    );
  }
}

class PorteMonnaie extends StatelessWidget {
  const PorteMonnaie({super.key, required this.pieces});

  final int pieces;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: const Icon(Icons.savings),
        title: const Text('Bourse'),
        trailing: Text('$pieces', style: const TextStyle(fontSize: 20)),
      ),
    );
  }
}

class SacAObjets extends StatelessWidget {
  const SacAObjets({super.key, required this.nombreDePotions});

  final int nombreDePotions;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: const Icon(Icons.backpack),
        title: const Text('Potions'),
        trailing: Text('$nombreDePotions', style: const TextStyle(fontSize: 20)),
      ),
    );
  }
}

class BoutonAchat extends StatelessWidget {
  const BoutonAchat({
    super.key,
    required this.libelle,
    required this.actif,
    required this.onAppui,
  });

  final String libelle;
  final bool actif;
  final VoidCallback onAppui;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: actif ? onAppui : null,
        child: Text(libelle),
      ),
    );
  }
}
```

**Résultat après deux achats :**

```text
┌──────────────────────────────────────────┐
│ Boutique du village                      │
├──────────────────────────────────────────┤
│ [$] Bourse                          20   │
│ [B] Potions                          2   │
│                                          │
│ [   Acheter une potion (50 pièces)   ]   │   grisé : 20 < 50
│ [ Revendre une potion (25 pièces)    ]   │
└──────────────────────────────────────────┘
```

Observez la répartition finale :

- **un seul** `StatefulWidget` : `EcranBoutique` ;
- **trois** `StatelessWidget` : ils affichent, ils ne décident de rien ;
- une **source unique de vérité** : `_pieces` et `_potions` ;
- les enfants sont **réutilisables** : `PorteMonnaie` fonctionnerait dans n'importe
  quel écran.

---

## 45.29 — Les callbacks pour faire redescendre l'action (rappel chapitre 07)

Nous avons vu en 45.7.1 qu'un enfant ne peut pas envoyer une donnée à son parent.
Le **callback** contourne cela.

Rappel du chapitre 07 : en Dart, une fonction est une valeur. On peut la stocker
dans une variable, la passer en paramètre, la retourner.

```text
   1. Le parent DONNE sa méthode à l'enfant
                    │
        parent ──── onAppui: _acheterUnePotion ────> enfant

   2. L'enfant STOCKE la fonction sans savoir ce qu'elle fait

   3. Au bon moment, l'enfant APPELLE la fonction
                    │
        enfant ──── onAppui() ────> exécute _acheterUnePotion du parent

   4. Le parent fait son setState() et se reconstruit
```

La donnée descend, l'action remonte. C'est le schéma fondamental de Flutter.

---

## 45.29.1 — Les types de callbacks

| Type Dart | Signature | Usage typique |
| --- | --- | --- |
| `VoidCallback` | `void Function()` | un appui sur un bouton |
| `ValueChanged<T>` | `void Function(T value)` | une valeur choisie, un texte saisi |
| `void Function(int, String)` | quelconque | plusieurs informations à remonter |
| `Future<void> Function()` | asynchrone | un rafraîchissement |

`VoidCallback` et `ValueChanged<T>` sont de simples alias définis par Flutter.
`VoidCallback` est identique à `void Function()`.

---

## 45.29.2 — Exemple avec `ValueChanged<T>`

L'enfant doit dire **laquelle** des armes a été choisie. Un `VoidCallback` ne
suffit plus : il faut transporter une valeur.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

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
  static const List<String> armes = ['Épée', 'Arc', 'Bâton', 'Dague'];

  String _armeEquipee = 'Épée';
  int _nombreDeChangements = 0;

  void _equiper(String arme) {
    if (arme == _armeEquipee) return;    // garde : voir 45.15
    setState(() {
      _armeEquipee = arme;
      _nombreDeChangements++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Armurerie')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('Arme équipée : $_armeEquipee',
                style: const TextStyle(fontSize: 22)),
            Text('Changements : $_nombreDeChangements'),
            const SizedBox(height: 16),
            for (final String arme in armes)
              LigneArme(
                nom: arme,
                estEquipee: arme == _armeEquipee,
                onChoisie: _equiper,        // ValueChanged<String>
              ),
          ],
        ),
      ),
    );
  }
}

class LigneArme extends StatelessWidget {
  const LigneArme({
    super.key,
    required this.nom,
    required this.estEquipee,
    required this.onChoisie,
  });

  final String nom;
  final bool estEquipee;
  final ValueChanged<String> onChoisie;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        title: Text(nom),
        trailing: estEquipee ? const Icon(Icons.check) : null,
        selected: estEquipee,
        onTap: () => onChoisie(nom),   // l'enfant remonte SA valeur
      ),
    );
  }
}
```

**Résultat après avoir choisi « Arc » :**

```text
┌────────────────────────────────────────┐
│ Armurerie                              │
├────────────────────────────────────────┤
│ Arme équipée : Arc                     │
│ Changements : 1                        │
│                                        │
│ ┌────────────────────────────────────┐ │
│ │ Épée                               │ │
│ ├────────────────────────────────────┤ │
│ │ Arc                            (v) │ │
│ ├────────────────────────────────────┤ │
│ │ Bâton                              │ │
│ ├────────────────────────────────────┤ │
│ │ Dague                              │ │
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

Points clés :

- `LigneArme` est `StatelessWidget` : elle ne décide pas de la sélection, elle la
  **signale** ;
- `onTap: () => onChoisie(nom)` enveloppe l'appel dans une fonction anonyme parce
  que `onTap` attend un `VoidCallback` sans paramètre ;
- `for (final String arme in armes)` est un *collection for* (chapitre 06) : il
  construit les quatre lignes.

---

## 45.29.3 — Le piège des parenthèses

Une erreur que tout le monde fait au moins une fois :

```dart
onTap: onChoisie(nom)      // FAUX : appelle la fonction pendant build()
onTap: () => onChoisie(nom) // CORRECT : passe une fonction à appeler plus tard
```

Dans la première version, `onChoisie(nom)` est **exécutée immédiatement**, pendant
la construction de l'écran. Comme elle contient un `setState()`, vous obtenez
l'exception vue en 45.15.1 :

```text
setState() or markNeedsBuild() called during build.
```

De plus, `onTap` reçoit alors le **résultat** de l'appel, c'est-à-dire `null`.

> Sans parenthèses, on parle de la fonction. Avec parenthèses, on l'exécute.

---

## 45.30 — Les limites de cette approche (annonce du chapitre 52)

Vous savez maintenant tout gérer avec `setState()` et la remontée d'état. Cette
approche est **suffisante** pour un écran, et même pour une petite application.

Elle atteint pourtant ses limites, et il faut savoir lesquelles.

### Limite 1 : le forage de propriétés (*prop drilling*)

```text
   EcranPrincipal   _pieces = 120
        │
        └─ Corps(pieces: 120)
              │
              └─ Colonne(pieces: 120)
                    │
                    └─ Panneau(pieces: 120)
                          │
                          └─ Ligne(pieces: 120)
                                │
                                └─ Badge(pieces: 120)   <- le seul qui en a besoin
```

Cinq widgets intermédiaires doivent déclarer une propriété dont ils ne font rien.
Ajouter une donnée oblige à modifier six fichiers. C'est pénible et fragile.

### Limite 2 : les reconstructions trop larges

L'état est en haut, donc `setState()` reconstruit tout le sous-arbre, y compris
des dizaines de widgets qui n'ont pas changé.

### Limite 3 : le partage entre écrans

`setState()` ne fonctionne qu'à l'intérieur d'un arbre. Deux écrans différents
(chapitre 50) ne peuvent pas partager un panier d'achat par ce moyen : quand vous
quittez l'écran, son `State` est détruit.

### Limite 4 : la logique métier mélangée à l'interface

Les règles du jeu (le prix d'une potion, la limite de vies) se retrouvent dans une
classe `State`, à côté du code d'affichage. Impossible de les tester sans lancer
l'interface.

### Ce que le chapitre 52 apportera

| Outil | Ce qu'il résout |
| --- | --- |
| `InheritedWidget` | accéder à une donnée sans la faire transiter par tous les niveaux |
| `ChangeNotifier` | un objet qui détient l'état et prévient ses abonnés |
| `provider` | le tout, avec une syntaxe simple et des reconstructions ciblées |

> **N'anticipez pas.** Beaucoup de débutants installent `provider` pour un
> compteur. C'est une erreur : `setState()` reste la solution correcte pour un
> état local. On change d'outil quand le problème l'exige, pas avant.

---

## 45.31 — Stateless ou Stateful : arbre de décision

Voici la question à se poser, sous forme d'arbre. Suivez-le à chaque nouveau
widget.

```text
                  J'écris un nouveau widget
                            │
                            v
        Ce widget doit-il SE SOUVENIR d'une information
        entre deux reconstructions, et la modifier lui-même ?
                            │
              ┌─────────────┴─────────────┐
             NON                         OUI
              │                           │
              v                           v
    ┌──────────────────┐      Ai-je besoin d'initialiser
    │ StatelessWidget  │      ou de libérer une ressource
    └──────────────────┘      (contrôleur, timer, flux) ?
                                          │
                              ┌───────────┴───────────┐
                             NON                     OUI
                              │                       │
                              v                       v
                  ┌──────────────────┐   ┌────────────────────────┐
                  │ StatefulWidget   │   │ StatefulWidget         │
                  │ + setState()     │   │ + initState()          │
                  └──────────────────┘   │ + dispose()            │
                                         │ + setState()           │
                                         └────────────────────────┘

        Et dans TOUS les cas, avant de créer un StatefulWidget :
                            │
                            v
        Cette information est-elle utile à un AUTRE widget ?
                            │
              ┌─────────────┴─────────────┐
             NON                         OUI
              │                           │
              v                           v
      état LOCAL, ici           REMONTER l'état chez l'ancêtre
                                commun, et redescendre
                                données + callbacks (45.28)
```

---

## 45.31.1 — Le tableau de décision rapide

| Situation | Réponse |
| --- | --- |
| une carte qui affiche un nom reçu du parent | `StatelessWidget` |
| un bouton qui appelle un callback du parent | `StatelessWidget` |
| un écran entier composé d'éléments figés | `StatelessWidget` |
| un compteur avec un bouton `+` | `StatefulWidget` |
| une case à cocher qui retient son état | `StatefulWidget` |
| un panneau qui se déplie | `StatefulWidget` |
| un `TextField` dont on lit la valeur | `StatefulWidget` (contrôleur) |
| une animation | `StatefulWidget` (`AnimationController`) |
| un écran qui charge des données au démarrage | `StatefulWidget` (`initState`) |
| une valeur partagée par deux widgets frères | `StatefulWidget` sur l'ancêtre commun |
| une valeur partagée par deux écrans | ni l'un ni l'autre : chapitre 52 |

---

## 45.31.2 — Trois conseils pour finir

**Conseil 1 : commencez toujours par `StatelessWidget`.** Convertissez en
`StatefulWidget` seulement quand vous butez sur un besoin réel. L'IDE propose la
conversion automatique (ampoule « Convert to StatefulWidget »).

**Conseil 2 : gardez les `StatefulWidget` petits.** Un écran de 600 lignes avec
douze variables d'état est ingérable. Découpez : chaque sous-widget qui a un état
propre le gère lui-même.

**Conseil 3 : mettez l'état le plus bas possible.** C'est bon pour les
performances, et c'est bon pour la lisibilité.

---

## 45.32 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| L'écran ne change pas quand j'appuie | l'état est modifié hors de `setState()` | encadrer la modification : `setState(() { _score += 10; });` |
| La valeur n'apparaît qu'après rotation ou hot reload | même cause : `build()` est relancé pour une autre raison | ajouter le `setState()` manquant |
| `A TextEditingController was leaked` | contrôleur créé mais jamais libéré | redéfinir `dispose()` et appeler `_controleur.dispose();` |
| `setState() or markNeedsBuild() called during build` | `setState()` appelé depuis `build()` ou `initState()` | dans `build()`, ne rien modifier ; dans `initState()`, affecter directement sans `setState()` |
| `onPressed` déclenche l'action au chargement | parenthèses en trop : `onPressed: _action()` | passer la référence : `onPressed: _action` ou `onPressed: () => _action(x)` |
| Le compteur repart à zéro tout seul | variable d'état déclarée dans le `StatefulWidget` au lieu du `State` | déplacer la variable dans la classe `_XxxState` |
| `This class is marked as '@immutable' but ... aren't final` | propriété de widget non `final` | ajouter `final` devant chaque champ du widget |
| `setState() called after dispose()` | `setState()` après un `await`, alors que l'écran est fermé | insérer `if (!mounted) return;` juste après l'`await` |
| `Don't use 'BuildContext's across async gaps` | `context` utilisé après un `await` | insérer `if (!mounted) return;` avant d'utiliser `context` |
| Le timer ne se met jamais à jour après un changement de réglage | `initState()` n'est pas rappelé quand une propriété change | redéfinir `didUpdateWidget()` et reconfigurer la ressource |
| L'application se fige, la console défile sans fin | `didUpdateWidget()` recrée une ressource sans comparer l'ancienne valeur | comparer : `if (oldWidget.x != widget.x) { ... }` |
| Deux timers tournent en même temps | `Timer.periodic` relancé sans annuler le précédent | appeler `_timer?.cancel();` avant de recréer |
| `_inventaire.add(...)` ne rafraîchit rien | modification en place d'une liste, hors `setState()` | `setState(() { _inventaire.add(objet); });` |
| Deux widgets affichent des valeurs différentes pour la même donnée | la donnée est dupliquée dans deux `State` | remonter l'état chez l'ancêtre commun (45.28) |
| L'enfant modifie `widget.valeur` | tentative d'écrire dans un champ `final` | remonter l'action par un callback vers le parent |
| Tout l'écran clignote au moindre changement | l'état est placé trop haut dans l'arbre | descendre l'état dans le sous-widget concerné |
| La valeur du parent n'arrive jamais à jour dans l'enfant | `widget.x` recopié une fois dans un champ du `State` | lire `widget.x` directement dans `build()` |
| `super.initState()` oublié | appel manquant à la classe parente | toujours `super.initState();` en première ligne |
| `dispose()` appelle `super.dispose()` en premier | ordre inversé : la libération arrive après la destruction | libérer d'abord, `super.dispose();` en dernier |
| `setState(() async { ... })` ne fonctionne pas | `setState()` attend une fonction synchrone | faire l'`await` avant, puis appeler `setState()` |

---

## 45.33 — Résumé du chapitre

### Les deux types de widgets

| Type | Contient un état ? | Nombre de classes | Méthode obligatoire |
| --- | --- | --- | --- |
| `StatelessWidget` | non | 1 | `build(context)` |
| `StatefulWidget` | oui, dans son `State` | 2 | `createState()` et `build(context)` |

### Le cycle de vie d'un `State`

| Méthode du cycle de vie | Quand est-elle appelée | À quoi elle sert |
| --- | --- | --- |
| `createState()` | à l'insertion du widget dans l'arbre, une fois | créer l'objet `State` — rien d'autre |
| `initState()` | juste après la création du `State`, une fois | initialiser l'état, créer les contrôleurs, lancer un chargement, s'abonner |
| `didChangeDependencies()` | après `initState()`, puis à chaque changement d'un `InheritedWidget` dont on dépend | recalculer ce qui dépend du thème, de la langue, de la taille d'écran |
| `build()` | après les trois précédentes, puis après chaque `setState()`, `didUpdateWidget()` ou reconstruction du parent | décrire l'interface — jamais agir |
| `didUpdateWidget(oldWidget)` | quand le parent reconstruit ce widget avec une nouvelle configuration | comparer ancienne et nouvelle propriété, reconfigurer une ressource |
| `deactivate()` | quand le `State` est retiré de l'arbre | rarement utilisée ; nettoyage lié à la position dans l'arbre |
| `activate()` | si le `State` est réinséré ailleurs dans la même frame | rarement utilisée ; reprendre après un `deactivate()` |
| `dispose()` | au retrait définitif, une fois | libérer contrôleurs, timers, abonnements |

### Les membres utiles de `State`

| Membre | Type | À retenir |
| --- | --- | --- |
| `widget` | le type du widget | la configuration courante, toujours à jour, en lecture seule |
| `context` | `BuildContext` | la position dans l'arbre ; inutilisable après `dispose()` |
| `mounted` | `bool` | `true` entre `initState()` et `dispose()` ; à tester après chaque `await` |
| `setState(fn)` | méthode | exécute `fn` puis marque l'élément à reconstruire |

### Les règles à ne jamais oublier

| Règle | Formulation courte |
| --- | --- |
| Immuabilité | toutes les propriétés d'un widget sont `final` |
| Emplacement de l'état | les variables d'état vont dans le `State`, jamais dans le widget |
| Signalement | toute modification d'état passe par `setState()` |
| Pureté | `build()` décrit, `build()` n'agit pas |
| Libération | tout ce qui est créé dans `initState()` est libéré dans `dispose()` |
| Sécurité asynchrone | `if (!mounted) return;` après chaque `await` |
| Source unique | une information n'existe qu'à un seul endroit |
| Communication | la donnée descend par le constructeur, l'action remonte par un callback |
| Placement | l'état vit chez le plus proche ancêtre commun, ni plus haut ni plus bas |
| Sobriété | on commence `StatelessWidget`, on convertit seulement si nécessaire |

---

## 45.34 — Exercices

Pour chaque exercice, partez d'un projet Flutter fonctionnel (chapitre 43) et
remplacez entièrement le contenu de `lib/main.dart`.

### Exercice 1 — La carte d'identité du héros (facile)

Écrivez un `StatelessWidget` nommé `CarteHeros` qui reçoit trois propriétés :
`nom` (`String`), `niveau` (`int`) et `classe` (`String`, valeur par défaut
`'Aventurier'`).

Il affiche une `Card` contenant un `ListTile` avec :

- en `leading` un `CircleAvatar` contenant la première lettre du nom ;
- en `title` le nom suivi du niveau, sous la forme `Alex (niveau 7)` ;
- en `subtitle` la classe.

Affichez trois héros différents dans un `Scaffold`, dont un sans préciser la
classe.

---

### Exercice 2 — Le compteur d'ennemis vaincus (facile)

Écrivez un `StatefulWidget` qui affiche un nombre d'ennemis vaincus, au départ 0,
et deux boutons :

- « Ennemi vaincu » qui ajoute 1 ;
- « Réinitialiser » qui remet à 0.

Le bouton « Réinitialiser » doit être **désactivé** quand le compteur vaut déjà 0.

---

### Exercice 3 — La barre d'énergie bornée (facile)

Écrivez un écran avec une énergie comprise entre 0 et 100, valeur initiale 50.

- un bouton « Courir » retire 10 ;
- un bouton « Se reposer » ajoute 20 ;
- l'énergie ne doit jamais sortir de l'intervalle [0, 100] ;
- affichez une barre textuelle de 10 caractères : un `#` par tranche de 10 points,
  un `.` sinon ;
- affichez `ÉPUISÉ` en rouge quand l'énergie vaut 0.

Utilisez la méthode `clamp` de Dart (chapitre 03) pour borner la valeur.

---

### Exercice 4 — Le mode nuit local (facile)

Écrivez un écran avec un `Switch` qui bascule un booléen `_modeNuit`.

Quand `_modeNuit` est vrai, le fond du `Scaffold` devient noir et le texte blanc ;
sinon l'inverse. Affichez aussi le libellé « Mode nuit activé » ou
« Mode jour activé ».

Le widget `Switch` s'utilise ainsi : `Switch(value: ..., onChanged: (bool v) { ... })`.

---

### Exercice 5 — Trouver et corriger les trois bugs (moyen)

Le code suivant contient trois erreurs vues dans ce chapitre. Recopiez-le,
identifiez chaque erreur, expliquez-la, puis corrigez-la.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApplication());

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: EcranScore());
  }
}

class EcranScore extends StatefulWidget {
  EcranScore({super.key});

  int score = 0;

  @override
  State<EcranScore> createState() => _EcranScoreState();
}

class _EcranScoreState extends State<EcranScore> {
  void _ajouter() {
    widget.score += 10;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Score : ${widget.score}'),
            ElevatedButton(
              onPressed: _ajouter(),
              child: const Text('+10'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### Exercice 6 — Le chronomètre avec pause (moyen)

Écrivez un chronomètre qui compte les secondes **écoulées** depuis 0.

- il démarre automatiquement à l'ouverture de l'écran ;
- un bouton bascule entre « Pause » et « Reprendre » ;
- un bouton « Remettre à zéro » remet le compteur à 0 sans arrêter le décompte ;
- le `Timer` doit être correctement annulé dans `dispose()`.

Affichez le temps au format `mm:ss` (utilisez `toString().padLeft(2, '0')`).

---

### Exercice 7 — L'inventaire avec ajout et suppression (moyen)

Écrivez un écran contenant une `List<String>` d'objets, initialement vide.

- un `TextField` avec un `TextEditingController` permet de saisir un nom d'objet ;
- un bouton « Ajouter » ajoute l'objet à la liste et vide le champ ;
- l'ajout est refusé si le champ est vide ou si l'objet est déjà présent ;
- chaque ligne de la liste possède une icône de suppression ;
- le nombre d'objets est affiché dans l'`AppBar` ;
- le contrôleur est libéré dans `dispose()`.

Utilisez une simple `Column` avec un `for` : la `ListView` sera vue au chapitre 48.

---

### Exercice 8 — La remontée d'état (difficile)

Écrivez un écran de combat contenant :

- un `StatefulWidget` `EcranCombat` qui détient les vies du joueur (100) et celles
  du boss (200) ;
- un `StatelessWidget` `BarreDeVie` qui reçoit un `nom`, des `vies`, des `viesMax`
  et une `couleur`, et affiche `nom : vies / viesMax` plus une barre textuelle de
  20 caractères ;
- un `StatelessWidget` `PanneauActions` qui reçoit deux callbacks
  (`onAttaquer`, `onSoigner`) et affiche deux boutons ;
- « Attaquer » retire 25 vies au boss, puis le boss riposte et retire 10 vies au
  joueur ;
- « Se soigner » rend 15 vies au joueur sans dépasser 100, mais le boss en profite
  pour attaquer et retire 10 vies au joueur ;
- toutes les vies restent bornées à l'intervalle [0, maximum] ;
- quand l'un des deux atteint 0, les boutons sont désactivés et un message de
  victoire ou de défaite s'affiche.

Aucune variable d'état ne doit se trouver ailleurs que dans `_EcranCombatState`.

---

### Exercice 9 — Le widget qui réagit à son parent (difficile)

Écrivez :

- un `StatefulWidget` parent `EcranDifficulte` qui détient une difficulté
  (`'Facile'`, `'Normal'`, `'Difficile'`) et un bouton pour en changer ;
- un `StatefulWidget` enfant `GenerateurDeDegats` qui reçoit la difficulté et
  possède son propre état `_totalDegats`, initialement 0 ;
- l'enfant possède un bouton « Frapper » qui ajoute 10, 20 ou 40 dégâts selon la
  difficulté reçue ;
- **quand la difficulté change, l'enfant doit remettre `_totalDegats` à 0** et
  journaliser le changement dans la console.

Utilisez `didUpdateWidget()` avec le test de comparaison obligatoire.

---

### Exercice 10 — Mini-projet : la fiche de personnage éditable (projet)

Réalisez une application d'une seule page permettant de créer et modifier la fiche
d'un personnage de jeu.

**Contenu de la fiche :**

| Donnée | Type | Contrainte |
| --- | --- | --- |
| nom | `String` | non vide, sinon `'Sans nom'` |
| classe | `String` | choisie parmi `Guerrier`, `Mage`, `Voleur`, `Archer` |
| niveau | `int` | de 1 à 50 |
| points de vie | `int` | de 0 à 100 |
| points d'attaque | `int` | de 0 à 50 |
| favori | `bool` | oui ou non |

**Comportement attendu :**

1. En haut, une **carte de présentation** en lecture seule qui montre l'état
   courant : nom, classe, niveau, une barre de vie textuelle, l'attaque, et une
   étoile si le personnage est favori.
2. En dessous, une **zone d'édition** :
   - un `TextField` pour le nom, avec un bouton « Renommer » ;
   - quatre boutons pour choisir la classe, celui de la classe active étant
     visuellement distinct ;
   - deux boutons `-` et `+` pour le niveau (bornés à 1 et 50) ;
   - deux boutons `-10` et `+10` pour les points de vie (bornés à 0 et 100) ;
   - deux boutons `-5` et `+5` pour l'attaque (bornés à 0 et 50) ;
   - un `Switch` pour le favori.
3. Un bouton « Réinitialiser la fiche » qui restaure toutes les valeurs par défaut
   (`'Alex'`, `Guerrier`, niveau 1, 100 vies, 10 d'attaque, non favori).
4. Un compteur « modifications : N » qui s'incrémente à chaque changement réel.

**Contraintes techniques :**

- exactement **un** `StatefulWidget` détient l'état : `EcranFichePersonnage` ;
- la carte de présentation et le sélecteur de classe sont des `StatelessWidget`
  qui reçoivent des données et remontent leurs actions par callbacks ;
- le `TextEditingController` est créé dans `initState()` et libéré dans
  `dispose()` ;
- aucun changement inutile ne doit incrémenter le compteur (gardes en début de
  méthode) ;
- toutes les bornes sont respectées par `clamp`.

---

## 45.35 — Corrections des exercices

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Héros',
      theme: ThemeData(useMaterial3: true),
      home: const EcranHeros(),
    );
  }
}

class EcranHeros extends StatelessWidget {
  const EcranHeros({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Nos héros')),
      body: const Padding(
        padding: EdgeInsets.all(12),
        child: Column(
          children: [
            CarteHeros(nom: 'Alex', niveau: 7, classe: 'Guerrier'),
            CarteHeros(nom: 'Sophie', niveau: 12, classe: 'Mage'),
            CarteHeros(nom: 'Samir', niveau: 3),
          ],
        ),
      ),
    );
  }
}

class CarteHeros extends StatelessWidget {
  const CarteHeros({
    super.key,
    required this.nom,
    required this.niveau,
    this.classe = 'Aventurier',
  });

  final String nom;
  final int niveau;
  final String classe;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(child: Text(nom[0])),
        title: Text('$nom (niveau $niveau)'),
        subtitle: Text(classe),
      ),
    );
  }
}
```

**Résultat :**

```text
┌──────────────────────────────────────────┐
│ Nos héros                                │
├──────────────────────────────────────────┤
│ ( A )  Alex (niveau 7)                   │
│        Guerrier                          │
│ ( S )  Sophie (niveau 12)                │
│        Mage                              │
│ ( S )  Samir (niveau 3)                  │
│        Aventurier                        │
└──────────────────────────────────────────┘
```

**Explication :** `CarteHeros` n'a aucun état : toutes ses informations viennent du
constructeur, et rien ne peut les modifier de l'intérieur. C'est donc un
`StatelessWidget`. Les trois champs sont `final`, ce qui permet le constructeur
`const` et donc le `const Column(...)` du parent : l'intégralité de ce sous-arbre
n'est construite qu'une seule fois. Le paramètre `classe` est déclaré avec la
valeur par défaut `'Aventurier'` : Samir, appelé sans ce paramètre, l'obtient
automatiquement. `nom[0]` extrait le premier caractère de la chaîne (chapitre 02)
et sert d'avatar, ce qui évite tout fichier image.

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranEnnemis(),
    );
  }
}

class EcranEnnemis extends StatefulWidget {
  const EcranEnnemis({super.key});

  @override
  State<EcranEnnemis> createState() => _EcranEnnemisState();
}

class _EcranEnnemisState extends State<EcranEnnemis> {
  int _vaincus = 0;

  void _vaincreUnEnnemi() {
    setState(() {
      _vaincus++;
    });
  }

  void _reinitialiser() {
    if (_vaincus == 0) return;
    setState(() {
      _vaincus = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tableau de chasse')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Ennemis vaincus'),
            Text(
              '$_vaincus',
              style: const TextStyle(fontSize: 64, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _vaincreUnEnnemi,
              child: const Text('Ennemi vaincu'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _vaincus == 0 ? null : _reinitialiser,
              child: const Text('Réinitialiser'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après quatre appuis :**

```text
┌──────────────────────────────────┐
│ Tableau de chasse                │
├──────────────────────────────────┤
│        Ennemis vaincus           │
│               4                  │
│                                  │
│      [ Ennemi vaincu ]           │
│      [ Réinitialiser ]           │
└──────────────────────────────────┘
```

**Explication :** l'écran doit se souvenir d'un nombre entre deux reconstructions
et le modifier lui-même : c'est donc un `StatefulWidget`. La variable `_vaincus`
est déclarée dans `_EcranEnnemisState`, avec le préfixe `_` puisqu'elle est privée.
Chaque modification passe par `setState()`. La désactivation du second bouton
s'obtient en passant `null` à `onPressed` : Flutter grise alors le bouton
automatiquement. La garde `if (_vaincus == 0) return;` dans `_reinitialiser()`
évite un `setState()` inutile, conformément à la section 45.15 ; ici elle est
redondante avec la désactivation du bouton, mais c'est une bonne habitude à prendre
car la méthode pourrait être appelée depuis ailleurs plus tard.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
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
  static const int energieMin = 0;
  static const int energieMax = 100;

  int _energie = 50;

  void _modifier(int variation) {
    final int nouvelle = (_energie + variation).clamp(energieMin, energieMax);
    if (nouvelle == _energie) return;
    setState(() {
      _energie = nouvelle;
    });
  }

  @override
  Widget build(BuildContext context) {
    final int segmentsPleins = _energie ~/ 10;
    final String barre = '#' * segmentsPleins + '.' * (10 - segmentsPleins);
    final bool epuise = _energie == energieMin;

    return Scaffold(
      appBar: AppBar(title: const Text('Énergie du joueur')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('$_energie / $energieMax', style: const TextStyle(fontSize: 28)),
            const SizedBox(height: 8),
            Text(
              barre,
              style: const TextStyle(fontSize: 26, letterSpacing: 4),
            ),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _energie > energieMin ? () => _modifier(-10) : null,
                  child: const Text('Courir (-10)'),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: _energie < energieMax ? () => _modifier(20) : null,
                  child: const Text('Se reposer (+20)'),
                ),
              ],
            ),
            if (epuise)
              const Padding(
                padding: EdgeInsets.only(top: 24),
                child: Text(
                  'ÉPUISÉ',
                  style: TextStyle(
                    fontSize: 32,
                    color: Colors.red,
                    fontWeight: FontWeight.bold,
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

**Résultat à 30 points d'énergie :**

```text
┌──────────────────────────────────────┐
│ Énergie du joueur                    │
├──────────────────────────────────────┤
│             30 / 100                 │
│        # # # . . . . . . .           │
│                                      │
│  [ Courir (-10) ] [ Se reposer ]     │
└──────────────────────────────────────┘
```

**Résultat à 0 :**

```text
│              0 / 100                 │
│        . . . . . . . . . .           │
│  [ Courir (-10) ] [ Se reposer ]     │
│      (grisé)                         │
│              ÉPUISÉ                  │
```

**Explication :** une seule méthode `_modifier(int variation)` gère les deux
boutons, ce qui évite de dupliquer la logique de bornage. `clamp(0, 100)` ramène
la valeur dans l'intervalle : appelé sur un `int` avec deux `int`, il retourne un
`int`, ce qui permet l'affectation directe. La garde
`if (nouvelle == _energie) return;` empêche un `setState()` inutile lorsque la
valeur est déjà à une borne. `barre` et `epuise` sont des variables **locales** à
`build()` : elles sont dérivées de `_energie` et n'ont donc rien à faire dans un
champ du `State`. Enfin, `if (epuise) ...` est un *collection if* : quand la
condition est fausse, aucun widget n'est inséré dans la `Column`.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranModeNuit(),
    );
  }
}

class EcranModeNuit extends StatefulWidget {
  const EcranModeNuit({super.key});

  @override
  State<EcranModeNuit> createState() => _EcranModeNuitState();
}

class _EcranModeNuitState extends State<EcranModeNuit> {
  bool _modeNuit = false;

  void _changerMode(bool actif) {
    if (actif == _modeNuit) return;
    setState(() {
      _modeNuit = actif;
    });
  }

  @override
  Widget build(BuildContext context) {
    final Color fond = _modeNuit ? Colors.black : Colors.white;
    final Color encre = _modeNuit ? Colors.white : Colors.black;

    return Scaffold(
      backgroundColor: fond,
      appBar: AppBar(title: const Text('Réglages d\'affichage')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              _modeNuit ? Icons.nightlight_round : Icons.wb_sunny,
              size: 80,
              color: encre,
            ),
            const SizedBox(height: 16),
            Text(
              _modeNuit ? 'Mode nuit activé' : 'Mode jour activé',
              style: TextStyle(fontSize: 24, color: encre),
            ),
            const SizedBox(height: 24),
            Switch(
              value: _modeNuit,
              onChanged: _changerMode,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat, interrupteur activé :**

```text
┌──────────────────────────────────┐
│ Réglages d'affichage             │
├──────────────────────────────────┤
│           (lune)                 │   fond noir
│      Mode nuit activé            │   texte blanc
│           [==O]                  │
└──────────────────────────────────┘
```

**Explication :** `_modeNuit` est un état booléen local à l'écran, donc un
`StatefulWidget` suffit. Le paramètre `onChanged` de `Switch` est de type
`ValueChanged<bool>`, c'est-à-dire `void Function(bool)` : la méthode
`_changerMode(bool actif)` a exactement cette signature, on peut donc la passer
directement, sans fonction anonyme et sans parenthèses. Les couleurs `fond` et
`encre` sont calculées dans `build()` à partir de l'état : c'est le principe
déclaratif de la section 45.1.1, on décrit l'écran **pour l'état courant** au lieu
de modifier des couleurs une par une. Notez que ce mode nuit n'agit que sur cet
écran ; le vrai mode sombre, appliqué à toute l'application via `ThemeData`, sera
vu au chapitre 51.

---

### Correction 5

Les trois erreurs sont les suivantes.

**Erreur 1 — la variable d'état est déclarée dans le widget.**

```dart
class EcranScore extends StatefulWidget {
  int score = 0;    // MAUVAIS ENDROIT
```

Un `StatefulWidget` est immuable et jetable (45.6 et 45.10). L'analyseur signale
`must_be_immutable`, et la valeur serait perdue à chaque reconstruction du parent.
La variable doit aller dans `_EcranScoreState`.

**Erreur 2 — l'état est modifié sans `setState()`.**

```dart
  void _ajouter() {
    widget.score += 10;    // aucune reconstruction déclenchée
  }
```

C'est le bug silencieux de la section 45.14 : la valeur changerait sans que
l'écran soit prévenu.

**Erreur 3 — la méthode est appelée au lieu d'être passée.**

```dart
  onPressed: _ajouter(),   // exécutée pendant build()
```

Les parenthèses provoquent l'appel immédiat pendant `build()` (45.29.3). Une fois
le `setState()` remis en place, cela produirait l'exception
`setState() or markNeedsBuild() called during build`.

Voici le code corrigé.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranScore(),
    );
  }
}

class EcranScore extends StatefulWidget {
  const EcranScore({super.key});     // correction 1a : constructeur const

  @override
  State<EcranScore> createState() => _EcranScoreState();
}

class _EcranScoreState extends State<EcranScore> {
  int _score = 0;                    // correction 1b : l'état est dans le State

  void _ajouter() {
    setState(() {                    // correction 2 : setState
      _score += 10;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Score')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Score : $_score', style: const TextStyle(fontSize: 28)),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: _ajouter,   // correction 3 : pas de parenthèses
              child: const Text('+10'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après trois appuis :**

```text
┌──────────────────────────────────┐
│ Score                            │
├──────────────────────────────────┤
│         Score : 30               │
│           [ +10 ]                │
└──────────────────────────────────┘
```

**Explication :** les trois erreurs sont les trois plus fréquentes du chapitre, et
elles se renforcent : la première rend l'état fragile, la deuxième empêche tout
affichage, la troisième déclenche une exception dès qu'on corrige la deuxième.
Retenez la chaîne de vérification : **l'état est-il dans le `State` ? est-il
modifié dans un `setState()` ? le callback est-il passé sans parenthèses ?**

---

### Correction 6

```dart
import 'dart:async';

import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranChronometre(),
    );
  }
}

class EcranChronometre extends StatefulWidget {
  const EcranChronometre({super.key});

  @override
  State<EcranChronometre> createState() => _EcranChronometreState();
}

class _EcranChronometreState extends State<EcranChronometre> {
  Timer? _timer;
  int _secondes = 0;
  bool _enPause = false;

  @override
  void initState() {
    super.initState();
    _demarrerLeTimer();
  }

  void _demarrerLeTimer() {
    _timer?.cancel();
    _timer = Timer.periodic(const Duration(seconds: 1), (_) {
      setState(() {
        _secondes++;
      });
    });
  }

  void _basculerLaPause() {
    setState(() {
      _enPause = !_enPause;
    });
    if (_enPause) {
      _timer?.cancel();
    } else {
      _demarrerLeTimer();
    }
  }

  void _remettreAZero() {
    if (_secondes == 0) return;
    setState(() {
      _secondes = 0;
    });
  }

  String _formater(int total) {
    final String minutes = (total ~/ 60).toString().padLeft(2, '0');
    final String secondes = (total % 60).toString().padLeft(2, '0');
    return '$minutes:$secondes';
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chronomètre de partie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              _formater(_secondes),
              style: const TextStyle(fontSize: 64, fontWeight: FontWeight.bold),
            ),
            Text(_enPause ? 'En pause' : 'En cours'),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _basculerLaPause,
              child: Text(_enPause ? 'Reprendre' : 'Pause'),
            ),
            const SizedBox(height: 8),
            OutlinedButton(
              onPressed: _secondes == 0 ? null : _remettreAZero,
              child: const Text('Remettre à zéro'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Résultat après 1 minute 25 secondes :**

```text
┌──────────────────────────────────┐
│ Chronomètre de partie            │
├──────────────────────────────────┤
│            01:25                 │
│           En cours               │
│                                  │
│          [ Pause ]               │
│      [ Remettre à zéro ]         │
└──────────────────────────────────┘
```

**Explication :** le `Timer` est une ressource : il naît dans `initState()` et il
meurt dans `dispose()`. Sans le `_timer?.cancel()` de `dispose()`, il continuerait
à tourner après la fermeture de l'écran et appellerait `setState()` sur un `State`
démonté, ce qui lèverait l'exception vue en 45.26. Le `_timer?.cancel()` placé au
début de `_demarrerLeTimer()` garantit qu'il n'existe jamais deux timers en
parallèle, même si l'utilisateur enchaîne les pauses. Le champ est déclaré `Timer?`
(nullable, chapitre 12) parce qu'il n'existe pas encore au moment de la déclaration
et qu'il peut être annulé ; on l'utilise ensuite avec `?.`. Enfin, `_formater()`
est une méthode utilitaire pure : elle ne touche à aucun état, elle transforme un
entier en chaîne, et `padLeft(2, '0')` garantit deux chiffres.

---

### Correction 7

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranInventaire(),
    );
  }
}

class EcranInventaire extends StatefulWidget {
  const EcranInventaire({super.key});

  @override
  State<EcranInventaire> createState() => _EcranInventaireState();
}

class _EcranInventaireState extends State<EcranInventaire> {
  late final TextEditingController _controleur;
  final List<String> _objets = <String>[];
  String? _messageErreur;

  @override
  void initState() {
    super.initState();
    _controleur = TextEditingController();
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  void _ajouter() {
    final String saisie = _controleur.text.trim();

    if (saisie.isEmpty) {
      setState(() {
        _messageErreur = 'Le nom de l\'objet ne peut pas être vide.';
      });
      return;
    }

    if (_objets.contains(saisie)) {
      setState(() {
        _messageErreur = '« $saisie » est déjà dans le sac.';
      });
      return;
    }

    setState(() {
      _objets.add(saisie);
      _messageErreur = null;
    });
    _controleur.clear();
  }

  void _supprimer(String objet) {
    setState(() {
      _objets.remove(objet);
      _messageErreur = null;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Inventaire (${_objets.length})')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            TextField(
              controller: _controleur,
              decoration: const InputDecoration(
                labelText: 'Nom de l\'objet',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (_) => _ajouter(),
            ),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: _ajouter,
              child: const Text('Ajouter au sac'),
            ),
            if (_messageErreur != null)
              Padding(
                padding: const EdgeInsets.only(top: 8),
                child: Text(
                  _messageErreur!,
                  style: const TextStyle(color: Colors.red),
                ),
              ),
            const SizedBox(height: 16),
            if (_objets.isEmpty)
              const Text('Le sac est vide.')
            else
              for (final String objet in _objets)
                Card(
                  child: ListTile(
                    leading: const Icon(Icons.inventory_2),
                    title: Text(objet),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete),
                      onPressed: () => _supprimer(objet),
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

**Résultat après avoir ajouté « Potion » et « Épée » :**

```text
┌────────────────────────────────────────┐
│ Inventaire (2)                         │
├────────────────────────────────────────┤
│ ┌────────────────────────────────────┐ │
│ │ Nom de l'objet                     │ │
│ └────────────────────────────────────┘ │
│ [        Ajouter au sac            ]   │
│                                        │
│ ┌────────────────────────────────────┐ │
│ │ [B] Potion                    [X]  │ │
│ ├────────────────────────────────────┤ │
│ │ [B] Épée                      [X]  │ │
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

**Résultat si l'on tente d'ajouter « Potion » une seconde fois :**

```text
│ « Potion » est déjà dans le sac.       │
```

**Explication :** le `TextEditingController` est créé dans `initState()` et libéré
dans `dispose()` : c'est le schéma obligatoire de la section 45.25. Il est déclaré
`late final` car il ne peut pas être construit sur la ligne de déclaration si l'on
veut pouvoir y injecter plus tard une valeur venue du widget. Attention au point le
plus subtil de cet exercice : `_objets.add(saisie)` modifie la liste **en place**,
sans affectation. Beaucoup pensent qu'un `setState()` n'est nécessaire que devant
un `=` ; c'est faux, toute modification d'état, y compris `add`, `remove` ou
`clear`, doit être encadrée (45.14.3). `_messageErreur` est un `String?` : `null`
signifie « pas d'erreur », et le *collection if* n'affiche la ligne rouge que
lorsqu'il y a réellement quelque chose à dire. Enfin, `_controleur.clear()` est
placé **hors** du `setState()` : le contrôleur notifie lui-même le `TextField`, il
n'a pas besoin de notre signal.

---

### Correction 8

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranCombat(),
    );
  }
}

/// Unique détenteur de l'état : c'est ici que vit toute la vérité du combat.
class EcranCombat extends StatefulWidget {
  const EcranCombat({super.key});

  @override
  State<EcranCombat> createState() => _EcranCombatState();
}

class _EcranCombatState extends State<EcranCombat> {
  static const int viesMaxJoueur = 100;
  static const int viesMaxBoss = 200;

  int _viesJoueur = viesMaxJoueur;
  int _viesBoss = viesMaxBoss;

  bool get _combatTermine => _viesJoueur == 0 || _viesBoss == 0;
  bool get _joueurGagne => _viesBoss == 0 && _viesJoueur > 0;

  void _attaquer() {
    if (_combatTermine) return;
    setState(() {
      _viesBoss = (_viesBoss - 25).clamp(0, viesMaxBoss);
      if (_viesBoss > 0) {
        _viesJoueur = (_viesJoueur - 10).clamp(0, viesMaxJoueur);
      }
    });
  }

  void _seSoigner() {
    if (_combatTermine) return;
    setState(() {
      _viesJoueur = (_viesJoueur + 15).clamp(0, viesMaxJoueur);
      _viesJoueur = (_viesJoueur - 10).clamp(0, viesMaxJoueur);
    });
  }

  void _recommencer() {
    setState(() {
      _viesJoueur = viesMaxJoueur;
      _viesBoss = viesMaxBoss;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Combat contre le boss')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            BarreDeVie(
              nom: 'Alex',
              vies: _viesJoueur,
              viesMax: viesMaxJoueur,
              couleur: Colors.green,
            ),
            const SizedBox(height: 16),
            BarreDeVie(
              nom: 'Boss',
              vies: _viesBoss,
              viesMax: viesMaxBoss,
              couleur: Colors.red,
            ),
            const SizedBox(height: 32),
            PanneauActions(
              actif: !_combatTermine,
              onAttaquer: _attaquer,
              onSoigner: _seSoigner,
            ),
            const SizedBox(height: 24),
            if (_combatTermine)
              Column(
                children: [
                  Text(
                    _joueurGagne ? 'VICTOIRE' : 'DÉFAITE',
                    style: TextStyle(
                      fontSize: 36,
                      fontWeight: FontWeight.bold,
                      color: _joueurGagne ? Colors.green : Colors.red,
                    ),
                  ),
                  const SizedBox(height: 8),
                  OutlinedButton(
                    onPressed: _recommencer,
                    child: const Text('Nouveau combat'),
                  ),
                ],
              ),
          ],
        ),
      ),
    );
  }
}

/// Widget purement descriptif : il reçoit, il affiche, il ne décide de rien.
class BarreDeVie extends StatelessWidget {
  const BarreDeVie({
    super.key,
    required this.nom,
    required this.vies,
    required this.viesMax,
    required this.couleur,
  });

  final String nom;
  final int vies;
  final int viesMax;
  final Color couleur;

  @override
  Widget build(BuildContext context) {
    const int longueur = 20;
    final int pleins = (vies * longueur / viesMax).round();
    final String barre = '#' * pleins + '.' * (longueur - pleins);

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('$nom : $vies / $viesMax',
            style: const TextStyle(fontWeight: FontWeight.bold)),
        Text(barre, style: TextStyle(color: couleur, fontSize: 18)),
      ],
    );
  }
}

/// Widget purement descriptif : il remonte les actions par callbacks.
class PanneauActions extends StatelessWidget {
  const PanneauActions({
    super.key,
    required this.actif,
    required this.onAttaquer,
    required this.onSoigner,
  });

  final bool actif;
  final VoidCallback onAttaquer;
  final VoidCallback onSoigner;

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        ElevatedButton(
          onPressed: actif ? onAttaquer : null,
          child: const Text('Attaquer'),
        ),
        const SizedBox(width: 16),
        ElevatedButton(
          onPressed: actif ? onSoigner : null,
          child: const Text('Se soigner'),
        ),
      ],
    );
  }
}
```

**Résultat après quatre attaques :**

```text
┌──────────────────────────────────────────────┐
│ Combat contre le boss                        │
├──────────────────────────────────────────────┤
│ Alex : 60 / 100                              │
│ ############........                         │
│                                              │
│ Boss : 100 / 200                             │
│ ##########..........                         │
│                                              │
│     [ Attaquer ]  [ Se soigner ]             │
└──────────────────────────────────────────────┘
```

**Résultat en fin de combat gagné :**

```text
│ Boss : 0 / 200                               │
│ ....................                         │
│     [ Attaquer ]  [ Se soigner ]  (grisés)   │
│                  VICTOIRE                    │
│            [ Nouveau combat ]                │
```

**Explication :** c'est l'application directe de la remontée d'état (45.28). Trois
widgets ont besoin de connaître les vies : les deux `BarreDeVie` et le
`PanneauActions` (pour savoir s'il doit se désactiver). Leur plus proche ancêtre
commun est `EcranCombat`, donc l'état y est placé, et **nulle part ailleurs**. Il
n'y a ainsi qu'une seule source de vérité : les deux barres ne peuvent pas
diverger. `BarreDeVie` et `PanneauActions` sont des `StatelessWidget` : le premier
reçoit des données descendantes, le second reçoit deux `VoidCallback` qui font
remonter l'action jusqu'au `State` du parent (45.29). Les getters `_combatTermine`
et `_joueurGagne` sont **dérivés** de l'état, jamais stockés : c'est la garantie
qu'ils ne peuvent pas devenir incohérents. Enfin, chaque calcul de vies est encadré
par `clamp(0, max)`, ce qui rend impossible une valeur négative, quelle que soit la
suite d'actions de l'utilisateur.

---

### Correction 9

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true),
      home: const EcranDifficulte(),
    );
  }
}

class EcranDifficulte extends StatefulWidget {
  const EcranDifficulte({super.key});

  @override
  State<EcranDifficulte> createState() => _EcranDifficulteState();
}

class _EcranDifficulteState extends State<EcranDifficulte> {
  static const List<String> difficultes = ['Facile', 'Normal', 'Difficile'];

  int _index = 0;

  String get _difficulte => difficultes[_index];

  void _difficulteSuivante() {
    setState(() {
      _index = (_index + 1) % difficultes.length;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Entraînement')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Difficulté : $_difficulte',
                style: const TextStyle(fontSize: 24)),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: _difficulteSuivante,
              child: const Text('Changer la difficulté'),
            ),
            const SizedBox(height: 32),
            const Divider(),
            const SizedBox(height: 16),
            GenerateurDeDegats(difficulte: _difficulte),
          ],
        ),
      ),
    );
  }
}

class GenerateurDeDegats extends StatefulWidget {
  const GenerateurDeDegats({super.key, required this.difficulte});

  final String difficulte;

  @override
  State<GenerateurDeDegats> createState() => _GenerateurDeDegatsState();
}

class _GenerateurDeDegatsState extends State<GenerateurDeDegats> {
  int _totalDegats = 0;

  int get _degatsParCoup {
    switch (widget.difficulte) {
      case 'Facile':
        return 10;
      case 'Normal':
        return 20;
      case 'Difficile':
        return 40;
      default:
        return 10;
    }
  }

  void _frapper() {
    setState(() {
      _totalDegats += _degatsParCoup;
    });
  }

  @override
  void didUpdateWidget(covariant GenerateurDeDegats oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.difficulte != widget.difficulte) {
      debugPrint(
        'difficulté ${oldWidget.difficulte} -> ${widget.difficulte} : '
        'compteur remis à zéro',
      );
      setState(() {
        _totalDegats = 0;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Dégâts par coup : $_degatsParCoup'),
        Text('Total infligé : $_totalDegats',
            style: const TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        ElevatedButton(
          onPressed: _frapper,
          child: const Text('Frapper'),
        ),
      ],
    );
  }
}
```

**Résultat dans la console après trois coups en « Facile » puis un changement :**

```text
difficulté Facile -> Normal : compteur remis à zéro
```

**Résultat à l'écran juste après ce changement :**

```text
┌──────────────────────────────────┐
│ Entraînement                     │
├──────────────────────────────────┤
│      Difficulté : Normal         │
│   [ Changer la difficulté ]      │
│  ──────────────────────────────  │
│      Dégâts par coup : 20        │
│         Total infligé : 0        │
│           [ Frapper ]            │
└──────────────────────────────────┘
```

**Explication :** l'enfant possède un état propre (`_totalDegats`) tout en
recevant une configuration du parent (`difficulte`). Quand le parent appelle
`setState()`, il recrée un objet `GenerateurDeDegats`, mais le `State` de l'enfant
est **conservé** : `initState()` n'est donc pas rappelé, et sans autre mécanisme le
compteur ne serait jamais remis à zéro. `didUpdateWidget()` est exactement le point
d'accroche prévu pour cela (45.22). Le test `oldWidget.difficulte != widget.difficulte`
est obligatoire : `didUpdateWidget()` est appelée à **chaque** reconstruction du
parent, y compris quand la difficulté n'a pas bougé, et sans ce test le compteur
serait effacé en permanence. Notez que `setState()` **est** autorisé dans
`didUpdateWidget()`, contrairement à `initState()` et à `build()`. Enfin,
`_degatsParCoup` est un getter calculé à partir de `widget.difficulte` : il lit la
configuration courante à chaque appel, sans jamais la recopier dans un champ
(règle 2 de la section 45.16.1).

---

### Correction 10 — Mini-projet : la fiche de personnage éditable

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MonApplication());
}

class MonApplication extends StatelessWidget {
  const MonApplication({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fiche de personnage',
      theme: ThemeData(useMaterial3: true),
      home: const EcranFichePersonnage(),
    );
  }
}

// ---------------------------------------------------------------------------
// L'UNIQUE détenteur de l'état de l'application.
// ---------------------------------------------------------------------------

class EcranFichePersonnage extends StatefulWidget {
  const EcranFichePersonnage({super.key});

  @override
  State<EcranFichePersonnage> createState() => _EcranFichePersonnageState();
}

class _EcranFichePersonnageState extends State<EcranFichePersonnage> {
  // Bornes et valeurs par défaut, regroupées pour être modifiables en un point.
  static const List<String> classesDisponibles = [
    'Guerrier',
    'Mage',
    'Voleur',
    'Archer',
  ];
  static const String nomParDefaut = 'Alex';
  static const String classeParDefaut = 'Guerrier';
  static const int niveauMin = 1;
  static const int niveauMax = 50;
  static const int viesMin = 0;
  static const int viesMax = 100;
  static const int attaqueMin = 0;
  static const int attaqueMax = 50;

  // L'état.
  String _nom = nomParDefaut;
  String _classe = classeParDefaut;
  int _niveau = 1;
  int _vies = 100;
  int _attaque = 10;
  bool _favori = false;
  int _modifications = 0;

  late final TextEditingController _controleurNom;

  @override
  void initState() {
    super.initState();
    _controleurNom = TextEditingController(text: _nom);
  }

  @override
  void dispose() {
    _controleurNom.dispose();
    super.dispose();
  }

  // ----- Les actions -------------------------------------------------------

  void _renommer() {
    final String saisie = _controleurNom.text.trim();
    final String nouveauNom = saisie.isEmpty ? 'Sans nom' : saisie;
    if (nouveauNom == _nom) return;
    setState(() {
      _nom = nouveauNom;
      _modifications++;
    });
  }

  void _choisirLaClasse(String classe) {
    if (classe == _classe) return;
    setState(() {
      _classe = classe;
      _modifications++;
    });
  }

  void _modifierLeNiveau(int variation) {
    final int nouveau = (_niveau + variation).clamp(niveauMin, niveauMax);
    if (nouveau == _niveau) return;
    setState(() {
      _niveau = nouveau;
      _modifications++;
    });
  }

  void _modifierLesVies(int variation) {
    final int nouveau = (_vies + variation).clamp(viesMin, viesMax);
    if (nouveau == _vies) return;
    setState(() {
      _vies = nouveau;
      _modifications++;
    });
  }

  void _modifierLAttaque(int variation) {
    final int nouveau = (_attaque + variation).clamp(attaqueMin, attaqueMax);
    if (nouveau == _attaque) return;
    setState(() {
      _attaque = nouveau;
      _modifications++;
    });
  }

  void _changerFavori(bool valeur) {
    if (valeur == _favori) return;
    setState(() {
      _favori = valeur;
      _modifications++;
    });
  }

  void _reinitialiser() {
    setState(() {
      _nom = nomParDefaut;
      _classe = classeParDefaut;
      _niveau = 1;
      _vies = 100;
      _attaque = 10;
      _favori = false;
      _modifications = 0;
    });
    _controleurNom.text = nomParDefaut;
  }

  // ----- La description de l'écran ----------------------------------------

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Fiche de personnage'),
        actions: [
          Padding(
            padding: const EdgeInsets.only(right: 16),
            child: Center(child: Text('modifications : $_modifications')),
          ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // --- La carte en lecture seule ---
          CartePersonnage(
            nom: _nom,
            classe: _classe,
            niveau: _niveau,
            vies: _vies,
            viesMax: viesMax,
            attaque: _attaque,
            favori: _favori,
          ),
          const SizedBox(height: 24),
          const Text('ÉDITION', style: TextStyle(fontWeight: FontWeight.bold)),
          const Divider(),

          // --- Le nom ---
          TextField(
            controller: _controleurNom,
            decoration: const InputDecoration(
              labelText: 'Nom du personnage',
              border: OutlineInputBorder(),
            ),
            onSubmitted: (_) => _renommer(),
          ),
          const SizedBox(height: 8),
          ElevatedButton(
            onPressed: _renommer,
            child: const Text('Renommer'),
          ),
          const SizedBox(height: 16),

          // --- La classe ---
          SelecteurDeClasse(
            classes: classesDisponibles,
            classeActive: _classe,
            onChoisie: _choisirLaClasse,
          ),
          const SizedBox(height: 16),

          // --- Les valeurs numériques ---
          LigneReglage(
            libelle: 'Niveau',
            valeur: '$_niveau',
            libelleMoins: '-1',
            libellePlus: '+1',
            peutDiminuer: _niveau > niveauMin,
            peutAugmenter: _niveau < niveauMax,
            onDiminuer: () => _modifierLeNiveau(-1),
            onAugmenter: () => _modifierLeNiveau(1),
          ),
          LigneReglage(
            libelle: 'Points de vie',
            valeur: '$_vies',
            libelleMoins: '-10',
            libellePlus: '+10',
            peutDiminuer: _vies > viesMin,
            peutAugmenter: _vies < viesMax,
            onDiminuer: () => _modifierLesVies(-10),
            onAugmenter: () => _modifierLesVies(10),
          ),
          LigneReglage(
            libelle: 'Attaque',
            valeur: '$_attaque',
            libelleMoins: '-5',
            libellePlus: '+5',
            peutDiminuer: _attaque > attaqueMin,
            peutAugmenter: _attaque < attaqueMax,
            onDiminuer: () => _modifierLAttaque(-5),
            onAugmenter: () => _modifierLAttaque(5),
          ),

          // --- Le favori ---
          SwitchListTile(
            title: const Text('Personnage favori'),
            value: _favori,
            onChanged: _changerFavori,
          ),
          const SizedBox(height: 16),

          OutlinedButton(
            onPressed: _reinitialiser,
            child: const Text('Réinitialiser la fiche'),
          ),
        ],
      ),
    );
  }
}

// ---------------------------------------------------------------------------
// Widgets d'affichage : aucun état, uniquement des données et des callbacks.
// ---------------------------------------------------------------------------

class CartePersonnage extends StatelessWidget {
  const CartePersonnage({
    super.key,
    required this.nom,
    required this.classe,
    required this.niveau,
    required this.vies,
    required this.viesMax,
    required this.attaque,
    required this.favori,
  });

  final String nom;
  final String classe;
  final int niveau;
  final int vies;
  final int viesMax;
  final int attaque;
  final bool favori;

  @override
  Widget build(BuildContext context) {
    const int longueurBarre = 20;
    final int pleins = (vies * longueurBarre / viesMax).round();
    final String barre = '#' * pleins + '.' * (longueurBarre - pleins);

    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                CircleAvatar(
                  radius: 24,
                  child: Text(nom.isEmpty ? '?' : nom[0]),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        nom,
                        style: const TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Text('$classe — niveau $niveau'),
                    ],
                  ),
                ),
                if (favori) const Icon(Icons.star, color: Colors.amber, size: 32),
              ],
            ),
            const SizedBox(height: 12),
            Text('Vie : $vies / $viesMax'),
            Text(
              barre,
              style: TextStyle(
                fontSize: 16,
                color: vies == 0 ? Colors.red : Colors.green,
              ),
            ),
            const SizedBox(height: 4),
            Text('Attaque : $attaque'),
            if (vies == 0)
              const Padding(
                padding: EdgeInsets.only(top: 8),
                child: Text(
                  'HORS DE COMBAT',
                  style: TextStyle(color: Colors.red, fontWeight: FontWeight.bold),
                ),
              ),
          ],
        ),
      ),
    );
  }
}

class SelecteurDeClasse extends StatelessWidget {
  const SelecteurDeClasse({
    super.key,
    required this.classes,
    required this.classeActive,
    required this.onChoisie,
  });

  final List<String> classes;
  final String classeActive;
  final ValueChanged<String> onChoisie;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text('Classe'),
        const SizedBox(height: 4),
        Wrap(
          spacing: 8,
          children: [
            for (final String classe in classes)
              if (classe == classeActive)
                ElevatedButton(
                  onPressed: () => onChoisie(classe),
                  child: Text(classe),
                )
              else
                OutlinedButton(
                  onPressed: () => onChoisie(classe),
                  child: Text(classe),
                ),
          ],
        ),
      ],
    );
  }
}

class LigneReglage extends StatelessWidget {
  const LigneReglage({
    super.key,
    required this.libelle,
    required this.valeur,
    required this.libelleMoins,
    required this.libellePlus,
    required this.peutDiminuer,
    required this.peutAugmenter,
    required this.onDiminuer,
    required this.onAugmenter,
  });

  final String libelle;
  final String valeur;
  final String libelleMoins;
  final String libellePlus;
  final bool peutDiminuer;
  final bool peutAugmenter;
  final VoidCallback onDiminuer;
  final VoidCallback onAugmenter;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        children: [
          Expanded(child: Text('$libelle : $valeur')),
          OutlinedButton(
            onPressed: peutDiminuer ? onDiminuer : null,
            child: Text(libelleMoins),
          ),
          const SizedBox(width: 8),
          OutlinedButton(
            onPressed: peutAugmenter ? onAugmenter : null,
            child: Text(libellePlus),
          ),
        ],
      ),
    );
  }
}
```

**Résultat après avoir renommé en « Sophie », choisi « Mage », monté au niveau 3 et
coché le favori :**

```text
┌────────────────────────────────────────────────────┐
│ Fiche de personnage           modifications : 4    │
├────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────┐ │
│ │ ( S )  Sophie                            (*)   │ │
│ │        Mage — niveau 3                         │ │
│ │                                                │ │
│ │ Vie : 100 / 100                                │ │
│ │ ####################                           │ │
│ │ Attaque : 10                                   │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ ÉDITION                                            │
│ ──────────────────────────────────────────────     │
│ ┌────────────────────────────────────────────────┐ │
│ │ Nom du personnage                              │ │
│ │ Sophie                                         │ │
│ └────────────────────────────────────────────────┘ │
│ [ Renommer ]                                       │
│                                                    │
│ Classe                                             │
│ [Guerrier] [ Mage ] [Voleur] [Archer]              │
│            (plein)                                 │
│                                                    │
│ Niveau : 3                        [-1]   [+1]      │
│ Points de vie : 100               [-10]  [+10]     │
│                                          (grisé)   │
│ Attaque : 10                      [-5]   [+5]      │
│                                                    │
│ Personnage favori                        [==O]     │
│                                                    │
│ [ Réinitialiser la fiche ]                         │
└────────────────────────────────────────────────────┘
```

**Résultat quand les points de vie tombent à 0 :**

```text
│ Vie : 0 / 100                                      │
│ ....................                               │
│ Attaque : 10                                       │
│ HORS DE COMBAT                                     │
```

**Explication :** ce projet réunit tout le chapitre.

**Note de lecture.** Trois widgets de mise en page apparaissent ici avant leur
chapitre : `Expanded` (pour qu'un texte occupe l'espace restant d'une `Row`),
`Wrap` (pour que les boutons de classe passent à la ligne si l'écran est étroit)
et `ListView` (pour rendre la page défilante). Ils ne sont pas le sujet de cette
correction ; recopiez-les tels quels, le chapitre 46 les expliquera en détail.

**Un seul détenteur de l'état.** `_EcranFichePersonnageState` contient les six
données de la fiche plus le compteur de modifications. Les trois autres classes,
`CartePersonnage`, `SelecteurDeClasse` et `LigneReglage`, sont des
`StatelessWidget` : elles reçoivent des valeurs et remontent des actions. C'est
exactement le partage de responsabilités décrit en 45.28 — une seule source de
vérité, aucune donnée dupliquée, aucune possibilité que la carte du haut et les
boutons du bas soient en désaccord.

**La donnée descend, l'action remonte.** `SelecteurDeClasse` reçoit
`classeActive` (donnée) et `onChoisie` (action, de type `ValueChanged<String>`).
Il ne sait pas ce que fait `onChoisie` ; il sait seulement qu'il doit l'appeler
avec la classe sur laquelle on a appuyé. `LigneReglage` reçoit deux
`VoidCallback`, et les fonctions anonymes `() => _modifierLeNiveau(-1)` du parent
capturent le pas à appliquer. Cette indirection rend `LigneReglage` réutilisable
pour n'importe quelle valeur numérique : niveau, vie, attaque, et n'importe quoi
d'autre demain.

**Le cycle de vie est respecté.** `_controleurNom` est créé dans `initState()`
avec le nom par défaut, puis libéré dans `dispose()`. Sans ce `dispose()`, chaque
ouverture de l'écran laisserait un contrôleur derrière elle. Notez que
`_reinitialiser()` modifie le contrôleur **hors** du bloc `setState()` : le
contrôleur prévient lui-même le `TextField`, il n'a pas besoin de notre signal.

**Aucun `setState()` inutile.** Chaque méthode d'action commence par une garde :
`if (nouveau == _niveau) return;`. Sans elle, appuyer sur `+1` alors qu'on est
déjà au niveau 50 incrémenterait le compteur de modifications sans que rien ne
change, ce qui serait un bug fonctionnel visible.

**Les bornes sont impossibles à franchir.** Chaque valeur passe par `clamp`. La
désactivation des boutons (`peutAugmenter`, `peutDiminuer`) est une amélioration
d'ergonomie, mais la sécurité réelle vient du `clamp` : même si un bouton était
mal désactivé, l'état resterait cohérent. Ne comptez jamais sur l'interface pour
garantir la validité de vos données.

**Les valeurs dérivées ne sont pas stockées.** La barre de vie, la lettre de
l'avatar et le message « HORS DE COMBAT » sont recalculés dans le `build()` de
`CartePersonnage` à partir des propriétés reçues. Aucun champ ne les mémorise,
donc aucun ne peut être oublié lors d'une mise à jour.

---

## Et maintenant ?

Vous savez désormais faire vivre une interface. Vous savez décider où placer un
état, le modifier proprement, initialiser et libérer des ressources, et faire
circuler l'information entre un parent et ses enfants. C'est la compétence
centrale de Flutter : tout ce qui suit s'appuie dessus.

Mais nos écrans restent laids et rigides. Une `Column` empile ses enfants du haut
vers le bas, un point c'est tout. Dès que vous voudrez placer deux éléments côte à
côte, occuper l'espace restant, superposer un badge sur une carte ou centrer
proprement un contenu, il vous manquera les outils de mise en page.

C'est l'objet du chapitre suivant : `Container`, `Padding`, `SizedBox`, `Row`,
`Column` et leurs alignements, `Expanded`, `Flexible`, `Stack`, et surtout le
système de **contraintes** qui explique pourquoi Flutter met parfois vos widgets
là où vous ne les attendiez pas.

Chapitre suivant : [46-PARTIE-1B—LES-LAYOUTS-ROW-COLUMN-STACK-EXPANDED.md](./46-PARTIE-1B—LES-LAYOUTS-ROW-COLUMN-STACK-EXPANDED.md)
