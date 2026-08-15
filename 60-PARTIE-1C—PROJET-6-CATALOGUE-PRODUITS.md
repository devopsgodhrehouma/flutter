# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 60 — PROJET 6 : LE CATALOGUE DE PRODUITS ET LE PANIER

> **Niveau :** intermédiaire
> **Durée estimée :** 14 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), PARTIE 1B (chapitres 43 à 54), et les projets 55 à 59
> **Ce que vous saurez faire à la fin :** construire une boutique complète — catalogue chargé depuis un fichier JSON, grille responsive, fiche produit animée, panier centralisé avec `provider`, calcul exact du sous-total, de la TVA et du total, persistance du panier, formulaire de commande validé et tests automatisés.

---

## 60.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- modéliser un produit avec un identifiant, un nom, une description, un prix, une catégorie, une note et un stock ;
- comprendre pourquoi un prix ne se stocke **jamais** dans un `double` et l'écrire en centimes ;
- écrire `fromJson` et `toJson` sur ce modèle (rappel du chapitre 17) ;
- placer un catalogue JSON dans les assets et le déclarer dans `pubspec.yaml` (rappel du chapitre 47) ;
- lire un asset avec `rootBundle.loadString` et le transformer en objets Dart ;
- définir un dépôt (`repository`) de produits et en fournir deux implémentations ;
- afficher une collection en grille avec `GridView.builder` (rappel du chapitre 48) ;
- rendre cette grille **responsive** avec `SliverGridDelegateWithMaxCrossAxisExtent` (rappel du chapitre 51) ;
- construire une carte produit qui ne déborde jamais, quelle que soit la largeur ;
- fabriquer une image sans aucun fichier local : un dégradé et une icône par catégorie ;
- brancher, en variante, une `Image.network` avec `loadingBuilder` et `errorBuilder` ;
- formater un prix en euros avec le paquet `intl` ;
- ouvrir une fiche produit sur un nouvel écran et lui passer des données (rappel du chapitre 50) ;
- animer la transition entre la grille et la fiche avec `Hero` ;
- écrire un panier sous forme de `ChangeNotifier` exposé par `provider` (rappel du chapitre 52) ;
- ajouter un article, le retirer, changer sa quantité, plafonner au stock disponible ;
- calculer un sous-total, une TVA et un total **exacts**, en arithmétique entière (rappel du chapitre 14) ;
- afficher une icône de panier surmontée d'un `Badge` qui ne se reconstruit que quand le compte change ;
- soigner l'écran de panier vide ;
- chercher dans le catalogue et le filtrer par catégorie ;
- trier par prix ou par note ;
- persister le panier entre deux lancements (rappel du chapitre 54) ;
- valider un formulaire d'adresse avec `Form` et `TextFormField` (rappel du chapitre 49) ;
- afficher un écran de confirmation et nettoyer correctement la pile de navigation ;
- écrire des tests unitaires sur la logique du panier et sur le calcul des tarifs.

---

## 60.0.1 — Aperçu du résultat final

Ce projet est le plus « application réelle » de la partie 1C. Voici ce que vous obtiendrez.

L'écran de catalogue, sur un téléphone en portrait, affiche deux colonnes :

```text
┌────────────────────────────────────────────────┐
│  Pixel Boutique                     [panier(4)]│
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │ (loupe) Rechercher un produit...         │  │
│  └──────────────────────────────────────────┘  │
│  (Tout) (Périphériques) (Audio) (Écrans) ...   │
│  15 produits                Trier : Prix (+)   │
├────────────────────────────────────────────────┤
│  ┌─────────────────┐   ┌─────────────────┐     │
│  │/////////////////│   │/////////////////│     │
│  │///  (manette) //│   │/// (clavier) ///│     │
│  │/////////////////│   │/////////////////│     │
│  ├─────────────────┤   ├─────────────────┤     │
│  │ Manette Nébula  │   │ Clavier         │     │
│  │ Pro             │   │ mécanique Rune… │     │
│  │ ★★★★☆ 4,6       │   │ ★★★★★ 4,8       │     │
│  │ 64,90 €         │   │ 129,00 €        │     │
│  └─────────────────┘   └─────────────────┘     │
│  ┌─────────────────┐   ┌─────────────────┐     │
│  │//// RUPTURE ////│   │/////////////////│     │
│  │/// (souris) ////│   │/// (casque) ////│     │
│  │/////////////////│   │/////////////////│     │
│  ├─────────────────┤   ├─────────────────┤     │
│  │ Souris Photon   │   │ Casque Aurora   │     │
│  │ 8K              │   │ 7.1             │     │
│  │ ★★★★☆ 4,3       │   │ ★★★★☆ 4,5       │     │
│  │ 79,50 €         │   │ 89,90 €         │     │
│  └─────────────────┘   └─────────────────┘     │
└────────────────────────────────────────────────┘
```

Sur une tablette, la **même** grille passe à quatre colonnes sans une ligne de code supplémentaire :

```text
┌────────────────────────────────────────────────────────────────────────┐
│  Pixel Boutique                                            [panier(4)] │
├────────────────────────────────────────────────────────────────────────┤
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐                             │
│  │//////│   │//////│   │//////│   │//////│                             │
│  ├──────┤   ├──────┤   ├──────┤   ├──────┤                             │
│  │Manet.│   │Clavi.│   │Souris│   │Casque│                             │
│  │64,90€│   │129,0€│   │79,50€│   │89,90€│                             │
│  └──────┘   └──────┘   └──────┘   └──────┘                             │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐                             │
│  │//////│   │//////│   │//////│   │//////│                             │
│  ├──────┤   ├──────┤   ├──────┤   ├──────┤                             │
│  │Micro │   │Écran │   │Écran │   │Siège │                             │
│  │112,5€│   │299,0€│   │189,0€│   │349,0€│                             │
│  └──────┘   └──────┘   └──────┘   └──────┘                             │
└────────────────────────────────────────────────────────────────────────┘
```

La fiche produit, atteinte en touchant une carte. La vignette s'y déplace par une animation `Hero` :

```text
┌────────────────────────────────────────────────┐
│  ←   Manette Nébula Pro             [panier(4)]│
├────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │//////////////////////////////////////////│  │
│  │///////////////  (manette)  ///////////////│ │
│  │//////////////////////////////////////////│  │
│  └──────────────────────────────────────────┘  │
│  (Périphériques)                               │
│                                                │
│  Manette Nébula Pro                            │
│  ★★★★☆ 4,6                                     │
│                                                │
│  64,90 €                          En stock (12)│
│                                                │
│  Manette sans fil à faible latence, gâchettes  │
│  à retour de force et autonomie de quarante    │
│  heures.                                       │
│                                                │
│  Quantité      [ − ]    1    [ + ]             │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │        AJOUTER AU PANIER — 64,90 €       │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

L'écran du panier, avec le récapitulatif chiffré. **Tous les nombres de cette maquette sont exacts** ; vous les retrouverez dans les tests du 60.22 :

```text
┌────────────────────────────────────────────────┐
│  ←   Mon panier (4 articles)                   │
├────────────────────────────────────────────────┤
│ ┌────┐ Manette Nébula Pro                      │
│ │////│ 64,90 € l'unité                         │
│ │////│ [ − ]   1   [ + ]              64,90 €  │
│ └────┘                                    [X]  │
├────────────────────────────────────────────────┤
│ ┌────┐ Figurine du Boss Final                  │
│ │////│ 24,50 € l'unité                         │
│ │////│ [ − ]   2   [ + ]              49,00 €  │
│ └────┘                                    [X]  │
├────────────────────────────────────────────────┤
│ ┌────┐ Casque Aurora 7.1                       │
│ │////│ 89,90 € l'unité                         │
│ │////│ [ − ]   1   [ + ]              89,90 €  │
│ └────┘                                    [X]  │
├────────────────────────────────────────────────┤
│  Sous-total (HT)                     203,80 €  │
│  Livraison                 offerte     0,00 €  │
│  TVA (20 %)                           40,76 €  │
│  ────────────────────────────────────────────  │
│  Total TTC                           244,56 €  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │        COMMANDER — 244,56 €              │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

Le panier vide, que les débutants oublient presque toujours :

```text
┌────────────────────────────────────────────────┐
│  ←   Mon panier                                │
├────────────────────────────────────────────────┤
│                                                │
│                                                │
│                 (panier barré)                 │
│                                                │
│            Votre panier est vide               │
│                                                │
│    Parcourez le catalogue et ajoutez votre     │
│              premier article.                  │
│                                                │
│        ┌──────────────────────────────┐        │
│        │      VOIR LE CATALOGUE       │        │
│        └──────────────────────────────┘        │
│                                                │
└────────────────────────────────────────────────┘
```

Le formulaire de commande, validé champ par champ :

```text
┌────────────────────────────────────────────────┐
│  ←   Commande                                  │
├────────────────────────────────────────────────┤
│  3 articles · 244,56 € TTC                     │
│                                                │
│  Nom complet *                                 │
│  ┌──────────────────────────────────────────┐  │
│  │ Camille Durand                           │  │
│  └──────────────────────────────────────────┘  │
│  Adresse *                                     │
│  ┌──────────────────────────────────────────┐  │
│  │ 12 rue des Sprites                       │  │
│  └──────────────────────────────────────────┘  │
│  Code postal *          Ville *                │
│  ┌───────────┐  ┌───────────────────────────┐  │
│  │ 3500      │  │ Rennes                    │  │
│  └───────────┘  └───────────────────────────┘  │
│  Le code postal doit comporter 5 chiffres.     │
│                                                │
│  Courriel *                                    │
│  ┌──────────────────────────────────────────┐  │
│  │ camille.durand@exemple.fr                │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │           VALIDER LA COMMANDE            │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

Et la confirmation :

```text
┌────────────────────────────────────────────────┐
│  Commande confirmée                            │
├────────────────────────────────────────────────┤
│                                                │
│                   (coche)                      │
│                                                │
│              Merci Camille !                   │
│                                                │
│         Commande PB-20260815-4821              │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ 3 références · 4 articles                │  │
│  │ Total réglé                    244,56 €  │  │
│  │ Livraison estimée   mardi 18 août 2026   │  │
│  │                                          │  │
│  │ 12 rue des Sprites                       │  │
│  │ 35000 Rennes                             │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │         RETOUR À LA BOUTIQUE             │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

---

## 60.0.2 — Cahier des charges

### Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | Un produit possède un identifiant, un nom, une description, un prix, une catégorie, une note sur 5 et un stock. | Le modèle compile et se sérialise. |
| O2 | Le catalogue est décrit dans un fichier JSON placé dans les assets. | Ajouter un produit au JSON le fait apparaître sans recompiler la logique. |
| O3 | Le catalogue s'affiche en grille. | 15 produits, aucune barre de débordement jaune. |
| O4 | Le nombre de colonnes s'adapte à la largeur. | Téléphone 2, tablette 4, bureau 6. |
| O5 | Aucun fichier image n'est fourni ; chaque produit reçoit une vignette générée. | Le projet ne contient aucun `.png`. |
| O6 | Les prix sont affichés au format français. | `6490` centimes s'affiche `64,90 €`. |
| O7 | Toucher une carte ouvre la fiche produit sur un nouvel écran. | Le bouton retour ramène à la grille. |
| O8 | La vignette est animée entre la grille et la fiche. | La transition `Hero` dure environ 300 ms. |
| O9 | On peut ajouter un produit au panier depuis sa fiche. | Le badge du panier s'incrémente. |
| O10 | On peut changer la quantité d'une ligne et supprimer une ligne. | La quantité est plafonnée au stock. |
| O11 | Le panier calcule un sous-total, des frais de port, la TVA et un total. | Les valeurs sont exactes au centime. |
| O12 | La livraison est offerte à partir de 100,00 € hors taxes. | 99,95 € HT → port facturé ; 100,00 € HT → port offert. |
| O13 | Un badge sur l'icône de panier indique le nombre d'articles. | 1 + 2 + 1 → badge « 4 ». |
| O14 | Le panier vide affiche un message et un bouton de retour au catalogue. | Vider le panier. |
| O15 | On peut chercher un produit et filtrer par catégorie. | La recherche est insensible à la casse. |
| O16 | On peut trier par prix croissant, prix décroissant ou meilleure note. | L'ordre change à l'écran. |
| O17 | Le panier survit à la fermeture de l'application. | Tuer l'application, la relancer. |
| O18 | La commande passe par un formulaire d'adresse validé. | Un code postal à 4 chiffres est refusé. |
| O19 | Une confirmation récapitule la commande et vide le panier. | Le badge revient à zéro. |
| O20 | La logique du panier et le calcul des tarifs sont couverts par des tests. | `flutter test` passe. |

### Fonctionnalités bonus

| # | Exigence |
| --- | --- |
| B1 | Une variante de vignette utilisant `Image.network` avec `errorBuilder`. |
| B2 | Un filtre « en stock seulement ». |
| B3 | Un mode sombre suivant le réglage du système. |
| B4 | Un `SnackBar` « ajouté au panier » avec une action VOIR. |

---

## 60.0.3 — Notions mobilisées

Ce projet n'introduit aucune notion nouvelle. Il assemble ce que vous savez déjà. Si une ligne du tableau vous surprend, relisez le chapitre indiqué **avant** de commencer.

| Notion | Chapitre | Usage exact dans ce projet |
| --- | --- | --- |
| `int`, `double`, division entière | 02, 03 | Les prix en centimes, le calcul de la TVA. |
| `List`, `Map` | 06 | Le catalogue, l'index du panier par identifiant. |
| Classes, champs `final` | 08 | `Produit`, `LignePanier`, `Tarifs`, `Adresse`. |
| Constructeurs nommés, `required` | 09 | `Produit.fromJson`, `copyWith`. |
| `enum` enrichi | 11 | `Categorie`, `TriProduits`. |
| Null safety, `??`, `?.` | 12 | La lecture défensive du JSON. |
| Exceptions, `try`/`catch` | 13 | Le chargement du catalogue. |
| `map`, `where`, `fold`, `sort` | 14 | Le filtrage, le tri, la somme du panier. |
| `Future`, `async`, `await` | 15 | Le dépôt, la persistance. |
| `pubspec.yaml`, paquets, assets | 16, 47 | `provider`, `intl`, `shared_preferences`, `catalogue.json`. |
| `jsonEncode`, `jsonDecode` | 17 | La sérialisation des produits et du panier. |
| `MaterialApp`, `Scaffold`, `AppBar` | 44 | La structure des écrans. |
| `StatefulWidget`, `setState`, `dispose` | 45 | La quantité de la fiche, le champ de recherche. |
| `Row`, `Column`, `Expanded`, `Stack` | 46 | Les cartes, le bandeau « rupture ». |
| `Text`, `Icon`, images réseau | 47 | Les vignettes, les étoiles. |
| `GridView.builder`, `Card`, `ListView` | 48 | La grille et l'écran de panier. |
| `Form`, `TextFormField`, `validator` | 49 | L'adresse de livraison. |
| `Navigator.push`, `pushAndRemoveUntil` | 50 | Fiche, panier, commande, confirmation. |
| `MediaQuery`, grille adaptative | 51 | Le nombre de colonnes. |
| `ChangeNotifier`, `provider`, `select` | 52 | Le panier et le catalogue. |
| `shared_preferences` | 54 | La persistance du panier. |

---

## 60.0.4 — Arborescence du projet

Voici l'arborescence finale. Elle est donnée dès maintenant pour que vous sachiez où va chaque fichier ; nous la construirons pas à pas.

```text
pixel_boutique/
├── pubspec.yaml
├── assets/
│   └── donnees/
│       └── catalogue.json                le catalogue, 15 produits
├── lib/
│   ├── main.dart                         point d'entrée, thème, providers
│   ├── modeles/
│   │   ├── categorie.dart                enum Categorie
│   │   ├── produit.dart                  Produit + JSON
│   │   ├── ligne_panier.dart             produit + quantité
│   │   ├── tarifs.dart                   sous-total, port, TVA, total
│   │   └── commande.dart                 adresse + référence
│   ├── logique/
│   │   ├── criteres.dart                 enum TriProduits
│   │   ├── recherche.dart                filtrer, trier (fonctions pures)
│   │   └── calcul_tarifs.dart            le calcul, sans Flutter
│   ├── donnees/
│   │   ├── depot_produits.dart           interface
│   │   ├── depot_assets.dart             lecture du JSON des assets
│   │   ├── depot_memoire.dart            jeu de données pour les tests
│   │   ├── depot_panier.dart             interface de persistance
│   │   └── depot_panier_prefs.dart       shared_preferences
│   ├── etat/
│   │   ├── panier.dart                   ChangeNotifier du panier
│   │   └── etat_catalogue.dart           ChangeNotifier du catalogue
│   ├── ecrans/
│   │   ├── ecran_catalogue.dart          la grille
│   │   ├── ecran_produit.dart            la fiche
│   │   ├── ecran_panier.dart             le panier
│   │   ├── ecran_commande.dart           le formulaire d'adresse
│   │   └── ecran_confirmation.dart       le récapitulatif final
│   ├── widgets/
│   │   ├── vignette_produit.dart         dégradé + icône, cible du Hero
│   │   ├── carte_produit.dart            une cellule de la grille
│   │   ├── etoiles_note.dart             la note sur 5
│   │   ├── bouton_panier.dart            icône + Badge
│   │   ├── selecteur_quantite.dart       − 1 +
│   │   ├── barre_filtres.dart            recherche + catégories + tri
│   │   ├── tuile_ligne_panier.dart       une ligne du panier
│   │   ├── recapitulatif_tarifs.dart     le bloc chiffré
│   │   └── panier_vide.dart              l'état vide
│   └── utilitaires/
│       ├── formats.dart                  intl : prix, dates
│       └── apparence.dart                icône et couleurs par catégorie
└── test/
    ├── produit_test.dart                 aller-retour JSON
    ├── calcul_tarifs_test.dart           les totaux
    ├── panier_test.dart                  la logique du panier
    └── recherche_test.dart               filtrage et tri
```

**Pourquoi cette découpe ?** Chaque dossier répond à une seule question :

```text
modeles/      QUOI ?        les données pures, sans Flutter
logique/      COMMENT ?     les règles métier, testables sans écran
donnees/      OÙ ?          la lecture et l'écriture
etat/         QUAND ?       ce qui change et qui prévient l'interface
ecrans/       À QUOI ÇA     les pages complètes
widgets/      RESSEMBLE ?   les morceaux réutilisables
utilitaires/  AVEC QUOI ?   le formatage et l'apparence
```

Aucun fichier de `modeles/` ni de `logique/` n'importe `package:flutter/material.dart`. C'est cette règle qui rend les tests du 60.22 possibles sans lancer d'émulateur. Un seul fichier de `donnees/` fait exception, `depot_assets.dart`, parce que `rootBundle` vient de Flutter ; nous le signalerons explicitement.

---

## 60.1 — Créer le projet et poser le squelette

### Créer le projet

```text
flutter create pixel_boutique
cd pixel_boutique
```

### Ajouter les dépendances

Ajoutez les paquets un par un. `flutter pub add` écrit toujours la contrainte de version la plus récente compatible ; c'est préférable à recopier un numéro qui vieillira.

```text
flutter pub add provider
flutter pub add intl
flutter pub add shared_preferences
flutter pub add flutter_localizations --sdk=flutter
```

Créez ensuite le dossier des assets :

```text
mkdir -p assets/donnees
```

Le `pubspec.yaml` obtenu ressemble à ceci. Les numéros sont ceux constatés à la rédaction ; les vôtres peuvent être supérieurs, c'est normal.

**`pixel_boutique/pubspec.yaml`**

```yaml
name: pixel_boutique
description: "Un catalogue de produits avec panier, tarifs et commande."
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

  # Formatage des prix et des dates (chapitre 58.10)
  intl: ^0.20.3

  # Persistance du panier (chapitre 54)
  shared_preferences: ^2.5.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  # Le catalogue est une donnée, pas du code : il vit dans les assets.
  assets:
    - assets/donnees/catalogue.json
```

> **Attention.** L'indentation d'un `pubspec.yaml` est significative : deux espaces, jamais de tabulation. `assets:` doit être aligné sous `uses-material-design:`, tous deux à l'intérieur de la section `flutter:`. Une erreur d'indentation ici produit le message `Error on line NN, column N: Expected a key while parsing a block mapping`.

### Le squelette

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationBoutique());
}

/// Racine de l'application.
///
/// Pour l'instant elle n'installe qu'un thème Material 3 et un écran
/// vide. Tout le reste viendra s'y greffer au fil du chapitre.
class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Pixel Boutique',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4F46E5)),
      ),
      home: const EcranCatalogue(),
    );
  }
}

