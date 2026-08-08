# PARTIE 1A — DART
# CHAPITRE 01 — INTRODUCTION À DART

> **Niveau :** débutant
> **Durée estimée :** 1 h 30
> **Pré-requis :** aucun
> **Ce que vous saurez faire à la fin :** écrire, lire et exécuter un programme Dart minimal, et comprendre le rôle de Dart par rapport à Flutter.

---

## Présentation de la Partie 1A

Avant de développer des interfaces avec Flutter, il est indispensable de comprendre le langage utilisé par Flutter : **Dart**.

Cette première partie a pour objectif de construire progressivement les bases nécessaires pour être capable de :

- lire du code Dart ;
- écrire des programmes Dart ;
- manipuler des variables ;
- utiliser des conditions ;
- créer des boucles ;
- travailler avec des listes et des collections ;
- créer des fonctions ;
- comprendre la programmation orientée objet ;
- créer des classes et des objets ;
- utiliser l'héritage ;
- comprendre la null safety ;
- manipuler les exceptions ;
- comprendre la programmation asynchrone ;
- utiliser `Future` ;
- utiliser `async` et `await` ;
- organiser un projet Dart ;
- être suffisamment à l'aise avec Dart pour commencer Flutter.

La progression sera volontairement très graduelle.

| Chapitre | Sujet |
| --- | --- |
| 01 | Introduction à Dart |
| 02 | Variables, constantes et types |
| 03 | Opérateurs et expressions |
| 04 | Conditions : `if`, `else`, `switch` |
| 05 | Boucles : `for`, `while`, `do while` |
| 06 | Listes, Sets et Maps |
| 07 | Fonctions |
| 08 | Programmation orientée objet |
| 09 | Constructeurs, propriétés et méthodes |
| 10 | Encapsulation, héritage et polymorphisme |
| 11 | Classes abstraites, interfaces, mixins et enums |
| 12 | Null Safety |
| 13 | Exceptions |
| 14 | Programmation fonctionnelle utile à Flutter |
| 15 | Futures, `async` et `await` |
| 16 | Packages et organisation d'un projet Dart |
| 17 | JSON et manipulation de données |
| 18 | Mini-projet final Dart |

---

## 01.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qu'est Dart et pourquoi Flutter en dépend ;
- distinguer clairement Dart (le langage) de Flutter (le framework) ;
- expliquer pourquoi on apprend Dart avant Flutter ;
- écrire un premier programme Dart complet ;
- expliquer le rôle de `main()`, de `void` et des accolades `{ }` ;
- utiliser `print()` pour afficher du texte et des nombres ;
- effectuer des calculs simples avec Dart ;
- écrire des chaînes de caractères avec guillemets simples ou doubles ;
- terminer correctement une instruction avec `;` ;
- comprendre que Dart est sensible à la casse ;
- écrire des commentaires sur une ligne et sur plusieurs lignes ;
- comprendre l'exécution séquentielle d'un programme Dart ;
- utiliser DartPad pour exécuter du Dart dans le navigateur ;
- reconnaître un fichier `.dart` et sa convention de nommage `snake_case`.

---

## 01.1 — Qu'est-ce que Dart ?

Dart est un langage de programmation développé par Google.

Il est principalement connu parce qu'il constitue le langage utilisé par **Flutter**.

Lorsque nous écrirons plus tard :

```dart
Text('Bonjour')
```

ou :

```dart
ElevatedButton(
  onPressed: () {
    print('Bouton cliqué');
  },
  child: Text('Cliquez ici'),
)
```

nous utiliserons en réalité le langage Dart.

Flutter fournit les composants graphiques.

Dart fournit le langage permettant de les programmer.

On peut donc représenter la relation ainsi :

```text
DART
  |
  | langage de programmation
  |
  v
FLUTTER
  |
  | framework graphique
  |
  v
APPLICATION
  |
  +--> Android
  +--> iOS
  +--> Web
  +--> Windows
  +--> macOS
  +--> Linux
```

---

