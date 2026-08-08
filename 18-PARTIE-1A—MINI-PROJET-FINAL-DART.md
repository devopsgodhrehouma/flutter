# PARTIE 1A — DART
# CHAPITRE 18 — MINI-PROJET FINAL : LE MOTEUR DE JEU EN CONSOLE

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** les chapitres 01 à 17 de la partie 1A
> **Ce que vous saurez faire à la fin :** concevoir, écrire, tester et livrer une application Dart complète de plusieurs centaines de lignes, organisée en modèles, services et point d'entrée, avec sauvegarde JSON sur disque.

---

## 18.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- partir d'un cahier des charges et en déduire une liste d'entités ;
- traduire ces entités en classes, en enums et en mixins ;
- dessiner un diagramme de classes et le respecter pendant l'écriture ;
- organiser un projet Dart réel en `bin/`, `lib/models/`, `lib/services/` ;
- écrire une classe abstraite qui factorise ce qui est commun à plusieurs personnages ;
- encapsuler une collection derrière une API sûre ;
- concevoir des exceptions métier et les rattraper au bon endroit ;
- sérialiser et désérialiser un graphe d'objets complet en JSON ;
- lire et valider des saisies clavier sans jamais faire planter le programme ;
- écrire une boucle de jeu qui reste lisible malgré sa complexité ;
- calculer des statistiques avec `where`, `map` et `fold` ;
- écrire des tests unitaires sur les règles du jeu ;
- relire votre propre code et identifier ce qui pourrait être amélioré.

---

## 18.1 — Objectif du projet et aperçu du résultat final

Vous allez construire **« Donjon de Dart »**, un jeu de rôle au tour par tour qui se joue entièrement dans le terminal.

Le principe est classique, donc facile à décrire en une phrase :

> Un aventurier traverse six salles d'un donjon, combat les créatures qui les occupent, ramasse ce qu'elles laissent tomber, monte en niveau, et affronte le seigneur du donjon dans la dernière salle.

Ce projet n'apporte **aucune notion nouvelle**. Il ne fait qu'assembler ce que vous connaissez déjà. C'est précisément son intérêt : jusqu'ici, chaque chapitre vous montrait une pièce isolée. Ici, vous allez découvrir que la difficulté ne vient pas des pièces, mais de leur **assemblage**.

Voici à quoi ressemble une session de jeu une fois le projet terminé.

```text
==================
  DONJON DE DART
==================

1. Nouvelle partie
2. Charger la partie
0. Quitter
Votre choix > 1

Quel est le nom de votre aventurier ? > Alex

Bienvenue, Alex. 6 salles vous séparent du trône de Kraghar.

--------------------------------------------------------
Salle 1/6 : Le seuil moussu
Alex  niv.1  [####################] 100/100 PV  or: 20
--------------------------------------------------------
Une dalle usée marque l'entrée du donjon. L'air sent la pierre humide.

1. Explorer la salle
2. Combattre
3. Inventaire
4. Salle suivante
5. Sauvegarder
6. Statistiques
0. Quitter
Votre choix > 1

Vous fouillez la salle...
Alex ramasse Potion mineure.
Cette salle est désormais sûre.

Votre choix > 4

Vous passez dans la salle suivante.

--------------------------------------------------------
Salle 2/6 : La galerie des torches
Alex  niv.1  [####################] 100/100 PV  or: 20
--------------------------------------------------------
Des torches brûlent seules le long d'un couloir trop long.

Votre choix > 2

Un Gobelin (30 PV) bloque le passage !

--- Tour 1 ---
Alex     [####################] 100/100
Gobelin  [####################] 30/30

1. Attaquer
2. Boire une potion
3. Fuir
Votre choix > 1

Alex inflige 12 dégâts à Gobelin.
Gobelin inflige 6 dégâts à Alex.

--- Tour 2 ---
Alex     [###################.] 94/100
Gobelin  [############........] 18/30

Votre choix > 1

COUP CRITIQUE ! Alex inflige 18 dégâts à Gobelin.
Gobelin est vaincu !

Alex gagne 40 points d'expérience.
Alex récupère 8 pièces d'or.
Alex ramasse Épée rouillée.
Cette salle est désormais sûre.

Votre choix > 3

--- Inventaire (2/10) ---
1. Potion mineure (Potion, Commune, puissance 25)
2. Épée rouillée (Arme, Commune, puissance 6)

Que faire ?
1. Utiliser un objet
0. Retour
Votre choix > 1
Quel emplacement ? > 2

Alex équipe Épée rouillée. Attaque totale : 18.
```

Et voici l'écran de fin, une fois Kraghar vaincu :

```text
============
  VICTOIRE
============

Kraghar s'effondre. Le donjon est à vous.

--- Statistiques de fin de partie ---
Aventurier          : Alex
Niveau atteint      : 4
Salles visitées     : 6 / 6
Salles nettoyées    : 6 / 6
Ennemis vaincus     : 5
Dégâts infligés     : 335
Soins reçus         : 140
Or amassé           : 351
Objets dans le sac  : 4 (valeur 620 pièces)
Répartition         : Commune 1, Rare 1, Épique 1, Légendaire 1
Puissance équipée   : 29
Tours de jeu        : 38
```

Prenez le temps de relire cette sortie. Chaque ligne affichée correspond à une méthode que vous allez écrire. Savoir **à quoi doit ressembler le résultat** avant de coder est la première compétence d'un développeur.

---

## 18.2 — Cahier des charges

Un cahier des charges sépare toujours deux choses : ce qui **doit** exister pour que le produit soit livrable, et ce qui serait agréable en plus. Confondre les deux est la cause numéro un des projets jamais terminés.

### Fonctionnalités obligatoires

| # | Fonctionnalité | Critère de réussite |
| --- | --- | --- |
| O1 | Créer un aventurier | Le joueur saisit un nom, la partie démarre avec des statistiques de départ. |
| O2 | Explorer une salle | Une description s'affiche, les trésors passent dans l'inventaire. |
| O3 | Combattre au tour par tour | Le joueur choisit une action, l'ennemi riposte, le combat s'arrête à la mort de l'un des deux. |
| O4 | Calculer des dégâts variables | Les dégâts dépendent de l'attaque, de la défense, du hasard, et peuvent être critiques. |
| O5 | Gérer un inventaire limité | Le sac a une capacité maximale ; le dépassement lève une exception métier. |
| O6 | Équiper une arme et une armure | L'équipement modifie l'attaque et la défense effectives. |
| O7 | Boire une potion | Les points de vie remontent sans jamais dépasser le maximum. |
| O8 | Monter de niveau | L'expérience accumulée déclenche une amélioration des statistiques. |
| O9 | Avancer de salle en salle | Impossible d'avancer tant qu'un ennemi vivant occupe la salle. |
| O10 | Gagner ou perdre | La mort du joueur ou celle du boss termine la partie. |
| O11 | Sauvegarder en JSON | L'état complet de la partie est écrit dans un fichier. |
| O12 | Charger une sauvegarde | Une partie reprend exactement là où elle s'était arrêtée. |
| O13 | Résister aux saisies invalides | Aucune saisie clavier ne doit provoquer un plantage. |
| O14 | Afficher des statistiques | Un rapport de fin de partie est calculé à partir des collections. |
| O15 | Être testé | Au moins cinq tests unitaires passent avec `dart test`. |

### Fonctionnalités optionnelles

| # | Fonctionnalité | Pourquoi elle est optionnelle |
| --- | --- | --- |
| F1 | Boutique entre deux salles | Aucune règle du jeu n'en dépend. |
| F2 | Sorts et points de magie | Ajoute une deuxième ressource à équilibrer. |
| F3 | Plusieurs emplacements de sauvegarde | Le format JSON le permet déjà, c'est un ajout d'interface. |
| F4 | Niveaux de difficulté | Un simple multiplicateur sur les statistiques ennemies. |
| F5 | Donjon généré aléatoirement | Le donjon fixe suffit à valider les règles. |
| F6 | Coloration ANSI du terminal | Purement esthétique. |

> **Règle de méthode :** vous n'écrivez pas une seule ligne d'une fonctionnalité optionnelle tant que les quinze obligatoires ne fonctionnent pas. Les optionnelles sont d'ailleurs proposées en fin de chapitre sous forme d'exercices d'extension.

---

## 18.3 — Analyse du besoin : identifier les entités

L'analyse consiste à relire le cahier des charges en surlignant les **noms communs**. Un nom commun qui revient souvent et qui porte des données est presque toujours une classe.

Reprenons les phrases du cahier des charges :

```text
  « Le JOUEUR explore des SALLES »
  « Le joueur combat des ENNEMIS »
  « Le joueur ramasse des OBJETS »
  « Le joueur gère son INVENTAIRE »
  « Le DONJON contient six salles »
  « Le combat suit des règles »
  « La PARTIE peut être sauvegardée »
```

Les noms en majuscules sont nos candidats. Passons-les au crible avec trois questions.

| Candidat | A-t-il des données propres ? | A-t-il un comportement propre ? | Décision |
| --- | --- | --- | --- |
| Joueur | nom, PV, attaque, niveau, or | attaquer, boire, équiper | **classe** |
| Ennemi | nom, PV, attaque, butin | attaquer | **classe** |
| Objet | nom, type, rareté, puissance | aucun vrai comportement | **classe** (données) |
| Inventaire | une liste d'objets, une capacité | ajouter, retirer, filtrer | **classe** |
| Salle | numéro, description, ennemi, trésors | aucun | **classe** (données) |
| Donjon | une liste de salles | parcourir | **classe** |
| Partie | joueur + donjon + compteurs | aucun | **classe** (état) |
| Combat | rien à stocker durablement | calculer, arbitrer | **service** |
| Sauvegarde | rien à stocker | écrire, lire | **service** |
| Type d'objet | un ensemble fini de valeurs | — | **enum** |
| Rareté | un ensemble fini de valeurs | — | **enum** |

Deux observations importantes.

**Première observation :** « Joueur » et « Ennemi » partagent beaucoup de choses. Tous deux ont un nom, des points de vie, une attaque, une défense ; tous deux peuvent subir des dégâts et mourir. C'est exactement la situation qui appelle une **classe abstraite** commune : `Personnage`.

**Deuxième observation :** « Combat » et « Sauvegarde » ne stockent rien de durable. Ce ne sont pas des choses, ce sont des **actions**. On les met dans des classes appelées **services**, rangées dans `lib/services/`. Un service, c'est une classe qui fait quelque chose plutôt qu'une classe qui est quelque chose.

---

## 18.4 — Modélisation : le diagramme des classes

Voici la carte du projet. Gardez-la sous les yeux pendant toute la lecture.

```text
                        ┌──────────────────────────┐
                        │   <<abstract>>           │
                        │   Personnage             │
                        ├──────────────────────────┤
                        │ + nom : String           │
                        │ - _pointsDeVie : int     │
                        │ - _pointsDeVieMax : int  │
                        │ + attaque : int          │
                        │ + defense : int          │
                        ├──────────────────────────┤
                        │ + estVivant : bool       │
                        │ + subirDegats(int) : int │
                        │ + description  (abstrait)│
                        │ + toJson()     (abstrait)│
                        └────────────┬─────────────┘
                                     │  extends
                  ┌──────────────────┴───────────────────┐
                  │                                      │
        ┌─────────▼──────────┐               ┌───────────▼──────────┐
        │  Joueur            │               │  Ennemi              │
        │  with Soignable    │               ├──────────────────────┤
        ├────────────────────┤               │ + experienceDonnee   │
        │ + niveau : int     │               │ + orDonne : int      │
        │ + experience : int │               │ + estBoss : bool     │
        │ + or : int         │               │ + butin : Objet?     │
        │ + inventaire       │               ├──────────────────────┤
        │ + armeEquipee : ?  │               │ Ennemi.gobelin()     │
        │ + armureEquipee: ? │               │ Ennemi.squelette()   │
        ├────────────────────┤               │ Ennemi.golem()       │
        │ + attaqueTotale    │               │ Ennemi.boss()        │
        │ + gagnerExperience │               └──────────────────────┘
        │ + equiper(Objet)   │
        └─────────┬──────────┘
                  │ possède 1
        ┌─────────▼──────────┐        contient 0..n     ┌──────────────────┐
        │  Inventaire        │─────────────────────────>│  Objet           │
        ├────────────────────┤                          ├──────────────────┤
        │ - _objets : List   │                          │ + nom : String   │
        │ + capacite : int   │                          │ + type : TypeObjet
        │ + estPlein : bool  │                          │ + rarete : Rarete│
        ├────────────────────┤                          │ + puissance : int│
        │ + ajouter(Objet)   │                          │ + valeur : int   │
        │ + retirerA(int)    │                          └──────────────────┘
        └────────────────────┘

        ┌──────────────────┐  contient 1..n   ┌─────────────────────────┐
        │  Donjon          │─────────────────>│  Salle                  │
        ├──────────────────┤                  ├─────────────────────────┤
        │ - _salles : List │                  │ + numero, nom, descr.   │
        │ + estTermine     │                  │ + ennemi : Ennemi?      │
        └──────────────────┘                  │ + tresors : List<Objet> │
                                              │ + visitee, nettoyee     │
                                              └─────────────────────────┘

        ┌──────────────────────────┐
        │  Partie   (état global)  │
        ├──────────────────────────┤
        │ + joueur : Joueur        │
        │ + donjon : Donjon        │
        │ + tourDeJeu, terminee    │
        └──────────────────────────┘

   SERVICES (ils agissent, ils ne stockent pas l'état du jeu)

   ┌────────────────────┐  ┌──────────────────────┐  ┌──────────────────┐
   │  MoteurCombat      │  │  SauvegardeService   │  │  Console         │
   ├────────────────────┤  ├──────────────────────┤  ├──────────────────┤
   │ + attaquer()       │  │ + sauvegarder()      │  │ + ecrire()       │
   │ + calculerDegats() │  │ + charger()          │  │ + lireLigne()    │
   │ + jouerTour()      │  │ + existe()           │  │ + lireEntier()   │
   └────────────────────┘  └──────────────────────┘  └──────────────────┘

   MIXIN                    ENUMS                    EXCEPTIONS
   ┌──────────────┐   ┌───────────────┐   ┌────────────────────────────────┐
   │ Soignable    │   │ TypeObjet     │   │ ErreurDeJeu (abstraite)        │
   │  on          │   │ Rarete        │   │  ├─ InventairePleinException   │
   │  Personnage  │   │ ActionCombat  │   │  ├─ ObjetIntrouvableException  │
   └──────────────┘   └───────────────┘   │  └─ SauvegardeCorrompueExcept. │
                                          └────────────────────────────────┘
```

Trois règles de lecture de ce diagramme :

1. **Les flèches d'héritage remontent.** `Joueur` et `Ennemi` pointent vers `Personnage`, jamais l'inverse. `Personnage` ne sait pas qu'un joueur existe.
2. **Les services connaissent les modèles, jamais l'inverse.** `MoteurCombat` importe `Joueur` ; `Joueur` n'importe jamais `MoteurCombat`. Si vous voyez apparaître une flèche dans les deux sens, votre conception est fausse.
3. **`Soignable` s'applique au joueur seulement.** Les ennemis de ce donjon ne se soignent pas. C'est exactement le rôle d'un mixin : ajouter une capacité à certaines classes, pas à toute une branche d'héritage.

---

## 18.5 — Arborescence du projet

Créez le projet avec l'outil officiel, comme au chapitre 16.

```bash
dart create -t console donjon_de_dart
cd donjon_de_dart
```

Puis créez les dossiers manquants :

```bash
mkdir -p lib/models lib/services lib/exceptions lib/utils test sauvegardes
```

Vous devez obtenir exactement cette arborescence :

```text
donjon_de_dart/
├── bin/
│   └── main.dart                     point d'entrée : 10 lignes, pas plus
├── lib/
│   ├── jeu.dart                      la boucle de jeu et les menus
│   ├── exceptions/
│   │   └── erreurs_jeu.dart          les exceptions métier du projet
│   ├── models/
│   │   ├── donjon.dart
│   │   ├── ennemi.dart
│   │   ├── inventaire.dart
│   │   ├── joueur.dart
│   │   ├── objet.dart
│   │   ├── partie.dart
│   │   ├── personnage.dart
│   │   ├── salle.dart
│   │   ├── soignable.dart
│   │   └── types.dart                les enums TypeObjet et Rarete
│   ├── services/
│   │   ├── console.dart
│   │   ├── moteur_combat.dart
│   │   ├── sauvegarde_service.dart
│   │   └── statistiques.dart
│   └── utils/
│       └── outils.dart               fonctions utilitaires sans état
├── sauvegardes/
│   └── partie.json                   créé à la première sauvegarde
├── test/
│   └── donjon_de_dart_test.dart
├── analysis_options.yaml
└── pubspec.yaml
```

Le `pubspec.yaml` du projet :

```yaml
name: donjon_de_dart
description: Un jeu de rôle au tour par tour en console, projet final de la partie Dart.
version: 1.0.0
publish_to: none

environment:
  sdk: ^3.5.0

dev_dependencies:
  lints: ^4.0.0
  test: ^1.25.0
```

Puis :

```bash
dart pub get
```

**Trois conventions de nommage à respecter :**

- un fichier Dart s'écrit en `snake_case` : `moteur_combat.dart`, jamais `MoteurCombat.dart` ;
- un fichier contient en principe une seule classe publique, et porte son nom ;
- à l'intérieur du package, on s'importe avec `package:donjon_de_dart/...` depuis `bin/` et `test/`, et avec des chemins relatifs (`../models/joueur.dart`) à l'intérieur de `lib/`.

---

## 18.6 — Les enums `TypeObjet` et `Rarete`

Un objet du jeu appartient à une catégorie, et cette catégorie fait partie d'une liste **fermée**. On ne veut pas d'un `String type` qui autoriserait `'Potiion'` avec deux « i ». C'est le cas d'usage exact d'un `enum`.

```dart
enum TypeObjet {
  arme('Arme'),
  armure('Armure'),
  potion('Potion'),
  cle('Clé'),
  tresor('Trésor');

  const TypeObjet(this.libelle);

  final String libelle;

  static TypeObjet? parNom(Object? nom) {
    for (final TypeObjet type in TypeObjet.values) {
      if (type.name == nom) return type;
    }
    return null;
  }
}
```

Trois points à noter :

- l'enum porte une **donnée associée** (`libelle`), ce qui évite un `switch` d'affichage éparpillé dans le code ;
- `type.name` renvoie l'identifiant Dart (`'potion'`), c'est ce qu'on écrira dans le JSON, car il ne changera pas si vous traduisez les libellés ;
- `parNom` renvoie `TypeObjet?`. Elle vaut `null` si le JSON contient une valeur inconnue. On ne lève pas d'exception ici : c'est l'appelant qui décidera quoi faire.

La rareté, elle, porte un **multiplicateur** de puissance :

```dart
enum Rarete {
  commune('Commune', 1.0),
  rare('Rare', 1.5),
  epique('Épique', 2.0),
  legendaire('Légendaire', 3.0);

  const Rarete(this.libelle, this.multiplicateur);

  final String libelle;
  final double multiplicateur;

  static Rarete? parNom(Object? nom) {
    for (final Rarete rarete in Rarete.values) {
      if (rarete.name == nom) return rarete;
    }
    return null;
  }
}
```

Une potion de puissance `30` en rareté `rare` soignera donc `30 × 1.5 = 45` points de vie. Toute la règle d'équilibrage tient dans une seule colonne de l'enum : pour rendre les objets épiques plus forts, vous changez **un seul nombre**, pas vingt objets.

Petit programme de vérification, exécutable tel quel :

```dart
void main() {
  for (final Rarete rarete in Rarete.values) {
    final int puissanceBrute = 30;
    final int effective = (puissanceBrute * rarete.multiplicateur).round();
    print('${rarete.libelle.padRight(12)} x${rarete.multiplicateur}  ->  $effective');
  }
}
```

**Résultat :**

```text
Commune      x1.0  ->  30
Rare         x1.5  ->  45
Épique       x2.0  ->  60
Légendaire   x3.0  ->  90
```

---

## 18.7 — La classe abstraite `Personnage`

`Personnage` répond à une question précise : **qu'est-ce qui est vrai pour tout être vivant du donjon ?**

Réponse : il a un nom, des points de vie bornés entre zéro et un maximum, une attaque, une défense ; il peut subir des dégâts et mourir.

En revanche, on ne peut pas répondre à « comment se décrit-il ? » ni « comment se sérialise-t-il ? » de façon générale. Ces deux méthodes seront donc **abstraites** : déclarées ici, obligatoirement écrites dans les sous-classes.

```dart
abstract class Personnage {
  Personnage({
    required this.nom,
    required int pointsDeVieMax,
    required this.attaque,
    required this.defense,
    int? pointsDeVie,
  })  : _pointsDeVieMax = pointsDeVieMax,
        _pointsDeVie = borner(pointsDeVie ?? pointsDeVieMax, 0, pointsDeVieMax);

  final String nom;
  int attaque;
  int defense;

  int _pointsDeVieMax;
  int _pointsDeVie;

  int get pointsDeVieMax => _pointsDeVieMax;
  int get pointsDeVie => _pointsDeVie;

  bool get estVivant => _pointsDeVie > 0;
  bool get estMort => _pointsDeVie <= 0;

  String get barreVie => barreDeVie(_pointsDeVie, _pointsDeVieMax);

  void definirPointsDeVie(int valeur) {
    _pointsDeVie = borner(valeur, 0, _pointsDeVieMax);
  }

  int subirDegats(int degats) {
    if (degats <= 0) return 0;
    final int avant = _pointsDeVie;
    definirPointsDeVie(avant - degats);
    return avant - _pointsDeVie;
  }

  void augmenterPointsDeVieMax(int bonus) {
    if (bonus <= 0) return;
    _pointsDeVieMax += bonus;
    _pointsDeVie += bonus;
  }

  String get description;

  Map<String, dynamic> toJson();

  @override
  String toString() => '$nom ($_pointsDeVie/$_pointsDeVieMax PV)';
}
```

Analysons les décisions de conception, une par une.

**`_pointsDeVie` est privé.** C'est l'application directe de l'encapsulation du chapitre 10. Si le champ était public, n'importe quelle ligne du projet pourrait écrire `joueur.pointsDeVie = -500` ou `joueur.pointsDeVie = 999999`. Ici, c'est impossible : la seule porte d'entrée est `definirPointsDeVie`, qui **borne** systématiquement la valeur.

