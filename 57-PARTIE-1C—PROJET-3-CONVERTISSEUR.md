# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 57 — PROJET 3 : LE CONVERTISSEUR D'UNITÉS

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** chapitre 56 — Projet 2 : la calculatrice
> **Ce que vous saurez faire à la fin :** construire une application complète pilotée par un formulaire, séparer proprement une logique métier testable de son interface, et écrire les tests unitaires qui prouvent que vos calculs sont justes.

---

## 57.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- modéliser un domaine métier avec un `enum` enrichi et une classe de valeur ;
- expliquer ce qu'est une **unité pivot** et pourquoi elle divise le travail par cent ;
- démontrer pourquoi la température **ne peut pas** se convertir avec un simple facteur ;
- généraliser un modèle « facteur » en modèle « couple de fonctions » ;
- écrire une fonction de conversion **pure**, sans aucune dépendance à Flutter ;
- brancher un `TextField` sur un `TextEditingController` et réagir à chaque frappe ;
- restreindre la saisie aux caractères numériques avec `inputFormatters` ;
- distinguer ce que filtre un `inputFormatter` de ce que valide `double.tryParse` ;
- gérer la virgule décimale française et les espaces de groupement ;
- construire deux `DropdownButton<Unite>` synchronisés ;
- filtrer la liste des unités selon la catégorie choisie avec `where` ;
- inverser l'unité source et l'unité cible en un seul appel à `setState` ;
- convertir en temps réel via `onChanged` sans bouton « Calculer » ;
- formater un résultat lisible avec `NumberFormat` du paquet `intl` ;
- choisir un nombre de décimales adapté à l'ordre de grandeur du résultat ;
- basculer en notation scientifique pour les nombres extrêmes ;
- organiser six catégories en onglets avec `TabController` et `TabBarView` ;
- conserver l'état de chaque onglet avec `AutomaticKeepAliveClientMixin` ;
- tenir un historique des conversions validées ;
- persister le dernier choix de l'utilisateur avec `shared_preferences` ;
- écrire une suite de tests unitaires qui vérifie chaque famille de conversion.

---

## 57.1 — Aperçu du résultat final

Voici l'écran principal, tel qu'il sera à la fin du chapitre.

```text
┌──────────────────────────────────────────────────┐
│  Convertisseur                        [hist]     │  <- AppBar + historique
├──────────────────────────────────────────────────┤
│  Longueur │ Masse │ Température │ Volume │ Don…  │  <- TabBar défilante
│  ━━━━━━━━                                        │
├──────────────────────────────────────────────────┤
│                                                  │
│   Valeur à convertir                             │
│  ┌────────────────────────────────────────────┐  │
│  │ 42,195                                  x │  │  <- TextField
│  └────────────────────────────────────────────┘  │
│                                                  │
│   De                                             │
│  ┌────────────────────────────────────────────┐  │
│  │ Kilomètre (km)                          v  │  │  <- DropdownButton
│  └────────────────────────────────────────────┘  │
│                                                  │
│                   ┌──────────┐                   │
│                   │    swap     │                   │  <- bouton d'inversion
│                   └──────────┘                   │
│                                                  │
│   Vers                                           │
│  ┌────────────────────────────────────────────┐  │
│  │ Mile (mi)                               v  │  │  <- DropdownButton
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │                                            │  │
│  │            26,2188 mi                      │  │  <- carte de résultat
│  │                                            │  │
│  │  42,195 km = 26,2188 mi                    │  │
│  │                            [ Enregistrer ] │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│   1 km = 0,621371 mi                             │  <- taux unitaire
│                                                  │
└──────────────────────────────────────────────────┘
```

Et l'écran d'historique, atteint par l'icône de l'`AppBar` :

```text
┌──────────────────────────────────────────────────┐
│  <  Historique                        [vider]     │
├──────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────┐  │
│  │ 42,195 km  =  26,2188 mi                   │  │
│  │ Longueur — il y a 2 minutes                │  │
│  ├────────────────────────────────────────────┤  │
│  │ 100 °F  =  37,7778 °C                      │  │
│  │ Température — il y a 5 minutes             │  │
│  ├────────────────────────────────────────────┤  │
│  │ 70 kg  =  154,3236 lb                      │  │
│  │ Masse — il y a 8 minutes                   │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
└──────────────────────────────────────────────────┘
```

Rien d'impressionnant à l'œil. C'est voulu.

L'intérêt de ce projet n'est pas graphique : il est **architectural**. Un convertisseur est le plus petit programme qui vous oblige à répondre honnêtement à trois questions difficiles.

1. Comment représenter un domaine (les unités) sans écrire trois cents fonctions ?
2. Comment isoler le calcul de l'affichage, pour pouvoir le tester ?
3. Comment transformer une chaîne de caractères tapée par un humain en un nombre sûr ?

Ces trois questions reviendront dans **tous** vos projets futurs. Autant les traiter maintenant sur un sujet où la vérité est vérifiable : 1 pouce vaut 2,54 cm, ce n'est pas une opinion.

---

## 57.2 — Cahier des charges

### 57.2.1 — Fonctionnalités obligatoires

| # | Exigence | Critère d'acceptation |
| --- | --- | --- |
| O1 | Six catégories : longueur, masse, température, volume, données, devises | Six onglets accessibles |
| O2 | Au moins six unités par catégorie | Les listes déroulantes en proposent six ou plus |
| O3 | Saisie d'une valeur numérique | Le champ n'accepte que chiffres, séparateur décimal et signe moins |
| O4 | Choix d'une unité source et d'une unité cible | Deux `DropdownButton` distincts |
| O5 | Les unités proposées appartiennent à la catégorie active | Impossible de convertir des kilomètres en litres |
| O6 | Conversion en temps réel | Le résultat change à chaque frappe, sans bouton « Calculer » |
| O7 | Bouton d'inversion des deux unités | Un appui échange source et cible et met le résultat à jour |
| O8 | Gestion de la température | 100 °F donne 37,7778 °C, pas une multiplication |
| O9 | Saisie invalide signalée, jamais de plantage | Un champ vide ou `-` seul affiche un message, pas une exception |
| O10 | Résultat formaté lisiblement | Séparateurs de milliers, décimales adaptées |
| O11 | Historique des conversions enregistrées | Écran dédié, effaçable |
| O12 | Mémorisation du dernier choix | Au redémarrage, on retrouve sa catégorie et ses unités |
| O13 | Tests unitaires des conversions | `flutter test` passe au vert |

### 57.2.2 — Bonus

| # | Bonus |
| --- | --- |
| B1 | Affichage du taux unitaire (« 1 km = 0,621371 mi ») |
| B2 | Copie du résultat dans le presse-papiers |
| B3 | Notation scientifique automatique pour les nombres extrêmes |
| B4 | Mode sombre suivant le thème du système |
| B5 | Persistance de l'historique entre deux lancements |

### 57.2.3 — Hors périmètre

On ne fait **pas** :

- de taux de change en temps réel (cela demande une API, ce sera le chapitre 61) ;
- de conversion entre catégories différentes (des kilogrammes en litres n'a de sens que si l'on connaît une densité) ;
- de gestion des unités composées (km/h, m/s²) — c'est le défi 9.

Les devises du projet sont **des taux fixes et définitifs** : les taux de conversion irrévocables des anciennes monnaies de la zone euro. Ce sont des constantes légales, pas des cours de marché. Elles ne bougeront plus jamais, ce qui en fait un excellent jeu de données pour un exercice.

---

## 57.3 — Notions mobilisées

| Notion | Chapitre d'origine |
| --- | --- |
| `enum` enrichi, membres et constructeur d'énumération | 11 — POO avancée |
| Classe de valeur, `==` et `hashCode` | 09, 10 |
| Fonctions de premier ordre stockées dans un champ | 07 — Les fonctions |
| `int.tryParse`, `double.tryParse`, absence d'exception | 13 — Exceptions |
| `where`, `map`, `firstWhere`, `fold` | 14 — Programmation fonctionnelle |
| Types nullables, `?`, `??`, `!` | 12 — Null Safety |
| `pubspec.yaml`, ajout d'un paquet | 16 — Organisation d'un projet |
| Sérialisation JSON pour l'historique | 17 — JSON |
| `StatefulWidget`, `setState`, `initState`, `dispose` | 45 |
| `Column`, `Row`, `Expanded`, `Padding`, `Card` | 46 |
| `TextField`, `TextEditingController`, validation | 49 |
| `TabBar`, `TabBarView`, `Navigator.push` | 50 |
| `ThemeData`, Material 3 | 51 |
| `shared_preferences` | 54 |

Deux nouveautés seulement dans ce chapitre : `inputFormatters` (paquet `services` de Flutter) et `NumberFormat` (paquet `intl`). Tout le reste, vous le connaissez déjà. C'est le principe d'un chapitre-projet : **assembler**, pas apprendre dix API de plus.

---

## 57.4 — Arborescence du projet

```text
convertisseur/
├── pubspec.yaml
├── lib/
│   ├── main.dart                       point d'entrée, thème, chargement des préférences
│   ├── modeles/
│   │   ├── categorie.dart              l'enum enrichi des six catégories
│   │   ├── unite.dart                  la classe Unite (nom, symbole, fonctions)
│   │   └── conversion.dart             une ligne d'historique
│   ├── donnees/
│   │   └── catalogue.dart              toutes les unités de toutes les catégories
│   ├── logique/
│   │   ├── convertisseur.dart          la fonction pure convertir()
│   │   ├── lecture_nombre.dart         texte saisi -> double?
│   │   └── formatage.dart              double -> texte lisible (intl)
│   ├── services/
│   │   └── preferences_service.dart    lecture/écriture du dernier choix
│   ├── ecrans/
│   │   ├── ecran_accueil.dart          les onglets
│   │   ├── page_categorie.dart         un convertisseur pour une catégorie
│   │   └── ecran_historique.dart       la liste des conversions
│   └── widgets/
│       ├── champ_valeur.dart           le TextField numérique
│       ├── selecteur_unite.dart        un DropdownButton<Unite>
│       └── carte_resultat.dart         l'encadré du résultat
└── test/
    ├── convertisseur_test.dart         les conversions
    ├── lecture_nombre_test.dart        le parsing
    └── formatage_test.dart             l'affichage
```

Observez la ligne de partage. Les dossiers `modeles`, `donnees` et `logique` **n'importent jamais Flutter**. Ils n'importent que `dart:core` et, pour le formatage, `package:intl`.

Conséquence directe : ces fichiers sont testables en une milliseconde, sans émulateur, sans widget, sans rendu. C'est là que vit toute la difficulté du projet, et c'est là que les tests iront. Les dossiers `ecrans` et `widgets` ne font que de l'affichage et du branchement.

Retenez cette phrase :

> Si votre logique métier importe `package:flutter/material.dart`, vous ne pourrez pas la tester correctement.

---

## 57.5 — Étape 1 : le projet vide qui démarre

### 57.5.1 — Création

```text
flutter create convertisseur
cd convertisseur
flutter pub add intl
flutter pub add shared_preferences
```

Nous utilisons `flutter pub add` plutôt qu'un numéro de version écrit à la main : la commande va chercher la dernière version compatible sur pub.dev et l'écrit elle-même dans le `pubspec.yaml`. À la rédaction de ce chapitre, `intl` est en 0.20.3 et `shared_preferences` en 2.5.5, mais **ne recopiez pas ces numéros** : lancez la commande.

### 57.5.2 — Le fichier `pubspec.yaml`

**Fichier : `pubspec.yaml`**

```yaml
name: convertisseur
description: Un convertisseur d'unites multi-categories.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter
  intl: ^0.20.3
  shared_preferences: ^2.5.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

Aucune section `assets`. Ce projet ne contient **aucun fichier image** : tout est fait avec des `Icon` et des couleurs du thème.

### 57.5.3 — Le point d'entrée provisoire

**Fichier : `lib/main.dart`**

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationConvertisseur());
}

/// Racine de l'application.
///
/// Pour l'instant elle n'affiche qu'un ecran vide : nous validons
/// simplement que le projet compile et se lance.
class ApplicationConvertisseur extends StatelessWidget {
  const ApplicationConvertisseur({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Convertisseur',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Convertisseur')),
        body: const Center(child: Text('Bientot un convertisseur.')),
      ),
    );
  }
}
```

**État exécutable.** `flutter run` affiche une page blanche avec un titre. C'est peu, mais c'est **compilable**, et chaque étape suivante restera compilable. Ne passez jamais à l'étape suivante avec une compilation cassée.

---

## 57.6 — Étape 2 : modéliser une catégorie avec un `enum` enrichi

### 57.6.1 — Pourquoi un `enum` et pas des chaînes

Un débutant écrit ceci :

```dart
String categorie = 'longueur';
```

Puis, trois jours plus tard, il écrit `'Longueur'` ailleurs, et plus rien ne fonctionne. Le compilateur ne l'a pas prévenu, parce que `'Longueur'` est une chaîne parfaitement valide.

Un `enum` supprime le problème à la racine : la liste des valeurs possibles est **fermée**, et le compilateur refuse tout ce qui n'y est pas.

Au chapitre 11, vous avez vu que Dart permet des `enum` **enrichis** : chaque valeur peut porter des champs, avoir un constructeur `const`, et même des méthodes. Nous allons nous en servir.

### 57.6.2 — Le code

**Fichier : `lib/modeles/categorie.dart`**

```dart
/// Les six familles d'unites gerees par l'application.
///
/// C'est un `enum` enrichi (chapitre 11) : chaque valeur porte un libelle
/// affichable, le symbole de son unite pivot, et un identifiant stable
/// utilise pour la persistance.
///
/// Ce fichier n'importe PAS Flutter : il doit rester testable sans widget.
enum Categorie {
  longueur(
    identifiant: 'longueur',
    libelle: 'Longueur',
    symbolePivot: 'm',
    description: 'Distances, hauteurs, tailles.',
  ),
  masse(
    identifiant: 'masse',
    libelle: 'Masse',
    symbolePivot: 'kg',
    description: 'Poids au sens courant du terme.',
  ),
  temperature(
    identifiant: 'temperature',
    libelle: 'Temperature',
    symbolePivot: '°C',
    description: 'Le seul cas qui ne se convertit pas par un facteur.',
  ),
  volume(
    identifiant: 'volume',
    libelle: 'Volume',
    symbolePivot: 'L',
    description: 'Capacites, contenances, recettes de cuisine.',
  ),
  donnees(
    identifiant: 'donnees',
    libelle: 'Donnees',
    symbolePivot: 'o',
    description: 'Octets decimaux et octets binaires.',
  ),
  devise(
    identifiant: 'devise',
    libelle: 'Devises',
    symbolePivot: 'EUR',
    description: 'Taux fixes et definitifs de la zone euro.',
  );

  /// Constructeur `const` de l'enumeration.
  ///
  /// Il ne peut etre appele que dans la liste des valeurs, ci-dessus.
  const Categorie({
    required this.identifiant,
    required this.libelle,
    required this.symbolePivot,
    required this.description,
  });

  /// Identifiant stable, sauvegarde dans les preferences.
  ///
  /// On ne sauvegarde JAMAIS `index` : si un jour on insere une categorie
  /// au milieu de la liste, tous les index enregistres deviendraient faux.
  final String identifiant;

  /// Nom affiche a l'utilisateur.
  final String libelle;

  /// Symbole de l'unite pivot de la categorie (voir etape 3).
  final String symbolePivot;

  /// Phrase d'explication, affichee en bas de page.
  final String description;

  /// Retrouve une categorie a partir de son identifiant.
  ///
  /// Renvoie `null` si l'identifiant est inconnu : c'est volontaire, cela
  /// permet a l'appelant de retomber sur une valeur par defaut (Null Safety,
  /// chapitre 12).
  static Categorie? parIdentifiant(String identifiant) {
    for (final Categorie c in Categorie.values) {
      if (c.identifiant == identifiant) {
        return c;
      }
    }
    return null;
  }
}
```

### 57.6.3 — Le piège de `index`

La méthode `parIdentifiant` mérite qu'on s'y arrête. Dart fournit gratuitement `Categorie.values[2]` et `maCategorie.index`. La tentation est grande d'enregistrer l'index dans les préférences.

Ne le faites pas. Voici pourquoi, en trois temps.

```text
VERSION 1 DE L'APPLICATION
  index 0 -> longueur
  index 1 -> masse
  index 2 -> temperature      <- l'utilisateur enregistre "2"

VERSION 2, on insere une categorie apres "masse"
  index 0 -> longueur
  index 1 -> masse
  index 2 -> surface          <- nouveau
  index 3 -> temperature

AU DEMARRAGE
  On relit "2" -> l'utilisateur atterrit sur "surface".
  Personne ne comprend pourquoi.
```

Un identifiant textuel ne souffre pas de ce problème : `'temperature'` reste `'temperature'` quel que soit l'ordre. Cette règle vaut pour **toute** donnée sortie du programme : fichier, base, réseau, préférences.

**État exécutable.** Le projet compile toujours ; le fichier n'est pas encore utilisé.

---

## 57.7 — Étape 3 : l'unité pivot et le facteur de conversion

### 57.7.1 — Le problème combinatoire

Prenons la catégorie « longueur » avec neuf unités : millimètre, centimètre, mètre, kilomètre, pouce, pied, yard, mile, mille marin.

Combien de conversions faut-il écrire pour tout couvrir ?

```text
9 unites source x 8 unites cible = 72 conversions
```

Soixante-douze. Pour une seule catégorie. Avec six catégories, on dépasse les trois cents fonctions. Chacune contenant un nombre à ne pas se tromper. C'est impossible à écrire et impossible à maintenir.

### 57.7.2 — La solution : passer par un pivot

On choisit **une** unité de référence par catégorie, appelée **unité pivot**. Pour la longueur : le mètre.

Ensuite, chaque unité ne connaît qu'**une seule chose** : combien de pivots vaut une de ses unités.

```text
CONVERSION EN DEUX TEMPS

   5 kilometres                      ? miles
        │                                ^
        │  x 1000                        │  / 1609.344
        │  (1 km = 1000 m)               │  (1 mi = 1609.344 m)
        v                                │
   5000 metres  ─────────────────────────┘
                    LE PIVOT
```

Le nombre de constantes à écrire tombe de 72 à 9. Et surtout, ces 9 constantes sont **vérifiables une par une** dans n'importe quelle table officielle.

```text
9 unites  ->  9 facteurs  au lieu de 72 fonctions
```

### 57.7.3 — Le facteur, défini précisément

Le facteur d'une unité, c'est **la valeur d'une unité exprimée dans le pivot**.