## 01.2 — Dart et Flutter ne sont pas la même chose

C'est une distinction fondamentale.

### Dart

Dart est un :

> langage de programmation.

Il contient notamment :

```text
Variables
Conditions
Boucles
Fonctions
Classes
Objets
Collections
Exceptions
Programmation asynchrone
```

### Flutter

Flutter est un :

> framework permettant de construire des interfaces et des applications multiplateformes.

Flutter utilise Dart.

On peut faire l'analogie suivante :

```text
Dart        = langage
Flutter     = boîte à outils
Application = résultat
```

Une autre comparaison serait :

```text
JavaScript → React
C#         → .NET
Dart       → Flutter
```

Cette comparaison n'est pas parfaite, mais elle aide à comprendre le rôle de Dart.

---

## 01.3 — Pourquoi apprendre Dart avant Flutter ?

Il est techniquement possible de commencer directement Flutter.

Mais cela provoque rapidement des difficultés.

Imaginez le code suivant :

```dart
class Produit {
  final String nom;
  final double prix;

  Produit({
    required this.nom,
    required this.prix,
  });
}
```

Une personne qui ne connaît pas Dart pourrait se demander :

```text
Pourquoi class ?
Pourquoi final ?
Pourquoi String ?
Pourquoi double ?
Pourquoi this.nom ?
Pourquoi required ?
Pourquoi { } ?
Pourquoi le constructeur porte le même nom que la classe ?
```

Nous allons répondre progressivement à chacune de ces questions.

Lorsque nous commencerons Flutter, ce code devra paraître normal.

---

## 01.4 — Premier programme Dart

Un programme Dart minimal peut être écrit ainsi :

```dart
void main() {
  print('Bonjour Dart');
}
```

**Résultat :**

```text
Bonjour Dart
```

Cette petite application contient déjà plusieurs concepts importants.

---

## 01.5 — Comprendre `main()`

Regardons :

```dart
void main() {
  print('Bonjour Dart');
}
```

La fonction :

```dart
main()
```

représente le **point de départ du programme**.

Lorsqu'un programme Dart est exécuté, Dart cherche la fonction :

```dart
main()
```

et commence l'exécution à cet endroit.

Nous pouvons visualiser cela ainsi :

```text
Lancement du programme
        |
        v
     main()
        |
        v
première instruction
        |
        v
deuxième instruction
        |
        v
       ...
```

---

## 01.6 — Comprendre `void`

Dans :

```dart
void main()
```

le mot :

```dart
void
```

indique que la fonction ne retourne aucune valeur.

Nous étudierons les fonctions beaucoup plus en détail plus tard.

Pour le moment, retenez simplement :

```dart
void main()
```

signifie approximativement :

> Voici la fonction principale du programme.

---

## 01.7 — Les accolades `{ }`

Dans :

```dart
void main() {
  print('Bonjour');
}
```

les accolades :

```text
{
}
```

délimitent un bloc de code.

Tout ce qui se trouve entre les accolades appartient ici à la fonction `main`.

Exemple :

```dart
void main() {
  print('Bonjour');
  print('Bienvenue');
  print('Nous apprenons Dart');
}
```

Les trois instructions appartiennent à `main()`.

---

## 01.8 — La fonction `print()`

La fonction :

```dart
print()
```

permet d'afficher une valeur dans la console.

Exemple :

```dart
void main() {
  print('Bonjour');
}
```

**Résultat :**

```text
Bonjour
```

Autre exemple :

```dart
void main() {
  print('Dart');
  print('Flutter');
  print('Développement mobile');
}
```

**Résultat :**

```text
Dart
Flutter
Développement mobile
```

---

## 01.9 — Afficher des nombres

`print()` ne sert pas uniquement à afficher du texte.

Nous pouvons écrire :

```dart
void main() {
  print(10);
  print(25);
  print(100);
}
```

**Résultat :**

```text
10
25
100
```

Nous pouvons également afficher le résultat d'un calcul :

```dart
void main() {
  print(10 + 5);
}
```

