# PARTIE 1C — MINI-PROJETS FLUTTER
# CHAPITRE 59 — PROJET 5 : LE QUIZ

> **Niveau :** intermédiaire
> **Durée estimée :** 12 h
> **Pré-requis :** PARTIE 1A (chapitres 01 à 18), PARTIE 1B (chapitres 43 à 54), et les projets 55 à 58
> **Ce que vous saurez faire à la fin :** construire une application de quiz complète — banque de questions en JSON embarquée dans les assets, chargement asynchrone, moteur de quiz pur et testable, compte à rebours par question, retour visuel immédiat, transitions animées, écran de résultat détaillé, meilleur score persistant et tests automatisés.

---

## 59.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- modéliser une question à choix multiples avec un `enum` de difficulté (rappel du chapitre 11) ;
- écrire un `fromJson` défensif qui refuse une question mal formée (rappels des chapitres 13 et 17) ;
- ranger une banque de questions au format JSON dans les assets et la déclarer dans `pubspec.yaml` (rappel du chapitre 47) ;
- charger un asset texte de façon asynchrone avec `rootBundle.loadString` (rappel du chapitre 15) ;
- écrire un moteur de quiz sous forme de classe pure, sans aucune dépendance à Flutter ;
- afficher un écran d'accueil avec choix de la catégorie et du nombre de questions ;
- construire un écran de question lisible et réutilisable ;
- écrire un bouton de réponse qui change d'apparence selon cinq états ;
- donner un retour visuel immédiat, vert ou rouge, sans jamais bloquer l'interface ;
- afficher une explication après chaque réponse ;
- animer une barre de progression avec `TweenAnimationBuilder` ;
- gérer un compte à rebours par question avec `Timer.periodic` (rappel du chapitre 45) ;
- traiter l'expiration du temps comme une réponse à part entière ;
- enchaîner les questions avec une transition `AnimatedSwitcher` ;
- mélanger les questions et les réponses avec `shuffle`, et rendre ce mélange testable ;
- construire un écran de résultat avec un message adapté au score ;
- afficher un récapitulatif question par question ;
- persister le meilleur score par catégorie avec `shared_preferences` (rappel du chapitre 54) ;
- déplacer tout l'état, minuteur compris, dans un `ChangeNotifier` exposé par `provider` (rappel du chapitre 52) ;
- rendre l'application utilisable au lecteur d'écran et correcte en mode sombre (rappel du chapitre 51) ;
- écrire des tests unitaires sur la logique de scoring et sur le mélange.

---

## 59.0.1 — Aperçu du résultat final

Voici l'application terminée. L'écran d'accueil, où l'on choisit ce que l'on va jouer :

```text
┌────────────────────────────────────────────────┐
│  Quiz                                    [◐]   │
├────────────────────────────────────────────────┤
│                                                │
│                   ▣ QUIZ                       │
│         18 questions, 3 catégories             │
│                                                │
│  Catégorie                                     │
│  ┌──────────────────────────────────────────┐  │
│  │ (•) Toutes les catégories        18 q.   │  │
│  ├──────────────────────────────────────────┤  │
│  │ ( ) Dart                          6 q.   │  │
│  ├──────────────────────────────────────────┤  │
│  │ ( ) Flutter                       6 q.   │  │
│  ├──────────────────────────────────────────┤  │
│  │ ( ) Jeu vidéo                     6 q.   │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Nombre de questions                           │
│  [ 5 ]   [ 10 ]   [ Toutes ]                   │
│                                                │
│  Meilleur score : 14 / 20                      │
│                                                │
│         ┌──────────────────────────────┐       │
│         │         COMMENCER            │       │
│         └──────────────────────────────┘       │
└────────────────────────────────────────────────┘
```

L'écran de question, avant que l'on ait répondu :

```text
┌────────────────────────────────────────────────┐
│  ✕   Question 3 / 10                    7 pts  │
├────────────────────────────────────────────────┤
│  ███████████░░░░░░░░░░░░░░░░░░░░░░░             │
├────────────────────────────────────────────────┤
│                                       ┌─────┐  │
│  DART · MOYENNE                       │ 11  │  │
│                                       └─────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │                                          │  │
│  │  Quel type de valeur renvoie une         │  │
│  │  fonction déclarée `async` ?             │  │
│  │                                          │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ A   Un Stream                            │  │
│  ├──────────────────────────────────────────┤  │
│  │ B   Un Future                            │  │
│  ├──────────────────────────────────────────┤  │
│  │ C   void                                 │  │
│  ├──────────────────────────────────────────┤  │
│  │ D   Un Iterable                          │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

Le même écran juste après un mauvais choix. La bonne réponse est révélée, l'explication apparaît, le minuteur s'arrête :

```text
┌────────────────────────────────────────────────┐
│  ✕   Question 3 / 10                    7 pts  │
├────────────────────────────────────────────────┤
│  ███████████░░░░░░░░░░░░░░░░░░░░░░░             │
├────────────────────────────────────────────────┤
│  DART · MOYENNE                       ┌─────┐  │
│                                       │  9  │  │
│  ┌──────────────────────────────────┐ └─────┘  │
│  │  Quel type de valeur renvoie une │          │
│  │  fonction déclarée `async` ?      │         │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ A   Un Stream                        ✕   │  │  <- rouge
│  ├──────────────────────────────────────────┤  │
│  │ B   Un Future                        ✓   │  │  <- vert
│  ├──────────────────────────────────────────┤  │
│  │ C   void                                 │  │
│  ├──────────────────────────────────────────┤  │
│  │ D   Un Iterable                          │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ (i)  Toute fonction `async` renvoie un   │  │
│  │      Future. Le `return` alimente ce     │  │
│  │      Future ; il ne renvoie pas la       │  │
│  │      valeur directement.                 │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│         ┌──────────────────────────────┐       │
│         │        QUESTION SUIVANTE     │       │
│         └──────────────────────────────┘       │
└────────────────────────────────────────────────┘
```

Quand le compte à rebours atteint zéro sans réponse :

```text
│  ┌──────────────────────────────────────────┐  │
│  │ B   Un Future                        ✓   │  │  <- vert, révélée
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ (!)  Temps écoulé. La bonne réponse      │  │
│  │      était : Un Future.                  │  │
│  └──────────────────────────────────────────┘  │
```

L'écran de résultat :

```text
┌────────────────────────────────────────────────┐
│  Résultat                                      │
├────────────────────────────────────────────────┤
│                                                │
│                     ★                          │
│                 14 / 20 pts                    │
│                                                │
│           8 bonnes réponses sur 10             │
│                                                │
│  ██████████████████████░░░░░░░░░░░  70 %       │
│                                                │
│           Très bien. Vous maîtrisez.           │
│                                                │
│  NOUVEAU RECORD (ancien : 11)                  │
│                                                │
│  Récapitulatif                                 │
│  ┌──────────────────────────────────────────┐  │
│  │ ✓ 1. Quel mot-clé déclare une constante… │  │
│  │      Votre réponse : const                │ │
│  ├──────────────────────────────────────────┤  │
│  │ ✕ 2. Que vaut 2 + 3 * 4 en Dart ?        │  │
│  │      Votre réponse : 20                   │ │
│  │      Bonne réponse : 14                   │ │
│  ├──────────────────────────────────────────┤  │
│  │ (t)3. Quel type renvoie une fonction…     │  │
│  │      Temps écoulé                         │ │
│  └──────────────────────────────────────────┘  │
│                                                │
│    ┌──────────────┐    ┌──────────────┐        │
│    │   REJOUER    │    │   ACCUEIL    │        │
│    └──────────────┘    └──────────────┘        │
└────────────────────────────────────────────────┘
```

---

## 59.0.2 — Cahier des charges

Comme au chapitre 58, on écrit d'abord ce que l'application doit faire. Une exigence non écrite est une exigence qui sera oubliée.

### Fonctionnalités obligatoires

| # | Exigence | Vérification |
| --- | --- | --- |
| O1 | Une question porte un énoncé, une liste de réponses, l'indice de la bonne, une explication, une catégorie et une difficulté. | Le modèle compile et se relit depuis le JSON. |
| O2 | Les questions sont stockées dans un fichier JSON embarqué dans les assets. | Modifier le JSON change le quiz sans recompiler le code Dart. |
| O3 | Le chargement des questions est asynchrone et affiche un indicateur. | Un écran de chargement apparaît au démarrage. |
| O4 | Une donnée JSON invalide est signalée, pas ignorée en silence. | Corrompre le fichier affiche un écran d'erreur lisible. |
| O5 | L'utilisateur choisit une catégorie et un nombre de questions. | Le quiz ne contient que la catégorie choisie. |
| O6 | Les questions et les réponses sont mélangées à chaque partie. | Deux parties de suite n'ont pas le même ordre. |
| O7 | Un appui sur une réponse verrouille la question et révèle la bonne. | Un second appui ne change rien. |
| O8 | Le retour visuel est immédiat : vert pour juste, rouge pour faux. | Aucune attente, aucun dialogue modal. |
| O9 | L'explication de la question s'affiche après la réponse. | Elle est absente avant. |
| O10 | Une barre de progression indique l'avancement. | 3 questions sur 10 → 30 %. |
| O11 | Chaque question est chronométrée ; la durée dépend de la difficulté. | 20 s en facile, 10 s en difficile. |
| O12 | L'expiration du temps compte comme une réponse fausse et révèle la bonne. | Ne rien toucher pendant 20 s. |
| O13 | Le passage à la question suivante est animé. | Transition en fondu et glissement. |
| O14 | À la fin, un écran de résultat affiche le score, le pourcentage et un message. | 10 questions répondues → écran de résultat. |
| O15 | Un récapitulatif liste chaque question avec la réponse donnée et la bonne. | Trois cas distincts : juste, faux, temps écoulé. |
| O16 | Le meilleur score est conservé par catégorie entre deux lancements. | Tuer l'application, la relancer. |
| O17 | L'état est centralisé dans un `ChangeNotifier` fourni par `provider`. | Plus aucun `setState` de données. |
| O18 | La logique de scoring est couverte par des tests. | `flutter test` passe. |

### Fonctionnalités bonus

| # | Exigence |
| --- | --- |
| B1 | Mode sombre commutable et persisté. |
| B2 | Chaque bouton de réponse est annoncé correctement par le lecteur d'écran. |
| B3 | Une confirmation avant d'abandonner une partie en cours. |
| B4 | Un bonus de points pour la rapidité. |

---

## 59.0.3 — Notions mobilisées

Ce projet ne contient aucune notion nouvelle. Si une ligne du tableau vous surprend, relisez le chapitre indiqué avant de commencer.

| Notion | Chapitre | Usage exact dans ce projet |
| --- | --- | --- |
| `List`, `Map`, `Set` | 06 | La banque de questions, les réponses. |
| Classes, propriétés, getters | 08 | `Question`, `Quiz`, `ReponseDonnee`. |
| Constructeurs nommés, `required` | 09 | `Question.fromJson`. |
| `enum` enrichi | 11 | `Difficulte` porte libellé, points et durée. |
| Null safety, `?`, `??` | 12 | `int? indexChoisi` : `null` signifie « temps écoulé ». |
| Exceptions, `FormatException` | 13 | Le refus d'une question mal formée. |
| `map`, `where`, `fold`, `take`, `shuffle` | 14 | Filtrage par catégorie, calcul du score. |
| `Future`, `async`, `await` | 15 | Le chargement des assets et des préférences. |
| `pubspec.yaml`, paquets | 16 | `provider`, `shared_preferences`. |
| `jsonDecode`, `fromJson` | 17 | La lecture de la banque de questions. |
| `MaterialApp`, `Scaffold`, `AppBar` | 44 | La structure des trois écrans. |
| `StatefulWidget`, `initState`, `dispose`, `Timer` | 45 | Le compte à rebours. |
| `Row`, `Column`, `Expanded`, `Stack` | 46 | La mise en page de l'écran de question. |
| `Text`, `TextStyle`, `Icon` | 47 | Les énoncés, les icônes de correction. |
| Assets et `pubspec.yaml` | 47 | Le fichier `questions.json`. |
| `ListView.builder`, `Card` | 48 | Le récapitulatif final. |
| Boutons, `InkWell` | 49 | Les boutons de réponse. |
| `Navigator.push`, `pushReplacement` | 50 | L'enchaînement accueil → question → résultat. |
| `ThemeData`, mode sombre, `Semantics` | 51 | Le thème et l'accessibilité. |
| `ChangeNotifier`, `provider` | 52 | L'état centralisé, minuteur compris. |
| `shared_preferences` | 54 | Le meilleur score par catégorie. |

---

## 59.0.4 — Arborescence du projet

Voici l'arborescence finale. Elle est donnée dès maintenant pour que vous sachiez où va chaque fichier ; nous la construirons pas à pas.

```text
mon_quiz/
├── pubspec.yaml
├── assets/
│   └── data/
│       └── questions.json               la banque de questions
├── lib/
│   ├── main.dart                        point d'entrée, thème, providers
│   ├── modeles/
│   │   ├── difficulte.dart              enum Difficulte
│   │   ├── categorie.dart               classe Categorie
│   │   └── question.dart                classe Question + fromJson
│   ├── logique/
│   │   ├── banque_questions.dart        catégories + questions + filtrage
│   │   ├── quiz.dart                    moteur du quiz, pur et testable
│   │   └── melange.dart                 tirage et mélange
│   ├── donnees/
│   │   ├── depot_questions.dart         interface + lecture des assets
│   │   └── depot_scores.dart            interface + shared_preferences
│   ├── etat/
│   │   ├── etat_quiz.dart               ChangeNotifier principal
│   │   └── etat_theme.dart              ChangeNotifier du thème
│   ├── ecrans/
│   │   ├── ecran_accueil.dart           choix de la catégorie
│   │   ├── ecran_question.dart          le cœur du jeu
│   │   └── ecran_resultat.dart          score et récapitulatif
│   ├── widgets/
│   │   ├── bouton_reponse.dart          une réponse, cinq apparences
│   │   ├── barre_progression.dart       avancement animé
│   │   ├── compte_a_rebours.dart        le minuteur circulaire
│   │   ├── carte_explication.dart       l'explication après la réponse
│   │   └── ligne_recapitulatif.dart     une ligne du bilan final
│   └── utilitaires/
│       ├── couleurs_reponse.dart        vert, rouge, neutre
│       └── themes.dart                  thème clair et thème sombre
└── test/
    ├── quiz_test.dart                   tests du scoring
    └── melange_test.dart                tests du tirage
```

**Pourquoi cette découpe ?** C'est exactement celle du chapitre 58, avec les mêmes responsabilités :

```text
modeles/      QUOI ?        les données brutes, sans Flutter
logique/      COMMENT ?     les règles du jeu, pures, testables sans écran
donnees/      OÙ ?          la lecture des assets et des préférences
etat/         QUAND ?       ce qui change et qui prévient l'interface
ecrans/       À QUOI ÇA     les trois pages
widgets/      RESSEMBLE ?   les morceaux réutilisables
utilitaires/  AVEC QUOI ?   les couleurs et les thèmes
```

Un fichier de `modeles/` ou de `logique/` n'importera **jamais** `package:flutter/material.dart`. C'est cette règle qui rendra les tests du 59.22 possibles sans lancer d'application.

---

## 59.1 — Créer le projet et poser le squelette

### Créer le projet

```text
flutter create mon_quiz
cd mon_quiz
```

### Ajouter les dépendances

Deux paquets suffisent. On les ajoute avec `flutter pub add`, qui écrit toujours la version la plus récente compatible.

```text
flutter pub add provider
flutter pub add shared_preferences
```

Le `pubspec.yaml` obtenu ressemble à ceci. Les numéros de version sont ceux constatés à la rédaction ; les vôtres peuvent être supérieurs.

**`mon_quiz/pubspec.yaml`**

```yaml
name: mon_quiz
description: "Un quiz à choix multiples chronométré."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.12.0

dependencies:
  flutter:
    sdk: flutter

  # Gestion d'état (chapitre 52)
  provider: ^6.1.5

  # Meilleur score persistant (chapitre 54)
  shared_preferences: ^2.5.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

flutter:
  uses-material-design: true

  # La banque de questions (chapitre 47).
  # Le slash final est obligatoire : il déclare le DOSSIER.
  assets:
    - assets/data/
```

> **Attention à l'indentation.** `assets:` est une clé de `flutter:`, donc décalée de deux espaces. C'est l'erreur du chapitre 47 la plus fréquente : placée en colonne 0, la déclaration est ignorée et toute lecture échoue avec `Unable to load asset`.

Créez le dossier tout de suite, sinon `flutter pub get` se plaindra d'un dossier d'assets inexistant :

```text
mkdir -p assets/data
```

### Le squelette

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ApplicationQuiz());
}

/// Racine de l'application.
///
/// Pour l'instant elle installe un thème Material 3 et affiche un
/// écran vide. Tout le reste viendra s'y greffer, étape par étape.
class ApplicationQuiz extends StatelessWidget {
  const ApplicationQuiz({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Quiz',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const EcranAccueil(),
    );
  }
}

/// Écran d'accueil, provisoirement vide.
class EcranAccueil extends StatelessWidget {
  const EcranAccueil({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Quiz')),
      body: const Center(child: Text('Rien pour l\'instant')),
    );
  }
}
```

**État exécutable.** `flutter run` affiche une barre d'application violette et un texte centré. Le projet compile et les dépendances sont résolues.

```text
┌────────────────────────────────────────────────┐
│  Quiz                                          │
├────────────────────────────────────────────────┤
│                                                │
│              Rien pour l'instant               │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 59.2 — Le modèle : l'énumération `Difficulte`

Une question a une difficulté. Cette difficulté sert à deux choses : elle décide du nombre de points gagnés, et elle décide du temps accordé. Un débutant écrirait deux `if` dispersés dans l'interface. C'est là que les bugs naissent.

L'`enum` enrichi du chapitre 11 fait mieux : chaque valeur transporte ses propres données, et il devient impossible d'oublier un cas.

**`lib/modeles/difficulte.dart`**

```dart
/// Niveau de difficulté d'une question.
///
/// C'est un `enum` enrichi (chapitre 11) : chaque valeur porte son
/// libellé affichable, le nombre de points qu'elle rapporte et le
/// temps accordé pour y répondre.
///
/// Regrouper ces trois informations ici évite de les disperser dans
/// l'interface. Le jour où vous décidez qu'une question difficile
/// rapporte 5 points, une seule ligne change.
enum Difficulte {
  facile('Facile', 1, 20),
  moyenne('Moyenne', 2, 15),
  difficile('Difficile', 3, 10);

  const Difficulte(this.libelle, this.points, this.secondes);

  /// Texte affiché à l'utilisateur.
  final String libelle;

  /// Points gagnés quand la réponse est juste.
  final int points;

  /// Durée du compte à rebours, en secondes.
  final int secondes;

  /// Durée du compte à rebours sous forme de `Duration`.
  Duration get duree => Duration(seconds: secondes);

