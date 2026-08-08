# PARTIE 1A — DART
# CHAPITRE 16 — ORGANISATION D'UN PROJET DART

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 6 h
> **Pré-requis :** chapitre 15 — Programmation asynchrone (`Future`, `async`, `await`)
> **Ce que vous saurez faire à la fin :** installer Dart sur votre machine, créer un vrai projet avec `dart create`, le découper en plusieurs fichiers, y ajouter des packages venus de pub.dev, l'analyser, le formater, le tester et le compiler en exécutable.

---

## 16.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer pourquoi DartPad ne suffit plus à partir d'un certain point ;
- dire ce que contient le Dart SDK ;
- installer Dart sur Windows, macOS ou Linux et vérifier l'installation ;
- installer VS Code et l'extension Dart ;
- créer un projet avec `dart create mon_jeu` ;
- choisir entre les modèles `console` et `package` ;
- nommer chaque dossier d'un projet Dart et dire à quoi il sert ;
- lancer un programme avec `dart run` ;
- analyser votre code avec `dart analyze` ;
- formater votre code avec `dart format` ;
- produire un exécutable avec `dart compile exe` ;
- lire et écrire un fichier `pubspec.yaml`, champ par champ ;
- comprendre le rôle de `pubspec.lock` ;
- expliquer ce qu'est un package et naviguer sur pub.dev ;
- ajouter une dépendance avec `dart pub add` ;
- distinguer `dependencies` et `dev_dependencies` ;
- lire une contrainte de version et comprendre le caret `^` ;
- écrire les trois formes d'`import` (`dart:`, `package:`, chemin relatif) ;
- filtrer un import avec `show` et `hide` ;
- préfixer un import avec `as` ;
- découper une classe par fichier ;
- expliquer `part` / `part of` et pourquoi on l'évite ;
- créer un fichier « barrel » avec `export` ;
- comprendre la portée d'un membre privé `_` à l'échelle d'un fichier ;
- configurer `analysis_options.yaml` et activer des lints ;
- écrire un premier test avec `package:test` et le lancer avec `dart test` ;
- écrire un `.gitignore` correct pour un projet Dart ;
- organiser un projet propre de bout en bout.

---

## 16.1 — Pourquoi quitter DartPad

Depuis le chapitre 1, vous travaillez dans DartPad. C'était le bon choix : aucune installation, aucun réglage, un bouton « Run » et le résultat s'affiche. Pour apprendre la syntaxe, c'est idéal.

Mais vous avez atteint la limite de l'outil. Voici précisément ce que DartPad ne sait pas faire.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  Ce que DartPad ne permet pas                                  │
  ├────────────────────────────────────────────────────────────────┤
  │  1. Plusieurs fichiers                                         │
  │     Tout votre code tient dans une seule zone de texte.        │
  │     Un jeu réel a 20, 50, 300 fichiers.                        │
  │                                                                │
  │  2. Installer des packages                                     │
  │     Vous ne pouvez pas ajouter une bibliothèque écrite par     │
  │     quelqu'un d'autre (lecture de fichiers, HTTP, tests...).   │
  │                                                                │
  │  3. Lire et écrire des fichiers                                │
  │     Sauvegarder une partie ? Charger un niveau ? Impossible.   │
  │                                                                │
  │  4. Lancer des tests automatiques                              │
  │     Aucun moyen de vérifier 200 comportements en une commande. │
  │                                                                │
  │  5. Produire un programme distribuable                         │
  │     Vous ne pouvez pas donner votre jeu à un ami.              │
  │                                                                │
  │  6. Garder votre travail                                       │
  │     Fermez l'onglet et tout disparaît. Pas de sauvegarde,      │
  │     pas d'historique, pas de Git.                              │
  └────────────────────────────────────────────────────────────────┘
```

Prenons le fil rouge. Vous avez écrit une classe `Joueur`, une classe `Ennemi`, une classe `Arme`, une classe `Inventaire`, une classe `Niveau`, plus une dizaine de fonctions utilitaires. Dans DartPad, cela donne un fichier de 900 lignes dans lequel vous passez votre temps à faire défiler. Chercher `class Arme` devient une corvée. Modifier une classe sans casser les autres devient risqué.

Sur une vraie machine, le même code donne ceci :

```text
  mon_jeu/
    bin/mon_jeu.dart          le point d'entrée
    lib/joueur.dart           la classe Joueur, et rien d'autre
    lib/ennemi.dart           la classe Ennemi, et rien d'autre
    lib/arme.dart             la classe Arme, et rien d'autre
    lib/inventaire.dart       la classe Inventaire
    test/joueur_test.dart     les tests de Joueur
```

Chaque fichier tient sur un écran. Vous savez où aller. C'est cela, « organiser un projet ».

Ce chapitre est donc un chapitre d'outillage. On y écrit moins de Dart et davantage de commandes. Ne le sautez pas : tout le reste de la formation, et toute la partie Flutter, supposent que vous savez créer et manipuler un projet.

> **À retenir :** DartPad est un brouillon. Un projet Dart est un atelier.

---

## 16.2 — Le Dart SDK

Pour exécuter du Dart sur votre machine, il faut installer le **Dart SDK**.

« SDK » signifie *Software Development Kit*, en français « kit de développement logiciel ». C'est un ensemble de programmes livrés ensemble.

Que contient-il exactement ?

| Élément | Rôle |
| --- | --- |
| La VM Dart | Exécute votre code Dart directement. |
| Le compilateur | Transforme votre code en exécutable natif ou en JavaScript. |
| L'analyseur | Détecte les erreurs et les avertissements sans exécuter le code. |
| Le formateur | Réécrit votre code avec une mise en forme officielle. |
| Le gestionnaire de packages (`pub`) | Télécharge les bibliothèques dont vous dépendez. |
| Les bibliothèques de base | `dart:core`, `dart:io`, `dart:math`, `dart:convert`, `dart:async`... |

Tout cela s'utilise par une seule commande, `dart`, suivie d'un sous-mot :

```text
  dart <sous-commande> [options]

  dart create      créer un projet
  dart run         exécuter
  dart analyze     analyser
  dart format      formater
  dart test        tester
  dart compile     compiler
  dart pub         gérer les packages
  dart doc         générer la documentation
```

Une seule commande à retenir, `dart`, et un vocabulaire de sous-commandes. C'est volontairement simple.

> **Remarque :** si vous installez plus tard Flutter, le Dart SDK est **inclus** dans le Flutter SDK. Vous n'aurez donc pas à l'installer deux fois. Pour cette partie 1A, nous installons Dart seul, c'est plus léger.

---

## 16.3 — Installer Dart et vérifier avec `dart --version`

L'installation dépend de votre système. Choisissez votre section, faites-la, puis passez à la vérification commune en fin de section.

### 16.3.1 — Windows

La méthode la plus simple passe par **Chocolatey**, un gestionnaire de paquets pour Windows.

Si vous n'avez pas Chocolatey, installez-le d'abord : ouvrez **PowerShell en tant qu'administrateur** (clic droit sur le menu Démarrer, « Terminal (administrateur) ») puis suivez les instructions du site officiel de Chocolatey.

Ensuite, toujours en PowerShell administrateur :

```bash
choco install dart-sdk
```

Fermez la fenêtre, rouvrez un terminal **normal**, et passez à la vérification.

Si vous préférez éviter Chocolatey, vous pouvez aussi télécharger l'archive ZIP du SDK sur le site officiel `dart.dev/get-dart`, la décompresser dans `C:\dart`, puis ajouter `C:\dart\dart-sdk\bin` à la variable d'environnement `Path`.

> **Piège classique Windows :** après avoir modifié le `Path`, il faut **fermer et rouvrir** le terminal. Un terminal déjà ouvert garde l'ancienne valeur et vous répondra obstinément que la commande `dart` est introuvable.

### 16.3.2 — macOS

La méthode standard passe par **Homebrew**.

```bash
brew tap dart-lang/dart
brew install dart
```

La première ligne indique à Homebrew où trouver les formules Dart. La seconde installe le SDK.

Si vous n'avez pas Homebrew, installez-le d'abord depuis `brew.sh`.

### 16.3.3 — Linux (Debian, Ubuntu)

On ajoute le dépôt officiel de Google, puis on installe.

```bash
sudo apt-get update
sudo apt-get install apt-transport-https wget gnupg
wget -qO- https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo gpg --dearmor -o /usr/share/keyrings/dart.gpg
echo 'deb [signed-by=/usr/share/keyrings/dart.gpg arch=amd64] https://storage.googleapis.com/download.dartlang.org/linux/debian stable main' | sudo tee /etc/apt/sources.list.d/dart_stable.list
sudo apt-get update
sudo apt-get install dart
```

Cela paraît long, mais chaque ligne a un rôle simple :

```text
  1. mettre à jour la liste des paquets
  2. installer les outils nécessaires au téléchargement sécurisé
  3. récupérer la clé de signature de Google
  4. déclarer le dépôt Dart
  5. mettre à jour la liste avec ce nouveau dépôt
  6. installer Dart
```

Sur d'autres distributions, ou si vous voulez gérer plusieurs versions, l'archive ZIP officielle fonctionne aussi : décompressez-la et ajoutez le dossier `dart-sdk/bin` à votre `PATH`.

### 16.3.4 — Vérifier l'installation

Quelle que soit votre plateforme, ouvrez un **nouveau** terminal et tapez :

```bash
dart --version
```

**Résultat :**

```text
Dart SDK version: 3.5.0 (stable) (Tue Aug 6 2024) on "linux_x64"
```

Le numéro exact sera différent chez vous, c'est normal. Ce qui compte est que la commande réponde quelque chose de cette forme.

Si vous obtenez plutôt :

```text
dart : Le terme «dart» n'est pas reconnu comme nom d'applet de commande...
```

ou, sur macOS et Linux :

```text
bash: dart: command not found
```

alors le système ne trouve pas le programme. Trois causes, dans l'ordre de fréquence :

| Cause | Correction |
| --- | --- |
| Terminal ouvert avant l'installation | Fermez-le, rouvrez-en un neuf. |
| Dossier `bin` absent du `PATH` | Ajoutez `.../dart-sdk/bin` au `PATH`. |
| Installation interrompue | Relancez la commande d'installation et lisez les messages. |

> **Conseil :** ne passez pas à la suite tant que `dart --version` ne répond pas. Tout le chapitre en dépend.

---

## 16.4 — L'éditeur : VS Code + extension Dart

Le SDK sait exécuter du Dart. Il ne sait pas l'écrire confortablement. Pour cela, il vous faut un éditeur.

Nous utiliserons **Visual Studio Code** (VS Code) : gratuit, disponible sur les trois systèmes, et l'extension Dart y est excellente. C'est aussi l'éditeur que nous utiliserons pour Flutter.

### 16.4.1 — Installer VS Code

Rendez-vous sur `code.visualstudio.com`, téléchargez la version correspondant à votre système, installez-la. Rien de particulier à signaler.

### 16.4.2 — Installer l'extension Dart

1. Ouvrez VS Code.
2. Cliquez sur l'icône des extensions dans la barre latérale gauche (le carré composé de petits blocs), ou tapez `Ctrl+Shift+X` (`Cmd+Shift+X` sur macOS).
3. Tapez `Dart` dans la zone de recherche.
4. Choisissez l'extension nommée **Dart**, éditée par « Dart Code ».
5. Cliquez sur **Install**.

### 16.4.3 — Ce que l'extension vous apporte

Ce n'est pas un confort accessoire. C'est un changement de nature du travail.

| Fonction | Ce que cela change pour vous |
| --- | --- |
| Soulignement des erreurs | Le trait rouge apparaît pendant que vous tapez, avant toute exécution. |
| Autocomplétion | Tapez `joueur.` et la liste des méthodes disponibles s'affiche. |
| Documentation au survol | Passez la souris sur `List.where` et vous lisez sa doc. |
| Aller à la définition | `F12` sur un nom vous emmène à l'endroit où il est défini. |
| Renommer partout | `F2` renomme une classe dans tout le projet, sans oubli. |
| Corrections rapides | `Ctrl+.` propose d'ajouter l'import manquant, de créer la méthode absente... |
| Formatage automatique | Le fichier est mis en forme à chaque sauvegarde. |

### 16.4.4 — Deux réglages recommandés

Ouvrez les réglages (`Ctrl+,`) et activez :

- **Editor: Format On Save** — votre code est formaté à chaque `Ctrl+S`. Vous ne réfléchissez plus jamais à l'indentation.
- **Dart: Preview Flutter UI Guides** — inutile en Dart pur, utile plus tard en Flutter. Vous pouvez le laisser.

> **Remarque :** vous pouvez utiliser un autre éditeur (Android Studio, IntelliJ IDEA, Neovim...). Le SDK et les commandes du chapitre restent strictement identiques. Seules les captures et les raccourcis changent.

---

## 16.5 — `dart create mon_projet`

Un projet Dart n'est pas un dossier vide dans lequel on jette des fichiers. C'est une structure conventionnelle, que le SDK sait générer pour vous.

Placez-vous dans le dossier où vous rangez vos projets, puis :

```bash
dart create mon_jeu
```

**Résultat :**

```text
Creating mon_jeu using template console...

  .gitignore
  analysis_options.yaml
  CHANGELOG.md
  pubspec.yaml
  README.md
  bin/mon_jeu.dart
  lib/mon_jeu.dart
  test/mon_jeu_test.dart

Running pub get...                     2.3s
  Resolving dependencies...
  Got dependencies.

Created project mon_jeu in mon_jeu! In order to get started, run the following commands:

  cd mon_jeu
  dart run
```

En une commande, vous avez obtenu : un dossier, sept fichiers, une configuration d'analyse, un fichier Git prêt, et les dépendances déjà téléchargées.

Entrons dans le projet et lançons-le :

```bash
cd mon_jeu
dart run
```

**Résultat :**

```text
Building package executable...
Built mon_jeu:mon_jeu.
Hello world: 42!
```

Votre premier programme Dart hors DartPad fonctionne.

### 16.5.1 — Le nom du projet : une règle stricte

Le nom que vous donnez devient le nom du **package**. Dart impose des règles :

| Règle | Exemple valide | Exemple invalide |
| --- | --- | --- |
| Uniquement des minuscules | `mon_jeu` | `MonJeu` |
| Mots séparés par `_` | `moteur_de_jeu` | `moteur-de-jeu` |
| Pas d'espace | `mon_jeu` | `mon jeu` |
| Pas d'accent | `mon_jeu` | `mon_jeu_créé` |
| Ne commence pas par un chiffre | `jeu2d` | `2djeu` |
| N'est pas un mot réservé Dart | `mon_jeu` | `class` |

Ce style s'appelle le **snake_case** (« casse serpent »), parce que les mots sont reliés par des tirets bas comme les anneaux d'un serpent.

Si vous vous trompez, le SDK refuse tout de suite :

```bash
dart create MonJeu
```

**Résultat :**

```text
"MonJeu" is not a valid Dart project name.

The name should be all lowercase, with underscores to separate words,
"just_like_this".
```

> **À retenir :** un nom de package se lit `comme_ceci`, jamais `CommeCeci`, jamais `comme-ceci`.

---

## 16.6 — Les modèles de projet (`console`, `package`)

`dart create` accepte l'option `-t` (pour *template*, « modèle ») qui décide de ce que le SDK génère.

```bash
dart create -t console mon_jeu
```

Les modèles les plus utiles en partie 1A sont au nombre de deux.

### 16.6.1 — `console` (le modèle par défaut)

Un programme que l'on **exécute**. Il a un point d'entrée, donc un `main`.

```text
  mon_jeu/
    bin/mon_jeu.dart        <- contient main() : ce que l'on lance
    lib/mon_jeu.dart
    test/mon_jeu_test.dart
