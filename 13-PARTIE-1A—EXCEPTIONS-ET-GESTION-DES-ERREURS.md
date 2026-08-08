# PARTIE 1A — DART
# CHAPITRE 13 — EXCEPTIONS ET GESTION DES ERREURS

> **Niveau :** débutant / intermédiaire
> **Durée estimée :** 5 h
> **Pré-requis :** chapitre 12 — Le null safety
> **Ce que vous saurez faire à la fin :** écrire un programme Dart qui ne s'arrête jamais brutalement devant une donnée invalide, en signalant les problèmes avec `throw` et en les rattrapant au bon endroit avec `try` / `on` / `catch` / `finally`.

---

## 13.0 — Objectifs du chapitre

À la fin de ce chapitre, vous serez capable de :

- expliquer ce qui se passe, étape par étape, quand un programme Dart plante ;
- distinguer une `Error` (faute du programmeur) d'une `Exception` (accident prévisible) ;
- lire une trace d'erreur de la VM Dart sans paniquer ;
- protéger un morceau de code avec `try` ;
- rattraper un problème avec `catch` ;
- récupérer l'objet lancé avec `catch (e)` et la pile d'appels avec `catch (e, s)` ;
- filtrer un type d'exception précis avec `on` ;
- enchaîner plusieurs clauses `on` dans le bon ordre ;
- combiner `on` et `catch` avec `on ... catch (e)` ;
- garantir l'exécution d'un nettoyage avec `finally` ;
- décrire précisément l'ordre d'exécution de `try`, `catch` et `finally` ;
- signaler vous-même un problème avec `throw` ;
- faire remonter une exception déjà attrapée avec `rethrow` ;
- créer vos propres classes d'exception métier pour votre jeu ;
- comprendre et provoquer une `FormatException` ;
- expliquer pourquoi `int.parse()` échoue et comment le message est construit ;
- remplacer une exception par un test grâce à `int.tryParse()` et `double.tryParse()` ;
- valider une saisie utilisateur complète, du texte brut à la valeur exploitable ;
- utiliser `ArgumentError` et `assert` pour protéger vos propres fonctions ;
- articuler exceptions et null safety ;
- décider, dans un vrai projet, quand attraper une exception et quand laisser le programme planter.

---

## 13.1 — Pourquoi un programme plante

Un programme est une suite d'instructions. Chacune suppose que les conditions nécessaires sont réunies.

- `int.parse(texte)` suppose que `texte` contient bien un nombre.
- `inventaire[3]` suppose que la liste possède au moins quatre cases.
- `sauvegarde['niveau']` suppose que la clé `'niveau'` existe.
- `or - prix` suppose que le joueur possède assez d'or.

Quand une de ces suppositions est fausse, l'instruction ne peut **pas** produire un résultat correct. Il lui reste deux possibilités :

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Une instruction ne peut pas faire son travail               │
  ├──────────────────────────────────────────────────────────────┤
  │  Option 1 : renvoyer une valeur bidon                        │
  │             -> le bug continue et se propage silencieusement │
  │                                                              │
  │  Option 2 : s'interrompre et SIGNALER le problème            │
  │             -> c'est ce que fait Dart : il lance une         │
  │                exception                                     │
  └──────────────────────────────────────────────────────────────┘
```

Dart choisit toujours l'option 2. C'est un choix de conception : mieux vaut un arrêt bruyant qu'un score faux enregistré dans la sauvegarde du joueur.

Prenons le fil rouge du chapitre : un jeu qui charge une sauvegarde.

```text
  fichier de sauvegarde       ce que le jeu attend
  ────────────────────        ────────────────────
  "niveau=7"            ->    un entier : 7          OK
  "niveau=sept"         ->    un entier : ?          KO impossible
  "niveau="             ->    un entier : ?          KO impossible
```

Sur les deux dernières lignes, `int.parse` ne peut rien inventer. Il **lance une exception**.

Le mot « exception » est important. Ce n'est pas une valeur de retour : c'est un objet lancé **à la place** d'un retour normal, qui interrompt le déroulement du programme.

Voyons cela :

```dart
void main() {
  print('1. Ouverture de la sauvegarde');
  final String brut = 'sept';
  final int niveau = int.parse(brut);
  print('2. Niveau chargé : $niveau');
  print('3. Lancement du jeu');
}
```

**Résultat :**

```text
1. Ouverture de la sauvegarde
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
sept
^

#0      int.parse (dart:core-patch/integers_patch.dart:63:39)
#1      main (file:///jeu/main.dart:4:26)
```

Observez bien : la ligne `1.` s'affiche, les lignes `2.` et `3.` ne s'affichent **jamais**. Le programme s'arrête net à l'instruction fautive.

C'est exactement ce que ce chapitre va vous apprendre à contrôler.

---

## 13.2 — Erreur (`Error`) contre exception (`Exception`)

Dart possède deux grandes familles d'objets « lançables ». La distinction n'est pas cosmétique : elle décide de ce que vous devez faire.

```text
  ┌──────────────────────────────────────────────────────────────┐
  │  Object                                                      │
  │    ├── Error       « le programmeur s'est trompé »           │
  │    │     ├── ArgumentError                                   │
  │    │     ├── RangeError                                      │
  │    │     ├── StateError                                      │
  │    │     ├── UnsupportedError                                │
  │    │     ├── TypeError                                       │
  │    │     └── AssertionError                                  │
  │    │                                                         │
  │    └── Exception    « un accident prévisible s'est produit »  │
  │          ├── FormatException                                 │
  │          ├── IntegerDivisionByZeroException                  │
  │          └── vos propres exceptions métier                   │
  └──────────────────────────────────────────────────────────────┘
```

La règle mentale à retenir :

| Famille | Signification | Ce qu'il faut faire |
| --- | --- | --- |
| `Error` | Le code est **faux**. Ce cas ne devrait jamais arriver. | **Corriger le code.** Ne pas attraper. |
| `Exception` | Le code est correct, mais la **donnée** ou l'environnement est mauvais. | **Attraper** et réagir proprement. |

Un exemple de chaque, dans le jeu.

```dart
void main() {
  final List<String> inventaire = ['potion', 'clé'];

  // Error : lire la case 5 d'une liste de 2 éléments est une faute de code.
  print(inventaire[5]);
}
```

**Résultat :**

```text
Unhandled exception:
RangeError (index): Invalid value: Not in inclusive range 0..1: 5
#0      List.[] (dart:core-patch/growable_array.dart:264:36)
#1      main (file:///jeu/main.dart:5:19)
```

Ici, attraper l'erreur serait absurde. Il faut corriger l'indice, ou vérifier `inventaire.length` avant.

```dart
void main() {
  // Exception : le fichier de sauvegarde contient n'importe quoi.
  // Le code est correct, c'est la donnée qui est mauvaise.
  final String brut = 'niveau-sept';
  print(int.parse(brut));
}
```

**Résultat :**

```text
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
niveau-sept
^
```

Ici, attraper a un sens : le joueur ne doit pas voir son jeu se fermer parce qu'un fichier est abîmé. On affichera un message et on repartira d'une sauvegarde vierge.

> **Remarque technique :** rien dans le langage ne vous *interdit* d'attraper une `Error`. C'est simplement une très mauvaise idée : vous masqueriez un bug au lieu de le corriger. Un `RangeError` attrapé et ignoré, c'est un bug qui vous reviendra un mois plus tard sous une forme incompréhensible.

Notez aussi un détail de vocabulaire français : on dit couramment « une erreur » pour tout ce qui plante. En Dart, `Error` a un sens **précis** et restreint. Dans ce chapitre, quand nous écrivons `Error` en police de code, nous parlons de la classe Dart.

---

## 13.3 — Ce qui se passe sans protection

Comprendre le mécanisme est indispensable avant d'apprendre la syntaxe.

Quand une instruction lance une exception, Dart ne « saute pas la ligne ». Il **remonte la pile d'appels** en cherchant un gestionnaire.

```text
  main()
    └── chargerPartie()
          └── lireNiveau()
                └── int.parse('sept')   <- LANCE FormatException

  Recherche d'un gestionnaire, en remontant :

  lireNiveau()      : pas de try/catch  -> on abandonne cette fonction
  chargerPartie()   : pas de try/catch  -> on abandonne cette fonction
  main()            : pas de try/catch  -> on abandonne cette fonction
  ────────────────────────────────────────────────────────────────
  Plus rien au-dessus -> le programme s'arrête, la VM imprime
  l'exception et la trace.
```

Vérifions-le sur du vrai code.

```dart
int lireNiveau(String brut) {
  print('  [lireNiveau] début');
  final int n = int.parse(brut);
  print('  [lireNiveau] fin');
  return n;
}

void chargerPartie(String brut) {
  print(' [chargerPartie] début');
  final int niveau = lireNiveau(brut);
  print(' [chargerPartie] niveau = $niveau');
  print(' [chargerPartie] fin');
}

void main() {
  print('[main] début');
  chargerPartie('sept');
  print('[main] fin');
}
```

**Résultat :**

```text
[main] début
 [chargerPartie] début
  [lireNiveau] début
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
sept
^

#0      int.parse (dart:core-patch/integers_patch.dart:63:39)
#1      lireNiveau (file:///jeu/main.dart:3:22)
#2      chargerPartie (file:///jeu/main.dart:10:22)
#3      main (file:///jeu/main.dart:17:3)
```

Trois observations capitales :

1. Aucun des `print` placés **après** l'instruction fautive ne s'exécute. Ni dans `lireNiveau`, ni dans `chargerPartie`, ni dans `main`. Les trois fonctions sont abandonnées d'un coup.
2. Le mot-clé du message est `Unhandled exception`, littéralement « exception non gérée ». La VM vous dit : personne n'a voulu de cette exception.
3. La liste `#0 #1 #2 #3` est la **trace de pile** (*stack trace*). Elle se lit **de haut en bas**, du plus profond au plus superficiel. `#0` est l'endroit exact où l'exception est née ; le dernier numéro est `main`.

> **Comment lire une trace :** cherchez la première ligne qui mentionne **votre** fichier. Ici c'est `#1 lireNiveau (file:///jeu/main.dart:3:22)`. Les lignes `dart:core-patch/...` sont du code de la bibliothèque standard : ce n'est presque jamais là que se trouve votre bug.

Le format `main.dart:3:22` signifie : fichier `main.dart`, **ligne 3**, **colonne 22**.

> **Remarque DartPad :** DartPad exécute parfois votre code compilé en JavaScript. Les traces y sont moins lisibles et les numéros de lignes peuvent différer. Le message principal (`FormatException: ...`) reste identique, et c'est lui qui compte.

---

## 13.4 — Le bloc `try`

`try` signifie « essayer ». Vous entourez d'accolades le code susceptible d'échouer.

```dart
try {
  // code surveillé
}
```

Écrit seul, `try` ne compile pas :

```text
Error: A try block must be followed by an 'on', 'catch', or 'finally' clause.
```

C'est logique : surveiller sans rien faire du problème n'aurait aucun intérêt. `try` doit toujours être suivi d'au moins une clause `catch`, `on` ou `finally`.

Ce que `try` change, c'est la **zone de recherche** décrite en 13.3 :

```text
  Sans try                        Avec try
  ────────                        ────────
  int.parse -> exception          try {
       ↓ remonte                    int.parse -> exception
  lireNiveau                      } catch (e) {
       ↓ remonte                    ← LE VOYAGE S'ARRÊTE ICI
  chargerPartie                     ← on exécute ce bloc
       ↓ remonte                  }
  main                            print('la suite continue');
       ↓
  CRASH
```

Un point souvent mal compris : dès que l'exception survient, **le reste du bloc `try` est abandonné**. On ne revient jamais à l'instruction suivante.

```dart
void main() {
  try {
    print('A');
    final int niveau = int.parse('sept');
    print('B - jamais affiché, niveau = $niveau');
  } catch (e) {
    print('C - problème rattrapé');
  }
  print('D');
}
```

**Résultat :**

```text
A
C - problème rattrapé
D
```

`B` n'apparaît pas. `D` apparaît : le programme a survécu.

> **Règle de portée :** mettez dans le `try` le **minimum** de code. Un `try` qui entoure cinquante lignes rend impossible de savoir laquelle a échoué.

---

## 13.5 — Le bloc `catch`

`catch` signifie « attraper ». Il s'exécute **uniquement** si le bloc `try` a lancé quelque chose.

```dart
void main() {
  final String sauvegarde = 'sept';

  try {
    final int niveau = int.parse(sauvegarde);
    print('Bienvenue, niveau $niveau.');
  } catch (e) {
    print('Sauvegarde illisible. Démarrage d\'une nouvelle partie.');
  }

  print('Le jeu tourne.');
}
```

**Résultat :**

```text
Sauvegarde illisible. Démarrage d'une nouvelle partie.
Le jeu tourne.
```

Et si la sauvegarde est correcte ?

```dart
void main() {
  final String sauvegarde = '7';

  try {
    final int niveau = int.parse(sauvegarde);
    print('Bienvenue, niveau $niveau.');
  } catch (e) {
    print('Sauvegarde illisible. Démarrage d\'une nouvelle partie.');
  }

  print('Le jeu tourne.');
}
```

**Résultat :**

```text
Bienvenue, niveau 7.
Le jeu tourne.
```

Le bloc `catch` est totalement ignoré. Il ne coûte rien quand tout va bien.

Attention à un piège de portée. Une variable déclarée **dans** le `try` n'existe pas en dehors :

```dart
void main() {
  try {
    final int niveau = int.parse('7');
  } catch (e) {
    print('Erreur');
  }
  print(niveau); // refusé
}
```

**Résultat :**