  /// Reconstruit une difficulté à partir du nom écrit dans le JSON.
  ///
  /// Si le nom est absent ou inconnu, on retombe sur `moyenne` plutôt
  /// que de lever une exception : une question dont la difficulté est
  /// mal orthographiée reste une question jouable.
  static Difficulte depuisNom(String? nom) {
    for (final Difficulte d in Difficulte.values) {
      if (d.name == nom) {
        return d;
      }
    }
    return Difficulte.moyenne;
  }
}
```

### Vérification en console

Ce fichier n'importe pas Flutter. Vous pouvez le coller dans DartPad avec le `main` suivant.

```dart
// Collez le contenu de difficulte.dart au-dessus de ce main.
void main() {
  for (final Difficulte d in Difficulte.values) {
    print('${d.name}: ${d.libelle}, ${d.points} pt, ${d.secondes} s');
  }
  print(Difficulte.depuisNom('difficile'));
  print(Difficulte.depuisNom('impossible'));
}
```

**Résultat :**

```text
facile: Facile, 1 pt, 20 s
moyenne: Moyenne, 2 pt, 15 s
difficile: Difficile, 3 pt, 10 s
Difficulte.difficile
Difficulte.moyenne
```

> **Remarque.** `d.name` est offert gratuitement par Dart sur tout `enum` : il renvoie le nom de la valeur sous forme de `String`. C'est ce que l'on écrit dans un fichier JSON. N'écrivez jamais `d.index` dans un fichier persisté : le jour où vous insérez une valeur au milieu de l'énumération, tous les fichiers existants désignent la mauvaise difficulté.

**État exécutable.** Le fichier compile seul. L'application n'a pas encore changé.

---

## 59.3 — Le modèle : la classe `Categorie`

Une catégorie regroupe des questions. Elle a un identifiant technique (`dart`), un nom affichable (`Dart`) et une description.

Elle porte aussi un nom d'icône, sous forme de `String`. Pourquoi pas un `IconData` directement ? Parce que `IconData` vient de Flutter, et qu'un modèle ne doit rien connaître de Flutter. La traduction du nom en icône se fera dans l'interface, au 59.7.

**`lib/modeles/categorie.dart`**

```dart
/// Une catégorie de questions.
///
/// [id] est l'identifiant technique, celui écrit dans le JSON des
/// questions et utilisé comme clé de stockage du meilleur score.
/// Il ne doit jamais changer.
///
/// [nom] est le libellé affiché. Il peut changer librement.
class Categorie {
  const Categorie({
    required this.id,
    required this.nom,
    required this.description,
    required this.icone,
  });

  final String id;
  final String nom;
  final String description;

  /// Nom logique de l'icône, par exemple `code` ou `manette`.
  ///
  /// Ce n'est volontairement PAS un `IconData` : ce fichier ne doit
  /// pas importer Flutter. La conversion se fait dans l'interface.
  final String icone;

  /// Lecture depuis le JSON (chapitre 17).
  ///
  /// Chaque champ est lu avec un type nullable puis complété par `??`.
  /// Une catégorie sans description reste une catégorie valable.
  factory Categorie.fromJson(Map<String, dynamic> json) {
    final String? id = json['id'] as String?;
    if (id == null || id.isEmpty) {
      throw const FormatException('Catégorie sans identifiant.');
    }
    return Categorie(
      id: id,
      nom: json['nom'] as String? ?? id,
      description: json['description'] as String? ?? '',
      icone: json['icone'] as String? ?? 'aide',
    );
  }

  @override
  String toString() => 'Categorie($id)';
}
```

> **Pourquoi une exception ici et pas au 59.2 ?** Parce que les deux cas n'ont pas la même gravité. Une difficulté mal orthographiée est cosmétique : on peut jouer quand même. Une catégorie sans identifiant, en revanche, rend impossible le rattachement des questions et le stockage du score. Une donnée qui casse le fonctionnement doit lever ; une donnée qui gêne l'affichage doit retomber sur une valeur par défaut. Ce jugement est à faire champ par champ.

**État exécutable.** Le fichier compile seul.

---

## 59.4 — Le modèle : la classe `Question`

C'est le cœur du projet. Une question porte :

```text
id                  identifiant unique, stable
categorie           l'identifiant d'une Categorie
enonce              le texte de la question
reponses            2 à 6 propositions
indexBonneReponse   la position de la bonne dans `reponses`
explication         le texte affiché après la réponse
difficulte          l'enum du 59.2
```

Le point délicat est `indexBonneReponse`. Stocker un indice plutôt que le texte de la bonne réponse a un avantage décisif : quand on mélangera les réponses au 59.16, il suffira de recalculer cet indice. Mais cet indice doit impérativement rester dans les bornes de la liste, sinon l'application plantera avec `RangeError` au moment de l'affichage. On le vérifie donc à la construction, une fois pour toutes.

**`lib/modeles/question.dart`**

```dart
import 'difficulte.dart';

/// Une question à choix multiples.
///
/// L'objet est immuable : tous les champs sont `final`. Pour obtenir
/// une variante (réponses mélangées, par exemple), on fabrique une
/// nouvelle instance ; on ne modifie jamais celle-ci.
class Question {
  Question({
    required this.id,
    required this.categorie,
    required this.enonce,
    required this.reponses,
    required this.indexBonneReponse,
    required this.explication,
    required this.difficulte,
  }) {
    // Ces deux contrôles sont volontairement dans le constructeur, et
    // non dans `fromJson`. Ainsi ils protègent AUSSI les questions
    // construites à la main dans les tests.
    if (reponses.length < 2) {
      throw FormatException(
        'La question "$id" doit proposer au moins deux réponses.',
      );
    }
    if (indexBonneReponse < 0 || indexBonneReponse >= reponses.length) {
      throw FormatException(
        'La question "$id" désigne la réponse $indexBonneReponse alors '
        'qu\'il n\'y en a que ${reponses.length}.',
      );
    }
  }

  /// Identifiant unique et stable, par exemple `dart-03`.
  final String id;

  /// Identifiant de la catégorie (voir `Categorie.id`).
  final String categorie;

  /// Le texte de la question.
  final String enonce;

  /// Les propositions, dans l'ordre où elles seront affichées.
  final List<String> reponses;

  /// Position de la bonne réponse dans [reponses].
  final int indexBonneReponse;

  /// Texte pédagogique affiché une fois la réponse donnée.
  final String explication;

  final Difficulte difficulte;

  /// Le texte de la bonne réponse.
  String get bonneReponse => reponses[indexBonneReponse];

  /// `true` si [index] désigne la bonne réponse.
  ///
  /// [index] vaut `null` quand le temps est écoulé : dans ce cas la
  /// réponse est fausse, sans que l'utilisateur ait rien choisi.
  bool estJuste(int? index) => index != null && index == indexBonneReponse;

  /// Lettre affichée devant une proposition : A, B, C, D...
  static String lettre(int index) => String.fromCharCode(65 + index);

  /// Construction depuis le JSON (chapitre 17).
  ///
  /// Chaque champ est lu de façon défensive :
  /// - on lit dans un type nullable (`as String?`) ;
  /// - on vérifie, ou on complète avec `??` ;
  /// - on lève une `FormatException` explicite quand la donnée est
  ///   inutilisable.
  ///
  /// Un message d'erreur qui nomme l'identifiant de la question fait
  /// gagner un quart d'heure le jour où le fichier a 200 entrées.
  factory Question.fromJson(Map<String, dynamic> json) {
    final String id = json['id'] as String? ?? '(sans id)';

    final String? enonce = json['enonce'] as String?;
    if (enonce == null || enonce.trim().isEmpty) {
      throw FormatException('Question "$id" : énoncé manquant.');
    }

    final Object? brutReponses = json['reponses'];
    if (brutReponses is! List) {
      throw FormatException('Question "$id" : "reponses" doit être une liste.');
    }
    // `List<dynamic>` -> `List<String>`. `map` vient du chapitre 14.
    final List<String> reponses =
        brutReponses.map((Object? r) => '$r').toList();

    final Object? brutIndex = json['bonne'];
    if (brutIndex is! int) {
      throw FormatException('Question "$id" : "bonne" doit être un entier.');
    }

    return Question(
      id: id,
      categorie: json['categorie'] as String? ?? 'divers',
      enonce: enonce,
      reponses: reponses,
      indexBonneReponse: brutIndex,
      explication: json['explication'] as String? ?? '',
      difficulte: Difficulte.depuisNom(json['difficulte'] as String?),
    );
  }

  /// Écriture vers le JSON. Utile pour les tests d'aller-retour et
  /// pour l'export du défi 7.
  Map<String, dynamic> toJson() {
    return <String, dynamic>{
      'id': id,
      'categorie': categorie,
      'enonce': enonce,
      'reponses': reponses,
      'bonne': indexBonneReponse,
      'explication': explication,
      'difficulte': difficulte.name,
    };
  }

  @override
  String toString() => 'Question($id, ${reponses.length} réponses)';
}
```

### Vérification en console

```dart
// Collez difficulte.dart et question.dart au-dessus de ce main.
void main() {
  final Question q = Question(
    id: 'test-01',
    categorie: 'dart',
    enonce: 'Que vaut 2 + 3 * 4 ?',
    reponses: <String>['20', '14', '24', '9'],
    indexBonneReponse: 1,
    explication: 'La multiplication est prioritaire.',
    difficulte: Difficulte.facile,
  );

  print(q);
  print('Bonne réponse : ${q.bonneReponse}');
  print('Choix 1 juste ? ${q.estJuste(1)}');
  print('Choix 0 juste ? ${q.estJuste(0)}');
  print('Aucun choix juste ? ${q.estJuste(null)}');
  print('Lettres : ${Question.lettre(0)}${Question.lettre(3)}');

  try {
    Question(
      id: 'test-02',
      categorie: 'dart',
      enonce: 'Cassée',
      reponses: <String>['a', 'b'],
      indexBonneReponse: 7,
      explication: '',
      difficulte: Difficulte.facile,
    );
  } on FormatException catch (e) {
    print('Refusée : ${e.message}');
  }
}
```

**Résultat :**

```text
Question(test-01, 4 réponses)
Bonne réponse : 14
Choix 1 juste ? true
Choix 0 juste ? false
Aucun choix juste ? false
Lettres : AD
Refusée : La question "test-02" désigne la réponse 7 alors qu'il n'y en a que 2.
```

> **Remarque sur `String.fromCharCode(65 + index)`.** 65 est le code du caractère `A` en ASCII. `65 + 3` donne 68, c'est-à-dire `D`. C'est une manière courte d'écrire la numérotation A, B, C, D sans écrire de liste. Au-delà de 26 réponses la formule sortirait de l'alphabet, mais aucun quiz raisonnable ne propose 27 réponses.

**État exécutable.** Le fichier compile seul.

---

## 59.5 — La banque de questions en JSON

Nous aurions pu écrire les questions directement en Dart, dans une `List<Question>` constante. Ce serait plus simple. Mais ce serait une mauvaise idée, pour trois raisons :

```text
En Dart, dans le code               En JSON, dans les assets
─────────────────────────           ─────────────────────────
Modifier une question =             Modifier une question =
  recompiler l'application            éditer un fichier texte

Faire relire par un non-            Le relecteur ouvre le fichier
  développeur : impossible            dans un éditeur de texte

Télécharger de nouvelles            Le même format sert au fichier
  questions plus tard : à réécrire    local et à la réponse d'une API
```

Ce troisième point est le plus important. Au chapitre 61, vous lirez du JSON venu d'une API. Le code de lecture sera identique à celui d'aujourd'hui ; seule la source changera. En choisissant le JSON dès maintenant, vous préparez ce changement sans le savoir.

### La structure du fichier

```text
{
  "version": 1,
  "categories": [ { id, nom, description, icone }, ... ],
  "questions":  [ { id, categorie, enonce, reponses, bonne,
                    explication, difficulte }, ... ]
}
```

Le champ `version` ne sert à rien aujourd'hui. Il servira le jour où vous changerez le format : le code pourra alors dire « ce fichier est en version 1, je sais le lire » ou « version 3, je ne sais pas ». Cela coûte une ligne et évite un plantage inexplicable trois ans plus tard.

### Le fichier

**`assets/data/questions.json`**

```json
{
  "version": 1,
  "categories": [
    {
      "id": "dart",
      "nom": "Dart",
      "description": "Le langage : syntaxe, types, asynchrone.",
      "icone": "code"
    },
    {
      "id": "flutter",
      "nom": "Flutter",
      "description": "Les widgets, l'état, la navigation.",
      "icone": "widgets"
    },
    {
      "id": "jeu",
      "nom": "Jeu vidéo",
      "description": "Culture générale et vocabulaire technique.",
      "icone": "manette"
    }
  ],
  "questions": [
    {
      "id": "dart-01",
      "categorie": "dart",
      "enonce": "Quel mot-clé déclare une constante connue dès la compilation ?",
      "reponses": ["var", "final", "const", "static"],
      "bonne": 2,
      "explication": "`const` est évalué à la compilation. `final` est aussi une constante, mais sa valeur peut n'être connue qu'à l'exécution.",
      "difficulte": "facile"
    },
    {
      "id": "dart-02",
      "categorie": "dart",
      "enonce": "Que vaut l'expression 2 + 3 * 4 en Dart ?",
      "reponses": ["20", "14", "24", "9"],
      "bonne": 1,
      "explication": "La multiplication est prioritaire sur l'addition : 3 * 4 vaut 12, puis 2 + 12 vaut 14.",
      "difficulte": "facile"
    },
    {
      "id": "dart-03",
      "categorie": "dart",
      "enonce": "Quel type de valeur renvoie une fonction déclarée `async` ?",
      "reponses": ["Un Stream", "Un Future", "void", "Un Iterable"],
      "bonne": 1,
      "explication": "Toute fonction `async` renvoie un Future. Le `return` alimente ce Future ; il ne renvoie pas la valeur directement.",
      "difficulte": "moyenne"
    },
    {
      "id": "dart-04",
      "categorie": "dart",
      "enonce": "Quel opérateur fournit une valeur de repli quand l'expression de gauche est nulle ?",
      "reponses": ["?.", "??", "!", "?["],
      "bonne": 1,
      "explication": "`a ?? b` vaut a si a n'est pas nul, sinon b. `a?.b` appelle b seulement si a n'est pas nul. `a!` affirme au compilateur que a n'est pas nul.",
      "difficulte": "moyenne"
    },
    {
      "id": "dart-05",
      "categorie": "dart",
      "enonce": "Quelle méthode applique une fonction à chaque élément et renvoie une nouvelle collection ?",
      "reponses": ["forEach", "where", "map", "fold"],
      "bonne": 2,
      "explication": "`map` transforme et renvoie un Iterable. `forEach` ne renvoie rien, `where` filtre, `fold` réduit à une seule valeur.",
      "difficulte": "facile"
    },
    {
      "id": "dart-06",
      "categorie": "dart",
      "enonce": "Combien de fois s'exécute le corps d'une boucle `do { ... } while (false);` ?",
      "reponses": ["Zéro fois", "Une fois", "Deux fois", "Indéfiniment"],
      "bonne": 1,
      "explication": "Dans un `do ... while`, la condition est évaluée APRÈS le corps. Celui-ci s'exécute donc toujours au moins une fois.",
      "difficulte": "difficile"
    },
    {
      "id": "flutter-01",
      "categorie": "flutter",
      "enonce": "Quelle méthode demande à un StatefulWidget de se reconstruire ?",
      "reponses": ["build()", "setState()", "initState()", "dispose()"],
      "bonne": 1,
      "explication": "`setState()` marque l'élément comme sale ; le framework rappellera `build()` au prochain rendu. On n'appelle jamais `build()` soi-même.",
      "difficulte": "facile"
    },
    {
      "id": "flutter-02",
      "categorie": "flutter",
      "enonce": "Quel widget place ses enfants les uns au-dessous des autres ?",
      "reponses": ["Row", "Column", "Stack", "Wrap"],
      "bonne": 1,
      "explication": "Column empile verticalement, Row horizontalement, Stack superpose en profondeur, Wrap passe à la ligne quand la place manque.",
      "difficulte": "facile"
    },
    {
      "id": "flutter-03",
      "categorie": "flutter",
      "enonce": "Quelle méthode du State est appelée une seule fois, avant le premier build ?",
      "reponses": ["createState", "initState", "didUpdateWidget", "dispose"],
      "bonne": 1,
      "explication": "`initState` est l'endroit où l'on démarre un minuteur ou où l'on crée un contrôleur. `dispose` est son symétrique de fin de vie.",
      "difficulte": "moyenne"
    },
    {
      "id": "flutter-04",
      "categorie": "flutter",
      "enonce": "Quel widget affiche une longue liste en ne construisant que les éléments visibles ?",
      "reponses": ["Column", "ListView(children: ...)", "ListView.builder", "Stack"],
      "bonne": 2,
      "explication": "`ListView.builder` appelle son itemBuilder à la demande. Les deux premiers construisent tous les enfants d'un coup, ce qui devient inacceptable au-delà de quelques dizaines d'éléments.",
      "difficulte": "moyenne"
    },
    {
      "id": "flutter-05",
      "categorie": "flutter",
      "enonce": "Comment un écran récupère-t-il la valeur passée à Navigator.pop ?",
      "reponses": [
        "En lisant une variable globale",
        "En attendant le Future renvoyé par Navigator.push",
        "En appelant setState",
        "C'est impossible"
      ],
      "bonne": 1,
      "explication": "`Navigator.push` renvoie un Future qui se complète lors du pop, avec la valeur passée en second argument.",
      "difficulte": "moyenne"
    },
    {
      "id": "flutter-06",
      "categorie": "flutter",
      "enonce": "Quel widget donne une hauteur bornée à une ListView placée dans une Column ?",
      "reponses": ["Center", "Expanded", "Padding", "Align"],
      "bonne": 1,
      "explication": "Une Column offre une hauteur infinie à ses enfants ; une ListView en réclame une finie. `Expanded` lui attribue la place restante et résout l'erreur « unbounded height ».",
      "difficulte": "difficile"
    },
    {
      "id": "jeu-01",
      "categorie": "jeu",
      "enonce": "En quelle année la borne d'arcade Pong est-elle sortie ?",
      "reponses": ["1962", "1972", "1978", "1985"],
      "bonne": 1,
      "explication": "Atari commercialise Pong en 1972. C'est le premier jeu d'arcade à rencontrer un vrai succès commercial.",
      "difficulte": "moyenne"
    },
    {
      "id": "jeu-02",
      "categorie": "jeu",
      "enonce": "Quel personnage est la mascotte de Nintendo ?",
      "reponses": ["Sonic", "Mario", "Pac-Man", "Crash Bandicoot"],
      "bonne": 1,
      "explication": "Mario apparaît en 1981 dans Donkey Kong. Sonic appartient à SEGA, Pac-Man à Namco, Crash Bandicoot à Naughty Dog.",
      "difficulte": "facile"
    },
    {
      "id": "jeu-03",
      "categorie": "jeu",
      "enonce": "Combien d'images par seconde vise-t-on couramment pour un jeu fluide sur mobile ?",
      "reponses": ["12", "24", "60", "600"],
      "bonne": 2,
      "explication": "60 images par seconde, soit environ 16,7 millisecondes par image. Le cinéma se contente de 24, mais un jeu doit réagir à l'utilisateur.",
      "difficulte": "facile"
    },
    {
      "id": "jeu-04",
      "categorie": "jeu",
      "enonce": "Que désigne le « delta time » dans une boucle de jeu ?",
      "reponses": [
        "Le temps total de la partie",
        "Le temps écoulé depuis l'image précédente",
        "Le nombre d'images par seconde",
        "Le délai du réseau"
      ],
      "bonne": 1,
      "explication": "Multiplier les déplacements par le delta time rend la vitesse indépendante du nombre d'images par seconde de la machine.",
      "difficulte": "moyenne"
    },
    {
      "id": "jeu-05",
      "categorie": "jeu",
      "enonce": "Qu'appelle-t-on une « hitbox » ?",
      "reponses": [
        "La zone visible du sprite",
        "La zone servant à détecter les collisions",
        "La barre de vie affichée à l'écran",
        "Le cadre de la caméra"
      ],
      "bonne": 1,
      "explication": "La hitbox est souvent plus petite que le sprite : cela rend le jeu plus indulgent et donc plus agréable.",
      "difficulte": "moyenne"
    },
    {
      "id": "jeu-06",
      "categorie": "jeu",
      "enonce": "Dans le repère d'un écran 2D classique, dans quel sens croît l'axe des Y ?",
      "reponses": ["Vers le haut", "Vers le bas", "Vers la droite", "Vers la gauche"],
      "bonne": 1,
      "explication": "L'origine est en haut à gauche et Y croît vers le bas. C'est l'inverse du repère mathématique, et la source d'erreur numéro un des débutants.",
      "difficulte": "difficile"
    }
  ]
}
```

> **Attention aux virgules.** Le JSON n'autorise pas la virgule après le dernier élément d'un tableau ou d'un objet. Une virgule en trop provoquera au lancement un message du genre `FormatException: Unexpected character (at line 42, character 5)`. Votre éditeur signale généralement ces erreurs en rouge ; ne les ignorez pas.

**État exécutable.** Le fichier est en place. Rien ne le lit encore : c'est l'objet de la section suivante.

---

## 59.6 — La banque : regrouper catégories et questions

Le fichier JSON contient deux listes liées. Plutôt que de les promener séparément dans toute l'application, on les regroupe dans un objet unique, muni des quelques opérations dont l'interface aura besoin.

**`lib/logique/banque_questions.dart`**

```dart
import '../modeles/categorie.dart';
import '../modeles/question.dart';

