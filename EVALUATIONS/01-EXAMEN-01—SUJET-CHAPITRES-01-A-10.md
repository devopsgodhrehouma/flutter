# EXAMEN 1 — PARTIE 1A, CHAPITRES 01 À 10

## Feuille d'examen

| | |
| --- | --- |
| **Formation** | Flutter + Jeu 2D — PARTIE 1A : le langage Dart |
| **Programme** | Chapitres 01 à 10 |
| **Durée** | 3 heures |
| **Barème** | 100 points |
| **Documents autorisés** | Aucun |
| **Machine** | Aucune (examen sur papier) — voir la variante « sur machine » en fin de sujet |

**Nom et prénom :** ................................................................

**Date :** ..................................  **Note :** ............ / 100

---

## Programme évalué

| Ch. | Notions |
| --- | --- |
| 01 | `main()`, `void`, `print()`, point-virgule, casse, commentaires, exécution séquentielle, conventions de nommage |
| 02 | Variables, `String`, `int`, `double`, `bool`, `num`, `var`, `dynamic`, `final`, `const`, `late`, interpolation, `toString()`, `int.parse()`, `double.parse()` |
| 03 | Opérateurs arithmétiques, `~/`, `%`, affectation composée, `++` et `--`, comparaisons, `&&`, `\|\|`, `!`, priorité, ternaire |
| 04 | `if`, `else if`, `else`, conditions imbriquées, `switch`, `case`, `default`, switch expression, motif `_` |
| 05 | `for`, `while`, `do while`, compteurs, accumulateurs, `break`, `continue`, boucles imbriquées, grilles 2D |
| 06 | `List`, `Set`, `Map`, index, méthodes de collection, collections imbriquées |
| 07 | Déclaration et appel, `return`, paramètres positionnels, optionnels, nommés, `required`, valeurs par défaut, `=>`, fonctions anonymes, portée |
| 08 | Classe, objet, instance, propriété, méthode, `this`, interaction entre objets |
| 09 | Constructeurs (par défaut, personnalisé, nommés, `const`, `factory`), initializer list, getters, setters, `toString()` |
| 10 | Encapsulation, membres privés `_`, getters et setters validés, `extends`, `super`, `@override`, polymorphisme, `is`, `as` |

**Hors programme.** Les notions des chapitres 11 et suivants ne sont pas évaluées et
n'apparaissent nulle part dans ce sujet : classes abstraites, interfaces, mixins, enums,
extensions, null safety avancé (`?`, `??`, `?.`, `!`), exceptions (`try`, `catch`, `throw`),
méthodes fonctionnelles de collection (`map`, `where`, `fold`, `sort`), asynchrone
(`Future`, `async`, `await`), packages et JSON.

---

## Structure de l'épreuve

| Partie | Intitulé | Questions | Points | Durée conseillée |
| --- | --- | ---: | ---: | ---: |
| A | Questionnaire à choix multiples | 40 | 20 | 30 min |
| B | Vrai ou faux, avec justification | 10 | 10 | 15 min |
| C | Lecture de code : qu'affiche ce programme ? | 10 | 15 | 30 min |
| D | Débogage : trouver et corriger l'erreur | 8 | 10 | 20 min |
| E | Compléter le code | 10 | 10 | 20 min |
| F | Questions de cours rédigées | 5 | 10 | 20 min |
| G | Exercices de programmation | 5 | 25 | 45 min |
| | **Total** | **88** | **100** | **3 h** |

---

## Consignes

1. Lisez le sujet en entier avant de commencer, puis traitez les parties dans l'ordre qui
   vous arrange. Les parties A, B et E se traitent vite : commencez par elles si vous
   craignez de manquer de temps.
2. Pour la partie C, écrivez la sortie **exactement** telle que le programme la produirait,
   une ligne par `print()`, en respectant les espaces et la ponctuation.
3. Pour la partie D, une réponse complète comporte trois éléments : l'erreur identifiée,
   son explication, et le code corrigé. Les trois sont notés séparément.
4. Pour la partie G, un code qui compile mais ne produit pas la sortie demandée obtient une
   partie des points. Un code qui ne compile pas en obtient beaucoup moins : relisez-vous.
5. L'indentation et le nommage comptent. Les conventions du chapitre 01 (`lowerCamelCase`
   pour les variables et fonctions, `UpperCamelCase` pour les classes) sont exigées.
6. Aucune notion hors programme n'est nécessaire. Si vous pensez avoir besoin d'un `try`
   ou d'un `map()`, c'est que vous compliquez : il existe une solution avec les seuls
   chapitres 01 à 10.

---


---

## PARTIE A — QUESTIONNAIRE À CHOIX MULTIPLES

**40 questions — 0,5 point par question — 20 points au total.**

Pour chaque question, une seule proposition est correcte. Indiquez la lettre choisie sur votre copie.
Aucune justification n'est demandée dans cette partie.

---

**A1.** *(ch. 01)* **(0,5 pt)** Quelle fonction constitue le point d'entrée obligatoire d'un programme Dart ?

a) `start()`
b) `main()`
c) `init()`
d) `run()`

---

**A2.** *(ch. 01)* **(0,5 pt)** Que signifie le mot-clé `void` écrit devant `main` ?

a) La fonction ne reçoit aucun paramètre
b) La fonction est vide et ne contient aucune instruction
c) La fonction est accessible depuis tout le fichier
d) La fonction ne renvoie aucune valeur

---

**A3.** *(ch. 01)* **(0,5 pt)** Qu'affiche exactement ce programme ?

```dart
void main() {
  print('Niveau 1');
  /* print('Niveau 2');
  print('Niveau 3'); */
  print('Niveau 4');
}
```

a) `Niveau 1` puis `Niveau 4`
b) `Niveau 1`, `Niveau 2`, `Niveau 3` puis `Niveau 4`
c) `Niveau 1` uniquement
d) `Niveau 1`, `Niveau 2` puis `Niveau 4`

---

**A4.** *(ch. 01)* **(0,5 pt)** Pourquoi ce programme refuse-t-il de compiler ?

```dart
void main() {
  print('Vies restantes')
  print(3);
}
```

a) `print` doit s'écrire avec une majuscule : `Print`
b) Il manque un point-virgule à la fin de la première instruction
c) `print(3)` doit recevoir un texte entre guillemets
d) `main()` ne peut pas contenir deux appels à `print`

---

**A5.** *(ch. 02)* **(0,5 pt)** Quelle déclaration stocke correctement le prix 19,99 en Dart ?

a) `double prix = 19,99;`
b) `int prix = 19.99;`
c) `double prix = 19.99;`
d) `String prix = 19.99;`

---

**A6.** *(ch. 02)* **(0,5 pt)** Quelle est la différence entre `final` et `const` ?

a) `final` accepte une valeur calculée à l'exécution, `const` l'exige dès la compilation
b) `final` autorise une seconde réassignation, `const` interdit toute réassignation
c) `final` s'applique aux nombres, `const` s'applique uniquement aux textes
d) `final` concerne les variables locales, `const` les variables globales

---