**`subirDegats` renvoie un `int`.** Elle renvoie les dégâts *réellement* appliqués. Si un ennemi a 3 PV et reçoit 20 dégâts, il perd 3 points, pas 20. Le moteur de combat a besoin de ce chiffre exact pour les statistiques de fin de partie.

**`definirPointsDeVie` est publique et non privée.** Elle doit rester accessible au mixin `Soignable`, qui vit dans un autre fichier. Rappelez-vous : en Dart, `_` signifie « privé à la **bibliothèque** », c'est-à-dire au fichier. Un mixin déclaré ailleurs ne verrait pas `_pointsDeVie`.

**`description` et `toJson` sont abstraites.** Elles n'ont pas de corps. Toute classe concrète qui étend `Personnage` est obligée de les écrire, sinon le code ne compile pas. C'est du polymorphisme garanti par le compilateur.

**Aucun `!` dans cette classe.** Conformément au chapitre 12.

Les deux fonctions `borner` et `barreDeVie` viennent de `lib/utils/outils.dart` :

```dart
int borner(int valeur, int minimum, int maximum) {
  if (valeur < minimum) return minimum;
  if (valeur > maximum) return maximum;
  return valeur;
}

String barreDeVie(int actuel, int maximum, {int largeur = 20}) {
  if (maximum <= 0) return '[${'.' * largeur}]';
  final int pleins = borner((actuel * largeur / maximum).round(), 0, largeur);
  return '[${'#' * pleins}${'.' * (largeur - pleins)}]';
}
```

Vérification rapide :

```dart
void main() {
  print(barreDeVie(100, 100));
  print(barreDeVie(50, 100));
  print(barreDeVie(7, 100));
  print(barreDeVie(0, 100));
}
```

**Résultat :**

```text
[####################]
[##########..........]
[#...................]
[....................]
```

---

## 18.8 — La classe `Joueur`

`Joueur` étend `Personnage` et lui ajoute tout ce qui est propre à un être humain aux commandes : progression, économie, équipement, position dans le donjon.

```dart
class Joueur extends Personnage with Soignable {
  Joueur({
    required String nom,
    int pointsDeVieMax = 100,
    int? pointsDeVie,
    int attaque = 12,
    int defense = 5,
    this.niveau = 1,
    this.experience = 0,
    this.or = 20,
    this.salleCourante = 0,
    this.ennemisVaincus = 0,
    this.degatsInfliges = 0,
    this.armeEquipee,
    this.armureEquipee,
    Inventaire? inventaire,
  })  : inventaire = inventaire ?? Inventaire(),
        super(
          nom: nom,
          pointsDeVieMax: pointsDeVieMax,
          pointsDeVie: pointsDeVie,
          attaque: attaque,
          defense: defense,
        );

  int niveau;
  int experience;
  int or;
  int salleCourante;
  int ennemisVaincus;
  int degatsInfliges;

  Objet? armeEquipee;
  Objet? armureEquipee;

  final Inventaire inventaire;

  int get experienceRequise => niveau * 100;

  int get attaqueTotale => attaque + (armeEquipee?.puissanceEffective ?? 0);
  int get defenseTotale => defense + (armureEquipee?.puissanceEffective ?? 0);

  @override
  String get description =>
      '$nom  niv.$niveau  $barreVie $pointsDeVie/$pointsDeVieMax PV  or: $or';
}
```

Notez trois choses.

**Les valeurs par défaut sont dans le constructeur.** `pointsDeVieMax = 100`, `attaque = 12`, `defense = 5`, `or = 20` : ce sont les statistiques de départ du jeu. On peut toutes les surcharger, ce qui rendra les tests unitaires beaucoup plus simples à écrire.

**`armeEquipee` et `armureEquipee` sont nullables.** C'est le bon usage du nullable vu au chapitre 12 : « pas d'arme » est une information réelle du jeu, pas un oubli d'initialisation.

**`attaqueTotale` utilise `?.` et `??`.** Relisez l'expression :

```dart
int get attaqueTotale => attaque + (armeEquipee?.puissanceEffective ?? 0);
```

Elle se lit : « ma valeur d'attaque brute, plus la puissance de mon arme si j'en ai une, plus zéro sinon ». Deux opérateurs, une ligne, aucun `if`, aucun `!`.

Les méthodes du joueur :

```dart
  String ramasser(Objet objet) {
    inventaire.ajouter(objet);
    return '$nom ramasse ${objet.nom}.';
  }

  String equiper(Objet objet) {
    if (objet.type == TypeObjet.arme) {
      final Objet? ancienne = armeEquipee;
      armeEquipee = objet;
      if (ancienne != null) {
        inventaire.ajouter(ancienne);
      }
      return '$nom équipe ${objet.nom}. Attaque totale : $attaqueTotale.';
    }
    if (objet.type == TypeObjet.armure) {
      final Objet? ancienne = armureEquipee;
      armureEquipee = objet;
      if (ancienne != null) {
        inventaire.ajouter(ancienne);
      }
      return '$nom enfile ${objet.nom}. Défense totale : $defenseTotale.';
    }
    throw ActionImpossibleException('${objet.nom} ne s\'équipe pas.');
  }
```

Une subtilité qui vaut d'être soulignée : dans `equiper`, on remet l'ancienne arme dans le sac avec `inventaire.ajouter`. Ne pourrait-elle pas lever `InventairePleinException` ? Non, parce que `equiper` est toujours appelée après avoir **retiré** l'objet du sac. Une place vient donc de se libérer. C'est le genre d'invariant qu'il faut savoir énoncer et, mieux encore, tester.

La montée de niveau et l'utilisation d'objets font l'objet des sections 18.17 et 18.11.

---

## 18.9 — La classe `Ennemi` et ses constructeurs nommés

Un ennemi est un `Personnage` qui, en plus, **récompense** celui qui le tue.

```dart
class Ennemi extends Personnage {
  Ennemi({
    required String nom,
    required int pointsDeVieMax,
    required int attaque,
    required int defense,
    int? pointsDeVie,
    this.experienceDonnee = 20,
    this.orDonne = 5,
    this.estBoss = false,
    this.butin,
  }) : super(
          nom: nom,
          pointsDeVieMax: pointsDeVieMax,
          pointsDeVie: pointsDeVie,
          attaque: attaque,
          defense: defense,
        );

  final int experienceDonnee;
  final int orDonne;
  final bool estBoss;
  final Objet? butin;
```

Jusqu'ici, rien de neuf. La partie intéressante, ce sont les **constructeurs nommés redirigés** du chapitre 09 :

```dart
  Ennemi.gobelin()
      : this(
          nom: 'Gobelin',
          pointsDeVieMax: 30,
          attaque: 8,
          defense: 2,
          experienceDonnee: 40,
          orDonne: 8,
          butin: Catalogue.epeeRouillee,
        );

  Ennemi.squelette()
      : this(
          nom: 'Squelette',
          pointsDeVieMax: 45,
          attaque: 12,
          defense: 5,
          experienceDonnee: 70,
          orDonne: 20,
          butin: Catalogue.potionMajeure,
        );

  Ennemi.golem()
      : this(
          nom: 'Golem de pierre',
          pointsDeVieMax: 70,
          attaque: 16,
          defense: 9,
          experienceDonnee: 130,
          orDonne: 45,
          butin: Catalogue.cotteDeMailles,
        );

  Ennemi.boss()
      : this(
          nom: 'Kraghar',
          pointsDeVieMax: 160,
          attaque: 22,
          defense: 12,
          experienceDonnee: 400,
          orDonne: 250,
          estBoss: true,
          butin: Catalogue.couronneDuDonjon,
        );
```

Le mot-clé `this(...)` signifie : « appelle mon propre constructeur principal avec ces valeurs ». C'est un **constructeur redirigé**. Il n'a pas de corps, pas d'accolades, juste une liste d'initialisation.

Pourquoi est-ce meilleur qu'une fonction `creerGobelin()` ?

| Avec `Ennemi.gobelin()` | Avec une fonction libre |
| --- | --- |
| Le bestiaire est dans la classe qu'il concerne | Le bestiaire est ailleurs, on doit le chercher |
| L'autocomplétion propose tous les monstres après `Ennemi.` | Il faut connaître le nom exact de la fonction |
| Impossible d'oublier un champ requis | Idem, mais rien ne le rappelle |

Vérification :

```dart
void main() {
  final List<Ennemi> bestiaire = [
    Ennemi.gobelin(),
    Ennemi.squelette(),
    Ennemi.golem(),
    Ennemi.boss(),
  ];

  for (final Ennemi ennemi in bestiaire) {
    print(ennemi.description);
  }
}
```

**Résultat :**

```text
Gobelin  [####################] 30/30  att 8  def 2  (40 xp, 8 or)
Squelette  [####################] 45/45  att 12  def 5  (70 xp, 20 or)
Golem de pierre  [####################] 70/70  att 16  def 9  (130 xp, 45 or)
Kraghar  [####################] 160/160  att 22  def 12  (400 xp, 250 or)
```

`description` est bien la méthode abstraite de `Personnage`, redéfinie ici :

```dart
  @override
  String get description =>
      '$nom  $barreVie $pointsDeVie/$pointsDeVieMax  att $attaque  def $defense'
      '  ($experienceDonnee xp, $orDonne or)';
```

Un même appel, `personnage.description`, produit une phrase différente selon que l'objet est un `Joueur` ou un `Ennemi`. C'est le polymorphisme du chapitre 10, en situation réelle.

---

## 18.10 — La classe `Objet`

Un objet du jeu est une **donnée immuable** : une épée rouillée ne change jamais de nom ni de puissance. Tous ses champs sont donc `final`, et son constructeur est `const`.

```dart
class Objet {
  const Objet({
    required this.nom,
    required this.type,
    required this.rarete,
    this.puissance = 0,
    this.valeur = 0,
    this.description = '',
  });

  final String nom;
  final TypeObjet type;
  final Rarete rarete;
  final int puissance;
  final int valeur;
  final String description;

  int get puissanceEffective => (puissance * rarete.multiplicateur).round();

  bool get estConsommable => type == TypeObjet.potion;
  bool get estEquipable => type == TypeObjet.arme || type == TypeObjet.armure;

  @override
  String toString() =>
      '$nom (${type.libelle}, ${rarete.libelle}, puissance $puissanceEffective)';
}
```

### Pourquoi il faut redéfinir `==`

Voici un piège dans lequel tombent presque tous les débutants, et il est spécifique à ce projet.

Quand le joueur boit une potion, on veut la retirer du sac :

```dart
inventaire.retirer(potion);
```

`List.remove` compare les éléments avec `==`. Par défaut, `==` en Dart compare les **identités** : deux objets sont égaux seulement s'ils occupent la même case mémoire.

Tant que la partie tourne, cela fonctionne par chance, car les objets viennent tous du catalogue `const` et Dart **canonise** les constantes : deux `const Objet(...)` identiques sont un seul et même objet en mémoire.

Mais après un **chargement de sauvegarde**, chaque objet est reconstruit par `Objet.fromJson`. Ce sont de nouvelles instances. `remove` ne trouve plus rien, et le jeu déclare que la potion n'existe pas alors qu'elle est affichée à l'écran.

La correction consiste à définir l'égalité **par valeur** :

```dart
  @override
  bool operator ==(Object autre) {
    if (identical(this, autre)) return true;
    return autre is Objet &&
        autre.nom == nom &&
        autre.type == type &&
        autre.rarete == rarete &&
        autre.puissance == puissance &&
        autre.valeur == valeur;
  }

  @override
  int get hashCode => Object.hash(nom, type, rarete, puissance, valeur);
```

> **Règle absolue :** si vous redéfinissez `==`, vous **devez** redéfinir `hashCode`. Sinon un `Set` ou une `Map` se comporteront de façon incohérente.

Démonstration du problème et de sa correction :

```dart
void main() {
  const Objet a = Objet(
      nom: 'Potion mineure', type: TypeObjet.potion, rarete: Rarete.commune,
      puissance: 25, valeur: 10);

  final Objet b = Objet(
      nom: 'Potion mineure', type: TypeObjet.potion, rarete: Rarete.commune,
      puissance: 25, valeur: 10);

  print('identiques en mémoire ? ${identical(a, b)}');
  print('égales par valeur ?     ${a == b}');
}
```

**Résultat (avec `==` redéfini) :**

```text
identiques en mémoire ? false
égales par valeur ?     true
```

### Le catalogue

Tous les objets du jeu sont déclarés une fois pour toutes :

```dart
class Catalogue {
  Catalogue._();

  static const Objet potionMineure = Objet(
    nom: 'Potion mineure',
    type: TypeObjet.potion,
    rarete: Rarete.commune,
    puissance: 25,
    valeur: 10,
    description: 'Un flacon tiède au goût de fer.',
  );

  static const Objet potionMajeure = Objet(
    nom: 'Potion majeure',
    type: TypeObjet.potion,
    rarete: Rarete.rare,
    puissance: 30,
    valeur: 30,
    description: 'Elle sent la menthe et la magie.',
  );

  // ... (voir le code source complet en fin de chapitre)

  static const List<Objet> tous = <Objet>[
    potionMineure,
    potionMajeure,
    epeeRouillee,
    lameDeFeu,
    armureDeCuir,
    cotteDeMailles,
    amuletteAncienne,
    couronneDuDonjon,
  ];
}
```

Le constructeur privé `Catalogue._()` empêche d'écrire `Catalogue()`. C'est le signal, adressé au lecteur, que cette classe n'est qu'un porte-constantes : on ne l'instancie jamais.

---

## 18.11 — La classe `Inventaire`

L'inventaire est le meilleur exemple d'**encapsulation** de tout le projet. Il contient une `List<Objet>`, mais il ne la donne à personne.

```dart
class Inventaire {
  Inventaire({this.capacite = 10, List<Objet>? objets})
      : _objets = List<Objet>.from(objets ?? const <Objet>[]);

  final int capacite;
  final List<Objet> _objets;

  List<Objet> get objets => List<Objet>.unmodifiable(_objets);

  int get taille => _objets.length;
  bool get estPlein => _objets.length >= capacite;
  bool get estVide => _objets.isEmpty;

  void ajouter(Objet objet) {
    if (estPlein) {
      throw InventairePleinException(objet.nom, capacite);
    }
    _objets.add(objet);
  }

  Objet retirerA(int index) {
    if (index < 0 || index >= _objets.length) {
      throw ObjetIntrouvableException('emplacement ${index + 1}');
    }
    return _objets.removeAt(index);
  }

  bool retirer(Objet objet) => _objets.remove(objet);

  Objet? consulter(int index) {
    if (index < 0 || index >= _objets.length) return null;
    return _objets[index];
  }

  List<Objet> parType(TypeObjet type) =>
      _objets.where((Objet objet) => objet.type == type).toList();

  int get valeurTotale =>
      _objets.fold(0, (int total, Objet objet) => total + objet.valeur);
}
```

Quatre décisions méritent une explication.

**1. `_objets` est privée.** Personne, hors de ce fichier, ne peut faire `inventaire._objets.add(...)` et contourner la vérification de capacité.

**2. Le getter `objets` renvoie une copie non modifiable.** C'est le point le plus important, et le plus souvent raté.

```dart
  List<Objet> get objets => List<Objet>.unmodifiable(_objets);
```

Si vous écriviez simplement `List<Objet> get objets => _objets;`, vous rendriez l'encapsulation **inutile** : n'importe quel appelant récupérerait une référence vers la vraie liste et pourrait la modifier.

Démonstration de la faille, puis de la protection :

```dart
void main() {
  final Inventaire sac = Inventaire(capacite: 2);
  sac.ajouter(Catalogue.potionMineure);

  print('taille : ${sac.taille}');

  try {
    sac.objets.add(Catalogue.potionMajeure); // tentative de contournement
  } on UnsupportedError {
    print('Modification refusée : la liste exposée est non modifiable.');
  }

  print('taille après tentative : ${sac.taille}');
}
```

**Résultat :**

```text
taille : 1
Modification refusée : la liste exposée est non modifiable.
taille après tentative : 1
```

**3. Le constructeur **copie** la liste reçue.** `List<Objet>.from(...)` crée une nouvelle liste. Sans cette copie, l'appelant garderait une référence sur la liste interne : même faille, par une autre porte.

**4. `ajouter` lève une exception, `consulter` renvoie `null`.** Ce n'est pas une incohérence, c'est une règle :

> Une exception signale une situation **anormale** ; un `null` signale une situation **prévue**.

Ajouter dans un sac plein est une erreur du joueur, il faut le lui dire. Consulter un emplacement vide est une question légitime à laquelle la réponse est simplement « rien ».

---

## 18.12 — Le mixin `Soignable`

Le joueur peut se soigner. Les monstres de ce donjon, non. Nous ne pouvons donc pas mettre `soigner` dans `Personnage` : cela donnerait aux gobelins une capacité qu'ils n'ont pas.

Créer une classe intermédiaire `PersonnageSoignable` entre `Personnage` et `Joueur` serait lourd, et deviendrait ingérable dès qu'on ajouterait une deuxième capacité (empoisonnable, volant, invisible...). C'est exactement le problème que résolvent les **mixins** du chapitre 11.

```dart
mixin Soignable on Personnage {
  int _soinsRecus = 0;

  int get soinsRecus => _soinsRecus;

  bool get estBlesse => pointsDeVie < pointsDeVieMax;

  int get pointsDeVieManquants => pointsDeVieMax - pointsDeVie;

  int soigner(int points) {
    if (points <= 0) return 0;
    final int avant = pointsDeVie;
    definirPointsDeVie(avant + points);
    final int gagnes = pointsDeVie - avant;
    _soinsRecus += gagnes;
    return gagnes;
  }

  int soignerCompletement() => soigner(pointsDeVieManquants);
}
```

Le mot-clé décisif est `on Personnage`. Il signifie : « ce mixin ne peut être appliqué qu'à une classe qui hérite de `Personnage` ». En échange, le mixin a le droit d'utiliser `pointsDeVie`, `pointsDeVieMax` et `definirPointsDeVie` comme s'ils étaient à lui.

C'est ce qui permet à `soigner` de rester si courte : le bornage à `pointsDeVieMax` est déjà garanti par `definirPointsDeVie`.

`soigner` renvoie le nombre de points **réellement** gagnés. Un joueur à 95/100 qui boit une potion de 45 ne gagne que 5 points, et le message affiché doit dire 5, pas 45.

Application :

```dart
class Joueur extends Personnage with Soignable { ... }
```

Vérification :

```dart
void main() {
  final Joueur alex = Joueur(nom: 'Alex');

  alex.subirDegats(70);
  print('Après le combat : ${alex.pointsDeVie}/${alex.pointsDeVieMax}');
  print('Blessé ? ${alex.estBlesse}  Manquants : ${alex.pointsDeVieManquants}');

  final int gagnes1 = alex.soigner(45);
  print('Potion majeure : +$gagnes1 PV  ->  ${alex.pointsDeVie}');

  final int gagnes2 = alex.soigner(45);
  print('Deuxième potion : +$gagnes2 PV  ->  ${alex.pointsDeVie}');

  print('Total soigné sur la partie : ${alex.soinsRecus}');
}
```

**Résultat :**

```text
Après le combat : 30/100
Blessé ? true  Manquants : 70
Potion majeure : +45 PV  ->  75
Deuxième potion : +25 PV  ->  100
Total soigné sur la partie : 70
```

Observez la deuxième potion : elle ne rend que 25 points, car le joueur plafonne à 100. Aucun `if` n'a été écrit pour cela dans `soigner` ; le bornage vient de `Personnage`. C'est le bénéfice d'une bonne répartition des responsabilités.

---

## 18.13 — La classe `Salle`

Une salle est une donnée de décor : un numéro, un nom, une description, éventuellement un occupant, éventuellement des trésors, et deux drapeaux d'état.

```dart
class Salle {
  Salle({
    required this.numero,
    required this.nom,
    required this.description,
    this.ennemi,
    List<Objet>? tresors,
    this.visitee = false,
    this.nettoyee = false,
  }) : tresors = tresors ?? <Objet>[];

  final int numero;
  final String nom;
  final String description;

  Ennemi? ennemi;
  final List<Objet> tresors;

  bool visitee;
  bool nettoyee;

  bool get contientUnEnnemiVivant {
    final Ennemi? occupant = ennemi;
    return occupant != null && occupant.estVivant;
  }

  String get etat {
    if (contientUnEnnemiVivant) return 'DANGER';
    if (nettoyee) return 'sûre';
    if (visitee) return 'explorée';
    return 'inconnue';
  }

  List<Objet> viderLesTresors() {
    final List<Objet> butin = List<Objet>.from(tresors);
    tresors.clear();
    return butin;
  }
}
```

Le getter `contientUnEnnemiVivant` est un rappel direct du chapitre 12 :

```dart
  bool get contientUnEnnemiVivant {
    final Ennemi? occupant = ennemi;      // copie locale
    return occupant != null && occupant.estVivant;
  }
```

Pourquoi cette copie locale ? Parce que `ennemi` est un **champ de classe**. Dart refuse de promouvoir un champ, même après un test `!= null`, car rien ne garantit qu'un autre bout de code ne l'a pas remis à `null` entre-temps. En le copiant dans une variable locale `final`, la promotion fonctionne et `occupant.estVivant` compile sans `!`.

Si vous écriviez `return ennemi != null && ennemi.estVivant;`, le compilateur refuserait avec le message :

```text
Error: The property 'estVivant' can't be unconditionally accessed because
the receiver can be 'null'.
```

`viderLesTresors` renvoie une copie **puis** vide la liste. Ainsi, on ne peut pas ramasser deux fois le même trésor en revenant dans la salle.

---

## 18.14 — La classe `Donjon`

Le donjon est une liste ordonnée de salles, encapsulée selon le même principe que l'inventaire.

```dart
class Donjon {
  Donjon({required this.nom, required List<Salle> salles})
      : _salles = List<Salle>.from(salles);

  final String nom;
  final List<Salle> _salles;

  List<Salle> get salles => List<Salle>.unmodifiable(_salles);

  int get nombreDeSalles => _salles.length;

  Salle salleA(int index) {
    if (index < 0 || index >= _salles.length) {
      throw ActionImpossibleException('la salle numéro ${index + 1} n\'existe pas.');
    }
    return _salles[index];
  }

  bool get estTermine => _salles.every((Salle salle) => salle.nettoyee);

  int get sallesVisitees =>
      _salles.where((Salle salle) => salle.visitee).length;

  factory Donjon.parDefaut() {
    return Donjon(
      nom: 'Le Donjon de Kraghar',
      salles: <Salle>[
        Salle(
          numero: 1,
          nom: 'Le seuil moussu',
          description: 'Une dalle usée marque l\'entrée du donjon. '
              'L\'air sent la pierre humide.',
          tresors: <Objet>[Catalogue.potionMineure],
        ),
        Salle(
          numero: 2,
          nom: 'La galerie des torches',
          description: 'Des torches brûlent seules le long d\'un couloir trop long.',
          ennemi: Ennemi.gobelin(),
        ),
        // ... quatre autres salles
      ],
    );
  }
}
```

