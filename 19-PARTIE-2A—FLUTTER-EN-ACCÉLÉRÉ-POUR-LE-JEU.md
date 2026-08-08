# PARTIE 2A — DU DART AU JEU 2D
# CHAPITRE 19 — FLUTTER EN ACCÉLÉRÉ : CE QU'IL FAUT POUR FAIRE UN JEU

> **Niveau :** débutant en Flutter, intermédiaire en Dart
> **Durée estimée :** 8 h
> **Pré-requis :** chapitres 01 à 18 (Dart console complet : POO ch. 08 à 11, null safety ch. 12, exceptions ch. 13, asynchrone ch. 15, organisation d'un projet ch. 16)
> **Ce que vous saurez faire à la fin :** installer le SDK Flutter, créer un projet, comprendre un arbre de widgets, gérer un état avec `setState()`, dessiner sur un `Canvas` avec `CustomPainter` et animer un carré à l'écran avec un `Ticker`, en Flutter pur, sans aucun moteur de jeu.

> **Version vérifiée :** Flutter **3.44** (canal `stable`).
> **Date de vérification :** **8 août 2026**, sur `https://docs.flutter.dev/get-started/install`, `https://docs.flutter.dev/install/manual`, `https://docs.flutter.dev/release/release-notes` et `https://api.flutter.dev`.
> Flutter évolue vite. Si votre `flutter --version` affiche un numéro plus grand, ce n'est pas un problème : tout ce qui est enseigné dans ce chapitre fait partie du socle stable de Flutter et n'a pas changé depuis des années.

---

## 19.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer en une phrase la différence entre Dart, Flutter et Flame ;
- installer le SDK Flutter sur Windows, macOS ou Linux ;
- lire et corriger un rapport `flutter doctor` ;
- configurer VS Code avec l'extension Flutter ;
- choisir une cible d'exécution adaptée au développement d'un jeu ;
- créer un projet avec `flutter create` et nommer ses fichiers correctement ;
- décrire le rôle de chaque dossier d'un projet Flutter ;
- lancer un projet avec `flutter run` et utiliser le hot reload ;
- lire le `main.dart` par défaut ligne par ligne, sans zone d'ombre ;
- expliquer ce que fait `runApp()` ;
- définir ce qu'est un widget et pourquoi il est immuable ;
- dessiner mentalement un arbre de widgets ;
- utiliser `MaterialApp` et `Scaffold` ;
- écrire un `StatelessWidget` et un `StatefulWidget` ;
- expliquer pourquoi `setState()` provoque un redessin ;
- utiliser `initState()` et `dispose()` au bon moment ;
- assembler une interface avec `Container`, `Center`, `Column`, `Row`, `Stack`, `SizedBox` et `Padding` ;
- afficher du texte stylé avec `Text` et `TextStyle` ;
- réagir à un clic avec un bouton et avec `GestureDetector` ;
- décrire le système de coordonnées de l'écran ;
- dessiner des formes avec `CustomPaint`, `CustomPainter`, `Canvas` et `Paint` ;
- animer un objet avec un `Ticker` ou un `AnimationController` ;
- écrire un « carré qui bouge » complet en Flutter pur ;
- expliquer pourquoi Flame existe et ce qu'il apporte.

---

## 19.1 — Ce que ce chapitre couvre et ce qu'il ne couvre pas

Vous venez de terminer dix-huit chapitres de Dart pur. Vous savez écrire des classes, gérer des exceptions, manipuler des collections, modéliser un donjon complet en console. Tout cela reste vrai et tout cela va servir.

Il vous manque une chose : **des pixels**.

Ce chapitre est un **condensé**. Il n'a pas pour but de faire de vous un développeur d'applications Flutter. Il a un but beaucoup plus étroit et beaucoup plus précis :

> Vous donner exactement les briques Flutter nécessaires pour comprendre ce qui se passe quand, au chapitre 27, vous écrirez votre premier `FlameGame`.

### Ce que ce chapitre couvre

| Sujet | Pourquoi c'est indispensable pour un jeu |
| --- | --- |
| Installation du SDK | sans SDK, rien ne tourne |
| `flutter create`, `flutter run` | créer et lancer le projet du jeu |
| Hot reload | modifier la vitesse d'un ennemi sans tout relancer |
| Widget, arbre de widgets | Flame s'insère dans cet arbre via `GameWidget` |
| `StatelessWidget` / `StatefulWidget` | les menus, le HUD, les écrans de pause en sont |
| `setState()` | mettre à jour un score affiché en dehors du jeu |
| `initState()` / `dispose()` | démarrer et arrêter la boucle de jeu proprement |
| `Stack`, `Center`, `Column` | superposer un HUD au-dessus du jeu |
| `GestureDetector` | détecter un tap, un glissement du doigt |
| Coordonnées écran | tout le jeu 2D repose dessus |
| `CustomPaint` / `Canvas` | c'est **exactement** ce que Flame utilise en interne |
| `Ticker` / `AnimationController` | c'est **exactement** la boucle de jeu |

### Ce que ce chapitre NE couvre PAS

| Sujet écarté | Où il sera traité |
| --- | --- |
| Navigation entre pages (`Navigator`, routes) | PARTIE 1B, Flutter complet |
| Formulaires, `TextField`, validation | PARTIE 1B |
| Listes défilantes (`ListView`, `GridView`) | PARTIE 1B |
| Appels réseau, `http`, API REST | PARTIE 1B |
| Gestion d'état avancée (Provider, Riverpod, Bloc) | PARTIE 1B |
| Thèmes, Material 3 en profondeur, responsive design | PARTIE 1B |
| Tests de widgets | chapitre 42 |
| Publication sur les stores | chapitre 42 |

Autrement dit : ce chapitre est une **route directe**. Il n'y a pas de détour touristique. Chaque section existe parce qu'elle sera réutilisée dans les chapitres 20 à 42.

> **Remarque importante.** Si vous avez déjà fait du Flutter, vous pouvez survoler les sections 19.3 à 19.10 (installation et outillage) et commencer sérieusement à 19.11. En revanche, ne sautez pas 19.28 à 19.33 : ce sont les sections qui préparent réellement le jeu.

---

## 19.2 — Dart, Flutter, Flame : qui fait quoi

C'est la première source de confusion chez les débutants. Réglons-la immédiatement.

Trois noms, trois rôles très différents :

```text
  ┌───────────────────────────────────────────────────────────────┐
  │                            FLAME                              │
  │            (moteur de jeu 2D, un package Dart)                │
  │                                                               │
  │   Sprites  ·  Composants  ·  Collisions  ·  Caméra  ·  Audio  │
  │   Boucle de jeu prête à l'emploi (update / render)            │
  └───────────────────────────────────────────────────────────────┘
                              repose sur
                                  |
                                  v
  ┌───────────────────────────────────────────────────────────────┐
  │                           FLUTTER                             │
  │              (SDK d'interface, écrit en Dart + C++)           │
  │                                                               │
  │   Widgets  ·  Rendu à l'écran  ·  Canvas  ·  Événements       │
  │   Ticker (60 images par seconde)  ·  Multi-plateforme         │
  └───────────────────────────────────────────────────────────────┘
                              écrit en
                                  |
                                  v
  ┌───────────────────────────────────────────────────────────────┐
  │                             DART                              │
  │                (le langage — chapitres 01 à 18)               │
  │                                                               │
  │   Classes  ·  Fonctions  ·  Collections  ·  async/await       │
  │   Null safety  ·  Mixins  ·  Exceptions                       │
  └───────────────────────────────────────────────────────────────┘
```

Formulé en trois phrases :

- **Dart** est la **langue**. C'est ce que vous avez appris pendant dix-huit chapitres.
- **Flutter** est le **peintre**. Il sait afficher des choses à l'écran, sur Android, iOS, Windows, macOS, Linux et le Web, à partir du même code Dart.
- **Flame** est l'**assistant du peintre spécialisé en jeux**. Il ne remplace pas Flutter : il s'installe dedans et fournit tout ce qui manque à Flutter pour faire un jeu (sprites, collisions, caméra, boucle de jeu).

Une analogie qui fonctionne bien :

```text
  Dart    =  le français
  Flutter =  un traitement de texte
  Flame   =  un modèle de mise en page pour écrire un roman
```

Vous pouvez écrire un roman sans le modèle. Ce sera juste beaucoup plus long, et vous referez à la main ce que le modèle fait tout seul. C'est exactement ce que nous allons faire dans les chapitres 20 à 26 : **un jeu sans Flame**, pour comprendre ce que Flame fait à notre place à partir du chapitre 27.

> **À retenir :** Flame **n'est pas** un concurrent de Flutter. Un jeu Flame **est** une application Flutter. Le `main.dart` d'un jeu Flame contient toujours un `runApp()`.

---

## 19.3 — Installer le Flutter SDK

Le SDK (Software Development Kit, « kit de développement ») est un dossier contenant le compilateur, les outils en ligne de commande et le moteur graphique. À partir de Flutter 3.44, le SDK Flutter **inclut déjà le SDK Dart** : vous n'avez pas à installer Dart séparément.

> **Vérifié le 8 août 2026** sur `https://docs.flutter.dev/install/manual`.

### 19.3.1 — Le principe, identique sur les trois systèmes

Quel que soit votre système, la procédure suit toujours les mêmes quatre étapes :

```text
  1. Installer les outils requis (Git, et selon le système : unzip, Xcode CLI...)
  2. Télécharger l'archive du SDK Flutter
  3. L'extraire dans un dossier SANS espace ni accent dans le chemin
  4. Ajouter le sous-dossier flutter/bin au PATH
```

Le point 3 est le piège numéro un. Un chemin comme `C:\Users\Élodie Martin\Mes Documents\flutter` provoque des erreurs difficiles à diagnostiquer. Utilisez un chemin simple : `C:\dev\flutter`, `~/develop/flutter`.

Le point 4 mérite une explication. Le **PATH** est la liste des dossiers dans lesquels votre terminal cherche les programmes quand vous tapez une commande. Sans cette étape, taper `flutter` dans un terminal donnera `commande introuvable`, alors que le fichier existe bel et bien sur le disque.

```text
  Vous tapez : flutter --version
                  |
                  v
  Le terminal parcourt le PATH, dossier par dossier :
      C:\Windows\system32     -> pas de flutter.bat ici
      C:\Program Files\Git\cmd -> pas de flutter.bat ici
      C:\dev\flutter\bin       -> TROUVÉ
                  |
                  v
  Il exécute C:\dev\flutter\bin\flutter.bat
```

### 19.3.2 — Installation sur Windows

Prérequis : **Git for Windows** (`https://git-scm.com/downloads/win`). Flutter s'en sert en interne pour gérer ses versions ; sans lui, rien ne fonctionne.

Téléchargez ensuite l'archive `flutter_windows_3.44.0-stable.zip` depuis `https://docs.flutter.dev/install/archive`, puis, dans un terminal **PowerShell** :

```bash
# 1. Créer le dossier de destination
mkdir C:\dev

# 2. Extraire l'archive (adaptez le nom du fichier à la version téléchargée)
Expand-Archive -Path $env:USERPROFILE\Downloads\flutter_windows_3.44.0-stable.zip -DestinationPath C:\dev\

# 3. Vérifier que le dossier existe bien
dir C:\dev\flutter\bin
```

Ajout au PATH, via l'interface graphique :

```text
  Touche Windows + Pause
      -> Paramètres système avancés
          -> Variables d'environnement
              -> section « Variables utilisateur »
                  -> sélectionner « Path » -> Modifier
                      -> Nouveau -> C:\dev\flutter\bin
                          -> OK, OK, OK
```

**Fermez ensuite tous vos terminaux et rouvrez-en un neuf.** Un terminal déjà ouvert garde en mémoire l'ancien PATH : c'est la deuxième erreur la plus fréquente de cette section.

Vérification :

```bash
flutter --version
```

**Résultat attendu :**

```text
Flutter 3.44.0 • channel stable • https://github.com/flutter/flutter.git
Framework • revision a1b2c3d4e5 (il y a 6 jours) • 2026-08-02 10:12:33 -0700
Engine • revision f6e5d4c3b2
Tools • Dart 3.11.0 • DevTools 2.48.0
```

> **Remarque antivirus.** Certains antivirus mettent en quarantaine `flutter.bat` pendant l'extraction. Si `C:\dev\flutter\bin\flutter.bat` est absent, ajoutez `C:\dev\flutter` aux exclusions de l'antivirus et ré-extrayez l'archive.

### 19.3.3 — Installation sur macOS

Prérequis : les **outils en ligne de commande de Xcode**.

```bash
xcode-select --install
```

Vérifiez ensuite votre processeur (menu Pomme, « À propos de ce Mac ») : **Apple Silicon** ou **Intel**. Téléchargez l'archive correspondante, puis :

```bash
# 1. Créer le dossier de destination
mkdir -p ~/develop

# 2. Extraire l'archive
unzip ~/Downloads/flutter_macos_arm64_3.44.0-stable.zip -d ~/develop/

# 3. Ajouter au PATH (zsh est le shell par défaut de macOS)
echo 'export PATH="$HOME/develop/flutter/bin:$PATH"' >> ~/.zprofile

# 4. Recharger la configuration du shell
source ~/.zprofile

# 5. Vérifier
flutter --version
```

> **Remarque.** Flutter est en train d'abandonner la prise en charge des Mac Intel (x64). Sur un Mac Intel, préférez la cible Web pour suivre cette formation ; tout le contenu des chapitres 19 à 42 fonctionne dans un navigateur.

### 19.3.4 — Installation sur Linux (Debian, Ubuntu et dérivés)

```bash
# 1. Installer les paquets requis
sudo apt-get update -y
sudo apt-get install -y curl git unzip xz-utils zip libglu1-mesa

# 2. Créer le dossier de destination
mkdir -p ~/develop

# 3. Extraire l'archive téléchargée
tar -xf ~/Downloads/flutter_linux_3.44.0-stable.tar.xz -C ~/develop/

# 4. Ajouter au PATH (bash)
echo 'export PATH="$HOME/develop/flutter/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 5. Vérifier
flutter --version
```

Pour zsh, remplacez `~/.bashrc` par `~/.zshenv`. Pour fish :

```bash
fish_add_path -g -p ~/develop/flutter/bin
```

> **Attention au paquet `snap`.** Il existe un `snap install flutter`. Il fonctionne, mais il complique la configuration d'Android Studio et le partage du SDK entre utilisateurs. Pour une formation, l'installation manuelle décrite ci-dessus est plus prévisible.

### 19.3.5 — Espace disque

Comptez large :

| Élément | Espace |
| --- | --- |
| SDK Flutter décompressé | environ 3 Go |
| Caches d'artefacts (téléchargés au premier `flutter run`) | 2 à 5 Go |
| Android Studio + SDK Android + un émulateur | 10 à 15 Go |
| Visual Studio (Windows desktop uniquement) | 8 à 10 Go |

Si vous n'avez que quelques giga-octets libres, ciblez uniquement le **Web** : c'est de très loin la cible la moins gourmande, et elle suffit pour toute cette formation.

---

## 19.4 — `flutter doctor` : le diagnostic

`flutter doctor` est la commande la plus utile de tout le SDK. Elle inspecte votre machine et dresse la liste de ce qui fonctionne et de ce qui manque.

```bash
flutter doctor
```

**Résultat typique sur une machine Windows correctement configurée :**

```text
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.44.0, on Microsoft Windows [version 10.0.22631.4317], locale fr-FR)
[✓] Windows Version (Windows 11 or higher, 24H2, 2009)
[✓] Android toolchain - develop for Android devices (Android SDK version 35.0.0)
[✓] Chrome - develop for the web
[✓] Visual Studio - develop Windows apps (Visual Studio Community 2022 17.11.4)
[✓] Android Studio (version 2024.2)
[✓] VS Code (version 1.95.0)
[✓] Connected device (3 available)
[✓] Network resources

• No issues found!
```

### 19.4.1 — Lire les symboles

| Symbole | Signification | Faut-il agir ? |
| --- | --- | --- |
| `[✓]` | tout va bien | non |
| `[!]` | avertissement, partiellement configuré | seulement si cette cible vous intéresse |
| `[✗]` | absent ou cassé | oui, si cette cible vous intéresse |

Le point crucial, celui que 90 % des débutants ratent :

> **Il n'est PAS nécessaire d'avoir uniquement des `[✓]`.**

Si vous développez pour le Web, un `[✗] Xcode` ne vous concerne pas. Vous n'avez besoin que des lignes correspondant à **votre** cible.

### 19.4.2 — Les avertissements les plus fréquents

**Les licences Android non acceptées :**

```text
[!] Android toolchain - develop for Android devices (Android SDK version 35.0.0)
    ✗ Android license status unknown.
```

Correction :

```bash
flutter doctor --android-licenses
```

Puis tapez `y` puis Entrée pour chaque licence.

**Le composant `cmdline-tools` manquant :**

```text
    ✗ cmdline-tools component is missing
```

Correction : dans Android Studio, `Settings` -> `Languages & Frameworks` -> `Android SDK` -> onglet `SDK Tools` -> cocher **Android SDK Command-line Tools (latest)** -> `Apply`.

**Visual Studio absent (Windows) :**

```text
[✗] Visual Studio - develop Windows apps
    ✗ Visual Studio not installed
```

Ce composant sert **uniquement** à compiler une application Windows native. Si vous ciblez le Web ou Android, ignorez complètement cette ligne.

### 19.4.3 — Le mode détaillé

Quand une ligne est en `[!]` sans explication claire :

```bash
flutter doctor -v
```

La sortie devient longue mais précise : chemins exacts, versions exactes, cause exacte de l'avertissement.

> **Réflexe à prendre pour toute la formation.** Avant de poser une question sur un problème d'environnement, exécutez `flutter doctor -v` et lisez la sortie. Elle contient presque toujours la réponse.

---

## 19.5 — VS Code et l'extension Flutter

Vous pouvez développer en Flutter avec n'importe quel éditeur, mais deux environnements sont réellement supportés : **Visual Studio Code** et **Android Studio / IntelliJ**. Cette formation utilise VS Code, plus léger et identique sur les trois systèmes.

### 19.5.1 — Installer l'extension

```text
  1. Ouvrir VS Code
  2. Barre latérale gauche -> icône Extensions (ou Ctrl+Shift+X)
  3. Rechercher : Flutter
  4. Installer l'extension « Flutter » publiée par Dart Code
     (l'extension « Dart » est installée automatiquement avec elle)
  5. Redémarrer VS Code
```

Une seule extension suffit. Il n'y a pas besoin d'installer l'extension « Dart » séparément : elle est une dépendance de « Flutter ».

### 19.5.2 — Vérifier que l'extension voit le SDK

Ouvrez la palette de commandes :

```text
  Ctrl+Shift+P   (Windows / Linux)
  Cmd+Shift+P    (macOS)
```

Tapez `Flutter: Run Flutter Doctor`. Si l'extension trouve le SDK, le rapport s'affiche dans le terminal intégré. Sinon, elle vous propose de localiser le SDK : indiquez-lui le dossier `flutter` (celui qui contient `bin`), pas le dossier `bin` lui-même.

### 19.5.3 — Les raccourcis à mémoriser dès maintenant

| Raccourci | Effet | Fréquence d'usage |
| --- | --- | --- |
| `Ctrl+Shift+P` | palette de commandes | constante |
| `F5` | lancer en mode debug | constante |
| `Ctrl+S` | enregistrer, et **déclencher le hot reload** | permanente |
| `Ctrl+.` | actions rapides (« Quick Fix ») | permanente |
| `Ctrl+Espace` | complétion | permanente |
| `Shift+Alt+F` | formater le fichier | fréquente |
| `F12` | aller à la définition | fréquente |

`Ctrl+.` mérite une mention spéciale en Flutter. Placez le curseur sur un widget et appuyez dessus : VS Code propose « Wrap with Center », « Wrap with Column », « Wrap with Padding », « Remove this widget ». C'est le moyen le plus rapide de manipuler un arbre de widgets sans se perdre dans les parenthèses.

### 19.5.4 — Deux réglages qui font gagner des heures

Ouvrez `File` -> `Preferences` -> `Settings`, cherchez chaque réglage :

| Réglage | Valeur | Pourquoi |
| --- | --- | --- |
| `Editor: Format On Save` | coché | le code reste toujours propre |
| `Dart: Preview Flutter UI Guides` | coché | affiche des lignes reliant les widgets imbriqués |

Le second est particulièrement précieux quand on débute : il dessine dans la marge la structure de l'arbre de widgets, ce qui aide énormément à ne pas se tromper de niveau d'imbrication.

---

## 19.6 — Choisir sa cible : Web, Android ou Windows

Flutter compile le même code pour six cibles. Pour apprendre le développement de jeux, elles ne se valent pas du tout.

```text
  ┌────────────┬──────────────┬────────────┬──────────────┬─────────────┐
  │   Cible    │  Mise en     │  Vitesse   │  Hot reload  │  Recommandé │
  │            │  place       │  de test   │              │  ici ?      │
  ├────────────┼──────────────┼────────────┼──────────────┼─────────────┤
  │ Web        │  Immédiate   │  Rapide    │  Oui         │  OUI        │
  │ (Chrome)   │  (rien à     │            │              │  pour       │
  │            │   installer) │            │              │  apprendre  │
  ├────────────┼──────────────┼────────────┼──────────────┼─────────────┤
  │ Windows    │  Moyenne     │  Très      │  Oui         │  Bon choix  │
  │ desktop    │  (Visual     │  rapide    │              │  si déjà    │
  │            │   Studio)    │            │              │  installé   │
  ├────────────┼──────────────┼────────────┼──────────────┼─────────────┤
  │ Android    │  Longue      │  Lente au  │  Oui         │  À la fin,  │
  │ (émulateur)│  (10-15 Go)  │  démarrage │              │  ch. 42     │
  ├────────────┼──────────────┼────────────┼──────────────┼─────────────┤
  │ Android    │  Moyenne     │  Rapide    │  Oui         │  Excellent  │
  │ (téléphone)│  (câble USB, │            │              │  pour le    │
  │            │   mode dév.) │            │              │  tactile    │
  ├────────────┼──────────────┼────────────┼──────────────┼─────────────┤
  │ iOS / macOS│  Mac obliga- │  Rapide    │  Oui         │  Hors sujet │
  │            │  toire       │            │              │  ici        │
  └────────────┴──────────────┴────────────┴──────────────┴─────────────┘
```

### 19.6.1 — La recommandation de cette formation

> **Développez sur le Web (Chrome) pendant les chapitres 19 à 41. Testez sur Android au chapitre 42.**

Trois raisons :

1. **Zéro installation supplémentaire.** Si Chrome est installé, la cible Web fonctionne déjà.
2. **Le cycle est court.** `flutter run -d chrome` démarre en quelques secondes, contre parfois une minute pour un émulateur Android froid.
3. **Le clavier fonctionne.** Beaucoup de chapitres utilisent les flèches du clavier pour déplacer le joueur. Sur un émulateur mobile, c'est laborieux ; dans un navigateur sur ordinateur, c'est immédiat.

### 19.6.2 — Les limites de la cible Web

Il faut les connaître pour ne pas être surpris plus tard :

| Limite | Conséquence | Contournement |
| --- | --- | --- |
| L'audio ne démarre qu'après une interaction | la musique ne se lance pas toute seule | écran « Appuyez pour jouer » (chapitre 40) |
| Premier chargement plus lent | quelques secondes d'attente | normal, ne pas s'inquiéter |
| Performances légèrement en dessous du natif | quelques FPS en moins | invisible sur un jeu 2D simple |
| Pas d'accès direct au système de fichiers | sauvegardes différentes | `shared_preferences` (chapitre 40) |

### 19.6.3 — Lister les cibles disponibles

```bash
flutter devices
```

**Résultat typique :**

```text
Found 3 connected devices:
  Windows (desktop) • windows • windows-x64    • Microsoft Windows [version 10.0.22631.4317]
  Chrome (web)      • chrome  • web-javascript • Google Chrome 128.0.6613.120
  Edge (web)        • edge    • web-javascript • Microsoft Edge 128.0.2739.79
```

La deuxième colonne (`windows`, `chrome`, `edge`) est l'**identifiant** à passer à l'option `-d` de `flutter run`.

---

## 19.7 — `flutter create mon_jeu`

La commande qui crée un projet complet, prêt à s'exécuter :

```bash
flutter create mon_jeu
```

**Résultat :**

```text
Creating project mon_jeu...
Resolving dependencies in mon_jeu...
Got dependencies in mon_jeu.
Wrote 129 files.

All done!
You can find general documentation for Flutter at: https://docs.flutter.dev/
Detailed API documentation is available at: https://api.flutter.dev/

In order to run your application, type:

  $ cd mon_jeu
  $ flutter run

Your application code is in mon_jeu/lib/main.dart.
```

### 19.7.1 — Les règles de nommage

Le nom du projet devient le nom du package Dart. Il obéit aux mêmes règles que celles vues au chapitre 16 :

| Règle | Exemple valide | Exemple invalide |
| --- | --- | --- |
| Minuscules uniquement | `mon_jeu` | `MonJeu` |
| Séparateur : le tiret bas | `donjon_de_dart` | `donjon-de-dart` |
| Pas d'accent, pas d'espace | `donjon_de_dart` | `donjon de dart` |
| Ne commence pas par un chiffre | `jeu2d` | `2d_jeu` |
| N'est pas un mot réservé de Dart | `mon_jeu` | `class`, `switch` |

Une erreur de nommage donne un message explicite :

```text
"Donjon-De-Dart" is not a valid Dart package name.

The name should be all lowercase, with underscores to separate words,
"just_like_this".
```

### 19.7.2 — Les options utiles

**Limiter les plateformes générées** (projet beaucoup plus léger) :

```bash
flutter create --platforms=web,android mon_jeu
```

**Choisir l'identifiant d'organisation** (utile seulement pour publier plus tard) :

```bash
flutter create --org com.monstudio mon_jeu
```

Cela produit l'identifiant d'application `com.monstudio.mon_jeu`.

**Créer le projet dans le dossier courant** (le point désigne « ici ») :

```bash
mkdir donjon_de_dart
cd donjon_de_dart
flutter create .
```

**Créer un projet avec un `main.dart` minimal, sans le compteur de démonstration :**

```bash
flutter create --empty mon_jeu
```

C'est très pratique pour un jeu : le compteur par défaut serait de toute façon effacé. Nous garderons toutefois le projet complet dans ce chapitre, car la section 19.11 dissèque justement ce `main.dart` par défaut.

### 19.7.3 — Le projet de la formation

Créez dès maintenant le projet qui servira de terrain d'expérimentation pour les chapitres 19 à 26 :

```bash
flutter create --platforms=web,android donjon_de_dart
cd donjon_de_dart
```

> **Rappel du chapitre 16.** Un projet Flutter est un projet Dart : il possède un `pubspec.yaml`, un dossier `lib/`, un fichier de verrouillage `pubspec.lock`. Tout ce que vous avez appris sur l'organisation d'un projet Dart reste valable.

---

## 19.8 — Structure d'un projet Flutter

Voici ce que `flutter create` a produit. Ne vous laissez pas impressionner par le nombre de fichiers : dans cette formation, vous n'en toucherez que trois ou quatre.

```text
donjon_de_dart/
├── android/              Projet Android natif (Gradle, manifeste, icônes)
├── web/                  Page HTML hôte pour la cible Web
├── lib/                  <<< VOTRE CODE DART. C'est ici que tout se passe.
│   └── main.dart         Point d'entrée du programme
├── test/                 Tests automatisés (chapitre 42)
├── assets/               À CRÉER : images, sons, polices
├── .gitignore            Fichiers ignorés par Git
├── analysis_options.yaml Règles de l'analyseur statique
├── pubspec.yaml          <<< Dépendances et déclaration des assets
├── pubspec.lock          Versions exactes verrouillées (généré)
└── README.md             Description du projet
```

### 19.8.1 — Les trois fichiers et dossiers qui comptent

| Élément | Rôle | Le modifierez-vous ? |
| --- | --- | --- |
| `lib/` | tout votre code Dart | en permanence |
| `pubspec.yaml` | dépendances (`flame`...) et assets | à chaque nouveau package ou image |
| `assets/` | images, sons, polices | à partir du chapitre 22 |
| `android/`, `web/` | code natif généré | presque jamais |
| `pubspec.lock` | généré automatiquement | **jamais à la main** |

### 19.8.2 — Le `pubspec.yaml` par défaut

```yaml
name: donjon_de_dart
description: "A new Flutter project."
publish_to: 'none'

version: 1.0.0+1

environment:
  sdk: ^3.11.0

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
```

Analysons les lignes importantes, en prolongeant ce qui a été vu au chapitre 16 :

| Ligne | Signification |
| --- | --- |
| `name:` | nom du package, utilisé dans les `import 'package:donjon_de_dart/...'` |
| `publish_to: 'none'` | interdit la publication accidentelle sur pub.dev |
| `version: 1.0.0+1` | version visible `1.0.0`, numéro de build `1` (obligatoire pour les stores) |
| `environment: sdk:` | versions de Dart compatibles |
| `dependencies:` | packages nécessaires à l'exécution |
| `flutter: sdk: flutter` | le framework lui-même, fourni par le SDK |
| `dev_dependencies:` | packages nécessaires uniquement au développement |
| `uses-material-design: true` | active les icônes Material (`Icons.add`, etc.) |

> **Attention : le YAML est sensible à l'indentation.** Deux espaces, jamais de tabulation. Une tabulation dans un `pubspec.yaml` provoque une erreur d'analyse et le projet refuse de compiler. C'est une erreur classique et son message n'est pas toujours limpide.

### 19.8.3 — Déclarer des assets (à retenir pour le chapitre 22)

Un fichier posé dans `assets/` n'est **pas** automatiquement embarqué dans l'application. Il faut le déclarer :

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/
    - assets/audio/
```

Les points à ne jamais oublier :

- l'indentation de `assets:` est de **deux espaces** sous `flutter:` ;
- chaque entrée commence par `- ` (tiret puis espace) ;
- un chemin qui se termine par `/` inclut **tous** les fichiers du dossier, mais **pas** ses sous-dossiers ;
- après toute modification du `pubspec.yaml`, il faut **arrêter et relancer** l'application. Un simple hot reload ne suffit pas.

C'est l'une des erreurs les plus fréquentes de toute la PARTIE 2 ; elle figure dans le tableau des erreurs à la fin du chapitre.

---

## 19.9 — `flutter run`

Placez-vous dans le dossier du projet et lancez :

```bash
flutter run
```

Si plusieurs cibles sont disponibles, Flutter vous demande de choisir :

```text
Multiple devices found:
Windows (desktop) • windows • windows-x64    • Microsoft Windows [version 10.0.22631]
Chrome (web)      • chrome  • web-javascript • Google Chrome 128.0.6613.120
[1]: Windows (windows)
[2]: Chrome (chrome)
Please choose one (or "q" to quit):
```

Pour éviter la question, désignez directement la cible :

```bash
flutter run -d chrome
```

**Résultat :**

```text
Launching lib/main.dart on Chrome in debug mode...
Waiting for connection from debug service on Chrome...
This app is linked to the debug service: ws://127.0.0.1:52341/ws
Debug service listening on ws://127.0.0.1:52341/ws

Flutter run key commands.
r Hot reload.
R Hot restart.
h List all available interactive commands.
d Detach (terminate "flutter run" but leave application running).
c Clear the screen
q Quit (terminate the application on the device).

A Dart VM Service on Chrome is available at: http://127.0.0.1:52341/
The Flutter DevTools debugger and profiler on Chrome is available at:
http://127.0.0.1:9101?uri=http://127.0.0.1:52341/
```

### 19.9.1 — Les commandes interactives

Le terminal reste actif tant que l'application tourne. Ces touches, tapées dans le terminal, sont vos outils quotidiens :

| Touche | Effet | Durée typique |
| --- | --- | --- |
| `r` | **hot reload** : recharge le code, conserve l'état | moins d'une seconde |
| `R` | **hot restart** : redémarre l'application, état perdu | 1 à 5 secondes |
| `q` | quitter et arrêter l'application | immédiat |
| `h` | afficher toutes les commandes | immédiat |
| `c` | effacer l'écran du terminal | immédiat |

### 19.9.2 — Les trois modes de compilation

```bash
flutter run                # mode debug (par défaut)
flutter run --profile      # mode profile
flutter run --release      # mode release
```

| Mode | Hot reload | Vitesse | Bannière « DEBUG » | Usage |
| --- | --- | --- | --- | --- |
| `debug` | oui | lente | oui | développement quotidien |
| `profile` | non | proche du réel | non | mesurer les performances (ch. 42) |
| `release` | non | maximale | non | version finale livrée |

> **Point capital pour un jeu.** Ne jugez **jamais** les performances de votre jeu en mode debug. Le mode debug ajoute des vérifications coûteuses à chaque image. Un jeu qui rame à 25 FPS en debug tourne souvent à 60 FPS en release. Nous y reviendrons au chapitre 20, quand nous mesurerons les FPS.

### 19.9.3 — Lancer depuis VS Code

Appuyez sur `F5`. VS Code lance `flutter run` en mode debug, connecte le débogueur et vous donne accès aux points d'arrêt. La barre d'outils flottante qui apparaît en haut contient les boutons de hot reload et de hot restart.

---

## 19.10 — Le hot reload

Le hot reload (« rechargement à chaud ») est la fonctionnalité qui a fait le succès de Flutter, et c'est un avantage colossal pour le développement de jeux.

### 19.10.1 — Le principe

```text
  SANS hot reload (développement de jeu classique)

  Modifier la vitesse de l'ennemi
      -> recompiler ............................ 30 s
      -> relancer le jeu ....................... 5 s
      -> rejouer depuis le menu ................ 10 s
      -> retraverser le niveau jusqu'au boss ... 90 s
      -> observer le résultat
      TOTAL : plus de 2 minutes par essai


  AVEC hot reload

  Modifier la vitesse de l'ennemi
      -> Ctrl+S ................................ 0,4 s
      -> observer le résultat, le boss est toujours là
      TOTAL : moins d'une seconde par essai
```

Sur une session de réglage d'équilibrage (« l'ennemi est-il trop rapide ? »), la différence se compte en heures.

### 19.10.2 — Hot reload contre hot restart

C'est la distinction que tout débutant doit assimiler.

| | Hot reload (`r`) | Hot restart (`R`) |
| --- | --- | --- |
| Durée | < 1 s | 1 à 5 s |
| Le code est rechargé | oui | oui |
| L'**état** est conservé | **oui** | **non** |
| `initState()` est ré-exécuté | non | oui |
| `main()` est ré-exécuté | non | oui |
| Position du joueur, score | conservés | remis à zéro |

Traduit en situation de jeu :

```text
  Votre personnage est à x = 340, score = 1250, niveau 3.

  Vous changez la couleur du sol, puis :

    hot reload  -> le sol change de couleur
                   le personnage est TOUJOURS à x = 340, score 1250, niveau 3

    hot restart -> le sol change de couleur
                   retour au menu principal, score 0, niveau 1
```

### 19.10.3 — Ce que le hot reload ne peut pas faire

Dans ces quatre cas, un hot restart (ou un arrêt complet) est obligatoire :

| Modification | Pourquoi le hot reload échoue |
| --- | --- |
| Changer le code de `main()` | `main()` a déjà été exécuté |
| Modifier `initState()` | il a déjà été appelé pour les objets existants |
| Ajouter ou retirer un champ d'une classe existante | la forme des objets en mémoire change |
| Modifier `pubspec.yaml` (dépendance ou asset) | il faut relire le fichier et régénérer le code |

Dans le dernier cas, il faut même souvent arrêter (`q`) et relancer `flutter run`.

### 19.10.4 — Les variables `static` et `final` au niveau global

Une variable globale initialisée une seule fois n'est **pas** réévaluée par le hot reload :

```dart
final int vieMax = 100; // modifié en 200 -> le hot reload ne le voit pas
```

Si un réglage semble « ne pas prendre », c'est très souvent cela. Faites un hot restart avant de chercher un bug ailleurs.

> **Habitude à prendre.** Activez `Format On Save` (section 19.5.4) : chaque `Ctrl+S` formate le code **et** déclenche le hot reload. Vous obtenez un code propre et un aperçu instantané, sans effort.

---

## 19.11 — Le fichier `main.dart` par défaut, ligne par ligne

Ouvrez `lib/main.dart`. Voici, à quelques commentaires près, ce que `flutter create` a écrit. Nous allons le lire **intégralement**, sans rien laisser dans l'ombre.

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

### 19.11.1 — L'import

```dart
import 'package:flutter/material.dart';
```

Un seul import, mais il apporte énormément. `material.dart` réexporte les widgets de base (`Text`, `Container`, `Column`...), le système de thème, les couleurs, les icônes, et tout le style « Material Design » de Google.

C'est la même syntaxe d'import qu'au chapitre 16, avec le schéma `package:`.

> **Bon à savoir.** Il existe aussi `package:flutter/widgets.dart` (widgets sans Material) et `package:flutter/cupertino.dart` (style iOS). Pour un jeu, `material.dart` suffit largement : nous n'utilisons presque jamais le style Material, mais il apporte `MaterialApp` et `Scaffold`, très pratiques.

### 19.11.2 — La fonction `main()`

```dart
void main() {
  runApp(const MyApp());
}
```

C'est le même `main()` que dans vos dix-huit chapitres de Dart console. Le point d'entrée du programme n'a pas changé. Seul son contenu change : au lieu d'un `print()`, il appelle `runApp()`.

Remarquez le `const` devant `MyApp()`. Il est possible parce que le constructeur de `MyApp` est `const` (voir chapitre 09 sur les constructeurs constants). Nous verrons en 19.13.4 pourquoi c'est une optimisation importante.

### 19.11.3 — La classe racine `MyApp`

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});
```

`extends` : c'est l'héritage du **chapitre 10**. `MyApp` hérite de `StatelessWidget`, donc `MyApp` **est** un widget.

`{super.key}` est un paramètre nommé optionnel (chapitre 07) transmis au constructeur parent (chapitre 09). La `key` sert à Flutter pour identifier un widget quand une liste est réorganisée. Vous n'aurez pratiquement jamais à la fournir vous-même ; laissez-la, elle ne coûte rien.

### 19.11.4 — `@override` et `build()`

```dart
  @override
  Widget build(BuildContext context) {
```

`@override` est l'annotation vue au **chapitre 10** : elle signale que l'on redéfinit une méthode du parent. Elle n'est pas obligatoire mais l'analyseur la réclame, et elle évite les fautes de frappe silencieuses (`bulid` au lieu de `build` produirait une méthode inutile, jamais appelée).

`build()` est **la** méthode centrale de Flutter. Elle répond à une question :

> « À quoi cet élément de l'interface doit-il ressembler, maintenant, dans cette situation ? »

Elle retourne un `Widget`, jamais un pixel.

### 19.11.5 — Le corps de `MyApp`

```dart
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
```

| Paramètre | Rôle |
| --- | --- |
| `title` | nom de la tâche, visible dans le gestionnaire d'applications Android |
| `theme` | palette de couleurs, polices, styles par défaut |
| `ColorScheme.fromSeed` | génère une palette complète et cohérente à partir d'une seule couleur |
| `home` | le widget affiché au démarrage |

### 19.11.6 — Le widget de page

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
```

Trois choses à noter :

1. `final String title` : le champ est **`final`**. Un widget est immuable, tous ses champs le sont.
2. `required this.title` : paramètre nommé obligatoire (chapitre 07 et chapitre 12 pour le lien avec le null safety).
3. `createState()` crée l'objet qui, lui, aura le droit de changer. C'est la clé du système et nous y revenons en 19.19.

Le tiret bas devant `_MyHomePageState` en fait une classe **privée au fichier** : c'est l'encapsulation du **chapitre 10**, appliquée au niveau bibliothèque.

### 19.11.7 — L'état

```dart
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
```

`_counter` est une variable **modifiable**, contrairement aux champs du widget. C'est là que vit la donnée qui change.

`setState()` reçoit une fonction anonyme (chapitre 07). Elle modifie `_counter`, puis Flutter est prévenu qu'il faut redessiner.

### 19.11.8 — Le `build()` de l'état

```dart
    return Scaffold(
      appBar: AppBar(...),
      body: Center(child: Column(...)),
      floatingActionButton: FloatingActionButton(...),
    );
```

`Scaffold` (« échafaudage ») fournit la structure classique d'un écran : barre du haut, corps, bouton flottant. Nous y revenons en 19.16.

`widget.title` mérite une explication. Dans un objet `State`, la propriété `widget` donne accès à l'instance du `StatefulWidget` associé. C'est ainsi qu'un état lit les paramètres qu'on lui a passés.

```text
  MyHomePage(title: 'Flutter Demo Home Page')     <- le widget, immuable
        |
        | createState()
        v
  _MyHomePageState                                 <- l'état, mutable
        |
        +-- widget.title  ->  'Flutter Demo Home Page'
        +-- _counter      ->  0, puis 1, puis 2...
```

### 19.11.9 — La version « jeu » de ce fichier

Remplacez maintenant tout le contenu de `lib/main.dart` par ceci, qui sera notre base de travail :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Donjon de Dart',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF101018),
        body: const Center(
          child: Text(
            'Donjon de Dart',
            style: TextStyle(
              color: Colors.amber,
              fontSize: 32,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }
}
```

**Résultat à l'écran :**

```text
  ┌────────────────────────────────────┐
  │                                    │
  │                                    │
  │                                    │
  │         Donjon de Dart             │   (texte doré, gras, sur fond
  │                                    │    presque noir)
  │                                    │
  │                                    │
  └────────────────────────────────────┘
```

`debugShowCheckedModeBanner: false` supprime le ruban « DEBUG » du coin supérieur droit. C'est purement cosmétique, mais appréciable pour un jeu.

---

## 19.12 — `runApp()`

```dart
void runApp(Widget app)
```

Une seule ligne de code, mais c'est le pont entre le monde Dart que vous connaissez et le monde graphique.

`runApp()` fait trois choses :

```text
  runApp(const DonjonApp())
        |
        |  1. Initialise le moteur Flutter
        |     (fenêtre, surface de dessin, boucle d'événements)
        |
        |  2. Installe le widget reçu à la RACINE de l'arbre de widgets
        |
        |  3. Demande une première construction, puis rend la main
        |     à la boucle d'événements
        v
  L'application vit : elle attend des touches, des clics, des images à dessiner
```

### 19.12.1 — Le widget passé occupe tout l'écran

Le widget donné à `runApp()` est étiré pour remplir la totalité de la fenêtre. Il n'y a pas de marge, pas de barre de statut réservée : c'est à vous (ou à `MaterialApp` / `Scaffold`) de gérer cela.

Vérifiez-le avec l'exemple le plus court possible :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const ColoredBox(color: Colors.deepPurple),
  );
}
```

**Résultat :** un écran entièrement violet. Aucun `MaterialApp`, aucun `Scaffold`, et pourtant l'application tourne.

### 19.12.2 — L'erreur classique du texte « à l'envers »

Essayez ceci :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const Text('Bonjour'));
}
```