**A7.** *(ch. 02)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int vies = 3;
  print('Il reste $vies vies, plus que ${vies - 1} après le choc');
}
```

a) `Il reste $vies vies, plus que 2 après le choc`
b) `Il reste 3 vies, plus que ${vies - 1} après le choc`
c) `Il reste 3 vies, plus que 3 - 1 après le choc`
d) `Il reste 3 vies, plus que 2 après le choc`

---

**A8.** *(ch. 02)* **(0,5 pt)** Que produit ce programme ?

```dart
void main() {
  var score = 100;
  score = 'élevé';
  print(score);
}
```

a) Il affiche `élevé`, car `var` accepte n'importe quel type
b) Il ne compile pas : `var` a figé le type `int` dès la déclaration
c) Il affiche `100`, car la réassignation est simplement ignorée
d) Il affiche `null`, car la valeur assignée est invalide

---

**A9.** *(ch. 03)* **(0,5 pt)** Que vaut l'expression `15 ~/ 4` en Dart ?

a) `3.75`
b) `4`
c) `3`
d) `3.0`

---

**A10.** *(ch. 03)* **(0,5 pt)** Quelle expression teste qu'un `niveau` est un multiple de 5 ?

a) `niveau % 5 == 0`
b) `niveau ~/ 5 == 0`
c) `niveau / 5 == 0`
d) `niveau % 5 == 5`

---

**A11.** *(ch. 03)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int score = 10;
  int copie = score++;
  print('$copie $score');
}
```

a) `11 11`
b) `10 11`
c) `11 10`
d) `10 10`

---

**A12.** *(ch. 03)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int vies = 0;
  int score = 120;
  String etat = vies > 0 ? 'En jeu' : score > 100 ? 'Record' : 'Fini';
  print(etat);
}
```

a) `En jeu`
b) `Fini`
c) Rien : les opérateurs ternaires ne peuvent pas être imbriqués
d) `Record`

---

**A13.** *(ch. 04)* **(0,5 pt)** Dans une chaîne `if / else if / else`, dans quel ordre faut-il placer les conditions ?

a) De la condition la plus large à la condition la plus restrictive
b) Dans un ordre indifférent, car Dart évalue toutes les branches
c) De la condition la plus restrictive à la condition la plus large
d) Les conditions numériques d'abord, les conditions textuelles ensuite

---

**A14.** *(ch. 04)* **(0,5 pt)** Dans un `switch` classique, que se passe-t-il si un `case` non vide ne se termine pas par `break` ?

a) Le `case` suivant s'exécute à son tour, par effet de cascade
b) Le code ne compile pas : Dart interdit la cascade implicite
c) Le programme s'interrompt à l'exécution avec une erreur
d) Le bloc `default` s'exécute à la place de ce `case`

---

**A15.** *(ch. 04)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int score = 1200;
  if (score >= 500) {
    print('Argent');
  } else if (score >= 1000) {
    print('Or');
  } else {
    print('Bronze');
  }
}
```

a) `Argent`
b) `Or`
c) `Argent` puis `Or`
d) `Bronze`

---

**A16.** *(ch. 04)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  String touche = 'Z';
  String action = switch (touche) {
    'A' => 'Sauter',
    'B' => 'Attaquer',
    _ => 'Aucune action',
  };
  print(action);
}
```

a) `Sauter`
b) `_`
c) Rien : il manque un bloc `default`
d) `Aucune action`

---

**A17.** *(ch. 05)* **(0,5 pt)** Quelle boucle exécute son bloc au moins une fois, même si la condition est fausse dès le départ ?

a) `for`
b) `while`
c) `do while`
d) `for-in`

---

**A18.** *(ch. 05)* **(0,5 pt)** Que fait l'instruction `continue` à l'intérieur d'une boucle ?

a) Elle arrête définitivement la boucle en cours
b) Elle ignore le tour courant et passe à l'itération suivante
c) Elle relance la boucle depuis sa toute première itération
d) Elle reprend l'exécution juste après le bloc de la boucle

---

**A19.** *(ch. 05)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  for (int i = 1; i <= 5; i++) {
    if (i % 2 == 0) {
      continue;
    }
    print(i);
  }
}
```

a) `2` puis `4`
b) `1` uniquement
c) `1`, `2`, `3`, `4` puis `5`
d) `1`, `3` puis `5`

---

**A20.** *(ch. 05)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int compteur = 0;
  for (int ligne = 1; ligne <= 3; ligne++) {
    for (int colonne = 1; colonne <= 4; colonne++) {
      if (colonne == 3) break;
      compteur++;
    }
  }
  print(compteur);
}
```

a) `6`
b) `12`
c) `9`
d) `3`

---

**A21.** *(ch. 06)* **(0,5 pt)** Quelle collection garantit que chaque valeur n'apparaît qu'une seule fois ?

a) La `List`, dès qu'elle est typée
b) La `Map`, grâce à ses valeurs
c) Le `Set`
d) La `List` déclarée `const`

---

**A22.** *(ch. 06)* **(0,5 pt)** Pour une `List` contenant 5 éléments, quel est le dernier index valide ?

a) `5`
b) `-1`
c) `length`
d) `4`

---

**A23.** *(ch. 06)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  List<String> armes = ['Épée', 'Arc', 'Bâton'];
  armes.removeAt(1);
  armes.add('Hache');
  print(armes);
}
```

a) `[Épée, Bâton, Hache]`
b) `[Arc, Bâton, Hache]`
c) `[Épée, Arc, Hache]`
d) `[Épée, Bâton, Arc, Hache]`

---

**A24.** *(ch. 06)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  final List<int> scores = [10, 20];
  scores.add(30);
  print(scores);
}
```

a) Rien : une liste `final` ne peut pas être modifiée, le code ne compile pas
b) `[10, 20, 30]`
c) `[10, 20]`, car l'ajout est refusé silencieusement
d) Rien : le programme s'arrête à l'exécution, la liste est figée

---

**A25.** *(ch. 07)* **(0,5 pt)** Soit la fonction `void creerJoueur({required String nom, int vies = 3})`. Quel appel est correct ?

a) `creerJoueur('Alex', 3);`
b) `creerJoueur(nom = 'Alex');`
c) `creerJoueur(nom: 'Alex');`
d) `creerJoueur('Alex');`

---

**A26.** *(ch. 07)* **(0,5 pt)** Que signifie l'écriture `int doubler(int x) => x * 2;` ?

a) C'est la forme courte d'une fonction dont le corps tient en une seule expression renvoyée
b) C'est une fonction anonyme, stockée dans une variable qui s'appelle `doubler`
c) C'est une fonction qui n'effectue aucun retour, malgré son type `int` annoncé
d) C'est un callback, utilisable uniquement comme argument d'une autre fonction

---

**A27.** *(ch. 07)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
void ajouter(int valeur) {
  valeur = valeur + 10;
}

void main() {
  int score = 5;
  ajouter(score);
  print(score);
}
```

a) `15`
b) `10`
c) Rien : un paramètre ne peut pas être modifié, le code ne compile pas
d) `5`

---

**A28.** *(ch. 07)* **(0,5 pt)** Soit `void executer(Function action)` et une fonction `attaquer()`. Comment passer `attaquer` en callback ?

