# PARTIE 1B — FLUTTER
# CHAPITRE 43 — INSTALLER FLUTTER ET STRUCTURE D'UN PROJET

> **Niveau :** débutant
> **Durée estimée :** 6 h
> **Pré-requis :** la PARTIE 1A complète (chapitres 01 à 18), et tout particulièrement le chapitre 16 (organisation d'un projet Dart, `pubspec.yaml`, `dart pub`)
> **Ce que vous saurez faire à la fin :** installer le SDK Flutter sur votre système, diagnostiquer votre installation avec `flutter doctor`, créer un projet, en nommer chaque dossier, le lancer sur un émulateur, un téléphone, le navigateur ou le bureau, et modifier son interface sans le relancer grâce au hot reload.

> **Versions et date de vérification.** Les procédures de ce chapitre ont été vérifiées le **15 août 2026** sur la documentation officielle `docs.flutter.dev` (pages « Install », « Add Flutter to PATH », « Set up Android development », « Hot reload » et « flutter : The Flutter command-line tool »). À cette date, la dernière version **stable** publiée est **Flutter 3.47**. La formation est écrite pour **Flutter 3.44 ou plus récent** (Flutter 3.44 embarque **Dart 3.10**). Les numéros de version évoluent vite : ne vous inquiétez pas si `flutter --version` vous affiche un numéro différent, seul compte le fait qu'il soit **supérieur ou égal à 3.44**.

---

## 43.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer la différence entre « écrire du Dart » et « écrire une application Flutter » ;
- dire ce qu'est Flutter, et ce qu'il n'est pas ;
- expliquer ce que « multiplateforme » veut dire concrètement, et ce que cela coûte ;
- comparer honnêtement Flutter, React Native et le développement natif ;
- expliquer pourquoi Flutter dessine lui-même chaque pixel de l'écran ;
- installer le SDK Flutter sur Windows, macOS ou Linux ;
- expliquer ce qu'est la variable `PATH` et l'ajouter correctement ;
- lancer `flutter doctor` et lire chaque ligne de son rapport ;
- distinguer les canaux `stable` et `beta`, et mettre Flutter à jour avec `flutter upgrade` ;
- installer VS Code et l'extension Flutter ;
- installer Android Studio et les composants du SDK Android ;
- créer et démarrer un émulateur Android ;
- brancher un vrai téléphone en mode développeur ;
- activer les cibles Web et bureau ;
- lister vos appareils avec `flutter devices` ;
- créer un projet avec `flutter create mon_appli` ;
- utiliser les options `--org`, `--platforms`, `--project-name`, `--empty`, `--template` ;
- nommer chaque dossier d'un projet Flutter et dire à quoi il sert ;
- expliquer le rôle du dossier `lib/` et de `lib/main.dart` ;
- expliquer à quoi servent les dossiers `android/`, `ios/`, `web/` ;
- lire un `pubspec.yaml` d'application Flutter champ par champ ;
- installer des dépendances avec `flutter pub get` et `flutter pub add` ;
- lancer l'application avec `flutter run` ;
- distinguer le hot reload du hot restart, et savoir lequel utiliser ;
- citer les cas où le hot reload ne fonctionne pas ;
- analyser et formater votre code avec `flutter analyze` et `dart format` ;
- configurer `analysis_options.yaml` ;
- lire et expliquer le `main.dart` généré, ligne par ligne ;
- le remplacer par votre propre application « Bonjour » ;
- diagnostiquer et corriger les erreurs d'installation les plus fréquentes.

---

## 43.1 — De Dart à Flutter : ce qui change

Vous avez passé dix-huit chapitres à écrire du Dart. Vous savez déclarer une variable, écrire une boucle, modéliser un `Joueur`, gérer une exception, attendre un `Future`. Tout ce travail reste valable : **Flutter, c'est du Dart**. Vous ne changez pas de langage.

Ce qui change, c'est la **nature du programme** que vous écrivez.

Jusqu'ici, vos programmes ressemblaient à ceci :

```dart
void main() {
  final joueur = 'Lyra';
  var vies = 3;
  print('Bienvenue, $joueur.');
  while (vies > 0) {
    print('Il vous reste $vies vie(s).');
    vies--;
  }
  print('Partie terminée.');
}
```

**Résultat :**

```text
Bienvenue, Lyra.
Il vous reste 3 vie(s).
Il vous reste 2 vie(s).
Il vous reste 1 vie(s).
Partie terminée.
```

Ce programme a trois caractéristiques :

1. Il **commence**, il **fait des choses**, il **se termine**.
2. Il **décide de tout** : c'est lui qui choisit quand afficher, quand demander une saisie.
3. Sa sortie est du **texte**, dans un terminal.

Une application Flutter est différente sur les trois points.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  PROGRAMME CONSOLE (parties 1A)                                  │
  │                                                                  │
  │      main()                                                      │
  │        │                                                         │
  │        ├─ instruction 1                                          │
  │        ├─ instruction 2                                          │
  │        ├─ boucle                                                 │
  │        └─ fin  ──────────────►  le processus s'arrête            │
  │                                                                  │
  │  Le programme mène la danse. L'utilisateur suit.                 │
  └──────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────┐
  │  APPLICATION FLUTTER (partie 1B)                                 │
  │                                                                  │
  │      main()                                                      │
  │        │                                                         │
  │        └─ runApp(...)  ──────►  démarre une boucle infinie       │
  │                                                                  │
  │             ┌───────────────────────────────┐                    │
  │             │  attendre un évènement        │◄──────┐            │
  │             │  (appui, glissement, timer)   │       │            │
  │             ├───────────────────────────────┤       │            │
  │             │  mettre à jour l'état         │       │            │
  │             ├───────────────────────────────┤       │            │
  │             │  redessiner l'écran           │───────┘            │
  │             └───────────────────────────────┘                    │
  │                                                                  │
  │  L'utilisateur mène la danse. Le programme réagit.               │
  └──────────────────────────────────────────────────────────────────┘
```

On appelle cela la **programmation événementielle**. Votre `main()` ne fait presque rien : il démarre l'application, puis rend la main à Flutter. Ensuite, votre code n'est plus exécuté « de haut en bas » : il est **appelé** par Flutter, quand Flutter en a besoin.

Voici le tableau des changements, point par point.

| Aspect | Programme console (1A) | Application Flutter (1B) |
| --- | --- | --- |
| Point d'entrée | `void main()` | `void main()` **plus** `runApp(...)` |
| Durée de vie | quelques millisecondes ou secondes | tant que l'utilisateur ne ferme pas |
| Qui décide | votre code | l'utilisateur, via des évènements |
| Sortie | `print` dans un terminal | des pixels sur un écran |
| Entrée | `stdin.readLineSync()` | appuis, glissements, clavier virtuel |
| Unité de base | la fonction et la classe | le **widget** |
| Erreur typique | exception non attrapée | écran rouge « exception caught by widgets library » |
| Où l'on teste | le terminal | un émulateur, un téléphone, un navigateur |

Il y a aussi un changement de vocabulaire. Dans la partie 1A, votre brique de base était la **classe**. Dans la partie 1B, votre brique de base sera le **widget** — qui est, vous le verrez au chapitre 44, tout simplement une classe Dart d'un genre particulier. Rien de magique.

> **À retenir :** vous ne réapprenez pas un langage. Vous apprenez à écrire un programme qui **ne se termine jamais** et qui **réagit** au lieu d'agir.

Une dernière chose, rassurante. Tout ce que vous avez appris sert immédiatement :

| Chapitre 1A | Usage immédiat en Flutter |
| --- | --- |
| 02 — Variables et types | les propriétés d'un widget |
| 04 — Conditions | afficher un widget ou un autre |
| 05 — Boucles | construire une liste d'éléments |
| 06 — Collections | `List<Widget>`, `Map` de données |
| 07 — Fonctions | les fonctions de rappel (`onPressed`) |
| 08 à 11 — POO | tout widget est une classe qui **hérite** |
| 12 — Null safety | `String?`, `??`, `!` sont partout |
| 13 — Exceptions | gérer une erreur réseau |
| 14 — `map`, `where` | transformer des données en widgets |
| 15 — Asynchrone | charger des données depuis Internet |
| 16 — Projet et `pubspec` | ce chapitre-ci, en version Flutter |
| 17 — JSON | le chapitre 53 |

Aucun de ces chapitres n'a été écrit pour rien.

---

## 43.2 — Qu'est-ce que Flutter ?

Définition courte, à retenir telle quelle :

> **Flutter est une boîte à outils (« SDK ») créée par Google, qui permet de construire, à partir d'un seul code source écrit en Dart, des applications pour Android, iOS, le Web, Windows, macOS et Linux.**

Décortiquons chaque morceau de cette phrase.

### 43.2.1 — « une boîte à outils »

Flutter n'est pas un langage. Flutter n'est pas un éditeur de texte. Flutter est un ensemble de choses livrées ensemble :

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  LE SDK FLUTTER CONTIENT                                       │
  ├────────────────────────────────────────────────────────────────┤
  │                                                                │
  │  1. Le SDK Dart complet                                        │
  │     La VM, le compilateur, l'analyseur, le formateur, pub.     │
  │     (Vous n'avez donc PAS besoin d'installer Dart à part.)     │
  │                                                                │
  │  2. Le framework Flutter                                       │
  │     Des milliers de classes Dart prêtes à l'emploi :           │
  │     Text, Column, Button, ListView, Navigator, Theme...        │
  │                                                                │
  │  3. Le moteur de rendu (« engine »)                            │
  │     Écrit en C++. Il dessine les pixels et parle au système.   │
  │                                                                │
  │  4. L'outil en ligne de commande « flutter »                   │
  │     flutter create, flutter run, flutter build, flutter test.  │
  │                                                                │
  │  5. Les « embedders »                                          │
  │     Le morceau de code propre à chaque système (Android, iOS,  │
  │     Web, Windows...) qui héberge le moteur.                    │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘
```

Retenez surtout le point 1 : **installer Flutter installe Dart**. Si vous aviez installé Dart seul au chapitre 16, vous pouvez le désinstaller — ou le garder, cela ne gêne pas, à condition de savoir lequel des deux votre `PATH` trouve en premier (nous y reviendrons en 43.9).

### 43.2.2 — « créée par Google »

Flutter est développé par Google, en source ouverte, sous licence BSD. Le code est public sur GitHub. Cela a deux conséquences pratiques :

- vous pouvez **lire le code source** de n'importe quel widget ; c'est même l'une des meilleures façons d'apprendre, et votre éditeur vous y emmène d'un clic ;
- le projet suit un rythme de publication rapide : une version stable environ tous les trois mois.

### 43.2.3 — « un seul code source »

C'est la promesse centrale. Vous écrivez :

```dart
Text('Score : 1200')
```

et cette ligne produit un texte à l'écran sur les six plateformes, sans une seule ligne de code spécifique à Android ou à iOS.

Nous verrons en 43.3 que cette promesse est réelle, mais qu'elle a des limites qu'il faut connaître dès maintenant.

### 43.2.4 — Ce que Flutter n'est pas

Il est aussi utile de savoir ce que Flutter **ne fait pas**, pour ne pas être déçu.

| Ce que Flutter n'est pas | Précision |
| --- | --- |
| Un langage | Le langage est Dart. |
| Un moteur de jeu | Il sait afficher une interface. Pour un jeu, on ajoute Flame (partie 2). |
| Un serveur | Flutter fait le **client**. Le serveur, c'est autre chose. |
| Une base de données | Il faut un package (chapitre 54). |
| Un remplaçant du natif | Certaines applications ont de bonnes raisons de rester natives (43.4). |
| Un outil de conception graphique | Vous codez l'interface, vous ne la dessinez pas à la souris. |

> **Remarque :** on écrit souvent « le framework Flutter » et « le SDK Flutter » comme des synonymes. En toute rigueur, le SDK est la boîte complète, et le framework est la partie Dart de cette boîte (les widgets). Dans la pratique courante, personne ne vous reprendra.

---

## 43.3 — Multiplateforme : ce que cela veut dire vraiment

« Un seul code, six plateformes » est un slogan. Regardons ce qu'il recouvre en pratique, sans exagérer dans un sens ni dans l'autre.

### 43.3.1 — Le partage réel

Sur une application classique — un catalogue, une liste de tâches, une application météo, un client d'API — la part de code commune est très élevée :

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  RÉPARTITION TYPIQUE DU CODE D'UNE APPLICATION FLUTTER         │
  ├────────────────────────────────────────────────────────────────┤
  │                                                                │
  │  Interface (widgets, écrans, thème)          ~ 55 %  commun    │
  │  Logique métier (modèles, calculs, état)     ~ 30 %  commun    │
  │  Accès réseau, JSON, stockage                ~ 10 %  commun    │
  │  Code spécifique à une plateforme            ~  5 %  spécifique│
  │                                                                │
  │  ────────────────────────────────────────────────────────────  │
  │  Environ 95 % du code est écrit UNE fois.                      │
  └────────────────────────────────────────────────────────────────┘
```

Ces chiffres varient, mais l'ordre de grandeur est juste. Sur une application standard, vous n'écrirez sans doute **aucune** ligne de code spécifique à une plateforme pendant des semaines.

### 43.3.2 — Ce qui reste spécifique malgré tout

Les 5 % restants se cachent à des endroits précis. Les voici, listés honnêtement.

| Domaine | Pourquoi cela reste spécifique |
| --- | --- |
| Icône et nom de l'application | Chaque système a son format et son emplacement. |
| Permissions | Android utilise `AndroidManifest.xml`, iOS utilise `Info.plist`. |
| Signature et publication | Certificats iOS, `keystore` Android : rien de commun. |
| Notifications | La configuration diffère profondément. |
| Achats intégrés | Deux boutiques, deux systèmes de facturation. |
| Fonctions matérielles pointues | NFC, capteurs exotiques, API système récente. |
| Conventions d'interface | Le bouton « retour » physique d'Android n'existe pas sur iOS. |
| Web | Pas de système de fichiers local classique, URL visible, onglets. |

Aucun de ces points n'est bloquant. Tous demandent une demi-heure de configuration, une fois, par plateforme.

### 43.3.3 — « Écrire une fois » n'est pas « tester une fois »

C'est le piège le plus courant chez le débutant. Le code est commun, mais :

- le rendu n'est pas identique au pixel près (polices système, densité d'écran, encoche) ;
- les performances diffèrent (un vieux téléphone Android n'est pas un iPhone récent) ;
- les comportements du clavier, du défilement et des gestes ont des nuances ;
- le Web ajoute ses propres contraintes (taille de l'application téléchargée, retour navigateur).

> **Règle d'or :** vous écrivez une fois, mais vous **testez sur chaque plateforme que vous prétendez supporter**. Une application « qui marche sur Android » n'est pas une application « qui marche ».

### 43.3.4 — Les six cibles, et ce qu'elles demandent

| Cible | Machine de développement nécessaire | Outil supplémentaire |
| --- | --- | --- |
| Android | Windows, macOS ou Linux | Android Studio + SDK Android |
| iOS | **macOS uniquement** | Xcode |
| Web | Windows, macOS ou Linux | un navigateur (Chrome recommandé) |
| Windows | **Windows uniquement** | Visual Studio (charge « Desktop C++ ») |
| macOS | **macOS uniquement** | Xcode |
| Linux | **Linux uniquement** | GTK, `clang`, `ninja`, `pkg-config` |

Lisez bien ce tableau : **on ne peut pas compiler une application iOS depuis Windows.** C'est une contrainte d'Apple, pas de Flutter. Si vous êtes sous Windows et que vous visez iPhone, il vous faudra à terme un Mac, ou un service de compilation à distance.

Pour cette formation, **Android et le Web suffisent largement**. Vous pouvez suivre les parties 1B et 1C entièrement dans le navigateur si vous n'avez ni téléphone ni émulateur.

---

## 43.4 — Flutter vs React Native vs natif : tableau honnête

Vous allez investir des mois dans une technologie. Vous méritez une comparaison sans marketing.

Les trois approches :

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  NATIF                                                           │
  │                                                                  │
  │    Android          iOS                                          │
  │    Kotlin           Swift                                        │
  │    Jetpack Compose  SwiftUI                                      │
  │       │                │                                         │
  │       └── deux équipes, deux codes, deux calendriers ──┘         │
  └──────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────┐
  │  REACT NATIVE                                                    │
  │                                                                  │
  │    Un code JavaScript / TypeScript                               │
  │       │                                                          │
  │       └── traduit en composants NATIFS du système                │
  │           (un <Button> devient un vrai bouton Android/iOS)       │
  └──────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────┐
  │  FLUTTER                                                         │
  │                                                                  │
  │    Un code Dart                                                  │
  │       │                                                          │
  │       └── DESSINÉ par le moteur Flutter sur une toile            │
  │           (un bouton Flutter est un dessin, pas un widget natif) │
  └──────────────────────────────────────────────────────────────────┘
```

Le tableau comparatif :

| Critère | Natif (Kotlin / Swift) | React Native | Flutter |
| --- | --- | --- | --- |
| Langage | Kotlin, Swift | JavaScript / TypeScript | Dart |
| Nombre de codes à maintenir | 2 | 1 | 1 |
| Rendu | composants du système | composants du système | dessin par le moteur |
| Aspect « natif » parfait | oui, par construction | très proche | très proche, mais **imité** |
| Performance brute | référence | bonne | très bonne, proche du natif |
| Accès immédiat aux nouveautés du système | immédiat | différé | différé |
| Taille minimale de l'application | la plus petite | moyenne | plus grande (moteur embarqué) |
| Cohérence visuelle entre plateformes | faible (deux designs) | moyenne | **très forte** |
| Vitesse d'itération | moyenne | rapide | **rapide** (hot reload) |
| Écosystème de packages | énorme, par plateforme | énorme (npm) | large (pub.dev), plus jeune |
| Bureau et Web depuis le même code | non | partiel | **oui** |
| Nombre de développeurs sur le marché | très élevé | très élevé | élevé, en croissance |
| Courbe d'apprentissage | 2 langages, 2 SDK | JS + spécificités RN | 1 langage, 1 SDK |
| Adapté à un débutant seul | difficile | moyen | **oui** |

### 43.4.1 — Quand choisir le natif

Choisissez le natif si :

- vous visez **une seule** plateforme et vous voulez le meilleur résultat possible ;
- votre application repose massivement sur des fonctions système récentes (widgets d'écran d'accueil, extensions, montre connectée, réalité augmentée) ;
- la taille de l'application est critique (marchés à faible connectivité) ;
- vous avez déjà une équipe Kotlin ou Swift.

### 43.4.2 — Quand choisir React Native

Choisissez React Native si :

- votre équipe est déjà experte en JavaScript ou React ;
- vous partagez du code avec un site Web React existant ;
- vous tenez absolument à ce que les composants soient ceux du système.

### 43.4.3 — Quand choisir Flutter

Choisissez Flutter si :

- vous voulez une seule base de code pour mobile **et** Web **et** bureau ;
- vous voulez une identité visuelle forte et **identique partout** ;
- vous êtes seul ou en petite équipe ;
- vous aimez un langage typé et une compilation stricte ;
- vous voulez itérer vite sur l'interface.

C'est notre cas dans cette formation. Et c'est aussi le meilleur chemin vers la partie 2, puisque le moteur de jeu Flame est construit sur Flutter.

> **Honnêteté :** aucune de ces trois approches n'est « la meilleure ». Ce sont trois compromis différents. Le pire choix serait de croire qu'il n'y a pas de compromis.

---

## 43.5 — Le moteur de rendu, et pourquoi Flutter dessine tout lui-même

C'est le point le plus important pour comprendre Flutter en profondeur. Prenez le temps de le lire deux fois.

### 43.5.1 — Le problème que Flutter résout

Une application Android affiche un bouton en demandant au système : « Android, dessine-moi un bouton. » Android le dessine à sa façon. Sur iOS, la même demande donne un bouton d'aspect différent, de taille différente, avec une animation différente.

C'est très bien pour l'utilisateur, qui reconnaît son système. C'est très pénible pour le développeur multiplateforme, qui doit composer avec deux comportements qu'il ne contrôle pas, et qui changent à chaque mise à jour du système.

### 43.5.2 — La décision de Flutter

Flutter fait un choix radical :

> **Flutter ne demande rien au système, sauf une surface vide.**

Le système lui fournit un rectangle de pixels. Flutter dessine tout dedans lui-même : le fond, le texte, les bordures, les ombres, le curseur, l'animation d'appui. Absolument tout.

```text
  ┌──────────────────────────────────────────────────────────────────┐
  │  APPLICATION NATIVE / REACT NATIVE                               │
  │                                                                  │
  │   Votre code  ──►  « Système, donne-moi un Button »              │
  │                          │                                       │
  │                          ▼                                       │
  │                    Le SYSTÈME dessine le bouton                  │
  │                    (aspect imposé par Android ou iOS)            │
  └──────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────┐
  │  APPLICATION FLUTTER                                             │
  │                                                                  │
  │   Votre code  ──►  « Système, donne-moi une surface »            │
  │                          │                                       │
  │                          ▼                                       │
  │                    ┌───────────────────────┐                     │
  │                    │  surface vide         │                     │
  │                    │                       │                     │
  │                    │   ← MOTEUR FLUTTER    │                     │
  │                    │     dessine TOUT      │                     │
  │                    │     via le GPU        │                     │
  │                    └───────────────────────┘                     │
  └──────────────────────────────────────────────────────────────────┘
```

### 43.5.3 — Les couches de Flutter

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  VOTRE APPLICATION                             (Dart)          │
  │  main.dart, vos widgets, vos modèles                           │
  ├────────────────────────────────────────────────────────────────┤
  │  FRAMEWORK FLUTTER                             (Dart)          │
  │                                                                │
  │    material / cupertino   Text, Card, AppBar, Switch...        │
  │    widgets                StatelessWidget, StatefulWidget      │
  │    rendering              calcul des tailles et positions      │
  │    painting               traits, remplissages, dégradés       │
  │    animation, gestures    tweens, appuis, glissements          │
  │    foundation             briques de base                      │
  ├────────────────────────────────────────────────────────────────┤
  │  MOTEUR FLUTTER                                (C++)           │
  │                                                                │
  │    Impeller / Skia   la bibliothèque graphique                 │
  │    Dart runtime      exécute votre code compilé                │
  │    text layout       découpe et positionne le texte            │
  ├────────────────────────────────────────────────────────────────┤
  │  EMBEDDER                        (Java/Kotlin, ObjC/Swift, JS) │
  │  Crée la fenêtre, transmet les évènements, gère le cycle de vie│
  ├────────────────────────────────────────────────────────────────┤
  │  SYSTÈME D'EXPLOITATION                                        │
  │  Android, iOS, Windows, macOS, Linux, navigateur               │
  └────────────────────────────────────────────────────────────────┘
```

Vous travaillerez presque exclusivement dans les deux couches du haut. Les couches basses sont là pour votre culture — et pour le jour où une question « pourquoi ça rame ? » se posera.

### 43.5.4 — Les conséquences, bonnes et mauvaises

| Conséquence | Signe |
| --- | --- |
| Le rendu est **identique** sur toutes les plateformes | avantage |
| Vous contrôlez **chaque pixel**, animations comprises | avantage |
| Les mises à jour du système ne cassent pas votre interface | avantage |
| Le rendu est **fluide** (viser 60 ou 120 images par seconde) | avantage |
| Vous pouvez créer un design totalement original | avantage |
| L'application **embarque le moteur**, elle est donc plus lourde | inconvénient |
| Les widgets natifs les plus récents ne sont pas là le jour J | inconvénient |
| L'accessibilité et la sélection de texte sont **réimplémentées** | inconvénient (bien géré, mais réimplémenté) |
| Sur le Web, le premier chargement est plus long | inconvénient |

### 43.5.5 — Impeller et Skia

Historiquement, Flutter dessinait avec **Skia**, la bibliothèque graphique qui sert aussi à Chrome. Depuis quelques versions, Flutter utilise **Impeller**, un moteur de rendu conçu spécifiquement pour Flutter, qui précompile ses nuanceurs (« shaders ») afin d'éviter les micro-saccades du premier affichage d'une animation.

Vous n'avez rien à faire : c'est le comportement par défaut sur les plateformes où Impeller est activé. Retenez simplement le mot, vous le croiserez dans les messages de la console.

> **À retenir :** un bouton Flutter n'est pas un bouton Android. C'est un **dessin de bouton**, produit par Flutter, pixel par pixel. Toute la puissance et toutes les limites de Flutter découlent de cette seule décision.

---

## 43.6 — Installer le SDK sur Windows

Nous entrons dans la partie outillage. Elle est longue et un peu ingrate, mais elle se fait **une seule fois**. Suivez-la dans l'ordre, sans sauter d'étape.

### 43.6.1 — Ce qu'il vous faut avant de commencer

| Élément | Détail |
| --- | --- |
| Windows | Windows 10 ou 11, 64 bits |
| Espace disque | prévoyez 15 Go libres (Flutter + Android Studio + émulateur) |
| Mémoire | 8 Go minimum, 16 Go confortable |
| Git pour Windows | obligatoire, Flutter s'en sert en interne |
| PowerShell | version 5 ou plus, livrée avec Windows |

Installez d'abord **Git pour Windows** depuis `https://git-scm.com/downloads/win`. Acceptez les options par défaut. Vérifiez ensuite dans un nouveau terminal :

```bash
git --version
```

**Résultat :**

```text
git version 2.51.0.windows.1
```

Si cette commande échoue, ne continuez pas : Flutter ne fonctionnera pas.

### 43.6.2 — Deux chemins possibles

| Chemin | Pour qui | Principe |
| --- | --- | --- |
| A — VS Code | débutant, chemin recommandé | l'extension Flutter télécharge le SDK pour vous |
| B — Manuel | si vous voulez maîtriser l'emplacement | vous téléchargez l'archive et vous réglez le `PATH` |

Nous décrivons les deux. Le chemin B est celui que la documentation appelle « custom setup » ; c'est celui que je recommande pour comprendre ce qui se passe, mais le chemin A est parfaitement valable.

### 43.6.3 — Chemin A : par VS Code

1. Installez **Visual Studio Code** depuis `https://code.visualstudio.com`.
2. Ouvrez VS Code.
3. Ouvrez la palette de commandes : `Ctrl` + `Shift` + `P`.
4. Tapez `Flutter: New Project` et validez.
5. VS Code détecte qu'aucun SDK n'est installé et propose **Download SDK**.
6. Choisissez un dossier de destination **sans espace ni accent**, par exemple `C:\Users\VotreNom\develop`.
7. Attendez : le téléchargement fait environ 1 Go.
8. Quand VS Code propose **Add SDK to PATH**, acceptez.

Passez ensuite à la vérification (43.6.5).

### 43.6.4 — Chemin B : installation manuelle

**Étape 1.** Rendez-vous sur `https://docs.flutter.dev/install/archive` et téléchargez la dernière archive **stable** pour Windows. Le fichier s'appelle `flutter_windows_<version>-stable.zip`.

**Étape 2.** Créez un dossier de destination. Deux règles absolues :

- pas d'espace, pas d'accent, pas de caractère spécial dans le chemin ;
- pas de dossier nécessitant des droits administrateur (donc **pas** `C:\Program Files`).

Le bon choix est votre dossier utilisateur :

```text
  C:\Users\VotreNom\develop\
```

**Étape 3.** Décompressez l'archive dans ce dossier. En PowerShell :

```bash
Expand-Archive -Path $env:USERPROFILE\Downloads\flutter_windows_3.47.0-stable.zip -DestinationPath $env:USERPROFILE\develop\
```

Adaptez le numéro de version à celui du fichier que vous avez téléchargé.

Vous obtenez :

```text
  C:\Users\VotreNom\develop\flutter\
      bin\
      packages\
      examples\
      ...
```

Le dossier qui nous intéresse est `bin` : c'est lui qui contient le programme `flutter`.

**Étape 4.** Ajoutez `bin` au `PATH` :

1. Appuyez sur la touche `Windows`, tapez « variables d'environnement », ouvrez **Modifier les variables d'environnement pour votre compte**.
2. Dans **Variables utilisateur**, sélectionnez `Path`, puis **Modifier**.
3. **Nouveau**, et saisissez :

```text
%USERPROFILE%\develop\flutter\bin
```

4. Déplacez cette entrée **en haut** de la liste avec le bouton « Monter ».
5. Validez par **OK** trois fois.
6. **Fermez tous vos terminaux et VS Code**, puis rouvrez-en un neuf.

Le point 6 n'est pas une formalité. Un terminal déjà ouvert conserve l'ancien `PATH` et vous répondra obstinément que la commande est introuvable. C'est de très loin l'erreur numéro un.

### 43.6.5 — Vérification

Ouvrez un **nouveau** terminal :

```bash
flutter --version
```

**Résultat :**

```text
Flutter 3.47.0 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 4d5f1e2b3c (5 days ago) • 2026-08-10 09:14:22 -0700
Engine • revision 9a8b7c6d5e
Tools • Dart 3.13.0 • DevTools 2.52.0
```

Les numéros exacts et la révision seront différents chez vous. Ce qui compte :

- la commande répond ;
- le canal est `stable` ;
- la version est ≥ 3.44.

Vérifiez au passage que Dart est bien là :

```bash
dart --version
```

**Résultat :**

```text
Dart SDK version: 3.13.0 (stable) (Mon Aug 10 2026) on "windows_x64"
```

> **Piège Windows :** si vous avez installé Dart seul au chapitre 16, votre `PATH` contient deux `dart`. Celui qui gagne est celui dont le dossier apparaît **en premier** dans le `PATH`. C'est pourquoi on remonte l'entrée Flutter en haut de la liste. Voir 43.9.

### 43.6.6 — Antivirus et Windows Defender

Flutter écrit et exécute beaucoup de petits fichiers. Certains antivirus ralentissent énormément les compilations, jusqu'à multiplier par cinq le temps de démarrage.

Si `flutter run` met plusieurs minutes, ajoutez une **exclusion** dans votre antivirus pour :

```text
  C:\Users\VotreNom\develop\flutter
  le dossier de vos projets
```

---

## 43.7 — Installer le SDK sur macOS

### 43.7.1 — Prérequis

| Élément | Détail |
| --- | --- |
| macOS | version récente supportée par le dernier Xcode |
| Processeur | Apple Silicon recommandé (les versions Intel sont en fin de support) |
| Espace disque | 25 Go si vous installez Xcode |
| Outils en ligne de commande Xcode | obligatoires |

Installez d'abord les outils en ligne de commande :

```bash
xcode-select --install
```

Une fenêtre s'ouvre. Cliquez sur **Installer**, puis **Terminé**. Cette étape installe notamment `git`.

Vérifiez :

```bash
git --version
```

**Résultat :**

```text
git version 2.51.0
```

### 43.7.2 — Connaître son processeur

C'est important : il existe deux archives différentes.

Menu  > **À propos de ce Mac**. Regardez la ligne « Puce » ou « Processeur ».

| Ce que vous lisez | Archive à télécharger |
| --- | --- |
| Apple M1, M2, M3, M4... | version **Apple Silicon (arm64)** |
| Intel Core i5, i7, i9... | version **Intel (x64)** |

En ligne de commande :

```bash
uname -m
```

**Résultat sur Apple Silicon :**

```text
arm64
```

**Résultat sur Intel :**

```text
x86_64
```

### 43.7.3 — Télécharger et décompresser

Téléchargez l'archive `flutter_macos_<version>-stable.zip` (ou `flutter_macos_arm64_<version>-stable.zip`) depuis `https://docs.flutter.dev/install/archive`.

Créez un dossier de destination et décompressez :

```bash
mkdir -p ~/develop
unzip ~/Downloads/flutter_macos_3.47.0-stable.zip -d ~/develop/
```

Adaptez le nom du fichier à celui que vous avez téléchargé.

Vous obtenez `~/develop/flutter/`.

### 43.7.4 — Ajouter au PATH

Le shell par défaut de macOS est **zsh**. Son fichier de configuration de session est `~/.zprofile`.

```bash
nano ~/.zprofile
```

Ajoutez cette ligne à la fin du fichier :

```bash
export PATH="$HOME/develop/flutter/bin:$PATH"
```

Enregistrez avec `Ctrl` + `O`, `Entrée`, puis quittez avec `Ctrl` + `X`.

Rechargez la configuration :

```bash
source ~/.zprofile
```

Puis **fermez et rouvrez** toutes vos fenêtres de terminal.

> **Remarque :** si votre shell est bash (à vérifier avec `echo $SHELL`), le fichier est `~/.bash_profile` et non `~/.zprofile`.

### 43.7.5 — Vérification

```bash
flutter --version
```

**Résultat :**

```text
Flutter 3.47.0 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 4d5f1e2b3c (5 days ago) • 2026-08-10 09:14:22 -0700
Engine • revision 9a8b7c6d5e
Tools • Dart 3.13.0 • DevTools 2.52.0
```

### 43.7.6 — Rosetta sur Apple Silicon

Certains outils de la chaîne iOS sont encore compilés pour Intel. Sur un Mac Apple Silicon, installez Rosetta :

```bash
sudo softwareupdate --install-rosetta --agree-to-license
```

Ce n'est nécessaire que si vous visez iOS. Pour Android et le Web, vous pouvez sauter cette étape.

### 43.7.7 — Xcode, uniquement si vous visez iOS ou macOS

Xcode pèse plus de 10 Go et s'installe depuis le Mac App Store. Après installation :

```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

La première commande indique à votre système quel Xcode utiliser. La seconde accepte les licences et installe les composants additionnels.

Si vous ne visez pas iOS pour l'instant, ignorez cette section : `flutter doctor` affichera un avertissement sur Xcode, et ce n'est pas grave (voir 43.10).

---

## 43.8 — Installer le SDK sur Linux

### 43.8.1 — Les paquets nécessaires

Sur Debian et Ubuntu :

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install -y curl git unzip xz-utils zip libglu1-mesa
```

Détail de ces paquets :

| Paquet | Rôle |
| --- | --- |
| `curl` | téléchargements effectués par l'outil `flutter` |
| `git` | Flutter est un dépôt Git, il s'en sert pour se mettre à jour |
| `unzip`, `zip`, `xz-utils` | décompression des archives |
| `libglu1-mesa` | bibliothèque OpenGL utilisée par l'émulateur |

Sur Fedora, l'équivalent :

```bash
sudo dnf install curl git unzip xz zip mesa-libGLU
```

Sur Arch :

```bash
sudo pacman -S curl git unzip xz zip glu
```

### 43.8.2 — Télécharger et décompresser

Téléchargez `flutter_linux_<version>-stable.tar.xz` depuis `https://docs.flutter.dev/install/archive`, puis :

```bash
mkdir -p ~/develop
tar -xf ~/Downloads/flutter_linux_3.47.0-stable.tar.xz -C ~/develop/
```

Vous obtenez `~/develop/flutter/`.

### 43.8.3 — Ajouter au PATH

Selon votre shell :

**Bash :**

```bash
echo 'export PATH="$HOME/develop/flutter/bin:$PATH"' >> ~/.bashrc
```

**Zsh :**

```bash
echo 'export PATH="$HOME/develop/flutter/bin:$PATH"' >> ~/.zshenv
```

**Fish :**

```bash
fish_add_path -g -p ~/develop/flutter/bin
```

Fermez et rouvrez votre terminal, ou rechargez la configuration :

```bash
source ~/.bashrc
```

### 43.8.4 — Vérification

```bash
flutter --version
```

**Résultat :**

```text
Flutter 3.47.0 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 4d5f1e2b3c (5 days ago) • 2026-08-10 09:14:22 -0700
Engine • revision 9a8b7c6d5e
Tools • Dart 3.13.0 • DevTools 2.52.0
```

### 43.8.5 — Le cas snap

Sur Ubuntu, il existe un paquet snap `flutter`. Il fonctionne, mais il complique parfois la vie : chemins inhabituels, mises à jour automatiques, permissions restreintes vers les périphériques USB.

> **Conseil :** préférez l'archive officielle décrite ci-dessus. Si vous avez déjà installé le snap et que vous rencontrez des problèmes bizarres, désinstallez-le avec `sudo snap remove flutter` avant de repartir de l'archive.

### 43.8.6 — Si vous visez le bureau Linux

Pour compiler une application Flutter **pour Linux**, il faut en plus la chaîne C++ et GTK :

```bash
sudo apt-get install -y clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev
```

Ce n'est pas nécessaire pour Android ni pour le Web.

---

## 43.9 — La variable PATH

Cette section explique **pourquoi** les manipulations précédentes fonctionnent. C'est la source de la moitié des problèmes d'installation ; comprenez-la une fois et vous ne serez plus jamais bloqué.

### 43.9.1 — Le problème

Vous tapez `flutter` dans un terminal. Votre système doit trouver un fichier exécutable qui s'appelle `flutter`. Mais où ?

Il ne va pas parcourir tout le disque : ce serait lent et dangereux. Il consulte une liste de dossiers préenregistrée. Cette liste s'appelle le **`PATH`**.

```text
  Vous tapez :  flutter --version
                   │
                   ▼
  Le système lit la variable PATH, dossier par dossier, DANS L'ORDRE :

     1. C:\Users\Moi\develop\flutter\bin    ← trouvé ! on s'arrête ici
     2. C:\Windows\system32
     3. C:\Windows
     4. C:\Program Files\Git\cmd
     ...

  Si aucun dossier ne contient « flutter » :
     « flutter n'est pas reconnu... » / « command not found »
```

Deux enseignements immédiats :

1. **Si le dossier n'est pas dans le `PATH`, la commande n'existe pas** pour vous.
2. **L'ordre compte.** Le premier trouvé gagne.

### 43.9.2 — Afficher son PATH

**Windows (PowerShell) :**

```bash
$env:Path -split ';'
```

**macOS et Linux :**

```bash
echo $PATH | tr ':' '\n'
```

**Résultat (Linux) :**

```text
/home/moi/develop/flutter/bin
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
```

La ligne Flutter doit apparaître. Si elle n'y est pas, votre modification n'a pas été prise en compte : soit vous avez édité le mauvais fichier, soit vous n'avez pas rouvert le terminal.

### 43.9.3 — Savoir quel exécutable est réellement utilisé

C'est la commande qui règle 90 % des disputes.

**macOS et Linux :**

```bash
which flutter
which dart
```

**Résultat :**

```text
/home/moi/develop/flutter/bin/flutter
/home/moi/develop/flutter/bin/dart
```

**Windows (PowerShell) :**

```bash
Get-Command flutter
Get-Command dart
```

**Résultat :**

```text
CommandType  Name         Source
-----------  ----         ------
Application  flutter.bat  C:\Users\Moi\develop\flutter\bin\flutter.bat
Application  dart.exe     C:\Users\Moi\develop\flutter\bin\dart.exe
```

Le cas problématique classique, si vous aviez installé Dart seul au chapitre 16 :

```text
/usr/lib/dart/bin/dart          ← ce dart-là n'est PAS celui de Flutter
```

Ce n'est pas dramatique, mais cela peut produire des messages incohérents. La solution : mettre le dossier `flutter/bin` **avant** l'ancien dans le `PATH`.

### 43.9.4 — Les trois erreurs classiques

| Erreur | Symptôme | Correction |
| --- | --- | --- |
| Terminal non rouvert | la commande reste introuvable | fermer **tous** les terminaux et l'éditeur, rouvrir |
| Mauvais fichier de configuration | fonctionne dans une session, pas dans l'autre | `echo $SHELL` puis éditer le bon fichier |
| Chemin pointant sur `flutter` au lieu de `flutter/bin` | introuvable malgré tout | le `PATH` doit finir par `/bin` |

Retenez cette dernière ligne. On ajoute :

```text
  .../develop/flutter/bin          CORRECT
  .../develop/flutter              INCORRECT
```

### 43.9.5 — Rendre la modification permanente

Sur Windows, la fenêtre « Variables d'environnement » écrit dans le registre : c'est permanent.

Sur macOS et Linux, une commande `export PATH=...` tapée directement dans le terminal ne vaut **que pour cette fenêtre**. Pour qu'elle survive, elle doit être écrite dans un fichier lu à chaque ouverture de session :

| Shell | Fichier |
| --- | --- |
| zsh (macOS par défaut) | `~/.zprofile` ou `~/.zshrc` |
| bash (Linux par défaut) | `~/.bashrc` |
| fish | `~/.config/fish/config.fish`, ou `fish_add_path` |

Pour savoir quel shell vous utilisez :

```bash
echo $SHELL
```

**Résultat :**

```text
/bin/zsh
```

---

## 43.10 — `flutter doctor` et la lecture de son rapport

`flutter doctor` est l'outil de diagnostic de Flutter. Il inspecte votre machine et vous dit, poste par poste, ce qui est prêt et ce qui manque.

**Prenez l'habitude de le lancer chaque fois que quelque chose ne marche pas.**

### 43.10.1 — La commande

```bash
flutter doctor
```

**Résultat typique après une première installation :**

```text
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.47.0, on Ubuntu 24.04 x86_64, locale fr_FR.UTF-8)
[!] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from:
      https://developer.android.com/studio/index.html
[✓] Chrome - develop for the web
[!] Linux toolchain - develop for Linux desktop
    ✗ clang++ is required for Linux development.
[✓] Android Studio (not installed)
[✓] VS Code (version 1.104.0)
[✓] Connected device (1 available)
[✓] Network resources

! Doctor found issues in 2 categories.
```

### 43.10.2 — Lire les symboles

Il n'y a que trois symboles à connaître. C'est tout le secret.

| Symbole | Signification | Que faire |
| --- | --- | --- |
| `[✓]` | Tout va bien pour cette catégorie. | Rien. |
| `[!]` | Avertissement : incomplet, mais utilisable. | Lire le détail. Souvent, on peut ignorer. |
| `[✗]` | Problème bloquant pour cette catégorie. | Corriger si la catégorie vous concerne. |

> **Point capital, à lire deux fois :** **vous n'avez pas besoin de tout mettre en vert.** Un `[✗]` sur « Xcode » est parfaitement normal sous Windows ou Linux, où iOS est impossible. Un `[!]` sur « Linux toolchain » est sans conséquence si vous ne visez pas le bureau Linux.
>
> La seule ligne obligatoire est la première : `[✓] Flutter`. Ensuite, il vous faut au moins **une** cible en vert (Android, Chrome, ou une cible bureau).

### 43.10.3 — Les catégories, une par une

| Catégorie | Ce qu'elle vérifie | Obligatoire ? |
| --- | --- | --- |
| `Flutter` | Le SDK est trouvé, la version, le canal, la locale. | **Oui, toujours.** |
| `Android toolchain` | SDK Android, `platform-tools`, `cmdline-tools`, licences. | Oui si vous visez Android. |
| `Xcode` | Xcode, CocoaPods, outils en ligne de commande. | Oui si vous visez iOS/macOS. Impossible ailleurs. |
| `Chrome` | Un navigateur Chrome ou Chromium est trouvé. | Oui si vous visez le Web. |
| `Windows / Linux toolchain` | Visual Studio ou clang/CMake/ninja/GTK. | Oui si vous visez le bureau. |
| `Android Studio` | Le logiciel et ses greffons Flutter/Dart. | Non, mais très pratique. |
| `VS Code` | L'éditeur et son extension. | Non. |
| `Connected device` | Émulateurs et appareils branchés. | Il en faut **au moins un**. |
| `Network resources` | Accès aux serveurs de Flutter et pub.dev. | Oui (proxy, pare-feu). |

### 43.10.4 — Le mode détaillé

Quand un message reste obscur :

```bash
flutter doctor -v
```

**Extrait de résultat :**

```text
[✓] Flutter (Channel stable, 3.47.0, on Ubuntu 24.04 x86_64, locale fr_FR.UTF-8)
    • Flutter version 3.47.0 on channel stable at /home/moi/develop/flutter
    • Upstream repository https://github.com/flutter/flutter.git
    • Framework revision 4d5f1e2b3c (5 days ago), 2026-08-10 09:14:22 -0700
    • Engine revision 9a8b7c6d5e
    • Dart version 3.13.0
    • DevTools version 2.52.0

[!] Android toolchain - develop for Android devices (Android SDK version 36.0.0)
    • Android SDK at /home/moi/Android/Sdk
    • Platform android-36, build-tools 36.0.0
    ✗ Android license status unknown.
      Run `flutter doctor --android-licenses` to accept the SDK licenses.
```

Le mode `-v` affiche les **chemins réels**. C'est précieux : il vous dit exactement quel SDK il a trouvé, et où.

### 43.10.5 — Les corrections les plus fréquentes

| Message de `flutter doctor` | Correction |
| --- | --- |
| `Unable to locate Android SDK` | Installer Android Studio (43.13). |
| `Android license status unknown` | `flutter doctor --android-licenses` puis `y` à chaque question. |
| `cmdline-tools component is missing` | SDK Manager > SDK Tools > cocher **Android SDK Command-line Tools**. |
| `Xcode installation is incomplete` | `sudo xcodebuild -runFirstLaunch`. |
| `CocoaPods not installed` | `sudo gem install cocoapods` (macOS). |
| `Chrome - develop for the web: Cannot find Chrome` | Installer Chrome, ou définir `CHROME_EXECUTABLE`. |
| `No devices available` | Démarrer un émulateur, ou brancher un téléphone (43.14, 43.15). |
| `Network resources ... failed` | Proxy d'entreprise : configurer `HTTP_PROXY` et `HTTPS_PROXY`. |

### 43.10.6 — Le cas des licences Android

Cette étape surprend tout le monde parce qu'elle n'est pas automatique :

```bash
flutter doctor --android-licenses
```

Le terminal affiche de longs textes juridiques et demande, plusieurs fois :

```text
Accept? (y/N):
```

Tapez `y` puis `Entrée` **à chaque question** (il y en a généralement cinq à sept).

**Résultat final :**

```text
All SDK package licenses accepted.
```

Relancez ensuite `flutter doctor` : la ligne Android doit passer au vert.

---

## 43.11 — Les canaux (`stable`, `beta`) et `flutter upgrade`

Flutter est un dépôt Git. Les « canaux » sont tout simplement des branches de ce dépôt.

### 43.11.1 — Les canaux disponibles

| Canal | Contenu | Rythme | Pour qui |
| --- | --- | --- | --- |
| `stable` | version testée et recommandée | environ tous les 3 mois | **tout le monde**, et vous |
| `beta` | version candidate du prochain stable | environ tous les mois | curieux, testeurs |
| `master` | le développement en cours, non testé | en continu | contributeurs à Flutter |

> **Règle pour cette formation :** restez sur `stable`. Toujours. Une application de production n'est jamais construite depuis `master`.

### 43.11.2 — Connaître son canal

```bash
flutter channel
```

**Résultat :**

```text
Flutter channels:
  master
  beta
* stable
```

L'astérisque indique le canal actif.

### 43.11.3 — Changer de canal

```bash
flutter channel beta
flutter upgrade
```

La première commande bascule la branche, la seconde télécharge effectivement les fichiers. **Les deux sont nécessaires** : changer de canal sans faire `flutter upgrade` laisse votre installation dans un état incohérent.

Pour revenir :

```bash
flutter channel stable
flutter upgrade
```

### 43.11.4 — Mettre à jour

```bash
flutter upgrade
```

**Résultat :**

```text
Upgrading Flutter to 3.47.0 from 3.44.7 in /home/moi/develop/flutter...
Downloading Dart SDK from Flutter engine ...
Building flutter tool...
Running pub upgrade...
Flutter 3.47.0 • channel stable
Framework • revision 4d5f1e2b3c
Engine • revision 9a8b7c6d5e
Tools • Dart 3.13.0 • DevTools 2.52.0
```

Après une mise à jour, dans chacun de vos projets :

```bash
flutter pub get
```

Cela réaligne les dépendances sur le nouveau SDK.

### 43.11.5 — Revenir en arrière

Si une mise à jour casse votre projet :

```bash
flutter downgrade
```

Cette commande revient à la version précédente **du canal courant**. Elle ne fonctionne que s'il y a bien une version antérieure connue localement.

### 43.11.6 — Quand mettre à jour, et quand ne pas le faire

| Situation | Décision |
| --- | --- |
| Vous démarrez un projet | Mettez à jour d'abord. |
| Vous apprenez (cas de cette formation) | Mettez à jour tranquillement, tous les mois. |
| Vous êtes à trois jours de livrer | **Ne mettez pas à jour.** |
| Un package exige une version plus récente | Mettez à jour, puis testez tout. |
| Vous travaillez en équipe | Mettez à jour **tous ensemble**, le même jour. |

> **Conseil pratique :** notez la version de Flutter dans le `README.md` de votre projet. Quand un collègue ou vous-même reprendrez le projet dans six mois, cette ligne vous fera gagner une heure.

### 43.11.7 — Le champ `sdk` du `pubspec.yaml`

Votre projet déclare la version de Dart qu'il exige :

```yaml
environment:
  sdk: ^3.10.0
```

Si votre Flutter embarque un Dart plus ancien, `flutter pub get` refusera de fonctionner avec un message clair. C'est un garde-fou, pas un bug.

---

## 43.12 — VS Code et l'extension Flutter

Le SDK sait compiler. Il ne sait pas vous aider à écrire. C'est le rôle de l'éditeur.

Nous utilisons **Visual Studio Code** : gratuit, disponible sur les trois systèmes, léger, et son extension Flutter est excellente. C'est aussi celui que vous avez utilisé au chapitre 16.

### 43.12.1 — Installer l'extension

1. Ouvrez VS Code.
2. Cliquez sur l'icône **Extensions** dans la barre latérale (ou `Ctrl` + `Shift` + `X`, `Cmd` + `Shift` + `X` sur macOS).
3. Tapez `Flutter` dans la zone de recherche.
4. Installez l'extension **Flutter**, publiée par **Dart Code**.

L'extension **Dart** est installée automatiquement en même temps : ne l'installez pas séparément.

### 43.12.2 — Vérifier que VS Code trouve le SDK

Ouvrez la palette de commandes (`Ctrl` + `Shift` + `P`) et tapez :

```text
Flutter: Run Flutter Doctor
```

Le rapport de `flutter doctor` s'affiche dans le panneau **Sortie**. Si l'extension se plaint de ne pas trouver le SDK, ouvrez les réglages (`Ctrl` + `,`), cherchez `dart.flutterSdkPath` et renseignez le chemin complet, par exemple :

```text
/home/moi/develop/flutter
```

Attention : c'est le dossier `flutter`, **pas** `flutter/bin`. C'est l'inverse du `PATH`. Cette différence piège beaucoup de monde.

### 43.12.3 — Ce que l'extension vous apporte

| Fonction | Détail |
| --- | --- |
| Complétion | propose les widgets et leurs paramètres pendant la frappe |
| Erreurs en direct | souligne les erreurs sans compiler |
| Correctifs rapides | `Ctrl` + `.` sur une ligne soulignée |
| Refactorisations | « Wrap with Column », « Extract Widget », « Remove widget » |
| Formatage automatique | à l'enregistrement, si vous l'activez |
| Lancement et débogage | `F5`, points d'arrêt, inspection des variables |
| Hot reload automatique | à chaque enregistrement du fichier |
| Sélecteur d'appareil | en bas à droite de la fenêtre |
| Aperçu des couleurs | une pastille colorée en marge de `Colors.blue` |
| DevTools | l'inspecteur de widgets, en un clic |

### 43.12.4 — Réglages recommandés

Ouvrez les réglages au format JSON : palette de commandes, puis `Preferences: Open User Settings (JSON)`. Ajoutez :

```text
{
  "editor.formatOnSave": true,
  "editor.rulers": [80],
  "dart.previewFlutterUiGuides": true,
  "dart.openDevTools": "flutter",
  "[dart]": {
    "editor.defaultFormatter": "Dart-Code.dart-code",
    "editor.codeActionsOnSave": {
      "source.fixAll": "always"
    }
  }
}
```

| Réglage | Effet |
| --- | --- |
| `editor.formatOnSave` | formate le fichier à chaque `Ctrl` + `S` |
| `editor.rulers` | trace un repère vertical à 80 colonnes |
| `dart.previewFlutterUiGuides` | dessine des lignes reliant les widgets imbriqués |
| `dart.openDevTools` | ouvre DevTools au lancement |
| `source.fixAll` | applique les corrections automatiques à l'enregistrement |

Le troisième réglage est particulièrement utile pour un débutant : il matérialise l'arbre de widgets directement dans l'éditeur, ce qui aide énormément à ne pas se perdre dans les parenthèses.

### 43.12.5 — Les trois raccourcis à connaître par cœur

| Raccourci (Windows/Linux) | macOS | Effet |
| --- | --- | --- |
| `Ctrl` + `.` | `Cmd` + `.` | correctif rapide / envelopper un widget |
| `Ctrl` + `Espace` | `Cmd` + `Espace` | forcer la complétion |
| `F5` | `F5` | lancer en mode débogage |

Le premier est le plus précieux. Placez le curseur sur un widget, faites `Ctrl` + `.`, et VS Code vous propose « Wrap with Container », « Wrap with Padding », « Wrap with Column »... Cela vous évite de réécrire des parenthèses à la main.

### 43.12.6 — Autres éditeurs

Vous pouvez utiliser **Android Studio** ou **IntelliJ IDEA** avec les greffons Flutter et Dart. Ils sont excellents, mais plus lourds. Les captures et les raccourcis de cette formation supposent VS Code.

---

## 43.13 — Android Studio et le SDK Android

Attention à une confusion très répandue :

> **Android Studio** est un éditeur de code.
> **Le SDK Android** est l'ensemble des outils qui compilent et installent une application Android.

Vous n'êtes pas obligé d'utiliser Android Studio pour écrire du code. Mais c'est **de très loin** le moyen le plus simple d'installer et de gérer le SDK Android et les émulateurs. On l'installe donc, même si l'on code dans VS Code.

### 43.13.1 — Installer Android Studio

Téléchargez-le depuis `https://developer.android.com/studio` et installez-le avec les options par défaut.

Sur Linux, installez d'abord les bibliothèques 32 bits requises indiquées par la page d'installation d'Android Studio.

Au premier lancement, l'assistant vous propose une installation « Standard ». Acceptez : elle télécharge le SDK, la plateforme la plus récente et l'émulateur. Comptez plusieurs Go.

### 43.13.2 — Vérifier les composants du SDK

Dans Android Studio, ouvrez le **SDK Manager** :

```text
  Menu Tools  >  SDK Manager
  (ou, sur l'écran d'accueil : More Actions > SDK Manager)
```

**Onglet SDK Platforms.** Cochez la version d'API la plus récente proposée (au moment de la rédaction, **API 36**). Une seule suffit pour commencer.

**Onglet SDK Tools.** Vérifiez que ces éléments sont cochés :

```text
  [x] Android SDK Build-Tools
  [x] Android SDK Command-line Tools          ← souvent oublié !
  [x] Android Emulator
  [x] Android SDK Platform-Tools
  [x] CMake
  [x] NDK (Side by side)
```

La ligne **Android SDK Command-line Tools** est celle que `flutter doctor` réclame le plus souvent. Elle n'est pas cochée par défaut dans certaines versions. Cochez-la, cliquez sur **Apply**, puis **OK**.

`CMake` et le `NDK` ne sont indispensables que si un package utilise du code natif C/C++ ; installez-les tout de suite, cela vous évitera un aller-retour plus tard.

### 43.13.3 — Installer les greffons Flutter et Dart

Même si vous codez dans VS Code, installez les greffons : ils permettent à `flutter doctor` de reconnaître Android Studio comme un environnement complet.

```text
  Menu File  >  Settings  >  Plugins   (macOS : Android Studio > Settings > Plugins)
  Onglet Marketplace
  Chercher « Flutter »  >  Install
```

Le greffon Dart est proposé automatiquement. Acceptez, puis redémarrez Android Studio.

### 43.13.4 — Accepter les licences

Cette étape est obligatoire, et elle ne se fait pas toute seule :

```bash
flutter doctor --android-licenses
```

Répondez `y` à chaque question.

**Résultat :**

```text
All SDK package licenses accepted.
```

### 43.13.5 — Vérifier

```bash
flutter doctor
```

**Résultat attendu pour la partie Android :**

```text
[✓] Android toolchain - develop for Android devices (Android SDK version 36.0.0)
[✓] Android Studio (version 2026.1)
```

### 43.13.6 — Si Flutter ne trouve pas le SDK Android

Indiquez-lui le chemin explicitement :

```bash
flutter config --android-sdk /chemin/vers/Android/Sdk
```

Les emplacements habituels :

| Système | Chemin par défaut |
| --- | --- |
| Windows | `C:\Users\VotreNom\AppData\Local\Android\Sdk` |
| macOS | `~/Library/Android/sdk` |
| Linux | `~/Android/Sdk` |

Vous pouvez retrouver le chemin exact dans le SDK Manager : il est affiché en haut de la fenêtre, sous le libellé « Android SDK Location ».

---

## 43.14 — Créer un émulateur Android

Un **émulateur** est un téléphone Android simulé sur votre ordinateur. On l'appelle aussi **AVD** (*Android Virtual Device*).

### 43.14.1 — Créer l'appareil virtuel

Dans Android Studio :

```text
  Menu Tools  >  Device Manager
  Bouton  +  (Create Virtual Device)
```

Puis :

1. **Catégorie** : choisissez `Phone`.
2. **Modèle** : prenez un `Pixel` récent. Un modèle à écran moyen est plus confortable qu'un très grand.
3. **System Image** : choisissez une image dont le niveau d'API correspond à celui installé. Cliquez sur **Download** si nécessaire.
4. **Emulated Performance** : activez l'accélération matérielle si elle est proposée.
5. **Finish**.

> **Choix important : x86_64 ou arm64 ?** Prenez l'image qui correspond au processeur de **votre ordinateur**, pas à un vrai téléphone. Sur un PC Intel ou AMD, prenez `x86_64`. Sur un Mac Apple Silicon, prenez `arm64`. Une image qui ne correspond pas à votre processeur devra être émulée instruction par instruction : ce sera insupportablement lent.

### 43.14.2 — Démarrer l'émulateur

Depuis Android Studio, cliquez sur le bouton **Run** (triangle) en face de l'appareil dans le Device Manager.

Depuis le terminal, sans ouvrir Android Studio :

```bash
flutter emulators
```

**Résultat :**

```text
2 available emulators:

Id                 • Name                • Manufacturer • Platform
-----------------------------------------------------------------
Pixel_8_API_36     • Pixel 8 API 36      • Google       • android
Pixel_Tablet_API_36• Pixel Tablet API 36 • Google       • android

To run an emulator, run 'flutter emulators --launch <emulator id>'.
To create a new emulator, run 'flutter emulators --create [--name xyz]'.
```

Pour le lancer :

```bash
flutter emulators --launch Pixel_8_API_36
```

Pour en créer un depuis le terminal :

```bash
flutter emulators --create --name mon_pixel
```

### 43.14.3 — L'accélération matérielle

Sans accélération matérielle, l'émulateur est inutilisable : plusieurs minutes de démarrage et une interface saccadée.

| Système | Technologie | Comment l'activer |
| --- | --- | --- |
| Windows | Hyper-V ou WHPX | Activer « Plateforme d'hyperviseur Windows » dans les fonctionnalités Windows |
| Windows (ancien) | Intel HAXM | installé par le SDK Manager |
| macOS | Hypervisor.framework | activé d'office |
| Linux | KVM | `sudo apt install qemu-kvm` puis ajouter l'utilisateur au groupe `kvm` |

Sur Linux, la commande d'ajout au groupe :

```bash
sudo adduser $USER kvm
```

Il faut ensuite **fermer la session** et la rouvrir pour que l'appartenance au groupe soit prise en compte.

### 43.14.4 — Vérifier que l'émulateur est vu par Flutter

L'émulateur étant démarré :

```bash
flutter devices
```

**Résultat :**

```text
Found 3 connected devices:
  sdk gphone64 x86 64 (mobile) • emulator-5554 • android-x64 • Android 16 (API 36) (emulator)
  Chrome (web)                 • chrome        • web-javascript • Google Chrome 139.0
  Linux (desktop)              • linux         • linux-x64      • Ubuntu 24.04
```

La première ligne est votre émulateur. Le nom `emulator-5554` est son identifiant : c'est ce que vous passerez à `flutter run -d`.

### 43.14.5 — Si l'émulateur est trop lent malgré tout

| Solution | Détail |
| --- | --- |
| Utiliser un vrai téléphone | souvent plus rapide qu'un émulateur (43.15) |
| Cibler Chrome | `flutter run -d chrome`, démarrage quasi instantané |
| Cibler le bureau | `flutter run -d windows` / `-d macos` / `-d linux` |
| Réduire la définition | choisir un modèle de téléphone plus petit |
| Fermer les autres logiciels | l'émulateur réclame 2 à 4 Go de mémoire |
| Prendre une image « sans Google Play » | plus légère |

> **Conseil pour les machines modestes :** faites toute la partie 1B avec `flutter run -d chrome`. Le rendu est le même à 95 %, et vous n'attendrez jamais.

---

## 43.15 — Utiliser un vrai téléphone (mode développeur, USB)

Tester sur un vrai appareil est irremplaçable : vitesse réelle, vrai clavier, vrais gestes, vraie densité d'écran.

### 43.15.1 — Activer le mode développeur sur Android

C'est une manipulation volontairement cachée par Google.

```text
  1. Réglages  >  À propos du téléphone
  2. Trouvez la ligne « Numéro de build » (ou « Numéro de version »)
  3. Appuyez dessus SEPT fois de suite
  4. Un message apparaît : « Vous êtes maintenant développeur ! »
```

Sur certaines marques, la ligne se trouve dans **Réglages > À propos > Informations sur le logiciel**.

### 43.15.2 — Activer le débogage USB

```text
  Réglages  >  Système  >  Options pour les développeurs
  Activez  « Débogage USB »
```

Sur Xiaomi, Oppo, Realme et quelques autres, il faut **en plus** activer une option nommée « Installation via USB » ou « Débogage USB (réglages de sécurité) ».

### 43.15.3 — Brancher et autoriser

Branchez le téléphone en USB.

Un dialogue apparaît **sur le téléphone** :

```text
  Autoriser le débogage USB ?
  Empreinte de la clé RSA de l'ordinateur :
  1A:2B:3C:...

  [ ] Toujours autoriser depuis cet ordinateur

  ANNULER          AUTORISER
```

Cochez la case et touchez **AUTORISER**. Si ce dialogue n'apparaît pas, débranchez et rebranchez le câble.

### 43.15.4 — Vérifier

```bash
flutter devices
```

**Résultat :**

```text
Found 2 connected devices:
  Pixel 7 (mobile) • 2B111FDH2000AB • android-arm64 • Android 15 (API 35)
  Chrome (web)     • chrome         • web-javascript • Google Chrome 139.0
```

Puis :

```bash
flutter run -d 2B111FDH2000AB
```

### 43.15.5 — Si le téléphone n'apparaît pas

Parcourez cette liste dans l'ordre. La cause est presque toujours dans les trois premières lignes.

| Cause | Correction |
| --- | --- |
| Câble « charge seulement » | Changez de câble. C'est la cause numéro un. |
| Mode USB réglé sur « charge » | Sur le téléphone, choisissez « Transfert de fichiers (MTP) ». |
| Autorisation refusée ou expirée | Options développeur > « Révoquer les autorisations de débogage USB », rebranchez. |
| Pilote USB manquant (Windows) | Installez le pilote OEM du constructeur. |
| Serveur ADB bloqué | `adb kill-server` puis `adb devices`. |
| Permissions Linux | Ajoutez une règle `udev`, ou ajoutez-vous au groupe `plugdev`. |
| Port USB défaillant | Essayez un autre port, de préférence à l'arrière du boîtier. |

La commande de diagnostic de plus bas niveau :

```bash
adb devices
```

**Résultat correct :**

```text
List of devices attached
2B111FDH2000AB	device
```

**Résultat problématique :**

```text
List of devices attached
2B111FDH2000AB	unauthorized
```

`unauthorized` signifie que vous n'avez pas validé le dialogue sur le téléphone.

> **Remarque :** `adb` se trouve dans `<SDK Android>/platform-tools`. Si la commande est introuvable, ajoutez ce dossier au `PATH`, ou appelez-la par son chemin complet.

### 43.15.6 — Le débogage sans fil

Depuis Android 11, on peut se passer du câble.

```text
  Options pour les développeurs  >  Débogage sans fil  >  Activer
  Puis :  « Associer l'appareil à l'aide d'un code d'association »
```

Le téléphone affiche un code, une adresse IP et un port. Sur l'ordinateur :

```bash
adb pair 192.168.1.42:37135
```

Saisissez le code affiché, puis connectez-vous au port de débogage (différent du port d'association) :

```bash
adb connect 192.168.1.42:39221
```

L'ordinateur et le téléphone doivent être sur le **même réseau Wi-Fi**.

### 43.15.7 — Et un iPhone ?

Il faut un Mac, Xcode, et un compte développeur Apple (le compte gratuit suffit pour tester sur son propre appareil, avec une signature valable sept jours). Branchez, faites confiance à l'ordinateur, puis :

```bash
flutter devices
flutter run
```

Le premier lancement demande d'ouvrir `ios/Runner.xcworkspace` dans Xcode pour choisir une équipe de signature.

---

## 43.16 — Cibler le Web et le bureau

### 43.16.1 — Le Web

Le support Web est activé par défaut depuis plusieurs versions. Aucune installation n'est nécessaire à part **Chrome**.

```bash
flutter devices
```

**Résultat :**

```text
  Chrome (web)     • chrome     • web-javascript • Google Chrome 139.0
  Web Server (web) • web-server • web-javascript • Flutter Tools
```

Deux cibles apparaissent :

| Cible | Usage |
| --- | --- |
| `chrome` | Flutter ouvre Chrome et s'y connecte. Hot reload et débogage complets. |
| `web-server` | Flutter démarre seulement un serveur et vous donne une URL. Pratique pour tester dans Firefox ou Safari, ou depuis un téléphone du même réseau. |

Lancer dans Chrome :

```bash
flutter run -d chrome
```

Lancer un serveur accessible depuis un autre appareil :

```bash
flutter run -d web-server --web-hostname 0.0.0.0 --web-port 8080
```

Si `flutter doctor` ne trouve pas Chrome (par exemple si vous utilisez Chromium), indiquez-lui le chemin :

```bash
export CHROME_EXECUTABLE=/usr/bin/chromium
```

**Ce que le Web ne permet pas :** l'accès direct au système de fichiers, `dart:io`, `sqflite`. Retenez-le pour le chapitre 54.

### 43.16.2 — Le bureau

Une cible bureau ne peut être compilée que **sur le système correspondant**.

| Cible | Système requis | Outils requis |
| --- | --- | --- |
| Windows | Windows | Visual Studio avec la charge « Développement Desktop en C++ » |
| macOS | macOS | Xcode |
| Linux | Linux | `clang`, `cmake`, `ninja-build`, `pkg-config`, `libgtk-3-dev` |

Vérifiez que la cible est active :

```bash
flutter config
```

**Résultat :**

```text
Settings:
  enable-web: true
  enable-linux-desktop: true
  enable-macos-desktop: false
  enable-windows-desktop: false
```

Pour activer une cible :

```bash
flutter config --enable-windows-desktop
flutter config --enable-macos-desktop
flutter config --enable-linux-desktop
flutter config --enable-web
```

Pour en désactiver une :

```bash
flutter config --no-enable-linux-desktop
```

Puis lancez :

```bash
flutter run -d windows
flutter run -d macos
flutter run -d linux
```

> **Attention :** activer une cible dans `flutter config` ne crée pas les dossiers correspondants dans un projet **déjà existant**. Il faut, à la racine du projet, exécuter `flutter create .` pour générer les dossiers manquants (voir 43.19).

### 43.16.3 — Quelle cible choisir pour apprendre ?

| Votre situation | Cible conseillée |
| --- | --- |
| Machine puissante, téléphone Android sous la main | le téléphone |
| Machine puissante, pas de téléphone | l'émulateur |
| Machine modeste | **Chrome** |
| Vous voulez juste avancer vite dans le cours | **Chrome** |
| Vous travaillez le rendu tactile et les gestes | un vrai téléphone |

---

## 43.17 — `flutter devices`

C'est la commande que vous taperez le plus souvent, juste après `flutter run`.

```bash
flutter devices
```

**Résultat :**

```text
Found 4 connected devices:
  Pixel 7 (mobile)             • 2B111FDH2000AB • android-arm64  • Android 15 (API 35)
  sdk gphone64 x86 64 (mobile) • emulator-5554  • android-x64    • Android 16 (API 36) (emulator)
  Linux (desktop)              • linux          • linux-x64      • Ubuntu 24.04
  Chrome (web)                 • chrome         • web-javascript • Google Chrome 139.0
```

### 43.17.1 — Lire une ligne

```text
  sdk gphone64 x86 64 (mobile) • emulator-5554 • android-x64 • Android 16 (API 36) (emulator)
  └──────── nom lisible ─────┘   └── ID ─────┘   └ archi ──┘   └──── version ────┘
```

| Colonne | Rôle |
| --- | --- |
| Nom lisible | pour vous |
| **Identifiant** | **c'est celui-là qu'on passe à `-d`** |
| Architecture | processeur ciblé |
| Version | version du système ou du navigateur |

### 43.17.2 — Utiliser l'identifiant

```bash
flutter run -d emulator-5554
flutter run -d chrome
flutter run -d linux
```

Un préfixe suffit s'il est sans ambiguïté :

```bash
flutter run -d emu
```

### 43.17.3 — Aucun appareil trouvé

**Résultat :**

```text
No devices found yet. Checking for wireless devices...

No supported devices found with name or id matching ''.
Run "flutter emulators" to list and start any available device emulators.
```

Dans ce cas, dans l'ordre :

1. `flutter emulators --launch <id>` pour démarrer un émulateur ;
2. brancher un téléphone (43.15) ;
3. `flutter run -d chrome` : le Web est toujours disponible.

### 43.17.4 — Voir aussi ce qui **n'est pas** disponible

```bash
flutter devices --machine
```

produit la même liste au format JSON, utile pour un script. Et pour comprendre pourquoi une plateforme manque :

```bash
flutter doctor -v
```

---

## 43.18 — `flutter create mon_appli`

L'outillage est en place. Créons enfin un projet.

Placez-vous dans le dossier où vous rangez vos projets, puis :

```bash
flutter create mon_appli
```

**Résultat :**

```text
Creating project mon_appli...
Resolving dependencies in `mon_appli`...
Downloading packages...
Got dependencies in `mon_appli`.
Wrote 129 files.

All done!
You can find general documentation for Flutter at: https://docs.flutter.dev/
Detailed API documentation is available at: https://api.flutter.dev/
If you prefer video documentation, consider: https://www.youtube.com/c/flutterdev

In order to run your application, type:

  $ cd mon_appli
  $ flutter run

Your application code is in mon_appli/lib/main.dart.
```

Cent vingt-neuf fichiers pour une application qui affiche un compteur. C'est normal, et la section 43.20 vous dira exactement à quoi ils servent.

### 43.18.1 — Les règles de nommage

Le nom du projet devient le nom du package Dart. Il doit donc respecter les mêmes règles qu'au chapitre 16 :

| Règle | Exemple correct | Exemple incorrect |
| --- | --- | --- |
| Que des minuscules | `mon_appli` | `MonAppli` |
| Mots séparés par `_` | `carnet_de_bord` | `carnet-de-bord` |
| Pas d'accent | `journal_de_jeu` | `journal_de_jeù` |
| Ne commence pas par un chiffre | `appli2` | `2appli` |
| Pas un mot réservé de Dart | `mon_test` | `class`, `switch` |
| Pas le nom d'un package existant si vous publiez | `mon_appli_perso` | `http` |

Si vous vous trompez, Flutter refuse clairement :

```bash
flutter create MonAppli
```

**Résultat :**

```text
"MonAppli" is not a valid Dart package name.

The name should be all lowercase, with underscores to separate words,
"just_like_this". Use only basic Latin letters and Arabic digits: [a-z0-9_].
```

### 43.18.2 — Créer dans le dossier courant

Le dernier argument est en réalité un **chemin de dossier**, pas un nom.

```bash
mkdir mon_appli
cd mon_appli
flutter create .
```

Le point signifie « ici ». Flutter prend alors le nom du dossier courant comme nom de projet.

Cette forme sert surtout à **réparer** ou **compléter** un projet existant : elle régénère les fichiers manquants sans toucher à votre `lib/`. Nous nous en servirons en 43.19.

### 43.18.3 — Ce que Flutter a fait pour vous

```text
  flutter create mon_appli
      │
      ├─ crée l'arborescence des dossiers
      ├─ écrit un pubspec.yaml complet
      ├─ écrit lib/main.dart (l'application de démonstration)
      ├─ génère les projets natifs Android, iOS, Web, bureau
      ├─ écrit un test d'exemple dans test/
      ├─ écrit analysis_options.yaml et .gitignore
      └─ lance « flutter pub get » pour télécharger les dépendances
```

### 43.18.4 — Premier lancement

```bash
cd mon_appli
flutter run
```

S'il n'y a qu'un seul appareil disponible, Flutter le choisit tout seul. S'il y en a plusieurs, il vous demande :

```text
Multiple devices found:
Pixel 7 (mobile)             • 2B111FDH2000AB • android-arm64  • Android 15 (API 35)
sdk gphone64 x86 64 (mobile) • emulator-5554  • android-x64    • Android 16 (API 36)
Chrome (web)                 • chrome         • web-javascript • Google Chrome 139.0
[1]: Pixel 7 (2B111FDH2000AB)
[2]: sdk gphone64 x86 64 (emulator-5554)
[3]: Chrome (chrome)
Please choose one (or "q" to quit):
```

Tapez le numéro et validez.

Le premier lancement est **long** : de deux à dix minutes selon la machine, car Gradle télécharge la chaîne de compilation Android. Les suivants prennent quelques secondes. Ne concluez pas que « Flutter est lent » sur cette base.

---

## 43.19 — Les options de `flutter create`

Toutes les options sont listées par :

```bash
flutter create --help
```

Voici celles qui comptent vraiment.

### 43.19.1 — `--org` : l'identifiant d'organisation

C'est l'option la plus importante, et la plus souvent oubliée.

```bash
flutter create --org com.monstudio mon_appli
```

Cet identifiant, écrit en **notation de domaine inversée**, sert à construire :

- le nom de paquet Java/Kotlin sous Android : `com.monstudio.mon_appli` ;
- l'identifiant de lot (« bundle identifier ») sous iOS : `com.monstudio.monAppli`.

Cet identifiant est **l'identité mondiale de votre application** dans les boutiques. Deux applications ne peuvent pas partager le même.

La valeur par défaut est :

```text
com.example
```

`com.example` est un domaine réservé aux exemples. **Google Play et l'App Store refusent toute application dont l'identifiant commence par `com.example`.**

Comment le choisir :

| Vous possédez | Utilisez |
| --- | --- |
| Le domaine `monstudio.fr` | `fr.monstudio` |
| Le domaine `jeux-lyra.com` | `com.jeuxlyra` (pas de tiret) |
| Aucun domaine | `com.votrepseudo` ou `io.github.votrepseudo` |

> **Avertissement :** changer l'`--org` **après** la création demande de modifier une dizaine de fichiers dans `android/` et `ios/`. Choisissez-le au moment de la création. C'est trente secondes de réflexion qui vous économisent une heure.

### 43.19.2 — `--platforms` : les plateformes à générer

Par défaut, Flutter génère les dossiers de **toutes** les plateformes disponibles sur votre machine. Cela fait beaucoup de fichiers dont vous n'avez pas besoin.

```bash
flutter create --platforms=android,web mon_appli
```

Valeurs acceptées :

```text
  android    ios    web    windows    macos    linux    darwin
```

Séparez-les par des virgules, **sans espace**.

Exemples typiques :

```bash
# Une application mobile uniquement
flutter create --platforms=android,ios mon_appli

# Un prototype web, très léger
flutter create --platforms=web mon_prototype

# Ce que nous utiliserons dans cette formation
flutter create --org com.monstudio --platforms=android,web mon_appli
```

**Ajouter une plateforme plus tard** est simple : depuis la racine du projet,

```bash
flutter create --platforms=windows .
```

Flutter ajoute le dossier `windows/` sans rien détruire dans `lib/`.

### 43.19.3 — `--project-name` : dissocier dossier et package

Utile quand le nom du dossier ne peut pas être un nom de package valide.

```bash
flutter create --project-name mon_appli "Mon Appli"
```

Le dossier s'appellera `Mon Appli`, mais le package Dart `mon_appli`.

En pratique, évitez : gardez le même nom pour les deux, cela évite les confusions.

### 43.19.4 — `--empty` : partir d'une page blanche

```bash
flutter create --empty mon_appli
```

Le `lib/main.dart` généré est alors minimal, sans l'application « compteur » ni ses cent lignes de commentaires.

C'est excellent **une fois que vous savez lire le fichier complet**. Pour votre tout premier projet, gardez le modèle par défaut : nous allons le décortiquer en 43.30.

### 43.19.5 — `--template` : le type de projet

```bash
flutter create --template=package ma_bibliotheque
```

| Valeur | Produit | Usage |
| --- | --- | --- |
| `app` | une application complète | **le défaut**, c'est ce que vous voulez |
| `module` | un module intégrable dans une appli native existante | intégration progressive |
| `package` | une bibliothèque Dart pure, sans interface | code réutilisable |
| `plugin` | un package avec du code natif par plateforme | accès au matériel |
| `plugin_ffi` | un plugin qui appelle du C via FFI | interop C |
| `package_ffi` | un package FFI | interop C |

Pour toute la partie 1B et 1C, c'est `app`, donc rien à préciser.

### 43.19.6 — `--description` : la description du projet

```bash
flutter create --description "Le carnet de bord de mes parties" mon_appli
```

Elle est écrite dans le champ `description` du `pubspec.yaml`. Vous pouvez aussi la modifier après coup, c'est sans conséquence.

### 43.19.7 — `--android-language`

```bash
flutter create --android-language kotlin mon_appli
```

La valeur par défaut est déjà `kotlin`, qui est le choix recommandé. `java` existe encore pour les projets anciens. Vous n'aurez normalement jamais à toucher à cette option.

### 43.19.8 — La commande complète recommandée

Pour cette formation :

```bash
flutter create --org com.monstudio --platforms=android,web mon_appli
```

Remplacez `com.monstudio` par votre propre identifiant.

---

## 43.20 — Arborescence d'un projet Flutter, dossier par dossier

Voici ce que `flutter create` a produit. Ne prenez pas peur : sur ces cent vingt-neuf fichiers, vous en toucherez **trois**.

```text
mon_appli/
│
├── lib/                       ← VOTRE CODE DART. C'est ici que vous vivez.
│   └── main.dart              ← le point d'entrée de l'application
│
├── test/                      ← les tests automatiques
│   └── widget_test.dart
│
├── android/                   ← le projet Android natif (Gradle, Kotlin)
├── ios/                       ← le projet iOS natif (Xcode, Swift)
├── web/                       ← les fichiers du site (index.html, manifeste)
├── windows/                   ← le projet Windows natif (C++, CMake)
├── macos/                     ← le projet macOS natif
├── linux/                     ← le projet Linux natif
│
├── build/                     ← GÉNÉRÉ. Ne jamais éditer, ne jamais versionner.
├── .dart_tool/                ← GÉNÉRÉ. Cache de l'outillage Dart.
│
├── pubspec.yaml               ← la carte d'identité : nom, version, dépendances
├── pubspec.lock               ← GÉNÉRÉ. Les versions exactes retenues.
├── analysis_options.yaml      ← les règles de l'analyseur
├── .gitignore                 ← ce que Git doit ignorer
├── .metadata                  ← GÉNÉRÉ. Suivi de version par l'outil Flutter.
└── README.md                  ← la documentation de votre projet
```

### 43.20.1 — Le tableau de référence

| Élément | Vous l'éditez ? | Versionné dans Git ? | Rôle |
| --- | --- | --- | --- |
| `lib/` | **oui, tout le temps** | oui | votre code Dart |
| `test/` | oui | oui | vos tests |
| `pubspec.yaml` | oui | oui | nom, version, dépendances, assets |
| `analysis_options.yaml` | oui, rarement | oui | règles de l'analyseur |
| `README.md` | oui | oui | documentation |
| `android/` | rarement | oui | permissions, icône, signature |
| `ios/` | rarement | oui | permissions, icône, signature |
| `web/` | rarement | oui | titre de l'onglet, favicon |
| `windows/`, `macos/`, `linux/` | rarement | oui | configuration bureau |
| `pubspec.lock` | **jamais** | oui pour une appli | versions figées |
| `build/` | **jamais** | **non** | sortie de compilation |
| `.dart_tool/` | **jamais** | **non** | cache |
| `.metadata` | **jamais** | oui | suivi interne |

Trois règles à graver :

1. **Vous travaillez dans `lib/`.**
2. **`build/` et `.dart_tool/` sont jetables.** Vous pouvez les supprimer à tout moment ; ils se régénèrent.
3. **`pubspec.lock` se versionne pour une application**, se laisse hors de Git pour une bibliothèque publiée. C'est la même règle qu'au chapitre 16.

### 43.20.2 — `build/` et `.dart_tool/`

Ces deux dossiers grossissent très vite : quelques centaines de mégaoctets ne sont pas rares.

Pour les nettoyer :

```bash
flutter clean
```

**Résultat :**

```text
Deleting build...                                                   112ms
Deleting .dart_tool...                                               24ms
Deleting Generated.xcconfig...                                        0ms
Deleting flutter_export_environment.sh...                             0ms
Deleting .flutter-plugins-dependencies...                             0ms
```

Après un `flutter clean`, il faut refaire :

```bash
flutter pub get
```

`flutter clean` est le remède universel quand une compilation échoue « sans raison ». C'est l'équivalent de « éteindre et rallumer ». Il est un peu magique, mais il fonctionne souvent — et il ne détruit jamais votre code.

### 43.20.3 — Le `.gitignore` généré

Flutter écrit un `.gitignore` correct pour vous :

```text
# Miscellaneous
*.class
*.log
*.swp
.DS_Store

# IntelliJ related
*.iml
*.ipr
*.iws
.idea/

# Flutter/Dart/Pub related
**/doc/api/
.dart_tool/
build/
.flutter-plugins
.flutter-plugins-dependencies
.pub-cache/
.pub/

# Symbolication and obfuscation
app.*.symbols
app.*.map.json
```

Vous n'avez rien à ajouter pour commencer.

### 43.20.4 — Schéma mental à retenir

```text
  ┌───────────────────────────────────────────────────────────┐
  │                                                           │
  │      lib/           ←  99 % de votre temps                │
  │      pubspec.yaml   ←   1 % de votre temps                │
  │                                                           │
  │      tout le reste  ←  vous y allez trois fois par an     │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

---

## 43.21 — Le dossier `lib/`

`lib` est l'abréviation de « library ». C'est le dossier de votre code Dart.

### 43.21.1 — Une seule règle imposée

Flutter n'impose qu'une chose : le fichier lancé par défaut est

```text
  lib/main.dart
```

et il doit contenir une fonction `main()`.

Tout le reste de l'organisation est **libre**.

### 43.21.2 — L'organisation par couches

C'est celle que nous utiliserons dans la partie 1C.

```text
lib/
├── main.dart                  point d'entrée, uniquement runApp()
│
├── models/                    les données pures, sans interface
│   ├── joueur.dart
│   ├── ennemi.dart
│   └── partie.dart
│
├── screens/                   un fichier par écran complet
│   ├── ecran_accueil.dart
│   ├── ecran_jeu.dart
│   └── ecran_scores.dart
│
├── widgets/                   les morceaux réutilisables
│   ├── carte_joueur.dart
│   ├── barre_de_vie.dart
│   └── bouton_principal.dart
│
├── services/                  réseau, stockage, calculs
│   ├── api_scores.dart
│   └── stockage_local.dart
│
└── utils/                     constantes et fonctions utilitaires
    ├── couleurs.dart
    └── formats.dart
```

Vous reconnaissez la logique du chapitre 16 : **un fichier, une responsabilité**.

### 43.21.3 — L'organisation par fonctionnalité

Sur un gros projet, on préfère souvent regrouper par domaine métier :

```text
lib/
├── main.dart
├── commun/
│   ├── widgets/
│   └── theme/
├── authentification/
│   ├── ecran_connexion.dart
│   ├── modele_utilisateur.dart
│   └── service_auth.dart
├── catalogue/
│   ├── ecran_liste.dart
│   ├── ecran_detail.dart
│   └── modele_produit.dart
└── panier/
    ├── ecran_panier.dart
    └── service_panier.dart
```

L'avantage : tout ce qui concerne « le panier » est au même endroit. On peut supprimer la fonctionnalité en supprimant un dossier.

### 43.21.4 — Comment importer entre vos fichiers

Deux formes, exactement comme au chapitre 16.

```dart
// Chemin relatif : privilégié à l'intérieur d'un même dossier.
import 'barre_de_vie.dart';
import '../models/joueur.dart';

// Chemin absolu de package : privilégié entre dossiers éloignés.
import 'package:mon_appli/models/joueur.dart';
```

`mon_appli` est le nom déclaré dans le champ `name` du `pubspec.yaml`. C'est ce qui rend la deuxième forme possible.

> **Recommandation :** utilisez `package:` dès que vous changez de dossier. Vos imports restent valables si vous déplacez le fichier, et il n'y a pas de suite de `../../..` illisible.

### 43.21.5 — Ce qu'il ne faut pas mettre dans `lib/`

| Ne mettez pas | Mettez plutôt |
| --- | --- |
| Des images | `assets/images/` (chapitre 47) |
| Des polices | `assets/fonts/` |
| Des tests | `test/` |
| Des fichiers de données volumineux | `assets/` |
| Du code natif | `android/`, `ios/` |

### 43.21.6 — Quand découper ?

Une règle simple et efficace pour un débutant :

> **Dès qu'un fichier dépasse 200 à 300 lignes, découpez-le.**

Et dès qu'un widget est utilisé à deux endroits, sortez-le dans `widgets/`. Nous appliquerons cette règle dès le chapitre 44.

---

## 43.22 — Les dossiers `android/`, `ios/`, `web/`

Ces dossiers contiennent un **vrai projet natif** pour chaque plateforme. C'est ce projet natif qui est réellement compilé ; votre code Dart y est embarqué.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  lib/main.dart  (Dart)                                       │
  │        │                                                     │
  │        │  compilé en code machine ou en JavaScript           │
  │        ▼                                                     │
  │  ┌────────────────┐  ┌───────────────┐  ┌─────────────────┐  │
  │  │  android/      │  │  ios/         │  │  web/           │  │
  │  │  → un .apk     │  │  → un .ipa    │  │  → un site      │  │
  │  └────────────────┘  └───────────────┘  └─────────────────┘  │
  └──────────────────────────────────────────────────────────────┘
```

### 43.22.1 — `android/`

```text
android/
├── app/
│   ├── build.gradle.kts            configuration de compilation de l'appli
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml  ← permissions, nom, activité principale
│       │   ├── kotlin/.../MainActivity.kt
│       │   └── res/                 ← icônes, couleurs, libellés
│       ├── debug/
│       │   └── AndroidManifest.xml  ← ajouts propres au mode débogage
│       └── profile/
├── build.gradle.kts                configuration du projet
├── gradle.properties               mémoire allouée à Gradle, options
├── settings.gradle.kts             modules et versions des greffons
└── gradle/wrapper/                 la version de Gradle à télécharger
```

Ce que vous y ferez, un jour :

| Besoin | Fichier |
| --- | --- |
| Ajouter une permission (Internet, caméra) | `app/src/main/AndroidManifest.xml` |
| Changer le nom affiché sous l'icône | `AndroidManifest.xml`, attribut `android:label` |
| Changer l'icône | dossiers `res/mipmap-*` |
| Changer la version minimale d'Android | `app/build.gradle.kts`, champ `minSdk` |
| Signer l'application pour publication | `app/build.gradle.kts` + un fichier `key.properties` |

Exemple d'ajout de permission, dans `AndroidManifest.xml` :

```text
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application
        android:label="Mon Appli"
        android:icon="@mipmap/ic_launcher">
        ...
    </application>
</manifest>
```

> **Bonne nouvelle :** la permission `INTERNET` est déjà présente dans le manifeste de **débogage** généré par Flutter. C'est pourquoi le chapitre 53 fonctionnera sans configuration en mode debug. Pour une version publiée, il faudra l'ajouter au manifeste principal.

### 43.22.2 — `ios/`

```text
ios/
├── Runner/
│   ├── Info.plist                 ← nom, permissions, orientations
│   ├── AppDelegate.swift
│   └── Assets.xcassets/           ← icônes et écran de lancement
├── Runner.xcodeproj/
├── Runner.xcworkspace/            ← à ouvrir dans Xcode (pas le .xcodeproj)
└── Podfile                        ← dépendances natives (CocoaPods)
```

Les permissions iOS s'écrivent dans `Info.plist` avec un texte **expliqué à l'utilisateur** :

```text
<key>NSCameraUsageDescription</key>
<string>Cette application utilise l'appareil photo pour votre avatar.</string>
```

Apple **refuse** une application qui demande une permission sans fournir ce texte.

### 43.22.3 — `web/`

Le plus simple des trois.

```text
web/
├── index.html          la page qui héberge l'application
├── manifest.json       nom, couleurs, icônes pour l'installation PWA
├── favicon.png         l'icône de l'onglet
└── icons/              les icônes de l'application installée
```

Dans `index.html`, la balise à personnaliser :

```text
<title>Mon Appli</title>
```

C'est ce qui s'affiche dans l'onglet du navigateur. Le titre passé à `MaterialApp` ne change pas cela.

### 43.22.4 — Faut-il versionner ces dossiers ?

**Oui.** Ils contiennent de la configuration que vous modifierez et que vous ne voudrez pas refaire. Le `.gitignore` généré exclut déjà les sous-dossiers de compilation à l'intérieur.

### 43.22.5 — En cas de dégâts

Si vous cassez un de ces dossiers en bidouillant :

```bash
rm -rf android
flutter create --platforms=android .
```

Flutter le régénère. Vous perdrez vos modifications de configuration, mais **jamais** votre code `lib/`.

---

## 43.23 — Le `pubspec.yaml` d'une application Flutter

Vous connaissez déjà ce fichier depuis le chapitre 16. La version Flutter ajoute une section.

Voici le fichier généré, entier :

```yaml
name: mon_appli
description: "A new Flutter project."
publish_to: 'none'

version: 1.0.0+1

environment:
  sdk: ^3.10.0

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true
```

Reprenons champ par champ.

### 43.23.1 — `name`

```yaml
name: mon_appli
```

Le nom du package. C'est lui qui apparaît dans vos imports :

```dart
import 'package:mon_appli/models/joueur.dart';
```

Le changer oblige à corriger tous les imports du projet. Ne le changez pas à la légère.

### 43.23.2 — `description`

```yaml
description: "Un carnet de bord pour mes parties."
```

Purement documentaire. Modifiez-la, cela fait sérieux.

### 43.23.3 — `publish_to: 'none'`

```yaml
publish_to: 'none'
```

Cette ligne dit : « ce package ne doit **jamais** être publié sur pub.dev ». C'est un garde-fou : une application n'a rien à faire sur pub.dev.

**Ne supprimez pas cette ligne** dans une application.

### 43.23.4 — `version`

```yaml
version: 1.0.0+1
```

Ce champ a une syntaxe particulière en Flutter :

```text
   1  .  0  .  0  +  1
   │     │     │     │
   │     │     │     └── numéro de build (versionCode Android, CFBundleVersion iOS)
   │     │     └──────── correctif
   │     └────────────── version mineure
   └──────────────────── version majeure
```

| Partie | Vue par | Règle |
| --- | --- | --- |
| `1.0.0` | l'utilisateur, dans la boutique | version sémantique |
| `+1` | la boutique uniquement | **doit augmenter à chaque envoi** |

Google Play refuse un envoi dont le numéro de build n'a pas augmenté, même si la version affichée a changé. Retenez-le pour le jour de votre première publication.

### 43.23.5 — `environment`

```yaml
environment:
  sdk: ^3.10.0
```

La version de **Dart** exigée. Le caret `^3.10.0` signifie « au moins 3.10.0 et strictement inférieur à 4.0.0 » — exactement la règle du chapitre 16.

On peut aussi exiger une version minimale de Flutter :

```yaml
environment:
  sdk: ^3.10.0
  flutter: ">=3.44.0"
```

Ce second champ est facultatif, mais utile en équipe.

### 43.23.6 — `dependencies`

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
```

Deux formes différentes apparaissent ici, et c'est important.

```yaml
  flutter:
    sdk: flutter
```

Cette forme signifie : « prends le package `flutter` **dans le SDK installé** », et non sur pub.dev. C'est pour cela qu'il n'y a pas de numéro de version : la version est celle de votre SDK.

```yaml
  cupertino_icons: ^1.0.8
```

Forme classique : un package téléchargé depuis pub.dev, avec une contrainte de version.

`cupertino_icons` fournit les icônes de style iOS. Vous pouvez le retirer si vous n'en voulez pas ; il ne pèse presque rien.

### 43.23.7 — `dev_dependencies`

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0
```

Ces packages ne sont **pas** embarqués dans l'application livrée. Ils servent pendant le développement :

| Package | Rôle |
| --- | --- |
| `flutter_test` | écrire des tests de widgets |
| `flutter_lints` | le jeu de règles officiel de l'analyseur |

### 43.23.8 — La section `flutter:`

C'est la nouveauté par rapport au chapitre 16.

```yaml
flutter:
  uses-material-design: true
```

`uses-material-design: true` embarque la police d'icônes Material. Sans elle, tous vos `Icon(Icons.star)` afficheraient un carré vide. **Laissez cette ligne.**

C'est aussi dans cette section que l'on déclarera les images et les polices, au chapitre 47 :

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/data/niveaux.json

  fonts:
    - family: Pixelade
      fonts:
        - asset: assets/fonts/Pixelade-Regular.ttf
        - asset: assets/fonts/Pixelade-Bold.ttf
          weight: 700
```

### 43.23.9 — Les pièges du YAML

Le YAML est **sensible à l'indentation**, et il refuse les tabulations.

| Piège | Symptôme |
| --- | --- |
| Une tabulation au lieu d'espaces | `Error on line 12: found a tab character` |
| Mauvaise indentation | le champ est ignoré, sans message clair |
| Deux espaces manquants sous `flutter:` | l'asset n'est pas trouvé à l'exécution |
| Un `-` oublié dans une liste | erreur de parsing |
| Un accent dans une clé | erreur de parsing |

Règle mnémotechnique : **deux espaces par niveau, jamais de tabulation.** Configurez votre éditeur pour convertir les tabulations en espaces dans les fichiers `.yaml`.

---

## 43.24 — `flutter pub get` et `flutter pub add`

Vous connaissez `dart pub` depuis le chapitre 16. En Flutter, on écrit `flutter pub`.

> **Règle simple :** dans un projet Flutter, on utilise **toujours** `flutter pub`, jamais `dart pub`. La première commande connaît les packages spécifiques à Flutter ; la seconde peut produire un état incohérent.

### 43.24.1 — `flutter pub get`

```bash
flutter pub get
```

**Résultat :**

```text
Resolving dependencies...
Downloading packages...
  characters 1.4.0 (1.4.1 available)
  collection 1.19.1 (1.20.0 available)
  cupertino_icons 1.0.8
  material_color_utilities 0.13.0
  meta 1.17.0
Got dependencies!
2 packages have newer versions incompatible with dependency constraints.
Try `flutter pub outdated` for more information.
```

Cette commande lit `pubspec.yaml`, résout les versions, télécharge les packages et écrit `pubspec.lock`.

Quand la lancer :

| Situation | Pourquoi |
| --- | --- |
| Après avoir modifié `pubspec.yaml` à la main | pour appliquer le changement |
| Après avoir récupéré un projet depuis Git | les packages ne sont pas versionnés |
| Après `flutter clean` | tout a été effacé |
| Après `flutter upgrade` | réaligner sur le nouveau SDK |
| Quand l'éditeur affiche `Target of URI doesn't exist` | c'est souvent cela |

> **Astuce :** dans VS Code, enregistrer le `pubspec.yaml` déclenche automatiquement `flutter pub get`. Vous n'aurez donc presque jamais à taper la commande.

### 43.24.2 — `flutter pub add`

C'est la bonne façon d'ajouter une dépendance. Ne modifiez pas `pubspec.yaml` à la main pour cela.

```bash
flutter pub add http
```

**Résultat :**

```text
Resolving dependencies...
+ http 1.5.0
+ http_parser 4.1.2
Changed 2 dependencies!
```

Et le `pubspec.yaml` a été modifié pour vous :

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  http: ^1.5.0
```

Pourquoi cette commande plutôt qu'une édition manuelle :

- elle écrit **la dernière version compatible**, vous n'avez pas à la chercher ;
- elle écrit la contrainte au bon format ;
- elle lance `pub get` dans la foulée ;
- elle refuse tout de suite si le package est incompatible avec votre SDK.

Plusieurs packages d'un coup :

```bash
flutter pub add provider shared_preferences intl
```

Un package de développement :

```bash
flutter pub add --dev build_runner
```

### 43.24.3 — Retirer un package

```bash
flutter pub remove http
```

**Résultat :**

```text
Resolving dependencies...
These packages are no longer being depended on:
- http 1.5.0
- http_parser 4.1.2
Changed 2 dependencies!
```

### 43.24.4 — Voir ce qui est périmé

```bash
flutter pub outdated
```

**Résultat :**

```text
Showing outdated packages.
[*] indicates versions that are not the latest available.

Package Name  Current  Upgradable  Resolvable  Latest

direct dependencies:
http          *1.4.0   1.5.0       1.5.0       1.5.0

dev_dependencies: all up-to-date.

1 upgradable dependency is locked (in pubspec.lock) to an older version.
To update it, use `flutter pub upgrade`.
```

Lecture des colonnes :

| Colonne | Sens |
| --- | --- |
| `Current` | ce que vous utilisez réellement |
| `Upgradable` | ce que vous obtiendriez sans toucher au `pubspec.yaml` |
| `Resolvable` | ce que vous obtiendriez en assouplissant les contraintes |
| `Latest` | la dernière version publiée |

### 43.24.5 — Mettre à jour

```bash
flutter pub upgrade
```

met à jour dans les limites des contraintes écrites dans `pubspec.yaml`.

```bash
flutter pub upgrade --major-versions
```

réécrit les contraintes pour passer aux versions majeures suivantes. **À faire seulement quand vous avez le temps de tout retester** : une version majeure peut casser du code.

### 43.24.6 — Le rôle de `pubspec.lock`

`pubspec.yaml` dit « je veux `http` version 1.x ».
`pubspec.lock` dit « et aujourd'hui, c'est exactement `1.5.0` ».

Sans ce fichier, deux développeurs qui installent le même projet à deux jours d'écart pourraient obtenir deux versions différentes, et l'un des deux aurait un bug incompréhensible.

> **Règle, identique au chapitre 16 :** on versionne `pubspec.lock` pour une **application**, on l'exclut pour une **bibliothèque publiée**.

### 43.24.7 — Choisir un package sur pub.dev

Avant d'ajouter un package, ouvrez sa page sur `https://pub.dev` et regardez :

| Indicateur | Ce qu'on veut voir |
| --- | --- |
| Score « Likes » | signe d'adoption |
| Score « Pub Points » | proche du maximum |
| Badge « Dart 3 compatible » | présent |
| Date de la dernière publication | moins d'un an |
| Plateformes supportées | celles que vous visez |
| Badge « Flutter Favorite » | excellent signe |
| Section « Changelog » | bien tenue |

Un package abandonné depuis trois ans finira par vous coûter plus cher que le temps qu'il vous fait gagner.

---

## 43.25 — `flutter run`

La commande qui compile, installe et lance votre application.

```bash
cd mon_appli
flutter run
```

**Résultat :**

```text
Launching lib/main.dart on sdk gphone64 x86 64 in debug mode...
Running Gradle task 'assembleDebug'...                             48,2s
✓ Built build/app/outputs/flutter-apk/app-debug.apk
Installing build/app/outputs/flutter-apk/app-debug.apk...            3,1s
Syncing files to device sdk gphone64 x86 64...                       112ms

Flutter run key commands.
r Hot reload.
R Hot restart.
h List all available interactive commands.
d Detach (terminate "flutter run" but leave application running).
c Clear the screen
q Quit (terminate the application on the device).

A Dart VM Service on sdk gphone64 x86 64 is available at:
http://127.0.0.1:41235/xY7bQ2zK8pM=/
The Flutter DevTools debugger and profiler on sdk gphone64 x86 64 is available at:
http://127.0.0.1:9101?uri=http://127.0.0.1:41235/xY7bQ2zK8pM=/
```

Ce terminal reste **occupé** tant que l'application tourne. C'est normal : il est votre console de pilotage. Ouvrez un second terminal si vous avez d'autres commandes à taper.

### 43.25.1 — Les touches interactives

| Touche | Effet |
| --- | --- |
| `r` | **hot reload** : recharge le code, garde l'état |
| `R` | **hot restart** : recharge le code, remet l'état à zéro |
| `h` | affiche toutes les commandes disponibles |
| `d` | détache : la commande se termine, l'application continue |
| `c` | efface l'écran du terminal |
| `q` | quitte et ferme l'application |

Attention : `R` est un **R majuscule**. Sur un clavier AZERTY, c'est `Maj` + `R`.

### 43.25.2 — Choisir l'appareil

```bash
flutter run -d chrome
flutter run -d emulator-5554
flutter run -d 2B111FDH2000AB
flutter run -d linux
```

Lancer sur **tous** les appareils connectés en même temps :

```bash
flutter run -d all
```

C'est spectaculaire, et très utile pour comparer un rendu Android et un rendu Web côte à côte.

### 43.25.3 — Les trois modes de compilation

C'est un point que beaucoup de débutants ignorent, et qui explique bien des malentendus sur « les performances de Flutter ».

| Mode | Commande | Compilation | Hot reload | Vitesse | Usage |
| --- | --- | --- | --- | --- | --- |
| debug | `flutter run` | JIT | **oui** | lente | développement |
| profile | `flutter run --profile` | AOT | non | réelle | mesurer les performances |
| release | `flutter run --release` | AOT | non | maximale | production |

En mode **debug** :

- le code est compilé « juste à temps » (JIT), ce qui permet le hot reload ;
- les vérifications (`assert`) sont actives ;
- la bannière rouge « DEBUG » s'affiche en haut à droite ;
- l'application est **notablement plus lente** et plus lourde.

> **Règle absolue :** ne jugez **jamais** les performances d'une application Flutter en mode debug. Une animation qui saccade en debug peut être parfaitement fluide en release. Pour mesurer, utilisez `--profile`.

Comparez vous-même :

```bash
flutter run --release
```

La différence de fluidité est immédiatement visible sur un émulateur.

### 43.25.4 — Les options utiles

| Option | Effet |
| --- | --- |
| `-d <id>` | choisir l'appareil |
| `--release` | mode release |
| `--profile` | mode profil |
| `-v` | sortie détaillée, pour diagnostiquer |
| `-t lib/autre.dart` | lancer un autre fichier que `lib/main.dart` |
| `--dart-define=CLE=valeur` | passer une constante à la compilation |
| `--web-port=8080` | fixer le port en Web |
| `--no-sound-null-safety` | obsolète, n'existe plus |

### 43.25.5 — Depuis VS Code

Vous n'êtes pas obligé de passer par le terminal.

| Action | Raccourci |
| --- | --- |
| Lancer en débogage | `F5` |
| Lancer sans débogage | `Ctrl` + `F5` |
| Arrêter | `Maj` + `F5` |
| Hot reload | à l'enregistrement, ou `Ctrl` + `F5` |
| Hot restart | `Maj` + `Ctrl` + `F5` |
| Choisir l'appareil | cliquer sur le nom en bas à droite |

Le mode débogage vous donne en plus les points d'arrêt et l'inspection des variables, exactement comme en Dart console.

### 43.25.6 — `flutter build`, pour livrer

`flutter run` sert à développer. Pour produire un fichier à distribuer :

```bash
flutter build apk              # un .apk Android
flutter build appbundle        # un .aab pour Google Play
flutter build ios              # iOS (macOS requis)
flutter build web              # un site statique dans build/web
flutter build windows          # un exécutable Windows
flutter build linux            # un exécutable Linux
flutter build macos            # une application macOS
```

Ces commandes compilent en mode release par défaut. Nous n'en aurons pas besoin avant la fin de la partie 1C, mais vous savez qu'elles existent.

---

## 43.26 — Le hot reload et le hot restart : la différence

C'est **la** fonction qui fait aimer Flutter. Comprenez-la bien et vous gagnerez des heures chaque semaine.

### 43.26.1 — Le problème historique

Dans le développement mobile classique, modifier une couleur imposait ce cycle :

```text
  modifier le code
     → recompiler          (30 à 120 secondes)
     → réinstaller         (5 à 20 secondes)
     → relancer l'appli
     → renaviguer jusqu'à l'écran concerné   (20 secondes)
     → constater le résultat
```

Deux minutes pour changer une couleur. Trente fois par jour.

### 43.26.2 — Ce que fait le hot reload

```text
  modifier le code
     → appuyer sur « r » ou enregistrer
     → moins d'une seconde
     → l'écran est mis à jour, VOUS ÊTES TOUJOURS AU MÊME ENDROIT
```

Techniquement, Flutter injecte le nouveau code dans la machine virtuelle Dart déjà en cours d'exécution, puis force la reconstruction de l'arbre de widgets. Les objets existants, eux, ne sont pas recréés.

**Conséquence essentielle : l'état est conservé.**

### 43.26.3 — Le hot restart

Le hot restart recharge aussi le code, mais **détruit l'application et la relance depuis `main()`**.

- L'état est perdu.
- Vous revenez au premier écran.
- Cela prend quelques secondes, pas quelques millisecondes.
- Mais cela n'a pas les limites du hot reload.

### 43.26.4 — Le tableau de décision

| | Hot reload (`r`) | Hot restart (`R`) | Relance complète (`q` puis `flutter run`) |
| --- | --- | --- | --- |
| Durée | < 1 s | 1 à 5 s | 30 s à 3 min |
| État conservé | **oui** | non | non |
| `main()` réexécuté | non | **oui** | oui |
| `initState()` réexécuté | non | **oui** | oui |
| Variables globales réinitialisées | non | **oui** | oui |
| Prend en compte les changements de code Dart | oui | oui | oui |
| Prend en compte un nouveau package | non | non | **oui** |
| Prend en compte un nouvel asset | parfois | oui | oui |
| Prend en compte du code natif | non | non | **oui** |

### 43.26.5 — Exemple concret

Prenons un compteur, et supposons que vous l'ayez amené à 42 en appuyant 42 fois.

```dart
// Version 1
Text(
  'Score : $_score',
  style: const TextStyle(fontSize: 20, color: Colors.black),
)
```

Vous changez la couleur :

```dart
// Version 2
Text(
  'Score : $_score',
  style: const TextStyle(fontSize: 20, color: Colors.deepPurple),
)
```

| Action | Résultat à l'écran |
| --- | --- |
| Hot reload (`r`) | « Score : 42 » en violet. **Le 42 est conservé.** |
| Hot restart (`R`) | « Score : 0 » en violet. Vous devez tout refaire. |

Sur une application à cinq écrans où le bug se produit au quatrième, cette différence est énorme.

### 43.26.6 — Le hot reload dans VS Code

Si `editor.formatOnSave` est activé, chaque `Ctrl` + `S` déclenche un hot reload. Vous ne taperez donc presque jamais `r` : vous enregistrez, et l'écran se met à jour.

C'est la boucle de travail idéale :

```text
  écrire  →  Ctrl+S  →  regarder  →  écrire  →  Ctrl+S  →  regarder ...
```

### 43.26.7 — Quand le hot reload semble ne rien faire

Avant de crier au bug, vérifiez dans l'ordre :

1. Le terminal affiche-t-il une **erreur de compilation** ? Le hot reload est alors refusé.
2. Avez-vous modifié `main()` ou `initState()` ? Il faut `R`.
3. Avez-vous ajouté un package ? Il faut arrêter et relancer.
4. Le fichier est-il bien enregistré ?

La section suivante détaille tous les cas.

---

## 43.27 — Ce que le hot reload ne peut pas faire

Il faut connaître ces limites, sinon vous perdrez du temps à chercher un bug qui n'existe pas.

### 43.27.1 — Les modifications de `main()`

```dart
void main() {
  runApp(const MonApplication());
}
```

Si vous modifiez le contenu de `main()`, le hot reload ne le verra pas : `main()` n'est **pas réexécuté**.

**Solution :** hot restart (`R`).

### 43.27.2 — Les modifications d'`initState()`

Même raison. `initState()` (chapitre 45) s'exécute une seule fois, à la création de l'état. Le hot reload ne recrée pas l'état.

```dart
@override
void initState() {
  super.initState();
  _score = 100; // Modifier cette valeur ne se verra pas au hot reload.
}
```

**Solution :** hot restart (`R`).

### 43.27.3 — Les initialiseurs de variables globales et de champs statiques

```dart
final tableauDesNiveaux = [
  'Forêt',
  'Caverne',
  'Château',
];
```

Ces valeurs sont calculées une seule fois. Modifier la liste ne se verra pas au hot reload.

**Contournement recommandé par la documentation :** utiliser `const`, ou passer par un accesseur.

```dart
// Solution 1 : const
const tableauDesNiveaux = ['Forêt', 'Caverne', 'Château'];

// Solution 2 : un getter, recalculé à chaque appel
List<String> get tableauDesNiveaux => ['Forêt', 'Caverne', 'Château'];
```

### 43.27.4 — Passer d'une `enum` à une classe (ou l'inverse)

```dart
// Avant
enum TypeArme { epee, arc, baton }

// Après — hot restart obligatoire
class TypeArme {
  const TypeArme(this.nom, this.degats);
  final String nom;
  final int degats;
}
```

Changer la **nature** d'un type est trop profond pour un rechargement à chaud.

**Solution :** hot restart (`R`).

### 43.27.5 — Modifier les paramètres génériques d'une classe

```dart
// Avant
class Inventaire<T> {
  final List<T> objets = [];
}

// Après — hot restart obligatoire
class Inventaire<T, C> {
  final List<T> objets = [];
  final List<C> categories = [];
}
```

Ajouter ou retirer un paramètre de type modifie la signature de la classe.

**Solution :** hot restart (`R`).

### 43.27.6 — Le code natif

Toute modification dans `android/`, `ios/`, `windows/`, `macos/`, `linux/` — Kotlin, Java, Swift, Objective-C, C++, Gradle, `Info.plist`, `AndroidManifest.xml` — exige une **recompilation complète**.

**Solution :** arrêter (`q`) et relancer `flutter run`.

### 43.27.7 — L'ajout d'un package

```bash
flutter pub add http
```

Le nouveau package n'est pas dans l'application en cours d'exécution.

**Solution :** arrêter et relancer.

### 43.27.8 — L'ajout d'un asset

Ajouter une image dans `assets/` et la déclarer dans `pubspec.yaml` demande au minimum un hot restart, souvent une relance complète.

**Solution :** hot restart, et si cela ne suffit pas, relance complète.

### 43.27.9 — Une erreur de compilation

Si votre code ne compile pas, le hot reload est **rejeté** :

```text
lib/main.dart:42:7: Error: Expected ';' after this.
      Text('Bonjour')
      ^
Hot reload was rejected.
```

Ce n'est pas une panne : c'est l'analyseur qui vous protège. Corrigez l'erreur et enregistrez à nouveau.

### 43.27.10 — Le `CupertinoTabView`

Cas particulier documenté : les modifications apportées au `builder` d'un `CupertinoTabView` ne sont pas prises par le hot reload.

**Solution :** hot restart.

### 43.27.11 — L'application a été tuée par le système

Si vous laissez l'application en arrière-plan trop longtemps sur un téléphone, Android peut la fermer pour libérer de la mémoire. La connexion est alors rompue.

**Solution :** relancer.

### 43.27.12 — Tableau de synthèse

| Modification | Action nécessaire |
| --- | --- |
| Un texte, une couleur, une taille | hot reload |
| Ajouter, retirer, déplacer un widget | hot reload |
| Le corps d'une méthode `build()` | hot reload |
| Ajouter une méthode ou un champ | hot reload |
| Corriger une erreur de compilation | hot reload (après correction) |
| Le contenu de `main()` | **hot restart** |
| Le contenu d'`initState()` | **hot restart** |
| Une variable globale ou un champ statique | **hot restart** |
| `enum` ↔ classe | **hot restart** |
| Paramètres génériques d'une classe | **hot restart** |
| `CupertinoTabView.builder` | **hot restart** |
| Un nouveau package | **relance complète** |
| Un nouvel asset | hot restart, sinon relance |
| Du code natif ou Gradle | **relance complète** |
| `pubspec.yaml` (dépendances) | **relance complète** |

> **Règle mnémotechnique :** *si vous ne comprenez pas pourquoi votre modification ne s'affiche pas, faites `R`. Si `R` ne suffit pas, faites `q` puis `flutter run`. Dans 100 % des cas, l'un des deux règle le problème.*

---

## 43.28 — `flutter analyze` et `dart format`

Deux outils, deux rôles différents, que l'on confond souvent.

```text
  ┌────────────────────────────────────────────────────────────┐
  │  flutter analyze  →  « Ton code est-il CORRECT ? »         │
  │                      erreurs, avertissements, mauvaises     │
  │                      pratiques                              │
  ├────────────────────────────────────────────────────────────┤
  │  dart format      →  « Ton code est-il BIEN PRÉSENTÉ ? »   │
  │                      indentation, retours à la ligne,       │
  │                      espaces                                │
  └────────────────────────────────────────────────────────────┘
```

### 43.28.1 — `flutter analyze`

```bash
flutter analyze
```

**Résultat quand tout va bien :**

```text
Analyzing mon_appli...
No issues found! (ran in 3,1s)
```

**Résultat avec des problèmes :**

```text
Analyzing mon_appli...

   info • Unused import: 'dart:math' • lib/main.dart:2:8 • unused_import
warning • The value of the local variable 'score' isn't used •
          lib/main.dart:18:9 • unused_local_variable
  error • The method 'Tex' isn't defined for the class '_EcranAccueilState' •
          lib/main.dart:34:14 • undefined_method

3 issues found. (ran in 3,8s)
```

Lecture d'une ligne :

```text
  error • Le message expliqué • fichier:ligne:colonne • nom_de_la_regle
   │
   └── niveau de gravité
```

| Niveau | Signification | Bloque la compilation ? |
| --- | --- | --- |
| `error` | Le code est invalide. | **oui** |
| `warning` | Le code compile mais c'est probablement un bug. | non |
| `info` | Suggestion de style ou de bonne pratique. | non |

> **Discipline recommandée :** visez systématiquement `No issues found!`. Un projet qui accumule 200 `info` est un projet où personne ne lit plus les messages — et le jour où un vrai `warning` apparaît, il passe inaperçu.

Analyser un seul dossier :

```bash
flutter analyze lib/models
```

### 43.28.2 — `dart format`

```bash
dart format .
```

**Résultat :**

```text
Formatted lib/main.dart
Formatted lib/models/joueur.dart
Formatted 2 files (2 changed) in 0.31 seconds.
```

Le point signifie « tout le projet, récursivement ».

Le formateur applique le style officiel de Dart. Il n'y a **rien à configurer** et c'est volontaire : le style officiel évite les débats stériles en équipe.

Ce qu'il fait :

- indente de 2 espaces ;
- coupe les lignes trop longues ;
- normalise les espaces autour des opérateurs ;
- aligne les paramètres nommés ;
- retire les espaces en fin de ligne.

Avant :

```dart
class Joueur{
final String nom;int   vies;
Joueur(this.nom,{this.vies=3});
}
```

Après `dart format .` :

```dart
class Joueur {
  final String nom;
  int vies;
  Joueur(this.nom, {this.vies = 3});
}
```

### 43.28.3 — La virgule finale, un outil de mise en forme

Un détail propre à Flutter, mais très important au quotidien.

Sans virgule après le dernier argument, le formateur essaie de tout mettre sur une ligne :

```dart
Column(children: [Text('Score'), Text('1200'), Text('Niveau 3')]);
```

Avec une virgule finale, il met chaque élément sur sa propre ligne :

```dart
Column(
  children: [
    Text('Score'),
    Text('1200'),
    Text('Niveau 3'),
  ],
);
```

La seconde forme est infiniment plus lisible et se modifie sans risque.

> **Règle à adopter dès maintenant :** **mettez toujours une virgule après le dernier argument** d'un widget. C'est la convention universelle en Flutter, et c'est ce qui rend l'arbre de widgets lisible.

### 43.28.4 — Vérifier sans modifier

Utile dans un script d'intégration continue :

```bash
dart format --output=none --set-exit-if-changed .
```

Cette commande ne modifie rien, mais renvoie un code d'erreur si un fichier n'était pas formaté.

### 43.28.5 — `dart fix`

Un troisième outil, moins connu, qui applique automatiquement une partie des corrections suggérées par l'analyseur.

Voir ce qui serait corrigé :

```bash
dart fix --dry-run
```

**Résultat :**

```text
Computing fixes in mon_appli (dry run)...

lib/main.dart
  prefer_const_constructors • 3 fixes
  unnecessary_new • 1 fix

4 proposed fixes in 1 file.
```

Appliquer :

```bash
dart fix --apply
```

**Résultat :**

```text
Computing fixes in mon_appli...
Applying fixes...

lib/main.dart
  prefer_const_constructors • 3 fixes
  unnecessary_new • 1 fix

4 fixes made in 1 file.
```

C'est un excellent moyen d'apprendre : lancez `dart fix --dry-run`, lisez les noms des règles, et allez voir ce qu'elles signifient.

### 43.28.6 — Le rituel avant chaque commit

```bash
dart format .
flutter analyze
flutter test
```

Trois commandes, dix secondes, et vous ne livrerez jamais de code mal formé ou cassé.

---

## 43.29 — `analysis_options.yaml`

Ce fichier configure l'analyseur : quelles règles activer, quels fichiers ignorer, quelle sévérité appliquer.

### 43.29.1 — Le fichier généré

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # avoid_print: false
    # prefer_single_quotes: true
```

Trois lignes utiles, le reste est commenté.

### 43.29.2 — La ligne `include`

```yaml
include: package:flutter_lints/flutter.yaml
```

Cette ligne importe le jeu de règles officiel recommandé par l'équipe Flutter. Il contient plusieurs dizaines de règles, ce qui vous évite de les écrire une par une.

Il existe trois jeux officiels, du plus souple au plus strict :

| Jeu | Package | Contenu |
| --- | --- | --- |
| `core` | `lints` | les règles indiscutables |
| `recommended` | `lints` | `core` + les bonnes pratiques Dart |
| `flutter` | `flutter_lints` | `recommended` + les règles propres à Flutter |

`flutter_lints` est le bon choix, et c'est le défaut.

### 43.29.3 — Activer et désactiver des règles

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # Interdire print() dans le code livré : on préfère un vrai logger.
    avoid_print: true

    # Imposer les guillemets simples, comme dans tout le SDK Flutter.
    prefer_single_quotes: true

    # Exiger un type de retour explicite sur chaque fonction.
    always_declare_return_types: true

    # Exiger const partout où c'est possible : gain de performance réel.
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true

    # Ne pas exiger de commentaire de documentation partout (trop strict ici).
    public_member_api_docs: false
```

La liste complète des règles est publiée sur `https://dart.dev/tools/linter-rules`.

### 43.29.4 — Changer la sévérité d'une règle

Vous pouvez transformer un `info` en `error` pour vous forcer à le corriger :

```yaml
analyzer:
  errors:
    unused_import: error
    unused_local_variable: error
    todo: ignore
    prefer_const_constructors: warning
```

| Valeur | Effet |
| --- | --- |
| `error` | bloque la compilation |
| `warning` | avertit |
| `info` | simple information |
| `ignore` | n'affiche plus rien |

### 43.29.5 — Exclure des fichiers

Utile pour ne pas analyser du code généré automatiquement.

```yaml
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "build/**"
    - "lib/generated/**"
```

### 43.29.6 — Ignorer une règle localement

Parfois, une règle a tort dans un cas précis. On peut la neutraliser sans toucher au fichier de configuration.

Pour une seule ligne :

```dart
// ignore: avoid_print
print('Trace de débogage temporaire');
```

Pour tout un fichier, à mettre en première ligne :

```dart
// ignore_for_file: avoid_print
```

> **Attention :** utilisez ces échappatoires avec parcimonie. Une règle qu'on ignore partout est une règle qu'il vaut mieux désactiver franchement dans `analysis_options.yaml`, avec un commentaire expliquant pourquoi.

### 43.29.7 — Une configuration complète pour cette formation

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    unused_import: error
    unused_local_variable: error
  exclude:
    - "**/*.g.dart"
    - "build/**"

linter:
  rules:
    prefer_single_quotes: true
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
    always_declare_return_types: true
    avoid_print: false
    sort_child_properties_last: true
    use_key_in_widget_constructors: true
```

Deux commentaires sur ces choix :

- `avoid_print: false` parce que nous utiliserons `print` pour apprendre. Dans un vrai projet livré, passez-le à `true`.
- `sort_child_properties_last` impose que `child:` soit le dernier paramètre d'un widget, ce qui rend l'arbre bien plus lisible.

---

## 43.30 — Lire le `main.dart` généré, ligne par ligne

C'est le moment le plus important du chapitre. Ne le survolez pas.

Voici le fichier tel que `flutter create` l'écrit, débarrassé de ses longs commentaires anglais.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text('You have pushed the button this many times:'),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

Une soixantaine de lignes. Disséquons-les.

### 43.30.1 — L'import

```dart
import 'package:flutter/material.dart';
```

Cette unique ligne donne accès à **tout** le catalogue Material : `Text`, `Column`, `Scaffold`, `AppBar`, `Colors`, `ThemeData`, `FloatingActionButton`, `Icons`... plus de mille classes.

Il existe deux autres bibliothèques d'interface :

| Import | Style |
| --- | --- |
| `package:flutter/material.dart` | style Material (Google) — **le nôtre** |
| `package:flutter/cupertino.dart` | style iOS (Apple) |
| `package:flutter/widgets.dart` | les briques de base, sans style |

`material.dart` inclut `widgets.dart`. Un seul import suffit donc.

### 43.30.2 — `main()`

```dart
void main() {
  runApp(const MyApp());
}
```

Vous reconnaissez le `main()` du chapitre 01. La nouveauté est `runApp()`.

`runApp()` fait trois choses :

```text
  1. Attacher le widget donné à la racine de l'arbre de widgets.
  2. Le mesurer et le dessiner sur toute la surface disponible.
  3. Démarrer la boucle d'évènements, qui ne s'arrêtera jamais.
```

C'est pour cela que `main()` ne se termine pas au sens habituel : la fonction rend la main, mais le processus continue de tourner.

Le mot-clé `const` devant `MyApp()` est une optimisation : ce widget ne dépend d'aucune donnée variable, Flutter peut donc le créer une fois pour toutes.

### 43.30.3 — La classe `MyApp`

```dart
class MyApp extends StatelessWidget {
```

Vous reconnaissez l'héritage du chapitre 10. `MyApp` **est un** `StatelessWidget` : un widget sans état interne, dont l'apparence dépend uniquement de ses paramètres.

```dart
  const MyApp({super.key});
```

Un constructeur `const` avec un paramètre nommé `key`, transmis à la classe parente avec `super.key` (syntaxe vue au chapitre 09). La `key` sert à Flutter pour identifier un widget lorsqu'il réorganise l'arbre ; nous y reviendrons au chapitre 45. Pour l'instant : recopiez-la, elle ne coûte rien.

```dart
  @override
  Widget build(BuildContext context) {
```

**La méthode la plus importante de tout Flutter.**

- `@override` : nous redéfinissons une méthode du parent (chapitre 10).
- Elle renvoie un `Widget` : la description de ce qu'il faut afficher.
- Elle reçoit un `BuildContext` : la position de ce widget dans l'arbre. C'est grâce à lui que `Theme.of(context)` sait quel thème s'applique.

**Vous n'appelez jamais `build()` vous-même.** C'est Flutter qui l'appelle, à chaque fois qu'il a besoin de redessiner. Peut-être soixante fois par seconde.

### 43.30.4 — `MaterialApp`

```dart
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
```

`MaterialApp` est le widget racine d'une application de style Material. Il apporte, sans que vous ayez rien à faire :

| Apport | Détail |
| --- | --- |
| Le thème | couleurs, polices, formes, appliqués partout |
| La navigation | le `Navigator` du chapitre 50 |
| La localisation | textes système, sens de lecture |
| Les directions | gauche-à-droite ou droite-à-gauche |
| Le titre | affiché dans le sélecteur d'applications Android |

`ColorScheme.fromSeed(seedColor: Colors.deepPurple)` est une fonctionnalité de Material 3 : à partir d'**une seule couleur**, Flutter dérive une palette complète et harmonieuse (couleur principale, secondaire, de surface, d'erreur, et leurs variantes sur fond clair et sombre). Changez `Colors.deepPurple` en `Colors.teal` et faites un hot reload : toute l'application change de teinte.

`home:` désigne le premier écran affiché.

### 43.30.5 — `MyHomePage`, un `StatefulWidget`

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
```

Ici, le compteur change au fil du temps : il faut donc un widget **avec état**.

Un `StatefulWidget` s'écrit toujours en **deux classes** :

```text
  ┌────────────────────────────────────────────────────────────┐
  │  MyHomePage extends StatefulWidget                         │
  │    • immuable, tous ses champs sont final                  │
  │    • contient la CONFIGURATION (ici : title)               │
  │    • peut être détruit et recréé très souvent              │
  │                                                            │
  │              createState()                                 │
  │                    │                                       │
  │                    ▼                                       │
  │  _MyHomePageState extends State<MyHomePage>                │
  │    • MUTABLE, survit aux reconstructions                   │
  │    • contient les DONNÉES qui changent (ici : _counter)    │
  │    • contient build()                                      │
  └────────────────────────────────────────────────────────────┘
```

`required this.title` : un paramètre nommé obligatoire, syntaxe du chapitre 07 et du chapitre 12.

Le tiret bas de `_MyHomePageState` en fait une classe **privée au fichier**, comme au chapitre 10.

### 43.30.6 — L'état et `setState`

```dart
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
```

`_counter` est la donnée qui change.

`setState()` est la méthode centrale du chapitre 45. Elle fait deux choses, dans cet ordre :

```text
  1. Elle exécute la fonction que vous lui passez  →  _counter++
  2. Elle prévient Flutter : « mon état a changé, rappelle build() »
```

**Erreur classique du débutant :**

```dart
void _incrementCounter() {
  _counter++;          // La valeur change en mémoire...
}                      // ... mais l'écran ne bouge pas !
```

Sans `setState`, Flutter ne sait pas qu'il doit redessiner. L'écran reste figé, et l'on cherche pendant vingt minutes une erreur qui n'existe pas.

### 43.30.7 — Le `build()` de l'écran

```dart
    return Scaffold(
      appBar: AppBar(...),
      body: Center(...),
      floatingActionButton: FloatingActionButton(...),
    );
```

`Scaffold` (« échafaudage ») fournit la structure standard d'un écran Material :

```text
  ┌──────────────────────────────────┐
  │  appBar                          │  ← barre du haut
  ├──────────────────────────────────┤
  │                                  │
  │                                  │
  │            body                  │  ← contenu principal
  │                                  │
  │                                  │
  │                          ( + )   │  ← floatingActionButton
  ├──────────────────────────────────┤
  │  bottomNavigationBar             │  ← facultatif
  └──────────────────────────────────┘
```

Ligne par ligne :

```dart
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
```

`Theme.of(context)` remonte l'arbre depuis ce `context` jusqu'à trouver le thème fourni par `MaterialApp`. C'est le mécanisme de l'`InheritedWidget`, expliqué au chapitre 52.

`widget.title` : dans la classe d'état, `widget` désigne l'instance de `MyHomePage`. C'est ainsi que l'état accède à la configuration.

```dart
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text('You have pushed the button this many times:'),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
```

- `Center` centre son unique enfant.
- `Column` empile ses enfants verticalement.
- `mainAxisAlignment: MainAxisAlignment.center` centre verticalement dans la colonne.
- `children` est une `List<Widget>` : la collection du chapitre 06.
- `'$_counter'` est une interpolation de chaîne, chapitre 02.
- Le premier `Text` est `const` : il ne changera jamais, Flutter peut le réutiliser tel quel.
- Le second ne peut pas être `const` : il dépend de `_counter`.

```dart
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
```

`onPressed: _incrementCounter` : on passe **la fonction elle-même**, sans parenthèses. C'est la fonction de première classe du chapitre 07. Avec des parenthèses, on l'appellerait immédiatement, ce qui est une erreur classique.

### 43.30.8 — Le schéma d'ensemble

```text
  main()
    └── runApp()
          └── MyApp                     (StatelessWidget)
                └── MaterialApp         (thème + navigation)
                      └── MyHomePage    (StatefulWidget)
                            └── _MyHomePageState
                                  └── Scaffold
                                        ├── AppBar
                                        │     └── Text(title)
                                        ├── Center
                                        │     └── Column
                                        │           ├── Text('You have...')
                                        │           └── Text('$_counter')
                                        └── FloatingActionButton
                                              └── Icon(Icons.add)
```

Voilà l'**arbre de widgets**. C'est le sujet entier du chapitre 44.

### 43.30.9 — Ce que vous devez retenir de cette section

| Élément | Rôle |
| --- | --- |
| `import 'package:flutter/material.dart'` | ouvre le catalogue Material |
| `runApp()` | démarre l'application |
| `StatelessWidget` | widget sans état |
| `StatefulWidget` + `State` | widget avec état |
| `build()` | décrit ce qu'il faut afficher |
| `BuildContext` | la position dans l'arbre |
| `MaterialApp` | racine, thème, navigation |
| `Scaffold` | structure d'un écran |
| `setState()` | « redessine, quelque chose a changé » |
| `const` | optimisation, à mettre partout où c'est possible |

---

## 43.31 — Supprimer le fichier généré et écrire son propre « Bonjour »

Le meilleur moyen de vérifier que vous avez compris est de tout effacer et de repartir de zéro.

### 43.31.1 — Version minimale absolue

Ouvrez `lib/main.dart`, **effacez tout**, et écrivez :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Bonjour'),
        ),
      ),
    ),
  );
}
```

Sept lignes utiles. Enregistrez, puis faites un **hot restart** (`R`) : vous avez modifié `main()`, le hot reload ne suffirait pas.

**Résultat à l'écran :**

```text
  ┌──────────────────────────────────┐
  │                                  │
  │                                  │
  │            Bonjour               │
  │                                  │
  │                                  │
  └──────────────────────────────────┘