```text
Error: Undefined name 'niveau'.
```

La bonne pratique est de déclarer la variable **avant** le `try`, avec une valeur de repli :

```dart
void main() {
  int niveau = 1; // valeur par défaut : nouvelle partie

  try {
    niveau = int.parse('sept');
  } catch (e) {
    print('Sauvegarde corrompue, retour au niveau 1.');
  }

  print('Niveau utilisé : $niveau');
}
```

**Résultat :**

```text
Sauvegarde corrompue, retour au niveau 1.
Niveau utilisé : 1
```

C'est le schéma le plus courant en production : une valeur sûre par défaut, une tentative de la remplacer, un repli propre en cas d'échec.

---

## 13.6 — `catch (e)` et `catch (e, s)`

`catch` accepte un ou deux paramètres.

```text
  catch (e)        e = l'objet lancé
  catch (e, s)     e = l'objet lancé, s = la trace de pile (StackTrace)
```

Le premier paramètre est **l'objet** qui a été lancé. Ce n'est pas un texte : c'est un objet Dart, souvent une instance de `FormatException`, `RangeError`, ou d'une de vos classes.

```dart
void main() {
  try {
    int.parse('sept');
  } catch (e) {
    print('Type reçu  : ${e.runtimeType}');
    print('Affichage  : $e');
  }
}
```

**Résultat :**

```text
Type reçu  : FormatException
Affichage  : FormatException: Invalid radix-10 number (at character 1)
sept
^
```

`$e` appelle automatiquement `toString()` sur l'objet. C'est pour cela que le message est lisible.

Le second paramètre, la **trace de pile**, sert à savoir *d'où* venait l'exception. Il est indispensable dans un vrai jeu, où l'erreur est signalée dans un journal plutôt qu'affichée au joueur.

```dart
int lireNiveau(String brut) {
  return int.parse(brut);
}

void main() {
  try {
    lireNiveau('sept');
  } catch (e, s) {
    print('--- Incident ---');
    print('Message : $e');
    print('Origine :');
    print(s);
  }
  print('Le jeu continue.');
}
```

**Résultat :**

```text
--- Incident ---
Message : FormatException: Invalid radix-10 number (at character 1)
sept
^
Origine :
#0      int.parse (dart:core-patch/integers_patch.dart:63:39)
#1      lireNiveau (file:///jeu/main.dart:2:14)
#2      main (file:///jeu/main.dart:7:5)
Le jeu continue.
```

Remarquez la dernière ligne : le programme **n'a pas planté**. Nous avons obtenu exactement la même information qu'un crash, mais en gardant la main.

Deux remarques importantes.

1. Le nom des paramètres est libre. `catch (e, s)` est la convention ; `catch (erreur, trace)` fonctionne tout aussi bien et se lit mieux pour un débutant.
2. Si vous ne vous servez pas de la trace, ne la déclarez pas. `catch (e)` suffit.

```dart
void main() {
  try {
    int.parse('12.5');
  } catch (erreur, trace) {
    print('Erreur attrapée : $erreur');
    print('Première ligne de la trace : ${trace.toString().split('\n').first}');
  }
}
```

**Résultat :**

```text
Erreur attrapée : FormatException: Invalid radix-10 number (at character 3)
12.5
  ^
Première ligne de la trace : #0      int.parse (dart:core-patch/integers_patch.dart:63:39)
```

> **Détail à noter :** le message n'est pas figé. Ici, la position change (`at character 3` au lieu de `at character 1`) parce que les deux premiers caractères étaient valides et que le point ne l'est pas. Ne cherchez donc **jamais** à reconnaître une exception en comparant son message texte. Nous verrons en 13.7 la bonne méthode, qui consiste à filtrer sur le **type**.

---

## 13.7 — Attraper un type précis avec `on`

`catch (e)` attrape **tout**. C'est souvent trop large.

Imaginons une boutique. Deux choses peuvent mal tourner : le prix saisi n'est pas un nombre (`FormatException`), ou l'article demandé n'existe pas dans le catalogue (`RangeError`). Les deux problèmes n'appellent pas la même réponse.

Le mot-clé `on` filtre sur le **type** de l'objet lancé.

```dart
void main() {
  try {
    final int prix = int.parse('cent');
    print('Prix : $prix');
  } on FormatException {
    print('Le prix saisi n\'est pas un nombre.');
  }
}
```

**Résultat :**

```text
Le prix saisi n'est pas un nombre.
```

Notez qu'ici, `on FormatException` **sans** `catch` ne vous donne pas accès à l'objet. C'est parfait quand le type suffit à décider.

Que se passe-t-il si l'exception lancée n'est pas du type filtré ?

```dart
void main() {
  final List<String> catalogue = ['potion', 'épée'];

  try {
    print(catalogue[9]);
  } on FormatException {
    print('Le prix saisi n\'est pas un nombre.');
  }
  print('Fin');
}
```

**Résultat :**

```text
Unhandled exception:
RangeError (index): Invalid value: Not in inclusive range 0..1: 9
#0      List.[] (dart:core-patch/growable_array.dart:264:36)
#1      main (file:///jeu/main.dart:5:19)
```

Le `on FormatException` ne correspond pas : l'exception continue sa remontée, et le programme plante. C'est **le comportement souhaité** : on n'attrape que ce que l'on sait traiter.

Le filtrage tient compte de l'héritage. `on Exception` attrape `FormatException`, puisque `FormatException` est une sous-classe d'`Exception`.

```dart
void main() {
  try {
    int.parse('cent');
  } on Exception {
    print('Une exception (prévisible) est survenue.');
  }
}
```

**Résultat :**

```text
Une exception (prévisible) est survenue.
```

En revanche, `on Exception` n'attrape **pas** un `RangeError`, car `RangeError` descend d'`Error`, pas d'`Exception`. C'est précisément la séparation vue en 13.2, et elle vous rend service : `on Exception` est un filet raisonnable qui laisse remonter les bugs de programmation.

---

## 13.8 — Plusieurs `on` en cascade

Vous pouvez enchaîner autant de clauses `on` que nécessaire. Dart les essaie **dans l'ordre d'écriture** et exécute **la première qui correspond**.

```dart
void acheter(List<String> catalogue, String indexBrut, String orBrut) {
  final int index = int.parse(indexBrut);
  final String article = catalogue[index];
  final int or = int.parse(orBrut);
  print('Achat de $article pour $or pièces.');
}

void main() {
  final List<String> catalogue = ['potion', 'épée', 'bouclier'];

  final List<List<String>> essais = [
    ['1', '50'],
    ['deux', '50'],
    ['7', '50'],
  ];

  for (final essai in essais) {
    print('--- Essai ${essai[0]} / ${essai[1]} ---');
    try {
      acheter(catalogue, essai[0], essai[1]);
    } on FormatException {
      print('Saisie invalide : entrez un nombre.');
    } on RangeError {
      print('Cet article n\'existe pas dans la boutique.');
    }
  }
}
```

**Résultat :**

```text
--- Essai 1 / 50 ---
Achat de épée pour 50 pièces.
--- Essai deux / 50 ---
Saisie invalide : entrez un nombre.
--- Essai 7 / 50 ---
Cet article n'existe pas dans la boutique.
```

Chaque problème reçoit un message adapté, et la boucle continue. Aucune sortie brutale.

L'**ordre compte**, et il n'est pas négociable : allez toujours **du plus précis au plus général**.

```dart
void main() {
  try {
    int.parse('cent');
  } on Exception {
    print('Cas général');
  } on FormatException {
    print('Cas précis - jamais atteint');
  }
}
```

**Résultat :**

```text
Cas général
```

L'analyseur vous prévient :

```text
Dead code: this on-catch block will never be executed because
'FormatException' is a subtype of 'Exception' and hence will have been
caught already.
```

`FormatException` étant une sous-classe d'`Exception`, la première clause l'attrape toujours en premier. La seconde est du code mort.

La forme correcte :

```text
  on FormatException  { ... }   ← le plus précis en haut
  on Exception        { ... }
  catch (e)           { ... }   ← le fourre-tout en bas, si nécessaire
```

Un `catch (e)` sans type est nécessairement **le dernier**. Si vous placez une clause après lui, l'analyseur signale :

```text
Dead code: catch clauses after a 'catch (e)' or an 'on Object catch'
clause are never reached.
```

---

## 13.9 — `on ... catch (e)`

Vous voulez souvent les deux à la fois : filtrer sur le type **et** consulter l'objet. C'est la combinaison `on Type catch (e)`.

```dart
void main() {
  try {
    int.parse('niveau7');
  } on FormatException catch (e) {
    print('Format invalide.');
    print('Détail technique : ${e.message}');
    print('Texte fautif    : ${e.source}');
    print('Position        : ${e.offset}');
  }
}
```

**Résultat :**

```text
Format invalide.
Détail technique : Invalid radix-10 number
Texte fautif    : niveau7
Position        : 0
```

L'intérêt est énorme : parce que le type est connu, `e` est **typé** `FormatException`. Vous accédez donc à ses propriétés spécifiques (`message`, `source`, `offset`) sans aucune conversion.

Comparez avec un `catch (e)` nu :

```dart
void main() {
  try {
    int.parse('niveau7');
  } catch (e) {
    print(e.message);
  }
}
```

**Résultat :**

```text
Error: The getter 'message' isn't defined for the class 'Object'.
```

Avec `catch (e)` seul, `e` est de type `Object` : Dart ne sait pas ce qui a été lancé, donc il ne connaît aucune propriété particulière.

La forme complète accepte aussi la trace :

```dart
void main() {
  try {
    int.parse('niveau7');
  } on FormatException catch (e, s) {
    print('Message : ${e.message}');
    print('Origine : ${s.toString().split('\n')[1].trim()}');
  }
}
```

**Résultat :**

```text
Message : Invalid radix-10 number
Origine : #1      main (file:///jeu/main.dart:3:9)
```

Retenez la grille de choix :

| Écriture | Quand l'utiliser |
| --- | --- |
| `on Type { }` | le type suffit, l'objet ne vous apprend rien |
| `on Type catch (e) { }` | vous voulez lire les propriétés de l'exception |
| `on Type catch (e, s) { }` | vous journalisez l'incident |
| `catch (e) { }` | filet de sécurité de dernier recours |

---

## 13.10 — `finally`

`finally` signifie « pour finir ». Ce bloc s'exécute **toujours** :

- si le `try` s'est bien passé ;
- si une exception a été attrapée ;
- si une exception n'a **pas** été attrapée et remonte ;
- même si le `try` contient un `return`.

C'est le bloc du **nettoyage** : fermer un fichier, libérer une ressource, arrêter un chronomètre, remettre une variable d'état à sa place.

Cas 1 : tout se passe bien.

```dart
void main() {
  try {
    print('Ouverture de la sauvegarde');
    final int niveau = int.parse('7');
    print('Niveau : $niveau');
  } catch (e) {
    print('Problème : $e');
  } finally {
    print('Fermeture de la sauvegarde');
  }
}
```

**Résultat :**

```text
Ouverture de la sauvegarde
Niveau : 7
Fermeture de la sauvegarde
```

Cas 2 : une exception attrapée.

```dart
void main() {
  try {
    print('Ouverture de la sauvegarde');
    final int niveau = int.parse('sept');
    print('Niveau : $niveau');
  } catch (e) {
    print('Problème : sauvegarde illisible');
  } finally {
    print('Fermeture de la sauvegarde');
  }
}
```

**Résultat :**

```text
Ouverture de la sauvegarde
Problème : sauvegarde illisible
Fermeture de la sauvegarde
```

Cas 3 : aucune clause `catch`. Le `finally` s'exécute **quand même**, puis l'exception poursuit sa route.

```dart
void main() {
  try {
    print('Ouverture de la sauvegarde');
    int.parse('sept');
  } finally {
    print('Fermeture de la sauvegarde');
  }
  print('Jamais affiché');
}
```

**Résultat :**

```text
Ouverture de la sauvegarde
Fermeture de la sauvegarde
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
sept
^
```

C'est le cas le plus instructif. La ressource est bien libérée, et le problème n'est pas masqué : il remonte à qui saura le traiter.

Cas 4 : un `return` dans le `try`.

```dart
int chargerNiveau(String brut) {
  try {
    return int.parse(brut);
  } catch (e) {
    return 1;
  } finally {
    print('  (fichier refermé)');
  }
}

void main() {
  print('Résultat : ${chargerNiveau('7')}');
  print('Résultat : ${chargerNiveau('sept')}');
}
```

**Résultat :**

```text
  (fichier refermé)
Résultat : 7
  (fichier refermé)
Résultat : 1
```

Le `finally` s'exécute **avant** que la fonction ne rende réellement la main. La valeur de retour est déjà calculée, mais le nettoyage passe d'abord.

> **Piège à éviter absolument :** ne mettez jamais de `return` dans un `finally`. Il écraserait silencieusement la valeur retournée par le `try` ou par le `catch`, et masquerait même une exception en cours de remontée. C'est une source de bugs redoutable.

---

## 13.11 — Ordre d'exécution try / catch / finally

Il est temps de fixer l'ordre exact. Voici les trois scénarios, sous forme de schéma.

```text
  SCÉNARIO A : aucune exception
  ─────────────────────────────
  try     ██████████ tout le bloc s'exécute
  catch   ░░░░░░░░░░ ignoré
  finally ██████████ s'exécute
  suite   ██████████ s'exécute


  SCÉNARIO B : exception attrapée
  ───────────────────────────────
  try     ███░░░░░░░ s'arrête à l'instruction fautive
  catch   ██████████ s'exécute
  finally ██████████ s'exécute
  suite   ██████████ s'exécute


  SCÉNARIO C : exception NON attrapée (mauvais type, ou pas de catch)
  ──────────────────────────────────────────────────────────────────
  try     ███░░░░░░░ s'arrête à l'instruction fautive
  catch   ░░░░░░░░░░ ne correspond pas
  finally ██████████ s'exécute quand même
  suite   ░░░░░░░░░░ abandonné, l'exception remonte
```