a) `executer(attaquer());`
b) `executer('attaquer');`
c) `executer(attaquer);`
d) `executer(Function attaquer);`

---

**A29.** *(ch. 08)* **(0,5 pt)** Avec `class Player { }`, comment crée-t-on un objet en Dart moderne ?

a) `Player joueur = Player();`
b) `Player joueur = new Player();`
c) `Player joueur = Player.create();`
d) `Player joueur = Player;`

---

**A30.** *(ch. 08)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
class Player {
  String nom = 'Alex';
}

void main() {
  Player joueur = Player();
  print('Nom : $joueur.nom');
}
```

a) `Nom : Alex`
b) `Nom : Instance of 'Player'.nom`
c) `Nom : $joueur.nom`
d) Rien : l'interpolation est invalide, le code ne compile pas

---

**A31.** *(ch. 08)* **(0,5 pt)** Dans quel cas le mot-clé `this` est-il réellement nécessaire en Dart ?

a) Devant chaque propriété utilisée à l'intérieur d'une méthode
b) Uniquement dans les méthodes qui renvoient une valeur typée
c) Quand un paramètre porte le même nom qu'une propriété de la classe
d) Dès que la classe possède plus d'une propriété déclarée

---

**A32.** *(ch. 08)* **(0,5 pt)** Qu'affiche ce programme ?

```dart
class Player {
  int vies = 3;
}

void main() {
  Player a = Player();
  Player b = a;
  b.vies = 1;
  print(a.vies);
}
```

a) `3`
b) `0`
c) Rien : `vies` n'est pas accessible depuis `main()`
d) `1`

---

**A33.** *(ch. 09)* **(0,5 pt)** Que fait le constructeur `Potion(this.nom, this.soin);` ?

a) Il déclare dans la classe les deux propriétés `nom` et `soin`
b) Il affecte directement les deux arguments reçus aux propriétés de même nom
c) Il crée deux paramètres nommés obligatoires, `nom` et `soin`
d) Il recopie les valeurs d'une autre `Potion` passée en argument

---

**A34.** *(ch. 09)* **(0,5 pt)** Soit le getter `bool get estVivant => _pv > 0;`. Comment l'utilise-t-on sur un objet `joueur` ?

a) `joueur.estVivant()`
b) `joueur.get(estVivant)`
c) `joueur.estVivant`
d) `joueur.getEstVivant()`

---

**A35.** *(ch. 09)* **(0,5 pt)** Avec la classe ci-dessous, qu'affiche `print(Potion('Élixir', 30));` ?

```dart
class Potion {
  final String nom;
  final int soin;
  Potion(this.nom, this.soin);
  @override
  String toString() => '$nom : +$soin PV';
}
```

a) `Élixir : +30 PV`
b) `Instance of 'Potion'`
c) `Potion('Élixir', 30)`
d) `$nom : +$soin PV`

---

**A36.** *(ch. 09)* **(0,5 pt)** Pourquoi cette classe refuse-t-elle de compiler ?

```dart
class Position {
  int x;
  int y;
  const Position(this.x, this.y);
}
```

a) Un constructeur `const` doit obligatoirement porter un nom
b) Un constructeur `const` doit toujours posséder un corps `{ }`
c) Les propriétés `x` et `y` doivent être déclarées avec `var`
d) Un constructeur `const` exige que toutes les propriétés soient `final`

---

**A37.** *(ch. 10)* **(0,5 pt)** En Dart, que rend exactement le préfixe `_` placé devant le nom d'un membre ?

a) Il le rend privé à la bibliothèque, c'est-à-dire au fichier
b) Il le rend privé à la seule classe qui le déclare
c) Il le rend accessible uniquement aux sous-classes qui en héritent
d) Il le rend accessible en lecture, mais interdit toute écriture

---

**A38.** *(ch. 10)* **(0,5 pt)** Lorsque la classe parente exige des arguments, où doit figurer l'appel `super(...)` dans le constructeur de la sous-classe ?

a) Sur la toute première ligne du corps du constructeur
b) En premier dans la liste d'initialisation, avant les autres initialisations
c) En dernier dans la liste d'initialisation, après les autres initialisations
d) Nulle part : Dart appelle le constructeur parent automatiquement

---

**A39.** *(ch. 10)* **(0,5 pt)** Avec les classes ci-dessous, qu'affichent les instructions `Character c = Boss();` puis `c.attaquer();` ?

```dart
class Character {
  void attaquer() => print('Coup simple');
}

class Boss extends Character {
  @override
  void attaquer() => print('Coup dévastateur');
}
```

a) `Coup simple`
b) `Coup dévastateur`
c) `Coup simple` puis `Coup dévastateur`
d) Rien : une variable `Character` ignore la version de `Boss`

---

**A40.** *(ch. 10)* **(0,5 pt)** Quelle est la différence entre `is` et `as` ?

a) `is` force la conversion, `as` teste le type sans rien convertir
b) `is` et `as` sont deux écritures équivalentes du même test de type
c) `is` ne s'applique qu'aux classes mères, `as` qu'aux classes filles
d) `is` teste le type et promeut la variable, `as` force la conversion

---

---

## PARTIE B — VRAI OU FAUX, AVEC JUSTIFICATION

**10 questions × 1 point = 10 points**

Pour chaque affirmation, indiquez si elle est **vraie** ou **fausse**, puis **justifiez votre réponse
en une phrase**. Une réponse sans justification ne vaut que la moitié des points.

---

**B1.** *(ch. 01)* **(1 pt)** L'instruction `print(10 / 5);` affiche `2` dans la console.

---

**B2.** *(ch. 02)* **(1 pt)** Après la déclaration `var score = 10;`, l'instruction `score = 20;` est
acceptée, alors que l'instruction `score = 'vingt';` provoque une erreur de compilation.

---

**B3.** *(ch. 03)* **(1 pt)** En Dart, l'expression `-7 % 3` vaut `-1`.

---

**B4.** *(ch. 04)* **(1 pt)** Dans un `switch` classique, si l'on oublie le `break;` à la fin d'un
`case` qui contient des instructions, l'exécution se poursuit automatiquement dans le `case` suivant.

---

**B5.** *(ch. 05)* **(1 pt)** Le fragment de code suivant affiche `Tour` exactement une fois :

```dart
int tours = 10;
do {
  print('Tour');
  tours++;
} while (tours < 5);
```

---

**B6.** *(ch. 06)* **(1 pt)** Après la déclaration `final List<String> inventaire = ['Epee'];`,
l'appel `inventaire.add('Potion');` est parfaitement légal.

---

**B7.** *(ch. 07)* **(1 pt)** Dart autorise deux fonctions portant le même nom dans un même fichier,
à condition que leurs listes de paramètres soient différentes.

---

**B8.** *(ch. 08)* **(1 pt)** Avec la classe `Player` du chapitre 08, si l'on écrit
`Player b = a;` puis `b.health = 50;`, alors la lecture de `a.health` renvoie elle aussi `50`.

---

**B9.** *(ch. 09)* **(1 pt)** Un getter calculé déclaré `bool get estVivant => health > 0;`
s'appelle depuis l'extérieur de la classe avec `joueur.estVivant()`.

---

**B10.** *(ch. 10)* **(1 pt)** Le préfixe `_` de `int _hp = 100;` rend la propriété privée au **fichier**
et non à la classe : une autre classe déclarée dans le même fichier peut lire `_hp` directement.

---

## PARTIE C — LECTURE DE CODE : QU'AFFICHE CE PROGRAMME ?

**10 questions × 1,5 point = 15 points**

Pour chaque programme, écrivez la sortie console **exacte**, ligne par ligne, dans l'ordre
d'affichage. Respectez la ponctuation, les espaces et le format des nombres. Ne corrigez pas les
programmes : ils compilent tous.

---

**C1.** *(ch. 02)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  String pseudo = 'Aria';
  int niveau = 3;
  double energie = 7.5;
  var bonus = int.parse('20');
  final arme = 'Arc';

  print('$pseudo joue au niveau $niveau');
  print('Energie : $energie');
  print('Arme : $arme');
  print('Score : ${niveau * 10 + bonus}');
  print('Partage : ${10 / 4}');
  print(energie.toString() + ' PV');
}
```