Le constructeur `factory Donjon.parDefaut()` est le **niveau 1 du jeu**, écrit en dur. C'est volontaire : tant que les règles ne sont pas stables, un donjon fixe permet de rejouer exactement la même partie et de comparer les résultats. La génération aléatoire viendra plus tard, en exercice.

`estTermine` illustre `every` du chapitre 14 : « toutes les salles sont-elles nettoyées ? ». En une ligne, sans boucle ni compteur.

---

## 18.15 — Le service `MoteurCombat`

Le moteur de combat est un **service** : il ne stocke aucun état du jeu, il applique des règles. Sa seule donnée propre est un générateur de nombres aléatoires.

```dart
class MoteurCombat {
  MoteurCombat({Random? hasard}) : _hasard = hasard ?? Random();

  final Random _hasard;

  static const int degatsMinimum = 1;
  static const double chanceCritique = 0.15;
  static const double multiplicateurCritique = 2.0;
  static const double chanceEsquive = 0.08;
  static const double chanceFuite = 0.5;
```

Le paramètre `Random? hasard` peut sembler inutile. Il est en réalité **essentiel pour les tests** : en passant `Random(42)`, on obtient une suite de tirages toujours identique, donc un combat reproductible. Un service qui appelle `Random()` en dur à l'intérieur de ses méthodes est un service intestable.

L'attaque :

```dart
  ResultatAttaque attaquer(Personnage attaquant, Personnage cible) {
    final int attaqueEffective =
        attaquant is Joueur ? attaquant.attaqueTotale : attaquant.attaque;
    final int defenseEffective =
        cible is Joueur ? cible.defenseTotale : cible.defense;

    if (_hasard.nextDouble() < chanceEsquive) {
      return ResultatAttaque(
        attaquant: attaquant.nom,
        cible: cible.nom,
        degats: 0,
        critique: false,
        esquive: true,
      );
    }

    final bool critique = _hasard.nextDouble() < chanceCritique;
    final int degatsCalcules = calculerDegats(
      attaque: attaqueEffective,
      defense: defenseEffective,
      critique: critique,
    );
    final int degatsReels = cible.subirDegats(degatsCalcules);

    if (attaquant is Joueur) {
      attaquant.degatsInfliges += degatsReels;
    }

    return ResultatAttaque(
      attaquant: attaquant.nom,
      cible: cible.nom,
      degats: degatsReels,
      critique: critique,
      esquive: false,
    );
  }
```

Deux points de méthode.

**Le paramètre est un `Personnage`, pas un `Joueur`.** La même méthode sert pour l'attaque du joueur et pour la riposte de l'ennemi. C'est du polymorphisme : `attaquer(joueur, ennemi)` et `attaquer(ennemi, joueur)` empruntent le même code.

**Le test `attaquant is Joueur` promeut le type.** Après ce test, Dart sait que `attaquant` est un `Joueur` et autorise `attaquant.attaqueTotale`, qui n'existe pas sur `Personnage`. Cela fonctionne parce que `attaquant` est un **paramètre local**, pas un champ de classe.

Le résultat d'une attaque est une petite classe immuable, qui sait raconter ce qui s'est passé :

```dart
class ResultatAttaque {
  const ResultatAttaque({
    required this.attaquant,
    required this.cible,
    required this.degats,
    required this.critique,
    required this.esquive,
  });

  final String attaquant;
  final String cible;
  final int degats;
  final bool critique;
  final bool esquive;

  String get recit {
    if (esquive) return '$cible esquive l\'attaque de $attaquant !';
    if (critique) return 'COUP CRITIQUE ! $attaquant inflige $degats dégâts à $cible.';
    return '$attaquant inflige $degats dégâts à $cible.';
  }
}
```

Pourquoi ne pas afficher directement le texte dans `attaquer` avec un `print` ? Parce qu'un service ne doit **jamais** écrire à l'écran. Il renvoie des données ; la couche console décide de leur présentation. Le jour où vous porterez ce jeu en Flutter, `MoteurCombat` ne changera pas d'une ligne.

---

## 18.16 — Le calcul des dégâts et les coups critiques

Toute la sensation de jeu tient dans cette unique formule. Il faut donc qu'elle soit isolée, lisible et testable.

```dart
  int calculerDegats({
    required int attaque,
    required int defense,
    required bool critique,
  }) {
    int base = attaque - (defense ~/ 2);
    if (base < degatsMinimum) base = degatsMinimum;

    final int variation = _hasard.nextInt(5) - 2; // -2, -1, 0, +1 ou +2
    int degats = base + variation;
    if (degats < degatsMinimum) degats = degatsMinimum;

    if (critique) {
      degats = (degats * multiplicateurCritique).round();
    }
    return degats;
  }
```

Décomposons.

```text
  Étape 1  base       = attaque - (defense ~/ 2)
  Étape 2  plancher   = max(base, 1)
  Étape 3  variation  = hasard entre -2 et +2
  Étape 4  plancher   = max(base + variation, 1)
  Étape 5  critique   = x2 si le tirage l'a décidé
```

**Pourquoi `defense ~/ 2` et non `defense` ?** Parce qu'une défense soustraite entièrement rendrait les ennemis très défensifs invulnérables. En divisant par deux, l'armure réduit les dégâts sans jamais les annuler.

**Pourquoi `~/` et non `/` ?** Rappel du chapitre 03 : `/` renvoie un `double`, `~/` renvoie un `int`. Des points de vie sont des entiers ; on ne veut pas de `12.5` dégâts.

**Pourquoi un plancher à 1 ?** Un combat où les deux camps s'infligent zéro dégât est une boucle infinie. Le plancher garantit qu'un combat se termine toujours.

**Pourquoi une variation aléatoire ?** Sans elle, chaque combat contre un gobelin se déroulerait exactement à l'identique. La variation `-2..+2` suffit à rendre chaque partie différente sans casser l'équilibrage.

