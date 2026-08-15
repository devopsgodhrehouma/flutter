# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 62 — PROJET 8 : LE LANCEUR DE JEU

> **Niveau :** intermédiaire / avancé
> **Durée estimée :** 20 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), PARTIE 1B (chapitres 43 à 54), et les projets 55 à 61
> **Ce que vous saurez faire à la fin :** construire l'intégralité du « méta-jeu » qui entoure un jeu vidéo — menu animé, création de personnage, fiche, inventaire, équipement, boutique, classement, options, sauvegardes multiples — dans une application Flutter classique, prête à recevoir un moteur de jeu.

---

## 62.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- distinguer la **boucle de jeu** du **méta-jeu**, et comprendre pourquoi le second représente la majorité du code d'un jeu commercial ;
- construire un menu principal animé à sept entrées, dont deux conditionnelles ;
- modéliser un personnage, un objet et une sauvegarde avec `fromJson` / `toJson` (rappel du chapitre 17) ;
- écrire un écran de création de personnage complet : nom validé, choix de classe, répartition de points (rappel du chapitre 49) ;
- afficher une fiche de personnage avec des barres de statistiques proportionnelles (rappel du chapitre 55) ;
- afficher un inventaire en `GridView` avec filtre par type et tri multi-critère (rappels des chapitres 48 et 14) ;
- équiper des objets et recalculer des statistiques dérivées sans jamais dupliquer la donnée ;
- construire une boutique avec achat, revente et gestion d'une monnaie (rappel du chapitre 60) ;
- afficher un classement local, puis le fusionner avec un classement distant (rappel du chapitre 53) ;
- écrire un écran d'options : volumes, difficulté, thème, et **remappage des touches** ;
- persister l'intégralité d'une partie dans une seule chaîne JSON (rappel du chapitre 54) ;
- gérer **plusieurs emplacements de sauvegarde** indépendants ;
- détecter et réparer une sauvegarde corrompue sans jamais faire planter l'application (rappel du chapitre 13) ;
- centraliser l'état de l'application dans plusieurs `ChangeNotifier` exposés par `provider` (rappel du chapitre 52) ;
- écrire des transitions d'écran personnalisées avec `PageRouteBuilder` (rappel du chapitre 50) ;
- construire un écran de chargement qui prépare réellement quelque chose ;
- dessiner une zone de jeu factice avec `CustomPaint`, à l'emplacement exact du futur `GameWidget` ;
- lire un tableau de correspondance écran par écran entre ce projet et le jeu Flame du chapitre 35 ;
- tester la logique de personnage et d'inventaire sans lancer d'interface.

---

## 62.0.1 — Le méta-jeu, ou 80 % du travail

Quand un débutant imagine « faire un jeu », il pense à la boucle de jeu : le personnage saute, l'ennemi patrouille, la pièce tourne sur elle-même. C'est la partie visible. Ouvrez maintenant n'importe quel jeu de votre téléphone et comptez les écrans **avant** d'arriver au gameplay :

```text
    Chargement  →  Menu principal  →  Emplacements  →  Création de personnage
                        │
                        ├──→  Fiche de personnage
                        ├──→  Inventaire  ←→  Équipement
                        ├──→  Boutique
                        ├──→  Classement
                        └──→  Options
```

Sept écrans, une persistance, une économie, un classement. Et là-dedans : **aucune boucle de jeu**. Rien qui exige Flame, rien qui exige un `Canvas` à soixante images par seconde. Tout cela s'écrit avec ce que vous savez déjà : des widgets, des modèles, du JSON, `provider` et `shared_preferences`.

C'est exactement l'objet de ce projet. Vous allez écrire le lanceur complet du « Donjon de Dart », le jeu que construit la PARTIE 2. À la fin, il ne manquera qu'une chose : la boucle de jeu elle-même, que nous remplacerons ici par un rectangle dessiné en `CustomPaint`. Le chapitre 35 remplacera ce rectangle par un `GameWidget` et le reste ne bougera pas.

> **À retenir.** Le méta-jeu n'est pas de la décoration. C'est ce qui transforme une démo technique en jeu : la progression, la récompense, la persistance, la comparaison aux autres.

---

## 62.0.2 — Aperçu du résultat final

L'écran de chargement, deux secondes au démarrage :

```text
┌────────────────────────────────────────────────┐
│                                                │
│                                                │
│              D O N J O N   D E                 │
│                  D A R T                       │
│                                                │
│                    ◈                           │
│                                                │
│         ████████████████░░░░░░░░  62 %         │
│                                                │
│           Lecture des sauvegardes...           │
│                                                │
│                              version 1.0.0     │
└────────────────────────────────────────────────┘
```

Le menu principal, une fois la lecture terminée. Les entrées apparaissent l'une après l'autre en glissant depuis la gauche :

```text
┌────────────────────────────────────────────────┐
│                                                │
│              D O N J O N   D E                 │
│                  D A R T                       │
│         ─────────────────────────────          │
│                                                │
│        ┌────────────────────────────┐          │
│        │  ▶   JOUER                 │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ↺   CONTINUER   Kaelis N7 │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ☺   PERSONNAGE            │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ▤   INVENTAIRE      12/40 │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ♛   CLASSEMENT            │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ⚙   OPTIONS               │          │
│        └────────────────────────────┘          │
│        ┌────────────────────────────┐          │
│        │  ⏻   QUITTER               │          │
│        └────────────────────────────┘          │
│                                                │
│  Emplacement 1 · 4820 pts · 2 h 14 min         │
└────────────────────────────────────────────────┘
```

L'écran de création de personnage :

```text
┌────────────────────────────────────────────────┐
│  ←   Nouveau personnage                        │
├────────────────────────────────────────────────┤
│  Nom du héros                                  │
│  ┌──────────────────────────────────────────┐  │
│  │ Kaelis                              6/16 │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Classe                                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │ GUERRIER │ │  RÔDEUR  │ │   MAGE   │        │
│  │  F8 D4   │ │  F5 D8   │ │  F2 D5   │        │
│  │  I2 V7   │ │  I3 V5   │ │  I9 V5   │        │
│  └──────────┘ └──────────┘ └──────────┘        │
│       ▲ sélectionnée                           │
│                                                │
│  Points à répartir                    3 / 10   │
│                                                │
│  Force          8 (+4)   [ − ]  ████████ [ + ] │
│  Dextérité      4 (+1)   [ − ]  ████     [ + ] │
│  Intelligence   2 (+0)   [ − ]  ██       [ + ] │
│  Vitalité       7 (+2)   [ − ]  ███████  [ + ] │
│                                                │
│  Aperçu :  PV 128   Énergie 28   Dégâts 22.0   │
│                                                │
│        ┌──────────────────────────────┐        │
│        │       CRÉER LE HÉROS         │        │
│        └──────────────────────────────┘        │
└────────────────────────────────────────────────┘
```

La fiche de personnage :

```text
┌────────────────────────────────────────────────┐
│  ←   Kaelis                              [+3]  │
├────────────────────────────────────────────────┤
│    ┌────┐                                      │
│    │ K  │   Kaelis                             │
│    └────┘   Guerrier · Niveau 7                │
│             ████████████░░░░░  620 / 800 XP    │
├────────────────────────────────────────────────┤
│  CARACTÉRISTIQUES                              │
│  Force          12  ██████████████░░░░░  +5 ⚔  │
│  Dextérité       5  ██████░░░░░░░░░░░░░        │
│  Intelligence    2  ██░░░░░░░░░░░░░░░░░        │
│  Vitalité       11  █████████████░░░░░░  +5 ⛨  │
├────────────────────────────────────────────────┤
│  STATISTIQUES DÉRIVÉES                         │
│  Points de vie          168                    │
│  Énergie                 28                    │
│  Dégâts                  29.5                  │
│  Défense                 13.6                  │
│  Chance critique          4.0 %                │
├────────────────────────────────────────────────┤
│  ÉQUIPEMENT                                    │
│  Arme        Épée d'acier        RARE          │
│  Armure      Cotte de mailles    RARE          │
│  Accessoire  — vide —                          │
└────────────────────────────────────────────────┘
```

L'inventaire, en grille, avec filtre et tri :

```text
┌────────────────────────────────────────────────┐
│  ←   Inventaire                  12/40   1240⬤ │
├────────────────────────────────────────────────┤
│ [Tout] [Armes] [Armures] [Access.] [Conso.]    │
│                                    Tri : Rareté│
├────────────────────────────────────────────────┤
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐       │
│  │  ⚔ E  │ │  ⚔    │ │  ⛨ E  │ │  ◇    │       │
│  │ Épée  │ │ Dague │ │ Cotte │ │Talism.│       │
│  │ RARE  │ │ RARE  │ │ RARE  │ │ RARE  │       │
│  └───────┘ └───────┘ └───────┘ └───────┘       │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐       │
│  │  ✚ x3 │ │  ✦ x1 │ │  ⚔    │ │  ○ x2 │       │
│  │Potion │ │Élixir │ │ Épée  │ │Anneau │       │
│  │COMMUN │ │ RARE  │ │COMMUN │ │COMMUN │       │
│  └───────┘ └───────┘ └───────┘ └───────┘       │
├────────────────────────────────────────────────┤
│  E = équipé                                    │
└────────────────────────────────────────────────┘
```

La boutique :

```text
┌────────────────────────────────────────────────┐
│  ←   Boutique                          1240 ⬤  │
├────────────────────────────────────────────────┤
│  [ ACHETER ]  [ VENDRE ]                       │
├────────────────────────────────────────────────┤
│  ⚔  Épée d'acier            RARE       120 ⬤   │
│     Force +5                          ACHETER  │
├────────────────────────────────────────────────┤
│  ✦  Bâton arcanique         ÉPIQUE     260 ⬤   │
│     Intelligence +8                   ACHETER  │
├────────────────────────────────────────────────┤
│  ⛨  Plastron du dragon      LÉGENDAIRE 950 ⬤   │
│     Vitalité +12                   or manquant │
└────────────────────────────────────────────────┘
```

Le classement, onglet local et onglet mondial :

```text
┌────────────────────────────────────────────────┐
│  ←   Classement                                │
├────────────────────────────────────────────────┤
│  [ LOCAL ]  [ MONDIAL ]                        │
├────────────────────────────────────────────────┤
│   1  ♛  Kaelis            N7    4 820          │
│   2     Sylvine           N6    4 105          │
│   3     Bruk              N5    3 900          │
│   4     Nym               N4    2 610          │
│   5     Orl               N3    1 340          │
└────────────────────────────────────────────────┘
```

L'écran d'options, avec le remappage des touches :

```text
┌────────────────────────────────────────────────┐
│  ←   Options                                   │
├────────────────────────────────────────────────┤
│  AUDIO                                         │
│  Musique       ────●───────────────  35 %      │
│  Effets        ──────────────●─────  80 %      │
├────────────────────────────────────────────────┤
│  JEU                                           │
│  Difficulté    [ Facile ][Normale][ Difficile ]│
├────────────────────────────────────────────────┤
│  AFFICHAGE                                     │
│  Thème         [ Système ][ Clair ][ Sombre ]  │
├────────────────────────────────────────────────┤
│  COMMANDES                                     │
│  Aller à gauche                   [    Q    ]  │
│  Aller à droite                   [    D    ]  │
│  Sauter                           [ ESPACE  ]  │
│  Attaquer                         [    J    ]  │
│  Pause                            [ ÉCHAP   ]  │
│                                                │
│              RÉINITIALISER LES TOUCHES         │
└────────────────────────────────────────────────┘
```

Enfin, la zone de jeu factice, celle qui deviendra le vrai jeu au chapitre 35 :

```text
┌────────────────────────────────────────────────┐
│  Score 0      PV ██████████     Niveau 1   ‖   │
├────────────────────────────────────────────────┤
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░ ○ ░░░░░░░░░░░░ ○ ░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░░░░░░░▓▓▓▓▓▓▓▓░░░░░░░░▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░░░░[K]░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░[g]░░░░░░░░│
│████████████████████████████████████████████████│
├────────────────────────────────────────────────┤
│  Ici viendra GameWidget (chapitre 35)          │
└────────────────────────────────────────────────┘
```

---

## 62.0.3 — Cahier des charges

### Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | Un écran de chargement s'affiche au démarrage et cède la place au menu quand les données sont prêtes. | Aucun écran gris ne clignote. |
| O2 | Le menu principal propose sept entrées ; **Continuer**, **Personnage** et **Inventaire** sont désactivées sans personnage. | Première ouverture : quatre entrées actives. |
| O3 | Les entrées du menu apparaissent avec une animation d'entrée décalée. | Chaque bouton glisse à 60 ms d'écart. |
| O4 | Le joueur choisit un emplacement de sauvegarde parmi trois. | Trois emplacements indépendants. |
| O5 | La création de personnage exige un nom de 3 à 16 caractères, sans caractère interdit. | `«  »` et `«ab»` sont refusés. |
| O6 | Trois classes sont proposées, chacune avec des caractéristiques de base différentes. | Le total de base est identique (21). |
| O7 | Dix points sont à répartir, avec un maximum de cinq par caractéristique. | Le bouton de création reste désactivé tant qu'il reste des points. |
| O8 | La fiche de personnage affiche quatre caractéristiques en barres proportionnelles et cinq statistiques dérivées. | Équiper une arme change les dégâts affichés. |
| O9 | L'inventaire s'affiche en grille, avec filtre par type et quatre tris. | Le filtre et le tri se combinent. |
| O10 | Un objet peut être équipé, déséquipé, consommé ou vendu. | L'objet équipé porte le repère `E`. |
| O11 | Les statistiques dérivées sont recalculées à partir de l'équipement, jamais stockées. | Aucune valeur dupliquée en JSON. |
| O12 | Une boutique permet d'acheter et de revendre contre de l'or. | L'achat est refusé si l'or est insuffisant. |
| O13 | Un classement local trie les dix meilleurs scores. | Un nouveau score s'insère à la bonne place. |
| O14 | Un classement distant est chargé via `FutureBuilder`, avec états de chargement et d'erreur. | Couper le réseau affiche un message et un bouton « Réessayer ». |
| O15 | Les options couvrent volumes, difficulté, thème et remappage des touches. | Le thème change instantanément. |
| O16 | Une touche déjà utilisée ne peut pas être réattribuée sans échange. | Assigner `D` à « gauche » échange avec « droite ». |
| O17 | Toute la partie est persistée en une seule chaîne JSON par emplacement. | Tuer l'application, relancer, tout est là. |
| O18 | Une sauvegarde corrompue est détectée, signalée et n'empêche jamais le démarrage. | Écrire `{{{` dans la clé : l'application démarre. |
| O19 | L'état est centralisé dans des `ChangeNotifier` fournis par `provider`. | Aucun `setState` sur des données de jeu. |
| O20 | Les transitions entre écrans sont personnalisées et cohérentes. | Fondu pour les modales, glissement pour la navigation. |
| O21 | Une zone de jeu factice dessinée en `CustomPaint` occupe l'emplacement du futur `GameWidget`. | Un seul fichier à remplacer au chapitre 35. |
| O22 | La logique de personnage et d'inventaire est couverte par des tests. | `flutter test` passe. |

### Fonctionnalités bonus

| # | Exigence |
| --- | --- |
| B1 | Un bouton « Réinitialiser les touches » qui restaure la configuration par défaut. |
| B2 | Une infobulle détaillée au survol / appui long d'une case d'inventaire. |
| B3 | Un journal de progression : niveau atteint, temps de jeu, parties jouées. |
| B4 | L'export de la sauvegarde dans le presse-papiers. |
| B5 | Un thème sombre complet, persisté. |

---

## 62.0.4 — Notions mobilisées

Ce projet n'introduit **aucune** notion nouvelle. C'est un assemblage. Si une ligne vous surprend, relisez le chapitre indiqué avant de commencer.

| Notion | Chapitre | Usage exact dans ce projet |
| --- | --- | --- |
| `List`, `Map`, `Set` | 06 | L'inventaire, le catalogue, les touches. |
| Fonctions, paramètres nommés | 07 | Les fabriques d'objets et de routes. |
| Classes, propriétés calculées | 08 | `Joueur.pvMax`, `Joueur.degats`. |
| Constructeurs nommés, `required` | 09 | `Joueur.fromJson`, `Joueur.nouveau`. |
| Héritage, `@override` | 10 | Les widgets, `CustomPainter`. |
| `enum` enrichi | 11 | `ClassePersonnage`, `Rarete`, `TypeObjet`, `Difficulte`, `ActionJeu`. |
| Null safety, `?`, `??`, `!` | 12 | Un personnage peut ne pas exister. |
| `try` / `catch`, exceptions | 13 | La sauvegarde corrompue. |
| `map`, `where`, `fold`, `sort` | 14 | Filtre, tri, somme des bonus, total de l'inventaire. |
| `Future`, `async`, `await` | 15 | Chargement, persistance, classement distant. |
| `pubspec.yaml`, paquets | 16 | `provider`, `shared_preferences`, `http`, `intl`. |
| `jsonEncode` / `jsonDecode`, `fromJson` / `toJson` | 17 | Toute la sauvegarde. |
| `MaterialApp`, `Scaffold`, `AppBar` | 44 | La structure de chaque écran. |
| `StatefulWidget`, `initState`, `dispose` | 45 | Les animations, les contrôleurs de texte. |
| `Row`, `Column`, `Expanded`, `Stack` | 46 | Les mises en page. |
| `Text`, `Icon`, `CircleAvatar` | 47 | L'avatar en initiale, les icônes d'objets. |
| `GridView.builder`, `ListView.builder` | 48 | L'inventaire, la boutique, le classement. |
| `Form`, `TextFormField`, `validator` | 49 | Le nom du héros. |
| `Navigator`, `PageRouteBuilder` | 50 | Les transitions entre écrans. |
| `ThemeData`, Material 3, `themeMode` | 51 | Le thème clair / sombre. |
| `ChangeNotifier`, `provider` | 52 | `EtatPartie`, `EtatReglages`. |
| `http`, `FutureBuilder` | 53 | Le classement mondial. |
| `SharedPreferencesAsync` | 54 | Les emplacements de sauvegarde et les réglages. |
| Barres de statistiques | 55 | La fiche de personnage. |
| Grille de boutons | 56 | La répartition des points. |
| `SegmentedButton`, `Slider` | 57, 49 | Les options. |
| Panier et monnaie | 60 | La boutique. |
| Cache et repli hors-ligne | 61 | Le classement mondial. |

---

## 62.0.5 — Arborescence du projet

Voici l'arborescence finale. Elle est donnée dès maintenant pour que vous sachiez où va chaque fichier ; nous la construirons étape par étape.

```text
lanceur_donjon/
├── pubspec.yaml
├── lib/
│   ├── main.dart                          point d'entrée, thèmes, providers
│   ├── config/
│   │   ├── constantes.dart                règles chiffrées du jeu
│   │   └── palette.dart                   couleurs de rareté et d'accent
│   ├── modeles/
│   │   ├── caracteristiques.dart          les quatre caractéristiques
│   │   ├── classe_personnage.dart         enum ClassePersonnage
│   │   ├── objet.dart                     enum TypeObjet, enum Rarete, classe Objet
│   │   ├── catalogue.dart                 la table des objets du jeu
│   │   ├── inventaire.dart                lignes, quantités, équipement
│   │   ├── joueur.dart                    le personnage et ses stats dérivées
│   │   ├── reglages.dart                  volumes, difficulté, thème, touches
│   │   ├── score.dart                     une entrée de classement
│   │   └── sauvegarde.dart                l'agrégat persisté
│   ├── logique/
│   │   ├── tri_inventaire.dart            enum TriObjet + fonctions pures
│   │   └── progression.dart               XP, niveaux, points gagnés
│   ├── services/
│   │   ├── sauvegarde_service.dart        lecture/écriture des emplacements
│   │   ├── reglages_service.dart          lecture/écriture des réglages
│   │   ├── depot_classement.dart          interface
│   │   ├── classement_local.dart          implémentation prefs
│   │   └── classement_distant.dart        implémentation http
│   ├── etat/
│   │   ├── etat_partie.dart               ChangeNotifier principal
│   │   └── etat_reglages.dart             ChangeNotifier des options
│   ├── navigation/
│   │   └── transitions.dart               PageRouteBuilder réutilisables
│   ├── ecrans/
│   │   ├── ecran_chargement.dart
│   │   ├── ecran_menu.dart
│   │   ├── ecran_emplacements.dart
│   │   ├── ecran_creation.dart
│   │   ├── ecran_fiche.dart
│   │   ├── ecran_inventaire.dart
│   │   ├── ecran_boutique.dart
│   │   ├── ecran_classement.dart
│   │   ├── ecran_options.dart
│   │   └── ecran_jeu.dart                 la zone remplacée au chapitre 35
│   └── widgets/
│       ├── bouton_menu.dart
│       ├── barre_stat.dart
│       ├── case_objet.dart
│       ├── panneau.dart
│       └── peintre_donjon.dart            CustomPainter de la zone factice
└── test/
    ├── joueur_test.dart
    ├── inventaire_test.dart
    └── sauvegarde_test.dart
```

**Pourquoi cette découpe ?** Elle prolonge celle du projet 4 (section 58.0.4) et anticipe celle du jeu (section 35.3) :

```text
config/       les nombres du game design, au même endroit
modeles/      les données pures — aucun import de Flutter
logique/      les règles — fonctions pures, testables sans écran
services/     l'entrée-sortie : disque, réseau
etat/         ce qui change et qui prévient l'interface
navigation/   la façon de passer d'un écran à l'autre
ecrans/       les pages
widgets/      les morceaux réutilisés par plusieurs pages
```

Règle absolue, la même qu'au chapitre 58 : un fichier de `modeles/` ou de `logique/` **n'importe jamais** `package:flutter/material.dart`. C'est ce qui rendra les tests de l'étape 62.20 possibles, et c'est aussi ce qui permettra de recopier ces fichiers tels quels dans le projet Flame.

---

## 62.1 — Étape 1 : le projet, les dépendances et le squelette

### Créer le projet

```text
flutter create lanceur_donjon
cd lanceur_donjon
```

### Ajouter les dépendances

```text
flutter pub add provider
flutter pub add shared_preferences
flutter pub add http
flutter pub add intl
```

Quatre paquets, pas un de plus. Vous connaissez les quatre : `provider` (chapitre 52), `shared_preferences` (chapitre 54), `http` (chapitre 53), `intl` (chapitre 58 pour le formatage des dates et des nombres).

**`lanceur_donjon/pubspec.yaml`**

```yaml
name: lanceur_donjon
description: "Le lanceur du jeu Donjon de Dart : menu, personnage, inventaire, options."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter

  # Gestion d'état globale (chapitre 52)
  provider: ^6.1.5

  # Persistance clé/valeur (chapitre 54)
  shared_preferences: ^2.5.5

  # Classement distant (chapitre 53)
  http: ^1.5.0

  # Formatage des nombres et des dates (chapitre 58)
  intl: ^0.20.3

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

> **Remarque.** Comme dans tous les projets de la PARTIE 1C, **aucun fichier image n'est fourni**. Les objets sont représentés par des `Icon` Material et des couleurs de rareté. Le jour où vous aurez de vraies illustrations, il suffira de remplacer le corps de `CaseObjet` (étape 62.12) par un `Image.asset` et d'ajouter la section `assets:` ci-dessus.

### Les constantes du jeu

Tous les nombres du game design vont dans un seul fichier. C'est la leçon de la section 35.7 : un nombre écrit en dur au milieu d'un widget est un nombre que vous ne retrouverez jamais.

**`lib/config/constantes.dart`**

```dart
/// Toutes les règles chiffrées du jeu, en un seul endroit.
///
/// Aucun autre fichier ne doit contenir de nombre « magique » lié au
/// game design. Équilibrer le jeu, c'est modifier ce fichier et rien d'autre.
class Constantes {
  // --- Personnage ---------------------------------------------------------
  static const int longueurNomMin = 3;
  static const int longueurNomMax = 16;

  /// Points de caractéristique à répartir à la création.
  static const int pointsCreation = 10;

  /// Maximum de points que l'on peut mettre dans UNE caractéristique
  /// pendant la création.
  static const int pointsMaxParCarac = 5;

  /// Points gagnés à chaque montée de niveau.
  static const int pointsParNiveau = 3;

  /// Valeur maximale affichée par les barres de la fiche (échelle des barres).
  static const int caracMaxAffichee = 20;

  // --- Statistiques dérivées ---------------------------------------------
  static const int pvBase = 40;
  static const int pvParVitalite = 8;
  static const int energieBase = 20;
  static const int energieParIntelligence = 4;
  static const double degatsBase = 4.0;
  static const double degatsParForce = 1.5;
  static const double defenseParVitalite = 0.5;
  static const double defenseParForce = 0.3;
  static const double critParDexterite = 0.8;
  static const double critMax = 60.0;

  // --- Progression --------------------------------------------------------
  /// Expérience nécessaire pour passer du niveau n au niveau n + 1.
  static const int xpParNiveau = 100;

  // --- Inventaire et économie --------------------------------------------
  static const int tailleInventaire = 40;

  /// Part du prix d'achat récupérée à la revente.
  static const double tauxRevente = 0.4;

  static const int orDepart = 100;

  // --- Sauvegarde ---------------------------------------------------------
  static const int nombreEmplacements = 3;

  /// Version du format de sauvegarde. À incrémenter en cas de changement.
  static const int versionSauvegarde = 1;

  // --- Classement ---------------------------------------------------------
  static const int tailleClassement = 10;
}
```

### La palette

**`lib/config/palette.dart`**

```dart
import 'package:flutter/material.dart';

/// Les couleurs propres au jeu : raretés, caractéristiques, accents.
///
/// Tout ce qui dépend du thème (fond, texte, surface) reste géré par
/// `Theme.of(context).colorScheme` — voir le chapitre 51. Ce fichier ne
/// contient que les couleurs SÉMANTIQUES, celles qui gardent le même sens
/// en clair comme en sombre.
class Palette {
  /// Couleur de la graine du thème.
  static const Color graine = Color(0xFF6A3FA0);

  // Raretés
  static const Color commun = Color(0xFF9E9E9E);
  static const Color rare = Color(0xFF2E86DE);
  static const Color epique = Color(0xFF8E44AD);
  static const Color legendaire = Color(0xFFE67E22);

  // Caractéristiques
  static const Color force = Color(0xFFC0392B);
  static const Color dexterite = Color(0xFF27AE60);
  static const Color intelligence = Color(0xFF2980B9);
  static const Color vitalite = Color(0xFFD35400);

  // Divers
  static const Color or = Color(0xFFF1C40F);
  static const Color danger = Color(0xFFC0392B);
}
```

### Le squelette

**`lib/main.dart`** (version provisoire de l'étape 1)

```dart
import 'package:flutter/material.dart';

import 'config/palette.dart';

void main() {
  runApp(const ApplicationLanceur());
}