---

**C2.** *(ch. 03)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int score = 17;
  int equipes = 3;
  print(score ~/ equipes);
  print(score % equipes);

  int a = 5;
  int b = a++ + 2;
  print('$a $b');

  int c = 5;
  int d = ++c + 2;
  print('$c $d');

  score += 3;
  score = score ~/ 2;
  print(score);
  print(score > 8 && equipes != 3);
  print(score > 8 || equipes != 3);
}
```

---

**C3.** *(ch. 04)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int niveau = 7;
  if (niveau > 10) {
    print('Expert');
  } else if (niveau > 5) {
    print('Confirme');
  } else if (niveau > 3) {
    print('Intermediaire');
  } else {
    print('Debutant');
  }

  String touche = 'B';
  switch (touche) {
    case 'A':
    case 'B':
      print('Attaque');
      break;
    case 'C':
      print('Defense');
      break;
    default:
      print('Inconnu');
  }

  int degats = 0;
  String etat = switch (degats) {
    0 => 'Intact',
    1 => 'Blesse',
    _ => 'Critique',
  };
  print(etat);
}
```

---

**C4.** *(ch. 05)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  int total = 0;
  for (int i = 1; i <= 6; i++) {
    if (i % 2 == 0) {
      continue;
    }
    if (i > 4) {
      break;
    }
    total += i;
    print('i = $i');
  }
  print('Total : $total');

  int vies = 3;
  while (vies > 0) {
    vies -= 2;
  }
  print('Vies : $vies');

  int tour = 0;
  do {
    tour++;
  } while (tour < 0);
  print('Tour : $tour');
}
```

---

**C5.** *(ch. 05-06)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  List<String> inventaire = ['Epee', 'Potion', 'Arc', 'Potion'];
  print(inventaire.length);
  print(inventaire.indexOf('Potion'));
  print(inventaire.indexOf('Bouclier'));

  inventaire.removeAt(1);
  inventaire.add('Bouclier');
  print(inventaire);

  Set<String> uniques = <String>{};
  for (String objet in inventaire) {
    uniques.add(objet);
  }
  print(uniques.length);

  int compteur = 0;
  for (int i = 0; i < inventaire.length; i++) {
    if (inventaire[i].length > 3) {
      compteur++;
    }
  }
  print(compteur);
}
```

---

**C6.** *(ch. 06)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
void main() {
  Map<String, dynamic> joueur = {
    'nom': 'Aria',
    'vies': 3,
    'score': 250,
  };
  joueur['niveau'] = 4;
  joueur['score'] = joueur['score'] + 50;

  print(joueur['nom']);
  print(joueur['score']);
  print(joueur.length);
  print(joueur.containsKey('arme'));

  List<List<int>> grille = [
    [1, 2],
    [3, 4],
  ];
  int somme = 0;
  for (List<int> ligne in grille) {
    for (int valeur in ligne) {
      somme += valeur;
    }
  }
  print(somme);
  print(grille[1][0]);
}
```

---

**C7.** *(ch. 07)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
int doubler(int valeur) {
  valeur = valeur * 2;
  return valeur;
}

void ajouterObjet(List<String> sac, String objet) {
  sac.add(objet);
}

String decrire({required String nom, int niveau = 1}) => '$nom (niv $niveau)';

void main() {
  int puissance = 10;
  int resultat = doubler(puissance);
  print(puissance);
  print(resultat);

  List<String> sac = ['Epee'];
  ajouterObjet(sac, 'Potion');
  print(sac);
  print(sac.length);

  print(decrire(nom: 'Aria'));
  print(decrire(niveau: 5, nom: 'Bran'));
}
```

---

**C8.** *(ch. 08)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
class Player {
  String name = 'Inconnu';
  int health = 100;

  void takeDamage(int amount) {
    health -= amount;
    if (health < 0) {
      health = 0;
    }
  }

  bool isAlive() {
    return health > 0;
  }
}

void main() {
  Player a = Player();
  a.name = 'Aria';
  a.takeDamage(30);

  Player b = a;
  b.takeDamage(80);

  Player c = Player();
  c.name = 'Bran';

  print(a.health);
  print(b.health);
  print(a.name);
  print(a.isAlive());
  print(c.health);
}
```

---

**C9.** *(ch. 09)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
class Potion {
  final String nom;
  final int soin;

  Potion(this.nom, this.soin);

  Potion.grande(String nom) : this(nom, 50);

  int get valeur => soin * 2;

  @override
  String toString() => 'Potion($nom, +$soin PV)';
}

void main() {
  Potion p1 = Potion('Petite fiole', 15);
  Potion p2 = Potion.grande('Elixir');

  print(p1);
  print(p2);
  print(p1.valeur);
  print('${p2.nom} vaut ${p2.valeur}');
  print(p1.soin + p2.soin);
}
```

---

**C10.** *(ch. 10)* **(1,5 pt)** Qu'affiche ce programme ?

```dart
class Character {
  final String name;
  int _hp;

  Character(this.name, this._hp) {
    print('Character $name');
  }

  int get hp => _hp;

  void takeDamage(int degats) {
    _hp -= degats;
  }

  String describe() => '$name : $hp PV';
}

class Boss extends Character {
  final int armure;

  Boss(String nom, int pv, this.armure) : super(nom, pv) {
    print('Boss $nom');
  }

  @override
  void takeDamage(int degats) {
    super.takeDamage(degats - armure);
  }

  @override
  String describe() => 'BOSS ${super.describe()}';
}

void main() {
  List<Character> equipe = [Character('Aria', 100), Boss('Golem', 200, 15)];

  for (Character c in equipe) {
    c.takeDamage(40);
    print(c.describe());
  }

  print(equipe[1] is Boss);
  print(equipe[0] is Boss);
}
```

---

## PARTIE D — DÉBOGAGE : TROUVER ET CORRIGER L'ERREUR

**8 questions × 1,25 point — 10 points**

Chacun des programmes suivants contient **exactement une erreur**. Pour chaque extrait :

1. **identifiez** l'erreur (indiquez la ligne ou l'instruction fautive) ;
2. **expliquez** pourquoi cette erreur se produit ;
3. **écrivez la correction** (la ligne corrigée suffit, mais elle doit être exacte).