```

C'est le modèle à choisir pour un jeu en mode texte, un outil en ligne de commande, un exercice. **C'est celui que nous utiliserons dans tout ce chapitre.**

### 16.6.2 — `package`

Une bibliothèque que l'on **réutilise**. Elle n'a pas de `main` : on ne la lance pas, on l'importe depuis un autre projet.

```bash
dart create -t package moteur_de_jeu
```

```text
  moteur_de_jeu/
    lib/moteur_de_jeu.dart      <- le code réutilisable
    lib/src/                    <- les détails internes
    test/moteur_de_jeu_test.dart
    (pas de bin/)
```

C'est le modèle à choisir si vous écrivez un moteur de combat destiné à servir dans trois jeux différents, ou si vous comptez publier votre travail sur pub.dev.

### 16.6.3 — Les autres modèles

```bash
dart create --help
```

**Résultat (extrait) :**

```text
Available templates:
  console        A command-line application.
  package        A package containing shared Dart libraries.
  server-shelf   A server app using package:shelf.
  web            A web app that uses only core Dart libraries.
```

`server-shelf` sert à écrire un serveur HTTP, `web` une page web en Dart. Nous ne les utiliserons pas ici.

| Modèle | A un `main` ? | On le lance ? | On l'importe ? |
| --- | --- | --- | --- |
| `console` | oui | oui | rarement |
| `package` | non | non | oui |

> **Comment choisir :** posez-vous la question « est-ce que quelqu'un va **lancer** ce code, ou est-ce que quelqu'un va **l'utiliser dans son code** ? ». Lancer : `console`. Utiliser : `package`.

---

## 16.7 — Structure d'un projet Dart

Regardons l'ensemble de ce que `dart create mon_jeu` a produit.

```text
  mon_jeu/
  │
  ├── bin/
  │   └── mon_jeu.dart          point d'entrée, contient main()
  │
  ├── lib/
  │   └── mon_jeu.dart          le code réutilisable
  │
  ├── test/
  │   └── mon_jeu_test.dart     les tests automatiques
  │
  ├── .dart_tool/               fichiers techniques générés (ne pas toucher)
  │
  ├── pubspec.yaml              carte d'identité + dépendances
  ├── pubspec.lock              versions exactes réellement installées
  ├── analysis_options.yaml     règles de qualité du code
  ├── .gitignore                ce que Git doit ignorer
  ├── README.md                 présentation du projet
  └── CHANGELOG.md              historique des versions
```

Deux catégories à bien séparer dans votre tête :

```text
  ┌──────────────────────────────┬──────────────────────────────┐
  │  Vous écrivez / modifiez     │  Le SDK gère pour vous       │
  ├──────────────────────────────┼──────────────────────────────┤
  │  bin/                        │  .dart_tool/                 │
  │  lib/                        │  pubspec.lock                │
  │  test/                       │  (dossiers de build)         │
  │  pubspec.yaml                │                              │
  │  analysis_options.yaml       │                              │
  │  README.md, CHANGELOG.md     │                              │
  └──────────────────────────────┴──────────────────────────────┘
```

Cette structure n'est pas une préférence esthétique. Les outils **comptent dessus** : `dart run` cherche dans `bin/`, `dart test` cherche dans `test/`, `import 'package:...'` cherche dans `lib/`. Déplacez un fichier hors de sa case et l'outil ne le trouvera plus.

> **À retenir :** en Dart, la structure de dossiers est une convention **exécutable**, pas une décoration.

---

## 16.8 — Le dossier `bin/`

`bin` vient de *binary*, « binaire », c'est-à-dire « ce qui s'exécute ».

**Règle :** un fichier dans `bin/` contient une fonction `main` et constitue un point d'entrée du programme.

Voici ce que `dart create` y a mis :

```dart
import 'package:mon_jeu/mon_jeu.dart' as mon_jeu;

void main(List<String> arguments) {
  print('Hello world: ${mon_jeu.calculate()}!');
}
```

Remplaçons-le par quelque chose qui parle de notre fil rouge :

```dart
void main() {
  print('=== MON JEU ===');
  print('Chargement du niveau 1...');
  print('Le joueur entre dans la caverne.');
}
```

Lancez :

```bash
dart run
```

**Résultat :**

```text
=== MON JEU ===
Chargement du niveau 1...
Le joueur entre dans la caverne.
```

### 16.8.1 — Plusieurs points d'entrée

Un projet peut contenir plusieurs fichiers dans `bin/`. C'est utile pour fournir des outils annexes.

```text
  mon_jeu/
    bin/
      mon_jeu.dart              le jeu
      generer_niveaux.dart      un outil pour créer des niveaux
      exporter_scores.dart      un outil pour exporter les scores
```

On les lance en précisant le nom :

```bash
dart run
dart run :generer_niveaux
dart run :exporter_scores
```

Sans argument, `dart run` lance le fichier `bin/` qui porte le **nom du package**. C'est pour cela que `bin/mon_jeu.dart` est spécial dans un projet nommé `mon_jeu`.

### 16.8.2 — Le paramètre `arguments`

`main` peut recevoir les arguments passés en ligne de commande.

```dart
void main(List<String> arguments) {
  print('Arguments reçus : $arguments');

  if (arguments.isEmpty) {
    print('Aucun joueur indiqué. Utilisation : dart run <nom>');
    return;
  }

  final nom = arguments[0];
  print('Bienvenue, $nom !');
}
```

```bash
dart run bin/mon_jeu.dart Alex
```

**Résultat :**

```text
Arguments reçus : [Alex]
Bienvenue, Alex !
```

Et sans argument :

```bash
dart run bin/mon_jeu.dart
```

**Résultat :**

```text
Arguments reçus : []
Aucun joueur indiqué. Utilisation : dart run <nom>
```

> **Remarque :** `arguments` est toujours une `List<String>`. Même `dart run bin/mon_jeu.dart 42` vous donne `['42']`, la chaîne, pas le nombre. À vous de convertir avec `int.parse`.

---

## 16.9 — Le dossier `lib/`

`lib` vient de *library*, « bibliothèque ». C'est **le dossier le plus important** de votre projet.

**Règle :** `lib/` contient le code réutilisable — vos classes, vos fonctions, vos constantes. Pas de `main`.

Créons notre première classe dans un fichier dédié : `lib/joueur.dart`.

```dart
class Joueur {
  final String nom;
  int vies;
  int score;

  Joueur(this.nom, {this.vies = 3, this.score = 0});

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  bool get estVivant => vies > 0;

  @override
  String toString() => 'Joueur($nom, vies: $vies, score: $score)';
}
```

Et utilisons-la depuis `bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/joueur.dart';

void main() {
  final heros = Joueur('Alex');
  print(heros);

  heros.perdreUneVie();
  print('Après un coup : $heros');
  print('Encore vivant ? ${heros.estVivant}');
}
```

```bash
dart run
```

**Résultat :**

```text
Joueur(Alex, vies: 3, score: 0)
Après un coup : Joueur(Alex, vies: 2, score: 0)
Encore vivant ? true
```

Observez bien la ligne d'import :

```text
  import 'package:mon_jeu/joueur.dart';
                  ▲        ▲
                  │        └── chemin DANS lib/
                  └── nom du package (celui du pubspec.yaml)
```

Le préfixe `package:` signifie « cherche dans le dossier `lib/` du package indiqué ». On ne répète donc **jamais** `lib/` dans le chemin. `package:mon_jeu/lib/joueur.dart` est une erreur classique.

### 16.9.1 — Le sous-dossier `lib/src/`

Par convention, ce qui est dans `lib/src/` est considéré comme **interne** au package : les autres projets ne sont pas censés l'importer directement.

```text
  lib/
    mon_jeu.dart          l'API publique (ce que les autres importent)
    src/
      joueur.dart         détail d'implémentation
      ennemi.dart         détail d'implémentation
      combat.dart         détail d'implémentation
```

Cette convention n'est pas imposée par le compilateur, mais elle est vérifiée par les lints et respectée par toute la communauté. Nous y reviendrons à la section 16.29 avec le fichier « barrel ».

Pour un petit projet `console`, mettre les fichiers directement dans `lib/` est parfaitement acceptable. C'est ce que nous ferons dans ce chapitre.

---

## 16.10 — Le dossier `test/`

`test/` contient les **tests automatiques** : du code dont le seul rôle est de vérifier que votre autre code fonctionne.

**Règle :** un fichier de test se nomme `quelquechose_test.dart`. Le suffixe `_test` n'est pas décoratif : `dart test` ne prend que les fichiers qui le portent.

```text
  test/
    joueur_test.dart        teste lib/joueur.dart
    ennemi_test.dart        teste lib/ennemi.dart
    combat_test.dart        teste lib/combat.dart
```

Pourquoi écrire du code pour tester du code ? Parce que la vérification manuelle ne tient pas à l'échelle :

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Sans tests                                                  │
  │  Vous modifiez la classe Combat.                             │
  │  Vous relancez le jeu, vous jouez 5 minutes, ça marche.      │
  │  Vous n'avez pas revérifié Inventaire, ni Score, ni Boss.    │
  │  -> Le bug est découvert dans deux semaines.                 │
  ├──────────────────────────────────────────────────────────────┤
  │  Avec tests                                                  │
  │  Vous modifiez la classe Combat.                             │
  │  Vous tapez : dart test                                      │
  │  200 vérifications s'exécutent en 2 secondes.                │
  │  -> Le bug est découvert immédiatement, avec son nom.        │
  └──────────────────────────────────────────────────────────────┘
```

Nous écrirons un vrai test aux sections 16.32 et 16.33. Pour l'instant, retenez seulement que ce dossier existe et à quoi il sert.

---

## 16.11 — Le dossier `.dart_tool/`

Le point en début de nom signifie « dossier caché ». Selon votre système, il peut ne pas s'afficher par défaut.

`.dart_tool/` est un dossier **entièrement généré** par le SDK. Il contient notamment :

| Contenu | Rôle |
| --- | --- |
| `package_config.json` | La carte qui relie chaque nom de package à son emplacement sur le disque. |
| Caches de compilation | Des résultats intermédiaires, pour que la 2e exécution soit plus rapide. |
| Fichiers internes des outils | Utilisés par `dart test`, `dart analyze`, l'extension VS Code... |

Trois règles simples :

1. **Ne le modifiez jamais à la main.** Rien de ce qui s'y trouve n'est destiné à être lu ou édité par vous.
2. **Ne le mettez jamais dans Git.** Il est déjà listé dans le `.gitignore` généré.
3. **Vous pouvez le supprimer sans risque.** Un `dart pub get` le reconstruit intégralement.

Ce troisième point est d'ailleurs un remède connu quand les outils se comportent bizarrement :

```bash
rm -rf .dart_tool
dart pub get
```

Sous Windows PowerShell :

```bash
Remove-Item -Recurse -Force .dart_tool
dart pub get
```

**Résultat :**

```text
Resolving dependencies...
Got dependencies.
```

> **Analogie :** `.dart_tool/` est l'atelier de l'outil, pas le vôtre. On n'y range rien, on n'y cherche rien.

---

## 16.12 — `dart run`

`dart run` exécute votre programme.

### 16.12.1 — Les trois formes

```bash
dart run
```

Sans argument, lance `bin/<nom_du_package>.dart`. C'est la forme la plus courante.

```bash
dart run bin/mon_jeu.dart
```

Avec un chemin, lance exactement ce fichier. Fonctionne même hors de `bin/`.

```bash
dart run :generer_niveaux
```

Avec deux-points, lance `bin/generer_niveaux.dart`.

### 16.12.2 — Passer des arguments

Tout ce qui suit le nom du fichier est transmis à `main` :

```dart
void main(List<String> arguments) {
  final nom = arguments.isNotEmpty ? arguments[0] : 'Inconnu';
  final vies = arguments.length > 1 ? int.parse(arguments[1]) : 3;

  print('Joueur : $nom');
  print('Vies   : $vies');
}
```

```bash
dart run bin/mon_jeu.dart Alex 5
```

**Résultat :**

```text
Joueur : Alex
Vies   : 5
```

### 16.12.3 — Ce qui se passe réellement

```text
  dart run
     │
     ├─ 1. lit pubspec.yaml
     ├─ 2. vérifie que les dépendances sont là (.dart_tool)
     ├─ 3. compile le code en mémoire (JIT)
     └─ 4. exécute main()
```

La compilation est *juste-à-temps* (JIT, *just in time*). Conséquence : le démarrage prend une fraction de seconde, mais vous n'avez aucune étape de compilation à lancer vous-même. C'est idéal pendant le développement. Pour distribuer, on compilera vraiment (section 16.15).

> **Erreur fréquente :** lancer `dart run` depuis le mauvais dossier. La commande doit être exécutée **à la racine du projet**, celui qui contient `pubspec.yaml`. Sinon vous obtenez `Could not find a file named "pubspec.yaml"`.

---

## 16.13 — `dart analyze`

`dart analyze` lit votre code **sans l'exécuter** et signale tout ce qui est faux, douteux ou inutile.

```bash
dart analyze
```

**Résultat quand tout va bien :**

```text
Analyzing mon_jeu...
No issues found!
```

Introduisons volontairement trois problèmes dans `lib/joueur.dart` :

```dart
import 'dart:math';

class Joueur {
  final String nom;
  int vies;

  Joueur(this.nom, {this.vies = 3});

  void soigner(int points) {
    int bonus = 10;
    vies = vies + points
  }
}
```

```bash
dart analyze
```

**Résultat :**

```text
Analyzing mon_jeu...

  error • lib/joueur.dart:11:22 • Expected to find ';'. • expected_token
warning • lib/joueur.dart:10:9  • The value of the local variable 'bonus' isn't used. • unused_local_variable
   info • lib/joueur.dart:1:8   • Unused import: 'dart:math'. • unused_import

3 issues found.
```

Trois niveaux de gravité, à distinguer :

| Niveau | Signification | Le code compile-t-il ? |
| --- | --- | --- |
| `error` | Faute réelle. | Non. |
| `warning` | Code suspect, probablement une erreur de votre part. | Oui. |
| `info` | Style ou propreté. | Oui. |

Corrigeons :

```dart
class Joueur {
  final String nom;
  int vies;

  Joueur(this.nom, {this.vies = 3});

  void soigner(int points) {
    vies = vies + points;
  }
}
```

```bash
dart analyze
```

**Résultat :**

```text
Analyzing mon_jeu...
No issues found!
```

### 16.13.1 — Pourquoi l'utiliser si VS Code souligne déjà ?

Parce que VS Code n'analyse en profondeur que les fichiers ouverts, alors que `dart analyze` parcourt **tout** le projet. Un fichier que vous n'avez pas ouvert depuis trois semaines peut être cassé sans que rien ne vous alerte.

C'est aussi cette commande que l'on met dans une chaîne d'intégration continue, pour refuser automatiquement un code fautif.

> **Habitude à prendre :** lancez `dart analyze` avant chaque sauvegarde importante dans Git. Vingt secondes qui évitent des heures.

---

## 16.14 — `dart format`

`dart format` réécrit vos fichiers avec la mise en forme officielle de Dart : indentation, espaces, retours à la ligne, virgules finales.

Prenons un fichier volontairement mal présenté :

```dart
class Ennemi{
final String nom;
   int pointsDeVie;
      Ennemi( this.nom,this.pointsDeVie );
  void subirDegats(int  degats){
pointsDeVie=pointsDeVie-degats;
if(pointsDeVie<0){pointsDeVie=0;}
    }
}
```

```bash
dart format .
```

**Résultat :**

```text
Formatted lib/ennemi.dart
Formatted 1 file (1 changed) in 0.09 seconds.
```

Le fichier devient :