/// L'ensemble des catégories et des questions chargées.
///
/// Cette classe est purement logique : aucun import Flutter. Elle est
/// donc testable dans un simple test Dart (voir 59.22).
class BanqueQuestions {
  BanqueQuestions({
    required this.version,
    required this.categories,
    required this.questions,
  });

  /// Version du format du fichier, pour les évolutions futures.
  final int version;

  final List<Categorie> categories;
  final List<Question> questions;

  /// Nombre total de questions, toutes catégories confondues.
  int get total => questions.length;

  /// Les questions d'une catégorie.
  ///
  /// [idCategorie] vaut `null` pour « toutes les catégories ».
  /// On renvoie toujours une nouvelle liste : l'appelant peut la
  /// mélanger sans abîmer la banque.
  List<Question> pourCategorie(String? idCategorie) {
    if (idCategorie == null) {
      return List<Question>.of(questions);
    }
    return questions
        .where((Question q) => q.categorie == idCategorie)
        .toList();
  }

  /// Nombre de questions dans une catégorie.
  int compte(String? idCategorie) => pourCategorie(idCategorie).length;

  /// La catégorie correspondant à un identifiant, ou `null`.
  Categorie? categorieParId(String? id) {
    if (id == null) {
      return null;
    }
    for (final Categorie c in categories) {
      if (c.id == id) {
        return c;
      }
    }
    return null;
  }

  /// Nom affichable d'une sélection, y compris « Toutes les catégories ».
  String nomDe(String? idCategorie) {
    return categorieParId(idCategorie)?.nom ?? 'Toutes les catégories';
  }

  /// Construction depuis la structure JSON décrite au 59.5.
  ///
  /// Une question invalide fait échouer tout le chargement. C'est un
  /// choix délibéré : mieux vaut un message d'erreur clair au
  /// démarrage qu'une question fantôme découverte par un joueur.
  factory BanqueQuestions.fromJson(Map<String, dynamic> json) {
    final Object? brutCategories = json['categories'];
    final Object? brutQuestions = json['questions'];

    if (brutCategories is! List || brutQuestions is! List) {
      throw const FormatException(
        'Le fichier doit contenir les listes "categories" et "questions".',
      );
    }

    final List<Categorie> categories = brutCategories
        .map((Object? c) => Categorie.fromJson(c as Map<String, dynamic>))
        .toList();

    final List<Question> questions = brutQuestions
        .map((Object? q) => Question.fromJson(q as Map<String, dynamic>))
        .toList();

    if (questions.isEmpty) {
      throw const FormatException('Le fichier ne contient aucune question.');
    }

    return BanqueQuestions(
      version: json['version'] as int? ?? 1,
      categories: categories,
      questions: questions,
    );
  }

  /// Banque vide, utile comme valeur initiale avant chargement.
  static final BanqueQuestions vide = BanqueQuestions(
    version: 1,
    categories: const <Categorie>[],
    questions: const <Question>[],
  );
}
```

> **Remarque sur `is! List`.** On aurait pu écrire `json['categories'] as List`. En cas de type inattendu, le message serait `type 'String' is not a subtype of type 'List'` — techniquement exact, inutilisable pour l'auteur du fichier. Le test explicite permet d'écrire un message qui dit quoi corriger. C'est la différence entre un plantage et un diagnostic.

**État exécutable.** Le fichier compile seul.

---

## 59.7 — Le chargement asynchrone des questions

Lire un asset prend du temps. Ce n'est pas immédiat, ce n'est pas garanti, cela peut échouer. C'est donc une opération asynchrone, exactement au sens du chapitre 15 : la fonction renvoie un `Future`, et l'appelant doit `await`.

L'API à connaître est `rootBundle`, défini dans `package:flutter/services.dart` :

```dart
Future<String> loadString(String key, {bool cache = true})
```

`key` est le chemin déclaré dans `pubspec.yaml`, tel quel : `assets/data/questions.json`. Pas de `./`, pas de chemin absolu.

### L'interface et ses deux implémentations

Comme au chapitre 58, on définit d'abord **ce que l'on veut** (une interface), puis **comment on l'obtient** (les implémentations). L'interface permet d'écrire des tests sans lire le moindre fichier.

**`lib/donnees/depot_questions.dart`**

```dart
import 'dart:convert';

import 'package:flutter/services.dart' show rootBundle;

import '../logique/banque_questions.dart';
import '../modeles/categorie.dart';
import '../modeles/question.dart';
import '../modeles/difficulte.dart';

/// Ce que l'application attend d'une source de questions.
///
/// Une seule opération : fournir la banque. Peu importe qu'elle vienne
/// des assets, du réseau ou d'une liste écrite à la main.
abstract class DepotQuestions {
  Future<BanqueQuestions> charger();
}

/// Implémentation réelle : lecture du fichier JSON des assets.
class DepotQuestionsAssets implements DepotQuestions {
  const DepotQuestionsAssets({
    this.chemin = 'assets/data/questions.json',
  });

  /// Chemin de l'asset, tel que déclaré dans `pubspec.yaml`.
  final String chemin;

  @override
  Future<BanqueQuestions> charger() async {
    // 1. Lire le fichier. Opération asynchrone (chapitre 15).
    final String texte = await rootBundle.loadString(chemin);

    // 2. Décoder le JSON (chapitre 17). jsonDecode renvoie `dynamic` :
    //    on vérifie le type avant de l'utiliser.
    final Object? brut = jsonDecode(texte);
    if (brut is! Map<String, dynamic>) {
      throw const FormatException(
        'Le fichier de questions doit contenir un objet JSON.',
      );
    }

    // 3. Construire les objets Dart.
    return BanqueQuestions.fromJson(brut);
  }
}

/// Implémentation de test : une banque écrite en dur.
///
/// Elle ne touche à aucun fichier, ce qui la rend instantanée et
/// parfaitement reproductible. C'est elle que l'on branchera dans les
/// tests du 59.22.
class DepotQuestionsMemoire implements DepotQuestions {
  DepotQuestionsMemoire({BanqueQuestions? banque, this.retard = Duration.zero})
      : banque = banque ?? _banqueParDefaut();

  final BanqueQuestions banque;

  /// Retard simulé, pour voir l'écran de chargement en développement.
  final Duration retard;

  @override
  Future<BanqueQuestions> charger() async {
    if (retard > Duration.zero) {
      await Future<void>.delayed(retard);
    }
    return banque;
  }

  static BanqueQuestions _banqueParDefaut() {
    return BanqueQuestions(
      version: 1,
      categories: const <Categorie>[
        Categorie(
          id: 'test',
          nom: 'Test',
          description: 'Questions de démonstration.',
          icone: 'aide',
        ),
      ],
      questions: <Question>[
        Question(
          id: 'test-01',
          categorie: 'test',
          enonce: 'Combien font 1 + 1 ?',
          reponses: const <String>['1', '2', '3'],
          indexBonneReponse: 1,
          explication: 'Deux.',
          difficulte: Difficulte.facile,
        ),
        Question(
          id: 'test-02',
          categorie: 'test',
          enonce: 'Quelle est la couleur du cheval blanc d\'Henri IV ?',
          reponses: const <String>['Noir', 'Blanc'],
          indexBonneReponse: 1,
          explication: 'C\'est dans la question.',
          difficulte: Difficulte.facile,
        ),
      ],
    );
  }
}
```

### Vérifier le chargement dans l'application

Remplacez temporairement le corps de `EcranAccueil` pour afficher le résultat du chargement. On utilise `FutureBuilder` (chapitre 53), qui est fait exactement pour cela : afficher trois états — en cours, en erreur, réussi.

**`lib/main.dart`** (version de vérification)

```dart
import 'package:flutter/material.dart';

import 'donnees/depot_questions.dart';
import 'logique/banque_questions.dart';

void main() {
  runApp(const ApplicationQuiz());
}

class ApplicationQuiz extends StatelessWidget {
  const ApplicationQuiz({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Quiz',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const EcranAccueil(),
    );
  }
}

class EcranAccueil extends StatefulWidget {
  const EcranAccueil({super.key});

  @override
  State<EcranAccueil> createState() => _EcranAccueilState();
}

class _EcranAccueilState extends State<EcranAccueil> {
  /// Le Future est créé UNE fois, dans initState.
  ///
  /// Erreur classique : appeler `depot.charger()` directement dans
  /// `build`. Chaque reconstruction relancerait alors une lecture, et
  /// l'écran clignoterait indéfiniment.
  late final Future<BanqueQuestions> _chargement;

  @override
  void initState() {
    super.initState();
    _chargement = const DepotQuestionsAssets().charger();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Quiz')),
      body: FutureBuilder<BanqueQuestions>(
        future: _chargement,
        builder: (BuildContext context,
            AsyncSnapshot<BanqueQuestions> instantane) {
          if (instantane.connectionState != ConnectionState.done) {
            return const Center(child: CircularProgressIndicator());
          }
          if (instantane.hasError) {
            return Center(child: Text('Erreur : ${instantane.error}'));
          }
          final BanqueQuestions banque = instantane.data!;
          return Center(
            child: Text(
              '${banque.total} questions\n'
              '${banque.categories.length} catégories\n'
              'version ${banque.version}',
              textAlign: TextAlign.center,
            ),
          );
        },
      ),
    );
  }
}
```

**État exécutable.** `flutter run` affiche brièvement un cercle de chargement, puis :

```text
┌────────────────────────────────────────────────┐
│  Quiz                                          │
├────────────────────────────────────────────────┤
│                                                │
│                 18 questions                   │
│                 3 catégories                   │
│                   version 1                    │
│                                                │
└────────────────────────────────────────────────┘
```

Si vous obtenez `Unable to load asset: "assets/data/questions.json"`, c'est presque toujours l'une de ces trois causes :

| Cause | Correction |
| --- | --- |
| `assets:` mal indenté dans `pubspec.yaml` | deux espaces, sous `flutter:` |
| slash final oublié (`assets/data`) | écrire `assets/data/` |
| `pubspec.yaml` modifié sans relance | arrêter l'application et relancer `flutter run` |

Ce dernier point mérite d'être souligné : **le rechargement à chaud ne relit pas `pubspec.yaml`.** Après toute modification des assets ou des dépendances, il faut un redémarrage complet.

**État exécutable.** L'application lit sa banque de questions.

---
## 59.8 — Le moteur : la classe `Quiz`

Voici la pièce maîtresse. Une partie de quiz est un objet qui connaît :

```text
questions   la liste tirée pour cette partie, dans l'ordre
index       la question affichée en ce moment (0, 1, 2...)
reponses    ce que le joueur a répondu, dans l'ordre
```

Tout le reste — le score, le pourcentage, le nombre de bonnes réponses, la progression — se **calcule** à partir de ces trois champs. On ne stocke jamais une valeur qui peut être déduite : c'est la règle qui empêche deux informations de se contredire.

```text
MAUVAIS                            BON
─────────────────────              ─────────────────────
int score;                         int get score =>
void repondre(...) {                 _reponses.fold(0, ...);
  _reponses.add(...);
  score += points;    <- deux endroits à maintenir,
}                        donc un jour désynchronisés
```

### La réponse donnée

Chaque réponse mémorise trois choses : la question posée, le choix du joueur, et le temps qu'il restait. Le champ `indexChoisi` est nullable, et ce `null` a un sens précis : le temps s'est écoulé sans qu'aucun bouton ne soit touché.

**`lib/logique/quiz.dart`**

```dart
import '../modeles/question.dart';

/// Ce que le joueur a répondu à une question.
///
/// [indexChoisi] vaut `null` quand le temps s'est écoulé. C'est le
/// null safety du chapitre 12 utilisé pour porter une information :
/// « absence de choix » n'est pas « mauvais choix ».
class ReponseDonnee {
  const ReponseDonnee({
    required this.question,
    required this.indexChoisi,
    required this.secondesRestantes,
  });

  final Question question;
  final int? indexChoisi;
  final int secondesRestantes;

  /// `true` si le joueur a désigné la bonne réponse.
  bool get juste => question.estJuste(indexChoisi);

  /// `true` si le joueur n'a rien choisi à temps.
  bool get tempsEcoule => indexChoisi == null;

  /// Points rapportés : ceux de la difficulté, ou zéro.
  int get points => juste ? question.difficulte.points : 0;

  /// Texte de la réponse choisie, ou `null` si le temps est écoulé.
  String? get texteChoisi =>
      indexChoisi == null ? null : question.reponses[indexChoisi!];
}

/// Une partie de quiz en cours.
///
/// Cette classe est PURE : aucun import Flutter, aucun minuteur,
/// aucun affichage. Elle décrit les règles, rien d'autre. C'est ce qui
/// permet de la tester en quelques millisecondes au 59.22, et de
/// l'utiliser sans modification derrière `provider` au 59.22.
class Quiz {
  Quiz({required List<Question> questions})
      : _questions = List<Question>.unmodifiable(questions) {
    if (questions.isEmpty) {
      throw ArgumentError('Un quiz doit contenir au moins une question.');
    }
  }

  final List<Question> _questions;
  final List<ReponseDonnee> _reponses = <ReponseDonnee>[];
  int _index = 0;

  // ---------------------------------------------------------------
  // Lectures
  // ---------------------------------------------------------------

  List<Question> get questions => _questions;

  /// Position de la question courante, à partir de 0.
  int get index => _index;

  /// Numéro affiché à l'utilisateur, à partir de 1.
  int get numero => _index + 1;

  int get total => _questions.length;

  Question get questionCourante => _questions[_index];

  List<ReponseDonnee> get reponses => List<ReponseDonnee>.unmodifiable(_reponses);

  /// `true` si le joueur a déjà répondu à la question COURANTE.
  ///
  /// Les réponses sont ajoutées dans l'ordre : quand on est à l'index
  /// 3 et que la liste contient 4 éléments, la question courante a
  /// reçu sa réponse.
  bool get aRepondu => _reponses.length > _index;

  /// La réponse à la question courante, ou `null` si elle n'a pas
  /// encore été donnée.
  ReponseDonnee? get reponseCourante => aRepondu ? _reponses[_index] : null;

  /// `true` quand toutes les questions ont reçu une réponse.
  bool get termine => _reponses.length == total;

  /// `true` si la question courante est la dernière.
  bool get estDerniere => _index == total - 1;

  /// Avancement entre 0.0 et 1.0, pour la barre de progression.
  double get progression => _reponses.length / total;

  /// Score courant, recalculé à chaque lecture (chapitre 14).
  int get score =>
      _reponses.fold(0, (int somme, ReponseDonnee r) => somme + r.points);

  /// Score maximal atteignable dans cette partie.
  int get scoreMaximum => _questions.fold(
        0,
        (int somme, Question q) => somme + q.difficulte.points,
      );

  int get bonnesReponses =>
      _reponses.where((ReponseDonnee r) => r.juste).length;

  int get mauvaisesReponses =>
      _reponses.where((ReponseDonnee r) => !r.juste && !r.tempsEcoule).length;

  int get tempsEcoules =>
      _reponses.where((ReponseDonnee r) => r.tempsEcoule).length;

  /// Pourcentage du score maximal, entre 0.0 et 1.0.
  double get pourcentage => scoreMaximum == 0 ? 0 : score / scoreMaximum;

  // ---------------------------------------------------------------
  // Actions
  // ---------------------------------------------------------------

  /// Enregistre la réponse à la question courante.
  ///
  /// [indexChoisi] vaut `null` quand le temps est écoulé.
  ///
  /// Le second appel est ignoré. C'est ce qui verrouille la question
  /// (exigence O7) : deux appuis rapides sur deux boutons différents
  /// ne comptent que pour un. Cette protection est dans le MOTEUR, pas
  /// dans l'interface : c'est une règle du jeu, pas une décoration.
  void repondre(int? indexChoisi, {int secondesRestantes = 0}) {
    if (aRepondu) {
      return;
    }
    _reponses.add(
      ReponseDonnee(
        question: questionCourante,
        indexChoisi: indexChoisi,
        secondesRestantes: secondesRestantes,
      ),
    );
  }

  /// Passe à la question suivante.
  ///
  /// Renvoie `false` si le passage est impossible : soit on n'a pas
  /// encore répondu, soit on est déjà à la dernière question. Ce
  /// booléen évite à l'appelant de refaire les mêmes vérifications.
  bool passerALaSuivante() {
    if (!aRepondu || estDerniere) {
      return false;
    }
    _index++;
    return true;
  }

  /// Remet la partie à zéro, avec les mêmes questions.
  void recommencer() {
    _reponses.clear();
    _index = 0;
  }
}
```

### Vérification en console

```dart
// Collez difficulte.dart, question.dart et quiz.dart au-dessus.
void main() {
  final List<Question> questions = <Question>[
    Question(
      id: 'q1',
      categorie: 'test',
      enonce: '1 + 1 ?',
      reponses: <String>['1', '2'],
      indexBonneReponse: 1,
      explication: '',
      difficulte: Difficulte.facile, // 1 point
    ),
    Question(
      id: 'q2',
      categorie: 'test',
      enonce: '2 + 2 ?',
      reponses: <String>['4', '5'],
      indexBonneReponse: 0,
      explication: '',
      difficulte: Difficulte.difficile, // 3 points
    ),
  ];

  final Quiz quiz = Quiz(questions: questions);
  print('Départ : ${quiz.numero}/${quiz.total}, score ${quiz.score}');

  quiz.repondre(1, secondesRestantes: 12); // juste, +1
  quiz.repondre(0); // ignoré : déjà répondu
  print('Après Q1 : score ${quiz.score}, aRepondu ${quiz.aRepondu}');

  quiz.passerALaSuivante();
  quiz.repondre(null); // temps écoulé, +0
  print('Après Q2 : score ${quiz.score}, terminé ${quiz.termine}');
  print('Bonnes ${quiz.bonnesReponses}, temps écoulés ${quiz.tempsEcoules}');
  print('Maximum ${quiz.scoreMaximum}, pourcentage ${quiz.pourcentage}');
}
```

**Résultat :**

```text
Départ : 1/2, score 0
Après Q1 : score 1, aRepondu true
Après Q2 : score 1, terminé true
Bonnes 1, temps écoulés 1
Maximum 4, pourcentage 0.25
```

> **Remarque sur `List.unmodifiable`.** Les getters `questions` et `reponses` renvoient des listes non modifiables. Sans cela, un écran pourrait écrire `quiz.reponses.clear()` et vider la partie sans passer par une méthode. La liste non modifiable transforme cette faute en exception immédiate, à l'endroit exact où elle est commise.

**État exécutable.** Le moteur est écrit et vérifié. L'application ne l'utilise pas encore.

---

## 59.9 — Le mélange des questions et des réponses

Un quiz qui pose toujours les mêmes questions dans le même ordre s'apprend par cœur en trois parties. Il faut donc mélanger deux choses :

```text
1. les questions    on en tire 10 parmi 18, dans un ordre aléatoire
2. les réponses     la bonne réponse ne doit pas toujours être en B
```

Le second point est plus important qu'il n'y paraît. Si vous écrivez vos questions en plaçant systématiquement la bonne réponse en deuxième position, un joueur observateur s'en apercevra en quelques questions.

### L'outil : `shuffle`

`List.shuffle([Random? random])` mélange la liste **sur place**. Sans argument, il utilise une source aléatoire par défaut ; avec un `Random(graine)`, le mélange devient reproductible — indispensable pour écrire des tests.

```dart
final List<int> l = <int>[1, 2, 3, 4, 5];
l.shuffle(Random(42));   // toujours le même résultat pour la graine 42
```

### Mélanger les réponses d'une question

Mélanger les réponses casserait `indexBonneReponse`. Il faut donc retrouver l'indice après le mélange. La méthode est simple : mémoriser le **texte** de la bonne réponse avant, le rechercher après.

Ajoutez cette méthode à `Question` (fichier `lib/modeles/question.dart`), juste après `estJuste` :

```dart
  /// Renvoie une copie de la question avec les réponses mélangées.
  ///
  /// L'indice de la bonne réponse est recalculé après le mélange :
  /// on mémorise son TEXTE avant, on le recherche après.
  ///
  /// Limite connue : si deux propositions ont exactement le même
  /// texte, `indexOf` renverra la première. Ce n'est pas grave pour
  /// le score (les deux textes étant identiques, les deux sont
  /// « justes » à l'affichage), mais c'est le signe d'une question
  /// mal écrite. Ne dupliquez pas vos propositions.
  Question avecReponsesMelangees(Random alea) {
    final String texteBonneReponse = bonneReponse;
    final List<String> melangees = List<String>.of(reponses)..shuffle(alea);
    return Question(
      id: id,
      categorie: categorie,
      enonce: enonce,
      reponses: melangees,
      indexBonneReponse: melangees.indexOf(texteBonneReponse),
      explication: explication,
      difficulte: difficulte,
    );
  }
