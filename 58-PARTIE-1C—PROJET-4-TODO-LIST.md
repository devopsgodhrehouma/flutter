# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 58 — PROJET 4 : LA LISTE DE TÂCHES

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), PARTIE 1B (chapitres 43 à 54), et les projets 55, 56, 57
> **Ce que vous saurez faire à la fin :** construire de bout en bout une application de gestion de tâches complète — modèle de données sérialisable, dépôt interchangeable, liste filtrable, triable et cherchable, formulaire validé, suppression annulable, état centralisé avec `provider`, persistance sur le disque et tests automatisés.

---

## 58.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- modéliser une tâche avec une classe Dart munie d'un `enum` de priorité, d'une échéance `DateTime` et d'étiquettes ;
- écrire `fromJson` et `toJson` sur ce modèle (rappel du chapitre 17) ;
- écrire une méthode `copyWith` correcte, y compris pour un champ nullable ;
- définir une interface de dépôt (`repository`) et en fournir trois implémentations ;
- afficher une collection avec `ListView.builder` (rappel du chapitre 48) ;
- soigner l'état vide d'un écran ;
- cocher une tâche et barrer son titre avec `TextDecoration.lineThrough` ;
- ajouter une tâche par une boîte de dialogue `showDialog` (rappel du chapitre 49) ;
- construire un formulaire complet avec `Form`, `TextFormField` et `validator` ;
- réutiliser le même formulaire pour la création et la modification ;
- supprimer par glissement avec `Dismissible` et comprendre pourquoi la `Key` est obligatoire ;
- proposer une annulation de suppression avec un `SnackBar` et son `SnackBarAction` ;
- filtrer une liste avec `where` (rappel du chapitre 14) ;
- trier par échéance ou par priorité, en gérant proprement les valeurs nulles ;
- brancher une barre de recherche sur la liste ;
- afficher des compteurs et une `LinearProgressIndicator` d'avancement ;
- choisir une date avec `showDatePicker` ;
- formater une date en français avec le paquet `intl` ;
- mettre en évidence les tâches en retard ;
- extraire tout l'état dans un `ChangeNotifier` exposé par `provider` (rappel du chapitre 52) ;
- persister les tâches avec `shared_preferences` (rappel du chapitre 54) ;
- écrire la même persistance avec `sqflite` et comparer les deux ;
- ajouter un mode sombre commutable et persisté ;
- écrire des tests unitaires sur la logique de filtrage et de tri.

---

## 58.0.1 — Aperçu du résultat final

Voici l'application terminée. Écran principal, liste peuplée :

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                          [T]  ...  │
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │ Rechercher une tâche...                  │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  [ Toutes ]  [ Actives ]  [ Terminées ]        │
│                                                │
│  3 sur 7 terminées                             │
│  ████████████░░░░░░░░░░░░░░░░░░░  43 %         │
├────────────────────────────────────────────────┤
│ ▌☐  Corriger le bug de collision               │
│ ▌   URGENTE · 12 août 2026 · EN RETARD         │
│ ▌   #jeu  #bug                                 │
├────────────────────────────────────────────────┤
│ ▌☐  Dessiner le sprite du boss                 │
│ ▌   HAUTE · vendredi 21 août 2026              │
│ ▌   #graphisme                                 │
├────────────────────────────────────────────────┤
│ ▌☐  Écrire la musique du niveau 3              │
│ ▌   NORMALE · pas d'échéance                   │
├────────────────────────────────────────────────┤
│ ▌☑  Équilibrer les dégâts de l'épée            │
│ ▌   ̶N̶O̶R̶M̶A̶L̶E̶ · terminée                        │
├────────────────────────────────────────────────┤
│                                        ┌─────┐ │
│                                        │  +  │ │
│                                        └─────┘ │
└────────────────────────────────────────────────┘
```

Le glissement d'une tuile vers la gauche révèle le fond de suppression :

```text
├────────────────────────────────────────────────┤
│          ← ← ←  Dessiner le sprite  ░░░░░░  X  │
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │ Tâche supprimée            ANNULER       │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

L'écran de formulaire, utilisé aussi bien pour créer que pour modifier :

```text
┌────────────────────────────────────────────────┐
│  ←   Nouvelle tâche                            │
├────────────────────────────────────────────────┤
│  Titre *                                       │
│  ┌──────────────────────────────────────────┐  │
│  │ Dessiner le sprite du boss               │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Description                                   │
│  ┌──────────────────────────────────────────┐  │
│  │ 4 directions, 8 images par cycle.        │  │
│  │                                          │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Priorité                                      │
│  ( ) Basse  ( ) Normale  (•) Haute  ( ) Urgente│
│                                                │
│  Échéance                                      │
│  ┌──────────────────────────────────────────┐  │
│  │ [cal]  vendredi 21 août 2026         ✕   │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Étiquettes (séparées par des virgules)        │
│  ┌──────────────────────────────────────────┐  │
│  │ graphisme, boss                          │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│         ┌──────────────────────────────┐       │
│         │        ENREGISTRER           │       │
│         └──────────────────────────────┘       │
└────────────────────────────────────────────────┘
```

Et l'état vide, qu'un débutant oublie presque toujours :

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                          [T]  ...  │
├────────────────────────────────────────────────┤
│                                                │
│                                                │
│                    ☑                           │
│                                                │
│           Aucune tâche pour l'instant          │
│                                                │
│      Appuyez sur + pour créer votre première   │
│                     tâche.                     │
│                                                │
│                                        ┌─────┐ │
│                                        │  +  │ │
│                                        └─────┘ │
└────────────────────────────────────────────────┘
```

---

## 58.0.2 — Cahier des charges

Un projet commence toujours par un cahier des charges écrit. Sans lui, vous coderez au hasard et vous ne saurez jamais quand vous avez fini.

### Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | Une tâche possède un identifiant unique, un titre, une description, un état « faite », une priorité, une échéance facultative et des étiquettes. | Le modèle compile et se sérialise. |
| O2 | L'application affiche la liste des tâches sous forme défilante. | 200 tâches défilent sans saccade. |
| O3 | Quand la liste est vide, un message d'accueil s'affiche. | Supprimer toutes les tâches. |
| O4 | On peut cocher et décocher une tâche ; son titre est barré quand elle est faite. | Clic sur la case. |
| O5 | On peut créer une tâche via un formulaire validé. | Un titre vide est refusé. |
| O6 | On peut modifier une tâche existante avec le même formulaire. | Les champs sont pré-remplis. |
| O7 | On peut supprimer une tâche par glissement. | La tuile disparaît. |
| O8 | Toute suppression est annulable pendant quelques secondes. | Le bouton ANNULER restaure la tâche à sa position. |
| O9 | On peut filtrer : toutes, actives, terminées. | Les compteurs correspondent. |
| O10 | On peut trier par échéance, par priorité, par date de création ou par ordre alphabétique. | L'ordre change à l'écran. |
| O11 | On peut chercher dans le titre, la description et les étiquettes. | La recherche est insensible à la casse. |
| O12 | Un compteur et une barre de progression indiquent l'avancement. | 3 sur 7 → 43 %. |
| O13 | Les tâches en retard sont mises en évidence. | Une échéance passée non cochée devient rouge. |
| O14 | L'état est centralisé dans un `ChangeNotifier` fourni par `provider`. | Plus aucun `setState` de données. |
| O15 | Les tâches survivent à la fermeture de l'application. | Tuer l'application, la relancer. |
| O16 | La logique de filtrage et de tri est couverte par des tests. | `flutter test` passe. |

### Fonctionnalités bonus

| # | Exigence |
| --- | --- |
| B1 | Un mode sombre commutable depuis la barre d'application et persisté. |
| B2 | Une variante du dépôt utilisant `sqflite` au lieu de `shared_preferences`. |
| B3 | Un bouton « supprimer les tâches terminées » avec confirmation. |
| B4 | Un affichage des étiquettes sous forme de `Chip`. |

---

## 58.0.3 — Notions mobilisées

Ce projet ne contient aucune notion nouvelle. Il assemble ce que vous avez déjà appris. Si une ligne du tableau vous surprend, relisez le chapitre indiqué avant de commencer.

| Notion | Chapitre | Usage exact dans ce projet |
| --- | --- | --- |
| `List`, `Map`, boucle `for-in` | 06 | La collection de tâches. |
| Classes, propriétés, méthodes | 08 | La classe `Tache`. |
| Constructeurs nommés, `required` | 09 | `Tache.fromJson`, `copyWith`. |
| `enum` enrichi (avec champs) | 11 | `Priorite`, `Filtre`, `Tri`. |
| Null safety, `?`, `??`, `!` | 12 | L'échéance facultative. |
| Exceptions, `try`/`catch` | 13 | Lecture d'un JSON corrompu. |
| `map`, `where`, `sort`, `fold`, `any` | 14 | Filtrage, tri, compteurs. |
| `Future`, `async`, `await` | 15 | Le dépôt et la persistance. |
| `pubspec.yaml`, paquets | 16 | `provider`, `intl`, `uuid`, `sqflite`. |
| `jsonEncode`, `jsonDecode`, `fromJson`/`toJson` | 17 | La sérialisation des tâches. |
| `MaterialApp`, `Scaffold`, `AppBar` | 44 | La structure de l'écran. |
| `StatefulWidget`, `setState`, `dispose` | 45 | Les premières étapes, les contrôleurs. |
| `Row`, `Column`, `Expanded`, `Padding` | 46 | La mise en page des tuiles. |
| `Text`, `TextStyle`, `Icon` | 47 | Le titre barré, les icônes de priorité. |
| `ListView.builder`, `Dismissible`, `Card` | 48 | La liste et la suppression par glissement. |
| `Form`, `TextFormField`, `validator` | 49 | Le formulaire de tâche. |
| `Navigator.push`, retour de données | 50 | L'écran de formulaire. |
| `ThemeData`, Material 3, mode sombre | 51 | Le thème clair et sombre. |
| `ChangeNotifier`, `provider` | 52 | L'état centralisé. |
| `shared_preferences`, `sqflite` | 54 | La persistance. |

---

## 58.0.4 — Arborescence du projet

Voici l'arborescence finale, celle que vous obtiendrez à la fin du chapitre. Elle est donnée dès maintenant pour que vous sachiez où va chaque fichier ; nous la construirons progressivement.

```text
liste_taches/
├── pubspec.yaml
├── lib/
│   ├── main.dart                        point d'entrée, thème, provider
│   ├── modeles/
│   │   ├── priorite.dart                enum Priorite
│   │   └── tache.dart                   classe Tache + JSON + copyWith
│   ├── logique/
│   │   ├── criteres.dart                enum Filtre, enum Tri
│   │   └── filtrage.dart                fonctions pures, testables
│   ├── donnees/
│   │   ├── depot_taches.dart            interface abstraite
│   │   ├── depot_memoire.dart           implémentation volatile
│   │   ├── depot_prefs.dart             implémentation shared_preferences
│   │   └── depot_sqflite.dart           implémentation sqflite (variante)
│   ├── etat/
│   │   ├── etat_taches.dart             ChangeNotifier principal
│   │   └── etat_theme.dart              ChangeNotifier du thème
│   ├── ecrans/
│   │   ├── ecran_taches.dart            écran principal
│   │   └── ecran_formulaire.dart        création et modification
│   ├── widgets/
│   │   ├── tuile_tache.dart             une ligne de la liste
│   │   ├── barre_filtres.dart           les trois segments
│   │   ├── barre_avancement.dart        compteur + progression
│   │   └── etat_vide.dart               écran vide
│   └── utilitaires/
│       ├── dates.dart                   formatage intl
│       └── couleurs.dart                couleur par priorité
└── test/
    └── filtrage_test.dart               tests de la logique
```

**Pourquoi cette découpe ?** Chaque dossier répond à une seule question :

```text
modeles/      QUOI ?        les données, sans aucune dépendance à Flutter
logique/      COMMENT ?     les règles métier, pures, testables sans écran
donnees/      OÙ ?          la lecture et l'écriture (mémoire, disque, base)
etat/         QUAND ?       ce qui change et qui prévient l'interface
ecrans/       À QUOI ÇA     les pages complètes
widgets/      RESSEMBLE ?   les morceaux réutilisables
utilitaires/  AVEC QUOI ?   les petites fonctions de service
```

Un fichier de `modeles/` ou de `logique/` ne doit **jamais** importer `package:flutter/material.dart`. C'est la règle qui rend les tests du 58.24 possibles.

---

## 58.1 — Créer le projet et poser le squelette

### Créer le projet

```text
flutter create liste_taches
cd liste_taches
```

### Ajouter les dépendances

Ajoutez les paquets un par un avec `flutter pub add`. Cette commande écrit toujours la version la plus récente compatible ; c'est préférable à recopier un numéro qui vieillira.

```text
flutter pub add provider
flutter pub add intl
flutter pub add uuid
flutter pub add shared_preferences
flutter pub add sqflite
flutter pub add path
flutter pub add flutter_localizations --sdk=flutter
```

Le `pubspec.yaml` obtenu ressemble à ceci. Les numéros de version sont ceux constatés à la rédaction ; les vôtres peuvent être supérieurs, c'est normal.

**`liste_taches/pubspec.yaml`**

```yaml
name: liste_taches
description: "Une liste de tâches complète et persistante."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # Gestion d'état (chapitre 52)
  provider: ^6.1.5

  # Formatage des dates en français (chapitre 58.18)
  intl: ^0.20.3

  # Identifiants uniques des tâches
  uuid: ^4.6.0

  # Persistance simple clé/valeur (chapitre 54)
  shared_preferences: ^2.5.5

  # Persistance relationnelle, variante du 58.22
  sqflite: ^2.4.3
  path: ^1.9.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

> **Remarque.** Aucun fichier image n'est nécessaire dans ce projet. Toute l'interface repose sur des icônes Material et des couleurs de thème. Il n'y a donc pas de section `assets:`.

### Le squelette

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationTaches());
}

/// Racine de l'application.
///
/// Pour l'instant elle ne fait qu'installer un thème Material 3
/// et afficher un écran vide. Tout le reste viendra s'y greffer.
class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mes tâches',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
      ),
      home: const EcranTaches(),
    );
  }
}

/// Écran principal, provisoirement vide.
class EcranTaches extends StatelessWidget {
  const EcranTaches({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mes tâches')),
      body: const Center(child: Text('Rien pour l\'instant')),
    );
  }
}
```

**État exécutable.** `flutter run` affiche une barre d'application indigo et un texte centré. Le projet compile, les dépendances sont résolues. C'est le point de départ.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                                    │
├────────────────────────────────────────────────┤
│                                                │
│              Rien pour l'instant               │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 58.2 — L'énumération des priorités

Une tâche a une priorité. Un débutant écrirait `String priorite = 'haute'`. C'est une erreur : rien n'empêche alors d'écrire `'Haute'`, `'HAUTE'`, `'hute'`. Le compilateur ne dira rien et le tri se cassera silencieusement.

Le chapitre 11 vous a donné l'outil exact : l'`enum` enrichi, c'est-à-dire un `enum` qui porte ses propres champs et son propre constructeur.

**`lib/modeles/priorite.dart`**

```dart
/// Niveau d'urgence d'une tâche.
///
/// C'est un `enum` enrichi (chapitre 11) : chaque valeur transporte
/// un libellé affichable et un poids numérique servant au tri.
///
/// L'ordre de déclaration va du moins urgent au plus urgent, ce qui
/// permet d'utiliser directement `index` si besoin, mais nous
/// préférons `poids` car il est explicite.
enum Priorite {
  basse('Basse', 0),
  normale('Normale', 1),
  haute('Haute', 2),
  urgente('Urgente', 3);

  const Priorite(this.libelle, this.poids);

  /// Texte affiché à l'utilisateur.
  final String libelle;

  /// Poids utilisé pour trier : plus il est grand, plus c'est urgent.
  final int poids;

  /// Reconstruit une priorité à partir du nom stocké en JSON.
  ///
  /// Si le nom est inconnu (fichier corrompu, ancienne version de
  /// l'application), on ne lève pas d'exception : on retombe sur
  /// `normale`. Une donnée abîmée ne doit jamais empêcher le
  /// démarrage de l'application.
  static Priorite depuisNom(String? nom) {
    for (final Priorite p in Priorite.values) {
      if (p.name == nom) {
        return p;
      }
    }
    return Priorite.normale;
  }
}
```

### Vérification en console

Ce fichier n'importe pas Flutter. Vous pouvez donc le tester dans DartPad ou avec `dart run`.

```dart
// Collez le contenu de priorite.dart au-dessus de ce main pour l'essayer.
void main() {
  for (final Priorite p in Priorite.values) {
    print('${p.name} -> ${p.libelle} (poids ${p.poids})');
  }
  print(Priorite.depuisNom('haute'));
  print(Priorite.depuisNom('inconnue'));
}
```

**Résultat :**

```text
basse -> Basse (poids 0)
normale -> Normale (poids 1)
haute -> Haute (poids 2)
urgente -> Urgente (poids 3)
Priorite.haute
Priorite.normale
```

> **Remarque.** `p.name` est fourni gratuitement par Dart sur tout `enum` : il renvoie le nom de la valeur sous forme de `String`. C'est exactement ce qu'il faut écrire dans le JSON. N'écrivez jamais `p.index` dans un fichier persisté : le jour où vous insérez une priorité au milieu de la liste, tous les fichiers déjà enregistrés désignent la mauvaise valeur.

**État exécutable.** Le fichier compile seul. L'application ne change pas encore d'apparence.

---

## 58.3 — Le modèle `Tache`

Passons à la classe centrale du projet. Elle applique les chapitres 08 (classes), 09 (constructeurs) et 12 (null safety).

Décisions de modélisation, à comprendre avant de lire le code :

```text
id           String      obligatoire, unique, jamais modifié
titre        String      obligatoire, non vide
description  String      facultatif -> valeur par défaut '' (pas null)
faite        bool        par défaut false
priorite     Priorite    par défaut normale
echeance     DateTime?   VRAIMENT facultatif -> nullable
etiquettes   List<String> par défaut vide (pas null)
creeLe       DateTime    rempli automatiquement si absent
```

Règle générale : **on n'utilise `null` que quand l'absence a un sens métier**. Une description absente et une description vide, c'est la même chose pour l'utilisateur : on prend `''`. En revanche « pas d'échéance » et « échéance au 1er janvier 1970 » ne sont pas la même chose : là, `null` s'impose.

**`lib/modeles/tache.dart`** (première version, sans JSON — le JSON arrive au 58.4)

```dart
import 'priorite.dart';

/// Une tâche à réaliser.
///
/// Cette classe ne connaît NI Flutter, NI la base de données.
/// Elle décrit uniquement une donnée. C'est ce qui la rend
/// testable sans lancer d'application (voir 58.24).
class Tache {
  Tache({
    required this.id,
    required this.titre,
    this.description = '',
    this.faite = false,
    this.priorite = Priorite.normale,
    this.echeance,
    List<String> etiquettes = const <String>[],
    DateTime? creeLe,
  })  : etiquettes = List<String>.unmodifiable(etiquettes),
        creeLe = creeLe ?? DateTime.now();

  /// Identifiant unique et immuable.
  final String id;

  /// Titre affiché dans la liste. Ne doit jamais être vide.
  final String titre;

  /// Détail facultatif. Chaîne vide si absent, jamais `null`.
  final String description;

  /// `true` quand la tâche est terminée.
  final bool faite;

  /// Niveau d'urgence (chapitre 11).
  final Priorite priorite;

  /// Date limite. `null` signifie « pas d'échéance ».
  final DateTime? echeance;

  /// Mots-clés libres, sans le caractère `#`.
  final List<String> etiquettes;

  /// Date de création, utilisée pour le tri « plus récentes d'abord ».
  final DateTime creeLe;