```dart
class Ennemi {
  final String nom;
  int pointsDeVie;
  Ennemi(this.nom, this.pointsDeVie);
  void subirDegats(int degats) {
    pointsDeVie = pointsDeVie - degats;
    if (pointsDeVie < 0) {
      pointsDeVie = 0;
    }
  }
}
```

### 16.14.1 — Les options utiles

```bash
dart format .
```

Formate tout le projet (le `.` désigne le dossier courant).

```bash
dart format lib/joueur.dart
```

Formate un seul fichier.

```bash
dart format --output=none --set-exit-if-changed .
```

Ne modifie rien, mais **échoue** si un fichier n'est pas formaté. C'est la forme utilisée en intégration continue.

**Résultat :**

```text
Changed lib/ennemi.dart
```

et le code de sortie vaut 1, ce qui fait échouer la chaîne.

### 16.14.2 — Pourquoi accepter un format imposé ?

Parce que le débat sur le style est un débat sans fin et sans valeur. Dart le tranche pour vous. Trois bénéfices immédiats :

1. Tout le code Dart du monde se ressemble, donc se lit vite.
2. Les revues de code ne parlent plus d'espaces, mais de logique.
3. Vos différences Git ne contiennent plus de bruit de mise en forme.

> **Conseil pratique :** activez « Format On Save » dans VS Code (section 16.4.4). Vous n'aurez plus jamais à taper la commande.

---

## 16.15 — `dart compile exe`

`dart run` est parfait pour développer. Pour **donner** votre jeu à quelqu'un, il faut un vrai programme, qui ne nécessite ni le SDK Dart ni vos fichiers sources.

```bash
dart compile exe bin/mon_jeu.dart -o mon_jeu
```

**Résultat :**

```text
Info: Compiling with sound null safety.
Generated: /home/alex/projets/mon_jeu/mon_jeu
```

L'option `-o` (pour *output*, « sortie ») donne le nom du fichier produit. Sous Windows, écrivez `-o mon_jeu.exe`.

Lancez-le :

```bash
./mon_jeu
```

**Résultat :**

```text
=== MON JEU ===
Chargement du niveau 1...
Le joueur entre dans la caverne.
```

### 16.15.1 — Ce que vous obtenez

| Caractéristique | Détail |
| --- | --- |
| Autonome | Aucun besoin de Dart installé chez la personne qui l'exécute. |
| Démarrage instantané | Plus de compilation au lancement. |
| Taille | Environ 5 à 10 Mo (la machine virtuelle est incluse). |
| Non portable | Un exécutable Linux ne tourne pas sous Windows. |

Ce dernier point surprend souvent. **On ne peut pas compiler pour un autre système que le sien.** Pour livrer sur trois plateformes, il faut compiler sur trois machines (ou utiliser une intégration continue qui le fait pour vous).

### 16.15.2 — Les autres cibles de `dart compile`

```bash
dart compile --help
```

**Résultat (extrait) :**

```text
  exe          Compile Dart to a self-contained executable.
  aot-snapshot Compile Dart to an AOT snapshot.
  jit-snapshot Compile Dart to a JIT snapshot.
  kernel       Compile Dart to a kernel snapshot.
  js           Compile Dart to JavaScript.
```

En pratique, en partie 1A, seul `exe` vous servira.

> **Remarque :** ajoutez le nom de l'exécutable produit à votre `.gitignore`. Un binaire de 8 Mo n'a rien à faire dans un dépôt Git (section 16.34).

---

## 16.16 — Le fichier `pubspec.yaml`

`pubspec.yaml` est la **carte d'identité** de votre projet. C'est le seul fichier qu'un projet Dart doit obligatoirement posséder pour être un projet Dart.

Voici celui que `dart create` a produit, complété :

```yaml
name: mon_jeu
description: Un jeu de rôle en mode texte, fil rouge de la formation Dart.
version: 1.0.0
publish_to: 'none'

environment:
  sdk: ^3.5.0

dependencies:
  collection: ^1.18.0

dev_dependencies:
  lints: ^4.0.0
  test: ^1.24.0
```

Reprenons chaque champ.

### 16.16.1 — `name`

```yaml
name: mon_jeu
```

Le nom du package. **Obligatoire.** C'est lui que l'on écrit après `package:` dans les imports :

```dart
import 'package:mon_jeu/joueur.dart';
```

Mêmes règles qu'à la section 16.5.1 : minuscules, tirets bas, pas d'accent.

### 16.16.2 — `description`

```yaml
description: Un jeu de rôle en mode texte, fil rouge de la formation Dart.
```

Une phrase expliquant ce que fait le projet. Techniquement facultative pour un projet privé, obligatoire pour publier sur pub.dev (entre 60 et 180 caractères y sont recommandés).

### 16.16.3 — `version`

```yaml
version: 1.0.0
```

La version de **votre** projet, au format `MAJEUR.MINEUR.CORRECTIF` (voir section 16.23). Vous l'incrémentez à la main quand vous publiez une nouvelle version.

### 16.16.4 — `publish_to`

```yaml
publish_to: 'none'
```

Un garde-fou. Avec `'none'`, la commande `dart pub publish` refuse de s'exécuter. Cela évite de publier par accident sur pub.dev un projet personnel. **Laissez-le tant que vous ne publiez pas.**

### 16.16.5 — `environment`

```yaml
environment:
  sdk: ^3.5.0
```

La version du Dart SDK exigée. Si quelqu'un ouvre votre projet avec un SDK trop ancien, `dart pub get` refuse et affiche une explication claire plutôt que des erreurs incompréhensibles.

**Résultat en cas d'incompatibilité :**

```text
The current Dart SDK version is 3.2.0.

Because mon_jeu requires SDK version ^3.5.0, version solving failed.
```

### 16.16.6 — `dependencies`

```yaml
dependencies:
  collection: ^1.18.0
```

Les packages dont votre programme a besoin **pour fonctionner**. Ils seront livrés avec lui. Voir la section 16.22.

### 16.16.7 — `dev_dependencies`

```yaml
dev_dependencies:
  lints: ^4.0.0
  test: ^1.24.0
```

Les packages dont vous avez besoin **pour développer** : tests, lints, générateurs de code. Ils ne sont pas embarqués dans le programme final.

### 16.16.8 — `executables` (facultatif)

```yaml
executables:
  mon_jeu: mon_jeu
```

Déclare qu'installer ce package doit créer une commande. À gauche le nom de la commande, à droite le fichier de `bin/` sans son extension. Utile seulement si vous distribuez un outil en ligne de commande.

### 16.16.9 — La syntaxe YAML, et son piège

YAML est un format de configuration. Il repose entièrement sur l'**indentation par espaces**.

```text
  Règles YAML absolues :

  1. Indentation avec des ESPACES. Jamais de tabulation.
  2. Deux espaces par niveau (convention Dart).
  3. Un espace APRÈS le deux-points : "name: mon_jeu", pas "name:mon_jeu".
  4. Les enfants sont plus indentés que le parent.
```

Correct :

```yaml
dependencies:
  http: ^1.2.0
  collection: ^1.18.0
```

Incorrect (dépendances au même niveau que la clé) :

```yaml
dependencies:
http: ^1.2.0
```

**Résultat :**

```text
Error on line 8, column 1: Expected a key while parsing a block mapping.
  ╷
8 │ http: ^1.2.0
  │ ^
  ╵
```

Incorrect (tabulation invisible) :

```text
Error on line 9, column 1: Tab characters are not allowed as indentation.
```

> **Le piège numéro un du chapitre :** une tabulation dans un `pubspec.yaml`. Elle est invisible à l'œil nu. Configurez votre éditeur pour insérer des espaces, VS Code le fait par défaut.

---

## 16.17 — `pubspec.lock`

À côté de `pubspec.yaml`, `dart pub get` crée `pubspec.lock`.

La différence entre les deux fichiers est essentielle :

```text
  ┌───────────────────────────────┬──────────────────────────────────┐
  │  pubspec.yaml                 │  pubspec.lock                    │
  ├───────────────────────────────┼──────────────────────────────────┤
  │  Écrit par VOUS               │  Écrit par l'OUTIL               │
  │  Dit ce que vous ACCEPTEZ     │  Dit ce qui est INSTALLÉ         │
  │  "http: ^1.2.0"               │  "http: 1.2.2"                   │
  │  = une plage de versions      │  = une version exacte            │
  │  ~15 lignes                   │  ~100 lignes                     │
  └───────────────────────────────┴──────────────────────────────────┘
```

Extrait réel :

```text
packages:
  http:
    dependency: "direct main"
    description:
      name: http
      sha256: "b9c29a161230ee03d3ccf545097fccd9b87a5264228c5d348202e0f0c28f9010"
      url: "https://pub.dev"
    source: hosted
    version: "1.2.2"
```

À quoi cela sert-il concrètement ? À garantir que **tout le monde installe exactement la même chose**.

```text
  Sans pubspec.lock

  Vous, lundi ........ ^1.2.0 -> installe 1.2.0  -> tout marche
  Collègue, vendredi . ^1.2.0 -> installe 1.3.0  -> bug étrange
  -> "chez moi ça marche"

  Avec pubspec.lock

  Vous, lundi ........ lock dit 1.2.0 -> installe 1.2.0
  Collègue, vendredi . lock dit 1.2.0 -> installe 1.2.0
  -> même code, même comportement
```

### 16.17.1 — Faut-il le mettre dans Git ?

| Type de projet | `pubspec.lock` dans Git ? | Pourquoi |
| --- | --- | --- |
| Application (`console`, Flutter) | **Oui** | On veut des installations reproductibles. |
| Bibliothèque (`package` publié) | **Non** | Ce sont les applications qui décident des versions. |

Le `.gitignore` généré par `dart create -t package` ignore donc `pubspec.lock`, celui d'une application ne l'ignore pas. Ce n'est pas une incohérence : c'est intentionnel.

### 16.17.2 — Le mettre à jour

```bash
dart pub upgrade
```

**Résultat :**

```text
Resolving dependencies...
> http 1.3.0 (was 1.2.2)
Changed 1 dependency!
```

`dart pub upgrade` cherche les versions les plus récentes **compatibles avec vos contraintes** et réécrit le lock. `dart pub get`, lui, respecte le lock existant.

> **À retenir :** ne modifiez jamais `pubspec.lock` à la main. Modifiez `pubspec.yaml`, puis relancez `dart pub get`.

---

## 16.18 — Qu'est-ce qu'un package ?

Un **package** est un ensemble de code Dart écrit par quelqu'un d'autre (ou par vous), empaqueté pour être réutilisé.

L'idée est simple : ne réécrivez pas ce qui existe.

```text
  Vous voulez lire un fichier JSON de sauvegarde.

  Sans package :
     écrire un analyseur JSON  -> ~800 lignes, des semaines, des bugs

  Avec package :
     import 'dart:convert';    -> 1 ligne
```

Un package contient au minimum :

```text
  nom_du_package/
    lib/                 le code réutilisable
    pubspec.yaml         son nom, sa version, ses propres dépendances
    README.md            comment s'en servir
    CHANGELOG.md         ce qui a changé d'une version à l'autre
    LICENSE              les droits d'utilisation
```

Il existe trois familles de code réutilisable en Dart :

| Famille | Exemple | Installation |
| --- | --- | --- |
| Bibliothèques du SDK | `dart:core`, `dart:io`, `dart:math` | Aucune, déjà là. |
| Packages externes | `http`, `test`, `collection` | `dart pub add` |
| Vos propres fichiers | `lib/joueur.dart` | Aucune, ils sont chez vous. |

Quelques packages très utilisés, pour fixer les idées :

| Package | Ce qu'il apporte |
| --- | --- |
| `http` | Faire des requêtes réseau. |
| `test` | Écrire des tests automatiques. |
| `collection` | Des opérations avancées sur les listes et les maps. |
| `path` | Manipuler des chemins de fichiers proprement. |
| `intl` | Dates, nombres et traductions localisés. |
| `args` | Analyser les arguments de la ligne de commande. |
| `json_serializable` | Générer le code de conversion JSON (chapitre 17). |

> **Vocabulaire :** on dit « package » et non « librairie ». En Dart, le mot *library* désigne autre chose : un fichier Dart considéré comme unité de portée (voir section 16.30).

---

## 16.19 — pub.dev

**pub.dev** est le dépôt officiel des packages Dart et Flutter. Tout ce que vous installerez avec `dart pub add` en vient.

### 16.19.1 — Lire une fiche de package

Sur la page d'un package, quatre informations comptent vraiment.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  http 1.2.2                                                  │
  │  A composable, multi-platform, Future-based API for HTTP.    │
  │                                                              │
  │  LIKES 4.5k   PUB POINTS 140/160   DOWNLOADS 2.1M            │
  │                                                              │
  │  [Readme] [Changelog] [Example] [Installing] [Versions]      │
  │                                                              │
  │  Publisher: dart.dev                                         │
  │  Platforms: Android iOS Linux macOS Web Windows              │
  └──────────────────────────────────────────────────────────────┘
```

| Indicateur | Ce qu'il vous dit |
| --- | --- |
| **Likes** | Popularité déclarée par les développeurs. |
| **Pub points** | Note automatique de qualité sur 160 : documentation, null safety, formatage, dépendances à jour. |
| **Downloads** | Usage réel sur 30 jours. Le signal le plus fiable. |
| **Publisher** | Qui publie. `dart.dev`, `flutter.dev`, `google.dev` sont des éditeurs vérifiés. |
| **Platforms** | Sur quels systèmes le package fonctionne. |

### 16.19.2 — Choisir un package : une petite checklist

1. Le nombre de téléchargements est-il élevé ?
2. La dernière publication date-t-elle de moins d'un an ?
3. Les pub points dépassent-ils 120 sur 160 ?
4. Le README montre-t-il un exemple compréhensible ?
5. Le package supporte-t-il les plateformes qui vous concernent ?
6. La licence convient-elle (MIT et BSD sont permissives) ?

Un package abandonné depuis quatre ans, avec 40 téléchargements, est un risque : il ne sera pas corrigé, et il finira par bloquer vos mises à jour.

### 16.19.3 — L'onglet « Installing »

C'est celui qui vous intéresse en pratique. Il affiche exactement la commande à taper :

```bash
dart pub add http
```

et la ligne à ajouter si vous préférez modifier le `pubspec.yaml` à la main :

```yaml
dependencies:
  http: ^1.2.2
```

> **Prudence :** n'installez pas un package « pour voir ». Chaque dépendance ajoute du code que vous ne contrôlez pas, et qu'il faudra maintenir. Le meilleur package est souvent celui dont on peut se passer.

---

## 16.20 — Ajouter une dépendance (`dart pub add`)

Deux façons d'ajouter un package. La commande est de loin préférable.

### 16.20.1 — La bonne méthode

```bash
dart pub add http
```

**Résultat :**

```text
Resolving dependencies...
+ http 1.2.2
+ http_parser 4.0.2
Changed 2 dependencies!
```

En une commande, l'outil a :

1. cherché la dernière version de `http` sur pub.dev ;
2. ajouté `http: ^1.2.2` dans `dependencies` ;
3. installé aussi `http_parser`, dont `http` a besoin (dépendance transitive) ;
4. mis à jour `pubspec.lock` ;
5. lancé `dart pub get`.

Votre `pubspec.yaml` contient maintenant :

```yaml
dependencies:
  http: ^1.2.2
```

Vous n'avez rien tapé à la main, donc aucune faute de frappe, aucune erreur d'indentation, aucune version inventée.

### 16.20.2 — Ajouter en `dev_dependencies`

```bash
dart pub add dev:test
```

**Résultat :**

```text
Resolving dependencies...
+ test 1.25.7
Changed 1 dependency!
```

Le préfixe `dev:` place la ligne dans `dev_dependencies` :

```yaml
dev_dependencies:
  test: ^1.25.7