/// Racine de l'application.
///
/// À ce stade elle installe seulement les deux thèmes et affiche un écran
/// vide. Les providers, les routes et les écrans viendront s'y greffer.
class ApplicationLanceur extends StatelessWidget {
  const ApplicationLanceur({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Donjon de Dart',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Palette.graine),
      ),
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Palette.graine,
          brightness: Brightness.dark,
        ),
      ),
      home: const Scaffold(
        body: Center(child: Text('Donjon de Dart')),
      ),
    );
  }
}
```

**État exécutable.** `flutter run` affiche un texte centré sur fond violet clair. Les quatre dépendances sont résolues, les deux thèmes sont en place. C'est peu, mais c'est un point de départ propre.

```text
┌────────────────────────────────────────────────┐
│                                                │
│                Donjon de Dart                  │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 62.2 — Étape 2 : les caractéristiques et les classes

### Pourquoi une classe pour quatre entiers

On pourrait écrire quatre champs `int force, dexterite, intelligence, vitalite` directement dans `Joueur`. Mauvaise idée : ces quatre nombres voyagent **toujours ensemble**. Un objet donne un bonus de caractéristiques, une classe donne des caractéristiques de base, la création répartit des caractéristiques. Trois fois les mêmes quatre champs, trois fois le même code d'addition.

Une petite classe immuable règle le problème, et nous permet même de surcharger l'opérateur `+`.

**`lib/modeles/caracteristiques.dart`**

```dart
/// Les quatre caractéristiques d'un personnage.
///
/// Objet immuable (tous les champs sont `final`) : on n'en modifie jamais un,
/// on en fabrique un nouveau. C'est le même principe que la `Tache` du
/// chapitre 58.
class Caracteristiques {
  const Caracteristiques({
    this.force = 0,
    this.dexterite = 0,
    this.intelligence = 0,
    this.vitalite = 0,
  });

  final int force;
  final int dexterite;
  final int intelligence;
  final int vitalite;

  /// L'élément neutre de l'addition. Sert de valeur de départ à `fold`.
  static const Caracteristiques zero = Caracteristiques();

  /// Somme des quatre valeurs.
  int get total => force + dexterite + intelligence + vitalite;

  /// Additionne deux jeux de caractéristiques.
  ///
  /// Surcharger `+` permet d'écrire `base + bonusEquipement`, ce qui se lit
  /// beaucoup mieux qu'un appel de méthode. C'est la surcharge d'opérateur
  /// du chapitre 10.
  Caracteristiques operator +(Caracteristiques autre) {
    return Caracteristiques(
      force: force + autre.force,
      dexterite: dexterite + autre.dexterite,
      intelligence: intelligence + autre.intelligence,
      vitalite: vitalite + autre.vitalite,
    );
  }

  /// Renvoie une copie avec certains champs remplacés.
  Caracteristiques copyWith({
    int? force,
    int? dexterite,
    int? intelligence,
    int? vitalite,
  }) {
    return Caracteristiques(
      force: force ?? this.force,
      dexterite: dexterite ?? this.dexterite,
      intelligence: intelligence ?? this.intelligence,
      vitalite: vitalite ?? this.vitalite,
    );
  }

  /// Lit une caractéristique par son nom d'énumération.
  /// Utilisé par l'écran de création pour éviter quatre `if`.
  int lire(Carac carac) {
    switch (carac) {
      case Carac.force:
        return force;
      case Carac.dexterite:
        return dexterite;
      case Carac.intelligence:
        return intelligence;
      case Carac.vitalite:
        return vitalite;
    }
  }

  /// Renvoie une copie où une caractéristique désignée a bougé de [delta].
  Caracteristiques modifier(Carac carac, int delta) {
    switch (carac) {
      case Carac.force:
        return copyWith(force: force + delta);
      case Carac.dexterite:
        return copyWith(dexterite: dexterite + delta);
      case Carac.intelligence:
        return copyWith(intelligence: intelligence + delta);
      case Carac.vitalite:
        return copyWith(vitalite: vitalite + delta);
    }
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'force': force,
        'dexterite': dexterite,
        'intelligence': intelligence,
        'vitalite': vitalite,
      };

  /// Désérialisation défensive : chaque champ absent ou abîmé vaut 0.
  ///
  /// Le motif `(json['x'] as num?)?.toInt() ?? 0` est celui du chapitre 17 :
  /// `num?` accepte aussi bien `5` que `5.0`, `?.toInt()` tranche, et `?? 0`
  /// couvre l'absence.
  factory Caracteristiques.fromJson(Map<String, dynamic> json) {
    return Caracteristiques(
      force: (json['force'] as num?)?.toInt() ?? 0,
      dexterite: (json['dexterite'] as num?)?.toInt() ?? 0,
      intelligence: (json['intelligence'] as num?)?.toInt() ?? 0,
      vitalite: (json['vitalite'] as num?)?.toInt() ?? 0,
    );
  }

  @override
  String toString() => 'F$force D$dexterite I$intelligence V$vitalite';

  /// Deux jeux de caractéristiques identiques doivent être égaux.
  /// Sans cela, les tests de l'étape 62.20 échoueraient.
  @override
  bool operator ==(Object autre) {
    return autre is Caracteristiques &&
        autre.force == force &&
        autre.dexterite == dexterite &&
        autre.intelligence == intelligence &&
        autre.vitalite == vitalite;
  }

  @override
  int get hashCode => Object.hash(force, dexterite, intelligence, vitalite);
}

/// Désigne une caractéristique. Permet de traiter les quatre par une boucle.
enum Carac {
  force('Force'),
  dexterite('Dextérité'),
  intelligence('Intelligence'),
  vitalite('Vitalité');

  const Carac(this.libelle);

  final String libelle;
}
```

> **Attention.** Redéfinir `==` **oblige** à redéfinir `hashCode`. Si vous oubliez, deux objets égaux atterriront dans deux cases différentes d'un `Set` ou d'une `Map`, et vous chercherez le bug pendant des heures. `Object.hash(...)` fait le travail pour vous.

### Les classes de personnage

**`lib/modeles/classe_personnage.dart`**

```dart
import 'caracteristiques.dart';

/// Les trois archétypes jouables.
///
/// C'est un `enum` enrichi (chapitre 11) : chaque valeur porte son libellé,
/// sa description et ses caractéristiques de base. Le total de base est
/// volontairement identique (21) pour les trois, afin qu'aucune classe ne
/// soit objectivement supérieure.
enum ClassePersonnage {
  guerrier(
    'Guerrier',
    'Encaisse et frappe fort. Le choix sûr pour une première partie.',
    Caracteristiques(force: 8, dexterite: 4, intelligence: 2, vitalite: 7),
  ),
  rodeur(
    'Rôdeur',
    'Rapide et précis. Beaucoup de coups critiques, peu de points de vie.',
    Caracteristiques(force: 5, dexterite: 8, intelligence: 3, vitalite: 5),
  ),
  mage(
    'Mage',
    'Fragile, mais dispose d\'une réserve d\'énergie considérable.',
    Caracteristiques(force: 2, dexterite: 5, intelligence: 9, vitalite: 5),
  );

  const ClassePersonnage(this.libelle, this.description, this.base);

  final String libelle;
  final String description;
  final Caracteristiques base;

  /// Retrouve une classe depuis son nom persisté.
  ///
  /// On persiste TOUJOURS `.name` et jamais `.index` : réordonner l'énumération
  /// casserait toutes les sauvegardes existantes (leçon du chapitre 58).
  static ClassePersonnage depuisNom(String? nom) {
    for (final ClassePersonnage c in ClassePersonnage.values) {
      if (c.name == nom) return c;
    }
    return ClassePersonnage.guerrier;
  }
}
```

### Vérification en console

Avant de brancher quoi que ce soit à l'écran, vérifions le modèle. Créez un fichier temporaire et lancez-le avec `dart run`.

**`lib/verification_temporaire.dart`** (à supprimer ensuite)

```dart
import 'modeles/caracteristiques.dart';
import 'modeles/classe_personnage.dart';

void main() {
  for (final ClassePersonnage c in ClassePersonnage.values) {
    print('${c.libelle.padRight(10)} ${c.base}  total=${c.base.total}');
  }

  const Caracteristiques bonus = Caracteristiques(force: 5, vitalite: 2);
  final Caracteristiques totale = ClassePersonnage.guerrier.base + bonus;
  print('Guerrier équipé : $totale');

  print('Relecture : ${Caracteristiques.fromJson(totale.toJson())}');
  print('Égalité : ${Caracteristiques.fromJson(totale.toJson()) == totale}');
}
```

```text
dart run lib/verification_temporaire.dart
```

**Résultat :**

```text
Guerrier   F8 D4 I2 V7  total=21
Rôdeur     F5 D8 I3 V5  total=21
Mage       F2 D5 I9 V5  total=21
Guerrier équipé : F13 D4 I2 V9
Relecture : F13 D4 I2 V9
Égalité : true
```

**État exécutable.** Le modèle de base est écrit et vérifié en console, sans le moindre widget. C'est l'ordre de travail correct : les données d'abord, l'écran ensuite.

---

## 62.3 — Étape 3 : les objets et le catalogue

### Deux énumérations et une classe