/// Écran principal, provisoirement vide.
class EcranCatalogue extends StatelessWidget {
  const EcranCatalogue({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Pixel Boutique')),
      body: const Center(child: Text('Catalogue à venir')),
    );
  }
}
```

**État exécutable.** `flutter run` affiche une barre d'application violette et un texte centré. Le projet compile et les dépendances sont résolues.

```text
┌────────────────────────────────────────────────┐
│  Pixel Boutique                                │
├────────────────────────────────────────────────┤
│                                                │
│              Catalogue à venir                 │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 60.2 — Les catégories

Un produit appartient à une catégorie. Un débutant écrirait `String categorie = 'audio'`. C'est une erreur : rien n'empêche alors d'écrire `'Audio'`, `'AUDIO'` ou `'audoi'`. Le compilateur ne dira rien et le filtre ne trouvera jamais rien.

Le chapitre 11 vous a donné l'outil exact : l'`enum` enrichi, qui porte ses propres champs.

**`lib/modeles/categorie.dart`**

```dart
/// Rayon de la boutique auquel appartient un produit.
///
/// C'est un `enum` enrichi (chapitre 11) : chaque valeur transporte
/// son libellé affichable. Le nom technique, celui qui part dans le
/// JSON, est fourni gratuitement par Dart via `.name`.
enum Categorie {
  peripheriques('Périphériques'),
  audio('Audio'),
  ecrans('Écrans'),
  sieges('Sièges'),
  goodies('Goodies'),
  livres('Livres');

  const Categorie(this.libelle);

  /// Texte affiché à l'utilisateur, avec accents et majuscule.
  final String libelle;

  /// Reconstruit une catégorie à partir du nom lu dans le JSON.
  ///
  /// Si le nom est inconnu (catalogue mis à jour par une personne
  /// qui a fait une faute de frappe, ancienne version du fichier),
  /// on ne lève pas d'exception : on retombe sur `goodies`. Une
  /// donnée abîmée ne doit jamais empêcher le démarrage.
  static Categorie depuisNom(String? nom) {
    for (final Categorie c in Categorie.values) {
      if (c.name == nom) {
        return c;
      }
    }
    return Categorie.goodies;
  }
}
```

### Vérification en console

Ce fichier n'importe pas Flutter. Vous pouvez donc l'essayer dans DartPad.

```dart
// Collez le contenu de categorie.dart au-dessus de ce main.
void main() {
  for (final Categorie c in Categorie.values) {
    print('${c.name} -> ${c.libelle}');
  }
  print(Categorie.depuisNom('audio'));
  print(Categorie.depuisNom('AUDIO'));
}
```

**Résultat :**

```text
peripheriques -> Périphériques
audio -> Audio
ecrans -> Écrans
sieges -> Sièges
goodies -> Goodies
livres -> Livres
Categorie.audio
Categorie.goodies
```

> **Remarque.** Les noms techniques sont écrits **sans accent et sans espace** : `peripheriques`, `ecrans`, `sieges`. C'est volontaire. Un nom d'identifiant Dart ne peut de toute façon pas contenir d'espace, et un JSON sans accent traverse sans dommage tous les outils que vous rencontrerez. Les accents vivent dans `libelle`, c'est-à-dire dans l'interface, pas dans les données.

**État exécutable.** Le fichier compile seul. L'application ne change pas encore.

---

## 60.3 — Le modèle `Produit`

### Une décision importante : le prix est un entier

Voici l'erreur numéro un des applications de commerce écrites par des débutants :

```dart
double prix = 64.90;
```

Un `double` est un nombre à virgule **binaire**. Il ne peut pas représenter exactement 64,90, pas plus qu'un nombre décimal ne peut représenter exactement 1/3. Essayez ceci dans DartPad :

```dart
void main() {
  print(0.1 + 0.2);
  print(64.90 * 3);
  print(19.99 + 0.01);
}
```

**Résultat :**

```text
0.30000000000000004
194.70000000000002
20.0
```

Le deuxième affichage suffit à condamner l'approche : trois manettes à 64,90 € ne coûtent pas `194.70000000000002` €. Sur un panier de vingt lignes, ces poussières s'accumulent et le total affiché finit par différer d'un centime de celui qu'attend le client. En commerce, un centime d'écart est un incident.

La solution universelle est de **compter en centimes, avec des `int`**. Un `int` Dart est exact ; l'addition, la multiplication et la comparaison le sont aussi. On ne divise par 100 qu'au tout dernier moment, pour l'affichage.

```text
  Interne (int, exact)          Affiché (String)
  ────────────────────          ────────────────
        6490            ───►        64,90 €
       12900            ───►       129,00 €
       20380            ───►       203,80 €
```

Retenez la règle : **on calcule en centimes, on formate en euros**.

### La classe

**`lib/modeles/produit.dart`**

```dart
import 'categorie.dart';

/// Un article du catalogue.
///
/// Tous les champs sont `final` : un produit est immuable. Pour en
/// obtenir une variante, on passe par [copyWith], qui fabrique un
/// nouvel objet (chapitre 09).
class Produit {
  const Produit({
    required this.id,
    required this.nom,
    required this.description,
    required this.prixCentimes,
    required this.categorie,
    required this.note,
    required this.stock,
  });

  /// Identifiant stable, unique dans le catalogue.
  ///
  /// C'est lui qui sert de clé dans le panier et de `tag` pour les
  /// animations `Hero`. Il ne doit jamais changer.
  final String id;

  /// Nom affiché sur la carte et sur la fiche.
  final String nom;

  /// Texte long affiché uniquement sur la fiche.
  final String description;

  /// Prix unitaire, **en centimes**. 6490 signifie 64,90 €.
  final int prixCentimes;

  /// Rayon de la boutique.
  final Categorie categorie;

  /// Note moyenne, de 0.0 à 5.0. Ici un `double` est légitime :
  /// on ne fait aucune arithmétique monétaire avec.
  final double note;

  /// Nombre d'exemplaires disponibles. 0 signifie « rupture ».
  final int stock;

  /// Vrai si l'on peut encore commander l'article.
  bool get enStock => stock > 0;

  /// Vrai s'il ne reste que quelques exemplaires : on l'indique à
  /// l'utilisateur pour l'aider à décider.
  bool get stockFaible => stock > 0 && stock <= 3;

  /// Fabrique une copie en changeant seulement les champs fournis.
  Produit copyWith({
    String? id,
    String? nom,
    String? description,
    int? prixCentimes,
    Categorie? categorie,
    double? note,
    int? stock,
  }) {
    return Produit(
      id: id ?? this.id,
      nom: nom ?? this.nom,
      description: description ?? this.description,
      prixCentimes: prixCentimes ?? this.prixCentimes,
      categorie: categorie ?? this.categorie,
      note: note ?? this.note,
      stock: stock ?? this.stock,
    );
  }

  /// Deux produits sont égaux s'ils ont le même identifiant.
  ///
  /// C'est ce qui permet d'écrire `panier.contains(produit)` ou
  /// d'utiliser un produit comme clé de `Map` sans surprise. Dès que
  /// l'on redéfinit `==`, il faut redéfinir `hashCode` : la règle est
  /// absolue en Dart comme dans la plupart des langages.
  @override
  bool operator ==(Object autre) {
    if (identical(this, autre)) {
      return true;
    }
    return autre is Produit && autre.id == id;
  }

  @override
  int get hashCode => id.hashCode;

  @override
  String toString() => 'Produit($id, $nom, $prixCentimes c)';
}
```

### Vérification en console

```dart
void main() {
  const Produit manette = Produit(
    id: 'p01',
    nom: 'Manette Nébula Pro',
    description: 'Manette sans fil à faible latence.',
    prixCentimes: 6490,
    categorie: Categorie.peripheriques,
    note: 4.6,
    stock: 12,
  );

  print(manette);
  print('En stock : ${manette.enStock}');
  print('Stock faible : ${manette.stockFaible}');

  final Produit epuisee = manette.copyWith(stock: 0);
  print('Après copyWith(stock: 0) : ${epuisee.enStock}');
  print('Même identifiant, donc égaux : ${manette == epuisee}');
  print('Prix de 3 manettes en centimes : ${manette.prixCentimes * 3}');
}
```

**Résultat :**

```text
Produit(p01, Manette Nébula Pro, 6490 c)
En stock : true
Stock faible : false
Après copyWith(stock: 0) : false
Même identifiant, donc égaux : true
Prix de 3 manettes en centimes : 19470
```

`19470` centimes, soit 194,70 € exactement. Comparez avec le `194.70000000000002` du début de section : c'est toute la différence.

**État exécutable.** Le fichier compile seul.

---

## 60.4 — La sérialisation JSON (rappel du chapitre 17)

Le catalogue vivra dans un fichier JSON. Il faut donc savoir passer d'une `Map<String, dynamic>` à un `Produit`, et inversement.

### La règle du `fromJson` défensif

Un `fromJson` écrit naïvement plante dès qu'une clé manque :

```dart
// À NE PAS FAIRE
nom: json['nom'] as String,   // type 'Null' is not a subtype of type 'String'
```

Le fichier peut avoir été édité à la main, tronqué, ou provenir d'une version antérieure. La règle est simple : **lire en type nullable, puis fournir une valeur de repli**.

Ajoutez ces deux méthodes à la classe `Produit`, juste avant `copyWith`.

**`lib/modeles/produit.dart`** (extrait à insérer dans la classe)

```dart
  /// Reconstruit un produit depuis une entrée JSON décodée.
  ///
  /// Chaque lecture est défensive (chapitre 12) :
  ///  - `as String?` accepte l'absence de la clé ;
  ///  - `??` fournit une valeur de repli ;
  ///  - `as num?` accepte aussi bien `4` que `4.6`, car un JSON
  ///    contenant `4` produit un `int` et non un `double`.
  factory Produit.fromJson(Map<String, dynamic> json) {
    return Produit(
      id: json['id'] as String? ?? '',
      nom: json['nom'] as String? ?? 'Produit sans nom',
      description: json['description'] as String? ?? '',
      prixCentimes: (json['prix_centimes'] as num?)?.toInt() ?? 0,
      categorie: Categorie.depuisNom(json['categorie'] as String?),
      note: (json['note'] as num?)?.toDouble() ?? 0.0,
      stock: (json['stock'] as num?)?.toInt() ?? 0,
    );
  }

  /// Écrit le produit sous une forme directement encodable en JSON.
  ///
  /// La catégorie est écrite avec `.name`, jamais avec `.index` :
  /// le jour où l'on insère une catégorie au milieu de l'`enum`,
  /// tous les fichiers déjà enregistrés désigneraient la mauvaise.
  Map<String, dynamic> toJson() {
    return <String, dynamic>{
      'id': id,
      'nom': nom,
      'description': description,
      'prix_centimes': prixCentimes,
      'categorie': categorie.name,
      'note': note,
      'stock': stock,
    };
  }
```

> **Le piège de `as num?`.** Dans un JSON, `"note": 4` est décodé par Dart en `int`, alors que `"note": 4.0` est décodé en `double`. Écrire `json['note'] as double?` fonctionne pour le second et **plante** pour le premier. `as num?` couvre les deux, puis `.toDouble()` normalise. Le même raisonnement s'applique à `prix_centimes` : `as num?` puis `.toInt()`.

### Le test d'aller-retour

Le seul test qui prouve qu'une sérialisation est correcte est l'aller-retour : sérialiser puis désérialiser doit redonner un objet identique.

**`test/produit_test.dart`**

```dart
import 'dart:convert';

import 'package:flutter_test/flutter_test.dart';
import 'package:pixel_boutique/modeles/categorie.dart';
import 'package:pixel_boutique/modeles/produit.dart';

void main() {
  const Produit reference = Produit(
    id: 'p01',
    nom: 'Manette Nébula Pro',
    description: 'Manette sans fil à faible latence.',
    prixCentimes: 6490,
    categorie: Categorie.peripheriques,
    note: 4.6,
    stock: 12,
  );

  group('Produit', () {
    test('aller-retour JSON conservant toutes les valeurs', () {
      final String texte = jsonEncode(reference.toJson());
      final Map<String, dynamic> decode =
          jsonDecode(texte) as Map<String, dynamic>;
      final Produit relu = Produit.fromJson(decode);

      expect(relu.id, 'p01');
      expect(relu.nom, 'Manette Nébula Pro');
      expect(relu.prixCentimes, 6490);
      expect(relu.categorie, Categorie.peripheriques);
      expect(relu.note, 4.6);
      expect(relu.stock, 12);
      expect(relu, reference); // égalité par identifiant
    });

    test('un JSON incomplet ne fait pas planter la lecture', () {
      final Produit produit = Produit.fromJson(<String, dynamic>{'id': 'p99'});

      expect(produit.id, 'p99');
      expect(produit.nom, 'Produit sans nom');
      expect(produit.prixCentimes, 0);
      expect(produit.categorie, Categorie.goodies);
      expect(produit.note, 0.0);
      expect(produit.stock, 0);
      expect(produit.enStock, isFalse);
    });

    test('une note entière est acceptée', () {
      // "note": 4 arrive en int, pas en double.
      final Produit produit = Produit.fromJson(<String, dynamic>{
        'id': 'p98',
        'note': 4,
        'prix_centimes': 1000,
      });

      expect(produit.note, 4.0);
      expect(produit.prixCentimes, 1000);
    });

    test('une catégorie inconnue retombe sur goodies', () {
      final Produit produit = Produit.fromJson(<String, dynamic>{
        'id': 'p97',
        'categorie': 'chaussettes',
      });

      expect(produit.categorie, Categorie.goodies);
    });
  });
}
```

Lancez :

```text
flutter test test/produit_test.dart
```

**Résultat :**

```text
00:02 +4: All tests passed!
```

> **Remarque.** Les imports du dossier `test/` s'écrivent en `package:pixel_boutique/...`, jamais en `../lib/...`. Un import relatif depuis `test/` crée un **second exemplaire** de chaque classe aux yeux du compilateur, et vous obtenez des messages absurdes du genre `type 'Produit' is not a subtype of type 'Produit'`.

**État exécutable.** L'application est inchangée, mais quatre tests passent.

---

## 60.5 — Le catalogue en JSON dans les assets

### Pourquoi un asset et pas du code Dart

On pourrait écrire le catalogue directement en Dart, sous forme d'une `List<Produit>` constante. Trois raisons de ne pas le faire :

1. **Séparation.** Une donnée n'est pas du code. La personne qui met à jour les prix n'a pas à ouvrir un fichier `.dart`.
2. **Réalisme.** Dans une vraie boutique, le catalogue arrive d'une API REST au format JSON (chapitre 53). En le lisant depuis un asset, vous écrivez exactement le même code de désérialisation ; le jour où vous branchez le réseau, seul le dépôt change.
3. **Volume.** Un catalogue de 500 produits en Dart allonge le temps de compilation ; en JSON il ne coûte rien.

### Le fichier

**`assets/donnees/catalogue.json`**

```json
{
  "version": 1,
  "devise": "EUR",
  "produits": [
    {
      "id": "p01",
      "nom": "Manette Nébula Pro",
      "description": "Manette sans fil à faible latence, gâchettes à retour de force et autonomie de quarante heures.",
      "prix_centimes": 6490,
      "categorie": "peripheriques",
      "note": 4.6,
      "stock": 12
    },
    {
      "id": "p02",
      "nom": "Clavier mécanique Runeboard",
      "description": "Clavier mécanique compact, interrupteurs tactiles silencieux et rétroéclairage réglable touche par touche.",
      "prix_centimes": 12900,
      "categorie": "peripheriques",
      "note": 4.8,
      "stock": 5
    },
    {
      "id": "p03",
      "nom": "Souris Photon 8K",
      "description": "Souris de 58 grammes, capteur optique de 26 000 points par pouce et taux de rafraîchissement de 8 000 Hz.",
      "prix_centimes": 7950,
      "categorie": "peripheriques",
      "note": 4.3,
      "stock": 0
    },
    {
      "id": "p04",
      "nom": "Casque Aurora 7.1",
      "description": "Casque fermé à son positionnel virtuel, micro amovible et coussinets en similicuir remplaçables.",
      "prix_centimes": 8990,
      "categorie": "audio",
      "note": 4.5,
      "stock": 8
    },
    {
      "id": "p05",
      "nom": "Micro Studio Écho",
      "description": "Micro à condensateur cardioïde, pied de bureau et filtre anti-pop fournis.",
      "prix_centimes": 11250,
      "categorie": "audio",
      "note": 4.1,
      "stock": 3
    },
    {
      "id": "p06",
      "nom": "Écran 27 pouces Pixel 165 Hz",
      "description": "Dalle IPS de 2560 sur 1440 points à 165 Hz, temps de réponse d'une milliseconde.",
      "prix_centimes": 29900,
      "categorie": "ecrans",
      "note": 4.7,
      "stock": 4
    },
    {
      "id": "p07",
      "nom": "Écran portable 15 pouces Nomade",
      "description": "Écran USB-C de 15,6 pouces en 1920 sur 1080, housse-support intégrée.",
      "prix_centimes": 18900,
      "categorie": "ecrans",
      "note": 3.9,
      "stock": 6
    },
    {
      "id": "p08",
      "nom": "Siège Ergo Boss",
      "description": "Siège ergonomique à soutien lombaire réglable et accoudoirs sur quatre axes.",
      "prix_centimes": 34900,
      "categorie": "sieges",
      "note": 4.4,
      "stock": 2
    },
    {
      "id": "p09",
      "nom": "Repose-poignets Velours",
      "description": "Repose-poignets en mousse à mémoire de forme, housse lavable en machine.",
      "prix_centimes": 1990,
      "categorie": "peripheriques",
      "note": 4.0,
      "stock": 30
    },
    {
      "id": "p10",
      "nom": "Figurine du Boss Final",
      "description": "Figurine de collection de 18 centimètres, résine peinte à la main, socle inclus.",
      "prix_centimes": 2450,
      "categorie": "goodies",
      "note": 4.9,
      "stock": 15
    },
    {
      "id": "p11",
      "nom": "Mug Hello World",
      "description": "Mug en céramique de 350 millilitres, passe au lave-vaisselle et au micro-ondes.",
      "prix_centimes": 1290,
      "categorie": "goodies",
      "note": 4.2,
      "stock": 40
    },
    {
      "id": "p12",
      "nom": "Tapis de souris XXL Galaxie",
      "description": "Grand tapis de 90 sur 40 centimètres, surface tissée et base antidérapante.",
      "prix_centimes": 2290,
      "categorie": "peripheriques",
      "note": 4.5,
      "stock": 22
    },
    {
      "id": "p13",
      "nom": "Coussin lombaire Checkpoint",
      "description": "Coussin de maintien lombaire à sangle réglable, mousse haute densité.",
      "prix_centimes": 3450,
      "categorie": "sieges",
      "note": 4.2,
      "stock": 18
    },
    {
      "id": "p14",
      "nom": "Livre Boucle de jeu",
      "description": "Trois cents pages sur la boucle de jeu, le delta time et l'architecture d'un moteur 2D.",
      "prix_centimes": 3900,
      "categorie": "livres",
      "note": 4.6,
      "stock": 9
    },
    {
      "id": "p15",
      "nom": "Livre Dart et Flutter en pratique",
      "description": "Manuel de référence sur Dart, les widgets, la gestion d'état et les tests.",
      "prix_centimes": 4500,
      "categorie": "livres",
      "note": 4.8,
      "stock": 11
    }
  ]
}
```

Quinze produits, répartis ainsi :

| Catégorie | Produits | Nombre |
| --- | --- | --- |
| Périphériques | p01, p02, p03, p09, p12 | 5 |
| Audio | p04, p05 | 2 |
| Écrans | p06, p07 | 2 |
| Sièges | p08, p13 | 2 |
| Goodies | p10, p11 | 2 |
| Livres | p14, p15 | 2 |
| **Total** | | **15** |

Un seul produit est en rupture (`p03`, stock 0) et deux ont un stock faible (`p05` avec 3, `p08` avec 2). C'est volontaire : ces cas limites doivent être visibles dès le premier lancement, pas découverts en production.

> **Attention.** Le JSON n'autorise **pas** de virgule après le dernier élément d'un tableau ou d'un objet. `..., "stock": 11 },` suivi de `]` provoque `FormatException: Unexpected character`. C'est l'erreur de saisie la plus fréquente sur ce type de fichier.

> **Attention (2).** Après avoir ajouté une entrée dans la section `assets:` du `pubspec.yaml`, un simple *hot reload* ne suffit pas : il faut **arrêter et relancer** l'application (`flutter run`). Le paquet d'assets est construit au démarrage.

**État exécutable.** Le fichier existe et est déclaré. L'application ne le lit pas encore.

---

## 60.6 — Le dépôt de produits

### Pourquoi une interface

L'écran a besoin d'une liste de produits. Il n'a **pas** besoin de savoir d'où elle vient. Si vous écrivez `rootBundle.loadString` au milieu d'un `build`, trois choses deviennent impossibles :