**Résultat :**

```text
15
```

---

## 01.10 — Premier calcul avec Dart

Essayons :

```dart
void main() {
  print(10 + 5);
  print(10 - 5);
  print(10 * 5);
  print(10 / 5);
}
```

**Résultat :**

```text
15
5
50
2.0
```

Dart peut donc être utilisé comme une calculatrice.

Mais nous allons rapidement aller beaucoup plus loin.

---

## 01.11 — Les chaînes de caractères

Un texte est généralement placé entre guillemets.

Exemple :

```dart
print('Bonjour');
```

Nous pouvons également utiliser :

```dart
print("Bonjour");
```

Les deux sont valides.

Par exemple :

```dart
void main() {
  print('Dart');
  print("Flutter");
}
```

---

## 01.12 — Une instruction et le point-virgule

Regardons :

```dart
print('Bonjour');
```

Le caractère :

```text
;
```

indique généralement la fin d'une instruction Dart.

Par exemple :

```dart
void main() {
  print('Bonjour');
  print('Comment allez-vous ?');
  print('Bienvenue dans le cours');
}
```

Chaque appel à `print()` constitue une instruction.

---

## 01.13 — Attention à la casse

Dart est sensible aux majuscules et aux minuscules.

On dit que Dart est :

> case-sensitive.

Par exemple :

```dart
print('Bonjour');
```

est correct.

Mais :

```dart
Print('Bonjour');
```

n'est pas la même chose.

De même :

```text
nom
Nom
NOM
```

représentent potentiellement trois identifiants différents.

---

## 01.14 — Les commentaires

Les commentaires servent à écrire des explications dans le code.

Dart ignore ces lignes pendant l'exécution.

### Commentaire sur une ligne

```dart
// Ceci est un commentaire

void main() {
  print('Bonjour');
}
```

Nous pouvons également écrire :

```dart
void main() {
  // Afficher un message
  print('Bonjour');
}
```

---

## 01.15 — Commentaires sur plusieurs lignes

On peut utiliser :

```dart
/*
Ceci est
un commentaire
sur plusieurs lignes.
*/
```

Exemple :

```dart
void main() {
  /*
    Ce programme affiche
    un message de bienvenue.
  */

  print('Bienvenue');
}
```

---

## 01.16 — Premier programme un peu plus complet

Créons :

```dart
void main() {
  print('==========================');
  print('       COURS DART');
  print('==========================');

  print('Bienvenue dans le cours.');
  print('Nous allons apprendre Dart.');
  print('Puis nous apprendrons Flutter.');

  print('==========================');
}
```

**Résultat :**

```text
==========================
       COURS DART
==========================
Bienvenue dans le cours.
Nous allons apprendre Dart.
Puis nous apprendrons Flutter.
==========================
```

---

## 01.17 — Exécution ligne par ligne

Dart exécute normalement les instructions dans l'ordre.

Avec :

```dart
void main() {
  print('A');
  print('B');
  print('C');
}
```

le programme exécute :

```text
Étape 1 → afficher A

Étape 2 → afficher B

Étape 3 → afficher C
```

**Résultat :**

```text
A
B
C
```

Ce principe paraît évident aujourd'hui.

Mais il deviendra important lorsque nous utiliserons :

```text
conditions
boucles
fonctions
événements
programmation asynchrone
```

---

## 01.18 — Modifier l'ordre

Considérons :

```dart
void main() {
  print('Troisième');
  print('Premier');
  print('Deuxième');
}
```

Dart ne comprend pas la signification des mots.

Il exécute simplement les instructions dans l'ordre :

```text
Troisième
Premier
Deuxième
```

L'ordre du code est donc important.

---

## 01.19 — Utiliser DartPad

Pour les premiers chapitres, un excellent moyen d'apprendre Dart consiste à utiliser DartPad.

DartPad permet d'écrire et d'exécuter du Dart directement dans le navigateur.

Cela permet de se concentrer sur le langage sans devoir immédiatement configurer un environnement complexe.

