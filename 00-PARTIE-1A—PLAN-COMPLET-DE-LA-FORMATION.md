# FORMATION FLUTTER + JEU 2D

## Plan complet et sommaire de la PARTIE 1A — DART

Cette formation vous emmène de « je n'ai jamais programmé » jusqu'à « je publie un jeu 2D
écrit en Flutter ». Elle se lit dans l'ordre, un fichier après l'autre.

La **PARTIE 1A** que vous avez entre les mains couvre le langage **Dart** en 18 chapitres.
Aucune connaissance préalable n'est requise. Tous les exemples des chapitres 01 à 15
s'exécutent dans [DartPad](https://dartpad.dev) sans rien installer.

---

## 1 — Comment utiliser cette formation

1. Ouvrez le chapitre 01 et lisez-le en entier, dans l'ordre des sections numérotées.
2. Recopiez chaque bloc de code **à la main** dans DartPad. Ne faites pas de copier-coller :
   la mémoire des doigts fait partie de l'apprentissage.
3. Modifiez volontairement le code pour provoquer une erreur, puis lisez le message d'erreur.
4. Faites les exercices de fin de chapitre **avant** de regarder les corrections.
5. Ne passez au chapitre suivant que lorsque vous savez refaire les exercices sans aide.

Chaque chapitre suit toujours la même structure :

```text
En-tête (niveau, durée, pré-requis)
  NN.0  Objectifs du chapitre
  NN.1  Notion 1  →  explication, code, résultat, remarque
  NN.2  Notion 2
  ...
        Erreurs fréquentes      (tableau)
        Résumé du chapitre      (tableau)
        Exercices               (progressifs, du plus simple au plus difficile)
        Corrections commentées
        Et maintenant ?         (transition vers le chapitre suivant)
```

---

## 2 — Sommaire de la PARTIE 1A — DART