| Unité | Facteur (en mètres) | Lecture |
| --- | --- | --- |
| millimètre | 0.001 | 1 mm vaut 0,001 m |
| centimètre | 0.01 | 1 cm vaut 0,01 m |
| mètre | 1 | 1 m vaut 1 m (le pivot) |
| kilomètre | 1000 | 1 km vaut 1000 m |
| pouce | 0.0254 | 1 in vaut exactement 0,0254 m |
| pied | 0.3048 | 1 ft vaut exactement 0,3048 m |
| yard | 0.9144 | 1 yd vaut exactement 0,9144 m |
| mile | 1609.344 | 1 mi vaut exactement 1609,344 m |
| mille marin | 1852 | 1 nmi vaut exactement 1852 m |

Les cinq dernières valeurs ne sont pas des approximations. Depuis l'accord international de 1959, le pouce est **défini** comme valant exactement 25,4 millimètres. Tout le système impérial en découle : 1 ft = 12 in, 1 yd = 3 ft, 1 mi = 1760 yd.

Vérifions : 1760 × 3 × 12 × 0,0254 = 63360 × 0,0254 = 1609,344. Le compte est bon.

### 57.7.4 — Les deux opérations

Avec un facteur `f`, les deux sens s'écrivent en une ligne chacun.

```dart
double versPivot(double valeur) => valeur * f;   // 5 km -> 5000 m
double depuisPivot(double valeur) => valeur / f; // 5000 m -> 3.1069 mi
```

Et la conversion complète est la composition des deux :

```text
resultat = cible.depuisPivot( source.versPivot( valeur ) )
```

Une seule ligne de code, valable pour toutes les paires d'unités de toutes les catégories. C'est exactement ce que nous allons écrire à l'étape 5.

**État exécutable.** Rien de nouveau à compiler : cette étape est conceptuelle. Elle est pourtant la plus importante du chapitre.

---

## 57.8 — Étape 4 : le cas de la température

### 57.8.1 — La démonstration que le facteur ne suffit pas

Essayons naïvement d'appliquer le modèle du facteur à la température, avec le degré Celsius comme pivot.

Question : quel est le facteur du degré Fahrenheit ?

Un facteur `f` vérifie, par définition, `celsius = fahrenheit × f`. Testons deux points connus.

```text
Point 1 : l'eau gele        32 °F  =  0 °C
          0 = 32 x f   ->   f = 0

Point 2 : l'eau bout       212 °F  =  100 °C
          100 = 212 x f  ->  f = 0.4717...
```

Deux points, deux facteurs différents. Il n'existe donc **aucun** nombre `f` qui convienne. La démonstration est terminée : le modèle du facteur est faux pour la température.

### 57.8.2 — Pourquoi, physiquement

Un facteur, mathématiquement, c'est une fonction **linéaire** : `y = f × x`. Une telle fonction passe forcément par l'origine, c'est-à-dire que `0` se convertit en `0`.

C'est vrai pour les longueurs : 0 kilomètre, c'est 0 mile. Zéro longueur, c'est zéro longueur, quelle que soit l'unité. Le zéro est **absolu**.

Ce n'est pas vrai pour les températures usuelles : 0 °C, ce n'est pas 0 °F. Le zéro du Celsius (le gel de l'eau) et le zéro du Fahrenheit (une saumure) sont deux points **arbitraires et différents**. Le zéro est **conventionnel**.

La bonne famille de fonctions est donc **affine** : `y = a × x + b`.

```text
LINEAIRE                          AFFINE
y = a.x                           y = a.x + b

  y ^                               y ^
    │        /                        │        /
    │      /                          │      /
    │    /                            │    /
    │  /                              │  /  <- decale de b
────┼/──────────> x               ────/──────────> x
    │ 0 -> 0                        │b│ 0 -> b
```

### 57.8.3 — Les quatre échelles et leurs formules

Avec le degré Celsius comme pivot :

| Unité | Vers le pivot (°C) | Depuis le pivot (°C) |
| --- | --- | --- |
| Celsius | `c` | `c` |
| Kelvin | `k - 273.15` | `c + 273.15` |
| Fahrenheit | `(f - 32) × 5 / 9` | `c × 9 / 5 + 32` |
| Rankine | `(r - 491.67) × 5 / 9` | `(c + 273.15) × 9 / 5` |

Vérifications, à faire de tête :

```text
100 °F -> °C    (100 - 32) x 5/9 = 68 x 5/9 = 340/9 = 37,7778 °C
 37 °C -> °F    37 x 9/5 + 32 = 66,6 + 32 = 98,6 °F
  0 °C -> K     0 + 273,15 = 273,15 K
 20 °C -> K     20 + 273,15 = 293,15 K
-40 °C -> °F    -40 x 1,8 + 32 = -72 + 32 = -40 °F   (le point de croisement)
  0 °C -> °R    (0 + 273,15) x 1,8 = 491,67 °R
```

Le cas `-40` est célèbre : c'est l'unique température où les deux échelles affichent le même nombre. Il fera un excellent test unitaire.

### 57.8.4 — La généralisation qui sauve le modèle

Nous avons maintenant deux familles de conversions :

- longueur, masse, volume, données, devises : `y = a × x` ;
- température : `y = a × x + b`.

La tentation du débutant est d'ajouter un `if` dans la fonction de conversion :

```dart
// A NE PAS FAIRE
if (categorie == Categorie.temperature) {
  // ... code special ...
} else {
  // ... facteur ...
}
```

C'est une mauvaise idée. Ce `if` va se propager partout, et le jour où vous ajouterez les décibels (échelle logarithmique) ou les tailles de vêtements (table de correspondance), il faudra un troisième cas, puis un quatrième.

La bonne solution est de **remonter d'un cran d'abstraction**. Une unité ne stocke plus un nombre ; elle stocke **deux fonctions**.

```dart
final double Function(double) versPivot;
final double Function(double) depuisPivot;
```

Au chapitre 07, vous avez appris qu'une fonction est une valeur comme une autre en Dart : on peut la ranger dans une variable, la passer en paramètre, la stocker dans un champ. C'est exactement ce dont nous avons besoin.

Le cas du facteur devient alors un **cas particulier** : un constructeur nommé qui fabrique automatiquement les deux fonctions à partir d'un nombre.

```text
                 Unite
                   │
     ┌─────────────┴──────────────┐
     │                            │
Unite.facteur(2.54)        Unite.affine(...)  ou fonctions libres
     │                            │
versPivot  = v => v * f     versPivot  = f => (f - 32) * 5 / 9
depuisPivot= v => v / f     depuisPivot= c => c * 9 / 5 + 32
```

La fonction de conversion, elle, ne sait rien de tout cela. Elle appelle `versPivot` puis `depuisPivot`, sans jamais demander de quelle catégorie il s'agit. Aucun `if`.

**État exécutable.** Étape conceptuelle également. Le code arrive tout de suite.

---

## 57.9 — Étape 5 : la classe `Unite`

### 57.9.1 — Le code

**Fichier : `lib/modeles/unite.dart`**

```dart
import 'categorie.dart';

/// Signature d'une fonction de conversion : un nombre entre, un nombre sort.
///
/// On lui donne un nom pour que les declarations restent lisibles.
typedef Transformation = double Function(double valeur);

/// Une unite de mesure.
///
/// Une unite ne sait faire que deux choses :
///  - convertir une valeur exprimee dans cette unite VERS l'unite pivot ;
///  - convertir une valeur exprimee dans l'unite pivot VERS cette unite.
///
/// Elle ne connait aucune autre unite. C'est ce qui rend le modele extensible :
/// ajouter une unite ne modifie aucune unite existante.
class Unite {
  /// Constructeur general : on fournit les deux fonctions a la main.
  ///
  /// Reserve aux echelles qui ne sont pas de simples multiplications
  /// (temperature, et plus tard tout ce que vous voudrez).
  Unite({
    required this.identifiant,
    required this.nom,
    required this.symbole,
    required this.categorie,
    required this.versPivot,
    required this.depuisPivot,
  });

  /// Constructeur du cas courant : une unite definie par un facteur.
  ///
  /// [facteur] est la valeur d'UNE unite exprimee dans le pivot.
  /// Exemple : le pouce vaut 0.0254 metre, donc facteur = 0.0254.
  ///
  /// Note : les deux closures capturent le parametre `facteur`. C'est autorise
  /// dans une liste d'initialisation, contrairement a `this`.
  Unite.facteur({
    required this.identifiant,
    required this.nom,
    required this.symbole,
    required this.categorie,
    required double facteur,
  })  : assert(facteur > 0, 'Un facteur doit etre strictement positif.'),
        versPivot = ((double valeur) => valeur * facteur),
        depuisPivot = ((double valeur) => valeur / facteur);

  /// Constructeur des echelles affines : pivot = valeur * pente + origine.
  ///
  /// Exemple pour le Fahrenheit avec le Celsius en pivot :
  /// pente = 5/9, origine = -160/9, car (f - 32) * 5/9 = f * 5/9 - 160/9.
  /// On preferera toutefois ecrire les fonctions en clair, plus lisibles.
  Unite.affine({
    required this.identifiant,
    required this.nom,
    required this.symbole,
    required this.categorie,
    required double pente,
    required double origine,
  })  : assert(pente != 0, 'Une pente nulle rendrait la conversion irreversible.'),
        versPivot = ((double valeur) => valeur * pente + origine),
        depuisPivot = ((double valeur) => (valeur - origine) / pente);

  /// Identifiant stable, utilise pour la persistance et les tests.
  final String identifiant;

  /// Nom complet, affiche dans la liste deroulante.
  final String nom;

  /// Symbole court, affiche a cote du resultat.
  final String symbole;

  /// Categorie a laquelle appartient l'unite.
  final Categorie categorie;

  /// Convertit une valeur de CETTE unite vers l'unite pivot.
  final Transformation versPivot;

  /// Convertit une valeur de l'unite pivot vers CETTE unite.
  final Transformation depuisPivot;

  /// Libelle complet pour la liste deroulante : "Kilometre (km)".
  String get libelle => '$nom ($symbole)';

  /// Deux unites sont egales si elles ont le meme identifiant.
  ///
  /// C'est INDISPENSABLE pour `DropdownButton` : le widget compare la valeur
  /// selectionnee aux valeurs de ses items avec `==`. Sans cette redefinition,
  /// deux instances decrivant le meme metre seraient considerees differentes
  /// et le widget leverait une assertion.
  @override
  bool operator ==(Object autre) {
    if (identical(this, autre)) {
      return true;
    }
    return autre is Unite && autre.identifiant == identifiant;
  }

  @override
  int get hashCode => identifiant.hashCode;

  @override
  String toString() => 'Unite($identifiant)';
}
```

### 57.9.2 — Pourquoi trois constructeurs

| Constructeur | Quand l'utiliser | Exemple |
| --- | --- | --- |
| `Unite.facteur` | 95 % des cas : une simple multiplication | le kilomètre, le gramme, le litre |
| `Unite.affine` | Une droite décalée | le Kelvin (pente 1, origine -273.15) |
| `Unite(...)` | N'importe quoi d'autre | une échelle logarithmique, une table |

En pratique nous écrirons les températures avec le constructeur général, parce que `(f - 32) * 5 / 9` se lit mieux qu'une pente et une origine calculées à la main. Le constructeur `Unite.affine` reste disponible et servira au défi 6.

### 57.9.3 — Le piège de `==` et de `DropdownButton`

Cette redéfinition de `==` n'est pas de la coquetterie. Sans elle, le message d'erreur suivant vous attend :

```text
There should be exactly one item with [DropdownButton]'s value: Unite.
Either zero or 2 or more [DropdownMenuItem]s were detected with the same value
```

Il apparaît dès que la valeur passée à `value:` n'est pas **égale** à l'une des valeurs des `items`. Avec l'égalité par défaut de Dart (l'identité en mémoire), reconstruire la liste des unités à chaque `build` suffirait à casser l'application.

Nous verrons à l'étape 10 une seconde protection : construire la liste **une seule fois**, dans `initState`.

**État exécutable.** Le projet compile. Vérifiez avec `flutter analyze`.

---

## 57.10 — Étape 6 : le catalogue des unités

### 57.10.1 — Structure

Le catalogue est une simple `Map<Categorie, List<Unite>>`. On le déclare `final` au niveau du fichier : il est construit une fois au démarrage, puis lu partout.

L'ordre des unités dans chaque liste est l'ordre d'affichage dans les listes déroulantes. On y met les unités les plus courantes en premier.

### 57.10.2 — Le code

**Fichier : `lib/donnees/catalogue.dart`**

