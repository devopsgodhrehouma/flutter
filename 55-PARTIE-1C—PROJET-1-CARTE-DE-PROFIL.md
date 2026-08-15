# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 55 — PROJET 1 : LA CARTE DE PROFIL

> **Niveau :** intermédiaire
> **Durée estimée :** 8 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18) et PARTIE 1B (chapitres 43 à 54)
> **Ce que vous saurez faire à la fin :** construire de zéro un composant d'interface soigné, paramétrable et réutilisable, l'afficher en liste, l'adapter au mode sombre et à la tablette, et ouvrir un écran de détail au clic.

---

## 55.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- lire un cahier des charges et le traduire en une liste d'étapes techniques ;
- créer un projet Flutter propre et y organiser vos fichiers par rôle ;
- définir un thème clair et un thème sombre cohérents avec `ColorScheme.fromSeed()` ;
- écrire une classe modèle Dart pure, sans dépendance à Flutter ;
- afficher un avatar sans le moindre fichier image, à partir des initiales d'un nom ;
- dessiner une bannière avec un dégradé linéaire ;
- superposer un avatar sur une bannière avec `Stack` et `Positioned` ;
- aligner trois statistiques avec des séparateurs verticaux qui ont la bonne hauteur ;
- afficher une liste de badges qui passe automatiquement à la ligne avec `Wrap` ;
- afficher une barre de progression de niveau avec `LinearProgressIndicator` ;
- placer une ligne de boutons d'action qui ne déborde jamais ;
- découper un gros `build()` en petits widgets nommés et réutilisables ;
- rendre un widget paramétrable avec des champs `final` et des rappels (`VoidCallback`) ;
- afficher une liste de profils avec `ListView.builder` ;
- faire basculer toute l'application en mode sombre depuis l'`AppBar` ;
- changer de mise en page au-delà d'une largeur donnée avec `LayoutBuilder` ;
- ouvrir un écran de détail au clic avec `Navigator.push` et animer la transition avec `Hero` ;
- reconnaître et corriger les erreurs de mise en page les plus fréquentes de ce type d'écran.

---

## 55.0.1 — Ce qu'est un chapitre-projet

Les chapitres 43 à 54 vous ont présenté des notions, une par une, avec des exemples courts. Ce chapitre change de format, et tous ceux de la PARTIE 1C aussi.

Ici, il n'y a **plus de notion nouvelle**. Tout ce qui est utilisé a déjà été vu. Ce qui est nouveau, c'est l'**assemblage** : partir d'une page blanche, d'un cahier des charges, et arriver à un résultat fini.

Le déroulement est toujours le même :

```text
1.  Aperçu du résultat final        <- on regarde où on va
2.  Cahier des charges              <- on écrit ce qui est exigé
3.  Notions mobilisées              <- on rappelle d'où elles viennent
4.  Arborescence du projet          <- on décide où vont les fichiers
5.  Construction pas à pas          <- une étape = un état qui tourne
6.  Le code complet, fichier par fichier
7.  Erreurs fréquentes
8.  Ce que vous avez appris
9.  Extensions : 10 défis
10. Et maintenant ?
```

Une règle guide toutes les étapes de la partie 5 :

> **À la fin de chaque étape, l'application compile et se lance.**

Vous ne passez jamais quinze minutes avec un projet cassé. Vous tapez `flutter run` (ou vous sauvegardez pour déclencher le hot reload du chapitre 43), vous regardez, et vous passez à l'étape suivante. C'est ainsi qu'on travaille réellement : par petits pas vérifiables.

Deuxième règle :

> **Chaque bloc de code est précédé du chemin du fichier où le coller.**

Quand vous voyez `lib/widgets/carte_profil.dart` au-dessus d'un bloc, c'est le contenu **complet** de ce fichier, pas un extrait. Vous pouvez remplacer intégralement le fichier existant.

---

## 55.0.2 — Pourquoi une carte de profil comme premier projet

Le choix n'est pas anodin. Une carte de profil est le meilleur exercice de mise en page qui soit, pour quatre raisons.

**Elle contient tous les cas de mise en page en une seule vue.** Un empilement vertical (`Column`), une répartition horizontale (`Row`), une superposition (`Stack`), un retour à la ligne automatique (`Wrap`), une barre proportionnelle, du texte de plusieurs tailles. En quinze étapes, vous aurez révisé tout le chapitre 46.

**Elle n'a besoin d'aucune donnée extérieure.** Pas de serveur, pas de base de données, pas de fichier image. Uniquement des objets Dart écrits en dur. Vous ne perdrez pas une heure sur un problème réseau au lieu de travailler la mise en page.

**Elle est un composant, pas un écran.** Une carte de profil s'affiche seule, en liste, en grille, dans un tiroir latéral, dans un écran de détail. C'est donc l'occasion parfaite d'apprendre à écrire un widget **paramétrable** plutôt qu'un widget jetable.

**Elle se juge à l'œil.** Une calculatrice fonctionne ou ne fonctionne pas. Une carte de profil, elle, est soignée ou ne l'est pas. Ce projet vous entraîne à regarder votre écran avec un œil critique : marges régulières, alignements, hiérarchie de tailles, contraste.

---

## 55.1 — Aperçu du résultat final

Voici ce que vous aurez à la fin du chapitre. Prenez le temps de lire la maquette : chaque zone correspondra à une étape.

```text
╔═════════════════════════════════════════════════════╗
║  Cartes de profil                            [SOM]  ║  <- AppBar + bascule sombre
╠═════════════════════════════════════════════════════╣
║                                                     ║
║   ┌───────────────────────────────────────────────┐ ║
║   │///////////////////////////////////////////////│ ║  <- bannière : dégradé
║   │////  dégradé violet --> turquoise  /////[ OR ]│ ║     + pastille de rang
║   │///////////////////////////////////////////////│ ║
║   │  ,-----.  ////////////////////////////////////│ ║
║   │ (  AL   ) ////////////////////////////////////│ ║  <- avatar en initiales,
║   │  `-----'                                      │ ║     à cheval sur la bannière
║   │                                               │ ║
║   │   Aria Lumen                             ( ● )│ ║  <- nom + pastille en ligne
║   │   Archimage du Vent                           │ ║  <- titre
║   │   Guilde des Sentinelles                      │ ║  <- guilde
║   │                                               │ ║
║   │      128     │      17      │       43        │ ║  <- statistiques
║   │   Victoires  │   Défaites   │     Quêtes      │ ║     + séparateurs
║   │                                               │ ║
║   │   [ Feu ]  [ Vent ]  [ Soin ]  [ Bouclier ]   │ ║  <- badges en Wrap
║   │   [ Vétéran ]                                 │ ║     (passage à la ligne)
║   │                                               │ ║
║   │   Niveau 24                       820 / 1000  │ ║
║   │   ██████████████████░░░░░░░░░░░░░░░░░░░░░░░   │ ║  <- barre de progression
║   │                                               │ ║
║   │   ┌────────────────────┐ ┌──────────────────┐ │ ║
║   │   │      Défier        │ │     Message      │ │ ║  <- boutons d'action
║   │   └────────────────────┘ └──────────────────┘ │ ║
║   └───────────────────────────────────────────────┘ ║
║                                                     ║
║   ┌───────────────────────────────────────────────┐ ║
║   │  (autre profil, même carte, autres données)   │ ║  <- ListView.builder
║   └───────────────────────────────────────────────┘ ║
║                                                     ║
╚═════════════════════════════════════════════════════╝
```

Sur tablette, la même application affiche deux colonnes :

```text
╔═══════════════════════════════════════════════════════════════════════╗
║  Cartes de profil                                            [SOM]    ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────────────────────────────┐  ┌─────────────────────────────┐║
║   │  Aria Lumen                     │  │  Brann Korr                 │║
║   │  ...                            │  │  ...                        │║
║   └─────────────────────────────────┘  └─────────────────────────────┘║
║                                                                       ║
║   ┌─────────────────────────────────┐  ┌─────────────────────────────┐║
║   │  Cyra Vale                      │  │  Doran Fisk                 │║
║   │  ...                            │  │  ...                        │║
║   └─────────────────────────────────┘  └─────────────────────────────┘║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

Et un clic sur une carte ouvre un écran de détail :

```text
╔═════════════════════════════════════════════════════╗
║  <-   Aria Lumen                                    ║
╠═════════════════════════════════════════════════════╣
║   ┌───────────────────────────────────────────────┐ ║
║   │        (la même carte, en grand)              │ ║
║   └───────────────────────────────────────────────┘ ║
║                                                     ║
║   FICHE DÉTAILLÉE                                   ║
║   ───────────────────────────────────────────────   ║
║   Rang               Or                             ║
║   Niveau             24                             ║
║   Expérience         820 / 1000                     ║
║   Parties jouées     145                            ║
║   Taux de victoire   88 %                           ║
║   Quêtes terminées   43                             ║
║   Statut             En ligne                       ║
║                                                     ║
║   BADGES                                            ║
║   ───────────────────────────────────────────────   ║
║   [ Feu ] [ Vent ] [ Soin ] [ Bouclier ] [ Vétéran ]║
║                                                     ║
╚═════════════════════════════════════════════════════╝
```

---

## 55.2 — Cahier des charges

Un cahier des charges se lit avant d'écrire la première ligne de code. Il sépare toujours deux catégories : ce qui **doit** être fait, et ce qui **peut** l'être si le temps le permet.

### 55.2.1 — Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | L'application affiche au moins cinq profils différents. | On fait défiler, on compte. |
| O2 | Aucun fichier image n'est utilisé. Les avatars sont générés à partir des initiales. | Le dossier `assets/` n'existe pas. |
| O3 | Chaque carte affiche : avatar, nom, titre, guilde, rang, statut en ligne. | Lecture visuelle. |
| O4 | Chaque carte affiche trois statistiques séparées par des traits verticaux. | Lecture visuelle. |
| O5 | Chaque carte affiche les badges du joueur, avec passage à la ligne automatique. | Un profil a 6 badges : ils ne débordent pas. |
| O6 | Chaque carte affiche une barre de progression de niveau, proportionnelle à l'expérience. | Un profil à 50 % remplit la moitié de la barre. |
| O7 | Chaque carte propose deux boutons d'action qui réagissent au clic. | Un `SnackBar` s'affiche. |
| O8 | La carte est un widget **paramétrable** : elle reçoit un `Profil` et des rappels, elle ne connaît aucune donnée en dur. | On peut afficher deux profils sans dupliquer une ligne de code. |
| O9 | Toutes les couleurs et toutes les tailles de texte viennent du thème. | Aucun `Colors.blue` en dur dans les widgets de la carte. |
| O10 | L'application possède un mode clair et un mode sombre, basculables depuis l'`AppBar`. | On clique, tout change, rien n'est illisible. |
| O11 | Au-delà de 700 pixels de large, l'affichage passe à deux colonnes. | On agrandit la fenêtre sur bureau ou web. |
| O12 | Un clic sur une carte ouvre un écran de détail avec une flèche de retour. | On clique, on revient. |
| O13 | Aucune erreur `RenderFlex overflowed` sur un écran de 320 pixels de large. | On teste sur un petit appareil. |

### 55.2.2 — Bonus

| # | Bonus | Difficulté |
| --- | --- | --- |
| B1 | Un mode « compact » de la carte, sans badges ni boutons. | facile |
| B2 | Une animation `Hero` de l'avatar entre la liste et le détail. | moyenne |
| B3 | Une couleur d'accent différente selon le rang du joueur. | facile |
| B4 | Une bordure lumineuse autour de l'avatar des joueurs en ligne. | facile |
| B5 | Le support d'une photo réseau optionnelle (`Image.network`) quand l'URL est fournie. | moyenne |

Les cinq bonus sont traités dans le chapitre. Les dix défis de la section 55.24 vont plus loin.

---

## 55.3 — Notions mobilisées

Ce tableau est votre plan de révision. Si une ligne vous semble floue, retournez au chapitre indiqué **avant** de commencer.

| Notion utilisée | Chapitre | Où elle sert dans ce projet |
| --- | --- | --- |
| Classe, champs `final`, constructeur `const` | 08, 09 | La classe `Profil` |
| Paramètres nommés `required`, valeurs par défaut | 09 | Le constructeur de `Profil` et de tous les widgets |
| Getters calculés | 08 | `initiales`, `progression`, `tauxVictoire` |
| `enum` amélioré avec champ et constructeur | 11 | `Rang` (bronze, argent, or, diamant) |
| Null safety, `?`, `??`, `?.` | 12 | `urlAvatar`, les rappels `VoidCallback?` |
| `List`, `map`, `toList()` | 06, 14 | Les badges transformés en widgets |
| Organisation en fichiers et `import` relatifs | 16 | L'arborescence `lib/modeles`, `lib/widgets`… |
| `flutter create`, hot reload | 43 | Mise en place, étape 55.5 |
| Widget, arbre, composition, `const` | 44 | Extraction en petits widgets, étape 55.15 |
| `StatelessWidget` / `StatefulWidget`, `setState` | 45 | La bascule clair/sombre, étape 55.18 |
| `Container`, `Padding`, `SizedBox`, `Column`, `Row` | 46 | Toute la carte |
| `Expanded`, `Spacer` | 46 | Les boutons d'action, la ligne de statistiques |
| `Stack` et `Positioned` | 46 | L'avatar à cheval sur la bannière, étape 55.10 |
| `Wrap` | 46 | Les badges, étape 55.12 |
| `Text`, `TextStyle`, `textTheme` | 47 | Nom, titre, guilde, libellés |
| `Icon` et `Icons` | 47 | Les badges et les boutons |
| `CircleAvatar` | 47 | L'avatar, étape 55.8 |
| `Image.network` | 47 | La photo optionnelle, bonus B5 |
| `Card`, `ListView.builder` | 48 | La liste de profils, étape 55.17 |
| `GridView.builder` | 48 | Le mode tablette, étape 55.19 |
| `FilledButton`, `OutlinedButton`, `SnackBar` | 49 | Les boutons d'action, étape 55.14 |
| `Navigator.push`, `MaterialPageRoute`, `Hero` | 50 | L'écran de détail, étape 55.20 |
| `ThemeData`, `ColorScheme.fromSeed`, `themeMode` | 51 | Le thème, étapes 55.6 et 55.18 |
| `LayoutBuilder`, points de rupture | 51 | L'adaptation tablette, étape 55.19 |

---

## 55.4 — Arborescence du projet

Voici l'arborescence finale. Elle est donnée **maintenant**, avant la première ligne de code, pour que vous sachiez à tout moment où vous êtes.