Certaines erreurs empêchent la compilation, d'autres laissent le programme s'exécuter mais
produisent un résultat faux. Le comportement attendu est précisé dans chaque énoncé.

---

**D1.** *(ch. 02)* **(1,25 pt)** Ce programme devrait afficher le score du joueur, puis relever
le score maximum à 200 avant le second affichage. Il refuse de compiler.

```dart
void main() {
  final int scoreMax = 100;
  int score = 0;

  score += 40;
  print('Score : $score / $scoreMax');

  scoreMax = 200;
  score += 90;
  print('Score : $score / $scoreMax');
}
```

Sortie attendue :

```text
Score : 40 / 100
Score : 130 / 200
```

---

**D2.** *(ch. 03-04)* **(1,25 pt)** Ce programme devrait attribuer le rang `Or` à un score de
1200 (rang `Or` à partir de 1000, `Argent` à partir de 500, `Bronze` en dessous). Il compile et
s'exécute, mais il affiche `Rang : Argent`.

```dart
void main() {
  int score = 1200;
  String rang = 'Bronze';

  if (score >= 500) {
    rang = 'Argent';
  } else if (score >= 1000) {
    rang = 'Or';
  }

  print('Score : $score');
  print('Rang : $rang');
}
```

Sortie attendue :

```text
Score : 1200
Rang : Or
```

---

**D3.** *(ch. 05)* **(1,25 pt)** Ce programme devrait faire perdre une vie au joueur à chaque
tour, jusqu'à ce qu'il n'en ait plus, puis afficher `Game Over`. En réalité il ne s'arrête
jamais et affiche indéfiniment les deux mêmes lignes.

```dart
void main() {
  int vies = 3;

  while (vies > 0) {
    print('Il vous reste $vies vie(s)');
    print('Un ennemi attaque !');
  }

  print('Game Over');
}
```

Sortie attendue :

```text
Il vous reste 3 vie(s)
Un ennemi attaque !
Il vous reste 2 vie(s)
Un ennemi attaque !
Il vous reste 1 vie(s)
Un ennemi attaque !
Game Over
```

---

**D4.** *(ch. 06)* **(1,25 pt)** Ce programme devrait mémoriser les types d'ennemis vaincus sans
doublon, puis afficher la collection, le nombre d'éléments et la présence de `Orc`. Il refuse de
compiler.

```dart
void main() {
  var ennemisVaincus = {};

  ennemisVaincus.add('Gobelin');
  ennemisVaincus.add('Orc');
  ennemisVaincus.add('Gobelin');

  bool orcVaincu = ennemisVaincus.contains('Orc');

  print(ennemisVaincus);
  print('Ennemis différents vaincus : ${ennemisVaincus.length}');
  print('Orc vaincu ? $orcVaincu');
}
```

Sortie attendue :

```text
{Gobelin, Orc}
Ennemis différents vaincus : 2
Orc vaincu ? true
```

---

**D5.** *(ch. 07)* **(1,25 pt)** Ce programme devrait infliger 25 dégâts au gobelin, puis 60
dégâts au dragon avec un coup critique (qui double les dégâts). Il refuse de compiler.

```dart
void infligerDegats({
  required String cible,
  required int degats,
  bool critique = false,
}) {
  int total = degats;
  if (critique) {
    total = degats * 2;
  }
  print('$cible perd $total points de vie');
}

void main() {
  infligerDegats(cible: 'Gobelin', degats: 25);
  infligerDegats(cible: 'Dragon', critique: true);
}
```

Sortie attendue :

```text
Gobelin perd 25 points de vie
Dragon perd 120 points de vie
```

---

**D6.** *(ch. 08)* **(1,25 pt)** Ce programme devrait renommer le joueur en `Alex` puis lui
ajouter 250 points. Il compile et s'exécute, mais il affiche `Inconnu a 250 points`.

```dart
class Joueur {
  String nom = 'Inconnu';
  int score = 0;
  void renommer(String nom) {
    nom = nom;
  }
  void ajouterPoints(int points) {
    score += points;
  }
}

void main() {
  Joueur j = Joueur();
  j.renommer('Alex');
  j.ajouterPoints(250);
  print('${j.nom} a ${j.score} points');
}
```

Sortie attendue :

```text
Alex a 250 points
```

---

**D7.** *(ch. 09)* **(1,25 pt)** Ce programme devrait créer deux positions immuables et les
afficher grâce à `toString()`. Il refuse de compiler.

```dart
class Position {
  int x;
  int y;
  const Position(this.x, this.y);
  @override
  String toString() => 'Position($x, $y)';
}

void main() {
  const Position depart = Position(0, 0);
  const Position arrivee = Position(3, 5);
  print(depart);
  print(arrivee);
}
```

Sortie attendue :

```text
Position(0, 0)
Position(3, 5)
```

---

**D8.** *(ch. 10)* **(1,25 pt)** Ce programme devrait faire attaquer le boss avec **sa propre**
version de l'attaque, appelée à travers une variable déclarée `Personnage`. Il compile et
s'exécute, mais il affiche `Draknor inflige 50 dégâts`.

```dart
class Personnage {
  String nom;
  int degats;
  Personnage(this.nom, this.degats);
  void attaquer() {
    print('$nom inflige $degats dégâts');
  }
}

class Boss extends Personnage {
  Boss(String nom) : super(nom, 50);
  void attaque() {
    print('$nom déclenche une attaque dévastatrice de $degats dégâts');
  }
}

void main() {
  Personnage ennemi = Boss('Draknor');
  ennemi.attaquer();
}
```

Sortie attendue :

```text
Draknor déclenche une attaque dévastatrice de 50 dégâts
```

---

---

## PARTIE E — COMPLÉTER LE CODE

**10 questions × 1 point — 10 points**

Dans chaque programme, une ou deux expressions ont été remplacées par des marqueurs numérotés
`/* 1 */` et `/* 2 */`. Écrivez sur votre copie **ce qu'il faut mettre à la place de chaque
marqueur** pour que le programme produise **exactement** la sortie indiquée. Vous n'avez pas
besoin de recopier le programme entier.

---

**E1.** *(ch. 01)* **(1 pt)** Complétez pour obtenir la sortie indiquée.

```dart
/* 1 */ main() {
  // Écran d'accueil du jeu
  print('Bienvenue dans Donjon 2D');
  /* 2 */('Appuyez sur ENTRÉE pour commencer');
}
```

```text
Bienvenue dans Donjon 2D
Appuyez sur ENTRÉE pour commencer
```

---

**E2.** *(ch. 02)* **(1 pt)** Complétez pour obtenir la sortie indiquée. La variable `saisie`
représente un texte saisi par le joueur.

```dart
void main() {
  final String pseudo = 'Alex';
  String saisie = '150';

  int points = /* 1 */;
  int bonus = 50;

  print('Joueur : $pseudo');
  print('Total : /* 2 */ points');
}
```

```text
Joueur : Alex
Total : 200 points
```

---

**E3.** *(ch. 03)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Un sac contient au
maximum 10 pièces.