Un programme unique qui montre les trois scénarios :

```dart
void tester(String etiquette, String donnee) {
  print('=== $etiquette ===');
  try {
    print('  try : début');
    final int n = int.parse(donnee);
    print('  try : valeur = $n');
    print('  try : fin');
  } on FormatException {
    print('  catch : FormatException rattrapée');
  } finally {
    print('  finally : nettoyage');
  }
  print('  après le bloc');
}

void main() {
  tester('A - donnée valide', '7');
  tester('B - donnée invalide', 'sept');
}
```

**Résultat :**

```text
=== A - donnée valide ===
  try : début
  try : valeur = 7
  try : fin
  finally : nettoyage
  après le bloc
=== B - donnée invalide ===
  try : début
  catch : FormatException rattrapée
  finally : nettoyage
  après le bloc
```

Et le scénario C, où le type ne correspond pas :

```dart
void main() {
  final List<String> sac = ['potion'];

  print('début');
  try {
    print('  try : début');
    print(sac[5]);
    print('  try : fin');
  } on FormatException {
    print('  catch : jamais atteint');
  } finally {
    print('  finally : nettoyage quand même');
  }
  print('après le bloc - jamais atteint');
}
```

**Résultat :**

```text
début
  try : début
  finally : nettoyage quand même
Unhandled exception:
RangeError (index): Invalid value: Not in inclusive range 0..0: 5
#0      List.[] (dart:core-patch/growable_array.dart:264:36)
#1      main (file:///jeu/main.dart:7:14)
```

Retenez la phrase : **`finally` est le seul bloc dont l'exécution est garantie.**

---

## 13.12 — Lancer une exception avec `throw`

Jusqu'ici, nous attrapions des exceptions lancées par la bibliothèque standard. Vous pouvez, et devez, en lancer vous-même.

Le mot-clé est `throw` (« lancer »).

```dart
void acheter(int or, int prix) {
  if (or < prix) {
    throw Exception('Or insuffisant : $or pièces pour un prix de $prix');
  }
  print('Achat validé. Il vous reste ${or - prix} pièces.');
}

void main() {
  acheter(120, 50);
  acheter(30, 50);
  print('Jamais affiché');
}
```

**Résultat :**

```text
Achat validé. Il vous reste 70 pièces.
Unhandled exception:
Exception: Or insuffisant : 30 pièces pour un prix de 50
#0      acheter (file:///jeu/main.dart:3:5)
#1      main (file:///jeu/main.dart:10:3)
```

`throw` interrompt immédiatement la fonction, exactement comme un `return`, mais en signalant un échec au lieu de rendre une valeur.

Le même code, protégé :

```dart
void acheter(int or, int prix) {
  if (or < prix) {
    throw Exception('Or insuffisant : $or pièces pour un prix de $prix');
  }
  print('Achat validé. Il vous reste ${or - prix} pièces.');
}

void main() {
  for (final int or in [120, 30]) {
    try {
      acheter(or, 50);
    } on Exception catch (e) {
      print('Achat refusé -> $e');
    }
  }
  print('Retour au menu.');
}
```

**Résultat :**

```text
Achat validé. Il vous reste 70 pièces.
Achat refusé -> Exception: Or insuffisant : 30 pièces pour un prix de 50
Retour au menu.
```

Trois précisions techniques.

**1. `Exception('texte')` n'est pas une classe à part entière.** C'est une fabrique pratique qui produit un objet minimal. Son `toString()` donne `Exception: texte`. Pour du vrai code métier, préférez vos propres classes (section 13.14).

**2. Dart autorise de lancer n'importe quel objet non nul.** Y compris une chaîne :

```dart
void main() {
  try {
    throw 'La sauvegarde est corrompue';
  } catch (e) {
    print('Attrapé : $e');
    print('Type    : ${e.runtimeType}');
  }
}
```

**Résultat :**

```text
Attrapé : La sauvegarde est corrompue
Type    : String
```

Cela **fonctionne**, mais c'est une mauvaise pratique : on ne peut plus filtrer avec `on`, et l'objet ne porte aucune information structurée. Ne lancez que des `Exception` ou des `Error`.

**3. Lancer `null` est interdit** par le null safety :

```dart
void main() {
  throw null;
}
```

**Résultat :**

```text
Error: Can't throw a value of 'Null' as an exception.
```

Enfin, notez que `throw` est aussi une **expression**. On peut donc l'écrire dans une fonction fléchée :

```dart
int niveauValide(int n) => n > 0 ? n : throw ArgumentError('Niveau invalide : $n');

void main() {
  print(niveauValide(5));
  print(niveauValide(-2));
}
```

**Résultat :**

```text
5
Unhandled exception:
Invalid argument(s): Niveau invalide : -2
#0      niveauValide (file:///jeu/main.dart:1:44)
#1      main (file:///jeu/main.dart:5:9)
```

---

## 13.13 — `rethrow`

Parfois, vous voulez faire quelque chose au passage — journaliser, compter, fermer une ressource — mais **sans** prendre la responsabilité de régler le problème. Le mot-clé est `rethrow` (« relancer »).

Comparons. Voici d'abord la **mauvaise** version, avec `throw e` :

```dart
int lireNiveau(String brut) {
  try {
    return int.parse(brut);
  } catch (e) {
    print('[journal] échec de lecture du niveau');
    throw e; // mauvais
  }
}

void main() {
  lireNiveau('sept');
}
```

**Résultat :**

```text
[journal] échec de lecture du niveau
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
sept
^

#0      lireNiveau (file:///jeu/main.dart:6:5)
#1      main (file:///jeu/main.dart:11:3)
```

Regardez `#0` : la trace prétend que l'exception est née **ligne 6**, c'est-à-dire à la ligne du `throw e`. L'information d'origine (`int.parse`, ligne 3) a disparu. Vous avez perdu la piste du vrai coupable.

Maintenant, la **bonne** version, avec `rethrow` :

```dart
int lireNiveau(String brut) {
  try {
    return int.parse(brut);
  } catch (e) {
    print('[journal] échec de lecture du niveau');
    rethrow;
  }
}

void main() {
  lireNiveau('sept');
}
```

**Résultat :**

```text
[journal] échec de lecture du niveau
Unhandled exception:
FormatException: Invalid radix-10 number (at character 1)
sept
^

#0      int.parse (dart:core-patch/integers_patch.dart:63:39)
#1      lireNiveau (file:///jeu/main.dart:3:16)
#2      main (file:///jeu/main.dart:11:3)
```

La trace est **intacte** : `#0` pointe toujours sur `int.parse`. C'est exactement l'information dont vous avez besoin pour corriger.

> **Règle :** dans un `catch`, si vous voulez faire remonter la **même** exception, écrivez toujours `rethrow`, jamais `throw e`.

Un usage typique dans le jeu : la couche basse journalise, la couche haute décide.

```dart
Map<String, int> lireSauvegarde(String contenu) {
  try {
    final List<String> morceaux = contenu.split(';');
    return {
      'niveau': int.parse(morceaux[0]),
      'or': int.parse(morceaux[1]),
    };
  } catch (e) {
    print('[journal] sauvegarde illisible : "$contenu"');
    rethrow;
  }
}

void main() {
  for (final String fichier in ['7;250', '7;deux-cents']) {
    try {
      final Map<String, int> data = lireSauvegarde(fichier);
      print('Partie chargée : niveau ${data['niveau']}, or ${data['or']}');
    } catch (e) {
      print('Nouvelle partie créée.');
    }
  }
}
```

**Résultat :**

```text
Partie chargée : niveau 7, or 250
[journal] sauvegarde illisible : "7;deux-cents"
Nouvelle partie créée.
```

Deux responsabilités, deux endroits : `lireSauvegarde` **constate**, `main` **décide**.

Enfin, `rethrow` n'a de sens que dans un `catch` :

```dart
void main() {
  rethrow;
}
```

**Résultat :**

```text
Error: 'rethrow' can only be used in catch clauses.
```

---

## 13.14 — Exceptions personnalisées

`Exception('texte')` dépanne, mais il ne dit rien de votre jeu. Dans un vrai projet, on écrit des classes d'exception **métier** : une par situation anormale identifiée.

Une exception personnalisée est une classe ordinaire qui **implémente** `Exception` :

```dart
class OrInsuffisantException implements Exception {
  final int or;
  final int prix;

  OrInsuffisantException(this.or, this.prix);

  int get manque => prix - or;

  @override
  String toString() => 'OrInsuffisantException : il manque $manque pièces';
}
```

Trois éléments à repérer :

1. `implements Exception` : c'est ce qui rend la classe filtrable par `on Exception`.
2. Des **champs** qui portent le contexte (`or`, `prix`). C'est tout l'intérêt : votre `catch` recevra des données, pas une phrase.
3. Un `toString()` redéfini, pour un affichage lisible.

Voyons l'usage complet.

```dart
class OrInsuffisantException implements Exception {
  final int or;
  final int prix;

  OrInsuffisantException(this.or, this.prix);

  int get manque => prix - or;

  @override
  String toString() => 'OrInsuffisantException : il manque $manque pièces';
}

class InventairePleinException implements Exception {
  final int capacite;

  InventairePleinException(this.capacite);

  @override
  String toString() => 'InventairePleinException : $capacite emplacements occupés';
}

class Boutique {
  final List<String> sac;
  final int capacite;
  int or;

  Boutique({required this.or, required this.sac, this.capacite = 3});

  void acheter(String article, int prix) {
    if (or < prix) {
      throw OrInsuffisantException(or, prix);
    }
    if (sac.length >= capacite) {
      throw InventairePleinException(capacite);
    }
    or -= prix;
    sac.add(article);
    print('Vous achetez $article. Or restant : $or');
  }
}

void main() {
  final Boutique boutique = Boutique(or: 120, sac: ['potion']);

  final List<List<Object>> commandes = [
    ['épée', 50],
    ['armure', 500],
    ['bouclier', 30],
    ['casque', 20],
  ];

  for (final commande in commandes) {
    final String article = commande[0] as String;
    final int prix = commande[1] as int;
    try {
      boutique.acheter(article, prix);
    } on OrInsuffisantException catch (e) {
      print('Trop cher pour $article : il vous manque ${e.manque} pièces.');
    } on InventairePleinException catch (e) {
      print('Sac plein (${e.capacite} places). Videz-le avant d\'acheter $article.');
    }
  }

  print('Sac final : ${boutique.sac}');
}
```

**Résultat :**

```text
Vous achetez épée. Or restant : 70
Trop cher pour armure : il vous manque 430 pièces.
Vous achetez bouclier. Or restant : 40
Sac plein (3 places). Videz-le avant d'acheter casque.
Sac final : [potion, épée, bouclier]
```

Observez la ligne `il vous manque ${e.manque} pièces`. Le message affiché au joueur est **construit par l'interface**, à partir de données brutes fournies par l'exception. C'est le bon découpage : la classe métier ne décide pas de la formulation.

**Que se passe-t-il si vous oubliez `toString()` ?**

```dart
class SauvegardeCorrompueException implements Exception {
  final String fichier;
  SauvegardeCorrompueException(this.fichier);
}

void main() {
  throw SauvegardeCorrompueException('partie1.sav');
}
```

**Résultat :**

```text
Unhandled exception:
Instance of 'SauvegardeCorrompueException'
#0      main (file:///jeu/main.dart:7:3)
```

`Instance of '...'` est l'affichage par défaut de tout objet Dart. Il ne donne aucune information. **Redéfinissez toujours `toString()`** sur vos exceptions.

Avec le `toString()` :

```dart
class SauvegardeCorrompueException implements Exception {
  final String fichier;
  final String raison;

  SauvegardeCorrompueException(this.fichier, this.raison);

  @override
  String toString() => 'Sauvegarde "$fichier" illisible : $raison';
}

void main() {
  try {
    throw SauvegardeCorrompueException('partie1.sav', 'niveau non numérique');
  } on SauvegardeCorrompueException catch (e) {
    print('Message  : $e');
    print('Fichier  : ${e.fichier}');
    print('Raison   : ${e.raison}');
  }
}
```

**Résultat :**

```text
Message  : Sauvegarde "partie1.sav" illisible : niveau non numérique
Fichier  : partie1.sav
Raison   : niveau non numérique
```

> **`implements` et non `extends` :** `Exception` est une interface, pas une classe avec du comportement à hériter. La convention Dart est `implements Exception`. Vous pouvez en revanche `extends` une de **vos** exceptions pour créer une famille :

```dart
abstract class JeuException implements Exception {
  final String message;
  JeuException(this.message);

  @override
  String toString() => '$runtimeType : $message';
}

class SauvegardeException extends JeuException {
  SauvegardeException(super.message);
}

class BoutiqueException extends JeuException {
  BoutiqueException(super.message);
}

void main() {
  final List<JeuException> incidents = [
    SauvegardeException('fichier introuvable'),
    BoutiqueException('or insuffisant'),
  ];

  for (final JeuException incident in incidents) {
    try {
      throw incident;
    } on JeuException catch (e) {
      print('Incident de jeu -> $e');
    }
  }
}
```

**Résultat :**

```text
Incident de jeu -> SauvegardeException : fichier introuvable
Incident de jeu -> BoutiqueException : or insuffisant
```

Un seul `on JeuException` attrape toute la famille. C'est la raison d'être d'une classe de base commune.

---