  /// Renvoie une COPIE modifiée de la tâche.
  ///
  /// Les champs sont `final` : on ne modifie jamais une tâche,
  /// on en fabrique une nouvelle. C'est la même discipline que
  /// pour les widgets `const` du chapitre 44, et elle évite une
  /// classe entière de bugs (deux écrans qui modifient le même
  /// objet sans se prévenir).
  ///
  /// Le cas de `echeance` est délicat : comment distinguer
  /// « ne change pas l'échéance » de « efface l'échéance » ?
  /// Les deux s'écriraient `echeance: null`. On ajoute donc un
  /// drapeau explicite `effacerEcheance`.
  Tache copyWith({
    String? titre,
    String? description,
    bool? faite,
    Priorite? priorite,
    DateTime? echeance,
    bool effacerEcheance = false,
    List<String>? etiquettes,
  }) {
    return Tache(
      id: id,
      titre: titre ?? this.titre,
      description: description ?? this.description,
      faite: faite ?? this.faite,
      priorite: priorite ?? this.priorite,
      echeance: effacerEcheance ? null : (echeance ?? this.echeance),
      etiquettes: etiquettes ?? this.etiquettes,
      creeLe: creeLe,
    );
  }

  /// `true` si l'échéance est dépassée et que la tâche n'est pas faite.
  ///
  /// On compare à la FIN du jour d'échéance : une tâche due
  /// aujourd'hui n'est pas en retard avant minuit.
  bool get enRetard {
    final DateTime? limite = echeance;
    if (limite == null || faite) {
      return false;
    }
    final DateTime finDuJour = DateTime(
      limite.year,
      limite.month,
      limite.day,
      23,
      59,
      59,
    );
    return finDuJour.isBefore(DateTime.now());
  }

  /// `true` si l'échéance tombe aujourd'hui.
  bool get estPourAujourdHui {
    final DateTime? limite = echeance;
    if (limite == null) {
      return false;
    }
    final DateTime maintenant = DateTime.now();
    return limite.year == maintenant.year &&
        limite.month == maintenant.month &&
        limite.day == maintenant.day;
  }

  @override
  String toString() {
    return 'Tache($id, "$titre", faite: $faite, ${priorite.name})';
  }
}
```

### Deux points à ne pas manquer

**`List<String>.unmodifiable`.** Le constructeur reçoit une liste et en fait une copie non modifiable. Sans cela, l'appelant garderait une référence sur la liste interne et pourrait écrire `maListe.add('triche')` après coup, en contournant `copyWith`. Ce genre de fuite est la cause la plus fréquente des « bugs impossibles ».

**Le champ `creeLe` avec valeur par défaut calculée.** On ne peut pas écrire `DateTime creeLe = DateTime.now()` dans la liste des paramètres : une valeur par défaut doit être une constante de compilation. La solution vue au chapitre 09 est la liste d'initialisation : `creeLe = creeLe ?? DateTime.now()`.

**État exécutable.** Le modèle compile. Testons-le tout de suite en console :

```dart
void main() {
  final Tache t = Tache(
    id: 'a1',
    titre: 'Dessiner le sprite du boss',
    priorite: Priorite.haute,
    echeance: DateTime(2020, 1, 1),
    etiquettes: <String>['graphisme'],
  );

  print(t);
  print('En retard : ${t.enRetard}');

  final Tache faite = t.copyWith(faite: true);
  print(faite);
  print('En retard : ${faite.enRetard}');

  final Tache sansEcheance = t.copyWith(effacerEcheance: true);
  print('Échéance effacée : ${sansEcheance.echeance}');
}
```

**Résultat :**

```text
Tache(a1, "Dessiner le sprite du boss", faite: false, haute)
En retard : true
Tache(a1, "Dessiner le sprite du boss", faite: true, haute)
En retard : false
Échéance effacée : null
```

---

## 58.4 — La sérialisation JSON (rappel chapitre 17)

Pour enregistrer les tâches sur le disque au 58.21, il faut les transformer en texte. Le chapitre 17 a établi la méthode : une paire `toJson` / `fromJson`.

Rappel du principe :

```text
  Tache (objet Dart)
        │  toJson()
        ▼
  Map<String, dynamic>
        │  jsonEncode()
        ▼
  String  '{"id":"a1","titre":"..."}'
        │  écriture disque
        ▼
  ─────────── stockage ───────────
        │  lecture disque
        ▼
  String
        │  jsonDecode()
        ▼
  Map<String, dynamic>
        │  Tache.fromJson()
        ▼
  Tache (objet Dart)
```

Les types autorisés dans un JSON sont `String`, `num`, `bool`, `List`, `Map` et `null`. Ni `DateTime`, ni `enum` n'en font partie. Il faut donc les convertir :

```text
DateTime   ->  String au format ISO 8601   (toIso8601String / DateTime.parse)
Priorite   ->  String                       (p.name / Priorite.depuisNom)
```

Ajoutez ces deux méthodes à la classe `Tache`, juste après `copyWith`.

**`lib/modeles/tache.dart`** (fichier complet, version définitive du modèle)

```dart
import 'priorite.dart';

/// Une tâche à réaliser.
class Tache {
  Tache({
    required this.id,
    required this.titre,
    this.description = '',
    this.faite = false,
    this.priorite = Priorite.normale,
    this.echeance,
    List<String> etiquettes = const <String>[],
    DateTime? creeLe,
  })  : etiquettes = List<String>.unmodifiable(etiquettes),
        creeLe = creeLe ?? DateTime.now();

  /// Reconstruit une tâche depuis une `Map` issue de `jsonDecode`.
  ///
  /// Chaque lecture est défensive : le fichier a pu être écrit par
  /// une version plus ancienne de l'application, ou avoir été
  /// modifié à la main. On ne fait confiance qu'à `id` et `titre`.
  factory Tache.fromJson(Map<String, dynamic> json) {
    final Object? brutEcheance = json['echeance'];
    final Object? brutCreeLe = json['creeLe'];
    final Object? brutEtiquettes = json['etiquettes'];

    return Tache(
      id: json['id'] as String,
      titre: json['titre'] as String,
      description: json['description'] as String? ?? '',
      faite: json['faite'] as bool? ?? false,
      priorite: Priorite.depuisNom(json['priorite'] as String?),
      echeance: brutEcheance is String ? DateTime.tryParse(brutEcheance) : null,
      etiquettes: brutEtiquettes is List
          ? brutEtiquettes.map((Object? e) => e.toString()).toList()
          : const <String>[],
      creeLe: brutCreeLe is String ? DateTime.tryParse(brutCreeLe) : null,
    );
  }

  final String id;
  final String titre;
  final String description;
  final bool faite;
  final Priorite priorite;
  final DateTime? echeance;
  final List<String> etiquettes;
  final DateTime creeLe;

  /// Transforme la tâche en `Map` prête pour `jsonEncode`.
  Map<String, dynamic> toJson() {
    return <String, dynamic>{
      'id': id,
      'titre': titre,
      'description': description,
      'faite': faite,
      // `.name` donne 'haute', 'urgente'... Jamais `.index`.
      'priorite': priorite.name,
      // `?.` : si l'échéance est nulle, on écrit null dans le JSON.
      'echeance': echeance?.toIso8601String(),
      // On copie dans une liste modifiable : jsonEncode
      // n'accepte pas toutes les vues non modifiables.
      'etiquettes': etiquettes.toList(),
      'creeLe': creeLe.toIso8601String(),
    };
  }

  Tache copyWith({
    String? titre,
    String? description,
    bool? faite,
    Priorite? priorite,
    DateTime? echeance,
    bool effacerEcheance = false,
    List<String>? etiquettes,
  }) {
    return Tache(
      id: id,
      titre: titre ?? this.titre,
      description: description ?? this.description,
      faite: faite ?? this.faite,
      priorite: priorite ?? this.priorite,
      echeance: effacerEcheance ? null : (echeance ?? this.echeance),
      etiquettes: etiquettes ?? this.etiquettes,
      creeLe: creeLe,
    );
  }

  bool get enRetard {
    final DateTime? limite = echeance;
    if (limite == null || faite) {
      return false;
    }
    final DateTime finDuJour = DateTime(
      limite.year,
      limite.month,
      limite.day,
      23,
      59,
      59,
    );
    return finDuJour.isBefore(DateTime.now());
  }

  bool get estPourAujourdHui {
    final DateTime? limite = echeance;
    if (limite == null) {
      return false;
    }
    final DateTime maintenant = DateTime.now();
    return limite.year == maintenant.year &&
        limite.month == maintenant.month &&
        limite.day == maintenant.day;
  }

  @override
  String toString() {
    return 'Tache($id, "$titre", faite: $faite, ${priorite.name})';
  }
}
```

### Le test d'aller-retour

Le seul test qui prouve qu'une sérialisation est correcte est l'aller-retour : objet → JSON → objet, et on compare.

```dart
import 'dart:convert';

void main() {
  final Tache origine = Tache(
    id: 'a1',
    titre: 'Corriger le bug de collision',
    description: 'Le joueur traverse le sol à grande vitesse.',
    priorite: Priorite.urgente,
    echeance: DateTime(2026, 8, 12),
    etiquettes: <String>['jeu', 'bug'],
    creeLe: DateTime(2026, 8, 1, 9, 30),
  );

  final String texte = jsonEncode(origine.toJson());
  print(texte);

  final Tache relue = Tache.fromJson(
    jsonDecode(texte) as Map<String, dynamic>,
  );
  print(relue);
  print('Échéance relue : ${relue.echeance}');
  print('Étiquettes relues : ${relue.etiquettes}');
  print('Identique : ${relue.titre == origine.titre &&
      relue.echeance == origine.echeance &&
      relue.priorite == origine.priorite}');
}
```

**Résultat :**

```text
{"id":"a1","titre":"Corriger le bug de collision","description":"Le joueur traverse le sol à grande vitesse.","faite":false,"priorite":"urgente","echeance":"2026-08-12T00:00:00.000","etiquettes":["jeu","bug"],"creeLe":"2026-08-01T09:30:00.000"}
Tache(a1, "Corriger le bug de collision", faite: false, urgente)
Échéance relue : 2026-08-12 00:00:00.000
Étiquettes relues : [jeu, bug]
Identique : true
```

> **Remarque.** `DateTime.tryParse` renvoie `null` au lieu de lever une exception quand la chaîne est invalide. Dans un `fromJson`, préférez toujours `tryParse` à `parse` : une date abîmée fera perdre une échéance, pas planter l'application.

**État exécutable.** Le modèle est complet et vérifié. L'écran n'a toujours pas changé — c'est normal, nous avons construit les fondations.

---

## 58.5 — Le dépôt en mémoire

Vous pourriez ranger les tâches dans une simple `List` à l'intérieur d'un `State`. Ce serait plus court, et ce serait une impasse : au 58.21 il faudrait tout réécrire pour passer au disque.

La bonne pratique est de définir dès maintenant **une interface**, c'est-à-dire un contrat, puis d'en fournir une implémentation triviale. Le jour où l'on change de stockage, seule l'implémentation change ; l'interface, elle, ne bouge pas.

```text
        EtatTaches  (58.20)
             │  ne connaît QUE l'interface
             ▼
     ┌──────────────────┐
     │  DepotTaches     │   abstract class
     │  chargerTout()   │
     │  ajouter()       │
     │  modifier()      │
     │  supprimer()     │
     │  remplacerTout() │
     └──────────────────┘
        ▲       ▲       ▲
        │       │       │
  DepotMemoire  │   DepotSqflite
            DepotPrefs
     (58.5)   (58.21)    (58.22)
```

**`lib/donnees/depot_taches.dart`**

```dart
import '../modeles/tache.dart';

/// Contrat commun à tous les moyens de stocker des tâches.
///
/// Toutes les méthodes sont asynchrones (chapitre 15), même celles
/// de l'implémentation en mémoire qui pourraient répondre
/// immédiatement. Pourquoi ? Parce que le disque et la base de
/// données, eux, sont lents. Si l'interface était synchrone, il
/// faudrait la changer au 58.21 — et changer aussi tout le code
/// qui l'appelle. On paie d'avance le prix de l'asynchrone.
abstract class DepotTaches {
  /// Renvoie toutes les tâches enregistrées.
  Future<List<Tache>> chargerTout();

  /// Ajoute une tâche.
  Future<void> ajouter(Tache tache);

  /// Remplace la tâche portant le même `id`.
  Future<void> modifier(Tache tache);

  /// Supprime la tâche portant cet `id`.
  Future<void> supprimer(String id);

  /// Écrase entièrement le contenu du dépôt.
  ///
  /// Utile pour restaurer un ordre après une annulation (58.14)
  /// ou pour vider les tâches terminées.
  Future<void> remplacerTout(List<Tache> taches);
}
```

**`lib/donnees/depot_memoire.dart`**

```dart
import '../modeles/tache.dart';
import 'depot_taches.dart';

/// Dépôt volatile : tout est perdu à la fermeture de l'application.
///
/// Il sert pendant tout le développement de l'interface (58.6 à
/// 58.19) et, plus tard, dans les tests : un test ne doit jamais
/// écrire sur le vrai disque de la machine.
class DepotMemoire implements DepotTaches {
  DepotMemoire({List<Tache>? initiales})
      : _taches = List<Tache>.from(initiales ?? const <Tache>[]);

  final List<Tache> _taches;

  @override
  Future<List<Tache>> chargerTout() async {
    // On renvoie une COPIE : l'appelant ne doit pas pouvoir
    // modifier la liste interne du dépôt directement.
    return List<Tache>.from(_taches);
  }

  @override
  Future<void> ajouter(Tache tache) async {
    _taches.add(tache);
  }

  @override
  Future<void> modifier(Tache tache) async {
    final int index = _taches.indexWhere((Tache t) => t.id == tache.id);
    if (index != -1) {
      _taches[index] = tache;
    }
  }

  @override
  Future<void> supprimer(String id) async {
    _taches.removeWhere((Tache t) => t.id == id);
  }

  @override
  Future<void> remplacerTout(List<Tache> taches) async {
    _taches
      ..clear()
      ..addAll(taches);
  }
}
```

### Un jeu de données de démonstration

Travailler sur une liste vide est pénible. Fabriquons quelques tâches d'exemple, dans le même dossier.

**`lib/donnees/taches_demonstration.dart`**

```dart
import '../modeles/priorite.dart';
import '../modeles/tache.dart';

/// Quelques tâches pour développer l'interface sans partir du vide.
///
/// Les échéances sont calculées PAR RAPPORT à aujourd'hui pour que
/// l'exemple « en retard » du 58.19 reste vrai quelle que soit la
/// date à laquelle vous lisez ce chapitre.
List<Tache> tachesDeDemonstration() {
  final DateTime aujourdHui = DateTime.now();

  DateTime dans(int jours) {
    return DateTime(
      aujourdHui.year,
      aujourdHui.month,
      aujourdHui.day + jours,
    );
  }

  return <Tache>[
    Tache(
      id: 'demo-1',
      titre: 'Corriger le bug de collision',
      description: 'Le joueur traverse le sol à grande vitesse.',
      priorite: Priorite.urgente,
      echeance: dans(-3),
      etiquettes: <String>['jeu', 'bug'],
    ),
    Tache(
      id: 'demo-2',
      titre: 'Dessiner le sprite du boss',
      description: '4 directions, 8 images par cycle.',
      priorite: Priorite.haute,
      echeance: dans(6),
      etiquettes: <String>['graphisme'],
    ),
    Tache(
      id: 'demo-3',
      titre: 'Écrire la musique du niveau 3',
      priorite: Priorite.normale,
      etiquettes: <String>['audio'],
    ),
    Tache(
      id: 'demo-4',
      titre: 'Équilibrer les dégâts de l\'épée',
      faite: true,
      priorite: Priorite.normale,
      etiquettes: <String>['gameplay'],
    ),
    Tache(
      id: 'demo-5',
      titre: 'Tester le build Android',
      priorite: Priorite.basse,
      echeance: dans(0),
      etiquettes: <String>['build'],
    ),
  ];
}
```

**État exécutable.** Les données existent. Nous pouvons enfin les afficher.

---

## 58.6 — Afficher la liste avec `ListView.builder`

Le chapitre 48 a établi la règle : dès qu'une liste peut dépasser une dizaine d'éléments, on utilise `ListView.builder`, jamais `Column` ni `ListView(children: ...)`. `builder` ne construit que les tuiles visibles.

Pour cette étape, l'état reste local et géré par `setState` (chapitre 45). Nous le déplacerons dans un `ChangeNotifier` au 58.20 ; commencer simple permet de voir ce que `provider` fait gagner.

**`lib/ecrans/ecran_taches.dart`**

```dart
import 'package:flutter/material.dart';

import '../donnees/depot_memoire.dart';
import '../donnees/depot_taches.dart';
import '../donnees/taches_demonstration.dart';
import '../modeles/tache.dart';

/// Écran principal : la liste des tâches.
class EcranTaches extends StatefulWidget {
  const EcranTaches({super.key});

  @override
  State<EcranTaches> createState() => _EtatEcranTaches();
}

class _EtatEcranTaches extends State<EcranTaches> {
  /// Le dépôt. Déclaré du type de l'INTERFACE, pas de
  /// l'implémentation : ainsi, remplacer `DepotMemoire` par
  /// `DepotPrefs` au 58.21 ne changera qu'une seule ligne.
  late final DepotTaches _depot = DepotMemoire(
    initiales: tachesDeDemonstration(),
  );

  /// Copie locale des tâches, celle qu'affiche l'écran.
  List<Tache> _taches = <Tache>[];

  /// `true` tant que le chargement initial n'est pas terminé.
  bool _chargement = true;

  @override
  void initState() {
    super.initState();
    // `initState` ne peut pas être `async` : on lance le
    // chargement sans l'attendre (chapitre 45).
    _charger();
  }

  Future<void> _charger() async {
    final List<Tache> chargees = await _depot.chargerTout();
    // Après un `await`, le widget a pu être détruit. Ne jamais
    // appeler setState sans vérifier `mounted`.
    if (!mounted) {
      return;
    }
    setState(() {
      _taches = chargees;
      _chargement = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mes tâches')),
      body: _chargement
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              // itemCount : combien de tuiles au total.
              itemCount: _taches.length,
              // itemBuilder : appelé UNIQUEMENT pour les tuiles
              // qui entrent dans la zone visible.
              itemBuilder: (BuildContext context, int index) {
                final Tache tache = _taches[index];
                return ListTile(
                  leading: Icon(
                    tache.faite
                        ? Icons.check_circle
                        : Icons.radio_button_unchecked,
                  ),
                  title: Text(tache.titre),
                  subtitle: Text(tache.priorite.libelle),
                );
              },
            ),
    );
  }
}
```

Et le `main.dart` doit maintenant pointer vers ce fichier.

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

import 'ecrans/ecran_taches.dart';

void main() {
  runApp(const ApplicationTaches());
}

class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mes tâches',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
      ),
      home: const EcranTaches(),
    );
  }
}
```

**État exécutable.** Cinq tuiles s'affichent.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                                    │
├────────────────────────────────────────────────┤
│  ○  Corriger le bug de collision               │
│     Urgente                                    │
│  ○  Dessiner le sprite du boss                 │
│     Haute                                      │
│  ○  Écrire la musique du niveau 3              │
│     Normale                                    │
│  ●  Équilibrer les dégâts de l'épée            │
│     Normale                                    │
│  ○  Tester le build Android                    │
│     Basse                                      │
└────────────────────────────────────────────────┘
```

> **Remarque.** L'indicateur de chargement passe si vite qu'on ne le voit pas : `DepotMemoire` répond immédiatement. Il ne sera pas inutile pour autant : au 58.21, la lecture du disque prendra de vrais millisecondes, et au 58.22 la base de données peut mettre plus longtemps encore.

---

## 58.7 — L'état vide

Que voit l'utilisateur quand il installe l'application pour la première fois ? Avec le code actuel : un écran blanc. Un écran blanc, pour un débutant, veut dire « l'application est cassée ».

Un état vide correct répond à trois questions : **qu'est-ce que je regarde**, **pourquoi c'est vide**, **que dois-je faire**.

**`lib/widgets/etat_vide.dart`**

```dart
import 'package:flutter/material.dart';