**Résultat :**

```text
======== Exception caught by widgets library ========
No Directionality widget found.
Text widgets require a Directionality widget ancestor.
```

Un `Text` a besoin de savoir si l'on écrit de gauche à droite ou de droite à gauche. `MaterialApp` fournit cette information automatiquement. Sans lui, il faut la donner à la main :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const Directionality(
      textDirection: TextDirection.ltr,
      child: ColoredBox(
        color: Colors.black,
        child: Center(
          child: Text(
            'Bonjour',
            style: TextStyle(color: Colors.white, fontSize: 40),
          ),
        ),
      ),
    ),
  );
}
```

**Résultat :** « Bonjour » en blanc, centré sur fond noir.

En pratique, on utilise toujours `MaterialApp`, et le problème ne se pose jamais. Mais si vous voyez un jour l'erreur `No Directionality widget found`, vous saurez que vous avez oublié le `MaterialApp`.

### 19.12.3 — `runApp()` ne bloque pas

Point important pour la suite, et lié au **chapitre 15** sur l'asynchrone : `runApp()` retourne immédiatement. Le code écrit après continue de s'exécuter.

```dart
import 'package:flutter/material.dart';

void main() {
  print('avant runApp');
  runApp(const MaterialApp(home: Scaffold()));
  print('après runApp');
}
```

**Résultat dans la console :**

```text
avant runApp
après runApp
```

L'application ne s'arrête pas pour autant : le moteur Flutter tourne dans la boucle d'événements, exactement comme un `Timer.periodic` maintient un programme Dart en vie (chapitre 15).

### 19.12.4 — Quand il faut initialiser avant `runApp()`

Certains cas exigent que le moteur soit prêt avant d'exécuter du code asynchrone :

```dart
import 'package:flutter/material.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // Ici, on peut charger une sauvegarde, des réglages, etc.
  await Future<void>.delayed(const Duration(milliseconds: 200));
  runApp(const MaterialApp(home: Scaffold()));
}
```

`WidgetsFlutterBinding.ensureInitialized()` démarre le moteur sans afficher d'interface. On l'utilisera au chapitre 40 pour lire le meilleur score sauvegardé avant d'afficher le menu.

Notez le `Future<void> main() async` : c'est exactement l'asynchrone du **chapitre 15**, appliqué au point d'entrée.

---

## 19.13 — Qu'est-ce qu'un widget ?

C'est la notion centrale de Flutter. Prenez le temps de bien la comprendre : tout le reste en découle.

### 19.13.1 — Définition

> Un **widget** est un objet Dart immuable qui **décrit** une partie de l'interface.

Trois mots à peser :

- **objet Dart** : une instance de classe, exactement comme au chapitre 08 ;
- **immuable** : une fois créé, il ne change plus jamais ;
- **décrit** : il ne dessine rien. Il décrit ce qui doit être dessiné.

L'analogie qui marche le mieux :

```text
  Un widget est un PLAN, pas un BÂTIMENT.

  Le plan dit : « ici, un mur de 3 m, peint en bleu ».
  Le plan ne pèse rien, on peut le jeter et en refaire un autre.
  Flutter lit le plan et construit (ou modifie) le bâtiment réel.
```

Quand l'état change, Flutter **jette le plan** et en demande un neuf. Il ne détruit pas le bâtiment : il compare les deux plans et ne modifie que les différences.

### 19.13.2 — Tout est widget

En Flutter, la quasi-totalité de ce que vous manipulez est un widget :

| Vous voulez... | Widget |
| --- | --- |
| du texte | `Text` |
| une image | `Image` |
| centrer un élément | `Center` |
| ajouter une marge | `Padding` |
| empiler verticalement | `Column` |
| empiler horizontalement | `Row` |
| superposer | `Stack` |
| détecter un clic | `GestureDetector` |
| dessiner à la main | `CustomPaint` |
| afficher un jeu Flame | `GameWidget` |

Même les concepts abstraits (la marge, la direction du texte, le thème) sont des widgets. C'est déroutant au début, très pratique ensuite : il n'y a qu'un seul mécanisme à comprendre.

### 19.13.3 — L'immuabilité en pratique

Cette ligne est illégale :

```dart
final Text titre = Text('Score : 0');
titre.data = 'Score : 10'; // ERREUR : 'data' est final
```

**Résultat :**

```text
Error: 'data' can't be used as a setter because it's final.
```

On ne **modifie** pas un widget. On en **crée un nouveau** :

```dart
Text('Score : $score')
```

et on demande à Flutter de reconstruire. C'est la raison d'être de `setState()` (section 19.20).

> **Lien avec le chapitre 10.** L'immuabilité est ici obtenue par des champs `final` et des constructeurs `const`. Vous avez déjà tous les outils ; seule l'utilisation est nouvelle.

### 19.13.4 — Pourquoi `const` partout ?

Un constructeur `const` (chapitre 09) crée l'objet **à la compilation**. Deux conséquences énormes en Flutter :

1. L'objet n'est créé qu'une seule fois, même si `build()` est appelé 60 fois par seconde ;
2. Flutter reconnaît instantanément qu'un widget `const` n'a pas changé et **saute complètement** sa reconstruction.

```dart
// Reconstruit à chaque image : 60 objets créés par seconde
Text('Vies restantes')

// Créé une seule fois, réutilisé : 0 objet créé par seconde
const Text('Vies restantes')
```

Sur une interface classique, la différence est négligeable. Dans un jeu qui reconstruit son HUD 60 fois par seconde, elle est mesurable.

> **Règle simple :** mettez `const` partout où l'analyseur vous le propose (`Ctrl+.` -> « Add const »). Si le compilateur l'accepte, c'est que c'est correct.

### 19.13.5 — Widget, Element, RenderObject

Vous n'aurez pas à manipuler ces trois couches, mais les connaître évite bien des malentendus.

```text
  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
  │     WIDGET       │   │     ELEMENT      │   │   RENDEROBJECT   │
  │                  │   │                  │   │                  │
  │  Le plan.        │   │  Le contremaître.│   │  Le bâtiment.    │
  │  Immuable.       │──>│  Fait le lien.   │──>│  Mesure, place   │
  │  Jetable.        │   │  Persiste entre  │   │  et dessine      │
  │  Recréé souvent. │   │  deux builds.    │   │  réellement.     │
  └──────────────────┘   └──────────────────┘   └──────────────────┘
      pas cher              persistant             cher à créer