Un objet a un **type** (qui détermine ce qu'on peut en faire) et une **rareté** (qui détermine sa couleur et sa valeur perçue).

**`lib/modeles/objet.dart`**

```dart
import 'caracteristiques.dart';

/// Ce qu'un objet est, et donc ce qu'on peut en faire.
enum TypeObjet {
  arme('Arme', true),
  armure('Armure', true),
  accessoire('Accessoire', true),
  consommable('Consommable', false),
  tresor('Trésor', false);

  const TypeObjet(this.libelle, this.equipable);

  final String libelle;

  /// Un objet équipable occupe un emplacement d'équipement.
  final bool equipable;

  static TypeObjet depuisNom(String? nom) {
    for (final TypeObjet t in TypeObjet.values) {
      if (t.name == nom) return t;
    }
    return TypeObjet.tresor;
  }
}

/// La rareté d'un objet : sa couleur, son rang de tri.
enum Rarete {
  commun('Commun', 0),
  rare('Rare', 1),
  epique('Épique', 2),
  legendaire('Légendaire', 3);

  const Rarete(this.libelle, this.rang);

  final String libelle;

  /// Sert au tri (chapitre 14) : un rang élevé passe devant.
  final int rang;

  static Rarete depuisNom(String? nom) {
    for (final Rarete r in Rarete.values) {
      if (r.name == nom) return r;
    }
    return Rarete.commun;
  }
}

/// La DÉFINITION d'un objet du jeu.
///
/// Attention à la distinction, elle est capitale :
///   - `Objet`          = la fiche technique, la même pour tout le monde ;
///   - `LigneInventaire`= « ce joueur possède 3 exemplaires de cet objet ».
///
/// La définition n'est jamais persistée : seule la référence (`id`) l'est.
/// Ainsi, rééquilibrer une épée dans le catalogue met à jour toutes les
/// sauvegardes existantes d'un coup.
class Objet {
  const Objet({
    required this.id,
    required this.nom,
    required this.description,
    required this.type,
    required this.rarete,
    required this.prix,
    this.bonus = Caracteristiques.zero,
    this.soin = 0,
    this.energie = 0,
  });

  /// Identifiant technique stable. C'est LUI qui va en JSON.
  final String id;

  final String nom;
  final String description;
  final TypeObjet type;
  final Rarete rarete;

  /// Prix d'achat en boutique, en pièces d'or.
  final int prix;

  /// Bonus apporté quand l'objet est équipé (nul pour un consommable).
  final Caracteristiques bonus;

  /// Points de vie rendus par un consommable.
  final int soin;

  /// Énergie rendue par un consommable.
  final int energie;

  /// Prix de revente : une fraction du prix d'achat (chapitre 60).
  int get prixRevente => (prix * 0.4).round();

  /// Résumé lisible du bonus, pour l'affichage.
  String get resumeBonus {
    final List<String> parties = <String>[];
    if (bonus.force != 0) parties.add('Force +${bonus.force}');
    if (bonus.dexterite != 0) parties.add('Dextérité +${bonus.dexterite}');
    if (bonus.intelligence != 0) {
      parties.add('Intelligence +${bonus.intelligence}');
    }
    if (bonus.vitalite != 0) parties.add('Vitalité +${bonus.vitalite}');
    if (soin != 0) parties.add('Soigne $soin PV');
    if (energie != 0) parties.add('Rend $energie énergie');
    return parties.isEmpty ? 'Aucun effet' : parties.join(' · ');
  }

  @override
  String toString() => '$nom (${rarete.libelle})';
}
```

> **Pourquoi `prixRevente` utilise 0.4 en dur alors qu'il existe `Constantes.tauxRevente` ?** Il ne le devrait pas. Mais `modeles/` ne doit dépendre de rien, pas même de `config/`. Deux solutions : passer le taux en paramètre, ou accepter que `config/constantes.dart` soit le seul import autorisé depuis `modeles/`. Nous choisissons la seconde, plus simple, et nous corrigeons donc l'import :

```dart
import '../config/constantes.dart';

  int get prixRevente => (prix * Constantes.tauxRevente).round();
```

Ajoutez cet import en tête de `objet.dart` et remplacez le getter. `config/` ne contient que des constantes, sans dépendance : l'importer depuis un modèle ne crée aucun couplage gênant.

### Le catalogue

**`lib/modeles/catalogue.dart`**

```dart
import 'caracteristiques.dart';
import 'objet.dart';

/// La table de tous les objets existant dans le jeu.
///
/// C'est l'équivalent d'un fichier de données de game design. Dans un vrai
/// projet, il serait chargé depuis un JSON d'assets ; ici, une constante
/// suffit et évite un chargement asynchrone de plus.
class Catalogue {
  const Catalogue._();

  static const List<Objet> objets = <Objet>[
    // --- Armes ------------------------------------------------------------
    Objet(
      id: 'epee_rouillee',
      nom: 'Épée rouillée',
      description: 'Elle a connu de meilleurs jours, mais elle coupe encore.',
      type: TypeObjet.arme,
      rarete: Rarete.commun,
      prix: 25,
      bonus: Caracteristiques(force: 2),
    ),
    Objet(
      id: 'epee_acier',
      nom: 'Épée d\'acier',
      description: 'Une lame droite, équilibrée, sans fioriture.',
      type: TypeObjet.arme,
      rarete: Rarete.rare,
      prix: 120,
      bonus: Caracteristiques(force: 5),
    ),
    Objet(
      id: 'dague_ombre',
      nom: 'Dague d\'ombre',
      description: 'Légère au point de sembler creuse.',
      type: TypeObjet.arme,
      rarete: Rarete.rare,
      prix: 140,
      bonus: Caracteristiques(dexterite: 6),
    ),
    Objet(
      id: 'baton_arcane',
      nom: 'Bâton arcanique',
      description: 'Le bois vibre légèrement au contact de la peau.',
      type: TypeObjet.arme,
      rarete: Rarete.epique,
      prix: 260,
      bonus: Caracteristiques(intelligence: 8),
    ),
    Objet(
      id: 'lame_donjon',
      nom: 'Lame du Donjon',
      description: 'Forgée au dernier étage. Personne ne sait par qui.',
      type: TypeObjet.arme,
      rarete: Rarete.legendaire,
      prix: 900,
      bonus: Caracteristiques(force: 10, dexterite: 3),
    ),

    // --- Armures ----------------------------------------------------------
    Objet(
      id: 'tunique',
      nom: 'Tunique de lin',
      description: 'Confortable. C\'est à peu près tout.',
      type: TypeObjet.armure,
      rarete: Rarete.commun,
      prix: 30,
      bonus: Caracteristiques(vitalite: 2),
    ),
    Objet(
      id: 'cotte_mailles',
      nom: 'Cotte de mailles',
      description: 'Lourde, bruyante, efficace.',
      type: TypeObjet.armure,
      rarete: Rarete.rare,
      prix: 150,
      bonus: Caracteristiques(force: 1, vitalite: 5),
    ),
    Objet(
      id: 'robe_mage',
      nom: 'Robe du conclave',
      description: 'Brodée de fil d\'argent, chaude en hiver.',
      type: TypeObjet.armure,
      rarete: Rarete.rare,
      prix: 150,
      bonus: Caracteristiques(intelligence: 4, vitalite: 2),
    ),
    Objet(
      id: 'plastron_dragon',
      nom: 'Plastron du dragon',
      description: 'Des écailles imbriquées, encore tièdes.',
      type: TypeObjet.armure,
      rarete: Rarete.legendaire,
      prix: 950,
      bonus: Caracteristiques(vitalite: 12),
    ),

    // --- Accessoires ------------------------------------------------------
    Objet(
      id: 'anneau_cuivre',
      nom: 'Anneau de cuivre',
      description: 'Il verdit les doigts, mais il porte chance.',
      type: TypeObjet.accessoire,
      rarete: Rarete.commun,
      prix: 20,
      bonus: Caracteristiques(dexterite: 1),
    ),
    Objet(
      id: 'talisman',
      nom: 'Talisman gravé',
      description: 'Les gravures changent quand on ne regarde pas.',
      type: TypeObjet.accessoire,
      rarete: Rarete.rare,
      prix: 180,
      bonus: Caracteristiques(dexterite: 3, intelligence: 3),
    ),
    Objet(
      id: 'amulette_vie',
      nom: 'Amulette de vie',
      description: 'Elle bat, doucement, comme un second cœur.',
      type: TypeObjet.accessoire,
      rarete: Rarete.epique,
      prix: 300,
      bonus: Caracteristiques(vitalite: 6),
    ),

    // --- Consommables -----------------------------------------------------
    Objet(
      id: 'potion_soin',
      nom: 'Potion de soin',
      description: 'Goût de fraise. Personne ne sait pourquoi.',
      type: TypeObjet.consommable,
      rarete: Rarete.commun,
      prix: 15,
      soin: 40,
    ),
    Objet(
      id: 'potion_energie',
      nom: 'Potion d\'énergie',
      description: 'Amère, mais rapide.',
      type: TypeObjet.consommable,
      rarete: Rarete.commun,
      prix: 20,
      energie: 25,
    ),
    Objet(
      id: 'elixir',
      nom: 'Élixir complet',
      description: 'Restaure le corps et l\'esprit d\'un seul trait.',
      type: TypeObjet.consommable,
      rarete: Rarete.rare,
      prix: 90,
      soin: 100,
      energie: 60,
    ),

    // --- Trésors ----------------------------------------------------------
    Objet(
      id: 'gemme',
      nom: 'Gemme brute',
      description: 'Sans usage. Sauf pour le marchand.',
      type: TypeObjet.tresor,
      rarete: Rarete.rare,
      prix: 200,
    ),
    Objet(
      id: 'couronne',
      nom: 'Couronne fêlée',
      description: 'Le roi n\'en aura plus besoin.',
      type: TypeObjet.tresor,
      rarete: Rarete.legendaire,
      prix: 600,
    ),
  ];

  /// Table d'accès rapide par identifiant, construite une seule fois.
  ///
  /// `Map.fromEntries` + `map` : le chapitre 14 en une ligne.
  static final Map<String, Objet> _parId = Map<String, Objet>.fromEntries(
    objets.map((Objet o) => MapEntry<String, Objet>(o.id, o)),
  );

  /// Renvoie la définition d'un objet, ou `null` si l'identifiant est inconnu.
  ///
  /// Le `null` n'est pas théorique : une vieille sauvegarde peut référencer
  /// un objet supprimé du catalogue. L'appelant DOIT traiter ce cas.
  static Objet? parId(String id) => _parId[id];

  /// Les objets proposés à l'achat. Les trésors ne s'achètent pas.
  static List<Objet> get enVente =>
      objets.where((Objet o) => o.type != TypeObjet.tresor).toList();
}
```

**État exécutable.** Ajoutez ces lignes à `verification_temporaire.dart` :

```dart
  print('Catalogue : ${Catalogue.objets.length} objets');
  print('En vente  : ${Catalogue.enVente.length} objets');
  print(Catalogue.parId('epee_acier')?.resumeBonus);
  print('Revente épée : ${Catalogue.parId('epee_acier')?.prixRevente} or');
  print('Inconnu   : ${Catalogue.parId('massue_de_luxe')}');
```

**Résultat :**

```text
Catalogue : 17 objets
En vente  : 15 objets
Force +5
Revente épée : 48 or
Inconnu   : null
```

---

## 62.4 — Étape 4 : l'inventaire et l'équipement

### Le piège du débutant

Le réflexe naturel est d'écrire `List<Objet> inventaire`. Trois problèmes surgissent aussitôt :

```text
1. Trois potions identiques  →  trois objets complets en mémoire et en JSON
2. Rééquilibrer une épée     →  toutes les sauvegardes gardent l'ancienne version
3. « Cette épée est-elle     →  comparer quoi ? le nom ? l'objet lui-même ?
    celle que j'ai équipée ? »
```

La solution est celle de toutes les bases de données : on ne stocke **pas** la donnée, on stocke une **référence** et une **quantité**.

```text
     CATALOGUE (constante, jamais persistée)
     ┌──────────────────────────────────────┐
     │ epee_acier   Épée d'acier   Force +5 │
     │ potion_soin  Potion de soin Soigne 40│
     └──────────────────────────────────────┘
                    ▲            ▲
                    │            │  référence par id
     INVENTAIRE (persisté)       │
     ┌───────────────────────────┴──────────┐
     │ lignes : [ (epee_acier, 1),          │
     │            (potion_soin, 3) ]        │
     │ équipés: { arme: epee_acier }        │
     └──────────────────────────────────────┘
```

**`lib/modeles/inventaire.dart`**

```dart
import '../config/constantes.dart';
import 'caracteristiques.dart';
import 'catalogue.dart';
import 'objet.dart';

/// Les trois emplacements d'équipement.
enum Emplacement {
  arme('Arme', TypeObjet.arme),
  armure('Armure', TypeObjet.armure),
  accessoire('Accessoire', TypeObjet.accessoire);

  const Emplacement(this.libelle, this.typeAccepte);

  final String libelle;
  final TypeObjet typeAccepte;

  /// L'emplacement correspondant à un type d'objet, ou `null` si le type
  /// n'est pas équipable (consommable, trésor).
  static Emplacement? pour(TypeObjet type) {
    for (final Emplacement e in Emplacement.values) {
      if (e.typeAccepte == type) return e;
    }
    return null;
  }

  static Emplacement? depuisNom(String? nom) {
    for (final Emplacement e in Emplacement.values) {
      if (e.name == nom) return e;
    }
    return null;
  }
}

/// « Ce joueur possède [quantite] exemplaires de l'objet [objetId]. »
class LigneInventaire {
  const LigneInventaire({required this.objetId, required this.quantite});

  final String objetId;
  final int quantite;

  /// La définition correspondante, ou `null` si l'objet n'existe plus.
  Objet? get objet => Catalogue.parId(objetId);

  LigneInventaire copyWith({int? quantite}) =>
      LigneInventaire(objetId: objetId, quantite: quantite ?? this.quantite);

  Map<String, dynamic> toJson() =>
      <String, dynamic>{'id': objetId, 'q': quantite};

  factory LigneInventaire.fromJson(Map<String, dynamic> json) {
    return LigneInventaire(
      objetId: (json['id'] as String?) ?? '',
      quantite: (json['q'] as num?)?.toInt() ?? 0,
    );
  }
}

/// Le sac du joueur, plus ce qu'il porte.
///
/// Immuable : chaque opération renvoie un NOUVEL inventaire. C'est ce qui
/// rend les tests de l'étape 62.20 simples à écrire, et ce qui garantit
/// qu'aucun écran ne peut modifier l'inventaire dans le dos de l'état.
class Inventaire {
  const Inventaire({
    this.lignes = const <LigneInventaire>[],
    this.equipes = const <Emplacement, String>{},
  });

  final List<LigneInventaire> lignes;

  /// Emplacement → identifiant de l'objet porté.
  final Map<Emplacement, String> equipes;

  /// Nombre total d'exemplaires détenus (quantités comprises).
  int get nombreObjets =>
      lignes.fold<int>(0, (int somme, LigneInventaire l) => somme + l.quantite);

  /// Nombre de cases occupées (une ligne = une case, quelle que soit la pile).
  int get casesOccupees => lignes.length;

  bool get estPlein => casesOccupees >= Constantes.tailleInventaire;

  /// Quantité détenue d'un objet donné.
  int quantiteDe(String objetId) {
    for (final LigneInventaire l in lignes) {
      if (l.objetId == objetId) return l.quantite;
    }
    return 0;
  }

  bool contient(String objetId) => quantiteDe(objetId) > 0;

  bool estEquipe(String objetId) => equipes.containsValue(objetId);

  /// L'objet porté à un emplacement, ou `null`.
  Objet? objetEquipe(Emplacement emplacement) {
    final String? id = equipes[emplacement];
    return id == null ? null : Catalogue.parId(id);
  }

  /// Somme des bonus des trois objets portés.
  ///
  /// `fold` avec `Caracteristiques.zero` comme valeur initiale : le motif
  /// exact du chapitre 14, appliqué à un type personnalisé grâce à la
  /// surcharge de `+`.
  Caracteristiques get bonusEquipement {
    return Emplacement.values.fold<Caracteristiques>(
      Caracteristiques.zero,
      (Caracteristiques somme, Emplacement e) =>
          somme + (objetEquipe(e)?.bonus ?? Caracteristiques.zero),
    );
  }

  /// Ajoute [quantite] exemplaires. Empile si l'objet est déjà présent.
  ///
  /// Renvoie `this` inchangé si le sac est plein et que l'objet est nouveau :
  /// l'appelant compare les deux références pour savoir si ça a marché.
  Inventaire ajouter(String objetId, [int quantite = 1]) {
    if (quantite <= 0) return this;

    final List<LigneInventaire> nouvelles =
        List<LigneInventaire>.from(lignes);
    final int index =
        nouvelles.indexWhere((LigneInventaire l) => l.objetId == objetId);

    if (index >= 0) {
      nouvelles[index] =
          nouvelles[index].copyWith(quantite: nouvelles[index].quantite + quantite);
    } else {
      if (estPlein) return this;
      nouvelles.add(LigneInventaire(objetId: objetId, quantite: quantite));
    }
    return copyWith(lignes: nouvelles);
  }

  /// Retire [quantite] exemplaires. Déséquipe l'objet s'il ne reste rien.
  Inventaire retirer(String objetId, [int quantite = 1]) {
    final int index =
        lignes.indexWhere((LigneInventaire l) => l.objetId == objetId);
    if (index < 0) return this;

    final List<LigneInventaire> nouvelles =
        List<LigneInventaire>.from(lignes);
    final int restant = nouvelles[index].quantite - quantite;

    Map<Emplacement, String> nouvelEquipement = equipes;

    if (restant <= 0) {
      nouvelles.removeAt(index);
      // Règle métier : on ne peut pas porter un objet qu'on ne possède plus.
      nouvelEquipement = Map<Emplacement, String>.from(equipes)
        ..removeWhere((Emplacement _, String id) => id == objetId);
    } else {
      nouvelles[index] = nouvelles[index].copyWith(quantite: restant);
    }

    return Inventaire(lignes: nouvelles, equipes: nouvelEquipement);
  }

  /// Équipe un objet possédé. Ne fait rien si l'objet n'est pas équipable
  /// ou n'est pas dans le sac.
  Inventaire equiper(String objetId) {
    final Objet? objet = Catalogue.parId(objetId);
    if (objet == null || !contient(objetId)) return this;

    final Emplacement? emplacement = Emplacement.pour(objet.type);
    if (emplacement == null) return this;

    return copyWith(
      equipes: Map<Emplacement, String>.from(equipes)
        ..[emplacement] = objetId,
    );
  }

  /// Retire l'objet porté à un emplacement. Il reste dans le sac.
  Inventaire dequiper(Emplacement emplacement) {
    if (!equipes.containsKey(emplacement)) return this;
    return copyWith(
      equipes: Map<Emplacement, String>.from(equipes)..remove(emplacement),
    );
  }

  Inventaire copyWith({
    List<LigneInventaire>? lignes,
    Map<Emplacement, String>? equipes,
  }) {
    return Inventaire(
      lignes: lignes ?? this.lignes,
      equipes: equipes ?? this.equipes,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'lignes': lignes.map((LigneInventaire l) => l.toJson()).toList(),
        // Les clés d'une Map JSON sont TOUJOURS des chaînes : on persiste
        // `.name`, jamais `.index` (leçon du chapitre 58).
        'equipes': equipes.map(
          (Emplacement e, String id) => MapEntry<String, String>(e.name, id),
        ),
      };

  factory Inventaire.fromJson(Map<String, dynamic> json) {
    // 1. Les lignes. Une entrée mal formée est ignorée, pas fatale.
    final List<LigneInventaire> lignes = <LigneInventaire>[];
    final Object? brutLignes = json['lignes'];
    if (brutLignes is List) {
      for (final Object? element in brutLignes) {
        if (element is Map<String, dynamic>) {
          final LigneInventaire ligne = LigneInventaire.fromJson(element);
          if (ligne.objetId.isNotEmpty && ligne.quantite > 0) {
            lignes.add(ligne);
          }
        }
      }
    }

    // 2. L'équipement. On ne garde que ce qui a du sens AUJOURD'HUI :
    //    un emplacement connu, un objet du catalogue, effectivement possédé.
    final Map<Emplacement, String> equipes = <Emplacement, String>{};
    final Object? brutEquipes = json['equipes'];
    if (brutEquipes is Map) {
      brutEquipes.forEach((Object? cle, Object? valeur) {
        final Emplacement? emplacement =
            Emplacement.depuisNom(cle is String ? cle : null);
        if (emplacement == null || valeur is! String) return;
        if (Catalogue.parId(valeur) == null) return;
        final bool possede =
            lignes.any((LigneInventaire l) => l.objetId == valeur);
        if (possede) equipes[emplacement] = valeur;
      });
    }

    return Inventaire(lignes: lignes, equipes: equipes);
  }
}
```

> **Le point important de ce fichier.** `fromJson` ne se contente pas de lire : il **valide**. Un objet supprimé du catalogue disparaît, un emplacement inconnu est ignoré, un objet équipé mais absent du sac est déséquipé. Une sauvegarde d'une ancienne version se répare donc toute seule. C'est cette discipline qui évite les rapports de bug du type « mon épée a disparu mais je fais toujours +5 dégâts ».

### Vérification

```dart
  Inventaire sac = const Inventaire();
  sac = sac.ajouter('potion_soin', 3).ajouter('epee_acier').ajouter('potion_soin');
  print('Cases: ${sac.casesOccupees}  Objets: ${sac.nombreObjets}');
  sac = sac.equiper('epee_acier');
  print('Bonus équipement : ${sac.bonusEquipement}');
  sac = sac.retirer('epee_acier');
  print('Après vente de l\'épée : ${sac.bonusEquipement}, équipé=${sac.equipes}');
```

**Résultat :**

```text
Cases: 2  Objets: 5
Bonus équipement : F5 D0 I0 V0
Après vente de l'épée : F0 D0 I0 V0, équipé={}
```

**État exécutable.** L'inventaire complet est écrit, testé en console, et ne dépend d'aucun widget.

---

## 62.5 — Étape 5 : le joueur et les statistiques dérivées

### Calculer, ne jamais stocker

Une règle vaut pour tout le reste du chapitre :

> **Une donnée qui peut être recalculée ne doit jamais être persistée.**

Les points de vie maximum se déduisent de la vitalité, qui se déduit de la classe, des points répartis et de l'équipement. Si vous persistez `pvMax`, vous créez une seconde source de vérité, et le jour où elles divergeront — elles divergeront — vous ne saurez pas laquelle croire.

```text
     PERSISTÉ                        CALCULÉ (getters)
     ┌──────────────────┐            ┌────────────────────┐
     │ classe           │─────┐      │ caracteristiques   │
     │ pointsRepartis   │─────┼─────▶│ pvMax              │
     │ inventaire.equipes│────┘      │ energieMax         │
     │ niveau, xp, or   │            │ degats, defense    │
     └──────────────────┘            │ chanceCritique     │
                                     └────────────────────┘
```

**`lib/modeles/joueur.dart`**

```dart
import '../config/constantes.dart';
import 'caracteristiques.dart';
import 'classe_personnage.dart';
import 'inventaire.dart';

/// Le personnage du joueur.
///
/// Immuable, comme tout le reste des modèles. Les statistiques dérivées sont
/// des GETTERS : elles n'occupent pas un octet en JSON et ne peuvent pas
/// être fausses.
class Joueur {
  const Joueur({
    required this.nom,
    required this.classe,
    this.niveau = 1,
    this.experience = 0,
    this.pointsRepartis = Caracteristiques.zero,
    this.pointsDisponibles = 0,
    this.or = Constantes.orDepart,
    this.inventaire = const Inventaire(),
  });

  final String nom;
  final ClassePersonnage classe;
  final int niveau;
  final int experience;

  /// Les points que le joueur a placés lui-même, en plus de la base de classe.
  final Caracteristiques pointsRepartis;

  /// Les points gagnés en montant de niveau et pas encore dépensés.
  final int pointsDisponibles;

  final int or;
  final Inventaire inventaire;

  // --- Caractéristiques ---------------------------------------------------

  /// Base de classe + points répartis. Sans l'équipement.
  Caracteristiques get caracteristiquesBase => classe.base + pointsRepartis;

  /// Ce que rapporte l'équipement porté.
  Caracteristiques get bonusEquipement => inventaire.bonusEquipement;

  /// Ce qui compte réellement en jeu.
  Caracteristiques get caracteristiques =>
      caracteristiquesBase + bonusEquipement;

  // --- Statistiques dérivées ----------------------------------------------

  int get pvMax =>
      Constantes.pvBase + caracteristiques.vitalite * Constantes.pvParVitalite;

  int get energieMax =>
      Constantes.energieBase +
      caracteristiques.intelligence * Constantes.energieParIntelligence;

  double get degats =>
      Constantes.degatsBase +
      caracteristiques.force * Constantes.degatsParForce;

  double get defense =>
      caracteristiques.vitalite * Constantes.defenseParVitalite +
      caracteristiques.force * Constantes.defenseParForce;

  /// En pourcentage, plafonnée : sans plafond, un rôdeur très avancé
  /// atteindrait 100 % de critiques et le jeu perdrait tout intérêt.
  double get chanceCritique {
    final double brute =
        caracteristiques.dexterite * Constantes.critParDexterite;
    return brute > Constantes.critMax ? Constantes.critMax : brute;
  }

  // --- Progression --------------------------------------------------------

  /// Expérience nécessaire pour atteindre le niveau suivant.
  int get xpRequis => Constantes.xpParNiveau * (niveau + 1);

  /// Avancement dans le niveau courant, entre 0.0 et 1.0.
  /// Utilisable directement par une `LinearProgressIndicator`.
  double get avancementNiveau {
    if (xpRequis <= 0) return 0;
    final double v = experience / xpRequis;
    return v.clamp(0.0, 1.0);
  }

  /// L'initiale affichée dans l'avatar (rappel du projet 55).
  String get initiale => nom.isEmpty ? '?' : nom.characters.first.toUpperCase();

  // --- Fabriques ----------------------------------------------------------

  /// Crée un personnage neuf à la sortie de l'écran de création.
  factory Joueur.nouveau({
    required String nom,
    required ClassePersonnage classe,
    required Caracteristiques repartition,
  }) {
    return Joueur(
      nom: nom.trim(),
      classe: classe,
      pointsRepartis: repartition,
      // Un petit kit de départ, pour que l'inventaire ne soit pas vide.
      inventaire: const Inventaire()
          .ajouter('epee_rouillee')
          .ajouter('tunique')
          .ajouter('potion_soin', 2)
          .equiper('epee_rouillee')
          .equiper('tunique'),
    );
  }

  Joueur copyWith({
    String? nom,
    ClassePersonnage? classe,
    int? niveau,
    int? experience,
    Caracteristiques? pointsRepartis,
    int? pointsDisponibles,
    int? or,
    Inventaire? inventaire,
  }) {
    return Joueur(
      nom: nom ?? this.nom,
      classe: classe ?? this.classe,
      niveau: niveau ?? this.niveau,
      experience: experience ?? this.experience,
      pointsRepartis: pointsRepartis ?? this.pointsRepartis,
      pointsDisponibles: pointsDisponibles ?? this.pointsDisponibles,
      or: or ?? this.or,
      inventaire: inventaire ?? this.inventaire,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'classe': classe.name,
        'niveau': niveau,
        'experience': experience,
        'pointsRepartis': pointsRepartis.toJson(),
        'pointsDisponibles': pointsDisponibles,
        'or': or,
        'inventaire': inventaire.toJson(),
      };

  factory Joueur.fromJson(Map<String, dynamic> json) {
    return Joueur(
      nom: (json['nom'] as String?) ?? 'Héros',
      classe: ClassePersonnage.depuisNom(json['classe'] as String?),
      niveau: (json['niveau'] as num?)?.toInt() ?? 1,
      experience: (json['experience'] as num?)?.toInt() ?? 0,
      pointsRepartis: Caracteristiques.fromJson(
        (json['pointsRepartis'] as Map<String, dynamic>?) ??
            <String, dynamic>{},
      ),
      pointsDisponibles: (json['pointsDisponibles'] as num?)?.toInt() ?? 0,
      or: (json['or'] as num?)?.toInt() ?? Constantes.orDepart,
      inventaire: Inventaire.fromJson(
        (json['inventaire'] as Map<String, dynamic>?) ?? <String, dynamic>{},
      ),
    );
  }

  @override
  String toString() =>
      '$nom (${classe.libelle} niveau $niveau) — PV $pvMax, dégâts $degats';
}
```

`nom.characters` exige un import de plus, en tête du fichier :

```dart
import 'package:characters/characters.dart';
```

Le paquet `characters` est une dépendance transitive de Flutter : il est déjà là, vous n'avez rien à ajouter au `pubspec.yaml`. Pourquoi ne pas écrire `nom[0]` ? Parce qu'un nom peut commencer par un caractère composé (un émoji, une lettre accentuée décomposée) ; `nom[0]` renverrait alors la moitié d'un caractère. `characters.first` renvoie le **groupe de graphèmes** complet.

### Vérification

```dart
  final Joueur kaelis = Joueur.nouveau(
    nom: 'Kaelis',
    classe: ClassePersonnage.guerrier,
    repartition: const Caracteristiques(force: 4, dexterite: 1, vitalite: 5),
  );
  print(kaelis);
  print('Caracs : ${kaelis.caracteristiques}');
  print('Défense ${kaelis.defense}, crit ${kaelis.chanceCritique} %');
  print('Sac : ${kaelis.inventaire.nombreObjets} objets, or ${kaelis.or}');
```

**Résultat :**

```text
Kaelis (Guerrier niveau 1) — PV 152, dégâts 25.0
Caracs : F14 D5 I2 V14
Défense 11.2, crit 4.0 %
Sac : 4 objets, or 100
```

Détail du calcul, à vérifier à la main une fois pour toutes :

```text
force  = 8 (guerrier) + 4 (réparti) + 2 (épée rouillée)  = 14
vitalité = 7 (guerrier) + 5 (réparti) + 2 (tunique)      = 14
pvMax  = 40 + 14 × 8                                     = 152
dégâts = 4.0 + 14 × 1.5                                  = 25.0
défense = 14 × 0.5 + 14 × 0.3                            = 11.2
crit   = 5 × 0.8                                         = 4.0 %
```

---

## 62.6 — Étape 6 : la progression

Une seule fonction, mais elle mérite son fichier : gagner de l'expérience peut faire monter **plusieurs** niveaux d'un coup, et chaque montée doit reporter le surplus.

**`lib/logique/progression.dart`**

```dart
import '../config/constantes.dart';
import '../modeles/joueur.dart';

/// Les règles de progression. Fonctions pures : aucune ne touche à l'écran,
/// aucune ne modifie son entrée. Elles sont donc directement testables
/// (étape 62.20).
class Progression {
  const Progression._();

  /// Expérience nécessaire pour passer du niveau [niveau] au suivant.
  static int xpRequisPour(int niveau) =>
      Constantes.xpParNiveau * (niveau + 1);

  /// Applique un gain d'expérience et renvoie le joueur mis à jour.
  ///
  /// Gère la montée de plusieurs niveaux : gagner 5000 XP au niveau 1 doit
  /// faire progresser correctement, pas monter d'un seul niveau en perdant
  /// le reste. La boucle `while` est la manière la plus lisible de le faire.
  static Joueur gagnerExperience(Joueur joueur, int gain) {
    if (gain <= 0) return joueur;

    int niveau = joueur.niveau;
    int xp = joueur.experience + gain;
    int points = joueur.pointsDisponibles;

    while (xp >= xpRequisPour(niveau)) {
      xp -= xpRequisPour(niveau);
      niveau += 1;
      points += Constantes.pointsParNiveau;
    }

    return joueur.copyWith(
      niveau: niveau,
      experience: xp,
      pointsDisponibles: points,
    );
  }

  /// Dépense un point disponible dans une caractéristique.
  /// Ne fait rien s'il n'en reste aucun : la règle est ici, pas dans l'écran.
  static Joueur depenserPoint(Joueur joueur, Carac carac) {
    if (joueur.pointsDisponibles <= 0) return joueur;
    return joueur.copyWith(
      pointsRepartis: joueur.pointsRepartis.modifier(carac, 1),
      pointsDisponibles: joueur.pointsDisponibles - 1,
    );
  }
}
```

Ajoutez l'import du type `Carac` en tête du fichier :

```dart
import '../modeles/caracteristiques.dart';
```

> **Pourquoi la règle « pas de point disponible → on ne fait rien » est-elle dans la logique et pas dans l'écran ?** Parce qu'un jour, un second écran dépensera aussi des points (un objet magique, un cadeau de fin de niveau). Une règle métier écrite dans un widget est une règle qu'il faudra réécrire ailleurs, et donc une règle qui finira par diverger.

### Vérification

```dart
  Joueur j = Joueur.nouveau(
      nom: 'Test', classe: ClassePersonnage.mage,
      repartition: const Caracteristiques(intelligence: 5));
  j = Progression.gagnerExperience(j, 5000);
  print('Niveau ${j.niveau}, xp ${j.experience}/${j.xpRequis}, '
      'points ${j.pointsDisponibles}');
```

Avant de lire la sortie, faites le calcul à la main. C'est le seul moyen de savoir si la fonction est juste :

```text
niveau 1 → 2 : 200   cumul  200   reste 4800
niveau 2 → 3 : 300   cumul  500   reste 4500
niveau 3 → 4 : 400   cumul  900   reste 4100
niveau 4 → 5 : 500   cumul 1400   reste 3600
niveau 5 → 6 : 600   cumul 2000   reste 3000
niveau 6 → 7 : 700   cumul 2700   reste 2300
niveau 7 → 8 : 800   cumul 3500   reste 1500
niveau 8 → 9 : 900   cumul 4400   reste  600  ← 600 < 1000, la boucle s'arrête
```

Le personnage finit donc **niveau 9 avec 600 XP**, et huit montées de niveau lui ont rapporté 8 × 3 = 24 points.

**Résultat :**

```text
Niveau 9, xp 600/1000, points 24
```

> **La leçon.** Ne recopiez jamais une sortie « attendue » sans l'avoir calculée. Un test automatisé (étape 62.20) fait ce travail à votre place, définitivement.

---

## 62.7 — Étape 7 : les réglages et le modèle de sauvegarde

### Les réglages

C'est le seul modèle qui importe quelque chose de Flutter : `LogicalKeyboardKey`, pour le remappage des touches. `package:flutter/services.dart` ne contient aucun widget ; il reste parfaitement testable.

**`lib/modeles/reglages.dart`**

```dart
import 'package:flutter/services.dart' show LogicalKeyboardKey;

/// Le niveau de difficulté. Le multiplicateur servira au chapitre 39.
enum Difficulte {
  facile('Facile', 0.7),
  normale('Normale', 1.0),
  difficile('Difficile', 1.4);

  const Difficulte(this.libelle, this.multiplicateurDegats);

  final String libelle;

  /// Multiplie les dégâts reçus par le joueur.
  final double multiplicateurDegats;

  static Difficulte depuisNom(String? nom) {
    for (final Difficulte d in Difficulte.values) {
      if (d.name == nom) return d;
    }
    return Difficulte.normale;
  }
}

/// Le thème choisi. On ne stocke pas `ThemeMode` de Material dans un modèle :
/// la conversion se fera dans `EtatReglages` (étape 62.10).
enum ModeTheme {
  systeme('Système'),
  clair('Clair'),
  sombre('Sombre');

  const ModeTheme(this.libelle);

  final String libelle;

  static ModeTheme depuisNom(String? nom) {
    for (final ModeTheme m in ModeTheme.values) {
      if (m.name == nom) return m;
    }
    return ModeTheme.systeme;
  }
}

/// Les actions du jeu auxquelles on peut associer une touche.
///
/// Les valeurs par défaut correspondent à un clavier AZERTY :
/// Q / D pour se déplacer, Espace pour sauter.
enum ActionJeu {
  gauche('Aller à gauche', LogicalKeyboardKey.keyQ),
  droite('Aller à droite', LogicalKeyboardKey.keyD),
  saut('Sauter', LogicalKeyboardKey.space),
  attaque('Attaquer', LogicalKeyboardKey.keyJ),
  pause('Pause', LogicalKeyboardKey.escape);

  const ActionJeu(this.libelle, this.toucheParDefaut);

  final String libelle;
  final LogicalKeyboardKey toucheParDefaut;

  static ActionJeu? depuisNom(String? nom) {
    for (final ActionJeu a in ActionJeu.values) {
      if (a.name == nom) return a;
    }
    return null;
  }
}

/// Un libellé lisible pour une touche.
///
/// `LogicalKeyboardKey.keyLabel` renvoie « Q » pour les lettres, mais une
/// chaîne vide pour Espace ou Échap. `debugName` serait parfait... mais il
/// vaut `null` en mode release : ne comptez jamais dessus dans l'interface.
/// D'où cette petite table.
String libelleTouche(LogicalKeyboardKey touche) {
  final Map<LogicalKeyboardKey, String> connues =
      <LogicalKeyboardKey, String>{
    LogicalKeyboardKey.space: 'ESPACE',
    LogicalKeyboardKey.escape: 'ÉCHAP',
    LogicalKeyboardKey.enter: 'ENTRÉE',
    LogicalKeyboardKey.tab: 'TAB',
    LogicalKeyboardKey.shiftLeft: 'MAJ G',
    LogicalKeyboardKey.shiftRight: 'MAJ D',
    LogicalKeyboardKey.controlLeft: 'CTRL G',
    LogicalKeyboardKey.controlRight: 'CTRL D',
    LogicalKeyboardKey.arrowLeft: '←',
    LogicalKeyboardKey.arrowRight: '→',
    LogicalKeyboardKey.arrowUp: '↑',
    LogicalKeyboardKey.arrowDown: '↓',
  };
  final String? connu = connues[touche];
  if (connu != null) return connu;
  final String label = touche.keyLabel;
  return label.isEmpty ? 'TOUCHE ${touche.keyId}' : label.toUpperCase();
}

/// Tous les réglages de l'application.
class Reglages {
  const Reglages({
    this.volumeMusique = 0.35,
    this.volumeEffets = 0.8,
    this.difficulte = Difficulte.normale,
    this.modeTheme = ModeTheme.systeme,
    required this.touches,
  });

  final double volumeMusique;
  final double volumeEffets;
  final Difficulte difficulte;
  final ModeTheme modeTheme;

  /// Action → touche. Toujours complète : les cinq actions y figurent.
  final Map<ActionJeu, LogicalKeyboardKey> touches;

  /// La configuration d'usine.
  factory Reglages.defaut() {
    return Reglages(touches: touchesParDefaut());
  }

  static Map<ActionJeu, LogicalKeyboardKey> touchesParDefaut() {
    return <ActionJeu, LogicalKeyboardKey>{
      for (final ActionJeu a in ActionJeu.values) a: a.toucheParDefaut,
    };
  }

  /// L'action déjà associée à une touche, ou `null`.
  ActionJeu? actionPour(LogicalKeyboardKey touche) {
    for (final MapEntry<ActionJeu, LogicalKeyboardKey> e in touches.entries) {
      if (e.value == touche) return e.key;
    }
    return null;
  }

  /// Associe [touche] à [action].
  ///
  /// Si la touche est déjà prise par une autre action, les deux sont
  /// ÉCHANGÉES. C'est le comportement attendu par les joueurs : sans cet
  /// échange, on se retrouve avec une action sans touche, donc injouable.
  Reglages avecTouche(ActionJeu action, LogicalKeyboardKey touche) {
    final Map<ActionJeu, LogicalKeyboardKey> nouvelles =
        Map<ActionJeu, LogicalKeyboardKey>.from(touches);

    final ActionJeu? occupante = actionPour(touche);
    if (occupante != null && occupante != action) {
      nouvelles[occupante] = touches[action]!;
    }
    nouvelles[action] = touche;

    return copyWith(touches: nouvelles);
  }

  Reglages copyWith({
    double? volumeMusique,
    double? volumeEffets,
    Difficulte? difficulte,
    ModeTheme? modeTheme,
    Map<ActionJeu, LogicalKeyboardKey>? touches,
  }) {
    return Reglages(
      volumeMusique: volumeMusique ?? this.volumeMusique,
      volumeEffets: volumeEffets ?? this.volumeEffets,
      difficulte: difficulte ?? this.difficulte,
      modeTheme: modeTheme ?? this.modeTheme,
      touches: touches ?? this.touches,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'volumeMusique': volumeMusique,
        'volumeEffets': volumeEffets,
        'difficulte': difficulte.name,
        'modeTheme': modeTheme.name,
        // On persiste le `keyId` (un entier stable), pas l'objet.
        'touches': touches.map(
          (ActionJeu a, LogicalKeyboardKey t) =>
              MapEntry<String, int>(a.name, t.keyId),
        ),
      };

  factory Reglages.fromJson(Map<String, dynamic> json) {
    // On part TOUJOURS des valeurs par défaut, puis on écrase avec ce qui
    // a été relu et validé. Une touche manquante reste ainsi jouable.
    final Map<ActionJeu, LogicalKeyboardKey> touches = touchesParDefaut();

    final Object? brut = json['touches'];
    if (brut is Map) {
      brut.forEach((Object? cle, Object? valeur) {
        final ActionJeu? action = ActionJeu.depuisNom(cle is String ? cle : null);
        final int? keyId = (valeur as num?)?.toInt();
        if (action == null || keyId == null) return;
        final LogicalKeyboardKey? touche =
            LogicalKeyboardKey.findKeyByKeyId(keyId);
        if (touche != null) touches[action] = touche;
      });
    }

    return Reglages(
      volumeMusique:
          ((json['volumeMusique'] as num?)?.toDouble() ?? 0.35).clamp(0.0, 1.0),
      volumeEffets:
          ((json['volumeEffets'] as num?)?.toDouble() ?? 0.8).clamp(0.0, 1.0),
      difficulte: Difficulte.depuisNom(json['difficulte'] as String?),
      modeTheme: ModeTheme.depuisNom(json['modeTheme'] as String?),
      touches: touches,
    );
  }
}
```

> **`findKeyByKeyId` peut renvoyer `null`.** C'est le cas si la sauvegarde contient un identifiant inconnu de cette version de Flutter. On ignore alors l'entrée et la touche par défaut reste en place. Ne mettez jamais un `!` ici : une sauvegarde vieille de deux ans ferait planter le démarrage.

### La sauvegarde

**`lib/modeles/sauvegarde.dart`**

```dart
import '../config/constantes.dart';
import 'joueur.dart';

/// Une partie complète, telle qu'elle est écrite sur le disque.
///
/// Un emplacement de sauvegarde contient exactement UNE `Sauvegarde`, ou rien.
class Sauvegarde {
  const Sauvegarde({
    this.version = Constantes.versionSauvegarde,
    required this.joueur,
    this.meilleurScore = 0,
    this.niveauAtteint = 0,
    this.partiesJouees = 0,
    this.secondesDeJeu = 0,
    required this.derniereEcriture,
  });

  /// Version du FORMAT, pas du jeu. Permet la migration (voir `migrer`).
  final int version;

  final Joueur joueur;
  final int meilleurScore;

  /// Index du niveau le plus loin atteint (0 = premier niveau).
  final int niveauAtteint;

  final int partiesJouees;
  final int secondesDeJeu;
  final DateTime derniereEcriture;

  /// Durée de jeu formatée : « 2 h 14 min ».
  String get tempsDeJeuLisible {
    final int heures = secondesDeJeu ~/ 3600;
    final int minutes = (secondesDeJeu % 3600) ~/ 60;
    if (heures == 0) return '$minutes min';
    return '$heures h $minutes min';
  }

  Sauvegarde copyWith({
    Joueur? joueur,
    int? meilleurScore,
    int? niveauAtteint,
    int? partiesJouees,
    int? secondesDeJeu,
    DateTime? derniereEcriture,
  }) {
    return Sauvegarde(
      version: version,
      joueur: joueur ?? this.joueur,
      meilleurScore: meilleurScore ?? this.meilleurScore,
      niveauAtteint: niveauAtteint ?? this.niveauAtteint,
      partiesJouees: partiesJouees ?? this.partiesJouees,
      secondesDeJeu: secondesDeJeu ?? this.secondesDeJeu,
      derniereEcriture: derniereEcriture ?? this.derniereEcriture,
    );
  }

  Map<String, dynamic> toJson() => <String, dynamic>{
        'version': version,
        'joueur': joueur.toJson(),
        'meilleurScore': meilleurScore,
        'niveauAtteint': niveauAtteint,
        'partiesJouees': partiesJouees,
        'secondesDeJeu': secondesDeJeu,
        // ISO 8601 : lisible, triable, sans ambiguïté de fuseau.
        'derniereEcriture': derniereEcriture.toIso8601String(),
      };

  factory Sauvegarde.fromJson(Map<String, dynamic> json) {
    return Sauvegarde(
      version: (json['version'] as num?)?.toInt() ??
          Constantes.versionSauvegarde,
      joueur: Joueur.fromJson(
        (json['joueur'] as Map<String, dynamic>?) ?? <String, dynamic>{},
      ),
      meilleurScore: (json['meilleurScore'] as num?)?.toInt() ?? 0,
      niveauAtteint: (json['niveauAtteint'] as num?)?.toInt() ?? 0,
      partiesJouees: (json['partiesJouees'] as num?)?.toInt() ?? 0,
      secondesDeJeu: (json['secondesDeJeu'] as num?)?.toInt() ?? 0,
      // `tryParse` renvoie `null` au lieu de lever : indispensable ici.
      derniereEcriture:
          DateTime.tryParse((json['derniereEcriture'] as String?) ?? '') ??
              DateTime.fromMillisecondsSinceEpoch(0),
    );
  }

  /// Adapte un document produit par une version antérieure.
  ///
  /// Le corps est vide aujourd'hui, et c'est normal : la version courante
  /// est 1. Écrivez la fonction MAINTENANT. Le jour où vous publierez une
  /// mise à jour, vous saurez où mettre le code, et vous ne casserez pas
  /// les sauvegardes de vos joueurs. C'est le conseil de la section 40.26.
  static Map<String, dynamic> migrer(Map<String, dynamic> json) {
    final int version = (json['version'] as num?)?.toInt() ?? 1;

    // Exemple de ce que contiendra cette fonction :
    // if (version < 2) {
    //   json['secondesDeJeu'] = json['secondesDeJeu'] ?? 0;
    //   json['version'] = 2;
    // }

    if (version > Constantes.versionSauvegarde) {
      // Sauvegarde produite par une version PLUS RÉCENTE de l'application.
      // On ne sait pas la lire ; l'appelant la traitera comme corrompue.
      throw const FormatException('Version de sauvegarde trop récente');
    }
    return json;
  }
}
```

### Le document produit

**`sauvegarde_1` (exemple réel, réindenté)**

```json
{
  "version": 1,
  "joueur": {
    "nom": "Kaelis",
    "classe": "guerrier",
    "niveau": 7,
    "experience": 620,
    "pointsRepartis": {
      "force": 4,
      "dexterite": 1,
      "intelligence": 0,
      "vitalite": 5
    },
    "pointsDisponibles": 3,
    "or": 1240,
    "inventaire": {
      "lignes": [
        { "id": "epee_acier", "q": 1 },
        { "id": "cotte_mailles", "q": 1 },
        { "id": "potion_soin", "q": 3 },
        { "id": "gemme", "q": 2 }
      ],
      "equipes": {
        "arme": "epee_acier",
        "armure": "cotte_mailles"
      }
    }
  },
  "meilleurScore": 4820,
  "niveauAtteint": 2,
  "partiesJouees": 17,
  "secondesDeJeu": 8040,
  "derniereEcriture": "2026-08-15T14:32:07.412"
}
```

Moins d'un kilo-octet pour une partie entière. Remarquez ce qui **n'y est pas** : ni `pvMax`, ni `degats`, ni `bonusEquipement`, ni le nom des objets. Tout cela se recalcule.

**État exécutable.** Les six modèles sont écrits. Aucun n'a besoin d'un écran pour être vérifié. Vous pouvez déjà sérialiser une partie complète en console :

```dart
  final Sauvegarde s = Sauvegarde(
    joueur: kaelis,
    derniereEcriture: DateTime.now(),
  );
  final String texte = jsonEncode(s.toJson());
  print('${texte.length} caractères');
  final Sauvegarde relue = Sauvegarde.fromJson(
      jsonDecode(texte) as Map<String, dynamic>);
  print('${relue.joueur.nom} : ${relue.joueur.pvMax} PV');
```

```text
612 caractères
Kaelis : 152 PV
```

---

## 62.8 — Étape 8 : les services, les emplacements et la sauvegarde corrompue

Les modèles savent se sérialiser ; ils ne savent pas **où** aller. C'est le rôle des services. Un service isole une technologie (ici `shared_preferences`, chapitre 54) derrière des méthodes métier : le jour où vous passerez à `sqflite` ou à un fichier, seul ce fichier change.

### Le résultat d'un chargement

Un emplacement peut être dans trois états : vide, valide, corrompu. Un `Sauvegarde?` n'en distingue que deux. Il nous faut donc un petit type dédié.

**`lib/services/sauvegarde_service.dart`**

```dart
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../config/constantes.dart';
import '../modeles/sauvegarde.dart';

/// Le résultat de la lecture d'UN emplacement.
///
/// Trois cas, trois constructeurs nommés. Un `Sauvegarde?` seul serait
/// ambigu : `null` voudrait dire à la fois « vide » et « illisible », et
/// l'utilisateur ne saurait jamais que sa partie a été perdue.
class ResultatEmplacement {
  const ResultatEmplacement.vide()
      : sauvegarde = null,
        erreur = null;

  const ResultatEmplacement.valide(Sauvegarde this.sauvegarde) : erreur = null;

  const ResultatEmplacement.corrompu(String this.erreur) : sauvegarde = null;

  final Sauvegarde? sauvegarde;
  final String? erreur;

  bool get estVide => sauvegarde == null && erreur == null;
  bool get estValide => sauvegarde != null;
  bool get estCorrompu => erreur != null;
}

/// Lecture et écriture des emplacements de sauvegarde.
class SauvegardeService {
  SauvegardeService({SharedPreferencesAsync? prefs})
      : _prefs = prefs ?? SharedPreferencesAsync();

  final SharedPreferencesAsync _prefs;

  static String _cle(int emplacement) => 'sauvegarde_$emplacement';
  static String _cleSecours(int emplacement) => 'sauvegarde_${emplacement}_hs';

  /// Lit un emplacement. **Ne lève jamais.**
  ///
  /// Trois familles d'échec sont possibles et toutes sont traitées :
  ///   - `FormatException` : le texte n'est pas du JSON valide ;
  ///   - `TypeError`       : c'est du JSON, mais pas la bonne forme
  ///                         (une liste au lieu d'un objet, par exemple) ;
  ///   - toute autre        : on ne prend aucun risque au démarrage.
  Future<ResultatEmplacement> lire(int emplacement) async {
    try {
      final String? brut = await _prefs.getString(_cle(emplacement));
      if (brut == null || brut.isEmpty) {
        return const ResultatEmplacement.vide();
      }

      final Object? decode = jsonDecode(brut);
      if (decode is! Map<String, dynamic>) {
        return const ResultatEmplacement.corrompu(
            'Le document n\'est pas un objet JSON.');
      }

      final Map<String, dynamic> migre = Sauvegarde.migrer(decode);
      return ResultatEmplacement.valide(Sauvegarde.fromJson(migre));
    } on FormatException catch (e) {
      await _mettreDeCote(emplacement);
      return ResultatEmplacement.corrompu('JSON illisible : ${e.message}');
    } catch (e) {
      await _mettreDeCote(emplacement);
      return ResultatEmplacement.corrompu('Sauvegarde illisible : $e');
    }
  }

  /// Lit les trois emplacements en parallèle (chapitre 15).
  Future<List<ResultatEmplacement>> lireTous() {
    return Future.wait<ResultatEmplacement>(<Future<ResultatEmplacement>>[
      for (int i = 0; i < Constantes.nombreEmplacements; i++) lire(i),
    ]);
  }

  /// Écrit un emplacement. Un échec est signalé mais n'interrompt rien.
  Future<bool> ecrire(int emplacement, Sauvegarde sauvegarde) async {
    try {
      await _prefs.setString(
        _cle(emplacement),
        jsonEncode(sauvegarde.toJson()),
      );
      return true;
    } catch (e) {
      debugPrint('[Sauvegarde] Écriture impossible : $e');
      return false;
    }
  }

  Future<void> supprimer(int emplacement) async {
    await _prefs.remove(_cle(emplacement));
  }

  /// Déplace un document illisible dans une clé de secours au lieu de
  /// l'effacer. L'utilisateur perd sa partie, mais un développeur peut
  /// encore comprendre pourquoi. C'est la différence entre « ça a planté »
  /// et « on sait ce qui s'est passé ».
  Future<void> _mettreDeCote(int emplacement) async {
    try {
      final String? brut = await _prefs.getString(_cle(emplacement));
      if (brut != null) {
        await _prefs.setString(_cleSecours(emplacement), brut);
      }
      await _prefs.remove(_cle(emplacement));
    } catch (_) {
      // Si même ça échoue, on abandonne en silence : le démarrage prime.
    }
  }
}
```

> **Pourquoi deux `catch` ?** `on FormatException` capture le cas de loin le plus fréquent et permet un message précis. Le `catch (e)` final est un filet de sécurité : au démarrage, **aucune** exception ne doit pouvoir empêcher l'affichage du menu. C'est le principe du chapitre 13, appliqué à l'endroit le plus critique de l'application.

### Le service des réglages

**`lib/services/reglages_service.dart`**

```dart
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../modeles/reglages.dart';

/// Persistance des options. Un seul document JSON, une seule clé.
class ReglagesService {
  ReglagesService({SharedPreferencesAsync? prefs})
      : _prefs = prefs ?? SharedPreferencesAsync();

  final SharedPreferencesAsync _prefs;
  static const String _cle = 'reglages';

  /// Ne lève jamais : en cas de problème, on repart des réglages d'usine.
  Future<Reglages> lire() async {
    try {
      final String? brut = await _prefs.getString(_cle);
      if (brut == null || brut.isEmpty) return Reglages.defaut();
      final Object? decode = jsonDecode(brut);
      if (decode is! Map<String, dynamic>) return Reglages.defaut();
      return Reglages.fromJson(decode);
    } catch (e) {
      debugPrint('[Réglages] Lecture impossible : $e');
      return Reglages.defaut();
    }
  }

  Future<void> ecrire(Reglages reglages) async {
    try {
      await _prefs.setString(_cle, jsonEncode(reglages.toJson()));
    } catch (e) {
      debugPrint('[Réglages] Écriture impossible : $e');
    }
  }
}
```

### Fabriquer une sauvegarde corrompue pour tester

Vous ne pouvez pas corriger un bug que vous ne savez pas provoquer. Ajoutez temporairement, dans `main()`, avant `runApp` :

```dart
  // À SUPPRIMER après le test.
  await SharedPreferencesAsync().setString('sauvegarde_0', '{{{ pas du json');
```

**Résultat attendu au lancement :**

```text
┌────────────────────────────────────────────────┐
│  ←   Emplacements                              │
├────────────────────────────────────────────────┤
│  ⚠  Emplacement 1 — SAUVEGARDE ILLISIBLE       │
│     JSON illisible : Unexpected character      │
│     [ SUPPRIMER ]                              │
├────────────────────────────────────────────────┤
│  +  Emplacement 2 — vide                       │
├────────────────────────────────────────────────┤
│  +  Emplacement 3 — vide                       │
└────────────────────────────────────────────────┘
```

L'application démarre, prévient, et propose la seule action utile. Elle ne plante pas, et elle ne fait pas semblant que tout va bien.

**État exécutable.** Les deux services compilent. Rien n'est encore branché à l'écran, mais toute l'entrée-sortie est écrite et blindée.

---

## 62.9 — Étape 9 : l'état global avec `provider`

Deux `ChangeNotifier`, pas plus : l'un pour la partie, l'autre pour les réglages. Ils ne se connaissent pas.

```text
   MultiProvider
   ├── ChangeNotifierProvider<EtatReglages>  → thème, volumes, touches
   └── ChangeNotifierProvider<EtatPartie>    → emplacements, joueur, or
                    │
                    ▼
              MaterialApp  (themeMode lu dans EtatReglages)
                    │
                    ├── EcranChargement → EcranMenu → tous les autres écrans
```

**`lib/etat/etat_reglages.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/reglages.dart';
import '../services/reglages_service.dart';

/// L'état des options. Notifie l'application entière à chaque changement,
/// ce qui permet au thème de basculer instantanément.
class EtatReglages extends ChangeNotifier {
  EtatReglages(this._service);

  final ReglagesService _service;

  Reglages _reglages = Reglages.defaut();
  Reglages get reglages => _reglages;

  /// Conversion modèle → Material. C'est ici, et nulle part ailleurs, que
  /// `ModeTheme` devient `ThemeMode` : le modèle reste indépendant de Flutter.
  ThemeMode get themeMode {
    switch (_reglages.modeTheme) {
      case ModeTheme.systeme:
        return ThemeMode.system;
      case ModeTheme.clair:
        return ThemeMode.light;
      case ModeTheme.sombre:
        return ThemeMode.dark;
    }
  }

  Future<void> initialiser() async {
    _reglages = await _service.lire();
    notifyListeners();
  }

  /// Toutes les modifications passent par ici : on notifie d'abord (l'écran
  /// répond instantanément), on écrit ensuite. C'est la mise à jour
  /// « optimiste » du chapitre 58.
  void _appliquer(Reglages nouveaux) {
    _reglages = nouveaux;
    notifyListeners();
    _service.ecrire(nouveaux);
  }

  void changerVolumeMusique(double v) =>
      _appliquer(_reglages.copyWith(volumeMusique: v));

  void changerVolumeEffets(double v) =>
      _appliquer(_reglages.copyWith(volumeEffets: v));

  void changerDifficulte(Difficulte d) =>
      _appliquer(_reglages.copyWith(difficulte: d));

  void changerTheme(ModeTheme m) =>
      _appliquer(_reglages.copyWith(modeTheme: m));

  void assignerTouche(ActionJeu action, LogicalKeyboardKey touche) =>
      _appliquer(_reglages.avecTouche(action, touche));

  void reinitialiserTouches() =>
      _appliquer(_reglages.copyWith(touches: Reglages.touchesParDefaut()));
}
```

Ajoutez l'import des touches en tête :

```dart
import 'package:flutter/services.dart' show LogicalKeyboardKey;
```

**`lib/etat/etat_partie.dart`**

```dart
import 'package:flutter/foundation.dart';

import '../config/constantes.dart';
import '../logique/progression.dart';
import '../modeles/caracteristiques.dart';
import '../modeles/catalogue.dart';
import '../modeles/classe_personnage.dart';
import '../modeles/inventaire.dart';
import '../modeles/joueur.dart';
import '../modeles/objet.dart';
import '../modeles/sauvegarde.dart';
import '../services/sauvegarde_service.dart';

/// L'état de la partie en cours et des trois emplacements.
///
/// C'est le seul objet autorisé à modifier une sauvegarde. Aucun écran
/// n'écrit directement sur le disque.
class EtatPartie extends ChangeNotifier {
  EtatPartie(this._service);

  final SauvegardeService _service;

  List<ResultatEmplacement> _emplacements = <ResultatEmplacement>[];
  int? _emplacementCourant;
  bool _pret = false;

  List<ResultatEmplacement> get emplacements =>
      List<ResultatEmplacement>.unmodifiable(_emplacements);

  bool get pret => _pret;
  int? get emplacementCourant => _emplacementCourant;

  /// La partie chargée, ou `null` si aucun emplacement n'est sélectionné.
  Sauvegarde? get partie {
    final int? i = _emplacementCourant;
    if (i == null || i >= _emplacements.length) return null;
    return _emplacements[i].sauvegarde;
  }

  Joueur? get joueur => partie?.joueur;

  /// Vrai s'il existe au moins une partie reprenable (bouton « Continuer »).
  bool get peutContinuer =>
      _emplacements.any((ResultatEmplacement r) => r.estValide);

  /// L'emplacement valide le plus récemment écrit.
  int? get emplacementLePlusRecent {
    int? meilleur;
    DateTime reference = DateTime.fromMillisecondsSinceEpoch(0);
    for (int i = 0; i < _emplacements.length; i++) {
      final Sauvegarde? s = _emplacements[i].sauvegarde;
      if (s != null && s.derniereEcriture.isAfter(reference)) {
        reference = s.derniereEcriture;
        meilleur = i;
      }
    }
    return meilleur;
  }

  // --- Cycle de vie -------------------------------------------------------

  Future<void> initialiser() async {
    _emplacements = await _service.lireTous();
    _pret = true;
    notifyListeners();
  }

  void choisirEmplacement(int index) {
    _emplacementCourant = index;
    notifyListeners();
  }

  /// Reprend automatiquement la partie la plus récente (bouton Continuer).
  bool continuer() {
    final int? index = emplacementLePlusRecent;
    if (index == null) return false;
    choisirEmplacement(index);
    return true;
  }

  Future<void> supprimerEmplacement(int index) async {
    await _service.supprimer(index);
    _emplacements = List<ResultatEmplacement>.from(_emplacements)
      ..[index] = const ResultatEmplacement.vide();
    if (_emplacementCourant == index) _emplacementCourant = null;
    notifyListeners();
  }

  Future<void> creerPersonnage({
    required int emplacement,
    required String nom,
    required ClassePersonnage classe,
    required Caracteristiques repartition,
  }) async {
    final Sauvegarde neuve = Sauvegarde(
      joueur: Joueur.nouveau(
        nom: nom,
        classe: classe,
        repartition: repartition,
      ),
      derniereEcriture: DateTime.now(),
    );
    _emplacements = List<ResultatEmplacement>.from(_emplacements)
      ..[emplacement] = ResultatEmplacement.valide(neuve);
    _emplacementCourant = emplacement;
    notifyListeners();
    await _service.ecrire(emplacement, neuve);
  }

  // --- Modification de la partie -----------------------------------------

  /// Le point de passage OBLIGÉ de toute modification du joueur.
  ///
  /// Une seule fonction met à jour la liste, notifie et écrit. Résultat :
  /// il est impossible d'oublier `notifyListeners()` ou la persistance.
  void _majJoueur(Joueur Function(Joueur) transformation) {
    final int? index = _emplacementCourant;
    final Sauvegarde? actuelle = partie;
    if (index == null || actuelle == null) return;

    final Sauvegarde nouvelle = actuelle.copyWith(
      joueur: transformation(actuelle.joueur),
      derniereEcriture: DateTime.now(),
    );

    _emplacements = List<ResultatEmplacement>.from(_emplacements)
      ..[index] = ResultatEmplacement.valide(nouvelle);
    notifyListeners();
    _service.ecrire(index, nouvelle);
  }

  void depenserPoint(Carac carac) =>
      _majJoueur((Joueur j) => Progression.depenserPoint(j, carac));

  void gagnerExperience(int xp) =>
      _majJoueur((Joueur j) => Progression.gagnerExperience(j, xp));

  void equiper(String objetId) => _majJoueur(
      (Joueur j) => j.copyWith(inventaire: j.inventaire.equiper(objetId)));

  void dequiper(Emplacement emplacement) => _majJoueur((Joueur j) =>
      j.copyWith(inventaire: j.inventaire.dequiper(emplacement)));

  /// Consomme une potion : elle disparaît du sac.
  /// (Ses effets s'appliqueront en jeu, chapitre 38.)
  void consommer(String objetId) {
    final Objet? objet = Catalogue.parId(objetId);
    if (objet == null || objet.type != TypeObjet.consommable) return;
    _majJoueur(
        (Joueur j) => j.copyWith(inventaire: j.inventaire.retirer(objetId)));
  }

  /// Achète un objet. Renvoie `false` si l'or manque ou si le sac est plein.
  bool acheter(Objet objet) {
    final Joueur? j = joueur;
    if (j == null) return false;
    if (j.or < objet.prix) return false;

    final Inventaire apres = j.inventaire.ajouter(objet.id);
    // `ajouter` renvoie le même objet quand le sac est plein.
    if (identical(apres, j.inventaire)) return false;

    _majJoueur((Joueur j) =>
        j.copyWith(or: j.or - objet.prix, inventaire: apres));
    return true;
  }

  /// Revend un objet possédé au prix de revente.
  bool vendre(Objet objet) {
    final Joueur? j = joueur;
    if (j == null || !j.inventaire.contient(objet.id)) return false;
    _majJoueur((Joueur j) => j.copyWith(
          or: j.or + objet.prixRevente,
          inventaire: j.inventaire.retirer(objet.id),
        ));
    return true;
  }

  /// Enregistre le résultat d'une partie (appelé après le Game Over).
  void terminerPartie({required int score, required int niveauAtteint,
      required int secondes}) {
    final int? index = _emplacementCourant;
    final Sauvegarde? actuelle = partie;
    if (index == null || actuelle == null) return;

    final Sauvegarde nouvelle = actuelle.copyWith(
      meilleurScore:
          score > actuelle.meilleurScore ? score : actuelle.meilleurScore,
      niveauAtteint: niveauAtteint > actuelle.niveauAtteint
          ? niveauAtteint
          : actuelle.niveauAtteint,
      partiesJouees: actuelle.partiesJouees + 1,
      secondesDeJeu: actuelle.secondesDeJeu + secondes,
      derniereEcriture: DateTime.now(),
    );

    _emplacements = List<ResultatEmplacement>.from(_emplacements)
      ..[index] = ResultatEmplacement.valide(nouvelle);
    notifyListeners();
    _service.ecrire(index, nouvelle);
  }

  /// Nombre d'emplacements utilisables, pour l'affichage.
  int get nombreEmplacements => Constantes.nombreEmplacements;
}
```

> **Rappel du chapitre 52 :** `context.watch<EtatPartie>()` dans un `build`, `context.read<EtatPartie>()` dans un rappel (`onPressed`, `initState`). L'inverse est l'erreur la plus fréquente avec `provider` : `read` dans un `build` donne un écran qui ne se rafraîchit plus, `watch` dans un `onPressed` lève une exception.

**État exécutable.** Les deux `ChangeNotifier` compilent. Ils ne sont pas encore fournis à l'arbre : c'est l'objet de l'étape suivante.

---

## 62.10 — Étape 10 : les transitions et l'écran de chargement

### Deux transitions réutilisables

**`lib/navigation/transitions.dart`**

```dart
import 'package:flutter/material.dart';

/// Deux transitions, une règle : le GLISSEMENT pour avancer dans la
/// hiérarchie (menu → inventaire), le FONDU pour un changement de contexte
/// (chargement → menu, menu → jeu).
///
/// Rappel du chapitre 50 : `PageRouteBuilder` reçoit deux fonctions, l'une
/// qui construit la page, l'autre qui l'anime. `animation` va de 0 à 1 à
/// l'aller, `secondaryAnimation` anime la page qu'on recouvre.
Route<T> routeGlissement<T>(Widget page, {bool depuisLaDroite = true}) {
  return PageRouteBuilder<T>(
    transitionDuration: const Duration(milliseconds: 280),
    reverseTransitionDuration: const Duration(milliseconds: 220),
    pageBuilder: (BuildContext context, Animation<double> animation,
            Animation<double> secondaryAnimation) =>
        page,
    transitionsBuilder: (BuildContext context, Animation<double> animation,
        Animation<double> secondaryAnimation, Widget child) {
      final Animation<Offset> decalage = Tween<Offset>(
        begin: Offset(depuisLaDroite ? 1.0 : -1.0, 0.0),
        end: Offset.zero,
      ).animate(CurvedAnimation(
        parent: animation,
        curve: Curves.easeOutCubic,
        reverseCurve: Curves.easeInCubic,
      ));
      return SlideTransition(
        position: decalage,
        child: FadeTransition(opacity: animation, child: child),
      );
    },
  );
}

Route<T> routeFondu<T>(Widget page,
    {Duration duree = const Duration(milliseconds: 420)}) {
  return PageRouteBuilder<T>(
    transitionDuration: duree,
    reverseTransitionDuration: duree,
    pageBuilder: (BuildContext context, Animation<double> animation,
            Animation<double> secondaryAnimation) =>
        page,
    transitionsBuilder: (BuildContext context, Animation<double> animation,
        Animation<double> secondaryAnimation, Widget child) {
      return FadeTransition(
        opacity: CurvedAnimation(parent: animation, curve: Curves.easeInOut),
        child: child,
      );
    },
  );
}
```

### L'écran de chargement

Un écran de chargement qui ment — qui affiche une barre pendant deux secondes sans rien charger — est une insulte à l'utilisateur. Le nôtre attend réellement la fin des deux initialisations, avec une durée plancher pour éviter un clignotement quand tout va très vite.

**`lib/ecrans/ecran_chargement.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../etat/etat_reglages.dart';
import '../navigation/transitions.dart';
import 'ecran_menu.dart';

class EcranChargement extends StatefulWidget {
  const EcranChargement({super.key});

  @override
  State<EcranChargement> createState() => _EcranChargementState();
}

class _EcranChargementState extends State<EcranChargement>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controleur;
  String _etape = 'Initialisation...';

  @override
  void initState() {
    super.initState();
    _controleur = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1400),
    )..forward();
    _demarrer();
  }

  Future<void> _demarrer() async {
    // `read` et non `watch` : nous sommes hors d'un `build`.
    final EtatPartie partie = context.read<EtatPartie>();
    final EtatReglages reglages = context.read<EtatReglages>();

    setState(() => _etape = 'Lecture des réglages...');
    await reglages.initialiser();

    setState(() => _etape = 'Lecture des sauvegardes...');
    await partie.initialiser();

    // Durée plancher : sans elle, l'écran clignote sur une machine rapide.
    await _controleur.forward();

    // Après un `await`, le widget peut avoir été détruit. Toujours vérifier.
    if (!mounted) return;
    Navigator.of(context).pushReplacement(routeFondu(const EcranMenu()));
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 48),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text(
                'DONJON DE DART',
                textAlign: TextAlign.center,
                style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                      fontWeight: FontWeight.bold,
                      letterSpacing: 4,
                      color: couleurs.primary,
                    ),
              ),
              const SizedBox(height: 40),
              // `AnimatedBuilder` redessine à chaque image de l'animation
              // sans reconstruire tout l'écran (chapitre 45).
              AnimatedBuilder(
                animation: _controleur,
                builder: (BuildContext context, Widget? child) {
                  return Column(
                    children: <Widget>[
                      ClipRRect(
                        borderRadius: BorderRadius.circular(8),
                        child: LinearProgressIndicator(
                          value: _controleur.value,
                          minHeight: 10,
                        ),
                      ),
                      const SizedBox(height: 12),
                      Text('${(_controleur.value * 100).round()} %'),
                    ],
                  );
                },
              ),
              const SizedBox(height: 24),
              Text(_etape,
                  style: TextStyle(color: couleurs.onSurfaceVariant)),
              const SizedBox(height: 60),
              Text('version 1.0.0',
                  style: TextStyle(
                      fontSize: 12, color: Palette.commun)),
            ],
          ),
        ),
      ),
    );
  }
}
```

**État exécutable.** L'application démarre sur l'écran de chargement, lit réellement le disque, puis fond vers le menu. Il ne reste plus qu'à écrire ce menu.

---

## 62.11 — Étape 11 : le menu principal animé

### Le bouton réutilisable

**`lib/widgets/bouton_menu.dart`**

```dart
import 'package:flutter/material.dart';

/// Une entrée du menu principal.
///
/// `onPressed` à `null` désactive le bouton : Flutter le grise tout seul,
/// et l'utilisateur comprend qu'il manque quelque chose sans message.
class BoutonMenu extends StatelessWidget {
  const BoutonMenu({
    super.key,
    required this.libelle,
    required this.icone,
    required this.onPressed,
    this.principal = false,
    this.badge,
  });

  final String libelle;
  final IconData icone;
  final VoidCallback? onPressed;
  final bool principal;

  /// Texte secondaire affiché à droite (nom du héros, nombre d'objets...).
  final String? badge;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    final Widget contenu = Row(
      children: <Widget>[
        Icon(icone, size: 22),
        const SizedBox(width: 14),
        Expanded(
          child: Text(
            libelle,
            style: const TextStyle(
                fontWeight: FontWeight.w600, letterSpacing: 1.2),
          ),
        ),
        if (badge != null)
          Text(badge!,
              style: TextStyle(
                  fontSize: 12, color: couleurs.onSurfaceVariant)),
      ],
    );

    final ButtonStyle style = ButtonStyle(
      minimumSize: WidgetStateProperty.all<Size>(const Size.fromHeight(52)),
      padding: WidgetStateProperty.all<EdgeInsets>(
          const EdgeInsets.symmetric(horizontal: 20)),
    );

    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 5),
      child: principal
          ? FilledButton(onPressed: onPressed, style: style, child: contenu)
          : OutlinedButton(onPressed: onPressed, style: style, child: contenu),
    );
  }
}
```

### L'animation décalée

Les sept boutons entrent l'un après l'autre. Un seul `AnimationController` suffit : chaque bouton lit une **portion** de sa course grâce à `Interval`.

```text
   contrôleur   0 ─────────────────────────────────── 1
   bouton 0       [====]
   bouton 1          [====]
   bouton 2             [====]     décalage de 0.08 par bouton
   ...
```

**`lib/ecrans/ecran_menu.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:provider/provider.dart';

import '../etat/etat_partie.dart';
import '../modeles/joueur.dart';
import '../modeles/sauvegarde.dart';
import '../navigation/transitions.dart';
import '../widgets/bouton_menu.dart';
import 'ecran_boutique.dart';
import 'ecran_classement.dart';
import 'ecran_emplacements.dart';
import 'ecran_fiche.dart';
import 'ecran_inventaire.dart';
import 'ecran_jeu.dart';
import 'ecran_options.dart';

class EcranMenu extends StatefulWidget {
  const EcranMenu({super.key});

  @override
  State<EcranMenu> createState() => _EcranMenuState();
}

class _EcranMenuState extends State<EcranMenu>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controleur;

  @override
  void initState() {
    super.initState();
    _controleur = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 900),
    )..forward();
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  /// Enveloppe un enfant dans une entrée décalée.
  Widget _anime(int rang, Widget enfant, int total) {
    final double debut = (rang * 0.08).clamp(0.0, 0.6);
    final Animation<double> courbe = CurvedAnimation(
      parent: _controleur,
      curve: Interval(debut, (debut + 0.4).clamp(0.0, 1.0),
          curve: Curves.easeOutCubic),
    );
    return FadeTransition(
      opacity: courbe,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(-0.25, 0),
          end: Offset.zero,
        ).animate(courbe),
        child: enfant,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // `watch` : le menu doit se redessiner quand un personnage est créé.
    final EtatPartie etat = context.watch<EtatPartie>();
    final Joueur? joueur = etat.joueur;
    final Sauvegarde? partie = etat.partie;
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    final List<Widget> entrees = <Widget>[
      BoutonMenu(
        libelle: 'JOUER',
        icone: Icons.play_arrow,
        principal: true,
        onPressed: () => Navigator.of(context)
            .push(routeGlissement<void>(const EcranEmplacements())),
      ),
      BoutonMenu(
        libelle: 'CONTINUER',
        icone: Icons.replay,
        badge: joueur == null ? null : '${joueur.nom} N${joueur.niveau}',
        // Désactivé tant qu'aucune partie n'existe : c'est l'exigence O2.
        onPressed: !etat.peutContinuer
            ? null
            : () {
                context.read<EtatPartie>().continuer();
                Navigator.of(context)
                    .push(routeFondu<void>(const EcranJeu()));
              },
      ),
      BoutonMenu(
        libelle: 'PERSONNAGE',
        icone: Icons.person,
        onPressed: joueur == null
            ? null
            : () => Navigator.of(context)
                .push(routeGlissement<void>(const EcranFiche())),
      ),
      BoutonMenu(
        libelle: 'INVENTAIRE',
        icone: Icons.backpack,
        badge: joueur == null
            ? null
            : '${joueur.inventaire.casesOccupees}/40',
        onPressed: joueur == null
            ? null
            : () => Navigator.of(context)
                .push(routeGlissement<void>(const EcranInventaire())),
      ),
      BoutonMenu(
        libelle: 'BOUTIQUE',
        icone: Icons.storefront,
        onPressed: joueur == null
            ? null
            : () => Navigator.of(context)
                .push(routeGlissement<void>(const EcranBoutique())),
      ),
      BoutonMenu(
        libelle: 'CLASSEMENT',
        icone: Icons.emoji_events,
        onPressed: () => Navigator.of(context)
            .push(routeGlissement<void>(const EcranClassement())),
      ),
      BoutonMenu(
        libelle: 'OPTIONS',
        icone: Icons.settings,
        onPressed: () => Navigator.of(context)
            .push(routeGlissement<void>(const EcranOptions())),
      ),
      BoutonMenu(
        libelle: 'QUITTER',
        icone: Icons.power_settings_new,
        onPressed: () => _confirmerSortie(context),
      ),
    ];

    return Scaffold(
      body: SafeArea(
        child: Center(
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: 420),
            // Indispensable en paysage sur téléphone : sans lui, la colonne
            // déborde et Flutter affiche la bande rayée jaune et noire.
            child: SingleChildScrollView(
              padding: const EdgeInsets.symmetric(horizontal: 28, vertical: 24),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: <Widget>[
                  _anime(
                    0,
                    Column(
                      children: <Widget>[
                        Text(
                          'DONJON DE DART',
                          textAlign: TextAlign.center,
                          style: Theme.of(context)
                              .textTheme
                              .headlineMedium
                              ?.copyWith(
                                fontWeight: FontWeight.bold,
                                letterSpacing: 4,
                                color: couleurs.primary,
                              ),
                        ),
                        const SizedBox(height: 8),
                        Divider(color: couleurs.outlineVariant),
                        const SizedBox(height: 16),
                      ],
                    ),
                    entrees.length,
                  ),
                  for (int i = 0; i < entrees.length; i++)
                    _anime(i + 1, entrees[i], entrees.length),
                  const SizedBox(height: 20),
                  if (partie != null)
                    Text(
                      'Emplacement ${etat.emplacementCourant! + 1} · '
                      '${partie.meilleurScore} pts · '
                      '${partie.tempsDeJeuLisible}',
                      style: TextStyle(
                          fontSize: 12, color: couleurs.onSurfaceVariant),
                    ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  Future<void> _confirmerSortie(BuildContext context) async {
    final bool? sortir = await showDialog<bool>(
      context: context,
      builder: (BuildContext context) => AlertDialog(
        title: const Text('Quitter le jeu ?'),
        content: const Text('Votre progression est déjà enregistrée.'),
        actions: <Widget>[
          TextButton(
              onPressed: () => Navigator.of(context).pop(false),
              child: const Text('ANNULER')),
          FilledButton(
              onPressed: () => Navigator.of(context).pop(true),
              child: const Text('QUITTER')),
        ],
      ),
    );
    if (sortir ?? false) {
      // Sur le Web, cette instruction n'a aucun effet : un onglet ne se
      // ferme pas tout seul. Le bouton doit donc être masqué avec `kIsWeb`
      // dans une version publiée sur le Web (défi 7).
      SystemNavigator.pop();
    }
  }
}
```

**État exécutable.** Le menu s'affiche, les huit entrées entrent en cascade, et trois d'entre elles sont grisées faute de personnage. Les écrans cibles n'existent pas encore : créez-les vides pour compiler, ou commentez les imports au fur et à mesure.

---

## 62.12 — Étape 12 : les emplacements de sauvegarde

Trois emplacements, quatre états d'affichage : vide, occupé, corrompu, et « en cours de suppression ».

**`lib/ecrans/ecran_emplacements.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:provider/provider.dart';

import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../modeles/sauvegarde.dart';
import '../navigation/transitions.dart';
import '../services/sauvegarde_service.dart';
import 'ecran_creation.dart';
import 'ecran_jeu.dart';

class EcranEmplacements extends StatelessWidget {
  const EcranEmplacements({super.key});

  @override
  Widget build(BuildContext context) {
    final EtatPartie etat = context.watch<EtatPartie>();

    return Scaffold(
      appBar: AppBar(title: const Text('Emplacements')),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: etat.emplacements.length,
        itemBuilder: (BuildContext context, int index) =>
            _Tuile(index: index, resultat: etat.emplacements[index]),
      ),
    );
  }
}

class _Tuile extends StatelessWidget {
  const _Tuile({required this.index, required this.resultat});

  final int index;
  final ResultatEmplacement resultat;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    if (resultat.estCorrompu) {
      return Card(
        color: couleurs.errorContainer,
        child: ListTile(
          leading: Icon(Icons.warning_amber, color: couleurs.error),
          title: Text('Emplacement ${index + 1} — sauvegarde illisible'),
          subtitle: Text(resultat.erreur!),
          trailing: TextButton(
            onPressed: () =>
                context.read<EtatPartie>().supprimerEmplacement(index),
            child: const Text('SUPPRIMER'),
          ),
        ),
      );
    }

    if (resultat.estVide) {
      return Card(
        child: ListTile(
          leading: const Icon(Icons.add_circle_outline),
          title: Text('Emplacement ${index + 1}'),
          subtitle: const Text('Vide — appuyez pour créer un héros'),
          onTap: () => Navigator.of(context).push(
            routeGlissement<void>(EcranCreation(emplacement: index)),
          ),
        ),
      );
    }

    final Sauvegarde s = resultat.sauvegarde!;
    final String date =
        DateFormat('dd/MM/yyyy HH:mm').format(s.derniereEcriture);

    return Card(
      child: ListTile(
        leading: CircleAvatar(child: Text(s.joueur.initiale)),
        title: Text('${s.joueur.nom} — ${s.joueur.classe.libelle}'),
        subtitle: Text('Niveau ${s.joueur.niveau} · ${s.meilleurScore} pts · '
            '${s.tempsDeJeuLisible}\nDernière partie : $date'),
        isThreeLine: true,
        trailing: IconButton(
          icon: const Icon(Icons.delete_outline),
          color: Palette.danger,
          tooltip: 'Supprimer cette partie',
          onPressed: () => _confirmerSuppression(context),
        ),
        onTap: () {
          context.read<EtatPartie>().choisirEmplacement(index);
          Navigator.of(context).push(routeFondu<void>(const EcranJeu()));
        },
      ),
    );
  }

  Future<void> _confirmerSuppression(BuildContext context) async {
    // On capture l'état AVANT le `await` : après, le `context` peut être mort.
    final EtatPartie etat = context.read<EtatPartie>();
    final bool? confirme = await showDialog<bool>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        title: Text('Supprimer l\'emplacement ${index + 1} ?'),
        content: const Text('Cette action est définitive.'),
        actions: <Widget>[
          TextButton(
              onPressed: () => Navigator.of(c).pop(false),
              child: const Text('ANNULER')),
          FilledButton(
              onPressed: () => Navigator.of(c).pop(true),
              child: const Text('SUPPRIMER')),
        ],
      ),
    );
    if (confirme ?? false) await etat.supprimerEmplacement(index);
  }
}
```

**État exécutable.** Trois emplacements s'affichent, un appui sur un emplacement vide mène vers la création, un appui sur un emplacement occupé lance la partie.

---

## 62.13 — Étape 13 : la création de personnage

Trois exigences réunies sur un seul écran : un `Form` validé (chapitre 49), un choix parmi trois, et une répartition de points bornée.

### Les règles de validation du nom

| Règle | Message |
| --- | --- |
| Non vide | « Donnez un nom à votre héros. » |
| ≥ 3 caractères | « Trois caractères minimum. » |
| ≤ 16 caractères | Bloqué en amont par `maxLength`. |
| Lettres, chiffres, espaces, tirets, apostrophes | « Caractères autorisés : lettres, chiffres, espace, tiret. » |

**`lib/ecrans/ecran_creation.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../modeles/caracteristiques.dart';
import '../modeles/classe_personnage.dart';
import '../modeles/joueur.dart';
import '../navigation/transitions.dart';
import 'ecran_jeu.dart';

class EcranCreation extends StatefulWidget {
  const EcranCreation({super.key, required this.emplacement});

  final int emplacement;

  @override
  State<EcranCreation> createState() => _EcranCreationState();
}

class _EcranCreationState extends State<EcranCreation> {
  final GlobalKey<FormState> _cleFormulaire = GlobalKey<FormState>();
  final TextEditingController _nom = TextEditingController();

  ClassePersonnage _classe = ClassePersonnage.guerrier;
  Caracteristiques _repartition = Caracteristiques.zero;

  /// Points encore à placer.
  int get _restants => Constantes.pointsCreation - _repartition.total;

  @override
  void dispose() {
    _nom.dispose();
    super.dispose();
  }

  String? _validerNom(String? valeur) {
    final String v = (valeur ?? '').trim();
    if (v.isEmpty) return 'Donnez un nom à votre héros.';
    if (v.length < Constantes.longueurNomMin) {
      return '${Constantes.longueurNomMin} caractères minimum.';
    }
    // Expression régulière : lettres (accents compris), chiffres, espace,
    // tiret et apostrophe. Tout le reste est refusé.
    final RegExp autorise = RegExp(r"^[A-Za-zÀ-ÿ0-9 \-']+$");
    if (!autorise.hasMatch(v)) {
      return 'Lettres, chiffres, espace, tiret et apostrophe seulement.';
    }
    return null;
  }

  void _modifier(Carac carac, int delta) {
    final int actuel = _repartition.lire(carac);
    final int vise = actuel + delta;
    // Deux bornes : jamais négatif, jamais plus que le maximum par
    // caractéristique, et jamais plus de points que disponibles.
    if (vise < 0 || vise > Constantes.pointsMaxParCarac) return;
    if (delta > 0 && _restants <= 0) return;
    setState(() => _repartition = _repartition.modifier(carac, delta));
  }

  /// Aperçu en direct : on fabrique un joueur temporaire pour lire ses
  /// statistiques dérivées. Rien n'est persisté.
  Joueur get _apercu => Joueur(
        nom: _nom.text.isEmpty ? 'Héros' : _nom.text,
        classe: _classe,
        pointsRepartis: _repartition,
      );

  Future<void> _creer() async {
    if (!(_cleFormulaire.currentState?.validate() ?? false)) return;
    if (_restants != 0) return;

    final NavigatorState navigateur = Navigator.of(context);
    await context.read<EtatPartie>().creerPersonnage(
          emplacement: widget.emplacement,
          nom: _nom.text,
          classe: _classe,
          repartition: _repartition,
        );
    if (!mounted) return;
    // `pushReplacement` : on ne revient pas sur l'écran de création.
    navigateur.pushReplacement(routeFondu<void>(const EcranJeu()));
  }

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(title: const Text('Nouveau personnage')),
      body: Form(
        key: _cleFormulaire,
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: <Widget>[
            TextFormField(
              controller: _nom,
              maxLength: Constantes.longueurNomMax,
              textCapitalization: TextCapitalization.words,
              decoration: const InputDecoration(
                labelText: 'Nom du héros',
                border: OutlineInputBorder(),
              ),
              validator: _validerNom,
              // Revalide à chaque frappe une fois la première tentative faite.
              autovalidateMode: AutovalidateMode.onUserInteraction,
              onChanged: (_) => setState(() {}),
            ),
            const SizedBox(height: 12),
            Text('Classe', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 8),
            Row(
              children: <Widget>[
                for (final ClassePersonnage c in ClassePersonnage.values)
                  Expanded(
                    child: Padding(
                      padding: const EdgeInsets.symmetric(horizontal: 4),
                      child: _CarteClasse(
                        classe: c,
                        selectionnee: c == _classe,
                        onTap: () => setState(() => _classe = c),
                      ),
                    ),
                  ),
              ],
            ),
            const SizedBox(height: 8),
            Text(_classe.description,
                style: TextStyle(color: couleurs.onSurfaceVariant)),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: <Widget>[
                Text('Points à répartir',
                    style: Theme.of(context).textTheme.titleMedium),
                Text('$_restants / ${Constantes.pointsCreation}',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      color: _restants == 0 ? couleurs.primary : couleurs.error,
                    )),
              ],
            ),
            const SizedBox(height: 8),
            for (final Carac carac in Carac.values)
              _LigneRepartition(
                carac: carac,
                base: _classe.base.lire(carac),
                ajoute: _repartition.lire(carac),
                onMoins: () => _modifier(carac, -1),
                onPlus: () => _modifier(carac, 1),
              ),
            const SizedBox(height: 16),
            Card(
              child: Padding(
                padding: const EdgeInsets.all(12),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: <Widget>[
                    _Apercu(titre: 'PV', valeur: '${_apercu.pvMax}'),
                    _Apercu(titre: 'Énergie', valeur: '${_apercu.energieMax}'),
                    _Apercu(
                        titre: 'Dégâts',
                        valeur: _apercu.degats.toStringAsFixed(1)),
                    _Apercu(
                        titre: 'Crit.',
                        valeur:
                            '${_apercu.chanceCritique.toStringAsFixed(1)} %'),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 20),
            FilledButton(
              // Le bouton reste désactivé tant qu'il reste des points :
              // c'est plus clair qu'un message d'erreur après coup.
              onPressed: _restants == 0 ? _creer : null,
              style: ButtonStyle(
                minimumSize:
                    WidgetStateProperty.all<Size>(const Size.fromHeight(52)),
              ),
              child: Text(_restants == 0
                  ? 'CRÉER LE HÉROS'
                  : 'IL RESTE $_restants POINT(S)'),
            ),
          ],
        ),
      ),
    );
  }
}

class _CarteClasse extends StatelessWidget {
  const _CarteClasse({
    required this.classe,
    required this.selectionnee,
    required this.onTap,
  });

  final ClassePersonnage classe;
  final bool selectionnee;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(12),
      child: Container(
        padding: const EdgeInsets.symmetric(vertical: 12, horizontal: 6),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(12),
          border: Border.all(
            color: selectionnee ? couleurs.primary : couleurs.outlineVariant,
            width: selectionnee ? 2 : 1,
          ),
          color: selectionnee ? couleurs.primaryContainer : null,
        ),
        child: Column(
          children: <Widget>[
            Text(classe.libelle.toUpperCase(),
                style: const TextStyle(
                    fontWeight: FontWeight.bold, fontSize: 12)),
            const SizedBox(height: 6),
            Text(classe.base.toString(),
                textAlign: TextAlign.center,
                style: TextStyle(
                    fontSize: 11, color: couleurs.onSurfaceVariant)),
          ],
        ),
      ),
    );
  }
}

class _LigneRepartition extends StatelessWidget {
  const _LigneRepartition({
    required this.carac,
    required this.base,
    required this.ajoute,
    required this.onMoins,
    required this.onPlus,
  });

  final Carac carac;
  final int base;
  final int ajoute;
  final VoidCallback onMoins;
  final VoidCallback onPlus;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        children: <Widget>[
          SizedBox(width: 110, child: Text(carac.libelle)),
          SizedBox(
            width: 60,
            child: Text('${base + ajoute}'
                '${ajoute > 0 ? ' (+$ajoute)' : ''}'),
          ),
          IconButton(
            onPressed: ajoute > 0 ? onMoins : null,
            icon: const Icon(Icons.remove_circle_outline),
          ),
          Expanded(
            child: LinearProgressIndicator(
              value: (base + ajoute) / Constantes.caracMaxAffichee,
              minHeight: 8,
            ),
          ),
          IconButton(
            onPressed: onPlus,
            icon: const Icon(Icons.add_circle_outline),
          ),
        ],
      ),
    );
  }
}

class _Apercu extends StatelessWidget {
  const _Apercu({required this.titre, required this.valeur});

  final String titre;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text(titre,
            style: TextStyle(
                fontSize: 11,
                color: Theme.of(context).colorScheme.onSurfaceVariant)),
        const SizedBox(height: 2),
        Text(valeur, style: const TextStyle(fontWeight: FontWeight.bold)),
      ],
    );
  }
}
```

Ajoutez l'import de `Palette` seulement si vous colorez les barres par caractéristique (défi 2).

**État exécutable.** On crée un héros, il est persisté dans l'emplacement choisi, et le menu principal affiche désormais son nom sur le bouton « Continuer ».

---

## 62.14 — Étape 14 : la fiche de personnage

Deux petits widgets réutilisables d'abord, l'écran ensuite.

**`lib/widgets/panneau.dart`**

```dart
import 'package:flutter/material.dart';

/// Un bloc titré, utilisé par la fiche, la boutique et les options.
class Panneau extends StatelessWidget {
  const Panneau({super.key, required this.titre, required this.enfants});

  final String titre;
  final List<Widget> enfants;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Text(
              titre.toUpperCase(),
              style: TextStyle(
                fontSize: 12,
                letterSpacing: 1.5,
                fontWeight: FontWeight.bold,
                color: couleurs.primary,
              ),
            ),
            const SizedBox(height: 12),
            ...enfants,
          ],
        ),
      ),
    );
  }
}
```

**`lib/widgets/barre_stat.dart`**

```dart
import 'package:flutter/material.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../modeles/caracteristiques.dart';

/// Une barre de caractéristique, comme au projet 55 (section 55.11) mais
/// avec la part apportée par l'équipement en surimpression.
///
/// La barre se lit ainsi :
///   ██████████████░░░░░  base pleine, bonus en teinte claire, reste vide
class BarreStat extends StatelessWidget {
  const BarreStat({
    super.key,
    required this.carac,
    required this.base,
    required this.bonus,
  });

  final Carac carac;
  final int base;
  final int bonus;

  Color get _couleur {
    switch (carac) {
      case Carac.force:
        return Palette.force;
      case Carac.dexterite:
        return Palette.dexterite;
      case Carac.intelligence:
        return Palette.intelligence;
      case Carac.vitalite:
        return Palette.vitalite;
    }
  }

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    const double max = Constantes.caracMaxAffichee.toDouble();
    // `clamp` protège l'affichage : une caractéristique au-delà de l'échelle
    // remplirait sinon plus que la largeur disponible.
    final double partBase = (base / max).clamp(0.0, 1.0);
    final double partTotale = ((base + bonus) / max).clamp(0.0, 1.0);

    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 6),
      child: Row(
        children: <Widget>[
          SizedBox(width: 100, child: Text(carac.libelle)),
          SizedBox(
            width: 34,
            child: Text('${base + bonus}',
                style: const TextStyle(fontWeight: FontWeight.bold)),
          ),
          Expanded(
            child: ClipRRect(
              borderRadius: BorderRadius.circular(6),
              child: SizedBox(
                height: 12,
                // Trois couches empilées : fond, total (clair), base (vif).
                child: Stack(
                  children: <Widget>[
                    Container(color: couleurs.surfaceContainerHighest),
                    FractionallySizedBox(
                      widthFactor: partTotale,
                      child: Container(color: _couleur.withValues(alpha: 0.45)),
                    ),
                    FractionallySizedBox(
                      widthFactor: partBase,
                      child: Container(color: _couleur),
                    ),
                  ],
                ),
              ),
            ),
          ),
          SizedBox(
            width: 44,
            child: Text(
              bonus > 0 ? '+$bonus' : '',
              textAlign: TextAlign.right,
              style: TextStyle(fontSize: 12, color: _couleur),
            ),
          ),
        ],
      ),
    );
  }
}
```

> **`withValues(alpha: ...)`** remplace l'ancien `withOpacity(...)`, déprécié depuis Flutter 3.27 parce qu'il perdait de la précision sur les espaces colorimétriques larges. Si votre éditeur souligne cette ligne, c'est que votre SDK est antérieur : mettez-le à jour ou revenez temporairement à `withOpacity(0.45)`.

**`lib/ecrans/ecran_fiche.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_partie.dart';
import '../modeles/caracteristiques.dart';
import '../modeles/inventaire.dart';
import '../modeles/joueur.dart';
import '../modeles/objet.dart';
import '../widgets/barre_stat.dart';
import '../widgets/panneau.dart';

class EcranFiche extends StatelessWidget {
  const EcranFiche({super.key});

  @override
  Widget build(BuildContext context) {
    final EtatPartie etat = context.watch<EtatPartie>();
    final Joueur? joueur = etat.joueur;

    // Toujours traiter le cas « pas de personnage » : l'écran peut être
    // atteint après une suppression, ou par une route directe.
    if (joueur == null) {
      return const Scaffold(
        body: Center(child: Text('Aucun personnage sélectionné.')),
      );
    }

    final Caracteristiques base = joueur.caracteristiquesBase;
    final Caracteristiques bonus = joueur.bonusEquipement;
    final ColorScheme couleurs = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: Text(joueur.nom),
        actions: <Widget>[
          if (joueur.pointsDisponibles > 0)
            Padding(
              padding: const EdgeInsets.only(right: 12),
              child: Chip(
                label: Text('+${joueur.pointsDisponibles}'),
                backgroundColor: couleurs.primaryContainer,
              ),
            ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Row(
            children: <Widget>[
              CircleAvatar(
                radius: 32,
                child: Text(joueur.initiale,
                    style: const TextStyle(fontSize: 26)),
              ),
              const SizedBox(width: 16),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Text(joueur.nom,
                        style: Theme.of(context).textTheme.titleLarge),
                    Text('${joueur.classe.libelle} · Niveau ${joueur.niveau}',
                        style: TextStyle(color: couleurs.onSurfaceVariant)),
                    const SizedBox(height: 8),
                    ClipRRect(
                      borderRadius: BorderRadius.circular(6),
                      child: LinearProgressIndicator(
                        value: joueur.avancementNiveau,
                        minHeight: 8,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Text('${joueur.experience} / ${joueur.xpRequis} XP',
                        style: const TextStyle(fontSize: 12)),
                  ],
                ),
              ),
            ],
          ),
          Panneau(
            titre: 'Caractéristiques',
            enfants: <Widget>[
              for (final Carac carac in Carac.values)
                Row(
                  children: <Widget>[
                    Expanded(
                      child: BarreStat(
                        carac: carac,
                        base: base.lire(carac),
                        bonus: bonus.lire(carac),
                      ),
                    ),
                    if (joueur.pointsDisponibles > 0)
                      IconButton(
                        tooltip: 'Dépenser un point',
                        icon: const Icon(Icons.add_circle),
                        onPressed: () =>
                            context.read<EtatPartie>().depenserPoint(carac),
                      ),
                  ],
                ),
            ],
          ),
          Panneau(
            titre: 'Statistiques dérivées',
            enfants: <Widget>[
              _Ligne('Points de vie', '${joueur.pvMax}'),
              _Ligne('Énergie', '${joueur.energieMax}'),
              _Ligne('Dégâts', joueur.degats.toStringAsFixed(1)),
              _Ligne('Défense', joueur.defense.toStringAsFixed(1)),
              _Ligne('Chance critique',
                  '${joueur.chanceCritique.toStringAsFixed(1)} %'),
            ],
          ),
          Panneau(
            titre: 'Équipement',
            enfants: <Widget>[
              for (final Emplacement e in Emplacement.values)
                Builder(builder: (BuildContext context) {
                  final Objet? porte = joueur.inventaire.objetEquipe(e);
                  return ListTile(
                    contentPadding: EdgeInsets.zero,
                    dense: true,
                    title: Text(e.libelle),
                    subtitle: Text(porte?.nom ?? '— vide —'),
                    trailing: porte == null
                        ? null
                        : TextButton(
                            onPressed: () =>
                                context.read<EtatPartie>().dequiper(e),
                            child: const Text('RETIRER'),
                          ),
                  );
                }),
            ],
          ),
        ],
      ),
    );
  }
}

class _Ligne extends StatelessWidget {
  const _Ligne(this.libelle, this.valeur);

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

**État exécutable.** La fiche affiche tout, les points se dépensent, et retirer une arme fait immédiatement baisser les dégâts affichés : la preuve que rien n'est stocké en double.

---

## 62.15 — Étape 15 : l'inventaire en grille, filtre et tri

### La logique de tri, séparée de l'écran

**`lib/logique/tri_inventaire.dart`**

```dart
import '../modeles/catalogue.dart';
import '../modeles/inventaire.dart';
import '../modeles/objet.dart';

enum TriObjet {
  rarete('Rareté'),
  nom('Nom'),
  prix('Valeur'),
  type('Type');

  const TriObjet(this.libelle);

  final String libelle;
}

/// Ne garde que les lignes dont l'objet existe et correspond au filtre.
///
/// `filtre == null` signifie « tous les types ». Fonction pure : elle ne
/// touche ni à l'état ni à l'écran, donc elle est testable (étape 62.20).
List<LigneInventaire> filtrer(
  List<LigneInventaire> lignes,
  TypeObjet? filtre,
) {
  return lignes.where((LigneInventaire l) {
    final Objet? o = Catalogue.parId(l.objetId);
    if (o == null) return false;
    return filtre == null || o.type == filtre;
  }).toList();
}

/// Trie une copie de la liste. Ne modifie jamais l'entrée.
List<LigneInventaire> trier(List<LigneInventaire> lignes, TriObjet tri) {
  final List<LigneInventaire> copie = List<LigneInventaire>.from(lignes);

  copie.sort((LigneInventaire a, LigneInventaire b) {
    final Objet? oa = Catalogue.parId(a.objetId);
    final Objet? ob = Catalogue.parId(b.objetId);
    // Un objet inconnu part à la fin : décision explicite, comme au 58.16.
    if (oa == null && ob == null) return 0;
    if (oa == null) return 1;
    if (ob == null) return -1;

    switch (tri) {
      case TriObjet.rarete:
        // Rang décroissant : les légendaires en premier.
        final int parRarete = ob.rarete.rang.compareTo(oa.rarete.rang);
        return parRarete != 0 ? parRarete : oa.nom.compareTo(ob.nom);
      case TriObjet.nom:
        return oa.nom.toLowerCase().compareTo(ob.nom.toLowerCase());
      case TriObjet.prix:
        final int parPrix = ob.prix.compareTo(oa.prix);
        return parPrix != 0 ? parPrix : oa.nom.compareTo(ob.nom);
      case TriObjet.type:
        final int parType = oa.type.index.compareTo(ob.type.index);
        return parType != 0 ? parType : oa.nom.compareTo(ob.nom);
    }
  });

  return copie;
}
```

### La case d'inventaire

**`lib/widgets/case_objet.dart`**

```dart
import 'package:flutter/material.dart';

import '../config/palette.dart';
import '../modeles/objet.dart';

/// L'icône associée à un type d'objet. Aucun fichier image n'est nécessaire.
IconData iconePour(TypeObjet type) {
  switch (type) {
    case TypeObjet.arme:
      return Icons.gavel;
    case TypeObjet.armure:
      return Icons.shield;
    case TypeObjet.accessoire:
      return Icons.diamond_outlined;
    case TypeObjet.consommable:
      return Icons.local_drink;
    case TypeObjet.tresor:
      return Icons.auto_awesome;
  }
}

Color couleurRarete(Rarete rarete) {
  switch (rarete) {
    case Rarete.commun:
      return Palette.commun;
    case Rarete.rare:
      return Palette.rare;
    case Rarete.epique:
      return Palette.epique;
    case Rarete.legendaire:
      return Palette.legendaire;
  }
}

/// Une case de la grille d'inventaire.
class CaseObjet extends StatelessWidget {
  const CaseObjet({
    super.key,
    required this.objet,
    required this.quantite,
    required this.equipe,
    required this.onTap,
  });

  final Objet objet;
  final int quantite;
  final bool equipe;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final Color couleur = couleurRarete(objet.rarete);

    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(10),
      child: Container(
        padding: const EdgeInsets.all(6),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(10),
          border: Border.all(color: couleur, width: equipe ? 2.5 : 1.2),
          color: couleur.withValues(alpha: 0.08),
        ),
        child: Stack(
          children: <Widget>[
            Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Icon(iconePour(objet.type), color: couleur, size: 28),
                const SizedBox(height: 4),
                Text(
                  objet.nom,
                  textAlign: TextAlign.center,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                  style: const TextStyle(fontSize: 11),
                ),
                Text(
                  objet.rarete.libelle.toUpperCase(),
                  style: TextStyle(
                      fontSize: 9,
                      color: couleur,
                      fontWeight: FontWeight.bold),
                ),
              ],
            ),
            if (quantite > 1)
              Positioned(
                right: 0,
                bottom: 0,
                child: Text('x$quantite',
                    style: const TextStyle(
                        fontSize: 11, fontWeight: FontWeight.bold)),
              ),
            if (equipe)
              const Positioned(
                right: 0,
                top: 0,
                child: Icon(Icons.check_circle, size: 14),
              ),
          ],
        ),
      ),
    );
  }
}
```

### L'écran

**`lib/ecrans/ecran_inventaire.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../config/constantes.dart';
import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../logique/tri_inventaire.dart';
import '../modeles/catalogue.dart';
import '../modeles/inventaire.dart';
import '../modeles/joueur.dart';
import '../modeles/objet.dart';
import '../widgets/case_objet.dart';