Pour nos premiers exercices, le fonctionnement sera simplement :

```text
Écrire le programme
        |
        v
Exécuter
        |
        v
Observer le résultat
        |
        v
Modifier le programme
        |
        v
Exécuter à nouveau
```

---

## 01.20 — Notre premier fichier Dart

Les fichiers Dart utilisent l'extension :

```text
.dart
```

Exemples :

```text
main.dart
bonjour.dart
calcul.dart
produit.dart
utilisateur.dart
jeu.dart
```

Dans un projet Flutter, vous rencontrerez notamment :

```text
lib/main.dart
```

Ce fichier jouera un rôle central.

Mais nous verrons cela dans la Partie 1B consacrée à Flutter.

---

## 01.21 — Convention de nommage des fichiers

Pour les fichiers Dart, on utilise généralement le style :

```text
snake_case
```

Exemple recommandé :

```text
gestion_produits.dart
```

plutôt que :

```text
GestionProduits.dart
```

ou :

```text
gestionProduits.dart
```

Nous verrons progressivement les conventions utilisées dans l'écosystème Dart.

---

## 01.22 — Premier programme avec plusieurs informations

```dart
void main() {
  print('Nom : Alex');
  print('Cours : Dart');
  print('Module : Introduction');
  print('Progression : débutant');
}
```

**Résultat :**

```text
Nom : Alex
Cours : Dart
Module : Introduction
Progression : débutant
```

Il y a cependant un problème.

Toutes les informations sont directement inscrites dans le programme.

Que se passerait-il si nous voulions changer le nom ?

Il faudrait modifier directement :

```dart
print('Nom : Alex');
```

Nous allons bientôt améliorer cela grâce aux **variables**.

---

## 01.23 — Ce qui va arriver au chapitre suivant

Au prochain chapitre, nous allons transformer ceci :

```dart
void main() {
  print('Alex');
  print(20);
}
```

en quelque chose de beaucoup plus intéressant :

```dart
void main() {
  String nom = 'Alex';
  int age = 20;

  print(nom);
  print(age);
}
```

Puis :

```dart
void main() {
  String nom = 'Alex';
  int age = 20;

  print('Nom : $nom');
  print('Âge : $age');
}
```

Nous découvrirons alors :

```text
String
int
double
bool
var
dynamic
final
const
```

Ces notions sont extrêmement importantes en Flutter.

Nous étudierons aussi l'interpolation, par exemple :

```dart
print('Bonjour $nom');
```

et :

```dart
print('Vous avez ${age + 1} ans l'année prochaine.');
```

---

## 01.24 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Void main() { ... }` ne compile pas | Dart est sensible à la casse : `Void` avec une majuscule n'est pas reconnu | Écrire `void` en minuscules |
| `Print('Bonjour')` ne compile pas | Même problème de casse sur `print` | Écrire `print` en minuscules |
| Oubli du point-virgule après une instruction | Chaque instruction Dart doit se terminer par `;` | Ajouter `;` à la fin de chaque appel à `print(...)` |
| Le programme ne fait rien à l'exécution | Le code n'est pas placé à l'intérieur de `void main() { ... }` | Toujours écrire les instructions entre les accolades de `main()` |
| Une accolade ouvrante `{` sans accolade fermante `}` | Bloc de code non refermé | Vérifier que chaque `{` a bien son `}` correspondant |
| `print(Bonjour)` sans guillemets | Dart interprète `Bonjour` comme un identifiant (une variable), pas un texte | Entourer le texte de guillemets simples ou doubles : `print('Bonjour')` |
| Confusion entre `10 / 5` et un nombre entier attendu | La division `/` renvoie toujours un `double` en Dart (ici `2.0`, pas `2`) | Garder à l'esprit que `/` retourne un `double` |
| Nom de fichier en `PascalCase` ou `camelCase` | La convention Dart pour les fichiers est le `snake_case` | Nommer le fichier `mon_fichier.dart`, pas `MonFichier.dart` |

---

## 01.25 — Résumé du chapitre