```

Le point à retenir : **recréer des widgets ne coûte presque rien**. Ce qui coûte cher, ce sont les `RenderObject`, et Flutter s'arrange justement pour les réutiliser au maximum. C'est pour cela qu'on peut, sans remords, reconstruire une interface entière soixante fois par seconde.

---

## 19.14 — L'arbre de widgets

Les widgets s'imbriquent les uns dans les autres et forment un **arbre**.

### 19.14.1 — Du code à l'arbre

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Donjon')),
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text('Score : 1250'),
              Text('Vies : 3'),
            ],
          ),
        ),
      ),
    );
  }
}
```

L'arbre correspondant :

```text
  MaterialApp
      └── Scaffold
            ├── AppBar
            │     └── Text("Donjon")
            └── Center
                  └── Column
                        ├── Text("Score : 1250")
                        └── Text("Vies : 3")
```

**Résultat à l'écran :**

```text
  ┌────────────────────────────────────┐
  │  Donjon                            │  <- AppBar
  ├────────────────────────────────────┤
  │                                    │
  │                                    │
  │           Score : 1250             │  <- Column centrée
  │           Vies : 3                 │
  │                                    │
  │                                    │
  └────────────────────────────────────┘
```

### 19.14.2 — Deux formes de composition

| Type de widget | Propriété | Exemples |
| --- | --- | --- |
| Un seul enfant | `child` | `Center`, `Padding`, `Container`, `SizedBox` |
| Plusieurs enfants | `children` (une `List<Widget>`) | `Column`, `Row`, `Stack`, `ListView` |

`children` est une `List<Widget>` : c'est la `List` du **chapitre 06**. Vous pouvez donc y utiliser tout ce que vous savez faire sur les listes, y compris `map()` du **chapitre 14** :

```dart
Column(
  children: <int>[100, 250, 900]
      .map((int score) => Text('Score : $score'))
      .toList(),
)
```

### 19.14.3 — Lire un arbre sans se perdre

L'imbrication produit vite du code difficile à lire. Trois habitudes rendent la chose gérable :

1. **La virgule finale.** Terminez toujours la dernière propriété par une virgule. Le formateur passe alors chaque argument sur sa propre ligne.

```dart
// Sans virgule finale : tout sur une ligne, illisible
Center(child: Column(children: <Widget>[Text('a'), Text('b')]))

// Avec virgule finale : lisible
Center(
  child: Column(
    children: <Widget>[
      Text('a'),
      Text('b'),
    ],
  ),
)
```

2. **Extraire en widgets nommés.** Dès qu'un `build()` dépasse une trentaine de lignes, découpez-le en classes séparées. Un widget nommé `BarreDeVie` est infiniment plus lisible que six niveaux de `Container` imbriqués.

3. **`Ctrl+.` pour envelopper.** Placez le curseur sur un widget et utilisez « Wrap with... » plutôt que d'ajouter les parenthèses à la main.

### 19.14.4 — Ce que Flutter fait de l'arbre à chaque image

```text
  Un changement d'état est signalé
              |
              v
  Flutter appelle build() sur les widgets concernés
              |
              v
  Il obtient un NOUVEL arbre de widgets
              |
              v
  Il COMPARE le nouvel arbre à l'ancien, nœud par nœud
              |
              v
  Il ne met à jour que les RenderObject réellement différents
              |
              v
  Il dessine
```

Cette comparaison s'appelle la **réconciliation**. C'est elle qui rend acceptable le fait de reconstruire tout un écran pour changer un seul chiffre.

---

## 19.15 — `MaterialApp`

`MaterialApp` est le widget que l'on place presque toujours à la racine. Il ne dessine presque rien, mais il **installe des services** dont les widgets en dessous ont besoin.

### 19.15.1 — Ce qu'il apporte

| Service fourni | Sans lui |
| --- | --- |
| Direction du texte | erreur `No Directionality widget found` |
| Thème (`Theme.of(context)`) | erreur ou style par défaut brut |
| Navigation entre pages | `Navigator.push` impossible |
| Localisation (langue, formats) | absente |
| Média (taille d'écran via `MediaQuery`) | non disponible |

### 19.15.2 — Les paramètres utiles pour un jeu

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Donjon de Dart',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.amber,
          brightness: Brightness.dark,
        ),
        scaffoldBackgroundColor: const Color(0xFF101018),
      ),
      home: const EcranDeJeu(),
    );
  }
}

class EcranDeJeu extends StatelessWidget {
  const EcranDeJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text(
          'Donjon de Dart',
          style: Theme.of(context).textTheme.headlineLarge,
        ),
      ),
    );
  }
}
```

**Résultat :** un titre clair, centré, sur un fond sombre. Le style du texte vient du thème, pas d'un `TextStyle` écrit à la main.

| Paramètre | Effet dans un jeu |
| --- | --- |
| `debugShowCheckedModeBanner: false` | retire le ruban « DEBUG » |
| `brightness: Brightness.dark` | palette sombre, plus adaptée à un jeu |
| `scaffoldBackgroundColor` | couleur de fond de tous les `Scaffold` |
| `home` | l'écran de départ |

### 19.15.3 — `Theme.of(context)`

```dart
Theme.of(context).textTheme.headlineLarge
```

Cette écriture remonte l'arbre depuis le `context` courant jusqu'au premier `Theme` rencontré, et en lit une valeur. C'est le mécanisme qui permet de définir une couleur **une seule fois** et de l'utiliser partout.

```text
  MaterialApp  (contient le Theme)
      └── Scaffold
            └── Center
                  └── Text  <- Theme.of(context) remonte jusqu'à MaterialApp
```

> **Piège classique.** `Theme.of(context)` ne fonctionne que si le `context` utilisé se trouve **sous** le `MaterialApp`. Dans le `build()` de `DonjonApp`, le `context` est **au-dessus** : `Theme.of(context)` y renverrait le thème par défaut, pas le vôtre. C'est pour cette raison que l'exemple ci-dessus a extrait `EcranDeJeu` dans une classe séparée.

---

## 19.16 — `Scaffold`

`Scaffold` (« échafaudage ») fournit le squelette d'un écran Material : barre supérieure, corps, bouton flottant, barre inférieure, tiroir latéral.

### 19.16.1 — Anatomie

```text
  ┌────────────────────────────────────┐
  │  appBar                            │
  ├────────────────────────────────────┤
  │                                    │
  │                                    │
  │             body                   │
  │                                    │
  │                          ┌───┐     │
  │                          │ + │ <-- floatingActionButton
  ├──────────────────────────└───┘─────┤
  │  bottomNavigationBar               │
  └────────────────────────────────────┘
```

### 19.16.2 — Exemple complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: const Color(0xFF101018),
        appBar: AppBar(
          backgroundColor: Colors.deepPurple,
          foregroundColor: Colors.white,
          title: const Text('Donjon de Dart — Niveau 1'),
        ),
        body: const Center(
          child: Text(
            'Le héros entre dans la salle.',
            style: TextStyle(color: Colors.white70, fontSize: 20),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          backgroundColor: Colors.amber,
          onPressed: () {
            debugPrint('Attaque !');
          },
          child: const Icon(Icons.flash_on, color: Colors.black),
        ),
      ),
    );
  }
}
```

**Résultat :**

```text
  ┌────────────────────────────────────┐
  │  Donjon de Dart — Niveau 1         │  barre violette
  ├────────────────────────────────────┤
  │                                    │
  │   Le héros entre dans la salle.    │
  │                                    │
  │                          ( + )     │  bouton ambre
  └────────────────────────────────────┘
```

Chaque appui sur le bouton affiche dans la console :

```text
Attaque !
```

### 19.16.3 — `debugPrint()` plutôt que `print()`

En Flutter, préférez `debugPrint()` à `print()`. La différence :

| | `print()` | `debugPrint()` |
| --- | --- | --- |
| Sortie tronquée par Android | oui, au-delà d'environ 1000 caractères | non, découpé automatiquement |
| Peut être désactivé globalement | non | oui |
| Signalé par l'analyseur | oui (`avoid_print`) | non |

### 19.16.4 — Faut-il un `Scaffold` dans un jeu ?

Pas obligatoirement. Un jeu plein écran peut se contenter de :

```dart
MaterialApp(
  home: GameWidget(game: MonJeu()),   // à partir du chapitre 27
)
```

Mais le `Scaffold` reste très utile pour :

- l'écran de menu principal (chapitre 35) ;
- les boîtes de dialogue de pause et de Game Over (chapitre 40) ;
- fournir une couleur de fond uniforme.

Retenez qu'un `Scaffold` **sans** `appBar` est parfaitement valide :

```dart
Scaffold(
  backgroundColor: Colors.black,
  body: MonTerrainDeJeu(),
)
```

C'est la forme que nous utiliserons le plus souvent.

---

## 19.17 — `StatelessWidget`

Un `StatelessWidget` est un widget **sans état** : son apparence dépend uniquement des valeurs qu'on lui passe à la construction. Donnez-lui les mêmes paramètres, il produira toujours le même résultat.

### 19.17.1 — Le squelette

```dart
class MonWidget extends StatelessWidget {
  const MonWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return const SizedBox();
  }
}
```

Dans VS Code, tapez `stless` puis Tab : le squelette s'écrit tout seul.

### 19.17.2 — Un widget de jeu réutilisable

Écrivons un affichage de statistique, réutilisable pour le score, les vies et l'énergie.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              LigneStat(libelle: 'Score', valeur: 1250, couleur: Colors.amber),
              LigneStat(libelle: 'Vies', valeur: 3, couleur: Colors.redAccent),
              LigneStat(libelle: 'Énergie', valeur: 87, couleur: Colors.cyan),
            ],
          ),
        ),
      ),
    );
  }
}

class LigneStat extends StatelessWidget {
  const LigneStat({
    super.key,
    required this.libelle,
    required this.valeur,
    required this.couleur,
  });

  final String libelle;
  final int valeur;
  final Color couleur;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 6),
      child: Text(
        '$libelle : $valeur',
        style: TextStyle(color: couleur, fontSize: 24),
      ),
    );
  }
}
```

**Résultat :**

```text
  ┌────────────────────────────────────┐
  │                                    │
  │          Score : 1250              │  (ambre)
  │          Vies : 3                  │  (rouge)
  │          Énergie : 87              │  (cyan)
  │                                    │
  └────────────────────────────────────┘
```

Une seule classe, trois affichages différents. C'est de la **réutilisation par composition**, un principe déjà rencontré au chapitre 10.

### 19.17.3 — Quand choisir `StatelessWidget`

Choisissez-le dès que la réponse à cette question est « non » :

> « Ce widget doit-il changer d'apparence **tout seul**, sans que son parent le recrée ? »

| Élément d'un jeu | Type |
| --- | --- |
| Le titre du menu | `StatelessWidget` |
| Un bouton « Jouer » | `StatelessWidget` |
| Une ligne de statistique qui reçoit sa valeur | `StatelessWidget` |
| Un compteur de score qui s'incrémente lui-même | `StatefulWidget` |
| Le terrain de jeu animé | `StatefulWidget` |
| Un chronomètre | `StatefulWidget` |

> **Règle pratique.** Commencez toujours par un `StatelessWidget`. Passez à `StatefulWidget` uniquement quand vous constatez qu'il vous faut une donnée qui évolue **à l'intérieur** du widget. `Ctrl+.` propose la conversion automatique « Convert to StatefulWidget ».

---

## 19.18 — La méthode `build()` et `BuildContext`

### 19.18.1 — La signature

```dart
Widget build(BuildContext context)
```

Elle prend un `BuildContext`, elle rend un `Widget`. Rien d'autre.

### 19.18.2 — Les trois règles d'or de `build()`

**Règle 1 — `build()` peut être appelée très souvent.**

Soixante fois par seconde dans un jeu animé. Elle doit donc être **rapide**. On n'y met jamais :

```dart
// À NE JAMAIS FAIRE dans build()
@override
Widget build(BuildContext context) {
  final Data d = chargerLeFichierDeSauvegarde();  // lecture disque : NON
  _score = _score + 1;                            // effet de bord : NON
  demarrerLeChronometre();                        // effet de bord : NON
  return Text('$_score');
}
```

**Règle 2 — `build()` n'a pas d'effet de bord.**

Elle décrit, elle ne modifie rien. Toute modification d'état doit venir d'ailleurs : d'un appui sur un bouton, d'un `Ticker`, d'un `Future` (chapitre 15).

**Règle 3 — L'ordre des appels n'est pas garanti.**

Ne supposez jamais qu'un widget est construit avant un autre. Si vous avez besoin d'un ordre, c'est le signe qu'il faut déplacer la logique hors de `build()`.

### 19.18.3 — Qu'est-ce que le `BuildContext` ?

> Le `BuildContext` est la **position du widget dans l'arbre**.

Ce n'est ni des données, ni un état : c'est une adresse. Elle permet de **remonter** l'arbre pour interroger les ancêtres.

```text
  MaterialApp                    <- fournit Theme, MediaQuery
      └── Scaffold
            └── Center
                  └── Text       <- son context sait remonter jusqu'ici
```

Les usages courants :

```dart
Theme.of(context)                     // le thème le plus proche
MediaQuery.of(context).size           // la taille de l'écran
MediaQuery.of(context).size.width     // la largeur en pixels logiques
Navigator.of(context)                 // le navigateur (PARTIE 1B)
```

### 19.18.4 — La taille de l'écran, essentielle pour un jeu

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: AffichageTaille(),
      ),
    );
  }
}

class AffichageTaille extends StatelessWidget {
  const AffichageTaille({super.key});

  @override
  Widget build(BuildContext context) {
    final Size taille = MediaQuery.of(context).size;
    return Center(
      child: Text(
        'Terrain : ${taille.width.toStringAsFixed(0)} '
        'x ${taille.height.toStringAsFixed(0)}',
        style: const TextStyle(color: Colors.white, fontSize: 24),
      ),
    );
  }
}
```

**Résultat (dans une fenêtre Chrome de 1280 x 720) :**

```text
  ┌────────────────────────────────────┐
  │                                    │
  │       Terrain : 1280 x 720         │
  │                                    │
  └────────────────────────────────────┘
```

Redimensionnez la fenêtre : le texte se met à jour tout seul. `MediaQuery.of(context)` crée en effet un abonnement : quand la taille change, Flutter reconstruit automatiquement les widgets qui en dépendent.

C'est ainsi que l'on garde un joueur à l'intérieur des limites de l'écran, quelle que soit la taille de la fenêtre.

### 19.18.5 — L'erreur du `context` trop haut

C'est l'erreur de débutant la plus fréquente avec `BuildContext` :

```dart
class MonApp extends StatelessWidget {
  const MonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Text(
          'Titre',
          // ERREUR : ce context est AU-DESSUS du MaterialApp
          style: Theme.of(context).textTheme.headlineLarge,
        ),
      ),
    );
  }
}
```

Le `context` du paramètre de `build()` désigne l'emplacement de `MonApp`, donc **au-dessus** du `MaterialApp` que l'on est en train de créer. `Theme.of(context)` ne trouvera donc pas votre thème.

Deux corrections possibles :

1. extraire le contenu dans une classe séparée (la meilleure solution, voir 19.15.2) ;
2. utiliser un `Builder`, qui crée un nouveau `context` plus bas :

```dart
body: Builder(
  builder: (BuildContext contextInterne) {
    return Text(
      'Titre',
      style: Theme.of(contextInterne).textTheme.headlineLarge,
    );
  },
),
```

---

## 19.19 — `StatefulWidget`

Un `StatefulWidget` est un widget dont l'apparence peut changer au cours de sa vie, **de sa propre initiative**.

### 19.19.1 — Pourquoi deux classes ?

C'est la question que tout le monde se pose. La réponse tient en une phrase :

> Un widget est **immuable** et **jetable** ; l'état doit **survivre** aux reconstructions. Il faut donc deux objets distincts.

```text
  Le widget est recréé souvent :

     MyHomePage(title: 'x')   detruit
     MyHomePage(title: 'x')   detruit
     MyHomePage(title: 'x')   detruit
           |         |         |
           +---------+---------+
                     |
                     v
     _MyHomePageState  <- créé UNE SEULE FOIS, il survit à tout cela
        _counter = 0
        _counter = 1
        _counter = 2
```

Si l'état vivait dans le widget, il serait remis à zéro à chaque reconstruction et un compteur ne pourrait jamais dépasser 1.

### 19.19.2 — Le squelette complet

```dart
class MonWidget extends StatefulWidget {
  const MonWidget({super.key});

  @override
  State<MonWidget> createState() => _MonWidgetState();
}

class _MonWidgetState extends State<MonWidget> {
  @override
  Widget build(BuildContext context) {
    return const SizedBox();
  }
}
```

Dans VS Code, tapez `stful` puis Tab.

Remarquez la généricité (**chapitre 11**) : `State<MonWidget>`. Elle donne au `State` un accès typé au widget via `widget`, sans conversion.

### 19.19.3 — Premier exemple : un compteur de coups

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: CompteurDeCoups()),
      ),
    );
  }
}

class CompteurDeCoups extends StatefulWidget {
  const CompteurDeCoups({super.key});

  @override
  State<CompteurDeCoups> createState() => _CompteurDeCoupsState();
}

class _CompteurDeCoupsState extends State<CompteurDeCoups> {
  int _coups = 0;

  void _frapper() {
    setState(() {
      _coups++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          'Coups portés : $_coups',
          style: const TextStyle(color: Colors.white, fontSize: 28),
        ),
        const SizedBox(height: 24),
        ElevatedButton(
          onPressed: _frapper,
          child: const Text('Frapper le gobelin'),
        ),
      ],
    );
  }
}
```

**Résultat après trois appuis :**

```text
  ┌────────────────────────────────────┐
  │                                    │
  │       Coups portés : 3             │
  │                                    │
  │    ┌──────────────────────┐        │
  │    │  Frapper le gobelin  │        │
  │    └──────────────────────┘        │
  │                                    │
  └────────────────────────────────────┘
```

### 19.19.4 — Accéder aux paramètres avec `widget.`

Ajoutons un paramètre au widget et lisons-le depuis l'état :

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(
          child: Monstre(nom: 'Gobelin', pointsDeVie: 30),
        ),
      ),
    );
  }
}

class Monstre extends StatefulWidget {
  const Monstre({super.key, required this.nom, required this.pointsDeVie});

  final String nom;
  final int pointsDeVie;

  @override
  State<Monstre> createState() => _MonstreState();
}

class _MonstreState extends State<Monstre> {
  late int _pvActuels;

  @override
  void initState() {
    super.initState();
    _pvActuels = widget.pointsDeVie;
  }

  void _subirDegats(int montant) {
    setState(() {
      _pvActuels = (_pvActuels - montant).clamp(0, widget.pointsDeVie);
    });
  }

  @override
  Widget build(BuildContext context) {
    final bool vivant = _pvActuels > 0;
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          '${widget.nom} : $_pvActuels / ${widget.pointsDeVie} PV',
          style: TextStyle(
            color: vivant ? Colors.white : Colors.red,
            fontSize: 26,
          ),
        ),
        const SizedBox(height: 20),
        ElevatedButton(
          onPressed: vivant ? () => _subirDegats(7) : null,
          child: Text(vivant ? 'Attaquer (7 dégâts)' : 'Vaincu'),
        ),
      ],
    );
  }
}
```

**Résultat après cinq attaques :**

```text
  ┌────────────────────────────────────┐
  │       Gobelin : 0 / 30 PV          │  (en rouge)
  │                                    │
  │        ┌──────────┐                │
  │        │  Vaincu  │                │  (bouton désactivé)
  │        └──────────┘                │
  └────────────────────────────────────┘
```

Trois notions Dart déjà connues se croisent ici :

- `late int _pvActuels` : initialisation différée du **chapitre 12** ;
- `clamp(0, ...)` : bornage d'un nombre, vu au **chapitre 03** ;
- `onPressed: vivant ? ... : null` : un `onPressed` valant `null` **désactive** le bouton. C'est le null safety du **chapitre 12** utilisé comme fonctionnalité, pas comme contrainte.

---

## 19.20 — `State` et `setState()`

### 19.20.1 — La règle unique

> **Toute modification d'une donnée qui influe sur l'affichage doit se faire à l'intérieur de `setState()`.**

```dart
// FAUX : la variable change, l'écran ne bouge pas
void _frapper() {
  _coups++;
}

// CORRECT
void _frapper() {
  setState(() {
    _coups++;
  });
}
```

C'est l'erreur numéro un des débutants en Flutter, et elle est particulièrement traîtresse : **aucune erreur n'est affichée**. Le programme fonctionne, la valeur change réellement en mémoire, mais l'écran reste figé.

### 19.20.2 — Ce que fait réellement `setState()`

```text
  setState(() { _coups++; })
        |
        |  1. Exécute IMMÉDIATEMENT la fonction fournie
        |     -> _coups passe de 2 à 3
        |
        |  2. Marque cet élément comme « sale » (dirty)
        |
        |  3. Rend la main. RIEN n'est encore dessiné.
        v
  ... plus tard, à la prochaine image (dans moins de 16 ms) ...
        |
        |  4. Flutter appelle build() sur tous les éléments sales
        |  5. Il compare les arbres et met à jour ce qui diffère
        |  6. Il dessine
        v
  L'utilisateur voit « Coups portés : 3 »
```

Le point 3 est important : `setState()` **ne redessine pas**. Il **demande** un redessin. Le dessin a lieu à la prochaine image.

### 19.20.3 — Une conséquence pratique

Puisque `setState()` exécute sa fonction immédiatement, ces deux écritures sont équivalentes :

```dart
// Forme A
setState(() {
  _coups++;
});

// Forme B
_coups++;
setState(() {});
```

La forme A est très préférable : elle documente **ce qui** a changé. La forme B se rencontre parfois, notamment quand la donnée est modifiée par une autre classe, mais elle rend le code plus difficile à relire.

### 19.20.4 — Les quatre erreurs classiques

**Erreur 1 — Modifier une collection sans `setState()`.**

```dart
// FAUX : la liste change, l'écran ne suit pas
_inventaire.add('Potion');

// CORRECT
setState(() {
  _inventaire.add('Potion');
});
```

**Erreur 2 — Faire un travail long dans `setState()`.**

```dart
// FAUX : bloque l'interface
setState(() {
  _donnees = chargerUnGrosFichier();
});

// CORRECT : on travaille d'abord, on notifie ensuite
final Data d = await chargerUnGrosFichier();
setState(() {
  _donnees = d;
});
```

Le `await` est celui du **chapitre 15**. Notez que la fonction fournie à `setState()` doit être **synchrone** : n'y mettez jamais de `async`.

**Erreur 3 — Appeler `setState()` après la destruction du widget.**

```text
Unhandled Exception: setState() called after dispose()
```

Cela arrive quand un `Future` (chapitre 15) se termine alors que l'utilisateur a déjà quitté l'écran. La protection standard :

```dart
final String resultat = await chargerScore();
if (!mounted) return;
setState(() {
  _score = resultat;
});
```

`mounted` est un booléen fourni par `State` : il vaut `true` tant que le widget est attaché à l'arbre.

**Erreur 4 — Appeler `setState()` dans `build()`.**

```text
setState() or markNeedsBuild() called during build.
```

C'est une boucle infinie en puissance : construire déclenche une reconstruction, qui déclenche une reconstruction... Flutter la détecte et lève une erreur.

### 19.20.5 — `setState()` et le score du jeu

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: TableauDeScore()),
      ),
    );
  }
}

class TableauDeScore extends StatefulWidget {
  const TableauDeScore({super.key});

  @override
  State<TableauDeScore> createState() => _TableauDeScoreState();
}

class _TableauDeScoreState extends State<TableauDeScore> {
  int _score = 0;
  final List<String> _butin = <String>[];

  void _ramasserPiece() {
    setState(() {
      _score += 10;
      _butin.add('Pièce');
    });
  }

  void _ramasserCoffre() {
    setState(() {
      _score += 100;
      _butin.add('Coffre');
    });
  }

  void _reinitialiser() {
    setState(() {
      _score = 0;
      _butin.clear();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          'Score : $_score',
          style: const TextStyle(color: Colors.amber, fontSize: 36),
        ),
        const SizedBox(height: 8),
        Text(
          'Butin : ${_butin.length} objet(s)',
          style: const TextStyle(color: Colors.white70, fontSize: 18),
        ),
        const SizedBox(height: 24),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ElevatedButton(
              onPressed: _ramasserPiece,
              child: const Text('Pièce (+10)'),
            ),
            const SizedBox(width: 12),
            ElevatedButton(
              onPressed: _ramasserCoffre,
              child: const Text('Coffre (+100)'),
            ),
          ],
        ),
        const SizedBox(height: 12),
        TextButton(
          onPressed: _reinitialiser,
          child: const Text('Réinitialiser'),
        ),
      ],
    );
  }
}
```

**Résultat après deux pièces et un coffre :**

```text
  ┌────────────────────────────────────┐
  │           Score : 120              │
  │        Butin : 3 objet(s)          │
  │                                    │
  │   [ Pièce (+10) ]  [ Coffre (+100) ]│
  │            Réinitialiser            │
  └────────────────────────────────────┘