## 13.15 — `FormatException`

`FormatException` est l'exception standard qui signifie : **« ce texte n'a pas le format attendu »**. C'est de loin celle que vous rencontrerez le plus.

Elle est lancée par `int.parse`, `double.parse`, `DateTime.parse`, `Uri.parse`, `jsonDecode` (chapitre 17), et vous pouvez la lancer vous-même.

Elle possède trois propriétés :

| Propriété | Type | Contenu |
| --- | --- | --- |
| `message` | `dynamic` | la description du problème |
| `source` | `dynamic` | le texte qui a posé problème (peut être `null`) |
| `offset` | `int?` | la position du caractère fautif, comptée à partir de 0 |

Son `toString()` compose ces trois éléments :

```dart
void main() {
  final e1 = FormatException('Pseudo invalide');
  final e2 = FormatException('Pseudo invalide', 'Al ex#');
  final e3 = FormatException('Pseudo invalide', 'Al ex#', 5);

  print('--- 1 ---');
  print(e1);
  print('--- 2 ---');
  print(e2);
  print('--- 3 ---');
  print(e3);
}
```

**Résultat :**

```text
--- 1 ---
FormatException: Pseudo invalide
--- 2 ---
FormatException: Pseudo invalide
Al ex#
--- 3 ---
FormatException: Pseudo invalide (at character 6)
Al ex#
     ^
```

Le troisième affichage est celui que vous connaissez déjà : le message, la position humaine (`offset + 1`), le texte, et un accent circonflexe sous le caractère fautif.

Vous pouvez lancer une `FormatException` dans votre propre code de validation. C'est un bon réflexe : plutôt que d'inventer une classe, réutilisez celle du langage quand le problème est bien « un texte mal formé ».

```dart
String validerPseudo(String saisie) {
  for (int i = 0; i < saisie.length; i++) {
    final String c = saisie[i];
    final bool lettre = (c.compareTo('a') >= 0 && c.compareTo('z') <= 0) ||
        (c.compareTo('A') >= 0 && c.compareTo('Z') <= 0);
    final bool chiffre = c.compareTo('0') >= 0 && c.compareTo('9') <= 0;
    if (!lettre && !chiffre) {
      throw FormatException('Caractère non autorisé dans le pseudo', saisie, i);
    }
  }
  return saisie;
}

void main() {
  for (final String saisie in ['Alex42', 'Al ex#']) {
    try {
      print('Pseudo accepté : ${validerPseudo(saisie)}');
    } on FormatException catch (e) {
      print('Pseudo refusé :');
      print(e);
    }
  }
}
```

**Résultat :**

```text
Pseudo accepté : Alex42
Pseudo refusé :
FormatException: Caractère non autorisé dans le pseudo (at character 3)
Al ex#
  ^
```

Le joueur voit exactement quel caractère pose problème. C'est ce niveau de précision qui distingue un message d'erreur utile d'un « saisie invalide » inutile.

---

## 13.16 — `int.parse()` qui échoue

Reprenons `int.parse` en détail, car c'est la source d'exception numéro un chez les débutants.

`int.parse(String)` retourne un `int`. Il n'a **aucun** moyen de signaler un échec par sa valeur de retour : tous les `int` sont des résultats valides. Il ne lui reste que l'exception.

Ce qu'il accepte :

```dart
void main() {
  print(int.parse('42'));
  print(int.parse('-7'));
  print(int.parse('+7'));
  print(int.parse('007'));
  print(int.parse('  42  '));
  print(int.parse('ff', radix: 16));
  print(int.parse('1010', radix: 2));
}
```

**Résultat :**

```text
42
-7
7
7
42
255
10
```

Notez que les espaces de début et de fin sont tolérés, que `007` vaut bien `7`, et que `radix` permet de lire de l'hexadécimal ou du binaire.

Ce qu'il refuse :

| Saisie | Pourquoi cela échoue |
| --- | --- |
| `'sept'` | aucun chiffre |
| `'12.5'` | le point n'est pas un chiffre — c'est un `double`, pas un `int` |
| `'1 000'` | l'espace au milieu n'est pas toléré |
| `'12€'` | caractère non numérique |
| `''` | rien à lire |
| `'1,5'` | la virgule n'est pas reconnue, même en français |

Regardons comment le message situe l'erreur :

```dart
void main() {
  for (final String saisie in ['sept', '12.5', '1 000', '12€']) {
    print('--- "$saisie" ---');
    try {
      print(int.parse(saisie));
    } on FormatException catch (e) {
      print('message : ${e.message}');
      print('source  : ${e.source}');
      print('offset  : ${e.offset}');
    }
  }
}
```

**Résultat :**

```text
--- "sept" ---
message : Invalid radix-10 number
source  : sept
offset  : 0
--- "12.5" ---
message : Invalid radix-10 number
source  : 12.5
offset  : 2
--- "1 000" ---
message : Invalid radix-10 number
source  : 1 000
offset  : 1
--- "12€" ---
message : Invalid radix-10 number
source  : 12€
offset  : 2
```

L'`offset` est l'index du **premier caractère qui ne va pas**. Pour `'12.5'`, les caractères `1` et `2` sont valides ; le point, à l'index 2, ne l'est pas.

La forme protégée, celle que vous écrirez en attendant la section 13.17 :

```dart
int chargerNiveau(String brut) {
  try {
    return int.parse(brut);
  } on FormatException {
    print('Niveau illisible ("$brut"), retour au niveau 1.');
    return 1;
  }
}

void main() {
  print('Niveau : ${chargerNiveau('7')}');
  print('Niveau : ${chargerNiveau('sept')}');
}
```

**Résultat :**

```text
Niveau : 7
Niveau illisible ("sept"), retour au niveau 1.
Niveau : 1
```

Ce code fonctionne. Il est pourtant, dans ce cas précis, la **mauvaise** solution. Voyons pourquoi.

---

## 13.17 — `int.tryParse()` : éviter l'exception plutôt que l'attraper

Une exception est un mécanisme lourd. Elle interrompt le flux, construit une trace de pile, remonte les fonctions. Utilisée pour une saisie utilisateur — événement parfaitement ordinaire — elle est disproportionnée.

Dart fournit `int.tryParse`. Signature :

```text
  int   int.parse(String source)      -> lance FormatException si échec
  int?  int.tryParse(String source)   -> retourne null si échec
```

Le `?` du type de retour, c'est le chapitre 12 qui revient. `tryParse` transforme un problème d'exception en un problème de **nullité**, que vous savez déjà traiter avec `??`, `?.` et un simple `if`.

```dart
void main() {
  print(int.tryParse('42'));
  print(int.tryParse('sept'));
  print(int.tryParse('12.5'));
  print(int.tryParse(''));
  print(int.tryParse('ff', radix: 16));
}
```

**Résultat :**

```text
42
null
null
null
255
```

Aucune exception. Aucun `try`. Le programme ne s'interrompt jamais.

La même fonction que la section précédente, réécrite :

```dart
int chargerNiveau(String brut) {
  return int.tryParse(brut) ?? 1;
}

void main() {
  print('Niveau : ${chargerNiveau('7')}');
  print('Niveau : ${chargerNiveau('sept')}');
  print('Niveau : ${chargerNiveau('')}');
}
```

**Résultat :**

```text
Niveau : 7
Niveau : 1
Niveau : 1
```

Une ligne au lieu de six. C'est la version à retenir.

Quand vous avez besoin de distinguer « valeur absente » de « valeur valide », utilisez la promotion de type vue au chapitre 12 :

```dart
void definirNiveau(String saisie) {
  final int? niveau = int.tryParse(saisie);

  if (niveau == null) {
    print('"$saisie" n\'est pas un nombre.');
    return;
  }
  if (niveau < 1 || niveau > 50) {
    print('Le niveau doit être compris entre 1 et 50 (reçu : $niveau).');
    return;
  }

  print('Niveau réglé sur $niveau.');
}

void main() {
  definirNiveau('12');
  definirNiveau('douze');
  definirNiveau('99');
}
```

**Résultat :**

```text
Niveau réglé sur 12.
"douze" n'est pas un nombre.
Le niveau doit être compris entre 1 et 50 (reçu : 99).
```

Après le premier `if`, `niveau` est promu de `int?` à `int` : on peut le comparer sans `!` et sans `??`.

Le principe général, qui dépasse largement `int.parse` :

> Quand une bibliothèque propose une variante `tryXxx` qui retourne `null`, **préférez-la** à la variante qui lance. On n'utilise `try` / `catch` que lorsqu'il n'existe aucun moyen de tester à l'avance.

Un tableau de décision :

| Situation | Bon outil |
| --- | --- |
| Convertir une saisie utilisateur | `int.tryParse` / `double.tryParse` |
| Lire une clé de `Map` qui peut manquer | `map['cle'] ?? valeurParDefaut` |
| Vérifier un indice de liste | `if (i >= 0 && i < liste.length)` |
| Lire un fichier qui peut être absent | `try` / `catch` (aucun test préalable fiable) |
| Analyser un JSON reçu du réseau | `try` / `catch` |

---

## 13.18 — `double.tryParse()`

Le principe est identique pour les nombres à virgule.

```text
  double  double.parse(String source)      -> lance FormatException
  double? double.tryParse(String source)   -> retourne null
```

```dart
void main() {
  print(double.tryParse('87.5'));
  print(double.tryParse('87'));
  print(double.tryParse('-3.25'));
  print(double.tryParse('1e3'));
  print(double.tryParse('87,5'));
  print(double.tryParse('quatre-vingts'));
  print(double.tryParse(''));
}
```

**Résultat :**

```text
87.5
87.0
-3.25
1000.0
null
null
null
```

Trois remarques.

**1. Le séparateur décimal est le point, pas la virgule.** C'est le piège numéro un des francophones. `'87,5'` retourne `null`. Si vos joueurs saisissent une virgule, convertissez-la avant :

```dart
double? lireDecimal(String saisie) {
  return double.tryParse(saisie.trim().replaceAll(',', '.'));
}

void main() {
  print(lireDecimal('87,5'));
  print(lireDecimal(' 87.5 '));
  print(lireDecimal('abc'));
}
```

**Résultat :**

```text
87.5
87.5
null
```

**2. `double.tryParse('87')` retourne `87.0`, pas `87`.** Un `int` est accepté et converti en `double`. C'est utile : une énergie saisie « 87 » reste exploitable.

**3. Attention aux valeurs spéciales.** `double` connaît l'infini et « pas un nombre » :

```dart
void main() {
  final double? a = double.tryParse('Infinity');
  final double? b = double.tryParse('NaN');
  print('a = $a');
  print('b = $b');
  print('énergie plafonnée : ${(a ?? 0).clamp(0, 100)}');
}
```

**Résultat :**

```text
a = Infinity
b = NaN
énergie plafonnée : 100
```

`tryParse` ne retourne pas `null` sur ces textes : ce sont des `double` valides pour Dart. Si votre jeu ne doit jamais recevoir une énergie infinie, testez-le explicitement :

```dart
double? energieValide(String saisie) {
  final double? v = double.tryParse(saisie);
  if (v == null) return null;
  if (!v.isFinite) return null;
  if (v < 0 || v > 100) return null;
  return v;
}

void main() {
  for (final String s in ['87.5', 'Infinity', 'NaN', '250', 'abc']) {
    print('$s -> ${energieValide(s)}');
  }
}
```

**Résultat :**

```text
87.5 -> 87.5
Infinity -> null
NaN -> null
250 -> null
abc -> null
```

`isFinite` est `false` pour `Infinity`, `-Infinity` et `NaN` : un seul test suffit à écarter les trois.

---

## 13.19 — Valider une saisie utilisateur

Nous avons maintenant tous les outils. Assemblons-les sur le cas réel : le joueur saisit son pseudo, puis son niveau de départ.

> **Note sur la saisie :** la lecture du clavier se fait normalement avec `stdin.readLineSync()` de la bibliothèque `dart:io`. DartPad n'a pas de clavier interactif. Nous simulons donc les saisies par une liste de textes. Le code de validation, lui, est exactement celui que vous écririez en vrai.

Une validation propre suit toujours les mêmes cinq étapes :

```text
  ┌──────────────────────────────────────────────────────────┐
  │  1. NETTOYER    trim(), retirer les espaces inutiles       │
  │  2. NON VIDE    refuser une chaîne vide                    │
  │  3. CONVERTIR   tryParse -> valeur ou null                 │
  │  4. BORNER      vérifier les limites métier (1..50)        │
  │  5. ACCEPTER    la valeur est utilisable partout ensuite   │
  └──────────────────────────────────────────────────────────┘
```

Version complète :

```dart
class SaisieInvalideException implements Exception {
  final String champ;
  final String raison;

  SaisieInvalideException(this.champ, this.raison);

  @override
  String toString() => '$champ : $raison';
}

String validerPseudo(String saisie) {
  final String p = saisie.trim();

  if (p.isEmpty) {
    throw SaisieInvalideException('Pseudo', 'le pseudo ne peut pas être vide');
  }
  if (p.length < 3) {
    throw SaisieInvalideException('Pseudo', '3 caractères minimum (reçu : ${p.length})');
  }
  if (p.length > 12) {
    throw SaisieInvalideException('Pseudo', '12 caractères maximum (reçu : ${p.length})');
  }
  if (p.contains(' ')) {
    throw SaisieInvalideException('Pseudo', 'les espaces sont interdits');
  }
  return p;
}

int validerNiveau(String saisie) {
  final int? n = int.tryParse(saisie.trim());

  if (n == null) {
    throw SaisieInvalideException('Niveau', '"${saisie.trim()}" n\'est pas un nombre entier');
  }
  if (n < 1 || n > 50) {
    throw SaisieInvalideException('Niveau', 'valeur attendue entre 1 et 50 (reçu : $n)');
  }
  return n;
}

void inscrire(String pseudoBrut, String niveauBrut) {
  try {
    final String pseudo = validerPseudo(pseudoBrut);
    final int niveau = validerNiveau(niveauBrut);
    print('OK  -> $pseudo entre dans le donjon au niveau $niveau.');
  } on SaisieInvalideException catch (e) {
    print('KO  -> $e');
  }
}

void main() {
  inscrire('  Alex  ', '12');
  inscrire('', '12');
  inscrire('Al', '12');
  inscrire('Alexandre-le-Grand', '12');
  inscrire('Al ex', '12');
  inscrire('Alex', 'douze');
  inscrire('Alex', '99');
}
```