```

### 16.20.3 — Demander une version précise

```bash
dart pub add http:^1.1.0
```

Utile si une version plus récente casse quelque chose chez vous.

### 16.20.4 — Retirer une dépendance

```bash
dart pub remove http
```

**Résultat :**

```text
Resolving dependencies...
These packages are no longer being depended on:
- http 1.2.2
- http_parser 4.0.2
Changed 2 dependencies!
```

### 16.20.5 — La méthode manuelle

Vous pouvez éditer `pubspec.yaml` vous-même :

```yaml
dependencies:
  http: ^1.2.2
```

Mais il faut alors impérativement lancer :

```bash
dart pub get
```

Sinon le package n'est pas téléchargé et l'import échouera. C'est l'une des erreurs les plus fréquentes chez le débutant.

> **Règle simple :** utilisez `dart pub add`. Réservez l'édition manuelle aux cas particuliers (dépendance vers un dossier local, vers un dépôt Git).

---

## 16.21 — `dart pub get`

`dart pub get` installe les dépendances déclarées dans `pubspec.yaml`.

```bash
dart pub get
```

**Résultat :**

```text
Resolving dependencies...
Got dependencies!
```

### 16.21.1 — Ce que fait exactement la commande

```text
  dart pub get
     │
     ├─ 1. lit pubspec.yaml (ce que vous acceptez)
     ├─ 2. lit pubspec.lock (ce qui est déjà figé)
     ├─ 3. résout les versions compatibles de tout l'arbre
     ├─ 4. télécharge ce qui manque dans le cache global
     ├─ 5. écrit .dart_tool/package_config.json
     └─ 6. met à jour pubspec.lock si nécessaire
```

Le cache global se trouve dans votre dossier utilisateur (`~/.pub-cache` sous macOS et Linux, `%LOCALAPPDATA%\Pub\Cache` sous Windows). Les packages ne sont donc **pas** copiés dans chaque projet : ils sont téléchargés une fois et partagés.

### 16.21.2 — Quand faut-il la lancer ?

| Situation | `dart pub get` nécessaire ? |
| --- | --- |
| Après `dart pub add` | Non, c'est déjà fait automatiquement. |
| Après avoir édité `pubspec.yaml` à la main | **Oui.** |
| Après avoir cloné un projet depuis Git | **Oui.** |
| Après avoir supprimé `.dart_tool/` | **Oui.** |
| Après avoir changé de version du SDK | **Oui.** |

### 16.21.3 — L'erreur que vous verrez un jour

```dart
import 'package:http/http.dart' as http;
```

**Résultat :**

```text
Error: Couldn't resolve the package 'http' in 'package:http/http.dart'.
Target of URI doesn't exist: 'package:http/http.dart'.
```

Traduction : « je ne sais pas où est ce package ». Dans 90 % des cas, la réponse est `dart pub get`.

### 16.21.4 — `get` contre `upgrade`

```text
  dart pub get       respecte pubspec.lock
                     -> installe les versions déjà figées

  dart pub upgrade   ignore pubspec.lock
                     -> prend la plus récente autorisée par pubspec.yaml
                     -> réécrit pubspec.lock
```

Vous utiliserez `get` cent fois pour un `upgrade`.

---

## 16.22 — `dependencies` vs `dev_dependencies`

La distinction est simple une fois posée dans les bons termes.

```text
  ┌─────────────────────────────┬──────────────────────────────────┐
  │  dependencies               │  dev_dependencies                │
  ├─────────────────────────────┼──────────────────────────────────┤
  │  Nécessaire pour que le     │  Nécessaire seulement pour       │
  │  programme FONCTIONNE       │  DÉVELOPPER le programme         │
  │                             │                                  │
  │  Livré à l'utilisateur      │  Reste chez vous                 │
  │                             │                                  │
  │  http, collection, path     │  test, lints, build_runner       │
  └─────────────────────────────┴──────────────────────────────────┘
```

Le test décisif : **si je supprime ce package, le programme compilé fonctionne-t-il encore ?**

- Oui → `dev_dependencies`.
- Non → `dependencies`.

Exemple typé fil rouge :

```yaml
name: mon_jeu
description: Un jeu de rôle en mode texte.
version: 1.0.0
publish_to: 'none'

environment:
  sdk: ^3.5.0

dependencies:
  # Le jeu télécharge le classement en ligne : indispensable au jeu.
  http: ^1.2.2
  # Le jeu trie l'inventaire avec des comparateurs avancés.
  collection: ^1.18.0

dev_dependencies:
  # Sert à vérifier le jeu, pas à le faire tourner.
  test: ^1.25.7
  # Sert à surveiller la qualité du code, pas à le faire tourner.
  lints: ^4.0.0
```

### 16.22.1 — Une erreur importante

Mettre `test` dans `dependencies` :

```yaml
dependencies:
  test: ^1.25.7      # NON
```

Conséquences : votre exécutable embarque tout le framework de test, il grossit inutilement, et quiconque dépend de votre package hérite d'une dépendance dont il n'a rien à faire. L'analyseur vous le signalera d'ailleurs par un lint.

### 16.22.2 — Une conséquence pratique

Le code de `lib/` et de `bin/` ne peut utiliser que les packages de `dependencies`. Le code de `test/` peut utiliser les deux.

```dart
// Dans test/joueur_test.dart : autorisé
import 'package:test/test.dart';
import 'package:mon_jeu/joueur.dart';
```

```dart
// Dans lib/joueur.dart : interdit (test est une dev_dependency)
import 'package:test/test.dart';
```

**Résultat :**

```text
info • lib/joueur.dart:1:8 • The imported package 'test' isn't a dependency of
the importing package. • depend_on_referenced_packages
```

> **À retenir :** `dependencies` = ce qui part avec le jeu. `dev_dependencies` = ce qui reste dans l'atelier.

---

## 16.23 — Le versionnage sémantique et le caret `^`

### 16.23.1 — Les trois nombres

Une version Dart s'écrit toujours avec trois nombres :

```text
        1  .  4  .  2
        ▲     ▲     ▲
        │     │     └── CORRECTIF (patch)
        │     │         correction de bug, aucun changement d'usage
        │     │
        │     └──────── MINEUR (minor)
        │               ajout de fonctionnalité, sans rien casser
        │
        └────────────── MAJEUR (major)
                        changement incompatible : votre code peut casser
```

C'est le **versionnage sémantique** (*semantic versioning*, souvent abrégé « semver »). Il s'agit d'un contrat entre l'auteur du package et vous :

| Passage | Promesse de l'auteur |
| --- | --- |
| `1.4.2` → `1.4.3` | Un bug corrigé. Mettez à jour sans crainte. |
| `1.4.2` → `1.5.0` | Des nouveautés. Votre code existant continue de marcher. |
| `1.4.2` → `2.0.0` | Attention : quelque chose a changé. Lisez le CHANGELOG. |

### 16.23.2 — Le caret `^`

```yaml
dependencies:
  http: ^1.2.2
```

`^1.2.2` se lit : « au moins 1.2.2, et tout ce qui suit **tant que le nombre majeur ne change pas** ».

```text
  ^1.2.2  autorise
  ├── 1.2.2   oui
  ├── 1.2.9   oui
  ├── 1.3.0   oui
  ├── 1.99.5  oui
  └── 2.0.0   NON  (majeur différent)

  Équivalent écrit en clair : >=1.2.2 <2.0.0
```

C'est le bon compromis : vous recevez automatiquement les corrections de bugs et les nouveautés, mais jamais un changement susceptible de casser votre code.

### 16.23.3 — Les autres notations

| Écriture | Signification | Quand l'utiliser |
| --- | --- | --- |
| `^1.2.2` | `>=1.2.2 <2.0.0` | **Cas normal.** |
| `1.2.2` | Exactement cette version. | Quand une version précise est indispensable. |
| `>=1.2.0 <1.5.0` | Plage explicite. | Cas particuliers. |
| `any` | N'importe quelle version. | À éviter : imprévisible. |
| `sdk: ^3.5.0` | Contrainte sur le SDK. | Toujours dans `environment`. |

### 16.23.4 — Le cas des versions 0.x

Avant `1.0.0`, l'API est réputée instable. Le caret se comporte donc différemment :

```text
  ^0.3.4  autorise  >=0.3.4 <0.4.0
                    (le MINEUR joue le rôle du majeur)
```

C'est logique : en `0.x`, chaque changement de nombre mineur peut casser des choses.

### 16.23.5 — Voir les mises à jour disponibles

```bash
dart pub outdated
```

**Résultat :**

```text
Package Name  Current  Upgradable  Resolvable  Latest

direct dependencies:
http          1.2.2    1.2.2       1.3.0       1.3.0
collection    1.18.0   1.18.0      1.18.0      1.19.0

1 upgradable dependency is locked (in pubspec.lock) to an older version.
To update it, use `dart pub upgrade`.
```

Lecture des colonnes : `Current` est installé, `Resolvable` est atteignable en respectant vos contraintes, `Latest` est la dernière version publiée (parfois hors de vos contraintes, donc nécessitant une modification du `pubspec.yaml`).

> **À retenir :** le caret `^` est votre réglage par défaut. Ne l'enlevez que pour une raison précise et documentée.

---

## 16.24 — `import` : les trois formes

Un `import` dit à Dart : « va chercher du code ailleurs et rends-le utilisable ici ». Il en existe exactement trois formes, reconnaissables au début de la chaîne.

```text
  import 'dart:math';                    forme 1 : bibliothèque du SDK
  import 'package:http/http.dart';       forme 2 : package externe
  import 'joueur.dart';                  forme 3 : chemin relatif
```

### 16.24.1 — Forme 1 : `dart:`

Les bibliothèques livrées avec le SDK. Aucune installation, aucune ligne dans `pubspec.yaml`.

```dart
import 'dart:math';

void main() {
  final generateur = Random(42);
  final degats = 10 + generateur.nextInt(15);
  print('Dégâts infligés : $degats');
  print('Racine de 144   : ${sqrt(144)}');
}
```

**Résultat :**

```text
Dégâts infligés : 10
Racine de 144   : 12.0
```

Les plus utiles :

| Bibliothèque | Contenu |
| --- | --- |
| `dart:core` | `String`, `int`, `List`, `print`... **Importée automatiquement.** |
| `dart:math` | `Random`, `sqrt`, `pi`, `max`, `min`. |
| `dart:io` | Fichiers, dossiers, entrée clavier. Indisponible sur le web. |
| `dart:convert` | JSON, UTF-8 (chapitre 17). |
| `dart:async` | `Future`, `Stream`, `Timer` (chapitre 15). |

`dart:core` est le seul import implicite du langage. C'est pourquoi vous n'avez jamais eu à importer quoi que ce soit pour utiliser `print`.

### 16.24.2 — Forme 2 : `package:`

Un package externe, ou **votre propre package**.

```dart
import 'package:mon_jeu/joueur.dart';
```

Structure de la chaîne :

```text
  package:mon_jeu/combat/attaque.dart
          ▲       ▲
          │       └── chemin à partir de lib/ (donc lib/combat/attaque.dart)
          └── valeur du champ "name" du pubspec.yaml
```

Deux erreurs classiques :

```dart
import 'package:mon_jeu/lib/joueur.dart';   // NON : lib/ est déjà implicite
import 'package:MonJeu/joueur.dart';        // NON : le nom est en minuscules
```

### 16.24.3 — Forme 3 : chemin relatif

Un fichier de votre projet, désigné par sa position relative au fichier courant.

```dart
import 'joueur.dart';           // même dossier
import 'combat/attaque.dart';   // sous-dossier
import '../joueur.dart';        // dossier parent
```

Le `..` signifie « remonter d'un niveau ».

```text
  lib/
    joueur.dart
    combat/
      attaque.dart      pour atteindre joueur.dart : '../joueur.dart'
```

### 16.24.4 — Relatif ou `package:` ? La règle

| Situation | Forme à employer |
| --- | --- |
| Un fichier de `lib/` importe un autre fichier de `lib/` | Chemin relatif. |
| Un fichier de `bin/` importe un fichier de `lib/` | `package:`. |
| Un fichier de `test/` importe un fichier de `lib/` | `package:`. |
| N'importe quel fichier importe un package externe | `package:`. |

La raison est structurelle : `bin/` et `test/` sont **hors** de `lib/`. Un chemin relatif du type `../lib/joueur.dart` fonctionne parfois mais est considéré comme fautif, et les lints le refusent.

```dart
// Dans bin/mon_jeu.dart
import '../lib/joueur.dart';        // NON
import 'package:mon_jeu/joueur.dart';  // OUI
```

### 16.24.5 — Où placer les imports

Tous les `import` se placent **en haut du fichier**, avant toute déclaration. L'ordre conventionnel, appliqué par le lint `directives_ordering`, est :

```dart
// 1. dart:
import 'dart:io';
import 'dart:math';

// 2. package:
import 'package:collection/collection.dart';
import 'package:mon_jeu/joueur.dart';

// 3. relatifs
import 'ennemi.dart';
import 'combat/attaque.dart';
```

Chaque groupe est trié par ordre alphabétique, et séparé du suivant par une ligne vide.

---

## 16.25 — `show` et `hide`

Par défaut, un `import` rend visible **tout** ce que le fichier importé expose. Ce n'est pas toujours souhaitable.

### 16.25.1 — `show` : ne prendre que ceci

```dart
import 'dart:math' show Random, pi;

void main() {
  final generateur = Random(7);
  print('Nombre : ${generateur.nextInt(100)}');
  print('Pi     : $pi');
}
```

**Résultat :**

```text
Nombre : 7
Pi     : 3.141592653589793
```

Seuls `Random` et `pi` sont accessibles. `sqrt` ne l'est pas :

```dart
import 'dart:math' show Random, pi;

void main() {
  print(sqrt(16));
}
```

**Résultat :**

```text
Error: Method not found: 'sqrt'.
```

### 16.25.2 — `hide` : prendre tout sauf ceci

```dart
import 'dart:math' hide Random;

void main() {
  print(sqrt(81));
  print(max(12, 30));
}
```

**Résultat :**

```text
9.0
30
```

Tout `dart:math` est disponible, sauf `Random`.

### 16.25.3 — Le cas qui rend ces mots-clés indispensables

Deux fichiers définissent le même nom :

```dart
// lib/monstres/dragon.dart
class Dragon {
  final int puissanceDeFeu;
  Dragon(this.puissanceDeFeu);
  String decrire() => 'Dragon des montagnes ($puissanceDeFeu)';
}
```

```dart
// lib/boss/dragon.dart
class Dragon {
  final int phases;
  Dragon(this.phases);
  String decrire() => 'Dragon final ($phases phases)';
}
```

Importer les deux sans précaution :

```dart
import 'package:mon_jeu/monstres/dragon.dart';
import 'package:mon_jeu/boss/dragon.dart';

void main() {
  final d = Dragon(3);
  print(d.decrire());
}
```

**Résultat :**

```text
Error: 'Dragon' is imported from both 'package:mon_jeu/monstres/dragon.dart'
and 'package:mon_jeu/boss/dragon.dart'.
```

`hide` règle le conflit :

```dart
import 'package:mon_jeu/monstres/dragon.dart';
import 'package:mon_jeu/boss/dragon.dart' hide Dragon;

void main() {
  final d = Dragon(50);
  print(d.decrire());
}
```

**Résultat :**

```text
Dragon des montagnes (50)
```

### 16.25.4 — Faut-il en abuser ?

Non. `show` et `hide` alourdissent la lecture et doivent être mis à jour à chaque évolution. Réservez-les à deux cas :

1. un conflit de noms réel ;
2. un import volontairement restreint pour documenter une intention.

> **Remarque :** `show` et `hide` peuvent lister plusieurs noms, séparés par des virgules : `show Joueur, Ennemi, Arme`.

---

## 16.26 — `as` (préfixe)

`as` donne un **préfixe** à un import. Tout ce qui vient de cet import doit alors être écrit `prefixe.Nom`.

```dart
import 'package:mon_jeu/monstres/dragon.dart' as monstres;
import 'package:mon_jeu/boss/dragon.dart' as boss;