/// Message affiché quand aucune tâche n'est visible.
///
/// Le widget est volontairement générique : il sert à la fois pour
/// « aucune tâche du tout » et, au 58.15, pour « aucun résultat de
/// recherche ». Seuls le texte et l'icône changent.
class EtatVide extends StatelessWidget {
  const EtatVide({
    super.key,
    required this.icone,
    required this.titre,
    required this.message,
  });

  final IconData icone;
  final String titre;
  final String message;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Icon(
              icone,
              size: 72,
              // withValues remplace l'ancien withOpacity :
              // il travaille sans perte de précision.
              color: theme.colorScheme.primary.withValues(alpha: 0.4),
            ),
            const SizedBox(height: 16),
            Text(
              titre,
              style: theme.textTheme.titleLarge,
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 8),
            Text(
              message,
              style: theme.textTheme.bodyMedium?.copyWith(
                color: theme.colorScheme.onSurfaceVariant,
              ),
              textAlign: TextAlign.center,
            ),
          ],
        ),
      ),
    );
  }
}
```

Dans `ecran_taches.dart`, remplacez le corps de `build` par une méthode qui choisit entre trois affichages.

```dart
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Mes tâches')),
      body: _corps(),
    );
  }

  /// Trois cas, dans cet ordre : je charge, je n'ai rien, j'affiche.
  Widget _corps() {
    if (_chargement) {
      return const Center(child: CircularProgressIndicator());
    }
    if (_taches.isEmpty) {
      return const EtatVide(
        icone: Icons.check_circle_outline,
        titre: 'Aucune tâche pour l\'instant',
        message: 'Appuyez sur + pour créer votre première tâche.',
      );
    }
    return ListView.builder(
      itemCount: _taches.length,
      itemBuilder: (BuildContext context, int index) {
        final Tache tache = _taches[index];
        return ListTile(
          leading: Icon(
            tache.faite ? Icons.check_circle : Icons.radio_button_unchecked,
          ),
          title: Text(tache.titre),
          subtitle: Text(tache.priorite.libelle),
        );
      },
    );
  }
```

N'oubliez pas l'import :

```dart
import '../widgets/etat_vide.dart';
```

**État exécutable.** Pour voir l'état vide, remplacez temporairement `initiales: tachesDeDemonstration()` par `initiales: <Tache>[]`.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                                    │
├────────────────────────────────────────────────┤
│                                                │
│                    ☑                           │
│                                                │
│         Aucune tâche pour l'instant            │
│                                                │
│    Appuyez sur + pour créer votre première     │
│                   tâche.                       │
│                                                │
└────────────────────────────────────────────────┘
```

Remettez ensuite les tâches de démonstration.

---

## 58.8 — La case à cocher et le titre barré

Une liste de tâches sans case à cocher n'est qu'une liste. Nous allons :

1. remplacer l'icône par une vraie `Checkbox` ;
2. barrer le titre des tâches faites avec `TextDecoration.lineThrough` ;
3. colorer un liseré vertical selon la priorité.

### La couleur de priorité

La couleur est une décision d'interface, pas une propriété de la donnée. Elle n'a donc rien à faire dans `Priorite`. On l'isole dans un utilitaire.

**`lib/utilitaires/couleurs.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/priorite.dart';

/// Couleur associée à chaque priorité.
///
/// On passe le `BuildContext` pour pouvoir puiser dans le thème
/// (chapitre 51) : la couleur d'erreur du thème s'adapte
/// automatiquement au mode sombre du 58.23.
Color couleurPriorite(BuildContext context, Priorite priorite) {
  final ColorScheme couleurs = Theme.of(context).colorScheme;

  switch (priorite) {
    case Priorite.basse:
      return couleurs.outline;
    case Priorite.normale:
      return couleurs.primary;
    case Priorite.haute:
      return Colors.orange;
    case Priorite.urgente:
      return couleurs.error;
  }
}

/// Icône associée à chaque priorité.
IconData iconePriorite(Priorite priorite) {
  switch (priorite) {
    case Priorite.basse:
      return Icons.keyboard_arrow_down;
    case Priorite.normale:
      return Icons.remove;
    case Priorite.haute:
      return Icons.keyboard_arrow_up;
    case Priorite.urgente:
      return Icons.priority_high;
  }
}
```

> **Remarque.** Un `switch` sur un `enum` qui couvre toutes les valeurs n'a pas besoin de `default`, et c'est un avantage : le jour où vous ajoutez `Priorite.critique`, le compilateur signalera immédiatement les deux fonctions à compléter. Un `default` aurait masqué le problème.

### La tuile

**`lib/widgets/tuile_tache.dart`** (première version)

```dart
import 'package:flutter/material.dart';

import '../modeles/tache.dart';
import '../utilitaires/couleurs.dart';

/// Une ligne de la liste des tâches.
///
/// Ce widget est PUREMENT d'affichage : il ne modifie rien
/// lui-même, il prévient son parent par des rappels
/// (`onBascule`, `onAppui`). C'est la remontée d'état du
/// chapitre 45.
class TuileTache extends StatelessWidget {
  const TuileTache({
    super.key,
    required this.tache,
    required this.onBascule,
    this.onAppui,
  });

  final Tache tache;

  /// Appelé quand l'utilisateur coche ou décoche la tâche.
  final ValueChanged<bool> onBascule;

  /// Appelé quand l'utilisateur touche la tuile (édition, 58.12).
  final VoidCallback? onAppui;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Color couleur = couleurPriorite(context, tache.priorite);

    return Container(
      decoration: BoxDecoration(
        border: Border(
          // Le liseré vertical coloré, à gauche de la tuile.
          left: BorderSide(color: couleur, width: 5),
        ),
      ),
      child: CheckboxListTile(
        // La case est à gauche, comme dans la maquette.
        controlAffinity: ListTileControlAffinity.leading,
        value: tache.faite,
        // `bool?` car la case peut être tristate ; ici elle ne
        // l'est pas, mais la signature l'impose.
        onChanged: (bool? valeur) => onBascule(valeur ?? false),
        title: Text(
          tache.titre,
          style: TextStyle(
            // LE point de cette étape : barrer le titre.
            decoration: tache.faite ? TextDecoration.lineThrough : null,
            color: tache.faite
                ? theme.colorScheme.onSurfaceVariant
                : theme.colorScheme.onSurface,
            fontWeight: FontWeight.w500,
          ),
        ),
        subtitle: Text(
          tache.priorite.libelle.toUpperCase(),
          style: theme.textTheme.labelSmall?.copyWith(
            color: couleur,
            letterSpacing: 0.8,
          ),
        ),
        // Un appui hors de la case déclenche onAppui.
        onFocusChange: null,
      ),
    );
  }
}
```

Il reste un détail : `CheckboxListTile` occupe toute la ligne, on ne peut donc pas y brancher un `onTap` distinct. Nous corrigerons ce point au 58.12 en repassant à un `ListTile` classique avec une `Checkbox` explicite. Pour l'instant, retirez la ligne `onFocusChange: null;` inutile et gardez le rappel `onAppui` non branché.

### Brancher la bascule dans l'écran

Dans `ecran_taches.dart`, ajoutez la méthode qui bascule une tâche, et utilisez `TuileTache`.

```dart
  /// Coche ou décoche une tâche.
  ///
  /// On ne modifie PAS l'objet : on en crée une copie avec
  /// `copyWith`, on la range dans le dépôt, puis on rafraîchit
  /// la liste locale.
  Future<void> _basculer(Tache tache, bool faite) async {
    final Tache modifiee = tache.copyWith(faite: faite);
    await _depot.modifier(modifiee);
    final List<Tache> rechargees = await _depot.chargerTout();
    if (!mounted) {
      return;
    }
    setState(() => _taches = rechargees);
  }
```

Et dans `_corps()` :

```dart
    return ListView.builder(
      itemCount: _taches.length,
      itemBuilder: (BuildContext context, int index) {
        final Tache tache = _taches[index];
        return TuileTache(
          tache: tache,
          onBascule: (bool faite) => _basculer(tache, faite),
        );
      },
    );
```

Avec l'import :

```dart
import '../widgets/tuile_tache.dart';
```

**État exécutable.** Les cases se cochent, les titres se barrent, les liserés colorés apparaissent.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                                    │
├────────────────────────────────────────────────┤
│ ▌ ☐  Corriger le bug de collision              │
│ ▌    URGENTE                                   │
├────────────────────────────────────────────────┤
│ ▌ ☐  Dessiner le sprite du boss                │
│ ▌    HAUTE                                     │
├────────────────────────────────────────────────┤
│ ▌ ☑  ̶É̶q̶u̶i̶l̶i̶b̶r̶e̶r̶ ̶l̶e̶s̶ ̶d̶é̶g̶â̶t̶s̶ ̶d̶e̶ ̶l̶'̶é̶p̶é̶e̶            │
│ ▌    NORMALE                                   │
└────────────────────────────────────────────────┘
```

---

## 58.9 — Ajouter une tâche par une boîte de dialogue

Première façon d'ajouter une tâche : la plus rapide. Une boîte de dialogue avec un seul champ. Elle suffit pour 80 % des usages réels — on note un titre, on complètera plus tard.

Le chapitre 49 a présenté `TextField` et `TextEditingController` ; le chapitre 50, `showDialog` et le retour de données. On combine les deux.

### Le point clé : `showDialog` renvoie un `Future`

```text
final String? titre = await showDialog<String>(
  context: context,
  builder: (ctx) => AlertDialog(...),
);

       │
       ├─ l'utilisateur valide  -> Navigator.pop(ctx, 'mon titre')
       │                           => titre == 'mon titre'
       │
       └─ l'utilisateur annule  -> Navigator.pop(ctx)  ou touche l'extérieur
                                   => titre == null
```

Le type générique `<String>` est important : sans lui, `showDialog` renvoie `Future<dynamic>` et vous perdez toute vérification de type.

### Le dialogue

**`lib/widgets/dialogue_ajout_rapide.dart`**

```dart
import 'package:flutter/material.dart';

/// Boîte de dialogue à un seul champ : le titre de la tâche.
///
/// Elle renvoie le titre saisi, ou `null` si l'utilisateur annule.
/// C'est un `StatefulWidget` car elle possède un contrôleur de
/// texte, qui doit être libéré dans `dispose` (chapitre 45).
class DialogueAjoutRapide extends StatefulWidget {
  const DialogueAjoutRapide({super.key});

  @override
  State<DialogueAjoutRapide> createState() => _EtatDialogueAjoutRapide();
}

class _EtatDialogueAjoutRapide extends State<DialogueAjoutRapide> {
  final TextEditingController _controleur = TextEditingController();

  /// Message d'erreur affiché sous le champ, `null` si tout va bien.
  String? _erreur;

  @override
  void dispose() {
    // Un contrôleur non libéré est une fuite mémoire.
    _controleur.dispose();
    super.dispose();
  }

  void _valider() {
    final String titre = _controleur.text.trim();
    if (titre.isEmpty) {
      setState(() => _erreur = 'Le titre ne peut pas être vide.');
      return;
    }
    // On ferme le dialogue EN RENVOYANT le titre.
    Navigator.of(context).pop(titre);
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Nouvelle tâche'),
      content: TextField(
        controller: _controleur,
        // Le clavier s'ouvre tout de suite.
        autofocus: true,
        textCapitalization: TextCapitalization.sentences,
        decoration: InputDecoration(
          labelText: 'Titre',
          hintText: 'Dessiner le sprite du boss',
          errorText: _erreur,
        ),
        // La touche « Entrée » du clavier valide.
        textInputAction: TextInputAction.done,
        onSubmitted: (_) => _valider(),
      ),
      actions: <Widget>[
        TextButton(
          // pop() sans argument => le Future vaut null.
          onPressed: () => Navigator.of(context).pop(),
          child: const Text('Annuler'),
        ),
        FilledButton(
          onPressed: _valider,
          child: const Text('Ajouter'),
        ),
      ],
    );
  }
}
```

### L'appel depuis l'écran

Dans `ecran_taches.dart` :

```dart
  /// Ouvre le dialogue d'ajout rapide et crée la tâche saisie.
  Future<void> _ajoutRapide() async {
    final String? titre = await showDialog<String>(
      context: context,
      builder: (BuildContext _) => const DialogueAjoutRapide(),
    );

    // L'utilisateur a annulé.
    if (titre == null) {
      return;
    }

    final Tache nouvelle = Tache(
      id: _uuid.v4(),
      titre: titre,
    );
    await _depot.ajouter(nouvelle);
    final List<Tache> rechargees = await _depot.chargerTout();
    if (!mounted) {
      return;
    }
    setState(() => _taches = rechargees);
  }
```

Ajoutez le générateur d'identifiants comme champ de `_EtatEcranTaches` :

```dart
  final Uuid _uuid = const Uuid();
```

et le bouton flottant au `Scaffold` :

```dart
      floatingActionButton: FloatingActionButton(
        onPressed: _ajoutRapide,
        tooltip: 'Ajouter une tâche',
        child: const Icon(Icons.add),
      ),
```

Imports à ajouter en haut du fichier :

```dart
import 'package:uuid/uuid.dart';

import '../widgets/dialogue_ajout_rapide.dart';
```

> **Remarque sur `Uuid`.** `const Uuid()` fabrique un générateur. `v4()` produit un identifiant aléatoire de 128 bits, du type `110ec58a-a0f2-4ac4-8393-c866d813b8d1`. La probabilité de collision est négligeable, y compris entre plusieurs appareils. C'est exactement ce qu'il faut pour un identifiant de tâche. Ne comptez jamais sur `DateTime.now().millisecondsSinceEpoch` : deux ajouts dans la même milliseconde produiraient le même identifiant, et le `Dismissible` du 58.13 planterait.

**État exécutable.** Le bouton `+` ouvre le dialogue ; la tâche saisie apparaît en bas de liste.

```text
┌────────────────────────────────────────────────┐
│                                                │
│      ┌────────────────────────────────┐        │
│      │  Nouvelle tâche                │        │
│      │  ┌──────────────────────────┐  │        │
│      │  │ Titre                    │  │        │
│      │  │ Dessiner le sprite...    │  │        │
│      │  └──────────────────────────┘  │        │
│      │            ANNULER   AJOUTER   │        │
│      └────────────────────────────────┘        │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 58.10 — Formater les dates avec `intl`

Avant d'ouvrir un sélecteur de date, il faut savoir afficher une date proprement. `DateTime.toString()` produit `2026-08-21 00:00:00.000` : illisible.

Le paquet `intl` (chapitre 16 pour l'ajout de paquets) fournit `DateFormat`, qui sait écrire une date dans la langue de l'utilisateur.

### Initialiser la locale française

Deux initialisations sont nécessaires, et on les oublie systématiquement toutes les deux.

**1. Pour `intl` (les noms de mois et de jours).** Dans `main`, avant `runApp` :

```dart
await initializeDateFormatting('fr_FR', null);
Intl.defaultLocale = 'fr_FR';
```

**2. Pour les widgets Material** (le sélecteur de date, les libellés « Annuler », « OK »). Dans `MaterialApp` :

```dart
localizationsDelegates: GlobalMaterialLocalizations.delegates,
supportedLocales: const <Locale>[Locale('fr'), Locale('en')],
locale: const Locale('fr'),
```

**`lib/main.dart`** (version intermédiaire, avec la localisation)

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';

import 'ecrans/ecran_taches.dart';

Future<void> main() async {
  // Obligatoire avant tout appel asynchrone précédant runApp :
  // cela initialise le pont entre Dart et la plateforme.
  WidgetsFlutterBinding.ensureInitialized();

  // Charge les noms de mois et de jours en français.
  await initializeDateFormatting('fr_FR', null);
  // Évite d'avoir à répéter 'fr_FR' à chaque DateFormat.
  Intl.defaultLocale = 'fr_FR';

  runApp(const ApplicationTaches());
}

class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mes tâches',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
      ),
      // Traduit les widgets Material (dont showDatePicker).
      localizationsDelegates: GlobalMaterialLocalizations.delegates,
      supportedLocales: const <Locale>[
        Locale('fr'),
        Locale('en'),
      ],
      locale: const Locale('fr'),
      home: const EcranTaches(),
    );
  }
}
```

### L'utilitaire de dates

**`lib/utilitaires/dates.dart`**

```dart
import 'package:intl/intl.dart';

/// Formats réutilisables.
///
/// On les déclare une seule fois : construire un `DateFormat` est
/// coûteux, et le faire dans un `build` appelé 60 fois par seconde
/// est un vrai problème de performance.
final DateFormat _formatLong = DateFormat('EEEE d MMMM y');
final DateFormat _formatCourt = DateFormat('d MMM y');

/// « vendredi 21 août 2026 »
String dateLongue(DateTime date) => _formatLong.format(date);

/// « 21 août 2026 »
String dateCourte(DateTime date) => _formatCourt.format(date);

/// Rend une date lisible par rapport à aujourd'hui.
///
/// Renvoie « aujourd'hui », « demain », « hier », « dans 5 jours »,
/// « il y a 3 jours », ou la date courte au-delà d'une semaine.
/// C'est ce que l'utilisateur veut vraiment lire sur une tâche.
String dateRelative(DateTime date) {
  final DateTime maintenant = DateTime.now();

  // On compare des JOURS, pas des instants : sinon « aujourd'hui
  // à 8 h » serait « il y a 1 jour » quand il est 9 h le lendemain.
  final DateTime jourCible = DateTime(date.year, date.month, date.day);
  final DateTime jourActuel = DateTime(
    maintenant.year,
    maintenant.month,
    maintenant.day,
  );

  final int ecart = jourCible.difference(jourActuel).inDays;

  if (ecart == 0) {
    return 'aujourd\'hui';
  }
  if (ecart == 1) {
    return 'demain';
  }
  if (ecart == -1) {
    return 'hier';
  }
  if (ecart > 1 && ecart <= 7) {
    return 'dans $ecart jours';
  }
  if (ecart < -1 && ecart >= -7) {
    return 'il y a ${-ecart} jours';
  }
  return dateCourte(date);
}
```

### Vérification

```dart
void main() async {
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';

  final DateTime reference = DateTime(2026, 8, 21);
  print(dateLongue(reference));
  print(dateCourte(reference));
}
```

**Résultat :**

```text
vendredi 21 août 2026
21 août 2026
```

> **Remarque.** Si vous obtenez `Friday, August 21, 2026`, c'est que `Intl.defaultLocale` n'a pas été positionné. Si vous obtenez une exception `LocaleDataException: Locale data has not been initialized`, c'est que `initializeDateFormatting` a été oublié. Ces deux erreurs représentent la quasi-totalité des problèmes rencontrés avec `intl`.

### Afficher la date dans la tuile

Enrichissons le sous-titre de `TuileTache`. Remplacez le `subtitle` par :

```dart
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            const SizedBox(height: 2),
            Row(
              children: <Widget>[
                Text(
                  tache.priorite.libelle.toUpperCase(),
                  style: theme.textTheme.labelSmall?.copyWith(
                    color: couleur,
                    letterSpacing: 0.8,
                  ),
                ),
                if (tache.echeance != null) ...<Widget>[
                  Text(
                    '  ·  ',
                    style: theme.textTheme.labelSmall,
                  ),
                  Icon(
                    Icons.event,
                    size: 13,
                    color: theme.colorScheme.onSurfaceVariant,
                  ),
                  const SizedBox(width: 3),
                  Text(
                    dateRelative(tache.echeance!),
                    style: theme.textTheme.labelSmall,
                  ),
                ],
              ],
            ),
          ],
        ),