**Résultat :**

```text
OK  -> Alex entre dans le donjon au niveau 12.
KO  -> Pseudo : le pseudo ne peut pas être vide
KO  -> Pseudo : 3 caractères minimum (reçu : 2)
KO  -> Pseudo : 12 caractères maximum (reçu : 18)
KO  -> Pseudo : les espaces sont interdits
KO  -> Niveau : "douze" n'est pas un nombre entier
KO  -> Niveau : valeur attendue entre 1 et 50 (reçu : 99)
```

Analysons les choix de conception, car ils sont représentatifs de ce qu'on fait en production.

- **La validation lance, l'appelant attrape.** `validerPseudo` ne fait aucun `print`. Elle ne sait pas si le programme est une console, une application Flutter ou un serveur. Elle signale, c'est tout.
- **Une exception métier unique**, `SaisieInvalideException`, avec deux champs. Un seul `on` suffit à traiter tous les cas, tout en gardant un message précis.
- **`int.tryParse` à l'intérieur, exception à l'extérieur.** On n'attrape pas de `FormatException` : on l'évite, puis on lance notre propre exception, plus parlante.
- **Le `trim()` est fait une seule fois**, au début, et la valeur nettoyée est celle qui est retournée. Le reste du programme ne verra jamais d'espaces parasites.

Une variante fréquente consiste à collecter **toutes** les erreurs au lieu de s'arrêter à la première :

```dart
List<String> erreursPseudo(String saisie) {
  final String p = saisie.trim();
  final List<String> erreurs = [];

  if (p.isEmpty) erreurs.add('le pseudo est vide');
  if (p.length > 12) erreurs.add('12 caractères maximum');
  if (p.contains(' ')) erreurs.add('les espaces sont interdits');
  if (p.contains('#')) erreurs.add('le caractère # est interdit');

  return erreurs;
}

void main() {
  final List<String> problemes = erreursPseudo('Al ex#trop#long#ici');

  if (problemes.isEmpty) {
    print('Pseudo valide.');
  } else {
    print('Pseudo refusé pour ${problemes.length} raisons :');
    for (final String p in problemes) {
      print('  - $p');
    }
  }
}
```

**Résultat :**

```text
Pseudo refusé pour 3 raisons :
  - 12 caractères maximum
  - les espaces sont interdits
  - le caractère # est interdit
```

Cette forme, sans exception du tout, est souvent préférable pour un formulaire : elle évite au joueur de corriger ses erreurs une par une.

---

## 13.20 — `ArgumentError` et `assert`

Les sections précédentes traitaient de données venues de l'extérieur (fichier, joueur, réseau). Voyons maintenant comment se protéger d'un appel **fautif** venu de votre propre code.

### `ArgumentError`

`ArgumentError` est une `Error`. Elle dit : « l'appelant m'a passé un argument que je n'accepte pas ; c'est un bug ».

```dart
class Joueur {
  final String nom;
  int vies;

  Joueur(this.nom, this.vies);

  void perdreVies(int nombre) {
    if (nombre <= 0) {
      throw ArgumentError('Le nombre de vies perdues doit être positif');
    }
    vies -= nombre;
  }
}

void main() {
  final Joueur j = Joueur('Alex', 3);
  j.perdreVies(1);
  print('${j.nom} : ${j.vies} vies');
  j.perdreVies(0);
}
```

**Résultat :**

```text
Alex : 2 vies
Unhandled exception:
Invalid argument(s): Le nombre de vies perdues doit être positif
#0      Joueur.perdreVies (file:///jeu/main.dart:9:7)
#1      main (file:///jeu/main.dart:19:5)
```

Notez le `toString()` : `Invalid argument(s): ...`. Il ne contient pas le mot « ArgumentError ».

Trois constructeurs utiles :

```dart
void main() {
  print(ArgumentError('message libre'));
  print(ArgumentError.value(-5, 'prix', 'ne peut pas être négatif'));
  print(ArgumentError.notNull('pseudo'));
  print(RangeError.range(99, 1, 50, 'niveau'));
}
```

**Résultat :**

```text
Invalid argument(s): message libre
Invalid argument (prix): ne peut pas être négatif: -5
Invalid argument(s): Must not be null: pseudo
RangeError (niveau): Invalid value: Not in inclusive range 1..50: 99
```

`ArgumentError.value(valeur, nomDuParametre, explication)` est la forme la plus informative : elle affiche à la fois le paramètre concerné et la valeur reçue. Utilisez-la.

```dart
void definirPrix(int prix) {
  if (prix < 0) {
    throw ArgumentError.value(prix, 'prix', 'Le prix doit être positif ou nul');
  }
  print('Prix accepté : $prix');
}

void main() {
  definirPrix(50);
  definirPrix(-10);
}
```

**Résultat :**

```text
Prix accepté : 50
Unhandled exception:
Invalid argument (prix): Le prix doit être positif ou nul: -10
#0      definirPrix (file:///jeu/main.dart:3:5)
#1      main (file:///jeu/main.dart:11:3)
```

> **Question fréquente : faut-il attraper un `ArgumentError` ?** Non. C'est une `Error` : elle signale que votre code appelle mal une fonction. La correction se fait dans l'éditeur, pas dans un `catch`.

### `assert`

`assert(condition, 'message')` vérifie une condition qui doit **toujours** être vraie.

Sa particularité : `assert` n'est actif qu'en **mode développement**. En production (`dart compile exe`, build Flutter en mode *release*), les `assert` sont purement et simplement retirés du programme. Ils ne coûtent alors rien.

```dart
class Potion {
  final String nom;
  final int soin;

  Potion(this.nom, this.soin) {
    assert(soin > 0, 'Une potion doit soigner : soin = $soin');
    assert(nom.isNotEmpty, 'Une potion doit avoir un nom');
  }
}

void main() {
  final Potion p = Potion('Potion mineure', 25);
  print('${p.nom} soigne ${p.soin} PV');

  final Potion mauvaise = Potion('Potion cassée', -5);
  print(mauvaise.nom);
}
```

**Résultat :**

```text
Potion mineure soigne 25 PV
Unhandled exception:
'file:///jeu/main.dart': Failed assertion: line 6 pos 12: 'soin > 0': Une potion doit soigner : soin = -5
#0      _AssertionError._doThrowNew (dart:core-patch/errors_patch.dart:51:61)
#1      _AssertionError._throwNew (dart:core-patch/errors_patch.dart:40:5)
#2      new Potion (file:///jeu/main.dart:6:12)
#3      main (file:///jeu/main.dart:16:26)
```

Le message donne la ligne, la condition littérale (`'soin > 0'`) et votre explication. C'est très efficace pendant le développement.

Le tableau de décision entre les trois outils :

| Outil | Actif en production | Pour quoi |
| --- | --- | --- |
| `assert` | non | invariant interne, aide au développement |
| `ArgumentError` | oui | argument invalide passé par du code appelant |
| `Exception` métier | oui | donnée externe invalide (joueur, fichier, réseau) |

Une règle simple, appliquée au jeu :

```dart
class Boutique {
  final Map<String, int> catalogue;

  Boutique(this.catalogue) {
    // Invariant : jamais un prix négatif dans le catalogue.
    assert(catalogue.values.every((p) => p >= 0), 'Prix négatif dans le catalogue');
  }

  int prix(String article) {
    // Argument fourni par le code appelant : ArgumentError.
    if (!catalogue.containsKey(article)) {
      throw ArgumentError.value(article, 'article', 'Article inconnu');
    }
    return catalogue[article]!;
  }

  void acheter(String article, int or) {
    final int p = prix(article);
    // Donnée liée au joueur : exception métier.
    if (or < p) {
      throw Exception('Or insuffisant : $or / $p');
    }
    print('Achat de $article effectué.');
  }
}

void main() {
  final Boutique b = Boutique({'potion': 20, 'épée': 80});

  b.acheter('potion', 100);

  try {
    b.acheter('épée', 30);
  } on Exception catch (e) {
    print('Refusé : $e');
  }
}
```

**Résultat :**

```text
Achat de potion effectué.
Refusé : Exception: Or insuffisant : 30 / 80
```

Notez le `catalogue[article]!` de la méthode `prix` : le `!` est ici **justifié**, car la ligne précédente a vérifié `containsKey`. C'est l'un des rares cas légitimes vus au chapitre 12.

---

## 13.21 — Exceptions et null safety

Le chapitre 12 et le chapitre 13 traitent du même problème — « la valeur attendue n'est pas là » — avec deux mécanismes différents. Il faut savoir lequel choisir.

```text
  ┌────────────────────────────────────────────────────────────────┐
  │  L'absence est NORMALE et fréquente                            │
  │      -> type nullable, ?? , ?. , tryParse                      │
  │      exemple : le joueur n'a pas d'arme équipée                │
  ├────────────────────────────────────────────────────────────────┤
  │  L'absence est ANORMALE et empêche de continuer                │
  │      -> exception                                              │
  │      exemple : le fichier de sauvegarde est illisible          │
  └────────────────────────────────────────────────────────────────┘
```

Premier point de contact : l'opérateur `!` du chapitre 12 **lance une exception à l'exécution**.

```dart
void main() {
  final Map<String, int> sauvegarde = {'niveau': 7};
  print('Niveau : ${sauvegarde['niveau']!}');
  print('Or     : ${sauvegarde['or']!}');
}
```

**Résultat :**

```text
Niveau : 7
Unhandled exception:
Null check operator used on a null value
#0      main (file:///jeu/main.dart:4:37)
```

Ce message, `Null check operator used on a null value`, est une `TypeError`, donc une **`Error`**. Conséquence directe : un `on Exception` ne l'attrape pas.

```dart
void main() {
  final Map<String, int> sauvegarde = {'niveau': 7};

  try {
    print(sauvegarde['or']!);
  } on Exception catch (e) {
    print('Attrapé comme Exception : $e');
  }
  print('Fin');
}
```

**Résultat :**

```text
Unhandled exception:
Null check operator used on a null value
#0      main (file:///jeu/main.dart:5:32)
```

Le `catch` ne se déclenche pas et le programme plante. C'est cohérent avec 13.2 : un `!` qui échoue est un **bug**, pas un accident.

La bonne écriture n'utilise ni `!` ni `try` :

```dart
void main() {
  final Map<String, int> sauvegarde = {'niveau': 7};
  final int niveau = sauvegarde['niveau'] ?? 1;
  final int or = sauvegarde['or'] ?? 0;
  print('Niveau : $niveau | Or : $or');
}
```

**Résultat :**

```text
Niveau : 7 | Or : 0
```

Deuxième point de contact : une exception attrapée peut servir à **produire** une valeur de repli, et donc à revenir dans le monde non nullable.

```dart
class Joueur {
  final String pseudo;
  final int niveau;
  Joueur(this.pseudo, this.niveau);

  @override
  String toString() => '$pseudo (niv. $niveau)';
}

Joueur chargerJoueur(String ligne) {
  final List<String> champs = ligne.split(';');
  if (champs.length != 2) {
    throw FormatException('Deux champs attendus', ligne);
  }
  final String pseudo = champs[0].trim();
  final int? niveau = int.tryParse(champs[1].trim());
  if (pseudo.isEmpty) {
    throw FormatException('Pseudo vide', ligne);
  }
  if (niveau == null) {
    throw FormatException('Niveau non numérique', ligne, ligne.indexOf(';') + 1);
  }
  return Joueur(pseudo, niveau);
}

Joueur chargerOuNouveau(String ligne) {
  try {
    return chargerJoueur(ligne);
  } on FormatException catch (e) {
    print('  [journal] $e');
    return Joueur('Invité', 1);
  }
}

void main() {
  for (final String l in ['Alex;7', 'Alex;sept', ';7', 'Alex']) {
    print('--- "$l" ---');
    final Joueur j = chargerOuNouveau(l);
    print('  Joueur actif : $j');
  }
}
```

**Résultat :**

```text
--- "Alex;7" ---
  Joueur actif : Alex (niv. 7)
--- "Alex;sept" ---
  [journal] FormatException: Niveau non numérique (at character 6)
Alex;sept
     ^
  Joueur actif : Invité (niv. 1)
--- ";7" ---
  [journal] FormatException: Pseudo vide
;7
  Joueur actif : Invité (niv. 1)
--- "Alex" ---
  [journal] FormatException: Deux champs attendus
Alex
  Joueur actif : Invité (niv. 1)
```

Le point essentiel : `chargerOuNouveau` retourne un `Joueur`, **jamais** un `Joueur?`. Tout le reste du jeu manipule un joueur garanti non nul. C'est le principe « pousser le `null` vers les bords », déjà rencontré au chapitre 12, appliqué cette fois aux exceptions.

Troisième point de contact : ne remplacez pas une exception par un `null` quand l'appelant a besoin de savoir **pourquoi**.