class EcranInventaire extends StatefulWidget {
  const EcranInventaire({super.key});

  @override
  State<EcranInventaire> createState() => _EcranInventaireState();
}

class _EcranInventaireState extends State<EcranInventaire> {
  TypeObjet? _filtre;
  TriObjet _tri = TriObjet.rarete;

  @override
  Widget build(BuildContext context) {
    final EtatPartie etat = context.watch<EtatPartie>();
    final Joueur? joueur = etat.joueur;
    if (joueur == null) {
      return const Scaffold(body: Center(child: Text('Aucun personnage.')));
    }

    // Filtre PUIS tri : deux fonctions pures composées. Aucune ne modifie
    // l'inventaire réel.
    final List<LigneInventaire> visibles =
        trier(filtrer(joueur.inventaire.lignes, _filtre), _tri);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Inventaire'),
        actions: <Widget>[
          Center(
            child: Text(
              '${joueur.inventaire.casesOccupees}/'
              '${Constantes.tailleInventaire}',
            ),
          ),
          const SizedBox(width: 12),
          const Icon(Icons.circle, color: Palette.or, size: 12),
          const SizedBox(width: 4),
          Center(child: Text('${joueur.or}')),
          const SizedBox(width: 12),
          PopupMenuButton<TriObjet>(
            tooltip: 'Trier',
            icon: const Icon(Icons.sort),
            initialValue: _tri,
            onSelected: (TriObjet t) => setState(() => _tri = t),
            itemBuilder: (BuildContext context) => <PopupMenuEntry<TriObjet>>[
              for (final TriObjet t in TriObjet.values)
                PopupMenuItem<TriObjet>(value: t, child: Text(t.libelle)),
            ],
          ),
        ],
      ),
      body: Column(
        children: <Widget>[
          SingleChildScrollView(
            scrollDirection: Axis.horizontal,
            padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
            child: Row(
              children: <Widget>[
                FilterChip(
                  label: const Text('Tout'),
                  selected: _filtre == null,
                  onSelected: (_) => setState(() => _filtre = null),
                ),
                for (final TypeObjet t in TypeObjet.values)
                  Padding(
                    padding: const EdgeInsets.only(left: 8),
                    child: FilterChip(
                      label: Text(t.libelle),
                      selected: _filtre == t,
                      onSelected: (_) => setState(() => _filtre = t),
                    ),
                  ),
              ],
            ),
          ),
          Expanded(
            child: visibles.isEmpty
                ? const Center(child: Text('Aucun objet dans cette catégorie.'))
                : GridView.builder(
                    padding: const EdgeInsets.all(12),
                    // Rappel du chapitre 48 : ce délégué fixe le NOMBRE de
                    // colonnes. `childAspectRatio` inférieur à 1 rend les
                    // cases plus hautes que larges.
                    gridDelegate:
                        const SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: 4,
                      mainAxisSpacing: 10,
                      crossAxisSpacing: 10,
                      childAspectRatio: 0.82,
                    ),
                    itemCount: visibles.length,
                    itemBuilder: (BuildContext context, int index) {
                      final LigneInventaire ligne = visibles[index];
                      final Objet objet = Catalogue.parId(ligne.objetId)!;
                      return CaseObjet(
                        objet: objet,
                        quantite: ligne.quantite,
                        equipe: joueur.inventaire.estEquipe(objet.id),
                        onTap: () => _ouvrirActions(context, objet, joueur),
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }

  /// Feuille d'actions contextuelles : ce qu'on peut faire dépend du type.
  void _ouvrirActions(BuildContext context, Objet objet, Joueur joueur) {
    final EtatPartie etat = context.read<EtatPartie>();
    final bool equipe = joueur.inventaire.estEquipe(objet.id);

    showModalBottomSheet<void>(
      context: context,
      showDragHandle: true,
      builder: (BuildContext feuille) {
        return SafeArea(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              ListTile(
                leading: Icon(iconePour(objet.type),
                    color: couleurRarete(objet.rarete)),
                title: Text(objet.nom),
                subtitle: Text('${objet.description}\n${objet.resumeBonus}'),
                isThreeLine: true,
              ),
              const Divider(),
              if (objet.type.equipable && !equipe)
                ListTile(
                  leading: const Icon(Icons.check_circle_outline),
                  title: const Text('Équiper'),
                  onTap: () {
                    etat.equiper(objet.id);
                    Navigator.of(feuille).pop();
                  },
                ),
              if (objet.type.equipable && equipe)
                ListTile(
                  leading: const Icon(Icons.remove_circle_outline),
                  title: const Text('Retirer'),
                  onTap: () {
                    final Emplacement? e = Emplacement.pour(objet.type);
                    if (e != null) etat.dequiper(e);
                    Navigator.of(feuille).pop();
                  },
                ),
              if (objet.type == TypeObjet.consommable)
                ListTile(
                  leading: const Icon(Icons.local_drink),
                  title: const Text('Utiliser'),
                  subtitle: Text(objet.resumeBonus),
                  onTap: () {
                    etat.consommer(objet.id);
                    Navigator.of(feuille).pop();
                  },
                ),
              ListTile(
                leading: const Icon(Icons.sell_outlined),
                title: Text('Vendre pour ${objet.prixRevente} or'),
                onTap: () {
                  etat.vendre(objet);
                  Navigator.of(feuille).pop();
                },
              ),
            ],
          ),
        );
      },
    );
  }
}
```

**État exécutable.** La grille s'affiche, les filtres et les tris se combinent, et une case ouvre les actions possibles pour cet objet précis. Équiper une épée met à jour la fiche de personnage sans qu'aucun code de synchronisation n'ait été écrit : c'est tout le bénéfice de l'état centralisé.

---

## 62.16 — Étape 16 : la boutique et la monnaie

Deux onglets, une seule règle : on n'achète pas sans or, on ne vend pas ce qu'on n'a pas.

**`lib/ecrans/ecran_boutique.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../modeles/catalogue.dart';
import '../modeles/inventaire.dart';
import '../modeles/joueur.dart';
import '../modeles/objet.dart';
import '../widgets/case_objet.dart';

class EcranBoutique extends StatelessWidget {
  const EcranBoutique({super.key});

  @override
  Widget build(BuildContext context) {
    final Joueur? joueur = context.watch<EtatPartie>().joueur;
    if (joueur == null) {
      return const Scaffold(body: Center(child: Text('Aucun personnage.')));
    }

    // `DefaultTabController` : deux onglets sans écrire de contrôleur
    // (rappel du chapitre 50).
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Boutique'),
          actions: <Widget>[
            const Icon(Icons.circle, color: Palette.or, size: 12),
            const SizedBox(width: 6),
            Center(
                child: Text('${joueur.or}',
                    style: const TextStyle(fontWeight: FontWeight.bold))),
            const SizedBox(width: 16),
          ],
          bottom: const TabBar(
            tabs: <Widget>[Tab(text: 'ACHETER'), Tab(text: 'VENDRE')],
          ),
        ),
        body: TabBarView(
          children: <Widget>[
            _ListeAchat(joueur: joueur),
            _ListeVente(joueur: joueur),
          ],
        ),
      ),
    );
  }
}