void main() {
  final petit = monstres.Dragon(50);
  final final_ = boss.Dragon(3);

  print(petit.decrire());
  print(final_.decrire());
}
```

**Résultat :**

```text
Dragon des montagnes (50)
Dragon final (3 phases)
```

Les deux `Dragon` coexistent sans ambiguïté. C'est la solution la plus lisible en cas de conflit : contrairement à `hide`, elle ne vous prive de rien.

### 16.26.1 — L'usage le plus courant

Le package `http` est presque toujours importé avec un préfixe, par convention de son auteur :

```dart
import 'package:http/http.dart' as http;

Future<void> chargerClassement() async {
  final reponse = await http.get(Uri.parse('https://exemple.com/scores'));
  print('Statut : ${reponse.statusCode}');
}
```

Le préfixe indique clairement d'où vient `get`, un nom bien trop générique pour être laissé nu.

### 16.26.2 — Combiner `as` et `show`

```dart
import 'dart:math' as math show Random, pi;

void main() {
  print(math.pi);
  print(math.Random(1).nextInt(10));
}
```

**Résultat :**

```text
3.141592653589793
6
```

### 16.26.3 — Quand l'utiliser

| Cas | Préfixe utile ? |
| --- | --- |
| Deux packages exposent le même nom | Oui, c'est la meilleure solution. |
| Le package a des noms très génériques (`get`, `parse`) | Oui. |
| Vous importez un seul fichier de votre projet | Non, cela alourdit. |

> **Piège :** un préfixe n'est pas une variable. `monstres` seul ne vaut rien ; il n'existe qu'accolé à un nom : `monstres.Dragon`.

---

## 16.27 — Découper le code en plusieurs fichiers

Voici le cœur pratique du chapitre. Partons d'un fichier unique de type DartPad et découpons-le.

### 16.27.1 — Le point de départ

```dart
// bin/mon_jeu.dart — TOUT dans un seul fichier
class Joueur {
  final String nom;
  int vies;
  int score;
  Joueur(this.nom, {this.vies = 3, this.score = 0});
}

class Ennemi {
  final String nom;
  int pointsDeVie;
  final int degats;
  Ennemi(this.nom, this.pointsDeVie, this.degats);
}

class Arme {
  final String nom;
  final int puissance;
  const Arme(this.nom, this.puissance);
}

void main() {
  final heros = Joueur('Alex');
  final gobelin = Ennemi('Gobelin', 30, 5);
  const epee = Arme('Épée de feu', 12);
  print('${heros.nom} affronte ${gobelin.nom} avec ${epee.nom}');
}
```

Cela fonctionne. À 60 lignes, c'est encore lisible. À 600, ce ne l'est plus.

### 16.27.2 — La règle de découpage

> **Une classe publique = un fichier. Le fichier porte le nom de la classe, en snake_case.**

| Classe | Fichier |
| --- | --- |
| `Joueur` | `lib/joueur.dart` |
| `Ennemi` | `lib/ennemi.dart` |
| `Arme` | `lib/arme.dart` |
| `InventaireJoueur` | `lib/inventaire_joueur.dart` |

### 16.27.3 — Le découpage effectué

```text
  mon_jeu/
    bin/
      mon_jeu.dart
    lib/
      joueur.dart
      ennemi.dart
      arme.dart
```

`lib/arme.dart` :

```dart
class Arme {
  final String nom;
  final int puissance;

  const Arme(this.nom, this.puissance);

  @override
  String toString() => '$nom (+$puissance)';
}
```

`lib/joueur.dart` :

```dart
import 'arme.dart';

class Joueur {
  final String nom;
  int vies;
  int score;
  Arme? arme;

  Joueur(this.nom, {this.vies = 3, this.score = 0, this.arme});

  int get degats => 5 + (arme?.puissance ?? 0);

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  @override
  String toString() => '$nom (vies: $vies, score: $score)';
}
```

Notez l'import relatif `'arme.dart'` : les deux fichiers sont dans `lib/`.

`lib/ennemi.dart` :

```dart
class Ennemi {
  final String nom;
  int pointsDeVie;
  final int degats;

  Ennemi(this.nom, this.pointsDeVie, this.degats);

  bool get estVaincu => pointsDeVie <= 0;

  void subirDegats(int montant) {
    pointsDeVie -= montant;
    if (pointsDeVie < 0) pointsDeVie = 0;
  }

  @override
  String toString() => '$nom ($pointsDeVie PV)';
}
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/arme.dart';
import 'package:mon_jeu/ennemi.dart';
import 'package:mon_jeu/joueur.dart';

void main() {
  const epee = Arme('Épée de feu', 12);
  final heros = Joueur('Alex', arme: epee);
  final gobelin = Ennemi('Gobelin', 30, 5);

  print('$heros affronte $gobelin');

  while (!gobelin.estVaincu) {
    gobelin.subirDegats(heros.degats);
    print('Coup porté ! $gobelin');
  }

  print('${gobelin.nom} est vaincu.');
}
```

```bash
dart run
```

**Résultat :**

```text
Alex (vies: 3, score: 0) affronte Gobelin (30 PV)
Coup porté ! Gobelin (13 PV)
Coup porté ! Gobelin (0 PV)
Gobelin est vaincu.
```

Même comportement, mais quatre fichiers courts au lieu d'un long.

### 16.27.4 — Organiser en sous-dossiers

Au-delà d'une quinzaine de fichiers, regroupez par thème :

```text
  lib/
    personnages/
      joueur.dart
      ennemi.dart
      boss.dart
    objets/
      arme.dart
      potion.dart
      inventaire.dart
    monde/
      niveau.dart
      salle.dart
    utils/
      des.dart
```

Les imports deviennent :

```dart
import 'package:mon_jeu/personnages/joueur.dart';
import 'package:mon_jeu/objets/arme.dart';
```

Regroupez par **domaine** (personnages, objets, monde), et non par nature technique (classes, fonctions, constantes). Un dossier `models/` fourre-tout contenant 40 fichiers ne vous aide pas.

---

## 16.28 — `part` / `part of` (et pourquoi on l'évite)

Dart propose un second mécanisme pour répartir du code sur plusieurs fichiers : `part` et `part of`. Vous le rencontrerez, il faut donc savoir le lire.

### 16.28.1 — Comment cela s'écrit

Le fichier principal déclare ses parties :

```dart
// lib/combat.dart
part 'combat_attaque.dart';
part 'combat_defense.dart';

class Combat {
  final String lieu;
  Combat(this.lieu);
}
```

Chaque partie déclare son appartenance :

```dart
// lib/combat_attaque.dart
part of 'combat.dart';

int calculerAttaque(int force, int bonusArme) => force * 2 + bonusArme;
```

```dart
// lib/combat_defense.dart
part of 'combat.dart';

int calculerDefense(int armure) => armure ~/ 2;
```

À l'usage, tout se comporte comme si le code n'était qu'un seul fichier :

```dart
import 'package:mon_jeu/combat.dart';

void main() {
  print('Attaque : ${calculerAttaque(10, 12)}');
  print('Défense : ${calculerDefense(9)}');
}
```

**Résultat :**

```text
Attaque : 32
Défense : 4
```

### 16.28.2 — La différence avec `import`

```text
  import  : deux FICHIERS distincts, deux bibliothèques distinctes.
            Les membres privés _ ne traversent PAS la frontière.

  part    : un seul ensemble découpé en morceaux.
            Les membres privés _ sont partagés entre tous les morceaux.
```

Un fichier `part` ne peut pas avoir ses propres imports : il hérite de ceux du fichier principal. C'est justement là que le bât blesse.

### 16.28.3 — Pourquoi on l'évite

| Problème | Conséquence |
| --- | --- |
| Un `part` ne peut pas déclarer ses imports | Impossible de lire le fichier isolément : on ne sait pas d'où viennent les noms. |
| Couplage fort | Un `part` ne s'utilise pas hors de son fichier principal. |
| Ordre de déclaration | Oublier une ligne `part` casse la compilation avec un message obscur. |
| Encapsulation affaiblie | Les `_membres` deviennent visibles depuis tous les morceaux. |

La documentation officielle de Dart recommande d'utiliser `import` et `export`, et de réserver `part` à un usage précis.

### 16.28.4 — Le cas légitime : le code généré

Vous verrez `part` très souvent au chapitre 17, avec la génération de code JSON :

```dart
import 'package:json_annotation/json_annotation.dart';

part 'joueur.g.dart';

@JsonSerializable()
class Joueur {
  final String nom;
  final int score;
  Joueur(this.nom, this.score);

  factory Joueur.fromJson(Map<String, dynamic> json) => _$JoueurFromJson(json);
  Map<String, dynamic> toJson() => _$JoueurToJson(this);
}
```

Le fichier `joueur.g.dart` est écrit par une machine, jamais par vous. Il doit accéder aux membres privés de `Joueur`. `part` est ici le bon outil.

> **Règle pratique :** vous n'écrivez jamais `part` vous-même. Vous en écrivez uniquement quand un générateur de code vous l'impose.

---

## 16.29 — `export` et le fichier « barrel »

### 16.29.1 — Le problème

Après le découpage de la section 16.27, chaque utilisateur de votre code doit écrire :

```dart
import 'package:mon_jeu/personnages/joueur.dart';
import 'package:mon_jeu/personnages/ennemi.dart';
import 'package:mon_jeu/personnages/boss.dart';
import 'package:mon_jeu/objets/arme.dart';
import 'package:mon_jeu/objets/potion.dart';
import 'package:mon_jeu/objets/inventaire.dart';
import 'package:mon_jeu/monde/niveau.dart';
```

Sept lignes. Et si vous renommez un dossier, les sept lignes cassent, dans tous les fichiers du projet.

### 16.29.2 — `export`

`export` réexporte le contenu d'un autre fichier, comme s'il était défini sur place.

```dart
// lib/mon_jeu.dart — le fichier "barrel"
export 'personnages/joueur.dart';
export 'personnages/ennemi.dart';
export 'personnages/boss.dart';
export 'objets/arme.dart';
export 'objets/potion.dart';
export 'objets/inventaire.dart';
export 'monde/niveau.dart';
```

Ce fichier ne contient aucune classe. Il ne fait que rassembler.

Les utilisateurs n'écrivent plus qu'une ligne :

```dart
import 'package:mon_jeu/mon_jeu.dart';

void main() {
  const epee = Arme('Épée de feu', 12);
  final heros = Joueur('Alex', arme: epee);
  final gobelin = Ennemi('Gobelin', 30, 5);
  print('$heros contre $gobelin avec $epee');
}
```

**Résultat :**

```text
Alex (vies: 3, score: 0) contre Gobelin (30 PV) avec Épée de feu (+12)
```

### 16.29.3 — Pourquoi cela s'appelle un « barrel »

*Barrel* signifie « tonneau » : on verse dedans tout le contenu du package, et on n'ouvre qu'un seul robinet. Le fichier porte par convention le **nom du package** : `lib/mon_jeu.dart` pour le package `mon_jeu`. C'est d'ailleurs ce fichier que `dart create` génère par défaut.

### 16.29.4 — Filtrer un `export`

`export` accepte `show` et `hide`, exactement comme `import` :

```dart
export 'objets/inventaire.dart' show Inventaire;
export 'monde/niveau.dart' hide GenerateurInterne;
```

C'est ainsi qu'on expose une API publique nette : `lib/src/` contient tout, et le barrel choisit ce qui sort.

```text
  lib/
    mon_jeu.dart          <- API publique : quelques export
    src/
      joueur.dart         <- interne
      ennemi.dart         <- interne
      moteur_prive.dart   <- interne, JAMAIS exporté
```

### 16.29.5 — L'inconvénient

Un barrel importe tout, même ce dont vous n'avez pas besoin. Sur un package énorme, cela allonge légèrement l'analyse et peut créer des conflits de noms inattendus.

| Contexte | Recommandation |
| --- | --- |
| Package destiné à d'autres | Barrel indispensable. |
| Application `console` moyenne | Barrel confortable. |
| Petit projet de 4 fichiers | Facultatif, imports directs très bien. |

> **À retenir :** `import` fait entrer du code chez vous. `export` fait sortir du code de chez vous.

---

## 16.30 — Membres privés `_` et portée bibliothèque

Rappel du chapitre 10 : en Dart, un identifiant commençant par un tiret bas est **privé**.

```dart
class Joueur {
  final String nom;
  int _pointsDeVie;

  Joueur(this.nom, this._pointsDeVie);

  int get pointsDeVie => _pointsDeVie;

  void _reinitialiser() {
    _pointsDeVie = 100;
  }

  void ressusciter() {
    _reinitialiser();
  }
}
```

Ce que le chapitre 10 n'avait pas pu montrer, faute de projet multi-fichiers, c'est **jusqu'où** va cette privauté.

### 16.30.1 — La règle exacte

> En Dart, `_` n'est pas privé « à la classe ». Il est privé **à la bibliothèque**, c'est-à-dire au fichier.

```text
  ┌────────────────────────────────────────────────────────────┐
  │  lib/joueur.dart                                           │
  │                                                            │
  │    class Joueur {                                          │
  │      int _pointsDeVie;      <- privé                       │
  │    }                                                       │
  │                                                            │
  │    class Soigneur {                                        │
  │      void soigner(Joueur j) {                              │
  │        j._pointsDeVie = 100;   <- AUTORISÉ (même fichier)  │
  │      }                                                     │
  │    }                                                       │
  └────────────────────────────────────────────────────────────┘

  ┌────────────────────────────────────────────────────────────┐
  │  lib/potion.dart                                           │
  │                                                            │
  │    class Potion {                                          │
  │      void boire(Joueur j) {                                │
  │        j._pointsDeVie = 100;   <- INTERDIT (autre fichier) │
  │      }                                                     │
  │    }                                                       │
  └────────────────────────────────────────────────────────────┘
```

Le code du second fichier :

```dart
import 'joueur.dart';

class Potion {
  void boire(Joueur j) {
    j._pointsDeVie = 100;
  }
}
```

**Résultat :**

```text
Error: The getter '_pointsDeVie' isn't defined for the class 'Joueur'.
```

Le message est révélateur : depuis l'extérieur du fichier, le membre **n'existe pas**.

### 16.30.2 — La conséquence pratique du découpage

Tant que tout était dans un seul fichier DartPad, `_` ne vous gênait jamais. Dès que vous découpez, il devient une vraie frontière. La bonne réaction n'est pas de retirer le `_`, mais d'exposer une méthode publique :

```dart
class Joueur {
  final String nom;
  int _pointsDeVie;

  Joueur(this.nom, this._pointsDeVie);

  int get pointsDeVie => _pointsDeVie;

  void soigner(int montant) {
    _pointsDeVie += montant;
    if (_pointsDeVie > 100) _pointsDeVie = 100;
  }
}
```

```dart
import 'joueur.dart';

class Potion {
  final int soin;
  const Potion(this.soin);

  void boire(Joueur j) => j.soigner(soin);
}
```

```dart
import 'package:mon_jeu/joueur.dart';
import 'package:mon_jeu/potion.dart';