| Notion | À retenir |
| --- | --- |
| Dart | Langage de programmation développé par Google, utilisé par Flutter |
| Flutter | Framework graphique qui utilise le langage Dart |
| `main()` | Point de départ obligatoire d'un programme Dart |
| `void` | Indique qu'une fonction ne retourne aucune valeur |
| `{ }` | Délimitent un bloc de code (ici, le corps de `main()`) |
| `print()` | Affiche une valeur (texte, nombre, résultat de calcul) dans la console |
| `;` | Termine chaque instruction Dart |
| Casse | Dart distingue les majuscules des minuscules (`print` ≠ `Print`) |
| `// ...` | Commentaire sur une seule ligne |
| `/* ... */` | Commentaire sur plusieurs lignes |
| Exécution séquentielle | Les instructions s'exécutent dans l'ordre où elles sont écrites |
| DartPad | Outil en ligne pour écrire et exécuter du Dart sans installation |
| `.dart` | Extension des fichiers Dart |
| `snake_case` | Convention de nommage des fichiers Dart (ex. `gestion_produits.dart`) |

---

## 01.26 — Exercices

### Exercice 1 — Premier message (facile)

Écrivez un programme Dart affichant :

```text
Bonjour Dart
```

Structure attendue :

```dart
void main() {
  // Votre code
}
```

### Exercice 2 — Trois messages (facile)

Affichez :

```text
Bonjour
Bienvenue
Cours Dart
```

Chaque message doit utiliser un `print()` différent.

### Exercice 3 — Présentation (facile)

Créez un programme affichant :

```text
Nom : votre nom
Cours : Dart
Objectif : Flutter
```

Pour l'instant, écrivez directement les valeurs dans les `print()`.

### Exercice 4 — Calculs (facile)

À l'aide de `print()`, affichez les résultats des opérations suivantes :

```text
20 + 10
20 - 10
20 * 10
20 / 10
```

Ne calculez pas manuellement les résultats. Dart doit effectuer les calculs.

### Exercice 5 — Corriger le programme (intermédiaire)

Le programme suivant contient plusieurs erreurs :

```dart
Void main() {
  Print('Bonjour')
  print("Dart")
}
```

Corrigez-le.

### Exercice 6 — Ordre d'exécution (facile)

Sans exécuter le programme, indiquez le résultat :

```dart
void main() {
  print('C');
  print('A');
  print('B');
}
```

Puis exécutez le programme pour vérifier votre réponse.

### Exercice 7 — Commentaires (facile)

Ajoutez un commentaire avant chaque instruction.

Programme initial :

```dart
void main() {
  print('Dart');
  print('Flutter');
  print('Application mobile');
}
```

Exemple attendu :

```dart
void main() {
  // Premier commentaire
  print('Dart');

  // Deuxième commentaire
  print('Flutter');

  // Troisième commentaire
  print('Application mobile');
}
```

### Exercice 8 — Carte de présentation (intermédiaire)

Produisez exactement :

```text
----------------------------
PROFIL DU DÉVELOPPEUR
----------------------------
Nom : Alex
Technologie : Dart
Objectif : apprendre Flutter
----------------------------
```

Utilisez uniquement :

```text
void main()
print()
```

### Exercice 9 — Mini-calculatrice (intermédiaire)

Écrivez un programme produisant :

```text
Addition
15

Soustraction
5

Multiplication
50

Division
2.0
```

Les résultats doivent être calculés par Dart.

### Exercice 10 — Challenge (intermédiaire)

Essayez de produire :

```text
================================
      MON PARCOURS FLUTTER
================================
Étape 1 : apprendre Dart
Étape 2 : apprendre Flutter
Étape 3 : créer des applications
Étape 4 : apprendre Flame
Étape 5 : créer un jeu
================================
```

Vous ne devez utiliser que :

```dart
void main()
```

et :

```dart
print()
```

---

## 01.27 — Corrections des exercices

### Correction 1

```dart
void main() {
  print('Bonjour Dart');
}
```