Exemple concret, joueur (attaque 18 avec l'épée) contre gobelin (défense 2) :

```text
  base        = 18 - (2 ~/ 2) = 18 - 1 = 17
  variation   = -2 .. +2
  dégâts      = 15 à 19
  si critique = 30 à 38
```

Et gobelin (attaque 8) contre joueur (défense 5) :

```text
  base        = 8 - (5 ~/ 2) = 8 - 2 = 6
  dégâts      = 4 à 8
  si critique = 8 à 16
```

Programme de vérification statistique, exécutable tel quel :

```dart
void main() {
  final MoteurCombat moteur = MoteurCombat(hasard: Random(2024));

  int total = 0;
  int critiques = 0;
  const int tirages = 1000;

  for (int i = 0; i < tirages; i++) {
    final bool critique = i % 100 < 15; // 15 % de critiques simulés
    if (critique) critiques++;
    total += moteur.calculerDegats(attaque: 18, defense: 2, critique: critique);
  }

  print('Dégâts moyens sur $tirages coups : ${(total / tirages).toStringAsFixed(1)}');
  print('Coups critiques simulés          : $critiques');
  print('Dégâts minimum garantis          : ${MoteurCombat.degatsMinimum}');
}
```

**Résultat (l'ordre de grandeur est stable, la valeur exacte dépend de la graine) :**

```text
Dégâts moyens sur 1000 coups : 19.5
Coups critiques simulés          : 150
Dégâts minimum garantis          : 1
```

Cas extrême à vérifier absolument : une défense énorme.

```dart
void main() {
  final MoteurCombat moteur = MoteurCombat(hasard: Random(7));
  for (int i = 0; i < 5; i++) {
    print(moteur.calculerDegats(attaque: 5, defense: 1000, critique: false));
  }
}
```

**Résultat :**

```text
1
1
1
1
1
```

Le plancher tient. C'est ce que testera le test unitaire numéro 3.

---

## 18.17 — La montée de niveau

La progression suit une règle volontairement simple : le niveau `n` demande `n × 100` points d'expérience pour passer au niveau suivant.

```text
  niveau 1 -> 2 :  100 xp
  niveau 2 -> 3 :  200 xp
  niveau 3 -> 4 :  300 xp
  niveau 4 -> 5 :  400 xp
```

Le code, dans `Joueur` :

```dart
  int get experienceRequise => niveau * 100;

  List<String> gagnerExperience(int points) {
    final List<String> journal = <String>[];
    if (points <= 0) return journal;

    experience += points;
    journal.add('$nom gagne $points points d\'expérience.');

    while (experience >= experienceRequise) {
      experience -= experienceRequise;
      journal.addAll(_monterDeNiveau());
    }
    return journal;
  }

  List<String> _monterDeNiveau() {
    niveau++;
    augmenterPointsDeVieMax(20);
    attaque += 3;
    defense += 2;
    return <String>[
      'NIVEAU $niveau ATTEINT !',
      'PV max +20 ($pointsDeVieMax), attaque +3 ($attaque), défense +2 ($defense).',
    ];
  }
```

Trois détails de conception importants.

**Pourquoi un `while` et non un `if` ?** Parce que le boss donne 400 points d'expérience d'un seul coup. Avec un `if`, le joueur monterait d'un seul niveau et perdrait le reste. Le `while` enchaîne autant de niveaux que nécessaire.

**Pourquoi `experience -= experienceRequise` et non `experience = 0` ?** Pour ne pas gaspiller le surplus. Un joueur à 95 xp qui en gagne 70 doit se retrouver niveau 2 avec 65 xp d'avance, pas à zéro.

**Pourquoi renvoyer une `List<String>` au lieu d'afficher ?** Même raison qu'au 18.15 : le modèle ne parle pas à l'écran. Il renvoie ce qu'il y aurait à dire, l'appelant décide.

Attention au piège du `while` : la condition `experience >= experienceRequise` utilise un getter qui **change** à chaque tour de boucle, puisque `niveau` augmente. C'est voulu, et c'est ce qui fait que la boucle se termine.

Vérification complète :

```dart
void main() {
  final Joueur alex = Joueur(nom: 'Alex');
  print('Départ : niveau ${alex.niveau}, ${alex.experience} xp, '
        '${alex.pointsDeVieMax} PV max, attaque ${alex.attaque}');

  for (final int gain in <int>[40, 70, 130, 70, 400]) {
    print('');
    for (final String ligne in alex.gagnerExperience(gain)) {
      print(ligne);
    }
    print('-> niveau ${alex.niveau}, ${alex.experience}/${alex.experienceRequise} xp');
  }
}
```

**Résultat :**

```text
Départ : niveau 1, 0 xp, 100 PV max, attaque 12

Alex gagne 40 points d'expérience.
-> niveau 1, 40/100 xp

Alex gagne 70 points d'expérience.
NIVEAU 2 ATTEINT !
PV max +20 (120), attaque +3 (15), défense +2 (7).
-> niveau 2, 10/200 xp

Alex gagne 130 points d'expérience.
-> niveau 2, 140/200 xp

Alex gagne 70 points d'expérience.
NIVEAU 3 ATTEINT !
PV max +20 (140), attaque +3 (18), défense +2 (9).
-> niveau 3, 10/300 xp

Alex gagne 400 points d'expérience.
NIVEAU 4 ATTEINT !
PV max +20 (160), attaque +3 (21), défense +2 (11).
-> niveau 4, 110/400 xp
```

Suivez la dernière ligne : 10 + 400 = 410, on retire les 300 du niveau 3, il reste 110, ce qui est insuffisant pour les 400 du niveau 4. La boucle s'arrête. Le calcul est juste.

---

## 18.18 — Les exceptions personnalisées du projet

Le chapitre 13 vous a appris à écrire `throw`, `try`, `on` et `catch`. Reste une question de conception : **quelles** exceptions créer ?

La réponse tient en une règle :

> Créez une exception métier quand l'appelant doit pouvoir **réagir différemment** selon la cause de l'échec.

Dans ce projet, trois situations remplissent ce critère, plus une classe de base commune.

```dart
abstract class ErreurDeJeu implements Exception {
  const ErreurDeJeu(this.message);

  final String message;

  @override
  String toString() => '$runtimeType : $message';
}
```

Pourquoi `implements Exception` et non `extends Exception` ? Parce que `Exception` est une interface en Dart, sans implémentation à réutiliser. On l'implémente pour signaler l'intention ; on hérite du reste de notre propre classe de base.

### `InventairePleinException`

```dart
class InventairePleinException extends ErreurDeJeu {
  InventairePleinException(this.objetRefuse, this.capacite)
      : super('impossible de ramasser "$objetRefuse" : '
            'le sac est plein ($capacite emplacements).');

  final String objetRefuse;
  final int capacite;
}
```

Elle porte **des données**, pas seulement un texte : le nom de l'objet refusé et la capacité. L'interface pourra ainsi proposer « voulez-vous jeter quelque chose pour ramasser X ? ».

### `SauvegardeCorrompueException`

```dart
class SauvegardeCorrompueException extends ErreurDeJeu {
  SauvegardeCorrompueException(this.raison, {this.chemin})
      : super('sauvegarde inutilisable ($raison)'
            '${chemin == null ? '' : ' — fichier : $chemin'}');

  final String raison;
  final String? chemin;
}
```

Elle sera levée dans quatre cas distincts : fichier absent, JSON syntaxiquement invalide, version de format inconnue, champ obligatoire manquant. Dans les quatre cas, la réaction du jeu est la même : refuser le chargement et proposer une nouvelle partie. C'est pourquoi une seule classe suffit, avec une `raison` variable.

### `ObjetIntrouvableException` et `ActionImpossibleException`

```dart
class ObjetIntrouvableException extends ErreurDeJeu {
  ObjetIntrouvableException(this.reference)
      : super('aucun objet ne correspond à "$reference".');

  final String reference;
}

class ActionImpossibleException extends ErreurDeJeu {
  ActionImpossibleException(String raison) : super(raison);
}
```

### Le bénéfice : un `catch` sélectif

Grâce à la classe de base commune, la boucle de jeu peut écrire :

```dart
try {
  joueur.ramasser(objet);
} on InventairePleinException catch (erreur) {
  console.ecrire('Sac plein ! ${erreur.objetRefuse} reste au sol.');
} on ErreurDeJeu catch (erreur) {
  console.ecrire('Action refusée : ${erreur.message}');
}
```

Le premier `on` traite un cas précis avec un message adapté. Le second rattrape **toutes** les autres erreurs métier du projet. Et surtout, aucun `catch` nu : une erreur de programmation (une `RangeError`, un dépassement de pile) ne sera pas avalée silencieusement.

Démonstration :

```dart
void main() {
  final Inventaire sac = Inventaire(capacite: 2);

  for (int i = 1; i <= 3; i++) {
    try {
      sac.ajouter(Catalogue.potionMineure);
      print('Objet $i ajouté. Taille : ${sac.taille}');
    } on InventairePleinException catch (erreur) {
      print('Refusé : ${erreur.message}');
      print('Objet concerné : ${erreur.objetRefuse}');
    }
  }
}
```

**Résultat :**

```text
Objet 1 ajouté. Taille : 1
Objet 2 ajouté. Taille : 2
Refusé : impossible de ramasser "Potion mineure" : le sac est plein (2 emplacements).
Objet concerné : Potion mineure
```

---

## 18.19 — `fromJson` et `toJson` de chaque modèle

Le chapitre 17 a posé les deux règles. Les voici appliquées à un graphe d'objets complet :

- `toJson()` est une **méthode d'instance** qui renvoie une `Map<String, dynamic>` ;
- `fromJson(...)` est un **constructeur `factory`** qui prend une `Map<String, dynamic>`.

### Le principe de composition

La sérialisation est **récursive**. Chaque objet sérialise ses enfants :

```text
  Partie.toJson()
    ├── joueur.toJson()
    │     ├── inventaire.toJson()
    │     │     └── [ objet.toJson(), objet.toJson(), ... ]
    │     ├── armeEquipee?.toJson()
    │     └── armureEquipee?.toJson()
    └── donjon.toJson()
          └── [ salle.toJson(), ... ]
                 ├── ennemi?.toJson()
                 └── [ objet.toJson(), ... ]
```

Chaque classe ne connaît que ses voisins directs. Personne n'a besoin d'une vue globale.

### Trois fonctions de lecture défensive

Une sauvegarde peut être modifiée à la main par un joueur curieux. On ne fait donc jamais confiance au contenu. Ces trois fonctions vivent dans `lib/utils/outils.dart` :

```dart
int entierDepuis(Map<String, dynamic> json, String cle, {int parDefaut = 0}) {
  final Object? valeur = json[cle];
  if (valeur is int) return valeur;
  if (valeur is num) return valeur.toInt();
  if (valeur is String) {
    final int? converti = int.tryParse(valeur);
    if (converti != null) return converti;
  }
  return parDefaut;
}

String texteDepuis(Map<String, dynamic> json, String cle, {String parDefaut = ''}) {
  final Object? valeur = json[cle];
  return valeur is String ? valeur : parDefaut;
}

bool booleenDepuis(Map<String, dynamic> json, String cle, {bool parDefaut = false}) {
  final Object? valeur = json[cle];
  return valeur is bool ? valeur : parDefaut;
}
```

Elles ne lèvent jamais d'exception : une valeur absente ou mal typée est remplacée par une valeur par défaut. On réserve `SauvegardeCorrompueException` aux champs **sans lesquels l'objet n'a aucun sens** : un joueur sans nom, un inventaire sans liste d'objets.

### `Objet`

```dart
  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'type': type.name,
        'rarete': rarete.name,
        'puissance': puissance,
        'valeur': valeur,
        'description': description,
      };

  factory Objet.fromJson(Map<String, dynamic> json) {
    final Object? nom = json['nom'];
    if (nom is! String) {
      throw SauvegardeCorrompueException('un objet n\'a pas de champ "nom"');
    }
    final TypeObjet? type = TypeObjet.parNom(json['type']);
    if (type == null) {
      throw SauvegardeCorrompueException('type d\'objet inconnu : ${json['type']}');
    }
    final Rarete? rarete = Rarete.parNom(json['rarete']);
    if (rarete == null) {
      throw SauvegardeCorrompueException('rareté inconnue : ${json['rarete']}');
    }
    return Objet(
      nom: nom,
      type: type,
      rarete: rarete,
      puissance: entierDepuis(json, 'puissance'),
      valeur: entierDepuis(json, 'valeur'),
      description: texteDepuis(json, 'description'),
    );
  }
```

On écrit `type.name` (`'potion'`) et non `type.libelle` (`'Potion'`). Le `name` est un identifiant technique stable ; le libellé est de l'affichage, susceptible d'être traduit demain. **Ne sérialisez jamais de l'affichage.**

### `Joueur`

```dart
  @override
  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'pointsDeVieMax': pointsDeVieMax,
        'pointsDeVie': pointsDeVie,
        'attaque': attaque,
        'defense': defense,
        'niveau': niveau,
        'experience': experience,
        'or': or,
        'salleCourante': salleCourante,
        'ennemisVaincus': ennemisVaincus,
        'degatsInfliges': degatsInfliges,
        'inventaire': inventaire.toJson(),
        'arme': armeEquipee?.toJson(),
        'armure': armureEquipee?.toJson(),
      };
```

Remarquez `armeEquipee?.toJson()`. Si le joueur n'a pas d'arme, la valeur écrite est `null`, ce qui est parfaitement valide en JSON. À la relecture, on teste le type :

```dart
    final Object? armeJson = json['arme'];
    final Objet? arme =
        armeJson is Map<String, dynamic> ? Objet.fromJson(armeJson) : null;
```

Un seul test, `is Map<String, dynamic>`, couvre à la fois le cas « absent », le cas « null » et le cas « mal formé ». Aucun `!` n'est nécessaire.

### Exemple de fichier produit

```text
{
  "version": 1,
  "date": "2026-08-08T14:32:07.412",
  "partie": {
    "tourDeJeu": 12,
    "terminee": false,
    "victoire": false,
    "joueur": {
      "nom": "Alex",
      "pointsDeVieMax": 120,
      "pointsDeVie": 88,
      "attaque": 15,
      "defense": 7,
      "niveau": 2,
      "experience": 10,
      "or": 28,
      "salleCourante": 2,
      "ennemisVaincus": 1,
      "degatsInfliges": 34,
      "inventaire": {
        "capacite": 10,
        "objets": [
          {
            "nom": "Potion mineure",
            "type": "potion",
            "rarete": "commune",
            "puissance": 25,
            "valeur": 10,
            "description": "Un flacon tiède au goût de fer."
          }
        ]
      },
      "arme": {
        "nom": "Épée rouillée",
        "type": "arme",
        "rarete": "commune",
        "puissance": 6,
        "valeur": 15,
        "description": "Elle a connu de meilleurs jours."
      },
      "armure": null
    },
    "donjon": {
      "nom": "Le Donjon de Kraghar",
      "salles": [ "..." ]
    }
  }
}
```

### Le test qui doit toujours passer

Le test le plus utile sur une sérialisation est l'**aller-retour** : sérialiser, désérialiser, et vérifier qu'on retrouve les mêmes valeurs.

```dart
void main() {
  final Joueur avant = Joueur(nom: 'Alex', niveau: 3, or: 250)
    ..armeEquipee = Catalogue.lameDeFeu
    ..subirDegats(37);

  final Map<String, dynamic> json = avant.toJson();
  final Joueur apres = Joueur.fromJson(json);

  print('nom      : ${avant.nom} -> ${apres.nom}');
  print('niveau   : ${avant.niveau} -> ${apres.niveau}');
  print('PV       : ${avant.pointsDeVie} -> ${apres.pointsDeVie}');
  print('or       : ${avant.or} -> ${apres.or}');
  print('arme     : ${avant.armeEquipee?.nom} -> ${apres.armeEquipee?.nom}');
  print('attaque totale identique ? '
        '${avant.attaqueTotale == apres.attaqueTotale}');
}
```

**Résultat :**

```text
nom      : Alex -> Alex
niveau   : 3 -> 3
PV       : 63 -> 63
or       : 250 -> 250
arme     : Lame de feu -> Lame de feu
attaque totale identique ? true
```

---

## 18.20 — Le service `SauvegardeService`

Ce service est le seul du projet à toucher le disque, et donc le seul à être **asynchrone**. C'est l'occasion d'appliquer le chapitre 15.

```dart
class SauvegardeService {
  SauvegardeService({this.chemin = 'sauvegardes/partie.json'});

  final String chemin;

  static const int versionFormat = 1;

  Future<bool> existe() => File(chemin).exists();

  Future<void> sauvegarder(Partie partie) async {
    final File fichier = File(chemin);
    await fichier.parent.create(recursive: true);

    final Map<String, dynamic> donnees = <String, dynamic>{
      'version': versionFormat,
      'date': DateTime.now().toIso8601String(),
      'partie': partie.toJson(),
    };

    const JsonEncoder encodeur = JsonEncoder.withIndent('  ');
    await fichier.writeAsString(encodeur.convert(donnees));
  }

  Future<Partie> charger() async {
    final File fichier = File(chemin);

    if (!await fichier.exists()) {
      throw SauvegardeCorrompueException('fichier introuvable', chemin: chemin);
    }

    final String contenu = await fichier.readAsString();

    Object? brut;
    try {
      brut = jsonDecode(contenu);
    } on FormatException catch (erreur) {
      throw SauvegardeCorrompueException(
        'JSON illisible (${erreur.message})',
        chemin: chemin,
      );
    }

    if (brut is! Map<String, dynamic>) {
      throw SauvegardeCorrompueException(
        'la racine du fichier n\'est pas un objet JSON',
        chemin: chemin,
      );
    }

    final Object? version = brut['version'];
    if (version != versionFormat) {
      throw SauvegardeCorrompueException(
        'version $version incompatible (attendu $versionFormat)',
        chemin: chemin,
      );
    }

    final Object? partieJson = brut['partie'];
    if (partieJson is! Map<String, dynamic>) {
      throw SauvegardeCorrompueException('bloc "partie" absent', chemin: chemin);
    }

    return Partie.fromJson(partieJson);
  }

  Future<void> supprimer() async {
    final File fichier = File(chemin);
    if (await fichier.exists()) {
      await fichier.delete();
    }
  }
}
```

Détaillons les points clés.

**`await fichier.parent.create(recursive: true)`** crée le dossier `sauvegardes/` s'il n'existe pas. Sans cette ligne, la toute première sauvegarde échoue avec `FileSystemException: Cannot open file`. C'est l'erreur numéro un des débutants sur ce projet.

**`JsonEncoder.withIndent('  ')`** produit un JSON indenté, lisible par un humain. En production on écrirait `jsonEncode(donnees)`, plus compact. Pendant l'apprentissage, un fichier lisible vaut de l'or pour déboguer.

**Le champ `version`** est une assurance pour l'avenir. Le jour où vous ajouterez des sorts au joueur, les anciennes sauvegardes deviendront incompatibles. Avec ce champ, le jeu le détecte et refuse proprement, au lieu de charger une partie à moitié cassée.

**`on FormatException` cible précisément.** `jsonDecode` lève une `FormatException` sur du texte invalide. On la rattrape, on la **traduit** en exception métier, et on la relance. C'est le patron « traduction d'exception » : les couches hautes du jeu ne connaissent que `SauvegardeCorrompueException`, jamais les détails de `dart:convert`.

**Toutes les méthodes renvoient un `Future`.** Écrire sur disque prend du temps. En bloquant, on figerait l'application ; en Flutter, cela gèlerait l'interface. L'habitude se prend maintenant.

Test manuel des quatre cas d'échec :

```dart
Future<void> main() async {
  final SauvegardeService service =
      SauvegardeService(chemin: 'sauvegardes/test.json');

  Future<void> essayer(String titre) async {
    try {
      await service.charger();
      print('$titre : chargement réussi');
    } on SauvegardeCorrompueException catch (erreur) {
      print('$titre : ${erreur.message}');
    }
  }

  await service.supprimer();
  await essayer('Fichier absent    ');

  await File('sauvegardes/test.json').writeAsString('ceci n\'est pas du JSON');
  await essayer('JSON invalide     ');

  await File('sauvegardes/test.json').writeAsString('{"version": 99}');
  await essayer('Mauvaise version  ');

  await File('sauvegardes/test.json').writeAsString('{"version": 1}');
  await essayer('Bloc partie absent');
}
```

**Résultat :**

```text
Fichier absent     : sauvegarde inutilisable (fichier introuvable) — fichier : sauvegardes/test.json
JSON invalide      : sauvegarde inutilisable (JSON illisible (Unexpected character)) — fichier : sauvegardes/test.json
Mauvaise version   : sauvegarde inutilisable (version 99 incompatible (attendu 1)) — fichier : sauvegardes/test.json
Bloc partie absent : sauvegarde inutilisable (bloc "partie" absent) — fichier : sauvegardes/test.json
```

Quatre pannes différentes, quatre messages clairs, zéro plantage.

---

## 18.21 — Le menu console et la lecture des entrées

Toute interaction clavier passe par une seule classe, `Console`. Il y a deux raisons à cela : on n'écrit le code de lecture qu'une fois, et le jour où l'on remplace le terminal par une interface Flutter, on ne change qu'un fichier.

```dart
class Console {
  const Console();

  void ecrire(String texte) => stdout.writeln(texte);

  void sauterLigne() => stdout.writeln('');

  void separateur() => stdout.writeln('-' * 56);

  void titre(String texte) {
    final String ligne = '=' * (texte.length + 4);
    stdout.writeln(ligne);
    stdout.writeln('  $texte');
    stdout.writeln(ligne);
  }

  String? lireLigne(String invite) {
    stdout.write(invite);
    final String? saisie = stdin.readLineSync();
    return saisie?.trim();
  }
}
```

Trois remarques.

**`stdout.write` sans `ln` pour l'invite.** Le curseur reste sur la même ligne, ce qui donne `Votre choix > ` suivi de la frappe du joueur. Avec `print`, l'invite et la réponse seraient sur deux lignes.

**`stdin.readLineSync()` renvoie `String?`.** Ce n'est pas un caprice : la valeur est `null` quand l'entrée standard est **fermée** (fin de fichier, `Ctrl+D`, ou exécution dans un script sans clavier). Un jeu qui ignore ce `null` boucle à l'infini.

**`saisie?.trim()`** enlève les espaces et le retour chariot. Sans lui, un joueur qui tape « 1 » avec un espace obtiendrait une saisie invalide, ce qui est incompréhensible pour lui.

Le menu s'affiche avec une simple boucle sur une liste :

```dart
  void afficherMenu(List<String> options) {
    for (int i = 0; i < options.length; i++) {
      ecrire('${i + 1}. ${options[i]}');
    }
    ecrire('0. Retour');
  }
```

---

## 18.22 — La validation des saisies

Le joueur tapera « deux », « 42 », « », ou appuiera sur Entrée par erreur. Aucune de ces situations ne doit arrêter le programme.

L'outil central est `int.tryParse`, vu au chapitre 02 :

| Expression | Résultat |
| --- | --- |
| `int.parse('abc')` | lève une `FormatException` |
| `int.tryParse('abc')` | renvoie `null` |

`tryParse` est presque toujours le bon choix pour une saisie utilisateur : une frappe erronée est un événement **normal**, pas une anomalie.

```dart
  int? lireEntier(String invite, {required int minimum, required int maximum}) {
    for (int essai = 0; essai < 5; essai++) {
      final String? saisie = lireLigne(invite);

      if (saisie == null) return null; // entrée fermée

      final int? valeur = int.tryParse(saisie);
      if (valeur == null) {
        ecrire('Saisie invalide : "$saisie" n\'est pas un nombre entier.');
        continue;
      }
      if (valeur < minimum || valeur > maximum) {
        ecrire('Saisie hors limites : attendu entre $minimum et $maximum.');
        continue;
      }
      return valeur;
    }
    ecrire('Trop de saisies invalides, l\'action est annulée.');
    return null;
  }

  bool confirmer(String question) {
    for (int essai = 0; essai < 3; essai++) {
      final String? reponse = lireLigne('$question (o/n) > ');
      if (reponse == null) return false;
      final String r = reponse.toLowerCase();
      if (r == 'o' || r == 'oui') return true;
      if (r == 'n' || r == 'non') return false;
      ecrire('Répondez par "o" ou par "n".');
    }
    return false;
  }

  String lireNom(String invite, {String parDefaut = 'Aventurier'}) {
    final String? saisie = lireLigne(invite);
    if (saisie == null || saisie.isEmpty) return parDefaut;
    if (saisie.length > 20) return saisie.substring(0, 20);
    return saisie;
  }
```

Quatre garde-fous en une seule méthode `lireEntier` :

```text
  1. entrée fermée (null)        -> on renvoie null, l'appelant quitte proprement
  2. texte non numérique         -> message + nouvelle tentative
  3. nombre hors bornes          -> message + nouvelle tentative
  4. cinq échecs de suite        -> on abandonne, on ne boucle pas à l'infini
```

Le compteur `essai < 5` est ce qui distingue un programme robuste d'un programme qui se fige. Sans lui, une entrée standard fermée qui renvoie toujours la même valeur invalide produirait une boucle infinie consommant 100 % du processeur.

Simulation du comportement :

```text
Votre choix > deux
Saisie invalide : "deux" n'est pas un nombre entier.
Votre choix > 
Saisie invalide : "" n'est pas un nombre entier.
Votre choix > 42
Saisie hors limites : attendu entre 0 et 6.
Votre choix > -3
Saisie hors limites : attendu entre 0 et 6.
Votre choix > 2
```

Le programme n'a jamais planté, et le joueur a compris son erreur à chaque fois.

---

## 18.23 — La boucle de jeu principale

Toute la structure du jeu tient dans trois boucles imbriquées :

```text
  boucle 1 : MENU PRINCIPAL        (nouvelle partie / charger / quitter)
      |
      +-- boucle 2 : EXPLORATION   (tant que la partie n'est pas terminée)
              |
              +-- boucle 3 : COMBAT (tant que les deux camps sont vivants)
```

Chaque boucle a **une seule condition de sortie**, et chacune est écrite dans sa propre méthode. C'est ce qui rend le code lisible malgré la complexité du jeu.

### Boucle 1 — le menu principal

```dart
  Future<void> demarrer() async {
    console.titre('DONJON DE DART');

    bool actif = true;
    while (actif) {
      console.sauterLigne();
      console.ecrire('1. Nouvelle partie');
      console.ecrire('2. Charger la partie');
      console.ecrire('0. Quitter');

      final int? choix = console.lireEntier('Votre choix > ', minimum: 0, maximum: 2);

      if (choix == null || choix == 0) {
        actif = false;
        continue;
      }
      if (choix == 1) {
        await _boucleDeJeu(_creerPartie());
      } else {
        final Partie? chargee = await _charger();
        if (chargee != null) await _boucleDeJeu(chargee);
      }
    }
    console.ecrire('Merci d\'avoir joué.');
  }
```

### Boucle 2 — l'exploration

```dart
  Future<void> _boucleDeJeu(Partie partie) async {
    while (!partie.terminee) {
      partie.tourDeJeu++;
      final Salle salle = partie.salleCourante;

      _afficherEnTete(partie, salle);

      final int? choix = console.lireEntier('Votre choix > ', minimum: 0, maximum: 6);
      if (choix == null) {
        partie.terminee = true;
        continue;
      }

      try {
        switch (choix) {
          case 1:
            _explorer(partie, salle);
            break;
          case 2:
            _combattre(partie, salle);
            break;
          case 3:
            _ouvrirInventaire(partie);
            break;
          case 4:
            _avancer(partie);
            break;
          case 5:
            await _sauvegarder(partie);
            break;
          case 6:
            console.ecrire(Statistiques(partie).rapport);
            break;
          case 0:
            if (console.confirmer('Quitter la partie en cours ?')) {
              partie.terminee = true;
            }
            break;
        }
      } on ErreurDeJeu catch (erreur) {
        console.ecrire('Action impossible : ${erreur.message}');
      }

      if (partie.joueur.estMort) {
        partie.terminee = true;
        partie.victoire = false;
      }
    }
    _afficherFin(partie);
  }
```

Trois points de conception.

**Le `try` entoure tout le `switch`.** N'importe quelle action peut lever une `ErreurDeJeu` ; une seule capture suffit à empêcher que le jeu s'arrête. Chaque action n'a donc pas besoin de son propre `try`.

**La mort du joueur est vérifiée une seule fois, en fin de tour.** Plutôt que de tester `estMort` à dix endroits, on centralise. Le code est plus court et il est impossible d'oublier un cas.

**`partie.terminee` est la seule condition de sortie.** Toutes les fins de partie (mort, victoire, abandon) passent par ce booléen. Il n'y a aucun `return` au milieu de la boucle, aucun `break` caché : on peut lire la méthode de haut en bas et savoir exactement quand elle s'arrête.

### Boucle 3 — le combat

```dart
  void _combattre(Partie partie, Salle salle) {
    final Ennemi? occupant = salle.ennemi;
    if (occupant == null || occupant.estMort) {
      console.ecrire('Il n\'y a plus rien à combattre ici.');
      return;
    }

    console.ecrire('${occupant.nom} (${occupant.pointsDeVie} PV) bloque le passage !');

    bool combatEnCours = true;
    int tour = 0;

    while (combatEnCours) {
      tour++;
      console.sauterLigne();
      console.ecrire('--- Tour $tour ---');
      console.ecrire('${partie.joueur.nom.padRight(8)} ${partie.joueur.barreVie} '
          '${partie.joueur.pointsDeVie}/${partie.joueur.pointsDeVieMax}');
      console.ecrire('${occupant.nom.padRight(8)} ${occupant.barreVie} '
          '${occupant.pointsDeVie}/${occupant.pointsDeVieMax}');

      console.ecrire('1. Attaquer');
      console.ecrire('2. Boire une potion');
      console.ecrire('3. Fuir');

      final int? choix = console.lireEntier('Votre choix > ', minimum: 1, maximum: 3);
      if (choix == null) return;

      final ActionCombat action = ActionCombat.values[choix - 1];
      final ResultatTour resultat = moteur.jouerTour(partie.joueur, occupant, action);

      for (final String ligne in resultat.journal) {
        console.ecrire(ligne);
      }
      combatEnCours = !resultat.combatTermine;
    }

    if (occupant.estMort) {
      _recompenser(partie, salle, occupant);
    }
  }
```

`ActionCombat.values[choix - 1]` traduit le numéro tapé en valeur d'enum. Comme `lireEntier` garantit `1 <= choix <= 3`, l'index est toujours valide : on ne peut pas sortir du tableau.

Et le cœur des règles, dans `MoteurCombat` :

```dart
  ResultatTour jouerTour(Joueur joueur, Ennemi ennemi, ActionCombat action) {
    final List<String> journal = <String>[];
    bool fuiteReussie = false;

    switch (action) {
      case ActionCombat.attaquer:
        journal.add(attaquer(joueur, ennemi).recit);
        break;

      case ActionCombat.potion:
        final Objet? potion = joueur.meilleurePotion();
        if (potion == null) {
          journal.add('Aucune potion dans le sac. Le tour est perdu.');
        } else {
          journal.add(joueur.boirePotion(potion));
        }
        break;

      case ActionCombat.fuir:
        if (ennemi.estBoss) {
          journal.add('Impossible de fuir devant ${ennemi.nom} !');
        } else if (_hasard.nextDouble() < chanceFuite) {
          journal.add('${joueur.nom} parvient à fuir le combat.');
          fuiteReussie = true;
        } else {
          journal.add('La fuite échoue !');
        }
        break;
    }

    if (fuiteReussie) {
      return ResultatTour(journal: journal, combatTermine: true, fuiteReussie: true);
    }

    if (ennemi.estMort) {
      journal.add('${ennemi.nom} est vaincu !');
      return ResultatTour(journal: journal, combatTermine: true, fuiteReussie: false);
    }

    journal.add(attaquer(ennemi, joueur).recit);

    if (joueur.estMort) {
      journal.add('${joueur.nom} s\'effondre...');
      return ResultatTour(journal: journal, combatTermine: true, fuiteReussie: false);
    }

    return ResultatTour(journal: journal, combatTermine: false, fuiteReussie: false);
  }
```

L'ordre des vérifications est capital, et c'est un bug classique :

```text
  1. le joueur agit
  2. l'ennemi est-il mort ?   -> OUI : le combat s'arrête ICI
  3. l'ennemi riposte
  4. le joueur est-il mort ?  -> OUI : le combat s'arrête
```

Si vous inversez les étapes 2 et 3, un ennemi tué à zéro point de vie continue de frapper. C'est une injustice que les joueurs remarquent immédiatement.

---

## 18.24 — Le fichier `bin/main.dart`

Le point d'entrée doit être le fichier le plus **court** du projet.

```dart
import 'package:donjon_de_dart/jeu.dart';

Future<void> main(List<String> arguments) async {
  final String chemin =
      arguments.isNotEmpty ? arguments.first : 'sauvegardes/partie.json';

  final Jeu jeu = Jeu(cheminSauvegarde: chemin);
  await jeu.demarrer();
}
```

Quatre lignes utiles. C'est tout ce qu'un point d'entrée doit contenir : construire l'objet principal et lui donner la main.

Un `main` de 400 lignes est le symptôme d'un projet non structuré. Tant que vous n'y touchez pas, vous savez que votre logique est bien rangée dans `lib/`.

`main` est `Future<void>` parce que `demarrer()` est asynchrone (à cause de la sauvegarde). Le `await` garantit que le programme ne se termine pas avant la fin de la partie.

Le paramètre `arguments` permet de jouer plusieurs parties en parallèle :

```bash
dart run bin/main.dart
dart run bin/main.dart sauvegardes/partie2.json
```

---

## 18.25 — Les statistiques de fin de partie

C'est le moment de rentabiliser le chapitre 14. Toutes les statistiques se calculent avec `where`, `map` et `fold`, sans une seule boucle `for`.

```dart
class Statistiques {
  const Statistiques(this.partie);

  final Partie partie;

  Joueur get _joueur => partie.joueur;
  List<Salle> get _salles => partie.donjon.salles;

  int get sallesVisitees =>
      _salles.where((Salle salle) => salle.visitee).length;

  int get sallesNettoyees =>
      _salles.where((Salle salle) => salle.nettoyee).length;

  List<String> get ennemisRestants => _salles
      .where((Salle salle) => salle.contientUnEnnemiVivant)
      .map((Salle salle) => salle.ennemi?.nom ?? 'inconnu')
      .toList();

  int get valeurDuSac => _joueur.inventaire.objets
      .fold(0, (int total, Objet objet) => total + objet.valeur);

  int get puissanceEquipee =>
      (_joueur.armeEquipee?.puissanceEffective ?? 0) +
      (_joueur.armureEquipee?.puissanceEffective ?? 0);

  Map<Rarete, int> get repartitionParRarete =>
      _joueur.inventaire.objets.fold(
        <Rarete, int>{},
        (Map<Rarete, int> compteur, Objet objet) {
          compteur[objet.rarete] = (compteur[objet.rarete] ?? 0) + 1;
          return compteur;
        },
      );
```

Concentrons-nous sur `repartitionParRarete`, qui est le `fold` le plus riche du projet.

```text
  valeur initiale : {}                     une Map vide

  objet 1 (commune)   -> {commune: 1}
  objet 2 (rare)      -> {commune: 1, rare: 1}
  objet 3 (commune)   -> {commune: 2, rare: 1}
  objet 4 (légendaire)-> {commune: 2, rare: 1, legendaire: 1}
```

L'accumulateur n'est pas obligé d'être un nombre : ici c'est une `Map`. La ligne clé est :

```dart
compteur[objet.rarete] = (compteur[objet.rarete] ?? 0) + 1;
```

Elle se lit : « le compteur de cette rareté, ou zéro s'il n'existe pas encore, plus un ». Le `??` évite un `if (compteur.containsKey(...))`.

Le rapport final assemble le tout :

```dart
  String get rapport {
    final StringBuffer tampon = StringBuffer();
    final String repartition = repartitionParRarete.entries
        .map((MapEntry<Rarete, int> e) => '${e.key.libelle} ${e.value}')
        .join(', ');

    tampon.writeln('--- Statistiques de fin de partie ---');
    tampon.writeln('Aventurier          : ${_joueur.nom}');
    tampon.writeln('Niveau atteint      : ${_joueur.niveau}');
    tampon.writeln('Salles visitées     : $sallesVisitees / ${_salles.length}');
    tampon.writeln('Salles nettoyées    : $sallesNettoyees / ${_salles.length}');
    tampon.writeln('Ennemis vaincus     : ${_joueur.ennemisVaincus}');
    tampon.writeln('Dégâts infligés     : ${_joueur.degatsInfliges}');
    tampon.writeln('Soins reçus         : ${_joueur.soinsRecus}');
    tampon.writeln('Or amassé           : ${_joueur.or}');
    tampon.writeln('Objets dans le sac  : ${_joueur.inventaire.taille} '
        '(valeur $valeurDuSac pièces)');
    tampon.writeln('Répartition         : '
        '${repartition.isEmpty ? 'sac vide' : repartition}');
    tampon.writeln('Puissance équipée   : $puissanceEquipee');
    tampon.writeln('Tours de jeu        : ${partie.tourDeJeu}');
    return tampon.toString();
  }
}
```

`StringBuffer` est préférable à une concaténation `texte += ...` répétée : chaque `+=` sur un `String` crée une nouvelle chaîne en mémoire, alors que le tampon accumule sans recopier.

**Résultat pour une partie gagnée :**

```text
--- Statistiques de fin de partie ---
Aventurier          : Alex
Niveau atteint      : 4
Salles visitées     : 6 / 6
Salles nettoyées    : 6 / 6
Ennemis vaincus     : 5
Dégâts infligés     : 335
Soins reçus         : 140
Or amassé           : 351
Objets dans le sac  : 4 (valeur 620 pièces)
Répartition         : Commune 1, Rare 1, Épique 1, Légendaire 1
Puissance équipée   : 29
Tours de jeu        : 38
```

---

## 18.26 — Tester le projet

Un projet livrable est un projet testé. Créez `test/donjon_de_dart_test.dart` et lancez :

```bash
dart test
```

Voici cinq tests qui couvrent les cinq règles les plus importantes du jeu.

```dart
import 'dart:convert';
import 'dart:io';
import 'dart:math';

import 'package:donjon_de_dart/exceptions/erreurs_jeu.dart';
import 'package:donjon_de_dart/models/inventaire.dart';
import 'package:donjon_de_dart/models/joueur.dart';
import 'package:donjon_de_dart/models/objet.dart';
import 'package:donjon_de_dart/models/partie.dart';
import 'package:donjon_de_dart/models/donjon.dart';
import 'package:donjon_de_dart/services/moteur_combat.dart';
import 'package:donjon_de_dart/services/sauvegarde_service.dart';
import 'package:test/test.dart';

void main() {
  // TEST 1 — L'inventaire refuse d'accepter plus que sa capacité.
  // Règle métier O5 du cahier des charges. On vérifie deux choses :
  // que l'exception est bien du type attendu, et que la taille du sac
  // n'a PAS changé après le refus (pas d'effet de bord).
  test('un inventaire plein lève InventairePleinException', () {
    final Inventaire sac = Inventaire(capacite: 2);
    sac.ajouter(Catalogue.potionMineure);
    sac.ajouter(Catalogue.potionMajeure);

    expect(sac.estPlein, isTrue);
    expect(
      () => sac.ajouter(Catalogue.epeeRouillee),
      throwsA(isA<InventairePleinException>()),
    );
    expect(sac.taille, equals(2));
  });

  // TEST 2 — La montée de niveau enchaîne plusieurs paliers d'un coup
  // et conserve le surplus d'expérience.
  // C'est le test du 'while' de la section 18.17 : avec un 'if',
  // ce test échouerait au niveau 2 au lieu de 3.
  test('un gros gain d\'expérience fait monter plusieurs niveaux', () {
    final Joueur joueur = Joueur(nom: 'Testeur');

    joueur.gagnerExperience(310); // 100 (niv.2) + 200 (niv.3) + 10 de reste

    expect(joueur.niveau, equals(3));
    expect(joueur.experience, equals(10));
    expect(joueur.pointsDeVieMax, equals(140)); // 100 + 20 + 20
    expect(joueur.attaque, equals(18)); //  12 +  3 +  3
  });

  // TEST 3 — Les dégâts ne descendent jamais sous 1, même contre
  // une défense absurde. Sans ce plancher, un combat pourrait
  // ne jamais se terminer (boucle infinie).
  // On fixe la graine du Random pour rendre le test reproductible.
  test('les dégâts ont toujours un plancher de 1', () {
    final MoteurCombat moteur = MoteurCombat(hasard: Random(42));

    for (int i = 0; i < 100; i++) {
      final int degats =
          moteur.calculerDegats(attaque: 1, defense: 10000, critique: false);
      expect(degats, greaterThanOrEqualTo(MoteurCombat.degatsMinimum));
    }
  });

  // TEST 4 — L'aller-retour JSON conserve l'état du joueur.
  // C'est le test le plus rentable d'une sérialisation : il détecte
  // instantanément tout champ oublié dans toJson ou dans fromJson.
  test('un joueur sérialisé puis désérialisé est identique', () {
    final Joueur avant = Joueur(nom: 'Alex', niveau: 3, or: 250);
    avant.armeEquipee = Catalogue.lameDeFeu;
    avant.inventaire.ajouter(Catalogue.potionMajeure);
    avant.subirDegats(37);

    final Joueur apres = Joueur.fromJson(jsonDecode(jsonEncode(avant.toJson())));

    expect(apres.nom, equals(avant.nom));
    expect(apres.niveau, equals(avant.niveau));
    expect(apres.or, equals(avant.or));
    expect(apres.pointsDeVie, equals(avant.pointsDeVie));
    expect(apres.attaqueTotale, equals(avant.attaqueTotale));
    expect(apres.inventaire.taille, equals(1));
    expect(apres.inventaire.objets.first, equals(Catalogue.potionMajeure));
  });

  // TEST 5 — Un fichier de sauvegarde illisible est refusé proprement.
  // Ce test est asynchrone : il écrit un vrai fichier temporaire,
  // puis nettoie derrière lui dans un bloc 'finally'.
  test('une sauvegarde corrompue lève SauvegardeCorrompueException', () async {
    const String chemin = 'sauvegardes/test_corrompu.json';
    final SauvegardeService service = SauvegardeService(chemin: chemin);
    final File fichier = File(chemin);

    try {
      await fichier.parent.create(recursive: true);
      await fichier.writeAsString('{ ceci n\'est pas du JSON valide');

      expect(
        () => service.charger(),
        throwsA(isA<SauvegardeCorrompueException>()),
      );
    } finally {
      if (await fichier.exists()) await fichier.delete();
    }
  });
}
```

**Résultat :**

```text
$ dart test
00:01 +5: All tests passed!
```

Quelques principes que ces tests illustrent.

| Principe | Où il apparaît |
| --- | --- |
| Un test = une règle métier, pas une méthode | Test 2 teste la progression, pas `gagnerExperience` |
| Tester aussi l'absence d'effet de bord | Test 1 vérifie `sac.taille` après l'échec |
| Neutraliser le hasard par une graine | Test 3, `Random(42)` |
| L'aller-retour est le test roi d'une sérialisation | Test 4 |
| Nettoyer les fichiers créés | Test 5, bloc `finally` |

---

## 18.27 — Améliorations possibles

Voici, honnêtement, ce qui n'est pas parfait dans ce projet. Savoir critiquer son propre code fait partie du métier.

| Faiblesse actuelle | Conséquence | Piste d'amélioration |
| --- | --- | --- |
| Le donjon est écrit en dur dans `Donjon.parDefaut()` | Une seule aventure possible | Charger les salles depuis un fichier JSON de données |
| `Jeu` fait à la fois l'affichage et l'arbitrage | Classe longue, difficile à tester | Extraire une classe `AffichageJeu` |
| Aucune sauvegarde automatique | Une déconnexion perd la partie | Sauvegarder à chaque changement de salle |
| Pas de journal des combats conservé | Impossible de rejouer une partie | Stocker le journal dans `Partie` |
| L'équilibrage est dispersé dans les constructeurs | Difficile à régler finement | Centraliser dans une classe `Equilibrage` de constantes |
| Un seul fichier de sauvegarde par défaut | Pas de parties parallèles | Un dossier avec plusieurs emplacements nommés |
| Les tests ne couvrent pas la boucle de jeu | Un bug d'enchaînement passerait | Injecter une `Console` factice qui rejoue des saisies |

Cette dernière ligne mérite un mot. Comme toutes les entrées et sorties passent par la classe `Console`, vous pouvez en écrire une version de test :

```dart
class ConsoleFactice implements Console {
  ConsoleFactice(this.saisies);

  final List<String> saisies;
  final List<String> sorties = <String>[];
  int _index = 0;

  @override
  void ecrire(String texte) => sorties.add(texte);

  @override
  String? lireLigne(String invite) =>
      _index < saisies.length ? saisies[_index++] : null;

  // ... les autres méthodes de Console
}
```

Une partie entière devient alors testable, sans clavier. C'est le bénéfice concret d'avoir isolé les entrées-sorties dans un service.

---

## 18.28 — Ce que ce projet a mobilisé

Chaque chapitre de la partie 1A est présent dans ce projet. Voici où.

| Ch. | Notion | Où elle sert dans « Donjon de Dart » |
| --- | --- | --- |
| 01 | Structure d'un programme, `main`, `print` | `bin/main.dart`, tous les affichages console |
| 02 | Variables, `final`, `const`, types, `int.tryParse` | Champs `final` des modèles, catalogue `const`, validation des saisies |
| 03 | Opérateurs, `~/`, `%`, comparaisons | `defense ~/ 2` dans le calcul des dégâts, bornage des points de vie |
| 04 | `if`, `else`, `switch` | Menus, choix d'action de combat, `switch` sur `ActionCombat` |
| 05 | `for`, `while`, `do...while` | Boucle de jeu, boucle de combat, boucle de montée de niveau |
| 06 | `List`, `Set`, `Map` | `List<Objet>` de l'inventaire, `List<Salle>` du donjon, `Map<String, dynamic>` du JSON |
| 07 | Fonctions, paramètres nommés, valeurs par défaut | `calculerDegats({required ...})`, `Joueur({int pointsDeVieMax = 100})` |
| 08 | Classes, champs, méthodes, `this` | Les dix classes de `lib/models/` |
| 09 | Constructeurs, nommés, redirigés, `factory` | `Ennemi.gobelin()`, `Ennemi.boss()`, `Donjon.parDefaut()`, tous les `fromJson` |
| 10 | Encapsulation, héritage, polymorphisme | `_objets` privée, `Joueur extends Personnage`, `attaquer(Personnage, Personnage)` |
| 11 | `abstract`, `mixin`, `enum` | `Personnage` abstraite, `mixin Soignable on Personnage`, `TypeObjet` et `Rarete` |
| 12 | Null safety, `?.`, `??`, promotion de type | `armeEquipee?.puissanceEffective ?? 0`, copie locale dans `contientUnEnnemiVivant` |
| 13 | Exceptions, `throw`, `on`, `finally` | `InventairePleinException`, `SauvegardeCorrompueException`, `try` de la boucle de jeu |
| 14 | `where`, `map`, `fold`, `every`, `join` | Toute la classe `Statistiques`, `Donjon.estTermine`, `Inventaire.parType` |
| 15 | `Future`, `async`, `await` | `SauvegardeService.sauvegarder`, `charger`, `Jeu.demarrer` |
| 16 | Projet Dart, `pubspec.yaml`, imports | Arborescence `bin/lib/test`, dépendance `test`, imports `package:` |
| 17 | JSON, `jsonEncode`, `jsonDecode`, modélisation | Les sept couples `toJson` / `fromJson`, le champ `version` |

Si un chapitre de cette liste vous semble flou, c'est le moment de le relire : le projet vous montre exactement à quoi il sert.

---

## 18.29 — Code source complet

Voici l'intégralité du projet, fichier par fichier, dans l'ordre des dépendances : d'abord ce qui ne dépend de rien, ensuite ce qui s'appuie dessus.

Vous pouvez créer les fichiers un à un et lancer `dart analyze` après chacun. L'analyseur vous signalera immédiatement un import manquant ou une méthode oubliée.

Trois précisions avant de lire :

- à l'intérieur de `lib/`, les imports sont **relatifs** ; depuis `bin/` et `test/`, ils passent par `package:donjon_de_dart/...` ;
- le compteur `soinsRecus` du mixin `Soignable` n'est **pas** sauvegardé : c'est une statistique de session, elle repart de zéro au chargement, comme le compteur de temps de jeu d'un jeu commercial ;
- aucun fichier de `lib/models/` n'importe un fichier de `lib/services/`. Si vous vous surprenez à écrire un tel import, votre conception a dérapé.

---

### `pubspec.yaml`

```yaml
name: donjon_de_dart
description: Un jeu de rôle au tour par tour en console, projet final de la partie Dart.
version: 1.0.0
publish_to: none

environment:
  sdk: ^3.5.0

dev_dependencies:
  lints: ^4.0.0
  test: ^1.25.0
```

---

### `analysis_options.yaml`

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    - always_declare_return_types
    - prefer_final_locals
    - unnecessary_this
```

---

### `lib/utils/outils.dart`

```dart
/// Fonctions utilitaires sans état, utilisables partout dans le projet.
/// Aucune d'elles ne connaît le jeu : elles manipulent des nombres,
/// des textes et des Map. C'est ce qui les rend faciles à tester.

/// Ramène [valeur] dans l'intervalle [minimum] .. [maximum].
int borner(int valeur, int minimum, int maximum) {
  if (valeur < minimum) return minimum;
  if (valeur > maximum) return maximum;
  return valeur;
}

/// Construit une barre de vie textuelle : [#######.............]
String barreDeVie(int actuel, int maximum, {int largeur = 20}) {
  if (maximum <= 0) return '[${'.' * largeur}]';
  final int pleins = borner((actuel * largeur / maximum).round(), 0, largeur);
  return '[${'#' * pleins}${'.' * (largeur - pleins)}]';
}

/// Lit un entier dans une Map JSON sans jamais lever d'exception.
int entierDepuis(Map<String, dynamic> json, String cle, {int parDefaut = 0}) {
  final Object? valeur = json[cle];
  if (valeur is int) return valeur;
  if (valeur is num) return valeur.toInt();
  if (valeur is String) {
    final int? converti = int.tryParse(valeur);
    if (converti != null) return converti;
  }
  return parDefaut;
}

/// Lit un texte dans une Map JSON sans jamais lever d'exception.
String texteDepuis(Map<String, dynamic> json, String cle, {String parDefaut = ''}) {
  final Object? valeur = json[cle];
  return valeur is String ? valeur : parDefaut;
}

/// Lit un booléen dans une Map JSON sans jamais lever d'exception.
bool booleenDepuis(Map<String, dynamic> json, String cle, {bool parDefaut = false}) {
  final Object? valeur = json[cle];
  return valeur is bool ? valeur : parDefaut;
}

/// Renvoie la sous-Map située à [cle], ou null si elle est absente
/// ou mal formée. Un seul test couvre les trois cas.
Map<String, dynamic>? sousObjetDepuis(Map<String, dynamic> json, String cle) {
  final Object? valeur = json[cle];
  return valeur is Map<String, dynamic> ? valeur : null;
}

/// Renvoie la liste de sous-Map située à [cle]. Les éléments illisibles
/// sont ignorés silencieusement : une sauvegarde partiellement abîmée
/// vaut mieux qu'un refus total.
List<Map<String, dynamic>> sousListeDepuis(Map<String, dynamic> json, String cle) {
  final Object? valeur = json[cle];
  if (valeur is! List) return <Map<String, dynamic>>[];
  final List<Map<String, dynamic>> resultat = <Map<String, dynamic>>[];
  for (final Object? element in valeur) {
    if (element is Map<String, dynamic>) resultat.add(element);
  }
  return resultat;
}
```

---

### `lib/models/types.dart`

```dart
/// Les deux ensembles fermés de valeurs du jeu.
/// Un enum plutôt qu'un String : impossible d'écrire 'Potiion'.

enum TypeObjet {
  arme('Arme'),
  armure('Armure'),
  potion('Potion'),
  cle('Clé'),
  tresor('Trésor');

  const TypeObjet(this.libelle);

  final String libelle;

  /// Retrouve un TypeObjet à partir de son identifiant technique.
  /// Renvoie null si la valeur est inconnue : c'est l'appelant qui décide.
  static TypeObjet? parNom(Object? nom) {
    for (final TypeObjet type in TypeObjet.values) {
      if (type.name == nom) return type;
    }
    return null;
  }
}

enum Rarete {
  commune('Commune', 1.0),
  rare('Rare', 1.5),
  epique('Épique', 2.0),
  legendaire('Légendaire', 3.0);

  const Rarete(this.libelle, this.multiplicateur);

  final String libelle;
  final double multiplicateur;

  static Rarete? parNom(Object? nom) {
    for (final Rarete rarete in Rarete.values) {
      if (rarete.name == nom) return rarete;
    }
    return null;
  }
}
```

---

### `lib/exceptions/erreurs_jeu.dart`

```dart
/// Toutes les erreurs métier du projet héritent de ErreurDeJeu.
/// Cela permet à la boucle de jeu d'écrire un seul `on ErreurDeJeu`
/// pour rattraper l'ensemble, et des `on ...Exception` plus précis
/// quand elle veut réagir différemment.
abstract class ErreurDeJeu implements Exception {
  const ErreurDeJeu(this.message);

  final String message;

  @override
  String toString() => '$runtimeType : $message';
}

/// Le sac est plein. Porte le nom de l'objet refusé et la capacité,
/// pour que l'interface puisse proposer de faire de la place.
class InventairePleinException extends ErreurDeJeu {
  InventairePleinException(this.objetRefuse, this.capacite)
      : super('impossible de ramasser "$objetRefuse" : '
            'le sac est plein ($capacite emplacements).');

  final String objetRefuse;
  final int capacite;
}

/// Aucun objet ne correspond à la référence demandée.
class ObjetIntrouvableException extends ErreurDeJeu {
  ObjetIntrouvableException(this.reference)
      : super('aucun objet ne correspond à "$reference".');

  final String reference;
}

/// L'action demandée n'a pas de sens dans l'état actuel du jeu.
class ActionImpossibleException extends ErreurDeJeu {
  ActionImpossibleException(String raison) : super(raison);
}

/// La sauvegarde est inutilisable : fichier absent, JSON invalide,
/// version incompatible ou champ obligatoire manquant.
class SauvegardeCorrompueException extends ErreurDeJeu {
  SauvegardeCorrompueException(this.raison, {this.chemin})
      : super('sauvegarde inutilisable ($raison)'
            '${chemin == null ? '' : ' — fichier : $chemin'}');

  final String raison;
  final String? chemin;
}
```

---

### `lib/models/objet.dart`

```dart
import '../exceptions/erreurs_jeu.dart';
import '../utils/outils.dart';
import 'types.dart';

/// Un objet du jeu est une donnée immuable : tous ses champs sont final
/// et son constructeur est const. Deux objets identiques par valeur
/// sont considérés égaux (voir == plus bas).
class Objet {
  const Objet({
    required this.nom,
    required this.type,
    required this.rarete,
    this.puissance = 0,
    this.valeur = 0,
    this.description = '',
  });

  final String nom;
  final TypeObjet type;
  final Rarete rarete;
  final int puissance;
  final int valeur;
  final String description;

  /// La puissance affichée et utilisée en combat : la puissance brute
  /// multipliée par la rareté. Toute la courbe d'équilibrage tient ici.
  int get puissanceEffective => (puissance * rarete.multiplicateur).round();

  bool get estConsommable => type == TypeObjet.potion;
  bool get estEquipable => type == TypeObjet.arme || type == TypeObjet.armure;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'type': type.name,
        'rarete': rarete.name,
        'puissance': puissance,
        'valeur': valeur,
        'description': description,
      };

  factory Objet.fromJson(Map<String, dynamic> json) {
    final Object? nom = json['nom'];
    if (nom is! String) {
      throw SauvegardeCorrompueException('un objet n\'a pas de champ "nom"');
    }
    final TypeObjet? type = TypeObjet.parNom(json['type']);
    if (type == null) {
      throw SauvegardeCorrompueException('type d\'objet inconnu : ${json['type']}');
    }
    final Rarete? rarete = Rarete.parNom(json['rarete']);
    if (rarete == null) {
      throw SauvegardeCorrompueException('rareté inconnue : ${json['rarete']}');
    }
    return Objet(
      nom: nom,
      type: type,
      rarete: rarete,
      puissance: entierDepuis(json, 'puissance'),
      valeur: entierDepuis(json, 'valeur'),
      description: texteDepuis(json, 'description'),
    );
  }

  /// Égalité PAR VALEUR. Indispensable : après un chargement,
  /// les objets sont de nouvelles instances, et `List.remove`
  /// ne les retrouverait pas avec l'égalité par défaut.
  @override
  bool operator ==(Object autre) {
    if (identical(this, autre)) return true;
    return autre is Objet &&
        autre.nom == nom &&
        autre.type == type &&
        autre.rarete == rarete &&
        autre.puissance == puissance &&
        autre.valeur == valeur;
  }

  /// Redéfinir == oblige à redéfinir hashCode. Sans cela, un Set
  /// ou une Map se comporteraient de façon incohérente.
  @override
  int get hashCode => Object.hash(nom, type, rarete, puissance, valeur);

  @override
  String toString() =>
      '$nom (${type.libelle}, ${rarete.libelle}, puissance $puissanceEffective)';
}

/// Porte-constantes : tous les objets du jeu, déclarés une fois.
/// Le constructeur privé interdit d'écrire `Catalogue()`.
class Catalogue {
  Catalogue._();

  static const Objet potionMineure = Objet(
    nom: 'Potion mineure',
    type: TypeObjet.potion,
    rarete: Rarete.commune,
    puissance: 25,
    valeur: 10,
    description: 'Un flacon tiède au goût de fer.',
  );

  static const Objet potionMajeure = Objet(
    nom: 'Potion majeure',
    type: TypeObjet.potion,
    rarete: Rarete.rare,
    puissance: 30,
    valeur: 30,
    description: 'Elle sent la menthe et la magie.',
  );

  static const Objet epeeRouillee = Objet(
    nom: 'Épée rouillée',
    type: TypeObjet.arme,
    rarete: Rarete.commune,
    puissance: 6,
    valeur: 15,
    description: 'Elle a connu de meilleurs jours.',
  );

  static const Objet lameDeFeu = Objet(
    nom: 'Lame de feu',
    type: TypeObjet.arme,
    rarete: Rarete.epique,
    puissance: 9,
    valeur: 180,
    description: 'La lame reste tiède, même dans l\'eau.',
  );

  static const Objet armureDeCuir = Objet(
    nom: 'Armure de cuir',
    type: TypeObjet.armure,
    rarete: Rarete.commune,
    puissance: 4,
    valeur: 20,
    description: 'Souple, usée, mais elle a déjà sauvé quelqu\'un.',
  );

  static const Objet cotteDeMailles = Objet(
    nom: 'Cotte de mailles',
    type: TypeObjet.armure,
    rarete: Rarete.rare,
    puissance: 7,
    valeur: 60,
    description: 'Lourde. Rassurante.',
  );

  static const Objet amuletteAncienne = Objet(
    nom: 'Amulette ancienne',
    type: TypeObjet.tresor,
    rarete: Rarete.epique,
    valeur: 80,
    description: 'Une pierre noire montée sur un fil qui ne s\'use pas.',
  );

  static const Objet couronneDuDonjon = Objet(
    nom: 'Couronne du donjon',
    type: TypeObjet.tresor,
    rarete: Rarete.legendaire,
    valeur: 500,
    description: 'Elle appartenait à Kraghar. Elle vous appartient.',
  );

  static const List<Objet> tous = <Objet>[
    potionMineure,
    potionMajeure,
    epeeRouillee,
    lameDeFeu,
    armureDeCuir,
    cotteDeMailles,
    amuletteAncienne,
    couronneDuDonjon,
  ];
}
```

---

### `lib/models/personnage.dart`

```dart
import '../utils/outils.dart';

/// Ce qui est vrai de tout être vivant du donjon.
/// Deux membres sont abstraits : chaque sous-classe se décrit
/// et se sérialise à sa façon.
abstract class Personnage {
  Personnage({
    required this.nom,
    required int pointsDeVieMax,
    required this.attaque,
    required this.defense,
    int? pointsDeVie,
  })  : _pointsDeVieMax = pointsDeVieMax,
        _pointsDeVie = borner(pointsDeVie ?? pointsDeVieMax, 0, pointsDeVieMax);

  final String nom;
  int attaque;
  int defense;

  int _pointsDeVieMax;
  int _pointsDeVie;

  int get pointsDeVieMax => _pointsDeVieMax;
  int get pointsDeVie => _pointsDeVie;

  bool get estVivant => _pointsDeVie > 0;
  bool get estMort => _pointsDeVie <= 0;

  String get barreVie => barreDeVie(_pointsDeVie, _pointsDeVieMax);

  /// SEULE porte d'entrée pour modifier les points de vie.
  /// Publique (et non privée) pour rester accessible au mixin Soignable,
  /// qui vit dans un autre fichier.
  void definirPointsDeVie(int valeur) {
    _pointsDeVie = borner(valeur, 0, _pointsDeVieMax);
  }

  /// Renvoie les dégâts RÉELLEMENT appliqués : un ennemi à 3 PV
  /// qui reçoit 20 dégâts n'en perd que 3.
  int subirDegats(int degats) {
    if (degats <= 0) return 0;
    final int avant = _pointsDeVie;
    definirPointsDeVie(avant - degats);
    return avant - _pointsDeVie;
  }

  void augmenterPointsDeVieMax(int bonus) {
    if (bonus <= 0) return;
    _pointsDeVieMax += bonus;
    _pointsDeVie += bonus;
  }

  String get description;

  Map<String, dynamic> toJson();

  @override
  String toString() => '$nom ($_pointsDeVie/$_pointsDeVieMax PV)';
}
```

---

### `lib/models/soignable.dart`

```dart
import 'personnage.dart';

/// Capacité « peut être soigné ». Réservée au joueur : les monstres
/// de ce donjon ne se soignent pas.
/// `on Personnage` donne au mixin l'accès à pointsDeVie,
/// pointsDeVieMax et definirPointsDeVie.
mixin Soignable on Personnage {
  int _soinsRecus = 0;

  /// Statistique de session : volontairement non sauvegardée.
  int get soinsRecus => _soinsRecus;

  bool get estBlesse => pointsDeVie < pointsDeVieMax;

  int get pointsDeVieManquants => pointsDeVieMax - pointsDeVie;

  /// Renvoie les points RÉELLEMENT gagnés : à 95/100,
  /// une potion de 45 n'en rend que 5.
  int soigner(int points) {
    if (points <= 0) return 0;
    final int avant = pointsDeVie;
    definirPointsDeVie(avant + points);
    final int gagnes = pointsDeVie - avant;
    _soinsRecus += gagnes;
    return gagnes;
  }

  int soignerCompletement() => soigner(pointsDeVieManquants);
}
```

---

### `lib/models/inventaire.dart`

```dart
import '../exceptions/erreurs_jeu.dart';
import '../utils/outils.dart';
import 'objet.dart';
import 'types.dart';

/// Le meilleur exemple d'encapsulation du projet : la liste interne
/// n'est jamais exposée telle quelle.
class Inventaire {
  Inventaire({this.capacite = 10, List<Objet>? objets})
      : _objets = List<Objet>.from(objets ?? const <Objet>[]);

  final int capacite;
  final List<Objet> _objets;

  /// Copie NON MODIFIABLE. Sans `unmodifiable`, l'appelant
  /// contournerait la vérification de capacité.
  List<Objet> get objets => List<Objet>.unmodifiable(_objets);

  int get taille => _objets.length;
  bool get estPlein => _objets.length >= capacite;
  bool get estVide => _objets.isEmpty;

  /// Situation ANORMALE : on lève une exception.
  void ajouter(Objet objet) {
    if (estPlein) {
      throw InventairePleinException(objet.nom, capacite);
    }
    _objets.add(objet);
  }

  Objet retirerA(int index) {
    if (index < 0 || index >= _objets.length) {
      throw ObjetIntrouvableException('emplacement ${index + 1}');
    }
    return _objets.removeAt(index);
  }

  /// Retire le premier objet ÉGAL PAR VALEUR à [objet].
  bool retirer(Objet objet) => _objets.remove(objet);

  /// Situation PRÉVUE : on renvoie null.
  Objet? consulter(int index) {
    if (index < 0 || index >= _objets.length) return null;
    return _objets[index];
  }

  List<Objet> parType(TypeObjet type) =>
      _objets.where((Objet objet) => objet.type == type).toList();

  int get valeurTotale =>
      _objets.fold(0, (int total, Objet objet) => total + objet.valeur);

  Map<String, dynamic> toJson() => <String, dynamic>{
        'capacite': capacite,
        'objets': _objets.map((Objet objet) => objet.toJson()).toList(),
      };

  factory Inventaire.fromJson(Map<String, dynamic> json) {
    final List<Objet> objets = sousListeDepuis(json, 'objets')
        .map((Map<String, dynamic> brut) => Objet.fromJson(brut))
        .toList();
    return Inventaire(
      capacite: entierDepuis(json, 'capacite', parDefaut: 10),
      objets: objets,
    );
  }
}
```

---

### `lib/models/joueur.dart`

```dart
import '../exceptions/erreurs_jeu.dart';
import '../utils/outils.dart';
import 'inventaire.dart';
import 'objet.dart';
import 'personnage.dart';
import 'soignable.dart';
import 'types.dart';

class Joueur extends Personnage with Soignable {
  Joueur({
    required String nom,
    int pointsDeVieMax = 100,
    int? pointsDeVie,
    int attaque = 12,
    int defense = 5,
    this.niveau = 1,
    this.experience = 0,
    this.or = 20,
    this.salleCourante = 0,
    this.ennemisVaincus = 0,
    this.degatsInfliges = 0,
    this.armeEquipee,
    this.armureEquipee,
    Inventaire? inventaire,
  })  : inventaire = inventaire ?? Inventaire(),
        super(
          nom: nom,
          pointsDeVieMax: pointsDeVieMax,
          pointsDeVie: pointsDeVie,
          attaque: attaque,
          defense: defense,
        );

  int niveau;
  int experience;
  int or;
  int salleCourante;
  int ennemisVaincus;
  int degatsInfliges;

  Objet? armeEquipee;
  Objet? armureEquipee;

  final Inventaire inventaire;

  int get experienceRequise => niveau * 100;

  int get attaqueTotale => attaque + (armeEquipee?.puissanceEffective ?? 0);
  int get defenseTotale => defense + (armureEquipee?.puissanceEffective ?? 0);

  String ramasser(Objet objet) {
    inventaire.ajouter(objet);
    return '$nom ramasse ${objet.nom}.';
  }

  /// L'objet doit AVOIR ÉTÉ RETIRÉ du sac avant l'appel :
  /// une place est donc libre pour y remettre l'ancien équipement.
  String equiper(Objet objet) {
    if (objet.type == TypeObjet.arme) {
      final Objet? ancienne = armeEquipee;
      armeEquipee = objet;
      if (ancienne != null) {
        inventaire.ajouter(ancienne);
      }
      return '$nom équipe ${objet.nom}. Attaque totale : $attaqueTotale.';
    }
    if (objet.type == TypeObjet.armure) {
      final Objet? ancienne = armureEquipee;
      armureEquipee = objet;
      if (ancienne != null) {
        inventaire.ajouter(ancienne);
      }
      return '$nom enfile ${objet.nom}. Défense totale : $defenseTotale.';
    }
    throw ActionImpossibleException('${objet.nom} ne s\'équipe pas.');
  }

  /// La potion la plus puissante du sac, ou null si le sac n'en contient pas.
  /// `parType` renvoie une NOUVELLE liste : le tri ne touche pas l'inventaire.
  Objet? meilleurePotion() {
    final List<Objet> potions = inventaire.parType(TypeObjet.potion);
    if (potions.isEmpty) return null;
    potions.sort((Objet a, Objet b) =>
        b.puissanceEffective.compareTo(a.puissanceEffective));
    return potions.first;
  }

  String boirePotion(Objet potion) {
    if (!potion.estConsommable) {
      throw ActionImpossibleException('${potion.nom} ne se boit pas.');
    }
    if (!inventaire.retirer(potion)) {
      throw ObjetIntrouvableException(potion.nom);
    }
    final int gagnes = soigner(potion.puissanceEffective);
    return '$nom boit ${potion.nom} et récupère $gagnes points de vie.';
  }

  /// Point d'entrée unique du menu d'inventaire.
  String utiliser(int index) {
    final Objet? objet = inventaire.consulter(index);
    if (objet == null) {
      throw ObjetIntrouvableException('emplacement ${index + 1}');
    }
    if (objet.estConsommable) {
      return boirePotion(objet);
    }
    if (objet.estEquipable) {
      inventaire.retirerA(index);
      return equiper(objet);
    }
    throw ActionImpossibleException(
        '${objet.nom} ne s\'utilise pas : c\'est un trésor, il compte pour le score.');
  }

  /// `while` et non `if` : le boss donne 400 xp d'un coup et peut
  /// faire franchir plusieurs paliers.
  List<String> gagnerExperience(int points) {
    final List<String> journal = <String>[];
    if (points <= 0) return journal;

    experience += points;
    journal.add('$nom gagne $points points d\'expérience.');

    while (experience >= experienceRequise) {
      experience -= experienceRequise;
      journal.addAll(_monterDeNiveau());
    }
    return journal;
  }

  List<String> _monterDeNiveau() {
    niveau++;
    augmenterPointsDeVieMax(20);
    attaque += 3;
    defense += 2;
    return <String>[
      'NIVEAU $niveau ATTEINT !',
      'PV max +20 ($pointsDeVieMax), attaque +3 ($attaque), défense +2 ($defense).',
    ];
  }

  @override
  String get description =>
      '$nom  niv.$niveau  $barreVie $pointsDeVie/$pointsDeVieMax PV  or: $or';

  @override
  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'pointsDeVieMax': pointsDeVieMax,
        'pointsDeVie': pointsDeVie,
        'attaque': attaque,
        'defense': defense,
        'niveau': niveau,
        'experience': experience,
        'or': or,
        'salleCourante': salleCourante,
        'ennemisVaincus': ennemisVaincus,
        'degatsInfliges': degatsInfliges,
        'inventaire': inventaire.toJson(),
        'arme': armeEquipee?.toJson(),
        'armure': armureEquipee?.toJson(),
      };

  factory Joueur.fromJson(Map<String, dynamic> json) {
    final Object? nom = json['nom'];
    if (nom is! String || nom.isEmpty) {
      throw SauvegardeCorrompueException('le joueur n\'a pas de champ "nom"');
    }
    final Map<String, dynamic>? inventaireJson =
        sousObjetDepuis(json, 'inventaire');
    if (inventaireJson == null) {
      throw SauvegardeCorrompueException('le joueur n\'a pas d\'inventaire');
    }
    final int pvMax = entierDepuis(json, 'pointsDeVieMax', parDefaut: 100);
    final Map<String, dynamic>? armeJson = sousObjetDepuis(json, 'arme');
    final Map<String, dynamic>? armureJson = sousObjetDepuis(json, 'armure');

    return Joueur(
      nom: nom,
      pointsDeVieMax: pvMax,
      pointsDeVie: entierDepuis(json, 'pointsDeVie', parDefaut: pvMax),
      attaque: entierDepuis(json, 'attaque', parDefaut: 12),
      defense: entierDepuis(json, 'defense', parDefaut: 5),
      niveau: entierDepuis(json, 'niveau', parDefaut: 1),
      experience: entierDepuis(json, 'experience'),
      or: entierDepuis(json, 'or'),
      salleCourante: entierDepuis(json, 'salleCourante'),
      ennemisVaincus: entierDepuis(json, 'ennemisVaincus'),
      degatsInfliges: entierDepuis(json, 'degatsInfliges'),
      armeEquipee: armeJson == null ? null : Objet.fromJson(armeJson),
      armureEquipee: armureJson == null ? null : Objet.fromJson(armureJson),
      inventaire: Inventaire.fromJson(inventaireJson),
    );
  }
}
```

---

### `lib/models/ennemi.dart`

```dart
import '../utils/outils.dart';
import 'objet.dart';
import 'personnage.dart';