void main() {
  final heros = Joueur('Alex', 40);
  const petitePotion = Potion(35);

  print('Avant : ${heros.pointsDeVie}');
  petitePotion.boire(heros);
  print('Après : ${heros.pointsDeVie}');
  petitePotion.boire(heros);
  print('Encore : ${heros.pointsDeVie}');
}
```

**Résultat :**

```text
Avant : 40
Après : 75
Encore : 100
```

Le plafond à 100 est garanti par la classe `Joueur` elle-même. C'est exactement ce que l'encapsulation doit produire.

### 16.30.3 — Les fichiers privés

La convention `lib/src/` (section 16.9.1) joue le même rôle à l'échelle du package : tout y est techniquement public, mais rien n'y est destiné à l'extérieur. Un lint (`implementation_imports`) signale quiconque importe le `src/` d'un autre package.

> **À retenir :** `_` = privé au **fichier**. `lib/src/` = privé au **package**.

---

## 16.31 — `analysis_options.yaml` et les lints

`dart create` a produit ce fichier :

```yaml
include: package:lints/recommended.yaml
```

Une ligne, mais elle active des dizaines de règles.

### 16.31.1 — Qu'est-ce qu'un lint ?

Un **lint** est une règle de style ou de sûreté vérifiée par l'analyseur. Un lint ne dit pas « ce code est faux », il dit « ce code est légal mais discutable ».

```dart
void main() {
  var vies = 3;
  print(vies);
}
```

**Résultat de `dart analyze` :**

```text
info • bin/mon_jeu.dart:2:3 • Prefer using final for a local variable that
is never reassigned. • prefer_final_locals
```

Le programme fonctionne parfaitement. Le lint suggère simplement `final vies = 3;`.

### 16.31.2 — Les jeux de règles disponibles

| Jeu | Contenu | Pour qui |
| --- | --- | --- |
| `package:lints/core.yaml` | Le minimum vital. | Tout projet. |
| `package:lints/recommended.yaml` | Le minimum + le style officiel Dart. | **Défaut recommandé.** |
| `package:flutter_lints/flutter.yaml` | Recommandé + règles Flutter. | Projets Flutter. |
| `package:very_good_analysis/analysis_options.yaml` | Très strict. | Équipes exigeantes. |

### 16.31.3 — Ajouter des règles

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    - prefer_final_locals
    - always_declare_return_types
    - avoid_print
    - prefer_single_quotes
    - require_trailing_commas
    - unnecessary_this
```

| Règle | Ce qu'elle impose |
| --- | --- |
| `prefer_final_locals` | `final` sur les variables locales jamais réaffectées. |
| `always_declare_return_types` | Un type de retour explicite sur chaque fonction. |
| `avoid_print` | Pas de `print` (à activer seulement quand vous avez un vrai logger). |
| `prefer_single_quotes` | Guillemets simples pour les chaînes. |
| `require_trailing_commas` | Virgule finale dans les listes d'arguments multilignes. |
| `unnecessary_this` | Pas de `this.` quand il est superflu. |

La liste complète est publiée sur `dart.dev/tools/linter-rules`.

### 16.31.4 — Désactiver une règle

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    avoid_print: false
```

Ici on garde tout `recommended`, sauf `avoid_print` — ce qui est cohérent pour un jeu en console, où `print` est l'affichage principal.

### 16.31.5 — Exclure des fichiers

```yaml
include: package:lints/recommended.yaml

analyzer:
  exclude:
    - '**/*.g.dart'
    - 'build/**'
```

Les fichiers `.g.dart` sont générés (section 16.28.4) : les analyser n'a aucun intérêt, vous ne pouvez pas les corriger.

### 16.31.6 — Durcir la sévérité

```yaml
analyzer:
  errors:
    unused_import: error
    todo: ignore
```

Un import inutilisé devient une **erreur** bloquante, tandis que les commentaires `// TODO` cessent d'apparaître dans le rapport.

### 16.31.7 — Ignorer ponctuellement

```dart
// ignore: avoid_print
print('Message de débogage temporaire');
```

ou pour tout un fichier, en première ligne :

```dart
// ignore_for_file: avoid_print
```

À utiliser avec parcimonie. Un fichier truffé de `ignore` est un fichier dont les règles sont mal choisies.

> **Conseil :** commencez avec `recommended`. Ajoutez une règle quand une erreur vous a réellement coûté du temps. Ne copiez pas une configuration de 200 lignes trouvée en ligne.

---

## 16.32 — Écrire un premier test avec `package:test`

### 16.32.1 — Installer le package

```bash
dart pub add dev:test
```

**Résultat :**

```text
Resolving dependencies...
+ test 1.25.7
Changed 1 dependency!
```

### 16.32.2 — Le code à tester

`lib/joueur.dart` :

```dart
class Joueur {
  final String nom;
  int vies;
  int score;

  Joueur(this.nom, {this.vies = 3, this.score = 0});

  bool get estVivant => vies > 0;

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  void gagnerPoints(int points) {
    if (points < 0) {
      throw ArgumentError('Les points gagnés ne peuvent pas être négatifs.');
    }
    score += points;
  }
}
```

### 16.32.3 — Le fichier de test

`test/joueur_test.dart` :

```dart
import 'package:mon_jeu/joueur.dart';
import 'package:test/test.dart';

void main() {
  test('un joueur neuf a 3 vies et 0 point', () {
    final heros = Joueur('Alex');

    expect(heros.vies, 3);
    expect(heros.score, 0);
    expect(heros.estVivant, isTrue);
  });

  test('perdreUneVie retire exactement une vie', () {
    final heros = Joueur('Alex');

    heros.perdreUneVie();

    expect(heros.vies, 2);
  });

  test('les vies ne descendent jamais sous zéro', () {
    final heros = Joueur('Alex', vies: 1);

    heros.perdreUneVie();
    heros.perdreUneVie();
    heros.perdreUneVie();

    expect(heros.vies, 0);
    expect(heros.estVivant, isFalse);
  });

  test('gagnerPoints refuse une valeur négative', () {
    final heros = Joueur('Alex');

    expect(() => heros.gagnerPoints(-10), throwsArgumentError);
  });
}
```

### 16.32.4 — Anatomie d'un test

```text
  test('description lisible', () {
       ▲                      ▲
       │                      └── la fonction qui contient la vérification
       └── ce qui s'affiche en cas d'échec


  expect(valeurObtenue, valeurAttendue);
         ▲              ▲
         │              └── ce que vous exigez
         └── ce que le code a produit
```

L'ordre des arguments d'`expect` compte pour la lisibilité des messages d'erreur : **obtenu d'abord, attendu ensuite**.

### 16.32.5 — Les « matchers » utiles

| Matcher | Vérifie que |
| --- | --- |
| `expect(x, 3)` | `x` vaut 3. |
| `expect(x, isTrue)` | `x` vaut `true`. |
| `expect(x, isFalse)` | `x` vaut `false`. |
| `expect(x, isNull)` | `x` vaut `null`. |
| `expect(x, isNotNull)` | `x` ne vaut pas `null`. |
| `expect(liste, isEmpty)` | La collection est vide. |
| `expect(liste, hasLength(3))` | La collection a 3 éléments. |
| `expect(liste, contains('potion'))` | La collection contient cet élément. |
| `expect(x, greaterThan(10))` | `x` est supérieur à 10. |
| `expect(() => f(), throwsArgumentError)` | L'appel lève une `ArgumentError`. |
| `expect(() => f(), throwsA(isA<RangeError>()))` | L'appel lève ce type précis. |

### 16.32.6 — Grouper les tests

```dart
import 'package:mon_jeu/joueur.dart';
import 'package:test/test.dart';

void main() {
  group('Joueur — vies', () {
    late Joueur heros;

    setUp(() {
      heros = Joueur('Alex');
    });

    test('commence à 3', () {
      expect(heros.vies, 3);
    });

    test('descend à 2 après un coup', () {
      heros.perdreUneVie();
      expect(heros.vies, 2);
    });
  });

  group('Joueur — score', () {
    test('augmente correctement', () {
      final heros = Joueur('Alex');
      heros.gagnerPoints(150);
      expect(heros.score, 150);
    });
  });
}
```

`setUp` s'exécute **avant chaque** `test` du groupe. Chaque test repart donc d'un joueur neuf : les tests ne s'influencent jamais entre eux.

> **Remarque :** `late` est ici parfaitement adapté (chapitre 12) : `heros` sera initialisé par `setUp` avant tout usage.

---

## 16.33 — `dart test`

```bash
dart test
```

**Résultat :**

```text
00:01 +4: All tests passed!
```

Lecture : `+4` signifie quatre tests réussis.

### 16.33.1 — Quand un test échoue

Introduisons un bug dans `perdreUneVie` :

```dart
void perdreUneVie() {
  if (vies > 0) vies -= 2;
}
```

```bash
dart test
```

**Résultat :**

```text
00:01 +1 -1: perdreUneVie retire exactement une vie [E]

  Expected: <2>
    Actual: <1>

  package:test_api                        expect
  test/joueur_test.dart 16:5              main.<fn>

00:01 +3 -1: Some tests failed.
```

Le rapport donne tout : le nom du test, la valeur attendue, la valeur obtenue, le fichier et la ligne. Corriger prend quelques secondes.

### 16.33.2 — Les options utiles

```bash
dart test test/joueur_test.dart
```

Ne lance qu'un fichier.

```bash
dart test --name "vies"
```

Ne lance que les tests dont le nom contient « vies ».

```bash
dart test -r expanded
```

Affiche le détail de chaque test, un par ligne.

**Résultat :**

```text
00:00 +0: Joueur — vies commence à 3
00:00 +1: Joueur — vies descend à 2 après un coup
00:00 +2: Joueur — score augmente correctement
00:00 +3: All tests passed!
```

```bash
dart test --coverage=coverage
```

Mesure la couverture : quelle proportion de votre code est réellement exécutée par les tests.

### 16.33.3 — Les erreurs de démarrage

| Message | Cause | Correction |
| --- | --- | --- |
| `No tests were found` | Le fichier ne se termine pas par `_test.dart`, ou n'est pas dans `test/`. | Renommer, déplacer. |
| `Couldn't resolve the package 'test'` | `test` n'est pas dans le pubspec. | `dart pub add dev:test` |
| `Error: Not found: 'package:mon_jeu/joueur.dart'` | Nom de package erroné dans l'import. | Vérifier `name:` du pubspec. |

### 16.33.4 — La bonne habitude

```bash
dart format .
dart analyze
dart test
```

Ces trois commandes, dans cet ordre, avant chaque `git commit`. C'est le rituel standard d'un développeur Dart.

---

## 16.34 — Le `.gitignore` d'un projet Dart

Git enregistre l'historique de vos fichiers. Certains fichiers n'ont rien à y faire : ils sont générés, volumineux, ou personnels.

Voici le `.gitignore` produit par `dart create`, commenté :

```text
# Fichiers générés par les outils Dart et pub
.dart_tool/
.packages
build/

# Exécutables compilés
*.exe
mon_jeu

# Réglages personnels de l'éditeur
.idea/
*.iml
.vscode/

# Fichiers système
.DS_Store
Thumbs.db
```

Ligne par ligne :

| Motif | Pourquoi l'ignorer |
| --- | --- |
| `.dart_tool/` | Entièrement régénérable par `dart pub get`. |
| `.packages` | Ancien fichier de configuration, obsolète. |
| `build/` | Sorties de compilation. |
| `*.exe`, `mon_jeu` | Binaires de plusieurs Mo, régénérables. |
| `.idea/`, `.vscode/` | Réglages propres à votre machine. |
| `.DS_Store`, `Thumbs.db` | Fichiers créés par macOS et Windows. |

### 16.34.1 — Le cas `pubspec.lock`

Comme vu en 16.17.1 :

```text
  Application (console, Flutter) : pubspec.lock EST versionné
  Bibliothèque (package publié)  : pubspec.lock est ignoré
```

Pour un package, on ajoute donc :

```text
pubspec.lock
```

### 16.34.2 — Ce qu'il ne faut jamais ignorer

| Fichier | Raison |
| --- | --- |
| `pubspec.yaml` | Sans lui, le projet n'existe pas. |
| `analysis_options.yaml` | Les règles de qualité doivent être partagées. |
| `lib/`, `bin/`, `test/` | C'est votre travail. |
| `README.md` | La porte d'entrée du projet. |

### 16.34.3 — Ajouts fréquents

```text
# Secrets : jamais dans Git
.env
*.key
config/secrets.yaml

# Rapports de couverture
coverage/

# Sauvegardes de parties générées par le jeu
saves/
*.save
```

Le premier bloc est le plus important. Une clé d'API publiée par erreur dans un dépôt reste dans l'historique même après suppression.

> **À retenir :** on versionne ce qu'un humain a écrit. On ignore ce qu'une machine peut reproduire.

---

## 16.35 — Organisation type d'un projet propre

Voici la structure vers laquelle tendre pour le fil rouge, une fois toutes les notions du chapitre appliquées.

```text
  mon_jeu/
  │
  ├── bin/
  │   └── mon_jeu.dart              main() : lance la partie, rien d'autre
  │
  ├── lib/
  │   ├── mon_jeu.dart              barrel : les export publics
  │   └── src/
  │       ├── personnages/
  │       │   ├── personnage.dart   classe abstraite commune
  │       │   ├── joueur.dart
  │       │   ├── ennemi.dart
  │       │   └── boss.dart
  │       ├── objets/
  │       │   ├── objet.dart
  │       │   ├── arme.dart
  │       │   ├── potion.dart
  │       │   └── inventaire.dart
  │       ├── monde/
  │       │   ├── niveau.dart
  │       │   └── salle.dart
  │       └── moteur/
  │           ├── combat.dart
  │           └── des.dart
  │
  ├── test/
  │   ├── joueur_test.dart
  │   ├── ennemi_test.dart
  │   ├── inventaire_test.dart
  │   └── combat_test.dart
  │
  ├── .dart_tool/                   généré, ignoré par Git
  ├── .gitignore
  ├── analysis_options.yaml
  ├── pubspec.yaml
  ├── pubspec.lock
  ├── README.md
  └── CHANGELOG.md
```

### 16.35.1 — Les sept principes

1. **`bin/` reste mince.** Le `main` orchestre, il ne calcule rien. Toute la logique est dans `lib/`.
2. **Une classe publique par fichier**, nommé d'après elle en snake_case.
3. **Regrouper par domaine**, pas par nature technique.
4. **Un barrel** pour offrir une porte d'entrée unique.
5. **Un test par fichier de code** important, nommé `<fichier>_test.dart`.
6. **`dart format` + `dart analyze` + `dart test`** avant chaque commit.
7. **Rien de généré dans Git.**

### 16.35.2 — Le `bin/mon_jeu.dart` d'un projet propre

```dart
import 'package:mon_jeu/mon_jeu.dart';

void main(List<String> arguments) {
  final nomJoueur = arguments.isNotEmpty ? arguments[0] : 'Héros';

  final partie = Partie(nomJoueur: nomJoueur);
  partie.lancer();
}
```

Six lignes. Tout le reste vit dans `lib/`, donc tout le reste est testable sans lancer le jeu. C'est le critère qui distingue un projet organisé d'un projet accumulé.

### 16.35.3 — Quand appliquer tout cela

| Taille du projet | Structure adaptée |
| --- | --- |
| 1 à 3 fichiers | `bin/` + `lib/` à plat. Pas de barrel. |
| 4 à 15 fichiers | `lib/` avec sous-dossiers. Barrel utile. |
| Plus de 15 fichiers | `lib/src/` + barrel + tests systématiques. |

N'imposez pas une arborescence de dix dossiers à un exercice de trois classes. L'organisation doit servir le projet, pas l'inverse.

---