```

Ajoutez l'import correspondant en tête de `question.dart` :

```dart
import 'dart:math' show Random;
```

> **Le `..` de `List.of(reponses)..shuffle(alea)`.** C'est l'opérateur de cascade. `shuffle` ne renvoie rien (il modifie sur place) ; sans la cascade, `final l = List.of(r).shuffle(a)` affecterait `void` à `l`. La cascade dit « fais l'opération, puis rends-moi l'objet de départ ».

### Préparer une partie

**`lib/logique/melange.dart`**

```dart
import 'dart:math';

import '../modeles/question.dart';

/// Prépare la liste de questions d'une partie.
///
/// - tire au hasard [nombre] questions parmi [source] ;
/// - mélange l'ordre des questions ;
/// - mélange les réponses de chaque question retenue.
///
/// [alea] est injectable : en production on passe `null` (aléatoire
/// réel), dans les tests on passe `Random(42)` pour obtenir toujours
/// le même tirage. Cette injection est ce qui rend une fonction
/// aléatoire testable.
///
/// Si [nombre] dépasse la taille de [source], on prend tout : mieux
/// vaut une partie plus courte qu'une exception.
List<Question> preparerPartie(
  List<Question> source, {
  required int nombre,
  Random? alea,
}) {
  if (source.isEmpty) {
    return <Question>[];
  }
  final Random generateur = alea ?? Random();

  final List<Question> melangees = List<Question>.of(source)
    ..shuffle(generateur);

  final int combien = nombre <= 0 ? source.length : nombre;

  return melangees
      .take(combien)
      .map((Question q) => q.avecReponsesMelangees(generateur))
      .toList();
}
```

> **Pourquoi `take` et non `sublist` ?** `sublist(0, 10)` lève une exception si la liste contient 8 éléments. `take(10)` en renvoie 8 sans se plaindre. Le comportement souhaité ici est bien celui de `take` : une catégorie de 6 questions doit pouvoir se jouer même si l'utilisateur a demandé 10 questions.

### Vérification en console

```dart
// Avec difficulte.dart, question.dart, melange.dart collés au-dessus.
void main() {
  final List<Question> banque = List<Question>.generate(
    6,
    (int i) => Question(
      id: 'q$i',
      categorie: 'test',
      enonce: 'Question $i',
      reponses: <String>['A$i', 'B$i', 'C$i'],
      indexBonneReponse: 0, // la bonne est toujours en position 0
      explication: '',
      difficulte: Difficulte.facile,
    ),
  );

  final List<Question> partie =
      preparerPartie(banque, nombre: 3, alea: Random(42));

  for (final Question q in partie) {
    print('${q.id} -> ${q.reponses}, bonne = ${q.bonneReponse} '
        '(index ${q.indexBonneReponse})');
  }
}
```

**Résultat :** l'ordre exact dépend de l'implémentation de `Random`, mais deux propriétés sont garanties et c'est tout ce qui compte :

```text
q4 -> [C4, A4, B4], bonne = A4 (index 1)
q1 -> [A1, C1, B1], bonne = A1 (index 0)
q5 -> [B5, C5, A5], bonne = A5 (index 2)
```

Trois questions sur six, et la bonne réponse n'est plus systématiquement en première position. C'est précisément ce que les tests du 59.24 vérifieront.

**État exécutable.** Les fichiers compilent. L'interface arrive maintenant.

---

## 59.10 — L'écran d'accueil et le choix de la catégorie

L'accueil doit permettre trois choses : choisir une catégorie, choisir un nombre de questions, lancer la partie. Il affiche aussi le meilleur score, que nous brancherons au 59.21.

### Traduire un nom d'icône en `IconData`

Le modèle `Categorie` porte une chaîne (`code`, `widgets`, `manette`). C'est ici, dans l'interface, qu'on la convertit.

**`lib/utilitaires/couleurs_reponse.dart`** — nous y reviendrons au 59.13, commençons par la partie icônes.

```dart
import 'package:flutter/material.dart';

/// Convertit le nom d'icône d'une catégorie en icône Material.
///
/// Le modèle ne connaît pas Flutter : il transporte une simple
/// chaîne. La correspondance vit ici, dans la couche interface.
/// Ajouter une catégorie au JSON sans toucher à cette fonction
/// affichera l'icône par défaut, sans planter.
IconData iconeDeCategorie(String nom) {
  switch (nom) {
    case 'code':
      return Icons.code;
    case 'widgets':
      return Icons.widgets_outlined;
    case 'manette':
      return Icons.sports_esports_outlined;
    default:
      return Icons.help_outline;
  }
}
```

### L'écran

**`lib/ecrans/ecran_accueil.dart`**

```dart
import 'package:flutter/material.dart';

import '../donnees/depot_questions.dart';
import '../logique/banque_questions.dart';
import '../modeles/categorie.dart';
import '../utilitaires/couleurs_reponse.dart';

/// Écran d'accueil : choix de la catégorie et du nombre de questions.
///
/// À ce stade il gère lui-même son chargement et son état, avec
/// `setState`. Le 59.22 déplacera tout cela dans un `ChangeNotifier`.
class EcranAccueil extends StatefulWidget {
  const EcranAccueil({super.key, required this.depot});

  final DepotQuestions depot;

  @override
  State<EcranAccueil> createState() => _EcranAccueilState();
}

class _EcranAccueilState extends State<EcranAccueil> {
  late final Future<BanqueQuestions> _chargement;

  /// `null` signifie « toutes les catégories ».
  String? _categorieChoisie;

  /// Nombre de questions demandé ; 0 signifie « toutes ».
  int _nombreChoisi = 10;

  @override
  void initState() {
    super.initState();
    // Créé une seule fois : voir la remarque du 59.7.
    _chargement = widget.depot.charger();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Quiz')),
      body: FutureBuilder<BanqueQuestions>(
        future: _chargement,
        builder: (BuildContext context, AsyncSnapshot<BanqueQuestions> etat) {
          if (etat.connectionState != ConnectionState.done) {
            return const Center(child: CircularProgressIndicator());
          }
          if (etat.hasError) {
            return _Erreur(message: '${etat.error}');
          }
          return _Formulaire(
            banque: etat.data!,
            categorieChoisie: _categorieChoisie,
            nombreChoisi: _nombreChoisi,
            surCategorie: (String? id) =>
                setState(() => _categorieChoisie = id),
            surNombre: (int n) => setState(() => _nombreChoisi = n),
            surDemarrer: () {
              // Le lancement de la partie arrive au 59.11.
            },
          );
        },
      ),
    );
  }
}

/// Écran d'erreur de chargement (exigence O4).
///
/// Un message technique brut est illisible ; on l'encadre d'une
/// explication et on le laisse visible pour le développeur.
class _Erreur extends StatelessWidget {
  const _Erreur({required this.message});

  final String message;

  @override
  Widget build(BuildContext context) {
    final ColorScheme couleurs = Theme.of(context).colorScheme;
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Icon(Icons.error_outline, size: 64, color: couleurs.error),
          const SizedBox(height: 16),
          Text(
            'Impossible de charger les questions',
            style: Theme.of(context).textTheme.titleLarge,
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 8),
          Text(
            'Vérifiez le fichier assets/data/questions.json '
            'et sa déclaration dans pubspec.yaml.',
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 16),
          Text(
            message,
            style: TextStyle(color: couleurs.error, fontSize: 12),
            textAlign: TextAlign.center,
          ),
        ],
      ),
    );
  }
}

/// Le formulaire de configuration de la partie.
class _Formulaire extends StatelessWidget {
  const _Formulaire({
    required this.banque,
    required this.categorieChoisie,
    required this.nombreChoisi,
    required this.surCategorie,
    required this.surNombre,
    required this.surDemarrer,
  });

  final BanqueQuestions banque;
  final String? categorieChoisie;
  final int nombreChoisi;
  final ValueChanged<String?> surCategorie;
  final ValueChanged<int> surNombre;
  final VoidCallback surDemarrer;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final int disponibles = banque.compte(categorieChoisie);

    return ListView(
      padding: const EdgeInsets.all(16),
      children: <Widget>[
        // --- En-tête ---
        const SizedBox(height: 16),
        Icon(Icons.quiz_outlined,
            size: 72, color: theme.colorScheme.primary),
        const SizedBox(height: 8),
        Text(
          'QUIZ',
          textAlign: TextAlign.center,
          style: theme.textTheme.headlineMedium?.copyWith(
            fontWeight: FontWeight.bold,
            letterSpacing: 4,
          ),
        ),
        const SizedBox(height: 4),
        Text(
          '${banque.total} questions, ${banque.categories.length} catégories',
          textAlign: TextAlign.center,
          style: theme.textTheme.bodyMedium,
        ),
        const SizedBox(height: 32),

        // --- Catégories ---
        Text('Catégorie', style: theme.textTheme.titleMedium),
        const SizedBox(height: 8),
        Card(
          margin: EdgeInsets.zero,
          child: Column(
            children: <Widget>[
              _LigneCategorie(
                icone: Icons.all_inclusive,
                titre: 'Toutes les catégories',
                sousTitre: 'Un mélange des trois thèmes',
                compte: banque.total,
                choisie: categorieChoisie == null,
                surAppui: () => surCategorie(null),
              ),
              for (final Categorie c in banque.categories) ...<Widget>[
                const Divider(height: 1),
                _LigneCategorie(
                  icone: iconeDeCategorie(c.icone),
                  titre: c.nom,
                  sousTitre: c.description,
                  compte: banque.compte(c.id),
                  choisie: categorieChoisie == c.id,
                  surAppui: () => surCategorie(c.id),
                ),
              ],
            ],
          ),
        ),
        const SizedBox(height: 24),

        // --- Nombre de questions ---
        Text('Nombre de questions', style: theme.textTheme.titleMedium),
        const SizedBox(height: 8),
        Wrap(
          spacing: 8,
          children: <Widget>[
            for (final int n in <int>[5, 10, 0])
              ChoiceChip(
                label: Text(n == 0 ? 'Toutes' : '$n'),
                selected: nombreChoisi == n,
                onSelected: (bool _) => surNombre(n),
              ),
          ],
        ),
        const SizedBox(height: 8),
        Text(
          'La partie comptera '
          '${_nombreEffectif(nombreChoisi, disponibles)} question(s).',
          style: theme.textTheme.bodySmall,
        ),
        const SizedBox(height: 32),

        // --- Lancement ---
        FilledButton.icon(
          onPressed: disponibles == 0 ? null : surDemarrer,
          icon: const Icon(Icons.play_arrow),
          label: const Padding(
            padding: EdgeInsets.symmetric(vertical: 12),
            child: Text('COMMENCER'),
          ),
        ),
      ],
    );
  }

  /// Combien de questions la partie contiendra réellement.
  static int _nombreEffectif(int demande, int disponibles) {
    if (demande <= 0 || demande > disponibles) {
      return disponibles;
    }
    return demande;
  }
}

/// Une ligne de la liste des catégories, avec son bouton radio.
class _LigneCategorie extends StatelessWidget {
  const _LigneCategorie({
    required this.icone,
    required this.titre,
    required this.sousTitre,
    required this.compte,
    required this.choisie,
    required this.surAppui,
  });

  final IconData icone;
  final String titre;
  final String sousTitre;
  final int compte;
  final bool choisie;
  final VoidCallback surAppui;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Icon(icone),
      title: Text(titre),
      subtitle: Text(sousTitre, maxLines: 1, overflow: TextOverflow.ellipsis),
      trailing: Text('$compte q.'),
      selected: choisie,
      onTap: surAppui,
      // Le lecteur d'écran annonce « sélectionné » grâce à `selected`.
      // Le point sur la gauche viendrait d'un Radio ; ici la teinte de
      // sélection de Material 3 suffit, et évite un second point
      // d'appui pour la même action.
    );
  }
}
```

### Brancher l'écran

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';

import 'donnees/depot_questions.dart';
import 'ecrans/ecran_accueil.dart';

void main() {
  runApp(const ApplicationQuiz());
}

class ApplicationQuiz extends StatelessWidget {
  const ApplicationQuiz({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Quiz',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      // Une seule ligne à changer pour jouer sur des questions de test :
      // home: EcranAccueil(depot: DepotQuestionsMemoire()),
      home: const EcranAccueil(depot: DepotQuestionsAssets()),
    );
  }
}
```

**État exécutable.** L'accueil s'affiche, les catégories sont sélectionnables, le compteur en bas se met à jour. Le bouton COMMENCER ne fait encore rien.

```text
┌────────────────────────────────────────────────┐
│  Quiz                                          │
├────────────────────────────────────────────────┤
│                   ▣ QUIZ                       │
│         18 questions, 3 catégories             │
│                                                │
│  Catégorie                                     │
│  ┌──────────────────────────────────────────┐  │
│  │ ∞  Toutes les catégories         18 q.   │  │
│  │    Un mélange des trois thèmes            │ │
│  ├──────────────────────────────────────────┤  │
│  │ <> Dart                           6 q.   │  │
│  │    Le langage : syntaxe, types...         │ │
│  └──────────────────────────────────────────┘  │
│                                                │
│  Nombre de questions                           │
│  [ 5 ]  [(10)]  [ Toutes ]                     │
│  La partie comptera 10 question(s).            │
│                                                │
│         ┌──────────────────────────────┐       │
│         │      ▶  COMMENCER            │       │
│         └──────────────────────────────┘       │
└────────────────────────────────────────────────┘
```

---
## 59.11 — L'écran de question

L'écran de question est le seul écran de jeu. Il affiche une question, reçoit une réponse, puis passe à la suivante. Dans cette première version, un appui sur une proposition enchaîne immédiatement ; le retour visuel viendra au 59.12.

L'écran reçoit le `Quiz` déjà construit. Il ne le fabrique pas lui-même : construire la partie est le travail de l'accueil, jouer la partie est le travail de cet écran. Cette séparation est ce qui rendra le passage à `provider` presque indolore au 59.21.

**`lib/ecrans/ecran_question.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/quiz.dart';
import '../modeles/question.dart';

/// L'écran de jeu.
///
/// Il reçoit un [Quiz] déjà préparé (questions tirées et mélangées)
/// et le fait avancer. Il est `Stateful` car le déroulement de la
/// partie change au fil des appuis.
class EcranQuestion extends StatefulWidget {
  const EcranQuestion({super.key, required this.quiz});

  final Quiz quiz;

  @override
  State<EcranQuestion> createState() => _EcranQuestionState();
}

class _EcranQuestionState extends State<EcranQuestion> {
  Quiz get _quiz => widget.quiz;

  /// Enregistre la réponse choisie, puis reconstruit l'écran.
  ///
  /// Le moteur ignore un second appel : inutile de s'en protéger ici.
  void _repondre(int index) {
    setState(() {
      _quiz.repondre(index);
    });
    // Enchaînement provisoire. Le 59.12 le supprimera au profit d'un
    // bouton explicite, pour laisser le temps de lire la correction.
    _suivante();
  }

  /// Passe à la question suivante, ou termine la partie.
  void _suivante() {
    if (_quiz.estDerniere) {
      // L'écran de résultat arrive au 59.18. Pour l'instant on revient
      // à l'accueil.
      Navigator.of(context).pop();
      return;
    }
    setState(() {
      _quiz.passerALaSuivante();
    });
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Question question = _quiz.questionCourante;

    return Scaffold(
      appBar: AppBar(
        title: Text('Question ${_quiz.numero} / ${_quiz.total}'),
        actions: <Widget>[
          Center(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Text(
                '${_quiz.score} pts',
                style: theme.textTheme.titleMedium,
              ),
            ),
          ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          // Bandeau catégorie et difficulté.
          Text(
            '${question.categorie.toUpperCase()} · '
            '${question.difficulte.libelle.toUpperCase()}',
            style: theme.textTheme.labelMedium?.copyWith(
              color: theme.colorScheme.primary,
              letterSpacing: 1.5,
            ),
          ),
          const SizedBox(height: 12),

          // L'énoncé, dans une carte pour le détacher des réponses.
          Card(
            margin: EdgeInsets.zero,
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: Text(
                question.enonce,
                style: theme.textTheme.titleLarge,
              ),
            ),
          ),
          const SizedBox(height: 24),

          // Les propositions, provisoirement de simples boutons.
          for (int i = 0; i < question.reponses.length; i++)
            Padding(
              padding: const EdgeInsets.only(bottom: 8),
              child: OutlinedButton(
                onPressed: () => _repondre(i),
                child: Align(
                  alignment: Alignment.centerLeft,
                  child: Text(
                    '${Question.lettre(i)}   ${question.reponses[i]}',
                  ),
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```

### Lancer la partie depuis l'accueil

Dans `ecran_accueil.dart`, complétez le rappel `surDemarrer` du `FutureBuilder` :

```dart
            surDemarrer: () {
              final List<Question> questions = preparerPartie(
                etat.data!.pourCategorie(_categorieChoisie),
                nombre: _nombreChoisi,
              );
              Navigator.of(context).push(
                MaterialPageRoute<void>(
                  builder: (BuildContext _) =>
                      EcranQuestion(quiz: Quiz(questions: questions)),
                ),
              );
            },
```

et ajoutez les imports correspondants en tête du fichier :

```dart
import '../logique/melange.dart';
import '../logique/quiz.dart';
import '../modeles/question.dart';
import 'ecran_question.dart';
```

> **Remarque.** `Navigator.push` (chapitre 50) empile l'écran de jeu **au-dessus** de l'accueil. Le retour arrière ramène donc à l'accueil sans le reconstruire, et sans relire le fichier JSON. C'est exactement ce que l'on veut.

**État exécutable.** COMMENCER lance une partie. Les questions s'enchaînent, le score augmente dans la barre d'application, et la dernière question ramène à l'accueil. Le quiz est jouable, mais aveugle : on ne sait pas si l'on a bien répondu.

---

## 59.12 — Les boutons de réponse et le retour visuel immédiat

C'est la section la plus importante de l'interface. Un bouton de réponse n'a pas deux apparences mais **cinq**, selon l'état de la question et le choix du joueur :

```text
état            quand ?                                  apparence
──────────────  ───────────────────────────────────────  ──────────────
neutre          on n'a pas encore répondu                bordure grise
choisieJuste    on a choisi CE bouton, il est juste      fond vert, ✓
choisieFausse   on a choisi CE bouton, il est faux       fond rouge, ✕
bonneRevelee    on a répondu ailleurs, CE bouton est     bordure verte, ✓
                la bonne réponse
ignoree         on a répondu, ce bouton n'est ni le      grisé
                choix ni la bonne réponse
```