```

---

## 19.21 — Pourquoi `setState()` redessine

Comprendre le mécanisme évite d'écrire du code superstitieux (« j'ajoute un `setState()` au cas où »).

### 19.21.1 — Le cycle complet

```text
  ÉVÉNEMENT (clic, tap, image du Ticker)
        |
        v
  Votre code appelle setState(() { ... })
        |
        v
  L'Element est marqué « dirty » et ajouté à une liste
        |
        v
  Flutter demande une nouvelle image au système
        |
        |  ... attente de la prochaine synchronisation écran (16,7 ms à 60 Hz)
        v
  PHASE BUILD      : build() est appelée sur les Elements sales
        |
        v
  PHASE LAYOUT     : chaque RenderObject calcule sa taille et sa position
        |
        v
  PHASE PAINT      : chaque RenderObject se dessine sur le Canvas
        |
        v
  PHASE COMPOSITE  : les couches sont envoyées au GPU
        |
        v
  L'image apparaît à l'écran
```

### 19.21.2 — Seule la branche concernée est reconstruite

`setState()` ne reconstruit **pas** toute l'application. Il reconstruit uniquement le `State` sur lequel il a été appelé, et ses descendants.

```text
  MaterialApp                        (pas reconstruit)
      └── Scaffold                   (pas reconstruit)
            ├── AppBar               (pas reconstruit)
            └── Center
                  └── TableauDeScore      <-- setState() ici
                        ├── Text (score)  <-- RECONSTRUIT
                        ├── Text (butin)  <-- RECONSTRUIT
                        └── Row           <-- RECONSTRUIT
```

Conséquence directe et très utile :

> **Placez le `StatefulWidget` le plus bas possible dans l'arbre.**

Si votre score est géré tout en haut de l'application, chaque pièce ramassée reconstruit l'écran entier. S'il est géré par un petit widget dédié, seul l'affichage du score est reconstruit.

### 19.21.3 — Le coût réel

Reconstruire un widget consiste à créer quelques petits objets Dart : c'est de l'ordre de la microseconde. Le budget d'une image à 60 FPS est de 16,7 millisecondes.

```text
  Budget par image à 60 FPS : 16 700 microsecondes

  Créer 100 widgets .................... environ 100 µs   (0,6 % du budget)
  Layout de 100 RenderObject ........... environ 400 µs   (2,4 % du budget)
  Peindre 100 formes simples ........... environ 800 µs   (4,8 % du budget)
```

Ce n'est donc pas la reconstruction qui coûte cher, mais le dessin et surtout les calculs que **vous** ajoutez. Nous mesurerons tout cela précisément au chapitre 20.

### 19.21.4 — Visualiser les reconstructions

Ajoutez temporairement une trace dans un `build()` :

```dart
@override
Widget build(BuildContext context) {
  debugPrint('build de TableauDeScore');
  return Column(/* ... */);
}
```

Cliquez trois fois sur « Pièce » :

```text
build de TableauDeScore
build de TableauDeScore
build de TableauDeScore
```

C'est le moyen le plus simple de vérifier ce qui se reconstruit réellement, et de repérer un widget qui se reconstruit inutilement.

---

## 19.22 — Le cycle de vie : `initState()` et `dispose()`

Un objet `State` a une vie, avec un début et une fin. Trois méthodes en jalonnent le parcours.

### 19.22.1 — Le cycle complet

```text
  createState()
        |
        v
  initState()          <- UNE SEULE FOIS, avant le premier build
        |                 (démarrer un Ticker, initialiser des variables)
        v
  didChangeDependencies()   <- après initState, puis si un ancêtre change
        |
        v
  build()              <- appelée souvent
        |
        |<-------------------------+
        |                          |
        v                          |
  setState()  ---------------------+
        |
        v
  deactivate()         <- retiré de l'arbre
        |
        v
  dispose()            <- UNE SEULE FOIS, définitif
                          (arrêter le Ticker, libérer les ressources)
```

### 19.22.2 — `initState()`

Appelée **une seule fois**, avant le premier `build()`. C'est l'endroit pour :

- initialiser les variables qui dépendent de `widget.` ;
- créer un `AnimationController` ou un `Ticker` ;
- démarrer une lecture de sauvegarde ;
- s'abonner à un flux (`Stream`, chapitre 15).

```dart
@override
void initState() {
  super.initState();       // TOUJOURS en première ligne
  _pvActuels = widget.pointsDeVie;
}
```

> **Règle absolue :** `super.initState()` doit être la **première** instruction. Oublier cet appel provoque des comportements erratiques difficiles à diagnostiquer.

Une contrainte à connaître : dans `initState()`, on ne peut pas utiliser `Theme.of(context)` ni `MediaQuery.of(context)`. Le widget n'est pas encore complètement inséré dans l'arbre. Utilisez `didChangeDependencies()` ou, plus simplement, `build()`.

### 19.22.3 — `dispose()`

Appelée **une seule fois**, quand le widget disparaît définitivement. C'est l'endroit pour libérer tout ce qui continuerait à tourner :

```dart
@override
void dispose() {
  _controleur.dispose();   // AVANT super.dispose()
  _ticker.dispose();
  _abonnement.cancel();
  super.dispose();         // TOUJOURS en dernière ligne
}
```

> **Symétrie à retenir :** `super.initState()` en **premier**, `super.dispose()` en **dernier**.

Oublier `dispose()` est une fuite de ressources. Dans un jeu, c'est immédiatement visible : le `Ticker` de l'écran de jeu continue de tourner alors que le joueur est revenu au menu. Le processeur chauffe, la batterie se vide, et les objets censés être détruits appellent `setState()` sur un widget disparu.

```text
  Sans dispose() :

    Menu -> Jeu (Ticker 1 démarre)
         -> Menu (Ticker 1 tourne TOUJOURS)
         -> Jeu (Ticker 2 démarre, Ticker 1 tourne encore)
         -> Menu
         -> Jeu (Ticker 3...)

    Après dix parties : dix Tickers actifs, le jeu devient injouable.
```

### 19.22.4 — Exemple complet et vérifiable

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: DemoCycleDeVie()),
      ),
    );
  }
}

class DemoCycleDeVie extends StatefulWidget {
  const DemoCycleDeVie({super.key});

  @override
  State<DemoCycleDeVie> createState() => _DemoCycleDeVieState();
}

class _DemoCycleDeVieState extends State<DemoCycleDeVie> {
  bool _afficherLeMonstre = true;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        if (_afficherLeMonstre) const Gobelin(),
        const SizedBox(height: 24),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _afficherLeMonstre = !_afficherLeMonstre;
            });
          },
          child: Text(_afficherLeMonstre ? 'Tuer le gobelin' : 'Invoquer'),
        ),
      ],
    );
  }
}

class Gobelin extends StatefulWidget {
  const Gobelin({super.key});

  @override
  State<Gobelin> createState() => _GobelinState();
}

class _GobelinState extends State<Gobelin> {
  @override
  void initState() {
    super.initState();
    debugPrint('Gobelin : initState -> le gobelin apparaît');
  }

  @override
  void dispose() {
    debugPrint('Gobelin : dispose -> le gobelin disparaît');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('Gobelin : build');
    return const Text(
      'Un gobelin vous observe.',
      style: TextStyle(color: Colors.greenAccent, fontSize: 22),
    );
  }
}
```

**Résultat dans la console, en appuyant deux fois sur le bouton :**

```text
Gobelin : initState -> le gobelin apparaît
Gobelin : build
Gobelin : dispose -> le gobelin disparaît
Gobelin : initState -> le gobelin apparaît
Gobelin : build
```

Notez le `if` directement dans la liste `children` : c'est la **collection if** du **chapitre 06**, très utilisée en Flutter pour afficher un widget sous condition.

### 19.22.5 — Résumé du cycle de vie

| Méthode | Nombre d'appels | Usage typique dans un jeu |
| --- | --- | --- |
| `initState()` | 1 | démarrer le `Ticker`, charger le niveau |
| `didChangeDependencies()` | 1 ou plus | réagir à un changement de thème ou de taille |
| `build()` | très nombreux | décrire l'image courante |
| `setState()` | à la demande | signaler qu'une donnée a changé |
| `deactivate()` | 1 (ou plus) | rare |
| `dispose()` | 1 | arrêter le `Ticker`, couper la musique |

---

## 19.23 — `Container`, `Center`, `Column`, `Row`, `Stack`

Cinq widgets de mise en page. Nous n'en voyons ici que ce qui sert à un jeu.

### 19.23.1 — Le rôle de chacun

| Widget | Rôle | Enfants |
| --- | --- | --- |
| `Center` | centre son enfant | `child` |
| `Container` | boîte : taille, couleur, marge, bordure | `child` |
| `Column` | empile verticalement | `children` |
| `Row` | aligne horizontalement | `children` |
| `Stack` | **superpose** | `children` |

```text
   Column              Row                    Stack
  ┌───────┐      ┌─────┬─────┬─────┐      ┌─────────────┐
  │   A   │      │  A  │  B  │  C  │      │ A ┌───┐     │
  ├───────┤      └─────┴─────┴─────┘      │   │ B │     │
  │   B   │                               │   └───┘  ┌──┤
  ├───────┤                               │          │C │
  │   C   │                               └──────────┴──┘
  └───────┘
```

`Stack` est **le** widget du jeu vidéo : c'est lui qui permet de poser un HUD au-dessus du terrain, ou plusieurs sprites sur un fond.

### 19.23.2 — Les alignements

Pour `Column` et `Row`, deux axes :

- l'**axe principal** (`mainAxisAlignment`) : vertical pour `Column`, horizontal pour `Row` ;
- l'**axe secondaire** (`crossAxisAlignment`) : l'autre.

| Valeur | Effet sur l'axe principal |
| --- | --- |
| `start` | tout au début |
| `center` | au centre |
| `end` | tout à la fin |
| `spaceBetween` | premier au début, dernier à la fin, espace réparti |
| `spaceEvenly` | espaces égaux partout |
| `spaceAround` | espaces égaux, moitié aux extrémités |

### 19.23.3 — Exemple complet : une salle de donjon avec HUD

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(body: SalleDeDonjon()),
    );
  }
}

class SalleDeDonjon extends StatelessWidget {
  const SalleDeDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: <Widget>[
        // 1. Le fond : un Container qui remplit tout l'écran.
        Container(color: const Color(0xFF1B1B2F)),

        // 2. Le « joueur » : un carré posé à une position précise.
        Positioned(
          left: 120,
          top: 200,
          child: Container(
            width: 40,
            height: 40,
            decoration: BoxDecoration(
              color: Colors.amber,
              borderRadius: BorderRadius.circular(6),
              border: Border.all(color: Colors.white, width: 2),
            ),
          ),
        ),

        // 3. Le HUD, superposé en haut de l'écran.
        Positioned(
          left: 0,
          right: 0,
          top: 0,
          child: Container(
            color: Colors.black54,
            padding: const EdgeInsets.all(12),
            child: const Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: <Widget>[
                Text('Score : 1250',
                    style: TextStyle(color: Colors.amber, fontSize: 18)),
                Text('Vies : 3',
                    style: TextStyle(color: Colors.redAccent, fontSize: 18)),
                Text('Niveau 1',
                    style: TextStyle(color: Colors.white70, fontSize: 18)),
              ],
            ),
          ),
        ),

        // 4. Un message centré, par-dessus tout le reste.
        const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text('SALLE DU GOBELIN',
                  style: TextStyle(color: Colors.white, fontSize: 28)),
              SizedBox(height: 8),
              Text('Trouvez la clé.',
                  style: TextStyle(color: Colors.white54, fontSize: 16)),
            ],
          ),
        ),
      ],
    );
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────┐
  │ Score : 1250    Vies : 3      Niveau 1   │  <- HUD (Row)
  ├──────────────────────────────────────────┤
  │                                          │
  │            SALLE DU GOBELIN              │  <- Center + Column
  │             Trouvez la clé.              │
  │      ▣                                   │  <- carré ambre (Positioned)
  │                                          │
  └──────────────────────────────────────────┘
```

### 19.23.4 — `Positioned` : le placement absolu

`Positioned` ne fonctionne **que** dans un `Stack`. Il place son enfant à des coordonnées précises :

```dart
Positioned(left: 120, top: 200, child: monSprite)
```

C'est la première fois que nous plaçons quelque chose à des coordonnées choisies. Retenez cette ligne : c'est le principe de tout le positionnement en jeu 2D.

> **Ordre d'empilement.** Dans un `Stack`, le **premier** enfant de la liste est **au fond**, le **dernier** est **au-dessus**. Le fond se déclare donc en premier, le HUD en dernier.

### 19.23.5 — L'erreur `RenderFlex overflowed`

```text
A RenderFlex overflowed by 87 pixels on the bottom.
```

Traduction : une `Column` (ou une `Row`) contient plus de contenu que la place disponible. Une bande jaune et noire apparaît à l'écran.

Trois corrections possibles :

| Situation | Correction |
| --- | --- |
| Le contenu doit défiler | envelopper dans `SingleChildScrollView` |
| Un enfant doit prendre la place restante | l'envelopper dans `Expanded` |
| Le texte est trop long | l'envelopper dans `Flexible` ou `Expanded` |

---

## 19.24 — `SizedBox` et `Padding`

Deux widgets minuscules, utilisés en permanence.

### 19.24.1 — `SizedBox` : imposer une taille ou créer un espace

```dart
const SizedBox(height: 20)          // espace vertical de 20 pixels
const SizedBox(width: 12)           // espace horizontal de 12 pixels
const SizedBox(width: 64, height: 64, child: monSprite)   // taille imposée
const SizedBox.shrink()             // taille nulle : « rien »
const SizedBox.expand(child: ...)   // prend toute la place disponible
```

`SizedBox.shrink()` est très pratique pour « n'afficher rien » sous condition :

```dart
_partieTerminee ? const EcranGameOver() : const SizedBox.shrink()
```

### 19.24.2 — `Padding` et `EdgeInsets`

`Padding` ajoute une marge **intérieure** autour de son enfant.

| Écriture | Effet |
| --- | --- |
| `EdgeInsets.all(16)` | 16 pixels sur les quatre côtés |
| `EdgeInsets.symmetric(horizontal: 24, vertical: 8)` | gauche/droite 24, haut/bas 8 |
| `EdgeInsets.only(left: 12, top: 4)` | uniquement à gauche et en haut |
| `EdgeInsets.fromLTRB(8, 4, 8, 16)` | gauche, haut, droite, bas |

### 19.24.3 — Exemple complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: FicheInventaire()),
      ),
    );
  }
}

class FicheInventaire extends StatelessWidget {
  const FicheInventaire({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 280,
      decoration: BoxDecoration(
        color: const Color(0xFF232338),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.amber, width: 2),
      ),
      padding: const EdgeInsets.all(20),
      child: const Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Text('INVENTAIRE',
              style: TextStyle(color: Colors.amber, fontSize: 20)),
          SizedBox(height: 16),
          Text('Épée rouillée', style: TextStyle(color: Colors.white70)),
          SizedBox(height: 8),
          Text('Potion de soin x2',
              style: TextStyle(color: Colors.white70)),
          SizedBox(height: 8),
          Text('Clé du donjon', style: TextStyle(color: Colors.white70)),
        ],
      ),
    );
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────┐
  │  INVENTAIRE                  │
  │                              │
  │  Épée rouillée               │
  │  Potion de soin x2           │
  │  Clé du donjon               │
  └──────────────────────────────┘
       (cadre ambre arrondi)
```

`mainAxisSize: MainAxisSize.min` est indispensable ici : sans lui, la `Column` prendrait toute la hauteur de l'écran et le cadre serait immense.

---

## 19.25 — `Text` et `TextStyle`

### 19.25.1 — Les propriétés utiles

```dart
Text(
  'Score : 1250',
  style: TextStyle(
    color: Colors.amber,
    fontSize: 28,
    fontWeight: FontWeight.bold,
    fontStyle: FontStyle.italic,
    letterSpacing: 2,
    height: 1.4,
    shadows: <Shadow>[
      Shadow(color: Colors.black, offset: Offset(2, 2), blurRadius: 4),
    ],
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

| Propriété | Rôle |
| --- | --- |
| `color` | couleur |
| `fontSize` | taille en pixels logiques |
| `fontWeight` | de `w100` à `w900`, ou `bold` |
| `letterSpacing` | espacement entre les lettres |
| `height` | interligne, en multiple de `fontSize` |
| `shadows` | ombres portées, très utile pour lire un HUD sur un fond chargé |
| `textAlign` | alignement horizontal |
| `overflow` | comportement si le texte dépasse (`ellipsis`, `clip`, `fade`) |

### 19.25.2 — Les couleurs

| Écriture | Signification |
| --- | --- |
| `Colors.amber` | couleur nommée de la palette Material |
| `Colors.amber.shade700` | variante plus sombre |
| `Color(0xFF1B1B2F)` | code hexadécimal : `AA RR GG BB` |
| `Colors.black54` | noir à 54 % d'opacité |
| `Color.fromARGB(255, 27, 27, 47)` | composantes de 0 à 255 |

Le format `0xFF1B1B2F` se lit ainsi :

```text
  0x FF 1B 1B 2F
     |  |  |  |
     |  |  |  +-- bleu   (0x2F = 47)
     |  |  +----- vert   (0x1B = 27)
     |  +-------- rouge  (0x1B = 27)
     +----------- alpha  (0xFF = 255, totalement opaque)
```

Un oubli du `FF` initial donne une couleur totalement transparente : le texte semble avoir disparu. C'est une erreur fréquente.

### 19.25.3 — Exemple complet : un titre de jeu

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0D0D14),
        body: Center(child: TitreDuJeu()),
      ),
    );
  }
}

class TitreDuJeu extends StatelessWidget {
  const TitreDuJeu({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          'DONJON DE DART',
          textAlign: TextAlign.center,
          style: TextStyle(
            color: Colors.amber,
            fontSize: 46,
            fontWeight: FontWeight.w900,
            letterSpacing: 6,
            shadows: <Shadow>[
              Shadow(color: Colors.deepOrange, offset: Offset(0, 0),
                  blurRadius: 24),
            ],
          ),
        ),
        SizedBox(height: 12),
        Text(
          'Un jeu écrit en Dart et Flutter',
          style: TextStyle(
            color: Colors.white54,
            fontSize: 16,
            letterSpacing: 1.5,
          ),
        ),
      ],
    );
  }
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────┐
  │                                          │
  │        D O N J O N   D E   D A R T       │  (ambre, halo orange)
  │      Un jeu écrit en Dart et Flutter     │  (gris clair)
  │                                          │
  └──────────────────────────────────────────┘
```

---

## 19.26 — Boutons et `onPressed`

### 19.26.1 — Les trois boutons Material

| Widget | Aspect | Usage |
| --- | --- | --- |
| `ElevatedButton` | plein, avec relief | action principale (« Jouer ») |
| `OutlinedButton` | contour seul | action secondaire (« Options ») |
| `TextButton` | texte seul | action discrète (« Quitter ») |

Tous les trois prennent les mêmes paramètres essentiels :

```dart
ElevatedButton(
  onPressed: () { /* action */ },
  child: const Text('Jouer'),
)
```

### 19.26.2 — `onPressed: null` désactive le bouton

C'est un point de conception élégant, directement lié au null safety du **chapitre 12** :

```dart
ElevatedButton(
  onPressed: _clesTrouvees >= 3 ? _ouvrirLaPorte : null,
  child: const Text('Ouvrir la porte'),
)
```

Si la condition est fausse, `onPressed` vaut `null` : le bouton devient grisé et non cliquable, automatiquement.

### 19.26.3 — Styler un bouton

```dart
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.amber,
    foregroundColor: Colors.black,
    padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 18),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
    textStyle: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
  ),
  onPressed: () {},
  child: const Text('JOUER'),
)
```

### 19.26.4 — Exemple complet : un menu principal

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF0D0D14),
        body: Center(child: MenuPrincipal()),
      ),
    );
  }
}

class MenuPrincipal extends StatefulWidget {
  const MenuPrincipal({super.key});

  @override
  State<MenuPrincipal> createState() => _MenuPrincipalState();
}

class _MenuPrincipalState extends State<MenuPrincipal> {
  String _message = 'Choisissez une option.';
  bool _partieEnCours = false;

  void _jouer() {
    setState(() {
      _partieEnCours = true;
      _message = 'La partie commence !';
    });
  }

  void _reprendre() {
    setState(() {
      _message = 'Partie reprise.';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        const Text(
          'DONJON DE DART',
          style: TextStyle(color: Colors.amber, fontSize: 36,
              fontWeight: FontWeight.bold, letterSpacing: 4),
        ),
        const SizedBox(height: 40),
        ElevatedButton(
          style: ElevatedButton.styleFrom(
            backgroundColor: Colors.amber,
            foregroundColor: Colors.black,
            padding: const EdgeInsets.symmetric(horizontal: 40, vertical: 16),
          ),
          onPressed: _jouer,
          child: const Text('NOUVELLE PARTIE'),
        ),
        const SizedBox(height: 12),
        OutlinedButton(
          style: OutlinedButton.styleFrom(foregroundColor: Colors.white70),
          // Désactivé tant qu'aucune partie n'a été lancée.
          onPressed: _partieEnCours ? _reprendre : null,
          child: const Text('REPRENDRE'),
        ),
        const SizedBox(height: 12),
        TextButton(
          onPressed: () => setState(() => _message = 'À bientôt.'),
          child: const Text('QUITTER'),
        ),
        const SizedBox(height: 32),
        Text(_message, style: const TextStyle(color: Colors.white54)),
      ],
    );
  }
}
```

**Résultat au démarrage :**

```text
  ┌──────────────────────────────────────────┐
  │            DONJON DE DART                │
  │                                          │
  │        [ NOUVELLE PARTIE ]               │
  │        [ REPRENDRE ]        (grisé)      │
  │            QUITTER                       │
  │                                          │
  │      Choisissez une option.              │
  └──────────────────────────────────────────┘
```

Après un appui sur « NOUVELLE PARTIE », le bouton « REPRENDRE » devient actif et le message passe à « La partie commence ! ».

---

## 19.27 — `GestureDetector` : détecter un tap

Les boutons Material sont pratiques pour un menu, mais inadaptés à un terrain de jeu. Pour détecter un contact **n'importe où**, on utilise `GestureDetector`.

### 19.27.1 — Les rappels utiles pour un jeu

| Rappel | Déclenché quand |
| --- | --- |
| `onTap` | appui court |
| `onTapDown` | le doigt touche l'écran (donne la position) |
| `onTapUp` | le doigt se lève |
| `onDoubleTap` | double appui |
| `onLongPress` | appui prolongé |
| `onPanStart` | début d'un glissement |
| `onPanUpdate` | pendant le glissement (donne la position) |
| `onPanEnd` | fin du glissement |

### 19.27.2 — Position locale et position globale

C'est le point à comprendre absolument.

```dart
onTapDown: (TapDownDetails details) {
  final Offset local = details.localPosition;   // dans CE widget
  final Offset global = details.globalPosition; // dans TOUT l'écran
}
```

```text
  Écran complet (globalPosition)
  ┌──────────────────────────────────────┐
  │ (0,0)                                │
  │        Terrain de jeu                │
  │        ┌──────────────────────┐      │
  │        │(0,0) local           │      │
  │        │                      │      │
  │        │         X  <-- tap   │      │
  │        │                      │      │
  │        └──────────────────────┘      │
  └──────────────────────────────────────┘

  Pour le même tap :
     globalPosition = (340, 260)
     localPosition  = (250, 180)
```

> **Dans un jeu, utilisez presque toujours `localPosition`.** Elle est relative au terrain de jeu, donc directement comparable aux coordonnées de vos entités.

### 19.27.3 — Le piège du `behavior`

Un `GestureDetector` dont l'enfant est transparent ne reçoit **aucun** événement :

```dart
// Ne détecte RIEN : la zone est vide
GestureDetector(
  onTap: () => debugPrint('tap'),
  child: const SizedBox(width: 200, height: 200),
)