```text
carte_profil/
├── pubspec.yaml
├── analysis_options.yaml
└── lib/
    ├── main.dart                        <- point d'entrée, thème, bascule sombre
    │
    ├── modeles/
    │   └── profil.dart                  <- la classe Profil et l'enum Rang (Dart pur)
    │
    ├── donnees/
    │   └── profils_demo.dart            <- six profils écrits en dur
    │
    ├── theme/
    │   ├── dimensions.dart              <- espacements et rayons, centralisés
    │   └── theme_application.dart       <- themeClair, themeSombre, couleurs de rang
    │
    ├── widgets/
    │   ├── avatar_profil.dart           <- CircleAvatar en initiales
    │   ├── banniere_profil.dart         <- dégradé + pastille de rang
    │   ├── identite_profil.dart         <- nom, titre, guilde
    │   ├── statistiques_profil.dart     <- Row + séparateurs
    │   ├── badges_profil.dart           <- Wrap de badges
    │   ├── barre_niveau.dart            <- LinearProgressIndicator
    │   ├── actions_profil.dart          <- les deux boutons
    │   └── carte_profil.dart            <- l'assemblage des sept précédents
    │
    └── ecrans/
        ├── ecran_liste_profils.dart     <- liste ou grille selon la largeur
        └── ecran_detail_profil.dart     <- fiche détaillée
```

### 55.4.1 — Pourquoi ce découpage

Trois principes, hérités du chapitre 16.

**Un dossier par rôle, pas par écran.** `modeles/` contient des données, `widgets/` contient des morceaux d'interface réutilisables, `ecrans/` contient des pages entières. Quand un projet grossit, on trouve un fichier par ce qu'il **fait**, pas par l'endroit où il est utilisé.

**Un fichier par widget public.** Un fichier de 700 lignes contenant huit widgets est illisible et impossible à relire à deux. Un fichier de 60 lignes contenant un widget se lit d'un coup d'œil.

**Le modèle ne connaît pas Flutter.** `lib/modeles/profil.dart` n'importe **pas** `package:flutter/material.dart`. C'est du Dart pur, exactement comme au chapitre 09. Cette règle vous permettra plus tard de tester le modèle sans lancer d'interface, et de le réutiliser dans un serveur ou un jeu Flame.

> Un débutant est tenté de mettre une `Color` dans son modèle. Ne le faites pas. La couleur est une décision d'**affichage**, pas une donnée du joueur. Nous verrons en 55.7 comment relier un rang à une couleur sans polluer le modèle.

---

## 55.5 — Étape 1 : mise en place du projet

### 55.5.1 — Créer le projet

Ouvrez un terminal dans le dossier où vous rangez vos projets, puis lancez la commande vue au chapitre 43 :

```text
flutter create carte_profil
cd carte_profil
```

Vérifiez que tout fonctionne avant d'écrire quoi que ce soit :

```text
flutter run
```

Vous devez voir le compteur d'exemple de Flutter. Si ce n'est pas le cas, ne continuez pas : reprenez le chapitre 43, section `flutter doctor`.

### 55.5.2 — Faire le ménage

Le projet créé par `flutter create` contient une application de démonstration dont nous n'avons pas besoin. Créez d'abord les dossiers :

```text
mkdir -p lib/modeles lib/donnees lib/theme lib/widgets lib/ecrans
```

Puis remplacez intégralement `lib/main.dart` par le squelette suivant.

`lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

/// Racine de l'application.
///
/// Pour l'instant elle ne fait rien d'autre que poser un `MaterialApp`
/// et un `Scaffold`, comme au chapitre 44.
class ApplicationCarteProfil extends StatelessWidget {
  const ApplicationCarteProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      // Retire le bandeau rouge « DEBUG » du coin supérieur droit.
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('Cartes de profil')),
        body: const Center(
          child: Text('Le projet est en place.'),
        ),
      ),
    );
  }
}
```

### 55.5.3 — État attendu

Sauvegardez. Le hot reload s'applique.

```text
┌─────────────────────────────────┐
│  Cartes de profil               │
├─────────────────────────────────┤
│                                 │
│                                 │
│      Le projet est en place.    │
│                                 │
│                                 │
└─────────────────────────────────┘
```

**L'application compile et se lance.** Première étape terminée.

> Si vous voyez encore le compteur, c'est que vous avez modifié un autre fichier que `lib/main.dart`, ou que vous n'avez pas sauvegardé.

---

## 55.6 — Étape 2 : le thème de l'application

### 55.6.1 — Pourquoi commencer par le thème

Un débutant écrit d'abord les widgets, avec des couleurs choisies au hasard, puis « ajoute le thème » à la fin. C'est le meilleur moyen de passer une soirée à remplacer des `Colors.blue` un par un.

On fait l'inverse. On décide **d'abord** de la palette et des espacements. Ensuite, tous les widgets s'y réfèrent. C'est exactement le raisonnement du chapitre 51 :

> Un widget ne choisit jamais une couleur. Il **demande** au thème la couleur du rôle dont il a besoin.

### 55.6.2 — Les constantes de dimension

Commençons par le plus simple : les espacements et les rayons. Une seule échelle, utilisée partout, suffit à donner une impression de sérieux.

`lib/theme/dimensions.dart`

```dart
/// Échelle d'espacements de l'application.
///
/// `abstract final class` est la façon Dart 3 de déclarer un simple conteneur
/// de constantes : on ne peut ni l'instancier, ni en hériter (chapitre 11).
abstract final class Espace {
  /// 4 pixels : entre deux lignes de texte très liées.
  static const double xs = 4;

  /// 8 pixels : entre une icône et son libellé.
  static const double s = 8;

  /// 12 pixels : entre deux petits blocs.
  static const double m = 12;

  /// 16 pixels : marge intérieure standard d'une carte.
  static const double l = 16;

  /// 24 pixels : entre deux sections d'une carte.
  static const double xl = 24;
}

/// Échelle de rayons d'arrondi.
abstract final class Rayon {
  static const double petit = 8;
  static const double moyen = 16;
  static const double grand = 24;
}

/// Points de rupture pour l'adaptation aux grands écrans (chapitre 51).
abstract final class Rupture {
  /// En dessous : téléphone, une seule colonne.
  /// Au-dessus : tablette ou bureau, deux colonnes.
  static const double tablette = 700;
}
```

> Retenez le principe : **cinq espacements possibles, pas cinquante**. Si vous vous surprenez à écrire `EdgeInsets.all(13)`, c'est que vous tâtonnez.

### 55.6.3 — Le thème clair et le thème sombre

Une seule fonction construit les deux thèmes. Elle prend une `Brightness` et renvoie un `ThemeData` complet. C'est le schéma du chapitre 51 : ainsi, le mode sombre n'est jamais « oublié ».

`lib/theme/theme_application.dart`

```dart
import 'package:flutter/material.dart';

import 'dimensions.dart';

/// Couleur de graine de l'application.
///
/// Toute la palette Material 3 est dérivée de cette unique couleur.
const Color _graine = Color(0xFF5B4BE1);

/// Construit le thème complet pour une luminosité donnée.
///
/// Appelée deux fois : une fois en clair, une fois en sombre.
ThemeData construireTheme(Brightness luminosite) {
  final ColorScheme schema = ColorScheme.fromSeed(
    seedColor: _graine,
    brightness: luminosite,
  );

  return ThemeData(
    colorScheme: schema,

    appBarTheme: AppBarTheme(
      centerTitle: false,
      elevation: 0,
      scrolledUnderElevation: 2,
      backgroundColor: schema.surface,
      foregroundColor: schema.onSurface,
      titleTextStyle: TextStyle(
        fontSize: 20,
        fontWeight: FontWeight.w600,
        color: schema.onSurface,
      ),
    ),

    // Attention : le paramètre s'appelle `cardTheme` mais attend un
    // `CardThemeData` (et non `CardTheme`, qui est un widget). Voir 51.
    cardTheme: CardThemeData(
      elevation: 0,
      color: schema.surfaceContainerLow,
      margin: EdgeInsets.zero,
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(Rayon.grand),
      ),
    ),

    filledButtonTheme: FilledButtonThemeData(
      style: FilledButton.styleFrom(
        minimumSize: const Size(0, 48),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(Rayon.moyen),
        ),
      ),
    ),

    outlinedButtonTheme: OutlinedButtonThemeData(
      style: OutlinedButton.styleFrom(
        minimumSize: const Size(0, 48),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(Rayon.moyen),
        ),
      ),
    ),

    dividerTheme: DividerThemeData(
      color: schema.outlineVariant,
      thickness: 1,
      space: 1,
    ),

    snackBarTheme: SnackBarThemeData(
      behavior: SnackBarBehavior.floating,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(Rayon.moyen),
      ),
    ),
  );
}
```

Trois choix méritent une explication.

**`margin: EdgeInsets.zero` sur les cartes.** Par défaut, une `Card` s'entoure d'une marge de 4 pixels que vous ne contrôlez pas. Comme nous voulons maîtriser l'espacement depuis la liste, nous la mettons à zéro et nous gérons les marges nous-mêmes.

**`clipBehavior: Clip.antiAlias`.** Sans cela, la bannière en dégradé dépassera des coins arrondis de la carte. Ce détail est la cause numéro un des « coins carrés » sur ce type de composant. Nous le vérifierons à l'étape 55.10.

**`space: 1` sur les séparateurs.** Par défaut, un `Divider` réserve 16 pixels de hauteur, dont un seul est réellement le trait. Nous voulons des séparateurs dont l'épaisseur visuelle égale l'espace occupé ; l'espacement, lui, viendra de nos `Espace`.

### 55.6.4 — Brancher le thème

`lib/main.dart`

```dart
import 'package:flutter/material.dart';

import 'theme/theme_application.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

class ApplicationCarteProfil extends StatelessWidget {
  const ApplicationCarteProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      // Pour l'instant, on suit le réglage du système.
      // L'étape 55.18 rendra ce choix manuel.
      themeMode: ThemeMode.system,
      home: Scaffold(
        appBar: AppBar(title: const Text('Cartes de profil')),
        body: const Center(
          child: Text('Thème en place.'),
        ),
      ),
    );
  }
}
```

### 55.6.5 — État attendu

L'écran est presque identique, mais la barre du haut a changé de teinte : elle suit maintenant la palette violette dérivée de la graine.

**Vérification immédiate :** passez votre système en mode sombre. L'application doit devenir sombre sans que vous ayez touché à quoi que ce soit. Si ce n'est pas le cas, c'est que `darkTheme` est absent ou que `brightness` n'est pas transmis à `ColorScheme.fromSeed`.

**L'application compile et se lance.** Deuxième étape terminée.

---

## 55.7 — Étape 3 : la classe modèle `Profil`

### 55.7.1 — Ce que le modèle doit contenir

Reprenons la maquette de 55.1 et listons chaque donnée affichée. C'est la méthode la plus fiable pour concevoir un modèle : **on part de l'écran, pas de son imagination**.

```text
Zone de la maquette         Donnée nécessaire            Type
─────────────────────────   ──────────────────────────   ──────────────
Avatar « AL »               le nom (pour les initiales)  String
Pastille « OR »             le rang                      Rang (enum)
« Aria Lumen »              le nom                       String
« Archimage du Vent »       le titre                     String
« Guilde des Sentinelles »  la guilde                    String
Pastille verte              en ligne ou non              bool
« 128 / 17 / 43 »           victoires, défaites, quêtes  int, int, int
[ Feu ] [ Vent ] …          la liste des badges          List<String>
« Niveau 24 »               le niveau                    int
« 820 / 1000 »              expérience et palier         int, int
Photo (optionnelle)         une URL, ou rien             String?
```

Deux remarques importantes.

**Le taux de victoire n'est pas dans la liste.** C'est normal : il se **calcule** à partir des victoires et des défaites. Une donnée calculable ne se stocke jamais, sinon elle finit par contredire les autres. Elle devient un **getter** (chapitre 08).

**La couleur du rang n'est pas dans la liste non plus.** Le rang est une donnée du joueur ; la couleur est une décision d'affichage. Le modèle stocke `Rang.or`, l'interface décide que l'or se dessine en `0xFFC9971C`.

### 55.7.2 — L'enum `Rang`

`Rang` est un `enum` **amélioré** (chapitre 11) : chaque valeur transporte un libellé lisible et un niveau de prestige.

### 55.7.3 — Le fichier du modèle

`lib/modeles/profil.dart`