```

Remarques :

- pas d'`AppBar` : l'écran est nu, le texte est donc centré sur toute la hauteur ;
- l'ensemble est `const` : rien ne change jamais dans cette application ;
- `MaterialApp` reste indispensable pour disposer d'une `Directionality` et d'un thème.

> **Essayez :** retirez `MaterialApp` et laissez seulement `Scaffold`. Vous obtiendrez une erreur `No Directionality widget found`. C'est `MaterialApp` qui fournit le sens de lecture du texte.

### 43.31.2 — Version structurée

Écrire tout dans `runApp()` ne passe pas l'échelle. Voici la forme que nous emploierons dans tout le cours.

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
      title: 'Mon premier Flutter',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
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
      appBar: AppBar(
        title: const Text('Carnet de bord'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.videogame_asset, size: 96),
            SizedBox(height: 16),
            Text(
              'Bonjour, aventurier.',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Text('Votre premier écran Flutter fonctionne.'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
  ┌──────────────────────────────────┐
  │  Carnet de bord                  │  ← AppBar teintée
  ├──────────────────────────────────┤
  │                                  │
  │            [ icône ]             │  ← Icon, 96 pixels
  │                                  │
  │      Bonjour, aventurier.        │  ← 24 px, gras
  │                                  │
  │  Votre premier écran Flutter     │
  │  fonctionne.                     │
  │                                  │
  └──────────────────────────────────┘
```