```dart
void main() {
  int totalPieces = 47;
  int piecesParSac = 10;

  int sacsPleins = totalPieces /* 1 */ piecesParSac;
  int reste = totalPieces /* 2 */ piecesParSac;

  print('Sacs pleins : $sacsPleins');
  print('Pièces restantes : $reste');
}
```

```text
Sacs pleins : 4
Pièces restantes : 7
```

---

**E4.** *(ch. 04)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Toute touche non listée
doit donner `Action inconnue`.

```dart
void main() {
  String touche = 'B';

  String action = switch (touche) {
    'A' => 'Attaquer',
    'B' => /* 1 */,
    'C' => 'Courir',
    /* 2 */ => 'Action inconnue',
  };

  print('Touche $touche : $action');
}
```

```text
Touche B : Bloquer
```

---

**E5.** *(ch. 05)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Le programme parcourt
les niveaux 1 à 10, signale les niveaux de boss (tous les 5 niveaux) et cumule les numéros de
niveau.

```dart
void main() {
  int total = 0;

  for (int niveau = 1; /* 1 */; niveau++) {
    if (niveau /* 2 */ == 0) {
      print('Niveau $niveau : boss !');
    }
    total += niveau;
  }

  print('Total : $total');
}
```

```text
Niveau 5 : boss !
Niveau 10 : boss !
Total : 55
```

---

**E6.** *(ch. 06)* **(1 pt)** Complétez pour obtenir la sortie indiquée. La potion doit
disparaître de l'inventaire, et on veut savoir si la clé `bombe` figure dans le stock.

```dart
void main() {
  List<String> inventaire = ['épée', 'potion', 'bouclier'];
  inventaire./* 1 */('potion');
  inventaire.add('arc');

  Map<String, int> stock = {'potion': 3, 'bombe': 1};
  bool aBombe = stock./* 2 */('bombe');

  print(inventaire);
  print('Objets : ${inventaire.length}');
  print('Bombe en stock ? $aBombe');
}
```

```text
[épée, bouclier, arc]
Objets : 3
Bombe en stock ? true
```

---

**E7.** *(ch. 07)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Le paramètre `degats`
doit être obligatoire ; `magique` reste facultatif.

```dart
String decrireArme(String nom, {/* 1 */ int degats, bool magique = false}) {
  String texte = '$nom : $degats dégâts';
  if (magique) {
    texte = '$texte (magique)';
  }
  return texte;
}

void main() {
  print(decrireArme('Épée', degats: 10));
  print(decrireArme('Bâton', /* 2 */));
}
```

```text
Épée : 10 dégâts
Bâton : 25 dégâts (magique)
```

---

**E8.** *(ch. 08)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Le paramètre de
`definirNom` porte volontairement le même nom que la propriété.

```dart
class Joueur {
  String nom = 'Inconnu';
  int vies = 3;

  void definirNom(String nom) {
    /* 1 */ = nom;
  }

  void perdreVie() {
    vies--;
  }
}

void main() {
  Joueur j = /* 2 */;
  j.definirNom('Alex');
  j.perdreVie();
  print('${j.nom} : ${j.vies} vies');
}
```

```text
Alex : 2 vies
```

---

**E9.** *(ch. 09)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Une potion est dite
« puissante » à partir de 50 points de soin.

```dart
class Potion {
  final String nom;
  final int soin;

  Potion({required this.nom, required this.soin});

  bool get estPuissante => /* 1 */;

  @override
  String /* 2 */ => 'Potion($nom, +$soin PV)';
}

void main() {
  Potion elixir = Potion(nom: 'Élixir', soin: 60);
  Potion petite = Potion(nom: 'Petite potion', soin: 20);

  print(elixir);
  print('${elixir.nom} puissante ? ${elixir.estPuissante}');
  print('${petite.nom} puissante ? ${petite.estPuissante}');
}
```

```text
Potion(Élixir, +60 PV)
Élixir puissante ? true
Petite potion puissante ? false
```

---

**E10.** *(ch. 10)* **(1 pt)** Complétez pour obtenir la sortie indiquée. Les points de vie sont
stockés dans un champ privé et exposés en lecture seule ; le constructeur de `Boss` doit
transmettre le nom et les points de vie au constructeur parent.

```dart
class Personnage {
  final String nom;
  int _pv;

  Personnage(this.nom, this._pv);

  int get pv => /* 1 */;

  String decrire() => '$nom ($pv PV)';
}

class Boss extends Personnage {
  final String titre;

  Boss(String nom, int pv, this.titre) : /* 2 */;

  @override
  String decrire() => '${super.decrire()} - $titre';
}

void main() {
  Boss b = Boss('Draknor', 500, 'Seigneur des flammes');
  print(b.decrire());
  print('PV : ${b.pv}');
}
```

```text
Draknor (500 PV) - Seigneur des flammes
PV : 500
```

---

---

## PARTIE F — QUESTIONS DE COURS RÉDIGÉES

**Durée conseillée : 30 minutes — 10 points (5 questions × 2 points)**

Consignes : rédigez chaque réponse en 5 à 15 lignes, en français, avec des phrases complètes.
Lorsque la question s'y prête, appuyez votre explication sur un exemple de code court écrit en
Dart. Les exemples doivent rester dans le périmètre des chapitres 01 à 10.

---

### F1 — `var`, `final`, `const` et `dynamic` *(ch. 02)* **(2 pts)**

Expliquez ce que garantit chacun de ces quatre mots-clés et **à quel moment** cette garantie
s'applique (à la compilation ou à l'exécution).

Votre réponse doit préciser :

- ce que `var` fixe et ce qu'il laisse libre ;
- la différence essentielle entre `final` et `const` ;
- pourquoi `dynamic` est le plus permissif des quatre, et le risque que cela introduit.

Donnez un exemple de code court pour chacun des quatre mots-clés.

---

### F2 — `List`, `Set` et `Map` *(ch. 05-06)* **(2 pts)**

Comparez les trois collections `List`, `Set` et `Map` selon quatre axes :

- leur **structure** (comment les valeurs sont organisées) ;
- l'**unicité** des valeurs (les doublons sont-ils autorisés ?) ;
- le mode d'**accès** à un élément ;
- un **cas d'usage de jeu vidéo** concret pour chacune.

Illustrez votre réponse par une déclaration de chaque collection.

---

### F3 — Les trois familles de paramètres *(ch. 07)* **(2 pts)**

Une fonction Dart peut recevoir des paramètres **positionnels**, des paramètres
**positionnels optionnels** et des paramètres **nommés**.

Pour chacune des trois familles, indiquez :

- la **syntaxe** de déclaration ;
- le **comportement** à l'appel (ordre imposé ou libre, obligatoire ou facultatif) ;
- le critère qui vous fait **choisir** cette famille plutôt qu'une autre.

Écrivez une signature de fonction illustrant chaque famille.

---

### F4 — Classe, objet et constructeurs *(ch. 08-09)* **(2 pts)**

Répondez aux trois points suivants :

1. Quelle est la différence entre une **classe** et un **objet** ? Utilisez une analogie
   pour la rendre claire.