```dart
/// Rang d'un joueur, du plus bas au plus haut.
///
/// C'est un `enum` amélioré (chapitre 11) : chaque valeur porte un libellé
/// affichable et un niveau de prestige utilisable pour trier.
///
/// Remarquez ce que ce fichier n'importe PAS : `package:flutter/material.dart`.
/// Un modèle est du Dart pur. Il peut être testé sans lancer d'interface.
enum Rang {
  bronze('Bronze', 1),
  argent('Argent', 2),
  or('Or', 3),
  diamant('Diamant', 4);

  const Rang(this.libelle, this.prestige);

  /// Libellé affichable, accentué et avec une majuscule.
  final String libelle;

  /// Ordre de prestige : 1 pour le plus bas, 4 pour le plus haut.
  final int prestige;
}

/// Profil d'un joueur.
///
/// Tous les champs sont `final` et le constructeur est `const` : un profil est
/// une valeur **immuable** (chapitre 09). Pour « modifier » un profil, on en
/// fabrique une copie avec `copieAvec`.
class Profil {
  const Profil({
    required this.nom,
    required this.titre,
    required this.guilde,
    required this.rang,
    required this.niveau,
    required this.experience,
    required this.experienceRequise,
    required this.victoires,
    required this.defaites,
    required this.quetesTerminees,
    this.badges = const <String>[],
    this.enLigne = false,
    this.urlAvatar,
  });

  /// Nom complet, par exemple « Aria Lumen ».
  final String nom;

  /// Titre de jeu, par exemple « Archimage du Vent ».
  final String titre;

  /// Nom de la guilde, ou « Sans guilde ».
  final String guilde;

  /// Rang du joueur.
  final Rang rang;

  /// Niveau atteint.
  final int niveau;

  /// Expérience accumulée dans le niveau courant.
  final int experience;

  /// Expérience nécessaire pour passer au niveau suivant.
  final int experienceRequise;

  final int victoires;
  final int defaites;
  final int quetesTerminees;

  /// Badges obtenus. Peut être vide, jamais `null` (chapitre 12).
  final List<String> badges;

  /// `true` si le joueur est connecté en ce moment.
  final bool enLigne;

  /// URL d'une photo, ou `null` si le joueur n'en a pas.
  ///
  /// C'est le seul champ nullable du modèle, et c'est justifié : l'absence de
  /// photo est une information, pas une erreur.
  final String? urlAvatar;

  /// Une ou deux lettres à afficher dans l'avatar.
  ///
  /// « Aria Lumen » donne « AL », « Zoran » donne « Z », un nom vide donne « ? ».
  String get initiales {
    final List<String> mots = nom
        .trim()
        .split(' ')
        .where((String mot) => mot.isNotEmpty)
        .toList();

    if (mots.isEmpty) {
      return '?';
    }
    if (mots.length == 1) {
      return mots.first.substring(0, 1).toUpperCase();
    }
    return (mots.first.substring(0, 1) + mots.last.substring(0, 1))
        .toUpperCase();
  }

  /// Avancement dans le niveau courant, entre 0.0 et 1.0.
  ///
  /// La valeur est bornée à la main plutôt qu'avec `clamp` : c'est plus long
  /// d'une ligne, mais le type de retour est indiscutablement un `double`.
  double get progression {
    if (experienceRequise <= 0) {
      return 1.0;
    }
    final double brut = experience / experienceRequise;
    if (brut < 0.0) {
      return 0.0;
    }
    if (brut > 1.0) {
      return 1.0;
    }
    return brut;
  }

  /// Nombre total de parties jouées.
  int get partiesJouees => victoires + defaites;

  /// Taux de victoire arrondi, en pourcentage entier.
  ///
  /// Renvoie 0 si aucune partie n'a été jouée : on ne divise jamais par zéro.
  int get pourcentageVictoires {
    if (partiesJouees == 0) {
      return 0;
    }
    return (victoires * 100 / partiesJouees).round();
  }

  /// Renvoie une copie du profil avec certains champs remplacés.
  ///
  /// C'est le patron `copyWith` du chapitre 09, indispensable dès qu'un objet
  /// est immuable.
  Profil copieAvec({
    String? nom,
    String? titre,
    String? guilde,
    Rang? rang,
    int? niveau,
    int? experience,
    int? experienceRequise,
    int? victoires,
    int? defaites,
    int? quetesTerminees,
    List<String>? badges,
    bool? enLigne,
    String? urlAvatar,
  }) {
    return Profil(
      nom: nom ?? this.nom,
      titre: titre ?? this.titre,
      guilde: guilde ?? this.guilde,
      rang: rang ?? this.rang,
      niveau: niveau ?? this.niveau,
      experience: experience ?? this.experience,
      experienceRequise: experienceRequise ?? this.experienceRequise,
      victoires: victoires ?? this.victoires,
      defaites: defaites ?? this.defaites,
      quetesTerminees: quetesTerminees ?? this.quetesTerminees,
      badges: badges ?? this.badges,
      enLigne: enLigne ?? this.enLigne,
      urlAvatar: urlAvatar ?? this.urlAvatar,
    );
  }

  @override
  String toString() => 'Profil($nom, niveau $niveau, ${rang.libelle})';
}
```

### 55.7.4 — Tester le modèle sans interface

Le modèle est du Dart pur : vous pouvez le vérifier dans DartPad ou avec `dart run`, sans Flutter. Copiez le contenu de `profil.dart`, puis ajoutez ce `main` à la fin :

```dart
void main() {
  const Profil aria = Profil(
    nom: 'Aria Lumen',
    titre: 'Archimage du Vent',
    guilde: 'Guilde des Sentinelles',
    rang: Rang.or,
    niveau: 24,
    experience: 820,
    experienceRequise: 1000,
    victoires: 128,
    defaites: 17,
    quetesTerminees: 43,
    badges: <String>['Feu', 'Vent'],
    enLigne: true,
  );

  print(aria);
  print('Initiales      : ${aria.initiales}');
  print('Progression    : ${aria.progression}');
  print('Parties jouées : ${aria.partiesJouees}');
  print('Victoires      : ${aria.pourcentageVictoires} %');

  const Profil zoran = Profil(
    nom: 'Zoran',
    titre: 'Recrue',
    guilde: 'Sans guilde',
    rang: Rang.bronze,
    niveau: 1,
    experience: 0,
    experienceRequise: 100,
    victoires: 0,
    defaites: 0,
    quetesTerminees: 0,
  );

  print(zoran);
  print('Initiales      : ${zoran.initiales}');
  print('Progression    : ${zoran.progression}');
  print('Victoires      : ${zoran.pourcentageVictoires} %');
}
```

**Résultat :**

```text
Profil(Aria Lumen, niveau 24, Or)
Initiales      : AL
Progression    : 0.82
Parties jouées : 145
Victoires      : 88 %
Profil(Zoran, niveau 1, Bronze)
Initiales      : Z
Progression    : 0.0
Victoires      : 0 %
```

Notez les deux cas limites déjà couverts : le nom d'un seul mot donne une seule initiale, et un joueur sans partie ne provoque pas de division par zéro.

### 55.7.5 — Les données de démonstration

`lib/donnees/profils_demo.dart`

```dart
import '../modeles/profil.dart';

/// Six profils écrits en dur, pour travailler l'interface sans serveur.
///
/// La liste est `const` : elle est construite une fois pour toutes à la
/// compilation, et Flutter n'aura jamais à la reconstruire.
const List<Profil> profilsDemo = <Profil>[
  Profil(
    nom: 'Aria Lumen',
    titre: 'Archimage du Vent',
    guilde: 'Guilde des Sentinelles',
    rang: Rang.or,
    niveau: 24,
    experience: 820,
    experienceRequise: 1000,
    victoires: 128,
    defaites: 17,
    quetesTerminees: 43,
    badges: <String>['Feu', 'Vent', 'Soin', 'Bouclier', 'Vétéran'],
    enLigne: true,
  ),
  Profil(
    nom: 'Brann Korr',
    titre: 'Berserker de la Faille',
    guilde: 'Clan de la Hache',
    rang: Rang.argent,
    niveau: 17,
    experience: 340,
    experienceRequise: 700,
    victoires: 76,
    defaites: 54,
    quetesTerminees: 21,
    badges: <String>['Force', 'Rage', 'Endurance'],
  ),
  Profil(
    nom: 'Cyra Vale',
    titre: 'Voleuse d\'Ombre',
    guilde: 'Guilde des Sentinelles',
    rang: Rang.diamant,
    niveau: 31,
    experience: 1180,
    experienceRequise: 1400,
    victoires: 210,
    defaites: 33,
    quetesTerminees: 67,
    // Six badges : c'est ce profil qui prouvera que le Wrap fonctionne.
    badges: <String>[
      'Vitesse',
      'Discrétion',
      'Poison',
      'Vétéran',
      'Champion',
      'Explorateur',
    ],
    enLigne: true,
  ),
  Profil(
    nom: 'Doran Fisk',
    titre: 'Apprenti forgeron',
    guilde: 'Sans guilde',
    rang: Rang.bronze,
    niveau: 4,
    experience: 60,
    experienceRequise: 300,
    victoires: 8,
    defaites: 12,
    quetesTerminees: 3,
    // Aucun badge : c'est ce profil qui prouvera que la carte tient
    // sans la section des badges.
  ),
  Profil(
    nom: 'Elyn Sorrow',
    titre: 'Prêtresse de l\'Aube',
    guilde: 'Ordre du Calice',
    rang: Rang.or,
    niveau: 22,
    experience: 990,
    experienceRequise: 1000,
    victoires: 95,
    defaites: 40,
    quetesTerminees: 51,
    badges: <String>['Soin', 'Lumière', 'Bouclier'],
    enLigne: true,
    // Seul profil avec une photo : il servira à tester `Image.network`.
    urlAvatar: 'https://picsum.photos/200',
  ),
  Profil(
    nom: 'Kaeda Muun',
    titre: 'Maîtresse des bêtes',
    guilde: 'Cercle Sauvage',
    rang: Rang.argent,
    niveau: 13,
    experience: 150,
    experienceRequise: 600,
    victoires: 44,
    defaites: 39,
    quetesTerminees: 18,
    badges: <String>['Nature', 'Vitesse'],
  ),
];
```

> Notez `'Voleuse d\'Ombre'` : l'apostrophe française doit être échappée dans une chaîne délimitée par des apostrophes. Vous pouvez aussi écrire `"Voleuse d'Ombre"` avec des guillemets doubles. Les deux formes sont correctes en Dart (chapitre 02).

### 55.7.6 — État attendu

`lib/main.dart`

```dart
import 'package:flutter/material.dart';

import 'donnees/profils_demo.dart';
import 'modeles/profil.dart';
import 'theme/theme_application.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

class ApplicationCarteProfil extends StatelessWidget {
  const ApplicationCarteProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      themeMode: ThemeMode.system,
      home: const EcranDemo(),
    );
  }
}

/// Écran de travail temporaire.
///
/// Il ne sert qu'à vérifier que le modèle et les données sont accessibles.
/// Il sera remplacé à l'étape 55.17.
class EcranDemo extends StatelessWidget {
  const EcranDemo({super.key});

  @override
  Widget build(BuildContext context) {
    final Profil profil = profilsDemo.first;

    return Scaffold(
      appBar: AppBar(title: const Text('Cartes de profil')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Text(profil.nom),
            Text(profil.titre),
            Text('Initiales : ${profil.initiales}'),
            Text('Rang : ${profil.rang.libelle}'),
            Text('Progression : ${profil.progression}'),
            Text('Victoires : ${profil.pourcentageVictoires} %'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
┌─────────────────────────────────┐
│  Cartes de profil               │
├─────────────────────────────────┤
│  Aria Lumen                     │
│  Archimage du Vent              │
│  Initiales : AL                 │
│  Rang : Or                      │
│  Progression : 0.82             │
│  Victoires : 88 %               │
└─────────────────────────────────┘
```

C'est laid, et c'est voulu. À ce stade, nous vérifions uniquement que **les données arrivent bien jusqu'à l'écran**. Les cinq étapes suivantes ne feront que les habiller.

**L'application compile et se lance.** Troisième étape terminée.

---

## 55.8 — Étape 4 : l'avatar en initiales

### 55.8.1 — Le problème et sa solution

Nous n'avons aucun fichier image, et nous n'en voulons aucun. Il faut pourtant une vignette identifiable pour chaque joueur.

La solution universelle, employée par Gmail, Slack ou GitHub quand l'utilisateur n'a pas de photo : un **disque coloré contenant les initiales**. Flutter fournit exactement le widget qu'il faut, vu au chapitre 47 : `CircleAvatar`.

```text
        CircleAvatar
        ┌───────────────────────────────┐
        │ radius           rayon en px  │
        │ backgroundColor  le disque    │
        │ foregroundColor  le texte     │
        │ child            ce qu'on met │
        └───────────────────────────────┘
```

La couleur du disque viendra du rang. Nous avons donc besoin d'une fonction qui traduit un `Rang` en `Color`.

### 55.8.2 — Relier le rang à une couleur

Cette fonction est le pont entre le modèle (Dart pur) et l'interface. Sa place naturelle est donc le fichier de thème, pas le modèle.

Ajoutez ces éléments **à la fin** de `lib/theme/theme_application.dart` (le reste du fichier ne change pas ; le fichier complet est donné en 55.21) :

```dart
/// Couleur associée à chaque rang.
///
/// Le `switch` sur un `enum` est exhaustif : Dart vérifie à la compilation que
/// les quatre valeurs sont traitées. Si vous ajoutez `Rang.legende`, le
/// compilateur vous signalera ce fichier. C'est exactement ce qu'on veut.
Color couleurDuRang(Rang rang) {
  switch (rang) {
    case Rang.bronze:
      return const Color(0xFF9C6B3F);
    case Rang.argent:
      return const Color(0xFF7A8B99);
    case Rang.or:
      return const Color(0xFFC9971C);
    case Rang.diamant:
      return const Color(0xFF2FA8C6);
  }
}

/// Vert du statut « en ligne ».
///
/// Il ne vient pas du `ColorScheme` : c'est une couleur sémantique, dont le
/// sens ne doit pas changer avec la palette de l'application.
const Color vertEnLigne = Color(0xFF2E9E5B);
```

Et ajoutez l'import du modèle en haut du même fichier :

```dart
import '../modeles/profil.dart';
```

> Notez la dépendance : le **thème** connaît le **modèle**, jamais l'inverse. Une flèche dans un seul sens, c'est une architecture saine.

### 55.8.3 — Le code de l'avatar

`lib/main.dart`