```

avec l'import `import '../utilitaires/dates.dart';`.

**État exécutable.** Chaque tuile affiche sa priorité et, s'il y en a une, son échéance en français.

```text
├────────────────────────────────────────────────┤
│ ▌ ☐  Corriger le bug de collision              │
│ ▌    URGENTE  ·  [cal] il y a 3 jours          │
├────────────────────────────────────────────────┤
│ ▌ ☐  Dessiner le sprite du boss                │
│ ▌    HAUTE  ·  [cal] dans 6 jours              │
├────────────────────────────────────────────────┤
│ ▌ ☐  Écrire la musique du niveau 3             │
│ ▌    NORMALE                                   │
└────────────────────────────────────────────────┘
```

---

## 58.11 — Le formulaire complet avec validation

Le dialogue du 58.9 est trop pauvre : ni description, ni priorité, ni échéance, ni étiquettes. Il faut un vrai écran de formulaire.

Le chapitre 49 a posé les trois pièces :

```text
GlobalKey<FormState>      la télécommande du formulaire
Form(key: _cle)           le conteneur
TextFormField(            un champ qui SAIT se valider
  validator: (v) => ...   renvoie null si OK, un message sinon
)

_cle.currentState!.validate()   déclenche tous les validators
```

### Le sélecteur de date

`showDatePicker` renvoie un `Future<DateTime?>` : la date choisie, ou `null` si l'utilisateur annule. Ses trois paramètres obligatoires sont `context`, `firstDate` et `lastDate`.

```dart
final DateTime? choisie = await showDatePicker(
  context: context,
  initialDate: _echeance ?? DateTime.now(),
  firstDate: DateTime(2020),
  lastDate: DateTime(2100),
  helpText: 'Choisir une échéance',
);
```

`firstDate` et `lastDate` bornent le calendrier. Ne mettez pas `firstDate: DateTime.now()` : on veut pouvoir saisir une tâche déjà en retard.

### L'écran de formulaire

**`lib/ecrans/ecran_formulaire.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:uuid/uuid.dart';

import '../modeles/priorite.dart';
import '../modeles/tache.dart';
import '../utilitaires/couleurs.dart';
import '../utilitaires/dates.dart';

/// Écran de création ET de modification d'une tâche.
///
/// Un seul écran pour les deux usages : si `tacheExistante` est
/// nul, on crée ; sinon, on modifie. C'est le motif le plus
/// courant, et il évite d'écrire deux fois le même formulaire.
///
/// L'écran renvoie la tâche construite via `Navigator.pop`,
/// ou `null` si l'utilisateur abandonne (chapitre 50).
class EcranFormulaire extends StatefulWidget {
  const EcranFormulaire({super.key, this.tacheExistante});

  final Tache? tacheExistante;

  bool get estModification => tacheExistante != null;

  @override
  State<EcranFormulaire> createState() => _EtatEcranFormulaire();
}

class _EtatEcranFormulaire extends State<EcranFormulaire> {
  /// La clé qui donne accès à l'état du formulaire.
  ///
  /// Elle est `final` et créée UNE SEULE FOIS : recréer la clé à
  /// chaque build ferait perdre l'état de validation.
  final GlobalKey<FormState> _cleFormulaire = GlobalKey<FormState>();

  late final TextEditingController _controleurTitre;
  late final TextEditingController _controleurDescription;
  late final TextEditingController _controleurEtiquettes;

  late Priorite _priorite;
  DateTime? _echeance;

  @override
  void initState() {
    super.initState();
    final Tache? existante = widget.tacheExistante;

    // Pré-remplissage : c'est ici, et nulle part ailleurs, que se
    // fait la différence entre création et modification.
    _controleurTitre = TextEditingController(text: existante?.titre ?? '');
    _controleurDescription = TextEditingController(
      text: existante?.description ?? '',
    );
    _controleurEtiquettes = TextEditingController(
      text: existante?.etiquettes.join(', ') ?? '',
    );
    _priorite = existante?.priorite ?? Priorite.normale;
    _echeance = existante?.echeance;
  }

  @override
  void dispose() {
    _controleurTitre.dispose();
    _controleurDescription.dispose();
    _controleurEtiquettes.dispose();
    super.dispose();
  }

  /// Valide le titre. Renvoie `null` quand tout va bien.
  String? _validerTitre(String? valeur) {
    final String titre = (valeur ?? '').trim();
    if (titre.isEmpty) {
      return 'Le titre est obligatoire.';
    }
    if (titre.length < 3) {
      return 'Le titre doit faire au moins 3 caractères.';
    }
    if (titre.length > 80) {
      return 'Le titre ne doit pas dépasser 80 caractères.';
    }
    return null;
  }

  Future<void> _choisirEcheance() async {
    final DateTime? choisie = await showDatePicker(
      context: context,
      initialDate: _echeance ?? DateTime.now(),
      firstDate: DateTime(2020),
      lastDate: DateTime(2100),
      helpText: 'Choisir une échéance',
      cancelText: 'Annuler',
      confirmText: 'Valider',
    );

    if (choisie == null) {
      return;
    }
    setState(() => _echeance = choisie);
  }

  /// Découpe « jeu, bug ,  graphisme » en ['jeu', 'bug', 'graphisme'].
  List<String> _lireEtiquettes() {
    return _controleurEtiquettes.text
        .split(',')
        .map((String e) => e.trim().toLowerCase())
        .where((String e) => e.isNotEmpty)
        .toSet() // supprime les doublons
        .toList();
  }

  void _enregistrer() {
    // validate() appelle TOUS les validator du formulaire et
    // affiche les messages. Il renvoie false si l'un échoue.
    if (!_cleFormulaire.currentState!.validate()) {
      return;
    }

    final Tache? existante = widget.tacheExistante;

    final Tache resultat = existante == null
        ? Tache(
            id: const Uuid().v4(),
            titre: _controleurTitre.text.trim(),
            description: _controleurDescription.text.trim(),
            priorite: _priorite,
            echeance: _echeance,
            etiquettes: _lireEtiquettes(),
          )
        : existante.copyWith(
            titre: _controleurTitre.text.trim(),
            description: _controleurDescription.text.trim(),
            priorite: _priorite,
            echeance: _echeance,
            // Si l'utilisateur a retiré l'échéance, il faut
            // l'EFFACER, pas la laisser inchangée.
            effacerEcheance: _echeance == null,
            etiquettes: _lireEtiquettes(),
          );

    Navigator.of(context).pop(resultat);
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(
        title: Text(
          widget.estModification ? 'Modifier la tâche' : 'Nouvelle tâche',
        ),
      ),
      body: Form(
        key: _cleFormulaire,
        // onUserInteraction : les messages d'erreur apparaissent
        // dès la saisie, pas seulement à la validation finale.
        autovalidateMode: AutovalidateMode.onUserInteraction,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: <Widget>[
            TextFormField(
              controller: _controleurTitre,
              autofocus: !widget.estModification,
              textCapitalization: TextCapitalization.sentences,
              maxLength: 80,
              decoration: const InputDecoration(
                labelText: 'Titre *',
                hintText: 'Dessiner le sprite du boss',
                border: OutlineInputBorder(),
              ),
              validator: _validerTitre,
            ),
            const SizedBox(height: 8),
            TextFormField(
              controller: _controleurDescription,
              textCapitalization: TextCapitalization.sentences,
              // Un champ multiligne : minLines fixe la hauteur de
              // départ, maxLines la hauteur maximale.
              minLines: 3,
              maxLines: 6,
              decoration: const InputDecoration(
                labelText: 'Description',
                hintText: '4 directions, 8 images par cycle.',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
            ),
            const SizedBox(height: 24),
            Text('Priorité', style: theme.textTheme.titleSmall),
            const SizedBox(height: 8),
            _selecteurPriorite(),
            const SizedBox(height: 24),
            Text('Échéance', style: theme.textTheme.titleSmall),
            const SizedBox(height: 8),
            _selecteurEcheance(),
            const SizedBox(height: 24),
            TextFormField(
              controller: _controleurEtiquettes,
              decoration: const InputDecoration(
                labelText: 'Étiquettes',
                hintText: 'graphisme, boss',
                helperText: 'Séparez-les par des virgules.',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.label_outline),
              ),
            ),
            const SizedBox(height: 32),
            FilledButton.icon(
              onPressed: _enregistrer,
              icon: const Icon(Icons.check),
              label: const Text('Enregistrer'),
              style: FilledButton.styleFrom(
                minimumSize: const Size.fromHeight(52),
              ),
            ),
          ],
        ),
      ),
    );
  }

  /// Quatre puces de priorité, sous forme de `ChoiceChip`.
  Widget _selecteurPriorite() {
    return Wrap(
      spacing: 8,
      children: Priorite.values.map((Priorite p) {
        final bool selectionnee = p == _priorite;
        return ChoiceChip(
          label: Text(p.libelle),
          avatar: Icon(
            iconePriorite(p),
            size: 18,
            color: couleurPriorite(context, p),
          ),
          selected: selectionnee,
          onSelected: (bool _) => setState(() => _priorite = p),
        );
      }).toList(),
    );
  }

  /// Bouton d'ouverture du calendrier, avec une croix pour effacer.
  Widget _selecteurEcheance() {
    final DateTime? echeance = _echeance;

    return InkWell(
      onTap: _choisirEcheance,
      borderRadius: BorderRadius.circular(4),
      child: InputDecorator(
        decoration: const InputDecoration(
          border: OutlineInputBorder(),
          prefixIcon: Icon(Icons.calendar_today),
        ),
        child: Row(
          children: <Widget>[
            Expanded(
              child: Text(
                echeance == null
                    ? 'Aucune échéance'
                    : dateLongue(echeance),
                style: TextStyle(
                  color: echeance == null
                      ? Theme.of(context).colorScheme.onSurfaceVariant
                      : null,
                ),
              ),
            ),
            if (echeance != null)
              IconButton(
                icon: const Icon(Icons.close),
                tooltip: 'Retirer l\'échéance',
                onPressed: () => setState(() => _echeance = null),
              ),
          ],
        ),
      ),
    );
  }
}
```

### Ouvrir le formulaire depuis l'écran principal

Dans `ecran_taches.dart`, remplacez `_ajoutRapide` par :

```dart
  /// Ouvre le formulaire de création et enregistre le résultat.
  Future<void> _creer() async {
    final Tache? nouvelle = await Navigator.of(context).push<Tache>(
      MaterialPageRoute<Tache>(
        builder: (BuildContext _) => const EcranFormulaire(),
      ),
    );

    if (nouvelle == null) {
      return;
    }
    await _depot.ajouter(nouvelle);
    final List<Tache> rechargees = await _depot.chargerTout();
    if (!mounted) {
      return;
    }
    setState(() => _taches = rechargees);
  }
```

et branchez `onPressed: _creer` sur le `FloatingActionButton`. Import :

```dart
import 'ecran_formulaire.dart';
```

> **Remarque.** Le dialogue rapide du 58.9 n'est pas perdu : il reste utile en appui long sur le bouton `+`. C'est le défi 3 du 58.28.

**État exécutable.** Le bouton `+` ouvre un formulaire complet. Un titre vide ou trop court est refusé avec un message sous le champ. La date se choisit dans un calendrier en français.

```text
┌────────────────────────────────────────────────┐
│  ←   Nouvelle tâche                            │
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │ Titre *                                  │  │
│  │ ab                                       │  │
│  └──────────────────────────────────────────┘  │
│  Le titre doit faire au moins 3 caractères.    │
│                                          2/80  │
└────────────────────────────────────────────────┘
```

---

## 58.12 — Modifier une tâche

Toute la difficulté est déjà réglée : `EcranFormulaire` accepte une `tacheExistante`. Il reste à rendre la tuile cliquable.

`CheckboxListTile` occupe toute la ligne et capte l'appui : impossible d'y ajouter un `onTap` distinct de la case. On revient donc à un `ListTile` classique avec une `Checkbox` placée en `leading`.

**`lib/widgets/tuile_tache.dart`** (version définitive)

```dart
import 'package:flutter/material.dart';

import '../modeles/tache.dart';
import '../utilitaires/couleurs.dart';
import '../utilitaires/dates.dart';

/// Une ligne de la liste des tâches.
class TuileTache extends StatelessWidget {
  const TuileTache({
    super.key,
    required this.tache,
    required this.onBascule,
    required this.onAppui,
  });

  final Tache tache;
  final ValueChanged<bool> onBascule;
  final VoidCallback onAppui;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Color couleur = couleurPriorite(context, tache.priorite);