class Ennemi extends Personnage {
  Ennemi({
    required String nom,
    required int pointsDeVieMax,
    required int attaque,
    required int defense,
    int? pointsDeVie,
    this.experienceDonnee = 20,
    this.orDonne = 5,
    this.estBoss = false,
    this.butin,
  }) : super(
          nom: nom,
          pointsDeVieMax: pointsDeVieMax,
          pointsDeVie: pointsDeVie,
          attaque: attaque,
          defense: defense,
        );

  final int experienceDonnee;
  final int orDonne;
  final bool estBoss;
  final Objet? butin;

  // Le bestiaire, écrit en constructeurs redirigés : `this(...)`
  // appelle le constructeur principal ci-dessus.

  Ennemi.gobelin()
      : this(
          nom: 'Gobelin',
          pointsDeVieMax: 30,
          attaque: 8,
          defense: 2,
          experienceDonnee: 40,
          orDonne: 8,
          butin: Catalogue.epeeRouillee,
        );

  Ennemi.squelette()
      : this(
          nom: 'Squelette',
          pointsDeVieMax: 45,
          attaque: 12,
          defense: 5,
          experienceDonnee: 70,
          orDonne: 20,
          butin: Catalogue.potionMajeure,
        );

  Ennemi.golem()
      : this(
          nom: 'Golem de pierre',
          pointsDeVieMax: 70,
          attaque: 16,
          defense: 9,
          experienceDonnee: 130,
          orDonne: 45,
          butin: Catalogue.cotteDeMailles,
        );