```dart
import 'package:flutter/material.dart';

import 'donnees/profils_demo.dart';
import 'modeles/profil.dart';
import 'theme/dimensions.dart';
import 'theme/theme_application.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

class ApplicationCarteProfil extends StatelessWidget {
  const ApplicationCarteProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      themeMode: ThemeMode.system,
      home: const EcranDemo(),
    );
  }
}

class EcranDemo extends StatelessWidget {
  const EcranDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cartes de profil')),
      body: Padding(
        padding: const EdgeInsets.all(Espace.l),
        child: CarteProfil(profil: profilsDemo.first),
      ),
    );
  }
}

/// La carte de profil, en cours de construction.
///
/// Elle grossira à chaque étape jusqu'à 55.14, puis sera découpée en 55.15.
class CarteProfil extends StatelessWidget {
  const CarteProfil({super.key, required this.profil});

  final Profil profil;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Color accent = couleurDuRang(profil.rang);

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(Espace.l),
        child: Center(
          // L'anneau : un disque légèrement plus grand, de la couleur du fond
          // de la carte, qui détachera l'avatar de la bannière en 55.10.
          child: Container(
            padding: const EdgeInsets.all(3),
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              color: profil.enLigne ? vertEnLigne : theme.cardTheme.color,
            ),
            child: CircleAvatar(
              radius: 36,
              backgroundColor: accent,
              foregroundColor: Colors.white,
              child: Text(
                profil.initiales,
                style: const TextStyle(
                  fontSize: 26,
                  fontWeight: FontWeight.w700,
                  letterSpacing: 1,
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

### 55.8.4 — Trois détails qui font la différence

**Le `Text` n'a pas de couleur.** Il se contente d'une taille et d'une graisse. La couleur vient de `foregroundColor` du `CircleAvatar`, qui installe un `DefaultTextStyle` autour de son enfant. Si vous écriviez `color: Colors.white` dans le `TextStyle`, vous perdriez la possibilité de changer d'avis depuis un seul endroit.

**L'anneau est un `Container` circulaire avec 3 pixels de marge intérieure.** Il n'y a pas de « bordure » à dessiner : un disque un peu plus grand derrière un disque plus petit produit exactement le même effet, et gère mieux les arrondis.

**Le rayon vaut 36, la police 26.** Le rapport est d'environ 0,7. Retenez-le : si vous doublez le rayon, doublez la police. Nous rendrons ce calcul automatique en 55.15.

### 55.8.5 — Le cas de la photo réseau (bonus B5)

Le profil `Elyn Sorrow` possède une `urlAvatar`. Voici comment l'utiliser sans casser les autres. Remplacez le `CircleAvatar` par :

```dart
CircleAvatar(
  radius: 36,
  backgroundColor: accent,
  foregroundColor: Colors.white,
  // NetworkImage ne télécharge que si l'URL n'est pas nulle.
  backgroundImage: profil.urlAvatar == null
      ? null
      : NetworkImage(profil.urlAvatar!),
  // Le texte ne s'affiche QUE s'il n'y a pas de photo : sinon les initiales
  // se dessineraient par-dessus l'image.
  child: profil.urlAvatar == null
      ? Text(
          profil.initiales,
          style: const TextStyle(
            fontSize: 26,
            fontWeight: FontWeight.w700,
            letterSpacing: 1,
          ),
        )
      : null,
),
```

Le `!` après `profil.urlAvatar` est autorisé ici, et seulement ici : nous sommes dans la branche où le test `== null` a déjà échoué (chapitre 12).

> Sans connexion réseau, `NetworkImage` laisse simplement le disque coloré visible. L'application ne plante pas. Pour un vrai projet, on ajouterait le paquet `cached_network_image` ; ce n'est pas le sujet de ce chapitre.

### 55.8.6 — État attendu

```text
┌─────────────────────────────────┐
│  Cartes de profil               │
├─────────────────────────────────┤
│  ┌───────────────────────────┐  │
│  │                           │  │
│  │          ,-----.          │  │
│  │         (   AL  )         │  │
│  │          `-----'          │  │
│  │                           │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

Un disque doré (rang Or), entouré d'un anneau vert (joueur en ligne), portant les lettres « AL ».

**L'application compile et se lance.** Quatrième étape terminée.

---

## 55.9 — Étape 5 : le nom, le titre et la guilde

### 55.9.1 — La hiérarchie typographique

Trois textes vont cohabiter. S'ils ont la même taille et la même couleur, l'œil ne sait pas où se poser. Il faut une **hiérarchie**, et elle se règle avec le `textTheme` du chapitre 51.

```text
Aria Lumen                 titleLarge, gras, onSurface        <- le plus fort
Archimage du Vent          bodyMedium, onSurfaceVariant       <- moyen
Guilde des Sentinelles     bodySmall, onSurfaceVariant, icône <- le plus faible
```

Trois niveaux suffisent. Au-delà, ça devient du bruit.

### 55.9.2 — Le piège du texte long

« Guilde des Sentinelles » tient sur une ligne. Mais rien ne garantit qu'un autre nom tiendra. Deux protections systématiques, vues au chapitre 47 :

- `maxLines: 1` et `overflow: TextOverflow.ellipsis` sur le nom ;
- un `Expanded` autour de la colonne de textes, pour qu'elle sache **quelle largeur** elle a le droit d'occuper.

> Sans `Expanded`, une `Column` de textes placée dans une `Row` reçoit une largeur maximale infinie, et `TextOverflow.ellipsis` ne peut rien couper. C'est l'erreur la plus fréquente de tout le chapitre 46.

### 55.9.3 — Le code

`lib/main.dart` (seul le `build` de `CarteProfil` change ; le reste du fichier est identique à 55.8)

```dart
class CarteProfil extends StatelessWidget {
  const CarteProfil({super.key, required this.profil});

  final Profil profil;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme schema = theme.colorScheme;
    final Color accent = couleurDuRang(profil.rang);

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(Espace.l),
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            // ----- 1. l'avatar (étape 55.8) -----
            Container(
              padding: const EdgeInsets.all(3),
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: profil.enLigne ? vertEnLigne : theme.cardTheme.color,
              ),
              child: CircleAvatar(
                radius: 28,
                backgroundColor: accent,
                foregroundColor: Colors.white,
                child: Text(
                  profil.initiales,
                  style: const TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.w700,
                  ),
                ),
              ),
            ),
            const SizedBox(width: Espace.m),

            // ----- 2. l'identité -----
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  Text(
                    profil.nom,
                    style: theme.textTheme.titleLarge?.copyWith(
                      fontWeight: FontWeight.w700,
                    ),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: Espace.xs),
                  Text(
                    profil.titre,
                    style: theme.textTheme.bodyMedium?.copyWith(
                      color: schema.onSurfaceVariant,
                    ),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: Espace.xs),
                  Row(
                    children: <Widget>[
                      Icon(
                        Icons.shield_outlined,
                        size: 14,
                        color: schema.onSurfaceVariant,
                      ),
                      const SizedBox(width: Espace.xs),
                      // Flexible et non Expanded : le texte prend ce dont il a
                      // besoin, mais pas plus que la place restante.
                      Flexible(
                        child: Text(
                          profil.guilde,
                          style: theme.textTheme.bodySmall?.copyWith(
                            color: schema.onSurfaceVariant,
                          ),
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),

            // ----- 3. le statut -----
            const SizedBox(width: Espace.s),
            _statut(theme, schema),
          ],
        ),
      ),
    );
  }

  /// Pastille « En ligne » / « Hors ligne ».
  ///
  /// C'est une méthode privée, pas encore un widget : nous corrigerons ce
  /// choix en 55.15, et nous expliquerons pourquoi il est mauvais.
  Widget _statut(ThemeData theme, ColorScheme schema) {
    final bool enLigne = profil.enLigne;
    final Color couleur = enLigne ? vertEnLigne : schema.outline;

    return Container(
      padding: const EdgeInsets.symmetric(
        horizontal: Espace.s,
        vertical: Espace.xs,
      ),
      decoration: BoxDecoration(
        color: couleur.withValues(alpha: 0.12),
        borderRadius: BorderRadius.circular(Rayon.petit),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Container(
            width: 8,
            height: 8,
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              color: couleur,
            ),
          ),
          const SizedBox(width: Espace.xs),
          Text(
            enLigne ? 'En ligne' : 'Hors ligne',
            style: theme.textTheme.labelSmall?.copyWith(color: couleur),
          ),
        ],
      ),
    );
  }
}
```

### 55.9.4 — Pourquoi `mainAxisSize: MainAxisSize.min` sur la pastille

Un `Row` occupe par défaut **toute** la largeur qu'on lui autorise. Dans un `Container` qui doit épouser son contenu, cela produirait une pastille étirée sur toute la carte.

`MainAxisSize.min` dit : « fais-toi aussi petit que tes enfants ». C'est la règle du chapitre 46 : dès qu'un `Row` ou une `Column` sert de contenu à une boîte décorée, mettez `MainAxisSize.min`.

### 55.9.5 — État attendu

```text
┌───────────────────────────────────────────────┐
│  ,---.  Aria Lumen                 ( ● En ligne )
│ ( AL  ) Archimage du Vent                     │
│  `---'  # Guilde des Sentinelles              │
└───────────────────────────────────────────────┘
```

Testez tout de suite le cas critique : remplacez `profilsDemo.first` par `profilsDemo[3]` (Doran Fisk, hors ligne, sans badge). La pastille doit devenir grise et afficher « Hors ligne ».

**L'application compile et se lance.** Cinquième étape terminée.

---

## 55.10 — Étape 6 : la bannière avec dégradé

### 55.10.1 — Ce qu'on veut obtenir

```text
   ┌───────────────────────────────────────────┐
   │////////////////////////////////  [ OR ]  /│  <- bannière, 96 px de haut
   │///////////  dégradé  /////////////////////│
   │  ,-----.  ////////////////////////////////│
   ├─(  AL   )─────────────────────────────────┤  <- l'avatar est À CHEVAL
   │  `-----'                                  │
   │  Aria Lumen                               │
```

Deux difficultés distinctes :

1. dessiner un **dégradé** ;
2. faire **déborder** l'avatar en dehors de la bannière.

### 55.10.2 — Le dégradé

Un dégradé se déclare dans la `decoration` d'un `Container`, avec `LinearGradient` (chapitre 46) :

```dart
Container(
  height: 96,
  decoration: BoxDecoration(
    gradient: LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: <Color>[accent, schema.primary],
    ),
  ),
)
```

`begin` et `end` définissent la direction. Les valeurs les plus utiles :

| `begin` → `end` | Effet |
| --- | --- |
| `topCenter` → `bottomCenter` | dégradé vertical |
| `centerLeft` → `centerRight` | dégradé horizontal |
| `topLeft` → `bottomRight` | dégradé en diagonale (le nôtre) |

> **Piège :** on ne met **jamais** `color:` et `gradient:` dans le même `BoxDecoration`. Flutter lève une assertion. Le dégradé remplace la couleur.

### 55.10.3 — L'avatar à cheval : `Stack` + `Positioned`

C'est le cœur de l'étape. Le raisonnement, en trois temps :

```text
1. On empile la bannière et l'avatar dans un Stack.

2. On donne au Stack une hauteur PLUS GRANDE que la bannière,
   en ajoutant sous celle-ci un Padding du bas égal à la moitié
   de la hauteur de l'avatar.

        Stack (hauteur = 96 + 36 = 132)
        ┌──────────────────────────────┐  0
        │ bannière (96)                │
        │                              │
        │        ,-----.               │  60
        ├────────(  AL  )──────────────┤  96
        │        `-----'               │
        └──────────────────────────────┘  132
                                          l'avatar (72 de diamètre)
                                          est posé en bas : 132-72 = 60

3. On place l'avatar avec Positioned(left: 16, bottom: 0).
```

Pourquoi ne pas simplement sortir l'avatar du `Stack` avec une marge négative ? Parce que **les marges négatives n'existent pas en Flutter**. `EdgeInsets.only(top: -20)` déclenche une assertion. La superposition se fait avec `Stack`, toujours.

### 55.10.4 — Le code

`lib/main.dart` (le `build` de `CarteProfil` ; le reste du fichier est inchangé)

```dart
  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme schema = theme.colorScheme;
    final Color accent = couleurDuRang(profil.rang);

    // Hauteur de la bannière et rayon de l'avatar, nommés une seule fois :
    // les trois calculs qui suivent en dépendent.
    const double hauteurBanniere = 96;
    const double rayonAvatar = 36;

    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          // ================= BANNIÈRE + AVATAR =================
          Stack(
            children: <Widget>[
              // La bannière, avec sous elle la place de la moitié de l'avatar.
              Padding(
                padding: const EdgeInsets.only(bottom: rayonAvatar),
                child: Container(
                  height: hauteurBanniere,
                  width: double.infinity,
                  decoration: BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.topLeft,
                      end: Alignment.bottomRight,
                      colors: <Color>[accent, schema.primary],
                    ),
                  ),
                  // La pastille de rang, en haut à droite de la bannière.
                  child: Align(
                    alignment: Alignment.topRight,
                    child: Padding(
                      padding: const EdgeInsets.all(Espace.m),
                      child: Container(
                        padding: const EdgeInsets.symmetric(
                          horizontal: Espace.s,
                          vertical: Espace.xs,
                        ),
                        decoration: BoxDecoration(
                          color: Colors.black.withValues(alpha: 0.25),
                          borderRadius: BorderRadius.circular(Rayon.petit),
                        ),
                        child: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: <Widget>[
                            const Icon(
                              Icons.workspace_premium,
                              size: 14,
                              color: Colors.white,
                            ),
                            const SizedBox(width: Espace.xs),
                            Text(
                              profil.rang.libelle.toUpperCase(),
                              style: const TextStyle(
                                color: Colors.white,
                                fontSize: 11,
                                fontWeight: FontWeight.w700,
                                letterSpacing: 0.8,
                              ),
                            ),
                          ],
                        ),
                      ),
                    ),
                  ),
                ),
              ),

              // L'avatar, posé en bas à gauche du Stack.
              Positioned(
                left: Espace.l,
                bottom: 0,
                child: Container(
                  padding: const EdgeInsets.all(3),
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    color: profil.enLigne ? vertEnLigne : theme.cardTheme.color,
                  ),
                  child: CircleAvatar(
                    radius: rayonAvatar,
                    backgroundColor: accent,
                    foregroundColor: Colors.white,
                    child: Text(
                      profil.initiales,
                      style: const TextStyle(
                        fontSize: rayonAvatar * 0.7,
                        fontWeight: FontWeight.w700,
                        letterSpacing: 1,
                      ),
                    ),
                  ),
                ),
              ),
            ],
          ),

          // ================= IDENTITÉ =================
          // Exactement le bloc écrit en 55.9.3 : le Row contenant la colonne
          // nom / titre / guilde et l'appel à _statut(theme, schema).
          // Seule différence : il n'est plus dans la Row de l'avatar, mais
          // sous le Stack, enveloppé dans le Padding ci-dessous.
          Padding(
            padding: const EdgeInsets.fromLTRB(
              Espace.l,
              Espace.m,
              Espace.l,
              Espace.l,
            ),
            child: Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: <Widget>[
                      // ... les trois Text de 55.9.3, inchangés ...
                    ],
                  ),
                ),
                const SizedBox(width: Espace.s),
                _statut(theme, schema),
              ],
            ),
          ),
        ],
      ),
    );
  }
```

Et l'écran de démonstration doit maintenant laisser la carte occuper toute la largeur :

```dart
class EcranDemo extends StatelessWidget {
  const EcranDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cartes de profil')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(Espace.l),
        child: CarteProfil(profil: profilsDemo.first),
      ),
    );
  }
}
```

### 55.10.5 — Vérification indispensable : les coins

Regardez les **coins supérieurs** de la carte. Si le dégradé forme des angles droits alors que la carte est arrondie, c'est que le rognage manque.

```text
        SANS clipBehavior                AVEC clipBehavior
        ┌───────────────┐                 ╭───────────────╮
        │///////////////│                 │///////////////│
        │///////////////│                 │///////////////│
```

Nous l'avons déjà réglé en 55.6 dans le `cardTheme` :

```dart
clipBehavior: Clip.antiAlias,
```

Si vous l'aviez oublié, vous pouvez aussi l'écrire sur la carte elle-même : `Card(clipBehavior: Clip.antiAlias, ...)`. Retenez la règle générale :

> Dès qu'un enfant coloré touche le bord d'un conteneur arrondi, il faut rogner. Sinon il dépasse.

### 55.10.6 — État attendu

```text
╭───────────────────────────────────────────────╮
│///////////////////////////////////[ * OR ]////│
│///////////////  dégradé  /////////////////////│
│   ,-----.  ////////////////////////////////////
│  (  AL   )                                    │
│   `-----'                                     │
│  Aria Lumen                      ( ● En ligne)│
│  Archimage du Vent                            │
│  # Guilde des Sentinelles                     │
╰───────────────────────────────────────────────╯
```

**L'application compile et se lance.** Sixième étape terminée.

---

## 55.11 — Étape 7 : les statistiques avec séparateurs

### 55.11.1 — Le problème des séparateurs verticaux

Trois chiffres, deux traits entre eux, le tout réparti sur la largeur.

```text
      128      │      17      │      43
   Victoires   │   Défaites   │    Quêtes