| Signature | Ce que l'appelant apprend |
| --- | --- |
| `int? lireNiveau(String s)` | ça a échoué |
| `int lireNiveau(String s)` avec `throw` | ça a échoué, et pour quelle raison exacte |

Pour une conversion triviale, `null` suffit. Pour un chargement de sauvegarde à cinq causes d'échec possibles, une exception typée est bien meilleure.

---

## 13.22 — Bonnes pratiques : quand attraper, quand laisser planter

Voici la synthèse opérationnelle du chapitre. Ces règles valent aussi bien en Dart console qu'en Flutter.

### Règle 1 — Attrapez ce que vous savez traiter

Un `catch` doit produire une **action** : message au joueur, valeur de repli, nouvelle tentative, journalisation. Un `catch` qui ne fait rien est une faute.

```dart
// À NE JAMAIS FAIRE
void main() {
  try {
    int.parse('sept');
  } catch (e) {
    // silence
  }
  print('Tout va bien... en apparence.');
}
```

**Résultat :**

```text
Tout va bien... en apparence.
```

Le bug a disparu de l'écran, pas du programme. Vous le retrouverez plus tard, sous une forme incompréhensible.

### Règle 2 — Laissez planter les `Error`

`RangeError`, `StateError`, `TypeError`, `AssertionError`, `Null check operator used on a null value` : ce sont des bugs. Un plantage franc pendant le développement vaut mieux qu'un comportement faux en production.

### Règle 3 — Attrapez au bon niveau

L'endroit où l'exception naît n'est presque jamais l'endroit où l'on sait quoi faire.

```text
  int.parse           <- naissance de l'exception
     ↑
  lireNiveau          <- ne sait pas quoi faire : laisse remonter
     ↑
  chargerSauvegarde   <- journalise, puis rethrow
     ↑
  menuPrincipal       <- SAIT quoi faire : propose une nouvelle partie
```

### Règle 4 — Ne filtrez jamais sur le texte du message

```dart
// À NE JAMAIS FAIRE
void main() {
  try {
    int.parse('sept');
  } catch (e) {
    if (e.toString().contains('Invalid radix-10')) {
      print('Nombre invalide');
    }
  }
}
```

Le message peut changer d'une version de Dart à l'autre, et il n'est pas traduit. Filtrez sur le **type** : `on FormatException`.

### Règle 5 — Préférez `tryParse` à `try` / `catch`

Vu en 13.17. Une exception coûte cher ; une saisie invalide n'a rien d'exceptionnel.

### Règle 6 — `rethrow`, jamais `throw e`

Vu en 13.13. `throw e` détruit la trace d'origine.

### Règle 7 — Nommez vos exceptions d'après le domaine

`OrInsuffisantException` vaut mieux que `Exception('erreur 42')`. Le nom est lu par le prochain développeur — souvent vous, dans six mois.

### Règle 8 — Un `try` court

Entourez une ou deux instructions, pas cinquante. Sinon vous ne savez plus laquelle a échoué, et vous risquez d'attraper une exception que vous n'aviez pas prévue.

### Règle 9 — `finally` pour les ressources

Tout ce qui est ouvert doit être fermé, quelle que soit l'issue. Et jamais de `return` dans un `finally`.

### Règle 10 — Le joueur ne doit jamais lire une trace de pile

Séparez systématiquement le message technique (journal, développeur) du message humain (interface, joueur).

```dart
class OrInsuffisantException implements Exception {
  final int or;
  final int prix;
  OrInsuffisantException(this.or, this.prix);

  @override
  String toString() => 'OrInsuffisantException(or: $or, prix: $prix)';

  String get messageJoueur => 'Il vous manque ${prix - or} pièces d\'or.';
}

void main() {
  try {
    throw OrInsuffisantException(30, 80);
  } on OrInsuffisantException catch (e) {
    print('[journal développeur] $e');
    print('[écran du joueur]     ${e.messageJoueur}');
  }
}
```

**Résultat :**

```text
[journal développeur] OrInsuffisantException(or: 30, prix: 80)
[écran du joueur]     Il vous manque 50 pièces d'or.
```

### Checklist avant de valider votre code

```text
  [ ] Chaque catch fait quelque chose d'utile
  [ ] Aucun catch vide
  [ ] Aucune Error attrapée sans très bonne raison
  [ ] Les clauses on vont du plus précis au plus général
  [ ] tryParse utilisé partout où c'est possible
  [ ] rethrow utilisé à la place de throw e
  [ ] Aucun return dans un finally
  [ ] Les exceptions métier ont un toString() lisible
  [ ] Le joueur ne voit jamais un message technique brut
```

> **Et l'asynchrone ?** Un appel réseau ou une lecture de fichier réelle échouent aussi, mais leur code s'écrit avec `Future`, `async` et `await`. La syntaxe `try` / `catch` y fonctionne de la même manière, avec quelques précautions supplémentaires. Nous verrons cela au chapitre 15.

---

## 13.23 — Erreurs fréquentes

| Erreur | Cause | Correction |
| --- | --- | --- |
| `Unhandled exception: FormatException: Invalid radix-10 number (at character 1)` | `int.parse()` reçoit un texte qui n'est pas un entier. | `int.tryParse(texte) ?? 0`, ou entourez d'un `try` / `on FormatException`. |
| `Unhandled exception: FormatException: Invalid double` | `double.parse()` reçoit une virgule française ou du texte. | `double.tryParse(texte.replaceAll(',', '.'))`. |
| `Error: A try block must be followed by an 'on', 'catch', or 'finally' clause.` | Un `try` écrit seul, sans aucune clause. | Ajoutez au moins un `catch`, un `on` ou un `finally`. |
| `Error: Undefined name 'niveau'.` | Vous lisez, après le bloc, une variable déclarée **dans** le `try`. | Déclarez la variable avant le `try`, avec une valeur par défaut. |
| `Error: The getter 'message' isn't defined for the class 'Object'.` | Vous lisez une propriété d'exception après un `catch (e)` non typé. | Utilisez `on FormatException catch (e)` pour que `e` soit typé. |
| `Dead code: this on-catch block will never be executed because 'FormatException' is a subtype of 'Exception' and hence will have been caught already.` | Vous avez mis `on Exception` **avant** `on FormatException`. | Rangez les clauses du plus précis au plus général. |
| `Dead code: catch clauses after a 'catch (e)' or an 'on Object catch' clause are never reached.` | Une clause `on` est écrite après le `catch (e)` fourre-tout. | Le `catch (e)` sans type doit toujours être le dernier. |
| `Error: 'rethrow' can only be used in catch clauses.` | `rethrow` écrit en dehors d'un `catch`. | Utilisez `throw` pour lancer une nouvelle exception, `rethrow` seulement dans un `catch`. |
| `Error: Can't throw a value of 'Null' as an exception.` | `throw null;` ou `throw variableNullable;`. | Lancez un objet réel : `throw Exception('...')`. |
| `Unhandled exception: Instance of 'SauvegardeCorrompueException'` | Votre exception personnalisée n'a pas de `toString()`. | Ajoutez `@override String toString() => '...';`. |
| `Unhandled exception: Null check operator used on a null value` | Un `!` posé sur une valeur qui valait `null`. C'est une `Error`, donc `on Exception` ne l'attrape pas. | Remplacez le `!` par `??` ou par un `if` de vérification. |
| `Unhandled exception: RangeError (index): Invalid value: Not in inclusive range 0..1: 5` | Indice hors des bornes d'une liste. C'est un bug, pas un accident. | Vérifiez `index >= 0 && index < liste.length` avant de lire. |
| `Unhandled exception: Bad state: No element` | `firstWhere` n'a trouvé aucun élément, ou `first` sur une liste vide. | Ajoutez `orElse: () => valeurParDefaut`, ou testez `isNotEmpty`. |
| `Unhandled exception: Invalid argument(s): Le prix doit être positif` | Vous avez lancé un `ArgumentError` : le code appelant passe une valeur interdite. | Corrigez l'appel. N'attrapez pas : c'est une `Error`. |
| `Unhandled exception: Invalid argument (prix): Le prix doit être positif ou nul: -10` | Même chose avec `ArgumentError.value`. | Le message indique le paramètre et la valeur reçue : corrigez l'appelant. |
| `Unhandled exception: Unsupported operation: Result of truncating division is Infinity: 10 ~/ 0` | Division entière par zéro. | Testez le diviseur avant : `if (n != 0)`. |
| `'file:///jeu/main.dart': Failed assertion: line 6 pos 12: 'soin > 0': Une potion doit soigner` | Un `assert` a échoué en mode développement. | Corrigez la valeur passée : un `assert` signale un invariant cassé. |
| `Unhandled exception: Exception: Or insuffisant` | Une exception métier n'a été attrapée nulle part. | Ajoutez un `try` / `on` au niveau qui sait réagir (le menu, en général). |
| Le programme continue alors qu'il devrait s'arrêter | `catch (e) { }` vide : l'exception est avalée en silence. | Un `catch` doit toujours agir : message, valeur de repli, ou `rethrow`. |
| La trace pointe sur votre `catch`, pas sur la vraie cause | Vous avez écrit `throw e;` au lieu de `rethrow;`. | Remplacez par `rethrow;`. |

---

## 13.24 — Résumé du chapitre

| Mot-clé / outil | Rôle | Exemple |
| --- | --- | --- |
| `try` | délimite le code surveillé | `try { ... }` |
| `catch (e)` | attrape n'importe quel objet lancé | `catch (e) { print(e); }` |
| `catch (e, s)` | attrape l'objet **et** la trace de pile | `catch (e, s) { print(s); }` |
| `on Type` | attrape uniquement un type précis | `on FormatException { ... }` |
| `on Type catch (e)` | filtre par type et donne accès à l'objet typé | `on FormatException catch (e) { print(e.offset); }` |
| `finally` | bloc de nettoyage toujours exécuté | `finally { fermer(); }` |
| `throw` | lance une exception | `throw Exception('Or insuffisant');` |
| `rethrow` | relance l'exception courante sans perdre la trace | `catch (e) { log(e); rethrow; }` |
| `Exception` | famille des accidents prévisibles, à attraper | `class OrInsuffisantException implements Exception` |
| `Error` | famille des bugs de programmation, à corriger | `RangeError`, `StateError`, `TypeError` |
| `FormatException` | texte au mauvais format | `throw FormatException('Pseudo invalide', saisie, 3);` |
| `int.parse` | texte vers `int`, **lance** si échec | `int.parse('42')` |
| `int.tryParse` | texte vers `int?`, **retourne `null`** si échec | `int.tryParse('sept') ?? 1` |
| `double.tryParse` | texte vers `double?` | `double.tryParse('87.5')` |
| `ArgumentError` | argument invalide passé par le code appelant | `throw ArgumentError.value(prix, 'prix', 'négatif');` |
| `assert` | invariant vérifié en développement seulement | `assert(soin > 0, 'Potion invalide');` |
| `implements Exception` | rend une classe filtrable par `on Exception` | `class SaveException implements Exception` |
| `toString()` | rend l'exception lisible dans un message | `@override String toString() => '...';` |

Les cinq phrases à retenir :

1. Une `Error` est un bug à corriger ; une `Exception` est un accident à traiter.
2. `finally` est le seul bloc dont l'exécution est garantie.
3. Les clauses `on` se lisent du plus précis au plus général.
4. `rethrow` conserve la trace, `throw e` la détruit.
5. Quand un `tryParse` existe, il vaut mieux qu'un `try` / `catch`.

---

## 13.25 — Exercices

### Exercice 1 — Prédire la sortie (facile)

Sans exécuter le code, écrivez sur papier ce qui s'affiche, puis vérifiez dans DartPad.

```dart
void main() {
  print('A');
  try {
    print('B');
    int.parse('sept');
    print('C');
  } catch (e) {
    print('D');
  } finally {
    print('E');
  }
  print('F');
}
```

### Exercice 2 — Protéger une conversion (facile)

Écrivez une fonction `void afficherNiveau(String brut)` qui affiche `Niveau : 7` si la conversion réussit, et `Sauvegarde illisible : "sept"` sinon. Utilisez `try` / `on FormatException`. Testez avec `'7'`, `'sept'` et `'12.5'`.

### Exercice 3 — Passer à `tryParse` (facile)

Réécrivez l'exercice 2 **sans aucun `try`**, en utilisant `int.tryParse`. La sortie doit être identique.

### Exercice 4 — Inspecter l'exception (facile)

Écrivez un programme qui tente `int.parse('12€')` et affiche, dans un `catch`, trois lignes : le type réel de l'objet attrapé, son `message` et son `offset`. Attention : pour lire `message` et `offset`, il faut typer le `catch`.

### Exercice 5 — Deux `on` en cascade (moyen)

Une boutique possède le catalogue `['potion', 'épée', 'bouclier']`. Écrivez une fonction `void acheter(String indexBrut)` qui convertit l'indice puis lit l'article. Attrapez séparément :

- `FormatException` : afficher `Entrez un numéro valide.` ;
- `RangeError` : afficher `Cet article n'existe pas.`

Testez avec `'1'`, `'x'` et `'9'`.

### Exercice 6 — `finally` et ressource (moyen)

Simulez l'ouverture d'un fichier de sauvegarde avec une variable `bool fichierOuvert`. Écrivez `int chargerNiveau(String brut)` qui :

- passe `fichierOuvert` à `true` au début ;
- tente `int.parse(brut)` et retourne le résultat ;
- retourne `1` en cas de `FormatException` ;
- remet `fichierOuvert` à `false` dans un `finally` et affiche `(fichier refermé)`.

Vérifiez qu'après chaque appel, `fichierOuvert` vaut `false`.

### Exercice 7 — Lancer une exception (moyen)

Écrivez `int depenserOr(int solde, int montant)` qui retourne le nouveau solde, mais lance :