  Ennemi.boss()
      : this(
          nom: 'Kraghar',
          pointsDeVieMax: 160,
          attaque: 22,
          defense: 12,
          experienceDonnee: 400,
          orDonne: 250,
          estBoss: true,
          butin: Catalogue.couronneDuDonjon,
        );

  @override
  String get description =>
      '$nom  $barreVie $pointsDeVie/$pointsDeVieMax  att $attaque  def $defense'
      '  ($experienceDonnee xp, $orDonne or)';

  @override
  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'pointsDeVieMax': pointsDeVieMax,
        'pointsDeVie': pointsDeVie,
        'attaque': attaque,
        'defense': defense,
        'experienceDonnee': experienceDonnee,
        'orDonne': orDonne,
        'estBoss': estBoss,
        'butin': butin?.toJson(),
      };

  factory Ennemi.fromJson(Map<String, dynamic> json) {
    final int pvMax = entierDepuis(json, 'pointsDeVieMax', parDefaut: 10);
    final Map<String, dynamic>? butinJson = sousObjetDepuis(json, 'butin');
    return Ennemi(
      nom: texteDepuis(json, 'nom', parDefaut: 'Créature'),
      pointsDeVieMax: pvMax,
      pointsDeVie: entierDepuis(json, 'pointsDeVie', parDefaut: pvMax),
      attaque: entierDepuis(json, 'attaque', parDefaut: 1),
      defense: entierDepuis(json, 'defense'),
      experienceDonnee: entierDepuis(json, 'experienceDonnee', parDefaut: 20),
      orDonne: entierDepuis(json, 'orDonne', parDefaut: 5),
      estBoss: booleenDepuis(json, 'estBoss'),
      butin: butinJson == null ? null : Objet.fromJson(butinJson),
    );
  }
}
```

---

### `lib/models/salle.dart`

```dart
import '../utils/outils.dart';
import 'ennemi.dart';
import 'objet.dart';

class Salle {
  Salle({
    required this.numero,
    required this.nom,
    required this.description,
    this.ennemi,
    List<Objet>? tresors,
    this.visitee = false,
    this.nettoyee = false,
  }) : tresors = tresors ?? <Objet>[];

  final int numero;
  final String nom;
  final String description;

  Ennemi? ennemi;
  final List<Objet> tresors;

  bool visitee;
  bool nettoyee;

  /// La copie locale `occupant` permet la promotion de type :
  /// Dart refuse de promouvoir un CHAMP de classe.
  bool get contientUnEnnemiVivant {
    final Ennemi? occupant = ennemi;
    return occupant != null && occupant.estVivant;
  }

  String get etat {
    if (contientUnEnnemiVivant) return 'DANGER';
    if (nettoyee) return 'sûre';
    if (visitee) return 'explorée';
    return 'inconnue';
  }

  /// Renvoie une copie PUIS vide la liste : impossible de ramasser
  /// deux fois le même trésor en revenant dans la salle.
  List<Objet> viderLesTresors() {
    final List<Objet> butin = List<Objet>.from(tresors);
    tresors.clear();
    return butin;
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'numero': numero,
        'nom': nom,
        'description': description,
        'ennemi': ennemi?.toJson(),
        'tresors': tresors.map((Objet objet) => objet.toJson()).toList(),
        'visitee': visitee,
        'nettoyee': nettoyee,
      };

  factory Salle.fromJson(Map<String, dynamic> json) {
    final Map<String, dynamic>? ennemiJson = sousObjetDepuis(json, 'ennemi');
    return Salle(
      numero: entierDepuis(json, 'numero', parDefaut: 1),
      nom: texteDepuis(json, 'nom', parDefaut: 'Salle sans nom'),
      description: texteDepuis(json, 'description'),
      ennemi: ennemiJson == null ? null : Ennemi.fromJson(ennemiJson),
      tresors: sousListeDepuis(json, 'tresors')
          .map((Map<String, dynamic> brut) => Objet.fromJson(brut))
          .toList(),
      visitee: booleenDepuis(json, 'visitee'),
      nettoyee: booleenDepuis(json, 'nettoyee'),
    );
  }
}
```

---

### `lib/models/donjon.dart`

```dart
import '../exceptions/erreurs_jeu.dart';
import '../utils/outils.dart';
import 'ennemi.dart';
import 'objet.dart';
import 'salle.dart';

class Donjon {
  Donjon({required this.nom, required List<Salle> salles})
      : _salles = List<Salle>.from(salles);

  final String nom;
  final List<Salle> _salles;

  List<Salle> get salles => List<Salle>.unmodifiable(_salles);

  int get nombreDeSalles => _salles.length;

  Salle salleA(int index) {
    if (index < 0 || index >= _salles.length) {
      throw ActionImpossibleException(
          'la salle numéro ${index + 1} n\'existe pas.');
    }
    return _salles[index];
  }

  bool get estTermine => _salles.every((Salle salle) => salle.nettoyee);

  int get sallesVisitees =>
      _salles.where((Salle salle) => salle.visitee).length;

  /// Le niveau 1 du jeu, écrit en dur : la même partie est rejouable
  /// à l'identique, ce qui est précieux tant que l'équilibrage bouge.
  factory Donjon.parDefaut() {
    return Donjon(
      nom: 'Le Donjon de Kraghar',
      salles: <Salle>[
        Salle(
          numero: 1,
          nom: 'Le seuil moussu',
          description: 'Une dalle usée marque l\'entrée du donjon. '
              'L\'air sent la pierre humide.',
          tresors: <Objet>[Catalogue.potionMineure],
        ),
        Salle(
          numero: 2,
          nom: 'La galerie des torches',
          description:
              'Des torches brûlent seules le long d\'un couloir trop long.',
          ennemi: Ennemi.gobelin(),
        ),
        Salle(
          numero: 3,
          nom: 'La crypte fissurée',
          description: 'Trois sarcophages ouverts. Aucun n\'est vide.',
          ennemi: Ennemi.squelette(),
          tresors: <Objet>[Catalogue.armureDeCuir],
        ),
        Salle(
          numero: 4,
          nom: 'La salle d\'eau noire',
          description:
              'Un bassin immobile occupe le centre. Quelque chose y bouge.',
          ennemi: Ennemi.gobelin(),
          tresors: <Objet>[Catalogue.potionMineure],
        ),
        Salle(
          numero: 5,
          nom: 'La forge éteinte',
          description: 'Les braises sont froides depuis des siècles, '
              'mais l\'enclume est encore tiède.',
          ennemi: Ennemi.golem(),
          tresors: <Objet>[Catalogue.lameDeFeu, Catalogue.amuletteAncienne],
        ),
        Salle(
          numero: 6,
          nom: 'Le trône de Kraghar',
          description: 'Une salle immense. Au fond, une silhouette assise '
              'qui vous attendait.',
          ennemi: Ennemi.boss(),
        ),
      ],
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'salles': _salles.map((Salle salle) => salle.toJson()).toList(),
      };

  factory Donjon.fromJson(Map<String, dynamic> json) {
    final List<Salle> salles = sousListeDepuis(json, 'salles')
        .map((Map<String, dynamic> brut) => Salle.fromJson(brut))
        .toList();
    if (salles.isEmpty) {
      throw SauvegardeCorrompueException('le donjon ne contient aucune salle');
    }
    return Donjon(
      nom: texteDepuis(json, 'nom', parDefaut: 'Donjon sans nom'),
      salles: salles,
    );
  }
}
```

---

### `lib/models/partie.dart`

```dart
import '../exceptions/erreurs_jeu.dart';
import '../utils/outils.dart';
import 'donjon.dart';
import 'joueur.dart';
import 'salle.dart';

/// L'état global d'une session de jeu. C'est cet objet, et lui seul,
/// que l'on sérialise pour sauvegarder.
class Partie {
  Partie({
    required this.joueur,
    required this.donjon,
    this.tourDeJeu = 0,
    this.terminee = false,
    this.victoire = false,
  });

  final Joueur joueur;
  final Donjon donjon;

  int tourDeJeu;
  bool terminee;
  bool victoire;

  Salle get salleCourante => donjon.salleA(joueur.salleCourante);

  bool get estDerniereSalle => joueur.salleCourante >= donjon.nombreDeSalles - 1;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'tourDeJeu': tourDeJeu,
        'terminee': terminee,
        'victoire': victoire,
        'joueur': joueur.toJson(),
        'donjon': donjon.toJson(),
      };

  factory Partie.fromJson(Map<String, dynamic> json) {
    final Map<String, dynamic>? joueurJson = sousObjetDepuis(json, 'joueur');
    if (joueurJson == null) {
      throw SauvegardeCorrompueException('bloc "joueur" absent');
    }
    final Map<String, dynamic>? donjonJson = sousObjetDepuis(json, 'donjon');
    if (donjonJson == null) {
      throw SauvegardeCorrompueException('bloc "donjon" absent');
    }

    final Joueur joueur = Joueur.fromJson(joueurJson);
    final Donjon donjon = Donjon.fromJson(donjonJson);

    // Garde-fou : une sauvegarde modifiée à la main pourrait placer
    // le joueur dans une salle inexistante.
    joueur.salleCourante =
        borner(joueur.salleCourante, 0, donjon.nombreDeSalles - 1);

    return Partie(
      joueur: joueur,
      donjon: donjon,
      tourDeJeu: entierDepuis(json, 'tourDeJeu'),
      terminee: booleenDepuis(json, 'terminee'),
      victoire: booleenDepuis(json, 'victoire'),
    );
  }
}
```

---

### `lib/services/console.dart`

```dart
import 'dart:io';