Écrire ces cinq cas avec des `if` imbriqués dans le `build` donnerait un code illisible. On les nomme donc dans un `enum`, et une fonction pure décide de l'état.

### Les couleurs

Vert et rouge ne viennent pas du `ColorScheme` : ce sont des couleurs **sémantiques**, dont le sens ne doit pas changer avec le thème. On les définit une fois, avec une variante pour le mode sombre où un vert trop saturé fatigue l'œil.

Complétez **`lib/utilitaires/couleurs_reponse.dart`** (déjà créé au 59.10) :

```dart
/// Couleurs sémantiques de correction.
///
/// Elles ne viennent pas du ColorScheme : « juste » doit rester vert
/// même si l'on change la couleur d'amorce du thème. On prévoit
/// simplement deux nuances, l'une pour le fond clair, l'autre pour le
/// fond sombre, afin de conserver un contraste suffisant.
class CouleursReponse {
  const CouleursReponse._();

  static const Color _justeClair = Color(0xFF2E7D32); // green 800
  static const Color _justeSombre = Color(0xFF81C784); // green 300
  static const Color _fauxClair = Color(0xFFC62828); // red 800
  static const Color _fauxSombre = Color(0xFFE57373); // red 300

  static bool _sombre(BuildContext context) =>
      Theme.of(context).brightness == Brightness.dark;

  static Color juste(BuildContext context) =>
      _sombre(context) ? _justeSombre : _justeClair;

  static Color faux(BuildContext context) =>
      _sombre(context) ? _fauxSombre : _fauxClair;
}
```

### Le widget

**`lib/widgets/bouton_reponse.dart`**

```dart
import 'package:flutter/material.dart';

import '../modeles/question.dart';
import '../utilitaires/couleurs_reponse.dart';

/// Les cinq apparences possibles d'un bouton de réponse.
enum EtatReponse { neutre, choisieJuste, choisieFausse, bonneRevelee, ignoree }

/// Détermine l'état d'un bouton.
///
/// Fonction pure : mêmes entrées, même sortie, aucun effet de bord.
/// Elle est donc testable sans écran (voir 59.23).
///
/// [indexBouton]  la position de ce bouton.
/// [indexBonne]   la position de la bonne réponse.
/// [indexChoisi]  ce que le joueur a choisi ; `null` s'il n'a pas
///                encore répondu OU si le temps est écoulé.
/// [repondu]      `true` dès que la question est verrouillée.
EtatReponse etatDuBouton({
  required int indexBouton,
  required int indexBonne,
  required int? indexChoisi,
  required bool repondu,
}) {
  if (!repondu) {
    return EtatReponse.neutre;
  }
  if (indexBouton == indexChoisi) {
    return indexBouton == indexBonne
        ? EtatReponse.choisieJuste
        : EtatReponse.choisieFausse;
  }
  if (indexBouton == indexBonne) {
    return EtatReponse.bonneRevelee;
  }
  return EtatReponse.ignoree;
}

/// Un bouton de réponse.
class BoutonReponse extends StatelessWidget {
  const BoutonReponse({
    super.key,
    required this.index,
    required this.texte,
    required this.etat,
    required this.surAppui,
  });

  final int index;
  final String texte;
  final EtatReponse etat;

  /// `null` quand la question est verrouillée : le bouton devient
  /// inactif, et le lecteur d'écran l'annonce comme tel.
  final VoidCallback? surAppui;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final Color vert = CouleursReponse.juste(context);
    final Color rouge = CouleursReponse.faux(context);

    Color? fond;
    Color bordure = theme.colorScheme.outlineVariant;
    Color texteCouleur = theme.colorScheme.onSurface;
    IconData? icone;
    double opacite = 1;

    switch (etat) {
      case EtatReponse.neutre:
        break;
      case EtatReponse.choisieJuste:
        fond = vert.withValues(alpha: 0.15);
        bordure = vert;
        texteCouleur = vert;
        icone = Icons.check_circle;
      case EtatReponse.choisieFausse:
        fond = rouge.withValues(alpha: 0.15);
        bordure = rouge;
        texteCouleur = rouge;
        icone = Icons.cancel;
      case EtatReponse.bonneRevelee:
        bordure = vert;
        texteCouleur = vert;
        icone = Icons.check_circle_outline;
      case EtatReponse.ignoree:
        opacite = 0.5;
    }

    return Opacity(
      opacity: opacite,
      child: AnimatedContainer(
        // La transition de couleur dure 200 ms : assez pour être
        // perçue comme un changement, assez court pour ne pas
        // retarder la lecture.
        duration: const Duration(milliseconds: 200),
        margin: const EdgeInsets.only(bottom: 10),
        decoration: BoxDecoration(
          color: fond,
          border: Border.all(color: bordure, width: 2),
          borderRadius: BorderRadius.circular(12),
        ),
        child: Material(
          color: Colors.transparent,
          child: InkWell(
            onTap: surAppui,
            borderRadius: BorderRadius.circular(12),
            child: Padding(
              padding: const EdgeInsets.symmetric(
                horizontal: 16,
                vertical: 14,
              ),
              child: Row(
                children: <Widget>[
                  // La lettre A, B, C, D dans une pastille.
                  Container(
                    width: 28,
                    height: 28,
                    alignment: Alignment.center,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      border: Border.all(color: bordure),
                    ),
                    child: Text(
                      Question.lettre(index),
                      style: TextStyle(
                        color: texteCouleur,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: Text(
                      texte,
                      style: theme.textTheme.bodyLarge
                          ?.copyWith(color: texteCouleur),
                    ),
                  ),
                  // L'icône double l'information portée par la
                  // couleur : indispensable pour les daltoniens
                  // (voir 59.22).
                  if (icone != null) ...<Widget>[
                    const SizedBox(width: 8),
                    Icon(icone, color: texteCouleur),
                  ],
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

### Ne jamais bloquer l'interface

Le réflexe du débutant, à ce stade, est d'écrire ceci :

```dart
// À NE PAS FAIRE
void _repondre(int index) async {
  setState(() => _quiz.repondre(index));
  await Future<void>.delayed(const Duration(seconds: 2)); // on "laisse voir"
  _suivante();                                            // puis on enchaîne
}
```

Trois défauts, tous graves :

| Défaut | Conséquence |
| --- | --- |
| L'utilisateur ne décide plus du rythme | Le lecteur lent n'a pas fini de lire, le lecteur rapide s'ennuie. |
| Le `setState` après `await` peut arriver sur un écran détruit | `setState() called after dispose()` si l'on quitte pendant l'attente. |
| Le minuteur du 59.15 continue de tourner pendant l'attente | La question suivante démarre avec un temps déjà entamé. |

La règle est simple : **le retour visuel est immédiat, l'avancement est déclenché par l'utilisateur.** On change l'état, on reconstruit, et on affiche un bouton « QUESTION SUIVANTE ». Aucun `await`, aucun `showDialog`, aucun blocage.

### Brancher dans l'écran

Dans `_EcranQuestionState`, remplacez `_repondre` et la boucle de propositions :

```dart
  void _repondre(int index) {
    setState(() {
      _quiz.repondre(index);
    });
    // Et c'est tout. Pas d'attente, pas d'enchaînement automatique.
  }
```

```dart
          // Les propositions.
          for (int i = 0; i < question.reponses.length; i++)
            BoutonReponse(
              index: i,
              texte: question.reponses[i],
              etat: etatDuBouton(
                indexBouton: i,
                indexBonne: question.indexBonneReponse,
                indexChoisi: _quiz.reponseCourante?.indexChoisi,
                repondu: _quiz.aRepondu,
              ),
              surAppui: _quiz.aRepondu ? null : () => _repondre(i),
            ),

          // Le bouton d'avancement n'apparaît qu'une fois la réponse
          // donnée. Avant, il n'aurait aucun sens.
          if (_quiz.aRepondu) ...<Widget>[
            const SizedBox(height: 8),
            FilledButton.icon(
              onPressed: _suivante,
              icon: const Icon(Icons.arrow_forward),
              label: Padding(
                padding: const EdgeInsets.symmetric(vertical: 12),
                child: Text(
                  _quiz.estDerniere ? 'VOIR LE RÉSULTAT' : 'QUESTION SUIVANTE',
                ),
              ),
            ),
          ],
```

et ajoutez l'import :

```dart
import '../widgets/bouton_reponse.dart';
```

> **`withValues(alpha: 0.15)`.** C'est la méthode actuelle pour obtenir une couleur translucide. L'ancienne, `withOpacity(0.15)`, est dépréciée depuis Flutter 3.27 : elle perdait de la précision sur les espaces colorimétriques larges. Si votre analyseur signale `withValues` comme inconnue, votre SDK est plus ancien que celui de cette formation.

**État exécutable.** Répondre colore immédiatement le bouton choisi en vert ou en rouge, révèle la bonne réponse, estompe les autres et fait apparaître le bouton d'avancement.

```text
│  ┌──────────────────────────────────────────┐  │
│  │ (A)  Un Stream                       ✕   │  │  rouge
│  ├──────────────────────────────────────────┤  │
│  │ (B)  Un Future                       ✓   │  │  vert
│  ├──────────────────────────────────────────┤  │
│  │ (C)  void                                │  │  estompé
│  └──────────────────────────────────────────┘  │
│         ┌──────────────────────────────┐       │
│         │  →  QUESTION SUIVANTE        │       │
│         └──────────────────────────────┘       │
```

---

## 59.13 — L'explication après la réponse

Un quiz qui se contente de dire « faux » n'apprend rien. L'explication, écrite dans le JSON au 59.5, est la seule partie réellement pédagogique de l'application.

Elle apparaît seulement après la réponse, et son en-tête change selon les trois cas : juste, faux, temps écoulé.

**`lib/widgets/carte_explication.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/quiz.dart';
import '../utilitaires/couleurs_reponse.dart';

/// La carte de correction affichée sous les réponses.
///
/// Elle prend la [ReponseDonnee] plutôt que trois booléens : l'objet
/// contient déjà tout ce qu'il faut, et un futur champ (le temps mis,
/// par exemple) ne changera pas cette signature.
class CarteExplication extends StatelessWidget {
  const CarteExplication({super.key, required this.reponse});

  final ReponseDonnee reponse;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    late final Color couleur;
    late final IconData icone;
    late final String titre;

    if (reponse.tempsEcoule) {
      couleur = theme.colorScheme.tertiary;
      icone = Icons.timer_off_outlined;
      titre = 'Temps écoulé';
    } else if (reponse.juste) {
      couleur = CouleursReponse.juste(context);
      icone = Icons.check_circle;
      titre = 'Bonne réponse, +${reponse.points} pt';
    } else {
      couleur = CouleursReponse.faux(context);
      icone = Icons.cancel;
      titre = 'Mauvaise réponse';
    }

    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: couleur.withValues(alpha: 0.10),
        border: Border.all(color: couleur.withValues(alpha: 0.4)),
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          Row(
            children: <Widget>[
              Icon(icone, color: couleur, size: 20),
              const SizedBox(width: 8),
              Text(
                titre,
                style: theme.textTheme.titleSmall?.copyWith(color: couleur),
              ),
            ],
          ),
          // Quand le temps est écoulé, on rappelle la bonne réponse :
          // le joueur n'a rien choisi, il n'a donc aucun repère.
          if (reponse.tempsEcoule) ...<Widget>[
            const SizedBox(height: 8),
            Text('La bonne réponse était : ${reponse.question.bonneReponse}'),
          ],
          // Une question peut ne pas avoir d'explication : on
          // n'affiche pas un bloc vide.
          if (reponse.question.explication.isNotEmpty) ...<Widget>[
            const SizedBox(height: 8),
            Text(
              reponse.question.explication,
              style: theme.textTheme.bodyMedium,
            ),
          ],
        ],
      ),
    );
  }
}
```

Dans `ecran_question.dart`, insérez la carte juste avant le bouton d'avancement :

```dart
          if (_quiz.aRepondu) ...<Widget>[
            const SizedBox(height: 8),
            CarteExplication(reponse: _quiz.reponseCourante!),
            const SizedBox(height: 16),
            FilledButton.icon(
              // ... inchangé
```

avec l'import `import '../widgets/carte_explication.dart';`.

> **Le `!` de `_quiz.reponseCourante!`.** Il est ici légitime : nous sommes à l'intérieur d'un `if (_quiz.aRepondu)`, et `reponseCourante` ne vaut `null` que lorsque `aRepondu` est faux. Le compilateur ne peut pas relier ces deux getters, nous le lui affirmons. C'est l'usage correct de `!` décrit au chapitre 12 : une affirmation que le lecteur peut vérifier en deux lignes.

**État exécutable.** L'explication apparaît sous les réponses, avec l'en-tête adapté.

---

## 59.14 — La barre de progression

Deux informations manquent au joueur : où il en est, et combien il lui reste. Le titre « Question 3 / 10 » donne la première ; une barre donne la seconde, d'un coup d'œil.

On l'anime avec `TweenAnimationBuilder` : à chaque changement de valeur, le widget interpole entre l'ancienne et la nouvelle. La barre glisse au lieu de sauter.

**`lib/widgets/barre_progression.dart`**

```dart
import 'package:flutter/material.dart';

/// Barre d'avancement animée.
///
/// [valeur] est comprise entre 0.0 et 1.0.
///
/// `TweenAnimationBuilder` mémorise la valeur précédente et anime
/// jusqu'à la nouvelle à chaque reconstruction. Aucun contrôleur
/// d'animation à créer ni à libérer : c'est le moyen le plus court
/// d'animer une valeur qui change par à-coups.
class BarreProgression extends StatelessWidget {
  const BarreProgression({super.key, required this.valeur});

  final double valeur;

  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder<double>(
      tween: Tween<double>(begin: 0, end: valeur.clamp(0.0, 1.0)),
      duration: const Duration(milliseconds: 400),
      curve: Curves.easeOut,
      builder: (BuildContext context, double v, Widget? _) {
        return LinearProgressIndicator(
          value: v,
          minHeight: 8,
          borderRadius: BorderRadius.circular(4),
          // Le lecteur d'écran annonce « progression, 30 % ».
          semanticsLabel: 'Progression du quiz',
          semanticsValue: '${(v * 100).round()} %',
        );
      },
    );
  }
}
```

Dans `ecran_question.dart`, on la place sous la barre d'application, à l'aide de la propriété `bottom` d'`AppBar`. Cette propriété attend un widget qui déclare sa hauteur, d'où `PreferredSize`.

```dart
      appBar: AppBar(
        title: Text('Question ${_quiz.numero} / ${_quiz.total}'),
        actions: <Widget>[ /* ... inchangé ... */ ],
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(8),
          child: BarreProgression(valeur: _quiz.progression),
        ),
      ),
```

avec `import '../widgets/barre_progression.dart';`.

> **Attention à ce que mesure la barre.** `progression` vaut `reponses.length / total`, et non `index / total`. À la première question sans réponse, la barre est à 0 % ; après avoir répondu, elle est à 10 %. Si vous utilisiez l'index, la barre resterait à 0 % sur les deux premières questions et n'atteindrait jamais 100 %.

**État exécutable.** Une barre fine glisse sous la barre d'application à chaque réponse.

---
## 59.15 — Le compte à rebours par question

Chaque question est chronométrée, et la durée dépend de la difficulté : 20 s en facile, 15 s en moyenne, 10 s en difficile. Ces valeurs sont déjà dans l'`enum` du 59.2 ; il n'y a rien à décider ici, seulement à lire.

L'outil est `Timer.periodic`, vu au chapitre 45 :

```dart
Timer.periodic(Duration duration, void Function(Timer timer) callback)
```

Trois règles, apprises à la dure par tous les développeurs Flutter :

```text
1. Un Timer se crée dans initState, jamais dans build.
   Sinon chaque reconstruction en crée un de plus.

2. Un Timer s'annule dans dispose.
   Sinon il continue de tourner sur un écran détruit, et le premier
   setState lève « setState() called after dispose() ».

3. Avant de créer un Timer, on annule le précédent.
   `_minuteur?.cancel()` en première ligne : une ligne qui évite le
   bug le plus difficile à reproduire de tout le projet.
```

### Le widget d'affichage

**`lib/widgets/compte_a_rebours.dart`**

```dart
import 'package:flutter/material.dart';

import '../utilitaires/couleurs_reponse.dart';

/// Pastille circulaire affichant les secondes restantes.
///
/// Ce widget n'a AUCUNE logique de temps : il reçoit deux nombres et
/// dessine. Le minuteur est ailleurs. Un widget qui compte le temps
/// lui-même serait impossible à mettre en pause depuis l'extérieur.
class CompteARebours extends StatelessWidget {
  const CompteARebours({
    super.key,
    required this.restantes,
    required this.total,
    required this.actif,
  });

  final int restantes;
  final int total;

  /// `false` une fois la question répondue : le cercle se fige.
  final bool actif;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final bool urgence = actif && restantes <= 5;
    final Color couleur = urgence
        ? CouleursReponse.faux(context)
        : theme.colorScheme.primary;