- tester l'écran sans embarquer les assets ;
- remplacer plus tard le fichier par un appel réseau (chapitre 53) ;
- afficher un jeu de démonstration dans un test de widget.

La solution est le patron **dépôt** (`repository`) : une classe abstraite qui décrit ce que l'on veut, et des implémentations qui décrivent comment on l'obtient.

```text
      EcranCatalogue
            │  dépend de
            ▼
     DepotProduits (abstrait)
        ▲          ▲
        │          │
 DepotAssets   DepotMemoire
 (fichier JSON) (constantes Dart)
```

**`lib/donnees/depot_produits.dart`**

```dart
import '../modeles/produit.dart';

/// Source de produits, quelle qu'elle soit.
///
/// La méthode est asynchrone même quand l'implémentation est
/// instantanée. C'est un choix délibéré : le jour où le catalogue
/// vient du réseau, aucune signature ne change et aucun appelant
/// n'est à réécrire.
abstract class DepotProduits {
  Future<List<Produit>> chargerProduits();
}

/// Levée quand le catalogue est illisible.
///
/// On enveloppe l'erreur d'origine plutôt que de la laisser remonter
/// telle quelle : l'appelant a besoin d'un message affichable, pas
/// d'une `FormatException` brute (chapitre 13).
class ErreurCatalogue implements Exception {
  const ErreurCatalogue(this.message, [this.cause]);

  final String message;
  final Object? cause;

  @override
  String toString() => 'ErreurCatalogue: $message';
}
```

### L'implémentation qui lit les assets

**`lib/donnees/depot_assets.dart`**

```dart
import 'dart:convert';

import 'package:flutter/services.dart' show rootBundle;

import '../modeles/produit.dart';
import 'depot_produits.dart';

/// Charge le catalogue depuis un fichier JSON embarqué dans l'application.
///
/// C'est le seul fichier du dossier `donnees/` qui importe quelque
/// chose de Flutter : `rootBundle` en vient. Tout le reste de la
/// couche de données reste testable en Dart pur.
class DepotAssets implements DepotProduits {
  const DepotAssets({this.chemin = 'assets/donnees/catalogue.json'});

  /// Chemin de l'asset, tel qu'il est déclaré dans `pubspec.yaml`.
  final String chemin;

  @override
  Future<List<Produit>> chargerProduits() async {
    final String texte;
    try {
      texte = await rootBundle.loadString(chemin);
    } catch (erreur) {
      throw ErreurCatalogue(
        'Le fichier $chemin est introuvable. '
        'Vérifiez la section assets du pubspec.yaml, puis relancez '
        'complètement l\'application.',
        erreur,
      );
    }

    try {
      final Map<String, dynamic> racine =
          jsonDecode(texte) as Map<String, dynamic>;
      final List<dynamic> bruts =
          racine['produits'] as List<dynamic>? ?? <dynamic>[];

      return bruts
          .map((dynamic e) => Produit.fromJson(e as Map<String, dynamic>))
          .toList(growable: false);
    } catch (erreur) {
      throw ErreurCatalogue(
        'Le catalogue est mal formé. Une virgule en trop ou un '
        'guillemet manquant suffit à le rendre illisible.',
        erreur,
      );
    }
  }
}
```

> **Pourquoi deux `try` séparés ?** Parce que les deux échecs n'ont pas la même cause ni le même remède. « Fichier introuvable » signifie que le `pubspec.yaml` est incomplet ; « fichier mal formé » signifie que le JSON contient une faute. Un message générique du genre « une erreur est survenue » vous ferait perdre une demi-heure à chaque fois.

### L'implémentation en mémoire

Pour les tests et pour développer un écran sans dépendre du fichier, un jeu de données constant est précieux.

**`lib/donnees/depot_memoire.dart`**

```dart
import '../modeles/categorie.dart';
import '../modeles/produit.dart';
import 'depot_produits.dart';

/// Quelques produits en dur, utilisés par les tests et par les
/// aperçus. Ce fichier n'importe pas Flutter : il est utilisable
/// dans un test unitaire ordinaire.
const List<Produit> produitsDemonstration = <Produit>[
  Produit(
    id: 'p01',
    nom: 'Manette Nébula Pro',
    description: 'Manette sans fil à faible latence.',
    prixCentimes: 6490,
    categorie: Categorie.peripheriques,
    note: 4.6,
    stock: 12,
  ),
  Produit(
    id: 'p03',
    nom: 'Souris Photon 8K',
    description: 'Souris de 58 grammes, capteur optique haute précision.',
    prixCentimes: 7950,
    categorie: Categorie.peripheriques,
    note: 4.3,
    stock: 0,
  ),
  Produit(
    id: 'p04',
    nom: 'Casque Aurora 7.1',
    description: 'Casque fermé à son positionnel virtuel.',
    prixCentimes: 8990,
    categorie: Categorie.audio,
    note: 4.5,
    stock: 8,
  ),
  Produit(
    id: 'p05',
    nom: 'Micro Studio Écho',
    description: 'Micro à condensateur cardioïde.',
    prixCentimes: 11250,
    categorie: Categorie.audio,
    note: 4.1,
    stock: 3,
  ),
  Produit(
    id: 'p09',
    nom: 'Repose-poignets Velours',
    description: 'Mousse à mémoire de forme, housse lavable.',
    prixCentimes: 1990,
    categorie: Categorie.peripheriques,
    note: 4.0,
    stock: 30,
  ),
  Produit(
    id: 'p10',
    nom: 'Figurine du Boss Final',
    description: 'Résine peinte à la main, socle inclus.',
    prixCentimes: 2450,
    categorie: Categorie.goodies,
    note: 4.9,
    stock: 15,
  ),
  Produit(
    id: 'p11',
    nom: 'Mug Hello World',
    description: 'Céramique de 350 millilitres.',
    prixCentimes: 1290,
    categorie: Categorie.goodies,
    note: 4.2,
    stock: 40,
  ),
];

/// Dépôt volatile, sans entrée-sortie.
///
/// Le petit délai simulé n'est pas un caprice : il permet de voir
/// l'indicateur de chargement au moins une fois, donc de vérifier
/// qu'il fonctionne.
class DepotMemoire implements DepotProduits {
  const DepotMemoire({
    this.produits = produitsDemonstration,
    this.delai = Duration.zero,
  });

  final List<Produit> produits;
  final Duration delai;

  @override
  Future<List<Produit>> chargerProduits() async {
    if (delai > Duration.zero) {
      await Future<void>.delayed(delai);
    }
    return List<Produit>.unmodifiable(produits);
  }
}
```

**État exécutable.** Ces trois fichiers compilent. Rien n'a changé à l'écran ; le prochain pas y remédie.

---

## 60.7 — Formater les prix avec `intl`

### Le problème

`6490` est un entier. `'64,90 €'` est ce que le client doit lire. Entre les deux, trois décisions :

1. la virgule décimale française, pas le point anglais ;
2. deux décimales toujours, même pour 129,00 € ;
3. le symbole après le nombre, séparé par une espace insécable.

Écrire cela à la main est une source d'erreurs sans fin. Le paquet `intl` le fait correctement pour toutes les langues.

**`lib/utilitaires/formats.dart`**

```dart
import 'package:intl/intl.dart';

/// Formateur monétaire français.
///
/// Il est créé une seule fois, au niveau de la bibliothèque : la
/// construction d'un `NumberFormat` n'est pas gratuite, et le
/// reconstruire à chaque `build` de chaque carte serait un gaspillage
/// visible sur une grille de plusieurs centaines de cellules.
final NumberFormat _formatEuros = NumberFormat.currency(
  locale: 'fr_FR',
  symbol: '€',
  decimalDigits: 2,
);

/// Formate un prix exprimé **en centimes**.
///
/// La division par 100 n'a lieu qu'ici, au tout dernier moment, et
/// une seule fois par valeur affichée : aucune imprécision ne peut
/// donc s'accumuler. Le formateur arrondit ensuite à deux décimales.
///
/// ```dart
/// prixDepuisCentimes(6490);   // 64,90 €
/// prixDepuisCentimes(12900);  // 129,00 €
/// prixDepuisCentimes(0);      // 0,00 €
/// ```
String prixDepuisCentimes(int centimes) => _formatEuros.format(centimes / 100);

/// Formateur de note : une décimale, virgule française.
final NumberFormat _formatNote = NumberFormat('0.0', 'fr_FR');

/// Formate une note sur 5. `4.6` devient `4,6`.
String noteFormatee(double note) => _formatNote.format(note);

/// Formateur de date longue en français : « mardi 18 août 2026 ».
///
/// Contrairement à `NumberFormat`, `DateFormat` a besoin des données
/// de la locale. Il faut donc appeler `initializeDateFormatting('fr_FR')`
/// dans `main` avant de s'en servir (nous le ferons au 60.21).
final DateFormat _formatDateLongue = DateFormat('EEEE d MMMM y', 'fr_FR');

/// Formate une date en toutes lettres.
String dateLongue(DateTime date) => _formatDateLongue.format(date);
```

### Vérification

```dart
import 'package:pixel_boutique/utilitaires/formats.dart';

void main() {
  for (final int c in <int>[0, 490, 1290, 6490, 12900, 20380, 24456, 34900]) {
    print('$c centimes -> ${prixDepuisCentimes(c)}');
  }
  print(noteFormatee(4.6));
  print(noteFormatee(4.0));
}
```

**Résultat :**

```text
0 centimes -> 0,00 €
490 centimes -> 4,90 €
1290 centimes -> 12,90 €
6490 centimes -> 64,90 €
12900 centimes -> 129,00 €
20380 centimes -> 203,80 €
24456 centimes -> 244,56 €
34900 centimes -> 349,00 €
4,6
4,0
```

> **Remarque sur l'espace.** Le caractère entre le nombre et le `€` n'est pas une espace ordinaire mais une **espace insécable** (U+00A0). C'est voulu : elle empêche le symbole de se retrouver seul en début de ligne. Si vous écrivez un test qui compare à `'64,90 €'` tapé au clavier, il échouera. Comparez plutôt sur le nombre, ou construisez la chaîne attendue avec `' '`.

> **Pas d'initialisation pour les nombres.** `NumberFormat` fonctionne immédiatement pour toutes les locales : les symboles numériques sont compilés dans le paquet. Seul `DateFormat` réclame `initializeDateFormatting`. Beaucoup de débutants ajoutent l'appel « au cas où » partout ; il n'est nécessaire que pour les dates.

**État exécutable.** Le fichier compile. L'écran ne change pas encore.

---

## 60.8 — Une image sans fichier local

### La contrainte

Ce cours ne fournit aucun fichier image, et vous n'en avez pas besoin pour apprendre. Or une boutique sans visuel est illisible : l'utilisateur reconnaît un article à sa forme avant de lire son nom.

La réponse est de **générer** la vignette : un dégradé propre à la catégorie, et une icône Material au centre. Le résultat est reconnaissable, cohérent, il pèse zéro octet et il fonctionne hors ligne.

```text
  Périphériques      Audio           Écrans
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ indigo → │   │ sarcelle │   │  bleu →  │
  │  violet  │   │ → turquo.│   │   ciel   │
  │  (manette)│  │ (casque) │   │ (écran)  │
  └──────────┘   └──────────┘   └──────────┘

  Sièges          Goodies         Livres
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ brique → │   │ framboise│   │ olive →  │
  │  orange  │   │  → rose  │   │  citron  │
  │ (fauteuil)│  │ (cadeau) │   │ (livre)  │
  └──────────┘   └──────────┘   └──────────┘
```

### La table d'apparence

Les couleurs et les icônes appartiennent à l'interface, pas au modèle. C'est pourquoi `Categorie` ne les connaît pas : cette table vit dans `utilitaires/`, le seul endroit où l'on a le droit d'importer `material.dart`.

**`lib/utilitaires/apparence.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/categorie.dart';

/// Aspect visuel associé à une catégorie : une icône et deux couleurs
/// formant le dégradé de la vignette.
class ApparenceCategorie {
  const ApparenceCategorie({
    required this.icone,
    required this.debut,
    required this.fin,
  });

  final IconData icone;
  final Color debut;
  final Color fin;
}

/// Table d'apparence. Elle est `const` : elle ne coûte rien à
/// l'exécution et Dart la partage entre tous les widgets.
const Map<Categorie, ApparenceCategorie> apparences =
    <Categorie, ApparenceCategorie>{
  Categorie.peripheriques: ApparenceCategorie(
    icone: Icons.videogame_asset,
    debut: Color(0xFF4338CA),
    fin: Color(0xFF818CF8),
  ),
  Categorie.audio: ApparenceCategorie(
    icone: Icons.headphones,
    debut: Color(0xFF0F766E),
    fin: Color(0xFF2DD4BF),
  ),
  Categorie.ecrans: ApparenceCategorie(
    icone: Icons.desktop_windows,
    debut: Color(0xFF1D4ED8),
    fin: Color(0xFF60A5FA),
  ),
  Categorie.sieges: ApparenceCategorie(
    icone: Icons.event_seat,
    debut: Color(0xFF9A3412),
    fin: Color(0xFFFB923C),
  ),
  Categorie.goodies: ApparenceCategorie(
    icone: Icons.redeem,
    debut: Color(0xFF9D174D),
    fin: Color(0xFFF472B6),
  ),
  Categorie.livres: ApparenceCategorie(
    icone: Icons.menu_book,
    debut: Color(0xFF3F6212),
    fin: Color(0xFFA3E635),
  ),
};

/// Renvoie l'apparence d'une catégorie.
///
/// L'opérateur `!` est sûr ici, et seulement ici : la table couvre
/// exhaustivement `Categorie.values`. Si vous ajoutez une valeur à
/// l'`enum` sans compléter la table, le `!` lèvera immédiatement une
/// erreur au premier affichage — ce qui est exactement le
/// comportement souhaité, bien préférable à un affichage silencieux
/// et faux.
ApparenceCategorie apparenceDe(Categorie categorie) => apparences[categorie]!;
```

### Le widget de vignette

**`lib/widgets/vignette_produit.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/produit.dart';
import '../utilitaires/apparence.dart';

/// Visuel d'un produit, entièrement dessiné par Flutter.
///
/// Ce widget **remplit** l'espace que son parent lui donne. Il doit
/// donc toujours être placé sous une contrainte bornée : un
/// `Expanded`, un `AspectRatio`, un `SizedBox`. Le placer dans une
/// `Column` sans contrainte provoque l'erreur classique
/// « BoxConstraints forces an infinite height ».
class VignetteProduit extends StatelessWidget {
  const VignetteProduit({
    super.key,
    required this.produit,
    this.tailleIcone = 56,
    this.rayon = 0,
  });

  final Produit produit;

  /// Taille de l'icône centrale, en pixels logiques.
  final double tailleIcone;

  /// Rayon des coins arrondis. Zéro dans une carte, qui découpe
  /// déjà ses propres coins ; non nul sur la fiche produit.
  final double rayon;

  @override
  Widget build(BuildContext context) {
    final ApparenceCategorie apparence = apparenceDe(produit.categorie);

    return ClipRRect(
      borderRadius: BorderRadius.circular(rayon),
      child: DecoratedBox(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: <Color>[apparence.debut, apparence.fin],
          ),
        ),
        child: Center(
          child: Icon(
            apparence.icone,
            size: tailleIcone,
            // `withValues` remplace l'ancien `withOpacity`, retiré
            // des versions récentes de Flutter.
            color: Colors.white.withValues(alpha: 0.92),
          ),
        ),
      ),
    );
  }
}
```

### Variante : une vraie image réseau

Si vous voulez de vraies photographies, remplacez le corps du `build` par ceci. `Image.network` doit **toujours** être accompagné de ses deux rappels : sans `errorBuilder`, une coupure réseau affiche une zone grise et une exception rouge en développement ; sans `loadingBuilder`, l'utilisateur voit un trou pendant le téléchargement.

**`lib/widgets/vignette_produit.dart`** (variante à substituer)

```dart
  @override
  Widget build(BuildContext context) {
    final ApparenceCategorie apparence = apparenceDe(produit.categorie);

    // `picsum.photos` renvoie toujours la même image pour une graine
    // donnée : la vignette d'un produit reste stable d'un lancement
    // à l'autre. Avec un vrai catalogue, l'URL viendrait du JSON.
    final String url = 'https://picsum.photos/seed/${produit.id}/600/600';

    Widget deSecours() {
      return DecoratedBox(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: <Color>[apparence.debut, apparence.fin],
          ),
        ),
        child: Center(
          child: Icon(
            apparence.icone,
            size: tailleIcone,
            color: Colors.white.withValues(alpha: 0.92),
          ),
        ),
      );
    }

    return ClipRRect(
      borderRadius: BorderRadius.circular(rayon),
      child: Image.network(
        url,
        fit: BoxFit.cover,
        // Pendant le téléchargement : le dégradé, plus une barre de
        // progression quand la taille totale est connue.
        loadingBuilder: (
          BuildContext context,
          Widget enfant,
          ImageChunkEvent? progres,
        ) {
          if (progres == null) {
            return enfant; // image complètement chargée
          }
          final int? total = progres.expectedTotalBytes;
          return Stack(
            fit: StackFit.expand,
            children: <Widget>[
              deSecours(),
              Align(
                alignment: Alignment.bottomCenter,
                child: LinearProgressIndicator(
                  value: total == null
                      ? null
                      : progres.cumulativeBytesLoaded / total,
                ),
              ),
            ],
          );
        },
        // En cas d'échec : le dégradé, définitivement.
        errorBuilder: (
          BuildContext context,
          Object erreur,
          StackTrace? trace,
        ) {
          return deSecours();
        },
      ),
    );
  }
```

> **Attention.** `Image.network` exige l'autorisation Internet. Sur Android, ajoutez `<uses-permission android:name="android.permission.INTERNET"/>` dans `android/app/src/main/AndroidManifest.xml` (chapitre 53). Sur le Web, le serveur d'images doit autoriser le partage entre origines. Le reste du chapitre utilise la vignette dessinée, qui n'a aucune de ces contraintes.

**État exécutable.** Les deux fichiers compilent. On peut enfin construire une carte.

---

## 60.9 — La carte produit

### Les étoiles

**`lib/widgets/etoiles_note.dart`**

```dart
import 'package:flutter/material.dart';

import '../utilitaires/formats.dart';

/// Affiche une note sur 5 sous forme de cinq étoiles, plus la valeur
/// chiffrée.
///
/// L'arrondi se fait au demi-point le plus proche : 4,6 donne quatre
/// étoiles pleines et une demie, tout comme 4,9. C'est le
/// comportement attendu par les utilisateurs ; la valeur exacte reste
/// affichée à côté.
class EtoilesNote extends StatelessWidget {
  const EtoilesNote({
    super.key,
    required this.note,
    this.taille = 16,
    this.afficherValeur = true,
  });

  final double note;
  final double taille;
  final bool afficherValeur;

  /// Choisit l'icône de la i-ème étoile (i allant de 1 à 5).
  IconData _iconePour(int i) {
    if (note >= i) {
      return Icons.star;
    }
    if (note >= i - 0.5) {
      return Icons.star_half;
    }
    return Icons.star_border;
  }

  @override
  Widget build(BuildContext context) {
    final Color couleur = Colors.amber.shade700;

    return Row(
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        for (int i = 1; i <= 5; i++)
          Icon(_iconePour(i), size: taille, color: couleur),
        if (afficherValeur) ...<Widget>[
          SizedBox(width: taille * 0.35),
          Text(
            noteFormatee(note),
            style: TextStyle(
              fontSize: taille * 0.85,
              color: Theme.of(context).colorScheme.onSurfaceVariant,
            ),
          ),
        ],
      ],
    );
  }
}
```

Vérifions le rendu pour les notes du catalogue :

| Note | Étoiles pleines | Demie | Vides |
| --- | --- | --- | --- |
| 4,9 | 4 | 1 | 0 |
| 4,8 | 4 | 1 | 0 |
| 4,5 | 4 | 1 | 0 |
| 4,3 | 4 | 0 | 1 |
| 4,0 | 4 | 0 | 1 |
| 3,9 | 3 | 1 | 1 |

Pour 4,3 : la cinquième étoile demande `note >= 4.5`, ce qui est faux, donc `star_border`. Pour 3,9 : la quatrième demande `note >= 4`, faux, puis `note >= 3.5`, vrai, donc `star_half`.

### La carte

Le point délicat d'une carte dans une grille est qu'elle a une **hauteur imposée** par le `gridDelegate`. Si le contenu dépasse, Flutter affiche la barre rayée jaune et noire de débordement.

La parade est structurelle : un `Expanded` autour de la vignette. Le bloc de texte prend la place dont il a besoin, la vignette absorbe tout le reste. Il ne peut alors plus y avoir de débordement, quelle que soit la hauteur de cellule.

```text
  ┌─────────────────┐  ← hauteur imposée par le gridDelegate
  │                 │
  │    Expanded     │  ← la vignette prend ce qui reste
  │   (vignette)    │
  │                 │
  ├─────────────────┤
  │ Nom sur deux    │  ← hauteur naturelle du texte
  │ lignes au plus  │
  │ ★★★★☆ 4,6       │
  │ 64,90 €         │
  └─────────────────┘