// Détecte : on demande explicitement de capter toute la zone
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () => debugPrint('tap'),
  child: const SizedBox(width: 200, height: 200),
)
```

`HitTestBehavior.opaque` est la solution à cette erreur très fréquente. L'autre solution consiste à donner une couleur à l'enfant (même `Colors.transparent` ne suffit pas ; il faut une vraie couleur, ou un `Container(color: ...)`).

### 19.27.4 — Exemple complet : frapper le gobelin où il se trouve

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(body: TerrainTactile()),
    );
  }
}

class TerrainTactile extends StatefulWidget {
  const TerrainTactile({super.key});

  @override
  State<TerrainTactile> createState() => _TerrainTactileState();
}

class _TerrainTactileState extends State<TerrainTactile> {
  Offset _dernierTap = Offset.zero;
  int _nombreDeTaps = 0;

  void _surTap(TapDownDetails details) {
    setState(() {
      _dernierTap = details.localPosition;
      _nombreDeTaps++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      behavior: HitTestBehavior.opaque,
      onTapDown: _surTap,
      child: Stack(
        children: <Widget>[
          Container(color: const Color(0xFF1B1B2F)),
          Positioned(
            left: _dernierTap.dx - 15,
            top: _dernierTap.dy - 15,
            child: Container(
              width: 30,
              height: 30,
              decoration: const BoxDecoration(
                color: Colors.redAccent,
                shape: BoxShape.circle,
              ),
            ),
          ),
          Positioned(
            left: 16,
            top: 16,
            child: Text(
              'Taps : $_nombreDeTaps\n'
              'x = ${_dernierTap.dx.toStringAsFixed(1)}\n'
              'y = ${_dernierTap.dy.toStringAsFixed(1)}',
              style: const TextStyle(color: Colors.white, fontSize: 18),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Résultat après un appui à (312 ; 205) :**

```text
  ┌──────────────────────────────────────────┐
  │ Taps : 1                                 │
  │ x = 312.0                                │
  │ y = 205.0                                │
  │                        ●                 │  <- disque rouge au tap
  │                                          │
  └──────────────────────────────────────────┘
```

Le `- 15` dans `left: _dernierTap.dx - 15` centre le disque de 30 pixels sur le point touché. Sans lui, le disque apparaîtrait décalé vers le bas et la droite. C'est la première fois que nous rencontrons la notion d'**ancre**, centrale au chapitre 28.

---

## 19.28 — Le système de coordonnées de l'écran

Section courte mais capitale : tout le reste de la formation en dépend.

### 19.28.1 — L'axe Y est inversé par rapport aux mathématiques

```text
  MATHÉMATIQUES au lycée          ÉCRAN (Flutter, et tous les moteurs 2D)

        y                          (0,0) ──────────────────> x
        ^                            │
        │                            │
        │                            │
   (0,0)└──────> x                   v  y

   y augmente vers le HAUT      y augmente vers le BAS
```

Conséquence à mémoriser immédiatement :

| Action | Effet sur les coordonnées |
| --- | --- |
| aller à droite | `x` augmente |
| aller à gauche | `x` diminue |
| **monter** (sauter) | **`y` diminue** |
| **descendre** (tomber) | **`y` augmente** |

C'est pour cela que la gravité, au chapitre 23, s'écrit `vitesseY += gravite` avec une gravité **positive**.

### 19.28.2 — L'origine est en haut à gauche

```text
  (0,0)
    ┌───────────────────────────────────────┐
    │                                       │
    │      (200, 100)                       │
    │          ▣                            │
    │                                       │
    │                        (500, 300)     │
    │                             ▣         │
    │                                       │
    └───────────────────────────────────────┘
                                    (largeur, hauteur)
```

### 19.28.3 — Les pixels logiques

Flutter travaille en **pixels logiques**, pas en pixels physiques.

```text
  Téléphone à 3x :   1 pixel logique = 3 pixels physiques
  Écran classique :  1 pixel logique = 1 pixel physique
```

Cela signifie qu'un carré de 40 pixels logiques a **la même taille apparente** sur tous les écrans. Vous n'avez donc jamais à gérer la densité vous-même. Toutes les coordonnées de cette formation sont en pixels logiques.

### 19.28.4 — Les classes de géométrie à connaître

| Classe | Contenu | Exemple |
| --- | --- | --- |
| `Offset` | un point : `dx`, `dy` | `const Offset(120, 80)` |
| `Size` | une dimension : `width`, `height` | `const Size(40, 40)` |
| `Rect` | un rectangle | `Rect.fromLTWH(120, 80, 40, 40)` |

Les constructeurs de `Rect` les plus utiles :

```dart
Rect.fromLTWH(120, 80, 40, 40);          // gauche, haut, largeur, hauteur
Rect.fromLTRB(120, 80, 160, 120);        // gauche, haut, droite, bas
Rect.fromCenter(center: const Offset(140, 100), width: 40, height: 40);
Rect.fromCircle(center: const Offset(140, 100), radius: 20);
```

`Offset` supporte les opérateurs arithmétiques, ce qui est très pratique pour un jeu :

```dart
const Offset position = Offset(100, 50);
const Offset vitesse = Offset(3, -2);
final Offset suivante = position + vitesse;   // Offset(103, 48)
final double distance = (suivante - position).distance;
```

C'est la surcharge d'opérateurs, une notion Dart proche de ce qui a été vu au **chapitre 11**.

---

## 19.29 — `CustomPaint` et `CustomPainter`

Nous arrivons au cœur du sujet. Jusqu'ici, nous assemblions des widgets. Maintenant, nous allons **dessiner**.

> **Vérifié le 8 août 2026** sur `https://api.flutter.dev/flutter/rendering/CustomPainter-class.html`.

### 19.29.1 — Le duo

| Élément | Nature | Rôle |
| --- | --- | --- |
| `CustomPaint` | un widget | réserve une zone de l'écran |
| `CustomPainter` | une classe abstraite | dit **quoi** dessiner dans cette zone |

```text
  CustomPaint (widget)
      |
      | painter:
      v
  MonPainter extends CustomPainter
      |
      | paint(Canvas canvas, Size size)
      v
  Le Canvas reçoit les ordres de dessin
```

### 19.29.2 — Les deux méthodes à écrire

```dart
class MonPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Dessiner ici.
  }

  @override
  bool shouldRepaint(covariant MonPainter ancien) => true;
}
```

`paint()` reçoit :

- `canvas` : la surface de dessin ;
- `size` : la taille réelle de la zone, en pixels logiques. L'origine `(0,0)` est le **coin supérieur gauche de cette zone**, pas de l'écran.

`shouldRepaint()` répond à la question : « faut-il redessiner, sachant que l'ancien peintre était celui-ci ? »

| Valeur retournée | Effet |
| --- | --- |
| `true` | redessine toujours (simple, sûr, un peu coûteux) |
| `false` | ne redessine jamais (pour un décor fixe) |
| `ancien.x != x` | redessine seulement si quelque chose a changé (idéal) |

`covariant` autorise à typer le paramètre plus précisément que dans la classe parente. C'est une notion Dart avancée ; retenez simplement l'écriture.

### 19.29.3 — `Paint` : le pinceau

```dart
final Paint pinceau = Paint()
  ..color = Colors.amber
  ..style = PaintingStyle.fill
  ..strokeWidth = 4
  ..isAntiAlias = true;
```

Les `..` sont la **notation en cascade** du chapitre 09 : chaque `..` renvoie l'objet lui-même, ce qui permet d'enchaîner les réglages.

| Propriété | Valeurs | Effet |
| --- | --- | --- |
| `color` | une `Color` | couleur du trait ou du remplissage |
| `style` | `PaintingStyle.fill` / `.stroke` | rempli ou contour seul |
| `strokeWidth` | un `double` | épaisseur du contour |
| `strokeCap` | `round`, `square`, `butt` | extrémités des lignes |

### 19.29.4 — Premier dessin complet

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: CustomPaint(
          size: Size.infinite,
          painter: PeintreDeSalle(),
        ),
      ),
    );
  }
}

class PeintreDeSalle extends CustomPainter {
  const PeintreDeSalle();

  @override
  void paint(Canvas canvas, Size size) {
    // Le sol : un rectangle plein en bas de la zone.
    final Paint sol = Paint()..color = const Color(0xFF3A3A5A);
    canvas.drawRect(
      Rect.fromLTWH(0, size.height - 80, size.width, 80),
      sol,
    );

    // Le joueur : un carré ambre posé sur le sol.
    final Paint joueur = Paint()..color = Colors.amber;
    canvas.drawRect(
      Rect.fromLTWH(120, size.height - 120, 40, 40),
      joueur,
    );

    // Le gobelin : un cercle vert.
    final Paint gobelin = Paint()..color = Colors.greenAccent;
    canvas.drawCircle(Offset(size.width - 160, size.height - 100), 20, gobelin);

    // Le contour de la zone, en pointillé simulé par un cadre.
    final Paint cadre = Paint()
      ..color = Colors.white24
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3;
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      cadre,
    );
  }

  @override
  bool shouldRepaint(covariant PeintreDeSalle ancien) => false;
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────┐
  │                                          │
  │                                          │
  │                                          │
  │      ▣                          ●        │  carré ambre / cercle vert
  │ ─────────────────────────────────────────│
  │ ███████████ le sol gris-bleu ████████████│
  └──────────────────────────────────────────┘
```

`shouldRepaint` renvoie `false` : ce décor est fixe, inutile de le redessiner.

### 19.29.5 — Deux pièges à connaître

**Piège 1 — Un `CustomPaint` sans taille est invisible.**

```dart
// Taille nulle : rien ne s'affiche
CustomPaint(painter: MonPainter())

// Corrections possibles :
CustomPaint(size: Size.infinite, painter: MonPainter())
CustomPaint(size: const Size(400, 300), painter: MonPainter())
SizedBox.expand(child: CustomPaint(painter: MonPainter()))
```

**Piège 2 — Créer les `Paint` dans `paint()`.**

C'est acceptable pour quelques formes, mais si vous dessinez des centaines d'objets, créez les pinceaux **une seule fois**, en champs de la classe :

```dart
class PeintreRapide extends CustomPainter {
  PeintreRapide();

  static final Paint _pinceauJoueur = Paint()..color = Colors.amber;

  @override
  void paint(Canvas canvas, Size size) {
    canvas.drawRect(const Rect.fromLTWH(0, 0, 40, 40), _pinceauJoueur);
  }

  @override
  bool shouldRepaint(covariant PeintreRapide ancien) => true;
}
```

---

## 19.30 — Dessiner avec `Canvas`

Le `Canvas` est l'objet qui reçoit tous les ordres de dessin. Voici les méthodes qui suffisent pour un jeu 2D complet.

### 19.30.1 — Le catalogue essentiel

| Méthode | Dessine |
| --- | --- |
| `drawRect(Rect, Paint)` | un rectangle |
| `drawRRect(RRect, Paint)` | un rectangle à coins arrondis |
| `drawCircle(Offset, double, Paint)` | un cercle |
| `drawOval(Rect, Paint)` | une ellipse inscrite dans le rectangle |
| `drawLine(Offset, Offset, Paint)` | un segment |
| `drawPath(Path, Paint)` | une forme libre |
| `drawImage(Image, Offset, Paint)` | une image (chapitre 22) |
| `drawColor(Color, BlendMode)` | remplit toute la zone |

Et les transformations :

| Méthode | Effet |
| --- | --- |
| `translate(dx, dy)` | déplace l'origine |
| `rotate(radians)` | fait pivoter autour de l'origine |
| `scale(sx, sy)` | change l'échelle |
| `save()` / `restore()` | sauvegarde et restaure l'état des transformations |

### 19.30.2 — Rectangle et cercle : l'exemple de référence

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const DonjonApp());
}

class DonjonApp extends StatelessWidget {
  const DonjonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: CustomPaint(size: Size.infinite, painter: PeintreFormes()),
      ),
    );
  }
}

class PeintreFormes extends CustomPainter {
  const PeintreFormes();

  @override
  void paint(Canvas canvas, Size size) {
    // 1. Un rectangle plein.
    final Paint plein = Paint()..color = Colors.amber;
    canvas.drawRect(const Rect.fromLTWH(40, 40, 120, 80), plein);

    // 2. Un rectangle en contour seul.
    final Paint contour = Paint()
      ..color = Colors.cyanAccent
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;
    canvas.drawRect(const Rect.fromLTWH(200, 40, 120, 80), contour);

    // 3. Un rectangle à coins arrondis.
    final Paint arrondi = Paint()..color = Colors.deepPurpleAccent;
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        const Rect.fromLTWH(360, 40, 120, 80),
        const Radius.circular(16),
      ),
      arrondi,
    );

    // 4. Un cercle plein.
    final Paint disque = Paint()..color = Colors.greenAccent;
    canvas.drawCircle(const Offset(100, 220), 45, disque);

    // 5. Un cercle en contour.
    final Paint anneau = Paint()
      ..color = Colors.redAccent
      ..style = PaintingStyle.stroke
      ..strokeWidth = 6;
    canvas.drawCircle(const Offset(260, 220), 45, anneau);

    // 6. Une ligne.
    final Paint trait = Paint()
      ..color = Colors.white70
      ..strokeWidth = 3
      ..strokeCap = StrokeCap.round;
    canvas.drawLine(const Offset(360, 180), const Offset(480, 260), trait);
  }

  @override
  bool shouldRepaint(covariant PeintreFormes ancien) => false;
}
```

**Résultat :**

```text
  ┌────────────────────────────────────────────────┐
  │   ████████    ┌────────┐    ╭────────╮         │
  │   ████████    │        │    │        │         │
  │   ████████    └────────┘    ╰────────╯         │
  │                                                │
  │      ●●●         ◯◯◯              ╲            │
  │     ●●●●●       ◯   ◯              ╲           │
  │      ●●●         ◯◯◯                ╲          │
  └────────────────────────────────────────────────┘
   1: plein   2: contour   3: arrondi
   4: disque  5: anneau    6: ligne
```

### 19.30.3 — Le piège du cercle : centre contre coin

C'est l'erreur de placement la plus fréquente.

```text
  drawRect  -> Rect.fromLTWH(x, y, w, h)
               (x, y) est le COIN SUPÉRIEUR GAUCHE

     (x,y)
       ┌───────────┐
       │           │
       └───────────┘


  drawCircle -> drawCircle(Offset(x, y), r, paint)
                (x, y) est le CENTRE

            (x,y)
         ●●●●│●●●●
        ●────┼────●
         ●●●●│●●●●
```

Pour dessiner un cercle **inscrit** au même endroit qu'un carré de 40 pixels situé en `(120, 200)` :

```dart
canvas.drawRect(const Rect.fromLTWH(120, 200, 40, 40), pinceau);
canvas.drawCircle(const Offset(120 + 20, 200 + 20), 20, pinceau);
```

### 19.30.4 — `save()` et `restore()`

Toute transformation s'applique à **tout ce qui est dessiné ensuite**. Pour la limiter, on encadre :

```dart
canvas.save();                 // mémorise l'état
canvas.translate(200, 150);    // l'origine devient (200, 150)
canvas.rotate(0.7854);         // 45 degrés en radians
canvas.drawRect(const Rect.fromLTWH(-20, -20, 40, 40), pinceau);
canvas.restore();              // revient à l'état d'avant

// Ici, l'origine est de nouveau (0, 0) et il n'y a plus de rotation.
```

Ce motif `save` / transformation / dessin / `restore` est **le** moyen de faire tourner un sprite autour de son centre. Notez le `Rect.fromLTWH(-20, -20, 40, 40)` : le rectangle est centré sur la nouvelle origine, donc la rotation se fait autour de son milieu.

> **Règle d'or :** un `save()` doit toujours avoir son `restore()`. Un `restore()` oublié décale progressivement tout le dessin, image après image, et produit un bug très déroutant.

---

## 19.31 — `Ticker` et `AnimationController` : l'idée d'une boucle qui redessine

Nous savons dessiner. Nous ne savons pas encore **bouger**.

Un jeu n'est pas une image fixe : c'est une image fixe **remplacée soixante fois par seconde**. Il nous faut donc un mécanisme qui dise à Flutter : « à chaque nouvelle image de l'écran, préviens-moi ».

Ce mécanisme s'appelle le `Ticker`.

### 19.31.1 — Ce qu'est un `Ticker`

Un `Ticker` (« batteur de mesure ») est un objet qui appelle une fonction **une fois par image d'affichage**.

```text
  L'écran rafraîchit son image 60 fois par seconde (parfois 90, 120, 144).

  écran   |----|----|----|----|----|----|----|----|----|
  temps   0   16   33   50   66   83  100  116  133  ms

  Ticker   ↓    ↓    ↓    ↓    ↓    ↓    ↓    ↓    ↓
           appel de votre fonction, avec le temps écoulé depuis le départ
```

Le `Ticker` est synchronisé sur le rafraîchissement réel de l'écran (on parle de **vsync**, pour *vertical synchronization*). C'est important : il ne sert à rien de calculer 500 images par seconde si l'écran n'en affiche que 60. Le `Ticker` cale donc votre logique sur le rythme de l'affichage.

La signature de la fonction appelée est toujours la même :

```dart
void _surChaqueImage(Duration ecoule) {
  // 'ecoule' est le temps total depuis le démarrage du Ticker
}
```

Attention à ce point, qui surprend tout le monde : le paramètre `ecoule` n'est **pas** le temps depuis l'image précédente. C'est le temps depuis le **démarrage** du `Ticker`. Il grandit sans arrêt.

```text
  appel 1  ->  ecoule = 0 ms
  appel 2  ->  ecoule = 16 ms
  appel 3  ->  ecoule = 33 ms
  appel 4  ->  ecoule = 50 ms
  ...
```

Pour obtenir le temps **entre deux images** — ce que l'on appellera le *delta time* au chapitre 20 — il faut soustraire soi-même. Nous le ferons dans un instant.

### 19.31.2 — Le `vsync` et le mixin `SingleTickerProviderStateMixin`

Un `Ticker` a besoin d'une **source de battements**. En Flutter, cette source s'appelle un `TickerProvider`. Vous ne l'écrivez jamais vous-même : vous l'obtenez en ajoutant un **mixin** (revoyez les mixins au chapitre 11) sur votre classe `State`.

| Mixin | Quand l'utiliser |
| --- | --- |
| `SingleTickerProviderStateMixin` | votre `State` a **un seul** `Ticker` ou `AnimationController` |
| `TickerProviderStateMixin` | votre `State` en a **plusieurs** |

Syntaxe :

```dart
class _MonJeuState extends State<MonJeu> with SingleTickerProviderStateMixin {
  // ...
}
```

Le mot-clé `with` est exactement celui du chapitre 11. Le mixin ajoute à votre classe une méthode `createTicker()`.

### 19.31.3 — Le cycle de vie d'un `Ticker`

Un `Ticker` se crée dans `initState()`, se démarre, et se **détruit obligatoirement** dans `dispose()` (voir la section 19.22).

```dart
late final Ticker _ticker;          // 'late' : chapitre 12

@override
void initState() {
  super.initState();
  _ticker = createTicker(_surChaqueImage);
  _ticker.start();
}

@override
void dispose() {
  _ticker.dispose();                // sinon : fuite mémoire garantie
  super.dispose();
}
```

> **Avertissement.** Si vous oubliez `_ticker.dispose()`, Flutter vous le dira en toutes lettres au moment où le widget quitte l'écran : `A Ticker was started, but never stopped`. Le `Ticker` continue de tourner sur un widget qui n'existe plus, et chaque appel à `setState()` déclenche l'exception `setState() called after dispose()`.

Les trois commandes utiles :

| Appel | Effet |
| --- | --- |
| `_ticker.start()` | démarre les battements |
| `_ticker.stop()` | les arrête (le compteur `ecoule` repart de zéro au prochain `start()`) |
| `_ticker.muted = true` | met en sourdine sans arrêter (utile pour une pause) |
| `_ticker.dispose()` | libère définitivement |

### 19.31.4 — Premier programme : compter les images

Voici un `main.dart` complet. Il affiche le temps écoulé et le nombre d'images dessinées. C'est le plus petit programme animé possible en Flutter.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const CompteurImagesApp());
}

class CompteurImagesApp extends StatelessWidget {
  const CompteurImagesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: CompteurImages()),
      ),
    );
  }
}

class CompteurImages extends StatefulWidget {
  const CompteurImages({super.key});

  @override
  State<CompteurImages> createState() => _CompteurImagesState();
}

class _CompteurImagesState extends State<CompteurImages>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;

  Duration _ecoule = Duration.zero;
  int _images = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage);
    _ticker.start();
  }

  void _surChaqueImage(Duration ecoule) {
    setState(() {
      _ecoule = ecoule;
      _images++;
    });
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final double secondes = _ecoule.inMilliseconds / 1000;
    final double moyenne = secondes > 0 ? _images / secondes : 0;

    return Text(
      'temps écoulé : ${secondes.toStringAsFixed(2)} s\n'
      'images dessinées : $_images\n'
      'moyenne : ${moyenne.toStringAsFixed(1)} images/s',
      textAlign: TextAlign.center,
      style: const TextStyle(color: Colors.amber, fontSize: 26, height: 1.6),
    );
  }
}
```

**Résultat :**

```text
  temps écoulé : 4.03 s
  images dessinées : 242
  moyenne : 60.1 images/s
```

Les chiffres défilent. Sur un écran 120 Hz, la moyenne affichera environ 120. C'est normal, et c'est précisément le problème que le chapitre 20 va résoudre.

### 19.31.5 — Calculer le temps entre deux images

Reprenons. `ecoule` est un cumul. Pour obtenir l'intervalle, on mémorise la valeur précédente :

```dart
Duration _precedent = Duration.zero;

void _surChaqueImage(Duration ecoule) {
  final Duration intervalle = ecoule - _precedent;
  _precedent = ecoule;

  final double dt = intervalle.inMicroseconds / 1000000.0; // en secondes

  setState(() {
    _x += 120 * dt;   // 120 pixels par seconde, quel que soit l'écran
  });
}
```

Le nom `dt` (delta time) est universel dans le monde du jeu vidéo. Retenez tout de suite la règle qui lui donne son sens :

> On ne déplace jamais un objet « de 3 pixels par image ». On le déplace « de 120 pixels par seconde », et on multiplie par `dt`.

Pourquoi ? Parce que « 3 pixels par image » donne 180 pixels/seconde sur un écran 60 Hz et 360 pixels/seconde sur un écran 120 Hz. Le même jeu serait deux fois plus rapide sur un téléphone récent. Avec `dt`, la vitesse est identique partout.

Le chapitre 20 est entièrement consacré à ce sujet. Pour l'instant, retenez la formule :

```text
  nouvelle_position = ancienne_position + vitesse * dt

  vitesse en pixels par seconde
  dt      en secondes
```

### 19.31.6 — `AnimationController` : la version « clé en main »

Le `Ticker` est brut : il vous donne le temps, vous faites le reste. `AnimationController` est bâti **au-dessus** du `Ticker` et vous rend un service plus haut niveau : il fait varier une valeur de `0.0` à `1.0` sur une durée donnée.

```dart
_controleur = AnimationController(
  vsync: this,                              // le mixin fournit 'this'
  duration: const Duration(seconds: 2),
);
```

| Appel | Effet |
| --- | --- |
| `.forward()` | va de 0.0 à 1.0 |
| `.reverse()` | revient de 1.0 à 0.0 |
| `.repeat()` | recommence sans fin |
| `.repeat(reverse: true)` | fait l'aller-retour sans fin |
| `.stop()` | fige |
| `.value` | la valeur courante, entre 0.0 et 1.0 |

Exemple complet : une potion qui pulse.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const PotionApp());
}

class PotionApp extends StatelessWidget {
  const PotionApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Center(child: PotionQuiPulse()),
      ),
    );
  }
}

class PotionQuiPulse extends StatefulWidget {
  const PotionQuiPulse({super.key});

  @override
  State<PotionQuiPulse> createState() => _PotionQuiPulseState();
}

class _PotionQuiPulseState extends State<PotionQuiPulse>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controleur;

  @override
  void initState() {
    super.initState();
    _controleur = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 900),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controleur.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controleur,
      builder: (BuildContext context, Widget? enfant) {
        final double rayon = 30 + 20 * _controleur.value;
        return CustomPaint(
          size: const Size(200, 200),
          painter: PeintrePotion(rayon: rayon),
        );
      },
    );
  }
}

class PeintrePotion extends CustomPainter {
  const PeintrePotion({required this.rayon});

  final double rayon;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint halo = Paint()..color = Colors.pinkAccent.withValues(alpha: 0.3);
    final Paint fiole = Paint()..color = Colors.pinkAccent;

    final Offset centre = Offset(size.width / 2, size.height / 2);
    canvas.drawCircle(centre, rayon + 12, halo);
    canvas.drawCircle(centre, rayon, fiole);
  }

  @override
  bool shouldRepaint(covariant PeintrePotion ancien) => ancien.rayon != rayon;
}
```

**Résultat :**

```text
  Un disque rose grossit puis rétrécit sans fin, entouré d'un halo.
```

Deux détails importants dans ce code.