2. Quel est le **rôle du constructeur** ? Que se passe-t-il si l'on n'en écrit aucun ?
3. Qu'apporte un **constructeur nommé** par rapport à un unique constructeur à rallonge
   comportant de nombreux paramètres ?

Illustrez le point 3 par un exemple de code court sur une classe `Ennemi`.

---

### F5 — Les trois piliers de la POO *(ch. 10)* **(2 pts)**

Définissez en **une phrase chacun** :

- l'**encapsulation** ;
- l'**héritage** ;
- le **polymorphisme**.

Illustrez ensuite les **trois notions à la fois** sur un seul exemple de code organisé
autour d'une classe `Personnage` et de ses deux classes filles `Joueur` et `Ennemi`.
Votre exemple doit contenir au moins un champ privé, un getter, un `extends`, un `super`
et une méthode redéfinie avec `@override`.

---

---

## PARTIE G — EXERCICES DE PROGRAMMATION

**Durée conseillée : 75 minutes — 25 points**

Consignes générales :

- Écrivez du Dart exécutable tel quel dans DartPad, avec un `void main()`.
- Indentation de 2 espaces. Nommez vos variables en `lowerCamelCase`.
- N'utilisez que les notions des chapitres 01 à 10 : ni `try/catch`, ni `map()` / `where()` /
  `fold()` / `sort()`, ni `?`, `??`, `!`, ni `abstract` ou `enum`.
- **La sortie de votre programme doit être strictement identique à celle indiquée**, espaces,
  ponctuation et accents compris.

| Exercice | Sujet | Points |
| --- | --- | ---: |
| G1 | Statistiques de partie | 3 |
| G2 | Grille de donjon | 4 |
| G3 | Inventaire | 5 |
| G4 | Classe `Personnage` | 6 |
| G5 | Héritage et polymorphisme | 7 |
| | **Total** | **25** |

---

### G1 — Statistiques de partie *(ch. 02, 03, 05)* **(3 pts)**

On dispose des scores des sept manches d'une partie :

```dart
List<int> scores = [150, 320, 90, 475, 210, 60, 380];
```

Écrivez un programme qui affiche, dans cet ordre :

1. la liste complète des scores ;
2. le nombre de manches ;
3. le **total** des scores ;
4. la **moyenne**, affichée avec exactement **deux décimales** grâce à `toStringAsFixed(2)` ;
5. le **meilleur** score ;
6. le **pire** score ;
7. l'**écart** entre le meilleur et le pire.

**Contraintes**

- Le total, le meilleur et le pire doivent être obtenus avec **une seule boucle** et des
  variables accumulatrices.
- Aucune méthode de collection avancée n'est autorisée : ni `reduce()`, ni `fold()`, ni
  `sort()`, ni `List.max`. En dehors de `length`, de l'indexation et de la boucle `for-in`,
  aucune méthode de `List` ne doit être utilisée.
- N'écrivez aucune valeur numérique en dur dans les `print()` : tout doit être calculé.

**Sortie attendue**

```text
Scores de la partie : [150, 320, 90, 475, 210, 60, 380]
Nombre de manches : 7
Total : 1685
Moyenne : 240.71
Meilleur score : 475
Pire score : 60
Écart : 415
```

---

### G2 — Grille de donjon *(ch. 05, 06)* **(4 pts)**

Un donjon est représenté par une grille de caractères. Le symbole `#` est un mur, le symbole
`.` est une case libre, et le symbole `J` est la position du joueur.

```dart
List<List<String>> donjon = [
  ['#', '#', '#', '#', '#', '#'],
  ['#', '.', '.', '#', '.', '#'],
  ['#', '.', 'J', '.', '.', '#'],
  ['#', '#', '.', '.', '#', '#'],
  ['#', '.', '.', '.', '.', '#'],
  ['#', '#', '#', '#', '#', '#'],
];
```

Écrivez un programme qui, **en un seul parcours à boucles imbriquées** :

1. affiche la grille, une ligne par ligne, chaque ligne étant la concaténation de ses symboles
   (sans espace ni virgule) ;
2. compte le nombre total de murs ;
3. repère la position du joueur et la restitue sous la forme `ligne L, colonne C`, les indices
   commençant à `0`.

**Contraintes**

- Deux boucles imbriquées sont **obligatoires** : la boucle externe parcourt les lignes, la
  boucle interne parcourt les colonnes.
- Les dimensions doivent être lues sur la grille (`donjon.length` et `donjon[0].length`) et
  non écrites en dur.
- Interdiction d'utiliser `join()`, `toString()` sur une ligne, ou une méthode de recherche
  toute faite : construisez la chaîne d'affichage caractère par caractère.

**Sortie attendue**

```text
Grille du donjon :
######
#..#.#
#.J..#
##..##
#....#
######
Dimensions : 6 x 6
Nombre de murs : 23
Position du joueur : ligne 2, colonne 2
```

---

### G3 — Inventaire *(ch. 06, 07)* **(5 pts)**

L'inventaire du joueur est une `Map<String, int>` associant un nom d'objet à une quantité.

Écrivez les **cinq fonctions** suivantes, toutes en dehors de `main()` :

| Fonction | Signature | Rôle |
| --- | --- | --- |
| `quantiteDe` | `int quantiteDe(Map<String, int> sac, String objet)` | Renvoie la quantité d'un objet, ou `0` s'il est absent |
| `ajouter` | `void ajouter(Map<String, int> sac, String objet, int quantite)` | Ajoute la quantité, en créant la clé si nécessaire |
| `retirer` | `void retirer(Map<String, int> sac, String objet, int quantite)` | Retire la quantité **sans jamais descendre sous zéro** ; si la quantité atteint `0`, la clé est **supprimée** de la `Map` |
| `contient` | `bool contient(Map<String, int> sac, String objet)` | Indique si l'objet est présent |
| `afficherInventaire` | `void afficherInventaire(Map<String, int> sac)` | Affiche le nombre d'entrées puis une ligne par objet |

**Contraintes**

- Interdiction d'utiliser `?`, `??` ou `!` : pour lire une quantité, passez par `quantiteDe`,
  construit avec `containsKey()` ou un parcours de `sac.entries`.
- `afficherInventaire` affiche d'abord `Inventaire (N objets)` puis, pour chaque objet, une
  ligne indentée de deux espaces au format `  Nom xQuantité`, dans l'ordre d'insertion de la
  `Map`.

**`main()` imposé**

```dart
void main() {
  Map<String, int> sac = {
    'Potion': 3,
    'Épée': 1,
    'Clé': 2,
  };

  print('Inventaire de départ');
  afficherInventaire(sac);

  ajouter(sac, 'Potion', 2);
  ajouter(sac, 'Bouclier', 1);
  retirer(sac, 'Clé', 2);
  retirer(sac, 'Potion', 1);
  retirer(sac, 'Épée', 5);

  bool aPotion = contient(sac, 'Potion');
  bool aCle = contient(sac, 'Clé');
  print('Contient Potion : $aPotion');
  print('Contient Clé : $aCle');

  print('Inventaire final');
  afficherInventaire(sac);
}
```

**Sortie attendue**