```dart
import '../modeles/categorie.dart';
import '../modeles/unite.dart';

/// ---------------------------------------------------------------------------
/// LONGUEUR — pivot : le metre
///
/// Les valeurs imperiales sont EXACTES depuis l'accord international de 1959 :
/// le pouce vaut exactement 25,4 mm, et tout le reste en decoule.
/// ---------------------------------------------------------------------------
final List<Unite> _longueurs = <Unite>[
  Unite.facteur(
    identifiant: 'millimetre',
    nom: 'Millimetre',
    symbole: 'mm',
    categorie: Categorie.longueur,
    facteur: 0.001,
  ),
  Unite.facteur(
    identifiant: 'centimetre',
    nom: 'Centimetre',
    symbole: 'cm',
    categorie: Categorie.longueur,
    facteur: 0.01,
  ),
  Unite.facteur(
    identifiant: 'metre',
    nom: 'Metre',
    symbole: 'm',
    categorie: Categorie.longueur,
    facteur: 1,
  ),
  Unite.facteur(
    identifiant: 'kilometre',
    nom: 'Kilometre',
    symbole: 'km',
    categorie: Categorie.longueur,
    facteur: 1000,
  ),
  Unite.facteur(
    identifiant: 'pouce',
    nom: 'Pouce',
    symbole: 'in',
    categorie: Categorie.longueur,
    facteur: 0.0254, // exact par definition
  ),
  Unite.facteur(
    identifiant: 'pied',
    nom: 'Pied',
    symbole: 'ft',
    categorie: Categorie.longueur,
    facteur: 0.3048, // 12 x 0.0254, exact
  ),
  Unite.facteur(
    identifiant: 'yard',
    nom: 'Yard',
    symbole: 'yd',
    categorie: Categorie.longueur,
    facteur: 0.9144, // 3 x 0.3048, exact
  ),
  Unite.facteur(
    identifiant: 'mile',
    nom: 'Mile',
    symbole: 'mi',
    categorie: Categorie.longueur,
    facteur: 1609.344, // 1760 x 0.9144, exact
  ),
  Unite.facteur(
    identifiant: 'mille_marin',
    nom: 'Mille marin',
    symbole: 'nmi',
    categorie: Categorie.longueur,
    facteur: 1852, // exact, defini en 1929
  ),
];

/// ---------------------------------------------------------------------------
/// MASSE — pivot : le kilogramme
///
/// La livre avoirdupois vaut exactement 0,453 592 37 kg depuis 1959.
/// L'once est la livre divisee par 16, le stone la livre multipliee par 14.
/// ---------------------------------------------------------------------------
final List<Unite> _masses = <Unite>[
  Unite.facteur(
    identifiant: 'milligramme',
    nom: 'Milligramme',
    symbole: 'mg',
    categorie: Categorie.masse,
    facteur: 0.000001,
  ),
  Unite.facteur(
    identifiant: 'gramme',
    nom: 'Gramme',
    symbole: 'g',
    categorie: Categorie.masse,
    facteur: 0.001,
  ),
  Unite.facteur(
    identifiant: 'kilogramme',
    nom: 'Kilogramme',
    symbole: 'kg',
    categorie: Categorie.masse,
    facteur: 1,
  ),
  Unite.facteur(
    identifiant: 'tonne',
    nom: 'Tonne',
    symbole: 't',
    categorie: Categorie.masse,
    facteur: 1000,
  ),
  Unite.facteur(
    identifiant: 'once',
    nom: 'Once',
    symbole: 'oz',
    categorie: Categorie.masse,
    facteur: 0.028349523125, // 0.45359237 / 16, exact
  ),
  Unite.facteur(
    identifiant: 'livre',
    nom: 'Livre',
    symbole: 'lb',
    categorie: Categorie.masse,
    facteur: 0.45359237, // exact par definition
  ),
  Unite.facteur(
    identifiant: 'stone',
    nom: 'Stone',
    symbole: 'st',
    categorie: Categorie.masse,
    facteur: 6.35029318, // 14 x 0.45359237, exact
  ),
  Unite.facteur(
    identifiant: 'carat',
    nom: 'Carat metrique',
    symbole: 'ct',
    categorie: Categorie.masse,
    facteur: 0.0002, // 200 mg, exact
  ),
];

/// ---------------------------------------------------------------------------
/// TEMPERATURE — pivot : le degre Celsius
///
/// SEULE categorie ou l'on n'utilise PAS `Unite.facteur`.
/// Chaque unite fournit ses deux fonctions en clair.
/// ---------------------------------------------------------------------------
final List<Unite> _temperatures = <Unite>[
  Unite(
    identifiant: 'celsius',
    nom: 'Degre Celsius',
    symbole: '°C',
    categorie: Categorie.temperature,
    versPivot: (double c) => c,
    depuisPivot: (double c) => c,
  ),
  Unite(
    identifiant: 'fahrenheit',
    nom: 'Degre Fahrenheit',
    symbole: '°F',
    categorie: Categorie.temperature,
    versPivot: (double f) => (f - 32) * 5 / 9,
    depuisPivot: (double c) => c * 9 / 5 + 32,
  ),
  Unite(
    identifiant: 'kelvin',
    nom: 'Kelvin',
    symbole: 'K',
    categorie: Categorie.temperature,
    versPivot: (double k) => k - 273.15,
    depuisPivot: (double c) => c + 273.15,
  ),
  Unite(
    identifiant: 'rankine',
    nom: 'Degre Rankine',
    symbole: '°R',
    categorie: Categorie.temperature,
    // 0 °C vaut 491,67 °R ; un degre Rankine vaut 5/9 de degre Celsius.
    versPivot: (double r) => (r - 491.67) * 5 / 9,
    depuisPivot: (double c) => (c + 273.15) * 9 / 5,
  ),
  Unite(
    identifiant: 'reaumur',
    nom: 'Degre Reaumur',
    symbole: '°Re',
    categorie: Categorie.temperature,
    // Echelle historique : 0 °Re au gel, 80 °Re a l'ebullition de l'eau.
    versPivot: (double re) => re * 5 / 4,
    depuisPivot: (double c) => c * 4 / 5,
  ),
  Unite(
    identifiant: 'delisle',
    nom: 'Degre Delisle',
    symbole: '°De',
    categorie: Categorie.temperature,
    // Echelle inversee : plus il fait chaud, plus le nombre est petit.
    versPivot: (double de) => 100 - de * 2 / 3,
    depuisPivot: (double c) => (100 - c) * 3 / 2,
  ),
];

/// ---------------------------------------------------------------------------
/// VOLUME — pivot : le litre
///
/// Les mesures de cuisine sont les mesures AMERICAINES (US customary),
/// derivees du gallon US qui vaut exactement 3,785 411 784 L
/// (231 pouces cubes). Les mesures britanniques sont differentes :
/// c'est le defi 4.
/// ---------------------------------------------------------------------------
final List<Unite> _volumes = <Unite>[
  Unite.facteur(
    identifiant: 'millilitre',
    nom: 'Millilitre',
    symbole: 'mL',
    categorie: Categorie.volume,
    facteur: 0.001,
  ),
  Unite.facteur(
    identifiant: 'centilitre',
    nom: 'Centilitre',
    symbole: 'cL',
    categorie: Categorie.volume,
    facteur: 0.01,
  ),
  Unite.facteur(
    identifiant: 'litre',
    nom: 'Litre',
    symbole: 'L',
    categorie: Categorie.volume,
    facteur: 1,
  ),
  Unite.facteur(
    identifiant: 'metre_cube',
    nom: 'Metre cube',
    symbole: 'm3',
    categorie: Categorie.volume,
    facteur: 1000,
  ),
  Unite.facteur(
    identifiant: 'cuillere_cafe',
    nom: 'Cuillere a cafe (US)',
    symbole: 'tsp',
    categorie: Categorie.volume,
    facteur: 0.00492892159375, // gallon / 768, exact
  ),
  Unite.facteur(
    identifiant: 'cuillere_soupe',
    nom: 'Cuillere a soupe (US)',
    symbole: 'tbsp',
    categorie: Categorie.volume,
    facteur: 0.01478676478125, // gallon / 256, exact
  ),
  Unite.facteur(
    identifiant: 'tasse',
    nom: 'Tasse (US)',
    symbole: 'cup',
    categorie: Categorie.volume,
    facteur: 0.2365882365, // gallon / 16, exact
  ),
  Unite.facteur(
    identifiant: 'pinte_us',
    nom: 'Pinte (US)',
    symbole: 'pt',
    categorie: Categorie.volume,
    facteur: 0.473176473, // gallon / 8, exact
  ),
  Unite.facteur(
    identifiant: 'gallon_us',
    nom: 'Gallon (US)',
    symbole: 'gal',
    categorie: Categorie.volume,
    facteur: 3.785411784, // 231 pouces cubes, exact
  ),
];

/// ---------------------------------------------------------------------------
/// DONNEES — pivot : l'octet
///
/// ATTENTION, deux systemes coexistent et ne doivent pas etre melanges :
///   - decimal (SI)     : 1 ko = 1000 o     -> ko, Mo, Go, To
///   - binaire (IEC)    : 1 Kio = 1024 o    -> Kio, Mio, Gio, Tio
/// Le disque dur vendu "1 To" contient 1 000 000 000 000 octets, que le
/// systeme d'exploitation affiche comme 931 Gio. Rien n'a disparu :
/// ce sont deux unites differentes.
/// ---------------------------------------------------------------------------
final List<Unite> _donnees = <Unite>[
  Unite.facteur(
    identifiant: 'bit',
    nom: 'Bit',
    symbole: 'b',
    categorie: Categorie.donnees,
    facteur: 0.125, // 1 octet = 8 bits
  ),
  Unite.facteur(
    identifiant: 'octet',
    nom: 'Octet',
    symbole: 'o',
    categorie: Categorie.donnees,
    facteur: 1,
  ),
  Unite.facteur(
    identifiant: 'kilooctet',
    nom: 'Kilooctet (1000 o)',
    symbole: 'ko',
    categorie: Categorie.donnees,
    facteur: 1000,
  ),
  Unite.facteur(
    identifiant: 'megaoctet',
    nom: 'Megaoctet (10^6 o)',
    symbole: 'Mo',
    categorie: Categorie.donnees,
    facteur: 1000000,
  ),
  Unite.facteur(
    identifiant: 'gigaoctet',
    nom: 'Gigaoctet (10^9 o)',
    symbole: 'Go',
    categorie: Categorie.donnees,
    facteur: 1000000000,
  ),
  Unite.facteur(
    identifiant: 'teraoctet',
    nom: 'Teraoctet (10^12 o)',
    symbole: 'To',
    categorie: Categorie.donnees,
    facteur: 1000000000000,
  ),
  Unite.facteur(
    identifiant: 'kibioctet',
    nom: 'Kibioctet (1024 o)',
    symbole: 'Kio',
    categorie: Categorie.donnees,
    facteur: 1024,
  ),
  Unite.facteur(
    identifiant: 'mebioctet',
    nom: 'Mebioctet (1024^2 o)',
    symbole: 'Mio',
    categorie: Categorie.donnees,
    facteur: 1048576,
  ),
  Unite.facteur(
    identifiant: 'gibioctet',
    nom: 'Gibioctet (1024^3 o)',
    symbole: 'Gio',
    categorie: Categorie.donnees,
    facteur: 1073741824,
  ),
  Unite.facteur(
    identifiant: 'tebioctet',
    nom: 'Tebioctet (1024^4 o)',
    symbole: 'Tio',
    categorie: Categorie.donnees,
    facteur: 1099511627776,
  ),
];

/// ---------------------------------------------------------------------------
/// DEVISES — pivot : l'euro
///
/// Ce sont les TAUX DE CONVERSION IRREVOCABLES fixes par le Conseil de l'Union
/// europeenne lors du passage a l'euro. Ils ne changeront jamais : ce ne sont
/// pas des cours de marche, mais des definitions legales.
///
/// Le facteur d'une devise est donc 1 / taux, puisque le taux est exprime
/// en "unites de l'ancienne monnaie pour 1 euro".
/// ---------------------------------------------------------------------------
final List<Unite> _devises = <Unite>[
  Unite.facteur(
    identifiant: 'eur',
    nom: 'Euro',
    symbole: 'EUR',
    categorie: Categorie.devise,
    facteur: 1,
  ),
  Unite.facteur(
    identifiant: 'frf',
    nom: 'Franc francais',
    symbole: 'FRF',
    categorie: Categorie.devise,
    facteur: 1 / 6.55957,
  ),
  Unite.facteur(
    identifiant: 'dem',
    nom: 'Mark allemand',
    symbole: 'DEM',
    categorie: Categorie.devise,
    facteur: 1 / 1.95583,
  ),
  Unite.facteur(
    identifiant: 'itl',
    nom: 'Lire italienne',
    symbole: 'ITL',
    categorie: Categorie.devise,
    facteur: 1 / 1936.27,
  ),
  Unite.facteur(
    identifiant: 'esp',
    nom: 'Peseta espagnole',
    symbole: 'ESP',
    categorie: Categorie.devise,
    facteur: 1 / 166.386,
  ),
  Unite.facteur(
    identifiant: 'bef',
    nom: 'Franc belge',
    symbole: 'BEF',
    categorie: Categorie.devise,
    facteur: 1 / 40.3399,
  ),
  Unite.facteur(
    identifiant: 'nlg',
    nom: 'Florin neerlandais',
    symbole: 'NLG',
    categorie: Categorie.devise,
    facteur: 1 / 2.20371,
  ),
  Unite.facteur(
    identifiant: 'ats',
    nom: 'Schilling autrichien',
    symbole: 'ATS',
    categorie: Categorie.devise,
    facteur: 1 / 13.7603,
  ),
  Unite.facteur(
    identifiant: 'pte',
    nom: 'Escudo portugais',
    symbole: 'PTE',
    categorie: Categorie.devise,
    facteur: 1 / 200.482,
  ),
  Unite.facteur(
    identifiant: 'fim',
    nom: 'Mark finlandais',
    symbole: 'FIM',
    categorie: Categorie.devise,
    facteur: 1 / 5.94573,
  ),
  Unite.facteur(
    identifiant: 'iep',
    nom: 'Livre irlandaise',
    symbole: 'IEP',
    categorie: Categorie.devise,
    facteur: 1 / 0.787564,
  ),
  Unite.facteur(
    identifiant: 'grd',
    nom: 'Drachme grecque',
    symbole: 'GRD',
    categorie: Categorie.devise,
    facteur: 1 / 340.750,
  ),
];

/// Le catalogue complet : a chaque categorie, sa liste d'unites.
///
/// L'ordre des listes est l'ordre d'affichage dans les menus deroulants.
final Map<Categorie, List<Unite>> catalogue = <Categorie, List<Unite>>{
  Categorie.longueur: _longueurs,
  Categorie.masse: _masses,
  Categorie.temperature: _temperatures,
  Categorie.volume: _volumes,
  Categorie.donnees: _donnees,
  Categorie.devise: _devises,
};

/// Toutes les unites, toutes categories confondues.
///
/// Utilise `expand` (chapitre 14) : chaque valeur de la Map est une liste,
/// et `expand` les met bout a bout en une seule sequence.
final List<Unite> toutesLesUnites =
    catalogue.values.expand((List<Unite> liste) => liste).toList(growable: false);

/// Renvoie les unites d'une categorie donnee.
///
/// Ne renvoie jamais `null` : une categorie sans unite renverrait une liste
/// vide, ce qui simplifie tous les appelants (chapitre 12).
List<Unite> unitesDe(Categorie categorie) {
  return catalogue[categorie] ?? const <Unite>[];
}

/// Retrouve une unite par son identifiant, ou `null` si elle n'existe pas.
///
/// Sert au rechargement des preferences : si l'identifiant enregistre
/// correspond a une unite supprimee depuis, on retombera sur un defaut.
Unite? uniteParIdentifiant(String? identifiant) {
  if (identifiant == null) {
    return null;
  }
  for (final Unite unite in toutesLesUnites) {
    if (unite.identifiant == identifiant) {
      return unite;
    }
  }
  return null;
}

/// Unite par defaut d'une categorie, cote source.
Unite uniteSourceParDefaut(Categorie categorie) => unitesDe(categorie).first;

/// Unite par defaut d'une categorie, cote cible.
///
/// On prend la deuxieme de la liste pour ne pas afficher au demarrage
/// une conversion d'une unite vers elle-meme, qui n'apprend rien.
Unite uniteCibleParDefaut(Categorie categorie) {
  final List<Unite> unites = unitesDe(categorie);
  return unites.length > 1 ? unites[1] : unites.first;
}
```

### 57.10.3 — Vérifier les données, pas seulement le code

Un catalogue est un tableau de constantes. Le compilateur ne peut rien vérifier : si vous tapez `0.2540` au lieu de `0.0254`, tout compile et tout est faux.

Trois défenses, dans l'ordre :

1. **Écrire la source du chiffre en commentaire** (« exact par definition », « gallon / 16 »). Cela oblige à savoir d'où il vient.
2. **Exprimer les valeurs dérivées par un calcul** quand c'est possible : `1 / 6.55957` plutôt que `0.152449017`. Le lecteur reconnaît le taux officiel, et la division est faite par la machine, sans erreur de recopie.
3. **Tester** au moins une conversion connue par catégorie. Ce sera l'étape 8.

**État exécutable.** Le projet compile. Le catalogue est prêt mais pas encore utilisé.

---

## 57.11 — Étape 7 : la fonction de conversion pure

### 57.11.1 — Ce que « pure » signifie

Une fonction est **pure** quand :

- son résultat ne dépend **que** de ses paramètres ;
- elle ne modifie **rien** en dehors d'elle-même : pas de `setState`, pas d'écriture disque, pas de log, pas de variable globale.

Deux conséquences pratiques, énormes :

```text
FONCTION PURE                       FONCTION IMPURE
convertir(5, km, mi)                _mettreAJourResultat()
   -> toujours 3.10685...              -> depend de _controleur.text
   -> testable en 1 ligne              -> depend de _uniteSource
   -> aucun contexte necessaire        -> modifie _resultat
                                       -> il faut un widget pour la tester
```

Toute la logique difficile de ce projet tient dans une fonction pure de dix lignes. Le reste de l'application est du branchement.

### 57.11.2 — Le code

**Fichier : `lib/logique/convertisseur.dart`**

```dart
import '../modeles/unite.dart';

/// Erreur levee quand on tente de convertir entre deux categories.
///
/// On definit une exception dediee plutot que d'utiliser `Exception` :
/// l'appelant peut ainsi la rattraper precisement (chapitre 13).
class CategoriesIncompatibles implements Exception {
  const CategoriesIncompatibles(this.depuis, this.vers);

  final Unite depuis;
  final Unite vers;

  @override
  String toString() {
    return 'CategoriesIncompatibles : impossible de convertir '
        '${depuis.symbole} (${depuis.categorie.libelle}) en '
        '${vers.symbole} (${vers.categorie.libelle}).';
  }
}

/// Convertit [valeur], exprimee dans l'unite [depuis], vers l'unite [vers].
///
/// C'est LA fonction du projet. Elle est pure : meme entree, meme sortie,
/// aucun effet de bord, aucune dependance a Flutter.
///
/// Le principe tient en une ligne : on remonte au pivot, puis on redescend.
///
/// Leve [CategoriesIncompatibles] si les deux unites n'appartiennent pas a
/// la meme categorie. Ce cas ne devrait jamais arriver depuis l'interface,
/// qui ne propose que des unites de la categorie active, mais une fonction
/// publique doit se defendre elle-meme.
double convertir({
  required double valeur,
  required Unite depuis,
  required Unite vers,
}) {
  if (depuis.categorie != vers.categorie) {
    throw CategoriesIncompatibles(depuis, vers);
  }

  // Raccourci : convertir une unite vers elle-meme ne doit rien changer.
  // Sans ce test, 1 / 3 * 3 pourrait rendre 0.9999999999999999.
  if (depuis == vers) {
    return valeur;
  }

  final double pivot = depuis.versPivot(valeur);
  return vers.depuisPivot(pivot);
}

/// Combien vaut UNE unite [depuis] exprimee en [vers].
///
/// Sert a afficher la ligne "1 km = 0,621371 mi" sous le resultat.
///
/// Attention : pour la temperature, ce nombre n'a pas le sens qu'on croit.
/// "1 °C = 33,8 °F" est vrai en tant que temperature, mais un ECART de 1 °C
/// vaut 1,8 °F, pas 33,8. Voir la section des erreurs frequentes.
double tauxUnitaire({required Unite depuis, required Unite vers}) {
  return convertir(valeur: 1, depuis: depuis, vers: vers);
}
```

### 57.11.3 — Le raccourci `depuis == vers`

Ce petit `if` n'est pas une optimisation de vitesse. C'est une **garantie d'exactitude**.

Sans lui, convertir 1 tiers de quelque chose vers la même unité passerait par une multiplication puis une division en virgule flottante, et pourrait rendre un nombre très légèrement différent. Avec lui, l'identité est exacte par construction.

C'est un réflexe à prendre : dans toute fonction de transformation, traitez le cas neutre à part.

### 57.11.4 — Premier essai en console

Avant même d'avoir une interface, on peut vérifier la logique. Créez temporairement ce fichier et lancez-le avec `dart run outil/essai.dart`.

**Fichier temporaire : `outil/essai.dart`**

```dart
import '../lib/donnees/catalogue.dart';
import '../lib/logique/convertisseur.dart';
import '../lib/modeles/unite.dart';

void main() {
  final Unite km = uniteParIdentifiant('kilometre')!;
  final Unite mi = uniteParIdentifiant('mile')!;
  final Unite celsius = uniteParIdentifiant('celsius')!;
  final Unite fahrenheit = uniteParIdentifiant('fahrenheit')!;

  print(convertir(valeur: 5, depuis: km, vers: mi));
  print(convertir(valeur: 100, depuis: fahrenheit, vers: celsius));
  print(convertir(valeur: -40, depuis: celsius, vers: fahrenheit));
}
```

**Résultat :**

```text
3.1068559611866697
37.77777777777778
-40.0
```

Les trois valeurs sont justes. Le cœur du projet fonctionne, et il n'y a pas encore une seule ligne d'interface. Supprimez ensuite le dossier `outil/` : l'étape 8 le remplace par de vrais tests.

**État exécutable.** `dart run outil/essai.dart` affiche les trois lignes ci-dessus.

---

## 57.12 — Étape 8 : les premiers tests unitaires

### 57.12.1 — Pourquoi maintenant et pas à la fin

Les tests ne sont pas une corvée de fin de projet. Ils sont l'**outil** qui vous permet de modifier le catalogue sans peur pendant tout le reste du chapitre.

Nous les écrivons donc dès que la fonction pure existe, avant l'interface.

### 57.12.2 — Le piège de l'égalité des `double`

N'écrivez jamais ceci :

```dart
// FAUX
expect(convertir(valeur: 100, depuis: fahrenheit, vers: celsius), 37.7778);
```

Le résultat vaut `37.77777777777778`, pas `37.7778`. Le test échouera.

Et même sans arrondi, l'égalité stricte entre `double` est fragile : `0.1 + 0.2` ne vaut pas `0.3` en virgule flottante. On compare donc **à une tolérance près**, avec le matcher `closeTo`.

```dart
expect(resultat, closeTo(37.777778, 0.000001));
```

Lecture : « le résultat est à moins d'un millionième de 37,777778 ».

### 57.12.3 — Le code

**Fichier : `test/convertisseur_test.dart`**