1. `AnimatedBuilder` remplace `setState()`. Il ne reconstruit que ce qui est dans son `builder`, pas tout l'écran. C'est plus efficace, et cela évite l'oubli d'un `setState()`.
2. `shouldRepaint` renvoie `ancien.rayon != rayon` : on ne redessine que si la valeur a réellement changé.

> **Note sur `withValues`.** Depuis Flutter 3.27, `Color.withOpacity()` est déprécié au profit de `withValues(alpha: ...)`. Sur Flutter 3.44, utilisez `withValues`. Si vous lisez un tutoriel plus ancien avec `withOpacity(0.3)`, c'est l'équivalent exact.

### 19.31.7 — `Ticker` ou `AnimationController` : lequel choisir ?

| Critère | `Ticker` | `AnimationController` |
| --- | --- | --- |
| Ce qu'il donne | le temps brut | une valeur de 0.0 à 1.0 |
| Durée | infinie | fixée à l'avance |
| Convient à | une **boucle de jeu** | une **transition d'interface** |
| Courbes d'accélération | non | oui (`CurvedAnimation`) |
| Utilisé par Flame | oui, en interne | non |

La conclusion est nette et c'est celle qu'il faut retenir :

> Pour un jeu, on utilise un `Ticker`. `AnimationController` sert aux animations d'interface : un bouton qui grossit, un menu qui glisse, une fenêtre qui apparaît.

C'est d'ailleurs exactement ce que fait Flame : sa boucle de jeu repose sur un `Ticker`, et rien d'autre.

---

## 19.32 — Un premier « carré qui bouge » en Flutter pur, sans Flame

Nous avons toutes les pièces. Assemblons-les.

### 19.32.1 — Le cahier des charges

```text
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │     ▣  ──────────>                              │
  │     un carré ambre traverse l'écran             │
  │     et rebondit sur les bords                   │
  │                                                 │
  │  [ Pause ]  [ Recentrer ]        x = 312   FPS 60│
  └─────────────────────────────────────────────────┘
```

1. Un carré de 40 pixels de côté.
2. Il se déplace horizontalement à **180 pixels par seconde**.
3. Il rebondit sur les bords gauche et droit.
4. Un bouton met le mouvement en pause et le reprend.
5. Un bouton le replace au centre.
6. La vitesse est **indépendante du nombre d'images par seconde**.

### 19.32.2 — L'architecture

Trois classes, chacune avec un rôle unique.

```text
  ┌──────────────────────────────────────────────────────┐
  │  CarreQuiBouge (StatefulWidget)                      │
  │  -> déclare le widget                                │
  └──────────────────────────────────────────────────────┘
                        │
  ┌──────────────────────────────────────────────────────┐
  │  _CarreQuiBougeState (State + SingleTicker...)       │
  │  -> possède le Ticker                                │
  │  -> possède l'état : x, direction, pause             │
  │  -> calcule dt et met à jour x  ....... LA LOGIQUE   │
  └──────────────────────────────────────────────────────┘
                        │  x
  ┌──────────────────────────────────────────────────────┐
  │  PeintreCarre (CustomPainter)                        │
  │  -> dessine le sol et le carré  ....... LE RENDU     │
  └──────────────────────────────────────────────────────┘
```

Cette séparation **logique / rendu** n'est pas un caprice. C'est la structure de tous les moteurs de jeu, et c'est exactement celle que Flame impose avec ses méthodes `update()` et `render()`. Prenez l'habitude dès maintenant :

> `update()` modifie l'état, ne dessine rien. `paint()` dessine, ne modifie rien.

### 19.32.3 — Le programme complet

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const CarreApp());
}

class CarreApp extends StatelessWidget {
  const CarreApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Carré qui bouge',
      home: CarreQuiBouge(),
    );
  }
}

class CarreQuiBouge extends StatefulWidget {
  const CarreQuiBouge({super.key});

  @override
  State<CarreQuiBouge> createState() => _CarreQuiBougeState();
}

class _CarreQuiBougeState extends State<CarreQuiBouge>
    with SingleTickerProviderStateMixin {
  // --- Le moteur -----------------------------------------------------
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // --- L'état du monde ------------------------------------------------
  static const double cote = 40;      // taille du carré
  static const double vitesse = 180;  // pixels par seconde

  double _x = 20;                     // position du bord gauche du carré
  int _sens = 1;                      // 1 = vers la droite, -1 = vers la gauche
  bool _enPause = false;

  // --- Mesures --------------------------------------------------------
  double _dt = 0;
  Size _zone = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage);
    _ticker.start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  /// Appelée une fois par image d'affichage.
  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;

    // Première image : dt vaut 0, il n'y a rien à faire.
    if (dt <= 0) return;

    setState(() {
      _dt = dt;
      if (!_enPause) {
        _mettreAJour(dt);
      }
    });
  }

  /// LA LOGIQUE. Ne dessine rien.
  void _mettreAJour(double dt) {
    _x += vitesse * _sens * dt;

    final double xMax = _zone.width - cote;

    if (_x <= 0) {
      _x = 0;
      _sens = 1;
    } else if (_x >= xMax) {
      _x = xMax;
      _sens = -1;
    }
  }

  void _basculerPause() {
    setState(() => _enPause = !_enPause);
  }

  void _recentrer() {
    setState(() => _x = (_zone.width - cote) / 2);
  }

  @override
  Widget build(BuildContext context) {
    final double fps = _dt > 0 ? 1 / _dt : 0;

    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            // ---- La scène ------------------------------------------
            Expanded(
              child: LayoutBuilder(
                builder: (BuildContext context, BoxConstraints contraintes) {
                  _zone = Size(contraintes.maxWidth, contraintes.maxHeight);
                  return CustomPaint(
                    size: Size.infinite,
                    painter: PeintreCarre(x: _x, cote: cote),
                  );
                },
              ),
            ),

            // ---- Le tableau de bord --------------------------------
            Padding(
              padding: const EdgeInsets.all(12),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: <Widget>[
                  ElevatedButton(
                    onPressed: _basculerPause,
                    child: Text(_enPause ? 'Reprendre' : 'Pause'),
                  ),
                  ElevatedButton(
                    onPressed: _recentrer,
                    child: const Text('Recentrer'),
                  ),
                  Text(
                    'x = ${_x.toStringAsFixed(0)}   ${fps.toStringAsFixed(0)} img/s',
                    style: const TextStyle(color: Colors.white70, fontSize: 14),
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

/// LE RENDU. Ne modifie rien.
class PeintreCarre extends CustomPainter {
  const PeintreCarre({required this.x, required this.cote});

  final double x;
  final double cote;

  @override
  void paint(Canvas canvas, Size size) {
    final double solY = size.height - 60;

    // Le sol.
    final Paint sol = Paint()..color = const Color(0xFF2A2A40);
    canvas.drawRect(Rect.fromLTWH(0, solY, size.width, 60), sol);

    // Le carré, posé sur le sol.
    final Paint carre = Paint()..color = Colors.amber;
    canvas.drawRect(Rect.fromLTWH(x, solY - cote, cote, cote), carre);

    // Son ombre.
    final Paint ombre = Paint()..color = Colors.black.withValues(alpha: 0.35);
    canvas.drawOval(
      Rect.fromLTWH(x - 4, solY - 6, cote + 8, 12),
      ombre,
    );
  }

  @override
  bool shouldRepaint(covariant PeintreCarre ancien) => ancien.x != x;
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │                                                  │
  │                        ▣                         │
  │ ════════════════════════════════════════════════ │  le sol
  │  [ Pause ]   [ Recentrer ]     x = 318   60 img/s│
  └──────────────────────────────────────────────────┘
```

Le carré traverse l'écran, rebondit sur les bords, s'arrête quand vous appuyez sur `Pause`.

### 19.32.4 — Les points à comprendre absolument

**1. `LayoutBuilder` donne la taille de la zone de jeu.**

Nous ne pouvons pas connaître la largeur disponible avant que Flutter n'ait fait la mise en page. `LayoutBuilder` nous la fournit au moment du `build()`, dans `contraintes.maxWidth`. C'est ainsi que le carré sait où sont les bords, quelle que soit la taille de la fenêtre.

**2. `dt` est calculé par soustraction.**

```dart
final double dt =
    (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
```

`Duration.microsecondsPerSecond` vaut `1000000`. On utilise les microsecondes, et non les millisecondes, pour ne pas perdre de précision : à 60 images par seconde, un intervalle vaut 16,67 ms, ce qui s'arrondirait à 16 ou 17 en millisecondes entières, soit une erreur de 4 %.

**3. La première image est ignorée.**

Au tout premier appel, `ecoule` et `_precedent` valent tous les deux zéro, donc `dt` vaut zéro. Le `if (dt <= 0) return;` évite une division par zéro dans le calcul des images par seconde.

**4. Le rebond corrige la position.**

```dart
if (_x <= 0) {
  _x = 0;      // on recale AVANT d'inverser
  _sens = 1;
}
```

Sans la ligne `_x = 0`, le carré pourrait rester légèrement hors de l'écran et changer de sens à chaque image : il se mettrait à vibrer sur place. On appelle cela le *sticky wall bug*. Recaler la position est la correction standard.

**5. `setState()` est appelé soixante fois par seconde.**

C'est la ligne la plus coûteuse du programme, et c'est le cœur du problème de la section suivante.

### 19.32.5 — Variante : sans `setState()`

Puisque `setState()` reconstruit toute la branche de widgets, on peut faire mieux : utiliser un `ValueNotifier` et ne redessiner que le `CustomPaint`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const CarreRapideApp());
}

class CarreRapideApp extends StatelessWidget {
  const CarreRapideApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: CarreRapide(),
      ),
    );
  }
}

class CarreRapide extends StatefulWidget {
  const CarreRapide({super.key});

  @override
  State<CarreRapide> createState() => _CarreRapideState();
}

class _CarreRapideState extends State<CarreRapide>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  final ValueNotifier<double> _x = ValueNotifier<double>(20);

  Duration _precedent = Duration.zero;
  int _sens = 1;
  double _largeur = 0;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    _x.dispose();
    super.dispose();
  }

  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;
    if (dt <= 0 || _largeur == 0) return;

    double nouveau = _x.value + 180 * _sens * dt;
    if (nouveau <= 0) {
      nouveau = 0;
      _sens = 1;
    } else if (nouveau >= _largeur - 40) {
      nouveau = _largeur - 40;
      _sens = -1;
    }
    _x.value = nouveau; // aucun setState : seul le ValueListenableBuilder réagit
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints contraintes) {
        _largeur = contraintes.maxWidth;
        return ValueListenableBuilder<double>(
          valueListenable: _x,
          builder: (BuildContext context, double x, Widget? enfant) {
            return CustomPaint(
              size: Size.infinite,
              painter: PeintreCarreSimple(x: x),
            );
          },
        );
      },
    );
  }
}

class PeintreCarreSimple extends CustomPainter {
  const PeintreCarreSimple({required this.x});

  final double x;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint carre = Paint()..color = Colors.amber;
    canvas.drawRect(Rect.fromLTWH(x, size.height / 2 - 20, 40, 40), carre);
  }

  @override
  bool shouldRepaint(covariant PeintreCarreSimple ancien) => ancien.x != x;
}
```

**Explication de la différence :** avec `setState()`, Flutter reconstruit le `Scaffold`, le `Column`, les boutons, le `Text`... soixante fois par seconde, alors que rien n'a changé sauf une abscisse. Avec `ValueListenableBuilder`, seul le `CustomPaint` est reconstruit. Sur un jeu qui affiche cent objets, la différence est très visible.

Retenez la leçon : **Flutter n'a pas été conçu pour redessiner tout un arbre de widgets soixante fois par seconde.** Ce constat mène directement à la section suivante.

---

## 19.33 — Les limites de cette approche et pourquoi Flame existe

Le programme de la section 19.32 fonctionne. Il fait bouger un carré. Essayons maintenant d'en faire un vrai jeu, et regardons ce qui casse.

### 19.33.1 — Le test du passage à l'échelle

Ajoutons progressivement ce qu'un jeu de plateforme réclame, et notons ce que cela coûte en Flutter pur.

| Ce que le jeu demande | Ce qu'il faut écrire en Flutter pur |
| --- | --- |
| Un joueur qui bouge | une variable `x`, une variable `y`, un `Ticker` — **acceptable** |
| Dix gobelins | une `List<Gobelin>`, une boucle dans `update`, une boucle dans `paint` — encore acceptable |
| Chaque gobelin a sa propre animation | un compteur de frames par gobelin, géré à la main |
| Le joueur ramasse des pièces | une double boucle de test de collision, écrite à la main |
| Les pièces disparaissent | une suppression pendant l'itération : `ConcurrentModificationError` (chapitre 13) si vous n'y prenez pas garde |
| Le décor défile | une caméra, donc un décalage à soustraire dans **chaque** appel de dessin |
| Des sprites au lieu de carrés | `rootBundle`, décodage asynchrone (chapitre 15), gestion du cache d'images |
| Une animation de marche | une feuille de sprites, un découpage en rectangles, un minuteur de frames |
| Un ennemi devant le joueur | un tri par profondeur, refait à chaque image |
| Un son quand on frappe | un paquet audio, un pool de lecteurs |
| Un menu par-dessus le jeu | une pile d'écrans à gérer soi-même |
| Mettre le jeu en pause | un drapeau à tester dans chaque méthode |

Aucune de ces lignes n'est infaisable. Le problème est ailleurs :

> Vous passeriez trois mois à réécrire un moteur de jeu, et zéro semaine à faire votre jeu.

### 19.33.2 — Les quatre limites structurelles

**Limite 1 — Il n'existe pas d'arbre d'objets de jeu.**

Flutter a un arbre de **widgets**, qui décrit une interface. Un jeu a besoin d'un arbre d'**entités** : un niveau contient un joueur, le joueur porte une épée, l'épée porte une traînée de particules. Quand le joueur bouge, l'épée doit suivre. En Flutter pur, ce lien parent/enfant, vous devez le coder vous-même, y compris la composition des positions et des rotations.

```text
  Ce qu'un jeu veut :          Ce que Flutter fournit :

    Monde                        MaterialApp
     ├── Joueur                    └── Scaffold
     │    ├── Épée                      └── CustomPaint
     │    └── Barre de vie                   └── (un seul paint())
     ├── Gobelin
     └── Coffre
```

Dans le `CustomPainter`, tout est **plat** : une longue liste d'ordres de dessin. La hiérarchie n'existe que dans votre tête.

**Limite 2 — Le cycle `update` / `render` n'est pas donné.**

Nous l'avons bricolé à la main avec un `Ticker` et une soustraction de `Duration`. Il faut ensuite y ajouter le plafonnement du `dt`, la pause, le facteur de ralenti, le pas de temps fixe... Ce sont les sujets du chapitre 20, et ce sont des dizaines de lignes reproduites dans chaque projet.

**Limite 3 — `setState()` reconstruit trop.**

`setState()` marque tout le sous-arbre comme périmé. Pour une interface qui change quand l'utilisateur clique, c'est parfait. Pour un jeu qui change soixante fois par seconde, c'est du gaspillage : Flutter recrée des objets `Widget` en continu alors que seule une poignée de nombres a bougé.

```text
  Interface classique              Jeu

  clic -> setState -> rebuild      60 fois/s -> setState -> rebuild
  quelques fois par minute         soixante fois par seconde
  coût négligeable                 coût réel
```

**Limite 4 — Rien n'est prévu pour les ressources du jeu.**

Charger une image, la découper en frames, la mettre en cache, jouer un son court sans latence, lire une carte Tiled : Flutter ne propose rien de spécifique. Tout est à écrire.

### 19.33.3 — Ce que Flame apporte

Flame est un **moteur de jeu 2D construit sur Flutter**. Il ne remplace pas Flutter : il s'installe dedans, comme un widget parmi les autres (`GameWidget`). Tout ce que vous avez appris dans ce chapitre reste vrai et reste utilisé.

Voici la correspondance, point par point, entre ce que vous venez d'écrire à la main et ce que Flame fournit.

| Vous avez écrit à la main | Flame fournit | Chapitre |
| --- | --- | --- |
| un `Ticker` et un calcul de `dt` | `FlameGame.update(double dt)` appelé automatiquement | 27 |
| un `CustomPainter` | `Component.render(Canvas canvas)` | 27 |
| une `List` d'objets et deux boucles | un **arbre de composants** avec `add()` et `remove()` | 28 |
| des positions absolues | `PositionComponent` avec `position`, `size`, `angle`, `anchor` | 28 |
| un chargement d'image asynchrone | `game.images.load('joueur.png')` et `SpriteComponent` | 29 |
| un découpage de feuille de sprites | `SpriteAnimation.fromFrameData` | 29 |
| une gestion du clavier et du tactile | `KeyboardHandler`, `TapCallbacks`, `DragCallbacks`, `JoystickComponent` | 30 |
| un décalage de caméra dans chaque dessin | `CameraComponent` avec `follow()` et `World` | 31 |
| des tests de collision à la main | `HasCollisionDetection`, `RectangleHitbox`, `onCollisionStart` | 32 |
| des minuteurs et des interpolations | `Timer`, `MoveEffect`, `ScaleEffect`, `OpacityEffect` | 33 |
| un lecteur audio | `flame_audio`, `FlameAudio.bgm` | 34 |
| un tri par profondeur | la propriété `priority` d'un composant | 28 |
| une pile d'écrans | les `overlays` du `GameWidget` | 35 |

Le schéma d'ensemble :

```text
  ┌──────────────────────────────────────────────────────────┐
  │  VOTRE JEU                                               │
  │  (joueur, gobelins, niveaux, score)                      │
  ├──────────────────────────────────────────────────────────┤
  │  FLAME                                                   │
  │  boucle de jeu, composants, sprites, collisions, caméra  │
  ├──────────────────────────────────────────────────────────┤
  │  FLUTTER                                                 │
  │  widgets, Ticker, Canvas, gestes, rendu                  │  <- ce chapitre
  ├──────────────────────────────────────────────────────────┤
  │  DART                                                    │
  │  classes, listes, async, null safety                     │  <- chapitres 01 à 18
  └──────────────────────────────────────────────────────────┘
```

### 19.33.4 — Alors pourquoi avoir appris tout cela ?

Question légitime. Trois réponses.

1. **Parce que Flame ne cache rien.** Sa méthode `render(Canvas canvas)` reçoit le **même** `Canvas` que celui de la section 19.30. Ses `Vector2` remplacent `Offset`, mais l'idée est identique. Sans ce chapitre, vous utiliseriez Flame par imitation, sans comprendre.

2. **Parce que l'interface autour du jeu reste du Flutter pur.** Menu principal, écran de Game Over, réglages, HUD complexe, boutons : tout cela s'écrit avec `Scaffold`, `Column`, `Text` et `ElevatedButton`. Le chapitre 35 en dépend directement.

3. **Parce que les chapitres 20 à 26 se font en Flutter pur, volontairement.** Vous allez écrire vous-même une boucle de jeu, un système de collisions, une caméra. C'est le seul moyen de comprendre ce que Flame fera à votre place à partir du chapitre 27. Un moteur que l'on n'a jamais écrit soi-même reste une boîte noire.

> **Formule à retenir :** on n'apprend pas Flame pour éviter de comprendre le jeu vidéo. On apprend le jeu vidéo, puis on prend Flame pour aller plus vite.

---

## 19.34 — Ce qu'il faut retenir avant le chapitre 20

Ce chapitre était dense. Voici le strict minimum à avoir en tête pour attaquer la suite.

### 19.34.1 — Les huit acquis indispensables

**1. Un widget décrit, il ne dessine pas.**
Un widget est une description immuable. Flutter la compare à la précédente et met à jour l'écran en conséquence.

**2. `StatelessWidget` pour ce qui ne change pas, `StatefulWidget` pour le reste.**
Un jeu est un `StatefulWidget` : sa position, son score et ses vies changent.

**3. `setState()` est le seul moyen officiel de signaler un changement.**
Modifier une variable sans `setState()` ne redessine rien. Appeler `setState()` sans rien modifier ne sert à rien.

**4. `initState()` crée, `dispose()` détruit.**
Tout `Ticker` et tout `AnimationController` créé dans `initState()` doit être libéré dans `dispose()`. Sans exception.

**5. L'axe Y descend.**
L'origine est en haut à gauche. `y = 0` est en haut de l'écran, `y = 400` est plus bas. Ce n'est pas le repère des mathématiques.

**6. `CustomPainter` est la surface de dessin d'un jeu.**
`paint(Canvas, Size)` dessine, `shouldRepaint()` dit s'il faut recommencer.

**7. Un `Ticker` bat une fois par image.**
Il donne le temps **cumulé**. Le temps entre deux images se calcule par soustraction.

**8. On déplace en pixels par seconde, pas en pixels par image.**

```dart
position += vitesse * dt;   // vitesse en px/s, dt en secondes
```

### 19.34.2 — Le squelette à savoir écrire de mémoire

Si vous ne deviez retenir qu'un seul bloc de code de ce chapitre, ce serait celui-ci. C'est la structure de tous les programmes des chapitres 20 à 26.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() => runApp(const MonJeuApp());

class MonJeuApp extends StatelessWidget {
  const MonJeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        backgroundColor: Color(0xFF101018),
        body: Scene(),
      ),
    );
  }
}

class Scene extends StatefulWidget {
  const Scene({super.key});

  @override
  State<Scene> createState() => _SceneState();
}

class _SceneState extends State<Scene> with SingleTickerProviderStateMixin {
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_battement)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  void _battement(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;
    if (dt <= 0) return;
    setState(() => _update(dt));
  }

  void _update(double dt) {
    // toute la logique du jeu, ici
  }

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size.infinite,
      painter: _Peintre(),
    );
  }
}