```

Chaque colonne prend un tiers : c'est `Expanded` (chapitre 46). Mais le trait, lui, pose une difficulté connue :

> Un `VerticalDivider` placé dans un `Row` n'a **aucune hauteur naturelle**. Il prend toute la hauteur qu'on lui autorise, ce qui donne soit un trait invisible, soit un trait démesuré.

La solution du chapitre 46, section 46.36 : `IntrinsicHeight`. Il mesure d'abord la hauteur du contenu le plus haut, puis impose cette hauteur au `Row`. Combiné à `CrossAxisAlignment.stretch`, le trait s'arrête pile en bas du texte.

```text
   IntrinsicHeight
   ┌───────────────────────────────────────┐
   │  ┌────────┐ │ ┌────────┐ │ ┌────────┐ │
   │  │  128   │ │ │   17   │ │ │   43   │ │   <- même hauteur imposée
   │  │Victoire│ │ │Défaites│ │ │ Quêtes │ │
   │  └────────┘ │ └────────┘ │ └────────┘ │
   └───────────────────────────────────────┘
```

`IntrinsicHeight` coûte une passe de mesure supplémentaire. Sur trois colonnes de texte, c'est négligeable. Ne l'imbriquez jamais dans un autre `IntrinsicHeight`.

### 55.11.2 — Le code

Ajoutez ces deux méthodes à `CarteProfil`, dans `lib/main.dart` :

```dart
  /// La ligne des trois statistiques.
  Widget _statistiques(ThemeData theme, ColorScheme schema) {
    return IntrinsicHeight(
      child: Row(
        // stretch : chaque enfant reçoit la hauteur mesurée par IntrinsicHeight.
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          Expanded(
            child: _uneStat(theme, schema, '${profil.victoires}', 'Victoires'),
          ),
          const VerticalDivider(width: Espace.l, thickness: 1),
          Expanded(
            child: _uneStat(theme, schema, '${profil.defaites}', 'Défaites'),
          ),
          const VerticalDivider(width: Espace.l, thickness: 1),
          Expanded(
            child: _uneStat(
              theme,
              schema,
              '${profil.quetesTerminees}',
              'Quêtes',
            ),
          ),
        ],
      ),
    );
  }

  /// Une colonne : un grand nombre, un petit libellé.
  Widget _uneStat(
    ThemeData theme,
    ColorScheme schema,
    String valeur,
    String libelle,
  ) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        Text(
          valeur,
          textAlign: TextAlign.center,
          style: theme.textTheme.titleMedium?.copyWith(
            fontWeight: FontWeight.w700,
          ),
        ),
        const SizedBox(height: 2),
        Text(
          libelle,
          textAlign: TextAlign.center,
          maxLines: 1,
          overflow: TextOverflow.ellipsis,
          style: theme.textTheme.labelSmall?.copyWith(
            color: schema.onSurfaceVariant,
          ),
        ),
      ],
    );
  }
```

Puis, dans le `build`, transformez la zone d'identité en une `Column` et ajoutez les statistiques dessous. Remplacez `child: Row( ... _statut(theme, schema) ], ),` par :

```dart
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Row(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    // ... la colonne nom / titre / guilde, inchangée ...
                    const SizedBox(width: Espace.s),
                    _statut(theme, schema),
                  ],
                ),
                const SizedBox(height: Espace.l),
                _statistiques(theme, schema),
              ],
            ),
```

### 55.11.3 — État attendu

```text
│  Aria Lumen                       ( ● En ligne)│
│  Archimage du Vent                             │
│  # Guilde des Sentinelles                      │
│                                                │
│     128       │      17       │       43       │
│   Victoires   │   Défaites    │     Quêtes     │
```

**L'application compile et se lance.** Septième étape terminée.

---

## 55.12 — Étape 8 : les badges en `Wrap`

### 55.12.1 — Pourquoi `Wrap` et pas `Row`

Cyra Vale possède six badges. Dans un `Row`, ils dépasseraient, et Flutter afficherait la bande jaune et noire `RenderFlex overflowed by 84 pixels`.

`Wrap` (chapitre 46) place ses enfants côte à côte **et passe à la ligne** dès que la place manque.

```text
   Row   :  [Feu][Vent][Soin][Bouclier][Vété//////  <- déborde
   Wrap  :  [Feu][Vent][Soin][Bouclier]
            [Vétéran]                              <- passe à la ligne
```

Deux réglages, et deux seulement :

| Paramètre | Rôle |
| --- | --- |
| `spacing` | espace **horizontal** entre deux badges d'une même ligne |
| `runSpacing` | espace **vertical** entre deux lignes |

Oublier `runSpacing` est l'erreur classique : les lignes se touchent.

### 55.12.2 — Une icône par badge

Chaque badge affiche une icône. Une `Map` constante suffit à faire la correspondance, avec une icône de secours si le nom est inconnu (chapitre 12 : l'opérateur `??`).

Ajoutez ceci à la fin de `lib/theme/theme_application.dart` :

```dart
/// Icône associée à chaque nom de badge.
const Map<String, IconData> _iconesBadges = <String, IconData>{
  'Feu': Icons.local_fire_department,
  'Vent': Icons.air,
  'Soin': Icons.healing,
  'Bouclier': Icons.shield,
  'Vétéran': Icons.military_tech,
  'Force': Icons.fitness_center,
  'Rage': Icons.bolt,
  'Endurance': Icons.favorite,
  'Vitesse': Icons.speed,
  'Discrétion': Icons.visibility_off,
  'Poison': Icons.science,
  'Champion': Icons.emoji_events,
  'Explorateur': Icons.explore,
  'Nature': Icons.eco,
  'Lumière': Icons.light_mode,
};

/// Renvoie l'icône d'un badge, ou une étoile si le badge est inconnu.
IconData iconeDuBadge(String nom) => _iconesBadges[nom] ?? Icons.star;
```

### 55.12.3 — Le code

Ajoutez ces deux méthodes à `CarteProfil` :

```dart
  /// La zone des badges. Renvoie une boîte vide si le joueur n'en a aucun.
  Widget _badges(ThemeData theme, ColorScheme schema) {
    if (profil.badges.isEmpty) {
      // SizedBox.shrink() est le widget « rien du tout » : zéro pixel.
      return const SizedBox.shrink();
    }

    return Wrap(
      spacing: Espace.s,
      runSpacing: Espace.s,
      // map + toList : la transformation de collection du chapitre 14.
      children: profil.badges
          .map((String nom) => _unBadge(theme, schema, nom))
          .toList(),
    );
  }

  /// Une pastille : une icône et un nom.
  Widget _unBadge(ThemeData theme, ColorScheme schema, String nom) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: Espace.m, vertical: 6),
      decoration: BoxDecoration(
        color: schema.secondaryContainer,
        borderRadius: BorderRadius.circular(Rayon.petit),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Icon(
            iconeDuBadge(nom),
            size: 14,
            color: schema.onSecondaryContainer,
          ),
          const SizedBox(width: Espace.xs),
          Text(
            nom,
            style: theme.textTheme.labelMedium?.copyWith(
              color: schema.onSecondaryContainer,
            ),
          ),
        ],
      ),
    );
  }
```

Et dans le `build`, sous les statistiques :

```dart
                const SizedBox(height: Espace.l),
                _badges(theme, schema),
```

### 55.12.4 — État attendu

Affichez successivement `profilsDemo[2]` (six badges : deux lignes) et `profilsDemo[3]` (aucun badge : rien ne s'affiche, et surtout **pas** un espace vide).

```text
│   [ Feu ]  [ Vent ]  [ Soin ]  [ Bouclier ]    │
│   [ Vétéran ]                                  │
```

**L'application compile et se lance.** Huitième étape terminée.

---

## 55.13 — Étape 9 : la barre de progression de niveau

### 55.13.1 — Le widget

`LinearProgressIndicator` attend une `value` entre 0.0 et 1.0. Notre modèle la fournit déjà, bornée : c'est le getter `progression` écrit en 55.7.

Les paramètres utiles :

| Paramètre | Rôle |
| --- | --- |
| `value` | avancement, de 0.0 à 1.0 ; `null` donnerait une animation infinie |
| `minHeight` | épaisseur de la barre |
| `backgroundColor` | la partie non remplie |
| `color` | la partie remplie |
| `borderRadius` | arrondi de la barre |
| `stopIndicatorRadius` | rayon du point de fin dessiné par Material 3 ; `0` le supprime |

### 55.13.2 — Le code

Ajoutez cette méthode à `CarteProfil` :

```dart
  /// Le niveau et sa barre de progression.
  Widget _barreNiveau(ThemeData theme, ColorScheme schema, Color accent) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Row(
          children: <Widget>[
            Text(
              'Niveau ${profil.niveau}',
              style: theme.textTheme.labelLarge,
            ),
            // Spacer pousse le second texte tout à droite (chapitre 46).
            const Spacer(),
            Text(
              '${profil.experience} / ${profil.experienceRequise} XP',
              style: theme.textTheme.labelSmall?.copyWith(
                color: schema.onSurfaceVariant,
              ),
            ),
          ],
        ),
        const SizedBox(height: Espace.s),
        LinearProgressIndicator(
          value: profil.progression,
          minHeight: 10,
          backgroundColor: schema.surfaceContainerHighest,
          color: accent,
          borderRadius: BorderRadius.circular(Rayon.petit),
          stopIndicatorRadius: 0,
          // Lu par les lecteurs d'écran : l'accessibilité n'est pas optionnelle.
          semanticsLabel: 'Progression du niveau',
        ),
      ],
    );
  }
```

Et dans le `build`, sous les badges :

```dart
                const SizedBox(height: Espace.l),
                _barreNiveau(theme, schema, accent),
```

### 55.13.3 — État attendu

```text
│   Niveau 24                        820 / 1000 XP│
│   ████████████████████░░░░░░░░░░░░░░░░░░░░░░░░  │
```

Vérifiez la proportion : 820 sur 1000 doit remplir un peu plus des quatre cinquièmes. Testez aussi `profilsDemo[3]` (60 sur 300) : un cinquième seulement.

**L'application compile et se lance.** Neuvième étape terminée.

---

## 55.14 — Étape 10 : les boutons d'action

### 55.14.1 — Deux boutons qui ne débordent jamais

Deux boutons côte à côte, de largeurs égales, avec un espace au milieu. La recette est toujours la même :

```text
   Row
   ├── Expanded ──> FilledButton    (action principale, pleine couleur)
   ├── SizedBox(width: 12)
   └── Expanded ──> OutlinedButton  (action secondaire, contour seul)
```

`Expanded` garantit qu'aucun libellé, même long, ne provoquera de débordement : chaque bouton reçoit exactement la moitié de la largeur disponible.

La hiérarchie visuelle vient du chapitre 49 : un seul bouton plein par écran ou par carte, le reste en contour.

### 55.14.2 — Le code

Ajoutez cette méthode à `CarteProfil` :

```dart
  /// Les deux boutons d'action.
  ///
  /// `context` est nécessaire pour afficher un `SnackBar` (chapitre 49).
  Widget _actions(BuildContext context) {
    return Row(
      children: <Widget>[
        Expanded(
          child: FilledButton.icon(
            onPressed: () => _message(context, 'Défi envoyé à ${profil.nom}'),
            icon: const Icon(Icons.sports_kabaddi, size: 18),
            label: const Text('Défier'),
          ),
        ),
        const SizedBox(width: Espace.m),
        Expanded(
          child: OutlinedButton.icon(
            onPressed: () => _message(context, 'Message à ${profil.nom}'),
            icon: const Icon(Icons.chat_bubble_outline, size: 18),
            label: const Text('Message'),
          ),
        ),
      ],
    );
  }

  void _message(BuildContext context, String texte) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(texte)),
    );
  }
```

Et dans le `build`, sous la barre de niveau :

```dart
                const SizedBox(height: Espace.l),
                _actions(context),
```

### 55.14.3 — État attendu

```text
│   ┌────────────────────┐  ┌────────────────────┐│
│   │      Défier        │  │      Message       ││
│   └────────────────────┘  └────────────────────┘│
```

Cliquez : un `SnackBar` flottant apparaît en bas de l'écran.

**La carte est visuellement terminée.** Dixième étape terminée.

---

## 55.15 — Étape 11 : extraire en petits widgets réutilisables

### 55.15.1 — Regardez ce que vous avez écrit

Votre `main.dart` fait maintenant plus de 300 lignes, dont un `build` de 150 lignes et cinq méthodes privées. Trois symptômes doivent vous alerter :

- vous devez faire défiler pour voir le début d'un widget dont vous lisez la fin ;
- vous passez `theme` et `schema` en paramètre à chaque méthode ;
- rien n'est réutilisable ailleurs, alors que l'avatar, lui, servira dans trois écrans.

### 55.15.2 — Méthode privée ou classe de widget ?

C'est la question centrale de cette étape, et le chapitre 44 y a répondu.

| | Méthode `Widget _truc()` | Classe `class Truc extends StatelessWidget` |
| --- | --- | --- |
| Écriture | rapide | un peu plus longue |
| Peut être `const` | non | oui |
| Reconstruite quand le parent change | **toujours** | seulement si ses paramètres changent |
| Apparaît dans l'inspecteur de widgets | non | oui |
| Réutilisable ailleurs | non | oui |

> **Règle :** une méthode privée est acceptable pour trois lignes jamais réutilisées. Dès qu'un morceau a un nom, une identité et plus de dix lignes, c'est une **classe**.

Nous passons donc de un fichier à huit.

### 55.15.3 — `lib/widgets/avatar_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/theme_application.dart';

/// Avatar circulaire : la photo du profil si elle existe, ses initiales sinon.
///
/// L'anneau extérieur est vert quand le joueur est en ligne.
class AvatarProfil extends StatelessWidget {
  const AvatarProfil({super.key, required this.profil, this.rayon = 36});

  final Profil profil;

  /// Rayon du disque. La taille du texte en découle : aucun réglage à faire.
  final double rayon;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final bool avecPhoto = profil.urlAvatar != null;

    return Container(
      padding: const EdgeInsets.all(3),
      decoration: BoxDecoration(
        shape: BoxShape.circle,
        color: profil.enLigne ? vertEnLigne : theme.cardTheme.color,
      ),
      child: CircleAvatar(
        radius: rayon,
        backgroundColor: couleurDuRang(profil.rang),
        foregroundColor: Colors.white,
        backgroundImage: avecPhoto ? NetworkImage(profil.urlAvatar!) : null,
        child: avecPhoto
            ? null
            : Text(
                profil.initiales,
                style: TextStyle(
                  fontSize: rayon * 0.7,
                  fontWeight: FontWeight.w700,
                  letterSpacing: 1,
                ),
              ),
      ),
    );
  }
}
```

### 55.15.4 — `lib/widgets/banniere_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import '../theme/theme_application.dart';