```dart
import 'package:convertisseur/donnees/catalogue.dart';
import 'package:convertisseur/logique/convertisseur.dart';
import 'package:convertisseur/modeles/categorie.dart';
import 'package:convertisseur/modeles/unite.dart';
import 'package:flutter_test/flutter_test.dart';

/// Raccourci de lecture : `u('mile')` au lieu de la version longue.
///
/// Le `!` est acceptable ICI parce qu'un identifiant inconnu doit faire
/// echouer le test bruyamment.
Unite u(String identifiant) => uniteParIdentifiant(identifiant)!;

void main() {
  group('Longueur', () {
    test('5 km valent 3,1068559... miles', () {
      final double resultat =
          convertir(valeur: 5, depuis: u('kilometre'), vers: u('mile'));
      expect(resultat, closeTo(3.1068559612, 1e-9));
    });

    test('1 pouce vaut exactement 2,54 cm', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('pouce'), vers: u('centimetre'));
      expect(resultat, closeTo(2.54, 1e-12));
    });

    test('1 metre vaut 39,37007874... pouces', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('metre'), vers: u('pouce'));
      expect(resultat, closeTo(39.3700787402, 1e-9));
    });

    test('1 mille marin vaut 1,852 km', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('mille_marin'), vers: u('kilometre'));
      expect(resultat, closeTo(1.852, 1e-12));
    });

    test('un marathon fait 26,2187574... miles', () {
      final double resultat =
          convertir(valeur: 42.195, depuis: u('kilometre'), vers: u('mile'));
      expect(resultat, closeTo(26.2187574565, 1e-9));
    });
  });

  group('Masse', () {
    test('1 livre vaut exactement 0,45359237 kg', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('livre'), vers: u('kilogramme'));
      expect(resultat, closeTo(0.45359237, 1e-12));
    });

    test('70 kg valent 154,3235835... livres', () {
      final double resultat =
          convertir(valeur: 70, depuis: u('kilogramme'), vers: u('livre'));
      expect(resultat, closeTo(154.3235835294, 1e-9));
    });

    test('1 once vaut 28,349523125 g', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('once'), vers: u('gramme'));
      expect(resultat, closeTo(28.349523125, 1e-9));
    });

    test('1 stone vaut 14 livres', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('stone'), vers: u('livre'));
      expect(resultat, closeTo(14, 1e-9));
    });
  });

  group('Temperature', () {
    test('100 °F valent 37,7777... °C', () {
      final double resultat =
          convertir(valeur: 100, depuis: u('fahrenheit'), vers: u('celsius'));
      expect(resultat, closeTo(37.7777777778, 1e-9));
    });

    test('37 °C valent 98,6 °F', () {
      final double resultat =
          convertir(valeur: 37, depuis: u('celsius'), vers: u('fahrenheit'));
      expect(resultat, closeTo(98.6, 1e-9));
    });

    test('-40 est le point de croisement Celsius / Fahrenheit', () {
      final double resultat =
          convertir(valeur: -40, depuis: u('celsius'), vers: u('fahrenheit'));
      expect(resultat, closeTo(-40, 1e-9));
    });

    test('0 °C valent 273,15 K', () {
      final double resultat =
          convertir(valeur: 0, depuis: u('celsius'), vers: u('kelvin'));
      expect(resultat, closeTo(273.15, 1e-9));
    });

    test('le zero absolu vaut -273,15 °C', () {
      final double resultat =
          convertir(valeur: 0, depuis: u('kelvin'), vers: u('celsius'));
      expect(resultat, closeTo(-273.15, 1e-9));
    });

    test('0 °C valent 491,67 °R', () {
      final double resultat =
          convertir(valeur: 0, depuis: u('celsius'), vers: u('rankine'));
      expect(resultat, closeTo(491.67, 1e-9));
    });

    test('100 °C valent 80 °Reaumur', () {
      final double resultat =
          convertir(valeur: 100, depuis: u('celsius'), vers: u('reaumur'));
      expect(resultat, closeTo(80, 1e-9));
    });

    test('l ebullition vaut 0 °Delisle', () {
      final double resultat =
          convertir(valeur: 100, depuis: u('celsius'), vers: u('delisle'));
      expect(resultat, closeTo(0, 1e-9));
    });
  });

  group('Volume', () {
    test('1 gallon US vaut 3,785411784 L', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('gallon_us'), vers: u('litre'));
      expect(resultat, closeTo(3.785411784, 1e-12));
    });

    test('1 tasse US vaut 236,5882365 mL', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('tasse'), vers: u('millilitre'));
      expect(resultat, closeTo(236.5882365, 1e-9));
    });

    test('1 cuillere a soupe vaut 3 cuilleres a cafe', () {
      final double resultat = convertir(
        valeur: 1,
        depuis: u('cuillere_soupe'),
        vers: u('cuillere_cafe'),
      );
      expect(resultat, closeTo(3, 1e-9));
    });

    test('1 metre cube vaut 1000 L', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('metre_cube'), vers: u('litre'));
      expect(resultat, closeTo(1000, 1e-9));
    });
  });

  group('Donnees', () {
    test('1 octet vaut 8 bits', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('octet'), vers: u('bit'));
      expect(resultat, closeTo(8, 1e-12));
    });

    test('1 Kio vaut 1024 octets', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('kibioctet'), vers: u('octet'));
      expect(resultat, closeTo(1024, 1e-9));
    });

    test('1 To vaut 931,3225746... Gio', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('teraoctet'), vers: u('gibioctet'));
      expect(resultat, closeTo(931.3225746155, 1e-9));
    });

    test('1 Gio vaut 1,073741824 Go', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('gibioctet'), vers: u('gigaoctet'));
      expect(resultat, closeTo(1.073741824, 1e-12));
    });
  });

  group('Devises', () {
    test('1 euro vaut 6,55957 francs', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('eur'), vers: u('frf'));
      expect(resultat, closeTo(6.55957, 1e-9));
    });

    test('100 francs valent 15,2449017... euros', () {
      final double resultat =
          convertir(valeur: 100, depuis: u('frf'), vers: u('eur'));
      expect(resultat, closeTo(15.2449017237, 1e-9));
    });

    test('1 mark allemand vaut 0,5112918... euro', () {
      final double resultat =
          convertir(valeur: 1, depuis: u('dem'), vers: u('eur'));
      expect(resultat, closeTo(0.5112918812, 1e-9));
    });

    test('1000 lires valent 0,5164568... euro', () {
      final double resultat =
          convertir(valeur: 1000, depuis: u('itl'), vers: u('eur'));
      expect(resultat, closeTo(0.5164568991, 1e-9));
    });
  });

  group('Proprietes generales', () {
    test('convertir vers la meme unite ne change rien', () {
      for (final Unite unite in toutesLesUnites) {
        expect(
          convertir(valeur: 12.34, depuis: unite, vers: unite),
          equals(12.34),
          reason: 'echec sur ${unite.identifiant}',
        );
      }
    });

    test('aller-retour : convertir puis reconvertir redonne la valeur', () {
      for (final Categorie categorie in Categorie.values) {
        final List<Unite> unites = unitesDe(categorie);
        for (final Unite a in unites) {
          for (final Unite b in unites) {
            final double aller = convertir(valeur: 42, depuis: a, vers: b);
            final double retour = convertir(valeur: aller, depuis: b, vers: a);
            expect(
              retour,
              closeTo(42, 1e-6),
              reason: 'echec sur ${a.identifiant} -> ${b.identifiant}',
            );
          }
        }
      }
    });

    test('convertir entre categories differentes leve une exception', () {
      expect(
        () => convertir(valeur: 1, depuis: u('metre'), vers: u('litre')),
        throwsA(isA<CategoriesIncompatibles>()),
      );
    });

    test('tous les identifiants sont uniques', () {
      final Set<String> vus = <String>{};
      for (final Unite unite in toutesLesUnites) {
        expect(
          vus.add(unite.identifiant),
          isTrue,
          reason: 'identifiant duplique : ${unite.identifiant}',
        );
      }
    });

    test('chaque categorie a au moins six unites', () {
      for (final Categorie categorie in Categorie.values) {
        expect(
          unitesDe(categorie).length,
          greaterThanOrEqualTo(6),
          reason: 'categorie trop pauvre : ${categorie.identifiant}',
        );
      }
    });
  });
}
```

### 57.12.4 — Les deux tests qui valent tous les autres

Regardez les deux tests de la section « Propriétés générales » nommés *aller-retour* et *identifiants uniques*.

Le premier ne teste pas une valeur : il teste une **propriété** qui doit être vraie pour toutes les paires d'unités de toutes les catégories. Il exécute à lui seul plus de trois cents conversions. Si vous ajoutez demain une unité avec un facteur inversé par erreur, ce test tombera.

Le second protège la persistance : deux unités partageant le même identifiant casseraient à la fois `uniteParIdentifiant` et le `DropdownButton`.

Un test de propriété vaut cinquante tests d'exemples. Cherchez-en systématiquement.

**État exécutable.** `flutter test` affiche `All tests passed!`.

```text
00:02 +37: All tests passed!
```

---

## 57.13 — Étape 9 : le champ de saisie et son contrôleur

### 57.13.1 — Pourquoi un contrôleur

Au chapitre 49, vous avez vu qu'un `TextField` ne stocke pas son texte dans votre `State`. Il le garde en interne. Pour lire ce texte, deux voies :

| Voie | Quand |
| --- | --- |
| `onChanged: (String texte) { ... }` | On veut réagir à chaque frappe |
| `TextEditingController` | On veut **aussi** lire ou **écrire** le texte de l'extérieur |

Ici, nous avons besoin des deux capacités :

- réagir à chaque frappe, pour la conversion en temps réel (exigence O6) ;
- **écrire** dans le champ depuis le code, pour le bouton d'effacement et pour le bouton d'inversion.

Le contrôleur est donc obligatoire.

### 57.13.2 — La règle absolue du contrôleur

Un `TextEditingController` est un objet vivant, doté d'écouteurs. Si vous ne le détruisez pas, il reste en mémoire avec le widget qui l'écoutait : c'est une **fuite mémoire**.

```text
initState()   ->  je cree le controleur
build()       ->  je le branche sur le TextField
dispose()     ->  je le detruis   <-- NE JAMAIS OUBLIER
```

Le mode debug de Flutter finit par vous le signaler, mais bien plus tard, dans un message obscur. Prenez l'habitude d'écrire `dispose()` **immédiatement après** avoir créé le contrôleur, avant même d'écrire `build`.

### 57.13.3 — Première version de la page

**Fichier : `lib/ecrans/page_categorie.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/categorie.dart';

/// Un convertisseur pour UNE categorie.
///
/// Version 1 : uniquement le champ de saisie. Le resultat affiche pour
/// l'instant le texte brut, ce qui permet de verifier le branchement.
class PageCategorie extends StatefulWidget {
  const PageCategorie({super.key, required this.categorie});

  final Categorie categorie;

  @override
  State<PageCategorie> createState() => _PageCategorieState();
}

class _PageCategorieState extends State<PageCategorie> {
  /// Le controleur du champ de saisie.
  ///
  /// `late final` : cree une seule fois dans initState, jamais remplace.
  late final TextEditingController _controleur;

  /// Le texte actuellement saisi, recopie a chaque frappe.
  String _saisie = '';

  @override
  void initState() {
    super.initState();
    // On demarre a "1" : l'utilisateur voit tout de suite une conversion
    // significative au lieu d'un ecran vide.
    _controleur = TextEditingController(text: '1');
    _saisie = _controleur.text;
  }

  @override
  void dispose() {
    // Obligatoire. Sans cette ligne, le controleur fuit.
    _controleur.dispose();
    super.dispose();
  }

  /// Appele a CHAQUE frappe de l'utilisateur.
  void _surSaisie(String texte) {
    setState(() {
      _saisie = texte;
    });
  }

  /// Vide le champ et remet l'affichage a zero.
  void _effacer() {
    _controleur.clear();
    _surSaisie('');
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return ListView(
      // ListView plutot que Column : quand le clavier virtuel s'ouvre, il
      // mange la moitie de l'ecran. Sans zone defilante, on obtient
      // l'erreur "RenderFlex overflowed" du chapitre 46.
      padding: const EdgeInsets.all(16),
      children: <Widget>[
        Text('Valeur a convertir', style: theme.textTheme.labelLarge),
        const SizedBox(height: 8),
        TextField(
          controller: _controleur,
          onChanged: _surSaisie,
          autofocus: true,
          decoration: InputDecoration(
            border: const OutlineInputBorder(),
            hintText: 'Saisissez un nombre',
            suffixIcon: IconButton(
              icon: const Icon(Icons.backspace_outlined),
              tooltip: 'Effacer',
              onPressed: _effacer,
            ),
          ),
        ),
        const SizedBox(height: 24),
        Card(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Categorie : ${widget.categorie.libelle}\n'
              'Texte saisi : "$_saisie"',
              style: theme.textTheme.bodyLarge,
            ),
          ),
        ),
      ],
    );
  }
}
```

### 57.13.4 — Le brancher dans `main.dart`

**Fichier : `lib/main.dart`**

```dart
import 'package:flutter/material.dart';

import 'ecrans/page_categorie.dart';
import 'modeles/categorie.dart';

void main() {
  runApp(const ApplicationConvertisseur());
}

class ApplicationConvertisseur extends StatelessWidget {
  const ApplicationConvertisseur({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Convertisseur',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
      ),
      home: Scaffold(
        appBar: AppBar(title: const Text('Convertisseur')),
        // Une seule categorie pour l'instant ; les onglets viendront
        // a l'etape 16.
        body: const PageCategorie(categorie: Categorie.longueur),
      ),
    );
  }
}
```

**État exécutable.** Lancez l'application. Tapez `12ab`. La carte affiche :

```text
Categorie : Longueur
Texte saisi : "12ab"
```

Le champ accepte encore n'importe quoi. C'est le sujet de l'étape suivante.

---

## 57.14 — Étape 10 : n'accepter que des nombres

### 57.14.1 — Trois lignes de défense, pas une

Un débutant croit qu'il suffit de « mettre le clavier numérique ». C'est faux, et cette erreur produit des applications qui plantent. Il faut **trois** protections empilées, et chacune couvre ce que la précédente laisse passer.

```text
DEFENSE 1 : keyboardType
   Affiche le bon clavier sur mobile.
   -> CONFORT, pas securite.
   -> Contournable : clavier physique, copier-coller, clavier tiers.

DEFENSE 2 : inputFormatters
   Filtre les CARACTERES au moment ou ils entrent dans le champ.
   -> Empeche de taper une lettre.
   -> Ne garantit PAS que le texte est un nombre : "-" ou ",," passent.

DEFENSE 3 : double.tryParse
   Verifie que la chaine COMPLETE est un nombre valide.
   -> Seule verite. Ne leve jamais d'exception : renvoie null.
```

Retenez la distinction, elle est capitale :

> `inputFormatters` filtre des **caractères**. `tryParse` valide une **chaîne entière**. Les deux sont nécessaires, aucun ne remplace l'autre.

### 57.14.2 — Le clavier

```dart
keyboardType: const TextInputType.numberWithOptions(
  decimal: true,  // affiche la touche virgule / point
  signed: true,   // affiche la touche moins (utile pour les temperatures)
),
```

`signed: true` est indispensable ici : sans lui, un utilisateur d'iPhone ne peut pas taper `-40` en Celsius.

### 57.14.3 — Les formatteurs

`inputFormatters` attend une liste de `TextInputFormatter`, importés de `package:flutter/services.dart`. Flutter en fournit deux familles.

| Formatteur | Effet |
| --- | --- |
| `FilteringTextInputFormatter.digitsOnly` | Ne laisse passer que `0-9` |
| `FilteringTextInputFormatter.allow(RegExp(...))` | Ne laisse passer que les caractères qui correspondent |
| `FilteringTextInputFormatter.deny(RegExp(...))` | Bloque les caractères qui correspondent |
| `LengthLimitingTextInputFormatter(n)` | Limite la longueur totale |

`digitsOnly` ne convient pas : il refuserait la virgule et le signe moins. Nous utilisons donc `allow` avec une **classe de caractères**.

```dart
FilteringTextInputFormatter.allow(RegExp(r'[0-9.,\-]')),
```

Attention à un point subtil, qui piège beaucoup de monde : `FilteringTextInputFormatter.allow` ne teste pas la chaîne dans son ensemble. Il parcourt le texte, garde tous les **morceaux** qui correspondent au motif, et les recolle. Avec une classe de caractères, le comportement est simple à prévoir : chaque caractère est gardé ou jeté individuellement.

Si vous écrivez à la place une expression complète comme `RegExp(r'^-?\d+([.,]\d*)?$')`, vous obtiendrez des résultats surprenants, parce que le formatteur va tenter d'extraire des sous-chaînes correspondantes. **Utilisez une classe de caractères, et laissez `tryParse` juger de la structure.**

### 57.14.4 — Le fichier `lecture_nombre.dart`

Avant d'écrire l'interface, isolons la conversion « texte saisi vers nombre » dans une fonction pure, testable.

**Fichier : `lib/logique/lecture_nombre.dart`**

```dart
/// Convertit le texte saisi par l'utilisateur en nombre.
///
/// Renvoie `null` si le texte n'est pas un nombre exploitable. C'est le
/// principe de `tryParse` du chapitre 13 : on ne leve pas d'exception pour
/// une saisie incomplete, car l'utilisateur est en train de taper.
///
/// La fonction gere trois particularites francaises :
///  - la virgule decimale : "3,14" doit etre compris comme 3.14 ;
///  - les espaces de groupement, y compris l'espace insecable etroite
///    que collent certains claviers et le presse-papiers ;
///  - le signe moins, autorise uniquement en tete.
double? lireNombre(String texte) {
  // 1. On enleve tout ce qui n'est que decoration.
  String normalise = texte
      .replaceAll(' ', '') // espace ordinaire
      .replaceAll(' ', '') // espace insecable
      .replaceAll(' ', '') // espace insecable etroite (separateur fr)
      .replaceAll("'", ''); // apostrophe (separateur suisse)

  // 2. La virgule francaise devient le point que Dart comprend.
  normalise = normalise.replaceAll(',', '.');

  // 3. Une chaine vide n'est pas un nombre : l'utilisateur a tout efface.
  if (normalise.isEmpty) {
    return null;
  }

  // 4. `tryParse` fait le reste. Il refuse "-", ".", "1.2.3", "--5"...
  //    et renvoie `null` au lieu de lever une exception.
  final double? valeur = double.tryParse(normalise);

  // 5. On refuse aussi l'infini et NaN, qu'un texte comme "Infinity"
  //    pourrait produire si un formatteur laissait passer des lettres.
  if (valeur == null || !valeur.isFinite) {
    return null;
  }

  return valeur;
}

/// Message a afficher quand [lireNombre] a renvoye `null`.
///
/// On distingue le champ vide (silencieux : l'utilisateur n'a pas encore
/// tape) du champ mal rempli (message d'erreur explicite).
String? messageDeSaisie(String texte) {
  if (texte.trim().isEmpty) {
    return null; // rien a reprocher, le champ est simplement vide
  }
  if (lireNombre(texte) == null) {
    return 'Saisie incomplete ou invalide.';
  }
  return null;
}
```

### 57.14.5 — `int.tryParse` ou `double.tryParse` ?

Question légitime : pourquoi ne pas utiliser `int.tryParse` quand l'utilisateur tape un entier ?

| Fonction | `'5'` | `'5.2'` | `'abc'` | `'5e3'` |
| --- | --- | --- | --- | --- |
| `int.tryParse` | `5` | `null` | `null` | `null` |
| `double.tryParse` | `5.0` | `5.2` | `null` | `5000.0` |

`int.tryParse` refuse `'5.2'`. Pour un convertisseur, ce serait absurde. **On lit toujours en `double`.**

`int.tryParse` reste utile ailleurs : un nombre de vies, une quantité, un identifiant. Règle simple : si la grandeur est **dénombrable**, `int` ; si elle est **mesurable**, `double`.

Notez au passage que `double.tryParse` accepte la notation scientifique `'5e3'`. C'est un cadeau : votre application saura lire `1e-9` sans une ligne de code de plus.

### 57.14.6 — Les tests du parsing

**Fichier : `test/lecture_nombre_test.dart`**