class _Peintre extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // tout le dessin, ici
  }

  @override
  bool shouldRepaint(covariant _Peintre ancien) => true;
}
```

Copiez-le, gardez-le sous la main. Les chapitres suivants le remplissent, ils ne le changent pas.

### 19.34.3 — Ce que le chapitre 20 va apporter

Le squelette ci-dessus a encore trois défauts que vous ne voyez sans doute pas :

1. si l'application se fige une demi-seconde, `dt` vaut 0,5 et votre joueur traverse un mur ;
2. la vitesse d'affichage varie d'un appareil à l'autre, et certaines physiques deviennent instables ;
3. il n'y a ni pause propre, ni ralenti, ni mesure fiable des images par seconde.

Le chapitre 20 traite exactement ces trois points.

---

## 19.35 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `No Directionality widget found` | un `Text` a été passé à `runApp()` sans `MaterialApp` au-dessus | envelopper dans `MaterialApp(home: Scaffold(body: ...))` |
| Le texte s'affiche jaune sur fond noir avec un double soulignement | même cause : aucun thème Material n'est présent | ajouter `MaterialApp` |
| L'écran ne change pas quand la variable change | la variable a été modifiée sans `setState()` | placer la modification **dans** `setState(() { ... })` |
| `setState() called during build` | `setState()` est appelé depuis `build()` ou depuis un `builder` | déplacer l'appel dans un rappel d'événement (`onPressed`, `onTap`, `Ticker`) |
| `setState() called after dispose()` | un `Ticker` ou un `Future` a survécu au widget | libérer dans `dispose()`, et tester `if (!mounted) return;` après un `await` |
| `A Ticker was started, but never stopped` | `_ticker.dispose()` oublié | appeler `_ticker.dispose()` dans `dispose()`, avant `super.dispose()` |
| `Vsync is required` / `TickerProvider` introuvable | le mixin manque sur la classe `State` | ajouter `with SingleTickerProviderStateMixin` |
| `LateInitializationError: Field '_ticker' has not been initialized` | le champ `late` est lu avant `initState()` | initialiser dans `initState()` et ne jamais y accéder avant |
| Rien ne s'affiche alors que le `CustomPainter` est correct | le `CustomPaint` a une taille nulle | ajouter `size: Size.infinite` ou l'envelopper dans `SizedBox.expand` |
| Le dessin est figé alors que la valeur change | `shouldRepaint` renvoie toujours `false` | renvoyer `true` ou comparer les champs : `ancien.x != x` |
| `RenderFlex overflowed by N pixels` | un `Column` ou `Row` demande plus de place que disponible | utiliser `Expanded`, `Flexible` ou `SingleChildScrollView` |
| `Incorrect use of ParentDataWidget` | un `Positioned` a été mis ailleurs que dans un `Stack` | remettre le `Positioned` comme enfant direct d'un `Stack` |
| Le cercle n'est pas là où l'on croit | `drawCircle` prend le **centre**, `drawRect` prend le **coin** | ajouter le rayon : `Offset(x + r, y + r)` |
| Le jeu est deux fois plus rapide sur un téléphone récent | déplacement en « pixels par image » | multiplier par `dt` : `x += vitesse * dt` |
| Le carré vibre contre le bord | le sens s'inverse à chaque image car la position reste hors zone | recaler la position avant d'inverser : `_x = 0; _sens = 1;` |
| Le dessin se décale un peu plus à chaque image | un `canvas.save()` sans `canvas.restore()` | toujours appairer `save()` et `restore()` |
| `GestureDetector` ne réagit pas sur une zone vide | par défaut, seule la zone peinte est cliquable | ajouter `behavior: HitTestBehavior.opaque` |
| Le hot reload n'applique pas le changement | modification dans `main()`, dans un `initState()` déjà exécuté, ou d'une variable globale `final` | faire un hot restart (`R` majuscule) |
| `flutter run` ne trouve aucun appareil | aucune cible activée | `flutter devices`, puis `flutter config --enable-web` ou brancher un téléphone |
| `Error: Type 'Ticker' not found` | l'import manque | ajouter `import 'package:flutter/scheduler.dart';` |
| `withOpacity is deprecated` | API remplacée depuis Flutter 3.27 | utiliser `couleur.withValues(alpha: 0.5)` |

---

## 19.36 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Dart | le langage (chapitres 01 à 18) |
| Flutter | la boîte à outils d'interface et de rendu, au-dessus de Dart |
| Flame | le moteur de jeu, au-dessus de Flutter (chapitre 27) |
| SDK de référence | Flutter 3.44, Dart 3.12, canal `stable` |
| `flutter doctor` | diagnostic de l'installation ; `[!]` sur Android est sans gravité si vous visez le Web |
| `flutter create mon_jeu` | crée le projet ; nom en `snake_case`, jamais de tiret |
| `lib/main.dart` | le seul fichier que l'on écrit au début |
| `pubspec.yaml` | dépendances, assets, version du SDK |
| `flutter run -d chrome` | lance le projet dans le navigateur |
| Hot reload (`r`) | réinjecte le code, garde l'état |
| Hot restart (`R`) | relance l'application, perd l'état |
| `runApp()` | attache un widget à la racine de l'écran |
| Widget | description **immuable** de l'interface ; il ne dessine pas lui-même |
| Arbre de widgets | composition parent/enfant, reconstruite à chaque changement |
| `MaterialApp` | fournit thème, direction du texte, navigation |
| `Scaffold` | fournit la structure d'une page : `appBar`, `body`, boutons |
| `StatelessWidget` | aucune donnée interne changeante ; une seule classe |
| `StatefulWidget` | données changeantes ; deux classes : le widget et son `State` |
| `build()` | doit être rapide, sans effet de bord, et peut être appelé très souvent |
| `BuildContext` | position du widget dans l'arbre ; sert à `Theme.of`, `MediaQuery.of` |
| `setState()` | signale un changement d'état et déclenche la reconstruction |
| `initState()` | appelé une fois, à la création ; on y crée les `Ticker` |
| `dispose()` | appelé une fois, à la destruction ; on y libère les `Ticker` |
| `Container` | boîte à tout faire : taille, couleur, marge, décoration |
| `Center` | centre son unique enfant |
| `Column` / `Row` | empilent verticalement / horizontalement |
| `Stack` | superpose ; indispensable pour un HUD au-dessus du jeu |
| `SizedBox` | impose une taille ou crée un espace |
| `Padding` | ajoute une marge intérieure via `EdgeInsets` |
| `Text` / `TextStyle` | affichage et style du texte |
| `ElevatedButton.onPressed` | action au clic ; `null` désactive le bouton |
| `GestureDetector` | `onTapDown`, `onPanUpdate` : la base des contrôles tactiles |
| Coordonnées | origine en haut à gauche, X vers la droite, **Y vers le bas** |
| `Offset` / `Size` / `Rect` | point, dimensions, rectangle |
| `CustomPaint` + `CustomPainter` | la surface de dessin ; `paint()` et `shouldRepaint()` |
| `Canvas` | `drawRect`, `drawCircle`, `drawLine`, `drawOval`, `drawPath` |
| `Paint` | le pinceau : `color`, `style`, `strokeWidth` |
| `drawRect` vs `drawCircle` | l'un prend un coin, l'autre prend un **centre** |
| `save()` / `restore()` | encadrent une transformation ; toujours par paire |
| `Ticker` | rappel une fois par image ; donne le temps **cumulé** |
| `SingleTickerProviderStateMixin` | fournit le `vsync` nécessaire au `Ticker` |
| `dt` (delta time) | `(ecoule - precedent)` en secondes |
| Règle du mouvement | `position += vitesse * dt`, vitesse en pixels par seconde |
| `AnimationController` | valeur de 0.0 à 1.0 sur une durée ; pour l'interface, pas pour le jeu |
| `ValueListenableBuilder` | reconstruit une petite zone sans `setState()` |
| Séparation logique / rendu | `update()` modifie, `paint()` dessine ; jamais l'inverse |
| Pourquoi Flame | arbre de composants, sprites, collisions, caméra, audio : tout ce que Flutter ne fournit pas |

---

## 19.37 — Exercices

Tous les exercices se font dans un projet créé avec `flutter create` : vous remplacez entièrement le contenu de `lib/main.dart`. Aucun ne demande d'image ni de paquet supplémentaire.

### Exercice 1 — L'écran-titre (facile)

Écrivez un `StatelessWidget` qui affiche, sur un fond sombre `Color(0xFF101018)` :

- le titre « DONJON DE DART » en ambre, taille 40, gras, avec un espacement de lettres de 4 ;
- juste en dessous, le sous-titre « Appuyez pour commencer » en blanc à 60 % d'opacité, taille 16.

Le tout doit être centré verticalement et horizontalement. Utilisez `Column`, `SizedBox` et `Text`.

### Exercice 2 — Compteur de score cliquable (facile)

Écrivez un `StatefulWidget` qui affiche un score partant de 0.

- Un bouton « Pièce (+10) » ajoute 10 points.
- Un bouton « Gemme (+50) » ajoute 50 points.
- Un bouton « Réinitialiser » remet le score à 0. Il doit être **désactivé** (grisé) quand le score vaut déjà 0.
- Le score s'affiche en taille 60.
- Quand le score dépasse 100, le score s'affiche en vert au lieu d'ambre.

### Exercice 3 — Le HUD du donjon (facile)

Composez, en haut de l'écran, une barre d'état sur une seule ligne contenant, de gauche à droite :

- « Vies : 3 » ;
- « Score : 250 » ;
- « Niveau : 2 ».

Elle doit avoir un fond gris foncé, une marge intérieure de 12 pixels, et occuper toute la largeur. Les trois textes doivent être répartis régulièrement. Sous la barre, affichez le texte « Zone de jeu » centré dans l'espace restant.

Utilisez `Column`, `Container`, `Row`, `MainAxisAlignment.spaceEvenly`, `Expanded` et `Center`.

### Exercice 4 — La barre de vie dessinée (moyen)

Avec `CustomPaint` et `CustomPainter`, dessinez une barre de vie :

- un rectangle de fond gris foncé de 300 x 24 pixels ;
- par-dessus, un rectangle rouge dont la largeur vaut `300 * pourcentage` ;
- un contour blanc de 2 pixels autour de l'ensemble.

Le pourcentage est un `double` entre 0 et 1 passé au constructeur du peintre. Ajoutez deux boutons « Dégâts (-10 %) » et « Potion (+15 %) » qui modifient ce pourcentage, en le maintenant entre 0 et 1 avec `clamp`.

### Exercice 5 — Carré qui suit le doigt (moyen)

Dessinez un carré ambre de 50 pixels de côté. Il doit **suivre le doigt ou le curseur** :

- au premier contact, il se place immédiatement sous le point touché ;
- pendant le glissement, il suit ;
- il doit être **centré** sur le point de contact, pas positionné par son coin ;
- il ne doit jamais sortir de la zone de jeu.

Affichez également, en haut à gauche, les coordonnées courantes sous la forme `x = 123   y = 456`.

Utilisez `GestureDetector` avec `onPanDown` et `onPanUpdate`, `behavior: HitTestBehavior.opaque`, `LayoutBuilder` et `clamp`.

### Exercice 6 — La cible qui se déplace (moyen)

Affichez un cercle vert de 30 pixels de rayon à une position aléatoire. Quand l'utilisateur **touche le cercle** (et seulement le cercle) :

- le score augmente de 1 ;
- le cercle réapparaît à une nouvelle position aléatoire dans la zone.

Si l'utilisateur touche à côté, le score perd 1 point (sans descendre sous 0).

Indice : la distance entre le point touché et le centre se calcule avec `(pointTouche - centre).distance`.

### Exercice 7 — Le chronomètre du donjon (moyen)

Avec un `Ticker`, réalisez un chronomètre affichant les secondes avec deux décimales.

- Un bouton « Démarrer / Pause ».
- Un bouton « Remise à zéro ».
- Le temps ne doit avancer que lorsque le chronomètre n'est pas en pause. Attention : le `Ticker` continue de battre pendant la pause, c'est à vous de ne pas accumuler.

### Exercice 8 — Le carré qui rebondit en diagonale (moyen)

Reprenez le programme de la section 19.32 et faites bouger le carré **dans les deux dimensions** :

- vitesse horizontale 200 px/s, vitesse verticale 140 px/s ;
- rebond sur les quatre bords ;
- à chaque rebond, le carré change de couleur (par exemple en parcourant une liste de couleurs).

Affichez le nombre total de rebonds.

### Exercice 9 — La pluie de pièces (difficile)

Faites tomber des pièces du haut de l'écran.

- Une pièce est un cercle jaune de 12 pixels de rayon.
- Il y a 12 pièces, réparties horizontalement au hasard.
- Chaque pièce a sa propre vitesse de chute, entre 80 et 220 px/s.
- Quand une pièce sort par le bas, elle réapparaît en haut, à une nouvelle abscisse aléatoire, avec une nouvelle vitesse.

Créez une classe `Piece` (les classes ont été vues au chapitre 08) avec les champs `x`, `y` et `vitesse`, et une méthode `tomber(double dt)`.

### Exercice 10 — Mini-jeu : attraper les pièces (difficile)

Assemblez les exercices 5 et 9 en un jeu complet.

- Un panier : un rectangle ambre de 80 x 20 pixels, qui suit le doigt horizontalement, en bas de l'écran.
- Dix pièces qui tombent, comme à l'exercice 9.
- Quand une pièce touche le panier, le score augmente de 1 et la pièce repart du haut.
- Quand une pièce touche le sol, la partie perd une vie sur trois, et la pièce repart du haut.
- À zéro vie, le jeu s'arrête et affiche « PARTIE TERMINÉE » avec le score final et un bouton « Rejouer ».
- Un HUD affiche en permanence le score et les vies restantes.

Pour la collision, un test rectangle contre rectangle suffit : deux rectangles se touchent si `a.overlaps(b)` (méthode de `Rect`).

---

## 19.38 — Corrections des exercices

Chaque correction est un fichier `lib/main.dart` complet. Vous pouvez la copier telle quelle et lancer `flutter run`.

### Correction 1

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const EcranTitreApp());
}

class EcranTitreApp extends StatelessWidget {
  const EcranTitreApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Donjon de Dart',
      home: EcranTitre(),
    );
  }
}

class EcranTitre extends StatelessWidget {
  const EcranTitre({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'DONJON DE DART',
              style: TextStyle(
                color: Colors.amber,
                fontSize: 40,
                fontWeight: FontWeight.bold,
                letterSpacing: 4,
              ),
            ),
            const SizedBox(height: 16),
            Text(
              'Appuyez pour commencer',
              style: TextStyle(
                color: Colors.white.withValues(alpha: 0.6),
                fontSize: 16,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** l'écran ne contient aucune donnée qui change, donc un `StatelessWidget` suffit (section 19.17). `Center` centre horizontalement et verticalement son unique enfant ; le `Column` doit en plus recevoir `mainAxisAlignment: MainAxisAlignment.center` pour que ses deux enfants soient groupés au milieu de la hauteur plutôt qu'en haut. `SizedBox(height: 16)` sert uniquement d'espaceur : c'est l'usage le plus fréquent de ce widget (section 19.24). Notez que le `Column` n'est pas `const` : le sous-titre appelle `withValues()`, dont le résultat n'est pas connu à la compilation. Le titre, lui, reste `const`, ce qui évite de le reconstruire inutilement (section 19.13.4).

---

### Correction 2

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ScoreApp());
}

class ScoreApp extends StatelessWidget {
  const ScoreApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Compteur de score',
      home: CompteurDeScore(),
    );
  }
}

class CompteurDeScore extends StatefulWidget {
  const CompteurDeScore({super.key});

  @override
  State<CompteurDeScore> createState() => _CompteurDeScoreState();
}

class _CompteurDeScoreState extends State<CompteurDeScore> {
  int _score = 0;

  void _ajouter(int points) {
    setState(() {
      _score += points;
    });
  }

  void _reinitialiser() {
    setState(() {
      _score = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    final Color couleurScore = _score > 100 ? Colors.greenAccent : Colors.amber;

    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      appBar: AppBar(
        title: const Text('Compteur de score'),
        backgroundColor: const Color(0xFF1B1B2A),
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'SCORE',
              style: TextStyle(
                color: Colors.white54,
                fontSize: 18,
                letterSpacing: 6,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              '$_score',
              style: TextStyle(
                color: couleurScore,
                fontSize: 60,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                ElevatedButton(
                  onPressed: () => _ajouter(10),
                  child: const Text('Pièce (+10)'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () => _ajouter(50),
                  child: const Text('Gemme (+50)'),
                ),
              ],
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              // null désactive le bouton : c'est le mécanisme officiel.
              onPressed: _score == 0 ? null : _reinitialiser,
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.redAccent,
                foregroundColor: Colors.white,
              ),
              child: const Text('Réinitialiser'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explication :** le score change, donc il faut un `StatefulWidget` (section 19.19). Toute modification passe par `setState()` : sans cet appel, `_score` changerait bien en mémoire mais l'écran resterait figé (section 19.20). Le bouton « Réinitialiser » illustre la règle vue en 19.26.2 : passer `null` à `onPressed` grise automatiquement le bouton, il est inutile d'écrire une propriété `enabled`. La couleur du score est calculée **dans** `build()`, à partir de l'état : c'est la bonne façon de faire, car `build()` est justement rappelée après chaque `setState()`. Enfin, `() => _ajouter(10)` est une fonction anonyme (chapitre 07) : `onPressed` attend une fonction sans paramètre, on ne peut donc pas écrire `onPressed: _ajouter(10)`, qui appellerait la méthode immédiatement.

---

### Correction 3

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const HudApp());
}

class HudApp extends StatelessWidget {
  const HudApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'HUD du donjon',
      home: PageDonjon(),
    );
  }
}

class PageDonjon extends StatelessWidget {
  const PageDonjon({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            Container(
              width: double.infinity,
              color: const Color(0xFF2A2A40),
              padding: const EdgeInsets.all(12),
              child: const Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: <Widget>[
                  InfoHud(libelle: 'Vies', valeur: '3'),
                  InfoHud(libelle: 'Score', valeur: '250'),
                  InfoHud(libelle: 'Niveau', valeur: '2'),
                ],
              ),
            ),
            const Expanded(
              child: Center(
                child: Text(
                  'Zone de jeu',
                  style: TextStyle(color: Colors.white38, fontSize: 24),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class InfoHud extends StatelessWidget {
  const InfoHud({super.key, required this.libelle, required this.valeur});

  final String libelle;
  final String valeur;

  @override
  Widget build(BuildContext context) {
    return Text(
      '$libelle : $valeur',
      style: const TextStyle(
        color: Colors.amber,
        fontSize: 18,
        fontWeight: FontWeight.w600,
      ),
    );
  }
}
```

**Explication :** la structure est un `Column` de deux éléments. Le premier, le `Container`, prend la hauteur dont il a besoin ; le second est enveloppé dans `Expanded`, ce qui lui donne **tout l'espace restant** (section 19.23). Sans `Expanded`, le `Center` ne saurait pas quelle hauteur occuper. `width: double.infinity` force le `Container` à prendre toute la largeur disponible, sinon il se serrerait autour de sa `Row`. `MainAxisAlignment.spaceEvenly` répartit les trois textes avec des espaces égaux, y compris aux extrémités. Le petit widget `InfoHud` montre l'intérêt d'extraire un `StatelessWidget` réutilisable plutôt que de répéter trois fois le même `Text` stylé (section 19.17.2). `SafeArea` évite que le HUD passe sous l'encoche d'un téléphone.

---

### Correction 4

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const BarreDeVieApp());
}

class BarreDeVieApp extends StatelessWidget {
  const BarreDeVieApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Barre de vie',
      home: PageBarreDeVie(),
    );
  }
}

class PageBarreDeVie extends StatefulWidget {
  const PageBarreDeVie({super.key});

  @override
  State<PageBarreDeVie> createState() => _PageBarreDeVieState();
}

class _PageBarreDeVieState extends State<PageBarreDeVie> {
  double _pourcentage = 1.0;

  void _modifier(double delta) {
    setState(() {
      _pourcentage = (_pourcentage + delta).clamp(0.0, 1.0);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'Vie : ${(_pourcentage * 100).round()} %',
              style: const TextStyle(color: Colors.white70, fontSize: 20),
            ),
            const SizedBox(height: 16),
            CustomPaint(
              size: const Size(300, 24),
              painter: PeintreBarreDeVie(pourcentage: _pourcentage),
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                ElevatedButton(
                  onPressed: () => _modifier(-0.10),
                  child: const Text('Dégâts (-10 %)'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () => _modifier(0.15),
                  child: const Text('Potion (+15 %)'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

class PeintreBarreDeVie extends CustomPainter {
  const PeintreBarreDeVie({required this.pourcentage});

  final double pourcentage;

  @override
  void paint(Canvas canvas, Size size) {
    final Rect cadre = Rect.fromLTWH(0, 0, size.width, size.height);

    // 1. Le fond.
    final Paint fond = Paint()..color = const Color(0xFF2A2A40);
    canvas.drawRect(cadre, fond);

    // 2. La vie restante.
    final Paint vie = Paint()..color = Colors.redAccent;
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width * pourcentage, size.height),
      vie,
    );

    // 3. Le contour.
    final Paint contour = Paint()
      ..color = Colors.white
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;
    canvas.drawRect(cadre, contour);
  }

  @override
  bool shouldRepaint(covariant PeintreBarreDeVie ancien) =>
      ancien.pourcentage != pourcentage;
}
```

**Résultat :**

```text
  Vie : 75 %

  ┌───────────────────────────────────────┐
  │███████████████████████████            │
  └───────────────────────────────────────┘

  [ Dégâts (-10 %) ]   [ Potion (+15 %) ]
```

**Explication :** l'ordre des trois `drawRect` compte : le `Canvas` peint par-dessus ce qui existe déjà (section 19.30). On dessine donc le fond, puis la vie, puis le contour, sinon le contour serait recouvert. `clamp(0.0, 1.0)` empêche la barre de dépasser ou de devenir négative ; c'est une méthode de `num`, vue au chapitre 03. Le `CustomPaint` reçoit une taille explicite `Size(300, 24)`, sans quoi il serait invisible (piège de la section 19.29.5). Enfin, `shouldRepaint` compare l'ancien pourcentage au nouveau : renvoyer `false` figerait la barre, renvoyer toujours `true` fonctionnerait mais redessinerait pour rien.

---

### Correction 5

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const SuivreLeDoigtApp());
}

class SuivreLeDoigtApp extends StatelessWidget {
  const SuivreLeDoigtApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Carré qui suit le doigt',
      home: SuivreLeDoigt(),
    );
  }
}

class SuivreLeDoigt extends StatefulWidget {
  const SuivreLeDoigt({super.key});

  @override
  State<SuivreLeDoigt> createState() => _SuivreLeDoigtState();
}

class _SuivreLeDoigtState extends State<SuivreLeDoigt> {
  static const double cote = 50;

  Offset _centre = const Offset(120, 120);
  Size _zone = Size.zero;

  /// Empêche le carré de sortir de la zone de jeu.
  Offset _limiter(Offset p) {
    final double demi = cote / 2;
    if (_zone == Size.zero) return p;
    return Offset(
      p.dx.clamp(demi, _zone.width - demi),
      p.dy.clamp(demi, _zone.height - demi),
    );
  }

  void _placer(Offset position) {
    setState(() {
      _centre = _limiter(position);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: LayoutBuilder(
          builder: (BuildContext context, BoxConstraints contraintes) {
            _zone = Size(contraintes.maxWidth, contraintes.maxHeight);

            return GestureDetector(
              // Sans cette ligne, les zones vides n'attrapent pas le geste.
              behavior: HitTestBehavior.opaque,
              onPanDown: (DragDownDetails d) => _placer(d.localPosition),
              onPanUpdate: (DragUpdateDetails d) => _placer(d.localPosition),
              child: Stack(
                children: <Widget>[
                  CustomPaint(
                    size: Size.infinite,
                    painter: PeintreCurseur(centre: _centre, cote: cote),
                  ),
                  Positioned(
                    left: 12,
                    top: 12,
                    child: Text(
                      'x = ${_centre.dx.toStringAsFixed(0)}   '
                      'y = ${_centre.dy.toStringAsFixed(0)}',
                      style: const TextStyle(
                        color: Colors.white70,
                        fontSize: 16,
                      ),
                    ),
                  ),
                ],
              ),
            );
          },
        ),
      ),
    );
  }
}

class PeintreCurseur extends CustomPainter {
  const PeintreCurseur({required this.centre, required this.cote});

  final Offset centre;
  final double cote;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint carre = Paint()..color = Colors.amber;

    // Rect.fromCenter place le rectangle PAR SON CENTRE.
    canvas.drawRect(
      Rect.fromCenter(center: centre, width: cote, height: cote),
      carre,
    );

    // Une croix de visée, pour bien voir le point exact.
    final Paint croix = Paint()
      ..color = Colors.white24
      ..strokeWidth = 1;
    canvas.drawLine(Offset(0, centre.dy), Offset(size.width, centre.dy), croix);
    canvas.drawLine(Offset(centre.dx, 0), Offset(centre.dx, size.height), croix);
  }

  @override
  bool shouldRepaint(covariant PeintreCurseur ancien) => ancien.centre != centre;
}
```

**Explication :** trois points méritent l'attention.

D'abord `localPosition` et non `globalPosition` : `localPosition` donne les coordonnées **relatives au `GestureDetector`**, ce qui est exactement ce dont le `CustomPainter` a besoin. `globalPosition` inclurait la barre de statut et toute marge au-dessus, et le carré serait décalé (section 19.27.2).

Ensuite `Rect.fromCenter` : c'est ce qui rend le carré centré sur le doigt. Avec `Rect.fromLTWH(x, y, 50, 50)`, le doigt se trouverait au coin supérieur gauche et le carré paraîtrait décalé de 25 pixels vers le bas à droite.

Enfin `HitTestBehavior.opaque` : sans lui, un `GestureDetector` dont l'enfant ne peint rien à l'endroit touché laisse passer le geste (section 19.27.3). `onPanDown` traite le premier contact et `onPanUpdate` le glissement ; les deux sont nécessaires, sinon le carré ne bougerait qu'après un début de déplacement.

---

### Correction 6

```dart
import 'dart:math';

import 'package:flutter/material.dart';

void main() {
  runApp(const CibleApp());
}

class CibleApp extends StatelessWidget {
  const CibleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'La cible qui se déplace',
      home: PageCible(),
    );
  }
}

class PageCible extends StatefulWidget {
  const PageCible({super.key});

  @override
  State<PageCible> createState() => _PageCibleState();
}

class _PageCibleState extends State<PageCible> {
  static const double rayon = 30;

  final Random _hasard = Random();

  Offset _cible = const Offset(150, 150);
  Size _zone = Size.zero;
  int _score = 0;
  int _touches = 0;
  int _rates = 0;

  void _deplacerCible() {
    if (_zone == Size.zero) return;
    _cible = Offset(
      rayon + _hasard.nextDouble() * (_zone.width - 2 * rayon),
      rayon + _hasard.nextDouble() * (_zone.height - 2 * rayon),
    );
  }

  void _surTap(Offset point) {
    // La distance entre deux Offset est fournie par Flutter.
    final double distance = (point - _cible).distance;

    setState(() {
      if (distance <= rayon) {
        _score++;
        _touches++;
        _deplacerCible();
      } else {
        _score = max(0, _score - 1);
        _rates++;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            Container(
              width: double.infinity,
              color: const Color(0xFF2A2A40),
              padding: const EdgeInsets.all(12),
              child: Text(
                'Score : $_score      Touches : $_touches      Ratés : $_rates',
                textAlign: TextAlign.center,
                style: const TextStyle(color: Colors.amber, fontSize: 18),
              ),
            ),
            Expanded(
              child: LayoutBuilder(
                builder: (BuildContext context, BoxConstraints contraintes) {
                  _zone = Size(contraintes.maxWidth, contraintes.maxHeight);
                  return GestureDetector(
                    behavior: HitTestBehavior.opaque,
                    onTapDown: (TapDownDetails d) => _surTap(d.localPosition),
                    child: CustomPaint(
                      size: Size.infinite,
                      painter: PeintreCible(centre: _cible, rayon: rayon),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class PeintreCible extends CustomPainter {
  const PeintreCible({required this.centre, required this.rayon});

  final Offset centre;
  final double rayon;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint plein = Paint()..color = Colors.greenAccent;
    final Paint anneau = Paint()
      ..color = Colors.white70
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3;

    canvas.drawCircle(centre, rayon, plein);
    canvas.drawCircle(centre, rayon, anneau);
    canvas.drawCircle(centre, rayon / 3, anneau);
  }

  @override
  bool shouldRepaint(covariant PeintreCible ancien) => ancien.centre != centre;
}
```

**Explication :** le test « le point touché est-il dans le cercle ? » se ramène à une seule ligne : `(point - _cible).distance <= rayon`. La soustraction de deux `Offset` donne un `Offset` (le vecteur entre les deux points) et sa propriété `distance` en donne la longueur. C'est le test de collision cercle/point, revu en détail au chapitre 24.

Le tirage aléatoire est encadré : `rayon + hasard * (largeur - 2 * rayon)` garantit que le cercle entier reste visible. Sans cette précaution, une cible tirée en `x = 3` serait coupée par le bord gauche.

`Random` vient de `dart:math` : c'est un import de la bibliothèque standard Dart, pas de Flutter. `max(0, _score - 1)` empêche un score négatif ; c'est la même fonction `max` que celle du chapitre 03.

Notez enfin que `_deplacerCible()` ne contient **pas** d'appel à `setState()` : elle est appelée depuis l'intérieur du `setState()` de `_surTap`. Imbriquer deux `setState()` provoquerait un appel inutile ; on regroupe donc toutes les modifications dans un seul.

---

### Correction 7

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const ChronoApp());
}

class ChronoApp extends StatelessWidget {
  const ChronoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Chronomètre du donjon',
      home: PageChrono(),
    );
  }
}

class PageChrono extends StatefulWidget {
  const PageChrono({super.key});