    return Container(
      decoration: BoxDecoration(
        color: theme.colorScheme.surface,
        border: Border(left: BorderSide(color: couleur, width: 5)),
      ),
      child: ListTile(
        // La case à cocher : elle a son propre onChanged.
        leading: Checkbox(
          value: tache.faite,
          onChanged: (bool? v) => onBascule(v ?? false),
        ),
        // L'appui sur le reste de la tuile ouvre l'édition.
        onTap: onAppui,
        title: Text(
          tache.titre,
          maxLines: 2,
          overflow: TextOverflow.ellipsis,
          style: TextStyle(
            decoration: tache.faite ? TextDecoration.lineThrough : null,
            color: tache.faite
                ? theme.colorScheme.onSurfaceVariant
                : theme.colorScheme.onSurface,
            fontWeight: FontWeight.w500,
          ),
        ),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            const SizedBox(height: 3),
            Row(
              children: <Widget>[
                Text(
                  tache.priorite.libelle.toUpperCase(),
                  style: theme.textTheme.labelSmall
                      ?.copyWith(color: couleur, letterSpacing: 0.8),
                ),
                if (tache.echeance != null) ...<Widget>[
                  const SizedBox(width: 8),
                  Icon(
                    Icons.event,
                    size: 13,
                    color: theme.colorScheme.onSurfaceVariant,
                  ),
                  const SizedBox(width: 3),
                  Text(
                    dateRelative(tache.echeance!),
                    style: theme.textTheme.labelSmall,
                  ),
                ],
              ],
            ),
            if (tache.etiquettes.isNotEmpty) ...<Widget>[
              const SizedBox(height: 6),
              Wrap(
                spacing: 4,
                runSpacing: 2,
                children: tache.etiquettes
                    .map((String e) => Text(
                          '#$e',
                          style: theme.textTheme.labelSmall?.copyWith(
                            color: theme.colorScheme.primary,
                          ),
                        ))
                    .toList(),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

Dans `ecran_taches.dart`, ajoutez la méthode d'édition :

```dart
  /// Ouvre le formulaire pré-rempli et enregistre la modification.
  Future<void> _modifier(Tache tache) async {
    final Tache? modifiee = await Navigator.of(context).push<Tache>(
      MaterialPageRoute<Tache>(
        builder: (BuildContext _) => EcranFormulaire(tacheExistante: tache),
      ),
    );

    if (modifiee == null) {
      return;
    }
    await _depot.modifier(modifiee);
    final List<Tache> rechargees = await _depot.chargerTout();
    if (!mounted) {
      return;
    }
    setState(() => _taches = rechargees);
  }
```

et branchez-la :

```dart
        return TuileTache(
          tache: tache,
          onBascule: (bool faite) => _basculer(tache, faite),
          onAppui: () => _modifier(tache),
        );
```

**État exécutable.** Un appui sur une tuile ouvre le formulaire pré-rempli ; « Enregistrer » met la tuile à jour ; la flèche de retour abandonne sans rien changer.

---

## 58.13 — Supprimer par glissement avec `Dismissible`

Le chapitre 48 a présenté `Dismissible`. Rappel de son fonctionnement :

```text
Dismissible
  key            OBLIGATOIRE et unique      <- le point critique
  child          la tuile normale
  background     ce qui apparaît en glissant vers la droite
  secondaryBackground  ... vers la gauche
  direction      quelles directions sont permises
  confirmDismiss  Future<bool?> : autoriser ou annuler le geste
  onDismissed    appelé APRÈS l'animation de disparition
```

### Pourquoi la `Key` est obligatoire

Flutter réconcilie l'ancien et le nouvel arbre de widgets. Sans clé, deux `Dismissible` de même type au même niveau sont considérés comme interchangeables. Après une suppression, la liste se décale d'un cran, et Flutter croit que c'est *la même* tuile qui a changé de contenu : l'état interne du `Dismissible` (position du glissement, animation en cours) reste attaché à la mauvaise tuile.

```text
AVANT le glissement                APRÈS suppression de l'index 1

index 0 : [A]                      index 0 : [A]
index 1 : [B]  <- glissée          index 1 : [C]   <- Flutter voit
index 2 : [C]                      index 2 : [D]      « l'élément 1
index 3 : [D]                                          a changé de texte »

Sans Key  -> l'état « je suis en train d'être supprimée »
             reste sur l'index 1, donc sur C. Résultat :
             C disparaît aussi, ou l'application lève
             « A dismissed Dismissible widget is still part of the tree. »

Avec Key(id) -> Flutter identifie les tuiles par leur identité
                réelle, pas par leur position. Tout est correct.
```

C'est exactement pour cette raison que le 58.9 insistait sur `Uuid` : la clé doit être **unique et stable**. `ValueKey(index)` ne convient pas — l'index change. `ValueKey(titre)` non plus — deux tâches peuvent porter le même titre.

### Le code

Dans `ecran_taches.dart`, entourez la tuile :

```dart
      itemBuilder: (BuildContext context, int index) {
        final Tache tache = _taches[index];

        return Dismissible(
          // La clé, unique et stable : l'identifiant de la tâche.
          key: ValueKey<String>(tache.id),
          // Un seul sens de glissement : de la droite vers la gauche.
          direction: DismissDirection.endToStart,
          background: _fondSuppression(context),
          onDismissed: (DismissDirection _) => _supprimer(tache, index),
          child: TuileTache(
            tache: tache,
            onBascule: (bool faite) => _basculer(tache, faite),
            onAppui: () => _modifier(tache),
          ),
        );
      },
```

Ajoutez le fond rouge et la suppression :

```dart
  /// Fond rouge révélé pendant le glissement.
  Widget _fondSuppression(BuildContext context) {
    return Container(
      color: Theme.of(context).colorScheme.errorContainer,
      alignment: Alignment.centerRight,
      padding: const EdgeInsets.symmetric(horizontal: 24),
      child: Icon(
        Icons.delete_outline,
        color: Theme.of(context).colorScheme.onErrorContainer,
      ),
    );
  }

  /// Supprime la tâche du dépôt.
  ///
  /// ATTENTION : quand `onDismissed` est appelé, le widget a DÉJÀ
  /// disparu de l'écran. Si la donnée n'était pas retirée de la
  /// liste, le prochain build la reconstruirait et Flutter lèverait
  /// « A dismissed Dismissible widget is still part of the tree. »
  Future<void> _supprimer(Tache tache, int index) async {
    // On retire d'abord de la liste locale, immédiatement.
    setState(() => _taches.removeAt(index));
    await _depot.supprimer(tache.id);
  }
```

> **Remarque.** `_taches` était affecté par `setState(() => _taches = rechargees)`, donc c'est bien une liste modifiable (`List.from` renvoie une liste modifiable). `removeAt` fonctionne. Si vous obtenez `Unsupported operation: Cannot remove from an unmodifiable list`, c'est que quelque part vous avez affecté une liste `const` ou `unmodifiable`.

**État exécutable.** Glisser une tuile vers la gauche la supprime, avec un fond rouge et une icône de corbeille.

```text
├────────────────────────────────────────────────┤
│  ← ← ←   Dessiner le sprite du boss   ░░░  X   │
├────────────────────────────────────────────────┤
```

---

## 58.14 — Annuler la suppression avec un `SnackBar`

Une suppression par glissement est trop facile à déclencher par accident. La règle d'ergonomie est simple : **toute action destructive doit être annulable**.

Le mécanisme : on mémorise la tâche supprimée *et sa position*, on affiche un `SnackBar` avec un `SnackBarAction`, et si l'utilisateur appuie sur ANNULER on réinsère la tâche exactement là où elle était.

Modifiez `_supprimer` :

```dart
  Future<void> _supprimer(Tache tache, int index) async {
    setState(() => _taches.removeAt(index));
    await _depot.supprimer(tache.id);

    // Après un await, `context` peut être invalide : on vérifie.
    if (!mounted) {
      return;
    }

    final ScaffoldMessengerState messager = ScaffoldMessenger.of(context);
    // On enlève le SnackBar précédent : sinon, cinq suppressions
    // rapides empilent cinq messages qui s'afficheront à la file.
    messager.hideCurrentSnackBar();
    messager.showSnackBar(
      SnackBar(
        content: Text('« ${tache.titre} » supprimée'),
        duration: const Duration(seconds: 5),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () => _restaurer(tache, index),
        ),
      ),
    );
  }

  /// Réinsère une tâche supprimée à sa position d'origine.
  Future<void> _restaurer(Tache tache, int index) async {
    setState(() {
      // La liste a pu raccourcir entre-temps : on borne l'index.
      final int position = index.clamp(0, _taches.length);
      _taches.insert(position, tache);
    });
    // Le dépôt doit refléter exactement l'ordre affiché.
    await _depot.remplacerTout(_taches);
  }
```

> **Remarque.** `ScaffoldMessenger.of(context)` doit être appelé **avant** tout `await` si vous n'avez pas de garde `mounted`, car `context` peut être devenu invalide. Ici la garde est présente, donc l'appel après `await` est correct. C'est l'avertissement `use_build_context_synchronously` du linter : il ne se déclenche pas quand un `if (!mounted) return;` précède l'usage.

**État exécutable.** Après un glissement, un bandeau apparaît en bas ; ANNULER remet la tâche à sa place exacte.

```text
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │ « Dessiner le sprite » supprimée ANNULER │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

---

## 58.15 — Filtrer : toutes, actives, terminées

Nous entrons dans la partie « logique métier ». Elle doit vivre dans des **fonctions pures**, c'est-à-dire des fonctions qui ne dépendent que de leurs arguments et ne modifient rien. C'est ce qui les rend testables au 58.24 sans lancer d'écran.

### Les critères

**`lib/logique/criteres.dart`**

```dart
/// Quelles tâches afficher.
enum Filtre {
  toutes('Toutes'),
  actives('Actives'),
  terminees('Terminées');

  const Filtre(this.libelle);

  final String libelle;
}

/// Dans quel ordre les afficher.
enum Tri {
  echeance('Échéance'),
  priorite('Priorité'),
  creation('Plus récentes'),
  alphabetique('A → Z');

  const Tri(this.libelle);

  final String libelle;
}
```

### La fonction de filtrage

**`lib/logique/filtrage.dart`** (première version, le tri et la recherche arrivent en 58.16 et 58.17)

```dart
import '../modeles/tache.dart';
import 'criteres.dart';

/// Ne garde que les tâches correspondant au filtre.
///
/// Fonction PURE : même entrée, même sortie, aucun effet de bord.
/// La liste reçue n'est jamais modifiée ; `where` en produit une
/// nouvelle (chapitre 14).
List<Tache> filtrerParEtat(List<Tache> source, Filtre filtre) {
  switch (filtre) {
    case Filtre.toutes:
      return List<Tache>.from(source);
    case Filtre.actives:
      return source.where((Tache t) => !t.faite).toList();
    case Filtre.terminees:
      return source.where((Tache t) => t.faite).toList();
  }
}
```

### La barre de filtres

`SegmentedButton` (Material 3) est le widget exact pour ce besoin : un choix unique parmi trois. Ses deux paramètres obligatoires sont `segments` (une `List<ButtonSegment<T>>`) et `selected` (un `Set<T>`, car il gère aussi la sélection multiple).

**`lib/widgets/barre_filtres.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/criteres.dart';

/// Les trois segments Toutes / Actives / Terminées.
class BarreFiltres extends StatelessWidget {
  const BarreFiltres({
    super.key,
    required this.filtre,
    required this.onChange,
  });

  final Filtre filtre;
  final ValueChanged<Filtre> onChange;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      child: SegmentedButton<Filtre>(
        segments: Filtre.values
            .map((Filtre f) => ButtonSegment<Filtre>(
                  value: f,
                  label: Text(f.libelle),
                ))
            .toList(),
        // Un Set, même pour une sélection unique.
        selected: <Filtre>{filtre},
        showSelectedIcon: false,
        onSelectionChanged: (Set<Filtre> choix) => onChange(choix.first),
      ),
    );
  }
}
```

### Brancher dans l'écran

Dans `_EtatEcranTaches`, ajoutez le champ et la liste dérivée :

```dart
  Filtre _filtre = Filtre.toutes;

  /// La liste réellement affichée : les données brutes, filtrées.
  ///
  /// On la recalcule à chaque build. C'est volontaire : conserver
  /// une seconde liste en champ obligerait à la resynchroniser à
  /// chaque modification, et c'est ainsi qu'on crée des bugs.
  List<Tache> get _visibles => filtrerParEtat(_taches, _filtre);
```

Puis remplacez le corps par une `Column` : la barre en haut, la liste en dessous.

```dart
  Widget _corps() {
    if (_chargement) {
      return const Center(child: CircularProgressIndicator());
    }

    final List<Tache> visibles = _visibles;

    return Column(
      children: <Widget>[
        Padding(
          padding: const EdgeInsets.fromLTRB(12, 12, 12, 8),
          child: BarreFiltres(
            filtre: _filtre,
            onChange: (Filtre f) => setState(() => _filtre = f),
          ),
        ),
        // Expanded est INDISPENSABLE : une ListView dans une Column
        // sans contrainte de hauteur lève « Vertical viewport was
        // given unbounded height » (chapitre 46).
        Expanded(
          child: visibles.isEmpty
              ? _vide()
              : ListView.builder(
                  itemCount: visibles.length,
                  itemBuilder: (BuildContext context, int index) {
                    final Tache tache = visibles[index];
                    return Dismissible(
                      key: ValueKey<String>(tache.id),
                      direction: DismissDirection.endToStart,
                      background: _fondSuppression(context),
                      // ATTENTION : l'index est celui de la liste
                      // VISIBLE. Pour supprimer, il faut l'index
                      // dans la liste COMPLÈTE.
                      onDismissed: (DismissDirection _) => _supprimer(
                        tache,
                        _taches.indexWhere((Tache t) => t.id == tache.id),
                      ),
                      child: TuileTache(
                        tache: tache,
                        onBascule: (bool faite) => _basculer(tache, faite),
                        onAppui: () => _modifier(tache),
                      ),
                    );
                  },
                ),
        ),
      ],
    );
  }

  /// L'état vide dépend du filtre actif.
  Widget _vide() {
    if (_taches.isEmpty) {
      return const EtatVide(
        icone: Icons.check_circle_outline,
        titre: 'Aucune tâche pour l\'instant',
        message: 'Appuyez sur + pour créer votre première tâche.',
      );
    }
    if (_filtre == Filtre.terminees) {
      return const EtatVide(
        icone: Icons.hourglass_empty,
        titre: 'Aucune tâche terminée',
        message: 'Cochez une tâche pour la voir apparaître ici.',
      );
    }
    return const EtatVide(
      icone: Icons.celebration_outlined,
      titre: 'Tout est terminé',
      message: 'Vous n\'avez plus rien à faire. Profitez-en.',
    );
  }
```

**Ce piège de l'index mérite qu'on s'y arrête.** `itemBuilder` reçoit l'index dans la liste *visible*. Si le filtre est « Actives » et que vous supprimez la deuxième tuile affichée, ce n'est pas forcément la deuxième tâche de la liste complète. D'où le `indexWhere` sur l'identifiant. Au 58.20, le `ChangeNotifier` supprimera par identifiant et ce problème disparaîtra définitivement.

**État exécutable.** Les trois segments filtrent la liste ; chaque cas vide affiche un message adapté.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches                                    │
├────────────────────────────────────────────────┤
│  [ Toutes ] [ Actives ] [•Terminées ]          │
├────────────────────────────────────────────────┤
│ ▌ ☑  ̶É̶q̶u̶i̶l̶i̶b̶r̶e̶r̶ ̶l̶e̶s̶ ̶d̶é̶g̶â̶t̶s̶ ̶d̶e̶ ̶l̶'̶é̶p̶é̶e̶            │
│ ▌    NORMALE                                   │
└────────────────────────────────────────────────┘
```

---

## 58.16 — Trier par échéance ou par priorité

Le tri paraît trivial jusqu'à ce qu'on rencontre les valeurs nulles. Une tâche sans échéance doit-elle passer avant ou après ? La réponse est : après. Une tâche sans date n'est pas urgente.

`sort` modifie la liste **en place** et renvoie `void`. Il ne faut donc jamais trier la liste source directement : on trie une copie.

Complétez **`lib/logique/filtrage.dart`** :

```dart
import '../modeles/tache.dart';
import 'criteres.dart';

List<Tache> filtrerParEtat(List<Tache> source, Filtre filtre) {
  switch (filtre) {
    case Filtre.toutes:
      return List<Tache>.from(source);
    case Filtre.actives:
      return source.where((Tache t) => !t.faite).toList();
    case Filtre.terminees:
      return source.where((Tache t) => t.faite).toList();
  }
}

/// Compare deux échéances en plaçant les tâches sans date À LA FIN.
///
/// Convention de `compareTo` : négatif = a avant b,
/// zéro = égalité, positif = a après b.
int comparerEcheances(Tache a, Tache b) {
  final DateTime? da = a.echeance;
  final DateTime? db = b.echeance;

  if (da == null && db == null) {
    return 0;
  }
  if (da == null) {
    return 1; // a n'a pas de date -> a passe après
  }
  if (db == null) {
    return -1; // b n'a pas de date -> a passe avant
  }
  return da.compareTo(db);
}

/// Renvoie une NOUVELLE liste triée. La source n'est pas touchée.
List<Tache> trier(List<Tache> source, Tri tri) {
  final List<Tache> copie = List<Tache>.from(source);

  switch (tri) {
    case Tri.echeance:
      copie.sort(comparerEcheances);
    case Tri.priorite:
      // Du plus urgent au moins urgent : on inverse b et a.
      // À priorité égale, on départage par échéance.
      copie.sort((Tache a, Tache b) {
        final int parPriorite = b.priorite.poids.compareTo(a.priorite.poids);
        if (parPriorite != 0) {
          return parPriorite;
        }
        return comparerEcheances(a, b);
      });
    case Tri.creation:
      // Les plus récentes d'abord.
      copie.sort((Tache a, Tache b) => b.creeLe.compareTo(a.creeLe));
    case Tri.alphabetique:
      // toLowerCase : sinon 'Zèbre' passerait avant 'arbre',
      // car les majuscules ont un code inférieur.
      copie.sort((Tache a, Tache b) =>
          a.titre.toLowerCase().compareTo(b.titre.toLowerCase()));
  }

  return copie;
}
```

### Le menu de tri dans la barre d'application

Dans `ecran_taches.dart` :

```dart
  Tri _tri = Tri.echeance;

  List<Tache> get _visibles => trier(filtrerParEtat(_taches, _filtre), _tri);
```

et dans l'`AppBar` :

```dart
      appBar: AppBar(
        title: const Text('Mes tâches'),
        actions: <Widget>[
          PopupMenuButton<Tri>(
            icon: const Icon(Icons.sort),
            tooltip: 'Trier',
            initialValue: _tri,
            onSelected: (Tri t) => setState(() => _tri = t),
            itemBuilder: (BuildContext _) => Tri.values
                .map((Tri t) => PopupMenuItem<Tri>(
                      value: t,
                      child: Text(t.libelle),
                    ))
                .toList(),
          ),
        ],
      ),
```

> **Remarque sur `switch` sans `break`.** Les `case` ci-dessus n'ont pas de `break` : depuis Dart 3, un `case` d'un `switch` *statement* dont le corps n'est pas vide ne « tombe » plus sur le suivant. Le `break` n'est plus nécessaire.

**État exécutable.** L'icône de tri ouvre un menu à quatre entrées ; l'ordre de la liste change immédiatement.

---

## 58.17 — La recherche

La recherche porte sur trois champs : le titre, la description et les étiquettes. Elle doit être insensible à la casse.

Ajoutez à **`lib/logique/filtrage.dart`** :

```dart
/// Ne garde que les tâches contenant `recherche`.
///
/// La comparaison se fait en minuscules, sur le titre, la
/// description et les étiquettes. Une requête vide ne filtre rien.
List<Tache> filtrerParRecherche(List<Tache> source, String recherche) {
  final String requete = recherche.trim().toLowerCase();
  if (requete.isEmpty) {
    return List<Tache>.from(source);
  }

  return source.where((Tache t) {
    if (t.titre.toLowerCase().contains(requete)) {
      return true;
    }
    if (t.description.toLowerCase().contains(requete)) {
      return true;
    }
    // `any` renvoie true dès qu'UNE étiquette correspond (ch. 14).
    return t.etiquettes.any((String e) => e.toLowerCase().contains(requete));
  }).toList();
}

/// Enchaîne les trois traitements dans le bon ordre.
///
/// Filtrer d'abord, chercher ensuite, trier en dernier : trier
/// coûte O(n log n), autant le faire sur la liste la plus courte.
List<Tache> appliquerCriteres(
  List<Tache> source, {
  required Filtre filtre,
  required Tri tri,
  String recherche = '',
}) {
  final List<Tache> parEtat = filtrerParEtat(source, filtre);
  final List<Tache> parTexte = filtrerParRecherche(parEtat, recherche);
  return trier(parTexte, tri);
}
```

Dans `ecran_taches.dart`, le champ de recherche et son contrôleur :

```dart
  final TextEditingController _controleurRecherche = TextEditingController();
  String _recherche = '';

  @override
  void dispose() {
    _controleurRecherche.dispose();
    super.dispose();
  }

  List<Tache> get _visibles => appliquerCriteres(
        _taches,
        filtre: _filtre,
        tri: _tri,
        recherche: _recherche,
      );
```

et le widget, placé au-dessus de la barre de filtres :

```dart
        Padding(
          padding: const EdgeInsets.fromLTRB(12, 12, 12, 0),
          child: TextField(
            controller: _controleurRecherche,
            // onChanged filtre à CHAQUE frappe : c'est ce qu'on veut.
            onChanged: (String v) => setState(() => _recherche = v),
            decoration: InputDecoration(
              hintText: 'Rechercher une tâche...',
              prefixIcon: const Icon(Icons.search),
              border: const OutlineInputBorder(),
              isDense: true,
              suffixIcon: _recherche.isEmpty
                  ? null
                  : IconButton(
                      icon: const Icon(Icons.clear),
                      tooltip: 'Effacer',
                      onPressed: () {
                        _controleurRecherche.clear();
                        setState(() => _recherche = '');
                      },
                    ),
            ),
          ),
        ),
```

Et un troisième état vide, pour « aucun résultat » — à placer en tête de `_vide()` :

```dart
    if (_recherche.trim().isNotEmpty) {
      return EtatVide(
        icone: Icons.search_off,
        titre: 'Aucun résultat',
        message: 'Aucune tâche ne correspond à « $_recherche ».',
      );
    }
```

**État exécutable.** Taper « bug » ne laisse que la tâche correspondante ; la croix efface la recherche.

```text
┌────────────────────────────────────────────────┐
│  ┌──────────────────────────────────────────┐  │
│  │ Q  bug                                ✕  │  │
│  └──────────────────────────────────────────┘  │
│  [•Toutes ] [ Actives ] [ Terminées ]          │
├────────────────────────────────────────────────┤
│ ▌ ☐  Corriger le bug de collision              │
│ ▌    URGENTE · il y a 3 jours  #jeu #bug       │
└────────────────────────────────────────────────┘
```

---

## 58.18 — Compteurs et barre de progression

Trois chiffres suffisent : le nombre de tâches faites, le total, et le pourcentage.

Le chapitre 14 donne l'outil : `where(...).length`, ou plus élégamment `fold`.

Ajoutez à **`lib/logique/filtrage.dart`** :

```dart
/// Nombre de tâches terminées.
int nombreTerminees(List<Tache> source) {
  return source.where((Tache t) => t.faite).length;
}

/// Nombre de tâches en retard (échéance dépassée, non faites).
int nombreEnRetard(List<Tache> source) {
  return source.where((Tache t) => t.enRetard).length;
}

/// Avancement entre 0.0 et 1.0.
///
/// Le cas de la liste vide doit être traité EXPLICITEMENT :
/// 0 / 0 vaut NaN en Dart, et `LinearProgressIndicator` afficherait
/// n'importe quoi.
double avancement(List<Tache> source) {
  if (source.isEmpty) {
    return 0;
  }
  return nombreTerminees(source) / source.length;
}
```

**`lib/widgets/barre_avancement.dart`**

```dart
import 'package:flutter/material.dart';

/// Compteur textuel et barre de progression.
class BarreAvancement extends StatelessWidget {
  const BarreAvancement({
    super.key,
    required this.terminees,
    required this.total,
  });

  final int terminees;
  final int total;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final double ratio = total == 0 ? 0 : terminees / total;
    final int pourcent = (ratio * 100).round();

    return Padding(
      padding: const EdgeInsets.fromLTRB(12, 12, 12, 4),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: <Widget>[
              Text(
                '$terminees sur $total terminée${total > 1 ? 's' : ''}',
                style: theme.textTheme.labelLarge,
              ),
              Text(
                '$pourcent %',
                style: theme.textTheme.labelLarge?.copyWith(
                  color: theme.colorScheme.primary,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ],
          ),
          const SizedBox(height: 6),
          ClipRRect(
            borderRadius: BorderRadius.circular(6),
            child: LinearProgressIndicator(
              // value entre 0.0 et 1.0. Si on passait `null`, la
              // barre deviendrait une animation indéterminée.
              value: ratio,
              minHeight: 8,
              backgroundColor: theme.colorScheme.surfaceContainerHighest,
            ),
          ),
        ],
      ),
    );
  }
}
```

Dans `_corps()`, insérez la barre entre la recherche et les filtres :

```dart
        BarreAvancement(
          terminees: nombreTerminees(_taches),
          total: _taches.length,
        ),
```

> **Remarque.** Le compteur porte sur `_taches` (toutes les tâches), pas sur `visibles`. C'est un choix : l'avancement global ne doit pas changer quand on filtre. Si vous préférez l'inverse, passez `visibles` — mais soyez cohérent, et dites-le à l'utilisateur.

**État exécutable.**

```text
│  1 sur 5 terminées                       20 %  │
│  ██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │
```

---

## 58.19 — Mettre en évidence les tâches en retard

Le getter `enRetard` existe depuis le 58.3. Il reste à le rendre visible.

Trois signaux, du plus discret au plus fort :

```text
1. le liseré gauche passe en rouge
2. la date s'écrit en rouge et en gras
3. un badge « EN RETARD » apparaît
```

Dans **`lib/widgets/tuile_tache.dart`**, remplacez le calcul de couleur et le bloc de date :

```dart
    final bool enRetard = tache.enRetard;
    final Color couleurBase = couleurPriorite(context, tache.priorite);
    // Le retard prime sur la priorité pour le liseré.
    final Color couleur = enRetard ? theme.colorScheme.error : couleurBase;
```

puis, dans la `Row` du sous-titre, remplacez la partie échéance par :

```dart
                if (tache.echeance != null) ...<Widget>[
                  const SizedBox(width: 8),
                  Icon(
                    enRetard ? Icons.warning_amber : Icons.event,
                    size: 13,
                    color: enRetard
                        ? theme.colorScheme.error
                        : theme.colorScheme.onSurfaceVariant,
                  ),
                  const SizedBox(width: 3),
                  Text(
                    dateRelative(tache.echeance!),
                    style: theme.textTheme.labelSmall?.copyWith(
                      color: enRetard ? theme.colorScheme.error : null,
                      fontWeight: enRetard ? FontWeight.bold : null,
                    ),
                  ),
                  if (enRetard) ...<Widget>[
                    const SizedBox(width: 6),
                    Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 6,
                        vertical: 1,
                      ),
                      decoration: BoxDecoration(
                        color: theme.colorScheme.errorContainer,
                        borderRadius: BorderRadius.circular(4),
                      ),
                      child: Text(
                        'EN RETARD',
                        style: theme.textTheme.labelSmall?.copyWith(
                          color: theme.colorScheme.onErrorContainer,
                          fontSize: 9,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ],
                ],
```

Enfin, un rappel global dans la barre d'avancement. Dans `_corps()` :

```dart
        if (nombreEnRetard(_taches) > 0)
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12),
            child: Row(
              children: <Widget>[
                Icon(
                  Icons.warning_amber,
                  size: 16,
                  color: Theme.of(context).colorScheme.error,
                ),
                const SizedBox(width: 6),
                Text(
                  '${nombreEnRetard(_taches)} tâche(s) en retard',
                  style: TextStyle(
                    color: Theme.of(context).colorScheme.error,
                    fontSize: 12,
                  ),
                ),
              ],
            ),
          ),
```

> **Attention.** Cocher une tâche en retard doit faire disparaître le badge : c'est déjà le cas, car `enRetard` renvoie `false` dès que `faite` vaut `true`. Vérifiez-le, c'est le test le plus rapide de cette étape.

**État exécutable.**

```text
├────────────────────────────────────────────────┤
│ ▌ ☐  Corriger le bug de collision              │
│ ▌    URGENTE ! il y a 3 jours  [EN RETARD]     │
├────────────────────────────────────────────────┤
```

---

## 58.20 — Centraliser l'état avec `ChangeNotifier` et `provider`

### Le problème

`_EtatEcranTaches` fait aujourd'hui six métiers : il détient les données, il parle au dépôt, il détient les critères de filtrage, il construit l'interface, il gère les contrôleurs et il affiche les messages. Chaque nouvelle fonctionnalité l'allonge.

Pire, le motif « je modifie, je recharge tout le dépôt, je fais un `setState` » se répète cinq fois. Et si demain un second écran (des statistiques, un widget d'accueil) a besoin des mêmes tâches, il devra tout redemander.

Le chapitre 52 a donné la réponse : sortir l'état du widget.

```text
AVANT                              APRÈS

┌───────────────────┐              ┌──────────────────────┐
│ _EtatEcranTaches  │              │ EtatTaches           │
│  - données        │              │  extends             │
│  - dépôt          │              │  ChangeNotifier      │
│  - filtre/tri     │              │  - données           │
│  - recherche      │              │  - dépôt             │
│  - construction   │              │  - filtre/tri        │
│    de l'interface │              │  - recherche         │
└───────────────────┘              └──────────┬───────────┘
                                              │ notifyListeners()
                                   ┌──────────▼───────────┐
                                   │ EcranTaches          │
                                   │  - construction      │
                                   │    de l'interface    │
                                   └──────────────────────┘
```

### Le `ChangeNotifier`

**`lib/etat/etat_taches.dart`**

```dart
import 'package:flutter/foundation.dart';
import 'package:uuid/uuid.dart';

import '../donnees/depot_taches.dart';
import '../logique/criteres.dart';
import '../logique/filtrage.dart';
import '../modeles/tache.dart';

/// Le cerveau de l'application.
///
/// Il détient LA liste de référence des tâches, les critères
/// d'affichage, et il parle au dépôt. Il n'importe PAS
/// `material.dart` : il ne connaît aucun widget. C'est la
/// séparation qui permet de le tester sans écran.
class EtatTaches extends ChangeNotifier {
  EtatTaches(this._depot);

  final DepotTaches _depot;
  final Uuid _uuid = const Uuid();

  List<Tache> _taches = <Tache>[];
  Filtre _filtre = Filtre.toutes;
  Tri _tri = Tri.echeance;
  String _recherche = '';
  bool _chargement = true;
  String? _erreur;

  // --- Lecture seule pour l'interface ---------------------------

  /// Copie non modifiable : l'interface ne doit pas pouvoir
  /// bricoler la liste dans le dos du notificateur.
  List<Tache> get toutes => List<Tache>.unmodifiable(_taches);

  /// Les tâches réellement affichées, après filtre, recherche, tri.
  List<Tache> get visibles => appliquerCriteres(
        _taches,
        filtre: _filtre,
        tri: _tri,
        recherche: _recherche,
      );

  Filtre get filtre => _filtre;
  Tri get tri => _tri;
  String get recherche => _recherche;
  bool get chargement => _chargement;
  String? get erreur => _erreur;

  int get total => _taches.length;
  int get terminees => nombreTerminees(_taches);
  int get enRetard => nombreEnRetard(_taches);
  double get progression => avancement(_taches);

  // --- Chargement ----------------------------------------------

  /// À appeler une fois au démarrage.
  Future<void> initialiser() async {
    _chargement = true;
    _erreur = null;
    notifyListeners();

    try {
      _taches = await _depot.chargerTout();
    } catch (e) {
      // Un stockage illisible ne doit pas empêcher de démarrer.
      _erreur = 'Impossible de charger les tâches : $e';
      _taches = <Tache>[];
    } finally {
      _chargement = false;
      notifyListeners();
    }
  }

  // --- Critères d'affichage ------------------------------------

  void changerFiltre(Filtre valeur) {
    if (_filtre == valeur) {
      return; // rien n'a changé : pas de reconstruction inutile
    }
    _filtre = valeur;
    notifyListeners();
  }

  void changerTri(Tri valeur) {
    if (_tri == valeur) {
      return;
    }
    _tri = valeur;
    notifyListeners();
  }

  void chercher(String texte) {
    if (_recherche == texte) {
      return;
    }
    _recherche = texte;
    notifyListeners();
  }

  // --- Écriture -------------------------------------------------

  /// Crée une tâche à partir d'un titre seul (ajout rapide).
  Future<void> ajouterTitre(String titre) async {
    await ajouter(Tache(id: _uuid.v4(), titre: titre.trim()));
  }

  Future<void> ajouter(Tache tache) async {
    _taches = <Tache>[..._taches, tache];
    notifyListeners();
    await _depot.ajouter(tache);
  }

  Future<void> modifier(Tache tache) async {
    _taches = _taches
        .map((Tache t) => t.id == tache.id ? tache : t)
        .toList();
    notifyListeners();
    await _depot.modifier(tache);
  }

  /// Coche ou décoche par identifiant.
  Future<void> basculer(String id, bool faite) async {
    final int index = _taches.indexWhere((Tache t) => t.id == id);
    if (index == -1) {
      return;
    }
    await modifier(_taches[index].copyWith(faite: faite));
  }

  /// Supprime une tâche et renvoie sa position, pour l'annulation.
  ///
  /// Renvoyer l'index évite à l'écran de le calculer lui-même :
  /// c'est le notificateur qui connaît l'ordre réel.
  Future<int> supprimer(String id) async {
    final int index = _taches.indexWhere((Tache t) => t.id == id);
    if (index == -1) {
      return -1;
    }
    _taches = <Tache>[..._taches]..removeAt(index);
    notifyListeners();
    await _depot.supprimer(id);
    return index;
  }

  /// Réinsère une tâche à sa position d'origine.
  Future<void> restaurer(Tache tache, int index) async {
    final int position = index.clamp(0, _taches.length);
    _taches = <Tache>[..._taches]..insert(position, tache);
    notifyListeners();
    await _depot.remplacerTout(_taches);
  }

  /// Supprime toutes les tâches cochées.
  Future<void> viderTerminees() async {
    _taches = _taches.where((Tache t) => !t.faite).toList();
    notifyListeners();
    await _depot.remplacerTout(_taches);
  }
}
```

Deux détails importants :

**On notifie AVANT d'attendre le dépôt.** L'interface se met à jour instantanément, l'écriture disque se fait ensuite. C'est ce qu'on appelle une mise à jour optimiste : elle rend l'application réactive. Le prix à payer est qu'en cas d'échec d'écriture, l'écran et le disque divergent — le défi 9 du 58.28 propose de traiter ce cas.

**On remplace la liste, on ne la mute pas.** `_taches = <Tache>[..._taches, tache]` crée une nouvelle liste. C'est indispensable si vous passez un jour à un `Selector` ou à une comparaison par identité : muter la liste existante rendrait l'ancienne et la nouvelle indiscernables.

### Le branchement dans `main`

**`lib/main.dart`** (version définitive, sauf le thème du 58.23)

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';
import 'package:provider/provider.dart';

import 'donnees/depot_memoire.dart';
import 'donnees/depot_taches.dart';
import 'donnees/taches_demonstration.dart';
import 'ecrans/ecran_taches.dart';
import 'etat/etat_taches.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';

  // Le dépôt est construit ICI, une seule fois. C'est le seul
  // endroit à modifier pour passer à shared_preferences (58.21)
  // ou à sqflite (58.22).
  final DepotTaches depot = DepotMemoire(
    initiales: tachesDeDemonstration(),
  );

  runApp(ApplicationTaches(depot: depot));
}

class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({super.key, required this.depot});

  final DepotTaches depot;

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<EtatTaches>(
      // `create` n'est appelé qu'une fois. On enchaîne
      // l'initialisation immédiatement : le `..` renvoie l'objet.
      create: (BuildContext _) => EtatTaches(depot)..initialiser(),
      child: MaterialApp(
        title: 'Mes tâches',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        ),
        localizationsDelegates: GlobalMaterialLocalizations.delegates,
        supportedLocales: const <Locale>[Locale('fr'), Locale('en')],
        locale: const Locale('fr'),
        home: const EcranTaches(),
      ),
    );
  }
}
```

> **Attention.** `ChangeNotifierProvider` doit être **au-dessus** de `MaterialApp`, sinon les écrans poussés par `Navigator` (qui sont des routes racine) ne le trouveront pas et lèveront `ProviderNotFoundException`.

### L'écran réécrit

**`lib/ecrans/ecran_taches.dart`** (version définitive)

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_taches.dart';
import '../logique/criteres.dart';
import '../modeles/tache.dart';
import '../widgets/barre_avancement.dart';
import '../widgets/barre_filtres.dart';
import '../widgets/etat_vide.dart';
import '../widgets/tuile_tache.dart';
import 'ecran_formulaire.dart';

/// Écran principal : la liste des tâches.
///
/// Il ne détient plus AUCUNE donnée métier. Son seul état local
/// est le contrôleur du champ de recherche, qui est un objet
/// d'interface.
class EcranTaches extends StatefulWidget {
  const EcranTaches({super.key});

  @override
  State<EcranTaches> createState() => _EtatEcranTaches();
}

class _EtatEcranTaches extends State<EcranTaches> {
  final TextEditingController _controleurRecherche = TextEditingController();

  @override
  void dispose() {
    _controleurRecherche.dispose();
    super.dispose();
  }

  Future<void> _creer() async {
    // read : on ne veut pas écouter ici, juste appeler une méthode.
    final EtatTaches etat = context.read<EtatTaches>();
    final Tache? nouvelle = await Navigator.of(context).push<Tache>(
      MaterialPageRoute<Tache>(
        builder: (BuildContext _) => const EcranFormulaire(),
      ),
    );
    if (nouvelle != null) {
      await etat.ajouter(nouvelle);
    }
  }

  Future<void> _modifier(Tache tache) async {
    final EtatTaches etat = context.read<EtatTaches>();
    final Tache? modifiee = await Navigator.of(context).push<Tache>(
      MaterialPageRoute<Tache>(
        builder: (BuildContext _) => EcranFormulaire(tacheExistante: tache),
      ),
    );
    if (modifiee != null) {
      await etat.modifier(modifiee);
    }
  }

  Future<void> _supprimer(Tache tache) async {
    final EtatTaches etat = context.read<EtatTaches>();
    final ScaffoldMessengerState messager = ScaffoldMessenger.of(context);

    final int index = await etat.supprimer(tache.id);
    if (index == -1) {
      return;
    }

    messager.hideCurrentSnackBar();
    messager.showSnackBar(
      SnackBar(
        content: Text('« ${tache.titre} » supprimée'),
        duration: const Duration(seconds: 5),
        action: SnackBarAction(
          label: 'ANNULER',
          onPressed: () => etat.restaurer(tache, index),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // watch : ce build sera rejoué à chaque notifyListeners().
    final EtatTaches etat = context.watch<EtatTaches>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Mes tâches'),
        actions: <Widget>[
          PopupMenuButton<Tri>(
            icon: const Icon(Icons.sort),
            tooltip: 'Trier',
            initialValue: etat.tri,
            onSelected: etat.changerTri,
            itemBuilder: (BuildContext _) => Tri.values
                .map((Tri t) => PopupMenuItem<Tri>(
                      value: t,
                      child: Text(t.libelle),
                    ))
                .toList(),
          ),
          IconButton(
            icon: const Icon(Icons.cleaning_services_outlined),
            tooltip: 'Supprimer les tâches terminées',
            onPressed: etat.terminees == 0 ? null : _confirmerVidage,
          ),
        ],
      ),
      body: _corps(etat),
      floatingActionButton: FloatingActionButton(
        onPressed: _creer,
        tooltip: 'Ajouter une tâche',
        child: const Icon(Icons.add),
      ),
    );
  }

  Future<void> _confirmerVidage() async {
    final EtatTaches etat = context.read<EtatTaches>();
    final bool? oui = await showDialog<bool>(
      context: context,
      builder: (BuildContext ctx) => AlertDialog(
        title: const Text('Supprimer les tâches terminées ?'),
        content: Text('${etat.terminees} tâche(s) seront supprimées. '
            'Cette action est définitive.'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(ctx).pop(false),
            child: const Text('Annuler'),
          ),
          FilledButton(
            onPressed: () => Navigator.of(ctx).pop(true),
            child: const Text('Supprimer'),
          ),
        ],
      ),
    );
    if (oui ?? false) {
      await etat.viderTerminees();
    }
  }

  Widget _corps(EtatTaches etat) {
    if (etat.chargement) {
      return const Center(child: CircularProgressIndicator());
    }

    final List<Tache> visibles = etat.visibles;

    return Column(
      children: <Widget>[
        Padding(
          padding: const EdgeInsets.fromLTRB(12, 12, 12, 0),
          child: TextField(
            controller: _controleurRecherche,
            onChanged: etat.chercher,
            decoration: InputDecoration(
              hintText: 'Rechercher une tâche...',
              prefixIcon: const Icon(Icons.search),
              border: const OutlineInputBorder(),
              isDense: true,
              suffixIcon: etat.recherche.isEmpty
                  ? null
                  : IconButton(
                      icon: const Icon(Icons.clear),
                      tooltip: 'Effacer',
                      onPressed: () {
                        _controleurRecherche.clear();
                        etat.chercher('');
                      },
                    ),
            ),
          ),
        ),
        BarreAvancement(terminees: etat.terminees, total: etat.total),
        if (etat.enRetard > 0)
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12),
            child: Row(
              children: <Widget>[
                Icon(
                  Icons.warning_amber,
                  size: 16,
                  color: Theme.of(context).colorScheme.error,
                ),
                const SizedBox(width: 6),
                Text(
                  '${etat.enRetard} tâche(s) en retard',
                  style: TextStyle(
                    color: Theme.of(context).colorScheme.error,
                    fontSize: 12,
                  ),
                ),
              ],
            ),
          ),
        Padding(
          padding: const EdgeInsets.fromLTRB(12, 12, 12, 8),
          child: BarreFiltres(
            filtre: etat.filtre,
            onChange: etat.changerFiltre,
          ),
        ),
        Expanded(
          child: visibles.isEmpty
              ? _vide(etat)
              : ListView.builder(
                  itemCount: visibles.length,
                  itemBuilder: (BuildContext context, int index) {
                    final Tache tache = visibles[index];
                    return Dismissible(
                      key: ValueKey<String>(tache.id),
                      direction: DismissDirection.endToStart,
                      background: _fondSuppression(context),
                      // Plus besoin de chercher un index :
                      // le notificateur travaille par identifiant.
                      onDismissed: (DismissDirection _) => _supprimer(tache),
                      child: TuileTache(
                        tache: tache,
                        onBascule: (bool faite) =>
                            etat.basculer(tache.id, faite),
                        onAppui: () => _modifier(tache),
                      ),
                    );
                  },
                ),
        ),
      ],
    );
  }

  Widget _fondSuppression(BuildContext context) {
    return Container(
      color: Theme.of(context).colorScheme.errorContainer,
      alignment: Alignment.centerRight,
      padding: const EdgeInsets.symmetric(horizontal: 24),
      child: Icon(
        Icons.delete_outline,
        color: Theme.of(context).colorScheme.onErrorContainer,
      ),
    );
  }

  Widget _vide(EtatTaches etat) {
    if (etat.recherche.trim().isNotEmpty) {
      return EtatVide(
        icone: Icons.search_off,
        titre: 'Aucun résultat',
        message: 'Aucune tâche ne correspond à « ${etat.recherche} ».',
      );
    }
    if (etat.total == 0) {
      return const EtatVide(
        icone: Icons.check_circle_outline,
        titre: 'Aucune tâche pour l\'instant',
        message: 'Appuyez sur + pour créer votre première tâche.',
      );
    }
    if (etat.filtre == Filtre.terminees) {
      return const EtatVide(
        icone: Icons.hourglass_empty,
        titre: 'Aucune tâche terminée',
        message: 'Cochez une tâche pour la voir apparaître ici.',
      );
    }
    return const EtatVide(
      icone: Icons.celebration_outlined,
      titre: 'Tout est terminé',
      message: 'Vous n\'avez plus rien à faire. Profitez-en.',
    );
  }
}
```