```dart
import 'package:convertisseur/logique/lecture_nombre.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('Saisies valides', () {
    test('un entier', () => expect(lireNombre('42'), 42.0));
    test('un decimal avec point', () => expect(lireNombre('3.14'), 3.14));
    test('un decimal avec virgule', () => expect(lireNombre('3,14'), 3.14));
    test('un negatif', () => expect(lireNombre('-40'), -40.0));
    test('un zero', () => expect(lireNombre('0'), 0.0));
    test('sans zero devant la virgule', () => expect(lireNombre(',5'), 0.5));
    test('notation scientifique', () => expect(lireNombre('1e3'), 1000.0));
    test('espace de groupement', () => expect(lireNombre('1 234,5'), 1234.5));
    test('espace insecable etroite', () {
      expect(lireNombre('1 234'), 1234.0);
    });
  });

  group('Saisies invalides', () {
    test('chaine vide', () => expect(lireNombre(''), isNull));
    test('signe moins seul', () => expect(lireNombre('-'), isNull));
    test('separateur seul', () => expect(lireNombre(','), isNull));
    test('deux separateurs', () => expect(lireNombre('1,2,3'), isNull));
    test('deux signes', () => expect(lireNombre('--5'), isNull));
    test('des lettres', () => expect(lireNombre('12ab'), isNull));
    test('infini', () => expect(lireNombre('Infinity'), isNull));
  });

  group('Messages', () {
    test('champ vide : pas de message', () => expect(messageDeSaisie(''), isNull));
    test('champ correct : pas de message', () {
      expect(messageDeSaisie('12,5'), isNull);
    });
    test('champ fautif : message', () {
      expect(messageDeSaisie('12,,5'), isNotNull);
    });
  });
}
```

Deux cas méritent un commentaire.

`lireNombre(',5')` renvoie `0.5` : après normalisation, la chaîne devient `'.5'`, que `double.tryParse` accepte. C'est le comportement voulu, un utilisateur pressé tape souvent `,5`.

`lireNombre('-')` renvoie `null` : l'utilisateur a commencé à taper une température négative. On ne doit **pas** afficher une erreur agressive dans ce cas ; c'est pourquoi `messageDeSaisie` existe séparément et reste discret tant que le champ n'est pas manifestement fautif.

**État exécutable.** `flutter test` passe. Le champ de l'application ne filtre pas encore : c'est l'objet de l'étape suivante, où l'on assemble tout.

---

## 57.15 — Étape 11 : le champ numérique complet

### 57.15.1 — Extraire un widget

La page commence à grossir. Sortons le champ dans son propre fichier. Un widget extrait, c'est un widget qu'on peut relire, réutiliser et `const`ifier.

**Fichier : `lib/widgets/champ_valeur.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart'; // pour les inputFormatters

/// Le champ de saisie numerique du convertisseur.
///
/// Ce widget ne calcule rien et ne connait aucune unite. Il se contente de
/// recevoir un controleur et de prevenir son parent a chaque frappe.
/// C'est un widget « bete », donc reutilisable partout.
class ChampValeur extends StatelessWidget {
  const ChampValeur({
    super.key,
    required this.controleur,
    required this.onChanged,
    required this.onEffacer,
    this.messageErreur,
    this.suffixe,
  });

  /// Controleur possede par le PARENT.
  ///
  /// Ce widget ne le cree pas et ne le detruit pas : celui qui cree est
  /// celui qui detruit.
  final TextEditingController controleur;

  /// Appele a chaque frappe.
  final ValueChanged<String> onChanged;

  /// Appele quand l'utilisateur touche la gomme.
  final VoidCallback onEffacer;

  /// Message affiche en rouge sous le champ, ou `null` si tout va bien.
  final String? messageErreur;

  /// Symbole de l'unite source, affiche a droite ("km", "°C"...).
  final String? suffixe;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: controleur,
      onChanged: onChanged,
      autofocus: true,

      // DEFENSE 1 : le bon clavier.
      keyboardType: const TextInputType.numberWithOptions(
        decimal: true,
        signed: true,
      ),

      // DEFENSE 2 : le filtrage caractere par caractere.
      inputFormatters: <TextInputFormatter>[
        // Chiffres, separateurs decimaux, signe moins, notation scientifique.
        FilteringTextInputFormatter.allow(RegExp(r'[0-9.,\-eE]')),
        // Une valeur de 20 caracteres est deja demesuree ; au-dela, on
        // depasse de toute facon la precision d'un double.
        LengthLimitingTextInputFormatter(20),
      ],

      style: Theme.of(context).textTheme.headlineSmall,
      textInputAction: TextInputAction.done,
      decoration: InputDecoration(
        border: const OutlineInputBorder(),
        labelText: 'Valeur a convertir',
        hintText: 'Saisissez un nombre',
        errorText: messageErreur,
        suffixText: suffixe,
        prefixIcon: const Icon(Icons.tag),
        suffixIcon: IconButton(
          icon: const Icon(Icons.backspace_outlined),
          tooltip: 'Effacer',
          onPressed: onEffacer,
        ),
      ),
    );
  }
}
```

### 57.15.2 — Ce que le filtre laisse encore passer

Le filtre autorise `e` et `E` pour la notation scientifique. Il autorise donc aussi de taper `eee`, `1e`, `--`, `1,2,3`. C'est **normal et voulu** : un formatteur trop malin empêche de taper une valeur intermédiaire légitime.

Exemple concret. Un utilisateur veut saisir `1,5`. Il tape `1`, puis `,`. À cet instant précis, le texte du champ vaut `'1,'`, qui n'est pas un nombre valide. Si le formatteur avait refusé la virgule, l'utilisateur n'aurait jamais pu écrire `1,5`.

```text
FRAPPE     TEXTE     lireNombre       AFFICHAGE
  1         "1"         1.0           resultat affiche
  ,         "1,"        null          on garde le dernier resultat
  5         "1,5"       1.5           resultat mis a jour
```

D'où la règle d'ergonomie :

> Pendant que l'utilisateur tape, une saisie temporairement invalide n'est pas une erreur. On n'efface pas le résultat précédent, on ne hurle pas en rouge.

### 57.15.3 — La table des cas de saisie

| Saisie | Filtré ? | `lireNombre` | Comportement de l'application |
| --- | --- | --- | --- |
| `12` | passe | `12.0` | conversion |
| `12,5` | passe | `12.5` | conversion |
| `-40` | passe | `-40.0` | conversion |
| `1e3` | passe | `1000.0` | conversion |
| `a` | **bloqué** | — | rien ne s'affiche dans le champ |
| `€` | **bloqué** | — | rien ne s'affiche dans le champ |
| `-` | passe | `null` | champ en cours de frappe, résultat gelé |
| `1,2,3` | passe | `null` | message « Saisie incomplete ou invalide » |
| (vide) | passe | `null` | résultat vide, pas de message |

**État exécutable.** Ce widget compile mais n'est pas encore branché : l'étape 12 le fait, en même temps que les listes déroulantes.

---

## 57.16 — Étape 12 : les deux listes déroulantes d'unités

### 57.16.1 — `DropdownButton<T>` typé

Beaucoup de tutoriels utilisent `DropdownButton<String>` et manipulent des chaînes. C'est une erreur : il faut ensuite retrouver l'objet à partir de la chaîne, et l'on réintroduit exactement le problème que l'`enum` avait supprimé.

`DropdownButton` est **générique**. On peut donc lui faire manipuler directement nos objets `Unite`.

```dart
DropdownButton<Unite>(
  value: uniteSelectionnee,           // de type Unite
  items: <DropdownMenuItem<Unite>>[...],
  onChanged: (Unite? nouvelle) { ... }, // on recoit un Unite
)
```

Aucune conversion, aucun `firstWhere`, aucun risque de faute de frappe.

### 57.16.2 — Les trois règles de `DropdownButton`

| Règle | Conséquence si violée |
| --- | --- |
| `value` doit être **égal** à exactement un `item` | Assertion `There should be exactly one item...` |
| `onChanged` reçoit un `T?` **nullable** | Erreur de compilation si vous écrivez `(Unite nouvelle)` |
| `onChanged: null` désactive le bouton | Le menu ne s'ouvre plus, l'apparence devient grise |

La première règle est celle qui casse le plus d'applications. Elle est satisfaite ici grâce au `==` par identifiant écrit à l'étape 5, plus la précaution de construire la liste une seule fois.

### 57.16.3 — Le sélecteur

**Fichier : `lib/widgets/selecteur_unite.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/unite.dart';

/// Une liste deroulante d'unites.
///
/// Generique sur `Unite` : on ne manipule jamais de chaines de caracteres,
/// donc on ne peut pas se tromper d'unite par faute de frappe.
class SelecteurUnite extends StatelessWidget {
  const SelecteurUnite({
    super.key,
    required this.etiquette,
    required this.unites,
    required this.selection,
    required this.onChanged,
  });

  /// Titre au-dessus du menu : "De" ou "Vers".
  final String etiquette;

  /// Les unites proposees. Elles appartiennent TOUTES a la meme categorie :
  /// c'est le filtrage de l'etape 13.
  final List<Unite> unites;

  /// L'unite actuellement choisie. Doit etre presente dans [unites].
  final Unite selection;

  /// Appele quand l'utilisateur choisit une autre unite.
  final ValueChanged<Unite> onChanged;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Text(etiquette, style: theme.textTheme.labelLarge),
        const SizedBox(height: 4),
        DecoratedBox(
          decoration: BoxDecoration(
            border: Border.all(color: theme.colorScheme.outline),
            borderRadius: BorderRadius.circular(4),
          ),
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12),
            child: DropdownButton<Unite>(
              value: selection,

              // isExpanded : sans lui, un nom long provoque un debordement
              // horizontal (le fameux RenderFlex overflowed du chapitre 46).
              isExpanded: true,

              // underline vide : la bordure est deja dessinee par le
              // DecoratedBox ci-dessus.
              underline: const SizedBox.shrink(),

              items: unites.map((Unite unite) {
                return DropdownMenuItem<Unite>(
                  value: unite,
                  child: Text(
                    unite.libelle,
                    overflow: TextOverflow.ellipsis,
                  ),
                );
              }).toList(),

              // Le parametre est nullable : Flutter peut passer `null`.
              onChanged: (Unite? nouvelle) {
                if (nouvelle != null) {
                  onChanged(nouvelle);
                }
              },
            ),
          ),
        ),
      ],
    );
  }
}
```

### 57.16.4 — Le filtrage par catégorie

L'exigence O5 impose qu'on ne puisse jamais convertir des kilomètres en litres. Deux stratégies :

| Stratégie | Principe | Verdict |
| --- | --- | --- |
| Corrective | Tout proposer, refuser après coup | Mauvaise : l'utilisateur découvre l'erreur trop tard |
| **Préventive** | Ne proposer que ce qui est valide | **Bonne** : l'erreur devient impossible |

On applique la préventive. Le filtrage est un simple `where` du chapitre 14 :

```dart
final List<Unite> unitesDeLaCategorie = toutesLesUnites
    .where((Unite unite) => unite.categorie == widget.categorie)
    .toList(growable: false);
```

Notre catalogue étant déjà organisé par catégorie, la fonction `unitesDe(categorie)` fait la même chose sans parcourir la liste complète. Les deux sont corrects ; retenez le `where` pour les cas où vos données ne sont pas pré-triées.

### 57.16.5 — Le filtrage doit se faire une seule fois

Point crucial. **N'écrivez pas** le `where` dans `build`.

```dart
// A NE PAS FAIRE
@override
Widget build(BuildContext context) {
  final unites = toutesLesUnites.where(...).toList(); // recalcule a chaque frappe
  ...
}
```

`build` est appelé à chaque frappe de clavier, chaque animation, chaque rotation d'écran. Recréer la liste à chaque fois est du gaspillage, et surtout cela recrée des objets `DropdownMenuItem` en permanence.

La bonne place est `initState` :

```dart
late final List<Unite> _unites;

@override
void initState() {
  super.initState();
  _unites = unitesDe(widget.categorie);
}
```

`late final` exprime exactement l'intention : « calculé plus tard, mais une seule fois, et jamais remplacé ».

**État exécutable.** Ce widget compile isolément ; il est assemblé à l'étape suivante.

---

## 57.17 — Étape 13 : la conversion en temps réel et l'inversion

### 57.17.1 — Le cycle complet

Voici, de bout en bout, ce qui se passe quand l'utilisateur appuie sur une touche.

```text
   [ frappe clavier ]
           │
           v
   TextField.onChanged("42,1")
           │
           v
   _surSaisie("42,1")
           │
           ├── lireNombre("42,1")  ->  42.1        (logique pure)
           │
           ├── convertir(42.1, km, mi)  ->  26.16… (logique pure)
           │
           └── setState(() { _resultat = 26.16…; })
                       │
                       v
               build() reconstruit
                       │
                       v
               [ ecran mis a jour ]
```

Rien d'asynchrone, rien de coûteux. Une conversion est une multiplication : on peut sans crainte la refaire à chaque frappe. Ce ne serait pas le cas s'il fallait interroger un serveur — c'est justement le sujet du chapitre 61.

### 57.17.2 — Où placer le calcul

Deux emplacements possibles, et un seul est bon.

```dart
// OPTION A — calculer dans build()
@override
Widget build(BuildContext context) {
  final double? valeur = lireNombre(_controleur.text);
  final double? resultat = valeur == null
      ? null
      : convertir(valeur: valeur, depuis: _source, vers: _cible);
  ...
}

// OPTION B — calculer dans le gestionnaire, stocker le resultat
void _recalculer() {
  final double? valeur = lireNombre(_controleur.text);
  setState(() {
    _resultat = valeur == null
        ? null
        : convertir(valeur: valeur, depuis: _source, vers: _cible);
  });
}
```

L'option A est plus courte et souvent suffisante. Nous prenons pourtant l'**option B**, pour trois raisons :