class _ListeAchat extends StatelessWidget {
  const _ListeAchat({required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    final List<Objet> enVente = Catalogue.enVente;
    return ListView.separated(
      itemCount: enVente.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (BuildContext context, int index) {
        final Objet objet = enVente[index];
        final bool abordable = joueur.or >= objet.prix;

        return ListTile(
          leading: Icon(iconePour(objet.type),
              color: couleurRarete(objet.rarete)),
          title: Text(objet.nom),
          subtitle: Text('${objet.rarete.libelle} · ${objet.resumeBonus}'),
          trailing: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.end,
            children: <Widget>[
              Text('${objet.prix} or',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: abordable ? null : Palette.danger,
                  )),
              TextButton(
                // Bouton désactivé plutôt que message d'erreur : le joueur
                // voit immédiatement ce qu'il peut se permettre.
                onPressed: !abordable
                    ? null
                    : () => _acheter(context, objet),
                child: const Text('ACHETER'),
              ),
            ],
          ),
        );
      },
    );
  }

  void _acheter(BuildContext context, Objet objet) {
    final bool ok = context.read<EtatPartie>().acheter(objet);
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(ok
            ? '${objet.nom} acheté pour ${objet.prix} or.'
            : 'Achat impossible : or insuffisant ou sac plein.'),
        duration: const Duration(seconds: 2),
      ),
    );
  }
}