### `watch` ou `read` ?

| Méthode | Effet | À utiliser |
| --- | --- | --- |
| `context.watch<T>()` | lit ET s'abonne : le `build` sera rejoué | dans un `build`, pour afficher |
| `context.read<T>()` | lit sans s'abonner | dans un rappel (`onPressed`, `onTap`), pour agir |
| `context.select<T, R>()` | s'abonne à une seule propriété | quand un `build` coûteux ne dépend que d'un champ |

L'erreur classique est d'appeler `context.watch` dans un `onPressed` : le linter de `provider` lève alors une exception explicite. L'erreur inverse — `read` dans un `build` — ne lève rien mais fige l'écran, qui ne se met plus à jour. C'est la panne la plus fréquente chez les débutants avec `provider`.

**État exécutable.** L'application se comporte exactement comme avant. Aucune fonctionnalité gagnée, mais l'écran a perdu la moitié de son code et la logique est désormais testable seule.

---

## 58.21 — Persister avec `shared_preferences`

Fermez l'application, rouvrez-la : tout est perdu. Réglons cela.

Le chapitre 54 a présenté `shared_preferences` : un magasin clé/valeur pour de petites données. Il accepte `int`, `double`, `bool`, `String` et `List<String>`. Une liste de tâches n'en fait pas partie — mais une liste de tâches **encodée en JSON** est une `String`.

```text
List<Tache>
   │  map(toJson) + jsonEncode
   ▼
String  '[{"id":"a1",...},{"id":"a2",...}]'
   │  prefs.setString('taches', ...)
   ▼
─────────── disque de l'appareil ───────────
```

**`lib/donnees/depot_prefs.dart`**

```dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

import '../modeles/tache.dart';
import 'depot_taches.dart';

/// Dépôt persistant fondé sur `shared_preferences`.
///
/// Stratégie : on garde une copie en mémoire (rapide) et on
/// réécrit l'intégralité du JSON après chaque modification.
/// C'est parfaitement acceptable jusqu'à quelques centaines de
/// tâches. Au-delà, passez à `sqflite` (58.22).
class DepotPrefs implements DepotTaches {
  DepotPrefs._(this._prefs, this._cache);

  /// Constructeur asynchrone : un constructeur normal ne peut pas
  /// attendre. On passe donc par une fabrique statique.
  static Future<DepotPrefs> ouvrir() async {
    final SharedPreferencesAsync prefs = SharedPreferencesAsync();
    final List<Tache> cache = await _lire(prefs);
    return DepotPrefs._(prefs, cache);
  }

  static const String _cle = 'taches_v1';

  final SharedPreferencesAsync _prefs;
  final List<Tache> _cache;

  static Future<List<Tache>> _lire(SharedPreferencesAsync prefs) async {
    final String? brut = await prefs.getString(_cle);
    if (brut == null || brut.isEmpty) {
      return <Tache>[];
    }
    try {
      final List<dynamic> liste = jsonDecode(brut) as List<dynamic>;
      return liste
          .map((dynamic e) => Tache.fromJson(e as Map<String, dynamic>))
          .toList();
    } catch (e) {
      // JSON corrompu, ou format d'une version antérieure.
      // On repart d'une liste vide plutôt que de planter.
      return <Tache>[];
    }
  }

  Future<void> _ecrire() async {
    final String brut = jsonEncode(
      _cache.map((Tache t) => t.toJson()).toList(),
    );
    await _prefs.setString(_cle, brut);
  }

  @override
  Future<List<Tache>> chargerTout() async => List<Tache>.from(_cache);

  @override
  Future<void> ajouter(Tache tache) async {
    _cache.add(tache);
    await _ecrire();
  }

  @override
  Future<void> modifier(Tache tache) async {
    final int i = _cache.indexWhere((Tache t) => t.id == tache.id);
    if (i != -1) {
      _cache[i] = tache;
      await _ecrire();
    }
  }

  @override
  Future<void> supprimer(String id) async {
    _cache.removeWhere((Tache t) => t.id == id);
    await _ecrire();
  }

  @override
  Future<void> remplacerTout(List<Tache> taches) async {
    _cache
      ..clear()
      ..addAll(taches);
    await _ecrire();
  }
}
```

### Le branchement : une ligne

Dans `main.dart` :

```dart
  // AVANT
  // final DepotTaches depot = DepotMemoire(initiales: tachesDeDemonstration());

  // APRÈS
  final DepotTaches depot = await DepotPrefs.ouvrir();
```

avec `import 'donnees/depot_prefs.dart';`.

**C'est tout.** Aucune ligne de `EtatTaches`, d'`EcranTaches` ou de `TuileTache` ne change. C'est le bénéfice exact de l'interface définie au 58.5.

### Amorcer la première utilisation

Une première ouverture affichera une liste vide. Pour conserver les tâches de démonstration au tout premier lancement :

```dart
  final DepotTaches depot = await DepotPrefs.ouvrir();
  if ((await depot.chargerTout()).isEmpty) {
    await depot.remplacerTout(tachesDeDemonstration());
  }
```

> **Remarque sur les trois API.** `shared_preferences` propose `SharedPreferences` (l'API historique, avec `SharedPreferences.getInstance()` et des lectures synchrones — vouée à la dépréciation), `SharedPreferencesAsync` (tout est asynchrone, aucun cache) et `SharedPreferencesWithCache` (lectures synchrones après une préparation). Nous utilisons `SharedPreferencesAsync`, l'API recommandée aujourd'hui pour du code neuf. Si votre code existant utilise `SharedPreferences.getInstance()`, il fonctionne toujours ; la conversion se résume à remplacer `prefs.getString(k)` par `await prefs.getString(k)`.

**État exécutable.** Créez trois tâches, fermez complètement l'application, relancez-la : elles sont là.

---

## 58.22 — Variante : persister avec `sqflite`

`shared_preferences` réécrit tout le fichier à chaque modification. Avec 5 000 tâches, cocher une case réécrirait 5 000 tâches. Une vraie base de données ne réécrit qu'une ligne, et sait chercher, filtrer et trier en SQL.

### La table

SQLite ne connaît que `INTEGER`, `REAL`, `TEXT` et `BLOB`. Il faut donc traduire :

```text
Dart              SQLite
--------------    ---------------------------------
String            TEXT
bool              INTEGER  (0 ou 1)
DateTime          TEXT     (ISO 8601, triable tel quel)
Priorite          TEXT     (le .name)
List<String>      TEXT     (JSON)
```

Le format ISO 8601 a une vertu précieuse : l'ordre alphabétique des chaînes coïncide avec l'ordre chronologique. `ORDER BY echeance` fonctionne donc directement.

**`lib/donnees/depot_sqflite.dart`**

```dart
import 'dart:convert';

import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

import '../modeles/priorite.dart';
import '../modeles/tache.dart';
import 'depot_taches.dart';

/// Dépôt persistant fondé sur SQLite.
///
/// Chaque opération touche UNE ligne, pas tout le fichier.
class DepotSqflite implements DepotTaches {
  DepotSqflite._(this._db);

  static const String _table = 'taches';

  final Database _db;

  /// Ouvre (et crée si besoin) la base de données.
  static Future<DepotSqflite> ouvrir() async {
    // getDatabasesPath renvoie le dossier réservé aux bases de
    // données de l'application. `p.join` construit un chemin
    // valable sur toutes les plateformes.
    final String chemin = p.join(await getDatabasesPath(), 'taches.db');

    final Database db = await openDatabase(
      chemin,
      version: 1,
      onCreate: (Database db, int version) async {
        await db.execute('''
          CREATE TABLE $_table (
            id TEXT PRIMARY KEY,
            titre TEXT NOT NULL,
            description TEXT NOT NULL DEFAULT '',
            faite INTEGER NOT NULL DEFAULT 0,
            priorite TEXT NOT NULL DEFAULT 'normale',
            echeance TEXT,
            etiquettes TEXT NOT NULL DEFAULT '[]',
            cree_le TEXT NOT NULL
          )
        ''');
        // Un index accélère les tris et filtres sur l'échéance.
        await db.execute(
          'CREATE INDEX idx_echeance ON $_table (echeance)',
        );
      },
    );

    return DepotSqflite._(db);
  }

  /// Conversion Tache -> ligne SQL.
  Map<String, Object?> _versLigne(Tache t) {
    return <String, Object?>{
      'id': t.id,
      'titre': t.titre,
      'description': t.description,
      // SQLite n'a pas de booléen : 1 ou 0.
      'faite': t.faite ? 1 : 0,
      'priorite': t.priorite.name,
      'echeance': t.echeance?.toIso8601String(),
      'etiquettes': jsonEncode(t.etiquettes.toList()),
      'cree_le': t.creeLe.toIso8601String(),
    };
  }

  /// Conversion ligne SQL -> Tache.
  Tache _depuisLigne(Map<String, Object?> l) {
    final Object? echeance = l['echeance'];
    final Object? etiquettes = l['etiquettes'];

    return Tache(
      id: l['id']! as String,
      titre: l['titre']! as String,
      description: (l['description'] as String?) ?? '',
      faite: (l['faite'] as int? ?? 0) == 1,
      priorite: Priorite.depuisNom(l['priorite'] as String?),
      echeance: echeance is String ? DateTime.tryParse(echeance) : null,
      etiquettes: etiquettes is String
          ? (jsonDecode(etiquettes) as List<dynamic>)
              .map((dynamic e) => e.toString())
              .toList()
          : const <String>[],
      creeLe: DateTime.tryParse((l['cree_le'] as String?) ?? ''),
    );
  }

  @override
  Future<List<Tache>> chargerTout() async {
    final List<Map<String, Object?>> lignes = await _db.query(_table);
    return lignes.map(_depuisLigne).toList();
  }

  @override
  Future<void> ajouter(Tache tache) async {
    await _db.insert(
      _table,
      _versLigne(tache),
      // Si l'identifiant existe déjà, on écrase au lieu de lever
      // une erreur de contrainte.
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  @override
  Future<void> modifier(Tache tache) async {
    await _db.update(
      _table,
      _versLigne(tache),
      // Le `?` est un paramètre lié. N'écrivez JAMAIS
      // "id = '${tache.id}'" : c'est une injection SQL.
      where: 'id = ?',
      whereArgs: <Object?>[tache.id],
    );
  }

  @override
  Future<void> supprimer(String id) async {
    await _db.delete(_table, where: 'id = ?', whereArgs: <Object?>[id]);
  }

  @override
  Future<void> remplacerTout(List<Tache> taches) async {
    // Un batch envoie toutes les requêtes en un seul aller-retour.
    final Batch lot = _db.batch();
    lot.delete(_table);
    for (final Tache t in taches) {
      lot.insert(
        _table,
        _versLigne(t),
        conflictAlgorithm: ConflictAlgorithm.replace,
      );
    }
    await lot.commit(noResult: true);
  }

  /// Bonus : filtrer côté base plutôt qu'en Dart.
  ///
  /// Sur 100 000 lignes, cette requête est des ordres de grandeur
  /// plus rapide que `where` sur une liste chargée en mémoire.
  Future<List<Tache>> chargerActivesParEcheance() async {
    final List<Map<String, Object?>> lignes = await _db.query(
      _table,
      where: 'faite = ?',
      whereArgs: <Object?>[0],
      // Grâce à l'ISO 8601, l'ordre textuel est l'ordre du temps.
      orderBy: 'echeance IS NULL, echeance ASC',
    );
    return lignes.map(_depuisLigne).toList();
  }

  Future<void> fermer() => _db.close();
}
```

Le branchement, toujours une ligne dans `main.dart` :

```dart
  final DepotTaches depot = await DepotSqflite.ouvrir();
```

### Choisir entre les deux

| Critère | `shared_preferences` | `sqflite` |
| --- | --- | --- |
| Mise en place | 10 lignes | 60 lignes |
| Volume raisonnable | jusqu'à ~500 objets | des centaines de milliers |
| Coût d'une modification | réécrit tout | une ligne |
| Recherche et tri | en Dart, en mémoire | en SQL, indexé |
| Évolution du format | à votre charge | migrations `onUpgrade` |
| Web | oui | non sans `sqflite_common_ffi_web` |

Pour ce projet, `shared_preferences` suffit largement. Écrire la variante `sqflite` reste un excellent exercice : elle prouve que votre architecture est saine, puisque changer de base ne touche qu'un fichier.

> **Attention.** `getDatabasesPath()` n'existe pas sur le web ni sur les bureaux sans `sqflite_common_ffi`. Si vous développez sur Windows, macOS ou Linux, ajoutez `sqflite_common_ffi` et appelez `databaseFactory = databaseFactoryFfi;` avant l'ouverture. Sur Android et iOS, rien à faire.

**État exécutable.** Avec `DepotSqflite`, l'application se comporte à l'identique. Les données vivent maintenant dans un fichier `taches.db`.

---

## 58.23 — Le mode sombre

Le chapitre 51 a montré que `MaterialApp` accepte trois paramètres complémentaires :

```text
theme:      le thème clair
darkTheme:  le thème sombre
themeMode:  lequel utiliser -> system | light | dark
```

Avec `ThemeMode.system`, l'application suit le réglage de l'appareil. Nous voulons en plus un interrupteur manuel, dont le choix survit au redémarrage. C'est donc un second état persistant : un second `ChangeNotifier`.

**`lib/etat/etat_theme.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

/// Détient le mode de thème choisi et le persiste.
///
/// On stocke le `name` de l'énumération ('system', 'light',
/// 'dark') : lisible, stable, et insensible à un changement
/// d'ordre des valeurs.
class EtatTheme extends ChangeNotifier {
  EtatTheme({ThemeMode initial = ThemeMode.system}) : _mode = initial;

  static const String _cle = 'mode_theme';

  ThemeMode _mode;

  ThemeMode get mode => _mode;

  /// `true` si l'application est actuellement en sombre forcé.
  bool get estSombre => _mode == ThemeMode.dark;

  /// Lit le mode enregistré au démarrage.
  static Future<EtatTheme> charger() async {
    final SharedPreferencesAsync prefs = SharedPreferencesAsync();
    final String? nom = await prefs.getString(_cle);

    for (final ThemeMode m in ThemeMode.values) {
      if (m.name == nom) {
        return EtatTheme(initial: m);
      }
    }
    return EtatTheme();
  }

  Future<void> changer(ThemeMode nouveau) async {
    if (_mode == nouveau) {
      return;
    }
    _mode = nouveau;
    notifyListeners();
    await SharedPreferencesAsync().setString(_cle, nouveau.name);
  }

  /// Bascule clair <-> sombre depuis un simple bouton.
  Future<void> basculer() {
    return changer(estSombre ? ThemeMode.light : ThemeMode.dark);
  }
}
```

### Les deux thèmes

Une seule couleur de départ suffit : `ColorScheme.fromSeed` fabrique les deux palettes cohérentes.

**`lib/utilitaires/themes.dart`**

```dart
import 'package:flutter/material.dart';

const Color _graine = Colors.indigo;

ThemeData themeClair() {
  return ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: _graine,
      brightness: Brightness.light,
    ),
    // Des tuiles un peu plus compactes que la valeur par défaut.
    listTileTheme: const ListTileThemeData(
      contentPadding: EdgeInsets.symmetric(horizontal: 12),
    ),
  );
}

ThemeData themeSombre() {
  return ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: _graine,
      // C'est CETTE ligne qui produit toute la palette sombre.
      brightness: Brightness.dark,
    ),
    listTileTheme: const ListTileThemeData(
      contentPadding: EdgeInsets.symmetric(horizontal: 12),
    ),
  );
}
```

### Le `main.dart` définitif

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';
import 'package:provider/provider.dart';

import 'donnees/depot_prefs.dart';
import 'donnees/depot_taches.dart';
import 'donnees/taches_demonstration.dart';
import 'ecrans/ecran_taches.dart';
import 'etat/etat_taches.dart';
import 'etat/etat_theme.dart';
import 'utilitaires/themes.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';

  // Un seul endroit décide du stockage.
  final DepotTaches depot = await DepotPrefs.ouvrir();
  if ((await depot.chargerTout()).isEmpty) {
    await depot.remplacerTout(tachesDeDemonstration());
  }

  final EtatTheme theme = await EtatTheme.charger();

  runApp(ApplicationTaches(depot: depot, theme: theme));
}

class ApplicationTaches extends StatelessWidget {
  const ApplicationTaches({
    super.key,
    required this.depot,
    required this.theme,
  });

  final DepotTaches depot;
  final EtatTheme theme;

  @override
  Widget build(BuildContext context) {
    // MultiProvider évite d'imbriquer les providers les uns
    // dans les autres.
    return MultiProvider(
      providers: <SingleChildWidget>[
        ChangeNotifierProvider<EtatTaches>(
          create: (BuildContext _) => EtatTaches(depot)..initialiser(),
        ),
        // value : l'objet existe DÉJÀ, on ne le crée pas ici.
        ChangeNotifierProvider<EtatTheme>.value(value: theme),
      ],
      child: const _RacineMaterial(),
    );
  }
}

/// Widget séparé pour que `watch<EtatTheme>()` trouve le provider :
/// un `context` ne voit que les providers situés AU-DESSUS de lui.
class _RacineMaterial extends StatelessWidget {
  const _RacineMaterial();

  @override
  Widget build(BuildContext context) {
    final EtatTheme theme = context.watch<EtatTheme>();

    return MaterialApp(
      title: 'Mes tâches',
      debugShowCheckedModeBanner: false,
      theme: themeClair(),
      darkTheme: themeSombre(),
      themeMode: theme.mode,
      localizationsDelegates: GlobalMaterialLocalizations.delegates,
      supportedLocales: const <Locale>[Locale('fr'), Locale('en')],
      locale: const Locale('fr'),
      home: const EcranTaches(),
    );
  }
}
```

`MultiProvider` et `SingleChildWidget` viennent de `provider` ; l'import `package:provider/provider.dart` suffit, `SingleChildWidget` étant réexporté.

### L'interrupteur

Dans les `actions` de l'`AppBar` de `ecran_taches.dart`, en première position :

```dart
          IconButton(
            icon: Icon(
              context.watch<EtatTheme>().estSombre
                  ? Icons.light_mode_outlined
                  : Icons.dark_mode_outlined,
            ),
            tooltip: 'Changer de thème',
            onPressed: () => context.read<EtatTheme>().basculer(),
          ),
```

avec `import '../etat/etat_theme.dart';`.

> **Attention.** Toutes les couleurs écrites en dur dans ce projet viennent de `Theme.of(context).colorScheme`, sauf une : le `Colors.orange` de `couleurPriorite`. C'est volontaire — l'orange reste lisible sur les deux fonds. En revanche, si vous aviez écrit `Colors.black` quelque part, le mode sombre l'aurait rendu invisible. C'est le test qui révèle un thème mal fait.

**État exécutable.** L'icône de lune bascule toute l'application en sombre. Fermez, rouvrez : le choix est conservé.

```text
┌────────────────────────────────────────────────┐
│  Mes tâches            [T]  [tri]  [nettoyer]  │
├────────────────────────────────────────────────┤
│  (fond sombre, texte clair, liserés colorés)   │
└────────────────────────────────────────────────┘
```

---

## 58.24 — Tester la logique de filtrage

Toute la logique métier vit dans `lib/logique/filtrage.dart`, qui n'importe pas Flutter. On peut donc la tester sans lancer d'application, en quelques millisecondes.

Rappel du vocabulaire :

```text
test('description', () { ... })    un cas de test
group('titre', () { ... })         un regroupement
expect(obtenu, attendu)            l'assertion
setUp(() { ... })                  exécuté avant CHAQUE test
```

**`test/filtrage_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:liste_taches/logique/criteres.dart';
import 'package:liste_taches/logique/filtrage.dart';
import 'package:liste_taches/modeles/priorite.dart';
import 'package:liste_taches/modeles/tache.dart';

/// Fabrique de tâches pour les tests.
///
/// Elle donne des valeurs par défaut à tout, pour que chaque test
/// ne précise QUE ce qui l'intéresse. Un test doit se lire d'un
/// coup d'œil.
Tache t({
  required String id,
  String titre = 'Tâche',
  String description = '',
  bool faite = false,
  Priorite priorite = Priorite.normale,
  DateTime? echeance,
  List<String> etiquettes = const <String>[],
  DateTime? creeLe,
}) {
  return Tache(
    id: id,
    titre: titre,
    description: description,
    faite: faite,
    priorite: priorite,
    echeance: echeance,
    etiquettes: etiquettes,
    creeLe: creeLe ?? DateTime(2026, 1, 1),
  );
}