/// Bandeau en dégradé, avec la pastille de rang en haut à droite.
class BanniereProfil extends StatelessWidget {
  const BanniereProfil({super.key, required this.profil, this.hauteur = 96});

  final Profil profil;
  final double hauteur;

  @override
  Widget build(BuildContext context) {
    final ColorScheme schema = Theme.of(context).colorScheme;

    return Container(
      height: hauteur,
      width: double.infinity,
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: <Color>[couleurDuRang(profil.rang), schema.primary],
        ),
      ),
      child: Align(
        alignment: Alignment.topRight,
        child: Padding(
          padding: const EdgeInsets.all(Espace.m),
          child: _PastilleRang(rang: profil.rang),
        ),
      ),
    );
  }
}

/// Pastille sombre translucide portant le nom du rang.
///
/// Privée : elle n'a aucun sens en dehors de la bannière.
class _PastilleRang extends StatelessWidget {
  const _PastilleRang({required this.rang});

  final Rang rang;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(
        horizontal: Espace.s,
        vertical: Espace.xs,
      ),
      decoration: BoxDecoration(
        color: Colors.black.withValues(alpha: 0.25),
        borderRadius: BorderRadius.circular(Rayon.petit),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          const Icon(Icons.workspace_premium, size: 14, color: Colors.white),
          const SizedBox(width: Espace.xs),
          Text(
            rang.libelle.toUpperCase(),
            style: const TextStyle(
              color: Colors.white,
              fontSize: 11,
              fontWeight: FontWeight.w700,
              letterSpacing: 0.8,
            ),
          ),
        ],
      ),
    );
  }
}
```

### 55.15.5 — `lib/widgets/identite_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import '../theme/theme_application.dart';

/// Nom, titre, guilde, et pastille de statut.
class IdentiteProfil extends StatelessWidget {
  const IdentiteProfil({super.key, required this.profil});

  final Profil profil;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme schema = theme.colorScheme;

    return Row(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Expanded(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              Text(
                profil.nom,
                style: theme.textTheme.titleLarge?.copyWith(
                  fontWeight: FontWeight.w700,
                ),
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
              const SizedBox(height: Espace.xs),
              Text(
                profil.titre,
                style: theme.textTheme.bodyMedium?.copyWith(
                  color: schema.onSurfaceVariant,
                ),
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
              const SizedBox(height: Espace.xs),
              Row(
                children: <Widget>[
                  Icon(
                    Icons.shield_outlined,
                    size: 14,
                    color: schema.onSurfaceVariant,
                  ),
                  const SizedBox(width: Espace.xs),
                  Flexible(
                    child: Text(
                      profil.guilde,
                      style: theme.textTheme.bodySmall?.copyWith(
                        color: schema.onSurfaceVariant,
                      ),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
        const SizedBox(width: Espace.s),
        _Statut(enLigne: profil.enLigne),
      ],
    );
  }
}

/// Pastille « En ligne » / « Hors ligne ».
class _Statut extends StatelessWidget {
  const _Statut({required this.enLigne});

  final bool enLigne;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Color couleur = enLigne ? vertEnLigne : theme.colorScheme.outline;

    return Container(
      padding: const EdgeInsets.symmetric(
        horizontal: Espace.s,
        vertical: Espace.xs,
      ),
      decoration: BoxDecoration(
        color: couleur.withValues(alpha: 0.12),
        borderRadius: BorderRadius.circular(Rayon.petit),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Container(
            width: 8,
            height: 8,
            decoration: BoxDecoration(shape: BoxShape.circle, color: couleur),
          ),
          const SizedBox(width: Espace.xs),
          Text(
            enLigne ? 'En ligne' : 'Hors ligne',
            style: theme.textTheme.labelSmall?.copyWith(color: couleur),
          ),
        ],
      ),
    );
  }
}
```

### 55.15.6 — `lib/widgets/statistiques_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';

/// Les trois statistiques, séparées par des traits verticaux.
class StatistiquesProfil extends StatelessWidget {
  const StatistiquesProfil({super.key, required this.profil});

  final Profil profil;

  @override
  Widget build(BuildContext context) {
    return IntrinsicHeight(
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          Expanded(
            child: _Stat(valeur: '${profil.victoires}', libelle: 'Victoires'),
          ),
          const VerticalDivider(width: Espace.l, thickness: 1),
          Expanded(
            child: _Stat(valeur: '${profil.defaites}', libelle: 'Défaites'),
          ),
          const VerticalDivider(width: Espace.l, thickness: 1),
          Expanded(
            child: _Stat(
              valeur: '${profil.quetesTerminees}',
              libelle: 'Quêtes',
            ),
          ),
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
    final ThemeData theme = Theme.of(context);

    return Column(
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        Text(
          valeur,
          textAlign: TextAlign.center,
          style: theme.textTheme.titleMedium?.copyWith(
            fontWeight: FontWeight.w700,
          ),
        ),
        const SizedBox(height: 2),
        Text(
          libelle,
          textAlign: TextAlign.center,
          maxLines: 1,
          overflow: TextOverflow.ellipsis,
          style: theme.textTheme.labelSmall?.copyWith(
            color: theme.colorScheme.onSurfaceVariant,
          ),
        ),
      ],
    );
  }
}
```

### 55.15.7 — `lib/widgets/badges_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../theme/dimensions.dart';
import '../theme/theme_application.dart';

/// Liste de badges qui passe automatiquement à la ligne.
///
/// Ce widget ne reçoit PAS un `Profil` mais une `List<String>` : il est ainsi
/// réutilisable pour n'importe quelle liste d'étiquettes.
class BadgesProfil extends StatelessWidget {
  const BadgesProfil({super.key, required this.badges});

  final List<String> badges;

  @override
  Widget build(BuildContext context) {
    if (badges.isEmpty) {
      return const SizedBox.shrink();
    }

    return Wrap(
      spacing: Espace.s,
      runSpacing: Espace.s,
      children: badges.map((String nom) => _Badge(nom: nom)).toList(),
    );
  }
}

class _Badge extends StatelessWidget {
  const _Badge({required this.nom});

  final String nom;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme schema = theme.colorScheme;

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: Espace.m, vertical: 6),
      decoration: BoxDecoration(
        color: schema.secondaryContainer,
        borderRadius: BorderRadius.circular(Rayon.petit),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Icon(iconeDuBadge(nom), size: 14, color: schema.onSecondaryContainer),
          const SizedBox(width: Espace.xs),
          Text(
            nom,
            style: theme.textTheme.labelMedium?.copyWith(
              color: schema.onSecondaryContainer,
            ),
          ),
        ],
      ),
    );
  }
}
```

### 55.15.8 — `lib/widgets/barre_niveau.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import '../theme/theme_application.dart';

/// Niveau du joueur et barre de progression vers le niveau suivant.
class BarreNiveau extends StatelessWidget {
  const BarreNiveau({super.key, required this.profil});

  final Profil profil;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final ColorScheme schema = theme.colorScheme;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Row(
          children: <Widget>[
            Text('Niveau ${profil.niveau}', style: theme.textTheme.labelLarge),
            const Spacer(),
            Text(
              '${profil.experience} / ${profil.experienceRequise} XP',
              style: theme.textTheme.labelSmall?.copyWith(
                color: schema.onSurfaceVariant,
              ),
            ),
          ],
        ),
        const SizedBox(height: Espace.s),
        LinearProgressIndicator(
          value: profil.progression,
          minHeight: 10,
          backgroundColor: schema.surfaceContainerHighest,
          color: couleurDuRang(profil.rang),
          borderRadius: BorderRadius.circular(Rayon.petit),
          stopIndicatorRadius: 0,
          semanticsLabel: 'Progression du niveau',
        ),
      ],
    );
  }
}
```

### 55.15.9 — `lib/widgets/actions_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../theme/dimensions.dart';

/// Les deux boutons d'action de la carte.
///
/// Le widget ne DÉCIDE de rien : il se contente de prévenir son appelant.
/// Les deux rappels sont nullables ; un rappel nul désactive son bouton,
/// ce que Flutter signale visuellement tout seul.
class ActionsProfil extends StatelessWidget {
  const ActionsProfil({super.key, this.onDefier, this.onMessage});

  final VoidCallback? onDefier;
  final VoidCallback? onMessage;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: <Widget>[
        Expanded(
          child: FilledButton.icon(
            onPressed: onDefier,
            icon: const Icon(Icons.sports_kabaddi, size: 18),
            label: const Text('Défier'),
          ),
        ),
        const SizedBox(width: Espace.m),
        Expanded(
          child: OutlinedButton.icon(
            onPressed: onMessage,
            icon: const Icon(Icons.chat_bubble_outline, size: 18),
            label: const Text('Message'),
          ),
        ),
      ],
    );
  }
}
```

### 55.15.10 — `lib/widgets/carte_profil.dart`

L'assemblage. Remarquez sa taille : moins de cinquante lignes, alors qu'il décrit la carte entière.

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import 'actions_profil.dart';
import 'avatar_profil.dart';
import 'badges_profil.dart';
import 'banniere_profil.dart';
import 'barre_niveau.dart';
import 'identite_profil.dart';
import 'statistiques_profil.dart';

/// Carte de profil complète.
class CarteProfil extends StatelessWidget {
  const CarteProfil({super.key, required this.profil});

  final Profil profil;

  static const double _hauteurBanniere = 96;
  static const double _rayonAvatar = 36;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Stack(
            children: <Widget>[
              Padding(
                padding: const EdgeInsets.only(bottom: _rayonAvatar),
                child: BanniereProfil(
                  profil: profil,
                  hauteur: _hauteurBanniere,
                ),
              ),
              Positioned(
                left: Espace.l,
                bottom: 0,
                child: AvatarProfil(profil: profil, rayon: _rayonAvatar),
              ),
            ],
          ),
          Padding(
            padding: const EdgeInsets.fromLTRB(
              Espace.l,
              Espace.m,
              Espace.l,
              Espace.l,
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                IdentiteProfil(profil: profil),
                const SizedBox(height: Espace.l),
                StatistiquesProfil(profil: profil),
                if (profil.badges.isNotEmpty) ...<Widget>[
                  const SizedBox(height: Espace.l),
                  BadgesProfil(badges: profil.badges),
                ],
                const SizedBox(height: Espace.l),
                BarreNiveau(profil: profil),
                const SizedBox(height: Espace.l),
                const ActionsProfil(),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

Deux points de syntaxe, revus du chapitre 06 :

- `if (condition) ...<Widget>[ ... ]` est un `if` **dans une liste**, suivi d'un opérateur d'étalement. Si la condition est fausse, rien n'est ajouté : ni le `SizedBox`, ni les badges. C'est plus propre qu'un `SizedBox.shrink()`.
- `const ActionsProfil()` : sans rappel, les deux boutons apparaissent désactivés. Nous les branchons à l'étape suivante.

### 55.15.11 — Le nouveau `main.dart`

```dart
import 'package:flutter/material.dart';

import 'donnees/profils_demo.dart';
import 'theme/dimensions.dart';
import 'theme/theme_application.dart';
import 'widgets/carte_profil.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

class ApplicationCarteProfil extends StatelessWidget {
  const ApplicationCarteProfil({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      themeMode: ThemeMode.system,
      home: Scaffold(
        appBar: AppBar(title: const Text('Cartes de profil')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(Espace.l),
          child: CarteProfil(profil: profilsDemo.first),
        ),
      ),
    );
  }
}
```

Votre `main.dart` est passé de 300 lignes à 35. **L'écran est rigoureusement identique.** C'est le signe d'un bon remaniement : le code change, le résultat non.

**L'application compile et se lance.** Onzième étape terminée.

---

## 55.16 — Étape 12 : rendre la carte paramétrable

### 55.16.1 — Trois questions à se poser

Un widget réutilisable se conçoit en répondant à trois questions.

1. **Qu'est-ce qui varie d'un usage à l'autre ?** Ici : le profil, bien sûr, mais aussi le fait d'afficher ou non les badges et les boutons.
2. **Qui décide de ce qui se passe au clic ?** Pas la carte. Une carte affiche ; c'est l'écran qui sait s'il faut envoyer un défi ou ouvrir une conversation.
3. **Que se passe-t-il si le paramètre n'est pas fourni ?** Il doit exister une valeur par défaut raisonnable, sinon le widget est pénible à utiliser.

D'où la signature finale.

### 55.16.2 — Le constructeur final

Dans `lib/widgets/carte_profil.dart`, remplacez le constructeur et les champs par :

```dart
  const CarteProfil({
    super.key,
    required this.profil,
    this.compacte = false,
    this.onTap,
    this.onDefier,
    this.onMessage,
  });

  /// Le profil à afficher. Seul paramètre obligatoire.
  final Profil profil;

  /// En mode compact, ni badges ni boutons : utile en liste dense.
  final bool compacte;

  /// Appelé quand l'utilisateur touche la carte entière.
  final VoidCallback? onTap;

  /// Appelé par le bouton « Défier ».
  final VoidCallback? onDefier;

  /// Appelé par le bouton « Message ».
  final VoidCallback? onMessage;
```

Puis enveloppez le `Column` du `build` dans un `InkWell` et rendez les deux dernières sections conditionnelles :

```dart
    return Card(
      child: InkWell(
        // onTap nul : InkWell ne réagit plus et n'affiche aucun effet d'encre.
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            // ... bannière + avatar, inchangés ...
```

et, dans la colonne intérieure :

```dart
                if (!compacte && profil.badges.isNotEmpty) ...<Widget>[
                  const SizedBox(height: Espace.l),
                  BadgesProfil(badges: profil.badges),
                ],
                const SizedBox(height: Espace.l),
                BarreNiveau(profil: profil),
                if (!compacte) ...<Widget>[
                  const SizedBox(height: Espace.l),
                  ActionsProfil(onDefier: onDefier, onMessage: onMessage),
                ],
```

### 55.16.3 — Pourquoi `VoidCallback?` et pas un `SnackBar` dans la carte

En 55.14, la carte affichait elle-même le `SnackBar`. C'était pratique et c'était une faute d'architecture : la carte décidait du comportement de l'application.

> Un widget d'affichage **remonte** l'événement, il ne le traite pas. C'est le principe de la remontée d'état du chapitre 45.

Bénéfice immédiat : la même carte peut, dans l'écran A, ouvrir une conversation, et dans l'écran B, lancer un combat. Sans modifier une ligne de `carte_profil.dart`.

### 55.16.4 — Utilisation

Dans `main.dart`, remplacez `CarteProfil(profil: profilsDemo.first)` par :

```dart
          child: Builder(
            builder: (BuildContext context) {
              return CarteProfil(
                profil: profilsDemo.first,
                onDefier: () => ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Défi envoyé')),
                ),
                onMessage: () => ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Conversation ouverte')),
                ),
              );
            },
          ),