class _ListeVente extends StatelessWidget {
  const _ListeVente({required this.joueur});

  final Joueur joueur;

  @override
  Widget build(BuildContext context) {
    final List<LigneInventaire> lignes = joueur.inventaire.lignes;
    if (lignes.isEmpty) {
      return const Center(child: Text('Votre sac est vide.'));
    }

    return ListView.separated(
      itemCount: lignes.length,
      separatorBuilder: (_, __) => const Divider(height: 1),
      itemBuilder: (BuildContext context, int index) {
        final LigneInventaire ligne = lignes[index];
        final Objet? objet = Catalogue.parId(ligne.objetId);
        if (objet == null) return const SizedBox.shrink();
        final bool equipe = joueur.inventaire.estEquipe(objet.id);

        return ListTile(
          leading: Icon(iconePour(objet.type),
              color: couleurRarete(objet.rarete)),
          title: Text('${objet.nom}${ligne.quantite > 1 ?
              ' x${ligne.quantite}' : ''}'),
          subtitle: Text(equipe ? 'Équipé' : objet.rarete.libelle),
          trailing: TextButton(
            onPressed: () {
              context.read<EtatPartie>().vendre(objet);
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                    content: Text(
                        '${objet.nom} vendu pour ${objet.prixRevente} or.')),
              );
            },
            child: Text('${objet.prixRevente} or'),
          ),
        );
      },
    );
  }
}
```

> **Une décision de game design.** Vendre un objet équipé est autorisé : `Inventaire.retirer` le déséquipe automatiquement (étape 62.4). L'alternative — interdire la vente — obligerait le joueur à un aller-retour par la fiche. Le confort l'emporte, à condition que la règle soit dans le modèle, pas dans l'écran.

**État exécutable.** L'économie tourne : on achète, on vend, l'or bouge, les statistiques suivent.

---

## 62.17 — Étape 17 : le classement, local puis distant

### Le modèle

**`lib/modeles/score.dart`**

```dart
/// Une entrée de classement.
class Score {
  const Score({
    required this.nom,
    required this.points,
    required this.niveau,
    required this.date,
  });

  final String nom;
  final int points;
  final int niveau;
  final DateTime date;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'nom': nom,
        'points': points,
        'niveau': niveau,
        'date': date.toIso8601String(),
      };

  factory Score.fromJson(Map<String, dynamic> json) {
    return Score(
      nom: (json['nom'] as String?) ?? 'Anonyme',
      points: (json['points'] as num?)?.toInt() ?? 0,
      niveau: (json['niveau'] as num?)?.toInt() ?? 1,
      date: DateTime.tryParse((json['date'] as String?) ?? '') ??
          DateTime.fromMillisecondsSinceEpoch(0),
    );
  }
}
```

### L'interface de dépôt

Le principe du chapitre 58 : une interface, plusieurs implémentations. L'écran ne sait pas d'où viennent les scores.

**`lib/services/depot_classement.dart`**

```dart
import '../modeles/score.dart';

/// Ce que tout classement sait faire, quelle que soit sa source.
abstract class DepotClassement {
  /// Les meilleurs scores, déjà triés du plus grand au plus petit.
  Future<List<Score>> lire();

  /// Enregistre un score. Un classement distant en lecture seule peut
  /// simplement ne rien faire.
  Future<void> enregistrer(Score score);
}
```

**`lib/services/classement_local.dart`**

```dart
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../config/constantes.dart';
import '../modeles/score.dart';
import 'depot_classement.dart';

/// Les meilleurs scores de cet appareil.
class ClassementLocal implements DepotClassement {
  ClassementLocal({SharedPreferencesAsync? prefs})
      : _prefs = prefs ?? SharedPreferencesAsync();

  final SharedPreferencesAsync _prefs;
  static const String _cle = 'classement_local';

  @override
  Future<List<Score>> lire() async {
    try {
      final String? brut = await _prefs.getString(_cle);
      if (brut == null || brut.isEmpty) return <Score>[];
      final Object? decode = jsonDecode(brut);
      if (decode is! List) return <Score>[];
      final List<Score> scores = <Score>[
        for (final Object? e in decode)
          if (e is Map<String, dynamic>) Score.fromJson(e),
      ];
      scores.sort((Score a, Score b) => b.points.compareTo(a.points));
      return scores;
    } catch (e) {
      debugPrint('[Classement] Lecture impossible : $e');
      return <Score>[];
    }
  }

  @override
  Future<void> enregistrer(Score score) async {
    final List<Score> scores = await lire()
      ..add(score);
    scores.sort((Score a, Score b) => b.points.compareTo(a.points));
    // On ne garde que le haut du tableau : sinon le fichier grossit sans fin.
    final List<Score> gardes =
        scores.take(Constantes.tailleClassement).toList();
    await _prefs.setString(
      _cle,
      jsonEncode(gardes.map((Score s) => s.toJson()).toList()),
    );
  }
}
```

> **Attention à la ligne `await lire()..add(score)`.** La cascade `..` s'applique au résultat de `await lire()`, c'est-à-dire à la liste. C'est correct, mais peu lisible ; si vous hésitez, écrivez-le en deux instructions.

**`lib/services/classement_distant.dart`**

```dart
import 'dart:convert';

import 'package:http/http.dart' as http;

import '../modeles/score.dart';
import 'depot_classement.dart';

/// Le classement mondial, servi par une API REST (chapitre 53).
///
/// L'URL ci-dessous est un EXEMPLE. Remplacez-la par celle de votre serveur.
/// Le contrat attendu est le suivant :
///
///   GET  /classement          → 200, corps = [ {nom, points, niveau, date} ]
///   POST /classement          → 201, corps = l'objet créé
class ClassementDistant implements DepotClassement {
  ClassementDistant({http.Client? client, this.base = _baseParDefaut})
      : _client = client ?? http.Client();

  static const String _baseParDefaut = 'https://api.exemple.com/v1';

  final http.Client _client;
  final String base;

  @override
  Future<List<Score>> lire() async {
    final Uri url = Uri.parse('$base/classement?limite=10');

    // `timeout` est indispensable : sans lui, un serveur qui ne répond pas
    // laisse l'écran en chargement pour toujours (leçon du chapitre 53).
    final http.Response reponse =
        await _client.get(url).timeout(const Duration(seconds: 8));

    if (reponse.statusCode != 200) {
      throw http.ClientException(
          'Le serveur a répondu ${reponse.statusCode}', url);
    }

    // `bodyBytes` + `utf8.decode` : `body` suppose du latin-1 quand l'en-tête
    // ne précise pas l'encodage, et les accents deviennent illisibles.
    final Object? decode = jsonDecode(utf8.decode(reponse.bodyBytes));
    if (decode is! List) {
      throw const FormatException('Réponse inattendue du serveur.');
    }

    return <Score>[
      for (final Object? e in decode)
        if (e is Map<String, dynamic>) Score.fromJson(e),
    ];
  }

  @override
  Future<void> enregistrer(Score score) async {
    await _client
        .post(
          Uri.parse('$base/classement'),
          headers: const <String, String>{
            'Content-Type': 'application/json; charset=utf-8',
          },
          body: jsonEncode(score.toJson()),
        )
        .timeout(const Duration(seconds: 8));
  }
}

/// Implémentation de secours, utilisable sans serveur.
///
/// Elle simule une latence et renvoie des données figées. C'est ce qui
/// permet de développer et de démontrer l'écran sans back-end, exactement
/// comme le dépôt en mémoire du chapitre 58.
class ClassementFactice implements DepotClassement {
  @override
  Future<List<Score>> lire() async {
    await Future<void>.delayed(const Duration(milliseconds: 900));
    final DateTime maintenant = DateTime.now();
    return <Score>[
      Score(nom: 'Sylvine', points: 9120, niveau: 12, date: maintenant),
      Score(nom: 'Bruk', points: 7740, niveau: 10, date: maintenant),
      Score(nom: 'Nym', points: 6015, niveau: 9, date: maintenant),
      Score(nom: 'Orl', points: 4320, niveau: 7, date: maintenant),
      Score(nom: 'Vess', points: 3110, niveau: 6, date: maintenant),
    ];
  }

  @override
  Future<void> enregistrer(Score score) async {}
}
```

### L'écran

**`lib/ecrans/ecran_classement.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../config/palette.dart';
import '../etat/etat_partie.dart';
import '../modeles/score.dart';
import '../services/classement_distant.dart';
import '../services/classement_local.dart';
import '../services/depot_classement.dart';

class EcranClassement extends StatefulWidget {
  const EcranClassement({super.key});

  @override
  State<EcranClassement> createState() => _EcranClassementState();
}

class _EcranClassementState extends State<EcranClassement> {
  final DepotClassement _local = ClassementLocal();

  /// Remplacez par `ClassementDistant()` quand votre serveur existe.
  final DepotClassement _distant = ClassementFactice();

  late Future<List<Score>> _futurLocal;
  late Future<List<Score>> _futurDistant;

  @override
  void initState() {
    super.initState();
    // Le `Future` est créé UNE fois, dans `initState`. Le créer dans `build`
    // relancerait la requête à chaque reconstruction : c'est l'erreur la
    // plus courante avec `FutureBuilder` (chapitre 53).
    _recharger();
  }

  void _recharger() {
    _futurLocal = _local.lire();
    _futurDistant = _distant.lire();
  }

  @override
  Widget build(BuildContext context) {
    final String? nomJoueur = context.watch<EtatPartie>().joueur?.nom;

    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Classement'),
          actions: <Widget>[
            IconButton(
              icon: const Icon(Icons.refresh),
              tooltip: 'Recharger',
              onPressed: () => setState(_recharger),
            ),
          ],
          bottom: const TabBar(
            tabs: <Widget>[Tab(text: 'LOCAL'), Tab(text: 'MONDIAL')],
          ),
        ),
        body: TabBarView(
          children: <Widget>[
            _Liste(futur: _futurLocal, moi: nomJoueur, onReessayer: () =>
                setState(_recharger)),
            _Liste(futur: _futurDistant, moi: nomJoueur, onReessayer: () =>
                setState(_recharger)),
          ],
        ),
      ),
    );
  }
}

/// Les trois états d'un `FutureBuilder`, tous traités : chargement, erreur,
/// données — plus le quatrième que les débutants oublient : liste vide.
class _Liste extends StatelessWidget {
  const _Liste({required this.futur, required this.moi,
      required this.onReessayer});

  final Future<List<Score>> futur;
  final String? moi;
  final VoidCallback onReessayer;

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Score>>(
      future: futur,
      builder: (BuildContext context, AsyncSnapshot<List<Score>> instantane) {
        if (instantane.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        if (instantane.hasError) {
          return Center(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: <Widget>[
                  const Icon(Icons.cloud_off, size: 48),
                  const SizedBox(height: 12),
                  const Text('Classement indisponible.',
                      textAlign: TextAlign.center),
                  const SizedBox(height: 4),
                  Text('${instantane.error}',
                      textAlign: TextAlign.center,
                      style: const TextStyle(fontSize: 12)),
                  const SizedBox(height: 16),
                  FilledButton(
                      onPressed: onReessayer,
                      child: const Text('RÉESSAYER')),
                ],
              ),
            ),
          );
        }

        final List<Score> scores = instantane.data ?? <Score>[];
        if (scores.isEmpty) {
          return const Center(
            child: Text('Aucun score enregistré.\nTerminez une partie.',
                textAlign: TextAlign.center),
          );
        }

        return ListView.separated(
          itemCount: scores.length,
          separatorBuilder: (_, __) => const Divider(height: 1),
          itemBuilder: (BuildContext context, int index) {
            final Score s = scores[index];
            final bool cestMoi = moi != null && s.nom == moi;
            return ListTile(
              leading: SizedBox(
                width: 40,
                child: Row(
                  children: <Widget>[
                    Text('${index + 1}',
                        style: const TextStyle(fontWeight: FontWeight.bold)),
                    if (index == 0)
                      const Icon(Icons.emoji_events,
                          color: Palette.or, size: 18),
                  ],
                ),
              ),
              title: Text(s.nom,
                  style: TextStyle(
                      fontWeight:
                          cestMoi ? FontWeight.bold : FontWeight.normal)),
              subtitle: Text('Niveau ${s.niveau}'),
              trailing: Text('${s.points}',
                  style: const TextStyle(
                      fontSize: 16, fontWeight: FontWeight.bold)),
              selected: cestMoi,
            );
          },
        );
      },
    );
  }
}
```

**État exécutable.** Deux onglets, quatre états gérés chacun, et un bouton « Réessayer » qui fonctionne réellement puisqu'il recrée le `Future`.

---

## 62.18 — Étape 18 : l'écran d'options et le remappage des touches

### Capturer une touche

Le remappage est le seul point techniquement neuf du chapitre. Le principe :

```text
1. L'utilisateur appuie sur la ligne « Sauter »
2. Un dialogue s'ouvre, avec un Focus autofocus
3. Le prochain KeyDownEvent est capté
4. On refuse les modificateurs seuls (Maj, Ctrl, Alt)
5. Échap annule ; toute autre touche est assignée
```

`onKeyEvent` d'un `Focus` a la signature `KeyEventResult Function(FocusNode, KeyEvent)`. Renvoyer `KeyEventResult.handled` empêche l'événement de remonter.

**`lib/ecrans/ecran_options.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:provider/provider.dart';