void main() {
  group('filtrerParEtat', () {
    late List<Tache> source;

    setUp(() {
      source = <Tache>[
        t(id: '1', titre: 'A', faite: false),
        t(id: '2', titre: 'B', faite: true),
        t(id: '3', titre: 'C', faite: false),
      ];
    });

    test('Filtre.toutes renvoie tout', () {
      expect(filtrerParEtat(source, Filtre.toutes).length, 3);
    });

    test('Filtre.actives ne garde que les non faites', () {
      final List<Tache> r = filtrerParEtat(source, Filtre.actives);
      expect(r.length, 2);
      expect(r.every((Tache x) => !x.faite), isTrue);
    });

    test('Filtre.terminees ne garde que les faites', () {
      final List<Tache> r = filtrerParEtat(source, Filtre.terminees);
      expect(r.length, 1);
      expect(r.first.id, '2');
    });

    test('ne modifie pas la liste source', () {
      filtrerParEtat(source, Filtre.actives);
      expect(source.length, 3);
    });

    test('une liste vide reste vide', () {
      expect(filtrerParEtat(<Tache>[], Filtre.actives), isEmpty);
    });
  });

  group('filtrerParRecherche', () {
    final List<Tache> source = <Tache>[
      t(id: '1', titre: 'Corriger le bug de collision',
          etiquettes: <String>['jeu']),
      t(id: '2', titre: 'Dessiner le sprite',
          description: 'Huit images par cycle.'),
      t(id: '3', titre: 'Musique', etiquettes: <String>['audio']),
    ];

    test('une requête vide ne filtre rien', () {
      expect(filtrerParRecherche(source, '').length, 3);
      expect(filtrerParRecherche(source, '   ').length, 3);
    });

    test('cherche dans le titre', () {
      expect(filtrerParRecherche(source, 'bug').single.id, '1');
    });

    test('est insensible à la casse', () {
      expect(filtrerParRecherche(source, 'BUG').single.id, '1');
      expect(filtrerParRecherche(source, 'BuG').single.id, '1');
    });

    test('cherche dans la description', () {
      expect(filtrerParRecherche(source, 'images').single.id, '2');
    });

    test('cherche dans les étiquettes', () {
      expect(filtrerParRecherche(source, 'audio').single.id, '3');
    });

    test('renvoie une liste vide si rien ne correspond', () {
      expect(filtrerParRecherche(source, 'zzz'), isEmpty);
    });
  });

  group('trier', () {
    test('par échéance, les sans-date à la fin', () {
      final List<Tache> source = <Tache>[
        t(id: 'sans'),
        t(id: 'tard', echeance: DateTime(2026, 12, 1)),
        t(id: 'tot', echeance: DateTime(2026, 1, 5)),
      ];

      final List<String> ids =
          trier(source, Tri.echeance).map((Tache x) => x.id).toList();

      expect(ids, <String>['tot', 'tard', 'sans']);
    });

    test('par priorité, du plus urgent au moins urgent', () {
      final List<Tache> source = <Tache>[
        t(id: 'b', priorite: Priorite.basse),
        t(id: 'u', priorite: Priorite.urgente),
        t(id: 'n', priorite: Priorite.normale),
        t(id: 'h', priorite: Priorite.haute),
      ];

      final List<String> ids =
          trier(source, Tri.priorite).map((Tache x) => x.id).toList();

      expect(ids, <String>['u', 'h', 'n', 'b']);
    });

    test('par priorité, l\'échéance départage les ex aequo', () {
      final List<Tache> source = <Tache>[
        t(id: 'tard', priorite: Priorite.haute,
            echeance: DateTime(2026, 6, 1)),
        t(id: 'tot', priorite: Priorite.haute,
            echeance: DateTime(2026, 2, 1)),
      ];

      expect(trier(source, Tri.priorite).first.id, 'tot');
    });

    test('alphabétique, insensible à la casse', () {
      final List<Tache> source = <Tache>[
        t(id: '1', titre: 'Zèbre'),
        t(id: '2', titre: 'arbre'),
      ];

      expect(trier(source, Tri.alphabetique).first.titre, 'arbre');
    });

    test('ne modifie pas la liste source', () {
      final List<Tache> source = <Tache>[
        t(id: 'b', titre: 'B'),
        t(id: 'a', titre: 'A'),
      ];
      trier(source, Tri.alphabetique);
      expect(source.first.id, 'b');
    });
  });

  group('compteurs', () {
    test('avancement d\'une liste vide vaut 0, pas NaN', () {
      expect(avancement(<Tache>[]), 0);
    });

    test('avancement 1 sur 4 vaut 0.25', () {
      final List<Tache> source = <Tache>[
        t(id: '1', faite: true),
        t(id: '2'),
        t(id: '3'),
        t(id: '4'),
      ];
      expect(avancement(source), 0.25);
      expect(nombreTerminees(source), 1);
    });

    test('une tâche faite n\'est jamais en retard', () {
      final List<Tache> source = <Tache>[
        t(id: '1', faite: true, echeance: DateTime(2000, 1, 1)),
      ];
      expect(nombreEnRetard(source), 0);
    });

    test('une échéance passée non faite est en retard', () {
      final List<Tache> source = <Tache>[
        t(id: '1', echeance: DateTime(2000, 1, 1)),
      ];
      expect(nombreEnRetard(source), 1);
    });
  });

  group('appliquerCriteres', () {
    test('combine filtre, recherche et tri', () {
      final List<Tache> source = <Tache>[
        t(id: '1', titre: 'Bug de collision', faite: true),
        t(id: '2', titre: 'Bug de son', priorite: Priorite.basse),
        t(id: '3', titre: 'Bug de sauvegarde', priorite: Priorite.urgente),
        t(id: '4', titre: 'Musique'),
      ];

      final List<Tache> r = appliquerCriteres(
        source,
        filtre: Filtre.actives,
        tri: Tri.priorite,
        recherche: 'bug',
      );

      // '1' est écartée (faite), '4' est écartée (pas « bug »).
      // Reste '3' (urgente) puis '2' (basse).
      expect(r.map((Tache x) => x.id).toList(), <String>['3', '2']);
    });
  });

  group('sérialisation', () {
    test('l\'aller-retour JSON conserve les données', () {
      final Tache origine = t(
        id: 'a1',
        titre: 'Test',
        description: 'Une description',
        priorite: Priorite.urgente,
        echeance: DateTime(2026, 8, 12),
        etiquettes: <String>['jeu', 'bug'],
      );

      final Tache relue = Tache.fromJson(origine.toJson());

      expect(relue.id, origine.id);
      expect(relue.titre, origine.titre);
      expect(relue.description, origine.description);
      expect(relue.priorite, origine.priorite);
      expect(relue.echeance, origine.echeance);
      expect(relue.etiquettes, origine.etiquettes);
    });

    test('un JSON incomplet ne fait pas planter', () {
      final Tache relue = Tache.fromJson(<String, dynamic>{
        'id': 'x',
        'titre': 'Minimal',
      });

      expect(relue.description, '');
      expect(relue.faite, isFalse);
      expect(relue.priorite, Priorite.normale);
      expect(relue.echeance, isNull);
      expect(relue.etiquettes, isEmpty);
    });

    test('une priorité inconnue retombe sur normale', () {
      expect(Priorite.depuisNom('extraterrestre'), Priorite.normale);
      expect(Priorite.depuisNom(null), Priorite.normale);
    });
  });
}
```

Lancez :

```text
flutter test
```

**Résultat :**

```text
00:02 +26: All tests passed!
```

> **Remarque.** Les imports commencent par `package:liste_taches/...` et non par des chemins relatifs. Dans le dossier `test/`, un chemin relatif vers `lib/` fonctionne parfois mais crée deux copies distinctes des mêmes classes, et les comparaisons de types échouent mystérieusement. Utilisez toujours la forme `package:`.

**État exécutable.** 26 tests passent. Modifiez volontairement `comparerEcheances` pour renvoyer `-1` au lieu de `1` : deux tests virent au rouge instantanément. C'est exactement ce à quoi sert un test.

---

## 58.25 — Récapitulatif de l'arborescence finale

```text
liste_taches/
├── pubspec.yaml
├── lib/
│   ├── main.dart                         58.23
│   ├── modeles/
│   │   ├── priorite.dart                 58.2
│   │   └── tache.dart                    58.3, 58.4
│   ├── logique/
│   │   ├── criteres.dart                 58.15
│   │   └── filtrage.dart                 58.15, 58.16, 58.17, 58.18
│   ├── donnees/
│   │   ├── depot_taches.dart             58.5
│   │   ├── depot_memoire.dart            58.5
│   │   ├── taches_demonstration.dart     58.5
│   │   ├── depot_prefs.dart              58.21
│   │   └── depot_sqflite.dart            58.22
│   ├── etat/
│   │   ├── etat_taches.dart              58.20
│   │   └── etat_theme.dart               58.23
│   ├── ecrans/
│   │   ├── ecran_taches.dart             58.20
│   │   └── ecran_formulaire.dart         58.11
│   ├── widgets/
│   │   ├── tuile_tache.dart              58.12, 58.19
│   │   ├── barre_filtres.dart            58.15
│   │   ├── barre_avancement.dart         58.18
│   │   ├── etat_vide.dart                58.7
│   │   └── dialogue_ajout_rapide.dart    58.9
│   └── utilitaires/
│       ├── dates.dart                    58.10
│       ├── couleurs.dart                 58.8
│       └── themes.dart                   58.23
└── test/
    └── filtrage_test.dart                58.24
```

Le sens de dépendance, qui ne doit jamais s'inverser :

```text
  ecrans/  widgets/          <- connaissent Flutter
      │
      ▼
    etat/                    <- ne connaît que foundation
      │
      ├──────────────┐
      ▼              ▼
  logique/       donnees/    <- ne connaissent PAS Flutter
      │              │
      └──────┬───────┘
             ▼
         modeles/            <- ne connaît rien
```

Une flèche qui remonterait (un modèle qui importerait un écran) serait le signe d'une erreur d'architecture.

---

## 58.26 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `A dismissed Dismissible widget is still part of the tree.` | La donnée n'a pas été retirée de la liste dans `onDismissed`. | Supprimer la tâche de l'état AVANT le prochain `build`. |
| Une mauvaise tuile disparaît après un glissement. | `key: ValueKey(index)` au lieu de l'identifiant. | `key: ValueKey<String>(tache.id)`. |
| `Vertical viewport was given unbounded height.` | Une `ListView` dans une `Column` sans contrainte. | Envelopper la `ListView` dans un `Expanded` (chapitre 46). |
| L'écran ne se met plus à jour. | `context.read` utilisé dans un `build`. | Utiliser `context.watch` pour lire dans un `build`. |
| `Tried to use Provider with a subtype of Listenable/Stream` | `Provider` au lieu de `ChangeNotifierProvider`. | Utiliser `ChangeNotifierProvider`. |
| `ProviderNotFoundException` | Le provider est sous `MaterialApp`, ou le `context` utilisé est celui du widget qui crée le provider. | Placer le provider au-dessus de `MaterialApp` et lire depuis un widget enfant. |
| `LocaleDataException: Locale data has not been initialized` | `initializeDateFormatting` oublié. | L'appeler dans `main`, avant `runApp`. |
| Les dates s'affichent en anglais. | `Intl.defaultLocale` non renseigné. | `Intl.defaultLocale = 'fr_FR';`. |
| Le calendrier de `showDatePicker` est en anglais. | `localizationsDelegates` absent de `MaterialApp`. | Ajouter `GlobalMaterialLocalizations.delegates`. |
| `setState() called after dispose()` | `setState` après un `await` sur un widget détruit. | `if (!mounted) return;` avant le `setState`. |
| Le champ de texte perd le curseur à chaque frappe. | Le `TextEditingController` est recréé dans `build`. | Le déclarer en champ du `State` et le libérer dans `dispose`. |
| Fuite mémoire signalée par le profileur. | Contrôleur non libéré. | `dispose()` sur chaque contrôleur. |
| Effacer l'échéance ne fonctionne pas. | `copyWith(echeance: null)` signifie « ne change rien ». | Utiliser le drapeau `effacerEcheance: true`. |
| Le pourcentage vaut `NaN`. | Division `0 / 0` sur une liste vide. | Traiter explicitement `isEmpty`. |
| La priorité relue est fausse après une mise à jour. | `priorite.index` a été persisté au lieu de `priorite.name`. | Toujours persister `.name`. |
| `Unsupported operation: Cannot remove from an unmodifiable list` | Un `removeAt` sur une liste `unmodifiable`. | Travailler sur `List.from(...)`. |
| `type 'Null' is not a subtype of type 'String'` dans `fromJson` | Une clé absente du JSON. | Lire avec `as String?` et un `??`. |
| Les tests échouent avec « types incompatibles ». | Import relatif de `lib/` depuis `test/`. | Importer en `package:liste_taches/...`. |
| Texte invisible en mode sombre. | Une couleur écrite en dur (`Colors.black`). | Passer par `Theme.of(context).colorScheme`. |
| `getDatabasesPath` échoue sur bureau ou web. | `sqflite` n'y est pas pris en charge nativement. | Ajouter `sqflite_common_ffi` et `databaseFactory = databaseFactoryFfi;`. |

---

## 58.27 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Modèle immuable | Champs `final` + `copyWith`. On ne modifie jamais un objet, on en fabrique un nouveau. |
| `copyWith` et le nullable | Un drapeau explicite (`effacerEcheance`) est nécessaire pour distinguer « ne change pas » de « efface ». |
| `enum` enrichi | Il porte libellé et poids ; il rend les états impossibles à écrire de travers. |
| Persister un `enum` | Toujours `.name`, jamais `.index`. |
| `fromJson` défensif | `as Type?` + `??` + `tryParse` : une donnée abîmée ne doit jamais empêcher le démarrage. |
| Dépôt (`repository`) | Une interface abstraite ; changer de stockage ne touche qu'une ligne de `main`. |
| Interface asynchrone | Rendre asynchrone dès le départ, même quand ce n'est pas nécessaire, évite une réécriture générale plus tard. |
| `ListView.builder` | Seules les tuiles visibles sont construites. Obligatoire au-delà d'une dizaine d'éléments. |
| État vide | Trois questions : que regarde-t-on, pourquoi c'est vide, que faire. Un message par cas. |
| `Dismissible` | La `Key` doit être unique et stable : l'identifiant métier, jamais l'index. |
| Action destructive | Toujours annulable : `SnackBar` + `SnackBarAction`, en mémorisant la position d'origine. |
| Fonctions pures | Filtrer, trier et compter dans des fonctions sans effet de bord : c'est ce qui les rend testables. |
| Tri et valeurs nulles | Décider explicitement où vont les éléments sans valeur ; ici, à la fin. |
| `Form` et `validator` | Le `validator` renvoie `null` quand tout va bien. `validate()` déclenche tout le formulaire. |
| Un formulaire pour deux usages | `tacheExistante == null` distingue création et modification. |
| `showDialog` / `Navigator.push` | Les deux renvoient un `Future` typé ; `null` signifie « l'utilisateur a abandonné ». |
| `showDatePicker` | `context`, `firstDate` et `lastDate` sont obligatoires ; le retour est `Future<DateTime?>`. |
| `intl` | Deux initialisations : `initializeDateFormatting` pour les données, `localizationsDelegates` pour les widgets. |
| `ChangeNotifier` | Il détient l'état, expose des lectures seules, et appelle `notifyListeners()` après chaque changement. |
| `watch` / `read` | `watch` dans un `build`, `read` dans un rappel. C'est l'erreur la plus fréquente avec `provider`. |
| Mise à jour optimiste | Notifier d'abord, écrire ensuite : l'interface reste instantanée. |
| `shared_preferences` | Clé/valeur, une `String` JSON, réécriture complète. Parfait jusqu'à quelques centaines d'objets. |
| `sqflite` | Une ligne par tâche, index, `where`/`orderBy` en SQL. Toujours des paramètres liés (`?`), jamais de concaténation. |
| Thème | Une graine, deux `ColorScheme.fromSeed`, un `themeMode` persisté. |
| Sens des dépendances | `ecrans` → `etat` → `logique`/`donnees` → `modeles`. Jamais l'inverse. |

---

## 58.28 — Extensions : dix défis

Chaque défi est réalisable avec ce que vous savez déjà. L'indication donne la direction, pas la solution.

### Défi 1 — Les sous-tâches (facile)

Ajoutez à `Tache` une liste de sous-tâches, chacune avec un libellé et un booléen. Affichez « 2/5 » dans le sous-titre.

*Indication :* créez une petite classe `SousTache` avec son propre `toJson`/`fromJson`, et sérialisez la liste comme une `List<Map<String, dynamic>>` dans `Tache.toJson`. Le compteur est un `where(...).length` (chapitre 14).

### Défi 2 — Le filtre par étiquette (facile)

Un appui sur `#jeu` dans une tuile ne doit afficher que les tâches portant cette étiquette.

*Indication :* ajoutez `String? _etiquetteActive` à `EtatTaches`, un `filtrerParEtiquette` dans `filtrage.dart`, et une puce fermable au-dessus de la liste. Remplacez le `Text('#$e')` par un `InkWell`.

### Défi 3 — L'ajout rapide en appui long (facile)

Un appui long sur le bouton `+` ouvre le `DialogueAjoutRapide` du 58.9 au lieu du formulaire complet.

*Indication :* `FloatingActionButton` n'a pas de `onLongPress`, mais on peut l'envelopper dans un `GestureDetector`. Le dialogue renvoie une `String` ; appelez `etat.ajouterTitre(titre)`.

### Défi 4 — La réorganisation manuelle (moyen)

Permettez de réordonner les tâches par glisser-déposer.

*Indication :* `ReorderableListView.builder` (chapitre 48) avec `onReorder: (ancien, nouveau)`. Attention : ce tri manuel n'a de sens que si `Tri` propose une valeur `manuel`, sinon le tri automatique écrasera l'ordre choisi. Ajoutez un champ `ordre` à `Tache`.

### Défi 5 — Les statistiques (moyen)

Un second écran affiche : le nombre de tâches par priorité, le nombre terminées cette semaine, et l'étiquette la plus utilisée.

*Indication :* tout se calcule avec `fold` et `Map<String, int>` (chapitre 14). Le nouvel écran lit le même `EtatTaches` par `context.watch` — c'est précisément le bénéfice du 58.20.

### Défi 6 — Les tâches récurrentes (moyen)

Une tâche peut être quotidienne, hebdomadaire ou mensuelle. Quand on la coche, elle se décoche et son échéance avance d'une période.

*Indication :* un `enum Recurrence` avec une méthode `DateTime prochaine(DateTime depuis)`. Interceptez dans `EtatTaches.basculer` : si la tâche est récurrente et qu'on la coche, appelez plutôt `copyWith(faite: false, echeance: ...)`.

### Défi 7 — L'export et l'import JSON (moyen)

Un bouton copie toutes les tâches au format JSON dans le presse-papiers ; un autre les restaure depuis un texte collé.

*Indication :* `Clipboard.setData(ClipboardData(text: ...))` et `Clipboard.getData(Clipboard.kTextPlain)`, tous deux dans `package:flutter/services.dart`. La sérialisation est déjà écrite au 58.4. Validez le JSON importé dans un `try`/`catch` (chapitre 13).

### Défi 8 — La recherche insensible aux accents (difficile)

« ecrire » doit trouver « Écrire la musique ».

*Indication :* écrivez une fonction `String sansAccents(String s)` qui remplace chaque caractère accentué par sa version simple, via une `Map<String, String>` et `replaceAll`. Appliquez-la des deux côtés de la comparaison dans `filtrerParRecherche`. Écrivez le test avant la fonction.

### Défi 9 — Le traitement des échecs d'écriture (difficile)

Si l'écriture disque échoue, l'écran et le stockage divergent silencieusement (voir la remarque du 58.20).

*Indication :* entourez chaque appel au dépôt d'un `try`/`catch` dans `EtatTaches`. En cas d'échec, restaurez la liste précédente, positionnez `_erreur` et notifiez. L'écran affiche alors un `SnackBar` d'erreur. Testez en écrivant un `DepotQuiEchoue implements DepotTaches` qui lève toujours.

### Défi 10 — Les notifications d'échéance (difficile)

Une notification locale prévient la veille de chaque échéance.

*Indication :* le paquet `flutter_local_notifications`, plus `timezone` pour la planification. Attention aux autorisations Android 13+ et à la reprogrammation complète après chaque modification d'échéance. C'est le défi le plus long des dix : comptez une journée.

---

## Et maintenant ?

Vous venez d'écrire votre première application Flutter réellement complète : un modèle sérialisable, une architecture en couches, une interface riche, une persistance interchangeable et des tests automatisés. C'est la structure de la grande majorité des applications de gestion, quelle que soit leur taille — un carnet d'adresses, une application de notes ou un suivi d'entraînement ne diffèrent que par le nom du modèle.

Ce projet reposait sur une liste de données que l'utilisateur remplit lui-même. Le suivant repose sur des données que **vous** fournissez, et sur un enchaînement d'écrans : le chapitre 59 construit un quiz. Vous y retrouverez les modèles et la navigation, mais avec deux nouveautés de taille : un état qui progresse dans le temps (question 3 sur 10, score courant, minuteur) et des animations de transition.

Rendez-vous au chapitre 59 : [59-PARTIE-1C—PROJET-5-QUIZ.md](./59-PARTIE-1C—PROJET-5-QUIZ.md)