/// Tout ce qui touche au clavier et à l'écran passe par ici.
/// Un seul fichier à remplacer le jour où l'interface devient Flutter.
class Console {
  const Console();

  void ecrire(String texte) => stdout.writeln(texte);

  void sauterLigne() => stdout.writeln('');

  void separateur() => stdout.writeln('-' * 56);

  void titre(String texte) {
    final String ligne = '=' * (texte.length + 4);
    stdout.writeln(ligne);
    stdout.writeln('  $texte');
    stdout.writeln(ligne);
  }

  void afficherMenu(List<String> options) {
    for (int i = 0; i < options.length; i++) {
      ecrire('${i + 1}. ${options[i]}');
    }
    ecrire('0. Retour');
  }

  /// Renvoie null quand l'entrée standard est fermée (Ctrl+D, script
  /// sans clavier). Ignorer ce null, c'est boucler à l'infini.
  String? lireLigne(String invite) {
    stdout.write(invite);
    final String? saisie = stdin.readLineSync();
    return saisie?.trim();
  }

  /// Quatre garde-fous : entrée fermée, texte non numérique,
  /// hors bornes, et abandon après cinq échecs.
  int? lireEntier(String invite, {required int minimum, required int maximum}) {
    for (int essai = 0; essai < 5; essai++) {
      final String? saisie = lireLigne(invite);

      if (saisie == null) return null;

      final int? valeur = int.tryParse(saisie);
      if (valeur == null) {
        ecrire('Saisie invalide : "$saisie" n\'est pas un nombre entier.');
        continue;
      }
      if (valeur < minimum || valeur > maximum) {
        ecrire('Saisie hors limites : attendu entre $minimum et $maximum.');
        continue;
      }
      return valeur;
    }
    ecrire('Trop de saisies invalides, l\'action est annulée.');
    return null;
  }

  bool confirmer(String question) {
    for (int essai = 0; essai < 3; essai++) {
      final String? reponse = lireLigne('$question (o/n) > ');
      if (reponse == null) return false;
      final String r = reponse.toLowerCase();
      if (r == 'o' || r == 'oui') return true;
      if (r == 'n' || r == 'non') return false;
      ecrire('Répondez par "o" ou par "n".');
    }
    return false;
  }

  String lireNom(String invite, {String parDefaut = 'Aventurier'}) {
    final String? saisie = lireLigne(invite);
    if (saisie == null || saisie.isEmpty) return parDefaut;
    if (saisie.length > 20) return saisie.substring(0, 20);
    return saisie;
  }
}
```

---

### `lib/services/moteur_combat.dart`

```dart
import 'dart:math';

import '../models/ennemi.dart';
import '../models/joueur.dart';
import '../models/objet.dart';
import '../models/personnage.dart';

enum ActionCombat { attaquer, potion, fuir }

/// Ce qui s'est passé sur UNE attaque. Immuable, et capable
/// de se raconter — mais jamais de s'afficher elle-même.
class ResultatAttaque {
  const ResultatAttaque({
    required this.attaquant,
    required this.cible,
    required this.degats,
    required this.critique,
    required this.esquive,
  });

  final String attaquant;
  final String cible;
  final int degats;
  final bool critique;
  final bool esquive;

  String get recit {
    if (esquive) return '$cible esquive l\'attaque de $attaquant !';
    if (critique) {
      return 'COUP CRITIQUE ! $attaquant inflige $degats dégâts à $cible.';
    }
    return '$attaquant inflige $degats dégâts à $cible.';
  }
}

/// Ce qui s'est passé sur UN tour complet (action du joueur + riposte).
class ResultatTour {
  const ResultatTour({
    required this.journal,
    required this.combatTermine,
    required this.fuiteReussie,
  });

  final List<String> journal;
  final bool combatTermine;
  final bool fuiteReussie;
}

/// Un service : il applique des règles, il ne stocke pas l'état du jeu.
class MoteurCombat {
  /// `Random? hasard` n'est pas un luxe : c'est ce qui rend
  /// le moteur testable, en passant `Random(42)`.
  MoteurCombat({Random? hasard}) : _hasard = hasard ?? Random();

  final Random _hasard;

  static const int degatsMinimum = 1;
  static const double chanceCritique = 0.15;
  static const double multiplicateurCritique = 2.0;
  static const double chanceEsquive = 0.08;
  static const double chanceFuite = 0.5;

  int calculerDegats({
    required int attaque,
    required int defense,
    required bool critique,
  }) {
    int base = attaque - (defense ~/ 2);
    if (base < degatsMinimum) base = degatsMinimum;

    final int variation = _hasard.nextInt(5) - 2; // -2, -1, 0, +1 ou +2
    int degats = base + variation;
    if (degats < degatsMinimum) degats = degatsMinimum;

    if (critique) {
      degats = (degats * multiplicateurCritique).round();
    }
    return degats;
  }

  /// Fonctionne dans les deux sens : attaquer(joueur, ennemi)
  /// et attaquer(ennemi, joueur) empruntent le même code.
  ResultatAttaque attaquer(Personnage attaquant, Personnage cible) {
    final int attaqueEffective =
        attaquant is Joueur ? attaquant.attaqueTotale : attaquant.attaque;
    final int defenseEffective =
        cible is Joueur ? cible.defenseTotale : cible.defense;

    if (_hasard.nextDouble() < chanceEsquive) {
      return ResultatAttaque(
        attaquant: attaquant.nom,
        cible: cible.nom,
        degats: 0,
        critique: false,
        esquive: true,
      );
    }

    final bool critique = _hasard.nextDouble() < chanceCritique;
    final int degatsCalcules = calculerDegats(
      attaque: attaqueEffective,
      defense: defenseEffective,
      critique: critique,
    );
    final int degatsReels = cible.subirDegats(degatsCalcules);

    if (attaquant is Joueur) {
      attaquant.degatsInfliges += degatsReels;
    }

    return ResultatAttaque(
      attaquant: attaquant.nom,
      cible: cible.nom,
      degats: degatsReels,
      critique: critique,
      esquive: false,
    );
  }

  /// L'ordre des vérifications est capital : un ennemi tué
  /// ne doit PAS riposter.
  ResultatTour jouerTour(Joueur joueur, Ennemi ennemi, ActionCombat action) {
    final List<String> journal = <String>[];
    bool fuiteReussie = false;

    switch (action) {
      case ActionCombat.attaquer:
        journal.add(attaquer(joueur, ennemi).recit);
        break;

      case ActionCombat.potion:
        final Objet? potion = joueur.meilleurePotion();
        if (potion == null) {
          journal.add('Aucune potion dans le sac. Le tour est perdu.');
        } else {
          journal.add(joueur.boirePotion(potion));
        }
        break;

      case ActionCombat.fuir:
        if (ennemi.estBoss) {
          journal.add('Impossible de fuir devant ${ennemi.nom} !');
        } else if (_hasard.nextDouble() < chanceFuite) {
          journal.add('${joueur.nom} parvient à fuir le combat.');
          fuiteReussie = true;
        } else {
          journal.add('La fuite échoue !');
        }
        break;
    }

    if (fuiteReussie) {
      return ResultatTour(
          journal: journal, combatTermine: true, fuiteReussie: true);
    }

    if (ennemi.estMort) {
      journal.add('${ennemi.nom} est vaincu !');
      return ResultatTour(
          journal: journal, combatTermine: true, fuiteReussie: false);
    }

    journal.add(attaquer(ennemi, joueur).recit);

    if (joueur.estMort) {
      journal.add('${joueur.nom} s\'effondre...');
      return ResultatTour(
          journal: journal, combatTermine: true, fuiteReussie: false);
    }

    return ResultatTour(
        journal: journal, combatTermine: false, fuiteReussie: false);
  }
}
```

---

### `lib/services/sauvegarde_service.dart`

```dart
import 'dart:convert';
import 'dart:io';

import '../exceptions/erreurs_jeu.dart';
import '../models/partie.dart';

/// Le seul service qui touche au disque, donc le seul asynchrone.
class SauvegardeService {
  SauvegardeService({this.chemin = 'sauvegardes/partie.json'});

  final String chemin;

  static const int versionFormat = 1;

  Future<bool> existe() => File(chemin).exists();

  Future<void> sauvegarder(Partie partie) async {
    final File fichier = File(chemin);
    // Sans cette ligne, la toute première sauvegarde échoue :
    // le dossier sauvegardes/ n'existe pas encore.
    await fichier.parent.create(recursive: true);

    final Map<String, dynamic> donnees = <String, dynamic>{
      'version': versionFormat,
      'date': DateTime.now().toIso8601String(),
      'partie': partie.toJson(),
    };

    const JsonEncoder encodeur = JsonEncoder.withIndent('  ');
    await fichier.writeAsString(encodeur.convert(donnees));
  }

  Future<Partie> charger() async {
    final File fichier = File(chemin);

    if (!await fichier.exists()) {
      throw SauvegardeCorrompueException('fichier introuvable', chemin: chemin);
    }

    final String contenu = await fichier.readAsString();

    Object? brut;
    try {
      brut = jsonDecode(contenu);
    } on FormatException catch (erreur) {
      // Traduction d'exception : les couches hautes ne connaissent
      // que SauvegardeCorrompueException, jamais dart:convert.
      throw SauvegardeCorrompueException(
        'JSON illisible (${erreur.message})',
        chemin: chemin,
      );
    }

    if (brut is! Map<String, dynamic>) {
      throw SauvegardeCorrompueException(
        'la racine du fichier n\'est pas un objet JSON',
        chemin: chemin,
      );
    }

    final Object? version = brut['version'];
    if (version != versionFormat) {
      throw SauvegardeCorrompueException(
        'version $version incompatible (attendu $versionFormat)',
        chemin: chemin,
      );
    }

    final Object? partieJson = brut['partie'];
    if (partieJson is! Map<String, dynamic>) {
      throw SauvegardeCorrompueException('bloc "partie" absent', chemin: chemin);
    }

    return Partie.fromJson(partieJson);
  }

  Future<void> supprimer() async {
    final File fichier = File(chemin);
    if (await fichier.exists()) {
      await fichier.delete();
    }
  }
}
```

---

### `lib/services/statistiques.dart`

```dart
import '../models/joueur.dart';
import '../models/objet.dart';
import '../models/partie.dart';
import '../models/salle.dart';
import '../models/types.dart';

/// Tout se calcule avec where, map et fold : aucune boucle `for`.
class Statistiques {
  const Statistiques(this.partie);

  final Partie partie;

  Joueur get _joueur => partie.joueur;
  List<Salle> get _salles => partie.donjon.salles;

  int get sallesVisitees =>
      _salles.where((Salle salle) => salle.visitee).length;

  int get sallesNettoyees =>
      _salles.where((Salle salle) => salle.nettoyee).length;

  List<String> get ennemisRestants => _salles
      .where((Salle salle) => salle.contientUnEnnemiVivant)
      .map((Salle salle) => salle.ennemi?.nom ?? 'inconnu')
      .toList();

  int get valeurDuSac => _joueur.inventaire.objets
      .fold(0, (int total, Objet objet) => total + objet.valeur);

  int get puissanceEquipee =>
      (_joueur.armeEquipee?.puissanceEffective ?? 0) +
      (_joueur.armureEquipee?.puissanceEffective ?? 0);

  /// L'accumulateur d'un fold n'est pas obligé d'être un nombre :
  /// ici c'est une Map.
  Map<Rarete, int> get repartitionParRarete =>
      _joueur.inventaire.objets.fold(
        <Rarete, int>{},
        (Map<Rarete, int> compteur, Objet objet) {
          compteur[objet.rarete] = (compteur[objet.rarete] ?? 0) + 1;
          return compteur;
        },
      );

  String get rapport {
    final StringBuffer tampon = StringBuffer();
    final String repartition = repartitionParRarete.entries
        .map((MapEntry<Rarete, int> e) => '${e.key.libelle} ${e.value}')
        .join(', ');

    tampon.writeln('--- Statistiques de fin de partie ---');
    tampon.writeln('Aventurier          : ${_joueur.nom}');
    tampon.writeln('Niveau atteint      : ${_joueur.niveau}');
    tampon.writeln('Salles visitées     : $sallesVisitees / ${_salles.length}');
    tampon.writeln('Salles nettoyées    : $sallesNettoyees / ${_salles.length}');
    tampon.writeln('Ennemis vaincus     : ${_joueur.ennemisVaincus}');
    tampon.writeln('Dégâts infligés     : ${_joueur.degatsInfliges}');
    tampon.writeln('Soins reçus         : ${_joueur.soinsRecus}');
    tampon.writeln('Or amassé           : ${_joueur.or}');
    tampon.writeln('Objets dans le sac  : ${_joueur.inventaire.taille} '
        '(valeur $valeurDuSac pièces)');
    tampon.writeln('Répartition         : '
        '${repartition.isEmpty ? 'sac vide' : repartition}');
    tampon.writeln('Puissance équipée   : $puissanceEquipee');
    tampon.writeln('Tours de jeu        : ${partie.tourDeJeu}');
    return tampon.toString();
  }
}
```

---

### `lib/jeu.dart`

```dart
import 'dart:io';

import 'exceptions/erreurs_jeu.dart';
import 'models/donjon.dart';
import 'models/ennemi.dart';
import 'models/joueur.dart';
import 'models/objet.dart';
import 'models/partie.dart';
import 'models/salle.dart';
import 'services/console.dart';
import 'services/moteur_combat.dart';
import 'services/sauvegarde_service.dart';
import 'services/statistiques.dart';

/// L'arbitre : il enchaîne les menus, appelle les services
/// et affiche ce qu'ils renvoient. Il ne contient AUCUNE règle
/// de combat ni AUCUN calcul de dégâts.
class Jeu {
  Jeu({
    String cheminSauvegarde = 'sauvegardes/partie.json',
    Console? console,
    MoteurCombat? moteur,
  })  : console = console ?? const Console(),
        moteur = moteur ?? MoteurCombat(),
        sauvegarde = SauvegardeService(chemin: cheminSauvegarde);

  final Console console;
  final MoteurCombat moteur;
  final SauvegardeService sauvegarde;

  // ---------------------------------------------------------------
  // Boucle 1 : le menu principal
  // ---------------------------------------------------------------

  Future<void> demarrer() async {
    console.titre('DONJON DE DART');

    bool actif = true;
    while (actif) {
      console.sauterLigne();
      console.ecrire('1. Nouvelle partie');
      console.ecrire('2. Charger la partie');
      console.ecrire('0. Quitter');

      final int? choix =
          console.lireEntier('Votre choix > ', minimum: 0, maximum: 2);

      if (choix == null || choix == 0) {
        actif = false;
        continue;
      }
      if (choix == 1) {
        await _boucleDeJeu(_creerPartie());
      } else {
        final Partie? chargee = await _charger();
        if (chargee != null) await _boucleDeJeu(chargee);
      }
    }
    console.ecrire('Merci d\'avoir joué.');
  }

  Partie _creerPartie() {
    console.sauterLigne();
    final String nom = console.lireNom('Quel est le nom de votre aventurier ? > ');
    final Donjon donjon = Donjon.parDefaut();
    final Partie partie = Partie(joueur: Joueur(nom: nom), donjon: donjon);
    partie.salleCourante.visitee = true;

    console.sauterLigne();
    console.ecrire('Bienvenue, $nom. ${donjon.nombreDeSalles} salles '
        'vous séparent du trône de Kraghar.');
    return partie;
  }

  Future<Partie?> _charger() async {
    try {
      final Partie partie = await sauvegarde.charger();
      console.ecrire('Partie chargée : ${partie.joueur.nom}, '
          'niveau ${partie.joueur.niveau}, tour ${partie.tourDeJeu}.');
      return partie;
    } on SauvegardeCorrompueException catch (erreur) {
      console.ecrire('Chargement impossible : ${erreur.message}');
      return null;
    }
  }

  // ---------------------------------------------------------------
  // Boucle 2 : l'exploration
  // ---------------------------------------------------------------

  Future<void> _boucleDeJeu(Partie partie) async {
    while (!partie.terminee) {
      partie.tourDeJeu++;
      final Salle salle = partie.salleCourante;

      _afficherEnTete(partie, salle);

      final int? choix =
          console.lireEntier('Votre choix > ', minimum: 0, maximum: 6);
      if (choix == null) {
        partie.terminee = true;
        continue;
      }

      // Un seul try pour toutes les actions : aucune ErreurDeJeu
      // ne peut arrêter la partie.
      try {
        switch (choix) {
          case 1:
            _explorer(partie, salle);
            break;
          case 2:
            _combattre(partie, salle);
            break;
          case 3:
            _ouvrirInventaire(partie);
            break;
          case 4:
            _avancer(partie);
            break;
          case 5:
            await _sauvegarder(partie);
            break;
          case 6:
            console.sauterLigne();
            console.ecrire(Statistiques(partie).rapport);
            break;
          case 0:
            if (console.confirmer('Quitter la partie en cours ?')) {
              partie.terminee = true;
            }
            break;
        }
      } on ErreurDeJeu catch (erreur) {
        console.ecrire('Action impossible : ${erreur.message}');
      }

      // Une seule vérification de mort, en fin de tour.
      if (partie.joueur.estMort) {
        partie.terminee = true;
        partie.victoire = false;
      }
    }
    _afficherFin(partie);
  }

  void _afficherEnTete(Partie partie, Salle salle) {
    console.sauterLigne();
    console.separateur();
    console.ecrire('Salle ${salle.numero}/${partie.donjon.nombreDeSalles} : '
        '${salle.nom}');
    console.ecrire(partie.joueur.description);
    console.separateur();
    console.ecrire(salle.description);

    final Ennemi? occupant = salle.ennemi;
    if (occupant != null && occupant.estVivant) {
      console.ecrire('${occupant.nom} est là. (${occupant.pointsDeVie} PV)');
    }

    console.sauterLigne();
    console.ecrire('1. Explorer la salle');
    console.ecrire('2. Combattre');
    console.ecrire('3. Inventaire');
    console.ecrire('4. Salle suivante');
    console.ecrire('5. Sauvegarder');
    console.ecrire('6. Statistiques');
    console.ecrire('0. Quitter');
  }

  // ---------------------------------------------------------------
  // Les actions
  // ---------------------------------------------------------------

  void _explorer(Partie partie, Salle salle) {
    salle.visitee = true;
    console.sauterLigne();
    console.ecrire('Vous fouillez la salle...');

    if (salle.contientUnEnnemiVivant) {
      final Ennemi? occupant = salle.ennemi;
      console.ecrire('${occupant?.nom ?? 'Une créature'} vous en empêche.');
      return;
    }

    final List<Objet> butin = salle.viderLesTresors();
    if (butin.isEmpty) {
      console.ecrire('Rien à ramasser ici.');
    } else {
      for (final Objet objet in butin) {
        _tenterDeRamasser(partie, salle, objet);
      }
    }
    _marquerSiNettoyee(salle);
  }

  /// Un sac plein n'est pas une panne : l'objet retourne au sol.
  void _tenterDeRamasser(Partie partie, Salle salle, Objet objet) {
    try {
      console.ecrire(partie.joueur.ramasser(objet));
    } on InventairePleinException catch (erreur) {
      console.ecrire('Sac plein ! ${erreur.objetRefuse} reste au sol.');
      salle.tresors.add(objet);
    }
  }

  void _marquerSiNettoyee(Salle salle) {
    if (salle.contientUnEnnemiVivant) return;
    if (salle.tresors.isEmpty) {
      salle.nettoyee = true;
      console.ecrire('Cette salle est désormais sûre.');
    } else {
      console.ecrire('La voie est libre, mais il reste quelque chose au sol.');
    }
  }

  // ---------------------------------------------------------------
  // Boucle 3 : le combat
  // ---------------------------------------------------------------

  void _combattre(Partie partie, Salle salle) {
    final Ennemi? occupant = salle.ennemi;
    if (occupant == null || occupant.estMort) {
      console.ecrire('Il n\'y a plus rien à combattre ici.');
      return;
    }

    console.sauterLigne();
    console.ecrire('${occupant.nom} (${occupant.pointsDeVie} PV) '
        'bloque le passage !');

    bool combatEnCours = true;
    int tour = 0;

    while (combatEnCours) {
      tour++;
      console.sauterLigne();
      console.ecrire('--- Tour $tour ---');
      console.ecrire('${partie.joueur.nom.padRight(8)} ${partie.joueur.barreVie} '
          '${partie.joueur.pointsDeVie}/${partie.joueur.pointsDeVieMax}');
      console.ecrire('${occupant.nom.padRight(8)} ${occupant.barreVie} '
          '${occupant.pointsDeVie}/${occupant.pointsDeVieMax}');
      console.sauterLigne();
      console.ecrire('1. Attaquer');
      console.ecrire('2. Boire une potion');
      console.ecrire('3. Fuir');

      final int? choix =
          console.lireEntier('Votre choix > ', minimum: 1, maximum: 3);
      if (choix == null) return;

      // lireEntier garantit 1 <= choix <= 3 : l'index est toujours valide.
      final ActionCombat action = ActionCombat.values[choix - 1];
      final ResultatTour resultat =
          moteur.jouerTour(partie.joueur, occupant, action);

      console.sauterLigne();
      for (final String ligne in resultat.journal) {
        console.ecrire(ligne);
      }
      combatEnCours = !resultat.combatTermine;
    }

    if (occupant.estMort) {
      _recompenser(partie, salle, occupant);
    }
  }

  void _recompenser(Partie partie, Salle salle, Ennemi ennemi) {
    final Joueur joueur = partie.joueur;
    joueur.ennemisVaincus++;

    console.sauterLigne();
    for (final String ligne in joueur.gagnerExperience(ennemi.experienceDonnee)) {
      console.ecrire(ligne);
    }

    joueur.or += ennemi.orDonne;
    console.ecrire('${joueur.nom} récupère ${ennemi.orDonne} pièces d\'or.');

    final Objet? butin = ennemi.butin;
    if (butin != null) {
      _tenterDeRamasser(partie, salle, butin);
    }

    _marquerSiNettoyee(salle);

    if (ennemi.estBoss) {
      partie.terminee = true;
      partie.victoire = true;
    }
  }

  // ---------------------------------------------------------------
  // Inventaire, déplacement, sauvegarde, fin
  // ---------------------------------------------------------------