| # | Chapitre | Ce que vous apprenez | Sections | Lignes |
| --- | --- | --- | ---: | ---: |
| 01 | [Introduction à Dart](./01-PARTIE-1A—APPRENDRE-DART-DE-ZÉRO.md) | `main()`, `print()`, commentaires, DartPad, premières règles de syntaxe | 30 | 1 289 |
| 02 | [Variables, constantes et types](./02-PARTIE-1A—VARIABLES-CONSTANTES-ET-TYPES.md) | `String`, `int`, `double`, `bool`, `var`, `final`, `const`, `late`, interpolation | 75 | 2 755 |
| 03 | [Opérateurs et expressions](./03-PARTIE-1A—OPÉRATEURS-ET-EXPRESSIONS.md) | arithmétique, `~/`, `%`, `+=`, `++`, comparaisons, `&&`, `\|\|`, `!`, ternaire | 72 | 2 838 |
| 04 | [Conditions](./04-PARTIE-1A—CONDITIONS-IF-ELSE-SWITCH.md) | `if`, `else if`, `else`, `switch`, switch expression moderne | 66 | 2 747 |
| 05 | [Boucles](./05-PARTIE-1A—BOUCLES-FOR-WHILE-DO-WHILE.md) | `for`, `while`, `do while`, `break`, `continue`, boucles imbriquées, grilles 2D | 73 | 2 740 |
| 06 | [Collections](./06-PARTIE-1A—COLLECTIONS-LIST-SET-MAP.md) | `List`, `Set`, `Map`, collections imbriquées, inventaire de jeu | 106 | 2 701 |
| 07 | [Les fonctions](./07-PARTIE-1A—LES-FONCTIONS.md) | paramètres positionnels, optionnels, nommés, `required`, `=>`, callbacks, portée | 32 | 2 205 |
| 08 | [Introduction à la POO](./08-PARTIE-1A—INTRODUCTION-À-LA-POO.md) | classe, objet, instance, propriété, méthode, `this` | 62 | 3 289 |
| 09 | [Constructeurs et modélisation](./09-PARTIE-1A—CONSTRUCTEURS-ET-MODÉLISATION.md) | constructeurs nommés, initializer list, `const`, `factory`, getters/setters, `toString()` | 25 | 3 121 |
| 10 | [Encapsulation, héritage, polymorphisme](./10-PARTIE-1A—ENCAPSULATION-HÉRITAGE-POLYMORPHISME.md) | `_privé`, `extends`, `super`, `@override`, `is`, `as` | 26 | 3 049 |
| 11 | [POO avancée](./11-PARTIE-1A—POO-AVANCÉE-ABSTRACT-MIXINS-ENUMS.md) | `abstract`, `implements`, `mixin`/`with`, `enum`, enhanced enums, `extension` | 27 | 2 849 |
| 12 | [Le null safety](./12-PARTIE-1A—LE-NULL-SAFETY.md) | `?`, `?.`, `??`, `??=`, `!`, `late`, promotion de type | 26 | 3 081 |
| 13 | [Exceptions et gestion des erreurs](./13-PARTIE-1A—EXCEPTIONS-ET-GESTION-DES-ERREURS.md) | `try`, `catch`, `on`, `finally`, `throw`, `rethrow`, `tryParse()` | 28 | 3 415 |
| 14 | [Programmation fonctionnelle](./14-PARTIE-1A—PROGRAMMATION-FONCTIONNELLE-SUR-LES-COLLECTIONS.md) | `map()`, `where()`, `reduce()`, `fold()`, `sort()`, spread, collection `if`/`for` | 35 | 4 389 |
| 15 | [Programmation asynchrone](./15-PARTIE-1A—PROGRAMMATION-ASYNCHRONE-FUTURE-ASYNC-AWAIT.md) | event loop, `Future`, `async`, `await`, `Future.wait()`, `Stream` | 36 | 2 995 |
| 16 | [Organisation d'un projet Dart](./16-PARTIE-1A—ORGANISATION-DUN-PROJET-DART.md) | SDK, `dart create`, `pubspec.yaml`, packages, `import`/`export`, tests | 41 | 4 188 |
| 17 | [JSON et modélisation de données](./17-PARTIE-1A—JSON-ET-MODÉLISATION-DE-DONNÉES.md) | `dart:convert`, `jsonDecode`, `fromJson()`, `toJson()`, objets imbriqués | 39 | 3 684 |
| 18 | [Mini-projet final : Donjon de Dart](./18-PARTIE-1A—MINI-PROJET-FINAL-DART.md) | un jeu de rôle console complet qui mobilise les 17 chapitres | 34 | 5 004 |

**Total PARTIE 1A : 18 chapitres, 833 sections, 56 339 lignes.**

---

## 3 — Progression pédagogique

```text
  01 ─ 02 ─ 03 ─ 04 ─ 05 ─ 06      LES BASES DU LANGAGE
   │                        │      (tout se fait dans DartPad)
   │                        ▼
   │                       07      STRUCTURER : LES FONCTIONS
   │                        │
   │                        ▼
   │              08 ─ 09 ─ 10 ─ 11   PROGRAMMATION ORIENTÉE OBJET
   │                             │
   │                             ▼
   │                   12 ─ 13        CODE ROBUSTE
   │                        │         (null safety, exceptions)
   │                        ▼
   │                   14 ─ 15        CODE MODERNE
   │                        │         (fonctionnel, asynchrone)
   │                        ▼
   │                   16 ─ 17        VRAI PROJET
   │                        │         (SDK, packages, JSON)
   │                        ▼
   └──────────────────────► 18        MINI-PROJET FINAL
                                      « Donjon de Dart »
```

---

## 4 — Ce qui vient ensuite

| Partie | Contenu | État |
| --- | --- | --- |
| **1A — Dart** | Le langage, de zéro au mini-projet console | **Terminée** |
| **1B — Flutter** | Widgets, layouts, navigation, formulaires, thèmes, état, API REST, stockage local | À venir |
| **1C — Mini-projets Flutter** | 8 projets : carte de profil, calculatrice, convertisseur, todo list, quiz, catalogue, météo/API, projet préparatoire au jeu | À venir |
| **2A — Introduction aux jeux** | Game loop, FPS, delta time, coordonnées, sprites, collisions, caméra, architecture | À venir |
| **2B — Flame** | `FlameGame`, `GameWidget`, components, sprites animés, entrées clavier/tactile, collisions, audio, Tiled | À venir |
| **2C — Jeu complet** | Menu, joueur, ennemis, dégâts, vies, collectibles, score, niveaux, boss, sons, sauvegarde | À venir |
| **2D — Projet final** | Cahier des charges, Game Design Document, architecture, assets, tests, build Android et Web | À venir |

---

## 5 — Outils nécessaires

| Chapitres | Outil | Installation |
| --- | --- | --- |
| 01 → 15 | [DartPad](https://dartpad.dev) | Aucune, tout se passe dans le navigateur |
| 16 → 18 | Dart SDK + VS Code + extension Dart | Détaillée pas à pas au chapitre 16 |
| Partie 1B et suivantes | Flutter SDK | Détaillée au premier chapitre de la partie 1B |

---

## 6 — Conventions utilisées dans toute la formation

| Élément | Signification |
| --- | --- |
| Bloc ```` ```dart ```` | Du code Dart, à recopier dans DartPad ou dans votre projet |
| Bloc ```` ```text ```` | Ce que le programme affiche, un schéma, ou une arborescence de fichiers |
| Bloc ```` ```bash ```` | Une commande à taper dans le terminal |
| Bloc ```` ```yaml ```` | Le contenu d'un fichier `pubspec.yaml` ou `analysis_options.yaml` |
| `NN.M` | Numéro de section : chapitre `NN`, section `M` |
| **Explication :** | Le commentaire qui suit chaque correction d'exercice |

Le fil rouge de toute la formation est le **jeu vidéo** : joueur, ennemi, score, vies,
énergie, inventaire, niveau, arme, potion, boss. Ce n'est pas un hasard : chaque notion
apprise en Dart sera réutilisée telle quelle dans le jeu 2D de la partie 2.

---

Bonne formation. Commencez ici : [01-PARTIE-1A—APPRENDRE-DART-DE-ZÉRO.md](./01-PARTIE-1A—APPRENDRE-DART-DE-ZÉRO.md)