  @override
  State<PageChrono> createState() => _PageChronoState();
}

class _PageChronoState extends State<PageChrono>
    with SingleTickerProviderStateMixin {
  late final Ticker _ticker;

  Duration _precedent = Duration.zero;
  double _secondes = 0;
  bool _enMarche = false;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;

    // Le Ticker bat toujours ; c'est ICI que la pause se décide.
    if (!_enMarche || dt <= 0) return;

    setState(() {
      _secondes += dt;
    });
  }

  void _basculer() {
    setState(() => _enMarche = !_enMarche);
  }

  void _remiseAZero() {
    setState(() {
      _secondes = 0;
      _enMarche = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              '${_secondes.toStringAsFixed(2)} s',
              style: TextStyle(
                color: _enMarche ? Colors.amber : Colors.white54,
                fontSize: 64,
                fontWeight: FontWeight.bold,
                fontFeatures: const <FontFeature>[
                  FontFeature.tabularFigures(),
                ],
              ),
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                ElevatedButton(
                  onPressed: _basculer,
                  child: Text(_enMarche ? 'Pause' : 'Démarrer'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: _secondes == 0 ? null : _remiseAZero,
                  child: const Text('Remise à zéro'),
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

**Explication :** le piège de cet exercice est la pause. Le `Ticker` continue de battre même en pause ; si l'on se contentait de cumuler `ecoule` sans traitement, le temps mis en pause serait quand même compté au redémarrage. La solution est de toujours mettre `_precedent` à jour, **puis** de sortir par un `return` quand le chronomètre est arrêté. Ainsi, l'intervalle écoulé pendant la pause est bien mesuré, mais il n'est pas ajouté.

Regardez l'ordre des lignes : `_precedent = ecoule;` est **avant** le `return`. Si on l'écrivait après, au redémarrage `dt` vaudrait toute la durée de la pause et le chronomètre ferait un bond.

`FontFeature.tabularFigures()` demande des chiffres de largeur fixe : sans lui, le texte tressaute en largeur à chaque changement de chiffre. C'est un détail, mais tout compteur affiché en temps réel en profite.

---

### Correction 8

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const RebondApp());
}

class RebondApp extends StatelessWidget {
  const RebondApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Carré qui rebondit',
      home: PageRebond(),
    );
  }
}

class PageRebond extends StatefulWidget {
  const PageRebond({super.key});

  @override
  State<PageRebond> createState() => _PageRebondState();
}

class _PageRebondState extends State<PageRebond>
    with SingleTickerProviderStateMixin {
  static const double cote = 40;
  static const double vitesseX = 200;
  static const double vitesseY = 140;

  static const List<Color> palette = <Color>[
    Colors.amber,
    Colors.greenAccent,
    Colors.lightBlueAccent,
    Colors.pinkAccent,
    Colors.orangeAccent,
  ];

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  double _x = 40;
  double _y = 40;
  int _sensX = 1;
  int _sensY = 1;
  int _rebonds = 0;
  int _indexCouleur = 0;

  Size _zone = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;
    if (dt <= 0 || _zone == Size.zero) return;

    setState(() => _update(dt));
  }

  void _rebondir() {
    _rebonds++;
    _indexCouleur = (_indexCouleur + 1) % palette.length;
  }

  void _update(double dt) {
    _x += vitesseX * _sensX * dt;
    _y += vitesseY * _sensY * dt;

    final double xMax = _zone.width - cote;
    final double yMax = _zone.height - cote;

    if (_x <= 0) {
      _x = 0;
      _sensX = 1;
      _rebondir();
    } else if (_x >= xMax) {
      _x = xMax;
      _sensX = -1;
      _rebondir();
    }

    if (_y <= 0) {
      _y = 0;
      _sensY = 1;
      _rebondir();
    } else if (_y >= yMax) {
      _y = yMax;
      _sensY = -1;
      _rebondir();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: Stack(
          children: <Widget>[
            LayoutBuilder(
              builder: (BuildContext context, BoxConstraints contraintes) {
                _zone = Size(contraintes.maxWidth, contraintes.maxHeight);
                return CustomPaint(
                  size: Size.infinite,
                  painter: PeintreRebond(
                    x: _x,
                    y: _y,
                    cote: cote,
                    couleur: palette[_indexCouleur],
                  ),
                );
              },
            ),
            Positioned(
              left: 12,
              top: 12,
              child: Text(
                'Rebonds : $_rebonds',
                style: const TextStyle(color: Colors.white70, fontSize: 18),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class PeintreRebond extends CustomPainter {
  const PeintreRebond({
    required this.x,
    required this.y,
    required this.cote,
    required this.couleur,
  });

  final double x;
  final double y;
  final double cote;
  final Color couleur;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint bordure = Paint()
      ..color = Colors.white12
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;
    canvas.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), bordure);

    final Paint carre = Paint()..color = couleur;
    canvas.drawRect(Rect.fromLTWH(x, y, cote, cote), carre);
  }

  @override
  bool shouldRepaint(covariant PeintreRebond ancien) =>
      ancien.x != x || ancien.y != y || ancien.couleur != couleur;
}
```

**Explication :** le passage en deux dimensions ne change rien au principe : on applique la même formule `position += vitesse * sens * dt` sur `x` et sur `y`, indépendamment. Les deux tests de rebond sont écrits l'un après l'autre, sans `else`, car un carré peut toucher un coin et rebondir sur les deux axes dans la même image.

Le recalage `_x = 0;` avant l'inversion du sens est indispensable, pour la raison expliquée en 19.32.4 : sans lui, le carré resterait hors zone une image de plus et changerait de sens en boucle, donnant une vibration.

Le changement de couleur utilise le modulo : `(_indexCouleur + 1) % palette.length` revient à zéro après le dernier élément. C'est le motif classique de parcours cyclique d'une liste (chapitre 06), et c'est exactement celui qui servira à faire défiler les images d'une animation au chapitre 22.

---

### Correction 9

```dart
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const PluieApp());
}

class PluieApp extends StatelessWidget {
  const PluieApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Pluie de pièces',
      home: PagePluie(),
    );
  }
}

/// Une pièce qui tombe. Classe simple : voir le chapitre 08.
class Piece {
  Piece({required this.x, required this.y, required this.vitesse});

  double x;
  double y;
  double vitesse; // pixels par seconde

  void tomber(double dt) {
    y += vitesse * dt;
  }
}

class PagePluie extends StatefulWidget {
  const PagePluie({super.key});

  @override
  State<PagePluie> createState() => _PagePluieState();
}

class _PagePluieState extends State<PagePluie>
    with SingleTickerProviderStateMixin {
  static const int nombreDePieces = 12;
  static const double rayon = 12;

  final Random _hasard = Random();
  final List<Piece> _pieces = <Piece>[];

  late final Ticker _ticker;
  Duration _precedent = Duration.zero;
  Size _zone = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  /// Crée les pièces la première fois que la taille de la zone est connue.
  void _creerSiNecessaire() {
    if (_pieces.isNotEmpty || _zone == Size.zero) return;
    for (int i = 0; i < nombreDePieces; i++) {
      _pieces.add(
        Piece(
          x: _abscisseAleatoire(),
          y: -_hasard.nextDouble() * _zone.height,
          vitesse: _vitesseAleatoire(),
        ),
      );
    }
  }

  double _abscisseAleatoire() =>
      rayon + _hasard.nextDouble() * (_zone.width - 2 * rayon);

  double _vitesseAleatoire() => 80 + _hasard.nextDouble() * 140; // 80 à 220

  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;
    if (dt <= 0 || _zone == Size.zero) return;

    setState(() {
      _creerSiNecessaire();
      for (final Piece p in _pieces) {
        p.tomber(dt);
        if (p.y - rayon > _zone.height) {
          // Elle est sortie par le bas : on la recycle en haut.
          p.y = -rayon;
          p.x = _abscisseAleatoire();
          p.vitesse = _vitesseAleatoire();
        }
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: LayoutBuilder(
        builder: (BuildContext context, BoxConstraints contraintes) {
          _zone = Size(contraintes.maxWidth, contraintes.maxHeight);
          return CustomPaint(
            size: Size.infinite,
            painter: PeintrePluie(pieces: _pieces, rayon: rayon),
          );
        },
      ),
    );
  }
}

class PeintrePluie extends CustomPainter {
  const PeintrePluie({required this.pieces, required this.rayon});

  final List<Piece> pieces;
  final double rayon;

  @override
  void paint(Canvas canvas, Size size) {
    final Paint or = Paint()..color = const Color(0xFFFFD54F);
    final Paint reflet = Paint()..color = const Color(0xFFFFF8E1);

    for (final Piece p in pieces) {
      final Offset centre = Offset(p.x, p.y);
      canvas.drawCircle(centre, rayon, or);
      canvas.drawCircle(centre.translate(-rayon / 3, -rayon / 3), rayon / 4, reflet);
    }
  }

  @override
  bool shouldRepaint(covariant PeintrePluie ancien) => true;
}
```

**Explication :** c'est le premier programme à plusieurs entités, et il introduit le schéma que tous les chapitres suivants réutilisent :

```text
  une classe = une entité   ->  Piece { x, y, vitesse, tomber(dt) }
  une List<Entite>          ->  _pieces
  une boucle dans update    ->  for (p in _pieces) p.tomber(dt);
  une boucle dans paint     ->  for (p in _pieces) canvas.drawCircle(...);
```

Le recyclage remplace la suppression : plutôt que de retirer la pièce de la liste et d'en créer une autre, on remet simplement ses coordonnées en haut. C'est plus rapide et cela évite la `ConcurrentModificationError` que provoquerait un `remove()` pendant un `for` (chapitre 13). Le principe porte un nom, l'*object pooling*, et Flame le propose sous le nom de `ComponentPool`.

La création est différée dans `_creerSiNecessaire()` : au premier appel du `Ticker`, la mise en page n'a pas encore eu lieu et `_zone` vaut `Size.zero` ; on ne peut donc pas tirer d'abscisse aléatoire dans `initState()`. Le garde `if (_pieces.isNotEmpty || _zone == Size.zero) return;` assure que la création n'a lieu qu'une seule fois, au bon moment.

Enfin, `shouldRepaint` renvoie `true` : les positions sont modifiées **dans** les objets `Piece`, la liste elle-même reste la même instance, une comparaison `ancien.pieces != pieces` renverrait donc toujours `false` et l'écran se figerait. C'est un piège classique dès que l'on peint des objets mutables.

---

### Correction 10

```dart
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const AttrapePiecesApp());
}

class AttrapePiecesApp extends StatelessWidget {
  const AttrapePiecesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Attrape-pièces',
      home: Jeu(),
    );
  }
}

// ===========================================================
//  L'ENTITÉ
// ===========================================================

class Piece {
  Piece({required this.x, required this.y, required this.vitesse});

  double x;
  double y;
  double vitesse;

  static const double rayon = 12;

  void tomber(double dt) {
    y += vitesse * dt;
  }

  /// La boîte englobante, utilisée pour la collision.
  Rect get boite => Rect.fromCircle(center: Offset(x, y), radius: rayon);
}

// ===========================================================
//  LE JEU
// ===========================================================

class Jeu extends StatefulWidget {
  const Jeu({super.key});

  @override
  State<Jeu> createState() => _JeuState();
}

class _JeuState extends State<Jeu> with SingleTickerProviderStateMixin {
  // --- Constantes de réglage ------------------------------------------
  static const int nombreDePieces = 10;
  static const double largeurPanier = 80;
  static const double hauteurPanier = 20;
  static const double margeBas = 40;
  static const int viesDepart = 3;

  // --- Moteur ----------------------------------------------------------
  late final Ticker _ticker;
  Duration _precedent = Duration.zero;

  // --- Monde -----------------------------------------------------------
  final Random _hasard = Random();
  final List<Piece> _pieces = <Piece>[];

  double _panierX = 0;
  int _score = 0;
  int _vies = viesDepart;
  bool _termine = false;

  Size _zone = Size.zero;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker(_surChaqueImage)..start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  // --- Création et recyclage -------------------------------------------

  double _abscisseAleatoire() =>
      Piece.rayon + _hasard.nextDouble() * (_zone.width - 2 * Piece.rayon);

  double _vitesseAleatoire() => 90 + _hasard.nextDouble() * 130;

  void _preparerSiNecessaire() {
    if (_pieces.isNotEmpty || _zone == Size.zero) return;
    _panierX = _zone.width / 2;
    for (int i = 0; i < nombreDePieces; i++) {
      _pieces.add(
        Piece(
          x: _abscisseAleatoire(),
          y: -_hasard.nextDouble() * _zone.height,
          vitesse: _vitesseAleatoire(),
        ),
      );
    }
  }

  void _recycler(Piece p) {
    p.y = -Piece.rayon;
    p.x = _abscisseAleatoire();
    p.vitesse = _vitesseAleatoire();
  }

  // --- Le panier --------------------------------------------------------

  Rect get _boitePanier => Rect.fromCenter(
        center: Offset(_panierX, _zone.height - margeBas),
        width: largeurPanier,
        height: hauteurPanier,
      );

  void _deplacerPanier(double x) {
    if (_termine || _zone == Size.zero) return;
    setState(() {
      _panierX = x.clamp(largeurPanier / 2, _zone.width - largeurPanier / 2);
    });
  }

  // --- La boucle --------------------------------------------------------

  void _surChaqueImage(Duration ecoule) {
    final double dt =
        (ecoule - _precedent).inMicroseconds / Duration.microsecondsPerSecond;
    _precedent = ecoule;
    if (dt <= 0 || _zone == Size.zero) return;

    setState(() {
      _preparerSiNecessaire();
      if (!_termine) {
        _update(dt);
      }
    });
  }

  void _update(double dt) {
    final Rect panier = _boitePanier;

    for (final Piece p in _pieces) {
      p.tomber(dt);

      // 1. Attrapée ?
      if (p.boite.overlaps(panier)) {
        _score++;
        _recycler(p);
        continue;
      }

      // 2. Tombée au sol ?
      if (p.y - Piece.rayon > _zone.height) {
        _vies--;
        _recycler(p);
        if (_vies <= 0) {
          _vies = 0;
          _termine = true;
        }
      }
    }
  }

  void _rejouer() {
    setState(() {
      _score = 0;
      _vies = viesDepart;
      _termine = false;
      for (final Piece p in _pieces) {
        _recycler(p);
        p.y = -_hasard.nextDouble() * _zone.height;
      }
    });
  }

  // --- L'affichage ------------------------------------------------------

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF101018),
      body: SafeArea(
        child: Column(
          children: <Widget>[
            _construireHud(),
            Expanded(
              child: LayoutBuilder(
                builder: (BuildContext context, BoxConstraints contraintes) {
                  _zone = Size(contraintes.maxWidth, contraintes.maxHeight);

                  return GestureDetector(
                    behavior: HitTestBehavior.opaque,
                    onPanDown: (DragDownDetails d) =>
                        _deplacerPanier(d.localPosition.dx),
                    onPanUpdate: (DragUpdateDetails d) =>
                        _deplacerPanier(d.localPosition.dx),
                    child: Stack(
                      children: <Widget>[
                        CustomPaint(
                          size: Size.infinite,
                          painter: PeintreJeu(
                            pieces: _pieces,
                            panier: _zone == Size.zero ? Rect.zero : _boitePanier,
                          ),
                        ),
                        if (_termine) _construireEcranFin(),
                      ],
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _construireHud() {
    return Container(
      width: double.infinity,
      color: const Color(0xFF2A2A40),
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: <Widget>[
          Text(
            'Score : $_score',
            style: const TextStyle(
              color: Colors.amber,
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          Row(
            children: List<Widget>.generate(
              viesDepart,
              (int i) => Padding(
                padding: const EdgeInsets.only(left: 6),
                child: Icon(
                  Icons.favorite,
                  size: 22,
                  color: i < _vies ? Colors.redAccent : Colors.white24,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _construireEcranFin() {
    return Container(
      color: Colors.black.withValues(alpha: 0.75),
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const Text(
              'PARTIE TERMINÉE',
              style: TextStyle(
                color: Colors.redAccent,
                fontSize: 34,
                fontWeight: FontWeight.bold,
                letterSpacing: 3,
              ),
            ),
            const SizedBox(height: 12),
            Text(
              'Score final : $_score',
              style: const TextStyle(color: Colors.white, fontSize: 22),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _rejouer,
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.amber,
                foregroundColor: Colors.black,
                padding: const EdgeInsets.symmetric(
                  horizontal: 28,
                  vertical: 14,
                ),
              ),
              child: const Text('Rejouer', style: TextStyle(fontSize: 18)),
            ),
          ],
        ),
      ),
    );
  }
}

// ===========================================================
//  LE RENDU
// ===========================================================

class PeintreJeu extends CustomPainter {
  const PeintreJeu({required this.pieces, required this.panier});

  final List<Piece> pieces;
  final Rect panier;

  @override
  void paint(Canvas canvas, Size size) {
    // Le sol.
    final Paint sol = Paint()..color = const Color(0xFF20203A);
    canvas.drawRect(Rect.fromLTWH(0, size.height - 16, size.width, 16), sol);

    // Les pièces.
    final Paint or = Paint()..color = const Color(0xFFFFD54F);
    final Paint reflet = Paint()..color = const Color(0xFFFFF8E1);
    for (final Piece p in pieces) {
      final Offset centre = Offset(p.x, p.y);
      canvas.drawCircle(centre, Piece.rayon, or);
      canvas.drawCircle(
        centre.translate(-Piece.rayon / 3, -Piece.rayon / 3),
        Piece.rayon / 4,
        reflet,
      );
    }

    // Le panier.
    if (panier != Rect.zero) {
      final Paint bois = Paint()..color = Colors.amber;
      canvas.drawRRect(
        RRect.fromRectAndRadius(panier, const Radius.circular(6)),
        bois,
      );
    }
  }

  @override
  bool shouldRepaint(covariant PeintreJeu ancien) => true;
}
```

**Résultat :**

```text
  ┌──────────────────────────────────────────────────┐
  │ Score : 14                        vies 2/3          │  le HUD
  ├──────────────────────────────────────────────────┤
  │        ●                                         │
  │                    ●            ●                │
  │   ●                                              │
  │                          ●                       │
  │                     ▬▬▬▬▬                        │  le panier
  │ ════════════════════════════════════════════════ │  le sol
  └──────────────────────────────────────────────────┘
```

**Explication :** ce programme est un jeu complet en cent quatre-vingts lignes, et il contient déjà tous les éléments d'architecture des chapitres suivants.

**La séparation logique / rendu.** `_update(dt)` modifie l'état et ne dessine rien ; `PeintreJeu.paint()` dessine et ne modifie rien. C'est exactement le couple `update()` / `render()` de Flame (chapitre 27).

**La collision.** `Rect.overlaps` est un test AABB (*axis-aligned bounding box*), c'est-à-dire un chevauchement de deux rectangles alignés sur les axes. C'est le test de collision le plus utilisé en jeu 2D, et le chapitre 24 lui est consacré. Notez que la pièce est un cercle mais que sa *hitbox* est un carré : la hitbox n'a pas à correspondre exactement à la forme dessinée, et c'est même rarement souhaitable.

**Le `continue`.** Après avoir attrapé une pièce, on saute immédiatement au tour de boucle suivant. Sans ce `continue`, une pièce attrapée serait recyclée en haut avec `y = -12`, et le test suivant (`p.y - rayon > _zone.height`) serait faux, donc en pratique rien ne casserait — mais la logique deviendrait fragile dès qu'on ajouterait un troisième cas. Une pièce n'a le droit qu'à un seul sort par image.

**L'écran de fin.** Il est superposé grâce au `Stack` et au `if (_termine)` dans la liste des enfants. La syntaxe `if` dans un littéral de liste est celle du chapitre 06 (les *collection if*). Le `Container` semi-transparent qui l'entoure assombrit le jeu sans le masquer. Dans Flame, ce rôle est tenu par les `overlays` du `GameWidget` (chapitre 35), qui sont de véritables widgets Flutter placés au-dessus du jeu : ce que vous venez d'écrire à la main.

**Les vies.** `List<Widget>.generate(3, ...)` construit trois icônes de cœur ; celles dont l'indice dépasse le nombre de vies sont grisées. C'est la version « widget » du constructeur `List.generate` du chapitre 06.

**Le garde `_zone == Size.zero`.** Tant que la mise en page n'a pas eu lieu, la zone est inconnue : on ne peut ni placer le panier, ni tirer une abscisse, ni tester une sortie d'écran. Tous les points d'entrée (`_deplacerPanier`, `_surChaqueImage`, `_preparerSiNecessaire`) commencent donc par vérifier ce cas. Oublier ce garde produit des `NaN` qui se propagent silencieusement dans toutes les positions : le jeu s'affiche vide, sans message d'erreur. C'est un bug très difficile à trouver, et c'est pour cela qu'on l'écrit dès le premier jour.

---

## Et maintenant ?

Vous savez maintenant tout ce qu'il faut de Flutter pour faire un jeu : un widget qui occupe l'écran, un état qui change, un `Ticker` qui bat soixante fois par seconde, un `Canvas` sur lequel dessiner, et un `GestureDetector` pour écouter le joueur.

Vous avez aussi vu la faiblesse de ce que nous venons d'écrire. Notre boucle est naïve : elle fait confiance au `dt` qu'on lui donne. Que se passe-t-il si l'application se fige une demi-seconde parce que le système a lancé une autre tâche ? Le `dt` vaut alors 0,5 seconde, le joueur avance de cent pixels d'un coup, traverse un mur, et le jeu devient injouable. Que se passe-t-il sur un écran à 120 Hz ? Et sur un vieux téléphone qui peine à 30 images par seconde ?

Le chapitre 20 répond à ces questions. Il traite de la **boucle de jeu** proprement dite : ce qu'est réellement un `dt`, comment mesurer des images par seconde fiables, pourquoi il faut plafonner le `dt`, ce qu'est un pas de temps fixe et quand il devient indispensable, comment mettre le jeu en pause ou au ralenti sans rien casser. À la fin de ce chapitre, vous disposerez d'un petit moteur de boucle réutilisable, que les chapitres 21 à 26 rempliront.

C'est la première brique d'un vrai moteur de jeu, et vous allez l'écrire vous-même.

[20-PARTIE-2A—GAME-LOOP-FPS-ET-DELTA-TIME.md](./20-PARTIE-2A—GAME-LOOP-FPS-ET-DELTA-TIME.md)