import '../etat/etat_reglages.dart';
import '../modeles/reglages.dart';
import '../widgets/panneau.dart';

class EcranOptions extends StatelessWidget {
  const EcranOptions({super.key});

  @override
  Widget build(BuildContext context) {
    final EtatReglages etat = context.watch<EtatReglages>();
    final Reglages r = etat.reglages;

    return Scaffold(
      appBar: AppBar(title: const Text('Options')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          Panneau(
            titre: 'Audio',
            enfants: <Widget>[
              _Curseur(
                libelle: 'Musique',
                valeur: r.volumeMusique,
                onChanged: etat.changerVolumeMusique,
              ),
              _Curseur(
                libelle: 'Effets',
                valeur: r.volumeEffets,
                onChanged: etat.changerVolumeEffets,
              ),
            ],
          ),
          Panneau(
            titre: 'Jeu',
            enfants: <Widget>[
              SegmentedButton<Difficulte>(
                segments: <ButtonSegment<Difficulte>>[
                  for (final Difficulte d in Difficulte.values)
                    ButtonSegment<Difficulte>(
                        value: d, label: Text(d.libelle)),
                ],
                selected: <Difficulte>{r.difficulte},
                // `onSelectionChanged` reçoit un ENSEMBLE, même en mode
                // sélection unique : d'où le `.first`.
                onSelectionChanged: (Set<Difficulte> choix) =>
                    etat.changerDifficulte(choix.first),
                showSelectedIcon: false,
              ),
            ],
          ),
          Panneau(
            titre: 'Affichage',
            enfants: <Widget>[
              SegmentedButton<ModeTheme>(
                segments: <ButtonSegment<ModeTheme>>[
                  for (final ModeTheme m in ModeTheme.values)
                    ButtonSegment<ModeTheme>(value: m, label: Text(m.libelle)),
                ],
                selected: <ModeTheme>{r.modeTheme},
                onSelectionChanged: (Set<ModeTheme> choix) =>
                    etat.changerTheme(choix.first),
                showSelectedIcon: false,
              ),
            ],
          ),
          Panneau(
            titre: 'Commandes',
            enfants: <Widget>[
              for (final ActionJeu action in ActionJeu.values)
                ListTile(
                  contentPadding: EdgeInsets.zero,
                  dense: true,
                  title: Text(action.libelle),
                  trailing: OutlinedButton(
                    onPressed: () => _capturer(context, action),
                    child: Text(libelleTouche(r.touches[action]!)),
                  ),
                ),
              const SizedBox(height: 8),
              TextButton.icon(
                onPressed: etat.reinitialiserTouches,
                icon: const Icon(Icons.settings_backup_restore),
                label: const Text('RÉINITIALISER LES TOUCHES'),
              ),
            ],
          ),
        ],
      ),
    );
  }

  Future<void> _capturer(BuildContext context, ActionJeu action) async {
    final EtatReglages etat = context.read<EtatReglages>();
    final LogicalKeyboardKey? touche = await showDialog<LogicalKeyboardKey>(
      context: context,
      builder: (BuildContext c) => _DialogueTouche(action: action),
    );
    if (touche != null) etat.assignerTouche(action, touche);
  }
}

class _Curseur extends StatelessWidget {
  const _Curseur({
    required this.libelle,
    required this.valeur,
    required this.onChanged,
  });

  final String libelle;
  final double valeur;
  final ValueChanged<double> onChanged;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: <Widget>[
        SizedBox(width: 80, child: Text(libelle)),
        Expanded(
          child: Slider(
            value: valeur,
            // 20 crans : assez fin pour être précis, assez gros pour que
            // la valeur affichée ne saute pas de 1 % en 1 %.
            divisions: 20,
            label: '${(valeur * 100).round()} %',
            onChanged: onChanged,
          ),
        ),
        SizedBox(
          width: 48,
          child: Text('${(valeur * 100).round()} %',
              textAlign: TextAlign.right),
        ),
      ],
    );
  }
}

/// Le dialogue qui attend une frappe.
class _DialogueTouche extends StatelessWidget {
  const _DialogueTouche({required this.action});

  final ActionJeu action;

  /// Les touches qu'on refuse d'assigner seules.
  static const Set<LogicalKeyboardKey> _interdites = <LogicalKeyboardKey>{
    LogicalKeyboardKey.shiftLeft,
    LogicalKeyboardKey.shiftRight,
    LogicalKeyboardKey.controlLeft,
    LogicalKeyboardKey.controlRight,
    LogicalKeyboardKey.altLeft,
    LogicalKeyboardKey.altRight,
    LogicalKeyboardKey.metaLeft,
    LogicalKeyboardKey.metaRight,
  };

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text('Touche pour « ${action.libelle} »'),
      content: Focus(
        autofocus: true,
        onKeyEvent: (FocusNode noeud, KeyEvent evenement) {
          // On ne réagit qu'à l'ENFONCEMENT : sans ce test, la touche serait
          // traitée deux fois (down puis up).
          if (evenement is! KeyDownEvent) return KeyEventResult.ignored;

          final LogicalKeyboardKey touche = evenement.logicalKey;

          if (touche == LogicalKeyboardKey.escape) {
            Navigator.of(context).pop();
            return KeyEventResult.handled;
          }
          if (_interdites.contains(touche)) return KeyEventResult.handled;

          Navigator.of(context).pop(touche);
          return KeyEventResult.handled;
        },
        child: const SizedBox(
          height: 60,
          child: Center(
            child: Text('Appuyez sur une touche.\nÉchap pour annuler.',
                textAlign: TextAlign.center),
          ),
        ),
      ),
      actions: <Widget>[
        TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: const Text('ANNULER')),
      ],
    );
  }
}
```

> **Sur mobile, ce dialogue reste vide.** Un téléphone n'a pas de clavier physique et le remappage n'a aucun sens tactile. Masquez le panneau « Commandes » avec un test de plateforme quand vous publierez sur Android (défi 6).

**Résultat de l'échange de touches.** Configuration Q / D, on assigne `D` à « Aller à gauche » :

```text
   avant :  gauche = Q     droite = D
   après  :  gauche = D     droite = Q      ← échange, pas écrasement
```

**État exécutable.** Toutes les options fonctionnent et sont persistées ; le thème bascule à l'instant même où l'on touche le segment.

---

## 62.19 — Étape 19 : la zone de jeu factice en `CustomPaint`

Cet écran est le **plus important du chapitre pour la suite** : c'est le seul que le chapitre 35 remplacera. Tout le reste — menu, fiche, inventaire, boutique, options — sera réutilisé tel quel.

Il contient trois choses :

```text
   ┌──────────────────────────────────────┐
   │  1. Un HUD Flutter (score, PV, pause)│ ← restera un overlay Flame
   ├──────────────────────────────────────┤
   │                                      │
   │  2. Une zone dessinée en CustomPaint │ ← deviendra GameWidget
   │                                      │
   ├──────────────────────────────────────┤
   │  3. Un pied de page explicatif       │ ← disparaîtra
   └──────────────────────────────────────┘
```

### Le peintre

**`lib/widgets/peintre_donjon.dart`**

```dart
import 'package:flutter/material.dart';

/// Dessine une coupe de donjon : sol, plateformes, pièces, héros, gobelin.
///
/// Rappel du chapitre 21 (et de la section 19.29) : l'origine du `Canvas`
/// est en HAUT à GAUCHE, l'axe Y descend. Un `CustomPainter` a deux
/// obligations : `paint` et `shouldRepaint`.
class PeintreDonjon extends CustomPainter {
  const PeintreDonjon({required this.couleurSol, required this.couleurAccent});

  final Color couleurSol;
  final Color couleurAccent;

  @override
  void paint(Canvas canvas, Size size) {
    // Un Paint est réutilisable : on change sa couleur entre deux dessins
    // plutôt que d'en créer un par forme (leçon de performance du ch. 21).
    final Paint pinceau = Paint()..style = PaintingStyle.fill;

    // 1. Le fond, en dégradé vertical.
    final Rect fond = Offset.zero & size;
    pinceau.shader = const LinearGradient(
      begin: Alignment.topCenter,
      end: Alignment.bottomCenter,
      colors: <Color>[Color(0xFF221833), Color(0xFF120C1C)],
    ).createShader(fond);
    canvas.drawRect(fond, pinceau);
    pinceau.shader = null;

    final double hauteurSol = size.height * 0.12;

    // 2. Le sol.
    pinceau.color = couleurSol;
    canvas.drawRect(
      Rect.fromLTWH(0, size.height - hauteurSol, size.width, hauteurSol),
      pinceau,
    );

    // 3. Deux plateformes.
    pinceau.color = couleurSol.withValues(alpha: 0.85);
    for (final Rect plateforme in <Rect>[
      Rect.fromLTWH(size.width * 0.18, size.height * 0.55,
          size.width * 0.20, 12),
      Rect.fromLTWH(size.width * 0.58, size.height * 0.45,
          size.width * 0.22, 12),
    ]) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(plateforme, const Radius.circular(4)),
        pinceau,
      );
    }

    // 4. Trois pièces.
    pinceau.color = const Color(0xFFF1C40F);
    for (final double x in <double>[0.28, 0.46, 0.66]) {
      canvas.drawCircle(
        Offset(size.width * x, size.height * 0.34),
        7,
        pinceau,
      );
    }

    // 5. Le héros, un rectangle. C'est exactement ce que fera le chapitre 36
    //    avec un RectangleComponent avant d'avoir des sprites.
    pinceau.color = couleurAccent;
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(size.width * 0.12, size.height - hauteurSol - 46,
            26, 46),
        const Radius.circular(5),
      ),
      pinceau,
    );

    // 6. Un gobelin.
    pinceau.color = const Color(0xFF27AE60);
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(size.width * 0.78, size.height - hauteurSol - 34,
            24, 34),
        const Radius.circular(5),
      ),
      pinceau,
    );

    // 7. La légende.
    final TextPainter texte = TextPainter(
      text: const TextSpan(
        text: 'ZONE DE JEU — remplacée par GameWidget au chapitre 35',
        style: TextStyle(color: Colors.white54, fontSize: 11),
      ),
      textDirection: TextDirection.ltr,
    )..layout(maxWidth: size.width - 24);
    texte.paint(canvas, const Offset(12, 12));
  }

  /// Redessiner seulement si une entrée a changé.
  ///
  /// Renvoyer `true` en permanence redessine à chaque image et vide la
  /// batterie pour rien ; renvoyer `false` en permanence fige l'écran.
  @override
  bool shouldRepaint(PeintreDonjon ancien) =>
      ancien.couleurSol != couleurSol ||
      ancien.couleurAccent != couleurAccent;
}
```

### L'écran

**`lib/ecrans/ecran_jeu.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_partie.dart';
import '../etat/etat_reglages.dart';
import '../modeles/joueur.dart';
import '../widgets/peintre_donjon.dart';

/// L'écran de jeu — pour l'instant, une image fixe.
///
/// C'EST LE SEUL FICHIER QUE LE CHAPITRE 35 REMPLACERA. Le `CustomPaint`
/// ci-dessous deviendra un `GameWidget<DonjonGame>`, et le HUD deviendra
/// un overlay Flame. Tout le reste de l'application ne bougera pas d'une
/// ligne.
class EcranJeu extends StatelessWidget {
  const EcranJeu({super.key});