- un `ArgumentError` si `montant` est négatif ;
- une `Exception` si `montant` dépasse le solde.

Dans `main`, appelez la fonction avec `(100, 30)`, `(100, 500)` et `(100, -5)`, en n'attrapant **que** l'`Exception`. Observez ce qui se passe pour le troisième appel.

### Exercice 8 — Exception personnalisée (moyen)

Créez une classe `InventairePleinException implements Exception` avec un champ `capacite` et un `toString()` lisible. Écrivez une classe `Sac` de capacité 3 dont la méthode `ajouter(String objet)` lance cette exception quand le sac est plein. Ajoutez quatre objets dans une boucle protégée par un `try` / `on`.

### Exercice 9 — `rethrow` (moyen)

Écrivez deux fonctions :

- `int lireOr(String brut)` : convertit avec `int.parse`, journalise `[journal] or illisible : "..."` en cas d'échec, puis **relance** ;
- `void afficherOr(String brut)` : appelle `lireOr` et affiche `Or : 250`, ou `Or inconnu, remis à 0.` en cas d'exception.

Testez avec `'250'` puis `'beaucoup'`. Expliquez en une phrase pourquoi `rethrow` est préférable à `throw e` ici.

### Exercice 10 — Validation complète (difficile)

Écrivez une classe `Joueur(String pseudo, int niveau)` et une fonction `Joueur creerJoueur(String pseudoBrut, String niveauBrut)` qui lance une `FormatException` détaillée si :

- le pseudo, une fois `trim()`é, est vide ou dépasse 12 caractères ;
- le niveau n'est pas un entier ;
- le niveau n'est pas compris entre 1 et 50.

Dans `main`, testez cinq couples de valeurs et affichez soit le joueur créé, soit le message d'erreur. Le type de retour doit être `Joueur`, jamais `Joueur?`.

### Exercice 11 — Chargeur de sauvegarde (difficile)

Une sauvegarde est une chaîne `'pseudo;niveau;or'`. Écrivez :

- une classe `SauvegardeException implements Exception` avec les champs `ligne` et `raison` ;
- une fonction `Map<String, Object> lireSauvegarde(String ligne)` qui vérifie le nombre de champs, le pseudo non vide, le niveau et l'or numériques, et lance `SauvegardeException` sinon ;
- une fonction `Map<String, Object> chargerOuNouveau(String ligne)` qui journalise l'incident et retourne une partie neuve (`Invité`, niveau 1, 0 or).

Testez avec `'Alex;7;250'`, `'Alex;sept;250'`, `';7;250'` et `'Alex;7'`.

### Exercice 12 — Mini-projet : menu console robuste (difficile)

Écrivez un menu de jeu qui ne s'arrête **jamais** brutalement, quelle que soit la saisie.

Le menu affiche :

```text
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
```

Contraintes :

- DartPad n'a pas de clavier : simulez les saisies avec une `List<String>` de textes, par exemple `['1', '', 'trois', '9', '2', '3', '4']`.
- Toute saisie non numérique affiche `Saisie invalide : "trois" n'est pas un nombre.`
- Toute saisie hors de 1..4 affiche `Choix hors menu : 9. Entrez un nombre entre 1 et 4.`
- Une saisie vide affiche `Aucune saisie. Entrez un nombre entre 1 et 4.`
- L'option 2 tente de charger la sauvegarde `'Alex;sept'` : l'échec doit produire `Sauvegarde illisible, nouvelle partie créée.` et non un plantage.
- L'option 3 tente un achat de 80 pièces avec 30 pièces d'or : l'échec doit produire un message clair, avec le montant manquant.
- L'option 4 affiche `Au revoir.` et sort de la boucle.
- Le programme se termine par `Programme terminé normalement.`
- Aucune exception ne doit remonter jusqu'à la fin du programme.

Utilisez au moins : `int.tryParse`, une exception personnalisée, `on ... catch (e)`, et `finally`.

---

## 13.26 — Corrections des exercices

### Correction 1

```dart
void main() {
  print('A');
  try {
    print('B');
    int.parse('sept');
    print('C');
  } catch (e) {
    print('D');
  } finally {
    print('E');
  }
  print('F');
}
```

**Résultat :**

```text
A
B
D
E
F
```

**Explication :** `C` ne s'affiche jamais, car `int.parse('sept')` lance une `FormatException` et abandonne immédiatement le reste du bloc `try`. `D` s'affiche : le `catch` attrape. `E` s'affiche : le `finally` s'exécute toujours. `F` s'affiche : l'exception ayant été traitée, le programme reprend son cours normal après le bloc.

### Correction 2

```dart
void afficherNiveau(String brut) {
  try {
    final int niveau = int.parse(brut);
    print('Niveau : $niveau');
  } on FormatException {
    print('Sauvegarde illisible : "$brut"');
  }
}

void main() {
  afficherNiveau('7');
  afficherNiveau('sept');
  afficherNiveau('12.5');
}
```

**Résultat :**

```text
Niveau : 7
Sauvegarde illisible : "sept"
Sauvegarde illisible : "12.5"
```

**Explication :** `on FormatException` cible exactement le type lancé par `int.parse`. Un `catch (e)` nu fonctionnerait aussi, mais il attraperait également des problèmes sans rapport, comme un `RangeError`, ce qui masquerait un éventuel bug. `'12.5'` échoue parce que le point n'est pas un chiffre : `int.parse` refuse tout ce qui n'est pas un entier.

### Correction 3

```dart
void afficherNiveau(String brut) {
  final int? niveau = int.tryParse(brut);

  if (niveau == null) {
    print('Sauvegarde illisible : "$brut"');
    return;
  }
  print('Niveau : $niveau');
}

void main() {
  afficherNiveau('7');
  afficherNiveau('sept');
  afficherNiveau('12.5');
}
```

**Résultat :**

```text
Niveau : 7
Sauvegarde illisible : "sept"
Sauvegarde illisible : "12.5"
```

**Explication :** `int.tryParse` retourne `int?` : `null` remplace l'exception. Le retour anticipé (`return`) après le test provoque la promotion de type vue au chapitre 12 ; à la dernière ligne, `niveau` est donc un `int` ordinaire, utilisable sans `!` ni `??`. Cette version est plus courte, plus rapide, et ne crée aucune trace de pile.

### Correction 4

```dart
void main() {
  try {
    int.parse('12€');
  } on FormatException catch (e) {
    print('Type    : ${e.runtimeType}');
    print('Message : ${e.message}');
    print('Offset  : ${e.offset}');
  }
}
```

**Résultat :**

```text
Type    : FormatException
Message : Invalid radix-10 number
Offset  : 2
```

**Explication :** avec `catch (e)` seul, `e` serait de type `Object` et l'accès à `e.message` serait refusé à la compilation (`The getter 'message' isn't defined for the class 'Object'`). En écrivant `on FormatException catch (e)`, Dart sait que `e` est une `FormatException` et autorise ses propriétés. L'`offset` vaut `2` parce que les caractères `1` et `2` sont valides ; le premier caractère fautif est le `€`, à l'index 2.

### Correction 5

```dart
void acheter(String indexBrut) {
  final List<String> catalogue = ['potion', 'épée', 'bouclier'];

  try {
    final int index = int.parse(indexBrut);
    final String article = catalogue[index];
    print('Vous achetez : $article');
  } on FormatException {
    print('Entrez un numéro valide.');
  } on RangeError {
    print('Cet article n\'existe pas.');
  }
}

void main() {
  acheter('1');
  acheter('x');
  acheter('9');
}
```

**Résultat :**

```text
Vous achetez : épée
Entrez un numéro valide.
Cet article n'existe pas.
```

**Explication :** deux causes d'échec distinctes reçoivent deux messages distincts. L'ordre des clauses est ici sans importance, car `FormatException` et `RangeError` n'ont aucun lien de parenté : la première descend d'`Exception`, la seconde d'`Error`. Notez que `RangeError` est normalement un bug ; on l'attrape ici parce que l'indice vient d'une saisie du joueur, donc de l'extérieur. En production, on préférerait tester `index >= 0 && index < catalogue.length`.

### Correction 6

```dart
bool fichierOuvert = false;

int chargerNiveau(String brut) {
  fichierOuvert = true;
  try {
    return int.parse(brut);
  } on FormatException {
    return 1;
  } finally {
    fichierOuvert = false;
    print('  (fichier refermé)');
  }
}

void main() {
  for (final String brut in ['7', 'sept']) {
    print('--- chargement de "$brut" ---');
    final int niveau = chargerNiveau(brut);
    print('  niveau = $niveau');
    print('  fichier ouvert ? $fichierOuvert');
  }
}
```

**Résultat :**

```text
--- chargement de "7" ---
  (fichier refermé)
  niveau = 7
  fichier ouvert ? false
--- chargement de "sept" ---
  (fichier refermé)
  niveau = 1
  fichier ouvert ? false
```

**Explication :** le `finally` s'exécute dans les deux cas, y compris lorsque le `try` contient un `return`. C'est visible dans la sortie : `(fichier refermé)` apparaît **avant** l'affichage du niveau, parce que la fonction exécute son nettoyage avant de rendre la main. La ressource est donc toujours libérée, quelle que soit l'issue. Attention : il ne faut surtout pas mettre le `return` dans le `finally`, car il écraserait la valeur calculée.

### Correction 7

```dart
int depenserOr(int solde, int montant) {
  if (montant < 0) {
    throw ArgumentError.value(montant, 'montant', 'Le montant doit être positif');
  }
  if (montant > solde) {
    throw Exception('Or insuffisant : $solde pièces pour un achat de $montant');
  }
  return solde - montant;
}

void main() {
  for (final int montant in [30, 500, -5]) {
    print('--- dépense de $montant ---');
    try {
      print('  nouveau solde : ${depenserOr(100, montant)}');
    } on Exception catch (e) {
      print('  refusé : $e');
    }
  }
}
```

**Résultat :**

```text
--- dépense de 30 ---
  nouveau solde : 70
--- dépense de 500 ---
  refusé : Exception: Or insuffisant : 100 pièces pour un achat de 500
--- dépense de -5 ---
Unhandled exception:
Invalid argument (montant): Le montant doit être positif: -5
#0      depenserOr (file:///jeu/main.dart:3:5)
#1      main (file:///jeu/main.dart:16:32)
```

**Explication :** c'est le comportement attendu, et il illustre toute la section 13.2. Le solde insuffisant est un **accident** : le joueur a le droit d'essayer d'acheter trop cher, donc `Exception`, donc attrapé. Un montant négatif est un **bug** : aucune interface ne devrait produire cela, donc `ArgumentError`, qui descend d'`Error`. Comme `on Exception` ne filtre que la branche `Exception` de la hiérarchie, l'`ArgumentError` n'est pas attrapé et le programme plante — exactement ce qu'il faut pour que le développeur voie et corrige le problème.

### Correction 8

```dart
class InventairePleinException implements Exception {
  final int capacite;

  InventairePleinException(this.capacite);

  @override
  String toString() =>
      'InventairePleinException : le sac est plein ($capacite emplacements)';
}

class Sac {
  final List<String> objets = [];
  final int capacite;

  Sac({this.capacite = 3});

  void ajouter(String objet) {
    if (objets.length >= capacite) {
      throw InventairePleinException(capacite);
    }
    objets.add(objet);
    print('Ajouté : $objet (${objets.length}/$capacite)');
  }
}

void main() {
  final Sac sac = Sac();

  for (final String objet in ['potion', 'clé', 'épée', 'bouclier']) {
    try {
      sac.ajouter(objet);
    } on InventairePleinException catch (e) {
      print('Impossible d\'ajouter $objet.');
      print('  détail : $e');
      print('  capacité restante : ${e.capacite - sac.objets.length}');
    }
  }

  print('Sac final : ${sac.objets}');
}
```

**Résultat :**

```text
Ajouté : potion (1/3)
Ajouté : clé (2/3)
Ajouté : épée (3/3)
Impossible d'ajouter bouclier.
  détail : InventairePleinException : le sac est plein (3 emplacements)
  capacité restante : 0
Sac final : [potion, clé, épée]
```

**Explication :** la classe implémente `Exception`, ce qui la rend filtrable par `on Exception` comme par `on InventairePleinException`. Le champ `capacite` transporte une donnée exploitable : le `catch` peut calculer un message précis au lieu de recopier un texte figé. Le `toString()` redéfini évite l'affichage inutile `Instance of 'InventairePleinException'`. Enfin, la boucle continue après l'échec : le programme reste en vie.

### Correction 9

```dart
int lireOr(String brut) {
  try {
    return int.parse(brut);
  } on FormatException {
    print('[journal] or illisible : "$brut"');
    rethrow;
  }
}

void afficherOr(String brut) {
  try {
    print('Or : ${lireOr(brut)}');
  } on FormatException {
    print('Or inconnu, remis à 0.');
  }
}

void main() {
  afficherOr('250');
  afficherOr('beaucoup');
}
```

**Résultat :**

```text
Or : 250
[journal] or illisible : "beaucoup"
Or inconnu, remis à 0.
```

**Explication :** `lireOr` **constate** le problème et le note dans le journal, mais ne décide pas de la réaction : elle ne sait pas si l'appelant veut une valeur de repli, un message ou un arrêt. Elle relance donc avec `rethrow`. `afficherOr`, qui connaît le contexte, décide. `rethrow` est préférable à `throw e` parce qu'il conserve la trace de pile d'origine : la première ligne de la trace continue de pointer sur `int.parse`, et non sur la ligne du `catch` de `lireOr`.

### Correction 10

```dart
class Joueur {
  final String pseudo;
  final int niveau;