## 16.36 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Could not find a file named "pubspec.yaml"` | Vous lancez `dart run` depuis un sous-dossier (`bin/`) ou hors du projet. | Placez-vous à la racine du projet : `cd mon_jeu`. |
| `Error on line 8, column 1: Expected a key while parsing a block mapping.` | Indentation YAML cassée : une dépendance n'est pas décalée de deux espaces sous `dependencies:`. | Réindentez avec exactement deux espaces par niveau. |
| `Tab characters are not allowed as indentation.` | Une tabulation s'est glissée dans le `pubspec.yaml`. | Remplacez la tabulation par des espaces. Réglez l'éditeur sur « insérer des espaces ». |
| `Couldn't resolve the package 'http'` | Le package est écrit dans `pubspec.yaml` mais n'a jamais été téléchargé. | Lancez `dart pub get` (ou utilisez `dart pub add` qui le fait seul). |
| `Target of URI doesn't exist: 'package:mon_jeu/lib/joueur.dart'` | `lib/` a été répété dans un import `package:`. | Écrivez `package:mon_jeu/joueur.dart` : `lib/` est implicite. |
| `Target of URI doesn't exist: 'joueur.dart'` depuis `bin/` | Import relatif utilisé pour franchir la frontière `bin/` → `lib/`. | Utilisez `import 'package:mon_jeu/joueur.dart';`. |
| `"MonJeu" is not a valid Dart project name.` | Nom de package en majuscules, avec tirets, espaces ou accents. | Utilisez le snake_case : `mon_jeu`. |
| `The getter '_pointsDeVie' isn't defined for the class 'Joueur'.` | Accès à un membre privé depuis un **autre fichier**. | Exposez une méthode ou un getter public dans `Joueur`. |
| `'Dragon' is imported from both ...` | Deux fichiers importés définissent le même nom. | Ajoutez un préfixe `as`, ou filtrez avec `hide`. |
| `No tests were found` | Le fichier de test ne s'appelle pas `*_test.dart`, ou n'est pas dans `test/`. | Renommez en `joueur_test.dart` et placez-le dans `test/`. |
| `The imported package 'test' isn't a dependency of the importing package.` | `test` a été importé depuis `lib/` alors que c'est une `dev_dependency`. | N'importez `package:test` que dans `test/`. |
| `Because mon_jeu requires SDK version ^3.5.0, version solving failed.` | Votre SDK est plus ancien que la contrainte `environment`. | Mettez à jour Dart, ou abaissez la contrainte en connaissance de cause. |
| `bash: dart: command not found` | Le dossier `dart-sdk/bin` n'est pas dans le `PATH`, ou le terminal est antérieur à l'installation. | Rouvrez un terminal ; sinon ajoutez le dossier au `PATH`. |
| `dart run` lance le mauvais fichier | Plusieurs fichiers dans `bin/` ; sans argument, Dart prend celui qui porte le nom du package. | Précisez : `dart run :generer_niveaux`. |
| Le formatage change tout le fichier d'un coup | Le fichier n'avait jamais été passé à `dart format`. | Formatez une fois, committez ce seul changement, puis activez « Format On Save ». |
| Un binaire de 8 Mo apparaît dans Git | L'exécutable produit par `dart compile exe` n'est pas ignoré. | Ajoutez son nom (et `*.exe`) au `.gitignore`. |

---

## 16.37 — Résumé du chapitre

### 16.37.1 — Les commandes

| Commande | Rôle |
| --- | --- |
| `dart --version` | Vérifier que le SDK est installé et lequel. |
| `dart create mon_jeu` | Créer un projet complet (modèle `console` par défaut). |
| `dart create -t package moteur` | Créer une bibliothèque réutilisable, sans `main`. |
| `dart run` | Exécuter `bin/<nom_du_package>.dart`. |
| `dart run :outil` | Exécuter `bin/outil.dart`. |
| `dart analyze` | Détecter erreurs, avertissements et infos sans exécuter. |
| `dart format .` | Appliquer la mise en forme officielle à tout le projet. |
| `dart compile exe bin/mon_jeu.dart -o mon_jeu` | Produire un exécutable autonome. |
| `dart pub add http` | Ajouter une dépendance et l'installer. |
| `dart pub add dev:test` | Ajouter une dépendance de développement. |
| `dart pub remove http` | Retirer une dépendance. |
| `dart pub get` | Installer les dépendances déclarées dans `pubspec.yaml`. |
| `dart pub upgrade` | Monter les versions dans les limites des contraintes. |
| `dart pub outdated` | Lister les mises à jour disponibles. |
| `dart test` | Lancer tous les fichiers `test/*_test.dart`. |
| `dart test --name "vies"` | Lancer les tests dont le nom contient « vies ». |
| `dart doc` | Générer la documentation HTML du projet. |

### 16.37.2 — Les dossiers et les fichiers

| Élément | À retenir |
| --- | --- |
| `bin/` | Points d'entrée. Contient `main`. Reste mince. |
| `lib/` | Code réutilisable. Cible du préfixe `package:`. |
| `lib/src/` | Détails internes du package, non destinés à l'extérieur. |
| `test/` | Tests. Fichiers en `_test.dart`. |
| `.dart_tool/` | Généré. Ne pas toucher, ne pas versionner, supprimable. |
| `pubspec.yaml` | Carte d'identité : `name`, `description`, `version`, `environment`, dépendances. |
| `pubspec.lock` | Versions exactes installées. Versionné pour une application. |
| `analysis_options.yaml` | Règles de qualité (lints). |
| `.gitignore` | Ce que Git doit ignorer : généré, binaire, personnel. |

### 16.37.3 — Le langage

| Notion | À retenir |
| --- | --- |
| `import 'dart:math';` | Bibliothèque du SDK, aucune installation. |
| `import 'package:mon_jeu/joueur.dart';` | Package externe ou votre `lib/`. `lib/` n'est pas répété. |
| `import 'arme.dart';` | Chemin relatif, entre fichiers d'un même dossier logique. |
| `show` | Ne rendre visibles que les noms listés. |
| `hide` | Rendre visible tout sauf les noms listés. |
| `as` | Donner un préfixe : `math.pi`, `http.get`. |
| `part` / `part of` | Découpe historique. Réservée au code généré. |
| `export` | Réexporter : sert à construire un barrel `lib/mon_jeu.dart`. |
| `_membre` | Privé au **fichier**, pas à la classe. |
| `^1.2.2` | `>=1.2.2 <2.0.0`. Réglage par défaut. |
| `dependencies` | Nécessaire au fonctionnement, livré avec le programme. |
| `dev_dependencies` | Nécessaire au développement seulement. |

---

## 16.38 — Exercices

Faites-les dans l'ordre : ils construisent progressivement le même projet `mon_jeu`.

### Exercice 1 — Créer le projet (facile)

Créez un projet Dart nommé `mon_jeu` avec le modèle `console`. Entrez dedans, lancez-le, et vérifiez qu'il affiche le message d'origine. Donnez la liste des commandes utilisées et l'arborescence obtenue.

### Exercice 2 — Le rituel des trois commandes (facile)

Remplacez le contenu de `bin/mon_jeu.dart` par un programme volontairement mal indenté qui affiche trois lignes de démarrage du jeu, et qui déclare une variable locale jamais utilisée. Puis :

1. lancez `dart analyze` et notez ce qui est signalé ;
2. lancez `dart format .` ;
3. corrigez ce que l'analyse reproche ;
4. relancez `dart analyze` jusqu'à `No issues found!`.

### Exercice 3 — Sortir la classe `Joueur` dans `lib/` (facile)

Créez `lib/joueur.dart` contenant une classe `Joueur` avec :

- un champ `final String nom` ;
- un champ `int vies` valant 3 par défaut ;
- un champ `int score` valant 0 par défaut ;
- un getter `estVivant` ;
- une méthode `perdreUneVie()` qui ne descend jamais sous zéro ;
- un `toString()` lisible.

Utilisez-la depuis `bin/mon_jeu.dart` avec le bon import.

### Exercice 4 — Deux fichiers qui se parlent (facile)

Ajoutez `lib/arme.dart` avec une classe `Arme` (`final String nom`, `final int puissance`, constructeur `const`, `toString()`).

Modifiez `Joueur` pour qu'il possède un champ `Arme? arme` et un getter `degats` valant `5 + puissance de l'arme` (ou 5 sans arme). Attention : l'import entre `joueur.dart` et `arme.dart` doit être **relatif**.

### Exercice 5 — Réparer un `pubspec.yaml` (moyen)

Ce fichier refuse de fonctionner. Trouvez les quatre erreurs et réécrivez-le correctement.

```text
name: Mon-Jeu
description Un jeu de rôle en mode texte.
version: 1.0

environment:
sdk: ^3.5.0

dependencies:
	collection: ^1.18.0
```

### Exercice 6 — Ajouter un package (moyen)

Ajoutez le package `collection` à votre projet en ligne de commande. Puis, dans `bin/mon_jeu.dart`, utilisez sa fonction `maxBy` pour trouver l'ennemi le plus résistant d'une liste de trois ennemis.

Indication : `collection` fournit `maxBy` via l'extension sur les itérables ; sa signature est `maxBy(Iterable, valeur)` dans la fonction libre `maxBy<T, K extends Comparable<K>>(Iterable<T> values, K Function(T) keyOf)`.

### Exercice 7 — Résoudre un conflit de noms (moyen)

Créez deux fichiers définissant chacun une classe `Dragon` :

- `lib/monstres/dragon.dart` : `Dragon(int puissanceDeFeu)` avec `decrire()` ;
- `lib/boss/dragon.dart` : `Dragon(int phases)` avec `decrire()`.

Écrivez un `bin/mon_jeu.dart` qui utilise **les deux** dans le même `main`, et affiche leurs deux descriptions.

### Exercice 8 — Le fichier barrel (moyen)

Réorganisez `lib/` ainsi :

```text
  lib/
    src/
      personnages/joueur.dart
      personnages/ennemi.dart
      objets/arme.dart
```

puis créez le barrel `lib/mon_jeu.dart` qui exporte les trois. `bin/mon_jeu.dart` ne doit plus contenir qu'un seul `import`.

### Exercice 9 — Écrire et lancer des tests (moyen)

Créez `lib/inventaire.dart` avec une classe `Inventaire` :

- une liste interne privée `_objets` de `String` ;
- un getter `objets` en lecture seule ;
- `ajouter(String objet)` ;
- `retirer(String objet)` qui renvoie `true` si l'objet était présent ;
- un getter `estVide`.

Écrivez ensuite `test/inventaire_test.dart` avec au moins quatre tests, groupés, utilisant `setUp`. Lancez `dart test`.

### Exercice 10 — Le projet complet (difficile)

Assemblez tout :

1. l'arborescence finale conforme à la section 16.35 ;
2. un `pubspec.yaml` correct, avec `test` en `dev_dependencies` ;
3. un `analysis_options.yaml` basé sur `recommended`, avec `avoid_print` désactivé et `prefer_final_locals` activé ;
4. un `.gitignore` complet, ignorant aussi l'exécutable `mon_jeu` ;
5. le rituel `dart format .`, `dart analyze`, `dart test` sans aucune remarque ;
6. la compilation en exécutable autonome et son lancement.

Donnez l'arborescence, les fichiers de configuration, le `bin/mon_jeu.dart` final, et la suite de commandes.

---

## 16.39 — Corrections des exercices

### Correction 1

```bash
dart create mon_jeu
cd mon_jeu
dart run
```

**Arborescence obtenue :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
├── lib/
│   └── mon_jeu.dart
├── test/
│   └── mon_jeu_test.dart
├── .dart_tool/
├── .gitignore
├── analysis_options.yaml
├── CHANGELOG.md
├── pubspec.lock
├── pubspec.yaml
└── README.md
```

**Résultat :**

```text
Building package executable...
Built mon_jeu:mon_jeu.
Hello world: 42!
```

**Explication :** `dart create` génère la structure conventionnelle **et** lance `dart pub get`, ce qui explique la présence immédiate de `.dart_tool/` et de `pubspec.lock`. `dart run` sans argument choisit `bin/mon_jeu.dart` parce que ce fichier porte le nom du package déclaré dans `pubspec.yaml`. Il faut impérativement avoir fait `cd mon_jeu` avant : la commande cherche un `pubspec.yaml` dans le dossier courant.

---

### Correction 2

Version fautive de `bin/mon_jeu.dart` :

```dart
void main(){
  int niveauActuel=1;
print('=== MON JEU ===');
     print('Chargement du niveau...');
  print( 'Le joueur entre dans la caverne.' );
}
```

```bash
dart analyze
```

**Résultat :**

```text
Analyzing mon_jeu...

warning • bin/mon_jeu.dart:2:7 • The value of the local variable 'niveauActuel'
          isn't used. • unused_local_variable

1 issue found.
```

```bash
dart format .
```

**Résultat :**

```text
Formatted bin/mon_jeu.dart
Formatted 1 file (1 changed) in 0.08 seconds.
```

Version finale :

```dart
void main() {
  const niveauActuel = 1;
  print('=== MON JEU ===');
  print('Chargement du niveau $niveauActuel...');
  print('Le joueur entre dans la caverne.');
}
```

```bash
dart analyze
```

**Résultat :**

```text
Analyzing mon_jeu...
No issues found!
```

**Explication :** `dart format` corrige la présentation (accolades, espaces, indentation) mais **jamais** la logique : la variable inutilisée subsiste après formatage. Ce sont deux outils complémentaires — l'un met en forme, l'autre juge. La correction retenue utilise la variable plutôt que de la supprimer, ce qui rend le message plus informatif.

---

### Correction 3

**Arborescence :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
└── lib/
    └── joueur.dart
```

`lib/joueur.dart` :

```dart
class Joueur {
  final String nom;
  int vies;
  int score;

  Joueur(this.nom, {this.vies = 3, this.score = 0});

  bool get estVivant => vies > 0;

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  @override
  String toString() => '$nom (vies: $vies, score: $score)';
}
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/joueur.dart';

void main() {
  final heros = Joueur('Alex');
  print(heros);

  heros.perdreUneVie();
  heros.perdreUneVie();
  heros.perdreUneVie();
  print(heros);
  print('Encore vivant ? ${heros.estVivant}');
}
```

**Résultat :**

```text
Alex (vies: 3, score: 0)
Alex (vies: 0, score: 0)
Encore vivant ? false
```

**Explication :** le fichier `bin/mon_jeu.dart` est **hors** de `lib/`, il doit donc employer la forme `package:`. Le chemin s'écrit `package:mon_jeu/joueur.dart` : `mon_jeu` est la valeur du champ `name` du `pubspec.yaml`, et `joueur.dart` est le chemin **à partir de `lib/`**, jamais `lib/joueur.dart`. La garde `if (vies > 0)` dans `perdreUneVie` empêche les vies de devenir négatives, ce que montre le troisième appel.

---

### Correction 4

**Arborescence :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
└── lib/
    ├── arme.dart
    └── joueur.dart
```

`lib/arme.dart` :

```dart
class Arme {
  final String nom;
  final int puissance;

  const Arme(this.nom, this.puissance);

  @override
  String toString() => '$nom (+$puissance)';
}
```

`lib/joueur.dart` :

```dart
import 'arme.dart';

class Joueur {
  final String nom;
  int vies;
  int score;
  Arme? arme;

  Joueur(this.nom, {this.vies = 3, this.score = 0, this.arme});

  int get degats => 5 + (arme?.puissance ?? 0);

  bool get estVivant => vies > 0;

  void perdreUneVie() {
    if (vies > 0) vies--;
  }

  @override
  String toString() => '$nom (vies: $vies, score: $score)';
}
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/arme.dart';
import 'package:mon_jeu/joueur.dart';

void main() {
  final mainsNues = Joueur('Alex');
  print('${mainsNues.nom} sans arme : ${mainsNues.degats} dégâts');

  const epee = Arme('Épée de feu', 12);
  final arme = Joueur('Alex', arme: epee);
  print('${arme.nom} avec $epee : ${arme.degats} dégâts');
}
```

**Résultat :**

```text
Alex sans arme : 5 dégâts
Alex avec Épée de feu (+12) : 17 dégâts
```