1. le résultat doit être **conservé** quand la saisie devient temporairement invalide (`'1,'`) ; en option A il disparaîtrait ;
2. le résultat doit être disponible hors de `build` pour l'enregistrer dans l'historique ;
3. le calcul est fait **une fois par changement réel**, et non à chaque reconstruction (rotation d'écran, changement de thème, ouverture du clavier).

### 57.17.3 — L'inversion

Le bouton d'inversion échange source et cible. Une seule subtilité : il faut recalculer **après** l'échange, dans le même `setState`.

```dart
void _inverser() {
  setState(() {
    final Unite ancienne = _source;
    _source = _cible;
    _cible = ancienne;
  });
  _recalculer();
}
```

Un raccourci Dart permet d'écrire l'échange sans variable temporaire, grâce aux enregistrements (*records*) :

```dart
(_source, _cible) = (_cible, _source);
```

Les deux formes sont correctes. La première est plus explicite pour un débutant, la seconde est celle que vous verrez dans du code moderne.

### 57.17.4 — Que fait l'inversion du champ de saisie ?

Question de conception, à trancher explicitement. Deux comportements existent dans les applications réelles.

| Comportement | Exemple | Effet |
| --- | --- | --- |
| **A — Échanger les unités seulement** | 5 km → 3,107 mi devient 5 mi → 8,047 km | La valeur saisie ne bouge pas |
| **B — Échanger unités et valeurs** | 5 km → 3,107 mi devient 3,107 mi → 5 km | Le résultat remonte dans le champ |

Nous choisissons **A**, pour une raison précise : le comportement B **perd de la précision**. Le champ ne contient que le résultat arrondi à l'affichage ; le réinjecter fait dériver la valeur à chaque appui. Faites l'expérience une fois que l'application marche, en appuyant vingt fois de suite sur le bouton.

Le comportement B est proposé comme défi 3, avec la solution du problème de précision.

### 57.17.5 — La page complète

**Fichier : `lib/ecrans/page_categorie.dart`**

```dart
import 'package:flutter/material.dart';

import '../donnees/catalogue.dart';
import '../logique/convertisseur.dart';
import '../logique/lecture_nombre.dart';
import '../modeles/categorie.dart';
import '../modeles/unite.dart';
import '../widgets/champ_valeur.dart';
import '../widgets/selecteur_unite.dart';

/// Un convertisseur complet pour UNE categorie.
///
/// Version 2 : saisie + deux menus + inversion + conversion temps reel.
/// Le formatage du resultat arrive a l'etape 14 ; pour l'instant on affiche
/// le double brut, ce qui permet de voir la valeur exacte.
class PageCategorie extends StatefulWidget {
  const PageCategorie({super.key, required this.categorie});

  final Categorie categorie;

  @override
  State<PageCategorie> createState() => _PageCategorieState();
}

class _PageCategorieState extends State<PageCategorie> {
  late final TextEditingController _controleur;

  /// Les unites de cette categorie, calculees UNE SEULE FOIS.
  late final List<Unite> _unites;

  late Unite _source;
  late Unite _cible;

  /// Dernier resultat valide. Reste affiche si la saisie devient invalide.
  double? _resultat;

  /// Message d'erreur sous le champ, ou `null`.
  String? _messageErreur;

  @override
  void initState() {
    super.initState();
    _controleur = TextEditingController(text: '1');
    _unites = unitesDe(widget.categorie);
    _source = uniteSourceParDefaut(widget.categorie);
    _cible = uniteCibleParDefaut(widget.categorie);
    _recalculer();
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  /// Recalcule le resultat a partir de l'etat courant.
  ///
  /// Ne fait AUCUNE hypothese sur ce qui a change : saisie, unite source ou
  /// unite cible. Un seul chemin de calcul, donc un seul endroit ou se
  /// tromper.
  void _recalculer() {
    final String texte = _controleur.text;
    final double? valeur = lireNombre(texte);

    setState(() {
      _messageErreur = messageDeSaisie(texte);

      if (valeur == null) {
        // On NE remet PAS _resultat a null : l'utilisateur est peut-etre
        // en train de taper "1," et perdre l'affichage serait desagreable.
        return;
      }

      _resultat = convertir(valeur: valeur, depuis: _source, vers: _cible);
    });
  }

  void _surSaisie(String texte) => _recalculer();

  void _effacer() {
    _controleur.clear();
    setState(() {
      _resultat = null;
      _messageErreur = null;
    });
  }

  void _changerSource(Unite unite) {
    setState(() => _source = unite);
    _recalculer();
  }

  void _changerCible(Unite unite) {
    setState(() => _cible = unite);
    _recalculer();
  }

  void _inverser() {
    setState(() {
      final Unite ancienne = _source;
      _source = _cible;
      _cible = ancienne;
    });
    _recalculer();
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return ListView(
      padding: const EdgeInsets.fromLTRB(16, 16, 16, 32),
      children: <Widget>[
        ChampValeur(
          controleur: _controleur,
          onChanged: _surSaisie,
          onEffacer: _effacer,
          messageErreur: _messageErreur,
          suffixe: _source.symbole,
        ),
        const SizedBox(height: 24),
        SelecteurUnite(
          etiquette: 'De',
          unites: _unites,
          selection: _source,
          onChanged: _changerSource,
        ),
        const SizedBox(height: 8),
        Center(
          child: IconButton.filledTonal(
            onPressed: _inverser,
            icon: const Icon(Icons.swap_vert),
            tooltip: 'Inverser les unites',
          ),
        ),
        const SizedBox(height: 8),
        SelecteurUnite(
          etiquette: 'Vers',
          unites: _unites,
          selection: _cible,
          onChanged: _changerCible,
        ),
        const SizedBox(height: 24),
        Card(
          child: Padding(
            padding: const EdgeInsets.all(24),
            child: Center(
              child: Text(
                // Affichage brut, provisoire : l'etape 14 le remplace.
                _resultat == null ? '—' : '$_resultat ${_cible.symbole}',
                style: theme.textTheme.headlineMedium,
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

**État exécutable.** L'application convertit. Tapez `5` avec « Kilomètre » vers « Mile » :

```text
3.1068559611866697 mi
```

C'est juste, et c'est illisible. L'étape suivante s'en occupe.

---

## 57.18 — Étape 14 : formater le résultat avec `intl`

### 57.18.1 — Pourquoi `toStringAsFixed` ne suffit pas

Dart offre déjà deux méthodes de formatage. Elles ont chacune un défaut rédhibitoire.

| Expression | Résultat | Problème |
| --- | --- | --- |
| `3.1068559611866697.toString()` | `3.1068559611866697` | Illisible |
| `3.1068559611866697.toStringAsFixed(2)` | `3.11` | Pas de séparateur de milliers |
| `1234567.891.toStringAsFixed(2)` | `1234567.89` | Point décimal anglais, en français |
| `0.0000012.toStringAsFixed(2)` | `0.00` | Le résultat disparaît |

`toStringAsFixed` ignore la langue de l'utilisateur. En France on écrit `1 234 567,89`, pas `1234567.89`. C'est le rôle du paquet `intl` (pour *internationalization*).

### 57.18.2 — `NumberFormat` en trois lignes

```dart
import 'package:intl/intl.dart';

final NumberFormat format = NumberFormat.decimalPattern('fr_FR');
print(format.format(1234567.891));
```

**Résultat :**

```text
1 234 567,891
```

Notez deux choses.

1. Le séparateur de milliers affiché par `intl` en français n'est **pas** une espace ordinaire, mais une espace insécable. Visuellement identique, différente en mémoire. C'est pour cela que `lireNombre` la retire explicitement à l'étape 10 : un copier-coller du résultat vers le champ doit fonctionner.
2. Contrairement à `DateFormat`, `NumberFormat` **n'a pas besoin** d'appeler `initializeDateFormatting`. Les symboles numériques de toutes les langues sont compilés dans le paquet. Vous pouvez formater en `fr_FR` dès la première ligne de `main`.

### 57.18.3 — Les constructeurs utiles

| Constructeur | Usage | Exemple de sortie (fr_FR) |
| --- | --- | --- |
| `NumberFormat.decimalPattern(locale)` | Le cas général | `1 234,568` |
| `NumberFormat.decimalPatternDigits(locale: l, decimalDigits: n)` | Nombre fixe de décimales | `1 234,57` |
| `NumberFormat.compact(locale: l)` | Notation abrégée | `1,2 M` |
| `NumberFormat.scientificPattern(locale)` | Notation scientifique | `1,235E6` |
| `NumberFormat.percentPattern(locale)` | Pourcentages | `45 %` |
| `NumberFormat.currency(locale: l, symbol: '€')` | Monnaies | `1 234,57 €` |

Deux propriétés modifiables complètent le tableau :

```dart
final NumberFormat f = NumberFormat.decimalPattern('fr_FR')
  ..minimumFractionDigits = 0   // ne pas afficher "5,0000" pour 5
  ..maximumFractionDigits = 4;  // ne pas afficher 17 decimales
```

Attention à l'ordre : le *setter* de `maximumFractionDigits` abaisse automatiquement le minimum s'il le dépasse. Réglez donc le maximum **avant** le minimum si vous fixez les deux.

### 57.18.4 — Combien de décimales ?

Un nombre fixe de décimales ne convient jamais.

```text
2 decimales fixes :
   26,22 mi          bien
   0,00 mm           catastrophe : le resultat a disparu
   1 234 567,00 o    ridicule : ces deux zeros n'apprennent rien
```

La bonne règle dépend de l'**ordre de grandeur** du résultat. Plus le nombre est petit, plus il faut de décimales pour qu'il reste informatif.

| Valeur absolue | Décimales | Exemple |
| --- | --- | --- |
| ≥ 1 000 | 2 | `1 234,57` |
| ≥ 1 | 4 | `26,2188` |
| ≥ 0,001 | 6 | `0,001578` |
| ≥ 0 | 8 | `0,00000125` |

Le zéro exact est traité à part : on écrit `0`, jamais `0,00000000`.

### 57.18.5 — Les nombres extrêmes

Deux zones échappent à toute règle de décimales.

```text
1 To en bits          = 8 000 000 000 000 bits    -> lisible mais tres long
1 To en millibits ?   -> n'existe pas, heureusement
1 mm en km            = 0,000001 km                -> 6 zeros
1 mm en annees-lumiere -> 1,057e-19                -> impossible en decimal
```

Au-delà de `1e15`, un `double` ne représente même plus les entiers de façon exacte. En deçà de `1e-6`, l'écriture décimale devient une bouillie de zéros.

On bascule donc en **notation scientifique** hors de l'intervalle `[1e-6 ; 1e15[`.

### 57.18.6 — Le code

**Fichier : `lib/logique/formatage.dart`**

```dart
import 'package:intl/intl.dart';

/// Langue utilisee pour tous les affichages numeriques.
///
/// Dans une vraie application on lirait la langue du systeme ; ici on fige
/// le francais pour que les copies d'ecran du cours soient reproductibles.
const String localeParDefaut = 'fr_FR';

/// Seuil au-dela duquel on passe en notation scientifique.
const double _seuilHaut = 1e15;

/// Seuil en deca duquel on passe en notation scientifique.
const double _seuilBas = 1e-6;

/// Nombre de decimales adapte a l'ordre de grandeur.
///
/// Fonction pure, donc testable seule.
int decimalesPour(double valeur) {
  final double absolue = valeur.abs();
  if (absolue == 0) {
    return 0;
  }
  if (absolue >= 1000) {
    return 2;
  }
  if (absolue >= 1) {
    return 4;
  }
  if (absolue >= 0.001) {
    return 6;
  }
  return 8;
}

/// Met en forme un resultat de conversion.
///
/// Regles :
///  - `null` (saisie invalide)      -> un tiret cadratin ;
///  - non fini (NaN, infini)        -> un message explicite ;
///  - trop grand ou trop petit      -> notation scientifique ;
///  - sinon                         -> notation decimale locale, avec un
///    nombre de decimales adapte et SANS zeros inutiles a droite.
String formaterResultat(double? valeur, {String locale = localeParDefaut}) {
  if (valeur == null) {
    return '—';
  }
  if (valeur.isNaN) {
    return 'valeur impossible';
  }
  if (valeur.isInfinite) {
    return valeur.isNegative ? '-infini' : 'infini';
  }

  final double absolue = valeur.abs();
  if (absolue != 0 && (absolue >= _seuilHaut || absolue < _seuilBas)) {
    // NumberFormat.scientificPattern donne par exemple "1,057E-19".
    return NumberFormat.scientificPattern(locale).format(valeur);
  }

  final NumberFormat format = NumberFormat.decimalPattern(locale)
    ..maximumFractionDigits = decimalesPour(valeur)
    ..minimumFractionDigits = 0;

  return format.format(valeur);
}

/// Met en forme la valeur SAISIE, pour la ligne "42,195 km = 26,2188 mi".
///
/// On garde davantage de decimales que pour le resultat : ce que
/// l'utilisateur a tape ne doit jamais etre deforme a l'affichage.
String formaterSaisie(double valeur, {String locale = localeParDefaut}) {
  final NumberFormat format = NumberFormat.decimalPattern(locale)
    ..maximumFractionDigits = 10
    ..minimumFractionDigits = 0;
  return format.format(valeur);
}
```

### 57.18.7 — Pourquoi `minimumFractionDigits = 0`

Sans cette ligne, `NumberFormat` complète avec des zéros :

```text
minimumFractionDigits = 4  ->  5 devient "5,0000"
minimumFractionDigits = 0  ->  5 devient "5"
```

Pour un convertisseur, la seconde forme est la bonne : `1 m = 100 cm`, pas `1,0000 m = 100,0000 cm`.

### 57.18.8 — Les tests du formatage

**Fichier : `test/formatage_test.dart`**

```dart
import 'package:convertisseur/logique/formatage.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('decimalesPour', () {
    test('zero', () => expect(decimalesPour(0), 0));
    test('grand nombre', () => expect(decimalesPour(1234.5), 2));
    test('nombre courant', () => expect(decimalesPour(26.21), 4));
    test('petit nombre', () => expect(decimalesPour(0.0015), 6));
    test('tres petit nombre', () => expect(decimalesPour(0.0000015), 8));
    test('le signe ne compte pas', () => expect(decimalesPour(-1234.5), 2));
  });

  // On teste en 'en_US' : les separateurs y sont des caracteres ASCII
  // ordinaires (virgule et point), donc les chaines attendues sont
  // lisibles dans le fichier de test. Le francais utilise une espace
  // insecable qu'il serait fragile d'ecrire en dur.
  group('formaterResultat (en_US)', () {
    String f(double? v) => formaterResultat(v, locale: 'en_US');

    test('saisie invalide', () => expect(f(null), '—'));
    test('entier sans decimale inutile', () => expect(f(5), '5'));
    test('separateur de milliers', () => expect(f(1234567.891), '1,234,567.89'));
    test('quatre decimales sous 1000', () => expect(f(26.21875745), '26.2188'));
    test('six decimales sous 1', () => expect(f(0.00157828), '0.001578'));
    test('huit decimales sous 0,001', () => expect(f(0.00000125), '0.00000125'));
    test('zero exact', () => expect(f(0), '0'));
    test('negatif', () => expect(f(-40), '-40'));
    test('non fini', () => expect(f(double.nan), 'valeur impossible'));
    test('infini', () => expect(f(double.infinity), 'infini'));

    test('tres grand nombre : notation scientifique', () {
      expect(f(1e20), contains('E'));
    });
    test('tres petit nombre : notation scientifique', () {
      expect(f(1e-19), contains('E'));
    });
  });

  group('formaterResultat (fr_FR)', () {
    test('la virgule est le separateur decimal', () {
      expect(formaterResultat(26.2188), contains(','));
    });
    test('le point n est jamais utilise comme separateur decimal', () {
      expect(formaterResultat(26.2188), isNot(contains('.')));
    });
  });
}
```

**État exécutable.** `flutter test` passe. Le formatage est prêt, il reste à l'afficher joliment.

---

## 57.19 — Étape 15 : la carte de résultat

### 57.19.1 — Ce que doit montrer un bon résultat

Un convertisseur qui n'affiche qu'un nombre est frustrant : l'utilisateur ne sait plus ce qu'il a demandé. La carte affiche donc quatre informations :

```text
┌────────────────────────────────────────────┐
│                                            │
│            26,2188 mi                      │  <- 1. le resultat, en gros
│                                            │
│  42,195 km = 26,2188 mi                    │  <- 2. l'egalite complete
│                                            │
│  1 km = 0,621371 mi                        │  <- 3. le taux unitaire
│                            [ Enregistrer ] │  <- 4. l'action
└────────────────────────────────────────────┘
```

### 57.19.2 — Le code

**Fichier : `lib/widgets/carte_resultat.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart'; // pour le presse-papiers

import '../logique/convertisseur.dart';
import '../logique/formatage.dart';
import '../modeles/unite.dart';

/// L'encadre qui affiche le resultat d'une conversion.
///
/// Widget « bete » : il recoit des valeurs deja calculees et ne fait que
/// les mettre en forme. Il ne convertit rien lui-meme, sauf le taux
/// unitaire, qui est un simple appel a la fonction pure.
class CarteResultat extends StatelessWidget {
  const CarteResultat({
    super.key,
    required this.valeurSaisie,
    required this.resultat,
    required this.source,
    required this.cible,
    required this.onEnregistrer,
  });

  /// Valeur saisie par l'utilisateur, ou `null` si la saisie est invalide.
  final double? valeurSaisie;

  /// Resultat de la conversion, ou `null`.
  final double? resultat;

  final Unite source;
  final Unite cible;

  /// Appele quand l'utilisateur ajoute la conversion a l'historique.
  ///
  /// `null` desactive le bouton : c'est le cas quand il n'y a pas de
  /// resultat a enregistrer.
  final VoidCallback? onEnregistrer;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final String texteResultat = formaterResultat(resultat);

    // Le taux unitaire ne depend pas de la saisie : il vaut toujours
    // « combien vaut 1 source en cible ».
    final double taux = tauxUnitaire(depuis: source, vers: cible);