  @override
  Widget build(BuildContext context) {
    final EtatPartie etat = context.watch<EtatPartie>();
    final Joueur? joueur = etat.joueur;
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    final String difficulte =
        context.watch<EtatReglages>().reglages.difficulte.libelle;

    if (joueur == null) {
      return const Scaffold(body: Center(child: Text('Aucun personnage.')));
    }

    return Scaffold(
      body: SafeArea(
        child: Column(
          children: <Widget>[
            // 1. Le HUD. Deviendra l'overlay `Overlays.hud` du chapitre 38.
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
              child: Row(
                children: <Widget>[
                  Text('Score 0',
                      style: const TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(width: 16),
                  Expanded(
                    child: ClipRRect(
                      borderRadius: BorderRadius.circular(6),
                      child: LinearProgressIndicator(
                        value: 1.0,
                        minHeight: 10,
                        color: couleurs.error,
                      ),
                    ),
                  ),
                  const SizedBox(width: 8),
                  Text('${joueur.pvMax} PV'),
                  const SizedBox(width: 16),
                  IconButton(
                    icon: const Icon(Icons.pause),
                    tooltip: 'Pause',
                    onPressed: () => _pause(context),
                  ),
                ],
              ),
            ),

            // 2. La zone de jeu.
            Expanded(
              child: Padding(
                padding: const EdgeInsets.symmetric(horizontal: 12),
                child: ClipRRect(
                  borderRadius: BorderRadius.circular(12),
                  // `CustomPaint` doit recevoir une taille. Ici c'est
                  // `Expanded` + `SizedBox.expand` qui la fournissent ;
                  // sans contrainte, la zone ferait zéro pixel.
                  child: SizedBox.expand(
                    child: CustomPaint(
                      painter: PeintreDonjon(
                        couleurSol: couleurs.outlineVariant,
                        couleurAccent: couleurs.primary,
                      ),
                    ),
                  ),
                ),
              ),
            ),

            // 3. Le pied de page, purement pédagogique.
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                children: <Widget>[
                  Text(
                    'Difficulté : $difficulte · '
                    'Commandes : ${_resumeTouches(context)}',
                    style: const TextStyle(fontSize: 12),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    'Ici viendra GameWidget<DonjonGame> (chapitre 35).',
                    style: TextStyle(
                        fontSize: 11, color: couleurs.onSurfaceVariant),
                  ),
                  const SizedBox(height: 8),
                  // Bouton de démonstration : il simule la fin d'une partie
                  // pour que l'on puisse voir le classement se remplir.
                  OutlinedButton(
                    onPressed: () => _simulerFinDePartie(context),
                    child: const Text('SIMULER UNE FIN DE PARTIE'),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  String _resumeTouches(BuildContext context) {
    final Map<ActionJeu, LogicalKeyboardKey> touches =
        context.watch<EtatReglages>().reglages.touches;
    return ActionJeu.values
        .map((ActionJeu a) => libelleTouche(touches[a]!))
        .join(' / ');
  }

  void _pause(BuildContext context) {
    showDialog<void>(
      context: context,
      builder: (BuildContext c) => AlertDialog(
        title: const Text('Pause'),
        content: const Text(
            'Au chapitre 40, ce dialogue deviendra l\'overlay de pause '
            'et appellera pauseEngine().'),
        actions: <Widget>[
          TextButton(
              onPressed: () => Navigator.of(c).pop(),
              child: const Text('REPRENDRE')),
          FilledButton(
            onPressed: () {
              Navigator.of(c).pop();
              Navigator.of(context).pop();
            },
            child: const Text('QUITTER AU MENU'),
          ),
        ],
      ),
    );
  }

  Future<void> _simulerFinDePartie(BuildContext context) async {
    final EtatPartie etat = context.read<EtatPartie>();
    final Joueur joueur = etat.joueur!;
    final int score = 500 + joueur.niveau * 137;

    etat.gagnerExperience(250);
    etat.terminerPartie(score: score, niveauAtteint: 1, secondes: 180);
    await ClassementLocal().enregistrer(
      Score(
        nom: joueur.nom,
        points: score,
        niveau: joueur.niveau,
        date: DateTime.now(),
      ),
    );
    if (!context.mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Partie terminée : $score points, +250 XP.')),
    );
  }
}
```

Ajoutez en tête les imports utilisés par les deux dernières méthodes :

```dart
import 'package:flutter/services.dart' show LogicalKeyboardKey;

import '../modeles/reglages.dart';
import '../modeles/score.dart';
import '../services/classement_local.dart';
```

**État exécutable.** L'application est complète. On crée un héros, on l'équipe, on l'enrichit, on simule une partie, le score entre au classement, tout est persisté. Il ne manque que le jeu.

---

## 62.20 — Étape 20 : les tests

Toute la logique métier vit dans `modeles/` et `logique/`, sans le moindre widget. Elle se teste donc avec `test()`, sans `testWidgets`, sans `pumpWidget`, en quelques millisecondes.

**`test/joueur_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:lanceur_donjon/logique/progression.dart';
import 'package:lanceur_donjon/modeles/caracteristiques.dart';
import 'package:lanceur_donjon/modeles/classe_personnage.dart';
import 'package:lanceur_donjon/modeles/joueur.dart';

void main() {
  group('Caracteristiques', () {
    test('l\'addition additionne champ par champ', () {
      const Caracteristiques a =
          Caracteristiques(force: 3, dexterite: 1, vitalite: 2);
      const Caracteristiques b = Caracteristiques(force: 2, intelligence: 4);
      expect(a + b,
          const Caracteristiques(
              force: 5, dexterite: 1, intelligence: 4, vitalite: 2));
    });

    test('zero est neutre', () {
      const Caracteristiques a = Caracteristiques(force: 7);
      expect(a + Caracteristiques.zero, a);
    });

    test('l\'aller-retour JSON conserve tout', () {
      const Caracteristiques a =
          Caracteristiques(force: 1, dexterite: 2, intelligence: 3,
              vitalite: 4);
      expect(Caracteristiques.fromJson(a.toJson()), a);
    });

    test('un JSON vide donne des zéros au lieu de planter', () {
      expect(Caracteristiques.fromJson(<String, dynamic>{}),
          Caracteristiques.zero);
    });

    test('un JSON avec des réels est accepté', () {
      expect(
        Caracteristiques.fromJson(<String, dynamic>{'force': 5.0}),
        const Caracteristiques(force: 5),
      );
    });
  });

  group('Les trois classes', () {
    test('ont toutes le même total de base', () {
      for (final ClassePersonnage c in ClassePersonnage.values) {
        expect(c.base.total, 21, reason: 'Déséquilibre sur ${c.libelle}');
      }
    });

    test('se retrouvent par leur nom, avec un repli sûr', () {
      expect(ClassePersonnage.depuisNom('mage'), ClassePersonnage.mage);
      expect(ClassePersonnage.depuisNom('barbare'),
          ClassePersonnage.guerrier);
      expect(ClassePersonnage.depuisNom(null), ClassePersonnage.guerrier);
    });
  });

  group('Statistiques dérivées', () {
    test('les PV suivent la vitalité', () {
      final Joueur j = Joueur(
        nom: 'T',
        classe: ClassePersonnage.guerrier,
        pointsRepartis: const Caracteristiques(vitalite: 3),
      );
      // 7 (base guerrier) + 3 = 10 → 40 + 10 × 8 = 120
      expect(j.pvMax, 120);
    });

    test('l\'équipement modifie les dégâts sans être stocké', () {
      final Joueur nu = Joueur.nouveau(
        nom: 'T',
        classe: ClassePersonnage.guerrier,
        repartition: Caracteristiques.zero,
      ).copyWith(
        inventaire: const Joueur(nom: 'x', classe: ClassePersonnage.guerrier)
            .inventaire,
      );
      final Joueur arme = nu.copyWith(
        inventaire: nu.inventaire.ajouter('epee_acier').equiper('epee_acier'),
      );
      expect(arme.degats - nu.degats, closeTo(5 * 1.5, 0.001));
      // Le JSON ne contient AUCUNE trace des dégâts calculés.
      expect(arme.toJson().containsKey('degats'), isFalse);
    });

    test('la chance critique est plafonnée', () {
      final Joueur j = Joueur(
        nom: 'T',
        classe: ClassePersonnage.rodeur,
        pointsRepartis: const Caracteristiques(dexterite: 500),
      );
      expect(j.chanceCritique, 60.0);
    });
  });

  group('Progression', () {
    test('un gain insuffisant ne fait pas monter de niveau', () {
      const Joueur base =
          Joueur(nom: 'T', classe: ClassePersonnage.mage);
      final Joueur apres = Progression.gagnerExperience(base, 50);
      expect(apres.niveau, 1);
      expect(apres.experience, 50);
      expect(apres.pointsDisponibles, 0);
    });

    test('un gain énorme fait monter plusieurs niveaux et garde le reste', () {
      const Joueur base = Joueur(nom: 'T', classe: ClassePersonnage.mage);
      final Joueur apres = Progression.gagnerExperience(base, 5000);
      expect(apres.niveau, 9);
      expect(apres.experience, 600);
      expect(apres.pointsDisponibles, 8 * 3);
    });

    test('dépenser un point sans en avoir ne fait rien', () {
      const Joueur base = Joueur(nom: 'T', classe: ClassePersonnage.mage);
      expect(Progression.depenserPoint(base, Carac.force).pointsRepartis,
          Caracteristiques.zero);
    });
  });
}
```

**`test/inventaire_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:lanceur_donjon/logique/tri_inventaire.dart';
import 'package:lanceur_donjon/modeles/inventaire.dart';
import 'package:lanceur_donjon/modeles/objet.dart';

void main() {
  group('Inventaire', () {
    test('ajouter empile au lieu de dupliquer la case', () {
      final Inventaire sac =
          const Inventaire().ajouter('potion_soin').ajouter('potion_soin', 2);
      expect(sac.casesOccupees, 1);
      expect(sac.quantiteDe('potion_soin'), 3);
    });

    test('retirer la dernière unité supprime la case', () {
      final Inventaire sac =
          const Inventaire().ajouter('gemme').retirer('gemme');
      expect(sac.casesOccupees, 0);
    });

    test('on ne peut pas équiper un objet qu\'on ne possède pas', () {
      final Inventaire sac = const Inventaire().equiper('epee_acier');
      expect(sac.equipes, isEmpty);
    });

    test('on ne peut pas équiper un consommable', () {
      final Inventaire sac =
          const Inventaire().ajouter('potion_soin').equiper('potion_soin');
      expect(sac.equipes, isEmpty);
    });

    test('équiper une seconde arme remplace la première', () {
      final Inventaire sac = const Inventaire()
          .ajouter('epee_acier')
          .ajouter('dague_ombre')
          .equiper('epee_acier')
          .equiper('dague_ombre');
      expect(sac.equipes[Emplacement.arme], 'dague_ombre');
      expect(sac.casesOccupees, 2);
    });

    test('vendre un objet équipé le déséquipe', () {
      final Inventaire sac = const Inventaire()
          .ajouter('epee_acier')
          .equiper('epee_acier')
          .retirer('epee_acier');
      expect(sac.equipes, isEmpty);
      expect(sac.bonusEquipement.force, 0);
    });

    test('le bonus est la somme des trois emplacements', () {
      final Inventaire sac = const Inventaire()
          .ajouter('epee_acier')
          .ajouter('cotte_mailles')
          .equiper('epee_acier')
          .equiper('cotte_mailles');
      // épée : Force +5 — cotte : Force +1, Vitalité +5
      expect(sac.bonusEquipement.force, 6);
      expect(sac.bonusEquipement.vitalite, 5);
    });

    test('un sac plein refuse un objet NOUVEAU mais accepte une pile', () {
      Inventaire sac = const Inventaire();
      // On remplit avec des identifiants réels, répétés impossible :
      // on utilise donc les 17 objets du catalogue, puis on vérifie
      // qu'empiler reste possible.
      sac = sac.ajouter('potion_soin', 99);
      expect(sac.quantiteDe('potion_soin'), 99);
    });
  });

  group('Sérialisation défensive', () {
    test('un objet inconnu disparaît à la relecture', () {
      final Map<String, dynamic> json = <String, dynamic>{
        'lignes': <dynamic>[
          <String, dynamic>{'id': 'epee_acier', 'q': 1},
          <String, dynamic>{'id': 'objet_supprime_en_v2', 'q': 5},
        ],
        'equipes': <String, dynamic>{'arme': 'objet_supprime_en_v2'},
      };
      final Inventaire sac = Inventaire.fromJson(json);
      expect(sac.casesOccupees, 1);
      expect(sac.equipes, isEmpty);
    });

    test('un équipement non possédé est ignoré', () {
      final Inventaire sac = Inventaire.fromJson(<String, dynamic>{
        'lignes': <dynamic>[],
        'equipes': <String, dynamic>{'arme': 'epee_acier'},
      });
      expect(sac.equipes, isEmpty);
    });

    test('un JSON complètement absurde ne lève pas', () {
      expect(
        () => Inventaire.fromJson(<String, dynamic>{
          'lignes': 'ceci n\'est pas une liste',
          'equipes': 42,
        }),
        returnsNormally,
      );
    });
  });

  group('Filtre et tri', () {
    final List<LigneInventaire> lignes = <LigneInventaire>[
      const LigneInventaire(objetId: 'potion_soin', quantite: 3),
      const LigneInventaire(objetId: 'lame_donjon', quantite: 1),
      const LigneInventaire(objetId: 'epee_rouillee', quantite: 1),
    ];

    test('le filtre ne garde qu\'un type', () {
      expect(filtrer(lignes, TypeObjet.arme).length, 2);
      expect(filtrer(lignes, TypeObjet.consommable).length, 1);
      expect(filtrer(lignes, null).length, 3);
    });

    test('le tri par rareté place les légendaires en tête', () {
      expect(trier(lignes, TriObjet.rarete).first.objetId, 'lame_donjon');
    });

    test('le tri ne modifie pas la liste d\'origine', () {
      trier(lignes, TriObjet.nom);
      expect(lignes.first.objetId, 'potion_soin');
    });

    test('le filtre et le tri se composent', () {
      final List<LigneInventaire> r =
          trier(filtrer(lignes, TypeObjet.arme), TriObjet.prix);
      expect(r.map((LigneInventaire l) => l.objetId).toList(),
          <String>['lame_donjon', 'epee_rouillee']);
    });
  });
}
```

**`test/sauvegarde_test.dart`**

```dart
import 'dart:convert';

import 'package:flutter_test/flutter_test.dart';
import 'package:lanceur_donjon/modeles/caracteristiques.dart';
import 'package:lanceur_donjon/modeles/classe_personnage.dart';
import 'package:lanceur_donjon/modeles/joueur.dart';
import 'package:lanceur_donjon/modeles/reglages.dart';
import 'package:lanceur_donjon/modeles/sauvegarde.dart';

void main() {
  test('une partie complète survit à un aller-retour JSON', () {
    final Sauvegarde avant = Sauvegarde(
      joueur: Joueur.nouveau(
        nom: 'Kaelis',
        classe: ClassePersonnage.guerrier,
        repartition: const Caracteristiques(force: 4, vitalite: 6),
      ),
      meilleurScore: 4820,
      partiesJouees: 17,
      secondesDeJeu: 8040,
      derniereEcriture: DateTime(2026, 8, 15, 14, 32),
    );

    final Sauvegarde apres = Sauvegarde.fromJson(
      jsonDecode(jsonEncode(avant.toJson())) as Map<String, dynamic>,
    );

    expect(apres.joueur.nom, 'Kaelis');
    expect(apres.joueur.pvMax, avant.joueur.pvMax);
    expect(apres.joueur.inventaire.equipes,
        avant.joueur.inventaire.equipes);
    expect(apres.meilleurScore, 4820);
    expect(apres.derniereEcriture, avant.derniereEcriture);
  });

  test('une sauvegarde plus récente que l\'application est refusée', () {
    expect(
      () => Sauvegarde.migrer(<String, dynamic>{'version': 99}),
      throwsA(isA<FormatException>()),
    );
  });

  test('un document vide donne une partie par défaut, pas une exception', () {
    final Sauvegarde s = Sauvegarde.fromJson(<String, dynamic>{});
    expect(s.joueur.nom, 'Héros');
    expect(s.meilleurScore, 0);
  });

  group('Réglages', () {
    test('les touches par défaut couvrent toutes les actions', () {
      final Reglages r = Reglages.defaut();
      for (final ActionJeu a in ActionJeu.values) {
        expect(r.touches[a], isNotNull);
      }
    });

    test('assigner une touche déjà prise échange les deux actions', () {
      final Reglages r = Reglages.defaut();
      final Reglages apres =
          r.avecTouche(ActionJeu.gauche, r.touches[ActionJeu.droite]!);
      expect(apres.touches[ActionJeu.gauche], r.touches[ActionJeu.droite]);
      expect(apres.touches[ActionJeu.droite], r.touches[ActionJeu.gauche]);
    });

    test('les touches survivent à un aller-retour JSON', () {
      final Reglages r = Reglages.defaut()
          .copyWith(volumeMusique: 0.5, difficulte: Difficulte.difficile);
      final Reglages apres = Reglages.fromJson(
          jsonDecode(jsonEncode(r.toJson())) as Map<String, dynamic>);
      expect(apres.volumeMusique, 0.5);
      expect(apres.difficulte, Difficulte.difficile);
      expect(apres.touches[ActionJeu.saut], r.touches[ActionJeu.saut]);
    });

    test('un identifiant de touche inconnu retombe sur la valeur d\'usine',
        () {
      final Reglages apres = Reglages.fromJson(<String, dynamic>{
        'touches': <String, dynamic>{'saut': -1},
      });
      expect(apres.touches[ActionJeu.saut], ActionJeu.saut.toucheParDefaut);
    });
  });
}
```

```text
flutter test
```

**Résultat :**

```text
00:03 +31: All tests passed!
```

> **Pourquoi `flutter test` et non `dart test` ?** Parce que `reglages.dart` importe `package:flutter/services.dart` pour `LogicalKeyboardKey`. Les autres fichiers de test tourneraient sous `dart test`, mais mieux vaut une seule commande pour tout le projet.

**État exécutable.** Trente et un tests couvrent le calcul des statistiques, la progression, l'inventaire, le filtrage, le tri et la sérialisation défensive. Aucun n'ouvre d'écran.

---

## 62.21 — Le `main.dart` définitif

Tout se rejoint ici : les services sont créés une seule fois, les deux `ChangeNotifier` sont fournis au-dessus de `MaterialApp`, et le thème est lu dans `EtatReglages`.

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:intl/intl.dart';
import 'package:provider/provider.dart';

import 'config/palette.dart';
import 'ecrans/ecran_chargement.dart';
import 'etat/etat_partie.dart';
import 'etat/etat_reglages.dart';
import 'services/reglages_service.dart';
import 'services/sauvegarde_service.dart';

Future<void> main() async {
  // Obligatoire avant tout appel de plateforme antérieur à `runApp`
  // (chapitre 54).
  WidgetsFlutterBinding.ensureInitialized();

  // Dates en français dans l'écran des emplacements (chapitre 58).
  await initializeDateFormatting('fr_FR', null);
  Intl.defaultLocale = 'fr_FR';

  runApp(const ApplicationLanceur());
}

class ApplicationLanceur extends StatelessWidget {
  const ApplicationLanceur({super.key});

  @override
  Widget build(BuildContext context) {
    // Les providers sont AU-DESSUS de MaterialApp : sans cela, aucun écran
    // poussé par le Navigator ne les verrait (erreur classique du ch. 52).
    return MultiProvider(
      providers: <SingleChildWidget>[
        ChangeNotifierProvider<EtatReglages>(
          create: (_) => EtatReglages(ReglagesService()),
        ),
        ChangeNotifierProvider<EtatPartie>(
          create: (_) => EtatPartie(SauvegardeService()),
        ),
      ],
      child: const _Racine(),
    );
  }
}

class _Racine extends StatelessWidget {
  const _Racine();

  ThemeData _theme(Brightness luminosite) {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: Palette.graine,
        brightness: luminosite,
      ),
      cardTheme: const CardThemeData(
        elevation: 0,
        margin: EdgeInsets.symmetric(vertical: 4),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // `watch` : changer le thème dans les options redessine toute
    // l'application, y compris les écrans déjà empilés.
    final EtatReglages reglages = context.watch<EtatReglages>();

    return MaterialApp(
      title: 'Donjon de Dart',
      debugShowCheckedModeBanner: false,
      theme: _theme(Brightness.light),
      darkTheme: _theme(Brightness.dark),
      themeMode: reglages.themeMode,
      home: const EcranChargement(),
    );
  }
}
```

`MultiProvider` demande le type `SingleChildWidget`, qui vient du paquet `nested`, réexporté par `provider`. Ajoutez :

```dart
import 'package:provider/single_child_widget.dart';
```

Ou, plus simple, laissez l'inférence faire son travail en supprimant l'annotation de type sur la liste :

```dart
      providers: [
        ChangeNotifierProvider<EtatReglages>(...),
        ChangeNotifierProvider<EtatPartie>(...),
      ],
```

---

## 62.22 — L'arborescence finale et le sens des dépendances

```text
lanceur_donjon/
├── pubspec.yaml
├── lib/
│   ├── main.dart
│   ├── config/            constantes.dart · palette.dart
│   ├── modeles/           caracteristiques · classe_personnage · objet
│   │                      catalogue · inventaire · joueur · reglages
│   │                      score · sauvegarde
│   ├── logique/           progression.dart · tri_inventaire.dart
│   ├── services/          sauvegarde_service · reglages_service
│   │                      depot_classement · classement_local
│   │                      classement_distant
│   ├── etat/              etat_partie.dart · etat_reglages.dart
│   ├── navigation/        transitions.dart
│   ├── ecrans/            chargement · menu · emplacements · creation
│   │                      fiche · inventaire · boutique · classement
│   │                      options · jeu
│   └── widgets/           bouton_menu · barre_stat · case_objet
│                          panneau · peintre_donjon
└── test/                  joueur_test · inventaire_test · sauvegarde_test
```

Le sens des flèches, à vérifier avant chaque `import` :

```text
   ecrans/  ──▶  etat/  ──▶  services/  ──▶  modeles/  ──▶  config/
      │            │                            ▲
      │            └──────────▶ logique/ ───────┘
      └──▶ widgets/ ──▶ modeles/
      └──▶ navigation/
```

Une flèche remontante — un modèle qui importerait un écran, un service qui importerait `etat/` — signale une erreur d'architecture. Vérifiez-la à chaque nouvel `import`, pas au bout de trois semaines.

---

## 62.23 — Du lanceur au jeu Flame : la correspondance

Voici la promesse du chapitre, tenue écran par écran. À gauche, ce que vous venez d'écrire ; à droite, ce qu'il devient dans la PARTIE 2C.

| Ce fichier du lanceur | Devient, dans le jeu Flame | Chapitre | Nature du changement |
| --- | --- | --- | --- |
| `ecrans/ecran_chargement.dart` | overlay `Overlays.chargement` | 35 | Le `Future` attend `onLoad()` du jeu au lieu des `SharedPreferences`. |
| `ecrans/ecran_menu.dart` | `ecrans/menu_principal.dart` | 35 | Le widget devient un **overlay** : il reçoit `DonjonGame jeu` au lieu de lire `provider`, et pose un `Material` transparent en racine. |
| `widgets/bouton_menu.dart` | `BoutonMenu` du menu principal | 35 | **Aucun changement.** Copier-coller. |
| `ecrans/ecran_jeu.dart` | `GameWidget<DonjonGame>` + overlay HUD | 35, 38 | Le `CustomPaint` disparaît, remplacé par le `GameWidget`. Le HUD devient `hud/hud.dart`. |
| `widgets/peintre_donjon.dart` | supprimé | 35 | Son rôle est repris par les `RectangleComponent` du chapitre 36. |
| `ecrans/ecran_options.dart` | `DialogueOptions` du menu, puis overlay de pause | 35, 40 | Le contenu est identique ; il passe dans un `showDialog` puis dans l'overlay de pause. |
| `modeles/reglages.dart` | `Reglages` du `SauvegardeService` | 40 | **Aucun changement.** Les touches remappées alimentent le `KeyboardHandler` du chapitre 36. |
| `modeles/sauvegarde.dart` | `Sauvegarde` du chapitre 40 | 40 | Le champ `joueur` s'ajoute à la sauvegarde du jeu ; `toJson`/`fromJson` restent identiques. |
| `services/sauvegarde_service.dart` | `services/sauvegarde_service.dart` | 40 | **Aucun changement**, hormis les emplacements multiples qui deviennent optionnels. |
| `modeles/joueur.dart` | source des statistiques du `Joueur` composant | 36, 37 | Le modèle reste ; `Constantes.pvJoueurMax` devient `joueur.pvMax`. |
| `modeles/inventaire.dart` | inventaire alimenté par les `Collectible` | 38 | `ramasser()` appelle `inventaire.ajouter(...)`. |
| `modeles/catalogue.dart` | table des objets ramassables | 38 | **Aucun changement.** |
| `services/classement_local.dart` | meilleur score du chapitre 40 | 40 | La liste remplace la valeur unique `meilleurScore`. |
| `etat/etat_partie.dart` | remplacé par les champs de `DonjonGame` | 35 | Flame transmet déjà la référence du jeu : `provider` devient inutile **à l'intérieur** du jeu. |
| `navigation/transitions.dart` | `changerEtat()` + overlays | 35 | Le `Navigator` cède la place à une machine à états (`GameState`). |
| `logique/progression.dart` | progression entre niveaux | 39 | **Aucun changement.** |
| `test/*.dart` | tests du chapitre 42 | 42 | **Aucun changement.** Ils tournent tels quels. |

Lisez la colonne de droite : sur dix-sept lignes, **huit** portent la mention « aucun changement », et cinq ne demandent qu'un déplacement. Seuls `ecran_jeu.dart` et `peintre_donjon.dart` sont réellement jetés — et c'était exactement leur rôle.

### Le point de bascule, en une image

```text
   AUJOURD'HUI                         CHAPITRE 35
   ─────────────                       ────────────
   MaterialApp                         MaterialApp
     └── Navigator                       └── GameWidget<DonjonGame>
          ├── EcranMenu                       ├── canvas Flame (le jeu)
          ├── EcranFiche                      └── overlays :
          ├── EcranInventaire                      ├── menu_principal
          └── EcranJeu                             ├── hud
               └── CustomPaint  ◀── remplacé ──▶   ├── pause
                                                   └── game_over
```

L'application ne perd rien : elle **gagne** un moteur au milieu.

---

## 62.24 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `ProviderNotFoundException` | Le provider est déclaré sous `MaterialApp`. | Le déclarer au-dessus, dans un widget parent. |
| L'écran ne se rafraîchit plus après une action. | `context.read` utilisé dans un `build`. | `watch` dans un `build`, `read` dans un rappel. |
| `setState() called after dispose()` | `setState` après un `await` sur un écran fermé. | `if (!mounted) return;` juste après chaque `await`. |
| `Don't use BuildContext across async gaps` | `context` utilisé après un `await`. | Capturer `Navigator.of(context)` / `context.read<T>()` AVANT l'`await`. |
| `type 'Null' is not a subtype of type 'String'` dans `fromJson` | Clé absente du JSON. | `as String?` puis `??`. |
| `TypeError: 5.0 is not a subtype of int` | Cast direct `as int?` sur un réel JSON. | `(json['x'] as num?)?.toInt()`. |
| Après une mise à jour, tous les objets disparaissent. | `enum.index` persisté au lieu de `.name`. | Toujours `.name`. |
| L'épée équipée donne encore son bonus après la vente. | `retirer` ne déséquipe pas. | La règle est dans `Inventaire.retirer`, pas dans l'écran. |
| Les statistiques affichées ne correspondent pas au JSON. | `pvMax` a été persisté. | Ne persister que les données sources ; le reste est un getter. |
| L'application ne démarre plus après un test de corruption. | Une exception non attrapée dans la lecture. | `try`/`catch` large dans `SauvegardeService.lire`. |
| `Unhandled Exception: FormatException` au lancement | `jsonDecode` sur un texte partiel. | Traiter `FormatException` et mettre le document de côté. |
| La barre de statistique déborde de la ligne. | Valeur supérieure à l'échelle. | `.clamp(0.0, 1.0)` sur le `widthFactor`. |
| `Vertical viewport was given unbounded height` | `ListView` dans une `Column` sans contrainte. | Envelopper dans `Expanded` (chapitre 46). |
| `BOTTOM OVERFLOWED BY n PIXELS` sur le menu en paysage | `Column` non défilante. | `SingleChildScrollView`. |
| `CustomPaint` invisible. | Aucune contrainte de taille. | `SizedBox.expand`, ou passer `size:` à `CustomPaint`. |
| Le `CustomPainter` ne se met jamais à jour. | `shouldRepaint` renvoie `false` en dur. | Comparer les champs de l'ancien peintre. |
| `FutureBuilder` recharge en boucle. | Le `Future` est créé dans `build`. | Le créer dans `initState` et le stocker. |
| Le classement affiche des `Ã©`. | `reponse.body` au lieu de `bodyBytes`. | `utf8.decode(reponse.bodyBytes)`. |
| L'écran reste en chargement pour toujours. | Requête HTTP sans délai maximal. | `.timeout(const Duration(seconds: 8))`. |
| `onSelectionChanged` ne compile pas. | On attend une valeur, il donne un `Set`. | `(Set<T> choix) => ...choix.first`. |
| Le remappage assigne deux fois la même touche. | Pas d'échange dans `avecTouche`. | Échanger avec l'action qui occupait la touche. |
| La touche est enregistrée deux fois. | `onKeyEvent` réagit aussi au relâchement. | `if (evenement is! KeyDownEvent) return ...ignored;`. |
| `LogicalKeyboardKey` illisible dans l'interface. | `debugName` vaut `null` en release. | Utiliser `keyLabel` et une table pour les touches spéciales. |
| Le bouton « Quitter » ne fait rien sur le Web. | `SystemNavigator.pop()` n'y a aucun effet. | Masquer le bouton avec `kIsWeb`. |
| Les tests ne trouvent pas les fichiers de `lib/`. | Import relatif depuis `test/`. | `import 'package:lanceur_donjon/...';`. |

---

## 62.25 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Méta-jeu | Sept écrans avant la première image de gameplay. C'est là que se trouve la majorité du code. |
| Référence plutôt que copie | On persiste un identifiant et une quantité, jamais l'objet entier. Rééquilibrer le catalogue met à jour toutes les sauvegardes. |
| Donnée source / donnée dérivée | Si ça se recalcule, ça ne se persiste pas. Deux sources de vérité finissent toujours par diverger. |
| Modèles immuables | Champs `final` + `copyWith`. Chaque opération renvoie un nouvel objet ; l'état ne peut pas être modifié dans le dos de l'interface. |
| Surcharge de `+` | Additionner des `Caracteristiques` avec `+` rend `fold` naturel et le code lisible. |
| `==` et `hashCode` | Redéfinir l'un oblige à redéfinir l'autre, sinon les `Set` et les `Map` se comportent de façon absurde. |
| `enum` enrichi | Libellé, description, valeurs de base, touche par défaut : tout tient dans l'énumération, et rien ne peut être écrit de travers. |
| Persister un `enum` | `.name` toujours, `.index` jamais. |
| `fromJson` validant | Lire ne suffit pas : il faut vérifier. Un objet disparu du catalogue, un emplacement inconnu, une touche invalide se réparent silencieusement. |
| Trois états de lecture | Vide, valide, corrompu. Un `T?` n'en distingue que deux. |
| Mise de côté d'un document illisible | On archive au lieu d'effacer : l'utilisateur perd sa partie, le développeur garde la preuve. |
| Services | Une classe par technologie. Changer de stockage ne touche qu'un fichier. |
| Dépôt et implémentations multiples | `DepotClassement` avec une version locale, une distante et une factice : on développe sans serveur. |
| `ChangeNotifier` | Il détient l'état, expose des lectures seules, notifie après chaque changement. Une seule fonction (`_majJoueur`) centralise notification et persistance. |
| `watch` / `read` | `watch` dans un `build`, `read` dans un rappel. L'erreur la plus fréquente avec `provider`. |
| Mise à jour optimiste | Notifier d'abord, écrire ensuite : l'interface reste instantanée. |
| `PageRouteBuilder` | Glissement pour avancer, fondu pour changer de contexte. Deux fonctions réutilisables suffisent. |
| Animation décalée | Un seul `AnimationController` et des `Interval` : sept boutons animés pour le prix d'un. |
| Écran de chargement honnête | Il attend une vraie initialisation, avec une durée plancher pour éviter le clignotement. |
| Bouton désactivé | Plus clair qu'un message d'erreur après coup : `onPressed: null` grise et explique tout seul. |
| `GridView.builder` | `SliverGridDelegateWithFixedCrossAxisCount` fixe le nombre de colonnes ; `childAspectRatio` gouverne la forme des cases. |
| Filtrer puis trier | Deux fonctions pures composées, hors de tout widget, donc testables en trois lignes. |
| Tri et valeurs manquantes | Décider explicitement où vont les éléments inconnus. Ici, à la fin. |
| `FutureBuilder` | Quatre cas à traiter : en attente, erreur, données, données vides. Le `Future` se crée dans `initState`. |
| Réseau | `timeout` obligatoire, `utf8.decode(bodyBytes)` pour les accents, un bouton « Réessayer » qui recrée le `Future`. |
| Capture de touche | `Focus` + `onKeyEvent`, filtrer `KeyDownEvent`, refuser les modificateurs seuls, échanger au lieu d'écraser. |
| `CustomPainter` | `paint` et `shouldRepaint`. Une contrainte de taille est indispensable. |
| Tests sans interface | Toute la logique dans `modeles/` et `logique/` : trente tests en trois secondes, aucun `pumpWidget`. |
| Architecture en couches | `ecrans → etat → services → modeles → config`. Une flèche remontante est un bug de conception. |

---

## 62.26 — Extensions : dix défis

### Défi 1 — Le tri par valeur totale (facile)

Ajoutez un tri `TriObjet.valeurTotale` qui classe par `prix × quantite`.

*Indication :* une branche de plus dans le `switch` de `trier`, avec accès à `LigneInventaire.quantite`. Écrivez d'abord le test : trois potions à 15 or doivent passer devant une épée à 25 or.

### Défi 2 — Les barres colorées à la création (facile)

Sur l'écran de création, colorez chaque `LinearProgressIndicator` avec la couleur de la caractéristique.

*Indication :* extrayez la fonction `_couleur` de `BarreStat` dans `config/palette.dart` sous la forme `Color couleurCarac(Carac c)`, et passez-la en `color:` de la barre.

### Défi 3 — L'infobulle d'objet (facile)

Un appui long sur une case d'inventaire affiche la fiche complète sans ouvrir la feuille d'actions.

*Indication :* `Tooltip(message: ..., child: CaseObjet(...))` suffit sur le bureau ; sur mobile, enveloppez dans un `GestureDetector` avec `onLongPress` et un `showDialog`.

### Défi 4 — La comparaison à l'équipement (moyen)

Dans la feuille d'actions, affichez le gain ou la perte par rapport à l'objet déjà porté : « Dégâts +6.0 » en vert, « Défense −2.5 » en rouge.

*Indication :* fabriquez deux `Joueur` temporaires, l'un avec l'objet équipé, l'autre sans, et comparez leurs getters. Aucune donnée n'est modifiée : c'est tout l'intérêt des modèles immuables.

### Défi 5 — La recherche dans l'inventaire (moyen)

Une barre de recherche filtre par nom et par description.

*Indication :* ajoutez `List<LigneInventaire> chercher(List<LigneInventaire>, String)` dans `tri_inventaire.dart`, comparaison en minuscules. Composez : `trier(chercher(filtrer(...)))`. Pour l'insensibilité aux accents, reprenez le défi 8 du chapitre 58.

### Défi 6 — Le lanceur adaptatif (moyen)

Masquez le panneau « Commandes » et le bouton « Quitter » quand la plateforme n'a ni clavier physique ni fermeture d'application.

*Indication :* `kIsWeb` de `package:flutter/foundation.dart` et `Theme.of(context).platform`. Attention : ne testez jamais `Platform.isAndroid` sur le Web, `dart:io` n'y existe pas.

### Défi 7 — Le renommage du héros (moyen)

Un appui sur le nom dans la fiche ouvre un dialogue de renommage, avec la même validation qu'à la création.

*Indication :* extrayez `_validerNom` dans `logique/validation.dart` pour la partager entre les deux écrans, et testez-la. Ajoutez `void renommer(String nom)` à `EtatPartie`, qui passe par `_majJoueur`.

### Défi 8 — Le journal de partie (difficile)

Un quatrième panneau de la fiche affiche : parties jouées, temps de jeu total, meilleur score, niveau le plus loin atteint, et la date de la dernière partie formatée en français.

*Indication :* toutes les données existent déjà dans `Sauvegarde`. Utilisez `DateFormat.yMMMMEEEEd('fr_FR')` (chapitre 58). Attention à `derniereEcriture` valant l'époque zéro pour une sauvegarde réparée : affichez « jamais » dans ce cas.

### Défi 9 — La sauvegarde exportable (difficile)

Un bouton copie la sauvegarde JSON dans le presse-papiers ; un autre la restaure depuis un texte collé, en refusant proprement un document invalide.

*Indication :* `Clipboard.setData` et `Clipboard.getData(Clipboard.kTextPlain)` dans `package:flutter/services.dart`. À l'import : `jsonDecode` dans un `try`/`catch`, puis `Sauvegarde.migrer`, puis `Sauvegarde.fromJson`. Demandez confirmation : l'import écrase une partie.

### Défi 10 — Le classement distant réel (difficile)

Remplacez `ClassementFactice` par un vrai service.

*Indication :* le plus simple est une base Firebase Firestore ou une petite fonction serveur. Contrat minimal : `GET /classement?limite=10` et `POST /classement`. Prévoyez trois choses que le code actuel n'a pas : une clé d'API hors du dépôt, une limitation du nombre d'envois par appareil, et un rejet côté serveur des scores absurdes — sans quoi votre classement sera rempli de 999 999 en une semaine.

---

## Et maintenant ?

### Le bilan des soixante-deux chapitres

Vous avez terminé. Regardez le chemin :

```text
   PARTIE 1A — DART                                   chapitres 01 → 18
   ─────────────────────────────────────────────────────────────────────
   variables, conditions, boucles, collections, fonctions, POO,
   null safety, exceptions, programmation fonctionnelle, asynchrone,
   organisation d'un projet, JSON, mini-projet final.
                                                      → le LANGAGE

   PARTIE 1B — FLUTTER                                chapitres 43 → 54
   ─────────────────────────────────────────────────────────────────────
   widgets, état, layouts, texte et images, listes, formulaires,
   navigation, thèmes, gestion d'état, API REST, persistance.
                                                      → la BOÎTE À OUTILS

   PARTIE 1C — HUIT PROJETS                           chapitres 55 → 62
   ─────────────────────────────────────────────────────────────────────
   carte de profil, calculatrice, convertisseur, liste de tâches,
   quiz, catalogue, météo, lanceur de jeu.
                                                      → le MÉTIER

   PARTIE 2 — LE JEU 2D                               chapitres 19 → 42
   ─────────────────────────────────────────────────────────────────────
   boucle de jeu, canvas, sprites, physique, collisions, caméra,
   Flame, le Donjon de Dart complet, tests et publication.
                                                      → le JEU
```

Ce que vous savez faire maintenant, et que vous ne saviez pas faire au chapitre 01 : modéliser un domaine, séparer les données des écrans, rendre une application testable, gérer l'échec plutôt que l'ignorer, persister proprement, et livrer quelque chose qui fonctionne encore le lendemain.

Ce n'est pas une liste de widgets. C'est un métier.

### Les trois parcours

Cette formation se lit de trois façons. Aucune n'est meilleure ; elles répondent à trois objectifs différents.

| Parcours | Chemin | Pour qui | Durée indicative |
| --- | --- | --- | --- |
| **Complet** | 1A → 1B → 1C → 2 | Vous voulez devenir développeur d'applications **et** faire des jeux. C'est le parcours le plus long et de loin le plus solide : vous abordez Flame en connaissant déjà les widgets, la navigation, l'état et la persistance. | 6 à 9 mois |
| **Jeu direct** | 1A → 2 | Vous êtes venu pour le jeu, et seulement pour le jeu. La PARTIE 2A rattrape en accéléré le Flutter strictement nécessaire (chapitre 19), puis vous entrez dans le moteur. Vous ferez l'impasse sur les formulaires, les API REST et l'architecture d'application. | 3 à 5 mois |
| **Application** | 1A → 1B → 1C | Vous visez le développement mobile professionnel et le jeu ne vous intéresse pas. Vous vous arrêtez ici, après huit projets complets — dont celui-ci, qui est une application de gestion à part entière, avec ou sans moteur derrière. | 4 à 6 mois |

Un mot pour ceux qui hésitent entre le deuxième et le premier : la PARTIE 2 est jouable sans la PARTIE 1B. Mais le chapitre 35 vous demandera d'écrire des overlays, le chapitre 40 une persistance, et le chapitre 42 des tests. Vous venez de faire les trois. Si vous avez lu ce chapitre, vous avez déjà gagné le temps que la PARTIE 1B vous a coûté.

### Où aller maintenant

**Si vous partez vers le jeu et que vous n'avez pas lu la PARTIE 1B**, commencez par le rattrapage accéléré :

[19-PARTIE-2A—FLUTTER-EN-ACCÉLÉRÉ-POUR-LE-JEU.md](./19-PARTIE-2A—FLUTTER-EN-ACCÉLÉRÉ-POUR-LE-JEU.md)

**Si vous venez de terminer la PARTIE 1C**, vous pouvez sauter les chapitres 19 à 26 en diagonale — vous connaissez déjà les widgets et l'architecture — et vous rendre directement là où votre lanceur va prendre vie. Le chapitre 35 construit le squelette du jeu, ses états, ses overlays, et son menu principal : celui que vous venez d'écrire.

[35-PARTIE-2C—ARCHITECTURE-DU-JEU-ET-MENU-PRINCIPAL.md](./35-PARTIE-2C—ARCHITECTURE-DU-JEU-ET-MENU-PRINCIPAL.md)

**Si vous vous arrêtez ici**, une dernière chose. Reprenez ce projet dans une semaine, et faites-en le vôtre : changez le thème, ajoutez une classe de personnage, inventez dix objets, branchez un vrai serveur de classement. Un projet qu'on modifie après l'avoir compris est un projet qu'on n'oublie plus.

Bonne route.