Points nouveaux à noter :

| Élément | Explication |
| --- | --- |
| `debugShowCheckedModeBanner: false` | supprime la bannière rouge « DEBUG » du coin |
| `SizedBox(height: 16)` | une boîte vide qui sert d'espacement |
| `Icon(Icons.videogame_asset)` | une icône Material, disponible sans asset |
| `const` sur tout le `body` | rien n'y dépend d'une variable |
| Noms de classes en français | c'est un choix pédagogique assumé dans cette formation |

### 43.31.3 — Version avec état, pour faire le lien

Pour bien sentir ce qui vient au chapitre 45 :

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
      title: 'Compteur de score',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
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

  void _ajouterPoints() {
    setState(() {
      _score += 50;
    });
  }

  void _reinitialiser() {
    setState(() {
      _score = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Score de la partie'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Score actuel'),
            const SizedBox(height: 8),
            Text(
              '$_score',
              style: const TextStyle(
                fontSize: 64,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _reinitialiser,
              child: const Text('Réinitialiser'),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _ajouterPoints,
        tooltip: 'Ajouter 50 points',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**Résultat à l'écran, après trois appuis sur le bouton flottant :**

```text
  ┌──────────────────────────────────┐
  │  Score de la partie              │
  ├──────────────────────────────────┤
  │                                  │
  │          Score actuel            │
  │                                  │
  │              150                 │  ← 64 px, gras
  │                                  │
  │       [ Réinitialiser ]          │
  │                                  │
  │                          ( + )   │
  └──────────────────────────────────┘
```

Faites maintenant l'expérience décisive :

1. Appuyez trois fois sur `+`. Le score affiche 150.
2. Changez `Colors.orange` en `Colors.green` dans le code.
3. Enregistrez, ou tapez `r` dans le terminal.
4. **Le thème devient vert, et le score reste à 150.**
5. Tapez maintenant `R`. **Le score retombe à 0.**

Vous venez de comprendre, par l'expérience, toute la section 43.26.

### 43.31.4 — Nettoyer aussi le test

Si vous avez supprimé `MyHomePage`, le fichier `test/widget_test.dart` généré ne compile plus : il fait référence à des classes qui n'existent plus. `flutter test` échouera.

Remplacez son contenu par un test minimal :

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:mon_appli/main.dart';

void main() {
  testWidgets('L\'écran affiche le titre', (WidgetTester tester) async {
    await tester.pumpWidget(const MonApplication());
    expect(find.text('Score de la partie'), findsOneWidget);
  });
}
```

**Résultat :**

```text
00:02 +1: All tests passed!
```

Adaptez `mon_appli` au nom de votre projet.

---

## 43.32 — Les erreurs d'installation les plus fréquentes et leur solution

Cette section est une trousse de secours. Revenez-y chaque fois que quelque chose bloque.

### 43.32.1 — « flutter n'est pas reconnu » / « command not found »

**Symptôme, Windows :**

```text
flutter : Le terme «flutter» n'est pas reconnu comme nom d'applet de commande,
fonction, fichier de script ou programme exécutable.
```

**Symptôme, macOS et Linux :**

```text
bash: flutter: command not found
```

**Diagnostic dans l'ordre :**

1. Avez-vous **rouvert** le terminal après la modification du `PATH` ? (cause n° 1)
2. Le chemin ajouté finit-il bien par `/bin` ?
3. Le dossier existe-t-il vraiment ? Vérifiez-le dans l'explorateur.
4. Affichez votre `PATH` (43.9.2) et cherchez la ligne Flutter.

**Vérification rapide :** appelez le programme par son chemin complet.

```bash
~/develop/flutter/bin/flutter --version
```

Si cela fonctionne, le problème est **uniquement** le `PATH`.

### 43.32.2 — Le chemin contient un espace ou un accent

**Symptôme :** des erreurs incohérentes à la compilation, souvent dans Gradle, mentionnant un chemin tronqué.

**Cause :** un dossier comme `C:\Users\Jean Dupont\Mes Documents\Flutter Développement`.

**Correction :** déplacez le SDK **et** vos projets dans un chemin simple :

```text
  C:\Users\jean\develop\flutter
  C:\Users\jean\projets\mon_appli
```

C'est la règle la plus ennuyeuse de l'écosystème, mais elle ne se discute pas.

### 43.32.3 — « Android sdkmanager not found »

**Symptôme dans `flutter doctor` :**

```text
[✗] Android toolchain - develop for Android devices
    ✗ Android sdkmanager not found. Update to the latest Android SDK and ensure
      that the cmdline-tools are installed to resolve this.
```

**Correction :** Android Studio > SDK Manager > onglet **SDK Tools** > cocher **Android SDK Command-line Tools (latest)** > Apply.

### 43.32.4 — « Android license status unknown »

```bash
flutter doctor --android-licenses
```

Répondez `y` à chaque question.

Si la commande elle-même échoue avec une erreur Java, c'est que le JDK utilisé n'est pas le bon. Indiquez celui d'Android Studio :

```bash
flutter config --jdk-dir "/opt/android-studio/jbr"
```

Le chemin varie : `C:\Program Files\Android\Android Studio\jbr` sous Windows, `/Applications/Android Studio.app/Contents/jbr/Contents/Home` sous macOS.

### 43.32.5 — « Unable to locate Android SDK »

**Correction :** installez Android Studio (43.13), puis, si nécessaire :

```bash
flutter config --android-sdk /chemin/vers/Android/Sdk
```

### 43.32.6 — Gradle reste bloqué très longtemps

**Symptôme :**

```text
Running Gradle task 'assembleDebug'...
```

et rien pendant dix minutes.

**Causes possibles :**

| Cause | Correction |
| --- | --- |
| Premier lancement | c'est normal : Gradle télécharge sa chaîne. Patientez. |
| Connexion lente ou proxy | configurez `HTTP_PROXY` et `HTTPS_PROXY`. |
| Antivirus | excluez le dossier du projet et `~/.gradle`. |
| Cache Gradle corrompu | supprimez `~/.gradle/caches` et relancez. |
| Mémoire insuffisante | augmentez `org.gradle.jvmargs` dans `android/gradle.properties`. |

Nettoyage complet :

```bash
flutter clean
cd android
./gradlew clean
cd ..
flutter pub get
flutter run
```

### 43.32.7 — L'émulateur ne démarre pas

| Symptôme | Correction |
| --- | --- |
| Écran noir figé | Device Manager > menu > **Cold Boot Now** |
| « HAXM is not installed » | activez la virtualisation dans le BIOS/UEFI |
| Très lent | vérifiez que l'image correspond au processeur (43.14.1) |
| « emulator: ERROR: x86 emulation currently requires hardware acceleration » | activez WHPX ou KVM (43.14.3) |
| Toujours rien | utilisez `flutter run -d chrome` en attendant |

### 43.32.8 — « No devices found »

Rappel de la marche à suivre :

```bash
flutter devices
flutter emulators
flutter emulators --launch Pixel_8_API_36
```

Et en dernier recours, la cible toujours disponible :

```bash
flutter run -d chrome
```

### 43.32.9 — « Waiting for another flutter command to release the startup lock »

**Symptôme :**

```text
Waiting for another flutter command to release the startup lock...
```

**Cause :** un processus `flutter` précédent ne s'est pas terminé proprement.

**Correction :** fermez tous les terminaux, puis supprimez le verrou :

```bash
rm ~/develop/flutter/bin/cache/lockfile
```

Sous Windows, supprimez le fichier `lockfile` dans `...\flutter\bin\cache\`. Si le fichier refuse d'être supprimé, tuez le processus `dart.exe` dans le Gestionnaire des tâches.

### 43.32.10 — « Target of URI doesn't exist: 'package:...' »

**Symptôme dans l'éditeur :** un import souligné en rouge alors que le package est bien dans `pubspec.yaml`.

**Correction :**

```bash
flutter pub get
```

Puis, dans VS Code, palette de commandes > `Developer: Reload Window`.

### 43.32.11 — Version de Dart incompatible

**Symptôme :**

```text
The current Dart SDK version is 3.8.0.

Because mon_appli requires SDK version ^3.10.0, version solving failed.
```

**Correction :** mettez Flutter à jour.

```bash
flutter upgrade
flutter pub get
```

Ou, si vous ne pouvez pas mettre à jour, abaissez la contrainte dans `pubspec.yaml` — à condition que le code n'utilise pas de nouveautés du langage.

### 43.32.12 — Deux SDK Dart en conflit

**Symptôme :** `dart --version` et `flutter --version` annoncent deux versions de Dart différentes.

**Diagnostic :**

```bash
which dart
which flutter
```

**Correction :** placez `.../flutter/bin` **avant** l'ancienne installation Dart dans le `PATH` (43.9.3).

### 43.32.13 — Erreur réseau vers pub.dev

**Symptôme :**

```text
Got socket error trying to find package http at https://pub.dev.
```

**Corrections :**

| Cause | Correction |
| --- | --- |
| Proxy d'entreprise | définir `HTTP_PROXY` et `HTTPS_PROXY` |
| Pare-feu | autoriser `dart` et `flutter` |
| Serveur momentanément indisponible | réessayer plus tard |
| Miroir nécessaire | définir `PUB_HOSTED_URL` et `FLUTTER_STORAGE_BASE_URL` |

Exemple de configuration de proxy :

```bash
export HTTP_PROXY=http://proxy.entreprise.fr:3128
export HTTPS_PROXY=http://proxy.entreprise.fr:3128
```

### 43.32.14 — L'espace disque manque

Flutter, Android Studio, le SDK Android, les émulateurs et les caches Gradle occupent facilement 20 Go.

Pour récupérer de la place :

```bash
flutter clean                 # dans chaque projet
flutter pub cache clean       # le cache global des packages
```

Vous pouvez aussi supprimer les images système d'émulateurs que vous n'utilisez plus, depuis le SDK Manager.

### 43.32.15 — La méthode générale, en cinq étapes

Quand rien ne marche et que vous ne savez plus par où commencer :

```text
  1.  flutter doctor -v          → lire chaque ligne, sans en sauter une
  2.  flutter clean              → repartir d'une compilation propre
  3.  flutter pub get            → réinstaller les dépendances
  4.  fermer et rouvrir l'éditeur et le terminal
  5.  copier le message d'erreur EXACT dans un moteur de recherche
```

L'étape 5 n'est pas un aveu de faiblesse : c'est la compétence professionnelle la plus utile du métier. Le message d'erreur de Flutter est presque toujours précis, et quelqu'un l'a déjà rencontré avant vous.

---

## 43.33 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `flutter: command not found` | Le dossier `flutter/bin` n'est pas dans le `PATH`, ou le terminal n'a pas été rouvert. | Ajouter `.../flutter/bin` au `PATH`, fermer **tous** les terminaux, en rouvrir un. |
| `PATH` pointant sur `.../flutter` au lieu de `.../flutter/bin` | Confusion entre le dossier du SDK et le dossier des exécutables. | Le `PATH` doit finir par `/bin` ; le réglage `dart.flutterSdkPath` de VS Code, non. |
| `Unable to locate Android SDK` | Android Studio n'est pas installé, ou son SDK est ailleurs. | Installer Android Studio, ou `flutter config --android-sdk <chemin>`. |
| `Android license status unknown` | Les licences du SDK Android n'ont jamais été acceptées. | `flutter doctor --android-licenses`, répondre `y` à chaque question. |
| `cmdline-tools component is missing` | La case n'est pas cochée par défaut dans le SDK Manager. | SDK Manager > SDK Tools > **Android SDK Command-line Tools** > Apply. |
| `"MonAppli" is not a valid Dart package name` | Majuscules, tirets ou accents dans le nom du projet. | N'utiliser que `[a-z0-9_]`, par exemple `mon_appli`. |
| Application refusée par la boutique | L'`--org` est resté à `com.example`. | Créer le projet avec `flutter create --org com.monstudio ...`. |
| L'écran ne change pas quand on appuie sur un bouton | La valeur a été modifiée sans `setState()`. | Envelopper la modification dans `setState(() { ... });`. |
| Une modification de `main()` reste invisible après `r` | Le hot reload ne réexécute pas `main()`. | Faire un hot restart (`R`). |
| Un nouveau package est introuvable à l'exécution | Le hot reload et le hot restart ne rechargent pas les dépendances. | Arrêter (`q`) et relancer `flutter run`. |
| `Target of URI doesn't exist: 'package:http/http.dart'` | Les dépendances n'ont pas été téléchargées. | `flutter pub get`, puis recharger la fenêtre de l'éditeur. |
| `found a tab character that violates indentation` | Une tabulation s'est glissée dans `pubspec.yaml`. | Remplacer par des espaces : deux par niveau, jamais de tabulation. |
| `No Directionality widget found` | `Scaffold` ou `Text` utilisé sans `MaterialApp` au-dessus. | Envelopper l'arbre dans `MaterialApp`. |
| `Waiting for another flutter command to release the startup lock` | Un processus `flutter` précédent est resté bloqué. | Fermer les terminaux, supprimer `flutter/bin/cache/lockfile`. |
| Deux versions de Dart annoncées | Un Dart installé séparément est trouvé avant celui de Flutter. | Placer `.../flutter/bin` en tête du `PATH` ; vérifier avec `which dart`. |
| Compilation extrêmement lente sur Windows | L'antivirus analyse chaque fichier généré. | Exclure le dossier du SDK et le dossier des projets de l'antivirus. |
| Émulateur figé sur un écran noir | Instantané de démarrage corrompu, ou accélération matérielle absente. | Device Manager > **Cold Boot Now** ; activer WHPX, HAXM ou KVM. |
| Le téléphone n'apparaît pas dans `flutter devices` | Câble de charge seule, ou débogage USB non autorisé. | Changer de câble, activer le débogage USB, valider le dialogue sur le téléphone. |
| `adb devices` affiche `unauthorized` | Le dialogue d'autorisation RSA n'a pas été validé. | Rebrancher, cocher « Toujours autoriser », toucher **AUTORISER**. |
| Erreurs Gradle incompréhensibles | Cache de compilation corrompu. | `flutter clean`, puis `flutter pub get`, puis relancer. |
| `version solving failed` sur le SDK | Le projet exige un Dart plus récent que celui installé. | `flutter upgrade` puis `flutter pub get`. |
| L'application paraît lente et saccadée | Elle tourne en mode debug. | Mesurer avec `flutter run --profile` ou `--release`. |

---

## 43.34 — Résumé du chapitre

| Commande | Rôle |
| --- | --- |
| `flutter --version` | Affiche la version de Flutter, du moteur, de Dart et de DevTools. |
| `flutter doctor` | Diagnostique l'installation et signale ce qui manque. |
| `flutter doctor -v` | Même chose, avec les chemins réels et tous les détails. |
| `flutter doctor --android-licenses` | Accepte les licences du SDK Android. |
| `flutter channel` | Affiche le canal courant et la liste des canaux. |
| `flutter channel stable` | Bascule sur le canal indiqué (à suivre d'un `flutter upgrade`). |
| `flutter upgrade` | Met à jour le SDK Flutter sur le canal courant. |
| `flutter downgrade` | Revient à la version précédente du canal courant. |
| `flutter config` | Affiche les réglages globaux de l'outil. |
| `flutter config --enable-web` | Active une cible de compilation (`web`, `windows`, `macos`, `linux`). |
| `flutter config --android-sdk <chemin>` | Indique manuellement l'emplacement du SDK Android. |
| `flutter emulators` | Liste les émulateurs disponibles. |
| `flutter emulators --launch <id>` | Démarre un émulateur. |
| `flutter emulators --create --name <nom>` | Crée un nouvel émulateur. |
| `flutter devices` | Liste les appareils et cibles disponibles avec leur identifiant. |
| `flutter create mon_appli` | Crée un projet complet dans un nouveau dossier. |
| `flutter create .` | Régénère les fichiers manquants dans le projet courant. |
| `flutter create --org com.studio mon_appli` | Fixe l'identifiant d'organisation (Android et iOS). |
| `flutter create --platforms=android,web mon_appli` | Ne génère que les plateformes demandées. |
| `flutter create --empty mon_appli` | Crée un projet avec un `main.dart` minimal. |
| `flutter create --template=package ma_lib` | Crée une bibliothèque au lieu d'une application. |
| `flutter pub get` | Télécharge les dépendances déclarées dans `pubspec.yaml`. |
| `flutter pub add http` | Ajoute un package et met `pubspec.yaml` à jour. |
| `flutter pub add --dev build_runner` | Ajoute un package de développement. |
| `flutter pub remove http` | Retire un package. |
| `flutter pub outdated` | Liste les packages dont une version plus récente existe. |
| `flutter pub upgrade` | Met à jour dans les limites des contraintes déclarées. |
| `flutter run` | Compile, installe et lance l'application en mode debug. |
| `flutter run -d chrome` | Lance sur une cible précise. |
| `flutter run --release` | Lance en mode release, sans hot reload, à pleine vitesse. |
| `flutter run --profile` | Lance en mode profil, pour mesurer les performances. |
| `r` (dans `flutter run`) | Hot reload : recharge le code et conserve l'état. |
| `R` (dans `flutter run`) | Hot restart : recharge le code et réinitialise l'état. |
| `q` (dans `flutter run`) | Arrête l'application et rend le terminal. |
| `flutter analyze` | Analyse le code : erreurs, avertissements, suggestions. |
| `dart format .` | Formate tous les fichiers Dart du projet. |
| `dart fix --dry-run` | Montre les corrections automatiques possibles. |
| `dart fix --apply` | Applique ces corrections. |
| `flutter test` | Exécute les tests du dossier `test/`. |
| `flutter clean` | Supprime `build/` et `.dart_tool/`. |
| `flutter build apk` | Produit un `.apk` Android en mode release. |
| `flutter build web` | Produit un site statique dans `build/web`. |
| `adb devices` | Liste les appareils Android vus par le pont de débogage. |

| Fichier ou dossier | À retenir |
| --- | --- |
| `lib/` | Votre code Dart. Vous y passez 99 % de votre temps. |
| `lib/main.dart` | Le point d'entrée : `main()` puis `runApp()`. |
| `test/` | Les tests automatiques. |
| `pubspec.yaml` | Nom, version, dépendances, assets, polices. |
| `pubspec.lock` | Versions exactes retenues. Versionné pour une application. |
| `analysis_options.yaml` | Règles de l'analyseur, importées de `flutter_lints`. |
| `android/`, `ios/`, `web/` | Les projets natifs. Permissions, icônes, titre de l'onglet. |
| `build/`, `.dart_tool/` | Générés, jetables, jamais versionnés. |

---

## 43.35 — Exercices

### Exercice 1 — Faire le point sur son installation (facile)

Sans rien installer de nouveau, produisez un état des lieux de votre machine. Donnez les commandes qui permettent de connaître :

1. la version de Flutter et le canal actif ;
2. la version de Dart utilisée par Flutter ;
3. l'emplacement exact de l'exécutable `flutter` ;
4. la liste complète des appareils disponibles ;
5. la liste des cibles activées dans la configuration globale.

Rédigez ensuite, en trois phrases, ce que `flutter doctor` vous reproche et si cela vous concerne.

### Exercice 2 — Diagnostiquer un `PATH` cassé (facile)

Un camarade vous écrit :

```text
$ flutter --version
bash: flutter: command not found

$ ls ~/develop/flutter/bin
dart  flutter  flutter.bat  internal  cache
```

Le SDK est donc bien décompressé. Donnez :

1. la commande qui affiche le `PATH` de façon lisible ;
2. la ligne exacte à ajouter dans `~/.bashrc` ;
3. les deux commandes à taper ensuite ;
4. l'erreur qu'il aurait commise s'il avait ajouté `~/develop/flutter` au lieu de `~/develop/flutter/bin`.

### Exercice 3 — Créer un projet correctement paramétré (facile)

Créez un projet nommé `carnet_de_bord`, destiné à Android et au Web uniquement, dont l'identifiant d'organisation est `fr.monstudio` et dont la description est `Le carnet de bord de mes parties`.

Donnez la commande unique qui fait tout cela, puis les deux commandes qui permettent de vérifier le résultat dans le `pubspec.yaml` et dans la liste des dossiers créés.

### Exercice 4 — Nommer les dossiers (facile)

Pour chacun des éléments suivants, dites en une phrase à quoi il sert, s'il faut le versionner dans Git, et si vous le modifierez souvent :

```text
  lib/          test/          build/         .dart_tool/
  android/      web/           pubspec.yaml   pubspec.lock
  analysis_options.yaml        .metadata
```

Présentez votre réponse sous forme de tableau à quatre colonnes.

### Exercice 5 — Ajouter des dépendances (intermédiaire)

Dans le projet `carnet_de_bord`, vous voulez :

- le package `http` pour les appels réseau ;
- le package `intl` pour formater les dates ;
- le package `build_runner`, mais uniquement pendant le développement.

Donnez les commandes, puis l'extrait de `pubspec.yaml` obtenu. Expliquez pourquoi `build_runner` ne doit pas figurer dans `dependencies`.

### Exercice 6 — Hot reload ou hot restart ? (intermédiaire)

Pour chacune de ces dix modifications, indiquez l'action minimale nécessaire : hot reload (`r`), hot restart (`R`), ou relance complète.

1. Changer `Colors.blue` en `Colors.red` dans un `AppBar`.
2. Ajouter un `SizedBox(height: 16)` dans une `Column`.
3. Changer la valeur initiale d'un champ dans `initState()`.
4. Ajouter `flutter pub add shared_preferences`.
5. Transformer `enum Difficulte { facile, difficile }` en une classe.
6. Ajouter une méthode privée dans la classe d'état.
7. Modifier le contenu de `main()`.
8. Modifier `android/app/src/main/AndroidManifest.xml`.
9. Corriger une virgule manquante qui empêchait la compilation.
10. Modifier la valeur d'une liste globale déclarée avec `final` en dehors de toute classe.

### Exercice 7 — Écrire un « Bonjour » complet (intermédiaire)

Écrivez un `lib/main.dart` complet et copiable qui affiche :

- une `AppBar` intitulée `Mes aventures` ;
- au centre de l'écran, une icône `Icons.shield` de 80 pixels ;
- en dessous, le texte `Bienvenue, aventurier` en 22 pixels et en gras ;
- en dessous, le texte `Aucune partie en cours` ;
- un espacement de 12 pixels entre chaque élément ;
- un thème dérivé de `Colors.indigo` ;
- pas de bannière « DEBUG ».

L'application doit être entièrement `StatelessWidget` et utiliser `const` partout où c'est possible.

### Exercice 8 — Compter les vies (intermédiaire)

Écrivez un `lib/main.dart` complet qui affiche un compteur de vies :

- une `AppBar` intitulée `Vies` ;
- au centre, le nombre de vies en très gros (56 pixels) ;
- deux boutons `ElevatedButton` côte à côte : `Perdre une vie` et `Soigner` ;
- la valeur de départ est 3 ;
- le nombre de vies ne doit jamais descendre en dessous de 0 ni dépasser 5 ;
- quand les vies tombent à 0, le texte affiché devient `Partie perdue` en rouge à la place du nombre.

### Exercice 9 — Configurer l'analyseur (intermédiaire)

Écrivez un `analysis_options.yaml` qui :

- part du jeu de règles `flutter_lints` ;
- transforme `unused_import` et `unused_local_variable` en erreurs bloquantes ;
- ignore complètement les commentaires `TODO` ;
- exclut de l'analyse tous les fichiers finissant par `.g.dart` ainsi que le dossier `build/` ;
- impose les guillemets simples ;
- impose `const` partout où c'est possible ;
- impose que le paramètre `child` soit écrit en dernier.

Donnez ensuite les deux commandes qui vérifient le résultat et corrigent automatiquement ce qui peut l'être.

### Exercice 10 — La trousse de secours (difficile)

Un projet récupéré sur Git refuse de démarrer. Voici ce qui s'affiche :

```text
$ flutter run
Error: Target of URI doesn't exist: 'package:http/http.dart'.
lib/services/api.dart:1:8

$ flutter doctor
[✓] Flutter (Channel stable, 3.47.0)
[!] Android toolchain - develop for Android devices
    ✗ Android license status unknown.
[✓] Chrome - develop for the web
[✓] VS Code (version 1.104.0)
[!] Connected device
    ✗ No devices available
```

Rédigez une procédure de réparation en quatre étapes numérotées, avec la commande exacte de chaque étape, et une phrase expliquant ce que chaque commande corrige. Précisez également quelle est la cible sur laquelle vous pourrez lancer l'application le plus vite, et pourquoi.

---

## 43.36 — Corrections des exercices

### Correction 1

```bash
flutter --version
flutter doctor -v
which flutter          # Windows : Get-Command flutter
flutter devices
flutter config
```

**Explication :** `flutter --version` donne en une seule sortie la version du framework, le canal, la révision, la version de Dart et celle de DevTools : c'est la réponse aux points 1 et 2. `flutter doctor -v` complète en affichant le **chemin** du SDK réellement utilisé, ce qui lève tout doute en cas d'installations multiples. `which flutter` (ou `Get-Command flutter` en PowerShell) répond au point 3 en montrant quel exécutable le `PATH` sélectionne. `flutter devices` liste les cibles utilisables avec leur identifiant, celui que l'on passera à `flutter run -d`. Enfin, `flutter config` affiche les cibles activées globalement (`enable-web`, `enable-linux-desktop`...). Pour la rédaction finale, rappelez-vous la règle de lecture de 43.10.2 : un `[✗]` sur Xcode sous Windows ou Linux est normal et ne vous concerne pas ; seule la ligne `[✓] Flutter` est obligatoire, plus au moins une cible utilisable.

### Correction 2

```bash
# 1. Afficher le PATH de façon lisible
echo $PATH | tr ':' '\n'

# 2. Ligne à ajouter dans ~/.bashrc
export PATH="$HOME/develop/flutter/bin:$PATH"

# 3. Recharger puis vérifier
source ~/.bashrc
flutter --version
```

**Explication :** `echo $PATH | tr ':' '\n'` remplace les deux-points par des retours à la ligne, ce qui rend la liste lisible et permet de vérifier d'un coup d'œil si l'entrée Flutter s'y trouve et à quelle position. La ligne `export` doit être écrite dans un fichier lu à chaque ouverture de session, sinon elle serait perdue à la fermeture du terminal ; sous bash, ce fichier est `~/.bashrc`. On préfixe (`:$PATH` à la fin) plutôt qu'on suffixe, afin que le Dart livré avec Flutter soit trouvé **avant** un éventuel Dart installé séparément (43.9.3). Enfin, si le camarade avait ajouté `~/develop/flutter`, le système chercherait un exécutable nommé `flutter` **dans** ce dossier ; or il n'y a là que des sous-dossiers. Les exécutables sont dans `bin`. Le `PATH` doit donc toujours se terminer par `/bin` — à ne pas confondre avec le réglage `dart.flutterSdkPath` de VS Code, qui attend au contraire le dossier **sans** `/bin`.

### Correction 3

```bash
flutter create \
  --org fr.monstudio \
  --platforms=android,web \
  --description "Le carnet de bord de mes parties" \
  carnet_de_bord

cd carnet_de_bord
cat pubspec.yaml
ls -la
```

**Explication :** une seule invocation suffit ; toutes les options se cumulent. `--org fr.monstudio` détermine le nom de paquet Android (`fr.monstudio.carnet_de_bord`) et l'identifiant de lot iOS ; c'est l'option à ne surtout pas oublier, car la corriger après coup impose de modifier une dizaine de fichiers natifs, et parce que la valeur par défaut `com.example` est refusée par les boutiques. `--platforms=android,web` évite de générer les six dossiers natifs : le projet est plus léger et plus lisible. Les valeurs sont séparées par des virgules **sans espace**. `--description` remplit le champ correspondant du `pubspec.yaml`. Le nom `carnet_de_bord` respecte les règles de nommage : uniquement des minuscules, des chiffres et des tirets bas. À la vérification, `ls -la` doit montrer `android/`, `web/`, `lib/`, `test/`, mais **pas** `ios/`, `windows/`, `macos/` ni `linux/`. Si vous en aviez besoin plus tard, `flutter create --platforms=ios .` les ajouterait sans rien détruire.

### Correction 4

| Élément | À quoi il sert | Versionné ? | Modifié souvent ? |
| --- | --- | --- | --- |
| `lib/` | Contient tout votre code Dart, à commencer par `main.dart`. | Oui | **En permanence** |
| `test/` | Contient les tests automatiques exécutés par `flutter test`. | Oui | Régulièrement |
| `build/` | Sortie de compilation : APK, bundles, fichiers intermédiaires. | **Non** | Jamais à la main |
| `.dart_tool/` | Cache de l'outillage Dart, résolution des packages. | **Non** | Jamais à la main |
| `android/` | Le projet Android natif : manifeste, Gradle, icônes, signature. | Oui | Rarement |
| `web/` | Les fichiers du site : `index.html`, `manifest.json`, favicon. | Oui | Rarement |
| `pubspec.yaml` | Carte d'identité : nom, version, dépendances, assets, polices. | Oui | Régulièrement |
| `pubspec.lock` | Versions exactes retenues lors de la dernière résolution. | Oui (application) | Jamais à la main |
| `analysis_options.yaml` | Règles de l'analyseur et exclusions. | Oui | Une ou deux fois par projet |
| `.metadata` | Suivi interne de la version de l'outil Flutter. | Oui | Jamais |

**Explication :** trois familles se dégagent. La première regroupe ce que vous écrivez (`lib/`, `test/`, `pubspec.yaml`, `analysis_options.yaml`) : c'est versionné et vivant. La deuxième regroupe la configuration native (`android/`, `web/`) : versionnée, mais que l'on ouvre trois fois par an, pour une permission, une icône ou un titre d'onglet. La troisième regroupe ce qui est **généré** (`build/`, `.dart_tool/`) : jetable, exclu de Git par le `.gitignore` fourni, et régénérable à tout moment par `flutter pub get` et `flutter run`. Le cas de `pubspec.lock` est le seul qui demande de la nuance : on le versionne pour une **application**, afin que toute l'équipe compile avec exactement les mêmes versions ; on l'exclut pour une **bibliothèque publiée**, dont les utilisateurs doivent pouvoir résoudre eux-mêmes les versions.

### Correction 5

```bash
flutter pub add http
flutter pub add intl
flutter pub add --dev build_runner
```

Ou en une seule ligne pour les deux dépendances normales :

```bash
flutter pub add http intl
flutter pub add --dev build_runner
```

Extrait de `pubspec.yaml` obtenu (les numéros dépendent du jour) :

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  http: ^1.5.0
  intl: ^0.20.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0
  build_runner: ^2.4.15
```

**Explication :** on passe par `flutter pub add` plutôt que par une édition manuelle pour trois raisons : la commande écrit automatiquement la dernière version compatible avec votre SDK, elle utilise le bon format de contrainte avec le caret, et elle enchaîne le `pub get`. Si la version proposée était incompatible, elle échouerait immédiatement avec un message clair, au lieu de vous laisser découvrir le problème à la compilation. `build_runner` va dans `dev_dependencies` parce qu'il ne sert **qu'à générer du code sur votre machine** : il n'est jamais exécuté par l'application chez l'utilisateur. Le placer dans `dependencies` alourdirait inutilement l'analyse des dépendances et enverrait un mauvais signal sur la nature du package. La même logique s'applique à `flutter_test` et `flutter_lints`, déjà présents. Notez au passage que les numéros exacts changent : c'est pourquoi cette formation donne toujours la **commande** et jamais un numéro de version figé.

### Correction 6

| # | Modification | Action minimale |
| --- | --- | --- |
| 1 | `Colors.blue` → `Colors.red` dans un `AppBar` | hot reload `r` |
| 2 | Ajouter un `SizedBox(height: 16)` | hot reload `r` |
| 3 | Changer une valeur dans `initState()` | **hot restart `R`** |
| 4 | `flutter pub add shared_preferences` | **relance complète** |
| 5 | `enum` transformée en classe | **hot restart `R`** |
| 6 | Ajouter une méthode privée dans la classe d'état | hot reload `r` |
| 7 | Modifier le contenu de `main()` | **hot restart `R`** |
| 8 | Modifier `AndroidManifest.xml` | **relance complète** |
| 9 | Corriger une virgule manquante | hot reload `r` |
| 10 | Modifier une liste globale `final` | **hot restart `R`** |

**Explication :** la logique tient en une phrase. Le hot reload **réexécute uniquement les méthodes `build()`** ; tout ce qui n'est appelé qu'une seule fois lui échappe. C'est le cas de `main()` (n° 7), d'`initState()` (n° 3) et des initialiseurs de variables globales ou de champs statiques (n° 10) : ces valeurs sont déjà calculées en mémoire, et le hot reload ne recrée pas les objets existants. Changer la **nature** d'un type (n° 5, `enum` vers classe) ou ses paramètres génériques modifie sa signature en profondeur, ce que le rechargement à chaud ne sait pas propager. Les cas 1, 2, 6 et 9 vivent tous à l'intérieur d'un `build()` ou n'ont d'effet qu'au prochain appel de `build()` : le hot reload suffit. Enfin, un package (n° 4) et du code natif ou Gradle (n° 8) exigent une **recompilation** : ni `r` ni `R` ne les prendront en compte, il faut `q` puis `flutter run`. Retenez la règle de secours de 43.27.12 : en cas de doute, `R` ; si `R` ne suffit pas, relance complète.

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
      title: 'Mes aventures',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
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
      appBar: AppBar(
        title: const Text('Mes aventures'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.shield, size: 80),
            SizedBox(height: 12),
            Text(
              'Bienvenue, aventurier',
              style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Text('Aucune partie en cours'),
          ],
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
  ┌──────────────────────────────────┐
  │  Mes aventures                   │
  ├──────────────────────────────────┤
  │                                  │
  │           [ bouclier ]           │
  │                                  │
  │      Bienvenue, aventurier       │
  │                                  │
  │      Aucune partie en cours      │
  │                                  │
  └──────────────────────────────────┘
```

**Explication :** l'application se découpe en deux widgets, comme dans tout projet Flutter sérieux : `MonApplication` porte la configuration globale (titre, thème, écran d'accueil), `EcranAccueil` porte le contenu. `debugShowCheckedModeBanner: false` retire la bannière rouge du coin supérieur droit ; elle n'apparaît de toute façon qu'en mode debug. Le `Center` centre son unique enfant horizontalement et verticalement, et `mainAxisAlignment: MainAxisAlignment.center` centre les enfants **à l'intérieur** de la colonne : les deux sont nécessaires pour un centrage complet, car une `Column` occupe par défaut toute la hauteur disponible. Les `SizedBox(height: 12)` sont la façon idiomatique de créer un espacement en Flutter : une boîte vide de hauteur imposée. Enfin, le `const` est placé une seule fois, sur `Center` : comme aucun de ses descendants ne dépend d'une variable, tout le sous-arbre devient constant. C'est exactement ce que la règle de lint `prefer_const_constructors` recherche, et le gain est réel : Flutter réutilise l'objet au lieu de le reconstruire à chaque image.

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
      title: 'Vies',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
      ),
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
  static const int viesMax = 5;
  int _vies = 3;

  void _perdreUneVie() {
    setState(() {
      if (_vies > 0) {
        _vies--;
      }
    });
  }

  void _soigner() {
    setState(() {
      if (_vies < viesMax) {
        _vies++;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Vies'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (_vies == 0)
              const Text(
                'Partie perdue',
                style: TextStyle(
                  fontSize: 40,
                  fontWeight: FontWeight.bold,
                  color: Colors.red,
                ),
              )
            else
              Text(
                '$_vies',
                style: const TextStyle(
                  fontSize: 56,
                  fontWeight: FontWeight.bold,
                ),
              ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _perdreUneVie,
                  child: const Text('Perdre une vie'),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: _soigner,
                  child: const Text('Soigner'),
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

**Résultat à l'écran, après deux appuis sur « Perdre une vie » :**

```text
  ┌────────────────────────────────────────┐
  │  Vies                                  │
  ├────────────────────────────────────────┤
  │                                        │
  │                  1                     │
  │                                        │
  │  [ Perdre une vie ]   [ Soigner ]      │
  │                                        │
  └────────────────────────────────────────┘
```

**Explication :** l'écran change au fil du temps, il faut donc un `StatefulWidget`, écrit en deux classes. Toute modification de `_vies` est enveloppée dans `setState()` : sans cela, la variable changerait bien en mémoire mais l'écran resterait figé, ce qui est l'erreur la plus fréquente du débutant. Les bornes sont vérifiées **à l'intérieur** de `setState`, ce qui garantit qu'un appui inutile ne provoque pas d'incohérence ; on aurait pu aussi utiliser `_vies = (_vies - 1).clamp(0, viesMax);`. La constante `viesMax` est déclarée `static const` : elle appartient à la classe et non à l'instance, et sa valeur est connue à la compilation. Le `if / else` employé dans la liste `children` est la syntaxe des **collection-if** vue au chapitre 06 : elle permet de choisir un widget ou un autre directement dans la liste, sans construire la liste à part. Enfin, `Row` dispose les deux boutons horizontalement, et le `SizedBox(width: 16)` les sépare ; notez que l'on utilise `width` et non `height`, puisque l'axe principal d'une `Row` est horizontal.

### Correction 9

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    unused_import: error
    unused_local_variable: error
    todo: ignore
  exclude:
    - "**/*.g.dart"
    - "build/**"

linter:
  rules:
    prefer_single_quotes: true
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
    sort_child_properties_last: true
```

Vérification et correction automatique :

```bash
flutter analyze
dart fix --dry-run
dart fix --apply
dart format .
```

**Explication :** la ligne `include` importe le jeu de règles officiel de Flutter, ce qui évite d'écrire des dizaines de règles à la main ; tout ce que l'on ajoute ensuite vient **compléter ou surcharger** ce socle. La section `analyzer.errors` ne crée pas de règles, elle change la **sévérité** de règles existantes : un import inutilisé, qui n'est normalement qu'une information, devient bloquant, ce qui force à faire le ménage ; à l'inverse, `todo: ignore` fait taire les rappels sur les commentaires `TODO`, souvent nombreux pendant l'apprentissage. La section `analyzer.exclude` retire de l'analyse les fichiers générés en `.g.dart` — que vous n'écrivez pas et sur lesquels vous ne pouvez rien corriger — ainsi que le dossier `build/`. La section `linter.rules` active les règles de style demandées : guillemets simples comme dans tout le SDK Flutter, `const` partout où c'est possible (deux règles, une pour les constructeurs et une pour les littéraux de collections), et `child` en dernier paramètre, ce qui rend l'arbre de widgets bien plus lisible. Enfin, `dart fix --dry-run` liste ce qui peut être corrigé sans rien toucher : lisez cette liste, elle vous apprendra le nom des règles. `dart fix --apply` applique les corrections, et `dart format .` remet la mise en forme au propre.

### Correction 10

```bash
# 1. Réinstaller les dépendances déclarées dans pubspec.yaml
flutter pub get

# 2. Accepter les licences du SDK Android
flutter doctor --android-licenses

# 3. Obtenir un appareil : soit démarrer un émulateur...
flutter emulators
flutter emulators --launch Pixel_8_API_36

# 4. ... soit, plus rapide, lancer directement dans le navigateur
flutter run -d chrome
```

**Explication :** l'ordre compte. **Étape 1.** L'erreur `Target of URI doesn't exist: 'package:http/http.dart'` ne signifie pas que le package est absent du `pubspec.yaml` : elle signifie qu'il n'a pas été **téléchargé** sur cette machine. C'est normal après un clone Git, puisque les packages ne sont jamais versionnés. `flutter pub get` lit `pubspec.yaml` et `pubspec.lock` et récupère exactement les versions attendues. Si l'éditeur continue de souligner l'import après coup, un `Developer: Reload Window` dans VS Code règle l'affichage. **Étape 2.** Le `[!]` sur la chaîne Android vient uniquement des licences, jamais acceptées sur cette machine ; sans elles, aucune compilation Android ne démarrera. La commande affiche plusieurs textes juridiques : il faut répondre `y` à chacun, jusqu'au message `All SDK package licenses accepted.` **Étapes 3 et 4.** `No devices available` n'est pas une panne : c'est simplement qu'aucun émulateur ne tourne et qu'aucun téléphone n'est branché. La cible la plus rapide est **Chrome**, pour trois raisons : elle est déjà en vert dans le rapport de `flutter doctor`, elle ne dépend ni du SDK Android ni de ses licences, et elle démarre en quelques secondes là où un émulateur Android demande une à deux minutes et plusieurs gigaoctets de mémoire. On peut donc valider immédiatement que le projet compile avec `flutter run -d chrome`, puis régler tranquillement la partie Android. C'est exactement la méthode générale de 43.32.15 : lire le rapport, réparer ce qui est réparable, et se donner le plus vite possible une boucle de test qui fonctionne.

---

## Et maintenant ?

Votre machine est prête. Vous savez créer un projet, le lancer sur un émulateur, un téléphone ou dans un navigateur, modifier son interface en une seconde grâce au hot reload, et diagnostiquer une installation qui résiste. Vous savez aussi lire le `main.dart` généré, et vous l'avez remplacé par le vôtre.

Mais vous avez manipulé des mots dont nous n'avons donné que l'intuition : `StatelessWidget`, `MaterialApp`, `Scaffold`, `build()`, `BuildContext`, `const`, l'arbre de widgets. Il est temps de les comprendre vraiment.

Le chapitre suivant est le cœur de Flutter. Vous y découvrirez ce qu'est exactement un widget, pourquoi **tout** est un widget — jusqu'à l'espacement et à l'alignement —, comment Flutter construit et reconstruit son arbre soixante fois par seconde sans que cela coûte cher, ce que `BuildContext` désigne réellement, et pourquoi le mot-clé `const` devant un widget n'est pas un détail de style mais une véritable optimisation.

Rendez-vous au chapitre 44 : [44-PARTIE-1B—LES-WIDGETS-ET-LARBRE-DE-WIDGETS.md](./44-PARTIE-1B—LES-WIDGETS-ET-LARBRE-DE-WIDGETS.md)