**Explication :** deux frontières différentes cohabitent ici. Entre `lib/joueur.dart` et `lib/arme.dart`, les deux fichiers sont dans le même dossier : l'import relatif `'arme.dart'` est la forme correcte. Entre `bin/` et `lib/`, il faut la forme `package:`. Le getter `degats` combine deux opérateurs du chapitre 12 : `?.` pour ne pas planter si `arme` vaut `null`, et `??` pour fournir la valeur de repli `0`.

---

### Correction 5

Les quatre erreurs :

```text
1. name: Mon-Jeu          -> majuscules et tiret interdits
2. description ...        -> deux-points manquant après "description"
3. version: 1.0           -> il faut TROIS nombres : 1.0.0
4. sdk / collection       -> indentation absente (ligne 6) puis tabulation (ligne 9)
```

`pubspec.yaml` corrigé :

```yaml
name: mon_jeu
description: Un jeu de rôle en mode texte.
version: 1.0.0
publish_to: 'none'

environment:
  sdk: ^3.5.0

dependencies:
  collection: ^1.18.0
```

```bash
dart pub get
```

**Résultat :**

```text
Resolving dependencies...
+ collection 1.18.0
Changed 1 dependency!
Got dependencies!
```

**Explication :** YAML est un format à indentation stricte. `sdk:` doit être décalé de deux espaces sous `environment:` pour en devenir l'enfant ; sans ce décalage, l'analyseur voit deux clés de même niveau et rejette le fichier. La tabulation devant `collection` est le piège le plus vicieux, car elle est invisible : YAML l'interdit formellement comme caractère d'indentation. Enfin, `version` suit le versionnage sémantique et exige trois nombres, et `name` obéit aux règles du snake_case.

---

### Correction 6

```bash
dart pub add collection
```

**Résultat :**

```text
Resolving dependencies...
+ collection 1.18.0
Changed 1 dependency!
```

`pubspec.yaml` (extrait) :

```yaml
dependencies:
  collection: ^1.18.0
```

`lib/ennemi.dart` :

```dart
class Ennemi {
  final String nom;
  int pointsDeVie;

  Ennemi(this.nom, this.pointsDeVie);

  @override
  String toString() => '$nom ($pointsDeVie PV)';
}
```

`bin/mon_jeu.dart` :

```dart
import 'package:collection/collection.dart';
import 'package:mon_jeu/ennemi.dart';

void main() {
  final ennemis = [
    Ennemi('Gobelin', 30),
    Ennemi('Troll', 120),
    Ennemi('Rat géant', 12),
  ];

  final plusResistant = maxBy(ennemis, (Ennemi e) => e.pointsDeVie);
  print('Ennemi le plus résistant : $plusResistant');

  final total = ennemis.fold<int>(0, (somme, e) => somme + e.pointsDeVie);
  print('Points de vie cumulés    : $total');
}
```

**Résultat :**

```text
Ennemi le plus résistant : Troll (120 PV)
Points de vie cumulés    : 162
```

**Explication :** `dart pub add collection` a fait trois choses en une commande : écrire `collection: ^1.18.0` dans `dependencies`, télécharger le package, et mettre à jour `pubspec.lock`. Il n'y a donc pas de `dart pub get` à lancer ensuite. `collection` va dans `dependencies` et non dans `dev_dependencies` parce que le programme en a besoin **pour fonctionner** : sans lui, `maxBy` n'existe pas et le jeu ne démarre pas.

---

### Correction 7

**Arborescence :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
└── lib/
    ├── boss/
    │   └── dragon.dart
    └── monstres/
        └── dragon.dart
```

`lib/monstres/dragon.dart` :

```dart
class Dragon {
  final int puissanceDeFeu;

  const Dragon(this.puissanceDeFeu);

  String decrire() => 'Dragon des montagnes (feu: $puissanceDeFeu)';
}
```

`lib/boss/dragon.dart` :

```dart
class Dragon {
  final int phases;

  const Dragon(this.phases);

  String decrire() => 'Dragon final ($phases phases)';
}
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/boss/dragon.dart' as boss;
import 'package:mon_jeu/monstres/dragon.dart' as monstres;

void main() {
  const petit = monstres.Dragon(50);
  const grand = boss.Dragon(3);

  print(petit.decrire());
  print(grand.decrire());
}
```

**Résultat :**

```text
Dragon des montagnes (feu: 50)
Dragon final (3 phases)
```

**Explication :** sans préfixe, Dart signale `'Dragon' is imported from both ...` et refuse de compiler, car il n'a aucun moyen de savoir lequel des deux `Dragon` vous désignez. `as` résout le conflit sans rien sacrifier : les deux classes restent utilisables, chacune sous son préfixe. La solution `hide` aurait fonctionné aussi, mais en vous privant de l'une des deux classes — inacceptable ici puisque l'énoncé exige les deux. Notez que `boss` et `monstres` ne sont pas des variables : ils n'ont de sens qu'accolés à un nom.

---

### Correction 8

**Arborescence :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
└── lib/
    ├── mon_jeu.dart          <- le barrel
    └── src/
        ├── objets/
        │   └── arme.dart
        └── personnages/
            ├── ennemi.dart
            └── joueur.dart
```

`lib/src/objets/arme.dart` :

```dart
class Arme {
  final String nom;
  final int puissance;

  const Arme(this.nom, this.puissance);

  @override
  String toString() => '$nom (+$puissance)';
}
```

`lib/src/personnages/joueur.dart` :

```dart
import '../objets/arme.dart';

class Joueur {
  final String nom;
  int vies;
  Arme? arme;

  Joueur(this.nom, {this.vies = 3, this.arme});

  int get degats => 5 + (arme?.puissance ?? 0);

  @override
  String toString() => '$nom (vies: $vies)';
}
```

`lib/src/personnages/ennemi.dart` :

```dart
class Ennemi {
  final String nom;
  int pointsDeVie;

  Ennemi(this.nom, this.pointsDeVie);

  bool get estVaincu => pointsDeVie <= 0;

  void subirDegats(int montant) {
    pointsDeVie -= montant;
    if (pointsDeVie < 0) pointsDeVie = 0;
  }

  @override
  String toString() => '$nom ($pointsDeVie PV)';
}
```

`lib/mon_jeu.dart` (le barrel) :

```dart
export 'src/objets/arme.dart';
export 'src/personnages/ennemi.dart';
export 'src/personnages/joueur.dart';
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/mon_jeu.dart';

void main() {
  const epee = Arme('Épée de feu', 12);
  final heros = Joueur('Alex', arme: epee);
  final gobelin = Ennemi('Gobelin', 30);

  print('$heros affronte $gobelin');

  while (!gobelin.estVaincu) {
    gobelin.subirDegats(heros.degats);
    print('Coup porté ! $gobelin');
  }

  print('${gobelin.nom} est vaincu.');
}
```

**Résultat :**

```text
Alex (vies: 3) affronte Gobelin (30 PV)
Coup porté ! Gobelin (13 PV)
Coup porté ! Gobelin (0 PV)
Gobelin est vaincu.
```

**Explication :** le barrel `lib/mon_jeu.dart` ne contient aucune classe : il ne fait que réexporter. Les trois classes deviennent accessibles par un import unique, et l'arborescence interne de `lib/src/` peut être réorganisée sans casser une seule ligne dans `bin/` ou dans les tests. Remarquez l'import `'../objets/arme.dart'` dans `joueur.dart` : les deux fichiers sont dans `lib/`, donc chemin relatif, et `..` remonte de `personnages/` vers `src/`.

---

### Correction 9

`lib/inventaire.dart` :

```dart
class Inventaire {
  final List<String> _objets = [];

  List<String> get objets => List.unmodifiable(_objets);

  bool get estVide => _objets.isEmpty;

  int get taille => _objets.length;

  void ajouter(String objet) {
    _objets.add(objet);
  }

  bool retirer(String objet) {
    return _objets.remove(objet);
  }
}
```

`test/inventaire_test.dart` :

```dart
import 'package:mon_jeu/inventaire.dart';
import 'package:test/test.dart';

void main() {
  group('Inventaire — état initial', () {
    late Inventaire sac;

    setUp(() {
      sac = Inventaire();
    });

    test('un sac neuf est vide', () {
      expect(sac.estVide, isTrue);
      expect(sac.objets, isEmpty);
    });

    test('ajouter un objet remplit le sac', () {
      sac.ajouter('potion');

      expect(sac.estVide, isFalse);
      expect(sac.taille, 1);
      expect(sac.objets, contains('potion'));
    });
  });

  group('Inventaire — retrait', () {
    late Inventaire sac;

    setUp(() {
      sac = Inventaire()
        ..ajouter('potion')
        ..ajouter('clé');
    });

    test('retirer un objet présent renvoie true', () {
      expect(sac.retirer('potion'), isTrue);
      expect(sac.taille, 1);
      expect(sac.objets, isNot(contains('potion')));
    });

    test('retirer un objet absent renvoie false', () {
      expect(sac.retirer('épée'), isFalse);
      expect(sac.taille, 2);
    });

    test('la liste renvoyée est en lecture seule', () {
      expect(() => sac.objets.add('triche'), throwsUnsupportedError);
    });
  });
}
```

```bash
dart test
```

**Résultat :**

```text
00:01 +5: All tests passed!
```

Avec le rapport détaillé :

```bash
dart test -r expanded
```

**Résultat :**

```text
00:00 +0: Inventaire — état initial un sac neuf est vide
00:00 +1: Inventaire — état initial ajouter un objet remplit le sac
00:00 +2: Inventaire — retrait retirer un objet présent renvoie true
00:00 +3: Inventaire — retrait retirer un objet absent renvoie false
00:00 +4: Inventaire — retrait la liste renvoyée est en lecture seule
00:00 +5: All tests passed!
```

**Explication :** le champ `_objets` est privé au fichier `lib/inventaire.dart` : le test, qui vit dans un autre fichier, ne peut pas y toucher. C'est voulu — un test doit vérifier le **comportement public**, pas les détails internes. Le getter `objets` renvoie `List.unmodifiable(_objets)`, une vue en lecture seule : le dernier test vérifie précisément que toute tentative de modification depuis l'extérieur lève une `UnsupportedError`. `setUp` s'exécute avant chaque `test` du groupe, ce qui garantit que les tests ne se contaminent pas entre eux.

---

### Correction 10

**Arborescence finale :**

```text
mon_jeu/
├── bin/
│   └── mon_jeu.dart
├── lib/
│   ├── mon_jeu.dart
│   └── src/
│       ├── moteur/
│       │   └── partie.dart
│       ├── objets/
│       │   ├── arme.dart
│       │   └── inventaire.dart
│       └── personnages/
│           ├── ennemi.dart
│           └── joueur.dart
├── test/
│   ├── inventaire_test.dart
│   └── joueur_test.dart
├── .dart_tool/
├── .gitignore
├── analysis_options.yaml
├── CHANGELOG.md
├── pubspec.lock
├── pubspec.yaml
└── README.md
```

`pubspec.yaml` :

```yaml
name: mon_jeu
description: Un jeu de rôle en mode texte, fil rouge de la formation Dart.
version: 1.0.0
publish_to: 'none'

environment:
  sdk: ^3.5.0

dependencies:
  collection: ^1.18.0

dev_dependencies:
  lints: ^4.0.0
  test: ^1.25.7
```

`analysis_options.yaml` :

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    prefer_final_locals: true
    avoid_print: false

analyzer:
  exclude:
    - '**/*.g.dart'
```

`.gitignore` :

```text
# Généré par les outils Dart
.dart_tool/
build/

# Exécutables compilés
mon_jeu
*.exe

# Réglages d'éditeur
.idea/
.vscode/
*.iml

# Fichiers système
.DS_Store
Thumbs.db

# Secrets
.env
```

`lib/src/moteur/partie.dart` :

```dart
import '../objets/arme.dart';
import '../personnages/ennemi.dart';
import '../personnages/joueur.dart';

class Partie {
  final Joueur heros;
  final List<Ennemi> ennemis;

  Partie({required String nomJoueur})
      : heros = Joueur(nomJoueur, arme: const Arme('Épée de feu', 12)),
        ennemis = [
          Ennemi('Gobelin', 30),
          Ennemi('Troll', 60),
        ];

  void lancer() {
    print('=== MON JEU ===');
    print('${heros.nom} entre dans la caverne.');

    for (final ennemi in ennemis) {
      while (!ennemi.estVaincu) {
        ennemi.subirDegats(heros.degats);
      }
      print('${ennemi.nom} est vaincu.');
    }

    print('Caverne nettoyée. Bravo, ${heros.nom} !');
  }
}
```

`lib/mon_jeu.dart` :

```dart
export 'src/moteur/partie.dart';
export 'src/objets/arme.dart';
export 'src/objets/inventaire.dart';
export 'src/personnages/ennemi.dart';
export 'src/personnages/joueur.dart';
```

`bin/mon_jeu.dart` :

```dart
import 'package:mon_jeu/mon_jeu.dart';

void main(List<String> arguments) {
  final nomJoueur = arguments.isNotEmpty ? arguments[0] : 'Héros';

  final partie = Partie(nomJoueur: nomJoueur);
  partie.lancer();
}
```

**Suite de commandes :**

```bash
dart pub get
dart format .
dart analyze
dart test
dart run bin/mon_jeu.dart Alex
dart compile exe bin/mon_jeu.dart -o mon_jeu
./mon_jeu Alex
```

**Résultat :**

```text
Resolving dependencies...
Got dependencies!

Formatted 8 files (0 changed) in 0.21 seconds.

Analyzing mon_jeu...
No issues found!

00:01 +7: All tests passed!

=== MON JEU ===
Alex entre dans la caverne.
Gobelin est vaincu.
Troll est vaincu.
Caverne nettoyée. Bravo, Alex !

Info: Compiling with sound null safety.
Generated: /home/alex/projets/mon_jeu/mon_jeu

=== MON JEU ===
Alex entre dans la caverne.
Gobelin est vaincu.
Troll est vaincu.
Caverne nettoyée. Bravo, Alex !
```

**Explication :** ce projet applique les sept principes de la section 16.35. Le `main` fait six lignes : il lit un argument et délègue tout à `Partie`, qui vit dans `lib/` et reste donc testable sans lancer le jeu. Le barrel `lib/mon_jeu.dart` masque entièrement l'arborescence de `lib/src/`. Dans `analysis_options.yaml`, `avoid_print: false` est cohérent avec un jeu console dont l'affichage passe par `print`, tandis que `prefer_final_locals: true` durcit la règle sur les variables locales. Le `.gitignore` ignore l'exécutable `mon_jeu` produit par `dart compile exe` : c'est un binaire de plusieurs mégaoctets, entièrement régénérable, qui n'a rien à faire dans un dépôt. Enfin, les deux dernières sorties sont identiques, ce qui confirme que l'exécutable compilé se comporte exactement comme `dart run` — à ceci près qu'il démarre instantanément et fonctionne sur une machine où Dart n'est pas installé.

---

## Et maintenant ?

Vous êtes sorti de DartPad. Vous savez créer un projet, le découper, y ajouter des packages, l'analyser, le formater, le tester et le compiler. Autrement dit : vous disposez enfin d'un vrai atelier de développement.

Il manque une chose à votre jeu : la **persistance**. Un joueur qui ferme le programme perd son score, son inventaire et sa progression. Pour sauvegarder une partie, ou pour dialoguer avec un serveur de classement, il faut savoir transformer un objet Dart en texte, et retransformer ce texte en objet.

C'est exactement le sujet du chapitre suivant : le format JSON, sa lecture, son écriture, et la modélisation de données qui va avec.

Rendez-vous au chapitre 17 : [17-PARTIE-1A—JSON-ET-MODÉLISATION-DE-DONNÉES.md](17-PARTIE-1A—JSON-ET-MODÉLISATION-DE-DONNÉES.md)