```

**`lib/widgets/carte_produit.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/produit.dart';
import '../utilitaires/formats.dart';
import 'etoiles_note.dart';
import 'vignette_produit.dart';

/// Une cellule de la grille du catalogue.
class CarteProduit extends StatelessWidget {
  const CarteProduit({
    super.key,
    required this.produit,
    required this.onTap,
  });

  final Produit produit;

  /// Action déclenchée par un appui. La carte ne décide pas de la
  /// navigation : c'est l'écran qui la lui fournit. Ce widget reste
  /// ainsi réutilisable ailleurs (résultats de recherche, favoris).
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Card(
      // `Clip.antiAlias` fait respecter les coins arrondis par la
      // vignette, qui sinon dépasserait aux quatre angles.
      clipBehavior: Clip.antiAlias,
      margin: EdgeInsets.zero,
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Expanded(
              child: Stack(
                fit: StackFit.expand,
                children: <Widget>[
                  VignetteProduit(produit: produit),
                  if (!produit.enStock) const _BandeauRupture(),
                  if (produit.stockFaible)
                    _Pastille(
                      texte: 'Plus que ${produit.stock}',
                      couleur: Colors.orange.shade800,
                    ),
                ],
              ),
            ),
            Padding(
              padding: const EdgeInsets.fromLTRB(10, 8, 10, 10),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisSize: MainAxisSize.min,
                children: <Widget>[
                  Text(
                    produit.nom,
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                    style: theme.textTheme.titleSmall,
                  ),
                  const SizedBox(height: 4),
                  EtoilesNote(note: produit.note, taille: 13),
                  const SizedBox(height: 6),
                  Text(
                    prixDepuisCentimes(produit.prixCentimes),
                    style: theme.textTheme.titleMedium?.copyWith(
                      fontWeight: FontWeight.bold,
                      color: theme.colorScheme.primary,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Voile sombre et mention « rupture » posés sur la vignette.
class _BandeauRupture extends StatelessWidget {
  const _BandeauRupture();

  @override
  Widget build(BuildContext context) {
    return DecoratedBox(
      decoration: BoxDecoration(color: Colors.black.withValues(alpha: 0.45)),
      child: const Center(
        child: Text(
          'RUPTURE',
          style: TextStyle(
            color: Colors.white,
            fontWeight: FontWeight.bold,
            letterSpacing: 1.5,
          ),
        ),
      ),
    );
  }
}

/// Petite étiquette posée en haut à gauche de la vignette.
class _Pastille extends StatelessWidget {
  const _Pastille({required this.texte, required this.couleur});

  final String texte;
  final Color couleur;

  @override
  Widget build(BuildContext context) {
    return Align(
      alignment: Alignment.topLeft,
      child: Padding(
        padding: const EdgeInsets.all(6),
        child: DecoratedBox(
          decoration: BoxDecoration(
            color: couleur,
            borderRadius: BorderRadius.circular(6),
          ),
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
            child: Text(
              texte,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 11,
                fontWeight: FontWeight.w600,
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

> **Remarque.** `_BandeauRupture` et `_Pastille` sont préfixés d'un souligné : ce sont des classes **privées au fichier** (chapitre 10). Elles n'ont aucune raison d'être visibles ailleurs. C'est la bonne façon de découper un widget complexe sans polluer l'espace de noms du projet.

**État exécutable.** Le fichier compile. Il ne reste plus qu'à poser ces cartes dans une grille.

---

## 60.10 — La grille responsive

### Deux `gridDelegate`, un seul bon choix

Le chapitre 48 vous a montré `SliverGridDelegateWithFixedCrossAxisCount`, qui impose un nombre de colonnes. C'est le mauvais outil ici : 2 colonnes sur un téléphone sont correctes, sur un écran de bureau elles donnent des cartes de 600 pixels de large, ridicules.

Le bon outil est `SliverGridDelegateWithMaxCrossAxisExtent` : on ne fixe pas un nombre de colonnes, on fixe une **largeur maximale de cellule**, et Flutter calcule le nombre de colonnes qui convient.

L'algorithme exact, tiré du moteur de rendu, est le suivant :

```text
colonnes  = ceil( largeurDisponible / (maxCrossAxisExtent + crossAxisSpacing) )
colonnes  = max(1, colonnes)
utile     = largeurDisponible - crossAxisSpacing * (colonnes - 1)
largeurCellule = utile / colonnes
hauteurCellule = largeurCellule / childAspectRatio
```

Avec nos réglages — `maxCrossAxisExtent: 220`, `crossAxisSpacing: 12`, `mainAxisSpacing: 12`, `childAspectRatio: 0.68` et une marge intérieure de 12 de chaque côté :

| Appareil | Largeur écran | Largeur utile | Calcul | Colonnes | Cellule (l × h) |
| --- | --- | --- | --- | --- | --- |
| Téléphone portrait | 411 | 387 | 387 / 232 = 1,67 → 2 | 2 | 187,5 × 275,7 |
| Téléphone paysage | 731 | 707 | 707 / 232 = 3,05 → 4 | 4 | 167,8 × 246,7 |
| Tablette | 834 | 810 | 810 / 232 = 3,49 → 4 | 4 | 193,5 × 284,6 |
| Bureau | 1280 | 1256 | 1256 / 232 = 5,41 → 6 | 6 | 199,3 × 293,1 |

Vérifions la première ligne pas à pas, c'est le calcul que l'on refait le plus souvent :

```text
largeur écran            411
- marge gauche (12)      399
- marge droite (12)      387   ← largeurDisponible
387 / (220 + 12)       = 1,668
ceil(1,668)            = 2     ← colonnes
387 - 12 * (2 - 1)     = 375   ← utile
375 / 2                = 187,5 ← largeur d'une cellule
187,5 / 0,68           = 275,7 ← hauteur d'une cellule
```

Notez que la largeur obtenue (187,5) est **inférieure** au maximum demandé (220). C'est le principe : `maxCrossAxisExtent` est un plafond, pas une consigne.

> **Le sens de `childAspectRatio`.** C'est `largeur / hauteur`. Une valeur **inférieure à 1** donne une cellule plus haute que large — ce que nous voulons pour une fiche produit. Beaucoup de débutants font l'erreur inverse et se demandent pourquoi leur carte est écrasée. Souvenez-vous : `0.68` signifie « la hauteur vaut environ 1,47 fois la largeur ».

### La grille

**`lib/widgets/grille_produits.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/produit.dart';
import 'carte_produit.dart';

/// Grille responsive de cartes produit.
///
/// Le nombre de colonnes n'est écrit nulle part : il découle de la
/// largeur disponible. Le même widget sert sur téléphone, sur
/// tablette et sur bureau.
class GrilleProduits extends StatelessWidget {
  const GrilleProduits({
    super.key,
    required this.produits,
    required this.onProduitTouche,
  });

  final List<Produit> produits;
  final void Function(Produit produit) onProduitTouche;

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      padding: const EdgeInsets.all(12),
      gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
        maxCrossAxisExtent: 220,
        mainAxisSpacing: 12,
        crossAxisSpacing: 12,
        childAspectRatio: 0.68,
      ),
      itemCount: produits.length,
      itemBuilder: (BuildContext context, int index) {
        final Produit produit = produits[index];
        return CarteProduit(
          produit: produit,
          onTap: () => onProduitTouche(produit),
        );
      },
    );
  }
}
```

> **Pourquoi `.builder` ?** Comme pour `ListView`, la variante `GridView.builder` ne construit que les cellules visibles, plus une petite marge. Le constructeur `GridView(children: ...)` construit **tout**, y compris les 480 cartes qu'on ne verra jamais. Sur un catalogue, `.builder` n'est pas une optimisation : c'est la seule forme correcte.

**État exécutable.** Le fichier compile. Branchons-le.

---

## 60.11 — L'écran de catalogue

Le chargement est asynchrone : il faut donc gérer trois états — en cours, en erreur, réussi. `FutureBuilder` est fait pour cela (chapitre 53).

**Règle absolue :** le `Future` est créé dans `initState`, **jamais** dans `build`. Un `Future` créé dans `build` est relancé à chaque reconstruction, ce qui produit une boucle de chargement infinie.

**`lib/ecrans/ecran_catalogue.dart`**

```dart
import 'package:flutter/material.dart';

import '../donnees/depot_assets.dart';
import '../donnees/depot_produits.dart';
import '../modeles/produit.dart';
import '../widgets/grille_produits.dart';

class EcranCatalogue extends StatefulWidget {
  const EcranCatalogue({super.key, this.depot = const DepotAssets()});

  /// Injecté pour pouvoir passer un `DepotMemoire` dans les tests.
  final DepotProduits depot;

  @override
  State<EcranCatalogue> createState() => _EcranCatalogueState();
}

class _EcranCatalogueState extends State<EcranCatalogue> {
  late Future<List<Produit>> _futurProduits;

  @override
  void initState() {
    super.initState();
    _futurProduits = widget.depot.chargerProduits();
  }

  void _recharger() {
    setState(() {
      _futurProduits = widget.depot.chargerProduits();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Pixel Boutique')),
      body: FutureBuilder<List<Produit>>(
        future: _futurProduits,
        builder: (BuildContext context, AsyncSnapshot<List<Produit>> etat) {
          if (etat.connectionState != ConnectionState.done) {
            return const Center(child: CircularProgressIndicator());
          }
          if (etat.hasError) {
            return _MessageErreur(
              erreur: etat.error!,
              onReessayer: _recharger,
            );
          }

          final List<Produit> produits = etat.data ?? <Produit>[];
          if (produits.isEmpty) {
            return const Center(child: Text('Le catalogue est vide.'));
          }

          return GrilleProduits(
            produits: produits,
            onProduitTouche: (Produit produit) {
              // Provisoire : la fiche produit arrive au 60.12.
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text(produit.nom)),
              );
            },
          );
        },
      ),
    );
  }
}

/// Affichage d'erreur avec un bouton pour réessayer.
///
/// Un écran d'erreur sans action est une impasse : l'utilisateur ne
/// peut que fermer l'application. Il faut toujours proposer une
/// sortie.
class _MessageErreur extends StatelessWidget {
  const _MessageErreur({required this.erreur, required this.onReessayer});

  final Object erreur;
  final VoidCallback onReessayer;

  @override
  Widget build(BuildContext context) {
    final String texte = erreur is ErreurCatalogue
        ? (erreur as ErreurCatalogue).message
        : 'Le catalogue n\'a pas pu être chargé.';

    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Icon(
              Icons.error_outline,
              size: 56,
              color: Theme.of(context).colorScheme.error,
            ),
            const SizedBox(height: 16),
            Text(texte, textAlign: TextAlign.center),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: onReessayer,
              icon: const Icon(Icons.refresh),
              label: const Text('Réessayer'),
            ),
          ],
        ),
      ),
    );
  }
}
```

Mettez enfin à jour `main.dart` pour importer le vrai écran et supprimer l'écran provisoire.

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

import 'ecrans/ecran_catalogue.dart';

void main() {
  runApp(const ApplicationBoutique());
}

class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Pixel Boutique',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4F46E5)),
      ),
      home: const EcranCatalogue(),
    );
  }
}
```

**État exécutable.** Le catalogue s'affiche réellement : quinze cartes, deux colonnes sur téléphone, la souris `p03` barrée d'un bandeau RUPTURE, le micro `p05` et le siège `p08` marqués « Plus que 3 » et « Plus que 2 ». Tournez l'appareil : le nombre de colonnes change tout seul.

Pour vérifier l'écran d'erreur sans casser votre JSON, lancez temporairement :

```dart
home: const EcranCatalogue(depot: DepotAssets(chemin: 'assets/donnees/absent.json')),
```

**Résultat :**

```text
┌────────────────────────────────────────────────┐
│  Pixel Boutique                                │
├────────────────────────────────────────────────┤
│                     (!)                        │
│    Le fichier assets/donnees/absent.json est   │
│    introuvable. Vérifiez la section assets du  │
│    pubspec.yaml, puis relancez complètement    │
│    l'application.                              │
│              [ Réessayer ]                     │
└────────────────────────────────────────────────┘
```

---

## 60.12 — La fiche produit

Un nouvel écran, atteint par `Navigator.push`, à qui l'on passe le produit par le constructeur (chapitre 50). C'est la façon la plus simple et la plus sûre de transmettre une donnée : elle est typée, et le compilateur vérifie qu'elle est fournie.

Nous avons d'abord besoin d'un sélecteur de quantité, réutilisé plus tard dans le panier.

**`lib/widgets/selecteur_quantite.dart`**

```dart
import 'package:flutter/material.dart';

/// Trio « − valeur + ».
///
/// Les bornes sont explicites : en dessous de [minimum] et au-dessus
/// de [maximum], le bouton correspondant est désactivé. Un bouton
/// désactivé est infiniment préférable à un bouton qui ne fait rien :
/// l'utilisateur comprend immédiatement pourquoi.
class SelecteurQuantite extends StatelessWidget {
  const SelecteurQuantite({
    super.key,
    required this.valeur,
    required this.onChange,
    this.minimum = 1,
    required this.maximum,
  });

  final int valeur;
  final int minimum;
  final int maximum;
  final void Function(int nouvelleValeur) onChange;

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: <Widget>[
        IconButton.filledTonal(
          onPressed: valeur > minimum ? () => onChange(valeur - 1) : null,
          icon: const Icon(Icons.remove),
          tooltip: 'Diminuer',
        ),
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 12),
          child: Text(
            '$valeur',
            style: Theme.of(context).textTheme.titleMedium,
          ),
        ),
        IconButton.filledTonal(
          onPressed: valeur < maximum ? () => onChange(valeur + 1) : null,
          icon: const Icon(Icons.add),
          tooltip: 'Augmenter',
        ),
      ],
    );
  }
}
```

**`lib/ecrans/ecran_produit.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/produit.dart';
import '../utilitaires/formats.dart';
import '../widgets/etoiles_note.dart';
import '../widgets/selecteur_quantite.dart';
import '../widgets/vignette_produit.dart';

/// Fiche détaillée d'un produit.
///
/// L'écran est `Stateful` uniquement à cause de la quantité choisie,
/// qui est un état local et éphémère : elle disparaît avec l'écran.
/// Le panier, lui, sera un état global (60.15).
class EcranProduit extends StatefulWidget {
  const EcranProduit({super.key, required this.produit});

  final Produit produit;

  @override
  State<EcranProduit> createState() => _EcranProduitState();
}

class _EcranProduitState extends State<EcranProduit> {
  int _quantite = 1;

  @override
  Widget build(BuildContext context) {
    final Produit produit = widget.produit;
    final ThemeData theme = Theme.of(context);
    final int totalCentimes = produit.prixCentimes * _quantite;

    return Scaffold(
      appBar: AppBar(title: Text(produit.nom)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          // Un carré parfait, quelle que soit la largeur de l'écran.
          AspectRatio(
            aspectRatio: 1,
            child: VignetteProduit(
              produit: produit,
              tailleIcone: 120,
              rayon: 20,
            ),
          ),
          const SizedBox(height: 16),
          Chip(
            label: Text(produit.categorie.libelle),
            visualDensity: VisualDensity.compact,
          ),
          const SizedBox(height: 8),
          Text(produit.nom, style: theme.textTheme.headlineSmall),
          const SizedBox(height: 8),
          EtoilesNote(note: produit.note, taille: 20),
          const SizedBox(height: 16),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: <Widget>[
              Text(
                prixDepuisCentimes(produit.prixCentimes),
                style: theme.textTheme.headlineMedium?.copyWith(
                  fontWeight: FontWeight.bold,
                  color: theme.colorScheme.primary,
                ),
              ),
              _EtiquetteStock(produit: produit),
            ],
          ),
          const SizedBox(height: 16),
          Text(produit.description, style: theme.textTheme.bodyLarge),
          const SizedBox(height: 24),
          if (produit.enStock) ...<Widget>[
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: <Widget>[
                Text('Quantité', style: theme.textTheme.titleMedium),
                SelecteurQuantite(
                  valeur: _quantite,
                  maximum: produit.stock,
                  onChange: (int n) => setState(() => _quantite = n),
                ),
              ],
            ),
            const SizedBox(height: 16),
            FilledButton.icon(
              onPressed: () {
                // Provisoire : le panier arrive au 60.15.
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('$_quantite × ${produit.nom}'),
                  ),
                );
              },
              icon: const Icon(Icons.add_shopping_cart),
              label: Text(
                'Ajouter au panier — ${prixDepuisCentimes(totalCentimes)}',
              ),
              style: FilledButton.styleFrom(
                minimumSize: const Size.fromHeight(52),
              ),
            ),
          ] else
            const Card(
              child: ListTile(
                leading: Icon(Icons.inventory_2_outlined),
                title: Text('Article indisponible'),
                subtitle: Text('Ce produit sera de nouveau en stock '
                    'prochainement.'),
              ),
            ),
        ],
      ),
    );
  }
}

/// « En stock (12) », « Plus que 3 » ou « Rupture ».
class _EtiquetteStock extends StatelessWidget {
  const _EtiquetteStock({required this.produit});

  final Produit produit;

  @override
  Widget build(BuildContext context) {
    final (String texte, Color couleur) = switch (produit) {
      final Produit p when !p.enStock => ('Rupture', Colors.red.shade700),
      final Produit p when p.stockFaible =>
        ('Plus que ${p.stock}', Colors.orange.shade800),
      final Produit p => ('En stock (${p.stock})', Colors.green.shade700),
    };

    return Text(
      texte,
      style: TextStyle(color: couleur, fontWeight: FontWeight.w600),
    );
  }
}
```

> **Remarque.** Le `switch` d'expression avec clauses `when` et l'enregistrement `(String, Color)` viennent de Dart 3 (chapitre 11). Si cette forme vous gêne encore, une suite de `if` produirait exactement le même résultat.

Reste à ouvrir cet écran. Dans `ecran_catalogue.dart`, remplacez le rappel provisoire :

```dart
          return GrilleProduits(
            produits: produits,
            onProduitTouche: (Produit produit) {
              Navigator.of(context).push(
                MaterialPageRoute<void>(
                  builder: (BuildContext context) =>
                      EcranProduit(produit: produit),
                ),
              );
            },
          );
```

sans oublier l'import :

```dart
import 'ecran_produit.dart';
```

**État exécutable.** Toucher une carte ouvre sa fiche. Le bouton retour de la barre d'application ramène à la grille, à la position de défilement d'origine.

---

## 60.13 — L'animation `Hero`

### Le principe

`Hero` relie deux widgets portant le **même `tag`** sur deux écrans différents. Pendant la transition, Flutter retire les deux widgets de leur place, en fabrique un troisième qui vole de la position de départ à la position d'arrivée, puis le remet en place. Vous n'écrivez ni animation, ni contrôleur, ni interpolation : seulement deux `tag` identiques.

```text
   Écran catalogue                 Écran produit
   ┌───────────────┐               ┌───────────────┐
   │ ┌────┐        │    vol de     │ ┌───────────┐ │
   │ │ ▨  │ ──────────────────────►│ │     ▨     │ │
   │ └────┘        │  ~300 ms      │ └───────────┘ │
   └───────────────┘               └───────────────┘
   tag: 'vignette-p01'             tag: 'vignette-p01'
```

### Le tag

Le `tag` doit être **unique sur chaque écran** et **identique entre les deux**. L'identifiant du produit répond aux deux exigences ; l'index dans la grille n'y répond à aucune des deux.

Créez une fonction unique, pour que les deux écrans ne puissent pas diverger.

**`lib/widgets/vignette_produit.dart`** (à ajouter en fin de fichier)

```dart
/// Étiquette d'animation `Hero` d'un produit.
///
/// Une seule fonction pour les deux écrans : impossible d'écrire
/// `'vignette_p01'` d'un côté et `'vignette-p01'` de l'autre, faute
/// qui casse l'animation en silence.
String tagHeroProduit(Produit produit) => 'vignette-${produit.id}';
```

### Côté grille

Dans `carte_produit.dart`, enveloppez la vignette :

```dart
            Expanded(
              child: Stack(
                fit: StackFit.expand,
                children: <Widget>[
                  Hero(
                    tag: tagHeroProduit(produit),
                    child: VignetteProduit(produit: produit),
                  ),
                  if (!produit.enStock) const _BandeauRupture(),
                  if (produit.stockFaible)
                    _Pastille(
                      texte: 'Plus que ${produit.stock}',
                      couleur: Colors.orange.shade800,
                    ),
                ],
              ),
            ),