  void _ouvrirInventaire(Partie partie) {
    final Joueur joueur = partie.joueur;
    final List<Objet> objets = joueur.inventaire.objets;

    console.sauterLigne();
    console.ecrire('--- Inventaire (${joueur.inventaire.taille}/'
        '${joueur.inventaire.capacite}) ---');

    if (objets.isEmpty) {
      console.ecrire('Le sac est vide.');
      return;
    }
    for (int i = 0; i < objets.length; i++) {
      console.ecrire('${i + 1}. ${objets[i]}');
    }

    console.sauterLigne();
    console.ecrire('Que faire ?');
    console.ecrire('1. Utiliser un objet');
    console.ecrire('0. Retour');

    final int? choix =
        console.lireEntier('Votre choix > ', minimum: 0, maximum: 1);
    if (choix == null || choix == 0) return;

    final int? emplacement = console.lireEntier(
      'Quel emplacement ? > ',
      minimum: 1,
      maximum: objets.length,
    );
    if (emplacement == null) return;

    console.sauterLigne();
    console.ecrire(joueur.utiliser(emplacement - 1));
  }

  void _avancer(Partie partie) {
    final Salle salle = partie.salleCourante;
    if (salle.contientUnEnnemiVivant) {
      throw ActionImpossibleException(
          '${salle.ennemi?.nom ?? 'Une créature'} vous barre la route.');
    }
    if (partie.estDerniereSalle) {
      throw ActionImpossibleException('il n\'y a pas de salle après celle-ci.');
    }
    partie.joueur.salleCourante++;
    partie.salleCourante.visitee = true;

    console.sauterLigne();
    console.ecrire('Vous passez dans la salle suivante.');
  }

  Future<void> _sauvegarder(Partie partie) async {
    try {
      await sauvegarde.sauvegarder(partie);
      console.ecrire('Partie sauvegardée dans ${sauvegarde.chemin}.');
    } on FileSystemException catch (erreur) {
      console.ecrire('Écriture impossible : ${erreur.message}');
    }
  }

  void _afficherFin(Partie partie) {
    console.sauterLigne();
    console.titre(partie.victoire ? 'VICTOIRE' : 'FIN DE LA PARTIE');
    console.sauterLigne();

    if (partie.victoire) {
      console.ecrire('Kraghar s\'effondre. Le donjon est à vous.');
    } else if (partie.joueur.estMort) {
      console.ecrire('${partie.joueur.nom} ne se relèvera pas.');
    } else {
      console.ecrire('Vous quittez le donjon. Il vous attendra.');
    }

    console.sauterLigne();
    console.ecrire(Statistiques(partie).rapport);
  }
}
```

---

### `bin/main.dart`

```dart
import 'package:donjon_de_dart/jeu.dart';

Future<void> main(List<String> arguments) async {
  final String chemin =
      arguments.isNotEmpty ? arguments.first : 'sauvegardes/partie.json';

  final Jeu jeu = Jeu(cheminSauvegarde: chemin);
  await jeu.demarrer();
}
```

---

### `test/donjon_de_dart_test.dart`

Version sans les commentaires pédagogiques de la section 18.26 :

```dart
import 'dart:convert';
import 'dart:io';
import 'dart:math';

import 'package:donjon_de_dart/exceptions/erreurs_jeu.dart';
import 'package:donjon_de_dart/models/inventaire.dart';
import 'package:donjon_de_dart/models/joueur.dart';
import 'package:donjon_de_dart/models/objet.dart';
import 'package:donjon_de_dart/services/moteur_combat.dart';
import 'package:donjon_de_dart/services/sauvegarde_service.dart';
import 'package:test/test.dart';

void main() {
  test('un inventaire plein lève InventairePleinException', () {
    final Inventaire sac = Inventaire(capacite: 2);
    sac.ajouter(Catalogue.potionMineure);
    sac.ajouter(Catalogue.potionMajeure);

    expect(sac.estPlein, isTrue);
    expect(
      () => sac.ajouter(Catalogue.epeeRouillee),
      throwsA(isA<InventairePleinException>()),
    );
    expect(sac.taille, equals(2));
  });

  test('un gros gain d\'expérience fait monter plusieurs niveaux', () {
    final Joueur joueur = Joueur(nom: 'Testeur');

    joueur.gagnerExperience(310);

    expect(joueur.niveau, equals(3));
    expect(joueur.experience, equals(10));
    expect(joueur.pointsDeVieMax, equals(140));
    expect(joueur.attaque, equals(18));
  });

  test('les dégâts ont toujours un plancher de 1', () {
    final MoteurCombat moteur = MoteurCombat(hasard: Random(42));

    for (int i = 0; i < 100; i++) {
      final int degats =
          moteur.calculerDegats(attaque: 1, defense: 10000, critique: false);
      expect(degats, greaterThanOrEqualTo(MoteurCombat.degatsMinimum));
    }
  });

  test('un joueur sérialisé puis désérialisé est identique', () {
    final Joueur avant = Joueur(nom: 'Alex', niveau: 3, or: 250);
    avant.armeEquipee = Catalogue.lameDeFeu;
    avant.inventaire.ajouter(Catalogue.potionMajeure);
    avant.subirDegats(37);

    final Joueur apres =
        Joueur.fromJson(jsonDecode(jsonEncode(avant.toJson())));

    expect(apres.nom, equals(avant.nom));
    expect(apres.niveau, equals(avant.niveau));
    expect(apres.or, equals(avant.or));
    expect(apres.pointsDeVie, equals(avant.pointsDeVie));
    expect(apres.attaqueTotale, equals(avant.attaqueTotale));
    expect(apres.inventaire.taille, equals(1));
    expect(apres.inventaire.objets.first, equals(Catalogue.potionMajeure));
  });

  test('une sauvegarde corrompue lève SauvegardeCorrompueException', () async {
    const String chemin = 'sauvegardes/test_corrompu.json';
    final SauvegardeService service = SauvegardeService(chemin: chemin);
    final File fichier = File(chemin);

    try {
      await fichier.parent.create(recursive: true);
      await fichier.writeAsString('{ ceci n\'est pas du JSON valide');

      expect(
        () => service.charger(),
        throwsA(isA<SauvegardeCorrompueException>()),
      );
    } finally {
      if (await fichier.exists()) await fichier.delete();
    }
  });
}
```

---

### Vérification finale

Une fois les dix-neuf fichiers en place :

```bash
dart pub get
dart analyze
dart test
dart run bin/main.dart
```

**Résultat attendu :**

```text
$ dart analyze
Analyzing donjon_de_dart...
No issues found!

$ dart test
00:01 +5: All tests passed!
```

Si `dart analyze` signale quelque chose, corrigez-le **avant** de lancer le jeu. Un avertissement d'analyseur est presque toujours un bug qui ne s'est pas encore montré.

---

## 18.30 — Erreurs fréquentes

Voici les erreurs que ce projet précis provoque le plus souvent. Chacune a été rencontrée par des dizaines d'élèves avant vous.

| Erreur | Cause | Correction |
| --- | --- | --- |
| `FileSystemException: Cannot open file` à la première sauvegarde | Le dossier `sauvegardes/` n'existe pas | `await fichier.parent.create(recursive: true);` avant l'écriture |
| Après un chargement, `inventaire.retirer(potion)` ne trouve rien | `==` compare les identités, or les objets rechargés sont de nouvelles instances | Redéfinir `operator ==` **et** `hashCode` dans `Objet` |
| `The property 'estVivant' can't be unconditionally accessed` | Dart ne promeut pas un **champ** de classe après `!= null` | Copier le champ dans une variable locale `final` avant de le tester |
| Le sac dépasse sa capacité | Le getter renvoie la vraie liste : `List<Objet> get objets => _objets;` | Renvoyer `List<Objet>.unmodifiable(_objets)` et copier dans le constructeur |
| Le joueur reste au niveau 2 après avoir tué le boss | `if` au lieu de `while` dans `gagnerExperience` | Utiliser `while (experience >= experienceRequise)` |
| L'expérience en trop est perdue à chaque niveau | `experience = 0` après la montée | Utiliser `experience -= experienceRequise` |
| Un ennemi à 0 PV riposte encore | La riposte est calculée avant le test `ennemi.estMort` | Tester la mort de l'ennemi **avant** d'appeler sa riposte |
| Le combat ne se termine jamais | Les deux camps infligent 0 dégât (défenses trop hautes) | Garder le plancher `degatsMinimum = 1` dans `calculerDegats` |
| `RangeError` sur `ActionCombat.values[choix - 1]` | La saisie n'a pas été bornée | Passer `minimum: 1, maximum: 3` à `lireEntier` |
| Le programme boucle à l'infini et sature le processeur | `stdin.readLineSync()` renvoie `null` (entrée fermée) et le `null` est ignoré | Sortir de la boucle dès que `lireLigne` renvoie `null` |
| `FormatException` quand le joueur tape « deux » | Utilisation de `int.parse` sur une saisie utilisateur | Utiliser `int.tryParse` et tester le `null` |
| Le jeu s'arrête sur un sac plein | L'exception n'est rattrapée nulle part | Entourer l'action d'un `try` / `on InventairePleinException` |
| Un chargement plante avec `type 'Null' is not a subtype of 'String'` | Le JSON est lu sans vérification de type | Passer par `entierDepuis`, `texteDepuis`, `sousObjetDepuis` |
| Le jeu affiche `Instance of 'Objet'` | `toString()` non redéfini | Redéfinir `toString()` dans `Objet` |
| `A value of type 'Null' can't be returned` dans `fromJson` | Le `factory` ne renvoie rien sur un des chemins | Lever une `SauvegardeCorrompueException` plutôt que renvoyer `null` |
| Les libellés traduits cassent les anciennes sauvegardes | On a sérialisé `type.libelle` au lieu de `type.name` | Ne jamais sérialiser de l'affichage |
| `Undefined name 'Catalogue'` dans `ennemi.dart` | Import oublié | `import 'objet.dart';` en haut du fichier |
| Import circulaire entre `joueur.dart` et `moteur_combat.dart` | Un modèle importe un service | Les services connaissent les modèles, jamais l'inverse |

---

## 18.31 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Analyse | Les noms communs du cahier des charges deviennent des classes ; les verbes deviennent des services. |
| Modélisation | On dessine le diagramme **avant** d'écrire la première ligne, et on s'y tient. |
| Découpage | `bin/` pour le point d'entrée, `lib/models/` pour les données, `lib/services/` pour les actions. |
| Classe abstraite | `Personnage` factorise ce qui est commun et **impose** `description` et `toJson`. |
| Mixin | `Soignable on Personnage` ajoute une capacité au joueur seul, sans niveau d'héritage supplémentaire. |
| Encapsulation | Champ privé + getter `unmodifiable` + copie dans le constructeur : les trois vont ensemble. |
| Égalité | Redéfinir `==` sans `hashCode` est un bug garanti. |
| Null safety | Un champ de classe ne se promeut pas : passez par une variable locale `final`. |
| Exceptions | Une exception pour l'anormal, un `null` pour le prévu. Une classe de base commune pour un `catch` unique. |
| JSON | `toJson` est une méthode, `fromJson` un `factory`. La sérialisation est récursive et défensive. |
| Asynchrone | Seul le disque justifie `async` ici, mais il le justifie totalement. |
| Saisies | `int.tryParse` + bornes + compteur d'essais : trois lignes qui rendent le programme increvable. |
| Boucles | Une boucle, une condition de sortie, une méthode. |
| Collections | `where`, `map`, `fold`, `every`, `join` remplacent la quasi-totalité des `for`. |
| Tests | Testez les **règles**, pas les méthodes ; neutralisez le hasard par une graine. |
| Séparation | Un service renvoie des données ; seule la couche console affiche. C'est ce qui rendra le portage Flutter indolore. |

---

## 18.32 — Exercices d'extension

Les dix défis suivants reprennent les fonctionnalités optionnelles du cahier des charges. Chacun est réalisable avec les seules notions de la partie 1A. Traitez-les dans l'ordre : la difficulté est croissante.

### Défi 1 — La boutique du marchand (facile)

Ajoutez une salle sans ennemi tenue par un marchand. Le joueur peut y vendre les objets de type `tresor` au prix de leur `valeur`, et racheter des potions.

**Indication de solution :** créez `lib/services/boutique.dart` avec une classe `Boutique` prenant un `Joueur`. Pour vendre : `final Objet objet = joueur.inventaire.retirerA(index); joueur.or += objet.valeur;`. Pour acheter : vérifiez `joueur.or >= prix` **avant** d'appeler `joueur.ramasser`, et rattrapez `InventairePleinException` pour rembourser le joueur si le sac déborde. N'ajoutez aucun champ à `Joueur` : tout tient dans le service.

### Défi 2 — Les niveaux de difficulté (facile)

Proposez trois difficultés au lancement : facile, normal, difficile. Elles multiplient les points de vie et l'attaque des ennemis.

**Indication de solution :** créez `enum Difficulte { facile('Facile', 0.75), normal('Normal', 1.0), difficile('Difficile', 1.4) }` avec un champ `multiplicateur`. Dans `Donjon.parDefaut()`, ajoutez un paramètre `{Difficulte difficulte = Difficulte.normal}` et construisez les ennemis avec le constructeur principal plutôt que les constructeurs nommés : `Ennemi(nom: 'Gobelin', pointsDeVieMax: (30 * difficulte.multiplicateur).round(), ...)`. Sérialisez le nom de la difficulté dans `Partie.toJson` pour la retrouver au chargement.

### Défi 3 — Trois emplacements de sauvegarde (facile)

Remplacez le fichier unique par trois emplacements, avec un menu qui affiche le contenu de chacun.

**Indication de solution :** `SauvegardeService` prend déjà un `chemin` en paramètre. Instanciez-en trois : `SauvegardeService(chemin: 'sauvegardes/emplacement_$i.json')`. Pour l'aperçu, ajoutez une méthode `Future<String> resume()` qui lit le fichier, décode le JSON et renvoie `'Alex — niveau 3 — tour 12'` en cas de succès, `'(vide)'` si `existe()` renvoie `false`, et `'(corrompu)'` sur `SauvegardeCorrompueException`.

### Défi 4 — Le journal de la partie (facile)

Conservez les cinquante dernières lignes du récit et affichez-les à la demande.

**Indication de solution :** ajoutez `final List<String> journal = <String>[];` à `Partie`, et une méthode `void noter(String ligne) { journal.add(ligne); if (journal.length > 50) journal.removeAt(0); }`. Faites passer tous les `console.ecrire` du combat par cette méthode. Sérialisez la liste avec `'journal': journal` et relisez-la avec `sousListeDepuis`... attention, ce sont des `String`, pas des `Map` : écrivez une petite boucle `whereType<String>()`.

### Défi 5 — Les sorts et les points de magie (intermédiaire)

Donnez au joueur une réserve de magie et trois sorts : `Éclair` (dégâts), `Soin` (points de vie), `Bouclier` (défense temporaire).

**Indication de solution :** créez `enum Sort { eclair('Éclair', 12, 25), soin('Soin', 8, 40), bouclier('Bouclier', 10, 8) }` avec `coutMagie` et `puissance`. Ajoutez `int magie` et `int magieMax` à `Joueur`, ainsi qu'une valeur `ActionCombat.sort`. Dans `MoteurCombat.jouerTour`, ajoutez un `case ActionCombat.sort` qui vérifie `joueur.magie >= sort.coutMagie` et lève `ActionImpossibleException` sinon. Le bouclier est le plus délicat : stockez `int bonusDefenseTemporaire` sur le joueur et remettez-le à zéro à la fin du combat.

### Défi 6 — Le boss à deux phases (intermédiaire)

Sous 50 % de points de vie, Kraghar entre en rage : son attaque augmente de moitié et il annonce le changement.

**Indication de solution :** ajoutez `bool enrage = false;` à `Ennemi` et un getter `bool get devraitEnrager => estBoss && !enrage && pointsDeVie <= pointsDeVieMax ~/ 2;`. Dans `jouerTour`, juste après l'attaque du joueur, testez ce getter : si vrai, faites `ennemi.enrage = true; ennemi.attaque = (ennemi.attaque * 1.5).round();` et ajoutez une ligne au journal. Pensez à sérialiser `enrage`, sinon une sauvegarde rechargée en pleine phase 2 ferait rager le boss une deuxième fois.

### Défi 7 — Le donjon généré aléatoirement (intermédiaire)

Remplacez `Donjon.parDefaut()` par `Donjon.aleatoire(int nombreDeSalles, {int? graine})`.

**Indication de solution :** `factory Donjon.aleatoire(int n, {int? graine})` crée un `Random(graine)`, tire un nom de salle dans une `List<String> const`, tire un ennemi dans une liste de fabriques `List<Ennemi Function()> fabriques = [Ennemi.gobelin, Ennemi.squelette, Ennemi.golem]`, puis force `Ennemi.boss()` dans la dernière salle. Passez la graine dans le JSON : deux joueurs avec la même graine explorent le même donjon, ce qui est très utile pour comparer des scores.

### Défi 8 — Les objets à usage limité (intermédiaire)

Certaines armes se brisent après un nombre d'utilisations.

**Indication de solution :** `Objet` est immuable, ne le cassez pas. Créez plutôt une classe `Equipement { Equipement(this.objet, this.usagesRestants); final Objet objet; int usagesRestants; }` et remplacez `Objet? armeEquipee` par `Equipement? armeEquipee` dans `Joueur`. Décrémentez `usagesRestants` dans `MoteurCombat.attaquer` quand l'attaquant est un `Joueur`, et remettez `armeEquipee = null` à zéro usage. C'est un bon exercice de **composition** : on ajoute un état autour d'une donnée immuable plutôt que de la rendre mutable.

### Défi 9 — Le tableau des scores (avancé)

Enregistrez chaque fin de partie dans `sauvegardes/scores.json` et affichez le classement des dix meilleurs.

**Indication de solution :** créez `class Score { final String nom; final int niveau; final int or; final int tours; final bool victoire; final DateTime date; }` avec `toJson` / `fromJson`. Le fichier contient une **liste** JSON, pas un objet : `jsonDecode` renvoie une `List<dynamic>`, filtrez-la avec `whereType<Map<String, dynamic>>()`. Pour le classement : `scores.sort((a, b) => b.or.compareTo(a.or));` puis `scores.take(10)`. Calculez le score total avec un `fold` : `or + niveau * 100 + (victoire ? 500 : 0)`.

### Défi 10 — Le mode démonstration automatique (avancé)

Faites jouer l'ordinateur tout seul, sans clavier, pour tester l'équilibrage sur mille parties.

**Indication de solution :** c'est le défi qui valide toute l'architecture. Créez `class ConsoleSimulee implements Console` (ou faites de `Console` une classe abstraite avec deux implémentations) : `lireLigne` renvoie les valeurs d'une file préremplie, `ecrire` n'affiche rien. Ajoutez une stratégie très simple : boire une potion sous 30 % de points de vie, attaquer sinon. Lancez mille parties avec des graines différentes et faites un `fold` sur les résultats pour obtenir le taux de victoire. Si ce taux est de 100 % ou de 0 %, votre équilibrage est à revoir. Si vous y arrivez sans modifier une seule ligne de `MoteurCombat`, c'est la preuve que la séparation modèles / services / affichage était juste.

---

## Et maintenant ?

Vous venez de terminer la **PARTIE 1A — DART**.

Regardez le chemin parcouru. Au chapitre 01, vous affichiez `Bonjour` dans une console. Au chapitre 18, vous livrez une application de près de deux mille lignes, structurée en dix-neuf fichiers, testée, capable de sauvegarder son état sur disque et de le relire sans jamais planter.

Entre les deux, vous avez acquis, dans l'ordre :

```text
  01-07   les fondations      variables, types, conditions,
                              boucles, collections, fonctions
  08-11   la POO              classes, constructeurs, encapsulation,
                              héritage, polymorphisme, abstract,
                              mixins, enums
  12-13   la robustesse       null safety, exceptions
  14-15   l'élégance          programmation fonctionnelle, asynchrone
  16-17   le professionnalisme  organisation d'un projet, JSON
  18      l'assemblage        un vrai logiciel, de bout en bout
```

Ces notions ne sont pas des cases à cocher. Ce sont les mêmes que celles qu'utilise, tous les jours, un développeur Flutter en poste. Vous les possédez maintenant.

**Une remarque avant de tourner la page.** Le chapitre 18 vous a fait écrire du code que vous ne relirez peut-être jamais. Ce n'est pas grave. Ce que vous garderez, ce n'est pas `MoteurCombat` : c'est le réflexe d'analyser un besoin, de nommer des entités, de séparer les données des actions, et de refuser qu'une saisie utilisateur fasse tomber un programme. Ce réflexe-là ne s'oublie pas.

### La suite : PARTIE 1B — FLUTTER

Vous allez maintenant sortir de la console.

Dans la partie 1B, la même logique reste valable, mais l'affichage change du tout au tout. Là où vous écriviez :

```dart
console.ecrire('${joueur.nom}  ${joueur.barreVie}  ${joueur.pointsDeVie} PV');
```

vous écrirez un **widget** : un objet Dart, exactement comme vos classes, qui décrit un morceau d'écran.

Vous découvrirez notamment :

| Chapitre à venir | Sujet |
| --- | --- |
| Installation et premier projet | le SDK Flutter, `flutter create`, l'émulateur |
| Tout est widget | `StatelessWidget`, `StatefulWidget`, l'arbre de widgets |
| La mise en page | `Row`, `Column`, `Stack`, `Expanded`, `Padding` |
| L'état | `setState`, le cycle de vie d'un widget |
| Les interactions | boutons, formulaires, gestes |
| La navigation | passer d'un écran à l'autre, transmettre des données |
| Les listes | `ListView.builder`, affichage d'un inventaire réel |
| L'asynchrone à l'écran | `FutureBuilder`, chargement d'une sauvegarde |
| La persistance | fichiers, préférences, JSON — vos `toJson` reviendront intacts |

Et surtout : le « Donjon de Dart » que vous venez d'écrire **reviendra**. Ses modèles, ses services, ses exceptions et sa sérialisation seront réutilisés tels quels. Seule la classe `Console` disparaîtra, remplacée par des widgets. C'est précisément pour cette raison que nous avons interdit à `MoteurCombat` d'appeler `print` : ce jour-là, vous mesurerez ce que cette discipline vous fait gagner.

Vous n'avez plus rien à apprendre du langage. Vous allez apprendre à le faire voir.

Rendez-vous dans la **PARTIE 1B — FLUTTER**, chapitre 19 : [19-PARTIE-1B—INSTALLER-FLUTTER-ET-CRÉER-SON-PREMIER-PROJET.md](19-PARTIE-1B—INSTALLER-FLUTTER-ET-CRÉER-SON-PREMIER-PROJET.md)