  Joueur(this.pseudo, this.niveau);

  @override
  String toString() => '$pseudo (niveau $niveau)';
}

Joueur creerJoueur(String pseudoBrut, String niveauBrut) {
  final String pseudo = pseudoBrut.trim();

  if (pseudo.isEmpty) {
    throw FormatException('Le pseudo ne peut pas être vide', pseudoBrut);
  }
  if (pseudo.length > 12) {
    throw FormatException('Pseudo trop long (12 maximum)', pseudo, 12);
  }

  final String niveauTexte = niveauBrut.trim();
  final int? niveau = int.tryParse(niveauTexte);

  if (niveau == null) {
    throw FormatException('Le niveau doit être un entier', niveauTexte, 0);
  }
  if (niveau < 1 || niveau > 50) {
    throw FormatException('Niveau hors limites (1 à 50)', niveauTexte, 0);
  }

  return Joueur(pseudo, niveau);
}

void main() {
  final List<List<String>> essais = [
    ['  Alex  ', '7'],
    ['   ', '7'],
    ['Alexandre-le-Terrible', '7'],
    ['Alex', 'sept'],
    ['Alex', '99'],
  ];

  for (final List<String> essai in essais) {
    print('--- "${essai[0]}" / "${essai[1]}" ---');
    try {
      final Joueur j = creerJoueur(essai[0], essai[1]);
      print('  OK : $j');
    } on FormatException catch (e) {
      print('  KO : ${e.message}');
    }
  }
}
```

**Résultat :**

```text
--- "  Alex  " / "7" ---
  OK : Alex (niveau 7)
--- "   " / "7" ---
  KO : Le pseudo ne peut pas être vide
--- "Alexandre-le-Terrible" / "7" ---
  KO : Pseudo trop long (12 maximum)
--- "Alex" / "sept" ---
  KO : Le niveau doit être un entier
--- "Alex" / "99" ---
  KO : Niveau hors limites (1 à 50)
```

**Explication :** la fonction retourne un `Joueur`, jamais un `Joueur?`. C'est le point clé : soit elle réussit et fournit un objet complet, soit elle lance. Aucun code appelant n'aura à tester la nullité du résultat. Le `trim()` est appliqué une seule fois, en tête, et c'est la valeur nettoyée qui est stockée. Pour le niveau, on utilise `int.tryParse` en interne — inutile de laisser remonter une `FormatException` technique dont le message anglais ne signifie rien pour le joueur — puis on lance notre propre `FormatException`, avec un message écrit pour un humain. Dans le `catch`, on affiche `e.message` plutôt que `e` : cela évite de montrer la ligne de position et l'accent circonflexe, inutiles ici.

### Correction 11

```dart
class SauvegardeException implements Exception {
  final String ligne;
  final String raison;

  SauvegardeException(this.ligne, this.raison);

  @override
  String toString() => 'SauvegardeException("$ligne") : $raison';
}

Map<String, Object> lireSauvegarde(String ligne) {
  final List<String> champs = ligne.split(';');

  if (champs.length != 3) {
    throw SauvegardeException(ligne, '3 champs attendus, ${champs.length} trouvé(s)');
  }

  final String pseudo = champs[0].trim();
  if (pseudo.isEmpty) {
    throw SauvegardeException(ligne, 'pseudo vide');
  }

  final int? niveau = int.tryParse(champs[1].trim());
  if (niveau == null) {
    throw SauvegardeException(ligne, 'niveau non numérique : "${champs[1]}"');
  }

  final int? or = int.tryParse(champs[2].trim());
  if (or == null) {
    throw SauvegardeException(ligne, 'or non numérique : "${champs[2]}"');
  }

  return {'pseudo': pseudo, 'niveau': niveau, 'or': or};
}

Map<String, Object> chargerOuNouveau(String ligne) {
  try {
    return lireSauvegarde(ligne);
  } on SauvegardeException catch (e) {
    print('  [journal] $e');
    return {'pseudo': 'Invité', 'niveau': 1, 'or': 0};
  }
}

void main() {
  final List<String> fichiers = [
    'Alex;7;250',
    'Alex;sept;250',
    ';7;250',
    'Alex;7',
  ];

  for (final String f in fichiers) {
    print('--- "$f" ---');
    final Map<String, Object> partie = chargerOuNouveau(f);
    print('  Partie : ${partie['pseudo']} | niveau ${partie['niveau']} | or ${partie['or']}');
  }
}
```

**Résultat :**

```text
--- "Alex;7;250" ---
  Partie : Alex | niveau 7 | or 250
--- "Alex;sept;250" ---
  [journal] SauvegardeException("Alex;sept;250") : niveau non numérique : "sept"
  Partie : Invité | niveau 1 | or 0
--- ";7;250" ---
  [journal] SauvegardeException(";7;250") : pseudo vide
  Partie : Invité | niveau 1 | or 0
--- "Alex;7" ---
  [journal] SauvegardeException("Alex;7") : 3 champs attendus, 2 trouvé(s)
  Partie : Invité | niveau 1 | or 0
```

**Explication :** la séparation des responsabilités est nette. `lireSauvegarde` valide et lance : elle ne connaît ni l'affichage, ni la politique de repli. `chargerOuNouveau` attrape, journalise et fournit une partie neuve : elle retourne donc **toujours** une valeur exploitable. Le reste du jeu n'a aucun test de nullité à faire. Notez l'emploi systématique de `tryParse` à l'intérieur : les échecs de conversion deviennent des `null` testables, que l'on retraduit ensuite en une exception métier au message clair. Le type de retour `Map<String, Object>` mélange volontairement un `String` et deux `int` ; nous verrons au chapitre 17 comment remplacer cette `Map` par une vraie classe de modèle, beaucoup plus sûre.

### Correction 12 — Mini-projet : menu console robuste

```dart
// ---------- Exceptions métier ----------

class SauvegardeException implements Exception {
  final String raison;
  SauvegardeException(this.raison);

  @override
  String toString() => 'SauvegardeException : $raison';
}

class OrInsuffisantException implements Exception {
  final int or;
  final int prix;
  OrInsuffisantException(this.or, this.prix);

  int get manque => prix - or;

  @override
  String toString() => 'OrInsuffisantException(or: $or, prix: $prix)';

  String get messageJoueur => 'Il vous manque $manque pièces d\'or.';
}

class ChoixInvalideException implements Exception {
  final String saisie;
  final String raison;
  ChoixInvalideException(this.saisie, this.raison);

  @override
  String toString() => raison;
}

// ---------- État du jeu ----------

class Partie {
  String pseudo;
  int niveau;
  int or;

  Partie(this.pseudo, this.niveau, this.or);

  @override
  String toString() => '$pseudo | niveau $niveau | $or pièces';
}

// ---------- Logique ----------

int lireChoix(String saisie) {
  final String s = saisie.trim();

  if (s.isEmpty) {
    throw ChoixInvalideException(s, 'Aucune saisie. Entrez un nombre entre 1 et 4.');
  }

  final int? n = int.tryParse(s);
  if (n == null) {
    throw ChoixInvalideException(s, 'Saisie invalide : "$s" n\'est pas un nombre.');
  }
  if (n < 1 || n > 4) {
    throw ChoixInvalideException(s, 'Choix hors menu : $n. Entrez un nombre entre 1 et 4.');
  }
  return n;
}

Partie lireSauvegarde(String ligne) {
  final List<String> champs = ligne.split(';');
  if (champs.length != 2) {
    throw SauvegardeException('2 champs attendus, ${champs.length} trouvé(s)');
  }
  final int? niveau = int.tryParse(champs[1].trim());
  if (niveau == null) {
    throw SauvegardeException('niveau non numérique : "${champs[1]}"');
  }
  return Partie(champs[0].trim(), niveau, 0);
}

void acheter(Partie partie, String article, int prix) {
  if (partie.or < prix) {
    throw OrInsuffisantException(partie.or, prix);
  }
  partie.or -= prix;
  print('  Achat de $article validé. Or restant : ${partie.or}');
}

void afficherMenu() {
  print('=== MENU ===');
  print('1. Nouvelle partie');
  print('2. Charger une partie');
  print('3. Boutique');
  print('4. Quitter');
}

// ---------- Programme ----------

void main() {
  final List<String> saisies = ['1', '', 'trois', '9', '2', '3', '4'];
  Partie partie = Partie('Invité', 1, 30);
  bool enCours = true;
  int tour = 0;

  while (enCours && tour < saisies.length) {
    final String saisie = saisies[tour];
    tour++;

    afficherMenu();
    print('> saisie : "$saisie"');

    try {
      final int choix = lireChoix(saisie);

      switch (choix) {
        case 1:
          partie = Partie('Alex', 1, 30);
          print('  Nouvelle partie : $partie');
          break;

        case 2:
          try {
            partie = lireSauvegarde('Alex;sept');
            print('  Partie chargée : $partie');
          } on SauvegardeException catch (e) {
            print('  [journal] $e');
            print('  Sauvegarde illisible, nouvelle partie créée.');
            partie = Partie('Invité', 1, 30);
          }
          break;

        case 3:
          try {
            acheter(partie, 'épée longue', 80);
          } on OrInsuffisantException catch (e) {
            print('  [journal] $e');
            print('  Achat refusé. ${e.messageJoueur}');
          }
          break;

        case 4:
          print('  Au revoir.');
          enCours = false;
          break;
      }
    } on ChoixInvalideException catch (e) {
      print('  $e');
    } finally {
      print('  --- fin du tour $tour ---');
    }
  }

  print('Programme terminé normalement.');
}
```

**Résultat :**

```text
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "1"
  Nouvelle partie : Alex | niveau 1 | 30 pièces
  --- fin du tour 1 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : ""
  Aucune saisie. Entrez un nombre entre 1 et 4.
  --- fin du tour 2 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "trois"
  Saisie invalide : "trois" n'est pas un nombre.
  --- fin du tour 3 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "9"
  Choix hors menu : 9. Entrez un nombre entre 1 et 4.
  --- fin du tour 4 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "2"
  [journal] SauvegardeException : niveau non numérique : "sept"
  Sauvegarde illisible, nouvelle partie créée.
  --- fin du tour 5 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "3"
  [journal] OrInsuffisantException(or: 30, prix: 80)
  Achat refusé. Il vous manque 50 pièces d'or.
  --- fin du tour 6 ---
=== MENU ===
1. Nouvelle partie
2. Charger une partie
3. Boutique
4. Quitter
> saisie : "4"
  Au revoir.
  --- fin du tour 7 ---
Programme terminé normalement.
```

**Explication :** ce mini-projet applique l'ensemble du chapitre.

- **Trois exceptions métier** sont définies, chacune pour une situation précise, chacune avec un `toString()` lisible. `ChoixInvalideException` porte directement le message destiné au joueur ; `OrInsuffisantException` sépare le message technique (`toString()`, pour le journal) du message humain (`messageJoueur`, pour l'écran), conformément à la règle 10 de la section 13.22.
- **`int.tryParse` est utilisé pour la conversion**, jamais `int.parse`. Une saisie non numérique est un événement ordinaire : elle ne mérite pas une exception du langage. C'est nous qui lançons ensuite une exception métier, avec un message en français.
- **La validation est en entonnoir** : chaîne vide, puis conversion, puis bornes. Chaque échec produit un message différent, ce qui aide réellement le joueur à corriger.
- **Les `try` sont imbriqués et courts.** Le `try` extérieur ne s'occupe que du choix du menu. Chaque action possède son propre `try` intérieur, avec le `on` correspondant à son seul risque. On sait donc toujours quelle instruction a échoué.
- **Le `finally` garantit la trace de fin de tour**, que le tour se soit bien passé ou non. Il ne contient aucun `return`.
- **La boucle ne s'interrompt jamais sur une exception.** Seul le choix 4 met `enCours` à `false`. La dernière ligne, `Programme terminé normalement.`, est atteinte dans tous les cas : c'était l'objectif du sujet.
- **Aucune `Error` n'est attrapée.** Si un vrai bug survenait — un indice de liste hors bornes, par exemple — le programme planterait, et c'est voulu : nous voulons le voir et le corriger, pas le masquer.

---

## Et maintenant ?

Vous savez désormais faire la différence entre un bug à corriger et un accident à traiter. Vous savez entourer le code risqué avec `try`, filtrer précisément avec `on`, consulter l'objet lancé avec `catch (e)`, garantir un nettoyage avec `finally`, signaler un problème avec `throw`, le faire remonter intact avec `rethrow`, et modéliser les incidents de votre jeu avec vos propres classes d'exception. Vous savez surtout qu'un `tryParse` bien placé vaut mieux qu'un `try` / `catch`, et qu'un `catch` vide est une faute.

Vos programmes ne s'arrêtent donc plus brutalement. Il est temps de les rendre plus courts et plus expressifs.

Jusqu'ici, pour transformer une liste — filtrer les ennemis vivants, extraire les noms d'un inventaire, additionner les points d'une partie — vous écriviez une boucle `for` et une variable d'accumulation. Dart propose beaucoup mieux : `map`, `where`, `fold`, `any`, `every`, `reduce`, `expand`, `sorted`. Ces méthodes décrivent **ce que vous voulez obtenir** au lieu de décrire comment parcourir la collection. Le code devient plus court, plus lisible, et beaucoup plus difficile à casser.

C'est l'objet du chapitre suivant, consacré à la programmation fonctionnelle sur les collections.

Rendez-vous au chapitre 14 : [14-PARTIE-1A—PROGRAMMATION-FONCTIONNELLE-SUR-LES-COLLECTIONS.md](14-PARTIE-1A—PROGRAMMATION-FONCTIONNELLE-SUR-LES-COLLECTIONS.md)