```

Le bandeau et la pastille restent **en dehors** du `Hero` : seul le visuel commun aux deux écrans doit voler, sinon la transition est visuellement incohérente.

### Côté fiche

Dans `ecran_produit.dart` :

```dart
          AspectRatio(
            aspectRatio: 1,
            child: Hero(
              tag: tagHeroProduit(produit),
              child: VignetteProduit(
                produit: produit,
                tailleIcone: 120,
                rayon: 20,
              ),
            ),
          ),
```

**État exécutable.** La vignette se déplace et grandit en douceur de la carte vers la fiche, et revient au retour. Ralentissez l'animation avec `timeDilation = 5.0` (depuis `package:flutter/scheduler.dart`) pour l'observer image par image.

> **Les trois pièges de `Hero`.**
>
> 1. `There are multiple heroes that share the same tag within a subtree.` Deux widgets du même écran portent le même `tag`. Cela arrive dès qu'un produit apparaît deux fois — par exemple dans le panier et dans un bandeau « suggestions ». Un seul `Hero` par produit et par écran.
> 2. L'animation ne se produit pas. Les deux écrans doivent appartenir au **même** `Navigator`. Une `showModalBottomSheet` ou un `showDialog` ne déclenchent pas de vol.
> 3. Le contenu du `Hero` clignote. Évitez d'y mettre un `Material` ou un `Card` : pendant le vol, le widget est détaché de son arbre et perd son contexte de thème. Faites voler la vignette, pas la carte entière.

---

## 60.14 — Le modèle de panier

### La ligne

Un panier n'est pas une liste de produits : c'est une liste de **couples** produit + quantité. C'est le modèle `LignePanier`.

**`lib/modeles/ligne_panier.dart`**

```dart
import 'produit.dart';

/// Une ligne du panier : un produit et sa quantité.
class LignePanier {
  const LignePanier({required this.produit, required this.quantite})
      : assert(quantite > 0, 'Une ligne de quantité nulle ne doit pas exister');

  final Produit produit;
  final int quantite;

  /// Montant de la ligne, en centimes. Multiplication d'entiers :
  /// exacte par construction.
  int get totalCentimes => produit.prixCentimes * quantite;

  LignePanier copyWith({int? quantite}) {
    return LignePanier(
      produit: produit,
      quantite: quantite ?? this.quantite,
    );
  }

  /// Forme persistée : on n'enregistre **que** l'identifiant et la
  /// quantité, jamais le produit complet. Le prix, le nom et le stock
  /// appartiennent au catalogue et peuvent changer entre deux
  /// lancements ; les figer dans le panier reviendrait à vendre au
  /// prix d'hier.
  Map<String, dynamic> toJson() => <String, dynamic>{
        'produit_id': produit.id,
        'quantite': quantite,
      };

  @override
  String toString() => '$quantite × ${produit.nom}';
}
```

### Les tarifs

Le calcul est une règle métier. Il ne doit dépendre ni de Flutter, ni d'un écran, ni d'un `ChangeNotifier` : c'est une fonction pure, donc testable en une ligne.

**`lib/modeles/tarifs.dart`**

```dart
/// Détail chiffré d'un panier, entièrement en centimes.
class Tarifs {
  const Tarifs({
    required this.sousTotalCentimes,
    required this.portCentimes,
    required this.tvaCentimes,
    required this.totalCentimes,
  });

  /// Somme des lignes, hors taxes et hors port.
  final int sousTotalCentimes;

  /// Frais de livraison, hors taxes. Zéro si le franco est atteint.
  final int portCentimes;

  /// Taxe sur la valeur ajoutée.
  final int tvaCentimes;

  /// Ce que le client paie réellement.
  final int totalCentimes;

  static const Tarifs vide = Tarifs(
    sousTotalCentimes: 0,
    portCentimes: 0,
    tvaCentimes: 0,
    totalCentimes: 0,
  );

  bool get portOffert => portCentimes == 0 && sousTotalCentimes > 0;

  @override
  String toString() => 'Tarifs(HT $sousTotalCentimes + port $portCentimes '
      '+ TVA $tvaCentimes = $totalCentimes)';
}
```

**`lib/logique/calcul_tarifs.dart`**

```dart
import '../modeles/ligne_panier.dart';
import '../modeles/tarifs.dart';

/// Taux de TVA, exprimé en pour cent entiers.
const int tauxTvaPourCent = 20;

/// Frais de livraison hors taxes, en centimes : 4,90 €.
const int fraisPortCentimes = 490;

/// Franco de port à partir de 100,00 € hors taxes de marchandise.
const int seuilFrancoCentimes = 10000;

/// Calcule le détail chiffré d'un panier.
///
/// Règles appliquées, dans cet ordre :
///  1. le sous-total est la somme des lignes ;
///  2. le port est offert si le sous-total atteint [seuilFrancoCentimes] ;
///  3. la TVA porte sur la marchandise **et** sur le port, comme le
///     veut la réglementation française ;
///  4. le total est la somme des trois.
///
/// Tout est en `int` : le résultat est exact au centime.
Tarifs calculerTarifs(List<LignePanier> lignes) {
  if (lignes.isEmpty) {
    return Tarifs.vide;
  }

  final int sousTotal = lignes.fold<int>(
    0,
    (int somme, LignePanier ligne) => somme + ligne.totalCentimes,
  );

  final int port = sousTotal >= seuilFrancoCentimes ? 0 : fraisPortCentimes;
  final int base = sousTotal + port;

  // `round()` n'a d'effet que sur des prix qui ne sont pas multiples
  // de 5 centimes ; il garantit malgré tout un entier, et une règle
  // d'arrondi unique et documentée.
  final int tva = (base * tauxTvaPourCent / 100).round();

  return Tarifs(
    sousTotalCentimes: sousTotal,
    portCentimes: port,
    tvaCentimes: tva,
    totalCentimes: base + tva,
  );
}
```

### Vérification à la main

Reprenons le panier de la maquette du 60.0.1 : une manette, deux figurines, un casque.

```text
  1 × Manette Nébula Pro       6490 × 1 =   6490
  2 × Figurine du Boss Final   2450 × 2 =   4900
  1 × Casque Aurora 7.1        8990 × 1 =   8990
                                          ──────
  Sous-total HT                             20380   →  203,80 €
  20380 >= 10000  →  port offert                0   →    0,00 €
  Base taxable                              20380
  TVA = 20380 × 20 / 100                     4076   →   40,76 €
                                          ──────
  Total TTC = 20380 + 0 + 4076              24456   →  244,56 €
```

Contrôle : 203,80 × 1,20 = 244,56. Exact.

Et un petit panier, sous le franco :

```text
  1 × Mug Hello World          1290 × 1 =   1290
  1 × Repose-poignets Velours  1990 × 1 =   1990
                                          ──────
  Sous-total HT                              3280   →   32,80 €
  3280 < 10000    →  port facturé             490   →    4,90 €
  Base taxable = 3280 + 490                  3770
  TVA = 3770 × 20 / 100                       754   →    7,54 €
                                          ──────
  Total TTC = 3770 + 754                     4524   →   45,24 €
```

Contrôle : 37,70 × 1,20 = 45,24. Exact.

**État exécutable.** Trois fichiers de plus, aucun changement visible. Le panier vivant arrive maintenant.

---

## 60.15 — Le panier en `ChangeNotifier`

### Pourquoi un état global

Le panier est lu par la barre d'application du catalogue, par la fiche produit, par l'écran de panier et par l'écran de commande. Le faire remonter de `setState` en `setState` à travers quatre écrans est impraticable. C'est exactement le cas d'usage de `ChangeNotifier` + `provider` (chapitre 52).

```text
     ChangeNotifierProvider<Panier>   ← au-dessus de MaterialApp
                   │
      ┌────────────┼─────────────┬──────────────┐
      ▼            ▼             ▼              ▼
  Catalogue    Fiche produit   Panier       Commande
  (badge)      (ajouter)       (quantités)  (total)
```

### L'interface de persistance

Nous la déclarons dès maintenant pour ne pas avoir à réécrire le panier au 60.19.

**`lib/donnees/depot_panier.dart`**

```dart
/// Persistance du panier, sous la forme minimale « identifiant → quantité ».
abstract class DepotPanier {
  Future<Map<String, int>> lire();
  Future<void> enregistrer(Map<String, int> quantites);
}
```

### Le panier

**`lib/etat/panier.dart`**

```dart
import 'package:flutter/foundation.dart';

import '../donnees/depot_panier.dart';
import '../logique/calcul_tarifs.dart';
import '../modeles/ligne_panier.dart';
import '../modeles/produit.dart';
import '../modeles/tarifs.dart';

/// Le panier de l'utilisateur.
///
/// Les lignes sont rangées dans une `Map` indexée par identifiant de
/// produit : ajouter deux fois le même article doit incrémenter une
/// ligne, pas en créer une seconde. Les `Map` de Dart conservent
/// l'ordre d'insertion, donc l'affichage reste stable.
class Panier extends ChangeNotifier {
  Panier({DepotPanier? depot}) : _depot = depot;

  final DepotPanier? _depot;
  final Map<String, LignePanier> _lignes = <String, LignePanier>{};

  /// Vue en lecture seule : personne d'autre que le panier ne modifie
  /// le panier.
  List<LignePanier> get lignes => List<LignePanier>.unmodifiable(_lignes.values);

  bool get estVide => _lignes.isEmpty;

  /// Nombre de lignes distinctes. 1 manette + 2 figurines → 2.
  int get nombreReferences => _lignes.length;

  /// Nombre total d'articles. 1 manette + 2 figurines → 3.
  int get nombreArticles => _lignes.values
      .fold<int>(0, (int somme, LignePanier l) => somme + l.quantite);

  /// Détail chiffré, recalculé à la demande à partir des lignes.
  /// Aucune donnée dérivée n'est stockée : impossible qu'un total
  /// reste désynchronisé de son contenu.
  Tarifs get tarifs => calculerTarifs(lignes);

  int quantiteDe(Produit produit) => _lignes[produit.id]?.quantite ?? 0;

  bool contient(Produit produit) => _lignes.containsKey(produit.id);

  /// Fixe la quantité d'un produit.
  ///
  /// C'est la **seule** méthode qui modifie réellement le panier ;
  /// toutes les autres passent par elle. Un unique point de
  /// modification signifie un unique endroit où plafonner au stock,
  /// où supprimer une ligne vide et où notifier.
  void definirQuantite(Produit produit, int quantite) {
    final int voulue = quantite.clamp(0, produit.stock);

    if (voulue == 0) {
      if (_lignes.remove(produit.id) == null) {
        return; // rien à faire, donc rien à notifier
      }
    } else {
      final LignePanier? existante = _lignes[produit.id];
      if (existante != null && existante.quantite == voulue) {
        return;
      }
      _lignes[produit.id] = LignePanier(produit: produit, quantite: voulue);
    }

    _notifierEtEnregistrer();
  }

  void ajouter(Produit produit, {int quantite = 1}) {
    definirQuantite(produit, quantiteDe(produit) + quantite);
  }

  void retirer(Produit produit) => definirQuantite(produit, 0);

  void vider() {
    if (_lignes.isEmpty) {
      return;
    }
    _lignes.clear();
    _notifierEtEnregistrer();
  }

  /// Reconstitue le panier enregistré, en le confrontant au catalogue
  /// courant : un produit disparu est oublié, une quantité supérieure
  /// au stock actuel est ramenée au stock.
  Future<void> restaurer(List<Produit> catalogue) async {
    final DepotPanier? depot = _depot;
    if (depot == null) {
      return;
    }

    final Map<String, int> quantites = await depot.lire();
    if (quantites.isEmpty) {
      return;
    }

    final Map<String, Produit> parId = <String, Produit>{
      for (final Produit p in catalogue) p.id: p,
    };

    _lignes.clear();
    quantites.forEach((String id, int quantite) {
      final Produit? produit = parId[id];
      if (produit == null) {
        return;
      }
      final int voulue = quantite.clamp(0, produit.stock);
      if (voulue > 0) {
        _lignes[id] = LignePanier(produit: produit, quantite: voulue);
      }
    });

    notifyListeners();
  }

  /// Notifie l'interface **d'abord**, écrit sur le disque ensuite.
  /// L'utilisateur ne doit jamais attendre le disque pour voir son
  /// panier se mettre à jour (mise à jour optimiste).
  void _notifierEtEnregistrer() {
    notifyListeners();
    _depot?.enregistrer(<String, int>{
      for (final LignePanier l in _lignes.values) l.produit.id: l.quantite,
    });
  }
}
```

> **`clamp` fait tout le travail.** `quantite.clamp(0, produit.stock)` refuse d'un coup les valeurs négatives et les quantités supérieures au stock. Sur un produit en rupture (`stock == 0`), le résultat vaut toujours 0 : il devient impossible d'ajouter au panier un article indisponible, même par un chemin détourné.

### Le branchement

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'ecrans/ecran_catalogue.dart';
import 'etat/panier.dart';

void main() {
  runApp(const ApplicationBoutique());
}

class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    // Le provider est placé AU-DESSUS de MaterialApp : tous les
    // écrans poussés par le Navigator en héritent.
    return ChangeNotifierProvider<Panier>(
      create: (BuildContext context) => Panier(),
      child: MaterialApp(
        title: 'Pixel Boutique',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4F46E5)),
        ),
        home: const EcranCatalogue(),
      ),
    );
  }
}
```

Et dans `ecran_produit.dart`, remplacez le `onPressed` provisoire :

```dart
              onPressed: () {
                // `read` et non `watch` : nous sommes dans un rappel,
                // pas dans un `build`.
                context.read<Panier>().ajouter(produit, quantite: _quantite);
                ScaffoldMessenger.of(context)
                  ..hideCurrentSnackBar()
                  ..showSnackBar(
                    SnackBar(
                      content: Text('$_quantite × ${produit.nom} ajouté'),
                      duration: const Duration(seconds: 2),
                    ),
                  );
              },
```

avec l'import `import 'package:provider/provider.dart';`.

**État exécutable.** Le bouton alimente réellement le panier. Rien ne le montre encore à l'écran : c'est l'objet du pas suivant.

---

## 60.16 — L'icône de panier avec badge

Material 3 fournit `Badge.count`, qui gère l'ellipse, le centrage du nombre et le passage à « 999+ ».

**`lib/widgets/bouton_panier.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../ecrans/ecran_panier.dart';
import '../etat/panier.dart';

/// Icône de panier surmontée du nombre d'articles.
class BoutonPanier extends StatelessWidget {
  const BoutonPanier({super.key});

  @override
  Widget build(BuildContext context) {
    // `select` ne reconstruit ce widget que si le nombre d'articles
    // change. Avec `watch<Panier>()`, il se reconstruirait aussi pour
    // un simple changement de quantité qui ne modifie pas le total —
    // ou pire, à chaque `notifyListeners()` du panier.
    final int articles = context.select<Panier, int>(
      (Panier panier) => panier.nombreArticles,
    );

    return IconButton(
      tooltip: 'Voir le panier',
      icon: Badge.count(
        count: articles,
        isLabelVisible: articles > 0,
        child: const Icon(Icons.shopping_cart_outlined),
      ),
      onPressed: () {
        Navigator.of(context).push(
          MaterialPageRoute<void>(
            builder: (BuildContext context) => const EcranPanier(),
          ),
        );
      },
    );
  }
}
```

Ajoutez ce bouton dans les deux `AppBar` déjà écrites :

```dart
      appBar: AppBar(
        title: const Text('Pixel Boutique'),
        actions: const <Widget>[BoutonPanier()],
      ),
```

```dart
      appBar: AppBar(
        title: Text(produit.nom),
        actions: const <Widget>[BoutonPanier()],
      ),
```

**État exécutable.** Le badge s'incrémente à chaque ajout, sur les deux écrans à la fois. Le bouton ne mènera nulle part tant que `ecran_panier.dart` n'existe pas : écrivons-le.

---

## 60.17 — L'écran du panier, et son état vide

### L'état vide

Un panier vide n'est pas un bogue, c'est le cas le plus fréquent au premier lancement. Il mérite un vrai écran : ce que l'on regarde, pourquoi c'est vide, quoi faire ensuite.

**`lib/widgets/panier_vide.dart`**

```dart
import 'package:flutter/material.dart';

class PanierVide extends StatelessWidget {
  const PanierVide({super.key, required this.onRetourCatalogue});

  final VoidCallback onRetourCatalogue;

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
              Icons.remove_shopping_cart_outlined,
              size: 72,
              color: theme.colorScheme.outline,
            ),
            const SizedBox(height: 20),
            Text('Votre panier est vide', style: theme.textTheme.titleLarge),
            const SizedBox(height: 8),
            Text(
              'Parcourez le catalogue et ajoutez votre premier article.',
              textAlign: TextAlign.center,
              style: theme.textTheme.bodyMedium?.copyWith(
                color: theme.colorScheme.onSurfaceVariant,
              ),
            ),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: onRetourCatalogue,
              icon: const Icon(Icons.storefront),
              label: const Text('Voir le catalogue'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### La tuile d'une ligne

**`lib/widgets/tuile_ligne_panier.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/panier.dart';
import '../modeles/ligne_panier.dart';
import '../utilitaires/formats.dart';
import 'selecteur_quantite.dart';
import 'vignette_produit.dart';

class TuileLignePanier extends StatelessWidget {
  const TuileLignePanier({super.key, required this.ligne});

  final LignePanier ligne;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Panier panier = context.read<Panier>();

    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          // Pas de Hero ici : le produit est déjà « héros » sur le
          // catalogue, et deux Hero de même tag sur un même écran
          // lèvent une exception.
          SizedBox(
            width: 72,
            height: 72,
            child: VignetteProduit(
              produit: ligne.produit,
              tailleIcone: 32,
              rayon: 12,
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text(
                  ligne.produit.nom,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                  style: theme.textTheme.titleSmall,
                ),
                Text(
                  '${prixDepuisCentimes(ligne.produit.prixCentimes)} l\'unité',
                  style: theme.textTheme.bodySmall?.copyWith(
                    color: theme.colorScheme.onSurfaceVariant,
                  ),
                ),
                const SizedBox(height: 4),
                Row(
                  children: <Widget>[
                    SelecteurQuantite(
                      valeur: ligne.quantite,
                      maximum: ligne.produit.stock,
                      onChange: (int n) =>
                          panier.definirQuantite(ligne.produit, n),
                    ),
                    const Spacer(),
                    Text(
                      prixDepuisCentimes(ligne.totalCentimes),
                      style: theme.textTheme.titleMedium?.copyWith(
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
          IconButton(
            tooltip: 'Retirer du panier',
            icon: const Icon(Icons.delete_outline),
            onPressed: () => panier.retirer(ligne.produit),
          ),
        ],
      ),
    );
  }
}
```

### Le récapitulatif chiffré

**`lib/widgets/recapitulatif_tarifs.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/calcul_tarifs.dart';
import '../modeles/tarifs.dart';
import '../utilitaires/formats.dart';

class RecapitulatifTarifs extends StatelessWidget {
  const RecapitulatifTarifs({super.key, required this.tarifs});

  final Tarifs tarifs;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 8, 16, 16),
      child: Column(
        children: <Widget>[
          _Ligne(
            libelle: 'Sous-total (HT)',
            valeur: prixDepuisCentimes(tarifs.sousTotalCentimes),
          ),
          _Ligne(
            libelle: 'Livraison',
            valeur: prixDepuisCentimes(tarifs.portCentimes),
            note: tarifs.portOffert ? 'offerte' : null,
          ),
          _Ligne(
            libelle: 'TVA ($tauxTvaPourCent %)',
            valeur: prixDepuisCentimes(tarifs.tvaCentimes),
          ),
          const Divider(height: 20),
          _Ligne(
            libelle: 'Total TTC',
            valeur: prixDepuisCentimes(tarifs.totalCentimes),
            gras: true,
          ),
          if (!tarifs.portOffert && tarifs.sousTotalCentimes > 0)
            Padding(
              padding: const EdgeInsets.only(top: 8),
              child: Row(
                children: <Widget>[
                  const Icon(Icons.local_shipping_outlined, size: 16),
                  const SizedBox(width: 6),
                  Expanded(
                    child: Text(
                      'Plus que '
                      '${prixDepuisCentimes(seuilFrancoCentimes - tarifs.sousTotalCentimes)}'
                      ' pour la livraison offerte.',
                      style: theme.textTheme.bodySmall,
                    ),
                  ),
                ],
              ),
            ),
        ],
      ),
    );
  }
}

class _Ligne extends StatelessWidget {
  const _Ligne({
    required this.libelle,
    required this.valeur,
    this.note,
    this.gras = false,
  });

  final String libelle;
  final String valeur;
  final String? note;
  final bool gras;