```

Le `Builder` fournit un `context` situé **sous** le `Scaffold` : sans lui, `ScaffoldMessenger.of(context)` chercherait au-dessus et échouerait. C'est le piège du `context` décrit en 50.3.2.

**L'application compile et se lance.** Douzième étape terminée.

---

## 55.17 — Étape 13 : afficher une liste de profils

### 55.17.1 — `ListView` ou `ListView.builder` ?

Six profils tiendraient dans un simple `ListView(children: ...)`. Mais la règle du chapitre 48 est sans nuance :

> Dès que la liste vient d'une collection, on utilise `ListView.builder`. Il ne construit que les éléments visibles à l'écran.

Avec six cartes, le gain est invisible. Avec six cents, `ListView(children: ...)` construirait six cents cartes complètes au premier affichage, et l'application saccaderait.

### 55.17.2 — L'écran de liste

`lib/ecrans/ecran_liste_profils.dart`

```dart
import 'package:flutter/material.dart';

import '../donnees/profils_demo.dart';
import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import '../widgets/carte_profil.dart';

/// Écran principal : la liste des profils.
class EcranListeProfils extends StatelessWidget {
  const EcranListeProfils({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cartes de profil')),
      body: ListView.builder(
        padding: const EdgeInsets.all(Espace.l),
        itemCount: profilsDemo.length,
        itemBuilder: (BuildContext context, int index) {
          final Profil profil = profilsDemo[index];

          // La marge basse est portée par un Padding, et non par la Card :
          // le cardTheme de 55.6 a mis sa marge à zéro exprès.
          return Padding(
            padding: const EdgeInsets.only(bottom: Espace.l),
            child: CarteProfil(
              profil: profil,
              onTap: () => _message(context, 'Profil de ${profil.nom}'),
              onDefier: () => _message(context, 'Défi envoyé à ${profil.nom}'),
              onMessage: () => _message(context, 'Message à ${profil.nom}'),
            ),
          );
        },
      ),
    );
  }

  void _message(BuildContext context, String texte) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(texte), duration: const Duration(seconds: 1)),
    );
  }
}
```

Le `context` reçu par `itemBuilder` est situé sous le `Scaffold` : aucun `Builder` n'est nécessaire ici, contrairement à 55.16.

### 55.17.3 — Brancher l'écran

Dans `lib/main.dart`, remplacez tout le `home:` par :

```dart
      home: const EcranListeProfils(),
```

sans oublier l'import `import 'ecrans/ecran_liste_profils.dart';` et en supprimant les imports devenus inutiles (`donnees/`, `widgets/`, `theme/dimensions.dart`).

### 55.17.4 — État attendu

Six cartes défilent. Vérifiez les cas particuliers déjà préparés dans les données :

| Profil | Ce qu'il prouve |
| --- | --- |
| Cyra Vale | six badges sur deux lignes, sans débordement |
| Doran Fisk | aucun badge : la carte reste équilibrée |
| Elyn Sorrow | photo réseau au lieu des initiales |
| Brann Korr | statut « Hors ligne » en gris |

**L'application compile et se lance.** Treizième étape terminée.

---

## 55.18 — Étape 14 : le mode sombre

### 55.18.1 — Ce qu'il reste à faire

Le thème sombre existe depuis 55.6, et l'application le suit déjà si le système est en sombre. Il manque le contrôle **manuel**, demandé par l'exigence O10.

Le raisonnement est celui du chapitre 45 : la valeur `themeMode` change au fil du temps, donc elle doit vivre dans l'**état** d'un `StatefulWidget`, et ce widget doit être **au-dessus** du `MaterialApp`.

```text
   ApplicationCarteProfil  (StatefulWidget)   <- détient _sombre
        │
        MaterialApp(themeMode: ...)
            │
            EcranListeProfils(onBasculerTheme: ...)   <- appelle setState
```

L'état descend en paramètre, l'événement remonte en rappel. C'est la remontée d'état, encore.

### 55.18.2 — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

import 'ecrans/ecran_liste_profils.dart';
import 'theme/theme_application.dart';

void main() {
  runApp(const ApplicationCarteProfil());
}

/// Racine de l'application. Elle est `Stateful` pour une seule raison :
/// mémoriser le thème choisi par l'utilisateur.
class ApplicationCarteProfil extends StatefulWidget {
  const ApplicationCarteProfil({super.key});

  @override
  State<ApplicationCarteProfil> createState() => _ApplicationCarteProfilState();
}

class _ApplicationCarteProfilState extends State<ApplicationCarteProfil> {
  bool _sombre = false;

  void _basculerTheme() {
    setState(() {
      _sombre = !_sombre;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Cartes de profil',
      debugShowCheckedModeBanner: false,
      theme: construireTheme(Brightness.light),
      darkTheme: construireTheme(Brightness.dark),
      themeMode: _sombre ? ThemeMode.dark : ThemeMode.light,
      home: EcranListeProfils(
        sombre: _sombre,
        onBasculerTheme: _basculerTheme,
      ),
    );
  }
}
```

### 55.18.3 — L'action dans l'`AppBar`

Dans `lib/ecrans/ecran_liste_profils.dart`, ajoutez les deux paramètres et l'icône :

```dart
  const EcranListeProfils({
    super.key,
    required this.sombre,
    required this.onBasculerTheme,
  });

  final bool sombre;
  final VoidCallback onBasculerTheme;
```

```dart
      appBar: AppBar(
        title: const Text('Cartes de profil'),
        actions: <Widget>[
          IconButton(
            onPressed: onBasculerTheme,
            tooltip: sombre ? 'Passer en clair' : 'Passer en sombre',
            icon: Icon(sombre ? Icons.light_mode : Icons.dark_mode),
          ),
        ],
      ),
```

### 55.18.4 — La revue du mode sombre

Basculez, puis relisez chaque zone. Voici ce qui casse habituellement, et pourquoi notre carte tient.

| Zone | Risque | Pourquoi cela fonctionne ici |
| --- | --- | --- |
| Fond de carte | reste blanc | `cardTheme.color` vient du `ColorScheme` |
| Textes | restent noirs | aucun `TextStyle` ne fixe de couleur, sauf sur la bannière |
| Bannière | contraste perdu | le dégradé part de la couleur du rang, indépendante du thème |
| Texte sur la bannière | illisible | il est en blanc **fixe**, sur un fond toujours coloré : correct |
| Badges | invisibles | `secondaryContainer` / `onSecondaryContainer` sont une paire garantie |
| Anneau de l'avatar | disparaît | il utilise `theme.cardTheme.color`, qui suit le thème |

> C'est ici que le travail fait à l'étape 55.6 est payé : **aucune** modification n'a été nécessaire dans les widgets pour que le mode sombre fonctionne.

**L'application compile et se lance.** Quatorzième étape terminée.

---

## 55.19 — Étape 15 : l'adaptation à la tablette

### 55.19.1 — Le problème

Sur un écran de 1000 pixels de large, une carte étirée sur toute la largeur est ridicule : les lignes de texte deviennent trop longues et l'œil se perd.

Deux stratégies du chapitre 51, que nous combinons :

```text
   largeur < 700  ->  une colonne, cartes pleine largeur
   largeur >= 700 ->  deux colonnes, en grille
```

Le point de rupture est déjà nommé : `Rupture.tablette`, défini en 55.6.

### 55.19.2 — `LayoutBuilder` plutôt que `MediaQuery`

`MediaQuery.sizeOf(context).width` donne la largeur de la **fenêtre**. `LayoutBuilder` donne la largeur réellement **disponible pour ce widget**.

C'est cette seconde valeur qui nous intéresse : si demain la liste est placée à côté d'un menu latéral, `LayoutBuilder` s'adaptera tout seul, `MediaQuery` non.

### 55.19.3 — Le nouveau corps de l'écran

Dans `lib/ecrans/ecran_liste_profils.dart`, remplacez le `body:` par :

```dart
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints contraintes) {
          final bool large = contraintes.maxWidth >= Rupture.tablette;

          if (!large) {
            return ListView.builder(
              padding: const EdgeInsets.all(Espace.l),
              itemCount: profilsDemo.length,
              itemBuilder: (BuildContext context, int index) {
                return Padding(
                  padding: const EdgeInsets.only(bottom: Espace.l),
                  child: _carte(context, profilsDemo[index]),
                );
              },
            );
          }

          return GridView.builder(
            padding: const EdgeInsets.all(Espace.l),
            itemCount: profilsDemo.length,
            gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
              // Chaque tuile fait au plus 520 px de large : le nombre de
              // colonnes est calculé par Flutter, sans autre point de rupture.
              maxCrossAxisExtent: 520,
              mainAxisSpacing: Espace.l,
              crossAxisSpacing: Espace.l,
              // Rapport largeur / hauteur d'une tuile. À ajuster à l'œil :
              // trop haut, la carte est coupée ; trop bas, il reste du vide.
              childAspectRatio: 0.95,
            ),
            itemBuilder: (BuildContext context, int index) {
              return _carte(context, profilsDemo[index]);
            },
          );
        },
      ),
```

et ajoutez la fabrique de carte, pour ne pas écrire deux fois les mêmes rappels :

```dart
  Widget _carte(BuildContext context, Profil profil) {
    return CarteProfil(
      profil: profil,
      onTap: () => _message(context, 'Profil de ${profil.nom}'),
      onDefier: () => _message(context, 'Défi envoyé à ${profil.nom}'),
      onMessage: () => _message(context, 'Message à ${profil.nom}'),
    );
  }
```

### 55.19.4 — Le piège de la grille

Une tuile de `GridView` a une hauteur **imposée** par `childAspectRatio`. Si votre carte est plus haute que sa tuile, vous obtenez un débordement de quelques pixels.

Trois corrections possibles, par ordre de préférence :

1. baisser `childAspectRatio` (tuiles plus hautes) ;
2. passer les cartes en mode `compacte: true` dans la grille ;
3. envelopper la carte dans un `SingleChildScrollView`, en dernier recours.

Testez en agrandissant la fenêtre sur bureau ou dans un navigateur : le passage d'une à deux colonnes doit être instantané, sans redémarrage.

**L'application compile et se lance.** Quinzième étape terminée.

---

## 55.20 — Étape 16 : l'écran de détail au clic

### 55.20.1 — Passer l'objet, pas l'identifiant

Le chapitre 50 posait la question : faut-il transmettre le `Profil` entier ou seulement son nom ?

Ici, la réponse est claire : **l'objet entier**, par le constructeur. Les données sont déjà en mémoire, il n'y a rien à recharger, et le typage nous protège. On ne passerait un identifiant que si l'écran de détail devait interroger un serveur.

### 55.20.2 — `lib/ecrans/ecran_detail_profil.dart`

```dart
import 'package:flutter/material.dart';

import '../modeles/profil.dart';
import '../theme/dimensions.dart';
import '../widgets/badges_profil.dart';
import '../widgets/carte_profil.dart';

/// Fiche détaillée d'un profil.
class EcranDetailProfil extends StatelessWidget {
  const EcranDetailProfil({super.key, required this.profil});

  final Profil profil;

  /// Les lignes de la fiche, construites à partir du modèle.
  ///
  /// Une `Map` littérale, puis `entries.map(...)` : la transformation de
  /// collection du chapitre 14 évite sept fois le même copier-coller.
  Map<String, String> get _details => <String, String>{
        'Rang': profil.rang.libelle,
        'Niveau': '${profil.niveau}',
        'Expérience': '${profil.experience} / ${profil.experienceRequise} XP',
        'Parties jouées': '${profil.partiesJouees}',
        'Taux de victoire': '${profil.pourcentageVictoires} %',
        'Quêtes terminées': '${profil.quetesTerminees}',
        'Statut': profil.enLigne ? 'En ligne' : 'Hors ligne',
      };

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Scaffold(
      // La flèche de retour est ajoutée automatiquement par l'AppBar
      // dès qu'il y a une route en dessous (chapitre 50).
      appBar: AppBar(title: Text(profil.nom)),
      body: ListView(
        padding: const EdgeInsets.all(Espace.l),
        children: <Widget>[
          // En mode compact : ni badges ni boutons, ils sont repris plus bas.
          CarteProfil(profil: profil, compacte: true),
          const SizedBox(height: Espace.xl),

          _Section(titre: 'Fiche détaillée', theme: theme),
          ..._details.entries.map((MapEntry<String, String> ligne) {
            return Padding(
              padding: const EdgeInsets.symmetric(vertical: 6),
              child: Row(
                children: <Widget>[
                  Text(
                    ligne.key,
                    style: theme.textTheme.bodyMedium?.copyWith(
                      color: theme.colorScheme.onSurfaceVariant,
                    ),
                  ),
                  const Spacer(),
                  Text(
                    ligne.value,
                    style: theme.textTheme.bodyMedium?.copyWith(
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                ],
              ),
            );
          }),

          if (profil.badges.isNotEmpty) ...<Widget>[
            const SizedBox(height: Espace.xl),
            _Section(titre: 'Badges', theme: theme),
            BadgesProfil(badges: profil.badges),
          ],
        ],
      ),
    );
  }
}

/// Titre de section, suivi d'un filet.
class _Section extends StatelessWidget {
  const _Section({required this.titre, required this.theme});

  final String titre;
  final ThemeData theme;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        Text(
          titre.toUpperCase(),
          style: theme.textTheme.labelMedium?.copyWith(
            color: theme.colorScheme.primary,
            letterSpacing: 1,
          ),
        ),
        const SizedBox(height: Espace.s),
        const Divider(),
        const SizedBox(height: Espace.s),
      ],
    );
  }
}
```

Notez `..._details.entries.map(...)` sans `.toList()` : l'opérateur d'étalement accepte n'importe quel `Iterable`.

### 55.20.3 — Ouvrir l'écran

Dans `lib/ecrans/ecran_liste_profils.dart`, remplacez le `onTap` de `_carte` :

```dart
      onTap: () => Navigator.push(
        context,
        MaterialPageRoute<void>(
          builder: (BuildContext context) => EcranDetailProfil(profil: profil),
        ),
      ),
```

avec l'import `import 'ecran_detail_profil.dart';`.

Le type `MaterialPageRoute<void>` indique que cet écran ne renverra **aucune** valeur au retour. Le préciser vous évitera un `dynamic` silencieux (chapitre 50).

### 55.20.4 — L'animation `Hero` (bonus B2)

Un `Hero` fait voler un widget d'un écran à l'autre pendant la transition. Il suffit d'entourer le même widget, dans les deux écrans, d'un `Hero` portant le **même `tag`**.

Comme l'avatar est déjà un widget unique, une seule modification suffit. Dans `lib/widgets/avatar_profil.dart`, enveloppez le `Container` du `build` :