    return Card(
      elevation: 0,
      color: theme.colorScheme.surfaceContainerHighest,
      child: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            // 1. Le resultat, en gros.
            Center(
              child: SelectableText(
                '$texteResultat ${cible.symbole}',
                style: theme.textTheme.displaySmall,
                textAlign: TextAlign.center,
              ),
            ),
            const SizedBox(height: 12),

            // 2. L'egalite complete, pour se rappeler la question posee.
            if (valeurSaisie != null && resultat != null)
              Center(
                child: Text(
                  '${formaterSaisie(valeurSaisie!)} ${source.symbole}'
                  '  =  '
                  '$texteResultat ${cible.symbole}',
                  style: theme.textTheme.bodyMedium,
                  textAlign: TextAlign.center,
                ),
              ),

            const Divider(height: 24),

            // 3. Le taux unitaire.
            Text(
              '1 ${source.symbole} = ${formaterResultat(taux)} ${cible.symbole}',
              style: theme.textTheme.bodySmall,
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 8),

            // 4. Les actions.
            Row(
              mainAxisAlignment: MainAxisAlignment.end,
              children: <Widget>[
                TextButton.icon(
                  onPressed: resultat == null
                      ? null
                      : () {
                          Clipboard.setData(
                            ClipboardData(text: texteResultat),
                          );
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(
                              content: Text('Resultat copie.'),
                              duration: Duration(seconds: 1),
                            ),
                          );
                        },
                  icon: const Icon(Icons.copy_outlined),
                  label: const Text('Copier'),
                ),
                const SizedBox(width: 8),
                FilledButton.tonalIcon(
                  onPressed: onEnregistrer,
                  icon: const Icon(Icons.bookmark_add_outlined),
                  label: const Text('Enregistrer'),
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

### 57.19.3 — Le taux unitaire et la température

Le taux unitaire est parfaitement clair pour les longueurs : `1 km = 0,621371 mi`.

Il est **trompeur** pour la température. La carte affichera `1 °C = 33,8 °F`, ce qui est vrai en tant que température, mais faux si on le lit comme un facteur : un **écart** de 1 °C vaut 1,8 °F, pas 33,8.

C'est une conséquence directe de l'étape 4 : dans une échelle affine, le rapport `y/x` n'est pas constant. Une amélioration possible, laissée au défi 5, consiste à afficher pour la température une phrase différente : « un écart de 1 °C vaut 1,8 °F ».

**État exécutable.** Remplacez la `Card` provisoire de `page_categorie.dart` par `CarteResultat` :

```dart
CarteResultat(
  valeurSaisie: lireNombre(_controleur.text),
  resultat: _resultat,
  source: _source,
  cible: _cible,
  onEnregistrer: _resultat == null ? null : () {},
),
```

Tapez `42,195` en kilomètres vers miles :

```text
26,2188 mi
42,195 km  =  26,2188 mi
1 km = 0,621371 mi
```

---

## 57.20 — Étape 16 : les onglets par catégorie

### 57.20.1 — `DefaultTabController` ou `TabController` ?

Au chapitre 50, vous avez vu les deux formes. Rappel :

| Forme | Avantage | Limite |
| --- | --- | --- |
| `DefaultTabController` | Trois lignes, rien à détruire | On ne peut pas **écouter** les changements d'onglet |
| `TabController` explicite | On contrôle tout : index initial, écoute, changement programmé | Il faut un `State`, un `TickerProvider` et un `dispose` |

L'exigence O12 impose de **mémoriser** l'onglet actif. Il faut donc être prévenu quand l'utilisateur change d'onglet : c'est le `TabController` explicite.

### 57.20.2 — Le `vsync` et le mixin

Un `TabController` anime la transition entre onglets. Toute animation en Flutter a besoin d'un `TickerProvider`, c'est-à-dire d'un objet capable de dire « une nouvelle image vient d'être affichée ».

On l'obtient en ajoutant un mixin au `State` :

```dart
class _EcranAccueilState extends State<EcranAccueil>
    with SingleTickerProviderStateMixin {
```

`Single` parce qu'il n'y a **qu'une** animation dans cet écran. S'il y en avait plusieurs, ce serait `TickerProviderStateMixin` (sans `Single`). Utiliser le second quand un seul suffit fonctionne, mais consomme un peu plus ; utiliser le premier avec deux animations lève une assertion explicite.

### 57.20.3 — Le piège de la perte d'état

`TabBarView` ne construit que les pages voisines de la page courante. Quand vous glissez trois onglets plus loin, la première page est **détruite** : votre saisie disparaît.

Le remède est un second mixin, posé cette fois sur la page :

```dart
class _PageCategorieState extends State<PageCategorie>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context); // OBLIGATOIRE avec ce mixin
    ...
  }
}
```

L'appel `super.build(context)` est indispensable. L'oublier ne provoque pas d'erreur de compilation, mais l'état continue de se perdre, et l'on cherche longtemps.

### 57.20.4 — L'écran d'accueil

**Fichier : `lib/ecrans/ecran_accueil.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/categorie.dart';
import '../modeles/conversion.dart';
import '../services/preferences_service.dart';
import 'ecran_historique.dart';
import 'page_categorie.dart';

/// L'ecran principal : une barre d'onglets, une page par categorie.
///
/// Cet ecran est le proprietaire de l'historique : c'est lui qui detient la
/// liste, la complete et la transmet. Les pages ne font que remonter les
/// conversions par un callback (remontee d'etat, chapitre 45).
class EcranAccueil extends StatefulWidget {
  const EcranAccueil({super.key, required this.preferences});

  final PreferencesConvertisseur preferences;

  @override
  State<EcranAccueil> createState() => _EcranAccueilState();
}

class _EcranAccueilState extends State<EcranAccueil>
    with SingleTickerProviderStateMixin {
  late final TabController _onglets;

  /// L'historique, du plus recent au plus ancien.
  late List<Conversion> _historique;

  @override
  void initState() {
    super.initState();

    _historique = widget.preferences.lireHistorique();

    _onglets = TabController(
      length: Categorie.values.length,
      vsync: this,
      // On repart sur la categorie ou l'utilisateur s'etait arrete.
      initialIndex: widget.preferences.lireCategorie().index,
    );
    _onglets.addListener(_surChangementOnglet);
  }

  @override
  void dispose() {
    _onglets.removeListener(_surChangementOnglet);
    _onglets.dispose();
    super.dispose();
  }

  /// Enregistre l'onglet choisi.
  ///
  /// Le listener est appele DEUX fois par changement : au debut et a la fin
  /// de l'animation. `indexIsChanging` vaut `true` pendant l'animation ;
  /// on ne veut ecrire qu'une seule fois, a l'arrivee.
  void _surChangementOnglet() {
    if (_onglets.indexIsChanging) {
      return;
    }
    widget.preferences.enregistrerCategorie(Categorie.values[_onglets.index]);
  }

  void _ajouterAlHistorique(Conversion conversion) {
    setState(() {
      _historique = <Conversion>[conversion, ..._historique];
      // On borne la liste : un historique infini finit par ralentir
      // l'affichage et gonfler les preferences.
      if (_historique.length > 100) {
        _historique = _historique.sublist(0, 100);
      }
    });
    widget.preferences.enregistrerHistorique(_historique);

    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text('Conversion enregistree.'),
        duration: Duration(seconds: 1),
      ),
    );
  }

  void _viderHistorique() {
    setState(() => _historique = const <Conversion>[]);
    widget.preferences.enregistrerHistorique(_historique);
  }

  Future<void> _ouvrirHistorique() async {
    await Navigator.of(context).push(
      MaterialPageRoute<void>(
        builder: (BuildContext context) => EcranHistorique(
          conversions: _historique,
          onVider: _viderHistorique,
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Convertisseur'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.history),
            tooltip: 'Historique',
            onPressed: _ouvrirHistorique,
          ),
        ],
        bottom: TabBar(
          controller: _onglets,
          // Six onglets ne tiennent pas sur la largeur d'un telephone :
          // isScrollable les fait defiler au lieu de les compresser.
          isScrollable: true,
          tabs: Categorie.values
              .map((Categorie c) => Tab(text: c.libelle))
              .toList(),
        ),
      ),
      body: TabBarView(
        controller: _onglets,
        children: Categorie.values.map((Categorie categorie) {
          return PageCategorie(
            // La cle garantit que Flutter ne reutilise pas l'etat d'une
            // page pour une autre categorie lors d'une reconstruction.
            key: ValueKey<String>(categorie.identifiant),
            categorie: categorie,
            preferences: widget.preferences,
            onConversion: _ajouterAlHistorique,
          );
        }).toList(),
      ),
    );
  }
}
```

**État exécutable.** Les six onglets fonctionnent — dès que `preferences_service.dart` et `ecran_historique.dart` existent, ce qui est l'objet des deux étapes suivantes.

---

## 57.21 — Étape 17 : l'historique des conversions

### 57.21.1 — Que stocke-t-on exactement ?

Tentation du débutant : stocker les objets `Unite` eux-mêmes.

C'est une mauvaise idée pour la persistance. Une `Unite` contient deux **fonctions** : ce n'est pas sérialisable en JSON. Et si une unité disparaît du catalogue dans une version future, l'historique deviendrait illisible.

On stocke donc des données **plates et autonomes** : les nombres et les symboles, déjà résolus.

```text
MAUVAIS                     BON
Conversion {                Conversion {
  Unite source;               double valeurSource;
  Unite cible;                String symboleSource;
  ...                         double resultat;
}                             String symboleCible;
                              String libelleCategorie;
   -> non serialisable        DateTime date;
   -> depend du catalogue   }
                                -> JSON trivial, autonome
```

### 57.21.2 — Le modèle

**Fichier : `lib/modeles/conversion.dart`**

```dart
/// Une ligne d'historique : une conversion effectuee et enregistree.
///
/// Volontairement « plate » : que des types de base, aucune reference au
/// catalogue. Elle reste donc lisible meme si une unite disparait, et sa
/// serialisation JSON (chapitre 17) est immediate.
class Conversion {
  const Conversion({
    required this.valeurSource,
    required this.symboleSource,
    required this.resultat,
    required this.symboleCible,
    required this.libelleCategorie,
    required this.date,
  });

  final double valeurSource;
  final String symboleSource;
  final double resultat;
  final String symboleCible;
  final String libelleCategorie;
  final DateTime date;

  Map<String, Object?> versJson() {
    return <String, Object?>{
      'valeurSource': valeurSource,
      'symboleSource': symboleSource,
      'resultat': resultat,
      'symboleCible': symboleCible,
      'categorie': libelleCategorie,
      // Une date se serialise en texte ISO 8601 : trie, lisible, universel.
      'date': date.toIso8601String(),
    };
  }

  /// Reconstruit une conversion depuis du JSON.
  ///
  /// Renvoie `null` si le JSON est incomplet ou corrompu, au lieu de lever
  /// une exception : une preference abimee ne doit pas empecher
  /// l'application de demarrer.
  static Conversion? depuisJson(Map<String, Object?> json) {
    final Object? valeur = json['valeurSource'];
    final Object? resultat = json['resultat'];
    final Object? date = json['date'];

    if (valeur is! num || resultat is! num || date is! String) {
      return null;
    }
    final DateTime? quand = DateTime.tryParse(date);
    if (quand == null) {
      return null;
    }

    return Conversion(
      valeurSource: valeur.toDouble(),
      symboleSource: json['symboleSource'] as String? ?? '?',
      resultat: resultat.toDouble(),
      symboleCible: json['symboleCible'] as String? ?? '?',
      libelleCategorie: json['categorie'] as String? ?? '?',
      date: quand,
    );
  }
}
```

### 57.21.3 — L'écran d'historique

**Fichier : `lib/ecrans/ecran_historique.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/formatage.dart';
import '../modeles/conversion.dart';

/// La liste des conversions enregistrees.
///
/// Ecran purement consultatif : il recoit une liste deja construite et un
/// callback pour tout effacer. Il ne possede aucun etat.
class EcranHistorique extends StatelessWidget {
  const EcranHistorique({
    super.key,
    required this.conversions,
    required this.onVider,
  });

  final List<Conversion> conversions;
  final VoidCallback onVider;

  /// Transforme une date en phrase courte : "il y a 5 minutes".
  ///
  /// Volontairement ecrit a la main : `intl` sait le faire, mais cela
  /// demanderait de charger les donnees de dates, ce dont le chapitre n'a
  /// pas besoin.
  static String _depuis(DateTime date) {
    final Duration ecart = DateTime.now().difference(date);
    if (ecart.inSeconds < 60) {
      return 'a l instant';
    }
    if (ecart.inMinutes < 60) {
      return 'il y a ${ecart.inMinutes} min';
    }
    if (ecart.inHours < 24) {
      return 'il y a ${ecart.inHours} h';
    }
    return 'il y a ${ecart.inDays} j';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Historique'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.delete_outline),
            tooltip: 'Tout effacer',
            // onPressed a `null` desactive automatiquement le bouton
            // quand la liste est vide : pas besoin de `if`.
            onPressed: conversions.isEmpty
                ? null
                : () {
                    onVider();
                    Navigator.of(context).pop();
                  },
          ),
        ],
      ),
      body: conversions.isEmpty
          ? const Center(
              child: Padding(
                padding: EdgeInsets.all(32),
                child: Text(
                  'Aucune conversion enregistree.\n\n'
                  'Appuyez sur « Enregistrer » sous un resultat '
                  'pour le retrouver ici.',
                  textAlign: TextAlign.center,
                ),
              ),
            )
          : ListView.separated(
              itemCount: conversions.length,
              separatorBuilder: (_, __) => const Divider(height: 1),
              itemBuilder: (BuildContext context, int index) {
                final Conversion c = conversions[index];
                return ListTile(
                  leading: const Icon(Icons.swap_horiz),
                  title: Text(
                    '${formaterSaisie(c.valeurSource)} ${c.symboleSource}'
                    '  =  '
                    '${formaterResultat(c.resultat)} ${c.symboleCible}',
                  ),
                  subtitle: Text(
                    '${c.libelleCategorie} — ${_depuis(c.date)}',
                  ),
                );
              },
            ),
    );
  }
}
```

**État exécutable.** L'écran compile. Il reste à le remplir, donc à écrire le service de préférences.

---

## 57.22 — Étape 18 : la persistance du dernier choix

### 57.22.1 — Quelle API de `shared_preferences` ?

Le paquet propose aujourd'hui trois API. Le chapitre 54 les a présentées ; voici le rappel utile ici.

| API | Lecture | Écriture | Verdict |
| --- | --- | --- | --- |
| `SharedPreferences.getInstance()` | synchrone | asynchrone | Historique, déconseillée pour du neuf |
| `SharedPreferencesAsync` | **asynchrone** | asynchrone | Sûre, mais chaque lecture est un `await` |
| `SharedPreferencesWithCache` | **synchrone** | asynchrone | **Ce qu'il nous faut** |

Nous avons besoin de lectures **synchrones** : `initState` ne peut pas attendre. `SharedPreferencesWithCache` charge tout une fois au démarrage, puis répond instantanément.

### 57.22.2 — L'`allowList`

`SharedPreferencesWithCache.create` exige de déclarer la liste des clés utilisées :

```dart
SharedPreferencesWithCacheOptions(allowList: <String>{'derniere_categorie', ...})
```

Ce n'est pas une formalité. L'`allowList` limite ce que le cache charge au démarrage — donc le temps de lancement — et empêche votre code d'écraser par erreur une clé appartenant à une autre partie de l'application.

### 57.22.3 — Ce qu'on enregistre

```text
derniere_categorie        -> "longueur"
source_longueur           -> "kilometre"
cible_longueur            -> "mile"
source_temperature        -> "fahrenheit"
cible_temperature         -> "celsius"
...                          (un couple par categorie)
historique                -> ["{...}", "{...}", ...]   (liste de JSON)
```

Chaque catégorie garde **son propre** couple d'unités. C'est ce que l'utilisateur attend : revenir sur l'onglet Température ne doit pas y coller des kilomètres.

### 57.22.4 — Le service

**Fichier : `lib/services/preferences_service.dart`**

```dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

import '../donnees/catalogue.dart';
import '../modeles/categorie.dart';
import '../modeles/conversion.dart';
import '../modeles/unite.dart';

/// Lecture et ecriture des preferences de l'application.
///
/// Toutes les lectures sont SYNCHRONES (le cache est deja charge), toutes
/// les ecritures sont asynchrones et volontairement non attendues : perdre
/// une preference n'est pas grave, bloquer l'interface le serait.
class PreferencesConvertisseur {
  PreferencesConvertisseur(this._stockage);

  final SharedPreferencesWithCache _stockage;

  static const String _cleCategorie = 'derniere_categorie';
  static const String _cleHistorique = 'historique';

  static String _cleSource(Categorie c) => 'source_${c.identifiant}';
  static String _cleCible(Categorie c) => 'cible_${c.identifiant}';

  /// Toutes les cles utilisees par l'application.
  static Set<String> _cles() {
    final Set<String> cles = <String>{_cleCategorie, _cleHistorique};
    for (final Categorie c in Categorie.values) {
      cles.add(_cleSource(c));
      cles.add(_cleCible(c));
    }
    return cles;
  }

  /// Charge les preferences. A appeler UNE fois, dans `main`.
  static Future<PreferencesConvertisseur> charger() async {
    final SharedPreferencesWithCache stockage =
        await SharedPreferencesWithCache.create(
      cacheOptions: SharedPreferencesWithCacheOptions(allowList: _cles()),
    );
    return PreferencesConvertisseur(stockage);
  }

  // ---------------------------------------------------------------------
  // Categorie
  // ---------------------------------------------------------------------

  /// La derniere categorie utilisee, ou la longueur au premier lancement.
  ///
  /// Trois filets de securite : pas de valeur, valeur d'un type inattendu,
  /// identifiant inconnu. Aucun ne doit faire planter le demarrage.
  Categorie lireCategorie() {
    final String? identifiant = _stockage.getString(_cleCategorie);
    if (identifiant == null) {
      return Categorie.longueur;
    }
    return Categorie.parIdentifiant(identifiant) ?? Categorie.longueur;
  }

  void enregistrerCategorie(Categorie categorie) {
    // On n'attend pas : l'ecriture disque se fait en tache de fond.
    _stockage.setString(_cleCategorie, categorie.identifiant);
  }

  // ---------------------------------------------------------------------
  // Unites
  // ---------------------------------------------------------------------

  Unite lireUniteSource(Categorie categorie) {
    return _uniteValide(_stockage.getString(_cleSource(categorie)), categorie) ??
        uniteSourceParDefaut(categorie);
  }

  Unite lireUniteCible(Categorie categorie) {
    return _uniteValide(_stockage.getString(_cleCible(categorie)), categorie) ??
        uniteCibleParDefaut(categorie);
  }

  void enregistrerUnites(Categorie categorie, Unite source, Unite cible) {
    _stockage.setString(_cleSource(categorie), source.identifiant);
    _stockage.setString(_cleCible(categorie), cible.identifiant);
  }

  /// Verifie qu'un identifiant enregistre designe bien une unite EXISTANTE
  /// et appartenant a la BONNE categorie.
  ///
  /// Sans ce controle, une preference ancienne pourrait placer des litres
  /// dans l'onglet Longueur : le `DropdownButton` leverait alors son
  /// assertion « exactly one item ».
  Unite? _uniteValide(String? identifiant, Categorie categorie) {
    final Unite? unite = uniteParIdentifiant(identifiant);
    if (unite == null || unite.categorie != categorie) {
      return null;
    }
    return unite;
  }

  // ---------------------------------------------------------------------
  // Historique (bonus B5)
  // ---------------------------------------------------------------------

  List<Conversion> lireHistorique() {
    final List<String>? brut = _stockage.getStringList(_cleHistorique);
    if (brut == null) {
      return <Conversion>[];
    }

    final List<Conversion> resultat = <Conversion>[];
    for (final String ligne in brut) {
      try {
        final Object? decode = jsonDecode(ligne);
        if (decode is Map<String, Object?>) {
          final Conversion? conversion = Conversion.depuisJson(decode);
          if (conversion != null) {
            resultat.add(conversion);
          }
        }
      } on FormatException {
        // Ligne corrompue : on l'ignore et on continue. Le chapitre 13
        // insistait sur ce point : une erreur previsible se rattrape,
        // elle ne remonte pas jusqu'a l'utilisateur.
        continue;
      }
    }
    return resultat;
  }

  void enregistrerHistorique(List<Conversion> conversions) {
    final List<String> lignes = conversions
        .map((Conversion c) => jsonEncode(c.versJson()))
        .toList(growable: false);
    _stockage.setStringList(_cleHistorique, lignes);
  }
}
```

### 57.22.5 — Charger avant de dessiner

`main` devient asynchrone. Deux lignes sont obligatoires et souvent oubliées.

```dart
Future<void> main() async {
  // 1. Reveiller le moteur AVANT d'appeler un plugin.
  //    Sans cette ligne : "Binding has not yet been initialized".
  WidgetsFlutterBinding.ensureInitialized();

  // 2. Charger les preferences avant de construire l'interface.
  final PreferencesConvertisseur preferences =
      await PreferencesConvertisseur.charger();

  runApp(ApplicationConvertisseur(preferences: preferences));
}
```

L'alternative — construire l'interface puis charger dans un `FutureBuilder` — est possible mais fait clignoter l'écran au démarrage, et complique tout le code en aval. Ici le chargement dure quelques millisecondes : on l'attend.

**Fichier : `lib/main.dart`** (version finale)

```dart
import 'package:flutter/material.dart';

import 'ecrans/ecran_accueil.dart';
import 'services/preferences_service.dart';

Future<void> main() async {
  // Obligatoire avant tout appel de plugin depuis `main`.
  WidgetsFlutterBinding.ensureInitialized();

  final PreferencesConvertisseur preferences =
      await PreferencesConvertisseur.charger();

  runApp(ApplicationConvertisseur(preferences: preferences));
}

/// Racine de l'application.
class ApplicationConvertisseur extends StatelessWidget {
  const ApplicationConvertisseur({super.key, required this.preferences});

  final PreferencesConvertisseur preferences;

  @override
  Widget build(BuildContext context) {
    const Color graine = Color(0xFF00695C);

    return MaterialApp(
      title: 'Convertisseur',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: graine),
      ),
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: graine,
          brightness: Brightness.dark,
        ),
      ),
      // Le mode sombre suit le reglage du systeme (bonus B4).
      themeMode: ThemeMode.system,
      home: EcranAccueil(preferences: preferences),
    );
  }
}
```

**État exécutable.** Choisissez « Fahrenheit vers Celsius » dans l'onglet Température, fermez complètement l'application, relancez-la : vous retombez sur cet onglet, avec ces deux unités.

---

## 57.23 — Étape 19 : la page finale assemblée

Voici la version définitive de la page de catégorie. Elle réunit les étapes 9 à 18 : mixin de conservation d'état, préférences, historique, formatage.

**Fichier : `lib/ecrans/page_categorie.dart`** (version finale)

```dart
import 'package:flutter/material.dart';

import '../donnees/catalogue.dart';
import '../logique/convertisseur.dart';
import '../logique/lecture_nombre.dart';
import '../modeles/categorie.dart';
import '../modeles/conversion.dart';
import '../modeles/unite.dart';
import '../services/preferences_service.dart';
import '../widgets/carte_resultat.dart';
import '../widgets/champ_valeur.dart';
import '../widgets/selecteur_unite.dart';

/// Le convertisseur d'UNE categorie.
///
/// Cette page possede : le controleur de saisie, les deux unites choisies,
/// le dernier resultat. Elle ne possede PAS l'historique, qui appartient a
/// l'ecran d'accueil : elle le previent par le callback [onConversion].
class PageCategorie extends StatefulWidget {
  const PageCategorie({
    super.key,
    required this.categorie,
    required this.preferences,
    required this.onConversion,
  });

  final Categorie categorie;
  final PreferencesConvertisseur preferences;
  final ValueChanged<Conversion> onConversion;

  @override
  State<PageCategorie> createState() => _PageCategorieState();
}

class _PageCategorieState extends State<PageCategorie>
    with AutomaticKeepAliveClientMixin {
  late final TextEditingController _controleur;
  late final List<Unite> _unites;

  late Unite _source;
  late Unite _cible;

  double? _valeurSaisie;
  double? _resultat;
  String? _messageErreur;

  /// Demande a Flutter de NE PAS detruire cette page quand on change
  /// d'onglet. Sans cela, la saisie serait perdue au troisieme onglet.
  @override
  bool get wantKeepAlive => true;

  @override
  void initState() {
    super.initState();
    _controleur = TextEditingController(text: '1');
    _unites = unitesDe(widget.categorie);
    _source = widget.preferences.lireUniteSource(widget.categorie);
    _cible = widget.preferences.lireUniteCible(widget.categorie);
    _recalculer();
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  /// Unique chemin de calcul de la page.
  void _recalculer() {
    final String texte = _controleur.text;
    final double? valeur = lireNombre(texte);

    setState(() {
      _messageErreur = messageDeSaisie(texte);
      if (valeur == null) {
        // Saisie en cours : on garde le dernier resultat affiche.
        return;
      }
      _valeurSaisie = valeur;
      _resultat = convertir(valeur: valeur, depuis: _source, vers: _cible);
    });
  }

  void _effacer() {
    _controleur.clear();
    setState(() {
      _valeurSaisie = null;
      _resultat = null;
      _messageErreur = null;
    });
  }

  void _changerSource(Unite unite) {
    setState(() => _source = unite);
    _memoriserUnites();
    _recalculer();
  }

  void _changerCible(Unite unite) {
    setState(() => _cible = unite);
    _memoriserUnites();
    _recalculer();
  }

  void _inverser() {
    setState(() {
      // Echange par enregistrement (record) : pas de variable temporaire.
      (_source, _cible) = (_cible, _source);
    });
    _memoriserUnites();
    _recalculer();
  }

  void _memoriserUnites() {
    widget.preferences.enregistrerUnites(widget.categorie, _source, _cible);
  }

  /// Ajoute la conversion courante a l'historique.
  void _enregistrer() {
    final double? valeur = _valeurSaisie;
    final double? resultat = _resultat;
    if (valeur == null || resultat == null) {
      return;
    }

    widget.onConversion(
      Conversion(
        valeurSource: valeur,
        symboleSource: _source.symbole,
        resultat: resultat,
        symboleCible: _cible.symbole,
        libelleCategorie: widget.categorie.libelle,
        date: DateTime.now(),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // OBLIGATOIRE avec AutomaticKeepAliveClientMixin.
    super.build(context);

    final ThemeData theme = Theme.of(context);

    return ListView(
      padding: const EdgeInsets.fromLTRB(16, 16, 16, 40),
      children: <Widget>[
        ChampValeur(
          controleur: _controleur,
          onChanged: (_) => _recalculer(),
          onEffacer: _effacer,
          messageErreur: _messageErreur,
          suffixe: _source.symbole,
        ),
        const SizedBox(height: 20),
        SelecteurUnite(
          etiquette: 'De',
          unites: _unites,
          selection: _source,
          onChanged: _changerSource,
        ),
        const SizedBox(height: 4),
        Center(
          child: IconButton.filledTonal(
            onPressed: _inverser,
            icon: const Icon(Icons.swap_vert),
            tooltip: 'Inverser les unites',
          ),
        ),
        const SizedBox(height: 4),
        SelecteurUnite(
          etiquette: 'Vers',
          unites: _unites,
          selection: _cible,
          onChanged: _changerCible,
        ),
        const SizedBox(height: 24),
        CarteResultat(
          valeurSaisie: _valeurSaisie,
          resultat: _resultat,
          source: _source,
          cible: _cible,
          onEnregistrer: _resultat == null ? null : _enregistrer,
        ),
        const SizedBox(height: 16),
        Text(
          widget.categorie.description,
          style: theme.textTheme.bodySmall,
          textAlign: TextAlign.center,
        ),
      ],
    );
  }
}
```

### 57.23.1 — Récapitulatif des fichiers

| Fichier | Lignes environ | Importe Flutter ? |
| --- | --- | --- |
| `lib/modeles/categorie.dart` | 70 | non |
| `lib/modeles/unite.dart` | 100 | non |
| `lib/modeles/conversion.dart` | 60 | non |
| `lib/donnees/catalogue.dart` | 380 | non |
| `lib/logique/convertisseur.dart` | 60 | non |
| `lib/logique/lecture_nombre.dart` | 45 | non |
| `lib/logique/formatage.dart` | 70 | non |
| `lib/services/preferences_service.dart` | 130 | non |
| `lib/widgets/champ_valeur.dart` | 70 | oui |
| `lib/widgets/selecteur_unite.dart` | 65 | oui |
| `lib/widgets/carte_resultat.dart` | 105 | oui |
| `lib/ecrans/page_categorie.dart` | 190 | oui |
| `lib/ecrans/ecran_accueil.dart` | 130 | oui |
| `lib/ecrans/ecran_historique.dart` | 95 | oui |
| `lib/main.dart` | 50 | oui |

Huit fichiers sur quinze n'importent pas Flutter. Ce sont ceux qui contiennent la difficulté, et ce sont ceux que les tests couvrent entièrement.

**État exécutable.** L'application est terminée. `flutter analyze` ne signale rien, `flutter test` passe, `flutter run` fonctionne sur Android, iOS, web et bureau.

---

## 57.24 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `There should be exactly one item with [DropdownButton]'s value` | La valeur passée à `value:` n'est **égale** à aucun `item` | Redéfinir `==` et `hashCode` sur `Unite`, et construire la liste des unités dans `initState`, pas dans `build` |
| Le menu déroulant propose des litres dans l'onglet Longueur | Une préférence enregistrée pointe vers une unité d'une autre catégorie | Valider l'identifiant relu avec `_uniteValide` : unité existante **et** de la bonne catégorie |
| `A TextEditingController was used after being disposed` | Le contrôleur est utilisé après `dispose`, souvent depuis un `Timer` ou un `Future` | Toujours détruire dans `dispose`, et vérifier `mounted` avant tout `setState` différé |
| Fuite mémoire signalée en fin de session | `dispose()` oublié sur le `TextEditingController` ou le `TabController` | Écrire `dispose` immédiatement après avoir créé l'objet |
| `RenderFlex overflowed by N pixels` à l'ouverture du clavier | La page est une `Column` non défilante | Utiliser `ListView` ou `SingleChildScrollView` comme corps de page (chapitre 46) |
| Le nom long d'une unité déborde du menu | `isExpanded` absent sur `DropdownButton` | Ajouter `isExpanded: true` et `overflow: TextOverflow.ellipsis` sur le `Text` |
| L'application refuse `3,14` mais accepte `3.14` | `double.tryParse` ne connaît pas la virgule | Normaliser la saisie avant : `replaceAll(',', '.')` |
| Un copier-coller du résultat n'est pas relu | Le résultat contient une espace **insécable** de groupement | Retirer les espaces insécables dans `lireNombre`, pas seulement `' '` |
| Impossible de taper `-40` sur iPhone | `signed: true` manquant dans `TextInputType.numberWithOptions` | Ajouter `signed: true` |
| Impossible de taper `1,5` : la virgule est refusée | Un `inputFormatter` valide la chaîne entière au lieu des caractères | Utiliser une **classe de caractères** : `RegExp(r'[0-9.,\-eE]')` |
| Le résultat disparaît quand on tape une virgule | On remet `_resultat` à `null` dès que `tryParse` échoue | Conserver le dernier résultat valide tant que l'utilisateur tape |
| `Unhandled exception: FormatException` au démarrage | `double.parse` utilisé au lieu de `double.tryParse` | Ne jamais utiliser `parse` sur une saisie utilisateur (chapitre 13) |
| `100 °F` donne `55,56 °C` | La température a été traitée comme un facteur | Une échelle affine a besoin de deux fonctions, pas d'un nombre (étape 4) |
| `1 °C = 33,8 °F` semble faux | Confusion entre une **température** et un **écart** de température | Les deux sont justes : `1 °C` vaut `33,8 °F`, mais un écart de `1 °C` vaut `1,8 °F` |
| `0 °C` vers Rankine donne `491,66999999999996` | Arrondi normal des `double` | Ne jamais afficher un `double` brut : passer par `NumberFormat` |
| Un test échoue à `expect(resultat, 37.7778)` | Égalité stricte entre `double` | Utiliser `closeTo(valeur, tolerance)` |
| `Binding has not yet been initialized` | Un plugin est appelé avant `runApp` | Appeler `WidgetsFlutterBinding.ensureInitialized()` en première ligne de `main` |
| Après le passage à la version 2, tout le monde atterrit sur le mauvais onglet | On a enregistré `index` au lieu d'un identifiant textuel | Persister `identifiant`, jamais `index` (étape 2) |
| La saisie est perdue quand on revient sur un onglet | `TabBarView` détruit les pages éloignées | Ajouter `AutomaticKeepAliveClientMixin` **et** appeler `super.build(context)` |
| L'onglet est enregistré deux fois par changement | Le listener du `TabController` est appelé avant et après l'animation | Ignorer l'appel quand `indexIsChanging` vaut `true` |
| Le disque « 1 To » n'affiche que « 931 Go » | Confusion entre To (10^12 octets) et Tio (2^40 octets) | Ce sont deux unités différentes : proposer les deux dans le catalogue |
| L'inversion répétée fait dériver la valeur | Le résultat **arrondi** est réinjecté dans le champ | Ne pas modifier le champ à l'inversion, ou réinjecter la valeur non arrondie |

---

## 57.25 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Unité pivot | `n` unités et une référence remplacent `n × (n-1)` conversions |
| Facteur | La valeur d'**une** unité exprimée dans le pivot |
| Échelle affine | `y = a·x + b` : la température n'a pas de zéro absolu commun |
| Généralisation | Quand un `if` menace de se propager, remontez d'un cran : stockez des **fonctions**, pas des nombres |
| `enum` enrichi | Un constructeur `const`, des champs `final`, des méthodes statiques (chapitre 11) |
| Identifiant stable | On persiste une chaîne, jamais un `index` d'énumération |
| Fonction pure | Même entrée, même sortie, aucun effet de bord : testable en une ligne |
| Cas neutre | `depuis == vers` traité à part garantit l'exactitude, pas seulement la vitesse |
| Séparation des couches | Huit fichiers sur quinze n'importent pas Flutter, et concentrent toute la difficulté |
| `TextEditingController` | Créé dans `initState`, détruit dans `dispose`, sans exception |
| Trois défenses de saisie | `keyboardType` (confort), `inputFormatters` (caractères), `tryParse` (vérité) |
| `inputFormatters` | Filtre des **caractères** ; utilisez une classe de caractères, pas une expression complète |
| `double.tryParse` | Renvoie `null` au lieu de lever : la seule façon correcte de lire une saisie |
| `int` ou `double` | Grandeur dénombrable → `int` ; grandeur mesurable → `double` |
| Saisie transitoire | `'1,'` n'est pas une erreur : c'est un utilisateur en train de taper |
| `DropdownButton<T>` | Typez-le sur votre modèle, jamais sur `String` |
| Égalité et `DropdownButton` | Sans `==` cohérent, l'assertion « exactly one item » vous attend |
| Filtrage préventif | Ne proposer que le valide vaut mieux que refuser après coup |
| Calcul et `build` | `build` s'exécute des dizaines de fois : n'y mettez ni filtrage ni calcul coûteux |
| `NumberFormat` | Séparateurs et virgule décimale selon la langue, sans initialisation préalable |
| Décimales adaptatives | Le nombre de décimales dépend de l'ordre de grandeur du résultat |
| Nombres extrêmes | Hors de `[1e-6 ; 1e15[`, on bascule en notation scientifique |
| `TabController` explicite | Nécessaire dès qu'on veut **écouter** les changements d'onglet |
| `indexIsChanging` | Filtre le doublon d'appel pendant l'animation d'onglet |
| `AutomaticKeepAliveClientMixin` | Conserve l'état d'un onglet, à condition d'appeler `super.build` |
| `SharedPreferencesWithCache` | Lectures synchrones après un chargement unique : parfait pour `initState` |
| `allowList` | Déclarer ses clés accélère le démarrage et protège des collisions |
| Données persistées plates | On sérialise des nombres et des chaînes, jamais des objets porteurs de fonctions |
| Données corrompues | Une préférence illisible s'ignore ; elle ne fait pas planter le démarrage |
| `closeTo` | On ne compare jamais deux `double` avec `==` dans un test |
| Test de propriété | « Un aller-retour redonne la valeur » couvre 300 conversions en 10 lignes |

---

## 57.26 — Extensions : dix défis

### Défi 1 — Ajouter la catégorie Surface (facile)

Ajoutez une septième catégorie avec le mètre carré comme pivot : mm², cm², m², km², hectare (10 000 m²), acre (4046,8564224 m² exactement), pied carré (0,09290304 m²).

**Indication de solution.** Aucun code d'interface à écrire : ajoutez une valeur à l'`enum Categorie`, une liste dans `catalogue.dart`, une entrée dans la `Map`. Les onglets, les menus et les tests s'adaptent seuls — c'est la preuve que le modèle est bon. Vérifiez que l'acre vaut bien 4840 yards carrés : 4840 × 0,9144² = 4840 × 0,83612736 = 4046,8564224.

### Défi 2 — Rechercher une unité (facile)

Six catégories font beaucoup de défilement. Ajoutez un champ de recherche qui filtre les unités par nom ou par symbole.

**Indication de solution.** Un `TextField` supplémentaire, un `where` sur `toutesLesUnites` avec `nom.toLowerCase().contains(requete)`. Remplacez le `DropdownButton` par un `showModalBottomSheet` contenant une `ListView.builder` filtrée : un menu déroulant natif ne se prête pas à la recherche.

### Défi 3 — Inversion avec échange des valeurs (moyen)

Implémentez le comportement B de la section 57.17.4 : le bouton d'inversion fait remonter le résultat dans le champ de saisie.

**Indication de solution.** Le piège est la perte de précision. N'écrivez **pas** le texte affiché dans le champ ; écrivez le `double` non arrondi via `_resultat!.toString()`, puis reformatez. Testez en appuyant vingt fois de suite : la valeur doit revenir exactement à son point de départ, aux erreurs de `double` près.

### Défi 4 — Les unités britanniques (moyen)

Le gallon impérial vaut exactement 4,54609 L, la pinte impériale 0,56826125 L, l'once liquide impériale 0,0284130625 L. Ajoutez-les au catalogue en les distinguant clairement des unités américaines.

**Indication de solution.** Utilisez le champ `nom` pour lever l'ambiguïté : « Gallon (UK) » et « Gallon (US) ». Ajoutez un test qui vérifie qu'un gallon impérial vaut environ 1,20095 gallon américain (4,54609 / 3,785411784 = 1,2009499255).

### Défi 5 — L'écart de température (moyen)

Ajoutez un interrupteur « convertir un écart » dans l'onglet Température. En mode écart, `1 °C` doit donner `1,8 °F`, et `10 K` doit donner `18 °F`.

**Indication de solution.** Un écart se convertit avec la **pente seule**, sans l'origine. Une façon élégante : calculer `convertir(x) - convertir(0)`, ce qui annule mécaniquement le décalage, quelle que soit l'échelle. Vérifiez que la méthode marche aussi pour Delisle, dont la pente est négative.

### Défi 6 — Un constructeur `Unite.affine` réellement utilisé (moyen)

Réécrivez les unités de température avec `Unite.affine` plutôt qu'avec des fonctions explicites.

**Indication de solution.** Pour le Fahrenheit avec le Celsius en pivot : `(f - 32) × 5/9 = f × 5/9 - 160/9`, donc `pente = 5/9` et `origine = -160/9`. Pour Delisle : `pente = -2/3`, `origine = 100`. Écrivez d'abord les tests, puis la réécriture : ils doivent rester verts sans être modifiés.

### Défi 7 — Les favoris (moyen)

Permettez d'épingler un couple d'unités, et affichez les couples épinglés sous forme de puces cliquables en haut de la page.

**Indication de solution.** Une liste de chaînes `'kilometre>mile'` dans les préférences, un `Wrap` de `FilterChip` pour l'affichage. Le `Wrap` du chapitre 46 gère seul le passage à la ligne quand les puces sont nombreuses.

### Défi 8 — Le tableau de toutes les conversions (difficile)

Sous le résultat, affichez la valeur saisie convertie **simultanément** dans toutes les unités de la catégorie.

**Indication de solution.** Une `Column` construite avec `_unites.map(...)`, appelant `convertir` pour chacune. C'est le moment de mesurer : neuf conversions par frappe restent gratuites, mais profitez-en pour ouvrir le *DevTools Performance* et le vérifier au lieu de le supposer.

### Défi 9 — Les unités composées (difficile)

Ajoutez une catégorie Vitesse : m/s, km/h, mi/h, nœud (1 nœud = 1 mille marin par heure). Puis réfléchissez : peut-on la **dériver** des catégories Longueur et Temps plutôt que de la saisir à la main ?

**Indication de solution.** La version simple est un catalogue de plus, avec le m/s comme pivot : 1 km/h = 1/3,6 m/s, 1 mi/h = 1609,344/3600 = 0,44704 m/s exactement, 1 nœud = 1852/3600 = 0,5144444… m/s. La version dérivée demande de modéliser une unité comme un couple `(numérateur, dénominateur)` et de composer les facteurs : c'est un vrai exercice de conception, et le début d'une bibliothèque de dimensions physiques.

### Défi 10 — Des taux de change réels (difficile)

Remplacez les devises fixes par des taux téléchargés depuis une API publique, avec cache hors-ligne.

**Indication de solution.** Vous avez déjà tout ce qu'il faut : `http` et `FutureBuilder` du chapitre 53, le cache du chapitre 54. La difficulté n'est pas technique mais architecturale : une `Unite` est aujourd'hui immuable et construite au démarrage, alors qu'un taux change. Solution propre : ne pas modifier `Unite`, mais reconstruire la liste des devises à chaque rafraîchissement, et laisser la fonction `convertir` telle quelle. Si vous devez modifier `convertir`, c'est que votre conception dérive.

---

## 57.27 — Résumé du chapitre

Vous avez construit une application complète de quinze fichiers en dix-neuf étapes, chacune se terminant par un état exécutable.

La leçon centrale n'est pas dans les widgets. Elle tient en une image :

```text
       ┌──────────────────────────────────────────┐
       │  ecrans/ + widgets/                      │
       │  Flutter, setState, build                │
       │  -> facile a ecrire, difficile a tester  │
       └────────────────┬─────────────────────────┘
                        │  appelle
                        v
       ┌──────────────────────────────────────────┐
       │  modeles/ + donnees/ + logique/           │
       │  Dart pur, fonctions pures                │
       │  -> difficile a ecrire, facile a tester   │
       └──────────────────────────────────────────┘
```

Tout ce qui est difficile est en bas, et tout ce qui est en bas est testé. Si vous ne gardez qu'une habitude de ce chapitre, gardez celle-là : **la logique métier ne connaît pas Flutter**.

---

## Et maintenant ?

Le convertisseur manipule des données **calculées** : elles n'existent que le temps d'un affichage, et la seule chose que l'on conserve est un choix de menu.

Le projet suivant renverse la situation. Une liste de tâches manipule des données **créées par l'utilisateur** : elles doivent naître, vivre, être modifiées, glissées, supprimées, et surtout survivre à la fermeture de l'application. Vous y retrouverez `shared_preferences`, mais poussé bien plus loin, avec les listes dynamiques du chapitre 48 et le `Dismissible` qui supprime d'un glissement de doigt.

Rendez-vous au chapitre 58 : [58-PARTIE-1C—PROJET-4-TODO-LIST.md](./58-PARTIE-1C—PROJET-4-TODO-LIST.md)