  @override
  Widget build(BuildContext context) {
    final TextStyle? style = gras
        ? Theme.of(context).textTheme.titleLarge
        : Theme.of(context).textTheme.bodyLarge;

    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 2),
      child: Row(
        children: <Widget>[
          Text(libelle, style: style),
          if (note != null) ...<Widget>[
            const SizedBox(width: 8),
            Text(
              note!,
              style: TextStyle(color: Colors.green.shade700, fontSize: 12),
            ),
          ],
          const Spacer(),
          Text(valeur, style: style),
        ],
      ),
    );
  }
}
```

### L'écran

**`lib/ecrans/ecran_panier.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/panier.dart';
import '../modeles/ligne_panier.dart';
import '../utilitaires/formats.dart';
import '../widgets/panier_vide.dart';
import '../widgets/recapitulatif_tarifs.dart';
import '../widgets/tuile_ligne_panier.dart';
import 'ecran_commande.dart';

class EcranPanier extends StatelessWidget {
  const EcranPanier({super.key});

  @override
  Widget build(BuildContext context) {
    // Ici `watch` est correct : nous sommes dans un `build` et tout
    // l'écran dépend du panier.
    final Panier panier = context.watch<Panier>();
    final List<LignePanier> lignes = panier.lignes;

    return Scaffold(
      appBar: AppBar(
        title: Text(
          panier.estVide
              ? 'Mon panier'
              : 'Mon panier (${panier.nombreArticles} '
                  '${panier.nombreArticles > 1 ? "articles" : "article"})',
        ),
        actions: <Widget>[
          if (!panier.estVide)
            TextButton(
              onPressed: () => _confirmerVidage(context, panier),
              child: const Text('Vider'),
            ),
        ],
      ),
      body: panier.estVide
          ? PanierVide(onRetourCatalogue: () => Navigator.of(context).pop())
          : Column(
              children: <Widget>[
                Expanded(
                  child: ListView.separated(
                    itemCount: lignes.length,
                    separatorBuilder: (_, __) => const Divider(height: 1),
                    itemBuilder: (BuildContext context, int index) =>
                        TuileLignePanier(ligne: lignes[index]),
                  ),
                ),
                const Divider(height: 1),
                RecapitulatifTarifs(tarifs: panier.tarifs),
                Padding(
                  padding: const EdgeInsets.fromLTRB(16, 0, 16, 16),
                  child: FilledButton(
                    onPressed: () {
                      Navigator.of(context).push(
                        MaterialPageRoute<void>(
                          builder: (BuildContext context) =>
                              const EcranCommande(),
                        ),
                      );
                    },
                    style: FilledButton.styleFrom(
                      minimumSize: const Size.fromHeight(52),
                    ),
                    child: Text(
                      'Commander — '
                      '${prixDepuisCentimes(panier.tarifs.totalCentimes)}',
                    ),
                  ),
                ),
              ],
            ),
    );
  }

  Future<void> _confirmerVidage(BuildContext context, Panier panier) async {
    final bool? confirme = await showDialog<bool>(
      context: context,
      builder: (BuildContext context) => AlertDialog(
        title: const Text('Vider le panier ?'),
        content: const Text('Tous les articles seront retirés.'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: const Text('Annuler'),
          ),
          FilledButton(
            onPressed: () => Navigator.of(context).pop(true),
            child: const Text('Vider'),
          ),
        ],
      ),
    );

    if (confirme ?? false) {
      panier.vider();
    }
  }
}
```

> **Le piège de la `Column` et de la `ListView`.** La liste est enveloppée dans un `Expanded`. Sans lui, Flutter lève `Vertical viewport was given unbounded height` : une `Column` offre à ses enfants une hauteur infinie, et une `ListView` ne sait pas quoi en faire (chapitre 46).

**État exécutable.** Ajoutez la manette, deux figurines et le casque. L'écran de panier affiche exactement :

```text
  Sous-total (HT)                       203,80 €
  Livraison                 offerte       0,00 €
  TVA (20 %)                             40,76 €
  ─────────────────────────────────────────────
  Total TTC                             244,56 €
```

Retirez tout : l'écran vide apparaît. Le bouton « Commander » ne mène nulle part tant que `ecran_commande.dart` n'existe pas ; créez-le vide provisoirement si vous voulez compiler tout de suite.

---

## 60.18 — La recherche, le filtre par catégorie et le tri

### Les critères

**`lib/logique/criteres.dart`**

```dart
/// Ordres d'affichage proposés à l'utilisateur.
enum TriProduits {
  pertinence('Nos suggestions'),
  prixCroissant('Prix croissant'),
  prixDecroissant('Prix décroissant'),
  noteDecroissante('Meilleures notes'),
  nomAlphabetique('Nom (A-Z)');

  const TriProduits(this.libelle);

  final String libelle;
}
```

### Les fonctions pures

Filtrer et trier ne réclament ni écran, ni état, ni Flutter. Ce sont des fonctions qui prennent une liste et en rendent une autre — donc des fonctions testables en trois lignes (60.22).

**`lib/logique/recherche.dart`**

```dart
import '../modeles/categorie.dart';
import '../modeles/produit.dart';
import 'criteres.dart';

/// Ne conserve que les produits satisfaisant tous les critères fournis.
///
/// La recherche porte sur le nom, la description et le libellé de la
/// catégorie ; elle est insensible à la casse. Elle reste sensible aux
/// accents : « ecran » ne trouve pas « Écran ». Le défi 7 corrige ce
/// point.
List<Produit> filtrerProduits(
  List<Produit> source, {
  String recherche = '',
  Categorie? categorie,
  bool enStockSeulement = false,
}) {
  final String terme = recherche.trim().toLowerCase();

  return source.where((Produit produit) {
    if (categorie != null && produit.categorie != categorie) {
      return false;
    }
    if (enStockSeulement && !produit.enStock) {
      return false;
    }
    if (terme.isEmpty) {
      return true;
    }
    return produit.nom.toLowerCase().contains(terme) ||
        produit.description.toLowerCase().contains(terme) ||
        produit.categorie.libelle.toLowerCase().contains(terme);
  }).toList(growable: false);
}

/// Renvoie une **nouvelle** liste triée.
///
/// `sort` modifie la liste sur place ; trier directement le catalogue
/// le réordonnerait définitivement, et le tri « Nos suggestions » ne
/// pourrait plus jamais retrouver l'ordre d'origine. D'où la copie.
///
/// `sort` n'est de plus pas stable en Dart : à note égale, l'ordre
/// serait imprévisible. Chaque comparateur départage donc les ex æquo
/// par le nom, ce qui rend l'affichage déterministe et les tests
/// écrivables.
List<Produit> trierProduits(List<Produit> source, TriProduits tri) {
  final List<Produit> copie = List<Produit>.of(source);

  final int Function(Produit a, Produit b)? comparateur = switch (tri) {
    TriProduits.pertinence => null,
    TriProduits.prixCroissant => (Produit a, Produit b) =>
        a.prixCentimes.compareTo(b.prixCentimes),
    TriProduits.prixDecroissant => (Produit a, Produit b) =>
        b.prixCentimes.compareTo(a.prixCentimes),
    TriProduits.noteDecroissante => (Produit a, Produit b) {
        final int parNote = b.note.compareTo(a.note);
        return parNote != 0 ? parNote : a.nom.compareTo(b.nom);
      },
    TriProduits.nomAlphabetique => (Produit a, Produit b) =>
        a.nom.toLowerCase().compareTo(b.nom.toLowerCase()),
  };

  if (comparateur != null) {
    copie.sort(comparateur);
  }
  return copie;
}
```

Sur notre catalogue, `TriProduits.noteDecroissante` donne exactement :

```text
 1. 4,9  Figurine du Boss Final
 2. 4,8  Clavier mécanique Runeboard
 3. 4,8  Livre Dart et Flutter en pratique
 4. 4,7  Écran 27 pouces Pixel 165 Hz
 5. 4,6  Livre Boucle de jeu
 6. 4,6  Manette Nébula Pro
 7. 4,5  Casque Aurora 7.1
 8. 4,5  Tapis de souris XXL Galaxie
 9. 4,4  Siège Ergo Boss
10. 4,3  Souris Photon 8K
11. 4,2  Coussin lombaire Checkpoint
12. 4,2  Mug Hello World
13. 4,1  Micro Studio Écho
14. 4,0  Repose-poignets Velours
15. 3,9  Écran portable 15 pouces Nomade
```

Les paires à 4,8 puis 4,6 puis 4,5 puis 4,2 sont départagées par le nom : c'est le comparateur secondaire à l'œuvre.

> **Piège du tri alphabétique.** `String.compareTo` compare des unités de code UTF-16. « Écran » (É = U+00C9) se place donc **après** « Zèbre ». Pour un vrai classement français il faudrait normaliser les accents ; c'est le sujet du défi 7.

### L'état du catalogue

**`lib/etat/etat_catalogue.dart`**

```dart
import 'package:flutter/foundation.dart';

import '../logique/criteres.dart';
import '../logique/recherche.dart';
import '../modeles/categorie.dart';
import '../modeles/produit.dart';

/// Catalogue chargé, plus les critères d'affichage courants.
class EtatCatalogue extends ChangeNotifier {
  EtatCatalogue(List<Produit> produits)
      : tousLesProduits = List<Produit>.unmodifiable(produits);

  final List<Produit> tousLesProduits;

  String _recherche = '';
  Categorie? _categorie;
  TriProduits _tri = TriProduits.pertinence;
  bool _enStockSeulement = false;

  String get recherche => _recherche;
  Categorie? get categorie => _categorie;
  TriProduits get tri => _tri;
  bool get enStockSeulement => _enStockSeulement;

  /// Liste effectivement affichée. Elle est recalculée à chaque
  /// lecture : sur quelques centaines de produits le coût est
  /// négligeable, et aucune valeur dérivée ne peut se désynchroniser.
  List<Produit> get produitsAffiches => trierProduits(
        filtrerProduits(
          tousLesProduits,
          recherche: _recherche,
          categorie: _categorie,
          enStockSeulement: _enStockSeulement,
        ),
        _tri,
      );

  bool get filtreActif =>
      _recherche.isNotEmpty || _categorie != null || _enStockSeulement;

  void definirRecherche(String valeur) {
    if (valeur == _recherche) {
      return;
    }
    _recherche = valeur;
    notifyListeners();
  }

  void definirCategorie(Categorie? valeur) {
    if (valeur == _categorie) {
      return;
    }
    _categorie = valeur;
    notifyListeners();
  }

  void definirTri(TriProduits valeur) {
    if (valeur == _tri) {
      return;
    }
    _tri = valeur;
    notifyListeners();
  }

  void basculerEnStockSeulement() {
    _enStockSeulement = !_enStockSeulement;
    notifyListeners();
  }

  void reinitialiser() {
    _recherche = '';
    _categorie = null;
    _enStockSeulement = false;
    notifyListeners();
  }
}
```

> **Le test `if (valeur == _x) return;`** évite de notifier pour rien. Sans lui, chaque frappe identique reconstruirait toute la grille. C'est un réflexe à prendre dans tous vos `ChangeNotifier`.

### La barre de filtres

**`lib/widgets/barre_filtres.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_catalogue.dart';
import '../logique/criteres.dart';
import '../modeles/categorie.dart';

class BarreFiltres extends StatefulWidget {
  const BarreFiltres({super.key});

  @override
  State<BarreFiltres> createState() => _BarreFiltresState();
}

class _BarreFiltresState extends State<BarreFiltres> {
  // Le contrôleur est un champ du State, jamais une variable locale
  // de `build` : sinon le curseur saute à chaque frappe (chapitre 49).
  final TextEditingController _controleur = TextEditingController();

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final EtatCatalogue etat = context.watch<EtatCatalogue>();