```dart
    return Hero(
      // Le tag doit être unique dans un écran donné, et identique d'un
      // écran à l'autre. Le nom du joueur fait l'affaire ici.
      tag: 'avatar-${profil.nom}',
      child: Container(
        // ... le reste, inchangé ...
      ),
    );
```

Cliquez sur une carte : l'avatar glisse et grossit jusqu'à sa position dans l'écran de détail.

> **Erreur classique :** deux `Hero` avec le même `tag` visibles simultanément déclenchent l'exception `There are multiple heroes that share the same tag`. Si vous affichez deux fois le même profil dans une liste, ajoutez l'index au tag.

### 55.20.5 — État attendu

```text
╔═════════════════════════════════════════════════════╗
║  <-   Cyra Vale                                     ║
╠═════════════════════════════════════════════════════╣
║   ┌───────────────────────────────────────────────┐ ║
║   │  (carte compacte : bannière, avatar, identité,│ ║
║   │   statistiques, barre de niveau)              │ ║
║   └───────────────────────────────────────────────┘ ║
║                                                     ║
║   FICHE DÉTAILLÉE                                   ║
║   ─────────────────────────────────────────────     ║
║   Rang                                     Diamant  ║
║   Niveau                                        31  ║
║   Expérience                        1180 / 1400 XP  ║
║   Parties jouées                               243  ║
║   Taux de victoire                            86 %  ║
║   Quêtes terminées                              67  ║
║   Statut                                  En ligne  ║
║                                                     ║
║   BADGES                                            ║
║   ─────────────────────────────────────────────     ║
║   [Vitesse] [Discrétion] [Poison] [Vétéran]         ║
║   [Champion] [Explorateur]                          ║
╚═════════════════════════════════════════════════════╝
```

**Le projet est terminé.** Seizième et dernière étape.

---

## 55.21 — Le code complet, fichier par fichier

Le projet fini compte treize fichiers dans `lib/`. Chacun a été donné **en entier** dans une étape ; ce tableau vous dit laquelle, pour que vous puissiez vérifier votre travail sans relire tout le chapitre.

| Fichier | Rôle | Version finale donnée en |
| --- | --- | --- |
| `lib/main.dart` | racine, thème, bascule clair/sombre | 55.18.2 |
| `lib/modeles/profil.dart` | `Rang` et `Profil` | 55.7.3 |
| `lib/donnees/profils_demo.dart` | six profils en dur | 55.7.5 |
| `lib/theme/dimensions.dart` | `Espace`, `Rayon`, `Rupture` | 55.6.2 |
| `lib/theme/theme_application.dart` | thèmes clair et sombre, couleurs de rang, icônes de badges | 55.6.3 + 55.8.2 + 55.12.2 |
| `lib/widgets/avatar_profil.dart` | avatar en initiales, `Hero` | 55.15.3 + 55.20.4 |
| `lib/widgets/banniere_profil.dart` | dégradé et pastille de rang | 55.15.4 |
| `lib/widgets/identite_profil.dart` | nom, titre, guilde, statut | 55.15.5 |
| `lib/widgets/statistiques_profil.dart` | trois statistiques et séparateurs | 55.15.6 |
| `lib/widgets/badges_profil.dart` | badges en `Wrap` | 55.15.7 |
| `lib/widgets/barre_niveau.dart` | barre de progression | 55.15.8 |
| `lib/widgets/actions_profil.dart` | boutons « Défier » et « Message » | 55.15.9 |
| `lib/widgets/carte_profil.dart` | assemblage paramétrable | 55.15.10 + 55.16.2 |
| `lib/ecrans/ecran_liste_profils.dart` | liste ou grille, bascule de thème | 55.17.2 + 55.18.3 + 55.19.3 + 55.20.3 |
| `lib/ecrans/ecran_detail_profil.dart` | fiche détaillée | 55.20.2 |

### 55.21.1 — Le `pubspec.yaml`

Aucun paquet externe n'a été ajouté : tout vient du SDK Flutter. Votre `pubspec.yaml` est donc celui produit par `flutter create`, et la section `assets:` reste **absente** — c'était l'exigence O2.

```text
name: carte_profil
description: Carte de profil de joueur, réutilisable.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

> La version de `flutter_lints` évolue : ne recopiez pas ce numéro à l'aveugle, laissez celui que `flutter create` a écrit pour vous.

### 55.21.2 — Vérification finale du cahier des charges

Reprenez le tableau de 55.2.1 et cochez les treize exigences. Si l'une d'elles échoue, l'étape correspondante est indiquée dans la colonne « Vérification ».

Le test le plus révélateur reste le dernier : lancez l'application sur un appareil de 320 pixels de large (par exemple l'émulateur « small phone ») et faites défiler les six profils. Aucune bande jaune et noire ne doit apparaître.

---

## 55.22 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| Le dégradé dépasse des coins arrondis de la carte | la `Card` ne rogne pas ses enfants | `clipBehavior: Clip.antiAlias` sur la `Card` ou dans le `cardTheme` |
| `RenderFlex overflowed by N pixels` sur la ligne des badges | `Row` au lieu de `Wrap` | remplacer par `Wrap` et régler `spacing` **et** `runSpacing` |
| Le trait vertical entre deux statistiques est invisible ou immense | `VerticalDivider` sans hauteur imposée | `IntrinsicHeight` + `CrossAxisAlignment.stretch` |
| `A RenderFlex overflowed` sous le nom du joueur | la `Column` de textes n'est pas contrainte en largeur | l'entourer d'un `Expanded` et ajouter `maxLines` + `overflow` |
| L'avatar ne dépasse pas de la bannière | tentative de marge négative | superposer avec `Stack` + `Positioned` |
| `Incorrect use of ParentDataWidget` | `Positioned` utilisé hors d'un `Stack` | vérifier que le parent direct est bien un `Stack` |
| `The argument type 'CardTheme' can't be assigned to 'CardThemeData'` | confusion widget / données de thème | écrire `CardThemeData(...)` dans `ThemeData` |
| `Scaffold.of() called with a context that does not contain a Scaffold` | `ScaffoldMessenger.of(context)` appelé avec le `context` du `build` qui crée le `Scaffold` | interposer un `Builder`, ou appeler depuis un enfant |
| `There are multiple heroes that share the same tag` | deux `Hero` de même `tag` à l'écran | rendre le tag unique (ajouter l'index) |
| Le texte devient invisible en mode sombre | couleur écrite en dur dans un `TextStyle` | prendre la couleur dans `colorScheme` ou ne rien préciser |
| La barre de progression est toujours pleine | `value` supérieure à 1.0 | borner la valeur dans le modèle, comme le fait `progression` |
| La barre de progression tourne en boucle | `value: null` | fournir une valeur ; `null` signifie « progression inconnue » |
| Les cartes de la grille sont coupées | `childAspectRatio` trop grand | le baisser, ou passer les cartes en `compacte: true` |
| Les marges des cartes sont irrégulières | marge par défaut de `Card` qui s'ajoute à la vôtre | `margin: EdgeInsets.zero` dans le `cardTheme` |
| Les initiales se dessinent par-dessus la photo | `child` et `backgroundImage` fournis ensemble | mettre `child: null` quand il y a une image |

---

## 55.23 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Méthode de projet | on part de la maquette, on écrit le cahier des charges, puis on avance par étapes exécutables |
| Modèle Dart pur | un modèle n'importe jamais `material.dart` ; la couleur d'un rang est une décision d'affichage |
| Donnée calculée | tout ce qui se déduit d'autres champs devient un getter, jamais un champ stocké |
| Thème d'abord | définir couleurs, espacements et rayons **avant** les widgets évite un remaniement complet |
| `CircleAvatar` | un avatar ne nécessite aucun fichier image : initiales, couleur, et c'est tout |
| `Stack` + `Positioned` | la seule façon de faire déborder un widget : les marges négatives n'existent pas |
| `LinearGradient` | `begin`, `end`, `colors` ; jamais de `color` en plus dans le même `BoxDecoration` |
| `IntrinsicHeight` | donne aux séparateurs verticaux la hauteur du contenu, au prix d'une passe de mesure |
| `Wrap` | `spacing` horizontal, `runSpacing` vertical ; le seul widget qui passe à la ligne |
| `Expanded` | garantit qu'une `Row` ne débordera jamais, quel que soit le contenu |
| Classe plutôt que méthode | un widget nommé peut être `const`, apparaît dans l'inspecteur et se réutilise |
| Widget paramétrable | des champs `final`, des valeurs par défaut, et des `VoidCallback?` pour remonter les événements |
| Remontée d'état | un widget d'affichage prévient, il ne décide pas |
| `ListView.builder` | dès qu'une liste vient d'une collection, on construit à la demande |
| `LayoutBuilder` | mesure la place réellement disponible, contrairement à `MediaQuery` |
| Mode sombre | il est gratuit si, et seulement si, aucune couleur n'est écrite en dur |
| `Navigator.push` | on passe l'objet par le constructeur ; la flèche de retour est automatique |
| `Hero` | même `tag` dans les deux écrans, et l'animation est écrite |

---

## 55.24 — Extensions : dix défis

Ces défis sont classés du plus simple au plus exigeant. Chacun se traite sans notion nouvelle.

### Défi 1 — Trier les profils (facile)

Ajoutez trois boutons de tri dans l'`AppBar` : par niveau, par taux de victoire, par nom.

*Indication :* `List.of(profilsDemo)..sort((Profil a, Profil b) => b.niveau.compareTo(a.niveau))`. La liste d'origine est `const` : copiez-la avant de trier. Le tri doit vivre dans l'état de l'écran, donc `EcranListeProfils` devient `StatefulWidget`.

### Défi 2 — Filtrer par rang (facile)

Une ligne de `ChoiceChip` au-dessus de la liste : Tous, Bronze, Argent, Or, Diamant.

*Indication :* `Rang.values` fournit les quatre valeurs ; `profilsDemo.where((Profil p) => p.rang == _rangChoisi).toList()` fait le filtre (chapitre 14).

### Défi 3 — Un rang « Légende » (facile)

Ajoutez `legende('Légende', 5)` à l'`enum`.

*Indication :* le compilateur vous emmènera directement dans `couleurDuRang`, puisque le `switch` sur un `enum` est exhaustif. C'est le bénéfice recherché en 55.8.2. Aucun autre fichier ne devrait avoir besoin d'être modifié.

### Défi 4 — Rendre la carte cliquable partout sauf sur les boutons (facile)

Vérifiez qu'un clic sur « Défier » ne déclenche **pas** aussi `onTap`.

*Indication :* c'est déjà le cas, un bouton absorbe l'événement. Vérifiez-le, puis ajoutez un `SnackBar` distinct pour prouver que les deux ne se déclenchent jamais ensemble.

### Défi 5 — Barre de progression animée (moyenne)

À l'ouverture de la carte, la barre part de 0 et se remplit en 600 ms.

*Indication :* `TweenAnimationBuilder<double>` avec `tween: Tween<double>(begin: 0, end: profil.progression)` et `duration: const Duration(milliseconds: 600)`. Le `builder` reçoit la valeur intermédiaire et la passe à `LinearProgressIndicator`.

### Défi 6 — Une carte horizontale pour les grands écrans (moyenne)

Au-delà de 900 pixels, l'avatar passe à gauche et le contenu à droite.

*Indication :* un `LayoutBuilder` **dans** `CarteProfil`, et deux méthodes `_versionVerticale` / `_versionHorizontale` qui réutilisent les mêmes sept widgets. Aucun des sept fichiers de widget ne doit être modifié : c'est le test de leur bonne conception.

### Défi 7 — Statistiques configurables (moyenne)

Remplacez les trois statistiques figées par une `List<Statistique>` passée en paramètre, où `Statistique` est une petite classe `{ String libelle; String valeur; }`.

*Indication :* `StatistiquesProfil` ne reçoit alors plus un `Profil` mais une `List<Statistique>`, et devient réutilisable dans n'importe quelle application. Générez la liste depuis le `Profil` au niveau de `CarteProfil`.

### Défi 8 — Partager un profil (moyenne)

Un bouton qui copie le résumé textuel du profil dans le presse-papiers.

*Indication :* `Clipboard.setData(ClipboardData(text: resume))`, importé depuis `package:flutter/services.dart`. Ajoutez un getter `resume` au modèle : c'est du Dart pur, il y a sa place.

### Défi 9 — Mémoriser le thème choisi (difficile)

Le mode sombre doit survivre au redémarrage de l'application.

*Indication :* `shared_preferences` (chapitre 54). `flutter pub add shared_preferences`, lecture dans `initState` puis `setState`, écriture dans `_basculerTheme`. Attention à l'`async` dans `initState` : on appelle une méthode `Future<void>` sans l'attendre, et on protège le `setState` par `if (!mounted) return;` (chapitre 15).

### Défi 10 — Un éditeur de profil (difficile)

Un bouton « Modifier » dans l'écran de détail ouvre un formulaire, et le retour met la carte à jour.

*Indication :* trois notions se combinent ici. Le formulaire vient du chapitre 49 (`Form`, `TextFormField`, `validator`). Le retour de valeur vient du chapitre 50 : `Navigator.pop(context, profilModifie)`, récupéré par `final Profil? resultat = await Navigator.push<Profil>(...)`. La mise à jour utilise `copieAvec` du chapitre 09, puisque `Profil` est immuable. La liste devant changer, elle ne peut plus être `const` : c'est le moment de vous demander si `provider` (chapitre 52) ne serait pas plus confortable qu'un `setState` à la racine.

---

## Et maintenant ?

Vous venez de construire votre premier composant Flutter digne d'une vraie application : une carte soignée, paramétrable, réutilisable, qui survit au mode sombre, à la tablette et aux noms trop longs.

Regardez ce que vous n'avez **pas** eu à faire : aucune image à fournir, aucun paquet à installer, aucune couleur à ajuster à la main quand le thème a changé. C'est le résultat direct des trois décisions prises avant d'écrire du code : un thème d'abord, un modèle pur, des widgets nommés.

Ce projet était presque entièrement **statique** : les données ne changent jamais, et le seul `setState` de toute l'application sert à basculer le thème. C'est précisément ce qui va changer au chapitre suivant.

Le projet 2 est une calculatrice. Vous y affronterez une grille de dix-neuf boutons, un état qui évolue à chaque touche, la construction d'une expression caractère par caractère, une division par zéro à intercepter, et une contrainte que vous n'avez pas encore rencontrée : un écran qui doit tenir **entièrement**, sans défilement, sur n'importe quelle taille d'appareil.

Rendez-vous au chapitre 56 : [56-PARTIE-1C—PROJET-2-CALCULATRICE.md](./56-PARTIE-1C—PROJET-2-CALCULATRICE.md)