```text
Inventaire de départ
Inventaire (3 objets)
  Potion x3
  Épée x1
  Clé x2
Contient Potion : true
Contient Clé : false
Inventaire final
Inventaire (2 objets)
  Potion x4
  Bouclier x1
```

---

### G4 — Classe `Personnage` *(ch. 08, 09)* **(6 pts)**

Écrivez une classe `Personnage` respectant **exactement** la description suivante.

**Propriétés**

- `nom` : `final String` ;
- `pointsDeVieMax` : `final int` ;
- `niveau` : `int` ;
- `pointsDeVie` : `int`, jamais fourni par l'appelant, toujours égal à `pointsDeVieMax` à la
  création.

**Constructeur principal** — à **paramètres nommés** :

- `nom` est `required` ;
- `pointsDeVieMax` vaut `100` par défaut ;
- `niveau` vaut `1` par défaut ;
- `pointsDeVie` n'est **pas** un paramètre : il est initialisé à `pointsDeVieMax` dans la
  **liste d'initialisation**.

**Constructeur nommé** `Personnage.heros(String nom)` : redirige vers le constructeur
principal avec `pointsDeVieMax: 150` et `niveau: 5`.

**Getters calculés**

- `bool get estVivant` : vrai si les points de vie sont strictement supérieurs à `0` ;
- `int get pourcentageVie` : pourcentage **entier** de vie restante, calculé par division
  entière (`~/`).

**Méthodes**

- `void subirDegats(int degats)` : retire les dégâts, sans jamais descendre sous `0` ;
- `void soigner(int soin)` : ajoute les soins, sans jamais dépasser `pointsDeVieMax` ;
- `String toString()` redéfini avec `@override`, au format
  `nom (niveau N) - PV/PVMAX PV (P%)`.

**Contraintes**

- Aucune méthode ne doit rien afficher : seuls les `print()` de `main()` produisent la sortie.
- Le bornage doit être écrit dans les méthodes, pas dans `main()`.

**`main()` imposé**

```dart
void main() {
  Personnage gobelin = Personnage(nom: 'Gobelin');
  Personnage heroine = Personnage.heros('Aria');

  print(gobelin);
  print(heroine);

  gobelin.subirDegats(30);
  print(gobelin);

  gobelin.subirDegats(200);
  print(gobelin);
  print('Vivant : ${gobelin.estVivant}');

  heroine.subirDegats(60);
  print(heroine);

  heroine.soigner(25);
  print(heroine);

  heroine.soigner(500);
  print(heroine);
}
```

**Sortie attendue**

```text
Gobelin (niveau 1) - 100/100 PV (100%)
Aria (niveau 5) - 150/150 PV (100%)
Gobelin (niveau 1) - 70/100 PV (70%)
Gobelin (niveau 1) - 0/100 PV (0%)
Vivant : false
Aria (niveau 5) - 90/150 PV (60%)
Aria (niveau 5) - 115/150 PV (76%)
Aria (niveau 5) - 150/150 PV (100%)
```

---

### G5 — Héritage et polymorphisme *(ch. 10)* **(7 pts)**

Écrivez une hiérarchie de trois classes.

**Classe mère `Personnage`**

- `nom` : `final String` ; `pvMax` : `final int` ; `_pv` : champ **privé** de type `int` ;
- constructeur positionnel `Personnage(this.nom, this.pvMax)` qui initialise `_pv` à `pvMax`
  dans la liste d'initialisation ;
- getter `int get pv` et getter calculé `bool get estVivant` ;
- `void subirDegats(int degats)` : borne à `0` par le bas ;
- `void soigner(int soin)` : borne à `pvMax` par le haut ;
- `void attaquer()` qui affiche `nom attaque pour 5 dégâts.` ;
- `void afficherEtat()` qui affiche `nom : PV/PVMAX PV`.

**Classe fille `Joueur extends Personnage`**

- propriété supplémentaire `int potions` ;
- constructeur `Joueur(String nom, int pvMax, this.potions)` appelant `super(...)` ;
- `attaquer()` redéfini avec `@override` : il appelle **d'abord** `super.attaquer()`, puis
  affiche `  Bonus : coup critique, +10 dégâts.` (deux espaces d'indentation) ;
- `void boirePotion()` : s'il reste des potions, en consomme une, soigne de `20` PV et affiche
  `  Nom boit une potion (+20 PV). Potions restantes : N` ; sinon affiche
  `  Nom n'a plus de potion.`.

**Classe fille `Ennemi extends Personnage`**

- propriété supplémentaire `final int niveauMenace` ;
- constructeur `Ennemi(String nom, int pvMax, this.niveauMenace)` appelant `super(...)` ;
- `attaquer()` redéfini avec `@override` : il appelle `super.attaquer()`, puis affiche
  `  Menace de niveau N.`.

**Contraintes**

- `_pv` ne doit **jamais** être lu ni écrit depuis `main()` : passez par `pv`, `subirDegats`
  et `soigner`.
- La boucle de `main()` doit être écrite sur une `List<Personnage>` et **ne doit contenir
  aucun `if` sur le type**, à l'exception du test `is Joueur` demandé pour la potion.

**`main()` imposé**

```dart
void main() {
  List<Personnage> combattants = [
    Joueur('Aria', 120, 2),
    Ennemi('Gobelin', 20, 1),
    Ennemi('Orc', 80, 3),
  ];

  for (Personnage p in combattants) {
    p.attaquer();
    p.subirDegats(30);
    if (p is Joueur) {
      p.boirePotion();
    }
    p.afficherEtat();
    print('---');
  }

  int vivants = 0;
  for (Personnage p in combattants) {
    if (p.estVivant) {
      vivants++;
    }
  }
  print('Combattants encore en vie : $vivants');
}
```

**Sortie attendue**

```text
Aria attaque pour 5 dégâts.
  Bonus : coup critique, +10 dégâts.
  Aria boit une potion (+20 PV). Potions restantes : 1
Aria : 110/120 PV
---
Gobelin attaque pour 5 dégâts.
  Menace de niveau 1.
Gobelin : 0/20 PV
---
Orc attaque pour 5 dégâts.
  Menace de niveau 3.
Orc : 50/80 PV
---
Combattants encore en vie : 2
```

---

---

## Variante « sur machine »

Si l'épreuve se déroule avec un ordinateur et [DartPad](https://dartpad.dev), appliquez ces
ajustements :

| Partie | Ajustement |
| --- | --- |
| A, B, F | Inchangées. Ce sont des questions de compréhension : l'accès à une machine ne les fausse pas. |
| C | À supprimer, ou à remplacer par la consigne « prédisez la sortie **avant** d'exécuter, puis notez l'écart entre votre prédiction et le résultat ». Les 15 points sont alors reportés sur la partie G. |
| D | Inchangée, mais annoncez que l'exécution est autorisée : la note porte sur l'explication, pas sur la découverte. |
| E | Inchangée. |
| G | Inchangée, mais exigez la remise des fichiers `.dart`. Un code qui ne compile pas devient beaucoup plus sévèrement pénalisé, puisque l'étudiant pouvait le vérifier. |

Durée conseillée en version sur machine : 2 h 30 sans la partie C.

---

## Fin du sujet

Relisez la partie G : c'est le quart de la note.