    return Column(
      children: <Widget>[
        Padding(
          padding: const EdgeInsets.fromLTRB(12, 8, 12, 4),
          child: TextField(
            controller: _controleur,
            textInputAction: TextInputAction.search,
            decoration: InputDecoration(
              hintText: 'Rechercher un produit...',
              prefixIcon: const Icon(Icons.search),
              border: const OutlineInputBorder(),
              isDense: true,
              suffixIcon: etat.recherche.isEmpty
                  ? null
                  : IconButton(
                      icon: const Icon(Icons.clear),
                      onPressed: () {
                        _controleur.clear();
                        context.read<EtatCatalogue>().definirRecherche('');
                      },
                    ),
            ),
            onChanged: (String valeur) =>
                context.read<EtatCatalogue>().definirRecherche(valeur),
          ),
        ),
        SizedBox(
          height: 44,
          child: ListView(
            scrollDirection: Axis.horizontal,
            padding: const EdgeInsets.symmetric(horizontal: 12),
            children: <Widget>[
              _Puce(
                libelle: 'Tout',
                actif: etat.categorie == null,
                onTap: () =>
                    context.read<EtatCatalogue>().definirCategorie(null),
              ),
              for (final Categorie c in Categorie.values)
                _Puce(
                  libelle: c.libelle,
                  actif: etat.categorie == c,
                  onTap: () =>
                      context.read<EtatCatalogue>().definirCategorie(c),
                ),
            ],
          ),
        ),
        Padding(
          padding: const EdgeInsets.fromLTRB(16, 4, 8, 4),
          child: Row(
            children: <Widget>[
              Text('${etat.produitsAffiches.length} produits'),
              const Spacer(),
              PopupMenuButton<TriProduits>(
                initialValue: etat.tri,
                tooltip: 'Trier',
                onSelected: (TriProduits t) =>
                    context.read<EtatCatalogue>().definirTri(t),
                itemBuilder: (BuildContext context) => <PopupMenuEntry<TriProduits>>[
                  for (final TriProduits t in TriProduits.values)
                    PopupMenuItem<TriProduits>(
                      value: t,
                      child: Text(t.libelle),
                    ),
                ],
                child: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: <Widget>[
                    const Icon(Icons.sort, size: 18),
                    const SizedBox(width: 6),
                    Text(etat.tri.libelle),
                  ],
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

class _Puce extends StatelessWidget {
  const _Puce({
    required this.libelle,
    required this.actif,
    required this.onTap,
  });

  final String libelle;
  final bool actif;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: FilterChip(
        label: Text(libelle),
        selected: actif,
        onSelected: (_) => onTap(),
      ),
    );
  }
}
```

### L'écran de catalogue réécrit

Le chargement quitte l'écran pour remonter dans `main` : la grille n'a plus d'état propre et devient un simple `StatelessWidget`.

**`lib/ecrans/ecran_catalogue.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_catalogue.dart';
import '../modeles/produit.dart';
import '../widgets/barre_filtres.dart';
import '../widgets/bouton_panier.dart';
import '../widgets/grille_produits.dart';
import 'ecran_produit.dart';

class EcranCatalogue extends StatelessWidget {
  const EcranCatalogue({super.key});

  @override
  Widget build(BuildContext context) {
    final EtatCatalogue etat = context.watch<EtatCatalogue>();
    final List<Produit> produits = etat.produitsAffiches;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Pixel Boutique'),
        actions: const <Widget>[BoutonPanier()],
      ),
      body: Column(
        children: <Widget>[
          const BarreFiltres(),
          const Divider(height: 1),
          Expanded(
            child: produits.isEmpty
                ? _AucunResultat(
                    filtreActif: etat.filtreActif,
                    onReinitialiser: etat.reinitialiser,
                  )
                : GrilleProduits(
                    produits: produits,
                    onProduitTouche: (Produit produit) {
                      Navigator.of(context).push(
                        MaterialPageRoute<void>(
                          builder: (BuildContext context) =>
                              EcranProduit(produit: produit),
                        ),
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }
}

/// Deux messages distincts : « aucun résultat » n'a pas le même sens
/// que « catalogue vide ».
class _AucunResultat extends StatelessWidget {
  const _AucunResultat({
    required this.filtreActif,
    required this.onReinitialiser,
  });

  final bool filtreActif;
  final VoidCallback onReinitialiser;

  @override
  Widget build(BuildContext context) {
    if (!filtreActif) {
      return const Center(
        child: Padding(
          padding: EdgeInsets.all(32),
          child: Text(
            'Le catalogue n\'a pas pu être chargé. '
            'Relancez l\'application.',
            textAlign: TextAlign.center,
          ),
        ),
      );
    }

    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          const Icon(Icons.search_off, size: 56),
          const SizedBox(height: 12),
          const Text('Aucun produit ne correspond'),
          const SizedBox(height: 16),
          OutlinedButton(
            onPressed: onReinitialiser,
            child: const Text('Réinitialiser les filtres'),
          ),
        ],
      ),
    );
  }
}
```

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'donnees/depot_assets.dart';
import 'donnees/depot_produits.dart';
import 'ecrans/ecran_catalogue.dart';
import 'etat/etat_catalogue.dart';
import 'etat/panier.dart';
import 'modeles/produit.dart';

Future<void> main() async {
  // Obligatoire avant tout appel à un service de Flutter (ici
  // rootBundle) en amont de runApp.
  WidgetsFlutterBinding.ensureInitialized();

  const DepotProduits depot = DepotAssets();
  List<Produit> produits;
  try {
    produits = await depot.chargerProduits();
  } on ErreurCatalogue catch (erreur) {
    debugPrint('$erreur');
    produits = <Produit>[];
  }

  runApp(ApplicationBoutique(produits: produits));
}

class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({super.key, required this.produits});

  final List<Produit> produits;

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: <SingleChildWidget>[
        ChangeNotifierProvider<EtatCatalogue>(
          create: (_) => EtatCatalogue(produits),
        ),
        ChangeNotifierProvider<Panier>(create: (_) => Panier()),
      ],
      child: MaterialApp(
        title: 'Pixel Boutique',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF4F46E5)),
        ),
        darkTheme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(
            seedColor: const Color(0xFF4F46E5),
            brightness: Brightness.dark,
          ),
        ),
        home: const EcranCatalogue(),
      ),
    );
  }
}
```

> **`SingleChildWidget`** est le type attendu par `MultiProvider` ; il vient de `package:provider/provider.dart`, qui le réexporte. Aucun import supplémentaire n'est nécessaire.

> **Charger avant `runApp` : le compromis.** L'application ne s'affiche qu'une fois le catalogue lu, ce qui prolonge l'écran de démarrage natif de quelques millisecondes. En échange, aucun écran n'a plus à gérer l'état « en cours de chargement », et le panier pourra être restauré en confrontant les identifiants à un catalogue déjà connu (60.19). Pour un fichier local le compromis est excellent ; pour un appel réseau lent, il faudrait garder le `FutureBuilder` du 60.11.

**État exécutable.** Tapez « clav » : seul le Clavier mécanique Runeboard subsiste. Tapez « casque » : un seul résultat, le Casque Aurora 7.1. Tapez « micro » : deux résultats, car le mot figure dans le nom du Micro Studio Écho et dans la description du Casque Aurora 7.1 — la recherche porte bien sur les deux champs. Touchez la puce « Audio » : deux produits. Choisissez « Prix croissant » : le Mug Hello World passe en tête à 12,90 €.

---

## 60.19 — Persister le panier

### L'implémentation

**`lib/donnees/depot_panier_prefs.dart`**

```dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

import 'depot_panier.dart';

/// Panier enregistré dans les préférences, sous forme d'une chaîne JSON.
///
/// On n'enregistre que des couples identifiant/quantité : quelques
/// dizaines d'octets. `shared_preferences` est parfaitement
/// dimensionné pour cela (chapitre 54).
class DepotPanierPrefs implements DepotPanier {
  const DepotPanierPrefs({this.cle = 'panier_v1'});

  final String cle;

  @override
  Future<Map<String, int>> lire() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    final String? texte = prefs.getString(cle);
    if (texte == null || texte.isEmpty) {
      return <String, int>{};
    }

    try {
      final Map<String, dynamic> brut =
          jsonDecode(texte) as Map<String, dynamic>;
      return <String, int>{
        for (final MapEntry<String, dynamic> e in brut.entries)
          if (e.value is num) e.key: (e.value as num).toInt(),
      };
    } catch (_) {
      // Donnée corrompue : on repart d'un panier vide plutôt que de
      // faire échouer le démarrage.
      await prefs.remove(cle);
      return <String, int>{};
    }
  }

  @override
  Future<void> enregistrer(Map<String, int> quantites) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    if (quantites.isEmpty) {
      await prefs.remove(cle);
      return;
    }
    await prefs.setString(cle, jsonEncode(quantites));
  }
}
```

> **Pourquoi `panier_v1` ?** Le jour où le format changera, la clé deviendra `panier_v2` et les anciennes données seront simplement ignorées, sans code de migration ni plantage. Versionner une clé de stockage coûte quatre caractères et évite des heures d'ennuis.

### Le branchement

Trois lignes à changer dans `main`.

**`lib/main.dart`** (extrait)

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  const DepotProduits depot = DepotAssets();
  List<Produit> produits;
  try {
    produits = await depot.chargerProduits();
  } on ErreurCatalogue catch (erreur) {
    debugPrint('$erreur');
    produits = <Produit>[];
  }

  // Le panier connaît son dépôt : il s'enregistrera tout seul après
  // chaque modification.
  final Panier panier = Panier(depot: const DepotPanierPrefs());
  await panier.restaurer(produits);

  runApp(ApplicationBoutique(produits: produits, panier: panier));
}
```

et dans le widget racine :

```dart
class ApplicationBoutique extends StatelessWidget {
  const ApplicationBoutique({
    super.key,
    required this.produits,
    required this.panier,
  });

  final List<Produit> produits;
  final Panier panier;

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: <SingleChildWidget>[
        ChangeNotifierProvider<EtatCatalogue>(
          create: (_) => EtatCatalogue(produits),
        ),
        // `.value` et non `create` : l'objet existe déjà, il a été
        // construit et restauré avant runApp.
        ChangeNotifierProvider<Panier>.value(value: panier),
      ],
      child: MaterialApp(/* inchangé */),
    );
  }
}
```

sans oublier `import 'donnees/depot_panier_prefs.dart';`.

**État exécutable.** Ajoutez trois articles, tuez complètement l'application, relancez-la : le badge affiche de nouveau 4 et le panier est intact. Modifiez le stock de la figurine à 1 dans `catalogue.json`, relancez : la quantité 2 est automatiquement ramenée à 1 par le `clamp` de `restaurer`.

---

## 60.20 — La commande et le formulaire d'adresse

### Le modèle

**`lib/modeles/commande.dart`**

```dart
import 'ligne_panier.dart';
import 'tarifs.dart';

/// Adresse de livraison saisie par le client.
class Adresse {
  const Adresse({
    required this.nomComplet,
    required this.rue,
    required this.codePostal,
    required this.ville,
    required this.courriel,
  });

  final String nomComplet;
  final String rue;
  final String codePostal;
  final String ville;
  final String courriel;

  /// Prénom seul, pour le message de remerciement.
  String get prenom => nomComplet.trim().split(' ').first;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom_complet': nomComplet,
        'rue': rue,
        'code_postal': codePostal,
        'ville': ville,
        'courriel': courriel,
      };
}

/// Une commande validée. Elle fige le contenu du panier au moment de
/// l'achat : le panier, lui, sera vidé juste après.
class Commande {
  const Commande({
    required this.reference,
    required this.date,
    required this.adresse,
    required this.lignes,
    required this.tarifs,
  });

  final String reference;
  final DateTime date;
  final Adresse adresse;
  final List<LignePanier> lignes;
  final Tarifs tarifs;

  int get nombreReferences => lignes.length;

  int get nombreArticles =>
      lignes.fold<int>(0, (int s, LignePanier l) => s + l.quantite);

  /// Livraison annoncée à trois jours.
  DateTime get livraisonEstimee => date.add(const Duration(days: 3));

  /// Référence lisible du type « PB-20260815-4821 ».
  static String genererReference(DateTime date) {
    final String jour = '${date.year}'
        '${date.month.toString().padLeft(2, '0')}'
        '${date.day.toString().padLeft(2, '0')}';
    final String suffixe =
        (date.millisecondsSinceEpoch % 10000).toString().padLeft(4, '0');
    return 'PB-$jour-$suffixe';
  }
}
```

### L'écran

Rappel du chapitre 49 : un `Form` avec une `GlobalKey<FormState>`, un `validator` par champ qui renvoie `null` quand tout va bien, et `_cle.currentState!.validate()` qui déclenche l'ensemble.

**`lib/ecrans/ecran_commande.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:provider/provider.dart';

import '../etat/panier.dart';
import '../modeles/commande.dart';
import '../utilitaires/formats.dart';
import 'ecran_confirmation.dart';

class EcranCommande extends StatefulWidget {
  const EcranCommande({super.key});

  @override
  State<EcranCommande> createState() => _EcranCommandeState();
}

class _EcranCommandeState extends State<EcranCommande> {
  final GlobalKey<FormState> _cleFormulaire = GlobalKey<FormState>();

  final TextEditingController _nom = TextEditingController();
  final TextEditingController _rue = TextEditingController();
  final TextEditingController _codePostal = TextEditingController();
  final TextEditingController _ville = TextEditingController();
  final TextEditingController _courriel = TextEditingController();

  /// Compilée une seule fois : construire une `RegExp` dans un
  /// `validator` la reconstruirait à chaque frappe.
  static final RegExp _formeCourriel = RegExp(r'^[^@\s]+@[^@\s]+\.[^@\s]{2,}$');
  static final RegExp _formeCodePostal = RegExp(r'^\d{5}$');

  @override
  void dispose() {
    _nom.dispose();
    _rue.dispose();
    _codePostal.dispose();
    _ville.dispose();
    _courriel.dispose();
    super.dispose();
  }

  String? _obligatoire(String? valeur, String champ, {int minimum = 2}) {
    final String texte = (valeur ?? '').trim();
    if (texte.isEmpty) {
      return '$champ est obligatoire.';
    }
    if (texte.length < minimum) {
      return '$champ doit comporter au moins $minimum caractères.';
    }
    return null;
  }

  void _valider() {
    // `validate()` déclenche TOUS les validateurs et affiche toutes
    // les erreurs d'un coup. Il renvoie false dès qu'un seul échoue.
    if (!_cleFormulaire.currentState!.validate()) {
      // Le clavier masque souvent le champ fautif : on le referme.
      FocusScope.of(context).unfocus();
      return;
    }

    final Panier panier = context.read<Panier>();
    final DateTime maintenant = DateTime.now();

    final Commande commande = Commande(
      reference: Commande.genererReference(maintenant),
      date: maintenant,
      adresse: Adresse(
        nomComplet: _nom.text.trim(),
        rue: _rue.text.trim(),
        codePostal: _codePostal.text.trim(),
        ville: _ville.text.trim(),
        courriel: _courriel.text.trim(),
      ),
      lignes: panier.lignes,
      tarifs: panier.tarifs,
    );

    // La commande est figée : le panier peut être vidé.
    panier.vider();

    // `pushAndRemoveUntil` empile la confirmation et supprime tout ce
    // qui se trouve au-dessus du premier écran. La pile passe de
    // « catalogue → panier → commande » à « catalogue → confirmation ».
    // Un retour arrière depuis la confirmation ne peut donc plus
    // ramener sur un panier désormais vide.
    Navigator.of(context).pushAndRemoveUntil(
      MaterialPageRoute<void>(
        builder: (BuildContext context) => EcranConfirmation(commande: commande),
      ),
      (Route<dynamic> route) => route.isFirst,
    );
  }

  @override
  Widget build(BuildContext context) {
    final Panier panier = context.watch<Panier>();

    if (panier.estVide) {
      // Cas limite réel : l'utilisateur a retiré le dernier article
      // depuis un autre écran pendant qu'il remplissait le formulaire.
      return Scaffold(
        appBar: AppBar(title: const Text('Commande')),
        body: const Center(child: Text('Votre panier est vide.')),
      );
    }

    return Scaffold(
      appBar: AppBar(title: const Text('Commande')),
      body: Form(
        key: _cleFormulaire,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: <Widget>[
            Card(
              child: ListTile(
                leading: const Icon(Icons.shopping_bag_outlined),
                title: Text('${panier.nombreArticles} articles'),
                subtitle: Text(
                  '${prixDepuisCentimes(panier.tarifs.totalCentimes)} TTC',
                ),
              ),
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _nom,
              textCapitalization: TextCapitalization.words,
              decoration: const InputDecoration(
                labelText: 'Nom complet *',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) => _obligatoire(v, 'Le nom', minimum: 3),
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _rue,
              decoration: const InputDecoration(
                labelText: 'Adresse *',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) => _obligatoire(v, 'L\'adresse', minimum: 5),
            ),
            const SizedBox(height: 12),
            Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                SizedBox(
                  width: 130,
                  child: TextFormField(
                    controller: _codePostal,
                    keyboardType: TextInputType.number,
                    // Le clavier numérique ne suffit pas : sur bureau
                    // et sur Web on peut taper des lettres.
                    inputFormatters: <TextInputFormatter>[
                      FilteringTextInputFormatter.digitsOnly,
                      LengthLimitingTextInputFormatter(5),
                    ],
                    decoration: const InputDecoration(
                      labelText: 'Code postal *',
                      border: OutlineInputBorder(),
                    ),
                    validator: (String? v) {
                      if (!_formeCodePostal.hasMatch((v ?? '').trim())) {
                        return 'Le code postal doit comporter 5 chiffres.';
                      }
                      return null;
                    },
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: TextFormField(
                    controller: _ville,
                    textCapitalization: TextCapitalization.words,
                    decoration: const InputDecoration(
                      labelText: 'Ville *',
                      border: OutlineInputBorder(),
                    ),
                    validator: (String? v) => _obligatoire(v, 'La ville'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _courriel,
              keyboardType: TextInputType.emailAddress,
              autocorrect: false,
              decoration: const InputDecoration(
                labelText: 'Courriel *',
                helperText: 'Pour recevoir la confirmation de commande.',
                border: OutlineInputBorder(),
              ),
              validator: (String? v) {
                final String texte = (v ?? '').trim();
                if (texte.isEmpty) {
                  return 'Le courriel est obligatoire.';
                }
                if (!_formeCourriel.hasMatch(texte)) {
                  return 'Ce courriel ne semble pas valide.';
                }
                return null;
              },
            ),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: _valider,
              style: FilledButton.styleFrom(
                minimumSize: const Size.fromHeight(52),
              ),
              child: const Text('Valider la commande'),
            ),
            const SizedBox(height: 8),
            Text(
              'Boutique de démonstration : aucun paiement n\'est réellement '
              'effectué.',
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

> **Trois réflexes de formulaire.**
>
> 1. Un contrôleur déclaré dans le `State`, libéré dans `dispose`. Déclaré dans `build`, il est recréé à chaque frappe et le curseur saute au début.
> 2. Un `validator` renvoie `null` quand la valeur est bonne, et un **message** sinon. Renvoyer `''` affiche une erreur vide, ce qui déplace la mise en page sans rien expliquer.
> 3. `inputFormatters` complète `keyboardType`, il ne le remplace pas : `keyboardType` suggère un clavier, `inputFormatters` impose le contenu.

**État exécutable.** Saisissez `3500` comme code postal : le message « Le code postal doit comporter 5 chiffres. » apparaît sous le champ et le bouton ne navigue pas. Corrigez en `35000` : l'application passe à l'écran de confirmation, et le badge du panier retombe à zéro.

---

## 60.21 — La confirmation

Il reste à initialiser les données de locale pour `DateFormat`. Ajoutez ces deux lignes dans `main`, avant `runApp` :

```dart
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';
  // ... suite inchangée
}
```

Et, pour que le reste des widgets Material parle aussi français, complétez `MaterialApp` :

```dart
        localizationsDelegates: GlobalMaterialLocalizations.delegates,
        supportedLocales: const <Locale>[Locale('fr', 'FR')],
```

avec `import 'package:flutter_localizations/flutter_localizations.dart';`.

**`lib/ecrans/ecran_confirmation.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/commande.dart';
import '../modeles/ligne_panier.dart';
import '../utilitaires/formats.dart';

class EcranConfirmation extends StatelessWidget {
  const EcranConfirmation({super.key, required this.commande});

  final Commande commande;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Adresse adresse = commande.adresse;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Commande confirmée'),
        // Il n'y a plus rien derrière cet écran que le catalogue :
        // la flèche de retour automatique n'aurait pas de sens ici.
        automaticallyImplyLeading: false,
      ),
      body: ListView(
        padding: const EdgeInsets.all(24),
        children: <Widget>[
          Icon(
            Icons.check_circle_outline,
            size: 88,
            color: Colors.green.shade600,
          ),
          const SizedBox(height: 16),
          Text(
            'Merci ${adresse.prenom} !',
            textAlign: TextAlign.center,
            style: theme.textTheme.headlineSmall,
          ),
          const SizedBox(height: 8),
          Text(
            'Commande ${commande.reference}',
            textAlign: TextAlign.center,
            style: theme.textTheme.titleMedium?.copyWith(
              color: theme.colorScheme.onSurfaceVariant,
            ),
          ),
          const SizedBox(height: 24),
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  Text(
                    '${commande.nombreReferences} références · '
                    '${commande.nombreArticles} articles',
                    style: theme.textTheme.titleSmall,
                  ),
                  const Divider(height: 20),
                  for (final LignePanier ligne in commande.lignes)
                    Padding(
                      padding: const EdgeInsets.symmetric(vertical: 2),
                      child: Row(
                        children: <Widget>[
                          Expanded(
                            child: Text(
                              '${ligne.quantite} × ${ligne.produit.nom}',
                              maxLines: 1,
                              overflow: TextOverflow.ellipsis,
                            ),
                          ),
                          Text(prixDepuisCentimes(ligne.totalCentimes)),
                        ],
                      ),
                    ),
                  const Divider(height: 20),
                  Row(
                    children: <Widget>[
                      const Text('Total réglé'),
                      const Spacer(),
                      Text(
                        prixDepuisCentimes(commande.tarifs.totalCentimes),
                        style: theme.textTheme.titleMedium?.copyWith(
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Row(
                    children: <Widget>[
                      const Icon(Icons.local_shipping_outlined, size: 18),
                      const SizedBox(width: 8),
                      Expanded(
                        child: Text(
                          'Livraison estimée le '
                          '${dateLongue(commande.livraisonEstimee)}',
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Text(adresse.nomComplet),
                  Text(adresse.rue),
                  Text('${adresse.codePostal} ${adresse.ville}'),
                  Text(adresse.courriel, style: theme.textTheme.bodySmall),
                ],
              ),
            ),
          ),
          const SizedBox(height: 24),
          FilledButton.icon(
            onPressed: () {
              // Revient au premier écran de la pile, c'est-à-dire au
              // catalogue.
              Navigator.of(context).popUntil((Route<dynamic> r) => r.isFirst);
            },
            icon: const Icon(Icons.storefront),
            label: const Text('Retour à la boutique'),
            style: FilledButton.styleFrom(
              minimumSize: const Size.fromHeight(52),
            ),
          ),
        ],
      ),
    );
  }
}
```

**État exécutable.** Une commande passée le samedi 15 août 2026 affiche :

```text
              Merci Camille !
         Commande PB-20260815-4821

  3 références · 4 articles
  ─────────────────────────────────────
  1 × Manette Nébula Pro        64,90 €
  2 × Figurine du Boss Final    49,00 €
  1 × Casque Aurora 7.1         89,90 €
  ─────────────────────────────────────
  Total réglé                  244,56 €
  Livraison estimée le mardi 18 août 2026
```

Le bouton retour ramène au catalogue, panier vidé, badge éteint. La boucle complète est fermée.

> **`LocaleDataException: Locale data has not been initialized`** est l'erreur qui vous attend si vous oubliez `initializeDateFormatting`. Elle ne se déclenche qu'au moment du premier formatage de date, c'est-à-dire ici : au tout dernier écran, après une commande complète. Raison de plus pour ne pas l'oublier.

---

## 60.22 — Les tests

Toute la logique de ce projet vit dans `modeles/`, `logique/` et `etat/`, sans dépendance à un écran. Elle se teste donc sans émulateur, en quelques dixièmes de seconde.

### Le calcul des tarifs

**`test/calcul_tarifs_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:pixel_boutique/logique/calcul_tarifs.dart';
import 'package:pixel_boutique/modeles/categorie.dart';
import 'package:pixel_boutique/modeles/ligne_panier.dart';
import 'package:pixel_boutique/modeles/produit.dart';
import 'package:pixel_boutique/modeles/tarifs.dart';

Produit _produit(String id, int prixCentimes, {int stock = 50}) {
  return Produit(
    id: id,
    nom: 'Article $id',
    description: '',
    prixCentimes: prixCentimes,
    categorie: Categorie.goodies,
    note: 4.0,
    stock: stock,
  );
}

void main() {
  group('calculerTarifs', () {
    test('un panier vide ne coûte rien, port compris', () {
      final Tarifs t = calculerTarifs(<LignePanier>[]);

      expect(t.sousTotalCentimes, 0);
      expect(t.portCentimes, 0);
      expect(t.tvaCentimes, 0);
      expect(t.totalCentimes, 0);
      expect(t.portOffert, isFalse);
    });

    test('panier de référence : 203,80 HT, port offert, total 244,56', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('p01', 6490), quantite: 1),
        LignePanier(produit: _produit('p10', 2450), quantite: 2),
        LignePanier(produit: _produit('p04', 8990), quantite: 1),
      ]);

      expect(t.sousTotalCentimes, 20380); // 6490 + 4900 + 8990
      expect(t.portCentimes, 0); // 20380 >= 10000
      expect(t.tvaCentimes, 4076); // 20380 × 20 / 100
      expect(t.totalCentimes, 24456); // 20380 + 0 + 4076
      expect(t.portOffert, isTrue);
    });

    test('petit panier : le port est facturé et taxé', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('p11', 1290), quantite: 1),
        LignePanier(produit: _produit('p09', 1990), quantite: 1),
      ]);

      expect(t.sousTotalCentimes, 3280);
      expect(t.portCentimes, 490);
      expect(t.tvaCentimes, 754); // (3280 + 490) × 20 / 100
      expect(t.totalCentimes, 4524);
      expect(t.portOffert, isFalse);
    });

    test('la quantité multiplie bien la ligne', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('p11', 1290), quantite: 3),
      ]);

      expect(t.sousTotalCentimes, 3870); // 1290 × 3
      expect(t.portCentimes, 490);
      expect(t.tvaCentimes, 872); // 4360 × 20 / 100
      expect(t.totalCentimes, 5232);
    });

    test('le franco s\'applique exactement au seuil', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('x', seuilFrancoCentimes), quantite: 1),
      ]);

      expect(t.sousTotalCentimes, 10000);
      expect(t.portCentimes, 0);
      expect(t.tvaCentimes, 2000);
      expect(t.totalCentimes, 12000);
    });

    test('cinq centimes sous le seuil, le port revient', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('x', 9995), quantite: 1),
      ]);

      expect(t.sousTotalCentimes, 9995);
      expect(t.portCentimes, 490);
      expect(t.tvaCentimes, 2097); // 10485 × 20 / 100
      expect(t.totalCentimes, 12582);
    });

    test('mille articles restent exacts au centime', () {
      final Tarifs t = calculerTarifs(<LignePanier>[
        LignePanier(produit: _produit('x', 6490, stock: 1000), quantite: 1000),
      ]);

      expect(t.sousTotalCentimes, 6490000);
      expect(t.tvaCentimes, 1298000);
      expect(t.totalCentimes, 7788000); // 77 880,00 €
    });
  });
}
```

Le dernier test est le plus instructif : avec des `double`, 1000 × 64,90 aurait donné `64899.99999999999` ou `64900.00000000001` selon l'ordre des opérations. En `int`, c'est `6490000`, sans discussion possible.

### La logique du panier

**`test/panier_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:pixel_boutique/donnees/depot_memoire.dart';
import 'package:pixel_boutique/etat/panier.dart';
import 'package:pixel_boutique/modeles/produit.dart';

/// Retrouve un produit du jeu de démonstration par son identifiant.
Produit p(String id) =>
    produitsDemonstration.firstWhere((Produit x) => x.id == id);

void main() {
  late Panier panier;

  setUp(() {
    // Un panier neuf avant CHAQUE test : sans cela, l'ordre
    // d'exécution influencerait les résultats.
    panier = Panier();
  });

  group('Panier', () {
    test('un panier neuf est vide', () {
      expect(panier.estVide, isTrue);
      expect(panier.lignes, isEmpty);
      expect(panier.nombreArticles, 0);
      expect(panier.tarifs.totalCentimes, 0);
    });

    test('ajouter deux fois le même produit incrémente une seule ligne', () {
      panier.ajouter(p('p01'));
      panier.ajouter(p('p01'));

      expect(panier.nombreReferences, 1);
      expect(panier.nombreArticles, 2);
      expect(panier.quantiteDe(p('p01')), 2);
    });

    test('nombreReferences et nombreArticles ne mesurent pas la même chose',
        () {
      panier.ajouter(p('p01'));
      panier.ajouter(p('p10'), quantite: 2);
      panier.ajouter(p('p04'));

      expect(panier.nombreReferences, 3);
      expect(panier.nombreArticles, 4);
      expect(panier.tarifs.totalCentimes, 24456);
    });

    test('mettre une quantité à zéro supprime la ligne', () {
      panier.ajouter(p('p01'), quantite: 3);
      panier.definirQuantite(p('p01'), 0);

      expect(panier.estVide, isTrue);
      expect(panier.contient(p('p01')), isFalse);
    });

    test('une quantité négative est ramenée à zéro', () {
      panier.ajouter(p('p01'));
      panier.definirQuantite(p('p01'), -5);

      expect(panier.estVide, isTrue);
    });

    test('la quantité est plafonnée au stock', () {
      // Le micro p05 n'a que 3 exemplaires.
      panier.definirQuantite(p('p05'), 10);

      expect(panier.quantiteDe(p('p05')), 3);
      expect(panier.tarifs.sousTotalCentimes, 33750); // 11250 × 3
      expect(panier.tarifs.tvaCentimes, 6750);
      expect(panier.tarifs.totalCentimes, 40500);
    });

    test('un produit en rupture ne peut pas entrer dans le panier', () {
      // La souris p03 est à zéro.
      panier.ajouter(p('p03'), quantite: 2);

      expect(panier.estVide, isTrue);
    });

    test('retirer enlève la ligne, vider enlève tout', () {
      panier.ajouter(p('p01'));
      panier.ajouter(p('p11'));

      panier.retirer(p('p01'));
      expect(panier.nombreReferences, 1);

      panier.vider();
      expect(panier.estVide, isTrue);
    });

    test('les auditeurs sont prévenus une fois par changement réel', () {
      int appels = 0;
      panier.addListener(() => appels++);

      panier.ajouter(p('p01')); // 1
      panier.ajouter(p('p01')); // 2
      panier.definirQuantite(p('p01'), 2); // déjà 2 : aucun changement
      panier.retirer(p('p01')); // 3
      panier.retirer(p('p01')); // ligne absente : aucun changement

      expect(appels, 3);
    });

    test('lignes est une vue non modifiable', () {
      panier.ajouter(p('p01'));

      expect(
        () => panier.lignes.clear(),
        throwsUnsupportedError,
      );
    });
  });
}
```

### Le filtrage et le tri

**`test/recherche_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:pixel_boutique/donnees/depot_memoire.dart';
import 'package:pixel_boutique/logique/criteres.dart';
import 'package:pixel_boutique/logique/recherche.dart';
import 'package:pixel_boutique/modeles/categorie.dart';
import 'package:pixel_boutique/modeles/produit.dart';

void main() {
  const List<Produit> source = produitsDemonstration;

  group('filtrerProduits', () {
    test('sans critère, tout passe', () {
      expect(filtrerProduits(source).length, source.length);
    });

    test('par catégorie', () {
      final List<Produit> audio =
          filtrerProduits(source, categorie: Categorie.audio);

      expect(audio.length, 2);
      expect(audio.every((Produit p) => p.categorie == Categorie.audio), isTrue);
    });

    test('en stock seulement', () {
      final List<Produit> dispo =
          filtrerProduits(source, enStockSeulement: true);

      expect(dispo.length, source.length - 1); // la souris p03 est en rupture
      expect(dispo.any((Produit p) => p.id == 'p03'), isFalse);
    });

    test('recherche insensible à la casse dans le nom', () {
      expect(filtrerProduits(source, recherche: 'CASQUE').single.id, 'p04');
    });

    test('recherche dans la description', () {
      expect(filtrerProduits(source, recherche: 'mousse').single.id, 'p09');
    });

    test('les critères se combinent', () {
      final List<Produit> r = filtrerProduits(
        source,
        recherche: 'micro',
        categorie: Categorie.audio,
      );

      expect(r.single.id, 'p05');
    });

    test('une recherche sans résultat rend une liste vide, pas null', () {
      expect(filtrerProduits(source, recherche: 'trottinette'), isEmpty);
    });
  });

  group('trierProduits', () {
    test('la liste source n\'est jamais modifiée', () {
      final List<Produit> copie = List<Produit>.of(source);
      trierProduits(source, TriProduits.prixCroissant);

      expect(source, orderedEquals(copie));
    });

    test('prix croissant', () {
      final List<Produit> r = trierProduits(source, TriProduits.prixCroissant);

      expect(r.first.id, 'p11'); // Mug, 1290
      expect(r.last.id, 'p05'); // Micro, 11250
    });

    test('prix décroissant', () {
      final List<Produit> r =
          trierProduits(source, TriProduits.prixDecroissant);

      expect(r.first.id, 'p05');
      expect(r.last.id, 'p11');
    });

    test('meilleures notes, ex æquo départagés par le nom', () {
      final List<Produit> r =
          trierProduits(source, TriProduits.noteDecroissante);

      expect(r.first.id, 'p10'); // Figurine, 4,9
      expect(r.last.id, 'p09'); // Repose-poignets, 4,0
    });

    test('pertinence conserve l\'ordre du catalogue', () {
      final List<Produit> r = trierProduits(source, TriProduits.pertinence);

      expect(r, orderedEquals(source));
    });
  });
}
```

Lancez l'ensemble :

```text
flutter test
```

**Résultat :**

```text
00:03 +33: All tests passed!
```

> **Ce qui n'est pas testé ici.** Les écrans. Les tests de widget (`testWidgets`, `WidgetTester`, `pumpWidget`) existent et sont utiles, mais ils sont dix fois plus lents et bien plus fragiles. La bonne stratégie est celle de ce projet : sortir la logique des widgets, la tester exhaustivement, et ne réserver les tests de widget qu'aux quelques parcours critiques.

---

## 60.23 — Récapitulatif de l'arborescence finale

```text
pixel_boutique/
├── pubspec.yaml                          60.1
├── assets/donnees/catalogue.json         60.5
├── lib/
│   ├── main.dart                         60.1, 60.15, 60.18, 60.19, 60.21
│   ├── modeles/
│   │   ├── categorie.dart                60.2
│   │   ├── produit.dart                  60.3, 60.4
│   │   ├── ligne_panier.dart             60.14
│   │   ├── tarifs.dart                   60.14
│   │   └── commande.dart                 60.20
│   ├── logique/
│   │   ├── criteres.dart                 60.18
│   │   ├── recherche.dart                60.18
│   │   └── calcul_tarifs.dart            60.14
│   ├── donnees/
│   │   ├── depot_produits.dart           60.6
│   │   ├── depot_assets.dart             60.6
│   │   ├── depot_memoire.dart            60.6
│   │   ├── depot_panier.dart             60.15
│   │   └── depot_panier_prefs.dart       60.19
│   ├── etat/
│   │   ├── panier.dart                   60.15
│   │   └── etat_catalogue.dart           60.18
│   ├── ecrans/
│   │   ├── ecran_catalogue.dart          60.11, 60.18
│   │   ├── ecran_produit.dart            60.12, 60.13
│   │   ├── ecran_panier.dart             60.17
│   │   ├── ecran_commande.dart           60.20
│   │   └── ecran_confirmation.dart       60.21
│   ├── widgets/
│   │   ├── vignette_produit.dart         60.8, 60.13
│   │   ├── carte_produit.dart            60.9, 60.13
│   │   ├── etoiles_note.dart             60.9
│   │   ├── grille_produits.dart          60.10
│   │   ├── selecteur_quantite.dart       60.12
│   │   ├── bouton_panier.dart            60.16
│   │   ├── tuile_ligne_panier.dart       60.17
│   │   ├── recapitulatif_tarifs.dart     60.17
│   │   ├── panier_vide.dart              60.17
│   │   └── barre_filtres.dart            60.18
│   └── utilitaires/
│       ├── formats.dart                  60.7
│       └── apparence.dart                60.8
└── test/
    ├── produit_test.dart                 60.4
    ├── calcul_tarifs_test.dart           60.22
    ├── panier_test.dart                  60.22
    └── recherche_test.dart               60.22
```

Le sens des dépendances, qui ne doit jamais s'inverser :

```text
   ecrans/   widgets/   utilitaires/    <- connaissent Flutter
       │         │
       └────┬────┘
            ▼
          etat/                         <- ne connaît que foundation
            │
       ┌────┴────┐
       ▼         ▼
   logique/  donnees/                   <- Dart pur (sauf depot_assets)
       │         │
       └────┬────┘
            ▼
        modeles/                        <- ne connaît rien
```

---

## 60.24 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: assets/donnees/catalogue.json` | Asset non déclaré, ou application non relancée. | Compléter `assets:` dans `pubspec.yaml` puis relancer (pas un simple *hot reload*). |
| `FormatException: Unexpected character` | Virgule finale ou guillemet manquant dans le JSON. | Valider le fichier avant de le charger. |
| `type 'Null' is not a subtype of type 'String'` dans `fromJson` | Une clé absente lue en type non nullable. | `as String?` puis `??`. |
| `type 'int' is not a subtype of type 'double'` | `"note": 4` lu avec `as double?`. | `as num?` puis `.toDouble()`. |
| Le total affiché finit par `.00000000001` | Prix stockés en `double`. | Compter en centimes avec des `int`. |
| Le total est faux d'un centime | TVA calculée ligne par ligne puis additionnée. | Calculer la TVA une seule fois, sur la base totale. |
| Barre rayée jaune et noire sur une carte | Contenu plus haut que la cellule imposée par le `gridDelegate`. | `Expanded` autour de la vignette ; ne jamais fixer une hauteur en dur. |
| Cartes écrasées ou démesurément hautes | `childAspectRatio` inversé. | C'est `largeur / hauteur` : `0.68` donne une carte haute. |
| Une seule colonne sur tablette | `SliverGridDelegateWithFixedCrossAxisCount` utilisé. | `SliverGridDelegateWithMaxCrossAxisExtent`. |
| `Vertical viewport was given unbounded height` | `ListView` ou `GridView` directement dans une `Column`. | Envelopper dans un `Expanded` (chapitre 46). |
| `BoxConstraints forces an infinite height` sur la vignette | `VignetteProduit` sans contrainte de parent. | La placer sous un `Expanded`, un `AspectRatio` ou un `SizedBox`. |
| `There are multiple heroes that share the same tag` | Deux `Hero` de même `tag` sur un même écran. | Un seul `Hero` par produit et par écran ; pas de `Hero` dans le panier. |
| L'animation `Hero` ne se produit pas | Les deux écrans ne sont pas dans le même `Navigator`, ou les `tag` diffèrent. | Utiliser `tagHeroProduit` des deux côtés ; éviter les feuilles modales. |
| Une image réseau affiche un carré gris et une exception | `errorBuilder` absent. | Toujours fournir `errorBuilder` **et** `loadingBuilder`. |
| Chargement en boucle infinie | `Future` créé dans `build`. | Le créer dans `initState`. |
| `ProviderNotFoundException` | Provider placé sous `MaterialApp`, ou `context` du widget qui le crée. | Le placer au-dessus de `MaterialApp`, lire depuis un enfant. |
| L'écran ne se rafraîchit plus | `context.read` utilisé dans un `build`. | `watch` dans un `build`, `read` dans un rappel. |
| Le badge se reconstruit sans arrêt | `context.watch<Panier>()` pour n'en lire qu'un entier. | `context.select<Panier, int>((p) => p.nombreArticles)`. |
| `setState() called after dispose()` | `setState` après un `await` sur un widget détruit. | `if (!mounted) return;` avant. |
| Le curseur saute au début à chaque frappe | `TextEditingController` créé dans `build`. | Le déclarer dans le `State`, le libérer dans `dispose`. |
| Le formulaire se valide malgré un champ vide | `validator` renvoyant `''` au lieu d'un message, ou `validate()` non appelé. | Renvoyer `null` si valide, un message sinon. |
| `LocaleDataException: Locale data has not been initialized` | `initializeDateFormatting` oublié. | L'appeler dans `main` avant `runApp`. |
| Le prix s'affiche `64.9` au lieu de `64,90 €` | `toString()` au lieu du formateur. | `prixDepuisCentimes`. |
| Un test compare `'64,90 €'` et échoue | `intl` utilise une espace insécable (U+00A0). | Comparer sur le nombre, ou utiliser `' '`. |
| Le panier restauré contient un produit supprimé | Rehydratation sans confronter au catalogue. | Ignorer les identifiants inconnus dans `restaurer`. |
| Le retour arrière ramène sur un panier vide | `push` au lieu de `pushAndRemoveUntil`. | Nettoyer la pile après la commande. |
| `Unsupported operation: Cannot clear an unmodifiable list` | Modification de `panier.lignes`. | Passer par les méthodes du `Panier`. |
| Les tests échouent avec « types incompatibles » | Import relatif de `lib/` depuis `test/`. | Importer en `package:pixel_boutique/...`. |

---

## 60.25 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Prix en centimes | Un montant est un `int`. On divise par 100 une seule fois, pour l'affichage. |
| TVA | Calculée une fois sur la base totale, jamais ligne par ligne. |
| `enum` enrichi | Il porte son libellé ; on persiste `.name`, jamais `.index`. |
| `fromJson` défensif | `as Type?` + `??` : une donnée abîmée ne doit pas empêcher le démarrage. |
| `as num?` | Un JSON contenant `4` produit un `int`, pas un `double`. |
| Catalogue en asset | La donnée n'est pas du code ; le remplacer par une API ne touche qu'un dépôt. |
| Dépôt (`repository`) | Une interface abstraite, plusieurs implémentations, une seule ligne à changer dans `main`. |
| `GridView.builder` | Seules les cellules visibles sont construites. |
| `SliverGridDelegateWithMaxCrossAxisExtent` | On fixe une largeur maximale, Flutter en déduit le nombre de colonnes. |
| `childAspectRatio` | C'est `largeur / hauteur`. En dessous de 1, la cellule est plus haute que large. |
| Carte sans débordement | `Expanded` autour du visuel : le texte prend sa place, le visuel absorbe le reste. |
| Image sans fichier | Un dégradé et une icône suffisent, pèsent zéro octet et fonctionnent hors ligne. |
| `Image.network` | Toujours accompagnée de `loadingBuilder` et d'`errorBuilder`. |
| `intl` | `NumberFormat` fonctionne immédiatement ; `DateFormat` réclame `initializeDateFormatting`. |
| `Hero` | Deux `tag` identiques, un par écran, et Flutter écrit l'animation à votre place. |
| `ChangeNotifier` | Il détient l'état, expose des lectures seules, notifie après chaque changement **réel**. |
| Un seul point de modification | Toutes les méthodes du panier passent par `definirQuantite` : une seule règle à écrire, une seule à tester. |
| État dérivé | `tarifs` est un getter recalculé, jamais un champ stocké : il ne peut pas se désynchroniser. |
| `watch` / `read` / `select` | `watch` dans un `build`, `read` dans un rappel, `select` pour ne suivre qu'une valeur. |
| Mise à jour optimiste | Notifier d'abord, écrire sur le disque ensuite. |
| Persistance minimale | On enregistre des identifiants et des quantités, pas des prix : le catalogue fait foi. |
| Clé versionnée | `panier_v1` : changer de format ne casse rien. |
| Fonctions pures | Filtrer et trier sans effet de bord, donc testables en trois lignes. |
| `sort` | Il modifie sur place et n'est pas stable : copier la liste, et départager les ex æquo. |
| `Form` et `validator` | `null` signifie valide ; `validate()` déclenche tout le formulaire d'un coup. |
| `inputFormatters` | Le clavier se suggère, le contenu s'impose. |
| `pushAndRemoveUntil` | Après une action irréversible, on nettoie la pile de navigation. |
| État vide | Trois questions : que regarde-t-on, pourquoi c'est vide, que faire ensuite. |
| Sens des dépendances | `ecrans` → `etat` → `logique`/`donnees` → `modeles`. Jamais l'inverse. |

---

## 60.26 — Extensions : dix défis

Chaque défi est réalisable avec ce que vous savez déjà. L'indication donne la direction, pas la solution.

### Défi 1 — Les favoris (facile)

Un cœur sur chaque carte ajoute le produit à une liste de favoris, consultable depuis un onglet.

*Indication :* un second `ChangeNotifier` `Favoris` contenant un `Set<String>` d'identifiants, persisté avec `setStringList`. La carte lit `context.select<Favoris, bool>((f) => f.contient(produit.id))`.

### Défi 2 — Le filtre « en stock seulement » (facile)

`EtatCatalogue.basculerEnStockSeulement` existe déjà mais n'est branché sur rien.

*Indication :* un `FilterChip` supplémentaire dans `BarreFiltres`, ou un `SwitchListTile` dans une feuille de filtres ouverte par `showModalBottomSheet`.

### Défi 3 — La fourchette de prix (facile)

Un `RangeSlider` limite l'affichage à une plage de prix.

*Indication :* ajoutez `prixMin` et `prixMax` en paramètres nommés de `filtrerProduits`, puis deux champs à `EtatCatalogue`. Les bornes se déduisent du catalogue avec `reduce` (chapitre 14).

### Défi 4 — Le code promotionnel (moyen)

Le code `PIXEL10` retire 10 % du sous-total avant calcul du port et de la TVA.

*Indication :* ajoutez un paramètre `remisePourCent` à `calculerTarifs` et un champ `remiseCentimes` à `Tarifs`. Attention : la remise doit être arrondie une seule fois, et le franco de port s'apprécie **après** remise. Écrivez le test avant le code — c'est le défi où une erreur d'un centime est la plus facile à commettre.

### Défi 5 — L'historique des commandes (moyen)

Chaque commande validée est enregistrée ; un écran les liste de la plus récente à la plus ancienne.

*Indication :* `Commande.toJson`, une `List<String>` dans `shared_preferences`, un `DepotCommandes`. La confirmation devient alors accessible depuis l'historique, ce qui la rend réutilisable telle quelle.

### Défi 6 — Le catalogue depuis une API (moyen)

Remplacez `DepotAssets` par un `DepotHttp` interrogeant un service REST.

*Indication :* le paquet `http` et le chapitre 53. Seule la classe change : `Produit.fromJson` et tout le reste de l'application sont déjà écrits. C'est exactement le bénéfice promis au 60.6 — vérifiez-le en chronométrant.

### Défi 7 — La recherche insensible aux accents (moyen)

« ecran » doit trouver « Écran 27 pouces », et le tri alphabétique doit placer « Écran » entre « Coussin » et « Figurine ».

*Indication :* une fonction `String sansAccents(String s)` fondée sur une `Map<String, String>` et `replaceAll`. Appliquez-la des deux côtés de la comparaison dans `filtrerProduits`, et comme clé de tri dans `nomAlphabetique`. Écrivez les tests d'abord.

### Défi 8 — Les avis clients (moyen)

Chaque produit affiche trois avis, et l'utilisateur peut en déposer un, qui recalcule la note moyenne.

*Indication :* une classe `Avis` avec auteur, note et texte ; un `Map<String, List<Avis>>` persisté. La moyenne est un `fold` divisé par la longueur — attention à la division par zéro sur un produit sans avis.

### Défi 9 — Les quantités animées (difficile)

Le changement de quantité fait glisser le chiffre vers le haut ou vers le bas, et la suppression d'une ligne la fait disparaître en fondu.

*Indication :* `AnimatedSwitcher` avec une `SlideTransition` pour le chiffre, et `AnimatedList` (ou `AnimatedSize`) pour la liste. Le point délicat est la `Key` : sans clé stable par produit, `AnimatedSwitcher` ne détecte aucun changement.

### Défi 10 — Le panier partagé entre appareils (difficile)

Le panier suit l'utilisateur d'un appareil à l'autre.

*Indication :* il faut un identifiant de session, un service distant et une stratégie de fusion en cas de conflit — que faire si l'appareil A a 2 figurines et l'appareil B en a 3 ? La réponse n'est pas technique mais métier, et c'est ce qui rend ce défi le plus long des dix.

---

## Et maintenant ?

Vous venez d'écrire une application de commerce complète : un catalogue chargé depuis un fichier de données, une grille qui s'adapte de la montre au moniteur, une fiche animée, un panier centralisé et persistant, une arithmétique monétaire exacte au centime, un formulaire validé et un parcours d'achat qui se termine proprement. C'est, à quelques écrans près, la structure de toutes les boutiques que vous avez sur votre téléphone.

Une chose y manque pourtant, et c'est la plus caractéristique des applications modernes : les données ne venaient pas du réseau. Le fichier `catalogue.json` était toujours là, toujours identique, toujours instantané. Le monde réel est moins aimable : un serveur peut être lent, indisponible, renvoyer une erreur 404 ou un JSON inattendu, et l'utilisateur peut se trouver dans un tunnel.

Le chapitre 61 s'attaque à ce problème avec une application météo. Vous y retrouverez les modèles, la sérialisation et l'architecture en couches de ce projet, mais avec trois nouveautés déterminantes : de vrais appels HTTP, la gestion complète des états de chargement et d'erreur, et un cache local permettant d'afficher quelque chose même hors ligne.

Rendez-vous au chapitre 61 : [61-PARTIE-1C—PROJET-7-APPLICATION-MÉTÉO-API.md](./61-PARTIE-1C—PROJET-7-APPLICATION-MÉTÉO-API.md)