**Explication :** un seul appel à `print()`, avec le texte exact entre guillemets, suivi d'un point-virgule.

### Correction 2

```dart
void main() {
  print('Bonjour');
  print('Bienvenue');
  print('Cours Dart');
}
```

**Explication :** chaque ligne à afficher correspond à un appel distinct à `print()`, chacun terminé par `;`.

### Correction 3

```dart
void main() {
  print('Nom : Alex');
  print('Cours : Dart');
  print('Objectif : Flutter');
}
```

**Explication :** les valeurs sont écrites directement dans le texte, car nous ne connaissons pas encore les variables (chapitre 02).

### Correction 4

```dart
void main() {
  print(20 + 10);
  print(20 - 10);
  print(20 * 10);
  print(20 / 10);
}
```

**Explication :** les calculs sont passés directement à `print()`, sans guillemets, car ce sont des nombres et non du texte. Dart calcule le résultat avant de l'afficher.

### Correction 5

```dart
void main() {
  print('Bonjour');
  print('Dart');
}
```

Il fallait corriger :

```text
Void  → void
Print → print
```

**Explication :** Dart est sensible à la casse (`Void` et `Print` avec majuscule ne sont pas reconnus), et chaque instruction doit se terminer par un point-virgule qui manquait dans le code fourni.

### Correction 6

**Résultat :**

```text
C
A
B
```

**Explication :** le programme exécute les instructions dans leur ordre d'apparition, quel que soit l'ordre alphabétique ou logique des valeurs affichées.

### Correction 7

```dart
void main() {
  // Afficher le langage
  print('Dart');

  // Afficher le framework
  print('Flutter');

  // Afficher l'objectif
  print('Application mobile');
}
```

**Explication :** chaque commentaire `// ...` est placé juste avant l'instruction qu'il décrit et n'a aucun effet sur l'exécution du programme.

### Correction 8

```dart
void main() {
  print('----------------------------');
  print('PROFIL DU DÉVELOPPEUR');
  print('----------------------------');
  print('Nom : Alex');
  print('Technologie : Dart');
  print('Objectif : apprendre Flutter');
  print('----------------------------');
}
```

**Explication :** les lignes de séparation sont de simples chaînes de caractères composées de tirets, affichées comme n'importe quel autre texte.

### Correction 9

```dart
void main() {
  print('Addition');
  print(10 + 5);

  print('');

  print('Soustraction');
  print(10 - 5);

  print('');

  print('Multiplication');
  print(10 * 5);

  print('');

  print('Division');
  print(10 / 5);
}
```

**Explication :** `print('')` affiche une ligne vide, ce qui permet d'aérer la sortie entre chaque opération. La division affiche `2.0` car `/` renvoie toujours un `double`.

### Correction 10

```dart
void main() {
  print('================================');
  print('      MON PARCOURS FLUTTER');
  print('================================');
  print('Étape 1 : apprendre Dart');
  print('Étape 2 : apprendre Flutter');
  print('Étape 3 : créer des applications');
  print('Étape 4 : apprendre Flame');
  print('Étape 5 : créer un jeu');
  print('================================');
}
```

**Explication :** le titre est centré manuellement à l'aide d'espaces au début de la chaîne de caractères, avant d'être encadré par deux lignes de signes `=`.

---

## Et maintenant ?

Vous savez maintenant écrire, lire et exécuter un programme Dart minimal, et vous comprenez la différence entre Dart et Flutter. Au chapitre suivant, nous allons remplacer les valeurs écrites en dur dans nos `print()` par de véritables **variables**, découvrir les types (`String`, `int`, `double`, `bool`), les mots-clés `var`, `dynamic`, `final` et `const`, ainsi que l'interpolation de chaînes avec `$nom` et `${expression}`.

Rendez-vous au chapitre suivant : [02-PARTIE-1A—VARIABLES-CONSTANTES-ET-TYPES.md](./02-PARTIE-1A—VARIABLES-CONSTANTES-ET-TYPES.md)