    return Semantics(
      // Sans cela, le lecteur d'écran ne lit qu'un nombre nu.
      label: 'Temps restant',
      value: '$restantes secondes',
      child: SizedBox(
        width: 48,
        height: 48,
        child: Stack(
          alignment: Alignment.center,
          children: <Widget>[
            CircularProgressIndicator(
              value: total == 0 ? 0 : restantes / total,
              strokeWidth: 4,
              color: couleur,
              backgroundColor: theme.colorScheme.surfaceContainerHighest,
            ),
            Text(
              '$restantes',
              style: theme.textTheme.titleMedium?.copyWith(
                color: couleur,
                fontWeight: urgence ? FontWeight.bold : FontWeight.normal,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Le minuteur dans l'écran

Dans `ecran_question.dart`, ajoutez `import 'dart:async';` et `import '../widgets/compte_a_rebours.dart';`, puis remplacez le corps de l'état :

```dart
class _EcranQuestionState extends State<EcranQuestion> {
  Quiz get _quiz => widget.quiz;

  Timer? _minuteur;
  int _secondesRestantes = 0;

  @override
  void initState() {
    super.initState();
    _demarrerMinuteur();
  }

  @override
  void dispose() {
    // Règle 2 : sans cette ligne, quitter l'écran en cours de
    // question laisserait un minuteur vivant.
    _minuteur?.cancel();
    super.dispose();
  }

  /// (Re)démarre le compte à rebours pour la question courante.
  ///
  /// Cette méthode ne fait PAS de `setState` : elle est toujours
  /// appelée depuis `initState` (où la reconstruction suit de toute
  /// façon) ou depuis `_suivante` (qui reconstruit ensuite).
  void _demarrerMinuteur() {
    _minuteur?.cancel(); // règle 3
    _secondesRestantes = _quiz.questionCourante.difficulte.secondes;
    _minuteur = Timer.periodic(const Duration(seconds: 1), (Timer t) {
      if (_secondesRestantes <= 1) {
        _repondre(null); // expiration : voir 59.16
      } else {
        setState(() => _secondesRestantes--);
      }
    });
  }

  void _arreterMinuteur() {
    _minuteur?.cancel();
    _minuteur = null;
  }

  /// Enregistre une réponse. [index] vaut `null` si le temps a expiré.
  void _repondre(int? index) {
    _arreterMinuteur();
    setState(() {
      _quiz.repondre(
        index,
        secondesRestantes: index == null ? 0 : _secondesRestantes,
      );
      if (index == null) {
        _secondesRestantes = 0;
      }
    });
  }

  void _suivante() {
    if (_quiz.estDerniere) {
      Navigator.of(context).pop();
      return;
    }
    _quiz.passerALaSuivante();
    _demarrerMinuteur();
    setState(() {}); // une seule reconstruction pour les deux changements
  }

  // build() : voir plus bas.
}
```

Enfin, affichez la pastille à droite du bandeau catégorie, dans `build` :

```dart
          Row(
            children: <Widget>[
              Expanded(
                child: Text(
                  '${question.categorie.toUpperCase()} · '
                  '${question.difficulte.libelle.toUpperCase()}',
                  style: theme.textTheme.labelMedium?.copyWith(
                    color: theme.colorScheme.primary,
                    letterSpacing: 1.5,
                  ),
                ),
              ),
              CompteARebours(
                restantes: _secondesRestantes,
                total: question.difficulte.secondes,
                actif: !_quiz.aRepondu,
              ),
            ],
          ),
```

> **Pourquoi `setState(() {})` vide ?** Parce que les deux changements — l'avancement du quiz et la remise à zéro du compteur — ont déjà eu lieu juste avant. `setState` ne fait rien d'autre que demander une reconstruction ; son argument sert uniquement à regrouper les modifications au même endroit. Une seule reconstruction pour deux changements vaut mieux que deux reconstructions.

**État exécutable.** Le cercle se vide seconde par seconde et devient rouge sous 5 secondes. Il se fige dès que l'on répond.

---

## 59.16 — L'expiration du temps

Le travail est déjà fait : `_repondre(null)` est appelé par le minuteur quand il ne reste plus qu'une seconde. Reste à comprendre pourquoi ce `null` est la bonne conception.

```text
Trois issues possibles pour une question, et une seule donnée :

  indexChoisi = 1      le joueur a choisi B      -> juste ou faux
  indexChoisi = 0      le joueur a choisi A      -> juste ou faux
  indexChoisi = null   personne n'a rien choisi  -> temps écoulé
```

Un booléen `tempsEcoule` en plus de `indexChoisi` créerait des états impossibles à représenter : que signifierait `indexChoisi = 2` **et** `tempsEcoule = true` ? Avec un seul champ nullable, cette contradiction ne peut pas être écrite. C'est le principe du chapitre 12 : **rendre les états invalides impossibles à construire**.

Les conséquences sont automatiques :

| Endroit | Effet du `null` |
| --- | --- |
| `ReponseDonnee.juste` | `estJuste(null)` renvoie `false` : la réponse est comptée fausse. |
| `ReponseDonnee.points` | 0 point. |
| `etatDuBouton` | Aucun bouton n'est `choisi` ; seule la bonne réponse est révélée. |
| `CarteExplication` | En-tête « Temps écoulé » et rappel de la bonne réponse. |
| `Quiz.tempsEcoules` | Compteur distinct des mauvaises réponses, pour le bilan. |

Une dernière précaution : quand le temps expire, le joueur ne regardait peut-être pas l'écran. Le passage d'un état à l'autre doit donc rester visible même après coup, ce qui est le cas puisque rien ne s'enchaîne tout seul (59.12).

**État exécutable.** Laissez le temps s'écouler sans rien toucher :

```text
│  ┌──────────────────────────────────────────┐  │
│  │ (B)  Un Future                       ✓   │  │  vert révélé
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │ (t) Temps écoulé                          │  │
│  │    La bonne réponse était : Un Future.   │  │
│  └──────────────────────────────────────────┘  │
```

---

## 59.17 — La transition animée entre deux questions

Sans animation, la question suivante remplace la précédente instantanément. L'œil ne perçoit alors aucun changement de contenu — seulement un scintillement — et le joueur peut croire que son appui n'a rien fait.

`AnimatedSwitcher` résout cela en une ligne de principe : quand son enfant change **de clé**, il fait sortir l'ancien et entrer le nouveau.

```dart
AnimatedSwitcher({
  Key? key,
  Widget? child,
  required Duration duration,
  Duration? reverseDuration,
  Curve switchInCurve = Curves.linear,
  Curve switchOutCurve = Curves.linear,
  AnimatedSwitcherTransitionBuilder transitionBuilder =
      AnimatedSwitcher.defaultTransitionBuilder,
  AnimatedSwitcherLayoutBuilder layoutBuilder =
      AnimatedSwitcher.defaultLayoutBuilder,
})
```

**Le point qui piège tout le monde :** si l'ancien et le nouvel enfant ont le même type et la même clé, Flutter considère que c'est le *même* widget avec de nouveaux paramètres, et **n'anime rien**. C'est exactement notre cas : deux questions successives produisent deux `Column` identiques en type. Il faut donc donner une clé qui change à chaque question.

```dart
key: ValueKey<int>(_quiz.index)   // 0, 1, 2... une clé par question
```

### Le corps animé

Dans `build`, enveloppez le contenu variable — bandeau, énoncé, propositions, explication — dans un `AnimatedSwitcher`. La barre de progression et la barre d'application, elles, ne doivent **pas** être animées : elles sont communes à toutes les questions.

```dart
      body: AnimatedSwitcher(
        duration: const Duration(milliseconds: 350),
        switchInCurve: Curves.easeOut,
        switchOutCurve: Curves.easeIn,
        // Fondu + glissement : la nouvelle question arrive par la
        // droite, comme une page que l'on tourne.
        transitionBuilder: (Widget enfant, Animation<double> animation) {
          return FadeTransition(
            opacity: animation,
            child: SlideTransition(
              position: Tween<Offset>(
                begin: const Offset(0.15, 0),
                end: Offset.zero,
              ).animate(animation),
              child: enfant,
            ),
          );
        },
        child: ListView(
          // LA clé indispensable : elle change à chaque question.
          key: ValueKey<int>(_quiz.index),
          padding: const EdgeInsets.all(16),
          children: <Widget>[
            // ... tout le contenu des sections précédentes ...
          ],
        ),
      ),
```

> **Pourquoi la clé porte sur l'index et non sur l'identifiant de la question ?** Les deux fonctionnent. L'index a un avantage : après un `recommencer()`, il repart à 0 et l'animation rejoue. Avec l'identifiant, rejouer la même question dans le même ordre ne produirait aucune transition.

> **Attention au coût.** Pendant la transition, les deux `ListView` existent simultanément. C'est sans conséquence ici — une dizaine de widgets — mais évitez d'envelopper une liste de mille éléments dans un `AnimatedSwitcher`.

**État exécutable.** QUESTION SUIVANTE fait glisser la nouvelle question depuis la droite en fondu, pendant 350 ms. Le jeu est complet du départ à la dernière question.

---

## 59.18 — L'écran de résultat et le récapitulatif

Fin de partie. L'écran de résultat doit répondre à quatre questions dans cet ordre : combien ai-je marqué, est-ce bien, ai-je battu mon record, et où me suis-je trompé.

### La ligne de récapitulatif

**`lib/widgets/ligne_recapitulatif.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/quiz.dart';
import '../utilitaires/couleurs_reponse.dart';

/// Une ligne du bilan final : une question, ce qui a été répondu.
class LigneRecapitulatif extends StatelessWidget {
  const LigneRecapitulatif({
    super.key,
    required this.numero,
    required this.reponse,
  });

  final int numero;
  final ReponseDonnee reponse;

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    late final IconData icone;
    late final Color couleur;
    if (reponse.tempsEcoule) {
      icone = Icons.timer_off_outlined;
      couleur = theme.colorScheme.tertiary;
    } else if (reponse.juste) {
      icone = Icons.check_circle;
      couleur = CouleursReponse.juste(context);
    } else {
      icone = Icons.cancel;
      couleur = CouleursReponse.faux(context);
    }

    return ListTile(
      leading: Icon(icone, color: couleur),
      title: Text(
        '$numero. ${reponse.question.enonce}',
        style: theme.textTheme.bodyMedium,
      ),
      subtitle: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          if (reponse.tempsEcoule)
            const Text('Temps écoulé')
          else
            Text('Votre réponse : ${reponse.texteChoisi}'),
          // On ne rappelle la bonne réponse que si elle n'a pas été
          // trouvée : la répéter quand c'est juste est du bruit.
          if (!reponse.juste)
            Text(
              'Bonne réponse : ${reponse.question.bonneReponse}',
              style: TextStyle(color: CouleursReponse.juste(context)),
            ),
        ],
      ),
      isThreeLine: !reponse.juste,
    );
  }
}
```

### L'écran

**`lib/ecrans/ecran_resultat.dart`**

```dart
import 'package:flutter/material.dart';

import '../logique/quiz.dart';
import '../widgets/ligne_recapitulatif.dart';

/// Écran de fin de partie.
class EcranResultat extends StatelessWidget {
  const EcranResultat({
    super.key,
    required this.quiz,
    required this.surRejouer,
  });

  final Quiz quiz;

  /// Relance une partie avec les mêmes réglages.
  final VoidCallback surRejouer;

  /// Message adapté au pourcentage.
  ///
  /// Les seuils sont dans une seule fonction : les changer ne demande
  /// pas de fouiller le `build`.
  static String messagePour(double pourcentage) {
    if (pourcentage >= 0.9) {
      return 'Parfait ou presque. Il est temps de passer au chapitre suivant.';
    }
    if (pourcentage >= 0.7) {
      return 'Très bien. Vous maîtrisez l\'essentiel.';
    }
    if (pourcentage >= 0.5) {
      return 'Correct. Une relecture des explications vous fera progresser.';
    }
    return 'Le sujet mérite une révision. Rejouez après relecture.';
  }

  static IconData _icone(double pourcentage) {
    if (pourcentage >= 0.9) {
      return Icons.emoji_events;
    }
    if (pourcentage >= 0.5) {
      return Icons.star;
    }
    return Icons.school_outlined;
  }

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final int pourcent = (quiz.pourcentage * 100).round();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Résultat'),
        // On retire la flèche de retour : revenir sur la dernière
        // question n'aurait aucun sens, la partie est finie.
        automaticallyImplyLeading: false,
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          const SizedBox(height: 16),
          Icon(_icone(quiz.pourcentage),
              size: 72, color: theme.colorScheme.primary),
          const SizedBox(height: 8),
          Text(
            '${quiz.score} / ${quiz.scoreMaximum} pts',
            textAlign: TextAlign.center,
            style: theme.textTheme.headlineMedium
                ?.copyWith(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 4),
          Text(
            '${quiz.bonnesReponses} bonne(s) réponse(s) sur ${quiz.total}'
            '${quiz.tempsEcoules > 0 ? ' · ${quiz.tempsEcoules} hors délai' : ''}',
            textAlign: TextAlign.center,
            style: theme.textTheme.bodyMedium,
          ),
          const SizedBox(height: 16),
          LinearProgressIndicator(
            value: quiz.pourcentage,
            minHeight: 10,
            borderRadius: BorderRadius.circular(5),
            semanticsLabel: 'Score',
            semanticsValue: '$pourcent %',
          ),
          const SizedBox(height: 8),
          Text('$pourcent %', textAlign: TextAlign.center),
          const SizedBox(height: 16),
          Text(
            messagePour(quiz.pourcentage),
            textAlign: TextAlign.center,
            style: theme.textTheme.titleMedium,
          ),
          const SizedBox(height: 24),

          Text('Récapitulatif', style: theme.textTheme.titleMedium),
          const SizedBox(height: 8),
          Card(
            margin: EdgeInsets.zero,
            child: Column(
              children: <Widget>[
                for (int i = 0; i < quiz.reponses.length; i++) ...<Widget>[
                  if (i > 0) const Divider(height: 1),
                  LigneRecapitulatif(
                    numero: i + 1,
                    reponse: quiz.reponses[i],
                  ),
                ],
              ],
            ),
          ),
          const SizedBox(height: 24),

          Row(
            children: <Widget>[
              Expanded(
                child: FilledButton.icon(
                  onPressed: surRejouer,
                  icon: const Icon(Icons.replay),
                  label: const Text('REJOUER'),
                ),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: OutlinedButton.icon(
                  // popUntil ramène jusqu'à la PREMIÈRE route de la
                  // pile, c'est-à-dire l'accueil, quel que soit le
                  // nombre d'écrans empilés entre-temps.
                  onPressed: () => Navigator.of(context)
                      .popUntil((Route<dynamic> r) => r.isFirst),
                  icon: const Icon(Icons.home_outlined),
                  label: const Text('ACCUEIL'),
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

> **Pourquoi une `Column` de `LigneRecapitulatif` et non une `ListView` imbriquée ?** Parce qu'une `ListView` dans une `ListView` demande `shrinkWrap: true` et un `physics` désactivé, ce qui construit de toute façon tous les enfants. Avec dix à vingt lignes, la `Column` est plus simple et aussi rapide. Au-delà de cent, on passerait à `CustomScrollView` et `SliverList` (chapitre 48).

### Y aller depuis l'écran de question

Dans `_suivante`, remplacez le `pop` provisoire :

```dart
  void _suivante() {
    if (_quiz.estDerniere) {
      _arreterMinuteur();
      // pushReplacement : l'écran de question DISPARAÎT de la pile.
      // Le retour arrière depuis le résultat ramène ainsi à l'accueil
      // et non à la dernière question déjà répondue.
      Navigator.of(context).pushReplacement(
        MaterialPageRoute<void>(
          builder: (BuildContext _) => EcranResultat(
            quiz: _quiz,
            surRejouer: () {
              _quiz.recommencer();
              Navigator.of(context).pushReplacement(
                MaterialPageRoute<void>(
                  builder: (BuildContext _) => EcranQuestion(quiz: _quiz),
                ),
              );
            },
          ),
        ),
      );
      return;
    }
    _quiz.passerALaSuivante();
    _demarrerMinuteur();
    setState(() {});
  }
```

avec `import 'ecran_resultat.dart';`.

**État exécutable.** La partie va jusqu'au bout, affiche le score, le message, le bilan ligne par ligne, et propose de rejouer.

---
## 59.19 — Le meilleur score persistant

Un score qui disparaît à la fermeture de l'application ne motive personne. On mémorise donc le meilleur score, **par catégorie** : le record obtenu en Dart n'a rien à voir avec celui obtenu toutes catégories confondues.

`shared_preferences` (chapitre 54) est exactement l'outil qu'il faut : quelques entiers, une clé par catégorie, aucune structure.

```text
clé                          valeur
───────────────────────────  ──────
meilleur_score_toutes        14
meilleur_score_dart          9
meilleur_score_flutter       11
```

**`lib/donnees/depot_scores.dart`**

```dart
import 'package:shared_preferences/shared_preferences.dart';

/// Ce que l'application attend d'un stockage de scores.
abstract class DepotScores {
  /// Meilleur score enregistré pour cette clé, 0 si aucun.
  Future<int> meilleurScore(String cle);

  /// Enregistre [score] s'il bat le record.
  ///
  /// Renvoie `true` si c'est un nouveau record. C'est le dépôt qui
  /// compare, pas l'interface : la règle « on ne remplace que si
  /// c'est mieux » appartient au stockage.
  Future<bool> enregistrerSiMeilleur(String cle, int score);
}

/// Clé de stockage d'une sélection de catégorie.
///
/// `null` (toutes catégories) devient `toutes`. Cette fonction est le
/// seul endroit qui connaît cette convention.
String cleDeCategorie(String? idCategorie) => idCategorie ?? 'toutes';

/// Implémentation réelle, sur les préférences du système.
class DepotScoresPrefs implements DepotScores {
  const DepotScoresPrefs();

  /// Préfixe commun. Les préférences sont partagées par toute
  /// l'application : préfixer évite de heurter une autre clé.
  static const String _prefixe = 'meilleur_score_';

  @override
  Future<int> meilleurScore(String cle) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    // getInt renvoie null si la clé n'existe pas : première partie.
    return prefs.getInt('$_prefixe$cle') ?? 0;
  }

  @override
  Future<bool> enregistrerSiMeilleur(String cle, int score) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    final int ancien = prefs.getInt('$_prefixe$cle') ?? 0;
    if (score <= ancien) {
      return false;
    }
    await prefs.setInt('$_prefixe$cle', score);
    return true;
  }
}

/// Implémentation de test : une simple Map en mémoire.
class DepotScoresMemoire implements DepotScores {
  DepotScoresMemoire([Map<String, int>? initial])
      : _scores = <String, int>{...?initial};

  final Map<String, int> _scores;

  @override
  Future<int> meilleurScore(String cle) async => _scores[cle] ?? 0;

  @override
  Future<bool> enregistrerSiMeilleur(String cle, int score) async {
    if (score <= (_scores[cle] ?? 0)) {
      return false;
    }
    _scores[cle] = score;
    return true;
  }
}
```

> **Pourquoi `enregistrerSiMeilleur` renvoie-t-il un booléen ?** Parce que l'écran de résultat doit afficher « NOUVEAU RECORD ». Sans ce retour, il faudrait relire l'ancien score avant d'écrire, puis comparer dans l'interface : deux appels au lieu d'un, et la règle de comparaison dupliquée. Une méthode qui écrit et rend compte de ce qu'elle a fait évite ce doublon.

> **Pourquoi tout est-il `Future` ?** `SharedPreferences.getInstance()` est asynchrone (chapitre 54), donc tout ce qui en dépend l'est aussi. Le `DepotScoresMemoire` n'a besoin d'aucune attente, mais il respecte la même signature : sans cela, on ne pourrait pas substituer l'un à l'autre.

**État exécutable.** Le fichier compile. Le branchement complet arrive à la section suivante, en même temps que `provider`.

---

## 59.20 — Centraliser l'état avec `ChangeNotifier` et `provider`

### Le problème

Regardez ce que le code doit faire aujourd'hui pour rejouer une partie : l'écran de résultat appelle un rappel `surRejouer` fabriqué par l'écran de question, qui empile un nouvel écran de question, en réutilisant l'objet `Quiz` qu'il avait reçu de l'accueil. La partie vit dans un `widget.quiz` transporté d'écran en écran.

Trois choses deviennent alors impossibles ou pénibles :

```text
1. Le minuteur vit dans _EcranQuestionState. Il meurt avec l'écran.
   Impossible de mettre le jeu en pause depuis un autre écran.

2. L'écran de résultat ne connaît pas le dépôt de scores.
   Il faudrait le lui passer en paramètre, depuis l'écran de question,
   qui l'aurait reçu de l'accueil, qui l'aurait reçu de main.

3. L'accueil ne peut pas savoir que le record vient de changer.
   Il a été construit avant.
```

C'est le symptôme décrit au chapitre 52 : l'état est trop bas dans l'arbre. On le remonte dans un `ChangeNotifier` unique, exposé au-dessus de `MaterialApp`.

### Le `ChangeNotifier`

**`lib/etat/etat_quiz.dart`**

```dart
import 'dart:async';

import 'package:flutter/foundation.dart';

import '../donnees/depot_questions.dart';
import '../donnees/depot_scores.dart';
import '../logique/banque_questions.dart';
import '../logique/melange.dart';
import '../logique/quiz.dart';
import '../modeles/question.dart';

/// Les grandes phases de l'application.
enum PhaseQuiz { chargement, erreur, accueil, question, resultat }

/// L'état complet de l'application.
///
/// Il détient la banque, la partie en cours, le minuteur et les
/// records. Il n'importe QUE `foundation` : pas de `material`, pas de
/// `BuildContext`. Un état qui manipule des widgets est un état que
/// l'on ne peut plus tester.
class EtatQuiz extends ChangeNotifier {
  EtatQuiz({
    required DepotQuestions depotQuestions,
    required DepotScores depotScores,
  })  : _depotQuestions = depotQuestions,
        _depotScores = depotScores;

  final DepotQuestions _depotQuestions;
  final DepotScores _depotScores;

  PhaseQuiz _phase = PhaseQuiz.chargement;
  String _messageErreur = '';
  BanqueQuestions _banque = BanqueQuestions.vide;

  String? _categorie; // null = toutes
  int _nombreDemande = 10;

  Quiz? _quiz;
  Timer? _minuteur;
  int _secondesRestantes = 0;

  int _meilleurScore = 0;
  bool _nouveauRecord = false;

  // ---------------------------------------------------------------
  // Lectures. Aucune n'expose de champ modifiable.
  // ---------------------------------------------------------------

  PhaseQuiz get phase => _phase;
  String get messageErreur => _messageErreur;
  BanqueQuestions get banque => _banque;
  String? get categorie => _categorie;
  int get nombreDemande => _nombreDemande;
  Quiz get quiz => _quiz!;
  bool get partieEnCours => _quiz != null;
  int get secondesRestantes => _secondesRestantes;
  int get meilleurScore => _meilleurScore;
  bool get nouveauRecord => _nouveauRecord;

  /// Nombre de questions disponibles dans la sélection courante.
  int get disponibles => _banque.compte(_categorie);

  // ---------------------------------------------------------------
  // Chargement
  // ---------------------------------------------------------------

  Future<void> initialiser() async {
    _phase = PhaseQuiz.chargement;
    notifyListeners();
    try {
      _banque = await _depotQuestions.charger();
      _meilleurScore =
          await _depotScores.meilleurScore(cleDeCategorie(_categorie));
      _phase = PhaseQuiz.accueil;
    } on Object catch (e) {
      // On attrape tout : FormatException du JSON, mais aussi
      // l'erreur de chargement d'asset, qui n'est pas du même type.
      _messageErreur = '$e';
      _phase = PhaseQuiz.erreur;
    }
    notifyListeners();
  }

  // ---------------------------------------------------------------
  // Réglages de l'accueil
  // ---------------------------------------------------------------

  Future<void> choisirCategorie(String? id) async {
    _categorie = id;
    // Le record affiché doit suivre la catégorie choisie.
    _meilleurScore = await _depotScores.meilleurScore(cleDeCategorie(id));
    notifyListeners();
  }

  void choisirNombre(int n) {
    _nombreDemande = n;
    notifyListeners();
  }

  // ---------------------------------------------------------------
  // Déroulement de la partie
  // ---------------------------------------------------------------

  void demarrer() {
    final List<Question> questions = preparerPartie(
      _banque.pourCategorie(_categorie),
      nombre: _nombreDemande,
    );
    if (questions.isEmpty) {
      return;
    }
    _quiz = Quiz(questions: questions);
    _nouveauRecord = false;
    _phase = PhaseQuiz.question;
    _demarrerMinuteur();
    notifyListeners();
  }

  void repondre(int? index) {
    if (_quiz == null || _quiz!.aRepondu) {
      return;
    }
    _arreterMinuteur();
    _quiz!.repondre(
      index,
      secondesRestantes: index == null ? 0 : _secondesRestantes,
    );
    if (index == null) {
      _secondesRestantes = 0;
    }
    notifyListeners();
  }

  Future<void> suivante() async {
    if (_quiz == null || !_quiz!.aRepondu) {
      return;
    }
    if (_quiz!.estDerniere) {
      await _terminer();
      return;
    }
    _quiz!.passerALaSuivante();
    _demarrerMinuteur();
    notifyListeners();
  }

  Future<void> _terminer() async {
    _arreterMinuteur();
    _phase = PhaseQuiz.resultat;
    // On affiche le résultat AVANT d'attendre l'écriture disque :
    // l'utilisateur ne doit pas patienter pour voir son score.
    notifyListeners();

    final String cle = cleDeCategorie(_categorie);
    _nouveauRecord = await _depotScores.enregistrerSiMeilleur(cle, _quiz!.score);
    _meilleurScore = await _depotScores.meilleurScore(cle);
    notifyListeners();
  }

  void rejouer() {
    demarrer();
  }

  void retourAccueil() {
    _arreterMinuteur();
    _quiz = null;
    _phase = PhaseQuiz.accueil;
    notifyListeners();
  }

  // ---------------------------------------------------------------
  // Minuteur
  // ---------------------------------------------------------------

  void _demarrerMinuteur() {
    _minuteur?.cancel();
    _secondesRestantes = _quiz!.questionCourante.difficulte.secondes;
    _minuteur = Timer.periodic(const Duration(seconds: 1), (Timer t) {
      if (_secondesRestantes <= 1) {
        repondre(null);
      } else {
        _secondesRestantes--;
        notifyListeners();
      }
    });
  }

  void _arreterMinuteur() {
    _minuteur?.cancel();
    _minuteur = null;
  }

  /// Appelé par `ChangeNotifierProvider` quand l'état sort de l'arbre.
  ///
  /// Sans ce `cancel`, le minuteur continuerait d'appeler
  /// `notifyListeners()` sur un objet détruit : « A ChangeNotifier was
  /// used after being disposed ».
  @override
  void dispose() {
    _minuteur?.cancel();
    super.dispose();
  }
}
```

### Le point d'entrée

**`lib/main.dart`** (version définitive, hors thème sombre du 59.21)

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'donnees/depot_questions.dart';
import 'donnees/depot_scores.dart';
import 'ecrans/ecran_accueil.dart';
import 'ecrans/ecran_erreur.dart';
import 'ecrans/ecran_question.dart';
import 'ecrans/ecran_resultat.dart';
import 'etat/etat_quiz.dart';

void main() {
  runApp(const ApplicationQuiz());
}

class ApplicationQuiz extends StatelessWidget {
  const ApplicationQuiz({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<EtatQuiz>(
      // `create` construit l'état une seule fois, puis `initialiser`
      // lance le chargement. On ne peut pas mettre `await` ici :
      // `create` est synchrone. D'où un état `chargement` explicite.
      create: (BuildContext _) => EtatQuiz(
        depotQuestions: const DepotQuestionsAssets(),
        depotScores: const DepotScoresPrefs(),
      )..initialiser(),
      child: MaterialApp(
        title: 'Quiz',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        ),
        home: const RouteurQuiz(),
      ),
    );
  }
}

/// Choisit l'écran à afficher selon la phase.
///
/// Toute la navigation du projet tient dans ce `switch`. Il n'y a plus
/// de `Navigator.push` : changer de phase suffit. L'avantage est
/// immédiat — impossible d'empiler deux écrans de question, impossible
/// de revenir en arrière sur une question déjà répondue.
class RouteurQuiz extends StatelessWidget {
  const RouteurQuiz({super.key});

  @override
  Widget build(BuildContext context) {
    final PhaseQuiz phase = context.select<EtatQuiz, PhaseQuiz>(
      (EtatQuiz e) => e.phase,
    );

    switch (phase) {
      case PhaseQuiz.chargement:
        return const Scaffold(
          body: Center(child: CircularProgressIndicator()),
        );
      case PhaseQuiz.erreur:
        return const EcranErreur();
      case PhaseQuiz.accueil:
        return const EcranAccueil();
      case PhaseQuiz.question:
        return const EcranQuestion();
      case PhaseQuiz.resultat:
        return const EcranResultat();
    }
  }
}
```

> **`context.select`** ne redéclenche la reconstruction que si la valeur extraite change. Ici, `RouteurQuiz` ne se reconstruit donc pas à chaque battement de seconde du minuteur — seulement aux cinq changements de phase. Avec `context.watch`, il se reconstruirait vingt fois par question, pour rien.

### L'écran de question réécrit

Voici le fichier complet, avec tout ce que les sections 59.11 à 59.17 ont ajouté. C'est la version définitive.

**`lib/ecrans/ecran_question.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../etat/etat_quiz.dart';
import '../logique/quiz.dart';
import '../modeles/question.dart';
import '../widgets/barre_progression.dart';
import '../widgets/bouton_reponse.dart';
import '../widgets/carte_explication.dart';
import '../widgets/compte_a_rebours.dart';

/// L'écran de jeu.
///
/// Il est redevenu `StatelessWidget` : il ne détient plus rien. Le
/// minuteur, la partie et le score vivent dans `EtatQuiz`.
class EcranQuestion extends StatelessWidget {
  const EcranQuestion({super.key});

  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);

    // `watch` : cet écran doit se reconstruire à chaque seconde et à
    // chaque réponse. C'est ici que `watch` est justifié.
    final EtatQuiz etat = context.watch<EtatQuiz>();
    final Quiz quiz = etat.quiz;
    final Question question = quiz.questionCourante;
    final ReponseDonnee? reponse = quiz.reponseCourante;

    return Scaffold(
      appBar: AppBar(
        title: Text('Question ${quiz.numero} / ${quiz.total}'),
        leading: IconButton(
          icon: const Icon(Icons.close),
          tooltip: 'Abandonner la partie',
          // `read` : nous sommes dans un rappel, pas dans le build.
          onPressed: () => context.read<EtatQuiz>().retourAccueil(),
        ),
        actions: <Widget>[
          Center(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Text('${quiz.score} pts',
                  style: theme.textTheme.titleMedium),
            ),
          ),
        ],
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(8),
          child: BarreProgression(valeur: quiz.progression),
        ),
      ),
      body: AnimatedSwitcher(
        duration: const Duration(milliseconds: 350),
        switchInCurve: Curves.easeOut,
        switchOutCurve: Curves.easeIn,
        transitionBuilder: (Widget enfant, Animation<double> animation) {
          return FadeTransition(
            opacity: animation,
            child: SlideTransition(
              position: Tween<Offset>(
                begin: const Offset(0.15, 0),
                end: Offset.zero,
              ).animate(animation),
              child: enfant,
            ),
          );
        },
        child: ListView(
          key: ValueKey<int>(quiz.index),
          padding: const EdgeInsets.all(16),
          children: <Widget>[
            Row(
              children: <Widget>[
                Expanded(
                  child: Text(
                    '${etat.banque.nomDe(question.categorie).toUpperCase()}'
                    ' · ${question.difficulte.libelle.toUpperCase()}',
                    style: theme.textTheme.labelMedium?.copyWith(
                      color: theme.colorScheme.primary,
                      letterSpacing: 1.5,
                    ),
                  ),
                ),
                CompteARebours(
                  restantes: etat.secondesRestantes,
                  total: question.difficulte.secondes,
                  actif: !quiz.aRepondu,
                ),
              ],
            ),
            const SizedBox(height: 12),
            Card(
              margin: EdgeInsets.zero,
              child: Padding(
                padding: const EdgeInsets.all(20),
                child: Text(question.enonce,
                    style: theme.textTheme.titleLarge),
              ),
            ),
            const SizedBox(height: 24),
            for (int i = 0; i < question.reponses.length; i++)
              BoutonReponse(
                index: i,
                texte: question.reponses[i],
                etat: etatDuBouton(
                  indexBouton: i,
                  indexBonne: question.indexBonneReponse,
                  indexChoisi: reponse?.indexChoisi,
                  repondu: quiz.aRepondu,
                ),
                surAppui: quiz.aRepondu
                    ? null
                    : () => context.read<EtatQuiz>().repondre(i),
              ),
            if (reponse != null) ...<Widget>[
              const SizedBox(height: 8),
              CarteExplication(reponse: reponse),
              const SizedBox(height: 16),
              FilledButton.icon(
                onPressed: () => context.read<EtatQuiz>().suivante(),
                icon: const Icon(Icons.arrow_forward),
                label: Padding(
                  padding: const EdgeInsets.symmetric(vertical: 12),
                  child: Text(quiz.estDerniere
                      ? 'VOIR LE RÉSULTAT'
                      : 'QUESTION SUIVANTE'),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

### Les deux autres écrans

Les modifications sont mécaniques ; elles suivent toutes la même règle.

**`ecran_accueil.dart` :**

| Avant | Après |
| --- | --- |
| `class EcranAccueil extends StatefulWidget` | `class EcranAccueil extends StatelessWidget` |
| `FutureBuilder` + `initState` | plus rien : la banque est déjà chargée |
| `widget.depot`, `_categorieChoisie`, `_nombreChoisi` | `context.watch<EtatQuiz>()` |
| `setState(() => _categorieChoisie = id)` | `context.read<EtatQuiz>().choisirCategorie(id)` |
| `surDemarrer: () { ...push... }` | `context.read<EtatQuiz>().demarrer()` |

Ajoutez-y l'affichage du record, désormais disponible sans effort :

```dart
        if (etat.meilleurScore > 0)
          Text(
            'Meilleur score : ${etat.meilleurScore} pts',
            textAlign: TextAlign.center,
            style: theme.textTheme.titleSmall,
          ),
```

**`ecran_resultat.dart` :** supprimez les paramètres `quiz` et `surRejouer` du constructeur, lisez `final EtatQuiz etat = context.watch<EtatQuiz>();` puis `final Quiz quiz = etat.quiz;`, remplacez les deux boutons par `context.read<EtatQuiz>().rejouer()` et `context.read<EtatQuiz>().retourAccueil()`, et ajoutez le bandeau de record :

```dart
          if (etat.nouveauRecord)
            Card(
              color: theme.colorScheme.tertiaryContainer,
              child: ListTile(
                leading: const Icon(Icons.emoji_events),
                title: const Text('NOUVEAU RECORD'),
                subtitle: Text('Meilleur score : ${etat.meilleurScore} pts'),
              ),
            ),
```

Enfin, l'écran d'erreur du 59.10 devient un fichier à part, `lib/ecrans/ecran_erreur.dart`, qui lit `context.watch<EtatQuiz>().messageErreur`.

### `watch` ou `read` ?

C'est l'erreur numéro un avec `provider`, et elle ne produit aucun message : l'écran se fige simplement.

```text
Dans build()            -> context.watch<T>()   ou context.select
Dans onPressed, onTap,
  initState, un Timer   -> context.read<T>()

watch dans un rappel  : abonnement inutile, avertissement possible.
read dans un build    : l'écran ne se met plus jamais à jour.
```

**État exécutable.** L'application se comporte exactement comme avant — mais la pile de navigation a disparu, le minuteur survit à toute reconstruction, le record s'affiche sur l'accueil et se met à jour tout seul après une partie. Tuez l'application, relancez-la : le record est toujours là.

---
## 59.21 — L'accessibilité et le mode sombre

### Le mode sombre

Deux thèmes construits sur la même couleur d'amorce, et un `ChangeNotifier` qui mémorise le choix.

**`lib/utilitaires/themes.dart`**

```dart
import 'package:flutter/material.dart';

/// Couleur d'amorce commune aux deux thèmes.
const Color _amorce = Colors.deepPurple;

ThemeData get themeClair => ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(seedColor: _amorce),
    );

ThemeData get themeSombre => ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: _amorce,
        brightness: Brightness.dark,
      ),
    );
```

**`lib/etat/etat_theme.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

/// Mémorise et persiste le choix clair / sombre.
class EtatTheme extends ChangeNotifier {
  static const String _cle = 'mode_sombre';

  ThemeMode _mode = ThemeMode.system;
  ThemeMode get mode => _mode;

  /// Lit la préférence enregistrée. À appeler au démarrage.
  Future<void> charger() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    final bool? sombre = prefs.getBool(_cle);
    // Aucune préférence enregistrée : on suit le réglage du système.
    _mode = sombre == null
        ? ThemeMode.system
        : (sombre ? ThemeMode.dark : ThemeMode.light);
    notifyListeners();
  }

  Future<void> basculer(bool sombre) async {
    _mode = sombre ? ThemeMode.dark : ThemeMode.light;
    notifyListeners(); // l'interface change tout de suite
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setBool(_cle, sombre); // l'écriture suit
  }
}
```

Dans `main.dart`, empilez les deux fournisseurs avec `MultiProvider` et branchez les thèmes :

```dart
    return MultiProvider(
      providers: <SingleChildWidget>[
        ChangeNotifierProvider<EtatQuiz>(
          create: (BuildContext _) => EtatQuiz(
            depotQuestions: const DepotQuestionsAssets(),
            depotScores: const DepotScoresPrefs(),
          )..initialiser(),
        ),
        ChangeNotifierProvider<EtatTheme>(
          create: (BuildContext _) => EtatTheme()..charger(),
        ),
      ],
      child: Builder(
        builder: (BuildContext context) {
          return MaterialApp(
            title: 'Quiz',
            debugShowCheckedModeBanner: false,
            theme: themeClair,
            darkTheme: themeSombre,
            themeMode: context.watch<EtatTheme>().mode,
            home: const RouteurQuiz(),
          );
        },
      ),
    );
```

L'interrupteur, à placer dans les `actions` de l'`AppBar` de l'accueil :

```dart
          IconButton(
            icon: Icon(sombre ? Icons.light_mode : Icons.dark_mode),
            tooltip: sombre ? 'Passer en clair' : 'Passer en sombre',
            onPressed: () => context.read<EtatTheme>().basculer(!sombre),
          ),
```

avec, en tête du `build` :

```dart
    final bool sombre = Theme.of(context).brightness == Brightness.dark;
```

> **Le `Builder` est indispensable.** Sans lui, le `context` passé à `MaterialApp` serait celui du widget qui *crée* les fournisseurs : `context.watch<EtatTheme>()` remonterait au-dessus d'eux et lèverait `ProviderNotFoundException`. Le `Builder` fabrique un contexte enfant, situé sous le `MultiProvider`.

### Six vérifications d'accessibilité

Elles ne coûtent presque rien si l'on y pense en écrivant, et sont pénibles à rattraper après coup.

| Point | Ce que nous avons fait |
| --- | --- |
| La couleur seule ne suffit pas | Chaque bouton corrigé porte aussi une icône `✓` ou `✕` (59.12). Un daltonien distingue les formes, pas les teintes. |
| Les widgets purement visuels doivent être annoncés | `CompteARebours` est enveloppé dans `Semantics(label:, value:)`, sinon le lecteur d'écran ne lit qu'un nombre nu. |
| Les barres de progression | `semanticsLabel` et `semanticsValue` sur les deux `LinearProgressIndicator`. |
| Les boutons sans texte | `tooltip` sur chaque `IconButton` : il sert d'infobulle **et** de description vocale. |
| Les zones d'appui | Les boutons de réponse font 56 px de haut avec leurs marges, au-delà des 48 px recommandés par Material. |
| Les couleurs en dur | Aucune, sauf le vert et le rouge sémantiques, déclinés en deux nuances selon la luminosité (59.12). |

Un dernier réglage à connaître : le minuteur. Un temps limité est un obstacle pour de nombreux utilisateurs. Le défi 6 propose de le rendre désactivable ; dans une application réelle, cette option n'est pas un luxe.

**État exécutable.** L'interrupteur bascule instantanément entre les deux thèmes, et le choix survit au redémarrage. Activez TalkBack ou VoiceOver : chaque élément est annoncé.

---

## 59.22 — Tester la logique de scoring

Rien de tout cela ne serait testable si `Quiz` et `preparerPartie` importaient Flutter. C'est le bénéfice de la découpe du 59.0.4, et il se mesure en secondes : les tests suivants s'exécutent sans émulateur.

**`test/quiz_test.dart`**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mon_quiz/logique/quiz.dart';
import 'package:mon_quiz/modeles/difficulte.dart';
import 'package:mon_quiz/modeles/question.dart';

/// Fabrique une question jetable.
///
/// La bonne réponse est toujours en position 0 : cela rend les tests
/// lisibles. `repondre(0)` signifie « juste », tout le reste « faux ».
Question q(String id, Difficulte d) => Question(
      id: id,
      categorie: 'test',
      enonce: 'Énoncé $id',
      reponses: const <String>['bonne', 'mauvaise', 'autre'],
      indexBonneReponse: 0,
      explication: 'Parce que.',
      difficulte: d,
    );

void main() {
  group('Quiz — état initial', () {
    test('démarre sur la première question, score nul', () {
      final Quiz quiz = Quiz(questions: <Question>[
        q('a', Difficulte.facile),
        q('b', Difficulte.moyenne),
      ]);

      expect(quiz.index, 0);
      expect(quiz.numero, 1);
      expect(quiz.total, 2);
      expect(quiz.score, 0);
      expect(quiz.aRepondu, isFalse);
      expect(quiz.termine, isFalse);
      expect(quiz.progression, 0);
      // 1 point (facile) + 2 points (moyenne)
      expect(quiz.scoreMaximum, 3);
    });

    test('refuse une liste vide', () {
      expect(
        () => Quiz(questions: <Question>[]),
        throwsA(isA<ArgumentError>()),
      );
    });
  });

  group('Quiz — scoring', () {
    test('une bonne réponse rapporte les points de sa difficulté', () {
      final Quiz quiz =
          Quiz(questions: <Question>[q('a', Difficulte.difficile)]);

      quiz.repondre(0);

      expect(quiz.score, 3);
      expect(quiz.bonnesReponses, 1);
      expect(quiz.pourcentage, 1.0);
      expect(quiz.termine, isTrue);
    });

    test('une mauvaise réponse ne rapporte rien', () {
      final Quiz quiz = Quiz(questions: <Question>[q('a', Difficulte.facile)]);

      quiz.repondre(2);

      expect(quiz.score, 0);
      expect(quiz.bonnesReponses, 0);
      expect(quiz.mauvaisesReponses, 1);
      expect(quiz.tempsEcoules, 0);
    });

    test('le temps écoulé compte comme faux, mais séparément', () {
      final Quiz quiz = Quiz(questions: <Question>[q('a', Difficulte.facile)]);

      quiz.repondre(null);

      expect(quiz.score, 0);
      expect(quiz.tempsEcoules, 1);
      expect(quiz.mauvaisesReponses, 0);
      expect(quiz.reponses.single.tempsEcoule, isTrue);
      expect(quiz.reponses.single.texteChoisi, isNull);
    });

    test('la seconde réponse à la même question est ignorée', () {
      final Quiz quiz = Quiz(questions: <Question>[q('a', Difficulte.facile)]);

      quiz.repondre(1); // faux
      quiz.repondre(0); // ignoré, sinon le score passerait à 1

      expect(quiz.score, 0);
      expect(quiz.reponses.length, 1);
    });
  });

  group('Quiz — progression', () {
    test('passerALaSuivante exige une réponse', () {
      final Quiz quiz = Quiz(questions: <Question>[
        q('a', Difficulte.facile),
        q('b', Difficulte.facile),
      ]);

      expect(quiz.passerALaSuivante(), isFalse);
      expect(quiz.index, 0);

      quiz.repondre(0);
      expect(quiz.passerALaSuivante(), isTrue);
      expect(quiz.index, 1);
      expect(quiz.estDerniere, isTrue);
      expect(quiz.passerALaSuivante(), isFalse); // plus rien après
    });

    test('la progression suit les réponses, pas l\'index', () {
      final Quiz quiz = Quiz(questions: <Question>[
        q('a', Difficulte.facile),
        q('b', Difficulte.facile),
        q('c', Difficulte.facile),
        q('d', Difficulte.facile),
      ]);

      expect(quiz.progression, 0);
      quiz.repondre(0);
      expect(quiz.progression, 0.25);
      quiz.passerALaSuivante();
      expect(quiz.progression, 0.25); // avancer n'est pas répondre
    });

    test('recommencer remet tout à zéro', () {
      final Quiz quiz = Quiz(questions: <Question>[
        q('a', Difficulte.facile),
        q('b', Difficulte.facile),
      ]);
      quiz.repondre(0);
      quiz.passerALaSuivante();
      quiz.repondre(0);

      quiz.recommencer();

      expect(quiz.index, 0);
      expect(quiz.score, 0);
      expect(quiz.reponses, isEmpty);
    });
  });

  group('Question', () {
    test('refuse un index de bonne réponse hors bornes', () {
      expect(
        () => Question(
          id: 'x',
          categorie: 'test',
          enonce: 'e',
          reponses: const <String>['a', 'b'],
          indexBonneReponse: 5,
          explication: '',
          difficulte: Difficulte.facile,
        ),
        throwsA(isA<FormatException>()),
      );
    });

    test('fromJson refuse un énoncé manquant', () {
      expect(
        () => Question.fromJson(<String, dynamic>{
          'id': 'x',
          'reponses': <String>['a', 'b'],
          'bonne': 0,
        }),
        throwsA(isA<FormatException>()),
      );
    });

    test('aller-retour JSON', () {
      final Question avant = q('a', Difficulte.difficile);
      final Question apres = Question.fromJson(avant.toJson());

      expect(apres.id, avant.id);
      expect(apres.reponses, avant.reponses);
      expect(apres.indexBonneReponse, avant.indexBonneReponse);
      expect(apres.difficulte, Difficulte.difficile);
    });
  });
}
```

**`test/melange_test.dart`**

```dart
import 'dart:math';

import 'package:flutter_test/flutter_test.dart';
import 'package:mon_quiz/logique/melange.dart';
import 'package:mon_quiz/modeles/difficulte.dart';
import 'package:mon_quiz/modeles/question.dart';

List<Question> banque(int combien) => List<Question>.generate(
      combien,
      (int i) => Question(
        id: 'q$i',
        categorie: 'test',
        enonce: 'Énoncé $i',
        reponses: <String>['bonne$i', 'a$i', 'b$i', 'c$i'],
        indexBonneReponse: 0,
        explication: '',
        difficulte: Difficulte.facile,
      ),
    );

void main() {
  test('preparerPartie tire le nombre demandé', () {
    final List<Question> partie =
        preparerPartie(banque(20), nombre: 5, alea: Random(1));
    expect(partie.length, 5);
  });

  test('ne tire jamais plus que ce qui existe', () {
    final List<Question> partie =
        preparerPartie(banque(3), nombre: 10, alea: Random(1));
    expect(partie.length, 3);
  });

  test('nombre <= 0 signifie toutes les questions', () {
    expect(preparerPartie(banque(7), nombre: 0, alea: Random(1)).length, 7);
  });

  test('une source vide donne une partie vide, sans exception', () {
    expect(preparerPartie(<Question>[], nombre: 5), isEmpty);
  });

  test('le mélange conserve la bonne réponse', () {
    // La propriété essentielle : après mélange, indexBonneReponse
    // désigne toujours le MÊME TEXTE qu'avant.
    for (final Question melangee
        in preparerPartie(banque(10), nombre: 10, alea: Random(7))) {
      final int numero = int.parse(melangee.id.substring(1));
      expect(melangee.bonneReponse, 'bonne$numero');
      expect(melangee.reponses.length, 4);
      expect(melangee.reponses.toSet().length, 4); // aucun doublon perdu
    }
  });

  test('la même graine donne le même tirage', () {
    final List<String> a = preparerPartie(banque(20), nombre: 5, alea: Random(3))
        .map((Question q) => q.id)
        .toList();
    final List<String> b = preparerPartie(banque(20), nombre: 5, alea: Random(3))
        .map((Question q) => q.id)
        .toList();
    expect(a, b);
  });

  test('la bonne réponse ne reste pas toujours en position 0', () {
    // Avec 30 questions mélangées, il serait improbable que les 30
    // bonnes réponses retombent en première position.
    final List<Question> partie =
        preparerPartie(banque(30), nombre: 30, alea: Random(11));
    final int enPremier = partie
        .where((Question q) => q.indexBonneReponse == 0)
        .length;
    expect(enPremier, lessThan(30));
  });
}
```

Lancez-les :

```text
flutter test
```

**Résultat :**

```text
00:02 +19: All tests passed!
```

> **Une remarque sur le dernier test.** Il vérifie une propriété statistique, pas une valeur exacte. Écrire `expect(partie[0].indexBonneReponse, 2)` fonctionnerait avec `Random(11)` mais casserait au premier changement d'implémentation du générateur. Testez ce que votre code **promet**, jamais ce qu'il produit par hasard.

**État exécutable.** L'application est terminée et couverte par 19 tests.

---

## 59.23 — Récapitulatif de l'arborescence finale

```text
mon_quiz/
├── pubspec.yaml                          59.1
├── assets/data/questions.json            59.5
├── lib/
│   ├── main.dart                         59.20, 59.21
│   ├── modeles/
│   │   ├── difficulte.dart               59.2
│   │   ├── categorie.dart                59.3
│   │   └── question.dart                 59.4, 59.9
│   ├── logique/
│   │   ├── banque_questions.dart         59.6
│   │   ├── quiz.dart                     59.8
│   │   └── melange.dart                  59.9
│   ├── donnees/
│   │   ├── depot_questions.dart          59.7
│   │   └── depot_scores.dart             59.19
│   ├── etat/
│   │   ├── etat_quiz.dart                59.20
│   │   └── etat_theme.dart               59.21
│   ├── ecrans/
│   │   ├── ecran_accueil.dart            59.10, 59.20
│   │   ├── ecran_question.dart           59.11 → 59.17, 59.20
│   │   ├── ecran_resultat.dart           59.18, 59.20
│   │   └── ecran_erreur.dart             59.20
│   ├── widgets/
│   │   ├── bouton_reponse.dart           59.12
│   │   ├── carte_explication.dart        59.13
│   │   ├── barre_progression.dart        59.14
│   │   ├── compte_a_rebours.dart         59.15
│   │   └── ligne_recapitulatif.dart      59.18
│   └── utilitaires/
│       ├── couleurs_reponse.dart         59.10, 59.12
│       └── themes.dart                   59.21
└── test/
    ├── quiz_test.dart                    59.22
    └── melange_test.dart                 59.22
```

Le sens des dépendances, qui ne doit jamais s'inverser :

```text
  ecrans/  widgets/          <- connaissent material
      │
      ▼
    etat/                    <- ne connaît que foundation
      │
      ├──────────────┐
      ▼              ▼
  logique/       donnees/    <- logique ne connaît RIEN de Flutter
      │              │          donnees connaît services (rootBundle)
      └──────┬───────┘
             ▼
         modeles/            <- ne connaît rien
```

---

## 59.24 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unable to load asset: "assets/data/questions.json"` | `assets:` mal indenté, ou slash final oublié. | Deux espaces sous `flutter:`, et `assets/data/`. |
| L'asset reste introuvable après correction du `pubspec.yaml` | Le rechargement à chaud ne relit pas `pubspec.yaml`. | Arrêter et relancer `flutter run`. |
| `FormatException: Unexpected character (at line ...)` | Virgule en trop dans le JSON. | Supprimer la virgule après le dernier élément. |
| L'écran de chargement clignote sans fin | `depot.charger()` appelé dans `build`. | Créer le `Future` une seule fois, dans `initState`. |
| `RangeError (index): Invalid value` à l'affichage | `bonne` hors des bornes de `reponses` dans le JSON. | Le contrôle du constructeur de `Question` le signale : lire le message. |
| Après mélange, la bonne réponse est fausse | `indexBonneReponse` conservé tel quel. | Recalculer par `melangees.indexOf(texteBonneReponse)`. |
| `setState() called after dispose()` | Un `Timer` survit à l'écran. | `_minuteur?.cancel()` dans `dispose`. |
| Le compte à rebours accélère | Un `Timer.periodic` créé à chaque question sans annuler le précédent. | `_minuteur?.cancel()` avant chaque création. |
| `A ChangeNotifier was used after being disposed` | Le minuteur de `EtatQuiz` notifie après destruction. | Redéfinir `dispose()` dans le `ChangeNotifier`. |
| `AnimatedSwitcher` n'anime rien | Les deux enfants ont la même clé. | `key: ValueKey<int>(quiz.index)` sur l'enfant. |
| L'écran ne se met plus à jour | `context.read` utilisé dans un `build`. | `context.watch` dans `build`, `read` dans les rappels. |
| `ProviderNotFoundException` | Le `context` utilisé est celui qui crée le fournisseur. | Interposer un `Builder`, ou lire depuis un widget enfant. |
| L'écran clignote à chaque seconde du minuteur | `context.watch` là où seule la phase compte. | `context.select<EtatQuiz, PhaseQuiz>(...)`. |
| Le score double sur un appui rapide | Deux appuis avant la reconstruction. | Le garde `if (aRepondu) return;` dans `Quiz.repondre`. |
| `NaN` dans le pourcentage | Division par un `scoreMaximum` nul. | Traiter explicitement le cas `scoreMaximum == 0`. |
| Le record ne survit pas au redémarrage | `setInt` non attendu, ou clé différente à la relecture. | `await` l'écriture et centraliser la clé dans `cleDeCategorie`. |
| Texte invisible en mode sombre | Couleur écrite en dur. | Passer par `Theme.of(context).colorScheme`, sauf pour le vert et le rouge sémantiques, déclinés par luminosité. |
| `withOpacity is deprecated` | API remplacée depuis Flutter 3.27. | `withValues(alpha: ...)`. |
| Les tests ne compilent pas | Import relatif de `lib/` depuis `test/`. | `import 'package:mon_quiz/...';`. |

---

## 59.25 — Ce que vous avez appris

| Notion | À retenir |
| --- | --- |
| Données hors du code | Les questions vivent en JSON dans les assets : les modifier ne demande aucune recompilation, et le même format servira pour une API. |
| `rootBundle.loadString` | Lecture asynchrone d'un asset texte ; le chemin est celui déclaré dans `pubspec.yaml`, à la lettre près. |
| `fromJson` défensif | Vérifier le type, nommer l'objet fautif dans le message, lever quand la donnée est inutilisable, retomber sur une valeur sûre quand elle est seulement cosmétique. |
| Invariants dans le constructeur | Un index de bonne réponse hors bornes est refusé à la construction, donc jamais affiché. |
| Ne jamais stocker ce qui se calcule | Le score est un `fold` sur les réponses, pas un champ à maintenir. |
| Nullable porteur de sens | `indexChoisi == null` signifie « temps écoulé » : un seul champ, aucun état contradictoire possible. |
| Moteur pur | `Quiz` n'importe pas Flutter : il se teste en millisecondes et se réutilise derrière n'importe quelle interface. |
| Règles dans le moteur | Le verrouillage après réponse est dans `Quiz.repondre`, pas dans le bouton. |
| Un `enum` pour les apparences | Cinq états nommés valent mieux que quatre `if` imbriqués dans un `build`. |
| Retour visuel non bloquant | On change l'état et on reconstruit ; on n'attend jamais avec `Future.delayed` avant d'enchaîner. |
| Couleur + forme | Vert et rouge sont toujours doublés d'une icône : la couleur seule n'est pas une information accessible. |
| `Timer.periodic` | Créer dans `initState`, annuler avant de recréer, annuler dans `dispose`. Ces trois lignes évitent trois bugs. |
| `TweenAnimationBuilder` | Anime une valeur qui change par à-coups, sans contrôleur à gérer. |
| `AnimatedSwitcher` | N'anime que si la clé de l'enfant change ; sans `Key`, rien ne se passe. |
| Injection de l'aléatoire | Passer un `Random` en paramètre rend testable une fonction qui mélange. |
| `take` plutôt que `sublist` | Demander 10 éléments à une liste qui en contient 6 doit en rendre 6, pas lever une exception. |
| Dépôt (`repository`) | Une interface, deux implémentations : la vraie et celle des tests. |
| Écrire et rendre compte | `enregistrerSiMeilleur` renvoie `true` en cas de record : une seule opération, une seule règle. |
| État centralisé | Minuteur compris. Le `ChangeNotifier` remplace la pile de navigation par un `switch` sur une phase. |
| `select` contre `watch` | `select` ne reconstruit que si la valeur extraite change : indispensable quand l'état change chaque seconde. |
| `Builder` sous `MultiProvider` | Sans lui, le contexte est situé au-dessus des fournisseurs. |
| Mise à jour optimiste | On notifie l'interface, puis on écrit sur le disque. |
| Tester des propriétés | On vérifie que le mélange conserve la bonne réponse, pas qu'il produit un ordre précis. |

---

## 59.26 — Extensions : dix défis

Chaque défi est réalisable avec ce que vous savez déjà. L'indication donne la direction, pas la solution.

### Défi 1 — Le bonus de rapidité (facile)

Répondre juste en moins de 5 secondes rapporte un point supplémentaire.

*Indication :* `ReponseDonnee.secondesRestantes` est déjà enregistré, et le minuteur le renseigne. Modifiez le getter `points` — et écrivez d'abord le test qui échoue.

### Défi 2 — Le compteur de séries (facile)

Affichez « 3 d'affilée » quand trois bonnes réponses s'enchaînent.

*Indication :* un getter `serieCourante` sur `Quiz` qui parcourt `reponses` à l'envers et s'arrête à la première réponse fausse.

### Défi 3 — Le filtre par difficulté (facile)

L'accueil propose de ne jouer que les questions difficiles.

*Indication :* un `where` sur la difficulté dans `BanqueQuestions.pourCategorie`, et une seconde rangée de `ChoiceChip` alimentée par `Difficulte.values`.

### Défi 4 — Le mode révision (moyen)

Un bouton de l'écran de résultat relance une partie composée uniquement des questions ratées.

*Indication :* `quiz.reponses.where((r) => !r.juste).map((r) => r.question).toList()` fournit exactement cette liste. Ajoutez `demarrerAvec(List<Question>)` à `EtatQuiz`.

### Défi 5 — L'historique des parties (moyen)

Les dix dernières parties sont conservées avec leur date, leur catégorie et leur score.

*Indication :* une classe `Partie` sérialisable, une `List<String>` JSON dans `shared_preferences` (chapitre 54), et un troisième écran alimenté par `ListView.builder`.

### Défi 6 — Le minuteur désactivable (moyen)

Une option de l'accueil supprime la limite de temps.

*Indication :* un booléen dans `EtatQuiz`, testé avant `_demarrerMinuteur`. `CompteARebours` doit alors afficher une icône d'infini plutôt qu'un cercle vide. C'est aussi une amélioration d'accessibilité.

### Défi 7 — Le partage du score (moyen)

Un bouton copie « J'ai obtenu 14/20 au quiz Flutter » dans le presse-papiers.

*Indication :* `Clipboard.setData(ClipboardData(text: ...))`, dans `package:flutter/services.dart`, suivi d'un `SnackBar` de confirmation.

### Défi 8 — Les questions à réponses multiples (difficile)

Certaines questions attendent deux bonnes réponses.

*Indication :* remplacez `indexBonneReponse` par un `Set<int>`, et `indexChoisi` par un `Set<int>` également. Le JSON passe de `"bonne": 2` à `"bonnes": [1, 3]`. Prévoyez la compatibilité avec les anciennes questions : `json['bonnes'] ?? [json['bonne']]`. C'est un changement de modèle, donc à faire test en premier.

### Défi 9 — Les questions distantes (difficile)

Charger la banque depuis une URL, avec repli sur le fichier local si le réseau échoue.

*Indication :* une troisième implémentation de `DepotQuestions` utilisant `http` (chapitre 53), enveloppée dans un `try`/`catch` qui délègue à `DepotQuestionsAssets`. Aucun autre fichier ne change : c'est le bénéfice de l'interface du 59.7.

### Défi 10 — L'animation de la barre de score (difficile)

À l'arrivée sur l'écran de résultat, le score monte de 0 à sa valeur finale en une seconde.

*Indication :* `TweenAnimationBuilder<double>` de 0 à `quiz.pourcentage`, dont le `builder` alimente à la fois le `LinearProgressIndicator` et le texte. Attention : sans clé stable, la reconstruction déclenchée par l'enregistrement du record relancerait l'animation depuis zéro.

---

## Et maintenant ?

Ce projet vous a fait franchir un cap : pour la première fois, l'état de votre application **avance dans le temps** tout seul. Un minuteur qui tourne, une phase qui change, une animation qui accompagne la transition — et tout cela piloté depuis un unique `ChangeNotifier` qui ne connaît ni les widgets ni la navigation. C'est exactement l'architecture d'une boucle de jeu, que vous retrouverez dans la PARTIE 2.

Vous avez aussi appris à sortir vos données du code. Le fichier `questions.json` que vous éditez sans recompiler est le même objet, du point de vue du programme, que la réponse d'une API distante. Le chapitre 61 ne fera que remplacer la source.

Le projet suivant change d'échelle : plusieurs écrans, une grille d'articles, un détail, et un panier partagé entre toutes les pages. Vous y pousserez `provider` un cran plus loin, avec deux états qui coexistent et se croisent.

Rendez-vous au chapitre 60 : [60-PARTIE-1C—PROJET-6-CATALOGUE-PRODUITS.md](./60-PARTIE-1C—PROJET-6-CATALOGUE-PRODUITS.md)
